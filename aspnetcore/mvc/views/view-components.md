---
title: Zobrazit součásti v ASP.NET Core
author: rick-anderson
description: Přečtěte si, jak se v ASP.NET Core používají komponenty a jak je přidat do aplikací.
ms.author: riande
ms.custom: mvc
ms.date: 05/14/2019
uid: mvc/views/view-components
ms.openlocfilehash: e6990368519857a27b291d7d565c09072f23f1b0
ms.sourcegitcommit: 7001657c00358b082734ba4273693b9b3ed35d2a
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 07/31/2019
ms.locfileid: "68670083"
---
# <a name="view-components-in-aspnet-core"></a>Zobrazit součásti v ASP.NET Core

Podle [Rick Anderson](https://twitter.com/RickAndMSFT)

[Zobrazení nebo stažení ukázkového kódu](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/view-components/sample) ([stažení](xref:index#how-to-download-a-sample))

## <a name="view-components"></a>Komponenty zobrazení

Zobrazení součástí se podobá částečným zobrazením, ale jsou mnohem výkonnější. Zobrazení komponent nepoužívá vazbu modelu a závisí pouze na datech poskytnutých při jejich volání. Tento článek byl napsaný pomocí řadičů a zobrazení, ale kromě toho se v Razor Pages používají i komponenty.

Součást zobrazení:

* Vykreslí blok dat, nikoli celou odpověď.
* Zahrnuje stejné výhody a testování, které byly nalezeny mezi kontrolkou a zobrazením.
* Může mít parametry a obchodní logiku.
* Je obvykle vyvolána ze stránky rozložení.

Zobrazit součásti jsou určeny kdekoli, kde máte opakovaně použitelnou logiku vykreslování, která je příliš složitá pro částečné zobrazení, například:

* Dynamické navigační nabídky
* Označení cloudu (kde se dotazuje databáze)
* Panel přihlášení
* Nákupní košík
* Nedávno publikované články
* Obsah bočního panelu na typickém blogu
* Přihlašovací panel, který se vykreslí na každé stránce, a zobrazí buď odkazy pro odhlášení nebo přihlášení, v závislosti na stavu přihlášení uživatele.

Součást zobrazení se skládá ze dvou částí: třídy (obvykle odvozené od [ViewComponent](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponent)) a výsledek, který vrátí (obvykle zobrazení). Podobně jako řadiče se může jednat o součást zobrazení POCO, ale většina vývojářů bude chtít využít metody a vlastnosti, které jsou k dispozici odvozením `ViewComponent`z.

Při zvažování, zda zobrazit součásti vyhovují specifikacím aplikace, zvažte místo toho použití součástí Razor. Komponenty Razor také kombinují značky s C# kódem k tvorbě opakovaně použitelných jednotek uživatelského rozhraní. Komponenty Razor jsou navržené pro produktivitu vývojářů při poskytování logiky uživatelského rozhraní a způsobu složení na straně klienta. Další informace naleznete v tématu <xref:blazor/components>.

## <a name="creating-a-view-component"></a>Vytvoření komponenty zobrazení

Tato část obsahuje požadavky na vysoké úrovni pro vytvoření komponenty zobrazení. Později v tomto článku probereme podrobný přehled jednotlivých kroků a vytvoříme pohledovou komponentu.

### <a name="the-view-component-class"></a>Třída zobrazení součásti

Třídu zobrazení komponenty lze vytvořit pomocí kterékoli z následujících možností:

* Odvození z *ViewComponent*
* Upravení třídu s `[ViewComponent]` atributem nebo odvozením z třídy `[ViewComponent]` s atributem.
* Vytvoření třídy, kde název končí příponou *ViewComponent*

Stejně jako řadiče musí být zobrazení součásti veřejné, nevnořené a neabstraktní třídy. Název komponenty zobrazení je název třídy s odebraným příponou "ViewComponent". Dá se taky explicitně zadat pomocí `ViewComponentAttribute.Name` vlastnosti.

Třída zobrazení komponenty:

* Plně podporuje [vkládání závislostí](../../fundamentals/dependency-injection.md) konstruktoru

* Netrvá v životním cyklu kontroléru, což znamená, že nemůžete použít [filtry](../controllers/filters.md) v součásti zobrazení

### <a name="view-component-methods"></a>Zobrazit metody komponenty

Komponenta zobrazení `InvokeAsync` definuje svou logiku v metodě, která `Task<IViewComponentResult>` vrací nebo v `IViewComponentResult`synchronní `Invoke` metodě, která vrací. Parametry přicházejí přímo z vyvolání součásti zobrazení, nikoli z vazby modelu. Součást zobrazení nikdy nezpracovává požadavek přímo. Obvykle komponenta zobrazení inicializuje model a předá ho zobrazení voláním `View` metody. V části Souhrn si prohlédněte metody komponenty:

* Definujte metodu, která `Task<IViewComponentResult>` vrátí `Invoke` nebo`IViewComponentResult`synchronní metodu, která vrátí. `InvokeAsync`
* Obvykle inicializuje model a předá ho zobrazení voláním `ViewComponent` `View` metody.
* Parametry pocházejí z volající metody, nikoli HTTP. Neexistuje žádná vazba modelu.
* Nejsou dostupné přímo jako koncový bod HTTP. Jsou vyvolány z vašeho kódu (obvykle v zobrazení). Součást zobrazení nikdy nezpracovává požadavek.
* Jsou v signatuře přetížené místo jakýchkoli podrobností z aktuální žádosti HTTP.

### <a name="view-search-path"></a>Zobrazit cestu pro hledání

Modul runtime vyhledává zobrazení v následujících cestách:

* /Views/{Controller} název součásti/Components/{View}/{View}
* Název součásti/Views/Shared/Components/{View}/{View}
* Název součásti/Pages/Shared/Components/{View}/{View}

Cesta pro hledání se vztahuje na projekty pomocí řadičů a zobrazení a Razor Pages.

Výchozí název zobrazení pro součást zobrazení je *výchozí*, což znamená, že váš soubor zobrazení bude obvykle pojmenovaný *Default. cshtml*. Při vytváření výsledku zobrazení součásti nebo při volání `View` metody lze zadat jiný název zobrazení.

Doporučujeme pojmenovat soubor zobrazení *Default. cshtml* a použít cestu views */Shared/Components/{View Component Name}/{View Name}* . Komponenta zobrazení použitá v tomto příkladu používá zobrazení */Shared/Components/PriorityList/default. cshtml* pro zobrazení součástí zobrazení. `PriorityList`

## <a name="invoking-a-view-component"></a>Vyvolání komponenty zobrazení

Chcete-li použít součást zobrazení, zavolejte v rámci zobrazení následující:

```cshtml
@await Component.InvokeAsync("Name of view component", {Anonymous Type Containing Parameters})
```

Parametry budou předány `InvokeAsync` metodě. Součást zobrazení vyvinutá v článku se vyvolá z zobrazení zobrazení */TODO/index. cshtml.* `PriorityList` V následujícím `InvokeAsync` postupu je metoda volána se dvěma parametry:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

::: moniker range=">= aspnetcore-1.1"

## <a name="invoking-a-view-component-as-a-tag-helper"></a>Vyvolání komponenty zobrazení jako pomocné rutiny značky

Pro ASP.NET Core 1,1 a vyšší můžete vyvolat součást zobrazení jako [pomocníka značky](xref:mvc/views/tag-helpers/intro):

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexTagHelper.cshtml?range=37-38)]

Parametry třídy a metody použita jazyka Pascal pro pomocníky značek jsou přeloženy do jejich [kebabho případu](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101). Pomocný pomocník značek k vyvolání komponenty zobrazení používá `<vc></vc>` element. Součást zobrazení je určena následujícím způsobem:

```cshtml
<vc:[view-component-name]
  parameter1="parameter1 value"
  parameter2="parameter2 value">
</vc:[view-component-name]>
```

Chcete-li použít zobrazení komponenty jako pomocníka značky, zaregistrujte sestavení obsahující komponentu zobrazení pomocí `@addTagHelper` direktivy. Pokud je vaše součást zobrazení v sestavení s názvem `MyWebApp`, přidejte následující direktivu do souboru *_ViewImports. cshtml* :

```cshtml
@addTagHelper *, MyWebApp
```

Můžete zaregistrovat součást zobrazení jako pomocný modul značek pro libovolný soubor, který odkazuje na součást zobrazení. Další informace o registraci pomocných rutin značek najdete v tématu [Správa oboru pomocných značek](xref:mvc/views/tag-helpers/intro#managing-tag-helper-scope) .

`InvokeAsync` Metoda použitá v tomto kurzu:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

Značka pomocníka značky:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexTagHelper.cshtml?range=37-38)]

V ukázce výše `PriorityList` se zobrazí `priority-list`součást zobrazení. Parametry pro zobrazení komponenty jsou předány jako atributy v případě kebab.

::: moniker-end

### <a name="invoking-a-view-component-directly-from-a-controller"></a>Vyvolání komponenty zobrazení přímo z kontroleru

Zobrazení komponent se obvykle vyvolají ze zobrazení, ale můžete je vyvolat přímo z metody kontroleru. Přestože funkce View Components nedefinuje koncové body jako řadiče, můžete snadno implementovat akci kontroleru, která vrací obsah `ViewComponentResult`.

V tomto příkladu je součást zobrazení volána přímo z kontroleru:

[!code-csharp[](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

## <a name="walkthrough-creating-a-simple-view-component"></a>Návod: Vytvoření jednoduché součásti zobrazení

[Stažení](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/view-components/sample), sestavení a otestování počátečního kódu. Jedná se o jednoduchý projekt s `ToDo` řadičem, který zobrazuje seznam položek *TODO* .

![Seznam ToDo](view-components/_static/2dos.png)

### <a name="add-a-viewcomponent-class"></a>Přidat třídu ViewComponent

Vytvořte složku *ViewComponents* a přidejte následující `PriorityListViewComponent` třídu:

[!code-csharp[](view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponent1.cs?name=snippet1)]

Poznámky k kódu:

* Třídy zobrazení komponent mohou být obsaženy v **jakékoli** složce projektu.
* Vzhledem k tomu, že název třídy PriorityList**ViewComponent** končí příponou **ViewComponent**, modul runtime při odkazování na komponentu třídy ze zobrazení použije řetězec "PriorityList". Později vám ukážeme, že se dozvíte podrobněji.
* `[ViewComponent]` Atribut může změnit název, který se používá k odkazování na součást zobrazení. Mohli bychom například jmenovat třídu `XYZ` a `ViewComponent` použít atribut:

  ```csharp
  [ViewComponent(Name = "PriorityList")]
     public class XYZ : ViewComponent
     ```

* Výše uvedený `PriorityList` atribut oznamuje, že selektor komponent zobrazí název při hledání zobrazení přidružených k komponentě a při odkazování na komponentu třídy ze zobrazení pomocí řetězce "PriorityList". `[ViewComponent]` Později vám ukážeme, že se dozvíte podrobněji.
* Komponenta používá [vkládání závislostí](../../fundamentals/dependency-injection.md) k zpřístupnění kontextu dat.
* `InvokeAsync`zpřístupní metodu, která může být volána ze zobrazení a může mít libovolný počet argumentů.
* Metoda vrátí `isDone` sadu položek, které odpovídají parametrům a `maxPriority`. `ToDo` `InvokeAsync`

### <a name="create-the-view-component-razor-view"></a>Vytvoření zobrazení pro zobrazení Razor komponenty

* Vytvořte složku *views/Shared/Components* . Tato složka **musí** být pojmenována *součásti*.

* Vytvořte složku *views/Shared/Components/PriorityList* . Název této složky se musí shodovat s názvem třídy zobrazení, nebo názvem třídy minus přípona (Pokud následovala konvence a použila v názvu třídy příponu *ViewComponent* ). Pokud jste použili `ViewComponent` atribut, musí název třídy odpovídat označení atributu.

* Vytvoření zobrazení */Shared/Components/PriorityList/default. cshtml* Razor View:


  [!code-cshtml[](view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/Default1.cshtml)]

   Zobrazení Razor převezme seznam `TodoItem` a zobrazí je. Pokud metoda View komponenty `InvokeAsync` nepředá název zobrazení (jako v naší ukázce), je použita *Výchozí hodnota* pro název zobrazení podle konvence. Později v tomto kurzu ukážeme, jak předat název zobrazení. Chcete-li přepsat výchozí styl pro určitý kontroler, přidejte zobrazení do složky zobrazení pro konkrétní řadič (například *zobrazení/TODO/komponenty/PriorityList/default. cshtml)* .

    Pokud je součást zobrazení specifická pro konkrétního řadiče, můžete ji přidat do složky pro konkrétního řadiče (*zobrazení/TODO/komponenty/PriorityList/default. cshtml*).

* Přidejte objekt `div` obsahující volání do komponenty seznamu priorit do dolní části souboru *views/index. cshtml* :

    [!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFirst.cshtml?range=34-38)]

Značka `@await Component.InvokeAsync` zobrazuje syntaxi pro volání komponent zobrazení. První argument je název součásti, kterou chceme vyvolat nebo zavolat. Další parametry jsou předány do komponenty. `InvokeAsync`může mít libovolný počet argumentů.

Otestujte aplikaci. Následující obrázek znázorňuje seznam ToDo a prioritní položky:

![položky seznamu úkolů a prioritní](view-components/_static/pi.png)

Můžete také volat součást zobrazení přímo z kontroleru:

[!code-csharp[](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

![prioritní položky z akce IndexVC](view-components/_static/indexvc.png)

### <a name="specifying-a-view-name"></a>Zadání názvu zobrazení

Složitá součást zobrazení může při některých podmínkách určit jiné než výchozí zobrazení. Následující kód ukazuje, jak zadat zobrazení "PVC" z `InvokeAsync` metody. Aktualizujte `PriorityListViewComponent` metodu ve třídě. `InvokeAsync`

[!code-csharp[](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponentFinal.cs?highlight=4,5,6,7,8,9&range=28-39)]

Zkopírujte soubor *views/Shared/Components/PriorityList/default. cshtml* do zobrazení s názvem views */Shared/Components/PriorityList/PVC. cshtml*. Přidejte záhlaví, které indikuje, že se používá zobrazení typu PVC.

[!code-cshtml[](../../mvc/views/view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/PVC.cshtml?highlight=3)]

Aktualizace *zobrazení/TODO/index. cshtml*:

<!-- Views/ToDo/Index.cshtml is never imported, so change to test tutorial -->

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

Spusťte aplikaci a ověřte zobrazení PVC.

![Prioritní zobrazení součásti](view-components/_static/pvc.png)

Pokud se zobrazení typu PVC nevykresluje, ověřte, že voláte komponentu zobrazení s prioritou 4 nebo vyšší.

### <a name="examine-the-view-path"></a>Prohlédněte si cestu zobrazení

* Změňte parametr priority na tři nebo méně, aby zobrazení priority nebylo vráceno.
* Dočasně přejmenujte *zobrazení/TODO/komponenty/PriorityList/default. cshtml* na *1Default. cshtml*.
* Otestujte aplikaci, zobrazí se následující chyba:

   ```
   An unhandled exception occurred while processing the request.
   InvalidOperationException: The view 'Components/PriorityList/Default' wasn't found. The following locations were searched:
   /Views/ToDo/Components/PriorityList/Default.cshtml
   /Views/Shared/Components/PriorityList/Default.cshtml
   EnsureSuccessful
   ```

* Kopírování *views/TODO/Components/PriorityList/1Default. cshtml* na *views/Shared/Components/PriorityList/default. cshtml*.
* Přidáním kódu do zobrazení *sdílené* složky zobrazení TODO můžete zobrazit, že se zobrazení nachází ze *sdílené* složky.
* Otestujte zobrazení **sdílené** komponenty.

![Výstup ToDo se zobrazením sdílené komponenty](view-components/_static/shared.png)

### <a name="avoiding-hard-coded-strings"></a>Zamezení pevně zakódovaných řetězců

Pokud chcete zabezpečení času kompilace, můžete nahradit pevně kódovaný název komponenty zobrazení názvem třídy. Vytvořte součást zobrazení bez přípony "ViewComponent":

[!code-csharp[](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityList.cs?highlight=10&range=5-35)]

Přidejte příkaz do souboru zobrazení Razor a `nameof` použijte operátor: `using`

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexNameof.cshtml?range=1-6,35-)]

## <a name="perform-synchronous-work"></a>Provést synchronní práci

Architektura zpracovává synchronní `Invoke` metodu, pokud nepotřebujete provádět asynchronní práci. Následující metoda vytvoří synchronní `Invoke` součást zobrazení:

```csharp
public class PriorityList : ViewComponent
{
    public IViewComponentResult Invoke(int maxPriority, bool isDone)
    {
        var items = new List<string> { $"maxPriority: {maxPriority}", $"isDone: {isDone}" };
        return View(items);
    }
}
```

Soubor Razor komponenty zobrazit obsahuje seznam řetězců předaných `Invoke` metodě (views */Home/Components/PriorityList/default. cshtml*):

```cshtml
@model List<string>

<h3>Priority Items</h3>
<ul>
    @foreach (var item in Model)
    {
        <li>@item</li>
    }
</ul>
```

::: moniker range=">= aspnetcore-1.1"

Zobrazení komponenty je vyvoláno v souboru Razor (například views */Home/index. cshtml*) pomocí jednoho z následujících přístupů:

* <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper>
* [Pomocník značek](xref:mvc/views/tag-helpers/intro)

Chcete-li <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper> použít přístup, `Component.InvokeAsync`zavolejte:

::: moniker-end

::: moniker range="< aspnetcore-1.1"

Zobrazení komponenty je vyvoláno v souboru Razor (například views */Home/index. cshtml*) pomocí <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper>.

Volání `Component.InvokeAsync`:

::: moniker-end

```cshtml
@await Component.InvokeAsync(nameof(PriorityList), new { maxPriority = 4, isDone = true })
```

::: moniker range=">= aspnetcore-1.1"

Chcete-li použít pomocníka značky, zaregistrujte sestavení obsahující komponentu zobrazení `@addTagHelper` pomocí direktivy (zobrazení součásti je v sestavení s `MyWebApp`názvem):

```cshtml
@addTagHelper *, MyWebApp
```

Použijte nápovědu pro zobrazení značky komponenty v souboru značek Razor:

```cshtml
<vc:priority-list max-priority="999" is-done="false">
</vc:priority-list>
```

::: moniker-end

Signatura `PriorityList.Invoke` metody je synchronní, ale Razor najde a volá `Component.InvokeAsync` metodu v souboru označení.

## <a name="all-view-component-parameters-are-required"></a>Všechny parametry komponenty View jsou povinné.

Každý parametr v součásti zobrazení je povinný atribut. Podívejte se na [Tento problém GitHubu](https://github.com/aspnet/AspNetCore/issues/5011). Pokud je vynechán libovolný parametr:

* Signatura `InvokeAsync` metody se neshoduje, proto se metoda neprovede.
* ViewComponent nevykresluje žádné značky.
* Nebudou vyvolány žádné chyby.

## <a name="additional-resources"></a>Další zdroje

* [Injektáž závislostí do zobrazení](xref:mvc/views/dependency-injection)
