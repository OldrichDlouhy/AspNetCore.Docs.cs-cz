---
title: Image Docker pro ASP.NET Core
author: rick-anderson
description: Naučte se používat publikované image Docker .NET Core z registru Docker. Vyžádání imagí a sestavení vlastních imagí.
ms.author: riande
ms.custom: mvc
ms.date: 06/18/2019
uid: host-and-deploy/docker/building-net-docker-images
ms.openlocfilehash: 64503ed55438b24f2d3d87092107408ddcb515d7
ms.sourcegitcommit: fcdf9aaa6c45c1a926bd870ed8f893bdb4935152
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/09/2019
ms.locfileid: "72165266"
---
# <a name="docker-images-for-aspnet-core"></a>Image Docker pro ASP.NET Core

V tomto kurzu se dozvíte, jak spustit aplikaci ASP.NET Core v kontejnerech Docker.

V tomto kurzu se naučíte:
> [!div class="checklist"]
> * Další informace o imagích Docker Microsoft .NET Core
> * Stažení ukázkové aplikace ASP.NET Core
> * Spustit ukázkovou aplikaci místně
> * Spuštění ukázkové aplikace v kontejnerech Linux
> * Spuštění ukázkové aplikace v kontejnerech Windows
> * Ruční sestavení a nasazení

## <a name="aspnet-core-docker-images"></a>Image ASP.NET Core Docker

Pro tento kurz si stáhnete ukázkovou aplikaci ASP.NET Core a spustíte ji v kontejnerech Docker. Ukázka funguje s kontejnery pro Linux i Windows.

Vzorový souboru Dockerfile využívá [funkci buildu pro více fází](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) pro sestavování a spouštění v různých kontejnerech. Kontejnery sestavení a spuštění jsou vytvořeny z imagí, které jsou k dispozici v Docker Hub od společnosti Microsoft:

* `dotnet/core/sdk`

  Ukázka používá tuto image k sestavení aplikace. Obrázek obsahuje .NET Core SDK, která obsahuje nástroje příkazového řádku (CLI). Obrázek je optimalizován pro místní vývoj, ladění a testování částí. Nástroje nainstalované pro vývoj a kompilaci vytvářejí poměrně velký obrázek. 

* `dotnet/core/aspnet`

   Ukázka používá tuto image ke spuštění aplikace. Image obsahuje modul runtime a knihovny ASP.NET Core a je optimalizovaný pro spuštěné aplikace v produkčním prostředí. Bitová kopie je navržena pro rychlost nasazení a spouštění aplikací, takže je optimalizován výkon sítě z registru Docker na hostitele Docker. Do kontejneru se zkopírují jenom binární soubory a obsah potřebný ke spuštění aplikace. Obsah je připravený ke spuštění, což umožňuje nejrychlejší čas od `Docker run` až po spuštění aplikace. Dynamická kompilace kódu není v modelu Docker nutná.

## <a name="prerequisites"></a>Požadavky
::: moniker range="< aspnetcore-3.0"

* [Sada .NET Core 2,2 SDK](https://www.microsoft.com/net/core)
::: moniker-end

::: moniker range=">= aspnetcore-3.0"

* [.NET Core SDK 3,0](https://dotnet.microsoft.com/download)

::: moniker-end

* Klient Docker 18,03 nebo novější

  * Distribuce systému Linux
    * [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
    * [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
    * [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
    * [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  * [macOS](https://docs.docker.com/docker-for-mac/install/)
  * [Windows](https://docs.docker.com/docker-for-windows/install/)

* [Git](https://git-scm.com/download)

## <a name="download-the-sample-app"></a>Stažení ukázkové aplikace

* Stáhněte si ukázku klonování [úložiště Docker .NET Core](https://github.com/dotnet/dotnet-docker): 

  ```console
  git clone https://github.com/dotnet/dotnet-docker
  ```

## <a name="run-the-app-locally"></a>Místní spuštění aplikace

* Přejděte do složky projektu v *dotnet-Docker/Samples/aspnetapp/aspnetapp*.

* Spuštěním následujícího příkazu Sestavte a spusťte aplikaci místně:

  ```dotnetcli
  dotnet run
  ```

* Pro otestování aplikace použijte v prohlížeči `http://localhost:5000`.

* Stisknutím kombinace kláves CTRL + C na příkazovém řádku zastavte aplikaci.

## <a name="run-in-a-linux-container"></a>Spuštění v kontejneru Linux

* V klientovi Docker přepněte na kontejnery Linux.

* Přejděte do složky souboru Dockerfile v *dotnet-Docker/Samples/aspnetapp*.

* Spuštěním následujících příkazů Sestavte a spusťte ukázku v Docker:

  ```console
  docker build -t aspnetapp .
  docker run -it --rm -p 5000:80 --name aspnetcore_sample aspnetapp
  ```

  Argumenty příkazu `build`:
  * Pojmenujte bitovou kopii aspnetapp.
  * Vyhledejte souboru Dockerfile v aktuální složce (tečka na konci).

  Argumenty příkazu Run:
  * Přidělte pseudo-TTY a nechte ho otevřený i v případě, že není připojený. (Stejný efekt jako `--interactive --tty`)
  * Kontejner se po ukončení automaticky odebere.
  * Namapujte port 5000 na místním počítači na port 80 v kontejneru.
  * Pojmenujte kontejner aspnetcore_sample.
  * Zadejte bitovou kopii aspnetapp.

* Pro otestování aplikace použijte v prohlížeči `http://localhost:5000`.

## <a name="run-in-a-windows-container"></a>Spuštění v kontejneru Windows

* V klientovi Docker přepněte do kontejnerů Windows.

Přejděte do složky Docker File na `dotnet-docker/samples/aspnetapp`.

* Spuštěním následujících příkazů Sestavte a spusťte ukázku v Docker:

  ```console
  docker build -t aspnetapp .
  docker run -it --rm --name aspnetcore_sample aspnetapp
  ```

* Pro kontejnery Windows budete potřebovat IP adresu kontejneru (Přejít na `http://localhost:5000` nebude fungovat):
  * Otevřete další příkazový řádek.
  * Spusťte `docker ps` pro zobrazení spuštěných kontejnerů. Ověřte, jestli je kontejner "aspnetcore_sample".
  * Spusťte `docker exec aspnetcore_sample ipconfig`, aby se zobrazila IP adresa kontejneru. Výstup příkazu vypadá jako v tomto příkladu:

    ```console
    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : contoso.com
       Link-local IPv6 Address . . . . . : fe80::1967:6598:124:cfa3%4
       IPv4 Address. . . . . . . . . . . : 172.29.245.43
       Subnet Mask . . . . . . . . . . . : 255.255.240.0
       Default Gateway . . . . . . . . . : 172.29.240.1
    ```

* Zkopírujte adresu IPv4 kontejneru (například 172.29.245.43) a vložte ji do adresního řádku prohlížeče, aby se aplikace otestovala.

## <a name="build-and-deploy-manually"></a>Ruční sestavení a nasazení

V některých scénářích můžete chtít nasadit aplikaci do kontejneru tak, že do ní nakopírujete soubory aplikace, které jsou potřeba v době běhu. V této části se dozvíte, jak ručně nasadit.

* Přejděte do složky projektu v *dotnet-Docker/Samples/aspnetapp/aspnetapp*.

* Spusťte příkaz [dotnet Publish](/dotnet/core/tools/dotnet-publish) :

  ```dotnetcli
  dotnet publish -c Release -o published
  ```

  Argumenty příkazu:
  * Sestavte aplikaci v režimu vydání (výchozí je režim ladění).
  * Vytvořte soubory v *publikované* složce.

* Spusťte aplikaci.

  * Windows:

    ```dotnetcli
    dotnet published\aspnetapp.dll
    ```

  * Linux:

    ```dotnetcli
    dotnet published/aspnetapp.dll
    ```

* Pokud chcete zobrazit domovskou stránku, přejděte na `http://localhost:5000`.

Chcete-li použít manuálně publikovanou aplikaci v kontejneru Docker, vytvořte novou souboru Dockerfile a pomocí příkazu `docker build .` Sestavte kontejner.

::: moniker range="< aspnetcore-3.0"

```console
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Souboru Dockerfile

Tady je *souboru Dockerfile* , který používá příkaz `docker build`, který jste spustili dříve.  Používá `dotnet publish` stejným způsobem jako v tomto oddílu k sestavování a nasazování.  

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out


FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

```console
FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Souboru Dockerfile

Tady je *souboru Dockerfile* , který používá příkaz `docker build`, který jste spustili dříve.  Používá `dotnet publish` stejným způsobem jako v tomto oddílu k sestavování a nasazování.  

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out


FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

::: moniker-end

```console
FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

## <a name="additional-resources"></a>Další zdroje

* [Docker – příkaz buildu](https://docs.docker.com/engine/reference/commandline/build)
* [Příkaz Spustit jako Docker](https://docs.docker.com/engine/reference/commandline/run)
* [Ukázka docker ASP.NET Core](https://github.com/dotnet/dotnet-docker) (ten, který jste použili v tomto kurzu.)
* [Konfigurace ASP.NET Core pro práci se servery proxy a nástroji pro vyrovnávání zatížení](/aspnet/core/host-and-deploy/proxy-load-balancer)
* [Práce s nástroji Docker sady Visual Studio](https://docs.microsoft.com/aspnet/core/publishing/visual-studio-tools-for-docker)
* [Ladění ve Visual Studiu Code](https://code.visualstudio.com/docs/nodejs/debugging-recipes#_debug-nodejs-in-docker-containers) 

## <a name="next-steps"></a>Další kroky

Úložiště Git, které obsahuje ukázkovou aplikaci, také obsahuje dokumentaci. Přehled prostředků, které jsou k dispozici v úložišti, najdete v [souboru Readme](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/README.md). Konkrétně se dozvíte, jak implementovat protokol HTTPS:

> [!div class="nextstepaction"]
> [Vývoj aplikací ASP.NET Core pomocí Docker přes HTTPS](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md)
