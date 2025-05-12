# 5.1. Classes et objets

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les classes et les objets constituent le fondement de la programmation orientée objet (POO) en C#, un paradigme qui organise le code autour de "données" et de "comportements", plutôt que de simples fonctions et logiques. Cette section explore en détail comment créer et utiliser des classes en C#, leurs membres, les membres statiques et les différentes façons d'initialiser des objets.

## 5.1.1. Création et instanciation

Une classe en C# est un modèle qui définit les caractéristiques et les comportements d'un type d'objet. C'est à partir de ce modèle que l'on crée des instances, ou objets.

### Définition d'une classe

```csharp
// Structure de base d'une classe
public class Client
{
    // Champs (variables membres)
    private int id;
    private string nom;
    private string email;

    // Propriétés
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Email { get; set; }

    // Constructeurs
    public Client()
    {
        // Constructeur par défaut
    }

    public Client(int id, string nom, string email)
    {
        Id = id;
        Nom = nom;
        Email = email;
    }

    // Méthodes
    public string ObtenirInfos()
    {
        return $"Client {Id}: {Nom} ({Email})";
    }
}
```


### Modificateurs d'accès

Les classes et leurs membres peuvent avoir différents niveaux de visibilité :

```csharp
// Modificateurs d'accès pour les classes
public class ClassePublique { } // Accessible partout
internal class ClasseInterne { } // Accessible uniquement dans l'assembly
                                 // C'est le niveau par défaut si non spécifié

// Modificateurs d'accès pour les membres
public class Exemple
{
    public int PublicMembre;      // Accessible partout
    private int PrivéMembre;      // Accessible uniquement dans cette classe
    protected int ProtégéMembre;  // Accessible dans cette classe et ses dérivées
    internal int InterneMembre;   // Accessible dans cet assembly
    protected internal int ProtégéInterneMembre; // Combinaison de protected et internal
    private protected int PrivéProtégéMembre;    // Accessible dans cette classe et ses
                                                 // dérivées dans le même assembly
                                                 // (.NET Framework 4.7.2 et .NET 8)
}
```


### Instanciation d'objets

Pour créer une instance d'une classe, on utilise l'opérateur `new` :

```csharp
// Instanciation avec constructeur par défaut
Client client1 = new Client();
client1.Id = 1;
client1.Nom = "Dupont";
client1.Email = "dupont@exemple.com";

// Instanciation avec constructeur paramétré
Client client2 = new Client(2, "Martin", "martin@exemple.com");

// Syntaxe simplifiée avec inférence de type (C# 3.0+)
var client3 = new Client(3, "Durand", "durand@exemple.com");

// Syntaxe sans parenthèses (C# 9.0+, .NET 5+ uniquement, pas pour .NET Framework 4.7.2)
Client client4 = new(); // Utilise le constructeur par défaut
```


### Interfaces fluides

Les interfaces fluides permettent d'enchaîner les appels de méthodes, ce qui peut rendre le code plus lisible :

```csharp
public class ConstructeurEmail
{
    private string destinataire;
    private string sujet;
    private string corps;

    // Méthodes retournant this pour permettre le chaînage
    public ConstructeurEmail PourDestinataire(string email)
    {
        destinataire = email;
        return this;
    }

    public ConstructeurEmail AvecSujet(string nouveauSujet)
    {
        sujet = nouveauSujet;
        return this;
    }

    public ConstructeurEmail AvecCorps(string nouveauCorps)
    {
        corps = nouveauCorps;
        return this;
    }

    public void Envoyer()
    {
        // Code pour envoyer l'email
        Console.WriteLine($"Email envoyé à {destinataire} avec sujet '{sujet}'");
    }
}

// Utilisation de l'interface fluide
new ConstructeurEmail()
    .PourDestinataire("client@exemple.com")
    .AvecSujet("Bienvenue !")
    .AvecCorps("Merci de vous être inscrit.")
    .Envoyer();
```


### Destructeurs/Finaliseurs

Les destructeurs (aussi appelés finaliseurs) sont utilisés pour libérer des ressources non gérées :

```csharp
public class RessourceExterne
{
    private IntPtr handle; // Pointeur vers une ressource non gérée

    public RessourceExterne()
    {
        // Acquisition de ressource
        handle = NativeResource.Acquire();
    }

    // Finalizer (destructeur)
    ~RessourceExterne()
    {
        // Libération des ressources non gérées
        if (handle != IntPtr.Zero)
        {
            NativeResource.Release(handle);
            handle = IntPtr.Zero;
        }
    }
}
```


> **Note** : Il est généralement préférable d'implémenter l'interface `IDisposable` plutôt que de s'appuyer uniquement sur les finaliseurs, car le moment d'exécution du finaliseur n'est pas déterministe.

### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Classes de base | ✅ | ✅ |
| Instanciation avec `new()` sans parenthèses | ❌ | ✅ |
| Target-typed new expressions | ❌ | ✅ |
| Initializers améliorés | ❌ | ✅ |
| Record classes | ❌ | ✅ |

## 5.1.2. Membres de classe

Les membres d'une classe sont les éléments qui définissent la structure et le comportement de la classe.

### Champs

Les champs sont des variables déclarées au niveau de la classe :

```csharp
public class Compte
{
    // Champs
    private decimal solde;
    private readonly string numéroCompte;  // readonly: valeur assignée uniquement à la création ou dans constructeur
    public const decimal TAUX_INTERET = 0.02m;  // const: valeur constante connue à la compilation

    public Compte(string numéro)
    {
        numéroCompte = numéro;  // OK car c'est dans le constructeur
    }

    public void AjouterIntérêts()
    {
        solde += solde * TAUX_INTERET;
    }
}

// Utilisation d'une constante depuis l'extérieur
decimal taux = Compte.TAUX_INTERET;
```


### Propriétés

Les propriétés encapsulent des champs en fournissant des accesseurs get et set :

```csharp
public class Employé
{
    // Champ privé sous-jacent
    private string nom;

    // Propriété complète avec accesseurs personnalisés
    public string Nom
    {
        get { return nom; }
        set
        {
            if (!string.IsNullOrEmpty(value))
                nom = value;
        }
    }

    // Auto-propriété (le compilateur génère le champ sous-jacent)
    public string Prénom { get; set; }

    // Auto-propriété avec initialisation (.NET 6+ et .NET Framework 4.7.2)
    public DateTime DateEmbauche { get; set; } = DateTime.Now;

    // Propriété en lecture seule
    public string NomComplet => $"{Prénom} {Nom}";  // Expression-bodied member (C# 6.0+)

    // Propriété calculée
    public int AnnéesService
    {
        get
        {
            return (DateTime.Now - DateEmbauche).Days / 365;
        }
    }

    // Accesseurs avec différents niveaux d'accès
    public string Département { get; private set; } = "Non assigné";

    // Propriété avec backing field explicite et logique personnalisée
    private decimal _salaire;
    public decimal Salaire
    {
        get => _salaire;
        set
        {
            if (value < 0)
                throw new ArgumentException("Le salaire ne peut pas être négatif");

            var ancienSalaire = _salaire;
            _salaire = value;

            // Déclencher un événement, logger, etc.
            Console.WriteLine($"Salaire modifié: {ancienSalaire} -> {value}");
        }
    }
}
```


### Méthodes

Les méthodes définissent les comportements de la classe :

```csharp
public class Calculatrice
{
    // Méthode simple
    public int Additionner(int a, int b)
    {
        return a + b;
    }

    // Méthode avec expression-bodied (C# 6.0+)
    public int Multiplier(int a, int b) => a * b;

    // Méthode avec paramètres optionnels
    public decimal CalculerTaxe(decimal montant, decimal taux = 0.2m)
    {
        return montant * taux;
    }

    // Méthode avec paramètres nommés
    public string Formater(string valeur, bool majuscules = false, bool ajouterPréfixe = false)
    {
        if (majuscules)
            valeur = valeur.ToUpper();

        if (ajouterPréfixe)
            valeur = "PREFIX_" + valeur;

        return valeur;
    }

    // Méthode avec nombre variable d'arguments (params)
    public int Somme(params int[] nombres)
    {
        int résultat = 0;
        foreach (var n in nombres)
            résultat += n;
        return résultat;
    }

    // Méthode avec paramètres par référence
    public void PermutterValeurs(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }

    // Méthode avec paramètre de sortie
    public bool TryParse(string entrée, out int résultat)
    {
        return int.TryParse(entrée, out résultat);
    }

    // Méthode avec tuple (C# 7.0+)
    public (int Min, int Max) TrouverExtrêmes(int[] nombres)
    {
        if (nombres.Length == 0)
            return (0, 0);

        return (nombres.Min(), nombres.Max());
    }

    // Méthode asynchrone
    public async Task<string> ChargeDonnéesAsync(string url)
    {
        using (var client = new HttpClient())
        {
            return await client.GetStringAsync(url);
        }
    }
}
```


### Indexeurs

Les indexeurs permettent d'accéder aux éléments d'une classe comme s'il s'agissait d'un tableau :

```csharp
public class MaCollection
{
    private readonly string[] données = new string[10];

    // Indexeur simple
    public string this[int index]
    {
        get
        {
            if (index < 0 || index >= données.Length)
                throw new IndexOutOfRangeException();
            return données[index];
        }
        set
        {
            if (index < 0 || index >= données.Length)
                throw new IndexOutOfRangeException();
            données[index] = value;
        }
    }

    // Indexeur avec un type différent
    public string this[string clé]
    {
        get
        {
            int index = Array.IndexOf(données, clé);
            return index >= 0 ? données[index] : null;
        }
    }
}

// Utilisation
var collection = new MaCollection();
collection[0] = "Valeur1";
Console.WriteLine(collection[0]);  // Affiche "Valeur1"
Console.WriteLine(collection["Valeur1"]);  // Recherche par valeur
```


### Opérateurs

Vous pouvez surcharger les opérateurs pour vos classes :

```csharp
public struct Monnaie
{
    public decimal Montant { get; }
    public string Devise { get; }

    public Monnaie(decimal montant, string devise)
    {
        Montant = montant;
        Devise = devise;
    }

    // Surcharge de l'opérateur +
    public static Monnaie operator +(Monnaie a, Monnaie b)
    {
        if (a.Devise != b.Devise)
            throw new InvalidOperationException("Les devises doivent être identiques");

        return new Monnaie(a.Montant + b.Montant, a.Devise);
    }

    // Surcharge de l'opérateur ==
    public static bool operator ==(Monnaie a, Monnaie b)
    {
        return a.Montant == b.Montant && a.Devise == b.Devise;
    }

    // Surcharge de l'opérateur !=
    public static bool operator !=(Monnaie a, Monnaie b)
    {
        return !(a == b);
    }

    // Surcharge de l'opérateur implicite
    public static implicit operator decimal(Monnaie monnaie)
    {
        return monnaie.Montant;
    }

    // Surcharge de l'opérateur explicite
    public static explicit operator double(Monnaie monnaie)
    {
        return (double)monnaie.Montant;
    }

    // Override des méthodes Equals et GetHashCode
    public override bool Equals(object obj)
    {
        if (obj is Monnaie autre)
            return Montant == autre.Montant && Devise == autre.Devise;
        return false;
    }

    public override int GetHashCode()
    {
        return Montant.GetHashCode() ^ Devise.GetHashCode();
    }
}

// Utilisation
var euros1 = new Monnaie(10, "EUR");
var euros2 = new Monnaie(20, "EUR");
var total = euros1 + euros2;  // Résultat: 30 EUR

decimal montant = euros1;  // Conversion implicite: 10
double valeur = (double)euros2;  // Conversion explicite: 20.0
```


### Événements

Les événements permettent à une classe de notifier d'autres classes quand quelque chose se produit :

```csharp
public class StockItem
{
    private int quantité;

    // Délégué pour l'événement
    public delegate void StockChangedEventHandler(object sender, EventArgs e);

    // Déclaration de l'événement
    public event StockChangedEventHandler StockChanged;

    // Méthode pour déclencher l'événement
    protected virtual void OnStockChanged()
    {
        // Vérifier si quelqu'un écoute l'événement
        StockChanged?.Invoke(this, EventArgs.Empty);
    }

    public int Quantité
    {
        get => quantité;
        set
        {
            if (quantité != value)
            {
                quantité = value;
                OnStockChanged();  // Déclencher l'événement
            }
        }
    }
}

// Utilisation des événements
var produit = new StockItem();

// S'abonner à l'événement
produit.StockChanged += (sender, e) => {
    Console.WriteLine("Stock modifié: " + ((StockItem)sender).Quantité);
};

produit.Quantité = 10;  // Déclenche l'événement
```


Pour les événements plus complexes, vous pouvez créer une classe personnalisée dérivée de `EventArgs` :

```csharp
// Classe pour les données d'événement personnalisées
public class StockChangedEventArgs : EventArgs
{
    public int AncienneQuantité { get; }
    public int NouvelleQuantité { get; }

    public StockChangedEventArgs(int ancienneQuantité, int nouvelleQuantité)
    {
        AncienneQuantité = ancienneQuantité;
        NouvelleQuantité = nouvelleQuantité;
    }
}

public class StockItemAvancé
{
    private int quantité;

    // Utilisation du délégué générique EventHandler<T>
    public event EventHandler<StockChangedEventArgs> StockChanged;

    protected virtual void OnStockChanged(StockChangedEventArgs e)
    {
        StockChanged?.Invoke(this, e);
    }

    public int Quantité
    {
        get => quantité;
        set
        {
            if (quantité != value)
            {
                var ancienneValeur = quantité;
                quantité = value;
                OnStockChanged(new StockChangedEventArgs(ancienneValeur, quantité));
            }
        }
    }
}

// Utilisation
var produitAvancé = new StockItemAvancé();
produitAvancé.StockChanged += (sender, e) => {
    Console.WriteLine($"Stock modifié: {e.AncienneQuantité} -> {e.NouvelleQuantité}");
};

produitAvancé.Quantité = 20;  // "Stock modifié: 0 -> 20"
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Membres de classe de base | ✅ | ✅ |
| Expression-bodied members | Limité | Amélioré |
| Tuples nommés | ✅ (C# 7.0+) | ✅ |
| Propriétés à init-only | ❌ | ✅ |
| Required members | ❌ | ✅ |
| Static abstract members | ❌ | ✅ |

## 5.1.3. Membres statiques

Les membres statiques appartiennent à la classe elle-même plutôt qu'à une instance spécifique de la classe.

### Champs statiques

```csharp
public class Compteur
{
    // Champ statique - partagé par toutes les instances
    private static int nombreInstances = 0;

    // Champ d'instance - unique à chaque instance
    public int Id { get; }

    public Compteur()
    {
        // Incrémenter le compteur statique
        nombreInstances++;

        // Assigner l'ID basé sur le compteur
        Id = nombreInstances;
    }

    // Propriété statique pour accéder au compteur
    public static int NombreInstances => nombreInstances;
}

// Utilisation
var c1 = new Compteur();
var c2 = new Compteur();
var c3 = new Compteur();

Console.WriteLine(c1.Id);  // 1
Console.WriteLine(c2.Id);  // 2
Console.WriteLine(c3.Id);  // 3
Console.WriteLine(Compteur.NombreInstances);  // 3 (accès via la classe)
```


### Méthodes statiques

```csharp
public class MathUtils
{
    // Méthode statique
    public static double Carré(double valeur)
    {
        return valeur * valeur;
    }

    // Méthode d'instance
    public double CalculerAire(double longueur, double largeur)
    {
        return longueur * largeur;
    }
}

// Utilisation
double résultat = MathUtils.Carré(5);  // Appel direct via la classe (25)

var utils = new MathUtils();
double aire = utils.CalculerAire(5, 3);  // Nécessite une instance (15)
```


### Constructeurs statiques

```csharp
public class Configuration
{
    // Champs statiques
    public static string AppName { get; private set; }
    public static string Version { get; private set; }

    // Constructeur statique - exécuté une seule fois, lorsque
    // la classe est référencée la première fois
    static Configuration()
    {
        Console.WriteLine("Initialisation de la configuration...");

        // Lire depuis un fichier, une base de données, etc.
        AppName = "Mon Application";
        Version = "1.0.0";
    }

    // Méthodes statiques
    public static string ObtenirInfoVersion()
    {
        return $"{AppName} v{Version}";
    }
}

// Utilisation
// Le constructeur statique est exécuté automatiquement
// la première fois que la classe est référencée
Console.WriteLine(Configuration.ObtenirInfoVersion());
```


### Classes statiques

```csharp
// Classe statique - ne peut contenir que des membres statiques
// et ne peut pas être instanciée
public static class Utilitaires
{
    public static string FormatMonétaire(decimal montant)
    {
        return $"{montant:C2}";
    }

    public static bool EstEmailValide(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }

    public static string ObtenirDateFormatée()
    {
        return DateTime.Now.ToString("yyyy-MM-dd");
    }
}

// Utilisation
string montantFormaté = Utilitaires.FormatMonétaire(129.99m);
bool emailValide = Utilitaires.EstEmailValide("utilisateur@domaine.com");
```


### Méthodes d'extension

Les méthodes d'extension permettent d'ajouter des méthodes à des types existants sans modifier leur code source :

```csharp
// Les méthodes d'extension sont définies dans des classes statiques
public static class StringExtensions
{
    // Méthode d'extension pour le type string
    // Le premier paramètre avec 'this' indique le type étendu
    public static bool EstPalindrome(this string texte)
    {
        if (string.IsNullOrEmpty(texte))
            return false;

        texte = texte.ToLower().Replace(" ", "");
        int longueur = texte.Length;

        for (int i = 0; i < longueur / 2; i++)
        {
            if (texte[i] != texte[longueur - i - 1])
                return false;
        }

        return true;
    }

    // Autre méthode d'extension
    public static string TronquerA(this string texte, int longueur)
    {
        if (string.IsNullOrEmpty(texte) || texte.Length <= longueur)
            return texte;

        return texte.Substring(0, longueur) + "...";
    }
}

// Utilisation des méthodes d'extension
string test = "radar";
bool estPalindrome = test.EstPalindrome();  // true

string description = "Ceci est une longue description qui sera tronquée";
string résumé = description.TronquerA(20);  // "Ceci est une longue..."
```


### Classes génériques statiques

```csharp
// Classe générique statique
public static class Cache<T>
{
    // Dictionnaire statique spécifique au type
    private static Dictionary<string, T> _éléments = new Dictionary<string, T>();

    public static void AjouterOuMettreÀJour(string clé, T valeur)
    {
        _éléments[clé] = valeur;
    }

    public static bool TryGetValue(string clé, out T valeur)
    {
        return _éléments.TryGetValue(clé, out valeur);
    }

    public static void Supprimer(string clé)
    {
        _éléments.Remove(clé);
    }

    public static void Vider()
    {
        _éléments.Clear();
    }

    public static int Compteur => _éléments.Count;
}

// Utilisation
// Chaque type instancié a son propre cache
Cache<string>.AjouterOuMettreÀJour("clé1", "valeur1");
Cache<int>.AjouterOuMettreÀJour("compteur", 42);

string valeur;
if (Cache<string>.TryGetValue("clé1", out valeur))
{
    Console.WriteLine(valeur);  // "valeur1"
}

Console.WriteLine(Cache<string>.Compteur);  // 1
Console.WriteLine(Cache<int>.Compteur);    // 1
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Membres statiques de base | ✅ | ✅ |
| Champs et propriétés statiques | ✅ | ✅ |
| Méthodes statiques | ✅ | ✅ |
| Constructeurs statiques | ✅ | ✅ |
| Classes statiques | ✅ | ✅ |
| Méthodes d'extension | ✅ | ✅ |
| Classes génériques statiques | ✅ | ✅ |
| Membres abstraits statiques dans les interfaces | ❌ | ✅ |

## 5.1.4. Initialisation d'objets

C# offre plusieurs façons d'initialiser des objets, allant des constructeurs traditionnels aux initialiseurs d'objets et de propriétés.

### Constructeurs

Les constructeurs sont des méthodes spéciales qui sont appelées lors de la création d'un objet :

```csharp
public class Personne
{
    public string Nom { get; set; }
    public int Age { get; set; }

    // Constructeur par défaut
    public Personne()
    {
        Nom = "Anonyme";
        Age = 0;
    }

    // Constructeur avec paramètres
    public Personne(string nom, int age)
    {
        Nom = nom;
        Age = age;
    }

    // Constructeur qui appelle un autre constructeur
    public Personne(string nom) : this(nom, 0)
    {
        // Le corps peut ajouter des étapes d'initialisation supplémentaires
    }
}

// Utilisation
var p1 = new Personne();             // Nom = "Anonyme", Age = 0
var p2 = new Personne("Alice", 30);  // Nom = "Alice", Age = 30
var p3 = new Personne("Bob");        // Nom = "Bob", Age = 0
```


### Initialiseurs d'objets

Les initialiseurs d'objets permettent d'initialiser un objet sans appeler explicitement des constructeurs avec paramètres :

```csharp
public class Produit
{
    // Propriétés
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
    public string Catégorie { get; set; }

    // Constructeur par défaut
    public Produit()
    {
        // Valeurs par défaut
    }
}

// Utilisation des initialiseurs d'objets
var p1 = new Produit  // Le constructeur par défaut est appelé
{
    Id = 1,
    Nom = "Ordinateur portable",
    Prix = 999.99m,
    Catégorie = "Électronique"
};

// Combinaison avec un constructeur avec paramètres
public class Client
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public List<Adresse> Adresses { get; set; }

    public Client(int id)
    {
        Id = id;
        Adresses = new List<Adresse>();
    }
}

public class Adresse
{
    public string Rue { get; set; }
    public string Ville { get; set; }
    public string CodePostal { get; set; }
}

// Utilisation combinée
var client = new Client(1001)  // Appel du constructeur
{
    Nom = "Entreprise XYZ",    // Initialisation de propriété
    Adresses =                 // Initialisation de collection
    {
        new Adresse           // Initialiseur d'objet imbriqué
        {
            Rue = "123 Rue Principale",
            Ville = "Paris",
            CodePostal = "75001"
        },
        new Adresse
        {
            Rue = "456 Avenue Secondaire",
            Ville = "Lyon",
            CodePostal = "69001"
        }
    }
};
```


### Initialiseurs de collection

Les initialiseurs de collection permettent d'initialiser des collections en une seule expression :

```csharp
// Initialisation de tableau
int[] nombres = { 1, 2, 3, 4, 5 };

// Initialisation de List<T>
List<string> fruits = new List<string> { "Pomme", "Banane", "Orange" };

// Initialisation de Dictionary<TKey, TValue>
Dictionary<string, int> âges = new Dictionary<string, int>
{
    { "Alice", 30 },
    { "Bob", 25 },
    { "Charlie", 35 }
};

// Syntaxe alternative pour Dictionary (C# 6.0+)
Dictionary<string, int> scores = new Dictionary<string, int>
{
    ["Alice"] = 95,
    ["Bob"] = 87,
    ["Charlie"] = 92
};

// Initialisation d'un HashSet<T>
HashSet<char> voyelles = new HashSet<char> { 'a', 'e', 'i', 'o', 'u' };
```


### Propriétés en lecture seule avec initialisation

```csharp
public class Document
{
    // Propriété en lecture seule avec initialisation
    public string Id { get; } = Guid.NewGuid().ToString();

    // Propriété en lecture seule initialisée dans le constructeur
    public DateTime DateCréation { get; }

    // Auto-propriété calculée en lecture seule
    public bool EstExpirée => DateTime.Now > DateExpiration;

    // Propriété normale
    public DateTime DateExpiration { get; set; }

    public Document()
    {
        // L'initialisation des propriétés en lecture seule est possible dans le constructeur
        DateCréation = DateTime.Now;

        // Valeur par défaut pour la date d'expiration
        DateExpiration = DateTime.Now.AddYears(1);
    }
}

// Utilisation
var doc = new Document();
Console.WriteLine(doc.Id);            // GUID unique
Console.WriteLine(doc.DateCréation);  // Date/heure actuelle
Console.WriteLine(doc.EstExpirée);    // false (par défaut)

// Impossible de modifier une propriété en lecture seule
// doc.Id = "nouveau-id";  // Erreur de compilation
```


### Propriétés init-only (C# 9.0+, .NET 5+ / .NET 8)

Les propriétés init-only sont en lecture seule après l'initialisation, mais peuvent être définies lors de la création de l'objet :

```csharp
// Disponible uniquement dans .NET 5+ et .NET 8, pas dans .NET Framework 4.7.2
public class ImmutablePerson
{
    // Propriétés init-only
    public string Nom { get; init; }
    public int Age { get; init; }
    public DateTime DateNaissance { get; init; }

    // Constructeur par défaut
    public ImmutablePerson()
    {
    }
}

// Utilisation avec initialiseur d'objet
var personne = new ImmutablePerson
{
    Nom = "Marie",
    Age = 28,
    DateNaissance = new DateTime(1995, 5, 15)
};

// Impossible de modifier après l'initialisation
// personne.Age = 29;  // Erreur de compilation
```


### Required properties (C# 11.0+, .NET 7+ / .NET 8)

Les propriétés marquées comme `required` doivent être initialisées lors de la création de l'objet :

```csharp
// Disponible uniquement dans .NET 7+ et .NET 8, pas dans .NET Framework 4.7.2
public class Configuration
{
    // Propriétés requises
    public required string ApiKey { get; set; }
    public required string Endpoint { get; set; }

    // Propriétés optionnelles
    public int Timeout { get; set; } = 30;
    public bool EnableCache { get; set; } = true;
}

// Utilisation avec initialiseur d'objet
var config = new Configuration
{
    ApiKey = "abc123",     // Requis
    Endpoint = "api.exemple.com"  // Requis
    // Timeout et EnableCache sont optionnels
};

// Sans fournir les propriétés requises, une erreur de compilation est générée
// var configInvalide = new Configuration();  // Erreur: propriétés requises non initialisées
```


### Pattern Matching et Object Expressions (C# 9.0+, .NET 5+ / .NET 8)

Dans les versions récentes de C#, le pattern matching peut être utilisé avec des expressions d'objet :

```csharp
// Disponible uniquement dans .NET 5+ et .NET 8, pas dans .NET Framework 4.7.2
public bool EstÉligiblePourRéduction(Client client)
{
    // Pattern matching avec expression d'objet
    return client is { ÂgeClient: >= 65 } or { EstPremiereCommande: true };
}

// Utilisation avec déconstruction
public bool ValiderAdresse(Adresse adresse) =>
    adresse is { Pays: "France", CodePostal: { Length: 5 } code }
    && int.TryParse(code, out _);
```


### Initialisation et records (C# 9.0+, .NET 5+ / .NET 8)

Les records sont un type de référence qui facilite la création de classes immuables :

```csharp
// Disponible uniquement dans .NET 5+ et .NET 8, pas dans .NET Framework 4.7.2
// Définition d'un record avec des propriétés init-only
public record PersonneRecord
{
    public string Nom { get; init; }
    public int Age { get; init; }

    // Propriétés calculées
    public bool EstMajeur => Age >= 18;
}

// Record avec paramètres de constructeur (forme plus concise)
public record EmployéRecord(string Nom, string Poste, decimal Salaire);

// Utilisation
var employé = new EmployéRecord("Alice", "Développeur", 50000m);
Console.WriteLine($"{employé.Nom}, {employé.Poste}, {employé.Salaire}€");

// Création d'une copie avec modifications (immutable)
var employéPromu = employé with { Poste = "Chef de projet", Salaire = 65000m };
```


### Initialisation non-destructive avec `with` (C# 9.0+, .NET 5+ / .NET 8)

L'opérateur `with` permet de créer une copie d'un objet avec certaines propriétés modifiées :

```csharp
// Disponible uniquement dans .NET 5+ et .NET 8, pas dans .NET Framework 4.7.2
// Définition d'un record
public record Configuration(string Environnement, string ApiUrl, int Timeout);

// Utilisation de with pour la copie non-destructive
var configProduction = new Configuration("Production", "https://api.exemple.com", 30);
var configTest = configProduction with { Environnement = "Test", ApiUrl = "https://test-api.exemple.com" };

// configProduction reste inchangé
Console.WriteLine(configProduction);  // Configuration { Environnement = Production, ApiUrl = https://api.exemple.com, Timeout = 30 }
Console.WriteLine(configTest);        // Configuration { Environnement = Test, ApiUrl = https://test-api.exemple.com, Timeout = 30 }
```


### Initialisation en utilisant la construction par objet à valeur

Les structs (types de valeur) peuvent être initialisés de manière similaire aux classes :

```csharp
public struct Point
{
    public double X { get; set; }
    public double Y { get; set; }

    // Constructeur
    public Point(double x, double y)
    {
        X = x;
        Y = y;
    }

    // Méthode pour calculer la distance
    public double DistanceDeLOrigine()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
}

// Initialisation avec constructeur
Point p1 = new Point(3, 4);
Console.WriteLine(p1.DistanceDeLOrigine());  // 5

// Initialisation avec initialiseur d'objet
Point p2 = new Point { X = 5, Y = 12 };
Console.WriteLine(p2.DistanceDeLOrigine());  // 13
```


### Utilisation du constructeur dans l'assignation

```csharp
public class Journal
{
    private readonly List<string> _entrées = new List<string>();

    public void AjouterEntrée(string entrée)
    {
        _entrées.Add($"{DateTime.Now}: {entrée}");
    }

    public void AfficherJournal()
    {
        foreach (var entrée in _entrées)
        {
            Console.WriteLine(entrée);
        }
    }
}

// Initialisation et utilisation en une seule instruction
new Journal().AjouterEntrée("Initialisation du système");

// Ou avec une variable
Journal journal = new Journal();
journal.AjouterEntrée("Première entrée");
journal.AjouterEntrée("Deuxième entrée");
journal.AfficherJournal();
```


### Initialisation avancée avec les délégués et les événements

```csharp
public class Logger
{
    // Événement pour notifier des nouveaux logs
    public event EventHandler<string> LogAjouté;

    public void Log(string message)
    {
        Console.WriteLine($"LOG: {message}");
        LogAjouté?.Invoke(this, message);
    }
}

// Initialisation avec abonnement aux événements
var logger = new Logger();
logger.LogAjouté += (sender, message) => {
    File.AppendAllText("app.log", $"{DateTime.Now}: {message}\n");
};

logger.Log("Application démarrée");
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité d'initialisation | .NET Framework 4.7.2 | .NET 8 |
|---------------------------------|----------------------|--------|
| Constructeurs standard | ✅ | ✅ |
| Initialiseurs d'objets | ✅ | ✅ |
| Initialiseurs de collection | ✅ | ✅ |
| Propriétés en lecture seule | ✅ | ✅ |
| Propriétés init-only | ❌ | ✅ |
| Required properties | ❌ | ✅ |
| Pattern matching avec objets | ❌ | ✅ |
| Records | ❌ | ✅ |
| `with` expressions | ❌ | ✅ |
| Target-typed new | ❌ | ✅ |

### Bonnes pratiques pour l'initialisation d'objets

1. **Utilisez des constructeurs explicites** pour les objets qui nécessitent une initialisation complexe ou qui ont des dépendances obligatoires.

```csharp
public class DbService
{
    private readonly DbConnection _connection;
    private readonly ILogger _logger;

    // Constructeur explicite avec dépendances requises
    public DbService(DbConnection connection, ILogger logger)
    {
        _connection = connection ?? throw new ArgumentNullException(nameof(connection));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}
```


2. **Utilisez des initialiseurs d'objets** pour une initialisation simple et fluide.

```csharp
var utilisateur = new Utilisateur
{
    Nom = "Dupont",
    Email = "dupont@exemple.com",
    DateInscription = DateTime.Now
};
```


3. **Validez les données d'initialisation** pour assurer l'intégrité des objets.

```csharp
public class Produit
{
    private string _référence;
    public string Référence
    {
        get => _référence;
        set
        {
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("La référence est obligatoire");

            _référence = value;
        }
    }
}
```


4. **Utilisez des propriétés en lecture seule ou init-only** pour les objets qui ne devraient pas changer après l'initialisation.

```csharp
// .NET Framework 4.7.2
public class Commande
{
    public string NuméroCommande { get; }
    public DateTime DateCommande { get; }

    public Commande(string numéro)
    {
        NuméroCommande = numéro;
        DateCommande = DateTime.Now;
    }
}

// .NET 8
public class Commande
{
    public required string NuméroCommande { get; init; }
    public DateTime DateCommande { get; init; } = DateTime.Now;
}
```


5. **Considérez les fabriques (factory methods)** pour des scénarios d'initialisation complexes.

```csharp
public class Client
{
    public Guid Id { get; }
    public string Nom { get; set; }
    public string Email { get; set; }
    public List<Adresse> Adresses { get; }

    // Constructeur privé
    private Client(Guid id)
    {
        Id = id;
        Adresses = new List<Adresse>();
    }

    // Méthode de fabrique pour les nouveaux clients
    public static Client CréerNouveauClient(string nom, string email)
    {
        if (string.IsNullOrWhiteSpace(nom))
            throw new ArgumentException("Le nom est obligatoire");

        if (!email.Contains("@"))
            throw new ArgumentException("Email invalide");

        var client = new Client(Guid.NewGuid())
        {
            Nom = nom,
            Email = email
        };

        // Logique métier supplémentaire

        return client;
    }

    // Méthode de fabrique pour charger un client existant
    public static Client ChargerDepuisBaseDeDonnées(Guid id, string nom, string email)
    {
        return new Client(id)
        {
            Nom = nom,
            Email = email
        };
    }
}
```


6. **Utilisez les records** dans .NET 8 pour les objets immuables orientés données.

```csharp
// .NET 8 uniquement
public record InfoClient(Guid Id, string Nom, string Email);

// Utilisation
var info = new InfoClient(Guid.NewGuid(), "Martin", "martin@exemple.com");
```


## Conclusion

Les classes et les objets sont les éléments fondamentaux de la programmation orientée objet en C#. Cette section a exploré en détail comment définir des classes, déclarer leurs membres, utiliser des membres statiques et initialiser des objets de différentes façons.

Dans .NET Framework 4.7.2, vous disposez déjà d'un ensemble solide de fonctionnalités pour travailler avec les classes et les objets, y compris les constructeurs, les propriétés, les méthodes, les événements, et les initialiseurs d'objets.

.NET 8 étend ces fonctionnalités avec des améliorations significatives comme les propriétés init-only, les propriétés requises, les records, et les expressions with, qui facilitent la création d'objets immuables et l'écriture de code plus concis et plus sûr.

Que vous travailliez avec .NET Framework 4.7.2 ou .NET 8, une bonne compréhension des concepts de base des classes et des objets vous permettra d'écrire un code C# plus robuste, plus maintenable et plus expressif.

⏭️ 5.2. [Propriétés et encapsulation](/05-programmation-orientee-objet-en-csharp/5-2-proprietes-et-encapsulation.md)
