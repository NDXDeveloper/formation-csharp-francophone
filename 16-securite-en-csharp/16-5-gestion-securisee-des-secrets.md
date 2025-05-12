# 16.5. Gestion sécurisée des secrets

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les secrets sont des informations sensibles comme les chaînes de connexion, les API keys, les certificats ou les mots de passe qui doivent être protégés contre tout accès non autorisé. Une mauvaise gestion des secrets peut compromettre la sécurité de votre application et des données qu'elle manipule.

Dans cette section, nous allons explorer différentes techniques pour gérer ces secrets de manière sécurisée dans vos applications C#.

## 16.5.1. Azure Key Vault

Azure Key Vault est un service cloud qui permet de stocker et de gérer de manière sécurisée des clés cryptographiques, des certificats et d'autres secrets. C'est une solution robuste pour les applications hébergées sur Azure ou qui interagissent avec Azure.

### Pourquoi utiliser Azure Key Vault ?

- **Sécurité renforcée** : Les secrets sont chiffrés au repos et en transit
- **Contrôle d'accès** : Gestion fine des permissions via Azure Active Directory
- **Journalisation** : Suivi de tous les accès aux secrets
- **Gestion centralisée** : Un seul endroit pour gérer tous vos secrets
- **Haute disponibilité** : Service géré par Microsoft avec SLA de disponibilité
- **Rotation des secrets** : Possibilité d'automatiser la rotation des secrets

### Configuration d'Azure Key Vault

#### 1. Création d'un Key Vault dans Azure

Avant d'utiliser Azure Key Vault dans votre application, vous devez en créer un dans le portail Azure ou via l'Azure CLI :

```bash
# Connexion à Azure
az login

# Création d'un groupe de ressources (si nécessaire)
az group create --name MyResourceGroup --location westeurope

# Création d'un Key Vault
az keyvault create --name MyKeyVault --resource-group MyResourceGroup --location westeurope
```

#### 2. Ajout de secrets dans Azure Key Vault

```bash
# Ajout d'un secret
az keyvault secret set --vault-name MyKeyVault --name "ConnectionString" --value "Server=myserver;Database=mydb;User=myuser;Password=mypassword;"

# Ajout d'un autre secret
az keyvault secret set --vault-name MyKeyVault --name "ApiKey" --value "1234567890abcdef"
```

### Intégration avec une application ASP.NET Core

#### 1. Installation des packages NuGet nécessaires

```bash
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
dotnet add package Microsoft.Extensions.Configuration.AzureKeyVault
```

#### 2. Configuration dans Program.cs (ASP.NET Core 6+)

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

var builder = WebApplication.CreateBuilder(args);

// Configuration pour accéder à Azure Key Vault
if (!builder.Environment.IsDevelopment())
{
    var keyVaultUrl = builder.Configuration["KeyVault:Url"];

    // Option 1 : DefaultAzureCredential (recommandée)
    builder.Configuration.AddAzureKeyVault(
        new Uri(keyVaultUrl),
        new DefaultAzureCredential());

    // Option 2 : Utilisation explicite d'un client SecretClient
    // (utile si vous avez besoin de plus de contrôle)
    var secretClient = new SecretClient(
        new Uri(keyVaultUrl),
        new DefaultAzureCredential());

    builder.Services.AddSingleton(secretClient);
}

// Autres configurations...
var app = builder.Build();

// Middleware...
app.Run();
```

### Utilisation des secrets dans le code

#### Méthode 1 : Via IConfiguration

```csharp
public class MyService
{
    private readonly string _connectionString;
    private readonly string _apiKey;

    public MyService(IConfiguration configuration)
    {
        // Les secrets sont accessibles comme n'importe quelle autre configuration
        _connectionString = configuration["ConnectionString"];
        _apiKey = configuration["ApiKey"];
    }

    // Reste du service...
}
```

#### Méthode 2 : Via SecretClient

```csharp
public class MySecureService
{
    private readonly SecretClient _secretClient;

    public MySecureService(SecretClient secretClient)
    {
        _secretClient = secretClient;
    }

    public async Task<string> GetSecretAsync(string secretName)
    {
        try
        {
            KeyVaultSecret secret = await _secretClient.GetSecretAsync(secretName);
            return secret.Value;
        }
        catch (Exception ex)
        {
            // Gérer l'erreur (par exemple, secret inexistant, problème d'autorisation)
            Console.WriteLine($"Erreur lors de la récupération du secret: {ex.Message}");
            throw;
        }
    }

    public async Task DoSomethingWithSecretAsync()
    {
        string connectionString = await GetSecretAsync("ConnectionString");
        // Utiliser la chaîne de connexion de manière sécurisée
    }
}
```

### Bonnes pratiques pour Azure Key Vault

1. **Utiliser des identités gérées** : Évitez de stocker des identifiants d'application dans votre code

   ```csharp
   // Configuration avec une identité gérée
   builder.Configuration.AddAzureKeyVault(
       new Uri(keyVaultUrl),
       new ManagedIdentityCredential());
   ```

2. **Mettre en cache les secrets** pour réduire les appels à Key Vault

   ```csharp
   public class CachedSecretProvider
   {
       private readonly SecretClient _secretClient;
       private readonly IMemoryCache _cache;
       private readonly TimeSpan _cacheDuration = TimeSpan.FromMinutes(10);

       public CachedSecretProvider(SecretClient secretClient, IMemoryCache cache)
       {
           _secretClient = secretClient;
           _cache = cache;
       }

       public async Task<string> GetSecretAsync(string secretName)
       {
           return await _cache.GetOrCreateAsync(
               $"Secret_{secretName}",
               async entry =>
               {
                   entry.AbsoluteExpirationRelativeToNow = _cacheDuration;
                   KeyVaultSecret secret = await _secretClient.GetSecretAsync(secretName);
                   return secret.Value;
               });
       }
   }
   ```

3. **Surveiller les accès** à vos secrets via les journaux d'audit d'Azure

## 16.5.2. Secret Manager

Pour le développement local, stocker des secrets dans le code ou les fichiers de configuration présente un risque, surtout si vous utilisez un système de contrôle de version comme Git. Secret Manager est un outil inclus dans l'écosystème .NET qui permet de stocker des secrets localement, hors du contrôle de version.

### Configuration de Secret Manager

#### 1. Initialisation de Secret Manager dans votre projet

```bash
# À exécuter dans le répertoire du projet
dotnet user-secrets init
```

Cette commande ajoute un `UserSecretsId` à votre fichier de projet (.csproj) :

```xml
<PropertyGroup>
  <TargetFramework>net6.0</TargetFramework>
  <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
</PropertyGroup>
```

#### 2. Ajout de secrets

```bash
# Ajout d'un secret simple
dotnet user-secrets set "ConnectionString" "Server=localhost;Database=MyDevDb;User=devuser;Password=devpassword;"

# Ajout d'un secret dans une section
dotnet user-secrets set "Authentication:ApiKey" "mydevapikey123"

# Ajout de plusieurs secrets à partir d'un fichier JSON
dotnet user-secrets set --json "{ \"Secrets\": { \"Key1\": \"Value1\", \"Key2\": \"Value2\" } }"
```

#### 3. Listing et suppression des secrets

```bash
# Lister tous les secrets
dotnet user-secrets list

# Supprimer un secret
dotnet user-secrets remove "ConnectionString"

# Supprimer tous les secrets
dotnet user-secrets clear
```

### Accès aux secrets dans le code

En ASP.NET Core, les secrets sont automatiquement chargés dans la configuration en environnement de développement :

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// En développement, builder.Configuration contient déjà les secrets chargés
// via Secret Manager si vous avez configuré UserSecretsId

var app = builder.Build();
```

```csharp
// Dans un service ou contrôleur
public class MyService
{
    private readonly string _connectionString;

    public MyService(IConfiguration configuration)
    {
        _connectionString = configuration["ConnectionString"];
    }
}
```

### Utilisation en dehors d'ASP.NET Core

Pour les applications console ou autres types d'applications .NET, vous devez configurer explicitement Secret Manager :

```csharp
using Microsoft.Extensions.Configuration;

class Program
{
    static void Main(string[] args)
    {
        IConfiguration configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json", optional: true)
            .AddUserSecrets<Program>() // Charge les secrets pour l'assembly contenant Program
            .AddEnvironmentVariables()
            .Build();

        string connectionString = configuration["ConnectionString"];
        Console.WriteLine($"Connexion à : {connectionString}");
    }
}
```

### Limites et considérations

- **Uniquement pour le développement** : Ne pas utiliser en production
- **Stockage local** : Les secrets sont stockés en texte clair dans un fichier JSON (sous `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json` sur Windows)
- **Par développeur** : Chaque développeur doit configurer ses propres secrets

## 16.5.3. DPAPI (Data Protection API)

DPAPI est une API Windows qui permet de chiffrer et déchiffrer des données de manière sécurisée, en utilisant les informations d'identification de l'utilisateur ou de la machine.

### Utilisation de base de DPAPI

#### 1. Protection de données avec DPAPI

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class DpapiHelper
{
    // Protéger une chaîne avec DPAPI (niveau utilisateur)
    public static string ProtectData(string data)
    {
        if (string.IsNullOrEmpty(data))
            return string.Empty;

        byte[] dataBytes = Encoding.UTF8.GetBytes(data);
        byte[] protectedBytes = ProtectedData.Protect(
            dataBytes,
            optionalEntropy: null, // Sel supplémentaire optionnel
            scope: DataProtectionScope.CurrentUser); // Ou DataProtectionScope.LocalMachine

        return Convert.ToBase64String(protectedBytes);
    }

    // Déprotéger une chaîne avec DPAPI
    public static string UnprotectData(string protectedData)
    {
        if (string.IsNullOrEmpty(protectedData))
            return string.Empty;

        byte[] protectedBytes = Convert.FromBase64String(protectedData);
        byte[] originalBytes = ProtectedData.Unprotect(
            protectedBytes,
            optionalEntropy: null, // Doit être identique à celui utilisé pour protéger
            scope: DataProtectionScope.CurrentUser); // Doit être identique à celui utilisé pour protéger

        return Encoding.UTF8.GetString(originalBytes);
    }
}
```

#### 2. Exemple d'utilisation pour protéger une chaîne de connexion

```csharp
// Application console ou desktop
class Program
{
    private const string ConfigFileName = "app.config.protected";

    static void Main(string[] args)
    {
        if (args.Length > 0 && args[0] == "--init")
        {
            InitializeConfig();
        }
        else
        {
            RunWithProtectedConfig();
        }
    }

    static void InitializeConfig()
    {
        Console.WriteLine("Initialisation de la configuration sécurisée...");

        Console.Write("Entrez la chaîne de connexion : ");
        string connectionString = Console.ReadLine();

        // Protéger la chaîne de connexion
        string protectedConnectionString = DpapiHelper.ProtectData(connectionString);

        // Enregistrer dans un fichier
        File.WriteAllText(ConfigFileName, protectedConnectionString);

        Console.WriteLine("Configuration sécurisée enregistrée.");
    }

    static void RunWithProtectedConfig()
    {
        if (!File.Exists(ConfigFileName))
        {
            Console.WriteLine("Le fichier de configuration n'existe pas. Exécutez avec --init pour le créer.");
            return;
        }

        // Lire la chaîne protégée
        string protectedConnectionString = File.ReadAllText(ConfigFileName);

        // Déprotéger
        string connectionString = DpapiHelper.UnprotectData(protectedConnectionString);

        // Utiliser la chaîne de connexion
        Console.WriteLine("Connexion établie avec la base de données.");
        // ... code pour utiliser la connexion ...
    }
}
```

### Utilisation de DPAPI dans ASP.NET Core

ASP.NET Core inclut un système de protection des données basé sur DPAPI, mais avec des fonctionnalités supplémentaires.

#### 1. Configuration du service de protection des données

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Configuration de la protection des données
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\keys"))
    .SetDefaultKeyLifetime(TimeSpan.FromDays(14))
    .ProtectKeysWithDpapi();

var app = builder.Build();
```

#### 2. Utilisation dans un service

```csharp
public class SecureDataService
{
    private readonly IDataProtector _protector;

    public SecureDataService(IDataProtectionProvider provider)
    {
        // Créer un protecteur avec un objectif spécifique
        _protector = provider.CreateProtector("MyApp.SecureData");
    }

    public string ProtectData(string data)
    {
        return _protector.Protect(data);
    }

    public string UnprotectData(string protectedData)
    {
        return _protector.Unprotect(protectedData);
    }

    // Avec expiration
    public string ProtectWithExpiration(string data, TimeSpan lifetime)
    {
        var timeLimitedProtector = _protector.ToTimeLimitedDataProtector();
        return timeLimitedProtector.Protect(data, lifetime);
    }

    public string UnprotectWithExpiration(string protectedData)
    {
        var timeLimitedProtector = _protector.ToTimeLimitedDataProtector();
        return timeLimitedProtector.Unprotect(protectedData);
    }
}
```

### Avantages et limites de DPAPI

**Avantages** :
- Intégré au système d'exploitation Windows
- Simple à utiliser
- Pas besoin de gérer les clés de chiffrement

**Limites** :
- Spécifique à Windows
- Les données protégées avec `DataProtectionScope.CurrentUser` ne peuvent être déchiffrées que par l'utilisateur qui les a chiffrées
- Les données protégées avec `DataProtectionScope.LocalMachine` peuvent être déchiffrées par n'importe quel processus sur la machine

## 16.5.4. Stockage sécurisé en environnement conteneurisé

Les environnements conteneurisés comme Docker présentent des défis spécifiques pour la gestion des secrets, car les conteneurs sont conçus pour être éphémères et portables.

### Options pour les secrets dans Docker

#### 1. Variables d'environnement

La méthode la plus simple, mais à utiliser avec précaution.

**Dockerfile** (non recommandé pour les secrets) :
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=build /app/out .
# Ne PAS faire ceci pour des secrets réels !
ENV ConnectionString="Server=myserver;Database=mydb;User=myuser;Password=mypassword;"
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

**docker-compose.yml** (plus sûr car hors de l'image) :
```yaml
version: '3.8'
services:
  myapp:
    image: myapp:latest
    environment:
      - ConnectionString=Server=myserver;Database=mydb;User=myuser;Password=mypassword;
```

**Utilisation dans le code** :
```csharp
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);

// Les variables d'environnement sont automatiquement chargées
// Vous pouvez aussi le faire explicitement :
builder.Configuration.AddEnvironmentVariables();

var app = builder.Build();
```

#### 2. Docker Secrets

Pour les applications basées sur Docker Swarm, les Docker Secrets offrent une gestion des secrets plus sécurisée.

**Création d'un secret** :
```bash
printf "myStrongPassword123" | docker secret create db_password -
```

**docker-compose.yml avec secrets** :
```yaml
version: '3.8'
services:
  myapp:
    image: myapp:latest
    secrets:
      - db_password
    environment:
      - DB_HOST=myserver
      - DB_NAME=mydb
      - DB_USER=myuser

secrets:
  db_password:
    external: true
```

**Accès aux secrets dans le conteneur** :
Les secrets sont montés dans `/run/secrets/<secret_name>` dans le conteneur.

```csharp
// Dans le code C#
public string GetDatabasePassword()
{
    string secretPath = "/run/secrets/db_password";

    if (File.Exists(secretPath))
    {
        return File.ReadAllText(secretPath).Trim();
    }

    // Fallback pour le développement local
    return Environment.GetEnvironmentVariable("DB_PASSWORD");
}
```

#### 3. Kubernetes Secrets

Si vous utilisez Kubernetes, vous pouvez utiliser ses mécanismes de gestion des secrets.

**Création d'un secret Kubernetes** :
```bash
kubectl create secret generic db-credentials \
  --from-literal=username=myuser \
  --from-literal=password=myStrongPassword123
```

**Accès via des variables d'environnement dans le pod** :
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:latest
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

**Accès via des volumes** :
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:latest
    volumeMounts:
    - name: secrets
      mountPath: "/etc/secrets"
      readOnly: true
  volumes:
  - name: secrets
    secret:
      secretName: db-credentials
```

### Intégration avec des gestionnaires de secrets externes

Pour les environnements conteneurisés en production, il est recommandé d'utiliser un gestionnaire de secrets externe comme HashiCorp Vault ou Azure Key Vault.

#### Exemple avec HashiCorp Vault dans ASP.NET Core

```csharp
// Installation du package NuGet
// dotnet add package VaultSharp

// Program.cs
using VaultSharp;
using VaultSharp.V1.AuthMethods.Token;

var builder = WebApplication.CreateBuilder(args);

// Configuration de HashiCorp Vault
var vaultClient = new VaultClient(new VaultClientSettings(
    builder.Configuration["Vault:Address"],
    new TokenAuthMethodInfo(builder.Configuration["Vault:Token"])));

// Charger les secrets au démarrage
var secrets = await vaultClient.V1.Secrets.KeyValue.V2.ReadSecretAsync(
    path: "myapp",
    mountPoint: "secret");

// Ajouter les secrets à la configuration
foreach (var secret in secrets.Data.Data)
{
    builder.Configuration[secret.Key] = secret.Value.ToString();
}

var app = builder.Build();
```

### Bonnes pratiques pour les environnements conteneurisés

1. **Jamais dans l'image** : Ne stockez jamais de secrets dans l'image Docker
2. **Privilégier l'injection** : Injectez les secrets au moment de l'exécution
3. **Limiter l'accès** : Appliquez le principe du moindre privilège
4. **Chiffrer les secrets** : Utilisez des gestionnaires de secrets qui chiffrent au repos
5. **Rotation régulière** : Mettez en place une rotation des secrets
6. **Audit des accès** : Journalisez et surveillez les accès aux secrets

## 16.5.5. Rotation automatique des secrets

La rotation des secrets consiste à changer régulièrement les secrets (mots de passe, clés API, etc.) pour limiter l'impact d'une potentielle fuite.

### Pourquoi implémenter la rotation des secrets ?

- **Limiter l'exposition** : Réduire la fenêtre pendant laquelle un secret compromis peut être utilisé
- **Conformité** : Respecter les politiques de sécurité et les réglementations
- **Meilleure gestion** : Faciliter la revue et l'audit des accès

### Stratégies de rotation

#### 1. Rotation manuelle planifiée

La méthode la plus simple, mais qui nécessite une intervention humaine.

```csharp
// Service de notification pour la rotation des secrets
public class SecretRotationNotifier
{
    private readonly ILogger<SecretRotationNotifier> _logger;
    private readonly IEmailSender _emailSender;

    public SecretRotationNotifier(
        ILogger<SecretRotationNotifier> logger,
        IEmailSender emailSender)
    {
        _logger = logger;
        _emailSender = emailSender;
    }

    public async Task CheckSecretsExpirationAsync(List<SecretInfo> secrets)
    {
        var expiringSecrets = secrets
            .Where(s => s.ExpiryDate <= DateTime.UtcNow.AddDays(7))
            .ToList();

        foreach (var secret in expiringSecrets)
        {
            _logger.LogWarning(
                "Le secret {SecretName} expire le {ExpiryDate}",
                secret.Name,
                secret.ExpiryDate);

            await _emailSender.SendEmailAsync(
                "admin@example.com",
                $"Secret {secret.Name} à renouveler",
                $"Le secret {secret.Name} expire le {secret.ExpiryDate}. " +
                $"Veuillez le renouveler dès que possible.");
        }
    }
}
```

#### 2. Rotation automatique avec Azure Key Vault

Azure Key Vault permet de configurer une rotation automatique des secrets.

```csharp
// Installation du package NuGet
// dotnet add package Azure.Security.KeyVault.Secrets

// Service de gestion des secrets avec rotation automatique
public class KeyVaultSecretManager
{
    private readonly SecretClient _secretClient;
    private readonly ILogger<KeyVaultSecretManager> _logger;

    public KeyVaultSecretManager(
        SecretClient secretClient,
        ILogger<KeyVaultSecretManager> logger)
    {
        _secretClient = secretClient;
        _logger = logger;
    }

    public async Task RotateSecretAsync(string secretName, Func<string> generateNewSecret)
    {
        try
        {
            // Récupérer le secret actuel
            KeyVaultSecret currentSecret = await _secretClient.GetSecretAsync(secretName);

            // Générer une nouvelle valeur
            string newSecretValue = generateNewSecret();

            // Créer une nouvelle version du secret
            await _secretClient.SetSecretAsync(new KeyVaultSecret(secretName, newSecretValue));

            _logger.LogInformation("Secret {SecretName} rotated successfully", secretName);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to rotate secret {SecretName}", secretName);
            throw;
        }
    }
}
```

#### 3. Implémentation d'un système de rotation personnalisé

Pour des scénarios plus complexes, vous pouvez implémenter votre propre système de rotation.

```csharp
// Modèle de donnée pour un secret avec versions
public class SecretWithHistory
{
    public string Id { get; set; }
    public string Name { get; set; }
    public List<SecretVersion> Versions { get; set; } = new List<SecretVersion>();
    public TimeSpan RotationInterval { get; set; } = TimeSpan.FromDays(90);
    public DateTime LastRotated { get; set; }
    public bool IsRotationOverdue => DateTime.UtcNow - LastRotated > RotationInterval;
}

public class SecretVersion
{
    public string Id { get; set; } = Guid.NewGuid().ToString();
    public string Value { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public bool IsActive { get; set; }
    public DateTime? DeactivatedAt { get; set; }
}

// Service de rotation des secrets
public class SecretRotationService
{
    private readonly ISecretRepository _secretRepository;
    private readonly ILogger<SecretRotationService> _logger;
    private readonly Dictionary<string, Func<string>> _secretGenerators;

    public SecretRotationService(
        ISecretRepository secretRepository,
        ILogger<SecretRotationService> logger)
    {
        _secretRepository = secretRepository;
        _logger = logger;

        // Configurez les générateurs de secrets pour chaque type
        _secretGenerators = new Dictionary<string, Func<string>>
        {
            ["DatabasePassword"] = GenerateDatabasePassword,
            ["ApiKey"] = GenerateApiKey
        };
    }

    public async Task CheckAndRotateSecretsAsync()
    {
        var secrets = await _secretRepository.GetAllSecretsAsync();
        var overdue = secrets.Where(s => s.IsRotationOverdue).ToList();

        foreach (var secret in overdue)
        {
            await RotateSecretAsync(secret.Id);
        }
    }

    public async Task RotateSecretAsync(string secretId)
    {
        var secret = await _secretRepository.GetSecretByIdAsync(secretId);

        if (secret == null)
        {
            throw new KeyNotFoundException($"Secret with id {secretId} not found");
        }

        if (!_secretGenerators.TryGetValue(secret.Name, out var generator))
        {
            throw new InvalidOperationException($"No generator configured for secret {secret.Name}");
        }

        try
        {
            // Désactiver l'ancienne version
            var activeVersion = secret.Versions.FirstOrDefault(v => v.IsActive);
            if (activeVersion != null)
            {
                activeVersion.IsActive = false;
                activeVersion.DeactivatedAt = DateTime.UtcNow;
            }

            // Créer une nouvelle version
            var newVersion = new SecretVersion
            {
                Value = generator(),
                IsActive = true
            };

            secret.Versions.Add(newVersion);
            secret.LastRotated = DateTime.UtcNow;

            // Persister les changements
            await _secretRepository.UpdateSecretAsync(secret);

            _logger.LogInformation("Secret {SecretName} rotated successfully", secret.Name);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to rotate secret {SecretName}", secret.Name);
            throw;
        }
    }

    // Méthodes de génération de secrets
    private string GenerateDatabasePassword()
    {
        // Implémenter une logique de génération de mot de passe sécurisé
        return $"Pwd_{Guid.NewGuid().ToString().Replace("-", "")}";
    }

    private string GenerateApiKey()
    {
        // Implémenter une logique de génération de clé API
        byte[] bytes = new byte[32];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(bytes);
        }
        return Convert.ToBase64String(bytes);
    }
}
```

### Gestion de la transition lors de la rotation

Un aspect important de la rotation des secrets est de gérer la transition entre l'ancienne et la nouvelle valeur.

#### Double lecture pour les chaînes de connexion

```csharp
public class ConnectionProvider
{
    private readonly ISecretRepository _secretRepository;

    public ConnectionProvider(ISecretRepository secretRepository)
    {
        _secretRepository = secretRepository;
    }

    public async Task<string> GetConnectionStringAsync()
    {
        var secret = await _secretRepository.GetSecretByNameAsync("DatabaseConnection");
        var activeVersion = secret.Versions.FirstOrDefault(v => v.IsActive);

        if (activeVersion != null)
        {
            return activeVersion.Value;
        }

        // Fallback sur la version la plus récente si aucune n'est active
        return secret.Versions
            .OrderByDescending(v => v.CreatedAt)
            .First()
            .Value;
    }
}
```

#### Double écriture pour les mots de passe

```csharp
public class DatabasePasswordRotator
{
    private readonly IDbContext _dbContext;
    private readonly ISecretRepository _secretRepository;
    private readonly ILogger<DatabasePasswordRotator> _logger;

    public DatabasePasswordRotator(
        IDbContext dbContext,
        ISecretRepository secretRepository,
        ILogger<DatabasePasswordRotator> logger)
    {
        _dbContext = dbContext;
        _secretRepository = secretRepository;
        _logger = logger;
    }

    public async Task RotateDatabaseUserPasswordAsync(string username)
    {
        try
        {
            // Générer un nouveau mot de passe
            string newPassword = GenerateSecurePassword();

            // Étape 1: Récupérer le secret actuel
            var secret = await _secretRepository.GetSecretByNameAsync($"DbUser_{username}");
            var currentVersion = secret.Versions.FirstOrDefault(v => v.IsActive);

            if (currentVersion == null)
            {
                throw new InvalidOperationException($"No active secret found for user {username}");
            }

            // Étape 2: Créer une nouvelle version sans désactiver l'ancienne
            var newVersion = new SecretVersion
            {
                Value = newPassword,
                IsActive = true // Les deux versions sont actives pendant la transition
            };

            secret.Versions.Add(newVersion);
            await _secretRepository.UpdateSecretAsync(secret);

            // Étape 3: Mettre à jour le mot de passe dans la base de données
            await UpdateDatabasePasswordAsync(username, newPassword);

            // Étape 4: Une fois le mot de passe mis à jour dans la base de données,
            // désactiver l'ancienne version du secret
            currentVersion.IsActive = false;
            currentVersion.DeactivatedAt = DateTime.UtcNow;
            await _secretRepository.UpdateSecretAsync(secret);

            _logger.LogInformation("Password for user {Username} rotated successfully", username);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to rotate password for user {Username}", username);
            throw;
        }
    }

    private async Task UpdateDatabasePasswordAsync(string username, string newPassword)
    {
        try
        {
            // Exemple avec SQL Server
            // Attention : ceci est un exemple simplifié pour illustrer le concept
            string sql = $"ALTER LOGIN [{username}] WITH PASSWORD = '{newPassword}'";
            await _dbContext.ExecuteSqlRawAsync(sql);

            _logger.LogInformation("Database password updated for user {Username}", username);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to update database password for user {Username}", username);
            throw;
        }
    }

    private string GenerateSecurePassword()
    {
        // Générer un mot de passe fort et aléatoire
        const string validChars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*()_-+=<>?";
        const int passwordLength = 16;

        using var rng = RandomNumberGenerator.Create();
        byte[] data = new byte[passwordLength];
        rng.GetBytes(data);

        StringBuilder password = new StringBuilder(passwordLength);
        foreach (byte b in data)
        {
            password.Append(validChars[b % validChars.Length]);
        }

        return password.ToString();
    }
}
```

### Surveillance et audit de la rotation des secrets

La surveillance et l'audit sont essentiels pour s'assurer que la rotation des secrets fonctionne correctement et pour détecter toute tentative d'accès non autorisé.

```csharp
public class SecretRotationAuditService
{
    private readonly ILogger<SecretRotationAuditService> _logger;
    private readonly IAuditRepository _auditRepository;
    private readonly IEmailSender _emailSender;

    public SecretRotationAuditService(
        ILogger<SecretRotationAuditService> logger,
        IAuditRepository auditRepository,
        IEmailSender emailSender)
    {
        _logger = logger;
        _auditRepository = auditRepository;
        _emailSender = emailSender;
    }

    public async Task LogRotationEventAsync(string secretName, string action, string userId)
    {
        var auditEvent = new AuditEvent
        {
            EventType = "SecretRotation",
            ResourceName = secretName,
            Action = action,
            UserId = userId,
            Timestamp = DateTime.UtcNow,
            IPAddress = GetCurrentIpAddress(),
            Success = true
        };

        await _auditRepository.AddAuditEventAsync(auditEvent);
        _logger.LogInformation("Secret {SecretName} {Action} by {UserId}", secretName, action, userId);
    }

    public async Task AlertOnRotationFailureAsync(string secretName, Exception exception)
    {
        var auditEvent = new AuditEvent
        {
            EventType = "SecretRotation",
            ResourceName = secretName,
            Action = "RotationFailed",
            UserId = "System",
            Timestamp = DateTime.UtcNow,
            Success = false,
            ErrorMessage = exception.Message
        };

        await _auditRepository.AddAuditEventAsync(auditEvent);
        _logger.LogError(exception, "Secret rotation failed for {SecretName}", secretName);

        // Alerter les administrateurs
        await _emailSender.SendEmailAsync(
            "security-team@example.com",
            $"SECRET ROTATION FAILURE: {secretName}",
            $"La rotation du secret {secretName} a échoué.\n\nErreur: {exception.Message}\n\n" +
            $"Veuillez vérifier la configuration et résoudre ce problème rapidement.");
    }

    public async Task<List<AuditEvent>> GetRotationHistoryAsync(string secretName, DateTime? startDate = null, DateTime? endDate = null)
    {
        startDate ??= DateTime.UtcNow.AddMonths(-1);
        endDate ??= DateTime.UtcNow;

        return await _auditRepository.GetAuditEventsAsync(
            eventType: "SecretRotation",
            resourceName: secretName,
            startDate: startDate.Value,
            endDate: endDate.Value);
    }

    public async Task CheckComplianceAsync()
    {
        // Vérifier que tous les secrets ont été rotés selon la politique
        var secrets = await _auditRepository.GetResourcesWithoutRecentEventsAsync(
            eventType: "SecretRotation",
            action: "Rotated",
            olderThan: DateTime.UtcNow.AddDays(-90));

        if (secrets.Any())
        {
            _logger.LogWarning("{Count} secrets have not been rotated in the last 90 days", secrets.Count);

            // Alerter les responsables de la sécurité
            await _emailSender.SendEmailAsync(
                "security-team@example.com",
                "Secrets nécessitant une rotation",
                $"{secrets.Count} secrets n'ont pas été rotés au cours des 90 derniers jours:\n\n" +
                $"{string.Join("\n", secrets)}");
        }
    }

    private string GetCurrentIpAddress()
    {
        // Logique pour récupérer l'adresse IP actuelle
        // Dans un contexte web réel, on utiliserait HttpContext
        return "127.0.0.1";
    }
}
```

### Implémentation de la rotation de secrets dans des applications en production

Voici un exemple d'implémentation complète d'un service de rotation des secrets pour une application en production.

```csharp
public class ProductionSecretRotationService : IHostedService, IDisposable
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<ProductionSecretRotationService> _logger;
    private readonly IOptions<SecretRotationOptions> _options;
    private Timer _timer;

    public ProductionSecretRotationService(
        IServiceProvider serviceProvider,
        ILogger<ProductionSecretRotationService> logger,
        IOptions<SecretRotationOptions> options)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
        _options = options;
    }

    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Secret Rotation Service is starting.");

        // Exécuter la vérification selon l'intervalle configuré (par exemple, une fois par jour)
        _timer = new Timer(DoRotationCheck, null, TimeSpan.Zero,
            TimeSpan.FromHours(_options.Value.CheckIntervalHours));

        return Task.CompletedTask;
    }

    private async void DoRotationCheck(object state)
    {
        _logger.LogInformation("Performing secret rotation check");

        try
        {
            using (var scope = _serviceProvider.CreateScope())
            {
                var rotationManager = scope.ServiceProvider.GetRequiredService<ISecretRotationManager>();
                await rotationManager.CheckAndRotateSecretsAsync();
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred during secret rotation check");
        }
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Secret Rotation Service is stopping.");

        _timer?.Change(Timeout.Infinite, 0);

        return Task.CompletedTask;
    }

    public void Dispose()
    {
        _timer?.Dispose();
    }
}

// Options de configuration pour la rotation des secrets
public class SecretRotationOptions
{
    public int CheckIntervalHours { get; set; } = 24; // Vérification quotidienne par défaut
    public Dictionary<string, int> RotationIntervals { get; set; } = new Dictionary<string, int>
    {
        // Nombre de jours entre chaque rotation par type de secret
        ["DatabaseCredentials"] = 90,
        ["ApiKeys"] = 180,
        ["SslCertificates"] = 365,
        ["JwtSigningKeys"] = 180
    };
}
```

### Gestion de la rotation des secrets dans une architecture microservices

Dans une architecture microservices, la rotation des secrets est plus complexe car plusieurs services peuvent dépendre d'un même secret.

```csharp
public class MicroserviceSecretRotationOrchestrator
{
    private readonly ILogger<MicroserviceSecretRotationOrchestrator> _logger;
    private readonly ISecretRepository _secretRepository;
    private readonly IServiceDiscovery _serviceDiscovery;
    private readonly ISecretRotationNotifier _notifier;

    public MicroserviceSecretRotationOrchestrator(
        ILogger<MicroserviceSecretRotationOrchestrator> logger,
        ISecretRepository secretRepository,
        IServiceDiscovery serviceDiscovery,
        ISecretRotationNotifier notifier)
    {
        _logger = logger;
        _secretRepository = secretRepository;
        _serviceDiscovery = serviceDiscovery;
        _notifier = notifier;
    }

    public async Task RotateSharedSecretAsync(string secretName, string secretType)
    {
        try
        {
            // Étape 1: Identifier les services qui utilisent ce secret
            var dependentServices = await _serviceDiscovery.GetServicesUsingSecretAsync(secretName);

            if (!dependentServices.Any())
            {
                _logger.LogWarning("No services found using secret {SecretName}", secretName);
                return;
            }

            // Étape 2: Générer un nouveau secret
            var generator = GetSecretGenerator(secretType);
            string newSecretValue = generator();

            // Étape 3: Stocker la nouvelle version du secret
            // (tout en gardant l'ancienne active)
            await _secretRepository.AddSecretVersionAsync(secretName, newSecretValue, isActive: true);

            // Étape 4: Notifier tous les services de la nouvelle version
            foreach (var service in dependentServices)
            {
                await _notifier.NotifySecretRotationAsync(service.Endpoint, secretName);
                _logger.LogInformation("Notified service {ServiceName} about secret rotation", service.Name);
            }

            // Étape 5: Attendre que tous les services aient reconnu la rotation
            await WaitForServicesAcknowledgementAsync(dependentServices, secretName);

            // Étape 6: Désactiver l'ancienne version du secret
            await _secretRepository.DeactivateOldSecretVersionsAsync(secretName, keepLatestVersionsCount: 1);

            _logger.LogInformation("Secret {SecretName} rotated successfully across all services", secretName);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to rotate shared secret {SecretName}", secretName);
            throw;
        }
    }

    private async Task WaitForServicesAcknowledgementAsync(List<ServiceInfo> services, string secretName)
    {
        int maxAttempts = 10;
        int attemptDelay = 5000; // 5 secondes

        for (int attempt = 0; attempt < maxAttempts; attempt++)
        {
            var pendingServices = new List<ServiceInfo>();

            foreach (var service in services)
            {
                bool acknowledged = await _notifier.CheckSecretAcknowledgementAsync(
                    service.Endpoint, secretName);

                if (!acknowledged)
                {
                    pendingServices.Add(service);
                }
            }

            if (!pendingServices.Any())
            {
                _logger.LogInformation("All services have acknowledged the secret rotation");
                return;
            }

            _logger.LogInformation(
                "{Count} services still pending acknowledgement. Waiting...",
                pendingServices.Count);

            await Task.Delay(attemptDelay);
        }

        throw new TimeoutException(
            $"Not all services acknowledged the secret rotation after {maxAttempts} attempts");
    }

    private Func<string> GetSecretGenerator(string secretType)
    {
        return secretType.ToLower() switch
        {
            "apikey" => GenerateApiKey,
            "password" => GenerateStrongPassword,
            "jwtsigningkey" => GenerateJwtSigningKey,
            _ => throw new ArgumentException($"Unknown secret type: {secretType}")
        };
    }

    private string GenerateApiKey()
    {
        byte[] bytes = new byte[32];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(bytes);
        }
        return Convert.ToBase64String(bytes);
    }

    private string GenerateStrongPassword()
    {
        // Implémentation d'un générateur de mots de passe forts
        return $"Pwd_{Guid.NewGuid().ToString("N").Substring(0, 16)}";
    }

    private string GenerateJwtSigningKey()
    {
        byte[] key = new byte[64]; // 512 bits
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(key);
        }
        return Convert.ToBase64String(key);
    }
}
```

## Bonnes pratiques pour la gestion des secrets

Pour clôturer cette section, voici un récapitulatif des bonnes pratiques pour la gestion sécurisée des secrets dans vos applications C# :

1. **Ne jamais stocker de secrets en clair** dans le code source ou les fichiers de configuration
2. **Utiliser des outils spécialisés** comme Azure Key Vault, Secret Manager ou HashiCorp Vault
3. **Appliquer le principe du moindre privilège** pour l'accès aux secrets
4. **Chiffrer les secrets au repos et en transit** avec des algorithmes robustes
5. **Mettre en place la rotation des secrets** selon une politique adaptée
6. **Auditer et journaliser les accès** aux secrets
7. **Gérer différemment les secrets selon l'environnement** (développement, test, production)
8. **Utiliser des identités gérées** plutôt que des clés d'accès statiques lorsque possible
9. **Impléenter une gestion des erreurs robuste** pour éviter les fuites d'informations
10. **Former les développeurs** aux bonnes pratiques de gestion des secrets

### Tableau comparatif des solutions de gestion des secrets

| Solution | Cas d'usage | Avantages | Limites |
|----------|-------------|-----------|---------|
| **Secret Manager** | Développement local | Simple à configurer, intégré à .NET | Stockage non chiffré, uniquement pour le développement |
| **DPAPI** | Applications Windows | Intégré au système d'exploitation, simple à utiliser | Spécifique à Windows, clé liée à l'utilisateur ou à la machine |
| **Azure Key Vault** | Applications en production sur Azure | Géré, haute disponibilité, journalisation, rotation | Dépendance à Azure, coût supplémentaire |
| **HashiCorp Vault** | Architectures multi-cloud ou on-premise | Polyvalent, open-source, haute disponibilité | Complexité d'installation et de gestion |
| **Docker/K8s Secrets** | Applications conteneurisées | Intégré à l'orchestrateur, portable | Protection limitée au repos (selon la configuration) |
| **Variables d'environnement** | Configuration simple | Facile à utiliser, portable | Moins sécurisé, visible dans les processus, logs système |

En choisissant la bonne solution pour votre contexte et en suivant ces bonnes pratiques, vous assurez une gestion sécurisée des secrets dans vos applications C#, réduisant significativement le risque de fuites d'informations sensibles.

⏭️
