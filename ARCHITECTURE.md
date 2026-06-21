# Kuply — System Architecture

> **Status:** Production-ready spec · Stack: Next.js 15 App Router + Supabase + Czech-first locale
> **Audience:** Claude Code, full-stack devs, dev-ops

---

## 1 · High-level diagram

```
┌────────────────────────────────────────────────────────────────────────┐
│                          KUPLY OS · Production                          │
│                                                                          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│   │   PUBLIC     │  │   SELLER     │  │   INVESTOR   │  │   TIPAR    │ │
│   │   landing    │  │   flow       │  │   dashboard  │  │   portal   │ │
│   │   /          │  │   /flow      │  │   /investor  │  │   /tipar   │ │
│   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
│          └────────────┬─────┴──────────┬──────┴────────────────┘        │
│                       │  Next.js 15 (RSC + Server Actions)              │
│                       │  Tailwind + shadcn/ui + Framer Motion           │
│                       └────────┬────────────────────────────────────────┘
│                                │
│              ┌─────────────────┴──────────────────┐
│              │       API LAYER (Next.js)          │
│              │   ▸ Server Actions (mutations)     │
│              │   ▸ Route Handlers (webhooks)      │
│              │   ▸ Edge functions (geo, valid.)   │
│              │   ▸ Zod schemas (input validation) │
│              └────┬─────────────┬──────────┬──────┘
│                   │             │          │
│        ┌──────────▼──────┐  ┌───▼─────┐  ┌─▼──────────────┐
│        │   SUPABASE      │  │ THIRD   │  │   AI / Sofia    │
│        │   ▸ Postgres+RLS│  │ PARTY   │  │   ▸ Anthropic   │
│        │   ▸ Auth        │  │ ▸ Twilio│  │     Claude API  │
│        │   ▸ Storage     │  │ ▸ Resend│  │   ▸ context     │
│        │   ▸ Realtime    │  │ ▸ Stripe│  │     mgmt        │
│        │   ▸ pgvector    │  │ ▸ Maps  │  │   ▸ tools       │
│        └─────────────────┘  └─────────┘  └─────────────────┘
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2 · Frontend struktura

### Stack rationale
| Volba | Důvod |
|---|---|
| **Next.js 15 (App Router)** | RSC = privátní data nikdy nejdou na klienta. Server Actions pro form submissions. SEO pro landing. Edge runtime pro geo. |
| **TypeScript strict** | Realitní transakce nesmí padat na undefined. |
| **Tailwind v3** | V prototypu je `--paper`, `--gold`, `--cocoa`, `--rust` design systém — zachováme `tailwind.config.ts` token mapping. |
| **shadcn/ui** | Radix primitivy + customizace. NavigationMenu, Dialog, Form, Toast. |
| **Framer Motion** | 3D karusel, sticky stack, page transitions. Prototyp používá vanilla JS — produkce přepíše na Framer. |
| **next-intl** | Czech-first, ale připravené na multilang (Phase 4). |
| **React Hook Form + Zod** | SpecForm má 6 typů schémat — runtime + compile-time validace. |
| **Lucide icons** | Konzistence napříč shadcn. |
| **GSAP (optional)** | Pro cinematic footer parallax + Sofia flight entity. |

### Složková struktura
```
kuply-web/
├── app/
│   ├── (public)/                    # public route group, žádná auth
│   │   ├── page.tsx                 # landing
│   │   ├── flow/                    # seller flow
│   │   │   ├── quick/page.tsx       # Path A — Tržní odhad
│   │   │   ├── private/page.tsx     # Path B — Privátní rozbor
│   │   │   └── konzultace/page.tsx  # Path C — Rychlá konzultace
│   │   └── pribehy/[slug]/page.tsx  # detail příběhu (SEO)
│   ├── (auth)/                      # auth required
│   │   ├── investor/
│   │   │   ├── layout.tsx           # KYC gate
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── lead/[id]/page.tsx
│   │   │   └── bid/[leadId]/page.tsx
│   │   └── tipar/
│   │       └── dashboard/page.tsx
│   ├── api/
│   │   ├── webhooks/
│   │   │   ├── twilio/route.ts      # SMS delivery status
│   │   │   ├── stripe/route.ts      # bid commitment payments
│   │   │   └── escrow/route.ts      # ČSOB callback
│   │   ├── geo/route.ts             # adresa → district, mapy
│   │   └── sofia/route.ts           # AI streaming endpoint
│   ├── actions/                     # Server Actions (mutations)
│   │   ├── lead.ts                  # createLead, updateSpec, etc.
│   │   ├── bid.ts                   # submitBid, acceptBid
│   │   ├── kyc.ts
│   │   └── auth.ts
│   ├── layout.tsx
│   └── globals.css                  # Kuply design tokens
├── components/
│   ├── ui/                          # shadcn primitivy
│   ├── landing/
│   │   ├── HeroCarousel3D.tsx
│   │   ├── ThesisTiltCards.tsx
│   │   ├── ProcessCarousel.tsx
│   │   ├── FomoBlock.tsx
│   │   ├── StoriesStack.tsx
│   │   ├── TrustGrid.tsx
│   │   ├── FaqList.tsx
│   │   └── CinematicFooter.tsx
│   ├── flow/
│   │   ├── ConsoleShell.tsx         # framing kontejner
│   │   ├── StepProgress.tsx
│   │   ├── TypePicker.tsx
│   │   ├── AddressAutocomplete.tsx  # geo
│   │   ├── SpecForm.tsx             # typed dle typu nemovitosti
│   │   ├── CompletenessMeter.tsx
│   │   ├── QualitativeQuestions.tsx
│   │   ├── MotivationStep.tsx
│   │   └── ContactSubmit.tsx
│   ├── sofia/
│   │   ├── SofiaOwl.tsx             # mascot (DOM + CSS)
│   │   ├── SofiaFlight.tsx          # floating entity
│   │   ├── SofiaBubble.tsx          # contextual messages
│   │   └── SofiaChat.tsx            # full chat panel (AI)
│   ├── investor/
│   │   ├── LeadCard.tsx
│   │   ├── LeadFilters.tsx
│   │   ├── BidForm.tsx
│   │   └── DealTimeline.tsx         # 10-step timeline
│   └── shared/
│       ├── GoldButton.tsx
│       ├── ChipPicker.tsx
│       └── NoiseLayer.tsx           # hero grain
├── lib/
│   ├── supabase/
│   │   ├── client.ts                # browser client
│   │   ├── server.ts                # RSC + Server Actions
│   │   └── middleware.ts            # session refresh
│   ├── schemas/                     # Zod
│   │   ├── lead.ts                  # 6 typů: byt, dum, pozemek, ...
│   │   ├── bid.ts
│   │   └── kyc.ts
│   ├── ai/
│   │   ├── sofia.ts                 # Anthropic client
│   │   ├── system-prompt.ts         # czech, kuply-specific
│   │   └── tools.ts                 # function calls (geo, db lookup)
│   ├── geo/
│   │   ├── districts.ts             # data z prototypu (PRICE_DATA)
│   │   └── autocomplete.ts          # adresy CR
│   ├── analytics.ts                 # PostHog
│   ├── notifications.ts             # Twilio + Resend wrappers
│   └── utils.ts                     # cn(), formatPrice(), etc.
├── design-system/
│   ├── tokens.css                   # kompletní token systém z prototypu
│   ├── tailwind.config.ts           # token mapping
│   └── motion.ts                    # Framer presets
├── public/
│   ├── owl/                         # SVG sova (z prototypu)
│   ├── noise.png                    # 96px grain texture
│   └── og/                          # OpenGraph images
├── tests/
│   ├── e2e/                         # Playwright
│   │   ├── seller-flow.spec.ts      # privátní rozbor end-to-end
│   │   ├── investor-bid.spec.ts
│   │   └── landing.spec.ts
│   └── unit/                        # Vitest
├── docs/                            # tato dokumentace
├── .env.local.example
├── middleware.ts
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## 3 · Backend (Supabase) struktura

### Postgres schémata

```sql
-- ─── auth (Supabase managed) ───
-- auth.users — Supabase Auth

-- ─── public schema ───

-- Prodávající
CREATE TABLE sellers (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      uuid REFERENCES auth.users(id),
  email        text,
  phone        text,
  name         text,
  created_at   timestamptz DEFAULT now()
);

-- Investor
CREATE TABLE investors (
  id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         uuid REFERENCES auth.users(id) NOT NULL,
  company_name    text,
  ico             text,
  kyc_status      text CHECK (kyc_status IN ('pending','verified','rejected')),
  kyc_verified_at timestamptz,
  bank_account    text,                          -- masked
  created_at      timestamptz DEFAULT now()
);

-- Tipař
CREATE TABLE referrers (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      uuid REFERENCES auth.users(id),
  payout_iban  text,                             -- masked
  created_at   timestamptz DEFAULT now()
);

-- Lead = privátní rozbor v různých fázích
CREATE TABLE leads (
  id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  seller_id       uuid REFERENCES sellers(id),
  referrer_id     uuid REFERENCES referrers(id), -- optional
  property_type   text CHECK (property_type IN
                   ('byt','dum','pozemek','cinzak','komercni','developer')),
  address_full    text,                          -- PII, encrypted at rest
  address_public  text,                          -- "Praha 2 · Vinohrady" — co vidí investor
  district        text,
  city            text,
  m2              numeric,
  spec            jsonb NOT NULL,                -- typed dle property_type (Zod schema)
  qualitative     jsonb,                         -- stav, parkování, výtah, ...
  motivation      text,
  status          text DEFAULT 'draft'
                  CHECK (status IN
                   ('draft','submitted','validated','open_for_bids',
                    'has_bids','offer_accepted','in_legal','signed',
                    'in_escrow','transferred','completed','cancelled')),
  completeness_pct integer DEFAULT 0,
  created_at      timestamptz DEFAULT now(),
  updated_at      timestamptz DEFAULT now()
);

-- Nabídka investora
CREATE TABLE bids (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id       uuid REFERENCES leads(id) NOT NULL,
  investor_id   uuid REFERENCES investors(id) NOT NULL,
  price         bigint NOT NULL,                 -- v haléřích, abychom se vyhnuli decimal
  conditions    jsonb,                           -- doba do podpisu, výjimky, ...
  move_in_date  date,
  status        text DEFAULT 'submitted'
                CHECK (status IN
                 ('submitted','withdrawn','accepted','rejected','expired')),
  expires_at    timestamptz,                     -- default +7 dní
  created_at    timestamptz DEFAULT now()
);

-- Stav transakce (po accepted bid)
CREATE TABLE transactions (
  id                uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id           uuid REFERENCES leads(id) NOT NULL UNIQUE,
  bid_id            uuid REFERENCES bids(id) NOT NULL,
  current_step      integer DEFAULT 1,
  step_history      jsonb DEFAULT '[]'::jsonb,   -- log kdy se posunulo
  escrow_provider   text,                        -- 'csob' | 'raiffeisen' | 'trinity_legal'
  escrow_ref        text,
  notary_name       text,
  notary_date       date,
  cadastre_ref      text,
  signed_at         timestamptz,
  funds_released_at timestamptz,
  completed_at      timestamptz,
  created_at        timestamptz DEFAULT now()
);

-- Audit log (všechny side-effecting akce)
CREATE TABLE audit_log (
  id           bigserial PRIMARY KEY,
  actor_id     uuid REFERENCES auth.users(id),
  actor_role   text,                             -- seller | investor | referrer | admin | system
  action       text NOT NULL,                    -- 'lead.created' | 'bid.submitted' | ...
  resource     text,                             -- 'lead' | 'bid' | 'transaction'
  resource_id  uuid,
  payload      jsonb,                            -- diff nebo plný snapshot
  ip_address   inet,
  created_at   timestamptz DEFAULT now()
);

-- Sofia konverzace (per lead)
CREATE TABLE sofia_conversations (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id     uuid REFERENCES leads(id),
  messages    jsonb DEFAULT '[]'::jsonb,         -- [{role, content, timestamp}]
  updated_at  timestamptz DEFAULT now()
);

-- Notifikace queue
CREATE TABLE notifications (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  recipient_id uuid REFERENCES auth.users(id),
  channel      text CHECK (channel IN ('email','sms','push')),
  template     text NOT NULL,
  payload      jsonb,
  status       text DEFAULT 'pending'
               CHECK (status IN ('pending','sent','failed','delivered','opened')),
  sent_at      timestamptz,
  error        text,
  created_at   timestamptz DEFAULT now()
);
```

### Row Level Security policies (kritické!)

```sql
-- Lead: vlastník vidí všechno, investor jen public fields
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;

CREATE POLICY "seller_own_leads" ON leads
  FOR ALL USING (
    seller_id IN (SELECT id FROM sellers WHERE user_id = auth.uid())
  );

CREATE POLICY "investor_public_view" ON leads
  FOR SELECT USING (
    status IN ('open_for_bids','has_bids')
    AND EXISTS (SELECT 1 FROM investors WHERE user_id = auth.uid() AND kyc_status = 'verified')
  );

-- Investor nikdy nevidí address_full ani seller PII
-- Řešíme přes view:
CREATE VIEW investor_lead_view AS
  SELECT id, property_type, address_public, district, city, m2, spec,
         qualitative, status, created_at
  FROM leads
  WHERE status IN ('open_for_bids','has_bids');

-- Bids: investor vidí jen své nabídky, prodávající všechny nabídky na svůj lead
ALTER TABLE bids ENABLE ROW LEVEL SECURITY;

CREATE POLICY "investor_own_bids" ON bids
  FOR ALL USING (
    investor_id IN (SELECT id FROM investors WHERE user_id = auth.uid())
  );

CREATE POLICY "seller_views_bids_on_own_lead" ON bids
  FOR SELECT USING (
    lead_id IN (
      SELECT id FROM leads
      WHERE seller_id IN (SELECT id FROM sellers WHERE user_id = auth.uid())
    )
  );

-- Investor nikdy nevidí ostatní investory ani jejich nabídky — anti-aukce princip
```

---

## 4 · Třetí strany

| Služba | Účel | Failure mode |
|---|---|---|
| **Anthropic Claude API** | Sofia AI průvodce | Fallback na statické FAQ + lidský operátor (e-mail) |
| **Twilio Verify + SMS** | OTP, callback notifications | Retry queue, fallback e-mail |
| **Resend** | Transakční e-maily | SES backup |
| **Stripe** | KYC platby investorů, deposit za bid commitment | Manuál bankovním převodem |
| **Google Maps API** | Adresa autocomplete, mapa | Statická textová pole + manuál |
| **ČSOB API** (Phase 2) | Bankovní úschova webhook | Manuální status updates |
| **PostHog** | Product analytics, funnel | Self-hosted alternativa |
| **Sentry** | Error monitoring | Mandatory |
| **Vercel** | Hosting, Edge functions | Failover na Cloudflare Pages |

---

## 5 · Modulární rozdělení

### Doménové moduly (clean architecture)

```
modules/
├── lead/                # Lead creation, validation, lifecycle
│   ├── domain/         # types, validators (Zod), business rules
│   ├── application/    # use cases (createLead, updateSpec, ...)
│   ├── infrastructure/ # Supabase repository
│   └── presentation/   # React komponenty
├── bid/
├── transaction/
├── kyc/
├── notification/
├── sofia/              # AI orchestrátor
└── analytics/
```

**Pravidlo závislostí:** `presentation → application → domain ← infrastructure`. Domain nikdy nezávisí na infrastrukturně (Supabase, Anthropic). Test-friendly.

### Shared kernel
- `lib/result.ts` — Result<T, E> pattern pro error handling.
- `lib/types/branded.ts` — `LeadId`, `BidId`, `PriceHaléř` (žádné holé stringy/numbers).
- `lib/feature-flags.ts` — Statsig nebo vlastní `flags.json`.

---

## 6 · Deployment

```
┌──────────────────────────────────────────────────┐
│  Vercel (production)                              │
│  ├── kuply.cz             (production)            │
│  ├── staging.kuply.cz     (per-PR previews)       │
│  └── dev.kuply.cz         (auto from main)        │
└──────────┬───────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────┐
│  Supabase (eu-central-1)                          │
│  ├── kuply-prod                                   │
│  └── kuply-staging                                │
└──────────────────────────────────────────────────┘
```

**CI/CD:** GitHub Actions
- PR → lint + typecheck + unit + e2e (preview env) + Lighthouse
- Merge to `main` → auto deploy staging → smoke tests → manual approval → prod

**Migration strategy:** Supabase migrations v `supabase/migrations/`, jména `YYYYMMDDHHMMSS_description.sql`. CI ověří proti staging DB před produkcí.

---

## 7 · Lokalizace & a11y

- **Czech-first**: všechny error messages, validace, AI prompty.
- **i18n připravenost**: keys v `messages/cs.json`, struktura ready pro SK/PL/HU.
- **WCAG AA**: kontrast min. 4.5:1 (token systém z prototypu už splňuje), klávesnice navigace, ARIA pro Sofia chat.
- **Reduced motion**: respektovat `prefers-reduced-motion` (prototyp už dělá — všechny animace mají fallback).
