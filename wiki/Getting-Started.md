# Szybki start

## Wymagania

- Dostęp do internetu
- Konto w systemie VCN MCP (tworzone przez administratora)
- Asystent AI obsługujący protokół MCP (Claude, ChatGPT z wtyczkami MCP)

---

## Krok 1: Uzyskaj konto

Skontaktuj się z administratorem systemu — to on tworzy konta użytkowników.

---

## Krok 2: Podłącz do Claude

### Claude.ai (strona internetowa)

1. Otwórz Claude.ai
2. Przejdź do ustawień → sekcja „Integrations" lub „MCP"
3. Kliknij „Add MCP server"
4. Wpisz URL serwera: `https://twoja-domena.pl/mcp`
5. Kliknij „Connect" — pojawi się formularz logowania
6. Zaloguj się swoim kontem VCN MCP

Claude automatycznie uzyska dostęp i zgłosi, że widzi nowe narzędzia z VCN.

### Claude Code (CLI)

Dodaj do konfiguracji MCP w Claude Code:
```json
{
  "mcpServers": {
    "vcn": {
      "type": "http",
      "url": "https://twoja-domena.pl/mcp"
    }
  }
}
```

Po uruchomieniu Claude Code pojawi się prośba o autoryzację — zaloguj się swoim kontem.

---

## Krok 3: Przetestuj połączenie

W Claude napisz:
> „Sprawdź jakie narzędzia masz dostępne z VCN MCP"

lub

> „Wylistuj dostępnych tenantów"

Jeśli Claude odpowie z listą narzędzi lub tenantów — połączenie działa.

---

## Pierwsze użycie

### Pytanie do bazy wiedzy

> „Czym jest produkt X? Odpowiedz na podstawie bazy wiedzy firmy."

### Sprawdzenie firmy

> „Sprawdź dane firmy o NIP 5213657029"

### Generowanie umowy

> „Wygeneruj umowę SaaS dla firmy ABC Sp. z o.o., NIP 1234567890, kwota miesięczna 500 zł netto."

---

## Masz problem z połączeniem?

Sprawdź:
1. Czy adres URL jest poprawny (bez `/` na końcu)
2. Czy login i hasło są prawidłowe
3. Czy asystent AI obsługuje MCP

Jeśli problem trwa → zgłoś go administratorowi lub napisz w Claude: „Zgłoś problem"

---

**Następny krok:** [How-It-Works](How-It-Works.md) — dowiedz się jak to działa
