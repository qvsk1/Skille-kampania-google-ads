---
name: google-ads-1-pomiar
description: Ustawia i sprawdza pomiar pod Google Ads krok po kroku - baner zgód, GA4, konwersja i test. Użyj na „ustaw konwersje”, „sprawdź mój pomiar”, „czy Google Ads liczy sprzedaż”, „czy mój formularz się liczy”. Pierwszy z trzech skilli.
---

# Pomiar pod Google Ads

Prowadzisz osobę początkującą do stanu, w którym Google Ads i GA4 liczą jej konwersję, a baner zgód działa poprawnie. Bez tego kampania wydaje pieniądze na ślepo, dlatego ten skill jest pierwszy.

Wynik: lista „jest / brakuje”, wynik testu i zapis w pliku `moja-firma.md`.

## Zasady

- Na kontach klika użytkownik. Ty mówisz, gdzie wejść i co wpisać, i sprawdzasz po tym, co wklei albo pokaże na zrzucie ekranu. Nie prosisz o hasła ani o dostęp do kont.
- Jeden krok naraz. Po każdym czekasz na „zrobione”, wklejony tekst albo zrzut.
- Prosty język. Każde nowe słowo (kontener, tag, reguła) tłumaczysz jednym zdaniem, gdy pada pierwszy raz. „Trigger” z polskiego GTM to zawsze „reguła”.
- Panele Google i Cookiebota się zmieniają. Gdy nie masz pewności, jak nazywa się przycisk albo gdzie jest opcja, albo użytkownik widzi coś innego, powiedz to i poproś o zrzut, zanim podasz ścieżkę.
- Identyfikatorów (GTM-…, G-…, AW-…) nie zgadujesz. Bierzesz je tylko od użytkownika.
- Imienia, e-maila ani telefonu z formularza nie przekazujesz do tagów ani GA4 (wyjątek: konwersje rozszerzone, Google je szyfruje).
- Nie jesteś prawnikiem. Baner ustawiasz technicznie; politykę prywatności i treść zgód użytkownik sprawdza sam.
- Trudniejsze przypadki: plik `zaawansowane.md` obok skilla. Sięgasz po niego, gdy coś się nie zgadza albo użytkownik chce więcej.

## Krok 0. Plik `moja-firma.md`

Poszukaj w folderze użytkownika pliku `moja-firma.md`. Jeśli jest, przeczytaj go i nie pytaj o to, co tam już stoi. Jeśli nie masz dostępu do plików, poproś o wklejenie jego treści, a na końcu oddaj cały zaktualizowany plik w jednym bloku do zapisania.

Gdy pliku nie ma, utwórz go po pierwszej rundzie pytań:

```
# Moja firma

## Podstawy
- Strona:
- Platforma:
- Co sprzedaję:
- Konwersja:

## Pomiar

## Oferta i rynek

## Pierwsza kampania
```

Pierwsza runda, tylko o to, czego brakuje w „Podstawach”:

1. Jaki jest adres Twojej strony i co na niej sprzedajesz?
2. Na czym stoi strona: Shopify, WooCommerce, WordPress bez sklepu czy coś innego? Przy formularzu: jaka wtyczka (np. Contact Form 7) i czy po wysłaniu otwiera się strona z podziękowaniem?
3. Co jest dla Ciebie konwersją, czyli działaniem, za które chcesz płacić reklamą: zakup, wysłany formularz czy telefon? Wybierz jedno najważniejsze.

Potem jedno pytanie o konta: które z nich już masz: Google Tag Manager, GA4, Google Ads, Cookiebot?

## Krok 1. Co już jest

Jeśli użytkownik ma Google Tag Managera, poproś o eksport kontenera: w GTM „Administracja” → „Eksport kontenera” → bieżący obszar roboczy → „Pobierz”, a plik JSON wklejony albo załączony do rozmowy. Ten eksport pokazuje ustawienia, nie to, co naprawdę się odpala.

Gdy coś już jest ustawione, poproś też o nagranie sesji: w GTM „Podgląd” → przejść ścieżkę konwersji → w Tag Assistant pobrać sesję jako JSON (menu z trzema kropkami). Widać w nim, które tagi odpaliły przy którym zdarzeniu i jaki był stan zgód. Plik ma kilka MB: czytaj go skryptem z `zaawansowane.md` (część A), nie w całości.

Jeśli umiesz otworzyć stronę, sprawdź w jej kodzie: kod GTM (`GTM-`), skrypt Cookiebota, tag Google wklejony na sztywno (`gtag/js?id=…`).

W eksportach sprawdź:

| Co | Po czym poznasz w JSON |
|---|---|
| Baner zgód | tag z szablonu o nazwie „Cookiebot CMP” z regułą inicjacji zgody, włączony Consent Mode |
| GA4 | tag typu `googtag` albo `gaawc` z identyfikatorem `G-…` |
| Tag łączący konwersje | tag typu `gclidw` na wszystkich stronach |
| Konwersja Google Ads | tag typu `awct` z identyfikatorem i etykietą oraz regułą zgodną ze znaczeniem działania (np. „Dodanie do koszyka” nie odpala się przy wejściu do kasy) |
| Wartość zakupu (tylko sklep) | w tagu konwersji wartość, waluta i numer zamówienia ze zmiennych, nie wpisane na sztywno |
| Kłopoty | tagi wstrzymane, dwa tagi GA4 z tym samym `G-…`, tagi HTML z kodem reklamowym bez wymaganej zgody, tag Google wklejony w motyw obok GTM (liczy podwójnie, psuje zgody; usunąć) |

Pokaż wynik jako krótką tabelę: element, „jest” albo „brakuje”, jedno zdanie uwagi. Dalej prowadzisz tylko przez to, czego brakuje albo co jest źle, w kolejności kroków 2–5.

Przed zmianami w istniejącym GTM sprawdź, czy w obszarze roboczym nie wiszą cudze nieopublikowane zmiany; publikacja wysłałaby je razem z nowymi.

## Krok 2. Google Tag Manager na stronie

GTM to jedno miejsce, z którego dodajesz kody pomiarowe bez grzebania w stronie. Konto zakłada się na tagmanager.google.com, kontener typu „Sieć”. Po założeniu GTM pokazuje dwa fragmenty kodu i identyfikator `GTM-…`.

- **WordPress i WooCommerce:** darmowa wtyczka GTM4WP, w jej ustawieniach wklejasz `GTM-…`. W jej integracjach włącz WooCommerce (sklep: zakup trafia do GTM jako zdarzenie `purchase`) i Contact Form 7, jeśli formularz stoi na tej wtyczce.
- **Własny kod albo inny kreator:** oba fragmenty wkleja ten, kto ma dostęp do kodu strony, w miejscach podanych przez GTM. Wiele kreatorów ma na to osobne pole w ustawieniach.
- **Shopify:** GTM wklejony w motyw nie widzi koszyka ani zakupu. Najprościej: aplikacja „Google & YouTube” ze sklepu Shopify ustawia GA4 i konwersję zakupu, a baner zgód daje aplikacja „Cookiebot” dla Shopify. Wtedy pomiń kroki 3–5. Test jest inny, bo podgląd GTM nie działa w kasie Shopify: użytkownik składa zamówienie testowe i sprawdza w GA4 w raporcie „Czas rzeczywisty”, czy pojawił się zakup.

## Krok 3. Baner zgód: Cookiebot z Consent Mode

Wyjaśnij jednym zdaniem: bez zgody użytkownika Google nie może używać danych do reklam, a Consent Mode przekazuje tagom Google, na co ktoś się zgodził.

Uprzedź przed założeniem konta: darmowy plan Cookiebota jest dla jednej domeny do 50 podstron. Większa strona, na przykład sklep z setką produktów, płaci abonament; aktualną cenę użytkownik sprawdza w cenniku Cookiebota przed rejestracją.

1. Konto na cookiebot.com, dodana domena, skopiowany identyfikator „Domain Group ID”. W ustawieniach Cookiebota automatyczne blokowanie plików cookie zostaje wyłączone, bo baner wchodzi przez GTM.
2. W GTM: „Szablony” → „Przeszukaj galerię” → „Cookiebot CMP” → dodaj do obszaru roboczego. Potem „Tagi” → „Nowy” → typ „Cookiebot CMP”.
3. W tagu: Domain Group ID wklejony w pole „Cookiebot ID”, opcja „Enable Google Consent Mode” zostaje zaznaczona, a domyślny stan zgód zostaje „denied”.
4. Reguła (czyli warunek, kiedy tag ma się uruchomić): „Inicjacja zgody – Wszystkie strony” (Consent Initialization – All Pages). Ten tag musi odpalić się przed wszystkimi innymi.

Tagi Google (GA4, konwersja, łączenie konwersji) same czytają zgody. Nie dodajesz im osobnych warunków.

## Krok 4. GA4

1. W GA4 (analytics.google.com): „Administracja” → „Strumienie danych” → strumień strony → skopiowany „Identyfikator pomiaru” `G-…`. Gdy nie ma konta, najpierw usługa i strumień typu „Sieć”.
2. W GTM: nowy tag typu „Tag Google”, w polu „Identyfikator tagu” wklejone `G-…`, reguła „Zdarzenie inicjujące – wszystkie strony” (Initialization – All Pages).
3. Przy formularzu: w „Pomiarze zaawansowanym” strumienia wyłącz „Interakcje z formularzami” (liczą też nieudane wysyłki).

## Krok 5. Konwersja Google Ads

Jeśli użytkownik dopiero zakłada konto Google Ads, uprzedź: Google od razu zaproponuje kampanię. Ma jej nie tworzyć, tylko poszukać opcji założenia konta bez kampanii. Kampania powstanie w trzecim skillu.

1. W Google Ads: „Cele” → „Podsumowanie” → „Utwórz działanie powodujące konwersję” → adres strony → „Skanuj” → kategoria zgodna z konwersją z „Podstaw” (zakup, przesłanie formularza, kontakt telefoniczny). Przy instalacji zakładka „Użyj Menedżera tagów Google”. Użytkownik kopiuje „Identyfikator konwersji” i „Etykietę konwersji”.
   Przed zapisaniem: liczenie „Jedna” przy formularzu i telefonie, „Każda” przy zakupie; działanie główne, bo pod główne Google optymalizuje. Drobne działania (np. dodanie do koszyka) najwyżej jako dodatkowe.
2. W GTM tag „Tag łączący konwersje”, reguła „Wszystkie strony”.
3. W GTM reguła, która łapie udaną konwersję, nie samą próbę:
   - **Formularz ze stroną podziękowania:** reguła „Wyświetlenie strony”, adres zawiera np. `/dziekuje`.
   - **Formularz bez strony podziękowania** (komunikat na tej samej stronie), pierwsza możliwa metoda:
     1. Zdarzenie po udanym wysłaniu, od dewelopera albo wtyczki. Contact Form 7 z GTM4WP: reguła „Zdarzenie niestandardowe” `gtm4wp.contactForm7MailSent`, nie `…Submitted` (to przychodzi przy każdej próbie). Nazwę skopiuj z Podglądu.
     2. „Widoczność elementu” na komunikat o sukcesie; poproś o zrzut albo kod komunikatu.
     3. Kliknięcie „Wyślij” tylko w ostateczności.
     Nie „Przesłanie formularza”: często liczy nieudane próby.
   - **Telefon:** reguła „Kliknięcie – tylko linki”, adres linku zaczyna się od `tel:`.
   - **Zakup w WooCommerce:** reguła „Zdarzenie niestandardowe” o nazwie `purchase`. Do tego trzy zmienne warstwy danych: `ecommerce.value`, `ecommerce.currency`, `ecommerce.transaction_id`.
4. W GTM tag „Śledzenie konwersji Google Ads”: identyfikator i etykieta z punktu 1, reguła z punktu 3. Przy zakupie dodatkowo wartość, waluta i numer zamówienia ze zmiennych.
5. Ta sama reguła odpala drugi tag: „Zdarzenie GA4” o nazwie `generate_lead` (formularz, telefon) albo `purchase` (zakup, z włączonym wysyłaniem danych e-commerce). Po pierwszym teście oznacz `generate_lead` w GA4 jako zdarzenie kluczowe (`purchase` jest nim od razu), ale nie importuj go do Google Ads: konwersję liczy już tag z punktu 4.

Niczego jeszcze nie publikuj. Najpierw test.

## Krok 6. Test

Uprzedź: Podgląd wysyła prawdziwe dane. GA4 zapisze konwersję, Google Ads może ją policzyć, formularz wyśle prawdziwy mail do firmy. Jeden test na ścieżkę, w treści „TEST”.

1. Okno incognito (zwykłe pamięta zgodę z banera). W GTM kliknij „Podgląd” i podaj adres strony. Otworzy się strona i okno Tag Assistant.
2. Przed kliknięciem w banerze: w Tag Assistant, w zakładce zgód, zgody reklamowe mają stan „odmowa”; po „Zezwól” — „zgoda”.
3. Tylko formularz: „Wyślij” przy pustym formularzu. Konwersja nie może się odpalić.
4. Zrób konwersję testową: wyślij poprawny formularz, kliknij numer telefonu albo złóż zamówienie testowe.
5. W Tag Assistant przy tym zdarzeniu na liście uruchomionych tagów są: konwersja Google Ads i zdarzenie GA4. Każda raz.
6. W GA4: „Administracja” → „Wyświetlanie danych” → „DebugView”. Widać tam Twoje wejście i zdarzenie konwersji.

Poproś o zrzut albo opis każdego punktu lub nagranie sesji (jak w kroku 1) i oceń: „działa” albo „nie działa, bo…”. W nagraniu: `gcs=G100` przed zgodą, `G111` po „Zezwól”. Gdy coś nie działa, popraw jedną rzecz i powtórz tylko ten punkt.

Gdy wszystko działa: w GTM „Prześlij” → „Opublikuj”. Status działania w Google Ads zmieni się po kilku godzinach, czasem po dobie; konwersje z reklam przyjdą po starcie kampanii.

**Konwersje rozszerzone** (Google dopasowuje zaszyfrowany e-mail do konta Google i przypisuje reklamom więcej konwersji) włączasz po udanym teście, według `zaawansowane.md` (część D). Przy formularzu zwykle to kilka kliknięć; przy zakupie bez pomocy technicznej wpisz je do „Do zrobienia”.

## Gdy coś się nie zgadza

Test nie przechodzi, konwersji jest za dużo albo za mało, zgody wyglądają dziwnie: znajdź objaw w `zaawansowane.md` (części E–H) i popraw jedną rzecz naraz.

## Wynik i zapis

Dopisz do `moja-firma.md` w „Podstawach” to, czego tam brakowało, a w sekcji „Pomiar”:

```
## Pomiar
Sprawdzone: [data]
- GTM na stronie: jest / brakuje ([GTM-…])
- Baner zgód z Consent Mode: jest / brakuje (test odmowy: działa / nie)
- GA4: jest / brakuje ([G-…])
- Konwersja Google Ads: [co liczymy, na jakie zdarzenie], jest / brakuje
- Konwersje rozszerzone: włączone / do zrobienia
- Test: [co zadziałało, co nie]
- Do zrobienia: [najwyżej 3 rzeczy, od najważniejszej]
```

Na koniec jedno zdanie: następny jest skill `google-ads-2-oferta`, który ustala, co i komu mówić w reklamach.
