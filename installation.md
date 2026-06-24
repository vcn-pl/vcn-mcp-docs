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

### 1. Klonowanie repozytorium

```bash
git clone <url> /opt/vnet-mcp
cd /opt/vnet-mcp
```

### 2. Środowisko Python

```bash
python3.12 -m venv /opt/vnet-mcp/venv
source /opt/venv/bin/activate
pip install -r mcp/requirements.txt
```

### 3. Konfiguracja

Skopiuj przykładowy config i dostosuj:

```bash
cp services/config.example.json mcp/config.json
```

Edytuj `mcp/config.json`:

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
    "version": "26.07.02",
    "host": "0.0.0.0",
    "port": 8000
  },
  "auth": {
    "token_expire_days": 90
  },
  "tenants": ["vnet", "tools"]
}
```

Szczegóły wszystkich opcji → [configuration.md](configuration.md).

### 4. Zmienne środowiskowe

Skopiuj przykładowy plik i uzupełnij:

```bash
cp services/vcn-mcp.env.example /etc/vcn-mcp.env
chmod 600 /etc/vcn-mcp.env
```

Wymagane zmienne:
```bash
TELEGRAM_BOT_TOKEN=your_token
TELEGRAM_CHAT_ID=your_personal_chat_id
TELEGRAM_LOG_CHAT_ID=your_log_chat_id
JWT_SECRET_KEY=your_random_secret_256bit
```

Generowanie `JWT_SECRET_KEY`:
```bash
python3 -c "import secrets; print(secrets.token_hex(32))"
```

### 5. Struktura danych

Utwórz katalogi:
```bash
mkdir -p /srv/vnet-mcp/data/base
mkdir -p /srv/vnet-mcp/data/knowledge
mkdir -p /srv/vnet-mcp/data/templates
mkdir -p /srv/vnet-mcp/data/generated
mkdir -p /srv/vnet-mcp/data/files
mkdir -p /srv/vnet-mcp/data/tmp/offers
mkdir -p /var/log/vcn-mcp
```

### 6. Weryfikacja konfiguracji

```bash
python3 scripts/config_check.py
```

### 7. Pierwszy start

```bash
cd mcp
python3 main.py
```

Serwer powinien:
- wypisać "✅ All checks passed"
- uruchomić się na `http://0.0.0.0:8000`
- wysłać powiadomienie Telegram o starcie

Sprawdź: `curl http://localhost:8000/health`

### 8. Pierwszy użytkownik (admin)

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

### Instalacja

```bash
cd ui
npm install
```

### Konfiguracja

UI czyta tę samą `mcp/config.json` co serwer Python.  
Dodatkowe ustawienia w `services/vcn-ui.env.example`.

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
cd /opt/vnet-mcp
git pull
pip install -r mcp/requirements.txt   # jeśli zmieniły się zależności
systemctl restart vcn-mcp
```

Serwer wyśle powiadomienie Telegram o zatrzymaniu (stop) i uruchomieniu (start).

---

## Rozwiązywanie problemów

### Serwer nie startuje

1. Sprawdź logi: `journalctl -u vcn-mcp -n 50`
2. Sprawdź config: `python3 scripts/config_check.py`
3. Upewnij się, że katalogi z `paths` istnieją i mają właściwe uprawnienia

### Brak powiadomień Telegram

1. Zweryfikuj `TELEGRAM_BOT_TOKEN` i `TELEGRAM_CHAT_ID`
2. Sprawdź, czy bot jest dodany do chatu
3. Wyślij ręcznie test: `curl "https://api.telegram.org/bot<TOKEN>/getMe"`

### 401 przy wywołaniu MCP

1. Sprawdź, czy token nie wygasł (90 dni)
2. Zweryfikuj `JWT_SECRET_KEY` — zmiana klucza unieważnia wszystkie tokeny
3. Sprawdź logi serwera — pojawiają się tam przyczyny odmowy
