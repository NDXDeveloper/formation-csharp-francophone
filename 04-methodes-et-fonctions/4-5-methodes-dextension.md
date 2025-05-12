# 4.5. Méthodes d'extension en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les méthodes d'extension sont l'une des fonctionnalités les plus élégantes et utiles de C#. Elles vous permettent d'ajouter de nouvelles méthodes à des types existants sans avoir à modifier leur code source ou à créer des classes dérivées. Dans cette section, nous allons explorer comment créer et utiliser ces méthodes, voir comment elles sont employées dans LINQ, et découvrir les bonnes pratiques ainsi que les pièges à éviter.

## Introduction aux méthodes d'extension

Imaginez que vous travaillez avec une classe `string` et que vous souhaitez ajouter une méthode `EstPalindrome()` qui vérifie si une chaîne se lit de la même façon dans les deux sens. Sans les méthodes d'extension, vous auriez deux options :

1. Créer une classe utilitaire avec une méthode statique :
   ```csharp
   public static class StringUtil
   {
       public static bool EstPalindrome(string texte)
       {
           // Implémentation
       }
   }

   // Utilisation
   bool estPalindrome = StringUtil.EstPalindrome("kayak");
   ```

2. Créer une classe dérivée (impossible avec `string` qui est scellée) :
   ```csharp
   public class MaChaine : string // Impossible car string est sealed !
   {
       public bool EstPalindrome()
       {
           // Implémentation
       }
   }
   ```

Avec les méthodes d'extension, vous pouvez faire beaucoup mieux :
```csharp
// Définition
public static class StringExtensions
{
    public static bool EstPalindrome(this string texte)
    {
        // Implémentation
    }
}

// Utilisation
bool estPalindrome = "kayak".EstPalindrome();
```

Comme vous pouvez le voir, la méthode d'extension vous permet d'appeler `EstPalindrome()` directement sur une instance de `string`, comme si cette méthode faisait partie de la classe d'origine !

## 4.5.1. Création et utilisation

### Création d'une méthode d'extension

Pour créer une méthode d'extension, vous devez suivre ces règles :

1. La méthode doit être définie dans une **classe statique**
2. La méthode elle-même doit être **statique**
3. Le premier paramètre doit être précédé du mot-clé **`this`** et représente le type que vous étendez
4. Les paramètres supplémentaires (s'il y en a) sont définis normalement

Voici la syntaxe de base :

```csharp
public static class TypeExtensions
{
    public static ReturnType NomMethode(this TypeAEtendre instance, AutresParametres...)
    {
        // Implémentation utilisant 'instance'
        return resultat;
    }
}
```

### Exemple concret : Extensions pour les chaînes de caractères

Créons quelques méthodes d'extension utiles pour le type `string` :

```csharp
using System;
using System.Linq;

public static class StringExtensions
{
    // Vérifie si une chaîne est un palindrome
    public static bool EstPalindrome(this string texte)
    {
        if (string.IsNullOrEmpty(texte))
            return false;

        string texteFiltré = new string(texte.Where(char.IsLetterOrDigit).ToArray()).ToLower();
        return texteFiltré.SequenceEqual(texteFiltré.Reverse());
    }

    // Compte le nombre de mots dans une chaîne
    public static int CompterMots(this string texte)
    {
        if (string.IsNullOrEmpty(texte))
            return 0;

        return texte.Split(new[] { ' ', '\t', '\n', '\r' }, StringSplitOptions.RemoveEmptyEntries).Length;
    }

    // Tronque une chaîne à une longueur maximale et ajoute "..." si nécessaire
    public static string Tronquer(this string texte, int longueurMax)
    {
        if (string.IsNullOrEmpty(texte) || texte.Length <= longueurMax)
            return texte;

        return texte.Substring(0, longueurMax) + "...";
    }
}
```

### Utilisation des méthodes d'extension

Une fois définies, les méthodes d'extension peuvent être utilisées comme si elles étaient des méthodes d'instance du type étendu :

```csharp
// Assurez-vous d'avoir inclus l'espace de noms où les extensions sont définies
using MonProjet.Extensions;

// Utilisation des extensions string
string phrase = "Engage le jeu que je le gagne";
bool estPalindrome = phrase.EstPalindrome();  // Retourne true
int nombreMots = phrase.CompterMots();        // Retourne 7
string version_courte = phrase.Tronquer(10);  // Retourne "Engage le..."

Console.WriteLine($"Est un palindrome : {estPalindrome}");
Console.WriteLine($"Nombre de mots : {nombreMots}");
Console.WriteLine($"Version tronquée : {version_courte}");
```

### Comment C# trouve les méthodes d'extension

Pour que les méthodes d'extension soient disponibles, deux conditions doivent être remplies :

1. La classe statique contenant les extensions doit être accessible (publique)
2. L'espace de noms contenant la classe d'extensions doit être importé avec `using`

Si vous n'importez pas l'espace de noms, vous pouvez toujours utiliser les méthodes d'extension, mais vous devrez le faire comme des méthodes statiques normales :

```csharp
// Sans using MonProjet.Extensions;
bool estPalindrome = StringExtensions.EstPalindrome("kayak");
```

### Étendre d'autres types

Vous pouvez créer des méthodes d'extension pour n'importe quel type, y compris :
- Les types primitifs (`int`, `double`, etc.)
- Les types du framework .NET (`DateTime`, `List<T>`, etc.)
- Vos propres classes et structures
- Les interfaces

#### Exemples pour différents types

```csharp
public static class DateTimeExtensions
{
    // Vérifie si une date est un jour férié en France
    public static bool EstJourFerieEnFrance(this DateTime date)
    {
        // Liste simplifiée de quelques jours fériés
        if (date.Month == 1 && date.Day == 1) return true;    // Jour de l'an
        if (date.Month == 5 && date.Day == 1) return true;    // Fête du travail
        if (date.Month == 5 && date.Day == 8) return true;    // Victoire 1945
        if (date.Month == 7 && date.Day == 14) return true;   // Fête nationale
        if (date.Month == 8 && date.Day == 15) return true;   // Assomption
        if (date.Month == 11 && date.Day == 1) return true;   // Toussaint
        if (date.Month == 11 && date.Day == 11) return true;  // Armistice
        if (date.Month == 12 && date.Day == 25) return true;  // Noël

        // Implémentation simplifiée, il faudrait aussi gérer Pâques, etc.
        return false;
    }

    // Retourne la date du prochain jour ouvré
    public static DateTime ProchainJourOuvre(this DateTime date)
    {
        DateTime resultat = date.AddDays(1);
        while (resultat.DayOfWeek == DayOfWeek.Saturday ||
               resultat.DayOfWeek == DayOfWeek.Sunday ||
               resultat.EstJourFerieEnFrance())
        {
            resultat = resultat.AddDays(1);
        }
        return resultat;
    }
}

public static class IntExtensions
{
    // Vérifie si un nombre est premier
    public static bool EstPremier(this int nombre)
    {
        if (nombre <= 1) return false;
        if (nombre == 2) return true;
        if (nombre % 2 == 0) return false;

        var limite = (int)Math.Sqrt(nombre);
        for (int i = 3; i <= limite; i += 2)
        {
            if (nombre % i == 0) return false;
        }

        return true;
    }

    // Convertit un nombre en texte écrit en français
    public static string EnTexte(this int nombre)
    {
        // Implémentation simplifiée
        if (nombre == 0) return "zéro";
        if (nombre == 1) return "un";
        if (nombre == 2) return "deux";
        // etc.

        return nombre.ToString(); // Par défaut retourne le nombre en texte
    }
}

// Extension d'une interface
public interface IIdentifiable
{
    int Id { get; }
}

public static class IdentifiableExtensions
{
    public static bool EstNouveau(this IIdentifiable entite)
    {
        return entite.Id == 0;
    }
}
```

### Exemples d'utilisation

```csharp
DateTime aujourdhui = DateTime.Now;
if (aujourdhui.EstJourFerieEnFrance())
{
    Console.WriteLine("C'est férié aujourd'hui !");
}

DateTime prochaineJourneeDetravail = aujourdhui.ProchainJourOuvre();
Console.WriteLine($"Prochain jour ouvré : {prochaineJourneeDetravail:d}");

int nombre = 17;
if (nombre.EstPremier())
{
    Console.WriteLine($"{nombre} est un nombre premier");
}

Console.WriteLine($"5 en lettres : {5.EnTexte()}");

// Utilisation avec une interface
public class Produit : IIdentifiable
{
    public int Id { get; set; }
    public string Nom { get; set; }
}

var produit = new Produit { Id = 0, Nom = "Nouveau produit" };
if (produit.EstNouveau())
{
    Console.WriteLine("Ce produit n'a pas encore été enregistré en base de données");
}
```

## 4.5.2. LINQ comme exemple de méthodes d'extension

LINQ (Language Integrated Query) est un excellent exemple de l'utilisation des méthodes d'extension en C#. LINQ permet d'écrire des requêtes directement dans le code C# pour manipuler des collections de données.

### Comment LINQ utilise les méthodes d'extension

La plupart des méthodes LINQ sont implémentées comme des extensions sur les types implémentant `IEnumerable<T>`. C'est ce qui permet d'écrire des requêtes de façon fluide et intuitive.

Voici quelques-unes des méthodes d'extension LINQ les plus courantes :

```csharp
// Ces méthodes sont définies comme extensions dans System.Linq
using System.Linq;

List<int> nombres = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Where - filtre les éléments selon un prédicat
var nombresPairs = nombres.Where(n => n % 2 == 0);

// Select - projette chaque élément en utilisant une fonction
var carres = nombres.Select(n => n * n);

// OrderBy - trie les éléments selon un critère
var nombresTries = nombres.OrderByDescending(n => n);

// FirstOrDefault - retourne le premier élément ou une valeur par défaut
var premier = nombres.FirstOrDefault(n => n > 5);

// Any - vérifie si au moins un élément satisfait une condition
bool contientImpair = nombres.Any(n => n % 2 != 0);

// Combinaison de méthodes (chaînage)
var resultat = nombres
    .Where(n => n > 3)
    .Select(n => n * 2)
    .OrderBy(n => n)
    .ToList();
```

### Aperçu de l'implémentation de LINQ

Voici comment certaines méthodes LINQ pourraient être implémentées (version simplifiée) :

```csharp
public static class MyLinqExtensions
{
    // Implémentation simplifiée de Where
    public static IEnumerable<T> Where<T>(this IEnumerable<T> source, Func<T, bool> predicate)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));

        if (predicate == null)
            throw new ArgumentNullException(nameof(predicate));

        // Utilisation du pattern "yield return" pour l'évaluation différée
        foreach (var item in source)
        {
            if (predicate(item))
                yield return item;
        }
    }

    // Implémentation simplifiée de Select
    public static IEnumerable<TResult> Select<TSource, TResult>(
        this IEnumerable<TSource> source,
        Func<TSource, TResult> selector)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));

        if (selector == null)
            throw new ArgumentNullException(nameof(selector));

        foreach (var item in source)
        {
            yield return selector(item);
        }
    }

    // Implémentation simplifiée de FirstOrDefault
    public static T FirstOrDefault<T>(this IEnumerable<T> source, Func<T, bool> predicate = null)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));

        if (predicate == null)
        {
            foreach (var item in source)
                return item;

            return default;
        }

        foreach (var item in source)
        {
            if (predicate(item))
                return item;
        }

        return default;
    }
}
```

### Créer vos propres extensions LINQ

Vous pouvez créer vos propres méthodes d'extension LINQ pour ajouter des fonctionnalités personnalisées :

```csharp
public static class MyCustomLinqExtensions
{
    // Méthode qui retourne un élément aléatoire d'une collection
    public static T AleatoireOuDefault<T>(this IEnumerable<T> source)
    {
        if (source == null || !source.Any())
            return default;

        var liste = source as IList<T> ?? source.ToList();
        Random rand = new Random();
        int index = rand.Next(0, liste.Count);
        return liste[index];
    }

    // Méthode qui divise une collection en sous-collections de taille spécifiée
    public static IEnumerable<IEnumerable<T>> Partitionner<T>(this IEnumerable<T> source, int taille)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));

        if (taille <= 0)
            throw new ArgumentException("La taille doit être positive", nameof(taille));

        var currentBatch = new List<T>(taille);
        foreach (var item in source)
        {
            currentBatch.Add(item);
            if (currentBatch.Count == taille)
            {
                yield return currentBatch;
                currentBatch = new List<T>(taille);
            }
        }

        if (currentBatch.Count > 0)
            yield return currentBatch;
    }

    // Méthode qui applique une action à chaque élément et retourne la collection
    public static IEnumerable<T> ForEach<T>(this IEnumerable<T> source, Action<T> action)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));

        if (action == null)
            throw new ArgumentNullException(nameof(action));

        foreach (var item in source)
        {
            action(item);
            yield return item;
        }
    }
}
```

Utilisation :

```csharp
List<string> fruits = new List<string> { "pomme", "banane", "orange", "fraise", "kiwi" };

// Obtenir un fruit aléatoire
string fruitAleatoire = fruits.AleatoireOuDefault();
Console.WriteLine($"Fruit choisi au hasard : {fruitAleatoire}");

// Diviser la liste en groupes de 2
var groupes = fruits.Partitionner(2).ToList();
for (int i = 0; i < groupes.Count; i++)
{
    Console.WriteLine($"Groupe {i+1}: {string.Join(", ", groupes[i])}");
}

// Afficher chaque fruit en majuscules et obtenir la liste modifiée
var fruitsEnMajuscules = fruits
    .ForEach(f => Console.WriteLine(f.ToUpper()))
    .Select(f => f.ToUpper())
    .ToList();
```

## 4.5.3. Bonnes pratiques et pièges courants

Les méthodes d'extension sont puissantes, mais elles doivent être utilisées judicieusement. Voici quelques bonnes pratiques et pièges à éviter.

### Bonnes pratiques

#### 1. Nommer clairement vos classes d'extensions

Suivez une convention de nommage cohérente pour vos classes d'extensions, généralement en utilisant le suffixe "Extensions" :

```csharp
public static class StringExtensions { /* ... */ }
public static class DateTimeExtensions { /* ... */ }
public static class EnumerableExtensions { /* ... */ }
```

#### 2. Organiser les extensions dans des espaces de noms appropriés

Placez vos extensions dans des espaces de noms qui correspondent à leur utilisation :

```csharp
namespace MonProjet.Extensions.Text
{
    public static class StringExtensions { /* ... */ }
}

namespace MonProjet.Extensions.Data
{
    public static class DbConnectionExtensions { /* ... */ }
}
```

#### 3. Documenter vos méthodes d'extension

Utilisez les commentaires XML pour documenter clairement vos extensions :

```csharp
/// <summary>
/// Vérifie si la chaîne est un palindrome, en ignorant la casse, les espaces et la ponctuation.
/// </summary>
/// <param name="texte">La chaîne à vérifier.</param>
/// <returns>Vrai si la chaîne est un palindrome, faux sinon.</returns>
/// <example>
/// bool resultat = "Engage le jeu que je le gagne".EstPalindrome(); // Retourne true
/// </example>
public static bool EstPalindrome(this string texte)
{
    // Implémentation...
}
```

#### 4. Gérer les valeurs null

Décidez comment gérer les instances null et documentez ce comportement :

```csharp
public static string Tronquer(this string texte, int longueurMax)
{
    // Option 1 : Retourner null si l'entrée est null
    if (texte == null)
        return null;

    // Option 2 : Lancer une exception
    // if (texte == null)
    //     throw new ArgumentNullException(nameof(texte));

    // Option 3 : Traiter null comme une chaîne vide
    // texte = texte ?? "";

    if (texte.Length <= longueurMax)
        return texte;

    return texte.Substring(0, longueurMax) + "...";
}
```

#### 5. Privilégier la cohérence sémantique

Vos méthodes d'extension devraient avoir un comportement cohérent avec les méthodes existantes du type que vous étendez :

```csharp
// Bon: Cohérent avec la méthode String.ToUpper()
public static string ToTitleCase(this string texte)
{
    if (string.IsNullOrEmpty(texte))
        return texte;

    TextInfo textInfo = new CultureInfo("fr-FR").TextInfo;
    return textInfo.ToTitleCase(texte.ToLower());
}

// Mauvais: Comportement incohérent, modifie l'instance au lieu de retourner une nouvelle valeur
public static string MettreEnMajuscules(this string texte)
{
    // Les chaînes sont immuables en C#, cette approche est incohérente
    texte = texte.ToUpper(); // Crée une confusion car cela semble modifier l'original
    return texte;
}
```

#### 6. Limiter la portée de vos extensions

Ne créez pas d'extensions trop génériques qui pourraient entrer en conflit avec d'autres :

```csharp
// Trop générique, risque de conflits
public static class ObjectExtensions
{
    public static string ToJson(this object obj)
    {
        // Implémentation...
    }
}

// Mieux: Cibler des types spécifiques
public static class EntityExtensions
{
    public static string ToJson<T>(this T entity) where T : IEntity
    {
        // Implémentation...
    }
}
```

### Pièges courants

#### 1. Conflit de noms avec les méthodes existantes

Si vous créez une méthode d'extension avec le même nom qu'une méthode d'instance existante, c'est la méthode d'instance qui sera toujours appelée :

```csharp
public static class StringExtensions
{
    // Ne sera jamais appelée car String.ToUpper() existe déjà
    public static string ToUpper(this string texte)
    {
        Console.WriteLine("Extension appelée");
        return texte.ToUpper();
    }
}

string test = "exemple";
test.ToUpper(); // Appelle String.ToUpper(), pas votre extension
```

#### 2. Masquage involontaire des méthodes

Les méthodes d'extension peuvent masquer involontairement d'autres méthodes avec des signatures similaires :

```csharp
public static class Extensions
{
    // Version 1: Extension pour IEnumerable<string>
    public static string Joindre(this IEnumerable<string> elements)
    {
        return string.Join(", ", elements);
    }

    // Version 2: Extension pour IEnumerable<T>
    public static string Joindre<T>(this IEnumerable<T> elements)
    {
        return string.Join(", ", elements);
    }
}

// Quelle méthode sera appelée ?
List<string> liste = new List<string> { "a", "b", "c" };
string resultat = liste.Joindre(); // Appelle la version 1 (plus spécifique)
```

#### 3. Performance et évaluation différée

Avec LINQ et les extensions sur `IEnumerable<T>`, attention à l'évaluation différée qui peut causer des surprises :

```csharp
// Piège courant
IEnumerable<int> GetNombres()
{
    var liste = new List<int>();
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine($"Génération de {i}");
        liste.Add(i);
    }

    // Le premier filtrage n'est pas encore exécuté !
    var filtre = liste.Where(n => n % 2 == 0);
    Console.WriteLine("Filtrage terminé"); // Exécuté immédiatement, mais le filtrage ne l'est pas

    return filtre; // L'évaluation sera effectuée seulement quand la séquence sera énumérée
}

// Utilisation
var nombres = GetNombres();
Console.WriteLine("Avant énumération");
foreach (var n in nombres) // C'est ICI que le filtrage est vraiment exécuté
{
    Console.WriteLine($"Nombre: {n}");
}
```

#### 4. Extension des types scellés

N'utilisez pas les méthodes d'extension pour contourner les restrictions de types scellés (sealed) en changeant leur comportement fondamental :

```csharp
// Mauvaise pratique: Tenter de modifier le comportement fondamental de String
public static class StringExtensions
{
    // Simule un comportement mutable sur String qui est censé être immuable
    public static void RemplacerContenu(this string source, string nouveau)
    {
        // Tentative de modifier une chaîne (impossible)
        // Ce code est trompeur et ne fonctionne pas comme suggéré
    }
}
```

#### 5. Trop d'extensions rendent le code difficile à suivre

Trop de méthodes d'extension peuvent surcharger l'IntelliSense et rendre le code difficile à comprendre :

```csharp
// Évitez de surcharger un type avec trop d'extensions non reliées
chaine
    .NormaliserCaracteres()
    .ConvertirEnJSON()
    .CalculerHash()
    .EnvoyerParMail()
    .SauvegarderDansFichier()
    .CompresserContenu();
```

### Exemples de bonnes pratiques en action

#### Extension générique bien conçue

```csharp
public static class EnumerableExtensions
{
    /// <summary>
    /// Applique une fonction à chaque élément et retourne uniquement les résultats non-null.
    /// </summary>
    public static IEnumerable<TResult> SelectNonNull<TSource, TResult>(
        this IEnumerable<TSource> source,
        Func<TSource, TResult> selector)
        where TResult : class
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
        if (selector == null)
            throw new ArgumentNullException(nameof(selector));

        foreach (var item in source)
        {
            var result = selector(item);
            if (result != null)
                yield return result;
        }
    }

    /// <summary>
    /// Retourne true si la séquence est null ou ne contient aucun élément.
    /// </summary>
    public static bool IsNullOrEmpty<T>(this IEnumerable<T> source)
    {
        return source == null || !source.Any();
    }
}
```

#### Organisation en bibliothèque d'extensions

```csharp
// Dans un fichier StringExtensions.cs
namespace MonProjet.Extensions
{
    public static class StringExtensions
    {
        // Méthodes d'extension pour String
    }
}

// Dans un fichier DateTimeExtensions.cs
namespace MonProjet.Extensions
{
    public static class DateTimeExtensions
    {
        // Méthodes d'extension pour DateTime
    }
}

// Dans une classe client
using MonProjet.Extensions;

public class Client
{
    public void ProcessData()
    {
        // Utilisation des extensions
        string texte = "exemple".Tronquer(3);
        DateTime prochainJour = DateTime.Now.ProchainJourOuvre();
    }
}
```

### Quand faut-il utiliser des méthodes d'extension ?

#### Bonnes situations pour utiliser des méthodes d'extension

1. **Ajouter des fonctionnalités à des types que vous ne pouvez pas modifier**
   ```csharp
   // Ajouter des méthodes à String, DateTime, etc.
   public static bool ContientChiffre(this string texte)
   {
       return texte.Any(char.IsDigit);
   }
   ```

2. **Implémenter le pattern "fluent interface"**
   ```csharp
   public static class BuilderExtensions
   {
       public static StringBuilder AppendDateHeure(this StringBuilder sb)
       {
           return sb.Append(DateTime.Now.ToString());
       }

       public static StringBuilder AppendLigne(this StringBuilder sb, string ligne)
       {
           return sb.AppendLine(ligne);
       }
   }

   // Utilisation
   var sb = new StringBuilder()
       .Append("Log: ")
       .AppendDateHeure()
       .AppendLigne(" - Opération démarrée")
       .AppendLigne(" - Traitement en cours");
   ```

3. **Créer des utilitaires pour le type `IEnumerable<T>`**
   ```csharp
   public static double Moyenne(this IEnumerable<double> source)
   {
       if (source == null || !source.Any())
           throw new ArgumentException("La séquence est vide ou null");

       return source.Sum() / source.Count();
   }
   ```

4. **Ajouter des fonctionnalités à des interfaces**
   ```csharp
   public interface IRepository<T>
   {
       IEnumerable<T> GetAll();
       T GetById(int id);
       void Add(T entity);
   }

   public static class RepositoryExtensions
   {
       public static IEnumerable<T> GetByIds<T>(this IRepository<T> repository, IEnumerable<int> ids)
       {
           return ids.Select(id => repository.GetById(id)).Where(e => e != null);
       }
   }
   ```

### Situations où il vaut mieux éviter les méthodes d'extension

3. **Pour remplacer des comportements existants**
   ```csharp
   // Évitez de créer une extension qui donne l'impression de modifier le comportement standard
   public static class StringExtensions
   {
       // Confusion: modifie subtilement le comportement de ToUpper
       public static string ToUpper(this string text, bool preserveAccents)
       {
           if (!preserveAccents)
               return text.ToUpper(); // Comportement standard

           // Autre implémentation qui préserve les accents
           // Cela peut créer des confusions pour les autres développeurs
       }
   }
   ```

4. **Pour des opérations lourdes ou complexes**
   ```csharp
   // Évitez des extensions qui cachent des opérations coûteuses
   public static class ImageExtensions
   {
       // Cette extension cache une opération potentiellement très coûteuse
       // derrière une méthode qui semble simple
       public static Image Redimensionner(this Image image, int largeur, int hauteur)
       {
           // Opération lourde de traitement d'image...
           return nouvellImage;
       }
   }
   ```

5. **Pour des fonctionnalités qui modifient l'état global**
   ```csharp
   // Les extensions ne devraient pas avoir d'effets secondaires inattendus
   public static class ConfigurationExtensions
   {
       // Extension problématique qui modifie l'état global
       public static T ChargerConfiguration<T>(this T objet) where T : class
       {
           // Charge des configurations globales et modifie potentiellement
           // des variables statiques ou d'autres états globaux
           return objet;
       }
   }
   ```

### Cas d'étude : Création d'une bibliothèque d'extensions utiles

Voyons comment appliquer les bonnes pratiques pour créer une petite bibliothèque d'extensions utiles.

#### Structure de la bibliothèque

```csharp
namespace UtilExtensions
{
    // Extensions pour les chaînes de caractères
    public static class StringExtensions
    {
        // Extensions ici...
    }

    // Extensions pour les collections
    public static class CollectionExtensions
    {
        // Extensions ici...
    }

    // Extensions pour travailler avec les dates
    public static class DateTimeExtensions
    {
        // Extensions ici...
    }

    // Extensions pour le traitement numérique
    public static class NumericExtensions
    {
        // Extensions ici...
    }
}
```

#### Implémentation de StringExtensions

```csharp
using System;
using System.Globalization;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;

namespace UtilExtensions
{
    /// <summary>
    /// Fournit des méthodes d'extension pour les chaînes de caractères.
    /// </summary>
    public static class StringExtensions
    {
        /// <summary>
        /// Vérifie si la chaîne est nulle ou vide ou ne contient que des espaces.
        /// </summary>
        public static bool EstVide(this string texte)
        {
            return string.IsNullOrWhiteSpace(texte);
        }

        /// <summary>
        /// Tronque la chaîne à une longueur maximale spécifiée.
        /// </summary>
        /// <param name="texte">La chaîne à tronquer.</param>
        /// <param name="longueurMax">Longueur maximale.</param>
        /// <param name="suffixe">Suffixe à ajouter si la chaîne est tronquée.</param>
        public static string Tronquer(this string texte, int longueurMax, string suffixe = "...")
        {
            if (texte == null)
                return null;

            if (texte.Length <= longueurMax)
                return texte;

            return texte.Substring(0, longueurMax) + suffixe;
        }

        /// <summary>
        /// Supprime les accents d'une chaîne.
        /// </summary>
        public static string SupprimerAccents(this string texte)
        {
            if (texte == null)
                return null;

            // Décomposer les caractères accentués
            string texteNormalise = texte.Normalize(NormalizationForm.FormD);

            // Supprimer les marques diacritiques
            var resultat = new StringBuilder();
            foreach (char c in texteNormalise)
            {
                UnicodeCategory cat = CharUnicodeInfo.GetUnicodeCategory(c);
                if (cat != UnicodeCategory.NonSpacingMark)
                {
                    resultat.Append(c);
                }
            }

            return resultat.ToString().Normalize(NormalizationForm.FormC);
        }

        /// <summary>
        /// Convertit la première lettre d'une chaîne en majuscule.
        /// </summary>
        public static string PremiereLettreMajuscule(this string texte)
        {
            if (texte.EstVide())
                return texte;

            return char.ToUpper(texte[0]) + texte.Substring(1);
        }

        /// <summary>
        /// Convertit une chaîne en "snake_case".
        /// </summary>
        public static string EnSnakeCase(this string texte)
        {
            if (texte.EstVide())
                return texte;

            // Ajouter des underscores avant chaque majuscule et convertir en minuscules
            var resultat = Regex.Replace(texte, @"([a-z])([A-Z])", "$1_$2").ToLower();

            // Remplacer les espaces et autres caractères non alphanumériques par des underscores
            resultat = Regex.Replace(resultat, @"[\s\W]+", "_");

            return resultat;
        }

        /// <summary>
        /// Vérifie si une chaîne est une adresse email valide.
        /// </summary>
        public static bool EstEmailValide(this string email)
        {
            if (email.EstVide())
                return false;

            // Expression régulière simplifiée pour validation d'email
            // Note: Dans un environnement de production, utilisez une validation plus robuste
            var regex = new Regex(@"^[^@\s]+@[^@\s]+\.[^@\s]+$");
            return regex.IsMatch(email);
        }
    }
}
```

#### Implémentation de CollectionExtensions

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace UtilExtensions
{
    /// <summary>
    /// Fournit des méthodes d'extension pour les collections.
    /// </summary>
    public static class CollectionExtensions
    {
        /// <summary>
        /// Vérifie si une collection est vide ou nulle.
        /// </summary>
        public static bool EstVideOuNull<T>(this IEnumerable<T> collection)
        {
            return collection == null || !collection.Any();
        }

        /// <summary>
        /// Retourne un élément aléatoire de la collection.
        /// </summary>
        /// <returns>Un élément aléatoire ou la valeur par défaut si la collection est vide.</returns>
        public static T ElementAleatoire<T>(this IEnumerable<T> collection)
        {
            if (collection.EstVideOuNull())
                return default;

            var liste = collection as IList<T> ?? collection.ToList();
            Random rand = new Random();
            return liste[rand.Next(0, liste.Count)];
        }

        /// <summary>
        /// Divise une collection en plusieurs partitions de taille égale.
        /// </summary>
        /// <param name="collection">La collection à diviser.</param>
        /// <param name="taille">Taille de chaque partition.</param>
        /// <returns>Une séquence de partitions.</returns>
        public static IEnumerable<IEnumerable<T>> Partitionner<T>(
            this IEnumerable<T> collection, int taille)
        {
            if (collection == null)
                throw new ArgumentNullException(nameof(collection));

            if (taille <= 0)
                throw new ArgumentException("La taille doit être positive", nameof(taille));

            var partition = new List<T>(taille);
            foreach (var item in collection)
            {
                partition.Add(item);
                if (partition.Count == taille)
                {
                    yield return partition;
                    partition = new List<T>(taille);
                }
            }

            // Retourner la dernière partition même si elle n'est pas complète
            if (partition.Count > 0)
                yield return partition;
        }

        /// <summary>
        /// Applique une action à chaque élément de la collection.
        /// </summary>
        public static void PourChaque<T>(this IEnumerable<T> collection, Action<T> action)
        {
            if (collection == null)
                throw new ArgumentNullException(nameof(collection));

            if (action == null)
                throw new ArgumentNullException(nameof(action));

            foreach (var item in collection)
            {
                action(item);
            }
        }

        /// <summary>
        /// Convertit une collection en chaîne de caractères avec un séparateur personnalisé.
        /// </summary>
        public static string JoindreAvec<T>(
            this IEnumerable<T> collection, string separateur = ", ")
        {
            if (collection == null)
                return string.Empty;

            return string.Join(separateur, collection);
        }

        /// <summary>
        /// Ajoute plusieurs éléments à une collection.
        /// </summary>
        public static void AjouterPlusieurs<T>(
            this ICollection<T> collection, params T[] elements)
        {
            if (collection == null)
                throw new ArgumentNullException(nameof(collection));

            if (elements == null)
                return;

            foreach (var item in elements)
            {
                collection.Add(item);
            }
        }
    }
}
```

#### Implémentation de DateTimeExtensions

```csharp
using System;
using System.Collections.Generic;
using System.Globalization;

namespace UtilExtensions
{
    /// <summary>
    /// Fournit des méthodes d'extension pour les dates et heures.
    /// </summary>
    public static class DateTimeExtensions
    {
        /// <summary>
        /// Vérifie si une date est aujourd'hui.
        /// </summary>
        public static bool EstAujourdhui(this DateTime date)
        {
            return date.Date == DateTime.Today;
        }

        /// <summary>
        /// Retourne l'âge en années calculé à partir d'une date de naissance.
        /// </summary>
        public static int CalculerAge(this DateTime dateNaissance)
        {
            DateTime aujourdhui = DateTime.Today;
            int age = aujourdhui.Year - dateNaissance.Year;

            // Ajustement si l'anniversaire n'est pas encore passé cette année
            if (dateNaissance > aujourdhui.AddYears(-age))
                age--;

            return age;
        }

        /// <summary>
        /// Retourne le début du mois pour la date donnée.
        /// </summary>
        public static DateTime DebutDuMois(this DateTime date)
        {
            return new DateTime(date.Year, date.Month, 1);
        }

        /// <summary>
        /// Retourne la fin du mois pour la date donnée.
        /// </summary>
        public static DateTime FinDuMois(this DateTime date)
        {
            return date.DebutDuMois().AddMonths(1).AddDays(-1);
        }

        /// <summary>
        /// Retourne le début de la semaine pour la date donnée.
        /// </summary>
        /// <param name="date">La date.</param>
        /// <param name="premierJour">Premier jour de la semaine (lundi par défaut).</param>
        public static DateTime DebutDeLaSemaine(
            this DateTime date, DayOfWeek premierJour = DayOfWeek.Monday)
        {
            int diff = (7 + (date.DayOfWeek - premierJour)) % 7;
            return date.AddDays(-diff).Date;
        }

        /// <summary>
        /// Formate une date en utilisant un format français standard.
        /// </summary>
        public static string FormaterFrancais(
            this DateTime date, bool inclureHeure = false)
        {
            string format = "dd/MM/yyyy";
            if (inclureHeure)
                format += " HH:mm";

            return date.ToString(format, CultureInfo.GetCultureInfo("fr-FR"));
        }

        /// <summary>
        /// Retourne toutes les dates entre deux dates.
        /// </summary>
        public static IEnumerable<DateTime> JusquA(
            this DateTime dateDebut, DateTime dateFin)
        {
            if (dateDebut > dateFin)
                throw new ArgumentException(
                    "La date de début doit être antérieure à la date de fin.");

            for (var date = dateDebut; date <= dateFin; date = date.AddDays(1))
            {
                yield return date;
            }
        }

        /// <summary>
        /// Vérifie si une date est un jour de week-end.
        /// </summary>
        public static bool EstWeekEnd(this DateTime date)
        {
            return date.DayOfWeek == DayOfWeek.Saturday ||
                   date.DayOfWeek == DayOfWeek.Sunday;
        }
    }
}
```

#### Implémentation de NumericExtensions

```csharp
using System;

namespace UtilExtensions
{
    /// <summary>
    /// Fournit des méthodes d'extension pour les types numériques.
    /// </summary>
    public static class NumericExtensions
    {
        /// <summary>
        /// Vérifie si un nombre est compris entre deux valeurs (incluses).
        /// </summary>
        public static bool EstEntre<T>(this T valeur, T min, T max)
            where T : IComparable<T>
        {
            return valeur.CompareTo(min) >= 0 && valeur.CompareTo(max) <= 0;
        }

        /// <summary>
        /// Limite une valeur à un intervalle spécifié.
        /// </summary>
        public static T Limiter<T>(this T valeur, T min, T max)
            where T : IComparable<T>
        {
            if (valeur.CompareTo(min) < 0)
                return min;

            if (valeur.CompareTo(max) > 0)
                return max;

            return valeur;
        }

        /// <summary>
        /// Convertit un nombre décimal en pourcentage formaté.
        /// </summary>
        public static string EnPourcentage(
            this double valeur, int decimales = 2, bool inclureSymbole = true)
        {
            string format = $"F{decimales}";
            string resultat = (valeur * 100).ToString(format);

            if (inclureSymbole)
                resultat += " %";

            return resultat;
        }

        /// <summary>
        /// Arrondit un nombre double à un nombre spécifié de décimales.
        /// </summary>
        public static double ArrondiA(this double valeur, int decimales)
        {
            return Math.Round(valeur, decimales);
        }

        /// <summary>
        /// Vérifie si un nombre entier est pair.
        /// </summary>
        public static bool EstPair(this int nombre)
        {
            return nombre % 2 == 0;
        }

        /// <summary>
        /// Vérifie si un nombre entier est impair.
        /// </summary>
        public static bool EstImpair(this int nombre)
        {
            return nombre % 2 != 0;
        }

        /// <summary>
        /// Vérifie si un nombre est premier.
        /// </summary>
        public static bool EstPremier(this int nombre)
        {
            if (nombre <= 1) return false;
            if (nombre == 2) return true;
            if (nombre % 2 == 0) return false;

            var limite = (int)Math.Sqrt(nombre);
            for (int i = 3; i <= limite; i += 2)
            {
                if (nombre % i == 0) return false;
            }

            return true;
        }
    }
}
```

### Utilisation de notre bibliothèque d'extensions

Voici comment utiliser cette bibliothèque d'extensions dans une application :

```csharp
using System;
using System.Collections.Generic;
using UtilExtensions;

public class Program
{
    public static void Main()
    {
        DemonstrationsChaines();
        DemonstrationsCollections();
        DemonstrationsDates();
        DemonstrationsNombres();
    }

    static void DemonstrationsChaines()
    {
        Console.WriteLine("=== Démonstration des extensions pour les chaînes ===");

        string texte = "Bonjour à tous les développeurs C#";
        Console.WriteLine($"Texte original: {texte}");
        Console.WriteLine($"Tronqué à 10 caractères: {texte.Tronquer(10)}");
        Console.WriteLine($"Sans accents: {texte.SupprimerAccents()}");
        Console.WriteLine($"En snake_case: {texte.EnSnakeCase()}");

        string email = "contact@exemple.com";
        Console.WriteLine($"'{email}' est un email valide: {email.EstEmailValide()}");

        string emailInvalide = "contact@exemple";
        Console.WriteLine($"'{emailInvalide}' est un email valide: {emailInvalide.EstEmailValide()}");

        Console.WriteLine();
    }

    static void DemonstrationsCollections()
    {
        Console.WriteLine("=== Démonstration des extensions pour les collections ===");

        List<string> fruits = new List<string> { "Pomme", "Banane", "Orange", "Fraise", "Kiwi" };
        Console.WriteLine($"Fruits: {fruits.JoindreAvec()}");
        Console.WriteLine($"Un fruit aléatoire: {fruits.ElementAleatoire()}");

        Console.WriteLine("Fruits partitionnés en groupes de 2:");
        var partitions = fruits.Partitionner(2).ToList();
        for (int i = 0; i < partitions.Count; i++)
        {
            Console.WriteLine($"  Groupe {i+1}: {partitions[i].JoindreAvec()}");
        }

        List<int> nombres = new List<int>();
        nombres.AjouterPlusieurs(1, 2, 3, 4, 5);
        Console.WriteLine($"Nombres ajoutés: {nombres.JoindreAvec()}");

        Console.WriteLine();
    }

    static void DemonstrationsDates()
    {
        Console.WriteLine("=== Démonstration des extensions pour les dates ===");

        DateTime maintenant = DateTime.Now;
        DateTime anniversaire = new DateTime(1990, 5, 15);

        Console.WriteLine($"Date actuelle: {maintenant}");
        Console.WriteLine($"Format français: {maintenant.FormaterFrancais(true)}");
        Console.WriteLine($"Aujourd'hui est un weekend: {maintenant.EstWeekEnd()}");
        Console.WriteLine($"Début du mois: {maintenant.DebutDuMois().FormaterFrancais()}");
        Console.WriteLine($"Fin du mois: {maintenant.FinDuMois().FormaterFrancais()}");
        Console.WriteLine($"Début de la semaine: {maintenant.DebutDeLaSemaine().FormaterFrancais()}");
        Console.WriteLine($"Âge calculé: {anniversaire.CalculerAge()} ans");

        Console.WriteLine();
    }

    static void DemonstrationsNombres()
    {
        Console.WriteLine("=== Démonstration des extensions pour les nombres ===");

        int nombre = 17;
        Console.WriteLine($"{nombre} est entre 10 et 20: {nombre.EstEntre(10, 20)}");
        Console.WriteLine($"{nombre} est pair: {nombre.EstPair()}");
        Console.WriteLine($"{nombre} est premier: {nombre.EstPremier()}");

        double valeur = 0.7523;
        Console.WriteLine($"{valeur} en pourcentage: {valeur.EnPourcentage()}");
        Console.WriteLine($"{valeur} arrondi à 1 décimale: {valeur.ArrondiA(1)}");

        int valeurALimiter = 150;
        Console.WriteLine($"{valeurALimiter} limité entre 0 et 100: {valeurALimiter.Limiter(0, 100)}");

        Console.WriteLine();
    }
}
```

### Conseils finaux pour travailler avec les méthodes d'extension

#### 1. Suivez les conventions de nommage .NET

Utilisez des noms clairs qui correspondent aux conventions .NET :

```csharp
// Bon: Suit les conventions .NET
public static bool IsNullOrEmpty<T>(this IEnumerable<T> source)

// Mauvais: Ne suit pas les conventions
public static bool est_vide<T>(this IEnumerable<T> source)
```

#### 2. Documentez le comportement avec des attributs de contrat

```csharp
using System.Diagnostics.Contracts;

public static class ContractExtensions
{
    [Pure] // Indique que la méthode ne modifie pas l'état
    public static bool EstPremier(this int nombre)
    {
        Contract.Requires(nombre >= 0);
        // Implémentation...
    }
}
```

#### 3. Utilisez des espaces de noms pertinents

Organisez vos extensions dans des espaces de noms pertinents pour faciliter leur importation sélective :

```csharp
namespace MaCompagnie.Extensions.Data { /* Extensions liées aux données */ }
namespace MaCompagnie.Extensions.Web { /* Extensions liées au web */ }
namespace MaCompagnie.Extensions.IO { /* Extensions liées aux fichiers */ }
```

#### 4. Créez des tests unitaires pour vos extensions

Les méthodes d'extension sont parfaites pour les tests unitaires car elles sont généralement pures et indépendantes :

```csharp
[TestClass]
public class StringExtensionsTests
{
    [TestMethod]
    public void EstPalindrome_AvecPalindromeSimple_RetourneVrai()
    {
        // Arrange
        string palindrome = "kayak";

        // Act
        bool resultat = palindrome.EstPalindrome();

        // Assert
        Assert.IsTrue(resultat);
    }

    [TestMethod]
    public void EstPalindrome_AvecNonPalindrome_RetourneFaux()
    {
        // Arrange
        string nonPalindrome = "bonjour";

        // Act
        bool resultat = nonPalindrome.EstPalindrome();

        // Assert
        Assert.IsFalse(resultat);
    }

    // Autres tests...
}
```

#### 5. Ne sur-utilisez pas cette fonctionnalité

N'ajoutez pas des méthodes d'extension juste parce que vous le pouvez. Chaque extension doit apporter une valeur significative.

#### 6. Pensez à la maintenabilité à long terme

Réfléchissez à comment vos extensions évolueront dans le temps et comment elles interagiront avec les futures versions de .NET.

### Conclusion

Les méthodes d'extension sont un outil puissant en C# qui vous permet d'étendre des types existants sans les modifier directement. Elles sont particulièrement utiles pour :

- Ajouter des fonctionnalités aux types du framework .NET
- Créer des API fluides et intuitives
- Regrouper des opérations courantes en méthodes réutilisables
- Implémenter des patterns comme LINQ

En suivant les bonnes pratiques décrites dans ce tutoriel, vous pourrez créer des méthodes d'extension qui rendent votre code plus lisible, plus maintenable et plus expressif. Cependant, comme toute fonctionnalité puissante, il faut l'utiliser avec discernement pour éviter de créer du code confus ou difficile à maintenir.

Gardez toujours à l'esprit que les meilleures méthodes d'extension sont celles qui semblent faire naturellement partie du type qu'elles étendent, comme si elles avaient toujours été là.

⏭️ 4.6. [Méthodes locales (C# 7+)](/04-methodes-et-fonctions/4-6-methodes-locales-csharp-7-plus.md)
