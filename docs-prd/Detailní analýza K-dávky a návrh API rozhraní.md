# Detailní analýza K-dávky a návrh API rozhraní

## 1. Detailní analýza ukázkové K-dávky (DASTA)

Ukázkový soubor `/home/ubuntu/upload/KDAVKA201-Výsledkyhledání.201` je textový soubor ve formátu **DASTA** (Datový Standard Ministerstva Zdravotnictví ČR), konkrétně **K-dávka** pro vykazování zdravotní péče pojišťovnám. Jedná se o formát s pevnou šířkou polí, kde každá řádka začíná identifikátorem bloku.

Pro účely validace a kódování jsou klíčové následující bloky:

### Blok A (Ambulantní péče) - Hlava ošetření/návštěvy

Blok `A` definuje jednu ambulantní návštěvu nebo ošetření.

| Řádek | Příklad | Popis pole (Dedukce) |
| :--- | :--- | :--- |
| 7 | `A  3055900  1201181101001      0010002014232K30                 0                0.00     85` | |
| **Pozice** | **Hodnota** | **Význam** |
| 1 | `A` | Identifikátor bloku (Ambulantní péče) |
| 3-9 | `3055900` | Interní číslo dokladu/návštěvy |
| 12-13 | `1` | Pořadové číslo návštěvy v dávce |
| 14-23 | `201181101001` | Kód poskytovatele (PZS) a IČP (Identifikační číslo pracoviště) |
| 27-29 | `001` | Kód odbornosti (např. 001 - Všeobecné praktické lékařství) |
| 30-43 | `0002014232` | Rodné číslo pacienta (bez lomítka) |
| 44-47 | `K30` | **Hlavní diagnóza** (MKN-10) pro danou návštěvu |
| 81-84 | `85` | **Kód výkonu** (pravděpodobně kód základního vyšetření, např. 01021 - Kontrolní vyšetření) |

### Blok V (Výkon) - Detail vykázaného výkonu

Blok `V` specifikuje jednotlivé vykázané výkony v rámci bloku `A`.

| Řádek | Příklad | Popis pole (Dedukce) |
| :--- | :--- | :--- |
| 8 | `V01092025015431   R104      85` | |
| **Pozice** | **Hodnota** | **Význam** |
| 1 | `V` | Identifikátor bloku (Výkon) |
| 2-9 | `01092025` | **Datum výkonu** (DDMMRRRR) |
| 10-16 | `015431` | **Kód výkonu** (Seznam zdravotních výkonů) |
| 20-23 | `R104` | **Diagnóza** (MKN-10) vázaná k výkonu |
| 32-35 | `85` | **Počet bodů** (nebo cena/počet jednotek) |

### Blok G (Globální diagnóza)

Blok `G` slouží k přiřazení globální diagnózy k bloku `A`.

| Řádek | Příklad | Popis pole (Dedukce) |
| :--- | :--- | :--- |
| 9 | `GR104` | |
| **Pozice** | **Hodnota** | **Význam** |
| 1 | `G` | Identifikátor bloku (Globální diagnóza) |
| 2-5 | `R104` | **Kód diagnózy** (MKN-10) |

---

## 2. Návrh REST API rozhraní

Navrhovaná služba bude implementována jako **REST API** s využitím formátu **JSON** pro komunikaci.

### A. User Story 1: Validace a doporučení kódů z K-dávky

Tato funkce bude přijímat K-dávku a provádět její validaci a analýzu.

#### Endpoint: `/api/v1/coding/validate-kdavka`

| Parametr | Typ | Popis | Příklad |
| :--- | :--- | :--- | :--- |
| **Metoda** | `POST` | | |
| **Vstup (Body)** | `multipart/form-data` | Soubor K-dávky v textovém formátu DASTA. | `file: KDAVKA201.201` |
| **Content-Type** | `text/plain` | Alternativně by bylo možné přijímat i JSON strukturu odvozenou z DASTA. | |

#### Výstup (Response Body - JSON)

```json
{
  "status": "OK",
  "validation_summary": {
    "total_visits": 20,
    "total_errors": 3,
    "total_warnings": 5,
    "total_points_value": 902.50
  },
  "visits": [
    {
      "visit_id": "3055900",
      "patient_id": "0002014232",
      "main_diagnosis": "K30",
      "status": "WARNING",
      "messages": [
        {
          "type": "WARNING",
          "code": "DIAG_PERF_MISMATCH",
          "description_cz": "Hlavní diagnóza K30 (Dyspepsie) je neobvyklá pro vykázaný výkon 015431 (EKG). Doporučeno zkontrolovat.",
          "field": "A.main_diagnosis",
          "value": "K30"
        }
      ],
      "performances": [
        {
          "performance_code": "015431",
          "date": "2025-09-01",
          "diagnosis_code": "R104",
          "points": 85,
          "status": "ERROR",
          "messages": [
            {
              "type": "ERROR",
              "code": "FREQ_LIMIT_EXCEEDED",
              "description_cz": "Překročen frekvenční limit výkonu 015431 (EKG) pro dané období. Maximálně 1x za 3 měsíce. Doporučeno smazat nebo nahradit.",
              "field": "V.performance_code",
              "value": "015431"
            }
          ],
          "recommendations": [
            {
              "code": "01021",
              "description_cz": "Základní vyšetření praktickým lékařem",
              "reason_cz": "Standardní výkon pro ambulantní návštěvu."
            }
          ]
        }
      ]
    }
    // ... další návštěvy
  ]
}
```

### B. User Story 2: Automatické kódování klinické události

Tato funkce bude analyzovat textový popis klinické události a navrhovat relevantní kódy.

#### Endpoint: `/api/v1/coding/code-clinical-event`

| Parametr | Typ | Popis | Příklad |
| :--- | :--- | :--- | :--- |
| **Metoda** | `POST` | | |
| **Vstup (Body)** | `application/json` | | |
| `clinical_event_text` | `string` | Textový popis klinické události (anamnéza, objektivní nález, závěr). | `"Pacient přichází s akutní bolestí břicha v pravém podžebří, trvající 2 dny. Objektivně palpační bolestivost v PPH. Podezření na cholecystitidu."` |
| `patient_context` | `object` | Volitelný kontext pacienta (věk, pohlaví, komorbidity). | `{"age": 45, "gender": "M", "comorbidities": ["I10"]}` |

#### Výstup (Response Body - JSON)

```json
{
  "status": "OK",
  "main_diagnosis_recommendation": {
    "code": "K810",
    "description_cz": "Akutní cholecystitida",
    "reason_snippet_cz": "akutní bolestí břicha v pravém podžebří, palpační bolestivost v PPH",
    "confidence_score": 0.95
  },
  "performance_recommendations": [
    {
      "code": "01021",
      "description_cz": "Základní vyšetření praktickým lékařem",
      "reason_cz": "Standardní vyšetření pacienta s akutními potížemi.",
      "diagnosis_link": "K810"
    },
    {
      "code": "09119",
      "description_cz": "Ultrasonografie břicha - cílené vyšetření",
      "reason_cz": "Podezření na cholecystitidu vyžaduje cílené ultrazvukové vyšetření.",
      "diagnosis_link": "K810"
    }
  ],
  "coding_notes": "Doporučeno doplnit laboratorní vyšetření (např. CRP, jaterní testy) a zvážit odeslání na chirurgii."
}
```

## 3. Požadavky na datové zdroje a logiku

Pro implementaci obou User Stories je nezbytné mít přístup k následujícím datovým zdrojům a pravidlům:

1.  **Číselník výkonů (Seznam zdravotních výkonů s bodovými hodnotami):** Kódy výkonů, jejich názvy, bodové hodnoty, odbornosti, frekvenční limity (např. 1x za 3 měsíce), a pravidla pro kombinaci s diagnózami.
2.  **Číselník diagnóz (MKN-10):** Kódy diagnóz a jejich názvy.
3.  **Úhradová pravidla/Metodiky:** Pravidla pro správné vykazování (např. kombinace výkonů, časové limity, vazba výkon-diagnóza).

Tato data budou tvořit základ **validace logiky** a **tréninkových dat** pro LLM v User Story 2.

## 4. Architektura a implementace (Fáze 4)

V další fázi bude navržena architektura, která bude zahrnovat:
*   **API Gateway/Web Server** (např. FastAPI/Flask) pro obsluhu REST endpointů.
*   **Parser DASTA** pro transformaci textové K-dávky na strukturovaný JSON.
*   **Validátor/Business Logic Engine** pro User Story 1 (implementace pravidel z bodu 3).
*   **LLM/AI Engine** pro User Story 2 (analýza textu a návrh kódů).
*   **Databáze číselníků** (např. PostgreSQL) pro uložení dat z bodu 3.

Nyní přecházím k Fázi 3: Návrh logiky validace a doporučování kódů a definice datového modelu.
