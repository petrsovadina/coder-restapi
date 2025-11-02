# Návrh logiky validace a doporučování kódů a definice datového modelu

Tato fáze se zaměřuje na definici logiky pro obě klíčové funkce API a na návrh datového modelu pro uložení nezbytných číselníků a pravidel.

## 1. Datový model (Číselníky a Pravidla)

Pro správnou funkci API je nutné mít k dispozici a udržovat aktuální databázi číselníků a pravidel. Navrhujeme následující klíčové tabulky:

### Tabulka 1: Ciselnik\_Vykonu (Seznam zdravotních výkonů - SZV)

| Pole | Datový typ | Popis | Příklad |
| :--- | :--- | :--- | :--- |
| `kod_vykonu` | VARCHAR(5) | Kód výkonu (např. 01021) | `01021` |
| `nazev` | VARCHAR(255) | Název výkonu | `Základní vyšetření praktickým lékařem` |
| `odbornost` | VARCHAR(3) | Kód odbornosti, pro kterou je výkon určen | `001` |
| `bodova_hodnota` | DECIMAL(10, 2) | Bodová hodnota výkonu | `85.00` |
| `frekvence_limit` | VARCHAR(50) | Frekvenční omezení (např. 1x/3M, 1x/rok) | `1/90D` |
| `platnost_od` | DATE | Datum počátku platnosti | `2024-01-01` |
| `platnost_do` | DATE | Datum konce platnosti | `NULL` |

### Tabulka 2: Ciselnik\_Diagnoz (Mezinárodní klasifikace nemocí - MKN-10)

| Pole | Datový typ | Popis | Příklad |
| :--- | :--- | :--- | :--- |
| `kod_diagnozy` | VARCHAR(10) | Kód diagnózy (např. I10, K30) | `I10` |
| `nazev` | VARCHAR(255) | Název diagnózy | `Esenciální (primární) hypertenze` |
| `kapitola` | VARCHAR(5) | Kapitola MKN-10 | `I` |

### Tabulka 3: Pravidla\_Vykazovani (Vazby a Logika)

| Pole | Datový typ | Popis | Příklad |
| :--- | :--- | :--- | :--- |
| `pravidlo_id` | INT | Unikátní identifikátor pravidla | `101` |
| `typ_pravidla` | VARCHAR(50) | Typ pravidla (např. DIAG\_VAZBA, FREKVENCE, ODBORNOST) | `DIAG_VAZBA` |
| `kod_vykonu` | VARCHAR(5) | Kód výkonu, ke kterému se pravidlo vztahuje | `01021` |
| `kod_diagnozy` | VARCHAR(10) | Kód diagnózy, se kterou se musí/nesmí kombinovat | `Z000` |
| `logika` | TEXT | Podrobný popis logiky (např. "Musí být vykázáno s diagnózou z kapitoly I") | `Diagnoza MUSI byt z kapitoly I (I00-I99)` |
| `zavaznost` | ENUM | Závažnost (ERROR, WARNING) | `WARNING` |

---

## 2. Logika User Story 1: Validace a doporučování kódů z K-dávky

Logika validace bude probíhat v několika krocích pro každou vykázanou návštěvu (Blok A) a každý výkon (Blok V) v K-dávce.

### A. Validace formátu (Parser DASTA)

1.  **Struktura souboru:** Kontrola, zda soubor odpovídá základní struktuře DASTA (hlavička DP, bloky P, A, V, G, Z).
2.  **Validace polí:** Kontrola, zda jsou všechna povinná pole v blocích `A`, `V`, `G` vyplněna a zda odpovídají datovému typu a délce (např. datum ve formátu DDMMRRRR, kód výkonu je 5-místný).
    *   *Výstup:* **ERROR** při selhání parsování nebo chybějícím povinném poli.

### B. Validace číselníků (Business Logic Engine)

1.  **Validace Kódu Výkonu:**
    *   Kontrola existence: Je `kod_vykonu` (z bloku V) v `Ciselnik_Vykonu`?
    *   Kontrola platnosti: Je výkon platný k datu výkonu (z bloku V)?
    *   Kontrola odbornosti: Odpovídá odbornost výkonu (z `Ciselnik_Vykonu`) odbornosti poskytovatele (z bloku A)?
    *   *Výstup:* **ERROR** při neexistujícím nebo neplatném kódu.

2.  **Validace Kódu Diagnózy:**
    *   Kontrola existence: Je `kod_diagnozy` (z bloků A, V, G) v `Ciselnik_Diagnoz`?
    *   *Výstup:* **ERROR** při neexistujícím kódu.

### C. Validace pravidel (Business Logic Engine)

1.  **Frekvenční limit:**
    *   Kontrola, zda počet vykázání daného `kod_vykonu` pro daného pacienta v rámci definovaného časového limitu (např. 90 dní) nepřekračuje limit stanovený v `Ciselnik_Vykonu`.
    *   *Výstup:* **ERROR** při překročení limitu.

2.  **Vazba Výkon-Diagnóza (DIAG\_VAZBA):**
    *   Kontrola, zda je `kod_vykonu` vykázán s relevantní diagnózou (MKN-10) dle pravidel v `Pravidla_Vykazovani`.
    *   *Příklad:* Výkon "EKG" (015431) by měl být vykázán s diagnózou z kapitoly I (nemoci oběhové soustavy) nebo s diagnózou, která EKG vyžaduje. Vykázání s `Z000` (Vyšetření bez zjištěné nemoci) může být **WARNING**.
    *   *Výstup:* **WARNING** nebo **ERROR** dle závažnosti pravidla.

3.  **Kombinace výkonů:**
    *   Kontrola, zda nejsou vykázány vzájemně se vylučující výkony (např. dva různé typy základního vyšetření v jeden den).
    *   *Výstup:* **ERROR**.

### D. Doporučování kódů (Recommendation Logic)

Pokud je zjištěn **ERROR** nebo **WARNING**, systém navrhne řešení:

1.  **Chybějící kód:** Pokud je vykázán pouze jeden výkon (např. EKG), ale chybí základní vyšetření (např. 01021), systém navrhne doplnění kódu `01021` s odůvodněním "Standardní výkon pro ambulantní návštěvu".
2.  **Nevhodný kód:** Pokud je kód označen jako **ERROR** (např. překročen limit), systém navrhne:
    *   **Smazání** kódu.
    *   **Nahrazení** kódu (např. nahradit výkon s překročeným limitem za jiný, méně frekventovaný, pokud je to možné).

---

## 3. Logika User Story 2: Automatické kódování klinické události

Tato funkce využívá **Large Language Model (LLM)** pro analýzu textu a transformaci na kódy.

### A. Zpracování vstupu (NLP/LLM)

1.  **Extrakce entit:** LLM analyzuje `clinical_event_text` a extrahuje klíčové medicínské entity:
    *   **Symptomy** (např. "akutní bolest břicha", "palpační bolestivost").
    *   **Anatomické lokace** (např. "pravé podžebří").
    *   **Podezření/Závěry** (např. "Podezření na cholecystitidu").
    *   **Kontext pacienta** (věk, pohlaví, komorbidity z `patient_context`).

2.  **Mapování na MKN-10:** LLM na základě extrahovaných entit a kontextu navrhne nejpravděpodobnější **kód diagnózy (MKN-10)**.
    *   *Klíčový požadavek:* LLM musí být trénován na českých medicínských datech a MKN-10.

### B. Doporučování výkonů (LLM a Pravidla)

1.  **Návrh výkonů:** LLM na základě navržené diagnózy a popisu události navrhne relevantní **kódy výkonů (SZV)**.
    *   *Příklad:* Pro diagnózu "Akutní cholecystitida" (K810) navrhne výkony: "Základní vyšetření" (01021) a "Ultrasonografie břicha" (09119).

2.  **Validace doporučení:** Navržené kódy výkonů jsou následně ověřeny proti `Ciselnik_Vykonu` a `Pravidla_Vykazovani` (Tabulky 1 a 3) pro zajištění, že:
    *   Výkon je platný pro danou odbornost.
    *   Výkon je kombinovatelný s navrženou diagnózou.

3.  **Generování odůvodnění:** LLM vygeneruje **snippety** z klinického textu, které odůvodňují přiřazení každého kódu (jak bylo požadováno v zadání).

Tato logika zajišťuje, že výstup je nejen relevantní (díky LLM), ale i **vykazatelně správný** (díky ověření proti číselníkům a pravidlům).
