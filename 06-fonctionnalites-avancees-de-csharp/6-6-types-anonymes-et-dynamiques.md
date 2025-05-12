# 6.6. Types anonymes et dynamiques

🔝 Retour au [Sommaire](/SOMMAIRE.md)

C# offre plusieurs moyens flexibles de manipuler des données, notamment à travers les types anonymes et le type `dynamic`. Ces fonctionnalités permettent aux développeurs de travailler avec des objets dont la structure n'est pas définie statiquement, ce qui peut s'avérer utile dans certains contextes. Cette section explore ces concepts en détail, leur utilisation et leurs limites.

## 6.6.1. Objets anonymes

Les objets anonymes permettent de créer instantanément des classes légères sans avoir à les définir explicitement. Ils sont particulièrement utiles pour regrouper temporairement des propriétés.

### Création d'objets anonymes

Un objet anonyme est créé en utilisant le mot-clé `new` suivi d'accolades contenant des initializations de propriétés :

```textmate
// Création d'un objet anonyme simple
var personne = new { Nom = "Dupont", Prénom = "Jean", Age = 35 };

// Accès aux propriétés
Console.WriteLine($"Personne: {personne.Prénom} {personne.Nom}, {personne.Age} ans");
```


### Inférence de nom de propriété

Si vous utilisez des variables comme valeurs, les noms des propriétés peuvent être inférés à partir des noms des variables :

```textmate
string nom = "Dupont";
string prénom = "Jean";
int age = 35;

// Les noms des propriétés correspondent aux noms des variables
var personne = new { nom, prénom, age };

Console.WriteLine($"Personne: {personne.prénom} {personne.nom}, {personne.age} ans");
```


### Projection et transformation de données

Les objets anonymes sont souvent utilisés avec LINQ pour projeter ou transformer des données :

```textmate
// Classe Produit existante
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
    public string Catégorie { get; set; }
    public string Fournisseur { get; set; }
}

// Utilisation avec LINQ
List<Produit> produits = ObtenirProduits();

var résultat = produits
    .Where(p => p.Prix < 100)
    .Select(p => new {
        p.Nom,
        p.Prix,
        PrixTTC = p.Prix * 1.2m
    });

foreach (var item in résultat)
{
    Console.WriteLine($"{item.Nom}: {item.Prix:C} (TTC: {item.PrixTTC:C})");
}
```


### Objets anonymes dans les méthodes

Les objets anonymes sont typés, mais leur type exact est généré par le compilateur et ne peut pas être explicitement spécifié. Pour passer un objet anonyme à une méthode, on utilise généralement `object` ou `dynamic` comme type de paramètre :

```textmate
// Passage d'un objet anonyme à une méthode
public void AfficherDétails(object item)
{
    // Utilisation de la réflexion pour accéder aux propriétés
    var type = item.GetType();
    var propriétés = type.GetProperties();

    foreach (var prop in propriétés)
    {
        Console.WriteLine($"{prop.Name}: {prop.GetValue(item)}");
    }
}

// Utilisation
var produit = new { Nom = "Tablette", Prix = 299.99m, Stock = 15 };
AfficherDétails(produit);
```


### Égalité structurelle des objets anonymes

Les objets anonymes implémentent l'égalité structurelle, ce qui signifie que deux objets anonymes avec les mêmes propriétés et les mêmes valeurs sont considérés comme égaux :

```textmate
var p1 = new { Nom = "Dupont", Age = 30 };
var p2 = new { Nom = "Dupont", Age = 30 };
var p3 = new { Nom = "Martin", Age = 30 };

Console.WriteLine(p1.Equals(p2)); // true - mêmes propriétés et valeurs
Console.WriteLine(p1 == p2);      // false - opérateur == compare les références
Console.WriteLine(p1.Equals(p3)); // false - valeurs différentes
```


### Objets anonymes imbriqués

Il est possible de créer des objets anonymes contenant d'autres objets anonymes :

```textmate
var commande = new {
    Numéro = "CMD-123",
    Date = DateTime.Now,
    Client = new {
        Id = 101,
        Nom = "Dupont",
        Email = "jean.dupont@exemple.com"
    },
    Articles = new[] {
        new { Référence = "REF-001", Quantité = 2, PrixUnitaire = 19.99m },
        new { Référence = "REF-002", Quantité = 1, PrixUnitaire = 29.99m }
    }
};

Console.WriteLine($"Commande n°{commande.Numéro} du {commande.Date:d}");
Console.WriteLine($"Client: {commande.Client.Nom} ({commande.Client.Email})");

foreach (var article in commande.Articles)
{
    Console.WriteLine($"- {article.Référence}: {article.Quantité} x {article.PrixUnitaire:C}");
}
```


### Utilisation dans les requêtes asynchrones (.NET 8)

Dans .NET 8, les objets anonymes peuvent être utilisés dans les contextes asynchrones, par exemple avec Entity Framework Core :

```textmate
// Requête asynchrone avec projection vers un type anonyme
var résultats = await dbContext.Clients
    .Where(c => c.Pays == "France")
    .Select(c => new {
        c.Nom,
        c.Email,
        NbCommandes = c.Commandes.Count,
        DernièreCommande = c.Commandes.OrderByDescending(cmd => cmd.Date).FirstOrDefault().Date
    })
    .ToListAsync();

foreach (var client in résultats)
{
    Console.WriteLine($"{client.Nom} ({client.Email}): {client.NbCommandes} commandes, " +
                     $"dernière commande le {client.DernièreCommande:d}");
}
```


### Limites des objets anonymes

Les objets anonymes ont certaines limitations :

1. **Lecture seule** - Leurs propriétés sont en lecture seule et ne peuvent pas être modifiées après initialisation
2. **Portée limitée** - Ils sont principalement utiles dans la méthode où ils sont créés
3. **Pas d'héritage** - Ils ne peuvent pas hériter d'autres classes ni implémenter des interfaces
4. **Pas de méthodes** - Ils ne peuvent contenir que des propriétés, pas des méthodes

```textmate
var personne = new { Nom = "Dupont", Age = 30 };

// Erreur de compilation - les propriétés sont en lecture seule
// personne.Age = 31;

// Pour "modifier" un objet anonyme, il faut en créer un nouveau
var personneModifiée = new { personne.Nom, Age = 31 };
```


### Cas d'utilisation recommandés

Les objets anonymes sont particulièrement adaptés pour :

1. Projections LINQ
2. Regroupement temporaire de données
3. Résultats intermédiaires dans une méthode
4. Retour de plusieurs valeurs depuis une méthode (bien que les tuples soient souvent préférables)

## 6.6.2. Type dynamic

Le type `dynamic` permet d'effectuer des opérations sur des objets dont le type n'est pas connu à la compilation. La vérification du type est reportée à l'exécution.

### Utilisation de base

Le mot-clé `dynamic` permet de déclarer des variables dont le type est résolu à l'exécution :

```textmate
// Déclaration d'une variable dynamic
dynamic valeur = 10;
Console.WriteLine(valeur + 5);  // 15

valeur = "Bonjour";
Console.WriteLine(valeur + " le monde");  // "Bonjour le monde"

valeur = new List<int> { 1, 2, 3 };
Console.WriteLine(valeur.Count);  // 3
```


### Appels de méthodes dynamiques

Les membres appelés sur un objet `dynamic` sont résolus à l'exécution :

```textmate
public class Personne
{
    public string Nom { get; set; }
    public string Prénom { get; set; }

    public string NomComplet() => $"{Prénom} {Nom}";
}

// Utilisation avec dynamic
dynamic personne = new Personne { Nom = "Dupont", Prénom = "Jean" };

// Ces appels sont vérifiés à l'exécution, pas à la compilation
Console.WriteLine(personne.Nom);         // "Dupont"
Console.WriteLine(personne.NomComplet()); // "Jean Dupont"

// Erreur à l'exécution, pas à la compilation
try
{
    Console.WriteLine(personne.MéthodeInexistante());
}
catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}
```


### Interopérabilité COM

Le type `dynamic` est particulièrement utile pour l'interopérabilité avec les composants COM, comme les applications Office :

```textmate
// Exemple d'utilisation de Excel via COM (nécessite une référence à Microsoft.Office.Interop.Excel)
using Microsoft.Office.Interop.Excel;

public void ManipulerExcel()
{
    // Avec dynamic, le code est plus propre et plus facile à écrire
    dynamic excel = Activator.CreateInstance(Type.GetTypeFromProgID("Excel.Application"));
    excel.Visible = true;

    dynamic workbooks = excel.Workbooks;
    dynamic workbook = workbooks.Add();
    dynamic worksheet = workbook.Worksheets[1];

    worksheet.Cells[1, 1].Value = "Nom";
    worksheet.Cells[1, 2].Value = "Prénom";
    worksheet.Cells[2, 1].Value = "Dupont";
    worksheet.Cells[2, 2].Value = "Jean";

    // Sans dynamic, il faudrait beaucoup plus de code et de casts
    try
    {
        worksheet.SaveAs(@"C:\Temp\exemple.xlsx");
        workbook.Close();
        excel.Quit();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erreur: {ex.Message}");
    }
    finally
    {
        // Libération des ressources COM
        if (worksheet != null) System.Runtime.InteropServices.Marshal.ReleaseComObject(worksheet);
        if (workbook != null) System.Runtime.InteropServices.Marshal.ReleaseComObject(workbook);
        if (workbooks != null) System.Runtime.InteropServices.Marshal.ReleaseComObject(workbooks);
        if (excel != null) System.Runtime.InteropServices.Marshal.ReleaseComObject(excel);
    }
}
```


### Manipulation de JSON

Le type `dynamic` peut être utile pour manipuler du JSON, bien que les systèmes de typage modernes comme `System.Text.Json` offrent des alternatives typées plus sûres :

```textmate
// Avec Newtonsoft.Json
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public void ManipulerJSON()
{
    string json = @"{
        ""nom"": ""Dupont"",
        ""prénom"": ""Jean"",
        ""adresse"": {
            ""rue"": ""123 Avenue des Champs-Élysées"",
            ""ville"": ""Paris"",
            ""codePostal"": ""75008""
        },
        ""téléphones"": [
            ""01 23 45 67 89"",
            ""06 12 34 56 78""
        ]
    }";

    // Approche avec dynamic
    dynamic client = JsonConvert.DeserializeObject(json);

    Console.WriteLine($"{client.prénom} {client.nom}");
    Console.WriteLine($"Ville: {client.adresse.ville}");
    Console.WriteLine($"Téléphone principal: {client.téléphones[0]}");

    // Modification dynamique
    client.email = "jean.dupont@exemple.com";
    client.adresse.pays = "France";

    // Resérialisation
    string nouveauJson = JsonConvert.SerializeObject(client, Formatting.Indented);
    Console.WriteLine(nouveauJson);
}
```


### Dynamic vs var

Il est important de comprendre la différence entre `dynamic` et `var` :

```textmate
// var - Le type est déterminé à la compilation
var x = 10;  // x est de type int
// x = "texte";  // Erreur de compilation

// dynamic - Le type est résolu à l'exécution
dynamic y = 10;  // y peut changer de type pendant l'exécution
y = "texte";     // OK - pas d'erreur
```


### Performance et considérations

L'utilisation de `dynamic` a des implications en termes de performance et de sécurité du code :

1. **Performance** - Les opérations sur des objets `dynamic` sont plus lentes car elles nécessitent une résolution à l'exécution
2. **Sécurité du code** - Les erreurs qui seraient détectées à la compilation sont reportées à l'exécution
3. **IntelliSense** - Pas de suggestions d'autocomplétion dans l'IDE pour les objets `dynamic`
4. **Refactoring** - Les outils de refactoring peuvent ne pas prendre en compte les utilisations dynamiques

```textmate
public void PerfomanceDynamicVsStatic()
{
    const int iterations = 1000000;
    var stopwatch = new System.Diagnostics.Stopwatch();

    // Test avec type statique
    stopwatch.Start();
    string texte = "Test";
    for (int i = 0; i < iterations; i++)
    {
        int longueur = texte.Length;
    }
    stopwatch.Stop();
    Console.WriteLine($"Type statique: {stopwatch.ElapsedMilliseconds} ms");

    // Test avec dynamic
    stopwatch.Restart();
    dynamic texteD = "Test";
    for (int i = 0; i < iterations; i++)
    {
        int longueur = texteD.Length;
    }
    stopwatch.Stop();
    Console.WriteLine($"Type dynamic: {stopwatch.ElapsedMilliseconds} ms");
}
```


### ExpandoObject

`ExpandoObject` est une classe qui peut être utilisée avec `dynamic` pour créer des objets dont les membres peuvent être ajoutés ou supprimés à l'exécution :

```textmate
using System.Dynamic;
using System.Collections.Generic;

public void UtiliserExpandoObject()
{
    // Création d'un objet dynamique extensible
    dynamic personne = new ExpandoObject();

    // Ajout de propriétés
    personne.Nom = "Dupont";
    personne.Prénom = "Jean";
    personne.Age = 30;

    // Ajout d'une méthode
    personne.Présenter = (Action)(() =>
    {
        Console.WriteLine($"Je m'appelle {personne.Prénom} {personne.Nom}, j'ai {personne.Age} ans.");
    });

    // Utilisation des propriétés et méthodes
    Console.WriteLine($"{personne.Prénom} {personne.Nom}");
    personne.Présenter();

    // ExpandoObject implémente IDictionary<string, object>
    var dictionnaire = (IDictionary<string, object>)personne;

    // Énumération des propriétés
    foreach (var item in dictionnaire)
    {
        Console.WriteLine($"{item.Key}: {item.Value}");
    }

    // Ajout et suppression de propriétés via l'interface dictionnaire
    dictionnaire.Add("Email", "jean.dupont@exemple.com");
    dictionnaire.Remove("Age");

    // Vérification de l'existence d'une propriété
    if (dictionnaire.ContainsKey("Email"))
    {
        Console.WriteLine($"Email: {personne.Email}");
    }

    if (!dictionnaire.ContainsKey("Age"))
    {
        Console.WriteLine("La propriété Age a été supprimée.");
    }
}
```


### DynamicObject

`DynamicObject` est une classe de base qui permet de créer des objets dynamiques personnalisés en surchargeant des méthodes spécifiques :

```textmate
using System.Dynamic;
using System.Collections.Generic;

// Classe dynamique personnalisée
public class DictionaireDynamique : DynamicObject
{
    // Dictionnaire interne pour stocker les propriétés
    private Dictionary<string, object> _propriétés = new Dictionary<string, object>();

    // Accès aux propriétés
    public override bool TryGetMember(GetMemberBinder binder, out object result)
    {
        if (_propriétés.TryGetValue(binder.Name, out result))
        {
            return true;
        }

        result = null;
        return false;
    }

    // Définition des propriétés
    public override bool TrySetMember(SetMemberBinder binder, object value)
    {
        _propriétés[binder.Name] = value;
        return true;
    }

    // Appels de méthodes dynamiques
    public override bool TryInvokeMember(InvokeMemberBinder binder, object[] args, out object result)
    {
        if (binder.Name == "ContientClé" && args.Length == 1 && args[0] is string)
        {
            result = _propriétés.ContainsKey((string)args[0]);
            return true;
        }

        return base.TryInvokeMember(binder, args, out result);
    }

    // Liste des membres
    public override IEnumerable<string> GetDynamicMemberNames()
    {
        return _propriétés.Keys;
    }
}

// Utilisation
public void UtiliserDynamicObject()
{
    dynamic obj = new DictionaireDynamique();

    // Définition de propriétés
    obj.Nom = "Dupont";
    obj.Age = 30;

    // Accès aux propriétés
    Console.WriteLine($"Nom: {obj.Nom}, Age: {obj.Age}");

    // Appel de méthode dynamique
    bool contientNom = obj.ContientClé("Nom");
    Console.WriteLine($"Contient la clé 'Nom': {contientNom}");

    // Propriété inexistante - retourne null sans erreur grâce à notre implémentation
    Console.WriteLine($"Adresse: {obj.Adresse ?? "Non définie"}");
}
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Objets anonymes | ✅ | ✅ |
| Type dynamic | ✅ | ✅ |
| ExpandoObject | ✅ | ✅ |
| DynamicObject | ✅ | ✅ |
| Performance du DLR | De base | Améliorée |
| Interopérabilité COM | ✅ (natif) | ✅ (nécessite NuGet pour certains cas) |
| Nullable avec dynamic | ❌ | ✅ |

## 6.6.3. Interopérabilité DLR

Le Dynamic Language Runtime (DLR) est une couche au-dessus du Common Language Runtime (CLR) qui prend en charge les langages dynamiques en .NET.

### Fonctionnement interne du DLR

Le DLR fonctionne en générant et en mettant en cache des arbres d'expressions qui représentent les opérations dynamiques :

```textmate
// Lorsque vous écrivez :
dynamic obj = GetObject();
int result = obj.Add(5, 10);

// Le DLR génère en interne quelque chose comme :
// Si obj est de type Calculator
if (obj is Calculator)
{
    result = ((Calculator)obj).Add(5, 10);
}
// Si obj est de type string
else if (obj is string)
{
    // Erreur car string n'a pas de méthode Add
    throw new RuntimeBinderException();
}
// etc...
```


### Communication avec des langages dynamiques

Le DLR facilite l'interopérabilité avec des langages dynamiques comme IronPython ou IronRuby :

```textmate
// Exemple d'utilisation d'IronPython depuis C#
// Nécessite les packages NuGet: IronPython et IronPython.StdLib
using IronPython.Hosting;
using Microsoft.Scripting.Hosting;

public void UtiliserIronPython()
{
    // Création d'un moteur Python
    ScriptEngine moteur = Python.CreateEngine();

    // Exécution de code Python simple
    ScriptScope portée = moteur.CreateScope();
    moteur.Execute("result = 5 + 10", portée);

    // Récupération des variables depuis l'environnement Python
    int résultat = portée.GetVariable<int>("result");
    Console.WriteLine($"Résultat: {résultat}");  // 15

    // Définition d'une fonction Python
    moteur.Execute(@"
def calculer_factoriel(n):
    if n <= 1:
        return 1
    else:
        return n * calculer_factoriel(n - 1)
", portée);

    // Appel de la fonction Python depuis C#
    dynamic calculerFactoriel = portée.GetVariable("calculer_factoriel");
    int factoriel = calculerFactoriel(5);
    Console.WriteLine($"Factoriel de 5: {factoriel}");  // 120

    // Passage d'objets C# à Python
    portée.SetVariable("personne", new { Nom = "Dupont", Age = 30 });
    moteur.Execute("info = f\"Nom: {personne.Nom}, Age: {personne.Age}\"", portée);
    string info = portée.GetVariable<string>("info");
    Console.WriteLine(info);  // "Nom: Dupont, Age: 30"
}
```


### Manipulation dynamique de JSON

Voici un exemple de manipulation dynamique de JSON avec le package `Newtonsoft.Json` (disponible à la fois pour .NET Framework 4.7.2 et .NET 8) :

```textmate
using Newtonsoft.Json;
using Newtonsoft.Json.Converters;

public void ManipulerJSONAvecExpando()
{
    string json = @"{
        ""nom"": ""Dupont"",
        ""prénom"": ""Jean"",
        ""compétences"": [""C#"", ""SQL"", ""JavaScript""],
        ""adresse"": {
            ""rue"": ""123 Avenue des Champs-Élysées"",
            ""ville"": ""Paris"",
            ""codePostal"": ""75008""
        }
    }";

    // Conversion du JSON en ExpandoObject
    var convertisseur = new ExpandoObjectConverter();
    dynamic personne = JsonConvert.DeserializeObject<System.Dynamic.ExpandoObject>(json, convertisseur);

    // Accès aux propriétés
    Console.WriteLine($"{personne.prénom} {personne.nom}");

    // Accès aux propriétés imbriquées
    Console.WriteLine($"Ville: {personne.adresse.ville}");

    // Parcours d'un tableau
    Console.Write("Compétences: ");
    foreach (string compétence in personne.compétences)
    {
        Console.Write($"{compétence} ");
    }
    Console.WriteLine();

    // Modification et ajout de propriétés
    personne.email = "jean.dupont@exemple.com";
    personne.adresse.pays = "France";

    // Conversion en JSON
    string nouveauJson = JsonConvert.SerializeObject(personne, Formatting.Indented);
    Console.WriteLine(nouveauJson);
}
```


### Réflexion dynamique

Le DLR peut être combiné avec la réflexion pour créer des systèmes d'appel de méthodes très flexibles :

```textmate
using System.Reflection;

public class ServiceDynamique
{
    // Méthode qui appelle dynamiquement une méthode sur un objet en fonction du nom
    public object AppelerMéthode(object instance, string nomMéthode, params object[] paramètres)
    {
        if (instance == null)
            throw new ArgumentNullException(nameof(instance));

        Type type = instance.GetType();
        MethodInfo méthode = type.GetMethod(nomMéthode);

        if (méthode == null)
            throw new ArgumentException($"La méthode '{nomMéthode}' n'existe pas sur le type {type.Name}");

        return méthode.Invoke(instance, paramètres);
    }

    // Version avec dynamic - plus simple mais moins de contrôle d'erreur
    public object AppelerMéthodeDynamique(object instance, string nomMéthode, params object[] paramètres)
    {
        dynamic objetDynamique = instance;

        try
        {
            // Utilise la réflexion en interne via le DLR
            return GetType()
                .GetMethod("AppelerDynamiquement", BindingFlags.NonPublic | BindingFlags.Instance)
                .MakeGenericMethod(paramètres.Select(p => p.GetType()).ToArray())
                .Invoke(this, new[] { objetDynamique, nomMéthode, paramètres });
        }
        catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException ex)
        {
            throw new ArgumentException($"Erreur lors de l'appel de la méthode '{nomMéthode}': {ex.Message}");
        }
    }

    // Méthode helper générique pour l'appel dynamique
    private object AppelerDynamiquement<T1, T2>(dynamic instance, string nomMéthode, object[] paramètres)
    {
        return instance.GetType().GetMethod(nomMéthode).Invoke(instance, paramètres);
    }
}

// Classe de test
public class Calculateur
{
    public int Additionner(int a, int b) => a + b;
    public int Multiplier(int a, int b) => a * b;
    public string Formater(int valeur) => $"La valeur est {valeur}";
}

// Utilisation
public void TesterAppelDynamique()
{
    var service = new ServiceDynamique();
    var calc = new Calculateur();

    // Appel via réflexion
    int résultat1 = (int)service.AppelerMéthode(calc, "Additionner", 5, 3);
    Console.WriteLine($"5 + 3 = {résultat1}");  // 8

    // Appel via dynamic
    int résultat2 = (int)service.AppelerMéthodeDynamique(calc, "Multiplier", 5, 3);
    Console.WriteLine($"5 * 3 = {résultat2}");  // 15

    // Appel avec un autre type de retour
    string message = (string)service.AppelerMéthode(calc, "Formater", 42);
    Console.WriteLine(message);  // "La valeur est 42"
}
```


### Génération dynamique d'interfaces utilisateur

Le type `dynamic` peut être utilisé pour générer dynamiquement des interfaces utilisateur basées sur des données :

```textmate
// Pour WPF (.NET Framework 4.7.2 ou .NET 8)
public void CréerInterfaceDynamique(dynamic données)
{
    // Création dynamique d'un formulaire en fonction des propriétés de l'objet
    var stackPanel = new System.Windows.Controls.StackPanel();

    // Récupération des propriétés via réflexion
    var propriétés = données.GetType().GetProperties();

    foreach (var propriété in propriétés)
    {
        // Création d'un libellé pour la propriété
        var label = new System.Windows.Controls.Label
        {
            Content = propriété.Name
        };
        stackPanel.Children.Add(label);

        // Création d'un contrôle de saisie adapté au type de la propriété
        object valeur = propriété.GetValue(données);
        System.Windows.UIElement contrôle;

        if (propriété.PropertyType == typeof(bool))
        {
            contrôle = new System.Windows.Controls.CheckBox
            {
                IsChecked = (bool)valeur
            };
        }
        else if (propriété.PropertyType == typeof(DateTime))
        {
            contrôle = new System.Windows.Controls.DatePicker
            {
                SelectedDate = (DateTime)valeur
            };
        }
        else
        {
            contrôle = new System.Windows.Controls.TextBox
            {
                Text = valeur?.ToString()
            };
        }

        stackPanel.Children.Add(contrôle);
    }

    // Création d'une fenêtre contenant le formulaire
    var fenêtre = new System.Windows.Window
    {
        Title = "Formulaire Dynamique",
        Content = stackPanel,
        Width = 300,
        Height = 400
    };

    fenêtre.ShowDialog();
}
```


Conclusion sur les types anonymes et dynamiques
Les types anonymes et dynamiques offrent une grande flexibilité pour manipuler des données en C#, mais cette flexibilité s'accompagne de compromis en termes de sécurité du typage et de performance.
Quand utiliser des types anonymes
Les types anonymes sont utiles dans les situations suivantes :
Projections LINQ temporaires
Création de structures de données légères et éphémères
Regroupement de valeurs connexes sans créer une classe formelle
Utilisation comme type de retour pour des résultats intermédiaires
Quand utiliser le type dynamic
Le type dynamic est particulièrement adapté dans ces contextes :
Interopérabilité avec des API COM (comme Office)
Manipulation de données JSON ou XML lorsque la structure n'est pas connue à l'avance
Interopérabilité avec des langages dynamiques comme JavaScript ou Python
Implémentation de comportements d'appel dynamiques (proxys, AOP)
Construction de frameworks ou bibliothèques génériques très flexibles
Bonnes pratiques
Limiter la portée - Restreindre l'utilisation de dynamic aux endroits où c'est vraiment nécessaire
Tests - Tester exhaustivement le code utilisant dynamic car les erreurs ne seront détectées qu'à l'exécution
Documentation - Documenter clairement les attendus pour les paramètres et retours de type dynamic
Préférer le typage statique - Utiliser des types statiques quand c'est possible pour bénéficier de la vérification à la compilation
Types anonymes pour les projections locales - Réserver les types anonymes aux projections locales et temporaires
Types d'applications adaptées
Les types anonymes et dynamiques sont particulièrement utiles dans :
Applications d'analyse de données
Outils d'intégration
Applications avec des structures de données flexibles
Frameworks et bibliothèques génériques
Extensions et plugins
Utilisés judicieusement, ces outils peuvent simplifier le code et accroître la flexibilité. Cependant, ils doivent être employés avec discernement, en tenant compte des compromis en matière de sécurité du type et de performance.


⏭️ 6.7. [Méthodes anonymes](/06-fonctionnalites-avancees-de-csharp/6-7-methodes-anonymes.md)
