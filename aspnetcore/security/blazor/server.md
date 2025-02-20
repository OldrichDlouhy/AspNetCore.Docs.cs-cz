---
title: Zabezpečené ASP.NET Core Blazor serverových aplikací
author: guardrex
description: Naučte se zmírnit bezpečnostní hrozby pro Blazor serverových aplikací.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 11/12/2019
no-loc:
- Blazor
- SignalR
uid: security/blazor/server
ms.openlocfilehash: 5cf83a4dd255959e8840fca3a8194b5b4e2ad0a8
ms.sourcegitcommit: 3fc3020961e1289ee5bf5f3c365ce8304d8ebf19
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/12/2019
ms.locfileid: "73963882"
---
# <a name="secure-aspnet-core-opno-locblazor-server-apps"></a>Zabezpečené ASP.NET Core Blazor serverových aplikací

[Javier Calvarro Nelson](https://github.com/javiercn)

aplikace Blazor serveru přijímají *stavový* model zpracování dat, ve kterém server a klient udržují dlouhotrvající relaci. Trvalý stav je udržován [okruhem](xref:blazor/state-management), který může zahrnovat připojení, která jsou také potenciálně dlouhodobá.

Když uživatel navštíví lokalitu serveru Blazor, Server vytvoří okruh v paměti serveru. Okruh indikuje prohlížeči, který obsah se má vykreslit a reagovat na události, například když uživatel vybere tlačítko v uživatelském rozhraní. Chcete-li provést tyto akce, okruh vyvolá funkce JavaScriptu v prohlížeči uživatele a metodách .NET na serveru. Tato obousměrná interakce založená na jazyce JavaScript je označována jako zprostředkovatel [komunikace JavaScript (zprostředkovatel komunikace js)](xref:blazor/javascript-interop).

Vzhledem k tomu, že interoperabilita JS probíhá přes Internet a klient používá vzdálený prohlížeč, Blazor serverových aplikací sdílí většinu otázek zabezpečení webových aplikací. Toto téma popisuje běžné hrozby pro Blazor serverových aplikací a poskytuje pokyny pro zmírnění hrozeb zaměřené na internetové aplikace.

V omezených prostředích, jako jsou v podnikových sítích nebo intranetech, se jedná o některé pokyny k omezení rizik:

* Neplatí v omezeném prostředí.
* Nestojí za implementaci, protože bezpečnostní riziko je nízké v omezeném prostředí.

## <a name="resource-exhaustion"></a>Vyčerpání prostředků

K vyčerpání prostředků může dojít, když klient komunikuje se serverem a způsobí, že Server spotřebovává nadměrné prostředky. Nadměrné využití prostředků primárně ovlivňuje:

* [VČETNĚ](#cpu)
* [Rezident](#memory)
* [Připojení klientů](#client-connections)

Útoky DoS (Denial of Service) obvykle hledají vyčerpání prostředků aplikace nebo serveru. Vyčerpání prostředků ale nemusí být nutně výsledkem útoku na systém. Například omezené prostředky je možné vyčerpat z důvodu vysoké poptávky uživatelů. V části věnované [útokům DOS (Denial of Service)](#denial-of-service-dos-attacks) se systém DOS podrobněji zabývá.

Prostředky, které jsou externí pro Blazor Framework, jako jsou databáze a obslužné rutiny souborů (slouží ke čtení a zápisu souborů), můžou taky vyčerpat prostředky. Další informace najdete v tématu <xref:performance/performance-best-practices>.

### <a name="cpu"></a>Procesor

K vyčerpání procesoru může dojít v případě, že jeden nebo více klientů vynutí, aby server prováděl náročné fungování procesoru.

Představte si třeba Blazor serverovou aplikaci, která vypočítá *Fibonnacci číslo*. Fibonnacci číslo je vytvořeno z Fibonnacci sekvence, kde každé číslo v sekvenci je součet dvou předcházejících čísel. Množství práce potřebné k dosažení odpovědi závisí na délce sekvence a na velikosti počáteční hodnoty. Pokud aplikace neumístí omezení na požadavky klienta, mohou výpočty náročné na procesor poznamenat čas procesoru a snížit výkon dalších úloh. Nadměrné využití prostředků je zabezpečení, které má vliv na dostupnost.

Vyčerpání výkonu procesoru je obavou pro všechny veřejné aplikace. V běžných webových aplikacích jsou požadavky a připojení vyprší jako ochrana, ale aplikace Blazor Server neposkytují stejné záruky. aplikace Blazor serveru musí před provedením práce náročné na procesor zahrnovat vhodné kontroly a omezení.

### <a name="memory"></a>Rezident

K vyčerpání paměti může dojít v případě, že jeden nebo více klientů vynutí Server, aby využíval velké množství paměti.

Zvažte například Blazoraplikace na straně serveru s komponentou, která přijímá a zobrazuje seznam položek. Pokud Blazor aplikace neumístí omezení počtu povolených položek nebo počet položek, které se vrátí klientovi, může zpracování a vykreslování náročné na paměť podmístit paměť serveru do bodu, ve kterém výkon serveru utrpí. Server může způsobit selhání nebo zpomalit bod, na kterém se zdá, že došlo k chybě.

Při údržbě a zobrazování seznamu položek, které se vztahují k potenciálnímu scénáři vyčerpání paměti na serveru, vezměte v úvahu následující scénář:

* Položky v `List<MyItem>` vlastnost nebo pole používají paměť serveru. Pokud aplikace umožňuje dosáhnout neohraničeného seznamu položek, dojde k riziku serveru, který nemá dostatek paměti. Nedostatek paměti způsobí, že aktuální relace skončí (zhroucení) a všechny souběžné relace v této instanci serveru obdrží výjimku z důvodu nedostatku paměti. Aby nedocházelo k tomuto scénáři, musí aplikace používat datovou strukturu, která ukládá omezení počtu položek na souběžných uživatelích.
* Pokud se schéma stránkování nepoužívá pro vykreslování, server používá další paměť pro objekty, které nejsou viditelné v uživatelském rozhraní. Bez omezení počtu položek mohou nároky na paměť vyčerpat dostupnou paměť serveru. Chcete-li zabránit tomuto scénáři, použijte jeden z následujících přístupů:
  * Při vykreslování použít stránkované seznamy.
  * Zobrazí se pouze prvních 100 až 1 000 položek a vyžaduje, aby uživatel zadal kritéria hledání, aby vyhledal položky nad zobrazenými položkami.
  * Pro pokročilejší scénář vykreslování implementujte seznamy nebo mřížky podporující *virtualizaci*. Pomocí virtualizace vykreslí pouze podmnožinu položek, které jsou aktuálně viditelné uživateli. Když uživatel pracuje s posuvníkem v uživatelském rozhraní, komponenta vykreslí pouze ty položky, které jsou nutné k zobrazení. Položky, které nejsou aktuálně požadovány k zobrazení, lze uchovávat v sekundárním úložišti, což je ideální přístup. Nezobrazené položky je také možné uchovávat v paměti, což je méně ideální.

aplikace Blazor serveru nabízejí podobný programovací model pro jiné architektury uživatelského rozhraní pro stavové aplikace, jako je WPF, model Windows Forms nebo Blazor WebAssembly. Hlavním rozdílem je to, že v několika rozhraních uživatelského rozhraní paměť spotřebovaná aplikací patří klientovi a ovlivňuje pouze jednotlivé klienty. Například aplikace Blazor WebAssembly běží zcela na klientovi a používá pouze prostředky paměti klienta. Ve scénáři Blazor serveru aplikace patří do serveru a sdílí se mezi klienty v instanci serveru.

Požadavky na paměť na straně serveru jsou zvážené pro všechny Blazor serverové aplikace. Většina webových aplikací je ale Bezstavová a při vrácení odpovědi se uvolní paměť, která se používá při zpracování žádosti. Obecně doporučujeme, abyste klientům nepovolili přidělit nevázanou velikost paměti jako v jakékoli jiné aplikaci na straně serveru, která se zachovává připojení klientů. Paměť spotřebovaná aplikací Blazor serveru trvá déle než jedna žádost.

> [!NOTE]
> Během vývoje lze použít Profiler nebo zaznamenané trasování pro vyhodnocení požadavků na paměť klientů. Profiler nebo trasování nezachycuje paměť přidělenou konkrétnímu klientovi. Pokud chcete zachytit využití paměti konkrétního klienta během vývoje, Zachyťte výpis paměti a prověřte požadavky na paměť všech objektů, které jsou v okruhu uživatele rootem.

### <a name="client-connections"></a>Připojení klientů

K vyčerpání spojení může dojít, když jeden nebo více klientů otevře příliš mnoho souběžných připojení k serveru, což brání ostatním klientům v navázání nových připojení.

klienti Blazor navážou jedno připojení na relaci a udržují připojení otevřené po dobu, po kterém je okno prohlížeče otevřené. Požadavky na server správy všech připojení nejsou specifické pro Blazor aplikace. Vzhledem k trvalé povaze připojení a stavové povaze aplikací Blazor serveru je vyčerpání spojení větším rizikem k dostupnosti aplikace.

Ve výchozím nastavení neexistuje žádné omezení počtu připojení na uživatele pro aplikaci Blazor Server. Pokud aplikace vyžaduje limit připojení, proveďte jednu nebo více následujících přístupů:

* Vyžaduje ověření, které přirozeně omezuje možnosti neautorizovaných uživatelů připojit se k aplikaci. Aby byl tento scénář účinný, je nutné uživatelům zabránit v zřizování nových uživatelů.
* Omezte počet připojení na uživatele. Omezení připojení je možné dosáhnout pomocí následujících přístupů. Dbejte na to, abyste uživatelům povolili přístup k aplikaci (například když je na základě IP adresy klienta vytvořen limit připojení).
  * Na úrovni aplikace:
    * Rozšiřitelnost směrování koncových bodů.
    * Vyžadovat ověřování pro připojení k aplikaci a sledování aktivních relací na uživatele.
    * Po dosažení limitu zamítnout nové relace.
    * Proxy připojení WebSocket k aplikaci prostřednictvím proxy serveru, jako je například [Služba Azure SignalR](/azure/azure-signalr/signalr-overview) , která multiplexuje připojení z klientů do aplikace. Tato funkce poskytuje aplikaci s větší kapacitou připojení, než může jeden klient navázat a zabránit klientovi v vyčerpání připojení k serveru.
  * Na úrovni serveru: před aplikací použijte proxy server nebo bránu. Například [přední dveře Azure](/azure/frontdoor/front-door-overview) umožňují definovat, spravovat a monitorovat globální směrování webového provozu do aplikace.

## <a name="denial-of-service-dos-attacks"></a>Útoky DoS (Denial of Service)

Útoky DoS (Denial of Service) zahrnují klienta, který způsobuje, že server vyčerpá jeden nebo víc svých prostředků, takže aplikace nebude k dispozici. Blazor serverové aplikace zahrnují některá výchozí omezení a spoléhají na další ASP.NET Core a SignalR omezení pro ochranu proti útokům DoS:

| omezení aplikace Blazor serveru                            | Popis | Výchozí |
| ------------------------------------------------------- | ----------- | ------- |
| `CircuitOptions.DisconnectedCircuitMaxRetained`         | Maximální počet odpojených okruhů, které daný server uchovává v paměti. | 100 |
| `CircuitOptions.DisconnectedCircuitRetentionPeriod`     | Maximální doba, po kterou je odpojený okruh uložený v paměti předtím, než se rozpustí. | 3 minuty |
| `CircuitOptions.JSInteropDefaultCallTimeout`            | Maximální doba, po kterou Server počká, než vyprší časový limit asynchronního vyvolání funkce JavaScriptu. | 1 minuta |
| `CircuitOptions.MaxBufferedUnacknowledgedRenderBatches` | Maximální počet nepotvrzených vykreslících vykreslování: Server udržuje paměť na okruh v daném čase pro zajištění podpory robustního opětovného připojení. Po dosažení limitu Server přestane vytvářet nové dávky vykreslování, dokud klient nepotvrdí jednu nebo více dávek. | 10pruhový |


| omezení SignalR a ASP.NET Core             | Popis | Výchozí |
| ------------------------------------------ | ----------- | ------- |
| `CircuitOptions.MaximumReceiveMessageSize` | Velikost zprávy pro jednotlivou zprávu | 32 KB |

## <a name="interactions-with-the-browser-client"></a>Interakce s prohlížečem (klient)

Klient komunikuje s odesláním a dokončením vykreslování událostí serveru prostřednictvím aplikace JS. Komunikace interoperability JS přechází mezi oběma způsoby mezi jazykem JavaScript a .NET:

* Události prohlížeče jsou odesílány z klienta na server asynchronním způsobem.
* Server odpoví asynchronním vykreslením uživatelského rozhraní podle potřeby.

### <a name="javascript-functions-invoked-from-net"></a>Funkce JavaScriptu vyvolané z .NET

Pro volání z metod .NET do JavaScriptu:

* Všechna volání mají konfigurovatelný časový limit, po jehož uplynutí selžou, vrácení <xref:System.OperationCanceledException> volajícímu.
  * Výchozí časový limit pro volání (`CircuitOptions.JSInteropDefaultCallTimeout`) po jednu minutu. Pokud chcete tento limit nakonfigurovat, přečtěte si téma <xref:blazor/javascript-interop#harden-js-interop-calls>.
  * Je možné zadat token zrušení pro řízení zrušení u jednotlivých volání. Spoléhá se na výchozí časový limit volání, kde je to možné, a s časovým limitem pro jakékoli volání klienta, pokud je k dispozici token zrušení.
* Výsledek volání JavaScriptu nemůže být důvěryhodný. Klient aplikace Blazor spuštěný v prohlížeči vyhledá funkci JavaScriptu, která se má vyvolat. Funkce je vyvolána a výsledek nebo chyba je vytvořena. Zlomyslný klient se může pokusit:
  * Způsobí problém v aplikaci vrácením chyby z funkce JavaScriptu.
  * Vyvolávat nezamýšlené chování na serveru vrácením neočekávaného výsledku z funkce JavaScriptu.

Pro ochranu před předchozími scénáři proveďte následující opatření:

* Zabalte volání interoperability JS v rámci příkazů [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) k účtu k chybám, které mohou nastat během vyvolání. Další informace najdete v tématu <xref:blazor/handle-errors#javascript-interop>.
* Před provedením jakékoli akce Ověřte data vrácená z volání Interop v JS, včetně chybových zpráv.

### <a name="net-methods-invoked-from-the-browser"></a>Metody .NET vyvolané z prohlížeče

Nemusíte důvěřovat volání metod z JavaScriptu do .NET. Pokud je k JavaScriptu k dispozici metoda .NET, zvažte, jak je vyvolána metoda .NET:

* Považovat jakoukoli metodu .NET zveřejněnou pro JavaScript stejně jako veřejný koncový bod pro aplikaci.
  * Ověřte vstup.
    * Ujistěte se, že hodnoty jsou v rámci očekávaných rozsahů.
    * Zajistěte, aby měl uživatel oprávnění k provedení požadované akce.
  * Nepřiřazujte nadměrné množství prostředků jako součást volání metod .NET. Například proveďte kontrolu a umístěte omezení využití procesoru a paměti.
  * Vezměte v úvahu, že statické a instanční metody můžou být vystavené klientům JavaScriptu. Vyhněte se sdílení stavu napříč relacemi, pokud se návrh nevolá pro stav sdílení s příslušnými omezeními.
    * Pro metody, které jsou vystaveny prostřednictvím `DotNetReference` objektů, které byly vytvořeny prostřednictvím injektáže Dependency vstřik (DI), by měly být objekty registrovány jako objekty s oborem. To platí pro libovolnou službu DI, kterou používá aplikace Blazor Server.
    * U statických metod Vyhněte se vytváření stavu, který nelze nastavit na klienta, pokud aplikace explicitně nesdílí stav pro všechny uživatele v instanci serveru.
  * Vyhněte se předávání dat poskytovaných uživatelem v parametrech volání JavaScript. Pokud je předávání dat v parametrech naprosto povinné, ujistěte se, že kód jazyka JavaScript zpracovává předávání dat bez nutnosti chyby zabezpečení [SKRIPTOVÁNÍ XSS (více lokalit)](#cross-site-scripting-xss) . Nepište například data dodaná uživatelem do model DOM (Document Object Model) (DOM) nastavením vlastnosti `innerHTML` elementu. Zvažte použití [zásad zabezpečení obsahu (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) k zakázání `eval` a dalších nebezpečných primitivních primitiv jazyka JavaScript.
* Neimplementujte implementaci vlastního odesílání volání .NET nad rámec implementace odesílajícího rozhraní. Vystavení metod .NET v prohlížeči je pokročilý scénář, který se nedoporučuje pro obecné Blazor vývoj.

### <a name="events"></a>Události

Události poskytují vstupní bod pro aplikaci Blazor Server. Stejná pravidla pro ochranu koncových bodů ve webových aplikacích se vztahují na zpracování událostí v aplikacích Blazor Server. Škodlivý klient může odesílat všechna data, která chce odeslat jako datovou část pro událost.

Příklad:

* Událost změny pro `<select>` by mohla odeslat hodnotu, která není v rámci možností, které aplikace prezentuje klientovi.
* `<input>` mohl odeslat do serveru jakákoli textová data a obejít tak ověřování na straně klienta.

Aplikace musí ověřit data pro všechny události, které aplikace zpracovává. [Komponenty formulářů](xref:blazor/forms-validation) rozhraní Blazor Framework provádějí základní ověření. Pokud aplikace používá vlastní součásti formulářů, je nutné napsat vlastní kód pro ověření dat události podle potřeby.

události Blazor serveru jsou asynchronní, takže je možné odeslat do serveru více událostí, než aplikace začne reagovat tím, že vygeneruje nové vykreslování. To má vliv na některé zabezpečení, které je potřeba vzít v úvahu. Omezení akcí klienta v aplikaci musí být provedeno uvnitř obslužných rutin událostí a nemusí být závislé na aktuálním stavu zobrazení.

Vezměte v úvahu komponentu čítače, která by uživateli umožnila zvýšit hodnotu čítače maximálně třikrát. Tlačítko pro zvýšení čítače je podmíněně na základě hodnoty `count`:

```cshtml
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        count++;
    }
}
```

Klient může odeslat jednu nebo více událostí přírůstku předtím, než architektura vytvoří nové vykreslování této součásti. Výsledkem je, že uživatel může `count` zvýšit *za trojnásobnou dobu* , protože toto tlačítko není v uživatelském rozhraní k dispozici dostatečně rychle. Správný způsob, jak dosáhnout limitu tří `count` přírůstcích, je znázorněno v následujícím příkladu:

```cshtml
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        if (count < 3)
        {
            count++;
        }
    }
}
```

Přidáním `if (count < 3) { ... }` kontroly v rámci obslužné rutiny se rozhodnutí o zvýšení `count` vychází z aktuálního stavu aplikace. Rozhodnutí není založené na stavu uživatelského rozhraní jako v předchozím příkladu, což může být dočasně zastaralé.

### <a name="guard-against-multiple-dispatches"></a>Ochrana proti více odesláním

Pokud zpětné volání události vyvolá dlouhou běžící operaci, například načtení dat z externí služby nebo databáze, zvažte použití ochrany. Ochrana může uživatelům zabránit ve zařazení více operací do fronty, zatímco operace probíhá pomocí vizuální zpětné vazby. Následující kód komponenty nastaví `isLoading`, aby `true`, zatímco `GetForecastAsync` získá data ze serveru. Když je `isLoading` `true`, je tlačítko v uživatelském rozhraní zakázané:

```cshtml
@page "/fetchdata"
@using BlazorServerSample.Data
@inject WeatherForecastService ForecastService

<button disabled="@isLoading" @onclick="UpdateForecasts">Update</button>

@code {
    private bool isLoading;
    private WeatherForecast[] forecasts;

    private async Task UpdateForecasts()
    {
        if (!isLoading)
        {
            isLoading = true;
            forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
            isLoading = false;
        }
    }
}
```

### <a name="cancel-early-and-avoid-use-after-dispose"></a>Zrušit počáteční a vyhnout se použití po vyřazení

Kromě používání ochrany, jak je popsáno v části [Ochrana proti více odesláních](#guard-against-multiple-dispatches) , zvažte použití <xref:System.Threading.CancellationToken> pro zrušení dlouhotrvajících operací při uvolnění součásti. Tento přístup má výhodu při zamezení používání funkcí *po Dispose* v součástech:

```cshtml
@implements IDisposable

...

@code {
    private readonly CancellationTokenSource TokenSource = 
        new CancellationTokenSource();

    private async Task UpdateForecasts()
    {
        ...

        forecasts = await ForecastService.GetForecastAsync(DateTime.Now, 
            TokenSource.Token);

        if (TokenSource.Token.IsCancellationRequested)
        {
           return;
        }

        ...
    }

    public void Dispose()
    {
        CancellationTokenSource.Cancel();
    }
}
```

### <a name="avoid-events-that-produce-large-amounts-of-data"></a>Vyhněte se událostem, které vytváří velké objemy dat

Některé události modelu DOM, například `oninput` nebo `onscroll`, mohou vytvořit velké množství dat. Nepoužívejte tyto události v aplikacích Blazor Server.

## <a name="additional-security-guidance"></a>Další pokyny k zabezpečení

Pokyny pro zabezpečení ASP.NET Corech aplikací se vztahují na Blazor serverových aplikací a jsou uvedené v následujících částech:

* [Protokolování a citlivá data](#logging-and-sensitive-data)
* [Ochrana informací při přenosu pomocí protokolu HTTPS](#protect-information-in-transit-with-https)
* [Skriptování mezi weby (XSS)](#cross-site-scripting-xss)
* [Ochrana mezi zdroji](#cross-origin-protection)
* [Kliknutí – zdířka](#click-jacking)
* [Otevřít přesměrování](#open-redirects)

### <a name="logging-and-sensitive-data"></a>Protokolování a citlivá data

Interakce interoperability v JS mezi klientem a serverem se zaznamenávají do protokolů serveru s <xref:Microsoft.Extensions.Logging.ILogger> instancemi. Blazor se vyhnete protokolování citlivých informací, jako jsou například skutečné události nebo vstupy a výstupy interoperability v JS.

Když na serveru dojde k chybě, rozhraní upozorní klienta a rozhlasí relaci. Ve výchozím nastavení klient obdrží obecnou chybovou zprávu, která se může zobrazit v vývojářských nástrojích prohlížeče.

Chyba na straně klienta neobsahuje zásobník volání a neposkytuje podrobné informace o příčině chyby, ale protokoly serveru obsahují takové informace. Pro účely vývoje lze klientovi zpřístupnit citlivé informace o chybách tím, že povolíte podrobné chyby.

Povolit podrobné chyby pomocí:

* `CircuitOptions.DetailedErrors`.
* konfigurační klíč `DetailedErrors`. Například nastavte proměnnou prostředí `ASPNETCORE_DETAILEDERRORS` na hodnotu `true`.

> [!WARNING]
> Odhalení informací o chybách klientům na internetu je bezpečnostní riziko, které by se mělo vždy vyhnout.

### <a name="protect-information-in-transit-with-https"></a>Ochrana informací při přenosu pomocí protokolu HTTPS

Blazor Server používá SignalR ke komunikaci mezi klientem a serverem. Blazor Server normálně používá přenos, který SignalR vyjednávat, což je obvykle WebSockets.

Blazor Server nezajišťuje integritu a důvěrnost dat odesílaných mezi serverem a klientem. Vždy používat protokol HTTPS.

### <a name="cross-site-scripting-xss"></a>Skriptování mezi weby (XSS)

Skriptování mezi weby (XSS) umožňuje neoprávněné straně spustit libovolnou logiku v kontextu prohlížeče. Ohrožená aplikace by mohla spustit libovolný kód v klientovi. Chybu zabezpečení lze použít k potenciálnímu provedení určitého množství škodlivých akcí proti serveru:

* Odeslání falešných/neplatných událostí na server.
* Odeslání neúspěšných nebo neplatných dokončení vykreslování.
* Vyhněte se odesílání doplňování vykreslování.
* Odešlete volání Interop z JavaScriptu do .NET.
* Upravte odpověď volání Interop z rozhraní .NET na JavaScript.
* Vyhněte se odesílání výsledků interoperability .NET to JS.

Rozhraní Blazor serveru přijímá kroky k ochraně před některými předchozími hrozbami:

* Ukončí vytváření nových aktualizací uživatelského rozhraní, pokud klient nepotvrzující dávky vykreslování. Nakonfigurováno pomocí `CircuitOptions.MaxBufferedUnacknowledgedRenderBatches`.
* Vyprší časový limit volání rozhraní .NET do JavaScriptu po jedné minutě, aniž by byla obdržena odpověď od klienta. Nakonfigurováno pomocí `CircuitOptions.JSInteropDefaultCallTimeout`.
* Provede základní ověření na všech vstupech přicházejících z prohlížeče během interoperability JS:
  * Odkazy .NET jsou platné a typ očekávaný metodou .NET.
  * Data nejsou poškozena.
  * V datové části je uveden správný počet argumentů pro metodu.
  * Argumenty nebo výsledky lze před vyvoláním metody deserializovat správně.
* Provede základní ověření ve všech vstupech přicházejících z prohlížeče z odeslaných událostí:
  * Událost má platný typ.
  * Data pro událost lze deserializovat.
  * K události je přidružená obslužná rutina události.

Kromě ochrany, kterou implementuje rozhraní, musí být aplikace kódována vývojářem k ochraně před hrozbami a provádět příslušné akce:

* Při zpracování událostí vždy ověřovat data.
* Po přijetí neplatných dat proveďte příslušnou akci:
  * Ignorujte data a vraťte se. Tím umožníte, aby aplikace pokračovala v zpracování požadavků.
  * Pokud aplikace zjistí, že vstup je illegitimate a nemohl být vytvořen legitimním klientem, vyvolejte výjimku. Došlo k výjimce při odtrhlině okruhu a ukončení relace.
* Nedůvěra na chybovou zprávu poskytovanou dokončováním dávek vykreslování zahrnutými v protokolech. Klient poskytuje chybu a nemůže být obecně důvěryhodný, protože může dojít k ohrožení zabezpečení klienta.
* Nedůvěřovat vstupu volání interoperability JS v obou směrech mezi metodami jazyka JavaScript a .NET.
* Aplikace zodpovídá za ověření, že obsah argumentů a výsledků je platný, i když jsou argumenty nebo výsledky správně deserializovány.

Aby mohla existovat ohrožení zabezpečení XSS, musí aplikace na vykreslené stránce zahrnovat vstup uživatele. komponenty serveru Blazor spouští krok v čase kompilace, ve kterém je kód v souboru *. Razor* transformován do procedurální C# logiky. V době běhu C# logika vytvoří *strom vykreslení* , který popisuje prvky, text a podřízené komponenty. To se aplikuje na DOM v prohlížeči pomocí sekvence instrukcí JavaScriptu (nebo je v případě předvykreslování serializovaná na HTML):

* Uživatelský vstup vykreslený pomocí normálního syntaxe Razor (například `@someStringValue`) nevystavuje zranitelnost XSS, protože syntaxe Razor je přidána do modelu DOM prostřednictvím příkazů, které mohou zapisovat pouze text. I v případě, že hodnota obsahuje kód HTML, hodnota se zobrazí jako statický text. Při předběžné vykreslování je výstupem kódovaný HTML, který také zobrazuje obsah jako statický text.
* Značky skriptu nejsou povoleny a neměly by být zahrnuty do stromu vykreslování součásti aplikace. Je-li značka skriptu obsažena v kódu komponenty, je vygenerována chyba při kompilaci.
* Autoři součástí mohou vytvářet komponenty v C# nástroji bez použití Razor. Autor komponenty zodpovídá za použití správných rozhraní API při generování výstupu. Můžete například použít `builder.AddContent(0, someUserSuppliedString)` a *nikoli* `builder.AddMarkupContent(0, someUserSuppliedString)`, protože by to mohlo vytvořit chybu zabezpečení XSS.

Jako součást ochrany proti útokům XSS zvažte implementaci zmírnění XSS, jako je například [zásada zabezpečení obsahu (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP).

Další informace najdete v tématu <xref:security/cross-site-scripting>.

### <a name="cross-origin-protection"></a>Ochrana mezi zdroji

Útoky mezi zdroji zahrnují klienta z jiného zdroje, který provádí akci na serveru. Škodlivá akce je obvykle požadavek GET nebo POST formuláře (pro padělání žádostí mezi weby, CSRF), ale otevírání škodlivého WebSocket je také možné. aplikace Blazor serveru nabízejí [stejné záruky jako všechny ostatní aplikace SignalR pomocí nabídky protokolu rozbočovače](xref:signalr/security):

* k aplikacím Blazor serveru se dá přihlédnout, pokud nejsou podniknuty další míry, aby se zabránilo. Pokud chcete zakázat přístup pro více zdrojů, buď zakažte CORS v koncovém bodu přidáním middlewaru CORS do kanálu a přidáním `DisableCorsAttribute` do metadat koncového bodu Blazor nebo omezte sadu povolených zdrojů tak, že [nakonfigurujete SignalR pro sdílení prostředků mezi zdroji](xref:signalr/security#cross-origin-resource-sharing).
* Pokud je CORS povolená, můžou se při ochraně aplikace v závislosti na konfiguraci CORS vyžadovat další kroky. Pokud je CORS povolena globálně, může být CORS pro Blazor serverového centra zakázaná přidáním metadat `DisableCorsAttribute` do metadat koncového bodu po volání `hub.MapBlazorHub()`.

Další informace najdete v tématu <xref:security/anti-request-forgery>.

### <a name="click-jacking"></a>Kliknutí – zdířka

Kliknutí na zásuvky zahrnuje vykreslování lokality jako `<iframe>` v rámci lokality z jiného zdroje, aby uživatel mohl v rámci útoku přimět uživatele k provádění akcí na webu.

K ochraně aplikace před vykreslováním v `<iframe>`použijte [zásady zabezpečení obsahu (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) a záhlaví `X-Frame-Options`. Další informace najdete v tématu [MDN web Docs: X-frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options).

### <a name="open-redirects"></a>Otevřít přesměrování

Když se spustí relace aplikace Blazor serveru, Server provede základní ověření adres URL odesílaných v rámci zahájení relace. Architektura kontroluje, zda je základní adresa URL nadřazena aktuální adrese URL před vytvořením okruhu. Rozhraní neprovádí žádné další kontroly.

Když uživatel vybere odkaz na klientovi, adresa URL odkazu se pošle na server, který určuje, jakou akci chcete provést. Aplikace může například provádět navigaci na straně klienta nebo naznačit prohlížeči, aby přešel do nového umístění.

Komponenty mohou také aktivovat požadavky na navigaci programově pomocí `NavigationManager`. V takových scénářích může aplikace provést navigaci na straně klienta nebo v prohlížeči označit, že se má přejít na nové umístění.

Komponenty musí:

* Nepoužívejte vstup uživatele jako součást argumentů volání navigace.
* Ověřte argumenty a ujistěte se, že je cíl povolený aplikací.

V opačném případě může uživatel se zlými úmysly vynutit, aby prohlížeč přešel k webu kontrolovanému útočníkem. V tomto scénáři útočník získá aplikaci do použití určitého vstupu uživatele v rámci vyvolání metody `NavigationManager.Navigate`.

Tato doporučení platí i při vykreslování odkazů jako součást aplikace:

* Pokud je to možné, používejte relativní odkazy.
* Ověřte, zda jsou cíle absolutních odkazů platné, než je zařadíte do stránky.

Další informace najdete v tématu <xref:security/preventing-open-redirects>.

## <a name="authentication-and-authorization"></a>Ověřování a autorizace

Pokyny k ověřování a autorizaci najdete v tématu <xref:security/blazor/index>.

## <a name="security-checklist"></a>Kontrolní seznam zabezpečení

Následující seznam bezpečnostních otázek není vyčerpávající:

* Ověřte argumenty z událostí.
* Ověřte vstupy a výsledky volání interoperability JS.
* Nepoužívejte (nebo ověřte předem) uživatelský vstup pro volání interoperability .NET to JS.
* Zabrání klientovi v přidělování nevázané velikosti paměti.
  * Data v rámci součásti.
  * do klienta byly vráceny `DotNetObject` odkazy.
* Ochrana proti více odesláním.
* Zrušit dlouhotrvající operace, když je komponenta vyřazena.
* Vyhněte se událostem, které vytváří velké objemy dat.
* Vyhněte se použití vstupu uživatele jako součásti volání `NavigationManager.Navigate` a ověřte vstup uživatele pro adresy URL před sadou povolených původních zdrojů, pokud je to nenevyhnutelné.
* Neprovádějte rozhodnutí o autorizaci na základě stavu uživatelského rozhraní, ale jenom ze stavu součásti.
* Zvažte použití [zásad zabezpečení obsahu (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) k ochraně před útoky XSS.
* Zvažte použití CSP a [možností X-frame –](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options) k ochraně před kliknutím na konektory.
* Zajistěte, aby nastavení CORS byla vhodná při povolování CORS nebo explicitně zakázal CORS pro aplikace Blazor.
* Otestujte, abyste zajistili, že omezení na straně serveru pro aplikaci Blazor poskytují přijatelné uživatelské prostředí bez nepřijatelných úrovní rizika.
