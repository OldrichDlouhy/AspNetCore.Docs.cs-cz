---
title: 'Kurz: Aktualizace souvisejících dat – ASP.NET MVC pomocí EF Core'
description: V tomto kurzu aktualizujete související data aktualizací polí cizího klíče a vlastností navigace.
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 03/27/2019
ms.topic: tutorial
uid: data/ef-mvc/update-related-data
ms.openlocfilehash: 98f9f780c5814c0bd6e33052ee812b01a2bce306
ms.sourcegitcommit: 7d3c6565dda6241eb13f9a8e1e1fd89b1cfe4d18
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/11/2019
ms.locfileid: "72259361"
---
# <a name="tutorial-update-related-data---aspnet-mvc-with-ef-core"></a>Kurz: Aktualizace souvisejících dat – ASP.NET MVC pomocí EF Core

V předchozím kurzu jste zobrazili související data. v tomto kurzu aktualizujete související data aktualizací polí cizího klíče a vlastností navigace.

Následující ilustrace znázorňují některé stránky, se kterými budete pracovat.

![Stránka pro úpravu kurzu](update-related-data/_static/course-edit.png)

![Stránka pro úpravu instruktora](update-related-data/_static/instructor-edit-courses.png)

V tomto kurzu:

> [!div class="checklist"]
> * Přizpůsobení stránek kurzů
> * Přidat stránku pro úpravu instruktorů
> * Přidání kurzů pro úpravu stránky
> * Aktualizace stránky pro odstranění
> * Přidání umístění Office a kurzů k vytvoření stránky

## <a name="prerequisites"></a>Požadované součásti

* [Čtení souvisejících dat](read-related-data.md)

## <a name="customize-courses-pages"></a>Přizpůsobení stránek kurzů

Když se vytvoří nová entita kurzu, musí mít relaci s existujícím oddělením. Pro usnadnění tohoto kódu se vygenerovaný kód skládá z metod kontroleru a vytváření a upravování zobrazení, která obsahují rozevírací seznam pro výběr oddělení. Rozevírací seznam nastaví vlastnost cizího klíče `Course.DepartmentID` a to je vše, co Entity Framework potřebuje, aby se načetla vlastnost navigace `Department` s příslušnou entitou oddělení. Použijete generovaný kód, ale mírně ho změníte a přidáte tak zpracování chyb a seřadíte rozevírací seznam.

V *CoursesController.cs*odstraňte čtyři metody Create a Edit a nahraďte je následujícím kódem:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreatePost)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditPost)]

Po metodě `Edit` HttpPost vytvořte novou metodu, která načte informace o oddělení v rozevíracím seznamu.

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_Departments)]

Metoda `PopulateDepartmentsDropDownList` načte seznam všech oddělení seřazených podle názvu, vytvoří kolekci `SelectList` pro rozevírací seznam a předá kolekci do zobrazení v `ViewBag`. Metoda přijímá volitelný parametr `selectedDepartment`, který umožňuje volajícímu kódu určit položku, která bude vybrána při vykreslení rozevíracího seznamu. Zobrazení pojmenuje název "DepartmentID" do pomocné rutiny značky `<select>` a pomoc pak ví, že hledá objekt `ViewBag` pro `SelectList` s názvem "DepartmentID".

Metoda HttpGet `Create` volá metodu `PopulateDepartmentsDropDownList` bez nastavení vybrané položky, protože pro nový kurz se oddělení ještě nevytvořilo:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=3&name=snippet_CreateGet)]

Metoda HttpGet `Edit` Nastaví vybranou položku na základě ID oddělení, které je již přiřazeno k upravovanému kurzu:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=15&name=snippet_EditGet)]

Metody HttpPost pro `Create` a `Edit` také obsahují kód, který nastaví vybranou položku při opětovném zobrazení stránky po chybě. Tím se zajistí, že se při zobrazení stránky zobrazí chybová zpráva bez ohledu na vybrané oddělení zůstane vybraná možnost.

### <a name="add-asnotracking-to-details-and-delete-methods"></a>Přidávání. AsNoTracking metody Details a DELETE

Pokud chcete optimalizovat výkon pro podrobnosti kurzu a odstranit stránky, přidejte do metod `Details` a HttpGet `Delete` volání `AsNoTracking`.

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_Details)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_DeleteGet)]

### <a name="modify-the-course-views"></a>Úprava zobrazení kurzu

V okně *zobrazení/kurzy/vytvořit. cshtml*přidejte možnost "vybrat oddělení" do rozevíracího seznamu **oddělení** , změňte titulek z **DepartmentID** na **oddělení**a přidejte ověřovací zprávu.

[!code-html[](intro/samples/cu/Views/Courses/Create.cshtml?highlight=2-6&range=29-34)]

V *zobrazeních, kurzech/úpravách. cshtml*udělejte stejnou změnu pro pole oddělení, které jste právě vytvořili v části *vytvoření. cshtml*.

Také v *zobrazeních/kurzech/upravit. cshtml*přidejte pole číslo kurzu před pole **název** . Vzhledem k tomu, že číslo kurzu je primární klíč, zobrazuje se, ale nedá se změnit.

[!code-html[](intro/samples/cu/Views/Courses/Edit.cshtml?range=15-18)]

Pro číslo kurzu v zobrazení pro úpravy již existuje skryté pole (`<input type="hidden">`). Přidání pomocníka značek `<label>` eliminuje nutnost skrytého pole, protože nezpůsobí, že se číslo kurzu zahrne do publikovaných dat, když uživatel klikne na **Uložit** na stránce pro **Úpravy** .

V *zobrazení/kurzy/odstranit. cshtml*přidejte do horní části pole číslo kurzu a změňte ID oddělení na název oddělení.

[!code-html[](intro/samples/cu/Views/Courses/Delete.cshtml?highlight=14-19,36)]

V *zobrazeních/kurzech/details. cshtml*udělejte stejnou změnu, kterou jste právě provedli pro *odstranění. cshtml*.

### <a name="test-the-course-pages"></a>Testování stránek kurzu

Spusťte aplikaci, vyberte kartu **kurzy** , klikněte na **vytvořit novou**a zadejte data nového kurzu:

![Stránka vytvoření kurzu](update-related-data/_static/course-create.png)

Klikněte na **Vytvořit**. Stránka s rejstříkem kurzů se zobrazí spolu s novým kurzem přidaným do seznamu. Název oddělení v seznamu stránek indexu pochází z navigační vlastnosti, která ukazuje, že relace byla správně vytvořena.

V kurzu na stránce s rejstříkem kurzů klikněte na **Upravit** .

![Stránka pro úpravu kurzu](update-related-data/_static/course-edit.png)

Změňte data na stránce a klikněte na **Uložit**. Stránka s rejstříkem kurzů se zobrazuje s aktualizovanými daty kurzu.

## <a name="add-instructors-edit-page"></a>Přidat stránku pro úpravu instruktorů

Když upravíte záznam instruktora, chcete mít schopnost aktualizovat přiřazení kanceláře instruktora. Entita Instructor má relaci 1:1 s entitou OfficeAssignment, což znamená, že váš kód musí zpracovávat následující situace:

* Pokud uživatel zruší přiřazení Office a původně měl hodnotu, odstraňte entitu OfficeAssignment.

* Pokud uživatel zadá hodnotu přiřazení Office, která byla původně prázdná, vytvořte novou entitu OfficeAssignment.

* Pokud uživatel změní hodnotu přiřazení kanceláře, změňte hodnotu v existující entitě OfficeAssignment.

### <a name="update-the-instructors-controller"></a>Aktualizace kontroleru instruktorů

V *InstructorsController.cs*změňte kód v metodě HttpGet `Edit` tak, aby byla načtena navigační vlastnost `OfficeAssignment` entity instruktora a volání `AsNoTracking`:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=8-11&name=snippet_EditGetOA)]

Chcete-li zpracovat aktualizace přiřazení sady Office, nahraďte metodu HttpPost `Edit` následujícím kódem:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EditPostOA)]

Kód provede následující:

* Změní název metody na `EditPost`, protože signatura je nyní shodná s metodou HttpGet `Edit` (atribut `ActionName` určuje, že se adresa URL `/Edit/` stále používá).

* Získá aktuální entitu instruktor z databáze pomocí Eager načítání pro navigační vlastnost `OfficeAssignment`. To se shoduje s tím, co jste provedli v metodě HttpGet `Edit`.

* Aktualizuje načtenou entitu Instructor hodnotami z pořadače modelů. Přetížení `TryUpdateModel` umožňuje seznam povolených vlastností, které chcete zahrnout. Tím zabráníte převzetí služeb při selhání, jak je vysvětleno v [druhém kurzu](crud.md).

    <!-- Snippets don't play well with <ul> [!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=241-244)] -->

    ```csharp
    if (await TryUpdateModelAsync<Instructor>(
        instructorToUpdate,
        "",
        i => i.FirstMidName, i => i.LastName, i => i.HireDate, i => i.OfficeAssignment))
    ```

* Pokud je umístění kanceláře prázdné, vlastnost Instructor. OfficeAssignment nastaví na hodnotu null, aby se související řádek v tabulce OfficeAssignment odstranil.

    <!-- Snippets don't play well with <ul>  "intro/samples/cu/Controllers/InstructorsController.cs"} -->

    ```csharp
    if (String.IsNullOrWhiteSpace(instructorToUpdate.OfficeAssignment?.Location))
    {
        instructorToUpdate.OfficeAssignment = null;
    }
    ```

* Uloží změny do databáze.

### <a name="update-the-instructor-edit-view"></a>Aktualizace zobrazení pro úpravy instruktora

V *zobrazeních/instruktorech/upravit. cshtml*přidejte nové pole pro úpravy umístění kanceláře, a to na konci před tlačítkem **Uložit** :

[!code-html[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=30-34)]

Spusťte aplikaci, vyberte kartu **instruktoři** a potom klikněte na tlačítko **Upravit** v instruktorovi. Změňte **umístění kanceláře** a klikněte na **Uložit**.

![Stránka pro úpravu instruktora](update-related-data/_static/instructor-edit-office.png)

## <a name="add-courses-to-edit-page"></a>Přidání kurzů pro úpravu stránky

Instruktoři můžou učit libovolný počet kurzů. Nyní vylepšíte stránku pro úpravu instruktoru přidáním možnosti změnit přiřazení kurzů pomocí skupiny zaškrtávacích políček, jak je znázorněno na následujícím snímku obrazovky:

![Stránka pro úpravu instruktorů s kurzy](update-related-data/_static/instructor-edit-courses.png)

Vztah mezi entitami Course a instruktorem je mnoho až mnoho. Chcete-li přidat a odebrat relace, přidejte a odeberte entity do a ze sady entit CourseAssignments JOIN.

Uživatelské rozhraní, které umožňuje změnit, ke kterým kurzům je instruktor přiřazen, je skupina zaškrtávacích políček. Zobrazí se zaškrtávací políčko pro každý kurz v databázi a jsou vybrány ty, ke kterým je instruktor aktuálně přiřazen. Uživatel může zaškrtnutím nebo zrušením zaškrtnutí políček změnit přiřazení kurzů. Pokud byl počet kurzů mnohem větší, pravděpodobně budete chtít použít jinou metodu, která prezentují data v zobrazení, ale použijte stejnou metodu manipulace s entitou JOIN k vytváření nebo odstraňování relací.

### <a name="update-the-instructors-controller"></a>Aktualizace kontroleru instruktorů

Chcete-li poskytnout data do zobrazení pro seznam zaškrtávacích políček, použijte třídu zobrazení modelu.

Vytvořte *AssignedCourseData.cs* ve složce *SchoolViewModels* a nahraďte existující kód následujícím kódem:

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/AssignedCourseData.cs)]

V *InstructorsController.cs*nahraďte metodu HttpGet `Edit` následujícím kódem. Změny jsou zvýrazněny.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=10,17,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&name=snippet_EditGetCourses)]

Kód přidá Eager načítání pro vlastnost navigace `Courses` a zavolá novou metodu `PopulateAssignedCourseData` k poskytnutí informací pro pole zaškrtávacího políčka pomocí třídy zobrazení `AssignedCourseData`.

Kód v metodě `PopulateAssignedCourseData` čte prostřednictvím všech entit kurzu, aby bylo možné načíst seznam kurzů pomocí třídy zobrazení modelu. Pro každý kurz kód kontroluje, zda se v rámci navigační vlastnosti `Courses` instruktory nachází. Chcete-li vytvořit efektivní vyhledávání při kontrole, zda je kurz přiřazen instruktorovi, jsou kurzy přiřazené k instruktorovi vloženy do kolekce `HashSet`. Vlastnost `Assigned` je nastavena na hodnotu true pro kurzy, ke kterým je instruktor přiřazen. Zobrazení použije tuto vlastnost k určení, která zaškrtávací políčka se musí zobrazit jako vybraná. Nakonec se seznam předává do zobrazení v `ViewData`.

Dále přidejte kód, který se spustí, když uživatel klikne na **Uložit**. Nahraďte metodu `EditPost` následujícím kódem a přidejte novou metodu, která aktualizuje navigační vlastnost `Courses` entity Instructor.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=1,3,12,13,25,39-40&name=snippet_EditPostCourses)]

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=1-31)]

Signatura metody se teď liší od metody HttpGet `Edit`, takže se název metody změní z `EditPost` zpátky na `Edit`.

Vzhledem k tomu, že zobrazení nemá kolekci entit kurzu, nemůže pořadač modelů automaticky aktualizovat navigační vlastnost `CourseAssignments`. Místo použití pořadače modelů k aktualizaci navigační vlastnosti `CourseAssignments` provedete v nové metodě `UpdateInstructorCourses`. Proto je nutné vyloučit vlastnost `CourseAssignments` z vazby modelu. To nevyžaduje žádné změny kódu, který volá `TryUpdateModel`, protože používáte přetížení seznamu povolených položek a `CourseAssignments` nejsou v seznamu zahrnutí.

Pokud nebyla vybrána žádná zaškrtávací políčka, kód v `UpdateInstructorCourses` inicializuje vlastnost navigace `CourseAssignments` s prázdnou kolekcí a vrátí:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=3-7)]

Kód pak projde všemi kurzy v databázi a zkontroluje každý kurz s těmi, které jsou aktuálně přiřazeny k instruktorovi, a k těm, které byly vybrány v zobrazení. Aby bylo možné efektivně vyhledávat, jsou tyto dvě kolekce uloženy v objektech `HashSet`.

Pokud je vybráno zaškrtávací políčko pro kurz, ale kurz není v navigační vlastnosti `Instructor.CourseAssignments`, kurz se přidá do kolekce v navigační vlastnosti.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=14-20&name=snippet_UpdateCourses)]

Pokud není vybrané zaškrtávací políčko pro kurz, ale kurz je v navigační vlastnosti `Instructor.CourseAssignments`, kurz se odebere z navigační vlastnosti.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=21-29&name=snippet_UpdateCourses)]

### <a name="update-the-instructor-views"></a>Aktualizace zobrazení instruktora

V *zobrazeních/instruktorech/upravit. cshtml*přidejte pole **kurzů** s polem zaškrtávacích políček přidáním následujícího kódu hned za prvky `div` pro pole **Office** a před element `div` pro **Save** . tlačítko.

<a id="notepad"></a>
> [!NOTE]
> Když kód vložíte v aplikaci Visual Studio, mohou být zalomení řádků změněny způsobem, který kód přerušuje. Pokud kód po vložení vypadá jinak, stiskněte klávesovou zkratku CTRL + Z, aby bylo automatické formátování vráceno zpět. Tím dojde k odstranění konců řádků, aby vypadaly jako v tomto příkladu. Odsazení nemusí být dokonalé, ale řádky `@</tr><tr>`, `@:<td>`, `@:</td>` a `@:</tr>` musí být na jednom řádku, jak je znázorněno, nebo se zobrazí chyba za běhu. Po vybrání bloku nového kódu stiskněte klávesu Tabulátor třikrát, aby se nový kód pořádek nastavil s existujícím kódem. Tento problém je opravený v aplikaci Visual Studio 2019.

[!code-html[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=35-61)]

Tento kód vytvoří tabulku HTML, která má tři sloupce. V každém sloupci je zaškrtávací políčko následované titulkem, který se skládá z čísla a názvu kurzu. Všechna zaškrtávací políčka mají stejný název ("selectedCourses"), který informuje pořadač modelů o tom, že se mají považovat za skupinu. Atribut Value každé zaškrtávací políčko je nastaven na hodnotu `CourseID`. Po zveřejnění stránky předává pořadač modelu pole do kontroleru, který se skládá z hodnot `CourseID` pouze u zaškrtnutých políček.

Když jsou tato zaškrtávací políčka zpočátku vykreslena, jsou pro kurzy přiřazené instruktorem zkontrolovány atributy, které je vyberou (zobrazí zaškrtnutí).

Spusťte aplikaci, vyberte kartu **instruktory** a klikněte na tlačítko **Upravit** na instruktorovi, aby se zobrazila stránka pro **Úpravy** .

![Stránka pro úpravu instruktorů s kurzy](update-related-data/_static/instructor-edit-courses.png)

Změňte některá přiřazení kurzů a klikněte na Uložit. Změny, které provedete, se projeví na stránce indexu.

> [!NOTE]
> Postup, který je zde k dispozici pro úpravu dat kurzu instruktora, funguje dobře, pokud existuje omezený počet kurzů. Pro kolekce, které jsou mnohem větší, by se vyžadovalo jiné uživatelské rozhraní a odlišná metoda aktualizace.

## <a name="update-delete-page"></a>Aktualizace stránky pro odstranění

V *InstructorsController.cs*odstraňte metodu `DeleteConfirmed` a vložte následující kód na místo.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=5-7,9-12&name=snippet_DeleteConfirmed)]

Tento kód provede následující změny:

* Načítá Eager pro navigační vlastnost `CourseAssignments`. Musíte zahrnout tento nebo EF neznáte související entity `CourseAssignment` a nebude je odstraňovat. Abyste se vyhnuli nutnosti jejich čtení, můžete v databázi nakonfigurovat kaskádové odstranění.

* Pokud je instruktor, který má být odstraněn, přiřazen jako správce jakékoli oddělení, odebere z těchto oddělení přiřazení instruktora.

## <a name="add-office-location-and-courses-to-create-page"></a>Přidání umístění Office a kurzů k vytvoření stránky

V *InstructorsController.cs*odstraňte metody HttpGet a HTTPPOST `Create` a pak na jejich místo přidejte následující kód:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Create&highlight=3-5,12,14-22,29)]

Tento kód se podobá tomu, co jste viděli pro metody `Edit` s tím rozdílem, že zpočátku nejsou vybrané žádné kurzy. Metoda HttpGet `Create` volá metodu `PopulateAssignedCourseData`, protože by mohly být vybrány kurzy, ale za účelem poskytnutí prázdné kolekce pro smyčku `foreach` v zobrazení (jinak kód zobrazení by vyvolal výjimku s nulovým odkazem).

Metoda HttpPost `Create` přidá každý vybraný kurz do vlastnosti navigace `CourseAssignments` před tím, než zkontroluje chyby ověřování a přidá nového instruktora do databáze. Kurzy se přidávají i v případě, že dojde k chybám modelu, takže když dojde k chybám modelu (například uživatel zaznamená neplatné datum) a stránka se znovu zobrazí s chybovou zprávou, všechny vybrané kurzy se automaticky obnoví.

Všimněte si, že aby bylo možné přidat kurzy do navigační vlastnosti `CourseAssignments`, musíte tuto vlastnost inicializovat jako prázdnou kolekci:

```csharp
instructor.CourseAssignments = new List<CourseAssignment>();
```

Jako alternativu k tomu, že se jedná o kód kontroleru, to můžete provést v modelu instruktoru změnou vlastnosti getter na automaticky vytvořit kolekci, pokud neexistuje, jak je znázorněno v následujícím příkladu:

```csharp
private ICollection<CourseAssignment> _courseAssignments;
public ICollection<CourseAssignment> CourseAssignments
{
    get
    {
        return _courseAssignments ?? (_courseAssignments = new List<CourseAssignment>());
    }
    set
    {
        _courseAssignments = value;
    }
}
```

Pokud upravíte vlastnost `CourseAssignments` tímto způsobem, můžete v řadiči odebrat explicitní inicializační kód vlastnosti.

V *zobrazení/instruktor/vytvořit. cshtml*, přidejte textové pole umístění kanceláře a zaškrtávací políčka pro kurzy před tlačítkem Odeslat. Jako v případě stránky pro úpravy [opravte formátování, pokud aplikace Visual Studio přeformátuje kód při jeho vložení](#notepad).

[!code-html[](intro/samples/cu/Views/Instructors/Create.cshtml?range=29-61)]

Otestujte spuštěním aplikace a vytvořením instruktora.

## <a name="handling-transactions"></a>Zpracování transakcí

Jak je vysvětleno v [kurzu CRUD](crud.md), Entity Framework implicitně implementuje transakce. V případě scénářů, kde potřebujete větší kontrolu – například pokud chcete zahrnout operace provedené mimo Entity Framework v transakci – viz [transakce](/ef/core/saving/transactions).

## <a name="get-the-code"></a>Získat kód

[Stažení nebo zobrazení dokončené aplikace.](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Další kroky

V tomto kurzu:

> [!div class="checklist"]
> * Stránky přizpůsobených kurzů
> * Stránka pro úpravu přidaných instruktorů
> * Přidání kurzů pro úpravu stránky
> * Aktualizace stránky pro odstranění
> * Přidání umístění Office a kurzů k vytvoření stránky

Přejděte k dalšímu kurzu, kde se dozvíte, jak zpracovávat konflikty souběžnosti.

> [!div class="nextstepaction"]
> [Zpracování konfliktů souběžnosti](concurrency.md)
