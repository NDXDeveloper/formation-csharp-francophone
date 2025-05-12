# 6.8. Async/Await et programmation asynchrone

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La programmation asynchrone est devenue un élément essentiel du développement en C#, permettant de créer des applications réactives et performantes. Le modèle basé sur les mots-clés `async` et `await` introduit dans C# 5.0 a considérablement simplifié l'écriture et la compréhension du code asynchrone.

## 6.8.1. Concepts de base

La programmation asynchrone en C# s'appuie sur plusieurs concepts clés qui permettent d'exécuter des opérations potentiellement longues sans bloquer le thread principal.

### Qu'est-ce que la programmation asynchrone ?

La programmation asynchrone permet d'effectuer des opérations potentiellement longues (comme les opérations d'E/S, les appels réseau ou les accès à la base de données) sans bloquer le thread d'exécution, améliorant ainsi la réactivité et l'efficacité des applications.

```textmate
// Exemple de fonction synchrone (bloquante)
public string TéléchargerDonnéesSynchrone(string url)
{
    // Cette opération bloque le thread jusqu'à ce que le téléchargement soit terminé
    using (var client = new WebClient())
    {
        return client.DownloadString(url);
    }
}

// Équivalent asynchrone (non bloquant)
public async Task<string> TéléchargerDonnéesAsync(string url)
{
    using (var client = new HttpClient())
    {
        // await libère le thread pendant que le téléchargement s'effectue
        return await client.GetStringAsync(url);
    }
}
```


### Les mots-clés async et await

- **`async`** : Modifie une méthode, une fonction lambda ou une méthode anonyme, indiquant qu'elle contient des opérations asynchrones et peut utiliser le mot-clé `await`.
- **`await`** : Suspend l'exécution de la méthode asynchrone jusqu'à ce que la tâche attendue soit terminée, sans bloquer le thread.

```textmate
// Méthode asynchrone simple
public async Task<int> CalculerAsync()
{
    Console.WriteLine($"Début de la méthode asynchrone sur le thread {Thread.CurrentThread.ManagedThreadId}");

    // Simulation d'une opération asynchrone
    await Task.Delay(1000);

    Console.WriteLine($"Après await sur le thread {Thread.CurrentThread.ManagedThreadId}");

    return 42;
}

// Utilisation
public async Task UtiliserAsyncAwait()
{
    Console.WriteLine($"Avant l'appel sur le thread {Thread.CurrentThread.ManagedThreadId}");

    int résultat = await CalculerAsync();

    Console.WriteLine($"Résultat: {résultat}, thread: {Thread.CurrentThread.ManagedThreadId}");
}
```


### Différence entre Task et Task&lt;T&gt;

- **`Task`** : Représente une opération asynchrone qui ne retourne pas de valeur.
- **`Task<T>`** : Représente une opération asynchrone qui retourne une valeur de type T.

```textmate
// Méthode asynchrone sans retour de valeur
public async Task EnregistrerJournalAsync(string message)
{
    await Task.Delay(100); // Simulation d'une opération d'E/S
    Console.WriteLine($"Journal: {message} à {DateTime.Now}");
}

// Méthode asynchrone avec retour de valeur
public async Task<int> CompterMotsAsync(string texte)
{
    await Task.Delay(100); // Simulation de traitement
    return texte.Split(new[] { ' ', '.', ',' }, StringSplitOptions.RemoveEmptyEntries).Length;
}

// Utilisation
public async Task DémontrerTypes()
{
    await EnregistrerJournalAsync("Démarrage");

    int nombreMots = await CompterMotsAsync("Ceci est une phrase de test.");

    Console.WriteLine($"Nombre de mots: {nombreMots}");
}
```


### Fonctionnement sous le capot

Sous le capot, le compilateur C# transforme les méthodes `async` en machines à états qui exécutent le code en plusieurs étapes, permettant au thread d'être libéré pendant l'attente.

```textmate
// Ce que vous écrivez:
public async Task<int> ExempleAsync()
{
    int a = 1;
    await Task.Delay(100);
    int b = 2;
    return a + b;
}

// Ce que le compilateur génère (simplifié):
// Une classe qui implémente une machine à états
// État 0: Début
// État 1: Après le premier await
// État 2: Fin
```


### Types de retour possibles pour les méthodes async

Une méthode marquée avec le mot-clé `async` peut retourner les types suivants :

- **`Task`** : Pour les méthodes asynchrones sans valeur de retour
- **`Task<T>`** : Pour les méthodes asynchrones qui retournent une valeur de type T
- **`void`** : Principalement pour les gestionnaires d'événements asynchrones (à éviter autrement)
- **`ValueTask`** et **`ValueTask<T>`** : Pour optimiser les performances dans certains scénarios (.NET 8)

```textmate
// Task
public async Task TâcheSansRetourAsync()
{
    await Task.Delay(100);
    Console.WriteLine("Terminé");
}

// Task<T>
public async Task<string> ObtenirMessageAsync()
{
    await Task.Delay(100);
    return "Message obtenu de manière asynchrone";
}

// async void (à éviter sauf pour les gestionnaires d'événements)
public async void GestionnaireÉvénementAsync(object sender, EventArgs e)
{
    try
    {
        await TâcheSansRetourAsync();
    }
    catch (Exception ex)
    {
        // Important de gérer les exceptions dans les méthodes async void
        Console.WriteLine($"Erreur: {ex.Message}");
    }
}

// ValueTask<T> (.NET 8)
public ValueTask<int> CalculerEfficacementAsync(int valeur)
{
    // Si le résultat est déjà disponible, retourne immédiatement sans allouer un Task
    if (valeur < 0)
    {
        return new ValueTask<int>(0);
    }

    // Sinon, effectue une opération asynchrone
    return new ValueTask<int>(CalculAsynchrone(valeur));
}

private async Task<int> CalculAsynchrone(int valeur)
{
    await Task.Delay(100);
    return valeur * 2;
}
```


### Comparaison des approches asynchrones historiques

En .NET Framework 4.7.2, plusieurs approches étaient utilisées :

```textmate
// .NET Framework 4.7.2 - Approches asynchrones

// 1. Modèle Begin/End (obsolète)
public void DémontrerBeginEnd()
{
    WebRequest requête = WebRequest.Create("https://exemple.com");

    // Appel asynchrone avec callback
    requête.BeginGetResponse(ar => {
        try
        {
            WebResponse réponse = requête.EndGetResponse(ar);
            using (StreamReader lecteur = new StreamReader(réponse.GetResponseStream()))
            {
                string contenu = lecteur.ReadToEnd();
                Console.WriteLine($"Contenu téléchargé: {contenu.Length} caractères");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur: {ex.Message}");
        }
    }, null);

    Console.WriteLine("Après l'appel asynchrone");
}

// 2. Event-based Asynchronous Pattern (EAP) (obsolète)
public void DémontrerEAP()
{
    WebClient client = new WebClient();

    // Gestionnaire d'événement pour la fin du téléchargement
    client.DownloadStringCompleted += (sender, e) => {
        if (e.Error != null)
        {
            Console.WriteLine($"Erreur: {e.Error.Message}");
            return;
        }

        Console.WriteLine($"Contenu téléchargé: {e.Result.Length} caractères");
    };

    // Démarrer l'opération asynchrone
    client.DownloadStringAsync(new Uri("https://exemple.com"));

    Console.WriteLine("Après l'appel asynchrone");
}

// 3. Task-based Asynchronous Pattern (TAP) avec continuation
public void DémontrerTAP()
{
    HttpClient client = new HttpClient();

    // Création d'une tâche et ajout d'une continuation
    Task<string> tâche = client.GetStringAsync("https://exemple.com");

    tâche.ContinueWith(t => {
        if (t.IsFaulted)
        {
            Console.WriteLine($"Erreur: {t.Exception.InnerException.Message}");
            return;
        }

        Console.WriteLine($"Contenu téléchargé: {t.Result.Length} caractères");
    });

    Console.WriteLine("Après l'appel asynchrone");
}

// 4. async/await (moderne)
public async Task DémontrerAsyncAwait()
{
    try
    {
        HttpClient client = new HttpClient();

        // Simplement et lisible
        string contenu = await client.GetStringAsync("https://exemple.com");

        Console.WriteLine($"Contenu téléchargé: {contenu.Length} caractères");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erreur: {ex.Message}");
    }

    Console.WriteLine("Après l'appel asynchrone");
}
```


### Comparaison avec .NET 8

.NET 8 offre des améliorations significatives pour la programmation asynchrone :

```textmate
// .NET 8 - Nouveautés pour async/await

// 1. ValueTask et ValueTask<T> pour réduire les allocations
public ValueTask<int> ObtenirConfigurationAsync(string clé, int valeurParDéfaut)
{
    // Cache en mémoire pour les configurations fréquemment utilisées
    if (_cacheConfiguration.TryGetValue(clé, out int valeur))
    {
        // Retourne immédiatement sans allouer un Task
        return new ValueTask<int>(valeur);
    }

    // Cas où une opération asynchrone est nécessaire
    return new ValueTask<int>(ChargerConfigurationDeLaBaseAsync(clé, valeurParDéfaut));
}

private async Task<int> ChargerConfigurationDeLaBaseAsync(string clé, int valeurParDéfaut)
{
    // Simulation d'une opération de base de données
    await Task.Delay(100);
    return valeurParDéfaut * 2;
}

// 2. Méthodes d'extension asynchrones pour IAsyncEnumerable
public async Task DémontrerAsyncEnumerable()
{
    await foreach (var élément in GénérerNombresAsync(10))
    {
        Console.WriteLine($"Nombre: {élément}");
    }
}

public async IAsyncEnumerable<int> GénérerNombresAsync(int max)
{
    for (int i = 0; i < max; i++)
    {
        // Simulation de traitement asynchrone
        await Task.Delay(100);
        yield return i;
    }
}

// 3. Méthodes asynchrones par défaut dans les interfaces
public interface IServiceUtilisateur
{
    // Méthode asynchrone dans l'interface
    Task<Utilisateur> ObtenirUtilisateurAsync(int id);

    // Méthode par défaut qui utilise la méthode asynchrone
    async Task<string> ObtenirNomUtilisateurAsync(int id)
    {
        var utilisateur = await ObtenirUtilisateurAsync(id);
        return utilisateur?.Nom ?? "Inconnu";
    }
}

// 4. ValueTask pooling pour réduire la pression sur le GC
public async Task DémontrerValueTaskPooling()
{
    using var client = new HttpClient();

    // Utilise ObjectPool pour réduire les allocations
    var réponse = await client.GetAsync("https://exemple.com");

    using var contenu = await réponse.Content.ReadAsStreamAsync();
    // Traiter le contenu...
}
```


## 6.8.2. Patterns asynchrones

Plusieurs patterns asynchrones sont couramment utilisés en C# pour résoudre différents types de problèmes.

### Pattern d'attente simultanée (Fan-out/fan-in)

Ce pattern consiste à démarrer plusieurs opérations asynchrones simultanément, puis à attendre qu'elles soient toutes terminées.

```textmate
// Attente simultanée de plusieurs tâches
public async Task<List<string>> TéléchargerPlusieursURLsAsync(List<string> urls)
{
    // Crée une liste de tâches à exécuter en parallèle
    List<Task<string>> tâches = new List<Task<string>>();
    using (HttpClient client = new HttpClient())
    {
        // Démarre toutes les tâches (fan-out)
        foreach (string url in urls)
        {
            tâches.Add(client.GetStringAsync(url));
        }

        // Attend que toutes les tâches soient terminées (fan-in)
        await Task.WhenAll(tâches);
    }

    // Récupère les résultats dans l'ordre des URLs
    return tâches.Select(t => t.Result).ToList();
}

// Utilisation
public async Task DémontrerFanOutFanIn()
{
    List<string> urls = new List<string>
    {
        "https://exemple.com/api/1",
        "https://exemple.com/api/2",
        "https://exemple.com/api/3"
    };

    List<string> résultats = await TéléchargerPlusieursURLsAsync(urls);

    for (int i = 0; i < urls.Count; i++)
    {
        Console.WriteLine($"URL: {urls[i]}, contenu: {résultats[i].Substring(0, Math.Min(50, résultats[i].Length))}...");
    }
}
```


### Utilisation de Task.WhenAny

Ce pattern attend la première tâche terminée parmi plusieurs tâches asynchrones :

```textmate
// Attendre la première tâche terminée
public async Task<string> ObtenirPremièreDonnéeDisponibleAsync(List<string> urls)
{
    // Crée une liste de tâches
    List<Task<string>> tâches = new List<Task<string>>();
    using (HttpClient client = new HttpClient())
    {
        // Démarre toutes les tâches
        foreach (string url in urls)
        {
            tâches.Add(client.GetStringAsync(url));
        }

        // Tâches restantes à surveiller
        List<Task<string>> tâchesRestantes = new List<Task<string>>(tâches);

        // Tant qu'il reste des tâches...
        while (tâchesRestantes.Count > 0)
        {
            // Attend la première tâche terminée
            Task<string> tâcheTerminée = await Task.WhenAny(tâchesRestantes);
            tâchesRestantes.Remove(tâcheTerminée);

            try
            {
                // Récupère le résultat de la tâche terminée
                return await tâcheTerminée;
            }
            catch (Exception)
            {
                // Si cette tâche a échoué, continue avec les autres
                if (tâchesRestantes.Count == 0)
                {
                    throw; // Plus de tâches, relance la dernière exception
                }
            }
        }
    }

    throw new InvalidOperationException("Toutes les URLs ont échoué");
}
```


### Pattern producteur-consommateur asynchrone

Ce pattern est utilisé lorsqu'un processus produit des données et un autre les consomme, potentiellement à des vitesses différentes :

```textmate
// Pattern producteur-consommateur avec BlockingCollection
public async Task DémontrerProducteurConsommateur()
{
    var collection = new BlockingCollection<int>(boundedCapacity: 10);

    // Tâche producteur
    Task producteur = Task.Run(async () =>
    {
        for (int i = 0; i < 20; i++)
        {
            await Task.Delay(100); // Simulation de production
            collection.Add(i);
            Console.WriteLine($"Produit: {i}");
        }

        // Indique la fin de la production
        collection.CompleteAdding();
    });

    // Tâche consommateur
    Task consommateur = Task.Run(async () =>
    {
        foreach (var item in collection.GetConsumingEnumerable())
        {
            Console.WriteLine($"Consommé: {item}");
            await Task.Delay(200); // Simulation de consommation plus lente
        }
    });

    // Attend que les deux tâches soient terminées
    await Task.WhenAll(producteur, consommateur);
}

// Version .NET 8 avec IAsyncEnumerable
public async Task DémontrerProducteurConsommateurModerne()
{
    // Producteur
    async IAsyncEnumerable<int> ProduireAsync(int nombre)
    {
        for (int i = 0; i < nombre; i++)
        {
            await Task.Delay(100); // Simulation de production
            Console.WriteLine($"Produit: {i}");
            yield return i;
        }
    }

    // Consommateur
    await foreach (var item in ProduireAsync(20))
    {
        Console.WriteLine($"Consommé: {item}");
        await Task.Delay(200); // Simulation de consommation plus lente
    }
}
```


### Pattern de tâche de continuation

Ce pattern enchaîne des opérations asynchrones :

```textmate
// Enchaînement de tâches avec ContinueWith
public Task<int> EnchaînerTâchesTraditionnel()
{
    return Task.Run(() => 10)
        .ContinueWith(t1 => t1.Result * 2)
        .ContinueWith(t2 => t2.Result + 5);
}

// Version moderne avec async/await
public async Task<int> EnchaînerTâchesModerne()
{
    int résultat = await Task.Run(() => 10);
    résultat = await Task.Run(() => résultat * 2);
    résultat = await Task.Run(() => résultat + 5);
    return résultat;
}
```


### Parallélisme limité

Ce pattern limite le nombre d'opérations asynchrones exécutées simultanément :

```textmate
// Exécution parallèle limitée d'opérations asynchrones
public async Task<List<string>> TéléchargerAvecLimitationAsync(List<string> urls, int parallélismeMax)
{
    List<string> résultats = new List<string>(new string[urls.Count]);

    // SemaphoreSlim pour limiter le nombre d'opérations simultanées
    using (SemaphoreSlim sémaphore = new SemaphoreSlim(parallélismeMax))
    {
        // Tâches à attendre
        List<Task> tâches = new List<Task>();

        for (int i = 0; i < urls.Count; i++)
        {
            int index = i; // Capture l'index pour le lambda

            // Attend qu'un slot soit disponible
            await sémaphore.WaitAsync();

            // Crée et démarre une tâche qui libère le sémaphore quand elle est terminée
            tâches.Add(Task.Run(async () =>
            {
                try
                {
                    using (HttpClient client = new HttpClient())
                    {
                        string contenu = await client.GetStringAsync(urls[index]);
                        résultats[index] = contenu;
                    }
                }
                finally
                {
                    // Libère le slot même en cas d'erreur
                    sémaphore.Release();
                }
            }));
        }

        // Attend que toutes les tâches soient terminées
        await Task.WhenAll(tâches);
    }

    return résultats;
}

// Utilisation
public async Task DémontrerParallélismeLimité()
{
    List<string> urls = new List<string>();
    for (int i = 1; i <= 20; i++)
    {
        urls.Add($"https://exemple.com/api/{i}");
    }

    Console.WriteLine("Téléchargement avec limitation de parallélisme à 5...");
    List<string> résultats = await TéléchargerAvecLimitationAsync(urls, 5);

    Console.WriteLine($"Téléchargement terminé. Nombre de résultats: {résultats.Count}");
}
```


### Étranglement asynchrone (Throttling)

Ce pattern limite le nombre d'opérations par unité de temps :

```textmate
// Classe d'étranglement asynchrone
public class ÉtrangleurAsynchrone
{
    private readonly int _limiteDemandes;
    private readonly TimeSpan _périodeTemps;
    private readonly Queue<DateTime> _demandes = new Queue<DateTime>();
    private readonly SemaphoreSlim _verrou = new SemaphoreSlim(1, 1);

    public ÉtrangleurAsynchrone(int limiteDemandes, TimeSpan périodeTemps)
    {
        _limiteDemandes = limiteDemandes;
        _périodeTemps = périodeTemps;
    }

    public async Task<T> ExécuterAsync<T>(Func<Task<T>> action)
    {
        await _verrou.WaitAsync();
        try
        {
            // Supprime les demandes anciennes
            while (_demandes.Count > 0 && DateTime.Now - _demandes.Peek() > _périodeTemps)
            {
                _demandes.Dequeue();
            }

            // Si la limite est atteinte, calcule le délai d'attente
            if (_demandes.Count >= _limiteDemandes)
            {
                DateTime plusAncienne = _demandes.Peek();
                TimeSpan attente = plusAncienne + _périodeTemps - DateTime.Now;

                if (attente > TimeSpan.Zero)
                {
                    await Task.Delay(attente);
                }

                // Nettoie à nouveau après l'attente
                while (_demandes.Count > 0 && DateTime.Now - _demandes.Peek() > _périodeTemps)
                {
                    _demandes.Dequeue();
                }
            }

            // Ajoute la demande actuelle
            _demandes.Enqueue(DateTime.Now);
        }
        finally
        {
            _verrou.Release();
        }

        // Exécute l'action une fois que nous avons vérifié les limites
        return await action();
    }
}

// Utilisation
public async Task DémontrerÉtranglement()
{
    // Limite: 5 demandes par seconde
    var étrangleur = new ÉtrangleurAsynchrone(5, TimeSpan.FromSeconds(1));

    using (HttpClient client = new HttpClient())
    {
        for (int i = 1; i <= 20; i++)
        {
            int index = i;
            await étrangleur.ExécuterAsync(async () =>
            {
                Console.WriteLine($"Démarrage de la requête {index} à {DateTime.Now:HH:mm:ss.fff}");
                string url = $"https://exemple.com/api/{index}";
                await client.GetStringAsync(url);
                return true;
            });
        }
    }
}
```


### Pattern de cache asynchrone

Ce pattern met en cache les résultats des opérations asynchrones pour éviter de les répéter :

```textmate
// Cache asynchrone
public class CacheAsynchrone<TClé, TValeur>
{
    private readonly Dictionary<TClé, Task<TValeur>> _cache = new Dictionary<TClé, Task<TValeur>>();
    private readonly SemaphoreSlim _verrou = new SemaphoreSlim(1, 1);

    public async Task<TValeur> ObtenirOuAjouterAsync(TClé clé, Func<TClé, Task<TValeur>> fournisseurValeur)
    {
        // Vérifier si déjà en cache (sans verrouillage)
        if (_cache.TryGetValue(clé, out Task<TValeur> tâcheCachée))
        {
            return await tâcheCachée;
        }

        // Acquérir le verrou
        await _verrou.WaitAsync();
        try
        {
            // Vérifier à nouveau avec le verrou (double-check locking)
            if (_cache.TryGetValue(clé, out tâcheCachée))
            {
                return await tâcheCachée;
            }

            // Créer une tâche pour obtenir la valeur
            Task<TValeur> tâche = fournisseurValeur(clé);

            // Stocker la tâche dans le cache
            _cache[clé] = tâche;

            return await tâche;
        }
        finally
        {
            _verrou.Release();
        }
    }
}

// Utilisation
public async Task DémontrerCacheAsynchrone()
{
    var cache = new CacheAsynchrone<string, string>();

    // Fonction simulant une opération coûteuse
    Func<string, Task<string>> opérationCoûteuse = async (clé) =>
    {
        Console.WriteLine($"Exécution de l'opération coûteuse pour '{clé}'");
        await Task.Delay(1000);
        return $"Résultat pour '{clé}'";
    };

    // Premier appel (non mis en cache)
    Console.WriteLine("Premier appel pour 'A':");
    string résultat1 = await cache.ObtenirOuAjouterAsync("A", opérationCoûteuse);
    Console.WriteLine($"Résultat: {résultat1}");

    // Deuxième appel (mis en cache)
    Console.WriteLine("\nDeuxième appel pour 'A':");
    string résultat2 = await cache.ObtenirOuAjouterAsync("A", opérationCoûteuse);
    Console.WriteLine($"Résultat: {résultat2}");

    // Appel pour une clé différente
    Console.WriteLine("\nPremier appel pour 'B':");
    string résultat3 = await cache.ObtenirOuAjouterAsync("B", opérationCoûteuse);
    Console.WriteLine($"Résultat: {résultat3}");
}
```


### Pattern d'interrogation asynchrone (Polling)

Ce pattern vérifie périodiquement l'état d'une opération asynchrone :

```textmate
// Interrogation asynchrone
public async Task<T> AttendreConditionAsync<T>(
    Func<Task<T>> opération,
    Func<T, bool> condition,
    TimeSpan intervalleInterrogation,
    TimeSpan délaiExpiration)
{
    DateTime expiration = DateTime.Now + délaiExpiration;

    while (DateTime.Now < expiration)
    {
        T résultat = await opération();

        if (condition(résultat))
        {
            return résultat;
        }

        await Task.Delay(intervalleInterrogation);
    }

    throw new TimeoutException($"Condition non remplie après {délaiExpiration.TotalSeconds} secondes");
}

// Exemple d'utilisation pour attendre qu'un service soit prêt
public async Task<bool> VérifierServiceAsync(string url)
{
    try
    {
        using (HttpClient client = new HttpClient())
        {
            client.Timeout = TimeSpan.FromSeconds(5);
            HttpResponseMessage réponse = await client.GetAsync(url);
            return réponse.IsSuccessStatusCode;
        }
    }
    catch
    {
        return false;
    }
}

// Utilisation
public async Task DémontrerInterrogation()
{
    try
    {
        bool serviceEnLigne = await AttendreConditionAsync(
            () => VérifierServiceAsync("https://exemple.com/status"),
            résultat => résultat == true,
            intervalleInterrogation: TimeSpan.FromSeconds(2),
            délaiExpiration: TimeSpan.FromSeconds(30)
        );

        Console.WriteLine("Le service est en ligne!");
    }
    catch (TimeoutException ex)
    {
        Console.WriteLine($"Timeout: {ex.Message}");
    }
}
```


### IAsyncEnumerable en .NET 8
.NET 8 introduit un nouveau pattern pour le streaming asynchrone de données :
``` csharp
// Production et consommation de flux de données asynchrone
public async Task DémontrerAsyncEnumerable()
{
    // Lecture d'un flux de données asynchrone
    await foreach (var nombre in GénérerNombresAsync(10, 100))
    {
        Console.WriteLine($"Reçu: {nombre}");
    }
}

// Producteur asynchrone
public async IAsyncEnumerable<int> GénérerNombresAsync(int quantité, int délaiMillisecondes)
{
    Random aléatoire = new Random();

    for (int i = 0; i < quantité; i++)
    {
        // Simulation d'un délai pour obtenir chaque élément
        await Task.Delay(délaiMillisecondes);

        int nombre = aléatoire.Next(1, 100);
        Console.WriteLine($"Génération: {nombre}");

        // Retourne l'élément dès qu'il est disponible
        yield return nombre;
    }
}

// Traitement par lot asynchrone
public async Task TraiterParLotsAsync()
{
    // Source de données
    IAsyncEnumerable<int> source = GénérerNombresAsync(100, 50);

    // Traitement par lots de 10
    int tailleLot = 10;
    List<int> lotCourant = new List<int>(tailleLot);

    await foreach (var item in source)
    {
        lotCourant.Add(item);

        // Traiter le lot quand il est plein
        if (lotCourant.Count >= tailleLot)
        {
            await TraiterLotAsync(lotCourant);
            lotCourant.Clear();
        }
    }

    // Traiter le dernier lot partiel si nécessaire
    if (lotCourant.Count > 0)
    {
        await TraiterLotAsync(lotCourant);
    }
}

private async Task TraiterLotAsync(List<int> lot)
{
    Console.WriteLine($"Traitement d'un lot de {lot.Count} éléments...");
    await Task.Delay(200); // Simulation de traitement
    Console.WriteLine($"Lot traité, somme: {lot.Sum()}");
}

// Filtrage et transformation avec LINQ to IAsyncEnumerable (.NET 8)
public async Task DémontrerLINQAsynchrone()
{
    var nombres = GénérerNombresAsync(20, 100);

    // Filtrage et transformation asynchrones
    var résultat = nombres
        .WhereAwait(async x =>
        {
            await Task.Delay(10); // Simulation d'une vérification asynchrone
            return x % 2 == 0;
        })
        .SelectAwait(async x =>
        {
            await Task.Delay(10); // Simulation d'une transformation asynchrone
            return x * x;
        });

    // Consommation du résultat
    await foreach (var item in résultat)
    {
        Console.WriteLine($"Nombre au carré: {item}");
    }
}
```
## 6.8.3. Gestion des exceptions
La gestion des exceptions dans le code asynchrone nécessite une attention particulière, car les erreurs peuvent se produire dans un contexte différent de celui où la méthode a été appelée.
### Try/Catch avec async/await
Le modèle async/await permet d'utiliser les blocs try/catch traditionnels :
``` csharp
public async Task DémontrerGestionExceptionsSimple()
{
    try
    {
        // Appel asynchrone qui peut lever une exception
        string résultat = await TéléchargerContenuAsync("https://site-inexistant.xyz");
        Console.WriteLine($"Contenu téléchargé: {résultat.Length} caractères");
    }
    catch (HttpRequestException ex)
    {
        // Gestion d'une exception spécifique
        Console.WriteLine($"Erreur de requête HTTP: {ex.Message}");
    }
    catch (Exception ex)
    {
        // Gestion d'une exception générique
        Console.WriteLine($"Erreur inattendue: {ex.Message}");
    }
    finally
    {
        // Nettoyage
        Console.WriteLine("Opération terminée, qu'elle ait réussi ou échoué.");
    }
}

private async Task<string> TéléchargerContenuAsync(string url)
{
    using (HttpClient client = new HttpClient())
    {
        return await client.GetStringAsync(url);
    }
}
```
### Exceptions dans les méthodes async void
Les exceptions dans les méthodes `async void` sont particulièrement dangereuses car elles ne peuvent pas être capturées par le contexte appelant :
``` csharp
// DANGEREUX: Les exceptions dans les méthodes async void ne peuvent pas être capturées normalement
public async void MéthodeAsyncVoidDangereuse()
{
    try
    {
        await Task.Delay(100);
        throw new InvalidOperationException("Erreur dans async void!");
    }
    catch (Exception ex)
    {
        // Cette exception est capturée ici, mais si elle ne l'était pas,
        // elle pourrait faire planter l'application
        Console.WriteLine($"Exception capturée dans async void: {ex.Message}");
    }
}

// Méthode standard pour tester
public void DémontrerAsyncVoidProblème()
{
    try
    {
        MéthodeAsyncVoidDangereuse();

        // L'exception de MéthodeAsyncVoidDangereuse NE PEUT PAS être capturée ici
        // car la méthode est async void
    }
    catch (Exception ex)
    {
        // Ce bloc ne capturera jamais l'exception de MéthodeAsyncVoidDangereuse
        Console.WriteLine($"Exception capturée (cela n'arrivera pas): {ex.Message}");
    }

    // Empêche la méthode de se terminer avant que l'async void ait une chance de s'exécuter
    Console.WriteLine("Appuyez sur une touche pour continuer...");
    Console.ReadKey();
}
```
### Exceptions avec Task.WhenAll
Lorsque plusieurs tâches sont exécutées en parallèle, la gestion des exceptions devient plus complexe :
``` csharp
public async Task DémontrerExceptionsMultiplesTâches()
{
    Task<string> tâche1 = TéléchargerAvecExceptionPossibleAsync("https://exemple.com/1");
    Task<string> tâche2 = TéléchargerAvecExceptionPossibleAsync("https://exemple.com/2");
    Task<string> tâche3 = TéléchargerAvecExceptionPossibleAsync("https://exemple.com/erreur");

    try
    {
        // Attend que toutes les tâches soient terminées
        string[] résultats = await Task.WhenAll(tâche1, tâche2, tâche3);

        // Si nous arrivons ici, aucune tâche n'a levé d'exception
        Console.WriteLine($"Toutes les tâches ont réussi. Nombre de résultats: {résultats.Length}");
    }
    catch (Exception ex)
    {
        // ATTENTION: Task.WhenAll ne capture que la première exception
        Console.WriteLine($"Une exception s'est produite: {ex.Message}");

        // Pour obtenir toutes les exceptions, il faut vérifier chaque tâche
        List<Exception> exceptions = new List<Exception>();

        if (tâche1.IsFaulted && tâche1.Exception != null)
            exceptions.Add(tâche1.Exception.InnerException);

        if (tâche2.IsFaulted && tâche2.Exception != null)
            exceptions.Add(tâche2.Exception.InnerException);

        if (tâche3.IsFaulted && tâche3.Exception != null)
            exceptions.Add(tâche3.Exception.InnerException);

        Console.WriteLine($"Nombre total d'exceptions: {exceptions.Count}");
        for (int i = 0; i < exceptions.Count; i++)
        {
            Console.WriteLine($"Exception {i+1}: {exceptions[i].Message}");
        }
    }
}

private async Task<string> TéléchargerAvecExceptionPossibleAsync(string url)
{
    // Simuler un échec pour certaines URLs
    if (url.Contains("erreur"))
    {
        await Task.Delay(200);
        throw new HttpRequestException($"Échec pour l'URL: {url}");
    }

    await Task.Delay(300);
    return $"Contenu pour {url}";
}
```
### AggregateException et Task.Exception
Les tâches encapsulent les exceptions dans une `AggregateException` :
``` csharp
public void DémontrerAggregateException()
{
    // Crée une tâche qui va échouer
    Task tâche = Task.Run(() =>
    {
        throw new InvalidOperationException("Erreur dans la tâche");
    });

    try
    {
        // Forcer l'attente de la tâche de manière synchrone (à éviter en pratique)
        tâche.Wait();
    }
    catch (AggregateException aex)
    {
        // AggregateException peut contenir plusieurs exceptions internes
        Console.WriteLine($"AggregateException capturée avec {aex.InnerExceptions.Count} exceptions internes");

        foreach (var exception in aex.InnerExceptions)
        {
            Console.WriteLine($"  - {exception.GetType().Name}: {exception.Message}");
        }

        // On peut aussi dérouler l'exception pour obtenir l'exception interne
        Console.WriteLine($"Exception déballée: {aex.Flatten().InnerException.Message}");
    }
}
```
### Gestion des exceptions avec .NET 8
.NET 8 introduit de nouvelles façons de gérer les exceptions dans les flux asynchrones :
``` csharp
// Gestion d'exceptions dans IAsyncEnumerable (.NET 8)
public async Task DémontrerGestionExceptionsAsyncEnumerable()
{
    try
    {
        await foreach (var item in GénérerAvecExceptionsAsync())
        {
            Console.WriteLine($"Élément: {item}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Exception capturée dans le flux asynchrone: {ex.Message}");
    }
}

public async IAsyncEnumerable<int> GénérerAvecExceptionsAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);

        if (i == 5)
        {
            throw new InvalidOperationException("Erreur au milieu du flux!");
        }

        yield return i;
    }
}

// Utilisation de ExceptionDispatchInfo pour préserver la pile d'appels
public async Task DémontrerExceptionDispatchInfo()
{
    ExceptionDispatchInfo capturedInfo = null;

    try
    {
        await Task.Run(() =>
        {
            try
            {
                // Méthode qui peut lever une exception
                throw new InvalidOperationException("Erreur dans la tâche avec pile d'appels");
            }
            catch (Exception ex)
            {
                // Capture l'exception avec sa pile d'appels
                capturedInfo = ExceptionDispatchInfo.Capture(ex);

                // On peut retourner normalement ou relancer une autre exception
                return;
            }
        });

        // Plus tard, on peut relancer l'exception avec sa pile d'appels originale
        if (capturedInfo != null)
        {
            Console.WriteLine("Relance de l'exception capturée...");
            capturedInfo.Throw();
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Exception relancée: {ex.Message}");
        Console.WriteLine($"Pile d'appels: {ex.StackTrace}");
    }
}
```
## 6.8.4. Annulation et timeout
La gestion de l'annulation est essentielle pour contrôler les opérations asynchrones de longue durée.
### Utilisation de CancellationToken
`CancellationToken` est le mécanisme standard pour annuler des opérations asynchrones :
``` csharp
public async Task DémontrerAnnulation()
{
    // Création d'une source d'annulation
    using (var source = new CancellationTokenSource())
    {
        // Obtention du jeton d'annulation
        CancellationToken token = source.Token;

        // Démarrer une tâche qui accepte l'annulation
        Task tâche = OpérationLongueAsync(token);

        // Simuler une demande d'annulation après un délai
        await Task.Delay(2000);
        Console.WriteLine("Demande d'annulation...");
        source.Cancel();

        try
        {
            await tâche;
            Console.WriteLine("Opération terminée avec succès (peu probable)");
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Opération annulée comme prévu");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur inattendue: {ex.Message}");
        }
    }
}

private async Task OpérationLongueAsync(CancellationToken token)
{
    Console.WriteLine("Opération longue démarrée...");

    for (int i = 0; i < 10; i++)
    {
        // Vérifier l'annulation avant chaque étape
        token.ThrowIfCancellationRequested();

        Console.WriteLine($"Étape {i+1}/10 en cours...");

        try
        {
            // Utiliser le jeton d'annulation avec Task.Delay
            await Task.Delay(500, token);
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Delay annulé");
            throw; // Relance l'exception pour indiquer l'annulation
        }
    }

    Console.WriteLine("Opération longue terminée avec succès");
}
```
### Timeout avec CancellationTokenSource
On peut implémenter un timeout en utilisant `CancellationTokenSource` :
``` csharp
public async Task DémontrerTimeout()
{
    try
    {
        // Défini un timeout de 3 secondes
        using (var timeoutSource = new CancellationTokenSource(TimeSpan.FromSeconds(3)))
        {
            // Exécute une opération avec timeout
            string résultat = await TéléchargerAvecTimeoutAsync("https://exemple.com/api/lent",
                                                              timeoutSource.Token);

            Console.WriteLine($"Résultat obtenu: {résultat}");
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("L'opération a dépassé le délai imparti (timeout)");
    }
}

private async Task<string> TéléchargerAvecTimeoutAsync(string url, CancellationToken token)
{
    using (var client = new HttpClient())
    {
        Console.WriteLine($"Téléchargement de {url} démarré...");

        // Simulation d'une opération lente
        await Task.Delay(5000, token); // Sera annulé si le token est annulé

        // Cette partie ne sera jamais atteinte si le timeout est dépassé
        return await client.GetStringAsync(url, token);
    }
}
```
### Combinaison de jetons d'annulation
On peut combiner plusieurs sources d'annulation :
``` csharp
public async Task DémontrerCombinaisonAnnulation()
{
    // Source d'annulation manuelle
    using (var sourceManuelle = new CancellationTokenSource())

    // Source d'annulation pour timeout
    using (var sourceTimeout = new CancellationTokenSource(TimeSpan.FromSeconds(5)))

    // Combinaison des deux sources
    using (var sourceCombinée = CancellationTokenSource.CreateLinkedTokenSource(
           sourceManuelle.Token, sourceTimeout.Token))
    {
        // Obtenir le jeton combiné
        CancellationToken tokenCombiné = sourceCombinée.Token;

        // Démarrer la tâche avec le jeton combiné
        Task tâche = OpérationLongueAsync(tokenCombiné);

        // Simuler une action utilisateur après 2 secondes
        bool annulationManuelle = await AttendreAnnulationUtilisateurAsync(2000);

        if (annulationManuelle)
        {
            Console.WriteLine("Annulation manuelle demandée");
            sourceManuelle.Cancel();
        }

        try
        {
            await tâche;
            Console.WriteLine("Opération terminée avec succès");
        }
        catch (OperationCanceledException)
        {
            // Déterminer la source de l'annulation
            if (sourceManuelle.IsCancellationRequested)
            {
                Console.WriteLine("Opération annulée manuellement");
            }
            else if (sourceTimeout.IsCancellationRequested)
            {
                Console.WriteLine("Opération annulée par timeout");
            }
            else
            {
                Console.WriteLine("Opération annulée (source indéterminée)");
            }
        }
    }
}

private async Task<bool> AttendreAnnulationUtilisateurAsync(int délaiSimulé)
{
    // Simuler une action utilisateur
    await Task.Delay(délaiSimulé);
    return true; // Simulation: l'utilisateur a demandé l'annulation
}
```
### Annulation propre avec cleanup
Il est important de nettoyer les ressources lors de l'annulation :
``` csharp
public async Task DémontrerAnnulationAvecNettoyage()
{
    // Création d'une source d'annulation
    using (var source = new CancellationTokenSource())
    {
        // Définir un timeout de 3 secondes
        source.CancelAfter(TimeSpan.FromSeconds(3));

        try
        {
            await OpérationAvecRessourcesAsync(source.Token);
            Console.WriteLine("Opération terminée avec succès");
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Opération annulée");
        }
    }
}

private async Task OpérationAvecRessourcesAsync(CancellationToken token)
{
    // Ressource à nettoyer
    MemoryStream stream = null;

    try
    {
        Console.WriteLine("Allocation de ressources...");
        stream = new MemoryStream();

        for (int i = 0; i < 10; i++)
        {
            // Vérifier l'annulation
            token.ThrowIfCancellationRequested();

            Console.WriteLine($"Étape {i+1}/10 en cours...");
            await Task.Delay(500, token);

            // Utiliser la ressource
            byte[] données = Encoding.UTF8.GetBytes($"Données {i}\n");
            await stream.WriteAsync(données, 0, données.Length, token);
        }

        Console.WriteLine($"Opération terminée, {stream.Length} octets écrits");
    }
    finally
    {
        // Nettoyage de la ressource, même en cas d'annulation
        if (stream != null)
        {
            Console.WriteLine("Nettoyage des ressources...");
            await stream.DisposeAsync();
            Console.WriteLine("Ressources libérées");
        }
    }
}
```
### Annulation progressive
Dans certains cas, il est préférable d'arrêter progressivement une opération :
``` csharp
public async Task DémontrerAnnulationProgressive()
{
    using (var source = new CancellationTokenSource())
    {
        // Démarrer une opération longue
        Task tâche = UploadFichierVolumineuxAsync("fichier.dat", source.Token);

        // Après un délai, demander l'annulation
        await Task.Delay(2000);
        Console.WriteLine("Demande d'arrêt progressif...");
        source.Cancel();

        try
        {
            await tâche;
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Upload annulé");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur pendant l'upload: {ex.Message}");
        }
    }
}

private async Task UploadFichierVolumineuxAsync(string fichier, CancellationToken token)
{
    // Simuler un upload par morceaux
    int tailleTotale = 100;
    int morceauxEnvoyés = 0;

    Console.WriteLine($"Démarrage de l'upload de {fichier} ({tailleTotale} morceaux)");

    try
    {
        for (int i = 0; i < tailleTotale; i++)
        {
            // Vérifier l'annulation avant chaque morceau
            if (token.IsCancellationRequested)
            {
                Console.WriteLine("Annulation détectée, finalisation propre...");

                // Si déjà commencé, envoyer une notification de fin prématurée
                if (morceauxEnvoyés > 0)
                {
                    await EnvoyerNotificationFinPrématuréeAsync(fichier, morceauxEnvoyés);
                }

                // Puis signaler l'annulation
                token.ThrowIfCancellationRequested();
            }

            // Simuler l'envoi d'un morceau
            await Task.Delay(100);
            morceauxEnvoyés++;

            if (i % 10 == 0 || token.IsCancellationRequested)
            {
                Console.WriteLine($"Progression: {morceauxEnvoyés}/{tailleTotale} morceaux ({morceauxEnvoyés * 100 / tailleTotale}%)");
            }
        }

        Console.WriteLine("Upload terminé avec succès");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine($"Upload annulé après {morceauxEnvoyés}/{tailleTotale} morceaux");
        throw;
    }
}

private async Task EnvoyerNotificationFinPrématuréeAsync(string fichier, int morceauxEnvoyés)
{
    Console.WriteLine($"Envoi de notification de fin prématurée pour {fichier}");
    await Task.Delay(200); // Simuler l'envoi
    Console.WriteLine("Notification envoyée");
}
```
### Annulation coopérative en .NET 8
.NET 8 améliore la gestion de l'annulation pour les flux asynchrones :
``` csharp
// Annulation avec IAsyncEnumerable (.NET 8)
public async Task DémontrerAnnulationAsyncEnumerable()
{
    using var source = new CancellationTokenSource();

    // Annuler après 3 secondes
    source.CancelAfter(TimeSpan.FromSeconds(3));

    try
    {
        // Passer le token à la méthode await foreach
        await foreach (var item in GénérerFluxInfiniAsync().WithCancellation(source.Token))
        {
            Console.WriteLine($"Élément reçu: {item}");
            await Task.Delay(300);
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Flux annulé");
    }
}

public async IAsyncEnumerable<int> GénérerFluxInfiniAsync()
{
    int i = 0;
    while (true)
    {
        await Task.Delay(200);
        yield return i++;
    }
}
```
## 6.8.5. ConfigureAwait et contexte de synchronisation
Le comportement par défaut de `await` est de capturer le contexte de synchronisation actuel et d'y revenir après l'opération asynchrone, ce qui peut avoir un impact sur les performances.
### Comprendre le contexte de synchronisation
``` csharp
public async Task DémontrerContexteSynchronisation()
{
    // Dans une application UI, le contexte est généralement le thread UI
    Console.WriteLine($"Avant await - Thread: {Thread.CurrentThread.ManagedThreadId}, " +
                    $"Contexte: {SynchronizationContext.Current?.GetType().Name ?? "null"}");

    // await capture le contexte par défaut
    await Task.Delay(1000);

    // Après await, nous sommes de retour dans le contexte original
    Console.WriteLine($"Après await - Thread: {Thread.CurrentThread.ManagedThreadId}, " +
                    $"Contexte: {SynchronizationContext.Current?.GetType().Name ?? "null"}");
}
```
### Utilisation de ConfigureAwait(false)
`ConfigureAwait(false)` permet d'éviter de revenir au contexte de synchronisation d'origine :
``` csharp
public async Task DémontrerConfigureAwait()
{
    // Départ dans le contexte actuel
    Console.WriteLine($"Avant await - Thread: {Thread.CurrentThread.ManagedThreadId}");

    // Avec ConfigureAwait(true) - comportement par défaut, revient au contexte d'origine
    await Task.Delay(1000).ConfigureAwait(true);
    Console.WriteLine($"Après await(true) - Thread: {Thread.CurrentThread.ManagedThreadId}");

    // Avec ConfigureAwait(false) - ne revient pas nécessairement au contexte d'origine
    await Task.Delay(1000).ConfigureAwait(false);
    Console.WriteLine($"Après await(false) - Thread: {Thread.CurrentThread.ManagedThreadId}");
}

// Exemple concret dans une bibliothèque
public async Task<string> ObtenirDonnéesAPIAsync(string url)
{
    using (HttpClient client = new HttpClient())
    {
        // Dans une bibliothèque, utiliser ConfigureAwait(false) pour éviter de bloquer le thread UI
        string résultat = await client.GetStringAsync(url).ConfigureAwait(false);

        // Traitement qui ne nécessite pas le contexte original
        résultat = résultat.Trim();

        return résultat;
    }
}
```
### Quand utiliser ConfigureAwait(false)
Guide pour savoir quand utiliser `ConfigureAwait(false)` :
``` csharp
// BIBLIOTHÈQUE - Utiliser ConfigureAwait(false)
public class BiblioUtilitaire
{
    public async Task<int> CalculerValeurAsync(string entrée)
    {
        // Dans une bibliothèque, toujours utiliser ConfigureAwait(false)
        // pour ne pas bloquer le thread UI de l'application appelante
        await Task.Delay(100).ConfigureAwait(false);

        using (var client = new HttpClient())
        {
            string réponse = await client.GetStringAsync("https://api.example.com")
                                        .ConfigureAwait(false);

            return réponse.Length;
        }
    }
}

// APPLICATION UI - Ne pas utiliser ConfigureAwait(false) quand on a besoin du contexte UI
public class ExempleUIViewModel
{
    public async Task ChargerDonnéesAsync()
    {
        // Dans le code UI, ne pas utiliser ConfigureAwait(false)
        // car nous voulons revenir au thread UI pour mettre à jour l'interface
        await Task.Delay(100); // Sans ConfigureAwait(false)

        // Mise à jour de l'UI qui nécessite le thread UI
        MettreÀJourUI("Données chargées");
    }

    private void MettreÀJourUI(string message)
    {
        // Cette méthode nécessite d'être exécutée sur le thread UI
        Console.WriteLine($"UI mise à jour avec: {message}");
    }
}
```
### Deadlocks et ConfigureAwait
L'utilisation incorrecte de méthodes asynchrones peut entraîner des blocages :
``` csharp
// Exemple de deadlock potentiel
public void DémontrerDeadlockPotentiel()
{
    // Crée un contexte de synchronisation (simulé)
    var contexte = new SingleThreadSynchronizationContext();
    SynchronizationContext.SetSynchronizationContext(contexte);

    try
    {
        // RISQUE DE DEADLOCK: Bloquer en attendant une méthode async
        // qui essaie de revenir au même thread
        var résultat = MéthodeAsynchroneSansConfigureAwait().Result;

        Console.WriteLine($"Résultat: {résultat}");
    }
    catch (AggregateException ex)
    {
        Console.WriteLine($"Exception: {ex.InnerException.Message}");
    }
    finally
    {
        SynchronizationContext.SetSynchronizationContext(null);
    }
}

// Cette méthode tente de revenir au contexte de synchronisation d'origine
private async Task<int> MéthodeAsynchroneSansConfigureAwait()
{
    await Task.Delay(1000); // Sans ConfigureAwait(false)
    return 42;
}

// Version corrigée avec ConfigureAwait(false)
private async Task<int> MéthodeAsynchroneAvecConfigureAwait()
{
    await Task.Delay(1000).ConfigureAwait(false);
    return 42;
}

// Synchronization context simplifié pour la démonstration
public class SingleThreadSynchronizationContext : SynchronizationContext
{
    private readonly BlockingCollection<(SendOrPostCallback, object)> _queue =
        new BlockingCollection<(SendOrPostCallback, object)>();

    public override void Post(SendOrPostCallback d, object state)
    {
        if (d == null) throw new ArgumentNullException(nameof(d));
        _queue.Add((d, state));
    }

    public override void Send(SendOrPostCallback d, object state)
    {
        if (d == null) throw new ArgumentNullException(nameof(d));
        d(state);
    }
}
```
### ConfigureAwait en .NET 8
.NET 8 apporte des améliorations pour la gestion du contexte de synchronisation :
``` csharp
// .NET 8
public async Task DémontrerConfigureAwaitNET8()
{
    // Version .NET 8
```
### ConfigureAwait en .NET 8

.NET 8 apporte des améliorations pour la gestion du contexte de synchronisation :

```textmate
// .NET 8
public async Task DémontrerConfigureAwaitNET8()
{
    // Version .NET 8
    Console.WriteLine("Démarrage de l'opération asynchrone");

    // Utilisation de ValueTask pour optimiser les performances
    ValueTask<string> tâche = ObtenirRéponseRapideAsync();

    // ConfigureAwait avec options avancées
    string résultat = await tâche.ConfigureAwait(
        continueOnCapturedContext: false,
        TaskScheduler.Default);

    Console.WriteLine($"Résultat: {résultat}");

    // Nouvelles méthodes de tâches sans capture de contexte
    await Task.Yield(); // Avec capture de contexte
    await Task.CompletedTask; // Sans capture de contexte
}

private async ValueTask<string> ObtenirRéponseRapideAsync()
{
    // Vérifier si nous avons déjà le résultat en cache
    if (_cacheDisponible)
    {
        // Retourne immédiatement sans allocation de Task
        return _résultatCache;
    }

    // Sinon, exécute une opération asynchrone
    await Task.Delay(100).ConfigureAwait(false);

    return "Réponse du service";
}

// Meilleures performances avec IValueTaskSource (.NET 8)
public async Task DémontrerIValueTaskSource()
{
    // Utilisation d'une implémentation personnalisée de IValueTaskSource
    // pour optimiser les performances en réduisant les allocations
    using var source = new ManualResetValueTaskSourceCore<string>();

    // Créer une ValueTask à partir de la source
    ValueTask<string> tâche = new ValueTask<string>(source, 0);

    // Compléter la source après un délai
    _ = Task.Run(async () =>
    {
        await Task.Delay(100);
        source.SetResult("Résultat de l'opération");
    });

    // Attendre le résultat
    string résultat = await tâche;
    Console.WriteLine($"Résultat obtenu: {résultat}");
}
```


### Bonnes pratiques pour ConfigureAwait

```textmate
// Bonnes pratiques pour ConfigureAwait

// 1. Dans les bibliothèques, toujours utiliser ConfigureAwait(false)
public class BiblioUtilitaire
{
    public async Task<string> ChargementDonnéesAsync()
    {
        using (var client = new HttpClient())
        {
            // Dans une bibliothèque, utilisez toujours ConfigureAwait(false)
            string données = await client.GetStringAsync("https://api.exemple.com")
                                        .ConfigureAwait(false);

            // Traitement qui ne dépend pas du contexte
            return données.ToUpper();
        }
    }
}

// 2. Dans le code d'application UI, ne pas utiliser ConfigureAwait(false)
// quand on a besoin d'accéder à l'UI après l'opération asynchrone
public class ViewModelExemple
{
    public async Task ChargementAsync()
    {
        // Phase 1: Afficher un indicateur de chargement
        AfficherChargement(true);

        try
        {
            // Phase 2: Chargement des données (utilise une bibliothèque)
            var biblio = new BiblioUtilitaire();
            string données = await biblio.ChargementDonnéesAsync();

            // Ne pas utiliser ConfigureAwait(false) ici car nous avons
            // besoin de revenir au thread UI pour mettre à jour l'interface

            // Phase 3: Mise à jour de l'UI avec les données
            MettreÀJourUI(données);
        }
        finally
        {
            // Phase 4: Cacher l'indicateur de chargement
            AfficherChargement(false);
        }
    }

    private void AfficherChargement(bool visible)
    {
        // Cette méthode accède à l'UI
        Console.WriteLine($"Indicateur de chargement: {(visible ? "visible" : "caché")}");
    }

    private void MettreÀJourUI(string données)
    {
        // Cette méthode accède à l'UI
        Console.WriteLine($"UI mise à jour avec: {données.Substring(0, Math.Min(20, données.Length))}...");
    }
}

// 3. Être consistant dans une chaîne d'appels
public class ServiceExemple
{
    // Bonne pratique - consistance dans l'utilisation de ConfigureAwait
    public async Task<int> TraitementComplexeAsync()
    {
        // Si la méthode utilise ConfigureAwait(false), toutes les méthodes
        // appelées devraient suivre le même modèle

        int étape1 = await PremièreÉtapeAsync().ConfigureAwait(false);
        string étape2 = await DeuxièmeÉtapeAsync(étape1).ConfigureAwait(false);
        return await TroisièmeÉtapeAsync(étape2).ConfigureAwait(false);
    }

    private async Task<int> PremièreÉtapeAsync()
    {
        await Task.Delay(100).ConfigureAwait(false);
        return 42;
    }

    private async Task<string> DeuxièmeÉtapeAsync(int valeur)
    {
        await Task.Delay(100).ConfigureAwait(false);
        return valeur.ToString();
    }

    private async Task<int> TroisièmeÉtapeAsync(string valeur)
    {
        await Task.Delay(100).ConfigureAwait(false);
        return int.Parse(valeur) * 2;
    }
}
```


## 6.8.6. Performance et bonnes pratiques

Optimiser les performances des opérations asynchrones est essentiel pour créer des applications réactives et efficaces.

### Éviter les erreurs courantes

Plusieurs erreurs courantes peuvent affecter les performances des opérations asynchrones :

```textmate
// ERREUR: Await inutile pour les tâches déjà terminées
public async Task DémontrerErreursAwait()
{
    // BAD: Await sur une tâche déjà complétée
    await Task.CompletedTask; // Crée une machine à états inutile

    // GOOD: Vérifier si la tâche est déjà terminée
    Task tâche = ObtenirTâcheAsync();
    if (tâche.IsCompleted)
    {
        // Traitement synchrone si possible
        TraiterRésultat(tâche.Result);
    }
    else
    {
        // Await seulement si nécessaire
        TraiterRésultat(await tâche);
    }
}

// ERREUR: Blocage avec .Result ou .Wait()
public void DémontrerBlocage()
{
    // BAD: Bloquer avec .Result peut causer des deadlocks
    string résultat = ObtenirDonnéesAsync().Result; // À éviter!

    // BAD: Bloquer avec .Wait() peut causer des deadlocks
    ObtenirDonnéesAsync().Wait(); // À éviter!

    // GOOD: Utiliser une méthode asynchrone tout au long
    // public async Task MéthodePrincipaleAsync()
    // {
    //     string résultat = await ObtenirDonnéesAsync();
    // }
}

private async Task<string> ObtenirDonnéesAsync()
{
    await Task.Delay(100);
    return "Données";
}

// ERREUR: async void
// BAD: Les méthodes async void ne peuvent pas être attendues et les exceptions sont perdues
public async void MéthodeAsyncVoid() // À éviter sauf pour les gestionnaires d'événements
{
    await Task.Delay(100);
    throw new Exception("Cette exception sera perdue!");
}

// GOOD: Utiliser async Task
public async Task MéthodeAsyncTask()
{
    await Task.Delay(100);
    throw new Exception("Cette exception peut être capturée par le code appelant");
}
```


### Optimiser les allocations avec ValueTask

`ValueTask<T>` permet de réduire les allocations mémoire pour les opérations qui se terminent souvent immédiatement :

```textmate
// Optimisation avec ValueTask
public ValueTask<int> CalculerEfficacementAsync(int valeur)
{
    // Si le résultat peut être calculé immédiatement
    if (valeur < 0)
    {
        // Retourne directement le résultat sans allouer un Task
        return new ValueTask<int>(0);
    }

    // Sinon, effectue une opération asynchrone
    return new ValueTask<int>(CalculAsynchrone(valeur));
}

private async Task<int> CalculAsynchrone(int valeur)
{
    await Task.Delay(100);
    return valeur * 2;
}

// Utilisation de ValueTask pour un cache
public class ServiceCaché
{
    private readonly Dictionary<string, string> _cache = new Dictionary<string, string>();
    private readonly SemaphoreSlim _verrou = new SemaphoreSlim(1, 1);

    public ValueTask<string> ObtenirDonnéesAsync(string clé)
    {
        // Vérifier si la valeur est déjà en cache
        if (_cache.TryGetValue(clé, out string valeurCachée))
        {
            // Retourner immédiatement sans allocation
            return new ValueTask<string>(valeurCachée);
        }

        // Sinon, effectuer une opération asynchrone
        return new ValueTask<string>(ChargementEtCacheAsync(clé));
    }

    private async Task<string> ChargementEtCacheAsync(string clé)
    {
        // Vérifier à nouveau avec verrou (double-check locking)
        await _verrou.WaitAsync();
        try
        {
            if (_cache.TryGetValue(clé, out string valeurCachée))
            {
                return valeurCachée;
            }

            // Simuler une opération coûteuse
            await Task.Delay(500);
            string résultat = $"Valeur pour {clé} chargée à {DateTime.Now}";

            // Stocker dans le cache
            _cache[clé] = résultat;

            return résultat;
        }
        finally
        {
            _verrou.Release();
        }
    }
}
```


### Optimiser l'entrée-sortie asynchrone

Les opérations d'entrée-sortie sont un cas d'utilisation idéal pour la programmation asynchrone :

```textmate
// Optimisation E/S asynchrone
public async Task DémontrerOptimisationES()
{
    // BAD: Lecture synchrone du fichier
    void LectureSynchrone(string chemin)
    {
        using (var stream = new FileStream(chemin, FileMode.Open))
        using (var lecteur = new StreamReader(stream))
        {
            string contenu = lecteur.ReadToEnd(); // Bloque le thread
        }
    }

    // GOOD: Lecture asynchrone du fichier
    async Task LectureAsynchroneAsync(string chemin)
    {
        using (var stream = new FileStream(
            chemin,
            FileMode.Open,
            FileAccess.Read,
            FileShare.Read,
            bufferSize: 4096,
            useAsync: true)) // Important pour les performances!
        using (var lecteur = new StreamReader(stream))
        {
            string contenu = await lecteur.ReadToEndAsync();
        }
    }

    // BETTER: Utiliser les méthodes d'aide de File
    async Task LectureSimpleAsync(string chemin)
    {
        string contenu = await File.ReadAllTextAsync(chemin);
    }

    // Lecture par morceaux pour les gros fichiers
    async Task LectureGrossFichierAsync(string chemin)
    {
        byte[] tampon = new byte[8192];

        using (var stream = new FileStream(
            chemin,
            FileMode.Open,
            FileAccess.Read,
            FileShare.Read,
            bufferSize: tampon.Length,
            useAsync: true))
        {
            long tailleTotale = stream.Length;
            long octetsLus = 0;

            while (octetsLus < tailleTotale)
            {
                int lus = await stream.ReadAsync(tampon, 0, tampon.Length);
                if (lus == 0) break; // Fin du fichier

                // Traiter le morceau de données...
                octetsLus += lus;

                Console.WriteLine($"Progression: {octetsLus * 100 / tailleTotale}%");
            }
        }
    }

    // Exemple d'utilisation (non exécuté)
    Console.WriteLine("Démonstration des méthodes d'optimisation E/S");
}
```


### Parallélisme asynchrone

Exécuter plusieurs opérations asynchrones en parallèle peut améliorer les performances :

```textmate
// Parallélisme asynchrone
public async Task DémontrerParallélismeAsynchrone()
{
    Console.WriteLine("Démarrage des opérations");
    Stopwatch chrono = Stopwatch.StartNew();

    // BAD: Exécution séquentielle
    await ExécuterOpérationLongueAsync(1);
    await ExécuterOpérationLongueAsync(2);
    await ExécuterOpérationLongueAsync(3);

    Console.WriteLine($"Séquentiel terminé en {chrono.ElapsedMilliseconds}ms");
    chrono.Restart();

    // GOOD: Exécution parallèle
    Task tâche1 = ExécuterOpérationLongueAsync(1);
    Task tâche2 = ExécuterOpérationLongueAsync(2);
    Task tâche3 = ExécuterOpérationLongueAsync(3);

    await Task.WhenAll(tâche1, tâche2, tâche3);

    Console.WriteLine($"Parallèle terminé en {chrono.ElapsedMilliseconds}ms");
    chrono.Restart();

    // BETTER: Exécution parallèle avec contrôle d'erreurs
    var tâches = new List<Task>
    {
        ExécuterOpérationAvecRetryAsync(1),
        ExécuterOpérationAvecRetryAsync(2),
        ExécuterOpérationAvecRetryAsync(3)
    };

    try
    {
        await Task.WhenAll(tâches);
    }
    catch (Exception)
    {
        // Même si WhenAll lève une exception, vérifier chaque tâche individuellement
        foreach (var tâche in tâches)
        {
            if (tâche.IsFaulted)
            {
                Console.WriteLine($"Tâche échouée: {tâche.Exception.InnerException.Message}");
            }
            else if (tâche.IsCompleted)
            {
                Console.WriteLine("Tâche complétée avec succès");
            }
        }
    }

    Console.WriteLine($"Parallèle avec gestion d'erreurs terminé en {chrono.ElapsedMilliseconds}ms");
}

private async Task ExécuterOpérationLongueAsync(int id)
{
    Console.WriteLine($"Démarrage opération {id}");
    await Task.Delay(1000); // Simuler une opération longue
    Console.WriteLine($"Fin opération {id}");
}

private async Task ExécuterOpérationAvecRetryAsync(int id)
{
    const int maxRetry = 3;
    int tentative = 0;

    while (true)
    {
        try
        {
            tentative++;
            Console.WriteLine($"Opération {id} - tentative {tentative}");

            // Simuler une erreur aléatoire
            await Task.Delay(500);
            if (id == 2 && tentative == 1)
            {
                throw new Exception($"Erreur simulée pour l'opération {id}");
            }

            // Si on arrive ici, c'est un succès
            Console.WriteLine($"Opération {id} réussie à la tentative {tentative}");
            return;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur dans l'opération {id}: {ex.Message}");

            if (tentative >= maxRetry)
            {
                Console.WriteLine($"Nombre maximal de tentatives atteint pour l'opération {id}");
                throw;
            }

            // Attendre un peu avant de réessayer (avec back-off exponentiel)
            int délaiRetry = 200 * (int)Math.Pow(2, tentative - 1);
            await Task.Delay(délaiRetry);
        }
    }
}
```


### Optimisations en .NET 8

.NET 8 introduit de nouvelles optimisations pour les opérations asynchrones :

```textmate
// Optimisations .NET 8
public async Task DémontrerOptimisationsNET8()
{
    // 1. TaskCompletionSource avec options pour réduire les allocations
    var tcs = new TaskCompletionSource<string>(
        TaskCreationOptions.RunContinuationsAsynchronously);

    // Compléter la tâche de manière asynchrone
    _ = Task.Run(async () =>
    {
        await Task.Delay(100);
        tcs.SetResult("Résultat");
    });

    // Attendre le résultat
    string résultat = await tcs.Task;

    // 2. Stream asynchrone optimisé
    using (var memStream = new MemoryStream())
    {
        // Écriture efficace avec minimisation des copies
        byte[] données = new byte[1024];
        await memStream.WriteAsync(données);

        // Lecture efficace
        memStream.Position = 0;
        var tamponLecture = new Memory<byte>(new byte[1024]);
        await memStream.ReadAsync(tamponLecture);
    }

    // 3. Utilisation de WaitAsync pour timeout optimisé
    try
    {
        // Combinaison de Task et timeout sans créer une tâche supplémentaire
        string réponse = await ObtenirDonnéesExternesAsync()
            .WaitAsync(TimeSpan.FromSeconds(2));

        Console.WriteLine($"Réponse reçue: {réponse}");
    }
    catch (TimeoutException)
    {
        Console.WriteLine("L'opération a dépassé le délai imparti");
    }

    // 4. PipeWriter et PipeReader pour streaming efficace
    var pipe = new Pipe();

    // Écriture dans le pipe
    Task tâcheÉcriture = ÉcrireDansPipeAsync(pipe.Writer);

    // Lecture depuis le pipe
    Task tâcheLecture = LireDePipeAsync(pipe.Reader);

    // Attendre que les deux opérations soient terminées
    await Task.WhenAll(tâcheÉcriture, tâcheLecture);
}

private async Task<string> ObtenirDonnéesExternesAsync()
{
    // Simuler une opération réseau lente
    await Task.Delay(3000); // Plus long que le timeout
    return "Données du service externe";
}

private async Task ÉcrireDansPipeAsync(PipeWriter writer)
{
    // Simuler la production de données
    for (int i = 0; i < 10; i++)
    {
        // Réserver de la mémoire pour écrire
        Memory<byte> mémoire = writer.GetMemory(4);

        // Écrire un entier dans la mémoire
        BitConverter.TryWriteBytes(mémoire.Span, i);

        // Indiquer combien d'octets ont été écrits
        writer.Advance(4);

        // Rendre les données disponibles pour le lecteur
        await writer.FlushAsync();

        await Task.Delay(100); // Simuler un délai de production
    }

    // Indiquer qu'il n'y a plus de données à écrire
    await writer.CompleteAsync();
}

private async Task LireDePipeAsync(PipeReader reader)
{
    while (true)
    {
        // Lire depuis le pipe
        ReadResult résultat = await reader.ReadAsync();
        ReadOnlySequence<byte> buffer = résultat.Buffer;

        // Traiter toutes les données disponibles
        SequencePosition position = buffer.Start;
        SequencePosition fin = buffer.End;

        while (buffer.TryGet(ref position, out ReadOnlyMemory<byte> mémoire))
        {
            if (mémoire.Length >= 4)
            {
                int valeur = BitConverter.ToInt32(mémoire.Span);
                Console.WriteLine($"Valeur lue: {valeur}");
            }
        }

        // Indiquer que nous avons consommé toutes les données
        reader.AdvanceTo(fin);

        // Si le pipe est terminé, sortir de la boucle
        if (résultat.IsCompleted)
            break;
    }

    // Terminer la lecture
    await reader.CompleteAsync();
}
```


### Bonnes pratiques générales

Résumé des bonnes pratiques pour la programmation asynchrone :

```textmate
// Bonnes pratiques pour le code asynchrone

// 1. Suivre le modèle "async all the way"
public async Task DémontrerAsyncAllTheWay()
{
    // BAD: Mélanger synchrone et asynchrone
    string résultat = ObtenirDonnéesAsync().Result; // Risque de deadlock

    // GOOD: Rester asynchrone du début à la fin
    string résultatAsync = await ObtenirDonnéesAsync();
}

// 2. Nommer correctement les méthodes asynchrones
// GOOD: Terminer les méthodes asynchrones par "Async"
public async Task<string> ObtenirDonnéesAsync()
{
    await Task.Delay(100);
    return "Données";
}

// 3. Implémenter l'annulation quand c'est approprié
public async Task<string> RechercherAsync(string requête, CancellationToken token)
{
    token.ThrowIfCancellationRequested();

    // Simuler une recherche longue
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(100, token);
    }

    return $"Résultats pour {requête}";
}

// 4. Utiliser des mécanismes appropriés pour la synchronisation
public class GestionnaireRessource
{
    private readonly Dictionary<string, string> _ressources = new Dictionary<string, string>();
    private readonly SemaphoreSlim _verrou = new SemaphoreSlim(1, 1);

    public async Task<string> ObtenirRessourceAsync(string clé)
    {
        await _verrou.WaitAsync();
        try
        {
            return _ressources.TryGetValue(clé, out string valeur)
                ? valeur
                : "Non trouvé";
        }
        finally
        {
            _verrou.Release();
        }
    }
}

// 5. Éviter async void
// GOOD: Utiliser async Task pour les méthodes asynchrones
public async Task MéthodeSécuriséeAsync()
{
    await Task.Delay(100);
    throw new Exception("Cette exception peut être gérée");
}

// 6. Gérer correctement les exceptions
public async Task DémontrerGestionExceptionsAsync()
{
    try
    {
        await MéthodeSécuriséeAsync();
    }
    catch (Exception ex)
    {
        // Logger l'exception
        Console.WriteLine($"Exception gérée: {ex.Message}");

        // Décider si on la relance ou pas
        if (ex is ArgumentException)
        {
            throw;
        }
    }
}

// 7. Utiliser Task.WhenAll pour le parallélisme
public async Task<IEnumerable<string>> RechercherMultipleSourcesAsync(string terme)
{
    Task<string> recherche1 = RechercherSourceAsync(terme, "source1");
    Task<string> recherche2 = RechercherSourceAsync(terme, "source2");
    Task<string> recherche3 = RechercherSourceAsync(terme, "source3");

    // Attendre que toutes les recherches soient terminées
    await Task.WhenAll(recherche1, recherche2, recherche3);

    // Récupérer les résultats
    return new[] { recherche1.Result, recherche2.Result, recherche3.Result };
}

private async Task<string> RechercherSourceAsync(string terme, string source)
{
    await Task.Delay(new Random().Next(500, 1500));
    return $"Résultats de {source} pour '{terme}'";
}

// 8. Libérer les ressources correctement
public async Task DémontrerLibérationRessourcesAsync()
{
    // Utiliser using pour garantir la libération des ressources
    using (HttpClient client = new HttpClient())
    {
        try
        {
            string résultat = await client.GetStringAsync("https://exemple.com");
            Console.WriteLine($"Réponse: {résultat.Substring(0, 100)}...");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur: {ex.Message}");
        }
    } // La ressource est libérée ici automatiquement

    // Avec using déclaration (C# 8+)
    await using var stream = new MemoryStream();
    await stream.WriteAsync(Encoding.UTF8.GetBytes("Test"));
    // stream est libéré automatiquement à la fin du bloc
}

// 9. Éviter les tâches "orphelines"
public void DémontrerTâchesOrphelines()
{
    // BAD: Tâche orpheline
    Task.Run(async () =>
    {
        await Task.Delay(1000);
        throw new Exception("Cette exception sera perdue");
    });

    // GOOD: Stocker la référence de la tâche
    Task tâche = Task.Run(async () =>
    {
        await Task.Delay(1000);
        Console.WriteLine("Tâche terminée");
    });

    // Optionnellement, configurer un gestionnaire de continuation pour gérer les erreurs
    tâche.ContinueWith(t =>
    {
        if (t.IsFaulted)
        {
            Console.WriteLine($"La tâche a échoué: {t.Exception.InnerException.Message}");
        }
    }, TaskContinuationOptions.OnlyOnFaulted);
}

// 10. Optimiser pour les cas rapides avec ValueTask
public ValueTask<int> CalculerRapideAsync(int valeur)
{
    // Si on peut calculer immédiatement, éviter l'allocation d'un Task
    if (valeur < 10)
    {
        return new ValueTask<int>(valeur * 2);
    }

    // Sinon, effectuer une opération asynchrone
    return new ValueTask<int>(CalculerLentAsync(valeur));
}

private async Task<int> CalculerLentAsync(int valeur)
{
    await Task.Delay(100);
    return valeur * 2;
}
```


### Conclusion sur async/await et la programmation asynchrone

La programmation asynchrone avec async/await est un outil puissant qui permet de créer des applications réactives, performantes et évolutives. Depuis son introduction, elle a considérablement simplifié l'écriture de code asynchrone en C#.

En suivant les bonnes pratiques et en comprenant les subtilités du modèle, vous pouvez exploiter pleinement la puissance de la programmation asynchrone tout en évitant les pièges courants. Les améliorations continues de .NET, particulièrement avec .NET 8, continuent d'optimiser les performances et la facilité d'utilisation des opérations asynchrones.

Pour les applications modernes qui nécessitent une réactivité maximale, la maîtrise de async/await et des concepts associés est devenue une compétence essentielle pour tout développeur C#.

⏭️ 7. [Windows Forms (WinForms)](/07-windows-forms-winforms/README.md)
