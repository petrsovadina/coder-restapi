# Návrh monetizační strategie a cenové struktury (Pricing Strategy)

Na základě analýzy trhu, konkurence a vysokého potenciálu obou User Stories (Validace K-dávky a LLM Kódování) navrhujeme **hybridní monetizační model** kombinující stabilní příjem z předplatného (SaaS) s příjmy založenými na spotřebě (Pay-per-Use).

## 1. Monetizační model: Hybridní SaaS + Pay-per-Use

Tento model je ideální pro B2B služby, které přinášejí přímou a měřitelnou hodnotu (úspora času, maximalizace úhrad).

### A. SaaS (Subscription as a Service) - Měsíční poplatek

*   **Co pokrývá:** Přístup k API, údržbu a aktualizaci databáze číselníků (SZV, MKN-10), implementaci úhradových vyhlášek, technickou podporu a základní infrastrukturu.
*   **Výhoda:** Zajišťuje stabilní a předvídatelný měsíční příjem.

### B. Pay-per-Use (Transakční poplatek) - Poplatek za volání API

*   **Co pokrývá:** Poplatek za každou transakci, tj. za každou validovanou návštěvu v K-dávce (User Story 1) nebo za každou analýzu klinického textu (User Story 2).
*   **Výhoda:** Přímo koreluje s hodnotou dodanou zákazníkovi. Zákazník platí jen za to, co skutečně využije.

## 2. Cenová struktura pro Validaci K-dávky (User Story 1)

Cílem je cenově se umístit pod náklady, které PZS vznikají chybovostí (zamítnuté dávky) a časem stráveným ruční kontrolou.

| Úroveň | Cílová skupina | Měsíční poplatek (SaaS) | Transakční poplatek (za 1 validovanou návštěvu) |
| :--- | :--- | :--- | :--- |
| **Basic** | Malé ambulance (PL, specialisté s nízkou frekvencí) | **1 290 Kč** | **1,80 Kč** / návštěva |
| **Standard** | Střední ambulance, specialisté s vysokou frekvencí | **2 490 Kč** | **1,40 Kč** / návštěva |
| **Premium** | Větší kliniky, polikliniky, IS dodavatelé (B2B) | **4 990 Kč** | **0,90 Kč** / návštěva |
| **Enterprise** | Integrace do velkých AIS/Nemocnic | Individuální nabídka | Individuální nabídka |

**Odůvodnění transakčního poplatku:** Průměrná úhrada za ambulantní návštěvu se pohybuje v řádu stovek až tisíců korun. Poplatek 0,90 Kč – 1,80 Kč je zanedbatelný v porovnání s rizikem zamítnutí celé dávky nebo ztrátou úhrady za opomenutý výkon.

## 3. Cenová struktura pro LLM Kódování (User Story 2)

Tato služba přináší nejvyšší přidanou hodnotu a má vyšší provozní náklady (volání LLM API).

*   **Model:** Čistý Pay-per-Use (LLM volání) s možností balíčků.
*   **Základní cena za volání API:** **12 Kč** za jedno volání endpointu `/api/v1/coding/code-clinical-event` (analýza jednoho textu/události).

| Balíček | Počet volání / měsíc | Cena za volání | Celková cena |
| :--- | :--- | :--- | :--- |
| **Start** | 100 | 12 Kč | **1 200 Kč** |
| **Pro** | 500 | 10 Kč | **5 000 Kč** |
| **Unlimited** | Neomezeno (FUP) | 8 Kč | **Individuální** |

**Odůvodnění ceny:** Průměrná minutová mzda lékaře v ČR se pohybuje kolem 1-2 Kč/min. Pokud LLM ušetří lékaři 5 minut času na kódování jedné komplexní události, úspora času je 5-10 Kč. Cena 12 Kč je tedy stále konkurenceschopná, protože LLM navíc zajišťuje vyšší přesnost a maximalizaci úhrady.

## 4. Strategické doporučení a finanční potenciál

### A. Strategie vstupu na trh

1.  **Zaměření na specialisty:** Cílit na ambulantní specialisty (kardiologie, neurologie, chirurgie), kde je kódování komplexnější a frekvenční limity jsou častější.
2.  **Pilotní projekt s AIS:** Navázat partnerství s menším dodavatelem Ambulantního IS (AIS) a nabídnout jim API jako **prémiový doplněk** k jejich základní validaci. To umožní rychlý vstup na trh a získání referencí.
3.  **Důraz na ROI:** Marketing by se měl zaměřit na **návratnost investice**. Např.: "Naše API vám ušetří 10 000 Kč měsíčně na zamítnutých dávkách a 5 hodin času."

### B. Odhad ročního příjmu (SOM)

Při realistickém cíli **1 500 PZS** (SOM) a průměrném využití:

| Příjmový kanál | Předpoklad | Roční příjem (Kč) |
| :--- | :--- | :--- |
| **SaaS Předplatné** | 1 500 PZS * 2 000 Kč/měsíc * 12 měsíců | **36 000 000 Kč** |
| **Validace K-dávky** | 1 500 PZS * 500 návštěv/měsíc * 1,40 Kč/návštěva * 12 měsíců | **12 600 000 Kč** |
| **LLM Kódování** | 1 500 PZS * 50 volání/měsíc * 10 Kč/volání * 12 měsíců | **9 000 000 Kč** |
| **Celkový roční příjem (Odhad SOM)** | | **~57 600 000 Kč** |

**Závěr:** Navrhovaná cenová struktura je realistická, konkurenceschopná a umožňuje dosáhnout ročního příjmu přes **50 milionů Kč** při získání 8% cílového trhu.
