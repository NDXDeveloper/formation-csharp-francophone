# 17.2. Garbage Collector en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le Garbage Collector (GC) est l'une des fonctionnalités les plus importantes de la plateforme .NET. Il vous libère de la gestion manuelle de la mémoire, mais comprendre son fonctionnement vous permettra d'écrire des applications plus performantes et d'éviter les problèmes de mémoire.

## 17.2.1. Fonctionnement du GC

Le Garbage Collector est un mécanisme automatique qui gère l'allocation et la libération de mémoire pour vos applications. Son objectif principal est d'identifier les objets qui ne sont plus utilisés et de libérer la mémoire qu'ils occupent.

### Principes fondamentaux

#### Allocation de mémoire

Lorsque vous créez un objet en C# (avec l'opérateur `new`), voici ce qui se passe :

1. Le runtime .NET alloue la mémoire nécessaire sur le tas (heap)
2. Le constructeur de l'objet est exécuté
3. Une référence à l'objet est retournée

```csharp
// Alloue de la mémoire sur le tas pour une nouvelle instance de Personne
Personne p = new Personne("Alice", 30);
```

#### Collecte des déchets

Le GC fonctionne sur un principe simple : si un objet n'est plus accessible par votre code (c'est-à-dire qu'il n'existe plus de référence vers cet objet), il peut être récupéré. Voici comment se déroule ce processus :

1. **Marquage** : Le GC identifie tous les objets accessibles (vivants)
2. **Nettoyage** : Les objets non marqués sont considérés comme des déchets et leur mémoire peut être récupérée
3. **Compactage** : La mémoire est réorganisée pour éliminer les espaces libres entre les objets

### Racines (Roots)

Le GC détermine quels objets sont toujours utilisés en partant des "racines" et en suivant toutes les références. Les racines comprennent :

- Variables locales et paramètres dans la pile d'exécution
- Variables statiques
- Objets finalisables qui ne sont pas encore finalisés
- Objets épinglés en mémoire (avec `fixed` ou `GCHandle`)
- Références fortes dans les handles COM

### Exemple visuel du processus

Voici une représentation simplifiée de comment le GC identifie les objets accessibles :

```
[Racines] → [Objet A] → [Objet B]
                      → [Objet C] → [Objet E]
            [Objet D]

// Dans cet exemple, les objets A, B, C et E sont accessibles
// à partir des racines et ne seront pas collectés.
// L'objet D n'est accessible par aucune racine et sera collecté.
```

### Déclenchement du GC

Le Garbage Collector s'exécute automatiquement dans plusieurs situations :

1. **Allocation de mémoire** : Quand le tas atteint un certain seuil
2. **Appel explicite** : Via `GC.Collect()` (à utiliser avec précaution)
3. **Faible mémoire système** : Quand le système d'exploitation signale une pression mémoire
4. **Application en arrière-plan** : Quand l'application devient inactive

```csharp
// Déclenchement explicite du GC (à éviter en général)
GC.Collect();

// Avec paramètres spécifiques (génération, mode)
GC.Collect(2, GCCollectionMode.Optimized);
```

> ⚠️ **Attention** : L'appel explicite à `GC.Collect()` est généralement déconseillé car le GC est conçu pour s'exécuter automatiquement de manière optimale. Utilisez-le uniquement dans des scénarios spécifiques et après avoir mesuré son impact.

### Modes de GC

Le GC .NET peut fonctionner selon différents modes :

1. **Mode Workstation** : Optimisé pour les applications de bureau, minimisant les pauses
2. **Mode Server** : Optimisé pour les serveurs, utilisant plusieurs threads pour la collecte
3. **Mode de collecte concurrente** : Permet à l'application de continuer pendant certaines phases de la collecte

Ces modes sont généralement configurés automatiquement selon l'environnement d'exécution, mais peuvent être ajustés via la configuration de l'application.

## 17.2.2. Générations

L'un des concepts clés du GC .NET est le système de générations. Il est basé sur deux observations empiriques :

1. Les objets récemment créés ont tendance à avoir une durée de vie plus courte
2. Les objets qui survivent longtemps ont tendance à continuer de survivre

### Les trois générations

Le GC .NET utilise trois générations pour classer les objets selon leur âge :

#### Génération 0 (Gen 0)

- Contient les objets nouvellement créés
- C'est la génération qui est collectée le plus fréquemment
- La collecte est très rapide car généralement, peu d'objets survivent

#### Génération 1 (Gen 1)

- Contient les objets qui ont survécu à au moins une collecte de Gen 0
- Sert de "tampon" entre les objets à courte durée de vie et les objets à longue durée de vie
- Collectée moins fréquemment que Gen 0

#### Génération 2 (Gen 2)

- Contient les objets qui ont survécu à au moins une collecte de Gen 1
- Objets à longue durée de vie
- Collectée le moins fréquemment
- Contient également les objets volumineux (large objects)

### Cycle de vie des objets à travers les générations

```
Création → Gen 0 → [Survit] → Gen 1 → [Survit] → Gen 2 → [Collecté éventuellement]
                 → [Collecté]       → [Collecté]
```

### Exemples de code pour visualiser les générations

```csharp
using System;

class Program
{
    static void Main()
    {
        // Affiche les statistiques initiales
        AfficherStatistiquesGC("Initial");

        // Alloue des objets qui iront en Gen 0
        AlloquerObjetsGen0();
        AfficherStatistiquesGC("Après allocation Gen 0");

        // Force une collection Gen 0
        GC.Collect(0);
        AfficherStatistiquesGC("Après collecte Gen 0");

        // Alloue plus d'objets
        AlloquerPlusObjets();
        AfficherStatistiquesGC("Après plus d'allocations");

        // Force une collection Gen 1
        GC.Collect(1);
        AfficherStatistiquesGC("Après collecte Gen 1");

        // Force une collection complète
        GC.Collect();
        AfficherStatistiquesGC("Après collecte complète");
    }

    static void AfficherStatistiquesGC(string étape)
    {
        Console.WriteLine($"\n--- {étape} ---");
        Console.WriteLine($"Gen 0: {GC.CollectionCount(0)} collections");
        Console.WriteLine($"Gen 1: {GC.CollectionCount(1)} collections");
        Console.WriteLine($"Gen 2: {GC.CollectionCount(2)} collections");
        Console.WriteLine($"Mémoire totale: {GC.GetTotalMemory(false) / 1024} KB");
    }

    static void AlloquerObjetsGen0()
    {
        // Crée 10 000 petits objets qui iront en Gen 0
        for (int i = 0; i < 10000; i++)
        {
            var obj = new byte[1000]; // 1KB chacun
        }
    }

    static object[] objetsRetenus = new object[100];

    static void AlloquerPlusObjets()
    {
        // Crée des objets et garde des références à certains
        // pour qu'ils survivent et passent en Gen 1
        for (int i = 0; i < 10000; i++)
        {
            var obj = new byte[1000]; // 1KB chacun

            // Garde des références à certains objets
            if (i % 100 == 0)
            {
                objetsRetenus[i / 100] = obj;
            }
        }
    }
}
```

### Grand Object Heap (GOH)

Le .NET GC traite différemment les objets volumineux (généralement plus de 85 000 octets) :

- Ces objets sont alloués directement sur le "Large Object Heap" (LOH)
- Ils sont considérés comme faisant partie de la génération 2
- Traditionnellement, le LOH n'était pas compacté (jusqu'à .NET Framework 4.5.1)
- Dans les versions récentes de .NET, le compactage du LOH est possible mais désactivé par défaut

L'allocation fréquente de grands objets peut entraîner une fragmentation de la mémoire et devrait être évitée lorsque c'est possible.

```csharp
// Allocation d'un objet volumineux (ira sur le LOH)
byte[] grandeTableau = new byte[100000];

// Pour forcer le compactage du LOH (à partir de .NET Framework 4.5.1)
GC.Collect(2, GCCollectionMode.Forced, true, true);
```

## 17.2.3. Configuration et tuning

Bien que le GC soit conçu pour fonctionner efficacement sans intervention, il existe plusieurs façons de le configurer pour des scénarios spécifiques.

### Configuration via le fichier de configuration

Dans un fichier `.csproj` ou `app.config`/`web.config`, vous pouvez configurer le comportement du GC :

```xml
<!-- Dans un fichier .csproj -->
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
  <RetainVMGarbageCollection>true</RetainVMGarbageCollection>
  <GCLOHThreshold>85000</GCLOHThreshold>
</PropertyGroup>

<!-- Ou dans app.config/web.config -->
<configuration>
  <runtime>
    <gcServer enabled="true"/>
    <gcConcurrent enabled="true"/>
  </runtime>
</configuration>
```

### Paramètres de configuration importants

#### 1. Mode Serveur vs Workstation

- **Workstation** (défaut pour les applications de bureau) : Optimisé pour la réactivité de l'interface utilisateur
- **Server** (défaut pour ASP.NET) : Optimisé pour le débit global, utilise plusieurs threads pour la collecte

```xml
<gcServer enabled="true"/>
```

#### 2. Collecte concurrente

Permet à l'application de continuer à s'exécuter pendant certaines phases de la collecte, réduisant les pauses.

```xml
<gcConcurrent enabled="true"/>
```

#### 3. Seuil LOH (Large Object Heap)

Définit la taille à partir de laquelle un objet est considéré comme "large" et alloué sur le LOH.

```xml
<GCLOHThreshold>85000</GCLOHThreshold>
```

### Configuration par code

Certains aspects du GC peuvent également être configurés par code :

```csharp
// Latence vs débit
GCSettings.LatencyMode = GCLatencyMode.Batch; // Priorise le débit
// ou
GCSettings.LatencyMode = GCLatencyMode.LowLatency; // Priorise la latence

// Compactage du LOH (à partir de .NET Framework 4.5.1)
GC.Collect(2, GCCollectionMode.Forced, true, true);

// Empêcher la finalisation
GC.SuppressFinalize(monObjet);

// Réactiver la finalisation
GC.ReRegisterForFinalize(monObjet);
```

### GCLatencyMode

Le `GCLatencyMode` permet de contrôler le compromis entre latence et débit :

- **Batch** : Priorité au débit, pauses plus longues mais moins fréquentes
- **Interactive** : Équilibre entre latence et débit (défaut)
- **LowLatency** : Priorité aux temps de réponse courts, utile pour les UI réactives
- **SustainedLowLatency** : Minimise encore plus les pauses, pour les applications critiques

```csharp
// Définir temporairement un mode de latence faible pour une opération critique
GCLatencyMode ancienMode = GCSettings.LatencyMode;
try
{
    GCSettings.LatencyMode = GCLatencyMode.LowLatency;
    // Exécuter l'opération critique...
}
finally
{
    // Restaurer le mode précédent
    GCSettings.LatencyMode = ancienMode;
}
```

### Bonnes pratiques pour optimiser le GC

1. **Évitez les allocations inutiles**
   - Réutilisez les objets quand c'est possible
   - Utilisez des structures (`struct`) pour les petits objets temporaires
   - Utilisez `StringBuilder` au lieu de concaténer des chaînes

2. **Attention aux collections**
   - Préallouez la capacité des collections (ex: `new List<int>(capacitéInitiale)`)
   - Privilégiez LINQ avec parcours différé sans créer des collections intermédiaires

3. **Gérez les grands objets avec précaution**
   - Évitez d'allouer/désallouer fréquemment de grands tableaux
   - Utilisez le pooling pour les grands objets temporaires

4. **N'appelez pas `GC.Collect()` sans raison**
   - Laissez le GC faire son travail automatiquement
   - Réservez son usage aux cas spécifiques (après avoir libéré de nombreuses ressources)

### Surveillance des performances du GC

Pour optimiser le GC, vous devez d'abord mesurer ses performances :

- **Compteurs de performance Windows** : Surveillez `.NET CLR Memory`
- **Outils de profilage** : Visual Studio Profiler, dotTrace, ANTS
- **Event Tracing for Windows (ETW)** : PerfView, Windows Performance Recorder

```csharp
// Exemple d'utilisation pour mesurer l'impact du GC
Stopwatch sw = Stopwatch.StartNew();

// Nombre de collectes avant
int gen0Avant = GC.CollectionCount(0);
int gen1Avant = GC.CollectionCount(1);
int gen2Avant = GC.CollectionCount(2);

// Exécuter le code à analyser
MonOpération();

// Nombre de collectes après
int gen0Après = GC.CollectionCount(0);
int gen1Après = GC.CollectionCount(1);
int gen2Après = GC.CollectionCount(2);

sw.Stop();

Console.WriteLine($"Temps écoulé: {sw.ElapsedMilliseconds}ms");
Console.WriteLine($"Collections Gen0: {gen0Après - gen0Avant}");
Console.WriteLine($"Collections Gen1: {gen1Après - gen1Avant}");
Console.WriteLine($"Collections Gen2: {gen2Après - gen2Avant}");
```

## 17.2.4. Gestion des ressources non managées

Le GC gère automatiquement la mémoire pour les objets managés, mais de nombreuses ressources système ne sont pas directement gérées par le runtime .NET :

- Fichiers
- Connexions réseau
- Connexions à des bases de données
- Handles Windows
- Mémoire non managée allouée avec P/Invoke

Ces ressources doivent être libérées explicitement pour éviter les fuites de ressources.

### Le pattern IDisposable

Le moyen standard de libérer des ressources non managées est d'implémenter l'interface `IDisposable` :

```csharp
public interface IDisposable
{
    void Dispose();
}
```

### Exemple d'implémentation de IDisposable

```csharp
public class ResourceManager : IDisposable
{
    // Ressource non managée (par exemple, un handle de fichier)
    private IntPtr _handle;
    // Ressource managée
    private ManagedResource _managedResource;
    // Flag pour éviter de disposer plusieurs fois
    private bool _disposed = false;

    public ResourceManager()
    {
        // Initialiser les ressources
        _handle = NativeMethod.OpenResource();
        _managedResource = new ManagedResource();
    }

    // Méthode publique Dispose de l'interface IDisposable
    public void Dispose()
    {
        // Appel à la méthode privée avec suppressFinalize=true
        Dispose(true);
        // Empêche le finalizer de s'exécuter pour cet objet
        GC.SuppressFinalize(this);
    }

    // Méthode protégée pour permettre aux classes dérivées de libérer leurs ressources
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Libérer les ressources managées
                if (_managedResource != null)
                {
                    _managedResource.Dispose();
                    _managedResource = null;
                }
            }

            // Libérer les ressources non managées
            if (_handle != IntPtr.Zero)
            {
                NativeMethod.CloseResource(_handle);
                _handle = IntPtr.Zero;
            }

            _disposed = true;
        }
    }

    // Finalizer
    ~ResourceManager()
    {
        // Appel à la méthode Dispose avec suppressFinalize=false
        Dispose(false);
    }

    // Autres méthodes de la classe...
    public void DoWork()
    {
        // Vérifier que l'objet n'a pas été disposé
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(ResourceManager));
        }

        // Faire le travail...
    }
}
```

### Utilisation avec l'instruction using

L'instruction `using` garantit que `Dispose()` est appelé même en cas d'exception :

```csharp
// Utilisation correcte de IDisposable avec using
using (var resource = new ResourceManager())
{
    resource.DoWork();
    // Dispose() est automatiquement appelé à la fin du bloc
}

// Équivalent en C# 8+ (using déclaration)
using var resource = new ResourceManager();
resource.DoWork();
// Dispose() est appelé à la fin de la portée
```

### SafeHandle

Pour les handles natifs, .NET fournit des classes comme `SafeHandle` qui simplifient la gestion des ressources :

```csharp
public class MySafeHandle : SafeHandleZeroOrMinusOneIsInvalid
{
    public MySafeHandle() : base(true) { }

    protected override bool ReleaseHandle()
    {
        // Libérer le handle natif
        return NativeMethod.CloseHandle(handle);
    }
}

// Utilisation
public class MyResourceManager
{
    private MySafeHandle _handle;

    public MyResourceManager()
    {
        _handle = NativeMethod.OpenResource();
    }

    // Plus besoin de finalizer ni de pattern IDisposable complet
    // car SafeHandle s'en charge
}
```

### Questions à se poser pour la gestion des ressources

1. **Ma classe utilise-t-elle des ressources non managées directement ?**
   - Si oui, implémentez le pattern IDisposable complet avec finalizer

2. **Ma classe utilise-t-elle des objets disposables sans les posséder ?**
   - Si oui, n'implémentez pas IDisposable, laissez le propriétaire s'en charger

3. **Ma classe possède-t-elle des objets disposables ?**
   - Si oui, implémentez IDisposable sans finalizer

## 17.2.5. Weak references et finalizers

### Finalizers

Les finalizers (ou destructeurs en C#) permettent d'exécuter du code avant qu'un objet ne soit récupéré par le GC. Ils sont utiles pour libérer des ressources non managées.

#### Déclaration d'un finalizer

```csharp
public class MyClass
{
    // Constructeur
    public MyClass()
    {
        Console.WriteLine("Objet créé");
    }

    // Finalizer (utilise la syntaxe du destructeur C#)
    ~MyClass()
    {
        Console.WriteLine("Finalizer appelé - l'objet va être collecté");
        // Libérer les ressources non managées ici
    }
}
```

#### Comment fonctionnent les finalizers

1. Lorsqu'un objet avec un finalizer devient inaccessible, il n'est pas immédiatement collecté
2. Il est placé dans une file d'attente spéciale (queue de finalisation)
3. Un thread spécial exécute les finalizers en arrière-plan
4. Après finalisation, l'objet devient éligible pour la collecte lors du prochain passage du GC

#### Impact des finalizers sur les performances

Les finalizers ont plusieurs inconvénients :

1. **Délai de collecte** : Les objets avec des finalizers survivent à au moins une collecte de plus
2. **Surcharge de mémoire** : Ces objets restent plus longtemps en mémoire
3. **Complexité** : Le timing de finalisation est non déterministe

Pour ces raisons, utilisez les finalizers uniquement lorsqu'ils sont absolument nécessaires, généralement pour libérer des ressources non managées.

```csharp
// Pattern recommandé : Combiner IDisposable et finalizer
public class ResourceOwner : IDisposable
{
    private IntPtr _nativeResource;
    private bool _disposed = false;

    public ResourceOwner()
    {
        _nativeResource = AllocateNativeResource();
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            // Libérer les ressources non managées
            if (_nativeResource != IntPtr.Zero)
            {
                FreeNativeResource(_nativeResource);
                _nativeResource = IntPtr.Zero;
            }

            _disposed = true;
        }
    }

    ~ResourceOwner()
    {
        // En cas d'oubli d'appel à Dispose()
        Dispose(false);
    }

    private IntPtr AllocateNativeResource()
    {
        // Code pour allouer une ressource native
        return new IntPtr(1);
    }

    private void FreeNativeResource(IntPtr handle)
    {
        // Code pour libérer une ressource native
        Console.WriteLine("Ressource native libérée");
    }
}
```

### Weak References (Références faibles)

Les références faibles permettent de référencer un objet sans empêcher sa collecte par le GC.

#### Types de références en .NET

1. **Références fortes** (normales) : Empêchent la collecte de l'objet
2. **Références faibles** : Permettent à l'objet d'être collecté
3. **Références faibles longues** : Maintiennent une référence jusqu'à la finalisation

#### Utilisations des références faibles

Les références faibles sont utiles pour :

- **Caches en mémoire** : Les objets peuvent être récupérés sous pression mémoire
- **Observer des objets** sans empêcher leur collecte
- **Éviter les cycles de références** qui empêcheraient la collecte

#### Exemple d'utilisation des références faibles

```csharp
using System;

class Program
{
    static void Main()
    {
        // Créer un objet volumineux
        var grandesDonnées = CréerGrandesDonnées();

        // Créer une référence faible vers l'objet
        WeakReference référenceFaible = new WeakReference(grandesDonnées);

        Console.WriteLine("Référence faible créée. Objet toujours accessible ? " +
                          référenceFaible.IsAlive);

        // Supprimer la référence forte
        grandesDonnées = null;

        // Forcer une collecte
        GC.Collect();
        GC.WaitForPendingFinalizers();

        // Vérifier si l'objet est toujours disponible
        Console.WriteLine("Après GC. Objet toujours accessible ? " +
                          référenceFaible.IsAlive);

        // Essayer de récupérer l'objet
        if (référenceFaible.IsAlive)
        {
            var données = référenceFaible.Target as byte[];
            Console.WriteLine($"Objet récupéré, taille: {données?.Length ?? 0}");
        }
        else
        {
            Console.WriteLine("Impossible de récupérer l'objet - il a été collecté");
        }
    }

    static byte[] CréerGrandesDonnées()
    {
        // Créer un tableau de 100 MB
        return new byte[100 * 1024 * 1024];
    }
}
```

#### WeakReference<T> générique

À partir de .NET 4.5, il existe une version générique de `WeakReference` qui est plus type-safe :

```csharp
// Création d'une référence faible typée
WeakReference<List<string>> refFaible = new WeakReference<List<string>>(maListe);

// Récupération de l'objet
List<string> listeRécupérée;
if (refFaible.TryGetTarget(out listeRécupérée))
{
    // L'objet est disponible
    Console.WriteLine($"Liste récupérée: {listeRécupérée.Count} éléments");
}
else
{
    // L'objet a été collecté
    Console.WriteLine("La liste a été collectée par le GC");
}
```

### Implémentation d'un cache avec des références faibles

Les références faibles sont particulièrement utiles pour créer des caches qui s'adaptent à la pression mémoire :

```csharp
public class CacheFaible<TKey, TValue> where TValue : class
{
    private Dictionary<TKey, WeakReference<TValue>> _cache =
        new Dictionary<TKey, WeakReference<TValue>>();

    public void Add(TKey key, TValue value)
    {
        _cache[key] = new WeakReference<TValue>(value);
    }

    public bool TryGetValue(TKey key, out TValue value)
    {
        value = null;
        WeakReference<TValue> référenceFaible;

        if (_cache.TryGetValue(key, out référenceFaible))
        {
            if (référenceFaible.TryGetTarget(out value))
            {
                return true;
            }
            else
            {
                // L'objet a été collecté, supprimer l'entrée
                _cache.Remove(key);
            }
        }

        return false;
    }

    public void NettoyerEntréesExpirées()
    {
        List<TKey> clésÀSupprimer = new List<TKey>();

        foreach (var paire in _cache)
        {
            TValue valeur;
            if (!paire.Value.TryGetTarget(out valeur))
            {
                clésÀSupprimer.Add(paire.Key);
            }
        }

        foreach (var clé in clésÀSupprimer)
        {
            _cache.Remove(clé);
        }
    }
}

// Utilisation
class Program
{
    static CacheFaible<string, byte[]> cache = new CacheFaible<string, byte[]>();

    static void Main()
    {
        // Ajouter des données au cache
        cache.Add("donnéesA", new byte[50 * 1024 * 1024]); // 50 MB
        cache.Add("donnéesB", new byte[50 * 1024 * 1024]); // 50 MB

        // Tenter de récupérer du cache
        byte[] données;
        if (cache.TryGetValue("donnéesA", out données))
        {
            Console.WriteLine("Données trouvées dans le cache!");
        }

        // Forcer une collection pour simuler une pression mémoire
        GC.Collect();
        GC.WaitForPendingFinalizers();

        // Nettoyer les entrées expirées
        cache.NettoyerEntréesExpirées();

        // Vérifier si les données sont encore disponibles
        if (cache.TryGetValue("donnéesA", out données))
        {
            Console.WriteLine("Données encore disponibles après GC");
        }
        else
        {
            Console.WriteLine("Données collectées par le GC");
        }
    }
}
```

### ConditionalWeakTable<TKey, TValue>

Le `ConditionalWeakTable` est une collection spéciale qui permet d'associer des données supplémentaires à un objet sans modifier sa classe, tout en respectant son cycle de vie :

```csharp
using System.Runtime.CompilerServices;

class Program
{
    // Table qui associe des métadonnées aux objets String
    static ConditionalWeakTable<string, StringMetadata> métadonnées =
        new ConditionalWeakTable<string, StringMetadata>();

    static void Main()
    {
        string message = "Bonjour le monde";

        // Ajouter des métadonnées à cette chaîne
        métadonnées.Add(message, new StringMetadata { DateCréation = DateTime.Now });

        // Récupérer les métadonnées
        StringMetadata md;
        if (métadonnées.TryGetValue(message, out md))
        {
            Console.WriteLine($"Message créé à: {md.DateCréation}");
        }

        // Quand 'message' sera collecté, les métadonnées associées
        // seront automatiquement collectées également
    }
}

class StringMetadata
{
    public DateTime DateCréation { get; set; }
    public int NombreAccès { get; set; }
}
```

#### Différences avec WeakReference

- `WeakReference` : Vous donne une référence faible vers un objet unique
- `ConditionalWeakTable` : Crée une association entre deux objets où la durée de vie de la valeur est liée à la durée de vie de la clé

#### Scénarios d'utilisation

- Ajout de propriétés à des objets existants sans modifier leur classe
- Stockage de métadonnées ou d'informations de cache liées à des objets
- Extension de types sealed ou types du framework

## Résumé et conseils pratiques

### Résumé des concepts clés du Garbage Collector

1. **Générations** : Le GC classe les objets en 3 générations selon leur âge pour optimiser les collectes
2. **Références** : Les références fortes empêchent la collecte, les références faibles permettent la collecte
3. **Finalizers** : Permettent d'exécuter du code avant la collecte d'un objet, mais ont un coût en performance
4. **IDisposable** : Pattern pour libérer explicitement les ressources, particulièrement les ressources non managées
5. **Configuration** : Le GC peut être configuré pour optimiser différents scénarios d'application

### Conseils pratiques

Voici un récapitulatif des meilleures pratiques pour travailler efficacement avec le GC :

1. **Libérer correctement les ressources non managées**
   - Implémentez toujours `IDisposable` pour les classes qui possèdent des ressources non managées
   - Utilisez l'instruction `using` pour garantir l'appel à `Dispose()`
   - Préférez les classes comme `SafeHandle` pour gérer les ressources natives

2. **Minimiser l'impact des finalizers**
   - Évitez les finalizers quand c'est possible
   - Implémentez `IDisposable` avec le modèle standard pour permettre la libération explicite
   - Appelez `GC.SuppressFinalize(this)` dans `Dispose()` pour éviter une finalisation inutile

3. **Réduire les allocations pour de meilleures performances**
   - Évitez la création fréquente d'objets temporaires, surtout dans les boucles
   - Utilisez des types valeur (`struct`) pour les petits objets à courte durée de vie
   - Réutilisez les objets quand c'est possible (pooling)

4. **Optimiser pour les générations**
   - Gardez à l'esprit que les objets à longue durée de vie finiront en génération 2
   - Évitez de créer et d'abandonner de grands objets fréquemment (ils vont dans le LOH)
   - Profitez du fait que Gen 0 est collecté très efficacement pour les objets à courte durée de vie

5. **Configuration du GC**
   - Utilisez le mode Serveur pour les applications à haut débit
   - Utilisez les modes de latence appropriés pour les applications interactives
   - Mesurez avant d'optimiser - les paramètres par défaut sont souvent les meilleurs

6. **Utilisation avancée**
   - Utilisez les références faibles pour les caches sensibles à la mémoire
   - Utilisez `ConditionalWeakTable` pour associer des données à des objets existants
   - Considérez les structures immuables pour réduire la pression sur le GC

### Exemple complet : gestionnaire de ressources robuste

Voici un exemple complet qui illustre les bonnes pratiques pour la gestion des ressources en C# :

```csharp
using System;
using System.IO;
using System.Runtime.InteropServices;

public class RobustResourceManager : IDisposable
{
    // Ressources managées
    private Stream _fileStream;
    private MemoryStream _memoryStream;

    // Ressources non managées
    private SafeFileHandle _fileHandle;
    private IntPtr _unmanagedBuffer;
    private bool _disposed = false;

    public RobustResourceManager(string filePath, int bufferSize)
    {
        // Allouer les ressources managées
        _fileStream = new FileStream(filePath, FileMode.OpenOrCreate);
        _memoryStream = new MemoryStream();

        // Allouer les ressources non managées
        _fileHandle = CreateFile(filePath, GENERIC_READ, 0, IntPtr.Zero,
                                OPEN_EXISTING, 0, IntPtr.Zero);
        _unmanagedBuffer = Marshal.AllocHGlobal(bufferSize);

        Console.WriteLine("Ressources allouées");
    }

    // Méthodes pour utiliser les ressources
    public void WriteData(byte[] data)
    {
        ThrowIfDisposed();
        _memoryStream.Write(data, 0, data.Length);
        _fileStream.Write(data, 0, data.Length);
    }

    public void ProcessUnmanagedBuffer()
    {
        ThrowIfDisposed();
        // Utiliser _unmanagedBuffer...
    }

    // Vérifier si l'objet a été disposé
    private void ThrowIfDisposed()
    {
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(RobustResourceManager));
        }
    }

    // Implémentation de IDisposable
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Libérer les ressources managées
                if (_fileStream != null)
                {
                    _fileStream.Dispose();
                    _fileStream = null;
                }

                if (_memoryStream != null)
                {
                    _memoryStream.Dispose();
                    _memoryStream = null;
                }
            }

            // Libérer les ressources non managées
            if (_fileHandle != null && !_fileHandle.IsInvalid)
            {
                _fileHandle.Dispose();
                _fileHandle = null;
            }

            if (_unmanagedBuffer != IntPtr.Zero)
            {
                Marshal.FreeHGlobal(_unmanagedBuffer);
                _unmanagedBuffer = IntPtr.Zero;
            }

            _disposed = true;

            Console.WriteLine("Ressources libérées");
        }
    }

    // Finalizer comme filet de sécurité
    ~RobustResourceManager()
    {
        Console.WriteLine("Finalizer appelé! (indique une fuite potentielle de ressources)");
        Dispose(false);
    }

    // P/Invoke pour CreateFile (exemple de ressource non managée)
    [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
    private static extern SafeFileHandle CreateFile(
        string filename,
        uint access,
        uint share,
        IntPtr securityAttributes,
        uint creationDisposition,
        uint flagsAndAttributes,
        IntPtr templateFile);

    private const uint GENERIC_READ = 0x80000000;
    private const uint OPEN_EXISTING = 3;
}

class Program
{
    static void Main()
    {
        Console.WriteLine("Démonstration de gestion robuste des ressources");

        // Utilisation correcte avec using
        Console.WriteLine("\nUtilisation avec 'using' (recommandée) :");
        using (var manager = new RobustResourceManager("test.dat", 1024))
        {
            manager.WriteData(new byte[] { 1, 2, 3, 4, 5 });
            manager.ProcessUnmanagedBuffer();
            // Dispose est automatiquement appelé à la fin du bloc
        }

        // Utilisation incorrecte sans using
        Console.WriteLine("\nUtilisation sans 'using' (non recommandée) :");
        {
            var manager = new RobustResourceManager("test2.dat", 1024);
            manager.WriteData(new byte[] { 6, 7, 8, 9, 10 });
            // Oups! On a oublié d'appeler Dispose()
            // Le finalizer agira comme filet de sécurité, mais tardivement
        }

        // Forcer une collecte complète pour démontrer la finalisation
        Console.WriteLine("\nForçage d'une collecte complète...");
        GC.Collect();
        GC.WaitForPendingFinalizers();

        Console.WriteLine("\nProgramme terminé");
    }
}
```

### Debugging des problèmes de mémoire

Voici quelques outils et techniques pour diagnostiquer les problèmes de mémoire :

1. **Visual Studio Diagnostic Tools**
   - Profiler mémoire
   - Analyse des allocations
   - Graphes d'objets

2. **Outils tiers**
   - JetBrains dotTrace/dotMemory
   - ANTS Memory Profiler
   - SciTech Memory Profiler

3. **Approches de diagnostic**
   - Prendre plusieurs "snapshots" de mémoire et comparer
   - Analyser les objets qui s'accumulent au fil du temps
   - Comparer les allocations après des opérations spécifiques

4. **Analyse avec PerfView**
   - Examen détaillé des allocations GC
   - Analyse de l'activité du GC
   - Identification des grandes allocations d'objets

### Tendances et évolutions du GC

Le Garbage Collector de .NET continue d'évoluer :

1. **GC de serveur amélioré**
   - Collecte plus parallèle et évolutive
   - Meilleure utilisation des architectures multi-cœurs

2. **Collecte à faible latence**
   - Modes de GC optimisés pour minimiser les pauses
   - "Background GC" amélioré

3. **Optimisations pour le cloud**
   - GC adaptif qui répond aux signaux du système d'exploitation
   - Collecte sensible aux environnements virtualisés

4. **Outils de diagnostic améliorés**
   - Meilleure visibilité dans l'activité du GC
   - Intégration avec le cloud pour l'analyse des performances

## Conclusion

Le Garbage Collector est un composant sophistiqué de la plateforme .NET qui automatise la gestion de la mémoire, réduisant la complexité du développement et évitant de nombreux types de bugs liés à la mémoire.

En comprenant son fonctionnement et en suivant les bonnes pratiques, vous pouvez écrire des applications C# qui utilisent efficacement la mémoire, avec des performances optimales et sans fuites de ressources.

Rappelez-vous que le GC est conçu pour fonctionner automatiquement dans la plupart des cas, mais que la gestion correcte des ressources non managées reste de votre responsabilité. L'implémentation du pattern IDisposable et l'utilisation de l'instruction `using` sont essentielles pour les objets qui possèdent de telles ressources.

Et enfin, lorsque vous travaillez avec le GC, suivez cette approche :

1. **Mesurer avant d'optimiser** - Assurez-vous que le GC est vraiment un goulot d'étranglement
2. **Comprendre le cycle de vie de vos objets** - Concevoir votre application en tenant compte des générations
3. **Libérer proprement les ressources rares** - Implémentez correctement IDisposable
4. **Utiliser les outils avancés au besoin** - Références faibles, pooling, etc.

En maîtrisant ces concepts, vous tirerez le meilleur parti du Garbage Collector tout en évitant ses pièges potentiels.

⏭️ 17.3. [Multithreading et parallélisme](/17-performances-et-optimisation/17-3-multithreading-et-parallelisme.md)
