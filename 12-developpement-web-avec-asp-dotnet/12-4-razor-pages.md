# 12.4. Razor Pages

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 12.4.1. Modèle de programmation

Razor Pages est un modèle de programmation introduit dans ASP.NET Core 2.0 qui simplifie le développement d'applications web en combinant le code et la présentation dans un format orienté page. Cette approche est particulièrement adaptée aux débutants et aux pages qui ne nécessitent pas la complexité complète du modèle MVC.

### Introduction à Razor Pages

Razor Pages adopte un modèle de développement basé sur des pages, où chaque page est autonome et contient à la fois sa logique et son interface utilisateur. Cette approche est plus intuitive pour les développeurs débutants et se rapproche davantage du modèle de développement web classique.

#### Structure d'une application Razor Pages

Une application Razor Pages typique présente la structure suivante :

```
MyRazorApp/
  ├─ Pages/                    // Répertoire contenant toutes les Razor Pages
  │   ├─ _ViewStart.cshtml     // Configuration commune à toutes les pages
  │   ├─ _ViewImports.cshtml   // Imports, taghelpers et namespaces communs
  │   ├─ Shared/              // Composants partagés (layouts, vues partielles)
  │   │   └─ _Layout.cshtml    // Template principal
  │   ├─ Index.cshtml          // Page d'accueil
  │   ├─ Index.cshtml.cs       // Classe "code-behind" pour la page d'accueil
  │   ├─ Privacy.cshtml        // Page de confidentialité
  │   └─ Privacy.cshtml.cs     // Classe "code-behind"
  ├─ wwwroot/                  // Fichiers statiques (CSS, JS, images)
  ├─ appsettings.json          // Configuration
  └─ Program.cs                // Point d'entrée et configuration de l'application
```

#### Anatomie d'une Razor Page

Chaque Razor Page se compose de deux fichiers :

1. **Fichier .cshtml** : Contient le balisage HTML et le code Razor
2. **Fichier .cshtml.cs** : Contient la logique de la page (appelé "modèle de page" ou "PageModel")

##### Exemple de fichier .cshtml (Contact.cshtml)

```cshtml
@page
@model ContactModel

<h2>Contactez-nous</h2>

<form method="post">
    <div class="form-group">
        <label asp-for="Contact.Name"></label>
        <input asp-for="Contact.Name" class="form-control" />
        <span asp-validation-for="Contact.Name" class="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Contact.Email"></label>
        <input asp-for="Contact.Email" class="form-control" />
        <span asp-validation-for="Contact.Email" class="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Contact.Message"></label>
        <textarea asp-for="Contact.Message" class="form-control" rows="5"></textarea>
        <span asp-validation-for="Contact.Message" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Envoyer</button>
</form>

@if (Model.Submitted)
{
    <div class="alert alert-success mt-3">
        Merci pour votre message ! Nous vous répondrons bientôt.
    </div>
}

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

##### Exemple de fichier .cshtml.cs (Contact.cshtml.cs)

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace MyRazorApp.Pages
{
    public class ContactModel : PageModel
    {
        private readonly IEmailService _emailService;

        // Propriété pour stocker les données du formulaire
        [BindProperty]
        public ContactForm Contact { get; set; }

        // Propriété pour suivre l'état de soumission
        public bool Submitted { get; private set; }

        // Injection de dépendances via le constructeur
        public ContactModel(IEmailService emailService)
        {
            _emailService = emailService;
            Contact = new ContactForm();
        }

        // Méthode exécutée pour les requêtes GET
        public void OnGet()
        {
            // Initialisation pour l'affichage initial
            Submitted = false;
        }

        // Méthode exécutée pour les requêtes POST
        public async Task<IActionResult> OnPostAsync()
        {
            // Vérifier si le modèle est valide
            if (!ModelState.IsValid)
            {
                return Page(); // Retourner la page avec les erreurs de validation
            }

            // Traitement du formulaire (envoi d'email)
            await _emailService.SendContactEmailAsync(Contact);

            // Marquer comme soumis
            Submitted = true;

            // Retourner la même page
            return Page();
        }
    }

    // Classe pour les données du formulaire
    public class ContactForm
    {
        [Required(ErrorMessage = "Le nom est obligatoire")]
        [Display(Name = "Nom")]
        public string Name { get; set; }

        [Required(ErrorMessage = "L'email est obligatoire")]
        [EmailAddress(ErrorMessage = "Format d'email invalide")]
        [Display(Name = "Email")]
        public string Email { get; set; }

        [Required(ErrorMessage = "Le message est obligatoire")]
        [StringLength(1000, MinimumLength = 10, ErrorMessage = "Le message doit contenir entre 10 et 1000 caractères")]
        [Display(Name = "Message")]
        public string Message { get; set; }
    }
}
```

### Concepts clés

#### Directive @page

La directive `@page` au début du fichier .cshtml transforme la vue en une Razor Page qui peut traiter directement les requêtes HTTP, sans passer par un contrôleur.

```cshtml
@page
```

Vous pouvez également spécifier un modèle de route directement dans la directive :

```cshtml
@page "{id:int?}"
```

#### Classe PageModel

La classe `PageModel` est le cœur d'une Razor Page, équivalente au contrôleur dans MVC. Elle contient :

- **Propriétés** : Données affichées dans la page
- **Gestionnaires** (`OnGet`, `OnPost`, etc.) : Méthodes qui traitent les requêtes HTTP
- **Logique de page** : Tout le code nécessaire au fonctionnement de la page

#### Méthodes de gestionnaire (Handlers)

Les méthodes de gestionnaire permettent de traiter différents types de requêtes HTTP :

- `OnGet()` : Traite les requêtes GET (affichage initial)
- `OnPost()` : Traite les requêtes POST (soumission de formulaire)
- `OnPut()` : Traite les requêtes PUT
- `OnDelete()` : Traite les requêtes DELETE
- etc.

Chaque méthode peut retourner différents types de résultats :

```csharp
// Retourner la page actuelle
public IActionResult OnGet()
{
    return Page();
}

// Rediriger vers une autre page
public IActionResult OnPost()
{
    return RedirectToPage("ThankYou");
}

// Rediriger vers une page avec des paramètres
public IActionResult OnPost()
{
    return RedirectToPage("Details", new { id = 123 });
}

// Renvoyer une page d'erreur
public IActionResult OnGet(int id)
{
    var product = _productService.GetById(id);
    if (product == null)
    {
        return NotFound();
    }

    Product = product;
    return Page();
}
```

#### Cycle de vie d'une Razor Page

Le cycle de vie d'une Razor Page suit ces étapes :

1. **Construction** : L'instance du PageModel est créée
2. **Liaison de modèle** : Les propriétés avec l'attribut `[BindProperty]` sont initialisées
3. **Validation** : Le ModelState est validé
4. **Gestionnaire** : La méthode du gestionnaire appropriée est appelée (`OnGet`, `OnPost`, etc.)
5. **Résultat** : Le résultat du gestionnaire est exécuté
6. **Vue** : Si le résultat est `Page()`, la vue est rendue

## 12.4.2. Handlers et binding

Les handlers et le binding (liaison de données) sont des concepts fondamentaux de Razor Pages qui simplifient considérablement le traitement des formulaires et des interactions utilisateur.

### Handlers (Gestionnaires)

Les handlers sont des méthodes du PageModel qui répondent à des requêtes HTTP spécifiques. Ils constituent le point d'entrée pour le traitement des actions de l'utilisateur.

#### Types de handlers

Razor Pages prend en charge différents types de handlers pour chaque méthode HTTP :

```csharp
// Requête GET standard
public void OnGet()
{
    Message = "Bienvenue sur notre page !";
}

// Requête GET asynchrone
public async Task OnGetAsync()
{
    Products = await _productService.GetAllAsync();
}

// Requête POST standard
public IActionResult OnPost()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    _dataService.Save(Customer);
    return RedirectToPage("Confirmation");
}

// Requête POST asynchrone
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    await _dataService.SaveAsync(Customer);
    return RedirectToPage("Confirmation");
}
```

#### Handlers nommés

Les handlers nommés permettent de gérer plusieurs actions sur une même page :

```cshtml
<form method="post" asp-page-handler="Save">
    <!-- Champs du formulaire -->
    <button type="submit">Enregistrer</button>
</form>

<form method="post" asp-page-handler="Delete">
    <input type="hidden" name="id" value="@Model.Id" />
    <button type="submit" class="btn btn-danger">Supprimer</button>
</form>
```

Dans le PageModel :

```csharp
// Traite la soumission du bouton "Enregistrer"
public async Task<IActionResult> OnPostSaveAsync()
{
    // Logique pour enregistrer
    return RedirectToPage();
}

// Traite la soumission du bouton "Supprimer"
public async Task<IActionResult> OnPostDeleteAsync(int id)
{
    await _dataService.DeleteAsync(id);
    return RedirectToPage();
}
```

#### Handlers avec paramètres de route

Vous pouvez définir des paramètres de route directement dans la directive `@page` et les récupérer dans les handlers :

```cshtml
@page "{id:int?}"
@model ProductDetailsModel
```

```csharp
public async Task OnGetAsync(int? id)
{
    if (id.HasValue)
    {
        Product = await _productService.GetByIdAsync(id.Value);
    }
}
```

### Binding (Liaison de données)

Le binding dans Razor Pages permet de lier automatiquement les données des formulaires HTML aux propriétés du PageModel.

#### Attribut [BindProperty]

L'attribut `[BindProperty]` indique qu'une propriété doit être liée aux données du formulaire :

```csharp
public class CustomerModel : PageModel
{
    [BindProperty]
    public Customer Customer { get; set; }

    public void OnGet()
    {
        // Initialisation pour l'affichage
        Customer = new Customer();
    }

    public IActionResult OnPost()
    {
        // Customer est automatiquement rempli avec les données du formulaire
        if (!ModelState.IsValid)
        {
            return Page();
        }

        _customerService.Save(Customer);
        return RedirectToPage("Confirmation");
    }
}
```

Par défaut, le binding se fait uniquement pour les requêtes POST. Pour activer le binding sur d'autres types de requêtes :

```csharp
[BindProperty(SupportsGet = true)]
public string SearchTerm { get; set; }
```

#### Binding complexe

Le binding peut lier automatiquement des objets complexes, y compris des collections :

```cshtml
<!-- Formulaire avec objets imbriqués -->
<form method="post">
    <div class="form-group">
        <label asp-for="Customer.Name"></label>
        <input asp-for="Customer.Name" class="form-control" />
    </div>

    <div class="form-group">
        <label asp-for="Customer.Address.Street"></label>
        <input asp-for="Customer.Address.Street" class="form-control" />
    </div>

    <div class="form-group">
        <label asp-for="Customer.Address.City"></label>
        <input asp-for="Customer.Address.City" class="form-control" />
    </div>

    <!-- Collection d'objets -->
    @for (int i = 0; i < Model.Customer.PhoneNumbers.Count; i++)
    {
        <div class="form-group">
            <label>Téléphone @(i+1)</label>
            <input asp-for="Customer.PhoneNumbers[i]" class="form-control" />
        </div>
    }

    <button type="submit" class="btn btn-primary">Enregistrer</button>
</form>
```

```csharp
[BindProperty]
public Customer Customer { get; set; }

public class Customer
{
    public string Name { get; set; }
    public Address Address { get; set; }
    public List<string> PhoneNumbers { get; set; } = new List<string>();
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

#### Attribut [BindProperties]

L'attribut `[BindProperties]` au niveau de la classe permet de lier automatiquement toutes les propriétés publiques :

```csharp
[BindProperties]
public class CheckoutModel : PageModel
{
    public CustomerInfo Customer { get; set; }
    public ShippingInfo Shipping { get; set; }
    public PaymentInfo Payment { get; set; }

    // Ces propriétés seront toutes automatiquement liées
}
```

#### Sécurité du binding

Pour des raisons de sécurité, utilisez des modèles spécifiques pour le binding au lieu d'entités directes de la base de données, afin d'éviter l'over-posting (attribution massive) :

```csharp
// ViewModel sécurisé pour le binding
[BindProperty]
public CustomerViewModel CustomerVM { get; set; }

public IActionResult OnPost()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    // Mapper manuellement vers l'entité
    var customer = new Customer
    {
        Name = CustomerVM.Name,
        Email = CustomerVM.Email
        // Pas de mappage des propriétés sensibles comme IsAdmin
    };

    _customerService.Save(customer);
    return RedirectToPage("Confirmation");
}
```

### Validation

La validation dans Razor Pages fonctionne de manière similaire à MVC, en utilisant les attributs de validation et le ModelState :

```csharp
public class RegisterModel : PageModel
{
    [BindProperty]
    public UserRegistration UserReg { get; set; }

    public IActionResult OnPost()
    {
        // Validation personnalisée
        if (UserReg.Password != UserReg.ConfirmPassword)
        {
            ModelState.AddModelError("UserReg.ConfirmPassword", "Les mots de passe ne correspondent pas");
        }

        if (!ModelState.IsValid)
        {
            return Page();
        }

        // Traitement de l'inscription
        return RedirectToPage("RegisterSuccess");
    }
}

public class UserRegistration
{
    [Required(ErrorMessage = "Le nom est obligatoire")]
    [Display(Name = "Nom complet")]
    public string Name { get; set; }

    [Required(ErrorMessage = "L'email est obligatoire")]
    [EmailAddress(ErrorMessage = "Format d'email invalide")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Le mot de passe est obligatoire")]
    [StringLength(100, MinimumLength = 8, ErrorMessage = "Le mot de passe doit contenir au moins 8 caractères")]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Required(ErrorMessage = "Veuillez confirmer votre mot de passe")]
    [DataType(DataType.Password)]
    [Display(Name = "Confirmer le mot de passe")]
    public string ConfirmPassword { get; set; }
}
```

Pour afficher les erreurs de validation :

```cshtml
<form method="post">
    <div asp-validation-summary="All" class="text-danger"></div>

    <div class="form-group">
        <label asp-for="UserReg.Name"></label>
        <input asp-for="UserReg.Name" class="form-control" />
        <span asp-validation-for="UserReg.Name" class="text-danger"></span>
    </div>

    <!-- Autres champs -->

    <button type="submit" class="btn btn-primary">S'inscrire</button>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

## 12.4.3. Comparison avec MVC

Razor Pages et MVC sont deux modèles de programmation complémentaires dans ASP.NET Core. Comprendre leurs différences vous aidera à choisir la meilleure approche pour votre projet.

### Principales différences

#### Organisation du code

**MVC (Model-View-Controller)** :
- **Séparation en trois composants** : Modèles, Vues et Contrôleurs
- **Contrôleurs centralisés** : Un contrôleur gère plusieurs actions et vues
- **Structure plus complexe** : Les fichiers sont répartis dans différents dossiers

```
Controllers/
  ├─ ProductsController.cs    // Gère toutes les actions liées aux produits
  ├─ CustomersController.cs
Models/
  ├─ Product.cs
  ├─ Customer.cs
Views/
  ├─ Products/
  │   ├─ Index.cshtml         // Liste des produits
  │   ├─ Details.cshtml       // Détails d'un produit
  │   ├─ Create.cshtml        // Formulaire de création
  │   ├─ Edit.cshtml          // Formulaire d'édition
  ├─ Customers/
  │   ├─ ...
```

**Razor Pages** :
- **Paradigme orienté page** : Chaque page est une unité autonome
- **Code-behind associé** : Chaque page a son propre modèle de page
- **Structure plus intuitive** : Semblable à ASP.NET Web Forms ou PHP

```
Pages/
  ├─ Products/
  │   ├─ Index.cshtml         // Page de liste des produits
  │   ├─ Index.cshtml.cs      // Modèle de page associé
  │   ├─ Details.cshtml       // Page de détails
  │   ├─ Details.cshtml.cs    // Modèle de page associé
  │   ├─ Create.cshtml        // Page de création
  │   ├─ Create.cshtml.cs     // Modèle de page associé
  ├─ Customers/
  │   ├─ ...
```

#### Routage

**MVC** :
- Routage centralisé, généralement défini dans `Program.cs`
- Routes personnalisées avec attributs dans les contrôleurs

```csharp
// Configuration globale du routage MVC
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Dans un contrôleur
[Route("products")]
public class ProductsController : Controller
{
    [Route("list")]  // Route : /products/list
    public IActionResult Index()
    {
        // ...
    }

    [Route("details/{id}")]  // Route : /products/details/5
    public IActionResult Details(int id)
    {
        // ...
    }
}
```

**Razor Pages** :
- Routage basé sur la structure des fichiers
- Routes personnalisées directement dans la directive `@page`

```cshtml
@page // Route par défaut : /Products/Details
@model DetailsModel

<!-- ou -->

@page "{id:int}" // Route personnalisée : /Products/Details/5
@model DetailsModel
```

#### Traitement des formulaires

**MVC** :
- Actions dédiées pour afficher et traiter les formulaires
- Généralement, une action GET pour afficher le formulaire et une action POST pour le traiter

```csharp
// Action pour afficher le formulaire
[HttpGet]
public IActionResult Create()
{
    return View();
}

// Action pour traiter le formulaire
[HttpPost]
public IActionResult Create(ProductViewModel model)
{
    if (ModelState.IsValid)
    {
        // Traitement...
        return RedirectToAction("Index");
    }
    return View(model);
}
```

**Razor Pages** :
- Une seule page gère l'affichage et le traitement
- Méthodes `OnGet` et `OnPost` dans le même modèle de page

```csharp
// Méthode pour afficher le formulaire
public void OnGet()
{
    // Initialisation...
}

// Méthode pour traiter le formulaire
public IActionResult OnPost()
{
    if (ModelState.IsValid)
    {
        // Traitement...
        return RedirectToPage("Index");
    }
    return Page();
}
```

### Avantages de Razor Pages

1. **Simplicité** : Plus facile à comprendre pour les débutants
2. **Cohésion** : Logique et vue au même endroit
3. **Encapsulation** : Chaque page est indépendante et autonome
4. **Développement plus rapide** : Moins de fichiers à gérer
5. **Meilleure organisation** pour les applications CRUD simples
6. **Tests plus faciles** : Les PageModel sont simples à tester unitairement

### Avantages de MVC

1. **Séparation des préoccupations** plus stricte
2. **Flexibilité** pour les applications complexes
3. **Réutilisation** des contrôleurs pour différentes vues ou formats (HTML, API, etc.)
4. **Mieux adapté** aux grandes applications avec beaucoup de logique partagée
5. **APIs RESTful** plus naturelles à implémenter
6. **Tradition et familiarité** pour les développeurs habitués au pattern MVC

### Quand choisir quelle approche ?

**Choisir Razor Pages pour** :
- Applications CRUD simples
- Pages avec peu de logique partagée
- Projets avec des délais serrés
- Équipes avec des débutants en ASP.NET
- Sites web principalement orientés contenu
- Prototypes et POC (preuve de concept)

**Choisir MVC pour** :
- Applications complexes avec beaucoup de logique partagée
- APIs RESTful
- Applications nécessitant une séparation stricte des préoccupations
- Équipes familières avec le pattern MVC
- Applications avec beaucoup de vues partageant la même logique
- Applications hybrides (interface utilisateur + API)

### Utilisation conjointe

Il est important de noter que **Razor Pages et MVC peuvent coexister dans la même application**. Vous pouvez utiliser Razor Pages pour certaines parties (comme les pages administratives simples) et MVC pour d'autres (comme l'API).

```csharp
// Configuration qui permet d'utiliser les deux approches
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services requis
builder.Services.AddRazorPages(); // Pour Razor Pages
builder.Services.AddControllersWithViews(); // Pour MVC

var app = builder.Build();

// Configurer le pipeline
app.UseRouting();

app.MapRazorPages(); // Active le routage Razor Pages
app.MapControllerRoute( // Active le routage MVC
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

## 12.4.4. Razor components et réutilisation

La réutilisation du code est essentielle pour maintenir des applications efficaces et maintenables. Razor Pages offre plusieurs mécanismes pour factoriser et réutiliser du code.

### Vues partielles

Les vues partielles sont des fragments de code Razor qui peuvent être inclus dans plusieurs pages. Elles sont idéales pour les éléments d'interface utilisateur réutilisables.

#### Création d'une vue partielle

Les vues partielles sont généralement stockées dans le dossier `Pages/Shared` ou dans un sous-dossier thématique. Par convention, leur nom commence par un underscore (`_`).

```cshtml
<!-- Pages/Shared/_ProductCard.cshtml -->
@model Product

<div class="card mb-3">
    <div class="card-header">
        <h5 class="card-title">@Model.Name</h5>
    </div>
    <div class="card-body">
        <p class="card-text">@Model.Description</p>
        <p class="card-text">
            <strong>Prix :</strong> @Model.Price.ToString("C")
        </p>
    </div>
    <div class="card-footer">
        <a asp-page="/Products/Details" asp-route-id="@Model.Id"
           class="btn btn-primary btn-sm">Détails</a>
    </div>
</div>
```

#### Utilisation d'une vue partielle

Vous pouvez inclure une vue partielle de plusieurs façons :

```cshtml
<!-- Méthode 1 : Tag Helper (recommandée) -->
<partial name="_ProductCard" model="product" />

<!-- Méthode 2 : HTML Helper -->
@Html.Partial("_ProductCard", product)

<!-- Méthode 3 : Render -->
@{
    Html.RenderPartial("_ProductCard", product);
}
```

#### Vues partielles avec modèle local

Vous pouvez également créer des vues partielles qui accèdent au modèle de la page parent :

```cshtml
<!-- _Pagination.cshtml -->
@{
    var totalPages = (int)Math.Ceiling(Model.TotalItems / (double)Model.PageSize);
}

<nav>
    <ul class="pagination">
        @for (int i = 1; i <= totalPages; i++)
        {
            <li class="page-item @(i == Model.CurrentPage ? "active" : "")">
                <a class="page-link" asp-page="./Index" asp-route-page="@i">@i</a>
            </li>
        }
    </ul>
</nav>
```

Utilisation :

```cshtml
<!-- Dans une page -->
<partial name="_Pagination" />
```

### Page Handlers réutilisables (Services)

Pour réutiliser la logique métier, vous pouvez créer des services réutilisables qui sont injectés dans vos PageModels.

```csharp
// Service
public interface IProductService
{
    Task<List<Product>> GetProductsAsync();
    Task<Product> GetProductByIdAsync(int id);
    Task CreateProductAsync(Product product);
    Task UpdateProductAsync(Product product);
    Task DeleteProductAsync(int id);
}

// Implémentation
public class ProductService : IProductService
{
    private readonly ApplicationDbContext _context;

    public ProductService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<List<Product>> GetProductsAsync()
    {
        return await _context.Products.ToListAsync();
    }

    // Autres méthodes...
}

// Enregistrement dans Program.cs
builder.Services.AddScoped<IProductService, ProductService>();

// Utilisation dans un PageModel
public class IndexModel : PageModel
{
    private readonly IProductService _productService;

    public IndexModel(IProductService productService)
    {
        _productService = productService;
    }

    public List<Product> Products { get; private set; }

    public async Task OnGetAsync()
    {
        Products = await _productService.GetProductsAsync();
    }
}
```

### Razor Components

Razor Components est un système de composants UI puissant introduit avec Blazor, mais qui peut également être utilisé dans les applications Razor Pages traditionnelles.

#### Définition d'un Razor Component

Les Razor Components sont stockés dans un dossier nommé `Components` ou dans un sous-dossier thématique.

```cshtml
<!-- Components/Alert.razor -->
<div class="alert alert-@Type" role="alert">
    @ChildContent
</div>

@code {
    [Parameter]
    public string Type { get; set; } = "info";

    [Parameter]
    public RenderFragment ChildContent { get; set; }
}
```

#### Configuration des Razor Components dans une application Razor Pages

Pour utiliser des Razor Components dans une application Razor Pages, vous devez configurer les services nécessaires :

```csharp
// Dans Program.cs
builder.Services.AddServerSideBlazor();
builder.Services.AddRazorComponents();

// Plus tard dans le pipeline
app.MapBlazorHub();
```

#### Utilisation d'un Razor Component dans une Page

```cshtml
@page
@model IndexModel
@{
    ViewData["Title"] = "Accueil";
}

<h1>Bienvenue</h1>

<component type="typeof(Alert)" render-mode="ServerPrerendered" param-Type="success">
    <p>Votre opération a réussi !</p>
</component>

<!-- Ou avec le Tag Helper plus récent -->
<Alert Type="danger">
    <p>Une erreur est survenue.</p>
</Alert>
```

### PageModel abstrait et héritage

Vous pouvez créer des classes de base réutilisables pour vos PageModels en utilisant l'héritage. Cette approche est utile pour partager du code commun entre plusieurs pages.

```csharp
// PageModel de base
public abstract class ProductPageModelBase : PageModel
{
    protected readonly IProductService _productService;

    public ProductPageModelBase(IProductService productService)
    {
        _productService = productService;
    }

    // Méthodes communes
    protected async Task<Product> GetProductAsync(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null)
        {
            throw new ProductNotFoundException(id);
        }
        return product;
    }

    // Méthodes de gestion d'erreurs communes
    protected IActionResult HandleNotFoundException(Exception ex)
    {
        TempData["ErrorMessage"] = ex.Message;
        return RedirectToPage("./Index");
    }
}

// Page qui hérite du PageModel de base
public class EditModel : ProductPageModelBase
{
    public EditModel(IProductService productService)
        : base(productService)
    {
    }

    [BindProperty]
    public Product Product { get; set; }

    public async Task<IActionResult> OnGetAsync(int id)
    {
        try
        {
            Product = await GetProductAsync(id);
            return Page();
        }
        catch (ProductNotFoundException ex)
        {
            return HandleNotFoundException(ex);
        }
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }

        await _productService.UpdateProductAsync(Product);
        return RedirectToPage("./Index");
    }
}
```

### ViewImports et ViewStart

Les fichiers `_ViewImports.cshtml` et `_ViewStart.cshtml` vous permettent de centraliser des configurations communes à toutes vos pages.

#### _ViewImports.cshtml

Ce fichier importe des espaces de noms et configure des directives communes pour toutes les pages :

```cshtml
@using MyRazorApp
@using MyRazorApp.Models
@using MyRazorApp.Services
@using Microsoft.AspNetCore.Mvc.Localization
@namespace MyRazorApp.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, MyRazorApp
```

#### _ViewStart.cshtml

Ce fichier définit la configuration initiale pour toutes les pages, comme le layout par défaut :

```cshtml
@{
    Layout = "_Layout";
}
```

### ViewData et TempData

`ViewData` et `TempData` sont des mécanismes pour partager des données entre les pages et les composants.

```csharp
// Dans OnGet
public void OnGet()
{
    ViewData["Title"] = "Liste des produits";
    ViewData["Description"] = "Découvrez nos produits de qualité";

    // TempData persiste pendant une redirection
    TempData["SuccessMessage"] = "Opération réussie !";
}
```

Utilisation dans la vue :

```cshtml
<h1>@ViewData["Title"]</h1>
<p>@ViewData["Description"]</p>

@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success">
        @TempData["SuccessMessage"]
    </div>
}
```

### Tag Helpers personnalisés

Les Tag Helpers vous permettent de créer des éléments HTML personnalisés et réutilisables.

```csharp
// TagHelper personnalisé
[HtmlTargetElement("page-title")]
public class PageTitleTagHelper : TagHelper
{
    [HtmlAttributeName("text")]
    public string Text { get; set; }

    [HtmlAttributeName("description")]
    public string Description { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div";
        output.Attributes.SetAttribute("class", "page-header");

        var content = new TagBuilder("h1");
        content.InnerHtml.Append(Text);

        output.Content.AppendHtml(content);

        if (!string.IsNullOrEmpty(Description))
        {
            var description = new TagBuilder("p");
            description.InnerHtml.Append(Description);
            description.AddCssClass("lead");

            output.Content.AppendHtml(description);
        }
    }
}
```

Utilisation :

```cshtml
<page-title text="Nos produits" description="Découvrez notre catalogue"></page-title>
```

### Extensions de méthode

Vous pouvez créer des extensions de méthode pour ajouter des fonctionnalités réutilisables à vos PageModels.

```csharp
// Extensions de PageModel
public static class PageModelExtensions
{
    public static void SetSuccessMessage(this PageModel page, string message)
    {
        page.TempData["SuccessMessage"] = message;
    }

    public static void SetErrorMessage(this PageModel page, string message)
    {
        page.TempData["ErrorMessage"] = message;
    }

    public static IActionResult RedirectToPageWithSuccess(this PageModel page, string pageName, string message)
    {
        page.SetSuccessMessage(message);
        return page.RedirectToPage(pageName);
    }
}

// Utilisation
public class DeleteModel : PageModel
{
    private readonly IProductService _productService;

    public DeleteModel(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IActionResult> OnPostAsync(int id)
    {
        await _productService.DeleteAsync(id);
        return this.RedirectToPageWithSuccess("./Index", "Produit supprimé avec succès");
    }
}
```

### ViewComponents

Les ViewComponents sont plus puissants que les vues partielles et peuvent inclure leur propre logique et injection de dépendances.

```csharp
// Définition du ViewComponent
public class PopularProductsViewComponent : ViewComponent
{
    private readonly IProductService _productService;

    public PopularProductsViewComponent(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IViewComponentResult> InvokeAsync(int count = 5)
    {
        var popularProducts = await _productService.GetPopularProductsAsync(count);
        return View(popularProducts);
    }
}
```

Par défaut, la vue est recherchée dans :
- `Pages/Shared/Components/PopularProducts/Default.cshtml` ou
- `Views/Shared/Components/PopularProducts/Default.cshtml`

```cshtml
<!-- Vue du ViewComponent -->
@model IEnumerable<Product>

<div class="popular-products">
    <h3>Produits populaires</h3>
    <div class="row">
        @foreach (var product in Model)
        {
            <div class="col-md-4 mb-3">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">@product.Name</h5>
                        <p class="card-text">@product.Price.ToString("C")</p>
                        <a asp-page="/Products/Details" asp-route-id="@product.Id"
                           class="btn btn-sm btn-outline-primary">Voir</a>
                    </div>
                </div>
            </div>
        }
    </div>
</div>
```

Utilisation :

```cshtml
<!-- Méthode 1 : Tag Helper -->
<vc:popular-products count="3"></vc:popular-products>

<!-- Méthode 2 : HTML Helper -->
@await Component.InvokeAsync("PopularProducts", new { count = 3 })
```

### Areas dans Razor Pages

Les Areas permettent d'organiser votre application en sections fonctionnelles, ce qui est particulièrement utile pour les grandes applications.

```
MyRazorApp/
  ├─ Areas/
  │   ├─ Admin/
  │   │   ├─ Pages/
  │   │   │   ├─ _ViewImports.cshtml
  │   │   │   ├─ _ViewStart.cshtml
  │   │   │   ├─ Index.cshtml
  │   │   │   ├─ Index.cshtml.cs
  │   ├─ Shop/
  │   │   ├─ Pages/
  │   │   │   ├─ _ViewImports.cshtml
  │   │   │   ├─ _ViewStart.cshtml
  │   │   │   ├─ Products/
  │   │   │   │   ├─ Index.cshtml
  │   │   │   │   ├─ Index.cshtml.cs
  ├─ Pages/
  │   ├─ Index.cshtml
  │   ├─ Index.cshtml.cs
```

Configuration dans Program.cs :

```csharp
app.MapRazorPages()
   .AddRazorPagesOptions(options =>
   {
       options.Conventions.AddAreaPageRoute("Admin", "/Index", "/admin");
       options.Conventions.AddAreaPageRoute("Shop", "/Products/Index", "/shop");
   });
```

Liens vers des pages dans des Areas :

```cshtml
<a asp-area="Admin" asp-page="/Index">Administration</a>
<a asp-area="Shop" asp-page="/Products/Index">Boutique</a>
```

## Meilleures pratiques pour Razor Pages

Pour conclure cette section sur Razor Pages, voici quelques meilleures pratiques pour développer des applications efficaces et maintenables :

### Organisation du code

1. **Suivez le principe de responsabilité unique** : Chaque PageModel ne doit avoir qu'une seule raison de changer.
2. **Utilisez des interfaces pour les dépendances** : Injectez des interfaces plutôt que des implémentations concrètes.
3. **Créez des services pour la logique métier** : Ne mettez pas la logique métier complexe dans les PageModels.
4. **Utilisez des modèles dédiés pour la liaison** : Évitez de lier directement aux entités de la base de données.

### Structure et nommage

1. **Organisez les pages par fonctionnalité** : Utilisez des sous-dossiers pour regrouper les pages liées.
2. **Utilisez des Areas pour les grandes applications** : Séparez les sections administratives des sections publiques.
3. **Suivez des conventions de nommage cohérentes** : Par exemple, `Index` pour les listes, `Details` pour les détails.
4. **Nommez les modèles de page avec un suffixe `Model`** : Par exemple, `IndexModel`, `DetailsModel`.

### Sécurité

1. **Validez toujours côté serveur** : Ne faites pas confiance uniquement à la validation côté client.
2. **Utilisez l'attribut [ValidateAntiForgeryToken]** : Pour se protéger contre les attaques CSRF.
3. **Utilisez des modèles dédiés pour le binding** : Pour éviter l'over-posting (attribution massive).
4. **Appliquez l'authentification et l'autorisation au niveau de la page** : Utilisez les attributs `[Authorize]`.

### Performance

1. **Utilisez des méthodes asynchrones** : Pour les opérations I/O comme les accès à la base de données.
2. **Chargez uniquement les données nécessaires** : Utilisez des projections et des filtres.
3. **Mettez en cache les données statiques ou qui changent rarement** : Utilisez le service `IMemoryCache`.
4. **Utilisez des vues partielles et des ViewComponents avec parcimonie** : Trop de composants peuvent impacter les performances.

### Réutilisation

1. **Créez des services partagés** : Pour la logique métier réutilisable.
2. **Utilisez des vues partielles** : Pour les éléments d'interface réutilisables.
3. **Utilisez des ViewComponents** : Pour les composants complexes avec leur propre logique.
4. **Créez des PageModels de base** : Pour partager du code entre pages similaires.
5. **Utilisez des Tag Helpers personnalisés** : Pour encapsuler la logique de rendu complexe.

### Tests

1. **Écrivez des tests unitaires pour vos PageModels** : Testez les méthodes `OnGet`, `OnPost`, etc.
2. **Moquez les dépendances** : Utilisez des frameworks comme Moq pour simuler les services.
3. **Testez les scénarios de validation** : Vérifiez que les validations fonctionnent correctement.
4. **Écrivez des tests d'intégration** : Pour vérifier l'interaction entre les composants.

## Conclusion

Razor Pages offre une approche simplifiée pour le développement web avec ASP.NET Core, particulièrement adaptée aux applications CRUD et aux pages avec une logique simple. Son modèle de programmation orienté page facilite l'apprentissage pour les débutants tout en offrant la puissance et la flexibilité nécessaires pour les développeurs expérimentés.

En comprenant les différences entre Razor Pages et MVC, vous pouvez choisir la meilleure approche pour chaque partie de votre application. Et grâce aux nombreuses options de réutilisation de code (vues partielles, ViewComponents, services, héritage, etc.), vous pouvez construire des applications maintenables et évolutives.

Razor Pages continue d'évoluer avec chaque nouvelle version d'ASP.NET Core, apportant de nouvelles fonctionnalités et améliorations pour répondre aux besoins des développeurs modernes.

⏭️
