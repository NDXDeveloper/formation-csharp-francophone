# 16.1. Gestion des chaînes de connexion

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La gestion des chaînes de connexion est un aspect crucial du développement d'applications C# qui interagissent avec des bases de données. Une gestion incorrecte des chaînes de connexion peut compromettre la sécurité de votre application et des données qu'elle manipule.

## 16.1.1. Protection des informations sensibles

Les chaînes de connexion contiennent généralement des informations sensibles comme des identifiants, des mots de passe, et des adresses de serveurs. Il est essentiel de les protéger contre tout accès non autorisé.

### Problèmes liés aux chaînes de connexion en clair

Voici un exemple de chaîne de connexion typique pour SQL Server :

```csharp
string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
```

Les risques associés à cette approche sont nombreux :

- Si cette chaîne est codée en dur dans votre application, elle sera visible dans le code source
- Si la chaîne est stockée dans un fichier de configuration non sécurisé, elle peut être accessible à quiconque a accès au système de fichiers
- Si votre application est décompilée, la chaîne peut être extraite

### Bonnes pratiques de base

1. **Ne jamais coder en dur les chaînes de connexion dans le code source**

   ```csharp
   // À éviter absolument
   public class DataAccess
   {
       private readonly string _connectionString = "Server=production.db;User=admin;Password=SuperSecret123!";

       // Méthodes d'accès aux données...
   }
   ```

2. **Éviter de stocker les chaînes de connexion dans des fichiers de configuration en texte clair**

3. **Utiliser des mécanismes de protection appropriés en fonction du contexte**
   - Applications de bureau : DPAPI (Data Protection API)
   - Applications web : Stockage sécurisé via des services cloud ou des coffres-forts de secrets

### Utilisation de DPAPI pour protéger les chaînes

Pour les applications Windows, vous pouvez utiliser la classe `ProtectedData` pour chiffrer et déchiffrer des chaînes :

```csharp
using System;
using System.Text;
using System.Security.Cryptography;

public class SecureConnectionStringManager
{
    // Une entropie supplémentaire pour renforcer le chiffrement
    private static readonly byte[] _entropy = Encoding.Unicode.GetBytes("MonEntropieSecrete");

    public static string ProtectConnectionString(string connectionString)
    {
        if (string.IsNullOrEmpty(connectionString))
            throw new ArgumentNullException(nameof(connectionString));

        byte[] plainTextBytes = Encoding.Unicode.GetBytes(connectionString);
        byte[] encryptedBytes = ProtectedData.Protect(
            plainTextBytes,
            _entropy,
            DataProtectionScope.CurrentUser);

        return Convert.ToBase64String(encryptedBytes);
    }

    public static string UnprotectConnectionString(string encryptedConnectionString)
    {
        if (string.IsNullOrEmpty(encryptedConnectionString))
            throw new ArgumentNullException(nameof(encryptedConnectionString));

        byte[] encryptedBytes = Convert.FromBase64String(encryptedConnectionString);
        byte[] plainTextBytes = ProtectedData.Unprotect(
            encryptedBytes,
            _entropy,
            DataProtectionScope.CurrentUser);

        return Encoding.Unicode.GetString(plainTextBytes);
    }
}
```

Avec cette approche, vous pouvez stocker la version chiffrée dans un fichier de configuration et la déchiffrer uniquement lorsque vous en avez besoin.

## 16.1.2. Utilisation de secrets

.NET offre plusieurs mécanismes pour gérer les secrets d'application, dont les chaînes de connexion.

### Secret Manager pour le développement

Secret Manager est un outil intégré à .NET qui vous permet de stocker des secrets pendant le développement, sans les ajouter au contrôle de version.

#### Configuration de Secret Manager

1. Initialisez Secret Manager dans votre projet :

```bash
dotnet user-secrets init --project <chemin-du-projet>
```

2. Ajoutez votre chaîne de connexion :

```bash
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=myServer;Database=myDb;User Id=myUser;Password=myPass;" --project <chemin-du-projet>
```

3. Accédez aux secrets dans votre application :

```csharp
// Program.cs ou Startup.cs
var builder = WebApplication.CreateBuilder(args);

// Secret Manager est automatiquement intégré en environnement de développement
// quand vous appelez CreateBuilder

// Accéder à la chaîne de connexion
string connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

### Configuration selon l'environnement

Pour différents environnements (développement, test, production), vous pouvez utiliser des fichiers de configuration spécifiques :

```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=DevDb;Trusted_Connection=True;"
  }
}

// appsettings.Production.json
{
  "ConnectionStrings": {
    "DefaultConnection": "${PRODUCTION_CONNECTION_STRING}"
  }
}
```

Dans cet exemple, la variable d'environnement `PRODUCTION_CONNECTION_STRING` sera utilisée en production.

### Azure Key Vault pour les applications cloud

Pour les applications hébergées sur Azure, Azure Key Vault offre une solution robuste pour stocker et gérer les secrets :

1. Créez un Key Vault dans Azure
2. Ajoutez votre chaîne de connexion comme secret
3. Configurez votre application pour utiliser Key Vault :

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Ajouter la configuration d'Azure Key Vault
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVault:VaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential());

// Vous pouvez maintenant accéder aux secrets comme s'ils faisaient partie de la configuration
var app = builder.Build();

// Accès à la chaîne de connexion stockée dans Key Vault
string connectionString = builder.Configuration["ConnectionStrings:DefaultConnection"];
```

## 16.1.3. Rotation des informations d'identification

La rotation des informations d'identification est une pratique de sécurité qui consiste à changer périodiquement les mots de passe et autres secrets pour limiter l'impact d'une éventuelle compromission.

### Pourquoi faire une rotation des identifiants ?

1. **Limiter la fenêtre d'opportunité** en cas de fuite de données
2. **Se conformer aux politiques de sécurité** et aux réglementations
3. **Réduire les risques** associés aux accès non autorisés

### Stratégies de rotation

#### Rotation manuelle planifiée

Établissez un calendrier régulier pour changer vos identifiants de base de données et mettre à jour votre application :

```csharp
// Classe qui centralise l'accès à la chaîne de connexion
public class ConnectionStringProvider
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<ConnectionStringProvider> _logger;

    public ConnectionStringProvider(IConfiguration configuration, ILogger<ConnectionStringProvider> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }

    public string GetConnectionString()
    {
        var connectionString = _configuration.GetConnectionString("DefaultConnection");

        // Vérification de validité (vous pourriez ajouter une vérification de date d'expiration)
        if (string.IsNullOrEmpty(connectionString))
        {
            _logger.LogError("La chaîne de connexion est manquante ou invalide");
            throw new InvalidOperationException("Impossible de récupérer une chaîne de connexion valide");
        }

        return connectionString;
    }
}
```

#### Rotation automatique avec Azure Key Vault

Azure Key Vault permet de configurer des stratégies de rotation automatique :

1. Configurez la durée de vie des secrets dans Azure Key Vault
2. Utilisez Azure Functions ou Logic Apps pour automatiser la rotation
3. Implémentez un mécanisme de récupération des secrets les plus récents :

```csharp
public class KeyVaultConnectionStringProvider
{
    private readonly IKeyVaultClient _keyVaultClient;
    private readonly string _secretIdentifier;
    private string _cachedConnectionString;
    private DateTime _cacheExpiration;

    public KeyVaultConnectionStringProvider(IKeyVaultClient keyVaultClient, string vaultUri, string secretName)
    {
        _keyVaultClient = keyVaultClient;
        _secretIdentifier = $"{vaultUri}/secrets/{secretName}";
        _cacheExpiration = DateTime.MinValue; // Force refresh on first use
    }

    public async Task<string> GetConnectionStringAsync()
    {
        // Rafraîchir le cache si expiré (par exemple, toutes les heures)
        if (DateTime.UtcNow > _cacheExpiration)
        {
            var secret = await _keyVaultClient.GetSecretAsync(_secretIdentifier);
            _cachedConnectionString = secret.Value;
            _cacheExpiration = DateTime.UtcNow.AddHours(1);
        }

        return _cachedConnectionString;
    }
}
```

### Gestion des connexions pendant la rotation

Pour éviter les temps d'arrêt lors de la rotation des identifiants, vous pouvez implémenter une stratégie de double connexion :

```csharp
public class DualConnectionStringProvider
{
    private readonly IConfiguration _configuration;

    public DualConnectionStringProvider(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public IEnumerable<string> GetAllConnectionStrings()
    {
        // Essaie d'abord la connexion principale
        string primaryConnection = _configuration.GetConnectionString("PrimaryConnection");
        if (!string.IsNullOrEmpty(primaryConnection))
            yield return primaryConnection;

        // Si disponible, fournit également la connexion secondaire
        string secondaryConnection = _configuration.GetConnectionString("SecondaryConnection");
        if (!string.IsNullOrEmpty(secondaryConnection))
            yield return secondaryConnection;
    }

    public string GetFallbackConnectionString(Exception ex)
    {
        // Si l'erreur indique que le mot de passe a expiré ou est incorrect,
        // on bascule sur la connexion secondaire
        if (ex.Message.Contains("password expired") || ex.Message.Contains("login failed"))
        {
            return _configuration.GetConnectionString("SecondaryConnection");
        }

        // Dans les autres cas, on relance l'exception
        throw ex;
    }
}
```

## 16.1.4. Environment variables et User Secrets

Les variables d'environnement et User Secrets sont deux approches complémentaires pour la gestion des chaînes de connexion.

### Variables d'environnement

Les variables d'environnement sont particulièrement utiles pour les déploiements et la configuration des applications en production.

#### Définition des variables d'environnement

Sur Windows (PowerShell) :
```powershell
$env:ConnectionStrings__DefaultConnection = "Server=prod;Database=myDb;User=prodUser;Password=prodPass"
```

Sur Linux/macOS :
```bash
export ConnectionStrings__DefaultConnection="Server=prod;Database=myDb;User=prodUser;Password=prodPass"
```

Dans les conteneurs Docker :
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/out .
ENV ConnectionStrings__DefaultConnection="Server=container-db;Database=myDb;User=dbuser;Password=dbpass"
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

#### Accès aux variables d'environnement

Dans .NET, l'accès aux variables d'environnement est simplifié grâce au système de configuration :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuration par défaut qui inclut les variables d'environnement
// En utilisant le préfixe "ASPNETCORE_" ou sans préfixe

// Accès direct à l'environnement
string connString = Environment.GetEnvironmentVariable("ConnectionStrings__DefaultConnection");

// Ou via IConfiguration (méthode recommandée)
string connString2 = builder.Configuration.GetConnectionString("DefaultConnection");
```

### User Secrets vs Variables d'environnement

| Caractéristique | User Secrets | Variables d'environnement |
|-----------------|--------------|---------------------------|
| **Usage principal** | Développement | Production |
| **Stockage** | Fichier local hors du projet | Système d'exploitation ou plateforme d'hébergement |
| **Partage** | Non partagé (par développeur) | Partagé au niveau du système |
| **Persistance** | Permanent | Peut être temporaire (selon configuration) |
| **Accès** | Via IConfiguration | Via IConfiguration ou Environment.GetEnvironmentVariable |

### Exemple complet avec priorité de configuration

Voici comment configurer une application pour qu'elle utilise différentes sources de configuration avec une priorité claire :

```csharp
// Program.cs pour une application ASP.NET Core
var builder = WebApplication.CreateBuilder(args);

// La configuration par défaut inclut déjà :
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. User Secrets (en développement)
// 4. Variables d'environnement
// 5. Arguments de ligne de commande

// Si vous voulez changer l'ordre ou ajouter des sources :
var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false)
    .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)
    .AddUserSecrets<Program>(optional: true) // Seulement en développement
    .AddEnvironmentVariables() // Les variables d'environnement peuvent remplacer les paramètres précédents
    .AddCommandLine(args) // Les arguments de ligne de commande ont la priorité la plus élevée
    .Build();

// Remplacer la configuration par défaut
builder.Configuration.Sources.Clear();
foreach (var source in configuration.Providers)
{
    builder.Configuration.AddConfiguration(configuration);
}

// Exemple d'utilisation de la chaîne de connexion
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Le reste de la configuration de l'application...
app.Run();
```

### Bonnes pratiques et recommandations

1. **En développement** :
   - Utilisez User Secrets pour stocker vos chaînes de connexion locales
   - Configurez différentes chaînes pour différents environnements de développement

2. **En test et intégration continue** :
   - Utilisez des variables d'environnement définies au niveau du pipeline CI/CD
   - Créez des bases de données temporaires pour les tests

3. **En production** :
   - Utilisez un service de gestion de secrets (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault)
   - Implémentez la rotation des identifiants
   - Limitez l'accès aux variables d'environnement sensibles

4. **Dans tous les environnements** :
   - Ne stockez jamais les chaînes de connexion en clair dans le code source
   - Validez les chaînes de connexion avant utilisation
   - Utilisez le principe du moindre privilège pour les comptes de base de données

En suivant ces bonnes pratiques, vous améliorerez considérablement la sécurité de vos applications C# qui interagissent avec des bases de données.

⏭️
