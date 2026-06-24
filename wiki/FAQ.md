# FAQ — Najczęstsze pytania

---

## Konto i dostęp

**Jak uzyskać konto?**  
Skontaktuj się z administratorem systemu — to on tworzy konta użytkowników.

**Jak długo ważne jest połączenie?**  
90 dni. Po wygaśnięciu Claude poprosi o ponowne zalogowanie — wystarczy powtórzyć procedurę łączenia.

**Zapomniałem hasła — co robić?**  
Skontaktuj się z administratorem. Brak automatycznego resetowania hasła (wersja podstawowa).

**Czy mogę podłączyć kilka asystentów AI?**  
Tak — każde podłączenie to osobna sesja. Możesz używać tego samego konta na wielu urządzeniach.

**Czy dostęp można zablokować?**  
Tak — administrator może unieważnić sesję z bazy. Natychmiastowa blokada. Jeśli podejrzewasz wyciek hasła — powiadom admina i zmień hasło.

---

## Korzystanie z Claude

**Claude nie widzi narzędzi VCN — co robię?**  
1. Sprawdź, czy MCP server jest dodany w ustawieniach Claude
2. Sprawdź, czy URL jest poprawny (bez spacji, bez `/` na końcu)
3. Sprawdź, czy jesteś zalogowany — w ustawieniach MCP powinna być widoczna aktywna sesja
4. Spróbuj: „Jakie narzędzia masz dostępne?" — Claude powinien wylistować narzędzia VCN

**Czy muszę ręcznie mówić Claude żeby użył narzędzi?**  
Nie. Claude sam rozpoznaje kiedy skorzystać z narzędzi VCN na podstawie kontekstu rozmowy. Możesz jednak być precyzyjny jeśli chcesz.

**Claude mówi że nie może użyć narzędzia — dlaczego?**  
Możliwe przyczyny:
- Wygasła sesja → odłącz i podłącz ponownie w ustawieniach Claude
- Brak uprawnień do zasobu → sprawdź z adminem swoje grupy (tenanty)
- Problem z serwerem → sprawdź `/health` lub skontaktuj z adminem

---

## Generowanie umów

**Z jakich formatów ofert mogę korzystać?**  
- Tekst wklejony bezpośrednio w rozmowie (najlepszy wynik)
- Plik PDF z tekstem (nie skan) — wgrywany przez link drag&drop

**PDF generuje złe wyniki — dlaczego?**  
PDFy będące skanami dokumentów (zdjęciami) nie dają się poprawnie odczytać. Parser potrzebuje „prawdziwego" tekstu w PDF. Rozwiązanie: skopiuj tekst ręcznie i wklej.

**Jak sprawdzić, że dane firmy są poprawne?**  
Przed generowaniem możesz osobno sprawdzić:
> „Sprawdź firmę o NIP 1234567890"
Claude wyświetli dane z rejestru — możesz je zweryfikować.

**Nie wiem jaki typ umowy wybrać — serwer dobierze sam?**  
Tak. System analizuje słowa kluczowe z oferty i dobiera najlepiej pasujące szablony. Możesz też podać wprost: „Wygeneruj umowę wdrożeniową" lub „Umowę SaaS".

**Ile plików DOCX zostanie wygenerowanych?**  
Zależy od typu umowy. Przykład dla wdrożenia: umowa główna + 3 załączniki (specyfikacja, wykaz osób, UPPDO) = 4 pliki.

**Przez ile czasu dostępne są wygenerowane pliki?**  
Do restartu serwera. Pobierz pliki od razu po wygenerowaniu.

---

## Strony i udostępnianie

**Co to jest „opublikowana strona"?**  
Strona HTML pod stałym publicznym adresem URL. Dostępna dla każdego kto ma link — bez logowania.

**Kto może zobaczyć udostępnioną rozmowę?**  
Każdy kto ma link. Uważaj co udostępniasz — mogą być tam wrażliwe dane firmy.

**Jak usunąć opublikowaną stronę lub rozmowę?**  
> „Usuń stronę o adresie /p/abc123"  
> „Usuń udostępnioną rozmowę abc123"

---

## Baza wiedzy

**Skąd Claude wie co zawiera firmowa wiedza?**  
Pliki bazy wiedzy (JSON/Markdown) przechowywane są na serwerze. Claude używa narzędzia `get_context` lub `search_knowledge` żeby je przeszukać.

**Baza wiedzy jest nieaktualna — co robię?**  
Powiedz Claude: „Odśwież bazę wiedzy" lub „refresh_knowledge". Jeśli dodano nowe pliki przez panel admina — odświeżenie jest potrzebne.

**Nie widzę wszystkich zasobów — dlaczego?**  
Twoje konto może być przypisane tylko do wybranych grup (tenantów). Skontaktuj się z administratorem żeby sprawdzić swoje uprawnienia.

---

## Bezpieczeństwo i prywatność

**Czy Anthropic (twórcy Claude) widzi moje dane?**  
Treść rozmów zarządzana jest przez Anthropic zgodnie z ich polityką prywatności. Serwer VCN MCP nie przesyła danych do Anthropic — jedynie odpowiada na zapytania Claude.

**Czy dane firm podawanych przy NIP lookup są bezpieczne?**  
NIP i dane firm to informacje publiczne. Serwer odpytuje publiczne rejestry MF i KRS — to samo co możesz zrobić ręcznie na stronie podatki.gov.pl.

**Administrator widzi co piszę do Claude?**  
Nie — treść rozmów zostaje po stronie Claude/Anthropic. Administrator widzi tylko: które narzędzia były wywołane, kiedy i przez kogo.

---

## Problemy techniczne

**Serwer nie odpowiada**  
Sprawdź: `https://twoja-domena.pl/health` — jeśli nie odpowiada, skontaktuj się z administratorem.

**Jak zgłosić błąd?**  
Powiedz Claude: „Zgłoś problem: [opis problemu]" — wyśle powiadomienie do administratora.  
Lub kontakt bezpośredni z adminem.

**Połączenie jest wolne**  
Narzędzie `lookup_company` odpytuje zewnętrzne API (MF, KRS) — przy wolnym łączu lub niedostępności API może to potrwać kilka sekund.
