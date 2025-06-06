# 2.3 Variables et constantes en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les variables et les constantes sont des éléments fondamentaux de tout programme informatique. Elles permettent de stocker, manipuler et réutiliser des données tout au long de l'exécution de votre code. Comprendre comment les déclarer, les initialiser et gérer leur portée est essentiel pour tout développeur C#.

## 2.3.1 Déclaration et initialisation

### Qu'est-ce qu'une variable ?

Une variable est un espace nommé en mémoire qui contient une valeur d'un certain type. En C#, chaque variable a :
- Un nom (identificateur)
- Un type (qui détermine quelle sorte de données elle peut contenir)
- Une valeur (qui peut changer au cours du programme)

### Déclaration de variables

Pour déclarer une variable en C#, vous devez spécifier son type suivi de son nom :

```csharp
// Syntaxe de base : Type Nom;
int age;
string nom;
bool estActif;
double salaire;
```

### Règles de nommage des variables

En C#, les noms de variables doivent suivre certaines règles :

- Ils doivent commencer par une lettre ou un underscore (_)
- Ils peuvent contenir des lettres, des chiffres et des underscores
- Ils sont sensibles à la casse (age et Age sont deux variables différentes)
- Ils ne peuvent pas être un mot-clé réservé du langage

**Convention de nommage** : En C#, on utilise généralement la convention "camelCase" pour les variables locales (première lettre en minuscule, puis majuscule à chaque nouveau mot).

```csharp
// Bons noms de variables
int compteur;
string nomUtilisateur;
double tauxInteret;
bool estAuthentifie;

// Noms de variables à éviter
int a;                 // Trop court, pas assez descriptif
string NOM_UTILISATEUR; // Pas en camelCase
double _123;           // Commence par un chiffre (invalide)
bool is;               // Mot réservé (invalide)
```

### Initialisation de variables

L'initialisation consiste à assigner une première valeur à une variable. Vous pouvez le faire au moment de la déclaration ou plus tard :

```csharp
// Initialisation lors de la déclaration
int age = 25;
string nom = "Marie";

// Déclaration puis initialisation séparée
double taille;
taille = 1.75;
```

### Initialisation par défaut

Si vous ne fournissez pas de valeur initiale, C# assigne des valeurs par défaut selon le type :

| Type | Valeur par défaut |
|------|------------------|
| Types numériques (`int`, `float`, etc.) | 0 |
| `bool` | false |
| `char` | '\0' (caractère nul) |
| Types référence (`string`, objets, etc.) | null |

> **🔍 Note:** Ces valeurs par défaut s'appliquent aux variables membres de classes, mais pas aux variables locales. Les variables locales doivent être explicitement initialisées avant utilisation.

### Déclaration de plusieurs variables

Vous pouvez déclarer plusieurs variables du même type en une seule ligne :

```csharp
// Plusieurs variables du même type
int x, y, z;

// Avec initialisation
int a = 1, b = 2, c = 3;

// Certaines avec initialisation, d'autres sans
string prenom = "Jean", nom = "Dupont", adresse;
```

### Variables mutables vs variables en lecture seule

Par défaut, les variables en C# sont mutables, ce qui signifie que leur valeur peut changer après l'initialisation. Nous verrons plus tard comment créer des variables en lecture seule avec les mots-clés `readonly` et `const`.

## 2.3.2 Portée (scope) des variables

La portée (ou scope) d'une variable définit la partie du code où cette variable est accessible. En C#, la portée est principalement déterminée par les accolades `{ }` qui entourent le code.

### Types de portée

#### 1. Variables locales

Les variables déclarées à l'intérieur d'une méthode, d'un constructeur ou d'un bloc sont des variables locales. Elles sont accessibles uniquement à l'intérieur du bloc où elles sont déclarées.

```csharp
void MaMethode()
{
    int x = 10;  // Variable locale à la méthode
    Console.WriteLine(x);  // OK

    if (x > 5)
    {
        int y = 20;  // Variable locale au bloc if
        Console.WriteLine(y);  // OK
        Console.WriteLine(x);  // OK - x est accessible dans ce bloc
    }

    // Console.WriteLine(y);  // ERREUR - y n'est pas accessible ici
}

// Console.WriteLine(x);  // ERREUR - x n'est pas accessible ici
```

#### 2. Variables membres (champs)

Les variables déclarées au niveau de la classe (en dehors des méthodes) sont des variables membres ou champs. Elles sont accessibles par toutes les méthodes de la classe.

```csharp
class MaClasse
{
    int compteur = 0;  // Variable membre (champ)
    string nom;        // Autre variable membre

    void Methode1()
    {
        compteur++;          // OK - compteur est accessible ici
        nom = "Exemple";     // OK - nom est accessible ici
    }

    void Methode2()
    {
        Console.WriteLine(compteur);  // OK - compteur est accessible ici
        Console.WriteLine(nom);       // OK - nom est accessible ici
    }
}
```

#### 3. Variables de paramètre

Les variables déclarées comme paramètres d'une méthode ont une portée limitée au corps de cette méthode.

```csharp
void AfficherMessage(string message, int repetitions)
{
    // message et repetitions sont des variables de paramètre
    for (int i = 0; i < repetitions; i++)
    {
        Console.WriteLine(message);
    }
}
```

### Masquage de variables (variable shadowing)

Lorsqu'une variable locale a le même nom qu'une variable membre, la variable locale "masque" la variable membre dans sa portée. C'est généralement une source de confusion à éviter.

```csharp
class MaClasse
{
    int compteur = 0;  // Variable membre

    void Methode()
    {
        int compteur = 5;  // Variable locale qui masque la variable membre
        Console.WriteLine(compteur);  // Affiche 5 (variable locale)

        // Pour accéder à la variable membre, utilisez "this"
        Console.WriteLine(this.compteur);  // Affiche 0 (variable membre)
    }
}
```

### Portée et durée de vie

Ne confondez pas la portée (où la variable est accessible) avec la durée de vie (combien de temps elle existe en mémoire) :

- Les variables locales existent uniquement pendant l'exécution du bloc où elles sont déclarées.
- Les variables membres existent tant que l'instance de la classe existe.
- Les variables statiques existent pendant toute la durée d'exécution du programme.

```csharp
void ExemplePortee()
{
    // Début de la portée de x
    int x = 10;

    if (x > 5)
    {
        // Début de la portée de y
        int y = 20;
        Console.WriteLine(y);
        // Fin de la portée de y
    }

    // y n'est plus accessible ici, mais x l'est
    Console.WriteLine(x);
    // Fin de la portée de x
}
```

## 2.3.3 Conversion de types (cast implicite/explicite)

Souvent, vous aurez besoin de convertir une valeur d'un type à un autre. C# offre deux types de conversions : implicites et explicites.

### Conversions implicites

Les conversions implicites sont automatiques et ne nécessitent pas de syntaxe spéciale. Elles sont permises lorsqu'il n'y a pas de risque de perte de données.

```csharp
// Conversions implicites
int nombre = 10;
long grandNombre = nombre;       // int → long (OK, pas de perte possible)

char caractere = 'A';
int valeurASCII = caractere;     // char → int (OK, le code ASCII est un entier)

float petit = 3.14f;
double grand = petit;            // float → double (OK, double a plus de précision)
```

#### Cas courants de conversions implicites :

- `byte` → `short` → `int` → `long` → `decimal`
- `int` → `double`
- `float` → `double`
- N'importe quel type numérique → `string` (via la concaténation)
- Tout type → `object` (boxing)

### Conversions explicites (cast)

Les conversions explicites (ou casts) sont nécessaires lorsqu'il y a un risque de perte de données. Vous devez indiquer explicitement au compilateur que vous êtes conscient de ce risque.

```csharp
// Conversions explicites (casts)
double nombre = 3.14;
int partieEntiere = (int)nombre;     // double → int (perte de la partie décimale)
                                    // Résultat : 3

int grandNombre = 200;
byte petitNombre = (byte)grandNombre; // int → byte (risque de dépassement)
                                    // OK si la valeur tient dans un byte

long valeur = 1234567890L;
int valeurReduite = (int)valeur;     // long → int (risque de dépassement)
```

#### Syntaxe du cast :

```csharp
destinationType variable = (destinationType)sourceVariable;
```

### Méthodes de conversion

C# fournit également plusieurs méthodes pour effectuer des conversions de façon sécurisée :

#### 1. Convert.ToXXX()

La classe `Convert` offre des méthodes pour convertir entre différents types :

```csharp
string nombreTexte = "123";
int nombre = Convert.ToInt32(nombreTexte);    // string → int

bool valeur = Convert.ToBoolean(1);           // int → bool (non-zéro = true)
double decimale = Convert.ToDouble("3,14");   // string → double
```

#### 2. Parse et TryParse

Les méthodes `Parse` et `TryParse` sont utiles pour convertir des chaînes en d'autres types :

```csharp
// Parse - lève une exception si échec
string nombreTexte = "123";
int nombre = int.Parse(nombreTexte);      // OK

// TryParse - plus sûr, ne lève pas d'exception
string texte = "abc";
bool succes = int.TryParse(texte, out int resultat);
// succes = false, resultat = 0
```

> **💡 Conseil :** Utilisez toujours `TryParse` plutôt que `Parse` pour les entrées utilisateur afin d'éviter les exceptions.

#### 3. ToString()

Pour convertir n'importe quel type en chaîne, utilisez la méthode `ToString()` :

```csharp
int nombre = 42;
string texte = nombre.ToString();      // "42"

double pi = 3.14159;
string piTexte = pi.ToString("F2");    // "3.14" (format à 2 décimales)
```

### Risques et pièges

#### Dépassement (overflow)

Lors de la conversion d'un type plus large vers un type plus étroit, il y a un risque de dépassement :

```csharp
int grandNombre = 1000;
byte petitNombre = (byte)grandNombre;   // Dépassement : 1000 ne tient pas dans un byte (max 255)
                                       // Résultat : 232 (1000 % 256)
```

#### Perte de précision

Lors de la conversion de types à virgule flottante vers des entiers, la partie décimale est tronquée :

```csharp
double d = 3.99;
int i = (int)d;    // Résultat : 3 (pas d'arrondi !)
```

#### Erreurs de format

Lors de la conversion de chaînes, assurez-vous que le format est correct :

```csharp
// Génère une FormatException
string texte = "abc";
int nombre = int.Parse(texte);    // Erreur !
```

## 2.3.4 var et inférence de type

L'inférence de type permet au compilateur de déterminer automatiquement le type d'une variable à partir de son initialisation. En C#, cela se fait à l'aide du mot-clé `var`.

### Utilisation de var

```csharp
// Au lieu de déclarer explicitement le type :
string message = "Bonjour";

// Vous pouvez utiliser var :
var message = "Bonjour";    // Le compilateur infère que message est de type string
```

Le compilateur détermine que `message` est de type `string` car il est initialisé avec une chaîne de caractères.

### Important à savoir sur var

- `var` **n'est pas** un type dynamique. Le type est déterminé à la compilation, pas à l'exécution.
- La variable doit être initialisée immédiatement (sur la même ligne).
- Une fois le type inféré, il ne peut pas être modifié.
- `var` ne peut être utilisé que pour les variables locales, pas pour les champs de classe, les paramètres ou les retours de méthode.

```csharp
// Correct
var age = 25;              // age est de type int
var nom = "Alice";         // nom est de type string
var liste = new List<int>(); // liste est de type List<int>

// Incorrect
var x;                     // Erreur : pas d'initialisation
var y = null;              // Erreur : impossible d'inférer le type
```

### Quand utiliser var ?

`var` est particulièrement utile dans les cas suivants :

1. **Types longs ou complexes** :
```csharp
// Sans var
Dictionary<string, List<Customer>> clientsParVille = new Dictionary<string, List<Customer>>();

// Avec var (plus lisible)
var clientsParVille = new Dictionary<string, List<Customer>>();
```

2. **Résultats de requêtes LINQ** :
```csharp
var resultats = from c in clients
                where c.Age > 30
                select new { c.Nom, c.Age };
```

3. **Quand le type est évident** :
```csharp
var message = "Bonjour";   // Clairement une chaîne
var nombre = 42;           // Clairement un entier
var liste = new List<int>(); // Clairement une liste d'entiers
```

### Contre-indications

Évitez `var` quand :
- Le type n'est pas évident à partir de l'initialisation
- La lisibilité du code en souffre
- Vous travaillez dans du code qui doit être particulièrement explicite

```csharp
// À éviter
var result = GetData();    // Quel est le type de result ?

// Préférez
SomeSpecificType result = GetData();
```

### var vs dynamic

Ne confondez pas `var` avec `dynamic` :
- `var` : Type déterminé à la compilation. Fort typage.
- `dynamic` : Type déterminé à l'exécution. Typage dynamique.

```csharp
var x = 10;      // x est un int, déterminé à la compilation
x = "texte";     // Erreur de compilation

dynamic y = 10;  // y peut changer de type à l'exécution
y = "texte";     // OK avec dynamic (mais peut causer des erreurs à l'exécution)
```

## 2.3.5 readonly et const

C# offre deux façons principales de créer des valeurs qui ne peuvent pas être modifiées : `readonly` et `const`. Bien qu'ils servent un objectif similaire, ils ont des différences importantes.

### Constantes (const)

Une constante est une valeur déterminée à la compilation et qui ne peut jamais changer.

#### Caractéristiques des constantes :

- Doivent être initialisées au moment de la déclaration
- Leur valeur est déterminée à la compilation
- Ne peuvent contenir que des types primitifs (int, string, etc.) et des expressions constantes
- Sont implicitement statiques (appartiennent à la classe, pas à une instance)

```csharp
class MaClasse
{
    // Exemples de constantes
    public const int AGE_MINIMUM = 18;
    public const string MESSAGE_ACCUEIL = "Bienvenue !";
    public const double PI = 3.14159;

    // Expressions constantes
    public const int DOUBLE_AGE = AGE_MINIMUM * 2;

    // Erreur - DateTime n'est pas un type primitif
    // public const DateTime DATE_CREATION = new DateTime(2023, 1, 1);
}
```

#### Utilisation des constantes :

```csharp
// Accès depuis l'extérieur de la classe
int age = MaClasse.AGE_MINIMUM;    // Notez l'accès via le nom de la classe
```

### Variables en lecture seule (readonly)

Une variable `readonly` est une variable dont la valeur ne peut être modifiée qu'au moment de sa déclaration ou dans un constructeur.

#### Caractéristiques des variables readonly :

- Peuvent être initialisées au moment de la déclaration ou dans un constructeur
- Leur valeur est déterminée à l'exécution (runtime)
- Peuvent être de n'importe quel type
- Par défaut, appartiennent à une instance de la classe (sauf si déclarées static)

```csharp
class MaClasse
{
    // Exemples de variables readonly
    public readonly int Id;
    public readonly DateTime DateCreation;
    public static readonly List<string> Categories = new List<string> { "A", "B", "C" };

    // Initialisation dans le constructeur
    public MaClasse(int id)
    {
        Id = id;  // OK - dans le constructeur
        DateCreation = DateTime.Now;  // OK - dans le constructeur
    }

    public void ModifierDonnees()
    {
        // Id = 100;  // Erreur - ne peut pas être modifié en dehors du constructeur

        // En revanche, on peut modifier le contenu d'un objet readonly
        Categories.Add("D");  // OK - on modifie le contenu, pas la référence
    }
}
```

### Différences clés entre const et readonly

| Caractéristique | const | readonly |
|----------------|-------|----------|
| **Moment de l'initialisation** | Déclaration uniquement | Déclaration ou constructeur |
| **Moment de l'évaluation** | Compilation | Exécution |
| **Types autorisés** | Types primitifs et expressions constantes | Tous types |
| **Statique/Instance** | Toujours statique | Peut être statique ou d'instance |
| **Modification du contenu** | N/A | Possible pour les types référence |

### Quand utiliser const vs readonly ?

- Utilisez **const** pour des valeurs vraiment constantes qui ne changeront jamais (π, constantes mathématiques, valeurs de configuration fixes).
- Utilisez **readonly** pour des valeurs qui :
  - Doivent être déterminées à l'exécution
  - Sont calculées dans le constructeur
  - Sont des types complexes (objets, collections)
  - Doivent être différentes pour chaque instance

```csharp
class Exemple
{
    // Bon usage de const
    public const double PI = 3.14159;
    public const int JOURS_SEMAINE = 7;

    // Bon usage de readonly
    public readonly Guid Id = Guid.NewGuid();    // Valeur unique par instance
    public readonly DateTime DateCreation = DateTime.Now;
    public static readonly HttpClient Client = new HttpClient();    // Ressource partagée
}
```

### readonly vs read-only properties

Ne confondez pas les champs `readonly` avec les propriétés en lecture seule :

```csharp
class Exemple
{
    // Champ readonly
    private readonly int _id;

    // Propriété en lecture seule
    public int Id { get; }    // Propriété avec getter uniquement

    // Autre façon de définir une propriété en lecture seule
    public string Nom
    {
        get { return "Exemple"; }
    }

    public Exemple(int id)
    {
        _id = id;
        Id = id;    // OK - initialisation dans le constructeur
    }
}
```

La principale différence est que les propriétés en lecture seule peuvent être initialisées dans des méthodes, alors que les champs `readonly` ne peuvent être initialisés que dans leur déclaration ou dans un constructeur.

## Résumé

- Les **variables** sont des espaces nommés en mémoire qui stockent des valeurs. Elles doivent être déclarées avec un type et peuvent être initialisées.
- La **portée d'une variable** définit où elle est accessible dans le code, généralement déterminée par les accolades `{}`.
- Les **conversions de types** permettent de transformer une valeur d'un type à un autre :
  - Les conversions implicites sont automatiques et sûres
  - Les conversions explicites (casts) nécessitent une syntaxe spéciale et peuvent entraîner des pertes de données
- Le mot-clé **var** permet l'inférence de type, où le compilateur détermine le type à partir de l'initialisation.
- Pour les valeurs immuables :
  - **const** est utilisé pour des valeurs constantes connues à la compilation
  - **readonly** est utilisé pour des valeurs qui ne peuvent être initialisées que dans le constructeur

La compréhension de ces concepts vous permettra de mieux gérer les données dans vos programmes C# et d'éviter des erreurs courantes liées à la manipulation des variables.

⏭️ 2.4. [Opérateurs](/02-fondamentaux-du-langage-csharp/2-4-operateurs.md)
