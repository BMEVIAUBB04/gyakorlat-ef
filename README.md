# EF Core

## Célkitűzés

EF Core programozás alapjainak elsajátítása CRUD műveletek írásán keresztül. LINQ-to-Entities lekérdezések írásának gyakorlása.

## Előfeltételek

A labor elvégzéséhez szükséges eszközök:

- Microsoft SQL Server (LocalDB vagy Express edition)
- Visual Studio 2019 .NET Core 3.1 SDK-val telepítve
- Adatbázis létrehozó script: [mssql.sql](https://raw.githubusercontent.com/BMEVIAUBB04/gyakorlat-mssql/master/mssql.sql)

Amit érdemes átnézned:

- EF Core előadás anyaga
- Microsoft SQL Server használata [segédlet](https://BMEVIAUBB04.github.io/gyakorlat-mssql/mssql-hasznalat.html) és [videó](https://youtu.be/gmY8reqSL7U)
- A használt adatbázis [sémája](https://BMEVIAUBB04.github.io/gyakorlat-mssql/sema.html)

Felkészülés ellenőrzése:

- A gyakorlatra való felkészüléshez használható [ezen kérdőív](https://forms.office.com/Pages/ResponsePage.aspx?id=q0g1anB1cUKRqFjaAGlwKf73d6yoiM1FuK24ZUEWuzFUOEVCUjFVVUdNS0pYWDI1TUhETkwwNThXMS4u).
- A gyakorlaton beugróként ugyanezen kérdőívet fogjuk használni, legalább 50% elérése szükséges.
  - Gyakorlatvezetői instrukció: a hallgató nyissa meg a kérdőívet és töltse ki. A válaszok megadása után az utolsó oldalon a "View results" gombra kattintva megtekinthető az eredmény és az elért pontszám. A hallgató ezen az oldalon álljon meg és mutassa meg eredményét a gyakorlatvezetőnek.

## Gyakorlat menete

Az első három feladatot a gyakorlatvezetővel együtt oldjuk meg. Az utolsó feladat önálló munka, amennyiben marad rá idő. A közös feladatok megoldásai megtalálhatóak az útmutatóban is. Előbb azonban próbáljuk magunk megoldani a feladatot!

## Feladat 0: Adatbázis létrehozása, ellenőrzése

Az adatbázis az adott géphez kötött, ezért nem biztos, hogy a korábban létrehozott adatbázis most is létezik. Ezért először ellenőrizzük, és ha nem találjuk, akkor hozzuk létre újra az adatbázist. (Ennek mikéntjét lásd a ["Tranzakciókezelés" gyakorlat anyagában](https://bmeviaubb04.github.io/gyakorlat-tranzakciok/).)

## Feladat 1: EF alap infrastruktúra kialakítása

Az EF, mint ORM eszköz használatához az alábbi összetevőre van szükség:
- **O**bjektummodel kódban
- **R**elációs modell az adatbázisban - ez kész
- Leképezés (**M**apping) az előbbi kettő között

---

Az itt található oktatási segédanyagok a BMEVIAUBB04 tárgy hallgatóinak készültek. Az anyagok oly módú felhasználása, amely a tárgy oktatásához nem szorosan kapcsolódik, csak a szerző(k) és a forrás megjelölésével történhet.

Az anyagok a tárgy keretében oktatott kontextusban értelmezhetőek. Az anyagokért egyéb felhasználás esetén a szerző(k) felelősséget nem vállalnak.
