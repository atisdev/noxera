# Noxera Bistro — Vendéglátóipari Menedzsment Rendszer

> Budapest · Kávézó + Étterem + Bár · 2028 óta

Noxera egy teljes körű, PHP-alapú belső menedzsment platform, amely kávézó, étterem és bár üzemeltetéséhez készült. Lefedi a nyilvános megjelenéstől az érintőképernyős kassza-terminálokon át a bérszámfejtés előkészítéséig az összes munkafolyamatot.

---

## Tartalomjegyzék

1. [Technológiai stack](#technológiai-stack)
2. [Telepítés és konfiguráció](#telepítés-és-konfiguráció)
3. [Adatbázis felépítése](#adatbázis-felépítése)
4. [Nyilvános oldalak](#nyilvános-oldalak)
   - [Opening oldal (Hamarosan nyitunk)](#opening-oldal)
   - [Főoldal](#főoldal)
   - [Étlap](#étlap)
   - [Itallap](#itallap)
   - [Elérhetőség & Foglalás](#elérhetőség--foglalás)
5. [Admin rendszer](#admin-rendszer)
   - [Hitelesítés és jogosultságok](#hitelesítés-és-jogosultságok)
   - [Manager Dashboard](#manager-dashboard)
   - [Termék Katalógus & Tej Kiválasztó](#termék-katalógus--tej-kiválasztó)
   - [Kassza (POS Terminál)](#kassza-pos-terminál)
   - [Beosztás](#beosztás)
   - [Napi Tervező](#napi-tervező)
   - [Statisztika](#statisztika)
   - [Blokkolások (Jelenléti napló)](#blokkolások-jelenléti-napló)
   - [Blokkolás Kiosk (Raspberry Pi)](#blokkolás-kiosk-raspberry-pi)
   - [Leltár](#leltár)
   - [Lejáratok & FIFO](#lejáratok--fifo)
   - [Alkalmazottak](#alkalmazottak)
   - [Közlemények](#közlemények)
   - [Értékelések](#értékelések)
   - [Riportok & Üzemofal](#riportok--üzemofal)
   - [Felhasználókezelés](#felhasználókezelés)
6. [Biztonsági megoldások](#biztonsági-megoldások)
7. [Fájlstruktúra](#fájlstruktúra)
8. [Alapértelmezett felhasználók](#alapértelmezett-felhasználók)

---

## Technológiai stack

| Réteg | Technológia |
|---|---|
| Backend | PHP 8+ (PDO, prepared statements) |
| Adatbázis | MariaDB / MySQL (`utf8mb4_unicode_ci`) |
| Frontend CSS | Tailwind CSS 3 (CDN, `darkMode: 'class'`) |
| Frontend JS | Alpine.js v3, vanilla JS |
| Ikonok | Bootstrap Icons 1.11 |
| Grafikonok | Chart.js |
| E-mail | PHPMailer + Gmail SMTP (TLS, port 587) |
| PDF generálás | FPDF (saját Gabarito betűkészlettel) |
| Titkosítás | AES-256-CBC + HMAC-SHA256 érzékeny adatokhoz |
| Hardver | Raspberry Pi 5" (800×480) érintőképernyős kiosk |

---

## Telepítés és konfiguráció

### Követelmények

- Apache 2.4+ (`.htaccess` mod_rewrite szükséges)
- PHP 8.0+ (`pdo_mysql`, `openssl`, `mbstring`, `gd`)
- MariaDB 10.6+ / MySQL 8+
- XAMPP (helyi fejlesztéshez)

### Lépések

```bash
# 1. Klónozd a repót az Apache document root alá
git clone ... /var/www/html/kavezo
# vagy XAMPP esetén: C:\xampp\htdocs\kavezo

# 2. Hozd létre az adatbázist
mysql -u root < database/schema.sql

# 3. Töltsd fel az alap recepteket és készletadatokat
php database/populate_recipes.php

# 4. Konfiguráld a rendszert
nano includes/config.php
```

### Konfiguráció (`includes/config.php`)

```php
// Adatbázis
define('DB_HOST', 'localhost');
define('DB_NAME', 'coffeeshop');
define('DB_USER', 'root');
define('DB_PASS', '');

// E-mail (PHPMailer / Gmail SMTP)
define('MAIL_HOST',     'smtp.gmail.com');
define('MAIL_PORT',     587);
define('MAIL_USERNAME', 'sajat@gmail.com');
define('MAIL_PASSWORD', 'app-specific-password');

// Tervezési paraméterek
define('REVENUE_PER_PERSON', 400000); // Ft/fő — staffing kalkulációhoz
define('FOOD_COST_PERCENT',  35);     // Becsült árubeszerzési arány

// Érzékeny adatok titkosítása
define('SENSITIVE_ENCRYPTION_KEY', '...32 bájt hex...');
define('SENSITIVE_HMAC_KEY',       '...32 bájt hex...');
```

### Feltöltési könyvtárak

A rendszer az alábbi könyvtárakba tölt fel fájlokat (Apache-nak írási joga kell):

```
uploads/profile_pics/     — profilképek
uploads/announcements/    — közlemény képek
uploads/performers/       — előadó/személyzet képek
```

---

## Adatbázis felépítése

**13 alaptábla** a `coffeeshop` adatbázisban:

| Tábla | Leírás |
|---|---|
| `users` | Adminisztrátori fiókok (admin / vezeto / barista) |
| `employees` | Alkalmazott törzsadatok (külön a `users`-től) |
| `shifts` | Műszaktervek (kód, időpont, alkalmazott) |
| `daily_projections` | Napi tervezett forgalom, rendezvény, vendégszám |
| `pos_products` | Kassza termékek (ár, kategória, DRS, státusz) |
| `pos_recipes` | Recept → készlet leképezés (termék per alapanyag) |
| `pos_sales` | POS értékesítési tételek (terminál, fizetési mód) |
| `product_modifier_groups` | Kiegészítő csoportok (pl. „Tej típusa") |
| `product_modifiers` | Kiegészítő opciók (pl. „Zabtej +50 Ft") |
| `product_modifier_inventory` | Modifier → készlet leképezés |
| `inventory_items` | Készlettételek (helyszín, egység, minimum) |
| `stock_movements` | Készletmozgás napló (be/ki/átadás) |
| `posts` | Riportok és közlemények |
| `post_tags` | Riport címkék |
| `settings` | Rendszerbeállítások (pl. engedélyezett POS IP-k) |

---

## Nyilvános oldalak

### Opening oldal

**`opening/index.php`** — Előzetes nyitásjelző oldal a vendégek számára.

- Teljes képernyős, sötét háttér háttérképpel (`bg.png`) és gradiens overlay-jel
- **Visszaszámláló** az Alpine.js `countdown()` komponenssel: `2029-06-01 18:00:00` célpont
- 4 glassmorphic kártya: Nap / Óra / Perc / Másodperc, élő másodpercfrissítéssel
- Noxera logó + „Coffee & Bar" felirat lüktető animációval
- Helyszín (Budapest) és nyitási dátum
- Szöveg: „Egy hely, ahol a reggelek frissen pörkölt klasszikus és egyedi kávéval, az esték pedig válogatott italokkal és zenékkel teljesednek ki."

Önálló oldal, nem használja a sitewide `header.php`/`footer.php` sablonokat.

---

### Főoldal

**`index.php`** — A vendégek számára látható marketing oldal.

**Szekciók:**

1. **Hero** — teljes magasságú (`88vh`) kép-háttér, félig átlátszó overlay, főcím: *„Reggel kávé, este bár, mindig élmény."*, két CTA gomb (Étlapunk / Foglalás & Kapcsolat)
2. **About** — konyha + kávé + bár bemutató, 3 statisztika-kártya (20+ fogás, 5★ értékelés, 100% friss alapanyag), latte-art kép hover-grayscale effekttel
3. **Menü teaser** — Classic Espresso és Creamy Latte kártyák képpel + árral, harmadik kártya CTA az étlaphoz
4. **Nyitvatartás & Térkép** — cím (1052 Budapest, Deák Ferenc tér 1.), nyitvatartás (Kávézó 06–14, Bár 18–02), telefon, Google Maps embed szürkeárnyalatos szűrővel

Teljes dark mode támogatás (`dark:` Tailwind osztályok).

---

### Étlap

**`etlap.php`** — Teljes ételkínálat két műszakra bontva.

- **Reggeli fogások** (06:00–14:00): 5 étel — pl. avokádós pirítós, granola, tojásos fogások
- **Vacsorai fogások** (18:00–02:00): 13 étel — szezonális bisztró ételek
- Minden tétel: név, leírás, ár
- Dark mode kompatibilis

---

### Itallap

**`itallap.php`** — Teljes italkínálat kategóriánként.

- **Kávék** (8 tétel): Espresso, Americano, Cappuccino, Latte, Flat White, Macchiato, Cold Brew, Frappuccino
- **Teák & üdítők** (4 tétel)
- **Signature koktélok** (4 tétel): házi kreációk leírással
- **Kézműves sörök & borok** (4 tétel)

---

### Elérhetőség & Foglalás

**`elerhetoseg.php`** — Kapcsolatfelvételi és asztallefoglalási oldal.

- Cím, telefonszám, e-mail megjelenítése
- Foglalási/kapcsolatfelvételi űrlap, PHPMailer-rel küld e-mailt
- Google Maps embed

---

## Admin rendszer

Az adminisztrációs felület az `admin/` alkönyvtárban található. Minden oldal CSRF-védett, szerepköralapú hozzáféréssel.

### Hitelesítés és jogosultságok

**`admin/login.php`** — Bejelentkezési oldal felhasználónév + jelszó kombinációval.

Három szerepkör:

| Szerepkör | Hozzáférés |
|---|---|
| `admin` | Teljes hozzáférés minden modulhoz |
| `vezeto` | Manager-szintű funkciók (tervezés, statisztika, blokkolások szerkesztése) |
| `barista` | Csak olvasás, saját beosztás, közlemények |

Jelszólejárat: 90 nap (`PASSWORD_EXPIRY_DAYS`). Lejárat után kötelező jelszócsere.
Munkamenet: 8 óra inaktivitás után automatikus kijelentkezés (`SESSION_LIFETIME = 28800`).

Jelszó-visszaállítás: `admin/reset_password.php` — e-mailben kiküldött tokennel.

---

### Manager Dashboard

**`admin/manager.php`** — Vezető nézet a legfontosabb mutatókkal.

**KPI kártyák:**
- Nyitott riportok száma
- Mai bevétel (készpénz / kártya bontásban, POS adatokból)
- Aktív felhasználók száma
- Alacsony készletű tételek figyelmeztetése
- Rendszer verziószám
- Legutóbbi értékelések összefoglalója
- Következő tervezett napok száma (Napi Tervező összekötve)

Gyorslink-csempék az összes admin modulhoz.

---

### Termék Katalógus & Tej Kiválasztó

**`admin/katalogus.php`** — A kassza teljes kínálatának kezelése. Három fül:

#### 1. Termékek

- Termékek listája táblázatban, kategória-színkódokkal
- Kategóriák: Kávé · Frappuccino · Koktél · Alkohol · Üdítő · Étel · Reggeli · Vacsora · Egyéb
- Minden termékhez: név, ár, kategória, aktív/inaktív státusz (valós idejű toggle), DRS palackdíj (+50 Ft)
- Termék hozzáadása / szerkesztése / törlése (törlés egyszerre törli a hozzá tartozó receptet is, tranzakcióban)
- Keresés termék névre
- AJAX státusztoggle — az oldal nem tölt újra

#### 2. Kedvezmények

- Kedvezmények kezelése: százalékos (%) vagy fix összegű (Ft)
- Aktív/inaktív státusz toggle
- A kassza automatikusan alkalmazza a beállított kedvezményeket

#### 3. Kiegészítő Sablonok (Tej Kiválasztó)

Ez a modul teszi lehetővé, hogy az italokhoz (pl. tejkávékhoz) a kasszán kötelező vagy opcionális kiegészítőket lehessen választani.

**Működési elv:**

1. **Sablon létrehozása** — pl. „Tej típusa" (kötelező) vagy „Méret" (opcionális)
2. **Opciók hozzáadása** a sablonhoz — pl.:
   - Tehéntej 1,5% — +0 Ft
   - Tehéntej 3,5% — +0 Ft
   - Zabtej — +50 Ft
   - Mandulatej — +80 Ft
   - Laktózmentes tej — +30 Ft
3. **Sablon hozzárendelése termékhez** — pl. a „Cappuccino" kap egy „Tej típusa (kötelező)" csoportot
4. **Kasszán megjelenés** — rendeléskor a POS automatikusan felkínálja a kiegészítőket; kötelező esetén a rendelés nem zárható le választás nélkül
5. **Készletlevonás** — minden tej-modifier be van kötve a megfelelő készlettételhez (`product_modifier_inventory` tábla), így a zabtej kiválasztása automatikusan vonja le a zabtej-készletet

**Adatmodellje:**

```
product_modifier_groups   → csoportok termékhez rendelve (required 0/1)
  └── product_modifiers   → opciók, extra_price-szal
        └── product_modifier_inventory → készlet-leképezés (item_id, qty)
```

---

### Kassza (POS Terminál)

**`admin/kassza.php`** — Érintőképernyő-optimalizált, többterminális értékesítési rendszer.

**Biztonsági hozzáférés:**
- IP-cím fehérlistás védelem — csak engedélyezett terminálok nyithatják meg (`settings` tábla `pos_terminals` kulcsa)
- 403-as visszautasítás más IP-ről
- Különálló 24 órás munkamenet (nem keveredik az admin sessionnel)
- Csúszóablakos cookie-megújítás

**Három terminál:**
- **Pult** (Counter) — bár/kávé pult
- **Terasz** (Terrace) — terasz
- **Bár** (Bar) — bárpult

**POS funkciók:**
- Alkalmazott bejelentkezés legördülő listából
- Manager PIN megerősítés érzékeny műveleteknél
- Termékkategóriák szűrése
- Kosár kezelése (mennyiség, törlés, módosítás)
- **Tej kiválasztó / Kiegészítők** — rendeléskor modal ablakban jelenik meg
- Kedvezmény alkalmazása (% vagy fix Ft)
- QR-kód beolvasó (html5-qrcode könyvtárral)
- Fizetési módok: készpénz / bankkártya
- DRS palackdíj automatikus kezelése
- PDF blokk generálása (FPDF, Gabarito betűkészlet)
- Terminál statisztikák (napi forgalom, tételszám)

**API backend:** `admin/api_pos.php` — JSON API a POS UI-hoz, kezeli a termékeket, rendeléseket, modifier csoportokat, manager PIN ellenőrzést.

**Kasszafelügyelet:** `admin/kassza_manager.php` — Manager view a nyitott rendelésekre, napi forgalomra.

---

### Beosztás

**`admin/beosztas.php`** — Heti és napi műszaktervező.

**Nézetek:**
- **Heti nézet** — teljes hét, minden alkalmazott sorban, napok oszlopban; cellák kattinthatók a szerkesztéshez
- **Napi nézet** — részletes napilap: műszakkódok, tervezett vendégszám, esemény neve (Tervező összekötve)

**Műszakkódok** (`SHIFT_CODES` konstans, `includes/config.php`):

| Kód | Megnevezés | Szín |
|---|---|---|
| MV | Műszakvezető | Piros |
| K | Koordinátor | Narancs |
| PN | Pultos | Zöld |
| É | Éjszakai | Barna |
| SZ | Szabadnap | Világoszöld |
| BT | Betegség | Szürke |
| TK | Továbbképzés | Lila |
| HO | Home Office | Kék |

**Automatikus staffing számítás:**
- `REVENUE_PER_PERSON = 400 000 Ft` — ennyi tervezett forgalomra jut 1 személyzeti fő
- A Napi Tervező adatai alapján kalkulálja az ajánlott létszámot

**Tervező összekötés:**
- A heti nézet oszlopfejlécein jelenik meg az adott napra rögzített esemény neve
- A napi nézetben: esemény neve, megjegyzés, Reggeli/Esti vendégszám (két műszak: 06:00–14:00 és 18:00–02:00)

---

### Napi Tervező

**`admin/tervezo.php`** — Naptáralapú, napi szintű tervezési modul.

**Funkciók:**
- Havi naptár nézet, hónap-navigációval (előző/következő)
- Minden nap cellája kattintható — megnyitja a szerkesztőt
- Meglévő adattal rendelkező napok vizuálisan jelölve
- Múltba és jövőbe is lehet rögzíteni

**Szerkeszthető mezők:**
- **Esemény neve** — pl. „Születésnapi party", „Céges vacsora"
- **Esemény megjegyzések** — részletes leírás
- **Reggeli vendégszám** (06:00–14:00 műszak)
- **Esti vendégszám** (18:00–02:00 műszak)
- **Átlagos terítékár** (Ft/fő, alapértelmezett: 4 500 Ft)

**Automatikus kalkuláció:**
- Tervezett forgalom = (Reggeli vendégek + Esti vendégek) × Átlagos terítékár
- Élő, valós idejű frissítés a szerkesztőben

**Adattárolás:** `daily_projections` tábla, `ON DUPLICATE KEY UPDATE` — egy naphoz mindig egy rekord.

**Integráció más modulokkal:**
- Beosztás: esemény jelzése fejlécben + vendégszám napi nézetben
- Statisztika: tervezett bevétel megjelenítése a forgalmi grafikonokon

---

### Statisztika

**`admin/statisztika.php`** — Forgalmi elemzés két nézetben.

#### Napi nézet (`?view=daily&date=YYYY-MM-DD`)

- Dátumválasztó
- Esemény-banner (ha van rendezvény az adott napra a Tervező alapján)
- **5 KPI kártya:**
  - Összes bevétel (Ft)
  - Tervezett bevétel (Tervező adatából)
  - Eltérés (tény vs. terv)
  - Készpénzes bevétel
  - Bankkártyás bevétel
- **Óránkénti forgalmi diagram** (Chart.js vonaldiagram) — 24 óra, POS értékesítés alapján

#### Havi nézet (`?view=monthly&y=YYYY&m=M`)

- Hónap-navigátor (előző/következő/aktuális hónap)
- **4 KPI kártya:**
  - Havi összbevétel
  - Havi össz-tervezett bevétel
  - Havi eltérés
  - Átlagos napi bevétel (aktív napokra vetítve)
- **Sávdiagram** (Chart.js) — tervezett (narancs körvonalas) vs. tényleges (zöld) bevétel naponként
- **Nap-szintű táblázat:**
  - Dátum (link a napi nézethez)
  - Esemény neve
  - Tervezett forgalom
  - POS bevétel
  - Tényleges bevétel
  - Eltérés (Ft)
  - Eltérés százaléka (színkódolt badge: zöld/piros)
  - Vendégszám összesen
  - Összesítő sor (lábléc)

---

### Blokkolások (Jelenléti napló)

**`admin/blokkolasok.php`** — Munkaidő-nyilvántartás kezelése. Vezető jogosultság szükséges.

**Szűrési lehetőségek:**
- Szabad szöveges keresés (alkalmazott neve, megjegyzés)
- Dátum szerinti szűrés
- Típus szerinti szűrés (BE/KI)

**Táblázat oszlopok:**
- Azonosító, alkalmazott neve, dátum/idő, típus (BE/KI), megjegyzés

**Szerkesztés (Vezető jog):**

Ha egy alkalmazott hibás időpontban blokkolt (pl. 19:56 helyett 20:00-kor kellett volna), a vezető javíthatja:

- Ceruza ikon gombra kattintva megnyílik a szerkesztőmodal
- Módosítható: dátum és pontos idő (`datetime-local` beviteli mező, percig), típus (BE/KI), megjegyzés
- Mentés POST kéréssel, szerver oldali regex validációval
- Törlés: megerősítő `confirm()` dialóggal

**Összesítő modal:**
- Szemmel ikon: alkalmazottanként napi összefoglalót jelenít meg (összes óra, BE/KI párosítás)

**CSV export** a szűrt eredményekre.

---

### Blokkolás Kiosk (Raspberry Pi)

**`blokkolas/blokkolas_kiosk.php`** — Dedikált érintőképernyős munkaóranyilvántartó terminál.

**Hardver célplatform:** Raspberry Pi 5" (800×480 pixel) érintőképernyő.

**Funkciók:**
- Alkalmazott neve vagy azonosítója alapján keresés / QR-kód beolvasás
- BE (check-in) és KI (check-out) rögzítés egy gombnyomással
- Élő óra — **percig megjelenítve** (HH:MM), másodperc nélkül
- Utolsó blokkolt esemény dátuma és típusa (percig pontossággal)
- Egyszerűsített, nagy felületi elemek érintőhasználatra optimalizálva
- API (`blokkolas/api.php`) JSON alapon kommunikál a kiosk UI-jával

---

### Leltár

**`admin/leltar.php`** — Készletszintek kezelése.

**Két helyszín:**
- `raktar` — Raktár (alapkészlet)
- `uzleti` — Üzleti (pulton lévő készlet)

**Funkciók:**
- Készlettételek listája kategóriánként (Kávé, Tej, Szirup, Alkohol, Kellék stb.)
- Mennyiség, egység (kg, l, db stb.), minimumszint megadása
- Közel lejáró / minimumszint alatti tételek vizuális figyelmeztetése
- Tételek között átadás (raktár → üzleti)
- Készletmozgás rögzítése (be / ki)

---

### Lejáratok & FIFO

**`admin/lejaratok.php`** — Élelmiszer-biztonsági lejárat-nyilvántartás.

- Csomagok lejárati dátumának rögzítése
- FIFO (First In, First Out) elv alapján sorolt lista
- Közeledő lejáratú tételek kiemelése (piros/sárga jelzés)
- Lejárt tételek kezelése

**`admin/leltar_elozmenyek.php`** — Készletmozgás napló: minden be-, ki- és átadási esemény időrendjben, szűrhető nézetben.

---

### Alkalmazottak

**`admin/alkalmazottak.php`** — Személyzeti törzsadatok kezelése.

- Alkalmazott felvétele / szerkesztése / deaktiválása
- Adatok: név, azonosítószám, e-mail, órabér, profilkép
- Érzékeny adatok (pl. órabér) AES-256-CBC + HMAC-SHA256 titkosítással tárolva az adatbázisban

---

### Közlemények

**`admin/kozlemenyek.php`** — Belső hírek, bejelentések kezelése.

- Közlemény létrehozása / szerkesztése / törlése
- Csatolmányok (képek) feltöltése (`uploads/announcements/`)
- Minden belépett alkalmazott látja a friss közleményeket

---

### Értékelések

**`admin/ertekelesek.php`** — Személyzeti teljesítményértékelés.

- Értékelések rögzítése (vezető → alkalmazott)
- Értékelési napló megtekintése

**`review/index.php`** — Értékelés beküldő felület (webes link megosztható).

---

### Riportok & Üzemofal

**`admin/report.php`** — Riport generálás és kezelés.

- Napi/heti riportok írása
- Riportok tag-eléssel jelölése
- Nyitott / lezárt státusz

**`admin/uzemofal.php`** — Operatív „faliújság": nyitott riportok és közlemények egy nézeten.

---

### Felhasználókezelés

**`admin/felhasznalok.php`** — Admin fiókok kezelése (csak `admin` szerepkör).

- Új felhasználó létrehozása (admin / vezeto / barista)
- Jelszó visszaállítás
- Szerepkör módosítása
- Fiók aktiválása / deaktiválása
- Profilkép kezelése (`uploads/profile_pics/`)
- 2FA titkos kulcs kezelése (infrastruktúra kész, TOTP)

---

## Biztonsági megoldások

| Megoldás | Implementáció |
|---|---|
| CSRF védelem | `csrf_token()` / `verify_csrf()` minden POST kérésnél |
| Jelszóhash | `password_hash()` bcrypt (cost 12) |
| SQL injection | Kizárólag PDO prepared statements |
| XSS | `htmlspecialchars()` minden kimenetiértéknél |
| Szerepkörvédelem | `Auth::requireAdmin()` / `Auth::requireVezeto()` / `Auth::requireLogin()` |
| Munkamenet | 8 órás inaktivitási timeout, httponly + samesite cookie |
| POS IP-whitelist | `settings` tábla alapján dinamikus IP-szűrés |
| Adattitkosítás | AES-256-CBC + HMAC-SHA256 érzékeny mezőkre |
| JSON HTML-escape | `JSON_HEX_APOS\|JSON_HEX_QUOT` minden PHP-ban generált inline JSON-nál |
| Jelszólejárat | 90 napos kötelező megújítás |
| E-mail token | 64 karakter random token, max. 1 órás érvényesség jelszó-visszaállításhoz |

---

## Fájlstruktúra

```
kavezo/
├── index.php                    # Főoldal (vendégek)
├── etlap.php                    # Étlap
├── itallap.php                  # Itallap
├── elerhetoseg.php              # Kapcsolat & foglalás
├── opening/
│   ├── index.php                # Coming soon visszaszámlálós oldal
│   └── bg.png                   # Háttérkép
├── admin/
│   ├── login.php / logout.php
│   ├── index.php                # Admin főoldal
│   ├── manager.php              # Manager dashboard
│   ├── katalogus.php            # Termékek, kedvezmények, tej kiválasztó sablonok
│   ├── kassza.php               # POS terminál UI
│   ├── kassza_manager.php       # POS menedzsment
│   ├── api_pos.php              # POS JSON API
│   ├── api_expiry.php           # Lejárat API
│   ├── beosztas.php             # Beosztás tervező
│   ├── tervezo.php              # Napi tervező (forgalom, vendégszám, esemény)
│   ├── statisztika.php          # Statisztika (napi / havi)
│   ├── blokkolasok.php          # Jelenléti napló (szerkesztéssel)
│   ├── leltar.php               # Leltár
│   ├── leltar_elozmenyek.php    # Készletmozgás napló
│   ├── lejaratok.php            # Lejárat & FIFO
│   ├── alkalmazottak.php        # Alkalmazott törzsadatok
│   ├── felhasznalok.php         # Admin felhasználókezelés
│   ├── kozlemenyek.php          # Belső közlemények
│   ├── ertekelesek.php          # Teljesítményértékelések
│   ├── report.php               # Riportok
│   ├── uzemofal.php             # Operatív faliújság
│   ├── reset_password.php       # Jelszó-visszaállítás
│   └── sidebar.php              # Navigációs oldalsáv
├── blokkolas/
│   ├── blokkolas_kiosk.php      # Raspberry Pi kiosk UI
│   ├── api.php                  # Kiosk API
│   ├── config.php               # Kiosk konfiguráció
│   └── install.sql              # Kiosk telepítő SQL
├── includes/
│   ├── config.php               # Rendszerkonfiguráció és konstansok
│   ├── db.php                   # PDO singleton
│   ├── auth.php                 # Hitelesítés és jogosultságok
│   ├── functions.php            # Globális segédfüggvények (CSRF, flash, format)
│   ├── mailer.php               # PHPMailer wrapper
│   ├── encryption.php           # AES-256-CBC + HMAC titkosítás
│   ├── header.php               # Oldalsablon fejléc
│   ├── footer.php               # Oldalsablon lábléc
│   ├── api_theme.php            # Téma API
│   ├── pdf_receipt.php          # PDF blokk generátor (FPDF)
│   └── fpdf/                    # FPDF könyvtár (Gabarito betűkészlet)
├── database/
│   ├── schema.sql               # Teljes adatbázis séma (seed adatokkal)
│   └── populate_recipes.php     # Recept + készlet seed szkript
├── assets/
│   ├── css/main.css             # Tailwind build + egyedi stílusok
│   ├── js/main.js               # Alpine.js komponensek + vanilla JS
│   └── img/                     # Hero, espresso, latte, bar képek
├── uploads/
│   ├── profile_pics/
│   ├── announcements/
│   └── performers/
└── review/
    └── index.php                # Értékelés beküldő
```

---

## Alapértelmezett felhasználók

Az adatbázis seed script (`database/schema.sql`) az alábbi fiókokat hozza létre. **Élesítés előtt minden jelszót cserélj le.**

| Felhasználónév | Szerepkör | Név | Jelszó |
|---|---|---|---|
| `admin` | admin | Kovács Adél | `admin123` |
| `vezeto` | vezeto | Szabó Bence | `admin123` |
| `barista1` | barista | Nagy Dóra | `admin123` |
| `barista2` | barista | Tóth Péter | `admin123` |
| `barista3` | barista | Varga Anikó | `admin123` |

---

*Noxera Bistro Management System — belső dokumentáció*
