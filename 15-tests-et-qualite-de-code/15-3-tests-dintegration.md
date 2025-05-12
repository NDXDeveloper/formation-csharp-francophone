# 15.3. Tests d'intégration en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction aux tests d'intégration

Contrairement aux tests unitaires qui isolent et testent des portions individuelles de code, les tests d'intégration vérifient comment différents composants de votre application fonctionnent ensemble. Ces tests sont essentiels pour s'assurer que les différentes parties de votre système communiquent correctement et que l'application fonctionne comme prévu dans un environnement plus réaliste.

### Différences entre tests unitaires et tests d'intégration

| Tests unitaires | Tests d'intégration |
|-----------------|---------------------|
| Testent une seule unité de code isolée | Testent plusieurs composants ensemble |
| Rapides à exécuter | Généralement plus lents à exécuter |
| Utilisent des mocks pour isoler les dépendances | Utilisent des composants réels ou des environnements de test |
| Faciles à maintenir | Peuvent être plus complexes à maintenir |
| Identifient des bugs dans la logique métier | Identifient des problèmes d'intégration entre composants |

### Quand utiliser des tests d'intégration ?

Les tests d'intégration sont particulièrement utiles pour vérifier :
- Les interactions avec les bases de données
- Les appels d'API (internes ou externes)
- Le fonctionnement des contrôleurs dans une application web
- Les communications entre microservices
- Les opérations d'entrée/sortie (fichiers, réseau)
- Les workflows complets de bout en bout

## 15.3.1. Configuration

Pour mettre en place des tests d'intégration efficaces, une bonne configuration est essentielle. Voici comment procéder.

### Structure du projet

Il est recommandé de créer un projet de test d'intégration séparé de vos tests unitaires :

```
MaSolution/
  ├── src/
  │   ├── MonApplication/
  │   │   └── MonApplication.csproj
  │   └── MonApplication.Core/
  │       └── MonApplication.Core.csproj
  └── tests/
      ├── MonApplication.UnitTests/
      │   └── MonApplication.UnitTests.csproj
      └── MonApplication.IntegrationTests/
          └── MonApplication.IntegrationTests.csproj
```

### Outils et packages nécessaires

Voici les packages couramment utilisés pour les tests d'intégration en .NET :

```bash
# Framework de test (xUnit est souvent utilisé pour les tests d'intégration)
dotnet add package xunit
dotnet add package xunit.runner.visualstudio

# Pour les tests d'API web
dotnet add package Microsoft.AspNetCore.Mvc.Testing

# Pour les tests de base de données
dotnet add package Microsoft.EntityFrameworkCore.InMemory
# Ou pour une BD SQL réelle dans les tests
dotnet add package Microsoft.EntityFrameworkCore.Sqlite

# Pour les assertions plus riches
dotnet add package FluentAssertions

# Pour les mocks (si nécessaire)
dotnet add package Moq
```

### Configuration de base pour ASP.NET Core

Pour les applications ASP.NET Core, Microsoft fournit un package `Microsoft.AspNetCore.Mvc.Testing` qui simplifie les tests d'intégration. Voici comment le configurer :

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using System.Net.Http;
using Xunit;

namespace MonApplication.IntegrationTests
{
    public class ApiIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
    {
        private readonly HttpClient _client;

        public ApiIntegrationTests(WebApplicationFactory<Program> factory)
        {
            // Créer un client HTTP qui utilise l'application web de test
            _client = factory.CreateClient();
        }

        [Fact]
        public async Task GetClients_ReturnsSuccessStatusCode()
        {
            // Arrange & Act
            var response = await _client.GetAsync("/api/clients");

            // Assert
            response.EnsureSuccessStatusCode(); // Vérifie que le statut est 2xx
        }
    }
}
```

### Environnement de test personnalisé

Pour des configurations plus avancées, vous pouvez hériter de `WebApplicationFactory` :

```csharp
public class CustomWebApplicationFactory<TStartup> : WebApplicationFactory<TStartup> where TStartup : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remplacer les services enregistrés par des services de test
            // Par exemple, remplacer la base de données réelle par une base de données en mémoire

            var descriptor = services.SingleOrDefault(d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
            if (descriptor != null)
            {
                services.Remove(descriptor);
            }

            services.AddDbContext<ApplicationDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDatabase");
            });

            // Exécuter les migrations ou initialiser la base de données de test
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            db.Database.EnsureCreated();

            // Ajouter des données de test
            db.Clients.Add(new Client { Id = 1, Name = "Test Client" });
            db.SaveChanges();
        });
    }
}
```

### Configuration de test.runsettings

Pour une configuration plus avancée, vous pouvez utiliser un fichier `test.runsettings` :

```xml
<?xml version="1.0" encoding="utf-8"?>
<RunSettings>
  <RunConfiguration>
    <MaxCpuCount>1</MaxCpuCount>
    <ResultsDirectory>.\TestResults</ResultsDirectory>
    <TargetPlatform>x64</TargetPlatform>
    <TargetFrameworkVersion>net8.0</TargetFrameworkVersion>
  </RunConfiguration>
  <TestRunParameters>
    <Parameter name="ConnectionString" value="Data Source=:memory:;Version=3;New=True;" />
    <Parameter name="BaseUrl" value="https://localhost:5001" />
  </TestRunParameters>
</RunSettings>
```

Vous pouvez ensuite récupérer ces paramètres dans vos tests :

```csharp
[Fact]
public void TestWithParameters()
{
    var connectionString = TestContext.Parameters["ConnectionString"];
    // Utiliser la chaîne de connexion...
}
```

## 15.3.2. Tests de base de données

Les tests d'intégration avec des bases de données sont essentiels pour vérifier que vos requêtes, transactions et mappages d'entités fonctionnent correctement.

### Options pour les tests de base de données

Vous avez plusieurs options pour tester les interactions avec une base de données :

1. **Base de données en mémoire** - rapide mais ne teste pas toutes les fonctionnalités du SGBD
2. **Base de données SQLite** - plus proche d'une BD réelle tout en restant légère
3. **Conteneurs Docker** - pour tester avec votre SGBD réel (SQL Server, PostgreSQL, etc.)
4. **Base de données de test locale** - option la plus fidèle mais potentiellement plus lourde

### Base de données en mémoire avec Entity Framework Core

```csharp
using Microsoft.EntityFrameworkCore;
using System;
using Xunit;

public class ClientRepositoryTests
{
    private DbContextOptions<ApplicationDbContext> CreateNewContextOptions()
    {
        // Crée une base de données en mémoire avec un nom unique pour l'isolation
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        return options;
    }

    [Fact]
    public async Task GetClientById_ExistingClient_ReturnsClient()
    {
        // Arrange
        var options = CreateNewContextOptions();

        // Préparer les données
        using (var context = new ApplicationDbContext(options))
        {
            context.Clients.Add(new Client { Id = 1, Name = "Test Client" });
            await context.SaveChangesAsync();
        }

        // Act
        using (var context = new ApplicationDbContext(options))
        {
            var repository = new ClientRepository(context);
            var client = await repository.GetByIdAsync(1);

            // Assert
            Assert.NotNull(client);
            Assert.Equal("Test Client", client.Name);
        }
    }
}
```

### Tests avec SQLite

SQLite est plus proche d'une base de données relationnelle réelle :

```csharp
[Fact]
public async Task GetClientById_WithSqlite_ReturnsClient()
{
    // Arrange - Créer une connexion SQLite en mémoire
    var connection = new SqliteConnection("DataSource=:memory:");
    connection.Open();

    try
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite(connection)
            .Options;

        // Créer le schéma et ajouter des données de test
        using (var context = new ApplicationDbContext(options))
        {
            context.Database.EnsureCreated();
            context.Clients.Add(new Client { Id = 1, Name = "Test Client" });
            await context.SaveChangesAsync();
        }

        // Act - Utiliser un nouveau contexte pour le test
        using (var context = new ApplicationDbContext(options))
        {
            var repository = new ClientRepository(context);
            var client = await repository.GetByIdAsync(1);

            // Assert
            Assert.NotNull(client);
            Assert.Equal("Test Client", client.Name);
        }
    }
    finally
    {
        connection.Close();
    }
}
```

### Utilisation de Respawn pour la réinitialisation de base de données

Pour les tests qui utilisent une vraie base de données, l'outil Respawn peut aider à réinitialiser l'état entre les tests :

```csharp
// dotnet add package Respawn

[Collection("Database")]
public class DatabaseTests
{
    private static Checkpoint _checkpoint = new Checkpoint
    {
        TablesToIgnore = new[] { "__EFMigrationsHistory" }
    };

    private readonly string _connectionString = "Server=localhost;Database=TestDb;User=sa;Password=P@ssw0rd;";

    [Fact]
    public async Task Test1()
    {
        // Réinitialiser la base de données
        await _checkpoint.Reset(_connectionString);

        // Exécuter le test...
    }
}
```

### Tests de procédures stockées

Pour tester des procédures stockées ou du SQL brut :

```csharp
[Fact]
public async Task ExecuteStoredProcedure_ReturnsExpectedResults()
{
    // Arrange
    using var connection = new SqlConnection(_connectionString);
    await connection.OpenAsync();

    // Créer la procédure stockée pour le test
    using (var command = connection.CreateCommand())
    {
        command.CommandText = @"
            CREATE PROCEDURE GetTopClients
            AS
            BEGIN
                SELECT TOP 5 * FROM Clients ORDER BY Revenue DESC
            END";
        await command.ExecuteNonQueryAsync();
    }

    // Act
    var results = new List<Client>();
    using (var command = connection.CreateCommand())
    {
        command.CommandText = "GetTopClients";
        command.CommandType = CommandType.StoredProcedure;

        using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            results.Add(new Client
            {
                Id = reader.GetInt32(0),
                Name = reader.GetString(1),
                Revenue = reader.GetDecimal(2)
            });
        }
    }

    // Assert
    Assert.NotEmpty(results);
    Assert.True(results.Count <= 5);
}
```

## 15.3.3. Tests d'API

Les tests d'API permettent de vérifier que vos points d'accès HTTP fonctionnent correctement et renvoient les réponses attendues.

### Tests d'API Web ASP.NET Core

Pour tester une API ASP.NET Core, utilisez `WebApplicationFactory` :

```csharp
public class ClientApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public ClientApiTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Remplacer les services réels par des services de test
                var descriptor = services.SingleOrDefault(d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
                if (descriptor != null)
                {
                    services.Remove(descriptor);
                }

                services.AddDbContext<ApplicationDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestApi");
                });

                // Initialiser la base de données
                var sp = services.BuildServiceProvider();
                using var scope = sp.CreateScope();
                var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
                db.Database.EnsureCreated();

                if (!db.Clients.Any())
                {
                    db.Clients.Add(new Client { Id = 1, Name = "Test Client" });
                    db.SaveChanges();
                }
            });
        });

        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetClients_ReturnsSuccessAndCorrectContentType()
    {
        // Act
        var response = await _client.GetAsync("/api/clients");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json; charset=utf-8",
            response.Content.Headers.ContentType.ToString());
    }

    [Fact]
    public async Task GetClientById_ExistingId_ReturnsClient()
    {
        // Act
        var response = await _client.GetAsync("/api/clients/1");

        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        var client = JsonSerializer.Deserialize<Client>(content,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true });

        Assert.NotNull(client);
        Assert.Equal(1, client.Id);
        Assert.Equal("Test Client", client.Name);
    }

    [Fact]
    public async Task CreateClient_ValidClient_ReturnsCreatedResponse()
    {
        // Arrange
        var newClient = new Client { Name = "New Client" };
        var content = new StringContent(
            JsonSerializer.Serialize(newClient),
            Encoding.UTF8,
            "application/json");

        // Act
        var response = await _client.PostAsync("/api/clients", content);

        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);

        // Vérifier que l'emplacement de la ressource est correctement renvoyé
        Assert.NotNull(response.Headers.Location);

        // Vérifier que le client a été créé en faisant une requête GET
        var getResponse = await _client.GetAsync(response.Headers.Location);
        getResponse.EnsureSuccessStatusCode();
    }
}
```

### Tests d'API externes

Pour tester les interactions avec des API externes, vous pouvez utiliser des outils de mock HTTP comme WireMock.NET :

```csharp
// dotnet add package WireMock.Net

[Fact]
public async Task WeatherService_GetCurrentWeather_ReturnsWeatherData()
{
    // Arrange - Configurer un serveur mock
    var server = WireMockServer.Start();

    // Configurer le stub pour simuler l'API externe
    server
        .Given(Request.Create().WithPath("/api/weather/paris").UsingGet())
        .RespondWith(Response.Create()
            .WithStatusCode(200)
            .WithHeader("Content-Type", "application/json")
            .WithBody(@"{""temperature"":22,""condition"":""Sunny""}"));

    // Créer le service avec l'URL mockée
    var weatherService = new WeatherService(server.Urls[0]);

    // Act
    var weather = await weatherService.GetCurrentWeatherAsync("paris");

    // Assert
    Assert.NotNull(weather);
    Assert.Equal(22, weather.Temperature);
    Assert.Equal("Sunny", weather.Condition);

    // Nettoyer
    server.Stop();
}
```

### Tests d'authentification et d'autorisation

Tester les aspects de sécurité est crucial pour les API :

```csharp
[Fact]
public async Task SecureEndpoint_WithoutToken_ReturnsUnauthorized()
{
    // Act
    var response = await _client.GetAsync("/api/secure-resource");

    // Assert
    Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
}

[Fact]
public async Task SecureEndpoint_WithValidToken_ReturnsSuccess()
{
    // Arrange - Créer un client avec un token JWT valide
    var factory = _factory.WithWebHostBuilder(builder =>
    {
        builder.ConfigureTestServices(services =>
        {
            // Configurer l'authentification de test
            services.AddAuthentication("Test")
                .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>(
                    "Test", options => { });
        });
    });

    var client = factory.CreateClient();
    client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Test");

    // Act
    var response = await client.GetAsync("/api/secure-resource");

    // Assert
    response.EnsureSuccessStatusCode();
}

// Handler d'authentification pour les tests
public class TestAuthHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    public TestAuthHandler(
        IOptionsMonitor<AuthenticationSchemeOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock)
        : base(options, logger, encoder, clock)
    {
    }

    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Créer un principal avec les claims nécessaires pour les tests
        var claims = new[] {
            new Claim(ClaimTypes.Name, "TestUser"),
            new Claim(ClaimTypes.Role, "Admin")
        };
        var identity = new ClaimsIdentity(claims, "Test");
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, "Test");

        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

## 15.3.4. Tests d'interface utilisateur

Les tests d'interface utilisateur vérifient que l'interface fonctionne correctement du point de vue de l'utilisateur.

### Tests de composants Blazor

Pour tester des composants Blazor, utilisez bUnit :

```csharp
// dotnet add package bunit

[Fact]
public void CounterComponent_ButtonClick_IncrementCounter()
{
    // Arrange
    using var ctx = new TestContext();
    var cut = ctx.RenderComponent<Counter>();

    // Act - Cliquer sur le bouton
    cut.Find("button").Click();

    // Assert - Vérifier que le compteur a été incrémenté
    cut.Find("p").MarkupMatches("<p>Current count: 1</p>");
}
```

### Tests avec Selenium WebDriver

Pour tester des applications web via un navigateur réel :

```csharp
// dotnet add package Selenium.WebDriver
// dotnet add package Selenium.WebDriver.ChromeDriver

[Fact]
public void LoginPage_ValidCredentials_RedirectsToDashboard()
{
    // Arrange
    using var driver = new ChromeDriver();
    driver.Navigate().GoToUrl("https://localhost:5001/login");

    // Act
    driver.FindElement(By.Id("username")).SendKeys("testuser");
    driver.FindElement(By.Id("password")).SendKeys("password");
    driver.FindElement(By.Id("login-button")).Click();

    // Attendre le chargement de la page
    var wait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));
    wait.Until(d => d.Url.Contains("/dashboard"));

    // Assert
    Assert.Contains("/dashboard", driver.Url);
    Assert.Contains("Welcome", driver.PageSource);
}
```

### Tests de WPF avec TestStack.White

Pour les applications de bureau WPF :

```csharp
// dotnet add package TestStack.White

[Fact]
public void MainWindow_AddButtonClick_AddsNewItem()
{
    // Arrange - Démarrer l'application
    var application = Application.Launch("MyWpfApp.exe");
    var window = application.GetWindow("Main Window", InitializeOption.NoCache);

    // Act - Interagir avec l'UI
    var textBox = window.Get<TextBox>("NameTextBox");
    textBox.Text = "Test Item";

    var addButton = window.Get<Button>("AddButton");
    addButton.Click();

    // Assert - Vérifier les résultats
    var listBox = window.Get<ListBox>("ItemsListBox");
    Assert.Equal(1, listBox.Items.Count);
    Assert.Equal("Test Item", listBox.Items[0].Text);

    // Nettoyer
    application.Close();
}
```

### Tests d'accessibilité

Vérifier l'accessibilité de votre application web est important :

```csharp
// dotnet add package Deque.AxeCore.Selenium

[Fact]
public void HomePage_ShouldBeAccessible()
{
    // Arrange
    using var driver = new ChromeDriver();
    driver.Navigate().GoToUrl("https://localhost:5001");

    // Act - Exécuter une analyse d'accessibilité
    var axe = new AxeBuilder(driver).Analyze();

    // Assert - Vérifier qu'il n'y a pas de violations
    Assert.Empty(axe.Violations);
}
```

## 15.3.5. Tests de performance

Les tests de performance permettent de vérifier que votre application répond rapidement et peut gérer la charge prévue.

### Tests de charge avec NBomber

NBomber est une bibliothèque de tests de charge moderne pour .NET :

```csharp
// dotnet add package NBomber
// dotnet add package NBomber.Http

[Fact]
public void Api_UnderLoad_RespondsWithinAcceptableTime()
{
    // Définir le scénario de test
    var scenario = ScenarioBuilder.CreateScenario("API Load Test", async context =>
    {
        var client = new HttpClient();

        // Simuler une requête client
        var response = await client.GetAsync("https://localhost:5001/api/clients");

        // Vérifier la réponse
        return response.IsSuccessStatusCode
            ? Response.Ok()
            : Response.Fail();
    })
    .WithWarmUpDuration(TimeSpan.FromSeconds(5))
    .WithLoadSimulations(
        Simulation.KeepConstant(10, TimeSpan.FromSeconds(30)) // 10 requêtes par seconde pendant 30 secondes
    );

    // Exécuter le test de charge
    var stats = NBomberRunner
        .RegisterScenarios(scenario)
        .WithReportFormats(ReportFormat.Txt, ReportFormat.Csv)
        .WithReportFileName("load-test-results")
        .Run();

    // Vérifier les métriques
    var stepStats = stats.ScenarioStats[0].StepStats[0];
    Assert.True(stepStats.Ok.Request.RPS > 8); // Au moins 8 requêtes par seconde
    Assert.True(stepStats.Ok.Latency.Percent95 < 500); // 95% des requêtes en moins de 500ms
}
```

### Tests de performance avec BenchmarkDotNet

Pour mesurer précisément les performances de code spécifique :

```csharp
// dotnet add package BenchmarkDotNet

[MemoryDiagnoser]
public class ServiceBenchmarks
{
    private ClientService _service;
    private ApplicationDbContext _dbContext;

    [GlobalSetup]
    public void Setup()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase("BenchmarkDb")
            .Options;

        _dbContext = new ApplicationDbContext(options);
        // Ajouter des données de test
        for (int i = 1; i <= 1000; i++)
        {
            _dbContext.Clients.Add(new Client { Id = i, Name = $"Client {i}" });
        }
        _dbContext.SaveChanges();

        _service = new ClientService(_dbContext);
    }

    [Benchmark]
    public async Task GetAllClients()
    {
        var clients = await _service.GetAllClientsAsync();
        return clients;
    }

    [Benchmark]
    public async Task GetClientById()
    {
        var client = await _service.GetClientByIdAsync(500);
        return client;
    }

    [GlobalCleanup]
    public void Cleanup()
    {
        _dbContext.Dispose();
    }
}

// Pour exécuter les benchmarks
public class Program
{
    public static void Main(string[] args)
    {
        var summary = BenchmarkRunner.Run<ServiceBenchmarks>();
    }
}
```

### Tests de performance avec dotnet-trace

Pour analyser les performances d'une application en cours d'exécution :

```bash
# Installer l'outil dotnet-trace
dotnet tool install -g dotnet-trace

# Démarrer le traçage
dotnet-trace collect --process-id <pid>

# Analyser le fichier de trace avec PerfView ou Visual Studio
```

### Tests de mémoire

Pour détecter les fuites de mémoire et autres problèmes :

```csharp
[Fact]
public async Task ProcessLargeDataSet_DoesNotLeakMemory()
{
    // Arrangement
    var service = new DataProcessingService();
    var data = GenerateLargeDataSet();

    // Action & Assertion
    var initialMemory = GC.GetTotalMemory(true);

    // Exécuter le traitement plusieurs fois
    for (int i = 0; i < 10; i++)
    {
        await service.ProcessDataAsync(data);

        // Forcer la collecte entre les exécutions
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }

    var finalMemory = GC.GetTotalMemory(true);

    // La mémoire ne devrait pas augmenter significativement
    // Tolérer une petite augmentation pour les allocations légitimes
    Assert.True(finalMemory - initialMemory < 1024 * 1024,
        "Fuite de mémoire détectée");
}
```

## Bonnes pratiques pour les tests d'intégration

Pour conclure, voici quelques conseils pour écrire des tests d'intégration efficaces :

1. **Isolez l'environnement de test** : Assurez-vous que les tests n'interfèrent pas entre eux ou avec les données de production.

2. **Utilisez des données de test réalistes** : Les données de test doivent refléter des scénarios réels pour être pertinentes.

3. **Nettoyez après les tests** : Restaurez l'état initial après chaque test, en particulier pour les ressources partagées.

4. **Utilisez des timeouts appropriés** : Les opérations d'intégration peuvent prendre plus de temps que les tests unitaires.

5. **Exécutez les tests d'intégration séparément** : Ces tests sont généralement plus lents, exécutez-les séparément des tests unitaires.

6. **Utilisez l'intégration continue** : Intégrez ces tests dans votre pipeline CI/CD pour détecter rapidement les problèmes.

7. **Équilibrez couverture et performance** : Trouvez un équilibre entre la couverture des tests et leur temps d'exécution.

8. **Testez les cas limites** : N'oubliez pas de tester les scénarios d'erreur et les cas limites, pas uniquement le chemin heureux.

## Conclusion

Les tests d'intégration sont un complément essentiel aux tests unitaires pour garantir la qualité de votre application. Ils permettent de vérifier que les différents composants fonctionnent correctement ensemble dans des conditions proches de la réalité.

En combinant tests unitaires (pour la logique métier), tests d'intégration (pour les interactions entre composants) et tests d'interface utilisateur (pour l'expérience utilisateur), vous pouvez construire une stratégie de test complète qui améliore la fiabilité de votre application.

## Ressources complémentaires

- [Documentation officielle des tests d'intégration ASP.NET Core](https://docs.microsoft.com/fr-fr/aspnet/core/test/integration-tests)
- [Entity Framework Core et tests](https://docs.microsoft.com/fr-fr/ef/core/testing/)
- [NBomber - Framework de test de charge](https://nbomber.com/)
- [BenchmarkDotNet](https://benchmarkdotnet.org/)
- [Selenium WebDriver pour .NET](https://www.selenium.dev/documentation/webdriver/)
- [bUnit pour tester Blazor](https://bunit.dev/)

⏭️ 15.4. [Analyse de code et refactoring](/15-tests-et-qualite-de-code/15-4-analyse-de-code-et-refactoring.md)
