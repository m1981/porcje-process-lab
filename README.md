# porcje-process-lab

> **Stan: import zachowanych materiałów projektu, nie gotowe wydanie 0.1.**
> W tym repozytorium nie ma jeszcze kodu biblioteki, generatorów HTML ani
> wykonywalnych testów. Nie należy utożsamiać publikacji dokumentacji
> z ukończeniem, weryfikacją ani akceptacją produktu.

## Cel

Lokalny pilot prowadzenia projektu przez PM/PO bez znajomości całego repozytorium.
Produkt testowy to **Porcje**: biblioteka skalująca ilości składników do zadanej
liczby porcji, bez własnego CLI i GUI. Statyczne HTML-e mają przedstawiać pracę,
decyzje, blokady, dowody i wpływ zmian; nie są interfejsem biblioteki.

Dwa oddzielne strumienie: **produkt** oraz **proces i narzędzia**.
Ukończenie narzędzia nie oznacza ukończenia produktu.

## Co faktycznie opublikowano

- [Raport ALIGN-001](docs/06-alignment-review.md) — zachowany audyt zgodności
  z celem PM/PO i architekturą, z opisem dwóch wykrytych defektów.
- [Obserwacje ALIGN-001](evidence/reviews/ALIGN-001/observations.json) —
  zachowany zapis wyników audytu i manifest analizowanych wtedy plików.

Oba pliki audytu zaimportowano bez zmiany treści.

## Ważne ograniczenia historycznych dowodów

Raporty odnoszą się do wcześniejszego punktu kontrolnego
`4755da33795d5d984d1b75394c872ab25ffd744b`. Jego kod i historia Git nie były
dostępne podczas tego importu i nie zostały odtworzone w tym repozytorium.
Wymienionych w raporcie `unit-tests.log` i `reproduce.py` również nie odzyskano.

Zapis o 64 przechodzących testach jest informacją z historycznego raportu,
**nie wynikiem bieżącego uruchomienia**. Na podstawie dostępnych plików nie można
powtórzyć tamtej próby. Opisane defekty nie zostały tutaj naprawione ani ponownie
przetestowane. Nie przeprowadzono odbioru użyteczności przez człowieka.

## Uzgodniona architektura — do wykonania

```text
Istniejące zapisy + niewielkie metadane
                  ↓
Walidacja struktury i znaczenia
                  ↓
Wspólny, znormalizowany obraz projektu
                  ↓
Niezależne generatory statycznych HTML
```

Każdy fakt ma jedno autorytatywne miejsce zapisu. Migawka i HTML są pochodne.
Oceny blokad, następnego ruchu, weryfikacji i akceptacji powstają przed
renderowaniem. Generator nie interpretuje ich ponownie.

## Zamrożony zakres planowanego wydania 0.1

1. Biblioteka Porcje: skalowanie bez konwersji jednostek i automatycznego
   zaokrąglania, z ilościami reprezentowanymi przez ułamki.
2. Jawny odczyt jednego formatu kart Markdown i raportów testów JSON;
   minimalne metadane i relacje bez ręcznego kopiowania statusów.
3. Samodzielna migawka `project-view.json` według kontraktu `project-view/0.1`.
4. Dwa generatory: `index.html` do prowadzenia projektu oraz `details.html`
   do wyjaśniania dowodów, decyzji i wpływu zmian. Działają z samej migawki.
5. Testy produktu, semantyki i końcowych HTML-i, w tym regresje AUD-UI-01
   oraz AUD-UI-02 i scenariusz zmiany unieważniającej aktualność dowodu.

Powyższe punkty są **planem**, nie listą dostarczonych funkcji.
Nie ma jeszcze instrukcji uruchomienia, ponieważ wykonywalne narzędzia
nie zostały dołączone. Pełny tekst standardu i załączone diagramy nie są
częścią tego importu.
