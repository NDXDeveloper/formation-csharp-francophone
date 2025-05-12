# 6.1. Délégués et événements

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les délégués et les événements sont des concepts fondamentaux en C# qui permettent d'implémenter la programmation événementielle et d'établir des communications entre différentes parties d'une application. Ils constituent la base de nombreux modèles de conception modernes et sont essentiels pour développer des applications réactives et modulaires.

## 6.1.1. Types de délégués (Action, Func, Predicate)

Un délégué est un type qui représente des références à des méthodes avec une signature particulière. En d'autres termes, c'est un type qui encapsule une méthode. Les délégués permettent de passer des méthodes en tant que paramètres, de stocker des méthodes dans des variables, et de définir des fonctions de rappel (callbacks).

### Délégués génériques intégrés

Le framework .NET fournit plusieurs types de délégués génériques prédéfinis qui couvrent la plupart des cas d'utilisation courants :

#### Action

Les délégués `Action` représentent des méthodes qui ne retournent pas de valeur (void) et peuvent prendre entre 0 et 16 paramètres d'entrée.

```textmate
using System;

class Program
{
    static void Main()
    {
        // Action sans paramètre
        Action sayHello = () => Console.WriteLine("Hello, World!");
        sayHello(); // Affiche: Hello, World!

        // Action avec un paramètre
        Action<string> greet = (name) => Console.WriteLine($"Hello, {name}!");
        greet("Alice"); // Affiche: Hello, Alice!

        // Action avec plusieurs paramètres
        Action<string, int> introduceWithAge = (name, age) =>
            Console.WriteLine($"My name is {name} and I am {age} years old.");
        introduceWithAge("Bob", 30); // Affiche: My name is Bob and I am 30 years old.

        // Utilisation d'Action avec des méthodes nommées
        Action<int, int> calculate = AddNumbers;
        calculate(5, 10); // Appelle AddNumbers(5, 10)

        // Action avec closure (capture de variables)
        string prefix = "User: ";
        Action<string> displayWithPrefix = (message) =>
            Console.WriteLine($"{prefix}{message}");
        displayWithPrefix("Logged in"); // Affiche: User: Logged in
    }

    static void AddNumbers(int a, int b)
    {
        Console.WriteLine($"Sum: {a + b}");
    }
}
```


#### Func

Les délégués `Func` représentent des méthodes qui retournent une valeur et peuvent prendre entre 0 et 16 paramètres d'entrée. Le dernier type générique dans la définition d'un `Func` représente toujours le type de retour.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        // Func sans paramètre, qui retourne un string
        Func<string> getCurrentTime = () => DateTime.Now.ToShortTimeString();
        Console.WriteLine($"Current time: {getCurrentTime()}"); // Affiche l'heure actuelle

        // Func avec un paramètre, qui retourne un booléen
        Func<int, bool> isEven = (number) => number % 2 == 0;
        Console.WriteLine($"Is 4 even? {isEven(4)}"); // Affiche: Is 4 even? True
        Console.WriteLine($"Is 7 even? {isEven(7)}"); // Affiche: Is 7 even? False

        // Func avec plusieurs paramètres
        Func<int, int, int> add = (a, b) => a + b;
        Console.WriteLine($"5 + 3 = {add(5, 3)}"); // Affiche: 5 + 3 = 8

        // Func avec méthode nommée
        Func<double, double, double> calculate = CalculateDistance;
        Console.WriteLine($"Distance: {calculate(3, 4):F2}"); // Appelle CalculateDistance(3, 4)

        // Utilisation de Func avec LINQ
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        // Filtrer les nombres pairs
        List<int> evenNumbers = numbers.Where(isEven).ToList();
        Console.WriteLine("Even numbers: " + string.Join(", ", evenNumbers));

        // Transformer chaque nombre en son carré
        Func<int, int> square = x => x * x;
        List<int> squares = numbers.Select(square).ToList();
        Console.WriteLine("Squares: " + string.Join(", ", squares));
    }

    static double CalculateDistance(double x, double y)
    {
        return Math.Sqrt(x * x + y * y);
    }
}
```


#### Predicate

Le délégué `Predicate<T>` représente une méthode qui prend un paramètre du type T et retourne un booléen. Il est souvent utilisé pour tester si un objet satisfait une condition particulière.

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        // Predicate qui vérifie si un nombre est supérieur à 5
        Predicate<int> isGreaterThanFive = n => n > 5;

        // Utilisation avec la méthode FindAll
        List<int> largeNumbers = numbers.FindAll(isGreaterThanFive);
        Console.WriteLine("Numbers greater than 5: " + string.Join(", ", largeNumbers));

        // Utilisation avec la méthode Find
        int firstLargeNumber = numbers.Find(isGreaterThanFive);
        Console.WriteLine("First number greater than 5: " + firstLargeNumber);

        // Utilisation avec la méthode Exists
        bool hasLargeNumber = numbers.Exists(isGreaterThanFive);
        Console.WriteLine("Contains number greater than 5: " + hasLargeNumber);

        // Exemple avec une classe personnalisée
        List<Person> people = new List<Person>
        {
            new Person { Name = "Alice", Age = 25 },
            new Person { Name = "Bob", Age = 30 },
            new Person { Name = "Charlie", Age = 20 },
            new Person { Name = "David", Age = 35 }
        };

        // Predicate pour trouver des personnes adultes
        Predicate<Person> isAdult = p => p.Age >= 18;

        // Trouver les adultes
        List<Person> adults = people.FindAll(isAdult);
        Console.WriteLine("Adults:");
        foreach (var person in adults)
        {
            Console.WriteLine($"- {person.Name}, {person.Age} years old");
        }

        // Predicate combinés
        Predicate<Person> isSenior = p => p.Age >= 30;
        Predicate<Person> startsWithA = p => p.Name.StartsWith("A");

        // Trouver les personnes qui sont soit seniors, soit dont le nom commence par A
        Predicate<Person> combinedPredicate = p => isSenior(p) || startsWithA(p);
        List<Person> result = people.FindAll(combinedPredicate);

        Console.WriteLine("\nSeniors or names starting with A:");
        foreach (var person in result)
        {
            Console.WriteLine($"- {person.Name}, {person.Age} years old");
        }
    }
}

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```


### Délégués personnalisés

Bien que les délégués génériques intégrés (`Action`, `Func` et `Predicate`) couvrent la plupart des cas d'utilisation, vous pouvez également définir vos propres types de délégués lorsque cela est nécessaire.

```textmate
using System;

// Définition d'un délégué personnalisé
public delegate bool StringValidator(string text);

// Délégué avec plusieurs paramètres
public delegate double MathOperation(double x, double y);

class Program
{
    static void Main()
    {
        // Utilisation du délégué StringValidator
        StringValidator isLongEnough = text => text.Length >= 8;
        StringValidator hasUpperCase = text => text.Any(char.IsUpper);
        StringValidator hasDigit = text => text.Any(char.IsDigit);

        string password = "Password123";

        Console.WriteLine($"Password: {password}");
        Console.WriteLine($"Is long enough: {isLongEnough(password)}");
        Console.WriteLine($"Has uppercase: {hasUpperCase(password)}");
        Console.WriteLine($"Has digit: {hasDigit(password)}");

        // Validation de mot de passe avec délégué composite
        bool isValid = ValidatePassword(password, isLongEnough, hasUpperCase, hasDigit);
        Console.WriteLine($"Password is valid: {isValid}");

        // Utilisation du délégué MathOperation
        MathOperation add = (x, y) => x + y;
        MathOperation subtract = (x, y) => x - y;
        MathOperation multiply = (x, y) => x * y;
        MathOperation divide = (x, y) => x / y;

        Console.WriteLine("\nMath Operations:");
        Console.WriteLine($"5 + 3 = {add(5, 3)}");
        Console.WriteLine($"5 - 3 = {subtract(5, 3)}");
        Console.WriteLine($"5 * 3 = {multiply(5, 3)}");
        Console.WriteLine($"5 / 3 = {divide(5, 3):F2}");

        // Passage du délégué en paramètre
        Console.WriteLine("\nCalculator Results:");
        Console.WriteLine($"Calculate(10, 5, add) = {Calculate(10, 5, add)}");
        Console.WriteLine($"Calculate(10, 5, subtract) = {Calculate(10, 5, subtract)}");
    }

    static bool ValidatePassword(string password, params StringValidator[] validators)
    {
        foreach (var validator in validators)
        {
            if (!validator(password))
                return false;
        }
        return true;
    }

    static double Calculate(double a, double b, MathOperation operation)
    {
        return operation(a, b);
    }
}
```


## 6.1.2. Délégués multicasts

Une des caractéristiques les plus puissantes des délégués en C# est leur capacité à pointer vers plusieurs méthodes, appelées délégués multicasts. Lorsqu'un délégué multicast est appelé, toutes les méthodes qu'il référence sont exécutées séquentiellement.

```textmate
using System;

class Program
{
    // Définition d'un délégué simple
    delegate void Logger(string message);

    static void Main()
    {
        // Création de délégués individuels
        Logger consoleLogger = message => Console.WriteLine($"CONSOLE: {message}");
        Logger fileLogger = message => Console.WriteLine($"FILE: Writing to file: {message}");
        Logger databaseLogger = message => Console.WriteLine($"DATABASE: Saving to database: {message}");

        Console.WriteLine("=== Délégués individuels ===");
        consoleLogger("Test message");   // Appelle uniquement consoleLogger
        fileLogger("Test message");      // Appelle uniquement fileLogger

        Console.WriteLine("\n=== Combinaison de délégués ===");

        // Création d'un délégué multicast en combinant des délégués
        Logger multiLogger = consoleLogger + fileLogger;
        multiLogger("Log this message");  // Appelle consoleLogger puis fileLogger

        // Ajout d'un autre délégué au délégué multicast
        multiLogger += databaseLogger;
        multiLogger("Another message");   // Appelle consoleLogger, fileLogger, puis databaseLogger

        Console.WriteLine("\n=== Suppression d'un délégué ===");

        // Suppression d'un délégué du délégué multicast
        multiLogger -= fileLogger;
        multiLogger("Final message");     // Appelle consoleLogger puis databaseLogger

        Console.WriteLine("\n=== Gestion des exceptions ===");

        // Délégué qui lève une exception
        Logger buggyLogger = message =>
        {
            Console.WriteLine("BUGGY: About to throw an exception");
            throw new Exception("Simulated error");
        };

        // Ajout du délégué buggy au délégué multicast
        multiLogger += buggyLogger;

        try
        {
            multiLogger("This will cause an exception");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Exception caught: {ex.Message}");
            Console.WriteLine("Note: Previous delegates in the chain were executed!");
        }

        Console.WriteLine("\n=== Délégués multicasts avec retour de valeur ===");

        // Délégués Func qui retournent une valeur
        Func<int, int> doubleValue = x => { Console.WriteLine($"Doubling {x}"); return x * 2; };
        Func<int, int> squareValue = x => { Console.WriteLine($"Squaring {x}"); return x * x; };

        // Combinaison de délégués qui retournent une valeur
        Func<int, int> transform = doubleValue + squareValue;

        // Seule la dernière valeur de retour est conservée !
        int result = transform(5);
        Console.WriteLine($"Result of multicast delegate: {result}");  // Seule la valeur de squareValue est retournée

        // C'est pourquoi on utilise généralement Action pour les délégués multicasts
        Console.WriteLine("\n=== Manière sécurisée d'obtenir toutes les valeurs de retour ===");

        // Liste de délégués individuels pour pouvoir les appeler séparément
        var transformations = new List<Func<int, int>> { doubleValue, squareValue };

        int input = 5;
        var allResults = new List<int>();

        foreach (var transformation in transformations)
        {
            allResults.Add(transformation(input));
        }

        Console.WriteLine($"All results: {string.Join(", ", allResults)}");
    }
}
```


### Points importants sur les délégués multicasts

1. **Ordre d'exécution** : Les méthodes sont exécutées dans l'ordre où elles ont été ajoutées au délégué multicast.

2. **Valeurs de retour** : Pour les délégués qui retournent une valeur (comme `Func`), seule la valeur de retour de la dernière méthode appelée est renvoyée. Les valeurs de retour des autres méthodes sont perdues.

3. **Exceptions** : Si une méthode dans la chaîne de délégués lève une exception, les méthodes précédentes auront déjà été exécutées, mais les méthodes suivantes ne seront pas appelées.

4. **Opérateurs** : Les opérateurs `+=` et `-=` permettent d'ajouter et de retirer des délégués d'un délégué multicast.

5. **Null safe** : L'invocation d'un délégué null ne provoque pas d'exception `NullReferenceException`. Cependant, il est préférable de vérifier si un délégué est null avant de l'appeler.

```textmate
public void OnSomethingHappened(string data)
{
    // Version sécurisée pour éviter NullReferenceException
    SomethingHappened?.Invoke(data);

    // Équivalent à :
    // if (SomethingHappened != null)
    //    SomethingHappened(data);
}
```


## 6.1.3. Événements et pattern observateur

Les événements sont une implémentation du pattern observateur (Observer Design Pattern) qui permet à un objet (l'observateur) de s'abonner à un autre objet (l'observable) pour recevoir des notifications lorsque des événements spécifiques se produisent.

En C#, les événements sont basés sur les délégués mais avec des restrictions supplémentaires pour garantir l'encapsulation :

- Seule la classe qui déclare l'événement peut le déclencher (invoquer le délégué)
- Les classes externes peuvent seulement s'abonner ou se désabonner de l'événement

### Implémentation basique d'événements

```textmate
using System;

// Classe qui génère des événements (observable)
public class StockMonitor
{
    // Déclaration d'un délégué pour l'événement
    public delegate void StockPriceChangedHandler(string stock, decimal price);

    // Déclaration de l'événement basé sur le délégué
    public event StockPriceChangedHandler StockPriceChanged;

    // Méthode qui déclenche l'événement
    public void UpdateStockPrice(string stock, decimal price)
    {
        Console.WriteLine($"StockMonitor: Price of {stock} changed to {price:C}");

        // Déclencher l'événement (vérifier si quelqu'un est abonné)
        StockPriceChanged?.Invoke(stock, price);
    }
}

// Classe qui observe les événements (observateur)
public class StockAnalyzer
{
    public string Name { get; }

    public StockAnalyzer(string name)
    {
        Name = name;
    }

    // Méthode qui sera appelée lorsque l'événement est déclenché
    public void OnStockPriceChanged(string stock, decimal price)
    {
        Console.WriteLine($"{Name}: Analyzing price change of {stock} to {price:C}");
    }
}

// Classe qui observe les événements (autre observateur)
public class Investor
{
    public string Name { get; }

    public Investor(string name)
    {
        Name = name;
    }

    // Méthode qui sera appelée lorsque l'événement est déclenché
    public void OnStockPriceChanged(string stock, decimal price)
    {
        if (price > 100)
            Console.WriteLine($"{Name}: Considering selling {stock} at {price:C}");
        else
            Console.WriteLine($"{Name}: Considering buying {stock} at {price:C}");
    }
}

class Program
{
    static void Main()
    {
        // Créer l'objet observable
        StockMonitor monitor = new StockMonitor();

        // Créer les objets observateurs
        StockAnalyzer analyzer1 = new StockAnalyzer("Technical Analyzer");
        StockAnalyzer analyzer2 = new StockAnalyzer("Fundamental Analyzer");
        Investor investor = new Investor("John Smith");

        Console.WriteLine("=== Abonnement aux événements ===");

        // Abonner les observateurs à l'événement
        monitor.StockPriceChanged += analyzer1.OnStockPriceChanged;
        monitor.StockPriceChanged += analyzer2.OnStockPriceChanged;
        monitor.StockPriceChanged += investor.OnStockPriceChanged;

        // Déclencher l'événement
        Console.WriteLine("\n--- Premier changement de prix ---");
        monitor.UpdateStockPrice("MSFT", 110.5m);

        Console.WriteLine("\n=== Désabonnement d'un observateur ===");

        // Désabonner un observateur
        monitor.StockPriceChanged -= analyzer2.OnStockPriceChanged;

        // Déclencher à nouveau l'événement
        Console.WriteLine("\n--- Deuxième changement de prix ---");
        monitor.UpdateStockPrice("AAPL", 95.75m);

        Console.WriteLine("\n=== Abonnement avec lambda ===");

        // Abonnement avec expression lambda
        monitor.StockPriceChanged += (stock, price) =>
        {
            if (stock == "GOOG")
                Console.WriteLine($"Lambda observer: Special interest in {stock} at {price:C}");
        };

        // Déclencher l'événement avec Google
        Console.WriteLine("\n--- Troisième changement de prix ---");
        monitor.UpdateStockPrice("GOOG", 1220.50m);
    }
}
```


### Implémentation du pattern observateur avec des événements génériques

```textmate
using System;
using System.Collections.Generic;

// Arguments d'événement personnalisés pour transporter des données
public class TemperatureChangedEventArgs : EventArgs
{
    public double NewTemperature { get; }
    public double OldTemperature { get; }
    public DateTime ChangedAt { get; }

    public TemperatureChangedEventArgs(double newTemperature, double oldTemperature)
    {
        NewTemperature = newTemperature;
        OldTemperature = oldTemperature;
        ChangedAt = DateTime.Now;
    }
}

// Observable - classe qui génère les événements
public class TemperatureSensor
{
    private double _currentTemperature;
    private readonly string _location;

    // Déclaration d'un événement avec le type générique EventHandler<T>
    public event EventHandler<TemperatureChangedEventArgs> TemperatureChanged;

    public TemperatureSensor(string location, double initialTemperature)
    {
        _location = location;
        _currentTemperature = initialTemperature;
    }

    // Propriété avec notification de changement
    public double CurrentTemperature
    {
        get => _currentTemperature;
        set
        {
            if (value != _currentTemperature)
            {
                double oldTemperature = _currentTemperature;
                _currentTemperature = value;

                // Déclencher l'événement
                OnTemperatureChanged(new TemperatureChangedEventArgs(value, oldTemperature));
            }
        }
    }

    // Méthode protégée pour déclencher l'événement
    protected virtual void OnTemperatureChanged(TemperatureChangedEventArgs e)
    {
        // Pattern thread-safe pour déclencher un événement
        TemperatureChanged?.Invoke(this, e);
    }

    // Simulation de changements de température aléatoires
    public void SimulateTemperatureChange()
    {
        // Génère une variation aléatoire entre -1.5 et +1.5 degrés
        Random rnd = new Random();
        double change = (rnd.NextDouble() * 3) - 1.5;

        // Mettre à jour la température (ceci déclenchera l'événement)
        CurrentTemperature += change;

        Console.WriteLine($"[{_location}] Temperature changed to {CurrentTemperature:F1}°C");
    }
}

// Observer - classe qui réagit aux événements
public class TemperatureMonitor
{
    public string Name { get; }

    public TemperatureMonitor(string name)
    {
        Name = name;
    }

    // Méthode pour s'abonner à un capteur
    public void Subscribe(TemperatureSensor sensor)
    {
        sensor.TemperatureChanged += OnTemperatureChanged;
        Console.WriteLine($"{Name} is now monitoring the sensor.");
    }

    // Méthode pour se désabonner d'un capteur
    public void Unsubscribe(TemperatureSensor sensor)
    {
        sensor.TemperatureChanged -= OnTemperatureChanged;
        Console.WriteLine($"{Name} has stopped monitoring the sensor.");
    }

    // Gestionnaire d'événement
    private void OnTemperatureChanged(object sender, TemperatureChangedEventArgs e)
    {
        if (sender is TemperatureSensor sensor)
        {
            Console.WriteLine($"{Name} detected: Temperature is now {e.NewTemperature:F1}°C (changed from {e.OldTemperature:F1}°C at {e.ChangedAt.ToLongTimeString()})");

            // Exemple de logique métier basée sur le changement de température
            double change = e.NewTemperature - e.OldTemperature;

            if (Math.Abs(change) > 1.0)
            {
                Console.WriteLine($"{Name} ALERT: Significant temperature change of {change:F1}°C!");
            }

            if (e.NewTemperature > 30)
            {
                Console.WriteLine($"{Name} WARNING: Temperature is too high!");
            }
            else if (e.NewTemperature < 10)
            {
                Console.WriteLine($"{Name} WARNING: Temperature is too low!");
            }
        }
    }
}

// Observer spécialisé pour l'archivage
public class TemperatureLogger
{
    private readonly List<string> _log = new List<string>();

    public void Subscribe(TemperatureSensor sensor)
    {
        sensor.TemperatureChanged += OnTemperatureChanged;
        Console.WriteLine("Temperature Logger is now recording.");
    }

    private void OnTemperatureChanged(object sender, TemperatureChangedEventArgs e)
    {
        string entry = $"{e.ChangedAt}: Temperature changed from {e.OldTemperature:F1}°C to {e.NewTemperature:F1}°C";
        _log.Add(entry);

        // Afficher juste un indicateur pour montrer que l'enregistrement a eu lieu
        Console.WriteLine("Temperature Logger: Entry recorded.");
    }

    public void DisplayLog()
    {
        Console.WriteLine("\n=== Temperature Log ===");
        foreach (var entry in _log)
        {
            Console.WriteLine(entry);
        }
    }
}

class Program
{
    static void Main()
    {
        // Créer les capteurs de température
        TemperatureSensor kitchenSensor = new TemperatureSensor("Kitchen", 22.5);
        TemperatureSensor outdoorSensor = new TemperatureSensor("Outdoor", 15.0);

        // Créer les moniteurs
        TemperatureMonitor homeMonitor = new TemperatureMonitor("Home Monitor");
        TemperatureMonitor mobileApp = new TemperatureMonitor("Mobile App");
        TemperatureLogger logger = new TemperatureLogger();

        // Abonnement des moniteurs aux capteurs
        homeMonitor.Subscribe(kitchenSensor);
        homeMonitor.Subscribe(outdoorSensor);
        mobileApp.Subscribe(kitchenSensor);
        logger.Subscribe(kitchenSensor);

        Console.WriteLine("\n=== Simulation de changements de température ===");

        // Simuler quelques changements de température
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"\n--- Iteration {i+1} ---");
            kitchenSensor.SimulateTemperatureChange();
            outdoorSensor.SimulateTemperatureChange();

            // Pause pour la lisibilité
            Console.WriteLine("Press Enter to continue...");
            Console.ReadLine();
        }

        // Se désabonner d'un capteur
        Console.WriteLine("\n=== Désabonnement ===");
        mobileApp.Unsubscribe(kitchenSensor);

        // Un autre changement
        Console.WriteLine("\n--- After unsubscribe ---");
        kitchenSensor.SimulateTemperatureChange();

        // Afficher le journal
        logger.DisplayLog();
    }
}
```


### Bonnes pratiques pour les événements

1. **Utilisez les types génériques EventHandler ou EventHandler<T>** : Ils offrent une convention standard pour la déclaration d'événements.

2. **Créez une méthode protégée virtuelle OnXxx** : Pour déclencher l'événement et permettre aux classes dérivées de surcharger le comportement.

3. **Vérifiez si l'événement est null avant de le déclencher** : Utilisez l'opérateur `?.Invoke()` ou vérifiez explicitement si l'événement est null.

4. **Capturez l'événement dans une variable locale** avant de vérifier s'il est null et de l'invoquer (pour éviter les problèmes de concurrence).

```textmate
protected virtual void OnPropertyChanged(string propertyName)
{
    // Capture l'événement dans une variable locale
    PropertyChangedEventHandler handler = PropertyChanged;

    // Vérifiez si l'événement est null
    if (handler != null)
    {
        // Déclenchez l'événement
        handler(this, new PropertyChangedEventArgs(propertyName));
    }

    // Ou, en C# 6.0 et versions ultérieures, utilisez l'opérateur de propagation conditionnelle null
    // PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
```


5. **Assurez-vous que les gestionnaires d'événements ne lèvent pas d'exceptions** : Les exceptions non gérées peuvent affecter d'autres gestionnaires d'événements.

6. **Respectez la convention de nommage** : Utilisez le nom au passé pour le nom de l'événement (`ItemAdded`, `PropertyChanged`, etc.) et le nom au présent pour la méthode virtuelle (`OnItemAdded`, `OnPropertyChanged`, etc.).

## 6.1.4. EventHandler et EventArgs

.NET Framework fournit deux types génériques pour travailler avec les événements : `EventHandler` et `EventHandler<TEventArgs>`. Ces types sont conçus pour suivre le modèle d'événement standard de .NET.

### EventHandler et EventArgs de base

```textmate
using System;

// Classe qui utilise EventHandler (sans données supplémentaires)
public class Button
{
    // EventHandler est un délégué prédéfini pour les événements sans données
    // public delegate void EventHandler(object sender, EventArgs e);
    public event EventHandler Click;

    // Simuler un clic sur le bouton
    public void SimulateClick()
    {
        Console.WriteLine("Button was clicked");

        // Déclencher l'événement Click
        OnClick(EventArgs.Empty);
    }

    // Méthode protégée pour déclencher l'événement
    protected virtual void OnClick(EventArgs e)
    {
        Click?.Invoke(this, e);
    }
}

class Program
{
    static void Main()
    {
        // Créer un bouton
        Button button = new Button();

        // S'abonner à l'événement Click
        button.Click += OnButtonClick;

        // S'abonner avec une expression lambda
        button.Click += (sender, e) => {
            Console.WriteLine("Button clicked (lambda handler)");
        };

        // Simuler un clic
        button.SimulateClick();
    }

    // Gestionnaire d'événement
    static void OnButtonClick(object sender, EventArgs e)
    {
        Console.WriteLine("Button clicked (method handler)");

        if (sender is Button)
        {
            Console.WriteLine("The sender was a Button");
        }
    }
}
```


### EventHandler\<TEventArgs> pour les événements avec données

```textmate
using System;

// Classe d'arguments d'événement personnalisée
public class ProductEventArgs : EventArgs
{
    public string ProductName { get; }
    public decimal Price { get; }
    public int Quantity { get; }

    public ProductEventArgs(string productName, decimal price, int quantity)
    {
        ProductName = productName;
        Price = price;
        Quantity = quantity;
    }
}

// Classe qui génère des événements
public class ShoppingCart
{
    // Événement utilisant EventHandler<T>
    public event EventHandler<ProductEventArgs> ProductAdded;
    public event EventHandler<ProductEventArgs> ProductRemoved;
    public event EventHandler CartCleared;

    // Ajouter un produit au panier
    public void AddProduct(string name, decimal price, int quantity)
    {
        Console.WriteLine($"Adding {quantity} x {name} (${price}) to cart");

        // Simuler l'ajout à une collection interne
        // ...

        // Déclencher l'événement ProductAdded
        OnProductAdded(new ProductEventArgs(name, price, quantity));
    }

    // Retirer un produit du panier
    public void RemoveProduct(string name, decimal price, int quantity)
    {
        Console.WriteLine($"Removing {quantity} x {name} from cart");

        // Simuler le retrait d'une collection interne
        // ...

        // Déclencher l'événement ProductRemoved
        OnProductRemoved(new ProductEventArgs(name, price, quantity));
    }

    // Vider le panier
    public void Clear()
    {
        Console.WriteLine("Clearing the cart");

        // Simuler le vidage du panier
        // ...

        // Déclencher l'événement CartCleared
        OnCartCleared(EventArgs.Empty);
    }

    // Méthodes protégées pour déclencher les événements
    protected virtual void OnProductAdded(ProductEventArgs e)
    {
        ProductAdded?.Invoke(this, e);
    }

    protected virtual void OnProductRemoved(ProductEventArgs e)
    {
        ProductRemoved?.Invoke(this, e);
    }

    protected virtual void OnCartCleared(EventArgs e)
    {
        CartCleared?.Invoke(this, e);
    }
}

// Classe qui observe les événements du panier
public class OrderProcessor
{
    public OrderProcessor(ShoppingCart cart)
    {
        // S'abonner aux événements du panier
        cart.ProductAdded += OnProductAdded;
        cart.ProductRemoved += OnProductRemoved;
        cart.CartCleared += OnCartCleared;
    }

    private void OnProductAdded(object sender, ProductEventArgs e)
    {
        decimal total = e.Price * e.Quantity;
        Console.WriteLine($"OrderProcessor: Product added - {e.ProductName}, Total: ${total}");
    }

    private void OnProductRemoved(object sender, ProductEventArgs e)
    {
        Console.WriteLine($"OrderProcessor: Product removed - {e.ProductName}");
    }

    private void OnCartCleared(object sender, EventArgs e)
    {
        Console.WriteLine("OrderProcessor: Cart has been cleared");
    }
}

// Classe qui observe les événements pour la journalisation
public class CartLogger
{
    public CartLogger(ShoppingCart cart)
    {
        // S'abonner à tous les événements avec une seule méthode
        cart.ProductAdded += LogCartEvent;
        cart.ProductRemoved += LogCartEvent;
        cart.CartCleared += LogCartEvent;
    }

    private void LogCartEvent(object sender, EventArgs e)
    {
        string eventDetails;

        if (e is ProductEventArgs productArgs)
        {
            eventDetails = $"Product: {productArgs.ProductName}, Quantity: {productArgs.Quantity}, Price: ${productArgs.Price}";
        }
        else
        {
            eventDetails = "No product details available";
        }

        Console.WriteLine($"LOG [{DateTime.Now}]: Cart event - {eventDetails}");
    }
}

class Program
{
    static void Main()
    {
        // Créer le panier d'achat
        ShoppingCart cart = new ShoppingCart();

        // Créer les observateurs
        OrderProcessor processor = new OrderProcessor(cart);
        CartLogger logger = new CartLogger(cart);

        // Manipuler le panier
        Console.WriteLine("\n=== Adding products ===");
        cart.AddProduct("Laptop", 999.99m, 1);
        cart.AddProduct("Mouse", 25.50m, 2);

        Console.WriteLine("\n=== Removing a product ===");
        cart.RemoveProduct("Mouse", 25.50m, 1);

        Console.WriteLine("\n=== Clearing the cart ===");
        cart.Clear();
    }
}
```


### Implémentation de l'interface INotifyPropertyChanged

L'interface `INotifyPropertyChanged` est largement utilisée pour la liaison de données dans les applications .NET, notamment dans WPF, UWP, et Xamarin.Forms. Elle permet aux objets de notifier les observateurs (généralement l'interface utilisateur) lorsqu'une propriété change.

```textmate
using System;
using System.ComponentModel;
using System.Collections.Generic;

// Classe qui implémente INotifyPropertyChanged
public class Person : INotifyPropertyChanged
{
    private string _firstName;
    private string _lastName;
    private int _age;

    // Événement requis par INotifyPropertyChanged
    public event PropertyChangedEventHandler PropertyChanged;

    // Propriétés avec notification de changement
    public string FirstName
    {
        get => _firstName;
        set
        {
            if (_firstName != value)
            {
                _firstName = value;
                OnPropertyChanged(nameof(FirstName));
                OnPropertyChanged(nameof(FullName)); // Propriété dépendante
            }
        }
    }

    public string LastName
    {
        get => _lastName;
        set
        {
            if (_lastName != value)
            {
                _lastName = value;
                OnPropertyChanged(nameof(LastName));
                OnPropertyChanged(nameof(FullName)); // Propriété dépendante
            }
        }
    }

    public int Age
    {
        get => _age;
        set
        {
            if (_age != value)
            {
                _age = value;
                OnPropertyChanged(nameof(Age));
            }
        }
    }

    // Propriété calculée
    public string FullName => $"{FirstName} {LastName}";

    // Méthode pour déclencher l'événement PropertyChanged
    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}

// Classe qui observe les changements de propriété
public class PersonViewModel
{
    private readonly Person _person;

    public PersonViewModel(Person person)
    {
        _person = person;

        // S'abonner à l'événement PropertyChanged
        _person.PropertyChanged += OnPersonPropertyChanged;
    }

    private void OnPersonPropertyChanged(object sender, PropertyChangedEventArgs e)
    {
        Console.WriteLine($"Property changed: {e.PropertyName}");

        // Réagir différemment selon la propriété qui a changé
        switch (e.PropertyName)
        {
            case nameof(Person.FirstName):
            case nameof(Person.LastName):
                Console.WriteLine($"Name is now: {_person.FullName}");
                break;
            case nameof(Person.Age):
                Console.WriteLine($"Age is now: {_person.Age}");
                break;
            case nameof(Person.FullName):
                // Mise à jour de l'interface utilisateur pour la propriété FullName
                Console.WriteLine($"Full name updated: {_person.FullName}");
                break;
        }
    }
}

// Exemple d'utilisation de INotifyPropertyChanged
class Program
{
    static void Main()
    {
        // Créer une personne
        Person person = new Person
        {
            FirstName = "John",
            LastName = "Doe",
            Age = 30
        };

        // Créer le ViewModel qui observe la personne
        PersonViewModel viewModel = new PersonViewModel(person);

        Console.WriteLine("\n=== Changing properties ===");

        // Modifier les propriétés pour déclencher les événements
        person.FirstName = "Jane";
        person.Age = 31;
        person.LastName = "Smith";
    }
}
```


### Bonnes pratiques pour EventHandler et EventArgs

1. **Dérivez de EventArgs pour les données d'événement** : Créez des classes qui héritent de `EventArgs` pour encapsuler les données d'événement.

2. **Nommez les classes d'arguments d'événement selon le modèle XXXEventArgs** : Par exemple, `ProductAddedEventArgs`.

3. **Utilisez EventHandler<T> plutôt que des délégués personnalisés** : Cela suit le modèle standard de .NET.

4. **Rendez les propriétés EventArgs en lecture seule** : Les données d'événement ne devraient pas être modifiables par les gestionnaires d'événements.

5. **Utilisez des méthodes protégées virtuelles OnXXX** : Pour permettre l'extension par héritage.

6. **Utilisez nameof pour les noms de propriétés** : Afin d'éviter les erreurs de frappe et faciliter le refactoring.

```textmate
// Recommandé
OnPropertyChanged(nameof(FirstName));

// À éviter
OnPropertyChanged("FirstName");
```


7. **Considérez l'utilisation de CallerMemberName pour INotifyPropertyChanged** : Pour automatiser la notification de changement de propriété.

```textmate
using System.Runtime.CompilerServices;

protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
{
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}

// Utilisation simplifiée sans spécifier le nom de la propriété
public string FirstName
{
    get => _firstName;
    set
    {
        if (_firstName != value)
        {
            _firstName = value;
            OnPropertyChanged(); // propertyName est automatiquement "FirstName"
        }
    }
}
```


8. **Évitez d'accéder aux propriétés dans les gestionnaires d'événements qui pourraient les modifier** : Cela pourrait créer des boucles infinies de notifications.

9. **Considérez l'accès thread-safe pour les événements** : Capturez l'événement dans une variable locale avant de vérifier s'il est null et de l'invoquer.

### Comparaison des différentes approches pour les délégués et événements

```textmate
using System;

class DelegateComparison
{
    // Approche 1: Délégué personnalisé
    public delegate void LogHandler(string message);

    // Approche 2: Action générique
    public Action<string> LogAction;

    // Approche 3: Événement avec délégué personnalisé
    public event LogHandler LogEvent;

    // Approche 4: Événement avec Action
    public event Action<string> LogActionEvent;

    // Approche 5: Événement avec EventHandler<T>
    public event EventHandler<LogEventArgs> LogStandardEvent;

    public void TriggerAllLogging(string message)
    {
        Console.WriteLine("\n=== Triggering all logging mechanisms ===");

        // Invocation du délégué
        if (LogHandler != null)
        {
            Console.WriteLine("1. Custom delegate:");
            LogHandler(message);
        }

        // Invocation de l'Action
        if (LogAction != null)
        {
            Console.WriteLine("2. Action<T>:");
            LogAction(message);
        }

        // Invocation de l'événement avec délégué personnalisé
        if (LogEvent != null)
        {
            Console.WriteLine("3. Event with custom delegate:");
            LogEvent(message);
        }

        // Invocation de l'événement avec Action
        if (LogActionEvent != null)
        {
            Console.WriteLine("4. Event with Action<T>:");
            LogActionEvent(message);
        }

        // Invocation de l'événement standard
        if (LogStandardEvent != null)
        {
            Console.WriteLine("5. Standard event pattern:");
            LogStandardEvent(this, new LogEventArgs(message));
        }
    }
}

// Classe pour les arguments d'événement
public class LogEventArgs : EventArgs
{
    public string Message { get; }
    public DateTime Timestamp { get; }

    public LogEventArgs(string message)
    {
        Message = message;
        Timestamp = DateTime.Now;
    }
}

class Program
{
    static void Main()
    {
        var demo = new DelegateComparison();

        // 1. S'abonner au délégué personnalisé
        demo.LogHandler += message => Console.WriteLine($"  Custom delegate received: {message}");

        // 2. S'abonner à l'Action
        demo.LogAction += message => Console.WriteLine($"  Action received: {message}");

        // 3. S'abonner à l'événement avec délégué personnalisé
        demo.LogEvent += message => Console.WriteLine($"  Event with custom delegate received: {message}");

        // 4. S'abonner à l'événement avec Action
        demo.LogActionEvent += message => Console.WriteLine($"  Event with Action received: {message}");

        // 5. S'abonner à l'événement standard
        demo.LogStandardEvent += (sender, e) =>
            Console.WriteLine($"  Standard event received: {e.Message} at {e.Timestamp}");

        // Déclencher tous les types de logging
        demo.TriggerAllLogging("Test message");

        Console.WriteLine("\n=== Points clés à retenir ===");
        Console.WriteLine("- Les délégués permettent l'assignation directe mais offrent moins d'encapsulation");
        Console.WriteLine("- Les événements protègent contre l'invocation externe et le remplacement accidentel");
        Console.WriteLine("- Action/Func sont plus concis que les délégués personnalisés");
        Console.WriteLine("- EventHandler<T> suit le modèle standard de .NET");
    }
}
```


### Conclusion

Les délégués et les événements sont des mécanismes essentiels en C# qui permettent d'implémenter le pattern observateur, de passer des méthodes en paramètres, et de construire des applications découplées.

- Les **délégués** fournissent un mécanisme pour référencer des méthodes, les passer en tant que paramètres, et créer des callbacks.
- Les **événements** sont basés sur les délégués mais ajoutent une couche d'encapsulation qui protège contre les appels indésirables.
- Les types génériques (`Action`, `Func`, `Predicate`, `EventHandler<T>`) simplifient l'utilisation des délégués et événements dans le code.
- Le pattern observateur, implémenté via des événements, est fondamental pour construire des applications réactives et modulaires.

En maîtrisant ces concepts, vous pouvez créer des architectures flexibles où les composants communiquent entre eux de manière faiblement couplée, facilitant ainsi la maintenance et l'extension de votre code.

⏭️ 6.2. [Expressions Lambda](/06-fonctionnalites-avancees-de-csharp/6-2-expressions-lambda.md)
