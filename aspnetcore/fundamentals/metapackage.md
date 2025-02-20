---
title: Microsoft. AspNetCore. All Metapackage for ASP.NET Core 2,0
author: Rick-Anderson
description: Microsoft. AspNetCore. All Metapackage se nedoporučuje pro ASP.NET Core 2,1 a novější.
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/25/2018
uid: fundamentals/metapackage
ms.openlocfilehash: 91f39fc59e5682fb19f8cbc6e9ebe5b30e5dcf3c
ms.sourcegitcommit: 8a36be1bfee02eba3b07b7a86085ec25c38bae6b
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 09/24/2019
ms.locfileid: "71219132"
---
# <a name="microsoftaspnetcoreall-metapackage-for-aspnet-core-20"></a>Microsoft. AspNetCore. All Metapackage for ASP.NET Core 2,0

::: moniker range=">= aspnetcore-3.0"

`Microsoft.AspNetCore.All` Metapackage není součástí ASP.NET Core 3,0 a novějších. Další informace najdete v tématu [tento problém Githubu](https://github.com/aspnet/Announcements/issues/314).

::: moniker-end

> [!NOTE]
> Doporučujeme, abyste pro aplikace cílí na ASP.NET Core 2,1 a později používali [Microsoft. AspNetCore. app Metapackage](xref:fundamentals/metapackage-app) a ne tento balíček. Viz téma [migrace z Microsoft. AspNetCore. All do Microsoft. AspNetCore. app](#migrate) v tomto článku.

Tato funkce vyžaduje ASP.NET Core 2. x cílící na rozhraní .NET Core 2. x.

[Microsoft. AspNetCore. All](https://www.nuget.org/packages/Microsoft.AspNetCore.All) je Metapackage, který odkazuje na sdílené rozhraní. *Sdílené rozhraní* je sada sestavení (soubory *. dll* ), která nejsou ve složkách aplikace. Aby bylo možné aplikaci spustit, musí být na počítači nainstalována sdílená rozhraní. Další informace najdete v tématu [sdílené rozhraní](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/).

Sdílené rozhraní, které `Microsoft.AspNetCore.All` odkazuje na:

* Všechny podporované balíčky vytvořené týmem ASP.NET Core.
* Všechny podporované balíčky Entity Framework Core.
* Interní závislosti a závislosti třetích stran používané v rámci ASP.NET Core a Entity Framework Core.

Do `Microsoft.AspNetCore.All` balíčku jsou zahrnuté všechny funkce ASP.NET Core 2. x a Entity Framework Core 2. x. Výchozí šablony projektu, které cílí na ASP.NET Core 2,0, používají tento balíček.

Číslo `Microsoft.AspNetCore.All` verze Metapackage představuje minimální verzi ASP.NET Core a Entity Framework Core verzi.

Následující soubor *. csproj* odkazuje na `Microsoft.AspNetCore.All` Metapackage pro ASP.NET Core:

[!code-xml[](metapackage/samples/Metapackage.All.Example.csproj?highlight=8)]

::: moniker range=">= aspnetcore-2.1"

## <a name="implicit-versioning"></a>Implicitní Správa verzí

V ASP.NET Core 2,1 nebo novější můžete zadat odkaz na `Microsoft.AspNetCore.All` balíček bez verze. Není-li zadána verze, je implicitní verze určena sadou SDK (`Microsoft.NET.Sdk.Web`). Doporučujeme, abyste se spoléhali na implicitní verzi určenou sadou SDK a neexplicitně nastavoval číslo verze na odkaz na balíček. Pokud máte dotazy týkající se tohoto přístupu, ponechte komentář GitHubu v [diskuzi o implicitní verzi Microsoft. AspNetCore. app](https://github.com/aspnet/AspNetCore.Docs/issues/6430).

Implicitní verze je nastavená na `major.minor.0` pro přenosné aplikace. Mechanismus pro přetečení sdíleného rozhraní spustí aplikaci na nejnovější kompatibilní verzi z nainstalovaných sdílených rozhraní. Aby bylo zaručeno, že stejná verze se používá ve vývoji, testování a produkčním prostředí, zajistěte, aby byla stejná verze sdíleného rozhraní nainstalovaná ve všech prostředích. U samostatných aplikací je implicitní číslo verze nastaveno na `major.minor.patch` sdílené rozhraní, které je v nainstalované sadě SDK součástí sady.

Zadání čísla verze na `Microsoft.AspNetCore.All` odkaz na balíček nezaručuje **,** že je zvolena verze sdíleného rozhraní. Například Předpokládejme, že je zadána verze "2.1.1", ale je nainstalována aplikace "2.1.3". V takovém případě bude aplikace používat "2.1.3". I když se to nedoporučuje, můžete zablokovat přeposlání (oprava nebo podverze). Další informace o tom, jak integrovat hostitele dotnet a jak nakonfigurovat jeho chování, najdete v tématu [dotnet Host – přesměrování](https://github.com/dotnet/core-setup/blob/master/Documentation/design-docs/roll-forward-on-no-candidate-fx.md).

Sada SDK projektu musí být nastavena na `Microsoft.NET.Sdk.Web` hodnotu v souboru projektu, aby používala implicitní `Microsoft.AspNetCore.All`verzi. `<Project Sdk="Microsoft.NET.Sdk">` Při zadání `Microsoft.NET.Sdk` sady SDK (v horní části souboru projektu) se vygeneruje následující upozornění:

*Upozornění NU1604: Závislost projektu Microsoft. AspNetCore. All neobsahuje žádné zahrnutí dolní meze. Zahrňte do verze závislosti dolní mez, aby se zajistilo konzistentní výsledky obnovení.*

Jedná se o známý problém se sadou .NET Core 2,1 SDK a bude opraven v sadě .NET Core 2,2 SDK.

::: moniker-end

<a name="migrate"></a>

## <a name="migrating-from-microsoftaspnetcoreall-to-microsoftaspnetcoreapp"></a>Migruje se z Microsoft. AspNetCore. All do Microsoft. AspNetCore. app.

K dispozici `Microsoft.AspNetCore.All` jsou tyto balíčky, ale `Microsoft.AspNetCore.App` ne balíček.

* `Microsoft.AspNetCore.ApplicationInsights.HostingStartup`
* `Microsoft.AspNetCore.AzureAppServices.HostingStartup`
* `Microsoft.AspNetCore.AzureAppServicesIntegration`
* `Microsoft.AspNetCore.DataProtection.AzureKeyVault`
* `Microsoft.AspNetCore.DataProtection.AzureStorage`
* `Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv`
* `Microsoft.AspNetCore.SignalR.Redis`
* `Microsoft.Data.Sqlite`
* `Microsoft.Data.Sqlite.Core`
* `Microsoft.EntityFrameworkCore.Sqlite`
* `Microsoft.EntityFrameworkCore.Sqlite.Core`
* `Microsoft.Extensions.Caching.Redis`
* `Microsoft.Extensions.Configuration.AzureKeyVault`
* `Microsoft.Extensions.Logging.AzureAppServices`
* `Microsoft.VisualStudio.Web.BrowserLink`

Pokud chcete přejít `Microsoft.AspNetCore.All` z `Microsoft.AspNetCore.App`na, pokud vaše aplikace používá rozhraní API z výše uvedených balíčků nebo balíčky, které tyto balíčky přidávají, přidejte odkazy na tyto balíčky v projektu.

Všechny závislosti předchozích balíčků, které jinak nejsou závislé `Microsoft.AspNetCore.App` na, nejsou implicitně zahrnuté. Příklad:

* `StackExchange.Redis`jako závislost`Microsoft.Extensions.Caching.Redis`
* `Microsoft.ApplicationInsights`jako závislost`Microsoft.AspNetCore.ApplicationInsights.HostingStartup`

## <a name="update-aspnet-core-21"></a>Aktualizace ASP.NET Core 2,1

Doporučujeme migrovat na `Microsoft.AspNetCore.App` Metapackage pro 2,1 a novější. Chcete-li nadále `Microsoft.AspNetCore.All` používat Metapackage a zajistit, aby byla nasazena nejnovější verze opravy:

* Ve vývojových počítačích a serverech sestavení: Nainstalujte nejnovější [.NET Core SDK](https://www.microsoft.com/net/download).
* Na serverech nasazení: Nainstalujte nejnovější [modul runtime .NET Core](https://www.microsoft.com/net/download).
 Vaše aplikace bude předána na nejnovější nainstalovanou verzi při restartu aplikace.
