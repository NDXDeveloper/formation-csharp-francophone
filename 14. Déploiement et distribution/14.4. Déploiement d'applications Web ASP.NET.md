# 14.4. Déploiement d'applications Web ASP.NET

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le déploiement d'applications web ASP.NET est une étape cruciale pour rendre votre application accessible aux utilisateurs. Contrairement aux applications de bureau qui s'exécutent directement sur l'ordinateur de l'utilisateur, les applications web nécessitent un serveur pour fonctionner. Ce chapitre vous guidera à travers les différentes méthodes de déploiement d'applications ASP.NET Core, de la configuration des serveurs à la mise en place de stratégies de supervision.

## 14.4.1. IIS (Internet Information Services)

IIS est le serveur web de Microsoft et constitue le choix le plus courant pour héberger des applications ASP.NET, particulièrement dans les environnements Windows.

### Installation et configuration d'IIS

Avant de déployer votre application, vous devez installer et configurer IIS sur votre serveur Windows.

#### Installation d'IIS sur Windows Server

1. Ouvrez le **Gestionnaire de serveur**
2. Sélectionnez **Ajouter des rôles et fonctionnalités**
3. Sélectionnez **Installation basée sur un rôle ou une fonctionnalité**
4. Choisissez votre serveur
5. Cochez **Serveur Web (IIS)** dans la liste des rôles
6. Dans la section **Services de rôle**, assurez-vous de sélectionner :
   - **ASP.NET Core Module**
   - **Authentification Windows** (si nécessaire)
   - **Document statique**
   - **Journalisation HTTP**
7. Terminez l'assistant et attendez la fin de l'installation

#### Installation d'IIS sur Windows 10/11

1. Ouvrez **Paramètres** > **Applications** > **Applications et fonctionnalités**
2. Cliquez sur **Fonctionnalités facultatives** > **Ajouter une fonctionnalité**
3. Recherchez et sélectionnez **Internet Information Services**
4. Cliquez sur **Installer**

#### Installation du module ASP.NET Core sur IIS

Pour héberger des applications ASP.NET Core, vous devez installer le module ASP.NET Core :

1. Téléchargez le bundle d'hébergement .NET Core depuis le site Microsoft
   ```
   https://dotnet.microsoft.com/download/dotnet/
   ```
2. Exécutez le programme d'installation
3. Suivez les instructions à l'écran

### Déploiement d'une application ASP.NET sur IIS

#### 1. Préparer l'application pour le déploiement

Dans Visual Studio :

1. Cliquez avec le bouton droit sur votre projet > **Publier**
2. Sélectionnez **Dossier** comme cible
3. Choisissez un dossier de destination
4. Cliquez sur **Publier**

Ou avec la CLI .NET :

```bash
dotnet publish -c Release -o ./publish
```

#### 2. Créer un site dans IIS

1. Ouvrez le **Gestionnaire des services Internet (IIS)**
2. Développez le nœud du serveur
3. Cliquez avec le bouton droit sur **Sites** > **Ajouter un site Web**
4. Renseignez les informations suivantes :
   - **Nom du site** : NomDeVotreApplication
   - **Chemin d'accès physique** : Chemin vers le dossier publié
   - **Port** : 80 (par défaut) ou un autre port
   - **Nom d'hôte** : optionnel (ex : monapp.example.com)
5. Cliquez sur **OK**

#### 3. Configurer le pool d'applications

1. Cliquez sur **Pools d'applications**
2. Sélectionnez le pool créé pour votre application
3. Cliquez sur **Paramètres avancés**
4. Définissez les paramètres suivants :
   - **Version .NET CLR** : Sans code managé (pour ASP.NET Core)
   - **Mode pipeline managé** : Intégré
   - **Identité** : Choisissez un compte approprié (NetworkService est souvent utilisé)

#### 4. Configurer le fichier web.config

Le fichier `web.config` est généré automatiquement lors de la publication, mais vous pourriez avoir besoin de le personnaliser :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\MonApplication.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

Options importantes :
- **processPath** : Chemin vers l'exécutable (dotnet pour ASP.NET Core)
- **arguments** : Chemin vers le DLL de votre application
- **stdoutLogEnabled** : Active/désactive la journalisation de la sortie standard
- **hostingModel** : "inprocess" (meilleure performance) ou "outofprocess" (meilleure isolation)

### Déploiement avec Web Deploy

Web Deploy est un outil qui simplifie considérablement le déploiement d'applications web.

#### Installation de Web Deploy

1. Téléchargez Web Deploy depuis le site Microsoft
2. Exécutez l'installateur
3. Suivez les instructions à l'écran

#### Configuration de la publication Web Deploy dans Visual Studio

1. Cliquez avec le bouton droit sur votre projet > **Publier**
2. Sélectionnez **IIS, FTP, etc.**
3. Sélectionnez **Web Deploy** comme méthode de publication
4. Configurez les paramètres suivants :
   - **Serveur** : Nom ou IP du serveur IIS
   - **Nom du site** : Nom du site dans IIS
   - **Nom d'utilisateur / Mot de passe** : Identifiants pour la connexion
5. Cliquez sur **Valider la connexion** pour vérifier
6. Cliquez sur **Publier**

#### Automatisation du déploiement avec MSBuild

Vous pouvez automatiser le déploiement avec MSBuild :

```bash
msbuild MonApplication.csproj /p:DeployOnBuild=true /p:WebPublishMethod=MSDeploy /p:MSDeployServiceURL=monserveur /p:DeployIisAppPath="NomDuSite" /p:UserName=utilisateur /p:Password=motdepasse
```

## 14.4.2. Kestrel

Kestrel est un serveur web multiplateforme inclus avec ASP.NET Core. Il est léger, rapide et peut être utilisé seul ou conjointement avec un serveur web comme IIS ou Nginx.

### Fonctionnement de Kestrel

Dans une application ASP.NET Core, Kestrel est configuré automatiquement dans la méthode `CreateHostBuilder` du fichier `Program.cs` :

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            // Kestrel est configuré ici par défaut
        });
```

### Configuration avancée de Kestrel

Vous pouvez personnaliser Kestrel dans la méthode `ConfigureWebHostDefaults` :

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.ConfigureKestrel(serverOptions =>
            {
                // Configuration de base
                serverOptions.Listen(IPAddress.Any, 5000);

                // Configuration HTTPS
                serverOptions.Listen(IPAddress.Any, 5001, listenOptions =>
                {
                    listenOptions.UseHttps("certificate.pfx", "mot_de_passe");
                });

                // Limites de connexion
                serverOptions.Limits.MaxConcurrentConnections = 100;
                serverOptions.Limits.MaxRequestBodySize = 10 * 1024 * 1024; // 10 MB
                serverOptions.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
            });
        });
```

### Déploiement autonome avec Kestrel

Vous pouvez déployer votre application pour qu'elle s'exécute directement avec Kestrel :

1. Publiez votre application :
   ```bash
   dotnet publish -c Release -o ./publish
   ```

2. Exécutez l'application sur le serveur :
   ```bash
   cd publish
   dotnet MonApplication.dll
   ```

3. Pour exécuter l'application en tant que service :

   Sur Windows, utilisez le gestionnaire de services Windows ou NSSM (Non-Sucking Service Manager) :
   ```
   nssm install MonServiceWeb "C:\chemin\vers\dotnet.exe" "C:\chemin\vers\MonApplication.dll"
   ```

   Sur Linux, créez un service systemd :
   ```
   sudo nano /etc/systemd/system/monapp.service
   ```

   Contenu du fichier service :
   ```
   [Unit]
   Description=Mon Application ASP.NET Core

   [Service]
   WorkingDirectory=/var/www/monapp
   ExecStart=/usr/bin/dotnet /var/www/monapp/MonApplication.dll
   Restart=always
   RestartSec=10
   SyslogIdentifier=monapp
   User=www-data
   Environment=ASPNETCORE_ENVIRONMENT=Production

   [Install]
   WantedBy=multi-user.target
   ```

   Activation du service :
   ```bash
   sudo systemctl enable monapp.service
   sudo systemctl start monapp.service
   ```

### Utilisation de Kestrel avec un serveur proxy inverse

En production, il est recommandé d'utiliser Kestrel derrière un serveur proxy inverse comme IIS, Nginx ou Apache.

#### Configuration avec Nginx

1. Installez Nginx :
   ```bash
   sudo apt-get install nginx
   ```

2. Configurez un site dans Nginx :
   ```
   sudo nano /etc/nginx/sites-available/monapp
   ```

   Contenu du fichier :
   ```
   server {
       listen 80;
       server_name monapp.example.com;

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

3. Activez le site :
   ```bash
   sudo ln -s /etc/nginx/sites-available/monapp /etc/nginx/sites-enabled/monapp
   sudo nginx -t
   sudo systemctl restart nginx
   ```

## 14.4.3. Docker et conteneurs

Docker permet de packager votre application avec toutes ses dépendances dans un conteneur, garantissant qu'elle fonctionnera de manière identique dans n'importe quel environnement.

### Avantages des conteneurs pour le déploiement ASP.NET

- **Cohérence** : L'application fonctionne de la même façon sur tous les environnements
- **Isolation** : Les dépendances ne créent pas de conflits avec d'autres applications
- **Portabilité** : Déployez sur n'importe quelle plateforme supportant Docker
- **Scalabilité** : Facilité pour créer plusieurs instances de l'application
- **CI/CD** : Intégration simplifiée avec les pipelines d'intégration continue

### Création d'un Dockerfile pour ASP.NET Core

Créez un fichier nommé `Dockerfile` à la racine de votre projet :

```dockerfile
# Image de base pour la compilation
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copie des fichiers projet et restauration des dépendances
COPY ["MonApplication.csproj", "./"]
RUN dotnet restore "MonApplication.csproj"

# Copie du code source et compilation
COPY . .
RUN dotnet build "MonApplication.csproj" -c Release -o /app/build

# Publication de l'application
FROM build AS publish
RUN dotnet publish "MonApplication.csproj" -c Release -o /app/publish

# Image finale
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .
EXPOSE 80
EXPOSE 443
ENTRYPOINT ["dotnet", "MonApplication.dll"]
```

### Construction et exécution de l'image Docker

```bash
# Construction de l'image
docker build -t mon-application:latest .

# Exécution du conteneur
docker run -d -p 8080:80 --name mon-app mon-application:latest
```

### Docker Compose pour les applications multi-conteneurs

Si votre application utilise plusieurs services (par exemple, une base de données), utilisez Docker Compose.

Créez un fichier `docker-compose.yml` :

```yaml
version: '3.8'

services:
  webapp:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - db
    environment:
      - ConnectionStrings__DefaultConnection=Server=db;Database=monapp;User=sa;Password=VotreMotDePasseComplexe!;

  db:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=VotreMotDePasseComplexe!
    ports:
      - "1433:1433"
    volumes:
      - db-data:/var/opt/mssql

volumes:
  db-data:
```

Lancez les services :

```bash
docker-compose up -d
```

### Déploiement sur un registre de conteneurs

Pour déployer votre application sur un serveur distant, vous devez d'abord pousser votre image vers un registre :

```bash
# Connexion à Docker Hub
docker login

# Tag de l'image
docker tag mon-application:latest votre-utilisateur/mon-application:latest

# Push de l'image
docker push votre-utilisateur/mon-application:latest
```

Vous pouvez également utiliser des registres privés comme Azure Container Registry ou AWS ECR.

### Orchestration avec Kubernetes

Pour les applications plus complexes nécessitant une haute disponibilité, utilisez Kubernetes pour orchestrer vos conteneurs.

Exemple de fichier `deployment.yaml` :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-application
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mon-application
  template:
    metadata:
      labels:
        app: mon-application
    spec:
      containers:
      - name: mon-application
        image: votre-utilisateur/mon-application:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
---
apiVersion: v1
kind: Service
metadata:
  name: mon-application
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: mon-application
```

Déploiement sur Kubernetes :

```bash
kubectl apply -f deployment.yaml
```

## 14.4.4. Azure App Service

Azure App Service est une plateforme de services gérés pour l'hébergement d'applications web, y compris ASP.NET Core.

### Avantages d'Azure App Service

- **Service géré** : Pas besoin de gérer l'infrastructure sous-jacente
- **Mise à l'échelle automatique** : Adaptation automatique aux variations de charge
- **CI/CD intégré** : Intégration facile avec GitHub, Azure DevOps, etc.
- **Diagnostics intégrés** : Outils de surveillance et dépannage
- **Slots de déploiement** : Environnements de préproduction avec échange sans temps d'arrêt
- **Sécurité avancée** : HTTPS, authentification, autorisations, etc.

### Déploiement sur Azure App Service depuis Visual Studio

1. Cliquez avec le bouton droit sur votre projet > **Publier**
2. Sélectionnez **Azure** > **Azure App Service**
3. Connectez-vous à votre compte Azure
4. Sélectionnez un App Service existant ou créez-en un nouveau
5. Cliquez sur **Publier**

### Déploiement avec Azure CLI

1. Installez Azure CLI
2. Connectez-vous à votre compte Azure :
   ```bash
   az login
   ```

3. Créez un groupe de ressources (si nécessaire) :
   ```bash
   az group create --name MonGroupe --location "France Central"
   ```

4. Créez un plan App Service :
   ```bash
   az appservice plan create --name MonPlan --resource-group MonGroupe --sku B1
   ```

5. Créez une application web :
   ```bash
   az webapp create --name MonApplication --resource-group MonGroupe --plan MonPlan
   ```

6. Déployez votre application :
   ```bash
   az webapp deployment source config-zip --resource-group MonGroupe --name MonApplication --src app.zip
   ```

### Configuration d'Azure App Service

#### Variables d'environnement

Définissez des variables d'environnement dans le portail Azure :
1. Ouvrez votre App Service
2. Accédez à **Configuration** > **Paramètres de l'application**
3. Ajoutez des paires clé-valeur pour vos variables

Ces variables seront accessibles via `Environment.GetEnvironmentVariable()` ou la configuration ASP.NET Core.

#### Connexions à la base de données

1. Créez une base de données Azure SQL
2. Accédez à **Chaînes de connexion** dans la configuration de l'App Service
3. Ajoutez votre chaîne de connexion

#### Configuration du domaine personnalisé

1. Accédez à **Domaines personnalisés** dans votre App Service
2. Cliquez sur **Ajouter un domaine personnalisé**
3. Suivez les instructions pour configurer les enregistrements DNS
4. Ajoutez un certificat SSL pour sécuriser votre domaine

### Slots de déploiement

Les slots de déploiement vous permettent de tester votre application avant de la publier en production.

1. Accédez à **Slots de déploiement** dans votre App Service
2. Cliquez sur **Ajouter un slot**
3. Donnez un nom au slot (par exemple, "staging")
4. Déployez votre application sur ce slot
5. Testez votre application sur le slot
6. Lorsque vous êtes prêt, cliquez sur **Swap** pour échanger le slot avec la production

### Auto-scaling

Configurez l'auto-scaling pour gérer automatiquement la charge :

1. Accédez à **Scale out (App Service plan)** dans votre App Service
2. Choisissez **Règles de mise à l'échelle automatique**
3. Définissez des règles basées sur des métriques (CPU, mémoire, etc.)

## 14.4.5. Configurations pour différents environnements

ASP.NET Core offre un système robuste pour gérer différentes configurations selon l'environnement (développement, test, production, etc.).

### Variables d'environnement ASPNETCORE_ENVIRONMENT

La variable d'environnement `ASPNETCORE_ENVIRONMENT` détermine l'environnement actuel :

- **Development** : Pour le développement local
- **Staging** : Pour les tests avant production
- **Production** : Pour l'environnement de production

Définition de la variable :

- Windows : `set ASPNETCORE_ENVIRONMENT=Development`
- Linux/macOS : `export ASPNETCORE_ENVIRONMENT=Development`
- IIS : Dans les paramètres avancés du pool d'applications
- Azure : Dans les paramètres de l'application

### Fichiers de configuration par environnement

ASP.NET Core charge automatiquement les fichiers de configuration spécifiques à l'environnement :

- `appsettings.json` : Configuration de base
- `appsettings.Development.json` : Surcharges pour le développement
- `appsettings.Production.json` : Surcharges pour la production

Exemple de `appsettings.json` :
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MaBase;User Id=utilisateur;Password=motdepasse;"
  }
}
```

Exemple de `appsettings.Production.json` :
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=MaBase;User Id=utilisateur;Password=motdepasse;"
  }
}
```

### Configuration programmatique par environnement

Dans la classe `Startup.cs`, vous pouvez appliquer une configuration spécifique à chaque environnement :

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseSwagger();
        app.UseSwaggerUI();
    }
    else if (env.IsStaging())
    {
        app.UseExceptionHandler("/Error");
        // Configurations spécifiques au staging
    }
    else // Production
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
        // Configurations spécifiques à la production
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

### Gestion des secrets

Ne stockez jamais de secrets (mots de passe, clés API, etc.) dans le code source ou les fichiers de configuration versionnés.

#### Secret Manager pour le développement

Pour le développement local, utilisez Secret Manager :

1. Initialisez Secret Manager pour votre projet :
   ```bash
   dotnet user-secrets init
   ```

2. Ajoutez des secrets :
   ```bash
   dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=dev-server;Database=MaBase;User Id=utilisateur;Password=motdepasse;"
   ```

3. Accédez aux secrets dans votre code :
   ```csharp
   var connectionString = Configuration.GetConnectionString("DefaultConnection");
   ```

#### Azure Key Vault pour la production

Pour la production, utilisez Azure Key Vault :

1. Créez un Key Vault dans Azure
2. Ajoutez vos secrets dans le Key Vault
3. Configurez votre application pour accéder au Key Vault

Dans `Program.cs` :
```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            if (context.HostingEnvironment.IsProduction())
            {
                var builtConfig = config.Build();
                var keyVaultEndpoint = builtConfig["AzureKeyVault:Endpoint"];

                if (!string.IsNullOrEmpty(keyVaultEndpoint))
                {
                    config.AddAzureKeyVault(
                        keyVaultEndpoint,
                        new DefaultAzureCredential());
                }
            }
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

### Transformations Web.config

Si vous déployez sur IIS, vous pouvez utiliser des transformations de fichier web.config :

1. Créez des fichiers spécifiques à l'environnement :
   - `web.Development.config`
   - `web.Production.config`

2. Définissez les transformations à appliquer :

```xml
<!-- web.Production.config -->
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.webServer>
    <aspNetCore>
      <environmentVariables xdt:Transform="Replace">
        <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />
      </environmentVariables>
    </aspNetCore>
  </system.webServer>
</configuration>
```

## 14.4.6. Supervision et diagnostics

Une bonne supervision est essentielle pour assurer la disponibilité et les performances de votre application en production.

### Journalisation (Logging)

ASP.NET Core intègre un système de journalisation extensible qui prend en charge plusieurs fournisseurs de logs.

#### Configuration de la journalisation

Dans `Program.cs` :

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureLogging((context, logging) =>
        {
            logging.ClearProviders();
            logging.AddConsole();
            logging.AddDebug();
            logging.AddEventSourceLogger();

            // Ajoutez Serilog, NLog ou d'autres fournisseurs
            if (context.HostingEnvironment.IsProduction())
            {
                logging.AddEventLog();
            }
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

#### Utilisation de Serilog

Serilog est une bibliothèque de journalisation populaire pour .NET :

1. Installez les packages NuGet :
   ```
   Serilog.AspNetCore
   Serilog.Sinks.File
   Serilog.Sinks.Console
   ```

2. Configurez Serilog dans `Program.cs` :
   ```csharp
   public static void Main(string[] args)
   {
       Log.Logger = new LoggerConfiguration()
           .MinimumLevel.Information()
           .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
           .Enrich.FromLogContext()
           .WriteTo.Console()
           .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
           .CreateLogger();

       try
       {
           Log.Information("Démarrage de l'application");
           CreateHostBuilder(args).Build().Run();
       }
       catch (Exception ex)
       {
           Log.Fatal(ex, "L'application s'est arrêtée de façon inattendue");
       }
       finally
       {
           Log.CloseAndFlush();
       }
   }

   public static IHostBuilder CreateHostBuilder(string[] args) =>
       Host.CreateDefaultBuilder(args)
           .UseSerilog()
           .ConfigureWebHostDefaults(webBuilder =>
           {
               webBuilder.UseStartup<Startup>();
           });
   ```

3. Utilisez le logger dans vos contrôleurs ou services :
   ```csharp
   public class MonController : ControllerBase
   {
       private readonly ILogger<MonController> _logger;

       public MonController(ILogger<MonController> logger)
       {
           _logger = logger;
       }

       [HttpGet]
       public IActionResult Get()
       {
           _logger.LogInformation("Action Get appelée à {Time}", DateTime.UtcNow);

           try
           {
               // Code métier
               return Ok(result);
           }
           catch (Exception ex)
           {
               _logger.LogError(ex, "Erreur lors de l'exécution de Get");
               return StatusCode(500);
           }
       }
   }
   ```

### Application Insights

Azure Application Insights est un service de surveillance des performances des applications (APM) qui permet de détecter, trier et diagnostiquer les problèmes.

#### Configuration d'Application Insights

1. Installez le package NuGet :
   ```
   Microsoft.ApplicationInsights.AspNetCore
   ```

2. Ajoutez Application Insights dans `Program.cs` :
   ```csharp
   public static IHostBuilder CreateHostBuilder(string[] args) =>
       Host.CreateDefaultBuilder(args)
           .ConfigureWebHostDefaults(webBuilder =>
           {
               webBuilder.UseStartup<Startup>();
               webBuilder.UseApplicationInsights();
           });
   ```

3. Configurez la clé d'instrumentation dans `appsettings.json` :
   ```json
   {
     "ApplicationInsights": {
       "InstrumentationKey": "votre-clé-d-instrumentation"
     }
   }
   ```

4. Utilisez le TelemetryClient dans vos contrôleurs :
   ```csharp
   public class MonController : ControllerBase
   {
       private readonly TelemetryClient _telemetryClient;

       public MonController(TelemetryClient telemetryClient)
       {
           _telemetryClient = telemetryClient;
       }

       [HttpGet]
       public IActionResult Get()
       {
           // Trace une dépendance
           using (_telemetryClient.StartOperation<DependencyTelemetry>("OperationName"))
           {
               // Code métier

               // Trace un événement personnalisé
               _telemetryClient.TrackEvent("GetActionCalled", new Dictionary<string, string>
               {
                   ["User"] = User.Identity.Name
               });

               return Ok(result);
           }
       }
   }
   ```

### Health Checks

Les health checks permettent de vérifier l'état de santé de votre application et de ses dépendances.

#### Configuration des health checks

1. Installez le package NuGet :
   ```
   Microsoft.AspNetCore.Diagnostics.HealthChecks
   AspNetCore.HealthChecks.SqlServer  # Pour vérifier SQL Server
   ```

2. Configurez les health checks dans `Startup.cs` :
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // Autres services...

       services.AddHealthChecks()
           // Vérification de base (ping)
           .AddCheck("self", () => HealthCheckResult.Healthy())
           // Vérification de la base de données
           .AddSqlServer(
               Configuration.GetConnectionString("DefaultConnection"),
               name: "database",
               tags: new[] { "db", "sql", "sqlserver" })
           // Vérification d'un service externe
           .AddUrlGroup(
               new Uri("https://api-externe.example.com/health"),
               name: "external-api",
               tags: new[] { "api" })
           // Vérification personnalisée
           .AddCheck<MonHealthCheck>("custom-check");
   }

   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       // Autres middlewares...

       app.UseEndpoints(endpoints =>
       {
           // Endpoint de base pour les health checks
           endpoints.MapHealthChecks("/health", new HealthCheckOptions
           {
               ResponseWriter = WriteHealthCheckResponse
           });

           // Endpoint pour les vérifications détaillées (accès restreint)
           endpoints.MapHealthChecks("/health/detailed", new HealthCheckOptions
           {
               Predicate = _ => true,
               ResponseWriter = WriteHealthCheckResponse,
               ResultStatusCodes =
               {
                   [HealthStatus.Healthy] = StatusCodes.Status200OK,
                   [HealthStatus.Degraded] = StatusCodes.Status200OK,
                   [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
               }
           }).RequireAuthorization("HealthCheckPolicy");

           // Autres endpoints...
       });
   }

   private static Task WriteHealthCheckResponse(HttpContext context, HealthReport report)
   {
       context.Response.ContentType = "application/json";

       var result = JsonSerializer.Serialize(new
       {
           status = report.Status.ToString(),
           duration = report.TotalDuration,
           checks = report.Entries.Select(e => new
           {
               name = e.Key,
               status = e.Value.Status.ToString(),
               description = e.Value.Description,
               duration = e.Value.Duration,
               exception = e.Value.Exception?.Message,
               data = e.Value.Data
           })
       });

       return context.Response.WriteAsync(result);
   }
   ```

3. Créez une vérification personnalisée :
   ```csharp
   public class MonHealthCheck : IHealthCheck
   {
       public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
       {
           try
           {
               // Vérifiez une condition métier spécifique
               bool conditionSatisfaite = VerifierCondition();

               if (conditionSatisfaite)
               {
                   return Task.FromResult(
                       HealthCheckResult.Healthy("Le service fonctionne normalement"));
               }

               return Task.FromResult(
                   HealthCheckResult.Degraded("Le service fonctionne mais avec des performances réduites"));
           }
           catch (Exception ex)
           {
               return Task.FromResult(
                   HealthCheckResult.Unhealthy("Le service ne fonctionne pas correctement", ex));
           }
       }

       private bool VerifierCondition()
       {
           // Implémentez votre logique de vérification
           return true;
       }
   }
   ```

### Métriques et surveillance

En plus des logs et health checks, il est important de collecter des métriques sur les performances de votre application.

#### Prometheus et Grafana

Prometheus est un système de surveillance open-source qui collecte des métriques, tandis que Grafana permet de les visualiser.

1. Installez le package NuGet :
   ```
   prometheus-net.AspNetCore
   ```

2. Configurez Prometheus dans `Startup.cs` :
   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       // Autres middlewares...

       // Exposition des métriques Prometheus
       app.UseMetricServer();
       app.UseHttpMetrics();

       // Autres middlewares...
   }
   ```

3. Ajoutez des métriques personnalisées :
   ```csharp
   public class MonService
   {
       private readonly Counter _operationsTotal;
       private readonly Histogram _operationDuration;

       public MonService()
       {
           _operationsTotal = Metrics.CreateCounter(
               "mon_service_operations_total",
               "Nombre total d'opérations effectuées",
               new CounterConfiguration
               {
                   LabelNames = new[] { "operation", "status" }
               });

           _operationDuration = Metrics.CreateHistogram(
               "mon_service_operation_duration_seconds",
               "Durée des opérations en secondes",
               new HistogramConfiguration
               {
                   LabelNames = new[] { "operation" },
                   Buckets = Histogram.ExponentialBuckets(0.01, 2, 10)
               });
       }

       public async Task<ResultatOperation> EffectuerOperation(string type)
       {
           using (var timer = _operationDuration.WithLabels(type).NewTimer())
           {
               try
               {
                   // Logique d'opération
                   var resultat = await CalculerResultat();

                   // Incrémente le compteur avec succès
                   _operationsTotal.WithLabels(type, "success").Inc();

                   return resultat;
               }
               catch (Exception)
               {
                   // Incrémente le compteur avec échec
                   _operationsTotal.WithLabels(type, "failure").Inc();
                   throw;
               }
           }
       }
   }
   ```

4. Configurez Grafana pour visualiser les métriques
   - Installez Grafana et Prometheus (via Docker ou directement)
   - Configurez Prometheus pour scraper votre endpoint (`/metrics`)
   - Configurez Grafana pour utiliser Prometheus comme source de données
   - Créez des tableaux de bord pour visualiser vos métriques

### Tracing distribué

Le tracing distribué permet de suivre le parcours d'une requête à travers plusieurs services.

#### OpenTelemetry

OpenTelemetry est un framework standard pour la collecte et l'envoi de télémétrie.

1. Installez les packages NuGet :
   ```
   OpenTelemetry.Extensions.Hosting
   OpenTelemetry.Instrumentation.AspNetCore
   OpenTelemetry.Exporter.Console  # Pour le développement
   OpenTelemetry.Exporter.Zipkin   # Pour la production
   ```

2. Configurez OpenTelemetry dans `Startup.cs` :
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // Autres services...

       services.AddOpenTelemetryTracing(builder =>
       {
           builder
               .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService("MonService"))
               .AddAspNetCoreInstrumentation()
               .AddHttpClientInstrumentation()
               .AddEntityFrameworkCoreInstrumentation()
               .AddSource("MonApplication")
               .AddConsoleExporter()
               .AddZipkinExporter(options =>
               {
                   options.Endpoint = new Uri("http://zipkin:9411/api/v2/spans");
               });
       });
   }
   ```

3. Utilisez le tracing dans votre code :
   ```csharp
   public class MonService
   {
       private static readonly ActivitySource _activitySource = new ActivitySource("MonApplication");

       public async Task<ResultatOperation> EffectuerOperation()
       {
           using (var activity = _activitySource.StartActivity("EffectuerOperation"))
           {
               activity?.SetTag("operation.type", "calcul");

               try
               {
                   // Logique d'opération
                   var resultat = await CalculerResultat();

                   activity?.SetTag("operation.success", true);
                   activity?.SetTag("operation.result", resultat.ToString());

                   return resultat;
               }
               catch (Exception ex)
               {
                   activity?.SetTag("operation.success", false);
                   activity?.SetTag("error", true);
                   activity?.SetTag("error.message", ex.Message);
                   throw;
               }
           }
       }
   }
   ```

### Diagnostics sur serveur en production

#### Collecte de dumps mémoire

Pour diagnostiquer des problèmes complexes, vous pourriez avoir besoin de collecter des dumps mémoire :

1. Installez PROCDUMP sur le serveur :
   ```
   https://docs.microsoft.com/en-us/sysinternals/downloads/procdump
   ```

2. Créez un dump lorsqu'un seuil de CPU est dépassé :
   ```
   procdump -ma -c 80 -s 5 -n 3 [PID] C:\dumps\app_dump
   ```

3. Créez un dump lors d'une exception spécifique :
   ```
   procdump -ma -e -f "System.OutOfMemoryException" [PID] C:\dumps\app_dump
   ```

4. Analysez le dump avec Visual Studio ou WinDbg

#### Mise en place d'un serveur de journalisation centralisé

Pour les applications distribuées, utilisez un système de journalisation centralisé :

1. Mettez en place Elasticsearch, Logstash et Kibana (ELK Stack) :
   - Elasticsearch stocke et indexe les logs
   - Logstash collecte et traite les logs
   - Kibana visualise les logs

2. Configurez Serilog pour envoyer les logs à Elasticsearch :
   ```csharp
   Log.Logger = new LoggerConfiguration()
       .MinimumLevel.Information()
       .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
       .Enrich.FromLogContext()
       .Enrich.WithMachineName()
       .Enrich.WithEnvironmentName()
       .WriteTo.Console()
       .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://elasticsearch:9200"))
       {
           AutoRegisterTemplate = true,
           IndexFormat = "monapp-logs-{0:yyyy.MM.dd}",
           ModifyConnectionSettings = x => x.BasicAuthentication("user", "password")
       })
       .CreateLogger();
   ```

### Alertes et notifications

Mettez en place des alertes pour être informé des problèmes avant vos utilisateurs.

#### Configuration d'alertes dans Application Insights

1. Accédez à votre ressource Application Insights dans le portail Azure
2. Allez dans "Alertes" > "Créer une règle d'alerte"
3. Définissez une condition (par exemple, taux d'échec > 5%)
4. Configurez un groupe d'actions (email, SMS, webhook, etc.)
5. Donnez un nom à votre alerte et activez-la

#### Création d'une API de vérification

Créez un endpoint qui vérifie et agrège l'état de votre application pour les systèmes de surveillance externes :

```csharp
[ApiController]
[Route("api/[controller]")]
public class StatusController : ControllerBase
{
    private readonly IHealthCheckService _healthCheckService;

    public StatusController(IHealthCheckService healthCheckService)
    {
        _healthCheckService = healthCheckService;
    }

    [HttpGet]
    [ProducesResponseType(typeof(StatusResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status503ServiceUnavailable)]
    public async Task<IActionResult> Get()
    {
        var report = await _healthCheckService.CheckHealthAsync();

        var result = new StatusResult
        {
            Status = report.Status.ToString(),
            Version = GetApplicationVersion(),
            UpTime = GetApplicationUpTime(),
            Checks = report.Entries.Select(e => new CheckResult
            {
                Name = e.Key,
                Status = e.Value.Status.ToString(),
                Description = e.Value.Description
            }).ToList()
        };

        if (report.Status == HealthStatus.Healthy)
        {
            return Ok(result);
        }

        return StatusCode(StatusCodes.Status503ServiceUnavailable, result);
    }

    private string GetApplicationVersion()
    {
        return Assembly.GetExecutingAssembly().GetName().Version.ToString();
    }

    private TimeSpan GetApplicationUpTime()
    {
        return DateTime.UtcNow - Process.GetCurrentProcess().StartTime.ToUniversalTime();
    }
}

public class StatusResult
{
    public string Status { get; set; }
    public string Version { get; set; }
    public TimeSpan UpTime { get; set; }
    public List<CheckResult> Checks { get; set; }
}

public class CheckResult
{
    public string Name { get; set; }
    public string Status { get; set; }
    public string Description { get; set; }
}
```

### Dashboard de monitoring

Créez un dashboard personnalisé pour visualiser l'état de votre application.

#### Dashboard avec ASP.NET Core et SignalR

1. Installez les packages NuGet :
   ```
   Microsoft.AspNetCore.SignalR
   ```

2. Créez un hub SignalR :
   ```csharp
   public class MonitoringHub : Hub
   {
       public async Task SendMetrics(MetricsData data)
       {
           await Clients.All.SendAsync("ReceiveMetrics", data);
       }
   }
   ```

3. Configurez SignalR dans `Startup.cs` :
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // Autres services...
       services.AddSignalR();
   }

   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       // Autres middlewares...

       app.UseEndpoints(endpoints =>
       {
           endpoints.MapHub<MonitoringHub>("/hubs/monitoring");
           // Autres endpoints...
       });
   }
   ```

4. Créez un service qui collecte périodiquement des métriques :
   ```csharp
   public class MetricsCollectorService : BackgroundService
   {
       private readonly IHubContext<MonitoringHub> _hubContext;
       private readonly ILogger<MetricsCollectorService> _logger;

       public MetricsCollectorService(
           IHubContext<MonitoringHub> hubContext,
           ILogger<MetricsCollectorService> logger)
       {
           _hubContext = hubContext;
           _logger = logger;
       }

       protected override async Task ExecuteAsync(CancellationToken stoppingToken)
       {
           while (!stoppingToken.IsCancellationRequested)
           {
               try
               {
                   var metrics = CollectMetrics();
                   await _hubContext.Clients.All.SendAsync("ReceiveMetrics", metrics, stoppingToken);
               }
               catch (Exception ex)
               {
                   _logger.LogError(ex, "Erreur lors de la collecte des métriques");
               }

               await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
           }
       }

       private MetricsData CollectMetrics()
       {
           var process = Process.GetCurrentProcess();

           return new MetricsData
           {
               Timestamp = DateTime.UtcNow,
               CpuUsage = GetCpuUsage(),
               MemoryUsageMb = process.WorkingSet64 / (1024 * 1024),
               ThreadCount = process.Threads.Count,
               HandleCount = process.HandleCount,
               RequestsPerSecond = GetRequestRate()
           };
       }

       // Implémentez les méthodes pour collecter les métriques
   }

   public class MetricsData
   {
       public DateTime Timestamp { get; set; }
       public double CpuUsage { get; set; }
       public long MemoryUsageMb { get; set; }
       public int ThreadCount { get; set; }
       public int HandleCount { get; set; }
       public double RequestsPerSecond { get; set; }
   }
   ```

5. Créez une interface utilisateur pour le dashboard (avec HTML, CSS et JavaScript)

## Résumé

Le déploiement d'applications web ASP.NET Core offre de nombreuses options et considérations. Retenez ces points clés :

1. **Options de déploiement** :
   - IIS : Plateforme traditionnelle pour Windows
   - Kestrel : Serveur léger, idéal derrière un proxy
   - Docker : Conteneurisation pour la portabilité
   - Azure App Service : Plateforme gérée pour un déploiement simplifié

2. **Configuration pour différents environnements** :
   - Utilisez des fichiers de configuration spécifiques à l'environnement
   - Stockez les secrets en dehors du code source
   - Adaptez la configuration programmatiquement selon l'environnement

3. **Supervision et diagnostics** :
   - Implémentez une journalisation appropriée
   - Utilisez des health checks pour vérifier l'état de l'application
   - Configurez la collecte de métriques et le tracing
   - Mettez en place des alertes proactives

En suivant ces bonnes pratiques, vous pourrez déployer et maintenir des applications ASP.NET Core robustes, performantes et faciles à diagnostiquer en production.

## Exercices pratiques

1. Déployez une application ASP.NET Core sur IIS et configurez-la pour utiliser HTTPS.

2. Créez un pipeline CI/CD qui déploie automatiquement votre application sur Azure App Service lorsque vous poussez du code sur la branche principale.

3. Conteneurisez une application ASP.NET Core et déployez-la sur Docker Desktop, puis sur une instance Docker distante.

4. Configurez une application ASP.NET Core pour fonctionner dans différents environnements (développement, test, production) avec des paramètres spécifiques à chaque environnement.

5. Implémentez un système complet de supervision pour une application ASP.NET Core, comprenant des logs, des métriques, des health checks et des alertes.

⏭️
