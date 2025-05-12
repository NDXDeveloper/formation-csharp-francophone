# 15.5. Débogage avancé

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

Le débogage est une compétence fondamentale pour tout développeur. Au-delà des techniques de base comme les points d'arrêt simples, Visual Studio et autres environnements de développement .NET offrent des fonctionnalités avancées qui peuvent transformer votre approche du débogage. Ce chapitre vous guidera à travers ces techniques qui vous permettront de résoudre des problèmes complexes plus efficacement.

## 15.5.1. Points d'arrêt conditionnels

Les points d'arrêt simples interrompent l'exécution chaque fois qu'une ligne de code spécifique est atteinte. Cependant, dans les applications avec des boucles ou des appels fréquents à une même méthode, cette approche peut s'avérer inefficace. C'est là que les points d'arrêt conditionnels entrent en jeu.

### Création d'un point d'arrêt conditionnel

Dans Visual Studio, vous pouvez créer un point d'arrêt conditionnel de plusieurs façons :

1. **Méthode rapide** :
   - Placez un point d'arrêt normal (cliquez dans la marge à gauche ou appuyez sur F9)
   - Faites un clic droit sur le point d'arrêt
   - Sélectionnez "Conditions..."

2. **Via le menu Déboguer** :
   - Déboguer → Nouveau point d'arrêt → Point d'arrêt conditionnel

### Types de conditions

Vous pouvez définir différents types de conditions :

#### 1. Expression conditionnelle

Le point d'arrêt s'active uniquement lorsqu'une expression booléenne est évaluée à `true`.

```csharp
// Exemple : une boucle qui traite des milliers d'éléments
for (int i = 0; i < 10000; i++)
{
    ProcessItem(items[i]); // Point d'arrêt conditionnel: i == 500
}
```

Dans cet exemple, vous pouvez placer un point d'arrêt sur la ligne `ProcessItem` avec la condition `i == 500`. L'exécution s'arrêtera uniquement lorsque la 501e itération sera atteinte.

Exemples de conditions utiles :
- `customer.Name == "Microsoft"` - S'arrête uniquement pour un client spécifique
- `amount > 1000` - S'arrête uniquement pour les montants importants
- `list.Count == 0` - S'arrête lorsqu'une liste est vide
- `DateTime.Now.Hour >= 18` - S'arrête uniquement en soirée

#### 2. Nombre d'occurrences

Le point d'arrêt s'active uniquement après avoir été atteint un certain nombre de fois.

Par exemple, si vous avez une méthode appelée des milliers de fois et que vous suspectez un problème après environ 200 appels, vous pouvez configurer un point d'arrêt pour s'activer à la 200e occurrence.

#### 3. Filtre par thread ou processus

Pour les applications multi-threads, vous pouvez définir un point d'arrêt qui s'active uniquement pour un thread spécifique. Ceci est particulièrement utile pour déboguer des problèmes de concurrence.

### Exemple pratique

Imaginons une application qui traite un lot de commandes et qui échoue occasionnellement :

```csharp
public void TraiterCommandes(List<Commande> commandes)
{
    foreach (var commande in commandes)
    {
        try
        {
            TraiterCommande(commande);
        }
        catch (Exception ex)
        {
            Logger.Log($"Erreur lors du traitement de la commande {commande.Id}: {ex.Message}");
        }
    }
}

private void TraiterCommande(Commande commande)
{
    // Point d'arrêt conditionnel: commande.MontantTotal < 0
    CalculerTaxes(commande);
    MettreÀJourInventaire(commande);
    EnvoyerConfirmation(commande);
}
```

En plaçant un point d'arrêt conditionnel sur la première ligne de `TraiterCommande` avec la condition `commande.MontantTotal < 0`, vous intercepterez uniquement les commandes avec un montant négatif, ce qui pourrait être la source du problème.

### Points d'arrêt avec actions

Visual Studio permet également de définir des actions à exécuter lorsqu'un point d'arrêt est atteint, sans nécessairement interrompre l'exécution :

1. **Imprimer un message** : Affiche un message dans la fenêtre "Sortie" du débogueur
2. **Exécuter une macro** (dans certaines versions de Visual Studio)
3. **Continuer l'exécution** : Le programme n'est pas interrompu

C'est particulièrement utile pour surveiller l'état d'une variable sans interrompre constamment l'exécution.

```csharp
// Configuration : imprimer "Traitement de la commande {commande.Id} avec montant {commande.MontantTotal}"
// et continuer l'exécution
private void TraiterCommande(Commande commande)
{
    CalculerTaxes(commande);
    MettreÀJourInventaire(commande);
    EnvoyerConfirmation(commande);
}
```

## 15.5.2. Visualisation des données

Pendant le débogage, comprendre l'état de vos données est crucial. Visual Studio offre plusieurs outils pour visualiser efficacement les données.

### Fenêtres de surveillance (Watch)

Les fenêtres de surveillance sont l'outil le plus basique pour suivre des variables spécifiques :

1. **Fenêtre Espion** (Watch) : Pour suivre des variables spécifiques
   - Pour l'ouvrir : Déboguer → Fenêtres → Espion → Espion 1
   - Ajoutez des expressions à surveiller

2. **Fenêtre Variables locales** : Affiche toutes les variables dans la portée actuelle
   - Pour l'ouvrir : Déboguer → Fenêtres → Variables locales

3. **Fenêtre Automatique** (Autos) : Affiche automatiquement les variables utilisées dans les lignes autour du point d'exécution actuel
   - Pour l'ouvrir : Déboguer → Fenêtres → Automatique

### Visualiseurs personnalisés

Visual Studio inclut des visualiseurs spécialisés pour certains types de données :

1. **Visualiseur de texte** : Pour les chaînes longues
2. **Visualiseur XML** : Pour formater et explorer des documents XML
3. **Visualiseur JSON** : Pour les données JSON
4. **Visualiseur HTML** : Pour prévisualiser le HTML

Pour utiliser un visualiseur :
- Pendant le débogage, survolez une variable
- Cliquez sur l'icône de loupe qui apparaît
- Sélectionnez le visualiseur approprié

### DebuggerDisplay et DebuggerTypeProxy

Vous pouvez personnaliser l'affichage de vos classes dans le débogueur :

#### DebuggerDisplay

L'attribut `[DebuggerDisplay]` permet de contrôler comment une classe est affichée dans les fenêtres du débogueur :

```csharp
[DebuggerDisplay("Client {Nom} (ID: {Id})")]
public class Client
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Email { get; set; }
    public List<Commande> Commandes { get; set; }
    // Autres propriétés...
}
```

Au lieu d'afficher simplement `Client`, le débogueur montrera par exemple `Client Jean Dupont (ID: 42)`.

#### DebuggerTypeProxy

Pour les classes avec de nombreux champs ou des structures complexes, vous pouvez créer un proxy qui simplifie la visualisation des données importantes :

```csharp
[DebuggerTypeProxy(typeof(ClientDebugView))]
public class Client
{
    // Définition de la classe Client...
}

internal class ClientDebugView
{
    private Client _client;

    public ClientDebugView(Client client)
    {
        _client = client;
    }

    public string Identité => $"{_client.Nom} (ID: {_client.Id})";
    public int NombreDeCommandes => _client.Commandes?.Count ?? 0;
    public decimal MontantTotal => _client.Commandes?.Sum(c => c.MontantTotal) ?? 0m;
}
```

### Affichage des données dans des vues personnalisées

#### DataTips améliorés

En survolant une variable pendant le débogage, vous pouvez épingler la fenêtre qui apparaît pour la garder visible. Vous pouvez également la personnaliser pour afficher des informations spécifiques.

#### Visualisateurs de données complexes

Pour les collections et les structures de données plus complexes, Visual Studio propose des visualisateurs spécialisés :

1. **Visualiseur de collection** : Pour explorer des listes et tableaux
   - Accédez-y en cliquant sur l'icône de loupe à côté d'une collection

2. **Explorateur de données** : Pour les DataSets et DataTables
   - Particulièrement utile pour les données provenant de bases de données

3. **Visualiseur d'arborescence** : Pour explorer des structures hiérarchiques

#### Astuces pour mieux visualiser les données

1. **Utiliser les évaluations immédiates** :
   - Pendant le débogage, appuyez sur Ctrl+Alt+I
   - Saisissez une expression à évaluer (par exemple `clients.Where(c => c.Nom.StartsWith("A")).ToList()`)

2. **Utiliser la fenêtre Espion pour les expressions LINQ** :
   - Ajoutez des expressions comme `commandes.Where(c => c.MontantTotal > 1000).Count()`

3. **Utiliser la fenêtre Immédiat** :
   - Ouvrez-la via Déboguer → Fenêtres → Immédiat
   - Exécutez des commandes et évaluez des expressions

## 15.5.3. Débogage distant

Le débogage distant permet de déboguer une application qui s'exécute sur un autre ordinateur, ce qui est essentiel pour les problèmes qui n'apparaissent que dans des environnements spécifiques.

### Configuration du débogage distant

#### 1. Sur l'ordinateur distant (cible)

1. **Installer les outils de débogage distant** :
   - Téléchargez et installez les "Outils de débogage distant pour Visual Studio" correspondant à votre version
   - Ces outils sont disponibles dans le programme d'installation de Visual Studio ou sur le site de Microsoft

2. **Configurer le service de débogage** :
   - Exécutez l'outil "Moniteur de débogage distant" (msvsmon.exe)
   - Configurez les autorisations et les options de sécurité

3. **Considérations de sécurité** :
   - Le débogage distant ouvre des ports et permet l'accès à distance
   - Utilisez l'authentification et limitez les autorisations
   - Considérez l'utilisation d'un VPN pour les connexions via Internet

#### 2. Sur l'ordinateur de développement (source)

1. **Configurer le projet pour le débogage distant** :
   - Ouvrez les propriétés du projet
   - Accédez à l'onglet "Déboguer"
   - Sélectionnez "Utiliser l'ordinateur distant"
   - Saisissez le nom ou l'adresse IP de l'ordinateur cible

2. **Pour les applications web** :
   - Accédez aux paramètres "Web" du projet
   - Configurez l'URL et les options de débogage distant

### Débogage d'applications déployées

Pour les applications déjà déployées :

1. **Associer le débogueur à un processus distant** :
   - Déboguer → Attacher au processus...
   - Cochez "Afficher les processus de tous les utilisateurs" si nécessaire
   - Dans "Qualifier", saisissez le nom ou l'adresse IP de l'ordinateur distant
   - Sélectionnez "Attacher"

2. **Associer des fichiers sources** :
   - Si les fichiers sources ne sont pas automatiquement trouvés, configurez le mappage des sources
   - Déboguer → Options → Débogage → Symboles

### Débogage via des proxys et pare-feu

Si vous déboguez à travers un pare-feu d'entreprise :

1. **Configurer les règles de pare-feu** pour autoriser le trafic de débogage
2. **Utiliser un proxy ou un tunnel SSH** si nécessaire
3. **Configurer les paramètres de proxy dans Visual Studio**

### Astuces pour le débogage distant

1. **Préparer les symboles de débogage** (fichiers .pdb) :
   - Assurez-vous que les fichiers PDB sont générés et disponibles
   - Utilisez un serveur de symboles si nécessaire

2. **Utiliser des journaux détaillés** :
   - Activez la journalisation détaillée sur l'application cible
   - Les journaux peuvent vous aider à identifier où commencer le débogage

3. **Considérer les différences d'environnement** :
   - Versions de framework
   - Configuration système
   - Permissions utilisateur

## 15.5.4. Débogage de production

Le débogage en production nécessite des approches différentes car l'interruption du service n'est généralement pas acceptable.

### Journalisation avancée

Une bonne stratégie de journalisation est essentielle pour déboguer les problèmes en production :

1. **Niveaux de journalisation** :
   - Trace/Verbose : Informations très détaillées pour le diagnostic
   - Debug : Informations utiles pendant le développement
   - Information : Événements normaux de l'application
   - Warning : Situations potentiellement problématiques
   - Error : Erreurs qui n'empêchent pas l'application de fonctionner
   - Critical : Défaillances graves nécessitant une attention immédiate

2. **Frameworks de journalisation** :
   - Serilog : Journalisation structurée flexible
   - NLog : Hautement configurable
   - log4net : Framework mature avec de nombreuses options
   - Microsoft.Extensions.Logging : Journalisation intégrée pour .NET Core

```csharp
// Exemple avec Serilog
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day)
    .WriteTo.Seq("http://seq-server:5341/")
    .Enrich.WithProperty("Application", "MonApp")
    .Enrich.WithMachineName()
    .CreateLogger();

try
{
    Log.Information("Application démarrée");

    // Code de l'application...

    Log.Information("Application arrêtée normalement");
}
catch (Exception ex)
{
    Log.Fatal(ex, "L'application s'est arrêtée de façon inattendue");
}
finally
{
    Log.CloseAndFlush();
}
```

3. **Journalisation contextuelle** :
   - Incluez des identifiants de corrélation pour suivre les requêtes
   - Enrichissez les logs avec des métadonnées (utilisateur, machine, etc.)

```csharp
using (LogContext.PushProperty("UserId", userId))
using (LogContext.PushProperty("RequestId", requestId))
{
    Log.Information("Traitement de la demande {Action}", actionName);
    // Code...
}
```

### Fonctionnalités de diagnostic à la demande

Intégrez des fonctionnalités permettant d'activer temporairement des diagnostics avancés :

1. **Commutateurs de traces** :
   - Mécanismes pour activer/désactiver des traces détaillées
   - Configurables via les fichiers de configuration ou une API

2. **API de diagnostic** :
   - Points de terminaison administratifs pour obtenir des informations de diagnostic
   - Protégés par des autorisations strictes

```csharp
// Exemple d'API de diagnostic
[Authorize(Roles = "Administrator")]
[Route("api/diagnostic")]
public class DiagnosticController : Controller
{
    [HttpGet("health")]
    public IActionResult GetHealth()
    {
        return Ok(new {
            Status = "Healthy",
            DbConnections = DatabaseMetrics.ActiveConnections,
            MemoryUsage = Process.GetCurrentProcess().WorkingSet64 / (1024 * 1024),
            Uptime = (DateTime.Now - Process.GetCurrentProcess().StartTime).ToString()
        });
    }

    [HttpPost("trace/enable")]
    public IActionResult EnableDetailedTracing([FromQuery] int durationMinutes = 30)
    {
        DiagnosticManager.EnableDetailedTracing(TimeSpan.FromMinutes(durationMinutes));
        return Ok(new { Message = $"Detailed tracing enabled for {durationMinutes} minutes" });
    }
}
```

### Surveillance et métriques

Utilisez des outils de surveillance pour détecter et diagnostiquer les problèmes avant qu'ils ne deviennent critiques :

1. **Application Performance Monitoring (APM)** :
   - Azure Application Insights
   - New Relic
   - AppDynamics
   - Datadog

2. **Métriques personnalisées** :
   - Instrumentez le code pour collecter des métriques importantes
   - Surveillez les tendances pour détecter les anomalies

```csharp
// Exemple avec Application Insights
public async Task<IActionResult> ProcessOrder(Order order)
{
    var stopwatch = Stopwatch.StartNew();

    try
    {
        // Traitement de la commande...

        stopwatch.Stop();

        // Envoyer une métrique personnalisée
        _telemetryClient.TrackMetric("OrderProcessingTime", stopwatch.ElapsedMilliseconds);

        return Ok();
    }
    catch (Exception ex)
    {
        _telemetryClient.TrackException(ex);
        throw;
    }
}
```

### Stratégies de redémarrage et de récupération

Implémentez des stratégies pour récupérer automatiquement des erreurs :

1. **Circuit Breaker Pattern** :
   - Détecte les défaillances et empêche les opérations susceptibles d'échouer
   - Permet au système de récupérer

2. **Retry Pattern** :
   - Réessaye automatiquement les opérations qui échouent
   - Utilise des stratégies de back-off pour éviter de surcharger les ressources

```csharp
// Exemple avec Polly
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
    .WaitAndRetryAsync(3, retryAttempt =>
        TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));

var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(5, TimeSpan.FromMinutes(1));

var combinedPolicy = Policy.WrapAsync(retryPolicy, circuitBreakerPolicy);

// Utilisation
var response = await combinedPolicy.ExecuteAsync(async () =>
    await _httpClient.GetAsync("https://api.example.com/data"));
```

## 15.5.5. Memory dumps et analyse post-mortem

L'analyse des dump mémoire est une technique puissante pour diagnostiquer des problèmes difficiles à reproduire ou des crashes en production.

### Capture de dumps mémoire

Un dump mémoire est un instantané de l'état d'un processus à un moment donné. Il peut être créé de plusieurs façons :

#### 1. Capture manuelle avec Visual Studio

1. Déboguer → Attacher au processus...
2. Sélectionnez le processus
3. Déboguer → Enregistrer le dump sous...

#### 2. Capture automatique lors de crashes

Configurez Windows Error Reporting pour capturer des dumps :

1. **Utilisation du Registre Windows** :
   ```
   [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps]
   "DumpFolder"="C:\\CrashDumps"
   "DumpType"=dword:00000002
   ```

2. **Utilisation de procdump** (outil Sysinternals) :
   ```
   procdump -ma -e -t MonApplication.exe C:\CrashDumps
   ```

3. **Avec Azure Application Insights** :
   - Configurez la collecte de dumps dans le portail Azure

#### 3. Dumps sur déclencheurs spécifiques

Créez des dumps lorsque certaines conditions sont remplies :

```csharp
if (conditionProblématique)
{
    // Référencez Microsoft.Diagnostics.Runtime (package ClrMD)
    using (var dataTarget = DataTarget.CreateSnapshotAndAttach(Process.GetCurrentProcess().Id))
    {
        dataTarget.CreateDump("C:\\Dumps\\declenchement_specifique.dmp", DumpType.Full);
    }
}
```

### Types de dumps mémoire

1. **Mini-dump** : Contient uniquement les informations de base
2. **Dump avec informations sur le tas** : Inclut les données des objets .NET
3. **Dump complet** : Contient toute la mémoire du processus

Le choix dépend du problème à diagnostiquer et des contraintes de taille.

### Analyse des dumps avec Visual Studio

1. Ouvrez Visual Studio
2. Fichier → Ouvrir → Fichier
3. Sélectionnez le fichier .dmp
4. Visual Studio chargera le dump et affichera des informations de diagnostic

Dans l'analyse de dump, vous pouvez :
- Voir la pile d'appels au moment du crash
- Examiner les variables locales et les objets
- Analyser l'utilisation de la mémoire
- Voir les exceptions non gérées

### Analyse avec WinDbg

WinDbg est un débogueur puissant pour l'analyse approfondie :

1. **Installation** :
   - Installez "Outils de débogage pour Windows" depuis le SDK Windows

2. **Chargement d'un dump** :
   - Ouvrez WinDbg
   - File → Open Crash Dump
   - Naviguez jusqu'au fichier .dmp

3. **Commandes SOS utiles** :
   - `.loadby sos clr` - Charge l'extension SOS
   - `!threads` - Affiche tous les threads
   - `!clrstack` - Affiche la pile d'appels .NET
   - `!dumpheap -stat` - Statistiques sur les objets en mémoire
   - `!dumpobj [address]` - Affiche les détails d'un objet
   - `!gcroot [address]` - Trouve les références qui maintiennent un objet en vie

### Outil Performance Monitor et Diagnostics

Les outils de diagnostic de Visual Studio offrent des fonctionnalités avancées :

1. **Analyser → Profileur de performances** :
   - CPU Usage - Pour les problèmes de performance
   - Memory Usage - Pour les fuites mémoire
   - UI Responsiveness - Pour les applications UI

2. **Diagnostics de mémoire** :
   - Prend des instantanés de la mémoire
   - Compare les instantanés pour trouver les fuites

#### Exemple de workflow pour analyser une fuite mémoire

1. Démarrez l'application avec le profileur
2. Prenez un instantané de base
3. Effectuez l'opération suspectée de causer une fuite
4. Prenez un second instantané
5. Comparez les deux instantanés pour voir quels objets se sont accumulés

### Analyse d'un dump typique

Voici un processus typique pour analyser un dump suite à un crash :

1. **Identifier l'exception** :
   ```
   !analyze -v
   ```

2. **Examiner la pile d'appels** :
   ```
   !clrstack
   ```

3. **Vérifier les exceptions actives** :
   ```
   !pe
   ```

4. **Examiner les variables locales** :
   ```
   !clrstack -l
   ```

5. **Vérifier l'utilisation mémoire** :
   ```
   !dumpheap -stat
   ```

### Analyse automatisée

Pour les équipes traitant de nombreux dumps, l'automatisation peut être précieuse :

1. **Microsoft.Diagnostics.Runtime (ClrMD)** :
   - API pour écrire des outils d'analyse de dump personnalisés

2. **Scripts de débogage** :
   - Créez des scripts WinDbg pour automatiser l'analyse

3. **Outils commerciaux** :
   - RedGate ANTS Memory Profiler
   - JetBrains dotMemory
   - SciTech Memory Profiler

## Conclusion

Le débogage avancé est un art qui s'affine avec l'expérience. Les techniques présentées dans ce chapitre vous aideront à résoudre même les problèmes les plus complexes. Souvenez-vous que le débogage efficace est souvent une question de méthodologie et de patience plutôt que de connaissances techniques pures.

Pratiquez ces techniques dans des scénarios contrôlés avant d'en avoir besoin dans des situations critiques. Cela vous permettra de développer une intuition sur quand et comment utiliser chaque outil à votre disposition.

## Ressources complémentaires

- [Documentation de débogage Visual Studio](https://docs.microsoft.com/fr-fr/visualstudio/debugger/)
- [Guide d'utilisation de WinDbg](https://docs.microsoft.com/fr-fr/windows-hardware/drivers/debugger/getting-started-with-windbg)
- [Advanced .NET Debugging](https://www.amazon.fr/Advanced-NET-Debugging-Mario-Hewardt/dp/0321578899) par Mario Hewardt
- [.NET Memory Profiling Tools and Methods](https://michaelscodingspot.com/find-fix-and-avoid-memory-leaks-in-c-net-8-best-practices/) par Michael Shpilt

⏭️
