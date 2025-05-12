# 11.5. Optimisation des performances

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'optimisation des performances est cruciale pour toute application qui interagit avec une base de données, particulièrement lorsque le volume de données ou le nombre d'utilisateurs augmente. MySQL et MariaDB offrent plusieurs outils et techniques pour améliorer les performances, que nous allons explorer dans cette section.

## 11.5.1. Indexation

L'indexation est l'une des techniques d'optimisation les plus puissantes pour améliorer les performances des requêtes. Un index permet à la base de données de trouver des données rapidement sans avoir à parcourir toutes les lignes d'une table.

### Principes de base de l'indexation

Un index dans MySQL/MariaDB fonctionne comme l'index d'un livre : il permet de trouver rapidement l'information recherchée sans lire tout le contenu.

```sql
-- Création d'un index simple sur une colonne
CREATE INDEX IX_Produits_Nom ON Produits(Nom);

-- Création d'un index composé sur plusieurs colonnes
CREATE INDEX IX_Commandes_ClientId_Date ON Commandes(ClientId, DateCommande);
```

### Types d'index dans MySQL/MariaDB

MySQL/MariaDB propose plusieurs types d'index pour différents besoins :

1. **Index B-Tree (par défaut)** : Index standard pour les recherches d'égalité et de plage.
2. **Index FULLTEXT** : Pour les recherches textuelles avancées.
3. **Index spatial** : Pour les données géographiques.
4. **Index HASH** (surtout dans MEMORY tables) : Pour les recherches exactes très rapides.

### Création d'index avec EF Core

Avec Entity Framework Core, vous pouvez créer des index de plusieurs façons :

#### Utilisation des Data Annotations

```csharp
public class Produit
{
    public int Id { get; set; }

    [Index(nameof(Nom))]
    public string Nom { get; set; }

    [Index(nameof(CategorieId), nameof(Prix))]
    public int CategorieId { get; set; }
    public decimal Prix { get; set; }
}
```

#### Utilisation de la Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Index simple
    modelBuilder.Entity<Produit>()
        .HasIndex(p => p.Nom);

    // Index composé
    modelBuilder.Entity<Produit>()
        .HasIndex(p => new { p.CategorieId, p.Prix });

    // Index unique
    modelBuilder.Entity<Client>()
        .HasIndex(c => c.Email)
        .IsUnique();
}
```

#### Création d'index FULLTEXT

Pour les recherches textuelles, MySQL propose les index FULLTEXT. Ces index ne sont pas directement supportés par EF Core, mais vous pouvez les créer via des migrations personnalisées :

```csharp
public partial class AjoutIndexFullText : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(
            "CREATE FULLTEXT INDEX FT_Produits_Description ON Produits(Description)");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql("DROP INDEX FT_Produits_Description ON Produits");
    }
}
```

### Quand indexer et quand ne pas le faire

| Quand créer un index | Quand éviter un index |
|----------------------|------------------------|
| Colonnes utilisées dans les clauses WHERE | Tables très petites |
| Colonnes utilisées pour les jointures (clés étrangères) | Colonnes rarement utilisées dans les recherches |
| Colonnes utilisées dans les clauses ORDER BY et GROUP BY | Colonnes fréquemment mises à jour |
| Colonnes avec une grande cardinalité (valeurs uniques) | Colonnes avec peu de valeurs distinctes |

### Conseils pour une indexation efficace

1. **Soyez sélectif** : N'indexez que les colonnes nécessaires.
2. **Attention à l'ordre des colonnes** dans un index composé (placez les colonnes les plus sélectives en premier).
3. **Surveillez la taille des index** : Trop d'index peuvent ralentir les opérations d'écriture.
4. **Analysez régulièrement l'utilisation des index** pour supprimer ceux qui sont inutilisés.

## 11.5.2. Connection pooling

Le connection pooling (ou groupement de connexions) est une technique qui réutilise les connexions aux bases de données au lieu d'en créer de nouvelles à chaque fois, ce qui réduit considérablement le temps d'établissement de connexion.

### Comment fonctionne le connection pooling

1. Lorsqu'une connexion est fermée, elle est retournée au pool au lieu d'être réellement fermée
2. Lorsqu'une nouvelle connexion est demandée, une connexion du pool est réutilisée
3. Les connexions sont réinitialisées avant réutilisation pour garantir un état propre

### Configuration du connection pooling avec EF Core et MySQL

Le connection pooling est activé par défaut dans les providers MySQL pour Entity Framework Core. Vous pouvez configurer ses paramètres via la chaîne de connexion :

```csharp
var connectionString =
    "Server=localhost;Database=ma_boutique;User=root;Password=pwd;" +
    "Pooling=true;" +               // Activer le pooling (par défaut)
    "MinimumPoolSize=5;" +          // Nombre minimum de connexions
    "MaximumPoolSize=100;" +        // Nombre maximum de connexions
    "ConnectionLifeTime=300;" +     // Durée de vie maximale en secondes
    "ConnectionTimeout=30;" +       // Temps d'attente avant échec
    "ConnectionReset=true;" +       // Réinitialiser la connexion à la sortie du pool
    "DefaultCommandTimeout=60;";    // Délai d'expiration des commandes
```

### Bonnes pratiques pour le connection pooling

1. **Fermez toujours vos connexions** : Utilisez `using` pour garantir que les connexions sont retournées au pool.

```csharp
using (var context = new MaBoutiqueContext())
{
    // Utiliser le contexte...
} // Le contexte et sa connexion sont automatiquement remis dans le pool
```

2. **Configurez la taille du pool** en fonction de votre charge :
   - Trop petit : attente de connexions disponibles
   - Trop grand : trop de ressources utilisées sur le serveur

3. **Évitez de maintenir des connexions ouvertes trop longtemps** :

```csharp
// À éviter
var context = new MaBoutiqueContext();
// ... du code qui s'exécute pendant longtemps ...
context.SaveChanges();
context.Dispose();
```

4. **Surveillez l'utilisation du pool** pour détecter les fuites de connexion.

## 11.5.3. Réglage des requêtes

L'optimisation des requêtes est essentielle pour obtenir les meilleures performances avec MySQL/MariaDB.

### Stratégies pour optimiser les requêtes LINQ

1. **Sélectionnez uniquement les données dont vous avez besoin** :

```csharp
// Éviter
var clients = await context.Clients.ToListAsync();

// Préférer
var clientsResumes = await context.Clients
    .Select(c => new { c.Id, c.Nom, c.Email })
    .ToListAsync();
```

2. **Filtrez les données côté serveur, pas côté client** :

```csharp
// Inefficace (filtrage côté client)
var produitsActifs = context.Produits
    .ToList()
    .Where(p => p.Statut == "Actif");

// Efficace (filtrage côté serveur)
var produitsActifs = context.Produits
    .Where(p => p.Statut == "Actif")
    .ToList();
```

3. **Utilisez les méthodes asynchrones** pour de meilleures performances :

```csharp
// Synchrone - bloque le thread
var produits = context.Produits.ToList();

// Asynchrone - libère le thread pendant l'attente
var produits = await context.Produits.ToListAsync();
```

4. **Chargez les entités associées efficacement** :

```csharp
// Inefficace (problème N+1)
var commandes = await context.Commandes.ToListAsync();
foreach (var commande in commandes)
{
    // Une requête par commande pour charger le client!
    var client = await context.Clients.FindAsync(commande.ClientId);
}

// Efficace (chargement anticipé)
var commandes = await context.Commandes
    .Include(c => c.Client)
    .ToListAsync();
```

5. **Utilisez des requêtes divisées pour les grandes charges de données** :

```csharp
// Utiliser AsSplitQuery pour éviter les jointures gigantesques
var blogs = await context.Blogs
    .Include(b => b.Posts)
    .ThenInclude(p => p.Comments)
    .AsSplitQuery()
    .ToListAsync();
```

### Optimisation des requêtes SQL générées

Pour les cas complexes, vous pouvez analyser et optimiser le SQL généré :

```csharp
// Voir le SQL généré
var query = context.Produits.Where(p => p.Prix > 100);
var sql = query.ToQueryString(); // À partir d'EF Core 3.0
Console.WriteLine(sql);

// Utiliser du SQL direct pour des requêtes très optimisées
var produitsCouteux = await context.Produits
    .FromSqlRaw("SELECT * FROM Produits WHERE Prix > 100 LIMIT 10")
    .ToListAsync();
```

### Optimisation avec les requêtes non-suivies

Si vous n'avez besoin que de lire des données sans les modifier, désactivez le suivi des entités :

```csharp
// Sans suivi - plus rapide pour les requêtes en lecture seule
var produits = await context.Produits
    .AsNoTracking()
    .ToListAsync();
```

### Pagination efficace

Implémentez la pagination pour limiter le volume de données retournées :

```csharp
// Pagination optimisée
public async Task<List<Produit>> ObtenirProduitsPagines(int page, int taillePage)
{
    return await context.Produits
        .OrderBy(p => p.Nom)
        .Skip((page - 1) * taillePage)
        .Take(taillePage)
        .ToListAsync();
}
```

## 11.5.4. Profiling et diagnostics

Pour optimiser efficacement vos performances, vous devez d'abord identifier les problèmes. Les outils de profiling et de diagnostics sont indispensables.

### Journalisation des requêtes avec EF Core

EF Core permet de journaliser les requêtes SQL générées :

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseMySql(connectionString, ServerVersion.AutoDetect(connectionString))
        // Journal des requêtes dans la console
        .LogTo(Console.WriteLine, LogLevel.Information)
        // Journaliser les paramètres (attention aux données sensibles)
        .EnableSensitiveDataLogging()
        // Messages d'erreur détaillés
        .EnableDetailedErrors();
}
```

Dans une application ASP.NET Core, vous pouvez configurer la journalisation dans `appsettings.json` :

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

### Utilisation de MySQL Slow Query Log

MySQL/MariaDB offre un journal des requêtes lentes pour identifier les requêtes problématiques :

```sql
-- Activer le journal des requêtes lentes
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';
SET GLOBAL long_query_time = 1; -- Journaliser les requêtes prenant plus d'1 seconde
```

### Outils de diagnostic MySQL

MySQL propose plusieurs outils pour analyser les performances :

1. **EXPLAIN** : Affiche le plan d'exécution d'une requête

```sql
-- Analyser une requête
EXPLAIN SELECT * FROM Produits WHERE CategorieId = 5 ORDER BY Prix;
```

2. **SHOW PROFILE** : Détaille le temps passé dans différentes étapes d'une requête

```sql
-- Activer le profilage
SET profiling = 1;

-- Exécuter votre requête
SELECT * FROM Produits WHERE CategorieId = 5;

-- Voir le profil
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;
```

3. **Performance Schema** : Outil avancé pour surveiller les performances du serveur MySQL

```sql
-- Exemple d'utilisation de Performance Schema
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY sum_timer_wait DESC LIMIT 10;
```

### Extensions de profilage tierces pour EF Core

Plusieurs outils tiers peuvent aider à profiler EF Core :

1. **MiniProfiler** : Intégration facile dans ASP.NET Core

```csharp
// Dans Program.cs ou Startup.cs
services.AddMiniProfiler(options =>
{
    options.RouteBasePath = "/profiler";
}).AddEntityFramework();

// Utilisation
using (MiniProfiler.Current.Step("Opération"))
{
    // Code à profiler
}
```

2. **Glimpse** : Fournit des informations détaillées sur les performances
3. **Entity Framework Core Profiler** (payant mais puissant)

## 11.5.5. Mise en cache

La mise en cache est une technique d'optimisation qui consiste à stocker temporairement des données fréquemment accédées pour éviter de répéter des opérations coûteuses.

### Types de mise en cache

1. **Mise en cache des résultats de requêtes**
2. **Mise en cache des objets métier**
3. **Mise en cache des pages ou fragments de page web**

### Mise en cache en mémoire avec .NET

Le package `Microsoft.Extensions.Caching.Memory` fournit un cache en mémoire simple :

```csharp
// Configuration
services.AddMemoryCache();

// Injection
private readonly IMemoryCache _cache;

public ProduitService(IMemoryCache cache)
{
    _cache = cache;
}

// Utilisation
public async Task<List<Produit>> ObtenirProduitsCategorieAvecCache(int categorieId)
{
    string cacheKey = $"produits_cat_{categorieId}";

    // Essayer de récupérer du cache
    if (!_cache.TryGetValue(cacheKey, out List<Produit> produits))
    {
        // Si pas en cache, charger depuis la base
        produits = await _context.Produits
            .Where(p => p.CategorieId == categorieId)
            .ToListAsync();

        // Stocker dans le cache pour 10 minutes
        var cacheOptions = new MemoryCacheEntryOptions()
            .SetAbsoluteExpiration(TimeSpan.FromMinutes(10));

        _cache.Set(cacheKey, produits, cacheOptions);
    }

    return produits;
}
```

### Mise en cache distribuée

Pour les applications multi-serveurs, utilisez un cache distribué comme Redis :

```csharp
// Installation: Microsoft.Extensions.Caching.StackExchangeRedis

// Configuration
services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "MaBoutique_";
});

// Injection
private readonly IDistributedCache _cache;

public ProduitService(IDistributedCache cache)
{
    _cache = cache;
}

// Utilisation
public async Task<List<Produit>> ObtenirProduitsCategorieAvecCacheDistribue(int categorieId)
{
    string cacheKey = $"produits_cat_{categorieId}";

    // Essayer de récupérer du cache
    string cachedJson = await _cache.GetStringAsync(cacheKey);

    List<Produit> produits;

    if (string.IsNullOrEmpty(cachedJson))
    {
        // Si pas en cache, charger depuis la base
        produits = await _context.Produits
            .Where(p => p.CategorieId == categorieId)
            .ToListAsync();

        // Sérialiser et stocker dans le cache
        string json = JsonSerializer.Serialize(produits);

        var cacheOptions = new DistributedCacheEntryOptions()
            .SetAbsoluteExpiration(TimeSpan.FromMinutes(10));

        await _cache.SetStringAsync(cacheKey, json, cacheOptions);
    }
    else
    {
        // Désérialiser depuis le cache
        produits = JsonSerializer.Deserialize<List<Produit>>(cachedJson);
    }

    return produits;
}
```

### Invalidation du cache

Il est crucial d'invalider le cache lorsque les données sous-jacentes changent :

```csharp
// Invalidation lors de la modification de données
public async Task MettreAJourProduit(Produit produit)
{
    _context.Produits.Update(produit);
    await _context.SaveChangesAsync();

    // Invalider les caches concernés
    string cacheKey = $"produits_cat_{produit.CategorieId}";
    _cache.Remove(cacheKey);

    // Aussi invalider la liste complète
    _cache.Remove("tous_produits");
}
```

### Le cache de second niveau d'EF Core

EF Core ne fournit pas de cache de second niveau intégré, mais vous pouvez utiliser des packages tiers comme `EFCoreSecondLevelCacheInterceptor` :

```csharp
// Installation: EFCoreSecondLevelCacheInterceptor

// Configuration
services.AddEFSecondLevelCache(options =>
    options.UseMemoryCacheProvider(CacheExpirationMode.Absolute, TimeSpan.FromMinutes(10))
);

// Dans la configuration du DbContext
optionsBuilder
    .UseMySql(connectionString, ServerVersion.AutoDetect(connectionString))
    .AddInterceptors(serviceProvider.GetRequiredService<SecondLevelCacheInterceptor>());

// Utilisation
var produitsEnCache = await _context.Produits
    .Where(p => p.CategorieId == 5)
    .Cacheable() // Cette requête sera mise en cache
    .ToListAsync();
```

### Stratégies de mise en cache

1. **Identifiez les données à mettre en cache** :
   - Données accédées fréquemment
   - Données peu modifiées
   - Données coûteuses à récupérer

2. **Choisissez la bonne durée de cache** :
   - Trop courte : avantage minimal
   - Trop longue : risque de données obsolètes

3. **Mettez en place une stratégie d'invalidation** :
   - Invalidation basée sur le temps
   - Invalidation explicite lors des modifications
   - Invalidation par dépendances

4. **Surveillez l'utilisation du cache** :
   - Taux de réussite (hit ratio)
   - Consommation de mémoire
   - Impact sur les performances

## Bonnes pratiques générales pour les performances

1. **Planifiez l'optimisation dès la conception** - N'attendez pas les problèmes
2. **Mesurez avant d'optimiser** - Identifiez les vrais goulots d'étranglement
3. **Optimisez par ordre d'impact** - Commencez par ce qui apporte le plus de gains
4. **Testez avec des volumes réalistes** - Les problèmes de performance apparaissent souvent à l'échelle
5. **Équilibrez complexité et gains** - Parfois, une solution simple mais moins optimale est préférable
6. **Surveillez en continu** - Les performances peuvent se dégrader avec le temps

En suivant ces conseils et en utilisant judicieusement les techniques d'indexation, de connection pooling, d'optimisation des requêtes, de profiling et de mise en cache, vous pourrez créer des applications MySQL/MariaDB performantes et évolutives.

⏭️ 11.6. [Sécurité et bonnes pratiques](/11-integration-avec-mysql-mariadb/11-6-securite-et-bonnes-pratiques.md)
