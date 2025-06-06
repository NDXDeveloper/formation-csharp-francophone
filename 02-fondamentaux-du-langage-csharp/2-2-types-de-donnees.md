# 2.2 Types de données en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les types de données sont un concept fondamental en programmation. Ils définissent la nature des données que nous manipulons et déterminent comment ces données sont stockées en mémoire. En C#, comprendre les différents types de données est essentiel pour écrire du code efficace et éviter les erreurs.

## 2.2.1 Types primitifs (int, float, bool, etc.)

Les types primitifs (ou types simples) sont les blocs de construction fondamentaux pour stocker des données en C#. Ils représentent des valeurs simples comme des nombres, des caractères ou des valeurs booléennes.

### Types numériques entiers

Ces types stockent des nombres entiers (sans partie décimale) :

| Type | Taille (octets) | Plage de valeurs | Utilisation |
|------|----------------|------------------|-------------|
| `sbyte` | 1 | -128 à 127 | Très petits nombres entiers (signés) |
| `byte` | 1 | 0 à 255 | Très petits nombres entiers (non signés) |
| `short` | 2 | -32 768 à 32 767 | Petits nombres entiers |
| `ushort` | 2 | 0 à 65 535 | Petits nombres entiers (non signés) |
| `int` | 4 | -2 147 483 648 à 2 147 483 647 | Nombres entiers (usage courant) |
| `uint` | 4 | 0 à 4 294 967 295 | Nombres entiers (non signés) |
| `long` | 8 | -9 223 372 036 854 775 808 à 9 223 372 036 854 775 807 | Très grands nombres entiers |
| `ulong` | 8 | 0 à 18 446 744 073 709 551 615 | Très grands nombres entiers (non signés) |

```csharp
int age = 25;
long populationMondiale = 8000000000L; // Le L indique un littéral de type long
byte octet = 255;
```

### Types numériques à virgule flottante

Ces types stockent des nombres avec une partie décimale :

| Type | Taille (octets) | Précision | Utilisation |
|------|----------------|-----------|-------------|
| `float` | 4 | ~7 chiffres | Précision simple (valeurs approchées) |
| `double` | 8 | ~15-16 chiffres | Précision double (valeurs approchées) |
| `decimal` | 16 | 28-29 chiffres | Haute précision (calculs financiers) |

```csharp
float distance = 23.5f;      // Le f indique un littéral de type float
double pi = 3.14159265359;   // Par défaut, les décimaux sont des double
decimal prix = 19.99m;       // Le m indique un littéral de type decimal
```

> **🔍 Note pour les débutants:**
> Les types `float` et `double` peuvent avoir des problèmes de précision avec certaines valeurs décimales (comme 0.1 ou 0.2). Pour les calculs financiers ou lorsque la précision est critique, utilisez toujours `decimal`.

### Type booléen

Le type `bool` représente une valeur logique qui peut être soit vraie (`true`), soit fausse (`false`) :

```csharp
bool estMajeur = true;
bool estConnecte = false;
```

### Type caractère

Le type `char` représente un caractère Unicode unique (lettre, chiffre, symbole, etc.) :

```csharp
char premiereLettreAlphabet = 'a';
char symboleEtoile = '*';
char lettreMajuscule = 'A';
```

Les caractères sont entourés de guillemets simples (`'`), contrairement aux chaînes de caractères qui utilisent des guillemets doubles (`"`).

### Littéraux et préfixes/suffixes

Pour indiquer explicitement le type d'un littéral, vous pouvez utiliser des suffixes :

```csharp
// Suffixes pour les nombres entiers
int n1 = 42;       // int par défaut
long n2 = 42L;     // L pour long
uint n3 = 42U;     // U pour unsigned
ulong n4 = 42UL;   // UL pour unsigned long

// Suffixes pour les nombres décimaux
float f1 = 3.14F;  // F ou f pour float
double d1 = 3.14;  // Double par défaut (pas de suffixe nécessaire)
decimal m1 = 3.14M; // M ou m pour decimal
```

### Conversions entre types numériques

Il existe deux types de conversions entre types numériques :

1. **Conversions implicites** (sûres, pas de perte de données) :
   ```csharp
   int petitNombre = 10;
   long grandNombre = petitNombre;  // Conversion implicite de int vers long
   ```

2. **Conversions explicites** (cast, possibilité de perte de données) :
   ```csharp
   long grandNombre = 1234567890L;
   int petitNombre = (int)grandNombre;  // Conversion explicite, risque de perte de données
   ```

> **⚠️ Attention aux débordements :**
> Si vous convertissez un nombre vers un type qui ne peut pas le contenir, vous obtiendrez un débordement (overflow). Par exemple, essayer de stocker 300 dans un `byte` provoquera un débordement.

## 2.2.2 Types référence vs types valeur

En C#, tous les types de données sont classés soit comme **types valeur**, soit comme **types référence**. Cette distinction est fondamentale car elle affecte la manière dont les variables sont stockées en mémoire et passées aux méthodes.

### Types valeur

Les types valeur stockent directement leurs données en mémoire. Ils sont généralement alloués sur la pile (stack).

**Caractéristiques des types valeur :**
- Stockent directement leurs données
- Une variable contient la valeur elle-même
- Copiés lors d'affectations ou de passages en paramètre
- Généralement plus rapides d'accès
- Ne peuvent pas être `null` (sauf les types nullables, voir section 2.2.5)

**Types valeur en C# :**
- Tous les types primitifs numériques (`int`, `float`, `double`, etc.)
- `bool` et `char`
- `struct` (structures personnalisées)
- `enum` (énumérations)

```csharp
int a = 10;
int b = a;  // Une copie de la valeur est créée
b = 20;     // Modifier b n'affecte pas a
Console.WriteLine(a);  // Affiche 10
Console.WriteLine(b);  // Affiche 20
```

### Types référence

Les types référence stockent une référence (adresse mémoire) vers l'objet réel qui est alloué sur le tas (heap).

**Caractéristiques des types référence :**
- Stockent une référence vers les données
- Une variable contient l'adresse de l'objet, pas l'objet lui-même
- Lors d'affectation ou de passage en paramètre, seule la référence est copiée
- Peuvent être `null` (absence de référence)
- Nettoyés par le garbage collector quand plus aucune référence ne pointe vers eux

**Types référence en C# :**
- Classes (`class`)
- Interfaces (`interface`)
- Délégués (`delegate`)
- Tableaux (`array`)
- Chaînes de caractères (`string`) - cas particulier, voir section 2.2.3

```csharp
// Exemple avec une classe (type référence)
class Personne
{
    public string Nom;
}

// Dans une méthode
Personne p1 = new Personne();
p1.Nom = "Alice";

Personne p2 = p1;  // p2 et p1 pointent vers le même objet
p2.Nom = "Bob";    // Modifier via p2 affecte aussi p1

Console.WriteLine(p1.Nom);  // Affiche "Bob"
Console.WriteLine(p2.Nom);  // Affiche "Bob"
```

### Différence de comportement - Illustration visuelle

![Types valeur vs Types référence](https://i.imgur.com/JVaOcTE.png)

> **📝 Note importante :**
> Comprendre la différence entre types valeur et types référence est crucial pour éviter des bugs subtils dans votre code, notamment lors du passage de paramètres à des méthodes.

### Passage de paramètres (par valeur ou par référence)

Par défaut, C# passe les paramètres par valeur, ce qui signifie :
- Pour les types valeur : une copie de la valeur est passée
- Pour les types référence : une copie de la référence est passée (mais les deux pointent vers le même objet)

Les mots-clés `ref` et `out` permettent de modifier ce comportement :

```csharp
// Passage par référence avec ref
void DoubleNombre(ref int n)
{
    n = n * 2;
}

int x = 5;
DoubleNombre(ref x);
Console.WriteLine(x);  // Affiche 10, x a été modifié
```

## 2.2.3 Chaînes de caractères et leurs méthodes

Les chaînes de caractères (`string`) sont un type de données fondamental qui représente une séquence de caractères. Bien que techniquement un type référence, les chaînes ont un comportement spécial en C#.

### Création de chaînes

```csharp
// Différentes façons de créer des chaînes
string nom = "Alice";
string message = "Bonjour, comment allez-vous ?";
string vide = "";
string chemin = @"C:\Users\Documents\fichier.txt";  // Chaîne verbatim (@ ignore les séquences d'échappement)
string multiligne = @"Ligne 1
Ligne 2
Ligne 3";  // Chaîne multiligne
```

### Caractéristiques spéciales des chaînes

- Les chaînes sont **immuables** : une fois créée, une chaîne ne peut pas être modifiée. Toute opération qui semble modifier une chaîne crée en réalité une nouvelle chaîne.
- L'opérateur `+` permet de concaténer des chaînes.
- Les chaînes peuvent être comparées avec `==` et `!=` pour vérifier l'égalité de contenu.
- Les chaînes peuvent être nulles (`null`), contrairement aux types valeur.

```csharp
string a = "Bonjour";
string b = a;

a = a + " le monde";  // Crée une nouvelle chaîne "Bonjour le monde" et l'assigne à a
Console.WriteLine(a);  // Affiche "Bonjour le monde"
Console.WriteLine(b);  // Affiche toujours "Bonjour"
```

### Méthodes et propriétés utiles

Les chaînes offrent de nombreuses méthodes pour manipuler et analyser le texte :

```csharp
string texte = "  Exemple de chaîne à manipuler  ";

// Propriétés
int longueur = texte.Length;  // 32 (espaces inclus)

// Accès aux caractères individuels
char premierChar = texte[2];  // 'E'

// Recherche
bool contient = texte.Contains("chaîne");  // true
int position = texte.IndexOf("de");  // 10
int dernierE = texte.LastIndexOf("e");  // 27

// Modification (crée de nouvelles chaînes)
string sansBlancs = texte.Trim();  // "Exemple de chaîne à manipuler"
string majuscules = texte.ToUpper();  // "  EXEMPLE DE CHAÎNE À MANIPULER  "
string minuscules = texte.ToLower();  // "  exemple de chaîne à manipuler  "
string remplace = texte.Replace("chaîne", "texte");  // "  Exemple de texte à manipuler  "

// Extraction
string sousChaine = texte.Substring(10, 10);  // "de chaîne "

// Division
string[] mots = texte.Split(' ', StringSplitOptions.RemoveEmptyEntries);  // ["Exemple", "de", "chaîne", "à", "manipuler"]

// Vérification
bool commence = texte.StartsWith("  Ex");  // true
bool termine = texte.EndsWith("er  ");  // true
bool estVide = string.IsNullOrEmpty(texte);  // false
bool estBlancOuVide = string.IsNullOrWhiteSpace(texte);  // false
```

### Interpolation de chaînes

L'interpolation de chaînes (depuis C# 6.0) permet d'insérer des expressions directement dans des chaînes de caractères en utilisant le préfixe `$` :

```csharp
string prenom = "Marie";
int age = 30;
string message = $"Bonjour, je m'appelle {prenom} et j'ai {age} ans.";
// Message contient : "Bonjour, je m'appelle Marie et j'ai 30 ans."
```

### StringBuilder - pour les modifications fréquentes

Lorsque vous devez effectuer de nombreuses modifications sur une chaîne (dans une boucle par exemple), utilisez `StringBuilder` pour améliorer les performances :

```csharp
using System.Text;

StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100; i++)
{
    sb.Append($"Ligne {i}\n");  // Plus efficace que concaténation
}
string resultat = sb.ToString();
```

## 2.2.4 Boxing et unboxing

Le boxing et l'unboxing sont des mécanismes qui permettent de convertir des types valeur en types référence et vice-versa. Ces opérations sont importantes à comprendre car elles peuvent affecter les performances de votre application.

### Boxing (mise en boîte)

Le **boxing** est la conversion d'un type valeur en type référence. Lors du boxing, la valeur est encapsulée dans un objet et stockée sur le tas (heap).

```csharp
int nombre = 42;           // Type valeur sur la pile
object obj = nombre;       // Boxing : conversion implicite vers object (sur le tas)
```

### Unboxing (déballage)

L'**unboxing** est l'opération inverse : extraire la valeur d'un objet boxé et la convertir en type valeur.

```csharp
object obj = 42;          // Valeur boxée (référence)
int nombre = (int)obj;    // Unboxing : conversion explicite vers int
```

### Implications du boxing/unboxing

Ces opérations ont plusieurs implications :

1. **Performance** : Le boxing et l'unboxing sont coûteux en termes de performance car ils impliquent des allocations mémoire et des copies.

2. **Risques d'erreurs** : L'unboxing peut provoquer des exceptions si le type réel de l'objet ne correspond pas au type attendu.

```csharp
object obj = "texte";   // obj contient une chaîne, pas un nombre
int nombre = (int)obj;  // ⚠️ InvalidCastException
```

3. **Perte de type** : Lors du boxing, les informations de type spécifiques peuvent être perdues.

### Exemple pratique

```csharp
// Collection non générique (utilise Object)
System.Collections.ArrayList liste = new System.Collections.ArrayList();

// Boxing implicite lors de l'ajout (int → object)
liste.Add(10);
liste.Add(20);
liste.Add(30);

// Unboxing explicite lors de la récupération
int premier = (int)liste[0];

// Alternative sans boxing avec les collections génériques
List<int> listeGenerique = new List<int>();
listeGenerique.Add(10);  // Pas de boxing ici
int valeur = listeGenerique[0];  // Pas d'unboxing
```

> **💡 Conseil :**
> Pour éviter les problèmes de performance liés au boxing/unboxing, privilégiez les collections génériques (comme `List<T>`) plutôt que les collections non génériques (comme `ArrayList`).

## 2.2.5 Types nullables et opérateurs associés

Par défaut, les types valeur ne peuvent pas avoir la valeur `null`. Cependant, il existe des situations où vous pourriez vouloir représenter l'absence de valeur, par exemple pour une date de naissance non renseignée ou un prix non défini.

### Types nullables - Définition et syntaxe

Un type nullable est un type valeur qui peut également prendre la valeur `null`. On le déclare en ajoutant un point d'interrogation (`?`) après le type.

```csharp
// Types non nullables (par défaut)
int compteur = 0;
bool estValide = false;
// compteur = null;  // ❌ Erreur de compilation

// Types nullables
int? compteurNullable = 0;
bool? estValideNullable = false;
compteurNullable = null;  // ✅ Valide
estValideNullable = null;  // ✅ Valide
```

### Propriétés des types nullables

Les types nullables disposent de deux propriétés importantes :

- `HasValue` : Retourne `true` si le type nullable contient une valeur (différente de `null`).
- `Value` : Retourne la valeur contenue (génère une exception si la valeur est `null`).

```csharp
int? nombre = 42;

if (nombre.HasValue)
{
    int valeur = nombre.Value;  // Accès sécurisé à la valeur
    Console.WriteLine(valeur);  // Affiche 42
}

nombre = null;
// int valeur = nombre.Value;  // ⚠️ InvalidOperationException
```

### Opérateur de fusion null (`??`)

L'opérateur `??` (fusion null ou null-coalescing) permet de spécifier une valeur par défaut à utiliser lorsqu'un type nullable est `null`.

```csharp
int? a = null;
int b = a ?? 0;  // Si a est null, b prend la valeur 0

string nom = null;
string nomAffiche = nom ?? "Anonyme";  // Si nom est null, nomAffiche est "Anonyme"
```

### Opérateur de propagation null (`?.`)

L'opérateur `?.` (propagation null ou null-conditional) permet d'accéder en toute sécurité à un membre (propriété, méthode) d'un objet potentiellement null. Si l'objet est null, l'expression entière devient null au lieu de générer une exception.

```csharp
// Sans l'opérateur ?., on doit vérifier manuellement
string texte = null;
int? longueur = null;

if (texte != null)
{
    longueur = texte.Length;
}

// Avec l'opérateur ?., c'est plus concis
longueur = texte?.Length;  // Si texte est null, longueur est null

// On peut enchaîner les opérateurs
class Personne
{
    public Adresse Adresse { get; set; }
}

class Adresse
{
    public string Ville { get; set; }
}

Personne p = null;
string ville = p?.Adresse?.Ville;  // Null si p ou p.Adresse est null
```

### Combinaison des opérateurs `?.` et `??`

Ces opérateurs sont souvent utilisés ensemble pour accéder en toute sécurité à des propriétés et fournir des valeurs par défaut :

```csharp
Personne p = GetPersonne();  // Peut retourner null
string ville = p?.Adresse?.Ville ?? "Ville inconnue";
```

Dans cet exemple, `ville` vaudra "Ville inconnue" si `p` est null, si `p.Adresse` est null, ou si `p.Adresse.Ville` est null.

### Types nullables et C# 8.0+ : Nullable Reference Types

À partir de C# 8.0, Microsoft a introduit les "Nullable Reference Types" qui permettent d'indiquer si un type référence peut ou non contenir null. Cette fonctionnalité aide à prévenir les erreurs de référence nulle (`NullReferenceException`).

```csharp
// Avec nullable reference types activés
string nonNullable = "texte";  // Ne devrait jamais être null
string? nullable = null;       // Peut être null

nonNullable = null;  // ⚠️ Warning du compilateur
```

Pour activer cette fonctionnalité, vous pouvez utiliser la directive `#nullable enable` ou la configurer au niveau du projet.

## Résumé

- Les **types primitifs** sont les blocs de construction fondamentaux pour stocker des valeurs simples.
- Les **types valeur** stockent directement leurs données tandis que les **types référence** stockent une référence vers les données.
- Les **chaînes de caractères** sont un type référence spécial avec de nombreuses méthodes utiles.
- Le **boxing** et l'**unboxing** permettent de convertir entre types valeur et types référence, mais peuvent impacter les performances.
- Les **types nullables** permettent aux types valeur d'accepter la valeur `null`, avec des opérateurs associés (`??` et `?.`) pour manipuler ces valeurs de manière sécurisée.

En maîtrisant ces concepts, vous serez mieux équipé pour choisir les types appropriés et éviter les erreurs courantes dans vos programmes C#.

⏭️ 2.3. [Variables et constantes](/02-fondamentaux-du-langage-csharp/2-3-variables-et-constantes.md)
