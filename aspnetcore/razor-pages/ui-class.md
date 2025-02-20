---
title: Opakovaně použitelné uživatelské rozhraní Razor v knihovnách tříd s ASP.NET Core
author: Rick-Anderson
description: Vysvětluje, jak vytvořit opakovaně použitelné uživatelské rozhraní Razor pomocí částečných zobrazení v knihovně tříd v ASP.NET Core.
ms.author: riande
ms.date: 10/26/2019
ms.custom: mvc, seodec18
uid: razor-pages/ui-class
ms.openlocfilehash: ff12eea5406c4f5392a466728741000e3dd16fc1
ms.sourcegitcommit: 16cf016035f0c9acf3ff0ad874c56f82e013d415
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/29/2019
ms.locfileid: "73034223"
---
# <a name="create-reusable-ui-using-the-razor-class-library-project-in-aspnet-core"></a>Vytvoření opakovaně použitelného uživatelského rozhraní pomocí projektu knihovny tříd Razor v ASP.NET Core

Od [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

Zobrazení Razor, stránky, řadiče, modely stránek, [komponenty Razor](xref:blazor/class-libraries), [zobrazení komponent](xref:mvc/views/view-components)a datové modely lze integrovat do knihovny tříd Razor (RCL). RCL může být zabalen a znovu použit. Aplikace mohou zahrnovat RCL a přepsat zobrazení a stránky, které obsahuje. V případě, že je v rámci webové aplikace i v RCL Nalezeno zobrazení, částečné zobrazení nebo stránka Razor, má přednost označení Razor (soubor *. cshtml* ) ve webové aplikaci.

[Zobrazit nebo stáhnout ukázkový kód](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/ui-class/samples) ([Jak stáhnout](xref:index#how-to-download-a-sample))

## <a name="create-a-class-library-containing-razor-ui"></a>Vytvoření knihovny tříd obsahující uživatelské rozhraní Razor

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* V aplikaci Visual Studio vyberte **vytvořit nový nový projekt**.
* > **Další**vyberte **knihovnu tříd Razor** .
* Pojmenujte knihovnu (například "RazorClassLib"), > **vytvořit**. Aby nedošlo ke kolizi názvů souborů s vygenerovanou knihovnou zobrazení, ujistěte se, že název knihovny nekončí `.Views`.
* Pokud potřebujete podporovat zobrazení, vyberte možnost **stránky podpory a zobrazení** . Ve výchozím nastavení jsou podporovány pouze Razor Pages. Vyberte **vytvořit**.

Šablona knihovny tříd Razor (RCL) je ve výchozím nastavení standardně pro vývoj komponent Razor. Možnost **stránky podpory a zobrazení** podporuje stránky a zobrazení.

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli)

Z příkazového řádku spusťte `dotnet new razorclasslib`. Příklad:

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
```

Šablona knihovny tříd Razor (RCL) je ve výchozím nastavení standardně pro vývoj komponent Razor. Pokud chcete poskytovat podporu pro stránky a zobrazení, předejte možnost `--support-pages-and-views` (`dotnet new razorclasslib --support-pages-and-views`).

Další informace najdete v tématu [dotnet New](/dotnet/core/tools/dotnet-new). Aby nedošlo ke kolizi názvů souborů s vygenerovanou knihovnou zobrazení, ujistěte se, že název knihovny nekončí `.Views`.

---

Přidejte soubory Razor do RCL.

Šablony ASP.NET Core předpokládají, že obsah RCL je ve složce *oblasti* . Pokud chcete vytvořit RCL, který zpřístupňuje obsah v `~/Pages` místo `~/Areas/Pages`, přečtěte si téma [rozložení stránek RCL](#rcl-pages-layout) .

## <a name="reference-rcl-content"></a>Odkaz na obsah RCL

Na RCL může odkazovat:

* Balíček NuGet. Viz téma [vytváření balíčků NuGet](/nuget/create-packages/creating-a-package) a [dotnet přidání balíčku](/dotnet/core/tools/dotnet-add-package) a [Vytvoření a publikování balíčku NuGet](/nuget/quickstart/create-and-publish-a-package-using-visual-studio).
* *{ProjectName}. csproj*. Viz [dotnet – přidat odkaz](/dotnet/core/tools/dotnet-add-reference).

## <a name="override-views-partial-views-and-pages"></a>Přepsat zobrazení, částečná zobrazení a stránky

V případě, že je v rámci webové aplikace i v RCL Nalezeno zobrazení, částečné zobrazení nebo stránka Razor, má přednost označení Razor (soubor *. cshtml* ) ve webové aplikaci. Například přidejte *WebApp1/areas/MyFeature/Pages/Page1. cshtml* do WebApp1 a Page1 v WebApp1 bude mít přednost před Page1 v RCL.

V ukázce stažení, přejmenujte *WebApp1/areas/MyFeature2* na *WebApp1/areas/MyFeature* na test priority.

Zkopírujte částečné zobrazení *RazorUIClassLib/areas/MyFeature/Pages/Shared/_Message. cshtml* na *WebApp1/areas/MyFeature/Pages/Shared/_Message. cshtml*. Aktualizujte kód tak, aby označoval nové umístění. Sestavte a spusťte aplikaci, abyste ověřili, že se používá částečná verze aplikace.

### <a name="rcl-pages-layout"></a>Rozložení stránek RCL

Pokud chcete odkazovat na obsah RCL, jako by byl *součástí složky webových* aplikací, vytvořte projekt RCL s následující strukturou souborů:

* *RazorUIClassLib/stránky*
* *RazorUIClassLib/stránky/sdílené*

Předpokládejme, že *RazorUIClassLib/Pages/Shared* obsahují dva částečné soubory: *_Header. cshtml* a *_Footer. cshtml*. Do souboru *_Layout. cshtml* se dají přidat značky `<partial>`:

```cshtml
<body>
  <partial name="_Header">
  @RenderBody()
  <partial name="_Footer">
</body>
```

## <a name="create-an-rcl-with-static-assets"></a>Vytvoření RCL se statickými prostředky

RCL může vyžadovat doprovodné statické prostředky, které mohou být odkazovány využitím aplikace RCL. ASP.NET Core umožňuje vytváření RCLs, které zahrnují statické prostředky, které jsou k dispozici pro náročné aplikace.

Chcete-li zahrnout doprovodné materiály jako součást RCL, vytvořte složku *wwwroot* v knihovně tříd a zahrňte všechny požadované soubory do této složky.

Při balení RCL jsou do balíčku automaticky zahrnuty všechny doprovodné materiály ve složce *wwwroot* .

### <a name="exclude-static-assets"></a>Vyloučení statických prostředků

Chcete-li vyloučit statické prostředky, přidejte požadovanou cestu vyloučení do skupiny vlastností `$(DefaultItemExcludes)` v souboru projektu. Jednotlivé položky oddělte středníkem (`;`).

V následujícím příkladu není šablona *lib. CSS* ve složce *wwwroot* považována za statický prostředek a není obsažena v publikovaném RCL:

```xml
<PropertyGroup>
  <DefaultItemExcludes>$(DefaultItemExcludes);wwwroot\lib.css</DefaultItemExcludes>
</PropertyGroup>
```

### <a name="typescript-integration"></a>Integrace TypeScriptu

Zahrnutí souborů TypeScriptu do RCL:

1. Soubory TypeScript ( *. TS*) umístěte mimo složku *wwwroot* . Umístěte například soubory do složky *klienta* .

1. Nakonfigurujte výstup sestavení TypeScriptu pro složku *wwwroot* . Nastavte vlastnost `TypescriptOutDir` uvnitř `PropertyGroup` v souboru projektu:

   ```xml
   <TypescriptOutDir>wwwroot</TypescriptOutDir>
   ```

1. Přidejte cíl TypeScript jako závislost `ResolveCurrentProjectStaticWebAssets` a přidejte následující cíl do `PropertyGroup` v souboru projektu:

   ```xml
   <ResolveCurrentProjectStaticWebAssetsInputsDependsOn>
     CompileTypeScript;
     $(ResolveCurrentProjectStaticWebAssetsInputs)
   </ResolveCurrentProjectStaticWebAssetsInputsDependsOn>
   ```

### <a name="consume-content-from-a-referenced-rcl"></a>Využití obsahu z odkazovaného RCL

Soubory zahrnuté ve složce *wwwroot* v RCL jsou zpřístupněné aplikacím, které jsou v předponě `_content/{LIBRARY NAME}/`. Například knihovna s názvem *Razor. Class. lib* má za následek cestu k statickému obsahu v `_content/Razor.Class.Lib/`.

Vybírající aplikace odkazuje na statické prostředky poskytované knihovnou s `<script>`, `<style>`, `<img>` a dalšími značkami HTML. Vybírající aplikace musí mít povolenou [podporu statických souborů](xref:fundamentals/static-files) v `Startup.Configure`:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseStaticFiles();

    ...
}
```

Při spuštění náročné aplikace z výstupu sestavení (`dotnet run`) jsou ve výchozím nastavení ve vývojovém prostředí povoleny statické webové prostředky. Pro podporu prostředků v jiných prostředích při spuštění z výstupu sestavení volejte `UseStaticWebAssets` na tvůrci hostitele v *program.cs*:

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStaticWebAssets();
                webBuilder.UseStartup<Startup>();
            });
}
```

Při spuštění aplikace z publikovaného výstupu se nevyžaduje volání `UseStaticWebAssets` (`dotnet publish`).

### <a name="multi-project-development-flow"></a>Vývojový tok pro více projektů

Když pracujete na náročné aplikaci:

* Prostředky v RCL zůstanou v původních složkách. Prostředky se nepřesunou do nenáročné aplikace.
* Jakákoli změna ve složce *WWWROOT* RCL se projeví v aplikaci po resestavení RCL a bez opětovného sestavení náročné aplikace.

Při sestavení RCL je vytvořen manifest, který popisuje umístění statických webových prostředků. Vybírající aplikace načte manifest za běhu za účelem využití prostředků z odkazovaných projektů a balíčků. Když se do RCL přidá nový Asset, musí se RCL znovu sestavit, aby se aktualizoval jeho manifest předtím, než bude mít k novému assetu přístup.

### <a name="publish"></a>Opětovn

Po publikování aplikace se doprovodné prostředky ze všech odkazovaných projektů a balíčků zkopírují do složky *wwwroot* publikované aplikace v části `_content/{LIBRARY NAME}/`.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Zobrazení Razor, stránky, řadiče, modely stránek, [komponenty Razor](xref:blazor/class-libraries), [zobrazení komponent](xref:mvc/views/view-components)a datové modely lze integrovat do knihovny tříd Razor (RCL). RCL může být zabalen a znovu použit. Aplikace mohou zahrnovat RCL a přepsat zobrazení a stránky, které obsahuje. V případě, že je v rámci webové aplikace i v RCL Nalezeno zobrazení, částečné zobrazení nebo stránka Razor, má přednost označení Razor (soubor *. cshtml* ) ve webové aplikaci.

[Zobrazit nebo stáhnout ukázkový kód](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/ui-class/samples) ([Jak stáhnout](xref:index#how-to-download-a-sample))

## <a name="create-a-class-library-containing-razor-ui"></a>Vytvoření knihovny tříd obsahující uživatelské rozhraní Razor

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* V nabídce **soubor** v aplikaci Visual Studio vyberte **Nový** **projekt**>.
* Vyberte **ASP.NET Core webové aplikace**.
* Pojmenujte knihovnu (například "RazorClassLib") > **OK**. Aby nedošlo ke kolizi názvů souborů s vygenerovanou knihovnou zobrazení, ujistěte se, že název knihovny nekončí `.Views`.
* Ověřte, zda je vybrána **ASP.NET Core 2,1** nebo novější.
* Vyberte **knihovnu tříd Razor** > **OK**.

RCL má následující soubor projektu:

[!code-xml[](ui-class/samples/cli/RazorUIClassLib/RazorUIClassLib.csproj)]

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli)

Z příkazového řádku spusťte `dotnet new razorclasslib`. Příklad:

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
```

Další informace najdete v tématu [dotnet New](/dotnet/core/tools/dotnet-new). Aby nedošlo ke kolizi názvů souborů s vygenerovanou knihovnou zobrazení, ujistěte se, že název knihovny nekončí `.Views`.

---

Přidejte soubory Razor do RCL.

Šablony ASP.NET Core předpokládají, že obsah RCL je ve složce *oblasti* . Pokud chcete vytvořit RCL, který zpřístupňuje obsah v `~/Pages` místo `~/Areas/Pages`, přečtěte si téma [rozložení stránek RCL](#rcl-pages-layout) .

## <a name="reference-rcl-content"></a>Odkaz na obsah RCL

Na RCL může odkazovat:

* Balíček NuGet. Viz téma [vytváření balíčků NuGet](/nuget/create-packages/creating-a-package) a [dotnet přidání balíčku](/dotnet/core/tools/dotnet-add-package) a [Vytvoření a publikování balíčku NuGet](/nuget/quickstart/create-and-publish-a-package-using-visual-studio).
* *{ProjectName}. csproj*. Viz [dotnet – přidat odkaz](/dotnet/core/tools/dotnet-add-reference).

## <a name="walkthrough-create-an-rcl-project-and-use-from-a-razor-pages-project"></a>Návod: vytvoření projektu RCL a použití z Razor Pages projektu

Můžete stáhnout [celý projekt](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/ui-class/samples) a otestovat ho místo jeho vytvoření. Ukázkový soubor ke stažení obsahuje další kód a odkazy, které usnadňují testování projektu. Zpětnou vazbu k [tomuto problému GitHubu](https://github.com/aspnet/AspNetCore.Docs/issues/6098) si můžete nechat s vašimi komentáři ke stažení ukázek a podrobnými pokyny.

### <a name="test-the-download-app"></a>Testování aplikace pro stažení

Pokud jste nestáhli dokončenou aplikaci a chcete místo toho vytvořit návodový projekt, přejděte k [Další části](#create-an-rcl).

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Otevřete soubor *. sln* v aplikaci Visual Studio. Spusťte aplikaci.

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli)

Z příkazového řádku v adresáři *CLI* Sestavte RCL a webovou aplikaci.

```dotnetcli
dotnet build
```

Přejděte do adresáře *WebApp1* a spusťte aplikaci:

```dotnetcli
dotnet run
```

---

Postupujte podle pokynů v části [test WebApp1](#test-webapp1)

## <a name="create-an-rcl"></a>Vytvoření RCL

V této části se vytvoří RCL. Soubory Razor jsou přidány do RCL.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Vytvořte projekt RCL:

* V nabídce **soubor** v aplikaci Visual Studio vyberte **Nový** **projekt**>.
* Vyberte **ASP.NET Core webové aplikace**.
* Pojmenujte aplikaci **RazorUIClassLib** > **OK**.
* Ověřte, zda je vybrána **ASP.NET Core 2,1** nebo novější.
* Vyberte **knihovnu tříd Razor** > **OK**.
* Přidejte soubor částečného zobrazení Razor s názvem *RazorUIClassLib/areas/MyFeature/Pages/Shared/_Message. cshtml*.

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli)

Z příkazového řádku spusťte následující příkaz:

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
dotnet new page -n _Message -np -o RazorUIClassLib/Areas/MyFeature/Pages/Shared
dotnet new viewstart -o RazorUIClassLib/Areas/MyFeature/Pages
```

Předchozí příkazy:

* Vytvoří `RazorUIClassLib` RCL.
* Vytvoří stránku _Message Razor a přidá ji do RCL. Parametr `-np` vytvoří stránku bez `PageModel`.
* Vytvoří soubor [_ViewStart. cshtml](xref:mvc/views/layout#running-code-before-each-view) a přidá ho do RCL.

K použití rozložení Razor Pagesho projektu (který je přidán v další části) je vyžadován soubor *_ViewStart. cshtml* .

---

### <a name="add-razor-files-and-folders-to-the-project"></a>Přidat soubory a složky Razor do projektu

* Nahraďte označení v *RazorUIClassLib/areas/MyFeature/Pages/Shared/_Message. cshtml* následujícím kódem:

  [!code-cshtml[](ui-class/samples/cli/RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml)]

* Nahraďte označení v *RazorUIClassLib/areas/MyFeature/Pages/Page1. cshtml* následujícím kódem:

  [!code-cshtml[](ui-class/samples/cli/RazorUIClassLib/Areas/MyFeature/Pages/Page1.cshtml)]

  `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` se vyžaduje pro použití částečného zobrazení (`<partial name="_Message" />`). Místo zahrnutí direktivy `@addTagHelper` můžete přidat soubor *_ViewImports. cshtml* . Příklad:

  ```dotnetcli
  dotnet new viewimports -o RazorUIClassLib/Areas/MyFeature/Pages
  ```

  Další informace o *_ViewImports. cshtml*najdete v tématu [Import sdílených direktiv](xref:mvc/views/layout#importing-shared-directives) .

* Sestavení knihovny tříd pro ověření, že nejsou k dispozici žádné chyby kompilátoru:

  ```dotnetcli
  dotnet build RazorUIClassLib
  ```

Výstup sestavení obsahuje *RazorUIClassLib. dll* a *RazorUIClassLib. views. dll*. *RazorUIClassLib. views. dll* obsahuje zkompilovaný obsah Razor.

### <a name="use-the-razor-ui-library-from-a-razor-pages-project"></a>Použití knihovny uživatelského rozhraní Razor z Razor Pagesho projektu

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Vytvoření webové aplikace Razor Pages:

* V **Průzkumník řešení**klikněte pravým tlačítkem na řešení > **přidat** **Nový projekt**>.
* Vyberte **ASP.NET Core webové aplikace**.
* Pojmenujte aplikaci **WebApp1**.
* Ověřte, zda je vybrána **ASP.NET Core 2,1** nebo novější.
* Vyberte možnost **Webová aplikace** > **OK**.

* Z **Průzkumník řešení**klikněte pravým tlačítkem na **WebApp1** a vyberte **nastavit jako spouštěný projekt**.
* V **Průzkumník řešení**klikněte pravým tlačítkem na **WebApp1** a vyberte závislosti **sestavení** > **závislosti projektu**.
* Ověřte **RazorUIClassLib** jako závislost **WebApp1**.
* V **Průzkumník řešení**klikněte pravým tlačítkem na **WebApp1** a vyberte **Přidat** **odkaz**>.
* V dialogovém okně **Správce odkazů** ověřte **RazorUIClassLib** > **OK**.

Spusťte aplikaci.

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli)

Vytvoření webové aplikace Razor Pages a souboru řešení obsahujícího aplikaci Razor Pages a RCL:

```dotnetcli
dotnet new webapp -o WebApp1
dotnet new sln
dotnet sln add WebApp1
dotnet sln add RazorUIClassLib
dotnet add WebApp1 reference RazorUIClassLib
```

Sestavte a spusťte webovou aplikaci:

```dotnetcli
cd WebApp1
dotnet run
```

---

### <a name="test-webapp1"></a>WebApp1 testu

Přejděte na `/MyFeature/Page1` a ověřte, zda se knihovna tříd uživatelského rozhraní Razor používá.

## <a name="override-views-partial-views-and-pages"></a>Přepsat zobrazení, částečná zobrazení a stránky

V případě, že je v rámci webové aplikace i v RCL Nalezeno zobrazení, částečné zobrazení nebo stránka Razor, má přednost označení Razor (soubor *. cshtml* ) ve webové aplikaci. Například přidejte *WebApp1/areas/MyFeature/Pages/Page1. cshtml* do WebApp1 a Page1 v WebApp1 bude mít přednost před Page1 v RCL.

V ukázce stažení, přejmenujte *WebApp1/areas/MyFeature2* na *WebApp1/areas/MyFeature* na test priority.

Zkopírujte částečné zobrazení *RazorUIClassLib/areas/MyFeature/Pages/Shared/_Message. cshtml* na *WebApp1/areas/MyFeature/Pages/Shared/_Message. cshtml*. Aktualizujte kód tak, aby označoval nové umístění. Sestavte a spusťte aplikaci, abyste ověřili, že se používá částečná verze aplikace.

### <a name="rcl-pages-layout"></a>Rozložení stránek RCL

Pokud chcete odkazovat na obsah RCL, jako by byl *součástí složky webových* aplikací, vytvořte projekt RCL s následující strukturou souborů:

* *RazorUIClassLib/stránky*
* *RazorUIClassLib/stránky/sdílené*

Předpokládejme, že *RazorUIClassLib/Pages/Shared* obsahují dva částečné soubory: *_Header. cshtml* a *_Footer. cshtml*. Do souboru *_Layout. cshtml* se dají přidat značky `<partial>`:

```cshtml
<body>
  <partial name="_Header">
  @RenderBody()
  <partial name="_Footer">
</body>
```

::: moniker-end
