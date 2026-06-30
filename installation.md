---
title: Instalacja
nav_order: 3
---

# Instalacja i wdrożenie

## Wymagania

| Składnik | Wersja minimalna |
|---|---|
| Python | 3.12 |
| Node.js (UI) | 22 |
| SQLite | 3.x (system) |
| System | Linux (systemd lub OpenRC) |

### Zależności Python (mcp/requirements.txt)

```
starlette
uvicorn
mcp          # MCP SDK
pyjwt        # JWT
pdfminer.six # PDF parsing
pypdf        # PDF fallback
docxtpl      # DOCX templates (Jinja2)
httpx        # async HTTP client
```

---

## Instalacja krok po kroku

### 1. Skopiuj binarki

```bash
cp dist/vcn-mcp /usr/local/bin/vcn-mcp
cp dist/vcn-ui  /usr/local/bin/vcn-ui
chmod +x /usr/local/bin/vcn-mcp /usr/local/bin/vcn-ui
```

### 2. Uruchom wizard konfiguracyjny

```bash
vcn-mcp --setup
```

Wizard interaktywnie:
- tworzy `/etc/vcn-mcp/config.json`
- tworzy katalogi danych (`/srv/vnet-mcp/data/` i podkatalogi)
- tworzy `/var/log/vcn-mcp/`
- tworzy pusty plik bazy danych (`vnet.db`)
- generuje `/etc/vcn-mcp/vcn-mcp.env` z losowym `SECRET_KEY` (chmod 600)

```bash
vcn-ui --setup
```

Wizard UI tworzy `/etc/vcn-mcp/vcn-ui.env` z losowym `UI_SESSION_SECRET`.

### 3. Uzupełnij env pliki

```bash
nano /etc/vcn-mcp/vcn-mcp.env
```

Wymagane do uzupełnienia:
```bash
TELEGRAM_BOT_TOKEN=your_token
TELEGRAM_CHAT_ID=your_personal_chat_id
TELEGRAM_LOG_CHAT_ID=your_log_chat_id
ADMIN_EMAIL=admin@firma.pl
```

Opcjonalnie (integracja z CMS):
```bash
PAGES_API_URL=https://twoja-domena.pl/api/pages/
```

Szczegóły wszystkich opcji → [configuration.md](configuration.md).

### 4. Pierwszy start

```bash
vcn-mcp
```

Serwer powinien:
- wypisać `✅ All checks passed`
- uruchomić się na `http://0.0.0.0:8000`
- wysłać powiadomienie Telegram o starcie

Sprawdź: `curl http://localhost:8000/health`

### 5. Pierwszy użytkownik (admin)

Otwórz `http://localhost:8000/setup` — zaloguj się i wygeneruj pierwszy token.

---

## Wdrożenie produkcyjne

### systemd (Ubuntu/Debian/RHEL)

```bash
cp services/vcn-mcp.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable vcn-mcp
systemctl start vcn-mcp
systemctl status vcn-mcp
```

Sprawdź logi:
```bash
journalctl -u vcn-mcp -f
```

### OpenRC (Alpine Linux)

```bash
cp services/vcn-mcp.initd /etc/init.d/vcn-mcp
chmod +x /etc/init.d/vcn-mcp
rc-update add vcn-mcp default
rc-service vcn-mcp start
```

---

## Panel UI (Node.js)

### Konfiguracja

UI czyta `config.json` z `/etc/vcn-mcp/config.json` (lub przez `MCP_CONFIG_PATH`).  
Env plik: `/etc/vcn-mcp/vcn-ui.env` — generowany przez `vcn-ui --setup`.

### Uruchomienie

```bash
node src/index.js
```

### systemd

```bash
cp services/vcn-ui.service /etc/systemd/system/
systemctl enable vcn-ui
systemctl start vcn-ui
```

---

## Reverse proxy (nginx)

Przykładowa konfiguracja nginx z HTTPS:

```nginx
server {
    listen 443 ssl;
    server_name twoja-domena.pl;

    ssl_certificate /etc/letsencrypt/live/twoja-domena.pl/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/twoja-domena.pl/privkey.pem;

    # Serwer MCP
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 120s;
    }

    # Panel UI (opcjonalnie pod inną subdomeną)
    # server_name admin.twoja-domena.pl;
    # location / { proxy_pass http://127.0.0.1:3000; }
}

server {
    listen 80;
    server_name twoja-domena.pl;
    return 301 https://$host$request_uri;
}
```

---

## Aktualizacja

```bash
# skopiuj nowe binarki
cp dist/vcn-mcp /usr/local/bin/vcn-mcp
cp dist/vcn-ui  /usr/local/bin/vcn-ui
systemctl restart vcn-mcp vcn-ui
```

Serwer wyśle powiadomienie Telegram o zatrzymaniu (stop) i uruchomieniu (start).

---

## Rozwiązywanie problemów

### Serwer nie startuje

1. Sprawdź logi: `journalctl -u vcn-mcp -n 50` lub `tail /var/log/vcn-mcp/mcp_server.log`
2. Jeśli brak config: `vcn-mcp --setup`
3. Upewnij się, że katalogi z `paths` istnieją (`/srv/vnet-mcp/data/`, `vnet.db`)

### Brak powiadomień Telegram

1. Zweryfikuj `TELEGRAM_BOT_TOKEN` i `TELEGRAM_CHAT_ID`
2. Sprawdź, czy bot jest dodany do chatu
3. Wyślij ręcznie test: `curl "https://api.telegram.org/bot<TOKEN>/getMe"`

### 401 przy wywołaniu MCP

1. Sprawdź, czy token nie wygasł (90 dni)
2. Zweryfikuj `SECRET_KEY` — zmiana klucza unieważnia wszystkie tokeny
3. Sprawdź logi serwera — pojawiają się tam przyczyny odmowy
