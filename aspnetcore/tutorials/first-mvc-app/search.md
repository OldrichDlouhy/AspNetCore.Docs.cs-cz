---
title: Přidání vyhledávání do aplikace ASP.NET Core MVC
author: rick-anderson
description: Ukazuje, jak přidat hledání do základní ASP.NET Core aplikace MVC.
ms.author: riande
ms.date: 12/13/2018
uid: tutorials/first-mvc-app/search
ms.openlocfilehash: 97ee5f66c142780d54d28013c109da61241d967b
ms.sourcegitcommit: 2719c70cd15a430479ab4007ff3e197fbf5dfee0
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 08/09/2019
ms.locfileid: "68862948"
---
# <a name="add-search-to-an-aspnet-core-mvc-app"></a>Přidání vyhledávání do aplikace ASP.NET Core MVC

Podle [Rick Anderson](https://twitter.com/RickAndMSFT)

V této části přidáte funkci hledání do `Index` metody Action, která umožňuje hledat filmy podle *žánru* nebo *názvu*.

Aktualizujte metodu nalezenou v *Controllers/MoviesController. cs* pomocí následujícího kódu: `Index`

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/MoviesController.cs?name=snippet_1stSearch)]

První řádek `Index` metody akce vytvoří dotaz [LINQ](/dotnet/standard/using-linq) pro výběr filmů:

```csharp
var movies = from m in _context.Movie
             select m;
```

Dotaz je definován *pouze* v tomto okamžiku, nebyl proto spuštěn proti databázi.

`searchString` Pokud parametr obsahuje řetězec, dotaz na filmy se upraví tak, aby se vyfiltroval na hodnotu hledaného řetězce:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/MoviesController.cs?name=snippet_SearchNull2)]

Výše uvedený kód je [výraz lambda.](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions) `s => s.Title.Contains()` Výrazy lambda se používají v dotazech [LINQ](/dotnet/standard/using-linq) založených na metodách jako argumenty standardních metod operátoru dotazu, jako je `Contains` například metoda [WHERE](/dotnet/api/system.linq.enumerable.where) nebo (použitá v kódu výše). Dotazy LINQ nejsou provedeny při jejich definování nebo při jejich úpravě voláním metody `Where`, jako je, `Contains`nebo `OrderBy`. Místo toho je provádění dotazu odloženo.  To znamená, že vyhodnocení výrazu je zpožděno, dokud se na jeho realizované hodnotě neprovádí iterace nebo `ToListAsync` je volána metoda. Další informace o odložení provádění dotazů naleznete v tématu [provádění dotazů](/dotnet/framework/data/adonet/ef/language-reference/query-execution).

Poznámka: Metoda [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) je spuštěna v databázi, nikoli v kódu jazyka c# zobrazeném výše. Rozlišování velkých a malých písmen na dotazu závisí na databázi a kolaci. V SQL Server [obsahuje](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) mapy [jako SQL](/sql/t-sql/language-elements/like-transact-sql), což rozlišuje velká a malá písmena. V SQLite s výchozí kolací rozlišuje velká a malá písmena.

Přejděte na adresu `/Movies/Index`. Připojit řetězec dotazu, například `?searchString=Ghost` k adrese URL. Zobrazí se filtrované filmy.

![Zobrazení indexu](~/tutorials/first-mvc-app/search/_static/ghost.png)

Změníte `Index` -li podpis metody tak, aby měl parametr s názvem `id`, `id` parametr bude odpovídat volitelnému `{id}` zástupnému symbolu pro výchozí trasy nastavené v *Startup.cs*.

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?highlight=5&name=snippet_1)]

Změňte parametr na `id` a všechny výskyty `searchString` změny na `id`.

Předchozí `Index` metoda:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/MoviesController.cs?highlight=1,6,8&name=snippet_1stSearch)]

Aktualizovaná `Index` metoda s `id` parametrem:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/MoviesController.cs?highlight=1,6,8&name=snippet_SearchID)]

Nyní můžete název hledání předat jako data trasy (segment adresy URL) místo jako hodnotu řetězce dotazu.

![Zobrazení indexu s slovem Ghost, které bylo přidáno k adrese URL a vráceným seznamem filmů dvou filmů, Ghostbusters a Ghostbusters 2](~/tutorials/first-mvc-app/search/_static/g2.png)

Nemůžete ale očekávat, že uživatelé můžou upravovat adresu URL pokaždé, když chtějí hledat film. Takže teď přidáte prvky uživatelského rozhraní, které jim pomůžou filtrovat filmy. Pokud jste změnili signaturu `Index` metody, abyste otestovali, jak předat parametr vázaný `ID` na trase, změňte ho tak, aby přebírá parametr s `searchString`názvem:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/MoviesController.cs?highlight=1,6,8&name=snippet_1stSearch)]

Otevřete soubor *views/Movies/index. cshtml* a přidejte následující `<form>` kód, který je zvýrazněný níže:

[!code-HTML[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/Movies/IndexForm1.cshtml?highlight=10-16&range=4-21)]

Značka HTML `<form>` používá pomocníka [značek Form](xref:mvc/views/working-with-forms), takže když formulář odešlete, vystaví se řetězec filtru do `Index` akce kontroleru filmů. Uložte změny a potom filtr otestujte.

![Zobrazení indexu se slovem Ghost, které jste zadali do textového pole filtru nadpisu](~/tutorials/first-mvc-app/search/_static/filter.png)

`[HttpPost]` Neexistuje přetížení `Index` metody, jak by bylo možné očekávat. Nepotřebujete ho, protože metoda nemění stav aplikace, stačí data filtrovat.

Můžete přidat následující `[HttpPost] Index` metodu.

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/MoviesController.cs?highlight=1&name=snippet_SearchPost)]

Parametr se používá k vytvoření přetížení `Index` pro metodu. `notUsed` O tom budeme mluvit později v tomto kurzu.

Pokud přidáte tuto metodu, akce původce by odpovídala `[HttpPost] Index` metodě a metoda se `[HttpPost] Index` spustí, jak je znázorněno na následujícím obrázku.

![Okno prohlížeče s odpovědí na aplikaci z indexu HttpPost: filtrování na Ghost](~/tutorials/first-mvc-app/search/_static/fo.png)

Nicméně i v případě, že přidáte `[HttpPost]` tuto verzi `Index` metody, existuje omezení, jak je vše implementováno. Představte si, že chcete vytvořit záložku konkrétního hledání nebo chcete poslat odkaz na přátele, na které můžou kliknout, aby se zobrazil stejný filtrovaný seznam filmů. Všimněte si, že adresa URL požadavku HTTP POST je stejná jako adresa URL žádosti GET (localhost: {PORT}/video/index) – v adrese URL nejsou žádné vyhledávací informace. Informace o hledaném řetězci jsou odesílány na server jako [hodnota pole formuláře](https://developer.mozilla.org/docs/Learn/HTML/Forms/Sending_and_retrieving_form_data). Můžete si je ověřit pomocí nástrojů pro vývojáře v prohlížeči nebo skvělého [nástroje Fiddler](https://www.telerik.com/fiddler). Následující obrázek ukazuje vývojářské nástroje prohlížeče Chrome:

![Karta síť Vývojářské nástroje v Microsoft Edge ukazující tělo žádosti s hodnotou searchString Ghost](~/tutorials/first-mvc-app/search/_static/f12_rb.png)

V textu žádosti vidíte parametr Search a token [XSRF](xref:security/anti-request-forgery) . Všimněte si, jak je uvedeno v předchozím kurzu [Pomocník značek formuláře](xref:mvc/views/working-with-forms) generuje token odolný proti falšování [XSRF](xref:security/anti-request-forgery) . Neupravujeme data, takže v metodě kontroleru nepotřebujeme token ověřit.

Vzhledem k tomu, že parametr Search je v textu žádosti, a ne v adrese URL, nemůžete zachytit informace o hledání do záložky nebo sdílet s ostatními. Opravte to zadáním požadavku, který `HTTP GET` se má najít v souboru views/ *Movies/index. cshtml* .

[!code-html[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexGet.cshtml?highlight=12&range=1-23)]

Když teď odešlete hledání, adresa URL obsahuje řetězec vyhledávacího dotazu. Hledání bude také jít na `HttpGet Index` metodu Action, i když `HttpPost Index` máte metodu.

![Okno prohlížeče zobrazující v adrese URL searchString = Ghost a vracené filmy, Ghostbusters a Ghostbusters 2, obsahují slovo Ghost](~/tutorials/first-mvc-app/search/_static/search_get.png)

Následující kód ukazuje změnu na `form` značku:

```html
<form asp-controller="Movies" asp-action="Index" method="get">
   ```

## <a name="add-search-by-genre"></a>Přidat hledání podle žánru

Přidejte následující `MovieGenreViewModel` třídu *modely* složky:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Models/MovieGenreViewModel.cs)]

Model zobrazení filmového žánru bude obsahovat:

* Seznam filmů.
* `SelectList` Obsahující seznam žánrů. To umožňuje uživateli vybrat ze seznamu Žánr.
* `MovieGenre`, který obsahuje vybraný Žánr.
* `SearchString`, který obsahuje text uživatelé, zadejte do textového pole Hledat.

Nahraďte `MoviesController.cs` metodu v následujícím kódem: `Index`

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MoviesController.cs?name=snippet_SearchGenre)]

Následující kód je `LINQ` dotaz, který načte všechny žánry z databáze.

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MoviesController.cs?name=snippet_LINQ)]

`SelectList` Žánry se vytvářejí vyvoláním různých žánrů (nechceme, aby seznam pro výběr měl duplicitní žánry).

Když uživatel vyhledá položku, vyhledávací hodnota se zachová do vyhledávacího pole.

## <a name="add-search-by-genre-to-the-index-view"></a>Přidat hledání podle žánru do zobrazení indexu

Aktualizace `Index.cshtml` se našla v *zobrazeních/filmech, a* to následujícím způsobem:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexFormGenreNoRating.cshtml?highlight=1,15,16,17,19,28,31,34,37,43)]

Prověřte výraz lambda použitý v následujícím Pomocníkovi HTML:

`@Html.DisplayNameFor(model => model.Movies[0].Title)`

V předchozím kódu `DisplayNameFor` pomocník HTML kontroluje `Title` vlastnost, na kterou se odkazuje ve výrazu lambda pro určení zobrazovaného názvu. Vzhledem k tomu, že je výraz lambda zkontrolován `model`, a `model.Movies[0]` nikoli vyhodnocen, neobdržíte porušení `model.Movies`přístupu, pokud `null` jsou nebo prázdné. Při vyhodnocování výrazu lambda (například `@Html.DisplayFor(modelItem => item.Title)`) jsou vyhodnocovány hodnoty vlastností modelu.

Otestujte aplikaci vyhledáním podle žánru, podle názvu filmu a pomocí obou:

![Okno prohlížeče zobrazující výsledky https://localhost:5001/Movies?MovieGenre=Comedy&SearchString=2](~/tutorials/first-mvc-app/search/_static/s2.png)

> [!div class="step-by-step"]
> [Předchozí](controller-methods-views.md)
> [Další](new-field.md)
