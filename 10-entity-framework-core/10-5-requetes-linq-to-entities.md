# 10.5. Requêtes LINQ to Entities

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'un des plus grands avantages d'Entity Framework Core est sa capacité à traduire des requêtes LINQ (Language Integrated Query) en requêtes SQL optimisées. Cette fonctionnalité, appelée "LINQ to Entities", vous permet d'interroger votre base de données en utilisant la syntaxe C# que vous connaissez déjà.

## 10.5.1. Requêtes de base

Commençons par les opérations de requête les plus courantes que vous utiliserez quotidiennement.

### Récupération de toutes les entités

Pour récupérer toutes les entités d'une table, utilisez simplement la propriété DbSet de votre contexte :

```csharp
// Obtenir tous les livres
var livres = context.Livres.ToList();
```

Cette requête est traduite en SQL comme :
```sql
SELECT * FROM Livres
```

### Filtrage avec Where

Pour filtrer les résultats, utilisez la méthode `Where` :

```csharp
// Obtenir tous les livres publiés après 2020
var livresRecents = context.Livres
    .Where(l => l.AnneePublication > 2020)
    .ToList();
```

SQL généré :
```sql
SELECT * FROM Livres WHERE AnneePublication > 2020
```

Vous pouvez combiner plusieurs conditions :

```csharp
// Livres récents d'un auteur spécifique
var livresRecentsDAuteur = context.Livres
    .Where(l => l.AnneePublication > 2020 && l.AuteurId == 5)
    .ToList();
```

### Tri avec OrderBy

Pour trier les résultats, utilisez `OrderBy` et `OrderByDescending` :

```csharp
// Livres triés par titre (A à Z)
var livresTriesParTitre = context.Livres
    .OrderBy(l => l.Titre)
    .ToList();

// Livres triés par année de publication (récent à ancien)
var livresParAnnee = context.Livres
    .OrderByDescending(l => l.AnneePublication)
    .ToList();
```

Pour appliquer plusieurs niveaux de tri :

```csharp
// Livres triés par auteur puis par titre
var livresTriesParAuteurPuisTitre = context.Livres
    .OrderBy(l => l.Auteur.Nom)
    .ThenBy(l => l.Titre)
    .ToList();
```

### Limiter les résultats avec Take et Skip

Pour limiter le nombre de résultats ou implémenter la pagination :

```csharp
// Récupérer seulement les 10 premiers livres
var premiersDixLivres = context.Livres
    .Take(10)
    .ToList();

// Pagination (page 3, 20 éléments par page)
var page = 3;
var elementsParPage = 20;
var livresPage3 = context.Livres
    .Skip((page - 1) * elementsParPage)
    .Take(elementsParPage)
    .ToList();
```

### Sélection de propriétés spécifiques

Pour récupérer seulement certaines colonnes, utilisez `Select` :

```csharp
// Récupérer seulement les titres et années
var infoLivres = context.Livres
    .Select(l => new { l.Titre, l.AnneePublication })
    .ToList();
```

### Requêtes avec FirstOrDefault, SingleOrDefault, etc.

Pour récupérer un élément spécifique :

```csharp
// Obtenir un livre par son ID
var livre = context.Livres
    .FirstOrDefault(l => l.Id == 42);

// Si aucun livre n'est trouvé, livre sera null
if (livre != null)
{
    Console.WriteLine($"Livre trouvé : {livre.Titre}");
}
```

Différences entre les méthodes :
- `First`/`FirstOrDefault` : Retourne le premier élément ou lève une exception/retourne null
- `Single`/`SingleOrDefault` : Vérifie qu'il n'y a qu'un seul résultat, sinon lève une exception
- `Last`/`LastOrDefault` : Comme First, mais prend le dernier élément (nécessite un tri préalable)

### Requêtes asynchrones

Dans les applications modernes, privilégiez les méthodes asynchrones pour éviter de bloquer le thread principal :

```csharp
// Utilisation asynchrone de ToListAsync
var livres = await context.Livres
    .Where(l => l.AnneePublication > 2020)
    .ToListAsync();

// Récupération asynchrone d'un élément
var livre = await context.Livres
    .FirstOrDefaultAsync(l => l.Id == 42);
```

> **Note** : Pour utiliser les méthodes asynchrones, n'oubliez pas d'ajouter `using Microsoft.EntityFrameworkCore;` en haut de votre fichier.

## 10.5.2. Requêtes avancées

Passons maintenant à des requêtes plus complexes.

### Utilisation de fonctions

EF Core traduit de nombreuses fonctions C# en leurs équivalents SQL :

```csharp
// Recherche avec LIKE
var livresAvecMotCle = context.Livres
    .Where(l => l.Titre.Contains("aventure"))
    .ToList();

// Avec insensibilité à la casse
var livresAvecMotCleInsensible = context.Livres
    .Where(l => EF.Functions.Like(l.Titre, "%aventure%"))
    .ToList();

// Fonctions de date
var livresAjoutesRecemment = context.Livres
    .Where(l => EF.Functions.DateDiffDay(l.DateAjout, DateTime.Today) < 30)
    .ToList();
```

### Requêtes de groupement

Pour regrouper des résultats, utilisez `GroupBy` :

```csharp
// Nombre de livres par année de publication
var livresParAnnee = context.Livres
    .GroupBy(l => l.AnneePublication)
    .Select(g => new {
        Annee = g.Key,
        NombreDeLivres = g.Count()
    })
    .OrderBy(x => x.Annee)
    .ToList();

// Moyenne des prix par catégorie
var prixMoyenParCategorie = context.Livres
    .GroupBy(l => l.Categorie.Nom)
    .Select(g => new {
        Categorie = g.Key,
        PrixMoyen = g.Average(l => l.Prix)
    })
    .ToList();
```

### Requêtes avec Any et All

Les méthodes `Any` et `All` sont utiles pour vérifier des conditions sur des collections :

```csharp
// Trouver les auteurs qui ont au moins un livre publié après 2020
var auteursAvecLivresRecents = context.Auteurs
    .Where(a => a.Livres.Any(l => l.AnneePublication > 2020))
    .ToList();

// Trouver les auteurs dont tous les livres coûtent plus de 20€
var auteursLivresCher = context.Auteurs
    .Where(a => a.Livres.All(l => l.Prix > 20))
    .ToList();
```

### Utilisation de l'opérateur ternaire

```csharp
// Classification des livres par prix
var livresAvecClassification = context.Livres
    .Select(l => new {
        l.Titre,
        l.Prix,
        Categorie = l.Prix > 50 ? "Premium" : (l.Prix > 20 ? "Standard" : "Économique")
    })
    .ToList();
```

### Utilisation de la méthode Let pour réutiliser un calcul

Avec une requête LINQ, vous pouvez utiliser des variables temporaires :

```csharp
var livresAvecRemise = from l in context.Livres
                      let remise = l.Prix * 0.1m
                      where remise > 5
                      select new { l.Titre, PrixOriginal = l.Prix, Remise = remise, PrixFinal = l.Prix - remise };
```

## 10.5.3. Chargement de données associées

Entity Framework Core offre plusieurs façons de charger des données associées (relations).

### Eager Loading avec Include

Le chargement anticipé (eager loading) charge les entités associées dans la même requête :

```csharp
// Charger les livres avec leurs auteurs
var livresAvecAuteurs = context.Livres
    .Include(l => l.Auteur)
    .ToList();

// Chargement de relations multi-niveaux
var livresAvecAuteursEtEditeurs = context.Livres
    .Include(l => l.Auteur)
    .Include(l => l.Editeur)
    .ToList();

// Chargement de relations imbriquées
var livresAvecCategories = context.Livres
    .Include(l => l.Auteur)
        .ThenInclude(a => a.Adresse)
    .ToList();
```

### Filtrer les données incluses avec Where

Vous pouvez filtrer les entités principales, mais pas directement les entités chargées avec Include :

```csharp
// Correct : Filtrer les livres
var livresRecentAvecAuteur = context.Livres
    .Where(l => l.AnneePublication > 2020)
    .Include(l => l.Auteur)
    .ToList();
```

Pour filtrer des entités associées, utilisez des solutions alternatives comme :

```csharp
// Approche 1 : Filtrer après chargement (côté client)
var livresAvecAuteursPopulaires = context.Livres
    .Include(l => l.Auteur)
    .ToList()
    .Where(l => l.Auteur.Popularite > 8)
    .ToList();

// Approche 2 : Utiliser des projections (préférable pour les grandes quantités)
var livresAvecAuteursPopulaires = context.Livres
    .Where(l => l.Auteur.Popularite > 8)
    .Select(l => new {
        Livre = l,
        Auteur = l.Auteur
    })
    .ToList();
```

### Explicit Loading

Le chargement explicite charge les entités associées à la demande, après avoir chargé l'entité principale :

```csharp
// Charger d'abord un livre
var livre = context.Livres.Find(42);

// Puis charger son auteur si nécessaire
if (livre != null)
{
    context.Entry(livre)
        .Reference(l => l.Auteur)
        .Load();

    Console.WriteLine($"Livre: {livre.Titre}, Auteur: {livre.Auteur.Nom}");
}
```

Pour les collections :

```csharp
// Charger d'abord un auteur
var auteur = context.Auteurs.Find(5);

// Puis charger ses livres
context.Entry(auteur)
    .Collection(a => a.Livres)
    .Load();
```

### Lazy Loading

Le chargement paresseux (lazy loading) charge automatiquement les entités associées lorsqu'elles sont accédées pour la première fois :

Pour activer le lazy loading :

1. Installez le package : `Microsoft.EntityFrameworkCore.Proxies`
2. Configurez votre contexte :

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseLazyLoadingProxies()
        .UseSqlServer("votre_chaine_de_connexion");
}
```

3. Modifiez vos entités pour avoir des propriétés de navigation virtuelles :

```csharp
public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }

    public int AuteurId { get; set; }
    public virtual Auteur Auteur { get; set; } // Le mot-clé "virtual" est important
}
```

Avec cette configuration, les données associées seront chargées automatiquement :

```csharp
var livre = context.Livres.Find(42);
// La ligne suivante déclenchera automatiquement une requête pour charger l'auteur
Console.WriteLine($"Auteur: {livre.Auteur.Nom}");
```

> **Attention** : Le lazy loading peut entraîner des problèmes de performance (problème N+1) si mal utilisé.

## 10.5.4. Requêtes brutes SQL

Bien que LINQ soit puissant, parfois vous avez besoin d'exécuter du SQL directement.

### FromSqlRaw pour les requêtes SELECT

Pour exécuter une requête SQL brute qui retourne des entités :

```csharp
var livres = context.Livres
    .FromSqlRaw("SELECT * FROM Livres WHERE AnneePublication > 2020")
    .ToList();
```

Avec des paramètres (sécurisé contre l'injection SQL) :

```csharp
var annee = 2020;
var livres = context.Livres
    .FromSqlRaw("SELECT * FROM Livres WHERE AnneePublication > {0}", annee)
    .ToList();
```

### Combinaison avec LINQ

Vous pouvez combiner des requêtes SQL brutes avec LINQ :

```csharp
var livres = context.Livres
    .FromSqlRaw("SELECT * FROM Livres")
    .Where(l => l.Prix < 20)
    .OrderBy(l => l.Titre)
    .ToList();
```

### Exécution de commandes non-query

Pour les requêtes qui ne retournent pas d'entités (INSERT, UPDATE, DELETE) :

```csharp
var rowsAffected = context.Database.ExecuteSqlRaw(
    "UPDATE Livres SET Prix = Prix * 0.9 WHERE AnneePublication < 2000");

Console.WriteLine($"{rowsAffected} livres ont été mis à jour.");
```

### Requêtes SQL retournant des types personnalisés

Pour les requêtes qui retournent des données qui ne correspondent pas à vos entités :

```csharp
// Créez d'abord une classe pour les résultats
public class StatistiqueLivre
{
    public string Categorie { get; set; }
    public int NombreDeLivres { get; set; }
    public decimal PrixMoyen { get; set; }
}

// Puis exécutez la requête
var stats = context.Database.SqlQuery<StatistiqueLivre>(@"
    SELECT c.Nom AS Categorie,
           COUNT(*) AS NombreDeLivres,
           AVG(l.Prix) AS PrixMoyen
    FROM Livres l
    JOIN Categories c ON l.CategorieId = c.Id
    GROUP BY c.Nom
    ORDER BY NombreDeLivres DESC")
    .ToList();
```

## 10.5.5. Procédures stockées et fonctions

Entity Framework Core prend également en charge l'appel de procédures stockées et de fonctions SQL.

### Exécution de procédures stockées

Pour exécuter une procédure stockée simple :

```csharp
var livresAjoutesRecemment = context.Livres
    .FromSqlRaw("EXEC GetLivresRecentsPar30Jours")
    .ToList();
```

Avec des paramètres :

```csharp
var categorieId = 5;
var livresParCategorie = context.Livres
    .FromSqlRaw("EXEC GetLivresParCategorie @p0", categorieId)
    .ToList();
```

### Mappage de procédures stockées aux méthodes

Vous pouvez mapper des procédures stockées à des méthodes dans votre contexte :

```csharp
// Dans votre classe de contexte
public IQueryable<Livre> GetLivresParAuteur(int auteurId)
{
    return Livres.FromSqlRaw("EXEC GetLivresParAuteur @p0", auteurId);
}

// Utilisation
var livres = context.GetLivresParAuteur(42).ToList();
```

### Utilisation de fonctions SQL

Vous pouvez mapper des fonctions SQL scalaires (qui retournent une valeur unique) ou tabulaires (qui retournent un ensemble de résultats) :

```csharp
// Dans OnModelCreating
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Fonction scalaire
    modelBuilder.HasDbFunction(
        typeof(BibliothequeContext).GetMethod(nameof(CalculerRemise),
        new[] { typeof(decimal), typeof(decimal) }))
        .HasName("CalculerRemise");

    // Fonction tabulaire
    modelBuilder.HasDbFunction(
        typeof(BibliothequeContext).GetMethod(nameof(LivresParMotCle),
        new[] { typeof(string) }));
}

// Méthodes dans votre contexte
public decimal CalculerRemise(decimal prix, decimal pourcentage)
    => throw new NotImplementedException(); // Implémentée par EF Core

public IQueryable<Livre> LivresParMotCle(string motCle)
    => FromExpression(() => LivresParMotCle(motCle));
```

Utilisation des fonctions :

```csharp
// Fonction scalaire
var livresAvecRemise = context.Livres
    .Select(l => new {
        l.Titre,
        l.Prix,
        Remise = context.CalculerRemise(l.Prix, 0.1m)
    })
    .ToList();

// Fonction tabulaire
var livresRomans = context.LivresParMotCle("roman")
    .Where(l => l.AnneePublication > 2000)
    .ToList();
```

### Exemple complet avec SP et sortie de paramètre

```csharp
// Définition d'une procédure stockée dans SQL Server
/*
CREATE PROCEDURE ObtenirNombreLivresParAuteur
    @AuteurId INT,
    @NombreDeLivres INT OUTPUT
AS
BEGIN
    SELECT @NombreDeLivres = COUNT(*)
    FROM Livres
    WHERE AuteurId = @AuteurId
END
*/

// Utilisation dans C#
using (var command = context.Database.GetDbConnection().CreateCommand())
{
    command.CommandText = "ObtenirNombreLivresParAuteur";
    command.CommandType = System.Data.CommandType.StoredProcedure;

    var auteurIdParam = command.CreateParameter();
    auteurIdParam.ParameterName = "@AuteurId";
    auteurIdParam.Value = 42;
    command.Parameters.Add(auteurIdParam);

    var nombreDeLivresParam = command.CreateParameter();
    nombreDeLivresParam.ParameterName = "@NombreDeLivres";
    nombreDeLivresParam.Direction = System.Data.ParameterDirection.Output;
    nombreDeLivresParam.DbType = System.Data.DbType.Int32;
    command.Parameters.Add(nombreDeLivresParam);

    if (context.Database.GetDbConnection().State != System.Data.ConnectionState.Open)
        context.Database.GetDbConnection().Open();

    command.ExecuteNonQuery();

    var nombreDeLivres = (int)nombreDeLivresParam.Value;
    Console.WriteLine($"L'auteur a écrit {nombreDeLivres} livres.");
}
```

---

LINQ to Entities est l'une des fonctionnalités les plus puissantes d'Entity Framework Core. Il vous permet d'interroger votre base de données avec une syntaxe naturelle en C#, tout en générant des requêtes SQL efficaces. Maîtriser les différentes techniques de requêtes et de chargement vous aidera à créer des applications performantes et maintenables.

⏭️ 10.6. [Relations entre entités](/10-entity-framework-core/10-6-relations-entre-entites.md)
