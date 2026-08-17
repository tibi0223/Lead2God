# Lead2Go weboldal

Statikus Astro oldal, Cloudflare Pages-re. A design a **Lead2Go Brand Kit v2** és az
alkalmazott landing terv alapján készült; a szerkezet a 22 magyar versenytárson végzett
mérésből származó blueprint 14 szekcióját követi.

---

## Egyetlen fájl, amit szerkeszteni kell

**`src/data/site.json`** — itt van minden üzleti adat: cégadat, elérhetőség, árak,
referenciák, vélemények, mérőkódok, űrlap-backend.

Két szabály:

1. **Ami üres marad, azt az oldal nem jeleníti meg.** Nincs csúnya lyuk, nincs kilátszó helyőrző.
   Az árszekció, a bizonyítéksáv, az esettanulmányok és a vélemények mind eltűnnek vagy
   őszinte változatra váltanak, ha nincs mögöttük valós adat.
2. **Kitalált szám, vélemény vagy logó soha.** A mért magyar mezőnyben a hitelesség a
   legritkább áru — ez a te előnyöd, ne dobd el.

Néhány kapcsoló, amit érdemes ismerni:

| Mit írsz be | Mi történik |
|---|---|
| `arak.savok[].ar` mindhárom kitöltve | Az árszekció 3 kártyás nyilvános árazásra vált. Amíg üres, a „Miért nincs kiírva ár?" változat megy. |
| `bizonyitek.szamok` vagy `.logok` vagy `.googleErtekeles` | A hero alatti garanciasáv helyére bizonyítéksáv kerül. |
| `referenciak[].eredmeny` | Az esettanulmány-szekció címe „Első projektek — a mérés fut"-ról „Mit hozott a gyakorlatban"-ra vált. |
| `kapcsolat.naptarUrl` | Beágyazódik a foglalónaptár a záró blokkba. **37 mért magyar versenytárs-oldalból nulla használ ilyet.** |
| `kapcsolat.telefon` | Megjelenik a fejlécben, a láblécben és a záró blokkban `tel:` linkként, külön konverziós eseménnyel. |
| `meres.ga4Id` / `adsId` / `clarityId` | Amelyik üres, ahhoz egyáltalán nem töltődik be script. |

---

## Parancsok

```bash
npm install
npm run dev        # http://localhost:4321
npm run build      # → dist/
node ellenorzes.mjs  # élesítés előtti automata ellenőrzés (build után futtasd)
node kep.mjs / fooldal   # képernyőkép asztali + mobil nézetben
node lh.mjs        # Lighthouse mobil audit
```

`node ellenorzes.mjs` 1-es kilépési kóddal áll le, ha bármelyik kapu bukik — CI-ba tehető.

---

## Mit ellenőriz az `ellenorzes.mjs`

- **Oldalanként:** konzolhiba, 404-es erőforrás, `lang`, title/description hossz, canonical,
  pontosan 1 db H1, alt szövegek, kép-méretek (CLS), JSON-LD, kilátszó helyőrzők,
  és **axe-core** teljes WCAG 2.2 AA futtatás.
- **Billentyűzet:** az első Tab a látható „Ugrás a tartalomra" linkre visz, minden fókuszálható
  elemen látszik a fókuszjelölés, a GYIK harmonika billentyűzetről nyílik.
- **Űrlap:** pontosan 4 mező, mindegyiknek `<label>`, `type="tel"` és `type="email"`,
  üres beküldésnél mind a 4 hibaüzenetet ad.
- **Consent Mode v2:** alapból minden hirdetési és mérési tárolás `denied`; az „Elfogadom"
  ténylegesen `consent update`-et küld; a döntés újratöltés után megmarad.
- **Belső linkek és horgonyok:** nincs törött link, minden `#horgony` célja létezik.
- **Oldalsúly.**

---

## Szerkezet

```
src/
  data/site.json      ← minden üzleti adat
  data/gyik.ts        ← a 8 GYIK kérdés (JSON-LD-be is bekerül)
  styles/global.css   ← design tokenek (szín, tipó, rács, mozgás)
  layouts/
    Base.astro        ← <head>, SEO, JSON-LD, fejléc/lábléc, cookie-sáv, mérés
    Szoveg.astro      ← jogi és szöveges oldalak
  components/         ← 14 szekció + fejléc, lábléc, űrlap, cookie-sáv, mérés
  pages/              ← 10 oldal
public/
  fonts/              ← 6 self-hostolt woff2, magyar karakterkészletre szűkítve (66,6 KB)
  img/logo.svg        ← vektorizált logó (5,4 KB), inverz és favicon változattal
  admin/              ← Sveltia CMS
  _headers, _redirects, robots.txt, site.webmanifest
```

### Betűtípusok

Archivo (display), Instrument Sans (törzs), JetBrains Mono (címkék) — **self-hostolva**,
a magyar karakterkészletre szűkítve. Két ok:

1. **A magyar `ő` és `ű` a latin-ext tartományban van**, a Google Fonts `latin` alkészletében nincs benne.
   Ha valaki csak a `latin` subsetet tölti be, az `ő` és `ű` rendszerfontra esik vissza — látszik is.
2. A Google Fonts CDN-ről töltött betű **GDPR-kockázat** (több EU-s bírósági döntés is volt róla).
   Egy akadálymentességet és jogi megfelelést áruló oldalon ez nem elegáns.

A nyíl, csillag és pipa **inline SVG**, nem betűkarakter — azok a szűkített fontokban nincsenek benne,
és rendszerfontra esnének vissza.

---

## Deploy — Cloudflare Pages

1. Töltsd fel a projektet GitHubra.
2. Cloudflare → Workers & Pages → **Create → Pages → Connect to Git**.
3. Build parancs: `npm run build` · Kimeneti mappa: `dist` · Node: 20 vagy újabb.
4. Custom domain: `lead2go.hu` + `www` átirányítás az apexre (Cloudflare Redirect Rules).
5. A `public/_headers` és `public/_redirects` automatikusan érvényre jut.

### Sveltia CMS (`/admin`)

`public/admin/config.yml` → a `backend.repo` mezőbe írd be a GitHub repódat.
Belépéshez GitHub-fiók kell. Mentéskor git commit → Cloudflare újraépít → fél percen belül él.

Elvárás-kezelés az ügyfél felé: a **tartalom** szerkeszthető (szöveg, kép, ár, nyitvatartás),
az **elrendezés** nem. Ezt mondd ki előre, ne az átadáskor derüljön ki.

---

## Űrlap-backend

Alapból [Web3Forms](https://web3forms.com) (ingyenes, e-mail-cím megadásával 1 perc alatt
kapsz access keyt). Írd be a `site.json` → `urlap.accessKey` mezőjébe.

Amíg nincs kulcs, az űrlap validál, de nem küld — helyette kiírja, hogy a backend nincs bekötve.
**Élesítés előtt küldj magadnak egy valódi tesztüzenetet, és nézd meg, hogy meg is érkezik.**

Formspree-hez: `urlap.szolgaltato: "formspree"` és `urlap.endpoint` a Formspree URL-je.

---

## Mérés

A `src/components/Meres.astro` a következőket kezeli:

- **Consent Mode v2** alapállapot: minden hirdetési és mérési tárolás `denied`,
  `ads_data_redaction: true`, `url_passthrough: true`. Az EGT-ben ez nem opció.
- **GA4** és **Google Ads** csak akkor töltődik be, ha van azonosító a configban.
- **Konverziós események**, mindegyik külön: `urlap_bekuldes`, `telefon_kattintas`,
  `naptar_foglalas`, `uzenet_kattintas`, `cta_kattintas`.
- A `tel:` kattintást és a Cal.com foglalás-visszajelzést automatikusan elkapja.
- Saját eseményt a `window.ltgEsemeny('nev', {...})` hívással küldhetsz.

**Átadási kapu:** futtass egy valódi tesztkonverziót, és nézd meg, megjelenik-e 24 órán belül
a Google Ads felületén. Amíg ezt nem láttad, a mérés nincs kész — csak úgy néz ki.
"# Lead2God" 
