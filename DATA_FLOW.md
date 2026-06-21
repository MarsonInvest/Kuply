# Kuply — Data Flow

> Jak tečou data systémem. User → UI → API → DB. Hlavní use-cases s sekvenčními diagramy.

---

## 1 · High-level data flow

```
┌─────────┐                                                     ┌──────────┐
│  USER   │                                                     │   DB     │
└────┬────┘                                                     └────┬─────┘
     │                                                                │
     │  1. Browse landing (no auth)                                    │
     ├──→ Next.js RSC ──→ static (cache 1h)                           │
     │                                                                │
     │  2. Otevře privátní rozbor                                      │
     ├──→ Client component <FlowSection>                              │
     │                                                                │
     │  3. Vyplňuje SpecForm                                          │
     ├──→ useFormState (local), žádný server roundtrip                │
     │                                                                │
     │  4. Odešle finální krok                                        │
     ├──→ Server Action `createLead(formData)`                        │
     │     ├─ Zod validate                                            │
     │     ├─ supabase.insert(leads)  ───────────────────────────────→│
     │     ├─ supabase.insert(audit_log) ────────────────────────────→│
     │     ├─ Resend.send(emailToSeller)                              │
     │     ├─ Resend.send(emailToOps)                                 │
     │     └─ revalidatePath('/flow')                                 │
     │                                                                │
     │  5. Redirect na /flow/private/success                          │
     └──→ RSC s lead.id (z cookies/searchParams)                      │
                                                                       │
                                                                       │
┌──────────┐                                                          │
│ INVESTOR │                                                          │
└────┬─────┘                                                          │
     │  6. Otevírá investor dashboard                                 │
     ├──→ Server Component `<DashPage>`                               │
     │     ├─ supabase.from('investor_lead_view').select() ──────────→│
     │     │   (RLS filtruje: jen open_for_bids, KYC verified)        │
     │     └─ render <LeadCard /> × N                                 │
     │                                                                │
     │  7. Submit bid                                                 │
     ├──→ Server Action `submitBid(leadId, formData)`                 │
     │     ├─ Zod validate                                            │
     │     ├─ Stripe.createPaymentIntent (commitment deposit)         │
     │     ├─ supabase.insert(bids) ─────────────────────────────────→│
     │     ├─ supabase.update(leads SET status=has_bids) ────────────→│
     │     ├─ notifications.queue(seller, 'new_bid')                  │
     │     └─ revalidatePath('/investor/dashboard')                   │
     │                                                                │
                                                                       │
┌──────────┐                                                          │
│  SELLER  │  8. Notifikace „máte nabídku"                           │
└────┬─────┘                                                          │
     │  9. Otevírá detail lead, vidí všechny bids                    │
     ├──→ RSC s RLS policy `seller_views_bids_on_own_lead`           │
     │                                                                │
     │  10. Klikne „Přijmout" na bid                                  │
     ├──→ Server Action `acceptBid(bidId)`                            │
     │     ├─ TRANSACTION:                                            │
     │     │   ├─ bids SET status=accepted ──────────────────────────→│
     │     │   ├─ bids SET status=rejected WHERE lead_id=X AND id!=Y →│
     │     │   ├─ leads SET status=offer_accepted ──────────────────→│
     │     │   └─ transactions INSERT (current_step=1) ──────────────→│
     │     ├─ Sofia.notify(investor, 'accepted')                     │
     │     ├─ Trinity Legal.intake_webhook                            │
     │     └─ realtime broadcast (Supabase Realtime)                  │
     │                                                                │
     │  11. Timeline auto-advance (DealTimeline)                      │
     └──→ Supabase Realtime subscription ←──────────────────────────  │
          ├─ webhook ČSOB (escrow funded) ──→ step 8                  │
          ├─ webhook ČÚZK (transfer) ──→ step 9                       │
          └─ manual ops (signature, etc.) ──→ admin updates           │
```

---

## 2 · Hlavní use-cases (sekvenční detail)

### UC-1: Anonymní tržní odhad (Path A)

```
Actor: Anonymous visitor
Goal:  Dostat cenové rozpětí bez registrace

Steps:
1. GET /                          → Landing page (RSC, cached)
2. Klik <HeroCarousel3D> card 0   → setPath('quick'), client state
3. setStep(0) → TypePicker        → pickType('byt')
4. setStep(1) → CityPicker        → pickCity('Praha 2')
5. setStep(2) → m² Input          → submitM2(84)
6. Compute na klientovi:
   - priceData = PRICE_DATA['p2']
   - min = 95000 × 84 × 1.0 = 7 980 000
   - max = 135000 × 84 × 1.0 = 11 340 000
7. Render <Result /> s rozpětím
8. CTA „Chci přesnější odhad" → setPath('private') → upgrade flow

API calls:    0
DB writes:    0 (pouze analytics event do PostHog)
PII:          žádné
```

### UC-2: Privátní rozbor → 100% zdravý lead (Path B) — kritická cesta

```
Actor: Seller (anonymous → identified at step 5)
Goal:  Dodat lead s úplnými daty (SpecForm) pro investory

Steps:
1. setPath('private')              → <FlowSection> auto-scroll center
2. setStep(0) TypePicker           → pickType('byt')
   ├─ useEffect: console-wrap.scrollIntoView(center)
3. setStep(1) AddressAutocomplete  → typing 'Vinohradská 12'
   ├─ debounced GET /api/geo?q=...  → suggestions
   ├─ pick → setAddress(suggestion)
4. setStep(2) phase='spec' <SpecForm>
   ├─ render SPEC_FIELDS['byt']     → 5 polí (2 povinná)
   ├─ user picks 2+kk, Osobní      → completeness 40%
   ├─ + č. jednotky 1234/5         → completeness 60%
   ├─ + Balkon                     → completeness 80%
   ├─ reqOk == true                → badge ZDRAVÝ LEAD svítí
   └─ klik 'Pokračovat'            → submitSpec → setStep 3
       ├─ console-wrap.scrollIntoView(center)
       └─ setPhase2('m2')
5. setStep(2) phase='m2'           → m² input + slider
6. setStep(3) QualitativeQuestions → 5 single-select otázek
7. setStep(4) MotivationStep       → pickMotivation('rozvod')
8. setStep(5) ContactSubmit        → name+phone+email+GDPR
9. submitLead Server Action:
   ├─ Zod LeadSchema.parse(payload)
   ├─ supabase.rpc('create_lead_atomic', {...})
   │   ├─ INSERT sellers (if new)
   │   ├─ INSERT leads
   │   └─ INSERT audit_log
   ├─ Resend: e-mail prodávajícímu (poděkování + co následuje)
   ├─ Resend: e-mail Marson Invest (notifikace nového leadu)
   ├─ Twilio: SMS s ověřovacím kódem (verify telefon)
   └─ Anthropic: pre-generate Sofia opening message for chat
10. redirect '/flow/private/success?id=...'

API calls:    1× geo, 1× submit
DB writes:    sellers, leads, audit_log, sofia_conversations
PII:          stored encrypted, RLS protected
```

### UC-3: Investor procházení leadů + submit bid

```
Actor: Investor (KYC verified)
Goal:  Najít vhodný lead a podat slepou nabídku

Steps:
1. GET /investor/dashboard
   ├─ middleware.ts: check session, KYC status
   ├─ Server Component fetches investor_lead_view
   │   (RLS: WHERE status IN ('open_for_bids','has_bids') AND kyc='verified')
   ├─ render <LeadCard /> × 12 (paginated)
2. <LeadFilters /> change → URL searchParams → re-fetch RSC
3. Klik <LeadCard> → GET /investor/lead/[id]
   ├─ supabase.from('investor_lead_view').select().eq('id', id)
   ├─ NIKDY se nevrátí: address_full, seller.email, seller.phone
4. Klik 'Podat nabídku' → <Dialog><BidForm /></Dialog>
5. Vyplnit price, podmínky, datum
6. Submit:
   ├─ Stripe.PaymentIntent (commitment 1% z bid amount, refundable)
   ├─ Stripe.confirmPayment (client SDK)
   ├─ Server Action `submitBid`:
   │   ├─ Zod BidSchema.parse
   │   ├─ supabase.insert(bids, status='submitted', expires_at=+7d)
   │   ├─ supabase.update(leads SET status='has_bids')
   │   ├─ notifications.queue(seller_id, 'new_bid', { lead_id, anonymized })
   │   ├─ audit_log.insert
   │   └─ revalidate
7. Toast: 'Nabídka odeslána. Prodávající má 7 dní.'

API calls:    Stripe + submitBid
DB writes:    bids, leads (status), audit_log, notifications
```

### UC-4: Seller přijme bid → start transakce

```
Actor: Seller
Goal:  Vybrat nabídku a posunout do legal/escrow fáze

Steps:
1. SMS notifikace 'Máte X nabídek'
2. Klik link → /flow/private/leads/[id]/bids
   ├─ RLS: lead.seller_id = current user
   ├─ render všechny bids s anonymizovaným investorem ('Investor A', 'Investor B')
3. Compare bids (cena × podmínky × doba)
4. Klik 'Přijmout' na konkrétní bid
5. Confirm dialog (irreversible action)
6. Server Action `acceptBid(bidId)`:
   ├─ pg TRANSACTION:
   │   ├─ UPDATE bids SET status='accepted' WHERE id = bidId
   │   ├─ UPDATE bids SET status='rejected' WHERE lead_id = X AND id != bidId
   │   ├─ UPDATE leads SET status='offer_accepted'
   │   ├─ INSERT transactions (lead_id, bid_id, current_step=1)
   │   └─ INSERT audit_log
   ├─ Anthropic: generate seller-facing summary
   ├─ Resend: e-mail vítěznému investorovi (kontakt seller, address)
   ├─ Resend: e-mail ostatním investorům (rejection, refund 1%)
   ├─ Stripe.refund() × N rejected bids
   ├─ Webhook → Trinity Legal intake API (start contract prep)
   └─ Supabase Realtime broadcast → buyer dashboard live update

7. Transaction → DealTimeline auto-progresses
```

### UC-5: Sofia AI conversation

```
Actor: Seller (mid-flow)
Goal:  Zeptat se Sofii uprostřed vyplňování

Steps:
1. Klik <SofiaFlight /> orb → <SofiaChat /> drawer opens
2. GET /api/sofia/[leadId]?stream=true
   ├─ load conversation history z sofia_conversations
   ├─ load lead context (spec, phase, motivation)
3. User type: 'Co když mám hypotéku?'
4. POST /api/sofia message
   ├─ Anthropic Messages API stream=true
   ├─ system prompt = base + lead context (Czech)
   ├─ tools available:
   │   - lookup_district_prices(district)
   │   - check_hypoteka_workflow()
   │   - escalate_to_human(reason)
   ├─ stream chunks → client SSE
5. UI streams response token-by-token
6. POST end → supabase.update(sofia_conversations SET messages = [...])
```

---

## 3 · Data validation flow

### Zod schémata jako single source of truth

```ts
// lib/schemas/lead.ts

export const BytSpecSchema = z.object({
  disp: z.enum(['1+kk','1+1','2+kk','2+1','3+kk','3+1','4+kk','4+1','5+kk a více','Atypický']),
  vlast: z.enum(['Osobní','Družstevní']),
  unit: z.string().optional(),
  floor: z.string().optional(),
  extra: z.array(z.enum(['Balkon','Lodžie','Terasa','Sklep','Garáž / stání','Výtah'])).optional(),
});

export const DumSpecSchema = z.object({
  typst: z.enum(['Samostatný','Řadový','Dvojdům']),
  plot: z.number().int().positive(),
  floors: z.enum(['Přízemní','Patrový','Vícepodlažní']).optional(),
  extra: z.array(z.enum(['Garáž','Bazén','Sklep','Vedlejší stavba'])).optional(),
});

// ... další typy

export const LeadSchema = z.discriminatedUnion('property_type', [
  z.object({ property_type: z.literal('byt'),     spec: BytSpecSchema, /* common fields */ }),
  z.object({ property_type: z.literal('dum'),     spec: DumSpecSchema, /*...*/ }),
  // ...
]);
```

**Validace běží na 3 místech:**
1. **Client (React Hook Form + zodResolver)** — okamžitý UX feedback
2. **Server Action** — autoritativní, nesmí důvěřovat klientovi
3. **DB CHECK constraints** — last line of defense (např. `m2 > 0`)

---

## 4 · State management

| Stát | Kde žije | Persist | Příklad |
|---|---|---|---|
| **URL state** | `searchParams` | URL | filtry investora, current step |
| **Form state** | React Hook Form | session (autosave do localStorage) | rozpracovaný SpecForm |
| **Server state** | Supabase via RSC | DB | leads, bids, transactions |
| **Realtime** | Supabase Realtime subscription | — | DealTimeline updates |
| **Client state** | useState/useReducer | — | UI toggly, carousel pos |
| **Cross-component** | Context (zřídka) / props drilling | — | theme, locale |
| **AI state** | server, streamed | sofia_conversations | Sofia chat |

**NIKDY do globálního store** (Zustand, Redux) — App Router preferuje co nejvíc na serveru. Pokud nějaký globální state přesto vznikne (např. session user), context provider stačí.

---

## 5 · Notifikace flow

```
Trigger event (Server Action)
       │
       ▼
INSERT notifications (status='pending')
       │
       ▼
Vercel Cron / Supabase scheduled function (every 30s)
       │
       ├─ SELECT WHERE status='pending' LIMIT 50
       ├─ for each:
       │   ├─ render template (channel)
       │   ├─ Resend.send(email) || Twilio.send(sms)
       │   ├─ UPDATE status='sent' || 'failed'
       │   └─ webhook callback → status='delivered' || 'opened'
```

**Šablony:**
- `lead.received` (seller) — poděkování, co následuje
- `lead.new_open` (ops) — nový lead k validaci
- `bid.received` (seller) — nová nabídka, link
- `bid.accepted` (investor) — vyhráli jste, kontakt seller
- `bid.rejected` (investor) — odmítnuto, refund
- `tx.escrow_funded` (seller + investor)
- `tx.completed` (seller + investor) — gratulace

---

## 6 · Audit log (compliance + GDPR)

**Každá modifikace = záznam.**

```ts
await db.audit_log.insert({
  actor_id: user.id,
  actor_role: 'investor',
  action: 'bid.accepted',
  resource: 'bid',
  resource_id: bidId,
  payload: { from: 'submitted', to: 'accepted', lead_id, price },
  ip_address: req.ip,
});
```

**Použití:**
- GDPR data export (seller si může stáhnout vše, co o něm máme).
- Forenzika („kdo a kdy přijal tuto nabídku?").
- Anti-fraud (podezřele rychlé akce).

---

## 7 · Real-time updates (Supabase Realtime)

**Subscribed channels:**
- `transaction:{id}` — DealTimeline updates pro buyer + seller.
- `lead:{id}` — nové bids → live count u prodávajícího.

```ts
// klient
useEffect(() => {
  const channel = supabase
    .channel(`transaction:${txId}`)
    .on('postgres_changes',
      { event: 'UPDATE', schema: 'public', table: 'transactions', filter: `id=eq.${txId}` },
      (payload) => setTimeline(payload.new))
    .subscribe();
  return () => { channel.unsubscribe(); };
}, [txId]);
```

---

## 8 · GDPR-critical data flows

| Akce | Co se děje |
|---|---|
| Seller submit lead | `address_full` šifrováno před INSERT (pgcrypto). `address_public` derived (jen čtvrť). |
| Investor view lead | RLS view skrývá PII. K odemčení potřeba accepted bid. |
| Seller žádá export | RPC `gdpr_export(user_id)` → JSON s vším, e-mail s odkazem. |
| Seller žádá smazání | RPC `gdpr_delete(user_id)` → leads anonymizovány (jméno → 'Smazaný uživatel'), audit_log zachován pro compliance. |
| KYC investora | Doklady v Supabase Storage, encrypted bucket, retention 5 let dle AML. |
| Cookie consent | `next-cookie-consent`, žádný analytics tracker bez souhlasu. |
