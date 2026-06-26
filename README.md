# Wzory dokumentów GJS

Repozytorium zawiera cztery bazowe szablony Worda kancelarii **Greloch Jaworski Stanis** oraz skrypty do generowania na ich podstawie gotowych dokumentów. Pliki bazowe niosą firmową identyfikację wizualną (logo, stopkę, motyw, numerację wielopoziomową) i **proprietarne style akapitowe GJS**. Treść przykładowa została usunięta — pliki nie zawierają danych klientów.

## Pliki bazowe

| Plik | Typ dokumentu | Język |
|------|---------------|-------|
| `GSJ_UmowaPOL.docx` | umowa / kontrakt | PL |
| `GSJ_UmowaANG.docx` | umowa / kontrakt | EN |
| `GSJ_MemorandumPOL.docx` | memorandum / opinia / nota | PL |
| `GSJ_MemorandumANG.docx` | memorandum / opinia / nota | EN |

Szablony **nie są wymienne**. Umowy i memoranda mają inne mapy stylów, inną strukturę numeracji i inne elementy (np. blok podpisów). Zawsze dobieraj plik bazowy do typu i języka dokumentu.

---

## Dla zespołu GJS (jak z tego korzystać)

Nie musisz znać Gita ani uruchamiać żadnych skryptów. Wystarczy, że w rozmowie z Claude:

1. wskażesz, który dokument tworzysz (np. „umowa najmu po polsku", „memorandum po angielsku"),
2. podasz treść albo dane, które mają wejść do dokumentu,
3. odbierzesz gotowy plik `.docx` linkiem do pobrania.

Claude sam pobierze właściwy szablon z tego repozytorium, wstrzyknie treść z poprawnymi stylami GJS i przepakuje plik z zachowaniem logo, stopki i numeracji. Jedyne, co musisz zrobić od strony technicznej, to upewnić się, że Claude ma dostęp do tego repozytorium (publiczny adres `git clone`).

**Czego nie robić:** nie proś o „dokument w stylu GJS od zera". Dokument generowany bez pliku bazowego traci logo, stopkę i numerację stron — to łamie cały sens tych wzorów.

---

## Dla osoby utrzymującej repo / dla agenta (procedura techniczna)

Zasada nadrzędna: **buduj NA szablonie, nie od zera.** Skopiuj plik bazowy, wstrzyknij treść do `word/document.xml`, przypisując istniejące ID stylów GJS, i przepakuj z flagą `--original` (zachowuje logo, stopkę, motyw, numerację).

```bash
# 1. Pobierz repo
git clone <ADRES_REPO>
cd <REPO>

# 2. Skopiuj właściwy plik bazowy do katalogu roboczego
cp GSJ_UmowaPOL.docx robocze.docx

# 3. Rozpakuj (skrypty z docx skill)
python scripts/office/unpack.py robocze.docx unpacked/

# 4. Edytuj unpacked/word/document.xml
#    - usuń treść placeholderów
#    - wstaw treść z <w:pStyle w:val="..."/> wskazującym styl GJS (mapa niżej)

# 5. Przepakuj z zachowaniem brandingu
python scripts/office/pack.py unpacked/ wynik.docx --original robocze.docx
```

**Zasady przy edycji XML:**

- Numerację dają wyłącznie style nagłówków przez powiązany `numPr` — **nie wpisuj ręcznie** `§`, `1.`, `(a)` itd.
- Nie zastępuj stylów ręcznym formatowaniem (pogrubienia, rozmiary czcionki). Używaj ID stylów.
- Nie generuj dokumentu narzędziem typu `docx-js` „w stylu GJS" — straci branding.

---

## Mapa stylów — typ treści → ID stylu

Przypisuj przez `<w:pStyle w:val="ID"/>`. ID są wspólne dla wersji PL i EN.

### Umowa

| Element treści | Styl (ID) |
|----------------|-----------|
| Tytuł umowy | `GJSAgreementTitle` |
| Podtytuł | `GJSAgreementSubTitle` |
| „pomiędzy" / „a", dane stron (wyśrodkowane) | `GJSCentered` |
| Tekst wstępu / preambuły | `GJSDocTxtL` |
| Numerowana lista preambuły / recitals — (1), a., i. | `GJS1` |
| Klauzula — poziom 1 (1.) | `GJSH1` |
| Klauzula — poziom 2 (1.1) | `GJSH2` |
| Klauzula — poziom 3 (1.1.1) | `GJSH3` |
| Klauzula — poziom 4 ((a)) | `GJSH4` |
| Klauzula — poziom 5 ((i)) | `GJSH5` |
| Tekst akapitu wciętego pod klauzulą | `GJSDocTxtL1` |
| Definicja (hasło + treść) | `GJSDef` |
| Załącznik — treść | `GJSSchedule` |
| Załącznik — tytuł | `GJSSchTitle` |
| Tytuł spisu treści | `GJSTOCTiltle` |
| Wpis spisu treści | `Spistreci1` |
| Blok podpisów — lewy / prawy (pogrubienie) | `GJSBoldLeft` / `GJSBoldRight` |

**Drugi, niezależny strumień numeracji — `GJSAltH1`–`GJSAltH8`.** Używaj, gdy w jednym dokumencie potrzebny jest osobny ciąg numeracji równolegle do głównego (np. odrębna numeracja w załączniku, by nie kontynuowała numeracji umowy głównej).

### Memorandum

| Element treści | Styl (ID) |
|----------------|-----------|
| Tytuł memorandum | `GJSTitle` |
| Etykieta pola nagłówka (Do / Od / Data / Dotyczy) | `Memorandum-Dane-Opis` |
| Wartość pola nagłówka | `Memorandum-dane` |
| Treść główna — nagłówki i akapity | te same `GJSH1`–`GJSH5`, `GJSDocTxtL`, `GJSDocTxtL1` co w umowie |

### Schemat numeracji (z definicji szablonu)

- `GJSH1`–`GJSH9`: `1.` → `1.2` → `1.2.3` → `(a)` → `(i)` → `(A)` …
- `GJS1`: `(1)` → `a.` → `i.` …

---

## Znane ograniczenia (nie są błędami)

- **`GJSTOCTiltle`** — literówka w nazwie stylu jest celowa. To wewnętrzne ID szablonu; używaj dokładnie tak, jak zapisano.
- **`GJSSchedule`** — renderuje numer i tytuł załącznika bez separatora (luka `w:suff`/`lvlText` w `numbering.xml`). Znany efekt, nie błąd aplikacji stylu.
- **Pole daty w memorandum** jest dynamiczne — przy otwarciu w Wordzie pokazuje bieżącą datę, nie statyczny placeholder.
- **Spis treści (TOC)** jest poprawnie zdefiniowany i wypełnia się w Wordzie. Regeneracja TOC poza Wordem (np. przez LibreOffice headless) bywa zablokowana w środowisku sandbox.

## Czyszczenie danych przed publikacją

Pliki bazowe w tym repo zostały oczyszczone z treści przykładowej (dane stron, numery rejestrowe, nazwiska, klient). Jeżeli dodajesz nowy plik bazowy: usuń wszystkie dane konkretnych spraw i osób, zachowując każde przypisanie stylu GJS. W publicznym repozytorium historia Gita jest trwała — danych z commita nie da się usunąć bez przepisania historii.
