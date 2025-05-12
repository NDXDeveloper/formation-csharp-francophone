# 10.6. Relations entre entités

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les bases de données relationnelles tirent leur puissance des relations entre les tables. Entity Framework Core vous permet de modéliser facilement ces relations entre vos classes d'entités. Comprendre comment configurer et utiliser ces relations est essentiel pour tirer pleinement parti d'EF Core.

## 10.6.1. One-to-One (Un-à-Un)

Une relation un-à-un signifie qu'une entité est associée à une seule autre entité, et vice versa. Par exemple, un utilisateur peut avoir un seul profil, et ce profil appartient à un seul utilisateur.

### Exemple de relation Un-à-Un

Considérons un exemple avec un `Utilisateur` et son `ProfilUtilisateur` :

```csharp
public class Utilisateur
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Email { get; set; }

    // Propriété de navigation vers ProfilUtilisateur
    public ProfilUtilisateur Profil { get; set; }
}

public class ProfilUtilisateur
{
    public int Id { get; set; }
    public string Bio { get; set; }
    public DateTime DateNaissance { get; set; }
    public string PhotoUrl { get; set; }

    // Clé étrangère
    public int UtilisateurId { get; set; }

    // Propriété de navigation vers Utilisateur
    public Utilisateur Utilisateur { get; set; }
}
```

### Configuration avec les conventions

Par défaut, EF Core identifiera cette relation en se basant sur les conventions de nommage :
- La propriété `UtilisateurId` dans `ProfilUtilisateur` correspond au `Id` dans `Utilisateur`
- Il existe des propriétés de navigation dans les deux classes

### Configuration avec les Data Annotations

Vous pouvez rendre la relation plus explicite avec des annotations :

```csharp
public class ProfilUtilisateur
{
    public int Id { get; set; }

    // Autres propriétés...

    [Key] // Indique que cette propriété est à la fois clé primaire et clé étrangère
    public int UtilisateurId { get; set; }

    [Required]
    [ForeignKey("UtilisateurId")]
    public Utilisateur Utilisateur { get; set; }
}
```

### Utilisation d'une relation Un-à-Un

```csharp
// Création d'un utilisateur avec son profil
var utilisateur = new Utilisateur
{
    Nom = "Marie Dupont",
    Email = "marie@exemple.fr",
    Profil = new ProfilUtilisateur
    {
        Bio = "Passionnée de développement",
        DateNaissance = new DateTime(1990, 5, 15),
        PhotoUrl = "photos/marie.jpg"
    }
};

// Ajout à la base de données
context.Utilisateurs.Add(utilisateur);
context.SaveChanges();

// Requête avec chargement du profil
var utilisateurAvecProfil = context.Utilisateurs
    .Include(u => u.Profil)
    .FirstOrDefault(u => u.Id == 1);

Console.WriteLine($"Bio: {utilisateurAvecProfil.Profil.Bio}");
```

## 10.6.2. One-to-Many (Un-à-Plusieurs)

Une relation un-à-plusieurs est l'une des plus courantes. Elle signifie qu'une entité peut être associée à plusieurs autres entités. Par exemple, un auteur peut avoir écrit plusieurs livres.

### Exemple de relation Un-à-Plusieurs

```csharp
public class Auteur
{
    public int Id { get; set; }
    public string Nom { get; set; }

    // Navigation vers la collection de livres
    public List<Livre> Livres { get; set; }
}

public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }
    public DateTime DatePublication { get; set; }

    // Clé étrangère
    public int AuteurId { get; set; }

    // Navigation vers l'auteur
    public Auteur Auteur { get; set; }
}
```

### Configuration avec les conventions

EF Core va automatiquement reconnaître cette relation car :
- `Livre` a une propriété `AuteurId` qui correspond à la clé primaire de `Auteur`
- `Auteur` a une collection `Livres`
- `Livre` a une propriété de navigation `Auteur`

### Utilisation d'une relation Un-à-Plusieurs

```csharp
// Création d'un auteur avec plusieurs livres
var auteur = new Auteur
{
    Nom = "Victor Hugo",
    Livres = new List<Livre>
    {
        new Livre { Titre = "Les Misérables", DatePublication = new DateTime(1862, 1, 1) },
        new Livre { Titre = "Notre-Dame de Paris", DatePublication = new DateTime(1831, 1, 1) }
    }
};

// Ajout à la base de données
context.Auteurs.Add(auteur);
context.SaveChanges();

// Requête avec chargement des livres
var auteurAvecLivres = context.Auteurs
    .Include(a => a.Livres)
    .FirstOrDefault(a => a.Id == 1);

Console.WriteLine($"Auteur: {auteurAvecLivres.Nom}");
Console.WriteLine($"Nombre de livres: {auteurAvecLivres.Livres.Count}");

// Ajouter un nouveau livre à un auteur existant
var nouveauLivre = new Livre
{
    Titre = "Les Travailleurs de la mer",
    DatePublication = new DateTime(1866, 1, 1),
    AuteurId = auteurAvecLivres.Id  // Ou simplement: Auteur = auteurAvecLivres
};

context.Livres.Add(nouveauLivre);
// Ou bien: auteurAvecLivres.Livres.Add(nouveauLivre);
context.SaveChanges();
```

## 10.6.3. Many-to-Many (Plusieurs-à-Plusieurs)

Une relation plusieurs-à-plusieurs signifie que plusieurs entités d'un type peuvent être associées à plusieurs entités d'un autre type. Par exemple, un livre peut avoir plusieurs catégories, et une catégorie peut contenir plusieurs livres.

### Avec Entity Framework Core 5+ (Relation directe)

À partir d'EF Core 5.0, vous pouvez modéliser une relation plusieurs-à-plusieurs directement :

```csharp
public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }

    // Relation plusieurs-à-plusieurs
    public List<Categorie> Categories { get; set; }
}

public class Categorie
{
    public int Id { get; set; }
    public string Nom { get; set; }

    // Relation plusieurs-à-plusieurs
    public List<Livre> Livres { get; set; }
}
```

EF Core générera automatiquement une table de jointure pour cette relation.

### Avec Entity Framework Core 3.x et versions antérieures (Entité de jointure explicite)

Pour les versions plus anciennes d'EF Core, vous devez définir une entité de jointure explicite :

```csharp
public class Livre
{
    public int Id { get; set; }
    public string Titre { get; set; }

    public List<LivreCategorie> LivreCategories { get; set; }
}

public class Categorie
{
    public int Id { get; set; }
    public string Nom { get; set; }

    public List<LivreCategorie> LivreCategories { get; set; }
}

public class LivreCategorie
{
    public int LivreId { get; set; }
    public Livre Livre { get; set; }

    public int CategorieId { get; set; }
    public Categorie Categorie { get; set; }
}
```

### Utilisation d'une relation Plusieurs-à-Plusieurs

```csharp
// Avec EF Core 5+ (relation directe)
var roman = new Categorie { Nom = "Roman" };
var fiction = new Categorie { Nom = "Fiction" };

var livre = new Livre
{
    Titre = "Les Misérables",
    Categories = new List<Categorie> { roman, fiction }
};

context.Livres.Add(livre);
context.SaveChanges();

// Requête avec chargement des catégories
var livreAvecCategories = context.Livres
    .Include(l => l.Categories)
    .FirstOrDefault(l => l.Id == 1);

foreach (var categorie in livreAvecCategories.Categories)
{
    Console.WriteLine($"Catégorie: {categorie.Nom}");
}

// Avec entité de jointure explicite (EF Core 3.x et antérieur)
var livre = new Livre { Titre = "Les Misérables" };
var roman = new Categorie { Nom = "Roman" };
var fiction = new Categorie { Nom = "Fiction" };

context.Livres.Add(livre);
context.Categories.AddRange(roman, fiction);
context.SaveChanges();

context.LivreCategories.AddRange(
    new LivreCategorie { LivreId = livre.Id, CategorieId = roman.Id },
    new LivreCategorie { LivreId = livre.Id, CategorieId = fiction.Id }
);
context.SaveChanges();
```

## 10.6.4. Self-referencing (Autoréférence)

Une relation autoréférencée est une relation où une entité fait référence à d'autres entités du même type. Par exemple, un employé peut avoir un manager qui est aussi un employé.

### Exemple de relation autoréférencée

```csharp
public class Employe
{
    public int Id { get; set; }
    public string Nom { get; set; }

    // Self-reference: un employé peut avoir un manager
    public int? ManagerId { get; set; }
    public Employe Manager { get; set; }

    // Collection des employés supervisés
    public List<Employe> Subordonnes { get; set; }
}
```

### Utilisation d'une relation autoréférencée

```csharp
// Création d'une structure hiérarchique
var directeur = new Employe { Nom = "Alice" };
context.Employes.Add(directeur);
context.SaveChanges();

var chefEquipe1 = new Employe { Nom = "Bob", ManagerId = directeur.Id };
var chefEquipe2 = new Employe { Nom = "Charlie", ManagerId = directeur.Id };
context.Employes.AddRange(chefEquipe1, chefEquipe2);
context.SaveChanges();

var employe1 = new Employe { Nom = "David", ManagerId = chefEquipe1.Id };
var employe2 = new Employe { Nom = "Eve", ManagerId = chefEquipe1.Id };
var employe3 = new Employe { Nom = "Frank", ManagerId = chefEquipe2.Id };
context.Employes.AddRange(employe1, employe2, employe3);
context.SaveChanges();

// Requête pour obtenir tous les employés d'un manager
var manager = context.Employes
    .Include(e => e.Subordonnes)
    .FirstOrDefault(e => e.Id == chefEquipe1.Id);

Console.WriteLine($"Manager: {manager.Nom}");
foreach (var subordonne in manager.Subordonnes)
{
    Console.WriteLine($"- Subordonné: {subordonne.Nom}");
}

// Requête pour obtenir la hiérarchie complète
var organigramme = context.Employes
    .Include(e => e.Subordonnes)
        .ThenInclude(e => e.Subordonnes)
    .FirstOrDefault(e => e.ManagerId == null); // Le directeur n'a pas de manager

Console.WriteLine($"Directeur: {organigramme.Nom}");
foreach (var chefEquipe in organigramme.Subordonnes)
{
    Console.WriteLine($"- Chef d'équipe: {chefEquipe.Nom}");
    foreach (var employe in chefEquipe.Subordonnes)
    {
        Console.WriteLine($"  - Employé: {employe.Nom}");
    }
}
```

## 10.6.5. Configuration des relations avec Fluent API

Bien que les conventions et les annotations puissent suffire pour des relations simples, la Fluent API offre un contrôle plus précis sur vos relations. Voici comment configurer chaque type de relation avec la Fluent API.

### Configuration One-to-One avec Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Utilisateur>()
        .HasOne(u => u.Profil)                // Un utilisateur a un profil
        .WithOne(p => p.Utilisateur)          // Un profil a un utilisateur
        .HasForeignKey<ProfilUtilisateur>(p => p.UtilisateurId); // Clé étrangère
}
```

### Configuration One-to-Many avec Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Livre>()
        .HasOne(l => l.Auteur)                // Un livre a un auteur
        .WithMany(a => a.Livres)              // Un auteur a plusieurs livres
        .HasForeignKey(l => l.AuteurId)       // Clé étrangère
        .OnDelete(DeleteBehavior.Restrict);   // Comportement de suppression
}
```

### Configuration Many-to-Many avec Fluent API

```csharp
// EF Core 5+ (table de jointure générée automatiquement)
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Livre>()
        .HasMany(l => l.Categories)           // Un livre a plusieurs catégories
        .WithMany(c => c.Livres)              // Une catégorie a plusieurs livres
        .UsingEntity(j => j.ToTable("LivreCategorie")); // Personnalisation de la table de jointure
}

// EF Core 3.x et antérieur (entité de jointure explicite)
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<LivreCategorie>()
        .HasKey(lc => new { lc.LivreId, lc.CategorieId }); // Clé primaire composite

    modelBuilder.Entity<LivreCategorie>()
        .HasOne(lc => lc.Livre)
        .WithMany(l => l.LivreCategories)
        .HasForeignKey(lc => lc.LivreId);

    modelBuilder.Entity<LivreCategorie>()
        .HasOne(lc => lc.Categorie)
        .WithMany(c => c.LivreCategories)
        .HasForeignKey(lc => lc.CategorieId);
}
```

### Configuration Self-referencing avec Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employe>()
        .HasOne(e => e.Manager)               // Un employé a un manager
        .WithMany(e => e.Subordonnes)         // Un manager a plusieurs subordonnés
        .HasForeignKey(e => e.ManagerId)      // Clé étrangère
        .IsRequired(false)                    // Optionnel (pour le directeur)
        .OnDelete(DeleteBehavior.Restrict);   // Éviter les suppressions en cascade
}
```

### Options avancées avec Fluent API

La Fluent API offre de nombreuses options supplémentaires pour configurer vos relations :

#### Configuration du comportement de suppression

```csharp
// Options disponibles: Cascade, Restrict, SetNull, NoAction
modelBuilder.Entity<Livre>()
    .HasOne(l => l.Auteur)
    .WithMany(a => a.Livres)
    .OnDelete(DeleteBehavior.Cascade); // Supprime les livres quand l'auteur est supprimé
```

#### Configuration du nom de la colonne de clé étrangère

```csharp
modelBuilder.Entity<Livre>()
    .HasOne(l => l.Auteur)
    .WithMany(a => a.Livres)
    .HasForeignKey(l => l.AuteurId)
    .HasConstraintName("FK_Livre_Auteur"); // Nom de la contrainte dans la base de données
```

#### Configuration des propriétés de navigation facultatives

```csharp
// Pour gérer le cas où vous n'avez pas de propriété de navigation dans une classe
modelBuilder.Entity<Commande>()
    .HasOne<Client>() // Pas de propriété de navigation => utiliser le type générique
    .WithMany(c => c.Commandes)
    .HasForeignKey(c => c.ClientId);
```

#### Configuration d'une relation avec clé alternative

```csharp
modelBuilder.Entity<Livre>()
    .HasOne(l => l.Editeur)
    .WithMany(e => e.LivresPublies)
    .HasForeignKey(l => l.CodeEditeur)  // Utilise une clé alternative au lieu de la clé primaire
    .HasPrincipalKey(e => e.Code);      // Spécifie la clé alternative
```

### Exemple complet de configuration d'une base de données de bibliothèque

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configuration Auteur - Livre (One-to-Many)
    modelBuilder.Entity<Livre>()
        .HasOne(l => l.Auteur)
        .WithMany(a => a.Livres)
        .HasForeignKey(l => l.AuteurId)
        .OnDelete(DeleteBehavior.Restrict);

    // Configuration Livre - Catégorie (Many-to-Many)
    modelBuilder.Entity<Livre>()
        .HasMany(l => l.Categories)
        .WithMany(c => c.Livres)
        .UsingEntity<LivreCategorie>(
            l => l.HasOne(lc => lc.Categorie).WithMany(c => c.LivreCategories),
            r => r.HasOne(lc => lc.Livre).WithMany(l => l.LivreCategories)
        );

    // Configuration Utilisateur - ProfilUtilisateur (One-to-One)
    modelBuilder.Entity<Utilisateur>()
        .HasOne(u => u.Profil)
        .WithOne(p => p.Utilisateur)
        .HasForeignKey<ProfilUtilisateur>(p => p.UtilisateurId);

    // Configuration Employe - Employe (Self-referencing)
    modelBuilder.Entity<Employe>()
        .HasOne(e => e.Manager)
        .WithMany(e => e.Subordonnes)
        .HasForeignKey(e => e.ManagerId)
        .IsRequired(false);
}
```

---

Les relations entre entités sont fondamentales dans tout système de base de données relationnelle. Entity Framework Core offre des moyens flexibles pour modéliser ces relations dans votre code, qu'il s'agisse de relations Un-à-Un, Un-à-Plusieurs, Plusieurs-à-Plusieurs ou autoréférencées. La Fluent API vous donne un contrôle précis sur ces relations, vous permettant de configurer tous les aspects selon vos besoins spécifiques.

En maîtrisant ces concepts, vous serez en mesure de concevoir des modèles de données robustes et expressifs qui reflètent fidèlement la logique métier de votre application.

⏭️ 10.7. [Chargement de données](/10-entity-framework-core/10-7-chargement-de-donnees.md)
