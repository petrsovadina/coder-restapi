# Analýza potenciálního trhu (TAM, SAM, SOM) a nasycení trhu v ČR

## 1. Základní data a odhady trhu

Analýza trhu pro navrhované REST API se zaměřuje na segment **ambulantních poskytovatelů zdravotní péče (PZS)** v České republice, kteří jsou primárními uživateli K-dávek a vykazují péči zdravotním pojišťovnám.

### 1.1. Celkový dostupný trh (Total Addressable Market - TAM)

TAM představuje maximální teoretický trh, který by mohl být obsloužen.

| Segment PZS | Odhadovaný počet (PZS/Ambulance) | Zdroj / Poznámka |
| :--- | :--- | :--- |
| **Ambulance specialistů** | 7 127 | Dle dat LK ČR [1] |
| **Ambulance ženských lékařů** | 1 129 | Dle dat LK ČR [1] |
| **Ambulance zubních lékařů** | 5 629 | Dle dat LK ČR [1] |
| **Ambulance praktických lékařů (PL)** | ~5 000 | Odhad (včetně PL pro děti a dorost) |
| **Celkový TAM** | **~19 000** | Celkový počet ambulantních PZS v ČR |

**Závěr TAM:** Celkový trh je robustní a stabilní, s přibližně **19 000** potenciálními zákazníky.

### 1.2. Obsluhovatelný dostupný trh (Serviceable Available Market - SAM)

SAM představuje část TAM, kterou je možné realisticky obsloužit s ohledem na geografii, legislativu a technologickou připravenost. V kontextu ČR a moderního API řešení je klíčová ochota PZS integrovat externí nástroje.

*   **Předpoklad:** 20-30% PZS bude otevřeno integraci moderního API řešení, které nabízí přímou úsporu nákladů/zvýšení úhrad.
*   **Odhad SAM:** 19 000 PZS * 25% = **4 750 PZS**

**Závěr SAM:** Trh s potenciálem **4 750** PZS je dostatečně velký pro vybudování ziskového podniku.

### 1.3. Realisticky dosažitelný trh (Serviceable Obtainable Market - SOM)

SOM představuje realistický cíl pro první 3 roky provozu. Zaměříme se na segmenty s největší potřebou a největší ochotou k inovacím.

*   **Cílové segmenty:**
    *   **Ambulantní specialisté** (vysoká frekvence výkonů, komplexní pravidla).
    *   **Nové ambulance** (hledají moderní a efektivní řešení).
    *   **PZS s vysokou chybovostí** (časté zamítnutí dávek pojišťovnami).
*   **Předpoklad:** 5-10% TAM.
*   **Odhad SOM:** 19 000 PZS * 8% = **1 520 PZS**

**Závěr SOM:** Realistický cíl je získat **1 500 - 2 000** PZS v prvních letech.

## 2. Analýza nasycení trhu a potenciální úspěšnosti

### 2.1. Nasycení trhu (User Story 1: Validace K-dávky)

Trh je **středně nasycen** základními funkcemi.

*   **Stávající řešení:** Většina AIS (DUNA MEDIK, SmartMEDIX atd.) má vestavěné moduly pro tvorbu a základní validaci K-dávek.
*   **Příležitost:** Stávající validace je často nedostatečná. Chybí dynamická, prediktivní kontrola proti **všem** aktuálním úhradovým vyhláškám a metodikám pojišťoven. API řešení se stane **"druhou linií obrany"** – komplexním pre-validátorem, který zachytí chyby, které AIS přehlédne.
*   **Úspěšnost:** Vysoká, pokud se prokáže, že API dokáže snížit počet zamítnutých dávek a zvýšit úhrady.

### 2.2. Nasycení trhu (User Story 2: Kódování klinické události)

Trh je **minimálně nasycen** (téměř nenasycen).

*   **Stávající řešení:** Kódování volného textu (anamnéza, nález) je v ČR stále doménou lékaře nebo kodéra.
*   **Příležitost:** Využití LLM pro transformaci textu na kódy MKN-10 a SZV je **inovativní a disruptivní**. Nabízí obrovskou úsporu času a snížení chybovosti.
*   **Úspěšnost:** Extrémně vysoká. Tato funkce představuje **hlavní konkurenční výhodu** a důvod, proč by se PZS měly pro API rozhodnout.

## 3. Hodnotová nabídka a potenciální úspěšnost

Úspěšnost projektu bude záviset na jasné komunikaci **návratnosti investice (ROI)** pro PZS.

| Hodnotová nabídka | Přínos pro PZS | Vliv na úspěšnost |
| :--- | :--- | :--- |
| **Snížení chybovosti** | Minimalizace zamítnutých dávek a nutnosti oprav. | **Klíčový faktor.** Přímá úspora administrativního času. |
| **Maximalizace úhrad** | Doporučení pro doplnění kódů (např. preventivní kódy, které lékař zapomněl vykázat). | **Klíčový faktor.** Přímé zvýšení příjmů. |
| **Úspora času (LLM)** | Automatické kódování klinického textu. | **Diferenciátor.** Umožní lékařům soustředit se na pacienta. |
| **Moderní API** | Snadná integrace s jakýmkoliv IS nebo webovou aplikací. | **Technologická výhoda.** Připravenost na budoucnost. |

**Celková potenciální úspěšnost:** **Velmi vysoká**. Projekt řeší reálný a bolestivý problém (komplexnost vykazování) a nabízí inovativní řešení (LLM kódování), které na trhu chybí.

---
**Reference:**
[1] Komora bojuje za všechny ambulantní specialisty. LK ČR.
[2] Nová verze programu 3.0.9.1. AISCZ.
[3] Zvýšení dostupnosti onkologické a další zdravotní péče... ÚZIS.
