---
title: 'Kurz: Implementace dědičnosti – ASP.NET MVC pomocí EF Core'
description: Tento kurz vám ukáže, jak implementovat dědičnost v datovém modelu pomocí Entity Framework Core v ASP.NET Core aplikaci.
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 03/27/2019
ms.topic: tutorial
uid: data/ef-mvc/inheritance
ms.openlocfilehash: c10df60a43f5d59f3ce13afd38aad42b88c80516
ms.sourcegitcommit: 7d3c6565dda6241eb13f9a8e1e1fd89b1cfe4d18
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/11/2019
ms.locfileid: "72259402"
---
# <a name="tutorial-implement-inheritance---aspnet-mvc-with-ef-core"></a>Kurz: Implementace dědičnosti – ASP.NET MVC pomocí EF Core

V předchozím kurzu jste zpracovali výjimky souběžnosti. Tento kurz vám ukáže, jak implementovat dědičnost v datovém modelu.

V objektově orientovaném programování můžete použít dědičnost k usnadnění opětovného použití kódu. V tomto kurzu změníte třídy `Instructor` a `Student` tak, aby byly odvozeny od základní třídy `Person`, která obsahuje vlastnosti, jako je například `LastName`, které jsou společné pro instruktory i studenty. Nepřidáte ani neměníte žádné webové stránky, ale změníte část kódu a tyto změny se automaticky projeví v databázi.

V tomto kurzu:

> [!div class="checklist"]
> * Mapování dědičnosti na databázi
> * Vytvoření třídy Person
> * Aktualizace instruktora a studenta
> * Přidat osobu do modelu
> * Vytváření a aktualizace migrací
> * Testování implementace

## <a name="prerequisites"></a>Požadované součásti

* [Souběžnost popisovačů](concurrency.md)

## <a name="map-inheritance-to-database"></a>Mapování dědičnosti na databázi

Třídy `Instructor` a `Student` v modelu školních dat mají identické i několik vlastností:

![Třídy student a instruktor](inheritance/_static/no-inheritance.png)

Předpokládejme, že chcete eliminovat redundantní kód pro vlastnosti, které jsou sdíleny entitami `Instructor` a `Student`. Nebo chcete napsat službu, která může formátovat názvy bez caring, jestli název pochází od instruktora nebo studenta. Můžete vytvořit základní třídu `Person`, která obsahuje pouze tyto sdílené vlastnosti, a poté nastavit třídy `Instructor` a `Student` z této základní třídy, jak je znázorněno na následujícím obrázku:

![Třídy studenta a instruktory odvozené od třídy Person](inheritance/_static/inheritance.png)

Existuje několik způsobů, jak lze tuto strukturu dědičnosti znázornit v databázi. Můžete mít tabulku Person, která obsahuje informace o studentech i instruktorech v jedné tabulce. Některé sloupce mohou být použity pouze pro instruktory (ZaměstnánOd), některé pouze pro studenty (EnrollmentDate), některá pro obě (LastName, FirstName). Obvykle byste měli sloupec diskriminátoru, který určuje, který typ každý řádek představuje. Sloupec diskriminátoru může mít například "instruktor" pro studenty a studenta.

![Příklad tabulky na hierarchii](inheritance/_static/tph.png)

Tento model generování struktury dědičnosti entit z jedné databázové tabulky se nazývá dědičnost typu tabulka-na hierarchii (TPH).

Alternativou je, že databáze vypadá podobně jako struktura dědičnosti. Například můžete mít pouze pole název v tabulce Person a mají samostatné tabulky instruktor a student s poli data.

![Dědičnost tabulek podle typu](inheritance/_static/tpt.png)

Tento model vytvoření tabulky databáze pro každou třídu entity se nazývá dědičnost tabulky podle typu (TPT).

Ještě další možností je mapovat všechny neabstraktní typy na jednotlivé tabulky. Všechny vlastnosti třídy, včetně děděných vlastností, jsou mapovány na sloupce odpovídající tabulky. Tento model se nazývá dědičnost tříd (TPC) podle konkrétní třídy. Pokud jste implementovali TPC dědění pro třídy Person, student a instruktor, jak je uvedeno výše, tabulky student a instruktor by po implementaci dědění nevypadaly jinak než předtím.

Vzorce dědičnosti TPC a TPH obvykle poskytují lepší výkon než vzory dědičnosti TPT, protože vzory TPT mohou mít za následek složité spojení dotazů.

Tento kurz ukazuje, jak implementovat dědičnosti TPH. TPH je jediný vzorek dědičnosti, který Entity Framework Core podporuje.  To, co uděláte, je vytvořit třídu `Person`, změnit třídy `Instructor` a `Student` tak, aby byly odvozeny od `Person`, přidat novou třídu do `DbContext` a vytvořit migraci.

> [!TIP]
> Zvažte uložení kopie projektu před provedením následujících změn.  Pak Pokud narazíte na problémy a potřebujete začít znovu, bude snazší začít z uloženého projektu místo vrácení kroků provedených pro tento kurz nebo přechod zpět na začátek celé řady.

## <a name="create-the-person-class"></a>Vytvoření třídy Person

Ve složce modely vytvořte Person.cs a nahraďte kód šablony následujícím kódem:

[!code-csharp[](intro/samples/cu/Models/Person.cs)]

## <a name="update-instructor-and-student"></a>Aktualizace instruktora a studenta

V *Instructor.cs*odvodíte třídu Instructor z třídy Person a odstraňte pole klíč a název. Kód bude vypadat jako v následujícím příkladu:

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_AfterInheritance&highlight=8)]

Udělejte stejné změny v *student.cs*.

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_AfterInheritance&highlight=8)]

## <a name="add-person-to-the-model"></a>Přidat osobu do modelu

Do *SchoolContext.cs*přidejte typ entity Person. Nové řádky jsou zvýrazněny.

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_AfterInheritance&highlight=19,30)]

To je vše, co Entity Framework potřebuje, aby bylo možné nakonfigurovat dědičnost tabulek na hierarchii. Jak vidíte, při aktualizaci databáze bude mít uživatelskou tabulku místo tabulek student a instruktor.

## <a name="create-and-update-migrations"></a>Vytváření a aktualizace migrací

Uložte změny a sestavte projekt. Pak otevřete okno příkazového řádku ve složce projektu a zadejte následující příkaz:

```dotnetcli
dotnet ef migrations add Inheritance
```

Ještě nespouštějte příkaz `database update`. Tento příkaz bude mít za následek ztrátu dat, protože odstraní tabulku instruktora a přejmenuje tabulku student na Person. Aby bylo možné zachovat existující data, je třeba zadat vlastní kód.

Otevřete *migrace/\<timestamp > _Inheritance. cs* a nahraďte metodu `Up` následujícím kódem:

[!code-csharp[](intro/samples/cu/Migrations/20170216215525_Inheritance.cs?name=snippet_Up)]

Tento kód má na starosti následující úlohy aktualizace databáze:

* Odebere omezení a indexy cizího klíče, které odkazují na tabulku studenta.

* Přejmenuje tabulku instruktora jako osobu a provede změny potřebné k uložení dat studenta:

* Přidá EnrollmentDate s možnou hodnotou null pro studenty.

* Přidá sloupec diskriminátoru, který označuje, zda je řádek určen studentem nebo instruktorem.

* Vytvoří hodnotu Nullable s možnou hodnotou null, protože řádky studenta nebudou mít data přijetí.

* Přidá dočasné pole, které bude použito k aktualizaci cizích klíčů, které odkazují na studenty. Když zkopírujete studenty do tabulky Person, získají se nové hodnoty primárního klíče.

* Zkopíruje data z tabulky student do tabulky Person (osoba). To způsobí, že studenti získají přiřazené nové hodnoty primárního klíče.

* Opravuje hodnoty cizích klíčů, které odkazují na studenty.

* Znovu vytvoří omezení a indexy cizího klíče, které se teď odkazují na tabulku Person.

(Pokud jste použili GUID místo celého čísla jako typ primárního klíče, hodnoty primárního klíče studenta se nemusejí změnit a některé z těchto kroků by mohly být vynechány.)

Spusťte příkaz `database update`:

```dotnetcli
dotnet ef database update
```

(V produkčním systému provedete odpovídající změny metody `Down` v případě, že byste někdy museli použít tuto metodu, abyste se mohli vrátit k předchozí verzi databáze. V tomto kurzu nebudete používat metodu `Down`.)

> [!NOTE]
> Při provádění změn schématu v databázi, která obsahuje existující data, je možné získat další chyby. Pokud získáte chyby migrace, které nelze vyřešit, můžete buď změnit název databáze v připojovacím řetězci nebo odstranit databázi. V případě nové databáze není k dispozici žádná data k migraci a příkaz Update-Database je pravděpodobnější, že se dokončí bez chyb. Databázi odstraníte tak, že použijete SSOX nebo spustíte příkaz `database drop` CLI.

## <a name="test-the-implementation"></a>Testování implementace

Spusťte aplikaci a vyzkoušejte si různé stránky. Vše funguje stejně jako dříve.

V **Průzkumník objektů systému SQL Server**rozbalte **data připojení/SchoolContext** a pak **tabulky**a uvidíte, že tabulky student a instruktor byly nahrazeny tabulkou Person. Otevřete návrháře tabulky osoba a uvidíte, že obsahuje všechny sloupce, které se používají v tabulkách student a instruktor.

![Tabulka Person v SSOX](inheritance/_static/ssox-person-table.png)

Klikněte pravým tlačítkem myši na tabulku Person a potom kliknutím na možnost **Zobrazit data tabulky** zobrazte sloupec diskriminátor.

![Tabulka Person v tabulce SSOX data](inheritance/_static/ssox-person-data.png)

## <a name="get-the-code"></a>Získat kód

[Stažení nebo zobrazení dokončené aplikace.](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a>Další materiály a zdroje informací

Další informace o dědičnosti v Entity Framework Core naleznete v tématu [Dědičnost](/ef/core/modeling/inheritance).

## <a name="next-steps"></a>Další kroky

V tomto kurzu:

> [!div class="checklist"]
> * Namapovaná dědičnost na databázi
> * Byla vytvořena třída Person.
> * Aktualizovaný instruktor a student
> * Do modelu se přidala osoba.
> * Vytvořené a aktualizované migrace
> * Otestování implementace

Přejděte k dalšímu kurzu, kde se dozvíte, jak zvládnout celou řadu poměrně pokročilých scénářů Entity Framework.

> [!div class="nextstepaction"]
> [Další: Pokročilá témata](advanced.md)
