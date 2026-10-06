# Instrukcje dla Copilota: portfolio fotograficzne

Ten plik opisuje projekt i zasady pracy. Czytaj go przed każdą zmianą. Jeśli polecenie jest sprzeczne z tym plikiem, zapytaj, zamiast zgadywać.

## 1. Cel projektu

**Zlecenie:** landing page z portfolio dla fotografa. Strona główna + podstrony kategorii: **Portret**, **Ślubna**, **Podróże i krajobraz**. Stack: **Astro + React + Tailwind**. Najważniejsza jest **łatwa aktualizacja**: dodawanie nowych postów (historii) w każdej kategorii, np. w Portrecie: Anna, Maria, Ola itd., ma polegać na dodaniu folderu ze zdjęciami i jednego pliku tekstowego, **bez edycji kodu stron**.

Prosta, szybka, statyczna strona z portfolio fotograficznym. Ma wyglądać profesjonalnie i działać na telefonie. Zdjęcia są najważniejsze, interfejs ma być cichy i minimalistyczny. Brak backendu.

- Hosting: **GitHub Pages** (strona statyczna).
- Adres docelowy: `https://jacobjs7.github.io` (podmień na właściwy w `astro.config.mjs`).
- Język strony: **polski** (`<html lang="pl">`).

## 2. Stack (nie zmieniaj bez pytania)

- **Astro** (najnowsza stabilna wersja), TypeScript w trybie **strict**.
- **React** (`@astrojs/react`) **tylko** dla elementów interaktywnych (lightbox, ewentualnie filtry). Strony, layouty, siatki i SEO piszemy w plikach `.astro`.
- Obrazy: wbudowany `astro:assets` (`<Image />` / `<Picture />`).
- Czcionki: lokalnie przez `@fontsource` (bez Google Fonts z CDN, ze względu na RODO).
- Sitemap: `@astrojs/sitemap`.
- **Nie dodawaj nowych zależności bez wyraźnej zgody.** Jeśli uważasz, że jest potrzebna, zapytaj i uzasadnij.
- **Tailwind CSS** do stylowania, zainstalowany **zgodnie z aktualną dokumentacją Astro** (https://docs.astro.build/en/guides/styling/#tailwind). Nowsze wersje używają wtyczki Vite `@tailwindcss/vite` i `@import "tailwindcss";` w `global.css`, a nie starej integracji `@astrojs/tailwind` i `tailwind.config.js`. Nie zgaduj, sprawdź dokumentację.
- Motyw (kolory, fonty, odstępy) definiuj w `global.css` w bloku `@theme`, nie wpisuj kolorów i fontów na sztywno w klasach.
- Powtarzalne układy (np. siatka sesji) wynoś do komponentów Astro zamiast kopiować długie ciągi klas. Nie używaj `@apply` na masową skalę.
- Poza Tailwindem **żadnych bibliotek UI** (shadcn, MUI itp.).

## 3. Struktura projektu

```
src/
├── assets/
│   └── <kategoria>/<sesja>/01.jpg, 02.jpg ...   # zdjęcia sesji
├── content/
│   └── sesje/<sesja>.md                          # metadane sesji
├── content.config.ts                             # definicja kolekcji
├── data/
│   └── kategorie.ts                              # ŹRÓDŁO PRAWDY dla kategorii
├── components/
│   ├── Header.astro
│   ├── Footer.astro
│   ├── Hero.astro
│   ├── OMnie.astro                               # sekcja na stronie głównej
│   ├── Kontakt.astro                             # sekcja na stronie głównej
│   ├── SesjaCard.astro                           # kafelek sesji
│   └── Lightbox.tsx                              # React, client:visible
├── layouts/
│   └── Layout.astro                              # head, SEO, header, footer
├── styles/
│   └── global.css
└── pages/
    ├── index.astro
    ├── [kategoria]/index.astro                   # siatka sesji w kategorii
    ├── [kategoria]/[sesja].astro                 # galeria jednej sesji
    └── 404.astro
public/
├── favicon.svg
├── og-image.jpg                                  # 1200x630, domyślny podgląd linku
└── robots.txt
```

## 4. Kategorie (bardzo ważne)

Kategorie są **danymi**, nie kodem. Definiuje je jedna lista w `src/data/kategorie.ts`:

```ts
export const kategorie = [
  { slug: "portret", nazwa: "Portret", opis: "Sesje indywidualne i rodzinne", widoczna: true },
  { slug: "slubna", nazwa: "Ślubna", opis: "Historie ślubne", widoczna: true },
  { slug: "podroze-krajobraz", nazwa: "Podróże i krajobraz", opis: "Miejsca i światło", widoczna: true },
] as const;
```

Zasady:

- **Nigdy nie wpisuj nazw ani slugów kategorii na sztywno** w komponentach, menu, stronie głównej ani trasach. Zawsze importuj z `kategorie.ts`.
- Menu, kafelki kategorii na stronie głównej i trasy `[kategoria]` generuj z tej listy.
- Kategorie z `widoczna: false` **nie pojawiają się** w menu, na stronie głównej ani w sitemap i nie generują stron (`getStaticPaths` je pomija).
- Dodanie nowej kategorii (np. psy) ma wymagać tylko: wpisu w `kategorie.ts`, folderu ze zdjęciami i plików sesji. **Bez nowych szablonów.**
- Pole `kategoria` w sesji musi być walidowane względem slugów z `kategorie.ts`.

## 5. Kolekcja `sesje`

Plik: `src/content.config.ts`. Używaj **aktualnego API content collections** (Astro 5+: `defineCollection`, loader `glob()`, `z` z `astro/zod`). Starsze API (`src/content/config.ts`, `slug` w kolekcji) może być nieaktualne, **sprawdź dokumentację: https://docs.astro.build/en/guides/content-collections/** i nie zgaduj składni.

Schemat (Zod):

| Pole | Typ | Opis |
|---|---|---|
| `tytul` | string | np. "Alicja i Patryk" |
| `podtytul` | string, opcjonalne | np. "Dolomity", rozróżnia sesje o tym samym tytule |
| `kategoria` | enum ze slugów z `kategorie.ts` | |
| `data` | date | sesja i rok pod tytułem biorą się stąd |
| `okladka` | `image()` | zdjęcie okładki (relatywna ścieżka do `src/assets`) |
| `okladkaPozycja` | enum `top`/`center`/`bottom`, domyślnie `center` | `object-position` dla okładki |
| `zdjecia` | tablica `image()`, **opcjonalne** | jeśli podane: kolejność = kolejność w galerii. Jeśli brak: galeria bierze wszystkie zdjęcia z folderu sesji, posortowane po nazwie pliku (`import.meta.glob`) |
| `alt` | string, opcjonalne | bazowy opis zdjęć, jeśli brak indywidualnych |
| `wyrozniona` | boolean, domyślnie false | możliwość pokazania na stronie głównej |
| `opis` | string, opcjonalne | krótki opis (też `meta description`) |

Przykład `src/content/sesje/alicja-i-patryk-dolomity.md`:

```md
---
tytul: "Alicja i Patryk"
podtytul: "Dolomity"
kategoria: "slubna"
data: 2026-06-20
okladka: "../../assets/slubna/alicja-i-patryk-dolomity/01.jpg"
zdjecia:
  - "../../assets/slubna/alicja-i-patryk-dolomity/01.jpg"
  - "../../assets/slubna/alicja-i-patryk-dolomity/02.jpg"
opis: "Plener ślubny w Dolomitach."
---
Krótka historia sesji: miejsce, klimat, okoliczności.
```

**Slug URL** pochodzi z nazwy pliku `.md`. Dwie sesje z tym samym tytułem muszą mieć różne nazwy plików (np. `alicja-i-patryk-dolomity.md` i `alicja-i-patryk-slub.md`).

## 5a. Dodawanie i aktualizacja treści (priorytet projektu)

Właściciel strony ma dodawać treści sam, bez ruszania kodu stron. Cały projekt ma być zaprojektowany pod ten scenariusz.

**Nowa historia/post w istniejącej kategorii** (np. Portret → Maria) wymaga wyłącznie:

1. Wrzucenia zdjęć do `src/assets/portret/maria/` (nazwy `01.jpg`, `02.jpg`, ...).
2. Utworzenia pliku `src/content/sesje/maria.md` z metadanymi (patrz sekcja 5).
3. `git push`: GitHub Actions buduje i publikuje stronę.

Strona kategorii, strona sesji, okładka w siatce, sitemap i `og:image` mają powstać **automatycznie**. Jeśli do dodania historii trzeba edytować jakikolwiek plik `.astro` lub `.tsx`, to znaczy, że projekt jest źle zbudowany. Popraw to.

**Nowa kategoria** (np. psy): wpis w `src/data/kategorie.ts` + folder ze zdjęciami + pliki sesji. Bez nowych szablonów.

**Zasady, które to zapewniają:**

- Treść (tytuły, daty, opisy, lista zdjęć) mieszka w plikach `.md` i `kategorie.ts`, a nie w komponentach.
- Siatka sesji i galeria zawsze renderują się z kolekcji, nigdy z ręcznie wypisanych list.
- Wygodny dodatek (po zapytaniu o zgodę): skrypt Node bez zależności `npm run nowa-sesja -- portret maria`, który tworzy folder `src/assets/portret/maria/` i szablon `maria.md` z uzupełnionym frontmatterem.
- W `README.md` opisz w kilku krokach, jak dodać historię i kategorię (instrukcja dla właściciela, nie dla programisty).

## 6. Strony

### `index.astro` (strona główna)
- Hero: jedno mocne zdjęcie na całą wysokość ekranu z imieniem/nazwą fotografa i krótkim hasłem.
- Pod spodem kafelki widocznych kategorii (z `kategorie.ts`), każdy z jednym zdjęciem reprezentatywnym (np. okładka najnowszej sesji z danej kategorii) i nazwą.
- Opcjonalnie sekcja wyróżnionych sesji (`wyrozniona: true`).
- Sekcja **O mnie** (`OMnie.astro`, kotwica `#o-mnie`): zdjęcie autora, 2-3 akapity, sprzęt (opcjonalnie).
- Sekcja **Kontakt** (`Kontakt.astro`, kotwica `#kontakt`): e-mail, linki do social mediów (Instagram wyeksponowany). Formularz tylko przez zewnętrzną usługę (Formspree/Web3Forms) albo `mailto:`. Bez własnego backendu.
- To jest **landing page**: O mnie i Kontakt są sekcjami strony głównej, nie osobnymi podstronami. Menu w nagłówku: Portret, Ślubna, Podróże i krajobraz (podstrony) oraz O mnie, Kontakt (kotwice `/#o-mnie`, `/#kontakt`, działające także z podstron).

### `[kategoria]/index.astro` (siatka sesji)
Wygląd wzorowany na zrzucie ekranu dostarczonym przez właściciela:

- Siatka **3 kolumn** (2 na tablecie do 900 px, 1 na telefonie do 560 px), `gap: 2rem 1.5rem`.
- Okładki w proporcjach **4:3**, `object-fit: cover`, `object-position` z pola `okladkaPozycja`.
- Pod okładką: tytuł (font szeryfowy, normalna waga), pod nim rok (mały, szary `#8a8a8a`, pogrubiony). Jeśli jest `podtytul`, wyświetl go po tytule (np. "Alicja i Patryk · Dolomity") lub w osobnej linii.
- Brak ramek, cieni i efektów wizualnych. Najwyżej delikatny hover (lekkie przyciemnienie lub `opacity`).
- Cały kafelek jest linkiem do `/[kategoria]/[sesja]/`.
- Sortowanie po `data` **malejąco**.
- Komponent: `SesjaCard.astro`.
- `getStaticPaths` generuje trasy wyłącznie dla widocznych kategorii.

### `[kategoria]/[sesja].astro` (pojedyncza sesja)
- Tytuł, podtytuł, data, opis (z treści `.md` lub pola `opis`).
- Galeria zdjęć (siatka lub masonry, miniatury ~800 px szerokości, `loading="lazy"`).
- Kliknięcie otwiera **lightbox** (React, `client:visible`): duże zdjęcie, strzałki lewo/prawo, klawisze ←/→/Esc, swipe na telefonie, zamykanie kliknięciem w tło. Do komponentu przekazuj gotowe adresy (`.src`) wygenerowane po stronie Astro, nie importy obrazów.
- Nawigacja "poprzednia / następna sesja" w tej samej kategorii (opcjonalnie).
- `og:image` = okładka sesji (JPEG, 1200×630).

### Pozostałe
- `404.astro`: prosta strona błędu z linkiem do strony głównej.
- Jeśli w przyszłości O mnie lub Kontakt mają stać się osobnymi podstronami, to osobne zadanie. Nie rób tego bez polecenia.

## 7. Layout i SEO

`Layout.astro` przyjmuje propsy: `title`, `description`, `ogImage?`, `ogType?`. Odpowiada za:

- `<html lang="pl">`, `<meta charset>`, `<meta name="viewport">`.
- `<title>` w formacie `{title} | {Nazwa fotografa}`.
- `meta description`, `link rel="canonical"` (na podstawie `Astro.site` i `Astro.url`).
- Open Graph: `og:title`, `og:description`, `og:image` (**adres bezwzględny**, `new URL(ogImage, Astro.site)`), `og:url`, `og:type`, `og:locale` = `pl_PL`.
- `twitter:card` = `summary_large_image`.
- Favicon, import czcionek i `global.css`.
- Header (logo/nazwa po lewej, menu po prawej, na telefonie hamburger) i Footer (© rok + nazwa, linki do social mediów).

Każda strona **musi** mieć własny `title` i `description`.

## 8. Obrazy

- Wszystkie zdjęcia sesji trzymamy w `src/assets/<kategoria>/<sesja>/` i renderujemy przez `<Image />` / `<Picture />` z `astro:assets` (automatyczny WebP/AVIF, `srcset`, wymiary).
- Zawsze podawaj `width`/`height` lub `widths` i `sizes`, żeby uniknąć skoku układu.
- `loading="lazy"` dla wszystkiego poza hero (hero: `loading="eager"` i `fetchpriority="high"`).
- Atrybut `alt` jest **obowiązkowy**. Opisowy, np. "Para młoda idąca drogą w Dolomitach". Nie używaj pustych ani "image1".
- Nazwy plików: małe litery, cyfry, myślniki. **Bez polskich znaków i spacji.**
- Nie dodawaj do repo oryginałów ani plików RAW. Zdjęcia mają być wyeksportowane do ~2400 px po dłuższym boku (JPEG).

## 9. Wygląd i design

- Styl: **jasny, minimalistyczny**, dużo białej przestrzeni, tło białe lub off-white, tekst ciemny. Bez kolorowych teł, jeden dyskretny akcent lub brak akcentu.
- Czcionki (maks. 2, z obsługą **Latin Extended** dla polskich znaków):
  - nagłówki/podpisy: szeryfowa (np. Cormorant Garamond albo systemowa Georgia),
  - tekst/menu: bezszeryfowa (np. Jost lub Inter).
- Kolory, fonty i odstępy w bloku `@theme` w `global.css` (tokeny Tailwinda, np. `--color-muted`, `--font-serif`), żeby łatwo zmienić motyw. W klasach używaj tych tokenów (`text-muted`, `font-serif`), bez wartości w stylu `text-[#8a8a8a]`.
- Układ siatki sesji w Tailwindzie, np. `grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-x-6 gap-y-8`, okładki `aspect-[4/3] object-cover w-full`.
- Minimum animacji: delikatny hover i płynne pojawianie się. Szanuj `prefers-reduced-motion`.
- **Mobile first**, układ musi być sprawdzony na szerokościach 360, 768 i 1280 px.

## 10. Dostępność i wydajność

- Semantyczny HTML (`header`, `nav`, `main`, `footer`, `h1` raz na stronę, logiczna hierarchia nagłówków).
- Widoczny stan `:focus-visible`, pełna obsługa klawiatury (menu, lightbox).
- Lightbox: `role="dialog"`, `aria-modal`, `aria-label` na przyciskach, zwracanie fokusu po zamknięciu.
- Kontrast tekstu min. 4.5:1 (szary rok `#8a8a8a` na bieli jest graniczny, w razie potrzeby użyj ciemniejszego `#767676`).
- Nie wysyłaj JS tam, gdzie nie jest potrzebny. React tylko w wyspach z `client:visible` (lub `client:idle`), nigdy `client:only` bez powodu.
- Cel: Lighthouse 90+ w każdej kategorii.

## 11. Prywatność

- Bez Google Analytics i zewnętrznych skryptów śledzących. Jeśli potrzebna analityka, tylko bez cookies (Plausible/GoatCounter), po uzgodnieniu.
- Czcionki hostowane lokalnie.
- Jeśli pojawi się formularz zewnętrzny, dodaj stronę `polityka-prywatnosci.astro`.
- Nie umieszczaj w kodzie ani w repo danych wrażliwych (klucze, prywatne adresy).

## 12. GitHub Pages i wdrożenie

- Workflow: `.github/workflows/deploy.yml` z oficjalną akcją `withastro/action` oraz `actions/deploy-pages`. Sprawdź aktualne wersje akcji w https://docs.astro.build/en/guides/deploy/github/.
- `astro.config.mjs`: ustaw `site`. Jeśli repo **nie** nazywa się `jacobjs7.github.io`, ustaw też `base: "/nazwa-repo"` i używaj `import.meta.env.BASE_URL` w linkach.
- Linki wewnętrzne zawsze z uwzględnieniem `base`, bez zakodowanych `/` na początku tam, gdzie `base` może być ustawione.
- Nie modyfikuj workflow ani `astro.config.mjs` bez wyraźnej potrzeby.

## 13. Zasady pracy (ważne)

1. **Pracuj małymi krokami.** Jedno zadanie = jedna zmiana. Nie przebudowuj całego projektu naraz.
2. **Po każdej zmianie projekt musi przechodzić `npm run build` bez błędów i ostrzeżeń TypeScript** (`npx astro check`). `npm run dev` nie wystarcza.
3. **Nie wymyślaj API.** Jeśli nie masz pewności co do składni Astro (zwłaszcza content collections, `astro:assets`), sprawdź dokumentację: https://docs.astro.build.
4. Nie usuwaj i nie przenoś istniejących plików bez wyraźnego polecenia.
5. Nie dodawaj martwego kodu, przykładowych treści "lorem ipsum" w finalnych stronach ani niepotrzebnych komentarzy.
6. Kod i komentarze po polsku lub angielsku, ale **spójnie**. Teksty widoczne dla użytkownika wyłącznie po polsku.
7. Style pisz klasami Tailwinda w komponentach. `global.css` zawiera tylko import Tailwinda, `@theme`, `@font-face`/importy czcionek i drobne style bazowe. Zmiany w nim rób ostrożnie.
8. Jeśli polecenie jest niejasne lub sprzeczne z tym plikiem, **zadaj jedno konkretne pytanie** zamiast zgadywać.

## 14. Kolejność prac (kamienie milowe)

1. Konfiguracja: Astro strict, React, Tailwind, sitemap, `astro.config.mjs`, workflow GitHub Pages, czcionki, `global.css`.
2. `kategorie.ts` + `Layout.astro` (SEO, header, footer, menu generowane z kategorii).
3. Kolekcja `sesje` (`content.config.ts`) i 1-2 przykładowe sesje.
4. `SesjaCard.astro` + `[kategoria]/index.astro` (siatka jak na zrzucie).
5. `[kategoria]/[sesja].astro` + galeria + `Lightbox.tsx`.
6. Strona główna (hero + kafelki kategorii).
7. Sekcje O mnie i Kontakt na stronie głównej, 404, `README.md` z instrukcją dodawania historii.
8. Finisz: favicon, `og-image.jpg`, `robots.txt`, test Lighthouse, test na telefonie, test podglądu linku.

## 15. Definicja "gotowe"

Zadanie jest skończone, gdy:

- `npm run build` i `npx astro check` przechodzą bez błędów,
- każda strona ma własny `title`, `description` i `og:image`,
- wszystkie obrazy mają sensowny `alt`,
- układ działa na telefonie, tablecie i desktopie,
- lightbox działa myszą, klawiaturą i gestem na telefonie,
- ukryta kategoria (`widoczna: false`) nie pojawia się w menu, na stronie głównej ani w sitemap,
- dodanie nowej kategorii i sesji nie wymaga zmian w szablonach,
- dodanie nowej historii (np. Portret → Maria) wymaga tylko folderu ze zdjęciami i jednego pliku `.md`, a po `git push` pojawia się w siatce kategorii, ma własną stronę, `og:image` i wpis w sitemap,
- Tailwind jest skonfigurowany według aktualnej dokumentacji, a kolory i fonty pochodzą z `@theme`.
