---
title: Vkládání závislostí ASP.NET Core Blazor
author: guardrex
description: Podívejte se, jak Blazor aplikace můžou vkládat služby do součástí.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
no-loc:
- Blazor
uid: blazor/dependency-injection
ms.openlocfilehash: a39d913636afc55ac9d831de923ba7ae8db1216b
ms.sourcegitcommit: 3fc3020961e1289ee5bf5f3c365ce8304d8ebf19
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/12/2019
ms.locfileid: "73963082"
---
# <a name="aspnet-core-opno-locblazor-dependency-injection"></a>Vkládání závislostí ASP.NET Core Blazor

Od [Rainer Stropek](https://www.timecockpit.com)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor podporuje [vkládání závislostí (di)](xref:fundamentals/dependency-injection). Aplikace mohou používat vestavěné služby jejich vložením do komponent. Aplikace můžou také definovat a registrovat vlastní služby a zpřístupnit je v celé aplikaci přes DI.

DI je technika přístupu ke službám nakonfigurovaným v centrálním umístění. To může být užitečné v Blazorch aplikacích:

* Sdílejte jednu instanci třídy služby napříč mnoha komponentami, která se označuje jako služba typu *singleton* .
* Oddělit komponenty od konkrétních tříd služeb pomocí abstrakcí odkazů. Představte si třeba rozhraní `IDataAccess` pro přístup k datům v aplikaci. Rozhraní je implementováno konkrétní třídou `DataAccess` a registrováno jako služba v kontejneru služby aplikace. Pokud komponenta používá DI k přijetí implementace `IDataAccess`, komponenta není spojena se konkrétním typem. Implementaci je možné prohodit, třeba pro podrobnější implementaci v testování částí.

## <a name="default-services"></a>Výchozí služby

Výchozí služby se automaticky přidají do kolekce služeb aplikace.

| Služba | Doba platnosti | Popis |
| ------- | -------- | ----------- |
| <xref:System.Net.Http.HttpClient> | Singleton | Poskytuje metody pro posílání požadavků HTTP a příjem odpovědí HTTP z prostředku identifikovaného identifikátorem URI. Všimněte si, že tato instance `HttpClient` používá prohlížeč pro zpracování provozu HTTP na pozadí. [HttpClient. BaseAddress](xref:System.Net.Http.HttpClient.BaseAddress) se automaticky nastaví na základní PŘEDPONu identifikátoru URI aplikace. Další informace najdete v tématu <xref:blazor/call-web-api>. |
| `IJSRuntime` | Singleton | Představuje instanci modulu runtime jazyka JavaScript, kde jsou odesílána volání jazyka JavaScript. Další informace najdete v tématu <xref:blazor/javascript-interop>. |
| `NavigationManager` | Singleton | Obsahuje nápovědu pro práci s identifikátory URI a stavem navigace. Další informace najdete v tématu věnovaném [identifikátorům URI a nápovědě k informacím o stavu navigace](xref:blazor/routing#uri-and-navigation-state-helpers). |

Vlastní zprostředkovatel služeb automaticky neposkytuje výchozí služby uvedené v tabulce. Pokud používáte vlastního poskytovatele služeb a potřebujete některou ze služeb zobrazených v tabulce, přidejte požadované služby k novému poskytovateli služeb.

## <a name="add-services-to-an-app"></a>Přidání služeb do aplikace

Po vytvoření nové aplikace si Projděte metodu `Startup.ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add custom services here
}
```

Metodě `ConfigureServices` se předává <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection>, což je seznam objektů deskriptoru služby (<xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor>). Služby se přidávají tím, že se do kolekce služeb poskytují popisovače služby. Následující příklad ukazuje koncept s rozhraním `IDataAccess` a jeho konkrétní implementací `DataAccess`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IDataAccess, DataAccess>();
}
```

Služby je možné konfigurovat s životností, která jsou uvedená v následující tabulce.

| Doba platnosti | Popis |
| -------- | ----------- |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Scoped*> | aplikace Blazor WebAssembly aktuálně nemají koncept typu DI scopes. služby registrované `Scoped`se chovají jako služby `Singleton`. Model hostování Blazor serveru však podporuje `Scoped` životního cyklu. V Blazorch serverových aplikacích je vymezená registrace služby vymezená na *připojení*. Z tohoto důvodu je vhodnější použití oboru služeb pro služby, které by měly být vymezeny na aktuálního uživatele, a to i v případě, že aktuální záměr je spustit na straně klienta v prohlížeči. |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Singleton*> | DI vytvoří *jednu instanci* služby. Všechny součásti, které vyžadují službu `Singleton`, obdrží instanci stejné služby. |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Transient*> | Pokaždé, když komponenta získá instanci služby `Transient` z kontejneru služby, obdrží *novou instanci* služby. |

Systém DI je založený na systému DI v ASP.NET Core. Další informace najdete v tématu <xref:fundamentals/dependency-injection>.

## <a name="request-a-service-in-a-component"></a>Vyžádání služby v součásti

Po přidání služeb do kolekce služeb tyto služby vloží do součástí pomocí direktivy [\@inject](xref:mvc/views/razor#inject) Razor. `@inject` má dva parametry:

* Zadejte &ndash; typ služby, kterou chcete vložit.
* Vlastnost &ndash; název vlastnosti, která přijímá vloženou službu App Service. Vlastnost nevyžaduje ruční vytvoření. Kompilátor vytvoří vlastnost.

Další informace najdete v tématu <xref:mvc/views/dependency-injection>.

Pro vložení různých služeb použijte více příkazů `@inject`.

Následující příklad ukazuje, jak použít `@inject`. Implementace služby `Services.IDataAccess` je vložena do vlastnosti komponenty `DataRepository`. Všimněte si, jak kód používá abstrakci `IDataAccess`:

[!code-cshtml[](dependency-injection/samples_snapshot/3.x/CustomerList.razor?highlight=2-3,23)]

Interně je vygenerovaná vlastnost (`DataRepository`) upravena atributem `InjectAttribute`. Obvykle se tento atribut nepoužívá přímo. Pokud je vyžadována základní třída pro součásti a vložené vlastnosti jsou také požadovány pro základní třídu, ručně přidejte `InjectAttribute`:

```csharp
public class ComponentBase : IComponent
{
    // DI works even if using the InjectAttribute in a component's base class.
    [Inject]
    protected IDataAccess DataRepository { get; set; }
    ...
}
```

V součástech odvozených ze základní třídy není direktiva `@inject` vyžadována. `InjectAttribute` základní třídy jsou dostatečné:

```cshtml
@page "/demo"
@inherits ComponentBase

<h1>Demo Component</h1>
```

## <a name="use-di-in-services"></a>Použití DI v službách

Komplexní služby můžou vyžadovat další služby. V předchozím příkladu může `DataAccess` vyžadovat výchozí službu `HttpClient`. `@inject` (nebo `InjectAttribute`) nejsou k dispozici pro použití v rámci služeb. Místo toho se musí použít *Injektáže konstruktoru* . Požadované služby jsou přidány přidáním parametrů do konstruktoru služby. Když DI vytvoří službu, rozpoznává služby, které vyžaduje v konstruktoru, a odpovídajícím způsobem je poskytne.

```csharp
public class DataAccess : IDataAccess
{
    // The constructor receives an HttpClient via dependency
    // injection. HttpClient is a default service.
    public DataAccess(HttpClient client)
    {
        ...
    }
}
```

Předpoklady pro vložení konstruktoru:

* Je nutné, aby jeden konstruktor existoval, jehož argumenty mohou být splněny pomocí DI. Další parametry, které nejsou pokryty parametrem DI, jsou povoleny, pokud určují výchozí hodnoty.
* Příslušný konstruktor musí být *veřejný*.
* Musí existovat jeden použitelný konstruktor. V případě nejednoznačnosti Vyvolá příkaz DI výjimku.

## <a name="utility-base-component-classes-to-manage-a-di-scope"></a>Základní třídy komponenty nástroje pro správu oboru DI

V aplikacích ASP.NET Core jsou oborové služby obvykle vymezeny na aktuální požadavek. Po dokončení žádosti se v systému DI odstraní všechny obory nebo přechodné služby. V Blazorch serverových aplikací je rozsah požadavků po dobu trvání připojení klienta, což může vést k přechodným a oborovým službám, které jsou mnohem delší, než se očekávalo.

Chcete-li obor služeb omezit na životnost komponenty, lze použít základní třídy `OwningComponentBase` a `OwningComponentBase<TService>`. Tyto základní třídy zpřístupňují vlastnost `ScopedServices` typu `IServiceProvider`, která řeší služby s vymezenou životností součásti. Chcete-li vytvořit komponentu, která dědí ze základní třídy v Razor, použijte direktivu `@inherits`.

```cshtml
@page "/users"
@attribute [Authorize]
@inherits OwningComponentBase<Data.ApplicationDbContext>

<h1>Users (@Service.Users.Count())</h1>
<ul>
    @foreach (var user in Service.Users)
    {
        <li>@user.UserName</li>
    }
</ul>
```

> [!NOTE]
> Služby vložené do komponenty pomocí `@inject` nebo `InjectAttribute` nejsou vytvořeny v oboru komponenty a jsou svázány s oborem požadavku.

## <a name="additional-resources"></a>Další zdroje

* <xref:fundamentals/dependency-injection>
* <xref:mvc/views/dependency-injection>
