# Analýza technologií pro REST API pro automatizaci kódování

Tato analýza porovnává a doporučuje nejvhodnější technologie pro klíčové komponenty navrhovaného REST API, s důrazem na výkon, škálovatelnost, rychlost vývoje a integraci LLM.

## 1. Backend Framework (API Server)

Backend musí být rychlý, asynchronní a vhodný pro vývoj robustního REST API. Python je ideální volbou pro rychlý vývoj a snadnou integraci s LLM a datovými knihovnami.

| Framework | Výhody | Nevýhody | Doporučení |
| :--- | :--- | :--- | :--- |
| **FastAPI** | **Vysoký výkon** (díky Starlette a Uvicorn), **Asynchronní** (async/await), Automatická dokumentace (Swagger/OpenAPI), Vestavěná validace dat (Pydantic). | Novější než Flask/Django, menší ekosystém (ale rychle roste). | **Doporučeno.** Ideální pro moderní, vysoce výkonné API s důrazem na datovou validaci a rychlost. |
| **Django REST Framework (DRF)** | Velký ekosystém, robustní ORM, vhodný pro komplexní business logiku. | Synchronní (vyžaduje ASGI adaptéry pro asynchronní provoz), pomalejší vývoj API endpointů. | Vhodný pro komplexní full-stack aplikace, ale pro čisté API je **FastAPI efektivnější**. |
| **Flask** | Minimalistický, flexibilní, rychlý start. | Vyžaduje mnoho externích knihoven (pro validaci, ORM, dokumentaci), nižší výkon než FastAPI. | Vhodný pro mikroservisy, ale pro robustní API s validací je **FastAPI lepší volbou**. |

## 2. Databáze (Číselníky a Pravidla)

Databáze musí být spolehlivá, relační a schopná efektivně spravovat komplexní číselníky a pravidla.

| Databáze | Výhody | Nevýhody | Doporučení |
| :--- | :--- | :--- | :--- |
| **PostgreSQL** | **Robustní**, pokročilé funkce (JSONB, Full-Text Search), vysoká spolehlivost, silná podpora pro transakce a integritu dat. | Vyšší nároky na konfiguraci než SQLite. | **Doporučeno.** Ideální pro uložení kritických, relačních dat (číselníky, pravidla) a pro škálovatelnost. |
| **MySQL** | Velmi populární, dobrá škálovatelnost. | Historicky slabší podpora pro pokročilé relační funkce než PostgreSQL. | Dobrá alternativa, ale **PostgreSQL je preferován** pro datovou integritu a pokročilé funkce. |
| **MongoDB (NoSQL)** | Flexibilní schéma, rychlé pro jednoduché operace. | Nevhodné pro relační data (vazby výkon-diagnóza, frekvenční limity), chybí transakční integrita. | **Nevhodné.** Relační povaha číselníků vyžaduje relační databázi. |

## 3. LLM (Large Language Model) pro kódování

Model musí být přesný v medicínském kontextu a mít dobrou znalost češtiny.

| Model | Výhody | Nevýhody | Doporučení |
| :--- | :--- | :--- | :--- |
| **Google Gemini (např. 2.5 Pro/Flash)** | **Med-Gemini** je speciálně trénován na medicínských datech, což slibuje vysokou přesnost v oboru [1]. Dobrá podpora češtiny. | Méně komunitních zkušeností s kódováním MKN-10/SZV v ČR než s GPT. | **Doporučeno.** Zvolit jako primární model pro User Story 2 díky specializaci na medicínu. |
| **OpenAI GPT-4o / GPT-4** | Špičkový výkon v obecném porozumění textu, robustní API, velký ekosystém. | Vyšší cena, menší specializace na medicínské kódování než Med-Gemini. | **Doporučeno jako záloha/alternativa.** Vhodný pro komplexní texty a generování odůvodnění. |
| **Cohere (např. Command R+)** | Silný v RAG (Retrieval-Augmented Generation) a generování textu. | Méně zkušeností s medicínským kódováním v ČR. | Vhodný pro experimenty s RAG (např. prohledávání dokumentace k úhradám), ale ne jako primární kódovací model. |

## 4. DASTA Parser a Business Logic

Pro parsování DASTA a implementaci logiky je nejvhodnější **vlastní řešení** v Pythonu.

*   **DASTA Parser:** Vzhledem k unikátnímu formátu DASTA (text s pevnou šířkou polí) neexistuje hotová, veřejně dostupná knihovna. Bude nutné implementovat **vlastní parser** (např. pomocí regulárních výrazů `re` nebo strukturovaného parsování textu) v Pythonu.
*   **Business Logic Engine:** Logika validace (frekvenční limity, vazby) musí být implementována přímo v Pythonu (např. v modulu `validation_engine.py`) s využitím dat z PostgreSQL.

## 5. Infrastruktura a Deployment

Pro zajištění škálovatelnosti a spolehlivosti je klíčová kontejnerizace.

| Komponenta | Technologie | Důvod |
| :--- | :--- | :--- |
| **Kontejnerizace** | **Docker** | Izolace prostředí, snadná replikace a nasazení. |
| **Orchestrace** | **Docker Compose** (pro vývoj), **Kubernetes** (pro produkci) | Automatické škálování a správa kontejnerů. |
| **Cache** | **Redis** | Rychlé ukládání často používaných dat (např. číselníků) pro snížení latence. |

---
**Reference:**
[1] Google s Med-Gemini prý poráží GPT-4. SvetAndroida.cz.
