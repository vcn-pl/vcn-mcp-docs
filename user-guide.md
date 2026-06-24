# Przewodnik użytkownika

> Ten przewodnik wyjaśnia, jak korzystać z VCN MCP bez znajomości programowania.

## Co to jest VCN MCP?

Wyobraź sobie, że masz asystenta AI (np. Claude), który zamiast tylko rozmawiać, może zajrzeć do firmowych dokumentów, sprawdzić dane firmy w rejestrach i automatycznie przygotować umowę. To właśnie robi VCN MCP — jest mostem między asystentem AI a systemami Twojej firmy.

---

## Jak się połączyć?

### Krok 1: Uzyskaj dostęp

Skontaktuj się z administratorem systemu — to on tworzy konta użytkowników.

### Krok 2: Podłącz do Claude

W ustawieniach Claude:
1. Znajdź sekcję "Integrations" lub "MCP"
2. Kliknij "Add MCP server"
3. Wpisz adres serwera: `https://twoja-domena.pl/mcp`
4. Kliknij "Connect" — pojawi się formularz logowania
5. Zaloguj się swoim kontem VCN MCP

Po połączeniu Claude będzie widział dodatkowe narzędzia i będzie mógł korzystać z firmowej bazy wiedzy.

---

## Co możesz zrobić?

### Pytać o firmową wiedzę

Claude może odpowiadać na pytania bazując na dokumentach i materiałach Twojej firmy.

**Przykład:**
> „Jakie są warunki gwarancji dla produktu X?"
> „Co zawiera oferta standardowa?"
> „Jakie dokumenty są wymagane przy podpisaniu umowy?"

### Sprawdzać dane firm

Wystarczy podać NIP, numer KRS lub nazwę firmy.

**Przykład:**
> „Sprawdź dane firmy o NIP 1234567890"
> „Czy firma ABC Sp. z o.o. jest aktywnym płatnikiem VAT?"
> „Podaj adres siedziby i REGON firmy XYZ"

Dane pobierane są z oficjalnych rejestrów:
- Ministerstwo Finansów (whitelist VAT)
- KRS (Krajowy Rejestr Sądowy)

### Generować umowy

To najbardziej zaawansowana funkcja. Wystarczy wkleić lub wgrać ofertę — serwer automatycznie:
1. Odczyta kluczowe dane (kwoty, daty, nazwy, usługi)
2. Sprawdzi dane firmy klienta w rejestrach
3. Dobierze odpowiednie szablony umów
4. Wypełni je i przygotuje gotowe pliki Word (.docx)

**Jak to zrobić:**

Opcja A — wklej tekst oferty:
> „Wygeneruj umowy na podstawie tej oferty: [wklejasz treść]"

Opcja B — wgraj plik PDF:
> „Wygeneruj umowy z pliku PDF"

Claude wyśle Ci link do strony, gdzie możesz przeciągnąć plik. Po wgraniu — Claude pobierze go i wygeneruje umowy.

### Publikować strony

Claude może stworzyć i opublikować stronę HTML pod stałym adresem URL. Przydatne np. do wysyłania podsumowań, raportów czy prezentacji klientom.

**Przykład:**
> „Stwórz stronę z podsumowaniem oferty dla klienta ABC i prześlij mi link"

### Udostępniać rozmowy

Możesz podzielić się treścią rozmowy z Claude — generuje publiczny link do strony z transkryptem.

**Przykład:**
> „Udostępnij tę rozmowę jako link do strony"

---

## Typowy scenariusz: generowanie umowy

**Sytuacja:** Masz ofertę od klienta i chcesz szybko przygotować umowę.

1. W Claude wpisujesz:
   > „Wygeneruj umowy na podstawie oferty od firmy ABC Sp. z o.o., NIP 1234567890. Przedmiot: wdrożenie systemu monitoringu, kwota 15 000 zł netto, czas realizacji 30 dni."

2. Claude:
   - Sprawdza firmę w rejestrze MF/KRS
   - Rozpoznaje typ umowy (wdrożenie → szablony wdrożeniowe)
   - Generuje 4 pliki DOCX (umowa główna + 3 załączniki)
   - Podaje linki do pobrania

3. Ty pobierasz pliki i sprawdzasz ich treść.

Cały proces zajmuje ok. 30 sekund.

---

## Często zadawane pytania

**Czy moje rozmowy są prywatne?**  
Tak — rozmowy nie są publiczne, chyba że sam je udostępnisz przez `share_conversation`.

**Jak długo ważne jest połączenie?**  
Sesja trwa 90 dni. Jeśli wygaśnie, Claude poprosi o ponowne zalogowanie — wystarczy powtórzyć kroki z Krok 2.

**Co się dzieje, gdy wgrany plik PDF jest nieczytelny?**  
Serwer spróbuje odczytać tekst z PDF. Jeśli PDF jest skanem (obrazek), wyodrębnienie danych może być niedokładne — lepiej wkleić tekst ręcznie.

**Czy mogę używać narzędzia z ChatGPT?**  
Tak — protokół MCP jest obsługiwany przez Claude i inne asystenty. Procedura podłączenia może się różnić.

**Nie mam konta — co robię?**  
Skontaktuj się z administratorem — musi dodać Twoje konto do systemu.

---

## Kontakt i zgłaszanie problemów

Jeśli coś nie działa, powiedz Claude:
> „Zgłoś problem z opisem: [opisujesz problem]"

Claude zapisze problem i wyśle powiadomienie do administratora.
