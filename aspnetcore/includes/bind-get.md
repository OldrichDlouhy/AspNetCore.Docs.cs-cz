> [!WARNING]
> Z bezpečnostních důvodů musíte vyjádřit výslovný souhlas s vazbou `GET` na data požadavku na vlastnosti modelu stránky. Před mapováním na vlastnosti ověřte vstup uživatele. Přizpůsobování `GET` vazby je užitečné při adresování scénářů, které spoléhají na řetězce dotazů nebo hodnoty tras.
>
> Chcete-li vytvořit navázání vlastnosti u `GET` požadavků, nastavte `SupportsGet` vlastnost atributu [[BindProperty]](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) na: `true`
>
> ```csharp
> [BindProperty(SupportsGet = true)]
> ```
>
> Další informace najdete v tématu [ASP.NET Core Community rychlou schůzku: Vazba na GET diskuze (YouTube](https://www.youtube.com/watch?v=p7iHB9V-KVU&feature=youtu.be&t=54m27s))
