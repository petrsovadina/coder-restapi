# 🏥 Healthcare Coding Automation API

REST API pro automatizaci kódování a vykazování zdravotní péče pojišťovnám v České republice.

## 📋 O projektu

Tento projekt řeší kritický problém českých zdravotnických zařízení: složitost a chybovost vykazování péče zdravotním pojišťovnám prostřednictvím dávkových souborů (K-dávka). Naše řešení kombinuje:

- **Validaci vykazovacích kódů** (User Story 1) - Automatická kontrola K-dávek před odesláním pojišťovně
- **AI-asistované kódování** (User Story 2) - Generování kódů MKN-10 a SZV z klinických událostí pomocí LLM

## 🎯 Hlavní funkce

### US1: Validace vykazovacích kódů

- Kontrola platnosti kódů dle číselníků MKN-10 a SZV
- Validace kombinací diagnóz a výkonů
- Kontrola frekvenčních omezení
- Podpora formátu KDAVKA.XXX (DASTA) a JSON

### US2: Generování kódů z klinických událostí

- Analýza volného klinického textu pomocí Google Gemini
- Automatické navržení kódů diagnóz (MKN-10)
- Doporučení zdravotních výkonů (SZV)
- Transparentní zdůvodnění s odkazy na text

## 🏗️ Architektura

```text
┌─────────────────────────────────────────┐
│        FastAPI API Gateway               │
│  /api/v1/coding/validate-kdavka         │
│  /api/v1/coding/code-clinical-event     │
└──────────────┬──────────────────────────┘
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
   ┌─────┐ ┌──────┐ ┌──────┐
   │DASTA│ │Valid.│ │ LLM  │
   │Parse│ │Engine│ │Service│
   └─────┘ └──────┘ └──────┘
       │       │        │
       └───────┼────────┘
               ▼
       ┌──────────────┐
       │  PostgreSQL  │
       │  + Redis     │
       └──────────────┘
```

## 🛠️ Technologický stack

- **Backend**: FastAPI (Python 3.11+)
- **Databáze**: PostgreSQL 15+
- **Cache**: Redis 7+
- **LLM Engine**: Google Gemini API
- **Deployment**: Docker + Kubernetes

## 📚 Dokumentace

- [Product Requirements Document (PRD)](./PRD.md)
- [Ústava projektu](./.specify/memory/constitution.md)
- [Technická specifikace](./docs-prd/Kompletní%20technická%20specifikace%20pro%20REST%20API%20pro%20automatizaci%20kódování%20zdravotní%20péče.md)
- [Architektura a implementace](./docs-prd/Návrh%20architektury%20REST%20API%20služby%20a%20plán%20implementace.md)

## 🚀 Rychlý start

```bash
# Klonování repozitáře
git clone <repository-url>
cd coder-restapi

# Vytvoření virtuálního prostředí
python3.11 -m venv venv
source venv/bin/activate

# Instalace závislostí
pip install -r requirements.txt

# Spuštění s Docker Compose
docker-compose up -d

# API dokumentace dostupná na
http://localhost:8000/docs
```

## 🧪 Testování

Projekt dodržuje přísný Test-First Development přístup (TDD):

```bash
# Spuštění všech testů
pytest

# Spuštění s code coverage
pytest --cov=app --cov-report=html

# Spuštění specifické test suite
pytest tests/unit/
pytest tests/integration/
pytest tests/contract/
```

## 📊 Compliance a standardy

- ✅ MKN-10 (česká verze, ÚZIS)
- ✅ Seznam zdravotních výkonů (SZV)
- ✅ DASTA formát (KDAVKA.XXX, VZP)
- ✅ GDPR a zákon č. 372/2011 Sb.
- ✅ OpenAPI 3.0+ specifikace

## 🔐 Bezpečnost

- TLS 1.3+ pro všechny komunikace
- JWT + RBAC autentizace
- Auditní logy (7 let retence)
- Rate limiting per klient
- Žádné PII ukládání bez souhlasu

## 📈 Výkon

- Validace: <200ms p95 latence
- LLM generování: <3s p95
- Dostupnost: 99.5% SLA
- Cache hit rate: >80%

## 👥 Cílová skupina

- **Primární**: Vývojáři ambulantních informačních systémů (AIS)
- **Nepřímí**: Lékaři a zdravotnický personál
- **Sekundární**: Revizní lékaři pojišťoven

## 💰 Monetizační model

Hybridní SaaS + Pay-per-Use:

- Měsíční předplatné: 1 290 - 4 990 Kč
- Transakční poplatky:
  - Validace: 0,90 - 1,80 Kč/návštěva
  - LLM kódování: 8 - 12 Kč/událost

## 📝 Governance

Projekt je řízen [ústavou](./.specify/memory/constitution.md) definující 8 klíčových principů:

1. Soulad s legislativou (Healthcare Compliance)
2. API-First Design
3. Test-First Development (NON-NEGOTIABLE)
4. Ochrana osobních údajů a bezpečnost
5. Transparentnost a auditovatelnost
6. Výkonnostní standardy
7. Verzování a zpětná kompatibilita
8. Modulární architektura

## 📄 Licence

[Bude doplněno]

## 🤝 Kontakt

[Bude doplněno]

---

**Verze projektu**: 0.1.0-alpha  
**Verze ústavy**: 1.1.0  
**Poslední aktualizace**: 2. listopadu 2025
