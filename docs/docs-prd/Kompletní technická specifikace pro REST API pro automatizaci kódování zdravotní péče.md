# Kompletní technická specifikace pro REST API pro automatizaci kódování zdravotní péče

Tento dokument shrnuje kompletní technickou specifikaci pro implementaci REST API služby, která automatizuje validaci K-dávek a generování kódů z klinických událostí.

## 1. Architektura a technologický stack

Doporučený stack je **FastAPI (Python)** pro backend, **PostgreSQL** pro databázi a **Google Gemini** pro LLM Engine.

| Komponenta | Technologie | Role |
| :--- | :--- | :--- |
| **API Gateway** | FastAPI | Asynchronní příjem požadavků a směrování. |
| **DASTA Parser** | Python (vlastní kód) | Transformace textové K-dávky na strukturovaný JSON. |
| **Validation Engine** | Python + PostgreSQL | Aplikace úhradových pravidel (US1). |
| **LLM Engine** | Python + Gemini API | Analýza klinického textu a návrh kódů (US2). |
| **Databáze** | PostgreSQL | Uložení číselníků (MKN-10, SZV) a pravidel. |

## 2. User Story 1: Validace vykazovacích kódů (DASTA)

### A. DASTA Parser

Parser transformuje K-dávku (fixed-width text) na strukturovaný JSON. Klíčové věty "A" (záhlaví) a "V" (výkony) jsou parsovány na základě přesných pozic polí definovaných v Metodice VZP.

**Příklad parsování věty "V" (Výkon):**
*   Datum výkonu (`VDAT`): Znaky 1-8 (DDMMRRRR)
*   Kód výkonu (`VKOD`): Znaky 9-13
*   Diagnóza (`VDIA`): Znaky 18-22

### B. Validation Engine (Logika)

Validation Engine provede kontroly ve třech fázích:

| Fáze | Kontrola | Závažnost | Zdroj dat |
| :--- | :--- | :--- | :--- |
| **1. Formální** | Platnost kódu (existence v číselníku), formát datumu, vyplnění povinných polí. | ERROR | `ciselnik_vykonu`, `ciselnik_diagnoz` |
| **2. Logická** | Vazba výkon-diagnóza, vazba výkon-odbornost, kombinace výkonů. | ERROR / WARNING | `pravidla_validace` |
| **3. Frekvenční** | Překročení frekvenčního limitu (např. 1x/90D) pro daného pacienta. | WARNING | `ciselnik_vykonu`, `historie_vykazovani` |

## 3. User Story 2: Generování kódů z klinických událostí (LLM)

### A. LLM Prompt Engineering

Pro zajištění přesnosti a transparentnosti je klíčové použití **System Promptu** a **JSON módu**.

*   **System Prompt:** Nastaví roli LLM jako experta na české kódování, vloží pravidla z MKN-10 Instrukční příručky a definuje požadovaný JSON výstup.
*   **User Prompt:** Obsahuje text klinické události a kontext pacienta (věk, pohlaví, komorbidity).

### B. Požadovaný JSON výstup

LLM musí vrátit strukturovaný JSON, který obsahuje:

*   `main_diagnosis`: Kód MKN-10 a **přesný textový úryvek** (snippet) z klinické události jako odůvodnění.
*   `secondary_diagnoses`: Seznam sekundárních diagnóz se snippety.
*   `performance_recommendations`: Seznam doporučených kódů SZV s odůvodněním a vazbou na MKN-10.

## 4. Datový model (PostgreSQL)

Navržené SQL schéma pro PostgreSQL:

| Tabulka | Účel | Klíčové sloupce |
| :--- | :--- | :--- |
| `ciselnik_vykonu` | SZV, bodové hodnoty, frekvenční limity. | `kod_vykonu`, `platnost_od`, `frekvence_limit_dny` |
| `ciselnik_diagnoz` | MKN-10, kategorizace (kapitola). | `kod_diagnozy`, `kapitola` |
| `pravidla_validace` | Komplexní vazby a úhradová pravidla. | `typ_pravidla`, `kod_vykonu_a`, `diagnoza_pattern`, `zavaznost` |
| `historie_vykazovani` | Historie pacienta pro kontrolu frekvence. | `patient_id`, `performance_code`, `date_of_performance` |

## 5. Závěr

Tato technická specifikace poskytuje kompletní plán pro vývoj REST API. Kombinace robustního **DASTA Parseru**, relačního **Validation Engine** a moderního **LLM Engine** zajistí, že produkt bude přesný, spolehlivý a připravený na integraci do ambulantních informačních systémů.
