# KUPLY OS — PROJECT STATE SUMMARY
> Referenční dokument. Slouží k navázání práce kdykoliv (i po limitu modelu).
> Stav: **v9 — FAQ wow · 3D tilt pohledy · globální grain (QA čistá) · čeká se na review.**

---

## 0 · KONTEXT A AKTUÁLNÍ STAV KÓDU

**Produkt:** Kuply — AI-powered Real Estate Operating System. Prodávající zadá nemovitost, prověření investoři posílají slepé nabídky v časovém okně, prodávající vybere nejlepší. Není to web, je to ekosystém.

**Poslední dodaný build (v5, NE finální vize):**
- Jeden soubor `index.html` (~557 kB), bez CDN. React 18 UMD + předkompilovaný JSX (babel), staticky vygenerovaný Tailwind (9 kB), dashboard vrstva vanilla JS scopovaná pod `#dashWrap`.
- Funguje: lead flow 5 kroků s odhalením po kontaktu, stupňující ceny, most `window.kuplyLaunch()` → seller dashboard, živé nabídky, investor modal (galerie/parametry/lokalita/srovnání/finanční karta/slepá licitace), tipař, přepínač pozadí dashboardu, sova v orbu (bude nahrazena), 3D swap kroků (bude nahrazen plnohodnotným karuselem).
- Pracovní soubory: `/home/claude/klika/` — `index_tmpl.html` (dashboard vrstva), `build/build.py` (assembler s `must()` asserty), `photos.json` (base64 crops: PH0–PH3, DASH, STAGE, BGLIGHT), `/home/claude/owl_defs.svg`.

---

## 1 · FINÁLNÍ ROZHODNUTÍ (schváleno Romanem)

| # | Téma | Rozhodnutí |
|---|------|-----------|
| 0 | Vizuální směr | **Dual Theme.** Landing + prodejní zóna = Warm Futurism (slonovina, papír, cocoa, matné vrstvené sklo, zlato jako světlo/energie; luxus + bezpečí pro 45+). Investor dashboard = **Full dark pro-terminal** (Linear/Bloomberg pro realitní deal-flow). |
| 1 | Typografie | **Bricolage Grotesque zůstává.** Fluid scale přes `clamp()`. Priorita oprav: mobil 360–390 px — Hero a karty cenových variant. Nic nesmí přetéct/oříznout se. |
| 2 | Kuply OS jazyk | Plná svoboda přeskládat sekce landingu pro lepší příběh. Statické ikony → **živé mikrovizualizace** (vrstvené sklo, parallax, ambient stíny, pulzující datové toky). |
| 3 | Footer | **CinematicFooter pouze na landingu**: curtain-reveal (fixovaný footer pod obsahem), obří obrysové KUPLY s parallaxem, dýchající aurora, diagonální marquee, magnetické skleněné pilulky, heartbeat badge, back-to-top orb. Dashboardy: čistá funkční lišta se specialistou 24/7 (existuje). |
| 4 | Hero | **3D prostorový karusel lead formulářů.** Čisté CSS 3D (`perspective:1200px`, `rotateY`, `translateZ`). Aktivní karta uprostřed 100% čitelná, `backdrop-filter: blur(40px)`; postranní karty se natáčejí do hloubky, tmavnou (gradient masky) a rozostřují. Pod karuselem pilulkové přepínače s momentum přetočením. |
| 5 | Sekce „Co následuje" | **Druhý 3D karusel — stejná mechanika.** 6 karet procesu (01 Sofiin rozbor → 06 Peníze v úschově), aktivní levituje (hluboké ambient stíny), pilulky 01–06. **Pod ním FOMO teaser**: anonymizovaný rozmazaný karusel privátních příležitostí. |
| 6 | Sofia Core | Sova v kolečku pryč. **Hybrid A+E**: zlatý pulzující energetický orb, při interakci/přemýšlení se uvnitř rozsvítí minimalistický glyph sovích očí. Persona: landing/prodávající = **„Sofia"**, investor dashboard = **„Kuply Intelligence"**. **Scroll-driven let**: orb doprovází scroll s lerping setrvačností, náklon dle rychlosti, dojezd při zastavení; reaguje na obsah (záblesk aury u aktivní 3D karty). |
| 7 | Investor timeline | Po akceptaci prodávajícím (demo: simulace po pár sekundách) → stav **„Offer Accepted"** + toast + změna na kartě → rozbalí se **interaktivní timeline 10 kroků** s % progresem a ETA. Kombinace reálných CZ termínů (Katastr ≈ 20 dní) + zvýraznění, že kroky Kuply trvají hodiny. |
| 8 | Nedotknutelné pilíře | (1) Tři **stupňující** varianty ceny (garance rychlosti vs. max zisk). (2) Přesný výsledek **až po kontaktu**. (3) Bezpečí: **advokátní úschova + Trinity** financování. |

**Proces:** implementace POUZE po explicitním schválení fáze. Před většími změnami návrh + UX logika + otázky.

---

## 2 · DESIGN SYSTÉM

### 2.1 Tokeny — Warm Futurism (landing + seller + tipař)
```css
--paper:#FBF7EE; --ivory:#FFFCF4; --paper-2:#F3EBD7;
--cocoa:#3D362E; --cocoa-2:#4A3F33; --cocoa-deep:#201913;  /* stage/scény */
--gold:#C9A876; --gold-l:#E0C997; --gold-d:#8E7340;
--rust:#B5663A; --rust-d:#9A5430; --positive:#5E7C4F;
--glass-w1:rgba(255,252,244,.93); --glass-w2:rgba(255,252,244,.74);
--glass-brd:rgba(255,253,247,.55);
/* sklo: blur 16–40px + saturate 160–175%, inset top highlight rgba(255,255,255,.95) */
```
Zlato = jediný „zdroj energie" (aury, jádra orbů, datové toky, aktivní stavy). Rust = akce/CTA.

### 2.2 Tokeny — Dark Terminal (investor)
```css
--void:#0B0B0E; --panel:#13131A; --panel-2:#1A1A23;
--line:rgba(255,255,255,.07); --line-2:rgba(255,255,255,.13);
--txt:#ECE9E2; --txt-2:rgba(236,233,226,.62); --txt-3:rgba(236,233,226,.38);
--acc:#E0C997;            /* brandové zlato = jediný akcent, drží koherenci */
--signal-up:#7FD99A; --signal-warn:#E8B45A; --signal-down:#E07A6B;
--glass-d:rgba(19,19,26,.72); /* + blur 20px; jemný gold rim-light na hover */
```
Hustota dat: mono čísla (JetBrains), 12–13px metadata, tabulkové řádky, sparklines. Pocit Bloomberg, ne marketing.

### 2.3 Typografie (fluid)
```css
--fs-hero: clamp(34px, 6.8vw, 78px);      /* Hero H1 */
--fs-h2:   clamp(26px, 4.6vw, 52px);
--fs-h3:   clamp(19px, 2.6vw, 28px);
--fs-body: clamp(14.5px, 1.6vw, 17px);
--fs-data: clamp(11px, 1.2vw, 13px);      /* mono */
line-height nadpisů 1.04–1.12; letter-spacing -0.02em; žádné fixní px nadpisy.
```
Fonty: Bricolage Grotesque (display), Inter (text), JetBrains Mono (data/labels).
**Audit povinný na:** 360 / 390 / 768 / 1024 / 1440 / 1920 px. Kritická místa: Hero H1, tři cenové karty, fin-hero číslo, timeline labels.

### 2.4 Anti-vzory (zakázané)
Běžné karty v gridu „nalepené na pozadí" · generické ikony · stock startup look · WordPress bloky · ploché 2D slidery · statické sekce bez vlastní identity.

---

## 3 · KOMPONENTY (inventář a stav)

| Komponenta | Stav | Poznámka |
|---|---|---|
| `Carousel3D` (engine) | **NOVÁ — Phase 1** | Jeden znovupoužitelný vanilla modul pro Hero i Proces. Spec §4. |
| Hero lead-karusel | **NOVÁ — Phase 1** | 3 karty: Tržní odhad · **Privátní rozbor (center, default)** · Rychlá konzultace (callback). Napojení na existující flow logiku. |
| `SofiaCore` orb | **NOVÁ — Phase 1** (statické stavy) / **Phase 2** (scroll-let) | Spec §5. Nahrazuje sovu všude. |
| Fluid type systém | **Phase 1** | Tokeny + audit + opravy přetékání. |
| Proces-karusel 6 kroků | **Phase 2** | Stejný engine, levitace, pilulky 01–06. |
| FOMO teaser karusel | **Phase 2** | Anonymizováno: „Praha 2 · 3+kk · výnos 4,8 %", cena rozmazaná, CTA „Získat přístup". |
| Mikrovizualizace | **Phase 2** | Pulzující body, orbity, sparklines, datové toky místo ikon. |
| Scroll choreografie | **Phase 2** | Reveal vrstvy, parallax, Sofia let. |
| `CinematicFooter` | **Phase 3** | Vanilla (bez GSAP): curtain-reveal, obrysové KUPLY, aurora, marquee, magnet pilulky. |
| Investor Dark Terminal | **Phase 4** | Kompletní restyle investor zóny do dark tokenů; rail + command bar + dense deal-flow; modal → dark panel. |
| `DealTimeline` | **Phase 4** | 10 kroků, %/ETA, simulace akceptace. Spec §6. |
| Seller/Tipař dashboard | **HOTOVO (v5)** | Zůstává Warm; jen kosmetika orbů + typo audit. |
| Lead flow logika | **HOTOVO (v5)** | 5 kroků, gate, stupňující ceny — beze změn logiky, nový obal. |
| Footer-lišta dashboardů | **HOTOVO (v5)** | Specialista 24/7, tel/mail. Zůstává. |

---

## 4 · MOTION SYSTÉM

### 4.1 Tokeny pohybu
```
dur: 150 (micro) · 300 (base) · 600 (scéna) · 900 (cinematic)
ease-standard: cubic-bezier(.2,.8,.2,1)
ease-enter:    cubic-bezier(.22,.9,.24,1)
ease-momentum: lerp faktor 0.08–0.14 (rAF)
```

### 4.2 Carousel3D — závazná specifikace
- Kontejner `perspective:1200px`; karty absolutně, pozice = f(offset od aktivní):
  `rotateY(±28° · offset)` · `translateX(±62% · offset)` · `translateZ(-180px · |offset|)` · `scale(1 − .07·|offset|)`.
- Aktivní: blur(40px) sklo, plná čitelnost, ambient stín ODDĚLENÝ pod kartou (vlastní element, ne box-shadow na kartě → levitace).
- Neaktivní: gradient maska tmavnutí (`linear-gradient` overlay, opacity dle |offset|), `filter:blur(2–4px)` **jen na malých kartách** (viz §8 výkonnostní pravidla), pointer-events none.
- Ovládání: pilulky pod karuselem (aktivní = zlatá), drag/touch s momentum (rAF lerp k cílovému indexu), wheel→horizontal, snap na střed, šipky klávesnice.
- Stav v JS objektu `{index, target, velocity}`; render přes `requestAnimationFrame`, pouze `transform/opacity`.

### 4.3 SofiaCore — stavy a let
- **Stavy:** `idle` (pomalý puls jádra 3 s), `thinking` (glyph očí fade-in + rychlejší puls + jiskry synapse), `alert` (zlatý ring expand), `landing-greet` (jednorázové rozsvícení).
- **Anatomie:** vnější skleněná koule (radial-gradienty, ŽÁDNÝ filter blur na velké ploše) → zlaté jádro (2 vrstvy radial, puls scale 0.96–1.04) → glyph očí (2 oblouky + 2 body, stroke-dashoffset reveal) → orbit částice (3–5 bodů, CSS offset-path).
- **Scroll-let (Phase 2):** pozice = lerp(current, targetY, .1); náklon `rotate(clamp(velocity·k, ±12°))`; dojezd přirozený z lerpu; kolize-reakce: při průletu kolem `[data-orb-react]` elementu (aktivní karta) → `alert` mikro-záblesk. Fallback `prefers-reduced-motion`: orb statický v rohu.

### 4.4 CinematicFooter mechanika
Obsah stránky `position:relative; z-index:1; margin-bottom:100vh` (resp. clip-path wrapper) → footer `position:fixed; bottom:0; height:100vh; z-index:0` → stránka ho scrollem „odkrývá". Parallax obřího obrysového KUPLY přes rAF (ne GSAP). Aurora = 2 radial vrstvy, animace transform/opacity. Magnetické pilulky: mousemove → lerp translate ±8 px + rim-light.

---

## 5 · UX LOGIKA — NARATIV LANDINGU (nové pořadí sekcí)

1. **Nav** — teplé sklo, demo přepínač rolí, Sofia orb mini.
2. **Hero = Kuply OS** — claim + cyklující slovo + **3D lead-karusel** (3 formulářové karty). Sofia orb se „probudí" a pozdraví.
3. **Live trust strip** — mikrovizualizace: počet investorů v síti (pulz), poslední transakce (anonym ticker), škrtnutá provize.
4. **Tři pohledy na cenu** — přepracovaná scéna v OS jazyce (vrstvené sklo, datové toky mezi kartami), pilíř stupňování nedotčen.
5. **Proces „Co následuje"** — druhý 3D karusel 01–06 + pod ním **FOMO teaser karusel**.
6. **Příběhy** — redesign na „záznamy transakcí" (mono metadata, výsledková čísla), ne testimonial bubliny.
7. **Otázky** — minimal accordion, mono indexy.
8. **CinematicFooter.**
Sofia orb letí celou cestou (pravý okraj, ~64 px, na mobilu 44 px), reaguje na sekce.

**Investor Dark Terminal (Phase 4) layout:** levý rail (ikonové moduly: Deal-flow / Moje nabídky / Portfolio / Programy / Off-market / Intelligence) · horní command bar (search, filtry, limit, „Kuply Intelligence" orb) · střed: dense deal řádky/karty s ratingem, výnosem, countdownem, Kč/m², sparkline lokality · detail = dark panel s galerií a slepou licitací · **Portfolio = DealTimeline**.

---

## 6 · DEALTIMELINE — datový model a demo simulace

Kroky (fixní pořadí, `id, label, owner, etaText, etaHours`):
```
01 Offer Submitted        · investor · okamžité        · 0
02 Seller Review          · seller   · hodiny          · 24
03 Offer Accepted         · seller   · —               · 0
04 Legal Verification     · KUPLY ⚡  · 24–48 h         · 36
05 Due Diligence          · KUPLY ⚡  · 2–4 dny         · 72
06 Contract Preparation   · KUPLY ⚡  · 24 h            · 24
07 Digital Signing        · obě str. · hodiny          · 12
08 Escrow Funding         · investor · 1–2 dny         · 36
09 Ownership Transfer     · Katastr  · ~20 dní (zákon) · 480
10 Investment Active      · —        · —               · 0
```
- ⚡ = „Kuply rychlost" badge (zlatý blesk) — vizuálně odlišit od zákonných lhůt (Katastr šedě, s vysvětlivkou).
- UI: vertikální interaktivní timeline; hotové = zlatá výplň + check; aktuální = pulzující uzel + ETA countdown; budoucí = tlumené; nahoře celkové **% dokončení** (vážené etaHours) + „očekávané dokončení: DATUM".
- Demo: po `submitBid` → za ~8 s seller akceptuje → toast „Nabídka akceptována" → karta do stavu Offer Accepted → timeline se rozbalí; kroky 04–06 auto-postupují à ~6 s (ať je vidět život), 07+ čekají (ETA běží).
- Stav: `won.status` rozšířit na `step:1..10, stepStartedAt`, render z modelu.

---

## 7 · BUILD PIPELINE & TECH POZNÁMKY (kritické pro navázání)

- **Assembler:** `/home/claude/klika/build/build.py` — extrahuje leady landing, aplikuje JSX/CSS mody (každý přes `must()` exact-string assert → před editem VŽDY grep přesný anchor), babel kompilace, statický tailwind (content scan `landing.js`+`static.html`), scope_css prefixuje dashboard CSS pod `#dashWrap` (přeskakuje :root/html/body, rekurze @media, @keyframes beze změny).
- **Fotky:** `photos.json` (PH0 kuchyň, PH1 koberec/sofa, PH2 stolek, PH3 teplé světlo, DASH blur pokoj, STAGE tmavý čistý crop, BGLIGHT) — generované z uploads; Pillow crop+blur+b64.
- **Knihovny:** react/react-dom 18.3.1 UMD inline; @babel/core+preset-react; tailwindcss 3.4.14 — vše v `build/node_modules`. GSAP/framer-motion/tsparticles NEpoužívat (no-UMD/výkon) — efekty vanilla.
- **Z-index mapa:** `#app{z-index:auto}` (NIKDY mu nedávat z-index — trap modalu), `.topbar{z-index:80}`, `.modal-bg{z-index:600}`, CinematicFooter pod obsahem (z 0 vs 1).
- **Shrink lekce:** flex/grid děti potřebují `min-width:0` (modal, bid-in input) — při nových layoutech hlídat.

### 7.1 Výkonnostní pravidla (tvrdě vynucovat)
1. ŽÁDNÝ `filter: blur()` na velkých plochách (zamrzlý first-frame; povolen jen na malých elementech ≤ ~250 px).
2. ŽÁDNÝ `mix-blend-mode` na velkých vrstvách.
3. NEkombinovat `transform-style:preserve-3d` s `backdrop-filter` na stejném elementu.
4. Animovat výhradně `transform` + `opacity`; rAF + lerp pro momentum.
5. `prefers-reduced-motion` fallback u všech systémů (karusel → fade, orb → statický, footer → bez parallaxu).
6. Headless QA: po aktivaci scény čekat ≥ 2,2 s před screenshotem (software renderer; na GPU ok).

### 7.2 QA protokol (každá fáze)
`node --check` obou skriptů → Playwright matice 360/390/768/1024/1380(+700 výška) → kontrola: žádné JS chyby, žádný horizontální overflow (`document.body.scrollWidth === innerWidth`), nadpisy nepřetékají (programový audit `el.scrollWidth > el.clientWidth`), klíčové flow kliknutelné. Screenshoty před označením fáze za hotovou.

---

## 8 · FÁZOVÝ PLÁN (každá fáze = schvalovací brána)

- **PHASE 1 — Foundations & Hero** *(čeká na schválení)*
  Dual-theme tokeny + fluid typografie + audit/oprava přetékání (priorita mobil Hero + cenové karty) · `Carousel3D` engine · Hero 3D lead-karusel (3 karty, pilulky, momentum, drag/touch) · `SofiaCore` orb v0 (idle/thinking/alert; nahrazuje sovu na landingu i v chatu; „Kuply Intelligence" label u investora) · výstup: build + screenshot matice.
- **PHASE 2 — Landing narativ:** přeskládání sekcí dle §5 · proces-karusel 01–06 · FOMO teaser · mikrovizualizace místo ikon · scroll choreografie + Sofia let.
- **PHASE 3 — CinematicFooter** (jen landing).
- **PHASE 4 — Investor Dark Terminal + DealTimeline** + simulace akceptace.
- **PHASE 5 — Polish:** mikrointerakce, kompletní QA matice, og/SEO, README + aktualizace tohoto dokumentu.

## 9 · PENDING / OTEVŘENÉ OTÁZKY
1. Hero karta č. 3 = „Rychlá konzultace — zavoláme do 15 minut" (jméno+telefon)? Návrh, čeká na potvrzení.
2. Dark terminal: zlato jako jediný akcent (doporučeno kvůli brandu) — potvrdit.
3. FOMO teaser: plně fiktivní anonymizovaná data s rozmazanou cenou — potvrdit.
4. Schválení **PHASE 1**.


---

## 10 · CHANGELOG — PHASE 1 (DOKONČENO)

**Dodáno (build v6, 577 kB, QA 19/19 PASS):**
- Dual-theme tokeny (`p1_theme.css`): `:root` warm + `[data-theme="dark"]` terminal sada (--acc #E0C997 jediný akcent) — dark se aplikuje ve Phase 4.
- Fluid typografie: `--fs-hero/h2/h3/card/body/data` přes clamp(); Hero h1 + cenové karty + flow nadpisy převedeny z fixních px; overflow audit 360–1380 čistý.
- `Carousel3D` engine (`p1_car3d.js`): perspective 1200, rotateY ±28°·off, translateZ −180·|off|, scale 1−.07·|off|, side opacity 0.65→0.3; **time-based lerp** `k = 1−0.88^(dt/16.7)`, dt cap 150 ms (fps-nezávislá inertia); drag + touch + wheel(deltaX, lock 260 ms) + pilulky + šipky; snap-to-center; oddělené levitační stíny (.lc-shadow); klik na boční kartu = fokus; root.isConnected GC.
- Hero 3D lead-karusel (`p1_hero.jsx`): Tržní odhad · **Privátní rozbor (střed, dark glass blur 40px)** · Rychlá konzultace; gradient veil + blur 2.4px max na bočních (desktop only).
- `QuickCall` (`p1_quickcall.jsx`): jméno+telefon, validace ≥9 číslic, success stav se Sofia thinking.
- `SofiaCore` v0 (`p1_sofia.js/.css`): orb (jádro+orbit+ring+glyph očí), stavy idle/thinking/alert; MutationObserver auto-mount na `[data-sofia]`; API `SofiaCore.set(el,state)`. **Event hooky z enginu:** drag start→thinking, release bez změny→idle, snap/act change→alert záblesk; QuickCall success→thinking. Nahrazuje sovu na landingu (badge, karta, floating chat, flow orby přes přepsaný SofiOrb→data-sofia mount).
- build.py: moduly načítány ze souborů `build/p1_*.{css,js,jsx}`; `engines.check.js` v node --check pipeline.

**Opravy za běhu (poučení):**
- Per-frame lerp ⇒ time-based (nízké/vysoké fps měnily rychlost dojezdu).
- Tablet 768–1023: nav menu linky přetékaly (`nav div.md\:flex.gap-1:not(.p-1){display:none}` v tom pásmu; pilulky rolí + CTA zůstávají).
- Hero section už má Tailwind `overflow-hidden` — 3D karty stránku nešíří (přidané cliply jsou no-op pojistka).
- QA drag: reálná myš ověřena sondou; matice používá syntetické PointerEvents (deterministické v headless).

**Engine API (pro Phase 2 — proces karusel + FOMO jen markup `[data-car3d]` + `.lc-card`/`.lc-pill`):**
`Car3D.scan()`, `car.go(i)`, `car.pos/target`, `data-start` atribut.


---

## 11 · CHANGELOG — BETA-WOW (v7, reakce na zamítnutí Phase 1)

**Sofia jako živá entita (`p2_flight.js` + fx CSS):** orb 64/46 px létá u pravého okraje, sleduje scroll s time-based lerp setrvačností, náklon dle rychlosti, bobbing, 3 trail částice; stavy thinking při letu / idle po dojezdu; kontextové bubliny u sekcí (IO na h2, uvítání po 1,6 s); dock u otevřené konzole (thinking „analyzuje"); klik = otevření Sofi chatu (proxy na skryté tlačítko, panel zachován); v dashboardech skryta. Oči = DOM oblouky + pupily (žádné SVG), aura radial puls.

**CinematicFooter (`p2_footer.*`, jen landing):** TRUE curtain — footer PORTÁLEM ven z #root (`position:fixed; z-1`), `#root{z-2; bg paper; box-shadow hrana}`, `.cine-spacer` portován na konec body = okno odhalení. Aurora dech ×2, jemná mřížka s maskou, obří obrysové KUPLY (23,5vw, stroke + gradient fill, rAF parallax dle progresu spaceru), diagonální marquee (CSS, duplikovaný track), magnetické pilulky (`[data-mag]`, lerp ±9 px), heartbeat ◆, back-to-top, CTA pilulky → scroll top ke karuselu. React onClick ve footeru NEfunguje po portálu — vše vanilla listenery.

**Hero atmosféra:** `.flow-section::before` vrstvené radiály (zlato/rust/horizont), `::after` noise textura (PNG 96px, 12 kB, `--noise`), `.glow` drift 16 s; `#cycleWord` zlato-rust shimmer (background-clip text).

**Scroll-to-flow:** druhý Hero effect `[path]` → `console-wrap.scrollIntoView({block:'start'})` po 80 ms — klik na Odhad/Rozbor vždy přemístí na formulář.

**Hloubka karuselu:** spacing 0.62→0.70 (desktop), veil .30→.78 gradient, side opacity 0.55, blur cap 3.2 — boční karty čtou jako hloubka, ne useknutí.

**P3 wow mikrovzory (adaptace z dodaných komponent, vanilla):**
- `AnimateNumber` → digit ticker `.kn` (mění se JEN změněné číslice, slide+blur .5 s, sloty klíčované zprava); živý počet investorů v `.lc-live`, tick à 4,5 s, API `kuplyLive.tick()`.
- `DigitalSerenity/BlurText` → word-appear H1: vanilla obal slov `.wa` (blur 12→0, y22→0, stagger .085 s); `#cycleWord` přeskočen (ID shimmer by přebil animaci a slovo by zmizelo!).
- `CinematicHero sheen` → `.lc-sheen` na tmavé kartě, radial světlo sleduje kurzor (CSS vars, rAF throttle).
- GSAP/shadcn/motion/radix komponenty NELZE 1:1 (single-file UMD bez npm runtime) — vždy přepsat vanilla dle výkonnostních pravidel.

**Kritické lekce:**
- Fixovaný footer pod transparentními sekcemi PROSVÍTÁ středem stránky — lift musí být na `#root`, ne `:where(section)`; spacer ven z rootu.
- Absolutně pozicovaná bublina v úzkém containing blocku (64px orb) se zmáčkne — nutné `width:max-content`.
- Footer slice v leady je uvnitř `const Footer = () => (...)` — náhrada se spacer+footer = adjacent JSX → obalit `<>…</>`.

**QA v7: 19/19** (entita letí, bubliny, ticker, sheen, word-appear, curtain bez prosvítání — elementFromPoint guttery, footer reveal + parallax, scroll-to-flow rect, dash skrývání, overflow matice 360–1380, 0 JS chyb).


---

## 12 · CHANGELOG — v8 (kompletní leady + redesign + sova zpět)

**SpecForm (`p4_flow.jsx`) — 100% zdravý lead:** nová fáze „Specifikace" v privátním flow (sub-phase kroku Plocha: `phase2 m2→spec`, `submitSpec→step 3`; steps label „Plocha"→„Specifikace"). Typově specifická pole `SPEC_FIELDS`: byt (dispozice✱, vlastnictví✱, č. jednotky, podlaží, příslušenství), dům (typ stavby✱, pozemek m²✱, podlažnost, příslušenství), pozemek (LV✱, kat. území✱, druh✱, sítě), činžák (jednotky✱, obsazenost, LV), komerční (účel✱, pronajato✱), developer (fáze✱, jednotky). Živý **měřič kompletnosti** (%, zlatá lišta), gate na povinných polích, badge „✓ ZDRAVÝ LEAD — PŘIPRAVEN PRO INVESTORY". Vstup do PrivateEstimate slice-em: guard `{step === 2 && selected && (` rozdvojen na spec/m2 větve (POZOR: guard obsahuje `selected &&`!).

**Sova zpět (Roman: „orb se mi nelíbí"):** SofiaCore renderuje `<use href="#owl-hero">` (86 %) uvnitř zlatého orbu — stejná sova jako v dashboardech, všude (flight, karty, mini citace). `hoistDefs()` přesouvá defs SVG z (display:none) dash šablony na začátek body, jinak by `use` nemusel renderovat. `.sf-eye` mrtvé (display:none).

**Hero karusel — definitivně bez „useknutých" nadpisů:** spread layout spacing 0.95/0.72, scale side 0.87, tz −140, blur ≤1.6, veil ≤0.5, opacity side ~0.78 → boční karty PLNĚ čitelné, nic nepřekrývá tituly; mobil: veil gradient do 0.92 (slivery = tmavé sklo beze čtitelných fragmentů).

**Proces sekce → druhý Carousel3D (`p4_proc.jsx`):** 6 tmavých levitujících karet 01–06 (zlatý badge, sofí citace s mini sovou), pilulky, stejná mechanika (Car3D.scan přidán do p3 boot intervalu — sekce je mimo Hero effect!). Pod ním **FOMO teaser**: 3 anonymizované příležitosti s blur(5px) cenami + „Získat přístup".

**Příběhy → sticky card-stack (`p4_stories.jsx`, adaptace CardsStack promptu):** karty `position:sticky; top:96+i·34; z:i` se vrší při scrollu, mono „ZÁZNAM 0i/06". Bez knihoven.

**Word-appear na h2 (adaptace DigitalSerenity):** p3 `wrapH2()` — slova `.wa` paused + IntersectionObserver spouští při 35 % viditelnosti; pojistka 7 s. H1 stagger zrychlen (0.06+i·0.05, dur .6s) kvůli headless screenshotům. Footer `.cine-h` zlatý glow (Illuminated adaptace).

**Slice mechanika (kritické pro navázání):** proces i příběhy používají IDENTICKÝ grid anchor `grid md:grid-cols-2 lg:grid-cols-3 gap-5` — pořadí slice-ů: první výskyt = proces, druhý = příběhy; konec bloku = `))}\n      </div>`.

**QA v8:** 18/18 reálných (jediný FAIL = artefakt testu dělícího projektovanou šířkou rotované karty) + e2e: Byt → adresa (suggestions jsou BUTTONy s textem bez čárky „Vinohradská 12Praha 2"!) → m² → SpecForm 0→40 % → Detaily; sova bbox v flight i kartě; proc pill snap; FOMO blur computed; sticky computed; overflow matice čistá; 0 JS chyb; H1 .wa opacity 1 @3.8 s.


---

## 13 · CHANGELOG — v9 (FAQ · tilt · grain)

**Tři pohledy → 3D tilt karty (`p5_tilt.js`/`p5.css`):** grid dostal `.tilt-row` (perspective 1100); každá karta mouse-tilt rotateX ±9° / rotateY ±11° + translateZ 10, rAF throttle, spring návrat .55 s; `.tilt-glare` radial světlo sleduje kurzor; hover ambient stín; vstupní stagger reveal přes IO (`.tin`, delay ×.12 s, pojistka 6 s). POZOR: žádný `(hover:hover)` gate — headless hlásí hover:none a gate zabíjel QA (lekce).

**FAQ wow:** render items vyměněn (slice v build.py): `<button>` řádky se zlatým mono indexem 01–0n, plus→× ikona (2 pseudo-bary, rotace 135° + zlatá výplň), glass gradient, hover gold rim + lift, open stav se zlatým rámem a dashed dělítkem; `.faq-a` max-height transition (specificita `.fx` přebíjí původní styly). Logika open/setOpen původní, nedotčená.

**Globální filmový grain:** `#grainFx` fixed vrstva (z-240, opacity .05, `--noise` 140px) přes celý landing vč. footeru; interval toggluje s viditelností #root (v dashboardech skrytý). Hero-only grain `::after` vypnut (content:none) — žádné zdvojení.

**QA v9:** grain computed + dash hide, tilt rotace probe (rotateX/Y v transformu), tin reveal, glare, FAQ toggle s výškou odpovědi, 0 JS chyb, overflow 360/1380 čistý.
