# Angelcsev Zoé — tanácsadói landing oldal

Statikus oldal, nincs build lépés. Egy `index.html` + `assets/` mappa.

## Fájlok

```
index.html              a teljes oldal (CSS és JS beágyazva)
assets/zoe.jpg          hero portré
assets/zoe-cartoon.jpg  hero flip hátlap (rajzolt)
assets/og-zoe.jpg       megosztási kép (Facebook, LinkedIn)
assets/partners.jpg     partner logósáv
assets/av-*.png|jpg     csapat avatarok
```

## 1. GitHub Pages

1. Új repo, a fenti fájlok a gyökérbe.
2. Settings → Pages → Source: `Deploy from a branch`, branch: `main`, mappa: `/ (root)`.
3. Pár perc múlva él: `https://<felhasznalo>.github.io/<repo>/`

## 2. Rackhost

Kétféle út:

**A) Kézi**: FTP-vel másold fel az `index.html` fájlt és az `assets/` mappát a `public_html` (vagy `web`) könyvtárba.

**B) Automatikus** (ajánlott): a `.github/workflows/deploy-rackhost.yml` minden `main` pushnál feltölti.
Állítsd be a repo Settings → Secrets and variables → Actions alatt:

| Secret | Érték |
|---|---|
| `FTP_SERVER` | pl. `ftp.sajatdomain.hu` |
| `FTP_USERNAME` | Rackhost FTP felhasználó |
| `FTP_PASSWORD` | FTP jelszó |

A workflow `server-dir` értékét igazítsd a tárhely könyvtárszerkezetéhez (`public_html/` vagy `web/zoe/`).

## 3. Domain

Ajánlott: `zoe.eaglz.hu` aldomain. Rackhostban vedd fel az aldomaint, mutasson a feltöltött mappára, és kérj rá Let's Encrypt tanúsítványt.

Ha nem ez lesz a végleges URL, cseréld az `index.html`-ben:
- `<link rel="canonical">`
- `og:url` és `og:image`
- a JSON-LD `url` és `image` mezői

## 4. Lead űrlap

A form a közös Supabase `web_leads` táblába POST-ol.

- Először `advisor: "angelcsev.zoe"` és `source: "zoe-landing"` mezőkkel próbálkozik.
- Ha a tábla ezeket még nem ismeri (400-as válasz), automatikusan újraküldi a régi sémával, és a tanácsadó azonosítóját a `topic` mező végére teszi `[angelcsev.zoe]` formában.

**Teendő a Supabase oldalon:**

```sql
alter table web_leads add column if not exists advisor text;
alter table web_leads add column if not exists source  text;
```

**Biztonsági ellenőrzés (fontos):** az anon kulcs nyilvános, ezért a táblán csak beszúrás legyen engedélyezve, olvasás nem.

```sql
alter table web_leads enable row level security;

create policy "anon insert only" on web_leads
  for insert to anon with check (true);
-- SELECT policy anon szerepre NE legyen
```

## 5. Új tanácsadó hozzáadása

Másold a mappát, és az `index.html`-ben cseréld:
név, `LEAD_ADVISOR`, telefonszám, e-mail, MNB-szám, naptárlink, `hash=` és `name=` paraméterek az online kötés linkekben, `assets/` képek, meta és JSON-LD adatok.
