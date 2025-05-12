# 19.5. Patterns spécifiques à .NET

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les **patterns spécifiques à .NET** sont des modèles de conception qui tirent parti des fonctionnalités particulières du framework .NET. Ces patterns vous aident à écrire du code plus efficace, maintenable et idiomatique en C#.

## Introduction aux Patterns .NET

Contrairement aux patterns classiques (GoF), ces patterns sont optimisés pour :
- **La gestion des ressources** en C#
- **La programmation asynchrone** native
- **Les fonctionnalités d'injection de dépendance** intégrées
- **L'écosystème .NET** en général

## 19.5.1. Disposable Pattern - "Libérer les ressources proprement"

### Qu'est-ce que le Disposable Pattern ?

Le Disposable Pattern fournit un **mécanisme déterministe pour libérer les ressources non managées** comme les fichiers, connexions réseau, ou les handles de base de données.

### Quand utiliser le Disposable Pattern ?

- Quand votre classe détient des ressources non managées
- Pour implémenter des connexions de base de données
- Pour gérer des fichiers ou des streams
- Pour libérer des ressources coûteuses de manière prévisible

### Exemple simple : Gestion d'un fichier de log

```csharp
// Classe simple implémentant IDisposable
public class SimpleLogger : IDisposable
{
    private StreamWriter fileWriter;
    private bool disposed = false;

    public SimpleLogger(string logFilePath)
    {
        fileWriter = new StreamWriter(logFilePath, append: true);
    }

    public void LogMessage(string message)
    {
        if (disposed)
            throw new ObjectDisposedException(nameof(SimpleLogger));

        fileWriter.WriteLine($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] {message}");
        fileWriter.Flush();
    }

    // Implémentation de IDisposable
    public void Dispose()
    {
        // Ne pas faire le cleanup deux fois
        if (!disposed)
        {
            // Libérer les ressources managées
            fileWriter?.Dispose();

            // Marquer comme disposé
            disposed = true;
        }
    }
}

// Utilisation avec using statement
using (var logger = new SimpleLogger("app.log"))
{
    logger.LogMessage("Application démarrée");
    logger.LogMessage("Traitement en cours...");
    logger.LogMessage("Application terminée");
} // logger.Dispose() est appelé automatiquement ici
```

### Pattern Disposable complet avec finalizer

```csharp
public class CompleteResourceManager : IDisposable
{
    // Ressources managées
    private StreamReader fileReader;
    private HttpClient httpClient;

    // Ressource non managée (pour l'exemple)
    private IntPtr unmanagedHandle;

    // Flag pour indiquer si l'objet a été disposé
    private bool disposed = false;

    public CompleteResourceManager(string filePath)
    {
        fileReader = new StreamReader(filePath);
        httpClient = new HttpClient();

        // Simuler l'allocation d'une ressource non managée
        unmanagedHandle = AllocateUnmanagedResource();
    }

    // Finalizer (destructeur) - appelé par le GC si Dispose n'a pas été appelé
    ~CompleteResourceManager()
    {
        // Nettoyer seulement les ressources non managées
        Dispose(false);
    }

    // Implémentation publique de IDisposable
    public void Dispose()
    {
        // Nettoyer toutes les ressources
        Dispose(true);

        // Supprimer le finalizer car on a déjà nettoyé
        GC.SuppressFinalize(this);
    }

    // Méthode protégée pour le nettoyage
    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // Nettoyer les ressources managées
                fileReader?.Dispose();
                httpClient?.Dispose();
            }

            // Nettoyer les ressources non managées
            if (unmanagedHandle != IntPtr.Zero)
            {
                FreeUnmanagedResource(unmanagedHandle);
                unmanagedHandle = IntPtr.Zero;
            }

            disposed = true;
        }
    }

    // Méthode pour vérifier si l'objet a été disposé
    private void ThrowIfDisposed()
    {
        if (disposed)
            throw new ObjectDisposedException(GetType().Name);
    }

    public string ReadFile()
    {
        ThrowIfDisposed();
        return fileReader.ReadToEnd();
    }

    // Méthodes simulées pour les ressources non managées
    private IntPtr AllocateUnmanagedResource()
    {
        // Simuler l'allocation
        return new IntPtr(12345);
    }

    private void FreeUnmanagedResource(IntPtr handle)
    {
        // Simuler la libération
        Console.WriteLine($"Libération de la ressource non managée: {handle}");
    }
}

// Utilisation
using (var manager = new CompleteResourceManager("data.txt"))
{
    var content = manager.ReadFile();
    Console.WriteLine(content);
} // Dispose est appelé automatiquement
```

### Pattern Disposable pour les classes async

```csharp
// IAsyncDisposable pour les ressources asynchrones
public class AsyncResourceManager : IAsyncDisposable
{
    private readonly SemaphoreSlim semaphore = new SemaphoreSlim(1, 1);
    private NetworkStream networkStream;
    private bool disposed = false;

    public async Task<string> DownloadDataAsync(string url)
    {
        await semaphore.WaitAsync();
        try
        {
            if (disposed)
                throw new ObjectDisposedException(nameof(AsyncResourceManager));

            // Simuler le téléchargement
            await Task.Delay(100);
            return "Downloaded data";
        }
        finally
        {
            semaphore.Release();
        }
    }

    public async ValueTask DisposeAsync()
    {
        if (!disposed)
        {
            // Attendre que toutes les opérations se terminent
            await semaphore.WaitAsync();

            try
            {
                // Libérer les ressources
                if (networkStream != null)
                {
                    await networkStream.DisposeAsync();
                }

                disposed = true;
            }
            finally
            {
                semaphore.Release();
                semaphore.Dispose();
            }
        }
    }
}

// Utilisation avec await using
await using (var manager = new AsyncResourceManager())
{
    var data = await manager.DownloadDataAsync("http://example.com");
    Console.WriteLine(data);
} // DisposeAsync est appelé automatiquement
```

## 19.5.2. Async/await patterns - "Programmation asynchrone efficace"

### Qu'est-ce que les patterns async/await ?

Les patterns async/await permettent d'**écrire du code asynchrone de manière synchrone**, simplifiant la gestion des opérations asynchrones et améliorant la réactivité des applications.

### Patterns courants avec async/await

#### 1. Pattern de base

```csharp
public class DataService
{
    private readonly HttpClient httpClient = new HttpClient();

    // Méthode async de base
    public async Task<string> GetDataAsync(string url)
    {
        try
        {
            var response = await httpClient.GetStringAsync(url);
            return response;
        }
        catch (HttpRequestException ex)
        {
            // Gérer les erreurs
            throw new ApplicationException($"Erreur lors de la récupération des données: {ex.Message}", ex);
        }
    }

    // Méthode async avec cancellation
    public async Task<string> GetDataWithCancellationAsync(string url, CancellationToken cancellationToken)
    {
        using var cts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);
        cts.CancelAfter(TimeSpan.FromSeconds(30)); // Timeout de 30 secondes

        return await httpClient.GetStringAsync(url, cts.Token);
    }
}
```

#### 2. Pattern pour les opérations parallèles

```csharp
public class ParallelOperationsService
{
    public async Task<IEnumerable<WeatherData>> GetWeatherForCitiesAsync(IEnumerable<string> cities)
    {
        // Créer toutes les tâches
        var tasks = cities.Select(city => GetWeatherForCityAsync(city));

        // Attendre que toutes les tâches se terminent
        var results = await Task.WhenAll(tasks);

        return results;
    }

    public async Task<ProcessingResult> ProcessMultipleFilesParallelAsync(IEnumerable<string> filePaths)
    {
        // Utiliser Parallel.ForEachAsync (disponible en .NET 6+)
        var results = new ConcurrentBag<FileProcessingResult>();

        await Parallel.ForEachAsync(filePaths, async (filePath, ct) =>
        {
            var result = await ProcessFileAsync(filePath, ct);
            results.Add(result);
        });

        return new ProcessingResult { Results = results.ToList() };
    }
}
```

#### 3. Pattern pour la gestion des exceptions

```csharp
public class RobustAsyncService
{
    public async Task<Result<T>> TryExecuteAsync<T>(Func<Task<T>> operation, int maxRetries = 3)
    {
        int attempt = 0;
        Exception lastException = null;

        while (attempt < maxRetries)
        {
            try
            {
                var result = await operation();
                return Result<T>.Success(result);
            }
            catch (Exception ex) when (attempt < maxRetries - 1)
            {
                lastException = ex;
                attempt++;

                // Attendre avant de réessayer (exponential backoff)
                await Task.Delay(TimeSpan.FromMilliseconds(Math.Pow(2, attempt) * 100));
            }
            catch (Exception ex)
            {
                lastException = ex;
                break;
            }
        }

        return Result<T>.Failure(lastException);
    }
}

// Classe Result pour encapsuler les résultats
public class Result<T>
{
    public bool IsSuccess { get; }
    public T Value { get; }
    public Exception Exception { get; }

    private Result(bool isSuccess, T value, Exception exception)
    {
        IsSuccess = isSuccess;
        Value = value;
        Exception = exception;
    }

    public static Result<T> Success(T value) => new Result<T>(true, value, null);
    public static Result<T> Failure(Exception exception) => new Result<T>(false, default, exception);
}
```

#### 4. Pattern de streaming asynchrone

```csharp
public class AsyncStreamingService
{
    // Utilisation d'IAsyncEnumerable pour les streams
    public async IAsyncEnumerable<LogEntry> ReadLogsAsync(
        string logFilePath,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        using var reader = new StreamReader(logFilePath);

        while (!reader.EndOfStream)
        {
            cancellationToken.ThrowIfCancellationRequested();

            var line = await reader.ReadLineAsync();
            if (line != null && TryParseLogEntry(line, out var logEntry))
            {
                yield return logEntry;
            }
        }
    }

    // Traitement par lots des streams
    public async Task<List<string>> ProcessLargeFileInBatchesAsync(string filePath, int batchSize = 1000)
    {
        var results = new List<string>();
        var currentBatch = new List<string>();

        await foreach (var logEntry in ReadLogsAsync(filePath))
        {
            currentBatch.Add(logEntry.Message);

            if (currentBatch.Count >= batchSize)
            {
                // Traiter le lot
                var processedBatch = await ProcessBatchAsync(currentBatch);
                results.AddRange(processedBatch);
                currentBatch.Clear();
            }
        }

        // Traiter le dernier lot
        if (currentBatch.Count > 0)
        {
            var processedBatch = await ProcessBatchAsync(currentBatch);
            results.AddRange(processedBatch);
        }

        return results;
    }
}
```

#### 5. Pattern pour la configuration et l'état

```csharp
public class ConfigurableAsyncService
{
    private readonly SemaphoreSlim semaphore;
    private readonly AsyncServiceOptions options;

    public ConfigurableAsyncService(IOptions<AsyncServiceOptions> options)
    {
        this.options = options.Value;
        this.semaphore = new SemaphoreSlim(options.Value.MaxConcurrentOperations);
    }

    public async Task<T> ExecuteWithThrottlingAsync<T>(Func<Task<T>> operation)
    {
        await semaphore.WaitAsync();
        try
        {
            using var cts = new CancellationTokenSource(options.OperationTimeout);

            // Exécuter l'opération avec timeout
            var task = operation();
            var timeoutTask = Task.Delay(Timeout.Infinite, cts.Token);

            var completedTask = await Task.WhenAny(task, timeoutTask);

            if (completedTask == timeoutTask)
            {
                throw new TimeoutException("L'opération a dépassé le timeout");
            }

            return await task;
        }
        finally
        {
            semaphore.Release();
        }
    }
}

public class AsyncServiceOptions
{
    public int MaxConcurrentOperations { get; set; } = 10;
    public TimeSpan OperationTimeout { get; set; } = TimeSpan.FromSeconds(30);
}
```

## 19.5.3. Options Pattern - "Configuration fortement typée"

### Qu'est-ce que l'Options Pattern ?

L'Options Pattern fournit un **moyen structuré et fortement typé** pour gérer la configuration dans les applications .NET, avec validation, binding automatique et injection de dépendances intégrée.

### Exemple simple : Configuration d'application

```csharp
// Classe d'options simple
public class EmailSettings
{
    public string SmtpHost { get; set; }
    public int SmtpPort { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public bool EnableSsl { get; set; }
    public string FromAddress { get; set; }
    public string FromName { get; set; }
}

// Service utilisant les options
public class EmailService
{
    private readonly EmailSettings settings;

    // Injection des options via IOptions<T>
    public EmailService(IOptions<EmailSettings> options)
    {
        settings = options.Value;
    }

    public async Task SendEmailAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(settings.SmtpHost, settings.SmtpPort);
        client.EnableSsl = settings.EnableSsl;
        client.Credentials = new NetworkCredential(settings.Username, settings.Password);

        var message = new MailMessage(settings.FromAddress, to)
        {
            Subject = subject,
            Body = body,
            IsBodyHtml = true
        };

        message.Headers.Add("From-Name", settings.FromName);

        await client.SendMailAsync(message);
    }
}

// Configuration dans appsettings.json
/*
{
  "EmailSettings": {
    "SmtpHost": "smtp.gmail.com",
    "SmtpPort": 587,
    "Username": "your-email@gmail.com",
    "Password": "your-password",
    "EnableSsl": true,
    "FromAddress": "noreply@yourapp.com",
    "FromName": "Your App"
  }
}
*/

// Configuration dans Program.cs
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));

builder.Services.AddSingleton<EmailService>();
```

### Options avancées avec validation

```csharp
// Options avec validation intégrée
public class DatabaseOptions : IValidateOptions<DatabaseOptions>
{
    public string ConnectionString { get; set; }
    public int CommandTimeout { get; set; } = 30;
    public int MaxRetryAttempts { get; set; } = 3;
    public TimeSpan RetryDelay { get; set; } = TimeSpan.FromSeconds(1);
    public bool EnableQueryLogging { get; set; }

    // Implémentation de la validation
    public ValidateOptionsResult Validate(string name, DatabaseOptions options)
    {
        var failures = new List<string>();

        if (string.IsNullOrWhiteSpace(options.ConnectionString))
        {
            failures.Add("ConnectionString est requis");
        }

        if (options.CommandTimeout <= 0)
        {
            failures.Add("CommandTimeout doit être positif");
        }

        if (options.MaxRetryAttempts < 0)
        {
            failures.Add("MaxRetryAttempts ne peut pas être négatif");
        }

        if (failures.Any())
        {
            return ValidateOptionsResult.Fail(failures);
        }

        return ValidateOptionsResult.Success;
    }
}

// Utilisation avec validation
public class DatabaseService
{
    private readonly DatabaseOptions options;

    public DatabaseService(IOptions<DatabaseOptions> options)
    {
        this.options = options.Value; // Validation appliquée ici
    }
}

// Configuration avec validation
builder.Services.Configure<DatabaseOptions>(
    builder.Configuration.GetSection("Database"));

// Ajouter la validation
builder.Services.AddSingleton<IValidateOptions<DatabaseOptions>, DatabaseOptions>();
```

### Options avec sections nommées

```csharp
// Options pour différentes configurations
public class ConnectionStrings
{
    public string Primary { get; set; }
    public string Reporting { get; set; }
    public string Logging { get; set; }
}

// Service utilisant plusieurs configurations
public class MultiDatabaseService
{
    private readonly string primaryConnection;
    private readonly string reportingConnection;

    public MultiDatabaseService(
        IOptions<ConnectionStrings> connectionOptions,
        IOptionsSnapshot<DatabaseOptions> databaseOptions)
    {
        primaryConnection = connectionOptions.Value.Primary;
        reportingConnection = connectionOptions.Value.Reporting;

        // IOptionsSnapshot permet de récupérer des configurations nommées
        var primaryConfig = databaseOptions.Get("Primary");
        var reportingConfig = databaseOptions.Get("Reporting");
    }
}

// Configuration dans appsettings.json
/*
{
  "ConnectionStrings": {
    "Primary": "Data Source=primary.db",
    "Reporting": "Data Source=reporting.db",
    "Logging": "Data Source=logging.db"
  },
  "Database:Primary": {
    "CommandTimeout": 30,
    "MaxRetryAttempts": 3
  },
  "Database:Reporting": {
    "CommandTimeout": 60,
    "MaxRetryAttempts": 5
  }
}
*/
```

### Options avec configuration dynamique

```csharp
// Service qui réagit aux changements de configuration
public class ConfigurableCache : IDisposable
{
    private readonly IOptionsMonitor<CacheOptions> optionsMonitor;
    private IDisposable optionsChangeListener;
    private MemoryCache cache;

    public ConfigurableCache(IOptionsMonitor<CacheOptions> optionsMonitor)
    {
        this.optionsMonitor = optionsMonitor;

        // Initialiser avec la configuration actuelle
        InitializeCache(optionsMonitor.CurrentValue);

        // S'abonner aux changements
        optionsChangeListener = optionsMonitor.OnChange(newOptions =>
        {
            // Reconfigurer le cache lors des changements
            cache?.Dispose();
            InitializeCache(newOptions);
        });
    }

    private void InitializeCache(CacheOptions options)
    {
        cache = new MemoryCache(new MemoryCacheOptions
        {
            SizeLimit = options.SizeLimit,
            ExpirationScanFrequency = options.ExpirationScanFrequency
        });
    }

    public void Set<T>(string key, T value)
    {
        cache.Set(key, value, optionsMonitor.CurrentValue.DefaultExpiration);
    }

    public void Dispose()
    {
        optionsChangeListener?.Dispose();
        cache?.Dispose();
    }
}

public class CacheOptions
{
    public long SizeLimit { get; set; } = 100;
    public TimeSpan ExpirationScanFrequency { get; set; } = TimeSpan.FromMinutes(5);
    public TimeSpan DefaultExpiration { get; set; } = TimeSpan.FromHours(1);
}
```

## 19.5.4. Factory Pattern avec DI - "Création d'objets avec injection de dépendances"

### Qu'est-ce que le Factory Pattern avec DI ?

Ce pattern combine la **flexibilité du Factory Pattern** avec la **puissance de l'injection de dépendances**, permettant de créer des objets tout en conservant la testabilité et le découplage.

### Pattern Factory simple avec DI

```csharp
// Interface pour les notifications
public interface INotification
{
    Task SendAsync(string recipient, string message);
}

// Implémentations concrètes
public class EmailNotification : INotification
{
    private readonly IEmailSender emailSender;

    public EmailNotification(IEmailSender emailSender)
    {
        this.emailSender = emailSender;
    }

    public async Task SendAsync(string recipient, string message)
    {
        await emailSender.SendEmailAsync(recipient, "Notification", message);
    }
}

public class SmsNotification : INotification
{
    private readonly ISmsSender smsSender;

    public SmsNotification(ISmsSender smsSender)
    {
        this.smsSender = smsSender;
    }

    public async Task SendAsync(string recipient, string message)
    {
        await smsSender.SendSmsAsync(recipient, message);
    }
}

// Factory interface
public interface INotificationFactory
{
    INotification Create(NotificationType type);
}

// Factory implementation
public class NotificationFactory : INotificationFactory
{
    private readonly IServiceProvider serviceProvider;

    public NotificationFactory(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
    }

    public INotification Create(NotificationType type)
    {
        return type switch
        {
            NotificationType.Email => serviceProvider.GetRequiredService<EmailNotification>(),
            NotificationType.Sms => serviceProvider.GetRequiredService<SmsNotification>(),
            _ => throw new ArgumentException($"Type de notification non supporté: {type}")
        };
    }
}

// Configuration dans Program.cs
builder.Services.AddScoped<IEmailSender, SmtpEmailSender>();
builder.Services.AddScoped<ISmsSender, TwilioSmsSender>();
builder.Services.AddScoped<EmailNotification>();
builder.Services.AddScoped<SmsNotification>();
builder.Services.AddScoped<INotificationFactory, NotificationFactory>();
```

### Factory générique avec paramètres

```csharp
// Factory générique pour créer des objets avec paramètres
public interface IFactory<T>
{
    T Create(params object[] parameters);
}

// Interface pour les handlers
public interface ICommandHandler<TCommand>
{
    Task HandleAsync(TCommand command);
}

// Factory pour les handlers
public class CommandHandlerFactory : IFactory<object>
{
    private readonly IServiceProvider serviceProvider;
    private readonly Dictionary<Type, Type> handlerMappings = new();

    public CommandHandlerFactory(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
        RegisterHandlers();
    }

    private void RegisterHandlers()
    {
        // Enregistrer automatiquement tous les handlers
        var handlerTypes = AppDomain.CurrentDomain.GetAssemblies()
            .SelectMany(assembly => assembly.GetTypes())
            .Where(type => type.GetInterfaces().Any(i =>
                i.IsGenericType &&
                i.GetGenericTypeDefinition() == typeof(ICommandHandler<>)));

        foreach (var handlerType in handlerTypes)
        {
            var commandType = handlerType.GetInterfaces()
                .First(i => i.IsGenericType && i.GetGenericTypeDefinition() == typeof(ICommandHandler<>))
                .GetGenericArguments()[0];

            handlerMappings[commandType] = handlerType;
        }
    }

    public T Create<TCommand>()
    {
        if (handlerMappings.TryGetValue(typeof(TCommand), out var handlerType))
        {
            return (T)serviceProvider.GetRequiredService(handlerType);
        }

        throw new InvalidOperationException($"Aucun handler trouvé pour {typeof(TCommand).Name}");
    }

    public object Create(params object[] parameters)
    {
        // Implémentation pour la compatibilité avec IFactory<object>
        throw new NotImplementedException("Utilisez Create<TCommand>() à la place");
    }
}
```

### Factory avec options et configuration

```csharp
// Options pour configurer la factory
public class ProcessorFactoryOptions
{
    public Dictionary<string, Type> ProcessorTypes { get; set; } = new();
    public Dictionary<string, Dictionary<string, object>> ProcessorConfigurations { get; set; } = new();
}

// Factory configurable
public class ConfigurableProcessorFactory
{
    private readonly IServiceProvider serviceProvider;
    private readonly ProcessorFactoryOptions options;

    public ConfigurableProcessorFactory(
        IServiceProvider serviceProvider,
        IOptions<ProcessorFactoryOptions> options)
    {
        this.serviceProvider = serviceProvider;
        this.options = options.Value;
    }

    public IDataProcessor CreateProcessor(string processorType, string configurationName = null)
    {
        // Obtenir le type du processeur
        if (!options.ProcessorTypes.TryGetValue(processorType, out var type))
        {
            throw new ArgumentException($"Type de processeur inconnu: {processorType}");
        }

        // Créer l'instance via DI
        var processor = (IDataProcessor)serviceProvider.GetRequiredService(type);

        // Appliquer la configuration si spécifiée
        if (configurationName != null &&
            options.ProcessorConfigurations.TryGetValue(configurationName, out var config))
        {
            ApplyConfiguration(processor, config);
        }

        return processor;
    }

    private void ApplyConfiguration(IDataProcessor processor, Dictionary<string, object> config)
    {
        var type = processor.GetType();

        foreach (var kvp in config)
        {
            var property = type.GetProperty(kvp.Key);
            if (property != null && property.CanWrite)
            {
                property.SetValue(processor, kvp.Value);
            }
        }
    }
}

// Utilisation
var processor = processorFactory.CreateProcessor("image", "thumbnail");
var result = await processor.ProcessAsync(data);
```

### Dependency Injection avancée pour Factory

```csharp
// Extension methods pour enregistrer les factories
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddCustomFactory<TInterface, TImplementation>(
        this IServiceCollection services)
        where TImplementation : class
        where TInterface : class
    {
        // Enregistrer la factory elle-même
        services.AddSingleton<IFactory<TInterface>>(provider =>
        {
            return new DynamicFactory<TInterface>(provider);
        });

        // Enregistrer le type à créer
        services.AddTransient<TImplementation>();

        return services;
    }

    public static IServiceCollection AddFactoryPattern<TFactory, TInterface>(
        this IServiceCollection services,
        Action<FactoryConfiguration<TInterface>> configure = null)
        where TFactory : class, IFactory<TInterface>
        where TInterface : class
    {
        var configuration = new FactoryConfiguration<TInterface>();
        configure?.Invoke(configuration);

        services.AddSingleton(configuration);
        services.AddSingleton<IFactory<TInterface>, TFactory>();

        return services;
    }
}

// Configuration pour la factory
public class FactoryConfiguration<T>
{
    public Dictionary<string, Type> TypeMappings { get; } = new();
    public Dictionary<string, Func<IServiceProvider, T>> CustomFactories { get; } = new();

    public FactoryConfiguration<T> Register<TImplementation>(string key)
        where TImplementation : class, T
    {
        TypeMappings[key] = typeof(TImplementation);
        return this;
    }

    public FactoryConfiguration<T> Register(string key, Func<IServiceProvider, T> factory)
    {
        CustomFactories[key] = factory;
        return this;
    }
}

// Utilisation dans Program.cs
builder.Services.AddFactoryPattern<NotificationFactory, INotification>(config =>
{
    config.Register<EmailNotification>("email")
          .Register<SmsNotification>("sms")
          .Register("slack", provider =>
              new SlackNotification(provider.GetRequiredService<ISlackClient>(), "default-channel"));
});
```

## Résumé des Patterns spécifiques à .NET

| Pattern | But principal | Cas d'utilisation typique |
|---------|--------------|--------------------------|
| **Disposable** | Gestion des ressources | Fichiers, connexions DB, HTTP clients |
| **Async/await** | Opérations asynchrones | API calls, I/O, opérations longues |
| **Options** | Configuration typée | Settings, configuration d'application |
| **Factory avec DI** | Création dynamique | Stratégies, plugins, handlers |

## Bonnes pratiques pour les Patterns .NET

### 1. Disposable Pattern
- Implémentez `IDisposable` pour toutes les ressources non managées
- Utilisez le pattern complet avec finalizer si nécessaire
- Supportez `IAsyncDisposable` pour les opérations async
- Toujours utiliser `using` statements ou `using` declarations
- Vérifiez l'état disposed dans les méthodes publiques
- Documentez quelles ressources sont libérées par `Dispose()`

### 2. Async/await patterns
- Utilisez `async Task` plutôt que `async void` (sauf pour les event handlers)
- Propagez les CancellationToken à travers la chaîne d'appel
- Utilisez `ConfigureAwait(false)` dans les bibliothèques
- Évitez de bloquer avec `.Wait()` ou `.Result`
- Gérez les exceptions proprement avec try-catch async
- Utilisez `IAsyncEnumerable` pour les streams de données

### 3. Options Pattern
- Validez toujours vos options au démarrage
- Utilisez `IOptions<T>` pour les configurations statiques
- Utilisez `IOptionsSnapshot<T>` pour les configurations qui peuvent changer
- Utilisez `IOptionsMonitor<T>` pour réagir aux changements
- Organisez vos options par domaine fonctionnel
- Documentez les options disponibles et leurs valeurs par défaut

### 4. Factory Pattern avec DI
- Évitez d'injecter `IServiceProvider` directement
- Utilisez des factories typées plutôt que des factories génériques
- Enregistrez les dépendances avec des lifetimes appropriés
- Permettez la configuration des factories via Options
- Gérez les cas d'erreur (type non trouvé, etc.)
- Documentez les types disponibles dans la factory

## Exercices pratiques

### Exercice 1 : ResourceManager avec Disposable
Créez un gestionnaire de ressources qui :
- Gère des connexions de base de données
- Implémente le Disposable pattern complet
- Supporte async dispose
- Suit le pattern singleton thread-safe

### Exercice 2 : Service de téléchargement Async
Développez un service qui :
- Télécharge des fichiers en parallèle
- Supporte l'annulation et le timeout
- Fournit une progression
- Gère la retry avec exponential backoff

### Exercice 3 : Système de plugin avec Factory
Implémentez un système de plugins qui :
- Utilise le Factory pattern avec DI
- Charge dynamiquement des plugins
- Configure les plugins via Options
- Supporte des plugins avec dépendances

### Exercice 4 : Configuration hiérarchique
Créez un système de configuration qui :
- Supporte plusieurs sources (JSON, env vars, etc.)
- Permet la validation à plusieurs niveaux
- Réagit aux changements de configuration
- Gère des configurations par environnement

## Combinaison des Patterns .NET

### Exemple : Service complet utilisant tous les patterns

```csharp
// Combinaison de tous les patterns pour un service de traitement de données
public class DataProcessingService : IDisposable, IAsyncDisposable
{
    private readonly IDataProcessorFactory factory;
    private readonly IOptions<DataProcessingOptions> options;
    private readonly ILogger<DataProcessingService> logger;
    private readonly SemaphoreSlim semaphore;
    private bool disposed = false;

    public DataProcessingService(
        IDataProcessorFactory factory,
        IOptions<DataProcessingOptions> options,
        ILogger<DataProcessingService> logger)
    {
        this.factory = factory;
        this.options = options;
        this.logger = logger;
        this.semaphore = new SemaphoreSlim(options.Value.MaxConcurrentOperations);
    }

    public async Task<ProcessingResult> ProcessDataAsync(
        DataRequest request,
        CancellationToken cancellationToken = default)
    {
        // Pattern Async/await avec gestion du cancellation
        await semaphore.WaitAsync(cancellationToken);

        try
        {
            // Factory Pattern with DI
            var processor = factory.CreateProcessor(request.ProcessorType);

            // Timeout configuration (Options Pattern)
            using var cts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);
            cts.CancelAfter(options.Value.ProcessingTimeout);

            // Execution avec retry
            return await ExecuteWithRetryAsync(
                () => processor.ProcessAsync(request.Data, cts.Token),
                options.Value.MaxRetryAttempts);
        }
        finally
        {
            semaphore.Release();
        }
    }

    private async Task<ProcessingResult> ExecuteWithRetryAsync(
        Func<Task<ProcessingResult>> operation,
        int maxAttempts)
    {
        for (int attempt = 1; attempt <= maxAttempts; attempt++)
        {
            try
            {
                return await operation();
            }
            catch (Exception ex) when (attempt < maxAttempts)
            {
                logger.LogWarning(ex, "Tentative {Attempt}/{Max} échouée", attempt, maxAttempts);
                await Task.Delay(TimeSpan.FromMilliseconds(Math.Pow(2, attempt) * 100));
            }
        }

        throw new ProcessingException("Échec après toutes les tentatives");
    }

    // Disposable Pattern
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    public async ValueTask DisposeAsync()
    {
        await DisposeAsyncCore();

        Dispose(false);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                semaphore?.Dispose();
            }

            disposed = true;
        }
    }

    protected virtual async ValueTask DisposeAsyncCore()
    {
        if (semaphore != null)
        {
            // Attendre que toutes les opérations se terminent
            for (int i = 0; i < options.Value.MaxConcurrentOperations; i++)
            {
                await semaphore.WaitAsync();
            }

            semaphore.Dispose();
        }
    }
}

// Options supportant la validation
public class DataProcessingOptions : IValidateOptions<DataProcessingOptions>
{
    public int MaxConcurrentOperations { get; set; } = 4;
    public TimeSpan ProcessingTimeout { get; set; } = TimeSpan.FromMinutes(5);
    public int MaxRetryAttempts { get; set; } = 3;

    public ValidateOptionsResult Validate(string name, DataProcessingOptions options)
    {
        var failures = new List<string>();

        if (options.MaxConcurrentOperations <= 0)
            failures.Add("MaxConcurrentOperations doit être positif");

        if (options.ProcessingTimeout <= TimeSpan.Zero)
            failures.Add("ProcessingTimeout doit être positif");

        if (options.MaxRetryAttempts < 0)
            failures.Add("MaxRetryAttempts ne peut pas être négatif");

        return failures.Any() ?
            ValidateOptionsResult.Fail(failures) :
            ValidateOptionsResult.Success;
    }
}
```

## Architecture moderne avec les Patterns .NET

### Microservices avec .NET 8+

```csharp
// Programme minimal utilisant tous les patterns
var builder = WebApplication.CreateBuilder(args);

// Configuration des options
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("Email"));

builder.Services.Configure<DatabaseOptions>(
    builder.Configuration.GetSection("Database"))
    .AddSingleton<IValidateOptions<DatabaseOptions>, DatabaseOptions>();

// Services avec lifetime approprié
builder.Services.AddSingleton<IServiceBus, AzureServiceBus>();
builder.Services.AddScoped<IUnitOfWork, EFCoreUnitOfWork>();
builder.Services.AddTransient<IEmailService, EmailService>();

// Factories avec DI
builder.Services.AddFactoryPattern<NotificationFactory, INotification>(config =>
{
    config.Register<EmailNotification>("email")
          .Register<SlackNotification>("slack")
          .Register<TeamsNotification>("teams");
});

// Hosted services pour background tasks
builder.Services.AddHostedService<EventProcessingBackgroundService>();

var app = builder.Build();

// Middleware pipeline
app.UseMiddleware<AsyncExceptionMiddleware>();
app.MapControllers();

// Graceful shutdown
var lifetime = app.Services.GetRequiredService<IHostApplicationLifetime>();
lifetime.ApplicationStopping.Register(async () =>
{
    // Cleanup async resources
    await app.Services.GetService<IAsyncDisposable>()?.DisposeAsync();
});

await app.RunAsync();
```

### Background Service utilisant les patterns

```csharp
public class EventProcessingBackgroundService : BackgroundService
{
    private readonly IServiceScopeFactory scopeFactory;
    private readonly IOptionsMonitor<EventProcessingOptions> optionsMonitor;
    private readonly ILogger<EventProcessingBackgroundService> logger;

    public EventProcessingBackgroundService(
        IServiceScopeFactory scopeFactory,
        IOptionsMonitor<EventProcessingOptions> optionsMonitor,
        ILogger<EventProcessingBackgroundService> logger)
    {
        this.scopeFactory = scopeFactory;
        this.optionsMonitor = optionsMonitor;
        this.logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // Réagir aux changements de configuration
        var optionsChange = optionsMonitor.OnChange(async newOptions =>
        {
            logger.LogInformation("Configuration changed, restarting with new options");
            // Réinitialiser le service si besoin
        });

        try
        {
            await ProcessEventsAsync(stoppingToken);
        }
        finally
        {
            optionsChange.Dispose();
        }
    }

    private async Task ProcessEventsAsync(CancellationToken cancellationToken)
    {
        await foreach (var eventItem in GetEventsAsync(cancellationToken))
        {
            using var scope = scopeFactory.CreateScope();
            var processor = scope.ServiceProvider.GetRequiredService<IEventProcessor>();

            try
            {
                await processor.ProcessAsync(eventItem, cancellationToken);
            }
            catch (Exception ex)
            {
                logger.LogError(ex, "Error processing event {EventId}", eventItem.Id);
            }
        }
    }

    private async IAsyncEnumerable<EventItem> GetEventsAsync(
        [EnumeratorCancellation] CancellationToken cancellationToken)
    {
        // Stream d'événements
        while (!cancellationToken.IsCancellationRequested)
        {
            // Récupérer et yield les événements
            yield return new EventItem();

            await Task.Delay(optionsMonitor.CurrentValue.PollingInterval, cancellationToken);
        }
    }
}
```

## Performance et optimisation

### Conseils pour optimiser les patterns .NET

1. **Réutiliser les ressources** : Pooling des objets coûteux
2. **Éviter les allocations** : Utiliser Span<T> et Memory<T>
3. **Configurer appropriément** : Lifetimes DI, sizes de pools
4. **Mesurer les performances** : BenchmarkDotNet pour valider
5. **Gérer la mémoire** : ArrayPool, ObjectPool pour réduire le GC

```csharp
// Exemple d'optimisation avec pooling
public class OptimizedProcessor : IDisposable
{
    private readonly ObjectPool<StringBuilder> stringBuilderPool;
    private readonly ArrayPool<byte> byteArrayPool;

    public OptimizedProcessor()
    {
        stringBuilderPool = new DefaultObjectPool<StringBuilder>(
            new StringBuilderPooledObjectPolicy());
        byteArrayPool = ArrayPool<byte>.Shared;
    }

    public async Task<string> ProcessDataAsync(ReadOnlyMemory<byte> data)
    {
        // Utiliser un StringBuilder du pool
        var sb = stringBuilderPool.Get();
        try
        {
            // Traiter avec Span pour éviter les allocations
            var span = data.Span;

            // Traitement...

            return sb.ToString();
        }
        finally
        {
            stringBuilderPool.Return(sb);
        }
    }
}
```

## Conclusion

Les patterns spécifiques à .NET sont essentiels pour :

1. **Maximiser les performances** des applications
2. **Gérer les ressources** de manière déterministe
3. **Structurer la configuration** de façon maintenable
4. **Créer des applications asynchrones** robustes
5. **Intégrer efficacement** avec l'écosystème .NET

### Points clés à retenir :

- **Disposable** est crucial pour la gestion des ressources
- **Async/await** est le standard pour toutes les opérations I/O
- **Options** simplifie et structure la configuration
- **Factory avec DI** permet la flexibilité sans sacrifier la testabilité

### Prochaines étapes :

1. **Pratiquez** avec des projets réels
2. **Mesurez** les performances de vos implémentations
3. **Explorez** les nuances avancées de chaque pattern
4. **Intégrez** ces patterns dans votre architecture
5. **Partagez** vos expériences avec la communauté

Ces patterns forment la **base d'une architecture .NET moderne et performante**. Maîtrisez-les pour créer des applications robustes et maintenables qui tirent pleinement parti de l'écosystème .NET.

⏭️ 20. [Projets pratiques](/20-projets-pratiques/README.md)
