# 12.8. Middleware et pipeline de requêtes

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 12.8.1. Concept de middleware

### Qu'est-ce qu'un middleware ?

Le middleware est un composant logiciel qui s'intercale dans le pipeline de traitement des requêtes HTTP d'une application ASP.NET Core. Imaginez une chaîne d'assemblage où chaque "station" (middleware) reçoit un colis (la requête HTTP), y effectue une opération spécifique, puis le transmet à la station suivante.

![Pipeline de middleware](https://raw.githubusercontent.com/dotnet/docs/main/docs/core/extensions/media/middleware-pipeline.svg)

### Caractéristiques principales du middleware

1. **Traitement séquentiel** : Les middlewares sont exécutés dans l'ordre où ils sont ajoutés au pipeline.
2. **Contrôle du pipeline** : Chaque middleware peut :
   - Traiter la requête entrante
   - Passer la requête au middleware suivant dans la chaîne
   - Court-circuiter le pipeline (ne pas appeler le middleware suivant)
   - Traiter la réponse sortante après le middleware suivant
3. **Bidirectionnel** : Le middleware peut traiter à la fois la requête (entrante) et la réponse (sortante).

### Le concept de "délégué suivant"

Dans ASP.NET Core, chaque middleware reçoit une référence au middleware suivant dans la chaîne, appelé "délégué suivant" (next delegate). Le middleware peut décider d'appeler ou non ce délégué.

```
Requête → Middleware 1 → Middleware 2 → Middleware 3 → Traitement final
                                                       ↓
Réponse ← Middleware 1 ← Middleware 2 ← Middleware 3 ←
```

### Pourquoi utiliser des middlewares ?

Les middlewares permettent de :
- Modulariser les fonctionnalités de votre application
- Réutiliser des composants dans différentes applications
- Contrôler précisément le traitement des requêtes HTTP
- Séparer les préoccupations (séparation des responsabilités)

## 12.8.2. Middleware intégré

ASP.NET Core fournit de nombreux middlewares prêts à l'emploi pour les tâches courantes. Voici les plus importants :

### UseExceptionHandler

Capture les exceptions non gérées et affiche une page d'erreur personnalisée ou retourne une réponse JSON en fonction de la configuration.

```csharp
// Dans Program.cs
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
```

### UseStaticFiles

Permet de servir des fichiers statiques (CSS, JavaScript, images, etc.) depuis le dossier `wwwroot`.

```csharp
app.UseStaticFiles();
```

### UseRouting

Configure le routage des requêtes HTTP vers les points de terminaison appropriés (contrôleurs, Razor Pages, etc.).

```csharp
app.UseRouting();
```

### UseAuthentication et UseAuthorization

Gèrent l'authentification et l'autorisation des utilisateurs.

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### UseCors

Configure les règles CORS (Cross-Origin Resource Sharing) pour permettre aux navigateurs d'accéder à votre API depuis différentes origines.

```csharp
app.UseCors("PolicyName");
```

### UseSession

Active le support des sessions pour stocker des données utilisateur temporaires.

```csharp
app.UseSession();
```

### UseResponseCompression

Compresse les réponses HTTP pour réduire la taille des données transmises.

```csharp
app.UseResponseCompression();
```

### UseHttpsRedirection

Redirige automatiquement les requêtes HTTP vers HTTPS.

```csharp
app.UseHttpsRedirection();
```

### UseEndpoints

Configure les points de terminaison pour le traitement des requêtes (mappages de routes).

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    endpoints.MapRazorPages();
});
```

### Ordre des middlewares

L'ordre d'ajout des middlewares est crucial pour le bon fonctionnement de votre application. Voici un ordre typique recommandé :

```csharp
// Exception Handler doit être au début
app.UseExceptionHandler("/Home/Error");
app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
app.UseSession();
app.UseResponseCompression();

// MapControllers, MapRazorPages, etc. à la fin
app.UseEndpoints(endpoints => {
    endpoints.MapControllers();
    endpoints.MapRazorPages();
});
```

Quelques règles à retenir :
- `UseExceptionHandler` doit être parmi les premiers pour capturer toutes les exceptions
- `UseStaticFiles` doit être avant `UseRouting`
- `UseAuthentication` doit être avant `UseAuthorization`
- `UseEndpoints` doit généralement être le dernier

## 12.8.3. Création de middleware personnalisé

Parfois, vous aurez besoin de créer votre propre middleware pour des fonctionnalités spécifiques. Il existe trois façons de créer un middleware personnalisé :

### 1. Middleware en ligne (fonction anonyme)

La méthode la plus simple pour des middlewares simples :

```csharp
// Dans Program.cs
app.Use(async (context, next) =>
{
    // Code exécuté avant le prochain middleware
    Console.WriteLine($"Requête entrante: {context.Request.Path}");

    // Appel du middleware suivant dans la chaîne
    await next();

    // Code exécuté après le prochain middleware
    Console.WriteLine($"Réponse sortante: {context.Response.StatusCode}");
});
```

### 2. Middleware en classe

Pour des middlewares plus complexes, créez une classe dédiée :

```csharp
// RequestLoggingMiddleware.cs
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            // Avant le prochain middleware
            _logger.LogInformation($"Début de traitement: {context.Request.Method} {context.Request.Path}");

            // Mesure du temps d'exécution
            var stopwatch = Stopwatch.StartNew();

            // Appel du middleware suivant
            await _next(context);

            stopwatch.Stop();

            // Après le prochain middleware
            _logger.LogInformation($"Fin de traitement: {context.Response.StatusCode} en {stopwatch.ElapsedMilliseconds}ms");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Une erreur s'est produite dans le pipeline");
            throw; // Relance l'exception pour qu'elle soit gérée par ExceptionHandler
        }
    }
}

// Méthode d'extension pour faciliter l'enregistrement
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}
```

Ensuite, enregistrez-le dans `Program.cs` :

```csharp
app.UseRequestLogging();
```

### 3. Middleware de terminal (Map/MapWhen)

Ce type de middleware intercepte et traite complètement certaines requêtes sans appeler le middleware suivant.

```csharp
// Traite toutes les requêtes vers /api/version
app.Map("/api/version", appBuilder =>
{
    appBuilder.Run(async context =>
    {
        await context.Response.WriteAsync("Version 1.0.0");
    });
});

// Traite les requêtes qui correspondent à une condition
app.MapWhen(context => context.Request.Query.ContainsKey("debug"),
    appBuilder =>
    {
        appBuilder.Run(async context =>
        {
            await context.Response.WriteAsync("Mode débogage activé");
        });
    });
```

### Exemple concret : Middleware d'authentification par API Key

Voici un exemple complet d'un middleware qui vérifie la présence d'une clé API dans les en-têtes HTTP :

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IConfiguration _configuration;
    private const string API_KEY_HEADER = "X-API-Key";

    public ApiKeyMiddleware(RequestDelegate next, IConfiguration configuration)
    {
        _next = next;
        _configuration = configuration;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Vérifier si le chemin commence par /api
        if (context.Request.Path.StartsWithSegments("/api"))
        {
            // Vérifier si l'en-tête API Key est présent
            if (!context.Request.Headers.TryGetValue(API_KEY_HEADER, out var apiKeyHeader))
            {
                context.Response.StatusCode = 401; // Unauthorized
                await context.Response.WriteAsync("API Key manquante");
                return; // Court-circuiter le pipeline
            }

            // Récupérer la clé API valide depuis la configuration
            var validApiKey = _configuration["ApiKey"];

            // Vérifier si la clé API est valide
            if (apiKeyHeader != validApiKey)
            {
                context.Response.StatusCode = 401; // Unauthorized
                await context.Response.WriteAsync("API Key invalide");
                return; // Court-circuiter le pipeline
            }
        }

        // Si tout est OK, passer au middleware suivant
        await _next(context);
    }
}

// Extension pour faciliter l'utilisation
public static class ApiKeyMiddlewareExtensions
{
    public static IApplicationBuilder UseApiKeyAuthentication(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<ApiKeyMiddleware>();
    }
}
```

Enregistrement dans `Program.cs` :

```csharp
// Placez-le avant UseAuthorization mais après UseRouting
app.UseRouting();
app.UseApiKeyAuthentication();
app.UseAuthorization();
```

## 12.8.4. Configuration du pipeline

La configuration du pipeline de middleware se fait principalement dans le fichier `Program.cs` (ou `Startup.cs` dans les versions antérieures).

### Structure de base de Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. CONFIGURATION DES SERVICES
// Ces services seront disponibles via l'injection de dépendances
builder.Services.AddControllersWithViews();
builder.Services.AddRazorPages();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddAuthentication();
builder.Services.AddAuthorization();

// Autres services...

var app = builder.Build();

// 2. CONFIGURATION DU PIPELINE DE MIDDLEWARE
// L'ordre est crucial!
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

// Middlewares personnalisés
app.UseRequestLogging();

// Points de terminaison
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
app.MapRazorPages();

app.Run();
```

### Middleware conditionnel

Vous pouvez ajouter des middlewares conditionnellement en fonction de l'environnement ou d'autres paramètres :

```csharp
if (app.Environment.IsDevelopment())
{
    // Middleware spécifique au développement
    app.UseDeveloperExceptionPage();
    app.UseWebAssemblyDebugging();
}
else
{
    // Middleware de production
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

// Middleware spécifique à certaines fonctionnalités
if (builder.Configuration.GetValue<bool>("EnableCompression"))
{
    app.UseResponseCompression();
}
```

### Configuration de middlewares avec options

Certains middlewares acceptent des options pour personnaliser leur comportement :

```csharp
// Configuration de CORS
app.UseCors(options =>
{
    options.WithOrigins("https://example.com")
           .AllowAnyMethod()
           .AllowAnyHeader();
});

// Configuration de la compression
app.UseResponseCompression(options =>
{
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
    options.EnableForHttps = true;
});

// Configuration des fichiers statiques avec cache
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        ctx.Context.Response.Headers.Append("Cache-Control", "public,max-age=3600");
    }
});
```

### Regroupement logique de middleware

Pour des applications complexes, vous pouvez organiser les middlewares en groupes logiques :

```csharp
// Middlewares de sécurité
app.UseHttpsRedirection();
app.UseHsts();
app.UseCors();

// Middlewares de contenu statique
app.UseStaticFiles();
app.UseWebAssemblyDebugging();

// Middlewares d'authentification
app.UseAuthentication();
app.UseAuthorization();

// Middlewares métier personnalisés
app.UseRequestLogging();
app.UseApiKeyAuthentication();

// Middlewares de routage et points de terminaison
app.UseRouting();
app.MapControllers();
app.MapRazorPages();
app.MapFallbackToFile("index.html");
```

## 12.8.5. Diagnostics et logging

Le diagnostic et la journalisation sont essentiels pour comprendre ce qui se passe dans votre application, surtout en production.

### Configuration du logging

ASP.NET Core intègre un système de journalisation flexible qui peut être configuré dans `Program.cs` :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuration de la journalisation
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.AddEventSourceLogger();

// Niveau de log par défaut
builder.Logging.SetMinimumLevel(LogLevel.Information);

// Niveaux de log spécifiques par catégorie
builder.Logging.AddFilter("Microsoft.AspNetCore.Mvc", LogLevel.Warning);
builder.Logging.AddFilter("Microsoft.EntityFrameworkCore", LogLevel.Warning);

// Ajouter Serilog (nécessite le package NuGet)
builder.Logging.AddSerilog(new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day)
    .CreateLogger());

// Reste de la configuration...
```

### Middleware de diagnostic

ASP.NET Core propose plusieurs middlewares de diagnostic utiles :

#### 1. UseDeveloperExceptionPage

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

#### 2. UseStatusCodePages

Affiche des messages pour les codes d'erreur HTTP comme 404 (Not Found) :

```csharp
app.UseStatusCodePages(); // Version simple

// Version plus élaborée
app.UseStatusCodePages(async context =>
{
    context.HttpContext.Response.ContentType = "text/plain";
    await context.HttpContext.Response.WriteAsync(
        $"Page d'erreur : Code de statut {context.HttpContext.Response.StatusCode}");
});

// Redirection vers une page d'erreur
app.UseStatusCodePagesWithRedirects("/errors/{0}");

// Ré-exécution avec le code d'erreur préservé (meilleur pour SEO)
app.UseStatusCodePagesWithReExecute("/errors/{0}");
```

#### 3. Middleware de journalisation des requêtes

```csharp
// Dans Program.cs
app.Use(async (context, next) =>
{
    var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();
    var requestPath = context.Request.Path;
    var method = context.Request.Method;

    logger.LogInformation("Début de la requête {Method} {Path}", method, requestPath);

    var stopwatch = Stopwatch.StartNew();
    try
    {
        await next();
    }
    finally
    {
        stopwatch.Stop();
        var statusCode = context.Response.StatusCode;
        var level = statusCode >= 500 ? LogLevel.Error :
                    statusCode >= 400 ? LogLevel.Warning :
                    LogLevel.Information;

        logger.Log(level, "Fin de la requête {Method} {Path} => {StatusCode} en {ElapsedMs}ms",
            method, requestPath, statusCode, stopwatch.ElapsedMilliseconds);
    }
});
```

### Création d'un middleware de diagnostic personnalisé

Voici un exemple de middleware qui enregistre les métriques de performance de chaque requête :

```csharp
public class PerformanceMetricsMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<PerformanceMetricsMiddleware> _logger;
    private readonly DiagnosticListener _diagnosticListener;

    public PerformanceMetricsMiddleware(
        RequestDelegate next,
        ILogger<PerformanceMetricsMiddleware> logger,
        DiagnosticListener diagnosticListener)
    {
        _next = next;
        _logger = logger;
        _diagnosticListener = diagnosticListener;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var startTime = Stopwatch.GetTimestamp();
        var initialMemory = GC.GetTotalMemory(false);

        // Collecte des informations sur la requête
        var path = context.Request.Path;
        var method = context.Request.Method;

        try
        {
            // Appel du middleware suivant
            await _next(context);
        }
        finally
        {
            var endTime = Stopwatch.GetTimestamp();
            var finalMemory = GC.GetTotalMemory(false);

            // Calcul des métriques
            var elapsed = TimeSpan.FromTicks(endTime - startTime);
            var memoryDelta = finalMemory - initialMemory;
            var statusCode = context.Response.StatusCode;

            // Journalisation des métriques
            _logger.LogInformation(
                "Requête {Method} {Path} => {StatusCode} en {ElapsedMs}ms (Mémoire: {MemoryDelta} octets)",
                method, path, statusCode, elapsed.TotalMilliseconds, memoryDelta);

            // Émission d'un événement diagnostic pour d'autres outils
            if (_diagnosticListener.IsEnabled("RequestPerformance"))
            {
                _diagnosticListener.Write("RequestPerformance", new
                {
                    Path = path,
                    Method = method,
                    StatusCode = statusCode,
                    Elapsed = elapsed,
                    MemoryDelta = memoryDelta
                });
            }
        }
    }
}

// Extension
public static class PerformanceMetricsMiddlewareExtensions
{
    public static IApplicationBuilder UsePerformanceMetrics(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<PerformanceMetricsMiddleware>();
    }
}
```

### Middleware de Health Checks

Les health checks permettent de surveiller l'état de santé de votre application et de ses dépendances :

```csharp
// Dans la section services
builder.Services.AddHealthChecks()
    .AddSqlServer(connectionString)
    .AddUrlGroup(new Uri("https://api.externe.com/health"), name: "api-externe")
    .AddCheck("Espace disque", () =>
    {
        var freeSpace = GetFreeDiskSpace();
        return freeSpace < 1024 * 1024 * 100
            ? HealthCheckResult.Degraded($"Espace disque faible: {freeSpace} octets")
            : HealthCheckResult.Healthy();
    });

// Dans la section middleware
app.UseHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";

        var result = new
        {
            status = report.Status.ToString(),
            results = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description,
                data = e.Value.Data
            })
        };

        await context.Response.WriteAsJsonAsync(result);
    }
});
```

### Intégration avec des outils de monitoring

Vous pouvez intégrer votre application avec des outils de monitoring comme Application Insights :

```csharp
// Dans Program.cs
builder.Services.AddApplicationInsightsTelemetry(builder.Configuration["ApplicationInsights:ConnectionString"]);
```

## Résumé

Le middleware est un concept fondamental dans ASP.NET Core qui permet de construire un pipeline de traitement modulaire pour les requêtes HTTP. Chaque middleware a une responsabilité spécifique et peut être réutilisé dans différentes applications.

Points clés à retenir :
- L'ordre des middlewares est crucial
- Chaque middleware peut traiter la requête avant et après l'appel au middleware suivant
- Un middleware peut court-circuiter le pipeline
- Créez des middlewares personnalisés pour encapsuler la logique métier
- Utilisez les outils de diagnostic et de journalisation pour surveiller votre application

En maîtrisant les concepts de middleware et de pipeline, vous pourrez construire des applications ASP.NET Core robustes, modulaires et faciles à maintenir.

## Exercices pratiques

1. Créez un middleware qui mesure le temps d'exécution de chaque requête et l'affiche dans les en-têtes de réponse.
2. Implémentez un middleware qui limite le nombre de requêtes par IP (rate limiting).
3. Développez un middleware qui ajoute automatiquement des en-têtes de sécurité à toutes les réponses.
4. Créez un middleware qui enregistre toutes les exceptions dans une base de données.
5. Implémentez un middleware qui redimensionne automatiquement les images téléchargées.

⏭️ 13. [Applications de bureau modernes avec .NET](/13-applications-de-bureau-modernes-avec-dotnet/README.md)
