---
title: Použití šablony úhlového projektu s ASP.NET Core
author: SteveSandersonMS
description: Naučte se, jak začít pracovat s ASP.NET Core šablonou projektu s jednou stránkou (SPA) pro úhlové a úhlové rozhraní příkazového řádku.
monikerRange: '>= aspnetcore-2.1'
ms.author: stevesa
ms.custom: mvc
ms.date: 03/07/2019
uid: spa/angular
ms.openlocfilehash: 62654ca040be99de8063a63c7e4ac09cbb8564eb
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 09/18/2019
ms.locfileid: "71080406"
---
# <a name="use-the-angular-project-template-with-aspnet-core"></a>Použití šablony úhlového projektu s ASP.NET Core

Aktualizovaná šablona úhlového projektu poskytuje pohodlný výchozí bod pro ASP.NET Core aplikace pomocí úhlů a úhlů CLI pro implementaci bohatě funkčního uživatelského rozhraní (UI) na straně klienta.

Šablona je ekvivalentní k vytvoření ASP.NET Core projektu, který se má chovat jako back-end rozhraní API, a s úhlovým projektem CLI, který bude fungovat jako uživatelské rozhraní. Šablona nabízí pohodlí hostování obou typů projektů v jednom projektu aplikace. V důsledku toho může být projekt aplikace sestaven a publikován jako jediná jednotka.

## <a name="create-a-new-app"></a>Vytvoření nové aplikace

Pokud máte nainstalovanou ASP.NET Core 2,1, není potřeba instalovat úhlovou šablonu projektu.

Vytvoří nový projekt z příkazového řádku pomocí příkazu `dotnet new angular` v prázdném adresáři. Například následující příkazy vytvoří aplikaci v adresáři *My-New-App* a přepne do tohoto adresáře:

```dotnetcli
dotnet new angular -o my-new-app
cd my-new-app
```

Spusťte aplikaci buď ze sady Visual Studio, nebo z .NET Core CLI:

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio/)

Otevřete vygenerovaný soubor *. csproj* a z něj spusťte aplikaci jako normální.

Proces sestavení obnoví závislosti npm při prvním spuštění, což může trvat několik minut. Následná sestavení jsou mnohem rychlejší.

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli/)

Ujistěte se, že máte proměnnou prostředí `ASPNETCORE_Environment` s názvem s `Development`hodnotou. V systému Windows (v případě výzev mimo PowerShell) spusťte `SET ASPNETCORE_Environment=Development`příkaz. V systému Linux nebo macOS spusťte `export ASPNETCORE_Environment=Development`.

Spusťte [sestavení dotnet](/dotnet/core/tools/dotnet-build) a ověřte správné sestavení aplikace. Při prvním spuštění proces sestavení obnoví závislosti NPM, což může trvat několik minut. Následná sestavení jsou mnohem rychlejší.

Spuštěním příkazu [dotnet](/dotnet/core/tools/dotnet-run) spusťte aplikaci. Protokoluje se zpráva podobná následující:

```console
Now listening on: http://localhost:<port>
```

V prohlížeči přejděte na tuto adresu URL.

Aplikace spustí instanci úhlového CLI serveru na pozadí. Protokoluje se zpráva podobná následující: *Živý&lt;vývojový server NG naslouchá na localhost: otherport&gt;, otevřete svůj prohlížeč na http://localhost:&lt otherport&gt;/* . Tuto zprávu ignorujte. Tato zpráva&mdash; **není** adresou URL pro kombinované ASP.NET Core a úhlové aplikace CLI.

---

Šablona projektu vytvoří aplikaci ASP.NET Core a úhlovou aplikaci. Aplikace ASP.NET Core je určena k použití pro přístup k datům, autorizaci a další aspekty na straně serveru. Úhlová aplikace, která je umístěná v podadresáři *clientapp* , má být použita pro všechny aspekty uživatelského rozhraní.

## <a name="add-pages-images-styles-modules-etc"></a>Přidat stránky, obrázky, styly, moduly atd.

Adresář *clientapp* obsahuje standardní aplikaci s úhlovými CLI. Další informace najdete v oficiálních [úhlových dokumentaci](https://github.com/angular/angular-cli/wiki) .

Existují mírné rozdíly mezi úhlovými aplikacemi vytvořenými touto šablonou a jednou vytvořeným úhlovým rozhraním příkazového `ng new`řádku (Via). možnosti aplikace se ale nezměnily. Aplikace vytvořená šablonou obsahuje rozložení založené na [bootstrap](https://getbootstrap.com/)a základní příklad směrování.

## <a name="run-ng-commands"></a>Spustit příkazy NG

V příkazovém řádku přejděte do podadresáře *clientapp* :

```console
cd ClientApp
```

Pokud máte `ng` nástroj nainstalovaný globálně, můžete spustit libovolný z jeho příkazů. Můžete například spustit `ng lint`, `ng test`nebo kterýkoli z dalších [úhlů příkazů CLI](https://github.com/angular/angular-cli/wiki#additional-commands). Tuto situaci nemusíte spouštět `ng serve` , protože vaše aplikace ASP.NET Core se zabývá poskytováním součástí aplikace na straně serveru i na straně klienta. Interně se používá `ng serve` při vývoji.

Pokud `ng` nástroj nemáte nainstalovaný, spusťte `npm run ng` ho. Například můžete spustit `npm run ng lint` nebo `npm run ng test`.

## <a name="install-npm-packages"></a>Instalace balíčků npm

K instalaci balíčků npm třetích stran použijte příkazový řádek v podadresáři *clientapp* . Příklad:

```console
cd ClientApp
npm install --save <package_name>
```

## <a name="publish-and-deploy"></a>Publikování a nasazení

Ve vývoji se aplikace spouští v režimu optimalizovaném pro usnadnění vývoje. Například sady prostředků JavaScriptu obsahují zdrojové mapy (takže při ladění uvidíte původní kód TypeScriptu). Aplikace sleduje změny souborů TypeScriptu, HTML a CSS na disku a automaticky znovu zkompiluje a znovu načte, když se tyto soubory uvidí.

V produkčním prostředí poskytuje verzi vaší aplikace optimalizované pro výkon. To se nakonfiguruje tak, aby se automaticky stalo. Když publikujete, konfigurace sestavení generuje minifikovaného sestavení zkompilovaného kódu na straně klienta (AoT). Na rozdíl od sestavení pro vývoj není v produkčním sestavení nutné nainstalovat Node. js na server (Pokud jste nepovolili vykreslování na straně serveru (SSR)).

Můžete použít standardní [ASP.NET Core hostování a metody nasazení](xref:host-and-deploy/index).

## <a name="run-ng-serve-independently"></a>Spustit "NG obsluhu" nezávisle

Projekt je nakonfigurován tak, aby spouštěl svou vlastní instanci serveru úhlů CLI na pozadí při spuštění aplikace ASP.NET Core v režimu vývoje. To je vhodné, protože nemusíte spouštět samostatný server ručně.

Tato výchozí instalace je nevýhodná. Pokaždé, když upravíte C# kód a ASP.NET Core aplikace se musí restartovat, dojde k restartování serveru s úhlovým CLI. Pro spuštění zálohování je nutné asi 10 sekund. Pokud provádíte časté C# úpravy kódu a nechcete čekat na restartování úhlového CLI, spusťte server úhlového CLI externě, nezávisle na procesu ASP.NET Core. Postup:

1. V příkazovém řádku přejděte do podadresáře *clientapp* a spusťte vývojový server s úhlovými CLI:

    ```console
    cd ClientApp
    npm start
    ```

    > [!IMPORTANT]
    > Použijte `npm start` ke spuštění vývojového serveru úhlového CLI, `ng serve`ne, aby byla dodržena konfigurace v *Package. JSON* . K předání dalších parametrů serveru úhlů CLI je přidejte do příslušného `scripts` řádku v souboru *Package. JSON* .

2. Upravte aplikaci ASP.NET Core tak, aby používala vnější instanci úhlů CLI místo spuštění vlastní. Ve vaší *spouštěcí* třídě nahraďte `spa.UseAngularCliServer` vyvolání následujícím způsobem:

    ```csharp
    spa.UseProxyToSpaDevelopmentServer("http://localhost:4200");
    ```

Když spustíte aplikaci ASP.NET Core, nespustí se server úhlů CLI. Místo toho se použije instance, kterou jste spustili ručně. To umožňuje, aby se rychleji spouštěla a restartovala. Nečeká se na to, že rozhraní příkazového řádku pro každou chvíli znovu sestaví klientskou aplikaci.

### <a name="pass-data-from-net-code-into-typescript-code"></a>Předání dat z kódu .NET do kódu TypeScript

Během služby SSR budete možná chtít předat data z vaší aplikace ASP.NET Core do své aplikace v úhlovém požadavku. Můžete třeba předat informace cookie nebo něco přečíst z databáze. Provedete to tak, že upravíte třídu *Startup* . Ve zpětném volání `UseSpaPrerendering`pro nastavte `options.SupplyData` hodnotu jako následující:

```csharp
options.SupplyData = (context, data) =>
{
    // Creates a new value called isHttpsRequest that's passed to TypeScript code
    data["isHttpsRequest"] = context.Request.IsHttps;
};
```

`SupplyData` Zpětné volání umožňuje předat libovolná data serializovatelný v rámci požadavku JSON (například řetězce, logické hodnoty nebo čísla). Váš *hlavní kód. Server. TS* ho obdrží jako `params.data`. Například předchozí ukázka kódu předá logickou hodnotu `params.data.isHttpsRequest` `createServerRenderer` do zpětného volání. Tuto možnost můžete předat ostatním částem aplikace jakýmkoli způsobem podporovaným úhlovým. Například viz, jak *hlavní. Server. TS* předá `BASE_URL` hodnotu libovolné součásti, jejíž konstruktor je deklarován pro přijetí.

### <a name="drawbacks-of-ssr"></a>Nevýhody SSR

Ne všechny aplikace využívají SSR. Hlavní výhodou je vnímaný výkon. Návštěvníci, kteří dosáhnou vaší aplikace přes pomalé připojení k síti nebo pomalá mobilní zařízení, uvidí počáteční uživatelské rozhraní rychle, a to i v případě, že nějakou dobu trvá a načítají nebo analyzují sady JavaScript. Mnohé jednostránkové se ale používají hlavně v rychlé interní síti společnosti na rychlých počítačích, kde se aplikace zobrazuje skoro okamžitě.

Ve stejnou chvíli existují významné nevýhody povolování SSR. Přináší složitost vašemu procesu vývoje. Váš kód musí běžet ve dvou různých prostředích: na straně klienta a na straně serveru (v prostředí Node. js vyvolaném z ASP.NET Core). Tady je několik věcí, které je potřeba mít na paměti:

* SSR vyžaduje instalaci Node. js na produkčních serverech. Toto je automaticky pro některé scénáře nasazení, jako je například Azure App Services, ale ne pro jiné, jako je například Azure Service Fabric.
* Povolením příznaku buildudojdekpublikováníadresáře`BuildServerSideRenderer` node_modules. Tato složka obsahuje 20 000 souborů, což zvyšuje dobu nasazení.
* Chcete-li spustit kód v prostředí Node. js, nemůže spoléhat na existenci rozhraní JavaScript API specifických pro prohlížeč, například `window` nebo. `localStorage` Pokud váš kód (nebo některá z knihoven třetích stran, na které odkazujete) se pokusí použít tato rozhraní API, během průběhu SSR se zobrazí chyba. Nepoužívejte například jQuery, protože odkazuje na rozhraní API specifická pro prohlížeč na mnoha místech. Aby nedocházelo k chybám, musíte buď zabránit SSR nebo nejenom k rozhraní API nebo knihovnám specifickým pro prohlížeč. V rámci kontrol můžete zabalit jakákoli volání těchto rozhraní API, aby se zajistilo, že se nebudou vyvolávat během SSR. Použijte například následující příkaz v kódu JavaScript nebo TypeScript:

    ```javascript
    if (typeof window !== 'undefined') {
        // Call browser-specific APIs here
    }
    ```

## <a name="additional-resources"></a>Další zdroje

* <xref:security/authentication/identity/spa>
