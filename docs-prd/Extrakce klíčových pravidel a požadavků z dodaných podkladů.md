# Extrakce klíčových pravidel a požadavků z dodaných podkladů

Na základě analýzy Product Requirements Document (`PRD.md`) a Metodiky vyúčtování VZP (`metodika-vyuctovani-datove-rozhrani-individualnich-dokladu.pdf`) byly extrahovány následující klíčové požadavky a pravidla pro implementaci.

## 1. Klíčové požadavky (z PRD.md)

| Požadavek | User Story | Implementační poznámka |
| :--- | :--- | :--- |
| **Kontrola platnosti kódů** | US1 | Nutná databáze s daty platnosti (od/do) pro SZV a MKN-10. |
| **Kontrola kombinací diagnóz a výkonů** | US1 | Vyžaduje tabulku `Pravidla_Vykazovani` (viz předchozí návrh). |
| **Kontrola frekvenčních omezení** | US1 | Nutné uložit frekvenční limity (např. 1x/90D) v `Ciselnik_Vykonu` a implementovat logiku kontroly historie. |
| **Generování kódů z textu** | US2 | Primární funkce LLM Engine. Musí vracet MKN-10 a SZV. |
| **Odůvodnění kódů (Snippety)** | US2 | LLM musí extrahovat relevantní textové pasáže pro transparentnost. |
| **Podpora formátu KDAVKA.XXX** | US1 | Vyžaduje implementaci **DASTA Parseru** pro pevné šířky polí. |
| **Podpora formátu JSON** | US1, US2 | Primární komunikační formát API. Doporučena FHIR kompatibilní struktura. |

## 2. Detailní specifikace DASTA (z Metodiky VZP)

Metodika VZP potvrzuje strukturu K-dávky a poskytuje přesné definice polí pro klíčové věty.

### A. Věta typu "A" - Záhlaví dokladu (Ambulantní péče)

| Název | Typ | Délka | Začátek | Popis |
| :--- | :--- | :--- | :--- | :--- |
| `TYP` | C | 1 | 0 | Typ věty, "A" - záhlaví |
| `HCID` | N | 7 | 1 | Číslo dokladu |
| `HTPP` | C | 1 | 16 | Typ pojištění (pro zákonné pojištění se vyplňuje kód '1') |
| `HICO` | C | 8 | 17 | Identifikační číslo pracoviště (IČP) |
| `HZDG` | C | 5 | 44 | **Číslo základní diagnózy** (MKN-10) |
| `HCEL` | S | 12.2 | 79 | Cena celkem - nepovinný údaj |
| `HCBOD` | N | 9 | 91 | **Body celkem** - nepovinný údaj |

### B. Věta typu "V" - Výkony

| Název | Typ | Délka | Začátek | Popis |
| :--- | :--- | :--- | :--- | :--- |
| `TYP` | C | 1 | 0 | Typ věty, "V" - výkony |
| `VDAT` | D | 8 | 1 | **Datum provedení výkonu** (DDMMRRRR) |
| `VKOD` | C | 5 | 9 | **Číslo výkonu** (SZV) |
| `VODB` | C | 3 | 14 | **Odbornost** (uvádí se povinně) |
| `VDIA` | C | 5 | 18 | **Diagnóza** (MKN-10) - je-li uvedena základní diagnóza, uvádí se jen u výkonů, které se k základní diagnóze nevztahují. |
| `VBOD` | N | 5 | 23 | **Body za výkon** - nepovinný údaj |

## 3. Extrakce klíčových pravidel pro Validaci (US1)

Z metodiky a PRD vyplývají následující kategorie pravidel, které musí `Validation Engine` implementovat:

### A. Formální a číselníková pravidla (ERROR)

1.  **Platnost kódu:** `VKOD` (číslo výkonu) musí existovat v `Ciselnik_Vykonu` a být platný k datu `VDAT`.
2.  **Platnost diagnózy:** `HZDG` (záhlaví) a `VDIA` (výkon) musí existovat v `Ciselnik_Diagnoz`.
3.  **Povinná pole:** Kontrola, zda jsou vyplněna povinná pole (např. `VDAT`, `VKOD`).

### B. Logická pravidla (ERROR / WARNING)

1.  **Vazba Výkon-Odbornost:** Odbornost výkonu (`VODB`) musí odpovídat odbornosti pracoviště (`HODB` z hlavičky) nebo být v rámci smluvního ujednání.
2.  **Frekvenční limit:** Kontrola, zda počet vykázání `VKOD` pro daného pacienta nepřekračuje limit definovaný v `Ciselnik_Vykonu` (např. 1x/90D).
3.  **Vazba Výkon-Diagnóza:** Kontrola, zda je výkon vykázán s relevantní diagnózou. (Např. výkon je vázán na diagnózu z určité kapitoly MKN-10).
4.  **Úplnost dokladu:** Kontrola, zda k základní diagnóze (`HZDG`) je vykázán alespoň jeden relevantní výkon.

## 4. Příprava na Fázi 2 a 3

Dodaný materiál poskytuje dostatečný základ pro detailní specifikaci:
*   **Fáze 2 (DASTA Parser a DB schéma):** Přesné pozice polí z metodiky VZP umožní vytvořit robustní DASTA parser a přesné SQL schéma.
*   **Fáze 3 (LLM Prompty):** Dokumenty MKN-10 (tabelární část, instrukční příručka) budou sloužit jako klíčový kontext pro LLM, aby generoval správné kódy a odůvodnění.

Nyní přecházím k Fázi 2: Detailní specifikace DASTA Parseru a návrh datového modelu.
