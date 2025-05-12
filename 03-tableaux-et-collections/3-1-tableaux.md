# 3.1. Tableaux en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les tableaux sont des structures de données fondamentales qui permettent de stocker plusieurs valeurs du même type dans une seule variable. Ils servent de base à de nombreuses structures de données plus complexes et sont essentiels pour manipuler efficacement des collections d'éléments dans vos applications.

## 3.1.1. Tableaux unidimensionnels

Un tableau unidimensionnel est la forme la plus simple de tableau, représentant une liste linéaire d'éléments.

### Déclaration et initialisation

Il existe plusieurs façons de déclarer et initialiser un tableau unidimensionnel en C# :

```csharp
// Déclaration d'un tableau de 5 entiers sans initialisation
int[] nombres = new int[5];

// Déclaration et initialisation simultanées
string[] jours = new string[] { "Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi", "Samedi", "Dimanche" };

// Syntaxe simplifiée (le compilateur infère la taille du tableau)
double[] prix = { 9.99, 14.50, 19.99, 24.75 };

// Déclaration d'abord, puis initialisation plus tard
char[] voyelles;
voyelles = new char[] { 'a', 'e', 'i', 'o', 'u' };
```

### Accès aux éléments

Les éléments d'un tableau sont accessibles via leur indice (qui commence à 0) :

```csharp
string[] pays = { "France", "Espagne", "Italie", "Allemagne" };

// Accès au premier élément (indice 0)
Console.WriteLine(pays[0]);  // Affiche "France"

// Accès au troisième élément (indice 2)
Console.WriteLine(pays[2]);  // Affiche "Italie"

// Modification d'un élément
pays[1] = "Portugal";
Console.WriteLine(pays[1]);  // Affiche "Portugal"
```

### Propriété Length

La propriété `Length` permet de connaître le nombre d'éléments dans un tableau :

```csharp
int[] nombres = { 10, 20, 30, 40, 50 };
Console.WriteLine($"Le tableau contient {nombres.Length} éléments.");  // Affiche 5
```

### Parcourir un tableau

Il existe plusieurs façons de parcourir un tableau unidimensionnel :

```csharp
string[] fruits = { "Pomme", "Banane", "Orange", "Fraise", "Kiwi" };

// 1. Avec une boucle for classique
Console.WriteLine("Parcours avec for :");
for (int i = 0; i < fruits.Length; i++)
{
    Console.WriteLine($"fruits[{i}] = {fruits[i]}");
}

// 2. Avec une boucle foreach (plus lisible pour simplement lire les éléments)
Console.WriteLine("\nParcours avec foreach :");
foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}
```

### Exemple complet

Voici un exemple concret d'utilisation d'un tableau unidimensionnel :

```csharp
static void Main(string[] args)
{
    // Création d'un tableau de notes
    double[] notes = { 12.5, 15, 8.75, 14, 10 };

    // Calcul de la moyenne
    double somme = 0;
    foreach (double note in notes)
    {
        somme += note;
    }
    double moyenne = somme / notes.Length;

    // Recherche de la note maximale
    double noteMax = notes[0];
    for (int i = 1; i < notes.Length; i++)
    {
        if (notes[i] > noteMax)
        {
            noteMax = notes[i];
        }
    }

    // Affichage des résultats
    Console.WriteLine($"Nombre de notes : {notes.Length}");
    Console.WriteLine($"Moyenne : {moyenne:F2}");
    Console.WriteLine($"Note la plus élevée : {noteMax}");
}
```

## 3.1.2. Tableaux multidimensionnels

Les tableaux multidimensionnels permettent de représenter des structures de données avec plusieurs dimensions. En C#, il existe deux types de tableaux multidimensionnels : les tableaux rectangulaires et les tableaux en dents de scie (jagged arrays).

### Tableaux rectangulaires (2D)

Un tableau rectangulaire possède un nombre fixe de lignes et de colonnes. Tous les éléments sont stockés dans une structure rectangulaire.

```csharp
// Déclaration d'un tableau 2D de 3 lignes et 4 colonnes
int[,] matrice = new int[3, 4];

// Initialisation d'un tableau 2D avec des valeurs
int[,] grille = new int[,]
{
    { 1, 2, 3, 4 },  // ligne 0
    { 5, 6, 7, 8 },  // ligne 1
    { 9, 10, 11, 12 }  // ligne 2
};

// Accès aux éléments
Console.WriteLine(grille[0, 0]);  // Affiche 1 (premier élément)
Console.WriteLine(grille[1, 2]);  // Affiche 7 (ligne 1, colonne 2)

// Modification d'un élément
grille[2, 3] = 99;
Console.WriteLine(grille[2, 3]);  // Affiche 99
```

### Obtenir les dimensions d'un tableau rectangulaire

```csharp
int[,] grille = new int[,]
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

// Nombre total d'éléments
Console.WriteLine($"Nombre total d'éléments : {grille.Length}");  // Affiche 12

// Nombre de dimensions
Console.WriteLine($"Nombre de dimensions : {grille.Rank}");  // Affiche 2

// Taille de chaque dimension
Console.WriteLine($"Nombre de lignes (dimension 0) : {grille.GetLength(0)}");  // Affiche 3
Console.WriteLine($"Nombre de colonnes (dimension 1) : {grille.GetLength(1)}");  // Affiche 4
```

### Parcourir un tableau 2D

```csharp
int[,] grille = new int[,]
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

// Parcours avec des boucles for imbriquées
for (int i = 0; i < grille.GetLength(0); i++)  // lignes
{
    for (int j = 0; j < grille.GetLength(1); j++)  // colonnes
    {
        Console.Write($"{grille[i, j]}\t");
    }
    Console.WriteLine();  // Nouvelle ligne après chaque ligne de la matrice
}
```

### Exemple pratique : Plateau de jeu

```csharp
static void AfficherPlateau(char[,] plateau)
{
    int lignes = plateau.GetLength(0);
    int colonnes = plateau.GetLength(1);

    // Afficher les indices de colonnes
    Console.Write("  ");
    for (int j = 0; j < colonnes; j++)
    {
        Console.Write($" {j} ");
    }
    Console.WriteLine();

    // Afficher le plateau
    for (int i = 0; i < lignes; i++)
    {
        Console.Write($"{i} ");  // Indice de ligne
        for (int j = 0; j < colonnes; j++)
        {
            Console.Write($"[{plateau[i, j]}]");
        }
        Console.WriteLine();
    }
}

static void Main(string[] args)
{
    // Créer un plateau de jeu 3x3 (morpion/tic-tac-toe)
    char[,] plateau = new char[3, 3];

    // Initialiser avec des espaces vides
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            plateau[i, j] = ' ';
        }
    }

    // Simuler quelques coups
    plateau[0, 0] = 'X';
    plateau[1, 1] = 'O';
    plateau[0, 2] = 'X';

    // Afficher le plateau
    AfficherPlateau(plateau);
}
```

### Tableaux de dimension supérieure

C# supporte également les tableaux de dimension supérieure à 2. Voici un exemple de tableau 3D :

```csharp
// Tableau 3D (cube de données)
int[,,] cube = new int[3, 3, 3];

// Initialisation d'une valeur
cube[0, 1, 2] = 42;

// Parcours d'un tableau 3D
for (int i = 0; i < cube.GetLength(0); i++)
{
    for (int j = 0; j < cube.GetLength(1); j++)
    {
        for (int k = 0; k < cube.GetLength(2); k++)
        {
            Console.WriteLine($"cube[{i}, {j}, {k}] = {cube[i, j, k]}");
        }
    }
}
```

## 3.1.3. Tableaux en dents de scie (jagged arrays)

Les tableaux en dents de scie (ou tableaux irréguliers) sont des tableaux de tableaux. Contrairement aux tableaux multidimensionnels rectangulaires, chaque "ligne" peut avoir une longueur différente.

### Déclaration et initialisation

```csharp
// Déclaration d'un tableau de tableaux d'entiers
int[][] nombres = new int[3][];

// Initialisation de chaque sous-tableau avec des tailles différentes
nombres[0] = new int[4] { 1, 2, 3, 4 };
nombres[1] = new int[2] { 5, 6 };
nombres[2] = new int[5] { 7, 8, 9, 10, 11 };

// Initialisation directe
int[][] matrice = new int[][]
{
    new int[] { 1, 2, 3 },
    new int[] { 4, 5 },
    new int[] { 6, 7, 8, 9 }
};
```

### Accès aux éléments

```csharp
int[][] matrice = new int[][]
{
    new int[] { 1, 2, 3 },
    new int[] { 4, 5 },
    new int[] { 6, 7, 8, 9 }
};

// Accès aux éléments
Console.WriteLine(matrice[0][1]);  // Affiche 2
Console.WriteLine(matrice[2][3]);  // Affiche 9

// Modification d'un élément
matrice[1][0] = 42;
Console.WriteLine(matrice[1][0]);  // Affiche 42
```

### Parcourir un tableau en dents de scie

```csharp
int[][] matrice = new int[][]
{
    new int[] { 1, 2, 3 },
    new int[] { 4, 5 },
    new int[] { 6, 7, 8, 9 }
};

// Parcours avec des boucles imbriquées
for (int i = 0; i < matrice.Length; i++)
{
    Console.Write($"Ligne {i}: ");

    for (int j = 0; j < matrice[i].Length; j++)
    {
        Console.Write($"{matrice[i][j]} ");
    }

    Console.WriteLine();
}
```

### Cas d'utilisation

Les tableaux en dents de scie sont utiles lorsque vous avez besoin de stocker des données avec des longueurs variables par ligne :

```csharp
// Exemple : Liste d'étudiants avec leurs notes (nombre variable par étudiant)
string[] noms = { "Alice", "Bob", "Charlie" };
double[][] notes = new double[3][];

notes[0] = new double[] { 15, 12, 18, 10 };  // 4 notes pour Alice
notes[1] = new double[] { 14, 13 };          // 2 notes pour Bob
notes[2] = new double[] { 16, 15, 17 };      // 3 notes pour Charlie

// Calculer et afficher la moyenne de chaque étudiant
for (int i = 0; i < noms.Length; i++)
{
    double somme = 0;
    foreach (double note in notes[i])
    {
        somme += note;
    }
    double moyenne = somme / notes[i].Length;

    Console.WriteLine($"{noms[i]} a une moyenne de {moyenne:F2} (sur {notes[i].Length} notes)");
}
```

## 3.1.4. Méthodes utiles pour manipuler des tableaux

La classe `System.Array` fournit de nombreuses méthodes statiques pour manipuler les tableaux. Voici les plus couramment utilisées :

### Redimensionner un tableau

```csharp
int[] nombres = { 1, 2, 3 };
Console.WriteLine($"Taille initiale : {nombres.Length}");  // Affiche 3

// Redimensionner le tableau à 5 éléments
Array.Resize(ref nombres, 5);
Console.WriteLine($"Nouvelle taille : {nombres.Length}");  // Affiche 5

// Les nouveaux éléments sont initialisés avec la valeur par défaut (0 pour les int)
foreach (int n in nombres)
{
    Console.Write($"{n} ");  // Affiche 1 2 3 0 0
}
Console.WriteLine();

// Réduire la taille (attention: les éléments en trop sont perdus)
Array.Resize(ref nombres, 2);
foreach (int n in nombres)
{
    Console.Write($"{n} ");  // Affiche 1 2
}
```

### Trier un tableau

```csharp
int[] nombres = { 5, 2, 8, 1, 9, 3 };

// Tri dans l'ordre croissant
Array.Sort(nombres);

Console.WriteLine("Tableau trié :");
foreach (int n in nombres)
{
    Console.Write($"{n} ");  // Affiche 1 2 3 5 8 9
}
Console.WriteLine();

// Tri d'un tableau de chaînes
string[] fruits = { "Orange", "Pomme", "Banane", "Kiwi" };
Array.Sort(fruits);

foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);  // Affiche Banane, Kiwi, Orange, Pomme (ordre alphabétique)
}
```

### Inverser l'ordre des éléments

```csharp
int[] nombres = { 1, 2, 3, 4, 5 };

// Inverser le tableau
Array.Reverse(nombres);

Console.WriteLine("Tableau inversé :");
foreach (int n in nombres)
{
    Console.Write($"{n} ");  // Affiche 5 4 3 2 1
}
```

### Rechercher des éléments

```csharp
string[] animaux = { "Chat", "Chien", "Lapin", "Oiseau", "Poisson" };

// Rechercher un élément
int index = Array.IndexOf(animaux, "Lapin");
Console.WriteLine($"'Lapin' trouvé à l'index {index}");  // Affiche 2

// Vérifier si un élément existe
bool contientChat = Array.Exists(animaux, animal => animal == "Chat");
Console.WriteLine($"Le tableau contient 'Chat' : {contientChat}");  // Affiche True

// Trouver le premier élément qui correspond à une condition
string premierAvecC = Array.Find(animaux, animal => animal.StartsWith("C"));
Console.WriteLine($"Premier animal commençant par 'C' : {premierAvecC}");  // Affiche Chat

// Trouver tous les éléments qui correspondent à une condition
string[] tousAvecC = Array.FindAll(animaux, animal => animal.StartsWith("C"));
Console.WriteLine("Tous les animaux commençant par 'C' :");
foreach (string animal in tousAvecC)
{
    Console.WriteLine(animal);  // Affiche Chat, Chien
}
```

### Copier un tableau

```csharp
int[] source = { 1, 2, 3, 4, 5 };

// Méthode 1 : Array.Copy
int[] destination1 = new int[source.Length];
Array.Copy(source, destination1, source.Length);

// Méthode 2 : CopyTo
int[] destination2 = new int[source.Length];
source.CopyTo(destination2, 0);

// Méthode 3 : Clone (crée une copie superficielle)
int[] destination3 = (int[])source.Clone();

// Vérification que les copies sont indépendantes
source[0] = 99;
Console.WriteLine($"Source[0] = {source[0]}");  // Affiche 99
Console.WriteLine($"Destination1[0] = {destination1[0]}");  // Affiche 1
Console.WriteLine($"Destination2[0] = {destination2[0]}");  // Affiche 1
Console.WriteLine($"Destination3[0] = {destination3[0]}");  // Affiche 1
```

### Remplir un tableau avec une valeur

```csharp
int[] nombres = new int[5];

// Remplir le tableau avec la valeur 42
Array.Fill(nombres, 42);

foreach (int n in nombres)
{
    Console.Write($"{n} ");  // Affiche 42 42 42 42 42
}
```

### Vérifier si tous les éléments répondent à une condition

```csharp
int[] nombres = { 2, 4, 6, 8, 10 };

// Vérifier si tous les éléments sont pairs
bool tousPairs = Array.TrueForAll(nombres, n => n % 2 == 0);
Console.WriteLine($"Tous les nombres sont pairs : {tousPairs}");  // Affiche True

// Vérifier si tous les éléments sont supérieurs à 5
bool tousSup5 = Array.TrueForAll(nombres, n => n > 5);
Console.WriteLine($"Tous les nombres sont > 5 : {tousSup5}");  // Affiche False
```

### Convertir un tableau

```csharp
int[] entiers = { 1, 2, 3, 4, 5 };

// Convertir un tableau d'entiers en tableau de doubles
double[] doubles = Array.ConvertAll(entiers, n => (double)n);

// Convertir un tableau d'entiers en tableau de chaînes
string[] chaines = Array.ConvertAll(entiers, n => n.ToString());

foreach (string s in chaines)
{
    Console.Write($"{s} ");  // Affiche "1 2 3 4 5"
}
```

## Quelques conseils pratiques

### Choix entre types de tableaux

- **Tableau unidimensionnel** : pour des listes simples d'éléments du même type.
- **Tableau multidimensionnel (rectangulaire)** : pour des données structurées en grille avec des dimensions fixes.
- **Tableau en dents de scie** : lorsque chaque "ligne" peut avoir une longueur différente.

### Bonnes pratiques

1. **Vérifiez les limites** : Avant d'accéder à un élément par son indice, assurez-vous qu'il est dans les limites valides du tableau pour éviter les exceptions `IndexOutOfRangeException`.

```csharp
if (index >= 0 && index < tableau.Length)
{
    // Accès sécurisé
    var element = tableau[index];
}
```

2. **Préférez foreach quand c'est possible** : Pour parcourir tous les éléments d'un tableau sans modification, `foreach` est plus lisible et moins sujet aux erreurs.

3. **Utilisez les méthodes LINQ** : Pour des opérations complexes sur les tableaux, les méthodes LINQ peuvent rendre votre code plus concis et lisible.

```csharp
int[] nombres = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Filtrer les nombres pairs et les multiplier par 2
var resultat = nombres.Where(n => n % 2 == 0).Select(n => n * 2);

foreach (var n in resultat)
{
    Console.Write($"{n} ");  // Affiche 4 8 12 16 20
}
```

4. **Attention aux performances** : Les tableaux de grande taille dans des dimensions élevées peuvent consommer beaucoup de mémoire. Pour les structures de données très complexes, envisagez d'autres approches.

### Exercices pratiques

Pour bien maîtriser les tableaux, essayez ces exercices :

1. Créez un tableau de nombres et calculez la somme, la moyenne, le minimum et le maximum.
2. Implémentez un algorithme de tri simple (comme le tri à bulles) sur un tableau d'entiers.
3. Créez une matrice 2D pour représenter un jeu de plateau (comme les échecs ou le morpion).
4. Utilisez un tableau en dents de scie pour stocker une liste de listes d'éléments de longueur variable.
5. Implémentez un algorithme de recherche binaire dans un tableau trié.

Ces exercices vous aideront à développer vos compétences et à comprendre les différentes façons d'utiliser les tableaux en C#.

⏭️ 3.2. [Collections standard](/03-tableaux-et-collections/3-2-collections-standard.md)
