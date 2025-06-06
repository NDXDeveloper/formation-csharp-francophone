# 2.5 Structures de contrôle en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les structures de contrôle sont les blocs de construction qui permettent à vos programmes de prendre des décisions, de répéter des actions et de contrôler le flux d'exécution. Sans elles, les programmes s'exécuteraient simplement de haut en bas sans aucune intelligence ou adaptabilité. Comprendre ces structures est essentiel pour tout développeur C#.

## 2.5.1 Conditionnelles (if, else, switch)

Les structures conditionnelles permettent à votre programme de prendre des décisions en exécutant différents blocs de code selon qu'une condition est vraie ou fausse.

### Instruction if

L'instruction `if` est la conditionnelle la plus simple. Elle exécute un bloc de code uniquement si une condition est évaluée à `true`.

```csharp
if (condition)
{
    // Ce code s'exécute uniquement si la condition est vraie
}
```

Exemple pratique :

```csharp
int age = 20;

if (age >= 18)
{
    Console.WriteLine("Vous êtes majeur.");
}
```

Si le bloc de code ne contient qu'une seule instruction, les accolades sont optionnelles, mais il est généralement recommandé de les conserver pour plus de clarté :

```csharp
if (age >= 18)
    Console.WriteLine("Vous êtes majeur.");  // Fonctionne, mais moins lisible
```

### Instruction if-else

L'instruction `if-else` permet d'exécuter un bloc de code si la condition est vraie, et un autre bloc si elle est fausse.

```csharp
if (condition)
{
    // Ce code s'exécute si la condition est vraie
}
else
{
    // Ce code s'exécute si la condition est fausse
}
```

Exemple pratique :

```csharp
int age = 15;

if (age >= 18)
{
    Console.WriteLine("Vous êtes majeur.");
}
else
{
    Console.WriteLine("Vous êtes mineur.");
}
```

### Instruction if-else if-else

Vous pouvez enchaîner plusieurs conditions avec `else if` pour tester plusieurs cas successifs.

```csharp
if (condition1)
{
    // Code exécuté si condition1 est vraie
}
else if (condition2)
{
    // Code exécuté si condition1 est fausse ET condition2 est vraie
}
else if (condition3)
{
    // Code exécuté si condition1 et condition2 sont fausses ET condition3 est vraie
}
else
{
    // Code exécuté si toutes les conditions précédentes sont fausses
}
```

Exemple pratique :

```csharp
int note = 85;

if (note >= 90)
{
    Console.WriteLine("Excellent !");
}
else if (note >= 80)
{
    Console.WriteLine("Très bien !");
}
else if (note >= 70)
{
    Console.WriteLine("Bien.");
}
else if (note >= 60)
{
    Console.WriteLine("Passable.");
}
else
{
    Console.WriteLine("Échec.");
}
```

> **🔍 Note importante :**
> Les conditions sont évaluées dans l'ordre. Dès qu'une condition est vraie, son bloc de code est exécuté et les conditions suivantes ne sont pas évaluées.

### Conditions imbriquées

Vous pouvez imbriquer des instructions `if` à l'intérieur d'autres instructions `if` ou `else`.

```csharp
if (estAuthentifie)
{
    if (estAdmin)
    {
        Console.WriteLine("Bienvenue, administrateur !");
    }
    else
    {
        Console.WriteLine("Bienvenue, utilisateur !");
    }
}
else
{
    Console.WriteLine("Authentification requise.");
}
```

Bien que les conditions imbriquées fonctionnent, elles peuvent rapidement rendre votre code difficile à lire. Si possible, essayez de simplifier la logique ou d'utiliser des conditions combinées :

```csharp
if (estAuthentifie && estAdmin)
{
    Console.WriteLine("Bienvenue, administrateur !");
}
else if (estAuthentifie)
{
    Console.WriteLine("Bienvenue, utilisateur !");
}
else
{
    Console.WriteLine("Authentification requise.");
}
```

### Instruction switch

L'instruction `switch` est utile lorsque vous avez plusieurs choix possibles basés sur la valeur d'une variable. Elle remplace avantageusement une longue série de `if-else if`.

```csharp
switch (expression)
{
    case valeur1:
        // Code à exécuter si expression égale valeur1
        break;
    case valeur2:
        // Code à exécuter si expression égale valeur2
        break;
    // ... autres cas ...
    default:
        // Code à exécuter si aucun cas ne correspond
        break;
}
```

Exemple pratique :

```csharp
int jour = 3;
string nomJour;

switch (jour)
{
    case 1:
        nomJour = "Lundi";
        break;
    case 2:
        nomJour = "Mardi";
        break;
    case 3:
        nomJour = "Mercredi";
        break;
    case 4:
        nomJour = "Jeudi";
        break;
    case 5:
        nomJour = "Vendredi";
        break;
    case 6:
        nomJour = "Samedi";
        break;
    case 7:
        nomJour = "Dimanche";
        break;
    default:
        nomJour = "Jour invalide";
        break;
}

Console.WriteLine($"Le jour {jour} est {nomJour}.");
```

#### Points importants sur switch

1. **L'instruction `break`** est nécessaire à la fin de chaque cas pour sortir du switch. Sans elle, l'exécution continue au cas suivant (ce qu'on appelle "fall-through").

2. **Le cas `default`** est optionnel mais recommandé. Il gère les valeurs qui ne correspondent à aucun cas.

3. **Plusieurs valeurs pour un même cas** (à partir de C# 7.0) :
   ```csharp
   case 1:
   case 2:
   case 3:
       Console.WriteLine("Premier trimestre");
       break;
   ```

4. **Types supportés** : Traditionnellement, `switch` fonctionnait avec des types simples (entiers, caractères, chaînes), mais les versions récentes de C# ont étendu ses capacités (voir la section sur le pattern matching).

## 2.5.2 Boucles (for, while, do-while, foreach)

Les boucles permettent d'exécuter un bloc de code plusieurs fois, ce qui est essentiel pour traiter des collections de données ou répéter des actions.

### Boucle for

La boucle `for` est idéale quand vous connaissez à l'avance le nombre d'itérations à effectuer. Elle comporte trois parties : l'initialisation, la condition et l'incrémentation.

```csharp
for (initialisation; condition; incrémentation)
{
    // Code à répéter tant que la condition est vraie
}
```

Exemple pratique :

```csharp
// Afficher les nombres de 1 à 5
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine(i);
}
```

Dans cet exemple :
- `int i = 1` : initialise la variable de compteur
- `i <= 5` : la boucle continue tant que cette condition est vraie
- `i++` : incrémente la variable après chaque itération

#### Variations sur la boucle for

1. **Comptage à rebours** :
   ```csharp
   for (int i = 10; i > 0; i--)
   {
       Console.WriteLine(i);
   }
   Console.WriteLine("Décollage !");
   ```

2. **Incrémentation différente** :
   ```csharp
   // Afficher les nombres pairs de 0 à 10
   for (int i = 0; i <= 10; i += 2)
   {
       Console.WriteLine(i);
   }
   ```

3. **Boucle for sans corps** :
   ```csharp
   // Calculer la somme des nombres de 1 à 100
   int somme = 0;
   for (int i = 1; i <= 100; i++) somme += i;
   ```

4. **Boucle for avec plusieurs variables** :
   ```csharp
   for (int i = 0, j = 10; i < j; i++, j--)
   {
       Console.WriteLine($"i = {i}, j = {j}");
   }
   ```

5. **Boucle for infinie** (à utiliser avec précaution) :
   ```csharp
   for (;;)
   {
       // Code à répéter indéfiniment
       // Assurez-vous d'avoir une condition de sortie !
       if (condition) break;
   }
   ```

### Boucle while

La boucle `while` répète un bloc de code tant qu'une condition reste vraie. Elle est utile quand vous ne connaissez pas à l'avance le nombre d'itérations nécessaires.

```csharp
while (condition)
{
    // Code à répéter tant que la condition est vraie
}
```

Exemple pratique :

```csharp
int compteur = 1;

while (compteur <= 5)
{
    Console.WriteLine(compteur);
    compteur++;
}
```

La boucle `while` évalue la condition avant d'exécuter le bloc. Si la condition est fausse dès le début, le bloc ne sera jamais exécuté.

```csharp
int x = 10;

while (x < 5)
{
    Console.WriteLine("Ce code ne s'exécutera jamais");
}
```

### Boucle do-while

La boucle `do-while` est similaire à `while`, mais elle évalue la condition après avoir exécuté le bloc. Cela garantit que le bloc s'exécute au moins une fois.

```csharp
do
{
    // Code à exécuter au moins une fois
    // puis à répéter tant que la condition est vraie
} while (condition);
```

Exemple pratique :

```csharp
int compteur = 1;

do
{
    Console.WriteLine(compteur);
    compteur++;
} while (compteur <= 5);
```

Même si la condition est fausse dès le début, le bloc s'exécute une fois :

```csharp
int x = 10;

do
{
    Console.WriteLine("Ce code s'exécute une fois");
} while (x < 5);
```

La boucle `do-while` est utile pour les scénarios où vous devez exécuter une action au moins une fois, comme demander une entrée utilisateur jusqu'à ce qu'elle soit valide.

```csharp
string mot;
do
{
    Console.Write("Entrez un mot d'au moins 5 lettres : ");
    mot = Console.ReadLine();
} while (mot.Length < 5);

Console.WriteLine($"Merci, vous avez entré : {mot}");
```

### Boucle foreach

La boucle `foreach` est spécialement conçue pour parcourir les éléments d'une collection (tableaux, listes, dictionnaires, etc.) sans se soucier des indices ou du nombre d'éléments.

```csharp
foreach (type élément in collection)
{
    // Code à exécuter pour chaque élément
}
```

Exemple pratique avec un tableau :

```csharp
string[] fruits = { "pomme", "banane", "orange", "fraise", "kiwi" };

foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}
```

Exemple avec une liste :

```csharp
List<int> nombres = new List<int> { 1, 2, 3, 4, 5 };

foreach (int nombre in nombres)
{
    Console.WriteLine(nombre * nombre);
}
```

#### Avantages de foreach

1. **Simplicité** : Pas besoin de gérer les indices ou le comptage
2. **Sécurité** : Pas de risque de dépasser les limites de la collection
3. **Universel** : Fonctionne avec toute collection qui implémente l'interface `IEnumerable`

#### Limitations de foreach

1. **Lecture seule** : Vous ne pouvez pas modifier la collection pendant l'itération
2. **Pas d'indice** : Si vous avez besoin de l'indice, utilisez plutôt `for`

> **💡 Conseil :**
> En général, privilégiez `foreach` pour parcourir des collections lorsque vous n'avez pas besoin de l'indice ou de modifier la collection pendant l'itération.

## 2.5.3 Instructions de saut (break, continue, return)

Les instructions de saut modifient le flux normal d'exécution d'une boucle ou d'une méthode. Elles vous permettent de sortir prématurément d'une boucle, de sauter à l'itération suivante ou de quitter une méthode.

### break

L'instruction `break` termine immédiatement la boucle ou le switch dans lequel elle se trouve.

Dans une boucle :

```csharp
for (int i = 1; i <= 10; i++)
{
    if (i == 5)
    {
        break;  // Sort de la boucle quand i atteint 5
    }
    Console.WriteLine(i);
}
// Affiche : 1 2 3 4
```

Dans un switch :

```csharp
switch (jour)
{
    case 1:
        Console.WriteLine("Lundi");
        break;  // Sort du switch
    case 2:
        Console.WriteLine("Mardi");
        break;
    // ...
}
```

L'instruction `break` est utile lorsque vous avez trouvé ce que vous cherchiez et n'avez pas besoin de continuer l'itération :

```csharp
bool EstPresent(int[] tableau, int valeur)
{
    foreach (int element in tableau)
    {
        if (element == valeur)
        {
            return true;  // Trouvé !
        }
    }
    return false;  // Non trouvé
}
```

### continue

L'instruction `continue` saute le reste de l'itération actuelle et passe directement à l'itération suivante.

```csharp
for (int i = 1; i <= 10; i++)
{
    if (i % 2 == 0)
    {
        continue;  // Saute les nombres pairs
    }
    Console.WriteLine(i);
}
// Affiche : 1 3 5 7 9
```

L'instruction `continue` est utile pour filtrer certains éléments sans compliquer la structure de votre boucle :

```csharp
foreach (string ligne in fichier)
{
    // Ignorer les lignes vides ou les commentaires
    if (string.IsNullOrWhiteSpace(ligne) || ligne.StartsWith("//"))
    {
        continue;
    }

    // Traiter les lignes normales ici
    TraiterLigne(ligne);
}
```

### return

L'instruction `return` quitte immédiatement la méthode et renvoie éventuellement une valeur.

```csharp
int Calculer(int a, int b)
{
    if (b == 0)
    {
        Console.WriteLine("Division par zéro impossible");
        return 0;  // Quitte la méthode immédiatement
    }

    return a / b;  // Quitte la méthode avec le résultat
}
```

L'instruction `return` est particulièrement utile pour les retours conditionnels précoces, qui peuvent souvent simplifier la logique de votre code :

```csharp
// Approche avec return précoce
bool EstValide(string texte)
{
    if (string.IsNullOrEmpty(texte))
    {
        return false;
    }

    if (texte.Length < 3)
    {
        return false;
    }

    if (!char.IsUpper(texte[0]))
    {
        return false;
    }

    return true;
}

// Au lieu de :
bool EstValideSansRetourPrecoce(string texte)
{
    bool resultat = true;

    if (string.IsNullOrEmpty(texte))
    {
        resultat = false;
    }
    else if (texte.Length < 3)
    {
        resultat = false;
    }
    else if (!char.IsUpper(texte[0]))
    {
        resultat = false;
    }

    return resultat;
}
```

### goto (à utiliser avec précaution)

L'instruction `goto` transfère directement l'exécution à une étiquette spécifiée. Elle est généralement déconseillée car elle rend le code difficile à suivre, mais elle peut être utile dans certains cas spécifiques, comme sortir de boucles imbriquées.

```csharp
for (int i = 0; i < 10; i++)
{
    for (int j = 0; j < 10; j++)
    {
        if (i * j > 50)
        {
            goto FinDesBoucles;  // Sort des deux boucles à la fois
        }
        Console.WriteLine($"{i} * {j} = {i * j}");
    }
}

FinDesBoucles:
Console.WriteLine("Boucles terminées");
```

> **⚠️ Attention :**
> L'utilisation de `goto` est généralement considérée comme une mauvaise pratique car elle peut rendre le code difficile à comprendre et à maintenir. Préférez les autres structures de contrôle quand c'est possible.

## 2.5.4 Pattern matching (à partir de C# 7.0)

Le pattern matching, introduit avec C# 7.0 et amélioré dans les versions suivantes, permet d'examiner un objet et d'agir différemment selon son type, sa structure ou sa valeur.

### Pattern matching avec is

L'opérateur `is` vérifie si un objet est d'un type spécifique, mais en plus, il peut assigner le résultat à une variable si le test réussit (pattern matching de type).

```csharp
// Avant C# 7.0
if (animal is Dog)
{
    Dog dog = (Dog)animal;
    dog.Bark();
}

// Avec pattern matching (C# 7.0+)
if (animal is Dog dog)
{
    dog.Bark();  // La variable dog est déjà typée et disponible
}
```

### Pattern matching avec constantes

Vous pouvez également faire correspondre une variable à une valeur constante :

```csharp
if (obj is 42)
{
    Console.WriteLine("La valeur est 42");
}

if (obj is null)
{
    Console.WriteLine("L'objet est null");
}
```

### Pattern matching dans switch

Le pattern matching est particulièrement puissant dans les instructions switch, où il permet de vérifier à la fois le type et les propriétés d'un objet :

```csharp
switch (shape)
{
    case Circle circle when circle.Radius > 10:
        Console.WriteLine($"Grand cercle de rayon {circle.Radius}");
        break;

    case Circle circle:
        Console.WriteLine($"Cercle de rayon {circle.Radius}");
        break;

    case Rectangle rectangle when rectangle.Width == rectangle.Height:
        Console.WriteLine($"Carré de côté {rectangle.Width}");
        break;

    case Rectangle rectangle:
        Console.WriteLine($"Rectangle de dimensions {rectangle.Width}x{rectangle.Height}");
        break;

    case null:
        Console.WriteLine("La forme est null");
        break;

    default:
        Console.WriteLine($"Autre forme: {shape.GetType().Name}");
        break;
}
```

Dans cet exemple :
- Nous vérifions d'abord si `shape` est un `Circle` avec un rayon supérieur à 10
- Sinon, nous vérifions si c'est un `Circle` quelconque
- Ensuite, nous vérifions si c'est un `Rectangle` avec largeur égale à la hauteur (un carré)
- Et ainsi de suite...

### Pattern matching avec la déconstruction (C# 8.0+)

Vous pouvez combiner le pattern matching avec la déconstruction pour examiner plusieurs propriétés d'un objet en une seule fois :

```csharp
// Avec une classe qui supporte la déconstruction
if (point is (0, 0))
{
    Console.WriteLine("Point à l'origine");
}

switch (point)
{
    case (0, 0):
        Console.WriteLine("Point à l'origine");
        break;
    case (var x, 0):
        Console.WriteLine($"Point sur l'axe X à {x}");
        break;
    case (0, var y):
        Console.WriteLine($"Point sur l'axe Y à {y}");
        break;
    case (var x, var y) when x == y:
        Console.WriteLine($"Point sur la diagonale à ({x}, {y})");
        break;
    default:
        Console.WriteLine($"Point à ({point.X}, {point.Y})");
        break;
}
```

### Property pattern (C# 8.0+)

Le property pattern permet d'examiner les propriétés d'un objet de manière concise :

```csharp
// Vérifier si une personne est un adulte de nationalité française
if (personne is { Age: >= 18, Nationalite: "Française" })
{
    Console.WriteLine("Adulte de nationalité française");
}

// Équivalent en switch
switch (personne)
{
    case { Age: >= 18, Nationalite: "Française" }:
        Console.WriteLine("Adulte de nationalité française");
        break;
    case { Age: < 18 }:
        Console.WriteLine("Mineur");
        break;
    default:
        Console.WriteLine("Adulte d'une autre nationalité");
        break;
}
```

## 2.5.5 Switch expressions (C# 8+)

C# 8.0 a introduit les expressions switch, une forme plus concise et expressive des instructions switch traditionnelles. Elles sont particulièrement utiles lorsque vous voulez assigner une valeur basée sur différents cas.

### Syntaxe de base

```csharp
// Instruction switch traditionnelle
string GetMessageClassique(int code)
{
    switch (code)
    {
        case 200:
            return "OK";
        case 404:
            return "Not Found";
        case 500:
            return "Server Error";
        default:
            return "Unknown";
    }
}

// Équivalent avec une expression switch
string GetMessage(int code) => code switch
{
    200 => "OK",
    404 => "Not Found",
    500 => "Server Error",
    _ => "Unknown"    // _ est le motif par défaut
};
```

Points clés à noter :
- La variable à tester (`code`) est placée avant le mot-clé `switch`
- Chaque cas utilise la syntaxe `motif => résultat`
- Le underscore `_` représente le cas par défaut
- La ponctuation est différente : virgules entre les cas et point-virgule à la fin

### Utilisation avec le pattern matching

Les expressions switch sont encore plus puissantes lorsqu'elles sont combinées avec le pattern matching :

```csharp
string FormatPerson(object obj) => obj switch
{
    Person { FirstName: "John", LastName: "Doe" } => "C'est John Doe !",
    Person { LastName: var name } => $"Bonjour M. {name}",
    Student { Major: "Computer Science", Year: > 3 } => "Étudiant en dernière année d'informatique",
    Student { Year: var year } => $"Étudiant en année {year}",
    null => "Objet null",
    _ => "Objet inconnu"
};
```

### Utilisation avec la déconstruction

```csharp
string GetQuadrant(Point p) => p switch
{
    (0, 0) => "Origine",
    (> 0, > 0) => "Quadrant 1",
    (< 0, > 0) => "Quadrant 2",
    (< 0, < 0) => "Quadrant 3",
    (> 0, < 0) => "Quadrant 4",
    (0, _) => "Axe Y",
    (_, 0) => "Axe X",
    _ => "Impossible"    // Nécessaire même si tous les cas sont couverts
};
```

### Expressions switch pour les calculs

Les expressions switch sont très utiles pour remplacer les longues chaînes d'instructions `if-else if` lors du calcul de valeurs :

```csharp
// Calculer le nombre de jours dans un mois
int GetDaysInMonth(int month, bool isLeapYear) => month switch
{
    1 or 3 or 5 or 7 or 8 or 10 or 12 => 31,
    4 or 6 or 9 or 11 => 30,
    2 => isLeapYear ? 29 : 28,
    _ => throw new ArgumentOutOfRangeException(nameof(month), "Mois invalide")
};
```

### Utilisation avec les tuples

Vous pouvez utiliser des tuples pour faire correspondre plusieurs valeurs à la fois :

```csharp
string GetGameResult(string player1, string player2) => (player1, player2) switch
{
    ("pierre", "ciseaux") => "Joueur 1 gagne",
    ("papier", "pierre") => "Joueur 1 gagne",
    ("ciseaux", "papier") => "Joueur 1 gagne",
    ("pierre", "pierre") => "Égalité",
    ("papier", "papier") => "Égalité",
    ("ciseaux", "ciseaux") => "Égalité",
    _ => "Joueur 2 gagne"
};
```

### Astuces et bonnes pratiques

1. **Exhaustivité** : Les expressions switch doivent couvrir tous les cas possibles, sinon le compilateur génère une erreur. Utilisez le motif `_` pour couvrir les cas restants.

2. **Ordre des cas** : L'ordre des cas est important. Le premier cas correspondant est choisi.

3. **Exceptions** : Vous pouvez lancer des exceptions dans les expressions switch pour les cas invalides.

4. **Pas d'instructions, seulement des expressions** : Les expressions switch doivent retourner une valeur, pas exécuter des instructions.

5. **Lisibilité** : Les expressions switch peuvent améliorer la lisibilité pour les logiques complexes de branchement, mais n'abusez pas des fonctionnalités avancées si cela rend le code difficile à comprendre.

### Imbrication de conditionnels

Vous pouvez avoir des logiques conditionnelles complexes dans les résultats :

```csharp
decimal CalculerPrix(Product produit, Customer client) => produit switch
{
    { Category: "Électronique" } => client.IsPremium ? produit.Price * 0.9m : produit.Price,
    { Category: "Livres" } => produit.Price * 0.95m,
    { IsDiscounted: true } => produit.Price * 0.8m,
    _ => produit.Price
};
```

## Résumé

- Les **conditionnelles** (`if`, `else`, `switch`) permettent d'exécuter différents blocs de code selon des conditions.
- Les **boucles** (`for`, `while`, `do-while`, `foreach`) permettent de répéter des blocs de code.
- Les **instructions de saut** (`break`, `continue`, `return`) modifient le flux d'exécution normal.
- Le **pattern matching** permet d'examiner un objet et d'agir différemment selon son type, sa structure ou sa valeur.
- Les **expressions switch** offrent une syntaxe concise pour assigner des valeurs basées sur différents cas.

Ces structures de contrôle sont les outils fondamentaux qui vous permettent de créer des programmes C# dynamiques et réactifs. En maîtrisant ces concepts, vous serez capable de concevoir des solutions élégantes à des problèmes complexes.

⏭️ 2.6. [Enregistrement des commentaires et documentation](/02-fondamentaux-du-langage-csharp/2-6-enregistrement-des-commentaires-et-documentation.md)
