# Návrh řešení: REST API pro automatizaci kódování a vykazování péče pojišťovnám

## Úvod

Tento dokument shrnuje analýzu požadavků, návrh řešení a plán implementace pro **REST API službu**, jejímž cílem je automatizovat a validovat kódování a vykazování zdravotní péče pojišťovnám v České republice. Projekt se zaměřuje na dva klíčové scénáře: **validaci existujících K-dávek** a **automatické kódování klinických událostí** pomocí umělé inteligence.

## 1. Analýza a kontext

### 1.1. Vstupní formát: K-dávka (DASTA)

Na základě přiloženého souboru (`KDAVKA201-Výsledkyhledání.201`) bylo potvrzeno, že vstupním formátem je **K-dávka** ve standardu **DASTA** (Datový Standard Ministerstva Zdravotnictví ČR). Jedná se o textový, řádkově orientovaný formát s pevnou šířkou polí.

Klíčové bloky pro vykazování:
*   **A (Ambulantní péče):** Hlava ošetření/návštěvy.
*   **V (Výkon):** Detail vykázaného zdravotního výkonu (kód výkonu, datum, diagnóza).
*   **G (Globální diagnóza):** Hlavní diagnóza vázaná k ošetření.

### 1.2. Klíčové User Stories

| User Story | Vstup | Výstup | Technologie |
| :--- | :--- | :--- | :--- |
| **1. Validace K-dávky** | Soubor K-dávky (DASTA) | JSON s detailní validací, chybami, varováními a doporučeními pro opravu kódů. | DASTA Parser, Business Logic Engine (Pravidla) |
| **2. Kódování klinické události** | Textový popis klinické události (JSON) | JSON s navrženými kódy diagnóz (MKN-10) a výkonů (SZV) s odůvodněním (snippety z textu). | LLM/AI Engine, Business Logic Engine (Pravidla) |

## 2. Návrh řešení: REST API

Navrhujeme implementaci **REST API** s využitím **FastAPI** (Python) pro rychlost a robustnost.

### 2.1. API Rozhraní (Endpoints)

| Funkce | Metoda | Endpoint | Vstupní formát |
| :--- | :--- | :--- | :--- |
| **Validace K-dávky** | `POST` | `/api/v1/coding/validate-kdavka` | `multipart/form-data` (DASTA soubor) |
| **Kódování události** | `POST` | `/api/v1/coding/code-clinical-event` | `application/json` (Text klinické události) |

### 2.2. Logika Validace (User Story 1)

Validace probíhá ve třech hlavních krocích:

1.  **Formální validace:** Kontrola správnosti formátu DASTA a parsování do strukturovaného JSON.
2.  **Validace číselníků:** Kontrola existence a platnosti kódů výkonů (SZV) a diagnóz (MKN-10) k datu výkonu.
3.  **Validace pravidel:** Aplikace komplexních úhradových pravidel:
    *   **Frekvenční limity:** Kontrola, zda nebyl překročen povolený počet vykázání výkonu za dané období.
    *   **Vazba Výkon-Diagnóza:** Kontrola, zda je výkon vykázán s relevantní diagnózou (např. výkon kardiologa s diagnózou z kapitoly I - nemoci oběhové soustavy).
    *   **Kombinace výkonů:** Kontrola vzájemně se vylučujících výkonů.

### 2.3. Logika Kódování (User Story 2)

Kódování využívá hybridní přístup:

1.  **LLM Analýza:** **Large Language Model** (např. Google Gemini) analyzuje text klinické události, extrahuje symptomy a navrhuje nejpravděpodobnější **MKN-10 diagnózu** a sadu **SZV výkonů**.
2.  **Generování odůvodnění:** LLM generuje **snippety** z klinického textu, které slouží jako důkaz pro přiřazení konkrétního kódu.
3.  **Ověření pravidel:** Navržené kódy jsou zpětně ověřeny proti databázi pravidel (viz 2.2.), aby byla zajištěna jejich **vykazatelnost** a **úhrada** pojišťovnou.

## 3. Architektura a implementace

### 3.1. Architektonické komponenty

| Komponenta | Účel | Technologie |
| :--- | :--- | :--- |
| **API Server** | Přijímání požadavků a odesílání odpovědí | FastAPI (Python) |
| **DASTA Parser** | Transformace DASTA textu na JSON | Vlastní modul (`dasta_parser.py`) |
| **Validation Engine** | Aplikace úhradových pravidel | Vlastní modul (`validation_engine.py`) |
| **LLM/AI Engine** | Analýza klinického textu | Google Gemini API / OpenAI API |
| **Databáze** | Uložení číselníků a pravidel | PostgreSQL |

### 3.2. Datový model (Klíčové tabulky)

Pro funkčnost systému je klíčová databáze s aktuálními číselníky:

*   `Ciselnik_Vykonu`: Kódy výkonů, názvy, bodové hodnoty, frekvenční limity.
*   `Ciselnik_Diagnoz`: Kódy MKN-10 a jejich názvy.
*   `Pravidla_Vykazovani`: Vazby mezi výkony a diagnózami, pravidla pro kombinace a frekvence.

### 3.3. Plán implementace (Přehled)

Implementace je rozdělena do šesti hlavních iterací s celkovou dobou trvání přibližně **15-16 týdnů**:

| Iterace | Zaměření | Doba trvání |
| :--- | :--- | :--- |
| **1.** | Základní infrastruktura (FastAPI, DB, Docker) | 1-2 týdny |
| **2.** | DASTA Parser a testy | 2 týdny |
| **3.** | Databáze a Číselníky (naplnění daty) | 2 týdny |
| **4.** | Validation Engine (Validace pravidel a doporučení) | 3 týdny |
| **5.** | LLM Integrace (Kódování klinických událostí) | 3 týdny |
| **6.** | Testování, Optimalizace a Dokumentace | 3-4 týdny |

## Závěr

Navržené řešení poskytuje komplexní a moderní platformu pro automatizaci kódování. Kombinací robustního **DASTA parseru** a **validátoru pravidel** s pokročilou **analýzou textu pomocí LLM** je možné výrazně snížit chybovost ve vykazování, optimalizovat úhrady a ušetřit čas zdravotnickému personálu.

Pro zahájení implementace je klíčové zajistit přístup k aktuálním a kompletním datům **Seznamu zdravotních výkonů** a **úhradových vyhlášek**.
