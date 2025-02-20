---
title: SignalR hostitelských ASP.NET Core ve službách na pozadí
author: bradygaster
description: Naučte se, jak odesílat zprávy klientům SignalR z tříd .NET Core BackgroundService.
monikerRange: '>= aspnetcore-2.2'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/12/2019
no-loc:
- SignalR
uid: signalr/background-services
ms.openlocfilehash: 000732115153eeafed3948c2a07acf77ffc34218
ms.sourcegitcommit: 3fc3020961e1289ee5bf5f3c365ce8304d8ebf19
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/12/2019
ms.locfileid: "73964046"
---
# <a name="host-aspnet-core-opno-locsignalr-in-background-services"></a>SignalR hostitelských ASP.NET Core ve službách na pozadí

Od [Brady gastera](https://twitter.com/bradygaster)

Tento článek poskytuje pokyny pro:

* Hostování SignalRch Center pomocí pracovního procesu na pozadí hostovaného s ASP.NET Core.
* Posílání zpráv pro připojené klienty z rozhraní .NET Core [BackgroundService](xref:Microsoft.Extensions.Hosting.BackgroundService).

[Zobrazit nebo stáhnout ukázkový kód](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/background-service/sample/) [(jak stáhnout)](xref:index#how-to-download-a-sample)

## <a name="wire-up-opno-locsignalr-during-startup"></a>Při spuštění SignalR síťový telefon

::: moniker range=">= aspnetcore-3.0"

Hostování ASP.NET Core SignalR centra v kontextu pracovního procesu na pozadí je stejné jako hostování centra ve webové aplikaci v ASP.NET Core. V metodě `Startup.ConfigureServices` volání `services.AddSignalR` přidá požadované služby do vrstvy ASP.NET Core (DI) pro podporu SignalR. V `Startup.Configure`je metoda `MapHub` volána ve zpětném volání `UseEndpoints` pro přenos koncových bodů rozbočovače v kanálu žádosti ASP.NET Core.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSignalR();
        services.AddHostedService<Worker>();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHub<ClockHub>("/hubs/clock");
        });
    }
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

Hostování ASP.NET Core SignalR centra v kontextu pracovního procesu na pozadí je stejné jako hostování centra ve webové aplikaci v ASP.NET Core. V metodě `Startup.ConfigureServices` volání `services.AddSignalR` přidá požadované služby do vrstvy ASP.NET Core (DI) pro podporu SignalR. V `Startup.Configure`je volána metoda `UseSignalR` pro přenos koncových bodů rozbočovače v kanálu žádosti ASP.NET Core.

[!code-csharp[Startup](background-service/sample/Server/Startup.cs?name=Startup)]

::: moniker-end

V předchozím příkladu třída `ClockHub` implementuje `Hub<T>` třídu pro vytvoření rozbočovače se silným typem. `ClockHub` byla nakonfigurovaná v třídě `Startup`, aby reagovala na žádosti na `/hubs/clock`koncového bodu.

Další informace o rozbočovačích se silnými typy najdete v tématu [použití Center v SignalR pro ASP.NET Core](xref:signalr/hubs#strongly-typed-hubs).

> [!NOTE]
> Tato funkce se neomezuje na [> třídy centra\<t](xref:Microsoft.AspNetCore.SignalR.Hub`1) . Budou fungovat i všechny třídy, které dědí z [rozbočovače](xref:Microsoft.AspNetCore.SignalR.Hub), například [DynamicHub](xref:Microsoft.AspNetCore.SignalR.DynamicHub).

[!code-csharp[Startup](background-service/sample/Server/ClockHub.cs?name=ClockHub)]

Rozhraní používané `ClockHub` silným typem je rozhraní `IClock`.

[!code-csharp[Startup](background-service/sample/HubServiceInterfaces/IClock.cs?name=IClock)]

## <a name="call-a-opno-locsignalr-hub-from-a-background-service"></a>Volání centra SignalR ze služby na pozadí

Při spuštění je `Worker` třída `BackgroundService`, která je zapojená pomocí `AddHostedService`.

```csharp
services.AddHostedService<Worker>();
```

Vzhledem k tomu, že SignalR je v rámci `Startup` fáze připojená i v případě, že je každé centrum připojené k jednotlivému koncovému bodu v kanálu požadavků HTTP ASP.NET Core, je každé z nich reprezentované `IHubContext<T>` na serveru. Pomocí funkcí ASP.NET Core DI jsou jiné třídy, které jsou vytvořeny instancí hostující vrstvy, jako jsou `BackgroundService` třídy, třídy kontroleru MVC nebo modely stránek Razor, schopny získat odkazy na centra na straně serveru tím, že během konstrukce přijímají instance `IHubContext<ClockHub, IClock>`.

[!code-csharp[Startup](background-service/sample/Server/Worker.cs?name=Worker)]

Jelikož je metoda `ExecuteAsync` volána iterativním způsobem ve službě na pozadí, je aktuální datum a čas serveru odesíláno připojeným klientům pomocí `ClockHub`.

## <a name="react-to-opno-locsignalr-events-with-background-services"></a>Reakce na události SignalR se službami na pozadí

Podobně jako jedna stránková aplikace, která používá klienta JavaScriptu pro SignalR nebo desktopovou aplikaci .NET, může pomocí <xref:signalr/dotnet-client>použít `BackgroundService` nebo `IHostedService` implementaci k připojení k SignalRm rozbočovači a reagovat na události.

Třída `ClockHubClient` implementuje rozhraní `IClock` i rozhraní `IHostedService`. Tímto způsobem je možné během `Startup` pracovat nepřetržitě a reagovat na události centra ze serveru.

```csharp
public partial class ClockHubClient : IClock, IHostedService
{
}
```

Během inicializace `ClockHubClient` vytvoří instanci `HubConnection` a drátovou metodu `IClock.ShowTime` jako obslužnou rutinu pro `ShowTime` událost centra.

[!code-csharp[The ClockHubClient constructor](background-service/sample/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

V `IHostedService.StartAsync` implementaci je `HubConnection` spouštěn asynchronně.

[!code-csharp[StartAsync method](background-service/sample/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

Během metody `IHostedService.StopAsync` je `HubConnection` odstraněn asynchronně.

[!code-csharp[StopAsync method](background-service/sample/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

## <a name="additional-resources"></a>Další zdroje

* [Začínáme](xref:tutorials/signalr)
* [Centra](xref:signalr/hubs)
* [Publikování do Azure](xref:signalr/publish-to-azure-web-app)
* [Rozbočovače silného typu](xref:signalr/hubs#strongly-typed-hubs)
