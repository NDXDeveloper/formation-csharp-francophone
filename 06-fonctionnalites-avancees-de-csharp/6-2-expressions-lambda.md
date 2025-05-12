# 6.2. Expressions Lambda

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les expressions lambda sont une fonctionnalité essentielle du C# moderne qui permettent de créer des fonctions anonymes de façon concise. Elles simplifient le code tout en s'intégrant parfaitement avec LINQ, les délégués et les APIs asynchrones. Les expressions lambda sont particulièrement utiles pour définir des opérations in-line sans avoir besoin de créer des méthodes nommées distinctes.

## 6.2.1. Syntaxe et utilisation

Une expression lambda est définie par des paramètres d'entrée suivis d'un opérateur lambda `=>` (prononcé "va vers" ou "devient"), puis d'une expression ou un bloc de code.

### Syntaxe de base

La forme générale d'une expression lambda est :

```textmate
(paramètres) => expression_ou_bloc_de_code
```


### Exemples de syntaxe

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Syntaxe des expressions lambda ===");

        // Expression lambda sans paramètre
        Action sayHello = () => Console.WriteLine("Hello, World!");
        sayHello(); // Affiche: Hello, World!

        // Expression lambda avec un paramètre
        // Les parenthèses sont optionnelles lorsqu'il n'y a qu'un seul paramètre sans type explicite
        Action<string> greet = name => Console.WriteLine($"Hello, {name}!");
        greet("Alice"); // Affiche: Hello, Alice!

        // Expression lambda avec type de paramètre explicite
        Action<string> greetTyped = (string name) => Console.WriteLine($"Hello, {name}!");
        greetTyped("Bob"); // Affiche: Hello, Bob!

        // Expression lambda avec plusieurs paramètres
        Func<int, int, int> add = (x, y) => x + y;
        Console.WriteLine($"5 + 3 = {add(5, 3)}"); // Affiche: 5 + 3 = 8

        // Expression lambda avec bloc de code
        Func<int, int> factorial = n =>
        {
            int result = 1;
            for (int i = 1; i <= n; i++)
            {
                result *= i;
            }
            return result;
        };
        Console.WriteLine($"Factorial of 5 = {factorial(5)}"); // Affiche: Factorial of 5 = 120

        // Expression lambda avec des types explicites
        Func<double, double, double> power = (double baseNum, double exponent) => Math.Pow(baseNum, exponent);
        Console.WriteLine($"2^3 = {power(2, 3)}"); // Affiche: 2^3 = 8

        // Lambda comme prédicat dans LINQ
        int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        var evenNumbers = numbers.Where(n => n % 2 == 0);
        Console.WriteLine("Even numbers: " + string.Join(", ", evenNumbers)); // Affiche: Even numbers: 2, 4, 6, 8, 10

        // Lambda comme sélecteur dans LINQ
        var squares = numbers.Select(n => n * n);
        Console.WriteLine("Squares: " + string.Join(", ", squares)); // Affiche: Squares: 1, 4, 9, 16, 25, 36, 49, 64, 81, 100

        // Lambda avec LINQ pour le tri
        List<Person> people = new List<Person>
        {
            new Person { Name = "Charlie", Age = 25 },
            new Person { Name = "Alice", Age = 30 },
            new Person { Name = "Bob", Age = 20 },
            new Person { Name = "Dave", Age = 35 }
        };

        // Tri par âge
        var sortedByAge = people.OrderBy(p => p.Age);

        Console.WriteLine("\nPeople sorted by age:");
        foreach (var person in sortedByAge)
        {
            Console.WriteLine($"{person.Name}: {person.Age}");
        }

        // Tri par longueur du nom puis par nom
        var sortedByNameLength = people.OrderBy(p => p.Name.Length).ThenBy(p => p.Name);

        Console.WriteLine("\nPeople sorted by name length, then by name:");
        foreach (var person in sortedByNameLength)
        {
            Console.WriteLine($"{person.Name} ({person.Name.Length} chars): {person.Age}");
        }
    }
}

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```


### Inférence de type

Le compilateur C# peut souvent déterminer automatiquement les types des paramètres d'une expression lambda en fonction du contexte, ce qui permet d'écrire un code plus concis.

```textmate
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Inférence de type avec les lambdas ===");

        // Le compilateur déduit que 'n' est un int
        Func<int, bool> isEven = n => n % 2 == 0;

        // Le compilateur déduit que 'x' et 'y' sont des doubles
        Func<double, double, double> calculateDistance = (x, y) => Math.Sqrt(x * x + y * y);

        // Avec LINQ, le compilateur déduit les types à partir de la collection
        int[] numbers = { 1, 2, 3, 4, 5 };
        var result = numbers.Select(n => n.ToString());
        // 'n' est déduit comme int, et le résultat est IEnumerable<string>

        // Exemple avec des types complexes
        var products = new[]
        {
            new { Name = "Laptop", Price = 1000.0m },
            new { Name = "Phone", Price = 500.0m },
            new { Name = "Tablet", Price = 300.0m }
        };

        // Le compilateur déduit le type de 'p' à partir du tableau 'products'
        var expensiveProducts = products.Where(p => p.Price > 400.0m);

        Console.WriteLine("Expensive products:");
        foreach (var product in expensiveProducts)
        {
            Console.WriteLine($"{product.Name}: ${product.Price}");
        }
    }
}
```


### Utilisation des expressions lambda

Les expressions lambda sont couramment utilisées dans les contextes suivants :

#### 1. Avec les délégués et les événements

```textmate
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Lambdas avec délégués et événements ===");

        // Avec les délégués Action
        Action<string> log = message => Console.WriteLine($"LOG: {message}");
        log("Application started");

        // Avec les délégués Func
        Func<int, int, int> calculator = (x, y) => x * y;
        Console.WriteLine($"7 × 6 = {calculator(7, 6)}");

        // Avec les délégués Predicate
        Predicate<int> isPositive = n => n > 0;
        Console.WriteLine($"Is 5 positive? {isPositive(5)}");
        Console.WriteLine($"Is -3 positive? {isPositive(-3)}");

        // Avec les événements
        Button button = new Button();
        button.Click += (sender, e) => Console.WriteLine("Button was clicked!");
        button.SimulateClick();
    }
}

// Classe simple pour démontrer l'utilisation avec des événements
class Button
{
    public event EventHandler Click;

    public void SimulateClick()
    {
        Click?.Invoke(this, EventArgs.Empty);
    }
}
```


#### 2. Avec LINQ (Language Integrated Query)

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Lambdas avec LINQ ===");

        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        // Filtrage avec Where
        var evenNumbers = numbers.Where(n => n % 2 == 0);
        Console.WriteLine("Even numbers: " + string.Join(", ", evenNumbers));

        // Transformation avec Select
        var doubled = numbers.Select(n => n * 2);
        Console.WriteLine("Doubled: " + string.Join(", ", doubled));

        // Combinaison d'opérations LINQ
        var evenSquares = numbers
            .Where(n => n % 2 == 0)
            .Select(n => n * n);
        Console.WriteLine("Squares of even numbers: " + string.Join(", ", evenSquares));

        // Agrégation avec lambda
        int sum = numbers.Aggregate(0, (acc, n) => acc + n);
        Console.WriteLine($"Sum: {sum}");

        // GroupBy avec lambda
        var products = new List<Product>
        {
            new Product { Name = "Apple", Category = "Fruit", Price = 0.5m },
            new Product { Name = "Banana", Category = "Fruit", Price = 0.3m },
            new Product { Name = "Carrot", Category = "Vegetable", Price = 0.4m },
            new Product { Name = "Potato", Category = "Vegetable", Price = 0.6m }
        };

        var groupsByCategory = products.GroupBy(p => p.Category);

        foreach (var group in groupsByCategory)
        {
            Console.WriteLine($"\nCategory: {group.Key}");
            foreach (var product in group)
            {
                Console.WriteLine($"  - {product.Name} (${product.Price})");
            }
        }

        // Utilisation de plusieurs lambdas ensemble
        var expensiveFruits = products
            .Where(p => p.Category == "Fruit")
            .Where(p => p.Price > 0.4m)
            .OrderBy(p => p.Name)
            .Select(p => new { CustomName = p.Name.ToUpper(), p.Price });

        Console.WriteLine("\nExpensive fruits:");
        foreach (var item in expensiveFruits)
        {
            Console.WriteLine($"  {item.CustomName}: ${item.Price}");
        }
    }
}

class Product
{
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
}
```


#### 3. Avec les APIs asynchrones

```textmate
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("=== Lambdas avec programmation asynchrone ===");

        // Lambda asynchrone avec Task.Run
        await Task.Run(async () =>
        {
            Console.WriteLine("Starting async operation...");
            await Task.Delay(1000); // Simule une opération asynchrone
            Console.WriteLine("Async operation completed!");
        });

        // Liste de tâches à exécuter
        List<Func<Task>> asyncOperations = new List<Func<Task>>
        {
            async () =>
            {
                await Task.Delay(500);
                Console.WriteLine("First operation complete");
            },
            async () =>
            {
                await Task.Delay(300);
                Console.WriteLine("Second operation complete");
            },
            async () =>
            {
                await Task.Delay(800);
                Console.WriteLine("Third operation complete");
            }
        };

        // Exécution séquentielle des tâches
        Console.WriteLine("\nExecuting tasks sequentially:");
        foreach (var operation in asyncOperations)
        {
            await operation();
        }

        // Exécution parallèle des tâches
        Console.WriteLine("\nExecuting tasks in parallel:");
        await Task.WhenAll(asyncOperations.Select(op => op()));

        // Utilisation de ContinueWith avec lambda
        Console.WriteLine("\nUsing ContinueWith:");
        Task initialTask = Task.Run(() =>
        {
            Console.WriteLine("Initial task executing...");
            return "Initial task result";
        });

        Task continuationTask = initialTask.ContinueWith(task =>
        {
            string result = task.Result;
            Console.WriteLine($"Continuation task received: {result}");
            return $"Processed: {result}";
        });

        string finalResult = await continuationTask;
        Console.WriteLine($"Final result: {finalResult}");
    }
}
```


## 6.2.2. Closures

Les closures permettent à une fonction lambda de capturer et d'utiliser des variables de la portée dans laquelle elle est définie. Cette fonctionnalité est puissante mais peut également être source de comportements inattendus si elle n'est pas bien comprise.

### Concept de closure

Une closure se produit lorsqu'une fonction lambda capture des variables de la portée extérieure, ce qui lui permet d'accéder à ces variables même après que leur portée originale ait cessé d'exister.

```textmate
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Closures avec expressions lambda ===");

        // Exemple simple de closure
        int factor = 10;
        Func<int, int> multiplier = n => n * factor;

        Console.WriteLine($"5 × {factor} = {multiplier(5)}"); // Affiche: 5 × 10 = 50

        // La lambda capture et conserve la référence à 'factor'
        factor = 20;
        Console.WriteLine($"5 × {factor} = {multiplier(5)}"); // Affiche: 5 × 20 = 100

        // Création de multiples lambda avec des closures
        Console.WriteLine("\n=== Création de fonctions multiplicatrices ===");
        List<Func<int, int>> multipliers = new List<Func<int, int>>();

        // Piège courant : capture d'une variable dans une boucle
        for (int i = 1; i <= 3; i++)
        {
            multipliers.Add(n => n * i);
        }

        // Toutes les lambda capturent la même variable 'i'
        // qui vaut 4 à la fin de la boucle
        foreach (var func in multipliers)
        {
            Console.WriteLine($"5 × ? = {func(5)}"); // Toutes affichent 5 × ? = 20
        }

        // Solution : créer une variable locale dans la boucle
        Console.WriteLine("\n=== Solution correcte ===");
        multipliers.Clear();

        for (int i = 1; i <= 3; i++)
        {
            int currentI = i; // Création d'une variable locale spécifique à chaque itération
            multipliers.Add(n => n * currentI);
        }

        // Maintenant chaque lambda capture une variable différente
        foreach (var func in multipliers)
        {
            Console.WriteLine($"5 × ? = {func(5)}"); // Affiche 5, 10, 15
        }

        // Dans C# 8 et versions ultérieures, cette solution est également valide :
        // for (int i = 1; i <= 3; i++)
        // {
        //     // La variable d'itération est maintenant propre à chaque itération
        //     multipliers.Add(n => n * i);
        // }

        Console.WriteLine("\n=== Factory de fonctions avec closures ===");

        // Utilisation de closures pour créer une factory de fonctions
        Func<int, Func<int, int>> powerGenerator = exponent => base_ =>
        {
            int result = 1;
            for (int i = 0; i < exponent; i++)
            {
                result *= base_;
            }
            return result;
        };

        Func<int, int> square = powerGenerator(2);
        Func<int, int> cube = powerGenerator(3);

        Console.WriteLine($"3² = {square(3)}"); // Affiche: 3² = 9
        Console.WriteLine($"3³ = {cube(3)}");   // Affiche: 3³ = 27

        Console.WriteLine("\n=== Closures avec état partagé ===");

        // Création d'un compteur avec état via closure
        Func<int> createCounter()
        {
            int count = 0;
            return () => ++count;
        }

        var counter1 = createCounter();
        var counter2 = createCounter();

        Console.WriteLine($"Counter1: {counter1()}"); // Affiche: Counter1: 1
        Console.WriteLine($"Counter1: {counter1()}"); // Affiche: Counter1: 2
        Console.WriteLine($"Counter1: {counter1()}"); // Affiche: Counter1: 3

        Console.WriteLine($"Counter2: {counter2()}"); // Affiche: Counter2: 1 (séparé de counter1)

        Console.WriteLine("\n=== Pièges des closures et mémoire ===");

        // Démonstration de fuite de mémoire potentielle
        MemoryLeakDemo();

        // Force le garbage collector pour la démo
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }

    static void MemoryLeakDemo()
    {
        // Création d'un objet volumineux
        var largeObject = new byte[10_000_000]; // ~10MB

        Console.WriteLine("Large object created");

        // Création d'un timer qui référence l'objet volumineux via closure
        var timer = new System.Timers.Timer(5000);
        timer.Elapsed += (sender, e) =>
        {
            // Cette lambda capture largeObject, ce qui l'empêche d'être récupéré
            // par le garbage collector tant que le timer existe
            Console.WriteLine($"Timer elapsed. Large object size: {largeObject.Length}");
        };

        // Timer démarré mais jamais arrêté
        timer.Start();

        Console.WriteLine("Fonction terminée, mais largeObject reste en mémoire à cause de la closure");

        // Pour éviter la fuite, il faudrait:
        // timer.Stop();
        // timer.Dispose();
    }
}
```


### Closures en détail

1. **Capture de variable** : Lorsqu'une lambda fait référence à une variable extérieure, le compilateur C# crée une classe anonyme pour stocker cette variable. La lambda devient alors une méthode de cette classe.

2. **Durée de vie des variables capturées** : Les variables capturées dans une closure survivent tant que la lambda existe, même si la portée d'origine de la variable est terminée.

3. **Variables partagées vs copies** : Quand une variable est capturée, c'est une référence à cette variable qui est capturée, pas une copie de sa valeur.

4. **Modifications des variables capturées** : Toute modification d'une variable capturée est visible pour toutes les lambdas qui la capturent.

5. **Capture dans les boucles** : Avant C# 8.0, la capture d'une variable d'itération d'une boucle capturait la même variable pour toutes les itérations, ce qui pouvait conduire à des comportements inattendus. À partir de C# 8.0, chaque itération a sa propre variable.

### Bonnes pratiques pour les closures

- **Évitez de capturer des variables mutables** si possible, ou faites des copies locales.
- **Soyez conscient des captures dans les boucles**, surtout avec des versions antérieures à C# 8.0.
- **Limitez la portée des lambdas qui capturent des ressources volumineuses** pour éviter les fuites de mémoire.
- **Libérez correctement les ressources** capturées dans les closures.

## 6.2.3. Délégués vs lambdas

Les expressions lambda et les délégués sont étroitement liés, mais ils présentent des différences importantes dans leur syntaxe, leur utilisation et leurs performances.

### Comparaison syntaxique

```textmate
using System;

class Program
{
    // Déclaration d'un délégué
    delegate int MathOperation(int x, int y);

    static void Main()
    {
        Console.WriteLine("=== Comparaison de la syntaxe entre délégués et lambdas ===");

        // 1. Méthode nommée
        MathOperation addMethod = Add;
        Console.WriteLine($"Using named method: 5 + 3 = {addMethod(5, 3)}");

        // 2. Méthode anonyme (C# 2.0)
        MathOperation addAnonymous = delegate(int x, int y) { return x + y; };
        Console.WriteLine($"Using anonymous method: 5 + 3 = {addAnonymous(5, 3)}");

        // 3. Expression lambda (C# 3.0+)
        MathOperation addLambda = (x, y) => x + y;
        Console.WriteLine($"Using lambda expression: 5 + 3 = {addLambda(5, 3)}");

        // 4. Expression lambda avec bloc
        MathOperation addLambdaBlock = (x, y) => { return x + y; };
        Console.WriteLine($"Using lambda block: 5 + 3 = {addLambdaBlock(5, 3)}");

        // 5. Avec les types génériques prédéfinis
        Func<int, int, int> addFunc = (x, y) => x + y;
        Console.WriteLine($"Using Func<T,T,T>: 5 + 3 = {addFunc(5, 3)}");

        // Comparaison de la concision
        Console.WriteLine("\n=== Concision du code ===");

        var numbers = new int[] { 1, 2, 3, 4, 5 };

        // Avec méthode nommée
        Array.ForEach(numbers, PrintNumber);

        // Avec méthode anonyme
        Array.ForEach(numbers, delegate(int n) { Console.Write(n + " "); });
        Console.WriteLine();

        // Avec lambda
        Array.ForEach(numbers, n => Console.Write(n + " "));
        Console.WriteLine();
    }

    static int Add(int x, int y)
    {
        return x + y;
    }

    static void PrintNumber(int n)
    {
        Console.Write(n + " ");
    }
}
```


### Différences fonctionnelles

```textmate
using System;
using System.Linq;
using System.Collections.Generic;

class Program
{
    delegate void SimpleDelegate();

    static void Main()
    {
        Console.WriteLine("=== Différences fonctionnelles entre délégués et lambdas ===");

        Console.WriteLine("\n1. Conversion en type de délégué");

        // Les méthodes nommées peuvent être converties en n'importe quel délégué compatible
        Action<int> printAction1 = PrintNumber;
        Func<int, string> printFunc = n => n.ToString();

        // Les lambdas sont plus flexibles et peuvent être converties en différents types de délégués
        var lambda = (int n) => Console.WriteLine(n);
        Action<int> printAction2 = lambda;
        SimpleDelegate del = () => Console.WriteLine("Hello");

        Console.WriteLine("\n2. Inférence de type");

        // Les lambdas bénéficient de l'inférence de type
        List<string> names = new List<string> { "Alice", "Bob", "Charlie" };

        // Le type de 'name' est inféré à partir du contexte
        var filteredNames = names.Where(name => name.StartsWith("A"));

        Console.WriteLine("\n3. Expressions vs instructions");

        // Lambda expression (peut être une expression unique)
        Func<int, int> square = n => n * n;

        // Lambda statement (doit utiliser des accolades et return explicite)
        Func<int, int> cube = n =>
        {
            int result = n * n * n;
            return result;
        };

        Console.WriteLine($"Square of 4: {square(4)}");
        Console.WriteLine($"Cube of 4: {cube(4)}");

        Console.WriteLine("\n4. Utilisation dans LINQ");

        // Les lambdas sont plus adaptées pour l'utilisation inline dans LINQ
        int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        // Utilisation de lambdas dans une chaîne LINQ
        var result = numbers
            .Where(n => n % 2 == 0)
            .Select(n => n * n)
            .OrderByDescending(n => n)
            .Take(3);

        Console.WriteLine("Top 3 squares of even numbers: " + string.Join(", ", result));

        Console.WriteLine("\n5. Performance et allocation mémoire");
        PerformanceExample();
    }

    static void PrintNumber(int n)
    {
        Console.WriteLine(n);
    }

    static void PerformanceExample()
    {
        // 1. Méthode nommée - généralement pas d'allocation
        Func<int, int> squareMethod = Square;

        // 2. Lambda sans capture - peut être mis en cache par le compilateur
        Func<int, int> squareLambda = n => n * n;

        // 3. Lambda avec closure - crée une allocation d'objet
        int factor = 2;
        Func<int, int> multiplier = n => n * factor;

        Console.WriteLine("La capture de variables dans des closures peut causer des allocations supplémentaires.");
        Console.WriteLine("Les lambdas sans capture et les méthodes nommées sont généralement plus efficaces.");
    }

    static int Square(int n)
    {
        return n * n;
    }
}
```


### Quand utiliser l'un plutôt que l'autre

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Cas d'utilisation des délégués vs lambdas ===");

        Console.WriteLine("\n1. Préférer les lambdas pour le code court et inline");

        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        // Bon cas d'utilisation pour lambda: opérations simples et inline
        var evenNumbers = numbers.Where(n => n % 2 == 0);
        Console.WriteLine("Even numbers: " + string.Join(", ", evenNumbers));

        Console.WriteLine("\n2. Préférer les méthodes nommées pour la logique complexe ou réutilisable");

        // Quand la logique est complexe, une méthode nommée est plus lisible
        var processedNumbers = numbers.Select(ProcessNumber);
        Console.WriteLine("Processed numbers: " + string.Join(", ", processedNumbers));

        Console.WriteLine("\n3. Préférer les méthodes nommées pour les gestionnaires d'événements importants");

        Button button = new Button();

        // Pour des gestionnaires d'événements complexes, utilisez des méthodes nommées
        button.Click += Button_Click;

        // Pour des gestionnaires simples, les lambdas sont plus concises
        button.Hover += (sender, e) => Console.WriteLine("Button hovered");

        button.SimulateClick();
        button.SimulateHover();

        Console.WriteLine("\n4. Considérations de performance");

        PerformanceConsiderations();
    }

    // Méthode complexe, mieux adaptée comme méthode nommée
    static int ProcessNumber(int n)
    {
        if (n < 5)
            return n * 2;
        else if (n < 8)
            return n * n;
        else
            return n / 2;
    }

    // Gestionnaire d'événement complexe
    static void Button_Click(object sender, EventArgs e)
    {
        Console.WriteLine("Button clicked");
        // Imaginez ici une logique plus complexe...
        if (sender is Button b)
        {
            Console.WriteLine("Processing button click...");
            // Plus de logique...
        }
    }

    static void PerformanceConsiderations()
    {
        Console.WriteLine("Performance considerations:");
        Console.WriteLine("- Les méthodes nommées ne créent pas d'objets délégués supplémentaires");
        Console.WriteLine("- Les lambdas sans capture sont souvent optimisées par le compilateur");
        Console.WriteLine("- Les lambdas avec capture créent des allocations supplémentaires");
        Console.WriteLine("- Pour le code critique en performance, préférez les méthodes nommées");
        Console.WriteLine("- Pour les opérations courantes, la simplicité des lambdas l'emporte souvent");
    }
}

// Classe simple pour la démonstration
class Button
{
    public event EventHandler Click;
    public event EventHandler Hover;

    public void SimulateClick()
    {
        Click?.Invoke(this, EventArgs.Empty);
    }

    public void SimulateHover()
    {
        Hover?.Invoke(this, EventArgs.Empty);
    }
}
```


### Résumé des différences

1. **Syntaxe** : Les lambdas offrent une syntaxe plus concise que les délégués traditionnels.

2. **Portabilité** : Les méthodes nommées peuvent être référencées à plusieurs endroits, tandis que les lambdas sont généralement définies là où elles sont utilisées.

3. **Inférence de type** : Les lambdas bénéficient de l'inférence de type, ce qui les rend plus flexibles.

4. **Closure** : Les lambdas peuvent capturer des variables externes, ce qui est plus difficile à faire avec des méthodes nommées.

5. **Performance** : Les méthodes nommées peuvent être plus efficaces pour le code critique en performance, car elles ne génèrent pas d'allocations supplémentaires.

6. **Lisibilité** : Pour la logique simple, les lambdas améliorent la lisibilité. Pour la logique complexe, les méthodes nommées sont souvent plus claires.

⏭️ 6.3. [LINQ (Language Integrated Query)](/06-fonctionnalites-avancees-de-csharp/6-3-linq-language-integrated-query.md)
