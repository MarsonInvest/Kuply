# Kuply

> Privátní AI-řízený realitní operating system. Prodávající dostává tři pohledy na cenu a AI průvodce Sofii. Investor dostává 100 % zdravé leady a slepé nabídky bez veřejné inzerce.

**Status:** Pre-production · Prototype validated (single-file HTML build v10) · Migrace do Next.js stacku.

---

## Quick start

```bash
# 1. Naklonovat repo
git clone https://github.com/marson-invest/kuply.git
cd kuply

# 2. Nainstalovat dependencies (pnpm preferováno)
pnpm install

# 3. Nastavit environment
cp .env.local.example .env.local
# vyplnit Supabase, Anthropic, Twilio, Resend, Stripe credentials

# 4. Spustit Supabase locally (volitelné, jinak cloud)
pnpm supabase:start
pnpm supabase:migrate

# 5. Seedovat dev data
pnpm db:seed

# 6. Dev server
pnpm dev
# → http://localhost:3000
```

**Otestovat lokálně:**
- Landing: http://localhost:3000
- Privátní rozbor flow: http://localhost:3000/flow/private
- Investor dashboard (mock auth): http://localhost:3000/investor/dashboard

---

## Co je v projektu

Kuply spojuje **prodávající nemovitostí** s **prověřenými investory** ve formátu **slepých nabídek bez veřejné inzerce**. Klíčové vlastnosti:

- **Tři pohledy na cenu** — veřejný odhad ± 15 % / bankovní odhad / privátní cena ± 3 %.
- **AI průvodce Sofia** — v češtině, v kontextu konkrétního leadu, ne generický chatbot.
- **SpecForm s živým měřičem kompletnosti** — investorům jdou jen 100 % zdravé leady (povinná pole dle typu nemovitosti: dispozice/LV/účel/...).
- **Slepé nabídky** — investor nikdy nevidí konkurenční nabídky, nedochází k aukci.
- **Trinity Legal + ČSOB úschova** — žádné peníze přes účet Kuply.
- **Pro prodávajícího 0 Kč** — marži platí investor.

**Více v** [`PROJECT_SPEC.md`](./docs/PROJECT_SPEC.md) (cíl, persony, MVP scope, success metriky).

---

## Tech stack (high-level)

| Vrstva | Technologie | Důvod |
|---|---|---|
| **Framework** | Next.js 15 (App Router, RSC) | Privátní data nikdy nejdou na klienta. |
| **Jazyk** | TypeScript strict | Realitní transakce nesmí padat. |
| **Styling** | Tailwind v3 + shadcn/ui + Framer Motion | Design tokens z prototypu zachovány. |
| **DB / Auth** | Supabase (Postgres + RLS + Storage + Realtime) | Row-Level Security pro GDPR-critical data. |
| **AI** | Anthropic Claude API (streaming) | Sofia s context window per lead. |
| **Validace** | Zod | Single source of truth pro klienta + server + DB. |
| **Notifikace** | Resend (e-mail) + Twilio (SMS) | Reliable transactional delivery. |
| **Payments** | Stripe (bid commitments) | KYC + refunds. |
| **Hosting** | Vercel (Edge functions) | EU region. |
| **Analytics** | PostHog (self-hosted) | GDPR-friendly. |
| **Error monitoring** | Sentry | Mandatory. |

**Více v** [`ARCHITECTURE.md`](./docs/ARCHITECTURE.md) (diagramy, RLS policies, deployment).

---

## Struktura repo

```
kuply/
├── app/                  # Next.js App Router
│   ├── (public)/        # landing + seller flow (no auth)
│   ├── (auth)/          # investor + tipar (auth + KYC required)
│   ├── api/             # Route handlers (webhooks)
│   └── actions/         # Server Actions (mutations)
├── components/
│   ├── ui/              # shadcn primitivy
│   ├── landing/         # HeroCarousel3D, ThesisTiltCards, ...
│   ├── flow/            # SpecForm, CompletenessMeter, ...
│   ├── sofia/           # SofiaOwl, SofiaFlight, SofiaChat
│   ├── investor/        # LeadCard, BidForm, DealTimeline
│   └── shared/          # GoldButton, ChipPicker, NoiseLayer
├── lib/
│   ├── supabase/        # client/server/middleware
│   ├── schemas/         # Zod schémata pro 6 typů nemovitostí
│   ├── ai/              # Anthropic client, system prompts, tools
│   ├── geo/             # district autocomplete, price data
│   └── utils.ts
├── design-system/
│   ├── tokens.css       # --paper, --gold, --cocoa, --rust, ...
│   └── tailwind.config  # token → utility mapping
├── supabase/
│   ├── migrations/      # SQL migrations
│   └── seed.sql         # dev data
├── tests/
│   ├── e2e/             # Playwright (seller flow, investor bid)
│   └── unit/            # Vitest
├── docs/
│   ├── PROJECT_SPEC.md
│   ├── ARCHITECTURE.md
│   ├── COMPONENTS.md
│   ├── DATA_FLOW.md
│   ├── API_SPEC.md
│   └── STATE_MANAGEMENT.md
├── public/
│   ├── owl/             # Sofia mascot SVG
│   └── noise.png        # 96px grain texture (hero only)
└── package.json
```

---

## Scripts

| Příkaz | Co dělá |
|---|---|
| `pnpm dev` | Next.js dev server na :3000 |
| `pnpm build` | Production build (Next.js + type-check) |
| `pnpm start` | Spustí produkční build |
| `pnpm lint` | ESLint + Prettier check |
| `pnpm typecheck` | `tsc --noEmit` |
| `pnpm test` | Vitest unit tests |
| `pnpm test:e2e` | Playwright e2e tests |
| `pnpm test:e2e:ui` | Playwright UI mode (interaktivní) |
| `pnpm supabase:start` | Spustí lokální Supabase (Docker) |
| `pnpm supabase:stop` | Zastaví lokální Supabase |
| `pnpm supabase:migrate` | Aplikuje migrations |
| `pnpm supabase:gen-types` | Generuje TS typy z DB schématu |
| `pnpm db:seed` | Seed dev data (10 leads, 3 investors, ...) |
| `pnpm storybook` | Storybook na :6006 |
| `pnpm analyze` | Bundle analyzer |

---

## Environment variables

Vše v `.env.local.example`:

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=     # server-only, never expose!

# Anthropic
ANTHROPIC_API_KEY=

# Resend (e-mail)
RESEND_API_KEY=

# Twilio (SMS, OTP)
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_VERIFY_SERVICE_SID=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# Google Maps (address autocomplete)
GOOGLE_MAPS_API_KEY=

# PostHog
NEXT_PUBLIC_POSTHOG_KEY=
NEXT_PUBLIC_POSTHOG_HOST=

# Sentry
SENTRY_DSN=
NEXT_PUBLIC_SENTRY_DSN=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_BASE_URL=http://localhost:3000  # legacy alias
```

---

## Development workflow

### Před commitem
1. `pnpm typecheck` — žádné TS errors.
2. `pnpm lint` — žádné lint errors.
3. `pnpm test` — unit testy projdou.
4. `pnpm test:e2e` — minimálně happy path e2e projde.
5. Manuálně otestovat `seller-flow` na 360/768/1380 viewportu.

### Git flow
- `main` = production (auto-deploy)
- `develop` = staging (auto-deploy na `staging.kuply.cz`)
- Feature branches: `feat/spec-form-pozemek`, `fix/sofia-stream-cancel`
- PR template vynucuje screenshot + checklist
- Squash merge do `develop`, merge commit do `main` (release tag)

### Conventional commits
```
feat(flow): add SpecForm validation for činžovní dům
fix(sofia): handle stream cancellation on tab close
chore(deps): bump @anthropic-ai/sdk to v0.30
docs(api): document /api/sofia streaming response
```

---

## Důležité konvence (vynucené v code review)

1. **Žádné `any` v TypeScriptu** — místo toho `unknown` + Zod parse.
2. **Žádné holé `useState<string>`** — branded types (`SellerId`, `LeadId`).
3. **Server Actions místo route handlerů** pro form submissions (pokud nejde o externí webhook).
4. **Žádný `console.log` v produkci** — logger.ts s úrovněmi.
5. **CZ texty NIKDY hardcoded v komponentě** — přes `next-intl` (`t('lead.spec.byt.dispozice')`).
6. **`prefers-reduced-motion` respektovat** — všechny animace fallback.
7. **RLS policy MUSÍ být na každé tabulce s PII**.
8. **Sofia prompt MUSÍ obsahovat:** „Jsi v kontextu konkrétního leadu, mluvíš česky, jsi empatická, nikdy nelžeš o ceně."

---

## Testing strategy

| Vrstva | Nástroj | Coverage cíl |
|---|---|---|
| **Pure functions** | Vitest | 90 %+ |
| **Zod schémata** | Vitest s fixture data | 100 % branches |
| **React komponenty** | Vitest + Testing Library | 70 %+ |
| **API routes** | Vitest + supertest-like | 80 %+ |
| **E2E happy paths** | Playwright | seller-flow, investor-bid, accept-bid, sofia-chat |
| **Visual regression** | Playwright snapshots | landing každý viewport |
| **a11y** | axe-playwright | žádné WCAG AA violations |

---

## Migration z prototypu

Existující single-file HTML build (`prototype/index.html`, build v10) obsahuje:

**Hotové komponenty k portu (1:1 stejné chování, vanilla → React + Framer):**
- HeroCarousel3D, ThesisTiltCards, ProcessCarousel, FomoBlock, StoriesStack, TrustGrid, FaqList, CinematicFooter
- SofiaOwl, SofiaFlight, SofiaBubble
- SpecForm s 6 typovými schématy
- CompletenessMeter

**K přepsání (vanilla JS → React):**
- Car3D engine → Framer Motion s drag/snap
- Sofia flight (rAF + scroll listener) → Framer's useScroll + useTransform
- CardsStack (CSS sticky) → zůstává CSS sticky, OK

**Nové komponenty (chybí v prototypu):**
- SofiaChat (AI streaming)
- LeadCard, BidForm
- DealTimeline (10 kroků)
- KYC flow pro investora
- Admin panel (Phase 2)

**Design tokens:** kompletně přeneseny z `p1_theme.css` → `design-system/tokens.css`.

**Kompletní katalog v** [`COMPONENTS.md`](./docs/COMPONENTS.md).

---

## Sledování chyb v Sentry — co je SEV-1

Cokoliv z níže = okamžitý alarm Romanovi + dev týmu.

- `lead.create` selhal → ztracený potenciální lead.
- `bid.accept` selhal v transakci → potenciálně nekonzistentní stav.
- Investor viděl PII (adresa, jméno) leadu, který by neměl vidět → GDPR incident.
- Sofia AI vrátila uživateli `'undefined'` nebo angličtinu → trust broken.

---

## Kontakt

- **Owner:** Roman Tulis, Marson Invest s.r.o.
- **E-mail:** roman@marsoninvest.cz
- **Sofi e-mail:** sofi@kuply.cz
- **Sofi telefon:** +420 731 211 822
- **Issue tracker:** GitHub Issues (interní pro team, public pro security disclosures via SECURITY.md)

---

## Licence

Proprietary © 2026 Marson Invest s.r.o. Všechna práva vyhrazena.
Není dovoleno použít kód, design ani strategii pro konkurenční účely.
