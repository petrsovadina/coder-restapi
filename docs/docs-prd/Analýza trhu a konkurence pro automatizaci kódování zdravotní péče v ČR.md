# Analýza trhu a konkurence pro automatizaci kódování zdravotní péče v ČR

## 1. Analýza konkurence (User Story 1: Validace K-dávky)

Hlavní konkurence pro validaci K-dávky a vykazování péče pojišťovnám se nachází v rámci stávajících **Ambulantních Informačních Systémů (AIS)** a specializovaných softwarů.

| Konkurent / Segment | Řešení | Funkce | Slabiny / Příležitost pro API |
| :--- | :--- | :--- | :--- |
| **Ambulantní IS** | SmartMEDIX, DUNA MEDIK, PROMEDIC – SERVICE, AIS (obecně) | Zahrnují modul pro tvorbu a odesílání K-dávek. Některé (např. AIS) již implementují základní **hlídání podmíněných a doplňkových kódů** [1]. | **Omezená hloubka validace:** Většinou se zaměřují na formální validaci DASTA a základní pravidla. Chybí jim komplexní, dynamická kontrola proti všem úhradovým vyhláškám a metodikám všech pojišťoven. |
| **Specializované SW** | Vitalsoft (zaměřený na cenu) | Nabízí program za nízkou cenu s asistencí při vykazování [2]. | **Chybí API:** Jsou to desktopové/lokální aplikace. Nejsou připraveny na moderní integraci přes REST API. |
| **Pojišťovny** | Vlastní kontrolní mechanismy | Pojišťovny provádějí finální kontrolu a zamítají chybné dávky. | **Pozdní zpětná vazba:** Lékař se o chybě dozví až po odeslání dávky, což je časově náročné na opravu. |

**Závěr k User Story 1:** Konkurence existuje, ale je roztříštěná a neposkytuje **komplexní, centralizovanou a prediktivní validaci v reálném čase** před odesláním dávky. API řešení má potenciál stát se **"pre-validátorem"** pro všechny IS.

## 2. Analýza konkurence (User Story 2: Kódování klinické události)

Tato oblast je v ČR stále v plenkách a představuje největší příležitost.

| Konkurent / Segment | Řešení | Funkce | Slabiny / Příležitost pro API |
| :--- | :--- | :--- | :--- |
| **Ambulantní IS** | Většinou ruční kódování | Kódy se vybírají ručně z číselníků nebo pomocí šablon. | **Chybí AI/LLM:** Neexistuje komerčně dostupné řešení, které by analyzovalo volný text (diktát, anamnézu) a navrhlo správné kódy MKN-10 a SZV. |
| **Akademické/Státní projekty** | Projekty ÚZIS (např. AI pro podporu sběru dat) [3] | Zaměřeno na sběr, validaci a zhodnocení dat, nikoli na komerční automatizaci kódování pro vykazování. | **Nejsou komerční:** Nejsou dostupné jako API pro integraci do IS. |

**Závěr k User Story 2:** Trh je v podstatě **nenasycený**. Řešení s využitím LLM pro automatické kódování volného textu představuje **významnou konkurenční výhodu** a potenciální **game-changer** v oboru.

## 3. Analýza potenciálního trhu (TAM, SAM, SOM)

Pro odhad trhu se zaměříme na segment **ambulantních poskytovatelů zdravotní péče (PZS)**, kteří jsou primárními uživateli K-dávek.

### A. Total Addressable Market (TAM) - Celkový dostupný trh

*   **Cílová skupina:** Všichni ambulantní poskytovatelé, kteří vykazují péči pojišťovnám.
*   **Odhad počtu PZS:**
    *   Ambulance lékařů specialistů: **~7 127** [4]
    *   Ambulance ženských lékařů: **~1 129** [4]
    *   Ambulance zubních lékařů: **~5 629** [4]
    *   Ambulance praktických lékařů (PL): **~5 000** (odhad)
    *   **Celkem TAM:** Přibližně **19 000** ambulantních PZS.

### B. Serviceable Available Market (SAM) - Obsluhovatelný dostupný trh

*   **Cílová skupina:** PZS, které jsou ochotny integrovat externí API řešení do svého IS nebo používat externí webovou aplikaci.
*   **Odhad:** Realisticky 20-30% TAM.
*   **SAM:** **~3 800 - 5 700** PZS.

### C. Serviceable Obtainable Market (SOM) - Realisticky dosažitelný trh (první 3 roky)

*   **Cílová skupina:** PZS, které se podaří získat v prvních letech, pravděpodobně menší ambulance, specialisté s vysokou frekvencí výkonů nebo PZS s problémy s úhradami.
*   **Odhad:** 5-10% TAM.
*   **SOM:** **~950 - 1 900** PZS.

**Potenciál:** Trh je velký a stabilní. I při získání 5% trhu (cca 950 PZS) se jedná o významný komerční potenciál.

## 4. Návrh monetizační strategie a cenové struktury

Vzhledem k povaze služby (API, úspora času, maximalizace úhrad) je nejvhodnější **transakční model** kombinovaný s **předplatným**.

### A. Monetizační strategie: Hybridní model (SaaS + Pay-per-Use)

1.  **Základní předplatné (SaaS):** Pokrývá náklady na údržbu číselníků, API infrastrukturu a základní podporu.
2.  **Transakční poplatek (Pay-per-Use):** Účtování za každou transakci, což přímo koreluje s hodnotou, kterou služba přináší (úspora času, správná úhrada).

### B. Cenová struktura (Návrh)

Navrhujeme rozdělení do tří úrovní, které reflektují objem a komplexnost využití.

| Úroveň | Cílová skupina | Měsíční poplatek (SaaS) | Transakční poplatek (za 1 návštěvu/validaci) |
| :--- | :--- | :--- | :--- |
| **Basic** | Malé ambulance (PL, specialisté s nízkou frekvencí) | **990 - 1 490 Kč** | **2,00 Kč** / návštěva (Validace K-dávky) |
| **Standard** | Střední ambulance, specialisté s vysokou frekvencí | **1 990 - 2 990 Kč** | **1,50 Kč** / návštěva (Validace K-dávky) |
| **Premium** | Větší kliniky, polikliniky, IS dodavatelé (B2B) | **4 990 - 9 990 Kč** | **1,00 Kč** / návštěva (Validace K-dávky) |

### C. Pricing pro LLM kódování (User Story 2)

Kódování klinické události pomocí LLM je nákladově náročnější (API volání LLM) a přináší vyšší hodnotu (úspora času lékaře).

*   **Model:** Pay-per-Use s vyšší cenou.
*   **Cena:** **10 - 15 Kč** za jedno volání endpointu `/api/v1/coding/code-clinical-event` (za analýzu jednoho textu/klinické události).
*   **Odůvodnění:** Tato cena je zlomkem ceny, kterou by klinika zaplatila za 5-10 minut času lékaře nebo kodéra.

### D. Finanční potenciál (Příklad)

Předpokládejme dosažení **SOM** (1 500 PZS) s průměrným měsíčním poplatkem 2 000 Kč a průměrným počtem 500 validací měsíčně na PZS.

| Položka | Výpočet | Roční příjem (Kč) |
| :--- | :--- | :--- |
| **Předplatné (SaaS)** | 1 500 PZS * 2 000 Kč/měsíc * 12 měsíců | **36 000 000 Kč** |
| **Transakce (Validace)** | 1 500 PZS * 500 transakcí/měsíc * 1,50 Kč/transakce * 12 měsíců | **13 500 000 Kč** |
| **LLM Kódování (Odhad)** | 1 500 PZS * 50 LLM volání/měsíc * 12 Kč/volání * 12 měsíců | **10 800 000 Kč** |
| **Celkový roční příjem (Odhad SOM)** | | **~60 300 000 Kč** |

**Závěr:** Projekt má na českém trhu velký potenciál. Kombinace silné validace (User Story 1) a inovativního LLM kódování (User Story 2) vytváří silnou hodnotovou nabídku, která ospravedlňuje navrhovanou cenovou strukturu.

---
**Reference:**
[1] Nová verze programu 3.0.9.1. AISCZ.
[2] vitalsoft.cz.
[3] Zvýšení dostupnosti onkologické a další zdravotní péče... ÚZIS.
[4] Česká republika jako ráj ambulantních specialistů? MT.
