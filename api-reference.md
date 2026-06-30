---
title: API Reference
nav_order: 5
---

# API Reference

## Protokół MCP (`/mcp`)

Główny endpoint. Obsługuje komunikację asystenta AI z serwerem przez protokół MCP (streamable HTTP).

**Autoryzacja:** `Authorization: Bearer <token>` lub `?token=<token>`  
**Metody:** GET, POST, DELETE

---

## Narzędzia MCP

### Wiedza (`knowledge`)

#### `get_context`
Główne narzędzie. Przeszukuje bazę wiedzy, moduły i instrukcje jednocześnie.

```
Parametry:
  query: str         — pytanie lub słowa kluczowe
  tenant: str        — identyfikator tenanta (opcjonalnie)
```

#### `search_knowledge`
Przeszukuje surowe pliki bazy wiedzy (bez modułów i instrukcji).

```
Parametry:
  query: str
  tenant: str
  limit: int         — max wyników (domyślnie 5)
```

#### `get_instructions`
Zwraca standardy i instrukcje firmy dla danego tenanta.

```
Parametry:
  tenant: str
```

#### `refresh_knowledge`
Odświeża indeks plików bazy wiedzy (po dodaniu nowych plików).

```
Parametry:
  tenant: str
```

#### `knowledge_status`
Zwraca liczbę plików w bazie i czas ostatniego odświeżenia.

```
Parametry:
  tenant: str
```

#### `list_modules` / `get_module`
Lista dostępnych modułów produktowych lub treść konkretnego modułu.

```
list_modules:
  tenant: str

get_module:
  tenant: str
  name: str          — identyfikator modułu
```

---

### Generowanie umów

#### `generate_contracts_from_offer`
Główne narzędzie kontraktowe. Parsuje ofertę (PDF lub tekst), wyszukuje dane firmy i generuje wszystkie pasujące szablony DOCX.

```
Parametry:
  source: str        — "pdf" lub "text"
  content: str       — tekst oferty LUB ścieżka do pliku PDF
  tenant: str        — identyfikator tenanta (np. "umowy")
```

Zwraca: lista linków do pobrania wygenerowanych plików DOCX.

#### `lookup_company`
Wyszukuje dane firmy po NIP, numerze KRS lub nazwie.

```
Parametry:
  query: str         — NIP (10 cyfr), numer KRS lub nazwa firmy
```

Zwraca: nazwa, adres, NIP, REGON, forma prawna, status VAT.

Kolejność źródeł:
1. MF Whitelist API (podatki.gov.pl) — dla NIP
2. KRS API (api.legal.gov.pl) — dla KRS lub nazwy
3. DuckDuckGo — fallback

#### `fill_contract`
Wypełnia konkretny szablon DOCX podanymi zmiennymi.

```
Parametry:
  template: str      — nazwa pliku szablonu (np. "VNET_UM_CLOUD.docx")
  tenant: str
  variables: dict    — {nazwa_zmiennej: wartość}
```

#### `list_contract_templates`
Lista dostępnych szablonów DOCX dla danego tenanta.

```
Parametry:
  tenant: str
```

---

### Strony CMS

#### `search_pages`
Szuka stron na stronie VCN po nazwie lub słowach kluczowych. Cache 5 minut.

```
Parametry:
  query: str         — szukana fraza, np. "monitoring", "kontakt"
  limit: int         — liczba wyników (domyślnie 5)
```

Zwraca: lista stron z polem `slug`, `h1`, URL i fragmentem treści.

#### `get_page`
Pobiera pełną stronę po slugu — zwraca surowy HTML z inline styles.

```
Parametry:
  slug: str          — slug strony, np. "kontakt" lub "oferta/monitoring"
```

Zwraca: metadane strony + pełny `htmlContent` gotowy do edycji/naśladowania stylu.

**Workflow edycji strony:** `search_pages` → znajdź slug → `get_page` → przeanalizuj HTML → (opcjonalnie `upload_file` dla zdjęć) → wygeneruj nowy HTML → user wkleja w CMS.

---

### Projekty i klienci

#### `list_projects`
Lista projektów.

```
Parametry:
  tenant: str
  client_id: int     — filtr po kliencie (opcjonalnie)
```

#### `get_project_details`
Szczegóły projektu.

```
Parametry:
  project_id: int
```

#### `create_project`
Tworzy nowy projekt.

```
Parametry:
  name: str
  client_id: int
  tenant: str
  description: str   — opcjonalnie
```

#### `add_client`
Dodaje nowego klienta.

```
Parametry:
  name: str
  nip: str           — opcjonalnie
  tenant: str
```

#### `list_tenants`
Lista tenantów dostępnych dla bieżącego użytkownika.

---

### Pliki i strony

#### `upload_file`
Generuje link drag&drop do wgrania pliku (obraz, PDF, wideo).  
Zwraca URL do strony upload — użytkownik otwiera link w przeglądarce.

```
Parametry:
  filename: str      — docelowa nazwa pliku
  expires_minutes: int — czas ważności linku (domyślnie 30)
```

#### `create_upload_url` / `check_upload`
Alternatywny mechanizm uploadu PDF dla ofert.

```
create_upload_url:
  expires_minutes: int

check_upload:
  token: str         — token z create_upload_url
```

#### `publish_page`
Publikuje stronę HTML pod stałym adresem URL.

```
Parametry:
  title: str
  html: str          — pełny kod HTML (z CSS/JS inline)
  slug: str          — opcjonalnie (auto-generowany z tytułu)
  tenant: str
```

Zwraca: publiczny URL strony (`/p/{slug}`).

#### `update_page` / `list_pages` / `delete_page`
Zarządzanie opublikowanymi stronami.

```
update_page:
  slug: str
  title: str         — opcjonalnie
  html: str          — opcjonalnie

list_pages:
  tenant: str

delete_page:
  slug: str
```

#### `share_conversation`
Udostępnia bieżącą rozmowę jako publiczną stronę HTML.

```
Parametry:
  title: str
  messages: list     — lista {role, content}
```

Zwraca: URL do strony (`/c/{slug}`) i link do JSON (`/c/{slug}.json`).

#### `list_shared_conversations` / `delete_shared_conversation`
Lista lub usuwanie udostępnionych rozmów.

---

### Inne

#### `list_skills` / `get_skill`
Lista instrukcji workflow lub treść konkretnej instrukcji.

#### `report_issue`
Zapisuje problem — treść rozmowy + powiadomienie Telegram do administratora.

```
Parametry:
  title: str
  description: str
  messages: list     — opcjonalnie (kontekst rozmowy)
```

---

## REST API (wewnętrzne, dla UI)

Endpointy `/api/*` przeznaczone dla panelu Node.js. Nie wymagają tokenu JWT — przeznaczone do użytku lokalnego lub w sieci wewnętrznej.

### GET `/api/templates`
Lista szablonów DOCX.

**Query params:** `tenant` (wymagany)  
**Zwraca:** `{"templates": ["VNET_UM_CLOUD.docx", ...]}`

### POST `/api/company/lookup`
Wyszukanie danych firmy.

**Body (JSON):**
```json
{"query": "1234567890"}
```

**Zwraca:**
```json
{
  "name": "Firma Sp. z o.o.",
  "nip": "1234567890",
  "regon": "123456789",
  "address": "ul. Przykładowa 1, 00-001 Warszawa",
  "status_vat": "active"
}
```

### POST `/api/generate`
Generowanie umów z oferty.

**Body (JSON):**
```json
{
  "source": "text",
  "content": "treść oferty...",
  "tenant": "umowy"
}
```

### POST `/api/fill`
Wypełnienie pojedynczego szablonu.

**Body (JSON):**
```json
{
  "template": "VNET_UM_CLOUD.docx",
  "tenant": "umowy",
  "variables": {
    "firma_nazwa": "Firma Sp. z o.o.",
    "kwota_netto": "1000"
  }
}
```

---

## Endpointy publiczne

### GET `/health`
Health check.

**Zwraca:** `{"status": "ok", "server": "vcn-mcp"}`

### GET `/c/{slug}`
Udostępniona rozmowa (HTML).

### GET `/c/{slug}.json`
Udostępniona rozmowa (JSON) — dla AI.

### GET `/p/{slug}`
Opublikowana strona HTML.

### GET `/f/{filename}`
Plik statyczny (obraz, PDF, wideo).

### GET `/download/{filename}`
Pobranie wygenerowanego pliku DOCX.

### GET `/upload/{token}` + POST `/upload/{token}`
Strona drag&drop do wgrania pliku PDF (oferta).

### GET `/file/{token}` + POST `/file/{token}`
Strona drag&drop do wgrania pliku (asset strony).

---

## OAuth 2.0 Discovery

### GET `/.well-known/oauth-authorization-server`
Metadane serwera autoryzacji (RFC 8414).

### GET `/.well-known/oauth-protected-resource`
Metadane chronionego zasobu (RFC 9728) — używane przez Claude do auto-discovery.

### GET `/setup`
Strona samodzielnego generowania tokenu po zalogowaniu.
