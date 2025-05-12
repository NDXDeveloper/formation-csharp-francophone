# 3.2. Collections standard en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les collections standard sont des structures de données prédéfinies dans le framework .NET qui permettent de stocker et manipuler des groupes d'objets associés. Contrairement aux tableaux qui ont une taille fixe, la plupart des collections peuvent s'agrandir ou se rétrécir dynamiquement au fur et à mesure que vous ajoutez ou supprimez des éléments.

Ces collections se trouvent dans l'espace de noms `System.Collections.Generic` et utilisent des types génériques pour offrir un typage fort et éviter les conversions (casting).

```csharp
// N'oubliez pas d'inclure cet espace de noms au début de votre fichier
using System.Collections.Generic;
```

## 3.2.1. List<T>

`List<T>` est sans doute la collection la plus utilisée en C#. Elle représente une liste d'éléments fortement typés, accessible par index et redimensionnable dynamiquement.

### Création et initialisation

```csharp
// Création d'une liste vide d'entiers
List<int> nombres = new List<int>();

// Création avec une capacité initiale (pour performance)
List<string> noms = new List<string>(100);

// Création et initialisation simultanées (C# 3.0+)
List<double> prix = new List<double> { 9.99, 14.50, 19.99, 24.75 };

// Création à partir d'un tableau ou d'une autre collection existante
string[] tableauFruits = { "Pomme", "Banane", "Orange" };
List<string> fruits = new List<string>(tableauFruits);
```

### Ajouter et insérer des éléments

```csharp
List<string> courses = new List<string>();

// Ajouter des éléments à la fin de la liste
courses.Add("Pain");
courses.Add("Lait");
courses.Add("Œufs");

// Ajouter plusieurs éléments à la fois
courses.AddRange(new[] { "Fromage", "Jambon" });

// Insérer un élément à une position spécifique
courses.Insert(1, "Beurre");  // Insère "Beurre" à l'index 1

// Insérer plusieurs éléments à une position spécifique
courses.InsertRange(3, new[] { "Salade", "Tomates" });
```

### Accéder aux éléments

```csharp
List<string> fruits = new List<string> { "Pomme", "Banane", "Orange", "Fraise", "Kiwi" };

// Accès par index (comme les tableaux)
Console.WriteLine(fruits[0]);  // Affiche "Pomme"
Console.WriteLine(fruits[2]);  // Affiche "Orange"

// Modifier un élément
fruits[1] = "Ananas";
Console.WriteLine(fruits[1]);  // Affiche "Ananas"

// Attention : l'accès à un index inexistant lève une exception
// Console.WriteLine(fruits[10]);  // Exception: Index out of range
```

### Parcourir une liste

```csharp
List<string> fruits = new List<string> { "Pomme", "Banane", "Orange", "Fraise", "Kiwi" };

// 1. Avec foreach (méthode recommandée pour la lisibilité)
Console.WriteLine("Parcours avec foreach:");
foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}

// 2. Avec for classique
Console.WriteLine("\nParcours avec for:");
for (int i = 0; i < fruits.Count; i++)  // Notez Count au lieu de Length
{
    Console.WriteLine($"{i}: {fruits[i]}");
}

// 3. Avec ForEach et une expression lambda (à partir de .NET 2.0)
Console.WriteLine("\nParcours avec ForEach et lambda:");
fruits.ForEach(fruit => Console.WriteLine(fruit));
```

### Rechercher des éléments

```csharp
List<string> fruits = new List<string> { "Pomme", "Banane", "Orange", "Fraise", "Kiwi", "Orange" };

// Vérifier si un élément existe
bool contientBanane = fruits.Contains("Banane");
Console.WriteLine($"La liste contient Banane: {contientBanane}");  // True

// Trouver l'index d'un élément
int indexOrange = fruits.IndexOf("Orange");
Console.WriteLine($"Première occurrence de Orange: {indexOrange}");  // 2

// Trouver l'index de la dernière occurrence
int dernierIndexOrange = fruits.LastIndexOf("Orange");
Console.WriteLine($"Dernière occurrence de Orange: {dernierIndexOrange}");  // 5

// Trouver le premier élément qui correspond à une condition
string premierFruitAvecA = fruits.Find(f => f.Contains("a"));
Console.WriteLine($"Premier fruit avec 'a': {premierFruitAvecA}");  // "Banane"

// Trouver tous les éléments qui correspondent à une condition
List<string> fruitsAvecI = fruits.FindAll(f => f.Contains("i"));
Console.WriteLine("Fruits avec 'i':");
foreach (string fruit in fruitsAvecI)
{
    Console.WriteLine(fruit);  // Affiche "Kiwi", "Fraise"
}
```

### Supprimer des éléments

```csharp
List<string> fruits = new List<string> { "Pomme", "Banane", "Orange", "Fraise", "Kiwi", "Orange" };

// Supprimer un élément spécifique (première occurrence)
fruits.Remove("Orange");
Console.WriteLine(string.Join(", ", fruits));  // "Pomme, Banane, Fraise, Kiwi, Orange"

// Supprimer l'élément à un index spécifique
fruits.RemoveAt(1);  // Supprime "Banane"
Console.WriteLine(string.Join(", ", fruits));  // "Pomme, Fraise, Kiwi, Orange"

// Supprimer tous les éléments qui correspondent à un prédicat
fruits.RemoveAll(f => f.Length > 5);  // Supprime "Fraise", "Orange"
Console.WriteLine(string.Join(", ", fruits));  // "Pomme, Kiwi"

// Vider la liste
fruits.Clear();
Console.WriteLine($"Nombre d'éléments après Clear: {fruits.Count}");  // 0
```

### Trier une liste

```csharp
List<int> nombres = new List<int> { 5, 2, 8, 1, 9, 3 };

// Tri par ordre croissant
nombres.Sort();
Console.WriteLine(string.Join(", ", nombres));  // "1, 2, 3, 5, 8, 9"

// Tri des chaînes
List<string> noms = new List<string> { "Zoé", "Alice", "Bob", "Charlie", "David" };
noms.Sort();
Console.WriteLine(string.Join(", ", noms));  // "Alice, Bob, Charlie, David, Zoé"

// Tri personnalisé avec un Comparison<T>
noms.Sort((a, b) => b.CompareTo(a));  // Tri descendant
Console.WriteLine(string.Join(", ", noms));  // "Zoé, David, Charlie, Bob, Alice"

// Tri d'objets complexes
List<Personne> personnes = new List<Personne>
{
    new Personne { Nom = "Dupont", Age = 25 },
    new Personne { Nom = "Martin", Age = 40 },
    new Personne { Nom = "Durand", Age = 30 }
};

// Tri par âge
personnes.Sort((p1, p2) => p1.Age.CompareTo(p2.Age));
foreach (var p in personnes)
{
    Console.WriteLine($"{p.Nom} ({p.Age} ans)");
}
```

### Autres opérations utiles

```csharp
List<int> nombres = new List<int> { 1, 2, 3, 4, 5 };

// Obtenir le nombre d'éléments
Console.WriteLine($"Nombre d'éléments: {nombres.Count}");

// Obtenir la capacité actuelle
Console.WriteLine($"Capacité: {nombres.Capacity}");

// Convertir en tableau
int[] tableauNombres = nombres.ToArray();

// Redimensionner la liste (tronquer ou ajouter des éléments par défaut)
nombres.AddRange(new[] { 6, 7, 8, 9, 10 });  // Ajoute plus d'éléments
nombres.RemoveRange(7, 3);  // Supprime 3 éléments à partir de l'index 7

// Inverser l'ordre des éléments
nombres.Reverse();
Console.WriteLine(string.Join(", ", nombres));

// Créer une copie superficielle de la liste
List<int> copie = new List<int>(nombres);
```

### Quand utiliser List<T>

- Lorsque vous avez besoin d'une collection d'éléments accessibles par index
- Quand vous devez ajouter ou supprimer des éléments dynamiquement
- Si vous avez besoin de trier fréquemment les éléments
- Quand l'ordre des éléments est important

## 3.2.2. Dictionary<TKey, TValue>

`Dictionary<TKey, TValue>` est une collection de paires clé-valeur, où chaque clé est unique. Elle permet d'accéder rapidement aux valeurs à l'aide de leur clé, sans avoir à parcourir toute la collection.

### Création et initialisation

```csharp
// Création d'un dictionnaire vide
Dictionary<string, int> ages = new Dictionary<string, int>();

// Création et initialisation simultanées
Dictionary<string, string> capitales = new Dictionary<string, string>
{
    { "France", "Paris" },
    { "Allemagne", "Berlin" },
    { "Italie", "Rome" },
    { "Espagne", "Madrid" }
};

// Syntaxe alternative d'initialisation (C# 6.0+)
Dictionary<string, string> traductions = new Dictionary<string, string>
{
    ["hello"] = "bonjour",
    ["goodbye"] = "au revoir",
    ["thank you"] = "merci"
};
```

### Ajouter et modifier des éléments

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();

// Ajouter des éléments
ages.Add("Alice", 30);
ages.Add("Bob", 25);
ages.Add("Charlie", 35);

// Syntaxe alternative pour ajouter ou modifier un élément
ages["David"] = 28;  // Ajoute une nouvelle paire clé-valeur
ages["Alice"] = 31;  // Modifie la valeur existante

// Vérifier si une clé existe avant d'ajouter
if (!ages.ContainsKey("Eve"))
{
    ages.Add("Eve", 22);
}

// Utiliser TryAdd (C# 7.0+)
bool ajouté = ages.TryAdd("Frank", 40);
Console.WriteLine($"Frank ajouté: {ajouté}");  // True
```

### Accéder aux valeurs

```csharp
Dictionary<string, string> capitales = new Dictionary<string, string>
{
    { "France", "Paris" },
    { "Allemagne", "Berlin" },
    { "Italie", "Rome" },
    { "Espagne", "Madrid" }
};

// Accès direct (peut lever KeyNotFoundException si la clé n'existe pas)
string capitaleItalie = capitales["Italie"];
Console.WriteLine($"La capitale de l'Italie est {capitaleItalie}");

// Vérifier si une clé existe avant d'accéder
if (capitales.ContainsKey("Portugal"))
{
    Console.WriteLine($"La capitale du Portugal est {capitales["Portugal"]}");
}
else
{
    Console.WriteLine("Le Portugal n'est pas dans le dictionnaire.");
}

// Utiliser TryGetValue (recommandé)
if (capitales.TryGetValue("Espagne", out string capitaleEspagne))
{
    Console.WriteLine($"La capitale de l'Espagne est {capitaleEspagne}");
}
else
{
    Console.WriteLine("L'Espagne n'est pas dans le dictionnaire.");
}
```

### Parcourir un dictionnaire

```csharp
Dictionary<string, string> capitales = new Dictionary<string, string>
{
    { "France", "Paris" },
    { "Allemagne", "Berlin" },
    { "Italie", "Rome" },
    { "Espagne", "Madrid" }
};

// 1. Parcourir les paires clé-valeur
Console.WriteLine("Parcours des paires clé-valeur:");
foreach (KeyValuePair<string, string> paire in capitales)
{
    Console.WriteLine($"{paire.Key}: {paire.Value}");
}

// 2. Avec déconstruction (C# 7.0+)
Console.WriteLine("\nParcours avec déconstruction:");
foreach (var (pays, capitale) in capitales)
{
    Console.WriteLine($"{pays}: {capitale}");
}

// 3. Parcourir uniquement les clés
Console.WriteLine("\nParcours des clés:");
foreach (string pays in capitales.Keys)
{
    Console.WriteLine(pays);
}

// 4. Parcourir uniquement les valeurs
Console.WriteLine("\nParcours des valeurs:");
foreach (string capitale in capitales.Values)
{
    Console.WriteLine(capitale);
}
```

### Supprimer des éléments

```csharp
Dictionary<string, string> capitales = new Dictionary<string, string>
{
    { "France", "Paris" },
    { "Allemagne", "Berlin" },
    { "Italie", "Rome" },
    { "Espagne", "Madrid" }
};

// Supprimer un élément par sa clé
bool supprimé = capitales.Remove("Italie");
Console.WriteLine($"Italie supprimée: {supprimé}");

// Vérifier si une clé existe avant de la supprimer
if (capitales.ContainsKey("Portugal"))
{
    capitales.Remove("Portugal");
}

// Supprimer tous les éléments
capitales.Clear();
Console.WriteLine($"Nombre d'éléments après Clear: {capitales.Count}");  // 0
```

### Exemple pratique

```csharp
// Dictionnaire pour compter la fréquence de mots dans un texte
string texte = "C'est un exemple de texte. C'est un exemple simple pour illustrer les dictionnaires.";

// Séparer le texte en mots
string[] mots = texte.ToLower()
                     .Replace(".", "")
                     .Replace(",", "")
                     .Replace("!", "")
                     .Replace("?", "")
                     .Split(' ');

// Compter les occurrences de chaque mot
Dictionary<string, int> fréquence = new Dictionary<string, int>();

foreach (string mot in mots)
{
    if (fréquence.ContainsKey(mot))
    {
        fréquence[mot]++;
    }
    else
    {
        fréquence[mot] = 1;
    }
}

// Afficher les résultats
Console.WriteLine("Fréquence des mots:");
foreach (var paire in fréquence.OrderByDescending(p => p.Value))
{
    Console.WriteLine($"'{paire.Key}': {paire.Value} occurrence(s)");
}
```

### Quand utiliser Dictionary<TKey, TValue>

- Lorsque vous avez besoin d'associer des valeurs à des clés uniques
- Pour un accès rapide aux données par une clé (recherche en O(1) en moyenne)
- Quand vous devez effectuer des recherches fréquentes
- Pour implémenter des tables de hachage, des caches ou des index

## 3.2.3. HashSet<T>

`HashSet<T>` est une collection d'éléments uniques, sans ordre particulier, optimisée pour les tests d'appartenance (vérifier si un élément existe dans l'ensemble).

### Création et initialisation

```csharp
// Création d'un HashSet vide
HashSet<int> ensemble1 = new HashSet<int>();

// Création à partir d'une collection existante
int[] nombres = { 1, 2, 3, 2, 4, 5, 3, 1 };
HashSet<int> ensemble2 = new HashSet<int>(nombres);
Console.WriteLine($"Nombres uniques: {ensemble2.Count}");  // 5 (les doublons sont éliminés)

// Création et initialisation simultanées
HashSet<string> couleurs = new HashSet<string> { "Rouge", "Vert", "Bleu", "Jaune" };
```

### Ajouter et supprimer des éléments

```csharp
HashSet<string> fruits = new HashSet<string>();

// Ajouter des éléments
fruits.Add("Pomme");
fruits.Add("Banane");
fruits.Add("Orange");

// Tenter d'ajouter un doublon (sera ignoré)
bool ajoutDoublon = fruits.Add("Pomme");
Console.WriteLine($"Ajout du doublon réussi: {ajoutDoublon}");  // False

// Ajouter plusieurs éléments d'un coup
fruits.UnionWith(new[] { "Fraise", "Kiwi", "Orange" });
Console.WriteLine($"Nombre de fruits: {fruits.Count}");  // 5 (pas de doublons)

// Supprimer un élément
bool supprimé = fruits.Remove("Banane");
Console.WriteLine($"Banane supprimée: {supprimé}");  // True

// Vider l'ensemble
fruits.Clear();
```

### Opérations d'ensemble

L'un des grands avantages de `HashSet<T>` est sa capacité à effectuer des opérations d'ensemble mathématiques efficaces.

```csharp
HashSet<int> ensemble1 = new HashSet<int> { 1, 2, 3, 4, 5 };
HashSet<int> ensemble2 = new HashSet<int> { 4, 5, 6, 7, 8 };

// Union (tous les éléments de A ou B)
HashSet<int> union = new HashSet<int>(ensemble1);
union.UnionWith(ensemble2);
Console.WriteLine($"Union: {string.Join(", ", union)}");  // 1, 2, 3, 4, 5, 6, 7, 8

// Intersection (éléments présents dans A et B)
HashSet<int> intersection = new HashSet<int>(ensemble1);
intersection.IntersectWith(ensemble2);
Console.WriteLine($"Intersection: {string.Join(", ", intersection)}");  // 4, 5

// Différence (éléments dans A mais pas dans B)
HashSet<int> différence = new HashSet<int>(ensemble1);
différence.ExceptWith(ensemble2);
Console.WriteLine($"Différence (A-B): {string.Join(", ", différence)}");  // 1, 2, 3

// Différence symétrique (éléments dans A ou B mais pas les deux)
HashSet<int> différenceSymétrique = new HashSet<int>(ensemble1);
différenceSymétrique.SymmetricExceptWith(ensemble2);
Console.WriteLine($"Différence symétrique: {string.Join(", ", différenceSymétrique)}");  // 1, 2, 3, 6, 7, 8
```

### Tests et vérifications

```csharp
HashSet<int> ensemble1 = new HashSet<int> { 1, 2, 3, 4, 5 };
HashSet<int> ensemble2 = new HashSet<int> { 2, 3, 4 };
HashSet<int> ensemble3 = new HashSet<int> { 6, 7, 8 };

// Vérifier si un élément existe
bool contient4 = ensemble1.Contains(4);
Console.WriteLine($"Ensemble1 contient 4: {contient4}");  // True

// Vérifier si un ensemble est un sous-ensemble d'un autre
bool estSousEnsemble = ensemble2.IsSubsetOf(ensemble1);
Console.WriteLine($"Ensemble2 est sous-ensemble de Ensemble1: {estSousEnsemble}");  // True

// Vérifier si un ensemble est un sur-ensemble d'un autre
bool estSurEnsemble = ensemble1.IsSupersetOf(ensemble2);
Console.WriteLine($"Ensemble1 est sur-ensemble de Ensemble2: {estSurEnsemble}");  // True

// Vérifier si deux ensembles n'ont aucun élément en commun
bool disjoints = ensemble1.IsProperSubsetOf(ensemble3);
Console.WriteLine($"Ensemble1 et Ensemble3 sont disjoints: {disjoints}");  // False
```

### Parcourir un HashSet

```csharp
HashSet<string> couleurs = new HashSet<string> { "Rouge", "Vert", "Bleu", "Jaune" };

// Parcours avec foreach
foreach (string couleur in couleurs)
{
    Console.WriteLine(couleur);
}

// Convertir en tableau ou liste si nécessaire
string[] tableauCouleurs = couleurs.ToArray();
List<string> listeCouleurs = couleurs.ToList();
```

### Exemple pratique

```csharp
// Trouver les caractères uniques dans une chaîne
string texte = "abracadabra";
HashSet<char> caractèresUniques = new HashSet<char>(texte);
Console.WriteLine($"Caractères uniques: {string.Join(", ", caractèresUniques)}");  // a, b, r, c, d

// Vérifier si deux tableaux ont des éléments en commun
int[] tableau1 = { 1, 2, 3, 4, 5 };
int[] tableau2 = { 4, 5, 6, 7, 8 };

HashSet<int> ensemble1 = new HashSet<int>(tableau1);
HashSet<int> intersection = new HashSet<int>(ensemble1);
intersection.IntersectWith(tableau2);

if (intersection.Count > 0)
{
    Console.WriteLine($"Éléments communs: {string.Join(", ", intersection)}");
}
else
{
    Console.WriteLine("Aucun élément commun");
}
```

### Quand utiliser HashSet<T>

- Pour stocker des collections d'éléments uniques
- Quand vous avez besoin de vérifier rapidement l'appartenance d'un élément
- Pour effectuer des opérations d'ensemble (union, intersection, différence)
- Quand l'ordre des éléments n'est pas important

## 3.2.4. Queue<T> et Stack<T>

### Queue<T> (File d'attente)

Une `Queue<T>` représente une collection de type "premier entré, premier sorti" (FIFO - First In, First Out).

```csharp
// Création d'une file vide
Queue<string> fileAttente = new Queue<string>();

// Ajouter des éléments à la fin de la file
fileAttente.Enqueue("Client 1");
fileAttente.Enqueue("Client 2");
fileAttente.Enqueue("Client 3");

Console.WriteLine($"Nombre de clients en attente: {fileAttente.Count}");

// Consulter le premier élément sans le retirer
string premierClient = fileAttente.Peek();
Console.WriteLine($"Prochain client: {premierClient}");  // Client 1

// Retirer et obtenir le premier élément
string clientServi = fileAttente.Dequeue();
Console.WriteLine($"Client servi: {clientServi}");  // Client 1
Console.WriteLine($"Clients restants: {fileAttente.Count}");  // 2

// Parcourir la file sans la modifier
Console.WriteLine("Clients en attente:");
foreach (string client in fileAttente)
{
    Console.WriteLine(client);  // Client 2, Client 3
}

// Vider la file
fileAttente.Clear();
```

#### Exemple pratique : Simulation d'une file d'attente

```csharp
Queue<string> fileImpression = new Queue<string>();

// Ajouter des documents à imprimer
fileImpression.Enqueue("Rapport mensuel.pdf");
fileImpression.Enqueue("Présentation.pptx");
fileImpression.Enqueue("Facture.docx");

Console.WriteLine($"{fileImpression.Count} documents en attente d'impression");

// Simuler le processus d'impression
Random random = new Random();

while (fileImpression.Count > 0)
{
    string document = fileImpression.Dequeue();
    Console.WriteLine($"Impression de {document}...");

    // Simuler le temps d'impression
    int tempsImpression = random.Next(1, 4);
    Console.WriteLine($"Impression terminée en {tempsImpression} secondes");

    Console.WriteLine($"Documents restants: {fileImpression.Count}");
    Console.WriteLine();
}

Console.WriteLine("Tous les documents ont été imprimés.");
```

### Stack<T> (Pile)

Une `Stack<T>` représente une collection de type "dernier entré, premier sorti" (LIFO - Last In, First Out).

```csharp
// Création d'une pile vide
Stack<int> pile = new Stack<int>();

// Ajouter des éléments au sommet de la pile
pile.Push(1);
pile.Push(2);
pile.Push(3);

Console.WriteLine($"Nombre d'éléments dans la pile: {pile.Count}");

// Consulter l'élément au sommet sans le retirer
int sommet = pile.Peek();
Console.WriteLine($"Élément au sommet: {sommet}");  // 3

// Retirer et obtenir l'élément au sommet
int élément = pile.Pop();
Console.WriteLine($"Élément retiré: {élément}");  // 3
Console.WriteLine($"Éléments restants: {pile.Count}");  // 2

// Parcourir la pile sans la modifier
Console.WriteLine("Éléments de la pile (du sommet vers la base):");
foreach (int valeur in pile)
{
    Console.WriteLine(valeur);  // 2, 1
}

// Vider la pile
pile.Clear();
```

#### Exemple pratique : Vérification des parenthèses équilibrées

```csharp
static bool SontParenthèsesÉquilibrées(string expression)
{
    Stack<char> pile = new Stack<char>();

    foreach (char c in expression)
    {
        switch (c)
        {
            case '(':
            case '[':
            case '{':
                pile.Push(c);
                break;

            case ')':
                if (pile.Count == 0 || pile.Pop() != '(')
                    return false;
                break;

            case ']':
                if (pile.Count == 0 || pile.Pop() != '[')
                    return false;
                break;

            case '}':
                if (pile.Count == 0 || pile.Pop() != '{')
                    return false;
                break;
        }
    }

    // Si la pile est vide, toutes les parenthèses sont équilibrées
    return pile.Count == 0;
}

// Test de la fonction
string[] expressions = {
    "(a + b) * (c - d)",
    "[(a + b) * {c - d}]",
    "((a + b) * (c - d)",
    "(a + b) * (c - d})"
};

foreach (string expr in expressions)
{
    bool équilibrée = SontParenthèsesÉquilibrées(expr);
    Console.WriteLine($"'{expr}' est {(équilibrée ? "équilibrée" : "déséquilibrée")}");
}
```

### Quand utiliser Queue<T> et Stack<T>

**Queue<T>**:
- Pour les situations "premier arrivé, premier servi"
- Files d'attente de traitement (impression, messages, etc.)
- Traitement par lots dans l'ordre d'arrivée
- Algorithmes de parcours en largeur (BFS)

**Stack<T>**:
- Pour les situations "dernier arrivé, premier servi"
- Pour annuler/rétablir des opérations
- Évaluation d'expressions
- Algorithmes de parcours en profondeur (DFS)
- Gestion des appels de fonction (pile d'appels)

## 3.2.5. LinkedList<T>

`LinkedList<T>` est une liste doublement chaînée, où chaque élément (nœud) contient une référence à l'élément précédent et suivant. Cette structure permet des insertions et suppressions efficaces à n'importe quelle position de la liste.

### Création et initialisation

```csharp
// Création d'une liste chaînée vide
LinkedList<string> listeChainée = new LinkedList<string>();

// Création à partir d'une collection existante
string[] fruits = { "Pomme", "Banane", "Orange" };
LinkedList<string> listeFruits = new LinkedList<string>(fruits);
```

### Ajouter des éléments

```csharp
LinkedList<string> liste = new LinkedList<string>();

// Ajouter au début
liste.AddFirst("Premier");

// Ajouter à la fin
liste.AddLast("Dernier");

// Ajouter après un nœud spécifique
LinkedListNode<string> nœudPremier = liste.First;
liste.AddAfter(nœudPremier, "Deuxième");

// Ajouter avant un nœud spécifique
LinkedListNode<string> nœudDernier = liste.Last;
liste.AddBefore(nœudDernier, "Avant-dernier");

// État de la liste: Premier -> Deuxième -> Avant-dernier -> Dernier
```

### Accéder aux nœuds et éléments (suite)

```csharp
LinkedList<string> liste = new LinkedList<string>();
liste.AddLast("Un");
liste.AddLast("Deux");
liste.AddLast("Trois");

// Obtenir le premier et le dernier nœud
LinkedListNode<string> premier = liste.First;
LinkedListNode<string> dernier = liste.Last;

Console.WriteLine($"Premier élément: {premier.Value}");  // Un
Console.WriteLine($"Dernier élément: {dernier.Value}");  // Trois

// Naviguer dans la liste avec les propriétés Next et Previous
LinkedListNode<string> deuxième = premier.Next;
Console.WriteLine($"Deuxième élément: {deuxième.Value}");  // Deux

LinkedListNode<string> avantDernier = dernier.Previous;
Console.WriteLine($"Avant-dernier élément: {avantDernier.Value}");  // Deux

// Rechercher un nœud contenant une valeur spécifique
LinkedListNode<string> nœudRecherché = liste.Find("Deux");
if (nœudRecherché != null)
{
    Console.WriteLine($"Trouvé: {nœudRecherché.Value}");

    // Accéder aux nœuds voisins
    if (nœudRecherché.Previous != null)
        Console.WriteLine($"Précédent: {nœudRecherché.Previous.Value}");

    if (nœudRecherché.Next != null)
        Console.WriteLine($"Suivant: {nœudRecherché.Next.Value}");
}
```

### Parcourir une liste chaînée

```csharp
LinkedList<string> liste = new LinkedList<string>();
liste.AddLast("Un");
liste.AddLast("Deux");
liste.AddLast("Trois");
liste.AddLast("Quatre");

// 1. Parcours du début à la fin avec foreach
Console.WriteLine("Parcours avec foreach:");
foreach (string élément in liste)
{
    Console.WriteLine(élément);
}

// 2. Parcours du début à la fin avec un nœud
Console.WriteLine("\nParcours avec nœud (début -> fin):");
for (LinkedListNode<string> nœud = liste.First; nœud != null; nœud = nœud.Next)
{
    Console.WriteLine(nœud.Value);
}

// 3. Parcours de la fin au début
Console.WriteLine("\nParcours avec nœud (fin -> début):");
for (LinkedListNode<string> nœud = liste.Last; nœud != null; nœud = nœud.Previous)
{
    Console.WriteLine(nœud.Value);
}
```

### Supprimer des éléments

```csharp
LinkedList<string> liste = new LinkedList<string>();
liste.AddLast("Un");
liste.AddLast("Deux");
liste.AddLast("Trois");
liste.AddLast("Quatre");

// Supprimer le premier élément
liste.RemoveFirst();

// Supprimer le dernier élément
liste.RemoveLast();

// Supprimer un élément spécifique par valeur (première occurrence)
bool supprimé = liste.Remove("Deux");
Console.WriteLine($"'Deux' supprimé: {supprimé}");

// Supprimer un nœud spécifique
LinkedListNode<string> nœud = liste.Find("Trois");
if (nœud != null)
{
    liste.Remove(nœud);
}

// Vider la liste
liste.Clear();
Console.WriteLine($"Nombre d'éléments après Clear: {liste.Count}");  // 0
```

### Exemple pratique : Éditeur de texte simplifié

```csharp
class LigneTexte
{
    public string Contenu { get; set; }

    public LigneTexte(string contenu)
    {
        Contenu = contenu;
    }

    public override string ToString()
    {
        return Contenu;
    }
}

class ÉditeurTexte
{
    private LinkedList<LigneTexte> lignes = new LinkedList<LigneTexte>();
    private LinkedListNode<LigneTexte> positionCurseur;

    public ÉditeurTexte()
    {
        // Commencer avec une ligne vide
        LigneTexte ligneDépart = new LigneTexte("");
        lignes.AddFirst(ligneDépart);
        positionCurseur = lignes.First;
    }

    public void AjouterLigne(string contenu)
    {
        LigneTexte nouvelleLigne = new LigneTexte(contenu);

        if (positionCurseur == null)
        {
            lignes.AddLast(nouvelleLigne);
            positionCurseur = lignes.Last;
        }
        else
        {
            // Insérer après la position actuelle du curseur
            lignes.AddAfter(positionCurseur, nouvelleLigne);
            positionCurseur = positionCurseur.Next;
        }
    }

    public void SupprimerLigne()
    {
        if (positionCurseur != null)
        {
            LinkedListNode<LigneTexte> suivant = positionCurseur.Next;
            lignes.Remove(positionCurseur);

            // Déplacer le curseur à la ligne suivante ou précédente
            positionCurseur = suivant ?? lignes.Last;

            // S'assurer qu'il y a toujours au moins une ligne
            if (lignes.Count == 0)
            {
                LigneTexte ligneDépart = new LigneTexte("");
                lignes.AddFirst(ligneDépart);
                positionCurseur = lignes.First;
            }
        }
    }

    public void DéplacerCurseurHaut()
    {
        if (positionCurseur != null && positionCurseur.Previous != null)
        {
            positionCurseur = positionCurseur.Previous;
        }
    }

    public void DéplacerCurseurBas()
    {
        if (positionCurseur != null && positionCurseur.Next != null)
        {
            positionCurseur = positionCurseur.Next;
        }
    }

    public void AfficherDocument()
    {
        int numéroLigne = 1;

        foreach (LigneTexte ligne in lignes)
        {
            string marqueur = (ligne == positionCurseur?.Value) ? "-> " : "   ";
            Console.WriteLine($"{marqueur}{numéroLigne}: {ligne.Contenu}");
            numéroLigne++;
        }
    }
}

// Utilisation de l'éditeur
ÉditeurTexte éditeur = new ÉditeurTexte();

éditeur.AjouterLigne("Première ligne");
éditeur.AjouterLigne("Deuxième ligne");
éditeur.AjouterLigne("Troisième ligne");

Console.WriteLine("Document initial:");
éditeur.AfficherDocument();

Console.WriteLine("\nAprès déplacement du curseur vers le haut:");
éditeur.DéplacerCurseurHaut();
éditeur.AfficherDocument();

Console.WriteLine("\nAprès suppression de la ligne courante:");
éditeur.SupprimerLigne();
éditeur.AfficherDocument();

Console.WriteLine("\nAprès ajout d'une nouvelle ligne:");
éditeur.AjouterLigne("Nouvelle ligne");
éditeur.AfficherDocument();
```

### Quand utiliser LinkedList<T>

- Lorsque vous avez besoin d'insérer ou supprimer fréquemment des éléments au milieu d'une collection
- Quand vous devez maintenir des références explicites aux nœuds de la liste
- Pour implémenter facilement des listes doublement chaînées, des files d'attente ou des piles personnalisées
- Lorsque vous n'avez pas besoin d'un accès rapide aux éléments par index

À noter que `LinkedList<T>` n'offre pas d'accès direct par index comme `List<T>`. Pour accéder à un élément à une position spécifique, vous devez parcourir la liste à partir du début ou de la fin.

## 3.2.6. SortedList<TKey, TValue> et SortedDictionary<TKey, TValue>

Ces deux collections maintiennent leurs éléments triés par clé. Elles sont utiles lorsque vous avez besoin d'accéder fréquemment aux éléments dans un ordre trié.

### SortedList<TKey, TValue>

`SortedList<TKey, TValue>` combine les fonctionnalités d'un tableau et d'un dictionnaire. Elle maintient deux tableaux internes: un pour les clés et un pour les valeurs.

```csharp
// Création d'une SortedList vide
SortedList<string, int> âges = new SortedList<string, int>();

// Ajouter des éléments (ils seront automatiquement triés par clé)
âges.Add("Zoé", 25);
âges.Add("Alice", 30);
âges.Add("Bob", 28);
âges.Add("Charlie", 35);

// Afficher les éléments (triés par clé)
foreach (KeyValuePair<string, int> personne in âges)
{
    Console.WriteLine($"{personne.Key}: {personne.Value} ans");
}
// Sortie:
// Alice: 30 ans
// Bob: 28 ans
// Charlie: 35 ans
// Zoé: 25 ans

// Accéder à une valeur par clé
int âgeAlice = âges["Alice"];
Console.WriteLine($"Âge d'Alice: {âgeAlice}");

// Vérifier si une clé existe
bool charlieExiste = âges.ContainsKey("Charlie");
Console.WriteLine($"Charlie existe: {charlieExiste}");

// Accès par index (puisque SortedList maintient un ordre)
string troisiemePersonne = âges.Keys[2];
int troisiemeÂge = âges.Values[2];
Console.WriteLine($"Troisième personne: {troisiemePersonne}, {troisiemeÂge} ans");  // Charlie, 35 ans

// Trouver l'index d'une clé
int indexAlice = âges.IndexOfKey("Alice");
Console.WriteLine($"Index d'Alice: {indexAlice}");  // 0 (car c'est la première alphabétiquement)
```

### SortedDictionary<TKey, TValue>

`SortedDictionary<TKey, TValue>` utilise une structure d'arbre rouge-noir pour maintenir les clés triées. Elle offre des performances différentes de `SortedList<TKey, TValue>`.

```csharp
// Création d'un SortedDictionary vide
SortedDictionary<string, double> notes = new SortedDictionary<string, double>();

// Ajouter des éléments
notes.Add("Mathématiques", 15.5);
notes.Add("Histoire", 14.0);
notes.Add("Physique", 16.5);
notes.Add("Anglais", 13.0);

// Afficher les éléments (triés par clé)
foreach (var matière in notes)
{
    Console.WriteLine($"{matière.Key}: {matière.Value}/20");
}
// Sortie:
// Anglais: 13/20
// Histoire: 14/20
// Mathématiques: 15.5/20
// Physique: 16.5/20

// Accéder et modifier une valeur
notes["Mathématiques"] = 16.0;

// Vérifier si une clé existe avant d'y accéder
if (notes.TryGetValue("Physique", out double notePhysique))
{
    Console.WriteLine($"Note en Physique: {notePhysique}/20");
}

// Supprimer un élément
notes.Remove("Histoire");
```

### Différences entre SortedList<TKey, TValue> et SortedDictionary<TKey, TValue>

```csharp
// Exemple comparatif
SortedList<int, string> listeTri = new SortedList<int, string>();
SortedDictionary<int, string> dictionnaireTri = new SortedDictionary<int, string>();

// Remplir les deux collections avec les mêmes données
Random random = new Random();
for (int i = 0; i < 1000; i++)
{
    int clé = random.Next(10000);
    string valeur = $"Valeur {clé}";

    // Éviter les doublons de clé
    if (!listeTri.ContainsKey(clé))
    {
        listeTri.Add(clé, valeur);
        dictionnaireTri.Add(clé, valeur);
    }
}

Console.WriteLine("Comparaison entre SortedList et SortedDictionary:");
Console.WriteLine($"Nombre d'éléments: {listeTri.Count}");

// Test de performance pour l'ajout
DateTime début = DateTime.Now;
for (int i = 0; i < 10000; i++)
{
    int clé = random.Next(100000);
    if (!listeTri.ContainsKey(clé))
    {
        listeTri.Add(clé, $"Nouvelle valeur {clé}");
    }
}
TimeSpan duréeListe = DateTime.Now - début;

début = DateTime.Now;
for (int i = 0; i < 10000; i++)
{
    int clé = random.Next(100000);
    if (!dictionnaireTri.ContainsKey(clé))
    {
        dictionnaireTri.Add(clé, $"Nouvelle valeur {clé}");
    }
}
TimeSpan duréeDictionnaire = DateTime.Now - début;

Console.WriteLine($"Temps pour 10000 ajouts:");
Console.WriteLine($"SortedList: {duréeListe.TotalMilliseconds} ms");
Console.WriteLine($"SortedDictionary: {duréeDictionnaire.TotalMilliseconds} ms");
```

Principales différences :

1. **Structure de données sous-jacente** :
   - `SortedList<TKey, TValue>` utilise deux tableaux parallèles (un pour les clés, un pour les valeurs)
   - `SortedDictionary<TKey, TValue>` utilise une structure d'arbre binaire de recherche équilibré (arbre rouge-noir)

2. **Performances** :
   - `SortedList` utilise moins de mémoire
   - `SortedDictionary` offre de meilleures performances pour les insertions et suppressions
   - `SortedList` est plus rapide pour l'accès séquentiel et lorsque la majorité des éléments sont ajoutés en ordre trié

3. **Fonctionnalités spécifiques** :
   - `SortedList` permet l'accès par index (avec `Keys[i]` et `Values[i]`)
   - `SortedDictionary` a généralement de meilleures performances pour les opérations de recherche dans de grandes collections

### Exemple pratique : Classement d'étudiants

```csharp
class Étudiant
{
    public string Nom { get; set; }
    public double Moyenne { get; set; }

    public Étudiant(string nom, double moyenne)
    {
        Nom = nom;
        Moyenne = moyenne;
    }

    public override string ToString()
    {
        return $"{Nom}: {Moyenne:F2}/20";
    }
}

// Fonction pour générer un classement d'étudiants
static void AfficherClassement(List<Étudiant> étudiants)
{
    // Utiliser un SortedDictionary pour trier les étudiants par moyenne (ordre décroissant)
    // Note: les clés doivent être uniques, donc on ajoute un identifiant pour gérer les ex-aequo
    SortedDictionary<double, List<Étudiant>> classement = new SortedDictionary<double, List<Étudiant>>(
        Comparer<double>.Create((a, b) => b.CompareTo(a))  // Tri décroissant
    );

    // Remplir le dictionnaire
    foreach (Étudiant étudiant in étudiants)
    {
        if (!classement.TryGetValue(étudiant.Moyenne, out List<Étudiant> groupe))
        {
            groupe = new List<Étudiant>();
            classement.Add(étudiant.Moyenne, groupe);
        }
        groupe.Add(étudiant);
    }

    // Afficher le classement
    int rang = 1;
    foreach (var groupe in classement)
    {
        foreach (var étudiant in groupe.Value)
        {
            Console.WriteLine($"{rang}. {étudiant}");
        }
        rang += groupe.Value.Count;
    }
}

// Utilisation
List<Étudiant> classe = new List<Étudiant>
{
    new Étudiant("Alice", 16.5),
    new Étudiant("Bob", 14.0),
    new Étudiant("Charlie", 16.5),
    new Étudiant("David", 12.5),
    new Étudiant("Emma", 18.0),
    new Étudiant("Frank", 14.0),
    new Étudiant("Grace", 15.5)
};

Console.WriteLine("Classement des étudiants par moyenne:");
AfficherClassement(classe);
```

### Quand utiliser SortedList<TKey, TValue> et SortedDictionary<TKey, TValue>

**SortedList<TKey, TValue>** :
- Lorsque vous avez besoin d'une collection triée qui utilise moins de mémoire
- Quand vous ajoutez la plupart des éléments en une seule fois, dans un ordre déjà trié
- Si vous avez besoin d'accéder aux éléments par index et par clé
- Pour des collections de petite à moyenne taille

**SortedDictionary<TKey, TValue>** :
- Pour les grandes collections où les performances sont critiques
- Quand vous effectuez de nombreuses insertions et suppressions aléatoires
- Si vous n'avez pas besoin d'accéder aux éléments par index
- Quand l'utilisation mémoire n'est pas la priorité principale

## Résumé des collections standard

Voici un tableau récapitulatif qui vous aidera à choisir la collection appropriée selon vos besoins :

| Collection | Description | Quand l'utiliser |
|------------|-------------|------------------|
| **List\<T>** | Liste ordonnée d'éléments accessibles par index | Pour une collection ordonnée avec accès fréquent par position |
| **Dictionary\<K,V>** | Collection de paires clé-valeur avec clés uniques | Pour un accès rapide aux valeurs par clé |
| **HashSet\<T>** | Collection d'éléments uniques sans ordre | Pour tester l'appartenance et réaliser des opérations d'ensemble |
| **Queue\<T>** | File d'attente FIFO (premier entré, premier sorti) | Pour traiter les éléments dans l'ordre d'arrivée |
| **Stack\<T>** | Pile LIFO (dernier entré, premier sorti) | Pour traiter les éléments dans l'ordre inverse d'arrivée |
| **LinkedList\<T>** | Liste doublement chaînée | Pour des insertions/suppressions fréquentes au milieu de la liste |
| **SortedList\<K,V>** | Dictionary trié par clé (utilise des tableaux) | Pour une collection triée avec peu d'insertions |
| **SortedDictionary\<K,V>** | Dictionary trié par clé (utilise un arbre) | Pour une collection triée avec insertions/suppressions fréquentes |

### Conseils pratiques pour l'utilisation des collections

1. **Choisissez la bonne collection pour votre cas d'utilisation**
   - L'utilisation de la collection adaptée peut considérablement améliorer les performances de votre application

2. **Utilisez les génériques pour le typage fort**
   - Préférez `List<string>` à `ArrayList` pour éviter les conversions de type et les erreurs potentielles

3. **Spécifiez la capacité initiale lorsque possible**
   - `List<T>` et `Dictionary<TKey, TValue>` redimensionnent automatiquement leur capacité interne, ce qui peut être coûteux
   - Si vous connaissez approximativement le nombre d'éléments à l'avance, spécifiez-le lors de la création

4. **Utilisez les méthodes de traitement par lot**
   - Préférez `AddRange` à plusieurs appels à `Add` individuels
   - Utilisez `RemoveAll` au lieu de multiples appels à `Remove`

5. **Implémentez vos propres comparateurs pour le tri personnalisé**
   - Utilisez `IComparer<T>` ou `Comparison<T>` pour personnaliser l'ordre de tri

6. **Considérez les collections thread-safe pour le multithreading**
   - Utilisez les collections de l'espace de noms `System.Collections.Concurrent` pour les environnements multi-threads

7. **Tirez parti de LINQ pour les opérations sur les collections**
   - LINQ (Language Integrated Query) offre des opérations puissantes sur les collections (filtrage, projection, tri, etc.)
   ```csharp
   // Exemple LINQ
   var personnesAdultes = personnes
       .Where(p => p.Age >= 18)
       .OrderBy(p => p.Nom)
       .Select(p => p.Nom);
   ```

8. **Faites attention aux performances**
   - Les collections ont des caractéristiques de performance différentes selon les opérations
   - Une liste chaînée est efficace pour les insertions mais lente pour l'accès aléatoire
   - Un dictionnaire est rapide pour les recherches mais ne maintient pas d'ordre

9. **Utilisez les structures de données immuables pour les scénarios fonctionnels**
   - Pour une programmation plus fonctionnelle, considérez les collections immuables de l'espace de noms `System.Collections.Immutable`

Avec ces connaissances sur les collections standard de C#, vous êtes maintenant équipé pour choisir la structure de données la plus adaptée à vos besoins et implémenter des solutions efficaces.

⏭️
