---
title: Ověřování přes Facebook, Google a externí poskytovatel bez ASP.NET Core identity
author: rick-anderson
description: Vysvětlení použití Facebooku, Google, Twitteru atd. ověřování uživatelů účtu bez ASP.NET Core identity.
ms.author: riande
ms.date: 11/19/2019
uid: security/authentication/social/social-without-identity
ms.openlocfilehash: 680ea091dcc5ed7f94879b5d277e8be7e5abeb7b
ms.sourcegitcommit: f40c9311058c9b1add4ec043ddc5629384af6c56
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/21/2019
ms.locfileid: "74289119"
---
# <a name="use-social-sign-in-provider-authentication-without-aspnet-core-identity"></a>Použití ověřování poskytovatele přihlašování přes sociální sítě bez ASP.NET Core identity

::: moniker range=">= aspnetcore-3.0"

<xref:security/authentication/social/index> popisuje, jak povolit uživatelům přihlášení pomocí OAuth 2,0 s přihlašovacími údaji od externích zprostředkovatelů ověřování. Přístup popsaný v tomto tématu zahrnuje ASP.NET Core identity jako poskytovatele ověřování.

Tato ukázka předvádí, jak použít externího zprostředkovatele ověřování **bez** ASP.NET Core identity. To je užitečné pro aplikace, které nevyžadují všechny funkce ASP.NET Core identity, ale stále vyžadují integraci s důvěryhodným externím poskytovatelem ověřování.

Tato ukázka používá [ověřování Google](xref:security/authentication/google-logins) pro ověřování uživatelů. Pomocí ověřování Google se posunou mnohé ze složitých procesů při správě procesu přihlašování do Google. Informace o integraci s jiným externím poskytovatelem ověřování naleznete v následujících tématech:

* [Ověřování Facebooku](xref:security/authentication/facebook-logins)
* [Ověřování Microsoftu](xref:security/authentication/microsoft-logins)
* [Ověřování Twitteru](xref:security/authentication/twitter-logins)
* [Další zprostředkovatelé](xref:security/authentication/otherlogins)

## <a name="configuration"></a>Konfiguraci

V metodě `ConfigureServices` nakonfigurujte ověřovací schémata aplikace pomocí metod <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*>, <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>a <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*>:

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Startup.cs?name=snippet1)]

Volání <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> nastaví <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme>aplikace. `DefaultScheme` je výchozí schéma používané následujícími metodami rozšíření ověřování `HttpContext`:

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

Nastavení `DefaultScheme` aplikace na [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) ("cookies") nakonfiguruje aplikaci tak, aby jako výchozí schéma pro tyto metody rozšíření používala soubory cookie. Nastavení <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> aplikace na [GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) ("Google") nakonfiguruje aplikaci tak, aby používala Google jako výchozí schéma pro volání `ChallengeAsync`. `DefaultChallengeScheme` Přepisuje `DefaultScheme`. Další vlastnosti, které přepisují `DefaultScheme` při nastavení, najdete v tématu <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions>.

V `Startup.Configure`volejte `UseAuthentication` a `UseAuthorization` mezi voláním `UseRouting` a `UseEndpoints`. Tím se nastaví vlastnost `HttpContext.User` a spustí middleware autorizace pro požadavky:

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Startup.cs?name=snippet2&highlight=3-4)]

Další informace o schématech ověřování a ověřování souborů cookie najdete v tématu <xref:security/authentication/cookie>.

## <a name="apply-authorization"></a>Použít autorizaci

Otestujte konfiguraci ověřování aplikace použitím atributu `AuthorizeAttribute` na kontroler, akci nebo stránku. Následující kód omezuje přístup na stránku *ochrany osobních údajů* na uživatele, kteří byli ověřeni:

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>Odhlásit se

Chcete-li odhlásit aktuálního uživatele a odstranit svůj soubor cookie, zavolejte [SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*). Následující kód přidá obslužnou rutinu stránky `Logout` do stránky *indexu* :

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Pages/Index.cshtml.cs?name=snippet&highlight=3-7)]

Všimněte si, že volání `SignOutAsync` neurčuje schéma ověřování. `DefaultScheme` aplikace `CookieAuthenticationDefaults.AuthenticationScheme` se používá jako vrácení.

## <a name="additional-resources"></a>Další zdroje informací:

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
::: moniker range="< aspnetcore-3.0"

<xref:security/authentication/social/index> popisuje, jak povolit uživatelům přihlášení pomocí OAuth 2,0 s přihlašovacími údaji od externích zprostředkovatelů ověřování. Přístup popsaný v tomto tématu zahrnuje ASP.NET Core identity jako poskytovatele ověřování.

Tato ukázka předvádí, jak použít externího zprostředkovatele ověřování **bez** ASP.NET Core identity. To je užitečné pro aplikace, které nevyžadují všechny funkce ASP.NET Core identity, ale stále vyžadují integraci s důvěryhodným externím poskytovatelem ověřování.

Tato ukázka používá [ověřování Google](xref:security/authentication/google-logins) pro ověřování uživatelů. Pomocí ověřování Google se posunou mnohé ze složitých procesů při správě procesu přihlašování do Google. Informace o integraci s jiným externím poskytovatelem ověřování naleznete v následujících tématech:

* [Ověřování Facebooku](xref:security/authentication/facebook-logins)
* [Ověřování Microsoftu](xref:security/authentication/microsoft-logins)
* [Ověřování Twitteru](xref:security/authentication/twitter-logins)
* [Další zprostředkovatelé](xref:security/authentication/otherlogins)

## <a name="configuration"></a>Konfiguraci

V metodě `ConfigureServices` nakonfigurujte ověřovací schémata aplikace pomocí metod `AddAuthentication`, `AddCookie`a `AddGoogle`:

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Startup.cs?name=snippet1)]

Volání [AddAuthentication](/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication#Microsoft_Extensions_DependencyInjection_AuthenticationServiceCollectionExtensions_AddAuthentication_Microsoft_Extensions_DependencyInjection_IServiceCollection_System_Action_Microsoft_AspNetCore_Authentication_AuthenticationOptions__) nastaví [DefaultScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme)aplikace. `DefaultScheme` je výchozí schéma používané následujícími metodami rozšíření ověřování `HttpContext`:

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

Nastavení `DefaultScheme` aplikace na [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) ("cookies") nakonfiguruje aplikaci tak, aby jako výchozí schéma pro tyto metody rozšíření používala soubory cookie. Nastavení <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> aplikace na [GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) ("Google") nakonfiguruje aplikaci tak, aby používala Google jako výchozí schéma pro volání `ChallengeAsync`. `DefaultChallengeScheme` Přepisuje `DefaultScheme`. Další vlastnosti, které přepisují `DefaultScheme` při nastavení, najdete v tématu <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions>.

V metodě `Configure` voláním metody `UseAuthentication` volejte middleware ověřování, který nastaví vlastnost `HttpContext.User`. Před voláním `UseMvcWithDefaultRoute` nebo `UseMvc`volejte metodu `UseAuthentication`:

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Startup.cs?name=snippet2)]

Další informace o schématech ověřování a ověřování souborů cookie najdete v tématu <xref:security/authentication/cookie>.

## <a name="apply-authorization"></a>Použít autorizaci

Otestujte konfiguraci ověřování aplikace použitím atributu `AuthorizeAttribute` na kontroler, akci nebo stránku. Následující kód omezuje přístup na stránku *ochrany osobních údajů* na uživatele, kteří byli ověřeni:

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>Odhlásit se

Chcete-li odhlásit aktuálního uživatele a odstranit svůj soubor cookie, zavolejte [SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*). Následující kód přidá obslužnou rutinu stránky `Logout` do stránky *indexu* :

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Pages/Index.cshtml.cs?name=snippet&highlight=3-7)]

Všimněte si, že volání `SignOutAsync` neurčuje schéma ověřování. `DefaultScheme` aplikace `CookieAuthenticationDefaults.AuthenticationScheme` se používá jako vrácení.

## <a name="additional-resources"></a>Další zdroje informací:

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
