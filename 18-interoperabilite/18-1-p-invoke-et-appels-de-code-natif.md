# 18.1. P/Invoke et appels de code natif

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

Bienvenue dans ce chapitre consacré à P/Invoke, une fonctionnalité puissante mais parfois complexe de C# qui permet d'appeler du code natif (généralement écrit en C ou C++) directement depuis votre code .NET.

Vous vous demandez peut-être : "Pourquoi aurais-je besoin d'appeler du code natif alors que C# est si complet ?" Voici quelques situations où cela devient nécessaire :

- Accéder à des fonctionnalités du système d'exploitation (Windows API) non exposées par le framework .NET
- Utiliser des bibliothèques C/C++ existantes et éprouvées
- Optimiser des sections critiques de votre code pour la performance
- Interagir avec du matériel spécifique via des pilotes

Dans ce chapitre, nous allons explorer P/Invoke (Platform Invocation Services) étape par étape, en commençant par les bases et en progressant vers des techniques plus avancées.

## 18.1.1. Marshaling

### Qu'est-ce que le marshaling ?

Le marshaling est le processus qui permet de convertir les données entre les environnements managés (.NET) et non managés (code natif). C'est un aspect critique de P/Invoke car les types de données et les conventions de mémoire diffèrent entre ces deux mondes.

### Les bases de P/Invoke

Pour appeler une fonction native, nous utilisons l'attribut `[DllImport]` dans C#. Voici un exemple simple pour appeler la fonction `MessageBox` de l'API Windows :

```csharp
using System;
using System.Runtime.InteropServices;

class Program
{
    // Déclaration de la fonction native MessageBox de User32.dll
    [DllImport("user32.dll", CharSet = CharSet.Unicode)]
    public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);

    static void Main()
    {
        // Appel de la fonction native
        MessageBox(IntPtr.Zero, "Ceci est un message", "Titre de la boîte de dialogue", 0);
    }
}
```

Dans cet exemple :
- `[DllImport("user32.dll")]` indique que nous voulons appeler une fonction de la bibliothèque user32.dll
- `CharSet = CharSet.Unicode` précise que nous utilisons des chaînes Unicode
- `static extern` déclare que la méthode est implémentée à l'extérieur de notre code managé
- Les paramètres définissent l'interface entre notre code C# et la fonction native

### Correspondance des types de base

| Type C/C++ | Type C# |
|------------|---------|
| int        | int     |
| long       | int (Windows) ou long (Unix) |
| char*      | string (avec CharSet approprié) |
| void*      | IntPtr ou UIntPtr |
| HANDLE     | IntPtr  |
| BOOL       | bool ou int |
| DWORD      | uint    |
| BYTE       | byte    |

### Marshaling de chaînes de caractères

Les chaînes requièrent une attention particulière car elles sont représentées différemment en .NET (Unicode) et en code natif (qui peut utiliser ASCII, Unicode, ou UTF-8) :

```csharp
// Pour les chaînes ANSI (ASCII)
[DllImport("user32.dll", CharSet = CharSet.Ansi)]
public static extern int MessageBoxA(IntPtr hWnd, string text, string caption, uint type);

// Pour les chaînes Unicode
[DllImport("user32.dll", CharSet = CharSet.Unicode)]
public static extern int MessageBoxW(IntPtr hWnd, string text, string caption, uint type);

// Auto (sélectionne A ou W selon la plateforme)
[DllImport("user32.dll", CharSet = CharSet.Auto)]
public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);
```

### Marshaling de structures

Pour passer des structures entre les mondes managé et non managé, nous devons définir leur mise en page mémoire avec `[StructLayout]` :

```csharp
// Définit une structure qui correspond à RECT dans Win32 API
[StructLayout(LayoutKind.Sequential)]
public struct RECT
{
    public int Left;
    public int Top;
    public int Right;
    public int Bottom;
}

// Importe une fonction qui utilise cette structure
[DllImport("user32.dll")]
public static extern bool GetWindowRect(IntPtr hWnd, out RECT lpRect);

// Utilisation
static void AfficherDimensionsFenetre(IntPtr handle)
{
    RECT rect;
    if (GetWindowRect(handle, out rect))
    {
        int largeur = rect.Right - rect.Left;
        int hauteur = rect.Bottom - rect.Top;
        Console.WriteLine($"Largeur: {largeur}, Hauteur: {hauteur}");
    }
}
```

### Types d'arrangement de structures

- `LayoutKind.Sequential` : Les membres sont arrangés dans l'ordre de leur déclaration
- `LayoutKind.Explicit` : Vous définissez explicitement la position de chaque membre avec `[FieldOffset]`
- `LayoutKind.Auto` : Le runtime détermine l'arrangement (rarement utilisé avec P/Invoke)

Exemple avec un arrangement explicite :

```csharp
[StructLayout(LayoutKind.Explicit)]
public struct UNION_EXEMPLE
{
    [FieldOffset(0)] public int entier;
    [FieldOffset(0)] public float flottant;
    // Les deux champs partagent le même emplacement mémoire, comme une union en C
}
```

### Marshaling de tableaux

Pour les tableaux, plusieurs approches sont possibles :

```csharp
// Tableau en paramètre d'entrée
[DllImport("malib.dll")]
public static extern int TraiterDonnees([In] int[] donnees, int taille);

// Tableau en paramètre de sortie
[DllImport("malib.dll")]
public static extern int ObtenirDonnees([Out] int[] buffer, int taille);

// Tableau en entrée et sortie
[DllImport("malib.dll")]
public static extern int ModifierDonnees([In, Out] int[] donnees, int taille);
```

Les attributs `[In]` et `[Out]` optimisent le marshaling en indiquant la direction des données.

## 18.1.2. Callbacks

Les callbacks permettent au code natif d'appeler des fonctions managées, créant ainsi une communication bidirectionnelle.

### Définition et utilisation d'un callback

```csharp
// Définition du type de délégué qui sera utilisé comme callback
// Doit correspondre exactement à la signature attendue par le code natif
[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
public delegate bool EnumWindowsProc(IntPtr hwnd, IntPtr lParam);

// Importation de la fonction qui prend un callback
[DllImport("user32.dll")]
public static extern bool EnumWindows(EnumWindowsProc enumFunc, IntPtr lParam);

// Implémentation du callback
public static bool ListeFenetre(IntPtr hwnd, IntPtr lParam)
{
    // Obtient le titre de la fenêtre
    int taille = GetWindowTextLength(hwnd);
    if (taille > 0)
    {
        StringBuilder sb = new StringBuilder(taille + 1);
        GetWindowText(hwnd, sb, sb.Capacity);
        Console.WriteLine($"Fenêtre: {sb.ToString()}");
    }
    return true; // Continuer l'énumération
}

// Fonctions auxiliaires
[DllImport("user32.dll")]
static extern int GetWindowTextLength(IntPtr hWnd);

[DllImport("user32.dll", CharSet = CharSet.Auto)]
static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);

// Utilisation
static void ListerFenetres()
{
    EnumWindows(ListeFenetre, IntPtr.Zero);
}
```

### Points importants pour les callbacks

1. Utilisez `[UnmanagedFunctionPointer]` pour spécifier la convention d'appel
2. Gardez une référence forte au délégué tant que le code natif peut l'appeler (pour éviter la collecte des déchets)
3. Attention aux opérations bloquantes dans les callbacks

### Préserver le délégué de la collecte des déchets

```csharp
class ExempleCallback
{
    // Stockage du délégué au niveau de la classe pour le préserver
    private static EnumWindowsProc _callback;

    public static void DemarrerEnumeration()
    {
        // Initialisation et stockage du délégué
        _callback = new EnumWindowsProc(ListeFenetre);

        // Passage du délégué à la fonction native
        EnumWindows(_callback, IntPtr.Zero);
    }

    // Implémentation du callback
    private static bool ListeFenetre(IntPtr hwnd, IntPtr lParam)
    {
        // Traitement...
        return true;
    }
}
```

## 18.1.3. Gestion des erreurs

La gestion des erreurs lors de l'utilisation de P/Invoke est cruciale car les fonctions natives ne lancent pas d'exceptions .NET.

### Codes de retour

La plupart des fonctions Windows API utilisent des codes de retour pour indiquer le succès ou l'échec :

```csharp
[DllImport("kernel32.dll", SetLastError = true)]
static extern bool CopyFile(string lpExistingFileName, string lpNewFileName, bool bFailIfExists);

public static void CopierFichier(string source, string destination)
{
    bool succes = CopyFile(source, destination, false);
    if (!succes)
    {
        // Récupère le code d'erreur
        int erreur = Marshal.GetLastWin32Error();
        throw new System.ComponentModel.Win32Exception(erreur);
    }
}
```

L'attribut `SetLastError = true` indique que la fonction utilise SetLastError pour signaler des erreurs.

### Win32Exception

La classe `System.ComponentModel.Win32Exception` est très utile car elle convertit automatiquement les codes d'erreur Windows en messages compréhensibles :

```csharp
try
{
    CopierFichier(@"C:\source.txt", @"C:\destination.txt");
}
catch (System.ComponentModel.Win32Exception ex)
{
    Console.WriteLine($"Erreur lors de la copie: {ex.Message} (Code: {ex.NativeErrorCode})");
}
```

### Codes d'erreur personnalisés

Pour les bibliothèques qui utilisent leurs propres codes d'erreur (et non SetLastError) :

```csharp
[DllImport("malib.dll")]
static extern int MaFonction(string param);

public static void ExecuterFonction(string param)
{
    int resultat = MaFonction(param);
    if (resultat != 0) // 0 indique généralement le succès
    {
        string message;
        switch (resultat)
        {
            case 1: message = "Paramètre invalide"; break;
            case 2: message = "Ressource non disponible"; break;
            default: message = $"Erreur inconnue (code: {resultat})"; break;
        }
        throw new Exception(message);
    }
}
```

## 18.1.4. Optimisation des performances

P/Invoke implique une surcharge due au marshaling des données. Voici comment optimiser les performances :

### Minimiser les appels P/Invoke

Chaque appel à travers la frontière managé/non-managé a un coût. Essayez de :

- Regrouper plusieurs opérations en un seul appel natif
- Traiter les données par lots plutôt qu'élément par élément

### Réutilisation des structures

Pour éviter des allocations répétées :

```csharp
class GestionnaireAPI
{
    // Réutilisation d'une structure pour plusieurs appels
    private RECT _rectTemporaire = new RECT();

    public (int largeur, int hauteur) ObtenirDimensions(IntPtr fenetre)
    {
        if (GetWindowRect(fenetre, out _rectTemporaire))
        {
            return (_rectTemporaire.Right - _rectTemporaire.Left,
                    _rectTemporaire.Bottom - _rectTemporaire.Top);
        }
        return (0, 0);
    }
}
```

### Utilisation de blittable types

Les "blittable types" sont des types qui ont la même représentation en mémoire en code managé et non managé. Ils sont plus efficaces car ils n'ont pas besoin d'être convertis :

Types blittable :
- byte, sbyte, short, ushort, int, uint, long, ulong, float, double
- Structures composées uniquement de types blittable et avec LayoutKind.Sequential ou LayoutKind.Explicit

```csharp
[StructLayout(LayoutKind.Sequential)]
public struct PointBlittable
{
    public int X;
    public int Y;
    // Structure efficace pour le marshaling car tous les champs sont blittable
}
```

### Préallocation de mémoire pour les chaînes

Pour les opérations répétées avec des chaînes de caractères :

```csharp
[DllImport("kernel32.dll", CharSet = CharSet.Unicode)]
static extern int GetModuleFileName(IntPtr hModule, StringBuilder lpFilename, int nSize);

public static string ObtenirCheminModule(IntPtr module)
{
    // Préallocation d'un StringBuilder pour éviter les allocations dynamiques
    StringBuilder sb = new StringBuilder(260); // MAX_PATH
    GetModuleFileName(module, sb, sb.Capacity);
    return sb.ToString();
}
```

### In-lining des petites structures avec les paramètres

```csharp
// Moins efficace : deux allocations distinctes
[DllImport("user32.dll")]
static extern bool SetCursorPos(int X, int Y);

// Plus efficace : structure sur la pile
[StructLayout(LayoutKind.Sequential)]
struct POINT
{
    public int X;
    public int Y;
}

[DllImport("user32.dll")]
static extern bool SetCursorPos(POINT point);
```

### Utilisation de 'unsafe' et pointeurs directs

Pour une performance maximale dans les sections critiques :

```csharp
// Nécessite que le projet soit compilé avec l'option "unsafe"
[DllImport("msvcrt.dll", EntryPoint = "memcpy", CallingConvention = CallingConvention.Cdecl)]
public static extern unsafe void* MemoryCopy(void* dest, void* src, int count);

public static unsafe void CopierMemoire(byte[] destination, byte[] source, int longueur)
{
    fixed (byte* pDest = destination)
    fixed (byte* pSrc = source)
    {
        MemoryCopy(pDest, pSrc, longueur);
    }
}
```

Le code unsafe permet de contourner les vérifications de type et de limite du runtime, mais nécessite des précautions supplémentaires.

## 18.1.5. LibraryImport (C# 11+)

Avec C# 11 et .NET 7+, Microsoft a introduit une nouvelle façon de faire des appels natifs avec l'attribut `[LibraryImport]`, qui est conçu pour remplacer `[DllImport]` avec de meilleures performances et une syntaxe plus moderne.

### Avantages de LibraryImport

- Performances améliorées grâce à la génération de code source
- Vérification de type plus stricte
- Simplification du marshaling
- Meilleure inférence des types

### Exemple de base

```csharp
using System.Runtime.InteropServices;
using System.Runtime.InteropServices.Marshalling;

class Program
{
    // L'ancienne façon avec DllImport
    [DllImport("user32.dll")]
    private static extern int MessageBoxA(IntPtr hWnd, string lpText, string lpCaption, uint uType);

    // La nouvelle façon avec LibraryImport
    [LibraryImport("user32.dll", StringMarshalling = StringMarshalling.Utf8)]
    private static partial int MessageBoxA(IntPtr hWnd, string lpText, string lpCaption, uint uType);

    static void Main()
    {
        MessageBoxA(IntPtr.Zero, "Message", "Titre", 0);
    }
}
```

Notez l'utilisation de `partial` - c'est parce que `LibraryImport` fonctionne avec les générateurs de source pour créer le code de marshaling à la compilation.

### Spécification du marshaling de chaînes

```csharp
// Spécifie explicitement l'encodage des chaînes
[LibraryImport("user32.dll", StringMarshalling = StringMarshalling.Utf8)]
private static partial int MessageBoxA(IntPtr hWnd, string lpText, string lpCaption, uint uType);

// Utilise la convention Custom pour un marshaling personnalisé
[LibraryImport("malib.dll", StringMarshalling = StringMarshalling.Custom,
    StringMarshallingCustomType = typeof(MyCustomStringMarshaller))]
private static partial void MaFonction(string texte);
```

### Structures avec LibraryImport

```csharp
// Structure pour LibraryImport
[NativeMarshalling(typeof(RectMarshaller))]
public struct RectManaged
{
    public int Left;
    public int Top;
    public int Right;
    public int Bottom;

    // Peut contenir des propriétés calculées, méthodes, etc.
    public int Width => Right - Left;
    public int Height => Bottom - Top;
}

// Marshaller personnalisé pour la structure
[CustomMarshaller(typeof(RectManaged), MarshalMode.ManagedToUnmanagedIn, typeof(RectMarshaller))]
public static class RectMarshaller
{
    // Structure native (à l'identique de la version Win32)
    public struct RectNative
    {
        public int Left;
        public int Top;
        public int Right;
        public int Bottom;
    }

    // Conversion managed -> native
    public static RectNative ConvertToUnmanaged(RectManaged managed)
    {
        return new RectNative
        {
            Left = managed.Left,
            Top = managed.Top,
            Right = managed.Right,
            Bottom = managed.Bottom
        };
    }

    // Méthode pour obtenir un pointeur vers la structure native
    public static RectNative* AllocateContainerForUnmanaged(RectManaged managed)
    {
        RectNative* ptr = (RectNative*)Marshal.AllocHGlobal(sizeof(RectNative));
        *ptr = ConvertToUnmanaged(managed);
        return ptr;
    }

    // Nettoyage
    public static void FreeUnmanagedContainer(RectNative* ptr)
    {
        Marshal.FreeHGlobal((IntPtr)ptr);
    }
}

// Déclaration de la fonction avec LibraryImport
[LibraryImport("user32.dll")]
private static partial bool GetWindowRect(IntPtr hWnd, out RectManaged lpRect);
```

### Callbacks avec LibraryImport

```csharp
// Définition du type de callback
[UnmanagedFunctionPointer(CallingConvention.StdCall)]
public delegate bool EnumWindowsDelegate(IntPtr hwnd, IntPtr lParam);

// Fonction qui utilise ce callback
[LibraryImport("user32.dll")]
private static partial bool EnumWindows(EnumWindowsDelegate lpEnumFunc, IntPtr lParam);

// Stockage du callback pour éviter la collecte des déchets
private static EnumWindowsDelegate _callback;

public static void EnumererFenetres()
{
    _callback = (hwnd, lParam) =>
    {
        Console.WriteLine($"Trouvé fenêtre: {hwnd}");
        return true; // continuer l'énumération
    };

    EnumWindows(_callback, IntPtr.Zero);
}
```

## Conclusion

P/Invoke est un outil puissant qui ouvre votre code C# au vaste écosystème de bibliothèques natives et d'APIs système. Bien que le marshaling entre les mondes managé et non managé puisse sembler complexe au premier abord, les techniques présentées dans ce chapitre vous permettront d'utiliser P/Invoke efficacement et en toute sécurité.

Quelques points à retenir :
- Utilisez la bonne correspondance de types entre C# et le code natif
- Portez une attention particulière au marshaling des chaînes et des structures
- Gérez correctement les erreurs avec SetLastError et GetLastWin32Error
- Optimisez les performances en minimisant les appels et le marshaling
- Considérez LibraryImport pour les nouveaux projets sur .NET 7+

## Exercices pratiques

1. Créez une application qui utilise l'API Windows pour obtenir la liste des processus en cours d'exécution
2. Écrivez un wrapper C# pour une fonction d'une bibliothèque C qui manipule des tableaux
3. Implémentez un callback qui est appelé depuis du code natif
4. Convertissez un exemple DllImport en LibraryImport et comparez les performances

## Ressources additionnelles

- [Documentation Microsoft sur P/Invoke](https://docs.microsoft.com/fr-fr/dotnet/standard/native-interop/pinvoke)
- [P/Invoke.net](http://www.pinvoke.net/) - Une ressource communautaire avec des milliers de signatures P/Invoke
- [Livre "Windows via C/C++" de Jeffrey Richter](https://www.amazon.fr/Windows-via-C-Jeffrey-Richter/dp/0735624240)

⏭️ 18.2. [COM Interop](/18-interoperabilite/18-2-com-interop.md)
