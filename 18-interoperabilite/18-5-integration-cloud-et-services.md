# 18.5. Intégration cloud et services

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

Bienvenue dans ce chapitre consacré à l'intégration de C# avec les services cloud et les architectures modernes. Le développement logiciel a considérablement évolué ces dernières années, passant d'applications monolithiques à des architectures distribuées, sans serveur (serverless) et basées sur des événements.

Dans ce chapitre, nous explorerons comment C# s'intègre parfaitement dans cet écosystème moderne grâce à :

- Azure Functions, pour exécuter facilement du code à la demande sans gérer d'infrastructure
- gRPC, un système d'appel de procédure à distance haute performance
- GraphQL, une alternative flexible aux API REST traditionnelles
- Les microservices en C#, pour créer des applications modulaires et évolutives
- L'architecture pilotée par les événements, pour des systèmes réactifs et découplés

Ces technologies vous permettront de créer des applications modernes, évolutives et hautement disponibles. Que vous développiez une petite application ou un système d'entreprise complexe, ces approches vous fourniront les outils nécessaires pour relever les défis actuels du développement logiciel.

Commençons notre exploration de ces passionnantes technologies !

## 18.5.1. Azure Functions

### Qu'est-ce qu'Azure Functions ?

Azure Functions est un service "serverless" (sans serveur) proposé par Microsoft Azure qui vous permet d'exécuter des morceaux de code (appelés "fonctions") sans avoir à vous soucier de l'infrastructure sous-jacente. C'est une solution idéale pour :

- Traiter des données en réponse à des événements (upload de fichier, message reçu...)
- Exécuter des tâches planifiées (nettoyage de base de données, rapports...)
- Créer des API légères et à faible coût
- Traiter des flux de données en temps réel

Les avantages principaux d'Azure Functions incluent :

- Paiement à l'usage (vous ne payez que le temps d'exécution)
- Mise à l'échelle automatique selon la demande
- Déploiement rapide et simple
- Intégration facile avec d'autres services Azure
- Support de plusieurs langages dont C# et F#

### Configuration de l'environnement

Pour commencer avec Azure Functions, vous aurez besoin de :

1. **Un compte Azure** : Créez un compte gratuit sur [portal.azure.com](https://portal.azure.com) si vous n'en avez pas déjà un.

2. **Visual Studio ou VS Code** :
   - Pour Visual Studio : Assurez-vous d'avoir les charges de travail "Développement Azure" et "Développement web et ASP.NET" installées.
   - Pour VS Code : Installez l'extension "Azure Functions".

3. **Azure Functions Core Tools** : Un outil en ligne de commande pour le développement local.
   ```bash
   npm install -g azure-functions-core-tools@4
   ```

### Création de votre première Azure Function

Commençons par créer une fonction simple qui répond aux requêtes HTTP.

#### Avec Visual Studio

1. Lancez Visual Studio et créez un nouveau projet.
2. Sélectionnez "Azure Functions" comme type de projet.
3. Nommez votre projet (par ex. "MaPremiereFonction").
4. Sélectionnez ".NET Core 6.0" comme version du framework.
5. Choisissez "HTTP trigger" comme modèle de fonction.
6. Sélectionnez "Anonymous" pour le niveau d'autorisation (pour simplifier ce tutoriel).

Visual Studio va créer un projet avec une structure similaire à celle-ci :

```
MaPremiereFonction/
├── Properties/
├── Function1.cs
├── host.json
├── local.settings.json
└── MaPremiereFonction.csproj
```

Le fichier principal, `Function1.cs`, devrait ressembler à ceci :

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace MaPremiereFonction
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string name = req.Query["name"];

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            name = name ?? data?.name;

            string responseMessage = string.IsNullOrEmpty(name)
                ? "Cette fonction HTTP a été exécutée avec succès. Passez un nom dans le paramètre query string ou dans le corps de la requête pour une réponse personnalisée."
                : $"Bonjour, {name}. Cette fonction HTTP a été exécutée avec succès.";

            return new OkObjectResult(responseMessage);
        }
    }
}
```

#### Avec VS Code

1. Ouvrez VS Code.
2. Appuyez sur `F1`, tapez "Azure Functions: Create New Project" et sélectionnez cette option.
3. Sélectionnez un dossier où créer le projet.
4. Choisissez C# comme langage.
5. Sélectionnez ".NET 6.0" comme runtime.
6. Choisissez "HTTP trigger" comme modèle.
7. Nommez votre fonction (par ex. "MaPremiereHttpFunction").
8. Sélectionnez "Anonymous" comme niveau d'autorisation.
9. Choisissez "Add to workspace" pour ajouter le projet à votre espace de travail.

### Test local de votre fonction

1. Appuyez sur `F5` dans Visual Studio ou VS Code pour démarrer le projet en mode débogage.
2. L'émulateur Azure Functions démarre et affiche une URL locale pour votre fonction (généralement `http://localhost:7071/api/Function1`).
3. Ouvrez un navigateur et accédez à cette URL.
4. Vous devriez voir le message de réponse par défaut. Essayez d'ajouter un paramètre, comme `http://localhost:7071/api/Function1?name=Alice`.

### Amélioration de notre fonction

Modifions notre fonction pour créer une API de gestion de tâches simple :

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System.Collections.Generic;
using System.Linq;

namespace MaPremiereFonction
{
    public static class TaskFunction
    {
        // Liste simple pour stocker les tâches (en production, vous utiliseriez une base de données)
        private static readonly List<Task> _tasks = new List<Task>
        {
            new Task { Id = 1, Title = "Apprendre Azure Functions", IsCompleted = false },
            new Task { Id = 2, Title = "Faire les courses", IsCompleted = true },
            new Task { Id = 3, Title = "Préparer la présentation", IsCompleted = false }
        };

        // Modèle de données pour une tâche
        public class Task
        {
            public int Id { get; set; }
            public string Title { get; set; }
            public bool IsCompleted { get; set; }
        }

        [FunctionName("GetAllTasks")]
        public static IActionResult GetAllTasks(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "tasks")] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Récupération de toutes les tâches.");
            return new OkObjectResult(_tasks);
        }

        [FunctionName("GetTaskById")]
        public static IActionResult GetTaskById(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "tasks/{id}")] HttpRequest req,
            ILogger log, int id)
        {
            log.LogInformation($"Récupération de la tâche avec l'ID: {id}");

            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
            {
                return new NotFoundResult();
            }

            return new OkObjectResult(task);
        }

        [FunctionName("CreateTask")]
        public static async Task<IActionResult> CreateTask(
            [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "tasks")] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Création d'une nouvelle tâche.");

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var input = JsonConvert.DeserializeObject<Task>(requestBody);

            // Validation simple
            if (string.IsNullOrEmpty(input.Title))
            {
                return new BadRequestObjectResult("Le titre de la tâche est requis.");
            }

            // Générer un nouvel ID
            var newId = _tasks.Count > 0 ? _tasks.Max(t => t.Id) + 1 : 1;

            var newTask = new Task
            {
                Id = newId,
                Title = input.Title,
                IsCompleted = input.IsCompleted
            };

            _tasks.Add(newTask);

            return new CreatedResult($"/api/tasks/{newTask.Id}", newTask);
        }

        [FunctionName("UpdateTask")]
        public static async Task<IActionResult> UpdateTask(
            [HttpTrigger(AuthorizationLevel.Anonymous, "put", Route = "tasks/{id}")] HttpRequest req,
            ILogger log, int id)
        {
            log.LogInformation($"Mise à jour de la tâche avec l'ID: {id}");

            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
            {
                return new NotFoundResult();
            }

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var updatedTask = JsonConvert.DeserializeObject<Task>(requestBody);

            // Validation simple
            if (string.IsNullOrEmpty(updatedTask.Title))
            {
                return new BadRequestObjectResult("Le titre de la tâche est requis.");
            }

            // Mettre à jour la tâche
            task.Title = updatedTask.Title;
            task.IsCompleted = updatedTask.IsCompleted;

            return new OkObjectResult(task);
        }

        [FunctionName("DeleteTask")]
        public static IActionResult DeleteTask(
            [HttpTrigger(AuthorizationLevel.Anonymous, "delete", Route = "tasks/{id}")] HttpRequest req,
            ILogger log, int id)
        {
            log.LogInformation($"Suppression de la tâche avec l'ID: {id}");

            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
            {
                return new NotFoundResult();
            }

            _tasks.Remove(task);

            return new OkResult();
        }
    }
}
```

### Déclencheurs et liaisons

Azure Functions supporte différents types de déclencheurs et de liaisons qui vous permettent d'interagir avec d'autres services. Dans notre exemple, nous avons utilisé le déclencheur HTTP, mais il en existe bien d'autres :

| Type de déclencheur | Description |
|---------------------|-------------|
| **HTTP** | Exécute le code en réponse à une requête HTTP |
| **Timer** | Exécute le code selon un planning prédéfini |
| **Blob Storage** | Exécute le code quand un fichier est ajouté/modifié dans Azure Blob Storage |
| **Queue Storage** | Exécute le code quand un message est ajouté à une file d'attente Azure |
| **Cosmos DB** | Exécute le code quand des données sont modifiées dans une base Cosmos DB |
| **Event Hub** | Exécute le code en réponse à des événements dans Azure Event Hubs |
| **Service Bus** | Exécute le code en réponse à des messages dans Azure Service Bus |

### Exemple avec un déclencheur Timer

Voici un exemple de fonction qui s'exécute toutes les 5 minutes pour nettoyer les tâches terminées depuis plus de 7 jours :

```csharp
[FunctionName("CleanupCompletedTasks")]
public static void Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, // Format CRON: minutes heures jour_du_mois mois jour_de_la_semaine
    ILogger log)
{
    log.LogInformation($"Fonction de nettoyage exécutée à: {DateTime.Now}");

    // En production, vous supprimeriez des éléments d'une base de données
    // Simulation avec notre liste en mémoire
    var tasksToRemove = _tasks.Where(t => t.IsCompleted &&
        /* En réalité, vous vérifieriez la date de complétion */
        false).ToList();

    foreach (var task in tasksToRemove)
    {
        _tasks.Remove(task);
        log.LogInformation($"Tâche supprimée: {task.Id} - {task.Title}");
    }
}
```

### Déploiement sur Azure

Pour déployer votre fonction sur Azure :

#### Avec Visual Studio

1. Cliquez-droit sur le projet dans l'Explorateur de solutions.
2. Sélectionnez "Publier..."
3. Choisissez "Azure" comme cible.
4. Sélectionnez "Azure Function App" comme cible spécifique.
5. Créez une nouvelle Function App en cliquant sur le "+" ou sélectionnez une existante.
6. Configurez les options de l'App Service, choisissez un plan de tarification.
7. Cliquez sur "Finir", puis "Publier".

#### Avec VS Code

1. Cliquez sur l'icône Azure dans la barre latérale.
2. Dans la section "FUNCTIONS", cliquez sur l'icône de déploiement (flèche vers le haut).
3. Sélectionnez "Deploy to Function App...".
4. Sélectionnez votre souscription Azure.
5. Sélectionnez "Create new Function App in Azure...".
6. Entrez un nom globalement unique pour votre Function App.
7. Sélectionnez un runtime (.NET 6).
8. Choisissez un emplacement géographique.

Le déploiement peut prendre quelques minutes. Une fois terminé, vous recevrez une notification vous indiquant comment accéder à votre fonction.

### Bonnes pratiques pour Azure Functions

1. **Conception pour la scalabilité** : Gardez vos fonctions légères et concentrées sur une seule tâche.
2. **Gestion appropriée des erreurs** : Implémentez une gestion robuste des exceptions.
3. **Surveillance et journalisation** : Utilisez ILogger de manière efficace pour suivre l'exécution.
4. **Sécurité** : Utilisez des niveaux d'autorisation appropriés (pas Anonymous en production).
5. **Configuration** : Stockez les paramètres dans les paramètres d'application plutôt qu'en dur dans le code.
6. **Tests** : Écrivez des tests unitaires pour vos fonctions.
7. **Durée d'exécution** : Gardez les fonctions rapides pour éviter des timeouts ou des coûts élevés.

## 18.5.2. gRPC

### Qu'est-ce que gRPC ?

gRPC (gRPC Remote Procedure Call) est un framework d'appel de procédure à distance (RPC) open-source développé par Google. Il utilise HTTP/2 pour le transport, Protocol Buffers (protobuf) comme langage de description d'interface (IDL), et fournit des fonctionnalités comme l'authentification, la compression, et le streaming bidirectionnel.

Pourquoi utiliser gRPC ?

- **Haute performance** : Plus rapide que REST grâce à HTTP/2 et à la sérialisation binaire.
- **Contrat d'API stricte** : Grâce aux fichiers .proto, les interfaces sont clairement définies.
- **Génération de code** : Génération automatique du code client et serveur dans de nombreux langages.
- **Streaming bidirectionnel** : Support du streaming unidirectionnel et bidirectionnel.
- **Interopérabilité** : Fonctionne entre différents langages et plateformes.

### Configuration d'un projet gRPC avec C#

Commençons par créer un service gRPC simple avec ASP.NET Core.

#### Création du service gRPC

1. Ouvrez Visual Studio et créez un nouveau projet.
2. Sélectionnez "Service gRPC ASP.NET Core" comme type de projet.
3. Nommez le projet (par ex. "MonServiceGrpc") et cliquez sur "Créer".

Visual Studio va générer un projet avec la structure suivante :

```
MonServiceGrpc/
├── Properties/
├── Protos/
│   └── greet.proto
├── Services/
│   └── GreeterService.cs
├── appsettings.json
├── Program.cs
└── MonServiceGrpc.csproj
```

Le fichier `greet.proto` définit l'interface du service :

```protobuf
syntax = "proto3";

option csharp_namespace = "MonServiceGrpc";

package greet;

// Le service de salutation
service Greeter {
  // Envoie une salutation
  rpc SayHello (HelloRequest) returns (HelloReply);
}

// La requête contenant le nom de l'utilisateur
message HelloRequest {
  string name = 1;
}

// La réponse contenant le message de salutation
message HelloReply {
  string message = 1;
}
```

Et le fichier `GreeterService.cs` implémente ce service :

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Grpc.Core;
using Microsoft.Extensions.Logging;

namespace MonServiceGrpc
{
    public class GreeterService : Greeter.GreeterBase
    {
        private readonly ILogger<GreeterService> _logger;
        public GreeterService(ILogger<GreeterService> logger)
        {
            _logger = logger;
        }

        public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        {
            return Task.FromResult(new HelloReply
            {
                Message = "Bonjour " + request.Name
            });
        }
    }
}
```

### Création d'un service gRPC plus complet

Modifions notre service pour qu'il gère des tâches, comme dans notre exemple Azure Functions. Commençons par définir un nouveau fichier proto.

Créez un nouveau fichier `tasks.proto` dans le dossier `Protos` :

```protobuf
syntax = "proto3";

option csharp_namespace = "MonServiceGrpc";

package tasks;

service TaskService {
  // Récupère toutes les tâches
  rpc GetAllTasks (EmptyRequest) returns (TaskList);

  // Récupère une tâche par ID
  rpc GetTaskById (TaskIdRequest) returns (TaskItem);

  // Crée une nouvelle tâche
  rpc CreateTask (TaskItem) returns (TaskItem);

  // Met à jour une tâche existante
  rpc UpdateTask (TaskItem) returns (TaskItem);

  // Supprime une tâche
  rpc DeleteTask (TaskIdRequest) returns (EmptyResponse);

  // Flux de tâches (démonstration de streaming côté serveur)
  rpc WatchTasks (EmptyRequest) returns (stream TaskItem);
}

// Requête vide
message EmptyRequest {}

// Réponse vide
message EmptyResponse {}

// Requête contenant un ID de tâche
message TaskIdRequest {
  int32 id = 1;
}

// Modèle d'une tâche
message TaskItem {
  int32 id = 1;
  string title = 2;
  bool is_completed = 3;
}

// Liste de tâches
message TaskList {
  repeated TaskItem tasks = 1;
}
```

Ensuite, mettez à jour le fichier `.csproj` pour inclure ce nouveau fichier proto :

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
  <Protobuf Include="Protos\tasks.proto" GrpcServices="Server" />
</ItemGroup>
```

Créez maintenant un nouveau fichier de service pour implémenter l'interface TaskService :

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Grpc.Core;
using Microsoft.Extensions.Logging;

namespace MonServiceGrpc
{
    public class TaskService : Tasks.TaskService.TaskServiceBase
    {
        private readonly ILogger<TaskService> _logger;

        // Liste simple pour stocker les tâches (en production, vous utiliseriez une base de données)
        private static readonly List<TaskItem> _tasks = new List<TaskItem>
        {
            new TaskItem { Id = 1, Title = "Apprendre gRPC", IsCompleted = false },
            new TaskItem { Id = 2, Title = "Faire les courses", IsCompleted = true },
            new TaskItem { Id = 3, Title = "Préparer la présentation", IsCompleted = false }
        };

        public TaskService(ILogger<TaskService> logger)
        {
            _logger = logger;
        }

        public override Task<TaskList> GetAllTasks(EmptyRequest request, ServerCallContext context)
        {
            _logger.LogInformation("Récupération de toutes les tâches");

            var response = new TaskList();
            response.Tasks.AddRange(_tasks);

            return Task.FromResult(response);
        }

        public override Task<TaskItem> GetTaskById(TaskIdRequest request, ServerCallContext context)
        {
            _logger.LogInformation($"Récupération de la tâche avec l'ID: {request.Id}");

            var task = _tasks.FirstOrDefault(t => t.Id == request.Id);
            if (task == null)
            {
                throw new RpcException(new Status(StatusCode.NotFound, $"Tâche avec ID {request.Id} non trouvée"));
            }

            return Task.FromResult(task);
        }

        public override Task<TaskItem> CreateTask(TaskItem request, ServerCallContext context)
        {
            _logger.LogInformation("Création d'une nouvelle tâche");

            // Validation
            if (string.IsNullOrEmpty(request.Title))
            {
                throw new RpcException(new Status(StatusCode.InvalidArgument, "Le titre de la tâche est requis"));
            }

            // Générer un nouvel ID
            var newId = _tasks.Count > 0 ? _tasks.Max(t => t.Id) + 1 : 1;

            var newTask = new TaskItem
            {
                Id = newId,
                Title = request.Title,
                IsCompleted = request.IsCompleted
            };

            _tasks.Add(newTask);

            return Task.FromResult(newTask);
        }

        public override Task<TaskItem> UpdateTask(TaskItem request, ServerCallContext context)
        {
            _logger.LogInformation($"Mise à jour de la tâche avec l'ID: {request.Id}");

            var task = _tasks.FirstOrDefault(t => t.Id == request.Id);
            if (task == null)
            {
                throw new RpcException(new Status(StatusCode.NotFound, $"Tâche avec ID {request.Id} non trouvée"));
            }

            // Validation
            if (string.IsNullOrEmpty(request.Title))
            {
                throw new RpcException(new Status(StatusCode.InvalidArgument, "Le titre de la tâche est requis"));
            }

            // Mettre à jour la tâche
            task.Title = request.Title;
            task.IsCompleted = request.IsCompleted;

            return Task.FromResult(task);
        }

        public override Task<EmptyResponse> DeleteTask(TaskIdRequest request, ServerCallContext context)
        {
            _logger.LogInformation($"Suppression de la tâche avec l'ID: {request.Id}");

            var task = _tasks.FirstOrDefault(t => t.Id == request.Id);
            if (task == null)
            {
                throw new RpcException(new Status(StatusCode.NotFound, $"Tâche avec ID {request.Id} non trouvée"));
            }

            _tasks.Remove(task);

            return Task.FromResult(new EmptyResponse());
        }

        public override async Task WatchTasks(EmptyRequest request, IServerStreamWriter<TaskItem> responseStream, ServerCallContext context)
        {
            _logger.LogInformation("Streaming des tâches démarré");

            // Envoie initiale de toutes les tâches
            foreach (var task in _tasks)
            {
                await responseStream.WriteAsync(task);
                // Simule un délai pour démontrer le streaming
                await System.Threading.Tasks.Task.Delay(500);
            }

            // En production, vous pourriez continuer à envoyer des mises à jour quand les tâches changent
            // ou utiliser un CancellationToken pour arrêter le streaming quand le client se déconnecte

            _logger.LogInformation("Streaming des tâches terminé");
        }
    }
}
```

Enfin, mettez à jour `Program.cs` pour enregistrer ce nouveau service :

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace MonServiceGrpc
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            // Ajouter les services au conteneur.
            builder.Services.AddGrpc();

            var app = builder.Build();

            // Configure le pipeline de requêtes HTTP.
            app.MapGrpcService<GreeterService>();
            app.MapGrpcService<TaskService>();
            app.MapGet("/", () => "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

            app.Run();
        }
    }
}
```

### Création d'un client gRPC

Maintenant, créons un client console simple pour notre service gRPC.

1. Ajoutez un nouveau projet "Application Console" à votre solution et nommez-le "GrpcClient".
2. Ajoutez les packages NuGet suivants au client :
   - `Google.Protobuf`
   - `Grpc.Net.Client`
   - `Grpc.Tools`

3. Ajoutez une référence aux fichiers proto du serveur. Modifiez le fichier `.csproj` du client :

```xml
<ItemGroup>
  <Protobuf Include="..\MonServiceGrpc\Protos\tasks.proto" GrpcServices="Client" />
</ItemGroup>
```

4. Créez le code client :

```csharp
using System;
using System.Threading.Tasks;
using Grpc.Net.Client;
using MonServiceGrpc;

namespace GrpcClient
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Client gRPC démarré...");

            // Création du canal gRPC
            using var channel = GrpcChannel.ForAddress("https://localhost:7042");

            // Création du client
            var client = new Tasks.TaskService.TaskServiceClient(channel);

            Console.WriteLine("1. Récupération de toutes les tâches");
            var allTasks = await client.GetAllTasksAsync(new EmptyRequest());
            foreach (var task in allTasks.Tasks)
            {
                Console.WriteLine($"  {task.Id}: {task.Title} ({(task.IsCompleted ? "Terminée" : "En cours")})");
            }

            Console.WriteLine("\n2. Création d'une nouvelle tâche");
            var newTask = await client.CreateTaskAsync(new TaskItem
            {
                Title = "Tâche créée depuis le client gRPC",
                IsCompleted = false
            });
            Console.WriteLine($"  Nouvelle tâche créée avec l'ID: {newTask.Id}");

            Console.WriteLine("\n3. Récupération d'une tâche par ID");
            try
            {
                var task = await client.GetTaskByIdAsync(new TaskIdRequest { Id = newTask.Id });
                Console.WriteLine($"  {task.Id}: {task.Title} ({(task.IsCompleted ? "Terminée" : "En cours")})");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"  Erreur: {ex.Message}");
            }

            Console.WriteLine("\n4. Mise à jour d'une tâche");
            var updatedTask = await client.UpdateTaskAsync(new TaskItem
            {
                Id = newTask.Id,
                Title = "Tâche mise à jour depuis le client gRPC",
                IsCompleted = true
            });
            Console.WriteLine($"  Tâche mise à jour: {updatedTask.Id}: {updatedTask.Title} ({(updatedTask.IsCompleted ? "Terminée" : "En cours")})");

            Console.WriteLine("\n5. Suppression d'une tâche");
            await client.DeleteTaskAsync(new TaskIdRequest { Id = newTask.Id });
            Console.WriteLine($"  Tâche avec ID {newTask.Id} supprimée");

            Console.WriteLine("\n6. Démonstration du streaming côté serveur");
            using var streamingCall = client.WatchTasks(new EmptyRequest());

            Console.WriteLine("  Flux de tâches commencé, appuyez sur une touche pour arrêter...");

            // Création d'un token d'annulation pour arrêter le streaming lorsque l'utilisateur appuie sur une touche
            var cts = new System.Threading.CancellationTokenSource();

            // Démarrer une tâche pour lire la console
            var consoleTask = Task.Run(() => {
                Console.ReadKey(true);
                cts.Cancel();
            });

            try
            {
                // Lire les tâches du flux jusqu'à ce que le token d'annulation soit activé
                await foreach (var task in streamingCall.ResponseStream.ReadAllAsync(cts.Token))
                {
                    Console.WriteLine($"  Reçu: {task.Id}: {task.Title} ({(task.IsCompleted ? "Terminée" : "En cours")})");
                }
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("  Streaming arrêté par l'utilisateur");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"  Erreur pendant le streaming: {ex.Message}");
            }

            Console.WriteLine("\nDémonstration terminée. Appuyez sur une touche pour quitter...");
            Console.ReadKey();
        }
    }
}
```

### Déploiement d'un service gRPC sur Azure

Pour déployer un service gRPC sur Azure, vous avez plusieurs options :

1. **Azure App Service** : Facile à configurer, mais ne supporte pas HTTP/2 sur toutes les configurations.
2. **Azure Kubernetes Service (AKS)** : Excellent pour les déploiements à grande échelle.
3. **Azure Container Instances** : Bonne option pour des conteneurs légers et simples.
4. **Azure Container Apps** : Un nouveau service de conteneurs serverless qui supporte bien les services gRPC.

Voici les étapes pour déployer sur Azure Container Apps :

1. Créez un Dockerfile dans votre projet gRPC :

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["MonServiceGrpc.csproj", "./"]
RUN dotnet restore "MonServiceGrpc.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "MonServiceGrpc.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MonServiceGrpc.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MonServiceGrpc.dll"]
```

2. Construisez et publiez l'image sur Azure Container Registry (ACR) :

```bash
# Création d'un registry Azure Container
az acr create --resource-group MonGroupe --name monregistryacr --sku Basic

# Connexion au registry
az acr login --name monregistryacr

# Construction et étiquetage de l'image
docker build -t monregistryacr.azurecr.io/monservicegrpc:latest .

# Poussée de l'image vers ACR
docker push monregistryacr.azurecr.io/monservicegrpc:latest
```

3. Créez une application de conteneur Azure :

```bash
# Création de l'environnement Container Apps
az containerapp env create --name mon-environnement --resource-group MonGroupe --location francecentral

# Création de l'application de conteneur
az containerapp create \
    --name mon-service-grpc \
    --resource-group MonGroupe \
    --environment mon-environnement \
    --image monregistryacr.azurecr.io/monservicegrpc:latest \
    --target-port 80 \
    --ingress external \
    --registry-server monregistryacr.azurecr.io \
    --registry-username monregistryacr \
    --registry-password $(az acr credential show -n monregistryacr --query "passwords[0].value" -o tsv)
```

### Bonnes pratiques pour gRPC

1. **Conception des contrats** : Concevez vos fichiers `.proto` avec soin car ils définissent votre API.
2. **Gestion des versions** : Utilisez des numéros de version dans les noms de package pour gérer l'évolution de votre API.
3. **Gestion des erreurs** : Utilisez les codes de statut appropriés et incluez des messages d'erreur détaillés.
4. **Sécurité** : Utilisez TLS pour sécuriser les communications.
5. **Streaming** : Utilisez le streaming pour les grandes collections ou les données en temps réel.
6. **Tests** : Écrivez des tests unitaires pour vos services gRPC.
7. **Documentation** : Documentez vos API gRPC avec des commentaires dans les fichiers `.proto`.

## 18.5.3. GraphQL

### Qu'est-ce que GraphQL ?

GraphQL est un langage de requête pour les API et un runtime pour exécuter ces requêtes. Contrairement aux API REST, où chaque endpoint retourne une structure de données fixe, GraphQL permet aux clients de spécifier exactement les données dont ils ont besoin. Cela réduit la quantité de données transférées et le nombre d'appels API.

Principaux avantages de GraphQL :

- **Requêtes précises** : Les clients spécifient exactement les données dont ils ont besoin
- **Une seule requête** : Récupération de plusieurs ressources en un seul appel API
- **Typage fort** : Schéma explicite qui décrit toutes les possibilités
- **Évolution sans versions** : Ajout de champs sans casser les clients existants
- **Introspection** : Les API peuvent être explorées et documentées automatiquement

### Création d'une API GraphQL avec Hot Chocolate

Hot Chocolate est une bibliothèque GraphQL pour .NET. Voici comment l'utiliser pour créer une API GraphQL :

1. Créez un nouveau projet ASP.NET Core Web API :

```bash
dotnet new webapi -n MonApiGraphQL
cd MonApiGraphQL
```

2. Ajoutez les packages Hot Chocolate :

```bash
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.Data
```

3. Créez des modèles de données :

```csharp
// Models/Task.cs
namespace MonApiGraphQL.Models
{
    public class Task
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }

        // Relation avec les tags
        public List<Tag> Tags { get; set; } = new List<Tag>();
    }
}

// Models/Tag.cs
namespace MonApiGraphQL.Models
{
    public class Tag
    {
        public int Id { get; set; }
        public string Name { get; set; }

        // Relation avec les tâches
        public List<Task> Tasks { get; set; } = new List<Task>();
    }
}
```

4. Créez un service de données (simulé pour cet exemple) :

```csharp
// Services/TaskService.cs
using MonApiGraphQL.Models;

namespace MonApiGraphQL.Services
{
    public class TaskService
    {
        private readonly List<Task> _tasks;
        private readonly List<Tag> _tags;
        private int _nextTaskId = 4;
        private int _nextTagId = 4;

        public TaskService()
        {
            // Création de tags
            _tags = new List<Tag>
            {
                new Tag { Id = 1, Name = "Urgent" },
                new Tag { Id = 2, Name = "Personnel" },
                new Tag { Id = 3, Name = "Travail" }
            };

            // Création de tâches
            _tasks = new List<Task>
            {
                new Task
                {
                    Id = 1,
                    Title = "Apprendre GraphQL",
                    Description = "Faire le tutoriel complet sur GraphQL",
                    IsCompleted = false,
                    DueDate = DateTime.Now.AddDays(7),
                    Tags = new List<Tag> { _tags[0], _tags[2] }
                },
                new Task
                {
                    Id = 2,
                    Title = "Faire les courses",
                    Description = "Acheter des fruits et légumes",
                    IsCompleted = true,
                    DueDate = DateTime.Now.AddDays(-1),
                    Tags = new List<Tag> { _tags[1] }
                },
                new Task
                {
                    Id = 3,
                    Title = "Préparer la présentation",
                    Description = "Finir les slides pour la réunion de lundi",
                    IsCompleted = false,
                    DueDate = DateTime.Now.AddDays(3),
                    Tags = new List<Tag> { _tags[0], _tags[2] }
                }
            };

            // Mettre à jour les tâches associées à chaque tag
            foreach (var task in _tasks)
            {
                foreach (var tag in task.Tags)
                {
                    var actualTag = _tags.First(t => t.Id == tag.Id);
                    actualTag.Tasks.Add(task);
                }
            }
        }

        public IEnumerable<Task> GetAllTasks() => _tasks;

        public Task GetTaskById(int id) => _tasks.FirstOrDefault(t => t.Id == id);

        public IEnumerable<Task> GetTasksByTag(string tagName) =>
            _tasks.Where(t => t.Tags.Any(tag => tag.Name.Equals(tagName, StringComparison.OrdinalIgnoreCase)));

        public IEnumerable<Tag> GetAllTags() => _tags;

        public Tag GetTagById(int id) => _tags.FirstOrDefault(t => t.Id == id);

        public Task AddTask(Task task)
        {
            task.Id = _nextTaskId++;

            // Gestion des tags existants
            var newTags = new List<Tag>();
            foreach (var tag in task.Tags)
            {
                var existingTag = _tags.FirstOrDefault(t => t.Name.Equals(tag.Name, StringComparison.OrdinalIgnoreCase));
                if (existingTag != null)
                {
                    newTags.Add(existingTag);
                    existingTag.Tasks.Add(task);
                }
                else
                {
                    var newTag = new Tag
                    {
                        Id = _nextTagId++,
                        Name = tag.Name,
                        Tasks = new List<Task> { task }
                    };
                    _tags.Add(newTag);
                    newTags.Add(newTag);
                }
            }

            task.Tags = newTags;
            _tasks.Add(task);

            return task;
        }

        public Task UpdateTask(int id, Task updatedTask)
        {
            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
                return null;

            // Mise à jour des propriétés
            task.Title = updatedTask.Title;
            task.Description = updatedTask.Description;
            task.IsCompleted = updatedTask.IsCompleted;
            task.DueDate = updatedTask.DueDate;

            // Gestion des tags (simplifiée pour l'exemple)

            return task;
        }

        public bool DeleteTask(int id)
        {
            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
                return false;

            // Supprimer la tâche des tags associés
            foreach (var tag in task.Tags.ToList())
            {
                var actualTag = _tags.First(t => t.Id == tag.Id);
                actualTag.Tasks.Remove(task);
            }

            return _tasks.Remove(task);
        }
    }
}
```

5. Créez des types GraphQL pour vos modèles :

```csharp
// GraphQL/Types/TaskType.cs
using HotChocolate.Types;
using MonApiGraphQL.Models;

namespace MonApiGraphQL.GraphQL.Types
{
    public class TaskType : ObjectType<Task>
    {
        protected override void Configure(IObjectTypeDescriptor<Task> descriptor)
        {
            descriptor.Field(t => t.Id).Type<NonNullType<IntType>>();
            descriptor.Field(t => t.Title).Type<NonNullType<StringType>>();
            descriptor.Field(t => t.Description).Type<StringType>();
            descriptor.Field(t => t.IsCompleted).Type<NonNullType<BooleanType>>();
            descriptor.Field(t => t.DueDate).Type<NonNullType<DateTimeType>>();

            descriptor
                .Field(t => t.Tags)
                .Type<ListType<NonNullType<TagType>>>()
                .Name("tags");
        }
    }
}

// GraphQL/Types/TagType.cs
using HotChocolate.Types;
using MonApiGraphQL.Models;

namespace MonApiGraphQL.GraphQL.Types
{
    public class TagType : ObjectType<Tag>
    {
        protected override void Configure(IObjectTypeDescriptor<Tag> descriptor)
        {
            descriptor.Field(t => t.Id).Type<NonNullType<IntType>>();
            descriptor.Field(t => t.Name).Type<NonNullType<StringType>>();

            descriptor
                .Field(t => t.Tasks)
                .Type<ListType<NonNullType<TaskType>>>()
                .Name("tasks");
        }
    }
}
```

6. Créez des objets Query, Mutation et Subscription :

```csharp
// GraphQL/Query.cs
using HotChocolate;
using MonApiGraphQL.Models;
using MonApiGraphQL.Services;

namespace MonApiGraphQL.GraphQL
{
    public class Query
    {
        [GraphQLDescription("Récupère toutes les tâches")]
        public IEnumerable<Task> GetTasks([Service] TaskService taskService)
            => taskService.GetAllTasks();

        [GraphQLDescription("Récupère une tâche par son ID")]
        public Task GetTask([Service] TaskService taskService, int id)
            => taskService.GetTaskById(id);

        [GraphQLDescription("Récupère tous les tags")]
        public IEnumerable<Tag> GetTags([Service] TaskService taskService)
            => taskService.GetAllTags();

        [GraphQLDescription("Récupère un tag par son ID")]
        public Tag GetTag([Service] TaskService taskService, int id)
            => taskService.GetTagById(id);

        [GraphQLDescription("Récupère les tâches par nom de tag")]
        public IEnumerable<Task> GetTasksByTag([Service] TaskService taskService, string tagName)
            => taskService.GetTasksByTag(tagName);
    }
}

// GraphQL/Mutation.cs
using HotChocolate;
using MonApiGraphQL.Models;
using MonApiGraphQL.Services;

namespace MonApiGraphQL.GraphQL
{
    public class Mutation
    {
        [GraphQLDescription("Crée une nouvelle tâche")]
        public Task CreateTask(
            [Service] TaskService taskService,
            string title,
            string description,
            bool isCompleted,
            DateTime dueDate,
            string[] tagNames)
        {
            var task = new Task
            {
                Title = title,
                Description = description,
                IsCompleted = isCompleted,
                DueDate = dueDate,
                Tags = tagNames.Select(name => new Tag { Name = name }).ToList()
            };

            return taskService.AddTask(task);
        }

        [GraphQLDescription("Met à jour une tâche existante")]
        public Task UpdateTask(
            [Service] TaskService taskService,
            int id,
            string title,
            string description,
            bool isCompleted,
            DateTime dueDate)
        {
            var task = new Task
            {
                Title = title,
                Description = description,
                IsCompleted = isCompleted,
                DueDate = dueDate
            };

            return taskService.UpdateTask(id, task);
        }

        [GraphQLDescription("Marque une tâche comme terminée")]
        public Task CompleteTask([Service] TaskService taskService, int id)
        {
            var task = taskService.GetTaskById(id);
            if (task == null)
                return null;

            task.IsCompleted = true;
            return taskService.UpdateTask(id, task);
        }

        [GraphQLDescription("Supprime une tâche")]
        public bool DeleteTask([Service] TaskService taskService, int id)
            => taskService.DeleteTask(id);
    }
}
```

7. Configurez votre application pour utiliser GraphQL dans `Program.cs` :

```csharp
using MonApiGraphQL.GraphQL;
using MonApiGraphQL.GraphQL.Types;
using MonApiGraphQL.Services;

var builder = WebApplication.CreateBuilder(args);

// Enregistrer le service de tâches
builder.Services.AddSingleton<TaskService>();

// Configurer GraphQL
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddType<TaskType>()
    .AddType<TagType>();

var app = builder.Build();

app.UseHttpsRedirection();

// Activer l'endpoint GraphQL et l'interface utilisateur Voyager (pour explorer le schéma GraphQL)
app.MapGraphQL();

app.Run();
```

### Test de l'API GraphQL

Une fois votre API démarrée, vous pouvez accéder à l'interface GraphQL Playground à l'adresse `https://localhost:7042/graphql` (le port peut varier).

Voici quelques exemples de requêtes GraphQL que vous pouvez essayer :

**Récupérer toutes les tâches avec leurs tags :**

```graphql
query {
  tasks {
    id
    title
    description
    isCompleted
    dueDate
    tags {
      id
      name
    }
  }
}
```

**Récupérer une tâche spécifique :**

```graphql
query {
  task(id: 1) {
    id
    title
    description
    isCompleted
    dueDate
    tags {
      name
    }
  }
}
```

**Récupérer les tâches par tag :**

```graphql
query {
  tasksByTag(tagName: "Urgent") {
    id
    title
    isCompleted
  }
}
```

**Créer une nouvelle tâche :**

```graphql
mutation {
  createTask(
    title: "Apprendre GraphQL"
    description: "Suivre le tutoriel complet"
    isCompleted: false
    dueDate: "2023-06-30T00:00:00Z"
    tagNames: ["Apprentissage", "Travail"]
  ) {
    id
    title
    tags {
      id
      name
    }
  }
}
```

**Mettre à jour une tâche :**

```graphql
mutation {
  updateTask(
    id: 1
    title: "Apprendre GraphQL avancé"
    description: "Aller plus loin avec GraphQL"
    isCompleted: false
    dueDate: "2023-07-15T00:00:00Z"
  ) {
    id
    title
    description
    dueDate
  }
}
```

**Marquer une tâche comme terminée :**

```graphql
mutation {
  completeTask(id: 1) {
    id
    title
    isCompleted
  }
}
```

**Supprimer une tâche :**

```graphql
mutation {
  deleteTask(id: 1)
}
```

### Déploiement d'une API GraphQL sur Azure

Vous pouvez déployer votre API GraphQL sur Azure App Service de la même manière qu'une API Web standard :

1. Cliquez-droit sur le projet dans Visual Studio et sélectionnez "Publier..."
2. Suivez l'assistant pour publier sur Azure App Service
3. Une fois déployée, votre API GraphQL sera accessible à l'adresse `https://votre-app.azurewebsites.net/graphql`

### Bonnes pratiques pour GraphQL

1. **Conception du schéma** : Concevez votre schéma GraphQL avec soin, en pensant aux cas d'utilisation.
2. **Pagination** : Utilisez la pagination pour les grandes collections de données.
3. **Contrôle des requêtes** : Limitez la profondeur et la complexité des requêtes pour éviter les abus.
4. **Caching** : Mettez en place des stratégies de mise en cache efficaces.
5. **Gestion des erreurs** : Fournissez des messages d'erreur clairs et utiles.
6. **Performance** : Surveillez et optimisez les requêtes N+1 (DataLoader peut aider).
7. **Versioning** : Utilisez la dépréciation plutôt que les versions pour faire évoluer votre API.

## 18.5.4. Microservices en C#

### Qu'est-ce qu'une architecture microservices ?

L'architecture microservices est une approche de développement d'applications où une application est structurée comme une collection de services faiblement couplés. Chaque service :

- Se concentre sur une seule fonctionnalité ou un domaine d'activité
- Est déployable indépendamment
- Communique via des API bien définies
- Peut être écrit dans différents langages et utiliser différentes technologies de stockage
- Est généralement de petite taille et maintenu par une petite équipe

Comparaison avec l'architecture monolithique :

| Architecture monolithique | Architecture microservices |
|---------------------------|----------------------------|
| Une seule base de code | Plusieurs bases de code |
| Un seul processus | Plusieurs processus distribués |
| Un seul déploiement | Déploiements indépendants |
| Une seule base de données | Bases de données potentiellement différentes |
| Mise à l'échelle globale | Mise à l'échelle par service |
| Un seul point de défaillance | Des défaillances isolées |
| Développement simple au début | Complexité initiale plus élevée |
| Devient complexe avec le temps | Maintient une complexité gérable |

### Création d'une architecture microservices simple

Créons une architecture microservices simple pour un système de gestion de tâches. Ce système comprendra trois services :

1. **Service de tâches** : Gère les opérations CRUD pour les tâches
2. **Service de notification** : Envoie des notifications par e-mail
3. **API Gateway** : Point d'entrée unique pour les clients

#### 1. Service de tâches

Créez un nouveau projet API Web ASP.NET Core :

```bash
dotnet new webapi -n TaskService
cd TaskService
```

Créez un modèle de tâche et un contrôleur :

```csharp
// Models/Task.cs
namespace TaskService.Models
{
    public class Task
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }
        public string AssignedTo { get; set; }
    }
}

// Controllers/TasksController.cs
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;
using TaskService.Models;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

namespace TaskService.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class TasksController : ControllerBase
    {
        private static readonly List<Task> _tasks = new List<Task>
        {
            new Task
            {
                Id = 1,
                Title = "Compléter le rapport",
                Description = "Finaliser le rapport trimestriel",
                IsCompleted = false,
                DueDate = DateTime.Now.AddDays(2),
                AssignedTo = "alice@example.com"
            },
            new Task
            {
                Id = 2,
                Title = "Préparer la présentation",
                Description = "Créer les slides pour la réunion",
                IsCompleted = true,
                DueDate = DateTime.Now.AddDays(-1),
                AssignedTo = "bob@example.com"
            }
        };

        private readonly HttpClient _httpClient;
        private readonly IConfiguration _configuration;

        public TasksController(HttpClient httpClient, IConfiguration configuration)
        {
            _httpClient = httpClient;
            _configuration = configuration;
        }

        [HttpGet]
        public ActionResult<IEnumerable<Task>> GetAllTasks()
        {
            return Ok(_tasks);
        }

        [HttpGet("{id}")]
        public ActionResult<Task> GetTask(int id)
        {
            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
                return NotFound();

            return Ok(task);
        }

        [HttpPost]
        public async Task<ActionResult<Task>> CreateTask(Task task)
        {
            if (string.IsNullOrEmpty(task.Title))
                return BadRequest("Le titre est requis");

            // Générer un nouvel ID
            task.Id = _tasks.Count > 0 ? _tasks.Max(t => t.Id) + 1 : 1;

            _tasks.Add(task);

            // Notifier le service de notification
            try
            {
                var notification = new
                {
                    Email = task.AssignedTo,
                    Subject = $"Nouvelle tâche assignée: {task.Title}",
                    Message = $"Vous avez été assigné à une nouvelle tâche: {task.Title}. Description: {task.Description}. À compléter avant le {task.DueDate:d}."
                };

                var json = JsonSerializer.Serialize(notification);
                var content = new StringContent(json, Encoding.UTF8, "application/json");

                var notificationServiceUrl = _configuration["Services:NotificationService"];
                await _httpClient.PostAsync($"{notificationServiceUrl}/api/notifications", content);
            }
            catch (Exception ex)
            {
                // Loguer l'erreur, mais ne pas échouer la création de tâche
                Console.WriteLine($"Erreur lors de l'envoi de la notification: {ex.Message}");
            }

            return CreatedAtAction(nameof(GetTask), new { id = task.Id }, task);
        }

        [HttpPut("{id}")]
        public ActionResult<Task> UpdateTask(int id, Task updatedTask)
        {
            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
                return NotFound();

            // Mise à jour des propriétés
            task.Title = updatedTask.Title;
            task.Description = updatedTask.Description;
            task.IsCompleted = updatedTask.IsCompleted;
            task.DueDate = updatedTask.DueDate;
            task.AssignedTo = updatedTask.AssignedTo;

            return Ok(task);
        }

        [HttpDelete("{id}")]
        public ActionResult DeleteTask(int id)
        {
            var task = _tasks.FirstOrDefault(t => t.Id == id);
            if (task == null)
                return NotFound();

            _tasks.Remove(task);

            return NoContent();
        }
    }
}
```

Configurez le service dans `Program.cs` :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddHttpClient();

var app = builder.Build();

// Configurer le pipeline HTTP
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

Configurez le fichier `appsettings.json` pour définir l'URL du service de notification :

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Services": {
    "NotificationService": "https://localhost:7002"
  }
}
```

#### 2. Service de notification

Créez un nouveau projet API Web ASP.NET Core :

```bash
dotnet new webapi -n NotificationService
cd NotificationService
```

Créez un modèle de notification et un contrôleur :

```csharp
// Models/Notification.cs
namespace NotificationService.Models
{
    public class Notification
    {
        public string Email { get; set; }
        public string Subject { get; set; }
        public string Message { get; set; }
    }
}

// Services/EmailService.cs
namespace NotificationService.Services
{
    public class EmailService
    {
        private readonly ILogger<EmailService> _logger;

        public EmailService(ILogger<EmailService> logger)
        {
            _logger = logger;
        }

        public Task SendEmailAsync(string to, string subject, string message)
        {
            // Dans une application réelle, vous utiliseriez un service d'envoi d'e-mails ici
            // Comme SendGrid, Mailgun, Amazon SES, etc.
            _logger.LogInformation($"Envoi d'e-mail à {to} avec le sujet: {subject}");
            _logger.LogInformation($"Message: {message}");

            // Simuler un délai d'envoi d'e-mail
            return Task.Delay(500);
        }
    }
}

// Controllers/NotificationsController.cs
using Microsoft.AspNetCore.Mvc;
using NotificationService.Models;
using NotificationService.Services;
using System.Threading.Tasks;

namespace NotificationService.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class NotificationsController : ControllerBase
    {
        private readonly EmailService _emailService;
        private readonly ILogger<NotificationsController> _logger;

        public NotificationsController(EmailService emailService, ILogger<NotificationsController> logger)
        {
            _emailService = emailService;
            _logger = logger;
        }

        [HttpPost]
        public async Task<IActionResult> SendNotification(Notification notification)
        {
            _logger.LogInformation($"Notification reçue pour {notification.Email}");

            if (string.IsNullOrEmpty(notification.Email))
                return BadRequest("L'adresse e-mail est requise");

            if (string.IsNullOrEmpty(notification.Subject))
                return BadRequest("Le sujet est requis");

            if (string.IsNullOrEmpty(notification.Message))
                return BadRequest("Le message est requis");

            await _emailService.SendEmailAsync(notification.Email, notification.Subject, notification.Message);

            return Ok(new { message = "Notification envoyée avec succès" });
        }
    }
}
```

Configurez le service dans `Program.cs` :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddSingleton<NotificationService.Services.EmailService>();

var app = builder.Build();

// Configurer le pipeline HTTP
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

#### 3. API Gateway avec Ocelot

L'API Gateway est un point d'entrée unique pour les clients. Il achemine les requêtes vers les microservices appropriés. Ocelot est une bibliothèque .NET pour implémenter une API Gateway.

Créez un nouveau projet API Web ASP.NET Core :

```bash
dotnet new webapi -n ApiGateway
cd ApiGateway
dotnet add package Ocelot
```

Supprimez les contrôleurs générés automatiquement et créez un fichier de configuration Ocelot `ocelot.json` :

```json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/tasks/{everything}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7001
        }
      ],
      "UpstreamPathTemplate": "/api/tasks/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post", "Put", "Delete" ]
    },
    {
      "DownstreamPathTemplate": "/api/notifications/{everything}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7002
        }
      ],
      "UpstreamPathTemplate": "/api/notifications/{everything}",
      "UpstreamHttpMethod": [ "Post" ]
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:7000"
  }
}
```

Configurez le service dans `Program.cs` :

```csharp
using Ocelot.DependencyInjection;
using Ocelot.Middleware;

var builder = WebApplication.CreateBuilder(args);

// Configurer Ocelot
builder.Configuration.AddJsonFile("ocelot.json", optional: false, reloadOnChange: true);
builder.Services.AddOcelot(builder.Configuration);

var app = builder.Build();

// Utiliser Ocelot
await app.UseOcelot();

app.Run();
```

### Déploiement de microservices sur Azure

Pour déployer vos microservices sur Azure, vous avez plusieurs options, notamment Azure Container Apps, Azure Kubernetes Service (AKS) ou Azure App Service.

Voici les étapes pour déployer sur Azure App Service :

1. Publiez chaque service individuellement :
   - Cliquez-droit sur le projet dans Visual Studio et sélectionnez "Publier..."
   - Suivez l'assistant pour publier sur Azure App Service
   - Répétez pour chaque service

2. Configurez l'API Gateway pour qu'il pointe vers les services déployés :
   - Mettez à jour le fichier `ocelot.json` avec les URLs des services déployés
   - Publiez l'API Gateway

3. Configurez CORS si nécessaire pour permettre aux clients d'accéder à votre API Gateway.

### Communication entre microservices

Dans notre exemple, nous avons utilisé une communication synchrone via HTTP entre le service de tâches et le service de notification. Cependant, il existe d'autres approches pour la communication entre microservices :

1. **Communication synchrone (HTTP/gRPC)** :
   - Simple à mettre en œuvre
   - Couplage fort entre les services
   - Propagation des défaillances

2. **Communication asynchrone (Message Queue)** :
   - Découplage des services
   - Meilleure résilience
   - Peut gérer des charges plus importantes
   - Plus complexe à mettre en œuvre

### Exemple de communication asynchrone avec Azure Service Bus

Voici comment modifier notre architecture pour utiliser Azure Service Bus :

1. Créez un namespace Azure Service Bus dans le portail Azure.

2. Modifiez le service de tâches pour publier des messages au lieu d'appeler directement le service de notification :

```csharp
// TasksController.cs dans le service de tâches
// Ajoutez le package NuGet Azure.Messaging.ServiceBus

using Azure.Messaging.ServiceBus;

// ...

private readonly ServiceBusClient _serviceBusClient;
private readonly IConfiguration _configuration;

public TasksController(ServiceBusClient serviceBusClient, IConfiguration configuration)
{
    _serviceBusClient = serviceBusClient;
    _configuration = configuration;
}

[HttpPost]
public async Task<ActionResult<Task>> CreateTask(Task task)
{
    // ... (même code qu'avant)

    // Publier un message à Service Bus au lieu d'appeler directement le service de notification
    try
    {
        var notification = new
        {
            Email = task.AssignedTo,
            Subject = $"Nouvelle tâche assignée: {task.Title}",
            Message = $"Vous avez été assigné à une nouvelle tâche: {task.Title}. Description: {task.Description}. À compléter avant le {task.DueDate:d}."
        };

        var json = JsonSerializer.Serialize(notification);
        var sender = _serviceBusClient.CreateSender("notifications");
        var message = new ServiceBusMessage(Encoding.UTF8.GetBytes(json));

        await sender.SendMessageAsync(message);
    }
    catch (Exception ex)
    {
        // Loguer l'erreur
        Console.WriteLine($"Erreur lors de l'envoi du message à Service Bus: {ex.Message}");
    }

    return CreatedAtAction(nameof(GetTask), new { id = task.Id }, task);
}
```

3. Modifiez le service de notification pour traiter les messages de Service Bus :

```csharp
// Program.cs dans le service de notification
// Ajoutez le package NuGet Azure.Messaging.ServiceBus

using Azure.Messaging.ServiceBus;
using NotificationService.Services;
using NotificationService.Models;
using System.Text.Json;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Ajouter les services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddSingleton<EmailService>();

// Configurer Service Bus
var serviceBusConnectionString = builder.Configuration["Azure:ServiceBus:ConnectionString"];
builder.Services.AddSingleton(new ServiceBusClient(serviceBusConnectionString));

var app = builder.Build();

// Configurer le pipeline HTTP
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

// Configurer le traitement des messages Service Bus
var serviceBusClient = app.Services.GetRequiredService<ServiceBusClient>();
var emailService = app.Services.GetRequiredService<EmailService>();
var logger = app.Services.GetRequiredService<ILogger<Program>>();

var processor = serviceBusClient.CreateProcessor("notifications");

processor.ProcessMessageAsync += async args =>
{
    try
    {
        var body = args.Message.Body.ToString();
        logger.LogInformation($"Message reçu: {body}");

        var notification = JsonSerializer.Deserialize<Notification>(body);

        await emailService.SendEmailAsync(notification.Email, notification.Subject, notification.Message);

        await args.CompleteMessageAsync(args.Message);
    }
    catch (Exception ex)
    {
        logger.LogError($"Erreur lors du traitement du message: {ex.Message}");
        await args.AbandonMessageAsync(args.Message);
    }
};

processor.ProcessErrorAsync += args =>
{
    logger.LogError($"Erreur du processeur: {args.Exception.Message}");
    return Task.CompletedTask;
};

await processor.StartProcessingAsync();

app.Run();
```

### Bonnes pratiques pour les microservices

1. **Conception de domaine** : Organisez vos microservices autour de domaines métier.
2. **Indépendance** : Chaque service doit être indépendant et déployable séparément.
3. **Données isolées** : Chaque service devrait avoir sa propre base de données.
4. **Communication** : Préférez la communication asynchrone lorsque c'est possible.
5. **Résilience** : Implémentez des mécanismes de résilience (retry, circuit breaker, etc.).
6. **Découverte de services** : Utilisez des mécanismes de découverte de services.
7. **Monitoring** : Mettez en place un monitoring complet (logs, métriques, traces).
8. **CI/CD** : Automatisez les déploiements avec des pipelines CI/CD.
9. **Testing** : Testez chaque service individuellement et en intégration.

## 18.5.5. Event-driven architecture

### Qu'est-ce que l'architecture pilotée par les événements ?

L'architecture pilotée par les événements (Event-Driven Architecture ou EDA) est un paradigme où le flux d'exécution est déterminé par des événements. Les composants communiquent en produisant et en consommant des événements asynchrones plutôt que par des appels synchrones.

Concepts clés :

- **Événement** : Notification d'un changement d'état significatif
- **Producteur d'événements** : Service qui génère des événements
- **Consommateur d'événements** : Service qui réagit aux événements
- **Canal d'événements** : Mécanisme de transport des événements (message broker)
- **Event sourcing** : Stockage de l'historique des événements comme source de vérité

Avantages :

- **Découplage** : Les producteurs et consommateurs ne se connaissent pas directement
- **Scalabilité** : Facilite le traitement parallèle
- **Extensibilité** : Ajout facile de nouveaux consommateurs
- **Résilience** : Moins sensible aux défaillances

### Implémentation d'une architecture pilotée par les événements

Créons un exemple d'architecture pilotée par les événements pour notre système de gestion de tâches :

1. **Service de tâches** : Produit des événements comme TaskCreated, TaskUpdated, TaskCompleted
2. **Service de notification** : Consomme les événements pour envoyer des notifications
3. **Service d'analyse** : Consomme les événements pour générer des statistiques

Nous utiliserons Azure Event Grid comme broker d'événements.

#### Modèle d'événements

```csharp
// Événement de base
public abstract class IntegrationEvent
{
    public Guid Id { get; }
    public DateTime CreationDate { get; }

    protected IntegrationEvent()
    {
        Id = Guid.NewGuid();
        CreationDate = DateTime.UtcNow;
    }
}

// Événement de création de tâche
public class TaskCreatedEvent : IntegrationEvent
{
    public int TaskId { get; }
    public string Title { get; }
    public string Description { get; }
    public DateTime DueDate { get; }
    public string AssignedTo { get; }

    public TaskCreatedEvent(int taskId, string title, string description, DateTime dueDate, string assignedTo)
    {
        TaskId = taskId;
        Title = title;
        Description = description;
        DueDate = dueDate;
        AssignedTo = assignedTo;
    }
}

// Événement de mise à jour de tâche
public class TaskUpdatedEvent : IntegrationEvent
{
    public int TaskId { get; }
    public string Title { get; }
    public string Description { get; }
    public DateTime DueDate { get; }
    public string AssignedTo { get; }

    public TaskUpdatedEvent(int taskId, string title, string description, DateTime dueDate, string assignedTo)
    {
        TaskId = taskId;
        Title = title;
        Description = description;
        DueDate = dueDate;
        AssignedTo = assignedTo;
    }
}

// Événement de complétion de tâche
public class TaskCompletedEvent : IntegrationEvent
{
    public int TaskId { get; }
    public string Title { get; }
    public string AssignedTo { get; }

    public TaskCompletedEvent(int taskId, string title, string assignedTo)
    {
        TaskId = taskId;
        Title = title;
        AssignedTo = assignedTo;
    }
}
```

#### Service de publication d'événements

```csharp
public interface IEventBus
{
    Task PublishAsync<T>(T @event) where T : IntegrationEvent;
}

public class AzureEventGridEventBus : IEventBus
{
    private readonly string _topicEndpoint;
    private readonly string _topicKey;
    private readonly ILogger<AzureEventGridEventBus> _logger;

    public AzureEventGridEventBus(IConfiguration configuration, ILogger<AzureEventGridEventBus> logger)
    {
        _topicEndpoint = configuration["EventGrid:TopicEndpoint"];
        _topicKey = configuration["EventGrid:TopicKey"];
        _logger = logger;
    }

    public async Task PublishAsync<T>(T @event) where T : IntegrationEvent
    {
        _logger.LogInformation($"Publication de l'événement {@event.GetType().Name} avec ID {@event.Id}");

        // Configuration du client Event Grid
        var client = new EventGridPublisherClient(
            new Uri(_topicEndpoint),
            new AzureKeyCredential(_topicKey));

        // Création de l'événement Event Grid
        var eventGridEvent = new EventGridEvent(
            subject: $"TaskManagement/{@event.GetType().Name}",
            eventType: @event.GetType().Name,
            dataVersion: "1.0",
            data: @event);

        // Publication de l'événement
        await client.SendEventAsync(eventGridEvent);

        _logger.LogInformation($"Événement {@event.GetType().Name} publié avec succès");
    }
}
```

#### Service de tâches mis à jour

```csharp
[ApiController]
[Route("api/[controller]")]
public class TasksController : ControllerBase
{
    private static readonly List<Task> _tasks = new List<Task>();
    private readonly IEventBus _eventBus;

    public TasksController(IEventBus eventBus)
    {
        _eventBus = eventBus;
    }

    [HttpPost]
    public async Task<ActionResult<Task>> CreateTask(Task task)
    {
        // Validation et création de la tâche
        if (string.IsNullOrEmpty(task.Title))
            return BadRequest("Le titre est requis");

        task.Id = _tasks.Count > 0 ? _tasks.Max(t => t.Id) + 1 : 1;
        _tasks.Add(task);

        // Publication de l'événement TaskCreated
        var @event = new TaskCreatedEvent(
            task.Id,
            task.Title,
            task.Description,
            task.DueDate,
            task.AssignedTo);

        await _eventBus.PublishAsync(@event);

        return CreatedAtAction(nameof(GetTask), new { id = task.Id }, task);
    }

    [HttpPut("{id}/complete")]
    public async Task<ActionResult<Task>> CompleteTask(int id)
    {
        var task = _tasks.FirstOrDefault(t => t.Id == id);
        if (task == null)
            return NotFound();

        task.IsCompleted = true;

        // Publication de l'événement TaskCompleted
        var @event = new TaskCompletedEvent(
            task.Id,
            task.Title,
            task.AssignedTo);

        await _eventBus.PublishAsync(@event);

        return Ok(task);
    }

    // Autres méthodes CRUD...
}
```

#### Service de notification

```csharp
[ApiController]
[Route("api/[controller]")]
public class NotificationsController : ControllerBase
{
    private readonly EmailService _emailService;
    private readonly ILogger<NotificationsController> _logger;

    public NotificationsController(EmailService emailService, ILogger<NotificationsController> logger)
    {
        _emailService = emailService;
        _logger = logger;
    }

    [HttpPost("webhook")]
    public async Task<IActionResult> HandleEvent([FromBody] CloudEventData cloudEvent)
    {
        _logger.LogInformation($"Événement reçu: {cloudEvent.EventType}");

        try
        {
            switch (cloudEvent.EventType)
            {
                case "TaskCreatedEvent":
                    var taskCreatedEvent = JsonSerializer.Deserialize<TaskCreatedEvent>(cloudEvent.Data.ToString());
                    await HandleTaskCreatedEvent(taskCreatedEvent);
                    break;

                case "TaskCompletedEvent":
                    var taskCompletedEvent = JsonSerializer.Deserialize<TaskCompletedEvent>(cloudEvent.Data.ToString());
                    await HandleTaskCompletedEvent(taskCompletedEvent);
                    break;

                default:
                    _logger.LogInformation($"Type d'événement non géré: {cloudEvent.EventType}");
                    break;
            }

            return Ok();
        }
        catch (Exception ex)
        {
            _logger.LogError($"Erreur lors du traitement de l'événement: {ex.Message}");
            return StatusCode(500);
        }
    }

    private async Task HandleTaskCreatedEvent(TaskCreatedEvent @event)
    {
        _logger.LogInformation($"Traitement de l'événement TaskCreated pour la tâche {@event.TaskId}");

        await _emailService.SendEmailAsync(
            @event.AssignedTo,
            $"Nouvelle tâche assignée: {@event.Title}",
            $"Vous avez été assigné à une nouvelle tâche: {@event.Title}. " +
            $"Description: {@event.Description}. " +
            $"À compléter avant le {@event.DueDate:d}.");
    }

    private async Task HandleTaskCompletedEvent(TaskCompletedEvent @event)
    {
        _logger.LogInformation($"Traitement de l'événement TaskCompleted pour la tâche {@event.TaskId}");

        await _emailService.SendEmailAsync(
            @event.AssignedTo,
            $"Tâche marquée comme terminée: {@event.Title}",
            $"La tâche '{@event.Title}' a été marquée comme terminée.");
    }
}
```

#### Service d'analyse

```csharp
[ApiController]
[Route("api/[controller]")]
public class AnalyticsController : ControllerBase
{
    private static int _totalTasksCreated = 0;
    private static int _totalTasksCompleted = 0;
    private static readonly Dictionary<string, int> _tasksByUser = new Dictionary<string, int>();

    private readonly ILogger<AnalyticsController> _logger;

    public AnalyticsController(ILogger<AnalyticsController> logger)
    {
        _logger = logger;
    }

    [HttpPost("webhook")]
    public IActionResult HandleEvent([FromBody] CloudEventData cloudEvent)
    {
        _logger.LogInformation($"Événement reçu: {cloudEvent.EventType}");

        try
        {
            switch (cloudEvent.EventType)
            {
                case "TaskCreatedEvent":
                    var taskCreatedEvent = JsonSerializer.Deserialize<TaskCreatedEvent>(cloudEvent.Data.ToString());
                    HandleTaskCreatedEvent(taskCreatedEvent);
                    break;

                case "TaskCompletedEvent":
                    var taskCompletedEvent = JsonSerializer.Deserialize<TaskCompletedEvent>(cloudEvent.Data.ToString());
                    HandleTaskCompletedEvent(taskCompletedEvent);
                    break;

                default:
                    _logger.LogInformation($"Type d'événement non géré: {cloudEvent.EventType}");
                    break;
            }

            return Ok();
        }
        catch (Exception ex)
        {
            _logger.LogError($"Erreur lors du traitement de l'événement: {ex.Message}");
            return StatusCode(500);
        }
    }

    private void HandleTaskCreatedEvent(TaskCreatedEvent @event)
    {
        _logger.LogInformation($"Mise à jour des statistiques pour la création de tâche {@event.TaskId}");

        _totalTasksCreated++;

        if (_tasksByUser.ContainsKey(@event.AssignedTo))
        {
            _tasksByUser[@event.AssignedTo]++;
        }
        else
        {
            _tasksByUser[@event.AssignedTo] = 1;
        }
    }

    private void HandleTaskCompletedEvent(TaskCompletedEvent @event)
    {
        _logger.LogInformation($"Mise à jour des statistiques pour la complétion de tâche {@event.TaskId}");

        _totalTasksCompleted++;
    }

    [HttpGet("statistics")]
    public IActionResult GetStatistics()
    {
        var statistics = new
        {
            TotalTasksCreated = _totalTasksCreated,
            TotalTasksCompleted = _totalTasksCompleted,
            CompletionRate = _totalTasksCreated > 0 ? (double)_totalTasksCompleted / _totalTasksCreated : 0,
            TasksByUser = _tasksByUser
        };

        return Ok(statistics);
    }
}
```

### Configuration d'Azure Event Grid

Pour configurer Azure Event Grid :

1. Dans le portail Azure, créez une Topic Event Grid :
   - Allez dans "Create a resource" > recherchez "Event Grid Topic" > cliquez sur "Create"
   - Remplissez les détails et créez la ressource

2. Configurez les abonnements aux événements :
   - Dans votre topic Event Grid, allez dans "Event Subscriptions" > "Add Event Subscription"
   - Nom : TaskCreatedSubscription
   - Types d'événements : TaskCreatedEvent
   - Endpoint type : Webhook
   - Endpoint : URL de votre service de notification (par exemple, https://votreservice.azurewebsites.net/api/notifications/webhook)
   - Répétez pour d'autres événements et services

3. Configurez vos services pour publier et recevoir des événements :
   - Ajoutez les paramètres de connexion à Event Grid dans les fichiers de configuration
   - Assurez-vous que vos webhooks sont accessibles publiquement

### Event Sourcing et CQRS

Dans une architecture pilotée par les événements avancée, vous pourriez également utiliser :

1. **Event Sourcing** : Stockage de toutes les modifications d'état sous forme d'événements, qui sont la source de vérité.

2. **CQRS (Command Query Responsibility Segregation)** : Séparation des opérations de lecture (queries) et d'écriture (commands).

Voici un exemple simplifié d'Event Sourcing pour notre service de tâches :

```csharp
// Modèle de données
public class Task
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public bool IsCompleted { get; set; }
    public DateTime DueDate { get; set; }
    public string AssignedTo { get; set; }

    // Méthode pour créer une tâche à partir d'une séquence d'événements
    public static Task FromEvents(IEnumerable<IntegrationEvent> events)
    {
        var task = new Task();

        foreach (var @event in events)
        {
            task.Apply(@event);
        }

        return task;
    }

    // Application d'un événement à l'état de la tâche
    private void Apply(IntegrationEvent @event)
    {
        switch (@event)
        {
            case TaskCreatedEvent created:
                Id = created.TaskId;
                Title = created.Title;
                Description = created.Description;
                DueDate = created.DueDate;
                AssignedTo = created.AssignedTo;
                IsCompleted = false;
                break;

            case TaskUpdatedEvent updated:
                Title = updated.Title;
                Description = updated.Description;
                DueDate = updated.DueDate;
                AssignedTo = updated.AssignedTo;
                break;

            case TaskCompletedEvent completed:
                IsCompleted = true;
                break;
        }
    }
}

// Service d'événements
public class EventStore
{
    private readonly Dictionary<int, List<IntegrationEvent>> _events = new Dictionary<int, List<IntegrationEvent>>();
    private readonly IEventBus _eventBus;

    public EventStore(IEventBus eventBus)
    {
        _eventBus = eventBus;
    }

    public async Task SaveEventAsync<T>(int aggregateId, T @event) where T : IntegrationEvent
    {
        if (!_events.ContainsKey(aggregateId))
        {
            _events[aggregateId] = new List<IntegrationEvent>();
        }

        _events[aggregateId].Add(@event);

        // Publication de l'événement sur le bus
        await _eventBus.PublishAsync(@event);
    }

    public List<IntegrationEvent> GetEvents(int aggregateId)
    {
        if (!_events.ContainsKey(aggregateId))
        {
            return new List<IntegrationEvent>();
        }

        return _events[aggregateId];
    }
}

// Service de tâches avec Event Sourcing
public class TaskService
{
    private readonly EventStore _eventStore;

    public TaskService(EventStore eventStore)
    {
        _eventStore = eventStore;
    }

    public async Task<int> CreateTaskAsync(string title, string description, DateTime dueDate, string assignedTo)
    {
        // Générer un nouvel ID
        var taskId = Guid.NewGuid().GetHashCode() & 0x7FFFFFFF; // Entier positif

        // Créer l'événement
        var @event = new TaskCreatedEvent(taskId, title, description, dueDate, assignedTo);

        // Sauvegarder l'événement
        await _eventStore.SaveEventAsync(taskId, @event);

        return taskId;
    }

    public async Task UpdateTaskAsync(int taskId, string title, string description, DateTime dueDate, string assignedTo)
    {
        // Vérifier que la tâche existe
        var events = _eventStore.GetEvents(taskId);
        if (!events.Any())
        {
            throw new KeyNotFoundException($"Tâche avec ID {taskId} non trouvée");
        }

        // Créer l'événement
        var @event = new TaskUpdatedEvent(taskId, title, description, dueDate, assignedTo);

        // Sauvegarder l'événement
        await _eventStore.SaveEventAsync(taskId, @event);
    }

    public async Task CompleteTaskAsync(int taskId)
    {
        // Vérifier que la tâche existe
        var events = _eventStore.GetEvents(taskId);
        if (!events.Any())
        {
            throw new KeyNotFoundException($"Tâche avec ID {taskId} non trouvée");
        }

        // Réconstruire l'état actuel
        var task = Task.FromEvents(events);

        // Créer l'événement
        var @event = new TaskCompletedEvent(taskId, task.Title, task.AssignedTo);

        // Sauvegarder l'événement
        await _eventStore.SaveEventAsync(taskId, @event);
    }

    public Task GetTaskAsync(int taskId)
    {
        var events = _eventStore.GetEvents(taskId);
        if (!events.Any())
        {
            return null;
        }

        return Task.FromEvents(events);
    }
}
```

### Intégration avec CQRS

CQRS (Command Query Responsibility Segregation) est un pattern qui sépare les opérations de lecture (queries) des opérations d'écriture (commands). Cela permet d'optimiser chaque côté indépendamment.

Voici comment nous pourrions étendre notre exemple avec CQRS :

```csharp
// Commandes (Côté écriture)
public class CreateTaskCommand
{
    public string Title { get; set; }
    public string Description { get; set; }
    public DateTime DueDate { get; set; }
    public string AssignedTo { get; set; }
}

public class UpdateTaskCommand
{
    public int TaskId { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public DateTime DueDate { get; set; }
    public string AssignedTo { get; set; }
}

public class CompleteTaskCommand
{
    public int TaskId { get; set; }
}

// Gestionnaires de commandes
public class TaskCommandHandler
{
    private readonly TaskService _taskService;

    public TaskCommandHandler(TaskService taskService)
    {
        _taskService = taskService;
    }

    public async Task<int> HandleAsync(CreateTaskCommand command)
    {
        return await _taskService.CreateTaskAsync(
            command.Title,
            command.Description,
            command.DueDate,
            command.AssignedTo);
    }

    public async Task HandleAsync(UpdateTaskCommand command)
    {
        await _taskService.UpdateTaskAsync(
            command.TaskId,
            command.Title,
            command.Description,
            command.DueDate,
            command.AssignedTo);
    }

    public async Task HandleAsync(CompleteTaskCommand command)
    {
        await _taskService.CompleteTaskAsync(command.TaskId);
    }
}

// Requêtes (Côté lecture)
public class GetTaskQuery
{
    public int TaskId { get; set; }
}

public class GetTasksByUserQuery
{
    public string AssignedTo { get; set; }
}

public class GetTasksCountQuery
{
}

// Modèle de lecture
public class TaskReadModel
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public bool IsCompleted { get; set; }
    public DateTime DueDate { get; set; }
    public string AssignedTo { get; set; }

    // Créer à partir du modèle de domaine
    public static TaskReadModel FromDomainModel(Task task)
    {
        return new TaskReadModel
        {
            Id = task.Id,
            Title = task.Title,
            Description = task.Description,
            IsCompleted = task.IsCompleted,
            DueDate = task.DueDate,
            AssignedTo = task.AssignedTo
        };
    }
}

// Gestionnaire de requêtes
public class TaskQueryHandler
{
    private readonly TaskService _taskService;
    private readonly Dictionary<string, List<int>> _tasksByUser;
    private int _tasksCount;

    public TaskQueryHandler(TaskService taskService)
    {
        _taskService = taskService;
        _tasksByUser = new Dictionary<string, List<int>>();
        _tasksCount = 0;

        // Dans une application réelle, vous auriez une base de données séparée pour la lecture
        // et vous maintiendriez cette base de données à jour en réagissant aux événements
    }

    public async Task<TaskReadModel> HandleAsync(GetTaskQuery query)
    {
        var task = await _taskService.GetTaskAsync(query.TaskId);
        if (task == null)
            return null;

        return TaskReadModel.FromDomainModel(task);
    }

    public async Task<List<TaskReadModel>> HandleAsync(GetTasksByUserQuery query)
    {
        // Dans une implémentation réelle, ceci viendrait d'une base de données optimisée pour la lecture
        if (!_tasksByUser.ContainsKey(query.AssignedTo))
            return new List<TaskReadModel>();

        var result = new List<TaskReadModel>();

        foreach (var taskId in _tasksByUser[query.AssignedTo])
        {
            var task = await _taskService.GetTaskAsync(taskId);
            if (task != null)
            {
                result.Add(TaskReadModel.FromDomainModel(task));
            }
        }

        return result;
    }

    public Task<int> HandleAsync(GetTasksCountQuery query)
    {
        // Dans une implémentation réelle, ceci viendrait d'une base de données optimisée pour la lecture
        return Task.FromResult(_tasksCount);
    }

    // Méthodes pour maintenir la vue de lecture à jour en réagissant aux événements
    public void Handle(TaskCreatedEvent @event)
    {
        _tasksCount++;

        if (!_tasksByUser.ContainsKey(@event.AssignedTo))
        {
            _tasksByUser[@event.AssignedTo] = new List<int>();
        }

        _tasksByUser[@event.AssignedTo].Add(@event.TaskId);
    }

    public void Handle(TaskUpdatedEvent @event)
    {
        // Mise à jour des données de lecture si nécessaire
    }

    public void Handle(TaskCompletedEvent @event)
    {
        // Mise à jour des données de lecture si nécessaire
    }
}
```

Avec cette architecture CQRS, nous avons:

1. Un côté commande qui modifie l'état du système à travers des commandes et génère des événements
2. Un côté requête qui lit l'état du système optimisé pour les différents cas d'utilisation
3. Une synchronisation entre les deux côtés via les événements

### Avantages de l'Event Sourcing et CQRS

1. **Traçabilité** : Historique complet de toutes les modifications
2. **Auditabilité** : Facile de savoir qui a fait quoi et quand
3. **Résilience** : Possibilité de reconstruire l'état à partir des événements
4. **Performances** : Optimisation séparée pour la lecture et l'écriture
5. **Flexibilité** : Possibilité d'ajouter de nouvelles vues sans modifier le modèle de domaine

### Défis et considérations

1. **Complexité** : Architecture plus complexe à mettre en œuvre
2. **Cohérence éventuelle** : Délai possible entre l'écriture et la mise à jour des vues de lecture
3. **Volume de données** : Stockage de tous les événements peut devenir volumineux
4. **Versionnement** : Gestion des évolutions de schéma d'événements

### Outils et frameworks pour l'Event Sourcing et CQRS en .NET

Plusieurs frameworks .NET peuvent vous aider à implémenter l'Event Sourcing et CQRS :

1. **EventStore** : Base de données dédiée à l'Event Sourcing
2. **NEventStore** : Solution d'Event Sourcing pour .NET
3. **MediatR** : Implémentation du pattern Mediator pour la gestion des requêtes et commandes
4. **Akka.NET** : Framework d'acteurs pour les systèmes distribués
5. **MassTransit** : Service Bus pour la communication entre services

### Conclusion

Dans ce chapitre, nous avons exploré cinq approches d'intégration cloud et de services modernes avec C# :

1. **Azure Functions** : Services serverless pour exécuter du code en réponse à des événements
2. **gRPC** : Framework RPC haute performance pour la communication entre services
3. **GraphQL** : Langage de requête flexible pour les API
4. **Microservices** : Architecture d'applications comme une collection de services faiblement couplés
5. **Architecture pilotée par les événements** : Paradigme où le flux est déterminé par des événements

Chacune de ces technologies a ses propres avantages et cas d'utilisation :

- **Azure Functions** est idéal pour des traitements événementiels légers et le pay-per-use.
- **gRPC** est excellent pour la communication haute performance entre services internes.
- **GraphQL** brille pour les API publiques qui desservent différents clients avec des besoins variés.
- **Microservices** permettent une évolution indépendante et une meilleure scalabilité.
- **L'architecture pilotée par les événements** favorise le découplage et l'extensibilité.

Ces technologies peuvent être combinées pour créer des systèmes robustes, évolutifs et maintenables. Par exemple, vous pourriez avoir :

- Des microservices qui communiquent via gRPC pour les interactions synchrones
- Une architecture pilotée par les événements pour la communication asynchrone
- Des Azure Functions pour traiter des événements spécifiques
- Une API GraphQL comme façade pour les clients

### Bonnes pratiques générales

1. **Choisissez la bonne approche** pour votre cas d'utilisation spécifique
2. **Commencez simple** et évoluez en fonction de vos besoins
3. **Documentez** vos choix architecturaux et vos interfaces
4. **Testez** abondamment, en particulier les interactions entre services
5. **Monitorez** votre système en production
6. **Automatisez** les déploiements via des pipelines CI/CD
7. **Sécurisez** toutes les communications entre services

### Exercices pratiques

1. **Créez une Azure Function** qui se déclenche lorsqu'un fichier est téléchargé dans Blob Storage et génère une image redimensionnée.

2. **Implémentez un service gRPC** pour une fonctionnalité de votre choix et créez un client pour le tester.

3. **Créez une API GraphQL** qui expose des données d'une base de données existante.

4. **Transformez une application monolithique** simple en une architecture microservices.

5. **Implémentez une architecture Event-Driven** pour un cas d'utilisation comme un système de commandes en ligne.

### Ressources supplémentaires

#### Azure Functions
- [Documentation officielle Azure Functions](https://docs.microsoft.com/fr-fr/azure/azure-functions/)
- [Azure Functions University](https://github.com/marcduiker/azure-functions-university)

#### gRPC
- [Documentation officielle gRPC](https://grpc.io/docs/)
- [gRPC pour ASP.NET Core](https://docs.microsoft.com/fr-fr/aspnet/core/grpc/)

#### GraphQL
- [Site officiel GraphQL](https://graphql.org/)
- [Hot Chocolate Documentation](https://chillicream.com/docs/hotchocolate/)

#### Microservices
- [Microservices .NET - eShopOnContainers](https://github.com/dotnet-architecture/eShopOnContainers)
- [Designing Microservices with .NET Core](https://www.packtpub.com/product/designing-microservices-with-net-core-second-edition/9781788473064)

#### Event-Driven Architecture
- [CQRS Journey by Microsoft](https://docs.microsoft.com/fr-fr/previous-versions/msp-n-p/jj554200(v=pandp.10))
- [Event Sourcing Basics](https://eventstore.com/event-sourcing/)

### Fin du chapitre

Ce chapitre vous a donné une introduction complète aux technologies modernes d'intégration cloud et de services avec C#. Ces connaissances vous permettront de créer des applications distribuées, évolutives et résilientes qui peuvent s'adapter aux besoins changeants de votre entreprise.

Rappelez-vous que l'architecture logicielle est un domaine en constante évolution, et il est important de rester à jour avec les dernières tendances et meilleures pratiques. Continuez à apprendre, à expérimenter, et à affiner vos compétences pour devenir un développeur C# cloud-native accompli.

⏭️
