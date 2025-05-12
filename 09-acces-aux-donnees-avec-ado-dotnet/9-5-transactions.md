# 9.5. Transactions

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les transactions sont essentielles pour maintenir l'intégrité des données dans les applications qui interagissent avec une base de données. Elles permettent de regrouper plusieurs opérations en une seule unité logique qui soit réussit complètement, soit échoue complètement, respectant ainsi le principe ACID (Atomicité, Cohérence, Isolation, Durabilité).

## 9.5.1. Création et gestion

### Principes fondamentaux

Une transaction regroupe plusieurs opérations de base de données et garantit que :
- Soit toutes les opérations réussissent et les changements sont enregistrés (commit)
- Soit au moins une opération échoue et tous les changements sont annulés (rollback)

### Transaction simple avec ADO.NET

Voici comment créer et gérer une transaction de base :

```textmate
string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Création de la transaction
    SqlTransaction transaction = connection.BeginTransaction();

    try
    {
        // Création des commandes liées à la transaction
        SqlCommand command1 = new SqlCommand("UPDATE CompteBancaire SET Solde = Solde - 100 WHERE Numero = 'A001'", connection);
        command1.Transaction = transaction;

        SqlCommand command2 = new SqlCommand("UPDATE CompteBancaire SET Solde = Solde + 100 WHERE Numero = 'B002'", connection);
        command2.Transaction = transaction;

        // Exécution des commandes
        command1.ExecuteNonQuery();
        command2.ExecuteNonQuery();

        // Si tout s'est bien passé, valider la transaction
        transaction.Commit();
        Console.WriteLine("Transaction réussie !");
    }
    catch (Exception ex)
    {
        // En cas d'erreur, annuler la transaction
        transaction.Rollback();
        Console.WriteLine($"Transaction annulée. Erreur : {ex.Message}");
    }
}
```


### Transactions avec nom (SQL Server)

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Création d'une transaction nommée
    SqlTransaction transaction = connection.BeginTransaction("TransfertFonds");

    try
    {
        // Suite du code comme précédemment...

        transaction.Commit();
    }
    catch
    {
        transaction.Rollback();
        throw;
    }
}
```


### Gestion des points de sauvegarde (Savepoints)

SQL Server et la plupart des bases de données modernes prennent en charge les points de sauvegarde, qui permettent d'annuler partiellement une transaction :

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();
    SqlTransaction transaction = connection.BeginTransaction();

    try
    {
        // Première opération
        SqlCommand command1 = new SqlCommand("INSERT INTO Journal (Description) VALUES ('Début opération')", connection, transaction);
        command1.ExecuteNonQuery();

        // Création d'un point de sauvegarde
        transaction.Save("PointA");

        // Deuxième opération
        SqlCommand command2 = new SqlCommand("UPDATE CompteBancaire SET Solde = Solde - 500 WHERE Numero = 'A001'", connection, transaction);
        command2.ExecuteNonQuery();

        // Simulons une condition qui nécessite d'annuler uniquement la deuxième opération
        bool conditionErreur = VerifierSoldeInsuffisant("A001", connection, transaction);

        if (conditionErreur)
        {
            // Retour au point de sauvegarde (annule uniquement la deuxième opération)
            transaction.Rollback("PointA");

            // Exécuter une opération alternative
            SqlCommand commandAlt = new SqlCommand("INSERT INTO Journal (Description) VALUES ('Opération annulée : solde insuffisant')", connection, transaction);
            commandAlt.ExecuteNonQuery();
        }

        // Validation de la transaction (avec soit toutes les opérations, soit seulement la première et l'alternative)
        transaction.Commit();
    }
    catch (Exception ex)
    {
        // Annulation complète de la transaction
        transaction.Rollback();
        Console.WriteLine($"Transaction complètement annulée. Erreur : {ex.Message}");
    }
}

// Méthode fictive pour vérifier le solde
bool VerifierSoldeInsuffisant(string compte, SqlConnection connection, SqlTransaction transaction)
{
    SqlCommand checkCommand = new SqlCommand(
        "SELECT Solde FROM CompteBancaire WHERE Numero = @Numero",
        connection,
        transaction);
    checkCommand.Parameters.AddWithValue("@Numero", compte);

    decimal solde = (decimal)checkCommand.ExecuteScalar();
    return solde < 0;
}
```


### Transactions avec paramètres (SQL Server)

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Création d'une transaction avec isolation spécifique
    SqlTransaction transaction = connection.BeginTransaction(
        IsolationLevel.ReadCommitted, // Niveau d'isolation (plus de détails dans la section suivante)
        "TransactionAvecParametres"); // Nom de la transaction

    try
    {
        // Suite du code comme précédemment...

        transaction.Commit();
    }
    catch
    {
        transaction.Rollback();
        throw;
    }
}
```


### Différences entre .NET Framework 4.7.2 et .NET 8

#### .NET Framework 4.7.2

```textmate
using System.Data.SqlClient; // Namespace pour SQL Server

using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();
    SqlTransaction transaction = connection.BeginTransaction();

    // Suite du code...
}
```


#### .NET 8

```textmate
using Microsoft.Data.SqlClient; // Nouveau namespace pour SQL Server

await using (SqlConnection connection = new SqlConnection(connectionString))
{
    await connection.OpenAsync();
    SqlTransaction transaction = connection.BeginTransaction();

    // Suite du code...
}
```


> **Remarque pour .NET 8** :
> - Utilisez le mot-clé `await using` pour une gestion asynchrone des ressources
> - Utilisez `OpenAsync()` pour ouvrir la connexion de manière asynchrone
> - Utilisez les méthodes asynchrones comme `ExecuteNonQueryAsync()` quand c'est possible

### Gestion des transactions avec différentes bases de données

#### SQLite (.NET 8)

```textmate
using Microsoft.Data.Sqlite;

await using (SqliteConnection connection = new SqliteConnection("Data Source=mabase.db"))
{
    await connection.OpenAsync();

    using (SqliteTransaction transaction = connection.BeginTransaction())
    {
        try
        {
            await using (SqliteCommand command1 = new SqliteCommand(
                "UPDATE CompteBancaire SET Solde = Solde - 100 WHERE Numero = 'A001'",
                connection, transaction))
            {
                await command1.ExecuteNonQueryAsync();
            }

            await using (SqliteCommand command2 = new SqliteCommand(
                "UPDATE CompteBancaire SET Solde = Solde + 100 WHERE Numero = 'B002'",
                connection, transaction))
            {
                await command2.ExecuteNonQueryAsync();
            }

            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
}
```


#### MySQL (.NET 8)

```textmate
using MySql.Data.MySqlClient;

await using (MySqlConnection connection = new MySqlConnection("Server=localhost;Database=mabase;Uid=root;Pwd=password;"))
{
    await connection.OpenAsync();

    using (MySqlTransaction transaction = connection.BeginTransaction())
    {
        try
        {
            MySqlCommand command1 = new MySqlCommand("UPDATE CompteBancaire SET Solde = Solde - 100 WHERE Numero = 'A001'", connection);
            command1.Transaction = transaction;
            await command1.ExecuteNonQueryAsync();

            MySqlCommand command2 = new MySqlCommand("UPDATE CompteBancaire SET Solde = Solde + 100 WHERE Numero = 'B002'", connection);
            command2.Transaction = transaction;
            await command2.ExecuteNonQueryAsync();

            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
}
```


### Transactions avec DataAdapter

Voici comment utiliser les transactions avec le modèle déconnecté (DataSet et DataAdapter) :

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
            // Configurer le DataAdapter
            SqlDataAdapter adapter = new SqlDataAdapter(
                "SELECT * FROM CompteBancaire",
                connection);
            adapter.SelectCommand.Transaction = transaction;

            // Configurer les commandes
            SqlCommandBuilder builder = new SqlCommandBuilder(adapter);
            adapter.UpdateCommand = builder.GetUpdateCommand();
            adapter.UpdateCommand.Transaction = transaction;

            // Remplir le DataSet
            DataSet ds = new DataSet();
            adapter.Fill(ds, "CompteBancaire");

            // Modifier les données
            DataTable table = ds.Tables["CompteBancaire"];

            // Débit sur le premier compte
            DataRow[] compteA = table.Select("Numero = 'A001'");
            if (compteA.Length > 0)
            {
                compteA[0]["Solde"] = (decimal)compteA[0]["Solde"] - 100;
            }

            // Crédit sur le deuxième compte
            DataRow[] compteB = table.Select("Numero = 'B002'");
            if (compteB.Length > 0)
            {
                compteB[0]["Solde"] = (decimal)compteB[0]["Solde"] + 100;
            }

            // Mettre à jour la base de données
            adapter.Update(ds, "CompteBancaire");

            // Valider la transaction
            transaction.Commit();
            Console.WriteLine("Transaction réussie !");
        }
        catch (Exception ex)
        {
            // Annuler la transaction
            transaction.Rollback();
            Console.WriteLine($"Transaction annulée. Erreur : {ex.Message}");
        }
    }
}
```


## 9.5.2. Isolation levels

Les niveaux d'isolation déterminent comment les transactions interagissent les unes avec les autres, notamment en ce qui concerne la visibilité des données modifiées mais non validées.

### Les différents niveaux d'isolation

ADO.NET supporte plusieurs niveaux d'isolation, représentés par l'énumération `IsolationLevel` :

| Niveau d'isolation | Description | Problèmes résolus | Cas d'utilisation |
|-------------------|------------|------------------|-------------------|
| ReadUncommitted | Permet la lecture de données non validées | Aucun | Reporting ou analyses où les données approximatives sont acceptables |
| ReadCommitted | Permet la lecture uniquement des données validées | Dirty reads | Niveau par défaut pour la plupart des opérations quotidiennes |
| RepeatableRead | Garantit que les mêmes données seront lues si la même requête est exécutée plusieurs fois | Dirty reads, Non-repeatable reads | Transactions qui lisent les mêmes données plusieurs fois |
| Serializable | Niveau le plus restrictif, isole complètement les transactions | Dirty reads, Non-repeatable reads, Phantom reads | Transactions critiques nécessitant la plus haute intégrité |
| Snapshot | Permet aux transactions de voir un instantané cohérent des données | Dirty reads, Non-repeatable reads, certains Phantom reads | Environnements à forte charge avec beaucoup de lectures |
| Unspecified | Utilise le niveau d'isolation par défaut du fournisseur | Dépend du niveau par défaut | À éviter, préférez spécifier explicitement |
| Chaos | Niveau spécifique à SQL Server, similaire à ReadCommitted mais sans les verrous partagés | Dirty reads | Rarement utilisé |

### Définir le niveau d'isolation

```textmate
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Spécifier le niveau d'isolation lors de la création de la transaction
    using (SqlTransaction transaction = connection.BeginTransaction(IsolationLevel.Serializable))
    {
        try
        {
            SqlCommand command = new SqlCommand("SELECT Solde FROM CompteBancaire WHERE Numero = 'A001'", connection, transaction);
            decimal solde = (decimal)command.ExecuteScalar();

            // Autres opérations...

            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
}
```


### Exemples de problèmes résolus par les niveaux d'isolation

#### Dirty Reads (lectures sales)

Sans isolation adéquate, une transaction peut lire des données modifiées par une autre transaction non encore validée.

```textmate
// Transaction 1 (IsolationLevel.ReadUncommitted) - Peut voir des données non validées
using (SqlConnection connection1 = new SqlConnection(connectionString))
{
    connection1.Open();

    using (SqlTransaction transaction1 = connection1.BeginTransaction(IsolationLevel.ReadUncommitted))
    {
        SqlCommand command1 = new SqlCommand("SELECT Solde FROM CompteBancaire WHERE Numero = 'A001'", connection1, transaction1);
        decimal solde1 = (decimal)command1.ExecuteScalar();
        Console.WriteLine($"Solde initial lu par Transaction 1: {solde1}");

        // Pendant ce temps, dans Transaction 2, le solde est modifié mais non validé
        // ...

        // Transaction 1 relit le solde (qui a été modifié mais non validé par Transaction 2)
        SqlCommand command1b = new SqlCommand("SELECT Solde FROM CompteBancaire WHERE Numero = 'A001'", connection1, transaction1);
        decimal solde1b = (decimal)command1b.ExecuteScalar();
        Console.WriteLine($"Solde relu par Transaction 1: {solde1b}"); // Peut afficher la valeur modifiée mais non validée

        transaction1.Commit();
    }
}

// Transaction 2 - Modifie des données sans les valider immédiatement
using (SqlConnection connection2 = new SqlConnection(connectionString))
{
    connection2.Open();

    using (SqlTransaction transaction2 = connection2.BeginTransaction())
    {
        SqlCommand command2 = new SqlCommand("UPDATE CompteBancaire SET Solde = Solde + 500 WHERE Numero = 'A001'", connection2, transaction2);
        command2.ExecuteNonQuery();

        Console.WriteLine("Transaction 2 a modifié le solde mais n'a pas encore validé");

        // Simuler un délai
        Thread.Sleep(5000);

        // Décider si on valide ou annule
        bool valider = DemanderConfirmation(); // Fonction fictive

        if (valider)
        {
            transaction2.Commit();
            Console.WriteLine("Transaction 2 validée");
        }
        else
        {
            transaction2.Rollback();
            Console.WriteLine("Transaction 2 annulée");
        }
    }
}
```


Pour éviter ce problème, utilisez au moins `IsolationLevel.ReadCommitted`.

#### Non-repeatable Reads (lectures non répétables)

Une transaction lit les mêmes données deux fois, mais entre-temps, une autre transaction a modifié et validé ces données.

```textmate
// Solution: Utiliser IsolationLevel.RepeatableRead ou supérieur
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    using (SqlTransaction transaction = connection.BeginTransaction(IsolationLevel.RepeatableRead))
    {
        SqlCommand command1 = new SqlCommand("SELECT Solde FROM CompteBancaire WHERE Numero = 'A001'", connection, transaction);
        decimal solde1 = (decimal)command1.ExecuteScalar();
        Console.WriteLine($"Première lecture: {solde1}");

        // Même si une autre transaction modifie et valide des données entre-temps,
        // la prochaine lecture dans cette transaction verra toujours les mêmes données

        SqlCommand command2 = new SqlCommand("SELECT Solde FROM CompteBancaire WHERE Numero = 'A001'", connection, transaction);
        decimal solde2 = (decimal)command2.ExecuteScalar();
        Console.WriteLine($"Deuxième lecture: {solde2}"); // Garantie d'être identique à solde1

        transaction.Commit();
    }
}
```


#### Phantom Reads (lectures fantômes)

Une transaction exécute deux fois la même requête, mais la deuxième fois retourne des lignes supplémentaires (ou manquantes) car une autre transaction a ajouté ou supprimé des lignes.

```textmate
// Solution: Utiliser IsolationLevel.Serializable
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    using (SqlTransaction transaction = connection.BeginTransaction(IsolationLevel.Serializable))
    {
        SqlCommand command1 = new SqlCommand("SELECT COUNT(*) FROM CompteBancaire WHERE Solde > 1000", connection, transaction);
        int count1 = (int)command1.ExecuteScalar();
        Console.WriteLine($"Premier comptage: {count1} comptes avec solde > 1000");

        // Même si une autre transaction ajoute ou supprime des comptes entre-temps,
        // la prochaine lecture dans cette transaction verra toujours le même ensemble de données

        SqlCommand command2 = new SqlCommand("SELECT COUNT(*) FROM CompteBancaire WHERE Solde > 1000", connection, transaction);
        int count2 = (int)command2.ExecuteScalar();
        Console.WriteLine($"Second comptage: {count2} comptes avec solde > 1000"); // Garantie d'être identique à count1

        transaction.Commit();
    }
}
```


### Quand utiliser chaque niveau d'isolation

- **ReadUncommitted** : Rarement recommandé, sauf pour des rapports non critiques où la précision exacte n'est pas essentielle.

- **ReadCommitted** : Bon niveau par défaut pour la plupart des applications. Offre un bon équilibre entre performances et protection des données.

- **RepeatableRead** : Pour les transactions qui doivent relire les mêmes données plusieurs fois et s'attendent à ce qu'elles soient cohérentes.

- **Snapshot** : Excellent pour les environnements avec beaucoup de lectures, car il évite de verrouiller les données sans sacrifier la cohérence.

- **Serializable** : Pour les transactions critiques nécessitant une intégrité absolue, mais attention aux problèmes de performance et de blocage.

### Exemple de choix du niveau d'isolation adapté

```textmate
public void TraiterPaiement(string compteSource, string compteDestination, decimal montant)
{
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();

        // Utiliser Serializable pour cette opération critique
        using (SqlTransaction transaction = connection.BeginTransaction(IsolationLevel.Serializable))
        {
            try
            {
                // Vérifier si le compte source a un solde suffisant
                SqlCommand checkCommand = new SqlCommand(
                    "SELECT Solde FROM CompteBancaire WHERE Numero = @Numero",
                    connection,
                    transaction);
                checkCommand.Parameters.AddWithValue("@Numero", compteSource);

                decimal soldeActuel = (decimal)checkCommand.ExecuteScalar();

                if (soldeActuel < montant)
                {
                    throw new InvalidOperationException("Solde insuffisant");
                }

                // Débiter le compte source
                SqlCommand debitCommand = new SqlCommand(
                    "UPDATE CompteBancaire SET Solde = Solde - @Montant WHERE Numero = @Numero",
                    connection,
                    transaction);
                debitCommand.Parameters.AddWithValue("@Montant", montant);
                debitCommand.Parameters.AddWithValue("@Numero", compteSource);
                debitCommand.ExecuteNonQuery();

                // Créditer le compte destination
                SqlCommand creditCommand = new SqlCommand(
                    "UPDATE CompteBancaire SET Solde = Solde + @Montant WHERE Numero = @Numero",
                    connection,
                    transaction);
                creditCommand.Parameters.AddWithValue("@Montant", montant);
                creditCommand.Parameters.AddWithValue("@Numero", compteDestination);
                creditCommand.ExecuteNonQuery();

                // Enregistrer le mouvement
                SqlCommand logCommand = new SqlCommand(
                    "INSERT INTO Mouvements (CompteSource, CompteDestination, Montant, DateOperation) VALUES (@Source, @Destination, @Montant, @Date)",
                    connection,
                    transaction);
                logCommand.Parameters.AddWithValue("@Source", compteSource);
                logCommand.Parameters.AddWithValue("@Destination", compteDestination);
                logCommand.Parameters.AddWithValue("@Montant", montant);
                logCommand.Parameters.AddWithValue("@Date", DateTime.Now);
                logCommand.ExecuteNonQuery();

                transaction.Commit();
                Console.WriteLine("Transfert réussi !");
            }
            catch (Exception ex)
            {
                transaction.Rollback();
                Console.WriteLine($"Transfert échoué : {ex.Message}");
                throw;
            }
        }
    }
}
```


## 9.5.3. Transactions distribuées

Les transactions distribuées permettent de coordonner des transactions sur plusieurs sources de données ou serveurs.

### Qu'est-ce qu'une transaction distribuée ?

Une transaction distribuée est une transaction qui englobe plusieurs ressources, comme :
- Plusieurs bases de données (par exemple SQL Server et Oracle)
- Plusieurs instances d'une même base de données
- Une base de données et un autre système transactionnel (comme un service de message)

### Utilisation de System.Transactions (.NET Framework 4.7.2 et .NET 8)

.NET fournit l'espace de noms `System.Transactions` qui simplifie la gestion des transactions distribuées.

#### TransactionScope

La classe `TransactionScope` est la façon la plus simple de gérer les transactions distribuées :

```textmate
using System.Transactions;

// Créer une portée de transaction
using (TransactionScope scope = new TransactionScope())
{
    // Premier contexte
    using (SqlConnection connection1 = new SqlConnection("Data Source=Serveur1;Initial Catalog=BaseDeDonnees1;Integrated Security=True"))
    {
        connection1.Open();
        SqlCommand command1 = new SqlCommand("UPDATE CompteBancaire SET Solde = Solde - 100 WHERE Numero = 'A001'", connection1);
        command1.ExecuteNonQuery();
    }

    // Deuxième contexte (peut être une autre base de données)
    using (SqlConnection connection2 = new SqlConnection("Data Source=Serveur2;Initial Catalog=BaseDeDonnees2;Integrated Security=True"))
    {
        connection2.Open();
        SqlCommand command2 = new SqlCommand("UPDATE Journal SET TotalTransactions = TotalTransactions + 1", connection2);
        command2.ExecuteNonQuery();
    }

    // Si tout s'est bien passé, valider la transaction
    scope.Complete();
}
// La transaction est automatiquement validée si scope.Complete() a été appelé,
// sinon elle est automatiquement annulée
```


#### TransactionScope avec options

```textmate
// Configuration des options de la transaction
TransactionOptions options = new TransactionOptions
{
    IsolationLevel = IsolationLevel.ReadCommitted,
    Timeout = TimeSpan.FromSeconds(30) // Limite la durée de la transaction à 30 secondes
};

using (TransactionScope scope = new TransactionScope(TransactionScopeOption.Required, options))
{
    // Code de la transaction...

    scope.Complete();
}
```


#### Transactions imbriquées

```textmate
using (TransactionScope scopeExterieure = new TransactionScope())
{
    // Opérations dans la transaction extérieure

    // Création d'une transaction imbriquée
    using (TransactionScope scopeImbriquee = new TransactionScope(TransactionScopeOption.Required))
    {
        // Opérations dans la transaction imbriquée

        // Si la transaction imbriquée réussit
        scopeImbriquee.Complete();
    }

    // Si la transaction extérieure réussit
    scopeExterieure.Complete();
}
// La transaction globale est validée seulement si les deux appels Complete() ont été exécutés
```


#### Options TransactionScopeOption

- **Required** (par défaut) : Utilise la transaction existante si elle existe, sinon en crée une nouvelle
- **RequiresNew** : Crée toujours une nouvelle transaction, indépendante de toute transaction existante
- **Suppress** : N'utilise pas de transaction, même s'il y en a une en cours

```textmate
// Exemple avec RequiresNew
using (TransactionScope scopeExterieure = new TransactionScope())
{
    // Code de la transaction extérieure

    // Cette transaction est indépendante de la transaction extérieure
    using (TransactionScope scopeIndependante = new TransactionScope(TransactionScopeOption.RequiresNew))
    {
        // Code de la transaction indépendante

        // Cette transaction peut être validée indépendamment
        scopeIndependante.Complete();
    }

    // La transaction extérieure peut continuer, qu'importe le résultat de la transaction indépendante
    scopeExterieure.Complete();
}
```


### Transactions distribuées avec ADO.NET natif

Vous pouvez également utiliser les transactions distribuées directement avec ADO.NET :

```textmate
// Création d'une transaction distribuée
using (System.Transactions.CommittableTransaction tx = new System.Transactions.CommittableTransaction())
{
    // Première connexion
    using (SqlConnection connection1 = new SqlConnection("Data Source=Serveur1;Initial Catalog=BD1;Integrated Security=True"))
    {
        connection1.Open();

        // Enrôler la connexion dans la transaction distribuée
        connection1.EnlistTransaction(tx);

        SqlCommand command1 = new SqlCommand("UPDATE Table1 SET Colonne = 'Valeur1'", connection1);
        command1.ExecuteNonQuery();
    }

    // Deuxième connexion
    using (SqlConnection connection2 = new SqlConnection("Data Source=Serveur2;Initial Catalog=BD2;Integrated Security=True"))
    {
        connection2.Open();

        // Enrôler la connexion dans la transaction distribuée
        connection2.EnlistTransaction(tx);

        SqlCommand command2 = new SqlCommand("UPDATE Table2 SET Colonne = 'Valeur2'", connection2);
        command2.ExecuteNonQuery();
    }

    // Valider la transaction distribuée
    tx.Commit();
}
```


### Considérations importantes

- **Performance** : Les transactions distribuées sont plus coûteuses en termes de performance que les transactions locales.
- **Dépendance à MSDTC** : Sur Windows, les transactions distribuées dépendent généralement du Microsoft Distributed Transaction Coordinator (MSDTC).
- **Timeout** : Configurez toujours un timeout raisonnable pour éviter de bloquer des ressources trop longtemps.
- **Disponibilité** : Dans les environnements de production, assurez-vous que MSDTC est correctement configuré et disponible.

## 9.5.4. Pattern Unit of Work

Le pattern Unit of Work (Unité de Travail) est un modèle de conception qui encapsule les opérations de base de données en une seule unité logique.

### Principes du pattern

1. **Regroupement des opérations** : Toutes les opérations de base de données sont regroupées en une seule unité.
2. **Tout ou rien** : L'unité entière réussit ou échoue comme un ensemble.
3. **Séparation des responsabilités** : Sépare la logique métier de la gestion des transactions.

### Implémentation de base

Voici une implémentation simple du pattern Unit of Work avec ADO.NET :

```textmate
// Interface de base
public interface IUnitOfWork : IDisposable
{
    void Begin();
    void Commit();
    void Rollback();

    // Commandes à exécuter
    void RegisterCommand(SqlCommand command);
}

// Implémentation
public class AdoNetUnitOfWork : IUnitOfWork
{
    private readonly string _connectionString;
    private SqlConnection _connection;
    private SqlTransaction _transaction;
    private readonly List<SqlCommand> _commands = new List<SqlCommand>();
    private bool _disposed = false;

    public AdoNetUnitOfWork(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void Begin()
    {
        _connection = new SqlConnection(_connectionString);
        _connection.Open();
        _transaction = _connection.BeginTransaction();
    }

    public void RegisterCommand(SqlCommand command)
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("La transaction n'a pas été démarrée.");
        }

        command.Connection = _connection;
        command.Transaction = _transaction;
        _commands.Add(command);
    }

    public void Commit()
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("La transaction n'a pas été démarrée.");
        }

        try
        {
            // Exécuter toutes les commandes
            foreach (SqlCommand command in _commands)
            {
                command.ExecuteNonQuery();
            }

            // Valider la transaction
            _transaction.Commit();
        }
        catch
        {
            Rollback();
            throw;
        }
        finally
        {
            _commands.Clear();
            _transaction = null;
        }
    }

    public void Rollback()
    {
        if (_transaction != null)
        {
            _transaction.Rollback();
            _transaction = null;
            _commands.Clear();
        }
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                if (_transaction != null)
                {
                    _transaction.Dispose();
                    _transaction = null;
                }

                if (_connection != null)
                {
                    _connection.Close();
                    _connection.Dispose();
                    _connection = null;
                }
            }

            _disposed = true;
        }
    }
}
```


### Utilisation de l'Unit of Work

```textmate
public void TransfererFonds(string compteSource, string compteDestination, decimal montant)
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    using (IUnitOfWork unitOfWork = new AdoNetUnitOfWork(connectionString))
    {
        try
        {
            unitOfWork.Begin();

            // Préparer les commandes
            SqlCommand debitCommand = new SqlCommand(
                "UPDATE CompteBancaire SET Solde = Solde - @Montant WHERE Numero = @CompteSource AND Solde >= @Montant");
            debitCommand.Parameters.AddWithValue("@Montant", montant);
            debitCommand.Parameters.AddWithValue("@CompteSource", compteSource);

            SqlCommand creditCommand = new SqlCommand(
                "UPDATE CompteBancaire SET Solde = Solde + @Montant WHERE Numero = @CompteDestination");
            creditCommand.Parameters.AddWithValue("@Montant", montant);
            creditCommand.Parameters.AddWithValue("@CompteDestination", compteDestination);

            SqlCommand journalCommand = new SqlCommand(
                "INSERT INTO Journal (CompteSource, CompteDestination, Montant, DateOperation) VALUES (@Source, @Destination, @Montant, @Date)");
            journalCommand.Parameters.AddWithValue("@Source", compteSource);
            journalCommand.Parameters.AddWithValue("@Destination", compteDestination);
            journalCommand.Parameters.AddWithValue("@Montant", montant);
            journalCommand.Parameters.AddWithValue("@Date", DateTime.Now);

            // Enregistrer les commandes auprès de l'unité de travail
            unitOfWork.RegisterCommand(debitCommand);
            unitOfWork.RegisterCommand(creditCommand);
            unitOfWork.RegisterCommand(journalCommand);

            // Exécuter toutes les commandes dans une transaction
            unitOfWork.Commit();

            Console.WriteLine("Transfert réussi !");
        }
        catch (Exception ex)
        {
            unitOfWork.Rollback();
            Console.WriteLine($"Erreur lors du transfert : {ex.Message}");
            throw;
        }
    }
}
```


### Version asynchrone pour .NET 8

```textmate
// Interface asynchrone
public interface IAsyncUnitOfWork : IAsyncDisposable
{
    Task BeginAsync();
    Task CommitAsync();
    Task RollbackAsync();

    void RegisterCommand(SqlCommand command);
}

// Implémentation asynchrone
public class AsyncAdoNetUnitOfWork : IAsyncUnitOfWork
{
    private readonly string _connectionString;
    private SqlConnection _connection;
    private SqlTransaction _transaction;
    private readonly List<SqlCommand> _commands = new List<SqlCommand>();
    private bool _disposed = false;

    public AsyncAdoNetUnitOfWork(string connectionString)
    {
        _connectionString = connectionString;
    }

    public async Task BeginAsync()
    {
        _connection = new SqlConnection(_connectionString);
        await _connection.OpenAsync();
        _transaction = _connection.BeginTransaction();
    }

    public void RegisterCommand(SqlCommand command)
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("La transaction n'a pas été démarrée.");
        }

        command.Connection = _connection;
        command.Transaction = _transaction;
        _commands.Add(command);
    }

    public async Task CommitAsync()
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("La transaction n'a pas été démarrée.");
        }

        try
        {
            // Exécuter toutes les commandes
            foreach (SqlCommand command in _commands)
            {
                await command.ExecuteNonQueryAsync();
            }

            // Valider la transaction
            await _transaction.CommitAsync();
        }
        catch
        {
            await RollbackAsync();
            throw;
        }
        finally
        {
            _commands.Clear();
            _transaction = null;
        }
    }

    public async Task RollbackAsync()
    {
        if (_transaction != null)
        {
            await _transaction.RollbackAsync();
            _transaction = null;
            _commands.Clear();
        }
    }

    public async ValueTask DisposeAsync()
    {
        await DisposeAsyncCore();
        GC.SuppressFinalize(this);
    }

    protected virtual async ValueTask DisposeAsyncCore()
    {
        if (!_disposed)
        {
            if (_transaction != null)
            {
                await _transaction.DisposeAsync();
                _transaction = null;
            }

            if (_connection != null)
            {
                await _connection.CloseAsync();
                await _connection.DisposeAsync();
                _connection = null;
            }

            _disposed = true;
        }
    }
}
```


### Utilisation asynchrone en .NET 8

```textmate
public async Task TransfererFondsAsync(string compteSource, string compteDestination, decimal montant)
{
    string connectionString = "Data Source=MonServeur;Initial Catalog=MaBaseDeDonnees;Integrated Security=True";

    await using (IAsyncUnitOfWork unitOfWork = new AsyncAdoNetUnitOfWork(connectionString))
    {
        try
        {
            await unitOfWork.BeginAsync();

            // Préparer les commandes
            SqlCommand debitCommand = new SqlCommand(
                "UPDATE CompteBancaire SET Solde = Solde - @Montant WHERE Numero = @CompteSource AND Solde >= @Montant");
            debitCommand.Parameters.AddWithValue("@Montant", montant);
            debitCommand.Parameters.AddWithValue("@CompteSource", compteSource);

            SqlCommand creditCommand = new SqlCommand(
                "UPDATE CompteBancaire SET Solde = Solde + @Montant WHERE Numero = @CompteDestination");
            creditCommand.Parameters.AddWithValue("@Montant", montant);
            creditCommand.Parameters.AddWithValue("@CompteDestination", compteDestination);

            SqlCommand journalCommand = new SqlCommand(
                "INSERT INTO Journal (CompteSource, CompteDestination, Montant, DateOperation) VALUES (@Source, @Destination, @Montant, @Date)");
            journalCommand.Parameters.AddWithValue("@Source", compteSource);
            journalCommand.Parameters.AddWithValue("@Destination", compteDestination);
            journalCommand.Parameters.AddWithValue("@Montant", montant);
            journalCommand.Parameters.AddWithValue("@Date", DateTime.Now);

            // Enregistrer les commandes auprès de l'unité de travail
            unitOfWork.RegisterCommand(debitCommand);
            unitOfWork.RegisterCommand(creditCommand);
            unitOfWork.RegisterCommand(journalCommand);

            // Exécuter toutes les commandes dans une transaction
            await unitOfWork.CommitAsync();

            Console.WriteLine("Transfert réussi !");
        }
        catch (Exception ex)
        {
            await unitOfWork.RollbackAsync();
            Console.WriteLine($"Erreur lors du transfert : {ex.Message}");
            throw;
        }
    }
}
```


### Extension du pattern avec les Repositories

Le pattern Unit of Work est souvent combiné avec le pattern Repository pour une meilleure organisation du code.

```textmate
// Interface Repository pour la couche d'accès aux données
public interface ICompteBancaireRepository
{
    Task<CompteBancaire> GetByNumeroAsync(string numero);
    void Debiter(string numero, decimal montant);
    void Crediter(string numero, decimal montant);
    void Save(CompteBancaire compte);
}

// Implémentation du Repository
public class CompteBancaireRepository : ICompteBancaireRepository
{
    private readonly IAsyncUnitOfWork _unitOfWork;

    public CompteBancaireRepository(IAsyncUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public async Task<CompteBancaire> GetByNumeroAsync(string numero)
    {
        SqlCommand command = new SqlCommand("SELECT Numero, Solde FROM CompteBancaire WHERE Numero = @Numero");
        command.Parameters.AddWithValue("@Numero", numero);
        _unitOfWork.RegisterCommand(command);

        using var reader = await command.ExecuteReaderAsync();
        if (await reader.ReadAsync())
        {
            return new CompteBancaire
            {
                Numero = reader.GetString(0),
                Solde = reader.GetDecimal(1)
            };
        }

        return null;
    }

    public void Debiter(string numero, decimal montant)
    {
        SqlCommand command = new SqlCommand(
            "UPDATE CompteBancaire SET Solde = Solde - @Montant WHERE Numero = @Numero AND Solde >= @Montant");
        command.Parameters.AddWithValue("@Montant", montant);
        command.Parameters.AddWithValue("@Numero", numero);

        _unitOfWork.RegisterCommand(command);
    }

    public void Crediter(string numero, decimal montant)
    {
        SqlCommand command = new SqlCommand(
            "UPDATE CompteBancaire SET Solde = Solde + @Montant WHERE Numero = @Numero");
        command.Parameters.AddWithValue("@Montant", montant);
        command.Parameters.AddWithValue("@Numero", numero);

        _unitOfWork.RegisterCommand(command);
    }

    public void Save(CompteBancaire compte)
    {
        SqlCommand command = new SqlCommand(
            "UPDATE CompteBancaire SET Solde = @Solde WHERE Numero = @Numero");
        command.Parameters.AddWithValue("@Solde", compte.Solde);
        command.Parameters.AddWithValue("@Numero", compte.Numero);

        _unitOfWork.RegisterCommand(command);
    }
}

// Modèle
public class CompteBancaire
{
    public string Numero { get; set; }
    public decimal Solde { get; set; }
}

// Service métier utilisant les repositories et l'Unit of Work
public class TransfertService
{
    private readonly IAsyncUnitOfWork _unitOfWork;
    private readonly ICompteBancaireRepository _compteRepository;

    public TransfertService(IAsyncUnitOfWork unitOfWork, ICompteBancaireRepository compteRepository)
    {
        _unitOfWork = unitOfWork;
        _compteRepository = compteRepository;
    }

    public async Task TransfererFondsAsync(string compteSource, string compteDestination, decimal montant)
    {
        if (montant <= 0)
        {
            throw new ArgumentException("Le montant doit être positif");
        }

        try
        {
            await _unitOfWork.BeginAsync();

            // Utiliser le repository pour les opérations
            _compteRepository.Debiter(compteSource, montant);
            _compteRepository.Crediter(compteDestination, montant);

            // Journal
            SqlCommand journalCommand = new SqlCommand(
                "INSERT INTO Journal (CompteSource, CompteDestination, Montant, DateOperation) " +
                "VALUES (@Source, @Destination, @Montant, @Date)");
            journalCommand.Parameters.AddWithValue("@Source", compteSource);
            journalCommand.Parameters.AddWithValue("@Destination", compteDestination);
            journalCommand.Parameters.AddWithValue("@Montant", montant);
            journalCommand.Parameters.AddWithValue("@Date", DateTime.Now);

            _unitOfWork.RegisterCommand(journalCommand);

            // Valider toutes les opérations
            await _unitOfWork.CommitAsync();

            Console.WriteLine("Transfert réussi !");
        }
        catch (Exception ex)
        {
            await _unitOfWork.RollbackAsync();
            Console.WriteLine($"Erreur lors du transfert : {ex.Message}");
            throw;
        }
    }
}
```


### Avantages du pattern Unit of Work

1. **Organisation du code** : Sépare clairement la logique métier de la logique de transaction.
2. **Réutilisabilité** : Les repositories peuvent être réutilisés dans différentes parties de l'application.
3. **Testabilité** : Facilite les tests unitaires grâce à l'abstraction des opérations de base de données.
4. **Cohérence** : Garantit que toutes les opérations sont exécutées ensemble ou pas du tout.
5. **Contrôle** : Donne un contrôle précis sur le moment où les transactions commencent et se terminent.

### Inconvénients et considérations

1. **Complexité** : Ajoute une couche d'abstraction supplémentaire.
2. **Performance** : Peut avoir un impact sur les performances pour les opérations simples.
3. **Mémoire** : Toutes les commandes sont stockées en mémoire avant d'être exécutées.
4. **Verrous** : Les transactions longues peuvent créer des problèmes de verrouillage en base de données.

## En résumé

Les transactions sont un mécanisme fondamental pour maintenir l'intégrité des données dans une application. Dans ce chapitre, nous avons exploré :

1. **Création et gestion des transactions** : Comment démarrer, valider ou annuler des transactions.
2. **Niveaux d'isolation** : Les différents niveaux qui déterminent comment les transactions interagissent entre elles.
3. **Transactions distribuées** : Comment gérer des transactions qui couvrent plusieurs bases de données ou services.
4. **Pattern Unit of Work** : Un modèle de conception qui encapsule les opérations de base de données en une seule unité logique.

En maîtrisant ces concepts, vous pourrez concevoir des applications robustes qui maintiennent l'intégrité des données même dans des scénarios complexes.

Pour optimiser vos transactions, n'oubliez pas ces bonnes pratiques :
- Gardez les transactions aussi courtes que possible
- Choisissez le niveau d'isolation approprié pour votre cas d'utilisation
- Utilisez des transactions distribuées uniquement lorsque c'est nécessaire
- Implémentez le pattern Unit of Work pour organiser et encapsuler votre logique de transaction

Ces techniques fonctionnent aussi bien dans .NET Framework 4.7.2 que dans .NET 8, avec les adaptations nécessaires pour profiter des fonctionnalités asynchrones modernes dans .NET 8.

⏭️
