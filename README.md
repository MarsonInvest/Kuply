# Kuply — finální prototyp (landing pro leady + dashboardy)

Jeden soubor `index.html` (~507 kB), **bez CDN závislostí** — React, Tailwind i fotky jsou zkompilované a vložené přímo v souboru. Funguje offline, načítá se okamžitě, nasazení = přetáhnout na Vercel.

## Základ: tvůj `kuply-landingpage_pro_leady.html` — 1:1
Design, texty, Sofi, dvě vstupní karty (Tržní odhad / Privátní rozbor), všechny sekce zachované beze změn. Upraveno jen to, co jsi zadal:

1. **Formuláře se točí na místě** — každý krok rozboru vjede do pevného skleněného panelu jemnou 3D rotací (perspective + rotateY + blur). Stránka neskáče, panel stojí, obsah se vyměňuje. (Respektuje `prefers-reduced-motion`.)
2. **Stupňující pořadí cen** — výsledek rozboru: **Bankovní → Portálový → ★ Privátní investorská** (nejvyšší, „+X % proti portálu"). Hodnoty i vizuální lišty rostou zleva doprava.
3. **Výsledek až po kontaktu** — beze změny logiky tvého souboru (loading → nález → kontakt → odhalení).
4. **Most do produktu** — po odhalení přibyl cocoa banner „DALŠÍ KROK · NEZÁVAZNĚ · 0 Kč" s CTA **„Spustit nabídky od investorů"** → založí nemovitost z dat rozboru a otevře dashboard prodávajícího; nabídky chodí živě (první ~92 % portálu, později i **nad odhad**).
5. **Přepínač rolí v nav** — Web · Prodávající · Investor · Tipař (demo).

## Dashboardy — stejný designový jazyk, skutečné fotky
- **Fotky všude:** karty nemovitostí, náhled „Stav nemovitosti", modaly detailu i vyhraných — reálné výřezy z dodaných interiérových fotek (vložené ~80 kB). Pozadí dashboardu = sklo plovoucí nad teplou fotkou interiéru.
- **Prodávající:** nabídky (hotovost/hypotéka + termín), countdown s Prodloužit/Smazat, **Moje nemovitost** (základní údaje zamčené se zdůvodněním, doplňující info + dokumenty editovatelné), chat se Sofií, FAQ, po prodeji profil + cross-sell.
- **Investor:** rating A+/A/B + výnos %, slepá licitace (hotovost/hypotéka → Trinity 3,99 %, termín 7/14/30/60 d), **Vyhrané** s procesem dokončení transakce, doprovodné programy, off-market, FAQ.
- **Tipař:** tipy + provize a stavy.
- **Footer:** specialista Marek Král, 24/7 na příjmu, klikací telefon a e-mail.

## Technika
- React 18 + zkompilovaný JSX (žádný Babel v prohlížeči), Tailwind vygenerovaný staticky (9 kB), dashboard vrstva ve vanilla JS scopovaná pod `#dashWrap` (nekoliduje s landingem).
- Ověřeno Playwrightem: celý lead flow proklikán (typ → adresa → m² → otázky → cíl → loading → kontakt → odhalení → spuštění nabídek), dashboardy, mobil 390 px — bez JS chyb.
- Napojení na backend: `window.kuplyLaunch(payload)` je jediný most landing → produkt; v dashboardu nahradit `store` za API.

## Nasazení
vercel.com → New Project → přetáhnout soubor. Hotovo.
