# Doporučený technologický stack pro REST API pro automatizaci kódování

Na základě detailní analýzy požadavků na výkon, škálovatelnost, rychlost vývoje a integraci LLM doporučujeme následující technologický stack.

## 1. Přehled doporučeného stacku

| Komponenta | Doporučená technologie | Odůvodnění |
| :--- | :--- | :--- |
| **Backend Framework** | **FastAPI (Python)** | Nejvyšší výkon, asynchronní zpracování, automatická dokumentace a vestavěná datová validace (Pydantic). Ideální pro moderní API. |
| **Databáze** | **PostgreSQL** | Robustní, relační databáze s pokročilými funkcemi (JSONB, transakce) nezbytnými pro integritu kritických číselníků a pravidel. |
| **LLM Engine** | **Google Gemini** (např. 2.5 Pro/Flash) | Specializace na medicínské úlohy (Med-Gemini) a silná podpora pro vícejazyčné (včetně češtiny) medicínské texty. |
| **Cache** | **Redis** | Snížení latence a zátěže DB rychlým ukládáním často dotazovaných číselníků. |
| **Parser DASTA** | **Vlastní implementace v Pythonu** | Neexistuje veřejná knihovna. Nutná implementace parsování textu s pevnou šířkou polí. |
| **Deployment** | **Docker** (kontejnerizace) | Zajištění konzistence prostředí, snadné škálování a nasazení. |

## 2. Detailní odůvodnění klíčových voleb

### A. FastAPI jako páteř API

Volba **FastAPI** je klíčová pro splnění požadavků na **výkon a rychlost**.

*   **Asynchronní zpracování:** Zpracování K-dávek a volání externích LLM API jsou I/O operace. FastAPI využívá `async/await`, což umožňuje efektivně zpracovávat velké množství souběžných požadavků bez blokování.
*   **Datová validace:** Pydantic modely zajišťují automatickou validaci vstupních a výstupních dat (JSON), což je kritické pro spolehlivost zdravotnického API.
*   **Rychlost vývoje:** Automatické generování dokumentace (Swagger UI) a jednoduchá syntaxe výrazně urychlí vývoj.

### B. PostgreSQL pro datovou integritu

**PostgreSQL** je nejlepší volbou pro uložení číselníků a pravidel.

*   **Integrita dat:** Relační povaha dat (vazby mezi kódy výkonů, diagnózami a pravidly) vyžaduje silnou transakční integritu, kterou PostgreSQL poskytuje.
*   **Pokročilé funkce:** Možnost ukládat komplexní pravidla ve formátu JSONB a využívat pokročilé indexování pro rychlé vyhledávání v číselnících.

### C. Google Gemini pro medicínské kódování

Pro User Story 2 (kódování klinické události) je klíčová přesnost v medicínském kontextu.

*   **Medicínská specializace:** Modely Gemini, zejména ty s medicínským zaměřením (Med-Gemini), prokazují vysokou přesnost v medicínských úlohách, často překonávající konkurenci [1].
*   **Čeština:** Modely od Google a OpenAI mají silnou podporu pro češtinu, což je nezbytné pro analýzu klinických textů v českém jazyce.
*   **Integrace:** Oba modely (Gemini i GPT) mají robustní API a snadno se integrují do Python backendu. Doporučujeme začít s Gemini a mít GPT jako záložní strategii.

## 3. Architektura a implementace

Doporučený stack se promítá do modulární architektury, kde každá komponenta plní specifickou roli:

| Modul | Technologie | Role |
| :--- | :--- | :--- |
| **API Gateway** | FastAPI | Přijímá požadavky, směruje je na příslušné služby. |
| **DASTA Parser** | Python (vlastní kód) | Transformuje textovou K-dávku na strukturovaný JSON. |
| **Validation Engine** | Python + PostgreSQL | Aplikuje úhradová pravidla a generuje chyby/varování. |
| **LLM Service** | Python + Gemini API | Analyzuje klinický text a navrhuje kódy MKN-10/SZV. |
| **Cache Service** | Redis | Ukládá číselníky pro rychlý přístup. |

## 4. Závěr

Doporučený stack **FastAPI + PostgreSQL + Gemini** představuje moderní, výkonné a škálovatelné řešení, které je optimálně připraveno na specifické požadavky vašeho projektu, zejména na kombinaci vysoké datové integrity a pokročilé umělé inteligence.

---
**Reference:**
[1] Google s Med-Gemini prý poráží GPT-4. SvetAndroida.cz.
