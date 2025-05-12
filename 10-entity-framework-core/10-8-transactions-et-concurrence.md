# 10.8. Transactions et concurrence

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Dans les applications de base de données modernes, deux défis majeurs se posent souvent : la gestion des transactions (pour garantir l'intégrité des données) et la gestion de la concurrence (pour gérer les accès multiples simultanés aux mêmes données). Entity Framework Core offre des outils puissants pour résoudre ces deux problèmes.

## 10.8.1. Gestion des transactions

Une transaction est un ensemble d'opérations qui doivent être exécutées comme une unité atomique : soit toutes les opérations réussissent, soit aucune n'est appliquée en cas d'échec. Cette approche garantit la cohérence de vos données.

### Transactions implicites avec SaveChanges

Par défaut, EF Core enveloppe chaque appel à `SaveChanges()` dans une transaction implicite :

```csharp
using (var context = new LibraryContext())
{
    // Toutes ces modifications seront intégrées dans une transaction implicite
    context.Livres.Add(new Livre { Titre = "Le Petit Prince" });
    context.Auteurs.Add(new Auteur { Nom = "Saint-Exupéry" });

    // Une seule transaction est créée pour toutes les opérations
    context.SaveChanges();
}
```

Si une erreur se produit pendant `SaveChanges()`, toutes les modifications sont annulées automatiquement.

### Transactions explicites

Pour les scénarios plus complexes, vous pouvez créer et gérer explicitement des transactions :

```csharp
using (var context = new LibraryContext())
{
    // Démarrer une transaction explicite
    using (var transaction = context.Database.BeginTransaction())
    {
        try
        {
            // Première opération
            var livre = new Livre { Titre = "1984" };
            context.Livres.Add(livre);
            context.SaveChanges();

            // Deuxième opération
            var auteur = new Auteur { Nom = "George Orwell" };
            context.Auteurs.Add(auteur);
            context.SaveChanges();

            // Lier le livre à l'auteur
            livre.AuteurId = auteur.Id;
            context.SaveChanges();

            // Valider la transaction si tout s'est bien passé
            transaction.Commit();
        }
        catch (Exception ex)
        {
            // En cas d'erreur, annuler toutes les modifications
            Console.WriteLine($"Erreur : {ex.Message}");
            transaction.Rollback();

            // Propager l'exception ou la gérer selon vos besoins
            throw;
        }
    }
}
```

### Transactions asynchrones

EF Core prend également en charge les transactions asynchrones :

```csharp
using (var context = new LibraryContext())
{
    // Démarrer une transaction asynchrone
    using (var transaction = await context.Database.BeginTransactionAsync())
    {
        try
        {
            // Opérations de base de données...
            await context.SaveChangesAsync();

            // Valider la transaction
            await transaction.CommitAsync();
        }
        catch (Exception)
        {
            // Annuler la transaction
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

### Niveaux d'isolation des transactions

Vous pouvez spécifier le niveau d'isolation de vos transactions pour contrôler comment les transactions interagissent entre elles :

```csharp
using (var transaction = context.Database.BeginTransaction(
    System.Data.IsolationLevel.ReadCommitted))
{
    // Opérations avec niveau d'isolation ReadCommitted
}
```

Les niveaux d'isolation courants sont :
- `ReadUncommitted` : Performance maximale, mais risque de lectures sales (dirty reads)
- `ReadCommitted` : Empêche les lectures sales, mais permet les lectures non reproductibles
- `RepeatableRead` : Garantit que les données lues ne changent pas pendant la transaction
- `Serializable` : Isolation maximale, mais performances potentiellement réduites

### Transactions distribuées

EF Core peut participer à des transactions distribuées impliquant plusieurs ressources :

```csharp
using (var scope = new TransactionScope())
{
    // Premier contexte (première base de données)
    using (var context1 = new FirstDbContext())
    {
        context1.SomeEntities.Add(new SomeEntity { /* ... */ });
        context1.SaveChanges();
    }

    // Deuxième contexte (deuxième base de données)
    using (var context2 = new SecondDbContext())
    {
        context2.OtherEntities.Add(new OtherEntity { /* ... */ });
        context2.SaveChanges();
    }

    // Si cette ligne est atteinte sans exception, les deux transactions sont validées
    scope.Complete();
}
```

> **Note** : Pour utiliser `TransactionScope`, vous aurez besoin d'ajouter une référence au package `System.Transactions.dll`.

## 10.8.2. Résolution de conflits

Dans un environnement multi-utilisateurs, plusieurs personnes peuvent tenter de modifier les mêmes données simultanément. EF Core fournit des mécanismes pour détecter et résoudre ces conflits.

### Détection de conflits de base

Par défaut, EF Core utilise une approche "Last Win" (le dernier gagne) sans aucune vérification de concurrence :

```csharp
// Utilisateur 1 charge un livre
using (var context1 = new LibraryContext())
{
    var livre1 = context1.Livres.Find(1);
    livre1.Titre = "Nouveau titre par utilisateur 1";

    // L'utilisateur 1 ne sauvegarde pas immédiatement
    // ...

    // Pendant ce temps, l'utilisateur 2 modifie le même livre
    using (var context2 = new LibraryContext())
    {
        var livre2 = context2.Livres.Find(1);
        livre2.Titre = "Titre modifié par utilisateur 2";
        context2.SaveChanges(); // Sauvegarde réussie
    }

    // Maintenant l'utilisateur 1 sauvegarde
    context1.SaveChanges(); // Écrase les modifications de l'utilisateur 2 sans avertissement
}
```

Cette approche est simple mais dangereuse, car elle peut entraîner des pertes de données.

### Détection des conflits avec tokens de concurrence

Pour détecter les conflits, vous pouvez utiliser un token de concurrence (nous le verrons plus en détail dans la section 10.8.3) :

```csharp
// Même scénario, mais avec détection de concurrence
using (var context1 = new LibraryContext())
{
    var livre1 = context1.Livres.Find(1);
    livre1.Titre = "Nouveau titre par utilisateur 1";

    // L'utilisateur 2 modifie et sauvegarde
    using (var context2 = new LibraryContext())
    {
        var livre2 = context2.Livres.Find(1);
        livre2.Titre = "Titre modifié par utilisateur 2";
        context2.SaveChanges(); // Sauvegarde réussie
    }

    try
    {
        context1.SaveChanges(); // Générera DbUpdateConcurrencyException
    }
    catch (DbUpdateConcurrencyException ex)
    {
        // Conflit détecté - voir résolution ci-dessous
    }
}
```

### Stratégies de résolution de conflits

Lorsqu'un conflit est détecté, vous avez plusieurs options pour le résoudre :

#### 1. Client gagne (Force l'écriture)

```csharp
try
{
    context1.SaveChanges();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        // Obtenir les valeurs actuelles de la base de données
        var databaseValues = entry.GetDatabaseValues();

        // Remplacer les valeurs originales (que EF Core utilise pour la détection)
        // par les valeurs actuelles de la base de données
        entry.OriginalValues.SetValues(databaseValues);
    }

    // Réessayer la sauvegarde - maintenant elle réussira en écrasant les modifications
    context1.SaveChanges();
}
```

#### 2. Serveur gagne (Abandonner les modifications locales)

```csharp
try
{
    context1.SaveChanges();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        // Rechargez l'entité depuis la base de données
        entry.Reload();
    }

    // Les modifications locales sont perdues, remplacées par les valeurs du serveur
}
```

#### 3. Fusion des valeurs

```csharp
try
{
    context1.SaveChanges();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        // Obtenir les valeurs de la base de données
        var databaseValues = entry.GetDatabaseValues();
        var databaseValuesAsTBook = (Livre)databaseValues.ToObject();

        // Obtenir les valeurs actuelles en mémoire
        var currentValues = entry.CurrentValues;
        var livre = (Livre)entry.Entity;

        // Choisir quelle propriété conserver
        // Par exemple : garder le titre local mais la date de la base
        livre.Titre = livre.Titre; // Garder notre valeur
        livre.DateModification = databaseValuesAsTBook.DateModification; // Prendre la valeur de la base

        // Mettre à jour les valeurs originales pour éviter un nouveau conflit
        entry.OriginalValues.SetValues(databaseValues);
    }

    // Réessayer la sauvegarde avec nos valeurs fusionnées
    context1.SaveChanges();
}
```

#### 4. Informer l'utilisateur

```csharp
try
{
    context1.SaveChanges();
}
catch (DbUpdateConcurrencyException ex)
{
    // Informer l'utilisateur du conflit
    var conflictMessage = "Vos modifications n'ont pas pu être enregistrées car les données " +
                         "ont été modifiées par un autre utilisateur. Voici les nouvelles valeurs:";

    foreach (var entry in ex.Entries)
    {
        var databaseValues = entry.GetDatabaseValues();
        var entity = (Livre)databaseValues.ToObject();

        conflictMessage += $"\nTitre: {entity.Titre}";
        // Ajouter d'autres propriétés pertinentes
    }

    // Afficher le message et permettre à l'utilisateur de choisir
    Console.WriteLine(conflictMessage);

    // Options pour l'utilisateur:
    // 1. Conserver ses modifications et forcer l'enregistrement
    // 2. Abandonner ses modifications et accepter les valeurs du serveur
    // 3. Fusionner manuellement et réessayer
}
```

## 10.8.3. Tokens de concurrence

Un token de concurrence est une propriété (ou un ensemble de propriétés) utilisée pour détecter si une entité a été modifiée depuis qu'elle a été chargée. Il peut s'agir d'une valeur de version, d'un timestamp, ou même d'un hachage de l'ensemble des données.

### Configuration d'un token de concurrence avec les attributs

```csharp
public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }

    [Timestamp]  // Marque cette propriété comme token de concurrence
    public byte[] VersionRow { get; set; }
}
```

### Configuration avec la Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Livre>()
        .Property(l => l.VersionRow)
        .IsRowVersion();  // Utilise un timestamp SQL Server

    // OU utiliser une propriété normale comme token de concurrence
    modelBuilder.Entity<Auteur>()
        .Property(a => a.DerniereModification)
        .IsConcurrencyToken();
}
```

### Utilisation d'un mécanisme de version personnalisé

Vous pouvez implémenter votre propre logique de version :

```csharp
public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }

    // Une propriété simple comme token de concurrence
    public Guid Version { get; set; }

    // Mettre à jour le token lors de chaque modification
    public void PrepareForUpdate()
    {
        Version = Guid.NewGuid();
    }
}

// Dans votre contexte ou service
public void UpdateBook(Livre livre)
{
    livre.PrepareForUpdate();
    context.SaveChanges();
}
```

### Tokens de concurrence sur plusieurs colonnes

Vous pouvez également utiliser plusieurs propriétés comme token de concurrence :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Commande>()
        .Property(c => c.Statut).IsConcurrencyToken();

    modelBuilder.Entity<Commande>()
        .Property(c => c.ModifiePar).IsConcurrencyToken();

    modelBuilder.Entity<Commande>()
        .Property(c => c.DerniereModification).IsConcurrencyToken();
}
```

## 10.8.4. Stratégies pessimistes et optimistes

Il existe deux approches principales pour gérer la concurrence : pessimiste et optimiste. EF Core utilise principalement la concurrence optimiste, mais vous pouvez implémenter des stratégies pessimistes si nécessaire.

### Concurrence optimiste

La concurrence optimiste permet à plusieurs utilisateurs de récupérer les mêmes données, puis vérifie s'il y a eu des conflits au moment de l'enregistrement. C'est ce que nous avons vu jusqu'à présent avec les tokens de concurrence.

**Avantages** :
- Meilleure évolutivité pour les applications à forte charge
- Pas de verrouillage prolongé des ressources
- Fonctionne bien quand les conflits sont rares

**Inconvénients** :
- Les conflits ne sont détectés qu'au moment de la sauvegarde
- Nécessite de gérer les erreurs de concurrence

```csharp
// Exemple de concurrence optimiste (comme vu précédemment)
try
{
    context.SaveChanges();
}
catch (DbUpdateConcurrencyException ex)
{
    // Résoudre le conflit
}
```

### Concurrence pessimiste

La concurrence pessimiste empêche les conflits en verrouillant les données pendant qu'un utilisateur les modifie, empêchant ainsi d'autres utilisateurs de les modifier simultanément.

EF Core ne fournit pas de mécanisme natif pour les verrous pessimistes, mais vous pouvez les implémenter avec des requêtes SQL directes ou des procédures stockées.

**Avantages** :
- Évite les conflits avant qu'ils ne se produisent
- Plus simple à gérer car pas d'erreur de concurrence

**Inconvénients** :
- Évolutivité limitée
- Risque de blocage (deadlocks)
- Les utilisateurs peuvent être bloqués dans l'attente

#### Implémentation de verrous pessimistes avec SQL Server

```csharp
// Verrouillage pessimiste avec UPDLOCK
using (var context = new LibraryContext())
{
    using (var transaction = context.Database.BeginTransaction())
    {
        try
        {
            // Obtenir et verrouiller la ligne
            var livre = context.Livres
                .FromSqlRaw("SELECT * FROM Livres WITH (UPDLOCK) WHERE Id = {0}", livreId)
                .FirstOrDefault();

            if (livre != null)
            {
                // Modifier l'entité
                livre.Titre = "Nouveau titre avec verrouillage";

                // Enregistrer les modifications
                context.SaveChanges();

                // Valider la transaction
                transaction.Commit();
            }
        }
        catch (Exception)
        {
            transaction.Rollback();
            throw;
        }
    }
}
```

#### Simulation de verrouillage pessimiste avec une table de verrous

```csharp
public class VerrouEntite
{
    public string TypeEntite { get; set; }
    public int IdEntite { get; set; }
    public string UtilisateurId { get; set; }
    public DateTime DateVerrou { get; set; }

    // Clé primaire composite
    public class VerrouEntiteKey
    {
        public string TypeEntite { get; set; }
        public int IdEntite { get; set; }
    }
}

// Dans le code d'application
public bool EssayerVerrouillerEntite(string typeEntite, int idEntite, string utilisateurId)
{
    try
    {
        context.VerrousEntites.Add(new VerrouEntite
        {
            TypeEntite = typeEntite,
            IdEntite = idEntite,
            UtilisateurId = utilisateurId,
            DateVerrou = DateTime.UtcNow
        });

        context.SaveChanges();
        return true; // Verrouillage réussi
    }
    catch (DbUpdateException)
    {
        return false; // L'entité est déjà verrouillée
    }
}

public void RelacherVerrou(string typeEntite, int idEntite, string utilisateurId)
{
    var verrou = context.VerrousEntites.Find(typeEntite, idEntite);

    if (verrou != null && verrou.UtilisateurId == utilisateurId)
    {
        context.VerrousEntites.Remove(verrou);
        context.SaveChanges();
    }
}
```

### Choisir la bonne stratégie

Le choix entre concurrence optimiste et pessimiste dépend de votre cas d'utilisation :

- **Choisissez la concurrence optimiste quand** :
  - Les conflits sont rares
  - La performance et l'évolutivité sont prioritaires
  - Les utilisateurs peuvent résoudre facilement les conflits

- **Choisissez la concurrence pessimiste quand** :
  - Les conflits seraient coûteux ou difficiles à résoudre
  - Vous avez besoin de garantir qu'une opération peut s'achever sans conflit
  - Le nombre d'utilisateurs simultanés est relativement faible

### Approche hybride

Dans de nombreuses applications, une approche hybride fonctionne bien :

```csharp
public async Task<Livre> ObtenirLivrePourModification(int livreId, string utilisateurId)
{
    // D'abord, essayer d'acquérir un verrou "doux"
    var verrou = await context.VerrousEditeurs.FindAsync(livreId);

    if (verrou != null && verrou.UtilisateurId != utilisateurId)
    {
        // L'entité est en cours d'édition par quelqu'un d'autre
        // Informer l'utilisateur, mais autoriser quand même l'édition
        // (concurrence optimiste comme filet de sécurité)
    }
    else
    {
        // Créer ou mettre à jour le verrou
        if (verrou == null)
        {
            context.VerrousEditeurs.Add(new VerrouEditeur
            {
                EntiteId = livreId,
                TypeEntite = "Livre",
                UtilisateurId = utilisateurId,
                DateVerrou = DateTime.UtcNow
            });
        }
        else
        {
            verrou.UtilisateurId = utilisateurId;
            verrou.DateVerrou = DateTime.UtcNow;
        }
        await context.SaveChangesAsync();
    }

    // Retourner l'entité avec la détection de concurrence optimiste activée
    return await context.Livres.FindAsync(livreId);
}
```

---

La gestion des transactions et de la concurrence est essentielle pour les applications de base de données robustes. Entity Framework Core offre une gamme d'outils pour implémenter la stratégie qui correspond le mieux à vos besoins spécifiques. Que vous choisissiez une approche optimiste, pessimiste ou hybride, l'important est de comprendre les compromis et les implications pour la cohérence des données et les performances de votre application.

⏭️ 11. [Intégration avec MySQL/MariaDB](/11-integration-avec-mysql-mariadb/README.md)
