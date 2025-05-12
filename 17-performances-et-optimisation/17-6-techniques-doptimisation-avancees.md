# 17.6. Techniques d'optimisation avancées en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'optimisation des performances est parfois nécessaire pour atteindre les objectifs de votre application. Ce chapitre explore des techniques avancées qui peuvent vous aider à repousser les limites de performance en C#. Bien que ces techniques soient puissantes, elles introduisent également des complexités et des risques. Par conséquent, il est important de les utiliser judicieusement et seulement lorsque les approches standard ne suffisent pas.

> **Important** : Mesurez toujours les performances avant et après l'optimisation pour vous assurer que les changements apportent réellement une amélioration. L'optimisation prématurée peut compliquer votre code sans bénéfice tangible.

## 17.6.1. Compilation JIT vs AOT

Pour comprendre comment optimiser efficacement votre code C#, il est essentiel de comprendre comment il est compilé et exécuté.

### Comment fonctionne la compilation en .NET

Le processus de compilation en .NET se déroule en deux étapes principales :

1. **Compilation en IL (Intermediate Language)** : Le compilateur C# convertit votre code source en un langage intermédiaire (IL, également appelé MSIL ou CIL).
2. **Compilation en code machine** : Ensuite, ce code IL est transformé en code machine natif qui peut être exécuté directement par le processeur.

La deuxième étape peut être réalisée de deux façons différentes : JIT (Just-In-Time) ou AOT (Ahead-Of-Time).

### Compilation JIT (Just-In-Time)

La compilation JIT est l'approche traditionnelle de .NET :

```
[Code C#] → [Compilation] → [Code IL dans un assembly .dll/.exe] → [Compilation JIT lors de l'exécution] → [Code machine]
```

#### Comment ça fonctionne

1. Lorsque votre application s'exécute, le runtime .NET charge les assemblies (fichiers .dll ou .exe).
2. Dès qu'une méthode est appelée pour la première fois, le compilateur JIT la compile en code machine.
3. Ce code machine est ensuite conservé en mémoire et réutilisé pour les appels ultérieurs.

#### Avantages de la compilation JIT

- **Portabilité** : Le même code IL peut être exécuté sur différentes architectures.
- **Optimisations spécifiques à la machine** : Le JIT peut optimiser le code pour le processeur exact sur lequel il s'exécute.
- **Optimisations basées sur l'exécution** : Le JIT peut optimiser le code en fonction des données réelles pendant l'exécution.

#### Inconvénients de la compilation JIT

- **Démarrage plus lent** : La compilation JIT prend du temps, ce qui peut ralentir le démarrage de l'application.
- **Utilisation de mémoire supplémentaire** : Le code IL et le code machine compilé doivent résider en mémoire.
- **Le niveau d'optimisation est limité** par le temps disponible pour la compilation.

### Compilation AOT (Ahead-Of-Time)

La compilation AOT compile tout le code en code machine avant l'exécution de l'application :

```
[Code C#] → [Compilation] → [Code IL] → [Compilation AOT] → [Code machine natif] → [Exécution directe]
```

#### Technologies de compilation AOT en .NET

1. **Native AOT** (disponible dans .NET 7+)
2. **ReadyToRun** (R2R)
3. **Xamarin AOT** (pour les applications mobiles)
4. **Mono AOT** (historiquement pour Mono)

#### Comment utiliser Native AOT

Pour utiliser Native AOT dans un projet .NET (à partir de .NET 7) :

1. Modifiez votre fichier de projet (.csproj) pour ajouter :

```xml
<PropertyGroup>
    <PublishAot>true</PublishAot>
</PropertyGroup>
```

2. Publiez votre application avec la commande :

```bash
dotnet publish -r win-x64 -c Release
```

3. Le résultat sera un exécutable natif optimisé pour la plateforme cible.

#### Limitations de Native AOT

- Pas de chargement dynamique d'assemblies
- Pas de génération dynamique de code (comme Expression Trees complexes)
- Réflexion limitée
- Incompatibilité avec certaines bibliothèques tierces

#### Avantages de la compilation AOT

- **Démarrage plus rapide** : Pas de compilation JIT au démarrage
- **Consommation mémoire réduite** : Le code IL n'a pas besoin d'être chargé
- **Optimisations plus poussées** : Plus de temps pour appliquer des optimisations
- **Déploiement autonome** : L'exécutable contient tout ce dont il a besoin

#### Inconvénients de la compilation AOT

- **Taille des fichiers plus grande** : Les exécutables sont plus volumineux
- **Portabilité réduite** : Compilation spécifique à une plateforme
- **Flexibilité réduite** : Certaines fonctionnalités dynamiques sont limitées

### Quand utiliser JIT vs AOT ?

#### Utilisez JIT (approche par défaut) quand :

- Vous avez besoin d'une flexibilité maximale
- La taille du déploiement est une préoccupation
- Vous utilisez des fonctionnalités dynamiques (réflexion, génération de code à la volée, etc.)
- Vous voulez minimiser la complexité du déploiement

#### Utilisez AOT quand :

- Le temps de démarrage est critique (microservices, applications CLI, etc.)
- L'empreinte mémoire doit être minimisée
- Vous développez pour des plateformes avec des ressources limitées
- Vous créez des applications qui s'exécutent souvent mais brièvement

### Exemple pratique : Comparaison JIT vs AOT

Voici un exemple simple pour comparer les temps de démarrage :

```csharp
// Program.cs
using System;
using System.Diagnostics;

class Program
{
    static void Main()
    {
        // Pour mesurer uniquement le temps d'exécution après le démarrage
        Stopwatch sw = Stopwatch.StartNew();

        // Simuler du travail
        CalculerFibonacci(40);

        sw.Stop();
        Console.WriteLine($"Temps d'exécution: {sw.ElapsedMilliseconds}ms");
    }

    static long CalculerFibonacci(int n)
    {
        if (n <= 1) return n;
        return CalculerFibonacci(n - 1) + CalculerFibonacci(n - 2);
    }
}
```

Compilez et exécutez ce programme en mode JIT standard, puis avec Native AOT. Vous remarquerez probablement un démarrage plus rapide avec la version AOT, mais les performances d'exécution seront similaires une fois le programme démarré.

## 17.6.2. Unsafe code et pointeurs

C# est généralement un langage "sûr" qui vous protège contre les erreurs de manipulation de mémoire. Cependant, pour des raisons de performance, C# permet d'utiliser du code "unsafe" qui vous donne un accès direct à la mémoire via des pointeurs, similaire à des langages comme C ou C++.

### Qu'est-ce que le code unsafe ?

Le code unsafe permet :
- D'utiliser des pointeurs pour accéder directement à la mémoire
- De manipuler des blocs de mémoire sans les protections habituelles du runtime .NET
- D'interopérer efficacement avec du code natif

### Comment utiliser le code unsafe

Pour utiliser du code unsafe, vous devez :

1. Marquer le bloc de code, la méthode ou la classe avec le mot-clé `unsafe`
2. Compiler votre code avec l'option `/unsafe` ou ajouter `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>` dans votre fichier .csproj

```csharp
// Dans le fichier .csproj
<PropertyGroup>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```

#### Exemple simple de code unsafe

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 10;

        // Bloc de code unsafe
        unsafe
        {
            // Obtenir un pointeur vers la variable x
            int* pX = &x;

            // Modifier la valeur via le pointeur
            *pX = 20;

            Console.WriteLine($"Valeur de x: {x}");  // Affiche 20

            // Arithmétique de pointeurs
            int[] nombres = { 1, 2, 3, 4, 5 };

            // Fixer le tableau en mémoire pour éviter qu'il ne soit déplacé par le GC
            fixed (int* pNombres = nombres)
            {
                // Accéder aux éléments via des pointeurs
                for (int i = 0; i < nombres.Length; i++)
                {
                    Console.WriteLine($"nombres[{i}] = {*(pNombres + i)}");
                }

                // Modifier un élément
                *(pNombres + 2) = 30;
            }

            Console.WriteLine($"nombres[2] = {nombres[2]}");  // Affiche 30
        }
    }
}
```

### Utilisation de stackalloc

`stackalloc` permet d'allouer de la mémoire directement sur la pile plutôt que sur le tas, ce qui peut être plus rapide pour de petites allocations temporaires :

```csharp
unsafe
{
    // Allouer un tableau de 10 ints sur la pile
    int* buffer = stackalloc int[10];

    // Initialiser les valeurs
    for (int i = 0; i < 10; i++)
    {
        buffer[i] = i * i;
    }

    // Utiliser les valeurs
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine($"buffer[{i}] = {buffer[i]}");
    }

    // Pas besoin de libérer la mémoire - elle est automatiquement
    // libérée quand la méthode se termine
}
```

### Exemple pratique : copie de mémoire rapide

Un cas d'utilisation courant du code unsafe est la copie de mémoire performante :

```csharp
using System;

class Program
{
    static void Main()
    {
        const int taille = 10_000_000;
        int[] source = new int[taille];
        int[] destination = new int[taille];

        // Initialiser le tableau source
        for (int i = 0; i < taille; i++)
        {
            source[i] = i;
        }

        // Mesurer la méthode sûre (standard)
        var sw = System.Diagnostics.Stopwatch.StartNew();
        CopieSûre(source, destination);
        sw.Stop();
        Console.WriteLine($"Copie sûre: {sw.ElapsedMilliseconds}ms");

        // Mesurer la méthode unsafe
        sw.Restart();
        CopieUnsafe(source, destination);
        sw.Stop();
        Console.WriteLine($"Copie unsafe: {sw.ElapsedMilliseconds}ms");

        // Vérifier que la copie est correcte
        Console.WriteLine($"Première valeur: {destination[0]}, Dernière valeur: {destination[taille-1]}");
    }

    static void CopieSûre(int[] source, int[] destination)
    {
        for (int i = 0; i < source.Length; i++)
        {
            destination[i] = source[i];
        }
    }

    static unsafe void CopieUnsafe(int[] source, int[] destination)
    {
        fixed (int* pSource = source, pDest = destination)
        {
            // Copier les octets directement
            int longueur = source.Length;

            for (int i = 0; i < longueur; i++)
            {
                pDest[i] = pSource[i];
            }

            // Alternative encore plus rapide pour de grands tableaux:
            // Buffer.MemoryCopy(pSource, pDest, destination.Length * sizeof(int), source.Length * sizeof(int));
        }
    }
}
```

### Risques et précautions avec le code unsafe

Le code unsafe peut être dangereux si mal utilisé :

- **Débordements de mémoire** (buffer overflows) : Accès à la mémoire en dehors des limites d'un tableau
- **Références invalides** : Utilisation de pointeurs vers des objets qui ont été déplacés par le GC
- **Fuites de mémoire** : Allocation manuelle de mémoire sans libération
- **Corruption de mémoire** : Écriture dans des zones mémoire utilisées par d'autres parties du programme

#### Bonnes pratiques pour le code unsafe

1. **Limitez la portée** du code unsafe au minimum nécessaire
2. **Vérifiez toujours les limites** avant d'accéder à la mémoire via des pointeurs
3. **Utilisez toujours `fixed`** pour les tableaux et les chaînes
4. **Testez rigoureusement** le code unsafe sous diverses conditions
5. **Documentez clairement** pourquoi le code unsafe est nécessaire
6. **Isolez le code unsafe** dans des classes ou méthodes dédiées
7. **Envisagez des alternatives sûres** comme Span<T> avant de recourir au code unsafe

### Alternatives modernes au code unsafe

Des alternatives plus sûres sont disponibles dans les versions récentes de .NET :

- **Span<T>** et **Memory<T>** : Offrent des performances similaires sans code unsafe
- **System.Buffers** : Fournit des API optimisées pour les opérations sur les tampons
- **System.Numerics.Vector** : Pour les opérations SIMD sans code unsafe

## 17.6.3. Interop avec du code natif

L'interopérabilité (interop) avec du code natif permet à votre code C# d'appeler des fonctions écrites en C, C++ ou d'autres langages compilés en binaires natifs. Cette capacité est essentielle lorsque vous avez besoin :

1. D'accéder à des fonctionnalités du système d'exploitation
2. D'utiliser des bibliothèques tierces existantes
3. D'optimiser des portions critiques de votre application

### Platform Invoke (P/Invoke)

P/Invoke est le mécanisme principal pour appeler des fonctions natives à partir de code C# managé.

#### Comment utiliser P/Invoke

Le processus comporte généralement ces étapes :

1. Déclarer la fonction native avec l'attribut `[DllImport]`
2. Spécifier les types de paramètres et de retour
3. Appeler la fonction comme une méthode normale

```csharp
using System;
using System.Runtime.InteropServices;

class Program
{
    // Importer la fonction MessageBox de user32.dll
    [DllImport("user32.dll", CharSet = CharSet.Unicode)]
    static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);

    static void Main()
    {
        // Appeler la fonction native
        MessageBox(IntPtr.Zero, "Hello from P/Invoke!", "Native Call", 0);
    }
}
```

#### Marshalling (maréchalage) des données

Le marshalling est le processus de conversion des types entre le monde managé (.NET) et le monde natif.

```csharp
// Exemple de marshalling de différents types
[DllImport("NativeBibliotheque.dll")]
static extern void ProcessData(
    int simpleValue,               // Type simple - marshalling automatique
    [MarshalAs(UnmanagedType.LPStr)] string texte,  // Chaîne en tant que ANSI
    byte[] buffer,                 // Tableau marshallé comme pointeur
    ref int valeurParRéférence,    // Passage par référence
    out int résultat               // Paramètre de sortie
);
```

#### Structures pour l'interop

Les structures sont couramment utilisées pour passer des données complexes :

```csharp
// Définition d'une structure pour l'interop
[StructLayout(LayoutKind.Sequential)]
struct RECT
{
    public int Left;
    public int Top;
    public int Right;
    public int Bottom;
}

// Importer une fonction qui utilise cette structure
[DllImport("user32.dll")]
static extern bool GetWindowRect(IntPtr hWnd, out RECT rect);

// Utilisation
static void AfficherDimensionsFenêtre(IntPtr handle)
{
    RECT rect;
    if (GetWindowRect(handle, out rect))
    {
        int largeur = rect.Right - rect.Left;
        int hauteur = rect.Bottom - rect.Top;
        Console.WriteLine($"Dimensions de la fenêtre: {largeur}x{hauteur}");
    }
}
```

#### Callbacks natifs

Vous pouvez également passer des fonctions .NET comme callbacks à du code natif :

```csharp
// Définir le type de délégué qui correspond à la signature du callback
[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
delegate void CallbackDelegate(int valeur);

// Importer la fonction qui prend un callback
[DllImport("NativeBibliotheque.dll")]
static extern void RegisterCallback(CallbackDelegate callback);

// Méthode qui sera utilisée comme callback
static void MonCallback(int valeur)
{
    Console.WriteLine($"Callback appelé avec la valeur: {valeur}");
}

// Utilisation
static void Main()
{
    // Créer une instance du délégué et la passer à la fonction native
    CallbackDelegate callback = MonCallback;
    RegisterCallback(callback);

    // Important: conservez une référence au délégué tant que la fonction native peut l'appeler
    // GC.KeepAlive(callback);
}
```

### Exemple pratique : utilisation d'une bibliothèque cryptographique native

Imaginons que nous voulons utiliser une fonction de hachage d'une bibliothèque cryptographique native :

```csharp
using System;
using System.Runtime.InteropServices;
using System.Text;

class Program
{
    // Importer les fonctions de la bibliothèque cryptographique
    [DllImport("cryptolib.dll", CallingConvention = CallingConvention.Cdecl)]
    static extern int ComputeHash(byte[] data, int length, [Out] byte[] hash);

    [DllImport("cryptolib.dll", CallingConvention = CallingConvention.Cdecl)]
    static extern void FreeResources();

    static void Main()
    {
        try
        {
            string message = "Message à hacher";
            byte[] data = Encoding.UTF8.GetBytes(message);
            byte[] hash = new byte[32]; // Supposons que la fonction produit un hash de 32 octets

            int result = ComputeHash(data, data.Length, hash);

            if (result == 0)
            {
                Console.WriteLine("Hash calculé avec succès:");
                Console.WriteLine(BitConverter.ToString(hash).Replace("-", ""));
            }
            else
            {
                Console.WriteLine($"Erreur lors du calcul du hash: {result}");
            }
        }
        finally
        {
            // Libérer les ressources natives
            FreeResources();
        }
    }
}
```

### Optimisations et bonnes pratiques pour P/Invoke

1. **Minimisez les appels natifs** : Chaque appel P/Invoke a un coût de performance
2. **Minimisez le marshalling** : Passez des données en bloc plutôt qu'élément par élément
3. **Réutilisez les structures natives** : Créez-les une fois et réutilisez-les
4. **Utilisez `[In]` et `[Out]` correctement** pour optimiser le marshalling
5. **Libérez toujours les ressources natives** : Implémentez `IDisposable` si nécessaire
6. **Attention aux conventions d'appel** : Spécifiez la bonne convention (`CallingConvention`)
7. **Utilisez `StringBuilder` pour les chaînes en sortie** : Plus efficace que `string`

### Alternatives modernes à P/Invoke

Dans .NET 7+, les nouvelles API simplifient l'interop native :

```csharp
// À partir de .NET 7
using System.Runtime.InteropServices;

// Définition de l'interface pour la bibliothèque native
[LibraryImport("user32.dll", StringMarshalling = StringMarshalling.Utf16)]
public static partial int MessageBox(IntPtr hWnd, string text, string caption, uint type);

static void Main()
{
    // Appel simplifié
    MessageBox(IntPtr.Zero, "Hello from LibraryImport!", "Modern P/Invoke", 0);
}
```

Ce nouveau style génère du code plus efficace et réduit le risque d'erreurs.

## 17.6.4. Structs en pile vs tas

Les types valeur (`struct`) et référence (`class`) sont stockés différemment en mémoire, ce qui peut avoir un impact significatif sur les performances.

### Différences entre la pile et le tas

#### La pile (stack)

- Zone de mémoire à allocation/désallocation **rapide** (opération LIFO)
- Taille **limitée** (typiquement quelques Mo par thread)
- Alloue et libère la mémoire **automatiquement** selon l'exécution
- Utilisée pour les **variables locales** et les **types valeur**

#### Le tas (heap)

- Zone de mémoire plus **grande** mais plus **lente**
- Géré par le Garbage Collector
- Utilisé pour les **types référence** et les objets de **grande taille**
- L'allocation et la désallocation sont plus **coûteuses**

### Impact des struct vs class

```csharp
// Type référence (class) - stocké sur le tas
class PointClass
{
    public int X;
    public int Y;
}

// Type valeur (struct) - stocké directement
struct PointStruct
{
    public int X;
    public int Y;
}

static void Main()
{
    // Utilisation d'une classe
    PointClass pc = new PointClass(); // Allocation sur le tas
    pc.X = 10;
    pc.Y = 20;

    // Utilisation d'une struct
    PointStruct ps; // Pas de 'new' nécessaire, allocation sur la pile
    ps.X = 10;
    ps.Y = 20;

    // Créer et remplir un tableau (pour comparaison)
    const int count = 1_000_000;

    // Tableau de classes (références)
    var sw = System.Diagnostics.Stopwatch.StartNew();
    PointClass[] pointsClasse = new PointClass[count];
    for (int i = 0; i < count; i++)
    {
        pointsClasse[i] = new PointClass { X = i, Y = i };
    }
    sw.Stop();
    Console.WriteLine($"Création de {count} PointClass: {sw.ElapsedMilliseconds}ms");

    // Tableau de structs (valeurs)
    sw.Restart();
    PointStruct[] pointsStruct = new PointStruct[count];
    for (int i = 0; i < count; i++)
    {
        pointsStruct[i] = new PointStruct { X = i, Y = i };
    }
    sw.Stop();
    Console.WriteLine($"Création de {count} PointStruct: {sw.ElapsedMilliseconds}ms");
}
```

### Quand utiliser des structs pour les performances

Les structs sont généralement plus performants quand :

1. **L'objet est petit** (généralement < 16 octets)
2. **L'objet est de courte durée**
3. **L'objet est souvent transmis comme valeur**
4. **L'objet est immuable ou rarement modifié**
5. **Vous évitez le boxing/unboxing**

#### Exemple : Utilisation optimale des structs

```csharp
// Struct optimisée pour les opérations rapides sur des coordonnées
readonly struct Vector2D
{
    public readonly double X;
    public readonly double Y;

    public Vector2D(double x, double y)
    {
        X = x;
        Y = y;
    }

    // Méthodes qui créent de nouvelles instances plutôt que de modifier l'existante
    public Vector2D Add(Vector2D other) => new Vector2D(X + other.X, Y + other.Y);
    public Vector2D Multiply(double scalar) => new Vector2D(X * scalar, Y * scalar);

    public double Length => Math.Sqrt(X * X + Y * Y);
}

static void Main()
{
    // Utilisation sans boxing
    Vector2D v1 = new Vector2D(3, 4);
    Vector2D v2 = new Vector2D(1, 2);

    Vector2D v3 = v1.Add(v2);
    Console.WriteLine($"Longueur: {v3.Length}");

    // Comparaison des performances avec un grand nombre de vecteurs
    const int iterations = 10_000_000;

    var sw = System.Diagnostics.Stopwatch.StartNew();
    Vector2D result = new Vector2D(0, 0);

    for (int i = 0; i < iterations; i++)
    {
        result = result.Add(new Vector2D(1, 1));
    }

    sw.Stop();
    Console.WriteLine($"Opérations sur {iterations} vecteurs: {sw.ElapsedMilliseconds}ms");
    Console.WriteLine($"Résultat final: ({result.X}, {result.Y})");
}
```

### Éviter le boxing et unboxing

Le boxing (conversion d'un type valeur en type référence) et l'unboxing (l'inverse) sont coûteux en performances :

```csharp
// Exemple de boxing et unboxing
int nombre = 42;
object boxed = nombre;    // Boxing - copie l'entier sur le tas
int unboxed = (int)boxed; // Unboxing - extrait la valeur du tas

// Comparaison des performances
const int iterations = 10_000_000;

// Sans boxing
var sw = System.Diagnostics.Stopwatch.StartNew();
int somme1 = 0;
for (int i = 0; i < iterations; i++)
{
    somme1 += i;
}
sw.Stop();
Console.WriteLine($"Sans boxing: {sw.ElapsedMilliseconds}ms");

// Avec boxing
sw.Restart();
object somme2 = 0;
for (int i = 0; i < iterations; i++)
{
    somme2 = (int)somme2 + i; // Boxing et unboxing à chaque itération
}
sw.Stop();
Console.WriteLine($"Avec boxing: {sw.ElapsedMilliseconds}ms");
```

### Optimisations avec les ReadOnlyRef struct

À partir de C# 7.2, vous pouvez utiliser `ref struct` et `readonly ref struct` pour créer des types qui existent uniquement sur la pile et ne peuvent jamais être alloués sur le tas :

```csharp
// Struct qui ne peut exister que sur la pile
ref struct PointRefStruct
{
    public int X;
    public int Y;

    public PointRefStruct(int x, int y)
    {
        X = x;
        Y = y;
    }
}

// Version immuable (readonly)
readonly ref struct Vector3D
{
    public readonly float X;
    public readonly float Y;
    public readonly float Z;

    public Vector3D(float x, float y, float z)
    {
        X = x;
        Y = y;
        Z = z;
    }

    public float Length => (float)Math.Sqrt(X * X + Y * Y + Z * Z);
}

static void Main()
{
    // Utilisation de ref struct
    PointRefStruct prs = new PointRefStruct(10, 20);
    Console.WriteLine($"Point: ({prs.X}, {prs.Y})");

    // Les limitations suivantes s'appliquent:
    // - Ne peut pas être utilisé comme type de champ dans une classe
    // - Ne peut pas être boxé
    // - Ne peut pas être utilisé dans les lambdas ou méthodes async
    // - Ne peut pas être utilisé dans les méthodes Iterator (yield return)
}
```

## 17.6.5. SIMD avec Vector<T>

SIMD (Single Instruction, Multiple Data) est une technique qui permet d'effectuer la même opération sur plusieurs données simultanément. .NET propose des classes dans le namespace `System.Numerics` pour utiliser SIMD sans avoir recours au code unsafe.

### Qu'est-ce que SIMD ?

SIMD utilise des instructions spéciales du processeur pour traiter plusieurs valeurs à la fois :

```
Addition scalaire (non-SIMD):
[a] + [b] = [c]  // Une opération à la fois

Addition SIMD:
[a1, a2, a3, a4] + [b1, b2, b3, b4] = [c1, c2, c3, c4]  // Une instruction, quatre opérations
```

### Utilisation de Vector<T>

Pour utiliser SIMD en C#, ajoutez une référence au package `System.Numerics.Vectors` ou utilisez directement les types disponibles dans .NET Core et .NET 5+.

```csharp
using System;
using System.Numerics;

class Program
{
    static void Main()
    {
        Console.WriteLine($"Vector<float> peut contenir {Vector<float>.Count} éléments float");

        // Créer des vecteurs
        Vector<float> v1 = new Vector<float>(1.0f);  // Tous les éléments à 1.0
        float[] données = { 1.0f, 2.0f, 3.0f, 4.0f, 5.0f, 6.0f, 7.0f, 8.0f };
        Vector<float> v2 = new Vector<float>(données);  // Initialiser à partir d'un tableau

        // Opérations vectorielles
        Vector<float> somme = v1 + v2;
        Vector<float> produit = v1 * v2;
        Vector<float> max = Vector.Max(v1, v2);

        // Convertir le vecteur en tableau pour affichage
        float[] résultat = new float[Vector<float>.Count];
        somme.CopyTo(résultat);

        Console.WriteLine("Résultat de la somme:");
        foreach (var val in résultat)
        {
            Console.Write($"{val} ");
        }
        Console.WriteLine();
    }
}
```

### Exemple : Accélération de calculs avec SIMD

Voici un exemple qui compare le traitement scalaire traditionnel avec le traitement vectoriel SIMD pour une opération simple : multiplier chaque élément d'un tableau par une constante.

```csharp
using System;
using System.Diagnostics;
using System.Numerics;

class Program
{
    static void Main()
    {
        const int taille = 50_000_000;  // 50 millions d'éléments

        // Initialiser les données
        float[] tableau = new float[taille];
        float[] résultatScalaire = new float[taille];
        float[] résultatVectoriel = new float[taille];

        for (int i = 0; i < taille; i++)
        {
            tableau[i] = i % 100;  // Valeurs simples de 0 à 99
        }

        // Facteur de multiplication
        float facteur = 2.5f;

        // 1. Mesurer la méthode scalaire traditionnelle
        Stopwatch sw = Stopwatch.StartNew();
        MultiplierScalaire(tableau, résultatScalaire, facteur);
        sw.Stop();
        Console.WriteLine($"Méthode scalaire: {sw.ElapsedMilliseconds}ms");

        // 2. Mesurer la méthode vectorielle SIMD
        sw.Restart();
        MultiplierVectoriel(tableau, résultatVectoriel, facteur);
        sw.Stop();
        Console.WriteLine($"Méthode vectorielle: {sw.ElapsedMilliseconds}ms");

        // Vérifier que les résultats sont identiques
        bool identiques = true;
        for (int i = 0; i < 1000; i++)  // Vérifier juste les premiers éléments
        {
            if (Math.Abs(résultatScalaire[i] - résultatVectoriel[i]) > 0.0001f)
            {
                identiques = false;
                Console.WriteLine($"Différence à l'index {i}: {résultatScalaire[i]} vs {résultatVectoriel[i]}");
                break;
            }
        }
        Console.WriteLine($"Résultats identiques: {identiques}");
    }

    // Méthode traditionnelle (scalaire)
    static void MultiplierScalaire(float[] source, float[] destination, float facteur)
    {
        for (int i = 0; i < source.Length; i++)
        {
            destination[i] = source[i] * facteur;
        }
    }

    // Méthode utilisant SIMD (vectorielle)
    static void MultiplierVectoriel(float[] source, float[] destination, float facteur)
    {
        int vectorSize = Vector<float>.Count;
        Vector<float> facteurVect = new Vector<float>(facteur);

        // Traiter les éléments par blocs de la taille du vecteur
        int i = 0;
        for (; i <= source.Length - vectorSize; i += vectorSize)
        {
            // Charger un bloc de données dans un vecteur
            Vector<float> vectSource = new Vector<float>(source, i);

            // Appliquer l'opération vectorielle
            Vector<float> vectRésultat = vectSource * facteurVect;

            // Stocker le résultat
            vectRésultat.CopyTo(destination, i);
        }

        // Traiter les éléments restants (non alignés sur la taille du vecteur)
        for (; i < source.Length; i++)
        {
            destination[i] = source[i] * facteur;
        }
    }
}
```

### Avantages et limitations de SIMD

#### Avantages

- **Performances accrues** : Traite plusieurs éléments en une seule instruction
- **API moderne** : Abstraction des instructions matérielles spécifiques
- **Sécurité de type** : Utilisation plus sûre que le code unsafe

#### Limitations

- **Dépendance matérielle** : Les performances dépendent du CPU et des instructions disponibles
- **Types limités** : Fonctionne uniquement avec certains types numériques
- **Complexité accrue** : Le code peut être plus difficile à comprendre et à maintenir
- **Alignement mémoire** : Les gains de performance optimaux nécessitent un alignement mémoire correct

### Autres types vectoriels disponibles

Outre `Vector<T>`, .NET propose d'autres types SIMD spécialisés :

```csharp
// Vecteurs de taille fixe pour des cas d'utilisation spécifiques
Vector2 v2 = new Vector2(1.0f, 2.0f);             // Vecteur 2D float
Vector3 v3 = new Vector3(1.0f, 2.0f, 3.0f);       // Vecteur 3D float
Vector4 v4 = new Vector4(1.0f, 2.0f, 3.0f, 4.0f); // Vecteur 4D float

// Opérations 3D courantes
Vector3 a = new Vector3(1, 0, 0);
Vector3 b = new Vector3(0, 1, 0);
Vector3 produitVectoriel = Vector3.Cross(a, b);    // Produit vectoriel
float produitScalaire = Vector3.Dot(a, b);         // Produit scalaire
```

### Cas d'utilisation de SIMD

SIMD est particulièrement utile pour :

1. **Traitement d'images** : Opérations sur les pixels
2. **Calculs scientifiques** : Opérations sur de grands ensembles de données
3. **Jeux et graphismes 3D** : Transformations de vecteurs et matrices
4. **Machine learning** : Opérations sur des tenseurs et matrices

## 17.6.6. ReadOnlySpan et stackalloc

`Span<T>` et `ReadOnlySpan<T>` sont des types introduits dans .NET Core 2.1 qui permettent de manipuler efficacement des séquences de mémoire contiguë sans allouer de mémoire supplémentaire. Lorsqu'ils sont combinés avec `stackalloc`, ils offrent des performances exceptionnelles pour de nombreux scénarios.

### Introduction à Span<T>

`Span<T>` est un type valeur qui représente une séquence contiguë de valeurs de type `T`. Il peut référencer :

- Un tableau ou une partie de tableau
- Une chaîne de caractères (pour `Span<char>`)
- Une mémoire allouée sur la pile avec `stackalloc`
- Une mémoire non managée

```csharp
using System;

class Program
{
    static void Main()
    {
        // 1. Span sur un tableau existant
        int[] tableau = { 1, 2, 3, 4, 5 };
        Span<int> span = tableau;

        // 2. Span sur une portion de tableau
        Span<int> milieu = tableau.AsSpan(1, 3); // Éléments 2, 3, 4

        // 3. Modifier les valeurs via le span
        milieu[0] = 22; // Modifie tableau[1]

        Console.WriteLine("Tableau après modification:");
        foreach (var n in tableau)
        {
            Console.Write($"{n} ");
        }
        Console.WriteLine();
    }
}
```

### ReadOnlySpan<T>

`ReadOnlySpan<T>` est une version en lecture seule de `Span<T>`. Elle est particulièrement utile pour travailler avec des chaînes de caractères et autres données immuables.

```csharp
using System;

static void Main()
{
    string phrase = "Bonjour le monde";

    // Créer un ReadOnlySpan à partir d'une chaîne
    // (sans allouer de nouvelles chaînes)
    ReadOnlySpan<char> span = phrase.AsSpan();

    // Travailler avec des sous-sections
    ReadOnlySpan<char> premier = span.Slice(0, 7); // "Bonjour"
    Console.WriteLine($"Premier mot: {premier.ToString()}");

    // Itérer sur les caractères un par un
    for (int i = 0; i < span.Length; i++)
    {
        Console.Write($"{span[i]} ");
    }
    Console.WriteLine();
}
```

### stackalloc pour l'allocation sur la pile

`stackalloc` est un opérateur C# qui alloue un bloc de mémoire directement sur la pile. Combiné avec `Span<T>`, il devient plus sûr et plus facile à utiliser.

```csharp
using System;

static void Main()
{
    // Allouer un tableau d'entiers sur la pile
    Span<int> nombres = stackalloc int[10];

    // Initialiser les valeurs
    for (int i = 0; i < nombres.Length; i++)
    {
        nombres[i] = i * i;
    }

    // Afficher les valeurs
    Console.WriteLine("Nombres:");
    foreach (var n in nombres)
    {
        Console.Write($"{n} ");
    }
    Console.WriteLine();

    // Pas besoin de libérer la mémoire - elle est automatiquement
    // libérée quand la méthode se termine
}
```

### Exemple pratique : Parsing sans allocation

Voici un exemple qui utilise `ReadOnlySpan<char>` et `stackalloc` pour parser un fichier CSV sans allouer de chaînes temporaires :

```csharp
using System;

class Program
{
    static void Main()
    {
        // Simuler le contenu d'un fichier CSV
        string csvData = "Nom,Age,Ville\nAlice,30,Paris\nBob,25,Lyon\nCharlie,40,Marseille";

        // Parser sans créer de sous-chaînes
        ParseCsvSansAllocation(csvData);
    }

    static void ParseCsvSansAllocation(string csvData)
    {
        ReadOnlySpan<char> données = csvData.AsSpan();

        // Trouver l'en-tête
        int posFinLigne = données.IndexOf('\n');
        if (posFinLigne == -1) return;

        ReadOnlySpan<char> enTête = données.Slice(0, posFinLigne);
        données = données.Slice(posFinLigne + 1);

        // Afficher les infos d'en-tête
        Console.WriteLine("En-tête:");
        AfficherChamps(enTête);

        // Traiter chaque ligne
        while (!données.IsEmpty)
        {
            posFinLigne = données.IndexOf('\n');

            // Dernière ligne ou ligne intermédiaire
            ReadOnlySpan<char> ligne = posFinLigne == -1
                ? données
                : données.Slice(0, posFinLigne);

            if (!ligne.IsEmpty)
            {
                Console.WriteLine("\nLigne:");
                AfficherChamps(ligne);
            }

            // Passer à la ligne suivante s'il en reste
            if (posFinLigne == -1) break;
            données = données.Slice(posFinLigne + 1);
        }
    }

    static void AfficherChamps(ReadOnlySpan<char> ligne)
    {
        int position = 0;

        while (position < ligne.Length)
        {
            int posVirgule = ligne.Slice(position).IndexOf(',');

            ReadOnlySpan<char> champ = posVirgule == -1
                ? ligne.Slice(position)
                : ligne.Slice(position, posVirgule);

            Console.WriteLine($"  [{champ.ToString()}]");

            if (posVirgule == -1) break;
            position += posVirgule + 1;
        }
    }
}
```

### Exemple de manipulation de mémoire efficace

Voici un exemple qui montre comment utiliser `Span<T>` et `stackalloc` pour une manipulation efficace de la mémoire :

```csharp
using System;
using System.Buffers.Binary;

class Program
{
    static void Main()
    {
        // Simuler la lecture d'un paquet réseau binaire
        byte[] données = { 0x01, 0x00, 0x00, 0x00, 0xF3, 0x01, 0x00, 0x00, 0x42 };

        // Décoder le paquet sans allouer de mémoire supplémentaire
        DécoderPaquet(données.AsSpan());
    }

    static void DécoderPaquet(ReadOnlySpan<byte> données)
    {
        if (données.Length < 9)
        {
            Console.WriteLine("Paquet trop court");
            return;
        }

        // Lire un entier (4 premiers octets)
        int id = BinaryPrimitives.ReadInt32LittleEndian(données.Slice(0, 4));

        // Lire un autre entier (4 octets suivants)
        int valeur = BinaryPrimitives.ReadInt32LittleEndian(données.Slice(4, 4));

        // Lire un octet (dernier octet)
        byte drapeau = données[8];

        Console.WriteLine($"ID: {id}");
        Console.WriteLine($"Valeur: {valeur}");
        Console.WriteLine($"Drapeau: 0x{drapeau:X2}");

        // Traitement supplémentaire sans allocation
        Span<byte> tamponRéponse = stackalloc byte[12];

        // Remplir le tampon de réponse
        BinaryPrimitives.WriteInt32LittleEndian(tamponRéponse.Slice(0, 4), id);
        BinaryPrimitives.WriteInt32LittleEndian(tamponRéponse.Slice(4, 4), valeur * 2);
        BinaryPrimitives.WriteInt32LittleEndian(tamponRéponse.Slice(8, 4), 0x12345678);

        Console.WriteLine("Tampon de réponse généré:");
        foreach (var b in tamponRéponse)
        {
            Console.Write($"0x{b:X2} ");
        }
        Console.WriteLine();
    }
}
```

### Avantages de Span<T>/ReadOnlySpan<T> avec stackalloc

1. **Zéro allocation** : Pas d'allocation sur le tas, pas de pression sur le GC
2. **Performances** : Accès direct à la mémoire sans indirection
3. **Sécurité** : Protection contre les débordements de mémoire, contrairement au code unsafe pur
4. **Versatilité** : Fonctionne avec différentes sources de données (tableaux, chaînes, mémoire native)
5. **API moderne** : Compatible avec de nombreuses méthodes du framework .NET

### Limitations et précautions

Malgré leurs avantages, ces techniques ont quelques limitations importantes :

1. **Span<T> est un `ref struct`**
   - Ne peut pas être utilisé comme champ dans une classe
   - Ne peut pas être utilisé dans les lambdas async
   - Ne peut pas être boxé
   - Ne peut pas être utilisé dans les méthodes iterator (yield return)

2. **stackalloc a des limitations**
   - Limité à la taille de la pile (généralement quelques Mo)
   - Réservé aux petites allocations temporaires
   - Risque de stack overflow en cas d'utilisation excessive

3. **Considérations sur le cycle de vie**
   - Les Span ne peuvent pas survivre à la méthode qui les a créés
   - Les références deviennent invalides quand la méthode se termine

### Bonnes pratiques

1. **Utilisez `stackalloc` pour des allocations petites et temporaires** (généralement < 1 Ko)
2. **Préférez `Span<T>` ou `ReadOnlySpan<T>` à des tableaux ou sous-chaînes temporaires**
3. **Utilisez `Memory<T>` ou `ReadOnlyMemory<T>` pour stocker des références long terme**
4. **Attention aux opérations imbriquées** qui pourraient consommer beaucoup d'espace de pile
5. **Encapsulez les opérations complexes** dans des méthodes dédiées

## Résumé et recommandations

Les techniques d'optimisation avancées que nous avons explorées peuvent apporter des gains de performance significatifs, mais elles introduisent également de la complexité et des risques. Voici quelques recommandations finales :

### Approche méthodique d'optimisation

1. **Identifiez les goulots d'étranglement** avec des outils de profilage
2. **Mesurez avant d'optimiser** : établissez une référence de performance
3. **Commencez par les optimisations algorithmiques** avant de passer aux optimisations de bas niveau
4. **Optimisez par étapes**, en mesurant l'impact de chaque changement
5. **Documentez vos optimisations** et leurs raisons

### Choisir la bonne technique

Pour vous aider à choisir la technique appropriée, voici un guide simplifié :

| Si vous voulez... | Envisagez... |
|-------------------|--------------|
| Démarrage plus rapide | AOT compilation |
| Manipulation mémoire directe | Unsafe code / Span<T> |
| Utiliser des bibliothèques natives | P/Invoke |
| Objets de petite taille, haute performance | Structs / ref structs |
| Calculs numériques intensifs | SIMD avec Vector<T> |
| Manipuler des chaînes sans allocation | ReadOnlySpan<char> |
| Tableaux temporaires sans allocation | stackalloc avec Span<T> |

### Équilibrer performance et maintenabilité

N'oubliez pas que le code optimisé est souvent :
- Plus difficile à comprendre
- Plus difficile à déboguer
- Plus difficile à maintenir
- Plus propice aux erreurs subtiles

Utilisez ces techniques avancées judicieusement, et uniquement lorsque les bénéfices en performance justifient clairement la complexité accrue.

### Pour approfondir

Si vous souhaitez approfondir ces sujets, voici quelques ressources recommandées :

1. **Documentation Microsoft** sur les performances : https://docs.microsoft.com/fr-fr/dotnet/core/performance/
2. **High Performance .NET** de Mark Farragher
3. **Pro .NET Memory Management** d'Konrad Kokosa
4. **Writing High-Performance .NET Code** de Ben Watson
5. **Blog de Stephen Toub** sur les performances : https://devblogs.microsoft.com/dotnet/author/stoub/

Avec ces connaissances et techniques avancées, vous êtes maintenant mieux équipé pour optimiser vos applications C# lorsque les approches standard ne suffisent pas. N'oubliez pas : la meilleure optimisation est celle qui apporte le plus grand bénéfice tout en restant maintenable et compréhensible.

⏭️
