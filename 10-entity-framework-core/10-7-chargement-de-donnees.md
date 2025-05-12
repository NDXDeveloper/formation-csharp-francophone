# 10.7. Chargement de données

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Dans une application utilisant Entity Framework Core, le chargement efficace des données associées (ou données de navigation) est crucial pour les performances. EF Core offre plusieurs stratégies pour charger ces données liées, chacune avec ses propres avantages et cas d'utilisation. Cette section vous aidera à comprendre ces différentes approches et à choisir la plus appropriée pour vos besoins.

## 10.7.1. Eager Loading

Le chargement anticipé (Eager Loading) consiste à charger les entités associées en même temps que l'entité principale dans une seule requête SQL. C'est généralement réalisé avec la méthode `Include()`.

### Fonctionnement de base

```csharp
// Chargement d'un livre avec son auteur dans une seule requête
var livreAvecAuteur = context.Livres
    .Include(l => l.Auteur)
    .FirstOrDefault(l => l.Id == 1);

// On peut maintenant accéder à l'auteur sans requête supplémentaire
Console.WriteLine($"Livre: {livreAvecAuteur.Titre}, Auteur: {livreAvecAuteur.Auteur.Nom}");
```

### Chargement de relations multi-niveaux

Vous pouvez charger des relations imbriquées avec `ThenInclude()` :

```csharp
// Chargement d'un livre avec son auteur et l'adresse de l'auteur
var livre = context.Livres
    .Include(l => l.Auteur)
        .ThenInclude(a => a.Adresse)
    .FirstOrDefault(l => l.Id == 1);

Console.WriteLine($"Livre: {livre.Titre}");
Console.WriteLine($"Auteur: {livre.Auteur.Nom}");
Console.WriteLine($"Ville: {livre.Auteur.Adresse.Ville}");
```

### Chargement de plusieurs relations

Vous pouvez inclure plusieurs relations dans une même requête :

```csharp
// Chargement d'un livre avec son auteur et ses catégories
var livre = context.Livres
    .Include(l => l.Auteur)
    .Include(l => l.Categories)
    .FirstOrDefault(l => l.Id == 1);

Console.WriteLine($"Livre: {livre.Titre}, Auteur: {livre.Auteur.Nom}");
Console.WriteLine("Catégories:");
foreach (var categorie in livre.Categories)
{
    Console.WriteLine($"- {categorie.Nom}");
}
```

### Filtrage des données incluses

À partir d'EF Core 5.0, vous pouvez filtrer les entités associées qui sont chargées :

```csharp
// Chargement d'un auteur avec seulement ses livres publiés après 2020
var auteur = context.Auteurs
    .Include(a => a.Livres.Where(l => l.AnneePublication > 2020))
    .FirstOrDefault(a => a.Id == 1);
```

### Avantages de l'Eager Loading

- **Performance** : Une seule requête SQL pour récupérer toutes les données nécessaires
- **Simplicité** : Les données sont immédiatement disponibles
- **Déconnecté** : Fonctionne même après la fermeture du contexte

### Inconvénients de l'Eager Loading

- **Surcharge mémoire** : Charge toutes les données, même si elles ne sont pas utilisées
- **Requêtes complexes** : Peut générer des jointures SQL complexes pour les relations multiniveaux
- **Surcharge réseau** : Transfère plus de données que nécessaire si seule une partie est utilisée

## 10.7.2. Lazy Loading

Le chargement paresseux (Lazy Loading) charge automatiquement les entités associées uniquement lorsqu'elles sont accédées pour la première fois. C'est une stratégie "à la demande".

### Configuration du Lazy Loading

Pour utiliser le Lazy Loading, vous devez d'abord l'activer. Il y a deux approches principales :

#### 1. Utilisation des proxies

```csharp
// 1. Installer le package NuGet
// Install-Package Microsoft.EntityFrameworkCore.Proxies

// 2. Configurer le contexte
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseLazyLoadingProxies()
        .UseSqlServer("votre_chaine_de_connexion");
}

// 3. Déclarer les propriétés de navigation comme virtuelles
public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }

    public int AuteurId { get; set; }
    public virtual Auteur Auteur { get; set; }  // Le mot-clé virtual est important
}
```

#### 2. Injection de service ILazyLoader

```csharp
// Approche alternative sans proxy
public class Livre
{
    private ILazyLoader _lazyLoader;
    private Auteur _auteur;

    public Livre() { }

    // Constructeur pour l'injection
    public Livre(ILazyLoader lazyLoader)
    {
        _lazyLoader = lazyLoader;
    }

    public int Id { get; set; }
    public string Titre { get; set; }
    public int AuteurId { get; set; }

    public Auteur Auteur
    {
        get => _lazyLoader.Load(this, ref _auteur);
        set => _auteur = value;
    }
}
```

### Utilisation du Lazy Loading

Une fois configuré, le Lazy Loading fonctionne automatiquement :

```csharp
// Chargement d'un livre sans ses relations
var livre = context.Livres.Find(1);

// La première fois que vous accédez à Auteur, EF Core exécutera
// automatiquement une requête pour charger cette entité
Console.WriteLine($"Auteur: {livre.Auteur.Nom}");  // Déclenche le chargement
```

### Avantages du Lazy Loading

- **Simplicité d'utilisation** : Chargement automatique sans code supplémentaire
- **Efficacité mémoire** : Charge uniquement les données nécessaires
- **Intuitivité** : Correspond au comportement attendu d'un objet

### Inconvénients du Lazy Loading

- **Problème N+1** : Peut entraîner de nombreuses petites requêtes SQL
- **Contexte ouvert** : Nécessite que le contexte de base de données reste ouvert
- **Difficile à déboguer** : Les requêtes sont implicites et peuvent être difficiles à tracer
- **Impact sur les performances** : Possible surcharge de la base de données avec de nombreuses petites requêtes

## 10.7.3. Explicit Loading

Le chargement explicite (Explicit Loading) vous permet de charger manuellement les entités associées à la demande, mais de manière contrôlée.

### Chargement d'une référence (relation un-à-un ou plusieurs-à-un)

```csharp
// Chargement d'un livre sans ses relations
var livre = context.Livres.Find(1);

// Chargement explicite de l'auteur
context.Entry(livre)
    .Reference(l => l.Auteur)
    .Load();

Console.WriteLine($"Auteur: {livre.Auteur.Nom}");
```

### Chargement d'une collection (relation un-à-plusieurs)

```csharp
// Chargement d'un auteur sans ses livres
var auteur = context.Auteurs.Find(1);

// Chargement explicite des livres
context.Entry(auteur)
    .Collection(a => a.Livres)
    .Load();

Console.WriteLine($"Nombre de livres: {auteur.Livres.Count}");
```

### Filtrage avec le chargement explicite

Vous pouvez filtrer les entités associées lors du chargement explicite :

```csharp
// Chargement filtré des livres d'un auteur
var auteur = context.Auteurs.Find(1);

context.Entry(auteur)
    .Collection(a => a.Livres)
    .Query()
    .Where(l => l.AnneePublication > 2020)
    .Load();

Console.WriteLine($"Nombre de livres récents: {auteur.Livres.Count}");
```

### Chargement explicite asynchrone

```csharp
// Version asynchrone du chargement explicite
var livre = await context.Livres.FindAsync(1);

await context.Entry(livre)
    .Reference(l => l.Auteur)
    .LoadAsync();
```

### Avantages de l'Explicit Loading

- **Contrôle précis** : Vous décidez exactement quand et quoi charger
- **Filtrage avancé** : Possibilité de filtrer les données associées
- **Performances** : Évite le problème N+1 du Lazy Loading
- **Clarté du code** : Le chargement est explicite et visible

### Inconvénients de l'Explicit Loading

- **Verbosité** : Nécessite plus de code pour charger les relations
- **Contexte ouvert** : Requiert que le contexte soit toujours actif
- **Risque d'oubli** : Possible oubli de charger certaines relations nécessaires

## 10.7.4. Stratégies de chargement et performance

Le choix de la stratégie de chargement a un impact significatif sur les performances de votre application. Voici quelques conseils pour optimiser le chargement de données.

### Comparaison des stratégies

| Stratégie | Points forts | Points faibles | Meilleur pour |
|-----------|--------------|-----------------|---------------|
| **Eager Loading** | Une seule requête, fonctionne avec contexte fermé | Peut charger trop de données | Données prévisibles, relations simples |
| **Lazy Loading** | Simple à utiliser, charge à la demande | Problème N+1, contexte ouvert | Prototypage, relations rarement utilisées |
| **Explicit Loading** | Contrôle précis, filtrage avancé | Verbeux, risque d'oubli | Logique conditionnelle complexe |

### Le problème N+1

Le problème N+1 est l'un des pièges les plus courants en termes de performance :

```csharp
// Problème N+1 avec Lazy Loading
var auteurs = context.Auteurs.ToList();  // 1 requête pour les auteurs

foreach (var auteur in auteurs)
{
    // N requêtes (une pour chaque auteur)
    Console.WriteLine($"Auteur: {auteur.Nom}, Livres: {auteur.Livres.Count}");
}
```

Ce code génère 1 + N requêtes SQL (1 pour charger les auteurs, puis N requêtes pour charger les livres de chaque auteur).

### Solutions au problème N+1

```csharp
// Solution 1: Eager Loading
var auteurs = context.Auteurs
    .Include(a => a.Livres)
    .ToList();  // 1 seule requête

// Solution 2: Projections
var donnees = context.Auteurs
    .Select(a => new
    {
        a.Nom,
        NombreLivres = a.Livres.Count
    })
    .ToList();  // 1 seule requête
```

### Quand utiliser chaque stratégie

#### Utiliser Eager Loading quand:
- Vous savez à l'avance quelles relations seront nécessaires
- Les entités associées sont presque toujours utilisées ensemble
- Vous avez besoin des données après fermeture du contexte
- Les relations ne sont pas trop profondes ou volumineuses

```csharp
// Bon usage de l'Eager Loading
var commandesClients = context.Commandes
    .Include(c => c.Client)
    .Include(c => c.LignesCommande)
        .ThenInclude(l => l.Produit)
    .Where(c => c.DateCommande >= DateTime.Today.AddDays(-30))
    .ToList();
```

#### Utiliser Lazy Loading quand:
- Vous ne savez pas quelles relations seront nécessaires
- Vous êtes en phase de prototypage
- Les relations sont rarement accédées
- La performance n'est pas critique

```csharp
// Bon usage du Lazy Loading
var client = context.Clients.Find(clientId);
Console.WriteLine($"Nom: {client.Nom}");

// La relation Commandes n'est chargée que si cette condition est vraie
if (afficherHistorique)
{
    foreach (var commande in client.Commandes)
    {
        Console.WriteLine($"Commande #{commande.Reference}");
    }
}
```

#### Utiliser Explicit Loading quand:
- Vous avez besoin d'un contrôle fin sur le chargement
- Vous devez filtrer les entités associées
- Vous chargez conditionnellement certaines relations
- Vous voulez éviter le problème N+1 tout en conservant de la flexibilité

```csharp
// Bon usage de l'Explicit Loading
var auteur = context.Auteurs.Find(auteurId);

// Chargement conditionnel et filtré
if (montrerLivresRecents)
{
    context.Entry(auteur)
        .Collection(a => a.Livres)
        .Query()
        .Where(l => l.AnneePublication >= DateTime.Today.Year - 2)
        .OrderByDescending(l => l.AnneePublication)
        .Load();
}
```

### Bonnes pratiques pour la performance

1. **Projections** : Utilisez `Select()` pour récupérer uniquement les données nécessaires

```csharp
// Au lieu de charger des entités complètes
var titresLivres = context.Livres
    .Where(l => l.AuteurId == auteurId)
    .Select(l => new { l.Id, l.Titre, l.AnneePublication })
    .ToList();
```

2. **Chargement divisé (Split Queries)** : Pour les relations volumineuses (EF Core 5.0+)

```csharp
var auteurs = context.Auteurs
    .Include(a => a.Livres)
    .AsSplitQuery()  // Exécute plusieurs requêtes au lieu d'une grande jointure
    .ToList();
```

3. **AsNoTracking** : Pour les requêtes en lecture seule

```csharp
var livres = context.Livres
    .Include(l => l.Auteur)
    .AsNoTracking()  // Améliore les performances en désactivant le suivi des entités
    .ToList();
```

4. **Pagination** : Limitez le nombre d'enregistrements chargés

```csharp
// Chargement paginé
var livresPage = context.Livres
    .Include(l => l.Auteur)
    .OrderBy(l => l.Titre)
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToList();
```

5. **Caching** : Mettez en cache les données fréquemment utilisées

```csharp
// Exemple simple de mise en cache
private static List<Categorie> _categoriesCache;

public List<Categorie> GetAllCategories()
{
    if (_categoriesCache == null)
    {
        _categoriesCache = context.Categories
            .AsNoTracking()
            .ToList();
    }

    return _categoriesCache;
}
```

### Profilage et optimisation

Pour identifier les problèmes de performance liés au chargement des données :

1. **Activez la journalisation des requêtes SQL**

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer("votre_chaine_de_connexion")
        .LogTo(Console.WriteLine, LogLevel.Information);
}
```

2. **Utilisez des outils de profilage**
   - SQL Server Profiler
   - Outils de débogage intégrés à Visual Studio
   - Extensions tierces comme Entity Framework Profiler

3. **Surveillez le nombre de requêtes et leur complexité**
   - Un nombre élevé de petites requêtes indique souvent un problème N+1
   - Des requêtes très complexes peuvent suggérer un besoin de simplifier les `Include`

---

Le choix d'une stratégie de chargement dépend du contexte spécifique de votre application. Dans la pratique, la plupart des applications utilisent une combinaison des trois approches selon les besoins. L'important est de comprendre les implications de chaque stratégie et de choisir celle qui offre le meilleur équilibre entre simplicité de développement et performance pour chaque cas d'utilisation.

⏭️ 10.8. [Transactions et concurrence](/10-entity-framework-core/10-8-transactions-et-concurrence.md)
