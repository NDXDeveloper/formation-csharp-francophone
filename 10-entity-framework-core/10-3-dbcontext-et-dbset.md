# 10.3. DbContext et DbSet

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Au cœur d'Entity Framework Core se trouvent deux classes fondamentales : `DbContext` et `DbSet<T>`. Comprendre ces classes est essentiel pour maîtriser EF Core. Cette section vous guidera à travers leur utilisation, de la configuration de base à des scénarios plus avancés.

## 10.3.1. Configuration du contexte

Le `DbContext` est la classe principale qui coordonne les fonctionnalités d'Entity Framework pour un modèle de données. Il représente une session avec la base de données et sert de "pont" entre vos classes d'entités et la base de données.

### Création d'un contexte de base

```csharp
public class BibliothequeContext : DbContext
{
    // DbSet représente une table dans la base de données
    public DbSet<Livre> Livres { get; set; }
    public DbSet<Auteur> Auteurs { get; set; }
    public DbSet<Categorie> Categories { get; set; }

    // Configuration de la connexion à la base de données
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=.;Database=Bibliotheque;Trusted_Connection=True;");
    }
}
```

### Configuration dans ASP.NET Core

Dans les applications ASP.NET Core, la configuration du contexte se fait généralement dans la méthode `ConfigureServices` du fichier `Startup.cs` (pour .NET 5 et versions antérieures) ou dans `Program.cs` (pour .NET 6+) :

```csharp
// Program.cs (.NET 6+)
builder.Services.AddDbContext<BibliothequeContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

Avec la chaîne de connexion définie dans `appsettings.json` :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=Bibliotheque;Trusted_Connection=True;"
  }
}
```

### Options de configuration avancées

Vous pouvez personnaliser davantage le comportement de votre contexte :

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer("Server=.;Database=Bibliotheque;Trusted_Connection=True;",
            sqlOptions =>
            {
                sqlOptions.EnableRetryOnFailure(
                    maxRetryCount: 5,
                    maxRetryDelay: TimeSpan.FromSeconds(30),
                    errorNumbersToAdd: null);

                sqlOptions.CommandTimeout(60); // 60 secondes
            })
        .LogTo(Console.WriteLine, LogLevel.Information) // Journalisation des requêtes SQL
        .EnableSensitiveDataLogging() // Afficher les valeurs des paramètres (à éviter en production)
        .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking); // Désactiver le tracking par défaut
}
```

### Constructeurs du contexte

Il est recommandé de définir plusieurs constructeurs pour faciliter différents scénarios :

```csharp
public class BibliothequeContext : DbContext
{
    public DbSet<Livre> Livres { get; set; }
    public DbSet<Auteur> Auteurs { get; set; }

    // Constructeur sans paramètre pour les migrations et certains scénarios
    public BibliothequeContext()
    {
    }

    // Constructeur pour l'injection de dépendances (ASP.NET Core)
    public BibliothequeContext(DbContextOptions<BibliothequeContext> options)
        : base(options)
    {
    }

    // Configuration si elle n'est pas déjà spécifiée par options
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseSqlServer("Server=.;Database=Bibliotheque;Trusted_Connection=True;");
        }
    }
}
```

## 10.3.2. Gestion du cycle de vie

La gestion correcte du cycle de vie du `DbContext` est cruciale pour les performances et la fiabilité de votre application.

### Durée de vie recommandée

Le `DbContext` est conçu pour être de **courte durée** (instance par requête/opération) :

```csharp
// Utilisation typique avec using pour garantir la libération des ressources
using (var context = new BibliothequeContext())
{
    var livres = context.Livres.ToList();
    // Traitement des livres...
}
// À cette ligne, le contexte est automatiquement libéré
```

### Pourquoi une instance par requête ?

1. **Suivi des entités** : Le contexte garde en mémoire toutes les entités suivies (tracked)
2. **Isolation des modifications** : Chaque contexte gère son propre ensemble de modifications
3. **Performance** : Un contexte à longue durée de vie peut accumuler des entités en mémoire
4. **Concurrence** : Le contexte n'est pas thread-safe

### Cycle de vie dans différents types d'applications

#### Applications Console/Desktop
```csharp
static void AfficherLivresParCategorie(string categorie)
{
    using (var context = new BibliothequeContext())
    {
        var livres = context.Livres
            .Where(l => l.Categorie.Nom == categorie)
            .ToList();

        foreach (var livre in livres)
        {
            Console.WriteLine($"{livre.Titre} - {livre.Auteur.Nom}");
        }
    }
}
```

#### ASP.NET Core (MVC/API)

Dans ASP.NET Core, le cycle de vie est généralement géré par le conteneur d'injection de dépendances :

```csharp
// Durée de vie par défaut : Scoped (une instance par requête HTTP)
builder.Services.AddDbContext<BibliothequeContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Dans un contrôleur
public class LivresController : Controller
{
    private readonly BibliothequeContext _context;

    // Injection du contexte
    public LivresController(BibliothequeContext context)
    {
        _context = context;
    }

    public async Task<IActionResult> Index()
    {
        var livres = await _context.Livres
            .Include(l => l.Auteur)
            .ToListAsync();

        return View(livres);
    }
}
```

### Gestion des erreurs et connexions

```csharp
try
{
    using (var context = new BibliothequeContext())
    {
        // Opérations de base de données...
        context.SaveChanges();
    }
}
catch (DbUpdateException ex)
{
    // Gérer les erreurs spécifiques à la mise à jour
    Console.WriteLine($"Erreur de mise à jour: {ex.Message}");
    // Examiner ex.InnerException pour plus de détails
}
catch (Exception ex)
{
    // Gérer les autres erreurs
    Console.WriteLine($"Erreur: {ex.Message}");
}
```

## 10.3.3. Fluent API vs annotations

EF Core offre deux approches principales pour configurer vos entités et leurs relations : **les annotations de données** et la **Fluent API**.

### Annotations de données

Les annotations sont des attributs appliqués directement aux classes et propriétés :

```csharp
public class Livre
{
    [Key]
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Titre { get; set; }

    [Column("AnneePublication")]
    public int Annee { get; set; }

    [Range(0, 5000)]
    public decimal Prix { get; set; }

    [ForeignKey("Auteur")]
    public int AuteurId { get; set; }

    public Auteur Auteur { get; set; }
}
```

Annotations courantes :
- `[Key]` : Définit la clé primaire
- `[Required]` : Spécifie qu'une propriété est obligatoire
- `[StringLength(x)]` : Limite la longueur d'une chaîne
- `[Column("Nom")]` : Spécifie le nom de la colonne dans la base de données
- `[Table("Nom")]` : Spécifie le nom de la table
- `[ForeignKey("Nom")]` : Définit une clé étrangère
- `[NotMapped]` : Exclut une propriété du mappage

### Fluent API

La Fluent API offre une approche plus puissante et flexible dans la méthode `OnModelCreating` :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Livre>(entity =>
    {
        entity.ToTable("Livres");

        entity.HasKey(e => e.Id);

        entity.Property(e => e.Titre)
            .IsRequired()
            .HasMaxLength(100);

        entity.Property(e => e.Annee)
            .HasColumnName("AnneePublication");

        entity.Property(e => e.Prix)
            .HasPrecision(10, 2);

        entity.HasOne(e => e.Auteur)
            .WithMany(a => a.Livres)
            .HasForeignKey(e => e.AuteurId)
            .OnDelete(DeleteBehavior.Restrict);
    });

    modelBuilder.Entity<Auteur>(entity =>
    {
        entity.ToTable("Auteurs");
        entity.HasKey(e => e.Id);

        entity.Property(e => e.Nom)
            .IsRequired()
            .HasMaxLength(50);
    });
}
```

### Comparaison et recommandations

| Aspect | Annotations | Fluent API |
|--------|-------------|------------|
| **Simplicité** | Plus simple, directement sur les classes | Plus verbeux, séparé des classes |
| **Puissance** | Fonctionnalités limitées | Configuration complète et avancée |
| **Séparation** | Mélange le modèle et sa configuration | Sépare le modèle de sa configuration |
| **Maintenance** | Plus facile pour configurations simples | Meilleure pour configurations complexes |
| **Héritage** | Limitations avec classes abstraites | Support complet |

**Recommandations pour débutants :**

1. **Commencez avec les annotations** pour les configurations simples
2. **Passez à la Fluent API** pour les configurations plus complexes ou lorsque :
   - Vous configurez des relations avancées
   - Vous ne pouvez pas modifier les classes d'entités
   - Vous avez besoin de configurations qui ne sont pas disponibles via annotations

3. **Combinaison** : Il est parfaitement valide (et courant) d'utiliser une combinaison des deux approches

## 10.3.4. Conventions de nommage

Entity Framework Core utilise des conventions pour réduire la quantité de configuration nécessaire. Comprendre ces conventions peut vous faire gagner du temps.

### Conventions par défaut

1. **Tables et DbSets**
   - Une propriété `DbSet<T>` génère une table nommée d'après la propriété
   - `public DbSet<Livre> Livres { get; set; }` → Table `Livres`

2. **Clés primaires**
   - Les propriétés nommées `Id` ou `[TypeName]Id` sont automatiquement des clés primaires
   - `public int Id { get; set; }` ou `public int LivreId { get; set; }`

3. **Relations**
   - Une propriété de navigation avec une propriété de clé étrangère correspondante établit automatiquement une relation
   - `public Auteur Auteur { get; set; }` avec `public int AuteurId { get; set; }` → Relation

4. **Noms de colonnes**
   - Les noms des colonnes correspondent par défaut aux noms des propriétés
   - `public string Titre { get; set; }` → Colonne `Titre`

### Personnalisation des conventions

Vous pouvez modifier ou étendre les conventions par défaut :

```csharp
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    // Définir la longueur maximale par défaut pour toutes les propriétés string
    configurationBuilder
        .Properties<string>()
        .HaveMaxLength(200);

    // Configurer les propriétés DateTime pour qu'elles soient stockées en UTC
    configurationBuilder
        .Properties<DateTime>()
        .HaveConversion<UtcDateTimeConverter>();

    // Configurer le type de colonne pour les propriétés décimales
    configurationBuilder
        .Properties<decimal>()
        .HavePrecision(18, 6);
}
```

### Bonnes pratiques pour les nommages

Pour une meilleure lisibilité et maintenabilité :

1. **Nommage cohérent**
   - Utilisez des noms au singulier pour les entités : `Livre`, pas `Livres`
   - Utilisez des noms au pluriel pour les DbSets : `Livres`, pas `Livre`

2. **Clés étrangères explicites**
   - Nommez les clés étrangères avec le format `[EntitéParente]Id`
   - `public int AuteurId { get; set; }`

3. **Collections de navigation**
   - Utilisez des noms pluriels pour les collections : `Livres`, pas `Livre`
   - `public List<Livre> Livres { get; set; }`

4. **Évitez les conflits de noms**
   - Évitez de nommer une propriété comme une table/entity : ne pas avoir une propriété `Auteur` dans la classe `Auteur`

### Exemple complet respectant les conventions

```csharp
public class Auteur
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Biographie { get; set; }

    // Collection de navigation (pluriel)
    public List<Livre> Livres { get; set; }
}

public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }
    public int Annee { get; set; }

    // Clé étrangère suivant la convention [EntitéParente]Id
    public int AuteurId { get; set; }

    // Propriété de navigation (singulier)
    public Auteur Auteur { get; set; }

    // Autre relation plusieurs-à-plusieurs
    public List<Categorie> Categories { get; set; }
}

public class Categorie
{
    public int Id { get; set; }
    public string Nom { get; set; }

    // Collection de navigation (pluriel)
    public List<Livre> Livres { get; set; }
}

public class BibliothequeContext : DbContext
{
    // DbSets au pluriel
    public DbSet<Livre> Livres { get; set; }
    public DbSet<Auteur> Auteurs { get; set; }
    public DbSet<Categorie> Categories { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=.;Database=Bibliotheque;Trusted_Connection=True;");
    }
}
```

## Conclusion

Le `DbContext` et les `DbSet<T>` sont les fondations sur lesquelles repose votre travail avec Entity Framework Core. Une bonne compréhension de leur configuration, de leur cycle de vie et des conventions utilisées vous aidera à construire des applications plus robustes et maintenables.

À mesure que vous progressez, n'hésitez pas à explorer des configurations plus avancées, mais commencez par maîtriser les bases présentées dans cette section.

⏭️ 10.4. [Migrations](/10-entity-framework-core/10-4-migrations.md)
