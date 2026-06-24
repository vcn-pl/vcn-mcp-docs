---
title: Architektura
nav_order: 6
---

# Architektura systemu

## Diagram ogólny

```
┌─────────────────────────────────────────────────────────────┐
│                        Użytkownik                           │
│            (Claude.ai / ChatGPT / custom agent)             │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS + JWT token
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    VCN MCP Server                           │
│                   (Python / Starlette)                      │
│                                                             │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │  OAuth 2.0  │  │  MCP Router  │  │    REST API        │  │
│  │  /oauth/*   │  │    /mcp      │  │    /api/*          │  │
│  └─────────────┘  └──────┬───────┘  └────────┬──────────┘  │
│                          │                   │             │
│  ┌───────────────────────┼───────────────────┼───────────┐ │
│  │              Moduły biznesowe              │           │ │
│  │  ┌────────────┐ ┌─────────────┐ ┌─────────────────┐  │ │
│  │  │  knowledge │ │offer_parser │ │ company_lookup  │  │ │
│  │  └────────────┘ └─────────────┘ └─────────────────┘  │ │
│  │  ┌────────────┐ ┌─────────────┐ ┌─────────────────┐  │ │
│  │  │  contracts │ │  database   │ │     auth/oauth  │  │ │
│  │  └────────────┘ └─────────────┘ └─────────────────┘  │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────┐  ┌──────────────────────────┐│
│  │    SQLite vnet.db        │  │   Filesystem              ││
│  │  users, tokens, pages,  │  │  /knowledge/ /templates/  ││
│  │  projects, clients,     │  │  /generated/ /files/      ││
│  │  tool_log               │  │  /tmp/offers/             ││
│  └──────────────────────────┘  └──────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
          ┌──────────────────────────┐
          │     Telegram Bot         │
          │  powiadomienia, alerty   │
          └──────────────────────────┘

Osobno:
┌──────────────────────────────────────┐
│     UI Panel (Node.js / Express)     │
│  Zarządzanie wiedzą, projektami,     │
│  klientami, szablonami               │
└──────────────────────────────────────┘
```

---

## Komponenty

### 1. Serwer MCP (`mcp/`)

Rdzeń systemu napisany w Pythonie. Obsługuje dwa rodzaje żądań:

**Protokół MCP (`/mcp`)** — standardowy protokół komunikacji AI z narzędziami. Asystent AI wysyła żądania `call_tool`, serwer je obsługuje i zwraca wyniki.

**REST API (`/api/*`)** — wewnętrzne endpointy dla panelu UI (Node.js). Nie są chronione tokenem MCP — przeznaczone do użytku lokalnego lub w sieci wewnętrznej.

### 2. Moduły Python

| Moduł | Rola |
|---|---|
| `main.py` | Starlette app, routery, lifespan (start/stop serwera) |
| `server.py` | Definicje narzędzi MCP i ich obsługa |
| `api.py` | Wewnętrzne REST API dla UI |
| `auth.py` | Weryfikacja JWT, filtrowanie tenantów |
| `oauth.py` | Pełny flow OAuth 2.0 (authorize, token, login, register) |
| `users.py` | Schemat DB, haszowanie haseł |
| `setup.py` | Strona `/setup` — samodzielne generowanie tokenu |
| `config.py` | Loader konfiguracji z `config.json` + zmiennych środowiskowych |
| `context.py` | `ContextVar` dla bieżącego użytkownika (per-request) |
| `database.py` | Operacje SQLite — projekty, klienci, tenanty |
| `offer_parser.py` | Parser PDF/tekstu oferty → zmienne kontraktu |
| `company_lookup.py` | Wyszukiwanie firmy po NIP/KRS/nazwie |
| `contracts.py` | Wypełnianie szablonów DOCX, lista szablonów |
| `knowledge.py` | Przeszukiwanie bazy wiedzy, moduły, instrukcje |

### 3. Panel UI (`ui/`)

Oddzielna aplikacja Node.js (Express). Pozwala administratorowi:
- zarządzać bazą wiedzy
- dodawać projekty i klientów
- zarządzać szablonami umów
- monitorować aktywność

Komunikuje się z serwerem MCP przez wewnętrzne REST API.

### 4. Baza danych (`vnet.db`)

SQLite — jedna baza dla całego systemu. Multi-tenancy realizowane przez filtrowanie po kolumnie `tenant` lub przez strukturę katalogów `knowledge/`.

---

## Przepływ danych

### Logowanie użytkownika (OAuth 2.0)

```
Użytkownik → GET /oauth/authorize
           → GET /oauth/login (formularz)
           → POST /oauth/login (email + hasło)
           → Redirect z authorization_code
           → POST /oauth/token (code → access_token JWT)
           → JWT ważny 90 dni
```

### Wywołanie narzędzia MCP

```
AI (Claude) → POST /mcp  [Authorization: Bearer <jwt>]
            → auth.verify_token() → user dict + accessible_tenants
            → context.current_user.set(user)
            → MCP router → call_tool(name, args)
            → moduł biznesowy
            → tool_log INSERT (ts, tool, username, ok)
            → JSON result → AI
```

### Generowanie umów z oferty

```
Plik PDF/tekst → offer_parser.py
              → extracted variables (kwoty, daty, nazwy)
              → company_lookup.py (NIP → dane rejestrowe)
              → template_registry.json (typ umowy → szablony)
              → contracts.fill_contract() × N szablonów
              → N plików .docx w /generated/
              → linki do pobrania dla AI
```

---

## Multi-tenancy

System obsługuje wielu najemców (tenantów) w jednej instalacji:

- Każdy użytkownik ma pole `teams` w DB (JSON array, np. `["vnet", "tools"]`)
- Token JWT zawiera informację o teamach
- `auth.verify_token()` mapuje teamy → `accessible_tenants`
- Baza wiedzy podzielona katalogami: `/knowledge/<tenant>/`
- Szablony umów podzielone: `/templates/<tenant>/`

Konfiguracja valid tenantów w `config.json` → `tenants`.

---

## Integracja Telegram

Serwer wysyła powiadomienia do dwóch chatów:

| Chat | Cel |
|---|---|
| `TELEGRAM_CHAT_ID` | Prywatny — debug/status (milcząco) |
| `TELEGRAM_LOG_CHAT_ID` | Logi operacyjne — start/stop/dzienne podsumowanie |

Powiadomienia: start serwera, stop serwera, raporty błędów (`report_issue`), daily digest (godz. 23:59).

Implementacja jest fire-and-forget — błędy Telegrama nie przerywają obsługi żądania.

---

## Technologie

| Warstwa | Technologia |
|---|---|
| HTTP server | Uvicorn (ASGI) |
| Framework | Starlette |
| MCP | `mcp` SDK (streamable HTTP, nie SSE) |
| Baza danych | SQLite |
| Auth | JWT (PyJWT) + OAuth 2.0 |
| PDF parsing | pdfminer.six + pypdf (fallback) |
| DOCX templates | docxtpl (Jinja2 dla Word) |
| Node.js UI | Express + node-sqlite3-wasm |
| Deployment | systemd / OpenRC |
