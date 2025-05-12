# 18.4. Intégration avec JavaScript/TypeScript

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

Bienvenue dans ce chapitre consacré à l'intégration de C# avec JavaScript et TypeScript. Dans le monde du développement moderne, il est rare qu'une application soit construite avec un seul langage ou une seule technologie. La combinaison de C# côté serveur avec JavaScript/TypeScript côté client est l'une des architectures les plus populaires pour les applications web et mobiles.

Pourquoi voudriez-vous intégrer C# avec JavaScript/TypeScript ? Voici quelques raisons :

- Tirer parti de la richesse de l'écosystème JavaScript pour les interfaces utilisateur
- Créer des applications web interactives avec une logique métier robuste en C#
- Développer des applications en temps réel nécessitant une communication bidirectionnelle
- Bénéficier de la sécurité de type de TypeScript tout en conservant l'infrastructure .NET
- Réutiliser vos compétences et votre code C# existants dans des applications web modernes

Dans ce chapitre, nous allons explorer cinq approches principales pour l'intégration de C# avec JavaScript/TypeScript :

1. **Blazor WebAssembly** - Exécuter du code C# directement dans le navigateur
2. **SignalR** - Communication en temps réel entre clients et serveur
3. **APIs REST et JSON** - Communication classique client-serveur
4. **gRPC-Web** - Communication efficace entre client et serveur avec protobuf
5. **JSInterop** - Appeler JavaScript depuis C# et inversement dans Blazor

Commençons notre exploration de ces technologies passionnantes !

## 18.4.1. Blazor WebAssembly

### Qu'est-ce que Blazor WebAssembly ?

Blazor WebAssembly (WASM) est un framework qui permet d'exécuter du code C# directement dans le navigateur web, sans plugin. Comment est-ce possible ? Grâce à WebAssembly, un format de bytecode qui peut s'exécuter à des vitesses proches du natif dans tous les navigateurs modernes.

Avec Blazor WebAssembly, vous pouvez :
- Écrire toute votre application (frontend et backend) en C#
- Réutiliser vos compétences .NET pour le développement web
- Partager du code et des bibliothèques entre le client et le serveur
- Exploiter l'écosystème .NET complet pour le développement frontend

### Installation et configuration

Pour commencer avec Blazor WebAssembly, vous aurez besoin de :

1. Le SDK .NET (version 6.0 ou supérieure)
2. Un IDE comme Visual Studio 2022 ou Visual Studio Code avec l'extension C#

Voici comment créer votre première application Blazor WebAssembly :

**Avec l'interface en ligne de commande (CLI) :**

```bash
# Créer une nouvelle application Blazor WebAssembly
dotnet new blazorwasm -o MonAppBlazor

# Ouvrir le répertoire du projet
cd MonAppBlazor

# Exécuter l'application
dotnet run
```

**Avec Visual Studio :**

1. Créez un nouveau projet
2. Sélectionnez "Application Blazor" comme type de projet
3. Choisissez "Blazor WebAssembly App" comme modèle
4. Configurez votre projet (nom, emplacement, etc.)
5. Cliquez sur Créer

### Structure d'un projet Blazor WebAssembly

Une fois votre projet créé, vous verrez une structure de fichiers similaire à celle-ci :

```
MonAppBlazor/
│
├── Pages/                  # Contient les composants Razor qui sont routables
│   ├── Counter.razor       # Exemple de composant avec un compteur
│   ├── FetchData.razor     # Exemple de composant qui récupère des données
│   └── Index.razor         # Page d'accueil
│
├── Shared/                 # Composants partagés entre plusieurs pages
│   ├── MainLayout.razor    # Mise en page principale de l'application
│   └── NavMenu.razor       # Menu de navigation
│
├── wwwroot/                # Ressources web statiques (HTML, CSS, JS, images)
│   ├── css/                # Fichiers CSS
│   ├── index.html          # Page HTML principale chargée par le navigateur
│   └── favicon.ico         # Icône du site
│
├── App.razor               # Composant racine de l'application
├── Program.cs              # Point d'entrée qui configure et démarre l'application
└── _Imports.razor          # Importations de namespaces communs pour tous les fichiers .razor
```

### Exemple de composant Blazor

Voici un exemple simple de composant Blazor qui montre un compteur interactif :

```razor
@page "/compteur"

<h1>Compteur</h1>

<p>Valeur actuelle : @compteurActuel</p>

<button class="btn btn-primary" @onclick="Incrementer">Incrémenter</button>

@code {
    private int compteurActuel = 0;

    private void Incrementer()
    {
        compteurActuel++;
    }
}
```

Ce composant :
- Définit une route `/compteur` avec la directive `@page`
- Affiche la valeur actuelle du compteur
- Fournit un bouton qui, lorsqu'on clique dessus, incrémente le compteur
- Contient un bloc `@code` où la logique C# est définie

### Appel d'une API Web depuis Blazor

L'un des avantages de Blazor est la facilité d'appel des API Web. Voici comment appeler une API REST :

```razor
@page "/meteo"
@using System.Net.Http.Json
@inject HttpClient Http

<h1>Prévisions météo</h1>

@if (previsions == null)
{
    <p><em>Chargement...</em></p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Temp. (C)</th>
                <th>Temp. (F)</th>
                <th>Résumé</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var prev in previsions)
            {
                <tr>
                    <td>@prev.Date.ToShortDateString()</td>
                    <td>@prev.TemperatureC</td>
                    <td>@prev.TemperatureF</td>
                    <td>@prev.Summary</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private WeatherForecast[]? previsions;

    protected override async Task OnInitializedAsync()
    {
        try
        {
            previsions = await Http.GetFromJsonAsync<WeatherForecast[]>("api/WeatherForecast");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur : {ex.Message}");
        }
    }

    public class WeatherForecast
    {
        public DateTime Date { get; set; }
        public int TemperatureC { get; set; }
        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
        public string? Summary { get; set; }
    }
}
```

### Modèle d'hébergement

Blazor WebAssembly peut être hébergé de différentes manières :

1. **Application autonome** : Le client Blazor est servi comme contenu statique sans backend .NET

2. **Application hébergée par ASP.NET Core** : Une solution comprenant :
   - Un projet client Blazor WebAssembly
   - Un projet serveur ASP.NET Core qui sert l'application et fournit des API
   - Un projet partagé pour le code et les modèles communs

3. **Progressive Web App (PWA)** : Une application Blazor qui peut être installée comme une application native et fonctionner hors ligne

### Avantages et inconvénients de Blazor WebAssembly

**Avantages :**
- Un seul langage (C#) pour le frontend et le backend
- Sécurité de type et refactoring facilité
- Partage de code et de logique entre client et serveur
- Excellent pour les équipes déjà familières avec .NET

**Inconvénients :**
- Taille du téléchargement initial (l'environnement d'exécution .NET doit être téléchargé)
- Performances de démarrage plus lentes que JavaScript pur
- Moins mature que les frameworks JavaScript établis
- Accès limité à certaines API du navigateur (bien que cela s'améliore)

## 18.4.2. SignalR

### Qu'est-ce que SignalR ?

ASP.NET Core SignalR est une bibliothèque qui simplifie l'ajout de fonctionnalités en temps réel à vos applications web. Contrairement au modèle classique requête/réponse, SignalR permet une communication bidirectionnelle entre le serveur et les clients.

SignalR est idéal pour les applications qui nécessitent des mises à jour fréquentes, comme :
- Chat et messagerie instantanée
- Tableaux de bord et monitoring en temps réel
- Jeux multijoueurs
- Applications collaboratives
- Notifications et alertes en direct

### Comment fonctionne SignalR

SignalR gère automatiquement la connexion en utilisant la meilleure technique disponible :
1. WebSockets (protocole préféré)
2. Server-Sent Events
3. Long Polling

Vous n'avez pas à vous soucier de ces détails techniques - SignalR s'adapte automatiquement.

### Mise en place de SignalR côté serveur

Pour commencer avec SignalR, vous devez d'abord configurer le serveur :

**1. Créez un nouveau projet ASP.NET Core :**

```bash
dotnet new web -o SignalRChat
cd SignalRChat
```

**2. Installez le package SignalR :**

```bash
dotnet add package Microsoft.AspNetCore.SignalR
```

**3. Créez un hub SignalR :**

Un hub est une classe qui sert de point central pour gérer les connexions clients et distribuer les messages.

```csharp
// ChatHub.cs
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace SignalRChat
{
    public class ChatHub : Hub
    {
        public async Task EnvoyerMessage(string utilisateur, string message)
        {
            // Envoie le message à tous les clients connectés
            await Clients.All.SendAsync("ReceiveMessage", utilisateur, message);
        }

        public override async Task OnConnectedAsync()
        {
            await Clients.All.SendAsync("UserConnected", Context.ConnectionId);
            await base.OnConnectedAsync();
        }

        public override async Task OnDisconnectedAsync(Exception exception)
        {
            await Clients.All.SendAsync("UserDisconnected", Context.ConnectionId);
            await base.OnDisconnectedAsync(exception);
        }
    }
}
```

**4. Configurez SignalR dans `Program.cs` :**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services SignalR
builder.Services.AddSignalR();

var app = builder.Build();

// Configure le pipeline de requêtes HTTP
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseStaticFiles();
app.UseRouting();

// Configure les endpoints SignalR
app.MapHub<ChatHub>("/chatHub");

app.Run();
```

### Mise en place de SignalR côté client (JavaScript)

Une fois le serveur configuré, vous devez mettre en place le client JavaScript :

**1. Créez un fichier HTML et JavaScript dans le dossier wwwroot :**

```html
<!-- wwwroot/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Chat SignalR</title>
</head>
<body>
    <div class="container">
        <div class="row">
            <div class="col-6">
                <input type="text" id="userInput" placeholder="Votre nom" />
                <input type="text" id="messageInput" placeholder="Votre message" />
                <button id="sendButton">Envoyer</button>
            </div>
        </div>
        <div class="row">
            <div class="col-12">
                <hr />
            </div>
        </div>
        <div class="row">
            <div class="col-12">
                <ul id="messagesList"></ul>
            </div>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/6.0.1/signalr.min.js"></script>
    <script src="js/chat.js"></script>
</body>
</html>
```

```javascript
// wwwroot/js/chat.js
"use strict";

// Crée la connexion
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .configureLogging(signalR.LogLevel.Information)
    .build();

// Récupère les références aux éléments DOM
const userInput = document.getElementById("userInput");
const messageInput = document.getElementById("messageInput");
const sendButton = document.getElementById("sendButton");
const messagesList = document.getElementById("messagesList");

// Désactive le bouton d'envoi jusqu'à ce que la connexion soit établie
sendButton.disabled = true;

// Gestionnaire d'événement pour recevoir des messages
connection.on("ReceiveMessage", (user, message) => {
    // Crée un nouvel élément de liste pour le message
    const li = document.createElement("li");
    li.textContent = `${user}: ${message}`;
    messagesList.appendChild(li);
});

// Gestionnaire d'événement pour les connexions d'utilisateurs
connection.on("UserConnected", (connectionId) => {
    const li = document.createElement("li");
    li.textContent = `Utilisateur connecté: ${connectionId}`;
    li.style.color = "green";
    messagesList.appendChild(li);
});

// Gestionnaire d'événement pour les déconnexions d'utilisateurs
connection.on("UserDisconnected", (connectionId) => {
    const li = document.createElement("li");
    li.textContent = `Utilisateur déconnecté: ${connectionId}`;
    li.style.color = "red";
    messagesList.appendChild(li);
});

// Démarrage de la connexion
connection.start()
    .then(() => {
        console.log("Connexion SignalR établie");
        sendButton.disabled = false;
    })
    .catch((err) => {
        console.error(err.toString());
    });

// Envoi de message lorsque le bouton est cliqué
sendButton.addEventListener("click", (event) => {
    // Appelle la méthode "EnvoyerMessage" sur le hub
    connection.invoke("EnvoyerMessage", userInput.value, messageInput.value)
        .catch((err) => {
            console.error(err.toString());
        });

    // Réinitialise le champ de message
    messageInput.value = "";
    event.preventDefault();
});
```

### Utilisation de SignalR avec TypeScript

Pour une meilleure expérience de développement, vous pouvez utiliser TypeScript avec SignalR :

**1. Installer TypeScript :**

```bash
npm init -y
npm install typescript @microsoft/signalr @types/node --save-dev
```

**2. Créer un fichier tsconfig.json :**

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "es2015",
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "wwwroot/js",
    "esModuleInterop": true,
    "lib": ["es2015", "dom"]
  },
  "include": ["scripts/**/*"]
}
```

**3. Créer un fichier TypeScript pour le client :**

```typescript
// scripts/chat.ts
import * as signalR from "@microsoft/signalr";

// Type pour les messages
interface Message {
    user: string;
    text: string;
    timestamp: Date;
}

// Crée la connexion
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .configureLogging(signalR.LogLevel.Information)
    .withAutomaticReconnect()
    .build();

// Récupère les références aux éléments DOM
const userInput = document.getElementById("userInput") as HTMLInputElement;
const messageInput = document.getElementById("messageInput") as HTMLInputElement;
const sendButton = document.getElementById("sendButton") as HTMLButtonElement;
const messagesList = document.getElementById("messagesList") as HTMLUListElement;

// Désactive le bouton d'envoi jusqu'à ce que la connexion soit établie
sendButton.disabled = true;

// Gestionnaire d'événement pour recevoir des messages
connection.on("ReceiveMessage", (user: string, message: string) => {
    addMessageToList(user, message);
});

// Fonction pour ajouter un message à la liste
function addMessageToList(user: string, text: string): void {
    const message: Message = {
        user,
        text,
        timestamp: new Date()
    };

    const li = document.createElement("li");
    li.textContent = `${message.timestamp.toLocaleTimeString()} - ${message.user}: ${message.text}`;
    messagesList.appendChild(li);
}

// Démarrage de la connexion
async function startConnection(): Promise<void> {
    try {
        await connection.start();
        console.log("Connexion SignalR établie");
        sendButton.disabled = false;
    } catch (err) {
        console.error(err);
        setTimeout(startConnection, 5000);
    }
}

// Envoi de message lorsque le bouton est cliqué
sendButton.addEventListener("click", async (event) => {
    const user = userInput.value;
    const message = messageInput.value;

    if (user && message) {
        try {
            // Appelle la méthode "EnvoyerMessage" sur le hub
            await connection.invoke("EnvoyerMessage", user, message);
            // Réinitialise le champ de message
            messageInput.value = "";
        } catch (err) {
            console.error(err);
        }
    }

    event.preventDefault();
});

// Gérer la connexion et les reconnexions
connection.onclose(async () => {
    console.log("Connexion perdue. Tentative de reconnexion...");
    await startConnection();
});

// Démarrer la connexion
startConnection();
```

### Fonctionnalités avancées de SignalR

SignalR offre de nombreuses fonctionnalités avancées :

**Groupes** - Pour envoyer des messages à des sous-ensembles de clients :

```csharp
// Côté serveur
public async Task RejoindreSalle(string salle)
{
    await Groups.AddToGroupAsync(Context.ConnectionId, salle);
    await Clients.Group(salle).SendAsync("SalleJointe", Context.ConnectionId, salle);
}

public async Task EnvoyerMessageSalle(string salle, string utilisateur, string message)
{
    await Clients.Group(salle).SendAsync("ReceiveMessage", utilisateur, message);
}
```

**Utilisateurs spécifiques** - Pour envoyer des messages à des utilisateurs particuliers :

```csharp
// Côté serveur
public async Task EnvoyerMessagePrive(string destinataireId, string utilisateur, string message)
{
    await Clients.User(destinataireId).SendAsync("ReceivePrivateMessage", utilisateur, message);
}
```

**Streaming** - Pour transmettre de grandes quantités de données en temps réel :

```csharp
// Côté serveur
public async IAsyncEnumerable<int> EmettreDonnees(int compteur, int delai, [EnumeratorCancellation] CancellationToken cancellationToken)
{
    for (var i = 0; i < compteur; i++)
    {
        // Vérifie si le client a annulé le stream
        cancellationToken.ThrowIfCancellationRequested();

        // Émet la valeur
        yield return i;

        // Simule un traitement
        await Task.Delay(delai, cancellationToken);
    }
}
```

```javascript
// Côté client
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/streamHub")
    .build();

connection.start().then(() => {
    connection.stream("EmettreDonnees", 10, 500)
        .subscribe({
            next: (item) => {
                console.log(item);
            },
            complete: () => {
                console.log("Stream terminé");
            },
            error: (err) => {
                console.error(err);
            },
        });
});
```

## 18.4.3. APIs REST et JSON

Les APIs REST (Representational State Transfer) sont le moyen le plus courant d'intégrer des applications C# avec JavaScript/TypeScript. Elles permettent aux applications frontales écrites en JavaScript de communiquer avec des backends C# via HTTP.

### Principes des APIs REST

Une API REST bien conçue respecte ces principes :

1. **Architecture client-serveur** - Séparation des préoccupations
2. **Sans état** - Chaque requête contient toutes les informations nécessaires
3. **Mise en cache** - Les réponses indiquent si elles peuvent être mises en cache
4. **Interface uniforme** - Identification des ressources, manipulation via représentations, messages auto-descriptifs, HATEOAS
5. **Système en couches** - Chaque composant ne voit que la couche avec laquelle il interagit

### Création d'une API REST avec ASP.NET Core

Voici comment créer une API REST simple avec ASP.NET Core :

**1. Créez un nouveau projet API :**

```bash
dotnet new webapi -o MonApiRest
cd MonApiRest
```

**2. Définissez vos modèles de données :**

```csharp
// Models/Produit.cs
namespace MonApiRest.Models
{
    public class Produit
    {
        public int Id { get; set; }
        public string Nom { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public decimal Prix { get; set; }
        public bool EnStock { get; set; }
    }
}
```

**3. Créez un contrôleur :**

```csharp
// Controllers/ProduitsController.cs
using Microsoft.AspNetCore.Mvc;
using MonApiRest.Models;
using System.Collections.Generic;
using System.Linq;

namespace MonApiRest.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProduitsController : ControllerBase
    {
        private static List<Produit> _produits = new List<Produit>
        {
            new Produit { Id = 1, Nom = "Ordinateur portable", Description = "Ordinateur portable haute performance", Prix = 999.99m, EnStock = true },
            new Produit { Id = 2, Nom = "Smartphone", Description = "Smartphone dernière génération", Prix = 699.99m, EnStock = true },
            new Produit { Id = 3, Nom = "Casque audio", Description = "Casque audio sans fil", Prix = 149.99m, EnStock = false }
        };

        // GET: api/produits
        [HttpGet]
        public ActionResult<IEnumerable<Produit>> ObtenirTousProduits()
        {
            return Ok(_produits);
        }

        // GET: api/produits/5
        [HttpGet("{id}")]
        public ActionResult<Produit> ObtenirProduit(int id)
        {
            var produit = _produits.FirstOrDefault(p => p.Id == id);
            if (produit == null)
            {
                return NotFound();
            }
            return Ok(produit);
        }

        // POST: api/produits
        [HttpPost]
        public ActionResult<Produit> CreerProduit(Produit produit)
        {
            produit.Id = _produits.Max(p => p.Id) + 1;
            _produits.Add(produit);
            return CreatedAtAction(nameof(ObtenirProduit), new { id = produit.Id }, produit);
        }

        // PUT: api/produits/5
        [HttpPut("{id}")]
        public IActionResult MettreAJourProduit(int id, Produit produit)
        {
            if (id != produit.Id)
            {
                return BadRequest();
            }

            var index = _produits.FindIndex(p => p.Id == id);
            if (index < 0)
            {
                return NotFound();
            }

            _produits[index] = produit;
            return NoContent();
        }

        // DELETE: api/produits/5
        [HttpDelete("{id}")]
        public IActionResult SupprimerProduit(int id)
        {
            var produit = _produits.FirstOrDefault(p => p.Id == id);
            if (produit == null)
            {
                return NotFound();
            }

            _produits.Remove(produit);
            return NoContent();
        }
    }
}
```

**4. Configurez CORS pour permettre les requêtes JavaScript :**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services au conteneur
builder.Services.AddControllers();

// Ajouter et configurer CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin",
        builder => builder
            .WithOrigins("http://localhost:5500") // L'URL de votre frontend
            .AllowAnyHeader()
            .AllowAnyMethod());
});

// En savoir plus sur la configuration de Swagger/OpenAPI
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configurer le pipeline de requêtes HTTP
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Utiliser CORS
app.UseCors("AllowSpecificOrigin");

app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Consommation de l'API depuis JavaScript

Vous pouvez appeler votre API REST depuis JavaScript à l'aide de l'API Fetch :

```html
<!DOCTYPE html>
<html>
<head>
    <title>Gestion de Produits</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        tr:hover { background-color: #f5f5f5; }
        button { margin: 5px; padding: 5px 10px; }
        .form-group { margin-bottom: 10px; }
        label { display: inline-block; width: 100px; }
    </style>
</head>
<body>
    <h1>Gestion de Produits</h1>

    <div id="formulaire-produit">
        <h2>Ajouter/Modifier un produit</h2>
        <input type="hidden" id="produit-id">
        <div class="form-group">
            <label for="produit-nom">Nom:</label>
            <input type="text" id="produit-nom">
        </div>
        <div class="form-group">
            <label for="produit-description">Description:</label>
            <input type="text" id="produit-description">
        </div>
        <div class="form-group">
            <label for="produit-prix">Prix:</label>
            <input type="number" id="produit-prix" step="0.01">
        </div>
        <div class="form-group">
            <label for="produit-enstock">En stock:</label>
            <input type="checkbox" id="produit-enstock">
        </div>
        <button id="btn-enregistrer">Enregistrer</button>
        <button id="btn-annuler">Annuler</button>
    </div>

    <h2>Liste des produits</h2>
    <button id="btn-rafraichir">Rafraîchir</button>
    <button id="btn-ajouter">Ajouter un produit</button>

    <table id="table-produits">
        <thead>
            <tr>
                <th>ID</th>
                <th>Nom</th>
                <th>Description</th>
                <th>Prix</th>
                <th>En stock</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <!-- Données chargées dynamiquement -->
        </tbody>
    </table>

    <script>
        // URL de base de l'API
        const API_URL = 'https://localhost:7001/api/produits';

        // Éléments du DOM
        const tableProduits = document.getElementById('table-produits').getElementsByTagName('tbody')[0];
        const btnRafraichir = document.getElementById('btn-rafraichir');
        const btnAjouter = document.getElementById('btn-ajouter');
        const btnEnregistrer = document.getElementById('btn-enregistrer');
        const btnAnnuler = document.getElementById('btn-annuler');
        const formulaireProduit = document.getElementById('formulaire-produit');
        const produitId = document.getElementById('produit-id');
        const produitNom = document.getElementById('produit-nom');
        const produitDescription = document.getElementById('produit-description');
        const produitPrix = document.getElementById('produit-prix');
        const produitEnStock = document.getElementById('produit-enstock');

        // Charger tous les produits
        async function chargerProduits() {
            try {
                const response = await fetch(API_URL);
                if (!response.ok) {
                    throw new Error(`Erreur HTTP: ${response.status}`);
                }
                const produits = await response.json();

                // Vider le tableau
                tableProduits.innerHTML = '';

                // Remplir le tableau avec les données
                produits.forEach(produit => {
                    const ligne = tableProduits.insertRow();

                    ligne.insertCell(0).textContent = produit.id;
                    ligne.insertCell(1).textContent = produit.nom;
                    ligne.insertCell(2).textContent = produit.description;
                    ligne.insertCell(3).textContent = `${produit.prix.toFixed(2)} €`;
                    ligne.insertCell(4).textContent = produit.enStock ? 'Oui' : 'Non';

                    const cellActions = ligne.insertCell(5);
                    const btnModifier = document.createElement('button');
                    btnModifier.textContent = 'Modifier';
                    btnModifier.onclick = () => chargerProduit(produit.id);

                    const btnSupprimer = document.createElement('button');
                    btnSupprimer.textContent = 'Supprimer';
                    btnSupprimer.onclick = () => supprimerProduit(produit.id);

                    cellActions.appendChild(btnModifier);
                    cellActions.appendChild(btnSupprimer);
                });
            } catch (error) {
                console.error('Erreur lors du chargement des produits:', error);
                alert('Impossible de charger les produits. Veuillez réessayer plus tard.');
            }
        }

        // Charger un produit spécifique
        async function chargerProduit(id) {
            try {
                const response = await fetch(`${API_URL}/${id}`);
                if (!response.ok) {
                    throw new Error(`Erreur HTTP: ${response.status}`);
                }
                const produit = await response.json();

                // Remplir le formulaire
                produitId.value = produit.id;
                produitNom.value = produit.nom;
                produitDescription.value = produit.description;
                produitPrix.value = produit.prix;
                produitEnStock.checked = produit.enStock;

                // Afficher le formulaire
                formulaireProduit.style.display = 'block';
            } catch (error) {
                console.error(`Erreur lors du chargement du produit ${id}:`, error);
                alert('Impossible de charger le produit. Veuillez réessayer plus tard.');
            }
        }

        // Créer ou mettre à jour un produit
        async function enregistrerProduit() {
            try {
                const produit = {
                    id: produitId.value ? parseInt(produitId.value) : 0,
                    nom: produitNom.value,
                    description: produitDescription.value,
                    prix: parseFloat(produitPrix.value),
                    enStock: produitEnStock.checked
                };

                const method = produit.id ? 'PUT' : 'POST';
                const url = produit.id ? `${API_URL}/${produit.id}` : API_URL;

                const response = await fetch(url, {
                    method: method,
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(produit)
                });

                if (!response.ok) {
                    throw new Error(`Erreur HTTP: ${response.status}`);
                }

                // Réinitialiser le formulaire et recharger les produits
                reinitialiserFormulaire();
                chargerProduits();

                alert(`Produit ${produit.id ? 'mis à jour' : 'créé'} avec succès!`);
            } catch (error) {
                console.error('Erreur lors de l\'enregistrement du produit:', error);
                alert('Impossible d\'enregistrer le produit. Veuillez réessayer plus tard.');
            }
        }

        // Supprimer un produit
        async function supprimerProduit(id) {
            if (!confirm(`Êtes-vous sûr de vouloir supprimer le produit ${id}?`)) {
                return;
            }

            try {
                const response = await fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                });

                if (!response.ok) {
                    throw new Error(`Erreur HTTP: ${response.status}`);
                }

                // Recharger les produits
                chargerProduits();

                alert('Produit supprimé avec succès!');
            } catch (error) {
                console.error(`Erreur lors de la suppression du produit ${id}:`, error);
                alert('Impossible de supprimer le produit. Veuillez réessayer plus tard.');
            }
        }

        // Réinitialiser le formulaire
        function reinitialiserFormulaire() {
            produitId.value = '';
            produitNom.value = '';
            produitDescription.value = '';
            produitPrix.value = '';
            produitEnStock.checked = false;
            formulaireProduit.style.display = 'none';
        }

        // Gestionnaires d'événements
        btnRafraichir.addEventListener('click', chargerProduits);

        btnAjouter.addEventListener('click', () => {
            reinitialiserFormulaire();
            formulaireProduit.style.display = 'block';
        });

        btnEnregistrer.addEventListener('click', enregistrerProduit);

        btnAnnuler.addEventListener('click', reinitialiserFormulaire);

        // Charger les produits au démarrage
        document.addEventListener('DOMContentLoaded', () => {
            reinitialiserFormulaire(); // Cacher le formulaire initialement
            chargerProduits();
        });
    </script>
</body>
</html>
```

### Consommation de l'API depuis TypeScript

Pour une meilleure expérience de développement, utilisons TypeScript pour consommer notre API :

**1. Configurez un projet TypeScript :**

```bash
mkdir frontend-ts
cd frontend-ts
npm init -y
npm install --save-dev typescript webpack webpack-cli ts-loader html-webpack-plugin webpack-dev-server
```

**2. Créez un fichier tsconfig.json :**

```json
{
  "compilerOptions": {
    "target": "es2015",
    "module": "es2015",
    "strict": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "outDir": "./dist",
    "lib": ["dom", "es2015", "es2016", "es2017"]
  },
  "include": ["./src/**/*"]
}
```

**3. Créez un fichier webpack.config.js :**

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: './src/index.ts',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  resolve: {
    extensions: ['.ts', '.js']
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ],
  devServer: {
    static: {
      directory: path.join(__dirname, 'dist'),
    },
    port: 8080,
    open: true
  }
};
```

**4. Créez les fichiers source :**

Structure de répertoire :
```
frontend-ts/
├── src/
│   ├── index.html
│   ├── index.ts
│   ├── styles.css
│   ├── api/
│   │   └── produits-api.ts
│   └── models/
│       └── produit.ts
├── package.json
├── tsconfig.json
└── webpack.config.js
```

**src/models/produit.ts :**
```typescript
export interface Produit {
    id: number;
    nom: string;
    description: string;
    prix: number;
    enStock: boolean;
}

export interface NouveauProduit extends Omit<Produit, 'id'> {
    id?: number;
}
```

**src/api/produits-api.ts :**
```typescript
import { Produit, NouveauProduit } from '../models/produit';

const API_URL = 'https://localhost:7001/api/produits';

export class ProduitsApi {
    async obtenirTousProduits(): Promise<Produit[]> {
        const response = await fetch(API_URL);
        if (!response.ok) {
            throw new Error(`Erreur HTTP: ${response.status}`);
        }
        return await response.json() as Produit[];
    }

    async obtenirProduit(id: number): Promise<Produit> {
        const response = await fetch(`${API_URL}/${id}`);
        if (!response.ok) {
            throw new Error(`Erreur HTTP: ${response.status}`);
        }
        return await response.json() as Produit;
    }

    async creerProduit(produit: NouveauProduit): Promise<Produit> {
        const response = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(produit)
        });
        if (!response.ok) {
            throw new Error(`Erreur HTTP: ${response.status}`);
        }
        return await response.json() as Produit;
    }

    async mettreAJourProduit(id: number, produit: Produit): Promise<void> {
        const response = await fetch(`${API_URL}/${id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(produit)
        });
        if (!response.ok) {
            throw new Error(`Erreur HTTP: ${response.status}`);
        }
    }

    async supprimerProduit(id: number): Promise<void> {
        const response = await fetch(`${API_URL}/${id}`, {
            method: 'DELETE'
        });
        if (!response.ok) {
            throw new Error(`Erreur HTTP: ${response.status}`);
        }
    }
}

// Singleton pour l'API
export const produitsApi = new ProduitsApi();
```

**src/index.html :**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Gestion de Produits - TypeScript</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <h1>Gestion de Produits</h1>

    <div id="formulaire-produit">
        <h2>Ajouter/Modifier un produit</h2>
        <form id="form-produit">
            <input type="hidden" id="produit-id">
            <div class="form-group">
                <label for="produit-nom">Nom:</label>
                <input type="text" id="produit-nom" required>
            </div>
            <div class="form-group">
                <label for="produit-description">Description:</label>
                <input type="text" id="produit-description">
            </div>
            <div class="form-group">
                <label for="produit-prix">Prix:</label>
                <input type="number" id="produit-prix" step="0.01" required>
            </div>
            <div class="form-group">
                <label for="produit-enstock">En stock:</label>
                <input type="checkbox" id="produit-enstock">
            </div>
            <button type="submit">Enregistrer</button>
            <button type="button" id="btn-annuler">Annuler</button>
        </form>
    </div>

    <h2>Liste des produits</h2>
    <button id="btn-rafraichir">Rafraîchir</button>
    <button id="btn-ajouter">Ajouter un produit</button>

    <table id="table-produits">
        <thead>
            <tr>
                <th>ID</th>
                <th>Nom</th>
                <th>Description</th>
                <th>Prix</th>
                <th>En stock</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <!-- Données chargées dynamiquement -->
        </tbody>
    </table>
</body>
</html>
```

**src/styles.css :**
```css
body {
    font-family: Arial, sans-serif;
    margin: 20px;
    line-height: 1.6;
}

h1, h2 {
    color: #333;
}

.form-group {
    margin-bottom: 15px;
}

label {
    display: inline-block;
    width: 120px;
    font-weight: bold;
}

input[type="text"], input[type="number"] {
    width: 300px;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
}

button {
    background-color: #4CAF50;
    color: white;
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-right: 10px;
}

button:hover {
    background-color: #45a049;
}

button[type="button"] {
    background-color: #f44336;
}

button[type="button"]:hover {
    background-color: #d32f2f;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

th, td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
}

th {
    background-color: #f2f2f2;
    font-weight: bold;
}

tr:nth-child(even) {
    background-color: #f9f9f9;
}

tr:hover {
    background-color: #f5f5f5;
}

#formulaire-produit {
    background-color: #f8f8f8;
    padding: 20px;
    border-radius: 5px;
    margin-bottom: 20px;
}

.actions-cell button {
    padding: 5px 10px;
    margin-right: 5px;
}
```

**src/index.ts :**
```typescript
import './styles.css';
import { Produit, NouveauProduit } from './models/produit';
import { produitsApi } from './api/produits-api';

// Éléments du DOM
const tableProduits = document.getElementById('table-produits') as HTMLTableElement;
const tbody = tableProduits.getElementsByTagName('tbody')[0];
const formProduit = document.getElementById('form-produit') as HTMLFormElement;
const formulaireProduit = document.getElementById('formulaire-produit') as HTMLDivElement;
const produitId = document.getElementById('produit-id') as HTMLInputElement;
const produitNom = document.getElementById('produit-nom') as HTMLInputElement;
const produitDescription = document.getElementById('produit-description') as HTMLInputElement;
const produitPrix = document.getElementById('produit-prix') as HTMLInputElement;
const produitEnStock = document.getElementById('produit-enstock') as HTMLInputElement;
const btnRafraichir = document.getElementById('btn-rafraichir') as HTMLButtonElement;
const btnAjouter = document.getElementById('btn-ajouter') as HTMLButtonElement;
const btnAnnuler = document.getElementById('btn-annuler') as HTMLButtonElement;

// Charger tous les produits
async function chargerProduits(): Promise<void> {
    try {
        const produits = await produitsApi.obtenirTousProduits();

        // Vider le tableau
        tbody.innerHTML = '';

        // Remplir le tableau avec les données
        produits.forEach(produit => {
            const ligne = tbody.insertRow();

            ligne.insertCell(0).textContent = produit.id.toString();
            ligne.insertCell(1).textContent = produit.nom;
            ligne.insertCell(2).textContent = produit.description;
            ligne.insertCell(3).textContent = `${produit.prix.toFixed(2)} €`;
            ligne.insertCell(4).textContent = produit.enStock ? 'Oui' : 'Non';

            const cellActions = ligne.insertCell(5);
            cellActions.className = 'actions-cell';

            const btnModifier = document.createElement('button');
            btnModifier.textContent = 'Modifier';
            btnModifier.addEventListener('click', () => chargerProduit(produit.id));

            const btnSupprimer = document.createElement('button');
            btnSupprimer.textContent = 'Supprimer';
            btnSupprimer.addEventListener('click', () => supprimerProduit(produit.id));

            cellActions.appendChild(btnModifier);
            cellActions.appendChild(btnSupprimer);
        });
    } catch (error) {
        console.error('Erreur lors du chargement des produits:', error);
        alert('Impossible de charger les produits. Veuillez réessayer plus tard.');
    }
}

// Charger un produit spécifique
async function chargerProduit(id: number): Promise<void> {
    try {
        const produit = await produitsApi.obtenirProduit(id);

        // Remplir le formulaire
        produitId.value = produit.id.toString();
        produitNom.value = produit.nom;
        produitDescription.value = produit.description;
        produitPrix.value = produit.prix.toString();
        produitEnStock.checked = produit.enStock;

        // Afficher le formulaire
        formulaireProduit.style.display = 'block';
    } catch (error) {
        console.error(`Erreur lors du chargement du produit ${id}:`, error);
        alert('Impossible de charger le produit. Veuillez réessayer plus tard.');
    }
}

// Enregistrer un produit (création ou mise à jour)
async function enregistrerProduit(event: Event): Promise<void> {
    event.preventDefault();

    try {
        const id = produitId.value ? parseInt(produitId.value) : 0;
        const produit: NouveauProduit = {
            nom: produitNom.value,
            description: produitDescription.value,
            prix: parseFloat(produitPrix.value),
            enStock: produitEnStock.checked
        };

        if (id) {
            // Mise à jour
            await produitsApi.mettreAJourProduit(id, { ...produit, id });
            alert('Produit mis à jour avec succès!');
        } else {
            // Création
            await produitsApi.creerProduit(produit);
            alert('Produit créé avec succès!');
        }

        // Réinitialiser le formulaire et recharger les produits
        reinitialiserFormulaire();
        chargerProduits();
    } catch (error) {
        console.error('Erreur lors de l\'enregistrement du produit:', error);
        alert('Impossible d\'enregistrer le produit. Veuillez réessayer plus tard.');
    }
}

// Supprimer un produit
async function supprimerProduit(id: number): Promise<void> {
    if (!confirm(`Êtes-vous sûr de vouloir supprimer le produit ${id}?`)) {
        return;
    }

    try {
        await produitsApi.supprimerProduit(id);
        alert('Produit supprimé avec succès!');
        chargerProduits();
    } catch (error) {
        console.error(`Erreur lors de la suppression du produit ${id}:`, error);
        alert('Impossible de supprimer le produit. Veuillez réessayer plus tard.');
    }
}

// Réinitialiser le formulaire
function reinitialiserFormulaire(): void {
    formProduit.reset();
    produitId.value = '';
    formulaireProduit.style.display = 'none';
}

// Gestionnaires d'événements
formProduit.addEventListener('submit', enregistrerProduit);
btnRafraichir.addEventListener('click', chargerProduits);
btnAjouter.addEventListener('click', () => {
    reinitialiserFormulaire();
    formulaireProduit.style.display = 'block';
});
btnAnnuler.addEventListener('click', reinitialiserFormulaire);

// Initialisation
document.addEventListener('DOMContentLoaded', () => {
    formulaireProduit.style.display = 'none';
    chargerProduits();
});
```

**5. Ajoutez des scripts à package.json :**

```json
"scripts": {
  "start": "webpack serve",
  "build": "webpack --mode=production",
  "dev": "webpack --mode=development"
}
```

**6. Exécutez l'application :**

```bash
npm start
```

Cette approche TypeScript offre :
- Une meilleure organisation du code avec séparation des préoccupations
- La sécurité de type qui prévient les erreurs avant même l'exécution
- Une meilleure expérience de développement avec autocomplétion
- Une structure plus maintenable pour les projets complexes

### Bonnes pratiques pour les API REST

1. **Utilisez SSL/TLS (HTTPS)** pour sécuriser les communications
2. **Implémentez l'authentification et l'autorisation** (JWT, OAuth, etc.)
3. **Versionnez votre API** (/api/v1/produits)
4. **Utilisez correctement les codes de statut HTTP**
5. **Implémentez la pagination** pour les grandes collections
6. **Incluez la gestion des erreurs complète**
7. **Documentez votre API** avec Swagger/OpenAPI
8. **Utilisez les en-têtes HTTP appropriés** (Cache-Control, Content-Type, etc.)
9. **Implémentez CORS de manière sécurisée**
10. **Utilisez DTOs (Data Transfer Objects)** pour séparer les modèles de base de données des modèles d'API

## 18.4.4. gRPC-Web

### Qu'est-ce que gRPC-Web ?

gRPC-Web est une version du framework gRPC adaptée pour les navigateurs web. gRPC est un framework RPC (Remote Procedure Call) haute performance développé par Google qui utilise HTTP/2 pour le transport et Protocol Buffers pour la sérialisation.

Les avantages de gRPC-Web par rapport aux API REST traditionnelles incluent :
- Une communication plus efficace avec la sérialisation binaire Protocol Buffers
- Des contrats d'API stricts avec une définition de service claire (.proto)
- Génération de code client automatique
- Communication bidirectionnelle et streaming
- Performances améliorées pour les appels fréquents

### Configuration de gRPC-Web avec ASP.NET Core

**1. Créez un nouveau projet ASP.NET Core gRPC :**

```bash
dotnet new grpc -o MonServiceGrpc
cd MonServiceGrpc
```

**2. Ajoutez les packages NuGet nécessaires :**

```bash
dotnet add package Grpc.AspNetCore.Web
```

**3. Définissez vos services avec Protocol Buffers :**

Créez ou modifiez le fichier `Protos/produits.proto` :

```protobuf
syntax = "proto3";

option csharp_namespace = "MonServiceGrpc.Protos";

package produits;

// Définition du service
service ProduitsService {
  // Obtenir tous les produits
  rpc ObtenirProduits (RequeteProduits) returns (ReponseProduits);

  // Obtenir un produit par ID
  rpc ObtenirProduit (RequeteIdProduit) returns (Produit);

  // Créer un nouveau produit
  rpc CreerProduit (Produit) returns (Produit);

  // Mettre à jour un produit
  rpc MettreAJourProduit (Produit) returns (Produit);

  // Supprimer un produit
  rpc SupprimerProduit (RequeteIdProduit) returns (ReponseVide);
}

// Message vide pour les réponses sans contenu
message ReponseVide {}

// Requête pour obtenir les produits (peut contenir des filtres)
message RequeteProduits {
  bool seulement_en_stock = 1;
}

// Requête avec juste un ID
message RequeteIdProduit {
  int32 id = 1;
}

// Réponse contenant plusieurs produits
message ReponseProduits {
  repeated Produit produits = 1;
}

// Modèle de produit
message Produit {
  int32 id = 1;
  string nom = 2;
  string description = 3;
  double prix = 4;
  bool en_stock = 5;
}
```

**4. Configurez votre fichier .csproj pour inclure le fichier proto :**

```xml
<ItemGroup>
  <Protobuf Include="Protos\produits.proto" GrpcServices="Server" />
</ItemGroup>
```

**5. Créez le service d'implémentation :**

```csharp
// Services/ProduitsService.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Grpc.Core;
using Microsoft.Extensions.Logging;
using MonServiceGrpc.Protos;

namespace MonServiceGrpc.Services
{
    public class ProduitsService : Protos.ProduitsService.ProduitsServiceBase
    {
        private readonly ILogger<ProduitsService> _logger;
        // Une liste en mémoire pour simuler une base de données
        private static readonly List<Produit> _produits = new List<Produit>
        {
            new Produit { Id = 1, Nom = "Ordinateur portable", Description = "Ordinateur portable haute performance", Prix = 999.99, EnStock = true },
            new Produit { Id = 2, Nom = "Smartphone", Description = "Smartphone dernière génération", Prix = 699.99, EnStock = true },
            new Produit { Id = 3, Nom = "Casque audio", Description = "Casque audio sans fil", Prix = 149.99, EnStock = false }
        };

        public ProduitsService(ILogger<ProduitsService> logger)
        {
            _logger = logger;
        }

        public override Task<ReponseProduits> ObtenirProduits(RequeteProduits requete, ServerCallContext context)
        {
            _logger.LogInformation("Récupération de tous les produits");

            var reponse = new ReponseProduits();

            var produits = _produits.AsQueryable();

            // Filtrer si nécessaire
            if (requete.SeulementEnStock)
            {
                produits = produits.Where(p => p.EnStock);
            }

            // Ajouter les produits à la réponse
            reponse.Produits.AddRange(produits);

            return Task.FromResult(reponse);
        }

        public override Task<Produit> ObtenirProduit(RequeteIdProduit requete, ServerCallContext context)
        {
            _logger.LogInformation($"Récupération du produit avec ID: {requete.Id}");

            var produit = _produits.FirstOrDefault(p => p.Id == requete.Id);

            if (produit == null)
            {
                throw new RpcException(new Status(StatusCode.NotFound, $"Produit avec ID {requete.Id} non trouvé"));
            }

            return Task.FromResult(produit);
        }

        public override Task<Produit> CreerProduit(Produit produit, ServerCallContext context)
        {
            _logger.LogInformation("Création d'un nouveau produit");

            // Générer un nouvel ID
            produit.Id = _produits.Count > 0 ? _produits.Max(p => p.Id) + 1 : 1;

            // Ajouter à la liste
            _produits.Add(produit);

            return Task.FromResult(produit);
        }

        public override Task<Produit> MettreAJourProduit(Produit produit, ServerCallContext context)
        {
            _logger.LogInformation($"Mise à jour du produit avec ID: {produit.Id}");

            var index = _produits.FindIndex(p => p.Id == produit.Id);

            if (index < 0)
            {
                throw new RpcException(new Status(StatusCode.NotFound, $"Produit avec ID {produit.Id} non trouvé"));
            }

            // Mettre à jour le produit
            _produits[index] = produit;

            return Task.FromResult(produit);
        }

        public override Task<ReponseVide> SupprimerProduit(RequeteIdProduit requete, ServerCallContext context)
        {
            _logger.LogInformation($"Suppression du produit avec ID: {requete.Id}");

            var produit = _produits.FirstOrDefault(p => p.Id == requete.Id);

            if (produit == null)
            {
                throw new RpcException(new Status(StatusCode.NotFound, $"Produit avec ID {requete.Id} non trouvé"));
            }

            // Supprimer le produit
            _produits.Remove(produit);

            return Task.FromResult(new ReponseVide());
        }
    }
}
```

**6. Configurez le middleware dans Program.cs :**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services au conteneur.
builder.Services.AddGrpc();

// Ajouter CORS pour permettre les requêtes du navigateur
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", builder =>
    {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader()
               .WithExposedHeaders("Grpc-Status", "Grpc-Message", "Grpc-Encoding", "Grpc-Accept-Encoding");
    });
});

var app = builder.Build();

// Configurer le pipeline de requêtes HTTP.
app.UseRouting();

// Utiliser CORS
app.UseCors("AllowAll");
app.UseGrpcWeb();

app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<ProduitsService>().EnableGrpcWeb().RequireCors("AllowAll");
    endpoints.MapGet("/", () => "Communication avec gRPC endpoints doit être faite via un client gRPC.");
});

app.Run();
```

### Configuration du client gRPC-Web en JavaScript

Pour utiliser gRPC-Web dans un navigateur, vous aurez besoin des bibliothèques client appropriées :

**1. Créez un projet frontend avec npm :**

```bash
mkdir grpc-web-client
cd grpc-web-client
npm init -y
npm install --save grpc-web google-protobuf
npm install --save-dev webpack webpack-cli webpack-dev-server ts-loader typescript
```

**2. Créez un fichier tsconfig.json :**

```json
{
  "compilerOptions": {
    "target": "es2015",
    "module": "es2015",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "dist",
    "sourceMap": true,
    "declaration": true,
    "lib": ["dom", "es2015", "es2016", "es2017"]
  },
  "include": ["src/**/*"]
}
```

**3. Générez le code client JavaScript à partir du proto :**

Pour générer le code client, vous avez besoin du compilateur protoc et du plugin gRPC-Web. Vous pouvez les installer à partir des releases GitHub :

- [protoc](https://github.com/protocolbuffers/protobuf/releases)
- [protoc-gen-grpc-web](https://github.com/grpc/grpc-web/releases)

Une fois installés, copiez votre fichier proto dans un répertoire du projet frontend, puis exécutez :

```bash
mkdir -p src/generated
protoc -I=. produits.proto \
  --js_out=import_style=commonjs:src/generated \
  --grpc-web_out=import_style=typescript,mode=grpcwebtext:src/generated
```

**4. Créez votre application TypeScript :**

```typescript
// src/index.ts
import { ProduitsServiceClient } from './generated/Produits_grpc_web_pb';
import { RequeteProduits, RequeteIdProduit, Produit } from './generated/Produits_pb';

// Créer une instance du client gRPC
const client = new ProduitsServiceClient('https://localhost:7042');

// Fonction pour charger tous les produits
function chargerProduits(seulementEnStock: boolean = false): void {
    const request = new RequeteProduits();
    request.setSeulementEnStock(seulementEnStock);

    client.obtenirProduits(request, {}, (err, response) => {
        if (err) {
            console.error('Erreur:', err);
            return;
        }

        const produitsList = response.getProduitsList();
        const produitsDiv = document.getElementById('produits');

        if (produitsDiv) {
            produitsDiv.innerHTML = '';

            produitsList.forEach(produit => {
                const produitElement = document.createElement('div');
                produitElement.className = 'produit';

                produitElement.innerHTML = `
                    <h3>${produit.getNom()}</h3>
                    <p>${produit.getDescription()}</p>
                    <p>Prix: ${produit.getPrix().toFixed(2)} €</p>
                    <p>En stock: ${produit.getEnStock() ? 'Oui' : 'Non'}</p>
                    <button onclick="modifierProduit(${produit.getId()})">Modifier</button>
                    <button onclick="supprimerProduit(${produit.getId()})">Supprimer</button>
                `;

                produitsDiv.appendChild(produitElement);
            });
        }
    });
}

// Fonction pour obtenir un produit par ID
function obtenirProduit(id: number): void {
    const request = new RequeteIdProduit();
    request.setId(id);

    client.obtenirProduit(request, {}, (err, response) => {
        if (err) {
            console.error('Erreur:', err);
            return;
        }

        const formulaire = document.getElementById('formulaire-produit') as HTMLFormElement;
        formulaire.elements['id'].value = response.getId().toString();
        formulaire.elements['nom'].value = response.getNom();
        formulaire.elements['description'].value = response.getDescription();
        formulaire.elements['prix'].value = response.getPrix().toString();
        formulaire.elements['enStock'].checked = response.getEnStock();

        // Afficher le formulaire
        formulaire.style.display = 'block';
    });
}

// Fonction pour créer ou mettre à jour un produit
function enregistrerProduit(event: Event): void {
    event.preventDefault();

    const formulaire = event.target as HTMLFormElement;
    const idValue = formulaire.elements['id'].value;

    const produit = new Produit();
    if (idValue) {
        produit.setId(parseInt(idValue));
    }
    produit.setNom(formulaire.elements['nom'].value);
    produit.setDescription(formulaire.elements['description'].value);
    produit.setPrix(parseFloat(formulaire.elements['prix'].value));
    produit.setEnStock(formulaire.elements['enStock'].checked);

    if (idValue) {
        // Mise à jour
        client.mettreAJourProduit(produit, {}, (err, response) => {
            if (err) {
                console.error('Erreur lors de la mise à jour:', err);
                return;
            }

            console.log('Produit mis à jour avec succès:', response.getId());
            formulaire.reset();
            formulaire.style.display = 'none';
            chargerProduits();
        });
    } else {
        // Création
        client.creerProduit(produit, {}, (err, response) => {
            if (err) {
                console.error('Erreur lors de la création:', err);
                return;
            }

            console.log('Produit créé avec succès:', response.getId());
            formulaire.reset();
            formulaire.style.display = 'none';
            chargerProduits();
        });
    }
}

// Fonction pour supprimer un produit
function supprimerProduit(id: number): void {
    if (!confirm(`Êtes-vous sûr de vouloir supprimer le produit ${id}?`)) {
        return;
    }

    const request = new RequeteIdProduit();
    request.setId(id);

    client.supprimerProduit(request, {}, (err, response) => {
        if (err) {
            console.error('Erreur lors de la suppression:', err);
            return;
        }

        console.log('Produit supprimé avec succès');
        chargerProduits();
    });
}

// Configuration des événements
document.addEventListener('DOMContentLoaded', () => {
    // Charger les produits au démarrage
    chargerProduits();

    // Configuration du formulaire
    const formulaire = document.getElementById('formulaire-produit') as HTMLFormElement;
    formulaire.addEventListener('submit', enregistrerProduit);

    // Bouton pour afficher le formulaire d'ajout
    const btnAjouter = document.getElementById('btn-ajouter');
    if (btnAjouter) {
        btnAjouter.addEventListener('click', () => {
            formulaire.reset();
            formulaire.elements['id'].value = '';
            formulaire.style.display = 'block';
        });
    }

    // Bouton pour filtrer les produits en stock
    const btnFiltrer = document.getElementById('btn-filtrer');
    if (btnFiltrer) {
        btnFiltrer.addEventListener('click', () => {
            const enStock = (document.getElementById('filtre-enstock') as HTMLInputElement).checked;
            chargerProduits(enStock);
        });
    }
});

// Exposer les fonctions au contexte global pour les utiliser dans les boutons
(window as any).modifierProduit = obtenirProduit;
(window as any).supprimerProduit = supprimerProduit;
```

**5. Créez votre fichier HTML :**

```html
<!DOCTYPE html>
<html>
<head>
    <title>gRPC-Web Produits</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .produit { border: 1px solid #ddd; padding: 15px; margin-bottom: 10px; }
        .formulaire { background-color: #f5f5f5; padding: 20px; margin-bottom: 20px; }
        .form-group { margin-bottom: 10px; }
        label { display: inline-block; width: 100px; }
        button { padding: 5px 10px; margin-right: 5px; }
    </style>
</head>
<body>
    <h1>Gestion des Produits avec gRPC-Web</h1>

    <div class="filtre">
        <input type="checkbox" id="filtre-enstock">
        <label for="filtre-enstock">Afficher uniquement les produits en stock</label>
        <button id="btn-filtrer">Filtrer</button>
    </div>

    <form id="formulaire-produit" class="formulaire" style="display: none;">
        <h2>Ajouter/Modifier un produit</h2>
        <input type="hidden" name="id">

        <div class="form-group">
            <label for="nom">Nom:</label>
            <input type="text" name="nom" id="nom" required>
        </div>

        <div class="form-group">
            <label for="description">Description:</label>
            <input type="text" name="description" id="description">
        </div>

        <div class="form-group">
            <label for="prix">Prix:</label>
            <input type="number" name="prix" id="prix" step="0.01" required>
        </div>

        <div class="form-group">
            <label for="enStock">En stock:</label>
            <input type="checkbox" name="enStock" id="enStock">
        </div>

        <button type="submit">Enregistrer</button>
        <button type="button" onclick="document.getElementById('formulaire-produit').style.display='none';">Annuler</button>
    </form>

    <button id="btn-ajouter">Ajouter un produit</button>

    <h2>Liste des produits</h2>
    <div id="produits">
        <!-- Les produits seront ajoutés ici dynamiquement -->
    </div>

    <script src="dist/main.js"></script>
</body>
</html>
```

**6. Configurez Webpack :**

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: [ '.tsx', '.ts', '.js' ]
  },
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  devServer: {
    static: {
      directory: path.join(__dirname, './'),
    },
    compress: true,
    port: 8080
  }
};
```

**7. Ajoutez des scripts à package.json :**

```json
"scripts": {
  "build": "webpack",
  "start": "webpack-dev-server --open"
}
```

### Avantages et inconvénients de gRPC-Web

**Avantages :**
- Contrats d'API stricts avec Protocol Buffers
- Performance accrue en raison de la sérialisation binaire efficace
- Génération automatique de code client
- API fortement typées
- Support du streaming côté serveur

**Inconvénients :**
- Configuration plus complexe par rapport à REST
- Courbe d'apprentissage plus abrupte
- Manque de support natif dans les navigateurs (nécessite un proxy)
- Moins adapté pour les APIs publiques par rapport à REST
- Outils de débogage moins développés que pour REST

## 18.4.5. JSInterop

### Qu'est-ce que JSInterop ?

JSInterop (JavaScript Interoperability) est une fonctionnalité de Blazor qui permet d'appeler du code JavaScript depuis C# et vice versa. Cela permet d'intégrer des bibliothèques JavaScript existantes dans vos applications Blazor et d'accéder aux API du navigateur qui ne sont pas directement exposées en C#.

### Appeler JavaScript depuis C#

**1. Référencer JavaScript dans votre application Blazor :**

Ajoutez un fichier JavaScript dans votre dossier `wwwroot/js` :

```javascript
// wwwroot/js/interop.js
window.interopFunctions = {
    // Afficher une alerte
    montrerAlerte: function (message) {
        alert(message);
        return "Alerte affichée !";
    },

    // Obtenir la position géographique
    obtenirPosition: function () {
        return new Promise((resolve, reject) => {
            if (!navigator.geolocation) {
                reject("La géolocalisation n'est pas supportée par ce navigateur.");
            } else {
                navigator.geolocation.getCurrentPosition(
                    position => {
                        resolve({
                            latitude: position.coords.latitude,
                            longitude: position.coords.longitude,
                            precision: position.coords.accuracy
                        });
                    },
                    error => {
                        reject(`Erreur de géolocalisation: ${error.message}`);
                    }
                );
            }
        });
    },

    // Manipuler le DOM
    mettreEnSurbrillance: function (elementId, couleur) {
        const element = document.getElementById(elementId);
        if (element) {
            // Enregistrer le style original
            const styleOriginal = element.style.backgroundColor;

            // Appliquer la surbrillance
            element.style.backgroundColor = couleur;

            // Revenir au style original après 2 secondes
            setTimeout(() => {
                element.style.backgroundColor = styleOriginal;
            }, 2000);

            return true;
        }
        return false;
    },

    // Utiliser une bibliothèque JavaScript existante (exemple avec Chart.js)
    creerGraphique: function (canvasId, donnees, options) {
        const ctx = document.getElementById(canvasId).getContext('2d');

        // Créer un graphique avec Chart.js
        new Chart(ctx, {
            type: options.type || 'bar',
            data: {
                labels: donnees.labels,
                datasets: [{
                    label: options.titre || 'Données',
                    data: donnees.valeurs,
                    backgroundColor: options.couleurs || [
                        'rgba(255, 99, 132, 0.2)',
                        'rgba(54, 162, 235, 0.2)',
                        'rgba(255, 206, 86, 0.2)',
                        'rgba(75, 192, 192, 0.2)',
                        'rgba(153, 102, 255, 0.2)'
                    ],
                    borderColor: options.bordures || [
                        'rgba(255, 99, 132, 1)',
                        'rgba(54, 162, 235, 1)',
                        'rgba(255, 206, 86, 1)',
                        'rgba(75, 192, 192, 1)',
                        'rgba(153, 102, 255, 1)'
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        });

        return true;
    }
};
```

**2. Référencez le fichier JavaScript dans votre fichier _Host.cshtml (Blazor Server) ou index.html (Blazor WebAssembly) :**

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="js/interop.js"></script>
```

**3. Injectez IJSRuntime dans votre composant Blazor :**

```razor
@page "/jsinterop"
@inject IJSRuntime JSRuntime

<h1>Démonstration de JSInterop</h1>

<div class="card mb-4">
    <div class="card-header">
        Alerte JavaScript
    </div>
    <div class="card-body">
        <div class="input-group">
            <input type="text" class="form-control" @bind="messageAlerte" placeholder="Entrez un message...">
            <button class="btn btn-primary" @onclick="MontrerAlerte">Afficher Alerte</button>
        </div>
        @if (!string.IsNullOrEmpty(resultatAlerte))
        {
            <div class="mt-2 alert alert-info">@resultatAlerte</div>
        }
    </div>
</div>

<div class="card mb-4">
    <div class="card-header">
        Géolocalisation
    </div>
    <div class="card-body">
        <button class="btn btn-primary" @onclick="ObtenirPosition">Obtenir ma position</button>
        @if (position != null)
        {
            <div class="mt-2">
                <p>Latitude: @position.Latitude</p>
                <p>Longitude: @position.Longitude</p>
                <p>Précision: @position.Precision mètres</p>
            </div>
        }
    </div>
</div>

<div class="card mb-4">
    <div class="card-header">
        Manipulation du DOM
    </div>
    <div class="card-body">
        <div id="elementCible" class="p-3 border rounded">
            Cet élément sera mis en surbrillance
        </div>
        <div class="mt-2">
            <button class="btn btn-success" @onclick="() => MettreEnSurbrillance('lightgreen')">Vert</button>
            <button class="btn btn-warning" @onclick="() => MettreEnSurbrillance('lightyellow')">Jaune</button>
            <button class="btn btn-danger" @onclick="() => MettreEnSurbrillance('lightcoral')">Rouge</button>
        </div>
    </div>
</div>

<div class="card mb-4">
    <div class="card-header">
        Graphique avec Chart.js
    </div>
    <div class="card-body">
        <div>
            <canvas id="monGraphique" width="400" height="200"></canvas>
        </div>
        <div class="mt-2">
            <button class="btn btn-primary" @onclick="CreerGraphique">Générer Graphique</button>
        </div>
    </div>
</div>

@code {
    private string messageAlerte = "Hello from Blazor!";
    private string resultatAlerte;
    private Position position;

    // Modèle pour la position
    private class Position
    {
        public double Latitude { get; set; }
        public double Longitude { get; set; }
        public double Precision { get; set; }
    }

    // Appel d'une fonction JavaScript simple
    private async Task MontrerAlerte()
    {
        resultatAlerte = await JSRuntime.InvokeAsync<string>(
            "interopFunctions.montrerAlerte",
            messageAlerte);
    }

    // Appel d'une fonction JavaScript qui retourne une promesse
    private async Task ObtenirPosition()
    {
        try
        {
            position = await JSRuntime.InvokeAsync<Position>(
                "interopFunctions.obtenirPosition");
        }
        catch (Exception ex)
        {
            resultatAlerte = $"Erreur: {ex.Message}";
        }
    }

    // Manipulation du DOM via JavaScript
    private async Task MettreEnSurbrillance(string couleur)
    {
        await JSRuntime.InvokeVoidAsync(
            "interopFunctions.mettreEnSurbrillance",
            "elementCible",
            couleur);
    }

    // Utilisation d'une bibliothèque JavaScript
    private async Task CreerGraphique()
    {
        var donnees = new
        {
            labels = new[] { "Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi" },
            valeurs = new[] { 12, 19, 3, 5, 2 }
        };

        var options = new
        {
            type = "bar",
            titre = "Ventes hebdomadaires"
        };

        await JSRuntime.InvokeVoidAsync(
            "interopFunctions.creerGraphique",
            "monGraphique",
            donnees,
            options);
    }
}
```

### Appeler C# depuis JavaScript

Pour appeler du code C# depuis JavaScript, nous devons exposer nos méthodes C# avec l'attribut `[JSInvokable]` :

**1. Créez une classe C# avec des méthodes exposées :**

```csharp
// Data/CalculService.cs
using System;
using Microsoft.JSInterop;

namespace MonAppBlazor.Data
{
    public class CalculService
    {
        [JSInvokable]
        public static int Additionner(int a, int b)
        {
            return a + b;
        }

        [JSInvokable]
        public static int Multiplier(int a, int b)
        {
            return a * b;
        }

        [JSInvokable]
        public string Formater(double valeur, string format)
        {
            return valeur.ToString(format);
        }
    }
}
```

**2. Exposez une instance de cette classe au JavaScript :**

```razor
@page "/appelcsharp"
@inject IJSRuntime JSRuntime
@using MonAppBlazor.Data

<h1>Appel de C# depuis JavaScript</h1>

<div class="card mb-4">
    <div class="card-header">
        Opérations mathématiques
    </div>
    <div class="card-body">
        <div class="row mb-3">
            <div class="col">
                <input type="number" class="form-control" id="nombre1" placeholder="Premier nombre">
            </div>
            <div class="col">
                <input type="number" class="form-control" id="nombre2" placeholder="Deuxième nombre">
            </div>
        </div>
        <div class="row">
            <div class="col">
                <button class="btn btn-primary" onclick="calculerAddition()">Additionner</button>
                <button class="btn btn-secondary" onclick="calculerMultiplication()">Multiplier</button>
            </div>
        </div>
        <div class="mt-3">
            <div id="resultat" class="alert alert-info" style="display:none;"></div>
        </div>
    </div>
</div>

<div class="card mb-4">
    <div class="card-header">
        Formatage de nombres
    </div>
    <div class="card-body">
        <div class="row mb-3">
            <div class="col">
                <input type="number" class="form-control" id="valeur" placeholder="Valeur" step="0.01">
            </div>
            <div class="col">
                <input type="text" class="form-control" id="format" placeholder="Format (ex: C2, N0, P1)">
            </div>
        </div>
        <div class="row">
            <div class="col">
                <button class="btn btn-primary" onclick="formaterNombre()">Formater</button>
            </div>
        </div>
        <div class="mt-3">
            <div id="resultatFormat" class="alert alert-info" style="display:none;"></div>
        </div>
    </div>
</div>

@code {
    private DotNetObjectReference<CalculService> _calculServiceRef;

    protected override void OnInitialized()
    {
        _calculServiceRef = DotNetObjectReference.Create(new CalculService());
    }

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            // Passer la référence de notre objet C# au JavaScript
            await JSRuntime.InvokeVoidAsync("initialiserInteropCSharp", _calculServiceRef);
        }
    }

    public void Dispose()
    {
        // Important : libérer la référence lorsque le composant est détruit
        _calculServiceRef?.Dispose();
    }
}
```

**3. Créez le code JavaScript pour appeler les méthodes C# :**

```javascript
// wwwroot/js/interop.js (ajoutez à la fin du fichier)

// Variable globale pour stocker la référence à l'objet .NET
var calculService = null;

// Fonction appelée depuis le composant Blazor pour initialiser la référence
window.initialiserInteropCSharp = function (dotnetRef) {
    calculService = dotnetRef;
};

// Fonction pour appeler la méthode C# 'Additionner'
window.calculerAddition = function () {
    const nombre1 = parseInt(document.getElementById('nombre1').value || 0);
    const nombre2 = parseInt(document.getElementById('nombre2').value || 0);

    // Appel de la méthode d'instance 'Additionner'
    calculService.invokeMethodAsync('Additionner', nombre1, nombre2)
        .then(resultat => {
            const resultatElement = document.getElementById('resultat');
            resultatElement.textContent = `Résultat de l'addition: ${resultat}`;
            resultatElement.style.display = 'block';
        })
        .catch(error => {
            console.error("Erreur lors de l'appel de la méthode Additionner:", error);
        });
};

// Fonction pour appeler la méthode C# 'Multiplier'
window.calculerMultiplication = function () {
    const nombre1 = parseInt(document.getElementById('nombre1').value || 0);
    const nombre2 = parseInt(document.getElementById('nombre2').value || 0);

    // Appel de la méthode d'instance 'Multiplier'
    calculService.invokeMethodAsync('Multiplier', nombre1, nombre2)
        .then(resultat => {
            const resultatElement = document.getElementById('resultat');
            resultatElement.textContent = `Résultat de la multiplication: ${resultat}`;
            resultatElement.style.display = 'block';
        })
        .catch(error => {
            console.error("Erreur lors de l'appel de la méthode Multiplier:", error);
        });
};

// Fonction pour appeler la méthode C# 'Formater'
window.formaterNombre = function () {
    const valeur = parseFloat(document.getElementById('valeur').value || 0);
    const format = document.getElementById('format').value || "F2";

    // Appel de la méthode d'instance 'Formater'
    calculService.invokeMethodAsync('Formater', valeur, format)
        .then(resultat => {
            const resultatElement = document.getElementById('resultatFormat');
            resultatElement.textContent = `Résultat du formatage: ${resultat}`;
            resultatElement.style.display = 'block';
        })
        .catch(error => {
            console.error("Erreur lors de l'appel de la méthode Formater:", error);
        });
};
```

### Appel de méthodes statiques C# depuis JavaScript

Vous pouvez également appeler des méthodes statiques C# sans avoir besoin de créer une instance :

```javascript
// Appel d'une méthode statique
function appelMethodeStatique() {
    DotNet.invokeMethodAsync('MonAppBlazor', 'Additionner', 10, 20)
        .then(resultat => {
            console.log("Résultat de l'addition: " + resultat);
        });
}
```

Pour que cela fonctionne, la méthode statique doit être marquée avec `[JSInvokable]` :

```csharp
public static class CalculsStatiques
{
    [JSInvokable]
    public static int Additionner(int a, int b)
    {
        return a + b;
    }
}
```

### Bonnes pratiques pour JSInterop

1. **Minimisez les appels entre C# et JavaScript** : Chaque appel a un coût en performance, essayez de grouper les opérations.

2. **Utilisez Dispose pour les références** : Appelez toujours `Dispose()` sur les `DotNetObjectReference` lorsque vous n'en avez plus besoin.

3. **Gérez les erreurs** : Utilisez try/catch côté C# et .catch() côté JavaScript.

4. **Considérez l'asynchronie** : Toutes les opérations JSInterop sont asynchrones, utilisez `async/await` pour éviter de bloquer le thread UI.

5. **Modularisez votre code JavaScript** : Organisez vos fonctions JavaScript dans des modules pour une meilleure maintenabilité.

6. **Attention aux conversions de types** : Les types ne correspondent pas toujours exactement entre C# et JavaScript, soyez prudent avec les conversions.

7. **Déboguer JSInterop** : Utilisez `console.log()` côté JavaScript et les outils de débogage de votre navigateur.

### Utiliser JSInterop avec des bibliothèques JavaScript populaires

Voici un exemple d'intégration d'une bibliothèque populaire comme Leaflet (pour les cartes interactives) :

**1. Ajoutez les références à la bibliothèque dans votre HTML :**

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
```

**2. Créez le wrapper JavaScript :**

```javascript
// wwwroot/js/leafletInterop.js
window.leafletInterop = {
    map: null,

    // Initialiser la carte
    initialiserCarte: function (elementId, latitude, longitude, zoom) {
        if (this.map) {
            this.map.remove();
        }

        this.map = L.map(elementId).setView([latitude, longitude], zoom);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(this.map);

        return true;
    },

    // Ajouter un marqueur
    ajouterMarqueur: function (latitude, longitude, titre, description) {
        if (!this.map) return false;

        const marker = L.marker([latitude, longitude]).addTo(this.map);

        if (titre || description) {
            marker.bindPopup(`<b>${titre || ''}</b><br>${description || ''}`);
        }

        return true;
    },

    // Centrer la carte sur une position
    centrerCarte: function (latitude, longitude, zoom) {
        if (!this.map) return false;

        this.map.setView([latitude, longitude], zoom || this.map.getZoom());
        return true;
    }
};
```

**3. Créez un composant Blazor qui utilise cette bibliothèque :**

```razor
@page "/carte"
@inject IJSRuntime JSRuntime

<h1>Carte Interactive avec Leaflet</h1>

<div class="card mb-4">
    <div class="card-body">
        <div id="maCarte" style="height: 400px;"></div>
    </div>
</div>

<div class="row mb-3">
    <div class="col">
        <div class="input-group">
            <span class="input-group-text">Latitude</span>
            <input type="number" class="form-control" @bind="pointLatitude" step="0.000001">
        </div>
    </div>
    <div class="col">
        <div class="input-group">
            <span class="input-group-text">Longitude</span>
            <input type="number" class="form-control" @bind="pointLongitude" step="0.000001">
        </div>
    </div>
</div>

<div class="row mb-3">
    <div class="col">
        <div class="input-group">
            <span class="input-group-text">Titre</span>
            <input type="text" class="form-control" @bind="pointTitre">
        </div>
    </div>
    <div class="col">
        <div class="input-group">
            <span class="input-group-text">Description</span>
            <input type="text" class="form-control" @bind="pointDescription">
        </div>
    </div>
</div>

<div class="mb-3">
    <button class="btn btn-primary" @onclick="AjouterMarqueur">Ajouter un marqueur</button>
    <button class="btn btn-secondary" @onclick="CentrerCarte">Centrer la carte</button>
</div>

@code {
    // Position par défaut (Paris)
    private double carteLatitude = 48.8566;
    private double carteLongitude = 2.3522;
    private int carteZoom = 13;

    // Valeurs pour un nouveau marqueur
    private double pointLatitude = 48.8584;
    private double pointLongitude = 2.2945;
    private string pointTitre = "Tour Eiffel";
    private string pointDescription = "Monument emblématique de Paris";

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await InitialiserCarte();
        }
    }

    private async Task InitialiserCarte()
    {
        await JSRuntime.InvokeVoidAsync(
            "leafletInterop.initialiserCarte",
            "maCarte",
            carteLatitude,
            carteLongitude,
            carteZoom);
    }

    private async Task AjouterMarqueur()
    {
        await JSRuntime.InvokeVoidAsync(
            "leafletInterop.ajouterMarqueur",
            pointLatitude,
            pointLongitude,
            pointTitre,
            pointDescription);
    }

    private async Task CentrerCarte()
    {
        await JSRuntime.InvokeVoidAsync(
            "leafletInterop.centrerCarte",
            pointLatitude,
            pointLongitude,
            carteZoom);
    }
}
```

### Isoler JSInterop avec des services

Pour une meilleure organisation, vous pouvez encapsuler vos appels JSInterop dans des services :

```csharp
// Services/CarteService.cs
using System.Threading.Tasks;
using Microsoft.JSInterop;

namespace MonAppBlazor.Services
{
    public class CarteService
    {
        private readonly IJSRuntime _jsRuntime;

        public CarteService(IJSRuntime jsRuntime)
        {
            _jsRuntime = jsRuntime;
        }

        public async Task InitialiserCarte(string elementId, double latitude, double longitude, int zoom)
        {
            await _jsRuntime.InvokeVoidAsync(
                "leafletInterop.initialiserCarte",
                elementId,
                latitude,
                longitude,
                zoom);
        }

        public async Task AjouterMarqueur(double latitude, double longitude, string titre, string description)
        {
            await _jsRuntime.InvokeVoidAsync(
                "leafletInterop.ajouterMarqueur",
                latitude,
                longitude,
                titre,
                description);
        }

        public async Task CentrerCarte(double latitude, double longitude, int? zoom = null)
        {
            await _jsRuntime.InvokeVoidAsync(
                "leafletInterop.centrerCarte",
                latitude,
                longitude,
                zoom);
        }
    }
}
```

Puis enregistrez-le dans le conteneur de services (Program.cs) :

```csharp
builder.Services.AddScoped<CarteService>();
```

Et utilisez-le dans votre composant :

```razor
@page "/carte"
@inject CarteService CarteService

<h1>Carte Interactive avec Leaflet</h1>

<!-- ... -->

@code {
    // ...

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await CarteService.InitialiserCarte("maCarte", carteLatitude, carteLongitude, carteZoom);
        }
    }

    private async Task AjouterMarqueur()
    {
        await CarteService.AjouterMarqueur(pointLatitude, pointLongitude, pointTitre, pointDescription);
    }

    private async Task CentrerCarte()
    {
        await CarteService.CentrerCarte(pointLatitude, pointLongitude, carteZoom);
    }
}
```

## Conclusion

L'intégration de C# avec JavaScript/TypeScript offre de nombreuses possibilités pour développer des applications web modernes. Dans ce chapitre, nous avons exploré cinq approches principales :

1. **Blazor WebAssembly** - Pour exécuter du code C# directement dans le navigateur, offrant une expérience de développement unifiée.

2. **SignalR** - Pour la communication en temps réel bidirectionnelle, idéale pour les applications nécessitant des mises à jour instantanées.

3. **APIs REST et JSON** - L'approche traditionnelle et flexible pour la communication client-serveur, avec une grande interopérabilité.

4. **gRPC-Web** - Pour une communication plus efficace et fortement typée, offrant des performances améliorées pour les applications complexes.

5. **JSInterop** - Pour intégrer du code JavaScript existant dans vos applications Blazor, combinant le meilleur des deux mondes.

Chaque approche a ses forces et ses faiblesses, et le choix dépendra de vos besoins spécifiques :

- **Utilisez Blazor WebAssembly** quand vous souhaitez une expérience de développement C# unifiée et que vous n'avez pas besoin d'utiliser des bibliothèques JavaScript complexes.

- **Utilisez SignalR** pour les applications nécessitant des mises à jour en temps réel comme les chats, les tableaux de bord en direct ou les jeux.

- **Utilisez REST** pour les interfaces les plus simples et les plus compatibles, en particulier pour les APIs publiques.

- **Utilisez gRPC-Web** pour une communication plus efficace avec des contrats stricts, particulièrement utile pour les systèmes internes complexes.

- **Utilisez JSInterop** lorsque vous devez intégrer des bibliothèques JavaScript existantes dans vos applications Blazor.

Ces technologies ne sont pas exclusives : vous pouvez les combiner dans une même application. Par exemple, vous pourriez avoir une application Blazor qui utilise JSInterop pour intégrer des bibliothèques JavaScript spécifiques, SignalR pour les notifications en temps réel, et des API REST pour certaines fonctionnalités.

En maîtrisant ces différentes approches, vous serez en mesure de choisir la meilleure solution pour chaque cas d'utilisation et de créer des applications web modernes, performantes et maintenables.

## Exercices pratiques

1. Créez une application de chat simple avec Blazor WebAssembly et SignalR qui permet à plusieurs utilisateurs de communiquer en temps réel.

2. Développez une application CRUD complète avec une API REST en C# et une interface utilisateur en TypeScript qui permet de gérer une collection d'articles.

3. Intégrez une bibliothèque JavaScript populaire (comme Three.js pour la 3D ou D3.js pour la visualisation de données) dans une application Blazor en utilisant JSInterop.

4. Créez un service gRPC-Web qui expose des données en temps réel (comme des cours boursiers ou des métriques système) et une application cliente qui les affiche.

5. Développez une application hybride qui utilise à la fois Blazor pour l'interface principale et React pour certains composants spécifiques, en les faisant communiquer via JSInterop.

## Ressources additionnelles

### Blazor WebAssembly
- [Documentation officielle Blazor](https://docs.microsoft.com/fr-fr/aspnet/core/blazor/)
- [Workshop Blazor](https://github.com/dotnet-presentations/blazor-workshop)
- [Awesome Blazor - liste de ressources](https://github.com/AdrienTorris/awesome-blazor)

### SignalR
- [Documentation SignalR](https://docs.microsoft.com/fr-fr/aspnet/core/signalr/introduction)
- [Tutoriel SignalR avec JavaScript](https://docs.microsoft.com/fr-fr/aspnet/core/tutorials/signalr)
- [SignalR avec Blazor](https://docs.microsoft.com/fr-fr/aspnet/core/blazor/tutorials/signalr-blazor)

### APIs REST
- [Tutoriel API Web ASP.NET Core](https://docs.microsoft.com/fr-fr/aspnet/core/tutorials/first-web-api)
- [REST API Best Practices](https://docs.microsoft.com/fr-fr/azure/architecture/best-practices/api-design)
- [TypeScript avec Fetch API](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)

### gRPC-Web
- [Documentation gRPC-Web](https://github.com/grpc/grpc-web)
- [gRPC avec ASP.NET Core](https://docs.microsoft.com/fr-fr/aspnet/core/grpc/)
- [Tutoriel gRPC-Web avec ASP.NET Core](https://docs.microsoft.com/fr-fr/aspnet/core/grpc/browser)

### JSInterop
- [Documentation JSInterop](https://docs.microsoft.com/fr-fr/aspnet/core/blazor/javascript-interoperability/)
- [Appel de fonctions JavaScript depuis C#](https://docs.microsoft.com/fr-fr/aspnet/core/blazor/javascript-interoperability/call-javascript-from-dotnet)
- [Appel de méthodes .NET depuis JavaScript](https://docs.microsoft.com/fr-fr/aspnet/core/blazor/javascript-interoperability/call-dotnet-from-javascript)

⏭️
