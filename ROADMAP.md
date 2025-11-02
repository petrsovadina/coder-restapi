# 🗺️ Implementační roadmap

**Projekt:** REST API pro automatizaci kódování zdravotní péče  
**Strategie:** Iterativní vývoj s MVP → Full Product  
**Metodika:** TDD (Red-Green-Refactor) + CI/CD

---

## 📅 Časový harmonogram

| Fáze | Trvání | Období |
|------|--------|--------|
| **0. Příprava prostředí** | 1 týden | W1 |
| **1. Core Infrastructure** | 2 týdny | W2-W3 |
| **2. DASTA Parser MVP** | 2 týdny | W4-W5 |
| **3. Validation Engine** | 3 týdny | W6-W8 |
| **4. LLM Service** | 2 týdny | W9-W10 |
| **5. Integration & Testing** | 2 týdny | W11-W12 |
| **6. Deployment & Launch** | 1 týden | W13 |

**Celkem:** ~13 týdnů (3 měsíce)

---

## 🎯 Fáze 0: Příprava prostředí (W1)

### Cíle
- Nastavit vývojové prostředí
- Připravit CI/CD pipeline
- Vytvořit základní projekt strukturu

### Úkoly

#### 0.1 Project Setup
```bash
# Vytvořit Python package strukturu
mkdir -p src/coder_api/{api,services,models,repositories,core,utils}
touch src/coder_api/__init__.py

# Poetry/pip setup
poetry init
poetry add fastapi uvicorn sqlalchemy psycopg2-binary redis pydantic-settings
poetry add --group dev pytest pytest-cov black ruff mypy
```

#### 0.2 Docker Compose pro dev
```yaml
# deploy/docker/docker-compose.yml
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: coder_api_dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

#### 0.3 GitHub Actions CI
```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest --cov=src --cov-report=xml
      - run: ruff check src/
      - run: mypy src/
```

#### 0.4 Pre-commit Hooks
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
```

**Výstupy:**
- ✅ Funkční `docker-compose up` s PostgreSQL + Redis
- ✅ GitHub Actions pipeline (green)
- ✅ Pre-commit hooks aktivní

**Acceptance Criteria:**
- [ ] `pytest` běží (i když bez testů)
- [ ] `docker-compose up` startuje DB a Redis
- [ ] CI pipeline prochází

---

## 🔧 Fáze 1: Core Infrastructure (W2-W3)

### Cíle
- Nastavit FastAPI aplikaci
- Připravit databázové připojení
- Implementovat autentizaci
- Vytvořit monitoring

### Úkoly

#### 1.1 FastAPI Main App (TDD)
**Test first:**
```python
# tests/unit/test_main.py
def test_app_health_endpoint():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "healthy"}
```

**Implementace:**
```python
# src/coder_api/main.py
from fastapi import FastAPI

app = FastAPI(
    title="Coder API",
    version="1.0.0",
    docs_url="/docs",
    openapi_url="/openapi.json"
)

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

#### 1.2 Config Management
```python
# src/coder_api/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    REDIS_URL: str
    GEMINI_API_KEY: str
    SECRET_KEY: str
    
    class Config:
        env_file = ".env"

settings = Settings()
```

#### 1.3 Database Setup (Alembic)
```bash
# Alembic init
alembic init migrations

# První migrace
alembic revision --autogenerate -m "Initial schema"
```

**SQL migrace:**
```sql
-- migrations/versions/001_initial_schema.py
def upgrade():
    op.create_table(
        'ciselnik_mkn10',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('kod', sa.String(10), unique=True),
        sa.Column('nazev', sa.Text),
        sa.Column('metadata', JSONB)
    )
    # ... další 3 tabulky
```

#### 1.4 Authentication (OAuth2 + JWT)
**Test:**
```python
# tests/unit/test_security.py
def test_jwt_token_generation():
    token = create_access_token({"sub": "user123"})
    payload = decode_token(token)
    assert payload["sub"] == "user123"
```

**Implementace:**
```python
# src/coder_api/core/security.py
from jose import jwt

def create_access_token(data: dict, expires_delta: timedelta = timedelta(hours=1)):
    to_encode = data.copy()
    expire = datetime.utcnow() + expires_delta
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm="HS256")
```

#### 1.5 Monitoring Setup
```python
# src/coder_api/core/metrics.py
from prometheus_client import Counter, Histogram, make_asgi_app

request_count = Counter('api_requests_total', 'Total requests', ['endpoint'])
request_duration = Histogram('api_request_duration_seconds', 'Duration')

# Mount Prometheus endpoint
metrics_app = make_asgi_app()
app.mount("/metrics", metrics_app)
```

**Výstupy:**
- ✅ FastAPI app s `/health`, `/docs`, `/metrics`
- ✅ PostgreSQL connection pool
- ✅ Redis connection
- ✅ JWT autentizace
- ✅ Alembic migrace (4 tabulky)

**Acceptance Criteria:**
- [ ] `curl http://localhost:8000/health` vrací 200
- [ ] `alembic upgrade head` vytvoří 4 tabulky
- [ ] JWT token se správně generuje a validuje
- [ ] `/metrics` endpoint vrací Prometheus data

---

## 🔍 Fáze 2: DASTA Parser MVP (W4-W5)

### Cíle
- Implementovat parser KDAVKA.201 formátu
- Validovat strukturu souboru
- Připravit API endpoint pro upload

### Úkoly

#### 2.1 DASTA Parser (TDD)
**Test:**
```python
# tests/unit/test_dasta_parser.py
def test_parse_kdavka_header():
    content = "DASTA:0100:KDAVKA201:202401:12345678:..."
    parser = DASTAParser()
    result = parser.parse_kdavka(content)
    
    assert result.header.version == "0100"
    assert result.header.format == "KDAVKA201"
    assert result.header.period == "202401"

def test_parse_kdavka_records():
    content = """
    DASTA:0100:KDAVKA201:...
    12345678|A001|20240115|Z001|...
    12345678|A002|20240116|Z002|...
    """
    parser = DASTAParser()
    result = parser.parse_kdavka(content)
    
    assert len(result.records) == 2
    assert result.records[0].patient_id == "12345678"
```

**Implementace:**
```python
# src/coder_api/services/dasta_parser.py
from dataclasses import dataclass
from typing import List

@dataclass
class KdavkaHeader:
    version: str
    format: str
    period: str
    provider_id: str

@dataclass
class KdavkaRecord:
    patient_id: str
    record_type: str
    date: str
    diagnosis_code: str

class DASTAParser:
    def parse_kdavka(self, content: str) -> KdavkaModel:
        lines = content.strip().split('\n')
        header = self._parse_header(lines[0])
        records = [self._parse_record(line) for line in lines[1:]]
        return KdavkaModel(header=header, records=records)
    
    def _parse_header(self, line: str) -> KdavkaHeader:
        parts = line.split(':')
        return KdavkaHeader(
            version=parts[1],
            format=parts[2],
            period=parts[3],
            provider_id=parts[4]
        )
    
    def _parse_record(self, line: str) -> KdavkaRecord:
        # Fixed-width parsing podle DASTA spec
        return KdavkaRecord(
            patient_id=line[0:8],
            record_type=line[9:13],
            date=line[14:22],
            diagnosis_code=line[23:28]
        )
```

#### 2.2 API Endpoint (US1)
**Test:**
```python
# tests/integration/test_api_kdavka.py
def test_validate_kdavka_endpoint(client, auth_headers):
    with open("tests/fixtures/valid_kdavka.201", "rb") as f:
        response = client.post(
            "/v1/validate-kdavka",
            files={"file": f},
            headers=auth_headers
        )
    
    assert response.status_code == 200
    data = response.json()
    assert "errors" in data
    assert "recommendations" in data
```

**Implementace:**
```python
# src/coder_api/api/v1/kdavka.py
from fastapi import APIRouter, UploadFile, Depends

router = APIRouter(prefix="/v1", tags=["kdavka"])

@router.post("/validate-kdavka", dependencies=[Depends(get_current_user)])
async def validate_kdavka(
    file: UploadFile,
    parser: DASTAParser = Depends(get_dasta_parser)
):
    content = await file.read()
    content_str = content.decode("utf-8")
    
    # Parse
    kdavka = parser.parse_kdavka(content_str)
    
    # Placeholder validation (Phase 3)
    return {
        "errors": [],
        "recommendations": [],
        "summary": {
            "total_records": len(kdavka.records),
            "parsed_successfully": True
        }
    }
```

**Výstupy:**
- ✅ DASTA Parser s unit testy (≥80% coverage)
- ✅ API endpoint `POST /v1/validate-kdavka`
- ✅ OpenAPI spec autogenerovaná
- ✅ Fixture data (`tests/fixtures/valid_kdavka.201`)

**Acceptance Criteria:**
- [ ] Parser zpracuje vzorovou K-dávku z `/docs/example/`
- [ ] API vrací 200 pro validní soubor
- [ ] API vrací 400 pro nevalidní formát
- [ ] Pytest coverage ≥ 80%

---

## ✅ Fáze 3: Validation Engine (W6-W8)

### Cíle
- Naimportovat MKN-10 a SZV číselníky do DB
- Implementovat validační pravidla
- Integrovat Redis cache
- Dokončit US1

### Úkoly

#### 3.1 Import Číselníků
**Script:**
```python
# scripts/load_codebooks.py
import pandas as pd
from sqlalchemy import create_engine

def load_mkn10_codebook():
    # Parse PDF nebo Excel z /docs/legislativni-docs/
    df = pd.read_csv("mkn10_export.csv")
    engine = create_engine(settings.DATABASE_URL)
    df.to_sql('ciselnik_mkn10', engine, if_exists='append', index=False)
    print(f"Loaded {len(df)} MKN-10 codes")

if __name__ == "__main__":
    load_mkn10_codebook()
```

#### 3.2 Codebook Repository
**Test:**
```python
# tests/unit/test_codebook_repo.py
def test_get_mkn10_code_exists(db_session):
    repo = CodebookRepository(db_session)
    code = repo.get_mkn10_code("A00.0")
    
    assert code is not None
    assert code.kod == "A00.0"
    assert "cholera" in code.nazev.lower()

def test_get_mkn10_code_not_exists(db_session):
    repo = CodebookRepository(db_session)
    code = repo.get_mkn10_code("INVALID")
    
    assert code is None
```

**Implementace:**
```python
# src/coder_api/repositories/codebook_repo.py
from sqlalchemy.orm import Session
from src.coder_api.models.database import CiselnikMKN10

class CodebookRepository:
    def __init__(self, db: Session):
        self.db = db
    
    def get_mkn10_code(self, kod: str) -> CiselnikMKN10 | None:
        return self.db.query(CiselnikMKN10).filter_by(kod=kod).first()
    
    def get_all_mkn10_codes(self) -> list[CiselnikMKN10]:
        return self.db.query(CiselnikMKN10).all()
```

#### 3.3 Cache Service (Redis)
**Test:**
```python
# tests/unit/test_cache_service.py
@pytest.mark.asyncio
async def test_cache_hit(redis_client):
    cache = CacheService(redis_client)
    
    # Set
    await cache.set_mkn10_code("A00.0", {"nazev": "Cholera"})
    
    # Get (hit)
    result = await cache.get_mkn10_code("A00.0")
    assert result["nazev"] == "Cholera"

@pytest.mark.asyncio
async def test_cache_miss(redis_client):
    cache = CacheService(redis_client)
    result = await cache.get_mkn10_code("NONEXISTENT")
    assert result is None
```

**Implementace:**
```python
# src/coder_api/services/cache_service.py
import json
from redis.asyncio import Redis

class CacheService:
    def __init__(self, redis: Redis):
        self.redis = redis
        self.ttl = 86400  # 24 hours
    
    async def get_mkn10_code(self, kod: str) -> dict | None:
        key = f"mkn10:{kod}"
        cached = await self.redis.get(key)
        return json.loads(cached) if cached else None
    
    async def set_mkn10_code(self, kod: str, data: dict):
        key = f"mkn10:{kod}"
        await self.redis.setex(key, self.ttl, json.dumps(data))
```

#### 3.4 Validation Engine
**Test:**
```python
# tests/unit/test_validation_engine.py
def test_validate_mkn10_code_valid():
    engine = ValidationEngine(codebook_repo, cache)
    result = engine.validate_mkn10_code("A00.0")
    
    assert result.is_valid == True
    assert len(result.errors) == 0

def test_validate_mkn10_code_invalid():
    engine = ValidationEngine(codebook_repo, cache)
    result = engine.validate_mkn10_code("INVALID")
    
    assert result.is_valid == False
    assert "Kód MKN-10 neexistuje" in result.errors[0].message
```

**Implementace:**
```python
# src/coder_api/services/validation_engine.py
from typing import List
from dataclasses import dataclass

@dataclass
class ValidationError:
    code: str
    message: str
    severity: str  # 'error' | 'warning'
    line_number: int

@dataclass
class ValidationResult:
    is_valid: bool
    errors: List[ValidationError]
    recommendations: List[str]

class ValidationEngine:
    def __init__(self, codebook_repo: CodebookRepository, cache: CacheService):
        self.codebook_repo = codebook_repo
        self.cache = cache
    
    async def validate_kdavka(self, kdavka: KdavkaModel) -> ValidationResult:
        errors = []
        
        for idx, record in enumerate(kdavka.records):
            # 1. Validate MKN-10 code exists
            if not await self._validate_mkn10_code(record.diagnosis_code):
                errors.append(ValidationError(
                    code="INVALID_MKN10",
                    message=f"Kód MKN-10 '{record.diagnosis_code}' neexistuje",
                    severity="error",
                    line_number=idx + 2  # Header is line 1
                ))
        
        return ValidationResult(
            is_valid=len(errors) == 0,
            errors=errors,
            recommendations=self._generate_recommendations(errors)
        )
    
    async def _validate_mkn10_code(self, kod: str) -> bool:
        # Try cache first
        cached = await self.cache.get_mkn10_code(kod)
        if cached:
            return True
        
        # Fallback to DB
        code = self.codebook_repo.get_mkn10_code(kod)
        if code:
            await self.cache.set_mkn10_code(kod, code.to_dict())
            return True
        
        return False
```

#### 3.5 Complete US1 Integration
```python
# src/coder_api/api/v1/kdavka.py (update)
@router.post("/validate-kdavka")
async def validate_kdavka(
    file: UploadFile,
    parser: DASTAParser = Depends(get_dasta_parser),
    validator: ValidationEngine = Depends(get_validation_engine)
):
    content = await file.read()
    content_str = content.decode("utf-8")
    
    # Parse
    kdavka = parser.parse_kdavka(content_str)
    
    # Validate
    result = await validator.validate_kdavka(kdavka)
    
    return {
        "is_valid": result.is_valid,
        "errors": [e.__dict__ for e in result.errors],
        "recommendations": result.recommendations,
        "summary": {
            "total_records": len(kdavka.records),
            "error_count": len(result.errors)
        }
    }
```

**Výstupy:**
- ✅ MKN-10 číselník v PostgreSQL (~14,000 záznamů)
- ✅ SZV číselník v PostgreSQL
- ✅ Redis cache pro číselníky
- ✅ Validation Engine s 5+ pravidly
- ✅ US1 plně funkční

**Acceptance Criteria:**
- [ ] `scripts/load_codebooks.py` naimportuje MKN-10
- [ ] Cache hit ratio ≥ 90% (po warm-up)
- [ ] Validace detekuje neplatné kódy
- [ ] Response time <200ms (bez LLM)
- [ ] E2E test pro US1 prochází

---

## 🤖 Fáze 4: LLM Service (W9-W10)

### Cíle
- Integrovat Google Gemini API
- Implementovat prompt engineering
- Dokončit US2
- Performance testing

### Úkoly

#### 4.1 LLM Service
**Test:**
```python
# tests/unit/test_llm_service.py
@pytest.mark.asyncio
async def test_code_clinical_event(mock_gemini_api):
    service = LLMService(api_key="test_key")
    
    text = "Pacient s akutní bolestí v krku, diagnostikována angína"
    result = await service.code_clinical_event(text)
    
    assert len(result.codes) > 0
    assert result.codes[0].code.startswith("J")  # MKN-10 J kategorie
    assert result.codes[0].confidence > 0.7
```

**Implementace:**
```python
# src/coder_api/services/llm_service.py
import google.generativeai as genai
from typing import List

@dataclass
class CodingResult:
    codes: List[dict]  # [{"code": "J03.9", "confidence": 0.85, "reasoning": "..."}]
    raw_response: str

class LLMService:
    def __init__(self, api_key: str):
        genai.configure(api_key=api_key)
        self.model = genai.GenerativeModel('gemini-1.5-pro')
    
    async def code_clinical_event(self, clinical_text: str) -> CodingResult:
        prompt = self._build_prompt(clinical_text)
        
        response = await self.model.generate_content_async(
            prompt,
            generation_config={"temperature": 0.2, "max_output_tokens": 500}
        )
        
        return self._parse_response(response.text)
    
    def _build_prompt(self, text: str) -> str:
        return f"""
        Jsi expert na kódování zdravotních událostí podle MKN-10.
        
        Klinický text: "{text}"
        
        Úkol: Identifikuj hlavní diagnózu a přiřaď MKN-10 kód.
        
        Formát odpovědi (JSON):
        {{
          "codes": [
            {{
              "code": "A00.0",
              "confidence": 0.85,
              "reasoning": "Vysvětlení..."
            }}
          ]
        }}
        """
    
    def _parse_response(self, raw_text: str) -> CodingResult:
        # Parse JSON z LLM response
        data = json.loads(raw_text)
        return CodingResult(codes=data["codes"], raw_response=raw_text)
```

#### 4.2 API Endpoint US2
**Test:**
```python
# tests/integration/test_api_coding.py
def test_code_event_endpoint(client, auth_headers):
    response = client.post(
        "/v1/code-event",
        json={"clinical_text": "Pacient s diabetem 2. typu"},
        headers=auth_headers
    )
    
    assert response.status_code == 200
    data = response.json()
    assert len(data["codes"]) > 0
    assert data["codes"][0]["code"].startswith("E11")  # Diabetes type 2
```

**Implementace:**
```python
# src/coder_api/api/v1/coding.py
from fastapi import APIRouter, Depends
from pydantic import BaseModel

router = APIRouter(prefix="/v1", tags=["coding"])

class CodingRequest(BaseModel):
    clinical_text: str

@router.post("/code-event")
async def code_event(
    request: CodingRequest,
    llm_service: LLMService = Depends(get_llm_service),
    validator: ValidationEngine = Depends(get_validation_engine)
):
    # LLM coding
    result = await llm_service.code_clinical_event(request.clinical_text)
    
    # Validate generated codes
    for code_entry in result.codes:
        is_valid = await validator._validate_mkn10_code(code_entry["code"])
        code_entry["is_valid"] = is_valid
    
    return {
        "codes": result.codes,
        "processing_time_ms": 2500  # TODO: actual timing
    }
```

#### 4.3 Performance Testing
```python
# tests/performance/test_load.py
from locust import HttpUser, task, between

class CoderAPIUser(HttpUser):
    wait_time = between(1, 3)
    
    @task
    def validate_kdavka(self):
        with open("tests/fixtures/valid_kdavka.201", "rb") as f:
            self.client.post("/v1/validate-kdavka", files={"file": f})
    
    @task
    def code_event(self):
        self.client.post("/v1/code-event", json={
            "clinical_text": "Pacient s hypertenzí"
        })
```

**Run:**
```bash
locust -f tests/performance/test_load.py --host=http://localhost:8000
```

**Výstupy:**
- ✅ Gemini API integrace
- ✅ Prompt template optimalizovaný
- ✅ US2 endpoint funkční
- ✅ Load test výsledky (p95 < 3s)

**Acceptance Criteria:**
- [ ] LLM generuje relevantní kódy (manuální review 10 vzorků)
- [ ] Confidence score ≥ 0.7 pro přijetí
- [ ] Validace ověřuje generované kódy
- [ ] p95 latency < 3s (US2)
- [ ] p95 latency < 200ms (US1)

---

## 🔗 Fáze 5: Integration & Testing (W11-W12)

### Cíle
- E2E testy pro oba user stories
- Security testing
- Dokumentace API
- Performance tuning

### Úkoly

#### 5.1 E2E Tests
```python
# tests/e2e/test_full_workflow.py
def test_full_validation_workflow(client, auth_token):
    # 1. Upload K-dávka
    with open("tests/fixtures/real_kdavka.201", "rb") as f:
        response = client.post(
            "/v1/validate-kdavka",
            files={"file": f},
            headers={"Authorization": f"Bearer {auth_token}"}
        )
    
    assert response.status_code == 200
    result = response.json()
    
    # 2. Verify errors detected
    assert len(result["errors"]) > 0
    
    # 3. Check audit log
    audit_response = client.get(
        "/v1/audit-logs",
        params={"endpoint": "/v1/validate-kdavka"},
        headers={"Authorization": f"Bearer {auth_token}"}
    )
    assert audit_response.status_code == 200
```

#### 5.2 Security Testing
```bash
# OWASP ZAP scan
docker run -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:8000

# Bandit (Python security linter)
bandit -r src/

# Safety (dependency check)
safety check --json
```

#### 5.3 API Documentation
```python
# src/coder_api/main.py (update)
app = FastAPI(
    title="Coder API",
    version="1.0.0",
    description="""
    REST API pro automatizaci kódování zdravotní péče podle MKN-10.
    
    ## Features
    - **US1:** Validace K-dávky podle VZP pravidel
    - **US2:** LLM kódování klinického textu
    
    ## Authentication
    OAuth2 + JWT tokens (1h TTL)
    """,
    contact={
        "name": "API Support",
        "email": "support@example.com"
    }
)
```

#### 5.4 Performance Tuning
**Database:**
```sql
-- Add indexes for hot queries
CREATE INDEX idx_mkn10_kod_trgm ON ciselnik_mkn10 USING gin(kod gin_trgm_ops);
CREATE INDEX idx_audit_endpoint ON audit_logs(endpoint, created_at);

-- Vacuum analyze
VACUUM ANALYZE;
```

**Redis:**
```python
# Warm-up cache on startup
async def warmup_cache():
    all_codes = codebook_repo.get_all_mkn10_codes()
    for code in all_codes[:1000]:  # Top 1000 codes
        await cache.set_mkn10_code(code.kod, code.to_dict())
```

**Výstupy:**
- ✅ E2E test suite (10+ scenarios)
- ✅ Security scan report
- ✅ OpenAPI spec v3.1 (auto-generated)
- ✅ Performance baseline (p95 < 200ms/3s)

**Acceptance Criteria:**
- [ ] E2E testy pokrývají happy path + error cases
- [ ] Zero high/critical security findings
- [ ] OpenAPI spec validní (Swagger Validator)
- [ ] Load test: 100 req/s sustained (10 min)

---

## 🚀 Fáze 6: Deployment & Launch (W13)

### Cíle
- Nasadit do Kubernetes
- Nastavit monitoring (Prometheus + Grafana)
- Launch checklist

### Úkoly

#### 6.1 Kubernetes Deployment
```bash
# Build Docker image
docker build -t coder-api:v1.0.0 -f deploy/docker/Dockerfile .

# Push to registry
docker tag coder-api:v1.0.0 registry.example.com/coder-api:v1.0.0
docker push registry.example.com/coder-api:v1.0.0

# Apply K8s manifests
kubectl apply -f deploy/k8s/namespace.yaml
kubectl apply -f deploy/k8s/secrets.yaml
kubectl apply -f deploy/k8s/postgres-statefulset.yaml
kubectl apply -f deploy/k8s/redis-deployment.yaml
kubectl apply -f deploy/k8s/api-deployment.yaml
kubectl apply -f deploy/k8s/ingress.yaml
```

#### 6.2 Monitoring Setup
```bash
# Prometheus + Grafana via Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack

# Import dashboard
kubectl apply -f monitoring/grafana-dashboards/api-performance.json
```

#### 6.3 Launch Checklist

**Pre-launch:**
- [ ] All tests passing (unit + integration + e2e)
- [ ] Security scan clean
- [ ] Load test passed (100 req/s)
- [ ] Database backups configured
- [ ] SSL certificates valid
- [ ] Environment variables set (production)
- [ ] Rollback plan documented

**Launch:**
- [ ] Deploy to production K8s cluster
- [ ] Smoke test (health endpoint)
- [ ] Monitor logs (ELK Stack)
- [ ] Monitor metrics (Grafana dashboard)
- [ ] Alert rules active (Alertmanager)

**Post-launch:**
- [ ] Announce to stakeholders
- [ ] Update documentation (URL, API keys)
- [ ] Monitor error rate (first 24h)
- [ ] Collect user feedback

**Výstupy:**
- ✅ Production deployment (K8s)
- ✅ Monitoring dashboards live
- ✅ Launch report

---

## 📊 Metriky úspěchu

| Metrika | Cíl | Měření |
|---------|-----|--------|
| **Test Coverage** | ≥80% | `pytest --cov` |
| **US1 Latency (p95)** | <200ms | Locust report |
| **US2 Latency (p95)** | <3s | Locust report |
| **Cache Hit Ratio** | ≥90% | Redis INFO stats |
| **Error Rate** | <1% | Prometheus `api_errors_total` |
| **Uptime (SLA)** | 99.5% | Uptime monitor (30 days) |
| **Security Findings** | 0 critical | OWASP ZAP + Bandit |

---

## 🔄 Iterační vývoj

Po úspěšném launchi (W13):

**Fáze 7: Optimalizace (W14-W16)**
- [ ] ML model fine-tuning (Gemini)
- [ ] Advanced validation rules
- [ ] Batch processing API
- [ ] Reporting dashboard

**Fáze 8: Škálování (W17-W20)**
- [ ] Multi-region deployment
- [ ] CDN integration
- [ ] Auto-scaling policies
- [ ] Cost optimization

---

## 🎯 Kritické cesty (dependencies)

```
Fáze 0 → Fáze 1 → Fáze 2 → Fáze 3 → Fáze 5 → Fáze 6
                              ↓
                         Fáze 4 ──→ Fáze 5
```

**Blocking dependencies:**
- Fáze 3 závisí na Fázi 2 (DASTA Parser)
- Fáze 4 může běžet paralelně s Fází 3
- Fáze 5 vyžaduje dokončení Fází 3 + 4
- Fáze 6 vyžaduje dokončení Fáze 5

---

## ✅ Next Steps

1. **Schválit roadmap** s týmem
2. **Alokovat resources** (dev + QA)
3. **Kickoff meeting** (W1)
4. **Weekly sprint reviews**

**Status:** 🟡 Draft (čeká na schválení)
