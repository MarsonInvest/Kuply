# Kuply — State Management

> Kde žije který stát, jak se přenáší a kdy se persistuje. Pravidla pro Claude Code, aby nevytvářel zbytečné globální story.

---

## 1 · Filozofie

**Server first.** Next.js 15 App Router preferuje server-side rendering a Server Actions. Globální klientský store (Zustand, Redux) je v 95 % případů antipattern — vede k duplikaci dat, hydration mismatchům a obtížnému debugování.

**Hierarchie preferencí (od nejlevnějšího po nejdražší):**
1. **URL state** (searchParams) — sdílí se přes link, F5 přežije.
2. **Server state** (RSC fetch) — single source of truth.
3. **Form state** (React Hook Form) — lokálně, optional autosave.
4. **Component state** (useState) — co opravdu jen do té komponenty.
5. **Context** (zřídka) — theme, locale, current user.
6. **External store** (Zustand) — jen pokud opravdu nutné (např. SofiaChat).

---

## 2 · State map — kde co žije

| Co | Kde | Persist | Příklad |
|---|---|---|---|
| **Filtry investora** | URL searchParams | URL | `/investor/dashboard?type=byt&city=praha-2` |
| **Pagination** | URL searchParams | URL | `?page=3` |
| **Current step in flow** | URL searchParams | URL | `/flow/private?step=3` |
| **Path selection (quick/private)** | useState v `<FlowSection>` | session | reset při refresh |
| **SpecForm rozpracovaný** | React Hook Form + localStorage autosave | session | obnovení po F5 |
| **Lead data** | Server (Supabase) | DB | RSC fetch v každé page |
| **Bids list** | Server | DB | RSC fetch |
| **Transaction status** | Server + Supabase Realtime | DB | live updates |
| **Sofia conversation** | Server + streaming | DB | sofia_conversations table |
| **Sofia drawer open/close** | useState v root layout | session | reset při refresh |
| **Carousel position** | useState (Framer Motion drag) | — | nic se nepersistuje |
| **Toast notifications** | Sonner queue | — | ephemeral |
| **Current user** | Supabase Auth session + Server | cookie | refresh přes middleware |
| **Theme (dark/light)** | next-themes (cookie) | cookie | persistent |
| **Locale (cs/sk/pl)** | next-intl (cookie) | cookie | persistent |
| **KYC verification result** | Server | DB | refetch on dashboard load |

---

## 3 · Konkrétní vzory

### A) URL state pro filtry investora

```tsx
// app/(auth)/investor/dashboard/page.tsx
export default async function Page({ searchParams }: { searchParams: Promise<{type?: string, city?: string, page?: string}> }) {
  const params = await searchParams;
  const leads = await getLeads({
    type: params.type,
    city: params.city,
    page: Number(params.page ?? 1),
  });
  return <LeadList leads={leads} />;
}

// components/investor/LeadFilters.tsx
'use client';
function LeadFilters() {
  const router = useRouter();
  const pathname = usePathname();
  const sp = useSearchParams();
  
  function setFilter(key: string, value: string | null) {
    const params = new URLSearchParams(sp);
    if (value) params.set(key, value);
    else params.delete(key);
    router.push(`${pathname}?${params}`);
  }
  // ...
}
```

**Proč:** Sdílí se přes link, F5 přežije, server route navigation = automatický refetch.

### B) Form state s autosave (SpecForm)

```tsx
// components/flow/SpecForm.tsx
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useLocalStorageAutosave } from '@/lib/hooks';

function SpecForm({ propertyType }: { propertyType: PropertyType }) {
  const schema = getSchemaFor(propertyType);
  const form = useForm({
    resolver: zodResolver(schema),
    defaultValues: useLocalStorageAutosave('spec-draft', { propertyType }),
  });
  
  // autosave každých 1s
  useEffect(() => {
    const sub = form.watch((values) => {
      localStorage.setItem('spec-draft', JSON.stringify(values));
    });
    return () => sub.unsubscribe();
  }, [form]);
  
  const completeness = computeCompleteness(form.watch(), schema);
  
  async function onSubmit(data: SpecForm) {
    const result = await createLead({ propertyType, spec: data, /*...*/ });
    if (result.ok) {
      localStorage.removeItem('spec-draft');
      router.push(`/flow/private/success?id=${result.leadId}`);
    }
  }
  // ...
}
```

**Proč:** Vyplňování trvá 5+ minut. Refresh nebo zavřená záložka = ztracený lead. Autosave do localStorage zabrání ztrátě.

### C) Server state s revalidation

```tsx
// app/(auth)/investor/lead/[id]/page.tsx
export default async function LeadPage({ params }: { params: Promise<{id: string}> }) {
  const { id } = await params;
  const lead = await getLeadForInvestor(id);  // server-side, RLS protected
  if (!lead) notFound();
  return <LeadDetail lead={lead} />;
}

// Server Action po submit bid:
'use server';
export async function submitBid(input: SubmitBidInput) {
  // ...
  revalidatePath(`/investor/lead/${input.lead_id}`);   // RSC re-fetch
  revalidatePath('/investor/dashboard');
}
```

**Proč:** RSC se sami refetchnou po `revalidatePath`. Žádný globální store, žádná manuální cache invalidation.

### D) Realtime updates pro DealTimeline

```tsx
'use client';
function DealTimeline({ initialData }: { initialData: Transaction }) {
  const [tx, setTx] = useState(initialData);
  
  useEffect(() => {
    const channel = supabase
      .channel(`transaction:${tx.id}`)
      .on('postgres_changes', {
        event: 'UPDATE',
        schema: 'public',
        table: 'transactions',
        filter: `id=eq.${tx.id}`,
      }, (payload) => {
        setTx(payload.new as Transaction);
        toast.success(`Krok ${payload.new.current_step} dokončen`);
      })
      .subscribe();
    return () => { channel.unsubscribe(); };
  }, [tx.id]);
  
  return <Timeline data={tx} />;
}
```

**Proč:** Supabase Realtime broadcasts → useState. Žádný global store potřeba.

### E) Sofia chat — výjimka (Zustand OK)

Sofia chat je jediný legitimní use-case pro globální store, protože:
- Drawer state musí přežít navigaci mezi stránkami
- Streaming response musí pokračovat i když user překlikne sekci
- Tools execution log je shared s analytics komponentou

```tsx
// stores/sofia-store.ts
import { create } from 'zustand';

export const useSofiaStore = create<SofiaState>((set, get) => ({
  isOpen: false,
  leadId: null,
  messages: [],
  isStreaming: false,
  
  open: (leadId) => set({ isOpen: true, leadId }),
  close: () => set({ isOpen: false }),
  sendMessage: async (content) => {
    set({ isStreaming: true });
    const stream = await fetch(`/api/sofia/${get().leadId}`, {
      method: 'POST',
      body: JSON.stringify({ content }),
    });
    // ... stream handling
    set({ isStreaming: false });
  },
}));
```

**Pravidlo:** Pokud potřebuješ Zustand, musíš to obhájit v PR review. Default je: nepotřebuješ.

---

## 4 · Antipatterns (NIKDY)

### ❌ Global store pro server data
```tsx
// ŠPATNĚ
const useLeadsStore = create((set) => ({
  leads: [],
  fetchLeads: async () => set({ leads: await fetch('/api/leads').then(r => r.json()) }),
}));
```
**Proč ne:** Hydration mismatch, race conditions, žádná RLS, duplicita s RSC.

**Místo toho:** RSC fetch + `revalidatePath`.

### ❌ Lifting state up pro filter, který se sdílí přes link
```tsx
// ŠPATNĚ — filtr v useState v parentovi
function Dashboard() {
  const [filter, setFilter] = useState({});
  // refresh = ztrata filtru
}
```

**Místo toho:** URL searchParams.

### ❌ Local state pro autentizovaného uživatele
```tsx
// ŠPATNĚ
const [user, setUser] = useState<User | null>(null);
useEffect(() => { fetch('/api/me').then(r => r.json()).then(setUser); }, []);
```

**Místo toho:** `getCurrentUser()` v RSC nebo Context provider z middleware.

### ❌ Synchronizace formuláře s parent state
```tsx
// ŠPATNĚ
function Parent() {
  const [spec, setSpec] = useState({});
  return <Child spec={spec} onChange={setSpec} />;
}
```

**Místo toho:** React Hook Form interně, parent dostane výsledek až přes `onSubmit`.

---

## 5 · Hydration safety

Server Components + Client Components mají odlišný runtime. Vyhnout se:
- `window`, `document`, `localStorage` v initial render → vždy v `useEffect`.
- Date/random/uuid bez `'use client'` → mismatch.
- Reading cookies in components — místo toho v `layout.tsx` server side.

**Pravidlo:** Pokud máš varování `Hydration mismatch`, používáš state na špatném místě.

---

## 6 · Performance: kdy memoize

| Případ | Memoize? |
|---|---|
| Server Components | Cache via `next/cache` (built-in) |
| Client komponenty čisté (props in → JSX out) | `React.memo` jen pokud profiler řekne ano |
| Expensive computation (computeCompleteness) | `useMemo` jen pokud je v hot path |
| Callbacky předávané do memoized children | `useCallback` |

**Default:** nememoizovat. React 19 + RSC dělají optimalizaci jinde.

---

## 7 · Debugging state

| Nástroj | Použití |
|---|---|
| **React DevTools** | Component tree, state inspector |
| **Tanstack Query DevTools** | (nepoužíváme, ale kdyby ano) |
| **Supabase logs** | RLS denials, query plans |
| **PostHog session replay** | Vidět co user vyplňoval v SpecForm |
| **Sentry breadcrumbs** | Posloupnost actions před chybou |

---

## 8 · Pravidla pro Claude Code

1. **Default = state na serveru**. Pokud nemůžeš obhájit klient, neměj klient.
2. **URL > useState** pokud se má state sdílet linkem nebo přežít refresh.
3. **Žádný Zustand, dokud někdo neřekne ano v PR review.**
4. **Forms = React Hook Form + Zod**. Žádné `useState<string>` pro pole.
5. **Realtime = Supabase Realtime subscription, ne polling.**
6. **Cache invalidation = `revalidatePath` nebo `revalidateTag`**, nikdy manual.
7. **Optimistic updates** jen pro instantní UI feedback (toast „odesláno..."), ne pro autoritativní data.
8. **Branded types** (`SellerId`, `LeadId`, `PriceHaléř`) místo holých `string`/`number` — vynucené přes `tsconfig` paths.
