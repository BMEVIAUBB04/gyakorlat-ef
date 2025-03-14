# Entity Framework Core

## Célkitűzés

ORM-alapú adatbáziskezelés alapjainak elsajátítása [CRUD-műveletek](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) írásán keresztül. _LINQ to Entities_-lekérdezések írásának gyakorlása.

## Előfeltételek

A labor elvégzéséhez szükséges eszközök:

- Microsoft SQL Server (LocalDB vagy Express edition)
- SQL Server Management Studio
- Visual Studio 2022 .NET 8 SDK-val telepítve
- Adatbázis létrehozó script: [mssql.sql](https://raw.githubusercontent.com/BMEVIAUBB04/gyakorlat-mssql/master/mssql.sql)

Amit érdemes átnézned:

- EF Core előadás anyaga
- SQL nyelv
- Microsoft SQL Server használata [segédlet](https://BMEVIAUBB04.github.io/gyakorlat-mssql/mssql-hasznalat.html)
- A használt adatbázis [sémája](https://BMEVIAUBB04.github.io/gyakorlat-mssql/sema.html)

Felkészülés ellenőrzése:

- A gyakorlatra való felkészüléshez használható [ezen kérdőív](https://forms.office.com/Pages/ResponsePage.aspx?id=q0g1anB1cUKRqFjaAGlwKfQJKflZc3NJoMvFrE-iHZ9UNkdMMk82UjFTSlI4TjQ2TElFUlBJVzRCQy4u).

## Gyakorlat menete

Az utolsó feladatot leszámítva a feladatokat a gyakorlatvezetővel együtt oldjuk meg. Az utolsó feladat önálló munka, amennyiben marad rá idő. A közös feladatok megoldásai megtalálhatóak az útmutatóban is. Előbb azonban próbáljuk magunk megoldani a feladatot!

## Feladat 0: Adatbázis létrehozása, ellenőrzése

Az adatbázis az adott géphez kötött, ezért nem biztos, hogy a korábban létrehozott adatbázis most is létezik. Ezért először ellenőrizzük, és ha nem találjuk, akkor hozzuk létre újra az adatbázist. (Ennek mikéntjét lásd a [_Tranzakciókezelés_ gyakorlat anyagában](https://bmeviaubb04.github.io/gyakorlat-tranzakciok/).)

## Feladat 1: EF-alapinfrastruktúra kialakítása

Az EF, mint ORM-eszköz használatához az alábbi összetevőkre van szükség:
- **o**bjektummodel a kódban
- **r**elációs modell az adatbázisban - ez már kész
- leképezés (**m**apping) az előbbi kettő között, szintén a kódban megadva
- maga az Entity Framework Core, mint komponens
- Entity Framework Core-kompatibilis adatbázisdriver
- adatbáziskapcsolódási adatok, _connection string_ formátumban

Az objektummodellt és a leképezést generáltatni fogjuk az adatbázis alapján - ez az ún. Reverse Engineering modellezési módszer.
1. Hozzunk létre Visual Studióban egy .NET 8 alapú C# nyelvű konzolalkalmazást. Ehhez válasszuk ki a C# nyelvű konzolalkalmazás sablonok közül a simát (ne a _.NET Framework_ jelölésűt). Futtatókörnyezetként válasszuk a .NET 8-at. Próbáljuk ki, hogy működik-e, kiíródik-e a "Hello, World!". .NET 8-ban nem kell `Main` függvényt írni, a belépési pontnak szánt kódfájlban (jelenleg a Program.cs-ben) írhatunk egyből utasításokat, nem kell se függvényt, se osztályt létrehoznunk.
2. Nyissuk meg a Package Manager Console-t (PMC) a _Tools :arrow_right: NuGet Package Manager :arrow_right: Package Manager Console_ menüponttal
3. Telepítsük fel az EF-kódgenerátor eszközt projektfüggőségként, illetve az SQL Server adatbázisdrivert. A generátoreszköznek már kapcsolódnia kell az adatbázishoz, amihez szüksége van a driverre. PMC-ben hajtsuk végre:

```powershell
Install-Package Microsoft.EntityFrameworkCore.Tools -Version 8.0.13 
Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 8.0.13
```
Ezek a csomagok függőségként magát az Entity Framework Core-t is telepítik.

4. Generáltassuk az adatbázismodellt az alábbi paranccsal. Paraméterei: kapcsolódási adatok (Connection), adatbázis driver neve (Provider), a projekten belül a generálandó fájlok könyvtára (OutputDir), a generálandó adatbáziskontextus neve (Context). 
[Dokumentáció itt](https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/powershell#scaffold-dbcontext). A connection stringben ne felejtsük el átírni az értékeket, pl. a Neptun-kódot kitölteni.
A connection string szerkezete SQL Server esetén: `kulcs=érték`-párok pontosvesszővel elválasztva. Nekünk most az alábbi adatok kellenek:
  - Server - a szerver neve
  - Database - adatbázis neve a szerveren belül (ez ennél a gyakorlatnál a neptun kód lesz)
  - Trusted_Connection=True - ez a Windows authentikációt takarja
  - NoPluralize - a táblák eléréshez többesszámosított property nevek generálódnának alapból, viszont ez csak angol nyelvű táblaneveknél működne jól, ezért ezt a funkciót kikapcsoljuk
  
A connection string szerkezete gyártónként eltér és elég sok paramétere lehet. Bővebben [itt](https://www.connectionstrings.com/).

Szintén PMC-ben:
```powershell
Scaffold-DbContext -Connection "Server=(localdb)\mssqllocaldb;Database=<neptun>;Trusted_Connection=True;" -Provider Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context ACMEShop -NoPluralize
```

A generált kódban figyeljük meg az alábbiakat:
  - az egyes táblabeli soroknak (rekordoknak) megfelelő C#-osztályokat (pl. Termek.cs), azon belül az oszlopoknak megfelelő egyszerű property-ket és a kapcsolódó tábláknak megfelelő más osztályok típusával rendelkező navigációs property-ket (pl. `Termek.Afa`). Ezek alkotják az objektummodellt.
  - a kontextusosztályt (pl. ACMEShop.cs), ami az általunk megadott névvel jött létre és magának az adatbázisnak a leképezése. Tartalmazza:
    - a tábláknak megfelelő `DbSet<RekordTipus>` propertyket, ezek a tábláknak felelenek meg
    - az adatbázisszintű konfigurációt (`OnConfiguring` függvény), pl. kapcsolódási adatokat, bár azt inkább külön konfig fájlban szoktuk tárolni, nem a kódban (erre van is figyelmeztetés a generált kódban)
    - a leképezések kódját az `OnModelCreating` függvényben. Minden rekordtípus minden property-jére megadja, hogy melyik tábla melyik oszlopára lépződik. Navigációs property-knél megadja a kapcsolat egyes résztvevőinek számosságát (1-N-es, N-N-es), valamint ha van, akkor az idegenkulcs-kényszert is.

Mindezzel minden összetevőt létrehoztunk az EF-alapinfrastruktúrához.

## Feladat 2: Hello, EF!

Írjuk meg az első EF Core-lekérdezésünket. Az SQL-mérés alapján itt is kérdezzük le az összes vevőt, majd írjuk ki azonosítójukat, nevüket és felhasználói nevüket is.

A Program.cs-be:

```csharp
using (var ctx = new ACMEShop()) // Context létrehozása. Using blokk, mert használat után illik az adatbáziskapcsolatot lezárni.
{
    foreach (var vevo in ctx.Vevo) // A ctx.Vevo a lekérdezésünk, ami a Vevo táblát reprezentálja.
    {
        Console.WriteLine($"{vevo.Nev} ({vevo.Id}, {vevo.Login})");
    }
}
```

Próbáljuk ki, és örvendezzünk, hogy milyen egyszerű és elegáns a kód. Semmilyen SQL stringet / SQL-kódot nem kellett írnunk.

## Feladat 3: Nyomkövetés (trace)

Minden olyan ORM használatakor, ahol az ORM állítja elő az SQL-t, rendkívül fontos, hogy lássuk, milyen SQL fut le az adatbázisszerveren. Nyomkövetni lehet az adatbázis oldalán (adatbáziseszközzel), illetve a programunk oldalán (nyomkövető komponenssel) is. Előbbire példa SQL Server esetén az SQL Server Profiler [Trace eszköze](https://docs.microsoft.com/en-us/sql/tools/sql-server-profiler/create-a-trace-sql-server-profiler). Mi most az utóbbi irányt követjük. A context `OnConfiguring` függvényébe:

```csharp
optionsBuilder.UseSqlServer("connection string") //ez a rész maradjon változatlan
	.LogTo(message => System.Diagnostics.Debug.WriteLine(message),LogLevel.Information); //ez a rész ékelődjön be a ; elé
```

A debug kimenet az Output ablakra van kötve - viszont csak akkor, ha a Visual Studio debugger csatlakoztatva van. Ezért fontos, hogy ha látni akarjuk az EF naplókat, akkor **Debug módban (zöld nyíl, `F5` billentyű) futtassuk az alkalmazást**.

Próbáljuk ki, az Output ablak alján meg kell jelennie egy hasonló SQL-nek:

> info: [időbélyeg] RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command) 
> 
> &nbsp;&nbsp;&nbsp;&nbsp;Executed DbCommand (26ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
>      
> &nbsp;&nbsp;&nbsp;&nbsp;SELECT [v].[ID], [v].[Email], [v].[Jelszo], [v].[KozpontiTelephely], [v].[Login], [v].[Nev], [v].[Szamlaszam]
>      
> &nbsp;&nbsp;&nbsp;&nbsp;FROM [Vevo] AS [v]

Ha hibakeresés miatt részletesebb naplóra van szükség, akkor a `LogLevel` típusú paramétert (ideiglenesen) állítsuk `Loglevel.Debug`-ra.

## Feladat 4: CRUD-műveletek (közös)

Minden részfeladatot a `using` blokkon belül írjunk. Ha zavar a többi részfeladat kódja, kommentezzük ki őket. Minden részfeladatnál ellenőrizzük a kiírt eredményt az adatbázisadatok alapján is (pl. Management Studio-ban) és figyeljük meg a generált SQL-t is az Output ablak alján.

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
   foreach (string? tn in termeknevek)
   {
     Console.WriteLine(tn);
   }
   ```
   
   Használjuk a navigációs property-ket és a `System.Linq` névtérben található LINQ operátorokat. Kiindulhatunk a termék, illetve a megrendeléstétel táblából (`DbSet`-ből) is. Érdemes kipróbálni mind a két változatot - nem ugyanolyan SQL generálódik.
	
   A `string` utáni kérdőjel nem kötelező, azzal jelezzük, hogy az érték lehet `null` is. Hiszen a terméknév nem kötelező adat az adatbázisban.
   
   </details>
   

1. Hány nem teljesített megrendelésünk van (a státusz alapján)?

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   Console.WriteLine(ctx.Megrendeles.Count(mr => mr.Statusz.Nev != "Kiszállítva"));
   ```

   Ez ujjgyakorlat navigációs propertyt használva.

   </details>
   
1. Melyek azok a fizetési módok, amiket soha nem választottak a megrendelőink?

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   var fmList=ctx.FizetesMod.Where(fm => !fm.Megrendeles.Any()).Select(fm=>fm.Mod);
   foreach (string? fm in fmList)
   {
       Console.WriteLine(fm);
   }
   ```
   </details>

1. Rögzítsünk egy új vevőt! Írjuk ki az újonnan létrejött rekord kulcsát!

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   var ujvevo = new Vevo 
   { 
      Nev = "Teszt Elek",
      Login = "t.elek",
      Jelszo = "******",
      Email = "t.elek@email.com"
   };
   ctx.Add(ujvevo);
   ctx.SaveChanges();
   Console.WriteLine(ujvevo.Id);

   ```

   Az adatváltozásokat (pl. beszúrások) a kontext gyűjti. Az `Add` sor után a kontext nyilvántartja az új példányunkat új rekordként, de még nem csinál az adatbázison semmit. A `SaveChanges` a felgyűlt módosításokat egy füst alatt érvényesíti az adatbázison és vissza is frissíti az érintett C# oldali objektumokat - így a `SaveChanges` után az új vevő azonosítója ki lesz töltve. Debuggerrel láthatjuk is, ahogy a `SaveChanges` soron átjutva kitöltődik az azonosító. Ellenőrizzük a változást az adatbázisban.
   
   A megoldás után ezt a részt kommentezzük ki, ne szúrjunk be minden kipróbálásnál új sort.
   </details>

1. A kategóriák között hibásan szerepel a _Fajáték_ kategórianév. Javítsuk át a kategória nevét *Fakockák*ra!

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   var fakocka = ctx.Kategoria.Single(k => k.Nev == "Fajáték");
   fakocka.Nev = "Fakockák";
   ctx.SaveChanges();
   ```
   Első lépésként le kell kérdeznünk a módosítandó entitást, elvégezni a módosítást, amit a kontext megint csak nyilvántart, végül érvényesíteni a módosításokat. A Watch ablakban figyeljük meg a kontext által nyilvántartott állapot változását a `ctx.Entry(fakocka).State` kifejezés figyelésével. Ellenőrizzük a változást az adatbázisban.
   
    A megoldás után ezt a részt kommentezzük ki, ha nincs az adatbázisban fajáték, a `Single` kivételt fog dobni, mivel pontosan egy darab rekordot vár.
    
    Sajnos ez a művelet így két SQL parancs (egy `SELECT` és egy `UPDATE`), jelenleg az EF Core nem tudja ezt egy utasításként megoldani, bár vannak hozzá [kiterjesztések](https://entityframework-extensions.net/update-from-query) erre a célra.
   </details>

1. Mely kategóriákban van a legtöbb termék? Írjuk ki ezen kategóriák nevét.

   <details><summary markdown="span">Megoldás</summary>

   ```csharp
   //naív megoldás, csak egy eredmény
   Console.WriteLine(ctx.Kategoria.OrderByDescending(k => k.Termek.Count())
				  .Select(k=>k.Nev).First());
   
   //navigációs propertyvel - kivételt dob!   
   var maxkat=ctx.Kategoria
   	.Where(k => k.Termek.Count == ctx.Kategoria.Max(kk=>kk.Termek.Count))
	.Select(k=>k.Nev);
   foreach (var k in maxkat)
   {
     Console.WriteLine(k);
   }
   
   //navigációs propertyvel - működik, de két lekérdezés
   var maxkat = ctx.Kategoria.Select(kk => kk.Termek.Count)
    .AsEnumerable() // ez elsüti a lekérdezést, minden kategória termékszámát lekérdezzük
    .Max();
   var ktg = ctx.Kategoria.Where(k => k.Termek.Count == maxkat).Select(k=>k.Nev);
   foreach (var k in ktg)
   {
      Console.WriteLine(k);
   }

   //másik megközelítés: azokat a kategóriákat keressük, amiknél nincs több termékes kategória
   //egy lekérdezés és működik is
   var maxkat= ctx.Kategoria
	.Where(k1=>!ctx.Kategoria.Any(k2=> k2.Termek.Count > k1.Termek.Count))
        .Select(k=>k.Nev);
   foreach (var k in maxkat)
   {
      Console.WriteLine(k);
   }      
   ```
   Sajnos elég nehéz olyan lekérdezést összerakni C#-ban, ami az SQL összerakásánál ne dobna kivételt vagy ne több lekérdezésből állna :cold_sweat:. Nem minden logikailag helyes, forduló LINQ kód alakítható SQL-lé, ráadásul ez csak futási időben derül ki. Érdemes ilyenkor megírni a sima SQL-t és ahhoz hasonlóan összerakni a C# kódot, plusz minden lekérdezést legalább egyszer **erősen** ajánlott kipróbálni futás közben is.

   </details>
   
1. Az egyes telephelyekre hány rendelés volt eddig? Írjuk ki a telephelyek azonosítóját és a rendelések számát!
   <details><summary markdown="span">Megoldás</summary>

   ```csharp
    var tmegrList = ctx.Telephely.Select(t => new { t.Id, MegrendelesSzam = t.Megrendeles.Count });
    foreach (var tmegr in tmegrList)
    {
        Console.WriteLine($"{tmegr.Id}-{tmegr.MegrendelesSzam}");
    }      
   ```

   Gyakran alkalmazzuk a `Select` operátorban az anonim típusokat. Ezekben a típusokban mi mondjuk meg, hogy milyen nevű property-k legyenek és azok hogyan töltődjelenek fel. Bár a típus nevét nem ismerjük (anonim), de tudunk hivatkozni a property-jeire a megadott névvel.

   </details>



## Feladat 5: Lekérdezések (önálló)

1. Mely termék(ek) áfakulcsa 15%-os? Írjuk ki ezen termék(ek) nevét!
1. Melyik város(ok)ba kérték a legtöbb rendelést? (**Nehéz!** :scream:)
   <details><summary markdown="span">Tipp</summary>
	
   Lásd a közös feladatokból a hasonlót. Egy lehetséges megoldás, ha először a megrendeléseket csoportosítjuk a kapcsolódó város neve szerint a `GroupBy` operátorral, majd vesszük minden csoport kulcsát és a csoport számosságát. Ezután ezen allekérdezésből kiindulva kell kikeresni, hogy melyek azok az allekérdezés elemek, amelyeknél nincsenek nagyobb számosságú más allekérdezés elem.
	
   </details>
3. Melyek azok a vevők, akik legalább 2-szer rendeltek már? Írjuk ki ezen vevők nevét és hogy hányszor rendeltek! (**Nehéz!** :scream:).
   <details><summary markdown="span">Tipp</summary>
	
   Így használhatjuk a `SelectMany` LINQ operátort egy `v` vevő összes rendelésének összeszedésére: `v.Telephely.SelectMany(t => t.Megrendeles)`. A `SelectMany`-t akkor használjuk, ha egy kollekció minden elemhez egy kollekciót rendelünk, de az eredményt nem listák listájaként szeretnénk megkapni, hanem csak sima kilapított listaként
	
   </details>
5. Mely számlá(k)nál nem egyezik meg a kiállítás és teljesítés dátuma? Írjuk ki ezen számla/számlák azonosítóját!
6. Írjuk ki a 2008. februári rendelések azonosítóját és ezen rendelések dátumát!
7. Írjuk ki azon rendelés(ek) azonosítóját, dátumát és határidejét, amely(ek)nél a határidő 5 napnál szűkebb a rendelés dátumához képest! (**Nehéz!** :scream:, TIPP: `EF.Functions.DateDiffDay(<dátum>,<határidő>)` megadja a `dátum` és `határidő` között eltelt napok számát)
8. Hány vevőnek van gmailes e-mail címe?
9. Melyik vevőknek van egynél több telephelye? Írjuk ki ezen vevők nevét és telephelyeik számát!
10. Mely vevő(k) adták le a legtöbb tételből álló rendelést? (Több ilyen is lehet!) Írjuk ki ezen vevők nevét! (**Nehéz!** :scream:)
    <details><summary markdown="span">Tipp</summary>
	
    Lásd a közös feladatokból a hasonlót. Egy lehetséges megoldás, ha először minden megrendeléshez kiszámoljuk a kapcsolódó vevő nevét és a megrendelés tételeinek számát. Ezután ezen allekérdezésből kiindulva kell kikeresni, hogy melyek azok az allekérdezés elemek, emelyeknél nincsenek nagyobb tételszámú más allekérdezés elem.
	
    </details>
---

Az itt található oktatási segédanyagok a BMEVIAUBB04 tárgy hallgatóinak készültek. Az anyagok oly módú felhasználása, amely a tárgy oktatásához nem szorosan kapcsolódik, csak a szerző(k) és a forrás megjelölésével történhet.

Az anyagok a tárgy keretében oktatott kontextusban értelmezhetőek. Az anyagokért egyéb felhasználás esetén a szerző(k) felelősséget nem vállalnak.
