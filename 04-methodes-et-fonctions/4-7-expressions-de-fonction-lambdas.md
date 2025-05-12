# 4.7. Expressions de fonction (lambdas) en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les expressions lambda sont l'une des fonctionnalités les plus puissantes et flexibles de C#. Elles vous permettent de créer des fonctions anonymes de façon concise et élégante. Cette section vous guidera à travers les concepts fondamentaux des lambdas, leur syntaxe, et leurs applications pratiques.

## Introduction aux expressions lambda

Une expression lambda est essentiellement une fonction sans nom (anonyme) qui peut être utilisée comme une valeur - vous pouvez la stocker dans une variable, la passer comme argument à une méthode, ou la retourner comme résultat d'une méthode. Les lambdas sont particulièrement utiles pour créer des fonctions simples et éphémères sans avoir à définir une méthode complète.

Avant d'entrer dans les détails, voici un aperçu rapide d'une expression lambda simple :

```csharp
// Une lambda qui prend un entier et retourne son carré
x => x * x
```

Cette petite ligne de code crée une fonction qui prend un paramètre `x` et retourne `x * x`. L'opérateur `=>` (qui se lit "va vers" ou "lambda") sépare la liste des paramètres de l'expression ou du bloc de code qui constitue le corps de la fonction.

## 4.7.1. Syntaxe des expressions lambda

Il existe différentes façons d'écrire des expressions lambda en C#, selon la complexité de ce que vous voulez exprimer.

### Lambdas avec un paramètre

La forme la plus simple prend un seul paramètre et retourne une expression :

```csharp
// Un paramètre sans type spécifié (type inféré par le contexte)
x => x * x

// Un paramètre avec type explicite
(int x) => x * x
```

Si le compilateur peut inférer le type du paramètre (ce qui est souvent le cas), vous pouvez omettre le type et les parenthèses. Sinon, vous devez spécifier le type et entourer le paramètre de parenthèses.

### Lambdas avec plusieurs paramètres

Pour les lambdas qui prennent plusieurs paramètres, les parenthèses sont obligatoires :

```csharp
// Deux paramètres sans types spécifiés
(x, y) => x + y

// Deux paramètres avec types explicites
(int x, double y) => x + y

// Aucun paramètre
() => Console.WriteLine("Hello, World!")
```

### Corps d'expression vs. corps de bloc

Il existe deux façons d'écrire le corps d'une lambda :

1. **Corps d'expression** : Une expression simple qui devient automatiquement la valeur de retour de la lambda.

```csharp
x => x * x  // Corps d'expression : retourne x au carré
```

2. **Corps de bloc** : Un bloc de code avec accolades, similaire au corps d'une méthode. Si vous utilisez un corps de bloc, vous devez utiliser l'instruction `return` pour spécifier la valeur de retour (sauf si la lambda retourne `void`).

```csharp
x => {
    Console.WriteLine($"Calcul du carré de {x}");
    return x * x;
}
```

### Exemples complets

Voici quelques exemples montrant différentes formes de syntaxe lambda :

```csharp
// Lambda sans paramètres
Action dire = () => Console.WriteLine("Bonjour!");

// Lambda avec un paramètre
Func<int, int> carre = x => x * x;

// Lambda avec type de paramètre explicite
Func<int, bool> estPair = (int x) => x % 2 == 0;

// Lambda avec plusieurs paramètres
Func<int, int, int> additionner = (x, y) => x + y;

// Lambda avec corps de bloc
Func<int, int> calculerCube = x => {
    var resultat = x * x * x;
    Console.WriteLine($"Le cube de {x} est {resultat}");
    return resultat;
};

// Lambda qui ne retourne rien (void)
Action<string> afficher = message => {
    Console.WriteLine("**************");
    Console.WriteLine(message);
    Console.WriteLine("**************");
};
```

### Inférence de type dans les lambdas

Le compilateur C# peut généralement inférer le type des paramètres d'une lambda en fonction du contexte dans lequel elle est utilisée. Par exemple, si vous assignez une lambda à une variable de type `Func<int, bool>`, le compilateur sait que le paramètre de la lambda est un `int` et que la lambda retourne un `bool`.

```csharp
// Le compilateur infère que x est un int
Func<int, bool> estPositif = x => x > 0;

// En comparaison avec le type explicite (moins concis)
Func<int, bool> estPositifExplicite = (int x) => x > 0;
```

## 4.7.2. Lambdas comme délégués et expressions

Les expressions lambda en C# sont étroitement liées aux types délégués et aux expressions tree. Comprendre ces relations est essentiel pour utiliser efficacement les lambdas.

### Lambdas comme délégués

En C#, les expressions lambda sont principalement utilisées pour créer des instances de types délégués. Un délégué est essentiellement un type qui représente une référence à une méthode. C# fournit plusieurs types délégués génériques intégrés qui sont couramment utilisés avec les lambdas :

- **`Action`** : Pour les méthodes qui ne retournent pas de valeur (void)
- **`Func`** : Pour les méthodes qui retournent une valeur
- **`Predicate`** : Un cas spécial de `Func` qui prend un paramètre et retourne un booléen

Voici comment assigner des lambdas à ces types de délégués :

```csharp
// Action - ne retourne pas de valeur
Action dire = () => Console.WriteLine("Bonjour!");
Action<string> saluer = nom => Console.WriteLine($"Bonjour, {nom}!");
Action<string, int> presenter = (nom, age) =>
    Console.WriteLine($"Je m'appelle {nom} et j'ai {age} ans.");

// Func - retourne une valeur
Func<int, int> carre = x => x * x;
Func<int, int, bool> estPlusPetit = (x, y) => x < y;
Func<string, string, string> concatener = (s1, s2) => s1 + s2;

// Predicate - retourne un booléen
Predicate<int> estPair = x => x % 2 == 0;
```

Une fois que vous avez assigné une lambda à une variable de délégué, vous pouvez l'invoquer comme une méthode normale :

```csharp
dire();                    // Affiche: Bonjour!
saluer("Marie");           // Affiche: Bonjour, Marie!
presenter("Jean", 30);     // Affiche: Je m'appelle Jean et j'ai 30 ans.

int resultat = carre(5);   // resultat = 25
bool test = estPlusPetit(3, 7);  // test = true
string nom = concatener("John", " Doe");  // nom = "John Doe"

bool pair = estPair(4);    // pair = true
```

### Utilisation des lambdas avec des méthodes

Les lambdas sont souvent passées comme arguments aux méthodes qui acceptent des délégués. Le cas d'utilisation le plus courant est probablement avec les méthodes LINQ :

```csharp
List<int> nombres = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Utilisation de lambda avec Where (qui prend un Func<int, bool>)
var nombresPairs = nombres.Where(n => n % 2 == 0);

// Utilisation de lambda avec Select (qui prend un Func<int, TResult>)
var carres = nombres.Select(n => n * n);

// Combinaison de méthodes LINQ avec lambdas
var sommeCarresPairs = nombres
    .Where(n => n % 2 == 0)
    .Select(n => n * n)
    .Sum();

Console.WriteLine($"Somme des carrés des nombres pairs : {sommeCarresPairs}");
// Affiche: Somme des carrés des nombres pairs : 220 (= 4 + 16 + 36 + 64 + 100)
```

### Lambdas comme expressions tree

En plus d'être compilées en délégués, les expressions lambda peuvent également être converties en arbres d'expressions (expression trees). Les arbres d'expressions représentent du code sous forme de structure de données que vous pouvez examiner et manipuler au runtime.

Cette fonctionnalité est principalement utilisée dans les scénarios avancés comme ORM (Object-Relational Mapping) où le code doit être traduit en autre chose (par exemple, des requêtes SQL). Entity Framework utilise cette fonctionnalité pour convertir vos lambdas LINQ en requêtes SQL.

Voici un exemple simple d'arbre d'expression :

```csharp
using System.Linq.Expressions;

// Création d'un arbre d'expression qui représente x => x + 1
Expression<Func<int, int>> expr = x => x + 1;

// Nous pouvons analyser cette expression
Console.WriteLine(expr.ToString());  // x => (x + 1)

// On peut aussi la compiler en délégué et l'exécuter
Func<int, int> fonction = expr.Compile();
int resultat = fonction(3);  // resultat = 4
```

Dans cet exemple :
- `x => x + 1` est une expression lambda
- En l'assignant à `Expression<Func<int, int>>`, elle est capturée comme un arbre d'expression
- Nous pouvons ensuite la compiler en un délégué exécutable avec la méthode `Compile()`

La plupart du temps, vous n'aurez pas besoin de travailler directement avec les arbres d'expressions, mais il est bon de savoir qu'ils existent et pourquoi des bibliothèques comme Entity Framework imposent certaines restrictions sur les lambdas que vous pouvez utiliser.

## 4.7.3. Capture de variables (closures)

L'une des caractéristiques les plus puissantes des expressions lambda est leur capacité à "capturer" des variables de la portée englobante. Cette fonctionnalité, appelée "closure" (fermeture), permet à une lambda d'accéder et de manipuler des variables définies en dehors de son propre corps.

### Concept de base des closures

Une closure se produit lorsqu'une fonction (comme une lambda) "capture" et conserve l'accès à des variables de sa portée environnante, même après que cette portée aurait normalement expiré.

Voici un exemple simple :

```csharp
public Func<int, int> CreerMultiplicateur(int facteur)
{
    // Cette lambda capture la variable 'facteur' de la méthode englobante
    return x => x * facteur;
}

// Utilisation
var multiplierPar2 = CreerMultiplicateur(2);
var multiplierPar10 = CreerMultiplicateur(10);

Console.WriteLine(multiplierPar2(5));   // Affiche: 10 (5 × 2)
Console.WriteLine(multiplierPar10(5));  // Affiche: 50 (5 × 10)
```

Dans cet exemple :
1. La méthode `CreerMultiplicateur` définit une variable locale `facteur`
2. La lambda `x => x * facteur` capture cette variable
3. La méthode retourne la lambda et termine son exécution
4. Normalement, `facteur` serait détruit, mais grâce à la closure, la lambda maintient son accès à la variable

Ainsi, même après que `CreerMultiplicateur` ait terminé son exécution, les lambdas `multiplierPar2` et `multiplierPar10` ont toujours accès à la valeur de `facteur` qu'elles ont capturée (respectivement 2 et 10).

### Capture de variables par référence

Il est crucial de comprendre que les variables sont capturées par référence, et non par valeur. Cela signifie que si la valeur de la variable capturée change après la création de la lambda mais avant son exécution, la lambda verra la nouvelle valeur.

```csharp
public Action[] CreerCompteurs()
{
    Action[] compteurs = new Action[3];

    for (int i = 0; i < 3; i++)
    {
        // Cette lambda capture la variable 'i' de la boucle
        compteurs[i] = () => Console.WriteLine(i);
    }

    return compteurs;
}

// Utilisation
Action[] mesCompteurs = CreerCompteurs();

// Exécution des actions
foreach (var compteur in mesCompteurs)
{
    compteur();  // Que va afficher ceci ?
}
```

Si vous pensez que ce code va afficher 0, 1, 2, vous serez surpris ! En réalité, il affichera 3, 3, 3.

Pourquoi ? Parce que :
1. Les trois lambdas capturent la même variable `i` par référence
2. À la fin de la boucle, `i` a la valeur 3 (condition de sortie de la boucle)
3. Lorsque les lambdas sont exécutées, elles voient toutes la valeur finale de `i`, qui est 3

Pour résoudre ce problème en C# avant la version 5.0, vous deviez créer une variable locale à chaque itération :

```csharp
public Action[] CreerCompteursCorrectement()
{
    Action[] compteurs = new Action[3];

    for (int i = 0; i < 3; i++)
    {
        // Création d'une copie locale de i
        int copieLocale = i;
        // La lambda capture maintenant 'copieLocale', pas 'i'
        compteurs[i] = () => Console.WriteLine(copieLocale);
    }

    return compteurs;
}
```

À partir de C# 5.0, le comportement des variables de boucle `foreach` a été modifié pour éviter ce problème spécifique. Et à partir de C# 7.0, les variables d'itération de la boucle `for` sont également considérées comme réinitialisées à chaque itération.

### Modification des variables capturées

Une lambda peut non seulement lire les variables capturées, mais aussi les modifier. Ces modifications sont visibles pour tout code ayant accès à ces variables.

```csharp
public class Compteur
{
    public Action Incrementer { get; private set; }
    public Action Afficher { get; private set; }
    public Func<int> ObtenirValeur { get; private set; }

    public Compteur()
    {
        int count = 0;  // Variable locale qui sera capturée

        // Définition des lambdas qui capturent et manipulent 'count'
        Incrementer = () => count++;
        Afficher = () => Console.WriteLine($"Valeur actuelle : {count}");
        ObtenirValeur = () => count;
    }
}

// Utilisation
var compteur = new Compteur();
compteur.Afficher();    // Affiche: Valeur actuelle : 0
compteur.Incrementer();
compteur.Incrementer();
compteur.Afficher();    // Affiche: Valeur actuelle : 2
int valeur = compteur.ObtenirValeur();  // valeur = 2
```

Dans cet exemple, les trois lambdas partagent l'accès à la même variable `count`. Lorsque `Incrementer` modifie `count`, les autres lambdas voient ces modifications.

Cette capacité permet de créer des fermetures utiles comme des compteurs, des accumulateurs, ou des caches qui maintiennent un état interne.

### Captures et durée de vie des objets

Lorsqu'une lambda capture une variable, cela prolonge la durée de vie de cette variable aussi longtemps que la lambda existe. Cela peut parfois conduire à des problèmes, en particulier si la lambda capture (directement ou indirectement) une référence à un objet volumineux qui n'est plus nécessaire.

```csharp
public Func<int, int> CreerProcesseurAvecDonneesCaptureesInutilement()
{
    // Imaginons que ceci est un très gros objet
    var grosDictionnaire = new Dictionary<string, string>(1000000);
    for (int i = 0; i < 1000000; i++)
    {
        grosDictionnaire[$"clé{i}"] = $"valeur{i}";
    }

    int multiplicateur = 10;

    // Cette lambda capture 'multiplicateur', mais aussi involontairement 'grosDictionnaire'
    // car ils sont tous deux dans la même portée
    return x => multiplicateur * x;
}
```

Dans cet exemple, `grosDictionnaire` n'est pas utilisé dans la lambda, mais il reste en mémoire aussi longtemps que la lambda existe, car il fait partie de la même closure.

Pour éviter ce problème, vous pouvez isoler les variables nécessaires :

```csharp
public Func<int, int> CreerProcesseurSansCapturerInutilement()
{
    // Création du gros dictionnaire...
    var grosDictionnaire = new Dictionary<string, string>(1000000);
    // ...

    int multiplicateur = 10;

    // Isolement de la valeur nécessaire
    return IsolerMultiplicateur(multiplicateur);

    // Fonction locale pour isoler la capture
    Func<int, int> IsolerMultiplicateur(int m)
    {
        return x => m * x;  // Ne capture que 'm', pas 'grosDictionnaire'
    }
}
```

Cette technique garantit que seules les variables nécessaires sont capturées dans la closure.

### Cas d'utilisation pratiques des closures

Les closures sont particulièrement utiles dans plusieurs scénarios :

#### 1. Créer des fonctions configurables

```csharp
public class FormatteurTexte
{
    public Func<string, string> CreerFormatteur(string prefix, string suffix)
    {
        return texte => $"{prefix}{texte}{suffix}";
    }
}

// Utilisation
var formatteur = new FormatteurTexte();
var accentuer = formatteur.CreerFormatteur("** ", " **");
var citer = formatteur.CreerFormatteur("\"", "\"");

Console.WriteLine(accentuer("Important"));  // Affiche: ** Important **
Console.WriteLine(citer("Citation"));       // Affiche: "Citation"
```

#### 2. Mémorisation (Caching)

```csharp
public class CalculateurAvecCache
{
    public Func<int, int> CreerCalculateurFibonacciAvecCache()
    {
        // Dictionnaire pour stocker les résultats déjà calculés
        var cache = new Dictionary<int, int>();

        // Déclaration pour permettre la récursion
        Func<int, int> fib = null;

        // Définition de la fonction récursive avec cache
        fib = n =>
        {
            if (n <= 1)
                return n;

            // Vérifie si le résultat est déjà dans le cache
            if (cache.TryGetValue(n, out int resultat))
                return resultat;

            // Calcule et stocke le résultat
            resultat = fib(n - 1) + fib(n - 2);
            cache[n] = resultat;
            return resultat;
        };

        return fib;
    }
}

// Utilisation
var calculateur = new CalculateurAvecCache();
var fibonacci = calculateur.CreerCalculateurFibonacciAvecCache();

Console.WriteLine(fibonacci(10));  // Calcule et met en cache
Console.WriteLine(fibonacci(10));  // Utilise la valeur en cache
Console.WriteLine(fibonacci(20));  // Utilise les sous-résultats déjà en cache
```

#### 3. Gestion d'événements et rappels

```csharp
public class GestionnaireUI
{
    private List<Button> _boutons = new List<Button>();

    public void AjouterBoutonAvecID(string texte, int id)
    {
        var bouton = new Button { Text = texte };

        // La lambda capture 'id' pour l'utiliser lors du clic
        bouton.Click += (sender, e) => {
            Console.WriteLine($"Bouton {id} cliqué : {texte}");
            TraiterAction(id);
        };

        _boutons.Add(bouton);
    }

    private void TraiterAction(int id)
    {
        // Traitement basé sur l'ID...
    }
}
```

### Pièges courants avec les closures

#### 1. Modification de variables capturées dans des boucles (déjà traité plus haut)

#### 2. Capture involontaire de `this`

Dans les méthodes d'instance, l'utilisation d'une lambda capture implicitement `this`, ce qui peut parfois causer des problèmes de cycle de vie ou de mémoire :

```csharp
public class Exemple
{
    private string _message = "Bonjour";

    // Cette méthode retourne une lambda qui capture implicitement 'this'
    public Func<string> CreerAfficheurMessage()
    {
        return () => _message;  // Capture implicitement 'this' pour accéder à _message
    }
}
```

Si vous voulez éviter de capturer `this`, vous pouvez copier les valeurs nécessaires dans des variables locales :

```csharp
public class Exemple
{
    private string _message = "Bonjour";

    public Func<string> CreerAfficheurMessageSansThis()
    {
        // Copie la valeur dans une variable locale
        string messageCopie = _message;
        return () => messageCopie;  // Ne capture pas 'this'
    }
}
```

#### 3. Mutabilité et concurrence

Si une closure modifie des variables capturées et est utilisée dans un contexte multithread, vous devez faire attention aux problèmes de concurrence :

```csharp
public class ExempleConcurrence
{
    public void DemonstrationProblemeConcurrence()
    {
        int compteur = 0;

        // Création de 10 tâches qui incrémentent le même compteur
        var taches = new List<Task>();
        for (int i = 0; i < 10; i++)
        {
            taches.Add(Task.Run(() => {
                for (int j = 0; j < 1000; j++)
                {
                    compteur++;  // Problème: accès non synchronisé à 'compteur'
                }
            }));
        }

        Task.WaitAll(taches.ToArray());
        Console.WriteLine($"Valeur finale : {compteur}");
        // La valeur sera probablement inférieure à 10000 en raison des conditions de concurrence
    }
}
```

Pour résoudre ce problème, utilisez les mécanismes de synchronisation appropriés comme `lock` ou les classes de la bibliothèque `System.Threading` :

```csharp
public void DemonstrationConcurrenceCorrecte()
{
    int compteur = 0;
    object verrou = new object();

    var taches = new List<Task>();
    for (int i = 0; i < 10; i++)
    {
        taches.Add(Task.Run(() => {
            for (int j = 0; j < 1000; j++)
            {
                lock (verrou)
                {
                    compteur++;  // Accès synchronisé
                }
            }
        }));
    }

    Task.WaitAll(taches.ToArray());
    Console.WriteLine($"Valeur finale : {compteur}");  // Sera toujours 10000
}
```

## Lambdas vs méthodes anonymes

Avant les lambdas (introduites dans C# 3.0), C# utilisait la syntaxe des "méthodes anonymes" pour créer des fonctions anonymes. Bien que cette syntaxe soit maintenant considérée comme obsolète en faveur des lambdas, vous pourriez la rencontrer dans du code existant :

```csharp
// Méthode anonyme (ancienne syntaxe)
Func<int, int> carreAncien = delegate(int x) { return x * x; };

// Expression lambda équivalente (nouvelle syntaxe)
Func<int, int> carreNouveau = x => x * x;
```

Les lambdas sont généralement plus concises et plus lisibles, surtout pour les fonctions simples.

## Conclusion

Les expressions lambda sont un outil incroyablement puissant dans la boîte à outils du développeur C#. Elles permettent d'écrire du code plus concis, plus expressif et parfois plus performant.

Points clés à retenir :

1. **Syntaxe** : Les lambdas utilisent l'opérateur `=>` pour séparer les paramètres du corps de la fonction.

2. **Délégués** : Les lambdas sont principalement utilisées pour créer des instances de délégués (`Action`, `Func`, `Predicate`, etc.).

3. **Expressions tree** : Les lambdas peuvent être converties en arbres d'expressions pour une analyse et une manipulation au runtime.

4. **Closures** : Les lambdas peuvent capturer et conserver l'accès aux variables de la portée englobante, ce qui permet de créer des fonctions avec état.

5. **Pièges** : Faites attention à la capture de variables dans les boucles, à la capture implicite de `this`, et aux problèmes de concurrence.

Les expressions lambda sont particulièrement utiles dans les scénarios tels que :
- Programmation fonctionnelle
- Méthodes LINQ
- Manipulation d'événements
- Callbacks et handlers asynchrones
- Création de fonctions configurables

En maîtrisant les expressions lambda, vous pourrez écrire un code C# plus élégant, plus expressif et plus modulaire.

⏭️ 5. [Programmation orientée objet en C#](/05-programmation-orientee-objet-en-csharp/README.md)
