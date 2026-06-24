# VCN MCP Server — Dokumentacja

> Serwer MCP (Model Context Protocol) łączący asystentów AI (Claude, ChatGPT) z zasobami firmy — bazą wiedzy, rejestrami firm i generatorem umów.

## Czym jest VCN MCP?

VCN MCP to most między asystentem AI a systemami firmy. Dzięki niemu Claude lub ChatGPT może:

- odpowiadać na pytania oparte na firmowej bazie wiedzy
- wyszukiwać dane firm po NIP lub nazwie
- automatycznie generować kompletne umowy na podstawie ofert (PDF lub tekst)
- publikować strony HTML dostępne pod stałym linkiem
- udostępniać treść rozmów jako strony WWW

Serwer działa jako usługa HTTP — asystent AI łączy się z nim przez standardowy protokół MCP.

---

## Dla kogo?

| Rola | Co zyska |
|---|---|
| **Handlowiec** | Generowanie umów z oferty w kilka sekund, lookup firmy po NIP |
| **Administrator** | Multi-tenant, OAuth 2.0, pełne logi, Telegram alerts |
| **Developer** | REST API, testy jednostkowe, modułowy Python |
| **Użytkownik Claude** | Dostęp do wiedzy firmy bez kopiowania dokumentów |

---

## Struktura dokumentacji

| Dokument | Opis |
|---|---|
| [Architektura](architecture.md) | Jak działa system — komponenty, przepływ danych |
| [Instalacja](installation.md) | Jak zainstalować i uruchomić serwer |
| [Konfiguracja](configuration.md) | Wszystkie opcje config.json i zmiennych środowiskowych |
| [API Reference](api-reference.md) | Endpointy REST + wszystkie narzędzia MCP |
| [Baza danych](database.md) | Schemat SQLite, tabele, relacje |
| [Bezpieczeństwo](security.md) | Model auth, zagrożenia, rekomendacje |
| [Przewodnik użytkownika](user-guide.md) | Jak korzystać z systemu (bez wiedzy technicznej) |
| [Przewodnik developera](development.md) | Testy, struktura kodu, jak wprowadzać zmiany |

### Wiki (GitHub)

Folder [wiki/](wiki/) zawiera treść gotową do wgrania na GitHub Wiki:

- [Home](wiki/Home.md) — strona startowa wiki
- [Jak to działa](wiki/How-It-Works.md) — opis dla osób nietechnicznych
- [Szybki start](wiki/Getting-Started.md) — pierwsze kroki
- [Narzędzia MCP](wiki/MCP-Tools-Reference.md) — kompletna lista narzędzi
- [Generowanie umów](wiki/Contract-Generation.md) — pipeline oferta→umowa
- [Bezpieczeństwo](wiki/Security-Overview.md) — model bezpieczeństwa
- [FAQ](wiki/FAQ.md) — najczęstsze pytania

---

## Szybki przegląd stosu technologicznego

```
Python 3.12 + Starlette + MCP SDK
SQLite (multi-tenant)
OAuth 2.0 + JWT (90 dni)
Node.js + Express (panel admina)
Telegram (powiadomienia)
```

## Wersja

Bieżąca wersja serwera zdefiniowana w `mcp/config.json` → `server.version`.
