---
title: Pomocná pomůcka značky v ASP.NET Core
author: rick-anderson
ms.author: riande
description: Seznamte se s atributy pomocníka značek ASP.NET Core odkazů a role, kterou každý atribut hraje při rozšiřování chování značky odkazu HTML.
ms.custom: mvc
ms.date: 09/24/2019
uid: mvc/views/tag-helpers/builtin-th/link-tag-helper
ms.openlocfilehash: e1e2e58b4ab9087e1f9de5b5c03b587feb88f1b9
ms.sourcegitcommit: fae6f0e253f9d62d8f39de5884d2ba2b4b2a6050
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 09/25/2019
ms.locfileid: "71256517"
---
# <a name="link-tag-helper-in-aspnet-core"></a>Pomocná pomůcka značky v ASP.NET Core

Podle [Rick Anderson](https://twitter.com/RickAndMSFT)

[Pomocník značek odkazů](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper) generuje odkaz na primární nebo vrácený soubor CSS. Primární soubor CSS je obvykle na [Content Delivery Network](/office365/enterprise/content-delivery-networks#what-exactly-is-a-cdn) (CDN).

[!INCLUDE[](~/includes/cdn.md)]

Pomocník značek odkazů umožňuje zadat CDN pro soubor CSS a zálohu, pokud CDN není k dispozici. Pomocník značek odkazů nabízí výkonnou výhodu sítě CDN s odolností místního hostování.

Následující kód Razor ukazuje `head` prvek souboru rozložení vytvořeného pomocí šablony ASP.NET Core webové aplikace:

[!code-html[](link-tag-helper/sample/_Layout.cshtml?name=snippet)]

Následující kód vykreslí kód HTML z předchozího kódu (v nevývojovém prostředí):

[!code-csharp[](link-tag-helper/sample/HtmlPage1.html)]

V předchozím kódu pomocník značek odkazů vygeneroval `<meta name="x-stylesheet-fallback-test" content="" class="sr-only" />` prvek a následující JavaScript, který slouží k ověření požadovaného souboru *bootstrap. min. CSS* , je k dispozici v CDN. V tomto případě byl k dispozici soubor CSS, aby pomocník značek vygeneroval `<link />` prvek se souborem. CSS CDN.

## <a name="commonly-used-link-tag-helper-attributes"></a>Běžně používané pomocné atributy značek odkazů

U všech pomocných atributů, vlastností a metod odkazů můžete zobrazit nápovědu pro [značku odkazu](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper) .

### <a name="href"></a>href

Upřednostňovaná adresa propojeného prostředku Adresa je předána na vygenerovaný kód HTML ve všech případech.

### <a name="asp-fallback-href"></a>ASP – Fallback – href

Adresa URL šablony stylů CSS, na kterou se má přejít v případě, že primární adresa URL není úspěšná

### <a name="asp-fallback-test-class"></a>ASP – Fallback-test-třída

Název třídy definovaný v šabloně stylů, která se má použít pro záložní test. Další informace naleznete v tématu <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestClass>.

### <a name="asp-fallback-test-property"></a>ASP – Fallback-test-vlastnost

Název vlastnosti CSS, který má být použit pro záložní test. Další informace naleznete v tématu <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestProperty>.

### <a name="asp-fallback-test-value"></a>ASP – Fallback-test-hodnota

Hodnota vlastnosti CSS, která má být použita pro záložní test. Další informace naleznete v tématu <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestValue>.

### <a name="asp-fallback-test-value"></a>ASP – Fallback-test-hodnota

Hodnota vlastnosti CSS, která má být použita pro záložní test. Další informace najdete v tématu.<xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestValue>

## <a name="additional-resources"></a>Další zdroje

* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
