# 9.2. Connexion aux bases de données

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La connexion à une base de données est la première étape pour accéder et manipuler des données avec ADO.NET. Cette section explique comment configurer et gérer correctement les connexions à différentes bases de données.

## 9.2.1. Chaînes de connexion

Une chaîne de connexion contient les informations nécessaires pour établir une connexion à une base de données, comme le serveur, la base de données, les identifiants, et divers paramètres de connexion.

### Structure d'une chaîne de connexion

Les chaînes de connexion sont généralement constituées de paires clé/valeur séparées par des points-virgules :

```
Paramètre1=Valeur1;Paramètre2=Valeur2;Paramètre3=Valeur3
```


### Chaînes de connexion pour différentes bases de données

#### SQL Server

```textmate
// Authentification Windows (intégrée)
string connectionString = "Data Source=NomServeur;Initial Catalog=NomBaseDeDonnées;Integrated Security=True";

// Authentification SQL Server
string connectionString = "Data Source=NomServeur;Initial Catalog=NomBaseDeDonnées;User ID=Utilisateur;Password=MotDePasse";

// Connexion à une instance nommée
string connectionString = "Data Source=NomServeur\\NomInstance;Initial Catalog=NomBaseDeDonnées;Integrated Security=True";

// Connexion à SQL Server Express LocalDB (.NET Framework 4.7.2)
string connectionString = "Data Source=(LocalDB)\\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\\MaBase.mdf;Integrated Security=True";

// Connexion à SQL Server Express LocalDB (.NET 8)
string connectionString = "Data Source=(LocalDB)\\MSSQLLocalDB;AttachDbFilename=%LOCALAPPDATA%\\MaBase.mdf;Integrated Security=True";
```


#### SQLite

```textmate
// Connexion à un fichier SQLite
string connectionString = "Data Source=chemindufichier.db";

// Connexion à un fichier SQLite avec options supplémentaires
string connectionString = "Data Source=chemindufichier.db;Version=3;Password=monmotdepasse;";
```


#### MySQL

```textmate
// Connexion MySQL de base
string connectionString = "Server=adresseserveur;Database=nombasededonnées;Uid=utilisateur;Pwd=motdepasse;";

// Avec options supplémentaires
string connectionString = "Server=adresseserveur;Port=3306;Database=nombasededonnées;Uid=utilisateur;Pwd=motdepasse;SslMode=none;";
```


#### PostgreSQL

```textmate
// Connexion PostgreSQL avec Npgsql
string connectionString = "Host=adresseserveur;Port=5432;Database=nombasededonnées;Username=utilisateur;Password=motdepasse;";
```


### Paramètres courants des chaînes de connexion

Voici quelques paramètres communs pour SQL Server :

| Paramètre | Description |
|-----------|-------------|
| Data Source | Nom ou adresse IP du serveur de base de données |
| Initial Catalog | Nom de la base de données |
| User ID | Nom d'utilisateur pour l'authentification SQL Server |
| Password | Mot de passe pour l'authentification SQL Server |
| Integrated Security | Utilise l'authentification Windows si `True` |
| Connect Timeout | Délai d'attente de connexion en secondes (par défaut : 15) |
| Encrypt | Active le chiffrement SSL si `True` |
| MultipleActiveResultSets | Permet plusieurs résultats actifs simultanément (MARS) |
| Pooling | Active le regroupement de connexions |
| Max Pool Size | Nombre maximum de connexions dans le pool |
| Min Pool Size | Nombre minimum de connexions maintenues dans le pool |

### Stockage sécurisé des chaînes de connexion

#### Dans le fichier App.config ou Web.config (.NET Framework 4.7.2)

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="MaConnexion"
         connectionString="Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
</configuration>
```


Pour lire la chaîne de connexion :

```textmate
using System.Configuration;

string connectionString = ConfigurationManager.ConnectionStrings["MaConnexion"].ConnectionString;
```


> **Note :** Pour utiliser `ConfigurationManager`, vous devez ajouter une référence à `System.Configuration.dll` ou installer le package NuGet `System.Configuration.ConfigurationManager`.

#### Dans un fichier appsettings.json (.NET 8)

```json
{
  "ConnectionStrings": {
    "MaConnexion": "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True"
  }
}
```


Pour lire la chaîne de connexion :

```textmate
// Dans Program.cs ou Startup.cs pour configurer les services
var builder = WebApplication.CreateBuilder(args);
var connectionString = builder.Configuration.GetConnectionString("MaConnexion");

// Ou dans une classe où IConfiguration est injecté
public class MonService
{
    private readonly string _connectionString;

    public MonService(IConfiguration configuration)
    {
        _connectionString = configuration.GetConnectionString("MaConnexion");
    }
}
```


#### Sécuriser les chaînes de connexion

Pour les applications déployées, évitez de stocker les chaînes de connexion avec des mots de passe en clair :

- **.NET Framework 4.7.2** : Utilisez la protection de configuration (`aspnet_regiis.exe -pe`)
- **.NET 8** : Utilisez le gestionnaire de secrets (`dotnet user-secrets`) pour le développement et des variables d'environnement ou Azure Key Vault pour la production

```shell script
# .NET 8 - Initialiser les secrets utilisateur
dotnet user-secrets init

# Ajouter une chaîne de connexion
dotnet user-secrets set "ConnectionStrings:MaConnexion" "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;User ID=MonUtilisateur;Password=MonMotDePasse"
```


## 9.2.2. Connection pooling

Le "connection pooling" (regroupement de connexions) est une technique qui permet de réutiliser les connexions à la base de données au lieu d'en créer de nouvelles à chaque fois, ce qui améliore considérablement les performances.

### Fonctionnement du connection pooling

1. Lorsqu'une connexion est ouverte pour la première fois, ADO.NET vérifie si une connexion appropriée existe déjà dans le pool.
2. Si une connexion disponible est trouvée, elle est réutilisée, évitant ainsi le coût de création d'une nouvelle connexion.
3. Si aucune connexion n'est disponible et que le pool n'a pas atteint sa taille maximale, une nouvelle connexion est créée.
4. Lorsqu'une connexion est fermée, elle est retournée au pool et reste disponible pour une utilisation future.

### Configuration du connection pooling

Le connection pooling est activé par défaut dans la plupart des providers ADO.NET. Vous pouvez configurer ses paramètres dans la chaîne de connexion :

```textmate
// Configuration complète du pooling pour SQL Server
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True;" +
                          "Pooling=true;Min Pool Size=5;Max Pool Size=100;Connection Timeout=30";
```


Paramètres de pooling courants :

| Paramètre | Description | Valeur par défaut |
|-----------|-------------|------------------|
| Pooling | Active ou désactive le pooling | `true` |
| Max Pool Size | Nombre maximum de connexions dans le pool | 100 |
| Min Pool Size | Nombre minimum de connexions maintenues dans le pool | 0 |
| Connection Lifetime | Durée de vie maximale d'une connexion en secondes | 0 (illimitée) |
| Connection Reset | Réinitialise l'état de la connexion en la retirant du pool | `true` |

### Différences entre les providers

Le pooling diffère légèrement entre les providers :

- **SQL Server** : Activé par défaut, très optimisé
- **SQLite** : N'utilise pas de pooling car il s'agit d'une base de données embarquée
- **MySQL** : Implémente son propre système de pooling
- **PostgreSQL (Npgsql)** : Supporte le pooling, mais avec des paramètres différents

### Exemple avec et sans pooling

```textmate
// Avec pooling (par défaut)
string withPooling = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True;";

// Sans pooling
string withoutPooling = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True;Pooling=false;";

// Test de performance
public void TestPerformance()
{
    var stopwatch = new System.Diagnostics.Stopwatch();

    // Test avec pooling
    stopwatch.Start();
    for (int i = 0; i < 1000; i++)
    {
        using (var connection = new SqlConnection(withPooling))
        {
            connection.Open();
            // Exécuter une requête simple
            using (var command = new SqlCommand("SELECT 1", connection))
            {
                command.ExecuteScalar();
            }
        }
    }
    stopwatch.Stop();
    Console.WriteLine($"Avec pooling: {stopwatch.ElapsedMilliseconds} ms");

    // Réinitialiser le chronomètre
    stopwatch.Reset();

    // Test sans pooling
    stopwatch.Start();
    for (int i = 0; i < 1000; i++)
    {
        using (var connection = new SqlConnection(withoutPooling))
        {
            connection.Open();
            // Exécuter une requête simple
            using (var command = new SqlCommand("SELECT 1", connection))
            {
                command.ExecuteScalar();
            }
        }
    }
    stopwatch.Stop();
    Console.WriteLine($"Sans pooling: {stopwatch.ElapsedMilliseconds} ms");
}
```


### Bonnes pratiques pour le connection pooling

1. **Fermez explicitement les connexions** avec `Close()` ou `using` pour les retourner au pool.
2. **Utilisez la même chaîne de connexion** pour bénéficier du même pool.
3. **Évitez de changer l'état des connexions** (comme modifier la base de données par défaut avec `USE Database`).
4. **Évitez de garder les connexions ouvertes trop longtemps** car elles ne sont pas disponibles pour d'autres utilisations.
5. **Dimensionnez correctement le pool** en fonction de la charge prévue de votre application.

## 9.2.3. Gestion des connexions

La gestion appropriée des connexions est essentielle pour développer des applications fiables et performantes.

### Ouverture et fermeture des connexions

L'approche recommandée est d'ouvrir les connexions le plus tard possible et de les fermer le plus tôt possible.

```textmate
// Approche recommandée avec using pour garantir la fermeture des ressources
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    try
    {
        // Ouvrir la connexion juste avant d'en avoir besoin
        connection.Open();

        // Exécuter les opérations nécessaires
        using (SqlCommand command = new SqlCommand("SELECT COUNT(*) FROM Clients", connection))
        {
            int count = (int)command.ExecuteScalar();
            Console.WriteLine($"Nombre de clients : {count}");
        }

        // La connexion est automatiquement fermée à la fin du bloc using
    }
    catch (SqlException ex)
    {
        Console.WriteLine($"Erreur de base de données : {ex.Message}");
        // Gérer l'exception spécifiquement
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erreur : {ex.Message}");
        // Gérer d'autres exceptions
    }
}
```


### Gestion des exceptions spécifiques

Chaque provider a ses propres types d'exceptions. Voici comment les gérer pour SQL Server :

```textmate
try
{
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        // Code utilisant la connexion
    }
}
catch (SqlException ex)
{
    // Gérer les erreurs spécifiques à SQL Server selon le code d'erreur
    switch (ex.Number)
    {
        case 4060: // Accès refusé à la base de données
            Console.WriteLine("La base de données n'existe pas ou l'accès est refusé.");
            break;
        case 18456: // Échec de connexion
            Console.WriteLine("Nom d'utilisateur ou mot de passe incorrect.");
            break;
        case 53: // Serveur introuvable
            Console.WriteLine("Le serveur n'a pas été trouvé ou n'est pas accessible.");
            break;
        case 40: // Délai d'attente dépassé
            Console.WriteLine("Délai de connexion dépassé.");
            break;
        default:
            Console.WriteLine($"Erreur SQL : {ex.Message} (Code {ex.Number})");
            break;
    }
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Erreur d'opération : {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"Erreur générale : {ex.Message}");
}
```


### Utilisation asynchrone des connexions

#### .NET Framework 4.7.2

```textmate
// Méthode asynchrone
public async Task<int> GetCustomerCountAsync()
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        await connection.OpenAsync();

        using (SqlCommand command = new SqlCommand("SELECT COUNT(*) FROM Clients", connection))
        {
            return (int)await command.ExecuteScalarAsync();
        }
    }
}
```


#### .NET 8

```textmate
// Méthode asynchrone plus concise avec .NET 8
public async Task<int> GetCustomerCountAsync()
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    await using (SqlConnection connection = new SqlConnection(connectionString))
    {
        await connection.OpenAsync();

        await using (SqlCommand command = new SqlCommand("SELECT COUNT(*) FROM Clients", connection))
        {
            return (int)await command.ExecuteScalarAsync();
        }
    }
}
```


### Vérification de l'état de connexion

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Vérifier l'état avant d'ouvrir
    if (connection.State == ConnectionState.Closed)
    {
        connection.Open();
    }

    // Utiliser la connexion...

    // Vérifier l'état avant de fermer
    if (connection.State == ConnectionState.Open)
    {
        connection.Close();
    }
}
```


### Réouverture d'une connexion fermée

```textmate
SqlConnection connection = new SqlConnection(connectionString);

try
{
    // Premier usage
    connection.Open();
    // Exécuter des commandes...
    connection.Close();

    // Deuxième usage (réutilisation de la même connexion)
    connection.Open();
    // Exécuter d'autres commandes...
}
finally
{
    // Garantir la fermeture
    if (connection.State == ConnectionState.Open)
    {
        connection.Close();
    }

    // Libérer les ressources
    connection.Dispose();
}
```


## 9.2.4. Sécurité des connexions

La sécurité des connexions aux bases de données est cruciale pour protéger vos données.

### Méthodes d'authentification

#### Authentification Windows (intégrée)

C'est généralement l'option la plus sécurisée car elle n'exige pas de stocker des mots de passe :

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;Integrated Security=True";
```


#### Authentification par mot de passe (SQL)

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;User ID=Utilisateur;Password=MotDePasse";
```


#### Authentification Azure AD (.NET 8)

```textmate
// Nécessite Microsoft.Data.SqlClient
string connectionString = "Data Source=monserveur.database.windows.net;Initial Catalog=MaBaseDeDonnées;Authentication=Active Directory Password;User ID=utilisateur@domaine.com;Password=MotDePasse";

// Ou avec un jeton d'accès
string connectionString = "Data Source=monserveur.database.windows.net;Initial Catalog=MaBaseDeDonnées;Authentication=Active Directory Default";
```


### Chiffrement des connexions

#### SQL Server - Forcer le chiffrement

```textmate
// .NET Framework 4.7.2
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;Integrated Security=True;Encrypt=True;TrustServerCertificate=False";

// .NET 8 (le chiffrement est activé par défaut)
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;Integrated Security=True;TrustServerCertificate=False";
```


> **Note :** Dans .NET 8 avec Microsoft.Data.SqlClient, le chiffrement est activé par défaut. Utilisez `Encrypt=False` pour le désactiver (non recommandé).

#### Éviter de faire confiance aux certificats non valides

```textmate
// Ne faites confiance qu'aux certificats valides (recommandé)
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;Integrated Security=True;TrustServerCertificate=False";

// Faire confiance à tous les certificats (à éviter en production)
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;Integrated Security=True;TrustServerCertificate=True";
```


### Protection contre l'injection SQL

Même si ce n'est pas directement lié à la connexion, c'est un aspect crucial de la sécurité :

```textmate
// INCORRECT - Vulnérable à l'injection SQL
string query = "SELECT * FROM Utilisateurs WHERE Nom = '" + nomUtilisateur + "'";

// CORRECT - Utiliser des paramètres
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT * FROM Utilisateurs WHERE Nom = @Nom";
    using (SqlCommand command = new SqlCommand(query, connection))
    {
        command.Parameters.AddWithValue("@Nom", nomUtilisateur);

        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Traiter les résultats...
        }
    }
}
```


### Sécurisation des chaînes de connexion

#### .NET Framework 4.7.2 - Chiffrement de sections web.config

```xml
<!-- Section à chiffrer dans Web.config -->
<connectionStrings>
  <add name="MaConnexion"
       connectionString="Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;User ID=Utilisateur;Password=MotDePasse"
       providerName="System.Data.SqlClient" />
</connectionStrings>
```


Puis, utilisez l'outil `aspnet_regiis.exe` pour chiffrer la section :

```
aspnet_regiis -pe "connectionStrings" -app "/MonApplication" -prov "DataProtectionConfigurationProvider"
```


#### .NET 8 - Variables d'environnement

```textmate
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);

// Utiliser une variable d'environnement pour la chaîne de connexion
var connectionString = Environment.GetEnvironmentVariable("DATABASE_CONNECTION") ??
                      builder.Configuration.GetConnectionString("Default");

// Configurer les services avec la connexion...
```


Configuration pour le développement avec les secrets utilisateur :

```shell script
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnées;User ID=Utilisateur;Password=MotDePasse"
```


### Principe de privilège minimum

Appliquez le principe de privilège minimum :

1. **Créez des utilisateurs de base de données dédiés** à votre application
2. **Accordez uniquement les autorisations nécessaires**
3. **Évitez d'utiliser des comptes administratifs** (sa, dbo) dans votre application
4. **Utilisez des utilisateurs différents** pour les opérations de lecture et d'écriture si possible

### Exemple : Configuration de connexion sécurisée

```textmate
public static class DbConnectionFactory
{
    public static SqlConnection CreateSecureConnection(string environmentName)
    {
        string connectionString;

        if (environmentName.Equals("Production", StringComparison.OrdinalIgnoreCase))
        {
            // En production, utilisez des variables d'environnement ou des services sécurisés
            string server = Environment.GetEnvironmentVariable("DB_SERVER");
            string database = Environment.GetEnvironmentVariable("DB_NAME");
            string userId = Environment.GetEnvironmentVariable("DB_USER");
            string password = Environment.GetEnvironmentVariable("DB_PASSWORD");

            connectionString = $"Data Source={server};Initial Catalog={database};" +
                              $"User ID={userId};Password={password};" +
                              "Encrypt=True;TrustServerCertificate=False;Connection Timeout=30";
        }
        else
        {
            // En développement, utilisez une configuration locale
            var config = new ConfigurationBuilder()
                .AddJsonFile("appsettings.json")
                .AddUserSecrets<Program>() // Utiliser les secrets utilisateur en développement
                .Build();

            connectionString = config.GetConnectionString("Default");
        }

        return new SqlConnection(connectionString);
    }
}
```


### Audit et surveillance des connexions

Pour renforcer la sécurité :

1. **Activez l'audit SQL Server** pour enregistrer les tentatives de connexion
2. **Surveillez les journaux** pour détecter les activités suspectes
3. **Limitez les tentatives de connexion** pour prévenir les attaques par force brute
4. **Configurez des alertes** pour les échecs de connexion répétés

## Récapitulatif

La gestion correcte des connexions aux bases de données est fondamentale pour développer des applications sécurisées et performantes :

1. **Chaînes de connexion** : Stockez-les de manière sécurisée et utilisez les paramètres appropriés
2. **Connection pooling** : Utilisez-le pour améliorer les performances
3. **Gestion des connexions** : Ouvrez-les tard, fermez-les tôt, utilisez `using`
4. **Sécurité** : Chiffrez les connexions, utilisez des paramètres, appliquez le principe de privilège minimum

En suivant ces bonnes pratiques, vous créerez des applications qui accèdent efficacement et en toute sécurité à vos bases de données.

⏭️ 9.3. [Exécution de commandes SQL](/09-acces-aux-donnees-avec-ado-dotnet/9-3-execution-de-commandes-sql.md)
