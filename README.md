# Noxera Bistro — Vendéglátóipari Menedzsment Rendszer

> Budapest · Kávézó + Étterem + Bár · 2028 óta

Noxera egy teljes körű, PHP-alapú belső menedzsment platform kávézó, étterem és bár üzemeltetéséhez. Lefedi a nyilvános megjelenéstől az érintőképernyős kassza-terminálokon és műszaktervezésen át a bérszámfejtés előkészítéséig az összes munkafolyamatot.

**Stack:** PHP 8+, MariaDB, Tailwind CSS 3, Alpine.js v3, Chart.js, PHPMailer, FPDF

---

## Nyilvános oldalak

### Opening oldal (`opening/index.php`)

Előzetes nyitásjelző oldal. Teljes képernyős sötét háttérrel, Alpine.js visszaszámlálóval a nyitás dátumáig (2029-06-01 18:00). Négy glassmorphic kártya mutatja a hátralévő napokat, órákat, perceket és másodperceket.

### Főoldal (`index.php`)

A vendégek számára látható marketing oldal. Hero szekció főcímmel és CTA gombokkal, bemutatkozó szekció a bistró koncepcióval, menü-teaser kártyákkal (espresso, latte), valamint nyitvatartási és helyszín szekció beágyazott Google Maps-szel.

### Étlap & Itallap (`etlap.php`, `itallap.php`)

Teljes kínálat két oldalon. Az étlapon reggeli fogások (06:00–14:00) és vacsorai fogások (18:00–02:00), az itallapon kávék, teák, signature koktélok és kézműves sörök/borok.

### Elérhetőség (`elerhetoseg.php`)

Cím, telefonszám, e-mail és asztallefoglalási / kapcsolatfelvételi űrlap, amely PHPMailer-rel küld értesítőt.

---

## Admin rendszer

Minden adminisztrációs oldal CSRF-védett és szerepköralapú. Három szint: **admin**, **vezeto** (manager), **barista**.

### Manager Dashboard (`admin/manager.php`)

Áttekintő felület a legfontosabb mutatókkal: mai bevétel készpénz/kártya bontásban, nyitott riportok, aktív felhasználók, alacsony készlet figyelmeztetés, legutóbbi értékelések összefoglalója, következő tervezett napok száma. Gyorslinkek az összes modulhoz.

### Termék Katalógus (`admin/katalogus.php`)

A kassza kínálatának kezelése három fülön:

**Termékek** — termékek listája kategória-színkódokkal (Kávé, Koktél, Étel, Reggeli, Vacsora stb.), ár, aktív/inaktív státusz valós idejű AJAX toggle-lal, DRS palackdíj (+50 Ft) jelöléssel. Termék hozzáadása, szerkesztése, törlése.

**Kedvezmények** — százalékos (%) vagy fix összegű (Ft) kedvezmények kezelése, amelyeket a kassza automatikusan alkalmaz.

**Kiegészítő Sablonok (Tej kiválasztó)** — globális modifier sablonok definiálása és termékekhez rendelése. Például a „Tej típusa" sablon kötelező kiegészítőként rendelhető tejkávékhoz: Tehéntej 1,5%, Tehéntej 3,5%, Zabtej (+50 Ft), Mandulatej (+80 Ft), Laktózmentes tej (+30 Ft). Kasszán rendeléskor modal ablakban jelenik meg a választó; kötelező modifier esetén a rendelés nem zárható le választás nélkül. Minden modifier opció be van kötve a megfelelő készlettételhez, így a kiválasztott tej típusa automatikusan csökkenti a leltárkészletet.

### Kassza POS (`admin/kassza.php`)

Érintőképernyő-optimalizált, többterminális értékesítési rendszer. IP-cím fehérlista védi a hozzáférést — csak az adatbázisban engedélyezett terminálokon nyílik meg, más IP-ről 403-as hibaoldalt ad. Három terminál: Pult, Terasz, Bár, mindegyik különálló 24 órás munkamenettel.

Funkciói: alkalmazott bejelentkezés legördülőből, manager PIN megerősítés érzékeny műveleteknél, termékkategória-szűrés, kosárkezelés, tej kiválasztó / kiegészítők, kedvezmény alkalmazás, QR-kód beolvasó, készpénz/kártya fizetés, DRS palackdíj, PDF blokk generálás Gabarito betűkészlettel.

### Beosztás (`admin/beosztas.php`)

Heti és napi műszaktervező. A heti nézetben minden alkalmazott sora kattintható cellákat tartalmaz napokra bontva. A napi nézetben megjelenik az adott napra rögzített esemény neve, megjegyzés, és a két műszak (06:00–14:00 reggeli, 18:00–02:00 esti) tervezett vendégszáma.

Műszakkódok: MV (Műszakvezető), K (Koordinátor), PN (Pultos), É (Éjszakai), SZ (Szabadnap), BT (Betegség), TK (Továbbképzés), HO (Home Office). Az ajánlott létszámot a rendszer automatikusan számolja a Napi Tervező adatai alapján (400 000 Ft tervezett forgalomra 1 személyzeti fő).

### Napi Tervező (`admin/tervezo.php`)

Naptáralapú tervezési modul. Havi nézet hónap-navigációval; minden nap cellája kattintható, a meglévő adattal rendelkező napok vizuálisan jelölve. Rögzíthető: esemény neve, megjegyzés, reggeli és esti vendégszám, átlagos terítékár. A tervezett forgalom automatikusan kalkulálódik: `(reggeli vendégek + esti vendégek) × terítékár`. Az adatok megjelennek a Beosztásban (eseményjelzés) és a Statisztikában (tervezett bevétel).

### Statisztika (`admin/statisztika.php`)

**Napi nézet** — dátumválasztóval, esemény-bannerrel (ha van rendezvény), 5 KPI kártyával (összes bevétel, tervezett, eltérés, készpénz, kártya), és óránkénti forgalmi vonaldiagrammal.

**Havi nézet** — hónap-navigátorral, 4 KPI kártyával (havi összbevétel, össz-tervezett, eltérés, átlag/nap), tervezett vs. tényleges sávdiagrammal naponként, és nap-szintű táblázattal (dátum, esemény, tervezett, POS, tényleges, eltérés Ft-ban és %-ban, vendégszám). Minden nap sor linkkel nyílik a napi nézetbe.

### Blokkolások (`admin/blokkolasok.php`)

Munkaidő-nyilvántartás kezelése. Szűrhető lista szabad szöveg, dátum és típus (BE/KI) alapján. Ha egy alkalmazott hibás időpontban blokkolt, a vezető javíthatja: ceruza ikonra kattintva szerkesztőmodal nyílik, ahol a dátum és pontos idő (percig), a típus és a megjegyzés módosítható. Bejegyzések törölhetők megerősítő dialóggal. Alkalmazottanként összesítő modal (összes óra, BE/KI párosítás). CSV export.

### Blokkolás Kiosk (`blokkolas/blokkolas_kiosk.php`)

Dedikált érintőképernyős jelenléti terminál Raspberry Pi 5" (800×480) kijelzőre optimalizálva. Alkalmazott keresése névvel vagy QR-kóddal, BE/KI rögzítés egy gombnyomással. Az óra percig pontossággal jelenik meg (másodperc nélkül), az utolsó esemény dátuma szintén percig látható.

### Leltár (`admin/leltar.php`, `admin/leltar_elozmenyek.php`)

Készletszintek kezelése két helyszínre: raktár és üzleti (pulton lévő). Kategóriánkénti lista mennyiséggel, egységgel, minimumszinttel; minimumszint alatti tételek vizuális figyelmeztetéssel. Átadás raktár → üzleti irányban. Minden mozgás naplózva az előzmények oldalon.

### Lejáratok & FIFO (`admin/lejaratok.php`)

Élelmiszer-biztonsági lejárat-nyilvántartás FIFO elv alapján. Csomagok lejárati dátuma rögzíthető, a közeledő lejáratú tételek piros/sárga jelzéssel kiemelve.

### Alkalmazottak (`admin/alkalmazottak.php`)

Személyzeti törzsadatok: név, azonosítószám, e-mail, órabér, profilkép, aktív státusz. Az órabér AES-256-CBC + HMAC-SHA256 titkosítással tárolódik az adatbázisban.

### Közlemények, Értékelések, Riportok

**Közlemények** (`kozlemenyek.php`) — belső hírek, bejelentések képmelléklettel, minden belépett alkalmazott látja.

**Értékelések** (`ertekelesek.php`) — személyzeti teljesítményértékelés rögzítése és naplója. Külső értékelőlink: `review/index.php`.

**Riportok & Üzenofal** (`report.php`, `uzenofal.php`) — napi/heti riportok tag-eléssel, nyitott/lezárt státusszal; az üzemofal egy nézetben mutatja a nyitott riportokat és friss közleményeket.

### Felhasználókezelés (`admin/felhasznalok.php`)

Admin fiókok kezelése (csak `admin` szerepkör): új fiók, jelszócsere, szerepkör módosítás, aktiválás/deaktiválás, profilkép. Jelszó-visszaállítás e-mailes tokennel (`reset_password.php`). Jelszólejárat 90 naponként kötelező.

---

## Adatbázis

`coffeeshop` adatbázis, 13+ tábla: `users`, `employees`, `shifts`, `daily_projections`, `pos_products`, `pos_recipes`, `pos_sales`, `product_modifier_groups`, `product_modifiers`, `product_modifier_inventory`, `inventory_items`, `stock_movements`, `posts`, `settings`.

---

## Biztonság

CSRF tokenek minden POST kérésnél · PDO prepared statements · bcrypt jelszóhash (cost 12) · szerepköralapú hozzáférés-vezérlés · POS IP-whitelist · AES-256-CBC + HMAC-SHA256 érzékeny adatokhoz · 8 órás munkamenet timeout · `JSON_HEX_APOS|JSON_HEX_QUOT` minden PHP-generált inline JSON-nál.

---

*Noxera Bistro Management System — belső dokumentáció*
