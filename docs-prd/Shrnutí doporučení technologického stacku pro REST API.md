# Shrnutí doporučení technologického stacku pro REST API

## 1. Přehled doporučeného stacku

Na základě analýzy požadavků na vysoký výkon, škálovatelnost, datovou integritu a integraci umělé inteligence doporučujeme následující technologický stack:

| Komponenta | Doporučená technologie | Klíčový přínos |
| :--- | :--- | :--- |
| **Backend Framework** | **FastAPI (Python)** | **Vysoký výkon** a asynchronní zpracování pro rychlé volání LLM a validaci. |
| **Databáze** | **PostgreSQL** | **Datová integrita** a pokročilé relační funkce pro správu číselníků a komplexních pravidel. |
| **LLM Engine** | **Google Gemini** | **Medicínská specializace** (Med-Gemini) pro přesné kódování klinických textů v češtině. |
| **Cache** | **Redis** | Snížení latence a zrychlení přístupu k často používaným číselníkům. |
| **Deployment** | **Docker** | Robustní a škálovatelná kontejnerizace. |

## 2. Odůvodnění klíčových rozhodnutí

### A. Proč FastAPI?

FastAPI je moderní Python framework, který je postaven na asynchronních základech. To je kritické pro váš projekt, protože:
1.  **Validace K-dávky** vyžaduje rychlé parsování a zpracování velkých dávek.
2.  **LLM Kódování** zahrnuje volání externího API (Gemini), což je I/O operace. Asynchronní kódování umožňuje API obsluhovat mnoho požadavků souběžně, aniž by čekalo na odpověď od LLM.

### B. Proč PostgreSQL?

Vykazování zdravotní péče je založeno na **komplexních relačních pravidlech** (vazby výkon-diagnóza, frekvenční limity). PostgreSQL poskytuje:
*   **Transakční integritu:** Zajišťuje, že data číselníků a pravidel jsou vždy konzistentní.
*   **Pokročilé funkce:** Umožňuje ukládat a efektivně dotazovat složitá pravidla, která by byla v NoSQL databázi obtížně spravovatelná.

### C. Proč Google Gemini?

Pro User Story 2 (analýza klinického textu) je nejdůležitější **přesnost a medicínská relevance**.
*   **Med-Gemini:** Modely Gemini jsou speciálně trénovány na medicínských datech, což je pro kódování MKN-10 a SZV klíčové.
*   **Čeština:** Model má silnou podporu pro český jazyk, což je nezbytné pro analýzu anamnéz a nálezů v češtině.

## 3. Další kroky

Doporučený stack je připraven pro implementaci. Následující kroky by měly zahrnovat:

1.  **Detailní specifikace DASTA Parseru:** Implementace vlastního parseru v Pythonu pro transformaci K-dávky na JSON.
2.  **Vytvoření datového modelu:** Implementace databázových modelů pro `Ciselnik_Vykonu`, `Ciselnik_Diagnoz` a `Pravidla_Vykazovani` v PostgreSQL.
3.  **Nastavení infrastruktury:** Konfigurace Docker Compose pro lokální vývojové prostředí (FastAPI, PostgreSQL, Redis).

Tento stack vám umožní rychle uvést na trh robustní, výkonné a inovativní řešení.
