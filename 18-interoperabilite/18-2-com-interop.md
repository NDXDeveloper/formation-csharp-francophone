# 18.2. COM Interop

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

Bienvenue dans ce chapitre consacré à COM Interop, une technologie qui permet à votre code C# d'interagir avec les composants COM (Component Object Model), une architecture logicielle développée par Microsoft dans les années 1990.

Vous vous demandez peut-être : "Pourquoi devrais-je apprendre COM Interop alors que C# et .NET sont modernes et complets ?" Voici quelques raisons importantes :

- De nombreuses applications Windows et bibliothèques existantes sont construites avec COM
- Les applications Microsoft Office exposent leur fonctionnalité via des interfaces COM
- Certains composants système Windows ne sont accessibles que via COM
- Vous pourriez avoir besoin d'intégrer votre nouveau code C# avec des systèmes hérités

Dans ce chapitre, nous allons explorer COM Interop de manière progressive, en commençant par les concepts de base jusqu'aux techniques plus avancées, le tout avec des exemples pratiques et accessibles.

## 18.2.1. Utilisation de composants COM

### Qu'est-ce que COM ?

Le Component Object Model (COM) est une spécification d'interface binaire qui permet la communication entre logiciels. Avant .NET, COM était la principale méthode pour créer des composants réutilisables sous Windows. Les composants COM :

- Sont indépendants du langage de programmation
- Utilisent des interfaces pour exposer leurs fonctionnalités
- S'identifient par des GUIDs (Globally Unique Identifiers)
- Fonctionnent selon un modèle de référence

### Référencer un composant COM dans votre projet

Pour utiliser un composant COM dans un projet C#, vous devez d'abord le référencer :

1. Dans Visual Studio, cliquez-droit sur votre projet dans l'Explorateur de solutions
2. Sélectionnez "Ajouter" > "Référence..."
3. Dans la boîte de dialogue, allez à l'onglet "COM"
4. Parcourez la liste des composants COM enregistrés sur votre système
5. Sélectionnez le composant souhaité et cliquez sur "OK"

![Ajouter une référence COM](https://example.com/images/add-com-reference.png)

Visual Studio génère automatiquement un assembly d'interopérabilité (wrapper) pour le composant COM sélectionné. Ce wrapper fait le pont entre votre code C# et le composant COM.

### Exemple : Utilisation de Microsoft Excel via COM

Voici un exemple simple d'utilisation de COM pour contrôler Microsoft Excel :

```csharp
using System;
using Microsoft.Office.Interop.Excel;
using System.Runtime.InteropServices;

class Program
{
    static void Main()
    {
        // Déclaration des variables COM
        Application excelApp = null;
        Workbook workbook = null;
        Worksheet worksheet = null;

        try
        {
            // Création d'une nouvelle instance d'Excel
            excelApp = new Application();
            excelApp.Visible = true;  // Rendre Excel visible

            // Ajouter un nouveau classeur
            workbook = excelApp.Workbooks.Add();

            // Obtenir la première feuille de calcul
            worksheet = workbook.Worksheets[1] as Worksheet;

            // Écrire quelques données
            worksheet.Cells[1, 1] = "Nom";
            worksheet.Cells[1, 2] = "Âge";
            worksheet.Cells[2, 1] = "Alice";
            worksheet.Cells[2, 2] = 30;
            worksheet.Cells[3, 1] = "Bob";
            worksheet.Cells[3, 2] = 25;

            // Mettre en forme les en-têtes en gras
            Range headers = worksheet.Range["A1", "B1"];
            headers.Font.Bold = true;

            // Ajuster la largeur des colonnes
            worksheet.Columns["A:B"].AutoFit();

            // Enregistrer le classeur
            workbook.SaveAs("C:\\temp\\exemple.xlsx");

            Console.WriteLine("Fichier Excel créé avec succès !");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur : {ex.Message}");
        }
        finally
        {
            // Nettoyage des ressources COM (TRÈS IMPORTANT)
            if (worksheet != null) Marshal.ReleaseComObject(worksheet);
            if (workbook != null)
            {
                workbook.Close(false);
                Marshal.ReleaseComObject(workbook);
            }
            if (excelApp != null)
            {
                excelApp.Quit();
                Marshal.ReleaseComObject(excelApp);
            }

            // Forcer le garbage collector
            GC.Collect();
            GC.WaitForPendingFinalizers();
        }
    }
}
```

Points importants à noter dans cet exemple :
- Nous utilisons le namespace `Microsoft.Office.Interop.Excel` qui contient les wrappers .NET pour les objets COM d'Excel
- Nous devons libérer explicitement les objets COM avec `Marshal.ReleaseComObject()` pour éviter les fuites de mémoire
- Un bloc `finally` garantit que les ressources sont libérées même en cas d'erreur

### L'importance de libérer les ressources COM

Les composants COM utilisent un système de comptage de références qui ne s'intègre pas parfaitement avec le garbage collector .NET. Si vous ne libérez pas explicitement vos références COM, vous risquez :

- Des fuites de mémoire
- Des processus qui restent en arrière-plan (comme Excel.exe qui ne se ferme pas)
- Une dégradation des performances

Pattern recommandé pour libérer proprement les ressources COM :

```csharp
// Déclaration
dynamic comObject = null;

try
{
    // Utilisation du composant COM
    comObject = new ComComponent();
    comObject.DoSomething();
}
finally
{
    // Libération des ressources
    if (comObject != null)
    {
        Marshal.ReleaseComObject(comObject);
        comObject = null;
    }

    // Forcer le garbage collector
    GC.Collect();
    GC.WaitForPendingFinalizers();
}
```

## 18.2.2. Création de wrappers

Parfois, interagir directement avec les objets COM n'est pas optimal. La création de wrappers (classes d'encapsulation) peut améliorer l'usabilité et la robustesse de votre code.

### Pourquoi créer des wrappers personnalisés ?

- Simplifier l'interface exposée à votre application
- Masquer la complexité des opérations COM
- Gérer automatiquement la libération des ressources
- Ajouter des fonctionnalités spécifiques à votre application
- Améliorer la documentation et l'intellisense

### Exemple : Wrapper pour Excel

Voici un exemple de wrapper simplifié pour Excel :

```csharp
using System;
using System.Runtime.InteropServices;
using Excel = Microsoft.Office.Interop.Excel;

public class ExcelWrapper : IDisposable
{
    private Excel.Application _excelApp;
    private Excel.Workbook _workbook;
    private Excel.Worksheet _activeSheet;
    private bool _disposed = false;

    public ExcelWrapper(bool visible = true)
    {
        _excelApp = new Excel.Application();
        _excelApp.Visible = visible;
        _workbook = _excelApp.Workbooks.Add();
        _activeSheet = _workbook.Worksheets[1];
    }

    // Méthode pour définir une valeur dans une cellule
    public void SetCellValue(int row, int column, object value)
    {
        _activeSheet.Cells[row, column] = value;
    }

    // Méthode pour récupérer une valeur d'une cellule
    public object GetCellValue(int row, int column)
    {
        return _activeSheet.Cells[row, column].Value2;
    }

    // Méthode pour mettre en forme une plage de cellules
    public void FormatRange(string range, bool bold = false, string backgroundColor = null)
    {
        Excel.Range rangeObj = _activeSheet.Range[range];

        if (bold)
            rangeObj.Font.Bold = true;

        if (backgroundColor != null)
        {
            // Conversion de la couleur en valeur Excel
            Excel.XlRgbColor color;
            if (Enum.TryParse(backgroundColor, out color))
                rangeObj.Interior.Color = color;
        }
    }

    // Méthode pour enregistrer le classeur
    public void SaveAs(string filePath)
    {
        _workbook.SaveAs(filePath);
    }

    // Implémentation de IDisposable pour gérer proprement les ressources COM
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Libérer les ressources managées
            }

            // Libérer les ressources COM non managées
            if (_activeSheet != null)
            {
                Marshal.ReleaseComObject(_activeSheet);
                _activeSheet = null;
            }

            if (_workbook != null)
            {
                _workbook.Close(false);
                Marshal.ReleaseComObject(_workbook);
                _workbook = null;
            }

            if (_excelApp != null)
            {
                _excelApp.Quit();
                Marshal.ReleaseComObject(_excelApp);
                _excelApp = null;
            }

            _disposed = true;
        }
    }

    ~ExcelWrapper()
    {
        Dispose(false);
    }
}
```

Utilisation du wrapper :

```csharp
// Utilisation simplifiée avec using pour garantir la libération des ressources
using (var excel = new ExcelWrapper(true))
{
    // Définir des en-têtes
    excel.SetCellValue(1, 1, "Produit");
    excel.SetCellValue(1, 2, "Prix");

    // Ajouter des données
    excel.SetCellValue(2, 1, "Ordinateur");
    excel.SetCellValue(2, 2, 1200);

    // Mettre en forme les en-têtes
    excel.FormatRange("A1:B1", true);

    // Enregistrer
    excel.SaveAs(@"C:\temp\catalogue.xlsx");
}
// À la fin du bloc using, Dispose() est automatiquement appelé
```

Avantages de cette approche :
- Interface plus simple et intuitive
- Gestion automatique des ressources grâce à `IDisposable` et au bloc `using`
- Code plus lisible et maintenable
- Meilleure encapsulation des fonctionnalités

## 18.2.3. Automation

L'automation est une fonctionnalité clé de COM qui permet à une application de contrôler programmatiquement une autre application. C'est ce qui nous permet, par exemple, de piloter Excel depuis C#.

### Principes de l'automation

- L'application qui expose ses fonctionnalités est appelée "serveur d'automation"
- L'application qui utilise ces fonctionnalités est appelée "client d'automation"
- Le serveur expose sa fonctionnalité via une "bibliothèque de types" (type library)
- Le client utilise cette bibliothèque pour interagir avec le serveur

### Applications courantes de l'automation

1. **Microsoft Office** (Word, Excel, Outlook, PowerPoint)
2. **Internet Explorer**
3. **Applications d'entreprise hérités**
4. **Outils de conception** (AutoCAD, SolidWorks)

### Exemple : Automation avec Microsoft Word

```csharp
using System;
using System.Runtime.InteropServices;
using Word = Microsoft.Office.Interop.Word;

class Program
{
    static void Main()
    {
        // Déclaration des variables COM
        Word.Application wordApp = null;
        Word.Document document = null;

        try
        {
            // Créer une nouvelle instance de Word
            wordApp = new Word.Application();
            wordApp.Visible = true;

            // Créer un nouveau document
            document = wordApp.Documents.Add();

            // Ajouter du texte
            document.Content.Text = "Bonjour, ceci est un document créé par automation COM !";

            // Mettre en forme le texte
            document.Content.Font.Size = 14;
            document.Content.Font.Bold = 1; // 1 = vrai en COM

            // Ajouter un titre
            Word.Paragraph titre = document.Content.Paragraphs.Add();
            titre.Range.Text = "Exemple d'automation avec Word";
            titre.Range.Font.Size = 20;
            titre.Format.Alignment = Word.WdParagraphAlignment.wdAlignParagraphCenter;

            // Insérer un tableau
            Word.Table tableau = document.Tables.Add(
                document.Paragraphs[document.Paragraphs.Count].Range,
                3, // Lignes
                2  // Colonnes
            );

            // Remplir le tableau
            tableau.Cell(1, 1).Range.Text = "Produit";
            tableau.Cell(1, 2).Range.Text = "Prix";
            tableau.Cell(2, 1).Range.Text = "Ordinateur";
            tableau.Cell(2, 2).Range.Text = "1200 €";
            tableau.Cell(3, 1).Range.Text = "Téléphone";
            tableau.Cell(3, 2).Range.Text = "800 €";

            // Mettre en forme la première ligne
            tableau.Rows[1].Range.Font.Bold = 1;

            // Enregistrer le document
            document.SaveAs2(@"C:\temp\exemple_word.docx");

            Console.WriteLine("Document Word créé avec succès !");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur : {ex.Message}");
        }
        finally
        {
            // Nettoyage des ressources COM
            if (document != null)
            {
                document.Close(false);
                Marshal.ReleaseComObject(document);
            }

            if (wordApp != null)
            {
                wordApp.Quit();
                Marshal.ReleaseComObject(wordApp);
            }

            GC.Collect();
            GC.WaitForPendingFinalizers();
        }
    }
}
```

### Découvrir les capacités d'automation d'une application

Pour explorer ce qu'une application COM expose via automation :

1. **Bibliothèque d'objets** : Dans Visual Studio, allez dans "Affichage" > "Navigateur d'objets"
2. **Documentation** : Consultez la documentation du développeur (comme la référence Microsoft Office)
3. **Explorateur d'objets** : Utilisez des outils comme "OleView" ou "Object Browser" pour examiner les bibliothèques de types

## 18.2.4. RCW et CCW

Pour permettre à .NET et COM de communiquer, le runtime .NET utilise deux types de wrappers :

- **RCW (Runtime Callable Wrapper)** : Permet à du code .NET d'appeler des objets COM
- **CCW (COM Callable Wrapper)** : Permet à du code COM d'appeler des objets .NET

### Runtime Callable Wrapper (RCW)

Le RCW est un proxy généré par le runtime .NET qui :

- Encapsule un objet COM
- Traduit les types COM en types .NET
- Gère le cycle de vie de l'objet COM sous-jacent
- Implémente les interfaces .NET qui correspondent aux interfaces COM

Lorsque vous ajoutez une référence COM à votre projet, Visual Studio génère automatiquement les RCWs nécessaires.

#### Fonctionnement interne du RCW

1. Vous instanciez une classe COM via son interface .NET
2. Le runtime .NET crée un RCW qui encapsule l'objet COM
3. Vos appels de méthodes .NET sont traduits en appels COM par le RCW
4. Les résultats COM sont convertis en types .NET et retournés à votre code

#### Gestion manuelle des RCWs

Dans certains cas, vous pourriez avoir besoin de travailler directement avec les RCWs :

```csharp
using System;
using System.Runtime.InteropServices;

class Program
{
    static void Main()
    {
        // Créer un objet COM
        dynamic comObject = Activator.CreateInstance(Type.GetTypeFromProgID("Excel.Application"));

        try
        {
            // Utiliser l'objet COM
            comObject.Visible = true;

            // Obtenir le RCW qui encapsule l'objet COM
            object rcw = Marshal.GetRCWForObject(comObject);
            Console.WriteLine($"Type du RCW : {rcw.GetType().FullName}");

            // Vérifier si deux variables référencent le même objet COM
            dynamic comObject2 = comObject;
            bool sameComObject = Marshal.AreComObjectsEqual(comObject, comObject2);
            Console.WriteLine($"Même objet COM ? {sameComObject}");

            // Obtenir le compte de références COM
            int refCount = Marshal.ReleaseComObject(comObject);
            Console.WriteLine($"Compte de références restant : {refCount}");
        }
        finally
        {
            // S'assurer que toutes les références sont libérées
            while (Marshal.ReleaseComObject(comObject) > 0) { }
        }
    }
}
```

### COM Callable Wrapper (CCW)

Le CCW est l'inverse du RCW - il permet à du code COM d'utiliser des objets .NET :

- Expose un objet .NET comme un objet COM
- Implémente les interfaces COM nécessaires
- Traduit les types .NET en types COM
- Gère le comptage de références pour COM

#### Exposer une classe .NET à COM

Pour rendre une classe .NET utilisable par COM :

```csharp
using System;
using System.Runtime.InteropServices;

// Définir un GUID pour la classe et l'interface
[Guid("12345678-1234-1234-1234-123456789012")]
[ComVisible(true)]
public interface IMonCalculateur
{
    int Additionner(int a, int b);
    double Multiplier(double a, double b);
}

// Implémenter l'interface avec la classe
[Guid("87654321-4321-4321-4321-210987654321")]
[ComVisible(true)]
[ClassInterface(ClassInterfaceType.None)]
public class MonCalculateur : IMonCalculateur
{
    public int Additionner(int a, int b)
    {
        return a + b;
    }

    public double Multiplier(double a, double b)
    {
        return a * b;
    }
}
```

Les attributs importants :
- `[ComVisible(true)]` : Rend la classe/interface visible pour COM
- `[Guid("...")]` : Assigne un identifiant unique global à la classe/interface
- `[ClassInterface(ClassInterfaceType.None)]` : Indique que la classe n'expose pas automatiquement ses membres via COM, mais uniquement via les interfaces explicitement implémentées

#### Enregistrer un assembly pour COM

Pour qu'un composant COM puisse utiliser votre classe .NET, vous devez enregistrer l'assembly :

1. Configurez votre projet pour qu'il soit enregistrable pour COM :
   - Dans les propriétés du projet, allez à l'onglet "Build"
   - Cochez "Register for COM interop"

2. Ou utilisez l'outil RegAsm.exe en ligne de commande :
   ```
   RegAsm.exe MonAssembly.dll /tlb:MonAssembly.tlb /codebase
   ```

## 18.2.5. Early vs Late binding

Il existe deux façons d'interagir avec les objets COM : early binding (liaison précoce) et late binding (liaison tardive).

### Early Binding (Liaison précoce)

Avec l'early binding :
- Les informations de type sont connues à la compilation
- Le compilateur vérifie les appels à l'objet COM
- Les performances sont meilleures
- L'IntelliSense est disponible

Pour utiliser l'early binding, vous avez besoin :
- D'une référence à la bibliothèque de types COM (via l'ajout de référence)
- D'un using pour le namespace généré

Exemple d'early binding avec Excel :

```csharp
using System;
using Excel = Microsoft.Office.Interop.Excel;

class Program
{
    static void Main()
    {
        Excel.Application excelApp = null;
        Excel.Workbook workbook = null;

        try
        {
            // Early binding - le type est connu à la compilation
            excelApp = new Excel.Application();
            excelApp.Visible = true;

            workbook = excelApp.Workbooks.Add();
            Excel.Worksheet sheet = workbook.Worksheets[1];

            // Le compilateur peut vérifier ces propriétés et méthodes
            sheet.Cells[1, 1].Value = "Bonjour";
            sheet.Cells[1, 1].Font.Bold = true;
        }
        finally
        {
            // Nettoyage
            if (workbook != null)
                Marshal.ReleaseComObject(workbook);

            if (excelApp != null)
            {
                excelApp.Quit();
                Marshal.ReleaseComObject(excelApp);
            }
        }
    }
}
```

### Late Binding (Liaison tardive)

Avec le late binding :
- Les informations de type sont découvertes à l'exécution
- Pas de vérification par le compilateur
- Performances légèrement moins bonnes
- Pas d'IntelliSense
- Plus flexible (peut s'adapter à différentes versions d'une application)

Pour utiliser le late binding, vous pouvez :
- Utiliser `dynamic` (C# 4.0+)
- Utiliser `Type.GetTypeFromProgID()` et `Activator.CreateInstance()`
- Utiliser `System.Reflection`

Exemple de late binding avec Excel :

```csharp
using System;
using System.Runtime.InteropServices;

class Program
{
    static void Main()
    {
        dynamic excelApp = null;
        dynamic workbook = null;

        try
        {
            // Late binding avec dynamic
            excelApp = Activator.CreateInstance(Type.GetTypeFromProgID("Excel.Application"));
            excelApp.Visible = true;

            workbook = excelApp.Workbooks.Add();
            dynamic sheet = workbook.Worksheets[1];

            // Le compilateur ne vérifie pas ces propriétés et méthodes
            sheet.Cells[1, 1].Value = "Bonjour";
            sheet.Cells[1, 1].Font.Bold = true;
        }
        catch (Exception ex)
        {
            // Les erreurs sont détectées seulement à l'exécution
            Console.WriteLine($"Erreur : {ex.Message}");
        }
        finally
        {
            // Nettoyage
            if (workbook != null)
                Marshal.ReleaseComObject(workbook);

            if (excelApp != null)
            {
                excelApp.Quit();
                Marshal.ReleaseComObject(excelApp);
            }
        }
    }
}
```

### Méthode alternative de late binding (avant C# 4.0)

Avant l'introduction du mot-clé `dynamic` en C# 4.0, le late binding était généralement réalisé via reflection :

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        object excelApp = null;
        object workbooks = null;
        object workbook = null;

        try
        {
            // Créer l'objet Excel.Application via reflection
            Type excelType = Type.GetTypeFromProgID("Excel.Application");
            excelApp = Activator.CreateInstance(excelType);

            // Définir Visible = true
            excelType.InvokeMember("Visible",
                BindingFlags.SetProperty,
                null,
                excelApp,
                new object[] { true });

            // Obtenir la collection Workbooks
            workbooks = excelType.InvokeMember("Workbooks",
                BindingFlags.GetProperty,
                null,
                excelApp,
                null);

            // Appeler Add() pour créer un nouveau classeur
            Type workbooksType = workbooks.GetType();
            workbook = workbooksType.InvokeMember("Add",
                BindingFlags.InvokeMethod,
                null,
                workbooks,
                null);

            Console.WriteLine("Nouveau classeur Excel créé !");
        }
        finally
        {
            // Nettoyage (également par reflection, pour simplifier)
            if (excelApp != null)
            {
                Type excelType = excelApp.GetType();
                excelType.InvokeMember("Quit",
                    BindingFlags.InvokeMethod,
                    null,
                    excelApp,
                    null);

                Marshal.ReleaseComObject(excelApp);
            }

            if (workbook != null)
                Marshal.ReleaseComObject(workbook);

            if (workbooks != null)
                Marshal.ReleaseComObject(workbooks);
        }
    }
}
```

### Choisir entre Early et Late Binding

| Critère | Early Binding | Late Binding |
|---------|--------------|-------------|
| Performance | Meilleure | Légèrement moins bonne |
| Vérification à la compilation | Oui | Non |
| IntelliSense | Disponible | Non disponible |
| Flexibilité | Moins flexible | Plus flexible |
| Compatibilité avec différentes versions | Peut nécessiter une recompilation | S'adapte automatiquement |
| Complexité du code | Plus simple | Plus complexe |

Recommandations :
- Utilisez l'early binding quand vous savez exactement avec quelle version du composant COM vous travaillez
- Utilisez le late binding quand vous avez besoin de fonctionner avec différentes versions du composant COM
- Utilisez `dynamic` pour le late binding si vous utilisez C# 4.0 ou supérieur

## Conclusion

COM Interop est un pont essentiel entre le monde moderne de .NET et les technologies COM plus anciennes mais toujours largement utilisées. Bien que travailler avec COM puisse parfois sembler complexe, les outils fournis par .NET facilitent grandement cette intégration.

Points clés à retenir :
- COM Interop permet à votre code C# de communiquer avec des composants COM
- Les RCWs permettent au code .NET d'appeler des objets COM
- Les CCWs permettent au code COM d'appeler des objets .NET
- L'early binding offre de meilleures performances et la vérification à la compilation
- Le late binding offre plus de flexibilité pour s'adapter à différentes versions
- La libération des ressources COM est cruciale pour éviter les fuites de mémoire

## Exercices pratiques

1. Créez une application qui génère un document Word avec un tableau et un graphique à partir de données stockées dans une base de données
2. Développez un wrapper pour l'application Outlook qui simplifie l'envoi d'emails avec pièces jointes
3. Exposez une classe .NET comme composant COM et utilisez-la depuis un script VBA dans Excel
4. Comparez les performances de l'early binding et du late binding dans une opération répétitive avec Excel

## Ressources additionnelles

- [Documentation Microsoft sur COM Interop](https://docs.microsoft.com/fr-fr/dotnet/framework/interop/com-interop)
- [Guide de l'API Office](https://docs.microsoft.com/fr-fr/office/client-developer/office-client-development)
- [Livre "COM and .NET Interoperability" par Andrew Troelsen](https://www.amazon.com/COM-NET-Interoperability-Andrew-Troelsen/dp/1590590503)

⏭️
