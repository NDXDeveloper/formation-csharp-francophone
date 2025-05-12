# 11.1. Installation des connecteurs

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Pour connecter une application .NET à une base de données MySQL ou MariaDB, vous avez besoin d'un connecteur approprié. Il existe plusieurs options disponibles, chacune avec ses propres avantages et cas d'utilisation. Dans cette section, nous examinerons les trois principaux connecteurs MySQL pour .NET : MySQL Connector/NET (officiel), Pomelo.EntityFrameworkCore.MySql et MySqlConnector.

## 11.1.1. MySQL Connector/NET

MySQL Connector/NET est le connecteur officiel développé par Oracle pour permettre aux applications .NET de communiquer avec les serveurs MySQL.

### Présentation

MySQL Connector/NET (souvent appelé simplement "Connector/NET") est le pilote ADO.NET officiel pour MySQL. Il est développé et maintenu par Oracle, la société qui possède MySQL.

### Installation

Il existe plusieurs façons d'installer MySQL Connector/NET :

1. **Via NuGet (recommandé)** :

   ```bash
   # Pour une application standard
   dotnet add package MySql.Data

   # Pour utiliser Entity Framework Core
   dotnet add package MySql.EntityFrameworkCore
   ```

   Ou via la Console du Gestionnaire de packages dans Visual Studio :

   ```powershell
   Install-Package MySql.Data
   Install-Package MySql.EntityFrameworkCore
   ```

2. **Via l'installateur MSI** :
   - Téléchargez l'installateur depuis le [site officiel de MySQL](https://dev.mysql.com/downloads/connector/net/)
   - Exécutez le fichier `.msi` et suivez les instructions

### Configuration de base

Voici un exemple de code simple pour établir une connexion à une base de données MySQL :

```csharp
using MySql.Data.MySqlClient;

string connectionString = "Server=localhost;Database=ma_base;Uid=utilisateur;Pwd=mot_de_passe;";

using (MySqlConnection connection = new MySqlConnection(connectionString))
{
    connection.Open();

    // Utiliser la connexion...
    using (MySqlCommand command = new MySqlCommand("SELECT * FROM ma_table", connection))
    {
        using (MySqlDataReader reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                // Traiter les données...
                Console.WriteLine(reader[0].ToString());
            }
        }
    }
}
```

### Avantages et inconvénients

**Avantages :**
- Produit officiel d'Oracle/MySQL
- Support pour toutes les fonctionnalités de MySQL
- Documentation complète
- Support pour différentes versions de .NET (Framework et Core)

**Inconvénients :**
- Performance parfois inférieure aux alternatives
- Mises à jour moins fréquentes
- Problèmes connus avec certaines migrations Entity Framework Core
- Licence GPL avec exception FOSS (peut poser problème dans certains contextes commerciaux)

## 11.1.2. Pomelo.EntityFrameworkCore.MySql

Pomelo.EntityFrameworkCore.MySql est un provider populaire pour Entity Framework Core spécialement optimisé pour MySQL et MariaDB.

### Présentation

Pomelo.EntityFrameworkCore.MySql est un provider Entity Framework Core open-source développé par la communauté. Il est construit sur MySqlConnector et offre une excellente prise en charge d'Entity Framework Core pour MySQL, MariaDB et les bases de données compatibles.

### Installation

L'installation se fait via NuGet :

```bash
dotnet add package Pomelo.EntityFrameworkCore.MySql
```

Ou via la Console du Gestionnaire de packages :

```powershell
Install-Package Pomelo.EntityFrameworkCore.MySql
```

### Configuration

Voici comment configurer Pomelo.EntityFrameworkCore.MySql dans une application ASP.NET Core :

```csharp
// Dans Program.cs (ou Startup.cs pour les versions antérieures)
public void ConfigureServices(IServiceCollection services)
{
    // Chaîne de connexion
    var connectionString = "server=localhost;user=root;password=mot_de_passe;database=ma_base";

    // Version du serveur MySQL ou MariaDB
    var serverVersion = new MySqlServerVersion(new Version(8, 0, 29));

    // Pour MariaDB, utilisez MariaDbServerVersion à la place
    // var serverVersion = new MariaDbServerVersion(new Version(10, 6, 7));

    services.AddDbContext<MonDbContext>(options =>
        options.UseMySql(connectionString, serverVersion)
            // Options de débogage (à retirer en production)
            .LogTo(Console.WriteLine, LogLevel.Information)
            .EnableSensitiveDataLogging()
            .EnableDetailedErrors()
    );
}
```

Et voici un exemple de définition de DbContext :

```csharp
public class MonDbContext : DbContext
{
    public MonDbContext(DbContextOptions<MonDbContext> options)
        : base(options)
    {
    }

    public DbSet<Produit> Produits { get; set; }
    public DbSet<Categorie> Categories { get; set; }

    // Configuration supplémentaire si nécessaire
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configuration des entités...
    }
}
```

### Avantages et inconvénients

**Avantages :**
- Performances élevées
- Support complet des fonctionnalités d'Entity Framework Core
- Développement actif par la communauté
- Excellente prise en charge de MySQL et MariaDB
- Licence MIT (très permissive)
- Basé sur MySqlConnector qui offre une vraie prise en charge asynchrone

**Inconvénients :**
- Spécifique à Entity Framework Core (pas un pilote ADO.NET générique)
- Ne fait pas partie de l'écosystème officiel MySQL

## 11.1.3. MySqlConnector

MySqlConnector est un pilote ADO.NET alternatif pour MySQL et MariaDB, qui met l'accent sur la performance et les opérations asynchrones.

### Présentation

MySqlConnector est une implémentation alternative du protocole MySQL pour .NET, créée pour résoudre certains problèmes de performance et de conception de MySQL Connector/NET. Il offre une API compatible mais avec des performances améliorées et une meilleure prise en charge des opérations asynchrones.

### Installation

L'installation se fait via NuGet :

```bash
dotnet add package MySqlConnector
```

Ou via la Console du Gestionnaire de packages :

```powershell
Install-Package MySqlConnector
```

### Utilisation

Le code pour utiliser MySqlConnector est très similaire à celui de MySQL Connector/NET, mais avec un espace de noms différent :

```csharp
using MySqlConnector;

string connectionString = "Server=localhost;Database=ma_base;Uid=utilisateur;Pwd=mot_de_passe;";

using (MySqlConnection connection = new MySqlConnection(connectionString))
{
    await connection.OpenAsync();

    // Utilisation asynchrone (un des points forts de MySqlConnector)
    using (MySqlCommand command = new MySqlCommand("SELECT * FROM ma_table", connection))
    {
        using (MySqlDataReader reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                // Traiter les données...
                Console.WriteLine(reader[0].ToString());
            }
        }
    }
}
```

### Intégration avec Entity Framework Core

Si vous souhaitez utiliser MySqlConnector avec Entity Framework Core, vous pouvez le faire via Pomelo.EntityFrameworkCore.MySql, qui l'utilise en interne comme pilote de base.

### Avantages et inconvénients

**Avantages :**
- Meilleures performances que MySQL Connector/NET
- Vraie prise en charge asynchrone
- Meilleure stabilité et moins de bugs
- Licence MIT (très permissive)
- Compatible avec une large gamme de serveurs MySQL-compatibles
- Développement actif et mises à jour fréquentes

**Inconvénients :**
- Quelques différences de comportement avec MySQL Connector/NET
- Non officiel (bien que cela soit rarement un problème en pratique)

## Comparaison et choix du connecteur

Voici un tableau comparatif pour vous aider à choisir le connecteur adapté à votre projet :

| Fonctionnalité | MySQL Connector/NET | Pomelo.EntityFrameworkCore.MySql | MySqlConnector |
|----------------|---------------------|----------------------------------|---------------|
| **Type** | Officiel | Communautaire | Communautaire |
| **Licence** | GPL avec FOSS Exception | MIT | MIT |
| **Performance** | Bonne | Excellente | Excellente |
| **Async/await** | Partiel | Complet | Complet |
| **EF Core** | Oui (via MySQL.EntityFrameworkCore) | Oui (directement) | Oui (via Pomelo) |
| **Développement actif** | Oui (mises à jour trimestrielles) | Oui (fréquent) | Oui (fréquent) |
| **Compatibilité DB** | MySQL | MySQL, MariaDB, etc. | MySQL, MariaDB, etc. |

### Recommandations

1. **Pour les nouveaux projets utilisant Entity Framework Core** :
   - Utilisez Pomelo.EntityFrameworkCore.MySql pour ses performances et sa fiabilité

2. **Pour les projets nécessitant ADO.NET pur (sans Entity Framework)** :
   - Utilisez MySqlConnector pour ses performances et ses opérations asynchrones

3. **Pour les projets existants ou d'entreprise ayant une préférence pour les solutions officielles** :
   - Utilisez MySQL Connector/NET (MySql.Data)

4. **Pour des projets nécessitant des fonctionnalités spécifiques à MySQL** :
   - Vérifiez si ces fonctionnalités sont disponibles dans les connecteurs communautaires avant de choisir

La bonne nouvelle est que vous pouvez facilement migrer d'un connecteur à l'autre si nécessaire, car ils partagent des APIs similaires.

⏭️
