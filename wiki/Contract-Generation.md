# Generowanie umów

> Kompletny przewodnik po automatycznym generowaniu umów z ofert.

---

## Jak to działa?

System analizuje ofertę (tekst lub plik PDF), wyciąga kluczowe informacje, sprawdza dane firmy klienta w rejestrach i wypełnia odpowiednie szablony umów. Cały proces trwa kilkanaście sekund.

### Przepływ krok po kroku

```
Oferta (tekst / PDF)
        │
        ▼
┌─────────────────────┐
│   Ekstrakcja danych │  ← offer_parser
│                     │
│  • nazwa klienta    │
│  • NIP              │
│  • kwoty netto/brutto│
│  • daty             │
│  • lista usług/sprzętu│
│  • typ umowy        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Lookup firmy        │  ← company_lookup
│                     │
│  MF Whitelist →     │
│  KRS API →          │
│  DuckDuckGo         │
│                     │
│  • pełna nazwa      │
│  • adres siedziby   │
│  • REGON            │
│  • status VAT       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Dobór szablonów    │  ← template_registry
│                     │
│  Słowa kluczowe →   │
│  Typ umowy →        │
│  Lista szablonów    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Wypełnienie DOCX   │  ← fill_contract × N
│                     │
│  Szablon + zmienne  │
│  → Gotowy plik DOCX │
└──────────┬──────────┘
           │
           ▼
  Linki do pobrania
```

---

## Jak podać ofertę?

### Opcja 1: Wklej tekst

Napisz do Claude:
> „Wygeneruj umowy na podstawie tej oferty:
> 
> Klient: ABC Sp. z o.o., NIP 1234567890
> Przedmiot: wdrożenie systemu monitoringu
> Kwota: 15 000 zł netto
> Czas realizacji: 30 dni
> Abonament miesięczny: 500 zł netto"

### Opcja 2: Wgraj PDF

Napisz do Claude:
> „Wygeneruj umowy z pliku PDF"

Claude wyśle Ci link — otwierasz go w przeglądarce i przeciągasz plik PDF.  
Po wgraniu Claude automatycznie pobierze plik i przetworzy.

---

## Co musi zawierać oferta?

Im więcej danych, tym lepsze wyniki. Serwer szuka:

| Dane | Przykład |
|---|---|
| Nazwa klienta | „ABC Sp. z o.o." |
| NIP klienta | „NIP: 1234567890" |
| Wartość umowy | „15 000 zł netto" |
| Data realizacji | „30 dni od podpisania" |
| Typ usług | „wdrożenie", „abonament", „SaaS" |
| Lista pozycji | sprzęt, licencje, usługi |

Jeśli NIP jest podany — dane firmy zostaną automatycznie uzupełnione z rejestru MF/KRS.

---

## Typy umów

System automatycznie dobiera typ umowy na podstawie słów kluczowych w ofercie:

| Typ umowy | Kiedy dobierany | Szablony |
|---|---|---|
| **Wdrożenie + abonament** | „wdrożenie", „montaż", „instalacja", „kamera", „hardware" | Umowa główna + 3 załączniki |
| **SaaS / Cloud** | „saas", „cloud", „abonament", „keepsober" | Umowa SaaS + załącznik |
| **SLA** | „sla", „serwis", „utrzymanie" | Umowa serwisowa |

Możesz też wymusić typ:
> „Wygeneruj umowę SaaS dla tego klienta"

---

## Zmienne w szablonach

Szablony DOCX używają placeholderów `{{nazwa_zmiennej}}`. Po generowaniu są zastępowane rzeczywistymi danymi.

Przykładowe zmienne:

| Zmienna | Skąd pochodzi |
|---|---|
| `{{firma_nazwa}}` | company_lookup |
| `{{firma_nip}}` | oferta / company_lookup |
| `{{firma_adres}}` | company_lookup |
| `{{firma_regon}}` | company_lookup |
| `{{kwota_netto}}` | offer_parser |
| `{{kwota_brutto}}` | offer_parser (obliczone) |
| `{{data_zawarcia}}` | offer_parser / domyślnie dziś |
| `{{czas_realizacji}}` | offer_parser |

---

## Pobieranie wygenerowanych plików

Po generowaniu Claude podaje linki do pobrania w formacie:
```
https://twoja-domena.pl/download/NazwaPliku.docx
```

Pliki dostępne są przez określony czas (domyślnie do restartu serwera).

---

## Typowe scenariusze

### Umowa wdrożeniowa z oferty handlowca

> „Mam ofertę od handlowca na wdrożenie systemu CCTV. Klient to Firma XYZ, NIP 9876543210, kwota 25 000 zł netto + abonament 800 zł/mies. Wygeneruj komplety dokumentów."

Wynik: 4 pliki DOCX (umowa + 3 załączniki)

### Szybka umowa SaaS

> „Wygeneruj umowę SaaS dla nowego klienta ABC Sp. z o.o. (NIP 5213657029), plan Business, 299 zł/mies. netto."

Wynik: 2 pliki DOCX (umowa SaaS + załącznik)

### Sprawdzenie danych przed generowaniem

> „Najpierw sprawdź firmę o NIP 5213657029, potem wygeneruj umowę SLA."

Claude najpierw wywoła `lookup_company`, pokaże wyniki, a po potwierdzeniu — wygeneruje umowę.

---

## Problemy i ich rozwiązania

**PDF nie daje dobrych wyników**  
PDFy będące skanami (zdjęciami) nie są czytelne dla parsera. Rozwiązanie: skopiuj tekst z PDF i wklej bezpośrednio.

**Brak NIP w ofercie**  
Serwer spróbuje odszukać firmę po nazwie. Wyniki mogą być mniej dokładne. Zawsze lepiej podać NIP.

**Nie pasuje żaden szablon**  
System nie rozpoznał typu umowy. Doprecyzuj:
> „Wygeneruj umowę wdrożeniową (nie SaaS)"

**Zła kwota w umowie**  
Edytuj wygenerowany plik DOCX ręcznie lub podaj precyzyjniej:
> „Kwota wdrożenia: 15 000 zł netto, abonament: 500 zł/mies. netto"
