# 17.1. Gestion de la mémoire en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La gestion de la mémoire est un aspect fondamental de la programmation en C#. Bien que le langage dispose d'un Garbage Collector (GC) qui s'occupe automatiquement de libérer la mémoire inutilisée, comprendre comment fonctionne la mémoire vous permet d'écrire des applications plus performantes et plus efficaces.

## 17.1.1. Types référence vs types valeur

En C#, il existe deux catégories fondamentales de types de données: les **types valeur** et les **types référence**. Cette distinction est cruciale pour comprendre comment vos données sont stockées et manipulées en mémoire.

### Types valeur

Les types valeur sont stockés directement sur la **pile** (stack) et contiennent directement leurs données.

**Caractéristiques principales:**
- Stockés sur la pile (une zone de mémoire rapide mais limitée)
- Passés par copie lors des affectations et des appels de méthodes
- Durée de vie liée à leur portée (scope)
- Ne peuvent pas être `null` (sauf s'ils sont déclarés comme nullables avec `?`)

**Exemples de types valeur:**
- Types primitifs: `int`, `float`, `double`, `bool`, `char`
- Structures (`struct`)
- Énumérations (`enum`)
- Tuples

**Exemple de code:**

```csharp
// Types valeur
int nombre = 42;
bool estVrai = true;
char caractere = 'A';

// Passage par copie (passage par valeur)
int a = 10;
int b = a;  // 'b' reçoit une COPIE de la valeur de 'a'
a = 20;     // Modifier 'a' n'affecte pas 'b'
Console.WriteLine($"a = {a}, b = {b}");  // Affiche "a = 20, b = 10"

// Structure (type valeur)
struct Point
{
    public int X;
    public int Y;
}

Point p1 = new Point { X = 10, Y = 20 };
Point p2 = p1;  // p2 est une COPIE de p1
p1.X = 30;      // Modifier p1 n'affecte pas p2
Console.WriteLine($"p1.X = {p1.X}, p2.X = {p2.X}");  // Affiche "p1.X = 30, p2.X = 10"
```

### Types référence

Les types référence stockent une **référence** (une adresse mémoire) qui pointe vers les données réelles stockées sur le **tas** (heap).

**Caractéristiques principales:**
- La référence est stockée sur la pile
- Les données réelles sont stockées sur le tas (une zone de mémoire plus grande mais plus lente)
- Passés par référence lors des affectations et des appels de méthodes
- Peuvent être `null` (absence de référence)
- Gérés par le Garbage Collector

**Exemples de types référence:**
- Classes (`class`)
- Interfaces
- Délégués
- Tableaux
- Chaînes de caractères (`string`) - bien que traitées de manière spéciale

**Exemple de code:**

```csharp
// Types référence
string message = "Bonjour";
int[] nombres = { 1, 2, 3, 4, 5 };

// Classes (types référence)
class Personne
{
    public string Nom;
    public int Age;
}

// Passage par référence
Personne p1 = new Personne { Nom = "Alice", Age = 30 };
Personne p2 = p1;  // p2 référence le MÊME objet que p1
p1.Age = 31;       // Modifier via p1 affecte aussi p2
Console.WriteLine($"p1.Age = {p1.Age}, p2.Age = {p2.Age}");  // Affiche "p1.Age = 31, p2.Age = 31"
```

### Différences visuelles entre types valeur et types référence

Pour mieux comprendre, visualisons comment ces types sont stockés en mémoire:

```
// Types valeur (sur la pile)
┌───────────┐
│ int a = 5 │  La valeur 5 est directement stockée dans 'a'
└───────────┘

// Types référence (référence sur la pile, données sur le tas)
┌───────────┐        ┌──────────────────────┐
│ Personne p│───────>│ Nom: "Alice"         │
└───────────┘        │ Age: 30              │
  (pile)             └──────────────────────┘
                      (tas)
```

### Impact sur les performances

- Les **types valeur** sont généralement plus rapides car:
  - Pas de recherche d'adresse mémoire (indirection)
  - Pas d'allocation sur le tas
  - Pas de gestion par le Garbage Collector

- Les **types référence** sont plus flexibles mais potentiellement plus lents car:
  - Nécessitent une indirection (suivre un pointeur)
  - Impliquent une allocation sur le tas
  - Doivent être nettoyés par le Garbage Collector

### Quand utiliser l'un ou l'autre?

- Utilisez des **types valeur** pour:
  - De petites structures de données simples
  - Des objets qui ne changent pas d'état (immutables)
  - Des situations où la performance est critique

- Utilisez des **types référence** pour:
  - Des objets complexes avec des méthodes
  - Des objets qui doivent être modifiés (mutables)
  - Des objets qui peuvent être `null`
  - Des objets qui participent à l'héritage

## 17.1.2. Stratégies d'allocation

Pour écrire du code C# efficace, il est important de comprendre comment et quand la mémoire est allouée.

### Allocation sur la pile (Stack)

La pile est une zone de mémoire limitée en taille mais très rapide d'accès. L'allocation et la désallocation sur la pile sont automatiques et suivent l'ordre d'exécution du programme.

**Caractéristiques:**
- Allocation et désallocation LIFO (Last In, First Out)
- Extrêmement rapide
- Taille limitée (généralement quelques Mo)
- Utilisée pour les variables locales, les types valeur et les références

**Exemple:**

```csharp
void MaMethode()
{
    int x = 42;       // Alloué sur la pile
    double y = 3.14;  // Alloué sur la pile

    // À la fin de la méthode, x et y sont automatiquement "désalloués"
    // quand la méthode se termine et que la pile se vide
}
```

### Allocation sur le tas (Heap)

Le tas est une zone de mémoire beaucoup plus grande, mais plus lente d'accès. L'allocation sur le tas est explicite (avec `new`) et la désallocation est gérée par le Garbage Collector.

**Caractéristiques:**
- Allocation explicite avec `new`
- Désallocation automatique par le GC
- Plus lente que la pile
- Taille beaucoup plus grande (limitée par la mémoire disponible)
- Utilisée pour les types référence

**Exemple:**

```csharp
void MaMethode()
{
    // La référence 'personne' est sur la pile,
    // mais l'objet lui-même est alloué sur le tas
    Personne personne = new Personne();

    // À la fin de la méthode, la référence 'personne' disparaît,
    // mais l'objet reste en mémoire jusqu'à ce que le GC le nettoie
}
```

### Stratégies d'optimisation

Voici quelques stratégies pour optimiser l'allocation de mémoire:

1. **Minimiser les allocations sur le tas:**
   - Utilisez des types valeur pour les petites structures de données
   - Évitez de créer inutilement des objets temporaires

2. **Réutiliser les objets:**
   - Plutôt que de créer et jeter des objets, réutilisez-les
   - Utilisez des pools d'objets pour les objets fréquemment créés

3. **Éviter les allocations dans les boucles:**
   - Déplacez les allocations hors des boucles quand c'est possible
   - Utilisez des structures de données préallouées

4. **Attention aux fermetures (closures):**
   - Les expressions lambda et les méthodes anonymes peuvent créer des allocations cachées

**Exemple d'optimisation:**

```csharp
// Non optimisé: crée une nouvelle chaîne à chaque itération
void NonOptimisé(int iterations)
{
    for (int i = 0; i < iterations; i++)
    {
        string message = "Itération " + i;  // Nouvelle allocation à chaque fois
        Console.WriteLine(message);
    }
}

// Optimisé: utilise StringBuilder pour minimiser les allocations
void Optimisé(int iterations)
{
    StringBuilder sb = new StringBuilder();  // Une seule allocation
    for (int i = 0; i < iterations; i++)
    {
        sb.Clear();
        sb.Append("Itération ").Append(i);  // Pas de nouvelle allocation
        Console.WriteLine(sb.ToString());    // Minimise les allocations
    }
}
```

## 17.1.3. Span<T> et Memory<T>

`Span<T>` et `Memory<T>` sont des types introduits dans .NET Core 2.1 pour manipuler efficacement des séquences contiguës de mémoire sans créer de copies.

### Span<T>

`Span<T>` représente une vue continue sur de la mémoire, que ce soit sur la pile, sur le tas, ou même sur de la mémoire native. C'est un type valeur qui doit être utilisé pour des opérations à courte durée de vie.

**Caractéristiques principales:**
- Type valeur très léger (seulement 2 mots)
- Peut représenter toute séquence contiguë en mémoire
- Ne peut pas être utilisé comme champ de classe (limité à la pile)
- N'alloue pas de mémoire supplémentaire
- Parfait pour les segments de tableaux, de chaînes, etc.

**Exemple de base:**

```csharp
// Utilisation de Span avec un tableau
int[] tableau = { 1, 2, 3, 4, 5 };
Span<int> span = tableau;  // Couvre tout le tableau

// Sous-section du tableau
Span<int> milieu = tableau.AsSpan(1, 3);  // Éléments 2, 3, 4

// Modification via Span
milieu[0] = 42;  // Modifie le deuxième élément du tableau original

// Affiche 1, 42, 3, 4, 5
foreach (var n in tableau)
{
    Console.Write($"{n} ");
}
```

**Exemple avec des chaînes:**

```csharp
string texte = "Bonjour le monde";
ReadOnlySpan<char> spanTexte = texte.AsSpan();

// Extraction sans allocation
ReadOnlySpan<char> premierMot = spanTexte.Slice(0, 7);  // "Bonjour"
Console.WriteLine(premierMot.ToString());

// Parcours sans allocation
for (int i = 0; i < spanTexte.Length; i++)
{
    Console.Write(spanTexte[i]);
}
```

### Memory<T>

`Memory<T>` est similaire à `Span<T>`, mais c'est un type référence qui peut être stocké comme un champ de classe et utilisé dans des méthodes asynchrones.

**Caractéristiques principales:**
- Type référence
- Peut être stocké comme champ de classe
- Peut être utilisé avec `async`/`await`
- Légèrement moins performant que `Span<T>`
- Peut être converti en `Span<T>` si nécessaire

**Exemple:**

```csharp
class TraitementDeDonnées
{
    // On peut stocker un Memory<T> comme champ (contrairement à Span<T>)
    private Memory<byte> _données;

    public TraitementDeDonnées(byte[] données)
    {
        _données = données;
    }

    public async Task TraiterAsync()
    {
        // Utilisation dans un contexte async
        await Task.Delay(100);

        // Conversion en Span pour utilisation efficace
        Span<byte> spanDonnées = _données.Span;

        // Traitement des données...
        for (int i = 0; i < spanDonnées.Length; i++)
        {
            spanDonnées[i] *= 2;
        }
    }
}
```

### Quand utiliser Span<T> vs Memory<T>?

- Utilisez **Span<T>** pour:
  - Opérations synchrones à courte durée de vie
  - Maximum de performance
  - Méthodes locales et code non-async

- Utilisez **Memory<T>** pour:
  - Stockage dans des champs de classe
  - Opérations asynchrones (avec `async`/`await`)
  - Quand vous avez besoin de conserver la vue plus longtemps

### Avantages de Span<T> et Memory<T>

1. **Réduction des allocations:**
   - Permet de manipuler des sections de tableaux ou de chaînes sans créer de copies
   - Réduit la pression sur le GC

2. **Performances accrues:**
   - Moins d'allocations = moins de GC = meilleures performances
   - Accès direct à la mémoire sous-jacente

3. **API unifiée:**
   - Fonctionne sur différents types de mémoire (tableau, chaîne, mémoire native)
   - Simplifie le code qui manipule des données binaires

## 17.1.4. Évitement des allocations inutiles

Réduire les allocations inutiles est l'une des techniques les plus efficaces pour améliorer les performances de vos applications C#, surtout dans les chemins critiques et les boucles internes.

### Techniques pour éviter les allocations

#### 1. Utiliser des types valeur plutôt que des types référence

Les structures (`struct`) peuvent être plus performantes que les classes pour les petits objets.

```csharp
// Classe (type référence) - allocation sur le tas
class PointClass
{
    public int X;
    public int Y;
}

// Structure (type valeur) - allocation sur la pile
struct PointStruct
{
    public int X;
    public int Y;
}

void Exemple()
{
    // Allocation sur le tas
    PointClass pc = new PointClass { X = 10, Y = 20 };

    // Pas d'allocation sur le tas
    PointStruct ps = new PointStruct { X = 10, Y = 20 };
}
```

#### 2. Utiliser StringBuilder pour la concaténation de chaînes

La concaténation répétée de chaînes crée de nombreuses allocations temporaires.

```csharp
// Inefficace: crée plusieurs chaînes temporaires
string BuildMessage(string name, int count)
{
    string result = "Bonjour " + name + ", vous avez " + count + " messages.";
    return result;
}

// Efficace: utilise StringBuilder
string BuildMessageOptimized(string name, int count)
{
    StringBuilder sb = new StringBuilder();
    sb.Append("Bonjour ");
    sb.Append(name);
    sb.Append(", vous avez ");
    sb.Append(count);
    sb.Append(" messages.");
    return sb.ToString();
}

// Alternative moderne: utiliser l'interpolation de chaînes
// (moins efficace que StringBuilder pour de nombreuses opérations,
// mais plus efficace qu'une concaténation manuelle)
string BuildMessageInterpolated(string name, int count)
{
    return $"Bonjour {name}, vous avez {count} messages.";
}
```

#### 3. Réutiliser des buffers/tableaux

Plutôt que de créer de nouveaux tableaux, réutilisez-les quand c'est possible.

```csharp
// Inefficace: crée un nouveau tableau à chaque appel
byte[] ProcessData(byte[] source)
{
    byte[] result = new byte[source.Length];
    for (int i = 0; i < source.Length; i++)
    {
        result[i] = (byte)(source[i] * 2);
    }
    return result;
}

// Efficace: utilise un buffer réutilisable
class DataProcessor
{
    private byte[] _buffer;

    public byte[] ProcessData(byte[] source)
    {
        // Redimensionne le buffer uniquement si nécessaire
        if (_buffer == null || _buffer.Length < source.Length)
        {
            _buffer = new byte[source.Length];
        }

        for (int i = 0; i < source.Length; i++)
        {
            _buffer[i] = (byte)(source[i] * 2);
        }

        return _buffer;
    }
}
```

#### 4. Utiliser des types Span pour éviter les copies

```csharp
// Inefficace: crée une nouvelle chaîne
string ExtractSubstring(string source, int start, int length)
{
    return source.Substring(start, length);
}

// Efficace: n'alloue pas de nouvelle mémoire
ReadOnlySpan<char> ExtractSpan(string source, int start, int length)
{
    return source.AsSpan(start, length);
}
```

#### 5. Utiliser des méthodes d'extension sans allocation

De nombreuses méthodes d'extension dans les nouvelles versions de .NET sont conçues pour réduire les allocations.

```csharp
// Avec allocation
bool ContainsTraditional(string source, string value)
{
    return source.Contains(value);
}

// Sans allocation (à partir de .NET Core 2.1)
bool ContainsNoAlloc(string source, string value)
{
    return source.AsSpan().Contains(value.AsSpan(), StringComparison.Ordinal);
}
```

#### 6. Éviter les closures et les captures de variables

Les lambdas et délégués qui capturent des variables locales peuvent provoquer des allocations cachées.

```csharp
// Inefficace: allocation d'une closure qui capture 'threshold'
void ProcessWithCapture(int[] numbers, int threshold)
{
    // Cette lambda capture 'threshold', créant une allocation
    var filtered = numbers.Where(n => n > threshold).ToArray();
}

// Efficace: pas de capture de variable
void ProcessWithoutCapture(int[] numbers, int threshold)
{
    // Évite la capture en passant le seuil comme paramètre
    var filtered = numbers.Where(GreaterThan).ToArray();

    bool GreaterThan(int n) => n > threshold;
}
```

### Détection des allocations cachées

Utilisez des outils comme:
- **Benchmark.NET** pour mesurer les performances
- **dotTrace** ou **ANTS Performance Profiler** pour identifier les allocations
- **SharpLab** (https://sharplab.io) pour voir comment le compilateur traite votre code

## 17.1.5. Pooling d'objets

Le pooling d'objets est une technique d'optimisation qui consiste à réutiliser des objets plutôt que d'en créer de nouveaux, réduisant ainsi la pression sur le Garbage Collector.

### Principe du pooling d'objets

Au lieu de créer et détruire constamment des objets, on maintient un "pool" (réservoir) d'objets préalloués. Quand on a besoin d'un objet, on en emprunte un du pool, et quand on a fini, on le retourne au pool plutôt que de le laisser pour le GC.

### Avantages du pooling

1. **Réduction des allocations:** Moins de travail pour le GC
2. **Meilleures performances:** Évite le coût de création d'objets
3. **Moins de fragmentation de la mémoire**
4. **Contrôle de la consommation de ressources**

### ObjectPool<T>

.NET Core (à partir de la version 2.0) inclut une classe `ObjectPool<T>` dans le package `Microsoft.Extensions.ObjectPool` qui facilite l'implémentation du pooling.

**Exemple d'utilisation:**

```csharp
using Microsoft.Extensions.ObjectPool;

// Créer une fabrique d'objets (définit comment les objets sont créés)
class StringBuilderPooledObjectPolicy : PooledObjectPolicy<StringBuilder>
{
    public override StringBuilder Create()
    {
        return new StringBuilder();
    }

    public override bool Return(StringBuilder obj)
    {
        obj.Clear();  // Réinitialiser l'objet avant de le retourner au pool
        return true;  // Accepter l'objet dans le pool
    }
}

class Program
{
    static void Main()
    {
        // Créer un pool
        var provider = new DefaultObjectPoolProvider();
        var pool = provider.Create(new StringBuilderPooledObjectPolicy());

        // Utiliser un objet du pool
        var sb = pool.Get();
        try
        {
            sb.Append("Exemple de texte");
            Console.WriteLine(sb.ToString());
        }
        finally
        {
            // Important: retourner l'objet au pool quand on a fini
            pool.Return(sb);
        }
    }
}
```

### ArrayPool<T>

Pour les tableaux, .NET fournit `ArrayPool<T>` (dans le namespace `System.Buffers`), qui est spécialement optimisé pour le pooling de tableaux.

**Exemple d'utilisation:**

```csharp
using System.Buffers;

void ProcessLargeData(Stream source)
{
    // Emprunter un tableau du pool
    byte[] buffer = ArrayPool<byte>.Shared.Rent(4096);
    try
    {
        int bytesRead;
        while ((bytesRead = source.Read(buffer, 0, buffer.Length)) > 0)
        {
            // Traiter les données...
            ProcessBuffer(buffer, bytesRead);
        }
    }
    finally
    {
        // Important: retourner le tableau au pool
        ArrayPool<byte>.Shared.Return(buffer);
    }
}
```

### Création d'un pool d'objets personnalisé

Vous pouvez également créer votre propre système de pooling pour des besoins spécifiques.

**Exemple simple:**

```csharp
public class SimpleObjectPool<T> where T : class, new()
{
    private readonly Stack<T> _objects;
    private readonly int _maxSize;
    private readonly Action<T> _resetAction;

    public SimpleObjectPool(int maxSize, Action<T> resetAction = null)
    {
        _maxSize = maxSize;
        _objects = new Stack<T>(maxSize);
        _resetAction = resetAction;
    }

    public T Get()
    {
        lock (_objects)
        {
            if (_objects.Count > 0)
            {
                return _objects.Pop();
            }
        }

        // Créer un nouvel objet si le pool est vide
        return new T();
    }

    public void Return(T item)
    {
        if (item == null) throw new ArgumentNullException(nameof(item));

        // Réinitialiser l'objet si nécessaire
        _resetAction?.Invoke(item);

        lock (_objects)
        {
            // Ne pas dépasser la taille maximale du pool
            if (_objects.Count < _maxSize)
            {
                _objects.Push(item);
            }
            // Si le pool est plein, l'objet sera collecté par le GC
        }
    }
}

// Exemple d'utilisation
class Program
{
    static void Main()
    {
        var personPool = new SimpleObjectPool<Person>(100, p =>
        {
            p.Name = null;
            p.Age = 0;
        });

        // Emprunter une personne du pool
        var person = personPool.Get();
        try
        {
            person.Name = "Alice";
            person.Age = 30;
            // Utiliser la personne...
        }
        finally
        {
            // Retourner la personne au pool
            personPool.Return(person);
        }
    }
}

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

### Bonnes pratiques pour le pooling d'objets

1. **Toujours retourner les objets au pool** (utilisez `try`/`finally`)
2. **Réinitialiser les objets** avant de les retourner au pool
3. **Limiter la taille du pool** pour éviter la surconsommation de mémoire
4. **Pooler seulement les objets coûteux à créer** ou fréquemment utilisés
5. **Attention aux objets qui contiennent des ressources** (fichiers, connexions, etc.)

### Cas d'utilisation courants

Le pooling est particulièrement utile pour:

- Tableaux et buffers de grande taille
- Objets complexes coûteux à initialiser
- Objets utilisés fréquemment dans des applications à haute performance
- Connexions réseau et bases de données
- Threads et tâches (via ThreadPool)

## Conclusion

La gestion efficace de la mémoire est essentielle pour développer des applications C# performantes. En comprenant la différence entre types valeur et types référence, en utilisant judicieusement Span<T> et Memory<T>, en évitant les allocations inutiles et en implémentant des stratégies de pooling d'objets quand c'est approprié, vous pouvez considérablement améliorer les performances de vos applications.

Ces optimisations sont particulièrement importantes dans:
- Les applications à haute performance
- Les systèmes temps réel
- Les applications traitant de grands volumes de données
- Les services web à fort trafic
- Les applications mobiles avec ressources limitées

Cependant, n'oubliez pas que l'optimisation prématurée est souvent contre-productive. Mesurez d'abord les performances, identifiez les goulots d'étranglement, puis appliquez ces techniques là où elles auront le plus d'impact.

⏭️ 17.2. [Garbage Collector](/17-performances-et-optimisation/17-2-garbage-collector.md)
