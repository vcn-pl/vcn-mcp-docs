---
title: Baza danych
nav_order: 7
---

# Baza danych

## Silnik

SQLite — plik `{vcn_root}/vnet.db`. Jeden plik dla całego systemu.  
Schemat tworzony automatycznie przy pierwszym starcie serwera.

---

## Tabele

### `users`

Konta użytkowników.

| Kolumna | Typ | Opis |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `username` | TEXT UNIQUE | Nazwa użytkownika |
| `password_hash` | TEXT | SHA-256 hasła |
| `email` | TEXT UNIQUE | Email (opcjonalny) |
| `teams` | TEXT | JSON array teamów, np. `["vnet", "tools"]` |
| `created_at` | DATETIME | Data rejestracji |

### `oauth_tokens`

Aktywne tokeny JWT.

| Kolumna | Typ | Opis |
|---|---|---|
| `token` | TEXT PK | JWT string |
| `username` | TEXT | FK → users.username |
| `issued_at` | REAL | Unix timestamp wydania |
| `expires_at` | REAL | Unix timestamp wygasania |

### `oauth_clients`

Zarejestrowane klienty OAuth (np. Claude.ai).

| Kolumna | Typ | Opis |
|---|---|---|
| `client_id` | TEXT PK | Identyfikator klienta |
| `client_secret` | TEXT | Sekret klienta |
| `name` | TEXT | Czytelna nazwa |
| `redirect_uris` | TEXT | JSON array URI przekierowań |
| `created_at` | REAL | Unix timestamp |

### `shared_conversations`

Udostępnione rozmowy.

| Kolumna | Typ | Opis |
|---|---|---|
| `slug` | TEXT PK | URL-friendly identyfikator (np. `abc123`) |
| `title` | TEXT | Tytuł rozmowy |
| `messages` | TEXT | JSON array `[{role, content}, ...]` |
| `created_by` | TEXT | Nazwa użytkownika |
| `created_at` | REAL | Unix timestamp |

### `preview_pages`

Opublikowane strony HTML.

| Kolumna | Typ | Opis |
|---|---|---|
| `slug` | TEXT PK | URL-friendly identyfikator |
| `title` | TEXT | Tytuł strony |
| `html` | TEXT | Pełny kod HTML |
| `created_by` | TEXT | Nazwa użytkownika |
| `created_at` | REAL | Unix timestamp |
| `updated_at` | REAL | Unix timestamp ostatniej aktualizacji |

### `login_log`

Próby logowania — używane przez rate limiting i detekcję brute-force.

| Kolumna | Typ | Opis |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `ts` | REAL | Unix timestamp |
| `ip` | TEXT | Adres IP klienta |
| `username` | TEXT | Podana nazwa użytkownika |
| `success` | INTEGER | 1 = sukces, 0 = nieudane |

### `tool_log`

Log wywołań narzędzi MCP.

| Kolumna | Typ | Opis |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `ts` | REAL | Unix timestamp |
| `tool` | TEXT | Nazwa narzędzia (np. `get_context`) |
| `username` | TEXT | Kto wywołał |
| `ok` | INTEGER | 1 = sukces, 0 = błąd |

Tabela używana przez daily digest (statystyki dzienne).

### `tenants`

Lista tenantów (wypełniana z config.json przy starcie).

| Kolumna | Typ | Opis |
|---|---|---|
| `id` | INTEGER PK | |
| `name` | TEXT UNIQUE | Identyfikator tenanta |

### `projects`

Projekty.

| Kolumna | Typ | Opis |
|---|---|---|
| `id` | INTEGER PK | |
| `name` | TEXT | Nazwa projektu |
| `client_id` | INTEGER | FK → clients.id |
| `tenant` | TEXT | FK → tenants.name |
| `description` | TEXT | Opis (opcjonalny) |
| `created_at` | DATETIME | |

### `clients`

Klienci.

| Kolumna | Typ | Opis |
|---|---|---|
| `id` | INTEGER PK | |
| `name` | TEXT | Nazwa firmy |
| `nip` | TEXT | NIP (opcjonalny) |
| `tenant` | TEXT | FK → tenants.name |
| `created_at` | DATETIME | |

---

## Diagram relacji

```
users (1) ──────────── (N) oauth_tokens
  │username                  username

users (1) ──────────── (N) shared_conversations
  │username                  created_by

users (1) ──────────── (N) preview_pages
  │username                  created_by

users (1) ──────────── (N) tool_log
  │username                  username

tenants (1) ────────── (N) projects
  │name                      tenant

tenants (1) ────────── (N) clients
  │name                      tenant

clients (1) ────────── (N) projects
  │id                        client_id
```

---

## Zarządzanie bazą

### Backup

```bash
sqlite3 /srv/vnet-mcp/data/vnet.db ".backup /srv/backups/vnet-$(date +%Y%m%d).db"
```

### Podgląd statystyk

```bash
sqlite3 /srv/vnet-mcp/data/vnet.db "
SELECT
  (SELECT COUNT(*) FROM users) AS users,
  (SELECT COUNT(*) FROM oauth_tokens WHERE expires_at > unixepoch()) AS active_tokens,
  (SELECT COUNT(*) FROM tool_log) AS total_calls,
  (SELECT COUNT(*) FROM preview_pages) AS pages,
  (SELECT COUNT(*) FROM shared_conversations) AS conversations;
"
```

### Rewokacja tokenu użytkownika

```bash
sqlite3 /srv/vnet-mcp/data/vnet.db \
  "DELETE FROM oauth_tokens WHERE username = 'username';"
```

### Wyświetlenie aktywności użytkownika

```bash
sqlite3 /srv/vnet-mcp/data/vnet.db "
SELECT datetime(ts, 'unixepoch', 'localtime') AS time, tool, ok
FROM tool_log
WHERE username = 'username'
ORDER BY ts DESC
LIMIT 20;
"
```

---

## Uwagi

- SQLite nie jest odpowiedni dla jednoczesnego dostępu wielu procesów. Serwer działa jednoprocesowo, więc to nie jest problem.
- Brak indeksów na `tool_log(ts, username)` — przy dużym wolumenie logów daily digest może być wolny.
- `teams` w tabeli `users` to surowy JSON string — nie ma walidacji na poziomie DB.
- Brak foreign key constraints (SQLite FK domyślnie wyłączone) — integralność danych zapewniana przez kod.
