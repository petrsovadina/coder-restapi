## **📄 Product Requirements Document (PRD)**  
**Produkt:** Automatizace kódování a vykazování zdravotní péče  
**Forma:** REST API služba  
**Verze:** Draft 1.0  

---

### **1. Úvod a cíle**
**Problém:**  
Zdravotnická zařízení v ČR vykazují péči zdravotním pojišťovnám prostřednictvím dávkových souborů (např. K-dávka). Proces kódování diagnóz a výkonů je složitý, náchylný k chybám a časově náročný. Chyby vedou k odmítnutí dávek a finančním ztrátám.

**Cíl produktu:**  
- Automatizovat kontrolu vykazovacích kódů před odesláním pojišťovně.  
- Navrhovat vhodné kódy na základě klinických událostí.  
- Zajistit soulad s legislativou a metodikami pojišťoven.  
- Snížit administrativní zátěž lékařů a zvýšit úspěšnost vykazování.  

---

### **2. Kontext a standardy**
- **Diagnózy:** MKN-10 (spravuje ÚZIS, česká verze).  
- **Výkony:** Seznam zdravotních výkonů s bodovými hodnotami (vyhláška MZ ČR).  
- **Materiály a léčiva:** Číselníky SÚKL, ZUM/ZULP.  
- **DRG:** CZ-DRG pro hospitalizační případy.  
- **Formát dávek:** KDAVKA.XXX (ASCII, pevná struktura), definováno metodikou VZP.  
- **Validační pravidla:** Platnost kódů, kombinace diagnóz a výkonů, frekvenční omezení, úplnost dokladů.  

---

### **3. Cíloví uživatelé**
- **Primární:** Vývojáři AIS (ambulantní informační systémy), nemocniční IS.  
- **Nepřímý:** Lékaři, zdravotnický personál (přes integraci AIS).  
- **Sekundární:** Revizní lékaři, pojišťovny (pro kontrolu správnosti vykazování).  

---

### **4. Hlavní uživatelské scénáře (User Stories)**
#### **US1: Validace vykazovacích kódů**
- **Jako** AIS  
- **Chci** poslat dávku (soubor KDAVKA nebo JSON)  
- **Abych** získal report chyb, varování a doporučení oprav.  

**Akceptační kritéria:**  
- API přijme dávku, zkontroluje platnost kódů, kombinace, frekvenční limity.  
- Vrátí seznam chyb a doporučení (např. „kód 17101 není platný k datu výkonu“).  

---

#### **US2: Generování kódů z klinických událostí**
- **Jako** AIS  
- **Chci** poslat klinickou událost (text nebo strukturovaná data)  
- **Abych** dostal seznam doporučených kódů (diagnózy, výkony) s odůvodněním.  

**Akceptační kritéria:**  
- API analyzuje vstupní data (text, strukturované údaje).  
- Vrátí seznam kódů (MKN-10, výkony) + důvod přiřazení (např. „hypertenze → I10“).  

---

### **5. Funkční požadavky**
- **Validace kódů:**  
  - Kontrola platnosti kódů dle číselníků.  
  - Kontrola kombinací diagnóz a výkonů.  
  - Kontrola frekvenčních omezení.  
- **Doporučení kódů:**  
  - Na základě klinických událostí (text, strukturovaná data).  
  - Poskytnutí důvodu doporučení (transparentnost).  
- **Podpora formátů:**  
  - KDAVKA.XXX (fixed-width text).  
  - JSON (FHIR kompatibilní struktura doporučena).  
- **Výstupy:**  
  - JSON s výsledky validace nebo návrhy kódů.  

---

### **9. Příklady vstupů a výstupů**
- **Validace:**  
  Vstup: KDAVKA.111 → Výstup: JSON {doklad: status, chyby, doporučení}.  
- **Generování:**  
  Vstup: {text: „Pacient s hypertenzí…“} → Výstup: {diagnózy: [I10], výkony: [17101], důvod: „zmínka o…“}.  
