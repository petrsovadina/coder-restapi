<!--
SYNC IMPACT REPORT
==================
Version Change: 1.0.0 → 1.1.0
Date: 2025-11-02

Modified Principles:
- ENHANCED: I. Regulatory Compliance - Added requirement for regular codebook updates
- ENHANCED: II. API-First Design - Strengthened OpenAPI documentation requirements
- ENHANCED: VI. Performance Standards - Added caching strategy requirements
- NEW: VIII. Modular Architecture - Component isolation and service boundaries

Added Sections:
- Cost Management subsection in Security Requirements
- Technology Stack Constraints in Development Workflow

Removed Sections:
- None

Templates Status:
- ✅ plan-template.md: UPDATED - Constitution Check now includes all 8 principles with specific checkboxes
- ✅ spec-template.md: Compatible - No changes needed
- ✅ tasks-template.md: Compatible - Modular architecture supports existing task structure
- ✅ README.md: UPDATED - Created with full project overview and constitution references

Follow-up TODOs:
- None - all changes are additive and backward compatible

Version Bump Rationale:
MINOR (1.1.0) - New principle added (Modular Architecture) and existing principles enhanced 
without breaking backward compatibility. No removal or redefinition of existing principles.
-->

# Ústava projektu Healthcare Coding Automation API

## Základní principy

### I. Soulad s legislativou (Healthcare Compliance)

Všechny funkce MUSÍ být v souladu s českou legislativou a standardy ve zdravotnictví:

- MKN-10 (česká verze) pro diagnózy spravované ÚZIS
- Seznam zdravotních výkonů (SZV) s bodovými hodnotami dle vyhlášky MZ ČR
- Číselníky SÚKL pro materiály a léčiva (ZUM/ZULP)
- CZ-DRG pro hospitalizační případy
- Formát DASTA (KDAVKA.XXX) dle metodiky VZP
- GDPR a zákon č. 372/2011 Sb. o zdravotních službách

**Aktualizace číselníků:**
- Číselníky MUSÍ být pravidelně aktualizovány (minimálně čtvrtletně)
- Systém MUSÍ podporovat verzování číselníků s historií platnosti
- Změny v číselních MUSÍ být automaticky detekovány a oznámeny administrátorům
- Každá validace MUSÍ být provedena proti číselníkům platným k datu výkonu

**Odůvodnění**: Nedodržení předpisů vede k zamítnutí dávek pojišťovnou, finančním ztrátám zdravotnických zařízení a potenciální právní odpovědnosti. Zastaralé číselníky způsobují falešné chyby. Tento princip je základem produktové hodnoty.

### II. API-First Design

Každá funkce MUSÍ být vystavena prostřednictvím REST API endpointů:

- Specifikace OpenAPI 3.0+ POVINNÁ pro všechny endpointy
- JSON jako primární formát výměny dat
- Podpora formátu DASTA fixed-width text kde je to relevantní
- Doporučená kompatibilita s FHIR pro strukturovaná klinická data
- Synchronní operace pro validaci (<200ms p95)
- Asynchronní operace pro LLM generování kódů se sledováním stavu

**Dokumentace:**
- Swagger UI MUSÍ být dostupné na `/docs` endpointu
- Všechny endpointy MUSÍ obsahovat příklady requestů a responses
- Chybové kódy a zprávy MUSÍ být dokumentovány v OpenAPI spec
- Changelog API verzí MUSÍ být veřejně dostupný

**Odůvodnění**: Primární uživatelé jsou vývojáři AIS integrující do existujících systémů. Stabilita API a jasná dokumentace přímo ovlivňuje míru adopce. Chybějící nebo špatná dokumentace vede k integračním problémům a ztrátě zákazníků.

### III. Test-First Development (NEJEDNATELNÉ)

Test-Driven Development je POVINNÝ pro veškerou implementační práci:

- Testy MUSÍ být napsány před implementačním kódem
- Testy MUSÍ nejprve selhat (red state)
- Implementace pokračuje pouze po schválení testů
- Testy MUSÍ projít (green state) před považováním za hotové
- Refaktoring MUSÍ udržovat projité testy
- Přísné dodržování cyklu Red-Green-Refactor

**Odůvodnění**: Chyby ve zdravotnickém kódování mají finanční dopady a ovlivňují péči o pacienty. TDD zajišťuje správnost a zabraňuje regresi v validační logice. Tento princip není předmětem vyjednávání.

### IV. Ochrana osobních údajů a bezpečnost

Zpracování zdravotnických dat MUSÍ splňovat nejvyšší bezpečnostní standardy:

- ZÁKAZ trvalého ukládání osobních identifikačních údajů (PII) bez explicitního souhlasu
- Veškerá API komunikace MUSÍ používat TLS 1.3+
- Autentizace POVINNÁ pro všechny endpointy (doporučeno API key + JWT)
- Auditní logování všech přístupů k datům a doporučení kódů
- Sanitizace vstupů proti injection útokům POVINNÁ
- Rate limiting pro prevenci zneužití (doporučeno: 100 req/min per klient)

**Odůvodnění**: GDPR článek 9 (zvláštní kategorie dat) a český zdravotnický zákon ukládají přísné požadavky. Porušení má vážné právní a reputační důsledky.

### V. Transparentnost a auditovatelnost

Všechna rozhodnutí o kódování MUSÍ být vysvětlitelná a sledovatelná:

- Validační chyby MUSÍ obsahovat konkrétní odkaz na pravidlo (např. "MKN-10 kombinační pravidlo C1.2")
- LLM doporučení kódů MUSÍ obsahovat textový úryvek z klinické události jako zdůvodnění
- Všechna doporučení MUSÍ citovat zdroj (číselník, pravidlo, inference)
- Strukturované logování průběhu validační logiky
- Verzování modelů, pravidel a číselníků použitých v každém rozhodnutí

**Odůvodnění**: Lékaři a revizní lékaři pojišťoven musí rozumět, proč byly kódy přijaty/zamítnuty. Black-box rozhodnutí podkopávají důvěru a komplikují řešení sporů. Vysvětlitelná AI je klíčovým diferenciátorem.

### VI. Výkonnostní standardy

Doba odezvy MUSÍ splňovat požadavky klinického workflow:

- Validační endpoint (US1): <200ms p95 latence pro jeden claim
- Dávková validace: <5s pro 100 claimů
- LLM generování kódů (US2): <3s p95 pro jednu klinickou událost
- Databázové dotazy: <50ms pro vyhledání v číselníku
- Dostupnost systému: 99.5% uptime SLA

**Caching strategie:**
- Redis MUSÍ být použit pro caching často používaných číselníků
- TTL pro cache MUSÍ odpovídat četnosti aktualizací číselníků
- Cache invalidace MUSÍ být automatická při aktualizaci číselníků
- Warm-up cache při startu aplikace pro kritické číselníky

**Odůvodnění**: Integrace AIS do real-time klinického workflow vyžaduje responzivní operace. Zpoždění frustrují uživatele a snižují adopci. Caching je kritický pro splnění výkonnostních cílů.

### VII. Verzování a zpětná kompatibilita

Stabilita API MUSÍ být zachována pro integrované klienty:

- Sémantické verzování (MAJOR.MINOR.PATCH) pro API
- Breaking changes vyžadují MAJOR version bump a 6měsíční deprecation notice
- Nová volitelná pole/endpointy → MINOR version bump
- Bug fixes/vyjasnění → PATCH version bump
- Všechny verze dokumentovány s migračními průvodci
- Minimálně 12měsíční podpora pro předchozí MAJOR verzi

**Odůvodnění**: Dodavatelé AIS plánují integrační cykly měsíce dopředu. Neočekávané breaking changes způsobují selhání nasazení a erodují důvěru.

### VIII. Modulární architektura

Systém MUSÍ být navržen jako sada nezávislých, znovupoužitelných komponent:

**Povinné komponenty:**
- **DASTA Parser**: Samostatná knihovna pro parsing KDAVKA formátu
- **Validation Engine**: Izolovaná služba pro aplikaci validačních pravidel
- **LLM Service**: Samostatná služba pro komunikaci s LLM API (Gemini)
- **Cache Service**: Abstrakce nad Redis pro správu cache
- **API Gateway**: FastAPI aplikace pro routing a orchestraci

**Požadavky na komponenty:**
- Každá komponenta MUSÍ mít jasně definované rozhraní (interface)
- Komponenty MUSÍ být testovatelné nezávisle
- Závislosti mezi komponentami MUSÍ být explicitní (dependency injection)
- Každá komponenta MUSÍ mít vlastní sadu unit testů
- Změna v jedné komponentě NESMÍ vyžadovat změny v jiných komponentách (loose coupling)

**Technologický stack:**
- Backend: FastAPI (Python 3.11+)
- Databáze: PostgreSQL 15+
- Cache: Redis 7+
- LLM: Google Gemini API (primárně), fallback na OpenAI GPT-4
- Deployment: Docker + Docker Compose (dev), Kubernetes (prod)

**Odůvodnění**: Modulární architektura umožňuje nezávislý vývoj, testování a nasazování komponent. Snižuje komplexitu, zlepšuje maintainability a umožňuje týmovou paralelní práci. Jasný tech stack zajišťuje konzistenci a prevenci technologického dluhu.

## Bezpečnostní požadavky

**Povinné bezpečnostní kontroly:**

- API autentizace: Bearer token (JWT) s role-based access control (RBAC)
- Transportní šifrování: TLS 1.3 pouze se silnými cipher suites
- Validace vstupů: Whitelist-based validace všech uživatelských vstupů
- Výstupní encoding: Context-aware escaping pro prevenci injection
- Audit trail: Neměnné logy všech validačních/doporučovacích requestů s timestamp, client ID, request hash
- Dependency scanning: Automatizované CVE kontroly v CI/CD pipeline
- Secrets management: Žádné hardcoded credentials; použití environment variables nebo vault

**Správa nákladů (Cost Management):**
- Rate limiting MUSÍ být konfigurované per zákazník na základě předplatného
- LLM API volání MUSÍ být monitorována a reportována pro cost tracking
- Cache hit rate MUSÍ být >80% pro snížení LLM API costs
- Batch processing MUSÍ být preferováno kde je to možné pro optimalizaci nákladů

**Retence dat:**

- Auditní logy: 7 let (dle českých požadavků na retenci zdravotní dokumentace)
- Anonymizované agregované statistiky: Neomezeně (pro zlepšování modelu)
- Request/response payloads s PII: Smazání po 24 hodinách pokud není explicitní souhlas

## Vývojový workflow

**Pre-Implementation Gates:**

1. Feature specifikace (spec.md) schválena s prioritizovanými user stories
2. Implementační plán (plan.md) schválen s constitution check pasováním
3. Testy napsány a schváleny reviewerem
4. Testy potvrzují selhání (red state)

**Implementační proces:**

1. Implementace minimálního kódu pro průchod jednoho testu
2. Ověření průchodu testu (green state)
3. Refaktoring pro čistotu/výkon při udržení green state
4. Opakování pro zbývající testy

**Pre-Merge požadavky:**

- Všechny testy projdou (unit, integration, contract)
- Code coverage ≥80% pro nový kód
- OpenAPI spec aktualizována pro API změny
- Changelog entry přidán s uvedením dopad na verzi
- Security review pro změny v autentizaci/zpracování dat
- Performance benchmarky splňují thresholdy z principu VI

**Technologické constrainty:**

- Python 3.11+ POVINNÉ (type hints, performance improvements)
- FastAPI framework POVINNÝ pro API endpoints
- PostgreSQL POVINNÉ pro persistence
- Redis POVINNÉ pro caching
- Pytest POVINNÉ pro testování
- Black + isort + flake8 POVINNÉ pro code formatting
- Pre-commit hooks MUSÍ být nastaveny pro všechny vývojáře

**Justifikace složitosti:**

Jakákoli odchylka od principů nebo přidání složitosti MUSÍ být dokumentována v plan.md s:

- Konkrétní porušený princip nebo přidanou složitostí
- Business justifikace (regulace, výkon, bezpečnost)
- Mitigační plán nebo akceptace trade-offu
- Schválení technical leadem

## Governance

**Autorita**: Tato ústava má přednost před všemi ostatními vývojovými praktikami. V případě konfliktu mají přednost principy ústavy.

**Proces změn:**

1. Navrhovaná změna dokumentována s odůvodněním a analýzou dopadu
2. Review technical leadem a product ownerem
3. Schválení vyžaduje konsenzus (žádné veta)
4. Version bump dle pravidel sémantického verzování
5. Migrační plán POVINNÝ pro MAJOR změny ovlivňující existující práci
6. Všechny šablony a dokumentace aktualizovány do 1 týdne od ratifikace

**Verifikace compliance:**

- Všechny feature plány MUSÍ obsahovat sekci "Constitution Check" potvrzující soulad
- Code reviews MUSÍ verifikovat dodržování principů (zejména III, IV, VII, VIII)
- Měsíční audit mergnutých features pro retrospektivní verifikaci compliance
- Nálezy non-compliance vyžadují okamžitý remediační plán

**Living Document**: Tato ústava bude revidována čtvrtletně a upravována dle vývoje potřeb projektu. Nicméně principy I, III a IV jsou fundamentální a vyžadují jednomyslné schválení pro modifikaci.

**Verze**: 1.1.0 | **Ratifikováno**: 2025-11-02 | **Poslední změna**: 2025-11-02
