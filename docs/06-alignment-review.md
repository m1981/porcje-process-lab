# ALIGN-001 — Kontrola zgodności z celem PM/PO i architekturą

## Werdykt

**Kierunek jest zgodny; aktualna implementacja jest tylko częściowo zgodna.**
Nie ma podstaw do ogłoszenia zakończenia pilota. Potrzebna jest korekta granic
warstw, obsługi istniejących zapisów i testów widoków, nie rozbudowa modelu procesu.

Audyt odnosi się do dwóch początkowych intencji użytkownika:

1. Prowadzenie produktu przez PM/PO bez znajomości całego repo, odpowiedzi na
   siedem pytań operacyjnych, istniejące zapisy, rozdzielenie produktu i procesu.
2. Wspólny format niezależny od pojedynczego narzędzia, z którego powstają
   niezależne statyczne HTML-e.

Uzgodniona architektura:

```text
Istniejące zapisy + niewielkie metadane
                  ↓
Walidacja struktury i znaczenia
                  ↓
Wspólny, znormalizowany obraz projektu
                  ↓
Niezależne generatory statycznych HTML
```

## Podstawa i odtwarzalność

Sprawdzono rzeczywiste pliki repo, a nie tylko opis architektury.
Punkt kontrolny kodu: `4755da33795d5d984d1b75394c872ab25ffd744b`.
To commit WIP, nie zatwierdzony baseline produktu. Raport zawiera manifest
SHA-256 sprawdzonych źródeł. Stan źródeł w tym punkcie był czysty.

Dowody: `evidence/reviews/ALIGN-001/observations.json`, `unit-tests.log`
oraz `reproduce.py`. Dane użyte w dodatkowych próbach są syntetyczne.
Brak akceptacji użytkownika, niezależnego audytu lub próby użyteczności z PM/PO.

Powtórzenie przy tym samym kodzie i zainstalowanych zależnościach:

```sh
python evidence/reviews/ALIGN-001/reproduce.py . /tmp/alignment-rerun-unique
```

Uruchamiacz wymaga nowego katalogu wyników, aby nie nadpisać poprzedniego dowodu.
Sam kod audytu przechowywany jest w późniejszym commicie niż testowany checkpoint;
przy odtwarzaniu można użyć kopii skryptu wobec checkoutu wskazanego checkpointu.

## Stan wykonania

- Istniejące testy jednostkowe: 64 wykonane, 64 PASS, 0 FAIL, 0 pominięć.
  W tym 20 testów produktu i 44 testy procesu.
- Dodatkowe próby poprawności odpowiedzi HTML: 2 wykonane, 2 FAIL.
- Kontrola obsługi istniejącego Markdown: czytnik zwrócił 0 rekordów;
  brak zadeklarowanego adaptera tego źródła. Nie oczekiwano zgadywania znaczenia prozy.
- Sześć zaplanowanych prób manualnych M01–M06: niewykonane.
- Brak rzeczywistych rekordów w `project/records/`, brak `tools/build.py`,
  `tools/check.py` i gotowego `dist/index.html` w punkcie kontroli.

Uruchomienie testów wymagało `PYTHONPATH=src:.`. Zwykłe discovery bez tego
ustawienia nie potrafiło zaimportować biblioteki `porcje`. To luka konfiguracji
roboczego repo, a nie niepowodzenie logiki skalowania. README poprawiono tak,
aby odróżniało stan wykonany od planowanego pakietu.

## Macierz siedmiu pytań

| Pytanie użytkownika | Stan w kodzie | Luka / warunek odbioru |
|---|---|---|
| Co teraz robimy i po co? | `work.lifecycle`, `purpose`, widoki strumieni | Brak rzeczywistych zapisów, priorytety bieżącej pracy i użyteczność niepotwierdzone. |
| Co gotowe, a co opisane lub zaplanowane? | Praca oddzielona od produktu pracy, weryfikacji i akceptacji | Potrzebny test końcowego HTML; dostarczenie pozostaje jawnie nieustalone. |
| Kto ma następny ruch? | Model rozpoznaje wymaganą decyzję i jej właściciela | HTML pokazuje czekającego wykonawcę jako następny ruch — AUD-UI-02. |
| Co wymaga mojej decyzji? | Osobny rekord i widok decyzji, kontrola deklarowanych uprawnień | Brak pełnego cyklu odpowiedź człowieka → zapis źródłowy → ponowna migawka. |
| Co blokuje? | Jawne `issue.blocks` i warunki gotowości | HTML myli otwarte ryzyko z blokadą — AUD-UI-01. |
| Jakie dowody potwierdzają ukończenie? | Podmiot, wersja, podstawa, polityka, konflikty | Obecny widok często kończy na surowym JSON; potrzebne czytelne kryterium/wynik/źródło. |
| Na co wpłynie zmiana? | Wpływ potwierdzony / potencjalny, jawny zakres analizy | Potrzebny test czytelnego wyjaśnienia i niepełnego pokrycia źródeł. |

## Macierz architektury

| Warstwa | Obserwacja | Wniosek |
|---|---|---|
| Istniejące zapisy + metadane | `processkit/io.py::load_records` czyta tylko `project/records/{product,process}/*.json`. Istnieją plikowe referencje i hashe produktów pracy. | Referencja do pliku nie oznacza jeszcze wykorzystania jego istniejących danych. Nie ma adaptera/mappingu i testu uniknięcia podwójnego statusowania. |
| Walidacja struktury i znaczenia | JSON Schema + `processkit/model.py::evaluate`; testy typów relacji, wersji, dowodów, konfliktów, uprawnień i historii. | Najbardziej rozwinięta warstwa. Sukces jej testów nie gwarantuje prawidłowości widoku. |
| Wspólny obraz projektu | `evaluate` zwraca słownik `view`; istnieje etykieta `project-view/0.1`. | `project.json` zapisywany jest dopiero przez renderer. Brak osobno walidowanego kontraktu wyjściowej migawki i jawnej proweniencji faktów. |
| Niezależne HTML | Renderer nie uruchamia serwera, nie wykonuje fetch i nie czyta repo; otrzymuje `records`, `view`, `meta`, `cases`, `test_report`. | Dobry fundament. Nie wykazano samodzielnego generowania różnych widoków z jednego pliku migawki. Renderer powiela część semantyki. |

Jeden pakiet Python i wspólny CSS nie naruszają niezależności generatorów.
Ważna jest zależność wyłącznie od kontraktu migawki, a nie liczba repozytoriów,
skryptów czy usług. Nie rekomenduje się mikroserwisów ani silnika workflow.

## Odtworzone defekty

### AUD-UI-01 — Ryzyko bez blokady pokazane jako blokada

Warunek: otwarte `issue` o `category=risk`, `blocks=[]`, aktywna praca.
Model: `blocked=false` dla pracy. Widok: umieszcza to ryzyko w sekcji
„Co blokuje?”. Przyczyna: renderer zbiera wszystkie otwarte issues zamiast
korzystać z wyliczonej listy rzeczywistych blokad.

Oczekiwane: ryzyko można pokazać w „Do uwagi”, ale nie jako istniejącą blokadę.
Status: **otwarty, odtworzony**. W tym audycie nie zmieniono kodu renderera.

### AUD-UI-02 — Zadanie czeka na PO, HTML wskazuje wykonawcę

Warunek: zadanie ma zaplanowany następny krok wykonawcy, ale wymaga pending
scope decision właściciela `human-po`. Model: `ready=false`,
`decision_owners=[human-po]`. Sekcja „Kto ma następny ruch?” pokazuje
`tester` i techniczny tekst `decision_pending`.

Oczekiwane: „Najpierw PO podejmuje decyzję DEC-001; potem wykonawca wykonuje
następny krok”. Rozdzielić ruch możliwy teraz i krok po odblokowaniu.
Status: **otwarty, odtworzony**. Nie utożsamiać tego z testem uprawnień.

## Doprecyzowania przed kontynuacją

### A. Jeden właściciel każdego faktu

Dla każdego rodzaju danych wskazać autorytatywne miejsce zapisu. Wymaganie
pozostaje tam, gdzie jest dziś; wynik testu pozostaje w raporcie runnera.
Metadane dodają ID, rolę, relacje i lokalizator, a nie kopię całej treści.
Pochodne statusy i skróty tworzy kompilator, nie człowiek.

JSON jako natywny zapis nowej decyzji może być sensowny. Wygenerowany JSON
migawki nie jest natomiast nową bazą źródłową. Nie wymuszać ani przepisywania
wszystkiego do JSON, ani masowej migracji do YAML front matter.
Adaptery obsługują wybrane, jawne formaty, bez wnioskowania o ukończeniu ze słów
„pass” i „done” w prozie. Jeden fakt nie ma dwóch równorzędnych ręcznych kopii.

### B. Osobny kontrakt migawki

Zachować cztery warstwy. Jawny odczyt/adapter i normalizacja są częścią
kompilacji; format źródła można sprawdzić przed normalizacją, a znaczenie
wspólnych rekordów po normalizacji. Całość musi zakończyć się przed rendererem.

Proponowany kontrakt wyjścia: `project-view/0.1`, z rekordami, relacjami,
ocenami, powodami ocen, listą braków, źródłami, aktualnym ruchem i wpływem zmian.
Kontekst obejmuje rewizje/digesty wejść, czas migawki, wersję schematu i reguł
oraz zakres obsłużonych źródeł. Sam `valid=true` nie oznacza kompletności repo.

Renderer otrzymuje tylko tę migawkę; nie ustala ponownie blokad, gotowości
ani akceptacji. Może formatować, filtrować i sortować już rozstrzygnięte dane.
Przykładowy test granicy: wygenerować dwa różne widoki w katalogu, w którym
nie ma źródłowego repo ani dostępu do niego, korzystając z tej samej migawki.

### C. Metadane techniczne nie są formularzem dla PM/PO

Hashe, wersje runnera, manifesty i kontekst środowiska są obowiązkiem narzędzi.
Człowiek podaje cel, decyzję, zakres i potrzebne odpowiedzialności. Liczba pól
wewnętrznego JSON nie wyznacza liczby ręcznych czynności wymaganych od człowieka.

Strumień produktu i strumień procesu zachowują osobne podsumowania. Zakończenie
narzędzia nie może zwiększać wskaźnika gotowości aplikacji kuchennej.
Połączenia między strumieniami pozostają jawne, nie znikają przez filtrowanie.

### D. Statyczny odczyt, jawny zapis decyzji

Produkt Porcje jest biblioteką bez własnego CLI i GUI. Statyczny HTML służy
człowiekowi do prowadzenia projektu; nie jest UI samej biblioteki.
Decyzja wraca do autorytatywnego zapisu przez uzgodniony sposób edycji/review.
Generator nie akceptuje produktu w imieniu człowieka. Nie dodawać pozornego
przycisku zatwierdzenia, który działa tylko w stanie przeglądarki.

## Uzupełnienie bramki testowej — do implementacji

Te punkty nie są zaliczonymi testami. Rozszerzają plan o kontrakt odbioru
architektury; nie zwiększają liczby wykonanych testów jednostkowych.

| ID | Warunek akceptacji | Aktualny dowód |
|---|---|---|
| AG01 | Istniejący zapis jest wykorzystany bez ręcznej kopii danych w osobnym rejestrze. | Luka adaptera; do wykonania. |
| AG02 | Zmiana autorytatywnego faktu wymaga jednej edycji; oba widoki zmieniają się spójnie. | Do wykonania; koszt z człowiekiem w M06. |
| AG03 | Migawka ma osobny walidowany kontrakt i daje się przenieść bez repo. | Do wykonania. |
| AG04 | Dwa generatory czytają tę samą migawkę bez importu reguł oceny i bez źródeł. | Do wykonania. |
| AG05 | Oba renderery zachowują poprawne znaczenie „blokuje” i „następny ruch”. | Dwie próby FAIL, wymagana regresja po poprawce. |
| AG06 | Budowa HTML nie modyfikuje danych źródłowych; zmiana szablonu nie zmienia ocen. | Do wykonania. |
| AG07 | Stary dowód, brak źródła, konflikt i niepełne pokrycie nie stają się zielonym ukończeniem w HTML. | Częściowe testy modelu; brak pełnej ścieżki. |
| AG08 | Narzędzie ukończone ≠ produkt ukończony; zależność między strumieniami pozostaje widoczna. | Częściowe testy modelu; do sprawdzenia widoki. |
| AG09 | Odpowiedzi na siedem pytań zawierają krótkie uzasadnienie i prowadzą do właściwego źródła. | Automatyczny test kontraktu odpowiedzi do wykonania; M01 manualny niewykonany. |
| AG10 | Odbiorca rozpoznaje czas migawki i zakres braków; decyzja człowieka ma pełny powrót do źródła. | Do wykonania; bez domniemanej akceptacji. |

Proponowane cele manualne (M01 i M06), a nie wyniki: siedem poprawnych odpowiedzi
bez czytania repo w około trzy minuty; brak podwójnego ręcznego statusowania.
Czasy i dopuszczalny koszt powinien ocenić rzeczywisty użytkownik.

## Decyzja techniczna audytu

Nie rozszerzać obecnie liczby typów rekordów ani zakresu normy. Najpierw:
wykorzystanie istniejących źródeł → autonomiczna migawka → renderowanie bez
powielania semantyki → sprawdzenie siedmiu odpowiedzi. Dotychczasowe prace nad
wersjami, dowodami i rozdzieleniem strumieni zachować. Dwa defekty pozostają
jawne do chwili poprawki i ponownego testu. Audyt nie jest akceptacją produktu.
