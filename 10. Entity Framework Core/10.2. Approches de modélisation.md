# 10.2. Approches de modélisation avec Entity Framework Core

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Entity Framework Core offre plusieurs approches pour modéliser votre application et sa base de données. Chaque approche a ses avantages et ses cas d'usage spécifiques. Dans cette section, nous allons explorer ces différentes approches pour vous aider à choisir celle qui convient le mieux à votre projet.

## 10.2.1. Code First

L'approche **Code First** part du principe que votre code C# est la source de vérité. Vous commencez par définir vos classes d'entités en C#, puis Entity Framework Core se charge de créer la base de données qui correspond à ces classes.

### Comment ça fonctionne

1. Vous écrivez vos classes qui représentent les entités de votre domaine
2. Vous configurez les relations et les propriétés avec des attributs ou la Fluent API
3. EF Core génère la base de données à partir de vos classes

### Exemple simple

```csharp
// Définition des entités
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
    public int CategorieId { get; set; }

    // Propriété de navigation
    public Categorie Categorie { get; set; }
}

public class Categorie
{
    public int Id { get; set; }
    public string Nom { get; set; }

    // Collection de navigation
    public List<Produit> Produits { get; set; }
}

// Contexte de base de données
public class BoutiqueContext : DbContext
{
    public DbSet<Produit> Produits { get; set; }
    public DbSet<Categorie> Categories { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=.;Database=Boutique;Trusted_Connection=True;");
    }

    // Configuration supplémentaire avec la Fluent API
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Produit>()
            .Property(p => p.Nom)
            .IsRequired()
            .HasMaxLength(100);

        modelBuilder.Entity<Produit>()
            .HasOne(p => p.Categorie)
            .WithMany(c => c.Produits)
            .HasForeignKey(p => p.CategorieId);
    }
}
```

### Création de la base de données

Pour créer la base de données à partir de ce modèle, vous pouvez utiliser les migrations:

```csharp
// Dans la console du Package Manager
Add-Migration InitialCreate
Update-Database

// Ou avec la CLI .NET
dotnet ef migrations add InitialCreate
dotnet ef database update
```

### Avantages du Code First

- **Contrôle total** sur vos modèles de données
- **Intégration parfaite** avec le contrôle de version (Git, etc.)
- **Approche centrée sur le domaine** qui favorise le Domain-Driven Design
- **Migrations** pour gérer facilement l'évolution de votre schéma

### Inconvénients

- Peut être moins adapté si vous travaillez avec une base de données existante complexe
- Nécessite une bonne compréhension des concepts ORM

## 10.2.2. Database First

L'approche **Database First** part d'une base de données existante. Cette approche est utile lorsque:
- Vous avez déjà une base de données en production
- La structure de la base de données est définie par des DBA (Database Administrators)
- Vous devez intégrer votre application à un système existant

### Comment ça fonctionne avec EF Core

Entity Framework Core utilise un processus appelé "reverse engineering" (ingénierie inverse) pour créer des classes C# à partir d'une base de données existante.

> **Note importante**: La génération de modèles visuels comme avec l'ancien EF6 (fichiers .edmx) n'est pas disponible dans EF Core. À la place, on utilise des outils en ligne de commande.

### Exemple de génération de modèle

```bash
# Avec la CLI .NET
dotnet ef dbcontext scaffold "Server=.;Database=Boutique;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

Ou dans la Console du Package Manager:

```powershell
Scaffold-DbContext "Server=.;Database=Boutique;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```

Ces commandes vont:
1. Se connecter à la base de données spécifiée
2. Analyser son schéma (tables, colonnes, relations)
3. Générer des classes C# pour chaque table
4. Générer une classe DbContext configurée pour cette base de données

### Classes générées (exemple)

```csharp
public partial class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
    public int CategorieId { get; set; }

    public virtual Categorie Categorie { get; set; }
}

public partial class Categorie
{
    public Categorie()
    {
        Produits = new HashSet<Produit>();
    }

    public int Id { get; set; }
    public string Nom { get; set; }

    public virtual ICollection<Produit> Produits { get; set; }
}

public partial class BoutiqueContext : DbContext
{
    public BoutiqueContext()
    {
    }

    public BoutiqueContext(DbContextOptions<BoutiqueContext> options)
        : base(options)
    {
    }

    public virtual DbSet<Produit> Produits { get; set; }
    public virtual DbSet<Categorie> Categories { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseSqlServer("Server=.;Database=Boutique;Trusted_Connection=True;");
        }
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configuration des relations, clés, etc.
        modelBuilder.Entity<Produit>(entity =>
        {
            entity.HasOne(d => d.Categorie)
                .WithMany(p => p.Produits)
                .HasForeignKey(d => d.CategorieId);
        });
    }
}
```

### Avantages du Database First

- Idéal pour travailler avec des bases de données **existantes**
- Vous évite de dupliquer la conception de la base de données
- Génère un modèle complet qui reflète fidèlement votre schéma
- Les classes générées sont partielles (`partial`), ce qui vous permet de les étendre

### Inconvénients

- Moins de contrôle sur vos modèles côté code
- Difficultés potentielles si vous souhaitez modifier la base de données par la suite
- La régénération du code peut écraser vos personnalisations

## 10.2.3. Model First

L'approche **Model First** consiste à créer un modèle conceptuel à l'aide d'outils visuels, puis à générer à la fois la base de données et les classes à partir de ce modèle. Cette approche était disponible dans Entity Framework 6 avec le designer EDMX.

**Important**: Entity Framework Core n'a pas d'équivalent direct à l'approche Model First avec un designer visuel comme EF6. Les outils visuels pour EF Core sont plus limités et ne permettent pas de créer un modèle complet visuellement, puis de générer le code et la base de données.

### Alternatives pour EF Core

Pour les utilisateurs habitués à l'approche Model First, voici les alternatives avec EF Core:

1. **Code First avec visualisation**:
   - Utilisez l'approche Code First
   - Utilisez des outils de visualisation de modèle tiers comme EntityFrameworkCore.Diagrams ou EF Core Power Tools pour visualiser votre modèle

2. **Outils de conception UML**:
   - Concevez votre modèle dans un outil UML
   - Générez des classes C# à partir de vos diagrammes
   - Utilisez ces classes avec l'approche Code First

### Exemple d'utilisation d'EF Core Power Tools

1. Installez l'extension "EF Core Power Tools" dans Visual Studio
2. Cliquez droit sur votre projet → EF Core Power Tools → Add DbContext Model Diagram
3. Visualisez et modifiez votre modèle existant

![Exemple de diagramme EF Core Power Tools](https://raw.githubusercontent.com/ErikEJ/EFCorePowerTools/master/img/ef-power-tools-diagram.png)

## 10.2.4. Reverse Engineering

Le **Reverse Engineering** est le processus qui consiste à générer des classes d'entités et un contexte à partir d'une base de données existante. C'est essentiellement la technique utilisée par l'approche Database First dans EF Core.

### Fonctionnalités avancées de Reverse Engineering

En plus de la génération basique vue dans la section Database First, vous pouvez personnaliser le processus:

#### 1. Sélectionner des tables spécifiques

```bash
dotnet ef dbcontext scaffold "Server=.;Database=Boutique;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models -t Produits -t Categories
```

#### 2. Utiliser les types de données .NET appropriés

```bash
dotnet ef dbcontext scaffold "..." Microsoft.EntityFrameworkCore.SqlServer --use-database-names
```

#### 3. Générer des interfaces DbContext pour les tests

```bash
dotnet ef dbcontext scaffold "..." Microsoft.EntityFrameworkCore.SqlServer --context-dir Data --context BoutiqueContext --context-namespace MonApplication.Data
```

#### 4. Personnaliser la génération avec des templates T4

Pour les utilisateurs avancés, il est possible de personnaliser le processus de génération en utilisant des templates T4 avec des outils comme:
- EF Core Tools
- EF Core Power Tools (extension Visual Studio)
- EF Core Reverse POCO Generator

### Stratégies pour maintenir le modèle à jour

1. **Régénération sélective**:
   - Générez dans un dossier temporaire
   - Comparez et fusionnez les changements manuellement

2. **Script de génération personnalisé**:
   - Créez un script qui génère le modèle
   - Appliquez des modifications post-génération avec des outils comme PowerShell

3. **Génération partielle**:
   - Utilisez la fonctionnalité de classes partielles
   - Placez vos personnalisations dans des fichiers séparés

```csharp
// Fichier généré: Produit.cs
public partial class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    // Propriétés générées...
}

// Votre fichier personnalisé: Produit.custom.cs
public partial class Produit
{
    // Méthodes et propriétés personnalisées
    public decimal PrixTTC => Prix * 1.2m;

    public bool EstEnPromotion()
    {
        // Votre logique métier personnalisée
        return Prix < 50;
    }
}
```

### Comparaison des approches

| Approche | Points forts | Points faibles | Cas d'usage idéal |
|----------|--------------|----------------|-------------------|
| **Code First** | Contrôle total, intégration Git, migrations | Courbe d'apprentissage, moins adapté aux bases existantes | Nouveaux projets, DDD |
| **Database First** | Rapide avec bases existantes, fidèle au schéma | Moins de contrôle, personnalisations difficiles | Bases de données existantes, intégration |
| **Model First** | N/A dans EF Core | N/A dans EF Core | Utilisez Code First + outils de visualisation |
| **Reverse Engineering** | Flexible, personnalisable | Nécessite des scripts pour maintenir | Bases complexes avec personnalisation |

## Conclusion

Pour les débutants, nous recommandons généralement:

- **Nouveau projet sans base de données existante**: Utilisez l'approche **Code First**
- **Projet avec base de données existante**: Utilisez l'approche **Database First** avec Reverse Engineering

Quelle que soit l'approche choisie, Entity Framework Core offre une grande flexibilité et peut s'adapter à différents scénarios de développement. N'hésitez pas à mixer les approches selon vos besoins spécifiques!

⏭️
