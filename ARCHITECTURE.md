# 🏗️ Návrh architektury projektu

**Verze:** 1.0.0  
**Datum:** 2024-01-XX  
**Status:** Navrženo

---

## 📁 Optimální struktura projektu

```
coder-restapi/
├── .github/                          # CI/CD workflows
│   └── workflows/
│       ├── ci.yml                    # pytest + linting
│       └── deploy.yml                # K8s deployment
│
├── .specify/                         # SpecKit governance
│   ├── memory/
│   │   └── constitution.md           # v1.1.0 (8 principů)
│   └── templates/
│       └── plan-template.md          # Template pro features
│
├── docs/                             # Dokumentace
│   ├── PRD.md                        # Product requirements
│   ├── docs-prd/                     # Technické specifikace
│   ├── legislativni-docs/            # MKN-10, VZP materiály
│   └── example/                      # Vzorové K-dávky
│
├── src/                              # Zdrojové kódy aplikace
│   └── coder_api/                    # Main Python package
│       ├── __init__.py
│       ├── main.py                   # FastAPI aplikace
│       ├── config.py                 # Settings (Pydantic BaseSettings)
│       │
│       ├── api/                      # API layer (routers)
│       │   ├── __init__.py
│       │   ├── deps.py               # Dependencies (DB, auth, cache)
│       │   └── v1/                   # API v1 endpoints
│       │       ├── __init__.py
│       │       ├── kdavka.py         # POST /v1/validate-kdavka
│       │       └── coding.py         # POST /v1/code-event
│       │
│       ├── services/                 # Business logic layer
│       │   ├── __init__.py
│       │   ├── dasta_parser.py       # DASTA fixed-width parser
│       │   ├── validation_engine.py  # Validation rules + codebooks
│       │   ├── llm_service.py        # Gemini API integration
│       │   └── cache_service.py      # Redis wrapper
│       │
│       ├── models/                   # Data models
│       │   ├── __init__.py
│       │   ├── domain.py             # Domain models (dataclasses)
│       │   ├── schemas.py            # Pydantic schemas (API I/O)
│       │   └── database.py           # SQLAlchemy models
│       │
│       ├── repositories/             # Data access layer
│       │   ├── __init__.py
│       │   ├── codebook_repo.py      # MKN-10, SZV queries
│       │   └── audit_repo.py         # Audit log persistence
│       │
│       ├── core/                     # Core infrastructure
│       │   ├── __init__.py
│       │   ├── database.py           # DB session + engine
│       │   ├── cache.py              # Redis connection
│       │   ├── security.py           # Auth, RBAC, JWT
│       │   ├── logging.py            # Structured logging
│       │   └── exceptions.py         # Custom exceptions
│       │
│       └── utils/                    # Shared utilities
│           ├── __init__.py
│           ├── validators.py         # Input validation helpers
│           └── formatters.py         # Output formatting
│
├── tests/                            # Test suite (≥80% coverage)
│   ├── __init__.py
│   ├── conftest.py                   # Pytest fixtures
│   ├── unit/                         # Unit tests
│   │   ├── test_dasta_parser.py
│   │   ├── test_validation_engine.py
│   │   └── test_llm_service.py
│   ├── integration/                  # Integration tests
│   │   ├── test_api_kdavka.py
│   │   └── test_api_coding.py
│   └── e2e/                          # End-to-end tests
│       └── test_full_workflow.py
│
├── migrations/                       # Alembic DB migrations
│   ├── versions/
│   │   └── 001_initial_schema.py     # 4 tabulky (MKN-10, SZV, rules, audit)
│   └── alembic.ini
│
├── scripts/                          # Utility scripts
│   ├── load_codebooks.py             # Import MKN-10, SZV do DB
│   └── generate_test_data.py        # Mock K-dávky pro testy
│
├── deploy/                           # Deployment configs
│   ├── docker/
│   │   ├── Dockerfile                # Multi-stage build
│   │   └── docker-compose.yml        # Local dev stack
│   └── k8s/                          # Kubernetes manifests
│       ├── api-deployment.yaml
│       ├── postgres-statefulset.yaml
│       └── redis-deployment.yaml
│
├── monitoring/                       # Observability
│   ├── prometheus.yml                # Metrics scraping
│   └── grafana-dashboards/
│       └── api-performance.json
│
├── .env.example                      # Environment variables template
├── .gitignore                        # Python + IDE ignores
├── pyproject.toml                    # Project metadata + deps (Poetry/setuptools)
├── requirements.txt                  # Production dependencies
├── requirements-dev.txt              # Dev dependencies (pytest, black, ruff)
├── README.md                         # Project overview
├── ARCHITECTURE.md                   # Tento soubor
└── CHANGELOG.md                      # Version history
```

---

## 🔧 Klíčové komponenty

### 1. API Gateway (`src/coder_api/api/`)
**Zodpovědnost:**
- REST API endpoints (OpenAPI 3.1)
- Request validation (Pydantic)
- Authentication & authorization
- Rate limiting
- API versioning (v1, v2)

**Technologie:**
- FastAPI (async)
- OAuth2 + JWT
- Pydantic v2

### 2. DASTA Parser (`src/coder_api/services/dasta_parser.py`)
**Zodpovědnost:**
- Parsing fixed-width DASTA formátu
- Extrakce hlaviček a záznamů
- Validace struktury

**Logika:**
```python
class DASTAParser:
    def parse_kdavka(self, file_content: str) -> KdavkaModel:
        """Parse KDAVKA.XXX file"""
        # Fixed-width column mapping
        # Header: DASTA:0100:KDAVKA201:...
        # Records: line-by-line parsing
```

### 3. Validation Engine (`src/coder_api/services/validation_engine.py`)
**Zodpovědnost:**
- Validace podle VZP pravidel
- Kontrola MKN-10 kódů
- Detekce nelogických kombinací
- Generování doporučení

**Logika:**
```python
class ValidationEngine:
    def __init__(self, codebook_repo: CodebookRepository, cache: CacheService):
        self.codebook_repo = codebook_repo
        self.cache = cache
    
    def validate_kdavka(self, kdavka: KdavkaModel) -> ValidationResult:
        """Validate K-dávka records"""
        errors = []
        # 1. Check MKN-10 codes exist (with cache)
        # 2. Validate SZV combinations
        # 3. Check business rules
        return ValidationResult(errors=errors, recommendations=[...])
```

### 4. LLM Service (`src/coder_api/services/llm_service.py`)
**Zodpovědnost:**
- Google Gemini API calls
- Prompt engineering
- Response parsing
- Error handling (rate limits, timeouts)

**Logika:**
```python
class LLMService:
    def __init__(self, gemini_api_key: str):
        self.client = genai.GenerativeModel('gemini-1.5-pro')
    
    async def code_clinical_event(self, text: str) -> CodingResult:
        """Generate MKN-10 codes from clinical text"""
        prompt = self._build_prompt(text)
        response = await self.client.generate_content_async(prompt)
        return self._parse_response(response)
```

### 5. Cache Service (`src/coder_api/services/cache_service.py`)
**Zodpovědnost:**
- Redis wrapper
- TTL management
- Cache invalidation

**Logika:**
```python
class CacheService:
    def __init__(self, redis_client: Redis):
        self.redis = redis_client
    
    async def get_mkn10_code(self, code: str) -> Optional[CodebookEntry]:
        """Get MKN-10 code with caching (TTL: 24h)"""
        key = f"mkn10:{code}"
        cached = await self.redis.get(key)
        if cached:
            return json.loads(cached)
        # Fallback to DB...
```

---

## 🎯 Architekturní principy

### Princip 1: Modular Architecture (Constitution VIII)
- **Jasné hranice modulů:** API ← Services ← Repositories ← DB
- **Dependency Injection:** FastAPI `Depends()` pro všechny závislosti
- **Loose coupling:** Services nekomunikují přímo, jen přes interfaces

### Princip 2: API-First (Constitution II)
- **OpenAPI 3.1 spec:** Autogenerovaná z FastAPI dekorátorů
- **Contract testing:** Validace responses podle schémat
- **Versioning:** `/v1/`, `/v2/` prefixes

### Princip 3: TDD Red-Green-Refactor (Constitution III)
- **Red:** Napsat failing test
- **Green:** Minimální kód pro pass
- **Refactor:** Clean up s coverage check

### Princip 4: Security by Design (Constitution IV)
```python
# Všechny endpoints s autentizací
@router.post("/v1/validate-kdavka", dependencies=[Depends(get_current_user)])
async def validate_kdavka(
    file: UploadFile,
    current_user: User = Depends(get_current_user)
):
    # Anonymize PHI before processing
    ...
```

---

## 📊 Data flow

### US1: Validace K-dávky
```
POST /v1/validate-kdavka
  ↓
[API Router] → Auth check → Rate limit
  ↓
[DASTA Parser] → Parse fixed-width → KdavkaModel
  ↓
[Validation Engine] → Check rules (cache MKN-10) → ValidationResult
  ↓
[Audit Repo] → Log request/response
  ↓
JSON Response (errors + recommendations)
```

### US2: LLM kódování
```
POST /v1/code-event
  ↓
[API Router] → Auth check → Rate limit
  ↓
[LLM Service] → Gemini API (prompt) → CodingResult
  ↓
[Validation Engine] → Verify generated codes
  ↓
[Audit Repo] → Log request/response
  ↓
JSON Response (codes + confidence)
```

---

## 🗄️ Datový model

### PostgreSQL Schema (4 tabulky)

```sql
-- 1. MKN-10 číselník
CREATE TABLE ciselnik_mkn10 (
    id SERIAL PRIMARY KEY,
    kod VARCHAR(10) UNIQUE NOT NULL,
    nazev TEXT NOT NULL,
    kategorie VARCHAR(50),
    platnost_od DATE,
    platnost_do DATE,
    metadata JSONB  -- Rozšiřující data
);
CREATE INDEX idx_mkn10_kod ON ciselnik_mkn10(kod);

-- 2. SZV číselník
CREATE TABLE ciselnik_szv (
    id SERIAL PRIMARY KEY,
    kod VARCHAR(10) UNIQUE NOT NULL,
    nazev TEXT NOT NULL,
    popis TEXT,
    metadata JSONB
);

-- 3. Validační pravidla
CREATE TABLE validation_rules (
    id SERIAL PRIMARY KEY,
    rule_type VARCHAR(50) NOT NULL,  -- 'mkn10_combination', 'szv_logic'
    rule_definition JSONB NOT NULL,  -- Flexibilní pravidla
    error_message TEXT NOT NULL,
    severity VARCHAR(20) DEFAULT 'error'  -- 'error', 'warning'
);

-- 4. Audit log
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    endpoint VARCHAR(100) NOT NULL,
    user_id VARCHAR(100),
    request_payload JSONB,
    response_payload JSONB,
    status_code INT,
    processing_time_ms INT,
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

---

## 🚀 Deployment architektura

### Kubernetes Stack

```yaml
# api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coder-api
spec:
  replicas: 3  # Horizontal scaling
  template:
    spec:
      containers:
      - name: api
        image: coder-api:v1.0.0
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: REDIS_URL
          value: redis://redis-service:6379
        - name: GEMINI_API_KEY
          valueFrom:
            secretKeyRef:
              name: gemini-credentials
              key: api-key
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

### Docker Multi-stage Build

```dockerfile
# Dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY pyproject.toml requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY src/ ./src/
EXPOSE 8000
CMD ["uvicorn", "coder_api.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 🔍 Monitoring & Observability

### Prometheus Metrics
```python
# src/coder_api/core/metrics.py
from prometheus_client import Counter, Histogram

request_count = Counter('api_requests_total', 'Total requests', ['endpoint', 'status'])
request_duration = Histogram('api_request_duration_seconds', 'Request duration', ['endpoint'])
validation_errors = Counter('validation_errors_total', 'Validation errors', ['error_type'])
```

### Grafana Dashboard Panels
1. **API Performance:** Request duration (p50, p95, p99)
2. **Error Rate:** 4xx/5xx responses over time
3. **Cache Hit Ratio:** Redis hits vs misses
4. **LLM Latency:** Gemini API response times
5. **DB Connection Pool:** Active/idle connections

---

## ✅ Constitution Alignment

| Princip | Implementace |
|---------|-------------|
| **I. Legislative Compliance** | MKN-10 číselník, VZP pravidla v DB, GDPR anonymizace |
| **II. API-First** | OpenAPI 3.1, Pydantic schemas, contract tests |
| **III. TDD** | Pytest coverage ≥80%, fixtures v `conftest.py` |
| **IV. Security** | OAuth2 + JWT, TLS 1.3+, secret management (K8s) |
| **V. Transparency** | Audit logs, structured logging (JSON), OpenAPI docs |
| **VI. Performance** | Redis cache (<50ms), async FastAPI, DB indexes |
| **VII. Versioning** | `/v1/` prefix, SemVer, CHANGELOG.md |
| **VIII. Modular** | Services layer, DI, clear boundaries |

---

## 📝 Další kroky

1. **Implementační roadmap** (Task 4)
2. **Proof of Concept:** DASTA Parser + 1 validační pravidlo
3. **CI/CD setup:** GitHub Actions workflow
4. **Load testing:** Locust tests pro <200ms cíl
5. **Dokumentace API:** Swagger UI + ReDoc

**Schváleno:** ❓  
**Připraveno k implementaci:** ✅
