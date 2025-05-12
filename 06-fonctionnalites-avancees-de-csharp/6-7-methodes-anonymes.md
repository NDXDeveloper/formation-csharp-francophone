# 6.7. Méthodes anonymes

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les méthodes anonymes sont une fonctionnalité importante de C# qui permet de définir des fonctions sans avoir à les déclarer explicitement avec un nom. Cette section explore les méthodes anonymes, leur comparaison avec les expressions lambda, et les cas d'utilisation appropriés.

## 6.7.1. Comparaison avec lambdas

Les méthodes anonymes et les expressions lambda sont deux façons de créer des délégués en C#. Bien que les expressions lambda aient en grande partie remplacé les méthodes anonymes dans le code moderne, il est important de comprendre les deux approches.

### Syntaxe de base

#### Méthode anonyme traditionnelle

```textmate
// Méthode anonyme avec le mot-clé delegate
Button button = new Button();
button.Click += delegate(object sender, EventArgs e)
{
    MessageBox.Show("Bouton cliqué");
};
```


#### Expression lambda équivalente

```textmate
// Expression lambda équivalente (syntaxe plus concise)
Button button = new Button();
button.Click += (sender, e) => MessageBox.Show("Bouton cliqué");
```


### Délégués avec méthodes anonymes

Les méthodes anonymes utilisent le mot-clé `delegate` :

```textmate
// Définition d'un délégué
public delegate int Calculateur(int x, int y);

// Utilisation avec une méthode anonyme
Calculateur addition = delegate(int a, int b)
{
    return a + b;
};

// Appel du délégué
int résultat = addition(5, 3);  // 8
```


### Délégués avec expressions lambda

Les expressions lambda utilisent l'opérateur `=>` (parfois appelé opérateur "goes to" ou "fat arrow") :

```textmate
// Définition d'un délégué
public delegate int Calculateur(int x, int y);

// Utilisation avec une expression lambda
Calculateur addition = (a, b) => a + b;

// Appel du délégué
int résultat = addition(5, 3);  // 8
```


### Comparaison syntaxique détaillée

Voici une comparaison plus complète des différentes syntaxes :

```textmate
// Délégué pour l'exemple
delegate void MonDélégué(string message);

public void DémontrerDifférentesSyntaxes()
{
    // 1. Méthode anonyme traditionnelle
    MonDélégué méthode1 = delegate(string msg)
    {
        Console.WriteLine($"Méthode anonyme : {msg}");
    };

    // 2. Expression lambda avec paramètre et bloc d'instructions
    MonDélégué méthode2 = (string msg) =>
    {
        Console.WriteLine($"Lambda avec bloc : {msg}");
    };

    // 3. Expression lambda avec inférence de type
    MonDélégué méthode3 = (msg) =>
    {
        Console.WriteLine($"Lambda avec inférence : {msg}");
    };

    // 4. Expression lambda concise (un seul paramètre, sans parenthèses)
    MonDélégué méthode4 = msg =>
    {
        Console.WriteLine($"Lambda concise : {msg}");
    };

    // 5. Expression lambda avec expression simple (sans accolades)
    MonDélégué méthode5 = msg => Console.WriteLine($"Lambda expression : {msg}");

    // Appel des différentes méthodes
    méthode1("Test 1");
    méthode2("Test 2");
    méthode3("Test 3");
    méthode4("Test 4");
    méthode5("Test 5");
}
```


### Différences fonctionnelles principales

Bien que similaires dans leur fonction, les méthodes anonymes et les expressions lambda présentent quelques différences importantes :

#### 1. Syntaxe et concision

Les lambdas sont généralement plus concises, en particulier pour les expressions simples :

```textmate
// Méthode anonyme
Func<int, bool> estPair = delegate(int n) { return n % 2 == 0; };

// Lambda équivalente - plus concise
Func<int, bool> estPairLambda = n => n % 2 == 0;
```


#### 2. Inférence de type

Les expressions lambda permettent l'inférence de type des paramètres, tandis que les méthodes anonymes exigent des types explicites :

```textmate
// Méthode anonyme - types explicites requis
Func<int, int, int> somme1 = delegate(int a, int b) { return a + b; };

// Lambda - types inférés du contexte
Func<int, int, int> somme2 = (a, b) => a + b;
```


#### 3. Expression sans paramètres

La syntaxe diffère pour les délégués sans paramètres :

```textmate
// Méthode anonyme sans paramètres
Action action1 = delegate() { Console.WriteLine("Hello"); };

// Lambda sans paramètres
Action action2 = () => Console.WriteLine("Hello");
```


#### 4. Capture de variables

Tant les méthodes anonymes que les lambdas peuvent capturer des variables du contexte environnant, mais les lambdas ont un comportement plus prévisible et consistent :

```textmate
for (int i = 0; i < 5; i++)
{
    // Les deux capturent la variable i

    // Méthode anonyme
    Action action1 = delegate() { Console.WriteLine($"Méthode anonyme: {i}"); };

    // Lambda équivalente
    Action action2 = () => Console.WriteLine($"Lambda: {i}");
}
```


#### 5. Compatibilité avec LINQ et types génériques

Les lambdas s'intègrent mieux avec LINQ et les délégués génériques comme `Func<>` et `Action<>` :

```textmate
List<int> nombres = new List<int> { 1, 2, 3, 4, 5 };

// Lambda avec LINQ - très lisible
var pairs = nombres.Where(n => n % 2 == 0);

// Méthode anonyme avec LINQ - plus verbeux
var pairs2 = nombres.Where(delegate(int n) { return n % 2 == 0; });
```


#### 6. Conversion implicite en types d'arbre d'expression

Les lambdas peuvent être converties implicitement en types `Expression<T>`, ce qui n'est pas possible avec les méthodes anonymes :

```textmate
// Fonctionne avec lambda - peut être convertie en arbre d'expression
Expression<Func<int, int>> doubler = x => x * 2;

// Ne compile pas - les méthodes anonymes ne peuvent pas être converties en arbres d'expression
// Expression<Func<int, int>> doubler2 = delegate(int x) { return x * 2; };
```


Cette capacité est particulièrement importante pour les bibliothèques comme Entity Framework qui analysent les expressions pour les convertir en requêtes SQL.

### Évolution historique

Un aperçu de l'évolution des délégués en C# :

| Version C# | Fonctionnalité | Exemple |
|------------|----------------|---------|
| 1.0 (.NET Framework) | Délégués nommés | `public void MonGestionnaire(object sender, EventArgs e) { ... }` |
| 2.0 (.NET Framework 2.0) | Méthodes anonymes | `delegate(int x) { return x * 2; }` |
| 3.0 (.NET Framework 3.5) | Expressions lambda | `x => x * 2` |
| 3.0+ | Expression lambda multi-lignes | `x => { var temp = x * 2; return temp; }` |
| 7.0+ (.NET 6/7/8) | Lambdas améliorées | `Func<int, int> f = x => x * 2;` (inférence de type améliorée) |

## 6.7.2. Cas d'usage

Les méthodes anonymes et les expressions lambda ont de nombreux cas d'utilisation dans le développement C# moderne. Bien que les expressions lambda soient généralement préférées aujourd'hui, il existe encore des situations où les méthodes anonymes traditionnelles peuvent être appropriées.

### Gestion d'événements

L'un des cas d'utilisation les plus courants pour les méthodes anonymes est la gestion d'événements, en particulier pour les gestionnaires simples :

#### .NET Framework 4.7.2

```textmate
// WinForms (.NET Framework)
private void InitializeButton()
{
    Button btnSubmit = new Button();
    btnSubmit.Text = "Soumettre";

    // Gestionnaire d'événement avec méthode anonyme
    btnSubmit.Click += delegate(object sender, EventArgs e)
    {
        // Traitement local qui ne nécessite pas une méthode séparée
        MessageBox.Show("Formulaire soumis");
        SaveData();
    };

    // Avec lambda (plus moderne)
    btnSubmit.Click += (sender, e) =>
    {
        MessageBox.Show("Formulaire soumis");
        SaveData();
    };

    this.Controls.Add(btnSubmit);
}
```


#### .NET 8

```textmate
// WPF (.NET 8)
private void InitializeButton()
{
    Button btnSubmit = new Button();
    btnSubmit.Content = "Soumettre";

    // Gestionnaire d'événement avec lambda (approche recommandée)
    btnSubmit.Click += (sender, e) =>
    {
        MessageBox.Show("Formulaire soumis");
        await SaveDataAsync();
    };

    // Moins courant mais toujours possible avec méthode anonyme
    btnSubmit.Click += delegate(object sender, RoutedEventArgs e)
    {
        MessageBox.Show("Formulaire soumis via méthode anonyme");
        SaveData();
    };

    mainGrid.Children.Add(btnSubmit);
}
```


### Callbacks et opérations asynchrones

Les méthodes anonymes et lambdas sont très utiles pour les callbacks, particulièrement dans les opérations asynchrones :

#### .NET Framework 4.7.2

```textmate
// Approche avec Task et continuation
public void ChargerDonnéesAsync()
{
    Task.Run(() => ChargementLongue())
        .ContinueWith(delegate(Task tâchePrécédente)
        {
            if (tâchePrécédente.Exception != null)
            {
                // Gestion des erreurs
                Console.WriteLine($"Erreur: {tâchePrécédente.Exception.InnerException.Message}");
                return;
            }

            // Traitement après chargement
            Console.WriteLine("Chargement terminé");
        });

    // Avec lambda (plus moderne et concis)
    Task.Run(() => ChargementLongue())
        .ContinueWith(tâchePrécédente =>
        {
            if (tâchePrécédente.Exception != null)
            {
                Console.WriteLine($"Erreur: {tâchePrécédente.Exception.InnerException.Message}");
                return;
            }

            Console.WriteLine("Chargement terminé");
        });
}

private void ChargementLongue()
{
    // Simulation d'une opération longue
    Thread.Sleep(2000);
}
```


#### .NET 8

```textmate
// Approche moderne avec async/await
public async Task ChargerDonnéesAsync()
{
    try
    {
        await Task.Run(() => ChargementLongue());
        Console.WriteLine("Chargement terminé");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erreur: {ex.Message}");
    }
}

// Utilisation avec HttpClient et lambdas
public async Task ObtenirDonnéesAPIAsync()
{
    using HttpClient client = new HttpClient();

    try
    {
        string url = "https://api.exemple.com/data";
        HttpResponseMessage response = await client.GetAsync(url);

        if (response.IsSuccessStatusCode)
        {
            string json = await response.Content.ReadAsStringAsync();

            // Traitement JSON avec lambda
            await Task.Run(() => TraiterDonnéesJSON(json));
        }
        else
        {
            Console.WriteLine($"Erreur API: {response.StatusCode}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erreur réseau: {ex.Message}");
    }
}

private void TraiterDonnéesJSON(string json)
{
    // Traitement des données JSON
}
```


### Filtrage et transformation avec LINQ

LINQ est l'un des domaines où les expressions lambda brillent vraiment, remplaçant presque entièrement les méthodes anonymes :

```textmate
public void ExemplesLINQ()
{
    List<int> nombres = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    // Filtrage avec méthode anonyme (style ancien)
    var nombresPairs = nombres.Where(delegate(int n) { return n % 2 == 0; }).ToList();

    // Filtrage avec lambda (style moderne)
    var nombresPairs2 = nombres.Where(n => n % 2 == 0).ToList();

    // Transformation avec lambda
    var nombresDoublés = nombres.Select(n => n * 2).ToList();

    // Filtrage et transformation chaînés
    var résultat = nombres
        .Where(n => n > 5)
        .Select(n => new { Valeur = n, Carré = n * n })
        .OrderByDescending(item => item.Carré)
        .ToList();

    // Affichage des résultats
    foreach (var item in résultat)
    {
        Console.WriteLine($"Valeur: {item.Valeur}, Carré: {item.Carré}");
    }
}
```


### Création de délégués complexes

Pour les délégués plus complexes ou les besoins d'encapsulation spécifiques :

```textmate
public void DéléguésComplexes()
{
    // Définition d'une variable locale à capturer
    string préfixe = "ID-";
    int compteur = 0;

    // Création d'un générateur d'ID avec une méthode anonyme qui capture des variables
    Func<string> générerID = delegate()
    {
        compteur++;
        return $"{préfixe}{compteur:D4}";
    };

    // Utilisation du délégué
    Console.WriteLine(générerID());  // "ID-0001"
    Console.WriteLine(générerID());  // "ID-0002"

    // Création d'un validateur avec une lambda
    Func<string, bool> estEmailValide = email =>
    {
        if (string.IsNullOrEmpty(email))
            return false;

        // Vérification simple de format d'email
        return email.Contains("@") && email.Contains(".");
    };

    // Test du validateur
    Console.WriteLine(estEmailValide("test@exemple.com"));  // true
    Console.WriteLine(estEmailValide("email-invalide"));    // false
}
```


### Créer des fonctionnalités personnalisables

Les méthodes anonymes et lambdas permettent de créer des composants dont le comportement est personnalisable :

```textmate
// Classe avec comportement personnalisable
public class ProcesseurTexte
{
    private Func<string, string> _transformateur;

    // Constructeur avec transformation personnalisable
    public ProcesseurTexte(Func<string, string> transformateur)
    {
        _transformateur = transformateur ?? (texte => texte);
    }

    public string Traiter(string texte)
    {
        if (string.IsNullOrEmpty(texte))
            return string.Empty;

        return _transformateur(texte);
    }
}

// Utilisation
public void UtiliserProcesseurTexte()
{
    // Création avec méthode anonyme
    var processeurMajuscules = new ProcesseurTexte(delegate(string texte)
    {
        return texte.ToUpper();
    });

    // Création avec lambda
    var processeurInverse = new ProcesseurTexte(texte => new string(texte.Reverse().ToArray()));

    var processeurPersonnalisé = new ProcesseurTexte(texte =>
    {
        // Transformation plus complexe
        return texte
            .Replace("a", "@")
            .Replace("e", "3")
            .Replace("i", "1")
            .Replace("o", "0");
    });

    // Test des différents processeurs
    string texteOriginal = "Bonjour le monde";

    Console.WriteLine(processeurMajuscules.Traiter(texteOriginal));  // "BONJOUR LE MONDE"
    Console.WriteLine(processeurInverse.Traiter(texteOriginal));     // "ednom el ruojnoB"
    Console.WriteLine(processeurPersonnalisé.Traiter(texteOriginal)); // "B0nj0ur l3 m0nd3"
}
```


### Tris personnalisés

Les méthodes anonymes ou lambdas sont idéales pour définir des critères de tri personnalisés :

```textmate
public class Personne
{
    public string Nom { get; set; }
    public string Prénom { get; set; }
    public DateTime DateNaissance { get; set; }

    public override string ToString()
    {
        return $"{Prénom} {Nom} ({DateNaissance:d})";
    }
}

public void TriPersonnalisé()
{
    List<Personne> personnes = new List<Personne>
    {
        new Personne { Nom = "Dupont", Prénom = "Jean", DateNaissance = new DateTime(1980, 5, 15) },
        new Personne { Nom = "Martin", Prénom = "Sophie", DateNaissance = new DateTime(1992, 11, 3) },
        new Personne { Nom = "Dubois", Prénom = "Pierre", DateNaissance = new DateTime(1985, 7, 21) },
        new Personne { Nom = "Durand", Prénom = "Marie", DateNaissance = new DateTime(1988, 3, 10) }
    };

    // Tri par nom avec méthode anonyme
    personnes.Sort(delegate(Personne p1, Personne p2)
    {
        return string.Compare(p1.Nom, p2.Nom, StringComparison.Ordinal);
    });

    Console.WriteLine("Tri par nom:");
    personnes.ForEach(p => Console.WriteLine(p));

    // Tri par date de naissance avec lambda
    personnes.Sort((p1, p2) => DateTime.Compare(p1.DateNaissance, p2.DateNaissance));

    Console.WriteLine("\nTri par date de naissance:");
    personnes.ForEach(p => Console.WriteLine(p));

    // Tri par prénom puis nom avec lambda
    personnes.Sort((p1, p2) =>
    {
        int comparaisonPrénom = string.Compare(p1.Prénom, p2.Prénom, StringComparison.Ordinal);
        if (comparaisonPrénom != 0)
            return comparaisonPrénom;

        return string.Compare(p1.Nom, p2.Nom, StringComparison.Ordinal);
    });

    Console.WriteLine("\nTri par prénom puis nom:");
    personnes.ForEach(p => Console.WriteLine(p));
}
```


### Mise en œuvre de stratégies

Le pattern Stratégie peut être implémenté simplement avec des lambdas :

```textmate
// Définition de l'interface de stratégie
public interface IStratégieTaxe
{
    decimal CalculerTaxe(decimal montant);
}

// Implémentation traditionnelle
public class TaxeFrance : IStratégieTaxe
{
    public decimal CalculerTaxe(decimal montant)
    {
        return montant * 0.2m; // TVA 20%
    }
}

// Classe de contexte qui peut utiliser des lambdas comme stratégies
public class CalculateurTaxe
{
    private Func<decimal, decimal> _stratégie;

    // Constructeur avec stratégie sous forme de lambda
    public CalculateurTaxe(Func<decimal, decimal> stratégie)
    {
        _stratégie = stratégie ?? throw new ArgumentNullException(nameof(stratégie));
    }

    // Surcharge pour utilisation avec interface traditionnelle
    public CalculateurTaxe(IStratégieTaxe stratégie)
    {
        if (stratégie == null) throw new ArgumentNullException(nameof(stratégie));
        _stratégie = stratégie.CalculerTaxe;
    }

    public decimal CalculerTaxe(decimal montant)
    {
        return _stratégie(montant);
    }
}

// Utilisation
public void DémontrerStratégies()
{
    // Utilisation avec classe traditionnelle
    var calculateur1 = new CalculateurTaxe(new TaxeFrance());

    // Utilisation avec lambda
    var calculateur2 = new CalculateurTaxe(montant => montant * 0.19m); // TVA Allemagne

    // Avec méthode anonyme
    var calculateur3 = new CalculateurTaxe(delegate(decimal montant)
    {
        // Logique plus complexe pour le Luxembourg
        if (montant <= 100)
            return montant * 0.17m;
        else
            return montant * 0.14m;
    });

    decimal montantHT = 1000;

    Console.WriteLine($"Montant HT: {montantHT:C}");
    Console.WriteLine($"TVA France: {calculateur1.CalculerTaxe(montantHT):C}");
    Console.WriteLine($"TVA Allemagne: {calculateur2.CalculerTaxe(montantHT):C}");
    Console.WriteLine($"TVA Luxembourg: {calculateur3.CalculerTaxe(montantHT):C}");
}
```


### Amélioration de la lisibilité avec des expressions lambda

Les expressions lambda peuvent significativement améliorer la lisibilité pour des expressions conditionnelles complexes :

```textmate
public void DémontrerLisibilité()
{
    List<Produit> produits = new List<Produit>
    {
        new Produit { Id = 1, Nom = "Ordinateur", Prix = 1200, Catégorie = "Informatique", EnStock = true },
        new Produit { Id = 2, Nom = "Smartphone", Prix = 800, Catégorie = "Téléphonie", EnStock = true },
        new Produit { Id = 3, Nom = "Imprimante", Prix = 300, Catégorie = "Informatique", EnStock = false },
        new Produit { Id = 4, Nom = "Clavier", Prix = 50, Catégorie = "Accessoires", EnStock = true }
    };

    // Approche traditionnelle avec condition complexe
    List<Produit> résultatsFiltrage = new List<Produit>();
    foreach (var produit in produits)
    {
        if ((produit.Catégorie == "Informatique" || produit.Catégorie == "Accessoires")
            && produit.Prix < 500
            && produit.EnStock)
        {
            résultatsFiltrage.Add(produit);
        }
    }

    // Même logique avec LINQ et lambda - plus lisible
    var résultatsFiltrageLambda = produits
        .Where(p => (p.Catégorie == "Informatique" || p.Catégorie == "Accessoires")
               && p.Prix < 500
               && p.EnStock)
        .ToList();

    // Lambda dans une variable pour améliorer la lisibilité
    Func<Produit, bool> estAccessoireInformatiquePasCher = p =>
        (p.Catégorie == "Informatique" || p.Catégorie == "Accessoires")
        && p.Prix < 500
        && p.EnStock;

    var résultatsFiltrageLisible = produits.Where(estAccessoireInformatiquePasCher).ToList();

    // Affichage
    foreach (var produit in résultatsFiltrageLisible)
    {
        Console.WriteLine($"{produit.Nom} - {produit.Prix:C}");
    }
}

public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
    public string Catégorie { get; set; }
    public bool EnStock { get; set; }
}
```


### Parallélisme et Lambdas (.NET 8)

En .NET 8, les lambdas s'intègrent parfaitement avec les fonctionnalités de parallélisme :

```textmate
public async Task TraitementParallèleAsync()
{
    // Création d'une liste de tâches à traiter
    List<int> éléments = Enumerable.Range(1, 100).ToList();

    // Traitement parallèle avec PLINQ
    var résultat1 = éléments
        .AsParallel()
        .Where(n => n % 2 == 0)
        .Select(n =>
        {
            Console.WriteLine($"Traitement de {n} sur thread {Thread.CurrentThread.ManagedThreadId}");
            return n * n;
        })
        .ToList();

    // Traitement asynchrone parallèle avec ForEachAsync (.NET 8)
    var options = new ParallelOptions
    {
        MaxDegreeOfParallelism = Environment.ProcessorCount
    };

    await Parallel.ForEachAsync(éléments, options, async (élément, token) =>
    {
        // Simulation d'une opération asynchrone
        await Task.Delay(50, token);
        Console.WriteLine($"Traitement asynchrone de {élément} sur thread {Thread.CurrentThread.ManagedThreadId}");
    });

    // TaskCompletionSource avec lambda
    var tcs = new TaskCompletionSource<bool>();

    // Simuler une opération asynchrone
    Task.Run(async () =>
    {
        await Task.Delay(1000);
        tcs.SetResult(true);
    });

    // Attendre le résultat
    bool résultat = await tcs.Task;
    Console.WriteLine($"Opération terminée avec résultat: {résultat}");
}
```


### Bonnes pratiques avec les méthodes anonymes et lambdas

Voici quelques bonnes pratiques pour l'utilisation des méthodes anonymes et lambdas :

1. **Préférer les lambdas aux méthodes anonymes** - Les lambdas offrent une syntaxe plus concise et moderne.

2. **Garder les expressions simples** - Si une lambda devient trop complexe, envisagez de la remplacer par une méthode nommée.

```textmate
// Trop complexe pour une lambda
   button.Click += (sender, e) =>
   {
       // Beaucoup de lignes de code...
       // Logique complexe...
       // Plus de code...
   };

   // Mieux - méthode nommée
   button.Click += Button_Click;

   private void Button_Click(object sender, EventArgs e)
   {
       // Code complexe ici...
   }
```


3. **Attention aux variables capturées** - Les variables capturées dans les closures peuvent causer des comportements inattendus.

```textmate
// Problème potentiel avec la capture de variables
   var actions = new List<Action>();

   for (int i = 0; i < 5; i++)
   {
       actions.Add(() => Console.WriteLine(i));
   }

   // Toutes les actions afficheront 5, pas les valeurs 0,1,2,3,4
   foreach (var action in actions)
   {
       action();
   }

   // Solution - capture la valeur actuelle, pas la variable
   var actionsCorirgées = new List<Action>();

   for (int i = 0; i < 5; i++)
   {
       int capturedI = i;  // Création d'une copie locale à capturer
       actionsCorirgées.Add(() => Console.WriteLine(capturedI));
   }

   // Affichera 0,1,2,3,4 comme prévu
   foreach (var action in actionsCorirgées)
   {
       action();
   }
```


4. **Utiliser des lambdas pour améliorer la lisibilité** - Spécialement avec LINQ.

5. **Considérer la performance** - Dans les scénarios critiques en termes de performance, vérifiez l'impact des allocations de délégués.

```textmate
// Peut causer des allocations à chaque appel
   public IEnumerable<int> FiltrerNombres(IEnumerable<int> source, int seuil)
   {
       return source.Where(n => n > seuil);
   }

   // Réutilisation de la lambda pour réduire les allocations
   private static readonly Func<int, int, bool> _filtreSupérieur = (n, seuil) => n > seuil;

   public IEnumerable<int> FiltrerNombresOptimisé(IEnumerable<int> source, int seuil)
   {
       return source.Where(n => _filtreSupérieur(n, seuil));
   }
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Méthodes anonymes | ✅ | ✅ |
| Expressions lambda de base | ✅ | ✅ |
| Inférence de type des lambdas | Limitée | Améliorée |
| Attributs pour les paramètres lambda | ❌ | ✅ |
| Lambdas avec modifiers (`static`, `async`) | ❌ | ✅ |
| Capture de struct | ❌ | ✅ (plus efficace) |
| Conversions de lambdas | De base | Améliorées |
| Performance | De base | Optimisée |

### Nouvelles fonctionnalités de lambdas dans .NET 8

.NET 8 offre plusieurs améliorations pour les expressions lambda qui ne sont pas disponibles dans .NET Framework 4.7.2 :

#### 1. Lambdas statiques

Les lambdas statiques empêchent la capture accidentelle de variables du contexte environnant :

```textmate
// Lambda statique - ne peut pas capturer de variables locales
Func<int, int, int> addition = static (a, b) => a + b;

// Erreur de compilation - ne peut pas accéder à la variable externe
int multiplicateur = 2;
Func<int, int> doubler = static x => x * multiplicateur; // Erreur
```


#### 2. Attributs sur les paramètres lambda

Vous pouvez appliquer des attributs aux paramètres des expressions lambda :

```textmate
// Attributs sur les paramètres lambda
Func<string, int> parseur = ([AllowNull] s) => int.TryParse(s, out var résultat) ? résultat : 0;

// Utile avec des attributs de validation
var validateur = ([Required][StringLength(50)] string nom) => !string.IsNullOrEmpty(nom);
```


#### 3. Lambdas naturellement asynchrones

Les lambdas peuvent être directement déclarées comme asynchrones :

```textmate
// Lambda async en .NET 8
Func<int, Task<int>> calculerAsync = async (n) =>
{
    await Task.Delay(100);  // Simulation d'une opération asynchrone
    return n * n;
};

// Utilisation
await calculerAsync(5);  // Retourne 25 après un délai
```


#### 4. Inférence de type améliorée

.NET 8 a amélioré l'inférence de type pour les lambdas :

```textmate
// En .NET 8, le type de la lambda est inféré automatiquement
var calculateur = (int x, int y) => x + y;  // Inféré comme Func<int, int, int>

// Pas besoin de déclarer explicitement le type de délégué
var estPair = x => x % 2 == 0;  // Inféré comme Func<int, bool>
```


#### 5. Discard patterns dans les lambdas

Utilisation de discards pour ignorer certains paramètres :

```textmate
// Ignorer certains paramètres avec _
var gestionnaire = (object _, EventArgs _) => Console.WriteLine("Événement déclenché");

// Dans un tuple
var extracteur = (string texte, (int id, _)) => $"{id}: {texte}";
```


#### 6. Meilleures performances avec lambdas

.NET 8 optimise les performances des lambdas :

```textmate
// Plus efficace en .NET 8
var liste = Enumerable.Range(1, 1000).ToList();

// Allocation réduite pour les lambdas simples
var carrés = liste.Select(x => x * x).ToList();

// Le compilateur peut générer du code optimisé pour les lambdas
var somme = liste.Sum(x => x);
```


## Conclusion sur les méthodes anonymes

Les méthodes anonymes et les expressions lambda sont des outils puissants dans l'écosystème C# qui permettent d'écrire du code plus concis et plus expressif. Bien que les expressions lambda aient largement remplacé les méthodes anonymes traditionnelles dans le code moderne, les deux approches restent valides et ont leur place dans certains contextes.

### Points clés à retenir

1. **Évolution historique** - Les méthodes anonymes sont apparues en C# 2.0, tandis que les lambdas ont été introduites en C# 3.0 comme une syntaxe plus concise et plus flexible.

2. **Choix de syntaxe** - Les expressions lambda sont généralement préférées pour leur concision et leur intégration avec les fonctionnalités modernes du langage.

3. **Cas d'utilisation** - Les méthodes anonymes et lambdas sont particulièrement utiles pour :
   - Les gestionnaires d'événements
   - Les callbacks et opérations asynchrones
   - LINQ et transformations de données
   - Stratégies et comportements configurables
   - Tris personnalisés

4. **Attention aux closures** - Soyez conscient des variables capturées et de leur cycle de vie, en particulier dans les boucles.

5. **Évolution vers .NET 8** - Les versions récentes de C# et .NET ont introduit de nombreuses améliorations aux expressions lambda, les rendant encore plus puissantes et flexibles.

Les méthodes anonymes et lambdas constituent un élément essentiel de la programmation fonctionnelle en C#, permettant d'écrire un code plus déclaratif et de passer des comportements comme des données. Que vous utilisiez .NET Framework 4.7.2 ou .NET 8, ces fonctionnalités vous aideront à écrire un code plus élégant et plus maintenable.

⏭️ 6.8. [Async/Await et programmation asynchrone](/06-fonctionnalites-avancees-de-csharp/6-8-async-await-et-programmation-asynchrone.md)
