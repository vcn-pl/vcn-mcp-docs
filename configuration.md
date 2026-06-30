---
title: Konfiguracja
nav_order: 4
---

# Konfiguracja

## Pliki konfiguracyjne

System konfigurowany jest przez dwa źródła:
1. `config.json` — ścieżki, ustawienia serwera, tenanty (domyślna lokalizacja: `/etc/vcn-mcp/config.json`)
2. Zmienne środowiskowe — sekrety (tokeny, klucze)

---

## config.json

Pełna struktura:

```json
{
  "paths": {
    "vcn_root": "/srv/vnet-mcp/data",
    "knowledge_root": "/srv/vnet-mcp/data/base",
    "log_dir": "/var/log/vcn-mcp"
  },
  "server": {
    "url": "https://twoja-domena.pl",
    "name": "vcn-mcp",
    "version": "26.07.05",
    "host": "0.0.0.0",
    "port": 8000
  },
  "auth": {
    "token_expire_days": 90
  },
  "tenants": ["vnet", "tools"],
  "integrations": {
    "pages_api_url": "https://twoja-domena.pl/api/pages/"
  }
}
```

### paths

| Klucz | Opis | Przykład |
|---|---|---|
| `vcn_root` | Główny katalog danych | `/srv/vnet-mcp/data` |
| `knowledge_root` | Baza wiedzy (pliki JSON/MD) | `/srv/vnet-mcp/data/base` |
| `log_dir` | Katalog logów | `/var/log/vcn-mcp` |

Z `vcn_root` wywodzone są automatycznie:
- `{vcn_root}/templates/` — szablony DOCX
- `{vcn_root}/generated/` — wygenerowane pliki DOCX
- `{vcn_root}/files/` — pliki statyczne (obrazy, PDF)
- `{vcn_root}/tmp/offers/` — tymczasowe pliki ofert

### server

| Klucz | Opis | Uwagi |
|---|---|---|
| `url` | Publiczny URL serwera | Musi zgadzać się z URL w OAuth discovery |
| `name` | Nazwa serwera MCP | Pojawia się w health check |
| `version` | Wersja aplikacji | Format `YY.MM.DD` |
| `host` | Adres bind | `0.0.0.0` = wszystkie interfejsy |
| `port` | Port TCP | Domyślnie 8000 |

### auth

| Klucz | Opis | Domyślnie |
|---|---|---|
| `token_expire_days` | Ważność tokenów JWT (dni) | 90 |

### integrations

| Klucz | Opis |
|---|---|
| `pages_api_url` | URL REST API stron CMS — używane przez narzędzia `search_pages` i `get_page` |

### tenants

Lista identyfikatorów tenantów dostępnych w systemie. Musi zgadzać się z katalogami w `knowledge_root` i `templates/`.

Przykład: `["vnet", "tools", "client_xyz"]`

---

## Zmienne środowiskowe

Sekrety **nigdy** nie powinny trafiać do `config.json`. Przekazywane przez zmienne środowiskowe (plik `.env` lub systemd `EnvironmentFile`).

### Wymagane

| Zmienna | Opis |
|---|---|
| `SECRET_KEY` | Klucz podpisywania JWT (min. 256 bitów). Generuj: `python3 -c "import secrets; print(secrets.token_hex(32))"` |
| `TELEGRAM_BOT_TOKEN` | Token bota Telegram (z BotFather) |
| `TELEGRAM_CHAT_ID` | ID czatu prywatnego (debug) |
| `TELEGRAM_LOG_CHAT_ID` | ID czatu logów operacyjnych |

### Opcjonalne

| Zmienna | Opis | Domyślnie |
|---|---|---|
| `MCP_CONFIG_PATH` | Ścieżka do `config.json` | katalog binarki |
| `LOG_DIR` | Nadpisuje `paths.log_dir` z config.json | `{vcn_root}/tmp/logs` |
| `MCP_HOST` | Adres bind serwera | wartość z config.json |
| `MCP_PORT` | Port serwera | wartość z config.json |
| `SERVER_URL` | Publiczny URL serwera | wartość z config.json |
| `PAGES_API_URL` | URL API stron CMS (nadpisuje `integrations.pages_api_url`) | — |

---

## Template Registry (`mcp/template_registry.json`)

Definiuje typy umów, słowa kluczowe i przypisane szablony DOCX.

```json
{
  "wdrozenie_abonament": {
    "description": "Umowa wdrożenia + abonament",
    "keywords": ["wdrożenie", "montaż", "instalacja", "kamera"],
    "templates": [
      "VNET_UMOWA_WDROŻENIE_i_ABONAMENT.docx",
      "VNET_WDROŻENIE_ABONAMENT_ZAŁĄCZNIK 1_SPECYFIKACJA USŁUG.docx"
    ]
  },
  "cloud_saas": {
    "description": "Umowa SaaS / Cloud",
    "keywords": ["saas", "cloud", "abonament"],
    "templates": ["VNET_UM_CLOUD.docx"]
  }
}
```

**Jak dodać nowy typ umowy:**
1. Stwórz szablon `.docx` z placeholderami `{{zmienna}}`
2. Skopiuj do `/templates/<tenant>/`
3. Dodaj wpis do `template_registry.json`

Placeholdery używają składni Jinja2. Przykłady zmiennych:
- `{{firma_nazwa}}` — nazwa firmy klienta
- `{{firma_nip}}` — NIP klienta
- `{{kwota_netto}}` — wartość netto
- `{{data_zawarcia}}` — data zawarcia umowy

---

## Baza danych

Ścieżka: `{vcn_root}/vnet.db` (SQLite).

Schemat tworzony automatycznie przy pierwszym starcie przez `users._ensure_db_schema()`.  
Nie wymaga ręcznej konfiguracji.

Szczegóły → [database.md](database.md).

---

## Środowisko deweloperskie

Minimalna konfiguracja do lokalnego developmentu:

**config.json:**
```json
{
  "paths": {
    "vcn_root": "/tmp/vnet-dev",
    "knowledge_root": "/tmp/vnet-dev/base",
    "log_dir": "/tmp/vnet-dev/logs"
  },
  "server": {
    "url": "http://localhost:8000",
    "name": "vcn-mcp-dev",
    "version": "dev",
    "host": "127.0.0.1",
    "port": 8000
  },
  "auth": {
    "token_expire_days": 1
  },
  "tenants": ["dev"]
}
```

**Zmienne środowiskowe (opcjonalne dla dev):**
```bash
export SECRET_KEY=dev-secret-not-for-production
export TELEGRAM_BOT_TOKEN=    # puste = Telegram wyłączony
export TELEGRAM_CHAT_ID=0
export TELEGRAM_LOG_CHAT_ID=0
```

Gdy `TELEGRAM_BOT_TOKEN` jest puste lub `TELEGRAM_CHAT_ID=0`, powiadomienia są pomijane (serwer obsługuje to cicho).
