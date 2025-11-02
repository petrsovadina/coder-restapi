# Návrh architektury REST API služby a plán implementace

## 1. Architektura systému

Navrhovaná architektura je modulární a škálovatelná, s následujícími klíčovými komponentami:

### Diagram architektury

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Klientská aplikace                              │
│                  (Ambulantní IS, Web frontend, CLI)                     │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │ HTTP/REST
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         API Gateway / Web Server                         │
│                    (FastAPI, Flask, nebo Django REST)                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ Endpoints:                                                       │   │
│  │ - POST /api/v1/coding/validate-kdavka                          │   │
│  │ - POST /api/v1/coding/code-clinical-event                      │   │
│  │ - GET  /api/v1/coding/health                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                ┌────────────────┼────────────────┐
                ▼                ▼                ▼
┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐
│  Parser DASTA        │ │ Validation Engine    │ │ LLM/AI Engine        │
│  (DASTA → JSON)      │ │ (Pravidla, Logika)   │ │ (Analýza textu)      │
└──────────────────────┘ └──────────────────────┘ └──────────────────────┘
                │                ▼                ▼
                │         ┌──────────────────────────────┐
                │         │  Business Logic Layer        │
                │         │  (Validace, Doporučení)      │
                │         └──────────────────────────────┘
                │                 │
                └─────────────────┼─────────────────┐
                                  ▼                 ▼
                    ┌──────────────────────┐ ┌──────────────────────┐
                    │  Databáze Číselníků  │ │  Cache (Redis)       │
                    │  (PostgreSQL)        │ │  (Výkony, Diagnózy)  │
                    │  - Ciselnik_Vykonu   │ │                      │
                    │  - Ciselnik_Diagnoz  │ └──────────────────────┘
                    │  - Pravidla_Vykazovani│
                    └──────────────────────┘
```

## 2. Technologický stack

| Komponenta | Technologie | Popis |
| :--- | :--- | :--- |
| **API Server** | FastAPI (Python) | Moderní, rychlý framework s automatickou dokumentací (Swagger/OpenAPI). |
| **Databáze** | PostgreSQL | Robustní relační databáze pro uložení číselníků a pravidel. |
| **Cache** | Redis | In-memory cache pro zrychlení přístupu k často používaným datům. |
| **LLM** | Google Gemini API / OpenAI API / Cohere API | Cloudové LLM služby pro analýzu textu klinických událostí. |
| **Deployment** | Docker + Kubernetes (nebo Docker Compose pro vývoj) | Kontejnerizace a orchestrace. |
| **Monitoring** | Prometheus + Grafana | Monitoring výkonu a dostupnosti. |
| **Logging** | ELK Stack (Elasticsearch, Logstash, Kibana) | Centralizované logování. |

## 3. Struktura projektu (Python/FastAPI)

```
healthcare-coding-api/
├── app/
│   ├── __init__.py
│   ├── main.py                    # Hlavní aplikace FastAPI
│   ├── config.py                  # Konfigurace (databáze, LLM, atd.)
│   ├── models/
│   │   ├── __init__.py
│   │   ├── database.py            # SQLAlchemy modely (Ciselnik_Vykonu, atd.)
│   │   └── schemas.py             # Pydantic schémata (Request/Response)
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── coding.py              # Endpoints pro /api/v1/coding/*
│   │   └── health.py              # Health check endpoint
│   ├── services/
│   │   ├── __init__.py
│   │   ├── dasta_parser.py        # Parser DASTA formátu
│   │   ├── validation_engine.py   # Logika validace (User Story 1)
│   │   ├── llm_service.py         # Integrace s LLM (User Story 2)
│   │   ├── recommendation_engine.py # Generování doporučení
│   │   └── cache_service.py       # Práce s Redis cache
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── logger.py              # Logging
│   │   └── exceptions.py          # Custom výjimky
│   └── tests/
│       ├── __init__.py
│       ├── test_dasta_parser.py
│       ├── test_validation_engine.py
│       └── test_llm_service.py
├── migrations/
│   └── versions/                  # Alembic migrations
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── requirements.txt               # Python dependencies
├── README.md
└── .env.example                   # Příklad konfigurace
```

## 4. Klíčové moduly a jejich odpovědnosti

### A. DASTA Parser (`dasta_parser.py`)

Modul pro transformaci textové K-dávky do strukturovaného JSON formátu.

**Vstup:** Textový soubor K-dávky (formát DASTA)
**Výstup:** JSON objekt s parsovanými bloky A, V, G

```python
class DASTAParser:
    def parse_kdavka(self, file_content: str) -> Dict:
        # Parsování řádků a extrakce bloků
        # Validace struktury
        # Vrácení strukturovaného JSON
        pass
```

### B. Validation Engine (`validation_engine.py`)

Modul pro implementaci logiky validace (User Story 1).

```python
class ValidationEngine:
    def validate_kdavka(self, parsed_data: Dict) -> ValidationResult:
        # Validace formátu
        # Validace číselníků
        # Validace pravidel
        # Generování doporučení
        pass
```

### C. LLM Service (`llm_service.py`)

Modul pro integraci s LLM (Google Gemini, OpenAI, Cohere).

```python
class LLMService:
    def analyze_clinical_event(self, clinical_text: str, patient_context: Dict) -> Dict:
        # Extrakce entit z textu
        # Návrh diagnózy (MKN-10)
        # Návrh výkonů (SZV)
        pass
```

### D. Recommendation Engine (`recommendation_engine.py`)

Modul pro generování doporučení na základě výsledků validace a analýzy.

```python
class RecommendationEngine:
    def generate_recommendations(self, validation_result: ValidationResult) -> List[Recommendation]:
        # Analýza chyb a varování
        # Návrh řešení (doplnění, nahrazení, smazání kódů)
        pass
```

## 5. Plán implementace (Fáze 5)

Implementace bude probíhat v následujících iteracích:

### Iterace 1: Základní infrastruktura (Týden 1-2)

- [ ] Nastavení FastAPI projektu a struktury
- [ ] Konfigurace PostgreSQL a Redis
- [ ] Implementace základních endpointů (health check)
- [ ] Nastavení Docker a docker-compose

### Iterace 2: DASTA Parser (Týden 3-4)

- [ ] Implementace `DASTAParser` pro transformaci textové K-dávky na JSON
- [ ] Testy parseru na ukázkovém souboru
- [ ] Dokumentace formátu DASTA

### Iterace 3: Databáze a Číselníky (Týden 5-6)

- [ ] Vytvoření SQLAlchemy modelů pro `Ciselnik_Vykonu`, `Ciselnik_Diagnoz`, `Pravidla_Vykazovani`
- [ ] Naplnění databáze údaji z veřejných zdrojů (ÚZIS, VZP)
- [ ] Implementace cache mechanismu (Redis)

### Iterace 4: Validation Engine (Týden 7-9)

- [ ] Implementace `ValidationEngine` s logikou validace formátu
- [ ] Implementace validace číselníků
- [ ] Implementace validace pravidel (frekvenční limity, vazby výkon-diagnóza)
- [ ] Implementace `RecommendationEngine`
- [ ] Testy na ukázkovém souboru K-dávky

### Iterace 5: LLM Integration (Týden 10-12)

- [ ] Integrace s LLM (Google Gemini, OpenAI, nebo Cohere)
- [ ] Implementace `LLMService` pro analýzu klinických textů
- [ ] Trénování/fine-tuning LLM na českých medicínských datech (pokud je to možné)
- [ ] Testy na různých klinických scénářích

### Iterace 6: Testování a Optimalizace (Týden 13-14)

- [ ] Komplexní testování obou User Stories
- [ ] Optimalizace výkonu (cache, databázové indexy)
- [ ] Bezpečnost (autentizace, autorizace, rate limiting)
- [ ] Dokumentace API (Swagger/OpenAPI)

### Iterace 7: Deployment a Monitoring (Týden 15-16)

- [ ] Nastavení CI/CD pipeline (GitHub Actions, GitLab CI)
- [ ] Deployment na produkční prostředí (Kubernetes, Docker Swarm)
- [ ] Nastavení monitoring a alerting (Prometheus, Grafana)
- [ ] Nastavení logging (ELK Stack)

## 6. Požadavky na data

Pro úspěšnou implementaci je nezbytné mít přístup k:

1.  **Číselník výkonů (SZV):** Oficiální seznam zdravotních výkonů s bodovými hodnotami od ÚZIS nebo VZP.
2.  **Číselník diagnóz (MKN-10):** Oficiální MKN-10 klasifikace od WHO/ÚZIS.
3.  **Úhradové vyhlášky a metodiky:** Pravidla pro vykazování od jednotlivých pojišťoven (VZP, ČPZP, OZP, atd.).
4.  **Tréninkové údaje pro LLM:** Anonymizované klinické texty a jejich správné kódování (pro fine-tuning LLM).

## 7. Bezpečnost a Compliance

- **Autentizace:** OAuth 2.0 / JWT tokeny pro přístup k API
- **Autorizace:** Role-based access control (RBAC)
- **Šifrování:** HTTPS/TLS pro komunikaci, šifrování dat v databázi
- **GDPR/HIPAA:** Anonymizace pacientských dat, audit logging
- **Rate limiting:** Ochrana proti DDoS útokům

## 8. Budoucí rozšíření

- **Integrační moduly:** Přímá integrace s populárními ambulantními IS (např. Medea, Helios)
- **Mobilní aplikace:** Native aplikace pro iOS/Android pro lékaře
- **Pokročilá analýza:** Machine Learning modely pro predikci chyb v kódování
- **Multilingual support:** Podpora dalších jazyků (angličtina, slovenština)
- **Blockchain:** Integrace blockchain technologie pro auditovatelnost a transparentnost
