# 17.4. Optimisation des requêtes LINQ en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

LINQ (Language Integrated Query) est l'une des fonctionnalités les plus puissantes et expressives de C#. Elle vous permet d'écrire des requêtes élégantes pour manipuler des données de diverses sources. Cependant, mal utilisée, LINQ peut entraîner des problèmes de performance significatifs. Ce chapitre vous expliquera comment optimiser vos requêtes LINQ pour obtenir les meilleures performances possible, tout en conservant un code clair et maintenable.

## 17.4.1. Évaluation différée vs immédiate

Un aspect fondamental de LINQ est son modèle d'exécution. Comprendre comment et quand vos requêtes sont réellement exécutées est essentiel pour optimiser leurs performances.

### Évaluation différée (Lazy Evaluation)

L'évaluation différée signifie que l'exécution de la requête est retardée jusqu'à ce que vous ayez réellement besoin des résultats. La plupart des méthodes LINQ utilisent cette approche.

```csharp
// Requête avec évaluation différée
var nombres = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Cette ligne ne fait que définir la requête, elle n'est pas exécutée
var nombresPairs = nombres.Where(n => n % 2 == 0);

// La requête est exécutée seulement maintenant, quand nous itérons sur les résultats
Console.WriteLine("Nombres pairs:");
foreach (var nombre in nombresPairs)
{
    Console.WriteLine(nombre);
}
```

#### Avantages de l'évaluation différée

1. **Efficacité** : La requête s'exécute uniquement lorsque vous avez besoin des résultats.
2. **Composition** : Vous pouvez construire des requêtes complexes en les composant, sans exécution intermédiaire.
3. **Données actualisées** : La requête s'exécute sur les données les plus récentes au moment de l'itération.

#### Méthodes LINQ qui utilisent l'évaluation différée

La plupart des méthodes d'extension LINQ retournent un `IEnumerable<T>` et utilisent l'évaluation différée :
- `Where`
- `Select`
- `OrderBy` / `OrderByDescending`
- `Skip` / `Take`
- `Join`
- `GroupBy`

### Évaluation immédiate (Eager Evaluation)

À l'inverse, l'évaluation immédiate signifie que la requête est exécutée dès qu'elle est définie, et les résultats sont stockés en mémoire.

```csharp
var nombres = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Ces méthodes forcent l'exécution immédiate de la requête
var nombresPairsList = nombres.Where(n => n % 2 == 0).ToList();  // Crée une nouvelle liste
var nombresPairsArray = nombres.Where(n => n % 2 == 0).ToArray();  // Crée un nouveau tableau
var comptage = nombres.Count(n => n % 2 == 0);  // Retourne simplement un nombre
```

#### Méthodes LINQ qui forcent l'évaluation immédiate

Ces méthodes exécutent la requête immédiatement :
- `ToList()` / `ToArray()` / `ToDictionary()` / `ToHashSet()`
- `Count()` / `LongCount()`
- `First()` / `FirstOrDefault()` / `Last()` / `LastOrDefault()`
- `Single()` / `SingleOrDefault()`
- `Min()` / `Max()` / `Sum()` / `Average()`
- `Any()` / `All()`
- `Contains()`

### Problèmes courants liés à l'évaluation différée

L'évaluation différée peut causer des problèmes si vous n'êtes pas vigilant :

#### 1. Exécutions multiples inattendues

```csharp
var nombres = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var requête = nombres.Where(n => {
    Console.WriteLine($"Évaluation pour {n}");  // Effet secondaire pour démonstration
    return n % 2 == 0;
});

// Première exécution
Console.WriteLine("Première itération:");
foreach (var n in requête)
    Console.WriteLine($"Trouvé: {n}");

// Deuxième exécution - la requête s'exécute à nouveau!
Console.WriteLine("Deuxième itération:");
foreach (var n in requête)
    Console.WriteLine($"Trouvé: {n}");
```

Dans cet exemple, le message "Évaluation pour [n]" s'affichera deux fois pour chaque nombre, car la requête est exécutée à chaque itération.

#### 2. Modification de la source de données

```csharp
var nombres = new List<int> { 1, 2, 3, 4, 5 };
var requête = nombres.Where(n => n > 2);

// Première itération
Console.WriteLine("Résultats initiaux:");
foreach (var n in requête)
    Console.WriteLine(n);  // Affiche 3, 4, 5

// Modification de la source
nombres.Add(6);
nombres.Add(7);

// Deuxième itération - inclut les nouveaux nombres!
Console.WriteLine("Après modification:");
foreach (var n in requête)
    Console.WriteLine(n);  // Affiche 3, 4, 5, 6, 7
```

#### 3. La source n'est plus disponible

```csharp
IEnumerable<string> ObtenirLignes()
{
    using (var reader = new StreamReader("fichier.txt"))
    {
        while (!reader.EndOfStream)
        {
            yield return reader.ReadLine();
        }
    }
    // Le fichier est fermé ici, après le bloc using
}

// Cette ligne semble inoffensive, mais elle est dangereuse
var lignes = ObtenirLignes();

// ERREUR: Quand on essaie d'accéder aux lignes, le fichier est déjà fermé!
foreach (var ligne in lignes)
{
    Console.WriteLine(ligne);
}
```

### Choisir entre évaluation différée et immédiate

Voici quelques lignes directrices pour décider quand utiliser chaque type d'évaluation :

**Utilisez l'évaluation différée quand :**
- Vous voulez composer plusieurs opérations sans exécution intermédiaire
- Vous travaillez avec de très grandes collections et n'avez besoin que d'une partie des résultats
- Vous utilisez LINQ pour définir une logique réutilisable

**Utilisez l'évaluation immédiate quand :**
- Vous allez utiliser les résultats plusieurs fois
- La source de données peut changer et vous voulez un "instantané"
- La source de données est éphémère (comme un StreamReader)
- Vous voulez capturer les exceptions au moment de la définition de la requête plutôt que lors de l'itération

### Exemple pratique : calcul de statistiques

```csharp
public class StatistiquesVentes
{
    public void AnalyserVentes(List<Vente> ventes)
    {
        // Mauvaise approche: requêtes similaires exécutées plusieurs fois
        Console.WriteLine($"Ventes totales: {ventes.Sum(v => v.Montant)}");
        Console.WriteLine($"Vente moyenne: {ventes.Average(v => v.Montant)}");
        Console.WriteLine($"Vente max: {ventes.Max(v => v.Montant)}");
        Console.WriteLine($"Vente min: {ventes.Min(v => v.Montant)}");

        // Meilleure approche: calculer tout en une seule passe
        var stats = ventes.Aggregate(
            new { Somme = 0m, Min = decimal.MaxValue, Max = decimal.MinValue, Compte = 0 },
            (acc, vente) => new
            {
                Somme = acc.Somme + vente.Montant,
                Min = Math.Min(acc.Min, vente.Montant),
                Max = Math.Max(acc.Max, vente.Montant),
                Compte = acc.Compte + 1
            }
        );

        Console.WriteLine($"Ventes totales: {stats.Somme}");
        Console.WriteLine($"Vente moyenne: {stats.Somme / stats.Compte}");
        Console.WriteLine($"Vente max: {stats.Max}");
        Console.WriteLine($"Vente min: {stats.Min}");
    }
}

public class Vente
{
    public decimal Montant { get; set; }
    public DateTime Date { get; set; }
    public string Produit { get; set; }
}
```

## 17.4.2. Analyse de performances

Avant d'optimiser vos requêtes LINQ, il est essentiel de comprendre où se situent les problèmes de performance. Plusieurs outils et techniques peuvent vous aider à analyser vos requêtes.

### Mesure du temps d'exécution

La méthode la plus simple pour analyser les performances est de mesurer le temps d'exécution à l'aide de la classe `Stopwatch`.

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;

public void ComparePerformance()
{
    List<int> nombres = Enumerable.Range(1, 1_000_000).ToList();

    // Mesurer l'approche 1
    Stopwatch sw = Stopwatch.StartNew();
    var résultat1 = Approche1(nombres);
    sw.Stop();
    Console.WriteLine($"Approche 1: {sw.ElapsedMilliseconds}ms");

    // Mesurer l'approche 2
    sw.Restart();
    var résultat2 = Approche2(nombres);
    sw.Stop();
    Console.WriteLine($"Approche 2: {sw.ElapsedMilliseconds}ms");
}

private List<int> Approche1(List<int> nombres)
{
    return nombres.Where(n => n % 2 == 0)
                 .OrderBy(n => n)
                 .Select(n => n * 2)
                 .ToList();
}

private List<int> Approche2(List<int> nombres)
{
    return nombres.Where(n => n % 2 == 0)
                 .Select(n => n * 2)
                 .OrderBy(n => n)
                 .ToList();
}
```

### Utilisation de BenchmarkDotNet

Pour des mesures plus précises, utilisez la bibliothèque [BenchmarkDotNet](https://benchmarkdotnet.org/), qui vous aide à créer des benchmarks rigoureux et fiables.

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;
using System.Collections.Generic;
using System.Linq;

[MemoryDiagnoser]
public class LinqBenchmarks
{
    private List<int> _nombres;

    [GlobalSetup]
    public void Setup()
    {
        _nombres = Enumerable.Range(1, 1_000_000).ToList();
    }

    [Benchmark]
    public List<int> Approche1()
    {
        return _nombres.Where(n => n % 2 == 0)
                      .OrderBy(n => n)
                      .Select(n => n * 2)
                      .ToList();
    }

    [Benchmark]
    public List<int> Approche2()
    {
        return _nombres.Where(n => n % 2 == 0)
                      .Select(n => n * 2)
                      .OrderBy(n => n)
                      .ToList();
    }
}

public class Program
{
    public static void Main()
    {
        var summary = BenchmarkRunner.Run<LinqBenchmarks>();
    }
}
```

### Profilage avec les outils Visual Studio

Visual Studio offre plusieurs outils de profilage qui peuvent vous aider à identifier les goulots d'étranglement :

1. **Performance Profiler** : Analysez l'utilisation du CPU, de la mémoire et plus encore.
2. **Memory Usage** : Identifiez les fuites mémoire et l'allocation excessive.
3. **IntelliTrace** : Explorez l'historique d'exécution de votre application.

### Analyse de requêtes avec LinqPad

[LinqPad](https://www.linqpad.net/) est un outil excellent pour tester et analyser vos requêtes LINQ. Il inclut des fonctionnalités pour :
- Visualiser le plan d'exécution des requêtes
- Mesurer le temps d'exécution
- Inspecter les résultats intermédiaires

### Décomposition des requêtes pour le débogage

Il peut être utile de décomposer les requêtes complexes pour identifier quelles parties prennent le plus de temps.

```csharp
// Requête complexe d'origine
var résultat = collection
    .Where(x => x.Propriété > 100)
    .OrderBy(x => x.AutrePropriété)
    .Select(x => new { x.Id, Total = x.Valeurs.Sum() })
    .ToList();

// Décomposée pour analyse
var étape1 = collection.Where(x => x.Propriété > 100);
Console.WriteLine($"Après Where: {étape1.Count()} éléments");

var étape2 = étape1.OrderBy(x => x.AutrePropriété);
Console.WriteLine("OrderBy terminé");

var étape3 = étape2.Select(x => new { x.Id, Total = x.Valeurs.Sum() });
Console.WriteLine("Select terminé");

var résultatFinal = étape3.ToList();
Console.WriteLine($"Résultat final: {résultatFinal.Count} éléments");
```

## 17.4.3. Stratégies d'optimisation

Après avoir identifié les goulots d'étranglement dans vos requêtes LINQ, vous pouvez appliquer diverses stratégies d'optimisation.

### 1. Filtrer d'abord, puis transformer

L'une des optimisations les plus simples et efficaces est de filtrer les données le plus tôt possible pour réduire la quantité de données à traiter dans les étapes suivantes.

```csharp
// Moins efficace: transformation avant filtrage
var résultat1 = collection
    .Select(x => TransformationCoûteuse(x))
    .Where(x => x.Valeur > 100)
    .ToList();

// Plus efficace: filtrage avant transformation
var résultat2 = collection
    .Where(x => EstimationRapide(x) > 100)
    .Select(x => TransformationCoûteuse(x))
    .ToList();
```

### 2. Optimiser l'ordre des opérations

L'ordre des opérations dans une chaîne LINQ peut avoir un impact significatif sur les performances.

```csharp
// Moins efficace: tri avant filtrage
var résultat1 = collection
    .OrderBy(x => x.Propriété)
    .Where(x => x.AutrePropriété > 100)
    .ToList();

// Plus efficace: filtrage avant tri
var résultat2 = collection
    .Where(x => x.AutrePropriété > 100)
    .OrderBy(x => x.Propriété)
    .ToList();
```

#### Règles générales pour l'ordre des opérations

Voici un ordre généralement efficace pour les opérations LINQ :

1. **Filtrage** (`Where`) : Réduisez d'abord le nombre d'éléments
2. **Projection distincte** (`Select`, lorsqu'elle réduit la taille des objets)
3. **Jointure** (`Join`, `GroupJoin`)
4. **Tri** (`OrderBy`, `OrderByDescending`, `ThenBy`, etc.)
5. **Regroupement** (`GroupBy`)
6. **Projection finale** (`Select`, pour la forme finale des résultats)
7. **Agrégation/quantification** (`Sum`, `Count`, `Any`, etc.)

### 3. Éviter les requêtes imbriquées non optimisées

Les requêtes imbriquées peuvent causer des problèmes de performance significatifs si elles ne sont pas optimisées.

```csharp
// Inefficace: requête imbriquée exécutée pour chaque client
var clientsAvecCommandesChères = clients
    .Where(client => client.Commandes.Any(commande => commande.Total > 1000))
    .ToList();

// Plus efficace: préfiltrer les commandes coûteuses
var commandesChères = commandes.Where(c => c.Total > 1000)
                              .Select(c => c.ClientId)
                              .Distinct()
                              .ToHashSet();

var clientsAvecCommandesChères = clients
    .Where(client => commandesChères.Contains(client.Id))
    .ToList();
```

### 4. Utiliser les surcharges optimisées

Certaines méthodes LINQ ont des surcharges qui peuvent être plus efficaces dans des situations spécifiques.

```csharp
var nombres = Enumerable.Range(1, 1_000_000).ToList();

// Moins efficace: deux passages sur la collection
bool contientPair = nombres.Any(n => n % 2 == 0);
int premierPair = nombres.First(n => n % 2 == 0);

// Plus efficace: un seul passage, s'arrête au premier élément trouvé
bool contientPair2 = nombres.Any(n => n % 2 == 0);
int premierPair2 = nombres.FirstOrDefault(n => n % 2 == 0);

// Inefficace pour de grandes collections déjà en mémoire
int compte = nombres.Count(n => n % 2 == 0);

// Plus efficace pour les collections qui implémentent ICollection<T>
int taille = nombres.Count;  // O(1) pour List<T>
```

### 5. Parallélisation avec PLINQ

Pour des opérations intensives sur de grandes collections, PLINQ peut offrir des gains de performance significatifs en utilisant plusieurs threads.

```csharp
// Séquentiel
var résultatSéquentiel = collection
    .Where(x => TestCoûteux(x))
    .Select(x => TransformationCoûteuse(x))
    .ToList();

// Parallèle
var résultatParallèle = collection
    .AsParallel()
    .Where(x => TestCoûteux(x))
    .Select(x => TransformationCoûteuse(x))
    .ToList();
```

> **Attention** : PLINQ n'est pas toujours plus rapide. Pour de petites collections ou des opérations simples, les frais généraux de la parallélisation peuvent dépasser les gains. Mesurez toujours les performances réelles.

### 6. Utiliser les index et les structures de données appropriées

Le choix de la structure de données sous-jacente peut avoir un impact énorme sur les performances des requêtes LINQ.

```csharp
// Recherche dans une liste: O(n)
var liste = Enumerable.Range(1, 1_000_000).ToList();
bool contient = liste.Contains(500000);  // Lent

// Recherche dans un HashSet: O(1)
var hashSet = new HashSet<int>(Enumerable.Range(1, 1_000_000));
bool contient2 = hashSet.Contains(500000);  // Très rapide

// Recherche dans un dictionnaire: O(1)
var dictionnaire = Enumerable.Range(1, 1_000_000)
                           .ToDictionary(n => n, n => n * n);
bool contient3 = dictionnaire.ContainsKey(500000);  // Très rapide
```

#### Structures de données recommandées selon l'opération

| Opération principale | Structure recommandée |
|----------------------|------------------------|
| Recherche par clé    | `Dictionary<K,V>`, `HashSet<T>` |
| Tri fréquent         | `SortedList<K,V>`, `SortedDictionary<K,V>` |
| Ajout/suppression fréquente | `LinkedList<T>` |
| Accès par index      | `List<T>`, `T[]` |
| Files d'attente      | `Queue<T>` |
| Piles                | `Stack<T>` |

## 17.4.4. Minimisation des requêtes

Une source commune de problèmes de performance est l'exécution répétée ou inutile de requêtes. Voici comment minimiser le nombre de requêtes que vous exécutez.

### 1. Matérialiser les résultats intermédiaires lorsque nécessaire

Si vous utilisez les résultats d'une requête plusieurs fois, matérialisez-les avec `ToList()`, `ToArray()`, etc.

```csharp
// Inefficace: requête exécutée deux fois
var requête = collection.Where(x => TestCoûteux(x));
var nombre = requête.Count();
var premier = requête.FirstOrDefault();

// Efficace: requête exécutée une seule fois
var résultats = collection.Where(x => TestCoûteux(x)).ToList();
var nombre = résultats.Count;
var premier = résultats.FirstOrDefault();
```

### 2. Utiliser une seule requête pour plusieurs opérations

Combinez plusieurs opérations dans une seule requête lorsque c'est possible.

```csharp
// Inefficace: trois requêtes distinctes
var moyennes = étudiants.Average(e => e.Note);
var max = étudiants.Max(e => e.Note);
var min = étudiants.Min(e => e.Note);

// Efficace: une seule requête (passe unique sur la collection)
var stats = étudiants.Aggregate(
    new { Somme = 0.0, Min = double.MaxValue, Max = double.MinValue, Compte = 0 },
    (acc, étudiant) => new {
        Somme = acc.Somme + étudiant.Note,
        Min = Math.Min(acc.Min, étudiant.Note),
        Max = Math.Max(acc.Max, étudiant.Note),
        Compte = acc.Compte + 1
    }
);

var moyennes = stats.Somme / stats.Compte;
var max = stats.Max;
var min = stats.Min;
```

### 3. Éviter les requêtes dans les boucles

Exécuter des requêtes à l'intérieur de boucles peut entraîner un nombre exponentiel d'opérations.

```csharp
// Très inefficace: requête dans une boucle
foreach (var département in départements)
{
    var employés = tous_employés.Where(e => e.DépartementId == département.Id).ToList();
    // Traiter les employés...
}

// Beaucoup plus efficace: réorganiser les données avant la boucle
var employésParDépartement = tous_employés.GroupBy(e => e.DépartementId)
                                       .ToDictionary(g => g.Key, g => g.ToList());

foreach (var département in départements)
{
    if (employésParDépartement.TryGetValue(département.Id, out var employés))
    {
        // Traiter les employés...
    }
}
```

### 4. Mettre en cache les résultats de requêtes coûteuses

Si une requête coûteuse est appelée fréquemment avec les mêmes paramètres, envisagez de mettre en cache les résultats.

```csharp
// Sans mise en cache
public List<Produit> ObtenirProduitsByCatégorie(int catégorieId)
{
    return _context.Produits
                 .Where(p => p.CatégorieId == catégorieId)
                 .ToList();
}

// Avec mise en cache simple
private Dictionary<int, List<Produit>> _cacheProduits = new Dictionary<int, List<Produit>>();

public List<Produit> ObtenirProduitsByCatégorie(int catégorieId)
{
    if (_cacheProduits.TryGetValue(catégorieId, out var produits))
    {
        return produits;
    }

    var résultat = _context.Produits
                          .Where(p => p.CatégorieId == catégorieId)
                          .ToList();

    _cacheProduits[catégorieId] = résultat;
    return résultat;
}
```

Pour des scénarios plus complexes, envisagez d'utiliser `MemoryCache` ou d'autres solutions de mise en cache.

### 5. Utiliser la composition de requêtes plutôt que la concaténation de résultats

Composer des requêtes peut être plus efficace que d'exécuter plusieurs requêtes et de concaténer leurs résultats.

```csharp
// Moins efficace: deux requêtes et concaténation
var produitsRécents = produits.Where(p => p.DateCréation > DateTime.Now.AddDays(-30))
                            .ToList();
var produitsPopulaires = produits.Where(p => p.NombreVentes > 1000)
                               .ToList();
var produitsÀAfficher = produitsRécents.Concat(produitsPopulaires).ToList();

// Plus efficace: une seule requête composée
var produitsÀAfficher = produits.Where(p =>
                                p.DateCréation > DateTime.Now.AddDays(-30) ||
                                p.NombreVentes > 1000)
                              .ToList();
```

## 17.4.5. Projections et sélection

Les projections (`Select`) sont un moyen puissant de transformer les données, mais elles peuvent aussi avoir un impact significatif sur les performances si elles ne sont pas utilisées judicieusement.

### 1. Sélectionner uniquement ce dont vous avez besoin

Ne sélectionnez que les propriétés dont vous avez réellement besoin, surtout lorsque vous travaillez avec de grandes entités ou des données distantes.

```csharp
// Inefficace: sélectionne toutes les propriétés
var clients = _context.Clients
                    .Where(c => c.Pays == "France")
                    .ToList();

// Efficace: sélectionne uniquement les propriétés nécessaires
var clientsOptimisés = _context.Clients
                             .Where(c => c.Pays == "France")
                             .Select(c => new { c.Id, c.Nom, c.Email })
                             .ToList();
```

Cette approche est particulièrement importante avec Entity Framework ou d'autres ORM, où la sélection peut réduire considérablement la taille des requêtes SQL générées.

### 2. Éviter les projections inutiles

Les projections qui ne modifient pas vraiment les données n'ajoutent que des frais généraux.

```csharp
// Inutile: projection qui ne change rien
var nombres = Enumerable.Range(1, 100)
                      .Select(n => n)  // Projection inutile
                      .ToList();

// Mieux: pas de projection
var nombres = Enumerable.Range(1, 100).ToList();
```

### 3. Utiliser les projections pour réduire la quantité de données

Les projections sont idéales pour réduire la quantité de données que vous traitez.

```csharp
// Base de données avec des entités volumineuses
public class Client
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Adresse { get; set; }
    public string Ville { get; set; }
    public string Pays { get; set; }
    public string Email { get; set; }
    public string Téléphone { get; set; }
    public byte[] Photo { get; set; }  // Potentiellement volumineux
    public string HistoriqueComplet { get; set; }  // Potentiellement volumineux
    // ... nombreuses autres propriétés
}

// Inefficace pour l'affichage d'une liste simple
var tousLesClients = _context.Clients.ToList();

// Beaucoup plus efficace
var listeClients = _context.Clients
                         .Select(c => new ClientListeViewModel
                         {
                             Id = c.Id,
                             Nom = c.Nom,
                             Email = c.Email
                         })
                         .ToList();
```

### 4. Utiliser des projections pour éviter le chargement de relations

Avec les ORM comme Entity Framework, les projections peuvent vous aider à éviter le chargement de relations complètes quand vous n'avez besoin que de quelques propriétés.

```csharp
// Inefficace: charge toutes les commandes pour chaque client
var clients = _context.Clients
                    .Include(c => c.Commandes)  // Charge toutes les commandes
                    .ToList();

// Efficace: projette uniquement les données nécessaires
var clientsAvecNombreCommandes = _context.Clients
    .Select(c => new
    {
        c.Id,
        c.Nom,
        NombreCommandes = c.Commandes.Count,
        ValeurTotale = c.Commandes.Sum(co => co.Total)
    })
    .ToList();
```

### 5. Différer les transformations coûteuses

Si une transformation est coûteuse et que vous n'aurez pas besoin de tous les résultats, envisagez de la différer.

```csharp
// Inefficace si vous n'utilisez pas tous les résultats
var tousLesRésultats = collection
    .Select(x => TransformationCoûteuse(x))
    .ToList();

// Premières transformations exécutées uniquement si nécessaire
var requêteDifférée = collection.Select(x => TransformationCoûteuse(x));

// Seules les 5 premières transformations seront effectuées
var premiers = requêteDifférée.Take(5).ToList();
```

### 6. Projections avancées avec des agrégations

Les projections peuvent combiner des données de plusieurs sources ou effectuer des calculs complexes.

```csharp
// Projection avec des données agrégées
var clientsAvecStatistiques = _context.Clients
    .Select(c => new
    {
        c.Id,
        c.Nom,
        DernièreCommande = c.Commandes.OrderByDescending(co => co.Date)
                                     .Select(co => co.Date)
                                     .FirstOrDefault(),
        ValeurMoyenne = c.Commandes.Any()
                      ? c.Commandes.Average(co => co.Total)
                      : 0,
        CatégoriePréférée = c.Commandes
                           .SelectMany(co => co.Lignes)
                           .GroupBy(l => l.Produit.CatégorieId)
                           .OrderByDescending(g => g.Count())
                           .Select(g => g.Key)
                           .FirstOrDefault()
    })
    .ToList();
```

### 7. Utiliser des projections anonymes vs nommées

Vous avez le choix entre des projections anonymes et des projections vers des types nommés.

```csharp
// Projection anonyme
var résultatAnonyme = clients
    .Select(c => new { c.Id, c.Nom, Age = DateTime.Now.Year - c.DateNaissance.Year })
    .ToList();

// Projection vers un type nommé
var résultatTypé = clients
    .Select(c => new ClientViewModel
    {
        Id = c.Id,
        Nom = c.Nom,
        Age = DateTime.Now.Year - c.DateNaissance.Year
    })
    .ToList();
```

**Avantages des types anonymes :**
- Moins de code à écrire
- Bien adaptés pour les requêtes locales ou les résultats temporaires

**Avantages des types nommés :**
- Meilleure maintenabilité pour des structures utilisées à plusieurs endroits
- Possibilité d'ajouter des méthodes et des propriétés calculées
- Meilleure expérience de débogage
- Facilité de passage entre les méthodes et les couches

### 8. Composition de projections

Vous pouvez composer plusieurs projections pour des transformations complexes.

```csharp
// Composition de projections
var résultat = collection
    .Select(x => new { x.Id, x.Nom, x.Valeurs })          // Première projection
    .Select(x => new { x.Id, x.Nom, Somme = x.Valeurs.Sum() }) // Deuxième projection
    .Where(x => x.Somme > 100)                           // Filtrage intermédiaire
    .Select(x => new RésultatFinal(x.Id, x.Nom, x.Somme)) // Projection finale
    .ToList();
```

Bien que cette approche fonctionne, elle peut être moins efficace qu'une projection unique. Comparez avec :

```csharp
// Projection unique plus efficace
var résultatOptimisé = collection
    .Where(x => x.Valeurs.Sum() > 100)  // Filtrage précoce
    .Select(x => new RésultatFinal(x.Id, x.Nom, x.Valeurs.Sum()))
    .ToList();
```

## Exemples pratiques d'optimisation LINQ

Maintenant que nous avons couvert les principes fondamentaux, voyons quelques exemples pratiques d'optimisation de requêtes LINQ dans des scénarios du monde réel.

### Exemple 1 : Optimiser une requête complexe

Voici une requête LINQ complexe et son optimisation progressive.

```csharp
// Classe de données
public class Commande
{
    public int Id { get; set; }
    public DateTime Date { get; set; }
    public string ClientId { get; set; }
    public List<LigneCommande> Lignes { get; set; }
    public decimal Total => Lignes.Sum(l => l.Prix * l.Quantité);
}

public class LigneCommande
{
    public int ProduitId { get; set; }
    public string NomProduit { get; set; }
    public decimal Prix { get; set; }
    public int Quantité { get; set; }
    public string Catégorie { get; set; }
}

// Version initiale (non optimisée)
public List<RapportVente> GénérerRapportVentes(List<Commande> commandes, DateTime début, DateTime fin)
{
    var rapport = new List<RapportVente>();

    // Filtrage des commandes
    var commandesFiltrees = commandes.Where(c => c.Date >= début && c.Date <= fin).ToList();

    // Récupération des catégories
    var catégories = commandesFiltrees
        .SelectMany(c => c.Lignes)
        .Select(l => l.Catégorie)
        .Distinct()
        .ToList();

    // Pour chaque catégorie, calculer les statistiques
    foreach (var catégorie in catégories)
    {
        var ligneCatégorie = commandesFiltrees
            .SelectMany(c => c.Lignes)
            .Where(l => l.Catégorie == catégorie)
            .ToList();

        var totalVentes = ligneCatégorie.Sum(l => l.Prix * l.Quantité);
        var nombreProduits = ligneCatégorie.Select(l => l.ProduitId).Distinct().Count();
        var quantitéVendue = ligneCatégorie.Sum(l => l.Quantité);

        rapport.Add(new RapportVente
        {
            Catégorie = catégorie,
            TotalVentes = totalVentes,
            NombreProduits = nombreProduits,
            QuantitéVendue = quantitéVendue
        });
    }

    // Tri par total des ventes décroissant
    return rapport.OrderByDescending(r => r.TotalVentes).ToList();
}

public class RapportVente
{
    public string Catégorie { get; set; }
    public decimal TotalVentes { get; set; }
    public int NombreProduits { get; set; }
    public int QuantitéVendue { get; set; }
}
```

**Problèmes avec cette version :**
1. Multiples parcours de la collection
2. Création de listes intermédiaires inutiles
3. Calculs redondants

**Version optimisée :**

```csharp
public List<RapportVente> GénérerRapportVentesOptimisé(List<Commande> commandes, DateTime début, DateTime fin)
{
    // Une seule requête LINQ pour tout faire
    return commandes
        .Where(c => c.Date >= début && c.Date <= fin)
        .SelectMany(c => c.Lignes)  // Aplatir toutes les lignes de commandes
        .GroupBy(l => l.Catégorie)  // Regrouper par catégorie
        .Select(g => new RapportVente
        {
            Catégorie = g.Key,
            TotalVentes = g.Sum(l => l.Prix * l.Quantité),
            NombreProduits = g.Select(l => l.ProduitId).Distinct().Count(),
            QuantitéVendue = g.Sum(l => l.Quantité)
        })
        .OrderByDescending(r => r.TotalVentes)
        .ToList();
}
```

**Avantages de la version optimisée :**
1. Une seule requête LINQ complète
2. Pas de listes intermédiaires
3. Chaque élément n'est traité qu'une seule fois
4. Code plus concis et lisible

### Exemple 2 : Pagination efficace

La pagination est un cas d'utilisation courant où l'optimisation LINQ peut faire une grande différence.

```csharp
// Approche naïve (inefficace pour de grandes collections)
public List<Produit> ObtenirProduitsPaginés(List<Produit> produits, int page, int taillePage)
{
    return produits
        .OrderBy(p => p.Nom)  // Tri de tous les produits
        .Skip((page - 1) * taillePage)
        .Take(taillePage)
        .ToList();
}

// Approche optimisée pour Entity Framework
public async Task<List<ProduitViewModel>> ObtenirProduitsPaginésOptimisé(
    int page,
    int taillePage,
    string tri = "nom",
    bool ascendant = true)
{
    // Requête de base
    IQueryable<Produit> requête = _context.Produits;

    // Appliquer le tri dynamiquement
    requête = AppliqueTri(requête, tri, ascendant);

    // Récupérer uniquement les colonnes nécessaires et paginer
    return await requête
        .Skip((page - 1) * taillePage)
        .Take(taillePage)
        .Select(p => new ProduitViewModel
        {
            Id = p.Id,
            Nom = p.Nom,
            Prix = p.Prix,
            Catégorie = p.Catégorie.Nom  // Évite de charger la catégorie entière
        })
        .ToListAsync();
}

private IQueryable<Produit> AppliqueTri(
    IQueryable<Produit> requête,
    string tri,
    bool ascendant)
{
    // Définir le tri en fonction du paramètre
    switch (tri.ToLower())
    {
        case "prix":
            return ascendant
                ? requête.OrderBy(p => p.Prix)
                : requête.OrderByDescending(p => p.Prix);
        case "catégorie":
            return ascendant
                ? requête.OrderBy(p => p.Catégorie.Nom)
                : requête.OrderByDescending(p => p.Catégorie.Nom);
        case "nom":
        default:
            return ascendant
                ? requête.OrderBy(p => p.Nom)
                : requête.OrderByDescending(p => p.Nom);
    }
}
```

**Points clés d'optimisation :**
1. Tri et sélection uniquement des données nécessaires
2. Pagination appliquée au niveau de la base de données (avec IQueryable)
3. Projection pour minimiser les données transférées

### Exemple 3 : Recherche et filtrage efficaces

Les opérations de recherche et de filtrage sont courantes et peuvent bénéficier d'une optimisation LINQ.

```csharp
// Méthode de recherche optimisée avec plusieurs critères
public async Task<List<ClientViewModel>> RechercherClients(RechercheClientModel critères)
{
    // Commencer avec une requête de base
    IQueryable<Client> requête = _context.Clients;

    // Appliquer les filtres uniquement si les critères sont fournis
    if (!string.IsNullOrWhiteSpace(critères.Nom))
    {
        requête = requête.Where(c => c.Nom.Contains(critères.Nom));
    }

    if (!string.IsNullOrWhiteSpace(critères.Email))
    {
        requête = requête.Where(c => c.Email.Contains(critères.Email));
    }

    if (critères.Actif.HasValue)
    {
        requête = requête.Where(c => c.Actif == critères.Actif.Value);
    }

    if (critères.DateInscriptionMin.HasValue)
    {
        requête = requête.Where(c => c.DateInscription >= critères.DateInscriptionMin.Value);
    }

    if (critères.DateInscriptionMax.HasValue)
    {
        requête = requête.Where(c => c.DateInscription <= critères.DateInscriptionMax.Value);
    }

    // Appliquer le tri demandé
    requête = critères.TriDesc
        ? requête.OrderByDescending(c => c.Nom)
        : requête.OrderBy(c => c.Nom);

    // Pagination
    requête = requête.Skip((critères.Page - 1) * critères.TaillePage)
                    .Take(critères.TaillePage);

    // Projection finale pour ne sélectionner que les données nécessaires
    return await requête
        .Select(c => new ClientViewModel
        {
            Id = c.Id,
            Nom = c.Nom,
            Email = c.Email,
            DateInscription = c.DateInscription,
            NombreCommandes = c.Commandes.Count,
            Actif = c.Actif
        })
        .ToListAsync();
}
```

**Avantages de cette approche :**
1. Les filtres sont appliqués au niveau de la base de données
2. Seuls les filtres pertinents sont utilisés
3. La pagination réduit le volume de données transférées
4. La projection optimise encore davantage le transfert de données

## Bonnes pratiques et astuces supplémentaires

Pour conclure, voici quelques bonnes pratiques et astuces supplémentaires pour optimiser vos requêtes LINQ :

### 1. Préférer les méthodes d'extension aux requêtes de type expression

Les méthodes d'extension (`Where`, `Select`, etc.) sont généralement plus faciles à déboguer et peuvent être plus efficaces que les expressions de requête (syntaxe `from ... where ... select`).

```csharp
// Expression de requête
var résultat1 = from p in produits
               where p.Prix > 100
               orderby p.Nom
               select new { p.Id, p.Nom, p.Prix };

// Méthodes d'extension (souvent préférées)
var résultat2 = produits
    .Where(p => p.Prix > 100)
    .OrderBy(p => p.Nom)
    .Select(p => new { p.Id, p.Nom, p.Prix });
```

### 2. Utiliser intelligemment les collections spécialisées

Certaines collections spécialisées peuvent améliorer les performances dans des scénarios spécifiques :

```csharp
// Pour des recherches fréquentes
HashSet<string> codesPostaux = new HashSet<string>(clients.Select(c => c.CodePostal));
bool existe = codesPostaux.Contains("75001");  // O(1)

// Pour des éléments triés
SortedDictionary<DateTime, List<Événement>> calendrier = new SortedDictionary<DateTime, List<Événement>>();
// Les clés sont automatiquement triées

// Pour un accès rapide dans les deux sens (par clé et par valeur)
var biMap = clients.ToDictionary(
    c => c.Id,             // Clé
    c => c,                // Valeur
    StringComparer.OrdinalIgnoreCase  // Comparateur personnalisé
);
```

### 3. Éviter les pièges de performance courants

```csharp
// Piège 1: Conversion ToList() avant filtrage
var résultatInefficient = collection.ToList().Where(x => x.Propriété > 10);  // Inefficace

// Correct
var résultatEfficient = collection.Where(x => x.Propriété > 10).ToList();

// Piège 2: Count() sur une collection qui connaît déjà sa taille
var nombreInefficient = maListe.Count();  // Inefficace pour List<T>

// Correct
var nombreEfficient = maListe.Count;  // Propriété O(1) sur List<T>

// Piège 3: Any() vs Count pour vérifier l'existence
bool existeInefficient = collection.Count() > 0;  // Inefficace

// Correct
bool existeEfficient = collection.Any();  // S'arrête dès qu'un élément est trouvé
```

### 4. Astuces avancées pour Entity Framework

Si vous utilisez LINQ avec Entity Framework, ces astuces peuvent améliorer considérablement les performances :

```csharp
// Charger uniquement ce dont vous avez besoin avec une projection
var produitLéger = _context.Produits
    .Select(p => new { p.Id, p.Nom })  // Ne charge pas toutes les colonnes
    .FirstOrDefault(p => p.Id == id);

// Utiliser AsNoTracking() pour les requêtes en lecture seule
var produits = _context.Produits
    .AsNoTracking()  // Évite le suivi des entités
    .Where(p => p.CatégorieId == catégorieId)
    .ToList();

// Éviter les filtres côté client (qui chargent toutes les données)
// Inefficace - le filtre s'exécute après avoir chargé toutes les commandes
var commandesInefficient = _context.Commandes.ToList()
    .Where(c => c.Date.Month == DateTime.Now.Month);

// Correct - le filtre s'exécute côté base de données
var commandesEfficient = _context.Commandes
    .Where(c => c.Date.Month == DateTime.Now.Month)
    .ToList();

// Préchargement de relations avec Include (et ThenInclude)
var clientsAvecCommandes = _context.Clients
    .Include(c => c.Commandes)
    .ThenInclude(co => co.Lignes)
    .ThenInclude(l => l.Produit)
    .Where(c => c.Pays == "France")
    .ToList();
```

### 5. Tester et mesurer, toujours

L'optimisation prématurée est souvent contre-productive. Suivez ces étapes pour une approche méthodique :

1. **Écrivez d'abord un code fonctionnel** et lisible
2. **Mesurez les performances** pour identifier les goulots d'étranglement réels
3. **Optimisez les parties critiques** en suivant les principes de ce chapitre
4. **Mesurez à nouveau** pour vérifier l'efficacité de vos optimisations
5. **Documentez les choix d'optimisation** pour aider les autres développeurs

### Conclusion

L'optimisation des requêtes LINQ nécessite une compréhension de son fonctionnement interne et une attention particulière aux détails. En suivant les principes et les bonnes pratiques présentés dans ce chapitre, vous pourrez écrire du code LINQ qui est à la fois élégant et performant.

N'oubliez pas que la lisibilité et la maintenabilité du code sont également importantes. Parfois, un léger compromis sur les performances peut valoir la peine si cela rend le code beaucoup plus clair et plus facile à maintenir. L'objectif est de trouver le bon équilibre entre des performances optimales et un code propre et maintenable.

⏭️ 17.5. [Profilage des applications](/17-performances-et-optimisation/17-5-profilage-des-applications.md)
