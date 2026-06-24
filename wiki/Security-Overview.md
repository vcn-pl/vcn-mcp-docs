# Bezpieczeństwo — przegląd

> Jak chroniony jest system VCN MCP.

---

## Dla użytkownika — najważniejsze zasady

### Token = klucz do systemu

Token wygenerowany w `/setup` daje dostęp do serwera w Twoim imieniu. Traktuj go jak hasło:

- **Nie udostępniaj** tokenu innym osobom
- **Nie wklejaj** w publicznych miejscach (GitHub, dokumenty współdzielone)
- **Odśwież** co 90 dni (lub wcześniej jeśli podejrzewasz wyciek)

Jeśli token zostanie skradziony — administrator może go natychmiast unieważnić.

### Co widzi administrator?

Administrator widzi w logach:
- kiedy i jakie narzędzia były wywołane
- kto je wywołał (nazwa użytkownika)
- czy wywołanie zakończyło się sukcesem czy błędem

**Administrator NIE widzi** treści Twoich rozmów z Claude (to zostaje po stronie Claude/Anthropic).

---

## Model autoryzacji

### Jak wygląda weryfikacja przy każdym żądaniu?

```
Żądanie do /mcp
      │
      ▼
Czy jest token? → NIE → 401 (odmowa dostępu)
      │
      ▼ TAK
Czy token jest ważny JWT? → NIE → 401
      │
      ▼ TAK
Czy token jest w bazie (nie odwołany)? → NIE → 401
      │
      ▼ TAK
Czy token nie wygasł (90 dni)? → NIE → 401
      │
      ▼ TAK
Dostęp przyznany — filtrowanie po grupach (tenantach)
```

### Grupy dostępu (multi-tenancy)

Każdy użytkownik należy do jednej lub kilku grup (tenantów). Grupy określają, do jakich zasobów masz dostęp:

- **Baza wiedzy** — każda grupa ma swoją sekcję
- **Szablony umów** — każda grupa ma swoje szablony
- **Projekty i klienci** — każda grupa widzi tylko swoje

Przykład: użytkownik w grupie `vnet` nie widzi zasobów grupy `tools` i odwrotnie.

---

## Szyfrowanie

### HTTPS

Serwer działa za reverse proxy (nginx/Caddy) z certyfikatem TLS/SSL. Całość komunikacji jest zaszyfrowana — tokeny i dane przesyłane są bezpiecznie.

Serwer nie powinien być dostępny przez HTTP (nieszyfrowany).

### Hasła

Hasła przechowywane są w bazie jako hash (nieodwracalny skrót). Nawet jeśli ktoś uzyska dostęp do bazy danych — haseł nie da się odczytać.

> **Uwaga techniczna dla administratorów:** Używany algorytm to SHA-256. Zalecana migracja do bcrypt dla lepszej odporności na brute-force.

---

## Prywatność danych

### Gdzie trafiają dane?

| Dane | Gdzie lądują |
|---|---|
| Treść ofert | Tymczasowy katalog na serwerze (`/tmp/offers/`) |
| Wygenerowane umowy | `/generated/` na serwerze |
| Dane firm (NIP lookup) | Zapytania do publicznych API (MF, KRS) — dane firmy są publiczne |
| Udostępnione strony | Baza SQLite na serwerze |
| Tokeny | Baza SQLite na serwerze |

**Dane NIE trafiają do:**
- Anthropic (twórców Claude)
- Zewnętrznych serwisów AI
- Innych firm

Wyjątek: `lookup_company` odpytuje publiczne rejestry (MF, KRS, DuckDuckGo) — wysyłane są tylko dane identyfikacyjne (NIP, nazwa firmy).

### Publiczne strony i rozmowy

Gdy korzystasz z `publish_page` lub `share_conversation` — generowany jest **publiczny link**. Każdy kto ma ten link może zobaczyć zawartość. Usuń stronę/rozmowę jeśli nie powinna być dostępna.

---

## Co zrobić gdy token wyciekł?

1. Powiadom administratora systemu
2. Administrator usunie token z bazy (natychmiastowa blokada)
3. Wygeneruj nowy token przez `/setup`

---

## Dla administratorów — krótka checklista

- [ ] Serwer działa za HTTPS (nie HTTP)
- [ ] Sekrety (JWT_SECRET_KEY, tokeny Telegram) są w zmiennych środowiskowych, nie w kodzie
- [ ] Port 8000 nie jest dostępny z zewnątrz (tylko przez proxy)
- [ ] Regularne backupy bazy SQLite
- [ ] Monitorowanie daily digest (liczba błędów, nieznane adresy IP)
- [ ] Hasła użytkowników są silne (enforce policy przy rejestracji lub administracyjnie)

Pełna dokumentacja bezpieczeństwa techniczna → [docs/security.md](../security.md)

---

## Zgłaszanie problemów bezpieczeństwa

Jeśli zauważysz podejrzaną aktywność lub lukę bezpieczeństwa:

1. W Claude: „Zgłoś problem: [opis]" — wyśle powiadomienie do administratora
2. Bezpośrednio do administratora przez e-mail/telefon

Nie publikuj informacji o lukach publicznie przed naprawą.
