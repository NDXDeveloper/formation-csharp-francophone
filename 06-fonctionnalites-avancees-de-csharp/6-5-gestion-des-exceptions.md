## 6.5.1. try-catch-finally

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le bloc try-catch-finally est la structure de base pour gérer les exceptions en C#.
### Structure de base
``` csharp
try
{
    // Code susceptible de générer une exception
    int résultat = 10 / 0; // Génère une DivideByZeroException
}
catch (DivideByZeroException ex)
{
    // Gestion spécifique pour les exceptions de division par zéro
    Console.WriteLine($"Erreur de division par zéro: {ex.Message}");
}
catch (Exception ex)
{
    // Gestion générique pour toutes les autres exceptions
    Console.WriteLine($"Une erreur s'est produite: {ex.Message}");
}
finally
{
    // Code exécuté qu'une exception soit levée ou non
    Console.WriteLine("Cette section est toujours exécutée");
}
```
### Ordre des blocs catch
Les blocs catch doivent être organisés du plus spécifique au plus général :
``` csharp
try
{
    // Ouverture d'un fichier qui n'existe pas
    using var stream = File.OpenRead("fichier_inexistant.txt");
}
catch (FileNotFoundException ex)
{
    // Gestion spécifique pour les fichiers manquants
    Console.WriteLine($"Le fichier n'a pas été trouvé: {ex.Message}");
}
catch (IOException ex)
{
    // Gestion pour toutes les erreurs d'entrée/sortie
    Console.WriteLine($"Erreur d'E/S: {ex.Message}");
}
catch (Exception ex)
{
    // Gestion générique en dernier recours
    Console.WriteLine($"Erreur générale: {ex.Message}");
}
```
### Le bloc finally
Le bloc finally est utilisé pour exécuter du code qui doit s'exécuter quoi qu'il arrive, même si une exception est levée :
``` csharp
FileStream fichier = null;
try
{
    fichier = File.OpenRead("données.txt");
    // Traitement du fichier...
}
catch (Exception ex)
{
    Console.WriteLine($"Erreur lors du traitement du fichier: {ex.Message}");
}
finally
{
    // Fermeture du fichier, même en cas d'erreur
    fichier?.Dispose();
}
```
### L'instruction using comme alternative au bloc finally
L'instruction using offre une syntaxe plus concise qui garantit automatiquement l'appel à Dispose() :
``` csharp
// Approche classique avec using
try
{
    using (var fichier = File.OpenRead("données.txt"))
    {
        // Traitement du fichier...
    } // Dispose() est appelé automatiquement ici
}
catch (Exception ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}

// Syntaxe simplifiée (C# 8.0+ / .NET 8)
try
{
    using var fichier = File.OpenRead("données.txt");
    // Traitement du fichier...
} // Dispose() est appelé automatiquement ici
catch (Exception ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}
```
### Lancer des exceptions
Vous pouvez lancer explicitement des exceptions avec le mot-clé : `throw`
``` csharp
public void Diviser(int a, int b)
{
    if (b == 0)
    {
        throw new ArgumentException("Le diviseur ne peut pas être zéro.", nameof(b));
    }

    int résultat = a / b;
    Console.WriteLine($"Résultat: {résultat}");
}
```
### Relancer une exception
Vous pouvez relancer une exception capturée pour préserver la pile d'appels d'origine :
``` csharp
try
{
    // Code qui peut générer une exception
    ProcessData();
}
catch (Exception ex)
{
    // Journaliser l'erreur
    Logger.Log(ex);

    // Relancer l'exception pour qu'elle soit traitée plus haut
    throw;
}
```
> **Important**: Utilisez simplement `throw;` et non `throw ex;` pour préserver la pile d'appels d'origine.
>

### Exceptions courantes en .NET
Voici quelques-unes des exceptions les plus couramment rencontrées en .NET :

| Exception | Description |
| --- | --- |
| `ArgumentException` | Argument invalide passé à une méthode |
| `ArgumentNullException` | Argument null passé à une méthode qui n'accepte pas les valeurs null |
| `ArgumentOutOfRangeException` | Argument en dehors de la plage autorisée |
| `FormatException` | Format invalide d'une chaîne lors d'une conversion |
| `InvalidOperationException` | Opération invalide dans le contexte actuel |
| `NullReferenceException` | Tentative d'accès à un membre d'un objet null |
| `IOException` | Erreur d'entrée/sortie |
| `FileNotFoundException` | Fichier non trouvé |
| `DirectoryNotFoundException` | Répertoire non trouvé |
| `UnauthorizedAccessException` | Accès non autorisé à une ressource |
| `TimeoutException` | Délai d'expiration atteint |
| `OutOfMemoryException` | Mémoire insuffisante |
| `SqlException` | Erreur SQL Server |
| `ObjectDisposedException` | Tentative d'utilisation d'un objet disposé |
### Accès aux propriétés d'une exception
Vous pouvez examiner diverses propriétés d'une exception pour obtenir des informations sur l'erreur :
``` csharp
try
{
    // Code qui peut générer une exception
    int[] tableau = new int[5];
    tableau[10] = 100; // Génère une IndexOutOfRangeException
}
catch (Exception ex)
{
    Console.WriteLine($"Message: {ex.Message}");
    Console.WriteLine($"Type d'exception: {ex.GetType().Name}");
    Console.WriteLine($"Pile d'appels: {ex.StackTrace}");

    if (ex.InnerException != null)
    {
        Console.WriteLine($"Exception interne: {ex.InnerException.Message}");
    }

    // Accès aux données spécifiques à l'exception
    foreach (DictionaryEntry data in ex.Data)
    {
        Console.WriteLine($"Donnée: {data.Key} = {data.Value}");
    }
}
```
### Exceptions imbriquées
Les exceptions peuvent être imbriquées pour fournir plus de contexte :
``` csharp
public void TraiterDonnées()
{
    try
    {
        // Tentative de traitement
        ChargerDonnéesDepuisAPI();
    }
    catch (Exception ex)
    {
        // Création d'une exception avec plus de contexte et l'exception d'origine comme InnerException
        throw new ApplicationException("Erreur lors du traitement des données", ex);
    }
}

public void ChargerDonnéesDepuisAPI()
{
    // Simulation d'une erreur
    throw new HttpRequestException("Échec de la requête API: timeout");
}

// Utilisation
try
{
    TraiterDonnées();
}
catch (Exception ex)
{
    Console.WriteLine($"Erreur principale: {ex.Message}");
    if (ex.InnerException != null)
    {
        Console.WriteLine($"Cause: {ex.InnerException.Message}");
    }
}
```
### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
| --- | --- | --- |
| try-catch-finally | ✅ | ✅ |
| Déclaration using simplifiée | ❌ | ✅ |
| Filtres d'exception | ✅ | ✅ |
| async/await dans les blocs catch/finally | ✅ | ✅ |
## 6.5.2. Exceptions personnalisées
Créer des exceptions personnalisées vous permet de définir des erreurs spécifiques à votre domaine métier.
### Création d'une exception personnalisée
``` csharp
// Exception personnalisée simple
public class ValidationClientException : Exception
{
    public ValidationClientException() : base("Erreur de validation du client")
    {
    }

    public ValidationClientException(string message) : base(message)
    {
    }

    public ValidationClientException(string message, Exception innerException)
        : base(message, innerException)
    {
    }
}
```
### Exception personnalisée avec informations supplémentaires
``` csharp
[Serializable] // Important pour .NET Framework 4.7.2 si l'exception doit être sérialisable
public class CommandeException : Exception
{
    public string NuméroCommande { get; }
    public decimal Montant { get; }

    public CommandeException(string message, string numéroCommande, decimal montant)
        : base(message)
    {
        NuméroCommande = numéroCommande;
        Montant = montant;
    }

    public CommandeException(string message, string numéroCommande, decimal montant, Exception innerException)
        : base(message, innerException)
    {
        NuméroCommande = numéroCommande;
        Montant = montant;
    }

    // Constructeur de sérialisation (requis pour .NET Framework 4.7.2)
    protected CommandeException(System.Runtime.Serialization.SerializationInfo info,
                               System.Runtime.Serialization.StreamingContext context)
        : base(info, context)
    {
        NuméroCommande = info.GetString(nameof(NuméroCommande));
        Montant = info.GetDecimal(nameof(Montant));
    }

    // Méthode de sérialisation (requise pour .NET Framework 4.7.2)
    public override void GetObjectData(System.Runtime.Serialization.SerializationInfo info,
                                      System.Runtime.Serialization.StreamingContext context)
    {
        base.GetObjectData(info, context);
        info.AddValue(nameof(NuméroCommande), NuméroCommande);
        info.AddValue(nameof(Montant), Montant);
    }
}
```
> **Note**: Dans .NET 8, la sérialisation des exceptions n'est pas aussi courante car d'autres mécanismes comme JSON sont généralement préférés. Les attributs `[Serializable]` et les constructeurs de sérialisation sont principalement utilisés pour la compatibilité avec .NET Framework.
>

### Utilisation des exceptions personnalisées
``` csharp
public class GestionnaireCommandes
{
    public void TraiterCommande(string numéroCommande, decimal montant)
    {
        if (string.IsNullOrEmpty(numéroCommande))
        {
            throw new ArgumentNullException(nameof(numéroCommande), "Le numéro de commande ne peut pas être vide");
        }

        if (montant <= 0)
        {
            throw new CommandeException(
                "Le montant de la commande doit être positif",
                numéroCommande,
                montant);
        }

        try
        {
            // Traitement de la commande
            Console.WriteLine($"Traitement de la commande {numéroCommande} pour {montant:C}");

            // Simuler une erreur aléatoire
            if (new Random().Next(10) == 0)
            {
                throw new IOException("Erreur de communication avec le système de paiement");
            }
        }
        catch (IOException ex)
        {
            throw new CommandeException(
                "Erreur lors du traitement de la commande",
                numéroCommande,
                montant,
                ex);
        }
    }
}

// Utilisation
var gestionnaire = new GestionnaireCommandes();

try
{
    gestionnaire.TraiterCommande("CMD-2023-001", 0);
}
catch (CommandeException ex)
{
    Console.WriteLine($"Erreur de commande: {ex.Message}");
    Console.WriteLine($"Numéro: {ex.NuméroCommande}, Montant: {ex.Montant:C}");

    if (ex.InnerException != null)
    {
        Console.WriteLine($"Cause: {ex.InnerException.Message}");
    }
}
```
### Hiérarchie d'exceptions personnalisées
Pour les applications complexes, vous pouvez créer une hiérarchie d'exceptions :
``` csharp
// Exception de base pour l'application
public class AppException : Exception
{
    public AppException(string message) : base(message) { }
    public AppException(string message, Exception innerException)
        : base(message, innerException) { }
}

// Exceptions liées aux entités
public class EntitéException : AppException
{
    public string EntitéId { get; }

    public EntitéException(string message, string entitéId)
        : base(message)
    {
        EntitéId = entitéId;
    }

    public EntitéException(string message, string entitéId, Exception innerException)
        : base(message, innerException)
    {
        EntitéId = entitéId;
    }
}

// Exceptions spécifiques
public class ClientException : EntitéException
{
    public ClientException(string message, string clientId)
        : base(message, clientId) { }

    public ClientException(string message, string clientId, Exception innerException)
        : base(message, clientId, innerException) { }
}

public class ProduitException : EntitéException
{
    public ProduitException(string message, string produitId)
        : base(message, produitId) { }

    public ProduitException(string message, string produitId, Exception innerException)
        : base(message, produitId, innerException) { }
}

// Utilisation
try
{
    // Code qui génère une exception
    throw new ClientException("Client non trouvé", "CLI-12345");
}
catch (ClientException ex)
{
    Console.WriteLine($"Erreur client: {ex.Message}, ID: {ex.EntitéId}");
}
catch (EntitéException ex)
{
    Console.WriteLine($"Erreur d'entité: {ex.Message}, ID: {ex.EntitéId}");
}
catch (AppException ex)
{
    Console.WriteLine($"Erreur d'application: {ex.Message}");
}
```
## 6.5.3. Meilleures pratiques
Voici les meilleures pratiques pour une gestion efficace des exceptions en C#.
### Quand lancer des exceptions
Les exceptions doivent être utilisées pour les conditions exceptionnelles et non pour le contrôle de flux normal :
``` csharp
// À éviter - utiliser des exceptions pour le contrôle de flux
public bool UtilisateurExiste(string username)
{
    try
    {
        var utilisateur = _repository.ObtenirUtilisateur(username);
        return true;
    }
    catch (UtilisateurNonTrouvéException)
    {
        return false;
    }
}

// Préférable - vérification explicite
public bool UtilisateurExiste(string username)
{
    return _repository.UtilisateurExiste(username);
}
```
### Validation des paramètres en entrée
Validez les paramètres en entrée pour éviter les exceptions prévisibles :
``` csharp
public void TraiterCommande(string numéroCommande, decimal montant, List<string> articles)
{
    // Validation des paramètres
    if (string.IsNullOrEmpty(numéroCommande))
        throw new ArgumentNullException(nameof(numéroCommande));

    if (montant <= 0)
        throw new ArgumentOutOfRangeException(nameof(montant), "Le montant doit être positif");

    if (articles == null)
        throw new ArgumentNullException(nameof(articles));

    if (articles.Count == 0)
        throw new ArgumentException("La liste d'articles ne peut pas être vide", nameof(articles));

    // Traitement normal...
}
```
### Utiliser les exceptions spécifiques
Lancez et capturez des exceptions spécifiques plutôt que génériques :
``` csharp
// À éviter
try
{
    // Code...
}
catch (Exception ex)
{
    // Traitement générique pour toutes les exceptions
    Console.WriteLine(ex.Message);
}

// Préférable
try
{
    // Code...
}
catch (SqlException ex)
{
    // Traitement spécifique pour les erreurs de base de données
    Console.WriteLine($"Erreur de base de données: {ex.Message}, Numéro: {ex.Number}");
}
catch (HttpRequestException ex)
{
    // Traitement spécifique pour les erreurs HTTP
    Console.WriteLine($"Erreur HTTP: {ex.Message}, Statut: {ex.StatusCode}");
}
catch (Exception ex)
{
    // Traitement pour les autres exceptions
    Console.WriteLine($"Erreur inattendue: {ex.Message}");
}
```
### Messages d'exception informatifs
Fournissez des messages d'exception clairs et informatifs :
``` csharp
// Message peu informatif
throw new ArgumentException("Valeur invalide");

// Message informatif
throw new ArgumentException("L'âge doit être compris entre 18 et 65 ans", nameof(age));
```
### Journalisation des exceptions
Journalisez les exceptions avec suffisamment de détails pour faciliter le débogage :
``` csharp
try
{
    // Code qui peut générer une exception
    ProcessOrder("ORD-123");
}
catch (Exception ex)
{
    // Journalisation avec contexte
    _logger.LogError(ex, "Erreur lors du traitement de la commande ORD-123. " +
                         "Utilisateur: {User}, Date: {Date}",
                         currentUser.Username, DateTime.Now);

    // Renvoyer une exception plus générique à l'utilisateur
    throw new ApplicationException("Impossible de traiter votre commande. Veuillez réessayer plus tard.");
}
```
### Gérer correctement les ressources
Utilisez `using` ou `try-finally` pour libérer les ressources, même en cas d'exception :
``` csharp
// Avec using
public void ExporterDonnées(string chemin)
{
    using var fichier = new StreamWriter(chemin);

    try
    {
        // Écriture des données...
        fichier.WriteLine("Données exportées");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Erreur lors de l'exportation des données");
        throw;
    }
    // Le fichier est automatiquement fermé grâce à using
}

// Avec try-finally
public void ExporterDonnées(string chemin)
{
    StreamWriter fichier = null;

    try
    {
        fichier = new StreamWriter(chemin);
        // Écriture des données...
        fichier.WriteLine("Données exportées");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Erreur lors de l'exportation des données");
        throw;
    }
    finally
    {
        fichier?.Dispose();
    }
}
```
### Ne pas avaler les exceptions
Évitez de capturer des exceptions sans les traiter correctement :
``` csharp
// À éviter - exception avalée
try
{
    // Code...
}
catch (Exception)
{
    // Aucun traitement, l'exception est ignorée
}

// Préférable - journalisation et gestion appropriée
try
{
    // Code...
}
catch (Exception ex)
{
    _logger.LogError(ex, "Une erreur s'est produite");

    // Soit relancer
    throw;

    // Soit traiter correctement
    // NotifyUser("Une erreur s'est produite. Veuillez réessayer.");
}
```
### Éviter les blocs try trop grands
Limitez la taille des blocs try pour cibler précisément les opérations qui peuvent échouer :
``` csharp
// À éviter - bloc try trop grand
try
{
    var utilisateur = _repository.ObtenirUtilisateur(username);
    var commande = CréerCommande(utilisateur);
    EnvoyerEmail(utilisateur, commande);
    MettreÀJourInventaire(commande);
    EnregistrerDansBD(commande);
}
catch (Exception ex)
{
    // Difficile de savoir quelle opération a échoué
    _logger.LogError(ex, "Erreur lors du traitement");
}

// Préférable - blocs try ciblés
try
{
    var utilisateur = _repository.ObtenirUtilisateur(username);
    var commande = CréerCommande(utilisateur);

    try
    {
        EnvoyerEmail(utilisateur, commande);
    }
    catch (SmtpException ex)
    {
        _logger.LogWarning(ex, "Échec de l'envoi de l'e-mail");
        // Continuer le traitement malgré l'échec de l'e-mail
    }

    try
    {
        MettreÀJourInventaire(commande);
        EnregistrerDansBD(commande);
    }
    catch (DbException ex)
    {
        _logger.LogError(ex, "Erreur de base de données");
        throw;
    }
}
catch (UtilisateurNonTrouvéException ex)
{
    _logger.LogWarning(ex, "Utilisateur non trouvé");
    throw;
}
```
### Gestion des exceptions asynchrones
Pour les méthodes asynchrones, utilisez `await` dans les blocs try-catch :
``` csharp
public async Task<UserData> GetUserDataAsync(string userId)
{
    try
    {
        // Appel asynchrone dans le bloc try
        var userData = await _apiClient.GetUserAsync(userId);
        return userData;
    }
    catch (HttpRequestException ex)
    {
        _logger.LogError(ex, "Échec de récupération des données utilisateur");

        // Relancer ou retourner une valeur par défaut
        throw new UserDataException($"Impossible d'obtenir les données pour l'utilisateur {userId}", ex);
    }
}
```
## 6.5.4. Filtres d'exception
Les filtres d'exception, disponibles dans .NET Framework 4.7.2 et .NET 8, permettent d'appliquer des conditions supplémentaires pour déterminer si un bloc catch doit traiter une exception.
### Syntaxe de base des filtres
``` csharp
try
{
    // Code qui peut générer une exception
    ProcessRequest();
}
catch (SqlException ex) when (ex.Number == 1205)
{
    // Gestion spécifique pour les erreurs de deadlock SQL (numéro 1205)
    Console.WriteLine("Deadlock détecté, nouvelle tentative...");
    RetryOperation();
}
catch (SqlException ex) when (ex.Number == 2627)
{
    // Gestion spécifique pour les violations de contrainte d'unicité (numéro 2627)
    Console.WriteLine("Données en double détectées.");
}
catch (SqlException ex)
{
    // Gestion pour les autres erreurs SQL
    Console.WriteLine($"Erreur SQL: {ex.Message}");
}
```
### Filtres avec expressions complexes
Les filtres peuvent contenir des expressions plus complexes :
``` csharp
try
{
    // Code qui peut générer une exception
    ProcessOrder();
}
catch (Exception ex) when (ex.Message.Contains("timeout") && DateTime.Now.Hour >= 23)
{
    Console.WriteLine("Timeout pendant les heures de maintenance");
    // Traitement spécifique pour les timeouts pendant la maintenance
}
catch (Exception ex) when (IsTransientError(ex))
{
    Console.WriteLine("Erreur transitoire détectée, nouvelle tentative...");
    // Méthode qui détermine si l'erreur est transitoire
}

bool IsTransientError(Exception ex)
{
    // Logique pour déterminer si une erreur est transitoire
    return ex is HttpRequestException ||
           (ex is SqlException sqlEx && (sqlEx.Number == 1205 || sqlEx.Number == -2));
}
```
### Filtres pour la journalisation sans interrompre la pile d'appels
Les filtres peuvent être utilisés pour journaliser les exceptions sans les capturer réellement :
``` csharp
try
{
    // Code qui peut générer une exception
    DiviserNombres(10, 0);
}
catch (Exception ex) when (LogException(ex))
{
    // Ce bloc ne sera jamais exécuté car LogException retourne false
}

bool LogException(Exception ex)
{
    // Journaliser l'exception
    Console.WriteLine($"ERREUR: {ex.Message}");

    // Retourner false pour que l'exception continue à se propager
    return false;
}
```
### Filtres pour le débogage
Les filtres peuvent être utilisés pour faciliter le débogage sans modifier le comportement normal :
``` csharp
try
{
    // Code qui peut générer une exception
    ProcessData();
}
catch (Exception ex) when (Debugger.IsAttached)
{
    // Ce bloc sera exécuté uniquement lors du débogage
    Console.WriteLine("Exception détectée pendant le débogage");
    Debugger.Break(); // Point d'arrêt pour l'inspection
    throw; // Relancer l'exception
}
catch (Exception ex)
{
    // Gestion normale des exceptions en production
    _logger.LogError(ex, "Erreur lors du traitement des données");
    throw;
}
```
## 6.5.5. Performance et considérations

La gestion des exceptions a un impact sur les performances de votre application. Voici quelques considérations importantes.

### Impact sur les performances

Les exceptions sont coûteuses en termes de performances, principalement lors de leur levée :

```textmate
// Approche avec exception (plus lente si l'exception est levée fréquemment)
public int ObtenirIndice(string[] tableau, string valeur)
{
    try
    {
        for (int i = 0; i < tableau.Length; i++)
        {
            if (tableau[i] == valeur)
                return i;
        }
        throw new ArgumentException($"Valeur '{valeur}' non trouvée dans le tableau");
    }
    catch (ArgumentException)
    {
        return -1;
    }
}

// Approche sans exception (plus rapide)
public int ObtenirIndice(string[] tableau, string valeur)
{
    for (int i = 0; i < tableau.Length; i++)
    {
        if (tableau[i] == valeur)
            return i;
    }
    return -1;  // Retourne -1 au lieu de lever une exception
}
```


> **Note**: L'exécution d'un bloc try-catch n'a pratiquement pas d'impact sur les performances s'il n'y a pas d'exception levée.

### Quand éviter les exceptions

Évitez d'utiliser les exceptions pour :

1. **Contrôle de flux normal** - Utilisez des retours de méthode appropriés
2. **Validation de paramètres prévisible** - Vérifiez les conditions en amont
3. **Conditions fréquentes** - Les exceptions doivent rester exceptionnelles

```textmate
// À éviter - utiliser des exceptions pour le contrôle de flux
public bool EstConnecté()
{
    try
    {
        var session = ObtenirSessionUtilisateur();
        return true;
    }
    catch (SessionExpiredException)
    {
        return false;
    }
}

// Préférable - vérification explicite
public bool EstConnecté()
{
    return SessionUtilisateurExiste() && !SessionUtilisateurExpirée();
}
```


### Exception suppression en .NET

.NET utilise l'optimisation "exception suppression" qui réduit le coût des blocs try non exécutés :

```textmate
// La présence de cette méthode a un impact négligeable sur les performances si aucune exception n'est levée
public void MéthodeAvecTryCatch()
{
    try
    {
        // Code normal...
    }
    catch (Exception ex)
    {
        // Gestion des exceptions...
    }
}
```


### Exceptions et code asynchrone

Les exceptions dans le code asynchrone méritent une attention particulière :

```textmate
// Gestion des exceptions dans une méthode asynchrone
public async Task TraiterCommandeAsync(string commandeId)
{
    try
    {
        var commande = await _repository.ObtenirCommandeAsync(commandeId);
        await _processor.TraiterAsync(commande);
        await _notifier.EnvoyerConfirmationAsync(commande);
    }
    catch (CommandeNotFoundException ex)
    {
        _logger.LogWarning(ex, "Commande {CommandeId} non trouvée", commandeId);
        throw;
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Erreur lors du traitement de la commande {CommandeId}", commandeId);

        // Agrégation d'exceptions - utilisée dans les opérations parallèles
        if (ex is AggregateException aggEx)
        {
            foreach (var innerEx in aggEx.InnerExceptions)
            {
                _logger.LogError(innerEx, "Exception interne");
            }
        }

        throw;
    }
}
```


### Exceptions dans les constructeurs

Soyez particulièrement prudent avec les exceptions dans les constructeurs :

```textmate
public class Client
{
    private readonly string _nom;
    private readonly ILogger _logger;

    public Client(string nom, ILogger logger)
    {
        // Validation des paramètres obligatoires
        _nom = nom ?? throw new ArgumentNullException(nameof(nom));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));

        try
        {
            // Initialisation qui peut échouer
            ChargerHistorique();
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Impossible de charger l'historique du client");
            // Ne pas laisser l'exception se propager - l'objet reste utilisable
        }
    }

    private void ChargerHistorique()
    {
        // Logique qui peut échouer
    }
}
```


### Utilisation d'exceptions vs codes d'erreur

Voici une comparaison entre l'utilisation des exceptions et des codes d'erreur :

#### Exceptions

**Avantages**:
- Séparent clairement le code de gestion des erreurs du code normal
- Impossible d'ignorer une erreur non gérée
- Propagent automatiquement les erreurs à travers les couches d'appel
- Fournissent des informations détaillées sur l'erreur (stacktrace, etc.)

**Inconvénients**:
- Coûteuses en performance lors de leur levée
- Peuvent rendre le flux d'exécution difficile à suivre
- Peuvent être utilisées de manière abusive

#### Codes d'erreur / Valeurs de retour

**Avantages**:
- Plus performants
- Flux de contrôle plus explicite
- Adaptés aux conditions d'erreur fréquentes

**Inconvénients**:
- Faciles à ignorer (oublier de vérifier le code de retour)
- Peuvent polluer les signatures de méthodes
- Propagation manuelle à travers les couches d'appel

### Approche hybride moderne: Pattern Result

Une approche hybride moderne consiste à utiliser le pattern Result pour les erreurs prévisibles tout en conservant les exceptions pour les conditions vraiment exceptionnelles :

```textmate
// Approche hybride avec le pattern Result - disponible dans .NET 8
public class Result<T>
{
    public bool Succès { get; }
    public T Valeur { get; }
    public string MessageErreur { get; }
    public Exception Exception { get; }

    private Result(bool succès, T valeur, string messageErreur, Exception exception)
    {
        Succès = succès;
        Valeur = valeur;
        MessageErreur = messageErreur;
        Exception = exception;
    }

    public static Result<T> Ok(T valeur) => new Result<T>(true, valeur, null, null);

    public static Result<T> Echec(string message) => new Result<T>(false, default, message, null);

    public static Result<T> Echec(Exception exception) =>
        new Result<T>(false, default, exception.Message, exception);
}

// Utilisation du pattern Result
public Result<Client> ObtenirClient(string id)
{
    try
    {
        if (string.IsNullOrEmpty(id))
            return Result<Client>.Echec("L'ID client ne peut pas être vide");

        var client = _repository.ObtenirClient(id);

        if (client == null)
            return Result<Client>.Echec($"Aucun client trouvé avec l'ID {id}");

        return Result<Client>.Ok(client);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Erreur lors de la récupération du client {ClientId}", id);
        return Result<Client>.Echec(ex);
    }
}

// Consommation du pattern Result
public void TraiterClient(string clientId)
{
    var résultat = ObtenirClient(clientId);

    if (!résultat.Succès)
    {
        Console.WriteLine($"Erreur: {résultat.MessageErreur}");
        return;
    }

    var client = résultat.Valeur;
    Console.WriteLine($"Client traité: {client.Nom}");
}
```


### Approche fonctionnelle avec Either<TError, TSuccess> (.NET 8)

Dans .NET 8, vous pouvez utiliser une approche plus fonctionnelle avec le type Either :

```textmate
public class Either<TError, TSuccess>
{
    private readonly TError _error;
    private readonly TSuccess _success;
    private readonly bool _isSuccess;

    private Either(TError error)
    {
        _error = error;
        _success = default;
        _isSuccess = false;
    }

    private Either(TSuccess success)
    {
        _error = default;
        _success = success;
        _isSuccess = true;
    }

    public static Either<TError, TSuccess> Error(TError error) => new Either<TError, TSuccess>(error);
    public static Either<TError, TSuccess> Success(TSuccess success) => new Either<TError, TSuccess>(success);

    public TResult Match<TResult>(
        Func<TError, TResult> onError,
        Func<TSuccess, TResult> onSuccess)
    {
        return _isSuccess ? onSuccess(_success) : onError(_error);
    }

    public void Match(
        Action<TError> onError,
        Action<TSuccess> onSuccess)
    {
        if (_isSuccess)
            onSuccess(_success);
        else
            onError(_error);
    }
}

public enum ErrorType
{
    NotFound,
    ValidationError,
    TechnicalError
}

public class Error
{
    public ErrorType Type { get; }
    public string Message { get; }
    public Exception Exception { get; }

    public Error(ErrorType type, string message, Exception exception = null)
    {
        Type = type;
        Message = message;
        Exception = exception;
    }
}

// Utilisation
public Either<Error, Client> ObtenirClient(string id)
{
    if (string.IsNullOrEmpty(id))
        return Either<Error, Client>.Error(
            new Error(ErrorType.ValidationError, "L'ID client ne peut pas être vide"));

    try
    {
        var client = _repository.ObtenirClient(id);

        if (client == null)
            return Either<Error, Client>.Error(
                new Error(ErrorType.NotFound, $"Aucun client trouvé avec l'ID {id}"));

        return Either<Error, Client>.Success(client);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Erreur lors de la récupération du client {ClientId}", id);
        return Either<Error, Client>.Error(
            new Error(ErrorType.TechnicalError, "Erreur technique lors de la récupération du client", ex));
    }
}

// Consommation
public void TraiterClient(string clientId)
{
    var résultat = ObtenirClient(clientId);

    résultat.Match(
        error =>
        {
            switch (error.Type)
            {
                case ErrorType.NotFound:
                    Console.WriteLine($"Client non trouvé: {error.Message}");
                    break;
                case ErrorType.ValidationError:
                    Console.WriteLine($"Erreur de validation: {error.Message}");
                    break;
                case ErrorType.TechnicalError:
                    Console.WriteLine($"Erreur technique: {error.Message}");
                    _logger.LogError(error.Exception, "Détails de l'erreur");
                    break;
            }
        },
        client =>
        {
            Console.WriteLine($"Client traité: {client.Nom}");
        });
}
```


### Utilisation des types nullables pour indiquer l'absence de valeur

Dans .NET 8, les types de référence nullables peuvent être utilisés pour indiquer l'absence de valeur de façon explicite :

```textmate
// Avec l'activation des types de référence nullables
#nullable enable

public Client? ObtenirClient(string id)
{
    if (string.IsNullOrEmpty(id))
        return null;

    return _repository.TrouverClientParId(id);
}

public void TraiterClient(string clientId)
{
    var client = ObtenirClient(clientId);

    if (client == null)
    {
        Console.WriteLine("Client non trouvé");
        return;
    }

    // client est garanti non-null à ce stade
    Console.WriteLine($"Client traité: {client.Nom}");
}
```


### Différences entre .NET Framework 4.7.2 et .NET 8

| Fonctionnalité | .NET Framework 4.7.2 | .NET 8 |
|----------------|----------------------|--------|
| Gestion des exceptions de base | ✅ | ✅ |
| Filtres d'exceptions | ✅ | ✅ |
| Types de référence nullables | ❌ | ✅ |
| Analyse statique améliorée | ❌ | ✅ |
| Optimisations de performance | Limitées | Améliorées |
| AsyncExceptionFilter | ❌ | ✅ (via middleware) |
| Pattern matching pour les exceptions | Limité | Avancé |

### Bonnes pratiques pour les exceptions dans les API

Dans les API, il est important de gérer correctement les exceptions pour fournir des réponses cohérentes :

```textmate
// Approche pour une API Web (.NET 8)
[ApiController]
[Route("api/[controller]")]
public class ClientsController : ControllerBase
{
    private readonly IClientService _clientService;
    private readonly ILogger<ClientsController> _logger;

    public ClientsController(IClientService clientService, ILogger<ClientsController> logger)
    {
        _clientService = clientService;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<ClientDto>> GetClient(string id)
    {
        try
        {
            var client = await _clientService.ObtenirClientAsync(id);

            if (client == null)
                return NotFound(new { message = $"Client {id} non trouvé" });

            return Ok(client);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Erreur lors de la récupération du client {ClientId}", id);

            // Ne pas exposer les détails de l'exception à l'extérieur
            return StatusCode(500, new { message = "Une erreur est survenue lors du traitement de votre demande" });
        }
    }
}
```


### Middleware d'exception global (.NET 8)

Dans .NET 8, vous pouvez utiliser un middleware d'exception global pour centraliser la gestion des exceptions dans les API Web :

```textmate
// Middleware d'exception personnalisé
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Une exception non gérée s'est produite");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";

        var statusCode = exception switch
        {
            ArgumentException _ => StatusCodes.Status400BadRequest,
            KeyNotFoundException _ => StatusCodes.Status404NotFound,
            UnauthorizedAccessException _ => StatusCodes.Status401Unauthorized,
            InvalidOperationException _ => StatusCodes.Status409Conflict,
            _ => StatusCodes.Status500InternalServerError
        };

        context.Response.StatusCode = statusCode;

        var response = new
        {
            status = statusCode,
            message = exception.Message,
            // Inclure le stacktrace uniquement en développement
            stackTrace = context.Environment.IsDevelopment() ? exception.StackTrace : null
        };

        await context.Response.WriteAsJsonAsync(response);
    }
}

// Enregistrement du middleware dans Program.cs ou Startup.cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Ajouter le middleware d'exception en premier
    app.UseMiddleware<ExceptionHandlingMiddleware>();

    // Autre middleware...
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```


### Gestion des exceptions dans les bibliothèques partagées

Dans les bibliothèques partagées, suivez ces bonnes pratiques :

1. **Documentation** - Documentez clairement les exceptions que votre API peut lever
2. **Exceptions spécifiques** - Utilisez ou créez des exceptions spécifiques à votre domaine
3. **Préservation du contexte** - Incluez suffisamment d'informations dans les exceptions
4. **Encapsulation** - Ne laissez pas fuiter les exceptions d'implémentation à travers votre API publique

```textmate
// Exemple de bibliothèque avec gestion des exceptions
namespace MyLibrary
{
    // Documentation XML pour indiquer les exceptions levées
    /// <summary>
    /// Fournit des fonctionnalités de traitement de paiement.
    /// </summary>
    public class PaymentProcessor
    {
        /// <summary>
        /// Traite un paiement.
        /// </summary>
        /// <param name="montant">Le montant à traiter</param>
        /// <param name="référence">La référence du paiement</param>
        /// <exception cref="ArgumentException">Levée lorsque le montant est négatif ou la référence vide</exception>
        /// <exception cref="PaymentProcessingException">Levée lorsque le traitement du paiement échoue</exception>
        public void TraiterPaiement(decimal montant, string référence)
        {
            // Validation des arguments
            if (montant <= 0)
                throw new ArgumentException("Le montant doit être positif", nameof(montant));

            if (string.IsNullOrEmpty(référence))
                throw new ArgumentException("La référence ne peut pas être vide", nameof(référence));

            try
            {
                // Logique de traitement qui peut échouer
                ProcessPaymentInternally(montant, référence);
            }
            catch (HttpRequestException ex)
            {
                // Encapsulation d'une exception d'implémentation en une exception d'API
                throw new PaymentProcessingException(
                    $"Échec de la communication avec le service de paiement pour la référence {référence}", ex);
            }
            catch (Exception ex) when (!(ex is PaymentProcessingException))
            {
                // Capture toutes les autres exceptions sauf celles que nous avons déjà encapsulées
                throw new PaymentProcessingException(
                    $"Erreur inattendue lors du traitement du paiement {référence}", ex);
            }
        }

        private void ProcessPaymentInternally(decimal montant, string référence)
        {
            // Implémentation interne...
        }
    }

    /// <summary>
    /// Exception levée lorsque le traitement d'un paiement échoue.
    /// </summary>
    public class PaymentProcessingException : Exception
    {
        public string RéférencePaiement { get; }

        public PaymentProcessingException(string message) : base(message)
        {
        }

        public PaymentProcessingException(string message, Exception innerException)
            : base(message, innerException)
        {
        }

        public PaymentProcessingException(string message, string référencePaiement)
            : base(message)
        {
            RéférencePaiement = référencePaiement;
        }

        public PaymentProcessingException(string message, string référencePaiement, Exception innerException)
            : base(message, innerException)
        {
            RéférencePaiement = référencePaiement;
        }
    }
}
```


### Récapitulatif des meilleures pratiques de performance

1. **Réservez les exceptions pour des conditions exceptionnelles** - Ne les utilisez pas pour le contrôle de flux normal
2. **Validez les entrées en amont** - Pour éviter de lever des exceptions prévisibles
3. **Utilisez des méthodes TryX()** - Les méthodes comme `int.TryParse()` ou `Dictionary.TryGetValue()` sont plus performantes
4. **Évitez de capturer et de relancer des exceptions inutilement** - Cela crée des piles d'appels plus longues
5. **Utilisez des exceptions spécifiques** - Facilitent la gestion et améliorent la lisibilité
6. **Libérez correctement les ressources** - Utilisez `using` ou `try-finally`
7. **Envisagez des alternatives aux exceptions** - Utilisez le pattern Result pour les erreurs prévisibles
8. **Gardez les blocs try les plus petits possibles** - Ne protégez que le code susceptible de générer une exception
9. **Évitez les exceptions dans les boucles critiques** - Elles peuvent considérablement dégrader les performances
10. **Considérez l'impact en production** - Journalisez de manière appropriée, mais évitez de surcharger les logs

## Conclusion

La gestion des exceptions est un aspect fondamental du développement en C#. Bien utilisées, les exceptions contribuent à créer des applications robustes et maintenables. Cependant, elles doivent être utilisées judicieusement, en tenant compte de leur impact sur les performances et la lisibilité du code.

En .NET Framework 4.7.2, vous disposez déjà des fonctionnalités essentielles pour gérer efficacement les exceptions. .NET 8 apporte des améliorations supplémentaires, notamment au niveau des types de référence nullables, de l'analyse statique et des optimisations de performance.

Quelle que soit la version de .NET que vous utilisez, suivez les meilleures pratiques décrites dans cette section pour créer un code plus robuste, plus maintenable et plus performant.

⏭️
