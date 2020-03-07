# EF Core

## Célkitűzés

ORM alapú adatbáziskezelés alapjainak elsajátítása CRUD műveletek írásán keresztül. LINQ-to-Entities lekérdezések írásának gyakorlása.

## Előfeltételek

A labor elvégzéséhez szükséges eszközök:

- Microsoft SQL Server (LocalDB vagy Express edition)
- SQL Server Management Studio
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

## Feladat 1: EF alapinfrastruktúra kialakítása

Az EF, mint ORM eszköz használatához az alábbi összetevőkre van szükség:
- **O**bjektummodel kódban
- **R**elációs modell az adatbázisban - ez már kész
- Leképezés (**M**apping) az előbbi kettő között, szintén kódban megadva
- maga az Entity Framework, mint komponens
- Entity Framework kompatibilis adatbázis driver
- adatbázis kapcsolódási adatok, connection string formátumban

Az objektummodellt és a leképezést generáltatni fogjuk az adatbázis alapján.
1. Hozzunk létre Visual Studio-ban egy .NET Core alapú C# nyelvű konzolalkalmazást. Próbáljuk ki, hogy működik-e, kiíródik-e a "Hello World".
2. Nyissuk meg a Package Manager Console-t (PMC) a Tools -> NuGet Package Manager -> Package Manager Console menüponttal
3. Telepítsük fel az EF kódgenerátor eszközt projektfüggőségként, illetve az SQL Server adatbázis drivert. A generátor eszköznek már kapcsolódnia kell az adatbázishoz, amihez szüksége van a driverre. PMC-ben hajtsuk végre:

```powershell
Install-Package Microsoft.EntityFrameworkCore.Tools
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```
Ezek a csomagok függőségként magát az Entity Framework Core-t is telepítik.

4. Generáltassuk az adatbázismodellt az alábbi paranccsal. Paraméterei: kapcsolódási adatok (Connection), adatbázis driver neve (Provider), a projekten belül a generálandó fájlok könyvtára (OutputDir), a generálandó adatbáziskontextus neve (Context). Bővebben [itt](https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/powershell#scaffold-dbcontext). A connection string-ben ne felejtsük el átírni az értékeket, pl. a neptun kódot kitölteni.
A connection string szerkezete SQL Server esetén: kulcs=érték párok pontosvesszővel elválasztva. Nekünk most az alábbi adatok kellenek:
  - Server - a szerver neve
  - Database - adatbázis neve a szerveren belül (ez ennél a gyakorlatnál a neptun kód lesz)
  - Trusted_Connection=True - ez a Windows authentikációt takarja.
A connection string szerkezete gyártónként eltér és elég sok paramétere lehet. Bővebben [itt](https://www.connectionstrings.com/)

```powershell
Scaffold-DbContext -Connection "Server=(localdb)\mssqllocaldb;Database=<neptun>;Trusted_Connection=True;" -Provider Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context ACMEShop
```

A generált kódban figyeljük meg az alábbiakat:
  - az egyes táblabeli soroknak (rekordoknak) megfelelő C# osztályokat (pl. Termek.cs), azon belül az oszlopoknak megfelelő egyszerű property-ket és a kapcsolódó tábláknak megfelelő más osztályok típusával rendelkező navigációs property-ket (pl. `Termek.Afa`). Ezek alkotják az objektummodellt.
  - a kontextusosztályt (pl. ACMEShop.cs), ami az általunk megadott névvel jött létre és magának az adatbázisnak a leképezése. Tartalmazza:
    - a tábláknak megfelelő `DbSet<RekordTipus>` property-ket, ezek a tábláknak felelenek meg
    - az adatbázis szintű konfigurációt (`OnConfiguring` függvény), pl. kapcsolódási adatokat, bár azt inkább külön konfig fájlban szoktuk tárolni
    - a leképezések kódját az `OnModelCreating` függvényben. Minden rekordtípus minden property-jére megadja, hogy melyik tábla melyik oszlopára lépződik. Navigációs property-knél megadja a kapcsolat egyes résztvevőinek számosságát (1-N-es, N-N-es), valamint ha van, akkor az idegen kulcs kényszert is.

Mindezzel minden összetevőt létrehoztunk az EF alapinfrastruktúrához.

## Feladat 2: Hello EF!

Írjuk meg az első EF Core lekérdezésünket. Az SQL mérés alapján itt is kérdezzük le az összes vevőt, majd írjuk ki azonosítójukat, nevüket és felhasználói nevüket is.

A `Main` függvénybe:

```csharp
using (ACMEShop ctx = new ACMEShop()) // kontext létrehozása. Using blokk, mert használat után illik az adatbáziskapcsolatot lezárni.
{
    foreach (Vevo vevo in ctx.Vevo) //a ctx.Vevo a lekérdezésünk, a Vevo táblát reprezentálja
    {
        Console.WriteLine($"{vevo.Nev} ({vevo.Id}, {vevo.Login})");
    }
}
```

Próbáljuk ki, örvendezzünk, hogy milyen egyszerű és elegáns a kód. Semmilyen SQL stringet/ SQL kódot nem kellett írnunk.

## Feladat 3: Nyomkövetés (trace)

Minden olyan ORM használatakor, ahol az ORM állítja elő az SQL-t, elementárisan fontos, hogy lássuk, milyen SQL fut le az adatbázisszerveren. Nyomkövetni lehet az adatbázis oldalán (adatbázis eszközzel), illetve a programunk oldalán (nyomkövető komponenssel) is. Előbbire példa SQL Server esetén az SQL Server Profiler [Trace eszköze](https://docs.microsoft.com/en-us/sql/tools/sql-server-profiler/create-a-trace-sql-server-profiler). Mi most az utóbbit alkalmazzuk. Adjunk hozzá a projektünkhöz egy általános naplózó komponenst, ami a _Debug_ kimenetre naplóz.

A PMC segítségével telepítsük a naplózó komponenst:

```powershell
Install-Package Microsoft.Extensions.Logging.Debug
```

A `Program` osztályba, függvényen kívül:

```csharp
 public static readonly ILoggerFactory DaLogger
            = LoggerFactory.Create(builder => { builder.AddDebug(); });
```

Ezzel létrehoztunk egy debug kimenetre naplózó infrastruktúrát (pontosabban ennek gyártóját), ezt adjuk meg az EF kotextusnak, mely beépítetten naplóz (ha van hova). A kontext `OnConfiguring` függvényébe:

```csharp
optionsBuilder //ez maradjon meg változatlanul
  .UseLoggerFactory(Program.DaLogger) //ez a rész ékelődjön be
    .UseSqlServer(""); //ez a rész is maradjon változatlan
```



---

Az itt található oktatási segédanyagok a BMEVIAUBB04 tárgy hallgatóinak készültek. Az anyagok oly módú felhasználása, amely a tárgy oktatásához nem szorosan kapcsolódik, csak a szerző(k) és a forrás megjelölésével történhet.

Az anyagok a tárgy keretében oktatott kontextusban értelmezhetőek. Az anyagokért egyéb felhasználás esetén a szerző(k) felelősséget nem vállalnak.
