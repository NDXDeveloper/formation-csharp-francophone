# 3.3. Collections spécialisées en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

En plus des collections standard que nous avons étudiées précédemment, C# offre plusieurs types de collections spécialisées conçues pour des scénarios particuliers. Ces collections ajoutent des fonctionnalités spécifiques pour répondre à des besoins comme la sécurité dans les environnements multi-threads, l'immuabilité des données, ou la notification des changements.

## 3.3.1. Collections concurrentes (ConcurrentDictionary, etc.)

Les collections concurrentes, situées dans l'espace de noms `System.Collections.Concurrent`, sont spécialement conçues pour être utilisées dans des environnements multi-threads. Elles permettent à plusieurs threads d'accéder et de modifier la collection simultanément de manière sécurisée, sans avoir besoin d'utiliser des verrous explicites.

Pour utiliser ces collections, vous devez ajouter:

```csharp
using System.Collections.Concurrent;
```

### Problèmes avec les collections standard en environnement multi-threads

Les collections standard ne sont pas thread-safe. Voici un exemple de problème qui peut survenir:

```csharp
// Collection non thread-safe
Dictionary<string, int> compteur = new Dictionary<string, int>();

// Simuler plusieurs threads essayant d'incrémenter le même compteur
Parallel.For(0, 1000, i =>
{
    string clé = "total";

    // Cette opération n'est pas atomique et peut causer des problèmes
    if (compteur.ContainsKey(clé))
    {
        compteur[clé]++;  // Risque de race condition
    }
    else
    {
        compteur[clé] = 1;  // Risque d'exception
    }
});

// Le résultat final peut être incorrect ou une exception peut être levée
```

### ConcurrentDictionary<TKey, TValue>

`ConcurrentDictionary<TKey, TValue>` est l'équivalent thread-safe de `Dictionary<TKey, TValue>`. Il permet des accès et des modifications concurrents sans risque de corruption des données.

```csharp
// Création d'un dictionnaire concurrent
ConcurrentDictionary<string, int> compteur = new ConcurrentDictionary<string, int>();

// Utilisation en environnement multi-threads
Parallel.For(0, 1000, i =>
{
    // Incrémenter de manière atomique
    compteur.AddOrUpdate(
        "total",           // clé
        1,                 // valeur à ajouter si la clé n'existe pas
        (clé, ancienneValeur) => ancienneValeur + 1  // fonction pour mettre à jour la valeur existante
    );
});

Console.WriteLine($"Compteur final: {compteur["total"]}");  // Devrait être 1000
```

#### Opérations atomiques courantes

`ConcurrentDictionary<TKey, TValue>` offre plusieurs méthodes atomiques qui garantissent la sécurité des opérations concurrentes:

```csharp
ConcurrentDictionary<string, int> stats = new ConcurrentDictionary<string, int>();

// Ajouter une nouvelle paire clé-valeur (ne fait rien si la clé existe déjà)
stats.TryAdd("visites", 1);

// Obtenir ou ajouter une valeur si la clé n'existe pas
int visites = stats.GetOrAdd("visites", 0);

// Mettre à jour une valeur existante, ou ajouter si elle n'existe pas
stats.AddOrUpdate("visites", 1, (clé, valeur) => valeur + 1);

// Essayer de mettre à jour une valeur existante
if (stats.TryGetValue("visites", out int valeurActuelle))
{
    // Tentative de mise à jour avec optimisme (peut échouer si modifiée par un autre thread)
    if (stats.TryUpdate("visites", valeurActuelle + 1, valeurActuelle))
    {
        Console.WriteLine("Mise à jour réussie");
    }
    else
    {
        Console.WriteLine("Mise à jour échouée (valeur modifiée entre-temps)");
    }
}
```

### ConcurrentQueue<T>

`ConcurrentQueue<T>` est une file d'attente thread-safe qui implémente le principe FIFO (Premier Entré, Premier Sorti).

```csharp
// Création d'une file d'attente concurrente
ConcurrentQueue<string> fileAttente = new ConcurrentQueue<string>();

// Producteur: ajouter des éléments à la file d'attente
Parallel.For(0, 100, i =>
{
    string message = $"Message {i}";
    fileAttente.Enqueue(message);
    Console.WriteLine($"Ajouté: {message}");
});

// Consommateur: traiter les éléments de la file d'attente
Parallel.For(0, 100, i =>
{
    if (fileAttente.TryDequeue(out string message))
    {
        Console.WriteLine($"Traité: {message}");
    }
    else
    {
        Console.WriteLine("File d'attente vide");
    }
});

// Vérifier si la file d'attente est vide
if (fileAttente.IsEmpty)
{
    Console.WriteLine("Tous les messages ont été traités");
}
else
{
    Console.WriteLine($"Il reste {fileAttente.Count} messages à traiter");
}
```

### ConcurrentStack<T>

`ConcurrentStack<T>` est une pile thread-safe qui implémente le principe LIFO (Dernier Entré, Premier Sorti).

```csharp
// Création d'une pile concurrente
ConcurrentStack<string> pile = new ConcurrentStack<string>();

// Ajouter des éléments à la pile
Parallel.For(0, 100, i =>
{
    string élément = $"Élément {i}";
    pile.Push(élément);
    Console.WriteLine($"Empilé: {élément}");
});

// Extraire et traiter des éléments de la pile
Parallel.For(0, 100, i =>
{
    if (pile.TryPop(out string élément))
    {
        Console.WriteLine($"Dépilé: {élément}");
    }
});

// Extraire plusieurs éléments à la fois
string[] éléments = new string[10];
int nbÉléments = pile.TryPopRange(éléments, 0, éléments.Length);
Console.WriteLine($"Dépilé {nbÉléments} éléments en une seule opération");
```

### ConcurrentBag<T>

`ConcurrentBag<T>` est une collection non ordonnée qui autorise les doublons. Elle est optimisée pour les scénarios où le même thread ajoute et retire des éléments.

```csharp
// Création d'un sac concurrent
ConcurrentBag<int> sac = new ConcurrentBag<int>();

// Ajouter des éléments
Parallel.For(0, 100, i =>
{
    sac.Add(i);
    sac.Add(i);  // Les doublons sont autorisés
});

// Afficher le nombre d'éléments
Console.WriteLine($"Le sac contient {sac.Count} éléments");  // Devrait être 200

// Extraire des éléments (l'ordre n'est pas garanti)
Parallel.For(0, 150, i =>
{
    if (sac.TryTake(out int élément))
    {
        Console.WriteLine($"Extrait: {élément}");
    }
    else
    {
        Console.WriteLine("Le sac est vide");
    }
});
```

### BlockingCollection<T>

`BlockingCollection<T>` implémente le modèle producteur-consommateur en permettant aux threads de se bloquer jusqu'à ce qu'une opération puisse être effectuée.

```csharp
// Création d'une collection bloquante limitée à 10 éléments
// (utilise ConcurrentQueue<T> par défaut comme stockage)
BlockingCollection<string> messages = new BlockingCollection<string>(boundedCapacity: 10);

// Thread producteur
Task producteur = Task.Run(() =>
{
    for (int i = 0; i < 20; i++)
    {
        string message = $"Message {i}";
        messages.Add(message);  // Bloque si la collection est pleine
        Console.WriteLine($"Produit: {message}");
        Thread.Sleep(100);  // Simuler un traitement
    }

    // Indiquer qu'il n'y aura plus d'ajouts
    messages.CompleteAdding();
});

// Thread consommateur
Task consommateur = Task.Run(() =>
{
    // Traiter les messages jusqu'à ce que la collection soit marquée comme complète
    // et que tous les éléments aient été traités
    foreach (string message in messages.GetConsumingEnumerable())
    {
        Console.WriteLine($"Consommé: {message}");
        Thread.Sleep(200);  // Simuler un traitement plus lent que le producteur
    }
});

// Attendre que les deux threads terminent
Task.WaitAll(producteur, consommateur);
```

### Exemple pratique: Cache concurrent

```csharp
public class CacheConcurrent<TClé, TValeur>
{
    private readonly ConcurrentDictionary<TClé, Tuple<TValeur, DateTime>> _cache =
        new ConcurrentDictionary<TClé, Tuple<TValeur, DateTime>>();

    private readonly TimeSpan _duréeExpiration;
    private readonly Func<TClé, TValeur> _fonctionChargement;

    public CacheConcurrent(TimeSpan duréeExpiration, Func<TClé, TValeur> fonctionChargement)
    {
        _duréeExpiration = duréeExpiration;
        _fonctionChargement = fonctionChargement;
    }

    public TValeur Obtenir(TClé clé)
    {
        // Vérifier si la clé existe et est valide
        if (_cache.TryGetValue(clé, out var tuple) &&
            DateTime.Now - tuple.Item2 < _duréeExpiration)
        {
            return tuple.Item1;  // Retourner la valeur du cache
        }

        // Sinon, charger la valeur et l'ajouter au cache
        TValeur nouvelleValeur = _fonctionChargement(clé);

        _cache.AddOrUpdate(
            clé,
            new Tuple<TValeur, DateTime>(nouvelleValeur, DateTime.Now),
            (k, ancien) => new Tuple<TValeur, DateTime>(nouvelleValeur, DateTime.Now)
        );

        return nouvelleValeur;
    }

    public void Invalider(TClé clé)
    {
        _cache.TryRemove(clé, out _);
    }

    public void Vider()
    {
        _cache.Clear();
    }

    public int NombreÉléments => _cache.Count;
}

// Utilisation du cache concurrent
static string RechercherDonnée(int id)
{
    Console.WriteLine($"Chargement des données pour l'ID {id}...");
    Thread.Sleep(1000);  // Simuler un accès à une base de données
    return $"Données pour ID {id}";
}

var cache = new CacheConcurrent<int, string>(
    TimeSpan.FromMinutes(10),
    RechercherDonnée
);

// Simuler plusieurs threads accédant au cache
Parallel.For(0, 5, i =>
{
    for (int j = 0; j < 3; j++)
    {
        int id = i;
        string données = cache.Obtenir(id);
        Console.WriteLine($"Thread {Task.CurrentId}: Obtenu {données}");
        Thread.Sleep(200);
    }
});

Console.WriteLine($"Nombre d'éléments dans le cache: {cache.NombreÉléments}");
```

### Quand utiliser des collections concurrentes

- Dans des applications multi-threads où plusieurs threads accèdent à la même collection
- Pour implémenter des modèles de type producteur-consommateur
- Pour éviter d'utiliser des verrous manuels qui peuvent causer des deadlocks
- Pour améliorer les performances en environnement hautement concurrent

## 3.3.2. Collections immuables (ImmutableList, etc.)

Les collections immuables, disponibles dans l'espace de noms `System.Collections.Immutable`, sont des collections dont le contenu ne peut pas être modifié après leur création. Toute opération qui semble modifier la collection renvoie en réalité une nouvelle collection, laissant l'originale inchangée.

Pour utiliser ces collections, vous devez installer le package NuGet `System.Collections.Immutable` et ajouter:

```csharp
using System.Collections.Immutable;
```

### Pourquoi utiliser des collections immuables?

Les collections immuables offrent plusieurs avantages:

1. **Sécurité des threads**: Puisqu'elles ne peuvent pas être modifiées, elles sont intrinsèquement thread-safe
2. **Cohérence des données**: Garantie qu'une collection ne changera pas pendant son utilisation
3. **Comportement prévisible**: Aucun effet secondaire lié à des modifications inattendues
4. **Simplification de la programmation fonctionnelle**: Parfaites pour les approches fonctionnelles

### ImmutableList<T>

`ImmutableList<T>` est l'équivalent immuable de `List<T>`. Une fois créée, elle ne peut pas être modifiée.

```csharp
// Création d'une liste immuable vide puis ajout d'éléments
ImmutableList<string> liste = ImmutableList<string>.Empty;
ImmutableList<string> listeAvecA = liste.Add("A");
ImmutableList<string> listeAvecAB = listeAvecA.Add("B");

// Création directe avec des valeurs initiales
ImmutableList<int> nombres = ImmutableList.Create(1, 2, 3, 4, 5);

// Création à partir d'une collection existante
List<double> valeurs = new List<double> { 1.1, 2.2, 3.3 };
ImmutableList<double> valeursImmuables = valeurs.ToImmutableList();

// La liste originale reste inchangée
Console.WriteLine($"Liste originale: {string.Join(", ", liste)}");  // Vide
Console.WriteLine($"Après ajout de A: {string.Join(", ", listeAvecA)}");  // A
Console.WriteLine($"Après ajout de B: {string.Join(", ", listeAvecAB)}");  // A, B
```

#### Opérations courantes sur ImmutableList<T>

```csharp
ImmutableList<int> nombres = ImmutableList.Create(1, 2, 3, 4, 5);

// Ajout d'éléments (retourne une nouvelle liste)
ImmutableList<int> plusNombres = nombres.Add(6);
plusNombres = plusNombres.AddRange(new[] { 7, 8, 9 });

// Insertion (retourne une nouvelle liste)
ImmutableList<int> avecInsertion = nombres.Insert(2, 99);  // Insère 99 à l'index 2

// Suppression (retourne une nouvelle liste)
ImmutableList<int> moinsDeux = nombres.Remove(2);  // Supprime la première occurrence de 2
ImmutableList<int> sansIndex = nombres.RemoveAt(1);  // Supprime l'élément à l'index 1

// Remplacement (retourne une nouvelle liste)
ImmutableList<int> avecRemplacement = nombres.SetItem(0, 100);  // Remplace l'élément à l'index 0 par 100

// Recherche
int index = nombres.IndexOf(3);
bool contient = nombres.Contains(4);

// Accès aux éléments (comme une liste normale)
int premier = nombres[0];
int dernier = nombres[nombres.Count - 1];

// Parcours
foreach (int n in nombres)
{
    Console.WriteLine(n);
}
```

### ImmutableDictionary<TKey, TValue>

`ImmutableDictionary<TKey, TValue>` est l'équivalent immuable de `Dictionary<TKey, TValue>`.

```csharp
// Création d'un dictionnaire immuable vide
ImmutableDictionary<string, int> vide = ImmutableDictionary<string, int>.Empty;

// Ajout de paires clé-valeur (retourne un nouveau dictionnaire)
ImmutableDictionary<string, int> âges = vide
    .Add("Alice", 30)
    .Add("Bob", 25)
    .Add("Charlie", 35);

// Création directe avec un constructeur
var capitales = ImmutableDictionary.CreateRange(new Dictionary<string, string>
{
    { "France", "Paris" },
    { "Allemagne", "Berlin" },
    { "Italie", "Rome" }
});

// Modification (chaque opération retourne un nouveau dictionnaire)
ImmutableDictionary<string, string> plusEspagne = capitales.Add("Espagne", "Madrid");
ImmutableDictionary<string, string> sansItalie = capitales.Remove("Italie");
ImmutableDictionary<string, string> françeNouvelleCapitale = capitales.SetItem("France", "Marseille");

// Accès
string capitaleAllemagne = capitales["Allemagne"];
bool contientPortugal = capitales.ContainsKey("Portugal");

if (capitales.TryGetValue("Italie", out string capitaleItalie))
{
    Console.WriteLine($"La capitale de l'Italie est {capitaleItalie}");
}
```

### ImmutableHashSet<T>

`ImmutableHashSet<T>` est l'équivalent immuable de `HashSet<T>`.

```csharp
// Création d'un ensemble immuable
ImmutableHashSet<int> ensemble = ImmutableHashSet.Create(1, 2, 3, 4, 5);

// Ajout et suppression (retournent de nouveaux ensembles)
ImmutableHashSet<int> plusSix = ensemble.Add(6);
ImmutableHashSet<int> moinsTrois = ensemble.Remove(3);

// L'ensemble original reste inchangé
Console.WriteLine($"Ensemble original: {string.Join(", ", ensemble)}");  // 1, 2, 3, 4, 5

// Opérations d'ensemble
ImmutableHashSet<int> ensemble2 = ImmutableHashSet.Create(4, 5, 6, 7, 8);

ImmutableHashSet<int> union = ensemble.Union(ensemble2);  // 1, 2, 3, 4, 5, 6, 7, 8
ImmutableHashSet<int> intersection = ensemble.Intersect(ensemble2);  // 4, 5
ImmutableHashSet<int> différence = ensemble.Except(ensemble2);  // 1, 2, 3
```

### ImmutableStack<T> et ImmutableQueue<T>

Versions immuables de `Stack<T>` et `Queue<T>`.

```csharp
// Pile immuable
ImmutableStack<string> pile = ImmutableStack<string>.Empty;
pile = pile.Push("Premier");
pile = pile.Push("Deuxième");
pile = pile.Push("Troisième");

string sommet;
ImmutableStack<string> pileSansSommet = pile.Pop(out sommet);
Console.WriteLine($"Élément retiré: {sommet}");  // Troisième
Console.WriteLine($"Pile originale: {string.Join(", ", pile)}");  // Troisième, Deuxième, Premier
Console.WriteLine($"Pile sans sommet: {string.Join(", ", pileSansSommet)}");  // Deuxième, Premier

// File immuable
ImmutableQueue<string> file = ImmutableQueue<string>.Empty;
file = file.Enqueue("Premier");
file = file.Enqueue("Deuxième");
file = file.Enqueue("Troisième");

string premier;
ImmutableQueue<string> fileSansPremier = file.Dequeue(out premier);
Console.WriteLine($"Élément retiré: {premier}");  // Premier
Console.WriteLine($"File originale: {string.Join(", ", file)}");  // Premier, Deuxième, Troisième
Console.WriteLine($"File sans le premier: {string.Join(", ", fileSansPremier)}");  // Deuxième, Troisième
```

### Constructeurs avec pattern builder

Pour des performances optimales lors de la création d'une collection immuable avec de nombreux éléments, utilisez les builders:

```csharp
// Utilisation d'un builder pour ImmutableList
var builder = ImmutableList.CreateBuilder<int>();
for (int i = 0; i < 1000; i++)
{
    builder.Add(i);
}
ImmutableList<int> liste = builder.ToImmutable();

// Utilisation d'un builder pour ImmutableDictionary
var dictBuilder = ImmutableDictionary.CreateBuilder<string, string>();
dictBuilder.Add("clé1", "valeur1");
dictBuilder.Add("clé2", "valeur2");
dictBuilder.Add("clé3", "valeur3");
ImmutableDictionary<string, string> dict = dictBuilder.ToImmutable();
```

### Exemple pratique: Historique des actions

```csharp
class Action
{
    public string Nom { get; }
    public DateTime Horodatage { get; }

    public Action(string nom)
    {
        Nom = nom;
        Horodatage = DateTime.Now;
    }

    public override string ToString()
    {
        return $"{Horodatage.ToShortTimeString()}: {Nom}";
    }
}

class HistoriqueActions
{
    private ImmutableList<Action> _historique = ImmutableList<Action>.Empty;

    public ImmutableList<Action> Historique => _historique;

    public void AjouterAction(string nomAction)
    {
        // Le changement de _historique est une opération atomique
        // qui ne modifie pas l'instance précédente
        _historique = _historique.Add(new Action(nomAction));
    }

    public void Annuler()
    {
        if (_historique.Count > 0)
        {
            _historique = _historique.RemoveAt(_historique.Count - 1);
        }
    }
}

// Utilisation
var historique = new HistoriqueActions();
historique.AjouterAction("Créer document");
historique.AjouterAction("Modifier texte");
historique.AjouterAction("Ajouter image");

Console.WriteLine("Historique complet:");
foreach (var action in historique.Historique)
{
    Console.WriteLine(action);
}

historique.Annuler();
Console.WriteLine("\nAprès annulation:");
foreach (var action in historique.Historique)
{
    Console.WriteLine(action);
}
```

### Quand utiliser des collections immuables

- Lorsque vous travaillez dans un environnement multi-threads sans avoir besoin de verrous
- Pour créer des API dont le comportement est prévisible et sans effets secondaires
- Dans une approche de programmation fonctionnelle
- Pour les données qui ne doivent jamais être modifiées après création
- Lorsque vous devez conserver l'historique des états d'une collection

## 3.3.3. Collections observables (ObservableCollection)

`ObservableCollection<T>` est une collection qui notifie les observateurs lorsque son contenu change. Elle est particulièrement utile dans les applications avec interfaces graphiques utilisant le pattern MVVM (Model-View-ViewModel).

Pour utiliser cette collection, vous devez ajouter:

```csharp
using System.Collections.ObjectModel;
using System.Collections.Specialized;
using System.ComponentModel;
```

### Création et utilisation de base

```csharp
// Création d'une collection observable
ObservableCollection<string> élèves = new ObservableCollection<string>();

// Ajouter un gestionnaire d'événements pour les changements
élèves.CollectionChanged += (sender, e) =>
{
    if (e.Action == NotifyCollectionChangedAction.Add)
    {
        foreach (string nouvelÉlève in e.NewItems)
        {
            Console.WriteLine($"Nouvel élève ajouté: {nouvelÉlève}");
        }
    }
    else if (e.Action == NotifyCollectionChangedAction.Remove)
    {
        foreach (string élèveSupprimé in e.OldItems)
        {
            Console.WriteLine($"Élève supprimé: {élèveSupprimé}");
        }
    }
    else if (e.Action == NotifyCollectionChangedAction.Replace)
    {
        Console.WriteLine("Un élève a été remplacé");
    }
    else if (e.Action == NotifyCollectionChangedAction.Move)
    {
        Console.WriteLine("Un élève a été déplacé");
    }
    else if (e.Action == NotifyCollectionChangedAction.Reset)
    {
        Console.WriteLine("La liste d'élèves a été réinitialisée");
    }
};

// Ajouter des éléments (déclenche l'événement CollectionChanged)
élèves.Add("Alice");
élèves.Add("Bob");
élèves.Add("Charlie");

// Supprimer un élément (déclenche l'événement CollectionChanged)
élèves.Remove("Bob");

// Modifier un élément (déclenche l'événement CollectionChanged)
int indexCharlie = élèves.IndexOf("Charlie");
élèves[indexCharlie] = "Charles";

// Vider la collection (déclenche l'événement CollectionChanged avec Reset)
élèves.Clear();
```

### Créer une classe observable

Pour créer une collection d'objets qui notifient également de leurs changements internes, les objets doivent implémenter l'interface `INotifyPropertyChanged`:

```csharp
// Classe d'élève observable
public class Élève : INotifyPropertyChanged
{
    private string _nom;
    private int _âge;

    public event PropertyChangedEventHandler PropertyChanged;

    // Méthode helper pour déclencher l'événement PropertyChanged
    protected void NotifierChangement([System.Runtime.CompilerServices.CallerMemberName] string nomPropriété = "")
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nomPropriété));
    }

    public string Nom
    {
        get => _nom;
        set
        {
            if (_nom != value)
            {
                _nom = value;
                NotifierChangement();
            }
        }
    }

    public int Âge
    {
        get => _âge;
        set
        {
            if (_âge != value)
            {
                _âge = value;
                NotifierChangement();
            }
        }
    }

    public Élève(string nom, int âge)
    {
        _nom = nom;
        _âge = âge;
    }

    public override string ToString()
    {
        return $"{Nom} ({Âge} ans)";
    }
}

// Utilisation avec ObservableCollection
ObservableCollection<Élève> classeÉlèves = new ObservableCollection<Élève>();

// Observer les changements de la collection
classeÉlèves.CollectionChanged += (sender, e) =>
{
    Console.WriteLine($"La collection a changé: {e.Action}");
};

// Ajouter des élèves
classeÉlèves.Add(new Élève("Alice", 15));
classeÉlèves.Add(new Élève("Bob", 16));

// Observer les changements de propriété pour chaque élève
Élève charlie = new Élève("Charlie", 14);
charlie.PropertyChanged += (sender, e) =>
{
    Console.WriteLine($"La propriété {e.PropertyName} de {((Élève)sender).Nom} a changé");
};
classeÉlèves.Add(charlie);

// Modifier une propriété d'un élève (déclenche l'événement PropertyChanged)
charlie.Âge = 15;
```

### Exemple pratique: Application de liste de tâches

```csharp
public class Tâche : INotifyPropertyChanged
{
    private string _titre;
    private bool _estTerminée;
    private DateTime _dateÉchéance;

    public event PropertyChangedEventHandler PropertyChanged;

    protected void NotifierChangement([System.Runtime.CompilerServices.CallerMemberName] string nomPropriété = "")
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nomPropriété));
    }

    public string Titre
    {
        get => _titre;
        set
        {
            if (_titre != value)
            {
                _titre = value;
                NotifierChangement();
            }
        }
    }

    public bool EstTerminée
    {
        get => _estTerminée;
        set
        {
            if (_estTerminée != value)
            {
                _estTerminée = value;
                NotifierChangement();
                // Si le statut change, le compteur de tâches doit être mis à jour
                NotifierChangement(nameof(Description));
            }
        }
    }

    public DateTime DateÉchéance
    {
        get => _dateÉchéance;
        set
        {
            if (_dateÉchéance != value)
            {
                _dateÉchéance = value;
                NotifierChangement();
                NotifierChangement(nameof(Description));
            }
        }
    }

    public string Description => $"{Titre} - {(EstTerminée ? "Terminée" : "En cours")} - Échéance: {DateÉchéance.ToShortDateString()}";

    public Tâche(string titre, DateTime dateÉchéance)
    {
        _titre = titre;
        _dateÉchéance = dateÉchéance;
        _estTerminée = false;
    }
}

public class GestionnaireTâches
{
    private ObservableCollection<Tâche> _tâches;

    public ObservableCollection<Tâche> Tâches => _tâches;

    public int NombreTâchesEnCours => _tâches.Count(t => !t.EstTerminée);

    public int NombreTâchesTerminées => _tâches.Count(t => t.EstTerminée);

    public GestionnaireTâches()
    {
        _tâches = new ObservableCollection<Tâche>();

        // S'abonner aux changements de la collection
        _tâches.CollectionChanged += (sender, e) =>
        {
            // Quand des tâches sont ajoutées
            if (e.NewItems != null)
            {
                foreach (Tâche tâche in e.NewItems)
                {
                    tâche.PropertyChanged += Tâche_PropertyChanged;
                }
            }

            // Quand des tâches sont supprimées
            if (e.OldItems != null)
            {
                foreach (Tâche tâche in e.OldItems)
                {
                    tâche.PropertyChanged -= Tâche_PropertyChanged;
                }
            }

            AfficherStatistiques();
        };
    }

    private void Tâche_PropertyChanged(object sender, PropertyChangedEventArgs e)
    {
        if (e.PropertyName == nameof(Tâche.EstTerminée))
        {
            AfficherStatistiques();
        }
    }

    public void AjouterTâche(string titre, DateTime dateÉchéance)
    {
        _tâches.Add(new Tâche(titre, dateÉchéance));
    }

    public void SupprimerTâche(Tâche tâche)
    {
        _tâches.Remove(tâche);
    }

    public void MarquerCommeTerminée(Tâche tâche)
    {
        tâche.EstTerminée = true;
    }

    public void AfficherTâches()
    {
        Console.WriteLine("\nListe des tâches:");
        foreach (var tâche in _tâches)
        {
            Console.WriteLine($"- {tâche.Description}");
        }
    }

    public void AfficherStatistiques()
    {
        Console.WriteLine($"\nStatistiques: {NombreTâchesTerminées} tâche(s) terminée(s), {NombreTâchesEnCours} tâche(s) en cours");
    }
}

// Utilisation du gestionnaire de tâches
static void Main(string[] args)
{
    var gestionnaire = new GestionnaireTâches();

    gestionnaire.AjouterTâche("Préparer présentation", DateTime.Now.AddDays(3));
    gestionnaire.AjouterTâche("Envoyer rapport", DateTime.Now.AddDays(1));
    gestionnaire.AjouterTâche("Répondre aux emails", DateTime.Now);

    gestionnaire.AfficherTâches();

    // Marquer une tâche comme terminée
    Tâche tâcheEmail = gestionnaire.Tâches.FirstOrDefault(t => t.Titre.Contains("email"));
    if (tâcheEmail != null)
    {
        gestionnaire.MarquerCommeTerminée(tâcheEmail);
    }

    gestionnaire.AfficherTâches();

    // Modifier une propriété
    Tâche tâcheRapport = gestionnaire.Tâches.FirstOrDefault(t => t.Titre.Contains("rapport"));
    if (tâcheRapport != null)
    {
        tâcheRapport.DateÉchéance = DateTime.Now.AddDays(2);
        Console.WriteLine($"Date d'échéance du rapport modifiée: {tâcheRapport.Description}");
    }

    // Supprimer une tâche
    Tâche tâcheASupprimer = gestionnaire.Tâches.FirstOrDefault(t => t.Titre.Contains("présentation"));
    if (tâcheASupprimer != null)
    {
        gestionnaire.SupprimerTâche(tâcheASupprimer);
        Console.WriteLine("Tâche 'Préparer présentation' supprimée");
    }

    gestionnaire.AfficherTâches();
}
```

### Utilisation d'ObservableCollection dans les applications WPF

Dans les applications WPF (Windows Presentation Foundation), `ObservableCollection<T>` est particulièrement utile pour lier des collections de données à des contrôles d'interface utilisateur. Voici un exemple simple en XAML (langage de balisage pour WPF) :

```xaml
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Gestionnaire de tâches" Height="350" Width="500">
    <Grid>
        <ListBox ItemsSource="{Binding Tâches}" Margin="10">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <CheckBox IsChecked="{Binding EstTerminée}" VerticalAlignment="Center"/>
                        <TextBlock Text="{Binding Titre}" Margin="5,0,0,0"
                                  TextDecorations="{Binding EstTerminée, Converter={StaticResource BoolToStrikethrough}}"/>
                        <TextBlock Text="{Binding DateÉchéance, StringFormat=d}" Margin="10,0,0,0"
                                  Foreground="Gray"/>
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>
    </Grid>
</Window>
```

Le code-behind correspondant :

```csharp
public partial class MainWindow : Window
{
    private GestionnaireTâches _gestionnaire;

    public MainWindow()
    {
        InitializeComponent();

        _gestionnaire = new GestionnaireTâches();
        _gestionnaire.AjouterTâche("Tâche 1", DateTime.Now.AddDays(1));
        _gestionnaire.AjouterTâche("Tâche 2", DateTime.Now.AddDays(2));

        // Définir le DataContext pour le binding
        DataContext = _gestionnaire;
    }
}
```

### Quand utiliser ObservableCollection<T>

- Dans les applications avec interface graphique utilisant le pattern MVVM
- Lorsque vous avez besoin d'être notifié des changements dans une collection
- Pour l'affichage automatique des changements de données à l'interface utilisateur
- Pour implémenter des fonctionnalités réactives où une partie du code doit réagir aux changements d'une autre partie

## 3.3.4. ReadOnlyCollection et autres wrappers en lecture seule

Les collections en lecture seule permettent d'exposer vos collections tout en empêchant leur modification. Contrairement aux collections immuables, les collections en lecture seule sont des wrappers autour de collections modifiables sous-jacentes.

La principale différence avec les collections immuables est que si la collection sous-jacente change, ces changements seront visibles à travers la vue en lecture seule.

Pour utiliser ces collections, vous devez ajouter :

```csharp
using System.Collections.ObjectModel;
```

### ReadOnlyCollection<T>

`ReadOnlyCollection<T>` est une vue en lecture seule d'une collection sous-jacente. Elle est utile pour exposer une collection à l'extérieur d'une classe tout en empêchant sa modification directe.

```csharp
// Liste sous-jacente modifiable
List<string> fruits = new List<string> { "Pomme", "Banane", "Orange" };

// Créer une vue en lecture seule
ReadOnlyCollection<string> fruitsEnLectureSeule = new ReadOnlyCollection<string>(fruits);

// Tentative de modification (lève une exception)
try
{
    // fruitsEnLectureSeule.Add("Fraise");  // Erreur de compilation: 'Add' n'existe pas
    // fruitsEnLectureSeule[0] = "Abricot";  // Erreur de compilation: impossible d'assigner à un indexeur
}
catch (NotSupportedException ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}

// Parcourir la collection en lecture seule
foreach (string fruit in fruitsEnLectureSeule)
{
    Console.WriteLine(fruit);
}

// Modifier la collection sous-jacente
fruits.Add("Fraise");
fruits[0] = "Abricot";

// Les modifications sont visibles à travers la vue en lecture seule
Console.WriteLine("\nAprès modification de la collection sous-jacente:");
foreach (string fruit in fruitsEnLectureSeule)
{
    Console.WriteLine(fruit);  // Affiche: Abricot, Banane, Orange, Fraise
}
```

### ReadOnlyDictionary<TKey, TValue>

`ReadOnlyDictionary<TKey, TValue>` est la version en lecture seule d'un dictionnaire.

```csharp
// Dictionnaire sous-jacent modifiable
Dictionary<string, int> âges = new Dictionary<string, int>
{
    { "Alice", 30 },
    { "Bob", 25 },
    { "Charlie", 35 }
};

// Créer une vue en lecture seule
ReadOnlyDictionary<string, int> âgesEnLectureSeule = new ReadOnlyDictionary<string, int>(âges);

// Accès aux éléments (autorisé)
int âgeAlice = âgesEnLectureSeule["Alice"];
Console.WriteLine($"Âge d'Alice: {âgeAlice}");

bool contientDavid = âgesEnLectureSeule.ContainsKey("David");
Console.WriteLine($"Contient David: {contientDavid}");

// Tentative de modification (lève une exception)
try
{
    // âgesEnLectureSeule.Add("David", 28);  // Erreur de compilation
    // âgesEnLectureSeule["Alice"] = 31;     // Erreur de compilation
}
catch (NotSupportedException ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}

// Modifier le dictionnaire sous-jacent
âges.Add("David", 28);
âges["Alice"] = 31;

// Les modifications sont visibles à travers la vue en lecture seule
Console.WriteLine("\nAprès modification du dictionnaire sous-jacent:");
foreach (var paire in âgesEnLectureSeule)
{
    Console.WriteLine($"{paire.Key}: {paire.Value}");
}
```

### AsReadOnly sur List<T>

La méthode `AsReadOnly()` de `List<T>` permet de créer facilement une `ReadOnlyCollection<T>`.

```csharp
List<double> prix = new List<double> { 9.99, 14.50, 19.99 };

// Créer une vue en lecture seule
ReadOnlyCollection<double> prixEnLectureSeule = prix.AsReadOnly();

// Utilisation de la collection en lecture seule
Console.WriteLine($"Nombre d'articles: {prixEnLectureSeule.Count}");
Console.WriteLine($"Prix du premier article: {prixEnLectureSeule[0]}");

// Modifier la liste sous-jacente
prix.Add(24.99);
prix[0] = 8.99;

// Les modifications sont visibles
Console.WriteLine("\nAprès modification:");
for (int i = 0; i < prixEnLectureSeule.Count; i++)
{
    Console.WriteLine($"Article {i+1}: {prixEnLectureSeule[i]}");
}
```

### IReadOnlyList<T> et IReadOnlyDictionary<TKey, TValue>

Ces interfaces, introduites dans .NET 4.5, permettent de déclarer des collections en lecture seule:

```csharp
// Utilisation avec List<T>
List<int> nombres = new List<int> { 1, 2, 3, 4, 5 };
IReadOnlyList<int> nombresEnLectureSeule = nombres;

// Utilisation avec Dictionary<TKey, TValue>
Dictionary<string, bool> statuts = new Dictionary<string, bool>
{
    { "Serveur1", true },
    { "Serveur2", false },
    { "Serveur3", true }
};
IReadOnlyDictionary<string, bool> statutsEnLectureSeule = statuts;

// Accès aux éléments
int troisième = nombresEnLectureSeule[2];  // 3
bool statutServeur2 = statutsEnLectureSeule["Serveur2"];  // false

// Parcours
foreach (int nombre in nombresEnLectureSeule)
{
    Console.WriteLine(nombre);
}

foreach (var paire in statutsEnLectureSeule)
{
    Console.WriteLine($"{paire.Key}: {paire.Value}");
}
```

### Exemple pratique: Catalogue de produits

```csharp
public class Produit
{
    public int Id { get; }
    public string Nom { get; }
    public decimal Prix { get; }
    public string Catégorie { get; }

    public Produit(int id, string nom, decimal prix, string catégorie)
    {
        Id = id;
        Nom = nom;
        Prix = prix;
        Catégorie = catégorie;
    }

    public override string ToString()
    {
        return $"{Nom} ({Catégorie}) - {Prix:C}";
    }
}

public class CatalogueProduits
{
    private List<Produit> _produits = new List<Produit>();
    private ReadOnlyCollection<Produit> _produitsEnLectureSeule;

    public CatalogueProduits()
    {
        _produitsEnLectureSeule = new ReadOnlyCollection<Produit>(_produits);
    }

    // Propriété qui expose les produits en lecture seule
    public ReadOnlyCollection<Produit> Produits => _produitsEnLectureSeule;

    // Méthodes pour modifier la collection en interne
    public void AjouterProduit(Produit produit)
    {
        _produits.Add(produit);
    }

    public bool SupprimerProduit(int id)
    {
        int index = _produits.FindIndex(p => p.Id == id);
        if (index >= 0)
        {
            _produits.RemoveAt(index);
            return true;
        }
        return false;
    }

    // Méthodes de recherche
    public IEnumerable<Produit> RechercherParCatégorie(string catégorie)
    {
        return _produits.Where(p => p.Catégorie.Equals(catégorie, StringComparison.OrdinalIgnoreCase));
    }

    public Produit RechercherParId(int id)
    {
        return _produits.FirstOrDefault(p => p.Id == id);
    }
}

// Utilisation du catalogue
static void Main(string[] args)
{
    var catalogue = new CatalogueProduits();

    // Ajouter des produits
    catalogue.AjouterProduit(new Produit(1, "Smartphone XYZ", 599.99m, "Électronique"));
    catalogue.AjouterProduit(new Produit(2, "Laptop ABC", 899.99m, "Électronique"));
    catalogue.AjouterProduit(new Produit(3, "Casque audio", 129.99m, "Accessoires"));
    catalogue.AjouterProduit(new Produit(4, "Clavier mécanique", 89.99m, "Accessoires"));

    // Accéder à la collection en lecture seule
    Console.WriteLine("Catalogue complet:");
    foreach (var produit in catalogue.Produits)
    {
        Console.WriteLine(produit);
    }

    // Rechercher par catégorie
    Console.WriteLine("\nProduits électroniques:");
    foreach (var produit in catalogue.RechercherParCatégorie("Électronique"))
    {
        Console.WriteLine(produit);
    }

    // Rechercher par ID
    Produit produit3 = catalogue.RechercherParId(3);
    if (produit3 != null)
    {
        Console.WriteLine($"\nProduit trouvé: {produit3}");
    }

    // Supprimer un produit
    bool supprimé = catalogue.SupprimerProduit(2);
    Console.WriteLine($"\nProduit 2 supprimé: {supprimé}");

    // Vérifier que la suppression est reflétée dans la collection en lecture seule
    Console.WriteLine("\nCatalogue après suppression:");
    foreach (var produit in catalogue.Produits)
    {
        Console.WriteLine(produit);
    }

    // Tentative de modification directe de la collection en lecture seule (ne compilera pas)
    // catalogue.Produits.Add(new Produit(5, "Tablette", 299.99m, "Électronique"));
}
```

### Un exemple de classe de collection personnalisée avec vue en lecture seule

```csharp
public class FileAttentePrioritaire<T>
{
    private List<T> _éléments = new List<T>();
    private IComparer<T> _comparateur;
    private ReadOnlyCollection<T> _vueEnLectureSeule;

    public FileAttentePrioritaire() : this(Comparer<T>.Default) { }

    public FileAttentePrioritaire(IComparer<T> comparateur)
    {
        _comparateur = comparateur;
        _vueEnLectureSeule = new ReadOnlyCollection<T>(_éléments);
    }

    // Exposer les éléments en lecture seule
    public ReadOnlyCollection<T> Éléments => _vueEnLectureSeule;

    public int Count => _éléments.Count;

    public void Ajouter(T élément)
    {
        // Insérer à la position correcte pour maintenir le tri
        int i = 0;
        while (i < _éléments.Count && _comparateur.Compare(_éléments[i], élément) <= 0)
        {
            i++;
        }

        _éléments.Insert(i, élément);
    }

    public T Retirer()
    {
        if (_éléments.Count == 0)
        {
            throw new InvalidOperationException("La file d'attente est vide");
        }

        T élément = _éléments[0];
        _éléments.RemoveAt(0);
        return élément;
    }

    public T Consulter()
    {
        if (_éléments.Count == 0)
        {
            throw new InvalidOperationException("La file d'attente est vide");
        }

        return _éléments[0];
    }

    public void Vider()
    {
        _éléments.Clear();
    }
}

// Utilisation de la file d'attente prioritaire
class Patient : IComparable<Patient>
{
    public string Nom { get; }
    public int NiveauUrgence { get; }  // 1 = urgent, 5 = non urgent

    public Patient(string nom, int niveauUrgence)
    {
        Nom = nom;
        NiveauUrgence = niveauUrgence;
    }

    public int CompareTo(Patient autre)
    {
        // Tri par niveau d'urgence (croissant = plus urgent d'abord)
        return NiveauUrgence.CompareTo(autre.NiveauUrgence);
    }

    public override string ToString()
    {
        return $"{Nom} (Niveau {NiveauUrgence})";
    }
}

// Utilisation
var fileAttente = new FileAttentePrioritaire<Patient>();

fileAttente.Ajouter(new Patient("Alice", 3));
fileAttente.Ajouter(new Patient("Bob", 1));  // Urgent
fileAttente.Ajouter(new Patient("Charlie", 4));
fileAttente.Ajouter(new Patient("David", 2));

Console.WriteLine("Liste d'attente actuelle:");
foreach (var patient in fileAttente.Éléments)
{
    Console.WriteLine(patient);
}

// Traiter le patient le plus urgent
Patient prochainePatient = fileAttente.Retirer();
Console.WriteLine($"\nPatient en cours de traitement: {prochainePatient}");

Console.WriteLine("\nListe d'attente mise à jour:");
foreach (var patient in fileAttente.Éléments)
{
    Console.WriteLine(patient);
}
```

### Quand utiliser des collections en lecture seule

- Pour exposer des collections internes tout en empêchant leur modification par des clients externes
- Lorsque vous souhaitez une vue en lecture seule d'une collection qui reflète les modifications de la collection sous-jacente
- Pour implémenter le principe d'encapsulation en permettant l'accès en lecture seule aux données
- Dans les API où les utilisateurs doivent pouvoir parcourir des données mais ne doivent pas les modifier

### Comparaison entre collections en lecture seule et collections immuables

| Caractéristique | Collections en lecture seule | Collections immuables |
|-----------------|------------------------------|------------------------|
| **Modification de la collection** | Interdite via l'interface en lecture seule | Impossible (crée une nouvelle collection) |
| **Modification de la collection sous-jacente** | Possible et visible via la vue en lecture seule | Pas de collection sous-jacente |
| **Performance de modification** | N/A (non autorisée) | Coût supplémentaire pour créer une nouvelle collection |
| **Utilisation mémoire** | Légère (référence à la collection existante) | Peut être importante (chaque modification crée une nouvelle collection) |
| **Thread safety** | Non garantie (dépend de la collection sous-jacente) | Garantie (les instances sont immuables) |
| **Cas d'utilisation typique** | Exposer des données en lecture seule tout en permettant leur modification interne | Garantir l'immutabilité complète, programmation fonctionnelle |

## Résumé des collections spécialisées

Voici un tableau récapitulatif des collections spécialisées présentées dans ce chapitre:

| Collection | Espace de noms | Description | Cas d'utilisation |
|------------|----------------|-------------|------------------|
| **ConcurrentDictionary** | System.Collections.Concurrent | Dictionnaire thread-safe | Accès concurrent par plusieurs threads |
| **ConcurrentQueue** | System.Collections.Concurrent | File d'attente thread-safe | Producteur-consommateur multi-threads |
| **ConcurrentStack** | System.Collections.Concurrent | Pile thread-safe | Accès LIFO concurrent |
| **ConcurrentBag** | System.Collections.Concurrent | Collection non ordonnée thread-safe | Stockage d'éléments sans ordre spécifique en multi-threads |
| **BlockingCollection** | System.Collections.Concurrent | Collection bloquante avec limite de capacité | Scénarios producteur-consommateur avec blocage |
| **ImmutableList** | System.Collections.Immutable | Liste qui ne peut jamais être modifiée | Programmation fonctionnelle, thread safety garantie |
| **ImmutableDictionary** | System.Collections.Immutable | Dictionnaire immuable | Collections partagées qui ne doivent jamais changer |
| **ImmutableHashSet** | System.Collections.Immutable | Ensemble immuable | Opérations d'ensemble immuables |
| **ObservableCollection** | System.Collections.ObjectModel | Collection qui notifie des changements | Applications UI, binding de données, MVVM |
| **ReadOnlyCollection** | System.Collections.ObjectModel | Vue non modifiable d'une collection | Exposer des données en lecture seule |
| **ReadOnlyDictionary** | System.Collections.ObjectModel | Dictionnaire en lecture seule | API publiques qui doivent prévenir les modifications |

### Conseils pratiques

1. **Choisissez la collection spécialisée adaptée à votre scénario**
   - Environnement multi-threads → Collections concurrentes
   - Programmation fonctionnelle → Collections immuables
   - Interface utilisateur avec data binding → ObservableCollection
   - Encapsulation → Collections en lecture seule

2. **Attention aux performances**
   - Les collections concurrentes ont un coût en termes de performances par rapport à leurs équivalents non thread-safe
   - Les collections immuables créent de nouvelles instances à chaque modification
   - Utilisez les builders pour créer des collections immuables avec de nombreux éléments initiaux

3. **Considérez la sérialisation**
   - Certaines collections spécialisées peuvent ne pas se sérialiser correctement avec tous les formats
   - Testez la sérialisation si votre application l'utilise

4. **Exposition d'API**
   - Utilisez `IReadOnlyList<T>` et `IReadOnlyDictionary<TKey, TValue>` dans les signatures de méthodes publiques pour exprimer clairement l'intention
   - Exposez les collections internes en tant que collections en lecture seule pour maintenir l'encapsulation

5. **Gestion de la mémoire**
   - Soyez attentif aux références retenues lors de l'utilisation de collections immuables, particulièrement dans les applications à longue durée de vie
   - Les collections concurrentes peuvent retenir plus de mémoire que nécessaire pour optimiser la concurrence

En maîtrisant ces collections spécialisées, vous pourrez choisir la structure de données la plus adaptée à chaque situation spécifique, améliorant ainsi la robustesse et la performance de vos applications C#.

⏭️ 4. [Méthodes et fonctions](/04-methodes-et-fonctions/README.md)
