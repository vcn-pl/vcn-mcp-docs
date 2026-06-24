# Narzędzia MCP — kompletna lista

> Każde narzędzie jest wywoływane automatycznie przez Claude na podstawie kontekstu rozmowy. Możesz też poprosić Claude wprost o użycie konkretnego narzędzia.

---

## Baza wiedzy

### `get_context`
**Najważniejsze narzędzie.** Przeszukuje jednocześnie bazę wiedzy, moduły produktowe i instrukcje firmowe.

**Kiedy używać:**
> „Co wiesz o produkcie X?"  
> „Jakie są warunki serwisu?"  
> „Opisz ofertę standardową"

**Parametry:** słowa kluczowe (zapytanie), opcjonalnie: tenant

---

### `search_knowledge`
Przeszukuje tylko surowe pliki bazy wiedzy (bez modułów i instrukcji).

**Kiedy używać:** gdy chcesz znaleźć coś konkretnego w dokumentach firmowych.

---

### `get_instructions`
Pobiera standardy i wytyczne firmy dla danej grupy (tenanta).

---

### `refresh_knowledge`
Odświeża indeks plików bazy wiedzy. Przydatne po dodaniu nowych dokumentów.

**Kiedy używać:** po aktualizacji bazy wiedzy przez administratora.

---

### `knowledge_status`
Pokazuje liczbę plików w bazie i czas ostatniej aktualizacji.

---

### `list_modules` / `get_module`
Lista modułów produktowych lub treść konkretnego modułu.

---

## Generowanie umów

### `generate_contracts_from_offer`
**Główne narzędzie kontraktowe.** Przetwarza ofertę (tekst lub PDF) i generuje wszystkie pasujące umowy.

**Kiedy używać:**
> „Wygeneruj umowy na podstawie tej oferty"  
> „Przygotuj dokumenty dla klienta ABC"

**Jak podać ofertę:**
- Wklejasz tekst bezpośrednio w rozmowie
- Wgrywasz plik PDF (Claude wyśle Ci link do strony upload)

**Wynik:** linki do gotowych plików DOCX do pobrania.

---

### `lookup_company`
Wyszukuje dane firmy z publicznych rejestrów.

**Kiedy używać:**
> „Sprawdź firmę o NIP 1234567890"  
> „Czy ABC Sp. z o.o. jest aktywnym płatnikiem VAT?"

**Źródła danych (w kolejności):**
1. Ministerstwo Finansów — whitelist VAT (dla NIP)
2. KRS API — Krajowy Rejestr Sądowy
3. DuckDuckGo — fallback gdy rejestry nie odpowiadają

**Zwracane dane:** nazwa, adres, NIP, REGON, forma prawna, status VAT.

---

### `fill_contract`
Wypełnia konkretny szablon DOCX podanymi zmiennymi.

**Kiedy używać:** gdy chcesz ręcznie kontrolować, który szablon i z jakimi danymi wypełnić.

---

### `list_contract_templates`
Lista dostępnych szablonów umów.

---

## Projekty i klienci

### `list_projects`
Lista projektów, opcjonalnie filtrowanych po kliencie.

### `get_project_details`
Szczegóły projektu.

### `create_project`
Tworzy nowy projekt.

**Kiedy używać:**
> „Utwórz projekt dla klienta ABC o nazwie 'Wdrożenie 2026'"

### `add_client`
Dodaje nowego klienta.

**Kiedy używać:**
> „Dodaj klienta ABC Sp. z o.o. z NIP 1234567890"

### `list_tenants`
Lista grup (tenantów) dostępnych dla Twojego konta.

---

## Pliki i strony

### `upload_file`
Generuje jednorazowy link do wgrania pliku (obraz, PDF, wideo, inne).

**Kiedy używać:**
> „Wgraj zdjęcie do strony"  
> „Potrzebuję wgrać logo klienta"

Claude wyśle Ci link — otwierasz go w przeglądarce i przeciągasz plik.  
Obsługiwane formaty: JPG, PNG, GIF, WEBP, SVG, PDF, MP4, WEBM, MP3, WAV.

---

### `publish_page`
Publikuje stronę HTML pod stałym publicznym adresem URL.

**Kiedy używać:**
> „Stwórz stronę z podsumowaniem oferty i daj mi link"  
> „Opublikuj raport jako stronę internetową"

**Wynik:** publiczny URL w formacie `https://twoja-domena.pl/p/{slug}`

---

### `update_page`
Aktualizuje treść opublikowanej strony.

### `list_pages`
Lista opublikowanych stron.

### `delete_page`
Usuwa opublikowaną stronę.

---

### `share_conversation`
Udostępnia bieżącą rozmowę jako publiczną stronę HTML.

**Kiedy używać:**
> „Udostępnij tę rozmowę jako link"  
> „Wygeneruj link do tego podsumowania"

**Wynik:**
- Strona HTML: `https://twoja-domena.pl/c/{slug}`
- Dane JSON (dla AI): `https://twoja-domena.pl/c/{slug}.json`

---

### `list_shared_conversations`
Lista udostępnionych rozmów.

### `delete_shared_conversation`
Usuwa udostępnioną rozmowę.

---

## Inne

### `list_skills` / `get_skill`
Lista instrukcji workflow lub treść konkretnej instrukcji.

### `report_issue`
Zgłasza problem — zapisuje opis i wysyła powiadomienie do administratora przez Telegram.

**Kiedy używać:**
> „Zgłoś problem: narzędzie lookup_company zwraca błąd dla NIP 123..."

---

## Jak poprosić Claude o użycie konkretnego narzędzia?

Zwykle nie musisz — Claude sam wybiera narzędzia. Ale możesz być precyzyjny:

> „Użyj narzędzia lookup_company żeby sprawdzić NIP 5213657029"  
> „Wywołaj generate_contracts_from_offer z poniższą ofertą..."  
> „Przeszukaj bazę wiedzy (search_knowledge) pod kątem: warunki gwarancji"
