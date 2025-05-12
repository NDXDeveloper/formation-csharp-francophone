# 19.1. Patterns créationnels en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les **patterns créationnels** sont des modèles de conception qui fournissent des mécanismes de création d'objets qui augmentent la flexibilité et la réutilisabilité du code existant. Ces patterns vous aident à créer des objets de manière plus efficace et plus flexible.

## Qu'est-ce qu'un Design Pattern ?

Avant de plonger dans les patterns créationnels, comprenons ce qu'est un design pattern :

- Un **pattern de conception** est une solution réutilisable à un problème qui se produit couramment dans le développement logiciel
- Ce sont des "recettes" éprouvées qui peuvent vous aider à structurer votre code de manière plus maintenable
- Ils ne sont pas du code spécifique, mais plutôt des concepts que vous pouvez adapter à vos besoins

## 19.1.1. Singleton - "Un seul pour tous"

### Qu'est-ce que le Singleton ?

Le pattern Singleton garantit qu'**une classe n'a qu'une seule instance** et fournit un point d'accès global à cette instance.

### Quand l'utiliser ?

- Pour gérer un fichier de configuration
- Pour une connexion à une base de données
- Pour un système de logging
- Pour un cache partagé

### Exemple simple :

```csharp
public class Singleton
{
    // Instance statique privée
    private static Singleton instance = null;

    // Lock object pour thread safety
    private static readonly object lockObject = new object();

    // Constructeur privé empêche la création externe
    private Singleton()
    {
        // Initialisation ici
    }

    public static Singleton Instance
    {
        get
        {
            // Double-checked locking pour optimiser les performances
            if (instance == null)
            {
                lock (lockObject)
                {
                    if (instance == null)
                    {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }

    public void DoSomething()
    {
        Console.WriteLine("Singleton fait quelque chose !");
    }
}

// Utilisation
var singleton1 = Singleton.Instance;
var singleton2 = Singleton.Instance;
// singleton1 et singleton2 sont la même instance
```

### Version moderne avec Lazy<T> :

```csharp
public class ModernSingleton
{
    private static readonly Lazy<ModernSingleton> instance =
        new Lazy<ModernSingleton>(() => new ModernSingleton());

    private ModernSingleton()
    {
        // Initialisation
    }

    public static ModernSingleton Instance => instance.Value;

    public void DoSomething()
    {
        Console.WriteLine("Modern Singleton fait quelque chose !");
    }
}
```

## 19.1.2. Factory Method - "Une méthode pour les créer tous"

### Qu'est-ce que le Factory Method ?

Le Factory Method est un pattern qui définit une **interface pour créer des objets**, mais laisse les sous-classes décider de la classe à instancier.

### Quand l'utiliser ?

- Quand vous devez créer différents types d'objets, mais vous ne savez pas à l'avance quel type spécifique sera nécessaire
- Pour centraliser la logique de création
- Pour faciliter l'ajout de nouveaux types d'objets

### Exemple simple :

```csharp
// Interface ou classe abstraite pour le produit
public abstract class Animal
{
    public abstract void MakeSound();
}

// Implémentations concrètes
public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}

public class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Meow!");
    }
}

// Classe Factory abstraite
public abstract class AnimalFactory
{
    public abstract Animal CreateAnimal();
}

// Factories concrètes
public class DogFactory : AnimalFactory
{
    public override Animal CreateAnimal()
    {
        return new Dog();
    }
}

public class CatFactory : AnimalFactory
{
    public override Animal CreateAnimal()
    {
        return new Cat();
    }
}

// Utilisation
void Main()
{
    // On peut changer facilement le factory utilisé
    AnimalFactory factory = new DogFactory();
    Animal animal = factory.CreateAnimal();
    animal.MakeSound(); // Output: Woof!

    factory = new CatFactory();
    animal = factory.CreateAnimal();
    animal.MakeSound(); // Output: Meow!
}
```

### Version simplifiée avec Factory Method statique :

```csharp
public static class AnimalFactorySimple
{
    public static Animal CreateAnimal(string type)
    {
        switch (type.ToLower())
        {
            case "dog":
                return new Dog();
            case "cat":
                return new Cat();
            default:
                throw new ArgumentException("Type d'animal non reconnu");
        }
    }
}

// Utilisation
var dog = AnimalFactorySimple.CreateAnimal("dog");
dog.MakeSound(); // Output: Woof!
```

## 19.1.3. Abstract Factory - "Des familles de produits"

### Qu'est-ce que l'Abstract Factory ?

L'Abstract Factory fournit une interface pour créer des **familles d'objets liés** sans spécifier leurs classes concrètes.

### Quand l'utiliser ?

- Quand vous devez créer des familles d'objets liés
- Pour assurer la cohérence entre objets d'une même famille
- Pour changer facilement toute une famille de produits

### Exemple simple :

```csharp
// Interfaces pour les produits
public interface IButton
{
    void Render();
}

public interface ICheckbox
{
    void Render();
}

// Produits Windows
public class WindowsButton : IButton
{
    public void Render()
    {
        Console.WriteLine("Affichage d'un bouton Windows");
    }
}

public class WindowsCheckbox : ICheckbox
{
    public void Render()
    {
        Console.WriteLine("Affichage d'une checkbox Windows");
    }
}

// Produits Mac
public class MacButton : IButton
{
    public void Render()
    {
        Console.WriteLine("Affichage d'un bouton Mac");
    }
}

public class MacCheckbox : ICheckbox
{
    public void Render()
    {
        Console.WriteLine("Affichage d'une checkbox Mac");
    }
}

// Interface Abstract Factory
public interface IGUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Factories concrètes
public class WindowsFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new WindowsButton();
    }

    public ICheckbox CreateCheckbox()
    {
        return new WindowsCheckbox();
    }
}

public class MacFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new MacButton();
    }

    public ICheckbox CreateCheckbox()
    {
        return new MacCheckbox();
    }
}

// Utilisation
void CreateGUI(IGUIFactory factory)
{
    var button = factory.CreateButton();
    var checkbox = factory.CreateCheckbox();

    button.Render();
    checkbox.Render();
}

// Main
void Main()
{
    IGUIFactory factory;

    // Choisir selon l'OS
    if (Environment.OSVersion.Platform == PlatformID.Win32NT)
    {
        factory = new WindowsFactory();
    }
    else
    {
        factory = new MacFactory();
    }

    CreateGUI(factory);
}
```

## 19.1.4. Builder - "Construire étape par étape"

### Qu'est-ce que le Builder ?

Le Builder permet de construire des objets complexes **étape par étape**, séparant la construction de la représentation.

### Quand l'utiliser ?

- Pour créer des objets complexes avec de nombreux paramètres
- Quand vous voulez une construction flexible
- Pour améliorer la lisibilité du code

### Exemple simple :

```csharp
// Produit complexe
public class Car
{
    public string Make { get; set; }
    public string Model { get; set; }
    public int Year { get; set; }
    public string Color { get; set; }
    public bool HasAirConditioning { get; set; }
    public bool HasGPS { get; set; }

    public override string ToString()
    {
        return $"{Year} {Make} {Model}, Color: {Color}, AC: {HasAirConditioning}, GPS: {HasGPS}";
    }
}

// Builder
public class CarBuilder
{
    private Car car = new Car();

    public CarBuilder WithMake(string make)
    {
        car.Make = make;
        return this;
    }

    public CarBuilder WithModel(string model)
    {
        car.Model = model;
        return this;
    }

    public CarBuilder WithYear(int year)
    {
        car.Year = year;
        return this;
    }

    public CarBuilder WithColor(string color)
    {
        car.Color = color;
        return this;
    }

    public CarBuilder WithAirConditioning(bool hasAC = true)
    {
        car.HasAirConditioning = hasAC;
        return this;
    }

    public CarBuilder WithGPS(bool hasGPS = true)
    {
        car.HasGPS = hasGPS;
        return this;
    }

    public Car Build()
    {
        // Ici on pourrait valider que tous les champs requis sont remplis
        return car;
    }
}

// Utilisation - interface fluide
var car = new CarBuilder()
    .WithMake("Toyota")
    .WithModel("Camry")
    .WithYear(2023)
    .WithColor("Blue")
    .WithAirConditioning()
    .WithGPS()
    .Build();

Console.WriteLine(car);
// Output: 2023 Toyota Camry, Color: Blue, AC: True, GPS: True
```

### Version avec Director :

```csharp
// Director pour orchestrer la construction
public class CarDirector
{
    private CarBuilder builder;

    public CarDirector(CarBuilder builder)
    {
        this.builder = builder;
    }

    public Car BuildBasicCar()
    {
        return builder
            .WithMake("Toyota")
            .WithModel("Corolla")
            .WithYear(2023)
            .WithColor("White")
            .Build();
    }

    public Car BuildLuxuryCar()
    {
        return builder
            .WithMake("Mercedes")
            .WithModel("S-Class")
            .WithYear(2023)
            .WithColor("Black")
            .WithAirConditioning()
            .WithGPS()
            .Build();
    }
}

// Utilisation avec Director
var director = new CarDirector(new CarBuilder());
var basicCar = director.BuildBasicCar();
var luxuryCar = director.BuildLuxuryCar();
```

## 19.1.5. Prototype - "Cloner plutôt que créer"

### Qu'est-ce que le Prototype ?

Le Prototype permet de créer de nouveaux objets en **clonant des instances existantes** plutôt qu'en construisant des objets à partir de zéro.

### Quand l'utiliser ?

- Quand la création d'objets est coûteuse
- Pour éviter de répéter la configuration complexe
- Quand vous voulez éviter les sous-classes

### Exemple simple :

```csharp
// Interface pour le clonage
public interface ICloneable<T>
{
    T Clone();
}

// Classe avec clonage
public class Document : ICloneable<Document>
{
    public string Title { get; set; }
    public string Content { get; set; }
    public List<string> Tags { get; set; }

    public Document()
    {
        Tags = new List<string>();
    }

    // Clonage profond
    public Document Clone()
    {
        var clone = new Document();
        clone.Title = this.Title;
        clone.Content = this.Content;
        clone.Tags = new List<string>(this.Tags); // Nouvelle liste avec les mêmes éléments
        return clone;
    }

    public override string ToString()
    {
        return $"Titre: {Title}, Contenu: {Content}, Tags: {string.Join(", ", Tags)}";
    }
}

// Utilisation
var original = new Document
{
    Title = "Document Original",
    Content = "Contenu du document",
    Tags = { "important", "urgent" }
};

// Cloner le document
var copy = original.Clone();
copy.Title = "Document Copie";
copy.Tags.Add("copie");

// Les objets sont indépendants
Console.WriteLine(original);
Console.WriteLine(copy);

// Output:
// Titre: Document Original, Contenu: Contenu du document, Tags: important, urgent
// Titre: Document Copie, Contenu: Contenu du document, Tags: important, urgent, copie
```

### Prototype avec objects complexes :

```csharp
// Exemple avec une structure plus complexe
public class Employee : ICloneable<Employee>
{
    public string Name { get; set; }
    public Address Address { get; set; }
    public List<Skill> Skills { get; set; }

    public Employee()
    {
        Skills = new List<Skill>();
    }

    public Employee Clone()
    {
        var clone = new Employee
        {
            Name = this.Name,
            Address = this.Address?.Clone(), // Cloner aussi les objets complexes
            Skills = this.Skills.Select(s => s.Clone()).ToList()
        };
        return clone;
    }
}

public class Address : ICloneable<Address>
{
    public string Street { get; set; }
    public string City { get; set; }

    public Address Clone()
    {
        return new Address
        {
            Street = this.Street,
            City = this.City
        };
    }
}

public class Skill : ICloneable<Skill>
{
    public string Name { get; set; }
    public int Level { get; set; }

    public Skill Clone()
    {
        return new Skill
        {
            Name = this.Name,
            Level = this.Level
        };
    }
}
```

## 19.1.6. Object Pool - "Recycler plutôt que créer"

### Qu'est-ce que l'Object Pool ?

L'Object Pool maintient un **pool d'objets réutilisables** pour éviter la création/destruction fréquente d'objets coûteux.

### Quand l'utiliser ?

- Quand la création d'objets est très coûteuse
- Pour améliorer les performances
- Quand vous avez des objets avec un cycle de vie court mais fréquemment utilisés

### Exemple simple :

```csharp
// Objet que nous voulons pooler
public class Connection
{
    public string ConnectionString { get; set; }
    public bool IsInUse { get; set; }

    public Connection(string connectionString)
    {
        ConnectionString = connectionString;
        IsInUse = false;
    }

    public void Connect()
    {
        Console.WriteLine($"Connexion établie avec {ConnectionString}");
    }

    public void Disconnect()
    {
        Console.WriteLine($"Connexion fermée avec {ConnectionString}");
    }
}

// Object Pool
public class ConnectionPool
{
    private readonly List<Connection> connections;
    private readonly int maxSize;
    private readonly string connectionString;

    public ConnectionPool(string connectionString, int maxSize = 10)
    {
        this.connectionString = connectionString;
        this.maxSize = maxSize;
        this.connections = new List<Connection>();

        // Créer quelques connexions initiales
        for (int i = 0; i < 3; i++)
        {
            connections.Add(new Connection(connectionString));
        }
    }

    public Connection GetConnection()
    {
        // Chercher une connexion libre
        var freeConnection = connections.FirstOrDefault(c => !c.IsInUse);

        if (freeConnection != null)
        {
            freeConnection.IsInUse = true;
            return freeConnection;
        }

        // Si aucune connexion libre et on peut encore en créer
        if (connections.Count < maxSize)
        {
            var newConnection = new Connection(connectionString);
            newConnection.IsInUse = true;
            connections.Add(newConnection);
            return newConnection;
        }

        // Pas de connexion disponible
        throw new InvalidOperationException("Aucune connexion disponible dans le pool");
    }

    public void ReleaseConnection(Connection connection)
    {
        if (connections.Contains(connection))
        {
            connection.IsInUse = false;
        }
    }

    public int AvailableConnections => connections.Count(c => !c.IsInUse);
    public int TotalConnections => connections.Count;
}

// Utilisation
var pool = new ConnectionPool("Server=localhost;Database=TestDB", 5);

// Obtenir une connexion du pool
var conn1 = pool.GetConnection();
conn1.Connect();

// Utiliser la connexion...

// Libérer la connexion
conn1.Disconnect();
pool.ReleaseConnection(conn1);

Console.WriteLine($"Connexions disponibles: {pool.AvailableConnections}/{pool.TotalConnections}");
```

### Version avec IDisposable :

```csharp
// Version améliorée avec support using
public class PooledConnection : IDisposable
{
    private Connection connection;
    private ConnectionPool pool;

    public PooledConnection(Connection connection, ConnectionPool pool)
    {
        this.connection = connection;
        this.pool = pool;
    }

    // Déléguer les appels à l'objet connection
    public void Connect() => connection.Connect();
    public void Disconnect() => connection.Disconnect();

    public void Dispose()
    {
        connection.Disconnect();
        pool.ReleaseConnection(connection);
    }
}

// Utilisation avec using
using (var pooledConn = pool.GetPooledConnection())
{
    pooledConn.Connect();
    // Faire le travail...
    // La connexion sera automatiquement libérée
}
```

## Résumé des Patterns Créationnels

| Pattern | But principal | Cas d'utilisation typique |
|---------|--------------|--------------------------|
| **Singleton** | Une seule instance | Configuration, Cache, Log |
| **Factory Method** | Création par famille | Différents types d'objets |
| **Abstract Factory** | Familles d'objets liés | Thèmes, Platforms |
| **Builder** | Construction complexe | Objets avec nombreux paramètres |
| **Prototype** | Clonage d'objets | Objets coûteux à créer |
| **Object Pool** | Réutilisation d'objets | Ressources limitées |

## Bonnes pratiques

1. **Ne pas overutiloser les patterns** : Utilisez-les seulement quand ils apportent une véritable valeur
2. **Comprendre avant d'implémenter** : Assurez-vous de bien comprendre le problème que le pattern résout
3. **Adapter aux besoins** : Les exemples sont des guides, pas des copies à coller
4. **Documentation** : Documentez pourquoi vous avez choisi un pattern particulier
5. **Tests** : Testez vos implémentations de patterns

## Exercices pratiques

### Exercice 1 : Singleton pour Configuration
Créez un Singleton pour gérer la configuration d'une application. Il doit pouvoir charger des valeurs depuis un fichier et les mettre en cache.

### Exercice 2 : Factory pour Notifications
Créez un Factory Method pour créer différents types de notifications (Email, SMS, Push).

### Exercice 3 : Builder pour Requête SQL
Créez un Builder pour construire des requêtes SQL SELECT avec différentes clauses (WHERE, ORDER BY, LIMIT).

### Exercice 4 : Object Pool pour Threads
Implémentez un pool de threads pour exécuter des tâches en parallèle.

## Conclusion

Les patterns créationnels sont des outils puissants pour gérer la création d'objets de manière flexible et maintenable. Ils vous aident à :

- Écrire du code plus modulaire
- Faciliter les modifications et extensions
- Améliorer les performances
- Respecter les principes SOLID

N'oubliez pas que ces patterns sont des guides, pas des règles strictes. Adaptez-les à vos besoins spécifiques et n'hésitez pas à les combiner quand c'est approprié.

⏭️ 19.2. [Patterns structurels](/19-design-patterns-en-csharp/19-2-patterns-structurels.md)
