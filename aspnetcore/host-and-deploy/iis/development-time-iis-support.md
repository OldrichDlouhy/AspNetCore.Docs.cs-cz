---
title: Podpora služby IIS v době vývoje v aplikaci Visual Studio pro ASP.NET Core
author: guardrex
description: Pokud používáte službu IIS v systému Windows Server, můžete zjišťovat podporu pro ladění aplikací ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/26/2019
uid: host-and-deploy/iis/development-time-iis-support
ms.openlocfilehash: 2d2c8d8e4b39d18a3a727e2a2bdef3be566897c1
ms.sourcegitcommit: 16cf016035f0c9acf3ff0ad874c56f82e013d415
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/29/2019
ms.locfileid: "73034148"
---
# <a name="development-time-iis-support-in-visual-studio-for-aspnet-core"></a>Podpora služby IIS v době vývoje v aplikaci Visual Studio pro ASP.NET Core

Od [sourabh Shirhatti](https://twitter.com/sshirhatti) a [Luke Latham](https://github.com/guardrex)

Tento článek popisuje podporu sady [Visual Studio](https://visualstudio.microsoft.com) pro ladění ASP.NET Core aplikací běžících na Windows serveru se službou IIS. Toto téma vás provede povolením tohoto scénáře a nastavením projektu.

## <a name="prerequisites"></a>Požadavky

* [Visual Studio pro Windows](https://visualstudio.microsoft.com/downloads/)
* **ASP.NET a webové vývojové** úlohy
* **Vývojová úloha .NET Core pro různé platformy**
* Certifikát zabezpečení X. 509 (pro podporu protokolu HTTPS)

## <a name="enable-iis"></a>Povolení služby IIS

1. V systému Windows přejděte do části **Ovládací panely** > **programy** >  programy**a funkce** > **zapnout nebo vypnout funkce systému Windows** (levá strana obrazovky).
1. Zaškrtněte políčko **Internetová informační služba** . Vyberte **OK**.

Instalace služby IIS může vyžadovat restartování systému.

## <a name="configure-iis"></a>Konfigurace služby IIS

Služba IIS musí mít nakonfigurovaný web s následujícím:

* **Název hostitele** &ndash; obvykle se používá **výchozí web** s **názvem hostitele** `localhost`. Libovolný platný web služby IIS, který má jedinečný název hostitele, ale funguje.
* **Vazba webu**
  * Pro aplikace, které vyžadují protokol HTTPS, vytvořte vazbu na port 443 s certifikátem. Obvykle se používá **IIS Express certifikát pro vývoj** , ale jakýkoli platný certifikát funguje.
  * U aplikací, které používají protokol HTTP, potvrďte existenci vazby k odeslání 80 nebo vytvořte vazbu na port 80 pro novou lokalitu.
  * Použijte jednu vazbu pro HTTP nebo HTTPS. **Vazba na oba porty HTTP i HTTPS současně není podporována.**

## <a name="enable-development-time-iis-support-in-visual-studio"></a>Povolení podpory služby IIS v době vývoje v aplikaci Visual Studio

1. Spusťte instalační program sady Visual Studio.
1. Vyberte možnost **Upravit** pro instalaci sady Visual Studio, kterou chcete použít pro podporu při vývoji služby IIS.
1. Pro úlohu **vývoje ASP.NET a webu** vyhledejte a nainstalujte komponentu **podpory služby IIS pro vývoj** .

   Tato součást je uvedena v části **volitelný** oddíl v části **čas vývoje** , který je podporován na panelu **podrobností o instalaci** v napravo od úloh. Komponenta nainstaluje [modul ASP.NET Core](xref:host-and-deploy/aspnet-core-module), což je NATIVNÍ modul IIS vyžadovaný ke spouštění ASP.NET Core aplikací se službou IIS.

## <a name="configure-the-project"></a>Konfigurace projektu

### <a name="https-redirection"></a>Přesměrování HTTPS

Pro nový projekt, který vyžaduje protokol HTTPS, zaškrtněte políčko pro **konfiguraci HTTPS** v okně **vytvořit novou ASP.NET Core webovou aplikaci** . Zaškrtnutím políčka přidáte [Přesměrování protokolu HTTPS a HSTS middleware](xref:security/enforcing-ssl) do aplikace, když se vytvoří.

U existujícího projektu, který vyžaduje protokol HTTPS, použijte přesměrování HTTPS a HSTS middleware v `Startup.Configure`. Další informace najdete v tématu <xref:security/enforcing-ssl>.

Pro projekt, který používá protokol HTTP, [přesměrování https a middleware HSTS](xref:security/enforcing-ssl) nejsou přidány do aplikace. Není nutná žádná konfigurace aplikace.

### <a name="iis-launch-profile"></a>Profil spuštění služby IIS

Vytvoření nového spouštěcího profilu pro přidání podpory služby IIS v době vývoje:

::: moniker range=">= aspnetcore-3.0"

1. Klikněte pravým tlačítkem na projekt v **Průzkumník řešení**. Vyberte **vlastnosti**. Otevřete kartu **ladění** .
1. Pro možnost **profil**vyberte tlačítko **Nový** . Pojmenujte profil "IIS" v místním okně. Vyberte **OK** a vytvořte profil.
1. V nastavení **spuštění** vyberte ze seznamu **službu IIS** .
1. Zaškrtněte políčko pro **spuštění prohlížeče** a zadejte adresu URL koncového bodu.

   Když aplikace vyžaduje HTTPS, použijte koncový bod HTTPS (`https://`). V případě protokolu HTTP použijte koncový bod HTTP (`http://`).

   Zadejte stejný název hostitele a port, protože [Konfigurace služby IIS zadaná dříve používá](#configure-iis), obvykle `localhost`.

   Zadejte název aplikace na konci adresy URL.

   Například `https://localhost/WebApplication1` (HTTPS) nebo `http://localhost/WebApplication1` (HTTP) jsou platné adresy URL koncového bodu.
1. V části **proměnné prostředí** vyberte tlačítko **Přidat** . Zadejte proměnnou prostředí s **názvem** `ASPNETCORE_ENVIRONMENT` a **hodnotou** `Development`.
1. V oblasti **nastavení webového serveru** nastavte **adresu URL aplikace** na stejnou hodnotu, která se používá pro adresu URL koncového bodu **spouštěcího prohlížeče** .
1. Pro nastavení **modelu hostování** v aplikaci Visual Studio 2019 nebo novější vyberte možnost **výchozí** pro použití modelu hostování používaného projektem. Pokud projekt nastaví vlastnost `<AspNetCoreHostingModel>` v souboru projektu, je použita hodnota vlastnosti (`InProcess` nebo `OutOfProcess`). Pokud vlastnost není k dispozici, použije se výchozí hostující model aplikace, který je v procesu. Pokud aplikace vyžaduje explicitní nastavení modelu hostování, které se liší od normálního hostujícího modelu aplikace, nastavte **model hostování** buď na hodnotu `In Process` nebo `Out Of Process` podle potřeby.
1. Uložte profil.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

1. Klikněte pravým tlačítkem na projekt v **Průzkumník řešení**. Vyberte **vlastnosti**. Otevřete kartu **ladění** .
1. Pro možnost **profil**vyberte tlačítko **Nový** . Pojmenujte profil "IIS" v místním okně. Vyberte **OK** a vytvořte profil.
1. V nastavení **spuštění** vyberte ze seznamu **službu IIS** .
1. Zaškrtněte políčko pro **spuštění prohlížeče** a zadejte adresu URL koncového bodu.

   Když aplikace vyžaduje HTTPS, použijte koncový bod HTTPS (`https://`). V případě protokolu HTTP použijte koncový bod HTTP (`http://`).

   Zadejte stejný název hostitele a port, protože [Konfigurace služby IIS zadaná dříve používá](#configure-iis), obvykle `localhost`.

   Zadejte název aplikace na konci adresy URL.

   Například `https://localhost/WebApplication1` (HTTPS) nebo `http://localhost/WebApplication1` (HTTP) jsou platné adresy URL koncového bodu.
1. V části **proměnné prostředí** vyberte tlačítko **Přidat** . Zadejte proměnnou prostředí s **názvem** `ASPNETCORE_ENVIRONMENT` a **hodnotou** `Development`.
1. V oblasti **nastavení webového serveru** nastavte **adresu URL aplikace** na stejnou hodnotu, která se používá pro adresu URL koncového bodu **spouštěcího prohlížeče** .
1. Pro nastavení **modelu hostování** v aplikaci Visual Studio 2019 nebo novější vyberte možnost **výchozí** pro použití modelu hostování používaného projektem. Pokud projekt nastaví vlastnost `<AspNetCoreHostingModel>` v souboru projektu, je použita hodnota vlastnosti (`InProcess` nebo `OutOfProcess`). Pokud vlastnost není k dispozici, použije se výchozí hostující model aplikace, který je mimo proces. Pokud aplikace vyžaduje explicitní nastavení modelu hostování, které se liší od normálního hostujícího modelu aplikace, nastavte **model hostování** buď na hodnotu `In Process` nebo `Out Of Process` podle potřeby.
1. Uložte profil.

::: moniker-end

Pokud nepoužíváte aplikaci Visual Studio, ručně přidejte profil spuštění do souboru [launchSettings. JSON](https://json.schemastore.org/launchsettings) ve složce *Properties (vlastnosti* ). Následující příklad nakonfiguruje profil pro použití protokolu HTTPS:

```json
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iis": {
      "applicationUrl": "https://localhost/WebApplication1",
      "sslPort": 0
    }
  },
  "profiles": {
    "IIS": {
      "commandName": "IIS",
      "launchBrowser": true,
      "launchUrl": "https://localhost/WebApplication1",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

Potvrďte, že se koncové body `applicationUrl` a `launchUrl` shodují a používají stejný protokol jako konfigurace vazby služby IIS, buď HTTP, nebo HTTPS.

## <a name="run-the-project"></a>Spustit projekt

Spusťte Visual Studio jako správce:

* Potvrďte, že rozevírací seznam konfigurace sestavení je nastavený na **ladit**.
* Nastavte [tlačítko Spustit ladění](/visualstudio/debugger/debugger-feature-tour) na profil **služby IIS** a kliknutím na tlačítko aplikaci spusťte.

Visual Studio se může vyzvat k restartování, pokud není spuštěný jako správce. Pokud se zobrazí výzva, restartujte Visual Studio.

Pokud se používá nedůvěryhodný vývojový certifikát, prohlížeč může vyžadovat, abyste vytvořili výjimku pro nedůvěryhodný certifikát.

> [!NOTE]
> Ladění konfigurace sestavení pro vydání pomocí [pouze můj kód](/visualstudio/debugger/just-my-code) a optimalizace kompilátoru mají za následek zhoršené prostředí. Například body přerušení nejsou k dispozice.

## <a name="additional-resources"></a>Další zdroje

* [Začínáme ve službě IIS pomocí Správce služby IIS](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8)
* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>
* <xref:security/enforcing-ssl>
