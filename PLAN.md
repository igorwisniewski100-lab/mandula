# PLAN.md — Strona marki (handmade przynęty wędkarskie)

> **Dla Claude Code:** To jest pełny brief projektu. Zbuduj aplikację zgodnie z sekcjami poniżej.
> Trzymaj się kolejności z sekcji **SEKWENCJA BUDOWY**. Na końcu każdego etapu odhacz
> **CHECKLISTĘ ODBIORU**. Nie buduj koszyka/płatności — dane produktów mają być gotowe pod
> doklejenie sklepu w Fazie 2, ale sam sklep NIE jest częścią tego zadania.

---

## 0. PLACEHOLDERY DO UZUPEŁNIENIA (zanim ruszysz — poproś użytkownika jeśli puste)

- `NAZWA_MARKI` = _[do uzupełnienia]_
- `TYP_PRZYNET` = _[wobblery / gumy / błystki / spinnerbaity / mix — do uzupełnienia]_
- `DOMENA` = _[np. nazwamarki.pl]_
- `DYSTRYBUTORZY` = _[3 sklepy/hurtownie: nazwa + miasto + link, do uzupełnienia]_
- `EMAIL_KONTAKT` = _[do uzupełnienia]_
- `TELEFON` = _[do uzupełnienia]_
- Kolory marki: domyślnie propozycja niżej — użytkownik może podmienić
- Logo: placeholder tekstowy, jeśli brak pliku

---

## 1. CEL PROJEKTU

Strona-wizytówka marki rzemieślniczych przynęt wędkarskich. **Priorytet #1: B2B** —
zbudować wiarygodność u sklepów/hurtowni i pozyskiwać nowych dystrybutorów. Katalog jest
**poglądowy** (bez cen do klienta końcowego i bez checkoutu). Drugorzędnie: leady B2C na
przynęty **na zamówienie / custom**.

Cele mierzalne strony:
1. Zapytanie hurtowe (nowy sklep chce brać towar) → formularz „Współpraca".
2. Zapytanie custom (klient chce przynętę na zamówienie) → formularz „Na zamówienie".
3. Budowa marki: historia rzemieślnika, proces produkcji, dowody skuteczności (zdjęcia ryb).
4. Fundament pod sklep e-commerce w Fazie 2 (dane produktów gotowe do rozszerzenia).

---

## 2. STACK

- **Next.js 14+ (App Router) + TypeScript**
- **Tailwind CSS** (+ ewentualnie shadcn/ui dla formularzy)
- Produkty i treści jako **typowane dane** (`/content/products/*.ts` lub JSON + Zod schema).
  Ma być jedno źródło prawdy, z którego później zasili się sklep.
- Obrazy: `next/image`, optymalizacja, lazy-load.
- Formularze: Route Handler (`app/api/…`) → wysyłka mailem przez **Resend** (klucz w env),
  honeypot + rate-limit na spam. Fallback: log do konsoli jeśli brak klucza.
- SEO: `next-sitemap` lub własny `sitemap.ts` + `robots.ts`, JSON-LD.
- Analytics: **GTM** (kontener w env) + **GA4**, **Consent Mode v2**, własny baner cookies
  (localStorage, klucze `q_c` / `q_c_s` — zgoda przed odpaleniem tagów).
- i18n scaffold: **PL domyślnie**, przygotować strukturę pod **EN/DE** (export później) —
  nie tłumaczyć jeszcze, tylko architektura gotowa (np. `next-intl`).
- Deploy: **Vercel**. Env przez `.env.local` + `.env.example`.

---

## 3. ARCHITEKTURA STRON (routing PL)

| Ścieżka | Strona | Cel |
|---|---|---|
| `/` | Home | Hero + historia + wyróżnione przynęty + dowody połowów + CTA B2B i custom |
| `/przynety` | Katalog | Siatka produktów, filtry: gatunek ryby / technika / typ |
| `/przynety/[slug]` | Karta produktu | Zdjęcia, specyfikacja, dostępność, „gdzie kupić" |
| `/rzemioslo` | O nas / proces | Historia twórcy, warsztat, proces produkcji (kluczowe dla handmade) |
| `/wspolpraca` | B2B / hurt | Oferta dla sklepów, warunki, formularz zapytania hurtowego |
| `/na-zamowienie` | Custom (B2C) | Jak działa zamówienie custom + formularz |
| `/gdzie-kupic` | Dystrybutorzy | Lista 3 dystrybutorów (mapka opcjonalnie) |
| `/kontakt` | Kontakt | Dane + formularz |
| `/blog` | Blog | Scaffold pod SEO (proces, poradniki, relacje z łowisk) — puste na start |

Nawigacja główna: Przynęty · Rzemiosło · Współpraca · Na zamówienie · Gdzie kupić · Kontakt.
Wyróżnij **„Współpraca"** jako główne CTA (to jest cel B2B).

---

## 4. MODEL DANYCH (fundament pod sklep w Fazie 2)

```ts
type Availability = 'u_dystrybutorow' | 'na_zamowienie' | 'wkrotce' | 'wyprzedane';

type Product = {
  id: string;
  slug: string;
  name: string;
  type: string;              // wobbler / guma / błystka / spinnerbait...
  targetFish: string[];      // np. ['szczupak','okoń','sandacz']
  technique: string[];       // np. ['spinning','trolling']
  sizeMm?: number;
  weightG?: number;
  colors: { name: string; hex?: string; imageUrl?: string }[];
  images: { url: string; alt: string }[];
  description: string;
  availability: Availability;
  sku?: string;
  price?: number;            // OPCJONALNE — puste teraz, pod sklep później
  featured?: boolean;
};
```

- Waliduj **Zod**. Utwórz **6–8 przykładowych produktów** (placeholder, realistyczne dla `TYP_PRZYNET`).
- Filtry katalogu czytają z tych samych danych.
- Karta produktu: galeria + specyfikacja + warianty kolorystyczne + `availability` badge +
  sekcja „Gdzie kupić" (link do `/gdzie-kupic`). Jeśli `na_zamowienie` → CTA do formularza custom.

---

## 5. KIERUNEK WIZUALNY

Estetyka: **rzemieślnicza, surowa, outdoorowa** — NIE korpo. Autentyczność handmade.
Zdjęcia rządzą (makro produktu + ryby + warsztat). Mobile-first (wędkarz przegląda na telefonie).

- **Typografia (domyślnie, do podmiany):** display `Fraunces` (charakterny, rzemieślniczy) +
  body `Inter`. Alternatywa surowsza: `Oswald`/`Bebas Neue` + `Inter`.
- **Paleta (domyślnie):** głęboka woda/stal jako baza (grafit `#1A1D21`, stal `#2E3439`),
  ciepły akcent (miedź/musztarda `#C8892A`) i biel złamana `#F4F1EA`. Podmienialne w `tailwind.config`.
- Dużo przestrzeni, mocne nagłówki, fotografia pełnoekranowa w hero.
- Komponent **„Dowody z wody"** — galeria zdjęć złowionych ryb (to najsilniejszy element
  sprzedażowy w tej niszy; zbuduj go jako reużywalną sekcję z danych).
- Sekcja **procesu produkcji** na `/rzemioslo` (kroki: toczenie/lakierowanie/testy) — timeline/kafelki.
- Wydajność: cel **Lighthouse 90+** desktop i mobile. Dostępność: podstawy WCAG (kontrast, alt, focus).

Placeholdery obrazów: użyj kolorowych bloków z etykietą (np. „FOTO: wobbler makro"), żeby
łatwo było podmienić na realne zdjęcia.

---

## 6. FORMULARZE

Trzy formularze → Route Handler → mail (Resend) na `EMAIL_KONTAKT`. Honeypot + walidacja Zod +
komunikat sukcesu/błędu. Zapis leada opcjonalnie do pliku/log (pod przyszłe CRM).

1. **Zapytanie hurtowe** (`/wspolpraca`): nazwa firmy, NIP, osoba, email, telefon,
   szacowany wolumen, wiadomość.
2. **Zamówienie custom** (`/na-zamowienie`): typ przynęty, gatunek docelowy, kolorystyka,
   ilość, email/telefon, opis.
3. **Kontakt** (`/kontakt`): imię, email, wiadomość.

---

## 7. SEO / ANALYTICS

- `metadata` per strona (title, description, OG). Dynamiczne OG dla kart produktów.
- JSON-LD: `Organization` + `Product` (na kartach) + `LocalBusiness` (jeśli stacjonarnie).
- `sitemap.ts` + `robots.ts`.
- Słowa kluczowe pod PL wędkarstwo w treściach: `TYP_PRZYNET` + gatunek + technika
  (np. „wobbler na szczupaka", „ręcznie robione przynęty na okonia").
- GTM + GA4 + Consent Mode v2 + baner cookies (zgoda przed tagami; klucze `q_c`/`q_c_s`).
- Zdarzenia konwersji: wysłanie formularza hurtowego i custom.

---

## 8. SEKWENCJA BUDOWY

1. Scaffold Next.js + TS + Tailwind + struktura folderów + `.env.example`.
2. Model danych (Zod) + 6–8 przykładowych produktów.
3. Layout: nav + footer + baner cookies + i18n scaffold (PL).
4. Home (hero, wyróżnione, dowody z wody, CTA B2B/custom).
5. Katalog `/przynety` + filtry + karty `/przynety/[slug]`.
6. `/rzemioslo` (proces) + `/gdzie-kupic` (dystrybutorzy).
7. `/wspolpraca` + `/na-zamowienie` + `/kontakt` + Route Handlery formularzy (Resend).
8. SEO (metadata, JSON-LD, sitemap, robots) + GTM/GA4/consent.
9. `/blog` scaffold (lista + pojedynczy wpis, MDX, na start puste).
10. Polish: obrazy/placeholdery, wydajność, dostępność, responsywność.

---

## 9. CHECKLISTA ODBIORU

- [ ] `npm run dev` i `npm run build` przechodzą bez błędów.
- [ ] Wszystkie strony z sekcji 3 istnieją i są w nawigacji.
- [ ] Katalog filtruje po gatunku / technice / typie z danych.
- [ ] Karta produktu pokazuje galerię, specyfikację, dostępność, „gdzie kupić".
- [ ] 3 formularze wysyłają mail (lub logują przy braku klucza) + walidacja + sukces/błąd.
- [ ] Baner cookies blokuje tagi do zgody; GTM/GA4 pod Consent Mode v2.
- [ ] `sitemap.xml` + `robots.txt` + JSON-LD Product na kartach.
- [ ] Lighthouse ≥ 90 (mobile i desktop) na Home i karcie produktu.
- [ ] Responsywność OK od 360px w górę.
- [ ] i18n scaffold obecny (PL działa, EN/DE gotowe do dodania).
- [ ] `.env.example` opisuje wszystkie klucze (Resend, GTM, GA4).
- [ ] Wszystkie `PLACEHOLDERY` z sekcji 0 wstrzyknięte przez config/env, nie zahardkodowane.

---

## 10. POZA ZAKRESEM (Faza 2 — NIE budować teraz)

Koszyk, checkout, płatności (Stripe/Przelewy24), stany magazynowe, konta klientów,
tłumaczenia EN/DE. Dane produktów mają jednak być gotowe pod te funkcje.
