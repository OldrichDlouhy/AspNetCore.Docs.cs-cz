---
title: Přidání modelu do aplikace v ASP.NET Core Razor Pages
author: rick-anderson
description: Objevte, jak přidat třídy pro správu filmy v databázi pomocí Entity Framework Core (jádro EF).
ms.author: riande
ms.date: 11/05/2019
uid: tutorials/razor-pages/model
ms.openlocfilehash: f2c9c2fc8112ef8a1a5afdbe448de6319c43521d
ms.sourcegitcommit: 6628cd23793b66e4ce88788db641a5bbf470c3c1
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/07/2019
ms.locfileid: "73761220"
---
# <a name="add-a-model-to-a-razor-pages-app-in-aspnet-core"></a>Přidání modelu do aplikace v ASP.NET Core Razor Pages

Podle [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

V této části jsou přidány třídy pro správu filmů v [databázi SQLite](https://www.sqlite.org/index.html)pro různé platformy. Aplikace vytvořené pomocí šablony ASP.NET Core používají databázi SQLite. Třídy modelu aplikace se používají s [Entity Framework Core (EF Core)](/ef/core) ([poskytovatel databáze SQLite EF Core](/ef/core/providers/sqlite)) pro práci s databází. EF Core je rozhraní pro mapování relačních objektů (ORM), které zjednodušuje přístup k datům.

Modelu třídy jsou označovány jako POCO třídy (od "prostý staré CLR objekty"), protože nemají žádné závislosti na EF Core. Definují vlastnosti data, která jsou uložena v databázi.

[!INCLUDE[View or download sample code](~/includes/rp/download.md)]

## <a name="add-a-data-model"></a>Přidání datového modelu

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Klikněte pravým tlačítkem myši **RazorPagesMovie** Projekt > **přidat** > **novou složku**. Název složky *modely*.

Klikněte pravým tlačítkem myši *modely* složky. Vyberte **přidat** > **třídy**. Název třídy **filmu**.

[!INCLUDE [model 1b](~/includes/RP/model1b.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Přidat složku s názvem *modely*.
* Přidat třídu *modely* složku s názvem *Movie.cs*.

[!INCLUDE [model 1b](~/includes/RP/model1b.md)]

[!INCLUDE [model 2](~/includes/RP/model2.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

* V Průzkumníku řešení klikněte pravým tlačítkem myši **RazorPagesMovie** projektu a pak vyberte **přidat** > **novou složku**. Název složky *modely*.
* Klikněte pravým tlačítkem na složku *modely* a pak vyberte **Přidat** > **nový soubor**.
* V **nový soubor** dialogové okno:

  * Vyberte **Obecné** v levém podokně.
  * V prostředním podokně vyberte **prázdnou třídu** .
  * Název třídy **film** a vyberte **nový**.

[!INCLUDE [model 1b](~/includes/RP/model1b.md)]

[!INCLUDE [model 2](~/includes/RP/model2.md)]

---

Sestavte projekt a ověřte, že nejsou žádné chyby během kompilace.

## <a name="scaffold-the-movie-model"></a>Vygenerované uživatelské rozhraní Video modelu

V této části je automaticky generovaný model video. To znamená vytvoří nástroj pro generování uživatelského rozhraní stránky pro operace vytvoření, čtení, aktualizace a odstranění (CRUD) pro model video.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Vytvoření *stránek/filmy* složky:

* Klikněte pravým tlačítkem na složku *stránky* > **Přidat** > **novou složku**.
* Název složky *filmy*

Klikněte pravým tlačítkem na složku *stránky nebo filmy* > **Přidat** > **Nová vygenerovaná položka**.

![Image z předchozích kroků.](model/_static/sca.png)

V dialogovém okně **Přidat generování uživatelského rozhraní** vyberte **Razor Pages pomocí Entity Framework (CRUD)** > **Přidat**.

![Image z předchozích kroků.](model/_static/add_scaffold.png)

Dokončení **přidat stránky Razor pomocí Entity Frameworku (CRUD)** dialogové okno:

* V **třída modelu** rozevírací seznam, vyberte **Movie (RazorPagesMovie.Models)** .
* V řádku **Třída kontextu dat** vyberte symbol **+** (plus) a změňte vygenerovaný název z RazorPagesMovie. **Modely**. RazorPagesMovieContext na RazorPagesMovie. **Data**. RazorPagesMovieContext. [Tato změna](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) se nevyžaduje. Vytvoří třídu kontextu databáze se správným oborem názvů.
* Vyberte **Přidat**.

![Image z předchozích kroků.](model/_static/3/arp.png)

*Appsettings.json* souboru aktualizovali připojovací řetězec použitý pro připojení k místní databázi.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* Otevřete okno příkazového řádku v adresáři projektu (adresář, který obsahuje *Program.cs*, *Startup.cs*, a *.csproj* soubory).
* Nainstalujte nástroj pro generování uživatelského rozhraní:

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* **Pro Windows**: spusťte následující příkaz:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **Pro macOS a Linux**: spusťte následující příkaz:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold gen params](~/includes/RP/model4.md)]

[!INCLUDE [use SQL Server in production](~/includes/RP/sqlitedev.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

* Otevřete okno příkazového řádku v adresáři projektu (adresář, který obsahuje *Program.cs*, *Startup.cs*, a *.csproj* soubory).
* Nainstalujte nástroj pro generování uživatelského rozhraní:

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* Spusťte následující příkaz:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold gen params](~/includes/RP/model4.md)]

[!INCLUDE [use SQL Server in production](~/includes/RP/sqlitedev.md)]

---

### <a name="files-created"></a>Soubory vytvořené

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Vygenerované uživatelské rozhraní proces vytvoří a aktualizuje následující soubory:

* *Stránky/filmy*: vytvoření, odstranění, podrobností, úpravy a Index.
* *Data/RazorPagesMovieContext.cs*

### <a name="updated"></a>Datum aktualizace

* *Startup.cs*

Vytvořený a aktualizované soubory jsou vysvětlené v následující části.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

Proces generování uživatelského rozhraní vytvoří následující soubory:

* *Stránky/filmy*: vytvoření, odstranění, podrobností, úpravy a Index.

Vytvořené soubory jsou vysvětleny v další části.

---

<a name="pmc"></a>

## <a name="initial-migration"></a>Počáteční migraci

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

V této části konzoly Správce balíčků (PMC) umožňuje:

* Přidáte počáteční migraci.
* Aktualizujte počáteční migraci databáze.

V nabídce **nástroje** vyberte **správce balíčků NuGet** > **konzolu Správce balíčků**.

  ![PMC nabídky](../first-mvc-app/adding-model/_static/pmc.png)

V konzole PMC zadejte následující příkazy:

```PMC
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE [initial migration](~/includes/RP/model3.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

[!INCLUDE [initial migration](~/includes/RP/model3.md)]

---

Předchozí příkazy generují následující upozornění: pro desetinný sloupec ' Price ' pro typ entity ' video ' nebyl zadán žádný typ ' Price '. To způsobí, že se hodnoty tiše zkrátí, pokud se nevejdou do výchozí přesnosti a rozsahu. Explicitně zadejte typ sloupce SQL serveru, který může obsahovat všechny hodnoty pomocí ' HasColumnType () '.

Toto upozornění můžete ignorovat, bude opraveno v pozdějším kurzu.

Příkaz migrace generuje kód pro vytvoření počátečního schématu databáze. Schéma je založené na modelu určeném v `DbContext`. `InitialCreate` Argument se používá k pojmenování migrace. Můžete použít libovolný název, ale podle konvence vybraný název, který popisuje migraci.

Příkaz `update` spustí metodu `Up` v migracích, které nebyly aplikovány. V takovém případě `update` spustí metodu `Up` v části *migrace/\<časového razítka > souboru _InitialCreate. cs* , který vytvoří databázi.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>Prozkoumání kontextu registrovaný pomocí vkládání závislostí

ASP.NET Core využívá rozhraní [injektáž závislostí](xref:fundamentals/dependency-injection). Služby (například kontext EF Core databáze) jsou registrované pomocí vkládání závislostí při spuštění aplikace. Komponenty, které vyžadují tyto služby (například stránky Razor) jsou k dispozici tyto služby prostřednictvím parametry konstruktoru. Později v tomto kurzu se zobrazí kód konstruktor, který získá instanci kontext databáze.

Nástroj pro generování uživatelského rozhraní automaticky vytvoří kontext databáze a zaregistrovaného kontejneru pro vkládání závislostí.

Zkontrolujte `Startup.ConfigureServices` metody. Zvýrazněný řádek byl přidán modulem scaffolder:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

`RazorPagesMovieContext` Souřadnice funkce EF Core (vytvoření, čtení, aktualizace, odstranění atd.) pro `Movie` modelu. Kontext dat (`RazorPagesMovieContext`) je odvozen z [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext). Kontext dat určuje entit, které jsou zahrnuty v datovém modelu.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Data/RazorPagesMovieContext.cs)]

Předchozí kód vytvoří vlastnost [`DbSet<Movie>`](/dotnet/api/microsoft.entityframeworkcore.dbset-1) pro sadu entit. Terminologie Entity Framework obvykle sadu entit odpovídá databázové tabulky. Entita odpovídající řádek v tabulce.

Název připojovacího řetězce je předán v rámci voláním metody na [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) objektu. Pro místní vývoj [ASP.NET Core konfigurační systém](xref:fundamentals/configuration/index) načte připojovací řetězec z *appsettings.json* souboru.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Zkontrolujte `Up` metody.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

Zkontrolujte `Up` metody.

---

<a name="test"></a>

### <a name="test-the-app"></a>Testování aplikace

* Spusťte aplikaci a připojit `/Movies` na adresu URL v prohlížeči (`http://localhost:port/movies`).

Pokud dojde k chybě:

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

Je provedena [kroku migrace](#pmc).

* Test **vytvořit** odkaz.

  ![Vytvoření stránky](model/_static/conan.png)

  > [!NOTE]
  > Není možné zadat desetinné čárky v `Price` pole. Pro podporu [k ověřování jQuery](https://jqueryvalidation.org/) pro neanglická národní prostředí, které používají čárkou (",") desetinné čárky a USA retweetovat neanglické formáty kalendářního data, aplikace musí být globalizována. Globalizace pokyny najdete v tématu [tento problém Githubu](https://github.com/aspnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).

* Test **upravit**, **podrobnosti**, a **odstranit** odkazy.

V dalším kurzu vysvětluje souborů vytvořených databázovým generování uživatelského rozhraní.

## <a name="additional-resources"></a>Další materiály a zdroje informací

> [!div class="step-by-step"]
> [Předchozí: Začínáme](xref:tutorials/razor-pages/razor-pages-start)
> [Další: generované uživatelské rozhraní pro stránky Razor](xref:tutorials/razor-pages/page)

::: moniker-end

<!--  ::: moniker previous version   -->
::: moniker range="< aspnetcore-3.0"

V této části jsou přidány třídy pro správu filmů v [databázi SQLite](https://www.sqlite.org/index.html)pro různé platformy. Aplikace vytvořené pomocí šablony ASP.NET Core používají databázi SQLite. Třídy modelu aplikace se používají s [Entity Framework Core (EF Core)](/ef/core) ([poskytovatel databáze SQLite EF Core](/ef/core/providers/sqlite)) pro práci s databází. EF Core je rozhraní pro mapování relačních objektů (ORM), které zjednodušuje přístup k datům.

Modelu třídy jsou označovány jako POCO třídy (od "prostý staré CLR objekty"), protože nemají žádné závislosti na EF Core. Definují vlastnosti data, která jsou uložena v databázi.

[!INCLUDE[](~/includes/rp/download.md)]

## <a name="add-a-data-model"></a>Přidání datového modelu

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Klikněte pravým tlačítkem myši **RazorPagesMovie** Projekt > **přidat** > **novou složku**. Název složky *modely*.

Klikněte pravým tlačítkem myši *modely* složky. Vyberte **přidat** > **třídy**. Název třídy **filmu**.

[!INCLUDE [model 1b](~/includes/RP/model1b.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Přidat složku s názvem *modely*.
* Přidat třídu *modely* složku s názvem *Movie.cs*.

[!INCLUDE [model 1b](~/includes/RP/model1b.md)]

[!INCLUDE [model 2](~/includes/RP/model2.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

* V Průzkumníku řešení klikněte pravým tlačítkem myši **RazorPagesMovie** projektu a pak vyberte **přidat** > **novou složku**. Název složky *modely*.
* Klikněte pravým tlačítkem na složku *modely* a pak vyberte **Přidat** > **nový soubor**.
* V **nový soubor** dialogové okno:

  * Vyberte **Obecné** v levém podokně.
  * V prostředním podokně vyberte **prázdnou třídu** .
  * Název třídy **film** a vyberte **nový**.

[!INCLUDE [model 1b](~/includes/RP/model1b.md)]

[!INCLUDE [model 2](~/includes/RP/model2.md)]

---

Sestavte projekt a ověřte, že nejsou žádné chyby během kompilace.

## <a name="scaffold-the-movie-model"></a>Vygenerované uživatelské rozhraní Video modelu

V této části je automaticky generovaný model video. To znamená vytvoří nástroj pro generování uživatelského rozhraní stránky pro operace vytvoření, čtení, aktualizace a odstranění (CRUD) pro model video.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Vytvoření *stránek/filmy* složky:

* Klikněte pravým tlačítkem na složku *stránky* > **Přidat** > **novou složku**.
* Název složky *filmy*

Klikněte pravým tlačítkem na složku *stránky nebo filmy* > **Přidat** > **Nová vygenerovaná položka**.

![Image z předchozích kroků.](model/_static/sca.png)

V dialogovém okně **Přidat generování uživatelského rozhraní** vyberte **Razor Pages pomocí Entity Framework (CRUD)** > **Přidat**.

![Image z předchozích kroků.](model/_static/add_scaffold.png)

Dokončení **přidat stránky Razor pomocí Entity Frameworku (CRUD)** dialogové okno:
<!-- In the next section, change 
(plus) sign and accept the generated name 
to use Data, it should not use models. That will make the namespace the same for the VS version and the CLI version
-->

* V **třída modelu** rozevírací seznam, vyberte **Movie (RazorPagesMovie.Models)** .
* V **třída kontextu dat** řádek, vyberte **+** (plus) podepsat a přijměte vygenerovaný název **RazorPagesMovie.Models.RazorPagesMovieContext**.
* Vyberte **Přidat**.

![Image z předchozích kroků.](model/_static/arp.png)

*Appsettings.json* souboru aktualizovali připojovací řetězec použitý pro připojení k místní databázi.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* Otevřete okno příkazového řádku v adresáři projektu (adresář, který obsahuje *Program.cs*, *Startup.cs*, a *.csproj* soubory).
* Nainstalujte nástroj pro generování uživatelského rozhraní:

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* **Pro Windows**: spusťte následující příkaz:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **Pro macOS a Linux**: spusťte následující příkaz:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold gen params](~/includes/RP/model4.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

* Otevřete okno příkazového řádku v adresáři projektu (adresář, který obsahuje *Program.cs*, *Startup.cs*, a *.csproj* soubory).
* Nainstalujte nástroj pro generování uživatelského rozhraní:

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* Spusťte následující příkaz:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold gen params](~/includes/RP/model4.md)]

---

Vygenerované uživatelské rozhraní proces vytvoří a aktualizuje následující soubory:

### <a name="files-created"></a>Soubory vytvořené

* *Stránky/filmy*: vytvoření, odstranění, podrobností, úpravy a Index.
* *Data/RazorPagesMovieContext.cs*

### <a name="file-updated"></a>Aktualizovat soubor

* *Startup.cs*

Vytvořený a aktualizované soubory jsou vysvětlené v následující části.

<a name="pmc"></a>

## <a name="initial-migration"></a>Počáteční migraci

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

V této části konzoly Správce balíčků (PMC) umožňuje:

* Přidáte počáteční migraci.
* Aktualizujte počáteční migraci databáze.

V nabídce **nástroje** vyberte **správce balíčků NuGet** > **konzolu Správce balíčků**.

  ![PMC nabídky](../first-mvc-app/adding-model/_static/pmc.png)

V konzole PMC zadejte následující příkazy:

```Powershell
Add-Migration Initial
Update-Database
```

`Add-Migration` Příkaz vygeneruje kód pro vytvoření schématu počáteční databáze. Schéma je založené na modelu určeném v `DbContext` (v souboru *RazorPagesMovieContext.cs* ). K pojmenování migrace se používá argument `InitialCreate`. Můžete použít libovolný název, ale podle konvence je název, který popisuje migraci použít. Další informace najdete v tématu <xref:data/ef-mvc/migrations>.

`Update-Database` Příkaz spustí `Up` metoda ve *migrace /\<časové razítko > _InitialCreate.cs* souboru. `Up` Metoda vytvoří databázi.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE [initial migration](~/includes/RP/model3.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

[!INCLUDE [initial migration](~/includes/RP/model3.md)]

---
> [!NOTE]
> Předchozí příkazy generují následující upozornění:*pro desetinný sloupec ' Price ' pro typ entity ' video ' nebyl zadán žádný typ ' Price '. To způsobí, že se hodnoty tiše zkrátí, pokud se nevejdou do výchozí přesnosti a rozsahu. Explicitně zadejte typ sloupce SQL serveru, který může obsahovat všechny hodnoty pomocí ' HasColumnType () '.* Toto upozornění můžete ignorovat, bude opraveno v pozdějším kurzu.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>Prozkoumání kontextu registrovaný pomocí vkládání závislostí

ASP.NET Core využívá rozhraní [injektáž závislostí](xref:fundamentals/dependency-injection). Služby (například kontext EF Core databáze) jsou registrované pomocí vkládání závislostí při spuštění aplikace. Komponenty, které vyžadují tyto služby (například stránky Razor) jsou k dispozici tyto služby prostřednictvím parametry konstruktoru. Později v tomto kurzu se zobrazí kód konstruktor, který získá instanci kontext databáze.

Nástroj pro generování uživatelského rozhraní automaticky vytvoří kontext databáze a zaregistrovaného kontejneru pro vkládání závislostí.

Zkontrolujte `Startup.ConfigureServices` metody. Zvýrazněný řádek byl přidán modulem scaffolder:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

`RazorPagesMovieContext` Souřadnice funkce EF Core (vytvoření, čtení, aktualizace, odstranění atd.) pro `Movie` modelu. Kontext dat (`RazorPagesMovieContext`) je odvozen z [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext). Kontext dat určuje entit, které jsou zahrnuty v datovém modelu.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Data/RazorPagesMovieContext.cs)]

Předchozí kód vytvoří vlastnost [`DbSet<Movie>`](/dotnet/api/microsoft.entityframeworkcore.dbset-1) pro sadu entit. Terminologie Entity Framework obvykle sadu entit odpovídá databázové tabulky. Entita odpovídající řádek v tabulce.

Název připojovacího řetězce je předán v rámci voláním metody na [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) objektu. Pro místní vývoj [ASP.NET Core konfigurační systém](xref:fundamentals/configuration/index) načte připojovací řetězec z *appsettings.json* souboru.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Zkontrolujte `Up` metody.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pro Mac](#tab/visual-studio-mac)

Zkontrolujte `Up` metody.

---

<a name="test"></a>

### <a name="test-the-app"></a>Testování aplikace

* Spusťte aplikaci a připojit `/Movies` na adresu URL v prohlížeči (`http://localhost:port/movies`).

Pokud dojde k chybě:

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

Je provedena [kroku migrace](#pmc).

* Test **vytvořit** odkaz.

  ![Vytvoření stránky](model/_static/conan.png)

  > [!NOTE]
  > Není možné zadat desetinné čárky v `Price` pole. Pro podporu [k ověřování jQuery](https://jqueryvalidation.org/) pro neanglická národní prostředí, které používají čárkou (",") desetinné čárky a USA retweetovat neanglické formáty kalendářního data, aplikace musí být globalizována. Globalizace pokyny najdete v tématu [tento problém Githubu](https://github.com/aspnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).

* Test **upravit**, **podrobnosti**, a **odstranit** odkazy.

V dalším kurzu vysvětluje souborů vytvořených databázovým generování uživatelského rozhraní.

## <a name="additional-resources"></a>Další materiály a zdroje informací

> [!div class="step-by-step"]
> [Předchozí: Začínáme](xref:tutorials/razor-pages/razor-pages-start)
> [Další: generované uživatelské rozhraní pro stránky Razor](xref:tutorials/razor-pages/page)

::: moniker-end
