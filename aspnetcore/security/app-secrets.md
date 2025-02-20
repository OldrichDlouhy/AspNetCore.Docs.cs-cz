---
title: Bezpečné ukládání tajných kódů aplikací ve vývoji v ASP.NET Core
author: rick-anderson
description: Naučte se ukládat a načítat citlivé informace jako tajné klíče aplikace při vývoji aplikace ASP.NET Core.
ms.author: scaddie
ms.custom: mvc
ms.date: 03/13/2019
uid: security/app-secrets
ms.openlocfilehash: 0203a5737caf1af809b739d9e266a6971cd1523b
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 09/18/2019
ms.locfileid: "71080717"
---
# <a name="safe-storage-of-app-secrets-in-development-in-aspnet-core"></a>Bezpečné ukládání tajných kódů aplikací ve vývoji v ASP.NET Core

Od [Rick Anderson](https://twitter.com/RickAndMSFT), [Daniel Skořepa](https://github.com/danroth27)a [Scott Addie](https://github.com/scottaddie)

[Zobrazení nebo stažení ukázkového kódu](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/app-secrets/samples) ([stažení](xref:index#how-to-download-a-sample))

Tento dokument vysvětluje techniky pro ukládání a načítání citlivých dat během vývoje aplikace ASP.NET Core. Nikdy neukládejte hesla nebo další citlivá data ve zdrojovém kódu. Provozní tajemství by se neměla používat pro vývoj nebo testování. Pomocí [poskytovatele konfigurace Azure Key Vault](xref:security/key-vault-configuration)můžete ukládat a chránit tajné kódy Azure test a produkčního prostředí.

## <a name="environment-variables"></a>Proměnné prostředí

Proměnné prostředí se používají k tomu, aby nedocházelo k ukládání tajných klíčů k aplikacím v kódu nebo v místních konfiguračních souborech. Proměnné prostředí přepíšou konfigurační hodnoty pro všechny dříve zadané zdroje konfigurace.

::: moniker range="<= aspnetcore-1.1"

Nakonfigurujte čtení hodnot proměnných prostředí voláním <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> `Startup` v konstruktoru:

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupConstructor&highlight=8)]

::: moniker-end

Vezměte v úvahu ASP.NET Core webovou aplikaci, ve které je povolené zabezpečení **individuálních uživatelských účtů** . Výchozí připojovací řetězec databáze je obsažen v souboru *appSettings. JSON* projektu s klíčem `DefaultConnection`. Výchozí připojovací řetězec je pro LocalDB, který běží v uživatelském režimu a nevyžaduje heslo. Během nasazování `DefaultConnection` aplikace se hodnota klíče dá přepsat hodnotou proměnné prostředí. Proměnná prostředí může ukládat úplný připojovací řetězec s citlivými přihlašovacími údaji.

> [!WARNING]
> Proměnné prostředí se většinou ukládají v prostém, nešifrovaném textu. Pokud dojde k ohrožení bezpečnosti počítače nebo procesu, jsou proměnné prostředí dostupné nedůvěryhodnými stranami. Další opatření, která zabraňují odhalení tajných kódů uživateli, mohou být povinná.

[!INCLUDE[](~/includes/environmentVarableColon.md)]

## <a name="secret-manager"></a>Správce tajných klíčů

Nástroj Správce tajných klíčů ukládá citlivé údaje během vývoje ASP.NET Core projektu. V tomto kontextu je část citlivých dat tajný klíč aplikace. Tajné klíče aplikace se ukládají v odděleném umístění ze stromu projektu. Tajné klíče aplikace jsou přidruženy k určitému projektu nebo sdíleny napříč několika projekty. Tajné kódy aplikace nejsou zaregistrovány do správy zdrojového kódu.

> [!WARNING]
> Nástroj Správce tajných klíčů nešifruje uložené tajné klíče a neměl by být považován za důvěryhodné úložiště. Pouze pro účely vývoje. Klíče a hodnoty jsou uložené v konfiguračním souboru JSON v adresáři profilu uživatele.

## <a name="how-the-secret-manager-tool-works"></a>Jak funguje nástroj Správce tajných klíčů

Nástroj Správce tajných klíčů vyříznout podrobnosti implementace, například WHERE a způsob uložení hodnot. Tento nástroj můžete použít bez znalosti těchto podrobností implementace. Hodnoty jsou uložené v konfiguračním souboru JSON ve složce profilu uživatele chráněné systémem v místním počítači:

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

Cesta k systému souborů:

`%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`

# <a name="linux--macostablinuxmacos"></a>[Linux / macOS](#tab/linux+macos)

Cesta k systému souborů:

`~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

---

V předchozích cestách souborů nahraďte `<user_secrets_id>` `UserSecretsId` hodnotou zadanou v souboru *. csproj* .

Nepište kód, který závisí na umístění nebo formátu dat uložených pomocí nástroje Správce tajných klíčů. Tyto podrobnosti implementace se můžou změnit. Například tajné hodnoty nejsou šifrovány, ale mohou být v budoucnu.

::: moniker range="<= aspnetcore-2.0"

## <a name="install-the-secret-manager-tool"></a>Instalace nástroje Správce tajných klíčů

Nástroj Správce tajných klíčů je součástí .NET Core CLI v .NET Core SDK 2.1.300 nebo novějším. Pro .NET Core SDK verze před 2.1.300 je instalace nástroje nutná.

> [!TIP]
> Pokud `dotnet --version` chcete zobrazit nainstalované číslo verze .NET Core SDK, spusťte z příkazového prostředí.

Zobrazí se upozornění, pokud .NET Core SDK používá nástroj:

```console
The tool 'Microsoft.Extensions.SecretManager.Tools' is now included in the .NET Core SDK. Information on resolving this warning is available at (https://aka.ms/dotnetclitools-in-box).
```

Do projektu ASP.NET Core nainstalujte balíček NuGet [Microsoft. Extensions. SecretManager. Tools](https://www.nuget.org/packages/Microsoft.Extensions.SecretManager.Tools/) . Příklad:

[!code-xml[](app-secrets/samples/1.x/UserSecrets/UserSecrets.csproj?name=snippet_CsprojFile&highlight=15-16)]

Spuštěním následujícího příkazu v příkazovém prostředí Ověřte instalaci nástroje:

```dotnetcli
dotnet user-secrets -h
```

Nástroj Správce tajných klíčů zobrazuje ukázkové použití, možnosti a nápovědu k příkazům:

```console
Usage: dotnet user-secrets [options] [command]

Options:
  -?|-h|--help                        Show help information
  --version                           Show version information
  -v|--verbose                        Show verbose output
  -p|--project <PROJECT>              Path to project. Defaults to searching the current directory.
  -c|--configuration <CONFIGURATION>  The project configuration to use. Defaults to 'Debug'.
  --id                                The user secret ID to use.

Commands:
  clear   Deletes all the application secrets
  list    Lists all the application secrets
  remove  Removes the specified user secret
  set     Sets the user secret to the specified value

Use "dotnet user-secrets [command] --help" for more information about a command.
```

> [!NOTE]
> Musíte být ve stejném adresáři jako soubor *. csproj* ke spuštění nástrojů definovaných v `DotNetCliToolReference` prvcích souboru *. csproj* .

::: moniker-end

## <a name="enable-secret-storage"></a>Povolit tajné úložiště

Nástroj Správce tajných klíčů pracuje na nastavení konfigurace specifické pro projekt uložené v profilu uživatele.

::: moniker range=">= aspnetcore-3.0"

Nástroj Správce tajných klíčů obsahuje `init` příkaz v .NET Core SDK 3.0.100 nebo novějším. Chcete-li použít tajné klíče uživatele, spusťte následující příkaz v adresáři projektu:

```dotnetcli
dotnet user-secrets init
```

Předchozí příkaz přidá `UserSecretsId` prvek `PropertyGroup` v rámci souboru *. csproj* . Ve výchozím nastavení je vnitřní text pro `UserSecretsId` identifikátor GUID. Vnitřní text je libovolný, ale je pro projekt jedinečný.

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

Pro použití uživatelských tajných klíčů definujte `UserSecretsId` prvek `PropertyGroup` v rámci souboru *. csproj* . Vnitřní text `UserSecretsId` je libovolný, ale je pro projekt jedinečný. Vývojáři obvykle generují identifikátor GUID pro `UserSecretsId`.

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](app-secrets/samples/2.x/UserSecrets/UserSecrets.csproj?name=snippet_PropertyGroup&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](app-secrets/samples/1.x/UserSecrets/UserSecrets.csproj?name=snippet_PropertyGroup&highlight=3)]

::: moniker-end

> [!TIP]
> V aplikaci Visual Studio klikněte pravým tlačítkem myši na projekt v Průzkumník řešení a v místní nabídce vyberte možnost **spravovat tajné klíče uživatele** . Tento gesto přidá `UserSecretsId` prvek, který je vyplněn identifikátorem GUID, do souboru *. csproj* .

## <a name="set-a-secret"></a>Nastavení tajného klíče

Definujte tajný klíč aplikace skládající se z klíče a jeho hodnoty. Tajný kód je přidružený k `UserSecretsId` hodnotě projektu. Například spusťte následující příkaz z adresáře, ve kterém existuje soubor *. csproj* :

```dotnetcli
dotnet user-secrets set "Movies:ServiceApiKey" "12345"
```

V předchozím příkladu dvojtečka označuje `Movies` literál objektu `ServiceApiKey` s vlastností.

Nástroj Správce tajných klíčů je možné použít i v jiných adresářích. Pomocí možnosti zadejte cestu k systému souborů, na které existuje soubor *. csproj.* `--project` Příklad:

```dotnetcli
dotnet user-secrets set "Movies:ServiceApiKey" "12345" --project "C:\apps\WebApp1\src\WebApp1"
```

### <a name="json-structure-flattening-in-visual-studio"></a>Sloučení struktury JSON v aplikaci Visual Studio

Gesto **Spravovat uživatelské tajemství** v aplikaci Visual Studio otevře v textovém editoru soubor *tajných kódů. JSON* . Nahraďte obsah *tajných klíčů. JSON* páry klíč-hodnota, které se mají uložit. Příklad:

```json
{
  "Movies": {
    "ConnectionString": "Server=(localdb)\\mssqllocaldb;Database=Movie-1;Trusted_Connection=True;MultipleActiveResultSets=true",
    "ServiceApiKey": "12345"
  }
}
```

Struktura JSON je po úpravách prostřednictvím `dotnet user-secrets remove` nebo `dotnet user-secrets set`. shrnuta. Například spuštění `dotnet user-secrets remove "Movies:ConnectionString"` sbalí `Movies` literál objektu. Změněný soubor se podobá následujícímu:

```json
{
  "Movies:ServiceApiKey": "12345"
}
```

## <a name="set-multiple-secrets"></a>Nastavení více tajných klíčů

Dávku tajných kódů lze nastavit pomocí `set` příkazuového řádku JSON pro příkaz. V následujícím příkladu je obsah souboru *input. JSON* v kanálu do `set` příkazu.

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

Otevřete příkazové prostředí a spusťte následující příkaz:

  ```dotnetcli
  type .\input.json | dotnet user-secrets set
  ```

# <a name="linux--macostablinuxmacos"></a>[Linux / macOS](#tab/linux+macos)

Otevřete příkazové prostředí a spusťte následující příkaz:

  ```dotnetcli
  cat ./input.json | dotnet user-secrets set
  ```

---

## <a name="access-a-secret"></a>Přístup ke tajnému kódu

[Rozhraní API pro konfiguraci ASP.NET Core](xref:fundamentals/configuration/index) poskytuje přístup ke tajným tajným klíčům správce.

::: moniker range=">= aspnetcore-2.0 <= aspnetcore-2.2"

Pokud je projekt cílen .NET Framework, nainstalujte balíček NuGet [Microsoft. Extensions. Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) .

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

V ASP.NET Core 2,0 nebo novějším je zdroj konfigurace tajných kódů uživatelských tajných kódů automaticky přidán do režimu vývoje <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> , pokud projekt volá inicializaci nové instance hostitele s předem konfigurovanými výchozími hodnotami. `CreateDefaultBuilder`volá <xref:Microsoft.Extensions.Configuration.UserSecretsConfigurationExtensions.AddUserSecrets*> se, <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName> když <xref:Microsoft.AspNetCore.Hosting.EnvironmentName.Development>je:

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Program.cs?name=snippet_CreateWebHostBuilder&highlight=2)]

Pokud `CreateDefaultBuilder` není volána, přidejte zdroj konfigurace tajných klíčů uživatele explicitně voláním <xref:Microsoft.Extensions.Configuration.UserSecretsConfigurationExtensions.AddUserSecrets*> v `Startup` konstruktoru. Volejte `AddUserSecrets` pouze v případě, že aplikace běží ve vývojovém prostředí, jak je znázorněno v následujícím příkladu:

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupConstructor&highlight=12)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

Nainstalujte balíček NuGet [Microsoft. Extensions. Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) .

Přidejte zdroj konfigurace uživatelských tajných klíčů s voláním <xref:Microsoft.Extensions.Configuration.UserSecretsConfigurationExtensions.AddUserSecrets*> do `Startup` v konstruktoru:

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupConstructor&highlight=12)]

::: moniker-end

Uživatelské tajné kódy lze načíst prostřednictvím `Configuration` rozhraní API:

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Startup.cs?name=snippet_StartupClass&highlight=14)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupClass&highlight=26)]

::: moniker-end

## <a name="map-secrets-to-a-poco"></a>Mapování tajných kódů na POCO

Mapování celého literálu objektu na POCO (jednoduchá třída .NET s vlastnostmi) je užitečná pro agregaci souvisejících vlastností.

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

K namapování předchozích tajných kódů na POCO použijte `Configuration` funkci [vazby grafu objektů](xref:fundamentals/configuration/index#bind-to-an-object-graph) rozhraní API. Následující kód se váže k vlastnímu `MovieSettings` POCO a přistupuje `ServiceApiKey` k hodnotě vlastnosti:

::: moniker range=">= aspnetcore-1.1"

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Startup3.cs?name=snippet_BindToObjectGraph)]

::: moniker-end

::: moniker range="= aspnetcore-1.0"

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup3.cs?name=snippet_BindToObjectGraph)]

::: moniker-end

Tajné kódy `Movies:ServiceApiKey`ajsou mapovány na příslušné vlastnosti v `MovieSettings`: `Movies:ConnectionString`

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Models/MovieSettings.cs?name=snippet_MovieSettingsClass)]

## <a name="string-replacement-with-secrets"></a>Výměna řetězců s tajnými kódy

Ukládání hesel v prostém textu je nezabezpečené. Připojovací řetězec databáze uložený v souboru *appSettings. JSON* může například zahrnovat heslo pro zadaného uživatele:

[!code-json[](app-secrets/samples/2.x/UserSecrets/appsettings-unsecure.json?highlight=3)]

Bezpečnější je ukládání hesla jako tajného klíče. Příklad:

```dotnetcli
dotnet user-secrets set "DbPassword" "pass123"
```

Odeberte dvojici klíč-hodnota z připojovacího řetězce v souboru *appSettings. JSON.* `Password` Příklad:

[!code-json[](app-secrets/samples/2.x/UserSecrets/appsettings.json?highlight=3)]

Hodnota tajného klíče může být nastavena na <xref:System.Data.SqlClient.SqlConnectionStringBuilder> <xref:System.Data.SqlClient.SqlConnectionStringBuilder.Password*> vlastnost objektu pro dokončení připojovacího řetězce:

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Startup2.cs?name=snippet_StartupClass&highlight=14-17)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup2.cs?name=snippet_StartupClass&highlight=26-29)]

::: moniker-end

## <a name="list-the-secrets"></a>Výpis tajných kódů

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

Z adresáře, ve kterém existuje soubor *. csproj* , spusťte následující příkaz:

```dotnetcli
dotnet user-secrets list
```

Zobrazí se následující výstup:

```console
Movies:ConnectionString = Server=(localdb)\mssqllocaldb;Database=Movie-1;Trusted_Connection=True;MultipleActiveResultSets=true
Movies:ServiceApiKey = 12345
```

V předchozím příkladu dvojtečka v názvech klíčů označuje hierarchii objektů v rámci *tajných kódů. JSON*.

## <a name="remove-a-single-secret"></a>Odebrání jednoho tajného klíče

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

Z adresáře, ve kterém existuje soubor *. csproj* , spusťte následující příkaz:

```dotnetcli
dotnet user-secrets remove "Movies:ConnectionString"
```

Soubor *tajných kódů. JSON* aplikace byl změněn tak, aby odstranil dvojici klíč-hodnota přidruženou `MoviesConnectionString` ke klíči:

```json
{
  "Movies": {
    "ServiceApiKey": "12345"
  }
}
```

Při `dotnet user-secrets list` spuštění se zobrazí následující zpráva:

```console
Movies:ServiceApiKey = 12345
```

## <a name="remove-all-secrets"></a>Odebrat všechna tajná klíčová

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

Z adresáře, ve kterém existuje soubor *. csproj* , spusťte následující příkaz:

```dotnetcli
dotnet user-secrets clear
```

Všechny tajné klíče uživatele pro aplikaci byly odstraněny ze souboru *tajných kódů. JSON* :

```json
{}
```

Při `dotnet user-secrets list` spuštění se zobrazí následující zpráva:

```console
No secrets configured for this application.
```

## <a name="additional-resources"></a>Další zdroje

* <xref:fundamentals/configuration/index>
* <xref:security/key-vault-configuration>
