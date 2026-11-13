# Pomiar pod Google Ads: wiedza zaawansowana

Plik referencyjny do skilla `google-ads-1-pomiar`. Sięgasz po niego, gdy:
- czytasz nagranie sesji z Tag Assistant (część A),
- test z kroku 6 nie przechodzi albo liczby się nie zgadzają (części B, E–H),
- użytkownik chce więcej niż jedną konwersję: porównanie landingów, konwersje rozszerzone przy zakupie (części C–D).

Zasady skilla obowiązują dalej: jeden krok naraz, prosty język, identyfikatorów nie zgadujesz. Nie znasz dokładnej nazwy przycisku albo miejsca w panelu? Powiedz to i poproś o zrzut ekranu, zanim podasz ścieżkę. Nazwy zdarzeń z wtyczek zawsze potwierdzasz w Podglądzie, zanim wpiszesz je do reguły.

---

## A. Nagranie sesji z Tag Assistant

**Skąd:** GTM → „Podgląd” → ścieżka do przejścia (np. strona → formularz → wysłanie) → w Tag Assistant pobranie sesji jako JSON (menu z trzema kropkami; gdy użytkownik go nie widzi, zrzut).

**Co w nim jest, czego nie ma w eksporcie kontenera:** każde zdarzenie po kolei, które tagi odpaliły przy którym zdarzeniu, stan zgód w tej chwili i co poszło do Google (z nazwą działania konwersji przy żądaniu do Google Ads). Plik ma zwykle 5–10 MB, więc nie czytasz go w całości.

**Skrypt** (Python 3, bez dodatkowych bibliotek; uruchom: `python3 ta.py nagranie.json`):

```python
import json, sys, ast
d = json.load(open(sys.argv[1], encoding="utf-8"))
for c in d["data"]["containers"]:
    if not c.get("messages"):
        continue
    print(f"\n=== {c['product']} {c['publicId']}")
    fired = {}
    for tag, lst in (c.get("tagsFired") or {}).items():
        for x in lst:
            fired.setdefault(x["index"], []).append(tag)
    for m in sorted(c["messages"], key=lambda m: m["index"]):
        cs = (m.get("consentData") or {}).get("consentStatus") or {}
        full = (m.get("consentData") or {}).get("fullConsentList") or {}
        ad = full.get("ad_storage", {}).get("isConsentGranted")
        an = full.get("analytics_storage", {}).get("isConsentGranted")
        hits = []
        for dd in m.get("data") or []:
            for h in dd.get("hitInfo") or []:
                ps = h.get("parameters") or []
                if isinstance(ps, str):
                    ps = ast.literal_eval(ps)
                names = {p.get("name"): p.get("value") for p in ps if isinstance(p, dict)}
                extra = f" gcs={names['gcs']}" if "gcs" in names else ""
                if "em" in names:
                    extra += " +dane_użytkownika(em)"
                hits.append(f"{h.get('displayName')} -> {h.get('destination')}{extra}")
        if not fired.get(m["index"]) and not hits and m.get("eventName", "").startswith(
                ("gtm.click", "gtm4wp.formElement", "gtm.formInteract", "gtm.scroll", "gtm.timer")):
            continue  # szum bez tagów i żądań
        print(f"#{m['index']:>3} {m.get('eventName', ''):<35} zgody ad={ad} analytics={an}"
              + (" ZGODA-ZA-PÓŹNO" if cs.get("wasSetLate") else ""))
        for t in fired.get(m["index"], []):
            print(f"      tag: {t}")
        for h in hits:
            print(f"      hit: {h}")
```

Wynik ma dwie części: `=== GTM` (zdarzenia i tagi z kontenera) oraz `=== GTAG` (żądania wysłane do Google, z `gcs` i nazwą działania konwersji). Numer `#` to pozycja zdarzenia na liście po lewej w Tag Assistant; numeracja w obu częściach może się różnić o 1–3.

Struktura pliku może się zmienić. Gdy skrypt się wysypie, wypisz klucze najwyższych poziomów (`data.containers[].messages[]`, `tagsFired`, `data[].hitInfo[]`) i dopasuj go, zamiast czytać plik w całości.

**Na co patrzysz:**

| W wyniku | Znaczy |
|---|---|
| Tag konwersji przy zdarzeniu, które nie pasuje do nazwy działania (np. „Dodanie do koszyka” przy wejściu do kasy) | Zła reguła. Konwersja liczy coś innego, niż mówi nazwa |
| Tag konwersji przy `gtm.formSubmit`, a po nim zdarzenie wtyczki (np. `gtm4wp.contactForm7MailSent`) bez tagów | Konwersja liczy próbę wysłania, nie udane wysłanie. Przepnij regułę (część B) |
| Ten sam tag dwa razy przy jednym zdarzeniu albo dwie konwersje przy jednym wysłaniu | Podwójne liczenie (część G) |
| Dwa zdarzenia `gtm.js` na jednej stronie | Na stronie jest drugi kod Google poza GTM (część F) |
| `ZGODA-ZA-PÓŹNO` (`wasSetLate`) | Domyślne zgody ustawione po pierwszych tagach (część E) |
| `gcs=G111` w nagraniu z incognito przed kliknięciem w banerze | Baner nie blokuje albo zgoda była zapisana; powtórz test |
| `+dane_użytkownika(em)` albo żądanie „Dane przekazywane przez użytkowników” przy konwersji | Konwersje rozszerzone działają (część D) |
| Brak żądań do innych narzędzi (np. Meta) | Nic nie znaczy: Tag Assistant pokazuje głównie Google. Sprawdź kod strony i panel tamtego narzędzia |

W raporcie dla użytkownika podawaj numer zdarzenia przy każdym wniosku, żeby mógł to znaleźć w Tag Assistant.

---

## B. Formularz bez strony podziękowania

Konwersja ma się odpalić po **udanym** wysłaniu. Kolejność metod, od najlepszej:

1. **Zdarzenie wysłane przez stronę po udanym wysłaniu** (od dewelopera albo z wtyczki) → reguła „Zdarzenie niestandardowe”.
2. **Widoczność komunikatu o sukcesie** → reguła „Widoczność elementu”, selektor komunikatu, uruchamianie raz na stronę.
3. **Klik w „Wyślij”** — tylko gdy nic innego nie działa; liczy też formularze z błędami.

Reguła „Przesłanie formularza” (`gtm.formSubmit`) i automatyczne „Interakcje z formularzami” w GA4 reagują na próbę, nie na sukces. Nie bierz ich na konwersję.

**Contact Form 7 + GTM4WP** (integracja CF7 włączona we wtyczce). Zdarzenia w dataLayer:

| Zdarzenie | Kiedy | Na konwersję? |
|---|---|---|
| `gtm4wp.contactForm7MailSent` | mail wysłany | **tak** |
| `gtm4wp.contactForm7Submitted` | każda próba wysłania | nie |
| `gtm4wp.contactForm7InvalidInput`, `…MailFailed`, `…SpamDetected` | błąd walidacji, błąd wysyłki, spam | nie |

Zdarzenie niesie `formid` (rozróżnisz formularze) i `inputs` (pola formularza; patrz część C).

**Gdy GTM4WP nie daje zdarzenia CF7** — tag „Niestandardowy kod HTML” z regułą „DOM gotowy” na stronach z formularzem:

```html
<script>
document.addEventListener('wpcf7mailsent', function (e) {
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({ event: 'form_sent', form_id: 'cf7_' + e.detail.contactFormId });
});
</script>
```

Reguła konwersji: „Zdarzenie niestandardowe” `form_sent`.

**Inne wtyczki** wysyłają w przeglądarce własne zdarzenie po sukcesie. Nazwy według dokumentacji wtyczek — przed użyciem sprawdź w Podglądzie, że się pojawia:

| Wtyczka | Zdarzenie po sukcesie | Jak nasłuchiwać |
|---|---|---|
| Contact Form 7 | `wpcf7mailsent` | `document.addEventListener` (jak wyżej) |
| Elementor Pro (formularz) | `submit_success` | jQuery: `jQuery(document).on('submit_success', function (e) { … })` |
| WPForms (wysyłka AJAX) | `wpformsAjaxSubmitSuccess` | jQuery: `jQuery(document).on('wpformsAjaxSubmitSuccess', function (e) { … })` |
| Gravity Forms (AJAX) | `gform_confirmation_loaded` | jQuery: `jQuery(document).on('gform_confirmation_loaded', function (e, formId) { … })` |
| HubSpot (osadzony) | `message` z `data.type === 'hsFormCallback'` i `data.eventName === 'onFormSubmitted'` | `window.addEventListener('message', function (e) { … })` |

W każdym wariancie w środku robisz `dataLayer.push({ event: 'form_sent', form_id: '…' })`.

**Prośba do dewelopera** (gdy formularz jest pisany ręcznie), do wklejenia:

> Po udanym wysłaniu formularza (po odpowiedzi serwera, nie po kliknięciu) wywołaj:
> `window.dataLayer = window.dataLayer || []; window.dataLayer.push({ event: 'generate_lead', form_id: 'NAZWA_FORMULARZA', form_location: 'MIEJSCE_NA_STRONIE' });`
> Bez imienia, e-maila i telefonu w tym wywołaniu.

**Testy obowiązkowe:** pusty formularz (zdarzenie nie przychodzi, tagi nie odpalają), poprawne wysłanie (tagi odpalają raz), drugi formularz na tej samej stronie (jeśli jest — czy rozróżniasz je po `form_id`).

---

## C. Parametry leada i porównanie landingów

Gdy użytkownik ma kilka formularzy albo testuje dwa landingi, jedno działanie konwersji w Google Ads wystarczy. Różnice widać w GA4 dzięki parametrom zdarzenia `generate_lead`:

| Parametr | Przykład | Po co |
|---|---|---|
| `form_id` | `kontakt_glowny` | który formularz |
| `form_location` | `hero`, `stopka` | gdzie na stronie |
| typ zapytania, budżet (pola wyboru z formularza) | `oferta`, `10-20k` | jakość leadów |

1. W tagu „Zdarzenie GA4” dodaj parametry (wartości stałe albo zmienne).
2. W GA4 zarejestruj każdy jako wymiar niestandardowy o zakresie „Zdarzenie” (Administracja → Wyświetlanie danych → Definicje niestandardowe; gdy menu wygląda inaczej — zrzut). Bez tego parametr nie pojawi się w raportach. Działa od chwili rejestracji, wstecz nie.

**Pola wyboru z CF7** — GTM4WP podaje je w `inputs` (lista `{name, value}`). Zmienna warstwy danych `inputs` (wersja 2), a potem „Niestandardowy kod JavaScript”:

```js
function () {
  var a = {{dlv - cf7 inputs}} || [];
  for (var i = 0; i < a.length; i++) {
    if (a[i].name === 'NAZWA-POLA') return a[i].value;
  }
  return undefined;
}
```

Nazwę pola (`NAZWA-POLA`) bierzesz z Podglądu, zakładka „Warstwa danych” przy zdarzeniu CF7. Wyciągasz tylko pola wyboru (typ, budżet, temat). **Imienia, e-maila, telefonu, firmy ani treści wiadomości nie wyciągasz i nie wysyłasz do żadnego tagu** — w GA4 to naruszenie zasad Google i RODO.

---

## D. Konwersje rozszerzone

Google dopasowuje zaszyfrowany (SHA-256) e-mail albo telefon do kont Google i przypisuje reklamom konwersje, których nie złapały pliki cookie. Włączasz je po udanym teście podstawowej konwersji.

**Formularz:**
1. W nagraniu (część A) przy konwersji szukaj żądania „Dane przekazywane przez użytkowników” albo `em` w żądaniu konwersji. Jest → tag Google zbiera e-mail automatycznie; zostaje włączenie konwersji rozszerzonych w ustawieniach konwersji Google Ads i akceptacja warunków (zrzut, jeśli nie widać).
2. Nie ma → w tagu „Śledzenie konwersji Google Ads” zaznacz uwzględnianie danych przekazywanych przez użytkowników → nowa zmienna → konfiguracja ręczna → e-mail z poniższej zmiennej:

```js
function () {
  var pola = document.querySelectorAll('input[type="email"]');
  for (var i = 0; i < pola.length; i++) {
    if (pola[i].value) return pola[i].value.trim().toLowerCase();
  }
  return undefined;
}
```

Działa, gdy pole e-mail jest jeszcze na stronie w chwili konwersji. Jeśli formularz znika po wysłaniu, deweloper dodaje e-mail do `dataLayer.push` tylko dla tego celu, a Ty mapujesz go wyłącznie do pola danych użytkownika w tagu Ads.
3. Test: w nagraniu przy konwersji `+dane_użytkownika(em)`.

**Zakup WooCommerce (GTM4WP):** dane klienta przy zamówieniu są w warstwie danych zdarzenia zakupu. Ścieżkę (np. e-mail rozliczeniowy w danych zamówienia) odczytaj w Podglądzie, zakładka „Warstwa danych” przy `purchase`, i zrób zmienne warstwy danych: e-mail, telefon, imię, nazwisko, ulica, miasto, kod, kraj. Wskaż je w tym samym polu tagu konwersji co wyżej. Nie twórz drugiego tagu konwersji.

**Warunek:** polityka prywatności mówi o przekazywaniu danych Google w celach reklamowych, a zgoda `ad_user_data` jest w banerze (Cookiebot z Consent Mode ją ma). To użytkownik potwierdza sam.

---

## E. Zgody

**Kod `gcs` w żądaniach:** `G1` + zgoda na reklamy + zgoda na analitykę.

| `gcs` | Reklamy | Analityka |
|---|---|---|
| `G100` | odmowa | odmowa |
| `G110` | zgoda | odmowa |
| `G101` | odmowa | zgoda |
| `G111` | zgoda | zgoda |

**Komplet testów** (każdy w nowym oknie incognito):
1. Wejście bez klikania w baner → żądania z `G100`.
2. „Odmów” → nadal `G100`; tagi Google mogą wysyłać żądania bez plików cookie, tak działa Consent Mode.
3. „Zezwól” → `G111` od tego zdarzenia.
4. Zamknięcie banera krzyżykiem (jeśli jest) → sprawdź, co ustawia; często zostaje odmowa.

**Zgody ustawiane za późno** (`wasSetLate`): tag Cookiebota nie jest na regule „Inicjacja zgody – Wszystkie strony”, albo na stronie jest kod Google spoza GTM, który startuje przed kontenerem (część F), albo baner wklejony w kod strony i jednocześnie w GTM. Baner ma być w jednym miejscu: w GTM.

**Tagi spoza Google** (Meta, Hotjar, TikTok i inne w „Niestandardowym kodzie HTML”) same zgód nie czytają. W ustawieniach zgody tagu wymagaj dodatkowej zgody: narzędzia reklamowe → `ad_storage`, analityczne → `analytics_storage`.

---

## F. Kod Google wklejony na sztywno

**Objawy:** w kodzie strony `gtag/js?id=AW-…` albo `gtag/js?id=G-…` obok kodu GTM; w nagraniu dwa `gtm.js`, `wasSetLate`; w Tag Assistant błąd o celu, którego nie da się przeanalizować; podwójne odsłony w GA4.

**Gdzie szukać w WordPressie:** plik `header.php` motywu, wtyczki do wstawiania kodu (np. WPCode, Insert Headers and Footers), Site Kit, wtyczki Google dla WooCommerce, ustawienia samego motywu.

**Przed usunięciem:** ustal, czyje jest konto z identyfikatora i czy ten kod nie liczy czegoś, czego GTM jeszcze nie liczy. Usuwasz dopiero, gdy GTM przejął jego rolę i przeszedł test.

---

## G. Konwersji za dużo albo za mało

| Objaw | Najczęstsza przyczyna | Jak sprawdzić |
|---|---|---|
| Dwa razy więcej konwersji w Ads | działanie z tagu Ads + to samo zaimportowane z GA4 jako główne | lista działań w Google Ads: dwa działania na jedno zdarzenie |
| Za dużo leadów | reguła na próbę wysłania; liczenie „Każda” przy leadach | nagranie: tag przy `gtm.formSubmit`; ustawienia działania |
| Zakupy dublowane | odświeżenie strony podziękowania; brak numeru zamówienia w tagu; dwie wtyczki wysyłające zakup | czy tag ma `transaction_id` (Ads po nim deduplikuje); kod strony |
| Podwójne odsłony w GA4 | dwa tagi z tym samym `G-…` albo kod na sztywno | eksport kontenera, część F |
| Konwersje w GA4, brak w Ads | brak tagu łączącego konwersje; odmowa zgód; ruch nie z reklam | nagranie; `gcs`; źródło ruchu w GA4 |
| Brak jakiegokolwiek ruchu w danych | kontener nieopublikowany; test w zwykłym oknie z blokerem reklam | wersja w GTM; test w czystej przeglądarce |

Różnica Ads ↔ GA4 do ok. 10–20% jest normalna (inne modele przypisania, zgody, blokery). Kilkadziesiąt procent to błąd pomiaru. Raporty GA4 mają opóźnienie do 24–48 h; DebugView i „Czas rzeczywisty” są natychmiast.

---

## H. WooCommerce

- **Konfiguracja:** GTM4WP → integracja WooCommerce włączona; zdarzenia GA4 (`view_item`, `add_to_cart`, `begin_checkout`, `purchase`) przychodzą do dataLayer. W Podglądzie sprawdź każde przy prawdziwym przejściu.
- **Brak zdarzenia przy dodaniu do koszyka:** zwykle winny motyw. Wtyczka rozpoznaje standardowe przyciski WooCommerce (klasy `add_to_cart_button`, `single_add_to_cart_button`); przycisk motywu bez nich nie wyśle zdarzenia. Naprawa: motyw albo deweloper. Reguła na tekst przycisku działa, ale psuje się przy zmianie tekstu i bierze stare dane koszyka (np. bez ilości). Po naprawie usuń taki obejściowy tag, inaczej zdarzenie pójdzie dwa razy.
- **Mikrokonwersje** (dodanie do koszyka, początek zamówienia) w Google Ads tylko jako działania dodatkowe, z wartością ze zmiennej, liczenie „Każda”. Główny zostaje zakup.
- **Dane klienta w dataLayer:** GTM4WP może wypychać dane zalogowanego klienta (imię, e-mail, telefon, adres). Nie mapuj ich do żadnego tagu poza polem danych użytkownika w konwersjach rozszerzonych (część D).

---

## I. Ustawienia działania konwersji w Google Ads

| Ustawienie | Lead (formularz, telefon) | Zakup |
|---|---|---|
| Liczenie | „Jedna” | „Każda” |
| Wartość | bez wartości albo stała (np. średnia wartość leada) | z tagu, różna dla każdej konwersji |
| Działanie | główne | główne |
| Okno konwersji | domyślne 30 dni; dłuższy namysł klienta — dłużej | domyślne |
| Atrybucja | domyślna (oparta na danych) | domyślna |

Konwersję liczy jedno źródło: tag Google Ads. Zdarzenia kluczowe z GA4 w Ads najwyżej jako dodatkowe, do porównania.

---

## J. Obszar roboczy, publikacja, cofanie

- Podgląd testuje obszar roboczy, czyli zmiany przed publikacją, a nie to, co jest na stronie.
- Przed zmianami: czy w obszarze roboczym nie ma cudzych zmian. Publikacja wysyła wszystko, co w nim jest.
- Publikacja: nazwa wersji mówi, co się zmieniło (np. „Konwersja formularza po udanym wysłaniu”).
- Coś się zepsuło po publikacji: w GTM w „Wersjach” opublikuj poprzednią wersję; to cofa zmiany w kilka minut.

---

## K. Słowniczek GTM (polski ↔ angielski)

| Po polsku | Po angielsku |
|---|---|
| reguła | trigger |
| zmienna warstwy danych | Data Layer Variable |
| zdarzenie niestandardowe | Custom Event |
| Inicjacja zgody – Wszystkie strony | Consent Initialization – All Pages |
| Zdarzenie inicjujące – Wszystkie strony | Initialization – All Pages |
| Tag łączący konwersje | Conversion Linker |
| Śledzenie konwersji Google Ads | Google Ads Conversion Tracking |
| Widoczność elementu | Element Visibility |
| Podgląd | Preview |
| Prześlij / Opublikuj | Submit / Publish |
| obszar roboczy | workspace |
