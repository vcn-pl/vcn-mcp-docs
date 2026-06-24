# Jak to działa?

> Wyjaśnienie dla osób bez wiedzy technicznej.

---

## Analogia

Wyobraź sobie sekretarkę znającą wszystkie firmowe dokumenty, zdolną sprawdzić dane dowolnej firmy w rejestrach i przygotować gotową umowę w 30 sekund. VCN MCP to właśnie taka sekretarka — tyle że cyfrowa i dostępna przez asystenta AI.

---

## Trzy warstwy systemu

```
Ty (użytkownik)
      │
      ▼
Asystent AI (Claude, ChatGPT)
      │  "sprawdź NIP tej firmy"
      ▼
VCN MCP Server  ←→  Baza wiedzy firmy
      │         ←→  Rejestry MF / KRS
      │         ←→  Szablony umów
      ▼
Wynik wraca do Claude → do Ciebie
```

Ty rozmawiasz z Claude w normalnym języku. Claude wie, kiedy warto zapytać serwer VCN o dodatkowe informacje, robi to automatycznie i włącza wyniki do swojej odpowiedzi.

---

## Czym jest protokół MCP?

MCP (Model Context Protocol) to standard, który pozwala asystentom AI korzystać z zewnętrznych narzędzi i danych. Działa podobnie jak wtyczki do przeglądarki — rozszerza możliwości asystenta o dodatkowe funkcje.

Dzięki MCP Claude nie tylko „zna" informacje (z treningu), ale może też **aktywnie pobierać aktualne dane** z systemów firmy.

---

## Jak Claude wie, co zrobić?

Każde narzędzie w serwerze ma opis. Kiedy piszesz do Claude:

> „Sprawdź firmę o NIP 5213657029"

Claude rozpoznaje, że to zadanie pasuje do narzędzia `lookup_company`, wywołuje je z odpowiednimi parametrami i wkleja wynik do swojej odpowiedzi.

Nie musisz znać nazw narzędzi — Claude dobiera je sam na podstawie kontekstu.

---

## Jak wygląda generowanie umów?

To najbardziej zaawansowana funkcja. Oto co się dzieje pod spodem:

```
1. Wklejasz tekst oferty lub wgrywasz PDF
          │
2. offer_parser — odczytuje kluczowe dane:
   kwoty, daty, nazwy usług, dane klienta
          │
3. company_lookup — sprawdza NIP klienta
   w rejestrze MF/KRS, pobiera adres, REGON
          │
4. template_registry — dobiera szablony umów
   pasujące do typu oferty
          │
5. fill_contract — wypełnia szablony Word (.docx)
   wstawionymi danymi
          │
6. Dostajesz linki do gotowych plików DOCX
```

Cały ten łańcuch dzieje się automatycznie w kilkanaście sekund.

---

## Gdzie przechowywane są dane?

| Dane | Gdzie |
|---|---|
| Konta użytkowników | Baza SQLite na serwerze |
| Baza wiedzy firmy | Pliki JSON/Markdown na serwerze |
| Szablony umów | Pliki DOCX na serwerze |
| Wygenerowane umowy | Tymczasowo na serwerze (do pobrania) |
| Udostępnione strony | Baza SQLite |
| Tokeny autoryzacyjne | Baza SQLite |

Żadne dane nie trafiają do Anthropic (twórców Claude) ani innych zewnętrznych serwisów — poza publicznymi API (MF, KRS) używanymi do sprawdzania danych firm.

---

## Bezpieczeństwo w skrócie

- Każde połączenie wymaga **tokenu** — bez niego serwer odmawia dostępu
- Token powiązany jest z Twoim kontem — widać kto co robi
- Każdy użytkownik widzi tylko zasoby przypisane do jego grupy (tenantu)
- Komunikacja odbywa się przez HTTPS — zaszyfrowana

Więcej szczegółów: [Security-Overview](Security-Overview.md)

---

## Czego serwer nie robi?

- Nie przechowuje treści Twoich rozmów z Claude (to zostaje w Claude)
- Nie wysyła danych do zewnętrznych AI
- Nie odpowiada na pytania sam — służy jako źródło danych dla Claude

---

**Następny krok:** [MCP-Tools-Reference](MCP-Tools-Reference.md) — co dokładnie możesz zrobić?
