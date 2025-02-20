---
title: Konfigurace linkeru pro ASP.NET Core Blazor
author: guardrex
description: Naučte se řídit linker zprostředkujícího jazyka (IL) při sestavování aplikace Blazor.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 11/21/2019
no-loc:
- Blazor
uid: host-and-deploy/blazor/configure-linker
ms.openlocfilehash: 0bc987d72d2f684b1ecbd4a883e9a09fac7c801e
ms.sourcegitcommit: 3e503ef510008e77be6dd82ee79213c9f7b97607
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/22/2019
ms.locfileid: "74317292"
---
# <a name="configure-the-linker-for-aspnet-core-opno-locblazor"></a>Konfigurace linkeru pro ASP.NET Core Blazor

Od [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor provádí propojování v [prostředním jazyce (IL)](/dotnet/standard/managed-code#intermediate-language--execution) během sestavení pro odebrání zbytečného Il z výstupních sestavení aplikace.

Ovládací prvek propojuje sestavení pomocí některého z následujících přístupů:

* Zakáže propojení globálně s [vlastností MSBuild](#disable-linking-with-a-msbuild-property).
* Řízení propojení podle jednotlivých sestavení pomocí [konfiguračního souboru](#control-linking-with-a-configuration-file).

## <a name="disable-linking-with-a-msbuild-property"></a>Zakázat propojení s vlastností MSBuild

Propojení je ve výchozím nastavení povoleno, pokud je aplikace sestavena, včetně publikování. Chcete-li zakázat propojování pro všechna sestavení, nastavte vlastnost `BlazorLinkOnBuild` MSBuild na `false` v souboru projektu:

```xml
<PropertyGroup>
  <BlazorLinkOnBuild>false</BlazorLinkOnBuild>
</PropertyGroup>
```

## <a name="control-linking-with-a-configuration-file"></a>Řízení propojování pomocí konfiguračního souboru

Řízení propojení na základě sestavení zadáním konfiguračního souboru XML a zadáním souboru jako položky MSBuild v souboru projektu:

```xml
<ItemGroup>
  <BlazorLinkerDescriptor Include="Linker.xml" />
</ItemGroup>
```

*Linker. XML*:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
  This file specifies which parts of the BCL or Blazor packages must not be
  stripped by the IL Linker even if they aren't referenced by user code.
-->
<linker>
  <assembly fullname="mscorlib">
    <!--
      Preserve the methods in WasmRuntime because its methods are called by 
      JavaScript client-side code to implement timers.
      Fixes: https://github.com/aspnet/Blazor/issues/239
    -->
    <type fullname="System.Threading.WasmRuntime" />
  </assembly>
  <assembly fullname="System.Core">
    <!--
      System.Linq.Expressions* is required by Json.NET and any 
      expression.Compile caller. The assembly isn't stripped.
    -->
    <type fullname="System.Linq.Expressions*" />
  </assembly>
  <!--
    In this example, the app's entry point assembly is listed. The assembly
    isn't stripped by the IL Linker.
  -->
  <assembly fullname="MyCoolBlazorApp" />
</linker>
```

Další informace naleznete v tématu [linkeru Il: syntaxe popisovače XML](https://github.com/mono/linker/blob/master/src/linker/README.md#syntax-of-xml-descriptor).

### <a name="configure-the-linker-for-internationalization"></a>Konfigurace linkeru pro mezinárodní využití

Ve výchozím nastavení Blazorkonfigurace linkeru pro Blazor aplikace pro WebAssembly odříznout informace o mezinárodním prostředí s výjimkou výslovně požadovaných místních hodnot. Odebrání těchto sestavení minimalizuje velikost aplikace.

Chcete-li určit, která sestavení I18N jsou zachována, nastavte vlastnost `<MonoLinkerI18NAssemblies>` MSBuild v souboru projektu:

```xml
<PropertyGroup>
  <MonoLinkerI18NAssemblies>{all|none|REGION1,REGION2,...}</MonoLinkerI18NAssemblies>
</PropertyGroup>
```

| Hodnota oblasti     | Sestavení oblasti mono    |
| ---------------- | ----------------------- |
| `all`            | Všechna sestavení, která jsou součástí |
| `cjk`            | *I18N. CJK. dll*          |
| `mideast`        | *I18N. MidEast. dll*      |
| `none` (výchozí) | Žádné                    |
| `other`          | *I18N. Jiná knihovna. dll*        |
| `rare`           | *I18N. Vzácná knihovna DLL*         |
| `west`           | *I18N. Západ. dll*         |

Oddělte více hodnot pomocí čárky (například `mideast,west`).

Další informace najdete v tématu [i18n: Pnetlib internationaling Framework knihovnu (úložiště GitHub mono/mono)](https://github.com/mono/mono/tree/master/mcs/class/I18N).
