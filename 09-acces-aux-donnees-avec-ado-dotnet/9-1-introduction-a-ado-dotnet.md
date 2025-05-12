# 9.1. Introduction à ADO.NET

🔝 Retour au [Sommaire](/SOMMAIRE.md)

ADO.NET est la technologie de Microsoft pour l'accès aux données dans les applications .NET. Elle fournit un ensemble de classes qui permettent d'interagir avec différentes sources de données, comme les bases de données relationnelles, les fichiers XML ou les services web.

## 9.1.1. Architecture d'ADO.NET

ADO.NET est conçu autour d'une architecture modulaire et flexible qui s'adapte à différents scénarios d'accès aux données.

### Composants principaux

L'architecture d'ADO.NET est composée de plusieurs couches et composants :

![Architecture ADO.NET](https://i.imgur.com/example.png)

1. **Providers de données** : Composants spécifiques à chaque type de base de données
2. **DataSet** : Représentation en mémoire des données, indépendante de la source
3. **DataReader** : Accès en lecture seule, en avant uniquement et optimisé pour les données
4. **Connection** : Gère la connexion à la base de données
5. **Command** : Exécute des commandes SQL ou des procédures stockées
6. **DataAdapter** : Fait le pont entre le DataSet et la source de données

### Flux de données

Le flux de données typique dans ADO.NET suit ces étapes :

1. Établir une connexion à la base de données
2. Créer et exécuter une commande SQL
3. Récupérer les résultats via un DataReader ou un DataAdapter
4. Si nécessaire, manipuler les données dans un DataSet
5. Mettre à jour la base de données avec les modifications

### Espaces de noms fondamentaux

ADO.NET utilise plusieurs espaces de noms clés :

```textmate
// Espaces de noms communs pour ADO.NET
using System.Data;              // Classes de base et interfaces
using System.Data.Common;       // Classes de base pour les providers
using System.Data.SqlClient;    // Provider pour SQL Server (.NET Framework)
using Microsoft.Data.SqlClient; // Provider pour SQL Server (.NET 8)
using System.Data.OleDb;        // Provider OLE DB
using System.Data.Odbc;         // Provider ODBC
```


## 9.1.2. Providers de données

Les providers de données sont des composants qui fournissent un accès spécifique à différents types de bases de données. Ils implémentent les interfaces standard d'ADO.NET pour permettre un accès uniforme.

### Providers courants

| Provider | Espace de noms | Utilisé pour |
|----------|----------------|--------------|
| SQL Server | `System.Data.SqlClient` (.NET Framework)<br>`Microsoft.Data.SqlClient` (.NET 8) | Bases de données Microsoft SQL Server |
| OLE DB | `System.Data.OleDb` | Sources de données OLE DB comme Access, Excel |
| ODBC | `System.Data.Odbc` | Sources de données ODBC |
| SQLite | `Microsoft.Data.Sqlite` | Bases de données SQLite |
| MySQL | `MySql.Data.MySqlClient` | Bases de données MySQL |
| PostgreSQL | `Npgsql` | Bases de données PostgreSQL |
| Oracle | `Oracle.ManagedDataAccess.Client` | Bases de données Oracle |

### Différences entre .NET Framework 4.7.2 et .NET 8

#### .NET Framework 4.7.2

Dans .NET Framework, les providers sont généralement inclus dans le framework ou disponibles via NuGet :

```textmate
// SQL Server dans .NET Framework
using System.Data.SqlClient;

string connectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";
using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Utilisation de la connexion
}
```


#### .NET 8

Dans .NET 8, la plupart des providers sont disponibles sous forme de packages NuGet séparés :

```textmate
// SQL Server dans .NET 8
using Microsoft.Data.SqlClient;

string connectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";
using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Utilisation de la connexion
}
```


Pour utiliser SQL Server avec .NET 8, vous devez ajouter le package NuGet :

```
dotnet add package Microsoft.Data.SqlClient
```


### Choisir le provider approprié

Le choix du provider dépend de plusieurs facteurs :

1. **Type de base de données** : Choisissez le provider natif pour votre base de données (ex: SqlClient pour SQL Server)
2. **Fonctionnalités requises** : Certains providers offrent des fonctionnalités spécifiques
3. **Performance** : Les providers natifs sont généralement plus performants
4. **Compatibilité** : Vérifiez la compatibilité avec votre version de base de données

### Exemple : Connexion à différentes bases de données

#### SQL Server

```textmate
// .NET Framework 4.7.2
using System.Data.SqlClient;

// .NET 8
// using Microsoft.Data.SqlClient;

string sqlConnectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";
using (SqlConnection connection = new SqlConnection(sqlConnectionString))
{
    connection.Open();

    using (SqlCommand command = new SqlCommand("SELECT * FROM Customers", connection))
    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            string customerName = reader["CustomerName"].ToString();
            Console.WriteLine(customerName);
        }
    }
}
```


#### SQLite (.NET 8)

```textmate
// Ajouter le package : dotnet add package Microsoft.Data.Sqlite
using Microsoft.Data.Sqlite;

string sqliteConnectionString = "Data Source=database.db";
using (SqliteConnection connection = new SqliteConnection(sqliteConnectionString))
{
    connection.Open();

    using (SqliteCommand command = new SqliteCommand("SELECT * FROM Customers", connection))
    using (SqliteDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            string customerName = reader["CustomerName"].ToString();
            Console.WriteLine(customerName);
        }
    }
}
```


#### MySQL

```textmate
// Ajouter le package : dotnet add package MySql.Data
using MySql.Data.MySqlClient;

string mysqlConnectionString = "Server=localhost;Database=mydb;Uid=username;Pwd=password;";
using (MySqlConnection connection = new MySqlConnection(mysqlConnectionString))
{
    connection.Open();

    using (MySqlCommand command = new MySqlCommand("SELECT * FROM Customers", connection))
    using (MySqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            string customerName = reader["CustomerName"].ToString();
            Console.WriteLine(customerName);
        }
    }
}
```


## 9.1.3. Modèle connecté vs déconnecté

ADO.NET propose deux approches principales pour travailler avec les données : le modèle connecté et le modèle déconnecté.

### Modèle connecté

Le modèle connecté maintient une connexion ouverte avec la base de données pendant la durée des opérations.

#### Caractéristiques

- Utilise principalement les objets `Connection`, `Command` et `DataReader`
- La connexion reste ouverte pendant le traitement des données
- Accès en lecture seule, en avant uniquement (avec `DataReader`)
- Utilisation optimale des ressources mémoire
- Idéal pour des opérations rapides ou des données volumineuses

#### Exemple de modèle connecté

```textmate
// Exemple avec SQL Server
string connectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Ouvrir la connexion
    connection.Open();

    // Créer la commande
    using (SqlCommand command = new SqlCommand("SELECT * FROM Products WHERE CategoryID = @CategoryID", connection))
    {
        // Ajouter des paramètres (évite les injections SQL)
        command.Parameters.AddWithValue("@CategoryID", 5);

        // Exécuter la commande et obtenir un DataReader
        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Parcourir les résultats ligne par ligne
            while (reader.Read())
            {
                int productId = (int)reader["ProductID"];
                string productName = reader["ProductName"].ToString();
                decimal price = reader["UnitPrice"] == DBNull.Value ? 0 : (decimal)reader["UnitPrice"];

                Console.WriteLine($"ID: {productId}, Nom: {productName}, Prix: {price:C}");
            }
        }
    }

    // La connexion est fermée automatiquement à la fin du bloc using
}
```


#### Avantages du modèle connecté

- Meilleure performance pour les lectures simples
- Utilisation minimale de la mémoire
- Plus simple pour les opérations CRUD basiques
- Idéal pour les applications avec beaucoup d'utilisateurs

#### Inconvénients du modèle connecté

- Maintient la connexion ouverte plus longtemps
- Moins flexible pour la manipulation de données complexes
- Ne permet pas de naviguer librement dans les résultats

### Modèle déconnecté

Le modèle déconnecté utilise une copie locale des données, ce qui permet de travailler sans connexion persistante à la base de données.

#### Caractéristiques

- Utilise principalement les objets `DataSet`, `DataTable` et `DataAdapter`
- La connexion est ouverte uniquement pour charger ou mettre à jour les données
- Permet la navigation bidirectionnelle dans les données
- Les données peuvent être manipulées sans connexion active
- Idéal pour les applications nécessitant une manipulation complexe des données

#### Exemple de modèle déconnecté

```textmate
// Exemple avec SQL Server
string connectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";

// Créer le DataSet qui contiendra les données
DataSet dataSet = new DataSet();

using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Créer un adaptateur de données
    SqlDataAdapter adapter = new SqlDataAdapter();

    // Configurer la commande de sélection
    adapter.SelectCommand = new SqlCommand("SELECT * FROM Customers", connection);

    // Configurer les commandes d'insertion, mise à jour et suppression
    SqlCommandBuilder commandBuilder = new SqlCommandBuilder(adapter);
    adapter.InsertCommand = commandBuilder.GetInsertCommand();
    adapter.UpdateCommand = commandBuilder.GetUpdateCommand();
    adapter.DeleteCommand = commandBuilder.GetDeleteCommand();

    // Remplir le DataSet (la connexion est ouverte et fermée automatiquement)
    adapter.Fill(dataSet, "Customers");

    // Maintenant nous pouvons travailler avec les données sans connexion active
    DataTable customersTable = dataSet.Tables["Customers"];

    // Afficher les données
    foreach (DataRow row in customersTable.Rows)
    {
        string customerName = row["CustomerName"].ToString();
        string contactName = row["ContactName"].ToString();
        Console.WriteLine($"Client: {customerName}, Contact: {contactName}");
    }

    // Modifier des données
    DataRow newRow = customersTable.NewRow();
    newRow["CustomerID"] = "NEWID";
    newRow["CustomerName"] = "Nouvelle Entreprise";
    newRow["ContactName"] = "John Doe";
    newRow["Country"] = "France";
    customersTable.Rows.Add(newRow);

    // Mettre à jour une ligne existante
    DataRow existingRow = customersTable.Rows[0];
    existingRow["ContactName"] = "Jane Smith";

    // Envoyer les modifications à la base de données
    adapter.Update(dataSet, "Customers");
}
```


#### Avantages du modèle déconnecté

- Réduit le temps de connexion à la base de données
- Permet une navigation flexible dans les données
- Support des relations entre tables et contraintes
- Idéal pour les applications avec interface utilisateur complexe

#### Inconvénients du modèle déconnecté

- Utilisation plus importante de la mémoire
- Performance moindre pour les grandes quantités de données
- Code plus complexe
- Risques de conflits lors de la mise à jour (problèmes de concurrence)

### Comparaison des modèles

| Aspect | Modèle connecté | Modèle déconnecté |
|--------|----------------|-------------------|
| Classes principales | Connection, Command, DataReader | DataSet, DataTable, DataAdapter |
| État de connexion | Maintenue pendant les opérations | Ouverte uniquement pour charger/enregistrer |
| Utilisation mémoire | Faible | Plus élevée |
| Navigation | Unidirectionnelle, en avant | Bidirectionnelle |
| Cas d'utilisation | Opérations simples, lecture seule, reporting | Applications UI, modifications batch, données complexes |
| Facilité d'utilisation | Plus simple | Plus complexe |

### Exemple hybride : DataAdapter avec DataTable

On peut aussi utiliser un hybride des deux approches, en utilisant un DataAdapter pour remplir un DataTable (plus léger qu'un DataSet complet) :

```textmate
// Exemple hybride
string connectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";

// Créer une seule DataTable (plus légère qu'un DataSet complet)
DataTable productsTable = new DataTable("Products");

using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Créer et configurer l'adaptateur
    SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Products WHERE CategoryID = @CategoryID", connection);
    adapter.SelectCommand.Parameters.AddWithValue("@CategoryID", 5);

    // Remplir la DataTable
    adapter.Fill(productsTable);
}

// Travailler avec les données hors connexion
foreach (DataRow row in productsTable.Rows)
{
    Console.WriteLine($"Produit: {row["ProductName"]}, Prix: {row["UnitPrice"]:C}");
}

// Modifier les données
productsTable.Rows[0]["UnitPrice"] = (decimal)productsTable.Rows[0]["UnitPrice"] * 1.1m;

// Enregistrer les modifications (rouvre la connexion)
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Products WHERE CategoryID = @CategoryID", connection);
    adapter.SelectCommand.Parameters.AddWithValue("@CategoryID", 5);

    SqlCommandBuilder commandBuilder = new SqlCommandBuilder(adapter);
    adapter.Update(productsTable);
}
```


### Nouveautés dans .NET 8

Dans .NET 8, il y a plusieurs améliorations pour ADO.NET :

#### Async/Await amélioré

```textmate
// Exemple d'accès asynchrone aux données dans .NET 8
string connectionString = "Data Source=ServerName;Initial Catalog=DatabaseName;Integrated Security=True";

await using (SqlConnection connection = new SqlConnection(connectionString))
{
    await connection.OpenAsync();

    await using (SqlCommand command = new SqlCommand("SELECT * FROM Products", connection))
    await using (SqlDataReader reader = await command.ExecuteReaderAsync())
    {
        while (await reader.ReadAsync())
        {
            // Traiter les données
            string productName = reader["ProductName"].ToString();
            Console.WriteLine(productName);
        }
    }
}
```


#### Gestion améliorée des types nullable

```textmate
// Gestion des nullable dans .NET 8
await using (SqlConnection connection = new SqlConnection(connectionString))
{
    await connection.OpenAsync();

    // Utiliser GetFieldValue<T> pour gérer les nullables
    await using (SqlCommand command = new SqlCommand("SELECT ProductID, ProductName, UnitPrice FROM Products", connection))
    await using (SqlDataReader reader = await command.ExecuteReaderAsync())
    {
        while (await reader.ReadAsync())
        {
            int productId = reader.GetFieldValue<int>(0);
            string productName = reader.GetFieldValue<string>(1);

            // Gestion propre des nullables
            decimal? unitPrice = reader.IsDBNull(2) ? null : reader.GetFieldValue<decimal>(2);

            Console.WriteLine($"ID: {productId}, Nom: {productName}, Prix: {unitPrice:C}");
        }
    }
}
```


## En résumé

ADO.NET est une technologie puissante et flexible pour l'accès aux données dans les applications .NET. Elle offre deux modèles principaux :

1. **Modèle connecté** : Idéal pour les opérations simples, rapides et peu gourmandes en mémoire, utilisant `Connection`, `Command` et `DataReader`.

2. **Modèle déconnecté** : Parfait pour les applications nécessitant une manipulation complexe des données sans connexion persistante, utilisant `DataSet`, `DataTable` et `DataAdapter`.

Le choix entre ces modèles dépend des besoins spécifiques de votre application, notamment en termes de performance, de fonctionnalités et de complexité.

Dans les prochaines sections, nous explorerons plus en détail comment utiliser ces modèles pour effectuer des opérations CRUD (Create, Read, Update, Delete) et comment gérer efficacement les connexions et les transactions avec ADO.NET.

⏭️
