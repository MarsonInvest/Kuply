# Kuply — Project Specification

> **Status:** Pre-production · Prototype validated (single-file HTML build v10, QA 10/10) · Připraveno pro produkční implementaci v Next.js + Supabase stacku.
> **Document version:** 1.0 (2026-06-14)
> **Maintainer:** Roman Tulis, Marson Invest s.r.o.

---

## 1 · Cíl projektu

**Kuply** je **privátní AI-řízený realitní operating system** pro český trh, který spojuje prodávající nemovitostí s prověřenými investory ve formátu **slepých nabídek bez veřejné inzerce**. Není to portál typu Sreality. Je to **uzavřený transakční ekosystém**, kde:

- Prodávající dostává **tři pohledy na cenu** (veřejný odhad / bankovní odhad / privátní cena investora) a **AI průvodce Sofii**, která ho vede celým procesem v češtině.
- Investor dostává **strukturované 100% zdravé leady** s ověřenými daty (LV, dispozice, vlastnictví, kompletnost) a nástroje pro slepé nabídky bez vidění nabídek konkurence.
- Tipař (broker, advokát, soused, kdokoliv) dostává **provizi za přivedený dotek**.
- Marson Invest jako provozovatel dostává **marži z transakce** + data o trhu, která veřejné portály nemají.

**Klíčový rozdíl proti existujícím řešením:**

| Stávající trh | Kuply |
|---|---|
| Veřejná inzerce, prohlídky cizími lidmi, tlak | Diskrétní privátní cesta, bez veřejné prezentace |
| Anonymní portálový průměr | Konkrétní cena konkrétního investora pro vaši nemovitost |
| Makléř bere 3-5 % od prodávajícího | Prodávající platí 0 Kč, marži platí investor |
| Manuální papírování, neznámý notář | Trinity Legal, ČSOB úschova, notář ve vašem okolí, vše v ceně |

---

## 2 · Cílový uživatel

### Persona A — Prodávající („Tichý prodej")

**Kdo:** Majitelé nemovitostí 35-65 let, kteří potřebují prodat ale nechtějí inzerát.

**Klíčové motivace (z prototypu, validováno na 6 příbězích):**

| Motivace | Důvod, proč nejde běžnou cestou |
|---|---|
| Hrozí dražba/exekuce | Časový tlak, stigma veřejné dražby |
| Rozvod | Diskrétnost, sousedi to nesmí vědět |
| Zděděný byt po rodičích | Emoční zátěž, neorientuje se v právu |
| Stěhování za prací do ciziny | Vzdálenost, potřebuje to vyřešit z dálky |
| Privátní financování (potřebují hotovost rychle) | Nemůžou čekat 6 měsíců na běžný prodej |
| Komplikovaná nemovitost (insolvence, podíl, hypotéka) | Běžní kupci to nezvládnou |

**Vstupní bod:** Tři karty na hero — `Tržní odhad` (zdarma, anonymní), `Privátní rozbor` (doporučeno, 2 minuty), `Rychlá konzultace` (volání do 15 min).

**Žádaná akce:** Vyplnit `SpecForm` až do stavu „**✓ ZDRAVÝ LEAD — PŘIPRAVEN PRO INVESTORY**" (100% kompletnost povinných polí pro daný typ nemovitosti).

### Persona B — Investor („Prověřený kupec")

**Kdo:** Realitní investoři, developeři, fondy s vlastní akviziční strategií. Cca 1 200+ v síti k cílovému stavu.

**Klíčové potřeby:**
- Vidět **pouze zdravé leady** s ověřenými parametry (žádné „byt nějaký v Praze").
- Mít exclusivitu — leady **nejsou** na veřejných portálech.
- Slepé nabídky — neví, co nabízí konkurence, soutěží na vlastní cenu/podmínky.
- KYC ověření, advokátní úschova, jasná pravidla.

**Vstupní bod:** `Investor` dashboard (oddělená sub-aplikace v `/investor`).

### Persona C — Tipař (broker, partner)

**Kdo:** Lidé s přístupem k off-market případům (advokáti, exekutoři, sousedi). 

**Hodnota:** Provize za přivedení leadu, který skončí transakcí. Sub-aplikace v `/tipar`.

### Persona D — Operátor (Marson Invest, interní)

**Kdo:** Tým Marson Invest, který spravuje leady, KYC, prověřuje nabídky.

**Sub-aplikace:** `/admin` (out of MVP — Phase 2).

---

## 3 · MVP scope (Phase 1 — 12 týdnů)

### Veřejný landing (`/`)
- ✅ Hero s 3D karuselem tří CTA (Tržní odhad / Privátní rozbor / Rychlá konzultace) — **hotovo v prototypu**.
- ✅ Sekce „Tři pohledy" s 3D tilt kartami — **hotovo**.
- ✅ Procesní karusel 6 kroků (Sofiin rozbor → Peníze v úschově) — **hotovo**.
- ✅ FOMO blok s blurovanými cenami — **hotovo**.
- ✅ Příběhy (sticky card-stack) — **hotovo**.
- ✅ Trust sekce (Trinity Legal, ČSOB úschova, notář) — **hotovo**.
- ✅ FAQ s expandable items — **hotovo**.
- ✅ Cinematic footer — **hotovo**.

### Flow prodávajícího
- **Tržní odhad (Path A — Quick):** Typ → Lokalita → m² → Cenové rozpětí (median z portálových dat).
- **Privátní rozbor (Path B — Lead-generation):** Typ → Adresa (geo-suggestion) → m² → **Specifikace dle typu** (povinná pole: dispozice/LV/účel/...) → Kvalitativní otázky → Motivace → Kontakt → Odeslání leadu.
- **Rychlá konzultace (Path C):** Jméno + telefon → callback do 15 minut.

### Investor dashboard (`/investor`)
- KYC onboarding (jednorázové ověření).
- Seznam aktivních leadů s filtrem (lokalita, typ, m², cenové rozpětí).
- Detail leadu (vše kromě adresy a kontaktu — odemkne se až po přijetí nabídky).
- Submit slepé nabídky (cena, podmínky, datum nástupu).
- Status timeline transakce (10 kroků, viz `COMPONENTS.md`).

### Tipař dashboard (`/tipar`)
- Submit lead formulář (anonymně, jen kontakt na majitele).
- Status leadu + provize.

### Backend
- Lead intake API s validací schémat dle typu nemovitosti.
- Notifikační systém (e-mail, SMS — Twilio + Resend).
- Encrypted storage (Supabase RLS, AES-256 pro PII).
- Audit log všech akcí.

---

## 4 · Out of scope (pro MVP)

| Položka | Důvod | Plánováno na |
|---|---|---|
| Vlastní nativní mobile app | Web-first; PWA stačí | Phase 3 |
| Aukční mechanika („nejvyšší vyhrává") | Vědomě se vyhýbáme — slepé nabídky | Nikdy |
| Veřejný katalog nemovitostí | Proti pricipu privátnosti | Nikdy |
| Integrace s katastrem nemovitostí (live API) | ČÚZK API je placené a omezené | Phase 2 |
| Hypoteční kalkulačka | Není to fintech, je to marketplace | Phase 3 |
| Slovensko / další trhy | Czech-only, právní specifikace | Phase 4+ |
| AI generování právních dokumentů | Trinity Legal to dělá manuálně | Phase 3 |
| Veřejná Sofia jako chatbot bez kontextu transakce | Sofia je vždy v kontextu konkrétního leadu | — |
| Sekundární trh (resale) | Není v doméně Kuply | Nikdy |

---

## 5 · Success metriky

### North Star
**Počet uzavřených transakcí měsíčně** (signed → katastr zapsán → peníze vyplaceny).

### Phase 1 (3 měsíce po launch)
- 200+ leadů submitted via privátní rozbor
- 60% leadů ve stavu „zdravý lead" (kompletnost ≥ 80%)
- 50+ aktivních investorů v síti (KYC dokončeno)
- 8+ uzavřených transakcí
- Median lead → 1. nabídka < 36 hodin
- NPS prodávajícího ≥ 50

### Phase 2 (6 měsíců)
- 500+ leadů/měsíc
- 300+ investorů
- 25+ transakcí/měsíc
- GMV (gross merchandise value) 250M+ Kč/měsíc
- Marže Kuply 1,5-3 % z GMV

### Anti-metriky (sledujeme, abychom je nepřekročili)
- Drop-off uprostřed privátního rozboru > 35 % → UX problém v SpecForm.
- Lead → bez nabídky do 7 dní > 20 % → málo investorů nebo špatný matching.
- Stížnost na únik dat = okamžitý audit, není akceptovatelná hodnota.

---

## 6 · Klíčová obchodní pravidla (vstup pro implementaci)

1. **Tři stupňující se ceny** — veřejný < bankovní < privátní. Pořadí je vždy stejné, narativ na tom celý stojí.
2. **Přesný výsledek až po kontaktu** — anonymní odhad dává jen rozpětí ± 15 %. Konkrétní cenu vidí prodávající až po dokončení privátního rozboru.
3. **Advokátní/bankovní úschova povinně** — žádné peníze nikdy nechodí přes účet Kuply. Trinity 3,99 % vč. DPH za úschovu.
4. **Notář ve vašem okolí** — nikdy korespondenčně, nikdy „přijeďte k nám".
5. **Pro prodávajícího 0 Kč** — marži platí investor.
6. **Sofia nikdy nelže o ceně** — pokud je trh slabý, řekne to. Důvěra > krátkodobá konverze.
7. **Žádná aukce** — investoři nikdy nevidí konkurenční nabídky.
8. **GDPR-first** — adresa a jméno odemčeno až po přijetí nabídky prodávajícím.

---

## 7 · Vize roadmap (post-MVP)

- **Phase 2:** Admin panel pro Marson Invest, ČÚZK integration, lead scoring AI, escrow automation.
- **Phase 3:** Privátní financování (bridge loans pro prodávající v tísni), private deals marketplace pro investory.
- **Phase 4:** Slovensko, Polsko, Maďarsko (CZ legal stack je nepřenosný — každý trh vlastní implementace).
- **Phase 5:** B2B nástroj pro banky (foreclosure prevention) — banka pošle Kuply své rizikové úvěry, ušetří hluboké odpisy.
