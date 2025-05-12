# 11.3. CRUD avec MySQL/MariaDB

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les opérations CRUD (Create, Read, Update, Delete) constituent la base de l'interaction avec toute base de données. Dans cette section, nous allons explorer comment effectuer ces opérations avec MySQL/MariaDB en utilisant Entity Framework Core, tout en tenant compte des spécificités de ces systèmes de gestion de base de données.

## 11.3.1. Considérations spécifiques

MySQL et MariaDB présentent certaines particularités qui peuvent affecter la façon dont vous implémentez vos opérations CRUD. Voici les points importants à considérer.

### Auto-incrémentation et identifiants

Par défaut, MySQL/MariaDB utilise le type `AUTO_INCREMENT` pour les clés primaires numériques. Entity Framework Core le détecte automatiquement, mais vous pouvez aussi le configurer explicitement :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Produit>()
        .Property(p => p.Id)
        .ValueGeneratedOnAdd();
}
```

### Sensibilité à la casse

MySQL peut être configuré pour être sensible ou non à la casse selon le système d'exploitation et les paramètres du serveur. Sur Linux, les noms de tables sont généralement sensibles à la casse, tandis que sur Windows, ils ne le sont pas. Pour éviter les problèmes :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Spécifier explicitement le nom de la table avec la casse exacte
    modelBuilder.Entity<Produit>()
        .ToTable("Produits");
}
```

### Moteurs de stockage

MySQL/MariaDB supporte plusieurs moteurs de stockage, dont InnoDB (par défaut) et MyISAM. InnoDB est recommandé car il supporte les transactions et les contraintes de clé étrangère. EF Core utilise InnoDB par défaut, mais vous pouvez spécifier le moteur :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Produit>()
        .ToTable("Produits", t => t.HasComment("Moteur: InnoDB"));
}
```

Pour un contrôle plus précis, vous pouvez utiliser une migration et modifier le SQL généré :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("ALTER TABLE Produits ENGINE = InnoDB;");
}
```

### Collations et jeux de caractères

MySQL/MariaDB utilise des collations pour définir comment les chaînes sont comparées et triées. La définition d'une collation appropriée est importante pour le support des caractères internationaux :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configuration au niveau de la propriété
    modelBuilder.Entity<Produit>()
        .Property(p => p.Nom)
        .UseCollation("utf8mb4_unicode_ci");
}
```

Pour configurer la collation au niveau de la base de données dans une migration :

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("ALTER DATABASE ma_base CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;");
}
```

## 11.3.2. Requêtes paramétrées

Les requêtes paramétrées sont essentielles pour prévenir les injections SQL. Entity Framework Core utilise automatiquement des requêtes paramétrées, mais il est important de comprendre comment elles fonctionnent.

### Requêtes paramétrées avec LINQ

Lorsque vous utilisez LINQ avec EF Core, les paramètres sont automatiquement gérés de manière sécurisée :

```csharp
// Recherche simple - les paramètres sont gérés de manière sécurisée
public Produit ObtenirProduitParNom(string nom)
{
    return _context.Produits
        .Where(p => p.Nom == nom) // Le nom est utilisé comme paramètre
        .FirstOrDefault();
}
```

### Utilisation de FromSqlRaw et FromSqlInterpolated

Si vous avez besoin d'exécuter du SQL brut, utilisez `FromSqlRaw` avec des paramètres ou `FromSqlInterpolated` :

```csharp
// Méthode 1 : FromSqlRaw avec paramètres
public List<Produit> RechercheProduits(string motCle)
{
    return _context.Produits
        .FromSqlRaw("SELECT * FROM Produits WHERE Nom LIKE {0}", "%" + motCle + "%")
        .ToList();
}

// Méthode 2 : FromSqlInterpolated (plus lisible)
public List<Produit> RechercheProduits2(string motCle)
{
    return _context.Produits
        .FromSqlInterpolated($"SELECT * FROM Produits WHERE Nom LIKE {$"%{motCle}%"}")
        .ToList();
}
```

> **Attention :** N'utilisez jamais la concaténation de chaînes pour construire des requêtes SQL, car cela expose votre application aux injections SQL.

### Exécution de procédures stockées

MySQL/MariaDB prend en charge les procédures stockées, que vous pouvez appeler avec EF Core :

```csharp
// Appel d'une procédure stockée qui retourne des entités
public List<Produit> ObtenirProduitsParCategorie(int categorieId)
{
    return _context.Produits
        .FromSqlRaw("CALL sp_produits_par_categorie({0})", categorieId)
        .ToList();
}

// Exécution d'une procédure stockée qui ne retourne pas de résultats
public void MettreAJourStock(int produitId, int quantite)
{
    _context.Database.ExecuteSqlRaw(
        "CALL sp_mettre_a_jour_stock({0}, {1})",
        produitId,
        quantite);
}
```

## 11.3.3. Types de données et mappings

Le mapping correct entre les types de données MySQL/MariaDB et les types C# est essentiel pour éviter des problèmes de conversion ou de perte de précision.

### Table de correspondance des types

| Type MySQL/MariaDB | Type C# | Notes |
|-------------------|---------|-------|
| TINYINT(1) | bool | Interprété comme un booléen |
| TINYINT, SMALLINT | byte, short | Selon la taille |
| INT, INTEGER | int | Type entier standard |
| BIGINT | long | Pour les grands nombres entiers |
| FLOAT | float | Précision simple |
| DOUBLE, REAL | double | Précision double |
| DECIMAL | decimal | Pour les valeurs monétaires |
| CHAR, VARCHAR | string | Chaînes de caractères |
| TEXT, LONGTEXT | string | Textes longs |
| DATETIME | DateTime | Date et heure |
| DATE | DateOnly (NET 6+) | Date uniquement |
| TIME | TimeOnly (NET 6+) | Heure uniquement |
| TIMESTAMP | DateTime | Horodatage UTC |
| BLOB, LONGBLOB | byte[] | Données binaires |
| ENUM | enum ou string | Peut être mappé aux deux |
| JSON | string ou class | Peut être stocké comme chaîne ou converti |

### Configuration du mapping des types

Pour configurer explicitement un mapping de type :

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configuration du type de colonne
    modelBuilder.Entity<Produit>()
        .Property(p => p.Prix)
        .HasColumnType("decimal(10,2)");

    // Pour un champ ENUM MySQL
    modelBuilder.Entity<Produit>()
        .Property(p => p.Statut)
        .HasColumnType("ENUM('Active', 'Inactive', 'Discontinued')");
}
```

### Gestion des types JSON

MySQL 5.7+ et MariaDB 10.2+ prennent en charge le type de données JSON. Vous pouvez le mapper de plusieurs façons :

```csharp
// Stockage sous forme de chaîne JSON
public class Produit
{
    public int Id { get; set; }
    public string Attributs { get; set; } // JSON stocké sous forme de chaîne
}

// Ou avec conversion automatique via EF Core 5.0+
public class Produit
{
    public int Id { get; set; }
    public Dictionary<string, object> Attributs { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Produit>()
        .Property(p => p.Attributs)
        .HasColumnType("json")
        .HasConversion(
            v => JsonSerializer.Serialize(v, null),
            v => JsonSerializer.Deserialize<Dictionary<string, object>>(v, null));
}
```

### Gestion des dates et heures

MySQL/MariaDB stocke les dates et heures différemment de SQL Server. Points importants à retenir :

```csharp
// Configuration pour les dates
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Préciser le format de date exactement
    modelBuilder.Entity<Commande>()
        .Property(c => c.DateCommande)
        .HasColumnType("datetime");

    // Pour les horodatages automatiques
    modelBuilder.Entity<Commande>()
        .Property(c => c.DateCreation)
        .HasColumnType("timestamp")
        .ValueGeneratedOnAdd();

    modelBuilder.Entity<Commande>()
        .Property(c => c.DateModification)
        .HasColumnType("timestamp")
        .ValueGeneratedOnAddOrUpdate();
}
```

## 11.3.4. Batch operations

Les opérations par lots (batch operations) permettent d'exécuter plusieurs opérations en une seule requête, ce qui améliore considérablement les performances.

### Insertions par lots

EF Core insère les entités une par une par défaut. Pour des insertions en masse, vous pouvez :

1. **Utiliser AddRange pour insérer plusieurs entités** :

```csharp
// Ajouter plusieurs produits en une seule fois
public void AjouterProduits(List<Produit> produits)
{
    _context.Produits.AddRange(produits);
    _context.SaveChanges();
}
```

Ceci exécutera quand même plusieurs requêtes SQL mais dans une seule transaction.

2. **Utiliser une extension tierce pour de vraies opérations par lots** :

```csharp
// Installation du package : dotnet add package EFCore.BulkExtensions
using EFCore.BulkExtensions;

// Insertion en masse
public void InsererEnMasse(List<Produit> produits)
{
    _context.BulkInsert(produits);
}

// Mise à jour en masse
public void MettreAJourEnMasse(List<Produit> produits)
{
    _context.BulkUpdate(produits);
}

// Suppression en masse
public void SupprimerEnMasse(List<Produit> produits)
{
    _context.BulkDelete(produits);
}
```

### Mises à jour par lots sans charger les entités

Vous pouvez effectuer des mises à jour en masse sans charger les entités en mémoire :

```csharp
// Mise à jour basée sur une condition
public void MettreAJourPrixParCategorie(int categorieId, decimal augmentation)
{
    _context.Produits
        .Where(p => p.CategorieId == categorieId)
        .ExecuteUpdate(s => s.SetProperty(
            p => p.Prix,
            p => p.Prix * (1 + augmentation / 100)
        ));
}
```

Cette approche utilise EF Core 7+ `ExecuteUpdate`.

### Suppressions par lots

De même, vous pouvez effectuer des suppressions en masse :

```csharp
// Suppression basée sur une condition
public void SupprimerProduitsObsoletes(DateTime dateLimite)
{
    _context.Produits
        .Where(p => p.DerniereVente < dateLimite)
        .ExecuteDelete();
}
```

Cette approche utilise EF Core 7+ `ExecuteDelete`.

### Transactions explicites pour les opérations par lots

Pour les opérations complexes impliquant plusieurs entités, utilisez des transactions explicites :

```csharp
public void TraiterCommande(Commande commande, List<LigneCommande> lignes)
{
    using var transaction = _context.Database.BeginTransaction();
    try
    {
        // Ajout de la commande
        _context.Commandes.Add(commande);
        _context.SaveChanges();

        // Mise à jour de l'ID de commande dans les lignes
        foreach (var ligne in lignes)
        {
            ligne.CommandeId = commande.Id;
        }

        // Ajout des lignes de commande
        _context.LignesCommandes.AddRange(lignes);

        // Mise à jour du stock
        foreach (var ligne in lignes)
        {
            var produit = _context.Produits.Find(ligne.ProduitId);
            produit.Stock -= ligne.Quantite;

            if (produit.Stock < 0)
            {
                throw new InvalidOperationException("Stock insuffisant");
            }
        }

        _context.SaveChanges();
        transaction.Commit();
    }
    catch
    {
        transaction.Rollback();
        throw;
    }
}
```

### Optimiser les performances des opérations par lots

Pour optimiser davantage les opérations par lots avec MySQL/MariaDB :

1. **Désactiver le suivi des entités** pour les opérations en lecture seule :

```csharp
var produits = _context.Produits
    .AsNoTracking()
    .ToList();
```

2. **Utiliser des commandes SQL directes** pour les opérations très volumineuses :

```csharp
public void AjusterTousPrix(decimal pourcentage)
{
    _context.Database.ExecuteSqlRaw(
        "UPDATE Produits SET Prix = Prix * (1 + {0}/100)",
        pourcentage);
}
```

3. **Ajuster la taille des lots** dans les extensions tierces :

```csharp
_context.BulkInsert(listeProduits, new BulkConfig { BatchSize = 1000 });
```

## Bonnes pratiques pour les opérations CRUD avec MySQL/MariaDB

1. **Utilisez toujours des requêtes paramétrées** pour éviter les injections SQL
2. **Choisissez les bons types de données** pour optimiser le stockage et les performances
3. **Utilisez des transactions** pour les opérations interdépendantes
4. **Préférez les opérations par lots** pour les insertions, mises à jour et suppressions massives
5. **Configurez correctement les index** pour accélérer les requêtes
6. **Limitez la quantité de données chargées** en mémoire en utilisant des projections et des filtres
7. **Testez les performances** avec des volumes de données réalistes

En suivant ces conseils, vous pourrez créer des applications .NET performantes et fiables qui interagissent efficacement avec vos bases de données MySQL ou MariaDB.

⏭️
