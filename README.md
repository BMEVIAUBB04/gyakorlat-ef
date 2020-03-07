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
- **o**bjektummodel kódban
- **r**elációs modell az adatbázisban - ez már kész
- leképezés (**m**apping) az előbbi kettő között, szintén kódban megadva
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

Minden olyan ORM használatakor, ahol az ORM állítja elő az SQL-t, elementárisan fontos, hogy lássuk, milyen SQL fut le az adatbázisszerveren. Nyomkövetni lehet az adatbázis oldalán (adatbázis eszközzel), illetve a programunk oldalán (nyomkövető komponenssel) is. Előbbire példa SQL Server esetén az SQL Server Profiler [Trace eszköze](https://docs.microsoft.com/en-us/sql/tools/sql-server-profiler/create-a-trace-sql-server-profiler). Mi most az utóbbi irányt követjük. Adjunk hozzá a projektünkhöz egy általános naplózó komponenst, ami a _Debug_ kimenetre naplóz.

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
    .UseSqlServer("connection string"); //ez a rész is maradjon változatlan
```

A debug kimenet az Output ablakra van kötve - viszont csak akkor, ha a Visual Studio debugger csatlakoztatva van. Ezért fontos, hogy ha látni akarjuk az EF naplókat, akkor **Debug módban (zöld nyíl, F5 billentyű) futtassuk az alkalmazást**.

Próbáljuk ki, az Output ablak alján meg kell jelennie egy hasonló SQL-nek:

> Microsoft.EntityFrameworkCore.Database.Command: Information: Executed DbCommand (42ms) [Parameters=[], CommandType='Text', CommandTimeout='30']

>SELECT [v].[ID], [v].[Email], [v].[Jelszo], [v].[KozpontiTelephely], [v].[Login], [v].[Nev], [v].[Szamlaszam] FROM [Vevo] AS [v]

## Feladat 4: CRUD műveletek (közös)

Minden részfeladatot a `using` blokkon belül írjunk. Ha zavar a többi részfeladat kódja, kommentezzük ki őket. Minden részfeladatnál ellenőrizzük a kiírt eredményt az adatbázisadatok alapján is (pl. Management Studio-ban) és figyeljük meg a generált SQL-t is.

1. Kérdezze le az összes vevőt, írja ki a nevüket, azonosítójukat és felhasználónevüket! - Ezt már megoldottuk!

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   foreach (Vevo vevo in ctx.Vevo)
   {
     Console.WriteLine($"{vevo.Nev} ({vevo.Id}, {vevo.Login})");
   }
   ```
   </details>
   
1. Listázza ki, hogy eddig milyen nevű termékeket rendeltek!

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   //bármelyik jó a két alábbi lekérdezés közül
   var termeknevek = ctx.MegrendelesTetel.Select(mt => mt.Termek.Nev).Distinct(); //MegrendelesTetel -> Termek
   //var termeknevek = ctx.Termek.Where(mt => mt.MegrendelesTetel.Any()).Select(t => t.Nev); //Termek -> MegrendelesTetel
   foreach (string tn in termeknevek)
   {
     Console.WriteLine(tn);
   }
   ```
   
   Használjuk a navigációs property-ket és a `System.Linq` névtérben található LINQ operátorokat. Kiindulhatunk a termék, illetve a megrendeléstétel táblából (`DbSet`-ből) is. Érdemes kipróbálni mind a két változatot - nem ugyanolyan SQL generálódik.
   
   </details>
   

1. Hány nem teljesített megrendelésünk van (a státusz alapján)?

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   Console.WriteLine(ctx.Megrendeles.Count(mr => mr.Statusz.Nev != "Kiszállítva"));
   ```

   Ez ujjgyakorlat navigációs propertyt használva.

   </details>
   
1. Melyek azok a fizetési módok, amit soha nem választottak a megrendelőink?

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   var fmList=ctx.FizetesMod.Where(fm => !fm.Megrendeles.Any()).Select(fm=>fm.Mod);
   foreach (string fm in fmList)
   {
       Console.WriteLine(fm);
   }
   ```

   A megoldás kulcsa az `outer join`, aminek köszönhetően láthatjuk, mely fizetési mód rekordhoz _nem_ tartozik egyetlen megrendelés se.

   </details>

1. Rögzítsünk be egy új vevőt! Kérdezzük le az újonnan létrejött rekord kulcsát!

   <details><summary markdown="span">Megoldás</summary>

   ```sql
   insert into Vevo(Nev, Login, Jelszo, Email)
   values ('Teszt Elek', 't.elek', '********', 't.elek@email.com')

   select @@IDENTITY
   ```

   Az `insert` után javasolt kiírni az oszlopneveket az egyértelműség végett, bár nem kötelező. Vegyük észre, hogy az ID oszlopnak nem adunk értéket, mert azt a tábla definíciójakor meghatározva a szerver adja automatikusan. Ezért kell utána lekérdeznünk, hogy tudjuk, milyen ID-t adott.

   </details>

1. A kategóriák között hibásan szerepel az _Fajáték_ kategória név. Javítsuk át a kategória nevét *Fakockák*ra!

   <details><summary markdown="span">Megoldás</summary>

   ```sql
   update Kategoria
   set Nev = 'Fakockák'
   where Nev = 'Fajáték'
   ```

   </details>

1. Melyik termék kategóriában van a legtöbb termék?

   <details><summary markdown="span">Megoldás</summary>

   ```sql
   select top 1 Nev, (select count(*) from Termek where Termek.KategoriaID = k.ID) as db
   from Kategoria k
   order by db desc
   ```

   A kérdésre több alternatív lekérdezés is eszünkbe juthat. Ez csak egyike a lehetséges megoldásoknak. Itt láthatunk példát az allekérdezésre (subquery) is. Viszont nem ad helyes megoldást akkor, ha több olyan kategória is van, amely ugyanannyi, maximális számú terméket tartalmaz, mert csak az elsőt ilyen kategóriát adja vissza A teljesen helyes megoldás ehelyett:

   ```sql
   select k.Nev 
   from Kategoria k
     join Termek t on t.KategoriaID = k.ID
   group by k.id, k.Nev
   having count(t.id) = 
     (select max(darab) from
       (
	    select count(t.id) AS darab
        from Kategoria k join Termek t on t.KategoriaID = k.ID
		group by k.id, k.Nev
	  ) AS darabszamok
    )
    ```

   

   </details>


---

Az itt található oktatási segédanyagok a BMEVIAUBB04 tárgy hallgatóinak készültek. Az anyagok oly módú felhasználása, amely a tárgy oktatásához nem szorosan kapcsolódik, csak a szerző(k) és a forrás megjelölésével történhet.

Az anyagok a tárgy keretében oktatott kontextusban értelmezhetőek. Az anyagokért egyéb felhasználás esetén a szerző(k) felelősséget nem vállalnak.
