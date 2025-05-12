# 9.4. DataSet et DataAdapter

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Dans cette section, nous explorerons le modèle déconnecté d'ADO.NET, qui utilise principalement les classes `DataSet` et `DataAdapter`. Ce modèle est particulièrement utile pour les applications qui doivent manipuler des données sans maintenir une connexion constante à la base de données.

## 9.4.1. Remplissage de DataSets

Le `DataSet` est un conteneur en mémoire qui représente un ensemble de tables et leurs relations. Le `DataAdapter` sert de pont entre votre base de données et votre `DataSet`.

### Structure de base

```textmate
// Création d'un DataSet vide
DataSet ds = new DataSet("MonDataSet");

// Configuration de la connexion
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";
SqlConnection connection = new SqlConnection(connectionString);

// Création d'un DataAdapter avec une commande SELECT
SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Clients", connection);

// Remplissage du DataSet
adapter.Fill(ds, "Clients");

// À ce stade, le DataSet contient une table "Clients" avec toutes les données
// et la connexion à la base de données est déjà fermée
```


### Remplir plusieurs tables

```textmate
// Création d'un DataSet pour stocker plusieurs tables
DataSet ds = new DataSet("MagasinDB");

string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";
SqlConnection connection = new SqlConnection(connectionString);

// Premier adaptateur pour la table Clients
SqlDataAdapter clientsAdapter = new SqlDataAdapter("SELECT * FROM Clients", connection);
clientsAdapter.Fill(ds, "Clients");

// Deuxième adaptateur pour la table Commandes
SqlDataAdapter commandesAdapter = new SqlDataAdapter("SELECT * FROM Commandes", connection);
commandesAdapter.Fill(ds, "Commandes");

// Troisième adaptateur pour la table Produits
SqlDataAdapter produitsAdapter = new SqlDataAdapter("SELECT * FROM Produits", connection);
produitsAdapter.Fill(ds, "Produits");

// À ce stade, le DataSet contient trois tables : "Clients", "Commandes" et "Produits"
Console.WriteLine($"Nombre de tables: {ds.Tables.Count}");
Console.WriteLine($"Clients: {ds.Tables["Clients"].Rows.Count} lignes");
Console.WriteLine($"Commandes: {ds.Tables["Commandes"].Rows.Count} lignes");
Console.WriteLine($"Produits: {ds.Tables["Produits"].Rows.Count} lignes");
```


### Filtrer les données lors du remplissage

```textmate
// Utilisation de paramètres dans le DataAdapter
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";
SqlConnection connection = new SqlConnection(connectionString);

// Créer une commande avec un paramètre
string query = "SELECT * FROM Commandes WHERE ClientID = @ClientID";
SqlCommand command = new SqlCommand(query, connection);
command.Parameters.AddWithValue("@ClientID", 5); // Filtrer pour le client #5

// Créer l'adaptateur avec cette commande
SqlDataAdapter adapter = new SqlDataAdapter(command);

// Remplir le DataSet
DataSet ds = new DataSet();
adapter.Fill(ds, "CommandesClient");

Console.WriteLine($"Commandes pour le client #5: {ds.Tables["CommandesClient"].Rows.Count}");
```


### Configuration du DataAdapter

Le `DataAdapter` nécessite des commandes SQL pour les opérations CRUD (Create, Read, Update, Delete). Vous pouvez les configurer manuellement ou utiliser un `CommandBuilder`.

#### Utilisation du CommandBuilder (approche simplifiée)

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

// Créer l'adaptateur avec une commande SELECT de base
SqlDataAdapter adapter = new SqlDataAdapter(
    "SELECT * FROM Produits",
    connectionString
);

// Le CommandBuilder génère automatiquement les commandes INSERT, UPDATE et DELETE
SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

// Maintenant l'adaptateur dispose des commandes nécessaires pour les mises à jour
DataSet ds = new DataSet();
adapter.Fill(ds, "Produits");

// Modifier les données (nous verrons cela dans la section suivante)
// ...

// Mettre à jour la base de données
adapter.Update(ds, "Produits");
```


#### Configuration manuelle des commandes (approche avancée)

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";
SqlConnection connection = new SqlConnection(connectionString);

// Créer le DataAdapter avec la commande SELECT
SqlDataAdapter adapter = new SqlDataAdapter();
adapter.SelectCommand = new SqlCommand("SELECT ID, Nom, Prix, Stock FROM Produits", connection);

// Configurer manuellement la commande INSERT
adapter.InsertCommand = new SqlCommand(
    "INSERT INTO Produits (Nom, Prix, Stock) VALUES (@Nom, @Prix, @Stock); SELECT SCOPE_IDENTITY()",
    connection
);
adapter.InsertCommand.Parameters.Add("@Nom", SqlDbType.NVarChar, 50, "Nom");
adapter.InsertCommand.Parameters.Add("@Prix", SqlDbType.Decimal, 0, "Prix");
adapter.InsertCommand.Parameters.Add("@Stock", SqlDbType.Int, 0, "Stock");
adapter.InsertCommand.UpdatedRowSource = UpdateRowSource.FirstReturnedRecord;

// Configurer manuellement la commande UPDATE
adapter.UpdateCommand = new SqlCommand(
    "UPDATE Produits SET Nom = @Nom, Prix = @Prix, Stock = @Stock WHERE ID = @ID",
    connection
);
adapter.UpdateCommand.Parameters.Add("@Nom", SqlDbType.NVarChar, 50, "Nom");
adapter.UpdateCommand.Parameters.Add("@Prix", SqlDbType.Decimal, 0, "Prix");
adapter.UpdateCommand.Parameters.Add("@Stock", SqlDbType.Int, 0, "Stock");
adapter.UpdateCommand.Parameters.Add("@ID", SqlDbType.Int, 0, "ID");

// Configurer manuellement la commande DELETE
adapter.DeleteCommand = new SqlCommand(
    "DELETE FROM Produits WHERE ID = @ID",
    connection
);
adapter.DeleteCommand.Parameters.Add("@ID", SqlDbType.Int, 0, "ID");

// Remplir le DataSet
DataSet ds = new DataSet();
adapter.Fill(ds, "Produits");
```


### Différences entre .NET Framework 4.7.2 et .NET 8

Les deux plateformes fonctionnent de manière similaire, mais il y a quelques différences à noter :

#### .NET Framework 4.7.2

```textmate
// Utilisation typique dans .NET Framework 4.7.2
using System.Data;
using System.Data.SqlClient; // Namespace pour SQL Server dans .NET Framework

string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Produits", connectionString);
SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

DataSet ds = new DataSet();
adapter.Fill(ds, "Produits");
```


#### .NET 8

```textmate
// Utilisation typique dans .NET 8
using System.Data;
using Microsoft.Data.SqlClient; // Nouveau namespace pour SQL Server dans .NET 8

string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Produits", connectionString);
SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

DataSet ds = new DataSet();
adapter.Fill(ds, "Produits");
```


> **Note pour .NET 8** : Vous devez installer le package NuGet `Microsoft.Data.SqlClient` et dans certains cas `System.Data.SqlClient` pour une compatibilité complète.

## 9.4.2. Manipulation des données

Une fois les données chargées dans un DataSet, vous pouvez les manipuler en mémoire sans avoir besoin de vous connecter à la base de données.

### Accéder aux données

```textmate
// Accéder à la table Produits
DataTable produitsTable = ds.Tables["Produits"];

// Parcourir toutes les lignes
foreach (DataRow row in produitsTable.Rows)
{
    int id = (int)row["ID"];
    string nom = row["Nom"].ToString();
    decimal prix = (decimal)row["Prix"];

    Console.WriteLine($"Produit #{id}: {nom}, Prix: {prix:C}");
}

// Accéder à une ligne spécifique (première ligne)
if (produitsTable.Rows.Count > 0)
{
    DataRow premierProduit = produitsTable.Rows[0];
    Console.WriteLine($"Premier produit: {premierProduit["Nom"]}");
}

// Accéder à la valeur d'une cellule spécifique
if (produitsTable.Rows.Count > 0)
{
    string nomPremierProduit = produitsTable.Rows[0]["Nom"].ToString();
    Console.WriteLine($"Nom du premier produit: {nomPremierProduit}");
}
```


### Filtrer et rechercher des données

```textmate
// Filtrer les données avec une expression
DataRow[] produitsChers = produitsTable.Select("Prix > 100");
Console.WriteLine($"Produits dont le prix est supérieur à 100€: {produitsChers.Length}");

foreach (DataRow row in produitsChers)
{
    Console.WriteLine($"{row["Nom"]} - {((decimal)row["Prix"]):C}");
}

// Rechercher par expression complexe
DataRow[] recherche = produitsTable.Select("Nom LIKE '%Apple%' AND (Categorie = 'Téléphones' OR Categorie = 'Tablettes')");

// Trier les résultats
DataRow[] produitsTries = produitsTable.Select("", "Prix DESC");
Console.WriteLine("Produits triés par prix décroissant:");
foreach (DataRow row in produitsTries)
{
    Console.WriteLine($"{row["Nom"]} - {((decimal)row["Prix"]):C}");
}
```


### Ajouter des données

```textmate
// Créer une nouvelle ligne
DataRow nouveauProduit = produitsTable.NewRow();
nouveauProduit["Nom"] = "Nouveau Smartphone";
nouveauProduit["Prix"] = 499.99m;
nouveauProduit["Stock"] = 25;

// Ajouter la ligne à la table
produitsTable.Rows.Add(nouveauProduit);

Console.WriteLine($"Nombre total de produits après ajout: {produitsTable.Rows.Count}");
```


### Modifier des données

```textmate
// Modifier une ligne existante
if (produitsTable.Rows.Count > 0)
{
    // Modifier la première ligne
    DataRow premiereLigne = produitsTable.Rows[0];
    premiereLigne["Prix"] = (decimal)premiereLigne["Prix"] * 1.1m; // Augmentation de 10%
    premiereLigne["Stock"] = (int)premiereLigne["Stock"] + 10; // Ajouter du stock

    Console.WriteLine($"Produit modifié: {premiereLigne["Nom"]}, Nouveau prix: {((decimal)premiereLigne["Prix"]):C}");
}

// Modifier plusieurs lignes (par exemple, augmenter le prix de tous les produits de 5%)
foreach (DataRow row in produitsTable.Rows)
{
    row["Prix"] = Math.Round((decimal)row["Prix"] * 1.05m, 2);
}
```


### Supprimer des données

```textmate
// Supprimer la première ligne
if (produitsTable.Rows.Count > 0)
{
    produitsTable.Rows[0].Delete();
    Console.WriteLine("Premier produit supprimé");
}

// Supprimer des lignes selon une condition
DataRow[] produitsObsoletes = produitsTable.Select("Stock = 0");
foreach (DataRow row in produitsObsoletes)
{
    row.Delete();
}
Console.WriteLine($"{produitsObsoletes.Length} produits en rupture de stock supprimés");

// Accepter les suppressions (supprime définitivement les lignes marquées pour suppression)
// Note: Les lignes restent dans la collection Rows jusqu'à ce que vous appeliez AcceptChanges()
produitsTable.AcceptChanges();
```


### Suivre les modifications

Le DataSet suit l'état de chaque ligne (ajoutée, modifiée, supprimée ou inchangée).

```textmate
// Vérifier l'état des lignes
foreach (DataRow row in produitsTable.Rows)
{
    string etat = "Inchangée";

    switch (row.RowState)
    {
        case DataRowState.Added:
            etat = "Ajoutée";
            break;
        case DataRowState.Modified:
            etat = "Modifiée";
            break;
        case DataRowState.Deleted:
            etat = "Supprimée";
            break;
    }

    if (row.RowState != DataRowState.Deleted)
    {
        Console.WriteLine($"Produit: {row["Nom"]}, État: {etat}");
    }
    else
    {
        Console.WriteLine($"Un produit a été supprimé, État: {etat}");
    }
}
```


### Gérer les erreurs de validation

```textmate
// Ajouter un gestionnaire d'événements pour la validation des colonnes
produitsTable.ColumnChanging += (sender, e) => {
    if (e.Column.ColumnName == "Prix" && e.ProposedValue != null)
    {
        // Valider que le prix est positif
        decimal prixPropose = (decimal)e.ProposedValue;
        if (prixPropose < 0)
        {
            e.ProposedValue = 0; // Corriger la valeur
            Console.WriteLine("Prix négatif corrigé à 0");
        }
    }
};

// Définir des contraintes sur la table
// La colonne Nom ne peut pas être vide
produitsTable.Columns["Nom"].AllowDBNull = false;

// La colonne Prix doit être positive
DataColumn prixColumn = produitsTable.Columns["Prix"];
prixColumn.Expression = "Prix >= 0";

try
{
    // Tentative d'ajouter un produit avec un nom vide
    DataRow produitInvalide = produitsTable.NewRow();
    produitInvalide["Nom"] = DBNull.Value; // Cela devrait échouer
    produitInvalide["Prix"] = 19.99m;
    produitsTable.Rows.Add(produitInvalide);
}
catch (ConstraintException ex)
{
    Console.WriteLine($"Erreur de validation: {ex.Message}");
}
```


## 9.4.3. Mise à jour de la base de données

Une fois que vous avez modifié les données dans votre DataSet, vous pouvez les synchroniser avec la base de données.

### Mise à jour de base

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

// Créer le DataAdapter
SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Produits", connectionString);

// Configurer le CommandBuilder pour générer les commandes
SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

// Remplir le DataSet
DataSet ds = new DataSet();
adapter.Fill(ds, "Produits");

// Obtenir la référence à la table
DataTable produitsTable = ds.Tables["Produits"];

// Modifier des données
DataRow row = produitsTable.Rows[0];
row["Prix"] = (decimal)row["Prix"] * 1.10m; // Augmenter le prix de 10%

// Ajouter un nouveau produit
DataRow newRow = produitsTable.NewRow();
newRow["Nom"] = "Produit Test";
newRow["Prix"] = 99.99m;
newRow["Stock"] = 50;
produitsTable.Rows.Add(newRow);

// Supprimer un produit
produitsTable.Rows[1].Delete();

// Envoyer les modifications à la base de données
int rowsAffected = adapter.Update(ds, "Produits");

Console.WriteLine($"{rowsAffected} ligne(s) mise(s) à jour dans la base de données");
```


### Mise à jour avec gestion d'erreurs

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Créer le DataAdapter
    SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Produits", connection);
    SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

    // Remplir le DataSet
    DataSet ds = new DataSet();
    adapter.Fill(ds, "Produits");

    DataTable produitsTable = ds.Tables["Produits"];

    try
    {
        // Modifier des données
        if (produitsTable.Rows.Count > 0)
        {
            DataRow row = produitsTable.Rows[0];
            row["Nom"] = "Produit Modifié";
            row["Prix"] = 199.99m;
        }

        // Ajouter un nouveau produit
        DataRow newRow = produitsTable.NewRow();
        newRow["Nom"] = "Nouveau Produit";
        newRow["Prix"] = 299.99m;
        newRow["Stock"] = 10;
        produitsTable.Rows.Add(newRow);

        // Mettre à jour la base de données
        int rowsAffected = adapter.Update(ds, "Produits");
        Console.WriteLine($"{rowsAffected} ligne(s) mise(s) à jour dans la base de données");

        // Accepter les modifications localement
        produitsTable.AcceptChanges();
    }
    catch (DBConcurrencyException ex)
    {
        Console.WriteLine($"Erreur de concurrence: {ex.Message}");
        Console.WriteLine("Un autre utilisateur a modifié les données. Veuillez rafraîchir vos données.");

        // Rejeter les modifications locales
        produitsTable.RejectChanges();
    }
    catch (SqlException ex)
    {
        Console.WriteLine($"Erreur SQL: {ex.Message}");

        // Rejeter les modifications locales
        produitsTable.RejectChanges();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erreur: {ex.Message}");

        // Rejeter les modifications locales
        produitsTable.RejectChanges();
    }
}
```


### Mise à jour avec transactions

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Démarrer une transaction
    using (SqlTransaction transaction = connection.BeginTransaction())
    {
        try
        {
            // Créer le DataAdapter et associer la transaction
            SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Produits", connection);
            adapter.SelectCommand.Transaction = transaction;

            // Configurer le CommandBuilder
            SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

            // S'assurer que toutes les commandes utilisent la transaction
            builder.GetInsertCommand().Transaction = transaction;
            builder.GetUpdateCommand().Transaction = transaction;
            builder.GetDeleteCommand().Transaction = transaction;

            // Remplir le DataSet
            DataSet ds = new DataSet();
            adapter.Fill(ds, "Produits");

            DataTable produitsTable = ds.Tables["Produits"];

            // Effectuer plusieurs modifications

            // 1. Modifier un produit existant
            if (produitsTable.Rows.Count > 0)
            {
                produitsTable.Rows[0]["Prix"] = (decimal)produitsTable.Rows[0]["Prix"] * 1.15m;
            }

            // 2. Ajouter un nouveau produit
            DataRow newRow = produitsTable.NewRow();
            newRow["Nom"] = "Produit Transaction";
            newRow["Prix"] = 149.99m;
            newRow["Stock"] = 25;
            produitsTable.Rows.Add(newRow);

            // 3. Supprimer le dernier produit
            if (produitsTable.Rows.Count > 1)
            {
                produitsTable.Rows[produitsTable.Rows.Count - 1].Delete();
            }

            // Mettre à jour la base de données
            int rowsAffected = adapter.Update(ds, "Produits");

            // Si tout s'est bien passé, valider la transaction
            transaction.Commit();

            Console.WriteLine($"{rowsAffected} ligne(s) mise(s) à jour dans la base de données");
            produitsTable.AcceptChanges();
        }
        catch (Exception ex)
        {
            // En cas d'erreur, annuler la transaction
            transaction.Rollback();
            Console.WriteLine($"Erreur: {ex.Message}");
            Console.WriteLine("Transaction annulée, aucune modification n'a été effectuée");
        }
    }
}
```


### Mise à jour avec gestion des conflits

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    // Créer un DataAdapter
    SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Produits", connection);
    SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

    // Configurer la détection des conflits
    adapter.MissingSchemaAction = MissingSchemaAction.AddWithKey;

    // Remplir le DataSet
    DataSet ds = new DataSet();
    adapter.Fill(ds, "Produits");

    DataTable produitsTable = ds.Tables["Produits"];

    // Simuler une modification concurrente dans la base de données
    SimulerModificationConcurrente(connectionString, 1); // Modifie le produit avec ID = 1

    // Maintenant, essayons de modifier le même produit dans notre DataSet
    DataRow[] rows = produitsTable.Select("ID = 1");
    if (rows.Length > 0)
    {
        DataRow row = rows[0];
        row["Prix"] = 199.99m;

        try
        {
            // Essayer de mettre à jour
            adapter.Update(ds, "Produits");
            Console.WriteLine("Mise à jour réussie");
        }
        catch (DBConcurrencyException ex)
        {
            Console.WriteLine($"Conflit détecté: {ex.Message}");

            // Gérer le conflit
            // 1. Récupérer les données actuelles de la base de données
            DataSet dsActuel = new DataSet();
            adapter.Fill(dsActuel, "Produits");
            DataRow rowActuelle = dsActuel.Tables["Produits"].Select("ID = 1")[0];

            // 2. Afficher les différences
            Console.WriteLine("Valeurs en conflit:");
            Console.WriteLine($"Votre version - Prix: {row["Prix"]}");
            Console.WriteLine($"Version de la base de données - Prix: {rowActuelle["Prix"]}");

            // 3. Options pour résoudre le conflit
            Console.WriteLine("\nOptions de résolution:");
            Console.WriteLine("1. Garder vos modifications");
            Console.WriteLine("2. Utiliser les valeurs de la base de données");
            Console.WriteLine("3. Fusionner les modifications");

            // Ici, nous choisissons l'option 3 dans cet exemple
            Console.WriteLine("Choix: 3");

            // 4. Mettre à jour la ligne avec la fusion des données
            row.BeginEdit();
            // Garder notre prix mais prendre les autres champs de la base de données
            foreach (DataColumn col in produitsTable.Columns)
            {
                if (col.ColumnName != "Prix" && col.ColumnName != "ID")
                {
                    row[col] = rowActuelle[col];
                }
            }
            row.EndEdit();

            // 5. Réessayer la mise à jour
            try
            {
                adapter.Update(ds, "Produits");
                Console.WriteLine("Conflit résolu et mise à jour réussie");
            }
            catch (Exception ex2)
            {
                Console.WriteLine($"Échec de la résolution: {ex2.Message}");
            }
        }
    }
}

// Méthode pour simuler une modification concurrente
static void SimulerModificationConcurrente(string connectionString, int productId)
{
    using (SqlConnection conn = new SqlConnection(connectionString))
    {
        conn.Open();
        using (SqlCommand cmd = new SqlCommand(
            "UPDATE Produits SET Nom = 'Modifié par un autre utilisateur', Prix = 249.99 WHERE ID = @ID",
            conn))
        {
            cmd.Parameters.AddWithValue("@ID", productId);
            cmd.ExecuteNonQuery();
        }
    }

    Console.WriteLine($"Simulation: Le produit {productId} a été modifié en arrière-plan");
}
```


## 9.4.4. DataTable, DataRow et DataColumn

Ces classes sont les briques de base d'un DataSet et permettent de manipuler les données de manière fine.

### DataTable

La classe `DataTable` représente une table en mémoire.

```textmate
// Création d'une table de toutes pièces (sans base de données)
DataTable clientsTable = new DataTable("Clients");

// Définir les colonnes
clientsTable.Columns.Add("ID", typeof(int));
clientsTable.Columns.Add("Nom", typeof(string));
clientsTable.Columns.Add("Email", typeof(string));
clientsTable.Columns.Add("DateInscription", typeof(DateTime));
clientsTable.Columns.Add("Solde", typeof(decimal));

// Définir la clé primaire
clientsTable.PrimaryKey = new DataColumn[] { clientsTable.Columns["ID"] };

// Définir des valeurs par défaut
clientsTable.Columns["DateInscription"].DefaultValue = DateTime.Now;
clientsTable.Columns["Solde"].DefaultValue = 0m;

// Ajouter des contraintes
clientsTable.Columns["ID"].AutoIncrement = true;
clientsTable.Columns["ID"].AutoIncrementSeed = 1;
clientsTable.Columns["ID"].AutoIncrementStep = 1;
clientsTable.Columns["Nom"].AllowDBNull = false;
clientsTable.Columns["Email"].Unique = true;

// Ajouter des lignes
DataRow row1 = clientsTable.NewRow();
row1["Nom"] = "Jean Dupont";
row1["Email"] = "jean.dupont@exemple.fr";
row1["Solde"] = 150.50m;
clientsTable.Rows.Add(row1);

DataRow row2 = clientsTable.NewRow();
row2["Nom"] = "Marie Martin";
row2["Email"] = "marie.martin@exemple.fr";
row2["Solde"] = 275.25m;
clientsTable.Rows.Add(row2);

// Parcourir la table
foreach (DataRow row in clientsTable.Rows)
{
    Console.WriteLine($"Client: {row["Nom"]}, Email: {row["Email"]}, Inscrit le: {((DateTime)row["DateInscription"]).ToShortDateString()}, Solde: {((decimal)row["Solde"]):C}");
}
```


### Propriétés et méthodes utiles de DataTable

```textmate
// Vérifier si une table contient des lignes
bool tableVide = clientsTable.Rows.Count == 0;

// Vérifier si une table a des modifications en attente
bool modifications = clientsTable.GetChanges() != null;

// Obtenir uniquement les lignes modifiées
DataTable modifications = clientsTable.GetChanges(DataRowState.Modified);

// Obtenir une vue filtrée de la table
DataView clientsRichesView = new DataView(clientsTable, "Solde > 200", "Nom ASC", DataViewRowState.CurrentRows);
foreach (DataRowView rowView in clientsRichesView)
{
    Console.WriteLine($"Client riche: {rowView["Nom"]}, Solde: {((decimal)rowView["Solde"]):C}");
}

// Vider une table (supprimer toutes les lignes)
clientsTable.Clear();

// Copier la structure (sans les données)
DataTable copieStructure = clientsTable.Clone();

// Copier la structure et les données
DataTable copieComplete = clientsTable.Copy();

// Fusionner des tables
DataTable autresClients = ObtienAutresClients(); // Supposons que cette fonction existe
clientsTable.Merge(autresClients);

// Charger à partir d'un fichier XML
clientsTable.ReadXml("clients.xml");

// Sauvegarder en XML
clientsTable.WriteXml("clients_sauvegardes.xml", XmlWriteMode.WriteSchema);
```


### DataRow

La classe `DataRow` représente une ligne dans une `DataTable`.

```textmate
// Créer une ligne pour une table existante
DataRow nouvelleLigne = clientsTable.NewRow();

// Définir des valeurs
nouvelleLigne["Nom"] = "Pierre Durand";
nouvelleLigne["Email"] = "pierre.durand@exemple.fr";
nouvelleLigne["Solde"] = 320.75m;

// Ajouter la ligne à la table
clientsTable.Rows.Add(nouvelleLigne);

// Accéder aux valeurs de différentes façons
string nom = nouvelleLigne["Nom"].ToString();
decimal solde = (decimal)nouvelleLigne["Solde"];

// Vérifier si une valeur est NULL
if (nouvelleLigne.IsNull("Email"))
{
    Console.WriteLine("L'email n'est pas défini.");
}

// Modifier une ligne avec suivi des modifications
DataRow ligne = clientsTable.Rows[0];
ligne.BeginEdit();
ligne["Nom"] = "Jean Martin"; // Nouveau nom
ligne["Solde"] = (decimal)ligne["Solde"] + 100.00m; // Augmenter le solde
ligne.EndEdit();

// Annuler les modifications (si pas encore acceptées)
DataRow ligneModifiee = clientsTable.Rows[1];
ligneModifiee.BeginEdit();
ligneModifiee["Solde"] = 0; // Changement drastique
// Opps, mauvaise idée, annulons ça
ligneModifiee.CancelEdit();

// Vérifier l'état d'une ligne
if (ligne.RowState == DataRowState.Modified)
{
    Console.WriteLine($"La ligne pour {ligne["Nom"]} a été modifiée");

    // Accéder aux valeurs originales (avant modification)
    string ancienNom = ligne["Nom", DataRowVersion.Original].ToString();
    decimal ancienSolde = (decimal)ligne["Solde", DataRowVersion.Original];

    Console.WriteLine($"Valeurs originales - Nom: {ancienNom}, Solde: {ancienSolde:C}");
    Console.WriteLine($"Nouvelles valeurs - Nom: {ligne["Nom"]}, Solde: {(decimal)ligne["Solde"]:C}");
}

// Marquer une ligne comme supprimée
clientsTable.Rows[2].Delete();

// Vérifier si une ligne est supprimée
if (clientsTable.Rows[2].RowState == DataRowState.Deleted)
{
    Console.WriteLine("La ligne a été marquée pour suppression");
}
```


### DataColumn

La classe `DataColumn` définit une colonne dans une `DataTable`.

```textmate
// Création d'une table
DataTable employesTable = new DataTable("Employes");

// Ajouter des colonnes - méthode simple
employesTable.Columns.Add("ID", typeof(int));
employesTable.Columns.Add("Nom", typeof(string));

// Ajouter une colonne - méthode détaillée
DataColumn colSalaire = new DataColumn();
colSalaire.ColumnName = "Salaire";
colSalaire.DataType = typeof(decimal);
colSalaire.DefaultValue = 0m;
colSalaire.AllowDBNull = false;
colSalaire.Caption = "Salaire mensuel"; // Titre convivial pour l'affichage
employesTable.Columns.Add(colSalaire);

// Ajouter une colonne calculée
DataColumn colBonus = new DataColumn();
colBonus.ColumnName = "Bonus";
colBonus.DataType = typeof(decimal);
colBonus.Expression = "Salaire * 0.1"; // 10% du salaire
employesTable.Columns.Add(colBonus);

// Ajouter une colonne avec auto-incrémentation
DataColumn colID = employesTable.Columns["ID"];
colID.AutoIncrement = true;
colID.AutoIncrementSeed = 1; // Valeur de départ
colID.AutoIncrementStep = 1; // Incrément

// Définir une clé primaire
employesTable.PrimaryKey = new DataColumn[] { colID };

// Ajouter des contraintes de validation
// Par exemple, le salaire doit être positif
colSalaire.Expression = "Salaire >= 0";

// Ajouter une contrainte unique
UniqueConstraint contrainteNomUnique = new UniqueConstraint("NomUnique", employesTable.Columns["Nom"]);
employesTable.Constraints.Add(contrainteNomUnique);

// Ajouter quelques données
DataRow emp1 = employesTable.NewRow();
emp1["Nom"] = "Alice Dupont";
emp1["Salaire"] = 2500m;
employesTable.Rows.Add(emp1);

DataRow emp2 = employesTable.NewRow();
emp2["Nom"] = "Bob Martin";
emp2["Salaire"] = 3200m;
employesTable.Rows.Add(emp2);

// Afficher les données avec les colonnes calculées
foreach (DataRow row in employesTable.Rows)
{
    Console.WriteLine($"ID: {row["ID"]}, Nom: {row["Nom"]}, Salaire: {(decimal)row["Salaire"]:C}, Bonus: {(decimal)row["Bonus"]:C}");
}

// Vérifier l'ordinal d'une colonne (sa position)
int ordinalNom = employesTable.Columns["Nom"].Ordinal;
Console.WriteLine($"La colonne 'Nom' est à la position {ordinalNom}");

// Vérifier si une colonne existe
bool exists = employesTable.Columns.Contains("DateEmbauche");
if (!exists)
{
    // Ajouter la colonne manquante
    employesTable.Columns.Add("DateEmbauche", typeof(DateTime));
}
```


### Relations entre tables

Les `DataRelation` permettent de créer des liens entre les tables, similaires aux clés étrangères dans une base de données.

```textmate
// Créer un DataSet pour contenir nos tables liées
DataSet entrepriseDS = new DataSet("Entreprise");

// Créer la table des départements
DataTable departementsTable = new DataTable("Departements");
departementsTable.Columns.Add("DeptID", typeof(int));
departementsTable.Columns.Add("NomDept", typeof(string));
departementsTable.PrimaryKey = new DataColumn[] { departementsTable.Columns["DeptID"] };

// Créer la table des employés
DataTable employesTable = new DataTable("Employes");
employesTable.Columns.Add("EmpID", typeof(int));
employesTable.Columns.Add("Nom", typeof(string));
employesTable.Columns.Add("DeptID", typeof(int)); // Clé étrangère
employesTable.PrimaryKey = new DataColumn[] { employesTable.Columns["EmpID"] };

// Ajouter les tables au DataSet
entrepriseDS.Tables.Add(departementsTable);
entrepriseDS.Tables.Add(employesTable);

// Créer une relation entre les tables
DataRelation relation = new DataRelation(
    "DeptEmployes", // Nom de la relation
    departementsTable.Columns["DeptID"], // Colonne parente (table principale)
    employesTable.Columns["DeptID"]      // Colonne enfant (table liée)
);

// Ajouter la relation au DataSet
entrepriseDS.Relations.Add(relation);

// Ajouter des données aux tables
DataRow dept1 = departementsTable.NewRow();
dept1["DeptID"] = 1;
dept1["NomDept"] = "Informatique";
departementsTable.Rows.Add(dept1);

DataRow dept2 = departementsTable.NewRow();
dept2["DeptID"] = 2;
dept2["NomDept"] = "Comptabilité";
departementsTable.Rows.Add(dept2);

DataRow emp1 = employesTable.NewRow();
emp1["EmpID"] = 101;
emp1["Nom"] = "Alice Dupont";
emp1["DeptID"] = 1; // Appartient au département Informatique
employesTable.Rows.Add(emp1);

DataRow emp2 = employesTable.NewRow();
emp2["EmpID"] = 102;
emp2["Nom"] = "Bob Martin";
emp2["DeptID"] = 1; // Appartient au département Informatique
employesTable.Rows.Add(emp2);

DataRow emp3 = employesTable.NewRow();
emp3["EmpID"] = 103;
emp3["Nom"] = "Charlie Durand";
emp3["DeptID"] = 2; // Appartient au département Comptabilité
employesTable.Rows.Add(emp3);

// Navigation entre tables liées
// Exemple: Lister tous les employés du département Informatique
DataRow deptInfo = departementsTable.Rows.Find(1); // Trouver le département avec ID = 1
if (deptInfo != null)
{
    Console.WriteLine($"Employés du département {deptInfo["NomDept"]}:");

    // Obtenir les lignes enfants (employés liés)
    DataRow[] employesDept = deptInfo.GetChildRows(relation);

    foreach (DataRow empRow in employesDept)
    {
        Console.WriteLine($"  - {empRow["Nom"]} (ID: {empRow["EmpID"]})");
    }
}

// Dans l'autre sens: Trouver le département d'un employé
DataRow employe = employesTable.Rows.Find(103); // Trouver l'employé avec ID = 103
if (employe != null)
{
    // Obtenir la ligne parente (département)
    DataRow deptParent = employe.GetParentRow(relation);

    Console.WriteLine($"L'employé {employe["Nom"]} travaille dans le département {deptParent["NomDept"]}");
}
```


### Contraintes de clé étrangère

Pour renforcer l'intégrité référentielle, vous pouvez ajouter des contraintes de clé étrangère :

```textmate
// Créer des tables avec contrainte de clé étrangère
DataTable clientsTable = new DataTable("Clients");
clientsTable.Columns.Add("ClientID", typeof(int));
clientsTable.Columns.Add("Nom", typeof(string));
clientsTable.PrimaryKey = new DataColumn[] { clientsTable.Columns["ClientID"] };

DataTable commandesTable = new DataTable("Commandes");
commandesTable.Columns.Add("CommandeID", typeof(int));
commandesTable.Columns.Add("ClientID", typeof(int));
commandesTable.Columns.Add("DateCommande", typeof(DateTime));
commandesTable.Columns.Add("Montant", typeof(decimal));
commandesTable.PrimaryKey = new DataColumn[] { commandesTable.Columns["CommandeID"] };

// Créer un DataSet
DataSet magasinDS = new DataSet("Magasin");
magasinDS.Tables.Add(clientsTable);
magasinDS.Tables.Add(commandesTable);

// Ajouter une contrainte de clé étrangère
ForeignKeyConstraint fk = new ForeignKeyConstraint(
    "FK_Commandes_Clients",
    clientsTable.Columns["ClientID"],   // Colonne de référence
    commandesTable.Columns["ClientID"]  // Colonne qui fait référence
);

// Configurer le comportement en cas de modification/suppression
fk.DeleteRule = Rule.Cascade;   // Si un client est supprimé, supprimer ses commandes
fk.UpdateRule = Rule.Cascade;   // Si l'ID d'un client change, mettre à jour ses commandes
fk.AcceptRejectRule = AcceptRejectRule.Cascade;

// Ajouter la contrainte
commandesTable.Constraints.Add(fk);

// Ajouter des données
DataRow client1 = clientsTable.NewRow();
client1["ClientID"] = 1;
client1["Nom"] = "Dupont SA";
clientsTable.Rows.Add(client1);

DataRow commande1 = commandesTable.NewRow();
commande1["CommandeID"] = 1001;
commande1["ClientID"] = 1;
commande1["DateCommande"] = DateTime.Now;
commande1["Montant"] = 1250.75m;
commandesTable.Rows.Add(commande1);

// Essayer d'ajouter une commande avec un ClientID inexistant
try
{
    DataRow commandeInvalide = commandesTable.NewRow();
    commandeInvalide["CommandeID"] = 1002;
    commandeInvalide["ClientID"] = 999; // Ce client n'existe pas
    commandeInvalide["DateCommande"] = DateTime.Now;
    commandeInvalide["Montant"] = 500.25m;
    commandesTable.Rows.Add(commandeInvalide);
}
catch (ConstraintException ex)
{
    Console.WriteLine($"Erreur de contrainte: {ex.Message}");
}

// Démontrer la suppression en cascade
Console.WriteLine($"Nombre de commandes avant suppression: {commandesTable.Rows.Count}");
clientsTable.Rows[0].Delete(); // Supprimer le client #1
clientsTable.AcceptChanges();
commandesTable.AcceptChanges();
Console.WriteLine($"Nombre de commandes après suppression: {commandesTable.Rows.Count}");
```


### Sérialisation et désérialisation

Les DataSets peuvent être facilement sérialisés en XML et désérialisés :

```textmate
// Supposons que nous avons déjà un DataSet rempli
DataSet ds = RemplirDataSet(); // Fonction hypothétique

// Sauvegarder le DataSet en XML avec son schéma
ds.WriteXml("donnees.xml", XmlWriteMode.WriteSchema);
Console.WriteLine("DataSet sauvegardé en XML");

// Charger un DataSet à partir d'un fichier XML
DataSet dsCharge = new DataSet();
dsCharge.ReadXml("donnees.xml");
Console.WriteLine($"DataSet chargé avec {dsCharge.Tables.Count} tables");

// Vous pouvez aussi manipuler la représentation XML en mémoire
StringBuilder sb = new StringBuilder();
using (StringWriter sw = new StringWriter(sb))
{
    ds.WriteXml(sw, XmlWriteMode.WriteSchema);
}
string xmlContenu = sb.ToString();
Console.WriteLine($"Taille du XML: {xmlContenu.Length} caractères");

// Recréer un DataSet à partir d'une chaîne XML
DataSet dsFromString = new DataSet();
using (StringReader sr = new StringReader(xmlContenu))
{
    dsFromString.ReadXml(sr);
}
```


### Utilisation avec les interfaces utilisateur

Les DataSets et DataTables sont parfaitement adaptés pour alimenter des contrôles d'interface utilisateur.

#### Windows Forms (.NET Framework 4.7.2)

```textmate
// Dans un formulaire Windows Forms
public partial class FormClients : Form
{
    private DataSet ds;
    private SqlDataAdapter adapter;
    private string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    public FormClients()
    {
        InitializeComponent();

        // Charger les données au démarrage
        ChargerDonnees();
    }

    private void ChargerDonnees()
    {
        try
        {
            ds = new DataSet();
            adapter = new SqlDataAdapter("SELECT * FROM Clients", connectionString);
            SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

            adapter.Fill(ds, "Clients");

            // Lier le DataGridView au DataTable
            dataGridViewClients.DataSource = ds.Tables["Clients"];

            // Vous pouvez aussi lier des contrôles individuels
            textBoxNom.DataBindings.Add("Text", ds.Tables["Clients"], "Nom");
            textBoxEmail.DataBindings.Add("Text", ds.Tables["Clients"], "Email");
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur lors du chargement des données: {ex.Message}");
        }
    }

    private void btnEnregistrer_Click(object sender, EventArgs e)
    {
        try
        {
            // Enregistrer les modifications
            adapter.Update(ds, "Clients");
            MessageBox.Show("Modifications enregistrées avec succès");
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur lors de l'enregistrement: {ex.Message}");
        }
    }
}
```


#### WPF (.NET Framework 4.7.2 ou .NET 8)

```textmate
// Dans un ViewModel (pattern MVVM)
public class ClientsViewModel : INotifyPropertyChanged
{
    private DataSet ds;
    private SqlDataAdapter adapter;
    private string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    private DataView _clientsView;
    public DataView ClientsView
    {
        get { return _clientsView; }
        set
        {
            _clientsView = value;
            OnPropertyChanged("ClientsView");
        }
    }

    public ClientsViewModel()
    {
        ChargerDonnees();
    }

    private void ChargerDonnees()
    {
        try
        {
            ds = new DataSet();
            adapter = new SqlDataAdapter("SELECT * FROM Clients", connectionString);
            SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

            adapter.Fill(ds, "Clients");

            // Créer une vue pour le binding
            ClientsView = ds.Tables["Clients"].DefaultView;
        }
        catch (Exception ex)
        {
            // Gérer l'erreur
        }
    }

    public void Enregistrer()
    {
        try
        {
            // Enregistrer les modifications
            adapter.Update(ds, "Clients");
        }
        catch (Exception ex)
        {
            // Gérer l'erreur
        }
    }

    // Implémentation de INotifyPropertyChanged
    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


XAML associé :

```xml
<Window x:Class="MonApplication.ClientsWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Gestion des clients" Height="450" Width="800">
    <Grid>
        <DataGrid ItemsSource="{Binding ClientsView}" AutoGenerateColumns="True" Margin="10" />
        <Button Content="Enregistrer" Command="{Binding EnregistrerCommand}" Margin="10" VerticalAlignment="Bottom" HorizontalAlignment="Right" />
    </Grid>
</Window>
```


#### ASP.NET Web Forms (.NET Framework 4.7.2)

```textmate
// Dans une page ASP.NET Web Forms
protected void Page_Load(object sender, EventArgs e)
{
    if (!IsPostBack)
    {
        ChargerDonnees();
    }
}

private void ChargerDonnees()
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    try
    {
        SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Clients", connectionString);
        DataSet ds = new DataSet();
        adapter.Fill(ds, "Clients");

        // Lier aux contrôles
        GridViewClients.DataSource = ds.Tables["Clients"];
        GridViewClients.DataBind();

        // Stocker le DataSet en session pour une utilisation ultérieure
        Session["ClientsDataSet"] = ds;
    }
    catch (Exception ex)
    {
        LabelErreur.Text = $"Erreur: {ex.Message}";
    }
}

protected void ButtonEnregistrer_Click(object sender, EventArgs e)
{
    if (Session["ClientsDataSet"] != null)
    {
        string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

        try
        {
            DataSet ds = (DataSet)Session["ClientsDataSet"];
            SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Clients", connectionString);
            SqlCommandBuilder builder = new SqlCommandBuilder(adapter);

            adapter.Update(ds, "Clients");
            LabelMessage.Text = "Modifications enregistrées avec succès";
        }
        catch (Exception ex)
        {
            LabelErreur.Text = $"Erreur lors de l'enregistrement: {ex.Message}";
        }
    }
}
```


## En résumé

Le modèle déconnecté d'ADO.NET avec les classes `DataSet` et `DataAdapter` offre :

1. **Manipulation des données sans connexion persistante** à la base de données, ce qui améliore la scalabilité de votre application.

2. **Une structure en mémoire complète** avec tables, relations et contraintes, similaire à une petite base de données.

3. **Un suivi automatique des modifications** permettant de mettre à jour facilement la base de données.

4. **Une intégration parfaite avec les interfaces utilisateur** grâce aux fonctionnalités de liaison de données.

5. **Une sérialisation facile en XML** pour le stockage ou la transmission des données.

Bien que les approches plus modernes comme Entity Framework soient souvent privilégiées pour les nouvelles applications, le modèle `DataSet`/`DataAdapter` reste pertinent dans de nombreux scénarios, notamment :

- Applications avec interfaces riches nécessitant de manipuler beaucoup de données en mémoire
- Systèmes hérités nécessitant des mises à jour
- Intégration avec des contrôles d'interface utilisateur traditionnels
- Opérations par lots nécessitant des modifications multiples avant synchronisation

Dans le développement .NET moderne (.NET 8), ces outils sont toujours disponibles et fonctionnels, bien que les approches orientées ORM comme Entity Framework Core soient généralement recommandées pour les nouveaux projets.

⏭️ 9.5. [Transactions](/09-acces-aux-donnees-avec-ado-dotnet/9-5-transactions.md)
