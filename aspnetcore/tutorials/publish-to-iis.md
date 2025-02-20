---
title: Publikování aplikace ASP.NET Core ve službě IIS
author: guardrex
description: Naučte se hostovat aplikaci ASP.NET Core na serveru IIS.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/03/2019
uid: tutorials/publish-to-iis
ms.openlocfilehash: 820527cc15f883c906d2fdf1c073d443a5b3b40e
ms.sourcegitcommit: d8b12cc1716ee329d7bd2300e201b61e15d506ac
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/04/2019
ms.locfileid: "71942888"
---
# <a name="publish-an-aspnet-core-app-to-iis"></a>Publikování aplikace ASP.NET Core ve službě IIS

Podle [Luke Latham](https://github.com/guardrex)

V tomto kurzu se dozvíte, jak hostovat aplikaci ASP.NET Core na serveru IIS.

Tento kurz se zabývá následujícími tématy:

> [!div class="checklist"]
> * Nainstalujte hostující sadu .NET Core na Windows Server.
> * Vytvořte web IIS ve Správci služby IIS.
> * Nasaďte aplikaci ASP.NET Core.

## <a name="prerequisites"></a>Požadavky

* [.NET Core SDK](/dotnet/core/sdk) nainstalované na vývojovém počítači.
* Windows Server nakonfigurovaný pomocí role serveru **webový server (IIS)** . Pokud váš server není nakonfigurovaný na hostování webů se službou IIS, postupujte podle pokynů v části <xref:host-and-deploy/iis/index#iis-configuration> *Konfigurace služby IIS* v článku a pak se vraťte k tomuto kurzu.

> [!WARNING]
> **Konfigurace služby IIS a zabezpečení webu zahrnují koncepty, na které se tento kurz nevztahuje.** Další informace najdete v dokumentaci ke službě IIS v dokumentaci ke službě [Microsoft IIS](https://www.iis.net/) a v [ASP.NET Core článku o hostování se službou IIS](xref:host-and-deploy/iis/index) před hostováním produkčních aplikací ve službě IIS.
>
> Důležité scénáře pro hostování služby IIS, na které se nevztahuje tento kurz, zahrnují:
>
> * [Vytvoření podregistru pro ochranu ASP.NET Core dat](xref:host-and-deploy/iis/index#data-protection)
> * [Konfigurace seznamu Access Control fondu aplikací (ACL)](xref:host-and-deploy/iis/index#application-pool-identity)
> * V tomto kurzu nasadíte v rámci konceptů nasazení služby IIS aplikaci bez zabezpečení HTTPS nakonfigurovanou ve službě IIS. Další informace o hostování aplikace, která je povolená pro protokol HTTPS, najdete v tématech zabezpečení v části [Další zdroje](#additional-resources) tohoto článku. Další pokyny pro hostování ASP.NET Core aplikací jsou uvedené v <xref:host-and-deploy/iis/index> článku.

## <a name="install-the-net-core-hosting-bundle"></a>Instalace .NET Core, který je hostitelem svazku

Na server IIS nainstalujte *hostující sadu .NET Core* . Nainstaluje sady .NET Core Runtime, knihovny .NET Core a [modul ASP.NET Core](xref:host-and-deploy/aspnet-core-module). Povolí modul ASP.NET Core aplikací ke spuštění za služby IIS.

Stažení instalačního programu pomocí následujícího odkazu:

[Aktuální instalační program sady hostování rozhraní .NET Core (s přímým přístupem ke stažení)](https://www.microsoft.com/net/permalink/dotnetcore-current-windows-runtime-bundle-installer)

1. Spusťte instalační program na serveru IIS.

1. Restartujte server nebo spusťte příkaz **net stop** s příponou " **net start w3svc** " v příkazovém prostředí.

## <a name="create-the-iis-site"></a>Vytvořte web služby IIS

1. Na serveru služby IIS vytvořte složku, která bude obsahovat publikované složky a soubory aplikace. V následujícím kroku je jako fyzická cesta k aplikaci služba IIS poskytována jako cesta k této složce.

1. Ve Správci služby IIS otevřete uzel serveru na panelu **připojení** . Klikněte pravým tlačítkem myši **lokality** složky. Vyberte **přidat web** z kontextové nabídky.

1. Zadejte **název lokality** a nastavte **fyzickou cestu** ke složce pro nasazení aplikace, kterou jste vytvořili. Zadejte konfiguraci **vazby** a vytvořte web výběrem **OK**.

## <a name="create-an-aspnet-core-razor-pages-app"></a>Vytvoření aplikace ASP.NET Core Razor Pages

Pokud chcete vytvořit aplikaci Razor Pages, postupujte podle kurzu.<xref:getting-started>

## <a name="publish-and-deploy-the-app"></a>Publikování a nasazení aplikace

*Publikování aplikace* znamená, že se vytvoří kompilovaná aplikace, která může být hostována serverem. *Nasazení aplikace* znamená přesunutí publikované aplikace do hostitelského systému. Krok publikování je zpracováván [.NET Core SDK](/dotnet/core/sdk), zatímco krok nasazení může být zpracován řadou přístupů. Tento kurz přijme přístup k nasazení *složky* , kde:

* Aplikace se publikuje do složky.
* Obsah složky se přesune do složky webu IIS ( **fyzická cesta** k lokalitě ve Správci služby IIS).

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

1. V **Průzkumník řešení** klikněte pravým tlačítkem na projekt a vyberte **publikovat**.
1. V dialogovém okně **vybrat cíl publikování** vyberte možnost publikování **složky** .
1. Nastavte **složku nebo cestu sdílení souborů** .
   * Pokud jste vytvořili složku pro web IIS, která je k dispozici ve vývojovém počítači jako sdílená síťová složka, zadejte cestu ke sdílené složce. Aktuální uživatel musí mít oprávnění k zápisu pro publikování do sdílené složky.
   * Pokud se nemůžete přímo nasadit do složky webu IIS na serveru služby IIS, proveďte publikování do složky na neodstranitelné médium a fyzicky přesuňte publikovanou aplikaci do složky webu IIS na serveru, což je **fyzická cesta** k webu ve Správci služby IIS. Přesuňte obsah složky *bin/Release/{Target Framework}/Publish* do složky webu IIS na serveru, který je **fyzickou cestou** lokality ve Správci služby IIS.

# <a name="net-core-clitabnetcore-cli"></a>[Rozhraní příkazového řádku .NET Core](#tab/netcore-cli)

1. V příkazovém prostředí publikujte aplikaci v konfiguraci vydaných verzí pomocí příkazu [dotnet Publish](/dotnet/core/tools/dotnet-publish) :

   ```dotnetcli
   dotnet publish --configuration Release
   ```

1. Přesuňte obsah složky *bin/Release/{Target Framework}/Publish* do složky webu IIS na serveru, který je **fyzickou cestou** lokality ve Správci služby IIS.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

1. Klikněte pravým tlačítkem na projekt v **řešení** a vyberte **publikovat** > **publikovat do složky**.
1. Nastavte cestu ke **složce** .
   * Pokud jste vytvořili složku pro web IIS, která je k dispozici ve vývojovém počítači jako sdílená síťová složka, zadejte cestu ke sdílené složce. Aktuální uživatel musí mít oprávnění k zápisu pro publikování do sdílené složky.
   * Pokud se nemůžete přímo nasadit do složky webu IIS na serveru služby IIS, proveďte publikování do složky na neodstranitelné médium a fyzicky přesuňte publikovanou aplikaci do složky webu IIS na serveru, což je **fyzická cesta** k webu ve Správci služby IIS. Přesuňte obsah složky *bin/Release/{Target Framework}/Publish* do složky webu IIS na serveru, který je **fyzickou cestou** lokality ve Správci služby IIS.

---

## <a name="browse-the-website"></a>Přejděte na web

Aplikace je v prohlížeči přístupná, až obdrží první požadavek. Vyžádejte aplikaci na vazbu koncového bodu, kterou jste navázali ve Správci služby IIS pro danou lokalitu.

## <a name="next-steps"></a>Další kroky

V tomto kurzu jste se naučili:

> [!div class="checklist"]
> * Nainstalujte hostující sadu .NET Core na Windows Server.
> * Vytvořte web IIS ve Správci služby IIS.
> * Nasaďte aplikaci ASP.NET Core.

Další informace o hostování ASP.NET Corech aplikací ve službě IIS najdete v článku Přehled služby IIS:

> [!div class="nextstepaction"]
> <xref:host-and-deploy/iis/index>

## <a name="additional-resources"></a>Další zdroje

### <a name="articles-in-the-aspnet-core-documentation-set"></a>Články v sadě dokumentace k ASP.NET Core

* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:test/troubleshoot-azure-iis>
* <xref:security/enforcing-ssl>

### <a name="articles-pertaining-to-aspnet-core-app-deployment"></a>Články týkající se ASP.NET Coreho nasazení aplikace

* <xref:tutorials/publish-to-azure-webapp-using-vs>
* <xref:tutorials/publish-to-azure-webapp-using-vscode>
* <xref:host-and-deploy/visual-studio-publish-profiles>
* [Publikování webové aplikace do složky pomocí Visual Studio pro Mac](/visualstudio/mac/publish-folder)

### <a name="articles-on-iis-https-configuration"></a>Články týkající se konfigurace protokolu HTTPS služby IIS

* [Konfigurace SSL ve Správci služby IIS](/iis/manage/configuring-security/configuring-ssl-in-iis-manager)
* [Jak nastavit SSL v IIS](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis)

### <a name="articles-on-iis-and-windows-server"></a>Články na IIS a Windows serveru

* [Lokality oficiální Microsoft IIS](https://www.iis.net/)
* [Technická knihovna obsahu Windows serveru](/windows-server/windows-server)
