# 10.1. Introduction à Entity Framework Core

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 10.1.1. ORM et ses avantages

**Qu'est-ce qu'un ORM ?**

Un ORM (Object-Relational Mapping) est un outil qui fait le pont entre le monde orienté objet (notre code C#) et le monde relationnel (notre base de données). Il vous permet de manipuler des données stockées dans une base de données relationnelle en utilisant des objets C#, sans avoir à écrire manuellement de requêtes SQL.

**Avantages d'un ORM comme Entity Framework Core :**

1. **Productivité accrue** : Vous écrivez moins de code et concentrez vos efforts sur la logique métier plutôt que sur la plomberie d'accès aux données.

2. **Code plus lisible** : Le code orienté objet est souvent plus facile à comprendre que des chaînes SQL complexes.

   ```csharp
   // Avec EF Core
   var clients = context.Clients
       .Where(c => c.Ville == "Paris")
       .OrderBy(c => c.Nom)
       .ToList();

   // Sans ORM (ADO.NET brut)
   // Beaucoup plus de code pour exécuter une requête SQL,
   // créer des objets Client, etc.
   ```

3. **Indépendance vis-à-vis du SGBD** : Vous pouvez changer de système de base de données (SQL Server, MySQL, SQLite...) avec un minimum de modifications dans votre code.

4. **Sécurité accrue** : EF Core se charge d'éviter les injections SQL en paramétrant automatiquement les requêtes.

5. **Maintenance simplifiée** : Les modifications du modèle de données peuvent être propagées à la base de données via des migrations.

## 10.1.2. Installation et configuration

### Installation d'Entity Framework Core

Pour commencer à utiliser Entity Framework Core, vous devez installer les packages NuGet nécessaires. Voici comment faire :

**1. Avec la console NuGet Package Manager :**
```
Install-Package Microsoft.EntityFrameworkCore
```

Vous devez également installer un provider spécifique pour votre base de données :
- SQL Server : `Microsoft.EntityFrameworkCore.SqlServer`
- SQLite : `Microsoft.EntityFrameworkCore.Sqlite`
- MySQL/MariaDB : `Pomelo.EntityFrameworkCore.MySql`
- PostgreSQL : `Npgsql.EntityFrameworkCore.PostgreSQL`

**2. Avec Visual Studio :**
- Clic droit sur votre projet → Gérer les packages NuGet
- Recherchez et installez les packages mentionnés ci-dessus

**3. Avec la ligne de commande .NET CLI :**
```
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

### Configuration de base

Une fois les packages installés, vous devez créer deux éléments essentiels :

**1. Les classes d'entité** - Elles représentent vos tables en base de données :

```csharp
public class Client
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Email { get; set; }
    public string Ville { get; set; }

    // Relation avec d'autres entités
    public List<Commande> Commandes { get; set; }
}

public class Commande
{
    public int Id { get; set; }
    public DateTime DateCommande { get; set; }
    public decimal Montant { get; set; }

    // Clé étrangère et propriété de navigation
    public int ClientId { get; set; }
    public Client Client { get; set; }
}
```

**2. La classe de contexte** - Elle est le point d'entrée principal pour interagir avec la base de données :

```csharp
public class ApplicationDbContext : DbContext
{
    // Tables dans votre base de données
    public DbSet<Client> Clients { get; set; }
    public DbSet<Commande> Commandes { get; set; }

    // Configuration de la connexion
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(
            "Server=monserveur;Database=mabase;User Id=utilisateur;Password=motdepasse;");
    }
}
```

### Utilisation dans une application ASP.NET Core

Dans les applications ASP.NET Core, la configuration est généralement faite dans la méthode `ConfigureServices` du fichier `Startup.cs` ou directement dans `Program.cs` pour les versions récentes :

```csharp
// Program.cs (.NET 6+)
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

Avec la chaîne de connexion stockée dans `appsettings.json` :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=monserveur;Database=mabase;User Id=utilisateur;Password=motdepasse;"
  }
}
```

## 10.1.3. Différences avec EF6

Entity Framework Core est une réécriture complète du framework Entity Framework 6, conçue pour être plus légère, extensible et moderne. Voici les principales différences :

### Améliorations dans EF Core

1. **Performance** : EF Core est généralement plus rapide et efficace qu'EF6.

2. **Prise en charge multi-plateformes** : EF Core fonctionne sur Windows, Linux et macOS grâce à .NET Core/.NET 5+.

3. **Migrations améliorées** : Le système de migrations est plus robuste et offre une meilleure expérience en ligne de commande.

4. **Requêtes plus efficaces** : EF Core génère souvent un SQL plus optimisé.

5. **Support de nouveaux types de bases de données** : En plus des SGBD relationnels, EF Core peut s'interfacer avec des bases NoSQL (avec des providers tiers).

### Fonctionnalités manquantes ou différentes

1. **Modèle EDMX** : EF Core ne prend pas en charge le designer visuel EDMX disponible dans EF6.

2. **Lazy Loading** : Absent initialement, puis ajouté dans EF Core 2.1 mais fonctionne différemment.

3. **Requêtes complexes** : Certaines requêtes LINQ complexes qui fonctionnaient dans EF6 peuvent nécessiter une réécriture dans EF Core.

4. **Héritage** : Les stratégies d'héritage (TPH, TPT, TPC) ont été implémentées progressivement dans EF Core.

5. **Mise à jour groupée** : Le comportement de certaines opérations comme `SaveChanges()` peut différer.

## 10.1.4. Versions et compatibilité

Entity Framework Core suit généralement le cycle de publication de .NET Core/.NET, avec des nouvelles versions majeures chaque année. Voici un aperçu des versions principales et leurs caractéristiques :

### Versions majeures

| Version EF Core | Version .NET compatible | Caractéristiques notables |
|-----------------|-------------------------|---------------------------|
| EF Core 1.0/1.1 | .NET Core 1.0/1.1      | Version initiale          |
| EF Core 2.0/2.1 | .NET Core 2.0/2.1      | Lazy loading, migrations améliorées |
| EF Core 3.0/3.1 | .NET Core 3.0/3.1      | Requêtes LINQ améliorées, meilleure performance |
| EF Core 5.0     | .NET 5                 | Many-to-many natif, types mappés aux colonnes |
| EF Core 6.0     | .NET 6                 | Requêtes compilées, intercepteurs, migrations améliorées |
| EF Core 7.0     | .NET 6/7               | Bulk updates, JSON columns |
| EF Core 8.0     | .NET 8                 | Date-only et time-only, filtres d'entités au niveau requête |

### Conseils de compatibilité

1. **Alignez les versions** : Utilisez idéalement la version d'EF Core qui correspond à votre version de .NET.

2. **Migration progressive** : Si vous migrez depuis EF6, commencez par identifier les fonctionnalités que vous utilisez et vérifiez leur support dans EF Core.

3. **Breaking changes** : Consultez toujours les notes de version lors d'une mise à jour, car certaines versions peuvent introduire des changements incompatibles.

4. **Support à long terme (LTS)** : Les versions avec un support à long terme (comme EF Core 3.1, 6.0 et 8.0) sont recommandées pour les applications de production.

### Exemple simplifié d'utilisation

Voici un exemple minimal qui illustre comment créer et interroger des données avec EF Core :

```csharp
// Création du contexte
using (var context = new ApplicationDbContext())
{
    // Ajout d'un client
    var nouveauClient = new Client
    {
        Nom = "Dupont",
        Email = "dupont@exemple.fr",
        Ville = "Paris"
    };

    context.Clients.Add(nouveauClient);
    context.SaveChanges();

    // Recherche de clients
    var clientsParisiens = context.Clients
        .Where(c => c.Ville == "Paris")
        .ToList();

    Console.WriteLine($"Clients à Paris : {clientsParisiens.Count}");

    // Requête avec jointure
    var commandesRecentes = context.Commandes
        .Include(c => c.Client)  // Chargement de l'entité liée
        .Where(c => c.DateCommande > DateTime.Now.AddDays(-30))
        .OrderByDescending(c => c.Montant)
        .ToList();
}
```

---

Entity Framework Core est un outil puissant qui facilite grandement l'interaction avec les bases de données dans vos applications C#. En maîtrisant ses concepts fondamentaux, vous pourrez développer des applications plus rapidement tout en maintenant un code propre et maintenable.

Dans les sections suivantes, nous explorerons plus en détail les différentes approches de modélisation, la configuration des relations entre entités, et les techniques avancées de requêtage avec EF Core.

⏭️ 10.2. [Approches de modélisation](/10-entity-framework-core/10-2-approches-de-modelisation.md)
