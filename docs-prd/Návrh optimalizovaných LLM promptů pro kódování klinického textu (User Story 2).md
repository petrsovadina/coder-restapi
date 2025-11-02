# Návrh optimalizovaných LLM promptů pro kódování klinického textu (User Story 2)

Tato fáze specifikuje, jak bude **LLM Engine** (doporučený Google Gemini) analyzovat text klinické události a generovat strukturované výstupy s kódy MKN-10 a SZV. Klíčem je použití **System Promptu** pro nastavení kontextu a **JSON módu** pro spolehlivý a validovatelný výstup.

## 1. Požadovaný výstupní formát (JSON Schema)

LLM bude instruován, aby generoval výstup přesně podle následujícího JSON schématu.

```json
{
  "type": "object",
  "properties": {
    "main_diagnosis": {
      "type": "object",
      "description": "Hlavní diagnóza (MKN-10) pro danou klinickou událost.",
      "properties": {
        "code": {"type": "string", "description": "Kód MKN-10 (např. 'I10')."},
        "reason_snippet_cz": {"type": "string", "description": "Přesný textový úryvek z klinické události, který odůvodňuje tuto diagnózu."}
      },
      "required": ["code", "reason_snippet_cz"]
    },
    "secondary_diagnoses": {
      "type": "array",
      "description": "Seznam sekundárních diagnóz (MKN-10).",
      "items": {
        "type": "object",
        "properties": {
          "code": {"type": "string"},
          "reason_snippet_cz": {"type": "string"}
        },
        "required": ["code", "reason_snippet_cz"]
      }
    },
    "performance_recommendations": {
      "type": "array",
      "description": "Seznam doporučených kódů výkonů (SZV) na základě klinické události.",
      "items": {
        "type": "object",
        "properties": {
          "code": {"type": "string", "description": "Kód výkonu SZV (např. '01021')."},
          "reason_cz": {"type": "string", "description": "Stručné odůvodnění, proč byl tento výkon navržen (např. 'Standardní vyšetření')."},
          "diagnosis_link": {"type": "string", "description": "Kód MKN-10, ke kterému se výkon primárně váže (např. 'I10')."}
        },
        "required": ["code", "reason_cz", "diagnosis_link"]
      }
    }
  },
  "required": ["main_diagnosis", "performance_recommendations"]
}
```

## 2. Návrh System Promptu (Kontext a Pravidla)

System Prompt je klíčový pro nastavení role LLM a vložení medicínského kontextu (z dodaných PDF).

```markdown
Jsi vysoce kvalifikovaný medicínský kodér a expert na české zdravotnické vykazování. Tvá role je analyzovat klinický text v češtině a navrhnout nejpřesnější kódy diagnóz (MKN-10) a výkonů (SZV) pro vykazování zdravotní pojišťovně.

### Pravidla pro kódování (MKN-10)
1.  **Hlavní diagnóza:** Musí být stav, který je primárně zodpovědný za potřebu péče. Použij pravidla z MKN-10 Instrukční příručky.
2.  **Kódování:** Používej pouze platné kódy MKN-10 (např. I10, K30, R10.4).
3.  **Odůvodnění:** Pro každou diagnózu MUSÍŠ extrahovat PŘESNÝ textový úryvek (snippet) z klinické události, který kód odůvodňuje.

### Pravidla pro doporučení výkonů (SZV)
1.  **Relevance:** Navrhni pouze výkony, které jsou logicky a medicínsky relevantní k navrženým diagnózám a popisu události.
2.  **Standardní péče:** Vždy zvaž standardní výkony (např. základní vyšetření, kontrolní vyšetření) v kontextu ambulantní péče.
3.  **Vazba:** Pro každý výkon uveď kód MKN-10, ke kterému se primárně váže.

### Formát výstupu
Tvůj výstup MUSÍ být striktně ve formátu JSON, který je definován v přiloženém schématu. NEZAHRNUJ žádný vysvětlující text mimo JSON.

### Kontekst číselníků (Zde bude dynamicky vložena část databáze)
[Zde bude vložena relevantní část číselníků MKN-10 a SZV, např. 50 nejčastějších kódů a jejich názvy, a pravidla pro kódování z MKN-10 Tabelární části.]
```

## 3. Návrh User Promptu (Vstup)

User Prompt bude obsahovat text klinické události a volitelný kontext pacienta.

```markdown
Analyzuj následující klinickou událost a vygeneruj JSON výstup s doporučenými kódy.

### Klinická událost (Text)
Pacient, 45 let, muž, přichází s akutní, silnou bolestí břicha v pravém podžebří, která trvá 2 dny a vystřeluje do zad. Objektivně palpační bolestivost v PPH, mírná febrilie (37.8°C). Laboratorní výsledky ukazují zvýšené CRP. Podezření na akutní cholecystitidu.

### Kontext pacienta
- Věk: 45
- Pohlaví: Muž
- Komorbidity: Esenciální hypertenze (I10)
```

## 4. Příklad očekávaného výstupu (JSON)

```json
{
  "main_diagnosis": {
    "code": "K810",
    "reason_snippet_cz": "akutní, silnou bolestí břicha v pravém podžebří, která trvá 2 dny. Objektivně palpační bolestivost v PPH. Podezření na akutní cholecystitidu."
  },
  "secondary_diagnoses": [
    {
      "code": "I10",
      "reason_snippet_cz": "Komorbidity: Esenciální hypertenze (I10)"
    }
  ],
  "performance_recommendations": [
    {
      "code": "01021",
      "reason_cz": "Základní vyšetření praktickým lékařem pro akutní stav.",
      "diagnosis_link": "K810"
    },
    {
      "code": "09119",
      "reason_cz": "Cílené ultrasonografické vyšetření břicha pro potvrzení diagnózy cholecystitidy.",
      "diagnosis_link": "K810"
    },
    {
      "code": "81001",
      "reason_cz": "Základní laboratorní vyšetření (CRP) pro zánětlivý proces.",
      "diagnosis_link": "K810"
    }
  ]
}
```

Tento návrh zajišťuje, že LLM bude pracovat s jasnými pravidly, generovat spolehlivý JSON výstup a poskytovat transparentní odůvodnění, což je klíčové pro důvěru uživatelů a následnou validaci.
