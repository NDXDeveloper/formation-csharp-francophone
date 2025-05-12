# 14.7. Configurations de déploiement

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

Le déploiement d'applications n'est pas simplement une question de transfert de fichiers d'un environnement à un autre. Les applications modernes nécessitent des configurations spécifiques à chaque environnement, des stratégies pour protéger les informations sensibles, et des plans pour gérer les migrations et les situations d'urgence. Ce chapitre vous guidera à travers les meilleures pratiques de configuration de déploiement pour garantir des transitions fluides et sécurisées de vos applications du développement à la production.

## 14.7.1. Transformations de configuration

Les applications .NET utilisent généralement des fichiers de configuration pour stocker des paramètres qui peuvent changer sans nécessiter une recompilation. Cependant, ces paramètres varient souvent entre les environnements (développement, test, production).

### Comprendre les transformations de configuration

Les transformations de configuration sont un mécanisme qui permet de modifier automatiquement les fichiers de configuration en fonction de l'environnement cible lors du déploiement.

#### Dans les applications .NET Framework

Pour les applications .NET Framework traditionnelles, les transformations de configuration fonctionnent principalement avec le fichier `Web.config` ou `App.config` :

1. **Structure de base** : Vous avez un fichier de base (`Web.config`) et des fichiers de transformation spécifiques à l'environnement (`Web.Release.config`, `Web.Debug.config`).

2. **Syntaxe de transformation** : Les transformations utilisent une syntaxe XML spéciale pour indiquer les modifications à apporter.

Exemple de fichier `Web.config` de base :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <connectionStrings>
    <add name="DefaultConnection"
         connectionString="Data Source=DevServer;Initial Catalog=MyDB;Integrated Security=True"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
  <appSettings>
    <add key="Environment" value="Development" />
    <add key="EnableDebugging" value="true" />
    <add key="ServiceUrl" value="http://localhost:8080/api" />
  </appSettings>
</configuration>
```

Exemple de fichier `Web.Release.config` avec des transformations :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <connectionStrings>
    <add name="DefaultConnection"
         connectionString="Data Source=ProdServer;Initial Catalog=MyDB;User ID=ProdUser;Password=ProdPass"
         xdt:Transform="SetAttributes" xdt:Locator="Match(name)" />
  </connectionStrings>
  <appSettings>
    <add key="Environment" value="Production"
         xdt:Transform="SetAttributes" xdt:Locator="Match(key)" />
    <add key="EnableDebugging" value="false"
         xdt:Transform="SetAttributes" xdt:Locator="Match(key)" />
    <add key="ServiceUrl" value="https://api.example.com"
         xdt:Transform="SetAttributes" xdt:Locator="Match(key)" />
  </appSettings>
</configuration>
```

#### Dans les applications ASP.NET Core / .NET 5+

Les applications .NET modernes utilisent une approche différente basée sur des fichiers JSON et le système de configuration ASP.NET Core :

1. **Fichiers de configuration par environnement** : Vous avez un fichier de base (`appsettings.json`) et des fichiers spécifiques à l'environnement (`appsettings.Development.json`, `appsettings.Production.json`).

2. **Superposition automatique** : Le système de configuration fusionne automatiquement les fichiers en fonction de l'environnement.

Exemple de fichier `appsettings.json` de base :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=DevServer;Initial Catalog=MyDB;Integrated Security=True"
  },
  "AppSettings": {
    "Environment": "Development",
    "EnableDebugging": true,
    "ServiceUrl": "http://localhost:8080/api"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

Exemple de fichier `appsettings.Production.json` :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=ProdServer;Initial Catalog=MyDB;User ID=ProdUser;Password=ProdPass"
  },
  "AppSettings": {
    "Environment": "Production",
    "EnableDebugging": false,
    "ServiceUrl": "https://api.example.com"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

### Techniques de transformation avancées

#### Transformations conditionnelles

Vous pouvez utiliser des conditions pour appliquer des transformations seulement dans certains cas :

```xml
<system.web>
  <compilation xdt:Transform="RemoveAttributes(debug)"
               xdt:Condition="@(Debug='false')" />
</system.web>
```

#### Insertion, suppression et remplacement

Différentes actions de transformation sont possibles :

```xml
<!-- Insérer un nouvel élément -->
<appSettings>
  <add key="NewSetting" value="NewValue"
       xdt:Transform="Insert" />
</appSettings>

<!-- Supprimer un élément -->
<appSettings>
  <add key="TemporarySetting" xdt:Transform="Remove"
       xdt:Locator="Match(key)" />
</appSettings>

<!-- Remplacer une section entière -->
<customSection xdt:Transform="Replace">
  <setting1>New Value 1</setting1>
  <setting2>New Value 2</setting2>
</customSection>
```

#### Transformations avec variables

Dans les processus de CI/CD modernes, vous pouvez utiliser des variables dans vos transformations :

```xml
<connectionStrings>
  <add name="DefaultConnection"
       connectionString="Data Source=#{DatabaseServer};Initial Catalog=#{DatabaseName};User ID=#{DatabaseUser};Password=#{DatabasePassword}"
       xdt:Transform="SetAttributes" xdt:Locator="Match(name)" />
</connectionStrings>
```

Ces variables (`#{DatabaseServer}`, etc.) seront remplacées par les valeurs réelles lors du déploiement par votre système CI/CD (Azure DevOps, Jenkins, etc.).

### Utilisation avec différents outils de déploiement

#### Visual Studio

Visual Studio applique automatiquement les transformations lors de la publication :

1. Cliquez avec le bouton droit sur le projet
2. Sélectionnez **Publier**
3. Choisissez un profil de publication (qui spécifie la configuration, par exemple "Release")
4. Les transformations seront appliquées lors de la publication

#### MSBuild

Pour appliquer des transformations via la ligne de commande :

```batch
msbuild MonApplication.csproj /p:Configuration=Release /p:TransformConfigFiles=true
```

#### Octopus Deploy

Octopus Deploy gère les variables et les transformations via son propre système :

```
#{ConnectionStrings:DefaultConnection}
```

#### Variables d'environnement dans .NET Core

En plus des fichiers de configuration, .NET Core prend également en charge les variables d'environnement qui peuvent remplacer n'importe quel paramètre :

```bash
# Sous Linux/Mac
export ConnectionStrings__DefaultConnection="Data Source=ProdServer;Initial Catalog=MyDB;..."

# Sous Windows (PowerShell)
$env:ConnectionStrings__DefaultConnection = "Data Source=ProdServer;Initial Catalog=MyDB;..."

# Sous Windows (CMD)
set ConnectionStrings__DefaultConnection=Data Source=ProdServer;Initial Catalog=MyDB;...
```

Dans le code, cette configuration est chargée automatiquement :

```csharp
var builder = WebApplication.CreateBuilder(args);
// La configuration inclut automatiquement les variables d'environnement
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

### Meilleures pratiques pour les transformations

1. **Structure cohérente** : Utilisez la même structure de base pour tous les environnements
2. **Minimisez les différences** : Ne transformez que ce qui est vraiment différent entre les environnements
3. **Validez les transformations** : Testez les transformations avant le déploiement réel
4. **Documentation** : Documentez les différences entre les environnements
5. **Évitez les secrets** : N'incluez pas de mots de passe ou d'informations sensibles directement dans les transformations (voir section suivante)

## 14.7.2. Gestion des secrets

Les applications modernes nécessitent souvent des informations sensibles comme des clés API, des mots de passe ou des jetons d'authentification. Ces "secrets" doivent être traités avec une attention particulière.

### Pourquoi la gestion des secrets est importante

1. **Sécurité** : Les secrets exposés peuvent conduire à des brèches de sécurité
2. **Conformité** : De nombreuses réglementations (GDPR, PCI-DSS, etc.) exigent la protection des données sensibles
3. **Rotation facile** : Les secrets doivent pouvoir être modifiés sans redéploiement de l'application
4. **Principe du moindre privilège** : Seules les personnes autorisées devraient accéder aux secrets

### Méthodes de gestion des secrets en .NET

#### 1. Secret Manager pour le développement

Pour le développement local, .NET Core fournit l'outil Secret Manager qui stocke les secrets en dehors de l'arborescence du projet :

```bash
# Initialiser Secret Manager pour votre projet
dotnet user-secrets init --project MonProjet.csproj

# Ajouter un secret
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Data Source=DevServer;..." --project MonProjet.csproj

# Ajouter un secret complexe (JSON)
dotnet user-secrets set "SmtpSettings" "{ \"Server\": \"smtp.example.com\", \"Port\": 587, \"Username\": \"user\", \"Password\": \"pass\" }" --project MonProjet.csproj

# Lister tous les secrets
dotnet user-secrets list --project MonProjet.csproj

# Supprimer un secret
dotnet user-secrets remove "ConnectionStrings:DefaultConnection" --project MonProjet.csproj
```

Les secrets sont stockés dans :
- Windows : `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`
- Mac/Linux : `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

Dans le code, ces secrets sont chargés automatiquement en développement :

```csharp
var builder = WebApplication.CreateBuilder(args);
// En développement, les secrets sont chargés automatiquement
```

#### 2. Azure Key Vault

Pour les environnements de production, Azure Key Vault est une solution robuste :

1. **Créez un Key Vault** dans Azure
2. **Ajoutez vos secrets** dans le Key Vault
3. **Configurez l'accès** pour votre application

Intégration dans votre application :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter Key Vault à la configuration
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVault:VaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential());

// Utilisation normale de la configuration
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

#### 3. AWS Secrets Manager

Si vous utilisez AWS :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter AWS Secrets Manager à la configuration
builder.Configuration.AddSecretsManager(options =>
{
    options.SecretFilter = entry => entry.Name.StartsWith("MyApp_");
    options.KeyGenerator = (entry, key) => key
        .Replace("MyApp_", string.Empty)
        .Replace("__", ":");
});

// Utilisation normale de la configuration
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

#### 4. HashiCorp Vault

Pour les environnements multi-cloud ou on-premises :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter Vault à la configuration
builder.Configuration.AddVaultConfiguration(
    options => {
        options.Address = "https://vault.example.com:8200";
        options.Token = builder.Configuration["Vault:Token"];
        options.MountPoint = "secret";
        options.SecretPath = "myapp";
    });

// Utilisation normale de la configuration
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

#### 5. Variables d'environnement (pour les conteneurs et Kubernetes)

Pour les déploiements basés sur des conteneurs :

```yaml
# docker-compose.yml
version: '3'
services:
  webapp:
    image: myapp:latest
    environment:
      - ConnectionStrings__DefaultConnection=Data Source=ProdServer;...
      - ApiKeys__ExternalService=secretkey123
```

```yaml
# kubernetes-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        env:
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: connection-string
```

### Meilleures pratiques pour la gestion des secrets

1. **Ne jamais stocker de secrets dans le code source** ou les fichiers de configuration versionnés
2. **Différents secrets pour chaque environnement**
3. **Rotation régulière des secrets** (tous les 30 à 90 jours)
4. **Audit d'accès** pour savoir qui accède aux secrets
5. **Révocation immédiate** en cas de compromission
6. **Utilisation d'une solution centralisée** plutôt que de multiples méthodes
7. **Chiffrement en transit et au repos**
8. **Gestion des secrets dans les pipelines CI/CD** (variables sécurisées)

### Mise en œuvre pratique : Protection complète des secrets

Voici un exemple complet de gestion des secrets dans une application ASP.NET Core :

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            var env = context.HostingEnvironment;

            // Configuration de base
            config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                  .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true);

            // Secrets pour le développement local
            if (env.IsDevelopment())
            {
                config.AddUserSecrets<Program>();
            }

            // Variables d'environnement (fonctionnent dans les conteneurs)
            config.AddEnvironmentVariables();

            // Charger les secrets depuis un coffre sécurisé en production
            if (!env.IsDevelopment())
            {
                var builtConfig = config.Build();
                var keyVaultName = builtConfig["KeyVault:Name"];

                if (!string.IsNullOrEmpty(keyVaultName))
                {
                    var credential = new DefaultAzureCredential(new DefaultAzureCredentialOptions
                    {
                        ManagedIdentityClientId = builtConfig["KeyVault:ManagedIdentityClientId"]
                    });

                    config.AddAzureKeyVault(
                        new Uri($"https://{keyVaultName}.vault.azure.net/"),
                        credential);
                }
            }
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

## 14.7.3. Stratégies de migration de données

Les applications stockent généralement des données dans des bases de données ou d'autres systèmes de stockage. Lors des déploiements, la migration de ces données doit être gérée avec soin.

### Principes fondamentaux de la migration de données

1. **Compatibilité descendante** : Les nouvelles versions de l'application doivent fonctionner avec les anciennes structures de données
2. **Migrations automatisées** : Les modifications de schéma doivent être appliquées automatiquement
3. **Possibilité de restauration** : Vous devez pouvoir revenir à l'état précédent en cas de problème
4. **Tests préalables** : Testez les migrations dans des environnements non-productifs
5. **Sauvegardes** : Effectuez toujours une sauvegarde avant la migration

### Stratégies de migration avec Entity Framework Core

Entity Framework Core offre un système de migrations puissant :

#### 1. Création d'une migration

Après avoir modifié vos classes de modèle, générez une migration :

```bash
dotnet ef migrations add AjoutTableProduits
```

Cela crée une classe de migration avec deux méthodes :
- `Up()` : Applique les changements
- `Down()` : Annule les changements

```csharp
public partial class AjoutTableProduits : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Produits",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Nom = table.Column<string>(maxLength: 100, nullable: false),
                Prix = table.Column<decimal>(type: "decimal(18,2)", nullable: false),
                DateCreation = table.Column<DateTime>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Produits", x => x.Id);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "Produits");
    }
}
```

#### 2. Application des migrations

Lors du déploiement, les migrations sont appliquées automatiquement :

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Au démarrage de l'application, applique les migrations automatiquement
    using (var serviceScope = app.ApplicationServices.GetRequiredService<IServiceScopeFactory>().CreateScope())
    {
        var context = serviceScope.ServiceProvider.GetService<ApplicationDbContext>();
        context.Database.Migrate();
    }

    // Suite de la méthode Configure...
}
```

Ou via la ligne de commande avant le déploiement :

```bash
dotnet ef database update
```

#### 3. Stratégies avancées avec EF Core

**Migrations personnalisées** : Pour des modifications complexes ou des manipulations de données

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // Ajout de colonne normale
    migrationBuilder.AddColumn<string>(
        name: "CodePostal",
        table: "Clients",
        maxLength: 10,
        nullable: true);

    // Exécution de SQL personnalisé
    migrationBuilder.Sql(@"
        UPDATE Clients
        SET CodePostal = '00000'
        WHERE CodePostal IS NULL
    ");

    // Maintenant on peut rendre la colonne non nullable
    migrationBuilder.AlterColumn<string>(
        name: "CodePostal",
        table: "Clients",
        maxLength: 10,
        nullable: false,
        oldMaxLength: 10,
        oldNullable: true);
}
```

**Migrations par étapes** : Pour les grosses bases de données, divisez les migrations en petites étapes

```bash
# Au lieu d'une grosse migration
dotnet ef migrations add GrosseMigration

# Utilisez plusieurs petites migrations
dotnet ef migrations add Etape1_AjoutTables
dotnet ef migrations add Etape2_ModificationColonnes
dotnet ef migrations add Etape3_AjoutContraintes
```

### Stratégies de migration avec Dapper ou ADO.NET

Pour les applications n'utilisant pas EF Core, vous pouvez créer votre propre système de migrations :

#### 1. Table de version

Créez une table qui garde trace de la version de la base de données :

```sql
CREATE TABLE [dbo].[SchemaVersions] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [ScriptName] NVARCHAR(255) NOT NULL,
    [Applied] DATETIME NOT NULL,
    CONSTRAINT [PK_SchemaVersions] PRIMARY KEY ([Id])
)
```

#### 2. Scripts de migration numérotés

Créez des scripts SQL numérotés pour chaque changement :

```
/Scripts
  /001_InitialSchema.sql
  /002_AjoutTableProduits.sql
  /003_AjoutColonneDescription.sql
```

#### 3. Exécuteur de migrations automatisé

Au démarrage de l'application :

```csharp
public class DatabaseMigrator
{
    private readonly string _connectionString;
    private readonly string _scriptsPath;

    public DatabaseMigrator(string connectionString, string scriptsPath)
    {
        _connectionString = connectionString;
        _scriptsPath = scriptsPath;
    }

    public void MigrateDatabase()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();

            // Créer la table de versions si elle n'existe pas
            CreateVersionTableIfNotExists(connection);

            // Récupérer les scripts déjà appliqués
            var appliedScripts = GetAppliedScripts(connection);

            // Récupérer tous les scripts dans le dossier
            var scriptFiles = Directory.GetFiles(_scriptsPath, "*.sql")
                                      .OrderBy(f => Path.GetFileName(f))
                                      .ToList();

            // Appliquer les scripts manquants
            foreach (var scriptFile in scriptFiles)
            {
                var scriptName = Path.GetFileName(scriptFile);

                if (!appliedScripts.Contains(scriptName))
                {
                    ApplyScript(connection, scriptName, scriptFile);
                }
            }
        }
    }

    // Autres méthodes d'implémentation...
}
```

### Migrations avec des outils dédiés

Des outils spécialisés existent également :

#### Flyway

Un outil populaire pour Java, mais qui fonctionne avec n'importe quelle base de données :

```
V1__Initial_schema.sql
V2__Add_products_table.sql
V3__Add_description_column.sql
```

#### DbUp

Spécifiquement conçu pour .NET :

```csharp
var upgrader = DeployChanges.To
    .SqlDatabase(connectionString)
    .WithScriptsFromFileSystem("Scripts")
    .LogToConsole()
    .Build();

var result = upgrader.PerformUpgrade();

if (!result.Successful)
{
    Console.WriteLine(result.Error);
    return -1;
}
```

#### Liquibase

Outil multiplateforme qui utilise XML, YAML, JSON ou SQL :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <changeSet id="1" author="dev">
        <createTable tableName="products">
            <column name="id" type="int" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="name" type="varchar(255)">
                <constraints nullable="false"/>
            </column>
            <column name="price" type="decimal(18,2)">
                <constraints nullable="false"/>
            </column>
        </createTable>
    </changeSet>
</databaseChangeLog>
```

### Meilleures pratiques pour les migrations de données

1. **Migrations incrémentales** : Petites modifications plutôt que grosses restructurations
2. **Idempotence** : Les scripts doivent pouvoir être exécutés plusieurs fois sans effets négatifs
3. **Sauvegarde avant migration** : Toujours sauvegarder avant d'appliquer des changements
4. **Tests sur des copies** : Testez les migrations sur des copies de la base de production
5. **Considérations de performances** : Attention aux impacts sur les performances pendant la migration
6. **Fenêtres de maintenance** : Planifiez les migrations pendant les périodes de faible activité
7. **Scripts de rollback** : Préparez des scripts pour annuler les migrations
8. **Journalisation détaillée** : Enregistrez tous les détails des migrations

## 14.7.4. Rollback et récupération

Malgré toutes les précautions, les déploiements peuvent parfois échouer. Il est crucial d'avoir une stratégie de rollback (retour à l'état précédent) et de récupération.

### Types de stratégies de rollback

#### 1. Rollback complet

Rétablissement complet de la version précédente :

- **Application** : Redéploiement de l'ancienne version
- **Base de données** : Restauration d'une sauvegarde ou exécution de scripts de rollback
- **Configuration** : Restauration des anciens paramètres

#### 2. Rollback partiel

Correction des problèmes spécifiques tout en conservant les autres changements :

- **Correction rapide** (hotfix) pour résoudre un problème spécifique
- **Désactivation de fonctionnalités** problématiques via des interrupteurs de fonctionnalités

#### 3. Rollforward

Au lieu de revenir en arrière, correction rapide vers l'avant :

- Déploiement d'une nouvelle version qui corrige les problèmes
- Souvent plus rapide qu'un rollback complet

### Implémentation d'un plan de rollback

#### 1. Préparation avant déploiement

Avant de déployer une nouvelle version :

```bash
# Sauvegarde de la base de données
mysqldump -u utilisateur -p --all-databases > backup_pre_deployment.sql

# Sauvegarde des fichiers de l'application
tar -czf app_backup.tar.gz /var/www/application/

# Capture de la configuration actuelle
cp /etc/application/settings.json /etc/application/settings.json.bak
```

#### 2. Scripts de rollback automatisés

Préparez des scripts qui peuvent être exécutés en cas de problème :

```bash
#!/bin/bash
# rollback.sh

# Arrêt de l'application
systemctl stop application

# Restauration des fichiers
rm -rf /var/www/application/*
tar -xzf /path/to/backups/app_backup.tar.gz -C /var/www/application/

# Restauration de la base de données
mysql -u utilisateur -p < /path/to/backups/backup_pre_deployment.sql

# Restauration de la configuration
cp /etc/application/settings.json.bak /etc/application/settings.json

# Redémarrage de l'application
systemctl start application

echo "Rollback completed successfully"
```

#### 3. Procédures de rollback pour différentes technologies

**Applications .NET (.NET Framework)** :

```powershell
# Script PowerShell de rollback pour IIS
param(
    [string]$BackupPath,
    [string]$ApplicationPath,
    [string]$SiteName,
    [string]$AppPoolName
)

# Arrêter le pool d'applications
Import-Module WebAdministration
Stop-WebAppPool -Name $AppPoolName

# Restaurer les fichiers
Remove-Item -Path $ApplicationPath -Recurse -Force
Copy-Item -Path $BackupPath -Destination $ApplicationPath -Recurse

# Redémarrer le pool d'applications
Start-WebAppPool -Name $AppPoolName

Write-Output "Rollback completed successfully"
```

**Applications .NET Core / .NET 5+** :

```bash
#!/bin/bash
# Rollback pour application .NET Core sur Linux

# Variables
APP_SERVICE="myapp.service"
BACKUP_DIR="/path/to/backups"
APP_DIR="/var/www/myapp"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Arrêter le service
systemctl stop $APP_SERVICE

# Sauvegarder la version actuelle (juste au cas où)
mkdir -p "$BACKUP_DIR/rollback_$TIMESTAMP"
cp -r $APP_DIR/* "$BACKUP_DIR/rollback_$TIMESTAMP/"

# Restaurer la version précédente
rm -rf $APP_DIR/*
cp -r $BACKUP_DIR/previous_version/* $APP_DIR/

# Attribuer les bonnes permissions
chown -R www-data:www-data $APP_DIR
chmod -R 755 $APP_DIR

# Redémarrer le service
systemctl start $APP_SERVICE

echo "Rollback to previous version completed at $TIMESTAMP"
```

#### 4. Rollback de base de données

Pour les bases de données, vous pouvez utiliser des approches spécifiques :

**Entity Framework Core** : Utilisez la commande de migration descendante

```bash
# Revenir à une migration spécifique
dotnet ef database update NomMigrationPrécédente
```

**Scripts SQL dédiés** : Préparez des scripts d'annulation pour chaque migration

```sql
-- Script de rollback pour la migration AjoutTableProduits
DROP TABLE IF EXISTS [dbo].[Produits];

-- Mettre à jour la table de versions
DELETE FROM [dbo].[SchemaVersions] WHERE [ScriptName] = '002_AjoutTableProduits.sql';
```

### Tests du plan de rollback

Le plan de rollback doit être testé régulièrement :

1. **Environnement de test** : Créez un environnement similaire à la production
2. **Simulation de déploiement** : Déployez la nouvelle version
3. **Simulation de problème** : Introduisez volontairement un problème
4. **Exécution du rollback** : Testez votre procédure de rollback
5. **Vérification de l'état du système** : Assurez-vous que le système est revenu à un état fonctionnel
6. **Documentation des résultats** : Notez les problèmes et améliorez le plan en conséquence

### Stratégies de récupération

La récupération va au-delà du simple rollback et concerne la manière dont vous gérez la situation après avoir détecté un problème.

#### 1. Détection rapide des problèmes

Pour pouvoir récupérer rapidement, vous devez d'abord détecter les problèmes :

```csharp
// Dans Program.cs ou Startup.cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Autres middlewares...

    // Middleware de surveillance de la santé de l'application
    app.UseHealthChecks("/health", new HealthCheckOptions
    {
        ResponseWriter = async (context, report) =>
        {
            context.Response.ContentType = "application/json";

            var result = new
            {
                status = report.Status.ToString(),
                components = report.Entries.Select(e => new
                {
                    component = e.Key,
                    status = e.Value.Status.ToString(),
                    description = e.Value.Description,
                    data = e.Value.Data
                })
            };

            await context.Response.WriteAsJsonAsync(result);
        }
    });

    // Connexion à un système de monitoring externe
    app.UseMiddleware<MonitoringMiddleware>();

    // Reste de la méthode Configure...
}
```

#### 2. Capacités de récupération automatique

Construisez vos applications avec des capacités de récupération automatique :

```csharp
// Politique de nouvelle tentative pour les appels à la base de données
services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        Configuration.GetConnectionString("DefaultConnection"),
        sqlOptions =>
        {
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 5,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: null);
        });
});

// Politique de nouvelle tentative pour les appels HTTP
services.AddHttpClient("resilient", client =>
{
    client.BaseAddress = new Uri("https://api.example.com/");
})
.AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(
    retryCount: 3,
    sleepDurationProvider: retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
    onRetry: (exception, retryCount, context) =>
    {
        Log.Warning($"Retrying request due to {exception.Message} (Attempt {retryCount})");
    }
))
.AddTransientHttpErrorPolicy(policy => policy.CircuitBreakerAsync(
    handledEventsAllowedBeforeBreaking: 5,
    durationOfBreak: TimeSpan.FromMinutes(1),
    onBreak: (exception, breakDelay) =>
    {
        Log.Error($"Circuit breaker opened due to {exception.Message}. Breaking for {breakDelay.TotalSeconds} seconds.");
    },
    onReset: () =>
    {
        Log.Information("Circuit breaker reset.");
    }
));
```

#### 3. Récupération progressive

Plutôt que de tout restaurer d'un coup, considérez une récupération progressive :

1. **Restaurer les fonctionnalités critiques** en premier
2. **Réactiver progressivement** les fonctionnalités moins importantes
3. **Surveiller étroitement** le système pendant la récupération

#### 4. Communication pendant la récupération

Un aspect souvent négligé mais crucial est la communication :

1. **Informez les utilisateurs** de manière transparente
2. **Définissez des attentes réalistes** sur le temps de récupération
3. **Fournissez des moyens alternatifs** pour accomplir les tâches critiques si possible
4. **Documentez l'incident** pour analyse future

```csharp
// Exemple de middleware pour afficher une page de maintenance
public class MaintenanceMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMaintenanceStatusService _maintenanceStatus;

    public MaintenanceMiddleware(RequestDelegate next, IMaintenanceStatusService maintenanceStatus)
    {
        _next = next;
        _maintenanceStatus = maintenanceStatus;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (_maintenanceStatus.IsInMaintenance)
        {
            context.Response.StatusCode = 503; // Service Unavailable
            context.Response.Headers["Retry-After"] = "300"; // Suggère de réessayer dans 5 minutes

            // Si c'est une requête API, renvoyer JSON
            if (context.Request.Path.StartsWithSegments("/api"))
            {
                context.Response.ContentType = "application/json";
                await context.Response.WriteAsJsonAsync(new
                {
                    status = "maintenance",
                    message = _maintenanceStatus.MaintenanceMessage,
                    estimatedCompletion = _maintenanceStatus.EstimatedCompletionTime
                });
            }
            else // Sinon renvoyer une page HTML
            {
                context.Response.ContentType = "text/html";
                await context.Response.WriteAsync($@"
                    <!DOCTYPE html>
                    <html>
                    <head>
                        <title>Maintenance en cours</title>
                        <style>
                            body {{ font-family: Arial, sans-serif; text-align: center; padding: 40px; }}
                            h1 {{ color: #e74c3c; }}
                            .container {{ max-width: 600px; margin: 0 auto; }}
                        </style>
                    </head>
                    <body>
                        <div class='container'>
                            <h1>Maintenance en cours</h1>
                            <p>{_maintenanceStatus.MaintenanceMessage}</p>
                            <p>Nous devrions être de retour vers : {_maintenanceStatus.EstimatedCompletionTime}</p>
                            <p>Nous nous excusons pour la gêne occasionnée.</p>
                        </div>
                    </body>
                    </html>");
            }

            return;
        }

        await _next(context);
    }
}
```

### Plan de récupération après sinistre (DRP)

Pour les situations plus graves (panne matérielle, catastrophe naturelle, etc.), vous devez avoir un plan de récupération après sinistre :

1. **Sauvegardes régulières** :
   - Sauvegardes de base de données (complètes, différentielles, journaux de transactions)
   - Sauvegardes des fichiers de l'application
   - Sauvegardes des fichiers de configuration

2. **Site de reprise** :
   - Environnement secondaire prêt à prendre le relais
   - Réplication des données vers le site de reprise
   - Procédures de basculement documentées

3. **Objectifs de récupération** :
   - RPO (Recovery Point Objective) : Combien de données pouvez-vous vous permettre de perdre ?
   - RTO (Recovery Time Objective) : Combien de temps pour restaurer le service ?

4. **Exercices réguliers** :
   - Simulez des désastres et testez votre plan
   - Chronométrez les temps de récupération
   - Identifiez les goulets d'étranglement

## 14.7.5. Blue-Green deployment

Le déploiement Blue-Green est une technique de déploiement avancée qui permet de réduire les temps d'arrêt et les risques liés aux mises à jour.

### Principe du Blue-Green

1. **Deux environnements identiques** :
   - **Blue** : L'environnement de production actuel
   - **Green** : Une copie exacte, mais avec la nouvelle version

2. **Processus** :
   - Déployer la nouvelle version sur l'environnement Green
   - Tester l'environnement Green
   - Basculer le trafic de Blue vers Green
   - L'ancien environnement Blue devient standby

3. **Avantages** :
   - Temps d'arrêt quasi nul
   - Possibilité de rollback instantané
   - Tests en conditions réelles avant basculement

### Mise en œuvre du Blue-Green

#### 1. Avec des serveurs physiques ou virtuels

Si vous utilisez des serveurs traditionnels :

1. **Configuration du load balancer** pour router le trafic vers Blue
2. **Déploiement** de la nouvelle version sur Green
3. **Tests** sur Green (avec un accès limité)
4. **Basculement** du load balancer vers Green
5. **Vérification** du bon fonctionnement
6. **Blue** devient désormais le prochain environnement Green

```
# Configuration NGINX pour Blue-Green
# /etc/nginx/nginx.conf

upstream backend {
    server blue.example.com weight=1;  # Environnement Blue
    server green.example.com weight=0;  # Environnement Green (poids 0 = inactif)
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Pour basculer vers Green, modifiez simplement les poids et rechargez NGINX :

```
# Basculement vers Green
upstream backend {
    server blue.example.com weight=0;  # Environnement Blue (poids 0 = inactif)
    server green.example.com weight=1;  # Environnement Green
}
```

```bash
# Recharger NGINX pour appliquer les changements
sudo nginx -s reload
```

#### 2. Avec des conteneurs et Kubernetes

Kubernetes facilite grandement le Blue-Green deployment :

```yaml
# Service pointant vers le déploiement Blue
apiVersion: v1
kind: Service
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    app: my-app
    version: blue  # Pointage initial vers Blue
  ports:
  - port: 80
    targetPort: 8080
---
# Déploiement Blue
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: my-app
        image: my-app:1.0.0
        ports:
        - containerPort: 8080
---
# Déploiement Green (nouvelle version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: my-app
        image: my-app:1.1.0  # Nouvelle version
        ports:
        - containerPort: 8080
```

Pour basculer, il suffit de modifier le sélecteur du service :

```bash
# Basculement du service de Blue vers Green
kubectl patch service my-app -p '{"spec":{"selector":{"version":"green"}}}'

# Vérification de la bonne santé de Green
# Si tout va bien, on peut continuer à utiliser Green
# Sinon, on peut rapidement revenir à Blue
kubectl patch service my-app -p '{"spec":{"selector":{"version":"blue"}}}'
```

#### 3. Avec Azure App Service

Azure App Service propose des "Deployment Slots" qui facilitent le Blue-Green :

```powershell
# Création d'un slot de déploiement "green"
az webapp deployment slot create --name MyApp --resource-group MyResourceGroup --slot green

# Déploiement de la nouvelle version sur le slot green
az webapp deployment source config-zip --name MyApp --resource-group MyResourceGroup --slot green --src package.zip

# Test du slot green via son URL
# https://myapp-green.azurewebsites.net

# Basculement (swap) de green vers production
az webapp deployment slot swap --name MyApp --resource-group MyResourceGroup --slot green --target-slot production

# Si nécessaire, retour en arrière (swap back)
az webapp deployment slot swap --name MyApp --resource-group MyResourceGroup --slot production --target-slot green
```

#### 4. Avec AWS

AWS propose plusieurs méthodes pour le Blue-Green :

**Avec Elastic Beanstalk** :

```bash
# Création d'un nouvel environnement clone (green)
aws elasticbeanstalk clone-environment --application-name MyApp --source-environment-name MyApp-Blue --environment-name MyApp-Green

# Déploiement de la nouvelle version sur Green
aws elasticbeanstalk update-environment --environment-name MyApp-Green --version-label v2.0.0

# Basculement des URL (swap)
aws elasticbeanstalk swap-environment-cnames --source-environment-name MyApp-Blue --destination-environment-name MyApp-Green
```

**Avec CodeDeploy** :

```json
{
  "applicationName": "MyApp",
  "deploymentGroupName": "MyDeploymentGroup",
  "revision": {
    "revisionType": "S3",
    "s3Location": {
      "bucket": "my-bucket",
      "key": "app-1.1.0.zip",
      "bundleType": "zip"
    }
  },
  "deploymentConfigName": "CodeDeployDefault.ECSAllAtOnce",
  "description": "Blue-Green deployment",
  "ignoreApplicationStopFailures": true
}
```

### Gestion de la persistance des données

Un défi majeur du Blue-Green est la gestion des données persistantes :

#### 1. Base de données partagée

La solution la plus simple mais avec des contraintes :

1. **Schéma compatible** : La nouvelle version doit être compatible avec le schéma existant
2. **Migrations sans rupture** : Déployer les changements de base de données séparément
   - Ajout de nouvelles tables/colonnes d'abord (avec valeurs par défaut)
   - Déploiement de l'application qui utilise ces nouvelles structures
   - Nettoyage des anciennes structures dans une opération ultérieure

#### 2. Double base de données et synchronisation

Une approche plus complexe :

1. **Deux bases de données** : Une pour Blue, une pour Green
2. **Synchronisation** : Réplication des données de Blue vers Green
3. **Cutover** : Moment où Green commence à écrire dans sa propre base
4. **Stratégie de réconciliation** : Gestion des modifications pendant la transition

#### 3. Base de données versionnée

Approche moderne avec coexistence des versions :

1. **Tables versionnées** : Préfixes ou suffixes de version
2. **Vues** : Présentation cohérente des données
3. **Migration progressive** : Cohabitation des anciennes et nouvelles structures

```sql
-- Exemple de schéma versionnée
CREATE TABLE Products_v1 (
    Id INT PRIMARY KEY,
    Name NVARCHAR(100),
    Price DECIMAL(18,2)
);

CREATE TABLE Products_v2 (
    Id INT PRIMARY KEY,
    Name NVARCHAR(100),
    Price DECIMAL(18,2),
    Description NVARCHAR(MAX),  -- Nouvelle colonne dans v2
    IsActive BIT NOT NULL DEFAULT 1  -- Nouvelle colonne dans v2
);

-- Vue pour la v1
CREATE VIEW Products_Current_v1 AS
SELECT Id, Name, Price FROM Products_v2;

-- Vue pour la v2
CREATE VIEW Products_Current_v2 AS
SELECT * FROM Products_v2;
```

### Validation progressive avec tests canary

Pour minimiser les risques, combinez Blue-Green avec des tests canary :

1. **Déploiement initial** : Green reçoit un petit pourcentage du trafic (5-10%)
2. **Surveillance** : Métriques de performance, taux d'erreur, etc.
3. **Augmentation progressive** : 25%, 50%, 75%, 100% du trafic
4. **Rollback automatique** : Si les métriques se dégradent

```yaml
# Exemple avec Istio Service Mesh
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app-route
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: blue
      weight: 90
    - destination:
        host: my-app
        subset: green
      weight: 10  # 10% du trafic vers Green
```

### Automatisation du Blue-Green

Pour rendre le Blue-Green vraiment efficace, automatisez le processus :

```yaml
# Exemple de pipeline CI/CD avec GitLab
stages:
  - build
  - test
  - deploy-green
  - test-green
  - switch
  - verify
  - finalize

build:
  stage: build
  script:
    - docker build -t ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA} .
    - docker push ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}

test:
  stage: test
  script:
    - run-unit-tests.sh
    - run-integration-tests.sh

deploy-green:
  stage: deploy-green
  script:
    - deploy-to-green-environment.sh ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}
  environment:
    name: green
    url: https://green.example.com

test-green:
  stage: test-green
  script:
    - run-smoke-tests.sh https://green.example.com
    - run-performance-tests.sh https://green.example.com

switch:
  stage: switch
  script:
    - switch-traffic-to-green.sh
  environment:
    name: production
    url: https://example.com
  when: manual  # Requiert une approbation manuelle

verify:
  stage: verify
  script:
    - verify-production-health.sh
  after_script:
    - |
      if [ $? -ne 0 ]; then
        rollback-to-blue.sh  # Rollback automatique si la vérification échoue
        exit 1
      fi

finalize:
  stage: finalize
  script:
    - cleanup-blue-environment.sh
  when: manual  # Requiert une confirmation manuelle
```

## Résumé et meilleures pratiques

Les configurations de déploiement modernes doivent prendre en compte de nombreux aspects pour garantir des déploiements fiables et des systèmes résilients.

### Points clés à retenir

1. **Transformations de configuration**
   - Utilisez des fichiers spécifiques à l'environnement
   - Évitez de stocker des secrets dans les fichiers de configuration
   - Automatisez les transformations dans votre pipeline CI/CD

2. **Gestion des secrets**
   - Utilisez des coffres-forts dédiés (Azure Key Vault, AWS Secrets Manager)
   - Rotation régulière des secrets
   - Accès limité aux personnes autorisées

3. **Migrations de données**
   - Planifiez des migrations incrémentales et compatibles avec les versions antérieures
   - Testez les migrations sur des copies de la production
   - Préparez toujours des scripts de rollback

4. **Rollback et récupération**
   - Testez régulièrement vos procédures de rollback
   - Implémentez des mécanismes de détection précoce des problèmes
   - Documentez clairement les procédures de récupération

5. **Blue-Green deployment**
   - Réduisez les temps d'arrêt à presque zéro
   - Combinez avec des déploiements canary pour plus de sécurité
   - Gérez soigneusement les bases de données persistantes

### Checklist de déploiement

Utilisez cette checklist pour vos déploiements :

- [ ] **Préparation**
  - [ ] Vérifier que tous les tests passent
  - [ ] Vérifier les transformations de configuration
  - [ ] Valider les scripts de migration
  - [ ] Préparer les scripts de rollback
  - [ ] Vérifier les permissions et accès

- [ ] **Backup**
  - [ ] Sauvegarder la base de données
  - [ ] Sauvegarder les fichiers d'application
  - [ ] Sauvegarder les configurations

- [ ] **Déploiement**
  - [ ] Informer les parties prenantes
  - [ ] Appliquer les migrations de base de données
  - [ ] Déployer la nouvelle version
  - [ ] Vérifier les fichiers de configuration
  - [ ] Vérifier les permissions des fichiers

- [ ] **Validation**
  - [ ] Exécuter les tests de fumée (smoke tests)
  - [ ] Vérifier les métriques clés
  - [ ] Contrôler les journaux d'erreurs
  - [ ] Valider les fonctionnalités critiques

- [ ] **Post-déploiement**
  - [ ] Informer de la réussite du déploiement
  - [ ] Mettre à jour la documentation
  - [ ] Archiver les sauvegardes
  - [ ] Réviser et améliorer le processus

## Exercices pratiques

1. **Créez un système de transformations de configuration** pour une application ASP.NET Core qui prend en charge trois environnements : Développement, Test et Production.

2. **Implémentez une gestion sécurisée des secrets** en utilisant Azure Key Vault ou AWS Secrets Manager.

3. **Créez un plan de migration de données** pour ajouter une nouvelle fonctionnalité qui nécessite des modifications du schéma de base de données.

4. **Développez et testez un script de rollback complet** qui restaure à la fois l'application et la base de données à leur état précédent.

5. **Mettez en place un déploiement Blue-Green** pour une application web simple, en utilisant soit des conteneurs Docker, soit des slots de déploiement dans Azure App Service.

⏭️ 15. [Tests et qualité de code](/15-tests-et-qualite-de-code/README.md)
