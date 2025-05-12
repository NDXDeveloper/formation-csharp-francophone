# 2.4 Opérateurs en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les opérateurs sont des symboles spéciaux qui permettent d'effectuer des opérations sur des variables et des valeurs. Ils sont essentiels dans tout programme car ils vous permettent de manipuler les données. C# offre une large gamme d'opérateurs pour différents types d'opérations.

## 2.4.1 Opérateurs arithmétiques et d'affectation

### Opérateurs arithmétiques

Les opérateurs arithmétiques vous permettent d'effectuer des calculs mathématiques de base sur des valeurs numériques.

| Opérateur | Nom | Exemple | Résultat |
|-----------|-----|---------|----------|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Soustraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `5 / 3` | `1` (pour les entiers) ou `1.6666...` (pour les flottants) |
| `%` | Modulo (reste de division) | `5 % 3` | `2` |
| `++` | Incrémentation | `i++` ou `++i` | Ajoute 1 à la variable |
| `--` | Décrémentation | `i--` ou `--i` | Soustrait 1 à la variable |

#### Division entière vs division flottante

Un point important à noter est que la division (`/`) se comporte différemment selon le type des opérandes :

```csharp
int a = 5;
int b = 2;
int resultatEntier = a / b;    // Résultat : 2 (la partie décimale est tronquée)

double c = 5;
double d = 2;
double resultatFlottant = c / d;    // Résultat : 2.5

// Si vous voulez une division flottante avec des entiers :
double resultatExact = (double)a / b;    // Résultat : 2.5
```

#### Incrémentation et décrémentation : pré vs post

Les opérateurs d'incrémentation (`++`) et de décrémentation (`--`) existent en deux versions :

1. **Pré-incrémentation/décrémentation** : L'opération est effectuée avant d'utiliser la valeur.
2. **Post-incrémentation/décrémentation** : L'opération est effectuée après avoir utilisé la valeur.

```csharp
int x = 5;
int y;

// Pré-incrémentation
y = ++x;    // x est d'abord incrémenté à 6, puis y reçoit la valeur 6

// Post-incrémentation
x = 5;      // Réinitialisez x à 5
y = x++;    // y reçoit d'abord la valeur 5, puis x est incrémenté à 6

Console.WriteLine($"Après pré-incrémentation : y = {y}, x = {x}");    // y = 6, x = 6
Console.WriteLine($"Après post-incrémentation : y = {y}, x = {6}");   // y = 5, x = 6
```

### Opérateurs d'affectation

Les opérateurs d'affectation permettent d'assigner des valeurs à des variables, souvent en combinant une opération arithmétique.

| Opérateur | Exemple | Équivalent à | Description |
|-----------|---------|--------------|-------------|
| `=` | `a = b` | | Affecte la valeur de `b` à `a` |
| `+=` | `a += b` | `a = a + b` | Addition puis affectation |
| `-=` | `a -= b` | `a = a - b` | Soustraction puis affectation |
| `*=` | `a *= b` | `a = a * b` | Multiplication puis affectation |
| `/=` | `a /= b` | `a = a / b` | Division puis affectation |
| `%=` | `a %= b` | `a = a % b` | Modulo puis affectation |

Ces opérateurs combinés sont très pratiques pour simplifier le code :

```csharp
int score = 10;

// Au lieu d'écrire :
score = score + 5;

// Vous pouvez écrire :
score += 5;    // score vaut maintenant 15

// Autres exemples
score -= 3;    // score = score - 3 (résultat : 12)
score *= 2;    // score = score * 2 (résultat : 24)
score /= 4;    // score = score / 4 (résultat : 6)
score %= 4;    // score = score % 4 (résultat : 2)
```

### Priorité des opérations

Comme en mathématiques, les opérations obéissent à un ordre de priorité :

1. Parenthèses `( )`
2. Incrémentation/décrémentation `++` `--`
3. Multiplication/division/modulo `*` `/` `%`
4. Addition/soustraction `+` `-`

```csharp
int resultat = 2 + 3 * 4;       // Résultat : 14 (car 3 * 4 est évalué d'abord)
int resultatAvecParentheses = (2 + 3) * 4;    // Résultat : 20
```

En cas de doute, utilisez des parenthèses pour rendre votre intention claire et éviter les erreurs.

## 2.4.2 Opérateurs logiques et bit à bit

### Opérateurs logiques

Les opérateurs logiques sont utilisés pour évaluer des expressions conditionnelles. Ils travaillent avec des valeurs booléennes (`true` ou `false`).

| Opérateur | Nom | Exemple | Description |
|-----------|-----|---------|-------------|
| `&&` | ET logique | `a && b` | `true` si `a` ET `b` sont `true` |
| `\|\|` | OU logique | `a \|\| b` | `true` si `a` OU `b` (ou les deux) est `true` |
| `!` | NON logique | `!a` | Inverse la valeur de `a` |

#### Exemples d'opérateurs logiques :

```csharp
bool estMajeur = true;
bool aDroitDeVote = true;
bool estCitoyen = false;

// ET logique (&&)
bool peutVoter = estMajeur && aDroitDeVote;    // true (les deux conditions sont vraies)
bool peutVoterAuxPresidentielles = peutVoter && estCitoyen;    // false (estCitoyen est false)

// OU logique (||)
bool peutEntrerAuCinema = estMajeur || estAccompagne;    // true (au moins une condition est vraie)

// NON logique (!)
bool estMineur = !estMajeur;    // false (inverse de estMajeur)
```

#### Évaluation court-circuit

Les opérateurs `&&` et `||` utilisent l'évaluation court-circuit :
- Avec `&&` : Si la première condition est `false`, la seconde n'est pas évaluée (car le résultat sera `false` quoi qu'il arrive).
- Avec `||` : Si la première condition est `true`, la seconde n'est pas évaluée (car le résultat sera `true` quoi qu'il arrive).

```csharp
// Exemple d'utilisation de l'évaluation court-circuit
bool VerifierUtilisateur()
{
    Console.WriteLine("Vérification de l'utilisateur...");
    return false;
}

bool utilisateurConnecte = false;
bool accesAutorise = utilisateurConnecte && VerifierUtilisateur();

// VerifierUtilisateur() ne sera pas appelé car utilisateurConnecte est déjà false
```

#### Opérateurs logiques non court-circuit

C# dispose également d'opérateurs logiques qui n'utilisent pas l'évaluation court-circuit : `&` et `|`. Ces opérateurs évaluent toujours les deux opérandes, même si le résultat peut être déterminé à partir du premier.

```csharp
bool resultat = utilisateurConnecte & VerifierUtilisateur();
// VerifierUtilisateur() sera appelé même si utilisateurConnecte est false
```

Ces opérateurs sont rarement utilisés pour la logique booléenne mais sont plus courants dans les opérations bit à bit.

### Opérateurs bit à bit

Les opérateurs bit à bit permettent de manipuler les bits individuels des valeurs entières. Ils sont utiles pour les masques de bits, les drapeaux et l'optimisation de certaines opérations.

| Opérateur | Nom | Exemple | Description |
|-----------|-----|---------|-------------|
| `&` | ET bit à bit | `a & b` | Compare chaque bit : 1 si les deux bits sont 1 |
| `\|` | OU bit à bit | `a \| b` | Compare chaque bit : 1 si au moins un bit est 1 |
| `^` | XOR bit à bit | `a ^ b` | Compare chaque bit : 1 si les bits sont différents |
| `~` | NON bit à bit | `~a` | Inverse tous les bits |
| `<<` | Décalage à gauche | `a << n` | Décale les bits de `a` de `n` positions vers la gauche |
| `>>` | Décalage à droite | `a >> n` | Décale les bits de `a` de `n` positions vers la droite |

#### Exemples d'opérations bit à bit :

```csharp
int a = 5;    // En binaire : 0000 0101
int b = 3;    // En binaire : 0000 0011

int resultatEt = a & b;     // Résultat : 1 (0000 0001)
int resultatOu = a | b;     // Résultat : 7 (0000 0111)
int resultatXor = a ^ b;    // Résultat : 6 (0000 0110)
int resultatNon = ~a;       // Résultat : -6 (1111 1010 en complément à deux)
int decalageGauche = a << 1;    // Résultat : 10 (0000 1010)
int decalageDroite = a >> 1;    // Résultat : 2 (0000 0010)
```

#### Utilisation pratique des opérateurs bit à bit :

```csharp
// Définition de drapeaux (permissions)
[Flags]
enum Permissions
{
    Aucune = 0,          // 0000
    Lecture = 1,         // 0001
    Ecriture = 2,        // 0010
    Execution = 4,       // 0100
    TousLesDroits = 7    // 0111
}

// Attribution de permissions
Permissions userPerms = Permissions.Lecture | Permissions.Ecriture;    // 0011

// Vérification de permission
bool peutLire = (userPerms & Permissions.Lecture) == Permissions.Lecture;    // true
bool peutExecuter = (userPerms & Permissions.Execution) == Permissions.Execution;    // false

// Ajout d'une permission
userPerms = userPerms | Permissions.Execution;    // ou userPerms |= Permissions.Execution;

// Retrait d'une permission
userPerms = userPerms & ~Permissions.Ecriture;    // ou userPerms &= ~Permissions.Ecriture;
```

> **🔍 Note pour les débutants :**
> Les opérations bit à bit sont un sujet avancé. Si vous débutez en programmation, ne vous inquiétez pas si ce concept semble complexe. Vous pourrez y revenir plus tard lorsque vous aurez besoin de les utiliser.

## 2.4.3 Comparaison et égalité

Les opérateurs de comparaison sont utilisés pour comparer des valeurs et retournent toujours un résultat booléen (`true` ou `false`).

### Opérateurs de comparaison

| Opérateur | Nom | Exemple | Description |
|-----------|-----|---------|-------------|
| `==` | Égalité | `a == b` | `true` si `a` est égal à `b` |
| `!=` | Inégalité | `a != b` | `true` si `a` n'est pas égal à `b` |
| `<` | Inférieur à | `a < b` | `true` si `a` est inférieur à `b` |
| `>` | Supérieur à | `a > b` | `true` si `a` est supérieur à `b` |
| `<=` | Inférieur ou égal à | `a <= b` | `true` si `a` est inférieur ou égal à `b` |
| `>=` | Supérieur ou égal à | `a >= b` | `true` si `a` est supérieur ou égal à `b` |

#### Exemples de comparaisons :

```csharp
int age = 25;
int limiteAge = 18;

bool estMajeur = age >= limiteAge;    // true
bool estMineur = age < limiteAge;     // false
bool estExactement25 = age == 25;     // true
bool estDifferentDe30 = age != 30;    // true
```

### Comparaison de valeurs vs référence

La comparaison se comporte différemment selon que vous comparez des types valeur ou des types référence :

#### Types valeur (int, double, struct, etc.)

Pour les types valeur, l'opérateur `==` compare les valeurs elles-mêmes :

```csharp
int a = 5;
int b = 5;
bool sontEgaux = (a == b);    // true, car les valeurs sont identiques
```

#### Types référence (objets, tableaux, etc.)

Pour les types référence, l'opérateur `==` compare par défaut les références (adresses mémoire), pas le contenu :

```csharp
class Personne
{
    public string Nom { get; set; }
}

Personne p1 = new Personne { Nom = "Alice" };
Personne p2 = new Personne { Nom = "Alice" };
Personne p3 = p1;

bool test1 = (p1 == p2);    // false, car ce sont deux objets différents en mémoire
bool test2 = (p1 == p3);    // true, car p3 référence le même objet que p1
```

#### Le cas spécial des chaînes de caractères

Les chaînes (`string`) sont des types référence, mais l'opérateur `==` a été surchargé pour comparer leur contenu, pas leurs références :

```csharp
string s1 = "Bonjour";
string s2 = "Bonjour";
bool sontEgales = (s1 == s2);    // true, car les contenus sont identiques
```

### Égalité de référence vs égalité de valeur

Pour comparer explicitement les références ou les valeurs, vous pouvez utiliser les méthodes suivantes :

```csharp
// Comparer les références
bool refEgales = ReferenceEquals(p1, p2);    // false

// Comparer les valeurs (nécessite que la classe implémente l'égalité)
bool valeursEgales = p1.Equals(p2);    // false par défaut, sauf si vous redéfinissez Equals
```

### Surcharge des opérateurs de comparaison

Vous pouvez redéfinir le comportement des opérateurs de comparaison pour vos propres classes :

```csharp
class Point
{
    public int X { get; set; }
    public int Y { get; set; }

    public static bool operator ==(Point a, Point b)
    {
        // Gérer les références null
        if (ReferenceEquals(a, null) && ReferenceEquals(b, null))
            return true;
        if (ReferenceEquals(a, null) || ReferenceEquals(b, null))
            return false;

        // Comparer les coordonnées
        return a.X == b.X && a.Y == b.Y;
    }

    public static bool operator !=(Point a, Point b)
    {
        return !(a == b);
    }

    // Il est recommandé de redéfinir aussi Equals et GetHashCode
    public override bool Equals(object obj)
    {
        if (obj is Point other)
            return this == other;
        return false;
    }

    public override int GetHashCode()
    {
        return X.GetHashCode() ^ Y.GetHashCode();
    }
}
```

> **💡 Conseil :**
> Si vous surchargez l'opérateur `==`, vous devez aussi surcharger l'opérateur `!=` et redéfinir les méthodes `Equals` et `GetHashCode` pour maintenir la cohérence.

## 2.4.4 Opérateur null-conditionnel (?.) et null-coalescent (??)

Ces opérateurs, introduits dans les versions récentes de C#, facilitent considérablement la gestion des valeurs null, rendant le code plus concis et plus sûr.

### Opérateur null-conditionnel (?.)

L'opérateur null-conditionnel (`?.`) vous permet d'accéder aux membres (propriétés, méthodes) d'un objet uniquement si celui-ci n'est pas null. Si l'objet est null, l'expression entière retourne null au lieu de provoquer une exception `NullReferenceException`.

```csharp
// Sans opérateur null-conditionnel
string nom = null;
int? longueur = null;

if (nom != null)
{
    longueur = nom.Length;    // Sécurisé car on a vérifié que nom n'est pas null
}

// Avec opérateur null-conditionnel
longueur = nom?.Length;    // Si nom est null, longueur sera null
                         // sinon, longueur aura la valeur de nom.Length
```

#### Chaînage avec l'opérateur null-conditionnel

Vous pouvez enchaîner plusieurs accès avec l'opérateur null-conditionnel, ce qui est particulièrement utile pour naviguer dans des objets imbriqués :

```csharp
class Commande
{
    public Client Client { get; set; }
}

class Client
{
    public Adresse AdresseLivraison { get; set; }
}

class Adresse
{
    public string Ville { get; set; }
}

// Sans opérateur null-conditionnel
string ville = null;
if (commande != null && commande.Client != null && commande.Client.AdresseLivraison != null)
{
    ville = commande.Client.AdresseLivraison.Ville;
}

// Avec opérateur null-conditionnel
ville = commande?.Client?.AdresseLivraison?.Ville;    // null si une référence dans la chaîne est null
```

#### Opérateur null-conditionnel avec méthodes et indexeurs

L'opérateur null-conditionnel fonctionne aussi avec les méthodes et les indexeurs :

```csharp
// Avec méthodes
string texte = null;
string majuscules = texte?.ToUpper();    // null si texte est null

// Avec indexeurs
List<string> liste = null;
string premier = liste?[0];    // null si liste est null

// Avec méthodes qui retournent void
Action action = null;
action?.Invoke();    // Ne fait rien si action est null
```

### Opérateur null-coalescent (??)

L'opérateur null-coalescent (`??`) vous permet de fournir une valeur par défaut à utiliser quand une expression est null.

```csharp
// Sans opérateur null-coalescent
string nom = GetNom();    // Peut retourner null
string affichage;

if (nom != null)
{
    affichage = nom;
}
else
{
    affichage = "Invité";
}

// Avec opérateur null-coalescent
affichage = nom ?? "Invité";    // Si nom est null, affichage sera "Invité"
```

#### Chaînage de l'opérateur null-coalescent

Vous pouvez chaîner plusieurs opérateurs null-coalescent pour tester plusieurs valeurs potentiellement null :

```csharp
string nom = prenom ?? nomFamille ?? "Anonyme";
// Utilise prenom si non-null, sinon nomFamille si non-null, sinon "Anonyme"
```

#### Opérateur d'affectation null-coalescent (??=)

C# 8.0 a introduit l'opérateur d'affectation null-coalescent (`??=`), qui assigne la valeur de droite à la variable de gauche uniquement si celle-ci est null :

```csharp
string message = null;
message ??= "Valeur par défaut";    // message devient "Valeur par défaut"

string autreMessage = "Bonjour";
autreMessage ??= "Autre valeur";    // autreMessage reste "Bonjour"
```

### Combinaison des opérateurs null-conditionnel et null-coalescent

Ces opérateurs sont particulièrement puissants lorsqu'ils sont utilisés ensemble :

```csharp
// Récupère la ville du client ou "Ville inconnue" si commande, client, adresse ou ville est null
string ville = commande?.Client?.AdresseLivraison?.Ville ?? "Ville inconnue";

// Récupère le premier élément de la liste ou 0 si la liste est null ou vide
int premier = liste?.FirstOrDefault() ?? 0;
```

Ces opérateurs permettent d'éliminer de nombreuses vérifications de null explicites, rendant le code plus concis et plus lisible. Ils sont particulièrement utiles dans les scénarios où vous travaillez avec des API qui peuvent retourner des valeurs null ou lorsque vous manipulez des données provenant de sources externes.

## 2.4.5 Expression conditionnelle ternaire

L'opérateur ternaire (`? :`) est un raccourci pour les instructions conditionnelles simples. Il vous permet d'évaluer une condition et de retourner une valeur parmi deux possibilités.

### Syntaxe

```csharp
condition ? valeur_si_vrai : valeur_si_faux
```

- Si `condition` est évaluée à `true`, l'expression retourne `valeur_si_vrai`.
- Si `condition` est évaluée à `false`, l'expression retourne `valeur_si_faux`.

### Exemples d'utilisation

#### Exemple simple

```csharp
// Sans opérateur ternaire
string message;
if (age >= 18)
{
    message = "Majeur";
}
else
{
    message = "Mineur";
}

// Avec opérateur ternaire
message = age >= 18 ? "Majeur" : "Mineur";
```

#### Assignation conditionnelle

```csharp
// Attribuer une couleur selon la température
int temperature = 25;
string couleur = temperature > 30 ? "Rouge" : temperature > 20 ? "Orange" : "Bleu";
```

#### Utilisation dans d'autres expressions

```csharp
// Calcul de prix avec ou sans réduction
decimal prix = estMembre ? prixBase * 0.9m : prixBase;

// Passage de paramètre conditionnel
AfficherMessage(estErreur ? "Erreur : " + message : message);

// Dans une interpolation de chaîne
Console.WriteLine($"Le candidat est {age >= 18 ? "éligible" : "non éligible"} pour voter.");
```

### Imbrication d'opérateurs ternaires

Vous pouvez imbriquer des opérateurs ternaires, mais cela peut rapidement rendre le code difficile à lire. Utilisez cette technique avec modération :

```csharp
// Notation imbriquée (difficile à lire)
string categorie = age < 12 ? "Enfant" : age < 18 ? "Adolescent" : age < 65 ? "Adulte" : "Senior";

// Version plus lisible avec des if-else
if (age < 12)
    categorie = "Enfant";
else if (age < 18)
    categorie = "Adolescent";
else if (age < 65)
    categorie = "Adulte";
else
    categorie = "Senior";
```

### Bonnes pratiques

1. **Gardez-le simple** : L'opérateur ternaire est idéal pour les conditions simples. Pour des logiques complexes, préférez les instructions `if-else`.

2. **Lisibilité avant tout** : Si l'expression devient trop longue ou complexe, envisagez d'utiliser une structure `if-else` classique.

3. **Évitez les effets de bord** : N'utilisez pas l'opérateur ternaire pour exécuter des actions, mais uniquement pour retourner des valeurs.

```csharp
// À éviter (action dans l'opérateur ternaire)
estValide ? compteur++ : LogErreur("Valeur invalide");

// Préférez
if (estValide)
    compteur++;
else
    LogErreur("Valeur invalide");
```

4. **Utilisez des parenthèses pour clarifier** lorsque vous combinez l'opérateur ternaire avec d'autres opérateurs :

```csharp
// Sans parenthèses (potentiellement ambigu)
string resultat = a > b ? a.ToString() : b.ToString() + " est plus grand";

// Avec parenthèses (plus clair)
string resultat = a > b ? a.ToString() : (b.ToString() + " est plus grand");
```

### Comparaison avec le pattern matching (C# 8.0+)

Dans les versions récentes de C#, vous pouvez utiliser l'expression switch comme alternative à l'opérateur ternaire pour des cas plus complexes :

```csharp
// Opérateur ternaire imbriqué
string taille = hauteur < 150 ? "Petit" : hauteur < 180 ? "Moyen" : "Grand";

// Équivalent avec une expression switch
string taille = hauteur switch
{
    < 150 => "Petit",
    < 180 => "Moyen",
    _ => "Grand"
};
```

L'expression switch est souvent plus lisible lorsque vous avez plus de deux cas à gérer.

## Résumé

- Les **opérateurs arithmétiques** (`+`, `-`, `*`, `/`, `%`) permettent d'effectuer des calculs mathématiques.
- Les **opérateurs d'affectation** (`=`, `+=`, `-=`, etc.) assignent des valeurs aux variables, parfois en combinant une opération.
- Les **opérateurs logiques** (`&&`, `||`, `!`) manipulent des valeurs booléennes et sont utilisés dans les conditions.
- Les **opérateurs bit à bit** (`&`, `|`, `^`, `~`, `<<`, `>>`) manipulent les bits individuels des valeurs entières.
- Les **opérateurs de comparaison** (`==`, `!=`, `<`, `>`, `<=`, `>=`) comparent des valeurs et retournent un booléen.
- L'**opérateur null-conditionnel** (`?.`) accède aux membres d'un objet uniquement s'il n'est pas null.
- L'**opérateur null-coalescent** (`??`) fournit une valeur par défaut lorsqu'une expression est null.
- L'**expression conditionnelle ternaire** (`? :`) évalue une condition et retourne une valeur parmi deux possibilités.

La maîtrise de ces opérateurs vous permettra d'écrire un code plus concis, plus expressif et plus robuste en C#.

⏭️ 2.5. [Structures de contrôle](/02-fondamentaux-du-langage-csharp/2-5-structures-de-controle.md)
