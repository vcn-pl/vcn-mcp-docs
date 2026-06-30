# Przewodnik developera

## Struktura projektu

```
vnet-mcp/
├── mcp/                  # Serwer Python (produkcja)
│   ├── main.py           # App + routery + lifespan
│   ├── server.py         # MCP tools definitions
│   ├── api.py            # REST API dla UI
│   ├── auth.py           # JWT verification
│   ├── oauth.py          # OAuth 2.0 flow
│   ├── users.py          # DB schema, hashing
│   ├── setup.py          # /setup page
│   ├── config.py         # Config loader
│   ├── context.py        # ContextVar current_user
│   ├── database.py       # SQLite helpers
│   ├── offer_parser.py   # PDF/text → variables
│   ├── company_lookup.py # NIP/KRS API
│   ├── contracts.py      # DOCX template fill
│   ├── knowledge.py      # Knowledge base search
│   ├── config.json       # Runtime config
│   ├── template_registry.json
│   └── requirements.txt
│
├── ui/                   # Panel admina (Node.js)
│   └── src/
│       ├── index.js
│       ├── config.js
│       ├── auth.js
│       ├── db.js
│       ├── mcp_client.js
│       └── telegram.js
│
├── services/             # Pliki deployment
│   ├── vcn-mcp.service
│   ├── vcn-mcp.initd
│   ├── vcn-ui.service
│   ├── vcn-ui.initd
│   ├── vcn-mcp.env.example
│   └── config.example.json
│
├── scripts/
│   └── config_check.py   # Walidacja config.json
│
├── tests/                # Testy jednostkowe
│   ├── test_auth.py
│   ├── test_offer_parser.py
│   ├── test_company_lookup.py
│   └── conftest.py
│
├── docs/                 # Ta dokumentacja
├── memory/               # Notatki dla agenta AI
├── Makefile              # Skróty deweloperskie
└── pytest.ini            # Konfiguracja testów
```

---

## Uruchamianie lokalnie

### Wymagania

- Python 3.12+
- virtualenv

### Setup

```bash
python3.12 -m venv venv
source venv/bin/activate
pip install -r mcp/requirements.txt
```

Stwórz minimalny config:

```bash
mkdir -p /tmp/vnet-dev/{base,logs,templates/dev,generated,files,tmp/offers}
```

`mcp/config.json` → patrz [configuration.md](configuration.md) (sekcja deweloperska).

Zmienne środowiskowe:
```bash
export SECRET_KEY=dev-only-not-for-prod
export TELEGRAM_BOT_TOKEN=
export TELEGRAM_CHAT_ID=0
export TELEGRAM_LOG_CHAT_ID=0
```

### Start

```bash
cd mcp
python3 main.py
```

---

## Testy

### Uruchomienie

```bash
pytest
```

lub przez Makefile:
```bash
make test
```

### Konfiguracja (`pytest.ini`)

```ini
[pytest]
testpaths = tests
asyncio_mode = auto
```

`asyncio_mode = auto` — wszystkie coroutines testowane bez `@pytest.mark.asyncio`.

### Co testowane

| Plik | Co testuje |
|---|---|
| `test_auth.py` | Weryfikacja JWT — ważny token, wygasły, zły klucz, brak pól |
| `test_offer_parser.py` | Parser oferty — kwoty, daty, pozycje, typ umowy |
| `test_company_lookup.py` | Lookup firmy — parsowanie adresów, mapowanie pól MF/KRS |

### Fixtura (`conftest.py`)

Zawiera wspólne fixture pytest, np. tymczasową bazę SQLite.

---

## Jak dodać nowe narzędzie MCP

1. **Napisz logikę** w odpowiednim module (lub nowym pliku w `mcp/`)

2. **Zarejestruj narzędzie** w `mcp/server.py`:

```python
@mcp_server.tool()
async def moje_narzedzie(param: str) -> str:
    """Opis narzędzia widoczny dla AI."""
    user = current_user.get()
    # ... logika
    return "wynik"
```

3. **Dodaj log** (opcjonalnie, ale rekomendowane):

```python
_log_call("moje_narzedzie", user["username"], ok=True)
```

4. **Napisz test** w `tests/test_*.py`

5. Zrestartuj serwer

---

## Jak dodać nowy typ umowy

1. Stwórz szablon DOCX z placeholderami Jinja2:
   - `{{firma_nazwa}}`, `{{kwota_netto}}`, `{{data_zawarcia}}` itd.

2. Skopiuj do `/templates/<tenant>/NazwaPliku.docx`

3. Dodaj wpis do `mcp/template_registry.json`:
   ```json
   "moj_typ": {
     "description": "Opis umowy",
     "keywords": ["słowo1", "słowo2"],
     "templates": ["NazwaPliku.docx"]
   }
   ```

4. Słowa kluczowe są używane przez `offer_parser` do doboru szablonów.

---

## Jak dodać treści do bazy wiedzy

1. Utwórz plik `.json` lub `.md` w `/knowledge/<tenant>/`

2. Format JSON:
   ```json
   {
     "title": "Tytuł dokumentu",
     "content": "Treść..."
   }
   ```

3. Wywołaj `refresh_knowledge` (przez Claude lub REST API) lub zrestartuj serwer.

---

## Makefile

```bash
make test      # Uruchom pytest
make lint      # Sprawdź kod (flake8/ruff)
make run       # Uruchom serwer deweloperski
make install   # Instalacja zależności
```

---

## Zależności

### Python (`mcp/requirements.txt`)

| Pakiet | Rola |
|---|---|
| `starlette` | ASGI framework |
| `uvicorn` | ASGI server |
| `mcp` | MCP SDK |
| `pyjwt` | JWT encode/decode |
| `pdfminer.six` | PDF text extraction (layout-aware) |
| `pypdf` | PDF fallback parser |
| `docxtpl` | Jinja2 templating dla DOCX |
| `httpx` | Async HTTP client (company lookup) |

### Node.js (`ui/package.json`)

| Pakiet | Rola |
|---|---|
| `express` | HTTP framework |
| `express-session` | Session management |
| `node-sqlite3-wasm` | SQLite w Node.js |
| `ejs` | Templating HTML |
| `bcryptjs` | Haszowanie haseł (UI) |
| `pdf-parse` | Parsowanie PDF |
| `mammoth` | Parsowanie DOCX |

---

## Znane ograniczenia

- **Brak masowej rewokacji tokenów** — zmiana hasła nie unieważnia aktywnych tokenów JWT; wymaga ręcznego DELETE z `oauth_tokens`.
- **SQLite** — brak ACID w przypadku wielu jednoczesnych zapisów. Przy dużym ruchu rozważ PostgreSQL.
- **Brak rate limiting** — `/oauth/login` podatne na brute-force. Zalecane nginx `limit_req`.
- **Rejestracja KRS** — KRS `/wyszukiwanie` endpoint zwraca 404. Używamy DDG jako fallback.
- **PDF skanowane** — `pdfminer.six` nie obsługuje OCR. Skanowane PDF dają złe wyniki.
