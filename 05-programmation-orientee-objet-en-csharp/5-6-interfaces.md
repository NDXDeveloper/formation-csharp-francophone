# 5.6. Interfaces

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les interfaces sont un concept fondamental en C# qui permettent de définir un contrat que les classes doivent respecter. Contrairement aux classes abstraites, les interfaces ne contiennent que des déclarations de méthodes, propriétés, événements et indexeurs, sans implémentation (sauf pour les interfaces par défaut depuis C# 8).

## 5.6.1. Définition et implémentation

Une interface définit un ensemble de membres que toute classe ou structure implémentant cette interface doit fournir.

```csharp
// Définition d'une interface
public interface IVehicule
{
    // Propriétés (sans implémentation)
    string Modele { get; set; }
    int AnneeProduction { get; }

    // Méthodes (sans implémentation)
    void Demarrer();
    void Arreter();

    // Les méthodes et propriétés sont implicitement publiques dans une interface
    // Donc pas besoin du mot-clé 'public'
}

// Implémentation de l'interface
public class Voiture : IVehicule
{
    // Implémentation des propriétés de l'interface
    public string Modele { get; set; }
    public int AnneeProduction { get; private set; }

    // Constructeur
    public Voiture(string modele, int anneeProduction)
    {
        Modele = modele;
        AnneeProduction = anneeProduction;
    }

    // Implémentation des méthodes de l'interface
    public void Demarrer()
    {
        Console.WriteLine($"La voiture {Modele} démarre.");
    }

    public void Arreter()
    {
        Console.WriteLine($"La voiture {Modele} s'arrête.");
    }

    // Méthodes supplémentaires qui ne sont pas dans l'interface
    public void Klaxonner()
    {
        Console.WriteLine("Tutut!");
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        // Instanciation d'une voiture
        Voiture maVoiture = new Voiture("Tesla Model 3", 2023);
        maVoiture.Demarrer();
        maVoiture.Klaxonner();
        maVoiture.Arreter();

        // Utilisation via l'interface
        IVehicule vehicule = maVoiture;
        vehicule.Demarrer();
        // vehicule.Klaxonner(); // Erreur de compilation - cette méthode n'est pas dans l'interface
        vehicule.Arreter();
    }
}
```


### Points clés :
- Les interfaces commencent généralement par un "I" (convention)
- Tous les membres d'une interface sont implicitement publics
- Une classe doit implémenter tous les membres déclarés dans l'interface
- Les interfaces permettent le polymorphisme (comme dans l'exemple ci-dessus avec `IVehicule vehicule = maVoiture`)

## 5.6.2. Interfaces multiples

Un des avantages majeurs des interfaces est qu'une classe peut implémenter plusieurs interfaces, contrairement à l'héritage simple en C#.

```csharp
// Interfaces
public interface IVehicule
{
    void Demarrer();
    void Arreter();
}

public interface IElectrique
{
    int NiveauBatterie { get; }
    void Recharger();
}

public interface IAmphibie
{
    void Naviguer();
}

// Implémentation de plusieurs interfaces
public class VoitureElectriqueAmphibie : IVehicule, IElectrique, IAmphibie
{
    private int niveauBatterie;

    // Constructeur
    public VoitureElectriqueAmphibie()
    {
        niveauBatterie = 100;
    }

    // Implémentation de IVehicule
    public void Demarrer()
    {
        if (niveauBatterie > 0)
        {
            Console.WriteLine("La voiture électrique amphibie démarre.");
            niveauBatterie -= 5;
        }
        else
        {
            Console.WriteLine("Batterie vide, impossible de démarrer.");
        }
    }

    public void Arreter()
    {
        Console.WriteLine("La voiture électrique amphibie s'arrête.");
    }

    // Implémentation de IElectrique
    public int NiveauBatterie => niveauBatterie;

    public void Recharger()
    {
        Console.WriteLine("Recharge en cours...");
        niveauBatterie = 100;
        Console.WriteLine("Batterie rechargée à 100%.");
    }

    // Implémentation de IAmphibie
    public void Naviguer()
    {
        if (niveauBatterie >= 10)
        {
            Console.WriteLine("La voiture navigue sur l'eau.");
            niveauBatterie -= 10;
        }
        else
        {
            Console.WriteLine("Niveau de batterie insuffisant pour naviguer.");
        }
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        VoitureElectriqueAmphibie voiture = new VoitureElectriqueAmphibie();

        // Utilisation directe
        voiture.Demarrer();
        voiture.Naviguer();
        Console.WriteLine($"Niveau de batterie: {voiture.NiveauBatterie}%");
        voiture.Recharger();

        // Utilisation à travers différentes interfaces
        IVehicule vehicule = voiture;
        vehicule.Demarrer();

        IElectrique electrique = voiture;
        Console.WriteLine($"Niveau de batterie via interface: {electrique.NiveauBatterie}%");
        electrique.Recharger();

        IAmphibie amphibie = voiture;
        amphibie.Naviguer();
    }
}
```


### Points clés :
- Une classe peut implémenter un nombre illimité d'interfaces
- Chaque interface représente un aspect ou une capacité spécifique
- Les interfaces permettent de créer des objets qui peuvent être utilisés dans différents contextes

## 5.6.3. Implémentation explicite

Parfois, deux interfaces peuvent avoir des membres avec la même signature, ou vous pourriez vouloir cacher certains membres d'interface pour qu'ils ne soient accessibles que via l'interface. L'implémentation explicite résout ces problèmes.

```csharp
// Deux interfaces avec une méthode de même nom
public interface IAnimal
{
    void Manger();
}

public interface IMachine
{
    void Manger(); // Même nom que dans IAnimal
}

// Implémentation normale - une seule méthode pour les deux interfaces
public class Robot : IAnimal, IMachine
{
    // Cette implémentation est utilisée pour les deux interfaces
    public void Manger()
    {
        Console.WriteLine("Le robot mange de l'électricité.");
    }
}

// Implémentation explicite
public class RobotAvance : IAnimal, IMachine
{
    // Implémentation explicite pour IAnimal
    void IAnimal.Manger()
    {
        Console.WriteLine("Le robot imite la consommation de nourriture.");
    }

    // Implémentation explicite pour IMachine
    void IMachine.Manger()
    {
        Console.WriteLine("Le robot consomme de l'énergie.");
    }

    // Méthode publique standard qui n'appartient à aucune interface
    public void Manger()
    {
        Console.WriteLine("Le robot se recharge.");
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        // Robot standard
        Robot robot = new Robot();
        robot.Manger();  // "Le robot mange de l'électricité."

        // Le même comportement est accessible via les interfaces
        IAnimal animalRobot = robot;
        animalRobot.Manger();  // "Le robot mange de l'électricité."

        IMachine machineRobot = robot;
        machineRobot.Manger();  // "Le robot mange de l'électricité."

        // Robot avancé avec implémentation explicite
        RobotAvance robotAvance = new RobotAvance();
        robotAvance.Manger();  // "Le robot se recharge." (méthode de classe)

        // Accès aux implémentations spécifiques via les interfaces
        IAnimal animalRobotAvance = robotAvance;
        animalRobotAvance.Manger();  // "Le robot imite la consommation de nourriture."

        IMachine machineRobotAvance = robotAvance;
        machineRobotAvance.Manger();  // "Le robot consomme de l'énergie."
    }
}
```


### Points clés :
- L'implémentation explicite s'écrit avec la syntaxe `void InterfaceName.MethodName()`
- Les membres implémentés explicitement ne sont accessibles que via une référence d'interface
- Ils ne sont pas accessibles directement depuis une instance de la classe
- Utile pour résoudre les conflits de noms ou cacher certaines fonctionnalités

## 5.6.4. Interfaces par défaut (C# 8+)

À partir de C# 8.0 (disponible sur .NET Core 3.0 et .NET 5/6/7/8), les interfaces peuvent fournir des implémentations par défaut pour leurs membres. Cette fonctionnalité facilite l'évolution des interfaces sans casser la compatibilité avec le code existant.

```csharp
// Interface avec implémentation par défaut (C# 8+)
public interface ILogger
{
    // Méthode traditionnelle sans implémentation
    void LogError(string message);

    // Méthode avec implémentation par défaut
    void LogWarning(string message)
    {
        Console.WriteLine($"AVERTISSEMENT: {message}");
    }

    // Propriété avec implémentation par défaut
    bool DebugMode { get; set; } = false;

    // Méthode statique (C# 8+)
    static string FormatMessage(string message, string level)
    {
        return $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] {level}: {message}";
    }
}

// Implémentation minimale
public class ConsoleLogger : ILogger
{
    // Seule LogError doit être implémentée explicitement
    public void LogError(string message)
    {
        Console.WriteLine($"ERREUR: {message}");
    }

    // On peut aussi remplacer l'implémentation par défaut
    public void LogWarning(string message)
    {
        Console.WriteLine($"ATTENTION: {message} - {DateTime.Now}");
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        ConsoleLogger logger = new ConsoleLogger();
        logger.LogError("Une erreur est survenue");
        logger.LogWarning("Attention à cette opération");

        // Accès via l'interface
        ILogger iLogger = logger;
        iLogger.LogError("Une autre erreur");
        iLogger.LogWarning("Un autre avertissement");

        // Utilisation de la propriété par défaut
        iLogger.DebugMode = true;
        Console.WriteLine($"Mode debug: {iLogger.DebugMode}");

        // Utilisation de la méthode statique
        string message = ILogger.FormatMessage("Message formaté", "INFO");
        Console.WriteLine(message);
    }
}
```


### Points clés :
- Nécessite .NET Core 3.0+ ou .NET 5+ et C# 8.0+
- Permet d'ajouter des méthodes à une interface sans casser le code existant
- Les implémentations par défaut sont utilisées uniquement si la classe n'implémente pas la méthode
- Les interfaces peuvent maintenant avoir des méthodes statiques, des propriétés avec implémentations par défaut, etc.
- Utile pour faire évoluer les API publiques sans casser la compatibilité

## 5.6.5. Interfaces génériques

Les interfaces peuvent être génériques, ce qui les rend plus flexibles et réutilisables avec différents types.

```csharp
// Interface générique
public interface IRepository<T> where T : class
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}

// Classes modèles
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
}

public class Client
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Email { get; set; }
}

// Implémentation pour Produit
public class ProduitRepository : IRepository<Produit>
{
    private readonly List<Produit> produits = new List<Produit>();
    private int nextId = 1;

    public Produit GetById(int id)
    {
        return produits.FirstOrDefault(p => p.Id == id);
    }

    public IEnumerable<Produit> GetAll()
    {
        return produits;
    }

    public void Add(Produit entity)
    {
        entity.Id = nextId++;
        produits.Add(entity);
    }

    public void Update(Produit entity)
    {
        var existingProduit = GetById(entity.Id);
        if (existingProduit != null)
        {
            existingProduit.Nom = entity.Nom;
            existingProduit.Prix = entity.Prix;
        }
    }

    public void Delete(int id)
    {
        var produit = GetById(id);
        if (produit != null)
        {
            produits.Remove(produit);
        }
    }
}

// Implémentation pour Client
public class ClientRepository : IRepository<Client>
{
    private readonly List<Client> clients = new List<Client>();
    private int nextId = 1;

    public Client GetById(int id)
    {
        return clients.FirstOrDefault(c => c.Id == id);
    }

    public IEnumerable<Client> GetAll()
    {
        return clients;
    }

    public void Add(Client entity)
    {
        entity.Id = nextId++;
        clients.Add(entity);
    }

    public void Update(Client entity)
    {
        var existingClient = GetById(entity.Id);
        if (existingClient != null)
        {
            existingClient.Nom = entity.Nom;
            existingClient.Email = entity.Email;
        }
    }

    public void Delete(int id)
    {
        var client = GetById(id);
        if (client != null)
        {
            clients.Remove(client);
        }
    }
}

// Service générique utilisant l'interface générique
public class GenericService<T> where T : class
{
    private readonly IRepository<T> repository;

    public GenericService(IRepository<T> repository)
    {
        this.repository = repository;
    }

    public void ProcessAll()
    {
        var items = repository.GetAll();
        foreach (var item in items)
        {
            Console.WriteLine($"Traitement de: {item}");
        }
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        // Création et utilisation du repository de produits
        var produitRepo = new ProduitRepository();
        produitRepo.Add(new Produit { Nom = "Ordinateur", Prix = 1200 });
        produitRepo.Add(new Produit { Nom = "Téléphone", Prix = 800 });

        foreach (var produit in produitRepo.GetAll())
        {
            Console.WriteLine($"Produit: {produit.Nom}, Prix: {produit.Prix}€");
        }

        // Création et utilisation du repository de clients
        var clientRepo = new ClientRepository();
        clientRepo.Add(new Client { Nom = "Jean Dupont", Email = "jean@example.com" });
        clientRepo.Add(new Client { Nom = "Marie Martin", Email = "marie@example.com" });

        foreach (var client in clientRepo.GetAll())
        {
            Console.WriteLine($"Client: {client.Nom}, Email: {client.Email}");
        }

        // Utilisation du service générique
        var produitService = new GenericService<Produit>(produitRepo);
        var clientService = new GenericService<Client>(clientRepo);

        produitService.ProcessAll();
        clientService.ProcessAll();
    }
}
```


### Points clés :
- Les interfaces génériques utilisent les mêmes conventions de type générique que les classes
- On peut appliquer des contraintes aux types génériques (`where T : class`, etc.)
- Elles permettent de créer des contrats réutilisables pour différents types
- Idéales pour les patterns comme Repository, Factory, Service, etc.
- Facilitent l'écriture de code sans duplication et type-safe

Les interfaces sont l'un des outils les plus puissants de C# pour créer du code découplé, maintenable et testable. Elles constituent un élément essentiel de l'architecture orientée objet et sont largement utilisées dans les patterns de conception comme la Dépendance Injection.

⏭️ 5.7. [Classes abstraites](/05-programmation-orientee-objet-en-csharp/5-7-classes-abstraites.md)
