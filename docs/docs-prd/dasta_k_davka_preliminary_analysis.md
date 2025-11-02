# Předběžná analýza struktury K-dávky (DASTA)

Na základě hloubkového výzkumu a analýzy přiloženého souboru `/home/ubuntu/upload/KDAVKA201-Výsledkyhledání.201` (dále jen "K-dávka") lze potvrdit, že se jedná o soubor ve formátu **DASTA** (Datový Standard Ministerstva Zdravotnictví ČR), konkrétně o tzv. **K-dávku** (dávka pro pojišťovny).

Tento formát je textový a řádkově orientovaný, kde každá řádka začíná identifikátorem bloku.

## Identifikované bloky v ukázkové K-dávce:

| Blok | Popis | Příklady řádků |
| :--- | :--- | :--- |
| **DP** | **Dávka Pojišťovně** - Úvodní a závěrečný blok dávky. Obsahuje informace o poskytovateli, pojišťovně, období a celkových součtech. | `DP05811010008100202509 3021 1 272 0.001 05:6.2.47` (Řádek 1) |
| **P** | **Pacient** - Blok s identifikačními údaji pacienta. | `P 3057520181101001 001 0.00 272 1` (Řádek 2) |
| **A** | **Ambulantní péče** - Blok pro vykázání ambulantní péče. Obsahuje údaje o jednotlivých návštěvách/ošetřeních. Klíčový blok pro **User Story 1** (validace kódů). | `A 3055900 1201181101001 0010002014232K30 0 0.00 85` (Řádek 7) |
| **V** | **Výkon** - Blok pro vykázání jednotlivých zdravotních výkonů. Je navázán na blok **A**. Obsahuje datum výkonu, kód výkonu a kód diagnózy. Klíčový blok pro **User Story 1**. | `V01092025015431 R104 85` (Řádek 8) |
| **G** | **Globální diagnóza** - Blok pro vykázání globální diagnózy (hlavní diagnóza pro danou návštěvu/ošetření). Je navázán na blok **A**. | `GR104` (Řádek 9) |
| **Z** | **Závěr dávky** - Závěrečný blok pro kontrolní součty. | `Z 3057100 1381101001 0016755031217 902.50` (Řádek 52) |
| **L** | **Léčivý přípravek** - Blok pro vykázání léčivých přípravků (ZULP). | `L030920251 0215956 1.000 902.50` (Řádek 53) |

## Klíčové informace pro User Story 1 (Validace K-dávky):

**Vstup:** Soubor K-dávky (textový formát DASTA).
**Cíl:** Validace kódů a doporučení.

1.  **Blok A (Ambulantní péče):** Obsahuje informace o pacientovi, poskytovateli, a pravděpodobně i hlavní diagnózu pro danou návštěvu (např. `K30` na řádku 7).
2.  **Blok V (Výkon):** Obsahuje kód výkonu (např. `015431` na řádku 8, i když v ukázce je to spíše kód diagnózy `R104` a kód výkonu je na začátku řádku `V`). **Nutná přesná specifikace DASTA pro blok V.**
3.  **Blok G (Globální diagnóza):** Potvrzuje hlavní diagnózu (např. `GR104` na řádku 9).

**Validace bude probíhat na úrovni:**
*   **Kódu výkonu (z bloku V):** Kontrola, zda je výkon platný pro danou odbornost, zda je správně spárován s diagnózou (MKN-10) a zda nebyl překročen frekvenční limit (např. počet výkonů za období).
*   **Kódu diagnózy (z bloků A, V, G):** Kontrola platnosti kódu MKN-10 a jeho relevance k vykázaným výkonům.

## Klíčové informace pro User Story 2 (Kódování klinické události):

**Vstup:** Klinická událost (textový popis, strukturovaná data?).
**Cíl:** Přiřazení relevantních kódů (výkonů, diagnóz).

Tato část vyžaduje použití **LLM (Large Language Model)** pro analýzu textu klinické události a jeho transformaci na standardizované kódy (MKN-10 pro diagnózy, Seznam zdravotních výkonů pro výkony).

## Další kroky:

1.  **Získat přesnou specifikaci DASTA pro bloky A, V, G** (zejména pro blok V, kde se nachází kód výkonu a diagnózy), abychom mohli přesně parsovat ukázkový soubor.
2.  **Získat přístup k číselníkům** (Seznam zdravotních výkonů, MKN-10) nebo alespoň k informacím o tom, kde je lze získat a jaká jsou základní pravidla pro jejich použití.

Přejdu k dalšímu, cílenějšímu výzkumu DASTA specifikace.
