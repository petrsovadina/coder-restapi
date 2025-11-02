# Závěrečná zpráva: Analýza trhu, potenciálu a monetizační strategie pro REST API pro automatizaci kódování zdravotní péče

## 1. Shrnutí projektu a potenciálu

Váš projekt, **REST API pro automatizaci kódování a vykazování péče pojišťovnám**, řeší kritický problém v českém zdravotnictví: komplexnost a chybovost vykazování. Kombinace **Validace K-dávky** (User Story 1) a **LLM Kódování klinických událostí** (User Story 2) vytváří silnou a inovativní hodnotovou nabídku.

### 1.1. Potenciální úspěšnost a nasycení trhu

| Aspekt | Zjištění | Hodnocení |
| :--- | :--- | :--- |
| **Tržní potenciál (TAM)** | **~19 000** ambulantních poskytovatelů zdravotní péče (PZS) v ČR. | **Velmi vysoký** |
| **Konkurence (Validace)** | Stávající AIS nabízejí jen základní validaci. Chybí komplexní, dynamický "pre-validátor". | **Střední nasycení** |
| **Konkurence (LLM Kódování)** | Trh je prakticky **nenasycený**. Neexistuje komerčně dostupné řešení pro automatické kódování volného textu. | **Nízké nasycení** |
| **Klíčová výhoda** | **LLM Kódování** (User Story 2) je **disruptivní** a představuje hlavní konkurenční výhodu. | **Extrémně vysoká** |

## 2. Analýza trhu (TAM, SAM, SOM)

Analýza potvrdila dostatečnou velikost trhu pro vybudování ziskového podniku.

| Metrika | Popis | Odhad (PZS) |
| :--- | :--- | :--- |
| **TAM** | Celkový dostupný trh (všichni ambulantní PZS) | **~19 000** |
| **SAM** | Obsluhovatelný dostupný trh (PZS otevřené API integraci) | **~4 750** |
| **SOM** | Realisticky dosažitelný trh (první 3 roky) | **~1 500 - 2 000** |

## 3. Návrh monetizační strategie a cenová struktura

Doporučujeme **Hybridní model** (SaaS + Pay-per-Use), který optimalizuje stabilní příjem a zároveň maximalizuje zisk z dodané hodnoty.

### 3.1. Cenová struktura (Validace K-dávky - User Story 1)

Cena je nastavena tak, aby byla zlomkem úspory, kterou PZS získá snížením chybovosti a maximalizací úhrad.

| Úroveň | Měsíční poplatek (SaaS) | Transakční poplatek (za 1 validovanou návštěvu) |
| :--- | :--- | :--- |
| **Basic** | **1 290 Kč** | **1,80 Kč** |
| **Standard** | **2 490 Kč** | **1,40 Kč** |
| **Premium** | **4 990 Kč** | **0,90 Kč** |

### 3.2. Cenová struktura (LLM Kódování - User Story 2)

Tato služba je dražší, protože přináší vyšší úsporu času lékaře a má vyšší provozní náklady (LLM API).

*   **Doporučená cena za volání API:** **10 - 12 Kč** za analýzu jedné klinické události.
*   **Odůvodnění:** Tato cena je výrazně nižší než minutová mzda lékaře, kterou by strávil ručním kódováním komplexní události.

### 3.3. Finanční potenciál (Odhad SOM)

Při dosažení realistického cíle **1 500 PZS** (SOM) je odhadovaný roční příjem:

| Příjmový kanál | Odhadovaný roční příjem |
| :--- | :--- |
| **SaaS Předplatné** | ~36 000 000 Kč |
| **Validace K-dávky (Transakce)** | ~12 600 000 Kč |
| **LLM Kódování (Transakce)** | ~9 000 000 Kč |
| **Celkový roční příjem (Odhad SOM)** | **~57 600 000 Kč** |

## 4. Doporučení pro další kroky

1.  **Prioritizace LLM Kódování:** Zaměřte se na rychlý vývoj a testování User Story 2. Je to váš hlavní **diferenciátor** a největší lákadlo pro trh.
2.  **Partnerství s AIS:** Navazujte kontakty s menšími dodavateli Ambulantních IS. Nabídněte jim API jako **prémiový modul** pro jejich zákazníky.
3.  **Zajištění dat:** Pro implementaci je klíčové zajistit aktuální a kompletní data **Seznamu zdravotních výkonů** a **úhradových vyhlášek** pro vytvoření robustního datového modelu.

Všechny detailní analýzy a návrhy (analýza trhu, konkurence, monetizace, architektura a implementace) jsou obsaženy v přiložených souborech z předchozí fáze.

Jsem připraven vám pomoci s dalším krokem, ať už se jedná o detailní specifikaci API, nebo přípravu podkladů pro jednání s potenciálními partnery.
