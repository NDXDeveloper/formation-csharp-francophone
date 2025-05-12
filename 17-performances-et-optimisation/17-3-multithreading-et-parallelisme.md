# 17.3. Multithreading et parallélisme en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Dans le monde moderne du développement logiciel, la capacité à exécuter plusieurs opérations simultanément est essentielle pour créer des applications performantes. Les ordinateurs modernes disposent généralement de plusieurs cœurs de processeur, et C# offre divers outils pour tirer parti de cette puissance de calcul. Ce chapitre explore les concepts et techniques de programmation parallèle en C#, en commençant par les bases et en progressant vers des modèles plus avancés.

## 17.3.1. Thread vs Task

La programmation parallèle en C# a considérablement évolué au fil du temps. Comprendre la différence entre les threads et les tâches est fondamental pour exploiter efficacement le parallélisme.

### Qu'est-ce qu'un Thread ?

Un **thread** (fil d'exécution) est un flux d'exécution indépendant dans un programme. Le système d'exploitation attribue du temps processeur à chaque thread pour qu'il puisse s'exécuter. Les threads représentent le concept de base du parallélisme.

#### Caractéristiques des Threads :
- Relativement lourds en termes de ressources système
- Créés et gérés directement par le système d'exploitation
- Permettent un contrôle de bas niveau
- Disponibles depuis les premières versions de .NET

#### Création et utilisation de base d'un Thread :

```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        Console.WriteLine($"Thread principal : {Thread.CurrentThread.ManagedThreadId}");

        // Création d'un nouveau thread
        Thread thread = new Thread(TravailSecondaire);

        // Démarrage du thread
        thread.Start();

        // Travail dans le thread principal
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Thread principal travaille : {i}");
            Thread.Sleep(100);  // Simule du travail
        }

        // Attente que le thread secondaire termine
        thread.Join();

        Console.WriteLine("Programme terminé");
    }

    static void TravailSecondaire()
    {
        Console.WriteLine($"Thread secondaire : {Thread.CurrentThread.ManagedThreadId}");

        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Thread secondaire travaille : {i}");
            Thread.Sleep(150);  // Simule du travail
        }
    }
}
```

#### Passage de paramètres à un Thread :

```csharp
// Création d'un thread avec un paramètre
Thread thread = new Thread(param => TravailAvecParametre((int)param));
thread.Start(42);

// Méthode exécutée par le thread
static void TravailAvecParametre(int valeur)
{
    Console.WriteLine($"Travail avec paramètre : {valeur}");
}
```

### Qu'est-ce qu'une Task ?

Une **Task** (tâche) est une abstraction de plus haut niveau représentant une opération asynchrone. Introduite avec .NET Framework 4.0, la classe `Task` fait partie de la Task Parallel Library (TPL) et offre une approche plus moderne et flexible pour la programmation parallèle.

#### Caractéristiques des Tasks :
- Plus légères que les threads
- Gérées par le ThreadPool pour une utilisation efficace des ressources
- Peuvent retourner des résultats (via `Task<T>`)
- Supportent nativement les opérations de composition (continuation, attente, etc.)
- Intégration parfaite avec les mots-clés `async` et `await`

#### Création et utilisation de base d'une Task :

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine($"Thread principal : {Thread.CurrentThread.ManagedThreadId}");

        // Création et démarrage d'une tâche
        Task task = Task.Run(() => TravailSecondaire());

        // Travail dans le thread principal
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Thread principal travaille : {i}");
            await Task.Delay(100);  // Version moderne de Thread.Sleep
        }

        // Attente que la tâche se termine
        await task;

        Console.WriteLine("Programme terminé");
    }

    static void TravailSecondaire()
    {
        Console.WriteLine($"Task s'exécute sur le thread : {Thread.CurrentThread.ManagedThreadId}");

        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Task travaille : {i}");
            Thread.Sleep(150);
        }
    }
}
```

#### Tâches avec valeur de retour :

```csharp
// Création d'une tâche qui retourne une valeur
Task<int> calculTask = Task.Run(() => CalculComplexe());

// Await peut être utilisé pour attendre le résultat
int résultat = await calculTask;
Console.WriteLine($"Résultat : {résultat}");

// Méthode exécutée par la tâche
static int CalculComplexe()
{
    Console.WriteLine("Exécution du calcul complexe...");
    Thread.Sleep(2000);  // Simule un long calcul
    return 42;
}
```

### Comparaison Thread vs Task

| Aspect | Thread | Task |
|--------|--------|------|
| Niveau d'abstraction | Bas niveau | Haut niveau |
| Gestion des ressources | Manuelle | Automatique via ThreadPool |
| Création | Plus coûteuse | Plus légère |
| Valeur de retour | Non | Oui (avec Task<T>) |
| Annulation | Complexe | Simple avec CancellationToken |
| Composition | Difficile | Facile avec await/ContinueWith |
| Gestion des exceptions | Complexe | Intégrée |
| Idéal pour | Opérations de longue durée<br>Contrôle précis des threads | La plupart des scénarios<br>Programmation asynchrone |

### Quand utiliser Thread vs Task ?

**Utilisez Thread quand :**
- Vous avez besoin d'un contrôle précis sur les priorités des threads
- Vous créez un thread à longue durée de vie (comme un thread de surveillance)
- Vous avez besoin de configurer des aspects spécifiques du thread (comme pour un thread d'interface utilisateur)

**Utilisez Task quand :**
- Vous voulez exploiter le ThreadPool pour une meilleure efficacité
- Vous avez besoin de valeurs de retour ou de gérer des exceptions
- Vous souhaitez composer des opérations asynchrones
- Vous utilisez async/await (presque toujours le meilleur choix dans le code moderne)

### Exemple pratique : calcul parallèle

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        const int taille = 10_000_000;
        double[] données = new double[taille];
        Random random = new Random();

        // Initialisation du tableau
        for (int i = 0; i < taille; i++)
        {
            données[i] = random.NextDouble();
        }

        Console.WriteLine("Comparaison des performances:");

        // Approche séquentielle
        Stopwatch sw = Stopwatch.StartNew();
        double sommeSeq = SommeSequentielle(données);
        sw.Stop();
        Console.WriteLine($"Séquentiel: {sw.ElapsedMilliseconds}ms, Résultat = {sommeSeq}");

        // Approche avec Thread
        sw.Restart();
        double sommeThread = SommeAvecThreads(données);
        sw.Stop();
        Console.WriteLine($"Threads: {sw.ElapsedMilliseconds}ms, Résultat = {sommeThread}");

        // Approche avec Task
        sw.Restart();
        double sommeTask = await SommeAvecTasks(données);
        sw.Stop();
        Console.WriteLine($"Tasks: {sw.ElapsedMilliseconds}ms, Résultat = {sommeTask}");

        // Approche avec Parallel
        sw.Restart();
        double sommeParallel = SommeAvecParallel(données);
        sw.Stop();
        Console.WriteLine($"Parallel: {sw.ElapsedMilliseconds}ms, Résultat = {sommeParallel}");
    }

    // Méthode séquentielle standard
    static double SommeSequentielle(double[] données)
    {
        double somme = 0;
        for (int i = 0; i < données.Length; i++)
        {
            somme += Math.Sqrt(données[i]);
        }
        return somme;
    }

    // Utilisation de threads manuels
    static double SommeAvecThreads(double[] données)
    {
        int cœurs = Environment.ProcessorCount;
        int taillePortion = données.Length / cœurs;
        double[] résultatsPartiels = new double[cœurs];
        Thread[] threads = new Thread[cœurs];

        for (int i = 0; i < cœurs; i++)
        {
            int indexThread = i;
            threads[i] = new Thread(() =>
            {
                int début = indexThread * taillePortion;
                int fin = (indexThread == cœurs - 1) ? données.Length : début + taillePortion;

                for (int j = début; j < fin; j++)
                {
                    résultatsPartiels[indexThread] += Math.Sqrt(données[j]);
                }
            });

            threads[i].Start();
        }

        // Attendre que tous les threads terminent
        foreach (var thread in threads)
        {
            thread.Join();
        }

        // Combiner les résultats
        double sommeFinale = 0;
        foreach (var résultat in résultatsPartiels)
        {
            sommeFinale += résultat;
        }

        return sommeFinale;
    }

    // Utilisation de Tasks
    static async Task<double> SommeAvecTasks(double[] données)
    {
        int cœurs = Environment.ProcessorCount;
        int taillePortion = données.Length / cœurs;
        Task<double>[] tâches = new Task<double>[cœurs];

        for (int i = 0; i < cœurs; i++)
        {
            int début = i * taillePortion;
            int fin = (i == cœurs - 1) ? données.Length : début + taillePortion;

            tâches[i] = Task.Run(() =>
            {
                double sommePartielle = 0;
                for (int j = début; j < fin; j++)
                {
                    sommePartielle += Math.Sqrt(données[j]);
                }
                return sommePartielle;
            });
        }

        // Attendre que toutes les tâches terminent et additionner les résultats
        double[] résultats = await Task.WhenAll(tâches);
        return résultats.Sum();
    }

    // Utilisation de Parallel.For
    static double SommeAvecParallel(double[] données)
    {
        double somme = 0;
        object verrou = new object();

        Parallel.For(0, données.Length, i =>
        {
            double valeurLocale = Math.Sqrt(données[i]);

            // Sécuriser l'accès à la variable partagée
            lock (verrou)
            {
                somme += valeurLocale;
            }
        });

        return somme;
    }
}
```

> **Note** : Dans cet exemple, `SommeAvecParallel` utilise un verrou, ce qui peut limiter les performances. Une meilleure approche serait d'utiliser `Parallel.ForEach` avec une option de réduction locale, comme nous le verrons dans la section suivante.

## 17.3.2. Parallel LINQ (PLINQ)

PLINQ (Parallel LINQ) est une implémentation parallèle de LINQ qui décompose automatiquement les opérations de requête pour tirer parti de plusieurs processeurs. C'est l'un des moyens les plus simples d'ajouter du parallélisme à votre code, particulièrement pour les opérations de traitement de données.

### Principes de base de PLINQ

Pour utiliser PLINQ, il suffit généralement d'ajouter la méthode d'extension `.AsParallel()` à une séquence LINQ. Le framework s'occupe automatiquement de diviser le travail et de gérer les threads.

```csharp
using System;
using System.Linq;

// LINQ standard (séquentiel)
var résultatSéquentiel = from n in collection
                         where EstPremier(n)
                         select n;

// PLINQ (parallèle)
var résultatParallèle = from n in collection.AsParallel()
                        where EstPremier(n)
                        select n;
```

### Exemple simple de PLINQ

```csharp
using System;
using System.Diagnostics;
using System.Linq;

class Program
{
    static void Main()
    {
        // Créer une grande collection pour démonstration
        int[] nombres = Enumerable.Range(1, 10_000_000).ToArray();

        // Mesurer le temps pour la version séquentielle
        Stopwatch sw = Stopwatch.StartNew();
        var nombresPremiersSéquentiels = nombres.Where(n => EstPremier(n)).ToList();
        sw.Stop();
        Console.WriteLine($"Version séquentielle: {sw.ElapsedMilliseconds}ms, " +
                         $"Nombre trouvé: {nombresPremiersSéquentiels.Count}");

        // Mesurer le temps pour la version parallèle
        sw.Restart();
        var nombresPremiersPlinq = nombres.AsParallel().Where(n => EstPremier(n)).ToList();
        sw.Stop();
        Console.WriteLine($"Version PLINQ: {sw.ElapsedMilliseconds}ms, " +
                         $"Nombre trouvé: {nombresPremiersPlinq.Count}");
    }

    static bool EstPremier(int n)
    {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0 || n % 3 == 0) return false;

        int i = 5;
        while (i * i <= n)
        {
            if (n % i == 0 || n % (i + 2) == 0)
                return false;
            i += 6;
        }
        return true;
    }
}
```

### Options et paramètres de PLINQ

PLINQ offre plusieurs méthodes pour configurer son comportement :

#### 1. Préserver l'ordre des résultats

Par défaut, PLINQ ne garantit pas que les résultats seront dans le même ordre que la séquence d'entrée, car cela pourrait limiter les performances.

```csharp
// Préserver l'ordre
var résultatsOrdonnés = nombres.AsParallel()
                              .AsOrdered()  // Maintient l'ordre original
                              .Where(n => EstPremier(n));
```

#### 2. ForAll pour un traitement parallèle sans construction de résultat

Quand vous avez juste besoin d'exécuter une action sur chaque élément en parallèle :

```csharp
// Traiter tous les éléments en parallèle sans créer de collection de résultats
nombres.AsParallel()
       .Where(n => EstPremier(n))
       .ForAll(nombre => Console.WriteLine($"Nombre premier trouvé: {nombre}"));
```

#### 3. Contrôler le degré de parallélisme

Vous pouvez spécifier combien de tâches parallèles PLINQ doit utiliser :

```csharp
// Limiter à 4 tâches parallèles
var résultat = nombres.AsParallel()
                     .WithDegreeOfParallelism(4)  // Utiliser 4 threads max
                     .Where(n => EstPremier(n));
```

#### 4. Gestion des exceptions

PLINQ collecte les exceptions de toutes les tâches parallèles et les encapsule dans une `AggregateException` :

```csharp
try
{
    var résultat = nombres.AsParallel()
                         .Where(n =>
                         {
                             if (n == 42) throw new Exception("Nombre 42 détecté!");
                             return EstPremier(n);
                         })
                         .ToList();
}
catch (AggregateException ex)
{
    foreach (var innerEx in ex.InnerExceptions)
    {
        Console.WriteLine($"Exception: {innerEx.Message}");
    }
}
```

### PLINQ vs. Parallel.ForEach

PLINQ et la classe Parallel offrent des approches différentes pour le parallélisme :

```csharp
// PLINQ
var résultatPlinq = nombres.AsParallel()
                          .Where(n => EstPremier(n))
                          .ToList();

// Parallel.ForEach avec une liste pour collecter les résultats
var résultatsParallel = new ConcurrentBag<int>();
Parallel.ForEach(nombres, nombre =>
{
    if (EstPremier(nombre))
        résultatsParallel.Add(nombre);
});
```

#### Quand utiliser PLINQ vs. Parallel ?

- **Utilisez PLINQ quand :**
  - Vous travaillez déjà avec LINQ
  - Vous avez besoin d'expressions de requête complexes
  - Vous voulez une solution concise et déclarative

- **Utilisez Parallel.ForEach quand :**
  - Vous avez besoin d'un contrôle plus précis
  - Vous effectuez des opérations plus complexes que de simples filtres ou projections
  - Vous n'avez pas besoin d'une séquence résultante

### Exemple avancé : agrégation parallèle avec PLINQ

```csharp
using System;
using System.Collections.Concurrent;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // Analyse de texte parallèle - compter les occurrences de mots
        string texte = "Ceci est un exemple de texte. " +
                      "Ce texte sera analysé pour compter les occurrences de chaque mot. " +
                      "L'analyse sera effectuée en parallèle pour montrer l'efficacité de PLINQ.";

        // Séparation en mots
        string[] mots = texte.Split(new[] { ' ', '.', ',', '!', '?', ';', ':', '-', '\n', '\r' },
                                   StringSplitOptions.RemoveEmptyEntries);

        Console.WriteLine($"Analyse de {mots.Length} mots.");

        // Version séquentielle LINQ
        Stopwatch sw = Stopwatch.StartNew();
        var fréquencesSeq = mots.GroupBy(mot => mot.ToLower())
                              .Select(groupe => new
                              {
                                  Mot = groupe.Key,
                                  Compte = groupe.Count()
                              })
                              .OrderByDescending(x => x.Compte)
                              .ToList();
        sw.Stop();
        Console.WriteLine($"LINQ séquentiel: {sw.ElapsedMilliseconds}ms");

        // Version PLINQ
        sw.Restart();
        var fréquencesPar = mots.AsParallel()
                              .GroupBy(mot => mot.ToLower())
                              .Select(groupe => new
                              {
                                  Mot = groupe.Key,
                                  Compte = groupe.Count()
                              })
                              .OrderByDescending(x => x.Compte)
                              .ToList();
        sw.Stop();
        Console.WriteLine($"PLINQ: {sw.ElapsedMilliseconds}ms");

        // Afficher les résultats
        Console.WriteLine("\nRésultats:");
        foreach (var item in fréquencesPar.Take(5))
        {
            Console.WriteLine($"'{item.Mot}': {item.Compte} occurrences");
        }
    }
}
```

> **Note** : Dans cet exemple avec un petit ensemble de données, PLINQ pourrait être plus lent que LINQ séquentiel en raison des frais généraux de parallélisation. PLINQ montre ses avantages avec de grandes collections et des opérations coûteuses.

## 17.3.3. Synchronisation

Lorsque plusieurs threads accèdent aux mêmes données, des problèmes de synchronisation peuvent survenir. C# offre plusieurs mécanismes pour coordonner l'accès aux ressources partagées.

### Le problème de la synchronisation

Sans synchronisation appropriée, les threads qui accèdent simultanément aux mêmes données peuvent provoquer des résultats imprévisibles ou incorrects.

```csharp
// Exemple de code non sécurisé pour les threads
class CompteurNonSécurisé
{
    private int _compteur = 0;

    public void Incrémenter()
    {
        _compteur++;  // Cette opération n'est pas atomique!
    }

    public int Valeur => _compteur;
}

// Utilisation problématique
var compteur = new CompteurNonSécurisé();
Parallel.For(0, 100000, _ => compteur.Incrémenter());
Console.WriteLine($"Valeur finale: {compteur.Valeur}");  // Probablement moins que 100000
```

### Mécanismes de synchronisation en C#

#### 1. Le mot-clé `lock`

L'instruction `lock` est le mécanisme de synchronisation le plus courant en C#. Elle garantit qu'un seul thread à la fois peut exécuter le bloc de code protégé.

```csharp
class CompteurSécurisé
{
    private int _compteur = 0;
    private readonly object _verrou = new object();

    public void Incrémenter()
    {
        lock (_verrou)  // Un seul thread peut entrer ici à la fois
        {
            _compteur++;
        }
    }

    public int Valeur
    {
        get
        {
            lock (_verrou)
            {
                return _compteur;
            }
        }
    }
}
```

**Bonnes pratiques pour `lock` :**
- Utilisez un objet privé dédié comme verrou, jamais `this` ou un type
- Gardez les blocs `lock` aussi courts que possible
- Évitez d'appeler du code externe dans un bloc `lock`
- Toujours acquérir les verrous dans le même ordre pour éviter les deadlocks

#### 2. Les classes de synchronisation du .NET Framework

C# offre plusieurs classes spécialisées pour la synchronisation :

**a) Monitor**

`Monitor` est le mécanisme sous-jacent de `lock`, mais offre plus de contrôle :

```csharp
private readonly object _verrou = new object();

public void MéthodeThreadSafe()
{
    bool verrouAcquis = false;
    try
    {
        Monitor.Enter(_verrou, ref verrouAcquis);
        // Code critique ici
    }
    finally
    {
        if (verrouAcquis)
            Monitor.Exit(_verrou);
    }
}
```

**b) Mutex**

`Mutex` peut être utilisé pour la synchronisation entre processus :

```csharp
private Mutex _mutex = new Mutex(false, "MonApplicationUnique");

public void MéthodeThreadSafe()
{
    try
    {
        _mutex.WaitOne();  // Attendre l'acquisition du mutex
        // Code critique ici
    }
    finally
    {
        _mutex.ReleaseMutex();
    }
}
```

**c) Semaphore et SemaphoreSlim**

Permettent à un nombre limité de threads d'accéder à une ressource :

```csharp
// Permettre à 3 threads d'accéder simultanément
private SemaphoreSlim _semaphore = new SemaphoreSlim(3);

public async Task MéthodeAvecAccèsLimité()
{
    await _semaphore.WaitAsync();
    try
    {
        // Jusqu'à 3 threads peuvent exécuter ce code en même temps
        await Task.Delay(1000);  // Simulation de travail
    }
    finally
    {
        _semaphore.Release();
    }
}
```

**d) ReaderWriterLockSlim**

Optimisé pour les scénarios avec beaucoup de lectures et peu d'écritures :

```csharp
private ReaderWriterLockSlim _verrou = new ReaderWriterLockSlim();
private Dictionary<int, string> _données = new Dictionary<int, string>();

public string Lire(int clé)
{
    _verrou.EnterReadLock();
    try
    {
        return _données.TryGetValue(clé, out string valeur) ? valeur : null;
    }
    finally
    {
        _verrou.ExitReadLock();
    }
}

public void Écrire(int clé, string valeur)
{
    _verrou.EnterWriteLock();
    try
    {
        _données[clé] = valeur;
    }
    finally
    {
        _verrou.ExitWriteLock();
    }
}
```

#### 3. Collections thread-safe

Le namespace `System.Collections.Concurrent` fournit des collections optimisées pour l'accès concurrent :

```csharp
// Liste thread-safe
ConcurrentBag<int> liste = new ConcurrentBag<int>();
Parallel.For(0, 1000, i => liste.Add(i));

// Dictionnaire thread-safe
ConcurrentDictionary<string, int> dictionnaire = new ConcurrentDictionary<string, int>();
Parallel.ForEach(new[] { "un", "deux", "trois" }, mot =>
{
    dictionnaire.AddOrUpdate(mot, 1, (clé, valeurActuelle) => valeurActuelle + 1);
});

// File d'attente thread-safe
ConcurrentQueue<string> file = new ConcurrentQueue<string>();
file.Enqueue("premier");
string résultat;
if (file.TryDequeue(out résultat))
{
    Console.WriteLine(résultat);
}
```

#### 4. SpinLock et SpinWait

Pour des verrous de très courte durée sur des systèmes multi-cœurs :

```csharp
private SpinLock _spinLock = new SpinLock();

public void MéthodeVérrouillée()
{
    bool verrouAcquis = false;
    try
    {
        _spinLock.Enter(ref verrouAcquis);
        // Code critique TRÈS court ici
    }
    finally
    {
        if (verrouAcquis)
            _spinLock.Exit();
    }
}
```

#### 5. AsyncLock pour le code asynchrone

Pour la synchronisation dans du code asynchrone (non inclus dans .NET, mais facilement implémentable) :

```csharp
public class AsyncLock
{
    private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1);

    public async Task<IDisposable> LockAsync()
    {
        await _semaphore.WaitAsync();
        return new Releaser(_semaphore);
    }

    private class Releaser : IDisposable
    {
        private readonly SemaphoreSlim _semaphore;

        public Releaser(SemaphoreSlim semaphore)
        {
            _semaphore = semaphore;
        }

        public void Dispose()
        {
            _semaphore.Release();
        }
    }
}

// Utilisation
private readonly AsyncLock _verrou = new AsyncLock();

public async Task MéthodeAsynchrone()
{
    using (await _verrou.LockAsync())
    {
        // Code critique avec await
        await Task.Delay(100);
    }
}
```

### Exemple pratique : cache thread-safe

Voici un exemple de mise en œuvre d'un cache thread-safe qui permet plusieurs lectures simultanées mais verrouille pour les écritures :

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

public class CacheThreadSafe<TClé, TValeur>
{
    private readonly Dictionary<TClé, TValeur> _cache = new Dictionary<TClé, TValeur>();
    private readonly ReaderWriterLockSlim _verrou = new ReaderWriterLockSlim();

    public TValeur GetOrAdd(TClé clé, Func<TClé, TValeur> valueFactory)
    {
        // Essayer d'abord en mode lecture
        _verrou.EnterReadLock();
        try
        {
            if (_cache.TryGetValue(clé, out TValeur valeurExistante))
            {
                return valeurExistante;
            }
        }
        finally
        {
            _verrou.ExitReadLock();
        }

        // Pas trouvé, acquérir un verrou d'écriture
        _verrou.EnterWriteLock();
        try
        {
            // Vérifier à nouveau en cas de mise à jour entre les verrous
            if (_cache.TryGetValue(clé, out TValeur valeurExistante))
            {
                return valeurExistante;
            }

            // Créer et stocker la nouvelle valeur
            TValeur nouvelleValeur = valueFactory(clé);
            _cache[clé] = nouvelleValeur;
            return nouvelleValeur;
        }
        finally
        {
            _verrou.ExitWriteLock();
        }
    }

    public bool TryGetValue(TClé clé, out TValeur valeur)
    {
        _verrou.EnterReadLock();
        try
        {
            return _cache.TryGetValue(clé, out valeur);
        }
        finally
        {
            _verrou.ExitReadLock();
        }
    }

    public void Clear()
    {
        _verrou.EnterWriteLock();
        try
        {
            _cache.Clear();
        }
        finally
        {
            _verrou.ExitWriteLock();
        }
    }

    public void Dispose()
    {
        _verrou.Dispose();
    }
}
```

Utilisation du cache:

```csharp
class Program
{
    static CacheThreadSafe<string, string> _cache = new CacheThreadSafe<string, string>();

    static async Task Main()
    {
        // Simuler plusieurs threads accédant au cache
        var tâches = new List<Task>();
        for (int i = 0; i < 100; i++)
        {
            int iteration = i;
            tâches.Add(Task.Run(() => UtiliserCache($"clé{iteration % 10}")));
        }

        await Task.WhenAll(tâches);
        Console.WriteLine("Toutes les tâches sont terminées");
    }

    static void UtiliserCache(string clé)
    {
        string valeur = _cache.GetOrAdd(clé, k =>
        {
            // Simuler une opération coûteuse
            Thread.Sleep(100);
            return $"Valeur pour {k} créée par le thread {Thread.CurrentThread.ManagedThreadId}";
        });

        Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} a obtenu: {valeur}");
    }
}
```

Ce cache utilise `ReaderWriterLockSlim` pour permettre un accès concurrent en lecture tout en garantissant un accès exclusif en écriture, ce qui est idéal pour les scénarios où les lectures sont beaucoup plus fréquentes que les écritures.

## 17.3.4. Problèmes courants (deadlocks, race conditions)

Lors de l'utilisation du multithreading, plusieurs types de problèmes peuvent survenir. Comprendre ces problèmes et savoir comment les éviter est essentiel pour développer des applications robustes.

### Race conditions (conditions de course)

Une condition de course se produit lorsque le comportement d'un programme dépend de l'ordre ou du timing relatif d'événements incontrôlables.

#### Exemple de race condition :

```csharp
class CompteBancaire
{
    private decimal _solde;

    public CompteBancaire(decimal soldeInitial)
    {
        _solde = soldeInitial;
    }

    // Méthode non sécurisée - susceptible aux conditions de course
    public void Déposer(decimal montant)
    {
        // Cette opération n'est pas atomique!
        // Un autre thread pourrait intervenir entre la lecture et l'écriture
        _solde = _solde + montant;
    }

    // Méthode non sécurisée - susceptible aux conditions de course
    public bool Retirer(decimal montant)
    {
        if (_solde >= montant)
        {
            // Un autre thread pourrait réduire le solde ici avant que
            // nous ne terminions l'opération
            _solde = _solde - montant;
            return true;
        }
        return false;
    }

    public decimal Solde => _solde;
}
```

Dans cet exemple, si deux threads appellent `Retirer` ou `Déposer` simultanément, le résultat final peut être incorrect car les opérations de lecture et d'écriture du solde ne sont pas atomiques.

#### Solution : utiliser la synchronisation

```csharp
class CompteBancaireSécurisé
{
    private decimal _solde;
    private readonly object _verrou = new object();

    public CompteBancaireSécurisé(decimal soldeInitial)
    {
        _solde = soldeInitial;
    }

    public void Déposer(decimal montant)
    {
        lock (_verrou)
        {
            _solde = _solde + montant;
        }
    }

    public bool Retirer(decimal montant)
    {
        lock (_verrou)
        {
            if (_solde >= montant)
            {
                _solde = _solde - montant;
                return true;
            }
            return false;
        }
    }

    public decimal Solde
    {
        get
        {
            lock (_verrou)
            {
                return _solde;
            }
        }
    }
}
```

### Deadlocks (interblocages)

Un deadlock se produit lorsque deux threads (ou plus) s'attendent mutuellement pour libérer des ressources, ce qui conduit à une situation où aucun ne peut progresser.

#### Exemple de deadlock :

```csharp
class ExempleDeadlock
{
    private readonly object _verrou1 = new object();
    private readonly object _verrou2 = new object();

    public void Méthode1()
    {
        lock (_verrou1)
        {
            Console.WriteLine("Méthode1: verrou1 acquis");

            // Simuler du travail
            Thread.Sleep(100);

            // Tenter d'acquérir le deuxième verrou
            lock (_verrou2)
            {
                Console.WriteLine("Méthode1: verrou2 acquis");
            }
        }
    }

    public void Méthode2()
    {
        lock (_verrou2)
        {
            Console.WriteLine("Méthode2: verrou2 acquis");

            // Simuler du travail
            Thread.Sleep(100);

            // Tenter d'acquérir le premier verrou
            lock (_verrou1)
            {
                Console.WriteLine("Méthode2: verrou1 acquis");
            }
        }
    }
}

// Utilisation qui peut causer un deadlock
var exemple = new ExempleDeadlock();

// Exécuter Méthode1 dans un thread
Thread thread1 = new Thread(() => exemple.Méthode1());
thread1.Start();

// Exécuter Méthode2 dans un autre thread
Thread thread2 = new Thread(() => exemple.Méthode2());
thread2.Start();

// Attendre que les threads se terminent
thread1.Join();
thread2.Join();
```

Dans cet exemple, si `thread1` acquiert `_verrou1` et `thread2` acquiert `_verrou2` en même temps, un deadlock se produit lorsque chaque thread tente d'acquérir le verrou que l'autre détient déjà.

#### Comment éviter les deadlocks :

1. **Acquérir les verrous toujours dans le même ordre**

```csharp
class ExempleSansDeadlock
{
    private readonly object _verrou1 = new object();
    private readonly object _verrou2 = new object();

    public void Méthode1()
    {
        lock (_verrou1)
        {
            Console.WriteLine("Méthode1: verrou1 acquis");

            // Simuler du travail
            Thread.Sleep(100);

            // Acquérir les verrous dans le même ordre que Méthode2
            lock (_verrou2)
            {
                Console.WriteLine("Méthode1: verrou2 acquis");
            }
        }
    }

    public void Méthode2()
    {
        lock (_verrou1)  // Acquérir _verrou1 d'abord, comme dans Méthode1
        {
            Console.WriteLine("Méthode2: verrou1 acquis");

            // Simuler du travail
            Thread.Sleep(100);

            lock (_verrou2)
            {
                Console.WriteLine("Méthode2: verrou2 acquis");
            }
        }
    }
}
```

2. **Utiliser `Monitor.TryEnter` avec timeout**

```csharp
public void MéthodeAvecTimeout()
{
    bool verrou1Acquis = false;
    bool verrou2Acquis = false;

    try
    {
        verrou1Acquis = Monitor.TryEnter(_verrou1, TimeSpan.FromSeconds(1));
        if (verrou1Acquis)
        {
            // Simuler du travail
            Thread.Sleep(100);

            verrou2Acquis = Monitor.TryEnter(_verrou2, TimeSpan.FromSeconds(1));
            if (verrou2Acquis)
            {
                // Travail avec les deux verrous
                Console.WriteLine("Les deux verrous acquis");
            }
            else
            {
                Console.WriteLine("Impossible d'acquérir verrou2, abandon");
            }
        }
        else
        {
            Console.WriteLine("Impossible d'acquérir verrou1, abandon");
        }
    }
    finally
    {
        if (verrou2Acquis)
            Monitor.Exit(_verrou2);

        if (verrou1Acquis)
            Monitor.Exit(_verrou1);
    }
}
```

3. **Éviter d'utiliser des verrous imbriqués si possible**

Concevez votre code pour minimiser le besoin de verrous imbriqués.

4. **Utilisez des classes de synchronisation de plus haut niveau**

`SemaphoreSlim`, `ReaderWriterLockSlim` et autres classes offrent des fonctionnalités qui peuvent aider à éviter les deadlocks.

### Livelocks

Un livelock est similaire à un deadlock, mais les threads changent continuellement d'état les uns par rapport aux autres, sans progresser.

#### Exemple de livelock :

```csharp
class ExempleLivelock
{
    private bool _thread1Actif = true;
    private bool _thread2Actif = true;

    public void Thread1Action()
    {
        while (_thread1Actif)
        {
            // Vérifier si l'autre thread est actif
            if (_thread2Actif)
            {
                Console.WriteLine("Thread 1: Thread 2 est actif, je me mets en attente");
                Thread.Sleep(100);
                continue;
            }

            // Faire le travail
            Console.WriteLine("Thread 1: Je fais mon travail");
            _thread1Actif = false;
        }
    }

    public void Thread2Action()
    {
        while (_thread2Actif)
        {
            // Vérifier si l'autre thread est actif
            if (_thread1Actif)
            {
                Console.WriteLine("Thread 2: Thread 1 est actif, je me mets en attente");
                Thread.Sleep(100);
                continue;
            }

            // Faire le travail
            Console.WriteLine("Thread 2: Je fais mon travail");
            _thread2Actif = false;
        }
    }
}
```

Dans cet exemple, les deux threads sont trop polis : chacun vérifie si l'autre est actif et, si c'est le cas, attend. Comme les deux sont initialement actifs, ils continuent à s'attendre mutuellement sans progresser.

### Starvation (famine)

La starvation se produit lorsqu'un thread ne peut pas accéder aux ressources dont il a besoin pour progresser, généralement parce que d'autres threads monopolisent ces ressources.

#### Comment éviter la starvation :

1. **Utiliser des priorités de thread avec prudence**

```csharp
// Évitez d'abuser des priorités de thread
thread.Priority = ThreadPriority.BelowNormal;
```

2. **Utiliser des verrous équitables**

```csharp
// ReaderWriterLockSlim avec l'option d'équité
var verrou = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);
```

3. **Limiter le temps pendant lequel un thread détient un verrou**

```csharp
lock (verrou)
{
    // Faire le minimum nécessaire sous le verrou
    // Éviter les opérations longues ou bloquantes ici
}
```

### Problèmes subtils : visibilité et réordonnancement

Le compilateur et le processeur peuvent réorganiser les instructions pour optimiser les performances. Cela peut causer des problèmes dans le code multithread.

#### Exemple de problème de visibilité :

```csharp
class ExempleVisibilité
{
    private bool _drapeau;
    private int _valeur;

    public void Thread1()
    {
        _valeur = 42;
        _drapeau = true;  // Indique que _valeur est prête
    }

    public void Thread2()
    {
        if (_drapeau)  // Vérifie si _valeur est prête
        {
            Console.WriteLine(_valeur);  // Pourrait ne pas afficher 42!
        }
    }
}
```

Dans cet exemple, sans barrières de mémoire, `Thread2` pourrait voir `_drapeau` à `true` mais toujours lire l'ancienne valeur de `_valeur`.

#### Solutions :

1. **Utiliser des variables `volatile`**

```csharp
private volatile bool _drapeau;
```

2. **Utiliser `Thread.MemoryBarrier`**

```csharp
_valeur = 42;
Thread.MemoryBarrier();  // Garantit que les écritures précédentes sont visibles
_drapeau = true;
```

3. **Utiliser des primitives de synchronisation comme `lock`**

```csharp
private readonly object _verrou = new object();

public void Thread1()
{
    lock (_verrou)
    {
        _valeur = 42;
        _drapeau = true;
    }
}

public void Thread2()
{
    lock (_verrou)
    {
        if (_drapeau)
        {
            Console.WriteLine(_valeur);  // Garantie d'afficher 42
        }
    }
}
```

## 17.3.5. Atomic operations et interlocked

Pour certaines opérations simples, il existe des alternatives plus légères et plus efficaces que le verrouillage complet avec `lock`.

### Opérations atomiques avec Interlocked

La classe `Interlocked` fournit des méthodes pour effectuer des opérations atomiques sur des variables partagées.

#### Incrémentation et décrémentation atomiques

```csharp
class CompteurAtomique
{
    private int _compteur;

    public void Incrémenter()
    {
        Interlocked.Increment(ref _compteur);
    }

    public void Décrémenter()
    {
        Interlocked.Decrement(ref _compteur);
    }

    public int Valeur => _compteur;
}
```

#### Échange atomique de valeurs

```csharp
private int _état = 0;

public void DéfinirÉtatSiInactif(int nouvelÉtat)
{
    // Change _état en nouvelÉtat seulement si _état est 0
    // Retourne la valeur précédente
    int ancienneValeur = Interlocked.CompareExchange(ref _état, nouvelÉtat, 0);

    if (ancienneValeur == 0)
    {
        Console.WriteLine($"État changé de 0 à {nouvelÉtat}");
    }
    else
    {
        Console.WriteLine($"État inchangé, valeur actuelle: {ancienneValeur}");
    }
}
```

#### Addition atomique

```csharp
private long _total = 0;

public void Ajouter(int valeur)
{
    Interlocked.Add(ref _total, valeur);
}
```

#### Lecture atomique de valeurs 64 bits

Sur certaines architectures, la lecture/écriture de valeurs 64 bits (long, double) n'est pas atomique par défaut.

```csharp
private long _valeur64Bits = 0;

public long LireValeur()
{
    // Lecture atomique garantie même sur du 32 bits
    return Interlocked.Read(ref _valeur64Bits);
}
```

### Exemple complet : compteur de performances atomique

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

public class CompteurPerformances
{
    private long _compteur;
    private long _tempsTotal;  // En ticks

    public void Mesurer(Action action)
    {
        Stopwatch sw = Stopwatch.StartNew();
        try
        {
            action();
        }
        finally
        {
            sw.Stop();
            Interlocked.Increment(ref _compteur);
            Interlocked.Add(ref _tempsTotal, sw.ElapsedTicks);
        }
    }

    public long NombreAppels => Interlocked.Read(ref _compteur);

    public TimeSpan TempsMoyen
    {
        get
        {
            long appels = Interlocked.Read(ref _compteur);
            long tempsTotal = Interlocked.Read(ref _tempsTotal);

            if (appels > 0)
            {
                long ticksMoyens = tempsTotal / appels;
                return TimeSpan.FromTicks(ticksMoyens);
            }

            return TimeSpan.Zero;
        }
    }

    public TimeSpan TempsTotal => TimeSpan.FromTicks(Interlocked.Read(ref _tempsTotal));
}
```

Utilisation:

```csharp
static async Task Main()
{
    var compteur = new CompteurPerformances();

    // Exécuter plusieurs tâches qui utilisent le compteur
    var tâches = new Task[100];
    for (int i = 0; i < tâches.Length; i++)
    {
        tâches[i] = Task.Run(() =>
        {
            compteur.Mesurer(() =>
            {
                // Simulation de travail
                Thread.Sleep(new Random().Next(10, 50));
            });
        });
    }

    await Task.WhenAll(tâches);

    Console.WriteLine($"Nombre d'appels: {compteur.NombreAppels}");
    Console.WriteLine($"Temps total: {compteur.TempsTotal.TotalMilliseconds:F2} ms");
    Console.WriteLine($"Temps moyen: {compteur.TempsMoyen.TotalMilliseconds:F2} ms");
}
```

### Lazy initialization (initialisation paresseuse)

Une autre utilisation commune des opérations atomiques est l'initialisation paresseuse thread-safe.

#### Pattern Double-Check Locking

```csharp
public class ServiceCouteux
{
    private static volatile ServiceCouteux _instance;
    private static readonly object _verrou = new object();

    private ServiceCouteux()
    {
        // Initialisation coûteuse
        Console.WriteLine("Initialisation coûteuse du service");
        Thread.Sleep(1000);
    }

    public static ServiceCouteux Instance
    {
        get
        {
            if (_instance == null)  // Premier check (non verrouillé)
            {
                lock (_verrou)
                {
                    if (_instance == null)  // Deuxième check (verrouillé)
                    {
                        _instance = new ServiceCouteux();
                    }
                }
            }

            return _instance;
        }
    }

    public void FaireQuelqueChose()
    {
        Console.WriteLine("Le service fait quelque chose...");
    }
}
```

#### Utilisation de Lazy<T>

.NET fournit la classe `Lazy<T>` qui simplifie l'initialisation paresseuse thread-safe :

```csharp
public class ServiceCouteux
{
    private static readonly Lazy<ServiceCouteux> _lazyInstance =
        new Lazy<ServiceCouteux>(() => new ServiceCouteux(), isThreadSafe: true);

    private ServiceCouteux()
    {
        // Initialisation coûteuse
        Console.WriteLine("Initialisation coûteuse du service");
        Thread.Sleep(1000);
    }

    public static ServiceCouteux Instance => _lazyInstance.Value;

    public void FaireQuelqueChose()
    {
        Console.WriteLine("Le service fait quelque chose...");
    }
}
```

L'utilisation reste la même, mais le code est beaucoup plus simple et moins susceptible aux erreurs :

```csharp
// Premier appel - initialise l'instance
ServiceCouteux.Instance.FaireQuelqueChose();

// Les appels suivants utilisent l'instance existante
ServiceCouteux.Instance.FaireQuelqueChose();
```

## 17.3.6. TPL Dataflow

La bibliothèque Task Parallel Library (TPL) Dataflow est une extension de la TPL qui simplifie la création d'applications parallèles, particulièrement pour les systèmes de traitement de données par pipeline.

### Concepts clés de TPL Dataflow

TPL Dataflow permet de construire des réseaux de blocs de traitement qui peuvent s'exécuter en parallèle, communiquer par messages et gérer les données comme un pipeline.

#### Blocs de base

1. **ActionBlock<T>** : Exécute une action pour chaque élément d'entrée
2. **TransformBlock<TInput, TOutput>** : Transforme chaque élément d'entrée en une sortie
3. **TransformManyBlock<TInput, TOutput>** : Transforme chaque élément d'entrée en zéro, un ou plusieurs éléments de sortie
4. **BatchBlock<T>** : Regroupe les éléments d'entrée en lots
5. **JoinBlock<T1, T2, ...>** : Attend un élément de chaque entrée et les combine
6. **BroadcastBlock<T>** : Envoie chaque élément d'entrée à tous les blocs liés
7. **BufferBlock<T>** : Stocke les éléments pour qu'ils soient consommés par d'autres blocs

### Installation de TPL Dataflow

TPL Dataflow est disponible via le package NuGet `System.Threading.Tasks.Dataflow`.

```bash
dotnet add package System.Threading.Tasks.Dataflow
```

### Exemple simple : pipeline de traitement de texte

Créons un pipeline qui lit du texte, compte les mots et calcule des statistiques.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
using System.Threading.Tasks.Dataflow;

class Program
{
    static async Task Main()
    {
        // 1. Bloc pour lire les fichiers texte
        var lecteurFichier = new TransformBlock<string, string>(async cheminFichier =>
        {
            Console.WriteLine($"Lecture du fichier: {cheminFichier}");
            return await File.ReadAllTextAsync(cheminFichier);
        });

        // 2. Bloc pour diviser le texte en mots
        var extracteurMots = new TransformBlock<string, string[]>(texte =>
        {
            Console.WriteLine("Extraction des mots...");
            return Regex.Split(texte, @"\W+")
                       .Where(mot => !string.IsNullOrWhiteSpace(mot))
                       .Select(mot => mot.ToLower())
                       .ToArray();
        });

        // 3. Bloc pour compter la fréquence des mots
        var compteurMots = new TransformBlock<string[], Dictionary<string, int>>(mots =>
        {
            Console.WriteLine("Comptage des mots...");
            return mots.GroupBy(mot => mot)
                      .ToDictionary(groupe => groupe.Key, groupe => groupe.Count());
        });

        // 4. Bloc pour calculer les statistiques
        var calculateurStats = new ActionBlock<Dictionary<string, int>>(dictionnaire =>
        {
            Console.WriteLine("\nStatistiques:");
            Console.WriteLine($"Nombre de mots uniques: {dictionnaire.Count}");

            if (dictionnaire.Count > 0)
            {
                int totalOccurrences = dictionnaire.Values.Sum();
                Console.WriteLine($"Nombre total de mots: {totalOccurrences}");

                var motsLesPlusFréquents = dictionnaire
                    .OrderByDescending(paire => paire.Value)
                    .Take(5)
                    .ToList();

                Console.WriteLine("\nMots les plus fréquents:");
                foreach (var paire in motsLesPlusFréquents)
                {
                    Console.WriteLine($"  '{paire.Key}': {paire.Value} occurrences");
                }
            }
        });

        // Lier les blocs pour former un pipeline
        lecteurFichier.LinkTo(extracteurMots, new DataflowLinkOptions { PropagateCompletion = true });
        extracteurMots.LinkTo(compteurMots, new DataflowLinkOptions { PropagateCompletion = true });
        compteurMots.LinkTo(calculateurStats, new DataflowLinkOptions { PropagateCompletion = true });

        // Démarrer le traitement
        string cheminFichier = "exemple.txt";

        // Créer un fichier d'exemple si nécessaire
        if (!File.Exists(cheminFichier))
        {
            File.WriteAllText(cheminFichier, @"
                TPL Dataflow est une bibliothèque qui simplifie la programmation parallèle
                en C#. Elle permet de construire des pipelines de traitement de données
                efficaces. Les pipelines peuvent traiter des données en parallèle et sont
                particulièrement utiles pour les applications qui traitent des flux de données.
            ");
        }

        // Envoyer le chemin du fichier au premier bloc
        lecteurFichier.Post(cheminFichier);

        // Signaler la fin des entrées
        lecteurFichier.Complete();

        // Attendre que tout le traitement soit terminé
        await calculateurStats.Completion;
    }
}
```

### Contrôle du parallélisme

Vous pouvez configurer le degré de parallélisme de chaque bloc :

```csharp
var options = new ExecutionDataflowBlockOptions
{
    MaxDegreeOfParallelism = Environment.ProcessorCount,
    BoundedCapacity = 100  // Limite la taille de la file d'attente
};

var traitementParallèle = new TransformBlock<int, int>(
    n =>
    {
        Console.WriteLine($"Traitement de {n} sur le thread {Thread.CurrentThread.ManagedThreadId}");
        Thread.Sleep(100);  // Simulation de travail
        return n * n;
    },
    options
);
```

### Gestion des erreurs

TPL Dataflow propage les exceptions à travers le pipeline jusqu'au bloc qui observe `Completion` :

```csharp
try
{
    // Attendre la complétion du dernier bloc
    await dernierBloc.Completion;
}
catch (AggregateException ex)
{
    foreach (var innerEx in ex.InnerExceptions)
    {
        Console.WriteLine($"Erreur dans le pipeline: {innerEx.Message}");
    }
}
```

### Filtrage et routage conditionnels

Vous pouvez router les données conditionnellement :

```csharp
// Bloc source
var source = new BufferBlock<int>();

// Deux blocs cibles
var blocPairs = new ActionBlock<int>(n => Console.WriteLine($"Pair: {n}"));
var blocImpairs = new ActionBlock<int>(n => Console.WriteLine($"Impair: {n}"));

// Lien conditionnel
source.LinkTo(blocPairs, n => n % 2 == 0);  // Route les nombres pairs
source.LinkTo(blocImpairs, n => n % 2 != 0);  // Route les nombres impairs

// Pour éviter que des éléments restent bloqués dans source
// si aucun prédicat ne correspond
source.LinkTo(DataflowBlock.NullTarget<int>());

// Envoyer des données
for (int i = 0; i < 10; i++)
{
    source.Post(i);
}

source.Complete();
```

### Exemple avancé : traitement d'images parallèle avec TPL Dataflow

Voici un exemple plus complet qui utilise TPL Dataflow pour créer un pipeline de traitement d'images. Ce pipeline va charger des images, appliquer différents filtres et les sauvegarder, le tout en parallèle.

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using System.Threading.Tasks.Dataflow;

class Program
{
    static async Task Main()
    {
        // Créer un dossier pour les images traitées
        string dossierSortie = Path.Combine(Directory.GetCurrentDirectory(), "images_traitées");
        if (!Directory.Exists(dossierSortie))
        {
            Directory.CreateDirectory(dossierSortie);
        }

        // Configurer les options des blocs
        var options = new ExecutionDataflowBlockOptions
        {
            MaxDegreeOfParallelism = Environment.ProcessorCount,
            BoundedCapacity = 5  // Limiter la file d'attente pour éviter de consommer trop de mémoire
        };

        // 1. Bloc pour trouver les fichiers image
        var découvreurFichiers = new TransformManyBlock<string, string>(dossier =>
        {
            Console.WriteLine($"Recherche d'images dans {dossier}");
            return Directory.GetFiles(dossier, "*.jpg")
                           .Concat(Directory.GetFiles(dossier, "*.png"));
        });

        // 2. Bloc pour charger les images
        var chargeurImages = new TransformBlock<string, ImageInfo>(async cheminFichier =>
        {
            Console.WriteLine($"Chargement de {Path.GetFileName(cheminFichier)}");

            // Simuler une opération I/O asynchrone
            await Task.Delay(100);

            return new ImageInfo
            {
                CheminFichier = cheminFichier,
                NomFichier = Path.GetFileName(cheminFichier),
                Image = new Bitmap(cheminFichier)
            };
        }, options);

        // 3. Bloc pour filtrer les images
        var filtreurImages = new TransformBlock<ImageInfo, ImageInfo>(imageInfo =>
        {
            Console.WriteLine($"Application de filtres à {imageInfo.NomFichier}");

            // Appliquer un filtre (exemple : niveaux de gris)
            imageInfo.Image = AppliquerFiltreNoirEtBlanc(imageInfo.Image);

            return imageInfo;
        }, options);

        // 4. Bloc pour redimensionner les images
        var redimensionneur = new TransformBlock<ImageInfo, ImageInfo>(imageInfo =>
        {
            Console.WriteLine($"Redimensionnement de {imageInfo.NomFichier}");

            // Réduire la taille de l'image
            int nouvelleLargeur = imageInfo.Image.Width / 2;
            int nouvelleHauteur = imageInfo.Image.Height / 2;

            var nouvelleImage = new Bitmap(nouvelleLargeur, nouvelleHauteur);
            using (var g = Graphics.FromImage(nouvelleImage))
            {
                g.DrawImage(imageInfo.Image, 0, 0, nouvelleLargeur, nouvelleHauteur);
            }

            // Libérer la mémoire de l'ancienne image
            imageInfo.Image.Dispose();
            imageInfo.Image = nouvelleImage;

            return imageInfo;
        }, options);

        // 5. Bloc pour sauvegarder les images
        var sauveurImages = new ActionBlock<ImageInfo>(async imageInfo =>
        {
            string cheminSortie = Path.Combine(dossierSortie,
                                              $"traité_{imageInfo.NomFichier}");

            Console.WriteLine($"Sauvegarde de {cheminSortie}");

            imageInfo.Image.Save(cheminSortie);

            // Libérer les ressources
            imageInfo.Image.Dispose();

            // Simuler une opération I/O asynchrone
            await Task.Delay(100);
        }, options);

        // Lier les blocs pour former un pipeline
        découvreurFichiers.LinkTo(chargeurImages, new DataflowLinkOptions { PropagateCompletion = true });
        chargeurImages.LinkTo(filtreurImages, new DataflowLinkOptions { PropagateCompletion = true });
        filtreurImages.LinkTo(redimensionneur, new DataflowLinkOptions { PropagateCompletion = true });
        redimensionneur.LinkTo(sauveurImages, new DataflowLinkOptions { PropagateCompletion = true });

        // Créer des images de test si nécessaire
        string dossierTest = Path.Combine(Directory.GetCurrentDirectory(), "images_test");
        if (!Directory.Exists(dossierTest))
        {
            Directory.CreateDirectory(dossierTest);
            for (int i = 1; i <= 5; i++)
            {
                CréerImageTest(Path.Combine(dossierTest, $"test{i}.png"), 400, 300);
            }
        }

        // Démarrer le traitement
        Stopwatch sw = Stopwatch.StartNew();

        découvreurFichiers.Post(dossierTest);
        découvreurFichiers.Complete();

        try
        {
            // Attendre que tout le pipeline termine
            await sauveurImages.Completion;

            sw.Stop();
            Console.WriteLine($"Traitement terminé en {sw.ElapsedMilliseconds} ms");
        }
        catch (AggregateException ex)
        {
            foreach (var innerEx in ex.InnerExceptions)
            {
                Console.WriteLine($"Erreur: {innerEx.Message}");
            }
        }
    }

    // Classe pour encapsuler les informations d'image
    class ImageInfo
    {
        public string CheminFichier { get; set; }
        public string NomFichier { get; set; }
        public Bitmap Image { get; set; }
    }

    // Appliquer un filtre noir et blanc
    static Bitmap AppliquerFiltreNoirEtBlanc(Bitmap originale)
    {
        Bitmap nouvelleImage = new Bitmap(originale.Width, originale.Height);

        for (int x = 0; x < originale.Width; x++)
        {
            for (int y = 0; y < originale.Height; y++)
            {
                Color pixel = originale.GetPixel(x, y);
                int gris = (int)(pixel.R * 0.3 + pixel.G * 0.59 + pixel.B * 0.11);
                Color nouveauPixel = Color.FromArgb(pixel.A, gris, gris, gris);
                nouvelleImage.SetPixel(x, y, nouveauPixel);
            }
        }

        return nouvelleImage;
    }

    // Créer une image de test
    static void CréerImageTest(string chemin, int largeur, int hauteur)
    {
        using (Bitmap bitmap = new Bitmap(largeur, hauteur))
        {
            using (Graphics g = Graphics.FromImage(bitmap))
            {
                // Remplir avec une couleur de fond
                g.Clear(Color.LightBlue);

                // Dessiner des formes aléatoires
                Random rand = new Random();
                using (SolidBrush brush = new SolidBrush(Color.FromArgb(
                    rand.Next(256), rand.Next(256), rand.Next(256))))
                {
                    for (int i = 0; i < 10; i++)
                    {
                        int x = rand.Next(largeur);
                        int y = rand.Next(hauteur);
                        int taille = rand.Next(20, 100);

                        brush.Color = Color.FromArgb(
                            rand.Next(256), rand.Next(256), rand.Next(256));

                        g.FillEllipse(brush, x, y, taille, taille);
                    }
                }

                // Ajouter du texte
                using (Font font = new Font("Arial", 16))
                {
                    using (SolidBrush textBrush = new SolidBrush(Color.Black))
                    {
                        g.DrawString("Image de test", font, textBrush,
                                    new PointF(largeur / 4, hauteur / 2));
                    }
                }
            }

            bitmap.Save(chemin);
        }
    }
}
```

> **Note** : Pour exécuter cet exemple, vous aurez besoin du package NuGet `System.Drawing.Common`. Notez également que l'exemple ci-dessus est simplifié pour des raisons didactiques. Dans une application réelle, vous utiliseriez des méthodes plus efficaces pour le traitement d'images, comme l'accès direct au buffer de l'image.

### Annulation dans TPL Dataflow

TPL Dataflow prend en charge l'annulation via les jetons d'annulation (CancellationToken) :

```csharp
async Task ExempleAnnulation()
{
    // Créer une source de jeton d'annulation
    var cts = new CancellationTokenSource();

    // Configurer les options avec le jeton d'annulation
    var options = new ExecutionDataflowBlockOptions
    {
        CancellationToken = cts.Token
    };

    // Créer un bloc qui met du temps à s'exécuter
    var bloc = new TransformBlock<int, int>(async n =>
    {
        Console.WriteLine($"Début du traitement de {n}");

        try
        {
            // Simuler un long traitement
            await Task.Delay(5000, cts.Token);
            return n * n;
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine($"Traitement de {n} annulé");
            throw;  // Propager l'annulation
        }
    }, options);

    // Poster quelques éléments
    for (int i = 0; i < 10; i++)
    {
        bloc.Post(i);
    }

    // Annuler après 2 secondes
    await Task.Delay(2000);
    Console.WriteLine("Demande d'annulation...");
    cts.Cancel();

    try
    {
        // Attendre la complétion ou l'annulation
        await bloc.Completion;
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Opération annulée avec succès");
    }
}
```

### Avantages et cas d'utilisation de TPL Dataflow

TPL Dataflow est particulièrement utile dans les scénarios suivants :

1. **Traitement de données en pipeline** : lorsque vos données passent par plusieurs étapes de traitement
2. **Systèmes producteur-consommateur** : quand des données sont produites et consommées à des rythmes différents
3. **Parallélisme avec flux de données** : quand les données circulent dans un graphe et où chaque étape peut être parallélisée
4. **ETL (Extract, Transform, Load)** : pour les applications qui extraient, transforment et chargent des données
5. **Traitement d'événements asynchrones** : pour gérer des flux d'événements avec traitement parallèle

## Conclusion et bonnes pratiques

Le multithreading et le parallélisme en C# offrent des moyens puissants d'optimiser les performances de vos applications. Voici un résumé des concepts clés et des bonnes pratiques à retenir :

### Résumé des concepts clés

1. **Thread vs Task** :
   - Les threads sont des unités d'exécution de bas niveau
   - Les tasks sont des abstractions de plus haut niveau pour le travail asynchrone et parallèle
   - Préférez les tasks pour la plupart des scénarios modernes

2. **Parallel LINQ (PLINQ)** :
   - Extension parallèle de LINQ pour traiter des collections
   - Simple à utiliser avec `.AsParallel()`
   - Optimisé pour les opérations de filtrage, transformation et agrégation

3. **Synchronisation** :
   - Nécessaire lorsque plusieurs threads accèdent aux mêmes données
   - Mécanismes comme `lock`, `Monitor`, `Mutex`, `SemaphoreSlim`
   - Collections thread-safe dans `System.Collections.Concurrent`

4. **Problèmes courants** :
   - Race conditions : accès non synchronisés aux données partagées
   - Deadlocks : threads qui s'attendent mutuellement
   - Livelocks : threads qui changent d'état sans progresser
   - Starvation : threads qui ne peuvent pas accéder aux ressources

5. **Opérations atomiques** :
   - Opérations indivisibles sur des variables partagées
   - `Interlocked` pour des opérations atomiques simples
   - Initialisation paresseuse avec `Lazy<T>`

6. **TPL Dataflow** :
   - Création de pipelines de traitement de données
   - Routage et filtrage des données
   - Contrôle fin du parallélisme
   - Gestion robuste des erreurs et annulations

### Bonnes pratiques

1. **Choisir le bon outil** :
   - Pour la programmation asynchrone simple (I/O) : `async`/`await`
   - Pour le parallélisme de données : PLINQ ou `Parallel.For/ForEach`
   - Pour des pipelines de traitement : TPL Dataflow
   - Pour des opérations de bas niveau : Tasks ou rarement Threads

2. **Éviter les pièges courants** :
   - Minimiser l'accès partagé à l'état mutable
   - Préférer l'immutabilité quand c'est possible
   - Acquérir les verrous dans un ordre cohérent
   - Garder les sections critiques courtes
   - Éviter les opérations bloquantes dans du code asynchrone

3. **Synchronisation** :
   - Utiliser des mécanismes de plus haut niveau quand c'est possible (collections thread-safe)
   - Utiliser des structures de données adaptées au scénario (lecture intensive vs écriture)
   - Utiliser `lock` pour des verrous simples, `ReaderWriterLockSlim` pour les scénarios lecture/écriture

4. **Performances** :
   - Mesurer avant d'optimiser
   - Le parallélisme n'est pas toujours plus rapide (frais généraux)
   - Ajuster le degré de parallélisme selon la charge de travail et le matériel
   - Éviter la surcharge du ThreadPool

5. **Débogage** :
   - Utiliser les outils de parallélisme de Visual Studio
   - Activer "Break on all exceptions" pour détecter les exceptions dans les threads
   - Examiner les stacks de tous les threads pour les deadlocks

### Exemple final : modèle asynchrone et parallèle complet

Voici un exemple qui combine différentes techniques de programmation parallèle et asynchrone :

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Net.Http;
using System.Text.RegularExpressions;
using System.Threading;
using System.Threading.Tasks;
using System.Threading.Tasks.Dataflow;

class Program
{
    static async Task Main()
    {
        await CrawlerWeb("https://example.com", profondeurMax: 2);
    }

    static async Task CrawlerWeb(string urlDépart, int profondeurMax)
    {
        // Suivi des URLs déjà visitées
        ConcurrentDictionary<string, bool> urlsVisitées = new ConcurrentDictionary<string, bool>();

        // Client HTTP réutilisable
        using HttpClient client = new HttpClient();

        // Configurer un jeton d'annulation avec timeout de 30 secondes
        var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));

        // Options pour limiter le parallélisme
        var options = new ExecutionDataflowBlockOptions
        {
            MaxDegreeOfParallelism = 10,  // Limiter à 10 téléchargements simultanés
            BoundedCapacity = 100,        // Limiter la file d'attente
            CancellationToken = cts.Token
        };

        // 1. Bloc pour télécharger les pages
        var téléchargeur = new TransformBlock<PageInfo, PageInfo>(async pageInfo =>
        {
            if (pageInfo.Profondeur > profondeurMax)
                return pageInfo;

            Console.WriteLine($"Téléchargement: {pageInfo.Url} (Profondeur: {pageInfo.Profondeur})");

            try
            {
                string html = await client.GetStringAsync(pageInfo.Url, cts.Token);
                pageInfo.ContenuHtml = html;
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Erreur lors du téléchargement de {pageInfo.Url}: {ex.Message}");
            }

            return pageInfo;
        }, options);

        // 2. Bloc pour extraire les liens et le texte
        var extracteur = new TransformBlock<PageInfo, PageInfo>(pageInfo =>
        {
            if (string.IsNullOrEmpty(pageInfo.ContenuHtml))
                return pageInfo;

            Console.WriteLine($"Extraction des liens: {pageInfo.Url}");

            // Extraire le texte (version simplifiée)
            pageInfo.TexteExtrait = Regex.Replace(pageInfo.ContenuHtml, "<[^>]*>", "");

            // Extraire les URLs
            var matches = Regex.Matches(pageInfo.ContenuHtml,
                                       @"href\s*=\s*[""']([^""']+)[""']",
                                       RegexOptions.IgnoreCase);

            foreach (Match match in matches)
            {
                string url = match.Groups[1].Value;

                // Ignorer les URLs non-HTTP
                if (!url.StartsWith("http"))
                    continue;

                pageInfo.LiensExtraits.Add(url);
            }

            return pageInfo;
        }, options);

        // 3. Bloc pour analyser le texte
        var analyseur = new ActionBlock<PageInfo>(pageInfo =>
        {
            if (string.IsNullOrEmpty(pageInfo.TexteExtrait))
                return;

            Console.WriteLine($"Analyse: {pageInfo.Url}");

            // Compter les mots
            string[] mots = pageInfo.TexteExtrait.Split(new[] { ' ', '\t', '\r', '\n' },
                                                      StringSplitOptions.RemoveEmptyEntries);

            Console.WriteLine($"Page: {pageInfo.Url}");
            Console.WriteLine($"  Profondeur: {pageInfo.Profondeur}");
            Console.WriteLine($"  Nombre de mots: {mots.Length}");
            Console.WriteLine($"  Nombre de liens: {pageInfo.LiensExtraits.Count}");

            // Soumettre les nouvelles URLs pour téléchargement
            foreach (string url in pageInfo.LiensExtraits)
            {
                // Ne pas revisiter les URLs déjà vues
                if (urlsVisitées.TryAdd(url, true))
                {
                    téléchargeur.Post(new PageInfo
                    {
                        Url = url,
                        Profondeur = pageInfo.Profondeur + 1
                    });
                }
            }
        }, options);

        // Lier les blocs
        téléchargeur.LinkTo(extracteur, new DataflowLinkOptions { PropagateCompletion = true });
        extracteur.LinkTo(analyseur, new DataflowLinkOptions { PropagateCompletion = true });

        // Démarrer le crawling
        Stopwatch sw = Stopwatch.StartNew();

        urlsVisitées.TryAdd(urlDépart, true);
        téléchargeur.Post(new PageInfo { Url = urlDépart, Profondeur = 0 });

        // Après avoir posté toutes les URLs initiales, marquer le bloc comme complet
        téléchargeur.Complete();

        try
        {
            // Attendre que tout le pipeline termine
            await analyseur.Completion;

            sw.Stop();
            Console.WriteLine($"Crawling terminé en {sw.ElapsedMilliseconds}ms");
            Console.WriteLine($"Pages visitées: {urlsVisitées.Count}");
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Opération annulée (timeout)");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur: {ex.Message}");
        }
    }

    class PageInfo
    {
        public string Url { get; set; }
        public int Profondeur { get; set; }
        public string ContenuHtml { get; set; }
        public string TexteExtrait { get; set; }
        public List<string> LiensExtraits { get; set; } = new List<string>();
    }
}
```

Ce crawler web combine plusieurs techniques avancées :
- Programmation asynchrone avec `async`/`await`
- TPL Dataflow pour le pipeline de traitement
- Collections thread-safe (`ConcurrentDictionary`)
- Contrôle de parallélisme (limitation du nombre de téléchargements simultanés)
- Annulation avec timeout
- Extraction d'expressions régulières dans un contexte parallèle

### Les tendances futures

La programmation parallèle et asynchrone continue d'évoluer en C# :

1. **Channels** : Similaires à TPL Dataflow mais avec une API plus simple et un focus sur les principes de communication des canaux.

```csharp
using System.Threading.Channels;

// Créer un canal non borné
var channel = Channel.CreateUnbounded<int>();

// Tâche du producteur
Task producteur = Task.Run(async () =>
{
    for (int i = 0; i < 100; i++)
    {
        await channel.Writer.WriteAsync(i);
        await Task.Delay(100);
    }
    channel.Writer.Complete();
});

// Tâche du consommateur
Task consommateur = Task.Run(async () =>
{
    await foreach (var item in channel.Reader.ReadAllAsync())
    {
        Console.WriteLine($"Reçu: {item}");
    }
});

await Task.WhenAll(producteur, consommateur);
```

2. **Pipelines** : API de bas niveau pour le traitement efficace des données binaires (System.IO.Pipelines).

3. **Support amélioré pour la concurrence dans .NET** : Avec chaque nouvelle version de .NET et C#, le support pour la programmation concurrente s'améliore.

En maîtrisant les concepts et techniques décrits dans ce chapitre, vous serez bien équipé pour développer des applications C# efficaces et performantes qui tirent pleinement parti des processeurs multi-cœurs modernes.

⏭️
