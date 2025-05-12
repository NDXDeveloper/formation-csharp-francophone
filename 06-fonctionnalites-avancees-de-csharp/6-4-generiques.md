# 6.4. Génériques

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les génériques sont une fonctionnalité fondamentale de C# qui permet d'écrire du code typé de manière flexible et réutilisable. Ils permettent de créer des classes, des méthodes et des interfaces qui opèrent sur des types arbitraires, tout en maintenant la sécurité de type lors de la compilation.

## 6.4.1. Classes et méthodes génériques

Les classes et méthodes génériques vous permettent de définir des fonctionnalités qui fonctionnent avec différents types de données sans sacrifier la sécurité de type.

### Classes génériques

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Classes génériques ===\n");

        // Utilisation de la classe générique avec différents types
        Box<int> intBox = new Box<int>(42);
        Console.WriteLine($"Boîte d'entier: {intBox.Content}");

        Box<string> stringBox = new Box<string>("Hello, Generics!");
        Console.WriteLine($"Boîte de chaîne: {stringBox.Content}");

        Box<DateTime> dateBox = new Box<DateTime>(DateTime.Now);
        Console.WriteLine($"Boîte de date: {dateBox.Content}");

        // Utilisation d'une classe générique avec plusieurs paramètres de type
        Pair<int, string> pair1 = new Pair<int, string>(1, "Premier");
        Console.WriteLine($"Paire: ({pair1.First}, {pair1.Second})");

        Pair<string, bool> pair2 = new Pair<string, bool>("Vérifié", true);
        Console.WriteLine($"Paire: ({pair2.First}, {pair2.Second})");

        // Utilisation d'une classe générique plus complexe
        Repository<Product> productRepository = new Repository<Product>();
        productRepository.Add(new Product { Id = 1, Name = "Laptop", Price = 999.99m });
        productRepository.Add(new Product { Id = 2, Name = "Smartphone", Price = 699.99m });
        productRepository.Add(new Product { Id = 3, Name = "Headphones", Price = 149.99m });

        Console.WriteLine("\nProduits dans le dépôt:");
        foreach (var product in productRepository.GetAll())
        {
            Console.WriteLine($" - {product.Id}: {product.Name}, ${product.Price}");
        }

        Product foundProduct = productRepository.GetById(2);
        Console.WriteLine($"\nProduit trouvé: {foundProduct?.Name ?? "Non trouvé"}");

        // Utilisation d'une classe générique encapsulant d'autres génériques
        Cache<int, string> cache = new Cache<int, string>();
        cache.Add(1, "Valeur un");
        cache.Add(2, "Valeur deux");

        Console.WriteLine($"\nValeur 1 dans le cache: {cache.Get(1)}");
        Console.WriteLine($"La valeur 3 existe-t-elle? {cache.Contains(3)}");
    }
}

// Classe générique simple avec un paramètre de type
class Box<T>
{
    public T Content { get; set; }

    public Box(T content)
    {
        Content = content;
    }
}

// Classe générique avec deux paramètres de type
class Pair<TFirst, TSecond>
{
    public TFirst First { get; set; }
    public TSecond Second { get; set; }

    public Pair(TFirst first, TSecond second)
    {
        First = first;
        Second = second;
    }
}

// Classe avec plusieurs propriétés servant d'exemple pour le repository
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Classe générique plus complexe représentant un dépôt
class Repository<T> where T : class
{
    private List<T> _items = new List<T>();

    public void Add(T item)
    {
        _items.Add(item);
    }

    public IEnumerable<T> GetAll()
    {
        return _items;
    }

    public T GetById(int id)
    {
        // Assume T has an Id property (handled through reflection for flexibility)
        foreach (var item in _items)
        {
            var idProperty = item.GetType().GetProperty("Id");
            if (idProperty != null && idProperty.GetValue(item) is int itemId && itemId == id)
            {
                return item;
            }
        }
        return null;
    }
}

// Classe générique encapsulant d'autres classes génériques
class Cache<TKey, TValue>
{
    private Dictionary<TKey, TValue> _cache = new Dictionary<TKey, TValue>();

    public void Add(TKey key, TValue value)
    {
        _cache[key] = value;
    }

    public TValue Get(TKey key)
    {
        if (_cache.TryGetValue(key, out TValue value))
        {
            return value;
        }
        throw new KeyNotFoundException($"La clé {key} n'existe pas dans le cache.");
    }

    public bool Contains(TKey key)
    {
        return _cache.ContainsKey(key);
    }
}
```


### Méthodes génériques

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Méthodes génériques ===\n");

        // Utilisation de méthodes génériques
        int maxInt = Max(10, 20);
        Console.WriteLine($"Max de 10 et 20: {maxInt}");

        double maxDouble = Max(3.14, 2.71);
        Console.WriteLine($"Max de 3.14 et 2.71: {maxDouble}");

        string maxString = Max("abc", "xyz");
        Console.WriteLine($"Max de \"abc\" et \"xyz\": {maxString}");

        // Échange de valeurs avec Swap
        int a = 5, b = 10;
        Console.WriteLine($"Avant échange: a = {a}, b = {b}");
        Swap(ref a, ref b);
        Console.WriteLine($"Après échange: a = {a}, b = {b}");

        string s1 = "Hello", s2 = "World";
        Console.WriteLine($"Avant échange: s1 = {s1}, s2 = {s2}");
        Swap(ref s1, ref s2);
        Console.WriteLine($"Après échange: s1 = {s1}, s2 = {s2}");

        // Utilisation d'une méthode générique avec un type de retour inféré
        var numbers = new List<int> { 1, 2, 3, 4, 5 };
        var doubled = TransformList(numbers, x => x * 2);
        Console.WriteLine("\nDoublement de liste d'entiers:");
        foreach (var num in doubled)
        {
            Console.Write($"{num} ");
        }
        Console.WriteLine();

        var words = new List<string> { "apple", "banana", "cherry" };
        var upperWords = TransformList(words, s => s.ToUpper());
        Console.WriteLine("\nMots en majuscules:");
        foreach (var word in upperWords)
        {
            Console.Write($"{word} ");
        }
        Console.WriteLine();

        // Utilisation d'une méthode générique avec plusieurs paramètres de type
        var dict = CreateMap(new[] { "one", "two", "three" }, new[] { 1, 2, 3 });
        Console.WriteLine("\nDictionnaire créé:");
        foreach (var kvp in dict)
        {
            Console.WriteLine($" - {kvp.Key}: {kvp.Value}");
        }

        // Utilisation d'une méthode avec type de retour générique
        string[] names = { "Alice", "Bob", "Charlie", "David" };

        Console.WriteLine("\nPremier élément satisfaisant le prédicat:");
        var result = Find(names, n => n.StartsWith("C"));
        Console.WriteLine($"Nom commençant par 'C': {result}");

        // Méthode générique avec type dynamique
        PrintTypes(1, "hello", true, 3.14);
    }

    // Méthode générique simple
    static T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) > 0 ? a : b;
    }

    // Méthode générique avec paramètres ref
    static void Swap<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }

    // Méthode générique avec paramètre de délégué/lambda
    static List<TOutput> TransformList<TInput, TOutput>(List<TInput> items, Func<TInput, TOutput> transformer)
    {
        List<TOutput> result = new List<TOutput>();
        foreach (var item in items)
        {
            result.Add(transformer(item));
        }
        return result;
    }

    // Méthode générique avec plusieurs paramètres de type
    static Dictionary<TKey, TValue> CreateMap<TKey, TValue>(TKey[] keys, TValue[] values)
    {
        if (keys.Length != values.Length)
        {
            throw new ArgumentException("Les tableaux de clés et de valeurs doivent avoir la même longueur.");
        }

        Dictionary<TKey, TValue> result = new Dictionary<TKey, TValue>();
        for (int i = 0; i < keys.Length; i++)
        {
            result.Add(keys[i], values[i]);
        }
        return result;
    }

    // Méthode générique avec type de retour
    static T Find<T>(T[] items, Predicate<T> predicate)
    {
        foreach (var item in items)
        {
            if (predicate(item))
            {
                return item;
            }
        }
        return default(T);
    }

    // Méthode générique avec nombre variable de paramètres
    static void PrintTypes<T1, T2, T3, T4>(T1 p1, T2 p2, T3 p3, T4 p4)
    {
        Console.WriteLine("\nTypes des paramètres:");
        Console.WriteLine($" - Paramètre 1: {typeof(T1).Name} (valeur: {p1})");
        Console.WriteLine($" - Paramètre 2: {typeof(T2).Name} (valeur: {p2})");
        Console.WriteLine($" - Paramètre 3: {typeof(T3).Name} (valeur: {p3})");
        Console.WriteLine($" - Paramètre 4: {typeof(T4).Name} (valeur: {p4})");
    }
}
```


### Avantages des classes et méthodes génériques

1. **Sécurité de type** : Les erreurs de type sont détectées lors de la compilation plutôt qu'à l'exécution.

2. **Réutilisation de code** : Écrire du code qui fonctionne avec plusieurs types sans duplication.

3. **Performance** : Les génériques évitent les conversions de type (boxing/unboxing) pour les types valeur, améliorant ainsi les performances.

4. **Spécialisation** : Le code générique peut être adapté à des types spécifiques lorsque c'est nécessaire.

5. **Lisibilité et maintenance** : Le code générique est souvent plus clair et plus facile à maintenir.

## 6.4.2. Contraintes sur les types génériques

Les contraintes de type permettent de restreindre les types qui peuvent être utilisés comme arguments de type pour un type ou une méthode générique, offrant ainsi plus de contrôle et d'expressivité.

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Contraintes sur les génériques ===\n");

        // Contrainte de classe (reference type)
        DataProcessor<Person> personProcessor = new DataProcessor<Person>();
        personProcessor.Process(new Person { Name = "Alice", Age = 30 });

        // Cette ligne ne compilerait pas car int est un type valeur:
        // DataProcessor<int> intProcessor = new DataProcessor<int>();

        // Contrainte d'interface
        var shapes = new List<IShape>
        {
            new Circle { Radius = 5 },
            new Rectangle { Width = 10, Height = 20 }
        };

        Console.WriteLine("Surfaces des formes:");
        foreach (var shape in shapes)
        {
            Console.WriteLine($" - {shape.GetType().Name}: {shape.CalculateArea()}");
        }

        // Utilisation de la contrainte d'interface
        ShapeDrawer<Circle> circleDrawer = new ShapeDrawer<Circle>(new Circle { Radius = 7 });
        circleDrawer.DrawShape();

        // Contrainte de constructeur sans paramètre
        ObjectFactory<Person> personFactory = new ObjectFactory<Person>();
        Person newPerson = personFactory.CreateInstance();
        Console.WriteLine($"Nouvelle personne créée: {newPerson.Name}, {newPerson.Age} ans");

        // Contrainte de type de valeur (struct)
        NumberProcessor<int> intProcessor = new NumberProcessor<int>();
        Console.WriteLine($"Est-ce que 42 est pair? {intProcessor.IsEven(42)}");

        NumberProcessor<double> doubleProcessor = new NumberProcessor<double>();
        Console.WriteLine($"Est-ce que 3.14 est pair? {doubleProcessor.IsEven(3.14)}");

        // Contraintes multiples
        Repository<Customer> customerRepo = new Repository<Customer>();
        Customer customer = new Customer { Id = 1, Name = "Bob", Email = "bob@example.com" };
        customerRepo.Save(customer);
        Console.WriteLine($"Client enregistré: {customer.Name}");

        // Contraintes génériques
        ComparableProcessor<AgeComparable> ageProcessor = new ComparableProcessor<AgeComparable>();
        AgeComparable ac1 = new AgeComparable { Age = 25 };
        AgeComparable ac2 = new AgeComparable { Age = 30 };
        int comparison = ageProcessor.Compare(ac1, ac2);
        Console.WriteLine($"Comparaison d'âge: {comparison}");

        // Contrainte de type spécifique (type de base)
        VehicleHandler<Car> carHandler = new VehicleHandler<Car>();
        carHandler.DisplayInfo(new Car { Make = "Toyota", Model = "Corolla", Year = 2020 });

        // Contrainte where T : U (contrainte de type générique sur un autre type générique)
        Converter<SuperClass, BaseClass> converter = new Converter<SuperClass, BaseClass>();
        SuperClass superObj = new SuperClass { Name = "SuperObject", ExtendedProperty = "Extra" };
        BaseClass converted = converter.Convert(superObj);
        Console.WriteLine($"Objet converti: {converted.Name}");
    }
}

// Classes et interfaces pour les exemples

class Person
{
    public string Name { get; set; } = "Unknown";
    public int Age { get; set; }
}

interface IShape
{
    double CalculateArea();
}

class Circle : IShape
{
    public double Radius { get; set; }
    public double CalculateArea() => Math.PI * Radius * Radius;
}

class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public double CalculateArea() => Width * Height;
}

interface IEntity
{
    int Id { get; set; }
}

class Customer : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

class AgeComparable : IComparable<AgeComparable>
{
    public int Age { get; set; }

    public int CompareTo(AgeComparable other)
    {
        if (other == null) return 1;
        return Age.CompareTo(other.Age);
    }
}

class Vehicle
{
    public string Make { get; set; }
    public string Model { get; set; }
}

class Car : Vehicle
{
    public int Year { get; set; }
}

class BaseClass
{
    public string Name { get; set; }
}

class SuperClass : BaseClass
{
    public string ExtendedProperty { get; set; }
}

// Classes génériques avec contraintes

// Contrainte où T doit être une classe (type référence)
class DataProcessor<T> where T : class
{
    public void Process(T data)
    {
        Console.WriteLine($"Traitement de données de type {typeof(T).Name}");
    }
}

// Contrainte où T doit implémenter une interface
class ShapeDrawer<T> where T : IShape
{
    private T _shape;

    public ShapeDrawer(T shape)
    {
        _shape = shape;
    }

    public void DrawShape()
    {
        Console.WriteLine($"Dessin d'une forme de type {typeof(T).Name} avec une surface de {_shape.CalculateArea()}");
    }
}

// Contrainte où T doit avoir un constructeur sans paramètre
class ObjectFactory<T> where T : new()
{
    public T CreateInstance()
    {
        return new T();
    }
}

// Contrainte où T doit être un type valeur (struct)
class NumberProcessor<T> where T : struct
{
    public bool IsEven(T value)
    {
        // Conversion en dynamic pour traiter différents types numériques
        dynamic dynamicValue = value;
        return dynamicValue % 2 == 0;
    }
}

// Combinaison de plusieurs contraintes
class Repository<T> where T : class, IEntity, new()
{
    public void Save(T entity)
    {
        // Simuler une sauvegarde
        Console.WriteLine($"Sauvegarde de l'entité {typeof(T).Name} avec l'ID {entity.Id}");
    }

    public T CreateNew()
    {
        return new T();
    }
}

// Contrainte où T doit implémenter IComparable<T>
class ComparableProcessor<T> where T : IComparable<T>
{
    public int Compare(T x, T y)
    {
        return x.CompareTo(y);
    }
}

// Contrainte où T doit hériter d'un type spécifique
class VehicleHandler<T> where T : Vehicle
{
    public void DisplayInfo(T vehicle)
    {
        Console.WriteLine($"Véhicule: {vehicle.Make} {vehicle.Model}");
    }
}

// Contrainte où T doit être ou hériter de U
class Converter<T, U> where T : U
{
    public U Convert(T input)
    {
        return input; // La conversion est implicite car T : U
    }
}
```


### Types de contraintes

1. **Contrainte de référence** (`where T : class`): Le type doit être un type référence.

2. **Contrainte de valeur** (`where T : struct`): Le type doit être un type valeur.

3. **Contrainte de constructeur** (`where T : new()`): Le type doit avoir un constructeur sans paramètre.

4. **Contrainte d'interface** (`where T : IInterface`): Le type doit implémenter l'interface spécifiée.

5. **Contrainte de classe de base** (`where T : BaseClass`): Le type doit être ou hériter de la classe spécifiée.

6. **Contrainte de type générique** (`where T : U`): Le type T doit être ou hériter du type U.

7. **Contraintes multiples**: Combinaison de plusieurs contraintes sur un même paramètre de type.

### Avantages des contraintes

1. **Accès aux membres**: Accéder aux propriétés et méthodes définies par la contrainte.

2. **Restrictions logiques**: Limiter les types possibles à ceux qui sont pertinents.

3. **Sécurité et clarté**: Exprimer des intentions et prévenir l'utilisation incorrecte.

4. **Performance**: Optimisations du compilateur basées sur les contraintes.

## 6.4.3. Covariance et contravariance

La covariance et la contravariance sont des concepts qui étendent la flexibilité des génériques en permettant la conversion entre des types génériques différents mais compatibles.

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Covariance et contravariance ===\n");

        // COVARIANCE (du plus spécifique vers le plus général)
        Console.WriteLine("--- Covariance ---");

        // Covariance avec les types de retour de délégués (Func<out T>)
        Func<string> getStringMessage = () => "Message en chaîne de caractères";
        Func<object> getMessage = getStringMessage; // Covariance: string -> object

        object result = getMessage();
        Console.WriteLine($"Résultat: {result} (type: {result.GetType().Name})");

        // Covariance avec les arrays (intégrée au langage)
        Console.WriteLine("\nCovariance avec les tableaux:");

        Dog[] dogs = new Dog[]
        {
            new Dog { Name = "Rex" },
            new Dog { Name = "Buddy" }
        };

        Animal[] animals = dogs; // Covariance: Dog[] -> Animal[]

        foreach (var animal in animals)
        {
            Console.WriteLine($" - Animal: {animal.Name}");
        }

        // REMARQUE: La covariance des tableaux n'est pas sûre au type:
        try
        {
            // Ceci lèvera une exception à l'exécution
            // animals[0] = new Cat { Name = "Whiskers" }; // Erreur à l'exécution
        }
        catch (ArrayTypeMismatchException)
        {
            Console.WriteLine(" (Une tentative d'assignation d'un chat dans un tableau de chiens échouerait)");
        }

        // Covariance avec les interfaces génériques (IEnumerable<out T>)
        Console.WriteLine("\nCovariance avec IEnumerable<T>:");

        IEnumerable<Dog> puppies = new List<Dog>
        {
            new Dog { Name = "Spot" },
            new Dog { Name = "Max" }
        };

        IEnumerable<Animal> pets = puppies; // Covariance: IEnumerable<Dog> -> IEnumerable<Animal>

        foreach (var pet in pets)
        {
            Console.WriteLine($" - Pet: {pet.Name}");
        }

        // CONTRAVARIANCE (du plus général vers le plus spécifique)
        Console.WriteLine("\n--- Contravariance ---");

        // Contravariance avec les paramètres de délégués (Action<in T>)
        Action<Animal> feedAnimal = animal => Console.WriteLine($"Alimentation de {animal.Name}");
        Action<Dog> feedDog = feedAnimal; // Contravariance: Action<Animal> -> Action<Dog>

        Dog myDog = new Dog { Name = "Fido" };
        feedDog(myDog);

        // Contravariance avec les interfaces (IComparer<in T>)
        Console.WriteLine("\nContravariance avec IComparer<T>:");

        IComparer<Animal> animalComparer = new AnimalComparer();
        IComparer<Dog> dogComparer = animalComparer; // Contravariance: IComparer<Animal> -> IComparer<Dog>

        Dog dog1 = new Dog { Name = "Buster", Age = 3 };
        Dog dog2 = new Dog { Name = "Charlie", Age = 5 };

        int comparisonResult = dogComparer.Compare(dog1, dog2);
        Console.WriteLine($"Comparaison: {dog1.Name} est {(comparisonResult < 0 ? "plus jeune que" : comparisonResult > 0 ? "plus âgé que" : "du même âge que")} {dog2.Name}");

        // UTILISATION PRATIQUE DE LA COVARIANCE ET DE LA CONTRAVARIANCE
        Console.WriteLine("\n--- Exemple pratique de covariance et contravariance ---");

        // Convertir
        List<Cat> cats = new List<Cat> { new Cat { Name = "Whiskers" }, new Cat { Name = "Felix" } };
        List<Dog> dogs2 = new List<Dog> { new Dog { Name = "Rex" }, new Dog { Name = "Buddy" } };

        // Utilisation de la covariance
        PrintAnimals(cats); // Covariance: IEnumerable<Cat> -> IEnumerable<Animal>
        PrintAnimals(dogs2); // Covariance: IEnumerable<Dog> -> IEnumerable<Animal>

        // Utilisation de la contravariance
        AnimalProcessor<Animal> animalProcessor = new AnimalProcessor<Animal>();
        CatProcessor catProcessor = new CatProcessor(animalProcessor); // Contravariance en action

        catProcessor.Process(new Cat { Name = "Garfield" });
    }

    // Méthode tirant parti de la covariance
    static void PrintAnimals(IEnumerable<Animal> animals)
    {
        Console.WriteLine("Animaux:");
        foreach (var animal in animals)
        {
            Console.WriteLine($" - {animal.GetType().Name}: {animal.Name}");
        }
    }
}

// Classes pour les exemples
class Animal
{
    public string Name { get; set; }
    public int Age { get; set; } = 1;
}

class Dog : Animal { }

class Cat : Animal { }

// Implémentation d'un comparateur pour démontrer la contravariance
class AnimalComparer : IComparer<Animal>
{
    public int Compare(Animal x, Animal y)
    {
        return x.Age.CompareTo(y.Age);
    }
}

// Interface covariante (utilise 'out' pour le paramètre de type)
interface IProducer<out T>
{
    T Produce();
}

// Implémentation covariante
class DogProducer : IProducer<Dog>
{
    public Dog Produce() => new Dog { Name = "New dog" };
}

// Interface contravariante (utilise 'in' pour le paramètre de type)
interface IConsumer<in T>
{
    void Consume(T item);
}

// Implémentation contravariante
class AnimalConsumer : IConsumer<Animal>
{
    public void Consume(Animal item)
    {
        Console.WriteLine($"Consommation de {item.Name}");
    }
}

// Démonstration d'une situation réelle avec contravariance
class AnimalProcessor<T> where T : Animal
{
    public void Process(T animal)
    {
        Console.WriteLine($"Traitement de {animal.GetType().Name} nommé {animal.Name}");
    }
}

class CatProcessor
{
    private readonly AnimalProcessor<Animal> _processor;

    public CatProcessor(AnimalProcessor<Animal> processor)
    {
        _processor = processor;
    }

    public void Process(Cat cat)
    {
        // Utilisation de la contravariance: un AnimalProcessor<Animal> peut traiter un Cat
        _processor.Process(cat);
    }
}
```


### Covariance

La covariance permet d'utiliser un type plus dérivé (plus spécifique) que celui initialement spécifié. En C#, elle est indiquée par le mot-clé `out` dans les définitions d'interfaces génériques et de délégués.

Exemples de types covariants:
- `IEnumerable<out T>`
- `IReadOnlyList<out T>`
- `Func<out TResult>`

La covariance est utile pour les types qui produisent ou retournent des valeurs.

### Contravariance

La contravariance permet d'utiliser un type moins dérivé (plus général) que celui initialement spécifié. En C#, elle est indiquée par le mot-clé `in` dans les définitions d'interfaces génériques et de délégués.

Exemples de types contravariants:
- `IComparer<in T>`
- `IEqualityComparer<in T>`
- `Action<in T>`

La contravariance est utile pour les types qui consomment ou acceptent des valeurs.

### Points clés à retenir

1. **Sécurité de type**: La covariance et la contravariance préservent la sécurité de type.

2. **Interfaces vs Classes**: En C#, seules les interfaces et les délégués génériques peuvent être covariants ou contravariants, pas les classes.

3. **Types référence uniquement**: La covariance et la contravariance ne fonctionnent qu'avec les types référence, pas avec les types valeur.

4. **Tableaux**: Les tableaux en C# sont naturellement covariants, mais cette covariance n'est pas sûre au type et peut causer des exceptions à l'exécution.

5. **Combinaison**: Une interface peut être à la fois covariante et contravariante sur différents paramètres de type.

## 6.4.4. Interfaces génériques

Les interfaces génériques définissent des contrats qui peuvent être implémentés par des types qui utilisent différents types de données, tout en préservant la sécurité de type.

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Interfaces génériques ===\n");

        // 1. Utilisation d'interfaces génériques standard
        Console.WriteLine("--- Interfaces génériques standard ---");

        // IEnumerable<T> - pour l'itération
        IEnumerable<string> names = new List<string> { "Alice", "Bob", "Charlie" };
        Console.WriteLine("Noms (via IEnumerable<string>):");
        foreach (var name in names)
        {
            Console.WriteLine($" - {name}");
        }

        // IList<T> - liste modifiable
        IList<int> numbers = new List<int> { 1, 2, 3 };
        numbers.Add(4);
        numbers.Remove(2);
        Console.WriteLine("\nNombres (après modifications via IList<int>): " + string.Join(", ", numbers));

        // IDictionary<TKey, TValue> - dictionnaire clé-valeur
        IDictionary<string, int> ages = new Dictionary<string, int>
        {
            ["Alice"] = 30,
            ["Bob"] = 25,
            ["Charlie"] = 35
        };
        Console.WriteLine("\nÂges (via IDictionary<string, int>):");
        foreach (var kvp in ages)
        {
            Console.WriteLine($" - {kvp.Key}: {kvp.Value}");
        }

        // 2. Implémentation d'interfaces génériques personnalisées
        Console.WriteLine("\n--- Implémentation d'interfaces génériques personnalisées ---");

        // Interface générique IRepository<T>
        IRepository<Product> productRepo = new ProductRepository();
        productRepo.Add(new Product { Id = 1, Name = "Laptop", Price = 999.99m });
        productRepo.Add(new Product { Id = 2, Name = "Smartphone", Price = 699.99m });

        Console.WriteLine("Produits dans le dépôt:");
        foreach (var product in productRepo.GetAll())
        {
            Console.WriteLine($" - {product.Id}: {product.Name}, ${product.Price}");
        }

        var laptop = productRepo.GetById(1);
        Console.WriteLine($"\nProduit avec ID 1: {laptop?.Name}");

        // Interface générique avec plusieurs paramètres de type
        IKeyValueStorage<int, string> storage = new InMemoryKeyValueStorage<int, string>();
        storage.SetValue(1, "One");
        storage.SetValue(2, "Two");

        Console.WriteLine($"\nValeur pour la clé 1: {storage.GetValue(1)}");
        Console.WriteLine($"Le stockage contient-il la clé 3? {storage.ContainsKey(3)}");

        // 3. Implémentations multiples d'interfaces génériques
        Console.WriteLine("\n--- Implémentations multiples d'interfaces génériques ---");

        DataStore<int> intStore = new DataStore<int>();
        intStore.Add(10);
        intStore.Add(20);
        intStore.Add(30);

        Console.WriteLine("Données du DataStore<int>:");
        foreach (var item in intStore)
        {
            Console.WriteLine($" - {item}");
        }

        int count = intStore.Count;
        bool contains20 = intStore.Contains(20);

        Console.WriteLine($"Nombre d'éléments: {count}");
        Console.WriteLine($"Contient 20? {contains20}");

        // 4. Implémentation d'interfaces génériques covariantes et contravariantes
        Console.WriteLine("\n--- Interfaces covariantes et contravariantes ---");

        // Covariance (utilisant out T)
        IFactory<Dog> dogFactory = new DogFactory();
        IFactory<Animal> animalFactory = dogFactory; // Covariance

        Animal animal = animalFactory.Create();
        Console.WriteLine($"Animal créé: {animal.GetType().Name}");

        // Contravariance (utilisant in T)
        IValidator<Animal> animalValidator = new AnimalValidator();
        IValidator<Cat> catValidator = animalValidator; // Contravariance

        Cat cat = new Cat { Name = "Whiskers" };
        bool isValid = catValidator.Validate(cat);
        Console.WriteLine($"Chat valide? {isValid}");

        // 5. Interfaces génériques imbriquées
        Console.WriteLine("\n--- Interfaces génériques imbriquées ---");

        IService<IProcessor<string>> stringProcessorService = new ProcessorService<string>();
        stringProcessorService.Execute("Hello World");

        // 6. Utilisation d'interfaces génériques pour l'inversion de dépendances
        Console.WriteLine("\n--- Inversion de dépendances avec interfaces génériques ---");

        var userService = new UserService(new UserRepository());
        var users = userService.GetAllUsers();

        Console.WriteLine("Utilisateurs:");
        foreach (var user in users)
        {
            Console.WriteLine($" - {user.Name} ({user.Email})");
        }
    }
}

// Interface générique de base
interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Update(T item);
    void Delete(int id);
}

// Classe de modèle pour les exemples
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Implémentation concrète d'interface générique
class ProductRepository : IRepository<Product>
{
    private List<Product> _products = new List<Product>();

    public void Add(Product item)
    {
        _products.Add(item);
    }

    public Product GetById(int id)
    {
        return _products.Find(p => p.Id == id);
    }

    public IEnumerable<Product> GetAll()
    {
        return _products;
    }

    public void Update(Product item)
    {
        var index = _products.FindIndex(p => p.Id == item.Id);
        if (index != -1)
        {
            _products[index] = item;
        }
    }

    public void Delete(int id)
    {
        _products.RemoveAll(p => p.Id == id);
    }
}

// Interface générique avec plusieurs paramètres de type
interface IKeyValueStorage<TKey, TValue>
{
    void SetValue(TKey key, TValue value);
    TValue GetValue(TKey key);
    bool ContainsKey(TKey key);
    void RemoveKey(TKey key);
}

// Implémentation d'interface générique avec plusieurs paramètres
class InMemoryKeyValueStorage<TKey, TValue> : IKeyValueStorage<TKey, TValue>
{
    private Dictionary<TKey, TValue> _storage = new Dictionary<TKey, TValue>();

    public void SetValue(TKey key, TValue value)
    {
        _storage[key] = value;
    }

    public TValue GetValue(TKey key)
    {
        if (_storage.TryGetValue(key, out TValue value))
        {
            return value;
        }
        return default(TValue);
    }

    public bool ContainsKey(TKey key)
    {
        return _storage.ContainsKey(key);
    }

    public void RemoveKey(TKey key)
    {
        _storage.Remove(key);
    }
}

// Classe implémentant plusieurs interfaces génériques
class DataStore<T> : IEnumerable<T>, ICollection<T>
{
    private List<T> _items = new List<T>();

    // IEnumerable<T>
    public IEnumerator<T> GetEnumerator()
    {
        return _items.GetEnumerator();
    }

    // IEnumerable (non générique)
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    // ICollection<T>
    public int Count => _items.Count;
    public bool IsReadOnly => false;

    public void Add(T item)
    {
        _items.Add(item);
    }

    public void Clear()
    {
        _items.Clear();
    }

    public bool Contains(T item)
    {
        return _items.Contains(item);
    }

    public void CopyTo(T[] array, int arrayIndex)
    {
        _items.CopyTo(array, arrayIndex);
    }

    public bool Remove(T item)
    {
        return _items.Remove(item);
    }
}

// Classes pour la démonstration de covariance et contravariance

class Animal
{
    public string Name { get; set; }
}

class Dog : Animal { }

class Cat : Animal { }

// Interface générique covariante (out T)
interface IFactory<out T>
{
    T Create();
}

// Implémentation de l'interface covariante
class DogFactory : IFactory<Dog>
{
    public Dog Create()
    {
        return new Dog { Name = "New Dog" };
    }
}

// Interface générique contravariante (in T)
interface IValidator<in T>
{
    bool Validate(T item);
}

// Implémentation de l'interface contravariante
class AnimalValidator : IValidator<Animal>
{
    public bool Validate(Animal item)
    {
        return item != null && !string.IsNullOrEmpty(item.Name);
    }
}

// Interfaces génériques imbriquées
interface IProcessor<T>
{
    void Process(T data);
}

interface IService<T>
{
    void Execute(string command);
}

class StringProcessor : IProcessor<string>
{
    public void Process(string data)
    {
        Console.WriteLine($"Traitement: {data.ToUpper()}");
    }
}

class ProcessorService<T> : IService<IProcessor<T>>
{
    private IProcessor<T> _processor = (IProcessor<T>)new StringProcessor();

    public void Execute(string command)
    {
        Console.WriteLine($"Exécution de la commande: {command}");
        // Dans une implémentation réelle, on utiliserait le processeur
    }
}

// Exemple d'inversion de dépendances avec des interfaces génériques
class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

class UserRepository : IRepository<User>
{
    private List<User> _users = new List<User>
    {
        new User { Id = 1, Name = "Alice", Email = "alice@example.com" },
        new User { Id = 2, Name = "Bob", Email = "bob@example.com" }
    };

    public void Add(User item)
    {
        _users.Add(item);
    }

    public User GetById(int id)
    {
        return _users.Find(u => u.Id == id);
    }

    public IEnumerable<User> GetAll()
    {
        return _users;
    }

    public void Update(User item)
    {
        var index = _users.FindIndex(u => u.Id == item.Id);
        if (index != -1)
        {
            _users[index] = item;
        }
    }

    public void Delete(int id)
    {
        _users.RemoveAll(u => u.Id == id);
    }
}

class UserService
{
    private readonly IRepository<User> _userRepository;

    public UserService(IRepository<User> userRepository)
    {
        _userRepository = userRepository;
    }

    public IEnumerable<User> GetAllUsers()
    {
        return _userRepository.GetAll();
    }

    public User GetUserById(int id)
    {
        return _userRepository.GetById(id);
    }
}
```


### Interfaces génériques standard dans .NET

.NET Framework et .NET Core/.NET fournissent plusieurs interfaces génériques essentielles:

1. **Collections**:
   - `IEnumerable<T>`, `ICollection<T>`, `IList<T>`
   - `IDictionary<TKey, TValue>`, `ISet<T>`
   - `IReadOnlyCollection<T>`, `IReadOnlyList<T>`, `IReadOnlyDictionary<TKey, TValue>`

2. **Comparaison**:
   - `IComparable<T>`, `IComparer<T>`, `IEqualityComparer<T>`

3. **Fonctionnelles**:
   - `IDisposable`, `IAsyncDisposable` (.NET Core 3.0+)
   - `IObservable<T>`, `IObserver<T>`

4. **LINQ**:
   - `IQueryable<T>`, `IOrderedQueryable<T>`
   - `IGrouping<TKey, TElement>`

### Avantages des interfaces génériques

1. **Polymorphisme avec sécurité de type**: Combinent le polymorphisme des interfaces avec la sécurité de type des génériques.

2. **Réutilisabilité**: Définissent des comportements qui peuvent être implémentés avec différents types de données.

3. **Abstraction**: Permettent de définir des contrats sans se soucier des détails d'implémentation.

4. **Flexibilité**: Peuvent être implémentées de différentes manières pour différents contextes.

5. **Inversion de dépendances**: Facilitent l'injection de dépendances et le découplage du code.

6. **Testabilité**: Simplifient la création de mocks pour les tests unitaires.

### Cas d'utilisation courants

1. **Repositories et services**: Accès aux données avec différents types d'entités.

2. **Fabriques**: Création d'objets avec une interface consistante.

3. **Stratégies**: Encapsulation d'algorithmes variants pour différents types.

4. **Intergiciels**: Traitement de requêtes et réponses de différents types.

5. **Caches**: Stockage de différents types de données avec une interface uniforme.

6. **Validateurs**: Validation de règles métier pour différents types d'objets.

## 6.4.5. Generic type inference

L'inférence de type générique est une fonctionnalité du compilateur C# qui permet d'omettre les arguments de type explicites lors de l'appel de méthodes génériques, le compilateur déduisant automatiquement les types appropriés.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Inférence de type générique ===\n");

        // 1. Inférence de type basique avec des méthodes génériques
        Console.WriteLine("--- Inférence de type basique ---");

        // Utilisation sans inférence (explicite)
        string result1 = Wrap<string>("Hello");
        Console.WriteLine($"Résultat explicite: {result1}");

        // Utilisation avec inférence (type déduit de l'argument)
        string result2 = Wrap("World"); // Le compilateur déduit T comme étant string
        Console.WriteLine($"Résultat avec inférence: {result2}");

        int number = Wrap(42); // Le compilateur déduit T comme étant int
        Console.WriteLine($"Résultat avec inférence (int): {number}");

        // 2. Inférence avec plusieurs paramètres de type
        Console.WriteLine("\n--- Inférence avec plusieurs paramètres de type ---");

        // Explicite
        var pair1 = CreatePair<int, string>(1, "One");
        Console.WriteLine($"Paire explicite: ({pair1.Item1}, {pair1.Item2})");

        // Avec inférence
        var pair2 = CreatePair(2, "Two"); // Déduit (int, string)
        Console.WriteLine($"Paire avec inférence: ({pair2.Item1}, {pair2.Item2})");

        var pair3 = CreatePair("Three", 3.14); // Déduit (string, double)
        Console.WriteLine($"Paire avec inférence: ({pair3.Item1}, {pair3.Item2})");

        // 3. Inférence de type avec des méthodes d'extension
        Console.WriteLine("\n--- Inférence avec méthodes d'extension ---");

        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

        // LINQ utilise beaucoup l'inférence de type
        var evenNumbers = numbers.Where(n => n % 2 == 0); // Déduit IEnumerable<int>
        Console.WriteLine($"Nombres pairs: {string.Join(", ", evenNumbers)}");

        var doubledNumbers = numbers.Select(n => n * 2); // Déduit IEnumerable<int>
        Console.WriteLine($"Nombres doublés: {string.Join(", ", doubledNumbers)}");

        var numberStrings = numbers.Select(n => n.ToString()); // Déduit IEnumerable<string>
        Console.WriteLine($"Nombres convertis en chaînes: {string.Join(", ", numberStrings)}");

        // Notre propre méthode d'extension avec inférence
        var transformedNumbers = numbers.Transform(n => n * n + 1); // Déduit l'entrée comme int et la sortie comme int
        Console.WriteLine($"Nombres transformés: {string.Join(", ", transformedNumbers)}");

        var wordLengths = new[] { "apple", "banana", "cherry" }.Transform(s => s.Length); // Déduit l'entrée comme string et la sortie comme int
        Console.WriteLine($"Longueurs des mots: {string.Join(", ", wordLengths)}");

        // 4. Inférence avec délégués et expressions lambda
        Console.WriteLine("\n--- Inférence avec délégués et lambdas ---");

        // Inférence dans la création de délégués
        Converter<int, string> converter = n => n.ToString("D4");
        Console.WriteLine($"Convertisseur: {converter(42)}");

        // Inférence à travers les lambdas et les types de retour de méthode
        var processedData = ProcessItems(numbers, x => x * 10, items => items.Sum());
        Console.WriteLine($"Données traitées: {processedData}");

        var joinedText = ProcessItems(
            new[] { "Hello", "Generic", "World" },
            s => s.ToUpper(),
            items => string.Join(" ", items)
        );
        Console.WriteLine($"Texte traité: {joinedText}");

        // 5. Inférence dans les constructeurs génériques
        Console.WriteLine("\n--- Inférence dans les constructeurs génériques ---");

        // .NET Framework 4.7.2 n'a pas l'inférence pour les constructeurs
        // Avec .NET Core/.NET, on pourrait faire:

        #if NET6_0_OR_GREATER
        // Cette syntaxe fonctionne avec .NET 6+ (pas avec .NET Framework 4.7.2)
        // DataProcessor processor = new(42);
        // Console.WriteLine($"Processeur: {processor.Data}");
        #endif

        // Sans inférence de constructeur (nécessaire pour .NET Framework 4.7.2)
        var processor1 = new DataProcessor<int>(42);
        Console.WriteLine($"Processeur (explicite): {processor1.Data}");

        // Avec l'inférence manuelle via une méthode d'aide
        var processor2 = Create(42);  // Au lieu du constructeur directement
        Console.WriteLine($"Processeur (via helper): {processor2.Data}");

        // 6. Inférence dans les types anonymes
        Console.WriteLine("\n--- Inférence dans les types anonymes ---");

        var person = new { Name = "Alice", Age = 30 }; // Types inférés (string, int)
        Console.WriteLine($"Personne: {person.Name}, {person.Age} ans");

        // Combinaison d'inférence dans les projections LINQ
        var people = new[]
        {
            new { Name = "Alice", Age = 30 },
            new { Name = "Bob", Age = 25 },
            new { Name = "Charlie", Age = 35 }
        };

        var oldestPerson = people.OrderByDescending(p => p.Age).First();
        Console.WriteLine($"Personne la plus âgée: {oldestPerson.Name}, {oldestPerson.Age} ans");

        // 7. Limites de l'inférence de type
        Console.WriteLine("\n--- Limites de l'inférence de type ---");

        // Les cas où l'inférence ne peut pas déterminer le type
        // CreateEmpty(); // Erreur: ne peut pas déduire T

        List<int> emptyList = CreateEmpty<int>(); // Besoin d'être explicite
        emptyList.Add(42);
        Console.WriteLine($"Liste créée explicitement: {string.Join(", ", emptyList)}");

        // L'inférence ne fonctionne pas pour les types de retour
        //var value = GetDefault(); // Erreur: ne peut pas déduire T

        var defaultInt = GetDefault<int>();
        Console.WriteLine($"Valeur par défaut d'un int: {defaultInt}");
    }

    // 1. Méthode générique simple pour l'inférence
    static T Wrap<T>(T value)
    {
        Console.WriteLine($"Traitement d'une valeur de type {typeof(T).Name}");
        return value;
    }

    // 2. Méthode avec plusieurs paramètres de type
    static Tuple<T1, T2> CreatePair<T1, T2>(T1 first, T2 second)
    {
        return new Tuple<T1, T2>(first, second);
    }

    // 3. Méthode d'extension générique
    public static IEnumerable<TOutput> Transform<TInput, TOutput>(
        this IEnumerable<TInput> source,
        Func<TInput, TOutput> transformer)
    {
        foreach (var item in source)
        {
            yield return transformer(item);
        }
    }

    // 4. Méthode utilisant des délégués génériques avec inférence
    static TResult ProcessItems<TItem, TIntermediate, TResult>(
        IEnumerable<TItem> items,
        Func<TItem, TIntermediate> transformer,
        Func<IEnumerable<TIntermediate>, TResult> aggregator)
    {
        var transformed = items.Select(transformer);
        return aggregator(transformed);
    }

    // 5. Helper pour l'inférence dans les constructeurs (.NET Framework 4.7.2)
    static DataProcessor<T> Create<T>(T data)
    {
        return new DataProcessor<T>(data);
    }

    // 6. Méthodes où l'inférence ne fonctionne pas
    static List<T> CreateEmpty<T>()
    {
        return new List<T>();
    }

    static T GetDefault<T>()
    {
        return default(T);
    }
}

// Classe générique utilisée pour démontrer l'inférence de constructeur
class DataProcessor<T>
{
    public T Data { get; }

    public DataProcessor(T data)
    {
        Data = data;
    }
}
```


### Comment fonctionne l'inférence de type

Le compilateur C# utilise plusieurs règles pour inférer les types génériques:

1. **Arguments de méthode**: Le type est déduit des types d'arguments passés à la méthode.

2. **Contexte d'utilisation**: Le type peut être influencé par la façon dont le résultat est utilisé.

3. **Types implicites**: L'utilisation de `var` permet au compilateur de déterminer le type à partir de l'expression.

4. **Expressions lambda**: Les types de paramètres des expressions lambda peuvent être déduits du contexte.

### Avantages de l'inférence de type

1. **Code plus concis**: Réduit la verbosité et améliore la lisibilité.

2. **Maintenance simplifiée**: Les changements de type se propagent automatiquement.

3. **Réduction des erreurs**: Moins de risques d'inconsistance entre les types déclarés et utilisés.

4. **Flexibilité**: Facilite l'utilisation de génériques complexes sans se soucier des détails de type.

### Limitations de l'inférence de type

1. **Méthodes sans paramètres**: L'inférence ne fonctionne pas pour les méthodes qui n'ont pas de paramètres.

2. **Types de retour**: L'inférence ne fonctionne que pour les paramètres, pas pour les types de retour seuls.

3. **Constructeurs dans .NET Framework**: L'inférence de constructeur n'est disponible que dans les versions récentes de C# (9.0+) et .NET Core/.NET.

4. **Ambiguïté**: Dans certains cas complexes, le compilateur peut ne pas être en mesure de déduire le type sans aide explicite.

### Meilleures pratiques

1. **Utiliser l'inférence quand c'est possible**: Pour un code plus lisible et concis.

2. **Être explicite quand nécessaire**: Pour clarifier l'intention ou résoudre les ambiguïtés.

3. **Concevoir des API qui facilitent l'inférence**: Placer les paramètres génériques en tant que paramètres de méthode.

4. **Comprendre les limites**: Reconnaître quand l'inférence ne peut pas fonctionner et être explicite.

5. **Documentation**: Documenter les types génériques même si l'inférence est utilisée, pour une meilleure compréhension.

L'inférence de type générique est une fonctionnalité puissante qui simplifie considérablement l'utilisation des génériques en C#, rendant le code plus lisible tout en maintenant la sécurité de type.

⏭️
