# 12.2. ASP.NET MVC

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 12.2.1. Architecture MVC

Le pattern MVC (Modèle-Vue-Contrôleur) est un modèle d'architecture logicielle qui divise une application en trois composants interconnectés. Cette séparation permet d'organiser le code de façon plus claire, de faciliter la maintenance et de promouvoir la réutilisation.

### Les trois composants du MVC

#### Modèle (Model)

- **Définition** : Représente les données et la logique métier de l'application.
- **Responsabilités** :
  - Accéder à la base de données
  - Valider les données
  - Appliquer les règles métier
  - Gérer l'état de l'application
  - Notifier les observateurs des changements (dans certaines implémentations)

- **Types de modèles** :
  1. **Modèles de domaine** : Représentent les entités métier (ex: Client, Produit)
  2. **Modèles de vue (ViewModel)** : Contiennent les données spécifiques à une vue
  3. **Modèles de transfert de données (DTO)** : Utilisés pour transférer des données entre les couches

- **Exemple de modèle** :
```csharp
public class Client
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Email { get; set; }
    public string Telephone { get; set; }
    public DateTime DateInscription { get; set; }

    // Méthode métier
    public bool EstClientPremium()
    {
        return (DateTime.Now - DateInscription).TotalDays > 365;
    }

    // Validation
    public bool EstValide()
    {
        return !string.IsNullOrEmpty(Nom) && !string.IsNullOrEmpty(Email);
    }
}
```

#### Vue (View)

- **Définition** : Interface utilisateur qui affiche les données au client et capture leurs interactions.
- **Responsabilités** :
  - Présenter les données au format approprié
  - Appliquer le formatage et les styles
  - Accepter les entrées de l'utilisateur
  - Transmettre les actions de l'utilisateur au contrôleur
  - Fournir une expérience utilisateur cohérente

- **Types de vues** :
  1. **Vues principales** : Pages complètes rendues par les actions du contrôleur
  2. **Vues partielles** : Fragments réutilisables inclus dans d'autres vues
  3. **Layouts** : Templates qui définissent la structure commune
  4. **Components** : (Dans ASP.NET Core) Vues encapsulées et réutilisables

- **Exemple de vue** (utilisant Razor) :
```cshtml
@model Client

<!DOCTYPE html>
<html>
<head>
    <title>Détails du client</title>
</head>
<body>
    <h1>Détails du client @Model.Id</h1>

    <div>
        <p><strong>Nom :</strong> @Model.Nom</p>
        <p><strong>Email :</strong> @Model.Email</p>
        <p><strong>Téléphone :</strong> @Model.Telephone</p>
        <p><strong>Date d'inscription :</strong> @Model.DateInscription.ToShortDateString()</p>

        @if (Model.EstClientPremium())
        {
            <p><strong>Statut :</strong> <span class="premium">Client Premium</span></p>
        }
    </div>

    <div>
        <a href="@Url.Action("Index")">Retour à la liste</a>
        <a href="@Url.Action("Edit", new { id = Model.Id })">Modifier</a>
    </div>
</body>
</html>
```

#### Contrôleur (Controller)

- **Définition** : Coordonne les interactions entre la vue et le modèle.
- **Responsabilités** :
  - Traiter les requêtes HTTP
  - Interagir avec le modèle pour obtenir ou mettre à jour des données
  - Valider les entrées utilisateur
  - Gérer le workflow de l'application
  - Sélectionner la vue appropriée à renvoyer
  - Transmettre les données du modèle à la vue
  - Gérer les erreurs et les redirections

- **Types de contrôleurs** :
  1. **Contrôleurs standard** : Gèrent les pages HTML traditionnelles
  2. **Contrôleurs API** : Renvoient des données (JSON, XML) plutôt que des vues HTML
  3. **Contrôleurs hybrides** : Peuvent faire les deux selon l'action

- **Exemple de contrôleur** :
```csharp
public class ClientsController : Controller
{
    private readonly IClientService _clientService;

    // Injection de dépendance
    public ClientsController(IClientService clientService)
    {
        _clientService = clientService;
    }

    // GET: /Clients/
    public IActionResult Index()
    {
        var clients = _clientService.ObtenirTousClients();
        return View(clients);
    }

    // GET: /Clients/Details/5
    public IActionResult Details(int id)
    {
        var client = _clientService.ObtenirClientParId(id);
        if (client == null)
        {
            return NotFound(); // Renvoie une réponse 404
        }
        return View(client);
    }

    // GET: /Clients/Create
    public IActionResult Create()
    {
        return View(new Client { DateInscription = DateTime.Now });
    }

    // POST: /Clients/Create
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Client client)
    {
        if (ModelState.IsValid)
        {
            _clientService.AjouterClient(client);
            return RedirectToAction(nameof(Index));
        }
        return View(client);
    }
}
```

### Flux de données complet dans le MVC

1. L'utilisateur interagit avec l'interface (ex: clique sur un lien "Détails du client")
2. Le navigateur envoie une requête HTTP au serveur
3. Le système de routing identifie le contrôleur et l'action à exécuter
4. Le contrôleur est instancié et l'action est appelée avec les paramètres appropriés
5. Le contrôleur interagit avec le modèle pour récupérer les données nécessaires
6. Le modèle effectue des opérations de données (requêtes BD, validations, calculs)
7. Le modèle renvoie les résultats au contrôleur
8. Le contrôleur prépare un modèle de vue (ViewModel) si nécessaire
9. Le contrôleur sélectionne une vue et lui passe le modèle
10. La vue rend le HTML en incorporant les données du modèle
11. Le contrôleur renvoie la réponse HTML au navigateur
12. Le navigateur affiche la page mise à jour à l'utilisateur

### Avantages du pattern MVC

- **Séparation des préoccupations** : Chaque composant a une responsabilité unique et bien définie.
- **Maintenabilité améliorée** : Modifier un composant a un impact minimal sur les autres.
- **Développement parallèle** : Différentes équipes peuvent travailler simultanément sur le modèle, la vue et le contrôleur.
- **Testabilité accrue** : Les composants peuvent être testés unitairement de façon isolée.
- **Réutilisabilité** : Les modèles et les contrôleurs peuvent être réutilisés avec différentes vues.
- **Séparation claire du code frontend et backend** : Les développeurs peuvent se spécialiser.
- **Structure cohérente** : Les applications MVC suivent une structure standardisée et prévisible.
- **Extensibilité** : Facilité d'ajout de nouvelles fonctionnalités sans perturber l'existant.

### Inconvénients et défis

- **Courbe d'apprentissage** : Plus complexe à comprendre pour les débutants qu'un modèle monolithique.
- **Surcharge potentielle** : Pour de petites applications, MVC peut sembler excessif.
- **Navigation entre fichiers** : Nécessite de naviguer entre plusieurs fichiers pour comprendre un flux complet.
- **Verbosité** : Peut nécessiter plus de code qu'une approche plus directe.

### MVC dans ASP.NET spécifiquement

ASP.NET MVC ajoute plusieurs éléments à l'architecture MVC de base :

- **Routing** : Système de mappage des URLs aux contrôleurs et actions
- **Razor** : Moteur de template pour les vues
- **Model Binding** : Liaison automatique des données HTTP aux paramètres d'action
- **Validation** : Système complet pour valider les entrées utilisateur
- **Filtres** : Ajout de comportements avant/après les actions
- **Areas** : Organisation des contrôleurs et vues en sections logiques
- **Dependency Injection** : Intégration native de l'injection de dépendances
- **Tag Helpers** : Aide à la génération de HTML dans les vues

## 12.2.2. Contrôleurs et actions

Dans ASP.NET MVC, les contrôleurs sont au cœur du framework. Ils traitent les requêtes entrantes, orchestrent les opérations sur les modèles et sélectionnent les vues à afficher.

### Anatomie d'un contrôleur

Un contrôleur dans ASP.NET MVC :
- Est une classe C# qui hérite généralement de la classe `Controller`
- A un nom qui se termine par "Controller" (ex: `ProductsController`)
- Contient des méthodes publiques appelées "actions"

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MonApplication.Controllers
{
    public class ClientsController : Controller
    {
        // Actions...
    }
}
```

La classe `Controller` de base fournit :
- Accès au contexte HTTP (`HttpContext`)
- Méthodes pour renvoyer différents types de résultats (`View()`, `Json()`, etc.)
- Propriétés comme `ModelState` pour la validation
- Helpers comme `Url` pour générer des URLs

### Structure du projet pour les contrôleurs

Dans un projet ASP.NET MVC standard, les contrôleurs sont placés dans un dossier nommé `Controllers` :

```
MonApplication/
  ├─ Controllers/
  │   ├─ HomeController.cs
  │   ├─ ClientsController.cs
  │   ├─ ProduitsController.cs
  │   └─ ...
  ├─ Models/
  ├─ Views/
  └─ ...
```

### Actions de contrôleur

Les actions sont des méthodes publiques dans une classe de contrôleur qui traitent les requêtes HTTP. Une action typique :

1. Reçoit des données d'entrée (paramètres)
2. Interagit avec les services ou modèles
3. Renvoie un `IActionResult`

#### Exemple d'actions CRUD complètes

```csharp
public class ClientsController : Controller
{
    private readonly IClientService _clientService;

    public ClientsController(IClientService clientService)
    {
        _clientService = clientService;
    }

    // GET: /Clients/
    public IActionResult Index(string searchTerm, int page = 1)
    {
        // Pagination et recherche
        var pageSize = 10;
        var clients = _clientService.ObtenirClientsParPage(searchTerm, page, pageSize, out int totalItems);

        // Création d'un ViewModel avec pagination
        var viewModel = new ClientsListViewModel
        {
            Clients = clients,
            Pagination = new PaginationInfo
            {
                CurrentPage = page,
                ItemsPerPage = pageSize,
                TotalItems = totalItems
            },
            SearchTerm = searchTerm
        };

        return View(viewModel);
    }

    // GET: /Clients/Details/5
    public IActionResult Details(int id)
    {
        var client = _clientService.ObtenirClientParId(id);
        if (client == null)
        {
            // Message d'erreur avec TempData (sera disponible dans la prochaine requête)
            TempData["ErrorMessage"] = $"Client avec ID {id} non trouvé.";
            return RedirectToAction(nameof(Index));
        }

        return View(client);
    }

    // GET: /Clients/Create
    public IActionResult Create()
    {
        // Préremplir des valeurs par défaut
        var client = new Client
        {
            DateInscription = DateTime.Now,
            Actif = true
        };

        // Préparer les listes déroulantes et autres données pour le formulaire
        ViewBag.Categories = _clientService.ObtenirToutesCategories();

        return View(client);
    }

    // POST: /Clients/Create
    [HttpPost]
    [ValidateAntiForgeryToken] // Protection CSRF
    public IActionResult Create(Client client)
    {
        // Validation manuelle supplémentaire si nécessaire
        if (string.IsNullOrEmpty(client.Email) && string.IsNullOrEmpty(client.Telephone))
        {
            ModelState.AddModelError("", "Vous devez fournir au moins un email ou un téléphone.");
        }

        if (ModelState.IsValid)
        {
            try
            {
                _clientService.AjouterClient(client);

                // Message de succès
                TempData["SuccessMessage"] = "Client créé avec succès !";

                return RedirectToAction(nameof(Details), new { id = client.Id });
            }
            catch (Exception ex)
            {
                // Gestion des erreurs
                ModelState.AddModelError("", $"Erreur lors de la création: {ex.Message}");
            }
        }

        // Si nous sommes ici, il y a eu une erreur, réafficher le formulaire
        ViewBag.Categories = _clientService.ObtenirToutesCategories();
        return View(client);
    }

    // GET: /Clients/Edit/5
    public IActionResult Edit(int id)
    {
        var client = _clientService.ObtenirClientParId(id);
        if (client == null)
        {
            return NotFound();
        }

        ViewBag.Categories = _clientService.ObtenirToutesCategories();
        return View(client);
    }

    // POST: /Clients/Edit/5
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Edit(int id, Client client)
    {
        // Vérifier que l'ID dans l'URL correspond à l'ID dans le modèle
        if (id != client.Id)
        {
            return BadRequest();
        }

        if (ModelState.IsValid)
        {
            try
            {
                _clientService.MettreAJourClient(client);
                TempData["SuccessMessage"] = "Client mis à jour avec succès !";
                return RedirectToAction(nameof(Details), new { id = client.Id });
            }
            catch (Exception ex)
            {
                ModelState.AddModelError("", $"Erreur lors de la mise à jour: {ex.Message}");
            }
        }

        ViewBag.Categories = _clientService.ObtenirToutesCategories();
        return View(client);
    }

    // GET: /Clients/Delete/5
    public IActionResult Delete(int id)
    {
        var client = _clientService.ObtenirClientParId(id);
        if (client == null)
        {
            return NotFound();
        }

        // Afficher une page de confirmation
        return View(client);
    }

    // POST: /Clients/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public IActionResult DeleteConfirmed(int id)
    {
        try
        {
            var client = _clientService.ObtenirClientParId(id);
            if (client == null)
            {
                return NotFound();
            }

            _clientService.SupprimerClient(id);
            TempData["SuccessMessage"] = "Client supprimé avec succès !";
            return RedirectToAction(nameof(Index));
        }
        catch (Exception ex)
        {
            // En cas d'erreur, réafficher la page de confirmation avec le message d'erreur
            TempData["ErrorMessage"] = $"Erreur lors de la suppression: {ex.Message}";
            return RedirectToAction(nameof(Delete), new { id });
        }
    }

    // Exemple d'action avec validation côté serveur via AJAX
    [AcceptVerbs("GET", "POST")]
    public IActionResult VerifierEmailDisponible(string email, int? id)
    {
        // Vérifier si l'email est disponible, en excluant le client actuel (en cas d'édition)
        var emailExiste = _clientService.EmailExiste(email, id);

        return Json(!emailExiste);
    }
}
```

### Types de résultats d'action (ActionResults)

ASP.NET MVC offre de nombreux types de résultats qui dérivent tous de `IActionResult` :

#### ViewResult (`return View()`)
Renvoie une vue HTML au client.

```csharp
// Vue par défaut (même nom que l'action)
return View();

// Vue spécifique
return View("Details");

// Vue avec modèle
return View(client);

// Vue spécifique avec modèle
return View("ClientDetails", client);

// Vue dans un autre dossier
return View("~/Views/Shared/Error.cshtml", errorViewModel);
```

#### PartialViewResult (`return PartialView()`)
Renvoie une vue partielle (fragment HTML).

```csharp
// Vue partielle avec modèle
return PartialView("_ClientCard", client);
```

#### JsonResult (`return Json()`)
Sérialise un objet en JSON et le renvoie.

```csharp
// Renvoyer du JSON simple
return Json(new { success = true, message = "Opération réussie" });

// Options de sérialisation
return Json(data, new JsonSerializerOptions
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
});
```

#### ContentResult (`return Content()`)
Renvoie du contenu texte simple.

```csharp
// Texte simple
return Content("Hello World");

// Avec type MIME et encodage
return Content("<h1>Hello</h1>", "text/html", Encoding.UTF8);
```

#### FileResult (`return File()`)
Renvoie un fichier binaire.

```csharp
// Fichier depuis un tableau d'octets
byte[] fileBytes = ...;
return File(fileBytes, "application/pdf", "rapport.pdf");

// Fichier depuis un stream
var stream = new FileStream("path/to/file.xlsx", FileMode.Open);
return File(stream, "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", "export.xlsx");

// Fichier physique
return PhysicalFile("/path/to/file.jpg", "image/jpeg", "photo.jpg");
```

#### RedirectResult (`return Redirect()`)
Redirige vers une URL spécifique.

```csharp
// Redirection absolue
return Redirect("https://example.com");

// Redirection relative
return Redirect("/Home/Index");

// Redirection permanente (code 301)
return RedirectPermanent("/nouvelle-page");
```

#### RedirectToActionResult (`return RedirectToAction()`)
Redirige vers une autre action, éventuellement dans un autre contrôleur.

```csharp
// Redirection vers une action du même contrôleur
return RedirectToAction("Index");

// Redirection avec paramètre
return RedirectToAction("Details", new { id = 123 });

// Redirection vers un autre contrôleur
return RedirectToAction("Index", "Home");

// Redirection avec contrôleur et paramètres
return RedirectToAction("Profile", "Users", new { username = "jdupont" });
```

#### StatusCodeResult (`return StatusCode()`)
Renvoie un code d'état HTTP spécifique.

```csharp
// Renvoyer un code 404 (Not Found)
return NotFound();

// Renvoyer un code 400 (Bad Request)
return BadRequest();

// Renvoyer un code 401 (Unauthorized)
return Unauthorized();

// Renvoyer un code 403 (Forbidden)
return Forbid();

// Renvoyer un code personnalisé avec contenu
return StatusCode(429, "Trop de requêtes. Veuillez réessayer plus tard.");
```

### Paramètres d'action

Les actions de contrôleur peuvent recevoir des paramètres de différentes sources :

#### Paramètres de requête URL (Query String)
```
GET /Clients/Search?terme=dupont&page=2
```

```csharp
public IActionResult Search(string terme, int page = 1)
{
    // terme = "dupont", page = 2
    ...
}
```

#### Paramètres de route
```
GET /Clients/Details/5
```

```csharp
[Route("Clients/Details/{id}")]
public IActionResult Details(int id)
{
    // id = 5
    ...
}
```

#### Paramètres de formulaire
```html
<form method="post">
    <input name="nom" value="Dupont" />
    <input name="email" value="dupont@example.com" />
    <button type="submit">Envoyer</button>
</form>
```

```csharp
[HttpPost]
public IActionResult Create(string nom, string email)
{
    // nom = "Dupont", email = "dupont@example.com"
    ...
}
```

#### Liaison de modèle (Model Binding)
```html
<form method="post">
    <input name="Nom" value="Dupont" />
    <input name="Email" value="dupont@example.com" />
    <input name="Adresse.Rue" value="123 Rue Principale" />
    <input name="Adresse.Ville" value="Paris" />
    <button type="submit">Envoyer</button>
</form>
```

```csharp
[HttpPost]
public IActionResult Create(Client client)
{
    // client.Nom = "Dupont"
    // client.Email = "dupont@example.com"
    // client.Adresse.Rue = "123 Rue Principale"
    // client.Adresse.Ville = "Paris"
    ...
}
```

#### Liaison de collection
```html
<form method="post">
    <input name="Clients[0].Nom" value="Dupont" />
    <input name="Clients[0].Email" value="dupont@example.com" />
    <input name="Clients[1].Nom" value="Martin" />
    <input name="Clients[1].Email" value="martin@example.com" />
    <button type="submit">Envoyer</button>
</form>
```

```csharp
[HttpPost]
public IActionResult CreateMultiple(List<Client> clients)
{
    // clients[0].Nom = "Dupont"
    // clients[0].Email = "dupont@example.com"
    // clients[1].Nom = "Martin"
    // clients[1].Email = "martin@example.com"
    ...
}
```

### Attributs HTTP et restrictions de méthode

Les actions peuvent être limitées à des méthodes HTTP spécifiques :

```csharp
// Répond uniquement aux requêtes GET
[HttpGet]
public IActionResult Create()
{
    return View();
}

// Répond uniquement aux requêtes POST
[HttpPost]
public IActionResult Create(Client client)
{
    // Traitement du formulaire...
    return RedirectToAction("Index");
}

// Répond à plusieurs méthodes HTTP
[AcceptVerbs("GET", "POST")]
public IActionResult VerifyEmail(string email)
{
    // Vérification...
    return Json(result);
}
```

Ces attributs peuvent être utilisés au niveau du contrôleur pour s'appliquer à toutes les actions :

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProduitsAPIController : ControllerBase
{
    [HttpGet] // GET: /api/ProduitsAPI
    public IActionResult GetAll()
    {
        // ...
    }

    [HttpGet("{id}")] // GET: /api/ProduitsAPI/5
    public IActionResult GetById(int id)
    {
        // ...
    }
}
```

### Action asynchrone

Les actions peuvent être asynchrones, ce qui est recommandé pour les opérations I/O :

```csharp
// Action asynchrone
public async Task<IActionResult> Details(int id)
{
    var client = await _clientService.ObtenirClientParIdAsync(id);
    if (client == null)
    {
        return NotFound();
    }
    return View(client);
}
```

### Contrôleurs d'API

Pour les API Web, ASP.NET propose `ControllerBase` comme classe de base :

```csharp
[ApiController]
[Route("api/[controller]")]
public class ClientsApiController : ControllerBase
{
    private readonly IClientService _clientService;

    public ClientsApiController(IClientService clientService)
    {
        _clientService = clientService;
    }

    // GET: api/ClientsApi
    [HttpGet]
    public ActionResult<IEnumerable<Client>> GetClients()
    {
        return _clientService.ObtenirTousClients().ToList();
    }

    // GET: api/ClientsApi/5
    [HttpGet("{id}")]
    public ActionResult<Client> GetClient(int id)
    {
        var client = _clientService.ObtenirClientParId(id);

        if (client == null)
        {
            return NotFound();
        }

        return client;
    }

    // POST: api/ClientsApi
    [HttpPost]
    public ActionResult<Client> PostClient(Client client)
    {
        _clientService.AjouterClient(client);

        return CreatedAtAction(
            nameof(GetClient),
            new { id = client.Id },
            client
        );
    }

    // PUT: api/ClientsApi/5
    [HttpPut("{id}")]
    public IActionResult PutClient(int id, Client client)
    {
        if (id != client.Id)
        {
            return BadRequest();
        }

        try
        {
            _clientService.MettreAJourClient(client);
        }
        catch (Exception)
        {
            if (!_clientService.ClientExiste(id))
            {
                return NotFound();
            }
            throw;
        }

        return NoContent();
    }

    // DELETE: api/ClientsApi/5
    [HttpDelete("{id}")]
    public IActionResult DeleteClient(int id)
    {
        var client = _clientService.ObtenirClientParId(id);
        if (client == null)
        {
            return NotFound();
        }

        _clientService.SupprimerClient(id);

        return NoContent();
    }
}
```

## 12.2.3. Vues et moteurs de template

Les vues sont responsables de l'affichage des données et de l'interface utilisateur. Dans ASP.NET MVC, elles utilisent principalement le moteur de template Razor.

### Razor : syntaxe de base

Razor est un moteur de template qui permet de mélanger du code C# avec du HTML. Sa syntaxe est conçue pour être concise et intuitive.

#### Syntaxe fondamentale

- **Bloc de code** : `@{ ... }` pour plusieurs lignes de code C#
- **Expression** : `@...` pour afficher le résultat d'une expression
- **Commentaire Razor** : `@* ... *@` pour les commentaires côté serveur

```cshtml
@{
    var message = "Bonjour, monde !";
    var currentDate = DateTime.Now;
}

<h1>@message</h1>
<p>Nous sommes le @currentDate.ToString("dd/MM/yyyy").</p>

@* Ce commentaire ne sera pas envoyé au client *@
<!-- Ce commentaire HTML sera visible dans le code source -->
```

#### Structures de contrôle

Razor permet d'utiliser les structures de contrôle C# directement dans la vue :

```cshtml
@{
    var clients = ViewBag.Clients as List<Client>;
}

@if (clients != null && clients.Any())
{
    <h2>Nous avons @clients.Count clients :</h2>
    <ul>
        @foreach (var client in clients)
        {
            <li>@client.Nom (@client.Email)</li>
        }
    </ul>
}
else
{
    <p>Aucun client trouvé.</p>
}
```

#### Échappement

Par défaut, Razor échappe automatiquement les expressions pour prévenir les attaques XSS :

```cshtml
@{
    var htmlContent = "<strong>Texte en gras</strong>";
}

<p>Échappé: @htmlContent</p> <!-- Affiche: <strong>Texte en gras</strong> -->

<p>Non échappé: @Html.Raw(htmlContent)</p> <!-- Affiche: Texte en gras -->
```

### Organisation des vues

ASP.NET MVC utilise une convention de nommage et d'organisation des vues pour les localiser automatiquement.

#### Structure des dossiers de vues

```
Views/
  ├─ Clients/                       // Pour ClientsController
  │   ├─ Index.cshtml               // Action Index
  │   ├─ Details.cshtml             // Action Details
  │   ├─ Create.cshtml              // Action Create (GET)
  │   ├─ Edit.cshtml                // Action Edit (GET)
  │   ├─ Delete.cshtml              // Action Delete (GET)
  │   └─ _ClientForm.cshtml         // Vue partielle pour formulaire
  ├─ Home/                          // Pour HomeController
  │   ├─ Index.cshtml
  │   ├─ About.cshtml
  │   └─ Contact.cshtml
  ├─ Shared/                        // Vues partagées entre contrôleurs
  │   ├─ _Layout.cshtml             // Template principal
  │   ├─ _ValidationScriptsPartial.cshtml
  │   ├─ Error.cshtml
  │   └─ _LoginPartial.cshtml
  ├─ _ViewStart.cshtml              // Code exécuté avant chaque vue
  └─ _ViewImports.cshtml            // Imports et using communs

#### Convention de nommage

Lorsqu'une action renvoie `View()` sans spécifier de nom, ASP.NET MVC :
1. Cherche une vue portant le même nom que l'action
2. Dans le dossier correspondant au nom du contrôleur

Par exemple :
- L'action `Index()` dans `ClientsController` cherchera `Views/Clients/Index.cshtml`
- L'action `About()` dans `HomeController` cherchera `Views/Home/About.cshtml`

Si vous utilisez `View("Details")`, le moteur cherchera :
1. `Views/[ControllerName]/Details.cshtml`
2. `Views/Shared/Details.cshtml`

### Fichiers spéciaux

#### _ViewStart.cshtml

Ce fichier contient du code qui s'exécute avant chaque vue :

```cshtml
@{
    Layout = "_Layout";
}
```

#### _ViewImports.cshtml

Ce fichier contient les imports et using communs à toutes les vues :

```cshtml
@using MonApplication
@using MonApplication.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

### Layouts (Gabarits)

Les layouts définissent une structure commune pour plusieurs vues, évitant la duplication de code.

#### Structure de base d'un layout

```cshtml
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Mon Application</title>

    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-light bg-white border-bottom">
            <div class="container">
                <a class="navbar-brand" asp-controller="Home" asp-action="Index">Mon Application</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse">
                    <ul class="navbar-nav">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-controller="Home" asp-action="Index">Accueil</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-controller="Clients" asp-action="Index">Clients</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-controller="Home" asp-action="About">À propos</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>

    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; @DateTime.Now.Year - Mon Application
        </div>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @RenderSection("Scripts", required: false)
</body>
</html>
```

#### Utilisation d'un layout

Les vues spécifient généralement leur layout via le fichier `_ViewStart.cshtml`. Mais elles peuvent également le définir explicitement :

```cshtml
@{
    Layout = "_AutreLayout";
}

<h1>Ma vue avec un autre layout</h1>
```

#### Sections

Les sections permettent aux vues de définir du contenu pour des zones spécifiques du layout :

Dans le layout :
```cshtml
<head>
    <!-- ... -->
    @RenderSection("Styles", required: false)
</head>
<body>
    <!-- ... -->
    @RenderSection("Scripts", required: false)
</body>
```

Dans une vue :
```cshtml
@section Styles {
    <link rel="stylesheet" href="~/css/clients.css" />
}

<h1>Liste des clients</h1>
<!-- Contenu principal -->

@section Scripts {
    <script src="~/js/clients.js"></script>
}
```

### Vues partielles

Les vues partielles sont des fragments de vue réutilisables qui permettent de modulariser l'interface utilisateur.

#### Création d'une vue partielle

Les vues partielles sont nommées avec un préfixe underscore (`_`) par convention :

```cshtml
<!-- _ClientCard.cshtml -->
@model Client

<div class="card mb-3">
    <div class="card-header">
        <h5 class="card-title">@Model.Nom</h5>
    </div>
    <div class="card-body">
        <p><strong>Email :</strong> @Model.Email</p>
        <p><strong>Téléphone :</strong> @Model.Telephone</p>
    </div>
    <div class="card-footer">
        <a asp-action="Details" asp-route-id="@Model.Id" class="btn btn-primary btn-sm">Détails</a>
        <a asp-action="Edit" asp-route-id="@Model.Id" class="btn btn-secondary btn-sm">Modifier</a>
    </div>
</div>
```

#### Utilisation des vues partielles

Avec la méthode `Html.Partial` (rendu synchrone) :
```cshtml
@foreach (var client in Model)
{
    @Html.Partial("_ClientCard", client)
}
```

Avec la méthode `Html.RenderPartial` (écrit directement dans la réponse) :
```cshtml
@foreach (var client in Model)
{
    @{ Html.RenderPartial("_ClientCard", client); }
}
```

Avec la balise `<partial>` (ASP.NET Core) :
```cshtml
@foreach (var client in Model)
{
    <partial name="_ClientCard" model="client" />
}
```

#### Vues partielles avec modèle différent

```cshtml
<!-- _ClientFormPartial.cshtml -->
@model ClientViewModel

<div class="form-group">
    <label asp-for="Nom"></label>
    <input asp-for="Nom" class="form-control" />
    <span asp-validation-for="Nom" class="text-danger"></span>
</div>

<div class="form-group">
    <label asp-for="Email"></label>
    <input asp-for="Email" class="form-control" />
    <span asp-validation-for="Email" class="text-danger"></span>
</div>
```

### ViewComponents

ASP.NET Core introduit les ViewComponents, plus puissants que les vues partielles :

```csharp
// Définition du ViewComponent
public class TopClientsViewComponent : ViewComponent
{
    private readonly IClientService _clientService;

    public TopClientsViewComponent(IClientService clientService)
    {
        _clientService = clientService;
    }

    public async Task<IViewComponentResult> InvokeAsync(int nombreClients = 5)
    {
        var clients = await _clientService.GetTopClientsAsync(nombreClients);
        return View(clients); // Cherche /Views/Shared/Components/TopClients/Default.cshtml
    }
}
```

Utilisation dans une vue :
```cshtml
<div class="sidebar">
    <h3>Nos meilleurs clients</h3>
    @await Component.InvokeAsync("TopClients", new { nombreClients = 3 })
</div>
```

Avec Tag Helper (ASP.NET Core) :
```cshtml
<div class="sidebar">
    <h3>Nos meilleurs clients</h3>
    <vc:top-clients nombre-clients="3"></vc:top-clients>
</div>
```

### Passage de données aux vues

Il existe plusieurs façons de passer des données du contrôleur à la vue :

#### 1. Modèle fortement typé

Le plus courant et recommandé :

```csharp
// Contrôleur
public IActionResult Details(int id)
{
    var client = _clientService.ObtenirClientParId(id);
    if (client == null)
    {
        return NotFound();
    }
    return View(client);
}
```

```cshtml
<!-- Vue -->
@model Client
<!DOCTYPE html>
<html>
<body>
    <h1>Détails de @Model.Nom</h1>
    <p>Email : @Model.Email</p>
    <!-- ... -->
</body>
</html>
```

#### 2. ViewData

Dictionary qui persiste uniquement pour la requête actuelle :

```csharp
// Contrôleur
public IActionResult Index()
{
    ViewData["Title"] = "Liste des clients";
    ViewData["Message"] = "Bienvenue sur la page clients";
    ViewData["Clients"] = _clientService.ObtenirTousClients();
    return View();
}
```

```cshtml
<!-- Vue -->
<h1>@ViewData["Title"]</h1>
<p>@ViewData["Message"]</p>

@foreach (var client in (IEnumerable<Client>)ViewData["Clients"])
{
    <p>@client.Nom</p>
}
```

#### 3. ViewBag

Wrapper dynamique autour de ViewData :

```csharp
// Contrôleur
public IActionResult Index()
{
    ViewBag.Title = "Liste des clients";
    ViewBag.Message = "Bienvenue sur la page clients";
    ViewBag.Clients = _clientService.ObtenirTousClients();
    return View();
}
```

```cshtml
<!-- Vue -->
<h1>@ViewBag.Title</h1>
<p>@ViewBag.Message</p>

@foreach (var client in ViewBag.Clients)
{
    <p>@client.Nom</p>
}
```

#### 4. TempData

Similaire à ViewData mais persiste pour une requête supplémentaire (utile pour les redirections) :

```csharp
// Première action
public IActionResult Create(Client client)
{
    if (ModelState.IsValid)
    {
        _clientService.AjouterClient(client);
        TempData["SuccessMessage"] = "Client créé avec succès !";
        return RedirectToAction(nameof(Index));
    }
    return View(client);
}

// Action après redirection
public IActionResult Index()
{
    var clients = _clientService.ObtenirTousClients();
    return View(clients);
}
```

```cshtml
<!-- Vue Index -->
@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success">
        @TempData["SuccessMessage"]
    </div>
}

<h1>Liste des clients</h1>
<!-- ... -->
```

#### 5. ViewModels

Classes spécifiques créées pour une vue particulière, combinant plusieurs modèles ou sous-ensembles de propriétés :

```csharp
// ViewModel
public class ClientDetailViewModel
{
    public Client Client { get; set; }
    public List<Commande> Commandes { get; set; }
    public StatistiquesClient Stats { get; set; }
    public bool EstUtilisateurAdmin { get; set; }
}

// Contrôleur
public IActionResult Details(int id)
{
    var client = _clientService.ObtenirClientParId(id);
    if (client == null)
    {
        return NotFound();
    }

    var viewModel = new ClientDetailViewModel
    {
        Client = client,
        Commandes = _commandeService.ObtenirCommandesClient(id),
        Stats = _statsService.CalculerStatistiquesClient(id),
        EstUtilisateurAdmin = User.IsInRole("Admin")
    };

    return View(viewModel);
}
```

```cshtml
<!-- Vue -->
@model ClientDetailViewModel

<h1>Détails de @Model.Client.Nom</h1>

<!-- Informations client -->
<div class="client-info">
    <p>Email : @Model.Client.Email</p>
    <!-- ... -->
</div>

<!-- Statistiques -->
<div class="stats">
    <h3>Statistiques</h3>
    <p>Total des achats : @Model.Stats.TotalAchats.ToString("C")</p>
    <p>Fréquence d'achat : @Model.Stats.FrequenceAchat jours</p>
</div>

@if (Model.Commandes.Any())
{
    <h3>Commandes récentes</h3>
    <table class="table">
        <!-- ... -->
    </table>
}

@if (Model.EstUtilisateurAdmin)
{
    <div class="admin-actions">
        <a asp-action="Delete" asp-route-id="@Model.Client.Id" class="btn btn-danger">Supprimer</a>
        <!-- Autres actions admin -->
    </div>
}
```

### Tag Helpers

ASP.NET Core introduit les Tag Helpers qui simplifient la création d'éléments HTML liés au serveur :

```cshtml
<!-- Formulaire qui pointe vers l'action Create -->
<form asp-controller="Clients" asp-action="Create" method="post">

    <!-- Champ de saisie lié à la propriété Nom du modèle -->
    <div class="form-group">
        <label asp-for="Nom"></label>
        <input asp-for="Nom" class="form-control" />
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <!-- Champ de saisie lié à la propriété Email du modèle -->
    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <!-- Liste déroulante liée à la propriété CategorieId -->
    <div class="form-group">
        <label asp-for="CategorieId"></label>
        <select asp-for="CategorieId" class="form-control"
                asp-items="@(new SelectList(ViewBag.Categories, "Id", "Nom"))">
            <option value="">-- Sélectionner une catégorie --</option>
        </select>
        <span asp-validation-for="CategorieId" class="text-danger"></span>
    </div>

    <!-- Boutons du formulaire -->
    <div class="form-group">
        <button type="submit" class="btn btn-primary">Enregistrer</button>
        <a asp-action="Index" class="btn btn-secondary">Annuler</a>
    </div>
</form>
```

### HtmlHelpers vs. Tag Helpers

Avant ASP.NET Core, les HtmlHelpers étaient utilisés pour générer du HTML :

```cshtml
<!-- HtmlHelper (ASP.NET MVC classique) -->
@using (Html.BeginForm("Create", "Clients", FormMethod.Post))
{
    @Html.AntiForgeryToken()

    <div class="form-group">
        @Html.LabelFor(m => m.Nom)
        @Html.TextBoxFor(m => m.Nom, new { @class = "form-control" })
        @Html.ValidationMessageFor(m => m.Nom, "", new { @class = "text-danger" })
    </div>

    <div class="form-group">
        @Html.LabelFor(m => m.Email)
        @Html.TextBoxFor(m => m.Email, new { @class = "form-control" })
        @Html.ValidationMessageFor(m => m.Email, "", new { @class = "text-danger" })
    </div>

    <div class="form-group">
        <input type="submit" value="Enregistrer" class="btn btn-primary" />
        @Html.ActionLink("Annuler", "Index", null, new { @class = "btn btn-secondary" })
    </div>
}

<!-- Tag Helper équivalent (ASP.NET Core) -->
<form asp-controller="Clients" asp-action="Create" method="post">

    <div class="form-group">
        <label asp-for="Nom"></label>
        <input asp-for="Nom" class="form-control" />
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="form-group">
        <button type="submit" class="btn btn-primary">Enregistrer</button>
        <a asp-action="Index" class="btn btn-secondary">Annuler</a>
    </div>
</form>
```

## 12.2.4. Modèles et validation

Les modèles représentent les données et la logique métier de l'application. ASP.NET MVC fournit un puissant système de validation pour garantir l'intégrité des données.

### Création de modèles

Les modèles sont généralement de simples classes C# (POCO - Plain Old CLR Objects) :

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace MonApplication.Models
{
    public class Client
    {
        public int Id { get; set; }

        [Required(ErrorMessage = "Le nom est obligatoire")]
        [StringLength(100, ErrorMessage = "Le nom ne peut pas dépasser {1} caractères")]
        [Display(Name = "Nom complet")]
        public string Nom { get; set; }

        [Required(ErrorMessage = "L'email est obligatoire")]
        [EmailAddress(ErrorMessage = "Format d'email invalide")]
        public string Email { get; set; }

        [Phone(ErrorMessage = "Format de téléphone invalide")]
        [Display(Name = "Numéro de téléphone")]
        public string Telephone { get; set; }

        [Display(Name = "Date d'inscription")]
        [DataType(DataType.Date)]
        public DateTime DateInscription { get; set; }

        [Range(0, 100000, ErrorMessage = "Le crédit doit être entre {1} et {2}")]
        [DisplayFormat(DataFormatString = "{0:C}", ApplyFormatInEditMode = false)]
        public decimal Credit { get; set; }

        [Display(Name = "Client actif")]
        public bool Actif { get; set; }

        // Propriété de navigation pour une relation
        public int? CategorieId { get; set; }
        public virtual Categorie Categorie { get; set; }

        // Collection pour une relation one-to-many
        public virtual ICollection<Commande> Commandes { get; set; }
    }
}
```

### Types de modèles

#### Modèles de domaine

Représentent les entités métier de l'application :

```csharp
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Description { get; set; }
    public decimal Prix { get; set; }
    public int Stock { get; set; }
    public int CategorieId { get; set; }

    // Relations
    public virtual Categorie Categorie { get; set; }
    public virtual ICollection<LigneCommande> LignesCommande { get; set; }

    // Méthodes métier
    public bool EstEnStock()
    {
        return Stock > 0;
    }

    public decimal CalculerPrixTTC(decimal tauxTVA)
    {
        return Prix * (1 + tauxTVA / 100);
    }
}
```

#### ViewModels

Modèles spécifiques à une vue, combinant plusieurs modèles ou ajoutant des propriétés supplémentaires :

```csharp
public class ProduitViewModel
{
    // Données produit
    public int Id { get; set; }

    [Required(ErrorMessage = "Le nom est obligatoire")]
    public string Nom { get; set; }

    [DataType(DataType.MultilineText)]
    public string Description { get; set; }

    [Required]
    [Range(0.01, 10000)]
    [DisplayFormat(DataFormatString = "{0:C}")]
    public decimal Prix { get; set; }

    // Liste des catégories pour le dropdown
    public List<SelectListItem> CategoriesDisponibles { get; set; }

    [Display(Name = "Catégorie")]
    [Required(ErrorMessage = "La catégorie est obligatoire")]
    public int CategorieId { get; set; }

    // Fichiers
    [Display(Name = "Image du produit")]
    public IFormFile ImageFile { get; set; }

    // Propriétés calculées
    public bool EstEnPromotion { get; set; }
    public decimal PrixPromotion { get; set; }

    // Propriétés pour l'interface utilisateur
    public bool ShowDeleteButton { get; set; }
}
```

#### DTOs (Data Transfer Objects)

Utilisés pour transférer des données entre les couches de l'application :

```csharp
public class ProduitDto
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public decimal Prix { get; set; }
    public string CategorieName { get; set; }
}
```

### Attributs de validation

ASP.NET MVC utilise des attributs pour la validation des modèles. Ces attributs sont définis dans l'espace de noms `System.ComponentModel.DataAnnotations`.

#### Attributs de validation courants

| Attribut | Description | Exemple |
|----------|-------------|---------|
| `[Required]` | Champ obligatoire | `[Required(ErrorMessage = "Le nom est obligatoire")]` |
| `[StringLength]` | Limite la longueur d'une chaîne | `[StringLength(100, MinimumLength = 3)]` |
| `[Range]` | Limite les valeurs numériques | `[Range(0, 9999.99)]` |
| `[RegularExpression]` | Valide avec une expression régulière | `[RegularExpression(@"^[A-Z]{2}\d{3}$")]` |
| `[EmailAddress]` | Vérifie le format de l'email | `[EmailAddress]` |
| `[Phone]` | Vérifie le format du téléphone | `[Phone]` |
| `[Url]` | Vérifie le format de l'URL | `[Url]` |
| `[CreditCard]` | Vérifie le format de la carte de crédit | `[CreditCard]` |
| `[Compare]` | Compare avec une autre propriété | `[Compare("Password")]` |
| `[Remote]` | Validation côté serveur via AJAX | `[Remote("VerifierEmail", "Clients")]` |

#### Attributs d'affichage

| Attribut | Description | Exemple |
|----------|-------------|---------|
| `[Display]` | Définit le nom d'affichage | `[Display(Name = "Date de naissance")]` |
| `[DisplayFormat]` | Définit le format d'affichage | `[DisplayFormat(DataFormatString = "{0:dd/MM/yyyy}")]` |
| `[DataType]` | Spécifie le type de données | `[DataType(DataType.Date)]` |
| `[ScaffoldColumn]` | Contrôle si le champ est généré | `[ScaffoldColumn(false)]` |
| `[HiddenInput]` | Rend le champ comme input hidden | `[HiddenInput]` |
| `[ReadOnly]` | Indique que le champ est en lecture seule | `[ReadOnly(true)]` |

### Validation côté client

ASP.NET MVC inclut un support pour la validation côté client basée sur jQuery Validate et jQuery Unobtrusive Validation.

#### Configuration dans le layout

```cshtml
<head>
    <!-- ... -->
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
</head>
```

#### Utilisation dans un formulaire

```cshtml
<form asp-action="Create">
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

    <div class="form-group">
        <label asp-for="Nom"></label>
        <input asp-for="Nom" class="form-control" />
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="form-group">
        <button type="submit" class="btn btn-primary">Enregistrer</button>
    </div>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

#### Validation côté client pour des scénarios avancés

```javascript
// Validation personnalisée pour un champ spécifique
$(document).ready(function () {
    $.validator.addMethod("commenceParA", function (value, element) {
        return this.optional(element) || /^A/.test(value);
    }, "Le champ doit commencer par la lettre A");

    $.validator.unobtrusive.adapters.add("commenceParA", function (options) {
        options.rules["commenceParA"] = true;
        options.messages["commenceParA"] = options.message;
    });
});
```

```csharp
// Attribut personnalisé correspondant
public class CommenceParAAttribute : ValidationAttribute, IClientValidatable
{
    public CommenceParAAttribute()
        : base("Le champ {0} doit commencer par la lettre A")
    {
    }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value == null || !(value is string))
            return ValidationResult.Success;

        var str = (string)value;
        if (str.StartsWith("A"))
            return ValidationResult.Success;

        return new ValidationResult(FormatErrorMessage(validationContext.DisplayName));
    }

    public IEnumerable<ModelClientValidationRule> GetClientValidationRules(ModelMetadata metadata, ControllerContext context)
    {
        var rule = new ModelClientValidationRule
        {
            ErrorMessage = FormatErrorMessage(metadata.DisplayName),
            ValidationType = "commencepar"
        };

        yield return rule;
    }
}
```

### Validation côté serveur

La validation côté client peut être contournée, il est donc essentiel de toujours valider les données côté serveur.

#### Vérification de ModelState

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Client client)
{
    // Validation manuelle supplémentaire si nécessaire
    if (string.IsNullOrEmpty(client.Email) && string.IsNullOrEmpty(client.Telephone))
    {
        ModelState.AddModelError("", "Vous devez fournir au moins un email ou un téléphone.");
    }

    // Vérification si le modèle est valide (attributs + validations manuelles)
    if (ModelState.IsValid)
    {
        try
        {
            _clientService.AjouterClient(client);
            return RedirectToAction(nameof(Index));
        }
        catch (Exception ex)
        {
            // Ajouter une erreur au ModelState
            ModelState.AddModelError("", $"Erreur lors de la création: {ex.Message}");
        }
    }

    // Si le modèle n'est pas valide, réafficher le formulaire
    return View(client);
}
```

#### Affichage des erreurs de validation

```cshtml
<form asp-action="Create">
    <!-- Résumé des erreurs globales (non liées à un champ spécifique) -->
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

    <!-- Ou tous les messages d'erreur -->
    <div asp-validation-summary="All" class="text-danger"></div>

    <!-- Champs du formulaire avec leurs messages d'erreur spécifiques -->
    <div class="form-group">
        <label asp-for="Nom"></label>
        <input asp-for="Nom" class="form-control" />
        <span asp-validation-for="Nom" class="text-danger"></span>
    </div>

    <!-- ... -->
</form>
```

### Validation personnalisée

#### Attribut de validation personnalisé

```csharp
// Attribut pour vérifier qu'une date est dans le passé
public class DatePasseeAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value == null)
            return ValidationResult.Success;

        DateTime dateValue;
        if (value is DateTime)
            dateValue = (DateTime)value;
        else if (DateTime.TryParse(value.ToString(), out dateValue) == false)
            return new ValidationResult("Format de date invalide");

        if (dateValue > DateTime.Now)
            return new ValidationResult(ErrorMessage ?? "La date doit être dans le passé");

        return ValidationResult.Success;
    }
}
```

Utilisation :
```csharp
public class Personne
{
    [Required]
    public string Nom { get; set; }

    [Required]
    [DatePassee(ErrorMessage = "La date de naissance doit être dans le passé")]
    [Display(Name = "Date de naissance")]
    [DataType(DataType.Date)]
    public DateTime DateNaissance { get; set; }
}
```

#### Implémentation de IValidatableObject

Une autre approche consiste à implémenter l'interface `IValidatableObject` dans la classe du modèle :

```csharp
public class Reservation : IValidatableObject
{
    public int Id { get; set; }

    [Required]
    [Display(Name = "Date de début")]
    [DataType(DataType.Date)]
    public DateTime DateDebut { get; set; }

    [Required]
    [Display(Name = "Date de fin")]
    [DataType(DataType.Date)]
    public DateTime DateFin { get; set; }

    [Required]
    [Range(1, 10)]
    [Display(Name = "Nombre de personnes")]
    public int NombrePersonnes { get; set; }

    // Méthode de validation personnalisée
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        // Vérifier que la date de fin est après la date de début
        if (DateFin <= DateDebut)
        {
            yield return new ValidationResult(
                "La date de fin doit être postérieure à la date de début",
                new[] { nameof(DateFin), nameof(DateDebut) });
        }

        // Vérifier que la réservation ne dépasse pas 30 jours
        if ((DateFin - DateDebut).TotalDays > 30)
        {
            yield return new ValidationResult(
                "La durée maximum d'une réservation est de 30 jours",
                new[] { nameof(DateFin) });
        }

        // Vérifier des conditions métier complexes
        var reservationService = (IReservationService)validationContext
            .GetService(typeof(IReservationService));

        if (reservationService.EstPerioDeChargee(DateDebut, DateFin) && NombrePersonnes > 5)
        {
            yield return new ValidationResult(
                "Les groupes de plus de 5 personnes ne sont pas acceptés pendant les périodes chargées",
                new[] { nameof(NombrePersonnes) });
        }
    }
}
```

#### Validation à distance (Remote Validation)

La validation à distance permet de vérifier des données sur le serveur via AJAX sans soumettre le formulaire entier :

```csharp
// Dans le modèle
public class Client
{
    // ...

    [Required]
    [EmailAddress]
    [Remote(action: "VerifierEmailDisponible", controller: "Clients", AdditionalFields = "Id")]
    public string Email { get; set; }

    // ...
}

// Dans le contrôleur
[AcceptVerbs("GET", "POST")]
public IActionResult VerifierEmailDisponible(string email, int? id)
{
    // Vérifier si l'email existe déjà, en excluant le client actuel en cas d'édition
    var emailExiste = _clientService.EmailExiste(email, id);

    if (emailExiste)
    {
        return Json($"L'email '{email}' est déjà utilisé.");
    }

    return Json(true); // Validation réussie
}
```

## 12.2.5. Routing

Le routing est le processus qui permet de faire correspondre des URLs à des actions de contrôleur. Il est essentiel pour créer des URLs lisibles et SEO-friendly.

### Configuration du routing

#### Routing conventionnel

Dans ASP.NET Core MVC, le routing conventionnel est configuré dans la méthode `Configure` de la classe `Startup` ou dans la méthode `Main` de `Program.cs` (pour .NET 6+) :

```csharp
// Pour ASP.NET Core 5 et antérieur (Startup.cs)
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}

// Pour ASP.NET Core 6+ (Program.cs)
var app = builder.Build();
// ...
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

Ce code définit une route par défaut qui se décompose en trois segments :
- `{controller}` : Nom du contrôleur (sans le suffixe "Controller")
- `{action}` : Nom de la méthode d'action
- `{id?}` : Paramètre optionnel

Par exemple, l'URL `/Clients/Details/5` sera mappée à :
- Contrôleur : `ClientsController`
- Action : `Details`
- id : `5`

#### Routes multiples

Vous pouvez définir plusieurs routes dans l'ordre de priorité (de la plus spécifique à la plus générale) :

```csharp
app.UseEndpoints(endpoints =>
{
    // Route spécifique pour les articles de blog
    endpoints.MapControllerRoute(
        name: "blog",
        pattern: "blog/{year}/{month}/{slug}",
        defaults: new { controller = "Blog", action = "Article" },
        constraints: new { year = @"\d{4}", month = @"\d{2}" });

    // Route pour les pages statiques
    endpoints.MapControllerRoute(
        name: "page",
        pattern: "page/{slug}",
        defaults: new { controller = "Pages", action = "Show" });

    // Route par défaut
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});
```

#### Contraintes de route

Les contraintes permettent de spécifier des critères pour les segments de route :

```csharp
// Route avec contraintes
endpoints.MapControllerRoute(
    name: "produit",
    pattern: "catalogue/{categorie}/{id:int}/{slug?}",
    defaults: new { controller = "Produits", action = "Details" },
    constraints: new {
        categorie = new RegexRouteConstraint(@"^[a-z0-9\-]+$"),
        id = new IntRouteConstraint()
    });
```

Contraintes intégrées :
- `int` : Entier
- `bool` : Booléen
- `datetime` : Date et heure
- `decimal` : Nombre décimal
- `guid` : GUID
- `float` : Nombre à virgule flottante
- `minlength` : Longueur minimale
- `maxlength` : Longueur maximale
- `length` : Longueur spécifique
- `min` : Valeur minimale
- `max` : Valeur maximale
- `range` : Plage de valeurs
- `alpha` : Caractères alphabétiques
- `regex` : Expression régulière
- `required` : Segment obligatoire

### Attributs de routing

En plus du routing conventionnel, vous pouvez utiliser des attributs pour définir des routes directement sur les contrôleurs et les actions.

#### Attributs au niveau du contrôleur

```csharp
[Route("api/[controller]")]
public class ProduitsController : ControllerBase
{
    // Routes définies au niveau des actions
}
```

#### Attributs au niveau de l'action

```csharp
[Route("api/[controller]")]
public class ProduitsController : ControllerBase
{
    // GET: api/produits
    [HttpGet]
    public IActionResult GetAll()
    {
        // ...
    }

    // GET: api/produits/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        // ...
    }

    // POST: api/produits
    [HttpPost]
    public IActionResult Create([FromBody] Produit produit)
    {
        // ...
    }

    // PUT: api/produits/5
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Produit produit)
    {
        // ...
    }

    // DELETE: api/produits/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        // ...
    }

    // GET: api/produits/categories/3/produits
    [HttpGet("categories/{categorieId}/produits")]
    public IActionResult GetByCategorieId(int categorieId)
    {
        // ...
    }
}
```

#### Combinaison d'attributs

```csharp
[Route("api/v{version:int}/[controller]")]
public class ClientsController : ControllerBase
{
    // GET: api/v1/clients/actifs
    [HttpGet("actifs")]
    public IActionResult GetActifs()
    {
        // ...
    }

    // GET: api/v1/clients/inactifs
    [HttpGet("inactifs")]
    public IActionResult GetInactifs()
    {
        // ...
    }

    // GET: api/v1/clients/5/commandes
    [HttpGet("{id}/commandes")]
    public IActionResult GetCommandesClient(int id)
    {
        // ...
    }
}
```

### Génération d'URL

ASP.NET MVC fournit plusieurs méthodes pour générer des URLs basées sur vos routes.

#### Génération d'URL dans les vues avec des Tag Helpers

```cshtml
<!-- Lien vers une action du même contrôleur -->
<a asp-action="Details" asp-route-id="5">Voir le client</a>

<!-- Lien vers une action d'un autre contrôleur -->
<a asp-controller="Produits" asp-action="Index">Liste des produits</a>

<!-- Lien avec plusieurs paramètres de route -->
<a asp-controller="Blog" asp-action="Article"
   asp-route-year="2023" asp-route-month="05" asp-route-slug="intro-aspnet">
    Article sur ASP.NET
</a>

<!-- Lien utilisant une route nommée -->
<a asp-route="blog" asp-route-year="2023" asp-route-month="05" asp-route-slug="intro-aspnet">
    Article sur ASP.NET
</a>

<!-- Formulaire pointant vers une action -->
<form asp-controller="Clients" asp-action="Create" method="post">
    <!-- ... -->
</form>
```

#### Génération d'URL dans les contrôleurs

```csharp
// Générer une URL pour une action
var url = Url.Action("Details", "Clients", new { id = 5 });

// Générer une URL pour une route nommée
var url = Url.RouteUrl("blog", new { year = 2023, month = "05", slug = "intro-aspnet" });

// Générer une URL absolue
var url = Url.Action("Index", "Home", null, Request.Scheme);
// Résultat: https://example.com/Home/Index

// Redirection vers une URL générée
return Redirect(Url.Action("Details", "Clients", new { id = 5 }));

// Ou plus simplement
return RedirectToAction("Details", "Clients", new { id = 5 });
```

### Areas

Les areas permettent de diviser une grande application en sections plus petites et plus gérables.

#### Structure d'une area

```
Areas/
  ├─ Admin/
  │   ├─ Controllers/
  │   │   ├─ DashboardController.cs
  │   │   └─ UsersController.cs
  │   ├─ Models/
  │   │   └─ AdminViewModels.cs
  │   ├─ Views/
  │   │   ├─ Dashboard/
  │   │   │   └─ Index.cshtml
  │   │   ├─ Users/
  │   │   │   ├─ Index.cshtml
  │   │   │   └─ Edit.cshtml
  │   │   └─ _ViewImports.cshtml
  │   └─ AdminAreaRegistration.cs
  │
  └─ Client/
      ├─ Controllers/
      ├─ Models/
      └─ Views/
```

#### Configuration des areas

```csharp
// Enregistrement des routes d'area
app.UseEndpoints(endpoints =>
{
    // Route pour les areas
    endpoints.MapControllerRoute(
        name: "areas",
        pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

    // Route par défaut
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});
```

#### Définition d'un contrôleur dans une area

```csharp
[Area("Admin")]
public class DashboardController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult Stats()
    {
        return View();
    }
}
```

#### Liens vers des actions dans une area

```cshtml
<!-- Lien vers une action dans la même area -->
<a asp-action="Stats">Statistiques</a>

<!-- Lien vers une action dans une area spécifique -->
<a asp-area="Admin" asp-controller="Dashboard" asp-action="Index">Tableau de bord</a>

<!-- Lien hors de toute area -->
<a asp-area="" asp-controller="Home" asp-action="Index">Accueil</a>
```

## 12.2.6. Filtres et attributs

Les filtres permettent d'ajouter des comportements transversaux avant ou après l'exécution des actions. Ils sont implémentés sous forme d'attributs.

### Types de filtres

ASP.NET MVC définit plusieurs types de filtres qui s'exécutent à différents moments du pipeline de traitement des requêtes :

1. **Filtres d'autorisation** : Exécutés en premier, contrôlent l'accès aux actions
2. **Filtres de ressource** : Exécutés avant et après tous les autres filtres
3. **Filtres d'action** : Exécutés avant et après l'action
4. **Filtres de résultat** : Exécutés avant et après l'exécution du résultat (vue)
5. **Filtres d'exception** : Gèrent les exceptions non traitées

### Filtres d'autorisation

Les filtres d'autorisation contrôlent si l'utilisateur a le droit d'accéder à une action.

#### [Authorize]

```csharp
// Nécessite un utilisateur authentifié
[Authorize]
public class AdminController : Controller
{
    // Toutes les actions nécessitent un utilisateur authentifié
}

// Nécessite un utilisateur avec un rôle spécifique
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser(int id)
{
    // ...
}

// Nécessite un utilisateur avec une revendication spécifique
[Authorize(Policy = "EditProducts")]
public IActionResult Edit(int id)
{
    // ...
}

// Nécessite un utilisateur authentifié avec un schéma spécifique
[Authorize(AuthenticationSchemes = "Bearer")]
public IActionResult GetUserInfo()
{
    // ...
}
```

#### [AllowAnonymous]

```csharp
[Authorize] // Contrôleur nécessitant un utilisateur authentifié
public class AccountController : Controller
{
    // Cette action peut être accédée sans authentification
    [AllowAnonymous]
    public IActionResult Login()
    {
        return View();
    }

    // Cette action nécessite un utilisateur authentifié
    public IActionResult Profile()
    {
        return View();
    }
}
```

#### Filtre d'autorisation personnalisé

```csharp
public class MinimumAgeAttribute : AuthorizeAttribute, IAuthorizationFilter
{
    private int _minimumAge;

    public MinimumAgeAttribute(int minimumAge)
    {
        _minimumAge = minimumAge;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        // Vérifier si l'utilisateur est authentifié
        if (!context.HttpContext.User.Identity.IsAuthenticated)
        {
            context.Result = new ChallengeResult();
            return;
        }

        // Récupérer la date de naissance de l'utilisateur
        var dateNaissanceClaim = context.HttpContext.User.FindFirst("DateNaissance");
        if (dateNaissanceClaim == null)
        {
            context.Result = new ForbidResult();
            return;
        }

        // Calculer l'âge
        var dateNaissance = DateTime.Parse(dateNaissanceClaim.Value);
        var age = DateTime.Today.Year - dateNaissance.Year;
        if (dateNaissance.Date > DateTime.Today.AddYears(-age))
            age--;

        // Vérifier l'âge minimum
        if (age < _minimumAge)
        {
            context.Result = new ForbidResult();
        }
    }
}
```

Utilisation :
```csharp
[MinimumAge(18)]
public IActionResult ContenuAdulte()
{
    return View();
}
```

### Filtres d'action

Les filtres d'action s'exécutent avant et après l'exécution de l'action.

#### Implémentation d'un filtre d'action

```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Code exécuté avant l'action
        _logger.LogInformation($"Exécution de l'action {context.ActionDescriptor.DisplayName}");

        // Vérifier les arguments
        foreach (var kvp in context.ActionArguments)
        {
            _logger.LogInformation($"- Paramètre: {kvp.Key}, Valeur: {kvp.Value}");
        }
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        // Code exécuté après l'action
        _logger.LogInformation($"Action {context.ActionDescriptor.DisplayName} exécutée");

        if (context.Exception != null)
        {
            _logger.LogError($"Exception: {context.Exception.Message}");
        }
    }
}
```

#### Enregistrement d'un filtre d'action nécessitant des services

```csharp
// Dans Startup.ConfigureServices ou Program.cs
services.AddScoped<LogActionFilter>();

// Utilisation avec injection de dépendances
[ServiceFilter(typeof(LogActionFilter))]
public IActionResult Index()
{
    return View();
}
```

#### Attribut de filtre d'action

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class AuditAttribute : ActionFilterAttribute
{
    public string Categorie { get; set; }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        var utilisateur = context.HttpContext.User.Identity.Name ?? "Anonyme";
        var controllerName = context.ActionDescriptor.RouteValues["controller"];
        var actionName = context.ActionDescriptor.RouteValues["action"];

        var auditService = context.HttpContext.RequestServices.GetService<IAuditService>();

        auditService.EnregistrerAction(
            utilisateur,
            $"{controllerName}.{actionName}",
            Categorie,
            DateTime.Now);
    }
}

// Utilisation
[Audit(Categorie = "Administration")]
public IActionResult SupprimerClient(int id)
{
    // ...
}
```

### Filtres de résultat

Les filtres de résultat s'exécutent avant et après l'exécution du résultat (typiquement le rendu d'une vue).

```csharp
public class CompressResponseAttribute : ResultFilterAttribute
{
    public override void OnResultExecuting(ResultExecutingContext context)
    {
        // Vérifier si le résultat est une vue
        if (context.Result is ViewResult)
        {
            // Configurer la compression
            var response = context.HttpContext.Response;
            response.Headers.Add("Content-Encoding", "gzip");

            // Remplacer le flux de réponse par un flux compressé
            var originalStream = response.Body;
            var compressedStream = new GZipStream(originalStream, CompressionMode.Compress);
            response.Body = compressedStream;
        }
    }

    public override void OnResultExecuted(ResultExecutedContext context)
    {
        // Nettoyer les ressources si nécessaire
        if (context.Result is ViewResult)
        {
            context.HttpContext.Response.Body.Dispose();
        }
    }
}
```

### Filtres d'exception

Les filtres d'exception permettent de gérer les exceptions non traitées dans les actions.

```csharp
public class CustomExceptionFilterAttribute : ExceptionFilterAttribute
{
    private readonly ILogger _logger;

    public CustomExceptionFilterAttribute(ILogger<CustomExceptionFilterAttribute> logger)
    {
        _logger = logger;
    }

    public override void OnException(ExceptionContext context)
    {
        // Journaliser l'exception
        _logger.LogError(context.Exception, "Une exception s'est produite");

        // Créer un ViewModel pour l'erreur
        var model = new ErrorViewModel
        {
            RequestId = Activity.Current?.Id ?? context.HttpContext.TraceIdentifier,
            Message = context.Exception.Message,
            StackTrace = context.Exception.StackTrace
        };

        // Renvoyer une vue d'erreur
        context.Result = new ViewResult
        {
            ViewName = "Error",
            ViewData = new ViewDataDictionary<ErrorViewModel>(
                new EmptyModelMetadataProvider(),
                new ModelStateDictionary())
            {
                Model = model
            }
        };

        // Marquer l'exception comme traitée
        context.ExceptionHandled = true;
    }
}
```

### Autres attributs courants

#### [ValidateAntiForgeryToken]

Protège contre les attaques CSRF (Cross-Site Request Forgery) :

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Client client)
{
    // ...
}
```

Avec le tag helper correspondant dans la vue :

```cshtml
<form asp-action="Create">
    @Html.AntiForgeryToken()
    <!-- ou automatiquement avec les tag helpers -->

    <!-- ... -->
</form>
```

#### [ResponseCache]

Configure la mise en cache des réponses :

```csharp
// Désactiver le cache
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public IActionResult Error()
{
    return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
}

// Activer le cache pour 60 secondes
[ResponseCache(Duration = 60)]
public IActionResult Index()
{
    return View();
}

// Cache plus avancé avec variations
[ResponseCache(Duration = 3600, VaryByQueryKeys = new[] { "id" })]
public IActionResult Details(int id)
{
    // ...
}
```

#### [RequireHttps]

Force l'utilisation de HTTPS :

```csharp
[RequireHttps]
public class SecureController : Controller
{
    // Toutes les actions seront accessibles uniquement en HTTPS
}
```

### Ordre d'exécution des filtres

Les filtres s'exécutent dans un ordre spécifique :

1. Filtres d'autorisation
2. Filtres de ressource (avant)
3. Filtres d'action (avant)
4. Filtres de résultat (avant)
5. Exécution de l'action
6. Filtres de résultat (après)
7. Filtres d'action (après)
8. Filtres de ressource (après)
9. Filtres d'exception (si une exception est levée)

Lorsque plusieurs filtres du même type sont présents, ils s'exécutent selon leur portée :
1. Filtres globaux (définis au niveau de l'application)
2. Filtres de contrôleur (définis au niveau de la classe)
3. Filtres d'action (définis au niveau de la méthode)

Dans chaque niveau, vous pouvez définir l'ordre avec la propriété `Order` :

```csharp
[TypeFilter(typeof(LogActionFilter), Order = 1)]
[ServiceFilter(typeof(AuditFilter), Order = 2)]
public IActionResult Index()
{
    return View();
}
```

### Enregistrement de filtres globaux

Vous pouvez enregistrer des filtres pour toutes les actions dans `Startup.cs` ou `Program.cs` :

```csharp
// Pour ASP.NET Core
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews(options =>
    {
        options.Filters.Add(new RequireHttpsAttribute());
        options.Filters.Add(typeof(LogActionFilter)); // Nécessite l'injection de dépendances
        options.Filters.Add(new CustomExceptionFilterAttribute());
    });
}
```

### Filtres d'action asynchrones

Vous pouvez également créer des filtres asynchrones :

```csharp
public class AsyncLogActionFilter : IAsyncActionFilter
{
    private readonly ILogger _logger;

    public AsyncLogActionFilter(ILogger<AsyncLogActionFilter> logger)
    {
        _logger = logger;
    }

    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        // Avant l'exécution de l'action
        _logger.LogInformation($"Exécution de l'action {context.ActionDescriptor.DisplayName}");

        // Exécuter l'action
        var resultContext = await next();

        // Après l'exécution de l'action
        if (resultContext.Exception == null)
        {
            _logger.LogInformation($"Action {context.ActionDescriptor.DisplayName} exécutée avec succès");
        }
        else
        {
            _logger.LogError($"Action {context.ActionDescriptor.DisplayName} a rencontré une exception: {resultContext.Exception.Message}");
        }
    }
}
```

## Conclusion

ASP.NET MVC est un framework puissant et flexible pour le développement d'applications web. Il offre une séparation claire des préoccupations grâce à son architecture Model-View-Controller, ce qui rend les applications plus maintenables et testables.

Dans ce chapitre, nous avons exploré :

1. **L'architecture MVC** - comment les modèles, vues et contrôleurs interagissent
2. **Les contrôleurs et actions** - le cœur du framework qui traite les requêtes HTTP
3. **Les vues et moteurs de template** - avec Razor pour générer l'interface utilisateur
4. **Les modèles et validation** - pour assurer l'intégrité des données
5. **Le routing** - pour créer des URLs propres et SEO-friendly
6. **Les filtres et attributs** - pour ajouter des comportements transversaux

Ces concepts fondamentaux vous donnent une base solide pour développer des applications web modernes avec ASP.NET MVC.

### Meilleures pratiques

- **Gardez les contrôleurs légers** : Déplacez la logique métier dans des services
- **Utilisez des ViewModels** : Plutôt que de passer des modèles de domaine directement aux vues
- **Validez toujours côté serveur** : La validation côté client peut être contournée
- **Protégez-vous contre le CSRF** : Utilisez `[ValidateAntiForgeryToken]` pour les formulaires
- **Utilisez des tag helpers (ASP.NET Core)** : Pour un code plus lisible et maintenable
- **Créez des composants réutilisables** : Avec des vues partielles ou des ViewComponents
- **Tests unitaires** : Les actions de contrôleur sont faciles à tester grâce à la séparation MVC

ASP.NET MVC continue d'évoluer avec chaque nouvelle version, en particulier avec ASP.NET Core, offrant des performances améliorées, une meilleure modularité et une prise en charge multiplateforme.

⏭️ 12.3. [ASP.NET Core](/12-developpement-web-avec-asp-dotnet/12-3-asp-dotnet-core.md)
