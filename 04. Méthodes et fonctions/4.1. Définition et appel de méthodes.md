# 4.1. Définition et appel de méthodes en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les méthodes sont au cœur de la programmation en C#. Elles vous permettent d'organiser votre code en blocs fonctionnels réutilisables. Dans cette section, nous allons explorer comment définir et appeler des méthodes, comprendre leurs signatures, les différents types de retour, et la différence entre méthodes statiques et méthodes d'instance.

## 4.1.1. Signature de méthode

La signature d'une méthode est sa "carte d'identité". Elle permet au compilateur de distinguer une méthode d'une autre et de déterminer quelle version d'une méthode doit être appelée lorsque plusieurs méthodes portent le même nom (surcharge).

### Composants d'une signature de méthode

Une signature de méthode complète comprend :

- **Modificateurs d'accès** : déterminent la visibilité de la méthode (`public`, `private`, `protected`, `internal`)
- **Type de retour** : le type de données que la méthode retourne
- **Nom de la méthode** : identifiant utilisé pour appeler la méthode
- **Paramètres** : les données que la méthode accepte en entrée
- **Modificateurs supplémentaires** : comme `static`, `virtual`, `override`, etc.

Voici la syntaxe générale :

```csharp
[modificateurs_accès] [modificateurs_supplémentaires] [type_retour] [nom_methode]([paramètres])
{
    // Corps de la méthode
    // Instructions à exécuter
}
```

### Exemples de signatures de méthodes

```csharp
// Méthode publique qui retourne un entier et prend deux paramètres entiers
public int Addition(int nombre1, int nombre2)
{
    return nombre1 + nombre2;
}

// Méthode privée qui ne retourne rien et prend une chaîne de caractères
private void AfficherMessage(string message)
{
    Console.WriteLine(message);
}

// Méthode statique publique qui retourne un booléen et prend un entier
public static bool EstPair(int nombre)
{
    return nombre % 2 == 0;
}
```

### Paramètres de méthodes

Les paramètres sont les données qu'une méthode accepte en entrée. Voici différentes façons de les définir :

#### Paramètres simples

```csharp
public void Saluer(string nom)
{
    Console.WriteLine($"Bonjour, {nom} !");
}
```

#### Paramètres avec valeur par défaut

```csharp
public void Saluer(string nom = "ami")
{
    Console.WriteLine($"Bonjour, {nom} !");
}

// Peut être appelé comme ceci :
// Saluer();         // Affiche : "Bonjour, ami !"
// Saluer("Marie");  // Affiche : "Bonjour, Marie !"
```

#### Paramètres nommés (lors de l'appel)

```csharp
public void AfficherInfos(string nom, int age, string ville)
{
    Console.WriteLine($"{nom}, {age} ans, habite à {ville}");
}

// Peut être appelé comme ceci :
// AfficherInfos(nom: "Pierre", age: 30, ville: "Paris");
// AfficherInfos(ville: "Lyon", nom: "Sophie", age: 25); // Ordre différent grâce aux noms
```

## 4.1.2. Valeurs de retour et void

Chaque méthode en C# peut retourner une valeur ou ne rien retourner. Le type de retour est spécifié dans la signature de la méthode.

### Méthodes avec valeur de retour

Lorsqu'une méthode retourne une valeur, vous devez spécifier le type de cette valeur dans sa signature et utiliser l'instruction `return` pour renvoyer la valeur :

```csharp
// Méthode qui retourne un entier
public int Multiplier(int a, int b)
{
    return a * b;
}

// Méthode qui retourne une chaîne de caractères
public string FormatNomComplet(string prenom, string nom)
{
    return $"{prenom} {nom}";
}

// Méthode qui retourne un booléen
public bool EstMajeur(int age)
{
    return age >= 18;
}
```

Pour appeler une méthode qui retourne une valeur, vous pouvez :
- Stocker la valeur retournée dans une variable
- Utiliser la valeur directement dans une expression
- Ignorer la valeur retournée (mais ce n'est généralement pas recommandé)

```csharp
// Stocker la valeur dans une variable
int resultat = Multiplier(4, 5);
Console.WriteLine(resultat);  // Affiche : 20

// Utiliser la valeur directement
Console.WriteLine(Multiplier(3, 7));  // Affiche : 21

// Utiliser la valeur dans une condition
if (EstMajeur(16))
{
    Console.WriteLine("La personne est majeure");
}
else
{
    Console.WriteLine("La personne est mineure");
}
```

### Méthodes sans valeur de retour (void)

Lorsqu'une méthode ne doit pas retourner de valeur, utilisez le mot-clé `void` comme type de retour :

```csharp
public void AfficherDate()
{
    Console.WriteLine(DateTime.Now.ToString("dd/MM/yyyy"));
}

public void EnvoyerEmail(string destinataire, string sujet, string corps)
{
    // Code pour envoyer un email
    Console.WriteLine($"Email envoyé à {destinataire}");
}
```

Les méthodes `void` sont appelées pour leur effet secondaire (affichage, modification d'état, etc.) et non pour obtenir une valeur :

```csharp
// Appel d'une méthode void
AfficherDate();
EnvoyerEmail("contact@exemple.com", "Bienvenue", "Bienvenue sur notre site !");
```

Dans une méthode `void`, vous pouvez utiliser l'instruction `return` sans valeur pour sortir prématurément de la méthode :

```csharp
public void VerifierAge(int age)
{
    if (age < 0)
    {
        Console.WriteLine("L'âge ne peut pas être négatif");
        return; // Sort de la méthode
    }

    Console.WriteLine($"Âge vérifié : {age} ans");
}
```

## 4.1.3. Méthodes statiques vs d'instance

En C#, il existe deux types principaux de méthodes : les méthodes d'instance et les méthodes statiques. Comprendre la différence est essentiel pour bien structurer votre code.

### Méthodes d'instance

Les méthodes d'instance sont liées à une instance spécifique d'une classe. Elles :
- Peuvent accéder aux champs et propriétés d'instance de la classe
- Sont appelées sur un objet spécifique
- Peuvent utiliser le mot-clé `this` pour référencer l'instance courante

```csharp
public class Calculatrice
{
    // Champ d'instance
    private int _memoire;

    // Méthode d'instance
    public int Additionner(int a, int b)
    {
        int resultat = a + b;
        // Peut accéder aux champs d'instance
        _memoire = resultat;
        return resultat;
    }

    // Autre méthode d'instance
    public int GetMemoire()
    {
        return _memoire;
    }
}
```

Pour utiliser ces méthodes, vous devez d'abord créer une instance de la classe :

```csharp
// Création d'une instance
Calculatrice calc = new Calculatrice();

// Appel de méthodes sur cette instance
int somme = calc.Additionner(5, 7);
Console.WriteLine(somme);  // Affiche : 12

int valeurEnMemoire = calc.GetMemoire();
Console.WriteLine(valeurEnMemoire);  // Affiche : 12
```

Si vous créez plusieurs instances, chacune aura son propre état indépendant :

```csharp
Calculatrice calc1 = new Calculatrice();
Calculatrice calc2 = new Calculatrice();

calc1.Additionner(10, 20);
calc2.Additionner(1, 2);

Console.WriteLine(calc1.GetMemoire());  // Affiche : 30
Console.WriteLine(calc2.GetMemoire());  // Affiche : 3
```

### Méthodes statiques

Les méthodes statiques appartiennent à la classe elle-même, et non à une instance spécifique. Elles :
- Sont marquées avec le mot-clé `static`
- Ne peuvent pas accéder aux membres d'instance (non statiques) directement
- Peuvent uniquement accéder à d'autres membres statiques de la classe
- Sont appelées directement sur la classe, sans créer d'instance

```csharp
public class MathUtils
{
    // Constante statique
    public static readonly double PI = 3.14159;

    // Méthode statique
    public static double CalculerAireCercle(double rayon)
    {
        // Peut accéder à d'autres membres statiques
        return PI * rayon * rayon;
    }

    // Autre méthode statique
    public static bool EstNombrePremier(int nombre)
    {
        if (nombre <= 1) return false;
        if (nombre == 2) return true;

        for (int i = 2; i <= Math.Sqrt(nombre); i++)
        {
            if (nombre % i == 0) return false;
        }

        return true;
    }
}
```

Pour appeler ces méthodes statiques, utilisez directement le nom de la classe :

```csharp
// Aucune instance n'est nécessaire
double aire = MathUtils.CalculerAireCercle(5);
Console.WriteLine(aire);  // Affiche : 78.53975

bool estPremier = MathUtils.EstNombrePremier(17);
Console.WriteLine(estPremier);  // Affiche : True
```

### Quand utiliser des méthodes statiques vs d'instance ?

#### Utilisez des méthodes statiques quand :
- La fonctionnalité ne dépend pas de l'état d'un objet spécifique
- La méthode effectue une opération utilitaire ou une fonction pure (même entrée = même sortie)
- Vous souhaitez regrouper des fonctionnalités liées sans créer d'instances
- La méthode opère sur des paramètres fournis et ne modifie pas l'état de la classe

#### Utilisez des méthodes d'instance quand :
- L'opération dépend de l'état de l'objet
- Vous travaillez avec les données d'une instance spécifique
- Vous avez besoin de maintenir un état entre les appels de méthode
- Vous suivez le paradigme de la programmation orientée objet

### Exemple complet combinant méthodes statiques et d'instance

```csharp
public class Personne
{
    // Champs d'instance
    public string Nom { get; private set; }
    public int Age { get; private set; }

    // Constructeur
    public Personne(string nom, int age)
    {
        Nom = nom;
        Age = age;
    }

    // Méthode d'instance
    public bool EstMajeur()
    {
        // Utilise l'état de l'instance (Age)
        return Age >= 18;
    }

    // Méthode d'instance
    public string GetDescription()
    {
        string statutMajorite = EstMajeur() ? "majeure" : "mineure";
        return $"{Nom}, {Age} ans, personne {statutMajorite}";
    }

    // Méthode statique - ne dépend pas d'une instance
    public static int CalculerAgeAPartirAnneeNaissance(int anneeNaissance)
    {
        int anneeActuelle = DateTime.Now.Year;
        return anneeActuelle - anneeNaissance;
    }

    // Méthode statique - factory method
    public static Personne CreerPersonneAdulte(string nom)
    {
        // Crée une nouvelle instance
        return new Personne(nom, 18);
    }
}
```

Utilisation :

```csharp
// Utilisation de méthodes d'instance
Personne alice = new Personne("Alice", 25);
Console.WriteLine(alice.GetDescription());  // Affiche : "Alice, 25 ans, personne majeure"

// Utilisation de méthodes statiques
int ageCalcule = Personne.CalculerAgeAPartirAnneeNaissance(1990);
Console.WriteLine($"Âge calculé : {ageCalcule} ans");

// Utilisation d'une factory method statique
Personne nouvelAdulte = Personne.CreerPersonneAdulte("Thomas");
Console.WriteLine(nouvelAdulte.GetDescription());  // Affiche : "Thomas, 18 ans, personne majeure"
```

### Astuces et bonnes pratiques

1. **Nommez clairement vos méthodes** : Le nom doit refléter ce que fait la méthode (commencez souvent par un verbe)
2. **Principe de responsabilité unique** : Une méthode ne doit faire qu'une seule chose, mais la faire bien
3. **Méthodes courtes** : Visez des méthodes courtes (moins de 20-30 lignes) pour améliorer la lisibilité
4. **Documentation** : Utilisez des commentaires XML `///` pour documenter vos méthodes
5. **Cohérence** : Gardez un style cohérent pour vos signatures de méthodes dans toute votre application

```csharp
/// <summary>
/// Calcule l'âge à partir de la date de naissance.
/// </summary>
/// <param name="dateNaissance">La date de naissance</param>
/// <returns>L'âge en années</returns>
public static int CalculerAge(DateTime dateNaissance)
{
    DateTime aujourdhui = DateTime.Today;
    int age = aujourdhui.Year - dateNaissance.Year;

    // Ajustement si l'anniversaire n'est pas encore passé cette année
    if (dateNaissance.Date > aujourdhui.AddYears(-age))
    {
        age--;
    }

    return age;
}
```

En maîtrisant la création et l'utilisation des méthodes en C#, vous construirez un code plus organisé, plus maintenable et plus réutilisable.

⏭️
