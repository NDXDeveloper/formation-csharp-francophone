# 9.3. Exécution de commandes SQL

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Une fois la connexion à la base de données établie, l'étape suivante consiste à exécuter des commandes SQL pour manipuler les données. ADO.NET fournit des classes dédiées pour créer et exécuter différents types de commandes SQL de manière sécurisée et efficace.

## 9.3.1. SELECT, INSERT, UPDATE, DELETE

Les quatre opérations fondamentales sur les données (souvent appelées CRUD - Create, Read, Update, Delete) correspondent aux instructions SQL : `INSERT`, `SELECT`, `UPDATE` et `DELETE`.

### Requête SELECT (lire des données)

La requête SELECT permet de récupérer des données d'une ou plusieurs tables.

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Créer une commande SELECT
    string selectQuery = "SELECT Id, Nom, Email, DateInscription FROM Clients";
    using (SqlCommand command = new SqlCommand(selectQuery, connection))
    {
        // Exécuter la commande et obtenir un DataReader
        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Vérifier s'il y a des résultats
            if (reader.HasRows)
            {
                // Parcourir les résultats ligne par ligne
                while (reader.Read())
                {
                    int id = reader.GetInt32(0); // Ou reader.GetInt32(reader.GetOrdinal("Id"));
                    string nom = reader.GetString(1);
                    string email = reader.GetString(2);
                    DateTime dateInscription = reader.GetDateTime(3);

                    Console.WriteLine($"Client ID: {id}, Nom: {nom}, Email: {email}, Inscrit le: {dateInscription:d}");
                }
            }
            else
            {
                Console.WriteLine("Aucun client trouvé.");
            }
        }
    }
}
```


### Requête INSERT (ajouter des données)

La requête INSERT permet d'ajouter de nouvelles lignes dans une table.

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Créer une commande INSERT
    string insertQuery = "INSERT INTO Clients (Nom, Email, DateInscription) VALUES (@Nom, @Email, @DateInscription)";

    using (SqlCommand command = new SqlCommand(insertQuery, connection))
    {
        // Ajouter des paramètres (nous verrons plus de détails dans la section suivante)
        command.Parameters.AddWithValue("@Nom", "Marie Dupont");
        command.Parameters.AddWithValue("@Email", "marie.dupont@exemple.fr");
        command.Parameters.AddWithValue("@DateInscription", DateTime.Now);

        // Exécuter la commande et récupérer le nombre de lignes affectées
        int rowsAffected = command.ExecuteNonQuery();

        Console.WriteLine($"{rowsAffected} client(s) ajouté(s) avec succès.");
    }
}
```


### Récupérer l'ID généré automatiquement après un INSERT

Souvent, les tables ont une clé primaire auto-incrémentée. Voici comment récupérer cette valeur après un INSERT :

#### SQL Server

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string insertQuery = @"
        INSERT INTO Clients (Nom, Email, DateInscription)
        VALUES (@Nom, @Email, @DateInscription);
        SELECT SCOPE_IDENTITY();"; // Récupère l'ID généré

    using (SqlCommand command = new SqlCommand(insertQuery, connection))
    {
        command.Parameters.AddWithValue("@Nom", "Jean Martin");
        command.Parameters.AddWithValue("@Email", "jean.martin@exemple.fr");
        command.Parameters.AddWithValue("@DateInscription", DateTime.Now);

        // ExecuteScalar retourne la première colonne de la première ligne
        int newId = Convert.ToInt32(command.ExecuteScalar());

        Console.WriteLine($"Nouveau client ajouté avec l'ID: {newId}");
    }
}
```


#### SQLite (.NET 8)

```textmate
using Microsoft.Data.Sqlite;

using (SqliteConnection connection = new SqliteConnection("Data Source=mabase.db"))
{
    connection.Open();

    string insertQuery = @"
        INSERT INTO Clients (Nom, Email, DateInscription)
        VALUES (@Nom, @Email, @DateInscription);
        SELECT last_insert_rowid();"; // Fonction SQLite pour l'ID généré

    using (SqliteCommand command = new SqliteCommand(insertQuery, connection))
    {
        command.Parameters.AddWithValue("@Nom", "Jean Martin");
        command.Parameters.AddWithValue("@Email", "jean.martin@exemple.fr");
        command.Parameters.AddWithValue("@DateInscription", DateTime.Now);

        long newId = (long)command.ExecuteScalar();

        Console.WriteLine($"Nouveau client ajouté avec l'ID: {newId}");
    }
}
```


### Requête UPDATE (modifier des données)

La requête UPDATE permet de modifier des données existantes.

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Créer une commande UPDATE
    string updateQuery = "UPDATE Clients SET Email = @NouvelEmail WHERE Id = @ClientId";

    using (SqlCommand command = new SqlCommand(updateQuery, connection))
    {
        // Définir les paramètres
        command.Parameters.AddWithValue("@NouvelEmail", "nouveau.email@exemple.fr");
        command.Parameters.AddWithValue("@ClientId", 5); // ID du client à modifier

        // Exécuter la commande
        int rowsAffected = command.ExecuteNonQuery();

        if (rowsAffected > 0)
        {
            Console.WriteLine($"Email du client mis à jour avec succès.");
        }
        else
        {
            Console.WriteLine("Aucun client trouvé avec cet ID.");
        }
    }
}
```


### Requête DELETE (supprimer des données)

La requête DELETE permet de supprimer des lignes d'une table.

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Créer une commande DELETE
    string deleteQuery = "DELETE FROM Clients WHERE Id = @ClientId";

    using (SqlCommand command = new SqlCommand(deleteQuery, connection))
    {
        // Définir le paramètre
        command.Parameters.AddWithValue("@ClientId", 5); // ID du client à supprimer

        // Exécuter la commande
        int rowsAffected = command.ExecuteNonQuery();

        if (rowsAffected > 0)
        {
            Console.WriteLine($"Client supprimé avec succès.");
        }
        else
        {
            Console.WriteLine("Aucun client trouvé avec cet ID.");
        }
    }
}
```


### Requêtes multiples et transactions

Pour exécuter plusieurs requêtes en garantissant qu'elles sont toutes exécutées ou aucune, utilisez les transactions :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Démarrer une transaction
    using (SqlTransaction transaction = connection.BeginTransaction())
    {
        try
        {
            // Première commande: insérer une commande
            string insertOrderQuery = @"
                INSERT INTO Commandes (ClientId, DateCommande, MontantTotal)
                VALUES (@ClientId, @DateCommande, @MontantTotal);
                SELECT SCOPE_IDENTITY();";

            int newOrderId;

            using (SqlCommand command = new SqlCommand(insertOrderQuery, connection, transaction))
            {
                command.Parameters.AddWithValue("@ClientId", 10);
                command.Parameters.AddWithValue("@DateCommande", DateTime.Now);
                command.Parameters.AddWithValue("@MontantTotal", 129.99m);

                newOrderId = Convert.ToInt32(command.ExecuteScalar());
            }

            // Deuxième commande: insérer des détails de commande
            string insertDetailsQuery = @"
                INSERT INTO DetailsCommande (CommandeId, ProduitId, Quantite, PrixUnitaire)
                VALUES (@CommandeId, @ProduitId, @Quantite, @PrixUnitaire)";

            using (SqlCommand command = new SqlCommand(insertDetailsQuery, connection, transaction))
            {
                command.Parameters.AddWithValue("@CommandeId", newOrderId);
                command.Parameters.AddWithValue("@ProduitId", 101);
                command.Parameters.AddWithValue("@Quantite", 2);
                command.Parameters.AddWithValue("@PrixUnitaire", 64.99m);

                command.ExecuteNonQuery();
            }

            // Si tout s'est bien passé, valider la transaction
            transaction.Commit();
            Console.WriteLine($"Commande {newOrderId} créée avec succès.");
        }
        catch (Exception ex)
        {
            // En cas d'erreur, annuler la transaction
            transaction.Rollback();
            Console.WriteLine($"Erreur lors de la création de la commande: {ex.Message}");
        }
    }
}
```


## 9.3.2. Paramètres de commande

L'utilisation de paramètres dans les commandes SQL est essentielle pour :
- Éviter les injections SQL (problème de sécurité majeur)
- Gérer correctement les types de données
- Améliorer la performance grâce à la réutilisation des plans d'exécution

### Ajout de paramètres de base

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT * FROM Produits WHERE Categorie = @Categorie AND Prix <= @PrixMax";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        // Méthode simple d'ajout de paramètres
        command.Parameters.AddWithValue("@Categorie", "Électronique");
        command.Parameters.AddWithValue("@PrixMax", 500.00m);

        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Traitement des résultats...
        }
    }
}
```


### Ajout de paramètres avec type explicite

L'approche avec type explicite est généralement préférable car elle donne un meilleur contrôle sur les types de données :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT * FROM Produits WHERE Categorie = @Categorie AND Prix <= @PrixMax";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        // Méthode avec type explicite
        SqlParameter categorieParam = new SqlParameter("@Categorie", SqlDbType.NVarChar, 50);
        categorieParam.Value = "Électronique";
        command.Parameters.Add(categorieParam);

        SqlParameter prixParam = new SqlParameter("@PrixMax", SqlDbType.Decimal);
        prixParam.Precision = 10;
        prixParam.Scale = 2;
        prixParam.Value = 500.00m;
        command.Parameters.Add(prixParam);

        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Traitement des résultats...
        }
    }
}
```


### Paramètres de sortie

Les paramètres de sortie sont utiles pour récupérer des valeurs calculées par des procédures stockées :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Supposons une procédure stockée qui calcule le montant total des commandes d'un client
    using (SqlCommand command = new SqlCommand("CalculerMontantTotalClient", connection))
    {
        command.CommandType = CommandType.StoredProcedure;

        // Paramètre d'entrée
        command.Parameters.AddWithValue("@ClientId", 5);

        // Paramètre de sortie
        SqlParameter totalParam = new SqlParameter();
        totalParam.ParameterName = "@MontantTotal";
        totalParam.SqlDbType = SqlDbType.Decimal;
        totalParam.Precision = 10;
        totalParam.Scale = 2;
        totalParam.Direction = ParameterDirection.Output;
        command.Parameters.Add(totalParam);

        // Exécuter la procédure stockée
        command.ExecuteNonQuery();

        // Récupérer la valeur du paramètre de sortie
        decimal montantTotal = (decimal)totalParam.Value;
        Console.WriteLine($"Montant total des commandes: {montantTotal:C}");
    }
}
```


### Paramètres avec valeur de retour

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    using (SqlCommand command = new SqlCommand("AjouterProduit", connection))
    {
        command.CommandType = CommandType.StoredProcedure;

        // Paramètres d'entrée
        command.Parameters.AddWithValue("@Nom", "Smartphone XYZ");
        command.Parameters.AddWithValue("@Description", "Le dernier modèle de smartphone");
        command.Parameters.AddWithValue("@Prix", 899.99m);
        command.Parameters.AddWithValue("@CategorieId", 2);

        // Paramètre de valeur de retour
        SqlParameter returnParam = new SqlParameter();
        returnParam.ParameterName = "@ReturnValue";
        returnParam.SqlDbType = SqlDbType.Int;
        returnParam.Direction = ParameterDirection.ReturnValue;
        command.Parameters.Add(returnParam);

        // Exécuter la procédure stockée
        command.ExecuteNonQuery();

        // Récupérer la valeur de retour
        int returnValue = (int)returnParam.Value;

        if (returnValue == 0)
        {
            Console.WriteLine("Produit ajouté avec succès.");
        }
        else
        {
            Console.WriteLine($"Erreur lors de l'ajout du produit. Code: {returnValue}");
        }
    }
}
```


### Paramètres NULL

Pour gérer les valeurs NULL dans les paramètres :

```textmate
using (SqlCommand command = new SqlCommand(query, connection))
{
    // Pour une valeur qui peut être NULL
    string description = null; // ou une valeur récupérée qui peut être null

    if (description == null)
    {
        command.Parameters.AddWithValue("@Description", DBNull.Value);
    }
    else
    {
        command.Parameters.AddWithValue("@Description", description);
    }

    // Ou avec un opérateur ternaire
    command.Parameters.AddWithValue("@Description", description == null ? DBNull.Value : description);

    // Suite du code...
}
```


### Paramètres avec liste de valeurs (IN)

Pour les requêtes avec opérateur IN, vous devez construire dynamiquement la requête :

```textmate
public List<Produit> ObtenirProduitsParCategories(List<int> categorieIds)
{
    List<Produit> produits = new List<Produit>();

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();

        // Construire dynamiquement les noms de paramètres
        List<string> paramNames = new List<string>();
        for (int i = 0; i < categorieIds.Count; i++)
        {
            paramNames.Add($"@CategorieId{i}");
        }

        // Construire la requête avec la clause IN
        string query = $"SELECT * FROM Produits WHERE CategorieId IN ({string.Join(",", paramNames)})";

        using (SqlCommand command = new SqlCommand(query, connection))
        {
            // Ajouter les valeurs des paramètres
            for (int i = 0; i < categorieIds.Count; i++)
            {
                command.Parameters.AddWithValue($"@CategorieId{i}", categorieIds[i]);
            }

            using (SqlDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    Produit produit = new Produit
                    {
                        Id = reader.GetInt32(reader.GetOrdinal("Id")),
                        Nom = reader.GetString(reader.GetOrdinal("Nom")),
                        // Autres propriétés...
                    };

                    produits.Add(produit);
                }
            }
        }
    }

    return produits;
}
```


## 9.3.3. ExecuteReader, ExecuteNonQuery, ExecuteScalar

La classe `SqlCommand` fournit trois méthodes principales pour exécuter des commandes SQL, chacune adaptée à un scénario spécifique :

### ExecuteReader

Utilisé pour exécuter des requêtes qui retournent un ensemble de résultats (typiquement SELECT).

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT Id, Nom, Prix FROM Produits WHERE CategorieId = @CategorieId";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        command.Parameters.AddWithValue("@CategorieId", 3);

        // ExecuteReader retourne un SqlDataReader
        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Vérifier s'il y a des lignes
            if (!reader.HasRows)
            {
                Console.WriteLine("Aucun produit trouvé.");
                return;
            }

            // Parcourir les résultats
            while (reader.Read())
            {
                int id = reader.GetInt32(0);
                string nom = reader.GetString(1);
                decimal prix = reader.GetDecimal(2);

                Console.WriteLine($"ID: {id}, Nom: {nom}, Prix: {prix:C}");
            }
        }
    }
}
```


#### Utilisation avancée de ExecuteReader

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT Id, Nom, Prix FROM Produits; SELECT Id, Nom FROM Categories;";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        // Exécuter plusieurs jeux de résultats
        using (SqlDataReader reader = command.ExecuteReader())
        {
            // Premier jeu de résultats (Produits)
            Console.WriteLine("Liste des produits:");
            while (reader.Read())
            {
                int id = reader.GetInt32(0);
                string nom = reader.GetString(1);
                decimal prix = reader.GetDecimal(2);

                Console.WriteLine($"  ID: {id}, Nom: {nom}, Prix: {prix:C}");
            }

            // Passer au jeu de résultats suivant (Catégories)
            reader.NextResult();

            Console.WriteLine("\nListe des catégories:");
            while (reader.Read())
            {
                int id = reader.GetInt32(0);
                string nom = reader.GetString(1);

                Console.WriteLine($"  ID: {id}, Nom: {nom}");
            }
        }
    }
}
```


### ExecuteNonQuery

Utilisé pour exécuter des commandes qui ne retournent pas de résultats (INSERT, UPDATE, DELETE).

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Exemple de commande UPDATE
    string query = "UPDATE Produits SET Prix = Prix * 1.10 WHERE CategorieId = @CategorieId";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        command.Parameters.AddWithValue("@CategorieId", 3);

        // ExecuteNonQuery retourne le nombre de lignes affectées
        int rowsAffected = command.ExecuteNonQuery();

        Console.WriteLine($"{rowsAffected} produit(s) mis à jour avec succès.");
    }
}
```


### ExecuteScalar

Utilisé pour exécuter des requêtes qui retournent une seule valeur (comme COUNT, SUM, etc.).

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Exemple de requête agrégée
    string query = "SELECT COUNT(*) FROM Clients WHERE Ville = @Ville";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        command.Parameters.AddWithValue("@Ville", "Paris");

        // ExecuteScalar retourne la première colonne de la première ligne
        object result = command.ExecuteScalar();

        int count = Convert.ToInt32(result);
        Console.WriteLine($"Nombre de clients à Paris: {count}");
    }
}
```


### Versions asynchrones

Pour les applications modernes, utilisez les versions asynchrones de ces méthodes :

#### .NET Framework 4.7.2

```textmate
public async Task<List<Produit>> ObtenirProduitsAsync(int categorieId)
{
    List<Produit> produits = new List<Produit>();

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        await connection.OpenAsync();

        string query = "SELECT Id, Nom, Prix FROM Produits WHERE CategorieId = @CategorieId";

        using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@CategorieId", categorieId);

            // Version asynchrone de ExecuteReader
            using (SqlDataReader reader = await command.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    Produit produit = new Produit
                    {
                        Id = reader.GetInt32(0),
                        Nom = reader.GetString(1),
                        Prix = reader.GetDecimal(2)
                    };

                    produits.Add(produit);
                }
            }
        }
    }

    return produits;
}
```


#### .NET 8

```textmate
public async Task<List<Produit>> ObtenirProduitsAsync(int categorieId)
{
    List<Produit> produits = new List<Produit>();

    // "await using" simplifie la gestion des ressources
    await using (SqlConnection connection = new SqlConnection(connectionString))
    {
        await connection.OpenAsync();

        string query = "SELECT Id, Nom, Prix FROM Produits WHERE CategorieId = @CategorieId";

        await using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@CategorieId", categorieId);

            // Version asynchrone de ExecuteReader
            await using (SqlDataReader reader = await command.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    Produit produit = new Produit
                    {
                        Id = reader.GetInt32(0),
                        Nom = reader.GetString(1),
                        Prix = reader.GetDecimal(2)
                    };

                    produits.Add(produit);
                }
            }
        }
    }

    return produits;
}
```


## 9.3.4. Gestion des types de données

La correspondance entre les types de données SQL et C# est importante pour éviter les erreurs de conversion et garantir l'intégrité des données.

### Tableau de correspondance des types

| Type SQL Server | Type .NET | SqlDbType | Remarques |
|----------------|-----------|-----------|-----------|
| bit | bool | SqlDbType.Bit | |
| tinyint | byte | SqlDbType.TinyInt | |
| smallint | short | SqlDbType.SmallInt | |
| int | int | SqlDbType.Int | |
| bigint | long | SqlDbType.BigInt | |
| decimal, numeric | decimal | SqlDbType.Decimal | Spécifier precision et scale |
| float | double | SqlDbType.Float | |
| real | float | SqlDbType.Real | |
| money | decimal | SqlDbType.Money | |
| smallmoney | decimal | SqlDbType.SmallMoney | |
| char | string | SqlDbType.Char | Taille fixe |
| varchar | string | SqlDbType.VarChar | Taille variable |
| nchar | string | SqlDbType.NChar | Unicode, taille fixe |
| nvarchar | string | SqlDbType.NVarChar | Unicode, taille variable |
| text | string | SqlDbType.Text | Déprécié, utiliser varchar(max) |
| ntext | string | SqlDbType.NText | Déprécié, utiliser nvarchar(max) |
| date | DateTime | SqlDbType.Date | Date uniquement |
| datetime | DateTime | SqlDbType.DateTime | Date et heure |
| datetime2 | DateTime | SqlDbType.DateTime2 | Précision étendue |
| smalldatetime | DateTime | SqlDbType.SmallDateTime | Précision réduite |
| time | TimeSpan | SqlDbType.Time | Heure uniquement |
| datetimeoffset | DateTimeOffset | SqlDbType.DateTimeOffset | Avec fuseau horaire |
| uniqueidentifier | Guid | SqlDbType.UniqueIdentifier | |
| varbinary | byte[] | SqlDbType.VarBinary | Données binaires |
| image | byte[] | SqlDbType.Image | Déprécié, utiliser varbinary(max) |
| xml | string ou XmlDocument | SqlDbType.Xml | |

### Lecture des données avec type explicite

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT Id, Nom, Description, Prix, DateCreation, EstDisponible, ImageProduit FROM Produits WHERE Id = @ProduitId";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        command.Parameters.AddWithValue("@ProduitId", 101);

        using (SqlDataReader reader = command.ExecuteReader())
        {
            if (reader.Read())
            {
                // Lecture avec le type correct
                int id = reader.GetInt32(reader.GetOrdinal("Id"));
                string nom = reader.GetString(reader.GetOrdinal("Nom"));

                // Gestion des valeurs NULL
                string description = null;
                int descriptionOrdinal = reader.GetOrdinal("Description");
                if (!reader.IsDBNull(descriptionOrdinal))
                {
                    description = reader.GetString(descriptionOrdinal);
                }

                decimal prix = reader.GetDecimal(reader.GetOrdinal("Prix"));
                DateTime dateCreation = reader.GetDateTime(reader.GetOrdinal("DateCreation"));
                bool estDisponible = reader.GetBoolean(reader.GetOrdinal("EstDisponible"));

                // Lecture de données binaires
                byte[] imageData = null;
                int imageOrdinal = reader.GetOrdinal("ImageProduit");
                if (!reader.IsDBNull(imageOrdinal))
                {
                    // Pour les petites images
                    imageData = (byte[])reader["ImageProduit"];

                    // Pour les grandes images, utiliser GetBytes
                    /*
                    long size = reader.GetBytes(imageOrdinal, 0, null, 0, 0);
                    imageData = new byte[size];
                    reader.GetBytes(imageOrdinal, 0, imageData, 0, (int)size);
                    */
                }

                // Traitement des données...
                Console.WriteLine($"Produit: {id} - {nom}");
                Console.WriteLine($"Description: {description ?? "Non disponible"}");
                Console.WriteLine($"Prix: {prix:C}");
                Console.WriteLine($"Date de création: {dateCreation:d}");
                Console.WriteLine($"Disponible: {(estDisponible ? "Oui" : "Non")}");
                Console.WriteLine($"Image disponible: {(imageData != null ? "Oui" : "Non")}");
            }
        }
    }
}
```


### Lecture avec GetFieldValue<T> (.NET 8)

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    string query = "SELECT Id, Nom, Description, Prix, DateCreation FROM Produits WHERE Id = @ProduitId";

    using (SqlCommand command = new SqlCommand(query, connection))
    {
        command.Parameters.AddWithValue("@ProduitId", 101);

        using (SqlDataReader reader = command.ExecuteReader())
        {
            if (reader.Read())
            {
                // Utilisation de GetFieldValue<T> pour une conversion typée
                int id = reader.GetFieldValue<int>(reader.GetOrdinal("Id"));
                string nom = reader.GetFieldValue<string>(reader.GetOrdinal("Nom"));

                // Gestion des valeurs NULL
                int descriptionOrdinal = reader.GetOrdinal("Description");
                string description = reader.IsDBNull(descriptionOrdinal) ? null : reader.GetFieldValue<string>(descriptionOrdinal);

                decimal prix = reader.GetFieldValue<decimal>(reader.GetOrdinal("Prix"));
                DateTime dateCreation = reader.GetFieldValue<DateTime>(reader.GetOrdinal("DateCreation"));

                // Traitement des données...
            }
        }
    }
}
```


### Conversion avec type nullable

```textmate
// Fonction utilitaire pour lire des valeurs nullables
public static T? GetNullableValue<T>(SqlDataReader reader, string columnName) where T : struct
{
    int ordinal = reader.GetOrdinal(columnName);
    if (reader.IsDBNull(ordinal))
    {
        return null;
    }

    return reader.GetFieldValue<T>(ordinal);
}

// Utilisation
using (SqlDataReader reader = command.ExecuteReader())
{
    if (reader.Read())
    {
        int id = reader.GetInt32(reader.GetOrdinal("Id"));
        string nom = reader.GetString(reader.GetOrdinal("Nom"));

        // Utilisation pour des champs qui peuvent être NULL
        decimal? prix = GetNullableValue<decimal>(reader, "Prix");
        DateTime? dateExpiration = GetNullableValue<DateTime>(reader, "DateExpiration");
        int? quantite = GetNullableValue<int>(reader, "Quantite");

        Console.WriteLine($"Produit: {id} - {nom}");
        Console.WriteLine($"Prix: {prix?.ToString("C") ?? "Non défini"}");
        Console.WriteLine($"Date d'expiration: {dateExpiration?.ToString("d") ?? "Non définie"}");
        Console.WriteLine($"Quantité: {quantite?.ToString() ?? "Non définie"}");
    }
}
```


### Gestion des types de données spécifiques

#### GUID (UniqueIdentifier)

```textmate
// Lecture d'un GUID
Guid productGuid = reader.GetGuid(reader.GetOrdinal("ProductGuid"));

// Insertion d'un GUID
command.Parameters.Add("@ProductId", SqlDbType.UniqueIdentifier).Value = Guid.NewGuid();
```


#### Données XML

```textmate
// Lecture de données XML
string xmlData = reader.GetString(reader.GetOrdinal("XmlConfig"));
XmlDocument doc = new XmlDocument();
doc.LoadXml(xmlData);

// Insertion de données XML
XmlDocument doc = new XmlDocument();
doc.Load("config.xml");
command.Parameters.Add("@XmlConfig", SqlDbType.Xml).Value = doc.OuterXml;
```


#### Données géospatiales (.NET 8)

```textmate
// Nécessite le package Microsoft.SqlServer.Types
using Microsoft.SqlServer.Types;

// Lecture de données géospatiales
SqlGeography point = SqlGeography.Parse(reader.GetString(reader.GetOrdinal("Location")));
double latitude = point.Lat.Value;
double longitude = point.Long.Value;

// Insertion de données géospatiales
string wkt = $"POINT({longitude} {latitude})";
command.Parameters.Add("@Location", SqlDbType.Udt).Value = SqlGeography.Parse(wkt);
command.Parameters["@Location"].UdtTypeName = "geography";
```


#### Données temporelles

```textmate
// Lecture d'une date sans heure
DateTime dateUniquement = reader.GetDateTime(reader.GetOrdinal("DateCommande")).Date;

// Lecture d'un TimeSpan
TimeSpan heure = reader.GetTimeSpan(reader.GetOrdinal("HeureOuverture"));

// Lecture d'un DateTimeOffset
DateTimeOffset dateAvecFuseau = reader.GetDateTimeOffset(reader.GetOrdinal("DateAvecFuseau"));

// Insertion d'une date
command.Parameters.Add("@DateCommande", SqlDbType.Date).Value = DateTime.Today;

// Insertion d'une heure
command.Parameters.Add("@HeureOuverture", SqlDbType.Time).Value = new TimeSpan(9, 0, 0); // 9h00

// Insertion d'un DateTimeOffset
command.Parameters.Add("@DateAvecFuseau", SqlDbType.DateTimeOffset).Value = DateTimeOffset.Now;
```


### Gestion des données binaires (BLOB)

#### Lecture de petits fichiers binaires

```textmate
byte[] imageData = null;
if (!reader.IsDBNull(reader.GetOrdinal("ImageProduit")))
{
    imageData = (byte[])reader["ImageProduit"];

    // Exemple: sauvegarder l'image récupérée
    File.WriteAllBytes("produit.jpg", imageData);
}
```


#### Lecture de grands fichiers binaires (streaming)

```textmate
long size = 0;
byte[] buffer = new byte[8040]; // Taille du buffer
int bytesRead;
int totalBytesRead = 0;

using (FileStream fs = new FileStream("grandefichier.pdf", FileMode.Create, FileAccess.Write))
{
    // Obtenir la taille totale
    size = reader.GetBytes(reader.GetOrdinal("FichierPDF"), 0, null, 0, 0);

    // Lire les données par blocs
    while (totalBytesRead < size)
    {
        bytesRead = (int)reader.GetBytes(reader.GetOrdinal("FichierPDF"), totalBytesRead, buffer, 0, buffer.Length);
        totalBytesRead += bytesRead;
        fs.Write(buffer, 0, bytesRead);

        // Afficher la progression
        Console.WriteLine($"Téléchargement: {totalBytesRead} sur {size} octets ({(double)totalBytesRead / size:P2})");
    }
}
```


#### Insertion de fichiers binaires

```textmate
// Pour un petit fichier
byte[] imageData = File.ReadAllBytes("produit.jpg");
command.Parameters.Add("@ImageProduit", SqlDbType.VarBinary).Value = imageData;

// Pour un fichier plus volumineux
using (FileStream fs = new FileStream("document.pdf", FileMode.Open, FileAccess.Read))
{
    byte[] fileData = new byte[fs.Length];
    fs.Read(fileData, 0, (int)fs.Length);

    command.Parameters.Add("@Document", SqlDbType.VarBinary, (int)fs.Length).Value = fileData;
}
```


### Problèmes courants et solutions

#### Conversion de type échouée

```textmate
// Problème potentiel
decimal prix = (decimal)reader["Prix"]; // Peut échouer si la valeur est NULL

// Solution sûre
decimal prix = 0;
if (!reader.IsDBNull(reader.GetOrdinal("Prix")))
{
    prix = reader.GetDecimal(reader.GetOrdinal("Prix"));
}
```


#### Éviter les erreurs de conversion de chaîne

```textmate
// Problème potentiel
string description = reader["Description"].ToString(); // Si NULL, retourne ""

// Solution sûre
string description = null;
int ordinal = reader.GetOrdinal("Description");
if (!reader.IsDBNull(ordinal))
{
    description = reader.GetString(ordinal);
}
```


#### Gérer les types décimaux avec précision

```textmate
// Problème potentiel
command.Parameters.AddWithValue("@Prix", 123.45m); // La précision peut être perdue

// Solution - spécifier explicitement le type et la précision
SqlParameter prixParam = command.Parameters.Add("@Prix", SqlDbType.Decimal);
prixParam.Precision = 10; // Nombre total de chiffres
prixParam.Scale = 2;      // Nombre de décimales
prixParam.Value = 123.45m;
```


### Exemple complet avec différents types de données

Voici un exemple complet qui montre comment gérer différents types de données en insérant et récupérant un produit :

```textmate
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Description { get; set; }
    public decimal? Prix { get; set; }
    public int? QuantiteStock { get; set; }
    public DateTime DateCreation { get; set; }
    public DateTime? DateExpiration { get; set; }
    public bool EstDisponible { get; set; }
    public Guid ReferenceUnique { get; set; }
    public byte[] Image { get; set; }
}

public int AjouterProduit(Produit produit)
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();

        string query = @"
            INSERT INTO Produits (
                Nom, Description, Prix, QuantiteStock, DateCreation,
                DateExpiration, EstDisponible, ReferenceUnique, Image
            )
            VALUES (
                @Nom, @Description, @Prix, @QuantiteStock, @DateCreation,
                @DateExpiration, @EstDisponible, @ReferenceUnique, @Image
            );

            SELECT SCOPE_IDENTITY();";

        using (SqlCommand command = new SqlCommand(query, connection))
        {
            // Paramètres texte
            command.Parameters.Add("@Nom", SqlDbType.NVarChar, 100).Value = produit.Nom;

            // Gestion des valeurs potentiellement NULL
            if (string.IsNullOrEmpty(produit.Description))
                command.Parameters.Add("@Description", SqlDbType.NVarChar).Value = DBNull.Value;
            else
                command.Parameters.Add("@Description", SqlDbType.NVarChar).Value = produit.Description;

            // Paramètres décimaux avec précision
            SqlParameter prixParam = command.Parameters.Add("@Prix", SqlDbType.Decimal);
            prixParam.Precision = 10;
            prixParam.Scale = 2;
            prixParam.Value = produit.Prix.HasValue ? (object)produit.Prix.Value : DBNull.Value;

            // Paramètres entiers nullables
            command.Parameters.Add("@QuantiteStock", SqlDbType.Int).Value =
                produit.QuantiteStock.HasValue ? (object)produit.QuantiteStock.Value : DBNull.Value;

            // Paramètres date/heure
            command.Parameters.Add("@DateCreation", SqlDbType.DateTime2).Value = produit.DateCreation;
            command.Parameters.Add("@DateExpiration", SqlDbType.Date).Value =
                produit.DateExpiration.HasValue ? (object)produit.DateExpiration.Value : DBNull.Value;

            // Paramètres booléens
            command.Parameters.Add("@EstDisponible", SqlDbType.Bit).Value = produit.EstDisponible;

            // Paramètre GUID
            command.Parameters.Add("@ReferenceUnique", SqlDbType.UniqueIdentifier).Value =
                produit.ReferenceUnique != Guid.Empty ? produit.ReferenceUnique : Guid.NewGuid();

            // Paramètre binaire (image)
            if (produit.Image == null || produit.Image.Length == 0)
                command.Parameters.Add("@Image", SqlDbType.VarBinary).Value = DBNull.Value;
            else
                command.Parameters.Add("@Image", SqlDbType.VarBinary).Value = produit.Image;

            // Exécuter la commande et récupérer l'ID généré
            return Convert.ToInt32(command.ExecuteScalar());
        }
    }
}

public Produit ObtenirProduit(int id)
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();

        string query = @"
            SELECT
                Id, Nom, Description, Prix, QuantiteStock, DateCreation,
                DateExpiration, EstDisponible, ReferenceUnique, Image
            FROM Produits
            WHERE Id = @Id";

        using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = command.ExecuteReader())
            {
                if (reader.Read())
                {
                    Produit produit = new Produit
                    {
                        Id = reader.GetInt32(reader.GetOrdinal("Id")),
                        Nom = reader.GetString(reader.GetOrdinal("Nom"))
                    };

                    // Lecture de champs potentiellement NULL
                    int descriptionOrdinal = reader.GetOrdinal("Description");
                    if (!reader.IsDBNull(descriptionOrdinal))
                    {
                        produit.Description = reader.GetString(descriptionOrdinal);
                    }

                    int prixOrdinal = reader.GetOrdinal("Prix");
                    if (!reader.IsDBNull(prixOrdinal))
                    {
                        produit.Prix = reader.GetDecimal(prixOrdinal);
                    }

                    int quantiteOrdinal = reader.GetOrdinal("QuantiteStock");
                    if (!reader.IsDBNull(quantiteOrdinal))
                    {
                        produit.QuantiteStock = reader.GetInt32(quantiteOrdinal);
                    }

                    produit.DateCreation = reader.GetDateTime(reader.GetOrdinal("DateCreation"));

                    int dateExpirationOrdinal = reader.GetOrdinal("DateExpiration");
                    if (!reader.IsDBNull(dateExpirationOrdinal))
                    {
                        produit.DateExpiration = reader.GetDateTime(dateExpirationOrdinal);
                    }

                    produit.EstDisponible = reader.GetBoolean(reader.GetOrdinal("EstDisponible"));
                    produit.ReferenceUnique = reader.GetGuid(reader.GetOrdinal("ReferenceUnique"));

                    int imageOrdinal = reader.GetOrdinal("Image");
                    if (!reader.IsDBNull(imageOrdinal))
                    {
                        produit.Image = (byte[])reader["Image"];
                    }

                    return produit;
                }

                return null; // Produit non trouvé
            }
        }
    }
}
```


## En résumé

L'exécution de commandes SQL est au cœur de l'accès aux données avec ADO.NET. Les points clés à retenir sont :

1. **Utilisez les bonnes commandes SQL** pour vos opérations (SELECT, INSERT, UPDATE, DELETE).

2. **Paramétrez toujours vos requêtes** pour éviter les injections SQL et assurer une gestion correcte des types.

3. **Choisissez la méthode d'exécution appropriée** :
   - `ExecuteReader` pour récupérer des résultats
   - `ExecuteNonQuery` pour les opérations sans résultat
   - `ExecuteScalar` pour récupérer une seule valeur

4. **Gérez correctement les types de données** :
   - Utilisez les méthodes typées comme `GetInt32()`, `GetString()`, etc.
   - Vérifiez toujours les valeurs NULL avec `IsDBNull()`
   - Utilisez les méthodes génériques comme `GetFieldValue<T>()` dans .NET 8

5. **Pour les applications modernes**, utilisez les versions asynchrones des méthodes d'exécution.

En suivant ces bonnes pratiques, vous créerez des applications robustes qui interagissent efficacement avec vos bases de données.

⏭️
