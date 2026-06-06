# Kuply — kompletní web (prototyp)

Jeden statický soubor `index.html`, bez buildu a závislostí. Maskot **sova Sofia** (z tvého brand kitu) provází celým procesem.

## Brand
- **Barvy i fonty z tvého kitu** (`kuply_cz-kit.html`): Bricolage Grotesque (nadpisy), Inter (text), JetBrains Mono (čísla); paleta paper/cream + gold + espresso + rust + sage.
- **Sova Sofia** vložená 1:1 z `koupime-ads__1_.html` (pózy hero / peek / happy / wink / wow + animace bliknutí a třepot křídel).
- Vizuál: teplý, prémiový **glassmorphism** (frosted sklo, jemný grain, zlaté přechody) — „future-tech", ale minimalistický. **Vše světlé**, žádné tmavé UI.

## Co web obsahuje
**Landing (Web):** hero s formulářem na leady hned navrchu · pruh s čísly · živý přehled aktivity · Jak to funguje (se Sofií) · Proč Kuply (benefity + srovnání) · Pro koho · Reference · Bezpečí a důvěra · sekce Pro investory · FAQ · CTA · patička.

**Provázený formulář:** 4 kroky, Sofia u každého kroku radí (mění pózy a bubliny). Na konci spočítá odhad ceny a spustí sběr nabídek.

**Dashboard prodávajícího (světlý, profi):** glass sidebar, statistiky, zlatá countdown karta, živě naskakující nabídky, sparkline trendu, 3D náhled, právní semafor, přijetí → oslavná obrazovka (Sofia).

**Dashboard investora (světlý, profi):** glass sidebar + topbar s vyhledáváním, statistiky, taby, deal-flow karty s 3D náhledem, detail se slepou nabídkou.

Vše propojené: co prodávající zadá, investor uvidí; co investor nabídne naslepo, vrátí se prodávajícímu v reálném čase. Přepínání rolí vpravo nahoře (Demo: Web / Prodávající / Investor). Data jsou ilustrativní a žijí v paměti (refresh = reset).

## Nasazení na Vercel
**A — přetažením:** vercel.com → New Project → přetáhni složku s `index.html`.
**B — CLI:** `npm i -g vercel`, pak ve složce `vercel`. Žádná konfigurace.

## Pro plný systém
Logika je v jednom `<script>`: `store` (stav), `render()` (router), komponenty `Landing / Intake / Seller / Investor`, `shell()` (dashboard layout), maskot přes `<use href="#owl-…">`. `store` se nahradí backendem (Supabase: `properties`, `offers`), `pushOffer` realtime kanálem, `submitProperty` API + soukromým odkazem. Texty/čísla/reference jsou nahoře v polích (`TESTIMONIALS`, `FAQS`, `SITUATIONS`).
