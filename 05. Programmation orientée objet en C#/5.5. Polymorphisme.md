# 5.5. Polymorphisme

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le polymorphisme est l'un des piliers fondamentaux de la programmation orientée objet. En C#, il permet à des objets de types différents d'être traités comme des objets d'un type commun, tout en conservant leurs comportements spécifiques. Ce concept puissant permet d'écrire du code plus flexible, extensible et réutilisable.

## 5.5.1. Polymorphisme d'inclusion

Le polymorphisme d'inclusion (aussi appelé polymorphisme par sous-typage) permet à une référence d'un type de base de désigner un objet d'un type dérivé. C'est la forme la plus courante de polymorphisme en C#.

```csharp
// Classe de base
public class Animal
{
    public virtual void FaireBruit()
    {
        Console.WriteLine("L'animal fait un bruit");
    }
}

// Classes dérivées
public class Chien : Animal
{
    public override void FaireBruit()
    {
        Console.WriteLine("Le chien aboie");
    }
}

public class Chat : Animal
{
    public override void FaireBruit()
    {
        Console.WriteLine("Le chat miaule");
    }
}

// Utilisation polymorphique
public class Program
{
    public static void Main()
    {
        // Polymorphisme : une référence de type Animal peut désigner
        // des objets de type Chien ou Chat
        Animal[] animaux = new Animal[2];
        animaux[0] = new Chien();
        animaux[1] = new Chat();

        foreach (Animal animal in animaux)
        {
            // La méthode appelée dépend du type réel de l'objet,
            // pas du type de la référence
            animal.FaireBruit();
        }
    }
}

// Sortie :
// Le chien aboie
// Le chat miaule
```


### Points clés :
- Les méthodes doivent être marquées comme `virtual` dans la classe de base
- Les classes dérivées utilisent `override` pour remplacer le comportement
- Le comportement exécuté dépend du type réel de l'objet à l'exécution, pas du type de référence

## 5.5.2. Polymorphisme ad hoc (surcharge)

Le polymorphisme ad hoc permet d'avoir plusieurs méthodes avec le même nom mais des signatures différentes dans une même classe. C'est ce qu'on appelle la surcharge de méthodes.

```csharp
public class Calculateur
{
    // Surcharge pour les entiers
    public int Additionner(int a, int b)
    {
        return a + b;
    }

    // Surcharge pour les décimaux
    public double Additionner(double a, double b)
    {
        return a + b;
    }

    // Surcharge avec un nombre variable de paramètres
    public int Additionner(params int[] nombres)
    {
        int somme = 0;
        foreach (var nombre in nombres)
        {
            somme += nombre;
        }
        return somme;
    }
}

public class Program
{
    public static void Main()
    {
        Calculateur calc = new Calculateur();

        // Appel de différentes versions selon les arguments fournis
        Console.WriteLine(calc.Additionner(5, 10));           // Appelle la première surcharge
        Console.WriteLine(calc.Additionner(5.5, 10.5));       // Appelle la deuxième surcharge
        Console.WriteLine(calc.Additionner(1, 2, 3, 4, 5));   // Appelle la troisième surcharge
    }
}
```


### Points clés :
- Les méthodes surchargées doivent avoir des signatures différentes (types ou nombre de paramètres)
- Le compilateur détermine quelle surcharge appeler en fonction des arguments fournis
- La résolution se fait à la compilation (et non à l'exécution comme pour le polymorphisme d'inclusion)

## 5.5.3. Polymorphisme paramétrique (génériques)

Le polymorphisme paramétrique permet d'écrire des classes et des méthodes qui fonctionnent avec n'importe quel type tout en préservant la sécurité des types. En C#, cela se fait grâce aux génériques.

```csharp
// Classe générique
public class File<T>
{
    private readonly List<T> elements = new List<T>();

    public void Ajouter(T element)
    {
        elements.Add(element);
    }

    public T Retirer()
    {
        if (elements.Count == 0)
        {
            throw new InvalidOperationException("La file est vide");
        }

        T premier = elements[0];
        elements.RemoveAt(0);
        return premier;
    }

    public int Compter()
    {
        return elements.Count;
    }
}

// Méthode générique
public static class Utilitaires
{
    public static void Echanger<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }
}

public class Program
{
    public static void Main()
    {
        // File d'entiers
        File<int> fileEntiers = new File<int>();
        fileEntiers.Ajouter(1);
        fileEntiers.Ajouter(2);
        Console.WriteLine(fileEntiers.Retirer());  // 1

        // File de chaînes
        File<string> fileChaines = new File<string>();
        fileChaines.Ajouter("Bonjour");
        fileChaines.Ajouter("Monde");
        Console.WriteLine(fileChaines.Retirer());  // Bonjour

        // Utilisation de la méthode générique
        int x = 10, y = 20;
        Utilitaires.Echanger(ref x, ref y);
        Console.WriteLine($"x = {x}, y = {y}");  // x = 20, y = 10
    }
}
```


### Points clés :
- Les génériques permettent d'écrire du code qui fonctionne avec n'importe quel type
- La vérification de type est préservée à la compilation
- Ils offrent de meilleures performances que d'utiliser `object` car ils évitent le boxing/unboxing
- Il est possible d'appliquer des contraintes sur les types génériques (`where T : ...)

## 5.5.4. Cast et is/as

Quand on travaille avec du code polymorphique, on a parfois besoin de traiter un objet selon son type concret plutôt que son type de base. C# offre plusieurs mécanismes pour cela.

```csharp
public class Program
{
    public static void Main()
    {
        // Création d'un tableau polymorphique
        Animal[] animaux = new Animal[]
        {
            new Chien(),
            new Chat(),
            new Animal()
        };

        foreach (Animal animal in animaux)
        {
            // 1. Utilisation de l'opérateur is pour vérifier le type
            if (animal is Chien)
            {
                Console.WriteLine("C'est un chien!");
                // Cast explicite - génère une exception si le cast échoue
                Chien chien = (Chien)animal;
                // Méthode spécifique à Chien
                chien.Aboyer();
            }

            // 2. Utilisation de l'opérateur as pour tenter un cast sécurisé
            Chat chat = animal as Chat;
            if (chat != null)
            {
                Console.WriteLine("C'est un chat!");
                // Méthode spécifique à Chat
                chat.Miauler();
            }

            // 3. Pattern matching (C# 7+)
            if (animal is Chien chien2)
            {
                // chien2 est automatiquement castée et utilisable ici
                chien2.Aboyer();
            }

            // 4. Switch expression avec pattern matching (C# 8+)
            string description = animal switch
            {
                Chien d => $"Un chien qui {d.Race}",
                Chat c => $"Un chat de couleur {c.Couleur}",
                _ => "Un animal quelconque"
            };
            Console.WriteLine(description);
        }
    }
}

// Classes étendues pour les exemples
public class Animal
{
    public virtual void FaireBruit()
    {
        Console.WriteLine("L'animal fait un bruit");
    }
}

public class Chien : Animal
{
    public string Race { get; set; } = "labrador";

    public override void FaireBruit()
    {
        Console.WriteLine("Le chien aboie");
    }

    public void Aboyer()
    {
        Console.WriteLine("Woof!");
    }
}

public class Chat : Animal
{
    public string Couleur { get; set; } = "noir";

    public override void FaireBruit()
    {
        Console.WriteLine("Le chat miaule");
    }

    public void Miauler()
    {
        Console.WriteLine("Miaou!");
    }
}
```


### Points clés :

#### Opérateur `is`
- Vérifie si un objet est compatible avec un type donné
- Retourne `true` ou `false`
- Peut être utilisé avec pattern matching pour créer une variable typée (`is Type variable`)

#### Opérateur `as`
- Tente de convertir un objet vers un type spécifié
- Retourne `null` si la conversion échoue (au lieu de lever une exception)
- Ne fonctionne qu'avec les types référence (pas avec les structs)

#### Cast explicite `(Type)`
- Force la conversion d'un objet vers un type spécifié
- Lève une `InvalidCastException` si la conversion échoue
- Peut être utilisé avec les types valeur et référence

#### Pattern matching
- Syntaxe plus concise introduite dans C# 7+
- Combine la vérification de type et le cast en une seule opération
- Particulièrement utile dans les expressions switch (C# 8+)

Le polymorphisme, combiné avec ces mécanismes de conversion et vérification de types, permet d'écrire du code flexible qui s'adapte dynamiquement au type réel des objets manipulés.

⏭️
