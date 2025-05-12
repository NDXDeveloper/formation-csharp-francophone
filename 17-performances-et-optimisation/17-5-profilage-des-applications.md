# 17.5. Profilage des applications en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le profilage est l'art de mesurer et d'analyser les performances de votre application pour identifier les goulots d'étranglement, les fuites de mémoire et d'autres problèmes qui peuvent affecter l'expérience utilisateur. Ce chapitre vous introduira aux concepts fondamentaux du profilage et vous montrera comment utiliser divers outils pour diagnostiquer et résoudre les problèmes de performance dans vos applications C#.

## 17.5.1. Outils de profilage

Avant de plonger dans les techniques spécifiques, familiarisons-nous avec les principaux outils de profilage disponibles pour les développeurs C#.

### Visual Studio Profiler

Visual Studio inclut un ensemble puissant d'outils de profilage intégrés, disponibles dans presque toutes les éditions (Community, Professional et Enterprise).

#### Comment lancer le profileur dans Visual Studio

1. Ouvrez votre projet dans Visual Studio
2. Dans le menu, cliquez sur **Debug > Performance Profiler**
3. Sélectionnez les outils de diagnostic que vous souhaitez activer
4. Cliquez sur **Start** pour lancer l'application avec le profileur

#### Types d'analyse disponibles

Visual Studio propose plusieurs types d'analyse :

- **CPU Usage** : Identifie les fonctions qui consomment le plus de temps CPU
- **Memory Usage** : Analyse la consommation de mémoire et aide à identifier les fuites
- **Application Timeline** : Visualise l'activité des threads et les événements au fil du temps
- **GPU Usage** : Mesure les performances du GPU pour les applications graphiques
- **Database** : Profile les requêtes de base de données
- **DotTrace Integration** : Pour l'édition Enterprise, intégration avec l'outil JetBrains DotTrace

![Interface du profileur Visual Studio](https://i.imgur.com/example1.png)

### Outils tiers populaires

En plus des outils intégrés à Visual Studio, plusieurs solutions tierces offrent des fonctionnalités avancées :

#### JetBrains dotTrace

Un profileur de performances complet qui s'intègre à Visual Studio et Rider.

**Points forts :**
- Profilage à distance
- Profilage de ligne de temps
- Filtrage et regroupement avancés
- Peut être utilisé sans redémarrer l'application (attach to process)

#### JetBrains dotMemory

Spécialisé dans l'analyse de mémoire et la détection de fuites.

**Points forts :**
- Visualisation des graphes d'objets
- Comparaison de snapshots mémoire
- Détection automatique des problèmes courants
- Analyse de génération GC

#### ANTS Performance Profiler (Red Gate)

Un outil complet pour le profilage de code .NET.

**Points forts :**
- Interface intuitive
- Mesure du temps passé dans le code, les bases de données et les requêtes HTTP
- Suivi des allocations d'objets
- Profilage en production

#### PerfView

Un outil gratuit et open-source de Microsoft pour le profilage avancé.

**Points forts :**
- Léger et portable (pas d'installation nécessaire)
- Très puissant pour l'analyse du Garbage Collector
- Peut collecter des données ETW (Event Tracing for Windows)
- Idéal pour les problèmes avancés

### Outils de ligne de commande

Pour les environnements sans interface graphique ou pour l'automatisation, plusieurs outils en ligne de commande sont disponibles :

#### dotnet-trace

Un outil de diagnostic inclus dans le SDK .NET Core.

```bash
# Installation
dotnet tool install --global dotnet-trace

# Collecte d'une trace
dotnet-trace collect --process-id <PID>
```

#### dotnet-counters

Pour surveiller les compteurs de performance en temps réel.

```bash
# Installation
dotnet tool install --global dotnet-counters

# Surveillance en temps réel
dotnet-counters monitor --process-id <PID>
```

#### dotnet-dump

Pour capturer et analyser des dumps mémoire.

```bash
# Installation
dotnet tool install --global dotnet-dump

# Capture d'un dump
dotnet-dump collect --process-id <PID>

# Analyse du dump
dotnet-dump analyze core_XXXXX.dump
```

### Choisir le bon outil

Le choix de l'outil dépendra de plusieurs facteurs :

- **Type de problème** : CPU, mémoire, réseau, etc.
- **Environnement** : développement, test, production
- **Niveau de détail requis** : de haut niveau ou analyse approfondie
- **Budget** : outils gratuits vs payants
- **Expérience** : certains outils sont plus accessibles aux débutants

Pour les débutants, je recommande de commencer avec les outils intégrés à Visual Studio, puis d'explorer les solutions tierces une fois que vous êtes à l'aise avec les concepts de base.

## 17.5.2. Mesure de performances

La mesure précise des performances est essentielle pour identifier les problèmes et valider les optimisations. Voyons différentes approches pour mesurer les performances de votre application C#.

### Utilisation de Stopwatch

La classe `Stopwatch` du namespace `System.Diagnostics` est l'outil le plus simple pour mesurer le temps d'exécution d'une portion de code.

```csharp
using System;
using System.Diagnostics;

public void MesurerPerformance()
{
    // Créer et démarrer un chronomètre
    Stopwatch sw = Stopwatch.StartNew();

    // Code à mesurer
    for (int i = 0; i < 1000000; i++)
    {
        // Opération coûteuse
        Math.Pow(i, 2);
    }

    // Arrêter le chronomètre
    sw.Stop();

    // Afficher les résultats
    Console.WriteLine($"Temps écoulé: {sw.ElapsedMilliseconds} ms");
    Console.WriteLine($"Temps écoulé (précision): {sw.Elapsed.TotalSeconds} secondes");

    // Réinitialiser et réutiliser
    sw.Reset();
    sw.Start();

    // Autre code à mesurer...

    sw.Stop();
    Console.WriteLine($"Deuxième mesure: {sw.ElapsedMilliseconds} ms");
}
```

#### Bonnes pratiques avec Stopwatch

- **Exécutez plusieurs fois** votre test pour tenir compte des variations naturelles
- **Ignorez la première exécution** car elle peut inclure des coûts d'initialisation (JIT compilation)
- **Évitez de mesurer des opérations trop courtes** (moins de quelques millisecondes)
- **Désactivez le débogueur** pour des mesures plus précises (exécutez en mode Release)

### Création d'une classe utilitaire pour les mesures

Pour simplifier les mesures répétitives, vous pouvez créer une classe utilitaire :

```csharp
public static class PerformanceUtil
{
    public static TimeSpan Mesurer(Action action, int iterations = 1)
    {
        // Échauffement (JIT compilation)
        action();

        // Mesure réelle
        Stopwatch sw = Stopwatch.StartNew();

        for (int i = 0; i < iterations; i++)
        {
            action();
        }

        sw.Stop();

        // Retourner le temps moyen par itération
        return TimeSpan.FromMilliseconds(sw.ElapsedMilliseconds / (double)iterations);
    }

    public static void ComparePerformance(
        string nom1, Action action1,
        string nom2, Action action2,
        int iterations = 1000)
    {
        Console.WriteLine($"Comparaison de performance ({iterations} itérations):");

        var temps1 = Mesurer(action1, iterations);
        var temps2 = Mesurer(action2, iterations);

        Console.WriteLine($"{nom1}: {temps1.TotalMilliseconds:F3} ms (par itération)");
        Console.WriteLine($"{nom2}: {temps2.TotalMilliseconds:F3} ms (par itération)");

        double ratio = temps1.TotalMilliseconds / temps2.TotalMilliseconds;
        string plusRapide = ratio > 1 ? nom2 : nom1;
        double pourcentage = Math.Abs(1 - ratio) * 100;

        Console.WriteLine($"{plusRapide} est {pourcentage:F1}% plus rapide");
    }
}
```

Utilisation :

```csharp
// Mesurer une seule méthode
TimeSpan temps = PerformanceUtil.Mesurer(() => MaFonction(param1, param2), 100);
Console.WriteLine($"Temps moyen: {temps.TotalMilliseconds} ms");

// Comparer deux approches
PerformanceUtil.ComparePerformance(
    "Méthode A", () => MéthodeA(),
    "Méthode B", () => MéthodeB(),
    1000
);
```

### Profiler avec Visual Studio Diagnostics Tools

Pour une analyse plus détaillée, utilisez les outils de diagnostic intégrés à Visual Studio :

1. Lancez l'application en mode Debug (F5)
2. Ouvrez la fenêtre Diagnostic Tools (Debug > Windows > Show Diagnostic Tools)
3. Cliquez sur "Record CPU Profile" pour commencer l'enregistrement
4. Effectuez l'opération que vous souhaitez profiler
5. Arrêtez l'enregistrement et analysez les résultats

![Diagnostic Tools dans Visual Studio](https://i.imgur.com/example2.png)

### Profilage CPU avec Visual Studio Profiler

Pour une analyse CPU plus approfondie :

1. Allez dans Debug > Performance Profiler
2. Sélectionnez "CPU Usage"
3. Cliquez sur "Start"
4. Effectuez l'opération à analyser
5. Arrêtez le profilage

Vous verrez alors un rapport détaillé :

- **Hot Paths** : chemins d'exécution les plus coûteux
- **Functions** : fonctions qui consomment le plus de CPU
- **Call Tree** : arbre d'appels avec temps passé dans chaque fonction
- **Caller/Callee** : qui appelle quoi et combien de temps est passé

#### Interpréter les résultats du profilage CPU

- **Inclusive Time** : temps total passé dans une fonction, y compris les appels qu'elle fait
- **Exclusive Time** : temps passé uniquement dans le code de la fonction, sans les appels
- **Module** : DLL ou exécutable contenant la fonction
- **Function** : nom de la fonction (peut être long avec génériques et namespaces)

### Profilage de code asynchrone

Le profilage de code asynchrone présente des défis particuliers car l'exécution traverse plusieurs threads et peut inclure des temps d'attente.

```csharp
// Exemple de code asynchrone à profiler
public async Task<string> ChargerDonnéesAsync()
{
    Stopwatch sw = Stopwatch.StartNew();

    // Opération réseau asynchrone
    var résultat = await client.GetStringAsync("https://api.exemple.com/données");

    // Traitement des données
    var donnéesTraitées = TraiterDonnées(résultat);

    sw.Stop();
    Console.WriteLine($"Temps total: {sw.ElapsedMilliseconds} ms");

    return donnéesTraitées;
}
```

Pour le code asynchrone, il est important de distinguer :
- **Temps total** : durée complète de l'opération (incluant les attentes)
- **Temps CPU** : temps réellement passé à exécuter du code
- **Temps d'attente** : temps passé à attendre des opérations I/O ou autres

Visual Studio Timeline Profiler est particulièrement utile pour visualiser ces différents aspects.

## 17.5.3. Diagnostic Memory Leak

Les fuites de mémoire (memory leaks) se produisent lorsque votre application alloue de la mémoire qu'elle n'arrive jamais à libérer. Dans les applications .NET, cela se produit généralement quand vous conservez des références à des objets qui ne sont plus nécessaires.

### Signes de fuites de mémoire

Comment reconnaître une potentielle fuite de mémoire :

- L'utilisation de la mémoire augmente progressivement au fil du temps
- Les performances se dégradent progressivement
- L'application finit par planter avec une `OutOfMemoryException`
- Le Garbage Collector semble s'exécuter fréquemment

### Causes courantes des fuites de mémoire

1. **Événements non désenregistrés** : s'abonner à un événement sans jamais se désabonner
2. **Collections statiques** : ajouter des objets à des collections statiques sans jamais les retirer
3. **Closures et captures de variables** : capturer par inadvertance des références dans des délégués ou lambdas
4. **Ressources non managées** : ne pas libérer correctement les ressources non managées
5. **Caches sans limite** : implémenter des caches qui grandissent sans contrôle

### Diagnostiquer les fuites avec Visual Studio Memory Profiler

Pour identifier les fuites de mémoire :

1. Ouvrez Debug > Performance Profiler
2. Cochez "Memory Usage"
3. Cliquez sur "Start"
4. Dans la session, cliquez sur "Take snapshot" pour capturer l'état initial
5. Effectuez l'opération susceptible de causer des fuites
6. Cliquez à nouveau sur "Take snapshot"
7. Répétez l'opération et prenez un troisième snapshot
8. Comparez les snapshots pour voir quels objets s'accumulent

![Comparaison de snapshots](https://i.imgur.com/example3.png)

### Comparer les snapshots mémoire

La comparaison de snapshots est une technique puissante :

1. Sélectionnez deux snapshots à comparer
2. Visual Studio affiche les différences en termes de :
   - Nombre d'objets
   - Taille totale
   - Types d'objets qui ont augmenté

3. Examinez les objets qui ont augmenté en nombre
4. Utilisez la vue "References" pour voir qui retient ces objets en mémoire

### Exemple : diagnostiquer une fuite d'événements

Considérons cet exemple classique de fuite de mémoire :

```csharp
public class Publisher
{
    public event EventHandler SomeEvent;

    public void RaiseSomeEvent()
    {
        SomeEvent?.Invoke(this, EventArgs.Empty);
    }
}

public class Subscriber
{
    private readonly Publisher _publisher;

    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        // S'abonner à l'événement
        _publisher.SomeEvent += OnSomeEvent;
    }

    private void OnSomeEvent(object sender, EventArgs e)
    {
        Console.WriteLine("Event received");
    }

    // Problème: pas de méthode pour se désabonner!
}

// Dans le code client
void CréerBeaucoupDeSubscribers()
{
    var publisher = new Publisher();

    for (int i = 0; i < 10000; i++)
    {
        var subscriber = new Subscriber(publisher);
        // subscriber tombe hors de portée, mais reste en mémoire
        // car publisher conserve une référence via l'événement
    }
}
```

Pour corriger cette fuite, il faudrait implémenter `IDisposable` dans `Subscriber` et se désabonner de l'événement :

```csharp
public class Subscriber : IDisposable
{
    private readonly Publisher _publisher;
    private bool _disposed = false;

    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        _publisher.SomeEvent += OnSomeEvent;
    }

    private void OnSomeEvent(object sender, EventArgs e)
    {
        Console.WriteLine("Event received");
    }

    public void Dispose()
    {
        if (!_disposed)
        {
            // Se désabonner de l'événement
            _publisher.SomeEvent -= OnSomeEvent;
            _disposed = true;
        }
    }
}

// Utilisation correcte
void CréerBeaucoupDeSubscribers()
{
    var publisher = new Publisher();

    for (int i = 0; i < 10000; i++)
    {
        using (var subscriber = new Subscriber(publisher))
        {
            // Faire quelque chose avec subscriber
        } // subscriber.Dispose() est appelé automatiquement ici
    }
}
```

### Utilisation d'outils spécialisés pour la mémoire

#### dotMemory de JetBrains

dotMemory offre des fonctionnalités avancées pour l'analyse mémoire :

1. **Object exploration** : exploration détaillée des objets en mémoire
2. **Retention paths** : visualisation des chaînes de références qui empêchent la collecte
3. **Dominator tree** : objets qui, s'ils étaient supprimés, libéreraient le plus de mémoire
4. **Génération analysis** : analyse par génération GC

#### Analyse de dumps mémoire

Pour les environnements de production où vous ne pouvez pas attacher un profileur :

1. Capturez un dump mémoire de l'application (via Task Manager ou `procdump`)
2. Analysez-le avec WinDbg, Visual Studio ou dotMemory
3. Recherchez les types d'objets avec de nombreuses instances

```powershell
# Capture d'un dump avec procdump
procdump -ma <PID> memdump.dmp
```

### Conseils pour éviter les fuites de mémoire

1. **Utilisez des références faibles** pour les caches et les gestionnaires d'événements
2. **Implémentez IDisposable** correctement pour les classes qui détiennent des ressources
3. **Limitez la taille des caches** et utilisez des politiques d'expiration
4. **Désabonnez-vous des événements** explicitement
5. **Évitez les collections statiques** qui grandissent sans contrôle
6. **Utilisez des outils de qualité de code** comme ReSharper ou SonarQube qui peuvent détecter certains patterns de fuite

## 17.5.4. Benchmark.NET

Stopwatch est utile pour des mesures simples, mais pour des benchmarks rigoureux et scientifiquement valides, [BenchmarkDotNet](https://benchmarkdotnet.org/) est l'outil de référence.

### Introduction à BenchmarkDotNet

BenchmarkDotNet est une bibliothèque open-source qui vous permet de transformer vos méthodes en benchmarks, d'exécuter ceux-ci de manière contrôlée, et d'obtenir des statistiques détaillées.

#### Installation

```bash
# Via NuGet Package Manager Console
Install-Package BenchmarkDotNet

# Via .NET CLI
dotnet add package BenchmarkDotNet
```

#### Exemple de benchmark basique

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;
using System;
using System.Linq;
using System.Collections.Generic;

[MemoryDiagnoser]  // Pour mesurer aussi les allocations mémoire
public class StringBenchmarks
{
    private string _chaîne = "Hello, World!";
    private int _iterations = 1000;

    [Benchmark]
    public string ConcaténationSimple()
    {
        string résultat = string.Empty;
        for (int i = 0; i < _iterations; i++)
        {
            résultat += _chaîne;
        }
        return résultat;
    }

    [Benchmark]
    public string StringBuilder()
    {
        var sb = new System.Text.StringBuilder();
        for (int i = 0; i < _iterations; i++)
        {
            sb.Append(_chaîne);
        }
        return sb.ToString();
    }

    [Benchmark]
    public string StringJoin()
    {
        var strings = Enumerable.Repeat(_chaîne, _iterations).ToArray();
        return string.Join("", strings);
    }
}

public class Program
{
    public static void Main(string[] args)
    {
        var summary = BenchmarkRunner.Run<StringBenchmarks>();
    }
}
```

Exécutez le programme en mode Release et vous obtiendrez un rapport détaillé :

```
BenchmarkDotNet=v0.13.2
| Method              | Mean       | Error    | StdDev   | Gen0     | Allocated |
|-------------------- |-----------:|---------:|---------:|---------:|----------:|
| ConcaténationSimple | 1,152.7 μs | 21.24 μs | 19.87 μs | 332.0313 | 1,366 KB  |
| StringBuilder       |    13.5 μs |  0.23 μs |  0.21 μs |   5.3711 |    22 KB  |
| StringJoin          |    35.1 μs |  0.69 μs |  0.81 μs |  11.3525 |    47 KB  |
```

### Fonctionnalités avancées de BenchmarkDotNet

#### Paramètres de benchmark

Vous pouvez tester différentes valeurs de paramètres :

```csharp
[Benchmark]
[Arguments(100)]
[Arguments(1000)]
[Arguments(10000)]
public string AvecTaille(int taille)
{
    var sb = new System.Text.StringBuilder();
    for (int i = 0; i < taille; i++)
    {
        sb.Append(_chaîne);
    }
    return sb.ToString();
}

// Alternative avec ParamsAttribute
[Params(100, 1000, 10000)]
public int Taille { get; set; }

[Benchmark]
public string AvecParamètreDePropriété()
{
    var sb = new System.Text.StringBuilder();
    for (int i = 0; i < Taille; i++)
    {
        sb.Append(_chaîne);
    }
    return sb.ToString();
}
```

#### Configuration personnalisée

Vous pouvez personnaliser l'exécution des benchmarks :

```csharp
[Config(typeof(MaConfig))]
public class StringBenchmarks
{
    private class MaConfig : ManualConfig
    {
        public MaConfig()
        {
            AddJob(Job.Default
                .WithWarmupCount(3)          // Nombre d'itérations d'échauffement
                .WithIterationCount(10)       // Nombre d'itérations de mesure
                .WithInvocationCount(1000));  // Nombre d'appels par itération

            AddExporter(HtmlExporter.Default); // Exporte les résultats en HTML
            AddDiagnoser(MemoryDiagnoser.Default); // Mesure les allocations mémoire
        }
    }

    // Benchmarks...
}
```

#### Filtres d'exécution

Vous pouvez exécuter seulement certains benchmarks :

```csharp
var summary = BenchmarkRunner.Run<StringBenchmarks>(
    new[] { "StringBuilder" }  // Exécute uniquement le benchmark "StringBuilder"
);
```

#### Exportation des résultats

BenchmarkDotNet peut exporter les résultats dans différents formats :

```csharp
public class MaConfig : ManualConfig
{
    public MaConfig()
    {
        AddExporter(HtmlExporter.Default);  // HTML
        AddExporter(CsvExporter.Default);   // CSV
        AddExporter(MarkdownExporter.GitHub); // Markdown pour GitHub
        AddExporter(JsonExporter.Full);     // JSON
    }
}
```

### Bonnes pratiques avec BenchmarkDotNet

1. **Exécutez toujours en mode Release**, jamais en Debug
2. **Fermez les autres applications** pendant l'exécution des benchmarks
3. **Incluez un échauffement** suffisant (par défaut, BenchmarkDotNet le fait pour vous)
4. **Benchmarkez des opérations réalistes**, ni trop courtes ni trop longues
5. **Analysez la variance** (colonne StdDev) pour vérifier la fiabilité des résultats
6. **Vérifiez les allocations mémoire** avec le `MemoryDiagnoser`
7. **Isolez les opérations** que vous voulez mesurer

## 17.5.5. Performance counters

Les compteurs de performance sont des indicateurs qui mesurent divers aspects du système et de l'application en temps réel. Ils sont particulièrement utiles pour la surveillance à long terme et le diagnostic des problèmes de performance.

### Types de compteurs de performance

Les applications .NET exposent plusieurs catégories de compteurs :

1. **.NET CLR Memory** : Allocation, GC, génération des objets
2. **.NET CLR Exceptions** : Taux d'exceptions levées
3. **.NET CLR Loading** : Chargement des assemblys et des classes
4. **.NET CLR LocksAndThreads** : Contentions de verrous et activité des threads
5. **ASP.NET** : Requêtes, sessions, etc. (pour les applications web)
6. **Processeur, Mémoire, Disque, Réseau** : Compteurs système standard

### Accès aux compteurs de performance via C#

Vous pouvez lire et écrire des compteurs de performance via le namespace `System.Diagnostics` :

```csharp
using System;
using System.Diagnostics;
using System.Threading;

class Program
{
    static void Main()
    {
        // Créer et configurer un compteur personnalisé
        if (!PerformanceCounterCategory.Exists("MonApplication"))
        {
            CounterCreationDataCollection compteurs = new CounterCreationDataCollection();

            compteurs.Add(new CounterCreationData(
                "Requêtes par seconde",
                "Nombre de requêtes traitées par seconde",
                PerformanceCounterType.RateOfCountsPerSecond32));

            compteurs.Add(new CounterCreationData(
                "Temps moyen de traitement",
                "Temps moyen de traitement d'une requête en ms",
                PerformanceCounterType.AverageTimer32));

            PerformanceCounterCategory.Create("MonApplication",
                "Compteurs pour mon application",
                PerformanceCounterCategoryType.SingleInstance,
                compteurs);
        }

        // Utiliser les compteurs
        using (PerformanceCounter requêtesCounter =
            new PerformanceCounter("MonApplication", "Requêtes par seconde", false))
        using (PerformanceCounter tempsCounter =
            new PerformanceCounter("MonApplication", "Temps moyen de traitement", false))
        {
            // Simuler des requêtes
            for (int i = 0; i < 100; i++)
            {
                Stopwatch sw = Stopwatch.StartNew();

                // Simuler le traitement d'une requête
                Thread.Sleep(new Random().Next(10, 100));

                sw.Stop();

                // Incrémenter le compteur de requêtes
                requêtesCounter.Increment();

                // Mettre à jour le temps moyen
                tempsCounter.IncrementBy(sw.ElapsedMilliseconds);

                Console.WriteLine($"Requête {i}: {sw.ElapsedMilliseconds} ms");
            }
        }
    }
}
```

> **Note** : La création de compteurs personnalisés nécessite généralement des droits d'administrateur.

### Surveillance des compteurs avec Performance Monitor (perfmon.exe)

Windows inclut un outil puissant pour surveiller les compteurs de performance :

1. Ouvrez Performance Monitor (tapez `perfmon` dans la recherche Windows)
2. Cliquez sur le bouton + pour ajouter des compteurs
3. Sélectionnez la catégorie .NET CLR Memory
4. Ajoutez les compteurs pertinents (ex: % Time in GC, # Bytes in all Heaps)
5. Observez les tendances pendant que votre application s'exécute

![Performance Monitor](https://i.imgur.com/example4.png)

### Compteurs de performance essentiels pour les applications .NET

#### Mémoire

- **# Bytes in all Heaps** : Taille totale des tas managés
- **% Time in GC** : Pourcentage de temps passé dans le Garbage Collector
- **# Gen 0/1/2 Collections** : Nombre de collections par génération
- **Large Object Heap Size** : Taille du tas pour les objets volumineux

#### CPU

- **% Processor Time** : Utilisation CPU globale
- **% User Time** : Temps CPU passé en mode utilisateur (votre code)
- **% Privileged Time** : Temps CPU passé en mode noyau (appels système)

#### Threads

- **# of current logical Threads** : Nombre de threads logiques
- **# of current physical Threads** : Nombre de threads physiques
- **Contention Rate / sec** : Taux de contentions de verrous

#### ASP.NET (pour applications web)

- **Requests/Sec** : Nombre de requêtes par seconde
- **Request Execution Time** : Temps d'exécution des requêtes
- **Requests in Application Queue** : Requêtes en attente

### Surveillance continue avec Application Insights

Pour une surveillance en production, envisagez d'utiliser Azure Application Insights qui collecte automatiquement de nombreuses métriques :

```csharp
// Dans Program.cs ou Startup.cs
services.AddApplicationInsightsTelemetry(Configuration["ApplicationInsights:InstrumentationKey"]);
```

Application Insights offre :
- Tableaux de bord de performance en temps réel
- Alertes sur les seuils de performance
- Corrélation des métriques avec les exceptions
- Suivi des requêtes de bout en bout

### Outils de ligne de commande pour les compteurs de performance

#### dotnet-counters

Un outil moderne pour surveiller les compteurs .NET Core :

```bash
# Installation
dotnet tool install --global dotnet-counters

# Liste des processus .NET disponibles
dotnet-counters ps

# Surveillance en temps réel des compteurs par défaut
dotnet-counters monitor --process-id <PID>

# Surveillance de compteurs spécifiques
dotnet-counters monitor --process-id <PID> System.Runtime[cpu-usage,working-set,gc-heap-size]

# Collecter les compteurs dans un fichier pour analyse ultérieure
dotnet-counters collect --process-id <PID> --output counter_output.json
```

#### PerfView

PerfView est un outil avancé pour collecter et analyser les compteurs de performance :

```bash
# Collecte de données ETW incluant les compteurs de performance
PerfView collect -GCCollectOnly -merge:true -zip:true

# Analyse des données collectées
PerfView run counter_output.etl
```

### Création d'un tableau de bord de performance personnalisé

Vous pouvez combiner ces techniques pour créer votre propre dashboard de monitoring :

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

public class PerformanceDashboard
{
    private readonly PerformanceCounter _cpuCounter;
    private readonly PerformanceCounter _memoryCounter;
    private readonly PerformanceCounter _gcCounter;
    private readonly PerformanceCounter _threadCounter;

    private readonly CancellationTokenSource _cts = new CancellationTokenSource();
    private Task _monitoringTask;

    public PerformanceDashboard(string processName)
    {
        _cpuCounter = new PerformanceCounter("Process", "% Processor Time", processName);
        _memoryCounter = new PerformanceCounter("Process", "Working Set", processName);
        _gcCounter = new PerformanceCounter(".NET CLR Memory", "% Time in GC", processName);
        _threadCounter = new PerformanceCounter(".NET CLR LocksAndThreads", "# of current logical Threads", processName);
    }

    public void StartMonitoring(int intervalMs = 1000)
    {
        _monitoringTask = Task.Run(async () =>
        {
            while (!_cts.Token.IsCancellationRequested)
            {
                var cpuUsage = _cpuCounter.NextValue() / Environment.ProcessorCount;
                var memoryUsageMb = _memoryCounter.NextValue() / 1024 / 1024;
                var gcTime = _gcCounter.NextValue();
                var threads = _threadCounter.NextValue();

                Console.Clear();
                Console.WriteLine($"=== Performance Dashboard ===");
                Console.WriteLine($"CPU Usage: {cpuUsage:F1}%");
                Console.WriteLine($"Memory: {memoryUsageMb:F1} MB");
                Console.WriteLine($"GC Time: {gcTime:F1}%");
                Console.WriteLine($"Threads: {threads}");
                Console.WriteLine($"Last Updated: {DateTime.Now}");

                try
                {
                    await Task.Delay(intervalMs, _cts.Token);
                }
                catch (TaskCanceledException)
                {
                    break;
                }
            }
        });
    }

    public void StopMonitoring()
    {
        _cts.Cancel();
        _monitoringTask.Wait(1000);

        _cpuCounter.Dispose();
        _memoryCounter.Dispose();
        _gcCounter.Dispose();
        _threadCounter.Dispose();
        _cts.Dispose();
    }
}

// Utilisation
public static void Main()
{
    var dashboard = new PerformanceDashboard(Process.GetCurrentProcess().ProcessName);
    dashboard.StartMonitoring();

    Console.WriteLine("Appuyez sur Entrée pour arrêter le monitoring...");
    Console.ReadLine();

    dashboard.StopMonitoring();
}
```

## Cas d'utilisation pratiques du profilage

Maintenant que nous avons couvert les outils et techniques essentiels, voyons comment les appliquer à des scénarios réels.

### Cas 1 : Optimisation d'une application lente

Imaginez que votre application WPF, ASP.NET ou console présente des temps de réponse lents. Voici comment procéder :

#### Étape 1 : Localiser le goulot d'étranglement

Utilisez le CPU Profiler de Visual Studio :

1. Lancez votre application via Debug > Performance Profiler > CPU Usage
2. Effectuez l'opération lente
3. Arrêtez le profilage
4. Analysez les résultats pour identifier les méthodes les plus coûteuses

#### Étape 2 : Analyser les méthodes problématiques

Une fois les méthodes coûteuses identifiées :

1. Examinez le code concerné
2. Utilisez BenchmarkDotNet pour mesurer précisément différentes approches
3. Identifiez les problèmes potentiels :
   - Algorithmes inefficaces (complexité O(n²) au lieu de O(n))
   - Allocations mémoire excessives
   - Opérations I/O bloquantes
   - Calculs redondants

#### Étape 3 : Optimiser et valider

1. Implémentez les optimisations
2. Vérifiez avec BenchmarkDotNet que les changements améliorent réellement les performances
3. Utilisez à nouveau le profileur pour confirmer que le goulot d'étranglement a été résolu

#### Exemple concret

```csharp
// Code original lent
public List<Commande> RechercherCommandes(List<Commande> commandes, DateTime début, DateTime fin)
{
    List<Commande> résultat = new List<Commande>();

    foreach (var commande in commandes)
    {
        if (commande.Date >= début && commande.Date <= fin)
        {
            résultat.Add(commande);
        }
    }

    return résultat;
}

// Version optimisée (après profilage)
public List<Commande> RechercherCommandesOptimisé(List<Commande> commandes, DateTime début, DateTime fin)
{
    // Pré-allouer la capacité pour éviter les redimensionnements
    var résultat = new List<Commande>(commandes.Count / 10);  // Estimation

    // Si les commandes sont triées par date, on peut optimiser davantage
    if (commandes.Count > 1000)  // Seuil arbitraire où le tri devient avantageux
    {
        var commandesTriées = commandes
            .OrderBy(c => c.Date)
            .SkipWhile(c => c.Date < début)
            .TakeWhile(c => c.Date <= fin)
            .ToList();

        return commandesTriées;
    }

    // Sinon, approche simple mais avec capacité préallouée
    foreach (var commande in commandes)
    {
        if (commande.Date >= début && commande.Date <= fin)
        {
            résultat.Add(commande);
        }
    }

    return résultat;
}
```

### Cas 2 : Diagnostic d'une fuite de mémoire

Supposons que votre application consomme de plus en plus de mémoire et finit par planter.

#### Étape 1 : Confirmer la fuite

Utilisez le Memory Profiler :

1. Lancez l'application via Debug > Performance Profiler > Memory Usage
2. Prenez un snapshot initial (snapshot 1)
3. Effectuez l'opération suspectée de causer la fuite
4. Déclenchez le Garbage Collector manuellement (GC.Collect())
5. Prenez un deuxième snapshot (snapshot 2)
6. Répétez l'opération suspectée
7. Déclenchez à nouveau le GC et prenez un troisième snapshot

#### Étape 2 : Analyser les snapshots

1. Comparez le snapshot 2 au snapshot 1
2. Comparez le snapshot 3 au snapshot 2
3. Identifiez les objets dont le nombre augmente constamment

#### Étape 3 : Trouver la source de la fuite

Pour chaque type d'objet qui s'accumule :

1. Dans la vue "Paths to Root" (Chemins vers la racine), examinez qui retient ces objets
2. Suivez les références jusqu'à identifier la cause de la rétention

#### Étape 4 : Corriger le problème

Selon la cause identifiée :

- Désabonnez-vous correctement des événements
- Limitez la taille des caches
- Utilisez des références faibles
- Corrigez les implémentations d'IDisposable
- Éliminez les références statiques inutiles

#### Exemple de correction de fuite mémoire

```csharp
// Code avec fuite
public class GestionnaireNotifications
{
    private static List<Abonné> _abonnés = new List<Abonné>();

    public void EnregistrerAbonné(Abonné abonné)
    {
        _abonnés.Add(abonné);  // Collection statique qui grandit sans limite
    }

    public void EnvoyerNotification(string message)
    {
        foreach (var abonné in _abonnés)
        {
            abonné.Notifier(message);
        }
    }
}

// Code corrigé
public class GestionnaireNotificationsSansLeak
{
    private static List<WeakReference<IAbonné>> _abonnés = new List<WeakReference<IAbonné>>();

    public void EnregistrerAbonné(IAbonné abonné)
    {
        // Utiliser une référence faible
        _abonnés.Add(new WeakReference<IAbonné>(abonné));
    }

    public void EnvoyerNotification(string message)
    {
        // Copier la liste pour éviter les modifications pendant l'itération
        var abonnésTemporaires = _abonnés.ToList();

        // Nouvelle liste pour garder uniquement les références valides
        var abonnésValides = new List<WeakReference<IAbonné>>();

        foreach (var référence in abonnésTemporaires)
        {
            if (référence.TryGetTarget(out var abonné))
            {
                abonné.Notifier(message);
                abonnésValides.Add(référence);
            }
            // Si TryGetTarget échoue, l'abonné a été collecté par le GC
        }

        // Remplacer la liste par celle ne contenant que les références valides
        lock (_abonnés)
        {
            _abonnés = abonnésValides;
        }
    }
}
```

### Cas 3 : Optimisation d'une application Web

Les applications web présentent des défis spécifiques en termes de performance.

#### Étape 1 : Déterminer ce qu'il faut optimiser

Utilisez Application Insights ou les compteurs ASP.NET pour identifier :
- Les endpoints les plus lents
- Les requêtes les plus fréquentes
- Les points avec la plus grande latence

#### Étape 2 : Profilez de manière ciblée

1. Utilisez les outils de développement du navigateur pour analyser le temps front-end
2. Utilisez les outils de profilage de Visual Studio pour le back-end
3. Utilisez SQL Server Profiler pour les problèmes de base de données

#### Étape 3 : Optimisez les problèmes identifiés

- **Problèmes de base de données** : Indexation, optimisation de requêtes
- **Problèmes d'algorithmes back-end** : Refactoring, mise en cache
- **Problèmes réseau** : Compression, regroupement de requêtes
- **Problèmes front-end** : Minification, lazy loading, optimisation d'images

#### Exemple d'amélioration de performances web

```csharp
// Contrôleur original (lent)
public class ProduitController : Controller
{
    private readonly ApplicationDbContext _context;

    public ProduitController(ApplicationDbContext context)
    {
        _context = context;
    }

    // Inefficace : charge tous les produits et toutes leurs propriétés
    public IActionResult Index()
    {
        var produits = _context.Produits
            .Include(p => p.Catégorie)
            .Include(p => p.Fournisseur)
            .Include(p => p.Évaluations)
            .ToList();

        return View(produits);
    }
}

// Contrôleur optimisé
public class ProduitController : Controller
{
    private readonly ApplicationDbContext _context;
    private readonly IMemoryCache _cache;

    public ProduitController(ApplicationDbContext context, IMemoryCache cache)
    {
        _context = context;
        _cache = cache;
    }

    // Optimisé : pagination, projection, mise en cache
    public IActionResult Index(int page = 1, int pageSize = 20)
    {
        string cacheKey = $"Produits_Page{page}_Size{pageSize}";

        if (!_cache.TryGetValue(cacheKey, out PaginatedList<ProduitViewModel> produits))
        {
            // Projeter uniquement les propriétés nécessaires
            var query = _context.Produits
                .AsNoTracking()  // Plus rapide pour les requêtes en lecture seule
                .Select(p => new ProduitViewModel
                {
                    Id = p.Id,
                    Nom = p.Nom,
                    Prix = p.Prix,
                    ImageUrl = p.ImageUrl,
                    CatégorieName = p.Catégorie.Nom,
                    NombreÉvaluations = p.Évaluations.Count,
                    NoteAverage = p.Évaluations.Any() ? p.Évaluations.Average(e => e.Note) : 0
                });

            // Paginer les résultats
            produits = PaginatedList<ProduitViewModel>.Create(
                query, page, pageSize);

            // Mettre en cache pour 1 minute
            _cache.Set(cacheKey, produits, TimeSpan.FromMinutes(1));
        }

        return View(produits);
    }
}
```

## Stratégies globales de profilage et d'optimisation

Pour conclure, voici quelques stratégies générales pour intégrer le profilage dans votre cycle de développement.

### Intégrer le profilage dans le cycle de développement

#### Profilage préventif

N'attendez pas que les problèmes surviennent. Intégrez le profilage tôt dans votre processus :

1. **Tests de performance automatisés** : Créez des tests qui vérifient les performances des opérations critiques
2. **Benchmarks de référence** : Établissez des mesures de référence pour les fonctionnalités clés
3. **Seuils de performance** : Définissez des seuils d'alerte pour la durée des opérations

```csharp
[Test]
public void RechercheClient_PourformaneMiniimale()
{
    // Préparer des données de test
    var clients = GénérerDonnéesTest(10000);

    // Mesurer le temps d'exécution
    Stopwatch sw = Stopwatch.StartNew();
    var résultat = _service.RechercherClients("Dupont");
    sw.Stop();

    // Vérifier que le temps reste sous le seuil
    Assert.Less(sw.ElapsedMilliseconds, 100, "La recherche de client doit prendre moins de 100ms");
}
```

#### Profilage continu

Intégrez le monitoring dans votre environnement de production :

1. **Application Insights** pour le monitoring d'applications Azure
2. **Health checks** pour vérifier régulièrement les performances
3. **Alertes** basées sur les seuils de performance
4. **Dashboards** pour visualiser les tendances de performance

### Concevoir pour des performances optimales

#### Règles générales d'optimisation

1. **Optimisez les algorithmes** avant d'optimiser le code
2. **Mesurez avant et après** chaque optimisation
3. **Concentrez-vous sur le code critique** (loi de Pareto : 20% du code consomme 80% des ressources)
4. **Équilibrez performance et lisibilité** du code

#### Modèles de conception pour de meilleures performances

- **Object Pooling** pour éviter des allocations répétées
- **Lazy Loading** pour charger les données uniquement quand nécessaire
- **Caching** pour éviter de recalculer des résultats identiques
- **Optimizations des I/O** (I/O asynchrone, bufferisation)

```csharp
// Exemple de pattern object pool
public class ObjectPool<T> where T : new()
{
    private readonly ConcurrentBag<T> _objets = new ConcurrentBag<T>();
    private readonly Func<T> _objectFactory;

    public ObjectPool(Func<T> objectFactory)
    {
        _objectFactory = objectFactory ?? (() => new T());
    }

    public T Get() => _objets.TryTake(out T item) ? item : _objectFactory();

    public void Return(T item) => _objets.Add(item);
}

// Utilisation
var byteArrayPool = new ObjectPool<byte[]>(() => new byte[4096]);

public void LireEtTraiterFichier(string fichier)
{
    byte[] buffer = byteArrayPool.Get();
    try
    {
        // Utiliser le buffer
        using (var stream = File.OpenRead(fichier))
        {
            // Traiter le fichier
        }
    }
    finally
    {
        // Retourner le buffer au pool
        byteArrayPool.Return(buffer);
    }
}
```

### Compromis et équilibres

L'optimisation implique souvent des compromis :

#### Mémoire vs CPU

- **Mémorisation** (memoization) : utilisez plus de mémoire pour économiser du CPU
- **Traitement par lots** : utilisez plus de CPU pour économiser de la mémoire
- **Recalcul vs stockage** : parfois il est moins coûteux de recalculer que de stocker

#### Performance vs maintenabilité

- **Code optimisé** est souvent plus complexe et difficile à maintenir
- **Algorithmes sophistiqués** nécessitent plus de tests et de documentation
- **Optimisations prématurées** peuvent compliquer le code sans bénéfice réel

### Checklist d'optimisation de performances

Utilisez cette checklist lorsque vous cherchez à optimiser une application :

1. **Définissez des objectifs mesurables** (ex : temps de réponse < 100ms)
2. **Mesurez la performance actuelle** avec des outils de profilage
3. **Identifiez les goulots d'étranglement** (pas plus de 3-5 à la fois)
4. **Formulez des hypothèses** sur les causes des problèmes
5. **Testez vos hypothèses** avec des outils de diagnostic
6. **Implémentez les optimisations** une par une
7. **Mesurez l'impact** de chaque optimisation
8. **Documentez** les changements et les résultats
9. **Répétez** jusqu'à atteindre vos objectifs de performance

## Conclusion

Le profilage et l'optimisation sont des compétences essentielles pour tout développeur C# sérieux. Bien que les outils modernes facilitent l'identification des problèmes, il faut toujours une analyse réfléchie et méthodique pour identifier les véritables causes des problèmes de performance.

Retenez ces principes fondamentaux :

1. **Mesurez, ne devinez pas** : utilisez des outils de profilage pour identifier les vrais problèmes
2. **Optimisez les points critiques** : concentrez vos efforts là où ils auront le plus d'impact
3. **Testez vos optimisations** : vérifiez que vos changements améliorent réellement les performances
4. **Équilibrez performance et lisibilité** : parfois, un code légèrement moins performant mais plus maintenable est préférable

En suivant ces principes et en utilisant les outils et techniques présentés dans ce chapitre, vous serez en mesure de créer des applications C# performantes et robustes.

### Ressources supplémentaires

Pour approfondir vos connaissances en profilage et optimisation :

- [Documentation Microsoft sur les outils de profilage](https://docs.microsoft.com/fr-fr/visualstudio/profiling/?view=vs-2022)
- [Site officiel de BenchmarkDotNet](https://benchmarkdotnet.org/)
- [Blog de Stephen Toub sur la performance .NET](https://devblogs.microsoft.com/dotnet/author/stoub/)
- [PerfView Tutorial](https://channel9.msdn.com/Series/PerfView-Tutorial)
- [Writing High-Performance .NET Code](https://www.writinghighperf.net/) de Ben Watson

⏭️ 17.6. [Techniques d'optimisation avancées](/17-performances-et-optimisation/17-6-techniques-doptimisation-avancees.md)
