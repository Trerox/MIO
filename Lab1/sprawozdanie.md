# Sprawozdanie — Laboratorium 1: Podstawy pracy z agentem

**Aplikacja i model:** Antigravity · Gemini 3.8 Flash · High  
**Link do repozytorium:** https://github.com/Trerox/MIO  
**Link do końcowego raportu:** [exercise-summary.json](project/results/exercise-summary.json)

---

### Odpowiedzi na pytania do sprawozdania

#### 1. Jaki wynik zlecono, jakie zmiany były dozwolone i o jakie informacje do odbioru poproszono agenta?
- **Wynik:** Zlecono wygenerowanie raportu podsumowującego etykiety dla zbioru `project/data/exercise.csv`, zapisanego w pliku `project/results/exercise-summary.json`.
- **Dozwolone zmiany:** W Etapie 1 obowiązywał całkowity zakaz wprowadzania zmian w kodzie skryptu, danych wejściowych oraz plikach testowych. W Etapie 2 dozwolona była wyłącznie zmiana etykiety jednej wiadomości w pliku `exercise.csv`.
- **Informacje do odbioru:** Poproszono agenta o wskazanie lokalizacji pliku raportu, odczytanie jego zawartości w sesji (w tym liczebności wiadomości i poszczególnych etykiet), przedstawienie dokładnego polecenia i kodu wyjścia procesu oraz uruchomienie gotowego testu jednostkowego wraz z interpretacją wyniku.

#### 2. Jak powstał raport? Wskaż wejście, polecenie, narzędzie, środowisko wykonania i wynik procesu. Dodaj link do końcowego raportu i wyjaśnij jego liczebności.
- **Wejście:** Plik `project/data/exercise.csv` zawierający 8 wiadomości e-mail.
- **Polecenie:**
  ```powershell
  Set-Location project; py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
  ```
- **Narzędzie:** Narzędzie agenta `run_command` wykonujące powłokę PowerShell w aplikacji Antigravity.
- **Środowisko wykonania:** Lokalny system operacyjny Windows użytkownika, interpreter Pythona `Python 3.14.0` (`C:\Users\jakub\AppData\Local\Programs\Python\Python314\python.exe`), katalog roboczy `C:\Users\jakub\Desktop\MIO\Lab1\project`.
- **Wynik procesu:** Kod wyjścia `0` (sukces), na standardowym wyjściu zwrócono:
  ```text
  Report saved: C:\Users\jakub\Desktop\MIO\Lab1\project\results\exercise-summary.json
  Messages: 8; labels: {'ham': 5, 'spam': 3}
  ```
- **Link do końcowego raportu:** [exercise-summary.json](project/results/exercise-summary.json)
- **Wyjaśnienie liczebności:**
  W początkowym uruchomieniu (Etap 1) raport wykazał łącznie `8` wiadomości (`total_messages: 8`), w tym `5` jako `ham` oraz `3` jako `spam`. W końcowym raporcie (po celowej modyfikacji jednej etykiety w Etapie 2) łączna liczba wiadomości pozostała bez zmian (`8`), natomiast rozkład etykiet wynosi po równo: `4` wiadomości `ham` i `4` wiadomości `spam`. Suma etykiet (4 + 4) w pełni pokrywa się z łączną liczbą wiadomości w zbiorze.

#### 3. Który fragment dokumentacji lub kodu wyjaśniłeś? Podaj plik i miejsce, własne wyjaśnienie oraz mały przypadek: przewidywanie, sposób sprawdzenia i wniosek.
- **Plik i miejsce:** [`project/describe_data.py`](project/describe_data.py), linie 26–27.
- **Fragment:**
  ```python
  if row["label"] not in counts:
      raise ValueError(f"Record {number}: label must be ham or spam.")
  ```
  *(gdzie `counts = {"ham": 0, "spam": 0}`)*.
- **Własne wyjaśnienie:** Podczas przetwarzania każdego wiersza pliku CSV skrypt weryfikuje, czy wartość w kolumnie `label` znajduje się w słowniku `counts`. Ponieważ dozwolone są wyłącznie etykiety `"ham"` oraz `"spam"`, jakakolwiek inna wartość skutkuje natychmiastowym przerwaniem programu i rzuceniem wyjątku `ValueError`.
- **Mały przypadek:** Podanie w pliku CSV etykiety wielkimi literami: `"SPAM"` zamiast `"spam"`.
- **Przewidywanie:** Program zgłosi błąd, ponieważ słownik zawiera klucze małymi literami, a pobrana wartość nie jest normalizowana funkcją `.lower()`.
- **Sposób sprawdzenia:** Wykonanie bezpiecznego testu z użyciem tymczasowego pliku CSV zawierającego rekord z etykietą `"SPAM"`.
- **Wniosek:** Skrypt rzucił wyjątek `ValueError: Record 1: label must be ham or spam.`, co potwierdziło przewidywanie — wielkość liter ma kluczowe znaczenie i program ściśle pilnuje zdefiniowanych etykiet.

#### 4. Jak uruchomiono gotowy test, jaki był wynik i czego dotyczyło sprawdzenie?
- **Polecenie uruchomienia:**
  ```powershell
  Set-Location project; py -3 -m unittest discover -s tests -v
  ```
- **Wynik testu:**
  Test zakończył się sukcesem (kod wyjścia `0`, status `OK`):
  ```text
  test_cli_writes_expected_counts (test_report.ReportTest.test_cli_writes_expected_counts)
  The CLI saves a report for the intended input with counts 3 / 2 / 1. ... ok

  ----------------------------------------------------------------------
  Ran 1 test in 0.086s

  OK
  ```
- **Czego dotyczyło sprawdzenie:**
  Test jednostkowy (`project/tests/test_report.py`) uruchomił program `describe_data.py` na osobnym, syntetycznym pliku testowym `project/tests/fixtures/messages.csv` (zawierającym dokładnie 3 wiadomości: 2 `ham` i 1 `spam`). Test weryfikował w katalogu tymczasowym:
  1. Czy proces zakończył się bez błędu (kod powrotu `0`),
  2. Czy utworzono poprawny plik raportu w formacie JSON,
  3. Czy pole `input_file` wskazuje na użyty plik danych,
  4. Czy liczebności są równe zdefiniowanym z góry wartościom oczekiwanym: 3 wiadomości łącznie, 2 `ham`, 1 `spam`.
  Test ten nie korzystał z pliku `exercise.csv` ani nie nadpisywał raportu `exercise-summary.json`.

#### 5. Jaką etykietę zmieniono w etapie 2? Jakiej zmiany liczebności oczekiwałeś, jakie operacje wybrałeś i dlaczego? Porównaj wynik przed zmianą i po nowym wykonaniu.
- **Zmieniona etykieta:**
  W pliku `project/data/exercise.csv` zmodyfikowano etykietę dla wiadomości o identyfikatorze `message_id`: **`exercise-001`**.
  - Stara etykieta: `ham`
  - Nowa etykieta: `spam`
  Pozostałe pola rekordu oraz inne pliki projektu nie były modyfikowane.
- **Oczekiwana zmiana liczebności:**
  - `total_messages`: bez zmian — nadal **8** (nie dodano ani nie usunięto rekordów),
  - `ham`: spadek o 1 — z **5** na **4**,
  - `spam`: wzrost o 1 — z **3** na **4**.
- **Wybrane operacje i uzasadnienie:**
  Zamiast ręcznej edycji pliku wynikowego JSON (co byłoby fałszowaniem wyniku procesu i pozostawiłoby niespójną sumę kontrolną), wybrano **ponowne uruchomienie skryptu** poleceniem:
  ```powershell
  Set-Location project; py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
  ```
  **Uzasadnienie:** Ponowne uruchomienie programu gwarantuje, że nowy raport jest bezpośrednią i wiarygodną reprezentacją zmodyfikowanych danych wejściowych, a także automatycznie wylicza nową sumę kontrolną SHA-256 (`input_sha256`), poświadczającą zmianę pliku wejściowego.
- **Porównanie wyników przed zmianą i po nowym wykonaniu:**

| Parametr w raporcie | Przed zmianą (Etap 1) | Po zmianie (Etap 2) | Różnica / Interpretacja |
|---|---|---|---|
| `total_messages` | 8 | 8 | Bez zmian (ta sama liczba maili) |
| `label_counts.ham` | 5 | 4 | Spadek o 1 (zmiana z ham na spam) |
| `label_counts.spam` | 3 | 4 | Wzrost o 1 (nowy spam) |
| `input_sha256` | `95e9049fe9f1...` | `7dc8080df480...` | Nowa suma kontrolna pliku CSV |
| Środowisko wykonania | Python 3.14.0 (Windows) | Python 3.14.0 (Windows) | To samo lokalne środowisko wykonawcze |
