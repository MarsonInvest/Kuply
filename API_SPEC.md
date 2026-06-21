# Kuply — API Specification

> Veřejné API kontrakty mezi klientem a serverem. Server Actions, Route Handlers, webhooks.

---

## 1 · Konvence

### Formáty
- **Request/Response:** JSON
- **Datum/čas:** ISO 8601 (`2026-06-14T10:30:00.000Z`)
- **Peněžní částky:** vždy v **haléřích** (bigint v DB, number v JSON) — nikdy `Float`!
- **ID:** UUID v4
- **Locale:** `cs-CZ` výchozí, error messages česky

### Error response (univerzální)
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dispozice je povinné pole pro byt.",
    "field": "spec.disp",
    "details": { /* Zod error chain */ }
  }
}
```

### Autentizace
- **Public endpoints:** žádná
- **Auth endpoints:** Supabase Session cookie (HTTP-only, secure)
- **Service-to-service (webhooks):** HMAC signature verify (`X-Kuply-Signature` header)

### Rate limiting
- Public: 60 req/min per IP
- Auth: 200 req/min per user
- Webhooks: 1000 req/min per integration

---

## 2 · Server Actions (nejčastější mutations)

### `createLead`
**Použití:** finální krok privátního rozboru, vytvoří nový lead.

```ts
'use server';
export async function createLead(input: CreateLeadInput): Promise<Result<LeadId, AppError>>;

type CreateLeadInput =
  | { property_type: 'byt';      spec: BytSpec;      common: CommonFields }
  | { property_type: 'dum';      spec: DumSpec;      common: CommonFields }
  | { property_type: 'pozemek';  spec: PozemekSpec;  common: CommonFields }
  | { property_type: 'cinzak';   spec: CinzakSpec;   common: CommonFields }
  | { property_type: 'komercni'; spec: KomercniSpec; common: CommonFields }
  | { property_type: 'developer';spec: DeveloperSpec; common: CommonFields };

type CommonFields = {
  address_full:  string;        // PII, encrypted at rest
  address_public: string;       // 'Praha 2 · Vinohrady'
  district:      string;
  city:          string;
  m2:            number;        // > 0
  qualitative:   QualitativeAnswers;
  motivation:    Motivation;
  contact: {
    name:        string;        // min 2
    email:       string;        // email format
    phone:       string;        // +420...
    gdpr_consent: true;         // literal true
  };
};

// Side effects:
//   - INSERT sellers (upsert by email)
//   - INSERT leads (status='submitted', completeness_pct computed)
//   - INSERT audit_log
//   - Resend.send('lead.received', seller.email)
//   - Resend.send('lead.new_open', 'ops@kuply.cz')
//   - Twilio.send OTP to phone
//   - INSERT sofia_conversations (empty)
//   - revalidatePath('/flow/private/success')

// Returns:
//   Ok({ leadId: 'uuid' })
//   Err({ code: 'VALIDATION_ERROR' | 'DUPLICATE_LEAD' | 'OTP_FAILED' | 'DB_ERROR' })
```

### `verifyPhoneOTP`
```ts
'use server';
export async function verifyPhoneOTP(leadId: LeadId, code: string): Promise<Result<void, AppError>>;
// Side effect: pokud OK, lead.status='validated', otherwise increment fail counter
```

### `updateLeadSpec`
**Použití:** Phase 2 — Marson ops doplňuje data, která prodávající neposkytl.
```ts
'use server';
export async function updateLeadSpec(leadId: LeadId, patch: Partial<LeadSpec>): Promise<Result<void, AppError>>;
// Requires: admin role
// Side effect: audit_log, recompute completeness_pct
```

### `openLeadToBids`
```ts
'use server';
export async function openLeadToBids(leadId: LeadId): Promise<Result<void, AppError>>;
// Requires: admin role + lead.status === 'validated'
// Side effect: status → 'open_for_bids', notifications.queue eligible investors
```

### `submitBid`
```ts
'use server';
export async function submitBid(input: SubmitBidInput): Promise<Result<BidId, AppError>>;

type SubmitBidInput = {
  lead_id:        LeadId;
  price:          number;       // v haléřích, > 0
  conditions:     string;       // max 1000 chars
  move_in_date:   string;       // ISO date
  commitment_payment_intent_id: string;  // Stripe ID
};

// Requires: investor role + KYC verified
// Validates: Stripe.paymentIntent.status === 'succeeded'
// Side effects:
//   - INSERT bids (status='submitted', expires_at = now + 7d)
//   - UPDATE leads SET status='has_bids' (if first bid)
//   - notifications.queue(seller, 'bid.received')
//   - audit_log
```

### `withdrawBid`
```ts
'use server';
export async function withdrawBid(bidId: BidId, reason: string): Promise<Result<void, AppError>>;
// Requires: bid.investor_id === current investor + bid.status === 'submitted'
// Side effect: Stripe.refund commitment
```

### `acceptBid`
```ts
'use server';
export async function acceptBid(bidId: BidId): Promise<Result<TransactionId, AppError>>;

// Requires: lead.seller_id === current seller + bid.status === 'submitted'
// Side effects (TRANSACTION):
//   - UPDATE bids SET status='accepted'
//   - UPDATE bids SET status='rejected' WHERE lead_id=X AND id!=Y
//   - UPDATE leads SET status='offer_accepted'
//   - INSERT transactions (current_step=1)
//   - Stripe.refund() × all rejected bids
//   - notifications: vítěz + porazí
//   - Webhook → Trinity Legal intake API
//   - audit_log
```

### `sendSofiaMessage`
```ts
// Streaming SSE, ne Server Action
POST /api/sofia/{leadId}
Body: { content: string }
Response: text/event-stream

// chunks:
data: {"type":"text","content":"Dob"}
data: {"type":"text","content":"rý "}
data: {"type":"text","content":"den..."}
data: {"type":"tool_use","tool":"lookup_district_prices","input":{"district":"Vinohrady"}}
data: {"type":"tool_result","content":[...]}
data: {"type":"end"}
```

---

## 3 · Route Handlers

### `GET /api/geo?q={query}`
**Použití:** address autocomplete v `<AddressAutocomplete />`.

```
Request:  GET /api/geo?q=Vinohradsk%C3%A1%2012
Response: 200 OK
{
  "suggestions": [
    {
      "id": "place_abc123",
      "label": "Vinohradská 12, Praha 2",
      "city": "Praha 2",
      "district": "Vinohrady",
      "lat": 50.0763,
      "lng": 14.4435,
      "estimate_base": 155000,    // odhad ceny za m² na základě district data
      "estimate_m2": 84            // (jen pro mock dev data)
    }
  ]
}

Errors: 400 (q < 3 chars), 429 (rate limit), 503 (Google Maps down)
```

### `GET /api/sofia/{leadId}/history`
**Použití:** načte historii Sofia conversation pro daný lead.

```
Auth: seller of leadId OR admin
Response: 200 OK
{
  "messages": [
    { "role": "assistant", "content": "Dobrý den...", "ts": "2026-06-14T10:30:00Z" },
    { "role": "user",      "content": "Co když mám hypotéku?", "ts": "..." }
  ]
}
```

### `POST /api/sofia/{leadId}` (streaming)
Viz výše `sendSofiaMessage`.

### `POST /api/webhooks/twilio`
**Použití:** delivery status SMS.
```
Auth: HMAC signature verify
Body: { MessageSid, MessageStatus: 'delivered'|'failed', ... }
Side effect: UPDATE notifications WHERE provider_ref = MessageSid
```

### `POST /api/webhooks/resend`
**Použití:** delivery status e-mailu (delivered/opened/bounced).

### `POST /api/webhooks/stripe`
**Použití:** payment_intent.succeeded → enable bid submission; charge.refunded → audit log.

### `POST /api/webhooks/csob-escrow` (Phase 2)
**Použití:** stav úschovy (funded / released / rejected).

```
Auth: HMAC signature verify
Body: { escrow_ref, status: 'funded'|'released'|'rejected', amount, ts }
Side effect: UPDATE transactions SET current_step (8 nebo 9 podle status)
```

### `POST /api/webhooks/cuzk` (Phase 3)
**Použití:** katastr zapsal převod vlastnictví.

```
Body: { cadastre_ref, status: 'transferred', deed_pdf_url, ts }
Side effect: UPDATE transactions SET current_step=10, completed_at=now()
```

---

## 4 · GraphQL? Nein.

**Záměrně NEpoužíváme GraphQL.** App Router s RSC + Server Actions řeší většinu use-case bez nutnosti REST/GraphQL endpointů. Pokud někdy bude potřeba public API pro 3rd-party investice/partneři, zvážíme tRPC pro typovou bezpečnost.

---

## 5 · Public REST API (Phase 3+)

Plánováno pro:
- Banky (foreclosure prevention integration)
- Brokeři (lead submission)
- Realitky (partner network)

```
POST /api/v1/leads          (API key auth)
GET  /api/v1/leads/{id}     (API key auth, jen vlastní)
POST /api/v1/bids
```

Specifikace bude separátně v `docs/PUBLIC_API_v1.md` (zatím out of scope).

---

## 6 · Idempotency

**Všechny side-effecting Server Actions akceptují `idempotency_key`** (UUID generovaný klientem). Pokud dorazí dvakrát stejný klíč do 24 h, second call vrátí cached response. Tabulka `idempotency_keys` s TTL.

```ts
type CreateLeadInput = {
  // ...
  idempotency_key: string;   // UUID v4
};
```

---

## 7 · Verzování

- **Server Actions:** žádné explicitní verzování (interní API, breaking changes → coordinated deploy).
- **Webhooks:** každý endpoint má `X-Kuply-API-Version` header, default `2026-06-14`.
- **Public REST API (Phase 3+):** `/api/v1/...`, semver pro breaking changes.

---

## 8 · Příklad: kompletní seller flow přes API

```
1. POST createLead (Server Action)
   ← { leadId }
2. POST verifyPhoneOTP (Server Action)
   ← { ok: true }
3. GET /api/sofia/{leadId}/history
   ← messages: []
4. POST /api/sofia/{leadId} (streaming)
   ← stream: "Dobrý den, jsem Sofia..."
5. (admin) POST openLeadToBids → status: open_for_bids
6. (investor) POST submitBid → { bidId }
7. (seller) POST acceptBid → { transactionId }
8. Realtime: subscribed to transaction:{id}
   ← updates: current_step 1 → 2 → 3 → ...
9. Webhooks: csob-escrow funded → step 8
10. Webhooks: cuzk transferred → step 10, completed
```
