---
title: IP adresa klienta Safelist pro ASP.NET Core
author: damienbod
description: Naučte se psát middleware nebo filtry akcí pro ověření vzdálených IP adres pro seznam schválených IP adres.
ms.author: riande
ms.custom: mvc
ms.date: 08/31/2018
uid: security/ip-safelist
ms.openlocfilehash: ca5b0f8088773027f7403120247cbeca8900bcf5
ms.sourcegitcommit: 16cf016035f0c9acf3ff0ad874c56f82e013d415
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/29/2019
ms.locfileid: "73034346"
---
# <a name="client-ip-safelist-for-aspnet-core"></a>IP adresa klienta Safelist pro ASP.NET Core

[Damien Bowden](https://twitter.com/damien_bod) a [Dykstra](https://github.com/tdykstra)
 
Tento článek ukazuje tři způsoby implementace IP Safelist (označované také jako seznam povolených) v aplikaci ASP.NET Core. Můžete použít:

* Middleware pro kontrolu vzdálené IP adresy každého požadavku.
* Filtry akcí pro kontrolu vzdálené IP adresy žádostí o konkrétní řadiče nebo metody akcí.
* Razor Pages filtry pro kontrolu vzdálené IP adresy žádostí o stránky Razor.

V každém případě je řetězec, který obsahuje schválené IP adresy klienta, uložen v nastavení aplikace. Middleware nebo filtr analyzuje řetězec na seznam a kontroluje, jestli je vzdálená IP adresa v seznamu. V takovém případě se vrátí stavový kód HTTP 403 zakázáno.

[Zobrazit nebo stáhnout ukázkový kód](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/ip-safelist/samples/2.x/ClientIpAspNetCore) ([Jak stáhnout](xref:index#how-to-download-a-sample))

## <a name="the-safelist"></a>Safelist

Seznam je nakonfigurovaný v souboru *appSettings. JSON* . Jedná se o středníkem oddělený seznam a může obsahovat adresy IPv4 a IPv6.

[!code-json[](ip-safelist/samples/2.x/ClientIpAspNetCore/appsettings.json?highlight=2)]

## <a name="middleware"></a>Middleware

Metoda `Configure` přidá middleware a předá do něj řetězec Safelist v parametru konstruktoru.

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_Configure&highlight=10)]

Middleware analyzuje řetězec do pole a hledá v poli vzdálenou IP adresu. Pokud se vzdálená IP adresa nenajde, middleware vrátí HTTP 401 zakázáno. Tento proces ověřování se pro požadavky HTTP GET nepoužívá.

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/AdminSafeListMiddleware.cs?name=snippet_ClassOnly)]

## <a name="action-filter"></a>Filtr akcí

Pokud chcete Safelist jenom pro konkrétní řadiče nebo metody akcí, použijte filtr akcí. Tady je příklad: 

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Filters/ClientIpCheckFilter.cs)]

Filtr akce se přidá do kontejneru služby.

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

Filtr je pak možné použít na řadiči nebo metodě akce.

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Controllers/ValuesController.cs?name=snippet_Filter&highlight=1)]

V ukázkové aplikaci se filtr aplikuje na metodu `Get`. Takže když aplikaci otestujete odesláním požadavku rozhraní API `Get`, atribut ověřuje IP adresu klienta. Při testování voláním rozhraní API pomocí jakékoli jiné metody HTTP middleware ověřuje IP adresu klienta.

## <a name="razor-pages-filter"></a>Filtr Razor Pages 

Pokud chcete Safelist pro aplikaci Razor Pages, použijte filtr Razor Pages. Tady je příklad: 

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Filters/ClientIpCheckPageFilter.cs)]

Tento filtr je povolen přidáním do kolekce filtry MVC.

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServices&highlight=7-9)]

Když aplikaci spustíte a požádáte o stránku Razor, filtr Razor Pages ověřuje IP adresu klienta.

## <a name="next-steps"></a>Další kroky

[Přečtěte si další informace o ASP.NET Core middlewaru](xref:fundamentals/middleware/index).
