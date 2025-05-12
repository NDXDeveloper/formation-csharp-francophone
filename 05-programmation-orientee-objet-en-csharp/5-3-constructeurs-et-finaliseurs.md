# 5.3. Constructeurs et finaliseurs

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les constructeurs et finaliseurs jouent un rôle crucial dans le cycle de vie d'un objet en C#, en contrôlant respectivement son initialisation et sa libération. Cette section explore les différents types de constructeurs, les techniques d'initialisation d'objets, le chaînage de constructeurs, et l'utilisation appropriée des finaliseurs avec le pattern IDisposable.

## 5.3.1. Constructeurs par défaut, paramétrés et statiques

Les constructeurs sont des méthodes spéciales qui sont appelées lors de la création d'une instance d'une classe, permettant d'initialiser les champs et propriétés de l'objet.

### Constructeur par défaut

Le constructeur par défaut est un constructeur sans paramètres. Si vous ne définissez aucun constructeur, le compilateur fournit automatiquement un constructeur par défaut public et sans paramètre.

```csharp
public class Client
{
    // Champs et propriétés
    public string Nom { get; set; }
    public string Email { get; set; }

    // Constructeur par défaut explicite
    public Client()
    {
        // Initialisation des champs/propriétés à des valeurs par défaut
        Nom = "Anonyme";
        Email = "non-défini";
    }
}

// Utilisation
Client client = new Client();
Console.WriteLine(client.Nom);  // Affiche "Anonyme"
```


> **Important**: Si vous définissez un constructeur avec des paramètres mais que vous ne définissez pas explicitement un constructeur sans paramètre, le compilateur ne fournira pas de constructeur par défaut.

```csharp
public class Produit
{
    public string Nom { get; set; }
    public decimal Prix { get; set; }

    // Seul constructeur défini
    public Produit(string nom, decimal prix)
    {
        Nom = nom;
        Prix = prix;
    }
}

// Utilisation
// Produit p = new Produit();  // Erreur: aucun constructeur par défaut n'est disponible
Produit p = new Produit("Ordinateur", 999.99m);  // OK
```


### Constructeurs paramétrés

Les constructeurs paramétrés acceptent des arguments qui peuvent être utilisés pour initialiser l'objet.

```csharp
public class Commande
{
    public string Référence { get; set; }
    public DateTime Date { get; set; }
    public List<LigneCommande> Lignes { get; private set; }
    public decimal Total { get; private set; }

    // Constructeur paramétré
    public Commande(string référence)
    {
        Référence = référence;
        Date = DateTime.Now;
        Lignes = new List<LigneCommande>();
    }

    // Autre constructeur paramétré
    public Commande(string référence, DateTime date)
    {
        Référence = référence;
        Date = date;
        Lignes = new List<LigneCommande>();
    }

    // Méthode pour ajouter une ligne et recalculer le total
    public void AjouterLigne(LigneCommande ligne)
    {
        Lignes.Add(ligne);
        RecalculerTotal();
    }

    private void RecalculerTotal()
    {
        Total = Lignes.Sum(l => l.PrixUnitaire * l.Quantité);
    }
}

public class LigneCommande
{
    public string Produit { get; set; }
    public decimal PrixUnitaire { get; set; }
    public int Quantité { get; set; }
}

// Utilisation
var commande = new Commande("CMD-2023-001");
commande.AjouterLigne(new LigneCommande
{
    Produit = "Ordinateur",
    PrixUnitaire = 999.99m,
    Quantité = 1
});
```


### Paramètres optionnels et nommés dans les constructeurs

Depuis C# 4.0, vous pouvez utiliser des paramètres optionnels et nommés dans les constructeurs :

```csharp
public class Configuration
{
    public string Serveur { get; }
    public int Port { get; }
    public bool SSL { get; }
    public int Timeout { get; }

    // Constructeur avec paramètres optionnels
    public Configuration(
        string serveur,
        int port = 8080,           // Paramètre optionnel avec valeur par défaut
        bool ssl = false,          // Paramètre optionnel avec valeur par défaut
        int timeout = 30)          // Paramètre optionnel avec valeur par défaut
    {
        Serveur = serveur;
        Port = port;
        SSL = ssl;
        Timeout = timeout;
    }
}

// Utilisation avec différentes combinaisons
var config1 = new Configuration("localhost");
var config2 = new Configuration("monserveur.com", 443, true);
var config3 = new Configuration("autreserveur.com", timeout: 60);  // Paramètre nommé
```


### Constructeurs privés

Les constructeurs privés empêchent l'instanciation directe d'une classe, ce qui est utile pour implémenter des patterns comme Singleton ou Factory Method :

```csharp
public class Logger
{
    // Instance unique (Singleton)
    private static Logger _instance;

    // Verrou pour l'accès concurrent
    private static readonly object _lock = new object();

    // Collection des messages
    private readonly List<string> _logs = new List<string>();

    // Constructeur privé
    private Logger()
    {
        // Initialisation
    }

    // Méthode pour obtenir l'instance unique
    public static Logger GetInstance()
    {
        // Double-check locking pattern
        if (_instance == null)
        {
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = new Logger();
                }
            }
        }

        return _instance;
    }

    // Méthodes de journalisation
    public void LogInfo(string message)
    {
        string formattedMessage = $"INFO [{DateTime.Now}]: {message}";
        _logs.Add(formattedMessage);
        Console.WriteLine(formattedMessage);
    }

    public void LogError(string message)
    {
        string formattedMessage = $"ERROR [{DateTime.Now}]: {message}";
        _logs.Add(formattedMessage);
        Console.WriteLine(formattedMessage);
    }
}

// Utilisation
// var logger = new Logger();  // Erreur: constructeur privé
var logger = Logger.GetInstance();
logger.LogInfo("Application démarrée");
```


### Constructeurs statiques

Un constructeur statique est utilisé pour initialiser des membres statiques d'une classe. Il est appelé automatiquement avant la première utilisation de la classe.

```csharp
public class DatabaseConfig
{
    // Membres statiques
    public static string ConnectionString { get; private set; }
    public static int MaxConnections { get; private set; }
    public static bool IsInitialized { get; private set; }

    // Constructeur statique
    static DatabaseConfig()
    {
        Console.WriteLine("Initialisation de DatabaseConfig...");

        // Charge la configuration depuis un fichier ou l'environnement
        ConnectionString = "Server=localhost;Database=MyDB;User Id=admin;Password=******;";
        MaxConnections = 100;
        IsInitialized = true;
    }

    // Méthode statique
    public static void DisplayConfig()
    {
        Console.WriteLine($"Connection String: {ConnectionString}");
        Console.WriteLine($"Max Connections: {MaxConnections}");
    }
}

// Utilisation
// Le constructeur statique s'exécute lors de la première référence à la classe
Console.WriteLine($"Database initialized: {DatabaseConfig.IsInitialized}");
DatabaseConfig.DisplayConfig();
```


Caractéristiques importantes des constructeurs statiques :

1. Un constructeur statique n'a pas de modificateurs d'accès et ne peut pas prendre de paramètres.
2. Une classe ne peut avoir qu'un seul constructeur statique.
3. Le constructeur statique s'exécute automatiquement avant la première utilisation de la classe.
4. Vous ne pouvez pas appeler explicitement un constructeur statique.
5. Les exceptions non gérées dans un constructeur statique rendent la classe inutilisable pour la durée du processus.

### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Constructeurs de base | ✅ | ✅ |
| Paramètres nommés et optionnels | ✅ | ✅ |
| Constructeurs privés | ✅ | ✅ |
| Constructeurs statiques | ✅ | ✅ |
| Constructeurs primaires (primary constructors) | ❌ | ✅ |
| Required members | ❌ | ✅ |

### Constructeurs primaires (.NET 8 uniquement)

.NET 8 avec C# 12 introduit les constructeurs primaires pour les classes, une fonctionnalité qui n'est pas disponible dans .NET Framework 4.7.2 :

```csharp
// Disponible uniquement dans .NET 8
public class Personne(string nom, int age)
{
    public string Nom { get; } = nom;
    public int Age { get; } = age;

    public bool EstMajeur => Age >= 18;

    public void AfficherInfos()
    {
        Console.WriteLine($"{Nom}, {Age} ans");
    }
}

// Utilisation
var personne = new Personne("Jean Dupont", 30);
personne.AfficherInfos();
```


## 5.3.2. Initialisation d'objets

Il existe plusieurs façons d'initialiser des objets en C#, chacune offrant différents niveaux de flexibilité et de concision.

### Initialisation avec constructeurs

L'approche la plus courante pour initialiser un objet consiste à utiliser des constructeurs :

```csharp
public class Adresse
{
    public string Rue { get; set; }
    public string Ville { get; set; }
    public string CodePostal { get; set; }
    public string Pays { get; set; }

    // Constructeur par défaut
    public Adresse()
    {
        Pays = "France";  // Valeur par défaut
    }

    // Constructeur complet
    public Adresse(string rue, string ville, string codePostal, string pays)
    {
        Rue = rue;
        Ville = ville;
        CodePostal = codePostal;
        Pays = pays;
    }
}

// Utilisation
Adresse adresse1 = new Adresse();
adresse1.Rue = "123 Rue de Paris";
adresse1.Ville = "Lyon";
adresse1.CodePostal = "69000";

Adresse adresse2 = new Adresse("45 Avenue Victor Hugo", "Paris", "75016", "France");
```


### Initialiseurs d'objets

Les initialiseurs d'objets introduits dans C# 3.0 permettent d'initialiser un objet et ses propriétés en une seule expression :

```csharp
// Utilisation d'initialiseurs d'objets
Adresse adresse3 = new Adresse
{
    Rue = "78 Boulevard Saint-Michel",
    Ville = "Paris",
    CodePostal = "75006",
    Pays = "France"
};

// Combinaison avec un constructeur
Adresse adresse4 = new Adresse("1 Place Bellecour", "Lyon", "69002", "France")
{
    // Vous pouvez remplacer des valeurs définies par le constructeur
    Pays = "FRANCE"
};
```


### Initialisation de collections

Les initialiseurs de collections permettent d'initialiser des collections en une seule expression :

```csharp
public class Répertoire
{
    public string Nom { get; set; }
    public List<Adresse> Adresses { get; set; }
    public Dictionary<string, string> Contacts { get; set; }

    public Répertoire()
    {
        Adresses = new List<Adresse>();
        Contacts = new Dictionary<string, string>();
    }
}

// Initialisation de collections
var répertoire = new Répertoire
{
    Nom = "Mes contacts",
    // Initialisation de la liste d'adresses
    Adresses =
    {
        new Adresse { Rue = "123 Rue Principale", Ville = "Paris", CodePostal = "75001" },
        new Adresse { Rue = "45 Avenue des Fleurs", Ville = "Lyon", CodePostal = "69000" }
    },
    // Initialisation du dictionnaire de contacts
    Contacts =
    {
        ["Personnel"] = "+33 6 12 34 56 78",
        ["Professionnel"] = "+33 1 23 45 67 89"
    }
};
```


### Champs et auto-propriétés avec initialiseurs

Depuis C# 6.0, vous pouvez initialiser des champs et des propriétés auto-implémentées directement à leur point de déclaration :

```csharp
public class Article
{
    // Initialisation de champ
    private readonly Guid _id = Guid.NewGuid();

    // Initialisation d'auto-propriétés
    public string Référence { get; set; } = "ART-" + DateTime.Now.Ticks.ToString().Substring(10);
    public DateTime DateCréation { get; set; } = DateTime.Now;
    public List<string> Étiquettes { get; set; } = new List<string>();
    public bool EstActif { get; set; } = true;

    // Propriétés normales
    public string Nom { get; set; }
    public decimal Prix { get; set; }

    public Article()
    {
        // Constructeur par défaut
    }

    public Article(string nom, decimal prix)
    {
        Nom = nom;
        Prix = prix;
    }
}

// Utilisation
var article = new Article("Smartphone", 499.99m);
Console.WriteLine(article.Référence);      // Référence générée
Console.WriteLine(article.DateCréation);   // Date actuelle
article.Étiquettes.Add("Électronique");
```


### Initialisation d'objets immuables

Pour les objets immuables, plusieurs approches sont possibles :

#### Approche .NET Framework 4.7.2 - Propriétés en lecture seule

```csharp
// .NET Framework 4.7.2
public class ConfigurationImmuable
{
    // Propriétés en lecture seule
    public string Environnement { get; }
    public string CheminDonnées { get; }
    public int NombreMaxUtilisateurs { get; }

    // Constructeur qui initialise toutes les propriétés
    public ConfigurationImmuable(string environnement, string cheminDonnées, int nombreMaxUtilisateurs)
    {
        Environnement = environnement;
        CheminDonnées = cheminDonnées;
        NombreMaxUtilisateurs = nombreMaxUtilisateurs;
    }

    // Méthode pour créer une copie modifiée
    public ConfigurationImmuable AvecEnvironnement(string nouvelEnvironnement)
    {
        return new ConfigurationImmuable(nouvelEnvironnement, CheminDonnées, NombreMaxUtilisateurs);
    }

    public ConfigurationImmuable AvecCheminDonnées(string nouveauCheminDonnées)
    {
        return new ConfigurationImmuable(Environnement, nouveauCheminDonnées, NombreMaxUtilisateurs);
    }
}

// Utilisation
var configProd = new ConfigurationImmuable("Production", "/var/data/prod", 1000);
var configTest = configProd.AvecEnvironnement("Test");
```


#### Approche .NET 8 - Propriétés init-only et records

```csharp
// .NET 8 uniquement
public class ConfigurationImmuable
{
    // Propriétés init-only
    public string Environnement { get; init; }
    public string CheminDonnées { get; init; }
    public int NombreMaxUtilisateurs { get; init; }

    // Constructeur facultatif
    public ConfigurationImmuable(string environnement, string cheminDonnées, int nombreMaxUtilisateurs)
    {
        Environnement = environnement;
        CheminDonnées = cheminDonnées;
        NombreMaxUtilisateurs = nombreMaxUtilisateurs;
    }
}

// Utilisation avec initialiseurs d'objets
var configProd = new ConfigurationImmuable
{
    Environnement = "Production",
    CheminDonnées = "/var/data/prod",
    NombreMaxUtilisateurs = 1000
};

// Création d'une copie modifiée
var configTest = new ConfigurationImmuable
{
    Environnement = "Test",
    CheminDonnées = configProd.CheminDonnées,
    NombreMaxUtilisateurs = configProd.NombreMaxUtilisateurs
};

// Alternative avec records (.NET 8)
public record ConfigRecord(string Environnement, string CheminDonnées, int NombreMaxUtilisateurs);

// Utilisation du record
var configRecProd = new ConfigRecord("Production", "/var/data/prod", 1000);

// Création d'une copie modifiée avec l'opérateur "with"
var configRecTest = configRecProd with { Environnement = "Test" };
```


### Initialisation avec des membres requis (.NET 8 uniquement)

.NET 8 introduit le concept de membres requis qui doivent être initialisés :

```csharp
// .NET 8 uniquement
public class DocumentRequired
{
    // Membres requis qui doivent être initialisés
    public required string Titre { get; set; }
    public required string Auteur { get; set; }

    // Membres optionnels
    public DateTime DateCréation { get; set; } = DateTime.Now;
    public int NombrePages { get; set; }
}

// Utilisation - doit spécifier Titre et Auteur
var document = new DocumentRequired
{
    Titre = "Guide C#",
    Auteur = "Jean Dupont"
};

// Erreur de compilation si les membres requis ne sont pas initialisés
// var docInvalide = new DocumentRequired { Titre = "Titre seul" };
```


## 5.3.3. Chaînage de constructeurs

Le chaînage de constructeurs vous permet d'appeler un constructeur à partir d'un autre constructeur dans la même classe ou dans la classe de base.

### Chaînage dans la même classe avec `this`

```csharp
public class Utilisateur
{
    public string Nom { get; set; }
    public string Email { get; set; }
    public DateTime DateInscription { get; }
    public bool EstActif { get; set; }

    // Constructeur principal
    public Utilisateur(string nom, string email, DateTime dateInscription, bool estActif)
    {
        Nom = nom;
        Email = email;
        DateInscription = dateInscription;
        EstActif = estActif;
    }

    // Constructeur qui chaîne vers le constructeur principal
    public Utilisateur(string nom, string email)
        : this(nom, email, DateTime.Now, true)
    {
        // Le corps peut contenir une logique supplémentaire
    }

    // Autre constructeur chaîné
    public Utilisateur(string email)
        : this("Utilisateur", email)
    {
        // Logique spécifique à ce constructeur
    }

    // Constructeur par défaut chaîné
    public Utilisateur()
        : this("anonymous@example.com")
    {
    }
}

// Utilisation
var user1 = new Utilisateur("Jean Dupont", "jean.dupont@example.com");
var user2 = new Utilisateur("marie@example.com");
var user3 = new Utilisateur();
```


### Chaînage vers la classe de base avec `base`

```csharp
public class Personne
{
    public string Nom { get; }
    public string Prénom { get; }

    public Personne(string nom, string prénom)
    {
        Nom = nom;
        Prénom = prénom;
    }

    public virtual void AfficherInfos()
    {
        Console.WriteLine($"Personne: {Prénom} {Nom}");
    }
}

public class Employé : Personne
{
    public string Fonction { get; }
    public decimal Salaire { get; }

    // Constructeur qui appelle le constructeur de la classe de base
    public Employé(string nom, string prénom, string fonction, decimal salaire)
        : base(nom, prénom)
    {
        Fonction = fonction;
        Salaire = salaire;
    }

    // Constructeur simplifié qui chaîne au constructeur local
    public Employé(string nom, string prénom, string fonction)
        : this(nom, prénom, fonction, 0)
    {
    }

    public override void AfficherInfos()
    {
        Console.WriteLine($"Employé: {Prénom} {Nom}, {Fonction}, {Salaire:C}");
    }
}

// Utilisation
var personne = new Personne("Dupont", "Jean");
personne.AfficherInfos();  // "Personne: Jean Dupont"

var employé = new Employé("Martin", "Marie", "Développeuse", 50000);
employé.AfficherInfos();   // "Employé: Marie Martin, Développeuse, 50 000,00 €"
```


### Initialisation dans une hiérarchie de classes

Dans une hiérarchie de classes, l'initialisation se fait dans cet ordre :

1. Les champs statiques sont initialisés (dans l'ordre de déclaration)
2. Le constructeur statique est appelé
3. Les champs d'instance sont initialisés (dans l'ordre de déclaration)
4. Le constructeur de la classe de base est appelé
5. Le corps du constructeur de la classe dérivée est exécuté

```csharp
public class A
{
    // Champs d'instance avec initialiseurs
    public string ChampA1 = "A1";
    public string ChampA2;

    // Constructeur
    public A()
    {
        Console.WriteLine("Constructeur A");
        Console.WriteLine($"  ChampA1: {ChampA1}");
        ChampA2 = "A2 depuis constructeur";
    }
}

public class B : A
{
    // Champs d'instance avec initialiseurs
    public string ChampB1 = "B1";
    public string ChampB2;

    // Constructeur
    public B()
    {
        Console.WriteLine("Constructeur B");
        Console.WriteLine($"  ChampB1: {ChampB1}");
        ChampB2 = "B2 depuis constructeur";
    }
}

// Utilisation
var obj = new B();

// Sortie:
// Constructeur A
//   ChampA1: A1
// Constructeur B
//   ChampB1: B1
```


### Constructeurs privés et Factory Methods

Les constructeurs privés combinés avec des méthodes de fabrique (factory methods) permettent de contrôler précisément la création d'instances :

```csharp
public class Produit
{
    public string Nom { get; }
    public decimal Prix { get; }
    public string Catégorie { get; }

    // Constructeur privé - empêche l'instanciation directe
    private Produit(string nom, decimal prix, string catégorie)
    {
        Nom = nom;
        Prix = prix;
        Catégorie = catégorie;
    }

    // Méthodes de fabrique statiques
    public static Produit CréerProduitStandard(string nom, decimal prix)
    {
        return new Produit(nom, prix, "Standard");
    }

    public static Produit CréerProduitPremium(string nom, decimal prix)
    {
        // Logique spécifique pour les produits premium
        return new Produit(nom, prix * 1.5m, "Premium");
    }

    public static Produit CréerProduitPromotionnel(string nom)
    {
        // Logique pour les produits promotionnels
        return new Produit(nom, 0.99m, "Promo");
    }
}

// Utilisation
var produit1 = Produit.CréerProduitStandard("Stylo", 1.50m);
var produit2 = Produit.CréerProduitPremium("Stylo luxe", 5.00m);  // Prix ajusté à 7.50
var produit3 = Produit.CréerProduitPromotionnel("Stylo promo");
```


## 5.3.4. Finaliseurs et pattern IDisposable

Les finaliseurs et le pattern IDisposable sont utilisés pour libérer des ressources non gérées, comme les fichiers, les connexions réseau, ou les handles système.

### Finaliseurs (Destructeurs)

Un finaliseur est une méthode spéciale qui est appelée par le Garbage Collector avant que l'objet ne soit récupéré, permettant ainsi la libération des ressources non gérées.

```csharp
public class RessourceNonGérée
{
    // Handle pour une ressource non gérée (comme un fichier ou une connexion)
    private IntPtr _handle;
    private bool _disposed = false;

    public RessourceNonGérée()
    {
        // Simuler l'acquisition d'une ressource
        Console.WriteLine("Acquisition de la ressource");
        _handle = new IntPtr(1);
    }

    // Finaliseur/Destructeur
    ~RessourceNonGérée()
    {
        Console.WriteLine("Finaliseur appelé");
        Dispose(false);
    }

    // Méthode pour libérer les ressources
    private void LibérerRessource()
    {
        if (_handle != IntPtr.Zero)
        {
            Console.WriteLine("Libération de la ressource");
            // Simuler la libération de la ressource
            _handle = IntPtr.Zero;
        }
    }

    // Méthode protégée pour libérer les ressources
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            // Libérer les ressources non gérées
            LibérerRessource();

            // Libérer les ressources gérées si demandé
            if (disposing)
            {
                // Libérer les ressources gérées ici
            }

            _disposed = true;
        }
    }
}

// Utilisation
var ressource = new RessourceNonGérée();
// ...
// Quand l'objet n'est plus référencé, le GC finira par appeler le finaliseur
```


> **Important**: Les finaliseurs ne sont pas déterministes et vous ne devez pas vous fier à eux pour libérer des ressources critiques, car vous ne savez pas quand ils seront appelés par le Garbage Collector. Utilisez toujours IDisposable pour une libération explicite et déterministe des ressources.

### Pattern IDisposable

Le pattern IDisposable fournit une méthode déterministe pour libérer des ressources sans dépendre du Garbage Collector :

```csharp
public class GestionnaireConnexion : IDisposable
{
    private SqlConnection _connexion;
    private bool _disposed = false;

    public GestionnaireConnexion(string chaîneConnexion)
    {
        _connexion = new SqlConnection(chaîneConnexion);
        _connexion.Open();
        Console.WriteLine("Connexion ouverte");
    }

    // Implémentation de l'interface IDisposable
    public void Dispose()
    {
        Dispose(true);
        // Supprime l'appel au finaliseur
        GC.SuppressFinalize(this);
    }

    // Finaliseur
    ~GestionnaireConnexion()
    {
        Dispose(false);
    }

    // Méthode protégée qui implémente la logique de nettoyage
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Libérer les ressources gérées
                if (_connexion != null)
                {
                    _connexion.Close();
                    _connexion.Dispose();
                    _connexion = null;
                    Console.WriteLine("Connexion fermée et disposée");
                }
            }

            // Libérer les ressources non gérées
            // (dans ce cas, tout est géré par SqlConnection)

            _disposed = true;
        }
    }

    // Méthode pour exécuter une requête
    public int ExécuterCommande(string sql)
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(GestionnaireConnexion));

        using (var commande = new SqlCommand(sql, _connexion))
        {
            return commande.ExecuteNonQuery();
        }
    }
}

// Utilisation avec "using" pour garantir la libération des ressources
using (var gestionnaire = new GestionnaireConnexion("Data Source=localhost;Initial Catalog=MaBase;"))
{
    gestionnaire.ExécuterCommande("INSERT INTO Logs (Message) VALUES ('Test')");
}  // Dispose est appelé automatiquement ici

// Alternative avec la syntaxe "using" déclaratif (C# 8.0+)
using var gestionnaire2 = new GestionnaireConnexion("Data Source=localhost;Initial Catalog=MaBase;");
gestionnaire2.ExécuterCommande("INSERT INTO Logs (Message) VALUES ('Test 2')");
// Dispose sera appelé à la fin du bloc englobant
```


### IAsyncDisposable (.NET Core 3.0+/.NET 8)

.NET Core 3.0 et versions ultérieures, y compris .NET 8, introduisent l'interface IAsyncDisposable pour la libération asynchrone des ressources :

```csharp
// Disponible uniquement dans .NET 8, pas dans .NET Framework 4.7.2
public class GestionnaireConnexionAsync : IAsyncDisposable
{
    private SqlConnection _connexion;
    private bool _disposed = false;

    public async Task InitialiserAsync(string chaîneConnexion)
    {
        _connexion = new SqlConnection(chaîneConnexion);
        await _connexion.OpenAsync();
        Console.WriteLine("Connexion ouverte de manière asynchrone");
    }

    // Implémentation de l'interface IAsyncDisposable
    public async ValueTask DisposeAsync()
    {
        await DisposeAsyncCore();

        // Supprime l'appel au finaliseur
        GC.SuppressFinalize(this);
    }

    // Méthode protégée pour la libération asynchrone des ressources
    protected virtual async ValueTask DisposeAsyncCore()
    {
        if (!_disposed)
        {
            if (_connexion != null)
            {
                await _connexion.CloseAsync();
                await _connexion.DisposeAsync();
                _connexion = null;
                Console.WriteLine("Connexion fermée et disposée de manière asynchrone");
            }

            _disposed = true;
        }
    }

    // Méthode asynchrone pour exécuter une requête
    public async Task<int> ExécuterCommandeAsync(string sql)
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(GestionnaireConnexionAsync));

        using var commande = new SqlCommand(sql, _connexion);
        return await commande.ExecuteNonQueryAsync();
    }
}

// Utilisation avec "await using" pour garantir la libération asynchrone des ressources
async Task ExempleAsync()
{
    await using var gestionnaire = new GestionnaireConnexionAsync();
    await gestionnaire.InitialiserAsync("Data Source=localhost;Initial Catalog=MaBase;");
    await gestionnaire.ExécuterCommandeAsync("INSERT INTO Logs (Message) VALUES ('Test Async')");
}  // DisposeAsync est appelé automatiquement ici

// Alternative avec la syntaxe "await using" déclaratif (C# 8.0+)
async Task AutreExempleAsync()
{
    await using var gestionnaire = new GestionnaireConnexionAsync();
    await gestionnaire.InitialiserAsync("Data Source=localhost;Initial Catalog=MaBase;");
    await gestionnaire.ExécuterCommandeAsync("INSERT INTO Logs (Message) VALUES ('Test Async 2')");
    // DisposeAsync sera appelé à la fin du bloc englobant
}
```


### Implémentation complète du pattern Dispose

Voici une implémentation complète du pattern Dispose avec support des ressources gérées et non gérées :

```csharp
public class GestionnaireFichier : IDisposable
{
    private FileStream _fichier;
    private IntPtr _handle = IntPtr.Zero;
    private bool _disposed = false;

    public GestionnaireFichier(string chemin)
    {
        try
        {
            // Ressource gérée
            _fichier = new FileStream(chemin, FileMode.OpenOrCreate);
            Console.WriteLine("Fichier ouvert");

            // Simulation d'une ressource non gérée
            _handle = new IntPtr(1);
            Console.WriteLine("Ressource non gérée acquise");
        }
        catch (Exception)
        {
            Dispose();
            throw;
        }
    }

    // Méthode publique d'implémentation de IDisposable
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    // Finaliseur
    ~GestionnaireFichier()
    {
        Console.WriteLine("Finaliseur appelé");
        Dispose(false);
    }

    // Méthode protégée pour libérer les ressources
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Libérer les ressources gérées
                if (_fichier != null)
                {
                    _fichier.Flush();
                    _fichier.Close();
                    _fichier.Dispose();
                    _fichier = null;
                    Console.WriteLine("Fichier fermé et disposé");
                }
            }

            // Libérer les ressources non gérées
            if (_handle != IntPtr.Zero)
            {
                // Code pour libérer la ressource native
                // Par exemple: CloseHandle(_handle) en P/Invoke
                _handle = IntPtr.Zero;
                Console.WriteLine("Ressource non gérée libérée");
            }

            _disposed = true;
        }
    }

    // Méthode pour vérifier si l'objet a été disposé
    protected void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(GestionnaireFichier), "Cette instance a déjà été disposée");
    }

    // Méthode publique qui utilise les ressources
    public void ÉcrireDonnées(byte[] données)
    {
        // Vérifier si l'objet a été disposé
        ThrowIfDisposed();

        _fichier.Write(données, 0, données.Length);
        Console.WriteLine($"{données.Length} octets écrits");
    }
}

// Utilisation
public void ExempleUtilisation()
{
    using (var gestionnaire = new GestionnaireFichier("exemple.txt"))
    {
        byte[] données = new byte[] { 65, 66, 67, 68 };  // ABCD
        gestionnaire.ÉcrireDonnées(données);
    }  // Dispose est appelé automatiquement ici

    // Si on essaie d'utiliser l'objet après Dispose
    // gestionnaire.ÉcrireDonnées(données);  // Lève ObjectDisposedException
}
```


### SafeHandle pour les ressources Win32

Pour gérer correctement les ressources Win32 non gérées, .NET fournit la classe abstraite SafeHandle qui simplifie la gestion des handles :

```csharp
using Microsoft.Win32.SafeHandles;
using System.Runtime.InteropServices;

public class GestionnaireHandle : IDisposable
{
    // Import des fonctions Win32
    [DllImport("kernel32.dll", SetLastError = true)]
    private static extern SafeFileHandle CreateFile(
        string fileName,
        uint dwDesiredAccess,
        uint dwShareMode,
        IntPtr lpSecurityAttributes,
        uint dwCreationDisposition,
        uint dwFlagsAndAttributes,
        IntPtr hTemplateFile);

    // Constantes Win32
    private const uint GENERIC_READ = 0x80000000;
    private const uint OPEN_EXISTING = 3;
    private const uint FILE_ATTRIBUTE_NORMAL = 0x80;

    // Handle sécurisé
    private SafeFileHandle _handle;
    private bool _disposed = false;

    public GestionnaireHandle(string chemin)
    {
        // Obtenir un handle sécurisé
        _handle = CreateFile(
            chemin,
            GENERIC_READ,
            0,
            IntPtr.Zero,
            OPEN_EXISTING,
            FILE_ATTRIBUTE_NORMAL,
            IntPtr.Zero);

        if (_handle.IsInvalid)
        {
            throw new System.ComponentModel.Win32Exception(Marshal.GetLastWin32Error());
        }

        Console.WriteLine("Handle acquis");
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
                // Libérer le handle sécurisé
                if (_handle != null && !_handle.IsInvalid)
                {
                    _handle.Dispose();
                    Console.WriteLine("Handle disposé");
                }
            }

            _disposed = true;
        }
    }
}
```


### Défis et bonnes pratiques

#### Défis courants

1. **Oubli d'appeler Dispose**
   Si Dispose n'est pas appelé, les ressources non gérées peuvent rester ouvertes jusqu'à ce que le finaliseur s'exécute (ce qui est non déterministe).

2. **Accès à un objet disposé**
   Après qu'un objet a été disposé, ses ressources ont été libérées, et les appels à ses méthodes peuvent échouer.

3. **Exceptions dans les constructeurs**
   Si une exception est levée dans un constructeur, le finaliseur peut être appelé pour un objet partiellement initialisé.

4. **Implémentation incorrecte du pattern Dispose**
   Une implémentation incorrecte peut conduire à des fuites de ressources ou à des erreurs d'exécution.

#### Bonnes pratiques

1. **Utilisez toujours l'instruction using pour les types IDisposable**
   L'instruction using garantit que Dispose est appelé, même en cas d'exception.

2. **Implémentez correctement le pattern Dispose**
   Suivez le modèle standard avec Dispose(bool) et GC.SuppressFinalize.

3. **Vérifiez si un objet a été disposé avant d'utiliser ses ressources**
   Utilisez un indicateur _disposed et lancez ObjectDisposedException si nécessaire.

4. **Protégez les constructeurs avec try/catch/dispose**
   Assurez-vous de libérer les ressources si l'initialisation échoue.

5. **Soyez prudent avec l'héritage**
   Rendez Dispose(bool) virtual pour permettre aux classes dérivées de libérer leurs propres ressources.

6. **Préférez SafeHandle pour les ressources Win32**
   SafeHandle gère automatiquement la libération des ressources, même en cas de finalisation forcée.

7. **Attention aux closures et aux callbacks**
   Les objets capturés dans des closures ou référencés par des delegates peuvent vivre plus longtemps que prévu.

## Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Constructeurs | ✅ | ✅ |
| Initialiseurs d'objets | ✅ | ✅ |
| Chaînage de constructeurs | ✅ | ✅ |
| Finaliseurs | ✅ | ✅ |
| IDisposable | ✅ | ✅ |
| Constructeurs primaires | ❌ | ✅ |
| IAsyncDisposable | ❌ | ✅ |
| await using | ❌ | ✅ |
| using déclaratif | ❌ | ✅ |
| required | ❌ | ✅ |

## Résumé et bonnes pratiques

### Constructeurs

1. **Utilisez des constructeurs explicites pour une initialisation complexe**
   Les constructeurs devraient initialiser l'objet dans un état valide et cohérent.

2. **Préférez les constructeurs paramétrés avec validation**
   Validez les paramètres et lancez des exceptions appropriées pour les entrées invalides.

3. **Utilisez le chaînage de constructeurs pour éviter la duplication de code**
   Le chaînage (`this` ou `base`) permet de réutiliser la logique d'initialisation.

4. **Considérez les constructeurs privés pour les patterns comme Singleton ou Factory**
   Les constructeurs privés vous donnent plus de contrôle sur la création d'instances.

5. **Utilisez les constructeurs statiques pour l'initialisation au niveau de la classe**
   Le constructeur statique est idéal pour initialiser des ressources partagées.

### Initialisation d'objets

1. **Utilisez les initialiseurs d'objets pour une initialisation concise**
   Les initialiseurs d'objets rendent le code plus lisible pour l'initialisation simple.

2. **Utilisez des valeurs par défaut sensibles pour les propriétés**
   Initialisez les propriétés avec des valeurs par défaut raisonnables quand c'est possible.

3. **Préférez l'immutabilité quand c'est approprié**
   Les objets immuables sont plus prévisibles et thread-safe.

4. **Fournissez des méthodes de fabrique pour les scénarios d'initialisation complexes**
   Les méthodes de fabrique peuvent encapsuler la logique de création complexe.

### Finaliseurs et IDisposable

1. **Implémentez IDisposable pour toutes les classes qui gèrent des ressources non gérées**
   Cela permet une libération déterministe des ressources.

2. **Suivez le pattern Dispose standard**
   Implémentez Dispose(bool) et GC.SuppressFinalize pour une gestion correcte des ressources.

3. **Utilisez using pour garantir que Dispose est appelé**
   L'instruction using assure l'appel à Dispose même en cas d'exception.

4. **Préférez SafeHandle pour les ressources Win32**
   SafeHandle est spécialement conçu pour gérer les handles natifs de manière sécurisée.

5. **Testez les scénarios de nettoyage de ressources**
   Assurez-vous que les ressources sont correctement libérées dans tous les scénarios.

6. **Implémentez IAsyncDisposable pour la libération asynchrone des ressources (dans .NET 8)**
   Utilisez IAsyncDisposable pour les opérations de nettoyage qui nécessitent des appels asynchrones.

La compréhension et l'application appropriées des concepts de constructeurs, d'initialisation d'objets et de gestion des ressources sont essentielles pour créer des applications C# robustes, efficaces et qui n'ont pas de fuites de ressources. Ces pratiques contribuent à la création de code qui est à la fois fiable et maintenable.

⏭️ 5.4. [Héritage](/05-programmation-orientee-objet-en-csharp/5-4-heritage.md)
