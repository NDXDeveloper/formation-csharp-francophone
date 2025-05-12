# 12.3. ASP.NET Core

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 12.3.1. Architecture et principes

ASP.NET Core représente une refonte complète du framework ASP.NET traditionnel. Il s'agit d'un framework web open-source, multiplateforme et à haute performance, conçu pour construire des applications web modernes, connectées au cloud et basées sur des API.

### Principes fondamentaux

#### Open-Source et communautaire

ASP.NET Core est entièrement open-source et hébergé sur GitHub. Cela permet :
- Une transparence totale dans le développement
- Des contributions de la communauté mondiale
- Des cycles de développement et de correction plus rapides
- Une meilleure qualité grâce à l'examen par les pairs

#### Multiplateforme

Contrairement à ASP.NET Framework qui était limité à Windows, ASP.NET Core peut s'exécuter sur :
- Windows
- macOS
- Linux
- Conteneurs Docker
- Architectures ARM (Raspberry Pi, etc.)

Cette portabilité élargit considérablement les options d'hébergement et de développement.

#### Modulaire

ASP.NET Core adopte une approche modulaire où toutes les fonctionnalités sont des packages NuGet optionnels. Cette conception apporte plusieurs avantages :
- Applications plus légères (vous n'incluez que ce dont vous avez besoin)
- Mises à jour indépendantes des différents composants
- Flexibilité pour remplacer des composants par des alternatives
- Réduction des dépendances non utilisées

#### Performance

ASP.NET Core a été conçu en mettant l'accent sur la performance :
- Compilation JIT améliorée
- Réduction des allocations de mémoire
- Middleware à faible overhead
- Gestion efficace des requêtes asynchrones
- Support natif pour les technologies modernes comme HTTP/2

Des tests de performance montrent qu'ASP.NET Core est l'un des frameworks web les plus rapides, surpassant souvent Node.js et Java dans les benchmarks TechEmpower.

### Architecture d'ASP.NET Core

#### Pipeline de requêtes basé sur le middleware

Le cœur d'ASP.NET Core est un pipeline de traitement des requêtes HTTP basé sur le concept de middleware.

Un middleware est un composant logiciel qui traite les requêtes HTTP et les réponses. Chaque middleware :
- Reçoit une requête HTTP
- Peut la traiter ou la modifier
- Peut passer la requête au middleware suivant ou court-circuiter le pipeline
- Peut traiter la réponse après que les middlewares suivants ont terminé

Cette architecture en "oignon" offre une grande flexibilité pour configurer le comportement de l'application.

```csharp
// Configuration du pipeline de middleware dans Program.cs (ASP.NET Core 6+)
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware d'exceptions
app.UseExceptionHandler("/Error");

// Middleware pour HTTPS
app.UseHttpsRedirection();

// Middleware pour fichiers statiques
app.UseStaticFiles();

// Middleware de routage
app.UseRouting();

// Middleware d'authentification et d'autorisation
app.UseAuthentication();
app.UseAuthorization();

// Middleware d'endpoints (contrôleurs, Razor Pages, etc.)
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    endpoints.MapRazorPages();
});

app.Run();
```

#### Serveur web intégré (Kestrel)

ASP.NET Core inclut son propre serveur web appelé Kestrel. Contrairement à ASP.NET traditionnel qui nécessitait IIS, Kestrel permet :
- De s'exécuter sur toutes les plateformes
- D'être utilisé comme serveur autonome
- D'être placé derrière un proxy inverse (IIS, Nginx, Apache)
- D'obtenir d'excellentes performances

#### Architecture de l'hôte

ASP.NET Core utilise un modèle d'hôte générique qui fournit :
- Gestion du cycle de vie de l'application
- Configuration (fichiers JSON, variables d'environnement, etc.)
- Injection de dépendances
- Journalisation
- Gestion des environnements (Dev, Staging, Production)

Cette architecture facilite le développement et les tests en fournissant une base commune pour tous les types d'applications.

#### Modèles d'application

ASP.NET Core prend en charge plusieurs modèles d'application :

1. **MVC (Model-View-Controller)** :
   - Architecture classique séparant les données, la logique et l'interface
   - Prise en charge complète des vues Razor, des API, etc.

2. **Razor Pages** :
   - Modèle simplifié basé sur les pages
   - Chaque page contient à la fois le HTML et le code C#
   - Plus simple pour les petites applications

3. **API Web** :
   - Optimisé pour créer des API RESTful
   - Support intégré pour JSON, XML, etc.
   - Documentation automatique via Swagger/OpenAPI

4. **Blazor** :
   - UI web interactive avec C# au lieu de JavaScript
   - Blazor Server ou Blazor WebAssembly

5. **gRPC** :
   - Services à hautes performances utilisant Protocol Buffers
   - Communication efficace entre services

### Différences clés avec ASP.NET Framework

| Aspect | ASP.NET Framework | ASP.NET Core |
|--------|-------------------|--------------|
| Plateforme | Windows uniquement | Windows, macOS, Linux |
| Dépendances | Monolithique (.NET Framework) | Modulaire (packages NuGet) |
| Déploiement | Assemblies GAC, IIS | Autonome ou hébergé par IIS, Nginx, etc. |
| Architecture | Système.Web couplé | Pipeline de middleware flexible |
| Performance | Bon | Excellent (2-10x plus rapide) |
| Open Source | Partiellement | Entièrement |
| Modèles | WebForms, MVC, Web API | MVC, Razor Pages, Blazor, gRPC |
| Configuration | Principalement Web.config | JSON, variables d'environnement, etc. |

## 12.3.2. Configuration et environnements

La configuration dans ASP.NET Core est flexible, extensible et indépendante de la plateforme. Elle utilise un système qui s'adapte à différents environnements de déploiement sans recompilation.

### Système de configuration

#### Fournisseurs de configuration

ASP.NET Core prend en charge plusieurs sources de configuration qui sont lues dans un ordre spécifique, les dernières écrasant les précédentes :

1. **Paramètres par défaut**
2. **Fichiers de configuration** (appsettings.json)
3. **Fichiers de configuration spécifiques à l'environnement** (appsettings.Development.json)
4. **Secret Manager** (développement local uniquement)
5. **Variables d'environnement**
6. **Arguments de ligne de commande**

Ce système offre une grande flexibilité, permettant de configurer une application différemment selon qu'elle s'exécute en développement, en test ou en production.

#### Fichiers appsettings.json

Les fichiers JSON sont la méthode de configuration la plus courante :

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=myserver;Database=mydb;User=myuser;Password=mypassword;"
  },
  "EmailSettings": {
    "SmtpServer": "smtp.example.com",
    "SmtpPort": 587,
    "SenderEmail": "noreply@example.com"
  },
  "FeatureFlags": {
    "NewUserInterface": false,
    "BetaFeatures": true
  }
}
```

Pour chaque environnement, vous pouvez créer un fichier spécifique qui remplace partiellement la configuration de base :

```json
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=mydb_dev;Trusted_Connection=True;"
  },
  "FeatureFlags": {
    "BetaFeatures": true,
    "NewUserInterface": true
  }
}
```

#### Variables d'environnement

Les variables d'environnement peuvent remplacer n'importe quel paramètre de configuration, ce qui est particulièrement utile en production et dans les environnements cloud :

```
# Configuration simple
LOGGING__LOGLEVEL__DEFAULT=Warning

# Chaîne de connexion
ConnectionStrings__DefaultConnection=Server=prod-server;Database=mydb;User=myuser;Password=mypassword;
```

Notez l'utilisation de `__` (double underscore) pour représenter la hiérarchie dans le JSON.

#### Accès à la configuration

Dans ASP.NET Core, l'accès à la configuration se fait généralement via l'injection de dépendances :

```csharp
public class EmailService
{
    private readonly IConfiguration _configuration;

    public EmailService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void SendEmail(string to, string subject, string body)
    {
        // Accès à des valeurs individuelles
        var smtpServer = _configuration["EmailSettings:SmtpServer"];
        var smtpPort = _configuration.GetValue<int>("EmailSettings:SmtpPort");
        var senderEmail = _configuration["EmailSettings:SenderEmail"];

        // Utilisation des paramètres...
    }
}
```

#### Classes de configuration typées

Pour une approche plus robuste, vous pouvez lier la configuration à des classes typées :

```csharp
// Classe qui représente une section de configuration
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int SmtpPort { get; set; }
    public string SenderEmail { get; set; }
    public string SenderName { get; set; }
    public bool EnableSsl { get; set; } = true;
}

// Dans Program.cs ou Startup.cs
services.Configure<EmailSettings>(Configuration.GetSection("EmailSettings"));

// Utilisation dans un service
public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }

    public void SendEmail(string to, string subject, string body)
    {
        // Accès typé aux paramètres
        var smtpServer = _settings.SmtpServer;
        var smtpPort = _settings.SmtpPort;

        // Utilisation des paramètres...
    }
}
```

### Gestion des environnements

ASP.NET Core reconnaît différents environnements d'exécution, généralement :
- **Development** (développement local)
- **Staging** (préproduction)
- **Production** (environnement de production)

#### Définition de l'environnement

L'environnement est défini via la variable d'environnement `ASPNETCORE_ENVIRONMENT`. ASP.NET Core la lit au démarrage et configure l'application en conséquence.

Sous Windows, vous pouvez définir cette variable via :
```
setx ASPNETCORE_ENVIRONMENT "Development"
```

Sous Linux/macOS :
```
export ASPNETCORE_ENVIRONMENT=Development
```

Dans Visual Studio, cette valeur est définie dans les propriétés du projet ou le fichier launchSettings.json.

#### Configuration spécifique à l'environnement

Vous pouvez adapter le comportement de l'application en fonction de l'environnement :

```csharp
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Configuration spécifique à l'environnement
if (app.Environment.IsDevelopment())
{
    // Afficher des pages d'erreur détaillées en développement
    app.UseDeveloperExceptionPage();
}
else
{
    // Page d'erreur générique en production
    app.UseExceptionHandler("/Error");
    // HTTPS strict en production
    app.UseHsts();
}

// Configuration commune
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
```

Cette approche permet d'avoir un comportement approprié dans chaque environnement :
- Plus de détails et d'outils de débogage en développement
- Plus de sécurité et de stabilité en production

#### Fichiers de configuration par environnement

ASP.NET Core charge automatiquement le fichier de configuration correspondant à l'environnement actuel :
- appsettings.json (base)
- appsettings.Development.json (en développement)
- appsettings.Production.json (en production)
- appsettings.Staging.json (en préproduction)

Vous pouvez également créer vos propres environnements personnalisés (QA, UAT, etc.) en définissant la variable ASPNETCORE_ENVIRONMENT correspondante.

#### User Secrets

Pour le développement local, .NET Core fournit un mécanisme appelé "User Secrets" pour stocker des informations sensibles sans les mettre dans le code source :

```bash
# Dans le répertoire du projet
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=(localdb)\\mssqllocaldb;Database=myapp;Trusted_Connection=True;"
dotnet user-secrets set "APIKeys:ExternalService" "abcdef123456"
```

Ces secrets sont stockés dans un fichier JSON en dehors du répertoire du projet et sont automatiquement chargés en environnement de développement.

## 12.3.3. Dependency Injection intégrée

L'injection de dépendances (DI) est un modèle de conception qui permet de rendre le code plus modulaire, testable et maintenable. Contrairement à ASP.NET Framework où il fallait utiliser des bibliothèques tierces, ASP.NET Core intègre nativement un conteneur d'injection de dépendances.

### Principes de l'injection de dépendances

L'injection de dépendances repose sur quelques principes clés :

1. **Inversion de contrôle (IoC)** : Les objets ne créent pas leurs dépendances, mais les reçoivent de l'extérieur.
2. **Dépendances explicites** : Les dépendances sont déclarées dans le constructeur, rendant les relations claires.
3. **Interface vs implémentation** : On dépend d'abstractions (interfaces) plutôt que d'implémentations concrètes.

Ces principes favorisent un couplage faible entre les composants, facilitant les tests et permettant de remplacer facilement des implémentations.

### Configuration du conteneur DI

Dans ASP.NET Core, la configuration du conteneur DI se fait généralement dans la méthode `ConfigureServices` de `Startup.cs` ou directement dans `Program.cs` pour les versions récentes :

```csharp
// ASP.NET Core 6+
var builder = WebApplication.CreateBuilder(args);

// Enregistrement des services dans le conteneur DI
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddSingleton<IConfiguration>(builder.Configuration);

// Configuration des services système
builder.Services.AddControllers();
builder.Services.AddRazorPages();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### Durées de vie des services

ASP.NET Core propose trois durées de vie principales pour les services :

1. **Transient (transitoire)** :
   - Un nouveau service est créé chaque fois qu'il est demandé
   - Idéal pour les services légers et sans état
   - Exemple : `services.AddTransient<IEmailService, EmailService>();`

2. **Scoped (limité à la portée)** :
   - Un service est créé une fois par requête HTTP
   - Tous les composants d'une même requête reçoivent la même instance
   - Parfait pour les services qui doivent maintenir un état pendant la requête
   - Exemple : `services.AddScoped<IUserRepository, UserRepository>();`

3. **Singleton** :
   - Un service est créé une seule fois pour toute la durée de vie de l'application
   - Tous les composants de toutes les requêtes partagent la même instance
   - Utile pour les services partagés et sans état entre les requêtes
   - Exemple : `services.AddSingleton<ICacheService, MemoryCacheService>();`

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Transient : nouvelle instance à chaque demande
    services.AddTransient<IEmailService, EmailService>();

    // Scoped : une instance par requête HTTP
    services.AddScoped<IUserRepository, UserRepository>();
    services.AddScoped<ShoppingCart>();

    // Singleton : une seule instance pour toute l'application
    services.AddSingleton<IGlobalCounter, GlobalCounter>();
    services.AddSingleton<IConfiguration>(Configuration);
}
```

### Injection dans les classes

Une fois les services enregistrés, ils peuvent être injectés dans les classes qui en ont besoin :

#### Injection dans les contrôleurs

```csharp
public class UserController : Controller
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService;

    // Les dépendances sont injectées automatiquement
    public UserController(IUserRepository userRepository, IEmailService emailService)
    {
        _userRepository = userRepository;
        _emailService = emailService;
    }

    public async Task<IActionResult> Register(RegisterViewModel model)
    {
        if (ModelState.IsValid)
        {
            var user = new User { Email = model.Email, Name = model.Name };
            await _userRepository.AddUserAsync(user);

            await _emailService.SendWelcomeEmailAsync(user.Email);

            return RedirectToAction("Index", "Home");
        }

        return View(model);
    }
}
```

#### Injection dans les services

```csharp
public class EmailService : IEmailService
{
    private readonly ILogger<EmailService> _logger;
    private readonly EmailSettings _settings;

    public EmailService(ILogger<EmailService> logger, IOptions<EmailSettings> settings)
    {
        _logger = logger;
        _settings = settings.Value;
    }

    public async Task SendWelcomeEmailAsync(string email)
    {
        _logger.LogInformation("Envoi d'un email de bienvenue à {Email}", email);
        // Logique d'envoi d'email...
    }
}
```

#### Injection dans les vues Razor

```cshtml
@inject IUserService UserService

<h1>Utilisateurs actifs</h1>

<ul>
    @foreach (var user in await UserService.GetActiveUsersAsync())
    {
        <li>@user.Name (@user.Email)</li>
    }
</ul>
```

### Services intégrés

ASP.NET Core fournit de nombreux services intégrés qui peuvent être injectés dans vos classes :

- `IConfiguration` : Accès à la configuration
- `ILogger<T>` : Journalisation typée
- `IHostEnvironment` : Informations sur l'environnement d'exécution
- `IWebHostEnvironment` : Informations spécifiques à l'environnement web
- `IHttpContextAccessor` : Accès au HttpContext
- `IOptionsMonitor<T>`, `IOptions<T>` : Configuration typée

### Injection de dépendances avancée

#### Enregistrement de services par convention

Pour éviter d'enregistrer manuellement chaque service, vous pouvez utiliser des techniques d'enregistrement automatique :

```csharp
// Enregistrement automatique de tous les services dans l'assembly
public static void AddServicesInAssembly(this IServiceCollection services)
{
    var assembly = Assembly.GetExecutingAssembly();

    var serviceTypes = assembly.GetTypes()
        .Where(t => t.IsClass && !t.IsAbstract && t.Name.EndsWith("Service"));

    foreach (var serviceType in serviceTypes)
    {
        var interfaceType = serviceType.GetInterfaces()
            .FirstOrDefault(i => i.Name == "I" + serviceType.Name);

        if (interfaceType != null)
        {
            services.AddScoped(interfaceType, serviceType);
        }
    }
}

// Utilisation
services.AddServicesInAssembly();
```

#### Injection de dépendances dans les filtres et autres composants

Pour les composants qui ne sont pas créés directement par le conteneur DI (comme les filtres), il existe des attributs spéciaux :

```csharp
[TypeFilter(typeof(CustomExceptionFilter))]
public IActionResult Index()
{
    return View();
}

// Ou
[ServiceFilter(typeof(CustomActionFilter))]
public IActionResult Details(int id)
{
    return View();
}
```

Pour utiliser `ServiceFilter`, le filtre doit être enregistré dans le conteneur DI :

```csharp
services.AddScoped<CustomActionFilter>();
```

#### Factory et ServiceProvider

Pour des scénarios avancés, vous pouvez utiliser des factory ou l'IServiceProvider directement :

```csharp
// Enregistrement avec factory
services.AddTransient<IEmailService>(sp =>
{
    var logger = sp.GetRequiredService<ILogger<EmailService>>();
    var config = sp.GetRequiredService<IConfiguration>();

    if (config["EmailProvider"] == "SendGrid")
        return new SendGridEmailService(logger, config);
    else
        return new SmtpEmailService(logger, config);
});

// Résolution manuelle (à éviter si possible)
app.Run(async context =>
{
    var serviceProvider = context.RequestServices;
    var emailService = serviceProvider.GetRequiredService<IEmailService>();

    await emailService.SendEmailAsync("example@example.com", "Test", "Message de test");
});
```

## 12.3.4. Hébergement et déploiement

ASP.NET Core offre de nombreuses options de déploiement, ce qui le rend flexible pour divers environnements d'hébergement, du développement local aux infrastructures cloud.

### Le serveur Kestrel

Kestrel est le serveur web multiplateforme intégré dans ASP.NET Core :

- Extrêmement performant et léger
- S'exécute sur Windows, macOS et Linux
- Peut fonctionner seul ou derrière un proxy inverse

Kestrel est automatiquement configuré dans la méthode `CreateHostBuilder` de `Program.cs` :

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseKestrel(options =>
            {
                // Configuration personnalisée de Kestrel
                options.Limits.MaxConcurrentConnections = 100;
                options.Limits.MaxRequestBodySize = 10 * 1024 * 1024; // 10 MB
                options.Listen(IPAddress.Any, 5000); // HTTP
                options.Listen(IPAddress.Any, 5001, listenOptions =>
                {
                    listenOptions.UseHttps("certificate.pfx", "password");
                }); // HTTPS
            });
            webBuilder.UseStartup<Startup>();
        });
```

### Options de déploiement

#### Déploiement autonome vs dépendant du framework

ASP.NET Core offre deux modèles de déploiement principaux :

1. **Dépendant du framework (FDD - Framework Dependent Deployment)** :
   - Nécessite que .NET Core Runtime soit installé sur la machine cible
   - Taille de déploiement beaucoup plus petite
   - Les applications peuvent bénéficier des mises à jour du runtime
   ```bash
   dotnet publish -c Release
   ```

2. **Autonome (SCD - Self Contained Deployment)** :
   - Inclut le runtime .NET Core dans le déploiement
   - Fonctionne sans installation préalable sur la machine cible
   - Taille de déploiement plus importante
   - Spécifique à un système d'exploitation
   ```bash
   dotnet publish -c Release -r win-x64 --self-contained
   ```

#### Trimming et AOT (Ahead-of-Time) Compilation

Pour optimiser davantage les déploiements autonomes :

1. **Trimming** : Supprime les parties inutilisées du framework
   ```bash
   dotnet publish -c Release -r win-x64 --self-contained /p:PublishTrimmed=true
   ```

2. **AOT Compilation** : Précompile l'application pour améliorer les performances de démarrage
   ```bash
   dotnet publish -c Release -r win-x64 --self-contained /p:PublishAot=true
   ```

### Déploiement sur différentes plateformes

#### Windows - IIS

IIS (Internet Information Services) est le serveur web intégré de Windows.

1. **Prérequis** :
   - Installer le module ASP.NET Core Hosting Bundle
   - Configurer le pool d'applications (.NET CLR Version = "No Managed Code")

2. **Publication** :
   ```bash
   dotnet publish -c Release
   ```

3. **Web.config** (généré automatiquement) :
   ```xml
   <configuration>
     <location path="." inheritInChildApplications="false">
       <system.webServer>
         <handlers>
           <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
         </handlers>
         <aspNetCore processPath="dotnet" arguments=".\MyApp.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
       </system.webServer>
     </location>
   </configuration>
   ```

4. **Options d'hébergement** :
   - **InProcess** : Plus performant, s'exécute dans le processus IIS
   - **OutOfProcess** : Plus isolé, IIS agit comme proxy inverse devant Kestrel

#### Linux - Nginx/Apache

Sur Linux, ASP.NET Core s'exécute généralement derrière un proxy inverse comme Nginx ou Apache.

1. **Service systemd** :
   ```ini
   [Unit]
   Description=My ASP.NET Core Application

   [Service]
   WorkingDirectory=/var/www/myapp
   ExecStart=/usr/bin/dotnet /var/www/myapp/MyApp.dll
   Restart=always
   RestartSec=10
   SyslogIdentifier=myapp
   User=www-data
   Environment=ASPNETCORE_ENVIRONMENT=Production

   [Install]
   WantedBy=multi-user.target
   ```

2. **Configuration Nginx** :
   ```nginx
   server {
       listen 80;
       server_name example.com;

       location / {
           proxy_pass http://localhost:5000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection keep-alive;
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

#### Conteneurs Docker

Docker est une option populaire pour déployer des applications ASP.NET Core de manière cohérente.

1. **Dockerfile** :
   ```dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
   WORKDIR /app
   EXPOSE 80
   EXPOSE 443

   FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
   WORKDIR /src
   COPY ["MyApp.csproj", "./"]
   RUN dotnet restore "MyApp.csproj"
   COPY . .
   RUN dotnet build "MyApp.csproj" -c Release -o /app/build

   FROM build AS publish
   RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

   FROM base AS final
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "MyApp.dll"]
   ```

2. **Compilation et exécution** :
   ```bash
   docker build -t myapp .
   docker run -d -p 8080:80 --name myapp_container myapp
   ```

### Déploiement Cloud

#### Microsoft Azure

Azure offre plusieurs options pour héberger des applications ASP.NET Core :

1. **Azure App Service** :
   - Service PaaS (Platform as a Service)
   - Mise à l'échelle et gestion automatisées
   - Intégration avec d'autres services Azure
   ```bash
   # Déploiement via CLI Azure
   az webapp up --sku F1 --name myappservice --resource-group myresourcegroup
   ```

2. **Azure Kubernetes Service (AKS)** :
   - Pour les applications conteneurisées
   - Orchestration avancée et mise à l'échelle
   - Adapté aux architectures microservices

3. **Azure Container Instances** :
   - Déploiement rapide de conteneurs sans gestion de cluster
   - Idéal pour les applications simples ou les tâches ponctuelles

#### AWS (Amazon Web Services)

AWS propose également plusieurs options :

1. **Elastic Beanstalk** :
   - Similaire à Azure App Service
   - Déploiement, mise à l'échelle et gestion automatisés

2. **ECS (Elastic Container Service)** ou **EKS (Elastic Kubernetes Service)** :
   - Pour les applications conteneurisées
   - Orchestration et gestion

3. **Lambda avec API Gateway** :
   - Serverless pour les fonctions ASP.NET Core minimalistes

#### Déploiement continu (CI/CD)

L'intégration avec des pipelines CI/CD est essentielle pour des déploiements fiables :

1. **GitHub Actions** :
   ```yaml
   name: .NET Core CI/CD

   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]

   jobs:
     build-and-deploy:
       runs-on: ubuntu-latest

       steps:
       - uses: actions/checkout@v2

       - name: Setup .NET Core
         uses: actions/setup-dotnet@v1
         with:
           dotnet-version: 6.0.x

       - name: Restore dependencies
         run: dotnet restore

       - name: Build
         run: dotnet build --configuration Release --no-restore

       - name: Test
         run: dotnet test --no-restore --verbosity normal

       - name: Publish
         run: dotnet publish --configuration Release --no-build -o ./publish

       - name: Deploy to Azure Web App
         uses: azure/webapps-deploy@v2
         with:
           app-name: 'my-app-name'
           publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
           package: ./publish
   ```

2. **Azure DevOps Pipelines** :
   ```yaml
   trigger:
   - main

   pool:
     vmImage: 'ubuntu-latest'

   variables:
     buildConfiguration: 'Release'

   steps:
   - task: UseDotNet@2
     inputs:
       packageType: 'sdk'
       version: '6.0.x'

   - task: DotNetCoreCLI@2
     inputs:
       command: 'restore'
       projects: '**/*.csproj'

   - task: DotNetCoreCLI@2
     inputs:
       command: 'build'
       projects: '**/*.csproj'
       arguments: '--configuration $(buildConfiguration) --no-restore'

   - task: DotNetCoreCLI@2
     inputs:
       command: 'test'
       projects: '**/*Tests/*.csproj'
       arguments: '--configuration $(buildConfiguration) --no-restore'

   - task: DotNetCoreCLI@2
     inputs:
       command: 'publish'
       publishWebProjects: true
       arguments: '--configuration $(buildConfiguration) --no-build --output $(Build.ArtifactStagingDirectory)'

   - task: AzureWebApp@1
     inputs:
       azureSubscription: 'Your-Azure-Subscription'
       appName: 'your-app-service-name'
       package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
   ```

### Meilleures pratiques de déploiement

1. **Automatiser tout** : Utiliser des pipelines CI/CD pour éviter les erreurs manuelles
2. **Versioning** : Marquer chaque build avec un numéro de version
3. **Tests automatisés** : Exécuter des tests avant chaque déploiement
4. **Déploiements bleu/vert** : Maintenir deux environnements identiques et basculer le trafic
5. **Surveillance** : Mettre en place une surveillance et des alertes après déploiement
6. **Configuration externe** : Stocker les paramètres sensibles dans des coffres sécurisés
7. **Journalisation centralisée** : Configurer un système central de journalisation
8. **Sauvegarde et plan de restauration** : Prévoir des procédures de récupération en cas d'échec

## 12.3.5. Blazor (Server et WebAssembly)

Blazor est un framework moderne de Microsoft qui permet de créer des applications web interactives utilisant C# au lieu de JavaScript. Il existe en deux variantes : Blazor Server et Blazor WebAssembly.

### Concepts fondamentaux de Blazor

#### Composants

Blazor est basé sur le concept de composants, similaire à des frameworks comme React ou Angular. Un composant Blazor :
- Est un élément d'interface utilisateur réutilisable
- Combine le balisage HTML avec la logique C#
- Peut être imbriqué dans d'autres composants
- Est identifié par l'extension `.razor`

```razor
@* Exemple de composant simple *@
<div class="alert alert-info">
    <h4>@Title</h4>
    <p>@ChildContent</p>
    <button @onclick="OnCloseClick">Fermer</button>
</div>

@code {
    [Parameter]
    public string Title { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void OnCloseClick()
    {
        // Logique de fermeture
        Console.WriteLine("Bouton fermer cliqué!");
    }
}
```

#### Cycle de vie des composants

Les composants Blazor ont un cycle de vie similaire à d'autres frameworks UI :

```csharp
// Méthodes de cycle de vie
protected override void OnInitialized()
{
    // Appelé une seule fois lors de l'initialisation du composant
}

protected override async Task OnInitializedAsync()
{
    // Version asynchrone de OnInitialized
    await LoadDataAsync();
}

protected override void OnParametersSet()
{
    // Appelé quand les paramètres du composant sont définis
}

protected override async Task OnParametersSetAsync()
{
    // Version asynchrone de OnParametersSet
}

protected override void OnAfterRender(bool firstRender)
{
    // Appelé après chaque rendu
    if (firstRender)
    {
        // Code exécuté uniquement après le premier rendu
        JsInterop.InitializeComponent();
    }
}

protected override async Task OnAfterRenderAsync(bool firstRender)
{
    // Version asynchrone de OnAfterRender
}
```

#### Liaison de données (Data Binding)

Blazor prend en charge la liaison de données bidirectionnelle :

```razor
<h3>Exemple de liaison de données</h3>

<!-- Liaison unidirectionnelle (one-way) -->
<p>Nom saisi : @name</p>

<!-- Liaison bidirectionnelle (two-way) -->
<input @bind="name" @bind:event="oninput" />
<input @bind="birthDate" type="date" />

<!-- Événements -->
<button @onclick="IncrementCount">Compteur : @count</button>

@code {
    private string name = "";
    private DateTime birthDate = DateTime.Now;
    private int count = 0;

    private void IncrementCount()
    {
        count++;
    }
}
```

### Blazor Server

Blazor Server garde l'essentiel de l'application sur le serveur et communique avec le navigateur via SignalR (une bibliothèque de communication en temps réel).

#### Fonctionnement

1. Le navigateur charge une page légère contenant une connexion SignalR
2. Les interactions utilisateur sont envoyées au serveur
3. Le serveur exécute la logique C# et met à jour un modèle DOM
4. Seules les différences (patches) sont renvoyées au navigateur

#### Avantages de Blazor Server

- **Démarrage rapide** : Petit téléchargement initial
- **Compatibilité** : Fonctionne sur tous les navigateurs modernes sans WebAssembly
- **Performance serveur** : Exécution du code sur le serveur avec toutes les ressources disponibles
- **Accès direct aux ressources serveur** : Bases de données, fichiers, etc.
- **Sécurité** : Le code sensible reste sur le serveur

#### Inconvénients de Blazor Server

- **Dépendance à la connexion** : Nécessite une connexion constante au serveur
- **Latence** : Chaque interaction implique un aller-retour réseau
- **Mise à l'échelle** : Chaque utilisateur maintient une connexion permanente avec le serveur
- **Hors ligne** : Ne fonctionne pas sans connexion internet

#### Configuration de Blazor Server

```csharp
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services Blazor Server
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddSingleton<WeatherForecastService>();

var app = builder.Build();

// Configuration du pipeline
app.UseStaticFiles();
app.UseRouting();

app.MapBlazorHub(); // Point de terminaison SignalR
app.MapFallbackToPage("/_Host"); // Page hôte par défaut

app.Run();
```

La page hôte (`_Host.cshtml`) contient le script SignalR et le composant racine :

```cshtml
@page "/"
@namespace MyBlazorApp.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@{
    Layout = "_Layout";
}

<component type="typeof(App)" render-mode="ServerPrerendered" />

<script src="_framework/blazor.server.js"></script>
```

### Blazor WebAssembly

Blazor WebAssembly (WASM) compile l'application .NET en bytecode WebAssembly qui s'exécute directement dans le navigateur.

#### Fonctionnement

1. Le navigateur télécharge les fichiers .NET nécessaires et le runtime WebAssembly
2. L'application s'exécute entièrement dans le navigateur
3. Les appels API au serveur sont effectués via HTTP standard

#### Avantages de Blazor WebAssembly

- **Exécution côté client** : Aucune dépendance de connexion constante
- **Fonctionnement hors ligne** : Peut continuer à fonctionner sans connexion
- **Distribution de la charge** : Utilise les ressources du client au lieu du serveur
- **Déploiement simple** : Peut être hébergé sur un simple CDN comme des fichiers statiques

#### Inconvénients de Blazor WebAssembly

- **Téléchargement initial** : Plus volumineux (runtime .NET + application)
- **Performance** : Moins rapide que JavaScript natif (bien que cela s'améliore)
- **Compatibilité** : Nécessite un navigateur supportant WebAssembly
- **Sécurité** : Le code est téléchargé sur le client et peut être examiné

#### Configuration de Blazor WebAssembly

```csharp
// Dans Program.cs d'une application Blazor WebAssembly
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

// Configurer HttpClient pour les appels API
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });

// Ajouter d'autres services
builder.Services.AddScoped<IWeatherService, WeatherService>();

await builder.Build().RunAsync();
```

Le fichier HTML d'hébergement est généralement un simple fichier statique :

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    <title>Ma première application Blazor WebAssembly</title>
    <base href="/" />
    <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" />
</head>
<body>
    <div id="app">Chargement...</div>

    <script src="_framework/blazor.webassembly.js"></script>
</body>
</html>
```

### Blazor hybride : applications hébergées (Hosted)

Blazor permet également un modèle "hébergé" qui combine les avantages des deux approches :

1. Une application Blazor WebAssembly pour l'interface utilisateur
2. Une API ASP.NET Core sur le serveur pour la logique métier
3. Un projet partagé pour les modèles communs

Cette architecture est idéale pour les applications complètes nécessitant :
- Une UI réactive
- Une API sécurisée pour les données sensibles
- Un partage de code entre client et serveur

#### Structure d'un projet hébergé

```
BlazorApp/
  ├─ Client/           # Application Blazor WebAssembly
  │   ├─ Pages/
  │   ├─ Shared/
  │   └─ Program.cs
  │
  ├─ Server/           # API ASP.NET Core
  │   ├─ Controllers/
  │   └─ Program.cs
  │
  └─ Shared/           # Code partagé
      └─ Models/
```

### Fonctionnalités avancées de Blazor

#### Interopérabilité JavaScript

Blazor peut appeler du code JavaScript et vice-versa :

```csharp
// Appel de JavaScript depuis C#
@inject IJSRuntime JSRuntime

@code {
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await JSRuntime.InvokeVoidAsync("showAlert", "Hello from Blazor!");
        }
    }
}
```

JavaScript :
```javascript
window.showAlert = function(message) {
    alert(message);
    return "Alert shown!";
};
```

Appel de C# depuis JavaScript :
```javascript
// Appel d'une méthode .NET depuis JavaScript
function updateCounter() {
    DotNet.invokeMethodAsync('BlazorApp', 'IncrementCounter')
        .then(result => {
            console.log(`Counter is now: ${result}`);
        });
}
```

Avec le C# correspondant :
```csharp
[JSInvokable]
public static int IncrementCounter()
{
    currentCount++;
    return currentCount;
}
```

#### Formulaires et validation

Blazor intègre la validation de formulaires avec les annotations de données :

```razor
<EditForm Model="@customer" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <div class="form-group">
        <label for="name">Name:</label>
        <InputText id="name" @bind-Value="customer.Name" class="form-control" />
        <ValidationMessage For="@(() => customer.Name)" />
    </div>

    <div class="form-group">
        <label for="email">Email:</label>
        <InputText id="email" @bind-Value="customer.Email" class="form-control" />
        <ValidationMessage For="@(() => customer.Email)" />
    </div>

    <button type="submit" class="btn btn-primary">Submit</button>
</EditForm>

@code {
    private Customer customer = new();

    private void HandleValidSubmit()
    {
        // Traitement du formulaire valide
    }

    public class Customer
    {
        [Required]
        [StringLength(50, ErrorMessage = "Name is too long.")]
        public string Name { get; set; }

        [Required]
        [EmailAddress]
        public string Email { get; set; }
    }
}
```

#### Authentication et Authorization

Blazor prend en charge l'authentification et l'autorisation :

```csharp
// Configuration de l'authentification dans Program.cs
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    // Configuration du fournisseur d'identité
});

// Dans un composant Blazor
@page "/secured-page"
@attribute [Authorize]

<h3>Page sécurisée</h3>

@if (User.IsInRole("Admin"))
{
    <div class="admin-panel">
        <!-- Contenu réservé aux administrateurs -->
    </div>
}

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    protected override async Task OnInitializedAsync()
    {
        var authState = await authenticationStateTask;
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            var userName = user.Identity.Name;
            // ...
        }
    }
}
```

#### Gestion d'état

Blazor offre plusieurs façons de gérer l'état de l'application :

1. **État du composant** : Variables dans les composants
2. **Paramètres en cascade** : Pour partager des données dans l'arborescence de composants
3. **Services injectés** : Pour un état global via l'injection de dépendances
4. **LocalStorage/SessionStorage** : Pour un état persistant
5. **State containers** : Pattern pour centraliser la gestion d'état

```csharp
// Exemple de state container
public class CounterState
{
    private int _count = 0;

    public int Count => _count;

    // Événement pour notifier les changements
    public event Action OnChange;

    public void IncrementCount()
    {
        _count++;
        NotifyStateChanged();
    }

    private void NotifyStateChanged() => OnChange?.Invoke();
}

// Enregistrement dans les services
services.AddScoped<CounterState>();

// Utilisation dans un composant
@inject CounterState CounterState
@implements IDisposable

<p>Count: @CounterState.Count</p>
<button @onclick="IncrementCounter">Increment</button>

@code {
    protected override void OnInitialized()
    {
        // S'abonner aux changements d'état
        CounterState.OnChange += StateHasChanged;
    }

    private void IncrementCounter()
    {
        CounterState.IncrementCount();
    }

    public void Dispose()
    {
        // Se désabonner pour éviter les fuites de mémoire
        CounterState.OnChange -= StateHasChanged;
    }
}
```

### Choisir entre Blazor Server et WebAssembly

| Critère | Blazor Server | Blazor WebAssembly |
|---------|---------------|-------------------|
| **Téléchargement initial** | Petit (~1.5 MB) | Plus grand (~5-10 MB) |
| **Performance perçue** | Dépend de la latence réseau | Excellente après chargement |
| **Utilisation hors ligne** | Non | Possible |
| **Charge serveur** | Plus élevée | Plus légère |
| **Sécurité du code** | Reste sur le serveur | Téléchargé sur le client |
| **Accès direct aux ressources serveur** | Oui | Non (nécessite une API) |
| **Idéal pour** | - Applications intranet<br>- Audiences ciblées<br>- Logique complexe | - Applications publiques<br>- Applications PWA<br>- Contenu statique enrichi |

### Évolution et futur de Blazor

Blazor continue d'évoluer rapidement. Parmi les développements récents et futurs :

1. **Blazor Native** : Blazor pour applications desktop et mobiles
2. **AOT Compilation** : Amélioration des performances par la compilation anticipée
3. **CSS Isolation** : Styles CSS isolés par composant
4. **Hot Reload** : Mise à jour du code sans redémarrage
5. **Amélioration des performances WebAssembly** : Chargement plus rapide et exécution optimisée

## Conclusion

ASP.NET Core représente une évolution majeure par rapport à ASP.NET Framework traditionnel. Grâce à sa conception modulaire, multiplateforme et orientée performance, il est devenu un choix de premier plan pour développer des applications web modernes.

Les principaux avantages d'ASP.NET Core sont :

1. **Flexibilité** : S'adapte à différents types de projets, du petit site au système d'entreprise
2. **Performance** : Conçu pour être rapide et efficace
3. **Multiplateforme** : Fonctionne sur Windows, macOS et Linux
4. **Moderne** : Intègre les meilleures pratiques web actuelles
5. **Open Source** : Développement transparent et communautaire
6. **Évolutif** : De l'application simple au système distribué

Avec l'ajout de Blazor, Microsoft a également offert une solution innovante pour créer des interfaces utilisateur riches en utilisant C# au lieu de JavaScript, ouvrant de nouvelles possibilités pour les développeurs .NET.

Que vous choisissiez ASP.NET Core MVC, Razor Pages, ou Blazor, vous bénéficierez d'un écosystème mature, bien supporté et en constante évolution.

⏭️
