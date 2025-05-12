# 5.4. Héritage

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'héritage est l'un des quatre piliers fondamentaux de la programmation orientée objet, permettant à une classe d'hériter des caractéristiques (propriétés et méthodes) d'une autre classe. Cette section explore les concepts essentiels de l'héritage en C#, les mécanismes de substitution, la classe Object et compare l'héritage à la composition.

## 5.4.1. Classes de base et dérivées

En C#, l'héritage vous permet de créer une nouvelle classe (appelée classe dérivée) basée sur une classe existante (appelée classe de base). La classe dérivée hérite des caractéristiques de la classe de base et peut les étendre ou les modifier.

### Syntaxe de base

Pour créer une classe dérivée, utilisez le symbole `:` suivi du nom de la classe de base :

```csharp
// Classe de base
public class Animal
{
    public string Nom { get; set; }
    public int Age { get; set; }

    public Animal(string nom, int age)
    {
        Nom = nom;
        Age = age;
    }

    public void Manger()
    {
        Console.WriteLine($"{Nom} est en train de manger.");
    }

    public void Dormir()
    {
        Console.WriteLine($"{Nom} est en train de dormir.");
    }
}

// Classe dérivée
public class Chien : Animal
{
    public string Race { get; set; }

    public Chien(string nom, int age, string race)
        : base(nom, age)  // Appel au constructeur de la classe de base
    {
        Race = race;
    }

    public void Aboyer()
    {
        Console.WriteLine($"{Nom} ({Race}) aboie : Wouaf !");
    }
}

// Utilisation
Animal monAnimal = new Animal("Creature", 5);
monAnimal.Manger();  // "Creature est en train de manger."

Chien monChien = new Chien("Rex", 3, "Berger Allemand");
monChien.Manger();   // Hérité de Animal : "Rex est en train de manger."
monChien.Dormir();   // Hérité de Animal : "Rex est en train de dormir."
monChien.Aboyer();   // Spécifique à Chien : "Rex (Berger Allemand) aboie : Wouaf !"
```


### Classes et méthodes scellées (sealed)

Le mot-clé `sealed` peut être utilisé pour empêcher qu'une classe soit héritée ou qu'une méthode soit remplacée :

```csharp
// Classe qui ne peut pas être héritée
public sealed class Paramètres
{
    public string Thème { get; set; }
    public int Timeout { get; set; }
}

// Tentative d'hériter d'une classe scellée - générera une erreur de compilation
// public class ParamètresÉtendus : Paramètres { }

// Classe avec méthode qui ne peut pas être remplacée dans les classes dérivées
public class Base
{
    public virtual void MéthodeVirtuelle()
    {
        Console.WriteLine("Méthode virtuelle dans la classe de base");
    }

    public sealed virtual void MéthodeScellée()
    {
        Console.WriteLine("Méthode scellée dans la classe de base");
    }
}

public class Dérivée : Base
{
    // OK - remplace la méthode virtuelle
    public override void MéthodeVirtuelle()
    {
        Console.WriteLine("Méthode virtuelle remplacée dans la classe dérivée");
    }

    // Erreur - ne peut pas remplacer une méthode scellée
    // public override void MéthodeScellée() { }
}
```


### Héritage simple et limitations

C# supporte uniquement l'héritage simple, ce qui signifie qu'une classe ne peut hériter que d'une seule classe de base à la fois. Cela évite le problème du "diamant" qui peut survenir avec l'héritage multiple.

```csharp
public class A
{
    public void MéthodeA() { Console.WriteLine("Méthode A"); }
}

public class B : A
{
    public void MéthodeB() { Console.WriteLine("Méthode B"); }
}

// Héritage en cascade (autorisé)
public class C : B
{
    public void MéthodeC() { Console.WriteLine("Méthode C"); }
}

// Utilisation
var objetC = new C();
objetC.MéthodeA();  // Hérité de A via B
objetC.MéthodeB();  // Hérité de B
objetC.MéthodeC();  // Défini dans C

// Héritage multiple (non autorisé en C#)
// public class D : A, B { }  // Erreur de compilation
```


### Accès aux membres de la classe de base

Une classe dérivée peut accéder aux membres non privés de sa classe de base à l'aide du mot-clé `base` :

```csharp
public class FormeGéométrique
{
    public string Couleur { get; set; }

    public FormeGéométrique(string couleur)
    {
        Couleur = couleur;
    }

    public virtual double CalculerAire()
    {
        return 0;
    }

    public virtual string Décrire()
    {
        return $"Une forme géométrique de couleur {Couleur}";
    }
}

public class Rectangle : FormeGéométrique
{
    public double Longueur { get; set; }
    public double Largeur { get; set; }

    public Rectangle(double longueur, double largeur, string couleur)
        : base(couleur)  // Appel du constructeur de la classe de base
    {
        Longueur = longueur;
        Largeur = largeur;
    }

    public override double CalculerAire()
    {
        return Longueur * Largeur;
    }

    public override string Décrire()
    {
        // Utilisation de base pour appeler la méthode de la classe de base
        string descriptionBase = base.Décrire();
        return $"{descriptionBase} - Rectangle de dimensions {Longueur}x{Largeur}";
    }
}

// Utilisation
var rectangle = new Rectangle(5, 3, "Bleu");
Console.WriteLine(rectangle.Décrire());
// "Une forme géométrique de couleur Bleu - Rectangle de dimensions 5x3"
Console.WriteLine($"Aire: {rectangle.CalculerAire()}");  // "Aire: 15"
```


### Niveaux d'accessibilité des membres et héritage

Le tableau suivant montre comment les niveaux d'accessibilité affectent l'héritage :

| Modificateur | Classe de base | Classe dérivée (même assembly) | Classe dérivée (assembly différent) |
|--------------|----------------|--------------------------------|-------------------------------------|
| `public` | Accessible | Accessible | Accessible |
| `protected` | Accessible par la classe elle-même | Accessible | Accessible |
| `internal` | Accessible | Accessible | Non accessible |
| `protected internal` | Accessible | Accessible | Accessible uniquement par la classe dérivée |
| `private` | Accessible par la classe elle-même | Non accessible | Non accessible |
| `private protected` (C# 7.2+) | Accessible par la classe elle-même | Accessible uniquement dans le même assembly | Non accessible |

Exemple d'utilisation des niveaux d'accessibilité avec l'héritage :

```csharp
public class ClasseBase
{
    public string ChampPublic = "Accessible partout";
    protected string ChampProtected = "Accessible dans la classe et les classes dérivées";
    internal string ChampInternal = "Accessible dans le même assembly";
    protected internal string ChampProtectedInternal = "Accessible dans le même assembly ou par les classes dérivées";
    private string ChampPrivate = "Accessible uniquement dans cette classe";
    private protected string ChampPrivateProtected = "Accessible par les classes dérivées dans le même assembly";

    public void MéthodePublique()
    {
        Console.WriteLine("Méthode accessible partout");
        // Peut accéder à tous les champs
        Console.WriteLine(ChampPrivate);
    }
}

public class ClasseDérivée : ClasseBase
{
    public void AfficherChamps()
    {
        Console.WriteLine(ChampPublic);            // OK - public
        Console.WriteLine(ChampProtected);         // OK - protected
        Console.WriteLine(ChampInternal);          // OK - internal (même assembly)
        Console.WriteLine(ChampProtectedInternal); // OK - protected internal
        // Console.WriteLine(ChampPrivate);        // Erreur - private
        Console.WriteLine(ChampPrivateProtected);  // OK - private protected (même assembly)
    }
}

// Dans un autre assembly
// public class ClasseDérivéeExterne : ClasseBase
// {
//     public void AfficherChamps()
//     {
//         Console.WriteLine(ChampPublic);            // OK - public
//         Console.WriteLine(ChampProtected);         // OK - protected
//         // Console.WriteLine(ChampInternal);       // Erreur - internal
//         Console.WriteLine(ChampProtectedInternal); // OK - protected internal
//         // Console.WriteLine(ChampPrivate);        // Erreur - private
//         // Console.WriteLine(ChampPrivateProtected); // Erreur - private protected
//     }
// }
```


### Classes abstraites

Une classe abstraite est une classe qui ne peut pas être instanciée directement et qui est conçue pour être héritée. Elle peut contenir des méthodes abstraites (sans implémentation) que les classes dérivées doivent implémenter.

```csharp
// Classe abstraite
public abstract class Employé
{
    public string Nom { get; set; }
    public string Prénom { get; set; }

    // Propriété abstraite sans implémentation
    public abstract decimal Salaire { get; }

    public Employé(string nom, string prénom)
    {
        Nom = nom;
        Prénom = prénom;
    }

    // Méthode abstraite sans implémentation
    public abstract string ObtenirDétailsPoste();

    // Méthode concrète avec implémentation
    public string ObtenirNomComplet()
    {
        return $"{Prénom} {Nom}";
    }

    // Méthode virtuelle avec implémentation par défaut
    public virtual string ObtenirRésumé()
    {
        return $"{ObtenirNomComplet()}, {ObtenirDétailsPoste()}, Salaire: {Salaire:C}";
    }
}

// Classe dérivée qui implémente la classe abstraite
public class Développeur : Employé
{
    public string Langage { get; set; }
    public int AnnéesExpérience { get; set; }

    // Implémentation de la propriété abstraite
    public override decimal Salaire => 50000 + (AnnéesExpérience * 5000);

    public Développeur(string nom, string prénom, string langage, int annéesExpérience)
        : base(nom, prénom)
    {
        Langage = langage;
        AnnéesExpérience = annéesExpérience;
    }

    // Implémentation de la méthode abstraite
    public override string ObtenirDétailsPoste()
    {
        return $"Développeur {Langage} avec {AnnéesExpérience} ans d'expérience";
    }
}

// Autre classe dérivée
public class Gestionnaire : Employé
{
    public int NombreSubordonnés { get; set; }

    // Implémentation de la propriété abstraite
    public override decimal Salaire => 70000 + (NombreSubordonnés * 2000);

    public Gestionnaire(string nom, string prénom, int nombreSubordonnés)
        : base(nom, prénom)
    {
        NombreSubordonnés = nombreSubordonnés;
    }

    // Implémentation de la méthode abstraite
    public override string ObtenirDétailsPoste()
    {
        return $"Gestionnaire de {NombreSubordonnés} employés";
    }

    // Remplacement de la méthode virtuelle
    public override string ObtenirRésumé()
    {
        return $"MANAGER: {base.ObtenirRésumé()}";
    }
}

// Utilisation
// Employé emp = new Employé("Nom", "Prénom");  // Erreur - ne peut pas instancier une classe abstraite

Développeur dev = new Développeur("Dupont", "Jean", "C#", 5);
Console.WriteLine(dev.ObtenirRésumé());
// "Jean Dupont, Développeur C# avec 5 ans d'expérience, Salaire: 75 000,00 €"

Gestionnaire gest = new Gestionnaire("Martin", "Sophie", 10);
Console.WriteLine(gest.ObtenirRésumé());
// "MANAGER: Sophie Martin, Gestionnaire de 10 employés, Salaire: 90 000,00 €"

// Utilisation du polymorphisme
Employé employé = new Développeur("Robert", "Pierre", "Java", 3);
Console.WriteLine(employé.ObtenirRésumé());
// "Pierre Robert, Développeur Java avec 3 ans d'expérience, Salaire: 65 000,00 €"
```


### Chaînage de constructeurs

Lorsqu'une classe dérive d'une autre, les constructeurs peuvent être chaînés pour initialiser correctement l'objet :

```csharp
public class Véhicule
{
    public string Marque { get; }
    public string Modèle { get; }
    public int Année { get; }

    public Véhicule(string marque, string modèle, int année)
    {
        Console.WriteLine("Constructeur de Véhicule appelé");
        Marque = marque;
        Modèle = modèle;
        Année = année;
    }

    // Constructeur par défaut
    public Véhicule() : this("Inconnue", "Inconnu", DateTime.Now.Year)
    {
        Console.WriteLine("Constructeur par défaut de Véhicule appelé");
    }
}

public class Voiture : Véhicule
{
    public int NombrePortes { get; }
    public string TypeCarburant { get; }

    // Constructeur complet qui appelle le constructeur de la classe de base
    public Voiture(string marque, string modèle, int année, int nombrePortes, string typeCarburant)
        : base(marque, modèle, année)
    {
        Console.WriteLine("Constructeur de Voiture appelé");
        NombrePortes = nombrePortes;
        TypeCarburant = typeCarburant;
    }

    // Constructeur qui utilise des valeurs par défaut pour certains paramètres
    public Voiture(string marque, string modèle, int année)
        : this(marque, modèle, année, 5, "Essence")
    {
        Console.WriteLine("Constructeur simplifié de Voiture appelé");
    }

    // Constructeur par défaut
    public Voiture() : base()
    {
        Console.WriteLine("Constructeur par défaut de Voiture appelé");
        NombrePortes = 4;
        TypeCarburant = "Essence";
    }
}

// Utilisation
var voiture1 = new Voiture("Renault", "Clio", 2022, 5, "Diesel");
// Sortie :
// Constructeur de Véhicule appelé
// Constructeur de Voiture appelé

var voiture2 = new Voiture("Peugeot", "308", 2023);
// Sortie :
// Constructeur de Véhicule appelé
// Constructeur de Voiture appelé
// Constructeur simplifié de Voiture appelé

var voiture3 = new Voiture();
// Sortie :
// Constructeur de Véhicule appelé
// Constructeur par défaut de Véhicule appelé
// Constructeur par défaut de Voiture appelé
```


### Ordre d'initialisation

Voici l'ordre d'initialisation dans une hiérarchie de classes :

1. Champs statiques de la classe de base
2. Constructeur statique de la classe de base
3. Champs statiques de la classe dérivée
4. Constructeur statique de la classe dérivée
5. Champs d'instance de la classe de base
6. Constructeur d'instance de la classe de base
7. Champs d'instance de la classe dérivée
8. Constructeur d'instance de la classe dérivée

## 5.4.2. Méthodes virtuelles et substitution

La substitution de méthodes (override) est un mécanisme qui permet aux classes dérivées de fournir une implémentation spécifique d'une méthode définie dans une classe de base.

### Méthodes virtuelles

Pour permettre la substitution d'une méthode, celle-ci doit être déclarée avec le mot-clé `virtual` dans la classe de base :

```csharp
public class Animal
{
    public string Nom { get; set; }

    public Animal(string nom)
    {
        Nom = nom;
    }

    // Méthode virtuelle pouvant être remplacée
    public virtual string ÉmettreUnSon()
    {
        return "L'animal émet un son";
    }

    // Méthode non virtuelle qui ne peut pas être remplacée
    public string Manger()
    {
        return $"{Nom} est en train de manger";
    }
}

public class Chien : Animal
{
    public Chien(string nom) : base(nom)
    {
    }

    // Remplacement de la méthode virtuelle
    public override string ÉmettreUnSon()
    {
        return $"{Nom} aboie : Wouaf!";
    }

    // Masquage de la méthode non virtuelle (à éviter généralement)
    public new string Manger()
    {
        return $"{Nom} croque ses croquettes";
    }
}

public class Chat : Animal
{
    public Chat(string nom) : base(nom)
    {
    }

    // Remplacement de la méthode virtuelle
    public override string ÉmettreUnSon()
    {
        return $"{Nom} miaule : Miaou!";
    }
}

// Utilisation
Animal animal = new Animal("Créature");
Console.WriteLine(animal.ÉmettreUnSon());  // "L'animal émet un son"
Console.WriteLine(animal.Manger());        // "Créature est en train de manger"

Chien chien = new Chien("Rex");
Console.WriteLine(chien.ÉmettreUnSon());   // "Rex aboie : Wouaf!"
Console.WriteLine(chien.Manger());         // "Rex croque ses croquettes"

// Démonstration du polymorphisme
Animal chat = new Chat("Felix");
Console.WriteLine(chat.ÉmettreUnSon());    // "Felix miaule : Miaou!"
Console.WriteLine(chat.Manger());          // "Felix est en train de manger" (méthode de la classe de base)

// Utilisation avec une référence de classe de base
Animal chienRef = new Chien("Max");
Console.WriteLine(chienRef.ÉmettreUnSon());  // "Max aboie : Wouaf!" (méthode substituée)
Console.WriteLine(chienRef.Manger());        // "Max est en train de manger" (méthode de la classe de base)
```


### Différences entre `override` et `new`

La différence essentielle entre `override` et `new` se manifeste lors de l'utilisation du polymorphisme :

```csharp
public class ClasseBase
{
    public virtual void MéthodeVirtuelle()
    {
        Console.WriteLine("ClasseBase.MéthodeVirtuelle()");
    }

    public void MéthodeNonVirtuelle()
    {
        Console.WriteLine("ClasseBase.MéthodeNonVirtuelle()");
    }
}

public class ClasseDérivée : ClasseBase
{
    // Remplace la méthode virtuelle
    public override void MéthodeVirtuelle()
    {
        Console.WriteLine("ClasseDérivée.MéthodeVirtuelle()");
    }

    // Masque la méthode non virtuelle
    public new void MéthodeNonVirtuelle()
    {
        Console.WriteLine("ClasseDérivée.MéthodeNonVirtuelle()");
    }
}

// Utilisation
ClasseBase baseRef = new ClasseDérivée();
baseRef.MéthodeVirtuelle();      // "ClasseDérivée.MéthodeVirtuelle()" - substitution
baseRef.MéthodeNonVirtuelle();   // "ClasseBase.MéthodeNonVirtuelle()" - méthode masquée non appelée

ClasseDérivée dérivéeRef = new ClasseDérivée();
dérivéeRef.MéthodeVirtuelle();     // "ClasseDérivée.MéthodeVirtuelle()"
dérivéeRef.MéthodeNonVirtuelle();  // "ClasseDérivée.MéthodeNonVirtuelle()"
```


### Méthodes abstraites

Les méthodes abstraites sont déclarées sans implémentation dans une classe abstraite et doivent être implémentées par les classes dérivées :

```csharp
public abstract class Forme
{
    // Méthode abstraite - pas d'implémentation
    public abstract double CalculerAire();

    // Méthode abstraite avec paramètres
    public abstract bool ContientPoint(double x, double y);

    // Méthode concrète qui utilise les méthodes abstraites
    public string Décrire()
    {
        return $"Forme avec une aire de {CalculerAire()}";
    }
}

public class Cercle : Forme
{
    public double Rayon { get; set; }
    public double X { get; set; }  // Coordonnée x du centre
    public double Y { get; set; }  // Coordonnée y du centre

    public Cercle(double x, double y, double rayon)
    {
        X = x;
        Y = y;
        Rayon = rayon;
    }

    // Implémentation de la méthode abstraite
    public override double CalculerAire()
    {
        return Math.PI * Rayon * Rayon;
    }

    // Implémentation de la méthode abstraite
    public override bool ContientPoint(double x, double y)
    {
        // Calcul de la distance entre le point et le centre
        double distance = Math.Sqrt(Math.Pow(X - x, 2) + Math.Pow(Y - y, 2));
        return distance <= Rayon;
    }
}

public class Rectangle : Forme
{
    public double X { get; set; }      // Coordonnée x du coin supérieur gauche
    public double Y { get; set; }      // Coordonnée y du coin supérieur gauche
    public double Largeur { get; set; }
    public double Hauteur { get; set; }

    public Rectangle(double x, double y, double largeur, double hauteur)
    {
        X = x;
        Y = y;
        Largeur = largeur;
        Hauteur = hauteur;
    }

    // Implémentation de la méthode abstraite
    public override double CalculerAire()
    {
        return Largeur * Hauteur;
    }

    // Implémentation de la méthode abstraite
    public override bool ContientPoint(double x, double y)
    {
        return x >= X && x <= X + Largeur && y >= Y && y <= Y + Hauteur;
    }
}

// Utilisation
Forme cercle = new Cercle(0, 0, 5);
Console.WriteLine($"Aire du cercle: {cercle.CalculerAire()}");        // ~78.54
Console.WriteLine($"Le cercle contient le point (3, 4): {cercle.ContientPoint(3, 4)}");  // true

Forme rectangle = new Rectangle(0, 0, 10, 5);
Console.WriteLine($"Aire du rectangle: {rectangle.CalculerAire()}");   // 50
Console.WriteLine($"Le rectangle contient le point (7, 3): {rectangle.ContientPoint(7, 3)}");  // true
```


### Méthodes et classes scellées (sealed)

Vous pouvez empêcher qu'une méthode soit remplacée dans les classes dérivées en utilisant le mot-clé `sealed` :

```csharp
public class Animal
{
    public virtual void Respirer()
    {
        Console.WriteLine("L'animal respire");
    }
}

public class Mammifère : Animal
{
    // Remplace la méthode et empêche qu'elle soit remplacée dans les classes dérivées
    public sealed override void Respirer()
    {
        Console.WriteLine("Le mammifère respire par les poumons");
    }

    public virtual void Allaiter()
    {
        Console.WriteLine("Le mammifère allaite ses petits");
    }
}

public class Chien : Mammifère
{
    // Erreur de compilation - ne peut pas remplacer une méthode scellée
    // public override void Respirer() { }

    // OK - remplace une méthode virtuelle non scellée
    public override void Allaiter()
    {
        Console.WriteLine("La chienne allaite ses chiots");
    }
}
```


### Membre de base caché

Vous pouvez accéder à un membre masqué de la classe de base en utilisant le mot-clé `base` :

```csharp
public class A
{
    public virtual void Méthode()
    {
        Console.WriteLine("A.Méthode()");
    }
}

public class B : A
{
    public override void Méthode()
    {
        Console.WriteLine("B.Méthode() - avant appel de base");
        base.Méthode();  // Appelle la méthode de la classe de base
        Console.WriteLine("B.Méthode() - après appel de base");
    }

    public void AppelerMéthodeBase()
    {
        base.Méthode();  // Appelle directement la méthode de la classe de base
    }
}

// Utilisation
B b = new B();
b.Méthode();
// Sortie :
// B.Méthode() - avant appel de base
// A.Méthode()
// B.Méthode() - après appel de base

b.AppelerMéthodeBase();
// Sortie :
// A.Méthode()
```


## 5.4.3. Classe Object et ses méthodes

En C#, toutes les classes dérivent implicitement de la classe `System.Object` (ou `object` en notation courte). Cette classe fournit plusieurs méthodes importantes que vous pouvez remplacer pour personnaliser le comportement de vos classes.

### Méthodes principales de Object

1. **`Equals(object)`** - Détermine si l'instance actuelle est égale à l'objet spécifié.
2. **`GetHashCode()`** - Retourne un code de hachage pour l'instance actuelle.
3. **`ToString()`** - Retourne une représentation sous forme de chaîne de l'instance actuelle.
4. **`GetType()`** - Retourne le type de l'instance actuelle (ne peut pas être remplacé).
5. **`Finalize()`** - Permet à un objet de libérer des ressources avant qu'il ne soit récupéré par le garbage collector.
6. **`MemberwiseClone()`** - Crée une copie superficielle de l'objet actuel (protégé, ne peut pas être remplacé).

### Remplacement de Equals et GetHashCode

Il est important de remplacer `Equals` et `GetHashCode` de manière cohérente pour que les objets fonctionnent correctement dans les collections basées sur des tables de hachage comme `Dictionary<TKey, TValue>` ou `HashSet<T>` :

```csharp
public class Personne
{
    public string Nom { get; set; }
    public string Prénom { get; set; }
    public DateTime DateNaissance { get; set; }

    public Personne(string nom, string prénom, DateTime dateNaissance)
    {
        Nom = nom;
        Prénom = prénom;
        DateNaissance = dateNaissance;
    }

    // Remplace la méthode Equals
    public override bool Equals(object obj)
    {
        // Vérifier si null ou de type différent
        if (obj == null || GetType() != obj.GetType())
            return false;

        // Convertir en Personne
        Personne autre = (Personne)obj;

        // Comparer les valeurs des propriétés
        return Nom == autre.Nom &&
               Prénom == autre.Prénom &&
               DateNaissance == autre.DateNaissance;
    }

    // Remplace la méthode GetHashCode
    public override int GetHashCode()
    {
        // Combiner les hash codes des propriétés
        unchecked  // Permet le dépassement arithmétique
        {
            int hash = 17;  // Nombre premier arbitraire
            hash = hash * 23 + (Nom?.GetHashCode() ?? 0);  // 23 est un autre nombre premier
            hash = hash * 23 + (Prénom?.GetHashCode() ?? 0);
            hash = hash * 23 + DateNaissance.GetHashCode();
            return hash;
        }
    }

    // Surcharge de l'opérateur == (optionnel mais recommandé si vous remplacez Equals)
    public static bool operator ==(Personne a, Personne b)
    {
        // Gérer les cas où une ou les deux références sont null
        if (ReferenceEquals(a, null))
            return ReferenceEquals(b, null);

        return a.Equals(b);
    }

    // Surcharge de l'opérateur != (obligatoire si vous surchargez ==)
    public static bool operator !=(Personne a, Personne b)
    {
        return !(a == b);
    }
}

// Utilisation
var personne1 = new Personne("Dupont", "Jean", new DateTime(1980, 5, 15));
var personne2 = new Personne("Dupont", "Jean", new DateTime(1980, 5, 15));
var personne3 = new Personne("Martin", "Sophie", new DateTime(1985, 10, 25));

Console.WriteLine(personne1.Equals(personne2));  // true - mêmes valeurs
Console.WriteLine(personne1 == personne2);       // true - opérateur surchargé
Console.WriteLine(personne1.Equals(personne3));  // false - valeurs différentes

// Utilisation avec des collections basées sur un hachage
var dictionnaire = new Dictionary<Personne, string>();
dictionnaire[personne1] = "Information de Jean";

// La recherche fonctionne correctement grâce à Equals et GetHashCode
if (dictionnaire.TryGetValue(personne2, out string info))
{
    Console.WriteLine(info);  // Affiche "Information de Jean"
}
```


### Version moderne avec IEquatable<T> (.NET Framework 4.7.2 et .NET 8)

Une approche plus moderne et typée pour implémenter l'égalité est d'utiliser l'interface `IEquatable<T>` :

```csharp
public class Produit : IEquatable<Produit>
{
    public string Code { get; }
    public string Nom { get; }
    public decimal Prix { get; }

    public Produit(string code, string nom, decimal prix)
    {
        Code = code;
        Nom = nom;
        Prix = prix;
    }

    // Implémentation de IEquatable<Produit>
    public bool Equals(Produit other)
    {
        if (other == null)
            return false;

        return Code == other.Code &&
               Nom == other.Nom &&
               Prix == other.Prix;
    }

    // Remplace Equals(object)
    public override bool Equals(object obj)
    {
        return Equals(obj as Produit);
    }

    // Remplace GetHashCode()
    public override int GetHashCode()
    {
        unchecked
        {
            int hash = 17;
            hash = hash * 23 + (Code?.GetHashCode() ?? 0);
            hash = hash * 23 + (Nom?.GetHashCode() ?? 0);
            hash = hash * 23 + Prix.GetHashCode();
            return hash;
        }
    }

    // Surcharge des opérateurs d'égalité
    public static bool operator ==(Produit a, Produit b)
    {
        if (ReferenceEquals(a, null))
            return ReferenceEquals(b, null);

        return a.Equals(b);
    }

    public static bool operator !=(Produit a, Produit b)
    {
        return !(a == b);
    }
}
```


### Remplacement de ToString

La méthode `ToString` est souvent remplacée pour fournir une représentation textuelle utile de l'objet :

```csharp
public class Commande
{
    public int NuméroCommande { get; set; }
    public DateTime Date { get; set; }
    public string Client { get; set; }
    public decimal Total { get; set; }

    public Commande(int numéroCommande, string client, decimal total)
    {
        NuméroCommande = numéroCommande;
        Date = DateTime.Now;
        Client = client;
        Total = total;
    }

    // Remplace ToString pour une représentation plus utile
    public override string ToString()
    {
        return $"Commande #{NuméroCommande} - {Client} - {Date:d} - {Total:C}";
    }
}

// Utilisation
var commande = new Commande(12345, "Dupont Jean", 125.75m);
Console.WriteLine(commande);  // "Commande #12345 - Dupont Jean - 15/05/2023 - 125,75 €"
```


### Utilisation de record en .NET 8

En .NET 8 (pas disponible dans .NET Framework 4.7.2), vous pouvez utiliser le type `record` qui simplifie considérablement l'implémentation de l'égalité basée sur les valeurs :

```csharp
// Disponible uniquement dans .NET 8
public record Client(string Id, string Nom, string Email);

// Utilisation
var client1 = new Client("C1001", "Dupont Jean", "jean.dupont@example.com");
var client2 = new Client("C1001", "Dupont Jean", "jean.dupont@example.com");
var client3 = new Client("C1002", "Martin Sophie", "sophie.martin@example.com");

Console.WriteLine(client1 == client2);  // true - mêmes valeurs
Console.WriteLine(client1 == client3);  // false - valeurs différentes

// Les records fournissent aussi une implémentation de ToString()
Console.WriteLine(client1);  // Client { Id = C1001, Nom = Dupont Jean, Email = jean.dupont@example.com }

// Et une méthode de clonage avec modifications (with)
var client4 = client1 with { Email = "jean.dupont@nouvelle-adresse.com" };
Console.WriteLine(client4);  // Client { Id = C1001, Nom = Dupont Jean, Email = jean.dupont@nouvelle-adresse.com }
```


### La méthode GetType

La méthode `GetType()` héritée de `Object` ne peut pas être remplacée et retourne toujours le type réel de l'objet :

```csharp
public class A { }
public class B : A { }

// Utilisation
A a = new A();
A b = new B();  // Référence de type A vers un objet de type B

Console.WriteLine(a.GetType().Name);  // "A"
Console.WriteLine(b.GetType().Name);  // "B" - retourne le type réel, pas le type de référence
```


### La méthode Finalize (destructeur)

Le destructeur (ou finaliseur) est une méthode spéciale qui est appelée juste avant que l'objet ne soit récupéré par le garbage collector. En C#, il est implémenté avec une syntaxe de destructeur :

```csharp
public class RessourceNonGérée
{
    private IntPtr _handle;

    public RessourceNonGérée()
    {
        // Acquérir une ressource non gérée
        _handle = new IntPtr(1);
        Console.WriteLine("Ressource acquise");
    }

    // Destructeur / finaliseur
    ~RessourceNonGérée()
    {
        // Libérer la ressource non gérée
        if (_handle != IntPtr.Zero)
        {
            // Simuler la libération
            _handle = IntPtr.Zero;
            Console.WriteLine("Ressource libérée par le finaliseur");
        }
    }
}
```


> **Note**: Il est généralement préférable d'implémenter l'interface `IDisposable` plutôt que de se fier uniquement à un finaliseur, car le moment de l'exécution du finaliseur est non déterministe.

## 5.4.4. Héritage vs composition

L'héritage et la composition sont deux approches différentes pour réutiliser le code et établir des relations entre les classes.

### Héritage : "est un"

L'héritage établit une relation "est un" entre deux classes. Par exemple, un `Chien` "est un" `Animal`.

Exemple d'héritage :

```csharp
public class Document
{
    public string Titre { get; set; }
    public string Auteur { get; set; }
    public DateTime DateCréation { get; set; }

    public Document(string titre, string auteur)
    {
        Titre = titre;
        Auteur = auteur;
        DateCréation = DateTime.Now;
    }

    public virtual void Afficher()
    {
        Console.WriteLine($"Document: {Titre} par {Auteur}, créé le {DateCréation:d}");
    }

    public virtual void Enregistrer(string chemin)
    {
        Console.WriteLine($"Enregistrement du document {Titre} à {chemin}");
    }
}

public class Rapport : Document
{
    public string Département { get; set; }
    public string Sujet { get; set; }

    public Rapport(string titre, string auteur, string département, string sujet)
        : base(titre, auteur)
    {
        Département = département;
        Sujet = sujet;
    }

    public override void Afficher()
    {
        base.Afficher();
        Console.WriteLine($"Rapport du département {Département} sur {Sujet}");
    }

    public void GénérerPDF()
    {
        Console.WriteLine($"Génération du PDF pour le rapport {Titre}");
    }
}

// Utilisation
Document doc = new Document("Notes de réunion", "Jean Dupont");
doc.Afficher();

Rapport rapport = new Rapport("Rapport Q3", "Marie Martin", "Marketing", "Analyse de marché");
rapport.Afficher();
rapport.GénérerPDF();

// Polymorphisme
Document docRapport = new Rapport("Rapport financier", "Pierre Robert", "Finance", "Résultats annuels");
docRapport.Afficher();  // Appelle la méthode remplacée
// docRapport.GénérerPDF();  // Erreur - méthode non disponible via la référence Document
```


### Composition : "a un"

La composition établit une relation "a un" entre deux classes. Par exemple, une `Voiture` "a un" `Moteur`.

Exemple de composition :

```csharp
public class Moteur
{
    public int Puissance { get; set; }
    public string Type { get; set; }

    public Moteur(int puissance, string type)
    {
        Puissance = puissance;
        Type = type;
    }

    public void Démarrer()
    {
        Console.WriteLine($"Le moteur {Type} de {Puissance} chevaux démarre.");
    }

    public void Arrêter()
    {
        Console.WriteLine($"Le moteur {Type} s'arrête.");
    }
}

public class Transmission
{
    public int NombreVitesses { get; set; }
    public bool EstAutomatique { get; set; }

    public Transmission(int nombreVitesses, bool estAutomatique)
    {
        NombreVitesses = nombreVitesses;
        EstAutomatique = estAutomatique;
    }

    public void ChangerVitesse(int vitesse)
    {
        Console.WriteLine($"Changement vers la vitesse {vitesse}");
    }
}

public class Voiture
{
    // Composition - la voiture "a un" moteur et "a une" transmission
    private readonly Moteur _moteur;
    private readonly Transmission _transmission;

    public string Marque { get; set; }
    public string Modèle { get; set; }

    public Voiture(string marque, string modèle, Moteur moteur, Transmission transmission)
    {
        Marque = marque;
        Modèle = modèle;
        _moteur = moteur;
        _transmission = transmission;
    }

    public void Démarrer()
    {
        Console.WriteLine($"La voiture {Marque} {Modèle} démarre.");
        _moteur.Démarrer();
    }

    public void Arrêter()
    {
        _moteur.Arrêter();
        Console.WriteLine($"La voiture {Marque} {Modèle} s'arrête.");
    }

    public void Accélérer()
    {
        Console.WriteLine($"La voiture {Marque} {Modèle} accélère.");
        _transmission.ChangerVitesse(2);
    }

    public void AfficherSpécifications()
    {
        Console.WriteLine($"Spécifications de {Marque} {Modèle}:");
        Console.WriteLine($"Moteur: {_moteur.Type}, {_moteur.Puissance} chevaux");
        Console.WriteLine($"Transmission: {_transmission.NombreVitesses} vitesses, " +
                         $"{(_transmission.EstAutomatique ? "automatique" : "manuelle")}");
    }
}

// Utilisation
var moteur = new Moteur(150, "Essence");
var transmission = new Transmission(6, false);
var voiture = new Voiture("Peugeot", "308", moteur, transmission);

voiture.Démarrer();
voiture.Accélérer();
voiture.AfficherSpécifications();
voiture.Arrêter();
```


### Composition avec interfaces

La composition peut être utilisée avec des interfaces pour atteindre une plus grande flexibilité :

```csharp
public interface IMoteur
{
    int Puissance { get; }
    string Type { get; }
    void Démarrer();
    void Arrêter();
}

public class MoteurEssence : IMoteur
{
    public int Puissance { get; }
    public string Type => "Essence";

    public MoteurEssence(int puissance)
    {
        Puissance = puissance;
    }

    public void Démarrer()
    {
        Console.WriteLine($"Le moteur à essence de {Puissance} ch démarre avec un bruit de combustion.");
    }

    public void Arrêter()
    {
        Console.WriteLine("Le moteur à essence s'arrête.");
    }
}

public class MoteurÉlectrique : IMoteur
{
    public int Puissance { get; }
    public string Type => "Électrique";

    public MoteurÉlectrique(int puissance)
    {
        Puissance = puissance;
    }

    public void Démarrer()
    {
        Console.WriteLine($"Le moteur électrique de {Puissance} kW démarre silencieusement.");
    }

    public void Arrêter()
    {
        Console.WriteLine("Le moteur électrique s'arrête.");
    }
}

public class VoitureAvecInterface
{
    private readonly IMoteur _moteur;

    public string Marque { get; }
    public string Modèle { get; }

    public VoitureAvecInterface(string marque, string modèle, IMoteur moteur)
    {
        Marque = marque;
        Modèle = modèle;
        _moteur = moteur;
    }

    public void Démarrer()
    {
        Console.WriteLine($"La voiture {Marque} {Modèle} démarre.");
        _moteur.Démarrer();
    }

    public void Arrêter()
    {
        _moteur.Arrêter();
        Console.WriteLine($"La voiture {Marque} {Modèle} s'arrête.");
    }
}

// Utilisation
var voitureEssence = new VoitureAvecInterface("Renault", "Clio", new MoteurEssence(90));
voitureEssence.Démarrer();
voitureEssence.Arrêter();

var voitureÉlectrique = new VoitureAvecInterface("Tesla", "Model 3", new MoteurÉlectrique(250));
voitureÉlectrique.Démarrer();
voitureÉlectrique.Arrêter();
```


### Comparaison entre héritage et composition

| Aspect | Héritage | Composition |
|--------|----------|-------------|
| Relation | "Est un" | "A un" |
| Couplage | Fort | Faible |
| Flexibilité | Moins flexible | Plus flexible |
| Modification au runtime | Non | Oui, avec des interfaces |
| Réutilisation | Basée sur la classe | Basée sur l'objet |
| Effets des changements | Les changements dans la classe de base peuvent affecter les classes dérivées | Les changements d'implémentation n'affectent pas les utilisateurs si l'interface reste la même |

### Quand utiliser l'héritage ou la composition ?

#### Utilisez l'héritage quand :

1. Il existe une véritable relation "est un" entre les classes.
2. La classe de base a été conçue pour être héritée.
3. Vous souhaitez réutiliser le code de la classe de base dans plusieurs classes dérivées.
4. Vous avez besoin de polymorphisme (traiter les objets de différentes classes dérivées via une référence commune).

#### Utilisez la composition quand :

1. Il existe une relation "a un" ou "utilise un" entre les classes.
2. Vous voulez une plus grande flexibilité pour changer le comportement au runtime.
3. Vous devez éviter les problèmes du "diamant" (héritage multiple).
4. Les comportements peuvent être composés de plusieurs parties indépendantes.
5. Vous voulez un faible couplage entre les classes.

### Principes de conception SOLID et héritage

Les principes SOLID peuvent guider le choix entre héritage et composition :

1. **S - Principe de responsabilité unique**
   La composition permet de mieux séparer les responsabilités en déléguant des comportements à des classes distinctes.

2. **O - Principe d'ouverture/fermeture**
   Les deux approches peuvent respecter ce principe, mais la composition avec des interfaces est souvent plus flexible.

3. **L - Principe de substitution de Liskov**
   L'héritage doit respecter ce principe : une instance d'une classe dérivée doit pouvoir être utilisée partout où une instance de la classe de base est attendue.

4. **I - Principe de ségrégation des interfaces**
   La composition, surtout avec des interfaces, permet de fournir uniquement les comportements nécessaires.

5. **D - Principe d'inversion des dépendances**
   La composition avec des interfaces facilite l'inversion des dépendances, en dépendant des abstractions plutôt que des implémentations concrètes.

### Exemple combinant héritage et composition

Souvent, la meilleure approche est de combiner judicieusement l'héritage et la composition :

```csharp
// Interface pour les stratégies de paiement
public interface IStratégiePaiement
{
    bool Traiter(decimal montant, string référence);
}

// Implémentations concrètes des stratégies
public class PaiementCarte : IStratégiePaiement
{
    public bool Traiter(decimal montant, string référence)
    {
        Console.WriteLine($"Traitement du paiement par carte de {montant:C} pour {référence}");
        // Logique de traitement
        return true;
    }
}

public class PaiementPayPal : IStratégiePaiement
{
    public bool Traiter(decimal montant, string référence)
    {
        Console.WriteLine($"Traitement du paiement PayPal de {montant:C} pour {référence}");
        // Logique de traitement
        return true;
    }
}

// Classe de base pour les commandes
public abstract class Commande
{
    public string Référence { get; }
    public DateTime Date { get; }
    public List<LigneCommande> Lignes { get; } = new List<LigneCommande>();

    // Composition - stratégie de paiement
    protected IStratégiePaiement _stratégiePaiement;

    public Commande(string référence, IStratégiePaiement stratégiePaiement)
    {
        Référence = référence;
        Date = DateTime.Now;
        _stratégiePaiement = stratégiePaiement;
    }

    public void AjouterLigne(string produit, decimal prix, int quantité)
    {
        Lignes.Add(new LigneCommande { Produit = produit, Prix = prix, Quantité = quantité });
    }

    public decimal CalculerTotal()
    {
        return Lignes.Sum(l => l.Prix * l.Quantité);
    }

    // Méthode abstraite à implémenter par les classes dérivées
    public abstract decimal CalculerFraisLivraison();

    // Utilise la stratégie de paiement (composition)
    public bool Payer()
    {
        decimal total = CalculerTotal() + CalculerFraisLivraison();
        return _stratégiePaiement.Traiter(total, Référence);
    }

    // Change la stratégie de paiement au runtime
    public void ChangerStratégiePaiement(IStratégiePaiement nouvelleSratégie)
    {
        _stratégiePaiement = nouvelleSratégie;
    }
}

public class LigneCommande
{
    public string Produit { get; set; }
    public decimal Prix { get; set; }
    public int Quantité { get; set; }
}

// Classes dérivées pour différents types de commandes
public class CommandeNationale : Commande
{
    public CommandeNationale(string référence, IStratégiePaiement stratégiePaiement)
        : base(référence, stratégiePaiement)
    {
    }

    public override decimal CalculerFraisLivraison()
    {
        decimal total = CalculerTotal();
        return total > 100 ? 0 : 5.95m;  // Livraison gratuite au-delà de 100€
    }
}

public class CommandeInternationale : Commande
{
    public string Pays { get; }

    public CommandeInternationale(string référence, string pays, IStratégiePaiement stratégiePaiement)
        : base(référence, stratégiePaiement)
    {
        Pays = pays;
    }

    public override decimal CalculerFraisLivraison()
    {
        decimal total = CalculerTotal();
        // Frais de livraison plus élevés pour l'international
        return total > 200 ? 15 : 25;
    }
}

// Utilisation
var stratégieCarte = new PaiementCarte();
var stratégiePayPal = new PaiementPayPal();

var commandeNationale = new CommandeNationale("NAT-2023-001", stratégieCarte);
commandeNationale.AjouterLigne("Livre", 19.99m, 2);
commandeNationale.AjouterLigne("DVD", 14.99m, 1);
Console.WriteLine($"Total: {commandeNationale.CalculerTotal():C}");
Console.WriteLine($"Frais de livraison: {commandeNationale.CalculerFraisLivraison():C}");
commandeNationale.Payer();

// Changement de stratégie de paiement au runtime
commandeNationale.ChangerStratégiePaiement(stratégiePayPal);
commandeNationale.Payer();

var commandeInternationale = new CommandeInternationale("INT-2023-001", "Allemagne", stratégiePayPal);
commandeInternationale.AjouterLigne("Smartphone", 599.99m, 1);
Console.WriteLine($"Total: {commandeInternationale.CalculerTotal():C}");
Console.WriteLine($"Frais de livraison: {commandeInternationale.CalculerFraisLivraison():C}");
commandeInternationale.Payer();
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Héritage | ✅ | ✅ |
| Classes abstraites | ✅ | ✅ |
| Méthodes virtuelles | ✅ | ✅ |
| Méthodes abstraites | ✅ | ✅ |
| Méthodes scellées | ✅ | ✅ |
| Records | ❌ | ✅ |
| Membres init-only | ❌ | ✅ |
| Pattern matching amélioré | Limité | ✅ |
| Interfaces par défaut | ❌ | ✅ |

## Bonnes pratiques pour l'héritage

1. **Préférez la composition à l'héritage quand c'est approprié**
   La composition offre souvent plus de flexibilité et un couplage plus faible.

2. **Suivez le principe de substitution de Liskov**
   Une classe dérivée doit pouvoir être utilisée partout où la classe de base est attendue sans causer de problèmes.

3. **Concevez explicitement pour l'héritage ou interdisez-le**
   Utilisez `sealed` pour les classes qui ne sont pas conçues pour l'héritage.

4. **Documentez votre API d'héritage**
   Indiquez clairement quelles méthodes sont conçues pour être remplacées.

5. **Évitez l'héritage profond**
   Limitez la profondeur de la hiérarchie des classes à 3 ou 4 niveaux au maximum.

6. **Favorisez les relations d'interface plutôt que l'héritage de classe**
   Les interfaces offrent plus de flexibilité que l'héritage de classe.

7. **N'utilisez pas l'héritage juste pour la réutilisation de code**
   L'héritage est pour la modélisation des relations "est un", pas seulement pour réutiliser du code.

8. **Évitez d'appeler des méthodes virtuelles dans les constructeurs**
   Cela peut conduire à des comportements inattendus si la méthode est remplacée dans une classe dérivée.

9. **Respectez le contrat de la classe de base dans les méthodes remplacées**
   Les méthodes substituées ne doivent pas affaiblir les préconditions ni renforcer les postconditions.

10. **Utilisez les mots-clés `base` et `this` pour clarifier l'accès aux membres**
    Cela rend le code plus lisible, surtout lorsqu'il y a des noms similaires dans les classes de base et dérivées.

## Conclusion

L'héritage est un mécanisme puissant en C# qui permet de créer des hiérarchies de classes et de réutiliser le code. Il établit une relation "est un" entre les classes et facilite le polymorphisme. Cependant, il crée également un couplage fort entre les classes.

La composition offre une alternative plus flexible en établissant des relations "a un" entre les classes. Elle permet de modifier le comportement au runtime et crée un couplage plus faible.

Dans la pratique, il est souvent judicieux de combiner ces deux approches : utiliser l'héritage pour les véritables relations "est un" et la composition pour les comportements qui peuvent varier indépendamment. Les principes SOLID et les patterns de conception comme la Stratégie ou le Décorateur peuvent guider ces choix d'architecture.

L'héritage et la composition sont tous deux disponibles dans .NET Framework 4.7.2 et .NET 8, mais .NET 8 offre des fonctionnalités supplémentaires comme les records et les interfaces par défaut qui peuvent influencer la façon dont vous modélisez vos classes et leurs relations.
    public override int GetHashCode()
    {
        // Combiner les hash codes des propriétés
        unchecked  // Permet le dépassement arithmétique
        {
            int hash = 17;  // Nombre premier arbitraire
            hash = hash * 23 + (Nom?.GetHashCode() ?? 0);  // 23 est un autre nombre premier
            hash = hash * 23 + (Prénom?.GetHashCode() ?? 0);
            hash = hash * 23 + DateNaissance.GetHashCode();
            return hash;
        }
    }

    // Surcharge de l'opérateur == (optionnel mais recommandé si vous remplacez Equals)
    public static bool operator ==(Personne a, Personne b)
    {
        // Gérer les cas où une ou les deux références sont null
        if (ReferenceEquals(a, null))
            return ReferenceEquals(b, null);

        return a.Equals(b);
    }

    // Surcharge de l'opérateur != (obligatoire si vous surchargez ==)
    public static bool operator !=(Personne a, Personne b)
    {
        return !(a == b);
    }
}

// Utilisation
var personne1 = new Personne("Dupont", "Jean", new DateTime(1980, 5, 15));
var personne
```

⏭️
