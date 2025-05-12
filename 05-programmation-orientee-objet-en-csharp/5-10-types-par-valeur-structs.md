# 5.10. Types par valeur (structs)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les types par valeur (structs) sont une caractéristique fondamentale de C# qui permet de définir des types légers et efficaces stockés directement sur la pile plutôt que sur le tas comme les types référence (classes). Ils sont particulièrement utiles pour représenter des données simples qui n'ont pas besoin d'héritage ou qui bénéficient d'une sémantique de valeur.

## 5.10.1. Différence avec les classes

Les structs et les classes ont des similitudes, mais aussi des différences fondamentales qui affectent la façon dont ils sont stockés, passés et utilisés.

### Principales différences

```csharp
using System;

// Exemple de classe (type référence)
public class PointClasse
{
    public int X { get; set; }
    public int Y { get; set; }

    public PointClasse(int x, int y)
    {
        X = x;
        Y = y;
    }

    public double Distance()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
}

// Exemple de struct (type valeur)
public struct PointStruct
{
    public int X { get; set; }
    public int Y { get; set; }

    public PointStruct(int x, int y)
    {
        X = x;
        Y = y;
    }

    public double Distance()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
}

// Utilisation des deux types
public class Program
{
    public static void Main()
    {
        Console.WriteLine("== Comparaison Classe vs Struct ==");

        // Création d'instances
        PointClasse p1Classe = new PointClasse(3, 4);
        PointStruct p1Struct = new PointStruct(3, 4);

        Console.WriteLine($"p1Classe: X={p1Classe.X}, Y={p1Classe.Y}, Distance={p1Classe.Distance()}");
        Console.WriteLine($"p1Struct: X={p1Struct.X}, Y={p1Struct.Y}, Distance={p1Struct.Distance()}");

        // Assignation
        PointClasse p2Classe = p1Classe;
        PointStruct p2Struct = p1Struct;

        // Modification des objets originaux
        p1Classe.X = 10;
        p1Struct.X = 10;

        Console.WriteLine("\nAprès modification des objets originaux:");

        // p2Classe est affecté par la modification de p1Classe (référence partagée)
        Console.WriteLine($"p2Classe: X={p2Classe.X}, Y={p2Classe.Y}"); // Affiche X=10

        // p2Struct n'est pas affecté par la modification de p1Struct (copie de valeur)
        Console.WriteLine($"p2Struct: X={p2Struct.X}, Y={p2Struct.Y}"); // Affiche X=3

        // Passage en paramètre
        ModifierPoint(p1Classe, p1Struct);

        Console.WriteLine("\nAprès appel de la méthode ModifierPoint:");
        Console.WriteLine($"p1Classe: X={p1Classe.X}, Y={p1Classe.Y}"); // Affiche X=20, Y=30
        Console.WriteLine($"p1Struct: X={p1Struct.X}, Y={p1Struct.Y}"); // Affiche X=10, Y=4 (non modifié par la méthode)

        // Test de performance
        DemonstrationPerformance();
    }

    static void ModifierPoint(PointClasse pc, PointStruct ps)
    {
        // Modification de la classe (affecte l'objet original)
        pc.X = 20;
        pc.Y = 30;

        // Modification de la struct (n'affecte que la copie locale)
        ps.X = 20;
        ps.Y = 30;

        // ps est une copie par valeur, donc ces modifications
        // ne sont pas visibles à l'extérieur de la méthode
    }

    static void DemonstrationPerformance()
    {
        const int iterations = 10_000_000;

        Console.WriteLine("\n== Test de performance ==");

        // Test avec struct
        DateTime debutStruct = DateTime.Now;
        PointStruct pointStruct = new PointStruct();

        for (int i = 0; i < iterations; i++)
        {
            pointStruct.X = i;
            pointStruct.Y = i;
            double d = pointStruct.Distance();
        }

        TimeSpan dureeStruct = DateTime.Now - debutStruct;

        // Test avec classe
        DateTime debutClasse = DateTime.Now;
        PointClasse pointClasse = new PointClasse(0, 0);

        for (int i = 0; i < iterations; i++)
        {
            pointClasse.X = i;
            pointClasse.Y = i;
            double d = pointClasse.Distance();
        }

        TimeSpan dureeClasse = DateTime.Now - debutClasse;

        Console.WriteLine($"Struct: {dureeStruct.TotalMilliseconds} ms");
        Console.WriteLine($"Classe: {dureeClasse.TotalMilliseconds} ms");
    }
}
```


### Tableau comparatif

| Caractéristique | Classe (type référence) | Struct (type valeur) |
|-----------------|-------------------------|----------------------|
| Stockage | Tas (heap) | Pile (stack) ou en ligne si membre d'un autre type |
| Assignation | Copie la référence | Copie la valeur entière (toutes les données) |
| Passage en paramètre | Par référence par défaut | Par copie par défaut |
| Héritage | Supporte l'héritage de classe | Ne peut pas hériter de classes, mais peut implémenter des interfaces |
| Constructeur par défaut | Non généré automatiquement | Généré automatiquement, ne peut pas être supprimé |
| Valeur par défaut | `null` | Une instance où tous les champs sont à leur valeur par défaut |
| Finalisation | Peut avoir un finaliseur | Ne peut pas avoir de finaliseur |
| Performance | Allocation plus coûteuse, potentiellement meilleure pour les objets complexes | Plus efficace pour les petits objets, possibles problèmes de performance si trop grands |
| GC (Garbage Collection) | Géré par le GC | N'implique pas le GC (sauf si boxed) |

## 5.10.2. Struct mutable vs immutable

Les structs peuvent être conçus pour être mutables (modifiables) ou immutables (non modifiables). Bien que les deux approches soient possibles, les meilleures pratiques recommandent généralement de concevoir des structs immutables pour éviter des comportements inattendus.

### Struct mutable

```csharp
// Struct mutable - peut être modifié après création
public struct Coordonnee
{
    public double X;
    public double Y;

    public Coordonnee(double x, double y)
    {
        X = x;
        Y = y;
    }

    public void Deplacer(double deltaX, double deltaY)
    {
        X += deltaX;
        Y += deltaY;
    }

    public override string ToString() => $"({X}, {Y})";
}

// Démonstration des pièges des structs mutables
public class ExempleStructMutable
{
    public static void Demontrer()
    {
        Console.WriteLine("== Structs Mutables ==");

        // Création et utilisation standard
        Coordonnee coord = new Coordonnee(3, 4);
        Console.WriteLine($"Coordonnée initiale: {coord}");

        coord.Deplacer(2, 1);
        Console.WriteLine($"Après déplacement: {coord}");

        // Premier piège: Comportement inattendu avec les tableaux de structs
        Coordonnee[] points = new Coordonnee[3];
        points[0] = new Coordonnee(1, 1);
        points[1] = new Coordonnee(2, 2);
        points[2] = new Coordonnee(3, 3);

        Console.WriteLine("\nPoints dans le tableau avant modification:");
        foreach (var p in points)
            Console.WriteLine(p);

        // Cette boucle ne modifie PAS le tableau car foreach travaille sur des copies
        foreach (var p in points)
            p.Deplacer(10, 10); // Modifie seulement la copie locale

        Console.WriteLine("\nPoints après foreach avec modification (pas d'effet):");
        foreach (var p in points)
            Console.WriteLine(p);

        // Pour réellement modifier le tableau, il faut utiliser l'index
        for (int i = 0; i < points.Length; i++)
            points[i].Deplacer(10, 10);

        Console.WriteLine("\nPoints après modification avec index:");
        foreach (var p in points)
            Console.WriteLine(p);

        // Deuxième piège: Propriétés et méthodes qui retournent des structs
        var gestionnaire = new GestionnaireCoordonnees();

        // Cette ligne ne modifie pas la coordonnée stockée dans le gestionnaire
        gestionnaire.Coordonnee.Deplacer(5, 5);
        Console.WriteLine($"\nCoordonnée du gestionnaire (non modifiée): {gestionnaire.Coordonnee}");

        // Solution: obtenir une copie locale, modifier puis réassigner
        var coordTemp = gestionnaire.Coordonnee;
        coordTemp.Deplacer(5, 5);
        gestionnaire.Coordonnee = coordTemp;
        Console.WriteLine($"Coordonnée du gestionnaire (modifiée correctement): {gestionnaire.Coordonnee}");
    }
}

// Classe auxiliaire pour la démonstration
public class GestionnaireCoordonnees
{
    public Coordonnee Coordonnee { get; set; } = new Coordonnee(0, 0);
}
```


### Struct immutable

```csharp
// Struct immutable - ne peut pas être modifié après création
public readonly struct Point3D
{
    // Champs en lecture seule
    public readonly double X { get; }
    public readonly double Y { get; }
    public readonly double Z { get; }

    // Constructeur qui initialise tous les champs
    public Point3D(double x, double y, double z)
    {
        X = x;
        Y = y;
        Z = z;
    }

    // Les méthodes retournent de nouvelles instances au lieu de modifier l'instance actuelle
    public Point3D Deplacer(double deltaX, double deltaY, double deltaZ)
    {
        return new Point3D(X + deltaX, Y + deltaY, Z + deltaZ);
    }

    // Opérations qui retournent de nouvelles instances
    public Point3D AdditionnerPoint(Point3D autre)
    {
        return new Point3D(X + autre.X, Y + autre.Y, Z + autre.Z);
    }

    public double Distance()
    {
        return Math.Sqrt(X * X + Y * Y + Z * Z);
    }

    public double DistanceA(Point3D autre)
    {
        double dx = X - autre.X;
        double dy = Y - autre.Y;
        double dz = Z - autre.Z;
        return Math.Sqrt(dx * dx + dy * dy + dz * dz);
    }

    // Toutes les méthodes peuvent être marquées readonly en C# 8.0+
    public readonly override string ToString() => $"({X}, {Y}, {Z})";
}

// Démonstration de l'utilisation correcte des structs immutables
public class ExempleStructImmutable
{
    public static void Demontrer()
    {
        Console.WriteLine("\n== Structs Immutables ==");

        // Création d'une instance
        Point3D point = new Point3D(1, 2, 3);
        Console.WriteLine($"Point initial: {point}");

        // Les opérations retournent de nouvelles instances
        Point3D pointDeplace = point.Deplacer(3, 4, 5);
        Console.WriteLine($"Nouveau point après déplacement: {pointDeplace}");
        Console.WriteLine($"Point original (inchangé): {point}");

        // Combinaison d'opérations
        Point3D autre = new Point3D(5, 6, 7);
        Point3D combinaison = point.AdditionnerPoint(autre).Deplacer(1, 1, 1);
        Console.WriteLine($"Point combiné: {combinaison}");

        // Calcul de distances
        Console.WriteLine($"Distance du point à l'origine: {point.Distance()}");
        Console.WriteLine($"Distance entre les deux points: {point.DistanceA(autre)}");

        // Utilisation dans un tableau
        Point3D[] points = new Point3D[]
        {
            new Point3D(1, 1, 1),
            new Point3D(2, 2, 2),
            new Point3D(3, 3, 3)
        };

        // Création d'un nouveau tableau avec points déplacés
        Point3D[] pointsDeplaces = new Point3D[points.Length];
        for (int i = 0; i < points.Length; i++)
        {
            pointsDeplaces[i] = points[i].Deplacer(10, 10, 10);
        }

        Console.WriteLine("\nPoints originaux:");
        foreach (var p in points)
            Console.WriteLine(p);

        Console.WriteLine("\nPoints déplacés:");
        foreach (var p in pointsDeplaces)
            Console.WriteLine(p);
    }
}

// Programme principal
public class Program
{
    public static void Main()
    {
        ExempleStructMutable.Demontrer();
        ExempleStructImmutable.Demontrer();

        Console.WriteLine("\n== Bonnes pratiques pour les structs ==");
        Console.WriteLine("1. Préférez les structs immutables (readonly)");
        Console.WriteLine("2. Gardez les structs petits (moins de 16 octets est idéal)");
        Console.WriteLine("3. Utilisez des structs pour les types qui agissent comme des valeurs simples");
        Console.WriteLine("4. Évitez de définir des structs mutables pour éviter les comportements inattendus");
        Console.WriteLine("5. Implémentez Equals() et GetHashCode() pour une égalité par valeur correcte");
    }
}
```


### Considérations de performance

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;

// Struct classique (16 octets)
public struct SmallStruct
{
    public int X;
    public int Y;
    public int Z;
    public int W;
}

// Struct plus grande (64 octets)
public struct LargeStruct
{
    public double A, B, C, D, E, F, G, H;
}

// Classe équivalente à SmallStruct
public class SmallClass
{
    public int X;
    public int Y;
    public int Z;
    public int W;

    public SmallClass(int x, int y, int z, int w)
    {
        X = x;
        Y = y;
        Z = z;
        W = w;
    }
}

// Tests de performance
public class PerformanceTests
{
    const int ITERATIONS = 10_000_000;

    public static void ExecuterTests()
    {
        Console.WriteLine("=== Tests de performance des structs vs classes ===");

        // Test 1: Création simple
        Console.WriteLine("\nTest 1: Création d'objets");
        TestCreation();

        // Test 2: Passage en paramètre
        Console.WriteLine("\nTest 2: Passage en paramètre");
        TestPassageParametre();

        // Test 3: Stockage dans des collections
        Console.WriteLine("\nTest 3: Stockage dans des collections");
        TestCollections();

        // Test 4: Modifications
        Console.WriteLine("\nTest 4: Modifications");
        TestModifications();

        // Test 5: Petit vs grand struct
        Console.WriteLine("\nTest 5: Petit struct vs grand struct");
        TestTailleStruct();
    }

    private static void TestCreation()
    {
        // Test de création de structs
        Stopwatch sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            SmallStruct s = new SmallStruct { X = i, Y = i, Z = i, W = i };
        }
        sw.Stop();
        Console.WriteLine($"Création de {ITERATIONS} SmallStruct: {sw.ElapsedMilliseconds} ms");

        // Test de création de classes
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            SmallClass c = new SmallClass(i, i, i, i);
        }
        sw.Stop();
        Console.WriteLine($"Création de {ITERATIONS} SmallClass: {sw.ElapsedMilliseconds} ms");
    }

    private static void TestPassageParametre()
    {
        SmallStruct structParam = new SmallStruct { X = 1, Y = 2, Z = 3, W = 4 };
        SmallClass classParam = new SmallClass(1, 2, 3, 4);

        // Test de passage de struct
        Stopwatch sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            ProcessStruct(structParam);
        }
        sw.Stop();
        Console.WriteLine($"Passage de {ITERATIONS} SmallStruct: {sw.ElapsedMilliseconds} ms");

        // Test de passage de classe
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            ProcessClass(classParam);
        }
        sw.Stop();
        Console.WriteLine($"Passage de {ITERATIONS} SmallClass: {sw.ElapsedMilliseconds} ms");
    }

    private static void TestCollections()
    {
        // Test de collection de structs
        List<SmallStruct> listeStructs = new List<SmallStruct>();
        Stopwatch sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS / 100; i++)
        {
            listeStructs.Add(new SmallStruct { X = i, Y = i, Z = i, W = i });
        }
        sw.Stop();
        Console.WriteLine($"Ajout de {ITERATIONS / 100} SmallStruct à une liste: {sw.ElapsedMilliseconds} ms");

        // Test de collection de classes
        List<SmallClass> listeClasses = new List<SmallClass>();
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS / 100; i++)
        {
            listeClasses.Add(new SmallClass(i, i, i, i));
        }
        sw.Stop();
        Console.WriteLine($"Ajout de {ITERATIONS / 100} SmallClass à une liste: {sw.ElapsedMilliseconds} ms");
    }

    private static void TestModifications()
    {
        SmallStruct[] structArray = new SmallStruct[1000];
        SmallClass[] classArray = new SmallClass[1000];

        for (int i = 0; i < 1000; i++)
        {
            structArray[i] = new SmallStruct { X = i, Y = i, Z = i, W = i };
            classArray[i] = new SmallClass(i, i, i, i);
        }

        // Test de modification de structs
        Stopwatch sw = Stopwatch.StartNew();
        for (int j = 0; j < ITERATIONS / 1000; j++)
        {
            for (int i = 0; i < 1000; i++)
            {
                structArray[i].X += 1;
                structArray[i].Y += 1;
            }
        }
        sw.Stop();
        Console.WriteLine($"Modification de structs: {sw.ElapsedMilliseconds} ms");

        // Test de modification de classes
        sw = Stopwatch.StartNew();
        for (int j = 0; j < ITERATIONS / 1000; j++)
        {
            for (int i = 0; i < 1000; i++)
            {
                classArray[i].X += 1;
                classArray[i].Y += 1;
            }
        }
        sw.Stop();
        Console.WriteLine($"Modification de classes: {sw.ElapsedMilliseconds} ms");
    }

    private static void TestTailleStruct()
    {
        // Test avec petit struct
        Stopwatch sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            SmallStruct s = new SmallStruct { X = i, Y = i, Z = i, W = i };
            ProcessStruct(s);
        }
        sw.Stop();
        Console.WriteLine($"Opérations sur {ITERATIONS} SmallStruct (16 octets): {sw.ElapsedMilliseconds} ms");

        // Test avec grand struct
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            LargeStruct l = new LargeStruct { A = i, B = i, C = i, D = i, E = i, F = i, G = i, H = i };
            ProcessLargeStruct(l);
        }
        sw.Stop();
        Console.WriteLine($"Opérations sur {ITERATIONS} LargeStruct (64 octets): {sw.ElapsedMilliseconds} ms");
    }

    // Méthodes auxiliaires pour les tests
    private static int ProcessStruct(SmallStruct s)
    {
        return s.X + s.Y + s.Z + s.W;
    }

    private static int ProcessClass(SmallClass c)
    {
        return c.X + c.Y + c.Z + c.W;
    }

    private static double ProcessLargeStruct(LargeStruct l)
    {
        return l.A + l.B + l.C + l.D + l.E + l.F + l.G + l.H;
    }
}

// Programme principal
public class Program
{
    public static void Main()
    {
        PerformanceTests.ExecuterTests();
    }
}
```


## 5.10.3. Struct records (C# 10+)

Depuis C# 10, il est possible de définir des records comme des types valeur (struct) en combinant les avantages des records (égalité basée sur les valeurs, déconstruction, with-expressions) et des structs (stockage sur la pile).

> **Note** : Cette fonctionnalité nécessite .NET 6 ou supérieur.

```csharp
// Record struct standard (C# 10+, .NET 6+)
public record struct Coordonnees(double Latitude, double Longitude)
{
    // Méthode supplémentaire
    public double DistanceDe(Coordonnees autre)
    {
        // Formule simplifiée de la distance euclidienne (à titre d'exemple)
        double dLat = Latitude - autre.Latitude;
        double dLon = Longitude - autre.Longitude;
        return Math.Sqrt(dLat * dLat + dLon * dLon);
    }
}

// Record struct en lecture seule (C# 10+, .NET 6+)
public readonly record struct Temperature(double Celsius)
{
    // Propriétés calculées
    public double Fahrenheit => Celsius * 9 / 5 + 32;
    public double Kelvin => Celsius + 273.15;

    // Méthode de fabrique statique
    public static Temperature FromFahrenheit(double fahrenheit)
    {
        return new Temperature((fahrenheit - 32) * 5 / 9);
    }

    // Opérateurs
    public static Temperature operator +(Temperature a, Temperature b)
    {
        return new Temperature(a.Celsius + b.Celsius);
    }

    public static Temperature operator -(Temperature a, Temperature b)
    {
        return new Temperature(a.Celsius - b.Celsius);
    }

    // Pas besoin de surcharger Equals ou GetHashCode - le record le fait automatiquement
}

// Démonstration des records struct
public class ExemplesRecordStruct
{
    public static void Demontrer()
    {
        Console.WriteLine("=== Record Structs (C# 10+) ===");

        // Utilisation du record struct Coordonnees
        Coordonnees paris = new Coordonnees(48.8566, 2.3522);
        Coordonnees lyon = new Coordonnees(45.7640, 4.8357);

        Console.WriteLine($"Paris: {paris}"); // ToString() généré automatiquement
        Console.WriteLine($"Lyon: {lyon}");
        Console.WriteLine($"Distance entre Paris et Lyon: {paris.DistanceDe(lyon):F2} degrés");

        // Création d'une copie avec with-expression
        Coordonnees parisModifie = paris with { Longitude = 2.3530 };
        Console.WriteLine($"Paris modifié: {parisModifie}");

        // Déconstruction
        var (lat, lon) = lyon;
        Console.WriteLine($"Lyon décomposé: Latitude = {lat}, Longitude = {lon}");

        // Égalité basée sur les valeurs
        Coordonnees paris2 = new Coordonnees(48.8566, 2.3522);
        Console.WriteLine($"paris == paris2: {paris == paris2}"); // True
        Console.WriteLine($"paris == parisModifie: {paris == parisModifie}"); // False

        // Utilisation du record struct readonly Temperature
        Temperature ambiante = new Temperature(22.5);
        Temperature congelation = new Temperature(0);

        Console.WriteLine($"\nTempérature ambiante: {ambiante.Celsius:F1}°C, {ambiante.Fahrenheit:F1}°F, {ambiante.Kelvin:F1}K");

        // Utilisation de méthodes de fabrique
        Temperature fahrenheit100 = Temperature.FromFahrenheit(100);
        Console.WriteLine($"100°F = {fahrenheit100.Celsius:F1}°C");

        // Opérateurs surchargés
        Temperature somme = ambiante + fahrenheit100;
        Console.WriteLine($"Somme: {somme.Celsius:F1}°C");

        // With-expression
        Temperature chaude = ambiante with { Celsius = 30 };
        Console.WriteLine($"Température modifiée: {chaude.Celsius:F1}°C");

        // Égalité et hachage
        Dictionary<Temperature, string> descriptions = new Dictionary<Temperature, string>
        {
            { new Temperature(0), "Congélation de l'eau" },
            { new Temperature(100), "Ébullition de l'eau" },
            { new Temperature(22.5), "Température ambiante standard" }
        };

        Console.WriteLine($"\nDescription pour 0°C: {descriptions[congelation]}");
        Console.WriteLine($"Description pour 22.5°C: {descriptions[ambiante]}");
    }
}
```


### Comparaison de performances entre struct, record class et record struct

```csharp
public class ComparaisonPerformance
{
    const int ITERATIONS = 1_000_000;

    // Struct standard
    public struct PointStruct
    {
        public int X { get; init; }
        public int Y { get; init; }

        public PointStruct(int x, int y)
        {
            X = x;
            Y = y;
        }

        public override bool Equals(object obj)
        {
            return obj is PointStruct ps && X == ps.X && Y == ps.Y;
        }

        public override int GetHashCode()
        {
            return HashCode.Combine(X, Y);
        }
    }

    // Record class
    public record class PointRecordClass(int X, int Y);

    // Record struct
    public record struct PointRecordStruct(int X, int Y);

    public static void ComparePerformance()
    {
        Console.WriteLine("\n=== Comparaison de performance ===");

        // Test avec struct standard
        Stopwatch sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            PointStruct p = new PointStruct(i % 100, i % 200);
            int hash = p.GetHashCode();
            bool equals = p.Equals(new PointStruct(i % 100, i % 200));
        }
        sw.Stop();
        Console.WriteLine($"Struct standard: {sw.ElapsedMilliseconds} ms");

        // Test avec record class
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            PointRecordClass p = new PointRecordClass(i % 100, i % 200);
            int hash = p.GetHashCode();
            bool equals = p.Equals(new PointRecordClass(i % 100, i % 200));
        }
        sw.Stop();
        Console.WriteLine($"Record class: {sw.ElapsedMilliseconds} ms");

        // Test avec record struct
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            PointRecordStruct p = new PointRecordStruct(i % 100, i % 200);
            int hash = p.GetHashCode();
            bool equals = p.Equals(new PointRecordStruct(i % 100, i % 200));
        }
        sw.Stop();
        Console.WriteLine($"Record struct: {sw.ElapsedMilliseconds} ms");

        // Test de copie avec with-expression
        // Record class
        PointRecordClass rc = new PointRecordClass(1, 2);
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            var copy = rc with { X = i % 100 };
        }
        sw.Stop();
        Console.WriteLine($"With-expression record class: {sw.ElapsedMilliseconds} ms");

        // Record struct
        PointRecordStruct rs = new PointRecordStruct(1, 2);
        sw = Stopwatch.StartNew();
        for (int i = 0; i < ITERATIONS; i++)
        {
            var copy = rs with { X = i % 100 };
        }
        sw.Stop();
        Console.WriteLine($"With-expression record struct: {sw.ElapsedMilliseconds} ms");
    }
}
```


### Avantages des record structs

Les record structs en C# 10+ combinent les avantages des structs et des records, offrant plusieurs bénéfices :

1. **Allocation sur la pile** : Comme les structs traditionnels, les record structs sont alloués sur la pile plutôt que sur le tas, ce qui peut améliorer les performances pour les petits objets fréquemment créés.

2. **Syntaxe concise** : La syntaxe des records permet de définir des structures de données de manière beaucoup plus concise que les structs traditionnels.

3. **Égalité basée sur les valeurs** : Les record structs implémentent automatiquement l'égalité structurelle, comparant les valeurs de toutes les propriétés.

4. **With-expressions** : La possibilité de créer une copie non-destructive avec des propriétés modifiées via la syntaxe `with`.

5. **Déconstruction automatique** : Permet de décomposer l'objet en ses composants individuels.

6. **ToString() amélioré** : L'implémentation par défaut de ToString() affiche toutes les propriétés de manière lisible.

### Limitations et considérations

Il existe certaines limitations et considérations à prendre en compte lors de l'utilisation des record structs :

1. **Pas d'héritage** : Comme les structs normaux, les record structs ne peuvent pas hériter d'autres classes ou structs, mais peuvent implémenter des interfaces.

2. **Risque de copie excessive** : Si vous passez des structs volumineux en paramètres ou effectuez de nombreuses copies, les performances peuvent se dégrader en raison de la copie par valeur.

3. **Version de .NET requise** : Les record structs nécessitent C# 10 et .NET 6 ou versions ultérieures.

4. **Constructeur par défaut** : Les struct records ont toujours un constructeur par défaut qui initialise toutes les propriétés à leur valeur par défaut, ce qui n'est pas toujours souhaitable.

### Exemple d'utilisation pratique

```csharp
// Exemple pratique d'utilisation de record struct pour un système de gestion de produits
public readonly record struct SKU(string Code, string Catégorie);

public readonly record struct Dimensions(double Longueur, double Largeur, double Hauteur)
{
    public double Volume => Longueur * Largeur * Hauteur;

    public Dimensions Redimensionner(double facteur) =>
        new Dimensions(Longueur * facteur, Largeur * facteur, Hauteur * facteur);
}

public readonly record struct Prix(decimal Montant, string Devise)
{
    public static Prix Euro(decimal montant) => new Prix(montant, "EUR");
    public static Prix Dollar(decimal montant) => new Prix(montant, "USD");

    public override string ToString() => $"{Montant:F2} {Devise}";
}

public readonly record struct Produit(string Nom, SKU CodeProduit, Prix PrixUnitaire, Dimensions Dimensions, double Poids)
{
    public double Volume => Dimensions.Volume;

    // Méthodes métier
    public Produit AvecRemise(decimal pourcentageRemise)
    {
        decimal facteur = 1 - (pourcentageRemise / 100);
        return this with
        {
            PrixUnitaire = new Prix(PrixUnitaire.Montant * facteur, PrixUnitaire.Devise)
        };
    }

    public Produit RedimensionnerEmballage(double facteur)
    {
        return this with
        {
            Dimensions = Dimensions.Redimensionner(facteur)
        };
    }
}
```


### Quand utiliser les record structs

Les record structs sont particulièrement adaptés dans les scénarios suivants :

1. **Petits types de données immuables** : Lorsque vous avez besoin de représenter de petites structures de données qui ne changeront pas après leur création.

2. **Objets à durée de vie courte** : Pour les objets qui sont fréquemment créés et détruits dans des boucles ou des opérations intensives.

3. **Collections de données** : Pour les éléments stockés dans des collections où l'égalité basée sur les valeurs est importante.

4. **Types à sémantique de valeur** : Pour les types qui représentent conceptuellement des valeurs plutôt que des entités (par exemple, coordonnées, températures, mesures).

5. **Alternatives aux tuples** : Comme alternative aux tuples lorsque vous avez besoin de plus de clarté et de fonctionnalités.

### Conseils pour choisir entre les différents types

```csharp
// Guide de choix entre class, struct, record class et record struct
public class GuideDeChoix
{
    public static void AfficherRecommandations()
    {
        Console.WriteLine("\n=== Conseils pour choisir le bon type ===");
        Console.WriteLine("Utilisez une CLASS lorsque :");
        Console.WriteLine("- Vous avez besoin d'une sémantique de référence");
        Console.WriteLine("- L'objet est mutable (ses propriétés changent fréquemment)");
        Console.WriteLine("- L'objet est grand (nombreuses propriétés)");
        Console.WriteLine("- Vous avez besoin d'héritage de classe");
        Console.WriteLine("- La logique est complexe ou le comportement est plus important que les données");

        Console.WriteLine("\nUtilisez un STRUCT lorsque :");
        Console.WriteLine("- L'objet est petit (généralement moins de 16 octets)");
        Console.WriteLine("- L'objet est utilisé fréquemment et de courte durée");
        Console.WriteLine("- Une sémantique de valeur est nécessaire");
        Console.WriteLine("- L'allocation sur le tas serait inefficace");
        Console.WriteLine("- L'objet représente une valeur simple comme une coordonnée");

        Console.WriteLine("\nUtilisez un RECORD CLASS lorsque :");
        Console.WriteLine("- Vous avez besoin d'une sémantique de référence");
        Console.WriteLine("- Vous voulez une égalité basée sur les valeurs");
        Console.WriteLine("- L'immutabilité est importante");
        Console.WriteLine("- Vous avez besoin de with-expressions pour les copies modifiées");
        Console.WriteLine("- Vous voulez une déconstruction automatique");

        Console.WriteLine("\nUtilisez un RECORD STRUCT lorsque :");
        Console.WriteLine("- Vous avez besoin d'une sémantique de valeur (allocation sur la pile)");
        Console.WriteLine("- Vous voulez une égalité basée sur les valeurs");
        Console.WriteLine("- L'immutabilité est importante (avec 'readonly record struct')");
        Console.WriteLine("- Vous voulez la concision des records");
        Console.WriteLine("- L'objet est petit et représente un ensemble de données simples");
        Console.WriteLine("- Vous voulez optimiser les performances pour des petits objets souvent copiés");
    }
}
```


Les record structs constituent une fusion élégante entre les types par valeur (structs) et les records introduits dans C# 9. Ils sont particulièrement utiles pour représenter des données immuables qui bénéficient d'une allocation sur la pile plutôt que sur le tas, offrant ainsi à la fois une syntaxe concise et des performances optimisées pour les petits objets de données.

--

Les types par valeur (structs) constituent un outil essentiel dans la boîte à outils du développeur C#, offrant une alternative performante aux classes traditionnelles pour certains scénarios spécifiques. Au fil des versions de C#, la flexibilité et les capacités des structs se sont considérablement améliorées, culminant avec l'introduction des record structs en C# 10.
Comme nous l'avons vu à travers les sections précédentes, les structs se distinguent des classes par leur comportement de copie par valeur, leur allocation sur la pile, et leur impossibilité d'héritage. Ces caractéristiques les rendent particulièrement adaptés pour représenter des données simples, de petite taille, et qui bénéficient d'une sémantique de valeur plutôt que de référence.
La question de la mutabilité des structs est cruciale pour éviter des comportements inattendus. Bien que les structs mutables soient techniquement possibles, ils peuvent conduire à des comportements contre-intuitifs, en particulier lors de l'utilisation avec des collections, des méthodes ou des propriétés. Les structs immutables, quant à eux, offrent une approche plus cohérente et prévisible, alignée avec la sémantique de valeur naturelle des structs.
L'arrivée des record structs dans C# 10 représente une évolution majeure, combinant la sémantique de valeur des structs traditionnels avec les fonctionnalités des records : égalité basée sur les valeurs, with-expressions pour les copies non-destructives, déconstruction, et ToString() amélioré. Cette fusion offre une syntaxe concise et expressive pour définir des types de données immuables qui bénéficient également des avantages de performance des structs.
Le choix entre classes, structs, record classes et record structs doit être guidé par plusieurs facteurs :
- La taille des données
- La fréquence de création/destruction des instances
- Le besoin d'héritage
- La mutabilité ou l'immutabilité requise
- La sémantique d'égalité souhaitée (référence vs valeur)
- Les performances recherchées

En comprenant les forces et les limitations de chaque approche, vous pouvez sélectionner le type le plus approprié pour représenter vos données et optimiser à la fois la lisibilité du code et les performances de votre application.
Les types par valeur, loin d'être un simple détail d'implémentation, peuvent avoir un impact significatif sur l'architecture, les performances et la maintenabilité de vos applications. Utilisés judicieusement, ils constituent un outil puissant dans la conception de systèmes robustes et efficaces en C#.

⏭️
