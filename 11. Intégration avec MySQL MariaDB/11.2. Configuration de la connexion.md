# 11.2. Configuration de la connexion

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La configuration correcte de la connexion à votre base de données MySQL ou MariaDB est une étape essentielle pour garantir les performances, la sécurité et la fiabilité de votre application. Dans cette section, nous allons explorer les différentes façons de configurer les connexions dans Entity Framework Core.

## 11.2.1. Chaînes de connexion

Une chaîne de connexion est une série de paires clé-valeur qui définissent comment se connecter à une base de données. Chaque paramètre est séparé par un point-virgule, et le nom du paramètre est séparé de sa valeur par un signe égal.

### Format de base

Voici le format de base d'une chaîne de connexion MySQL :

```
Server=serveur;Port=port;Database=base_de_donnees;Uid=utilisateur;Pwd=mot_de_passe;
```

Exemple concret :

```
Server=localhost;Port=3306;Database=ma_boutique;Uid=root;Pwd=mot2passe;
```

### Paramètres essentiels

Les paramètres les plus couramment utilisés sont :

| Paramètre | Description | Valeur par défaut |
|-----------|-------------|------------------|
| Server (ou Host) | Adresse du serveur MySQL | localhost |
| Port | Port du serveur MySQL | 3306 |
| Database | Nom de la base de données | (aucune) |
| Uid (ou User Id, Username) | Nom d'utilisateur | (aucun) |
| Pwd (ou Password) | Mot de passe | (aucun) |

### Méthodes pour définir la chaîne de connexion

#### 1. Directement dans le code

```csharp
var connectionString = "Server=localhost;Database=ma_boutique;Uid=root;Pwd=mot2passe;";

var optionsBuilder = new DbContextOptionsBuilder<MaBoutiqueContext>();
optionsBuilder.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));

using (var context = new MaBoutiqueContext(optionsBuilder.Options))
{
    // Utiliser le contexte...
}
```

#### 2. Dans le constructeur du DbContext

```csharp
public class MaBoutiqueContext : DbContext
{
    public DbSet<Produit> Produits { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        var connectionString = "Server=localhost;Database=ma_boutique;Uid=root;Pwd=mot2passe;";
        optionsBuilder.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));
    }
}
```

#### 3. Dans appsettings.json (recommandé pour ASP.NET Core)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=ma_boutique;Uid=root;Pwd=mot2passe;"
  }
}
```

Et dans Program.cs (ou Startup.cs pour les versions antérieures) :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var connectionString = Configuration.GetConnectionString("DefaultConnection");
    services.AddDbContext<MaBoutiqueContext>(options =>
        options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));
}
```

## 11.2.2. Options spécifiques à MySQL/MariaDB

MySQL et MariaDB offrent des options spécifiques qui peuvent être ajoutées à votre chaîne de connexion pour optimiser son comportement.

### Options de connexion courantes

| Option | Description | Exemple |
|--------|-------------|---------|
| Connect Timeout | Temps d'attente (en secondes) avant d'abandonner une tentative de connexion | `Connect Timeout=30;` |
| CharSet (ou Character Set) | Jeu de caractères à utiliser | `CharSet=utf8mb4;` |
| SslMode | Mode de connexion SSL | `SslMode=Required;` |
| AllowPublicKeyRetrieval | Autorise la récupération de la clé publique depuis le serveur | `AllowPublicKeyRetrieval=true;` |
| ConvertZeroDateTime | Convertit les valeurs zéro de date MySQL en MinValue .NET | `ConvertZeroDateTime=true;` |

### Exemple avec options avancées

```
Server=localhost;Database=ma_boutique;Uid=root;Pwd=mot2passe;CharSet=utf8mb4;SslMode=Required;Connect Timeout=30;ConvertZeroDateTime=true;
```

### Options de pool de connexions

Le pool de connexions est une technique qui maintient des connexions ouvertes pour les réutiliser, améliorant ainsi les performances.

| Option | Description | Valeur par défaut |
|--------|-------------|------------------|
| Pooling | Active ou désactive le pooling | true |
| ConnectionReset | Réinitialise la connexion lors de sa sortie du pool | true |
| MinimumPoolSize | Nombre minimal de connexions à maintenir dans le pool | 0 |
| MaximumPoolSize | Nombre maximal de connexions dans le pool | 100 |
| ConnectionLifeTime | Durée de vie maximale (en secondes) d'une connexion | 0 (illimité) |

Exemple avec options de pooling :

```
Server=localhost;Database=ma_boutique;Uid=root;Pwd=mot2passe;Pooling=true;MinimumPoolSize=5;MaximumPoolSize=50;ConnectionLifeTime=300;
```

### Configuration spécifique à MariaDB

Si vous utilisez MariaDB, il est recommandé de spécifier explicitement le type de serveur :

```csharp
// Pour MariaDB
var serverVersion = new MariaDbServerVersion(new Version(10, 6, 7));

// Puis dans la configuration
services.AddDbContext<MaBoutiqueContext>(options =>
    options.UseMySql(connectionString, serverVersion));
```

## 11.2.3. Configuration pour différents environnements

Dans une application réelle, vous devez souvent gérer plusieurs environnements (développement, test, production) avec des configurations de base de données différentes.

### Utilisation des environnements ASP.NET Core

ASP.NET Core offre un système intégré pour gérer les environnements via la variable d'environnement `ASPNETCORE_ENVIRONMENT`.

#### 1. Fichiers appsettings spécifiques aux environnements

Créez différents fichiers de configuration pour chaque environnement :

- appsettings.json (paramètres de base)
- appsettings.Development.json (développement)
- appsettings.Staging.json (préproduction)
- appsettings.Production.json (production)

Exemple pour le développement (appsettings.Development.json) :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=ma_boutique_dev;Uid=dev_user;Pwd=dev_password;"
  }
}
```

Exemple pour la production (appsettings.Production.json) :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=ma_boutique_prod;Uid=prod_user;Pwd=prod_password;SslMode=Required;"
  }
}
```

ASP.NET Core chargera automatiquement le fichier correspondant à l'environnement actuel.

#### 2. Utilisation des variables d'environnement

Vous pouvez remplacer les paramètres de configuration par des variables d'environnement :

```bash
# Sous Linux/macOS
export ConnectionStrings__DefaultConnection="Server=prod-server;Database=ma_boutique_prod;Uid=prod_user;Pwd=prod_password;"

# Sous Windows (PowerShell)
$env:ConnectionStrings__DefaultConnection="Server=prod-server;Database=ma_boutique_prod;Uid=prod_user;Pwd=prod_password;"
```

### Gestion sécurisée des secrets

Pour des raisons de sécurité, évitez de stocker des mots de passe et autres informations sensibles dans le code source ou les fichiers de configuration versionnés.

#### 1. User Secrets (Développement)

Pour le développement, utilisez l'outil User Secrets :

```bash
# Initialiser le stockage des secrets pour votre projet
dotnet user-secrets init --project MonProjet.csproj

# Ajouter une chaîne de connexion aux secrets
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=ma_boutique;Uid=root;Pwd=mot2passe;" --project MonProjet.csproj
```

#### 2. Services de stockage sécurisé (Production)

Pour la production, utilisez des services comme :
- Azure Key Vault
- AWS Secrets Manager
- HashiCorp Vault

Exemple avec Azure Key Vault :

```csharp
// Ajoutez le package NuGet Microsoft.Extensions.Configuration.AzureKeyVault
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            var builtConfig = config.Build();
            config.AddAzureKeyVault(
                builtConfig["KeyVault:Vault"],
                builtConfig["KeyVault:ClientId"],
                builtConfig["KeyVault:ClientSecret"]);
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

### Configuration conditionnelle basée sur l'environnement

Vous pouvez également adapter votre code en fonction de l'environnement :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var connectionString = Configuration.GetConnectionString("DefaultConnection");

    if (_env.IsDevelopment())
    {
        services.AddDbContext<MaBoutiqueContext>(options =>
            options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString))
                .EnableSensitiveDataLogging() // Affiche les valeurs des paramètres dans les logs
                .EnableDetailedErrors()); // Affiche des messages d'erreur détaillés
    }
    else
    {
        services.AddDbContext<MaBoutiqueContext>(options =>
            options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));
    }
}
```

## Bonnes pratiques pour la configuration de connexion

1. **Ne stockez jamais les mots de passe en clair dans le code source**
2. **Utilisez des utilisateurs distincts pour chaque environnement avec des permissions minimales**
3. **Activez SSL (SslMode=Required) en production**
4. **Configurez correctement le pool de connexions pour votre charge de travail**
5. **Utilisez un jeu de caractères moderne (utf8mb4) pour prendre en charge tous les caractères Unicode**
6. **Ajoutez un timeout de connexion raisonnable pour éviter les blocages prolongés**
7. **Activez la journalisation détaillée uniquement en développement**

En suivant ces conseils, vous aurez une configuration de connexion MySQL/MariaDB sécurisée, performante et adaptée à tous vos environnements.

⏭️
