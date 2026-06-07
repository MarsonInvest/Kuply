# Kuply — kompletní web + dashboardy (prototyp)

Jeden statický soubor `index.html`, bez buildu a závislostí. Maskot **sova Sofia** provází prodávající celým procesem. Vše ve světlém, prémiovém glassmorphism stylu z tvého kitu.

## Demo přepínač (vpravo nahoře): Web · Prodávající · Investor · Tipař

## Landing (Web) — důvěra a nízké bariéry pro prodávající
- **Gamifikovaná oceňovačka v heru** — hned ukáže cenové **rozpětí**. Jak vyplňuješ lokalitu, dispozici, plochu a stav, cena se hýbe (▲/▼ delta), roste lišta přesnosti a Sofia komentuje. Framing je pozitivní: *někteří investoři zaplatí i nad odhad — a poznáte to během hodin*. CTA „Chci přesné nabídky" otevře hlavní formulář.
- Bez cílení na investory. Důvěra přes **advokátní úschovu + Trinity Bank**, prověření katastru, reference, FAQ.
- **Program Tipař** přímo na landingu (provize až 0,5 %).

## Provázený formulář (Sofia pomáhá)
4 kroky, Sofia u každého radí *proč* vyplnit kompletní info (= vyšší a víc nabídek). Krok **úschova**: výběr **Advokátní úschova** vs **Trinity Bank** + délka sběru 24/48/72 h.

## Dashboard prodávajícího (světlý)
Statistiky, nabídky investorů s **formou úhrady (hotovost/hypotéka) a termínem dokončení**, zlatá countdown karta s **Prodloužit** (nastavení času) i **Smazat prodej**, **chat se Sofií**, **FAQ**, právní semafor, oslavná obrazovka po přijetí.

## Dashboard investora (bez Sofie)
- **Rating nabídek A+/A/B + odhad. hrubý výnos %.**
- Licitace naslepo: částka + **forma úhrady** (hotovost/hypotéka → **Trinity financování 3,99 %**) + **termín dokončení** (7/14/30/60 d).
- **Doprovodné programy:** správa nemovitosti, rekonstrukce, financování Trinity, právní servis.
- **Off-market:** investor nabídne vlastní nemovitost diskrétně dalším investorům.
- **FAQ** pro investory.

## Dashboard tipaře
Formulář na tip + přehled tipů se stavy (Oslovujeme / Prodáno) a **provizí** za úspěšný tip.

## Brand
Bricolage Grotesque + Inter + JetBrains Mono; paleta paper/cream + gold + espresso + rust; sova **Sofia** 1:1 z ad kitu (pózy + animace). Žádné tmavé UI.

## Nasazení na Vercel
A) vercel.com → New Project → přetáhni složku. B) `npm i -g vercel`, pak `vercel`. Bez konfigurace.

## K propojení s backendem
Logika v jednom `<script>`: `store` (stav) → router `render()` → `Landing / Intake / Seller / Investor / Tipar` + `shell()`. Stačí `store` nahradit API (např. Supabase: `properties`, `offers`, `tips`), `pushOffer` realtime kanálem. Cenový model = objekt `DISTRICTS`, rating = `gradeProp()`. Data, čísla i reference jsou ilustrativní (refresh = reset).
