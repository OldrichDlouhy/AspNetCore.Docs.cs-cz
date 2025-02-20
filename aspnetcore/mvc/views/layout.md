---
title: Rozložení v ASP.NET Core
author: ardalis
description: Naučte se používat běžné rozložení, direktivy sdílení a spouštět společný kód před vykreslením zobrazení v aplikaci ASP.NET Core.
ms.author: riande
ms.date: 07/30/2019
uid: mvc/views/layout
ms.openlocfilehash: 3ba2f459ca2b04a3001e261acab26880b6582500
ms.sourcegitcommit: f40c9311058c9b1add4ec043ddc5629384af6c56
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/21/2019
ms.locfileid: "74289000"
---
# <a name="layout-in-aspnet-core"></a>Rozložení v ASP.NET Core

[Steve Smith](https://ardalis.com/) a [Dave Brock](https://twitter.com/daveabrock)

Stránky a zobrazení často sdílí vizuální a programové prvky. Tento článek ukazuje, jak:

* Použijte společná rozložení.
* Direktivy Share.
* Před vykreslováním stránek nebo zobrazení spouštějte běžný kód.

Tento dokument popisuje rozložení dvou různých přístupů k ASP.NET Core MVC: Razor Pages a řadiče se zobrazeními. Rozdíly jsou pro toto téma minimální:

* Razor Pages jsou ve složce *stránky* .
* Řadiče se zobrazeními používají pro zobrazení složku *zobrazení* .

## <a name="what-is-a-layout"></a>Co je rozložení

Většina webových aplikací má společné rozložení, které uživatelům poskytuje konzistentní prostředí při přechodu ze stránky na stránku. Rozložení obvykle zahrnuje běžné prvky uživatelského rozhraní, jako jsou záhlaví aplikace, navigace nebo prvky nabídky a zápatí.

![Příklad rozložení stránky](layout/_static/page-layout.png)

Společné struktury HTML, jako jsou skripty a šablony stylů, často používá mnoho stránek v rámci aplikace. Všechny tyto sdílené prvky mohou být definovány v souboru *rozložení* , na který lze následně odkazovat jakýmkoli zobrazením použitým v aplikaci. Rozložení omezují duplicitní kód v zobrazeních.

Podle konvence je výchozím rozložením pro aplikaci ASP.NET Core název *_Layout. cshtml*. Soubory rozložení pro nové projekty ASP.NET Core vytvořené pomocí šablon jsou:

* Razor Pages: *Pages/Shared/_Layout. cshtml*

  ![Složka stránky v Průzkumník řešení](layout/_static/rp-web-project-views.png)

* Kontroler se zobrazeními: *views/Shared/_Layout. cshtml*

  ![Složka zobrazení v Průzkumník řešení](layout/_static/mvc-web-project-views.png)

Rozložení definuje šablonu na nejvyšší úrovni pro zobrazení v aplikaci. Aplikace nevyžadují rozložení. Aplikace mohou definovat více než jedno rozložení s různými zobrazeními, která určují různá rozložení.

Následující kód ukazuje soubor rozložení pro šablonu vytvořenou v projektu pomocí kontroleru a zobrazení:

[!code-cshtml[](~/common/samples/WebApplication1/Views/Shared/_Layout.cshtml?highlight=44,72)]

## <a name="specifying-a-layout"></a>Určení rozložení

Zobrazení Razor mají vlastnost `Layout`. Jednotlivá zobrazení určují rozložení nastavením této vlastnosti:

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewStart.cshtml?highlight=2)]

Zadané rozložení může použít úplnou cestu (například */Pages/Shared/_Layout. cshtml* nebo */views/Shared/_Layout. cshtml*) nebo částečný název (příklad: `_Layout`). Pokud je k dispozici částečný název, modul zobrazení Razor vyhledá soubor rozložení pomocí standardního procesu zjišťování. Složka, ve které existuje metoda obslužné rutiny (nebo kontrolér), je nejdříve prohledána a za ní následuje *sdílená* složka. Tento proces zjišťování je stejný jako proces, který se používá ke zjišťování [částečných zobrazení](xref:mvc/views/partial#partial-view-discovery).

Ve výchozím nastavení musí každé rozložení volat `RenderBody`. Všude, kde je umístěno volání `RenderBody`, se vykreslí obsah zobrazení.

<a name="layout-sections-label"></a>
<!-- https://stackoverflow.com/questions/23327578 -->
### <a name="sections"></a>Oddíly

Rozložení může volitelně odkazovat na jeden nebo více *oddílů*voláním `RenderSection`. Oddíly poskytují způsob, jak uspořádat, kde by měly být umístěny určité prvky stránky. Každé volání `RenderSection` může určit, zda je tato část povinná nebo volitelná:

```html
<script type="text/javascript" src="~/scripts/global.js"></script>

@RenderSection("Scripts", required: false)
```

Pokud není požadovaný oddíl nalezen, je vyvolána výjimka. Jednotlivá zobrazení určují obsah, který se má vykreslit v rámci oddílu, pomocí syntaxe Razor `@section`. Pokud stránka nebo zobrazení definují oddíl, je nutné jej vykreslit (nebo dojde k chybě).

Příklad definice `@section` v zobrazení Razor Pages:

```html
@section Scripts {
     <script type="text/javascript" src="~/scripts/main.js"></script>
}
```

V předchozím kódu se *skripty/Main. js* přidají do oddílu `scripts` na stránce nebo zobrazení. Jiné stránky nebo zobrazení ve stejné aplikaci pravděpodobně nevyžadují tento skript a nedefinují oddíl Scripts.

Následující kód používá [pomocníka částečné značky](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper) k vykreslování *_ValidationScriptsPartial. cshtml*:

```html
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

Předchozí kód byl vygenerován pomocí [identity generování uživatelského rozhraní](xref:security/authentication/scaffold-identity).

Oddíly definované na stránce nebo zobrazení jsou k dispozici pouze na stránce jejího okamžitého rozložení. Nelze na ně odkazovat z částečných, zobrazení součástí nebo jiných částí systému zobrazení.

### <a name="ignoring-sections"></a>Ignorují se oddíly.

Ve výchozím nastavení musí být tělo a všechny oddíly na stránce obsahu vykresleny na stránce rozložení. Tento modul zobrazení Razor vynutil sledování, zda text a jednotlivé oddíly byly vykresleny.

Chcete-li, aby modul zobrazení mohl ignorovat tělo nebo oddíly, zavolejte metody `IgnoreBody` a `IgnoreSection`.

Tělo a každý oddíl stránky Razor musí být buď vykreslené, nebo ignorovány.

<a name="viewimports"></a>

## <a name="importing-shared-directives"></a>Import sdílených direktiv

Zobrazení a stránky mohou pomocí direktiv Razor importovat obory názvů a používat [vkládání závislostí](dependency-injection.md). Direktivy sdílené pomocí mnoha zobrazení mohou být zadány v běžném souboru *_ViewImports. cshtml* . `_ViewImports` soubor podporuje následující direktivy:

* `@addTagHelper`
* `@removeTagHelper`
* `@tagHelperPrefix`
* `@using`
* `@model`
* `@inherits`
* `@inject`

Soubor nepodporuje jiné funkce Razor, jako jsou například definice funkcí a oddílů.

Vzorový `_ViewImports.cshtml` soubor:

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewImports.cshtml)]

Soubor *_ViewImports. cshtml* pro aplikaci ASP.NET Core MVC je obvykle umístěn ve složce *Pages* (nebo *views*). *_ViewImports soubor. cshtml* lze umístit do jakékoli složky. v takovém případě bude použito pouze na stránky nebo zobrazení v této složce a jejích podsložkách. soubory `_ViewImports` jsou zpracovávány na kořenové úrovni a následně pro každou složku, která je v umístění stránky nebo samotného zobrazení. nastavení `_ViewImports` zadaná na kořenové úrovni může být přepsána na úrovni složky.

Předpokládejme například:

* *_ViewImports soubor. cshtml* na kořenové úrovni zahrnuje `@model MyModel1` a `@addTagHelper *, MyTagHelper1`.
* Podsložka *_ViewImports soubor. cshtml* obsahuje `@model MyModel2` a `@addTagHelper *, MyTagHelper2`.

Stránky a zobrazení v podsložce budou mít přístup k oběma pomocníkům značek i k `MyModel2`mu modelu.

Pokud je v hierarchii souborů nalezeno více *_ViewImports. cshtml* souborů, kombinované chování direktiv jsou:

* `@addTagHelper``@removeTagHelper`: všechny spuštěné v daném pořadí
* `@tagHelperPrefix`: nejbližší z nich přepisuje všechny ostatní.
* `@model`: nejbližší z nich přepisuje všechny ostatní.
* `@inherits`: nejbližší z nich přepisuje všechny ostatní.
* `@using`: všechny jsou zahrnuté; duplicity se ignorují.
* `@inject`: pro každou vlastnost, která je nejblíže k zobrazení, potlačí všechny ostatní se stejným názvem vlastnosti.

<a name="viewstart"></a>

## <a name="running-code-before-each-view"></a>Spuštění kódu před každým zobrazením

Kód, který musí být spuštěn před každým zobrazením nebo stránkou, by měl být umístěn v souboru *_ViewStart. cshtml* . Podle konvence je soubor *_ViewStart. cshtml* umístěný ve složce *Pages* (nebo *views*). Příkazy uvedené v *_ViewStart. cshtml* jsou spouštěny před všemi úplnými zobrazeními (nikoli rozloženími, nikoli v částečných zobrazeních). Podobně jako [ViewImports. cshtml](xref:mvc/views/layout#viewimports)je *_ViewStart. cshtml* hierarchicky. Je-li soubor *_ViewStart. cshtml* definován ve složce zobrazení nebo stránky, bude spuštěn po jednom definovaném kořenu složky *stránky* (nebo *zobrazení*) (pokud existuje).

Ukázkový *_ViewStart soubor. cshtml* :

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewStart.cshtml)]

Výše uvedený soubor určuje, že všechna zobrazení budou používat rozvržení *_Layout. cshtml* .

*_ViewStart. cshtml* a *_ViewImports. cshtml* nejsou **obvykle umístěny** ve složce */Pages/Shared* (nebo */views/Shared*). Verze těchto souborů na úrovni aplikace by měly být umístěné přímo ve složce */Pages* (nebo */views*).
