# Bezpieczeństwo

## Model bezpieczeństwa — przegląd

VCN MCP używa warstwy OAuth 2.0 + JWT do autoryzacji. Każde żądanie do endpointu `/mcp` wymaga ważnego tokenu. System jest zaprojektowany do działania za reverse proxy (nginx/Caddy) z HTTPS.

---

## Uwierzytelnienie

### JWT (JSON Web Token)

- Tokeny ważne **90 dni** (konfigurowalnie przez `auth.token_expire_days`)
- Przechowywane w bazie (`oauth_tokens`) — możliwa revokacja
- Przekazywane nagłówkiem `Authorization: Bearer <token>` lub parametrem URL `?token=`
- Zawierają: `username`, `teams`, `exp` (expiry timestamp)

### OAuth 2.0

Serwer implementuje pełny flow OAuth 2.0 Authorization Code:

```
/oauth/authorize        → generuje authorization_code
/oauth/login            → formularz logowania (GET/POST)
/oauth/token            → wymiana code → access_token
/oauth/clients/register → rejestracja klientów OAuth
```

Obsługiwane przez klientów AI (Claude.ai używa tego flow do uzyskania tokenu).

### Hasła użytkowników

Hasła hashowane algorytmem **Argon2id** (implementacja w `users.py`). Stare hashe SHA-256 są automatycznie migrowane przy kolejnym logowaniu.

---

## Autoryzacja i Multi-tenancy

### Przepływ autoryzacji

```
JWT token → verify_token() → user dict
         → user['teams'] → accessible_tenants
         → filtrowanie zasobów (knowledge, templates)
```

Każdy użytkownik widzi tylko zasoby przypisane do jego teamów. Teamy przechowywane są jako JSON array w kolumnie `teams` tabeli `users`.

### Separator tenantów

Zasoby (wiedza, szablony) są izolowane przez strukturę katalogów:
```
/knowledge/<tenant>/
/templates/<tenant>/
```

Nie ma izolacji na poziomie bazy danych (jedna wspólna SQLite) — wszystkie tabele współdzielone.

---

## Ochrona przed atakami

### Path Traversal

Endpointy serwowania plików (`/f/{filename}`, `/download/{filename}`) stosują walidację:

```python
path = (VCN_ROOT / "files" / safe).resolve()
if not str(path).startswith(str((VCN_ROOT / "files").resolve())):
    return Response("Forbidden", status_code=403)
```

Sanityzacja nazwy pliku — dozwolone tylko alfanumeryczne, `.`, `_`, `-`.

### Upload plików

- Tokeny upload są jednorazowe i wygasają czasowo (`_upload_slots`, `_file_slots`)
- Nazwy plików sanityzowane przed zapisem na dysk
- Pliki lądują w `/tmp/offers/` lub `/files/` — nie w ścieżkach wykonywalnych

### MCP endpoint

- Brak tokenu → 401 z `WWW-Authenticate` header (discovery dla Claude)
- Zły token → 401 z `error="invalid_token"`
- Błąd parsowania tokenu logowany na poziomie WARNING

---

## Powierzchnia ataku

### Endpointy publiczne (bez autoryzacji)

| Endpoint | Ryzyko |
|---|---|
| `/oauth/login` | Brute-force haseł (rate limiting: 5 prób/60s per IP, alert Telegram) |
| `/c/{slug}` | Udostępnione rozmowy — treść widoczna bez auth |
| `/p/{slug}` | Opublikowane strony — treść widoczna bez auth |
| `/f/{filename}` | Pliki statyczne — dostępne bez auth |
| `/health` | Info o serwerze (ujawnia nazwę) |
| `/upload/{token}` | Przyjmuje pliki — tylko z ważnym tokenem |

### Potencjalne zagrożenia

| Zagrożenie | Obecna ochrona | Brak ochrony |
|---|---|---|
| Brute-force haseł | Argon2id + rate limiting + alerty Telegram | — |
| Token leak (URL) | `_TokenRedactor` filtr w loggerze maskuje `?token=` → `[REDACTED]` | — |
| SQLite injection | Parametryzowane zapytania | — |
| XSS w stronach | HTML escape w rozmowach | Strony `/p/{slug}` renderują surowe HTML (by design) |
| SSRF przez company_lookup | `_is_ssrf_url()` blokuje scraping prywatnych/lokalnych adresów IP | — |
| Ekspozycja logów | Logi na dysk | Mogą zawierać wrażliwe dane |

---

## Konfiguracja produkcyjna — checklist

### Wymagane

- [ ] Serwer za HTTPS reverse proxy (nginx, Caddy) — tokeny JWT w plain HTTP = kradzież sesji
- [ ] Firewall: port 8000 (MCP) dostępny tylko lokalnie lub dla proxy; port 8001 (UI) według potrzeb
- [ ] Sekrety w `/etc/vcn-mcp.env` i `/etc/vcn-ui.env` z uprawnieniami `600`

### Zalecane

- [ ] Rotacja `SECRET_KEY` co kilka miesięcy — unieważnia wszystkie aktywne tokeny MCP
- [ ] Backup SQLite (cron): `cp /srv/vnet-mcp/vnet.db /backup/vnet-$(date +%F).db`
- [ ] Skróć `auth.token_expire_days` w `config.json` jeśli serwer jest publiczny (domyślnie: 90)

### Opcjonalne

- [ ] 2FA dla konta admin w panelu UI — `/account/2fa`

---

## Zarządzanie tokenami

### Wydawanie

Tokeny generowane przez `/oauth/token` po udanej autoryzacji OAuth lub przez `/setup` (samodzielne generowanie po zalogowaniu).

### Rewokacja

Tokeny przechowywane w `oauth_tokens` — usunięcie rekordu = natychmiastowa rewokacja.

Każde żądanie do `/mcp` weryfikuje obecność tokenu w bazie (`is_token_active()`), więc usunięcie rekordu skutkuje natychmiastowym odrzuceniem tokenu bez restartu serwera.

Automatyczna rewokacja przy:
- zmianie hasła użytkownika MCP przez panel UI — `DELETE FROM oauth_tokens WHERE username=?`
- usunięciu użytkownika MCP przez panel UI — jak wyżej

### Czyszczenie wygasłych tokenów

Codzienne zadanie (`daily token cleanup`) automatycznie usuwa wygasłe rekordy z `oauth_tokens`.

### Wygasanie

`verify_token()` sprawdza pole `exp` z JWT. Wygasłe tokeny zwracają 401.

---

## Logi bezpieczeństwa

Każde wywołanie narzędzia MCP trafia do tabeli `tool_log`:
```sql
(ts, tool, username, ok)
```

Logowane jest też w pliku tekstowym:
- każde żądanie do `/mcp` (user, IP)
- odmowy autoryzacji (IP)
- uploady plików (token, rozmiar, ścieżka)

Próby logowania zapisywane w tabeli `login_log`. Wykrycie brute-force (przekroczenie rate limit) wysyła alert Telegram.

Daily digest o 23:59 raportuje: łączną liczbę wywołań, błędy, unikalnych użytkowników, top 5 narzędzi.
