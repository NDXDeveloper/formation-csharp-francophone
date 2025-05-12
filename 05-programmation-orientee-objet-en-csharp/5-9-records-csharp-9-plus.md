# 5.9. Records (C# 9+)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les records sont une fonctionnalité introduite dans C# 9.0 (disponible avec .NET 5 et versions ultérieures), conçue pour simplifier la création de types de données immuables, principalement utilisés pour représenter des données simples. Ils offrent une syntaxe concise et élégante pour créer des classes avec égalité basée sur les valeurs, plutôt que sur les références.

> **Note**: Les exemples de cette section nécessitent .NET 5 ou supérieur (idéalement .NET 8 comme spécifié dans ce tutoriel).

## 5.9.1. Immutabilité et égalité basée sur les valeurs

Les records sont par défaut immuables et implémentent automatiquement l'égalité basée sur les valeurs, ce qui signifie que deux records sont considérés comme égaux si toutes leurs propriétés ont les mêmes valeurs.

```csharp
// Définition d'un record simple
public record Personne(string Nom, string Prenom, int Age);

// Utilisation de records
public class Program
{
    public static void Main()
    {
        // Création d'instances de records
        Personne personne1 = new Personne("Dupont", "Jean", 30);
        Personne personne2 = new Personne("Dupont", "Jean", 30);
        Personne personne3 = new Personne("Martin", "Sophie", 25);

        // Affichage formaté automatique
        Console.WriteLine(personne1); // Affiche: Personne { Nom = Dupont, Prenom = Jean, Age = 30 }

        // Égalité basée sur les valeurs
        Console.WriteLine(personne1 == personne2); // True - mêmes valeurs
        Console.WriteLine(personne1 == personne3); // False - valeurs différentes
        Console.WriteLine(ReferenceEquals(personne1, personne2)); // False - ce sont des objets différents

        // GetHashCode() est également basé sur les valeurs
        Console.WriteLine($"HashCode 1: {personne1.GetHashCode()}");
        Console.WriteLine($"HashCode 2: {personne2.GetHashCode()}");
        Console.WriteLine($"HashCode 3: {personne3.GetHashCode()}");
        // Les hash codes de personne1 et personne2 sont identiques
    }
}
```


### Comparaison avec les classes traditionnelles

Pour obtenir le même comportement avec une classe, il faudrait beaucoup plus de code :

```csharp
// Classe équivalente au record Personne - beaucoup plus verbeuse
public class PersonneClasse
{
    public string Nom { get; }
    public string Prenom { get; }
    public int Age { get; }

    public PersonneClasse(string nom, string prenom, int age)
    {
        Nom = nom;
        Prenom = prenom;
        Age = age;
    }

    public override bool Equals(object obj)
    {
        if (obj == null || GetType() != obj.GetType())
            return false;

        PersonneClasse other = (PersonneClasse)obj;
        return Nom == other.Nom &&
               Prenom == other.Prenom &&
               Age == other.Age;
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(Nom, Prenom, Age);
    }

    public override string ToString()
    {
        return $"PersonneClasse {{ Nom = {Nom}, Prenom = {Prenom}, Age = {Age} }}";
    }
}
```


### Records avec des propriétés mutables

Bien que l'immutabilité soit encouragée avec les records, il est possible de déclarer des propriétés mutables :

```csharp
// Record avec propriétés positionnelles et propriété mutable
public record Utilisateur(string Id, string NomUtilisateur)
{
    // Propriété mutable
    public int NombreConnexions { get; set; } = 0;

    // Propriété calculée
    public bool EstNouvelUtilisateur => NombreConnexions < 5;

    // Méthode
    public void Connecter()
    {
        NombreConnexions++;
        Console.WriteLine($"{NomUtilisateur} s'est connecté ({NombreConnexions} connexions)");
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        Utilisateur user1 = new Utilisateur("U123", "jean.dupont");
        Utilisateur user2 = new Utilisateur("U123", "jean.dupont");

        Console.WriteLine(user1 == user2); // True initialement

        user1.Connecter(); // Modifie NombreConnexions

        // Toujours égaux, car NombreConnexions n'est pas considéré
        Console.WriteLine(user1 == user2); // True car seules les propriétés positionnelles entrent en compte pour l'égalité

        Console.WriteLine($"User1: {user1}"); // NombreConnexions n'apparaît pas dans ToString() par défaut
    }
}
```


### Records avec une syntaxe de corps de classe

On peut aussi définir des records avec une syntaxe semblable à celle des classes traditionnelles :

```csharp
// Record avec syntaxe de corps de classe
public record Produit
{
    // Propriétés auto-implémentées init-only
    public string Reference { get; init; }
    public string Nom { get; init; }
    public decimal Prix { get; init; }

    // Constructeur
    public Produit(string reference, string nom, decimal prix)
    {
        Reference = reference;
        Nom = nom;
        Prix = prix;
    }

    // Méthode
    public decimal CalculerPrixTTC(decimal tauxTVA)
    {
        return Prix * (1 + tauxTVA / 100);
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        Produit p1 = new Produit("REF001", "Écran 24 pouces", 199.99m);

        // Initialisation d'objet (n'est pas possible avec le constructeur primaire)
        Produit p2 = new Produit
        {
            Reference = "REF002",
            Nom = "Clavier mécanique",
            Prix = 89.99m
        };

        Console.WriteLine(p1);
        Console.WriteLine($"Prix TTC: {p1.CalculerPrixTTC(20):C}");

        // p1.Prix = 180.0m; // Erreur de compilation - les propriétés init ne peuvent être modifiées qu'à l'initialisation
    }
}
```


## 5.9.2. Avec-expressions pour les copies modifiées

Les records offrent une fonctionnalité appelée "avec-expressions" (with-expressions) qui permet de créer une copie d'un record avec certaines propriétés modifiées, tout en gardant les autres inchangées.

```csharp
// Record de base
public record Adresse(string Rue, string Ville, string CodePostal, string Pays);

public record Client(string Id, string Nom, Adresse AdresseLivraison, Adresse AdresseFacturation);

// Utilisation des avec-expressions
public class Program
{
    public static void Main()
    {
        // Création d'une adresse
        Adresse adresse1 = new Adresse("123 Rue Principale", "Paris", "75001", "France");

        // Création d'une copie modifiée avec 'with'
        Adresse adresse2 = adresse1 with { Ville = "Lyon", CodePostal = "69001" };

        Console.WriteLine(adresse1); // Adresse { Rue = 123 Rue Principale, Ville = Paris, CodePostal = 75001, Pays = France }
        Console.WriteLine(adresse2); // Adresse { Rue = 123 Rue Principale, Ville = Lyon, CodePostal = 69001, Pays = France }

        // Utilisation avec des records imbriqués
        Client client = new Client(
            "C12345",
            "Dupont Entreprises",
            adresse1,
            adresse1  // Même adresse pour la livraison et la facturation
        );

        // Modification de l'adresse de facturation uniquement
        Client clientModifie = client with { AdresseFacturation = adresse2 };

        Console.WriteLine("Client original:");
        Console.WriteLine($"Livraison: {client.AdresseLivraison.Ville}");
        Console.WriteLine($"Facturation: {client.AdresseFacturation.Ville}");

        Console.WriteLine("\nClient modifié:");
        Console.WriteLine($"Livraison: {clientModifie.AdresseLivraison.Ville}");
        Console.WriteLine($"Facturation: {clientModifie.AdresseFacturation.Ville}");
    }
}
```


### Copies profondes et superficielles

Il est important de noter que les avec-expressions font des copies superficielles. Si votre record contient des références à des objets mutables, ces références seront partagées entre l'original et la copie.

```csharp
// Record avec une liste (référence mutable)
public record Panier(string ClientId, List<string> Articles);

// Démonstration du problème de copie superficielle
public class Program
{
    public static void Main()
    {
        List<string> articles = new List<string> { "Pommes", "Bananes" };

        Panier panier1 = new Panier("C001", articles);

        // Création d'une copie modifiée
        Panier panier2 = panier1 with { ClientId = "C002" };

        // Modification de la liste d'articles
        panier1.Articles.Add("Oranges");

        Console.WriteLine("Panier 1:");
        foreach (var article in panier1.Articles)
            Console.WriteLine($"- {article}");

        Console.WriteLine("\nPanier 2 (partage la même référence à la liste):");
        foreach (var article in panier2.Articles)
            Console.WriteLine($"- {article}");

        // Solution: créer une nouvelle liste lors de la modification
        Panier panier3 = panier1 with {
            ClientId = "C003",
            Articles = new List<string>(panier1.Articles)
        };

        panier1.Articles.Add("Cerises");

        Console.WriteLine("\nPanier 3 (avec copie profonde):");
        foreach (var article in panier3.Articles)
            Console.WriteLine($"- {article}");
    }
}
```


## 5.9.3. Déconstruction et pattern matching

Les records supportent nativement la déconstruction et sont particulièrement adaptés pour le pattern matching.

### Déconstruction

```csharp
public record Point(double X, double Y, double Z);

public class Program
{
    public static void Main()
    {
        Point point = new Point(10.5, 20.3, 30.1);

        // Déconstruction en variables individuelles
        var (x, y, z) = point;
        Console.WriteLine($"Coordonnées: x={x}, y={y}, z={z}");

        // Déconstruction partielle
        var (xOnly, _, _) = point;
        Console.WriteLine($"X uniquement: {xOnly}");

        // Modification avec déconstruction
        Point deplace = point with { X = x + 5, Y = y - 3 };
        Console.WriteLine($"Point déplacé: {deplace}");

        // Record personnalisé avec déconstruction personnalisée
        AutreExempleDeconstruction();
    }

    // Déconstruction personnalisée
    private static void AutreExempleDeconstruction()
    {
        var rectangle = new Rectangle(10, 20);

        // Utilisation de la déconstruction automatique
        var (largeur, hauteur) = rectangle;
        Console.WriteLine($"Dimensions: {largeur}x{hauteur}");

        // Calcul de l'aire à partir des valeurs déconstruites
        double aire = largeur * hauteur;
        Console.WriteLine($"Aire: {aire}");
    }
}

// Record avec des propriétés supplémentaires
public record Rectangle(double Largeur, double Hauteur)
{
    // Propriété calculée
    public double Aire => Largeur * Hauteur;
    public double Perimetre => 2 * (Largeur + Hauteur);

    // On peut aussi personnaliser la déconstruction - mais ce n'est pas nécessaire
    // pour les propriétés positionnelles qui sont automatiquement déconstruites
}
```


### Pattern matching avec les records

Les records fonctionnent particulièrement bien avec le pattern matching introduit dans les versions récentes de C#.

```csharp
// Hiérarchie de records
public abstract record Forme;
public record Cercle(double Rayon) : Forme;
public record Rectangle(double Largeur, double Hauteur) : Forme;
public record Triangle(double CoteA, double CoteB, double CoteC) : Forme;

public class CalculateurForme
{
    public static double CalculerAire(Forme forme) => forme switch
    {
        Cercle(var rayon) => Math.PI * rayon * rayon,
        Rectangle(var l, var h) => l * h,
        Triangle(var a, var b, var c) => CalculerAireTriangle(a, b, c),
        _ => throw new ArgumentException("Forme non reconnue", nameof(forme))
    };

    public static double CalculerPerimetre(Forme forme) => forme switch
    {
        Cercle(var rayon) => 2 * Math.PI * rayon,
        Rectangle(var l, var h) => 2 * (l + h),
        Triangle(var a, var b, var c) => a + b + c,
        _ => throw new ArgumentException("Forme non reconnue", nameof(forme))
    };

    private static double CalculerAireTriangle(double a, double b, double c)
    {
        // Formule de Héron
        double s = (a + b + c) / 2;
        return Math.Sqrt(s * (s - a) * (s - b) * (s - c));
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        List<Forme> formes = new List<Forme>
        {
            new Cercle(5),
            new Rectangle(4, 6),
            new Triangle(3, 4, 5)
        };

        foreach (var forme in formes)
        {
            double aire = CalculateurForme.CalculerAire(forme);
            double perimetre = CalculateurForme.CalculerPerimetre(forme);

            // Pattern matching pour déterminer le type de forme et extraire ses propriétés
            string description = forme switch
            {
                Cercle(var r) => $"Cercle de rayon {r}",
                Rectangle(var l, var h) => $"Rectangle de dimensions {l}x{h}",
                Triangle(var a, var b, var c) => $"Triangle de côtés {a}, {b}, {c}",
                _ => "Forme inconnue"
            };

            Console.WriteLine($"{description}:");
            Console.WriteLine($"- Aire: {aire:F2}");
            Console.WriteLine($"- Périmètre: {perimetre:F2}");
            Console.WriteLine();
        }
    }
}
```


### Pattern matching avancé

Les records permettent des cas d'utilisation avancés avec le pattern matching, notamment en combinant des patterns de propriété.

```csharp
// Record représentant une personne
public record Personne(string Nom, int Age, string Profession, string Nationalite);

// Utilisation de pattern matching avancé
public class ExemplePatternMatching
{
    public static string CategoriserPersonne(Personne personne) => personne switch
    {
        // Pattern matching sur plusieurs propriétés
        { Age: < 18 } => "Mineur",

        // Combinaison de patterns avec des conditions
        { Profession: "Étudiant", Age: > 18 and < 30 } => "Étudiant adulte",

        // Pattern avec déconstruction et conditions
        Personne(_, var age, "Ingénieur", _) when age > 35 => "Ingénieur expérimenté",

        // Pattern avec conditions sur plusieurs propriétés
        { Profession: "Médecin", Nationalite: "France" or "Belgique" } => "Médecin francophone",

        // Pattern par défaut
        _ => "Autre"
    };

    public static void AfficherStatistiques(List<Personne> personnes)
    {
        // Comptage des personnes par catégorie
        int mineurs = personnes.Count(p => p is { Age: < 18 });
        int ingenieurs = personnes.Count(p => p is { Profession: "Ingénieur" });
        int francais = personnes.Count(p => p is { Nationalite: "France" });

        Console.WriteLine($"Total: {personnes.Count} personnes");
        Console.WriteLine($"Mineurs: {mineurs}");
        Console.WriteLine($"Ingénieurs: {ingenieurs}");
        Console.WriteLine($"Français: {francais}");

        // Utilisation du pattern matching pour grouper et filtrer
        var jeunesEmployes = personnes
            .Where(p => p is { Age: < 30, Profession: not "Étudiant" })
            .ToList();

        Console.WriteLine($"\nJeunes employés (moins de 30 ans, non étudiants): {jeunesEmployes.Count}");
        foreach (var personne in jeunesEmployes)
        {
            Console.WriteLine($"- {personne.Nom}, {personne.Age} ans, {personne.Profession}");
        }
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        List<Personne> personnes = new List<Personne>
        {
            new Personne("Alice Dupont", 17, "Étudiant", "France"),
            new Personne("Bob Martin", 24, "Ingénieur", "Belgique"),
            new Personne("Claire Thomas", 35, "Médecin", "France"),
            new Personne("David Smith", 42, "Ingénieur", "Royaume-Uni"),
            new Personne("Eva Müller", 28, "Avocate", "Allemagne"),
            new Personne("François Legrand", 19, "Étudiant", "France")
        };

        foreach (var personne in personnes)
        {
            string categorie = ExemplePatternMatching.CategoriserPersonne(personne);
            Console.WriteLine($"{personne.Nom}: {categorie}");
        }

        Console.WriteLine();
        ExemplePatternMatching.AfficherStatistiques(personnes);
    }
}
```


### Records struct (C# 10+)

À partir de C# 10 (disponible avec .NET 6 et versions ultérieures), il est possible de définir des records comme des types valeur avec le mot-clé `struct`.

```csharp
// Record struct (C# 10+, .NET 6+)
public record struct Coordonnees(double Latitude, double Longitude)
{
    public double DistanceDe(Coordonnees autre)
    {
        // Formule simplifiée de la distance euclidienne
        double dLat = Latitude - autre.Latitude;
        double dLon = Longitude - autre.Longitude;
        return Math.Sqrt(dLat * dLat + dLon * dLon);
    }
}

public class Program
{
    public static void Main()
    {
        Coordonnees paris = new Coordonnees(48.8566, 2.3522);
        Coordonnees londres = new Coordonnees(51.5074, -0.1278);

        Console.WriteLine($"Paris: {paris}");
        Console.WriteLine($"Londres: {londres}");

        double distance = paris.DistanceDe(londres);
        Console.WriteLine($"Distance approximative: {distance:F2} degrés");

        // Avec-expression fonctionne également avec les record struct
        Coordonnees parisModifie = paris with { Longitude = 2.3600 };
        Console.WriteLine($"Paris modifié: {parisModifie}");
    }
}
```


Les records constituent un ajout majeur à C# 9 et aux versions ultérieures, offrant une manière concise et élégante de manipuler des données immuables. Ils sont particulièrement utiles pour les types de transfert de données (DTOs), les modèles, les entités de domaine et dans les applications utilisant des approches fonctionnelles ou d'architecture en couches.

⏭️
