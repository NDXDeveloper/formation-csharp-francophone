# 6.3. LINQ (Language Integrated Query)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

LINQ (Language Integrated Query) est une fonctionnalité puissante de C# qui permet d'intégrer des opérations de requête directement dans le langage. LINQ fournit un ensemble unifié de méthodes d'extension qui permettent d'interroger, de filtrer et de manipuler diverses collections de données (tableaux, listes, XML, bases de données, etc.) avec une syntaxe cohérente et expressive.

## 6.3.1. Syntaxe de requête

La syntaxe de requête LINQ est inspirée du SQL et utilise des mots-clés comme `from`, `where`, `select`, `orderby`, etc. Cette syntaxe est souvent plus lisible pour les requêtes complexes.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        // Une collection simple pour les exemples
        List<Product> products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Electronics" },
            new Product { Id = 2, Name = "Mouse", Price = 25.50m, Category = "Electronics" },
            new Product { Id = 3, Name = "Desk Chair", Price = 179.99m, Category = "Furniture" },
            new Product { Id = 4, Name = "Coffee Table", Price = 249.99m, Category = "Furniture" },
            new Product { Id = 5, Name = "Headphones", Price = 149.99m, Category = "Electronics" },
            new Product { Id = 6, Name = "Bookshelf", Price = 159.99m, Category = "Furniture" },
            new Product { Id = 7, Name = "Smartphone", Price = 699.99m, Category = "Electronics" },
            new Product { Id = 8, Name = "Office Desk", Price = 299.99m, Category = "Furniture" }
        };

        Console.WriteLine("=== Syntaxe de requête LINQ ===\n");

        // Requête LINQ de base avec la syntaxe de requête
        var electronicsQuery =
            from product in products
            where product.Category == "Electronics"
            select product;

        Console.WriteLine("Produits électroniques:");
        foreach (var product in electronicsQuery)
        {
            Console.WriteLine($"- {product.Name} (${product.Price})");
        }

        // Requête avec tri
        var sortedFurnitureQuery =
            from product in products
            where product.Category == "Furniture"
            orderby product.Price ascending
            select product;

        Console.WriteLine("\nMeubles (triés par prix):");
        foreach (var product in sortedFurnitureQuery)
        {
            Console.WriteLine($"- {product.Name} (${product.Price})");
        }

        // Requête avec projection (sélection de propriétés spécifiques)
        var productInfoQuery =
            from product in products
            select new { product.Name, product.Price };

        Console.WriteLine("\nNoms et prix des produits:");
        foreach (var info in productInfoQuery)
        {
            Console.WriteLine($"- {info.Name}: ${info.Price}");
        }

        // Requête avec tri multiple
        var sortedProductsQuery =
            from product in products
            orderby product.Category, product.Price descending
            select product;

        Console.WriteLine("\nProduits triés par catégorie puis par prix (décroissant):");
        foreach (var product in sortedProductsQuery)
        {
            Console.WriteLine($"- {product.Category}: {product.Name} (${product.Price})");
        }

        // Requête avec regroupement
        var groupedProductsQuery =
            from product in products
            group product by product.Category into categoryGroup
            select new
            {
                Category = categoryGroup.Key,
                Products = categoryGroup.ToList(),
                Count = categoryGroup.Count(),
                AveragePrice = categoryGroup.Average(p => p.Price)
            };

        Console.WriteLine("\nProduits groupés par catégorie:");
        foreach (var group in groupedProductsQuery)
        {
            Console.WriteLine($"Catégorie: {group.Category} ({group.Count} produits, prix moyen: ${group.AveragePrice:F2})");
            foreach (var product in group.Products)
            {
                Console.WriteLine($"  - {product.Name} (${product.Price})");
            }
        }

        // Requête avec jointure
        List<Supplier> suppliers = new List<Supplier>
        {
            new Supplier { Id = 1, Name = "TechStore", Category = "Electronics" },
            new Supplier { Id = 2, Name = "FurniturePlus", Category = "Furniture" }
        };

        var joinQuery =
            from product in products
            join supplier in suppliers on product.Category equals supplier.Category
            select new
            {
                ProductName = product.Name,
                ProductPrice = product.Price,
                SupplierName = supplier.Name
            };

        Console.WriteLine("\nProduits avec leur fournisseur:");
        foreach (var item in joinQuery)
        {
            Console.WriteLine($"- {item.ProductName} (${item.ProductPrice}) fourni par {item.SupplierName}");
        }

        // Requête avec let (introduction de variable locale)
        var discountedPricesQuery =
            from product in products
            let discountRate = product.Price > 500 ? 0.10m : 0.05m
            let discountedPrice = product.Price * (1 - discountRate)
            where discountedPrice > 100
            select new
            {
                product.Name,
                OriginalPrice = product.Price,
                DiscountRate = discountRate * 100,
                FinalPrice = discountedPrice
            };

        Console.WriteLine("\nPrix après remise:");
        foreach (var item in discountedPricesQuery)
        {
            Console.WriteLine($"- {item.Name}: ${item.OriginalPrice} - {item.DiscountRate}% = ${item.FinalPrice:F2}");
        }
    }
}

class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}

class Supplier
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
}
```


### Caractéristiques de la syntaxe de requête

1. **Convivialité SQL** : La syntaxe ressemble au SQL, ce qui la rend plus accessible pour les développeurs familiers avec ce langage.

2. **Clauses** : Utilise des clauses spécifiques comme `from`, `where`, `select`, `orderby`, `group by`, etc.

3. **Lisibilité** : Souvent plus lisible pour les requêtes complexes, en particulier celles impliquant des jointures ou des groupements.

4. **Ordre des clauses** : L'ordre des clauses est important et doit suivre un modèle spécifique (commencer par `from`, etc.).

## 6.3.2. Syntaxe de méthode

La syntaxe de méthode LINQ utilise des méthodes d'extension sur les collections qui implémentent l'interface `IEnumerable<T>`. Cette approche est généralement plus concise et plus flexible.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        // Même collection de données que l'exemple précédent
        List<Product> products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Electronics" },
            new Product { Id = 2, Name = "Mouse", Price = 25.50m, Category = "Electronics" },
            new Product { Id = 3, Name = "Desk Chair", Price = 179.99m, Category = "Furniture" },
            new Product { Id = 4, Name = "Coffee Table", Price = 249.99m, Category = "Furniture" },
            new Product { Id = 5, Name = "Headphones", Price = 149.99m, Category = "Electronics" },
            new Product { Id = 6, Name = "Bookshelf", Price = 159.99m, Category = "Furniture" },
            new Product { Id = 7, Name = "Smartphone", Price = 699.99m, Category = "Electronics" },
            new Product { Id = 8, Name = "Office Desk", Price = 299.99m, Category = "Furniture" }
        };

        Console.WriteLine("=== Syntaxe de méthode LINQ ===\n");

        // Filtrage avec Where
        var electronics = products.Where(p => p.Category == "Electronics");

        Console.WriteLine("Produits électroniques:");
        foreach (var product in electronics)
        {
            Console.WriteLine($"- {product.Name} (${product.Price})");
        }

        // Filtrage et tri avec Where et OrderBy
        var sortedFurniture = products
            .Where(p => p.Category == "Furniture")
            .OrderBy(p => p.Price);

        Console.WriteLine("\nMeubles (triés par prix):");
        foreach (var product in sortedFurniture)
        {
            Console.WriteLine($"- {product.Name} (${product.Price})");
        }

        // Projection avec Select
        var productInfo = products.Select(p => new { p.Name, p.Price });

        Console.WriteLine("\nNoms et prix des produits:");
        foreach (var info in productInfo)
        {
            Console.WriteLine($"- {info.Name}: ${info.Price}");
        }

        // Tri multiple avec OrderBy et ThenByDescending
        var sortedProducts = products
            .OrderBy(p => p.Category)
            .ThenByDescending(p => p.Price);

        Console.WriteLine("\nProduits triés par catégorie puis par prix (décroissant):");
        foreach (var product in sortedProducts)
        {
            Console.WriteLine($"- {product.Category}: {product.Name} (${product.Price})");
        }

        // Groupement avec GroupBy
        var groupedProducts = products
            .GroupBy(p => p.Category)
            .Select(g => new
            {
                Category = g.Key,
                Products = g.ToList(),
                Count = g.Count(),
                AveragePrice = g.Average(p => p.Price)
            });

        Console.WriteLine("\nProduits groupés par catégorie:");
        foreach (var group in groupedProducts)
        {
            Console.WriteLine($"Catégorie: {group.Category} ({group.Count} produits, prix moyen: ${group.AveragePrice:F2})");
            foreach (var product in group.Products)
            {
                Console.WriteLine($"  - {product.Name} (${product.Price})");
            }
        }

        // Jointure avec Join
        List<Supplier> suppliers = new List<Supplier>
        {
            new Supplier { Id = 1, Name = "TechStore", Category = "Electronics" },
            new Supplier { Id = 2, Name = "FurniturePlus", Category = "Furniture" }
        };

        var joinResult = products
            .Join(
                suppliers,
                product => product.Category,
                supplier => supplier.Category,
                (product, supplier) => new
                {
                    ProductName = product.Name,
                    ProductPrice = product.Price,
                    SupplierName = supplier.Name
                }
            );

        Console.WriteLine("\nProduits avec leur fournisseur:");
        foreach (var item in joinResult)
        {
            Console.WriteLine($"- {item.ProductName} (${item.ProductPrice}) fourni par {item.SupplierName}");
        }

        // Enchaînement multiple de méthodes
        var discountedPrices = products
            .Select(p => new
            {
                p.Name,
                OriginalPrice = p.Price,
                DiscountRate = p.Price > 500 ? 0.10m : 0.05m
            })
            .Select(p => new
            {
                p.Name,
                p.OriginalPrice,
                DiscountRate = p.DiscountRate * 100,
                FinalPrice = p.OriginalPrice * (1 - p.DiscountRate)
            })
            .Where(p => p.FinalPrice > 100);

        Console.WriteLine("\nPrix après remise:");
        foreach (var item in discountedPrices)
        {
            Console.WriteLine($"- {item.Name}: ${item.OriginalPrice} - {item.DiscountRate}% = ${item.FinalPrice:F2}");
        }
    }
}

// Mêmes classes Product et Supplier que précédemment
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}

class Supplier
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
}
```


### Caractéristiques de la syntaxe de méthode

1. **Concision** : Souvent plus concise que la syntaxe de requête, en particulier pour les opérations simples.

2. **Enchaînement de méthodes** : Permet d'enchaîner facilement plusieurs opérations.

3. **Méthodes d'extension** : Fonctionne avec l'API des méthodes d'extension de C#.

4. **Flexibilité** : Certaines opérations LINQ ne sont disponibles qu'avec la syntaxe de méthode.

5. **Puissance** : Peut facilement être combinée avec des expressions lambda pour créer des filtres et des projecteurs personnalisés.

## 6.3.3. Opérateurs LINQ essentiels

LINQ propose un large éventail d'opérateurs pour manipuler des collections. Voici une exploration des opérateurs les plus courants et utiles.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        // Données d'exemple
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        List<Person> people = new List<Person>
        {
            new Person { Id = 1, Name = "Alice", Age = 25, Department = "IT" },
            new Person { Id = 2, Name = "Bob", Age = 30, Department = "HR" },
            new Person { Id = 3, Name = "Charlie", Age = 22, Department = "IT" },
            new Person { Id = 4, Name = "David", Age = 35, Department = "Finance" },
            new Person { Id = 5, Name = "Eva", Age = 28, Department = "HR" },
            new Person { Id = 6, Name = "Frank", Age = 32, Department = "Finance" },
            new Person { Id = 7, Name = "Grace", Age = 29, Department = "IT" }
        };

        Console.WriteLine("=== Opérateurs LINQ essentiels ===\n");

        // 1. Opérateurs de filtrage
        Console.WriteLine("-- Opérateurs de filtrage --");

        // Where - filtrer selon un prédicat
        var evenNumbers = numbers.Where(n => n % 2 == 0);
        Console.WriteLine($"Nombres pairs: {string.Join(", ", evenNumbers)}");

        // OfType - filtrer par type
        object[] mixedArray = { 1, "two", 3, "four", 5, "six" };
        var onlyStrings = mixedArray.OfType<string>();
        Console.WriteLine($"Seulement les chaînes: {string.Join(", ", onlyStrings)}");

        // 2. Opérateurs de projection
        Console.WriteLine("\n-- Opérateurs de projection --");

        // Select - transformer chaque élément
        var squares = numbers.Select(n => n * n);
        Console.WriteLine($"Carrés: {string.Join(", ", squares)}");

        // SelectMany - aplatir les collections imbriquées
        List<List<int>> nestedLists = new List<List<int>>
        {
            new List<int> { 1, 2, 3 },
            new List<int> { 4, 5, 6 },
            new List<int> { 7, 8, 9 }
        };
        var flattened = nestedLists.SelectMany(list => list);
        Console.WriteLine($"Liste aplatie: {string.Join(", ", flattened)}");

        // 3. Opérateurs de tri
        Console.WriteLine("\n-- Opérateurs de tri --");

        // OrderBy - tri croissant
        var sortedByAge = people.OrderBy(p => p.Age);
        Console.WriteLine("Personnes triées par âge (croissant):");
        foreach (var person in sortedByAge)
        {
            Console.WriteLine($"  {person.Name}, {person.Age} ans");
        }

        // OrderByDescending - tri décroissant
        var sortedByNameDesc = people.OrderByDescending(p => p.Name);
        Console.WriteLine("\nPersonnes triées par nom (décroissant):");
        foreach (var person in sortedByNameDesc)
        {
            Console.WriteLine($"  {person.Name}, {person.Age} ans");
        }

        // ThenBy et ThenByDescending - tri secondaire
        var sortedByDeptThenAge = people
            .OrderBy(p => p.Department)
            .ThenByDescending(p => p.Age);
        Console.WriteLine("\nPersonnes triées par département puis par âge (décroissant):");
        foreach (var person in sortedByDeptThenAge)
        {
            Console.WriteLine($"  {person.Department}: {person.Name}, {person.Age} ans");
        }

        // Reverse - inverser l'ordre
        var reversed = numbers.Reverse();
        Console.WriteLine($"\nNombres inversés: {string.Join(", ", reversed)}");

        // 4. Opérateurs d'ensemble
        Console.WriteLine("\n-- Opérateurs d'ensemble --");

        var set1 = new List<int> { 1, 2, 3, 4, 5 };
        var set2 = new List<int> { 4, 5, 6, 7, 8 };

        // Distinct - éliminer les doublons
        var duplicateNumbers = new List<int> { 1, 2, 2, 3, 3, 3, 4, 5, 5 };
        var distinctNumbers = duplicateNumbers.Distinct();
        Console.WriteLine($"Nombres sans doublons: {string.Join(", ", distinctNumbers)}");

        // Union - union de deux ensembles (sans doublons)
        var unionSet = set1.Union(set2);
        Console.WriteLine($"Union: {string.Join(", ", unionSet)}");

        // Intersect - intersection de deux ensembles
        var intersectSet = set1.Intersect(set2);
        Console.WriteLine($"Intersection: {string.Join(", ", intersectSet)}");

        // Except - différence d'ensembles
        var exceptSet = set1.Except(set2);
        Console.WriteLine($"Différence (set1 - set2): {string.Join(", ", exceptSet)}");

        // 5. Opérateurs de quantification
        Console.WriteLine("\n-- Opérateurs de quantification --");

        // Any - vérifie si au moins un élément satisfait une condition
        bool hasAnyEvenNumbers = numbers.Any(n => n % 2 == 0);
        Console.WriteLine($"Contient au moins un nombre pair: {hasAnyEvenNumbers}");

        // All - vérifie si tous les éléments satisfont une condition
        bool areAllPositive = numbers.All(n => n > 0);
        Console.WriteLine($"Tous les nombres sont positifs: {areAllPositive}");

        // Contains - vérifie si la collection contient un élément spécifique
        bool containsSeven = numbers.Contains(7);
        Console.WriteLine($"Contient le nombre 7: {containsSeven}");

        // 6. Opérateurs d'élément
        Console.WriteLine("\n-- Opérateurs d'élément --");

        // First, FirstOrDefault - premier élément (avec/sans valeur par défaut)
        var firstEven = numbers.First(n => n % 2 == 0);
        Console.WriteLine($"Premier nombre pair: {firstEven}");

        // Si aucun élément ne correspond, First lève une exception, mais FirstOrDefault retourne la valeur par défaut
        var firstNegative = numbers.FirstOrDefault(n => n < 0); // par défaut: 0 pour int
        Console.WriteLine($"Premier nombre négatif (ou défaut): {firstNegative}");

        // Last, LastOrDefault - dernier élément (avec/sans valeur par défaut)
        var lastEven = numbers.Last(n => n % 2 == 0);
        Console.WriteLine($"Dernier nombre pair: {lastEven}");

        // Single, SingleOrDefault - élément unique (lève une exception s'il y a plus d'un match)
        try
        {
            var exactlyOne = numbers.Single(n => n == 5);
            Console.WriteLine($"Nombre exactement égal à 5: {exactlyOne}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Exception: {ex.Message}");
        }

        // ElementAt, ElementAtOrDefault - élément à un index spécifique
        var thirdElement = numbers.ElementAt(2); // indexé à partir de 0
        Console.WriteLine($"Troisième élément: {thirdElement}");

        // 7. Opérateurs d'agrégation
        Console.WriteLine("\n-- Opérateurs d'agrégation --");

        // Count - nombre d'éléments
        int count = numbers.Count();
        int evenCount = numbers.Count(n => n % 2 == 0);
        Console.WriteLine($"Nombre total d'éléments: {count}");
        Console.WriteLine($"Nombre d'éléments pairs: {evenCount}");

        // Sum - somme des éléments
        int sum = numbers.Sum();
        double averageAge = people.Average(p => p.Age);
        Console.WriteLine($"Somme des nombres: {sum}");
        Console.WriteLine($"Âge moyen: {averageAge:F1} ans");

        // Min, Max - minimum et maximum
        int min = numbers.Min();
        int max = numbers.Max();
        Console.WriteLine($"Valeur minimale: {min}");
        Console.WriteLine($"Valeur maximale: {max}");

        // Aggregate - agrégation personnalisée
        string commaSeparated = numbers.Aggregate("", (result, n) => result + (result.Length > 0 ? ", " : "") + n.ToString());
        Console.WriteLine($"Chaîne concaténée: {commaSeparated}");

        // 8. Opérateurs de groupement
        Console.WriteLine("\n-- Opérateurs de groupement --");

        // GroupBy - grouper par une clé
        var peopleByDepartment = people.GroupBy(p => p.Department);

        Console.WriteLine("Personnes regroupées par département:");
        foreach (var group in peopleByDepartment)
        {
            Console.WriteLine($"  Département: {group.Key}");
            foreach (var person in group)
            {
                Console.WriteLine($"    - {person.Name}, {person.Age} ans");
            }
        }

        // 9. Opérateurs de jointure
        Console.WriteLine("\n-- Opérateurs de jointure --");

        List<Department> departments = new List<Department>
        {
            new Department { Id = 1, Name = "IT", Location = "Building A" },
            new Department { Id = 2, Name = "HR", Location = "Building B" },
            new Department { Id = 3, Name = "Finance", Location = "Building C" }
        };

        // Join - jointure interne
        var employeeDetails = people.Join(
            departments,
            person => person.Department,
            dept => dept.Name,
            (person, dept) => new
            {
                EmployeeName = person.Name,
                Department = dept.Name,
                Location = dept.Location
            }
        );

        Console.WriteLine("Détails des employés (jointure):");
        foreach (var detail in employeeDetails)
        {
            Console.WriteLine($"  {detail.EmployeeName} travaille dans {detail.Department} à {detail.Location}");
        }

        // GroupJoin - jointure avec groupement
        var departmentsWithEmployees = departments.GroupJoin(
            people,
            dept => dept.Name,
            person => person.Department,
            (dept, employees) => new
            {
                DepartmentName = dept.Name,
                Location = dept.Location,
                Employees = employees.ToList()
            }
        );

        Console.WriteLine("\nDépartements avec employés (jointure groupée):");
        foreach (var dept in departmentsWithEmployees)
        {
            Console.WriteLine($"  {dept.DepartmentName} ({dept.Location}), {dept.Employees.Count} employés:");
            foreach (var employee in dept.Employees)
            {
                Console.WriteLine($"    - {employee.Name}, {employee.Age} ans");
            }
        }

        // 10. Opérateurs de partitionnement
        Console.WriteLine("\n-- Opérateurs de partitionnement --");

        // Take - prendre n premiers éléments
        var firstThree = numbers.Take(3);
        Console.WriteLine($"Trois premiers éléments: {string.Join(", ", firstThree)}");

        // Skip - ignorer n premiers éléments
        var afterFirstFive = numbers.Skip(5);
        Console.WriteLine($"Éléments après les 5 premiers: {string.Join(", ", afterFirstFive)}");

        // TakeWhile, SkipWhile - prendre/ignorer tant qu'une condition est vraie
        var takeUntilBig = numbers.TakeWhile(n => n < 6);
        Console.WriteLine($"Éléments jusqu'à ce que n >= 6: {string.Join(", ", takeUntilBig)}");

        // Chunk (disponible dans .NET 6+) - diviser en morceaux de taille fixe
        if (Environment.Version.Major >= 6) // Vérifier si .NET 6+ est utilisé
        {
            // Les chunks ne sont pas disponibles dans .NET Framework 4.7.2
            #if NET6_0_OR_GREATER
            var chunks = numbers.Chunk(3);
            Console.WriteLine("\nNombres divisés en morceaux de 3:");
            foreach (var chunk in chunks)
            {
                Console.WriteLine($"  {string.Join(", ", chunk)}");
            }
            #else
            Console.WriteLine("\nLa méthode Chunk n'est pas disponible dans cette version de .NET.");
            #endif
        }
        else
        {
            Console.WriteLine("\nLa méthode Chunk n'est pas disponible dans cette version de .NET.");
        }
    }
}

class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public string Department { get; set; }
}

class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Location { get; set; }
}
```


### Catégories d'opérateurs LINQ

1. **Filtrage** : `Where`, `OfType`
2. **Projection** : `Select`, `SelectMany`
3. **Tri** : `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `Reverse`
4. **Ensemble** : `Distinct`, `Union`, `Intersect`, `Except`
5. **Quantification** : `Any`, `All`, `Contains`
6. **Élément** : `First`, `FirstOrDefault`, `Last`, `LastOrDefault`, `Single`, `SingleOrDefault`, `ElementAt`, `ElementAtOrDefault`
7. **Agrégation** : `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`
8. **Groupement** : `GroupBy`
9. **Jointure** : `Join`, `GroupJoin`
10. **Partitionnement** : `Take`, `Skip`, `TakeWhile`, `SkipWhile`, `Chunk` (disponible dans .NET 6+)

## 6.3.4. LINQ to Objects

LINQ to Objects fait référence à l'utilisation de requêtes LINQ directement sur des collections en mémoire qui implémentent `IEnumerable<T>`.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== LINQ to Objects ===\n");

        // Travail avec différents types de collections

        // 1. Tableau
        int[] numbersArray = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        var evenFromArray = numbersArray.Where(n => n % 2 == 0);
        Console.WriteLine($"Nombres pairs du tableau: {string.Join(", ", evenFromArray)}");

        // 2. List<T>
        List<string> fruitsList = new List<string> { "Apple", "Banana", "Orange", "Grape", "Pineapple" };
        var longFruits = fruitsList.Where(f => f.Length > 5).OrderBy(f => f);
        Console.WriteLine($"\nFruits avec noms longs: {string.Join(", ", longFruits)}");

        // 3. Dictionary<TKey, TValue>
        Dictionary<string, int> ages = new Dictionary<string, int>
        {
            { "Alice", 25 },
            { "Bob", 30 },
            { "Charlie", 22 },
            { "David", 35 }
        };

        // Travail avec les clés
        var namesStartingWithA = ages.Keys.Where(name => name.StartsWith("A"));
        Console.WriteLine($"\nNoms commençant par A: {string.Join(", ", namesStartingWithA)}");

        // Travail avec les paires clé-valeur
        var over25 = ages.Where(pair => pair.Value > 25)
                          .Select(pair => $"{pair.Key} ({pair.Value} ans)");
        Console.WriteLine($"\nPersonnes de plus de 25 ans: {string.Join(", ", over25)}");

        // 4. Utilisation avec des classes complexes
        List<Book> books = new List<Book>
        {
            new Book { Title = "The Great Gatsby", Author = "F. Scott Fitzgerald", Genre = "Novel", Pages = 180, Year = 1925 },
            new Book { Title = "1984", Author = "George Orwell", Genre = "Dystopian", Pages = 328, Year = 1949 },
            new Book { Title = "To Kill a Mockingbird", Author = "Harper Lee", Genre = "Novel", Pages = 281, Year = 1960 },
            new Book { Title = "The Hobbit", Author = "J.R.R. Tolkien", Genre = "Fantasy", Pages = 310, Year = 1937 },
            new Book { Title = "Brave New World", Author = "Aldous Huxley", Genre = "Dystopian", Pages = 288, Year = 1932 }
        };

        // Requêtes complexes sur une collection d'objets
        Console.WriteLine("\nLivres par genre:");
        var booksByGenre = books.GroupBy(b => b.Genre);
        foreach (var group in booksByGenre)
        {
            Console.WriteLine($"  Genre: {group.Key}");
            foreach (var book in group)
            {
                Console.WriteLine($"    - {book.Title} par {book.Author} ({book.Year})");
            }
        }

        // Filtrage et projection multiples
        var shortModernBooks = books
            .Where(b => b.Year > 1930 && b.Pages < 300)
            .OrderBy(b => b.Year)
            .Select(b => new { b.Title, b.Author, b.Year, PageCount = b.Pages });

        Console.WriteLine("\nLivres modernes courts (après 1930, moins de 300 pages):");
        foreach (var book in shortModernBooks)
        {
            Console.WriteLine($"  - {book.Title} par {book.Author} ({book.Year}), {book.PageCount} pages");
        }

        // 5. Travail avec des collections imbriquées
        List<Student> students = new List<Student>
        {
            new Student
            {
                Name = "Alice",
                Courses = new List<Course>
                {
                    new Course { Name = "Math", Grade = 85 },
                    new Course { Name = "Science", Grade = 92 },
                    new Course { Name = "History", Grade = 78 }
                }
            },
            new Student
            {
                Name = "Bob",
                Courses = new List<Course>
                {
                    new Course { Name = "Math", Grade = 70 },
                    new Course { Name = "Science", Grade = 82 },
                    new Course { Name = "English", Grade = 88 }
                }
            },
            new Student
            {
                Name = "Charlie",
                Courses = new List<Course>
                {
                    new Course { Name = "Math", Grade = 90 },
                    new Course { Name = "English", Grade = 85 },
                    new Course { Name = "History", Grade = 92 }
                }
            }
        };

        // Aplatir des collections imbriquées avec SelectMany
        var allCourses = students.SelectMany(s => s.Courses);
        Console.WriteLine($"\nNombre total de cours suivis: {allCourses.Count()}");

        // Calculer la moyenne des notes par matière
        var gradesBySubject = students
            .SelectMany(s => s.Courses)
            .GroupBy(c => c.Name)
            .Select(g => new { Subject = g.Key, AverageGrade = g.Average(c => c.Grade) });

        Console.WriteLine("\nMoyenne des notes par matière:");
        foreach (var grade in gradesBySubject)
        {
            Console.WriteLine($"  - {grade.Subject}: {grade.AverageGrade:F1}");
        }

        // Obtenir les meilleurs étudiants par matière
        var topStudentsBySubject = students
            .SelectMany(s => s.Courses.Select(c => new { StudentName = s.Name, c.Name, c.Grade }))
            .GroupBy(item => item.Name)
            .Select(g => new
            {
                Subject = g.Key,
                TopStudent = g.OrderByDescending(item => item.Grade).First()
            });

        Console.WriteLine("\nMeilleurs étudiants par matière:");
        foreach (var subject in topStudentsBySubject)
        {
            Console.WriteLine($"  - {subject.Subject}: {subject.TopStudent.StudentName} ({subject.TopStudent.Grade})");
        }

        // 6. Requêtes avec des cas d'utilisation pratiques

        // Statistiques sur les livres
        var bookStats = new
        {
            TotalBooks = books.Count,
            AveragePages = books.Average(b => b.Pages),
            OldestBook = books.OrderBy(b => b.Year).First(),
            NewestBook = books.OrderByDescending(b => b.Year).First(),
            GenreCount = books.GroupBy(b => b.Genre).Count(),
            PagesByGenre = books.GroupBy(b => b.Genre)
                                .Select(g => new
                                {
                                    Genre = g.Key,
                                    AveragePages = g.Average(b => b.Pages)
                                })
        };

        Console.WriteLine("\nStatistiques sur les livres:");
        Console.WriteLine($"  - Nombre total de livres: {bookStats.TotalBooks}");
        Console.WriteLine($"  - Nombre moyen de pages: {bookStats.AveragePages:F1}");
        Console.WriteLine($"  - Livre le plus ancien: {bookStats.OldestBook.Title} ({bookStats.OldestBook.Year})");
        Console.WriteLine($"  - Livre le plus récent: {bookStats.NewestBook.Title} ({bookStats.NewestBook.Year})");
        Console.WriteLine($"  - Nombre de genres: {bookStats.GenreCount}");

        Console.WriteLine("\n  Pages moyennes par genre:");
        foreach (var genre in bookStats.PagesByGenre)
        {
            Console.WriteLine($"    - {genre.Genre}: {genre.AveragePages:F1} pages");
        }

        // 7. Génération de rapports
        Console.WriteLine("\nRapport des étudiants:");
        var studentReport = students.Select(s => new
        {
            s.Name,
            CourseCount = s.Courses.Count,
            AverageGrade = s.Courses.Average(c => c.Grade),
            HighestGrade = s.Courses.Max(c => c.Grade),
            LowestGrade = s.Courses.Min(c => c.Grade),
            Courses = string.Join(", ", s.Courses.Select(c => c.Name))
        });

        foreach (var report in studentReport)
        {
            Console.WriteLine($"  {report.Name}:");
            Console.WriteLine($"    - Moyenne: {report.AverageGrade:F1}");
            Console.WriteLine($"    - Note la plus haute: {report.HighestGrade}");
            Console.WriteLine($"    - Note la plus basse: {report.LowestGrade}");
            Console.WriteLine($"    - Cours suivis ({report.CourseCount}): {report.Courses}");
        }
    }
}

class Book
{
    public string Title { get; set; }
    public string Author { get; set; }
    public string Genre { get; set; }
    public int Pages { get; set; }
    public int Year { get; set; }
}

class Student
{
    public string Name { get; set; }
    public List<Course> Courses { get; set; }
}

class Course
{
    public string Name { get; set; }
    public int Grade { get; set; }
}
```


### Avantages de LINQ to Objects

1. **Manipulation uniforme** : Permet de manipuler différents types de collections avec la même syntaxe.

2. **Code déclaratif** : Permet d'exprimer ce que vous voulez accomplir plutôt que comment le faire.

3. **Réduction du code** : Réduit considérablement la quantité de code nécessaire pour manipuler des collections.

4. **Lisibilité** : Améliore la lisibilité du code en évitant les boucles imbriquées et la logique complexe.

5. **Type Safety** : Fournit une vérification de type au moment de la compilation.

6. **Performance** : Optimisé pour de nombreuses opérations courantes sur les collections.

7. **Composition** : Permet de combiner facilement plusieurs opérations en chaîne.

## 6.3.5. Requêtes différées et requêtes immédiates

LINQ utilise le concept d'exécution différée (lazy evaluation) pour de nombreuses opérations, mais certaines opérations déclenchent une exécution immédiate. Comprendre cette distinction est crucial pour écrire du code LINQ efficace.

```textmate
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Requêtes différées vs immédiates ===\n");

        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

        Console.WriteLine("Liste initiale: " + string.Join(", ", numbers));

        // 1. Requête différée (lazy evaluation)
        var evenNumbers = numbers.Where(n => n % 2 == 0);

        // À ce stade, aucun filtrage n'a été effectué
        Console.WriteLine("\n--- Démonstration de l'évaluation différée ---");
        Console.WriteLine("Requête définie mais pas encore exécutée");

        // Modification de la collection source
        numbers.Add(6);
        numbers.Add(7);
        numbers.Add(8);

        Console.WriteLine("Liste modifiée: " + string.Join(", ", numbers));

        // L'énumération déclenche l'exécution de la requête
        Console.WriteLine("Nombres pairs (après modification): " + string.Join(", ", evenNumbers));
        // Remarque: evenNumbers inclut maintenant 6 et 8, qui ont été ajoutés après la définition de la requête

        // 2. Matérialisation de requêtes (exécution immédiate)
        Console.WriteLine("\n--- Démonstration de l'évaluation immédiate ---");

        // ToList() force l'exécution immédiate
        var evenNumbersList = numbers.Where(n => n % 2 == 0).ToList();
        Console.WriteLine("Nombres pairs (matérialisés en liste): " + string.Join(", ", evenNumbersList));

        // Modification de la source après matérialisation
        numbers.Add(10);

        Console.WriteLine("Liste modifiée: " + string.Join(", ", numbers));
        Console.WriteLine("Nombres pairs (toujours la même liste): " + string.Join(", ", evenNumbersList));
        // evenNumbersList ne change pas car il a été matérialisé avant l'ajout de 10

        // 3. Opérateurs à exécution immédiate
        Console.WriteLine("\n--- Opérateurs à exécution immédiate ---");

        int count = numbers.Count();
        int sum = numbers.Sum();
        double average = numbers.Average();
        int firstEven = numbers.First(n => n % 2 == 0);

        Console.WriteLine($"Count: {count}");
        Console.WriteLine($"Sum: {sum}");
        Console.WriteLine($"Average: {average}");
        Console.WriteLine($"First even: {firstEven}");

        // 4. Réutilisation de requêtes différées
        Console.WriteLine("\n--- Réutilisation de requêtes différées ---");

        var query = numbers.Where(n => n > 5);

        Console.WriteLine("Nombres > 5 (première exécution): " + string.Join(", ", query));

        // Modifier la source
        numbers.Add(12);

        Console.WriteLine("Nombres > 5 (deuxième exécution): " + string.Join(", ", query));
        // La requête est réévaluée et inclut maintenant 12

        // 5. Multiple énumération d'une requête différée
        Console.WriteLine("\n--- Problème d'énumération multiple ---");

        // La création d'une méthode avec effets secondaires pour la démonstration
        var expensiveQuery = numbers.Where(n => {
            Console.WriteLine($"Évaluant: {n}");
            return n % 2 == 0;
        });

        Console.WriteLine("Première énumération:");
        foreach (var item in expensiveQuery)
        {
            Console.WriteLine($"Trouvé: {item}");
        }

        Console.WriteLine("\nDeuxième énumération:");
        foreach (var item in expensiveQuery)
        {
            Console.WriteLine($"Trouvé: {item}");
        }
        // Notez que le prédicat est évalué deux fois pour chaque élément

        Console.WriteLine("\nSolution - matérialiser la requête:");
        var materialized = expensiveQuery.ToList();

        Console.WriteLine("\nPremière énumération de la liste:");
        foreach (var item in materialized)
        {
            Console.WriteLine($"Trouvé: {item}");
        }

        Console.WriteLine("\nDeuxième énumération de la liste:");
        foreach (var item in materialized)
        {
            Console.WriteLine($"Trouvé: {item}");
        }
        // La méthode avec effet secondaire n'est plus appelée

        // 6. Performance et optimisation
        Console.WriteLine("\n--- Performance et optimisation ---");

        // Pour de grandes collections, les requêtes chaînées peuvent être plus efficaces
        // si elles réduisent la taille de la collection intermédiaire

        // Moins efficace - traite tous les éléments dans Where avant Take
        var inefficientQuery = numbers
            .Where(n => {
                Console.WriteLine($"Filtrage inefficace: {n}");
                return n > 3;
            })
            .Take(2);

        Console.WriteLine("Résultat de la requête inefficace:");
        foreach (var item in inefficientQuery)
        {
            Console.WriteLine($"Obtenu: {item}");
        }

        // Plus efficace - Take est appliqué au fur et à mesure
        var efficientQuery = numbers
            .Take(2)
            .Where(n => {
                Console.WriteLine($"Filtrage efficace: {n}");
                return n > 0; // Condition toujours vraie pour la démo
            });

        Console.WriteLine("\nRésultat de la requête efficace:");
        foreach (var item in efficientQuery)
        {
            Console.WriteLine($"Obtenu: {item}");
        }
    }
}
```


### Requêtes différées (Lazy Evaluation)

La plupart des opérateurs LINQ retournent un objet qui stocke la requête mais ne l'exécute pas immédiatement. L'exécution est déclenchée seulement lorsque vous:

1. **Itérez sur les résultats** (par exemple, avec une boucle `foreach` ou `ToList()`)
2. **Invoquez un opérateur à exécution immédiate** (comme `Count()`, `Sum()`, etc.)

Avantages de l'évaluation différée:
- Meilleure performance dans certains cas
- Possibilité de construire des requêtes de manière incrémentale
- Les requêtes reflètent les modifications de la source

Inconvénients:
- Chaque énumération exécute la requête à nouveau
- Effets secondaires potentiels si le prédicat ou le sélecteur a des effets secondaires
- Des résultats différents si la source est modifiée entre les énumérations

### Requêtes immédiates (Immediate Execution)

Les requêtes sont exécutées immédiatement lorsque:

1. **Vous appelez des méthodes de matérialisation** comme:
   - `ToList()`, `ToArray()`, `ToDictionary()`, `ToHashSet()`
   - `ToLookup()`

2. **Vous utilisez des opérateurs d'agrégation** comme:
   - `Count()`, `Sum()`, `Average()`, `Min()`, `Max()`
   - `Aggregate()`

3. **Vous utilisez des opérateurs d'élément** comme:
   - `First()`, `FirstOrDefault()`, `Last()`, `LastOrDefault()`
   - `Single()`, `SingleOrDefault()`
   - `ElementAt()`, `ElementAtOrDefault()`

Avantages de l'exécution immédiate:
- Exécution une seule fois
- Capture l'état actuel de la source
- Évite les calculs répétés

Inconvénients:
- Utilisation de mémoire supplémentaire pour stocker les résultats
- Ne reflète pas les modifications ultérieures de la source

### Quand matérialiser une requête?

Il est judicieux de matérialiser une requête (en utilisant `ToList()`, `ToArray()`, etc.) lorsque:

1. Vous allez énumérer les résultats plusieurs fois
2. La source peut changer et vous voulez un instantané cohérent
3. La requête est coûteuse à évaluer (comme pour des opérations de base de données)
4. Vous souhaitez déconnecter les résultats de la source

## 6.3.6. Projection et transformation de données
La projection est l'une des opérations les plus puissantes de LINQ, permettant de transformer des données d'un format à un autre.
``` csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Projection et transformation de données ===\n");

        // Collection de données pour les exemples
        List<Product> products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Electronics", Tags = new[] { "computer", "portable", "work" } },
            new Product { Id = 2, Name = "Smartphone", Price = 699.99m, Category = "Electronics", Tags = new[] { "phone", "mobile", "communication" } },
            new Product { Id = 3, Name = "Desk Chair", Price = 179.99m, Category = "Furniture", Tags = new[] { "office", "comfort", "ergonomic" } },
            new Product { Id = 4, Name = "Coffee Maker", Price = 89.99m, Category = "Appliances", Tags = new[] { "kitchen", "beverage", "morning" } },
            new Product { Id = 5, Name = "Headphones", Price = 149.99m, Category = "Electronics", Tags = new[] { "audio", "music", "wireless" } }
        };

        List<Order> orders = new List<Order>
        {
            new Order
            {
                Id = 1,
                CustomerName = "Alice",
                OrderDate = new DateTime(2023, 5, 15),
                Items = new List<OrderItem>
                {
                    new OrderItem { ProductId = 1, Quantity = 1, UnitPrice = 999.99m },
                    new OrderItem { ProductId = 5, Quantity = 2, UnitPrice = 149.99m }
                }
            },
            new Order
            {
                Id = 2,
                CustomerName = "Bob",
                OrderDate = new DateTime(2023, 6, 2),
                Items = new List<OrderItem>
                {
                    new OrderItem { ProductId = 2, Quantity = 1, UnitPrice = 699.99m },
                    new OrderItem { ProductId = 4, Quantity = 1, UnitPrice = 89.99m }
                }
            },
            new Order
            {
                Id = 3,
                CustomerName = "Charlie",
                OrderDate = new DateTime(2023, 6, 10),
                Items = new List<OrderItem>
                {
                    new OrderItem { ProductId = 3, Quantity = 2, UnitPrice = 179.99m }
                }
            }
        };

        // 1. Projection simple avec Select
        Console.WriteLine("--- Projection simple ---");

        // Projection vers un type anonyme
        var productSummaries = products.Select(p => new
        {
            ProductName = p.Name,
            ProductPrice = p.Price
        });

        foreach (var summary in productSummaries)
        {
            Console.WriteLine($"{summary.ProductName}: ${summary.ProductPrice}");
        }

        // 2. Transformation avec des opérations
        Console.WriteLine("\n--- Transformation avec opérations ---");

        var discountedProducts = products.Select(p => new
        {
            p.Name,
            OriginalPrice = p.Price,
            DiscountRate = p.Price > 500 ? 0.10m : 0.05m,
            SalePrice = p.Price * (p.Price > 500 ? 0.90m : 0.95m)
        });

        foreach (var product in discountedProducts)
        {
            Console.WriteLine($"{product.Name}: ${product.OriginalPrice} - {product.DiscountRate * 100}% = ${product.SalePrice:F2}");
        }

        // 3. Projection vers un type nommé
        Console.WriteLine("\n--- Projection vers un type nommé ---");

        var productDTOs = products.Select(p => new ProductDTO
        {
            Id = p.Id,
            DisplayName = $"{p.Name} ({p.Category})",
            Price = p.Price
        });

        foreach (var dto in productDTOs)
        {
            Console.WriteLine($"{dto.Id}. {dto.DisplayName}: ${dto.Price}");
        }

        // 4. Aplatissement de collections imbriquées avec SelectMany
        Console.WriteLine("\n--- Aplatissement avec SelectMany ---");

        // Obtenir tous les tags de tous les produits
        var allTags = products.SelectMany(p => p.Tags);
        Console.WriteLine($"Tous les tags: {string.Join(", ", allTags)}");

        // Tags uniques
        var uniqueTags = products.SelectMany(p => p.Tags).Distinct();
        Console.WriteLine($"Tags uniques: {string.Join(", ", uniqueTags)}");

        // 5. Jointure de données et projection
        Console.WriteLine("\n--- Jointure et projection ---");

        var orderDetails = orders.SelectMany(o => o.Items.Select(item => new
        {
            OrderId = o.Id,
            Customer = o.CustomerName,
            OrderDate = o.OrderDate,
            ProductId = item.ProductId,
            Quantity = item.Quantity,
            ProductName = products.First(p => p.Id == item.ProductId).Name,
            LineTotal = item.Quantity * item.UnitPrice
        }));

        foreach (var detail in orderDetails)
        {
            Console.WriteLine($"Commande #{detail.OrderId} - {detail.Customer} ({detail.OrderDate:d}): " +
                             $"{detail.Quantity}x {detail.ProductName} = ${detail.LineTotal}");
        }

        // 6. Projection conditionnelle
        Console.WriteLine("\n--- Projection conditionnelle ---");

        var categorizedProducts = products.Select(p => new
        {
            p.Name,
            p.Price,
            PriceCategory = p.Price < 100 ? "Budget" : (p.Price < 500 ? "Mid-Range" : "Premium"),
            SaleStatus = p.Price > 500 ? "On Sale!" : "Regular Price"
        });

        foreach (var product in categorizedProducts)
        {
            Console.WriteLine($"{product.Name} (${product.Price}): {product.PriceCategory}, {product.SaleStatus}");
        }

        // 7. Projection avec indexation
        Console.WriteLine("\n--- Projection avec indexation ---");

        var rankedProducts = products.Select((p, index) => new
        {
            Rank = index + 1,
            p.Name,
            p.Price
        });

        foreach (var product in rankedProducts)
        {
            Console.WriteLine($"{product.Rank}. {product.Name}: ${product.Price}");
        }

        // 8. Projection hiérarchique
        Console.WriteLine("\n--- Projection hiérarchique ---");

        var orderSummaries = orders.Select(o => new
        {
            OrderId = o.Id,
            Customer = o.CustomerName,
            OrderDate = o.OrderDate,
            TotalAmount = o.Items.Sum(i => i.Quantity * i.UnitPrice),
            ItemCount = o.Items.Count,
            Items = o.Items.Select(i => new
            {
                ProductName = products.First(p => p.Id == i.ProductId).Name,
                i.Quantity,
                LineTotal = i.Quantity * i.UnitPrice
            }).ToList()
        });

        foreach (var order in orderSummaries)
        {
            Console.WriteLine($"Commande #{order.OrderId} - {order.Customer} ({order.OrderDate:d})");
            Console.WriteLine($"  Total: ${order.TotalAmount:F2}, {order.ItemCount} article(s)");

            foreach (var item in order.Items)
            {
                Console.WriteLine($"    - {item.Quantity}x {item.ProductName}: ${item.LineTotal:F2}");
            }
            Console.WriteLine();
        }

        // 9. Projection avec agrégations
        Console.WriteLine("--- Projection avec agrégations ---");

        var categoryStats = products.GroupBy(p => p.Category)
            .Select(g => new
            {
                Category = g.Key,
                ProductCount = g.Count(),
                AveragePrice = g.Average(p => p.Price),
                TotalValue = g.Sum(p => p.Price),
                PriceRange = new
                {
                    Min = g.Min(p => p.Price),
                    Max = g.Max(p => p.Price)
                },
                Products = g.Select(p => p.Name).ToList()
            });

        foreach (var category in categoryStats)
        {
            Console.WriteLine($"Catégorie: {category.Category}");
            Console.WriteLine($"  Nombre de produits: {category.ProductCount}");
            Console.WriteLine($"  Prix moyen: ${category.AveragePrice:F2}");
            Console.WriteLine($"  Valeur totale: ${category.TotalValue:F2}");
            Console.WriteLine($"  Fourchette de prix: ${category.PriceRange.Min} - ${category.PriceRange.Max}");
            Console.WriteLine($"  Produits: {string.Join(", ", category.Products)}");
            Console.WriteLine();
        }
    }
}

class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    public string[] Tags { get; set; }
}

class Order
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public DateTime OrderDate { get; set; }
    public List<OrderItem> Items { get; set; }
}

class OrderItem
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

class ProductDTO
{
    public int Id { get; set; }
    public string DisplayName { get; set; }
    public decimal Price { get; set; }
}
```
### Fonctionnalités clés de projection
1. **Select** : Transforme chaque élément d'une séquence en un nouvel élément.
2. **SelectMany** : Aplatit des collections imbriquées, en transformant chaque élément d'une séquence en zéro, un ou plusieurs éléments.
3. **Types anonymes** : Permet de créer des types à la volée pour des projections sans définir une classe explicite.
4. **Objets initialisés** : Permet de projeter vers des types explicites en utilisant des initialiseurs d'objet.
5. **Projections conditionnelles** : Utilisation d'opérateurs conditionnels pour créer différentes projections selon des conditions.
6. **Projections avec index** : Accès à l'index de chaque élément dans la séquence pendant la projection.
7. **Projections hiérarchiques** : Création de structures de données imbriquées à partir de données relationnelles.
8. **Projections avec agrégation** : Combine la projection avec des opérations d'agrégation pour créer des résumés de données.

### Cas d'utilisation courants
- **Mapping entre modèles** : Transformation de modèles de domaine en DTOs (Data Transfer Objects)
- **Transformation de format** : Conversion de données pour l'affichage, le stockage ou la transmission
- **Filtrage de propriétés** : Sélection d'un sous-ensemble de propriétés pour limiter les données transférées
- **Calcul de valeurs dérivées** : Ajout de propriétés calculées à partir des données existantes
- **Aplatissement de structures hiérarchiques** : Transformation de données imbriquées en structures plates
- **Construction de hiérarchies** : Création de structures hiérarchiques à partir de données plates

## 6.3.7. Agrégation et groupement
Les opérations d'agrégation et de groupement permettent d'analyser et de résumer des données de manière puissante.
``` csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Agrégation et groupement ===\n");

        // Données pour les exemples
        List<Sale> sales = new List<Sale>
        {
            new Sale { Id = 1, ProductName = "Laptop", Category = "Electronics", Price = 999.99m, Quantity = 2, Date = new DateTime(2023, 1, 15), Region = "East" },
            new Sale { Id = 2, ProductName = "Mouse", Category = "Electronics", Price = 25.50m, Quantity = 10, Date = new DateTime(2023, 2, 3), Region = "West" },
            new Sale { Id = 3, ProductName = "Desk Chair", Category = "Furniture", Price = 179.99m, Quantity = 3, Date = new DateTime(2023, 2, 17), Region = "North" },
            new Sale { Id = 4, ProductName = "Coffee Maker", Category = "Appliances", Price = 89.99m, Quantity = 5, Date = new DateTime(2023, 3, 2), Region = "South" },
            new Sale { Id = 5, ProductName = "Headphones", Category = "Electronics", Price = 149.99m, Quantity = 8, Date = new DateTime(2023, 3, 10), Region = "East" },
            new Sale { Id = 6, ProductName = "Laptop", Category = "Electronics", Price = 1099.99m, Quantity = 1, Date = new DateTime(2023, 3, 15), Region = "West" },
            new Sale { Id = 7, ProductName = "Office Desk", Category = "Furniture", Price = 299.99m, Quantity = 2, Date = new DateTime(2023, 4, 5), Region = "North" },
            new Sale { Id = 8, ProductName = "Blender", Category = "Appliances", Price = 79.99m, Quantity = 4, Date = new DateTime(2023, 4, 12), Region = "South" },
            new Sale { Id = 9, ProductName = "Smartphone", Category = "Electronics", Price = 699.99m, Quantity = 3, Date = new DateTime(2023, 5, 7), Region = "East" },
            new Sale { Id = 10, ProductName = "Dining Table", Category = "Furniture", Price = 399.99m, Quantity = 1, Date = new DateTime(2023, 5, 20), Region = "West" },
            new Sale { Id = 11, ProductName = "Microwave", Category = "Appliances", Price = 149.99m, Quantity = 2, Date = new DateTime(2023, 6, 8), Region = "North" },
            new Sale { Id = 12, ProductName = "Headphones", Category = "Electronics", Price = 149.99m, Quantity = 5, Date = new DateTime(2023, 6, 15), Region = "South" }
        };

        // 1. Opérations d'agrégation de base
        Console.WriteLine("--- Opérations d'agrégation de base ---");

        int totalSales = sales.Count();
        decimal totalRevenue = sales.Sum(s => s.Price * s.Quantity);
        decimal averageSalePrice = sales.Average(s => s.Price);
        decimal highestPrice = sales.Max(s => s.Price);
        decimal lowestPrice = sales.Min(s => s.Price);
        int totalItemsSold = sales.Sum(s => s.Quantity);

        Console.WriteLine($"Nombre total de ventes: {totalSales}");
        Console.WriteLine($"Revenu total: ${totalRevenue:F2}");
        Console.WriteLine($"Prix de vente moyen: ${averageSalePrice:F2}");
        Console.WriteLine($"Prix le plus élevé: ${highestPrice:F2}");
        Console.WriteLine($"Prix le plus bas: ${lowestPrice:F2}");
        Console.WriteLine($"Nombre total d'articles vendus: {totalItemsSold}");

        // 2. Agrégation conditionnelle
        Console.WriteLine("\n--- Agrégation conditionnelle ---");

        decimal electronicsRevenue = sales
            .Where(s => s.Category == "Electronics")
            .Sum(s => s.Price * s.Quantity);

        double percentageElectronics = (double)sales
            .Count(s => s.Category == "Electronics") / totalSales * 100;

        Console.WriteLine($"Revenu des produits électroniques: ${electronicsRevenue:F2}");
        Console.WriteLine($"Pourcentage de ventes d'électroniques: {percentageElectronics:F1}%");

        // 3. Agrégation composite
        Console.WriteLine("\n--- Agrégation composite ---");

        var salesSummary = new
        {
            TotalSales = totalSales,
            TotalRevenue = totalRevenue,
            AverageOrderValue = totalRevenue / totalSales,
            AverageQuantityPerOrder = (double)totalItemsSold / totalSales,
            RevenuePerItem = totalRevenue / totalItemsSold
        };

        Console.WriteLine($"Résumé des ventes:");
        Console.WriteLine($"  Nombre total de ventes: {salesSummary.TotalSales}");
        Console.WriteLine($"  Revenu total: ${salesSummary.TotalRevenue:F2}");
        Console.WriteLine($"  Valeur moyenne des commandes: ${salesSummary.AverageOrderValue:F2}");
        Console.WriteLine($"  Quantité moyenne par commande: {salesSummary.AverageQuantityPerOrder:F1}");
        Console.WriteLine($"  Revenu par article: ${salesSummary.RevenuePerItem:F2}");

        // 4. Groupement simple
        Console.WriteLine("\n--- Groupement simple ---");

        var salesByCategory = sales.GroupBy(s => s.Category);

        foreach (var group in salesByCategory)
        {
            Console.WriteLine($"Catégorie: {group.Key}");
            Console.WriteLine($"  Nombre de ventes: {group.Count()}");
            Console.WriteLine($"  Total vendu: {group.Sum(s => s.Quantity)} articles");
            Console.WriteLine($"  Revenu: ${group.Sum(s => s.Price * s.Quantity):F2}");
            Console.WriteLine();
        }

        // 5. Groupement avec projection
        Console.WriteLine("--- Groupement avec projection ---");

        var categoryStats = sales
            .GroupBy(s => s.Category)
            .Select(g => new
            {
                Category = g.Key,
                SalesCount = g.Count(),
                TotalRevenue = g.Sum(s => s.Price * s.Quantity),
                AveragePrice = g.Average(s => s.Price),
                TotalQuantity = g.Sum(s => s.Quantity),
                Products = g.Select(s => s.ProductName).Distinct().Count()
            })
            .OrderByDescending(x => x.TotalRevenue);

        foreach (var stat in categoryStats)
        {
            Console.WriteLine($"Catégorie: {stat.Category}");
            Console.WriteLine($"  Ventes: {stat.SalesCount}");
            Console.WriteLine($"  Revenu: ${stat.TotalRevenue:F2}");
            Console.WriteLine($"  Prix moyen: ${stat.AveragePrice:F2}");
            Console.WriteLine($"  Quantité totale: {stat.TotalQuantity}");
            Console.WriteLine($"  Produits distincts: {stat.Products}");
            Console.WriteLine();
        }

        // 6. Groupement multiple (par catégorie et région)
        Console.WriteLine("--- Groupement multiple ---");

        var salesByCategoryAndRegion = sales
            .GroupBy(s => new { s.Category, s.Region })
            .Select(g => new
            {
                g.Key.Category,
                g.Key.Region,
                SalesCount = g.Count(),
                TotalRevenue = g.Sum(s => s.Price * s.Quantity)
            })
            .OrderBy(x => x.Category)
            .ThenBy(x => x.Region);

        foreach (var group in salesByCategoryAndRegion)
        {
            Console.WriteLine($"{group.Category} - {group.Region}: " +
                             $"{group.SalesCount} ventes, ${group.TotalRevenue:F2}");
        }

        // 7. Groupement temporel (par mois)
        Console.WriteLine("\n--- Groupement temporel ---");

        var salesByMonth = sales
            .GroupBy(s => new { s.Date.Year, s.Date.Month })
            .Select(g => new
            {
                Year = g.Key.Year,
                Month = g.Key.Month,
                MonthName = System.Globalization.CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(g.Key.Month),
                SalesCount = g.Count(),
                TotalRevenue = g.Sum(s => s.Price * s.Quantity)
            })
            .OrderBy(x => x.Year)
            .ThenBy(x => x.Month);

        foreach (var month in salesByMonth)
        {
            Console.WriteLine($"{month.MonthName} {month.Year}: " +
                             $"{month.SalesCount} ventes, ${month.TotalRevenue:F2}");
        }

        // 8. Groupement imbriqué
        Console.WriteLine("\n--- Groupement imbriqué ---");

        var nestedGroups = sales
            .GroupBy(s => s.Category)
            .Select(categoryGroup => new
            {
                Category = categoryGroup.Key,
                TotalRevenue = categoryGroup.Sum(s => s.Price * s.Quantity),
                RegionGroups = categoryGroup
                    .GroupBy(s => s.Region)
                    .Select(regionGroup => new
                    {
                        Region = regionGroup.Key,
                        SalesCount = regionGroup.Count(),
                        Revenue = regionGroup.Sum(s => s.Price * s.Quantity),
                        Products = regionGroup.Select(s => s.ProductName).Distinct().ToList()
                    })
                    .OrderByDescending(x => x.Revenue)
                    .ToList()
            })
            .OrderByDescending(x => x.TotalRevenue);

        foreach (var category in nestedGroups)
        {
            Console.WriteLine($"Catégorie: {category.Category} (${category.TotalRevenue:F2})");

            foreach (var region in category.RegionGroups)
            {
                Console.WriteLine($"  Région: {region.Region}");
                Console.WriteLine($"    Ventes: {region.SalesCount}");
                Console.WriteLine($"    Revenu: ${region.Revenue:F2}");
                Console.WriteLine($"    Produits: {string.Join(", ", region.Products)}");
            }
            Console.WriteLine();
        }

        // 9. Agrégation avec clause Having (filtrage après groupement)
        Console.WriteLine("--- Agrégation avec clause Having ---");

        var highVolumeProducts = sales
            .GroupBy(s => s.ProductName)
            .Where(g => g.Sum(s => s.Quantity) > 5)  // Having
            .Select(g => new
            {
                Product = g.Key,
                TotalSold = g.Sum(s => s.Quantity),
                TotalRevenue = g.Sum(s => s.Price * s.Quantity)
            })
            .OrderByDescending(x => x.TotalSold);

        Console.WriteLine("Produits à haut volume (plus de 5 vendus):");
        foreach (var product in highVolumeProducts)
        {
            Console.WriteLine($"  {product.Product}: {product.TotalSold} unités, ${product.TotalRevenue:F2}");
        }

        // 10. Agrégation personnalisée
        Console.WriteLine("\n--- Agrégation personnalisée ---");

        string productList = sales
            .OrderByDescending(s => s.Price * s.Quantity)
            .Select(s => s.ProductName)
            .Aggregate((current, next) => current + ", " + next);

        Console.WriteLine($"Liste de tous les produits: {productList}");

        // Agrégation personnalisée avec seed
        decimal totalDiscountedRevenue = sales.Aggregate(0m, (total, sale) =>
        {
            decimal discount = sale.Price > 500 ? 0.1m : 0.05m;
            decimal discountedPrice = sale.Price * (1 - discount);
            return total + (discountedPrice * sale.Quantity);
        });

        Console.WriteLine($"Revenu total après remises: ${totalDiscountedRevenue:F2}");

        // 11. Quartiles de prix (exemple avancé)
        Console.WriteLine("\n--- Analyse statistique (quartiles) ---");

        var sortedPrices = sales.Select(s => s.Price).OrderBy(p => p).ToList();
        int count = sortedPrices.Count;

        decimal median = count % 2 == 0
            ? (sortedPrices[count / 2 - 1] + sortedPrices[count / 2]) / 2
            : sortedPrices[count / 2];

        decimal lowerQuartile = sortedPrices[count / 4];
        decimal upperQuartile = sortedPrices[3 * count / 4];

        Console.WriteLine($"Statistiques des prix:");
        Console.WriteLine($"  Minimum: ${sortedPrices.First():F2}");
        Console.WriteLine($"  Premier quartile (Q1): ${lowerQuartile:F2}");
        Console.WriteLine($"  Médiane: ${median:F2}");
        Console.WriteLine($"  Troisième quartile (Q3): ${upperQuartile:F2}");
        Console.WriteLine($"  Maximum: ${sortedPrices.Last():F2}");
    }
}

class Sale
{
    public int Id { get; set; }
    public string ProductName { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
    public DateTime Date { get; set; }
    public string Region { get; set; }
}
```
### Opérations d'agrégation
Les opérations d'agrégation permettent de calculer une valeur unique à partir d'une séquence d'éléments:
1. **Count / LongCount** : Compte le nombre d'éléments dans une séquence.
2. **Sum** : Calcule la somme des valeurs numériques dans une séquence.
3. **Average** : Calcule la moyenne des valeurs numériques dans une séquence.
4. **Min / Max** : Trouve la valeur minimale ou maximale dans une séquence.
5. **Aggregate** : Applique une fonction d'accumulation sur une séquence pour produire une valeur unique.

### GroupBy et opérations sur les groupes
Le groupement vous permet de diviser une séquence en sous-séquences basées sur des clés spécifiées:
1. **GroupBy** : Groupe les éléments selon une clé spécifiée.
2. **Opérations sur chaque groupe** : Vous pouvez appliquer des agrégations (Sum, Count, Average, etc.) sur chaque groupe.
3. **Projection après groupement** : Transforme les groupes en nouveaux types.
4. **Filtrage après groupement** (clause Having en SQL) : Filtre les groupes en fonction des agrégations.

### Modèles d'agrégation courants
1. **Résumés statistiques** : Min, Max, Moyenne, Somme, etc.
2. **Analyse de distribution** : Regroupement par catégories, fréquences, etc.
3. **Agrégation hiérarchique** : Sous-totaux à plusieurs niveaux.
4. **Analyse temporelle** : Agrégation par périodes (jour, mois, année).
5. **Calcul de métriques commerciales** : Total des ventes, moyenne des commandes, etc.

### Bonnes pratiques
1. **Utiliser des projections après le groupement** pour simplifier l'accès aux données agrégées.
2. **Attention à la performance** pour les opérations sur de grandes collections.
3. **Préférer une seule requête LINQ** avec plusieurs opérations plutôt que plusieurs requêtes distinctes.
4. **Matérialiser les résultats intermédiaires** (avec `ToList()`, etc.) si nécessaire pour éviter des calculs répétés.
5. **Considérer l'impact de l'exécution différée** sur les opérations d'agrégation.

### Conclusion
LINQ offre un ensemble puissant d'outils pour l'agrégation et le groupement de données qui vous permettent de:
- Analyser des données avec un code clair et expressif
- Extraire des informations significatives de grandes collections
- Créer des rapports et des visualisations de données
- Transformer des données brutes en informations exploitables

Ces opérations constituent la base de nombreuses applications d'analyse de données, de reporting et de business intelligence.

⏭️ 6.4. [Génériques](/06-fonctionnalites-avancees-de-csharp/6-4-generiques.md)
