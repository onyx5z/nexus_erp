# NEXUS Vállalati Könyvelési Rendszer

## Áttekintés

Ez a dokumentum átfogó útmutatót nyújt az ERP megoldásunkban implementált könyvelési rendszerhez. A rendszer modern könyvelési elveken alapul, moduláris architektúrával, amely elválasztja az alapvető funkcionalitást a specializált könyvelési folyamatoktól.

## Tartalomjegyzék

1. [Architektúra Áttekintés](#architektúra-áttekintés)
2. [Központi Könyvelési Modul](#központi-könyvelési-modul)
3. [Specializált Könyvelési Modulok](#specializált-könyvelési-modulok)
   - [Általános Könyvelés](#általános-könyvelés)
   - [Szállítói Könyvelés](#szállítói-könyvelés)
   - [Vevői Könyvelés](#vevői-könyvelés)
4. [Pénzügyi Jelentéskészítés](#pénzügyi-jelentéskészítés)
5. [Végponttól Végpontig Tartó Könyvelési Folyamat](#végponttól-végpontig-tartó-könyvelési-folyamat)
   - [Rendszerbeállítás](#rendszerbeállítás)
   - [Napi Műveletek](#napi-műveletek)
   - [Időszak Végi Eljárások](#időszak-végi-eljárások)
   - [Jelentéskészítési Folyamatok](#jelentéskészítési-folyamatok)
6. [Technikai Implementáció](#technikai-implementáció)

## Architektúra Áttekintés

A könyvelési rendszer moduláris architektúrával rendelkezik:

```
┌─────────────────────────────────────────────────┐
│                                                 │
│           Pénzügyi Jelentéskészítés             │
│                                                 │
└───────────────────┬─────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│                                                 │
│               Központi Könyvelés                │
│          (Számlatükör, Időszakok)               │
│                                                 │
└───────┬─────────────────┬────────────┬──────────┘
        │                 │            │
        ▼                 ▼            ▼
┌───────────────┐ ┌───────────────┐ ┌────────────────┐
│               │ │               │ │                │
│   Általános   │ │   Szállítói   │ │     Vevői      │
│   Könyvelés   │ │   Könyvelés   │ │    Könyvelés   │
│               │ │               │ │                │
└───────────────┘ └───────────────┘ └────────────────┘
```

- **Központi Könyvelés**: Meghatározza az alapvető struktúrákat (Számlatükör, pénzügyi időszakok), amelyeket az összes többi modul használ.
- **Specializált Modulok**: Kezelik a különböző típusú könyvelési tranzakciókat (általános tételek, szállítói számlák, vevői számlák).
- **Pénzügyi Jelentéskészítés**: Konszolidálja az adatokat az összes modulból, hogy pénzügyi kimutatásokat és jelentéseket generáljon.

## Központi Könyvelési Modul

A Központi Könyvelési modul (core_accounting) az egész könyvelési rendszer alapja. Önmagában nem rögzít tranzakciókat, hanem biztosítja az alapvető struktúrákat és referenciákat, amelyekre az összes többi könyvelési modul támaszkodik.

### Kulcsfontosságú Komponensek

#### Számla Típusok

A Számla Típusok meghatározzák a számlák alapvető osztályozását a rendszerben:

- **Eszköz Számlák**: A vállalat tulajdonában lévő erőforrások (tartozik normál)
- **Kötelezettség Számlák**: A vállalat által tartozó kötelezettségek (követel normál)
- **Saját Tőke Számlák**: A vállalat eszközeiben fennmaradó érdekeltség a kötelezettségek levonása után (követel normál)
- **Bevétel Számlák**: A vállalat által szerzett bevétel (követel normál)
- **Kiadás Számlák**: A vállalat által felmerült költségek (tartozik normál)

Minden számlatípus olyan tulajdonságokkal rendelkezik, mint:
- Normál esetben követellel vagy tartozikkal növekszik-e (is_credit_normal)
- A mérlegben vagy az eredménykimutatásban jelenik-e meg (is_balance_sheet)
- Sorrend a jelentésekben

#### Számla Kategóriák

A Számla Kategóriák részletesebb osztályozást biztosítanak az egyes számlatípusokon belül:

- Példák: "Forgóeszközök", "Befektetett eszközök", "Rövid lejáratú kötelezettségek" stb.
- A kategóriák szülő-gyermek kapcsolatokkal rendelkezhetnek hierarchikus struktúra létrehozásához
- Minden kategória egy számlatípushoz kapcsolódik

#### Számlatükör

A Számlatükör meghatározza a rendszerben használt összes számlát:

- **Alapinformációk**: Számlaszám, név, leírás
- **Struktúra**: Típus, kategória, főszámla, hierarchiaszint
- **Viselkedés**: Követel/tartozik normál, kontrolszámla státusz, könyvelhetőség
- **Jelentéskészítés**: Pénzügyi kimutatás szakasz, jelentéscsoport
- **Ellenőrzések**: Egyeztetési követelmények, manuális bevitel engedélyei

Fő funkciók:
- Hierarchikus számlastruktúra szülő-gyermek kapcsolatokkal
- Kontrolszámlák, amelyekre közvetlenül nem lehet könyvelni
- Pénzügyi kimutatás szakaszok osztályozása
- Jelentéscsoportok kategóriái specializált jelentésekhez

#### Pénzügyi Évek és Könyvelési Időszakok

A rendszer meghatározza:

- **Pénzügyi Évek**: A szervezet pénzügyi jelentéskészítési évei
- **Könyvelési Időszakok**: A pénzügyi évek részegységei (hónapok, negyedévek)
- Státuszkövetés (nyitott/zárt) időszakokra és évekre

#### Számlaegyenlegek

A rendszer fenntartja:

- Számlaegyenlegek minden könyvelési időszak végén
- Tartozik, követel és záró egyenlegek minden számlára
- Egyenlegszámítások a számlatípus alapján (is_credit_normal)

## Specializált Könyvelési Modulok

### Általános Könyvelés

Az Általános Könyvelési modul (general_accounting) rögzíti a különféle pénzügyi tranzakciókat, amelyek nem illeszkednek specializált modulokba, és a pénzügyi tranzakciók központi tárhelyeként szolgál.

#### Kulcsfontosságú Komponensek

- **Főkönyvi Tétel**: Rögzíti a tranzakciókat olyan részletekkel, mint a tranzakció típusa, dátumok, összegek, pénznem stb.
- **Főkönyvi Sor**: Egyedi sorokat tartalmaz számlával, tartozik/követel összeggel és elemzési dimenziókkal
- **Audit Naplózás**: A főkönyvi tételek és sorok változásainak átfogó követése

#### Tranzakció Típusok
- Manuális naplótételek
- Korrekciós tételek
- Elhatárolások és halasztások
- Belső átvezetések
- Időszak végi záró tételek
- Javító tételek

#### Munkafolyamat Állapotok
- Piszkozat: Kezdeti állapot, módosítható
- Könyvelt: Véglegesített és hatással van a számlaegyenlegekre
- Sztornózott: Érvénytelenített tranzakció

### Szállítói Könyvelés

A Szállítói Könyvelési modul (accounting_supplier) kezeli a szállítókkal és kiadásokkal kapcsolatos összes pénzügyi tranzakciót.

#### Kulcsfontosságú Komponensek

- **Szállítói Számla Fejléc**: Rögzíti a számla részleteit, mint dátumok, számlaszám, összegek, pénznem, stb.
- **Szállítói Számla Sor**: Egyedi sorokat tartalmaz számlával, leírással, tartozik összeggel
- **Audit Naplózás**: Követi a szállítói számlák változásait

#### Jellemzők
- Beérkező szállítói számlák rögzítése és kezelése
- Különböző számlatípusok támogatása (standard, jóváírások, terhelések)
- ÁFA/adó kezelés automatikus számításokkal
- Több pénznem támogatása
- Munkafolyamat státuszkövetés (piszkozat, könyvelt, sztornózott)
- Kifizetés követés és kezelés

### Vevői Könyvelés

A Vevői Könyvelési modul (accounting_customer) kezeli a vevőkkel és bevételekkel kapcsolatos összes pénzügyi tranzakciót.

#### Kulcsfontosságú Komponensek
- Vevői számla kezelés
- Bevétel követés és kategorizálás
- Beérkezett fizetések feldolgozása
- Vevői számla kezelés

#### Jellemzők
- Kimenő vevői számlák rögzítése és kezelése
- Különböző számlatípusok támogatása (standard, jóváírások, proforma)
- ÁFA/adó kezelés
- Több pénznem támogatása
- Bevétel kategorizálás és elemzés
- Beérkezett fizetések követése
- Vevői számlaegyenlegek figyelése

## Pénzügyi Jelentéskészítés

A Pénzügyi Jelentéskészítési modul (financial_reporting) pénzügyi kimutatásokat és jelentéseket generál az összes könyvelési modul adatai alapján.

### Kulcsfontosságú Komponensek

- **Jelentés Sablonok**: Meghatározzák a pénzügyi jelentések struktúráját és formátumát
- **Generált Jelentések**: A generált jelentések nyilvántartása
- **Jelentés Típusok**:
  - Mérleg
  - Eredménykimutatás
  - Cash Flow Kimutatás
  - Saját Tőke Változás Kimutatás
  - Adóbevallások
  - Egyedi Jelentések

### Jelentés Generálási Folyamat

1. **Adatgyűjtés**: Összesíti az adatokat az összes könyvelési modulból
2. **Adatfeldolgozás**: Alkalmazza a könyvelési szabályokat és számításokat
3. **Jelentés Formázás**: A jelentés sablonok szerint strukturálja az adatokat
4. **Kimeneti Generálás**: Jelentéseket hoz létre különböző formátumokban (PDF, Excel, XML, stb.)

## Végponttól Végpontig Tartó Könyvelési Folyamat

### Rendszerbeállítás

#### 1. Számlatükör Beállítás

1. **Számla Típusok Meghatározása**:
   - Szükséges számlatípusok létrehozása (Eszköz, Kötelezettség, Saját Tőke, Bevétel, Kiadás)
   - Követel/tartozik normál viselkedés beállítása minden típushoz
   - Jelentési sorrend meghatározása

2. **Számla Kategóriák Létrehozása**:
   - Kategóriák meghatározása minden számlatípuson belül
   - Kategória hierarchiák kialakítása, ha szükséges
   - Megfelelő kódok hozzárendelése a kategóriákhoz

3. **Számlatükör Felépítése**:
   - Számla bejegyzések létrehozása egyedi számlaszámokkal
   - Minden számla hozzárendelése a megfelelő számlatípushoz és kategóriához
   - Számla tulajdonságok konfigurálása (könyvelhetőségi státusz, egyeztetési követelmények)
   - Számla hierarchiák beállítása a jelentésekhez
   - Pénzügyi kimutatás szakaszok és jelentéscsoportok meghatározása

#### 2. Pénzügyi Időszak Konfiguráció

1. **Pénzügyi Évek Meghatározása**:
   - Pénzügyi évek létrehozása kezdő és záró dátumokkal
   - Összehangolás a szervezet jelentéskészítési naptárával

2. **Könyvelési Időszakok Konfigurálása**:
   - Havi vagy negyedéves időszakok beállítása a pénzügyi éveken belül
   - Annak biztosítása, hogy az időszakok a teljes pénzügyi évet lefedik hézagok vagy átfedések nélkül

#### 3. Rendszervezérlők Beállítása

1. **Felhasználói Hozzáférés Konfigurálása**:
   - Megfelelő jogosultságok beállítása a könyvelési személyzet számára
   - Jóváhagyási munkafolyamatok meghatározása a tranzakciókhoz

2. **Validálási Szabályok Kialakítása**:
   - Szabályok konfigurálása a tranzakciók validálásához
   - Kontrollok beállítása a zárt időszakokba történő könyvelés megakadályozására

### Napi Műveletek

#### 1. Szállítói Számla Feldolgozás

1. **Számla Érkeztetés és Rögzítés**:
   - Szállítói számlák rögzítése az összes releváns részlettel
   - Alátámasztó dokumentumok csatolása
   - Megfelelő számlakódok hozzárendelése a számlasorokhoz

2. **Számla Jóváhagyás és Könyvelés**:
   - Jóváhagyásra küldés a kialakított munkafolyamatok alapján
   - Számla kódolás ellenőrzése és jóváhagyása
   - Jóváhagyott számlák könyvelése a főkönyvbe

3. **Kifizetés Feldolgozás**:
   - Fizetési határidők követése
   - Kifizetések feldolgozása
   - Fizetési tranzakciók rögzítése a rendszerben

#### 2. Vevői Számla Feldolgozás

1. **Számla Létrehozás**:
   - Vevői számlák generálása megfelelő tételsorokkal
   - Helyes bevételszámlák hozzárendelése
   - Adók és végösszegek számítása

2. **Számla Kibocsátás**:
   - Számlák ellenőrzése és jóváhagyása
   - Kibocsátás a vevők részére
   - Könyvelés a főkönyvbe

3. **Beérkezett Fizetések Feldolgozása**:
   - Vevői fizetések rögzítése
   - Fizetések hozzárendelése a megfelelő számlákhoz
   - Vevői számlaegyenlegek frissítése

#### 3. Általános Könyvelési Tranzakciók

1. **Naplótétel Létrehozás**:
   - Naplótételek létrehozása különféle tranzakciókhoz
   - Kiegyensúlyozott tételek biztosítása (tartozik = követel)
   - Megfelelő leírások és dokumentációk megadása

2. **Ellenőrzés és Jóváhagyás**:
   - Naplótételek ellenőrzése a pontosság érdekében
   - Szükséges jóváhagyások beszerzése
   - Megfelelő számlakódolás biztosítása

3. **Könyvelés és Dokumentáció**:
   - Jóváhagyott tételek könyvelése a főkönyvbe
   - Alátámasztó dokumentáció fenntartása

### Időszak Végi Eljárások

#### 1. Számla Egyeztetés

1. **Bank Egyeztetés**:
   - Banki kivonatok egyeztetése a rendszer nyilvántartásával
   - Eltérések kivizsgálása és megoldása
   - Korrekciós tételek rögzítése, ha szükséges

2. **Alkönyvi Egyeztetés**:
   - Vevői korosítás egyeztetése a főkönyvvel
   - Szállítói korosítás egyeztetése a főkönyvvel
   - Eltérések kivizsgálása és megoldása

3. **Általános Számla Felülvizsgálat**:
   - Számlaegyenlegek felülvizsgálata észszerűség szempontjából
   - Szokatlan egyenlegek vagy tranzakciók kivizsgálása
   - Megfelelő elhatárolások és halasztások biztosítása

#### 2. Korrekciós Tételek

1. **Elhatárolások és Halasztások**:
   - Elhatárolások rögzítése a felmerült, de még nem könyvelt kiadásokhoz
   - Halasztások rögzítése az előre fizetett kiadásokhoz
   - Bevétel-elismerési korrekciók feldolgozása

2. **Értékcsökkenés és Amortizáció**:
   - Befektetett eszközök értékcsökkenésének számítása és rögzítése
   - Immateriális javak amortizációjának rögzítése

3. **Egyéb Korrekciók**:
   - Készlet korrekciók rögzítése
   - Deviza átértékelések feldolgozása
   - Bármilyen egyéb időszak végi korrekció rögzítése

#### 3. Időszak Zárás

1. **Zárás Előtti Felülvizsgálat**:
   - Az összes tranzakció könyvelésének ellenőrzése az időszakra
   - Az összes egyeztetés befejezésének biztosítása
   - Előzetes pénzügyi kimutatások felülvizsgálata

2. **Időszak Zárási Folyamat**:
   - Záró tételek feldolgozása, ha szükséges
   - Könyvelési időszak zártként jelölése
   - Tranzakciók zárolása a zárt időszakokban

### Jelentéskészítési Folyamatok

#### 1. Pénzügyi Kimutatás Generálás

1. **Mérleg Készítés**:
   - Mérleg generálása az időszak végéig
   - Összehasonlítás az előző időszakokkal
   - Ellenőrzés pontosság és teljesség szempontjából

2. **Eredménykimutatás Generálás**:
   - Eredménykimutatás generálása az időszakra
   - Összehasonlítás a költségvetéssel és az előző időszakokkal
   - Ellenőrzés pontosság és teljesség szempontjából

3. **Cash Flow Kimutatás**:
   - Cash flow kimutatás generálása
   - Cash flow elemzése működési, befektetési és finanszírozási tevékenységek szerint

#### 2. Vezetői és Elemzési Jelentések

1. **Költségvetés vs. Tényleges Jelentések**:
   - Költségvetés-összehasonlító jelentések generálása
   - Eltérések elemzése
   - Magyarázatok biztosítása a jelentős eltérésekhez

2. **Pénzügyi Arányszám Elemzés**:
   - Kulcsfontosságú pénzügyi arányszámok számítása
   - Időbeli trendelemzés
   - Összehasonlítás az iparági standardokkal

3. **Egyedi Jelentések**:
   - Speciális jelentések generálása szükség szerint
   - Osztály- vagy projekt-specifikus jelentések
   - Költséghely elemzés

#### 3. Külső Jelentéskészítés

1. **Adójelentések**:
   - Jelentések generálása adóbevallási célokra
   - Adatok exportálása a szükséges formátumokban
   - Adókövetelmények betartása

2. **Szabályozói Jelentéskészítés**:
   - Jelentések készítése szabályozó szervek számára
   - Jelentéskészítési standardok betartásának biztosítása
   - Beküldés a meghatározott időkereteken belül

3. **Részvényes/Érdekelt Fél Jelentéskészítés**:
   - Éves pénzügyi jelentések generálása
   - Befektetői prezentációk készítése
   - Szükséges közzétételek biztosítása

## Technikai Implementáció

A könyvelési rendszer implementációja a következőket használja:

- **Backend**: Django Django REST Framework-kel
- **Frontend**: Next.js shadcn/UI komponensekkel
- **Adatbázis**: Strukturált a referenciális integritás és az auditnyomvonalak fenntartására
- **API**: RESTful API integrációhoz más rendszermodulokkal
- **Biztonság**: Szerepalapú hozzáférés-vezérlés részletes jogosultságokkal

Minden könyvelési modul a következő elveket követi:

1. **Kettős Könyvelés**: Minden tranzakció fenntartja az egyensúlyt a tartozik és követel között
2. **Auditnyomvonalak**: Átfogó naplózása minden pénzügyi adat változásának
3. **Adatintegritás**: Validálási szabályok biztosítják a könyvelési konzisztenciát
4. **Munkafolyamat Állapotok**: A tranzakciók meghatározott munkafolyamatokat követnek a piszkozattól a könyveltig
5. **Felelősségi Körök Szétválasztása**: Az alapvető struktúrák elkülönülnek a tranzakció-feldolgozástól
