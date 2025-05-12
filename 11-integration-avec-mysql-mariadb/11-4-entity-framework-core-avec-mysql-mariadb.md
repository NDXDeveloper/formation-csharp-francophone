# 11.4. Entity Framework Core avec MySQL/MariaDB

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Entity Framework Core (EF Core) est un puissant ORM (Object-Relational Mapper) qui permet de manipuler des bases de données relationnelles à l'aide de classes C#. Dans cette section, nous allons explorer comment utiliser EF Core spécifiquement avec MySQL et MariaDB, en nous concentrant sur les configurations et fonctionnalités propres à ces systèmes de gestion de base de données.

## 11.4.1. Configuration spécifique

La configuration d'Entity Framework Core pour MySQL/MariaDB nécessite quelques étapes spécifiques pour garantir un fonctionnement optimal.

### Installation des packages nécessaires

Pour utiliser EF Core avec MySQL/MariaDB, commencez par installer les packages NuGet appropriés :

```bash
# Utilisation du provider Pomelo (recommandé)
dotnet add package Pomelo.EntityFrameworkCore.MySql

# Pour les outils de migration et de scaffolding
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Configuration du DbContext

La configuration du `DbContext` pour MySQL/MariaDB nécessite de spécifier le provider et la version du serveur :

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<Produit> Produits { get; set; }
    public DbSet<Categorie> Categories { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Chaîne de connexion MySQL
        var connectionString = "server=localhost;port=3306;database=ma_boutique;user=root;password=mot2passe";

        // Version du serveur MySQL
        var serverVersion = new MySqlServerVersion(new Version(8, 0, 28));

        // Pour MariaDB, utilisez :
        // var serverVersion = new MariaDbServerVersion(new Version(10, 6, 7));

        optionsBuilder.UseMySql(connectionString, serverVersion)
            // Options de débogage (à désactiver en production)
            .LogTo(Console.WriteLine, LogLevel.Information)
            .EnableSensitiveDataLogging()
            .EnableDetailedErrors();
    }
}
```

### Détection automatique de la version du serveur

Vous pouvez également détecter automatiquement la version du serveur :

```csharp
// Détection automatique (pratique mais nécessite une connexion au démarrage)
optionsBuilder.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));
```

### Configuration dans une application ASP.NET Core

Pour une application ASP.NET Core, configurez le DbContext dans `Program.cs` (ou `Startup.cs` pour les versions antérieures) :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var connectionString = Configuration.GetConnectionString("DefaultConnection");
    var serverVersion = new MySqlServerVersion(new Version(8, 0, 28));

    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseMySql(connectionString, serverVersion));
}
```

### Options spécifiques à MySQL/MariaDB

EF Core avec MySQL/MariaDB prend en charge plusieurs options spécifiques :

```csharp
optionsBuilder.UseMySql(connectionString, serverVersion, options =>
{
    // Définir le délai maximal d'exécution des commandes
    options.CommandTimeout(60); // 60 secondes

    // Configurer la tolérance aux erreurs/tentatives de reconnexion
    options.EnableRetryOnFailure(
        maxRetryCount: 5,
        maxRetryDelay: TimeSpan.FromSeconds(30),
        errorNumbersToAdd: null);

    // Désactiver la vérification des variations du schéma
    options.DisableBackslashEscaping();

    // Configurer le comportement des clés étrangères
    options.DefaultDataTypeMappings(mappings =>
        mappings.WithClrBoolean(MySqlBooleanType.TinyInt1));
});
```

## 11.4.2. Migrations

Les migrations sont un moyen puissant de faire évoluer votre schéma de base de données en synchronisation avec votre modèle de données C#. MySQL et MariaDB présentent quelques particularités à prendre en compte.

### Création et application de migrations

La création et l'application de migrations fonctionnent comme pour les autres bases de données :

```bash
# Création d'une migration initiale
dotnet ef migrations add InitialCreate

# Application de la migration à la base de données
dotnet ef database update
```

### Spécificités MySQL/MariaDB dans les migrations

Les migrations pour MySQL/MariaDB présentent quelques spécificités :

1. **Moteur de stockage** : Par défaut, EF Core utilise InnoDB pour MySQL, mais vous pouvez le modifier :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "Produits",
        columns: table => new
        {
            // Définition des colonnes...
        },
        constraints: table =>
        {
            // Définition des contraintes...
        })
        .Annotation("MySQL:Engine", "InnoDB");  // Spécifier explicitement le moteur
}
```

2. **Collation et jeu de caractères** : Vous pouvez spécifier la collation et le jeu de caractères :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "Produits",
        columns: table => new
        {
            // Définition des colonnes...
        })
        .Annotation("MySQL:Charset", "utf8mb4")
        .Annotation("MySQL:Collation", "utf8mb4_unicode_ci");
}
```

3. **Personnalisation avec le SQL brut** : Pour des fonctionnalités spécifiques à MySQL :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // Migration standard générée par EF Core

    // Ajouter du SQL spécifique à MySQL
    migrationBuilder.Sql(@"
        ALTER TABLE Produits ADD FULLTEXT INDEX IX_Produits_Nom_Description (Nom, Description);
    ");
}
```

### Gestion des types de données spécifiques

MySQL et MariaDB utilisent des types de données qui peuvent nécessiter une attention particulière dans les migrations :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configuration du type ENUM
    modelBuilder.Entity<Produit>()
        .Property(p => p.Statut)
        .HasColumnType("ENUM('Actif', 'Inactif', 'Épuisé')")
        .IsRequired();

    // Configuration pour le type JSON (MySQL 5.7+ ou MariaDB 10.2+)
    modelBuilder.Entity<Produit>()
        .Property(p => p.Attributs)
        .HasColumnType("json");

    // Configuration pour les types temporels
    modelBuilder.Entity<Produit>()
        .Property(p => p.DateCreation)
        .HasColumnType("timestamp")
        .HasDefaultValueSql("CURRENT_TIMESTAMP");

    modelBuilder.Entity<Produit>()
        .Property(p => p.DateModification)
        .HasColumnType("timestamp")
        .HasDefaultValueSql("CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP");
}
```

### Script de migration pour la production

Pour les environnements de production, il est recommandé de générer un script SQL plutôt que d'appliquer les migrations directement :

```bash
# Générer un script SQL à partir des migrations
dotnet ef migrations script -o migration.sql

# Ou générer un script à partir d'une migration spécifique
dotnet ef migrations script NomMigrationPrecedente NomMigrationCible -o migration.sql
```

Ce script peut ensuite être révisé et exécuté pendant une fenêtre de maintenance planifiée.

## 11.4.3. Scénarios avancés

Entity Framework Core offre plusieurs fonctionnalités avancées qui peuvent être particulièrement utiles avec MySQL/MariaDB.

### Optimisation des performances

MySQL et MariaDB peuvent nécessiter des optimisations spécifiques pour de meilleures performances :

1. **Utilisation de requêtes divisées (Split Queries)** pour éviter les jointures volumineuses :

```csharp
var blogAvecPosts = await context.Blogs
    .Include(b => b.Posts)
    .AsSplitQuery() // Exécute des requêtes séparées pour Blogs et Posts
    .FirstOrDefaultAsync(b => b.Id == id);
```

2. **Pagination efficace** pour les grands ensembles de données :

```csharp
// Pagination optimisée pour MySQL
var pageIndex = 2; // Page 2 (base 0)
var pageSize = 20; // 20 éléments par page

var produits = await context.Produits
    .OrderBy(p => p.Nom)
    .Skip(pageIndex * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

3. **Chargement sélectif des propriétés** pour réduire le volume de données :

```csharp
var produitsSummary = await context.Produits
    .Select(p => new { p.Id, p.Nom, p.Prix })
    .ToListAsync();
```

### Fonctionnalités de recherche avancées

MySQL et MariaDB offrent des fonctionnalités de recherche de texte avancées :

```csharp
// Recherche avec LIKE (fonctionne partout)
var resultats = await context.Produits
    .Where(p => EF.Functions.Like(p.Nom, $"%{motCle}%"))
    .ToListAsync();

// Recherche en texte intégral (nécessite un index FULLTEXT)
var resultatsFullText = await context.Produits
    .FromSqlRaw("SELECT * FROM Produits WHERE MATCH(Nom, Description) AGAINST({0} IN BOOLEAN MODE)", motCle)
    .ToListAsync();
```

Pour configurer un index FULLTEXT, utilisez une migration personnalisée :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("ALTER TABLE Produits ADD FULLTEXT INDEX FT_Produits_Nom_Description (Nom, Description)");
}
```

### Transactions et verrouillage

MySQL/MariaDB gère les transactions et le verrouillage de manière spécifique :

```csharp
// Transaction explicite avec niveau d'isolation
using (var transaction = await context.Database.BeginTransactionAsync(
    System.Data.IsolationLevel.ReadCommitted))
{
    try
    {
        // Opérations dans la transaction...

        // Verrouillage explicite (SQL brut pour MySQL)
        var produit = await context.Produits
            .FromSqlRaw("SELECT * FROM Produits WHERE Id = {0} FOR UPDATE", produitId)
            .FirstOrDefaultAsync();

        // Modifier le produit...
        produit.Stock -= quantite;

        // Sauvegarder et valider
        await context.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

### Haute disponibilité et réplication

Pour les scénarios de haute disponibilité avec MySQL/MariaDB, vous pouvez configurer plusieurs serveurs :

```csharp
var connectionString =
    "server=primary.example.com,replica1.example.com,replica2.example.com;" +
    "port=3306;database=ma_boutique;user=app_user;password=mot2passe;";

// Configuration pour lire à partir des répliques
options.UseMySql(connectionString, serverVersion,
    mysqlOptions => mysqlOptions.EnableRetryOnFailure());
```

## 11.4.4. Fonctionnalités spécifiques à MySQL/MariaDB

MySQL et MariaDB offrent certaines fonctionnalités spécifiques qui peuvent être exploitées avec EF Core.

### Utilisation de JSON

MySQL 5.7+ et MariaDB 10.2+ prennent en charge le type de données JSON natif :

```csharp
// Modèle avec propriété JSON
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }

    // Stocké comme JSON dans la base de données
    public Dictionary<string, object> Attributs { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Produit>()
        .Property(p => p.Attributs)
        .HasColumnType("json");
}

// Requêtes avec JSON
var produitsAvecAttributs = await context.Produits
    .FromSqlRaw("SELECT * FROM Produits WHERE JSON_EXTRACT(Attributs, '$.couleur') = 'rouge'")
    .ToListAsync();
```

Pour des opérations plus avancées, vous pouvez utiliser des fonctions JSON directement :

```csharp
// Extraction de valeur JSON avec SQL brut
var produitsRougeTaille = await context.Produits
    .FromSqlRaw(@"
        SELECT * FROM Produits
        WHERE JSON_EXTRACT(Attributs, '$.couleur') = 'rouge'
        AND JSON_EXTRACT(Attributs, '$.taille') = 'M'")
    .ToListAsync();
```

### Utilisation des ENUM MySQL

MySQL et MariaDB prennent en charge le type ENUM, que vous pouvez utiliser avec EF Core :

```csharp
// Dans le modèle C#
public enum StatutProduit
{
    Actif,
    Inactif,
    Epuise
}

public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public StatutProduit Statut { get; set; }
}

// Configuration avec Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Produit>()
        .Property(p => p.Statut)
        .HasColumnType("ENUM('Actif', 'Inactif', 'Epuise')")
        .HasConversion<string>();
}
```

### Types géographiques

MySQL et MariaDB prennent en charge les types de données spatiales (géographiques) :

```csharp
// Modèle avec propriété spatiale
public class Magasin
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public Point Localisation { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Magasin>()
        .Property(m => m.Localisation)
        .HasColumnType("point");
}

// Création d'un point
var magasin = new Magasin
{
    Nom = "Magasin Centre",
    Localisation = new Point(48.8566, 2.3522) { SRID = 4326 } // Paris avec SRID standard
};

// Requête spatiale
var magasinsProches = await context.Magasins
    .FromSqlRaw(@"
        SELECT * FROM Magasins
        WHERE ST_Distance_Sphere(Localisation, POINT({0}, {1})) < {2}",
        longitude, latitude, distanceEnMetres)
    .ToListAsync();
```

### Auto-incrémentation personnalisée

MySQL/MariaDB permet de configurer le comportement de l'auto-incrémentation :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Produit>()
        .Property(p => p.Id)
        .ValueGeneratedOnAdd()
        .HasAnnotation("MySQL:ValueGenerationStrategy", MySQLValueGenerationStrategy.IdentityColumn)
        .HasAnnotation("MySQL:AutoIncrement", "10000") // Commence à 10000
        .HasAnnotation("MySQL:AutoIncrementIncrement", "5"); // Incrémente de 5
}
```

### Utilisation de procédures stockées pour les opérations CRUD

MySQL/MariaDB prend en charge les procédures stockées qui peuvent être utilisées pour les opérations CRUD avec EF Core :

```csharp
// Appel de procédure stockée pour récupérer des produits
var produits = await context.Produits
    .FromSqlRaw("CALL GetProduitsByCategorie({0})", categorieId)
    .ToListAsync();

// Appel de procédure stockée pour mise à jour
var resultat = await context.Database
    .ExecuteSqlRawAsync("CALL UpdateProduitPrix({0}, {1})", produitId, nouveauPrix);
```

Pour créer une procédure stockée dans une migration :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql(@"
        CREATE PROCEDURE GetProduitsByCategorie(IN cat_id INT)
        BEGIN
            SELECT * FROM Produits WHERE CategorieId = cat_id;
        END;
    ");

    migrationBuilder.Sql(@"
        CREATE PROCEDURE UpdateProduitPrix(IN prod_id INT, IN nouveau_prix DECIMAL(10,2))
        BEGIN
            UPDATE Produits SET Prix = nouveau_prix WHERE Id = prod_id;
        END;
    ");
}
```

## Bonnes pratiques pour EF Core avec MySQL/MariaDB

1. **Utilisez la version appropriée du provider** pour votre version de MySQL/MariaDB
2. **Configurez correctement le jeu de caractères et la collation** (utf8mb4 recommandé)
3. **Créez des index** pour optimiser les performances des requêtes fréquentes
4. **Testez vos migrations** avant de les appliquer en production
5. **Utilisez les types de données appropriés** pour optimiser l'espace de stockage
6. **Activez le pooling de connexions** pour améliorer les performances
7. **Surveillez les performances** de vos requêtes les plus fréquentes
8. **Utilisez des transactions** pour les opérations interdépendantes

En suivant ces conseils et en exploitant les fonctionnalités spécifiques de MySQL et MariaDB, vous pouvez créer des applications performantes et fiables avec Entity Framework Core.

⏭️
