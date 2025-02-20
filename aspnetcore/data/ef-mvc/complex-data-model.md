---
title: 'Kurz: vytvoření složitého datového modelu – ASP.NET MVC pomocí EF Core'
description: V tomto kurzu přidejte další entity a vztahy a upravte datový model zadáním formátování, ověřování a pravidel mapování.
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 03/27/2019
ms.topic: tutorial
uid: data/ef-mvc/complex-data-model
ms.openlocfilehash: b8b1ade4c8c29d34200bf8c0944cff6adec0bb95
ms.sourcegitcommit: f40c9311058c9b1add4ec043ddc5629384af6c56
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/21/2019
ms.locfileid: "74288952"
---
# <a name="tutorial-create-a-complex-data-model---aspnet-mvc-with-ef-core"></a>Kurz: vytvoření složitého datového modelu – ASP.NET MVC pomocí EF Core

V předchozích kurzech jste pracovali s jednoduchým datovým modelem, který se skládá ze tří entit. V tomto kurzu přidáte další entity a vztahy a budete přizpůsobovat datový model zadáním pravidel formátování, ověřování a mapování databáze.

Až budete hotovi, třídy entit vytvoří dokončený datový model, který je znázorněn na následujícím obrázku:

![Diagram entit](complex-data-model/_static/diagram.png)

V tomto kurzu se naučíte:

> [!div class="checklist"]
> * Přizpůsobení datového modelu
> * Provedení změn v entitě studenta
> * Vytvořit entitu instruktora
> * Vytvořit entitu OfficeAssignment
> * Upravit entitu kurzu
> * Vytvořit entitu oddělení
> * Upravit entitu registrace
> * Aktualizace kontextu databáze
> * Počáteční databáze s testovacími daty
> * Přidání migrace
> * Změna připojovacího řetězce
> * Aktualizace databáze

## <a name="prerequisites"></a>Požadavky

* [Použití migrace EF Core](migrations.md)

## <a name="customize-the-data-model"></a>Přizpůsobení datového modelu

V této části se dozvíte, jak přizpůsobit datový model pomocí atributů, které určují pravidla formátování, ověřování a mapování databáze. Potom v několika následujících částech vytvoříte kompletní model školních dat přidáním atributů do tříd, které jste už vytvořili, a vytvořením nových tříd pro zbývající typy entit v modelu.

### <a name="the-datatype-attribute"></a>Atribut DataType

Pro data o registraci studenta se aktuálně zobrazují všechny webové stránky, které jsou aktuálně současně s datem, i když jsou pro toto pole k disstará data. Pomocí atributů datových poznámek můžete vytvořit jednu změnu kódu, která bude opravovat formát zobrazení v každém zobrazení, které zobrazuje data. Chcete-li zobrazit příklad, jak to provést, přidáte atribut do vlastnosti `EnrollmentDate` ve třídě `Student`.

V *modelech/student. cs*přidejte příkaz `using` pro obor názvů `System.ComponentModel.DataAnnotations` a přidejte `DataType` a `DisplayFormat` atributů do vlastnosti `EnrollmentDate`, jak je znázorněno v následujícím příkladu:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_DataType&highlight=3,12-13)]

Atribut `DataType` slouží k zadání datového typu, který je konkrétnější než vnitřní typ databáze. V tomto případě chceme sledovat pouze datum, nikoli datum a čas. Výčet `DataType` poskytuje mnoho datových typů, jako je datum, čas, PhoneNumber, měna, EmailAddress a další. Atribut `DataType` může také povolit aplikaci automatické poskytování funkcí specifických pro typ. Například odkaz `mailto:` lze vytvořit pro `DataType.EmailAddress`a selektor data lze zadat pro `DataType.Date` v prohlížečích, které podporují HTML5. Atribut `DataType` emituje atributy HTML 5 `data-` (vyslovované datové přerušované), které mohou prohlížeče HTML 5 pochopit. Atributy `DataType` neposkytují žádné ověřování.

`DataType.Date` neurčuje formát data, které se zobrazí. Ve výchozím nastavení se datové pole zobrazuje v závislosti na výchozích formátech na základě objektu CultureInfo serveru.

Atribut `DisplayFormat` slouží k explicitnímu zadání formátu data:

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

Nastavení `ApplyFormatInEditMode` určuje, zda má být formátování použito také v případě, že je hodnota zobrazena v textovém poli pro úpravy. (Možná nebudete chtít, aby se pro některá pole, například pro hodnoty měny, nemuseli v textovém poli pro úpravy chtít symbol měny.)

Můžete použít atribut `DisplayFormat` sám o sobě, ale obecně je vhodné použít atribut `DataType` také. Atribut `DataType` předává sémantiku dat na rozdíl od způsobu vykreslování na obrazovce a poskytuje následující výhody, které nezískáte pomocí `DisplayFormat`:

* Prohlížeč může povolit funkce HTML5 (například pro zobrazení ovládacího prvku kalendáře, symbolu měny odpovídající národním prostředí, e-mailových odkazů, některých ověření vstupu na straně klienta atd.).

* Ve výchozím nastavení bude prohlížeč data vykreslovat pomocí správného formátu na základě vašeho národního prostředí.

Další informace najdete v dokumentaci k [pomocníka značek\<input >](../../mvc/views/working-with-forms.md#the-input-tag-helper).

Spusťte aplikaci, klikněte na stránku indexu studentů a Všimněte si, že časy se už nezobrazuje pro data registrace. Totéž platí pro všechna zobrazení, která používají model studenta.

![Stránka indexu studentů zobrazující data bez časů](complex-data-model/_static/dates-no-times.png)

### <a name="the-stringlength-attribute"></a>Atribut StringLength

Můžete také zadat pravidla ověřování dat a chybové zprávy ověřování pomocí atributů. Atribut `StringLength` nastaví maximální délku v databázi a poskytuje ověřování na straně klienta a na straně serveru pro ASP.NET Core MVC. V tomto atributu můžete také zadat minimální délku řetězce, ale minimální hodnota nemá žádný vliv na schéma databáze.

Předpokládejme, že chcete zajistit, aby uživatelé nezadali více než 50 znaků pro název. Chcete-li přidat toto omezení, přidejte `StringLength` atributy do vlastností `LastName` a `FirstMidName`, jak je znázorněno v následujícím příkladu:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_StringLength&highlight=10,12)]

Atribut `StringLength` nezabrání uživateli v zadání prázdného místa pro název. Můžete použít atribut `RegularExpression` pro použití omezení na vstup. Například následující kód vyžaduje, aby první znak byl velkými písmeny a aby zbývající znaky byly abecedně:

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z""'\s-]*$")]
```

Atribut `MaxLength` poskytuje podobné funkce jako atribut `StringLength`, ale neposkytuje ověřování na straně klienta.

Model databáze se teď změnil způsobem, který vyžaduje změnu ve schématu databáze. Pomocí migrace aktualizujete schéma bez ztráty dat, která jste mohli přidat do databáze pomocí uživatelského rozhraní aplikace.

Uložte změny a sestavte projekt. Pak otevřete příkazové okno ve složce projektu a zadejte následující příkazy:

```dotnetcli
dotnet ef migrations add MaxLengthOnNames
```

```dotnetcli
dotnet ef database update
```

Příkaz `migrations add` upozorní na to, že může dojít ke ztrátě dat, protože změna činí maximální délku pro dva sloupce.  Migrace vytvoří soubor s názvem *\<časové razítko > _MaxLengthOnNames. cs*. Tento soubor obsahuje kód v metodě `Up`, který bude aktualizovat databázi tak, aby odpovídala aktuálnímu datovému modelu. Příkaz `database update` spustil tento kód.

Časové razítko s předponou názvu souboru migrace se používá Entity Framework k seřazení migrace. Před spuštěním příkazu Update-Database můžete vytvořit více migrací a všechny migrace pak budou aplikovány v pořadí, ve kterém byly vytvořeny.

Spusťte aplikaci, vyberte kartu **Students** , klikněte na **vytvořit novou**a zkuste zadat název delší než 50 znaků. Aplikace by vám měla zabránit v tom. 

### <a name="the-column-attribute"></a>Atribut Column

Můžete také použít atributy pro řízení způsobu, jakým jsou třídy a vlastnosti namapovány na databázi. Předpokládejme, že jste použili název `FirstMidName` pro pole jméno a příjmení, protože pole může obsahovat také prostřední jméno. Ale chcete, aby se sloupec databáze jmenoval `FirstName`, protože uživatelé, kteří budou psát ad hoc dotazy na databázi, jsou zvyklí na tento název. Chcete-li toto mapování provést, můžete použít atribut `Column`.

Atribut `Column` určuje, že při vytvoření databáze bude sloupec `Student` tabulky, která je mapována na vlastnost `FirstMidName`, pojmenován `FirstName`. Jinými slovy, pokud váš kód odkazuje na `Student.FirstMidName`, data budou pocházet z nebo aktualizována ve sloupci `FirstName` tabulky `Student`. Pokud neurčíte názvy sloupců, budou mít stejný název jako název vlastnosti.

V souboru *student.cs* přidejte příkaz `using` pro `System.ComponentModel.DataAnnotations.Schema` a přidejte atribut název sloupce do vlastnosti `FirstMidName`, jak ukazuje následující zvýrazněný kód:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Column&highlight=4,14)]

Přidání atributu `Column` změní model zálohování `SchoolContext`, takže se neshoduje s databází.

Uložte změny a sestavte projekt. Pak otevřete příkazové okno ve složce projektu a zadáním následujících příkazů vytvořte další migraci:

```dotnetcli
dotnet ef migrations add ColumnFirstName
```

```dotnetcli
dotnet ef database update
```

V **Průzkumník objektů systému SQL Server**otevřete Návrhář tabulky student dvojitým kliknutím na tabulku **student** .

![Tabulka studentů v SSOX po migraci](complex-data-model/_static/ssox-after-migration.png)

Před použitím prvních dvou migrací byly sloupce názvu typu nvarchar (MAX). Teď jsou nvarchar (50) a název sloupce se změnil z FirstMidName na FirstName.

> [!Note]
> Pokud se pokusíte kompilovat před dokončením vytváření všech tříd entit v následujících oddílech, může dojít k chybám kompilátoru.

## <a name="changes-to-student-entity"></a>Změny entity studenta

![Entita studenta](complex-data-model/_static/student-entity.png)

V *modelu/student. cs*nahraďte kód, který jste přidali dříve, následujícím kódem. Změny jsou zvýrazněné.

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_BeforeInheritance&highlight=11,13,15,18,22,24-31)]

### <a name="the-required-attribute"></a>Požadovaný atribut

Atribut `Required` vytváří pole vlastnosti názvu Required. Atribut `Required` není potřebný pro typy, které neumožňují hodnotu null, jako jsou například typy hodnot (DateTime, int, Double, float atd.). Typy, které nemůžou mít hodnotu null, se automaticky považují za povinná pole.

Atribut `Required` musí být použit s `MinimumLength` pro vymáhání `MinimumLength`.

```csharp
[Display(Name = "Last Name")]
[Required]
[StringLength(50, MinimumLength=2)]
public string LastName { get; set; }
```

### <a name="the-display-attribute"></a>Atribut zobrazení

Atribut `Display` určuje, že titulek pro textová pole by měl být "křestní jméno", "příjmení", "celé jméno" a "datum zápisu" místo názvu vlastnosti v každé instanci (která nemá mezeru dělit slova).

### <a name="the-fullname-calculated-property"></a>Vypočítaná vlastnost FullName

`FullName` je vypočtená vlastnost, která vrací hodnotu, která je vytvořena zřetězením dvou dalších vlastností. Proto má pouze přistupující objekt get a v databázi nebude vygenerován žádný `FullName` sloupec.

## <a name="create-instructor-entity"></a>Vytvořit entitu instruktora

![Entita instruktora](complex-data-model/_static/instructor-entity.png)

Vytvořte *modely/Instructor. cs*a nahraďte kód šablony následujícím kódem:

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_BeforeInheritance)]

Všimněte si, že několik vlastností je stejné v entitách student a instruktor. V kurzu [Implementace dědičnosti](inheritance.md) níže v této sérii můžete tento kód Refaktorovat, aby se vyloučila redundance.

Můžete umístit více atributů na jeden řádek, takže můžete také zapsat `HireDate` atributy následujícím způsobem:

```csharp
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

### <a name="the-courseassignments-and-officeassignment-navigation-properties"></a>Navigační vlastnosti CourseAssignments a OfficeAssignment

Vlastnosti `CourseAssignments` a `OfficeAssignment` jsou navigační vlastnosti.

Instruktor může naučit libovolný počet kurzů, takže `CourseAssignments` je definován jako kolekce.

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

Pokud navigační vlastnost může obsahovat více entit, musí být její typ seznam, ve kterém je možné přidávat, odstraňovat a aktualizovat položky.  Můžete zadat `ICollection<T>` nebo typ, jako je například `List<T>` nebo `HashSet<T>`. Pokud zadáte `ICollection<T>`, EF vytvoří ve výchozím nastavení kolekci `HashSet<T>`.

Důvod, proč jsou tyto entity `CourseAssignment`, jsou vysvětleny níže v části o relacích n:n.

Obchodní pravidla společnosti Contoso vysokých firem, že instruktor může mít jenom jednu kancelář, takže vlastnost `OfficeAssignment` obsahuje jednu entitu OfficeAssignment (která může být null, pokud není přiřazená žádná sada Office).

```csharp
public OfficeAssignment OfficeAssignment { get; set; }
```

## <a name="create-officeassignment-entity"></a>Vytvořit entitu OfficeAssignment

![OfficeAssignment – entita](complex-data-model/_static/officeassignment-entity.png)

Vytvořte *modely/OfficeAssignment. cs* s následujícím kódem:

[!code-csharp[](intro/samples/cu/Models/OfficeAssignment.cs)]

### <a name="the-key-attribute"></a>Klíčový atribut

Mezi instruktorem a entitami OfficeAssignment existuje vztah 1:1 nebo jedna. Přiřazení kanceláře existuje pouze ve vztahu k instruktorovi, kterému je přiřazeno, a proto jeho primární klíč je také cizí klíč k entitě instruktor. Entity Framework ale nemůžou automaticky rozpoznat InstructorID jako primární klíč této entity, protože jeho název nedodržuje zásady vytváření názvů classnameID. Proto atribut `Key` slouží k identifikaci jako klíč:

```csharp
[Key]
public int InstructorID { get; set; }
```

Atribut `Key` můžete použít také v případě, že má entita svůj vlastní primární klíč, ale chcete vlastnost pojmenovat jinou než classnameID nebo ID.

Ve výchozím nastavení EF považuje klíč za generovaný nedatabází, protože sloupec je určen pro identifikaci vztahu.

### <a name="the-instructor-navigation-property"></a>Navigační vlastnost instruktora

Entita instruktora má `OfficeAssignment` navigační vlastnost s možnou hodnotou null (protože instruktor nemusí mít přiřazený Office) a entita OfficeAssignment má vlastnost navigace, která nemůže mít hodnotu null `Instructor` (protože přiřazení kanceláře nemůže existovat bez typu instruktor--`InstructorID` nemůže mít hodnotu null). Pokud má entita instruktora související entitu OfficeAssignment, bude mít Každá entita odkaz na jinou entitu v její navigační vlastnosti.

Můžete umístit atribut `[Required]` do navigační vlastnosti instruktora a určit tak, že se musí jednat o související instruktor, ale nemusíte to dělat, protože `InstructorID` cizí klíč (což je také klíč k této tabulce), který není null.

## <a name="modify-course-entity"></a>Upravit entitu kurzu

![Entita kurzu](complex-data-model/_static/course-entity.png)

V *modelu/Course. cs*nahraďte kód, který jste přidali dříve, následujícím kódem. Změny jsou zvýrazněné.

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Final&highlight=2,10,13,16,19,21,23)]

Entita kurzu má vlastnost cizího klíče `DepartmentID` která odkazuje na související entitu oddělení a má `Department` navigační vlastnost.

Entity Framework nevyžaduje, abyste do datového modelu přidali vlastnost cizího klíče, když máte vlastnost navigace pro související entitu.  EF v databázi automaticky vytvoří cizí klíče bez ohledu na to, kde jsou potřeba, a vytvoří pro ně [vlastnosti stínu](/ef/core/modeling/shadow-properties) . Ale při používání cizího klíče v datovém modelu může být aktualizace jednodušší a efektivnější. Když například načtete entitu kurzu, která se má upravit, entita oddělení má hodnotu null, pokud ji nenačtete, takže když aktualizujete entitu kurzu, budete muset nejdřív načíst entitu oddělení. Pokud je v datovém modelu zahrnutá vlastnost cizího klíče `DepartmentID`, nemusíte před aktualizací načítat entitu oddělení.

### <a name="the-databasegenerated-attribute"></a>Atribut DatabaseGenerated

Atribut `DatabaseGenerated` s parametrem `None` u vlastnosti `CourseID` určuje, že uživatel bude poskytovat hodnoty primárního klíče, a ne vygenerované databází.

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

Ve výchozím nastavení Entity Framework předpokládá, že databáze generuje hodnoty primárního klíče. To je to, co chcete ve většině scénářů. U entit kurzu ale použijete uživatelem zadané číslo kurzu, jako je například řada 1000, pro jedno oddělení, 2000 Series pro jiné oddělení a tak dále.

Atribut `DatabaseGenerated` lze také použít ke generování výchozích hodnot, jako v případě databázových sloupců použitých k záznamu data vytvoření nebo aktualizace řádku.  Další informace najdete v tématu [vygenerované vlastnosti](/ef/core/modeling/generated-properties).

### <a name="foreign-key-and-navigation-properties"></a>Vlastnosti cizích klíčů a navigace

Vlastnosti cizího klíče a navigační vlastnosti v entitě kurzu odrážejí následující vztahy:

Kurz se přiřadí jednomu oddělení, takže je `DepartmentID` cizí klíč a vlastnost navigace `Department` z výše uvedených důvodů.

```csharp
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

Kurz může mít zaregistrovaný libovolný počet studentů, takže navigační vlastnost `Enrollments` je kolekce:

```csharp
public ICollection<Enrollment> Enrollments { get; set; }
```

Kurz může být výukou více instruktory, takže `CourseAssignments` navigační vlastnost je kolekce (typ `CourseAssignment` je vysvětlen [později](#many-to-many-relationships)):

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

## <a name="create-department-entity"></a>Vytvořit entitu oddělení

![Entita oddělení](complex-data-model/_static/department-entity.png)

Vytvořte *modely/oddělení. cs* s následujícím kódem:

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Begin)]

### <a name="the-column-attribute"></a>Atribut Column

Dříve jste použili atribut `Column` ke změně mapování názvu sloupce. V kódu pro entitu oddělení se atribut `Column` používá ke změně mapování datových typů SQL tak, aby byl sloupec definovaný pomocí SQL Server typu peníze v databázi:

```csharp
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

Mapování sloupce není obecně vyžadováno, protože Entity Framework zvolí vhodný SQL Server datový typ založený na typu CLR, který definujete pro vlastnost. Typ CLR `decimal` namapuje na typ `decimal` SQL Server. Ale v tomto případě víte, že se ve sloupci budou držet peněžní částky, a datový typ Money je pro to vhodnější.

### <a name="foreign-key-and-navigation-properties"></a>Vlastnosti cizích klíčů a navigace

Vlastnosti cizího klíče a navigace odrážejí následující vztahy:

Oddělení může nebo nemusí mít správce a správce je vždy instruktorem. Proto je vlastnost `InstructorID` obsažena jako cizí klíč pro entitu instruktor a otazník je přidána za označení `int` typu k označení vlastnosti jako Nullable. Navigační vlastnost má název `Administrator`, ale obsahuje entitu instruktora:

```csharp
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

Oddělení může mít spoustu kurzů, takže máme navigační vlastnost kurzů:

```csharp
public ICollection<Course> Courses { get; set; }
```

> [!NOTE]
> Podle konvence Entity Framework umožňuje odstranit Kaskádové odstraňování cizích klíčů, které neumožňují hodnotu null, a pro relace m:n. Výsledkem může být cyklická kaskádová odstranění pravidel, která způsobí výjimku při pokusu o přidání migrace. Pokud jste například nedefinovali vlastnost oddělení. InstructorID jako Nullable, EF by nakonfigurovala pravidlo kaskádového odstranění, které odstraní oddělení, když odstraníte instruktora, což nevede k tomu, co chcete mít. Pokud vaše obchodní pravidla vyžadují, aby vlastnost `InstructorID` nebyla null, je nutné použít následující příkaz rozhraní API Fluent k zakázání kaskádových odstranění v relaci:
>
> ```csharp
> modelBuilder.Entity<Department>()
>    .HasOne(d => d.Administrator)
>    .WithMany()
>    .OnDelete(DeleteBehavior.Restrict)
> ```

## <a name="modify-enrollment-entity"></a>Upravit entitu registrace

![Entita registrace](complex-data-model/_static/enrollment-entity.png)

V *modelů/zápisu. cs*nahraďte kód, který jste přidali dříve, pomocí následujícího kódu:

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Final&highlight=1-2,16)]

### <a name="foreign-key-and-navigation-properties"></a>Vlastnosti cizích klíčů a navigace

Vlastnosti cizího klíče a vlastnosti navigace odrážejí následující vztahy:

Záznam zápisu je pro jeden kurz, takže existuje `CourseID` vlastnost cizího klíče a vlastnost navigace `Course`:

```csharp
public int CourseID { get; set; }
public Course Course { get; set; }
```

Záznam zápisu je určen pro jednoho studenta, takže existuje `StudentID` vlastnost cizího klíče a `Student` navigační vlastnost:

```csharp
public int StudentID { get; set; }
public Student Student { get; set; }
```

## <a name="many-to-many-relationships"></a>Relace m:n

Mezi entitami student a Course existuje vztah n:n a entita registrace funguje jako tabulka JOIN typu m:n *s datovou částí* v databázi. "S datovou částí" znamená, že tabulka zápisu obsahuje další data kromě cizích klíčů pro Spojené tabulky (v tomto případě primární klíč a vlastnost třídy).

Následující ilustrace znázorňuje, co tyto vztahy vypadají jako v diagramu entit. (Tento diagram byl vygenerován pomocí Entity Framework nástrojů Power Tools pro EF 6. x; vytvoření diagramu není součástí kurzu, stačí ho použít jako ilustraci.)

![Mezi studenty hodně a mnoha](complex-data-model/_static/student-course.png)

Každá čára relace má 1 na jednom konci a hvězdičku (*) na druhé straně, která indikuje relaci 1: n.

Pokud tabulka zápisu neobsahuje informace o třídě, musí obsahovat pouze dva cizí klíče CourseID a StudentID. V takovém případě by to byla tabulka JOIN typu m:n (celá řada) bez datové části (nebo čistá spojovací tabulka) v databázi. Entity instruktor a Course mají tento druh relace m:n a vaším dalším krokem je vytvoření třídy entity, která bude fungovat jako spojovací tabulka bez datové části.

(EF 6. x podporuje implicitní spojení tabulek pro relace m:n, ale EF Core ne. Další informace najdete [v diskuzi v úložišti GitHub EF Core](https://github.com/aspnet/EntityFramework/issues/1368).)

## <a name="the-courseassignment-entity"></a>Entita CourseAssignment

![CourseAssignment – entita](complex-data-model/_static/courseassignment-entity.png)

Vytvořte *modely/CourseAssignment. cs* s následujícím kódem:

[!code-csharp[](intro/samples/cu/Models/CourseAssignment.cs)]

### <a name="join-entity-names"></a>Spojování názvů entit

V databázi je vyžadována tabulka JOIN pro relaci n:n typu n:1 a musí být reprezentovaná sadou entit... Je běžné pojmenovat entitu JOIN `EntityName1EntityName2`, což by v tomto případě bylo `CourseInstructor`. Doporučujeme však zvolit název, který popisuje vztah. Modely dat začínají jednoduchým a roste a často se nepoužívají spojení bez datové části, které později načítá datovou část. Pokud začnete s popisným názvem entity, nebudete muset později změnit název. V ideálním případě by entita JOIN měla vlastní přirozený název (případně jeden Word) v obchodní doméně. Například knihy a zákazníci mohou být propojeni pomocí hodnocení. Pro tento vztah je `CourseAssignment` lepší volbou než `CourseInstructor`.

### <a name="composite-key"></a>Složený klíč

Vzhledem k tomu, že cizí klíče neumožňují hodnotu null a společně jednoznačně identifikují každý řádek tabulky, není nutné používat samostatný primární klíč. Vlastnosti *InstructorID* a *CourseID* by měly fungovat jako složený primární klíč. Jediným způsobem, jak identifikovat složené primární klíče k EF, je použití *rozhraní Fluent API* (nemůže být provedeno pomocí atributů). V další části se dozvíte, jak nakonfigurovat složený primární klíč.

Složený klíč zajišťuje, že i když můžete mít více řádků pro jeden kurz a více řádků pro jednoho instruktora, nemůžete mít pro stejný instruktor a kurz více řádků. Entita `Enrollment` JOIN definuje svůj vlastní primární klíč, takže je možné duplicity tohoto řazení provést. Aby tyto duplicity nedocházelo, mohli byste do polí cizího klíče přidat jedinečný index nebo `Enrollment` nakonfigurovat primární složený klíč podobný `CourseAssignment`. Další informace najdete v tématu [indexy](/ef/core/modeling/indexes).

## <a name="update-the-database-context"></a>Aktualizace kontextu databáze

Do souboru *data/SchoolContext. cs* přidejte následující zvýrazněný kód:

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_BeforeInheritance&highlight=15-18,25-31)]

Tento kód přidá nové entity a nakonfiguruje složený primární klíč entity CourseAssignment.

## <a name="about-a-fluent-api-alternative"></a>O alternativní rozhraní API Fluent

Kód v metodě `OnModelCreating` třídy `DbContext` používá *rozhraní Fluent API* ke konfiguraci chování EF. Rozhraní API se nazývá "Fluent", protože se často používá k zřetězení řady volání metody do jednoho příkazu, jako v tomto příkladu v [dokumentaci EF Core](/ef/core/modeling/#use-fluent-api-to-configure-a-model):

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

V tomto kurzu používáte rozhraní Fluent API jenom pro mapování databáze, které nemůžete s atributy dělat. Rozhraní Fluent API ale můžete použít i k určení většiny pravidel formátování, ověřování a mapování, která můžete provádět pomocí atributů. Některé atributy, jako je `MinimumLength`, se nedají použít s rozhraním API Fluent. Jak již bylo zmíněno dříve, `MinimumLength` nemění schéma, používá pouze pravidlo ověřování na straně klienta a serveru.

Někteří vývojáři dávají přednost použití rozhraní Fluent API, aby mohli zachovat třídy entit "vyčistit". V případě potřeby můžete kombinovat atributy a rozhraní API Fluent a existuje několik úprav, které je možné provést jenom pomocí rozhraní Fluent API, ale obecně doporučujeme zvolit jednu z těchto dvou přístupů a použít ji konzistentně co nejvíce. Pokud použijete obojí, mějte na paměti, že pokud dojde ke konfliktu, rozhraní Fluent API Přepisuje atributy.

Další informace o atributech vs. Fluent API najdete v tématu [metody konfigurace](/ef/core/modeling/).

## <a name="entity-diagram-showing-relationships"></a>Diagram entit znázorňující vztahy

Následující ilustrace znázorňuje diagram, který nástroje Entity Framework Power Tools vytvoří pro dokončený školní model.

![Diagram entit](complex-data-model/_static/diagram.png)

Kromě řádků relace 1:1 (1 až \*) se tady můžete podívat na řádek relace 1:1 (1 – 0.1) mezi entitami instruktor a OfficeAssignment a řádkem relace 0.. 1 – n mezi entitami instruktora a oddělení (0.. 1 až *).

## <a name="seed-database-with-test-data"></a>Počáteční databáze s testovacími daty

Nahraďte kód v souboru *data/DbInitializer. cs* následujícím kódem, aby bylo možné poskytnout počáteční data pro nově vytvořené entity.

[!code-csharp[](intro/samples/cu/Data/DbInitializer.cs?name=snippet_Final)]

Jak jste viděli v prvním kurzu, většina tohoto kódu jednoduše vytvoří nové objekty entity a načte vzorová data do vlastností podle potřeby pro testování. Všimněte si, jak jsou zpracovávány relace m:n: kód vytvoří relace vytvořením entit v `Enrollments` a `CourseAssignment` spojí sady entit.

## <a name="add-a-migration"></a>Přidání migrace

Uložte změny a sestavte projekt. Pak otevřete příkazové okno ve složce projektu a zadejte příkaz `migrations add` (zatím neprovádějte příkaz Update-Database):

```dotnetcli
dotnet ef migrations add ComplexDataModel
```

Zobrazí se upozornění na možnou ztrátu dat.

```text
An operation was scaffolded that may result in the loss of data. Please review the migration for accuracy.
Done. To undo this action, use 'ef migrations remove'
```

Pokud jste se v tomto okamžiku pokusili spustit příkaz `database update`, měli byste obdržet následující chybu:

> Příkaz ALTER TABLE je v konfliktu s omezením CIZÍho klíče FK_dbo. Course_dbo. Department_DepartmentID ". Ke konfliktu došlo v databázi "ContosoUniversity", tabulce "dbo". Oddělení ", sloupec" DepartmentID ".

Při provádění migrace s existujícími daty je někdy potřeba vložit do databáze zástupná data, aby bylo možné splnit omezení cizího klíče. Generovaný kód v metodě `Up` přidá do tabulky Course DepartmentID cizí klíč, který nemůže mít hodnotu null. Pokud se v tabulce kurzu již nacházejí řádky, když je kód spuštěn, operace `AddColumn` nebude úspěšná, protože SQL Server neví, jakou hodnotu umístit do sloupce, který nemůže mít hodnotu null. Pro účely tohoto kurzu spustíte migraci na nové databázi, ale v produkční aplikaci, kterou byste museli udělat, aby migrace zpracovala existující data, takže následující pokyny ukazují příklad toho, jak to udělat.

Pokud chcete, aby tato migrace fungovala se stávajícími daty, je nutné změnit kód tak, aby nový sloupec poskytoval výchozí hodnotu, a vytvořit oddělení zástupných procedur s názvem "Temp", které bude fungovat jako výchozí oddělení. V důsledku toho budou existující řádky kurzu po spuštění metody `Up` v relaci k "dočasnému" oddělení.

* Otevřete soubor *{timestamp} _ComplexDataModel. cs* .

* Odkomentujte řádek kódu, který přidá sloupec DepartmentID do tabulky Course.

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CommentOut&highlight=9-13)]

* Po kódu, který vytvoří tabulku oddělení, přidejte následující zvýrazněný kód:

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CreateDefaultValue&highlight=22-32)]

V produkční aplikaci byste mohli napsat kód nebo skripty pro přidání řádků oddělení a související řádky kurzu pro nové řádky oddělení. Nebudete už potřebovat "dočasné" oddělení nebo výchozí hodnotu ve sloupci Course. DepartmentID.

Uložte změny a sestavte projekt.

## <a name="change-the-connection-string"></a>Změna připojovacího řetězce

Nyní máte nový kód ve třídě `DbInitializer`, který přidá počáteční data pro nové entity do prázdné databáze. Chcete-li, aby EF vytvořil novou prázdnou databázi, změňte název databáze v připojovacím řetězci v souboru *appSettings. JSON* na ContosoUniversity3 nebo na jiný název, který jste nepoužili na počítači, který používáte.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
```

Uložte změnu do souboru *appSettings. JSON*.

> [!NOTE]
> Jako alternativu ke změně názvu databáze můžete databázi odstranit. Použijte **Průzkumník objektů systému SQL Server** (SSOX) nebo příkaz `database drop` CLI:
>
> ```dotnetcli
> dotnet ef database drop
> ```

## <a name="update-the-database"></a>Aktualizace databáze

Poté, co jste změnili název databáze nebo odstranili databázi, spusťte příkaz `database update` v příkazovém okně a spusťte tak migrace.

```dotnetcli
dotnet ef database update
```

Spusťte aplikaci, aby byla metoda `DbInitializer.Initialize` spuštěna a naplnila novou databázi.

Otevřete databázi v SSOX jako dříve a rozbalte uzel **tabulky** , abyste viděli, že se vytvořily všechny tabulky. (Pokud máte stále SSOX otevřený ze staršího času, klikněte na tlačítko **aktualizovat** .)

![Tabulky v SSOX](complex-data-model/_static/ssox-tables.png)

Spusťte aplikaci, aby aktivovala inicializační kód, který vysemena databáze.

Klikněte pravým tlačítkem na tabulku **CourseAssignment** a vyberte **Zobrazit data** a ověřte, zda obsahuje data.

![Data CourseAssignment v SSOX](complex-data-model/_static/ssox-ci-data.png)

## <a name="get-the-code"></a>Získat kód

[Stažení nebo zobrazení dokončené aplikace.](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Další kroky

V tomto kurzu se naučíte:

> [!div class="checklist"]
> * Přizpůsobení datového modelu
> * Provedli jsme změny entity studenta
> * Vytvořená entita instruktora
> * Vytvořila se entita OfficeAssignment
> * Upravená entita kurzu
> * Entita vytvořeného oddělení
> * Upravená entita registrace
> * Aktualizace kontextu databáze
> * Dosazení databáze s testovacími daty
> * Přidání migrace
> * Změna připojovacího řetězce
> * Aktualizace databáze

V dalším kurzu se dozvíte víc o tom, jak přistupovat k souvisejícím datům.

> [!div class="nextstepaction"]
> [Další: přístup k datům v relaci](read-related-data.md)
