# Detailní specifikace DASTA Parseru a návrh datového modelu (SQL schéma)

Tato fáze detailně specifikuje implementaci **DASTA Parseru** pro User Story 1 a navrhuje **SQL schéma** pro uložení číselníků a pravidel, které jsou nezbytné pro `Validation Engine`.

## 1. Specifikace DASTA Parseru (KDAVKA.XXX)

DASTA Parser bude implementován v Pythonu a bude transformovat textový soubor s pevnou šířkou polí (fixed-width) do strukturovaného JSON objektu. Zaměříme se na klíčové věty "A" (záhlaví dokladu) a "V" (výkony).

### A. Věta typu "A" - Záhlaví dokladu (Ambulantní péče)

| Název pole | Typ | Délka | Začátek (Index 0) | Konec (Index 0) | Popis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `TYP` | C | 1 | 0 | 0 | Typ věty, "A" |
| `HCID` | N | 7 | 1 | 7 | Číslo dokladu |
| `HPOR` | N | 3 | 10 | 12 | Pořadové číslo listu v dávce |
| `HTPP` | C | 1 | 16 | 16 | Typ pojištění (1=zákonné) |
| `HICO` | C | 8 | 17 | 24 | Identifikační číslo pracoviště (IČP) |
| `HODB` | C | 3 | 31 | 33 | Smluvní odbornost pracoviště |
| `HROD` | C | 10 | 34 | 43 | Číslo pojištěnce (Rodné číslo) |
| `HZDG` | C | 5 | 44 | 48 | **Číslo základní diagnózy** (MKN-10) |
| `HCEL` | S | 12.2 | 79 | 90 | Cena celkem (nepovinné) |
| `HCBOD` | N | 9 | 91 | 99 | **Body celkem** (nepovinné) |

### B. Věta typu "V" - Výkony

| Název pole | Typ | Délka | Začátek (Index 0) | Konec (Index 0) | Popis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `TYP` | C | 1 | 0 | 0 | Typ věty, "V" |
| `VDAT` | D | 8 | 1 | 8 | **Datum provedení výkonu** (DDMMRRRR) |
| `VKOD` | C | 5 | 9 | 13 | **Číslo výkonu** (SZV) |
| `VODB` | C | 3 | 14 | 16 | **Odbornost** (uvádí se povinně) |
| `VDIA` | C | 5 | 18 | 22 | **Diagnóza** (MKN-10) vázaná k výkonu |
| `VBOD` | N | 5 | 23 | 27 | **Body za výkon** (nepovinné) |

### C. Výstupní JSON struktura (DASTA Parser)

Parser transformuje K-dávku do JSON struktury, která bude sloužit jako vstup pro `Validation Engine`.

```json
{
  "document_id": "HCID",
  "provider_icp": "HICO",
  "provider_specialization": "HODB",
  "patient_id": "HROD",
  "main_diagnosis_code": "HZDG",
  "total_points": "HCBOD",
  "visits": [
    {
      "visit_id": "HPOR",
      "date": "VDAT",
      "performance_code": "VKOD",
      "performance_specialization": "VODB",
      "diagnosis_code": "VDIA",
      "points": "VBOD"
    }
    // ... další výkony
  ]
}
```

## 2. Návrh datového modelu (SQL schéma)

Pro uložení číselníků a pravidel v PostgreSQL navrhujeme následující schéma.

### A. Tabulka: `ciselnik_vykonu` (Seznam zdravotních výkonů - SZV)

Tato tabulka je klíčová pro validaci platnosti, frekvenčních limitů a bodových hodnot.

```sql
CREATE TABLE ciselnik_vykonu (
    id SERIAL PRIMARY KEY,
    kod_vykonu VARCHAR(5) UNIQUE NOT NULL,
    nazev VARCHAR(255) NOT NULL,
    bodova_hodnota DECIMAL(10, 2) NOT NULL,
    odbornost VARCHAR(3) NOT NULL,
    platnost_od DATE NOT NULL,
    platnost_do DATE,
    frekvence_limit_dny INTEGER,  -- Např. 90 pro 1x/90D
    frekvence_limit_pocet INTEGER, -- Např. 1
    is_preventivni BOOLEAN DEFAULT FALSE,
    -- Index pro rychlé vyhledávání
    UNIQUE (kod_vykonu, platnost_od)
);
```

### B. Tabulka: `ciselnik_diagnoz` (MKN-10)

Základní číselník pro kontrolu existence a kategorizace diagnóz.

```sql
CREATE TABLE ciselnik_diagnoz (
    id SERIAL PRIMARY KEY,
    kod_diagnozy VARCHAR(5) UNIQUE NOT NULL,
    nazev VARCHAR(255) NOT NULL,
    kapitola VARCHAR(5) NOT NULL, -- Např. 'I' pro nemoci oběhové soustavy
    podkapitola VARCHAR(10),
    is_hlavni_diagnoza BOOLEAN DEFAULT TRUE -- Zda může být hlavní diagnózou
);
```

### C. Tabulka: `pravidla_validace` (Logika úhrad)

Tato tabulka bude obsahovat komplexní pravidla extrahovaná z metodik VZP a úhradových vyhlášek.

```sql
CREATE TABLE pravidla_validace (
    id SERIAL PRIMARY KEY,
    typ_pravidla VARCHAR(50) NOT NULL, -- Např. 'VAZBA_VYKON_DIAGNOZA', 'KOMBINACE_VYKONU'
    kod_vykonu_a VARCHAR(5) NOT NULL, -- Kód výkonu, na který se pravidlo vztahuje
    kod_vykonu_b VARCHAR(5),          -- Kód výkonu pro pravidla kombinace
    diagnoza_pattern VARCHAR(10),     -- MKN-10 pattern (např. 'I%', 'K30')
    zavaznost VARCHAR(10) NOT NULL,   -- 'ERROR' nebo 'WARNING'
    popis_chyby TEXT NOT NULL,        -- Popis chyby pro výstup API
    logika_sql TEXT,                  -- Volitelné: SQL fragment pro komplexní pravidla
    platnost_od DATE NOT NULL,
    platnost_do DATE
);
```

### D. Tabulka: `historie_vykazovani` (Pro frekvenční limity)

Tato tabulka by byla v reálném provozu plněna daty z AIS nebo by API muselo mít přístup k historii pacienta. Pro účely validace frekvenčních limitů je nezbytná.

```sql
CREATE TABLE historie_vykazovani (
    id BIGSERIAL PRIMARY KEY,
    patient_id VARCHAR(10) NOT NULL,
    performance_code VARCHAR(5) NOT NULL,
    date_of_performance DATE NOT NULL,
    provider_icp VARCHAR(8) NOT NULL,
    -- Index pro rychlé vyhledávání historie pacienta
    INDEX (patient_id, performance_code, date_of_performance)
);
```

## 3. Příprava na Fázi 3: LLM Prompty

V další fázi využijeme dodané dokumenty MKN-10 k vytvoření kontextu pro LLM.

*   **MKN-10 Tabelární část:** Poskytne strukturu a detailní popis diagnóz.
*   **MKN-10 Instrukční příručka:** Poskytne pravidla pro správné kódování (např. pravidla pro hlavní diagnózu, pravidla pro kódování symptomů).

Tyto informace budou integrovány do systémového promptu LLM, aby se zajistila maximální přesnost a soulad s českou metodikou kódování.
