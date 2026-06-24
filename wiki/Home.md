# VCN MCP Server — Wiki

Witaj w dokumentacji VCN MCP Server.

## Co to jest VCN MCP?

VCN MCP to serwer łączący asystentów AI (Claude, ChatGPT) z zasobami firmy. Dzięki niemu asystent AI może:

- odpowiadać na pytania z firmowej bazy wiedzy
- sprawdzać dane firm w publicznych rejestrach (NIP, KRS)
- automatycznie generować umowy na podstawie ofert
- publikować strony HTML pod stałym adresem

---

## Dla kogo jest ta dokumentacja?

| Jestem... | Zacznij od... |
|---|---|
| Nowym użytkownikiem | [Getting-Started](Getting-Started.md) |
| Ciekaw jak to działa | [How-It-Works](How-It-Works.md) |
| Chcę podłączyć Claude | [Getting-Started#podłączenie-do-claude](Getting-Started.md#podłączenie-do-claude) |
| Szukam opisu narzędzi | [MCP-Tools-Reference](MCP-Tools-Reference.md) |
| Chcę wygenerować umowę | [Contract-Generation](Contract-Generation.md) |
| Administrator / bezpieczeństwo | [Security-Overview](Security-Overview.md) |
| Mam pytanie | [FAQ](FAQ.md) |

---

## Szybki start

1. Uzyskaj konto od administratora systemu
2. W Claude → ustawienia → MCP → dodaj `https://twoja-domena.pl/mcp`
3. Zaloguj się w formularzu który się pojawi
4. Gotowe — Claude ma dostęp do firmowej wiedzy

---

## Strony tej wiki

- [Getting-Started](Getting-Started.md) — pierwsze kroki, podłączenie Claude
- [How-It-Works](How-It-Works.md) — jak działa serwer (bez żargonu technicznego)
- [MCP-Tools-Reference](MCP-Tools-Reference.md) — lista wszystkich dostępnych narzędzi
- [Contract-Generation](Contract-Generation.md) — generowanie umów z ofert
- [Security-Overview](Security-Overview.md) — jak chroniony jest system
- [FAQ](FAQ.md) — najczęstsze pytania

---

## Linki

- Repozytorium: GitHub → `vnet-mcp`
- Szczegółowa dokumentacja techniczna: [docs/](../)
- Zgłoszenie problemu: powiedz Claude „zgłoś problem z opisem: ..."
