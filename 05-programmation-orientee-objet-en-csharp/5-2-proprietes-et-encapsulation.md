# 5.2. Propriétés et encapsulation

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'encapsulation est l'un des principes fondamentaux de la programmation orientée objet, permettant de masquer les détails d'implémentation d'une classe et d'exposer uniquement les fonctionnalités nécessaires. En C#, les propriétés fournissent un mécanisme élégant pour implémenter l'encapsulation.

## 5.2.1. Getters et setters

Les getters et setters contrôlent l'accès aux champs d'une classe, permettant d'ajouter de la logique lors de la lecture ou de l'écriture des valeurs.

### Propriétés et champs privés

```csharp
public class Personne
{
    // Champ privé (convention: débute par _)
    private string _nom;

    // Propriété avec getter et setter
    public string Nom
    {
        get
        {
            // Logique exécutée lors de la lecture de la propriété
            return _nom ?? "Anonyme";
        }
        set
        {
            // Logique exécutée lors de l'écriture de la propriété
            if (!string.IsNullOrWhiteSpace(value))
            {
                _nom = value;
            }
            else
            {
                throw new ArgumentException("Le nom ne peut pas être vide.");
            }
        }
    }
}

// Utilisation
var personne = new Personne();
personne.Nom = "Jean Dupont";  // Appelle le setter
Console.WriteLine(personne.Nom);  // Appelle le getter, affiche "Jean Dupont"

try
{
    personne.Nom = "";  // Lance une exception
}
catch (ArgumentException ex)
{
    Console.WriteLine(ex.Message);  // "Le nom ne peut pas être vide."
}
```


### Validation de données

Les propriétés sont idéales pour valider les données avant de les assigner à un champ :

```csharp
public class Produit
{
    private string _référence;
    private decimal _prix;

    public string Référence
    {
        get => _référence;
        set
        {
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("La référence du produit est obligatoire.");

            _référence = value;
        }
    }

    public decimal Prix
    {
        get => _prix;
        set
        {
            if (value < 0)
                throw new ArgumentException("Le prix ne peut pas être négatif.");

            _prix = value;
        }
    }
}

// Utilisation
var produit = new Produit();
produit.Référence = "REF001";
produit.Prix = 19.99m;

// Exception si validation échoue
// produit.Prix = -5;  // ArgumentException: Le prix ne peut pas être négatif.
```


### Niveau d'accès des accesseurs

Les accesseurs get et set peuvent avoir des niveaux d'accès différents :

```csharp
public class Compte
{
    private decimal _solde;

    // Propriété avec accesseur public pour get et private pour set
    public decimal Solde
    {
        get => _solde;
        private set => _solde = value;
    }

    // Méthode pour modifier le solde
    public void Déposer(decimal montant)
    {
        if (montant <= 0)
            throw new ArgumentException("Le montant doit être positif.");

        Solde += montant;  // Utilise le setter privé
    }

    public bool Retirer(decimal montant)
    {
        if (montant <= 0)
            throw new ArgumentException("Le montant doit être positif.");

        if (montant > Solde)
            return false;  // Solde insuffisant

        Solde -= montant;  // Utilise le setter privé
        return true;
    }
}

// Utilisation
var compte = new Compte();
compte.Déposer(1000);
Console.WriteLine(compte.Solde);  // 1000

bool succès = compte.Retirer(500);
Console.WriteLine(succès);       // true
Console.WriteLine(compte.Solde);  // 500

// compile.Solde = 2000;  // Erreur de compilation: le setter est privé
```


Autres exemples de modificateurs d'accès pour les accesseurs :

```csharp
public class Entité
{
    // get public, set protected
    public Guid Id { get; protected set; } = Guid.NewGuid();

    // get public, set internal
    public DateTime DernièreModification { get; internal set; }

    // get protected, set protected
    protected string InformationInterne { get; set; }
}

// Classe dérivée
public class UtilisateurEntité : Entité
{
    public UtilisateurEntité()
    {
        // Peut accéder au setter protected
        InformationInterne = "Info confidentielle";

        // Peut accéder au setter protected de Id
        Id = Guid.NewGuid();
    }

    public void MettreÀJour()
    {
        // Peut lire la propriété protected
        Console.WriteLine(InformationInterne);
    }
}

// Utilisation
var entité = new Entité();
Console.WriteLine(entité.Id);  // Peut lire l'Id
// entité.Id = Guid.NewGuid();  // Erreur: setter est protected

var utilisateur = new UtilisateurEntité();
// Console.WriteLine(utilisateur.InformationInterne);  // Erreur: getter est protected
```


### Notification de changement

Les propriétés peuvent déclencher des événements ou des actions quand leurs valeurs changent :

```csharp
public class ElementObservable
{
    private string _nom;

    // Événement déclenché quand une propriété change
    public event PropertyChangedEventHandler PropertyChanged;

    // Méthode protégée pour déclencher l'événement
    protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    public string Nom
    {
        get => _nom;
        set
        {
            if (_nom != value)
            {
                _nom = value;
                OnPropertyChanged();  // Notifie que Nom a changé
            }
        }
    }
}

// Utilisation
var element = new ElementObservable();
element.PropertyChanged += (sender, e) => {
    Console.WriteLine($"La propriété {e.PropertyName} a changé.");
};

element.Nom = "Nouvel élément";  // Affiche: "La propriété Nom a changé."
```


### Propriétés calculées

Les propriétés peuvent être calculées à partir d'autres propriétés ou champs :

```csharp
public class Rectangle
{
    // Propriétés stockées
    public double Longueur { get; set; }
    public double Largeur { get; set; }

    // Propriété calculée
    public double Surface
    {
        get { return Longueur * Largeur; }
    }

    // Propriété calculée avec logique supplémentaire
    public string Catégorie
    {
        get
        {
            double surface = Surface;
            if (surface < 10) return "Petit";
            if (surface < 100) return "Moyen";
            return "Grand";
        }
    }
}

// Utilisation
var rectangle = new Rectangle { Longueur = 5, Largeur = 3 };
Console.WriteLine($"Surface: {rectangle.Surface}");        // 15
Console.WriteLine($"Catégorie: {rectangle.Catégorie}");   // "Moyen"
```


### Propriétés indexées

Les propriétés indexées permettent d'accéder aux éléments comme s'il s'agissait d'un tableau :

```csharp
public class MaListe<T>
{
    private readonly List<T> _éléments = new List<T>();

    // Indexeur - propriété spéciale à indice
    public T this[int index]
    {
        get
        {
            if (index < 0 || index >= _éléments.Count)
                throw new IndexOutOfRangeException();

            return _éléments[index];
        }
        set
        {
            if (index < 0)
                throw new IndexOutOfRangeException();

            // Extension automatique de la liste si nécessaire
            while (index >= _éléments.Count)
                _éléments.Add(default);

            _éléments[index] = value;
        }
    }

    // Propriété pour connaître le nombre d'éléments
    public int Nombre => _éléments.Count;
}

// Utilisation
var liste = new MaListe<string>();
liste[0] = "Premier";
liste[1] = "Deuxième";
liste[5] = "Sixième";  // Les indices 2, 3, 4 sont automatiquement initialisés à null

Console.WriteLine(liste[0]);      // "Premier"
Console.WriteLine(liste[3]);      // null
Console.WriteLine(liste.Nombre);  // 6
```


## 5.2.2. Propriétés auto-implémentées

Les propriétés auto-implémentées simplifient la syntaxe en générant automatiquement un champ privé sous-jacent.

### Syntaxe de base

```csharp
public class Client
{
    // Propriété auto-implémentée - le compilateur génère le champ privé
    public string Nom { get; set; }
    public string Email { get; set; }
    public DateTime DateInscription { get; set; }

    // Équivalent explicite (à titre de comparaison)
    private string _téléphone;
    public string Téléphone
    {
        get { return _téléphone; }
        set { _téléphone = value; }
    }
}

// Utilisation
var client = new Client
{
    Nom = "Marie Dupont",
    Email = "marie@exemple.com",
    DateInscription = DateTime.Now
};
```


### Propriétés auto-implémentées avec valeurs par défaut

Depuis C# 6.0, vous pouvez initialiser directement les propriétés auto-implémentées :

```csharp
public class Paramètres
{
    // Propriétés auto-implémentées avec valeurs par défaut
    public bool EstActif { get; set; } = true;
    public int NiveauVolume { get; set; } = 75;
    public string Thème { get; set; } = "Clair";
    public List<string> Favoris { get; set; } = new List<string>();
    public DateTime DernièreConnexion { get; set; } = DateTime.Now;
}

// Utilisation
var paramètres = new Paramètres();
Console.WriteLine(paramètres.EstActif);  // true (valeur par défaut)
Console.WriteLine(paramètres.Thème);     // "Clair" (valeur par défaut)

// Modification
paramètres.Thème = "Sombre";
paramètres.Favoris.Add("Page d'accueil");
```


### Propriétés auto-implémentées avec niveaux d'accès différents

```csharp
public class Document
{
    // get public, set public
    public string Titre { get; set; }

    // get public, set private
    public string NuméroRévision { get; private set; } = "1.0";

    // get public, set internal
    public DateTime DateCréation { get; internal set; } = DateTime.Now;

    // get public, set protected
    public bool EstArchivé { get; protected set; }

    public void IncrémenterRévision()
    {
        // Accès au setter privé
        string[] parties = NuméroRévision.Split('.');
        int majeur = int.Parse(parties[0]);
        int mineur = int.Parse(parties[1]);

        mineur++;
        if (mineur > 9)
        {
            majeur++;
            mineur = 0;
        }

        NuméroRévision = $"{majeur}.{mineur}";
    }
}

// Classe dérivée
public class RapportDocument : Document
{
    public void Archiver()
    {
        // Accès au setter protected
        EstArchivé = true;
    }
}

// Utilisation
var doc = new Document();
doc.Titre = "Mon Document";

doc.IncrémenterRévision();
Console.WriteLine(doc.NuméroRévision);  // "1.1"

// doc.NuméroRévision = "2.0";  // Erreur: setter est privé
// doc.EstArchivé = true;       // Erreur: setter est protected

var rapport = new RapportDocument();
rapport.Archiver();
Console.WriteLine(rapport.EstArchivé);  // true
```


## 5.2.3. Propriétés en lecture seule

Les propriétés en lecture seule ne peuvent être définies que pendant l'initialisation ou dans le constructeur de la classe.

### Propriétés auto-implémentées en lecture seule

```csharp
public class Utilisateur
{
    // Propriété en lecture seule - peut être définie uniquement dans le constructeur
    public Guid Id { get; }

    // Propriétés en lecture/écriture normales
    public string Nom { get; set; }
    public string Email { get; set; }

    public Utilisateur()
    {
        // Initialisation de la propriété en lecture seule
        Id = Guid.NewGuid();
    }

    public Utilisateur(Guid id)
    {
        // Paramètre pour initialiser la propriété en lecture seule
        Id = id;
    }
}

// Utilisation
var utilisateur1 = new Utilisateur();
var utilisateur2 = new Utilisateur(Guid.Parse("7b8c9d0e-1f2a-3b4c-5d6e-7f8a9b0c1d2e"));

Console.WriteLine(utilisateur1.Id);  // GUID généré aléatoirement
Console.WriteLine(utilisateur2.Id);  // GUID spécifié

// utilisateur1.Id = Guid.NewGuid();  // Erreur: propriété en lecture seule
```


### Propriétés calculées en lecture seule

```csharp
public class Employé
{
    // Propriétés classiques
    public string Prénom { get; set; }
    public string Nom { get; set; }
    public decimal SalaireAnnuel { get; set; }

    // Propriété calculée en lecture seule
    public string NomComplet
    {
        get { return $"{Prénom} {Nom}"; }
    }

    // Autre propriété calculée en lecture seule
    public decimal SalaireMensuel
    {
        get { return SalaireAnnuel / 12; }
    }
}

// Utilisation
var employé = new Employé
{
    Prénom = "Thomas",
    Nom = "Martin",
    SalaireAnnuel = 60000
};

Console.WriteLine(employé.NomComplet);    // "Thomas Martin"
Console.WriteLine(employé.SalaireMensuel); // 5000

// employé.NomComplet = "Paul Dupont";  // Erreur: propriété en lecture seule
```


### Propriétés en lecture seule avec backing field

```csharp
public class Article
{
    // Champ privé
    private readonly string _référence;

    // Propriété en lecture seule avec implementation personnalisée
    public string Référence
    {
        get { return _référence; }
    }

    public string Nom { get; set; }
    public decimal Prix { get; set; }

    public Article(string référence)
    {
        if (string.IsNullOrWhiteSpace(référence))
            throw new ArgumentException("La référence est obligatoire");

        _référence = référence.ToUpper();
    }
}

// Utilisation
var article = new Article("ref001");
Console.WriteLine(article.Référence);  // "REF001"

// article.Référence = "ref002";  // Erreur: propriété en lecture seule
```


### Propriétés en lecture seule avec initialisation

```csharp
public class Configuration
{
    // Propriété en lecture seule avec initialisation
    public string Version { get; } = "1.0.0";

    // Liste en lecture seule, mais son contenu est modifiable
    public List<string> ModulesActifs { get; } = new List<string>();

    // Collection en lecture seule non modifiable
    public IReadOnlyList<string> OptionsDisponibles { get; }
        = new List<string> { "Option1", "Option2", "Option3" }
          .AsReadOnly();
}

// Utilisation
var config = new Configuration();
Console.WriteLine(config.Version);  // "1.0.0"

// config.Version = "2.0.0";  // Erreur: propriété en lecture seule

// La liste elle-même ne peut pas être remplacée, mais son contenu peut être modifié
config.ModulesActifs.Add("Module1");
config.ModulesActifs.Add("Module2");

// La collection en lecture seule ne peut pas être modifiée
// config.OptionsDisponibles.Add("Option4");  // Erreur: méthode Add non disponible
```


## 5.2.4. Expression-bodied properties (C# 6+)

Les propriétés à corps d'expression (expression-bodied) offrent une syntaxe plus concise pour les propriétés dont l'implémentation est une expression simple.

### Propriétés calculées à corps d'expression

```csharp
public class Cercle
{
    public double Rayon { get; set; }

    // Propriété calculée avec corps d'expression (get uniquement)
    public double Diamètre => 2 * Rayon;

    // Propriété calculée avec corps d'expression
    public double Circonférence => 2 * Math.PI * Rayon;

    // Propriété calculée avec corps d'expression
    public double Surface => Math.PI * Rayon * Rayon;
}

// Utilisation
var cercle = new Cercle { Rayon = 5 };
Console.WriteLine($"Diamètre: {cercle.Diamètre}");      // 10
Console.WriteLine($"Circonférence: {cercle.Circonférence}");  // ~31.4159
Console.WriteLine($"Surface: {cercle.Surface}");        // ~78.5398
```


### Getters et setters à corps d'expression (C# 7+)

À partir de C# 7.0, les accesseurs get et set peuvent également être définis avec la syntaxe à corps d'expression :

```csharp
public class Température
{
    private double _celsius;

    // Propriété avec accesseurs à corps d'expression
    public double Celsius
    {
        get => _celsius;
        set => _celsius = value;
    }

    // Propriétés calculées avec corps d'expression
    public double Fahrenheit
    {
        get => _celsius * 9 / 5 + 32;
        set => _celsius = (value - 32) * 5 / 9;
    }

    public double Kelvin
    {
        get => _celsius + 273.15;
        set => _celsius = value - 273.15;
    }
}

// Utilisation
var temp = new Température();
temp.Celsius = 25;
Console.WriteLine($"Fahrenheit: {temp.Fahrenheit}");  // 77
Console.WriteLine($"Kelvin: {temp.Kelvin}");          // 298.15

temp.Fahrenheit = 68;
Console.WriteLine($"Celsius: {temp.Celsius}");        // 20
```


### Combinaison avec des propriétés auto-implémentées et la déclaration de types

```csharp
public class GPS
{
    // Propriétés auto-implémentées
    public double Latitude { get; set; }
    public double Longitude { get; set; }

    // Propriété calculée avec corps d'expression et type explicite
    public string Coordonnées => $"{Latitude}°, {Longitude}°";

    // Propriété calculée avec corps d'expression et logique conditionnelle
    public string Hémisphère => Latitude >= 0 ? "Nord" : "Sud";

    // Méthode à corps d'expression qui utilise les propriétés
    public double CalculerDistanceDe(GPS autre) =>
        Math.Sqrt(Math.Pow(Latitude - autre.Latitude, 2) +
                 Math.Pow(Longitude - autre.Longitude, 2));
}

// Utilisation
var position1 = new GPS { Latitude = 48.8566, Longitude = 2.3522 };
var position2 = new GPS { Latitude = 40.7128, Longitude = -74.0060 };

Console.WriteLine(position1.Coordonnées);  // "48.8566°, 2.3522°"
Console.WriteLine(position1.Hémisphère);   // "Nord"
Console.WriteLine(position2.Hémisphère);   // "Nord"

double distance = position1.CalculerDistanceDe(position2);
Console.WriteLine($"Distance: {distance}");  // Distance approximative
```


## 5.2.5. Init-only setters (C# 9+)

Les setters init-only, introduits dans C# 9.0, permettent de créer des propriétés qui ne peuvent être modifiées qu'au moment de l'initialisation de l'objet, offrant ainsi une meilleure immutabilité.

> **Note**: Cette fonctionnalité est disponible seulement dans .NET 5+ et .NET 8, mais pas dans .NET Framework 4.7.2.

### Propriétés auto-implémentées init-only

```csharp
// Disponible uniquement dans .NET 8, pas dans .NET Framework 4.7.2
public class PersonneImmutable
{
    // Propriétés init-only
    public string Nom { get; init; }
    public string Prénom { get; init; }
    public DateTime DateNaissance { get; init; }

    // Propriété calculée basée sur des propriétés init-only
    public int Age => DateTime.Now.Year - DateNaissance.Year -
        (DateTime.Now.DayOfYear < DateNaissance.DayOfYear ? 1 : 0);

    // Propriété en lecture/écriture normale
    public string Adresse { get; set; }
}

// Utilisation
var personne = new PersonneImmutable
{
    Nom = "Dupont",
    Prénom = "Jean",
    DateNaissance = new DateTime(1980, 5, 10),
    Adresse = "123 Rue de Paris"
};

// Après initialisation, les propriétés init-only ne peuvent plus être modifiées
// personne.Nom = "Martin";  // Erreur: propriété init-only
// personne.Prénom = "Pierre";  // Erreur: propriété init-only

// Mais les propriétés en lecture/écriture peuvent être modifiées
personne.Adresse = "456 Avenue des Champs-Élysées";
```


### Combinaison avec des champs en lecture seule

```csharp
// Disponible uniquement dans .NET 8, pas dans .NET Framework 4.7.2
public class Commande
{
    // Propriétés init-only pour l'identité de la commande
    public string NuméroCommande { get; init; }
    public DateTime DateCommande { get; init; } = DateTime.Now;
    public string ClientId { get; init; }

    // Liste d'éléments de commande - la liste elle-même est en lecture seule,
    // mais son contenu peut être modifié
    private readonly List<ElementCommande> _éléments = new List<ElementCommande>();

    // Propriété qui expose la liste en tant que collection en lecture seule
    public IReadOnlyCollection<ElementCommande> Éléments => _éléments.AsReadOnly();

    // Méthode pour ajouter des éléments
    public void AjouterÉlément(ElementCommande élément)
    {
        _éléments.Add(élément);
    }

    // Propriété calculée basée sur le contenu de la liste
    public decimal Total => _éléments.Sum(e => e.Prix * e.Quantité);
}

public class ElementCommande
{
    // Propriétés init-only
    public string ProduitId { get; init; }
    public string Description { get; init; }
    public decimal Prix { get; init; }

    // Propriété modifiable
    public int Quantité { get; set; }

    // Propriété calculée
    public decimal SousTotal => Prix * Quantité;
}

// Utilisation
var commande = new Commande
{
    NuméroCommande = "CMD-2023-001",
    ClientId = "CLT-1234"
};

var élément1 = new ElementCommande
{
    ProduitId = "PRD-001",
    Description = "Laptop XPS 13",
    Prix = 1299.99m,
    Quantité = 1
};

var élément2 = new ElementCommande
{
    ProduitId = "PRD-002",
    Description = "Souris sans fil",
    Prix = 49.99m,
    Quantité = 2
};

commande.AjouterÉlément(élément1);
commande.AjouterÉlément(élément2);

// Modification permise
élément2.Quantité = 3;

// Modifications interdites
// commande.NuméroCommande = "CMD-2023-002";  // Erreur: propriété init-only
// élément1.Prix = 1199.99m;  // Erreur: propriété init-only

Console.WriteLine($"Total de la commande: {commande.Total}€");  // ~1449.96
```


### Création d'objets immuables

```csharp
// Disponible uniquement dans .NET 8, pas dans .NET Framework 4.7.2
public class Configuration
{
    // Propriétés init-only
    public string Environnement { get; init; }
    public string ChaîneConnexion { get; init; }
    public int TempsExpiration { get; init; }
    public bool EnableCaching { get; init; }

    // Collection immuable
    public IReadOnlyDictionary<string, string> Settings { get; init; }
}

// Utilisation
var configProd = new Configuration
{
    Environnement = "Production",
    ChaîneConnexion = "Server=prod;Database=app;",
    TempsExpiration = 3600,
    EnableCaching = true,
    Settings = new Dictionary<string, string>
    {
        ["LogLevel"] = "Warning",
        ["MaxConnections"] = "100"
    }
};

// Pour créer une variante, il faut créer un nouvel objet
var configTest = new Configuration
{
    Environnement = "Test",
    ChaîneConnexion = "Server=test;Database=app_test;",
    TempsExpiration = configProd.TempsExpiration,
    EnableCaching = configProd.EnableCaching,
    Settings = new Dictionary<string, string>(configProd.Settings)
};
```


### Comparaison avec les records (C# 9+)

Les records fournissent une approche encore plus concise pour définir des types immuables :

```csharp
// Disponible uniquement dans .NET 8, pas dans .NET Framework 4.7.2

// Record avec propriétés init-only
public record PersonneRecord
{
    public string Nom { get; init; }
    public string Prénom { get; init; }
    public DateTime DateNaissance { get; init; }
}

// Syntaxe positionnelle plus concise pour les records
public record AdresseRecord(string Rue, string Ville, string CodePostal, string Pays);

// Utilisation
var personne = new PersonneRecord
{
    Nom = "Dupont",
    Prénom = "Jean",
    DateNaissance = new DateTime(1980, 5, 10)
};

var adresse = new AdresseRecord("123 Rue de Paris", "Paris", "75001", "France");

// Création d'une copie avec modifications (avec l'opérateur with)
var personne2 = personne with { Nom = "Martin" };

Console.WriteLine($"{personne2.Prénom} {personne2.Nom}");  // Jean Martin
```


### Modification d'objets avec propriétés init-only

Pour modifier un objet avec des propriétés init-only, vous devez créer une copie. Avec .NET 8 et C# 9+, l'opérateur `with` permet de le faire facilement pour les records :

```csharp
// Disponible uniquement dans .NET 8, pas dans .NET Framework 4.7.2
public record ConfigRecord(string Environnement, string ApiUrl, int Timeout);

// Utilisation
var configDev = new ConfigRecord("Development", "https://dev-api.exemple.com", 30);

// Création d'une copie avec modifications
var configProd = configDev with
{
    Environnement = "Production",
    ApiUrl = "https://api.exemple.com"
};

Console.WriteLine($"Config Dev: {configDev}");
// ConfigRecord { Environnement = Development, ApiUrl = https://dev-api.exemple.com, Timeout = 30 }

Console.WriteLine($"Config Prod: {configProd}");
// ConfigRecord { Environnement = Production, ApiUrl = https://api.exemple.com, Timeout = 30 }
```


### Alternative dans .NET Framework 4.7.2

Bien que les propriétés init-only ne soient pas disponibles dans .NET Framework 4.7.2, vous pouvez obtenir un comportement similaire en utilisant des propriétés en lecture seule avec une initialisation dans le constructeur :

```csharp
// Alternative pour .NET Framework 4.7.2
public class PersonneImmutable
{
    // Propriétés en lecture seule
    public string Nom { get; }
    public string Prénom { get; }
    public DateTime DateNaissance { get; }

    // Propriété calculée
    public int Age => DateTime.Now.Year - DateNaissance.Year -
        (DateTime.Now.DayOfYear < DateNaissance.DayOfYear ? 1 : 0);

    // Propriété modifiable
    public string Adresse { get; set; }

    // Constructeur qui initialise les propriétés en lecture seule
    public PersonneImmutable(string nom, string prénom, DateTime dateNaissance)
    {
        Nom = nom;
        Prénom = prénom;
        DateNaissance = dateNaissance;
    }

    // Méthode de fabrique pour créer une copie avec modifications
    public PersonneImmutable AvecNom(string nouveauNom)
    {
        return new PersonneImmutable(nouveauNom, this.Prénom, this.DateNaissance)
        {
            Adresse = this.Adresse
        };
    }

    public PersonneImmutable AvecPrénom(string nouveauPrénom)
    {
        return new PersonneImmutable(this.Nom, nouveauPrénom, this.DateNaissance)
        {
            Adresse = this.Adresse
        };
    }
}

// Utilisation
var personne = new PersonneImmutable("Dupont", "Jean", new DateTime(1980, 5, 10))
{
    Adresse = "123 Rue de Paris"
};

// Création d'une copie avec un nom modifié
var personne2 = personne.AvecNom("Martin");

Console.WriteLine($"{personne2.Prénom} {personne2.Nom}");  // Jean Martin
```


Ou en utilisant un pattern Builder plus complexe :

```csharp
// Pattern Builder pour .NET Framework 4.7.2
public class PersonneImmutable
{
    // Propriétés en lecture seule
    public string Nom { get; }
    public string Prénom { get; }
    public DateTime DateNaissance { get; }
    public string Adresse { get; }

    // Constructeur privé
    private PersonneImmutable(string nom, string prénom, DateTime dateNaissance, string adresse)
    {
        Nom = nom;
        Prénom = prénom;
        DateNaissance = dateNaissance;
        Adresse = adresse;
    }

    // Classe Builder interne
    public class Builder
    {
        private string _nom;
        private string _prénom;
        private DateTime _dateNaissance;
        private string _adresse;

        public Builder() { }

        public Builder(PersonneImmutable personne)
        {
            _nom = personne.Nom;
            _prénom = personne.Prénom;
            _dateNaissance = personne.DateNaissance;
            _adresse = personne.Adresse;
        }

        public Builder Nom(string nom)
        {
            _nom = nom;
            return this;
        }

        public Builder Prénom(string prénom)
        {
            _prénom = prénom;
            return this;
        }

        public Builder DateNaissance(DateTime dateNaissance)
        {
            _dateNaissance = dateNaissance;
            return this;
        }

        public Builder Adresse(string adresse)
        {
            _adresse = adresse;
            return this;
        }

        public PersonneImmutable Build()
        {
            return new PersonneImmutable(_nom, _prénom, _dateNaissance, _adresse);
        }
    }
}

// Utilisation
var personne = new PersonneImmutable.Builder()
    .Nom("Dupont")
    .Prénom("Jean")
    .DateNaissance(new DateTime(1980, 5, 10))
    .Adresse("123 Rue de Paris")
    .Build();

// Création d'une copie avec modifications
var personne2 = new PersonneImmutable.Builder(personne)
    .Nom("Martin")
    .Build();
```


## Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Propriétés de base avec getters et setters | ✅ | ✅ |
| Propriétés auto-implémentées | ✅ | ✅ |
| Propriétés auto-implémentées avec initialisation | ✅ | ✅ |
| Propriétés en lecture seule | ✅ | ✅ |
| Expression-bodied properties (get only) | ✅ (C# 6.0+) | ✅ |
| Expression-bodied accessors (get/set) | ✅ (C# 7.0+) | ✅ |
| Init-only setters | ❌ | ✅ |
| Records | ❌ | ✅ |
| Required members | ❌ | ✅ |

## Bonnes pratiques

### Encapsulation

1. **Utilisez des propriétés plutôt que des champs publics**
   Les propriétés permettent de modifier l'implémentation sans casser le code client.

```csharp
// Bon
   public class Client { public string Nom { get; set; } }

   // À éviter
   public class Client { public string Nom; }
```


2. **Validez les données dans les setters**
   Assurez-vous que les propriétés ne contiennent que des valeurs valides.

```csharp
public string Email
   {
       get => _email;
       set
       {
           if (!value.Contains("@"))
               throw new ArgumentException("Email invalide");
           _email = value;
       }
   }
```


3. **Utilisez les niveaux d'accès appropriés**
   Exposez uniquement ce qui est nécessaire.

```csharp
// Bonne pratique: setter privé pour contrôler les modifications
   public Guid Id { get; private set; }
```


### Immutabilité

1. **Utilisez des propriétés en lecture seule ou init-only pour les données qui ne devraient pas changer**

```csharp
// .NET Framework 4.7.2
   public Guid Id { get; }

   // .NET 8
   public Guid Id { get; init; }
```


2. **Rendez les collections exposées en lecture seule**

```csharp
private readonly List<string> _tags = new List<string>();
   public IReadOnlyCollection<string> Tags => _tags.AsReadOnly();
```


### Conception

1. **Gardez les propriétés simples et focalisées**
   Chaque propriété devrait avoir une responsabilité unique.

2. **Utilisez des noms descriptifs et cohérents**
   Suivez les conventions de nommage C# (PascalCase pour les propriétés).

3. **Limitez les effets secondaires dans les accesseurs**
   Les getters devraient être sans effet secondaire, les setters devraient se limiter à valider et stocker.

4. **Évitez les propriétés calculées coûteuses sans mise en cache**
   Si le calcul est lourd, envisagez de mettre en cache le résultat.

```csharp
private decimal? _moyenne;

   public decimal Moyenne
   {
       get
       {
           if (!_moyenne.HasValue)
               _moyenne = Éléments.Average(e => e.Valeur);
           return _moyenne.Value;
       }
   }

   public void AjouterÉlément(Élément e)
   {
       Éléments.Add(e);
       _moyenne = null;  // Invalider le cache
   }
```


5. **Envisagez le pattern Builder pour les objets complexes avec plusieurs propriétés init-only**
   Le pattern Builder simplifie la création d'objets immutables avec de nombreuses propriétés.

## Conclusion

Les propriétés et l'encapsulation sont des concepts fondamentaux en C# qui vous permettent de créer un code robuste et maintenable. Les propriétés offrent un moyen élégant de contrôler l'accès aux données d'une classe, tout en permettant la validation, le calcul dynamique et la notification de changements.

Dans .NET Framework 4.7.2, vous disposez déjà d'outils puissants comme les getters et setters personnalisés, les propriétés auto-implémentées, les propriétés en lecture seule et les expression-bodied properties pour implémenter efficacement l'encapsulation.

.NET 8 et les versions récentes de C# ont apporté des améliorations importantes avec les propriétés init-only et les records, qui facilitent la création d'objets immuables et simplifient la syntaxe pour des scénarios courants.

Quelle que soit la version que vous utilisez, l'utilisation appropriée des propriétés et de l'encapsulation reste une pratique essentielle pour créer des applications C# bien structurées et maintenables.

⏭️
