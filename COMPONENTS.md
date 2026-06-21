# Kuply — Component Catalog

> Kompletní katalog UI komponent. Sloupec **Status** ukazuje stav v prototypu (`✅ hotovo` / `🔄 přepsat z vanilla` / `⏳ nová`).
> Každá komponenta má: účel, props, vztahy, design notes z prototypu.

---

## 1 · Landing komponenty

### `<HeroCarousel3D />`  · ✅ hotovo
**Účel:** Tři stupňující se CTA karty v 3D karuselu na úvodu landingu.

**Props:**
```ts
type HeroCarouselProps = {
  defaultIndex?: 0 | 1 | 2;   // default 1 = Privátní rozbor
  onSelect: (path: 'quick' | 'private' | 'konzultace') => void;
};
```

**Karty:**
1. `Tržní odhad` (light, eyebrow `ZDARMA · 30 SEKUND`)
2. `Privátní rozbor` (dark, doporučeno, badge `★ DOPORUČUJEME`)
3. `Rychlá konzultace` (light s inline form, `ZAVOLÁME DO 15 MINUT`)

**Design notes:** Spacing 0.95 desktop / 0.72 mobile, scale side 0.87, tz −140, blur cap 1.6, veil opacity 0.5, side opacity ~0.78. Pilulky pod karuselem (`01` / `02` / `03`) pro snap. Engine vystavuje `__car.{pos, target, go(i), spacing}`.

**Vztahy:** vstupuje do `<FlowSection />` přes `onSelect`.

---

### `<ThesisTiltCards />`  · ✅ hotovo
**Účel:** Sekce „Každý kupec vidí jinou cenu" — tři karty s 3D tilt efektem.

**Props:** žádné (data jsou součást komponenty).

**3 karty:** Veřejný odhad (±15%), Konzervativní odhad (banky), Strategický kupec (±3%, tmavá karta s SofiOrb).

**Interakce:** mouse-tilt rotateX ±9° / rotateY ±11° + translateZ 10, glare radial sleduje kurzor, IO reveal stagger 0.12s, žádný `(hover:hover)` gate.

---

### `<ProcessCarousel />`  · ✅ hotovo (jako druhý Carousel3D)
**Účel:** Sekce „Po rozboru? Vy řídíte tempo." — 6 tmavých karet s kroky 01-06 + Sofiina citace na každé.

**6 kroků:**
1. Sofiin rozbor — „Žádný spěch."
2. PDF rozbor a hovor — „Vaše otázky jsou na prvním místě."
3. Otevření investorům — „Nejvyšší cena nemusí být nejlepší."
4. Slepé nabídky — „Když si nejste jistí, projdeme to spolu."
5. Notář a podpis — „Smlouvu si přečtu první."
6. Peníze v úschově — „Hlídám každý krok za vás."

**Engine:** stejný Car3D jako Hero, druhá instance s vlastním `data-car3d` root, `data-start="0"`.

---

### `<FomoBlock />`  · ✅ hotovo
**Účel:** 3 anonymizované aktuální příležitosti s rozmazanými cenami → CTA „Získat přístup k privátním dealům".

**Data:** Praha 2 / Vinohrady · Byt 3+kk 84m² · ceny blur(5px). Brno-střed · Činžovní dům 12 jednotek. Praha 9 / Vysočany · Pozemek 3 050 m².

**CTA:** scroll na top karusel (re-engagement, ne registrace).

---

### `<StoriesStack />`  · ✅ hotovo
**Účel:** Sticky card-stack (adaptace `CardsStack` z user promptu). 6 příběhů, každý zacvakne přes předchozí.

**Mechanika:** `position: sticky; top: 96 + i*34; z-index: i+1`. Mono label `ZÁZNAM 0i / 06`.

---

### `<TrustGrid />`  · ✅ hotovo
**Účel:** „KLIDNĚ SPĚTE — Za každou transakcí stojí skuteční odborníci." Trojice karet:

1. **Advokátní kancelář** ⚖️ — Trinity Legal s.r.o.
2. **Bankovní úschova** 🏦 — ČSOB · Raiffeisenbank · Moneta
3. **Notářské ověření** 🔏 — Trinity 3,99 % vč. DPH

**Důležité:** Tato sekce je pro důvěru kritická. Nikdy ji nevynechávat ani v zúžených verzích.

---

### `<FaqList />`  · ✅ hotovo
**Účel:** 9 otázek/odpovědí s expandable items, zlatým indexem 01-09, plus→× ikonou (rotace 135° + zlatá výplň).

**Props:** `items: { q, a }[]`.

**Behavior:** single-open (otevření zavře předchozí), `aria-expanded` na buttonu, max-height transition.

---

### `<CinematicFooter />`  · ✅ hotovo
**Účel:** Curtain-reveal footer s aurora, mřížkou, obrovským outline `KUPLY` (parallax), diagonálním marquee a magnetickými pilulkami.

**Mechanika:** Footer je portálem mimo `#root`, fixed s `z-index:1`. `#root` má `z-index:2 + bg:paper`. Spacer mezi nimi je portován na `body.appendChild`. Z prototypu: lekce že React `onClick` v portovaném footeru NEFUNGUJE — vše vanilla listenery.

**Marquee:** `ADVOKÁTNÍ ÚSCHOVA · TRINITY 3,99 % · NABÍDKY DO 48 HODIN · BEZ VEŘEJNÉ INZERCE · PRIVÁTNÍ PRODEJ · SLEPÉ NABÍDKY` (loop).

---

### `<NoiseLayer />`  · ✅ hotovo
**Účel:** Filmový grain JEN na hero pozadí (po reklamaci Romana že byl všude).

**CSS:** `.flow-section:not(.stage-on)::after` s `background-image: var(--noise)`, mask gradient do papíru v posledních 30 % výšky.

**Pozor:** žádný globální `#grainFx` fixed div. Globální grain explicitně zakázán.

---

## 2 · Flow (privátní rozbor) komponenty

### `<ConsoleShell />`  · ✅ hotovo
**Účel:** Framing kontejner pro celý flow (gold corners, auto-scroll do středu při step change).

**Props:** `children`, `step`, `totalSteps`, `onBack`.

**Auto-center:** `consoleRef.current.scrollIntoView({ behavior: 'smooth', block: 'center' })` na každý step change + při path change (kritické UX, validováno v prototypu).

---

### `<StepProgress />`  · ✅ hotovo
**Účel:** Progress bar `1/6 · TYP → ADRESA → SPECIFIKACE → DETAILY → CÍL → VÝSLEDEK`.

---

### `<TypePicker />`  · ✅ hotovo
**Účel:** Volba typu nemovitosti — 6 chipů (Byt / Rodinný dům / Pozemek / Činžovní dům / Komerční / Developerský).

**Props:** `value`, `onChange(type)`.

**Side effect:** uloží do leadu, otevře `SpecForm` s odpovídajícím schématem.

---

### `<AddressAutocomplete />`  · ✅ hotovo
**Účel:** Adresa s návrhy z Google Places (v prototypu mock data).

**Props:** `value`, `onChange(suggestion: { addr, base, m2 })`.

**Edge case:** suggestions jsou `<button>` s textem bez čárky („Vinohradská 12Praha 2") — důležité pro e2e testy.

---

### `<SpecForm />`  · ✅ hotovo (kritická!)
**Účel:** **Typově specifické pole** dle `property_type`. Klíčová komponenta pro „100 % zdravý lead".

**Schémata (`SPEC_FIELDS`):**

| Typ | Povinná pole | Volitelná |
|---|---|---|
| **byt** | dispozice (1+kk…), vlastnictví (osobní/družstevní) | č. jednotky, podlaží, příslušenství (multi) |
| **dum** | typ stavby (samostatný/řadový), pozemek m² | podlažnost, příslušenství |
| **pozemek** | LV (číslo), katastrální území, druh (stavební/zahrada/orná…) | sítě (multi: el/voda/kanal/plyn/příjezd) |
| **cinzak** | jednotky | obsazenost %, LV |
| **komercni** | účel (kanceláře/obchod/sklad/výroba/smíšené), pronajato (Ano/Částečně/Ne) | — |
| **developer** | fáze (záměr/UR/SP/v realizaci) | jednotky |

**Pole types:** `text` (s `inputMode='numeric'` kde dává smysl), `chips` (single-select), `multi` (multi-select).

**Live UX:**
- `<CompletenessMeter pct={...} />` v rohu, gold gradient bar.
- Badge `✓ ZDRAVÝ LEAD — PŘIPRAVEN PRO INVESTORY` se zelenou barvou jakmile všechna povinná pole vyplněna.
- CTA tlačítko `Pokračovat k detailům` je disabled (opacity 0.45) dokud `reqOk === false`.

---

### `<CompletenessMeter />`  · ✅ hotovo
**Účel:** % kompletnosti leadu (počet vyplněných polí / celkem * 100). Vizuálně: velké zlaté `XX %`, pod tím gold lišta, pod tím mono `KOMPLETNOST LEADU`.

**Animace:** `transition: width .45s var(--ease)`.

---

### `<QualitativeQuestions />`  · ⏳ nová
**Účel:** Krok 3 v Privátním rozboru — kvalitativní otázky (stav, parkování, výtah, výhled, sousedi…).

**Behavior:** 5-7 otázek, single-select chips, progress mini-dots „3/7".

---

### `<MotivationStep />`  · ✅ hotovo (částečně)
**Účel:** Volba motivace (Stěhování / Rozvod / Dědictví / Finanční tíseň / Investice / Jiné).

**Side effect:** motivace ovlivňuje, jaký kontakt dostane od Sofie a jaké podklady se připraví.

---

### `<ContactSubmit />`  · ✅ hotovo
**Účel:** Finální krok — jméno, telefon, e-mail, preferovaný způsob kontaktu, checkbox GDPR.

**Validace:** Zod `LeadContactSchema`, e-mail formát, CZ telefon (+420), GDPR vyžadováno.

**Po odeslání:** přesměrování na `<LeadReceived />` s thank-you + co následuje.

---

## 3 · Sofia AI komponenty

### `<SofiaOwl />`  · ✅ hotovo
**Účel:** SVG sova v gold orbu — sdílený mascot (landing + dashboardy + chat).

**Props:**
```ts
type SofiaOwlProps = {
  size?: number;              // default 64
  state?: 'idle' | 'thinking' | 'alert';
  ring?: boolean;             // doubling gold ring
};
```

**Z prototypu lekce:** SVG defs musí být v `<body>` (hoist do `document.body.insertBefore`), jinak `<use href="#owl-hero">` nerenderuje v některých remountech.

---

### `<SofiaFlight />`  · ✅ hotovo
**Účel:** Floating Sofia entity, která doprovází uživatele po stránce.

**Mechanika:**
- Pozice: u pravého okraje, sleduje scroll s time-based lerp inertia.
- Velikost: 64px desktop / 46px mobile.
- Tilt podle scroll velocity, bobbing animace.
- 3 trail particles.
- Contextual bubbles přes IO na sekce (`#root section h2` text-match).
- Dock animation u otevřené `console-wrap` → state 'thinking'.
- V dashboardech: skryta (`display: none`).

---

### `<SofiaBubble />`  · ✅ hotovo
**Účel:** Kontextová promluva Sofie u sekce.

**Default bubliny dle sekce:**
- Hero: „Dobrý den, jsem Sofia. Provedu vás." (1.6s po loadu)
- Tři pohledy: „Tři pohledy na cenu. Tři typy kupců."
- Procesní karusel: „Po rozboru? Vy řídíte tempo."
- FAQ: „Mám odpovědi na to, co vás trápí."

---

### `<SofiaChat />`  · ⏳ nová
**Účel:** Plný chat panel (drawer) pro AI rozhovor s Sofií v kontextu otevřeného leadu.

**Stack:**
- Streaming z `/api/sofia` (Vercel AI SDK + Anthropic).
- Kontext: lead spec, fáze flow, motivace, předchozí zprávy.
- Tools: `lookup_district_prices`, `check_address_validity`, `escalate_to_human`.

**UX:** otevírá se klikem na `<SofiaFlight />` nebo z `?sofia=open` query. Persistuje history v `sofia_conversations` table.

---

## 4 · Investor dashboard komponenty

### `<LeadCard />`  · ⏳ nová
**Účel:** Karta v seznamu leadů pro investora.

**Co vidí:** lokalita (obecná — Praha 2 · Vinohrady, nikdy ne přesnou adresu), typ, m², stručná spec, „otevřeno X dní", počet existujících nabídek (anonymně, ne kdo).

**Co NEVIDÍ:** přesná adresa, jméno prodávajícího, telefon, ceny ostatních nabídek.

---

### `<LeadFilters />`  · ⏳ nová
**Účel:** Filtry seznamu — lokalita (multi), typ, m² range, status (open / has bids).

---

### `<BidForm />`  · ⏳ nová
**Účel:** Formulář slepé nabídky.

**Pole:** cena (CZK), doba do podpisu (dní), datum nástupu, podmínky (textarea), commitment deposit (Stripe).

**Behavior:** po odeslání bid je `submitted`, expirace +7 dní. Lead přejde do `has_bids`. Sofiina notifikace prodávajícímu.

---

### `<DealTimeline />`  · ⏳ nová (klíčová pro Phase 2)
**Účel:** 10-step transaction timeline (po `offer_accepted`):

```
01 · Offer Submitted          ✓ <hodina>
02 · Seller Review            ✓ <hodiny>
03 · Offer Accepted           ✓ <minuty>  ← demo trigger
04 · Legal Verification       ⚡ Kuply-fast  <hodiny>
05 · Due Diligence            ⚡           <dny>
06 · Contract Preparation     ⚡           <dny>
07 · Digital Signing          ⚡           <hodiny>
08 · Escrow Funding           <hodiny>
09 · Ownership Transfer       ~20 dní (Katastr)
10 · Investment Active        ✓
```

**% progres:** vážené dle `etaHours` jednotlivých kroků. ⚡ badge na Kuply-fast krocích.

**Demo simulation (Phase 1):** po `submitBid` ~8s seller accepts → toast + step 03 + auto-advance 04-06 každých 6s.

---

## 5 · Shared komponenty

### `<GoldButton />` (CTA gold)  · ✅ hotovo
Zlaté primární tlačítko (gradient `--gold-l → --gold`), rounded full, arrow icon.

### `<ChipPicker />`  · ✅ hotovo
Pilulkové chipy single/multi-select s gold ring on hover, cocoa active state.

### `<Card />` (z shadcn)  · 🔄
Přepsat stávající `.lc-card`, `.entry-card`, `.fomo-card` na variant systém shadcn.

### `<Dialog />`, `<Drawer />`, `<Toast />`  · ⏳
shadcn primitivy pro Sofia chat, KYC modal, error toasts.

---

## 6 · Komponenty hierarchie

```
<App>
├── <PublicLayout>
│   ├── <Nav />                            (sticky top, z-50)
│   ├── <HeroSection>
│   │   ├── <NoiseLayer />                 (jen tady, ne globálně)
│   │   ├── <HeroIntro />                  (titulek + Sofia bubble)
│   │   ├── <HeroCarousel3D />
│   │   └── <FlowSection />                (vystředí console při path != null)
│   │       ├── <ConsoleShell>
│   │       │   ├── <StepProgress />
│   │       │   ├── <TypePicker />         (step 0)
│   │       │   ├── <AddressAutocomplete />(step 1)
│   │       │   ├── <SpecForm>             (step 2, NEW)
│   │       │   │   ├── <CompletenessMeter />
│   │       │   │   └── <ChipPicker /> × N
│   │       │   ├── <QualitativeQuestions />(step 3)
│   │       │   ├── <MotivationStep />     (step 4)
│   │       │   └── <ContactSubmit />      (step 5)
│   ├── <ThesisTiltCards />
│   ├── <ProcessCarousel />
│   ├── <FomoBlock />
│   ├── <StoriesStack />
│   ├── <TrustGrid />
│   ├── <FaqList />
│   ├── <CinematicFooter />
│   └── <SofiaFlight />                    (fixed, doprovází scroll)
│       └── <SofiaBubble />
│
├── <InvestorLayout>                        (auth required, KYC gate)
│   ├── <DashNav />
│   ├── <LeadFilters />
│   ├── <LeadCard /> × N
│   └── <Dialog>
│       └── <BidForm />
│
└── <DealTimelinePage>
    └── <DealTimeline />
```

---

## 7 · Design tokens (z prototypu, plus Tailwind mapping)

Kompletní set v `design-system/tokens.css`:

```css
:root {
  --paper:        #FBF6EC;
  --ivory:        #FFFDF7;
  --ink:          #2A211A;
  --ink-90:       #3D362E;
  --ink-75:       #4E4439;
  --ink-60:       #6B5E50;
  --ink-45:       #897A68;
  --cocoa:        #2F261E;
  --cocoa-deep:   #1F1813;
  --gold:         #C9A876;
  --gold-l:       #E0C997;
  --gold-d:       #B5965F;
  --rust:         #B5663A;
  --rule:         rgba(61,54,46,.10);
  --positive:     #5E7C4F;

  --fs-hero:      clamp(38px,6vw,72px);
  --fs-h3:        clamp(28px,4vw,42px);
  --fs-card:      clamp(20px,2.4vw,26px);
  --fs-body:      clamp(14.5px,1.6vw,17px);

  --ease:         cubic-bezier(.2,.8,.2,1);
  --ease-in:      cubic-bezier(.16,1,.3,1);
  --dur-2:        .25s;

  --noise:        url('/noise.png');
}
```

**Tailwind mapping v `tailwind.config.ts`:**
```ts
extend: {
  colors: {
    paper: 'var(--paper)',
    ivory: 'var(--ivory)',
    ink: { DEFAULT: 'var(--ink)', 90: 'var(--ink-90)', 75: 'var(--ink-75)', /*...*/ },
    cocoa: 'var(--cocoa)',
    gold: { DEFAULT: 'var(--gold)', light: 'var(--gold-l)', dark: 'var(--gold-d)' },
    rust: 'var(--rust)',
  },
  fontSize: {
    hero: 'var(--fs-hero)',
    'h3-fluid': 'var(--fs-h3)',
  },
}
```
