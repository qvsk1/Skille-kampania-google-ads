# Trzy skille do Google Ads w Claude

Autor: Oskar Mazur, oskarmazur.pl

Trzy skille, które prowadzą Cię krok po kroku od zera do pierwszej kampanii Google Ads w wyszukiwarce. Skill to instrukcja dla Claude: po jej wgraniu Claude wie, o co pytać, co sprawdzić i w jakiej kolejności Cię prowadzić.

Skille nie łączą się z Twoimi kontami i nie proszą o hasła. W Google Ads, GA4 i Google Tag Managerze klikasz Ty, a Claude mówi, gdzie wejść, co wpisać, i sprawdza wynik po tym, co mu wkleisz albo pokażesz na zrzucie ekranu. Gdy nie jest pewny, jak nazywa się przycisk w panelu, poprosi o zrzut.

## Kolejność

| Nr | Skill | Co dostajesz |
|---|---|---|
| 1 | `google-ads-1-pomiar` | Działający baner zgód, GA4 i konwersję Google Ads, sprawdzone testem (także odmowy zgód i pustego formularza) |
| 2 | `google-ads-2-oferta` | Porównanie z trzema konkurentami (co masz, czego nie masz, przewagi) i frazy sprawdzone w Planerze słów kluczowych: ile osób szuka i ile kosztuje kliknięcie |
| 3 | `google-ads-3-kampania` | Kampanię na tych frazach: grupy reklam, słowa kluczowe, wykluczenia, reklamy, przejście przez panel ekran po ekranie i listę kontrolną |

Idź po kolei. Każdy skill zapisuje wynik w pliku `moja-firma.md`, a następny go czyta, więc na te same pytania odpowiadasz raz.

## Co jest w repozytorium

```
google-ads-1-pomiar/
  SKILL.md          skill
  zaawansowane.md   wiedza na trudniejsze przypadki (Claude sięga po nią sam, gdy trzeba)
google-ads-2-oferta/
  SKILL.md
google-ads-3-kampania/
  SKILL.md
```

`zaawansowane.md` przydaje się, gdy pomiar już był ustawiony i coś się nie zgadza: konwersji jest za dużo albo za mało, formularz nie ma strony podziękowania, baner zgód działa dziwnie, chcesz konwersje rozszerzone albo porównać dwa landingi. Jest tam też skrypt, którym Claude czyta nagranie sesji z Tag Assistant. Ten plik musi leżeć w tym samym folderze co `SKILL.md`.

## Pobierz

[google-ads-1-pomiar.zip](https://github.com/qvsk1/Skille-kampania-google-ads/releases/latest/download/google-ads-1-pomiar.zip) · [google-ads-2-oferta.zip](https://github.com/qvsk1/Skille-kampania-google-ads/releases/latest/download/google-ads-2-oferta.zip) · [google-ads-3-kampania.zip](https://github.com/qvsk1/Skille-kampania-google-ads/releases/latest/download/google-ads-3-kampania.zip)

Instrukcja wgrania niżej, w „Jak zainstalować”.

## Czego potrzebujesz

- Konta Claude z włączonymi skillami (płatny plan).
- Strony internetowej, na której sprzedajesz albo zbierasz zapytania, i dostępu do jej ustawień.
- Konta Google. Google Tag Manager, GA4 i Google Ads założysz za darmo po drodze.
- Budżetu na reklamy. Baner zgód Cookiebot jest darmowy dla jednej domeny do 50 podstron; większa strona płaci abonament.

## Jak zainstalować

**Aplikacja Claude (przeglądarka albo komputer)**

1. Pobierz trzy gotowe paczki:
   - [google-ads-1-pomiar.zip](https://github.com/qvsk1/Skille-kampania-google-ads/releases/latest/download/google-ads-1-pomiar.zip)
   - [google-ads-2-oferta.zip](https://github.com/qvsk1/Skille-kampania-google-ads/releases/latest/download/google-ads-2-oferta.zip)
   - [google-ads-3-kampania.zip](https://github.com/qvsk1/Skille-kampania-google-ads/releases/latest/download/google-ads-3-kampania.zip)

   Nie rozpakowuj ich.
2. W ustawieniach, w części „Capabilities”, włącz wykonywanie kodu i tworzenie plików.
3. Wejdź w „Customize” → „Skills” → „+” → „Create skill” → „Upload a skill”, wgraj ZIP i włącz skill przełącznikiem. Powtórz dla trzech plików.

Nazwy w aplikacji mogą być po polsku albo się zmienić. Szukaj ustawień skilli i opcji wgrania własnego.

**Claude Code**

Skopiuj trzy foldery w całości do `~/.claude/skills/`. Skill uruchamiasz, wpisując na przykład `/google-ads-1-pomiar`.

## Jak używać

Najwygodniej pracować w jednym folderze na komputerze, w Cowork albo w Claude Code. Claude sam zapisuje tam plik `moja-firma.md` i czyta go przy kolejnym skillu.

W zwykłym czacie plik nie przechodzi między rozmowami. Na końcu każdego skilla dostaniesz jego treść do zapisania; załącz ją na początku następnej rozmowy.

Żeby zacząć, napisz po prostu, czego chcesz:

- „Ustaw mi pomiar pod Google Ads.” albo „Sprawdź, czy mój formularz się liczy.”
- „Sprawdź, jak się reklamować i co robi konkurencja.” i wklej link do swojej strony. Skill 2 działa też w pustym folderze.
- „Przygotuj moją pierwszą kampanię Google Ads.”

Każdy skill kończy się konkretnym wynikiem i mówi, który jest następny.

## Ważne przed testami

Podgląd w Google Tag Managerze wysyła prawdziwe dane: GA4 zapisze testową konwersję, a formularz wyśle prawdziwy mail do Twojej firmy. Rób jeden test na każdą ścieżkę i wpisuj w wiadomości „TEST”.

## Licencja

MIT — możesz używać, zmieniać i udostępniać dalej, także komercyjnie, z zachowaniem informacji o autorze. Treść w pliku [LICENSE](LICENSE).
