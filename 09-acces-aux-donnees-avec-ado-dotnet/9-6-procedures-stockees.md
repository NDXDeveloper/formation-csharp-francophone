 # 9.6. Procédures stockées

 🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les procédures stockées sont des programmes SQL précompilés et stockés dans la base de données. Elles permettent d'encapsuler la logique métier directement au niveau de la base de données, ce qui présente de nombreux avantages en termes de performance, de sécurité et de maintenance.

## 9.6.1. Appel de procédures stockées

### Concepts de base

Une procédure stockée est un ensemble d'instructions SQL qui est stocké dans la base de données sous un nom spécifique. Au lieu d'envoyer plusieurs requêtes SQL individuelles, vous pouvez appeler cette procédure et exécuter toutes ces instructions en une seule fois.

### Appel simple d'une procédure stockée

Voici comment appeler une procédure stockée simple avec ADO.NET :

```textmate
// Exemple pour .NET Framework 4.7.2
using System;
using System.Data;
using System.Data.SqlClient;

string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Création de la commande
    SqlCommand command = new SqlCommand("ObtenirClientsParVille", connection);

    // Spécifier que c'est une procédure stockée
    command.CommandType = CommandType.StoredProcedure;

    // Ajouter un paramètre
    command.Parameters.AddWithValue("@Ville", "Paris");

    // Ouvrir la connexion
    connection.Open();

    // Exécuter la procédure stockée et lire les résultats
    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine($"Client: {reader["Nom"]} {reader["Prenom"]}");
        }
    }
}
```


Voici la même opération avec .NET 8 en utilisant les fonctionnalités asynchrones :

```textmate
// Exemple pour .NET 8
using System;
using System.Threading.Tasks;
using Microsoft.Data.SqlClient;

string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

await using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("ObtenirClientsParVille", connection);
    command.CommandType = CommandType.StoredProcedure;
    command.Parameters.AddWithValue("@Ville", "Paris");

    await connection.OpenAsync();

    await using (SqlDataReader reader = await command.ExecuteReaderAsync())
    {
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"Client: {reader["Nom"]} {reader["Prenom"]}");
        }
    }
}
```


### Création d'une procédure stockée SQL Server (pour référence)

Bien que la création des procédures stockées ne fasse pas partie d'ADO.NET, il est utile de comprendre à quoi ressemble une procédure stockée simple dans SQL Server :

```sql
-- Exemple de procédure stockée SQL Server
CREATE PROCEDURE ObtenirClientsParVille
    @Ville NVARCHAR(50)
AS
BEGIN
    SELECT ClientID, Nom, Prenom, Email, Telephone
    FROM Clients
    WHERE Ville = @Ville
    ORDER BY Nom, Prenom;
END
```


## 9.6.2. Paramètres IN/OUT

Les procédures stockées peuvent avoir différents types de paramètres :
- Paramètres d'entrée (IN) : pour fournir des données à la procédure
- Paramètres de sortie (OUT) : pour récupérer des valeurs calculées par la procédure
- Paramètres d'entrée-sortie (INOUT) : combinaison des deux

### Paramètres d'entrée (IN)

Les paramètres d'entrée sont les plus courants et sont utilisés pour envoyer des données à la procédure stockée.

```textmate
// Exemple avec plusieurs paramètres d'entrée
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("AjouterClient", connection);
    command.CommandType = CommandType.StoredProcedure;

    // Ajouter plusieurs paramètres d'entrée
    command.Parameters.AddWithValue("@Nom", "Dupont");
    command.Parameters.AddWithValue("@Prenom", "Jean");
    command.Parameters.AddWithValue("@Email", "jean.dupont@exemple.fr");
    command.Parameters.AddWithValue("@Ville", "Paris");
    command.Parameters.AddWithValue("@DateInscription", DateTime.Now);

    connection.Open();

    // Exécuter la procédure sans attendre de résultats
    int lignesAffectees = command.ExecuteNonQuery();

    Console.WriteLine($"{lignesAffectees} ligne(s) affectée(s)");
}
```


### Paramètres de sortie (OUT)

Les paramètres de sortie permettent à la procédure stockée de renvoyer des valeurs calculées.

```textmate
// Procédure stockée SQL Server avec paramètre de sortie (pour référence)
/*
CREATE PROCEDURE CalculerTotalCommandes
    @ClientID INT,
    @Total DECIMAL(10, 2) OUTPUT
AS
BEGIN
    SELECT @Total = SUM(Montant)
    FROM Commandes
    WHERE ClientID = @ClientID;

    -- Si aucune commande trouvée, retourner 0
    IF @Total IS NULL
        SET @Total = 0;
END
*/

// Exemple d'utilisation du paramètre de sortie
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("CalculerTotalCommandes", connection);
    command.CommandType = CommandType.StoredProcedure;

    // Paramètre d'entrée
    command.Parameters.AddWithValue("@ClientID", 123);

    // Paramètre de sortie
    SqlParameter totalParam = new SqlParameter();
    totalParam.ParameterName = "@Total";
    totalParam.SqlDbType = SqlDbType.Decimal;
    totalParam.Precision = 10;
    totalParam.Scale = 2;
    totalParam.Direction = ParameterDirection.Output;
    command.Parameters.Add(totalParam);

    connection.Open();

    // Exécuter la procédure
    command.ExecuteNonQuery();

    // Récupérer la valeur du paramètre de sortie
    decimal totalCommandes = (decimal)totalParam.Value;
    Console.WriteLine($"Total des commandes : {totalCommandes:C}");
}
```


### Paramètres d'entrée-sortie (INOUT)

Les paramètres d'entrée-sortie permettent d'envoyer une valeur à la procédure stockée et de récupérer une valeur modifiée.

```textmate
// Procédure stockée SQL Server avec paramètre d'entrée-sortie (pour référence)
/*
CREATE PROCEDURE ConvertirDevise
    @Montant DECIMAL(10, 2) OUTPUT,
    @DeviseSource NVARCHAR(3),
    @DeviseDestination NVARCHAR(3)
AS
BEGIN
    DECLARE @TauxChange DECIMAL(10, 4);

    -- Trouver le taux de change entre les devises
    SELECT @TauxChange = Taux
    FROM TauxDeChange
    WHERE DeviseSource = @DeviseSource AND DeviseDestination = @DeviseDestination;

    -- Convertir le montant
    SET @Montant = @Montant * @TauxChange;
END
*/

// Exemple d'utilisation du paramètre d'entrée-sortie
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("ConvertirDevise", connection);
    command.CommandType = CommandType.StoredProcedure;

    // Paramètre d'entrée-sortie
    SqlParameter montantParam = new SqlParameter();
    montantParam.ParameterName = "@Montant";
    montantParam.SqlDbType = SqlDbType.Decimal;
    montantParam.Precision = 10;
    montantParam.Scale = 2;
    montantParam.Direction = ParameterDirection.InputOutput;
    montantParam.Value = 100.00m; // Valeur initiale
    command.Parameters.Add(montantParam);

    // Paramètres d'entrée
    command.Parameters.AddWithValue("@DeviseSource", "EUR");
    command.Parameters.AddWithValue("@DeviseDestination", "USD");

    connection.Open();

    // Exécuter la procédure
    command.ExecuteNonQuery();

    // Récupérer la valeur modifiée
    decimal montantConverti = (decimal)montantParam.Value;
    Console.WriteLine($"Montant converti : {montantConverti:C} USD");
}
```


### Paramètres avec valeurs par défaut

Certains paramètres de procédure stockée peuvent avoir des valeurs par défaut. Dans ce cas, vous pouvez choisir de ne pas les spécifier lors de l'appel.

```textmate
// Procédure stockée SQL Server avec paramètre optionnel (pour référence)
/*
CREATE PROCEDURE RechercherProduits
    @MotCle NVARCHAR(100),
    @CategorieProduit INT = NULL, -- Paramètre optionnel avec NULL par défaut
    @PrixMax DECIMAL(10, 2) = NULL -- Paramètre optionnel avec NULL par défaut
AS
BEGIN
    SELECT *
    FROM Produits
    WHERE Nom LIKE '%' + @MotCle + '%'
    AND (@CategorieProduit IS NULL OR CategorieID = @CategorieProduit)
    AND (@PrixMax IS NULL OR Prix <= @PrixMax);
END
*/

// Exemple d'utilisation avec certains paramètres omis
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("RechercherProduits", connection);
    command.CommandType = CommandType.StoredProcedure;

    // Paramètre obligatoire
    command.Parameters.AddWithValue("@MotCle", "ordinateur");

    // Spécifier seulement un des paramètres optionnels
    command.Parameters.AddWithValue("@PrixMax", 1000.00m);
    // Omettre @CategorieProduit qui utilisera sa valeur par défaut (NULL)

    connection.Open();

    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine($"Produit: {reader["Nom"]}, Prix: {reader["Prix"]:C}");
        }
    }
}
```


## 9.6.3. Récupération des résultats

Les procédures stockées peuvent retourner des données de différentes manières, et ADO.NET offre plusieurs façons de les récupérer.

### Récupération via SqlDataReader

Le `SqlDataReader` est efficace pour lire les ensembles de résultats en mode connecté :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("ObtenirDetailsClient", connection);
    command.CommandType = CommandType.StoredProcedure;
    command.Parameters.AddWithValue("@ClientID", 123);

    connection.Open();

    using (SqlDataReader reader = command.ExecuteReader())
    {
        // Lecture du premier ensemble de résultats (informations client)
        if (reader.Read())
        {
            Console.WriteLine("Informations client:");
            Console.WriteLine($"ID: {reader["ClientID"]}");
            Console.WriteLine($"Nom: {reader["Nom"]} {reader["Prenom"]}");
            Console.WriteLine($"Email: {reader["Email"]}");
        }

        // Passer au deuxième ensemble de résultats (commandes du client)
        reader.NextResult();

        Console.WriteLine("\nCommandes du client:");
        while (reader.Read())
        {
            Console.WriteLine($"Commande #{reader["CommandeID"]} - Date: {reader["DateCommande"]} - Montant: {reader["Montant"]:C}");
        }

        // Passer au troisième ensemble de résultats (statistiques)
        reader.NextResult();

        if (reader.Read())
        {
            Console.WriteLine($"\nNombre total de commandes: {reader["NbCommandes"]}");
            Console.WriteLine($"Montant total: {reader["MontantTotal"]:C}");
            Console.WriteLine($"Panier moyen: {reader["PanierMoyen"]:C}");
        }
    }
}
```


### Récupération via DataSet

Pour le modèle déconnecté, vous pouvez utiliser un `DataSet` pour récupérer tous les ensembles de résultats d'une procédure stockée :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlDataAdapter adapter = new SqlDataAdapter("ObtenirDetailsClient", connection);
    adapter.SelectCommand.CommandType = CommandType.StoredProcedure;
    adapter.SelectCommand.Parameters.AddWithValue("@ClientID", 123);

    DataSet ds = new DataSet();
    adapter.Fill(ds);

    // Accéder au premier ensemble de résultats (informations client)
    if (ds.Tables.Count > 0 && ds.Tables[0].Rows.Count > 0)
    {
        DataRow client = ds.Tables[0].Rows[0];
        Console.WriteLine("Informations client:");
        Console.WriteLine($"ID: {client["ClientID"]}");
        Console.WriteLine($"Nom: {client["Nom"]} {client["Prenom"]}");
        Console.WriteLine($"Email: {client["Email"]}");
    }

    // Accéder au deuxième ensemble de résultats (commandes du client)
    if (ds.Tables.Count > 1)
    {
        Console.WriteLine("\nCommandes du client:");
        foreach (DataRow commande in ds.Tables[1].Rows)
        {
            Console.WriteLine($"Commande #{commande["CommandeID"]} - Date: {commande["DateCommande"]} - Montant: {commande["Montant"]:C}");
        }
    }

    // Accéder au troisième ensemble de résultats (statistiques)
    if (ds.Tables.Count > 2 && ds.Tables[2].Rows.Count > 0)
    {
        DataRow stats = ds.Tables[2].Rows[0];
        Console.WriteLine($"\nNombre total de commandes: {stats["NbCommandes"]}");
        Console.WriteLine($"Montant total: {stats["MontantTotal"]:C}");
        Console.WriteLine($"Panier moyen: {stats["PanierMoyen"]:C}");
    }
}
```


### Récupération de valeurs uniques

Si votre procédure stockée renvoie une seule valeur, vous pouvez utiliser `ExecuteScalar` :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("CompterClientsVille", connection);
    command.CommandType = CommandType.StoredProcedure;
    command.Parameters.AddWithValue("@Ville", "Paris");

    connection.Open();

    // Récupérer une valeur unique
    int nombreClients = (int)command.ExecuteScalar();

    Console.WriteLine($"Nombre de clients à Paris : {nombreClients}");
}
```


### Récupération de valeurs de retour

Les procédures stockées SQL Server peuvent avoir une valeur de retour (différente des paramètres OUT) :

```textmate
// Procédure stockée SQL Server avec valeur de retour (pour référence)
/*
CREATE PROCEDURE InsererClient
    @Nom NVARCHAR(50),
    @Prenom NVARCHAR(50),
    @Email NVARCHAR(100)
AS
BEGIN
    -- Vérifier si l'email existe déjà
    IF EXISTS (SELECT 1 FROM Clients WHERE Email = @Email)
    BEGIN
        -- Retourner -1 si le client existe déjà
        RETURN -1;
    END
    ELSE
    BEGIN
        -- Insérer le nouveau client
        INSERT INTO Clients (Nom, Prenom, Email)
        VALUES (@Nom, @Prenom, @Email);

        -- Retourner l'ID du nouveau client
        RETURN SCOPE_IDENTITY();
    END
END
*/

// Exemple de récupération de la valeur de retour
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand("InsererClient", connection);
    command.CommandType = CommandType.StoredProcedure;

    // Ajouter les paramètres
    command.Parameters.AddWithValue("@Nom", "Martin");
    command.Parameters.AddWithValue("@Prenom", "Sophie");
    command.Parameters.AddWithValue("@Email", "sophie.martin@exemple.fr");

    // Ajouter un paramètre pour la valeur de retour
    SqlParameter returnValue = new SqlParameter();
    returnValue.Direction = ParameterDirection.ReturnValue;
    command.Parameters.Add(returnValue);

    connection.Open();

    // Exécuter la procédure
    command.ExecuteNonQuery();

    // Récupérer la valeur de retour
    int result = (int)returnValue.Value;

    if (result == -1)
    {
        Console.WriteLine("Un client avec cet email existe déjà.");
    }
    else
    {
        Console.WriteLine($"Nouveau client créé avec l'ID : {result}");
    }
}
```


### Gestion des résultats avec .NET 8 (asynchrone)

En .NET 8, vous pouvez utiliser les méthodes asynchrones pour plus d'efficacité :

```textmate
using Microsoft.Data.SqlClient;
using System.Threading.Tasks;

// Récupération via SqlDataReader (asynchrone)
public async Task AfficherDetailsClientAsync(int clientId)
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    await using (SqlConnection connection = new SqlConnection(connectionString))
    {
        SqlCommand command = new SqlCommand("ObtenirDetailsClient", connection);
        command.CommandType = CommandType.StoredProcedure;
        command.Parameters.AddWithValue("@ClientID", clientId);

        await connection.OpenAsync();

        await using (SqlDataReader reader = await command.ExecuteReaderAsync())
        {
            // Lecture du premier ensemble de résultats (informations client)
            if (await reader.ReadAsync())
            {
                Console.WriteLine("Informations client:");
                Console.WriteLine($"ID: {reader["ClientID"]}");
                Console.WriteLine($"Nom: {reader["Nom"]} {reader["Prenom"]}");
                Console.WriteLine($"Email: {reader["Email"]}");
            }

            // Passer au deuxième ensemble de résultats (commandes du client)
            await reader.NextResultAsync();

            Console.WriteLine("\nCommandes du client:");
            while (await reader.ReadAsync())
            {
                Console.WriteLine($"Commande #{reader["CommandeID"]} - Date: {reader["DateCommande"]} - Montant: {reader["Montant"]:C}");
            }
        }
    }
}
```


## 9.6.4. Bonnes pratiques

### Sécurité et prévention des injections SQL

Contrairement aux requêtes SQL directes, les procédures stockées offrent une protection naturelle contre les injections SQL. Cependant, il est important de suivre certaines bonnes pratiques :

```textmate
// À FAIRE : Utiliser des paramètres nommés
using (SqlCommand command = new SqlCommand("RechercherClientsParNom", connection))
{
    command.CommandType = CommandType.StoredProcedure;
    command.Parameters.AddWithValue("@Nom", nomRecherche);
    // ...
}

// À ÉVITER : Concaténer des chaînes SQL
string sql = "EXEC RechercherClientsParNom '" + nomRecherche + "'"; // Risque d'injection SQL !
using (SqlCommand command = new SqlCommand(sql, connection))
{
    // ...
}
```


### Gestion des exceptions

Capturer et gérer les exceptions spécifiques aux procédures stockées :

```textmate
try
{
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        SqlCommand command = new SqlCommand("TraiterCommande", connection);
        command.CommandType = CommandType.StoredProcedure;
        command.Parameters.AddWithValue("@CommandeID", 123);

        connection.Open();
        command.ExecuteNonQuery();

        Console.WriteLine("Commande traitée avec succès");
    }
}
catch (SqlException ex)
{
    if (ex.Number == 50000) // Code d'erreur personnalisé dans la procédure stockée
    {
        Console.WriteLine($"Erreur métier: {ex.Message}");
    }
    else if (ex.Number == 2627 || ex.Number == 2601) // Violations de contrainte unique
    {
        Console.WriteLine("Une entrée avec ces données existe déjà.");
    }
    else
    {
        Console.WriteLine($"Erreur SQL: {ex.Message} (Erreur #{ex.Number})");
    }
}
catch (Exception ex)
{
    Console.WriteLine($"Erreur inattendue: {ex.Message}");
}
```


### Performance

#### Durée de vie des connexions

```textmate
// Réutiliser les connexions pour plusieurs appels
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Premier appel
    using (SqlCommand command1 = new SqlCommand("ProcedureA", connection))
    {
        command1.CommandType = CommandType.StoredProcedure;
        command1.ExecuteNonQuery();
    }

    // Deuxième appel avec la même connexion
    using (SqlCommand command2 = new SqlCommand("ProcedureB", connection))
    {
        command2.CommandType = CommandType.StoredProcedure;
        command2.ExecuteNonQuery();
    }
}
```


#### Réutilisation des commandes

```textmate
// Réutiliser la même commande avec des paramètres différents
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    using (SqlCommand command = new SqlCommand("InsererProduit", connection))
    {
        command.CommandType = CommandType.StoredProcedure;

        // Paramètres communs à toutes les exécutions
        SqlParameter nomParam = command.Parameters.Add("@Nom", SqlDbType.NVarChar, 50);
        SqlParameter prixParam = command.Parameters.Add("@Prix", SqlDbType.Decimal);
        prixParam.Precision = 10;
        prixParam.Scale = 2;

        // Premier produit
        nomParam.Value = "Produit A";
        prixParam.Value = 19.99m;
        command.ExecuteNonQuery();

        // Deuxième produit (réutilisation de la commande)
        nomParam.Value = "Produit B";
        prixParam.Value = 29.99m;
        command.ExecuteNonQuery();

        // Troisième produit
        nomParam.Value = "Produit C";
        prixParam.Value = 39.99m;
        command.ExecuteNonQuery();
    }
}
```


#### Mise en cache des procédures

```textmate
// Utiliser CommandType.StoredProcedure pour permettre la mise en cache
SqlCommand command = new SqlCommand("MaProcedureStockee", connection);
command.CommandType = CommandType.StoredProcedure; // SQL Server peut mettre en cache le plan d'exécution
```


### Transactions

Utiliser des transactions pour regrouper plusieurs appels de procédures stockées :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Démarrer une transaction
    using (SqlTransaction transaction = connection.BeginTransaction())
    {
        try
        {
            // Première procédure stockée
            using (SqlCommand command1 = new SqlCommand("InsererCommande", connection, transaction))
            {
                command1.CommandType = CommandType.StoredProcedure;
                command1.Parameters.AddWithValue("@ClientID", 123);
                command1.Parameters.AddWithValue("@DateCommande", DateTime.Now);

                // Paramètre de sortie pour l'ID de la commande
                SqlParameter commandeIdParam = new SqlParameter("@CommandeID", SqlDbType.Int);
                commandeIdParam.Direction = ParameterDirection.Output;
                command1.Parameters.Add(commandeIdParam);

                command1.ExecuteNonQuery();

                int commandeId = (int)commandeIdParam.Value;

                // Deuxième procédure stockée (utilisant l'ID généré)
                using (SqlCommand command2 = new SqlCommand("InsererLigneCommande", connection, transaction))
                {
                    command2.CommandType = CommandType.StoredProcedure;
                    command2.Parameters.AddWithValue("@CommandeID", commandeId);
                    command2.Parameters.AddWithValue("@ProduitID", 456);
                    command2.Parameters.AddWithValue("@Quantite", 2);
                    command2.Parameters.AddWithValue("@PrixUnitaire", 29.99m);

                    command2.ExecuteNonQuery();
                }

                // Troisième procédure stockée
                using (SqlCommand command3 = new SqlCommand("MettreAJourStock", connection, transaction))
                {
                    command3.CommandType = CommandType.StoredProcedure;
                    command3.Parameters.AddWithValue("@ProduitID", 456);
                    command3.Parameters.AddWithValue("@QuantiteRetiree", 2);

                    command3.ExecuteNonQuery();
                }
            }

            // Si tout s'est bien passé, valider la transaction
            transaction.Commit();
            Console.WriteLine("Commande enregistrée avec succès");
        }
        catch (Exception ex)
        {
            // En cas d'erreur, annuler la transaction
            transaction.Rollback();
            Console.WriteLine($"Erreur lors de l'enregistrement de la commande: {ex.Message}");
        }
    }
}
```


### Organisation du code

Encapsulez les appels de procédures stockées dans des méthodes dédiées pour une meilleure réutilisation :

```textmate
public class ClientService
{
    private readonly string _connectionString;

    public ClientService(string connectionString)
    {
        _connectionString = connectionString;
    }

    public Client ObtenirClient(int clientId)
    {
        using (SqlConnection connection = new SqlConnection(_connectionString))
        {
            SqlCommand command = new SqlCommand("ObtenirClientParID", connection);
            command.CommandType = CommandType.StoredProcedure;
            command.Parameters.AddWithValue("@ClientID", clientId);

            connection.Open();

            using (SqlDataReader reader = command.ExecuteReader())
            {
                if (reader.Read())
                {
                    return new Client
                    {
                        ClientID = (int)reader["ClientID"],
                        Nom = reader["Nom"].ToString(),
                        Prenom = reader["Prenom"].ToString(),
                        Email = reader["Email"].ToString(),
                        Telephone = reader["Telephone"]?.ToString(),
                        DateInscription = (DateTime)reader["DateInscription"]
                    };
                }

                return null;
            }
        }
    }

    public List<Commande> ObtenirCommandesClient(int clientId)
    {
        List<Commande> commandes = new List<Commande>();

        using (SqlConnection connection = new SqlConnection(_connectionString))
        {
            SqlCommand command = new SqlCommand("ObtenirCommandesClient", connection);
            command.CommandType = CommandType.StoredProcedure;
            command.Parameters.AddWithValue("@ClientID", clientId);

            connection.Open();

            using (SqlDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    commandes.Add(new Commande
                    {
                        CommandeID = (int)reader["CommandeID"],
                        DateCommande = (DateTime)reader["DateCommande"],
                        Statut = reader["Statut"].ToString(),
                        MontantTotal = (decimal)reader["MontantTotal"]
                    });
                }
            }
        }

        return commandes;
    }

    // Version asynchrone pour .NET 8
    public async Task<Client> ObtenirClientAsync(int clientId)
    {
        await using (SqlConnection connection = new SqlConnection(_connectionString))
        {
            SqlCommand command = new SqlCommand("ObtenirClientParID", connection);
            command.CommandType = CommandType.StoredProcedure;
            command.Parameters.AddWithValue("@ClientID", clientId);

            await connection.OpenAsync();

            await using (SqlDataReader reader = await command.ExecuteReaderAsync())
            {
                if (await reader.ReadAsync())
                {
                    return new Client
                    {
                        ClientID = (int)reader["ClientID"],
                        Nom = reader["Nom"].ToString(),
                        Prenom = reader["Prenom"].ToString(),
                        Email = reader["Email"].ToString(),
                        Telephone = reader["Telephone"]?.ToString(),
                        DateInscription = (DateTime)reader["DateInscription"]
                    };
                }

                return null;
            }
        }
    }
}

// Classes modèles
public class Client
{
    public int ClientID { get; set; }
    public string Nom { get; set; }
    public string Prenom { get; set; }
    public string Email { get; set; }
    public string Telephone { get; set; }
    public DateTime DateInscription { get; set; }
}

public class Commande
{
    public int CommandeID { get; set; }
    public DateTime DateCommande { get; set; }
    public string Statut { get; set; }
    public decimal MontantTotal { get; set; }
}
```

### Tests et debugging

Simplifiez le débogage en utilisant des méthodes de test pour les procédures stockées :

```textmate
// Méthode de test pour vérifier le fonctionnement d'une procédure stockée
public void TesterProcedureCalculMontantTotal()
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();

        Console.WriteLine("Test de la procédure CalculMontantTotal");
        Console.WriteLine("--------------------------------------");

        int clientID = 123;
        DateTime dateDebut = new DateTime(2023, 1, 1);
        DateTime dateFin = new DateTime(2023, 12, 31);

        SqlCommand command = new SqlCommand("CalculMontantTotal", connection);
        command.CommandType = CommandType.StoredProcedure;

        // Paramètres d'entrée
        command.Parameters.AddWithValue("@ClientID", clientID);
        command.Parameters.AddWithValue("@DateDebut", dateDebut);
        command.Parameters.AddWithValue("@DateFin", dateFin);

        // Paramètre de sortie
        SqlParameter totalParam = new SqlParameter("@MontantTotal", SqlDbType.Decimal);
        totalParam.Direction = ParameterDirection.Output;
        totalParam.Precision = 10;
        totalParam.Scale = 2;
        command.Parameters.Add(totalParam);

        // Paramètre de sortie pour le nombre de commandes
        SqlParameter nbCommandesParam = new SqlParameter("@NbCommandes", SqlDbType.Int);
        nbCommandesParam.Direction = ParameterDirection.Output;
        command.Parameters.Add(nbCommandesParam);

        // Journaliser les valeurs d'entrée
        Console.WriteLine($"Paramètres d'entrée:");
        Console.WriteLine($"  ClientID: {clientID}");
        Console.WriteLine($"  DateDebut: {dateDebut:yyyy-MM-dd}");
        Console.WriteLine($"  DateFin: {dateFin:yyyy-MM-dd}");

        try
        {
            // Exécuter la procédure
            command.ExecuteNonQuery();

            // Récupérer les valeurs de sortie
            decimal montantTotal = (decimal)totalParam.Value;
            int nbCommandes = (int)nbCommandesParam.Value;

            // Afficher les résultats
            Console.WriteLine("\nRésultats:");
            Console.WriteLine($"  Nombre de commandes: {nbCommandes}");
            Console.WriteLine($"  Montant total: {montantTotal:C}");

            // Vérifier les résultats
            if (nbCommandes > 0)
            {
                Console.WriteLine($"  Montant moyen par commande: {montantTotal / nbCommandes:C}");
            }

            Console.WriteLine("\nTest réussi !");
        }
        catch (SqlException ex)
        {
            Console.WriteLine($"\nErreur SQL ({ex.Number}): {ex.Message}");

            // Afficher des informations spécifiques sur certaines erreurs
            if (ex.Number == 50000) // Erreur déclenchée par RAISERROR dans la procédure
            {
                Console.WriteLine($"Message personnalisé de la procédure: {ex.Message}");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"\nErreur générale: {ex.Message}");
        }
    }
}
```


### Logging des appels

Implémentez un système de logging pour suivre les appels aux procédures stockées :

```textmate
public T ExecuteProcedure<T>(string procedureName, Func<SqlCommand, T> executor, params SqlParameter[] parameters)
{
    Stopwatch stopwatch = Stopwatch.StartNew();
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    try
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            SqlCommand command = new SqlCommand(procedureName, connection);
            command.CommandType = CommandType.StoredProcedure;

            // Ajouter les paramètres
            if (parameters != null)
            {
                command.Parameters.AddRange(parameters);
            }

            // Journaliser l'appel (début)
            Console.WriteLine($"Début appel procédure {procedureName} à {DateTime.Now}");
            Console.WriteLine("Paramètres:");
            foreach (SqlParameter param in command.Parameters)
            {
                if (param.Direction != ParameterDirection.Output &&
                    param.Direction != ParameterDirection.ReturnValue)
                {
                    Console.WriteLine($"  {param.ParameterName}: {param.Value ?? "NULL"}");
                }
            }

            connection.Open();

            // Exécuter la procédure
            T result = executor(command);

            // Journaliser les paramètres de sortie
            Console.WriteLine("Paramètres de sortie:");
            foreach (SqlParameter param in command.Parameters)
            {
                if (param.Direction == ParameterDirection.Output ||
                    param.Direction == ParameterDirection.InputOutput ||
                    param.Direction == ParameterDirection.ReturnValue)
                {
                    Console.WriteLine($"  {param.ParameterName}: {param.Value ?? "NULL"}");
                }
            }

            stopwatch.Stop();
            Console.WriteLine($"Fin appel procédure {procedureName}, durée: {stopwatch.ElapsedMilliseconds} ms");

            return result;
        }
    }
    catch (Exception ex)
    {
        stopwatch.Stop();
        Console.WriteLine($"Erreur lors de l'appel à {procedureName}, durée: {stopwatch.ElapsedMilliseconds} ms");
        Console.WriteLine($"Exception: {ex.GetType().Name}: {ex.Message}");

        throw;
    }
}

// Exemples d'utilisation
public decimal CalculerTotalCommandes(int clientId)
{
    SqlParameter clientParam = new SqlParameter("@ClientID", clientId);

    SqlParameter totalParam = new SqlParameter("@Total", SqlDbType.Decimal)
    {
        Direction = ParameterDirection.Output,
        Precision = 10,
        Scale = 2
    };

    ExecuteProcedure<object>("CalculerTotalCommandes",
        cmd => { cmd.ExecuteNonQuery(); return null; },
        clientParam, totalParam);

    return (decimal)totalParam.Value;
}

public List<Client> RechercherClients(string nomPartiel, string ville = null)
{
    List<Client> clients = new List<Client>();

    SqlParameter nomParam = new SqlParameter("@NomPartiel", nomPartiel);
    SqlParameter villeParam = new SqlParameter("@Ville", (object)ville ?? DBNull.Value);

    ExecuteProcedure<object>("RechercherClients", cmd =>
    {
        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                clients.Add(new Client
                {
                    ClientID = (int)reader["ClientID"],
                    Nom = reader["Nom"].ToString(),
                    Prenom = reader["Prenom"].ToString(),
                    Email = reader["Email"].ToString(),
                    Telephone = reader["Telephone"]?.ToString(),
                    Ville = reader["Ville"]?.ToString()
                });
            }
        }

        return null;
    }, nomParam, villeParam);

    return clients;
}
```


### Version asynchrone (pour .NET 8)

```textmate
public async Task<T> ExecuteProcedureAsync<T>(string procedureName, Func<SqlCommand, Task<T>> executor, params SqlParameter[] parameters)
{
    Stopwatch stopwatch = Stopwatch.StartNew();
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    try
    {
        await using (SqlConnection connection = new SqlConnection(connectionString))
        {
            SqlCommand command = new SqlCommand(procedureName, connection);
            command.CommandType = CommandType.StoredProcedure;

            // Ajouter les paramètres
            if (parameters != null)
            {
                command.Parameters.AddRange(parameters);
            }

            // Journaliser l'appel (début)
            Console.WriteLine($"Début appel procédure {procedureName} à {DateTime.Now}");

            await connection.OpenAsync();

            // Exécuter la procédure
            T result = await executor(command);

            stopwatch.Stop();
            Console.WriteLine($"Fin appel procédure {procedureName}, durée: {stopwatch.ElapsedMilliseconds} ms");

            return result;
        }
    }
    catch (Exception ex)
    {
        stopwatch.Stop();
        Console.WriteLine($"Erreur lors de l'appel à {procedureName}, durée: {stopwatch.ElapsedMilliseconds} ms");
        Console.WriteLine($"Exception: {ex.GetType().Name}: {ex.Message}");

        throw;
    }
}

// Exemple d'utilisation asynchrone
public async Task<List<Client>> RechercherClientsAsync(string nomPartiel, string ville = null)
{
    List<Client> clients = new List<Client>();

    SqlParameter nomParam = new SqlParameter("@NomPartiel", nomPartiel);
    SqlParameter villeParam = new SqlParameter("@Ville", (object)ville ?? DBNull.Value);

    await ExecuteProcedureAsync<object>("RechercherClients", async cmd =>
    {
        await using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                clients.Add(new Client
                {
                    ClientID = (int)reader["ClientID"],
                    Nom = reader["Nom"].ToString(),
                    Prenom = reader["Prenom"].ToString(),
                    Email = reader["Email"].ToString(),
                    Telephone = reader["Telephone"]?.ToString(),
                    Ville = reader["Ville"]?.ToString()
                });
            }
        }

        return null;
    }, nomParam, villeParam);

    return clients;
}
```


## En résumé

Les procédures stockées offrent de nombreux avantages par rapport aux requêtes SQL directes :

1. **Performance** - Les procédures stockées sont précompilées et mises en cache, ce qui améliore les performances.

2. **Sécurité** - Les procédures stockées peuvent limiter l'accès direct aux tables et réduire les risques d'injection SQL.

3. **Encapsulation** - La logique métier complexe peut être encapsulée dans la base de données.

4. **Maintenance** - Modification de la logique métier sans changer le code de l'application.

5. **Réduction du trafic réseau** - Plusieurs opérations peuvent être effectuées en un seul aller-retour.

Pour travailler efficacement avec les procédures stockées dans ADO.NET :

- Utilisez toujours `CommandType.StoredProcedure` pour les appels de procédures stockées.
- Paramétrez correctement les types de données pour les paramètres d'entrée et de sortie.
- Utilisez les transactions pour les opérations qui impliquent plusieurs procédures.
- Gérez correctement les exceptions spécifiques aux procédures stockées.
- Testez rigoureusement les procédures stockées avec différentes entrées.
- Documentez vos procédures stockées et leurs paramètres.

En suivant ces bonnes pratiques, vous pourrez tirer pleinement parti des avantages des procédures stockées dans vos applications .NET, qu'elles soient développées avec .NET Framework 4.7.2 ou .NET 8.

⏭️
