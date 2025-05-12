# 12.6. Documentation API avec Swagger/OpenAPI

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

La documentation est un aspect fondamental du développement d'API. Une API bien documentée est non seulement plus facile à comprendre et à utiliser pour les développeurs externes, mais elle facilite également la maintenance et l'évolution de votre propre code. Swagger (désormais officiellement connu sous le nom d'OpenAPI) est devenu la norme de facto pour documenter les API RESTful dans l'écosystème .NET et au-delà.

Dans ce chapitre, nous allons explorer en profondeur comment documenter efficacement vos API ASP.NET Core à l'aide de Swagger/OpenAPI. Nous couvrirons tout, depuis l'installation et la configuration de base jusqu'aux techniques avancées de personnalisation, en passant par la génération de clients et l'intégration avec d'autres outils.

### Distinction entre OpenAPI et Swagger

Avant d'aller plus loin, clarifions une confusion courante :

- **OpenAPI** est une spécification standardisée pour décrire les API REST. C'est un format qui définit comment documenter une API de manière standardisée.
- **Swagger** est une suite d'outils qui implémente la spécification OpenAPI. Ces outils incluent Swagger UI, Swagger Editor, Swagger Codegen, etc.

Le projet Swagger a été donné à l'Initiative OpenAPI en 2015, et depuis, les deux termes sont souvent utilisés de manière interchangeable. Dans ce tutoriel, nous utiliserons principalement "Swagger" pour désigner les outils et "OpenAPI" pour désigner la spécification.

## 12.6.1. Installation et configuration de Swagger

### Packages principaux pour ASP.NET Core

Dans l'écosystème ASP.NET Core, deux implémentations principales de Swagger/OpenAPI sont disponibles :

1. **Swashbuckle** - La plus populaire et celle que nous utiliserons dans ce tutoriel
2. **NSwag** - Une alternative qui offre des fonctionnalités similaires

Pour Swashbuckle, trois packages principaux sont utilisés :

- **Swashbuckle.AspNetCore.Swagger** : Le middleware de base qui expose les documents OpenAPI en tant qu'endpoints JSON
- **Swashbuckle.AspNetCore.SwaggerGen** : Le générateur qui construit le document OpenAPI à partir de vos contrôleurs, routes et modèles
- **Swashbuckle.AspNetCore.SwaggerUI** : Une version intégrée de l'interface utilisateur Swagger pour visualiser et tester vos API

### Installation des packages

#### Via l'interface de ligne de commande (CLI)

```bash
# Installation du package principal (inclut les trois packages mentionnés ci-dessus)
dotnet add package Swashbuckle.AspNetCore

# Pour les annotations avancées (optionnel)
dotnet add package Swashbuckle.AspNetCore.Annotations

# Pour ReDoc (interface alternative à Swagger UI, optionnel)
dotnet add package Swashbuckle.AspNetCore.ReDoc
```

#### Via la console du Gestionnaire de packages (Visual Studio)

```powershell
Install-Package Swashbuckle.AspNetCore
Install-Package Swashbuckle.AspNetCore.Annotations
Install-Package Swashbuckle.AspNetCore.ReDoc
```

#### Via l'interface graphique de Visual Studio

1. Cliquez avec le bouton droit sur votre projet dans l'Explorateur de solutions
2. Sélectionnez "Gérer les packages NuGet"
3. Recherchez "Swashbuckle.AspNetCore"
4. Cliquez sur "Installer"

### Configuration de base

#### Pour ASP.NET Core avec le modèle de démarrage traditionnel (.NET 6 et versions antérieures)

Dans votre fichier `Startup.cs`, vous devez configurer Swagger dans les méthodes `ConfigureServices` et `Configure` :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    // Ajout des services Swagger
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo
        {
            Title = "Mon API",
            Version = "v1",
            Description = "Une API d'exemple pour le tutoriel C#",
            Contact = new OpenApiContact
            {
                Name = "Votre Nom",
                Email = "contact@exemple.com",
                Url = new Uri("https://www.exemple.com")
            },
            License = new OpenApiLicense
            {
                Name = "Licence MIT",
                Url = new Uri("https://opensource.org/licenses/MIT")
            }
        });

        // Inclure les commentaires XML (nous verrons cela plus en détail plus tard)
        var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
        var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
        c.IncludeXmlComments(xmlPath);
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();

        // Activer le middleware pour servir le document JSON généré
        app.UseSwagger();

        // Activer le middleware pour servir l'interface utilisateur Swagger
        app.UseSwaggerUI(c =>
        {
            c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");
            // Optionnel : définir Swagger UI comme page d'accueil
            // c.RoutePrefix = string.Empty;
        });
    }

    app.UseHttpsRedirection();
    app.UseRouting();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

#### Pour ASP.NET Core avec le modèle de démarrage minimal (.NET 7/8 et versions ultérieures)

Dans votre fichier `Program.cs` :

```csharp
using Microsoft.OpenApi.Models;
using System.Reflection;

var builder = WebApplication.CreateBuilder(args);

// Ajouter les services au conteneur
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer(); // Important pour les API minimales

// Configuration de Swagger
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Mon API",
        Version = "v1",
        Description = "Une API d'exemple pour le tutoriel C#",
        Contact = new OpenApiContact
        {
            Name = "Votre Nom",
            Email = "contact@exemple.com",
            Url = new Uri("https://www.exemple.com")
        }
    });

    // Inclure les commentaires XML
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    c.IncludeXmlComments(xmlPath);
});

var app = builder.Build();

// Configurer le pipeline de requêtes HTTP
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");
        // c.RoutePrefix = string.Empty; // Pour définir Swagger UI comme page d'accueil
    });
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Configuration du fichier XML pour les commentaires

Pour que Swagger puisse utiliser vos commentaires XML, vous devez configurer votre projet pour générer un fichier XML lors de la compilation.

#### Modification du fichier .csproj

Ajoutez les lignes suivantes dans votre fichier de projet (.csproj) :

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

L'option `GenerateDocumentationFile` indique au compilateur de générer un fichier XML de documentation.
L'option `NoWarn` supprime les avertissements pour les membres qui n'ont pas de commentaires XML (code 1591).

### Gestion des versions d'API

Si votre API possède plusieurs versions, vous pouvez les documenter séparément :

```csharp
services.AddSwaggerGen(c =>
{
    // Version 1
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Mon API",
        Version = "v1",
        Description = "Version 1 de mon API"
    });

    // Version 2
    c.SwaggerDoc("v2", new OpenApiInfo
    {
        Title = "Mon API",
        Version = "v2",
        Description = "Version 2 de mon API avec fonctionnalités supplémentaires"
    });

    // Configuration pour identifier la version correcte
    c.DocInclusionPredicate((docName, apiDesc) =>
    {
        if (!apiDesc.TryGetMethodInfo(out var methodInfo)) return false;

        var versions = methodInfo.DeclaringType
            .GetCustomAttributes(true)
            .OfType<ApiVersionAttribute>()
            .SelectMany(attr => attr.Versions);

        return versions.Any(v => $"v{v}" == docName);
    });
});

// Dans Configure ou le code équivalent dans .NET 8
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");
    c.SwaggerEndpoint("/swagger/v2/swagger.json", "Mon API V2");
});
```

Pour que ce code fonctionne, vous aurez besoin du package `Microsoft.AspNetCore.Mvc.Versioning` pour utiliser l'attribut `ApiVersionAttribute`.

### Vérification de l'installation

Une fois la configuration terminée, lancez votre application et accédez à :

- `/swagger/v1/swagger.json` pour voir le document OpenAPI brut au format JSON
- `/swagger` pour accéder à l'interface utilisateur Swagger

Si tout est configuré correctement, vous devriez voir l'interface Swagger UI avec vos endpoints API répertoriés et prêts à être testés.

## 12.6.2. Documentation des endpoints

Maintenant que vous avez configuré Swagger, il est temps de documenter vos API en détail. Voici plusieurs approches que vous pouvez utiliser, des plus simples aux plus avancées.

### Utilisation des commentaires XML

Les commentaires XML sont une méthode puissante pour documenter vos API, car ils sont directement intégrés dans votre code et servent à la fois de documentation pour les développeurs et pour Swagger.

#### Structure des commentaires XML

```csharp
/// <summary>
/// Description de la méthode ou de la classe
/// </summary>
/// <param name="parameterName">Description du paramètre</param>
/// <returns>Description de la valeur de retour</returns>
/// <remarks>
/// Informations supplémentaires qui peuvent inclure des exemples, des notes, etc.
/// </remarks>
/// <response code="200">Description de la réponse 200 OK</response>
/// <response code="400">Description de la réponse 400 Bad Request</response>
```

#### Exemple complet avec commentaires XML

```csharp
/// <summary>
/// Contrôleur pour gérer les opérations CRUD sur les produits
/// </summary>
[ApiController]
[Route("api/[controller]")]
public class ProduitsController : ControllerBase
{
    private readonly IProduitsService _produitsService;

    /// <summary>
    /// Initialise une nouvelle instance du contrôleur de produits
    /// </summary>
    /// <param name="produitsService">Service d'accès aux produits</param>
    public ProduitsController(IProduitsService produitsService)
    {
        _produitsService = produitsService;
    }

    /// <summary>
    /// Récupère tous les produits disponibles
    /// </summary>
    /// <remarks>
    /// Exemple de requête :
    ///
    ///     GET /api/produits
    ///
    /// </remarks>
    /// <returns>Une liste de tous les produits</returns>
    /// <response code="200">Retourne la liste des produits</response>
    /// <response code="404">Si aucun produit n'est trouvé</response>
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<Produit>), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<IEnumerable<Produit>>> GetAllProduits()
    {
        var produits = await _produitsService.GetAllAsync();

        if (produits == null || !produits.Any())
            return NotFound("Aucun produit trouvé");

        return Ok(produits);
    }

    /// <summary>
    /// Récupère un produit spécifique par son identifiant
    /// </summary>
    /// <param name="id">Identifiant unique du produit</param>
    /// <returns>Le produit correspondant à l'identifiant</returns>
    /// <response code="200">Retourne le produit trouvé</response>
    /// <response code="404">Si le produit n'existe pas</response>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(Produit), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<Produit>> GetProduitById(int id)
    {
        var produit = await _produitsService.GetByIdAsync(id);

        if (produit == null)
            return NotFound($"Produit avec l'ID {id} non trouvé");

        return Ok(produit);
    }

    /// <summary>
    /// Crée un nouveau produit
    /// </summary>
    /// <param name="produit">Objet produit à créer</param>
    /// <returns>Le produit nouvellement créé</returns>
    /// <response code="201">Retourne le produit créé</response>
    /// <response code="400">Si le produit est null ou invalide</response>
    [HttpPost]
    [Consumes("application/json")]
    [Produces("application/json")]
    [ProducesResponseType(typeof(Produit), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<Produit>> CreateProduit([FromBody] Produit produit)
    {
        if (produit == null)
            return BadRequest("Le produit ne peut pas être null");

        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var createdProduit = await _produitsService.CreateAsync(produit);

        return CreatedAtAction(
            nameof(GetProduitById),
            new { id = createdProduit.Id },
            createdProduit);
    }

    /// <summary>
    /// Met à jour un produit existant
    /// </summary>
    /// <param name="id">Identifiant du produit à mettre à jour</param>
    /// <param name="produit">Nouvelles valeurs pour le produit</param>
    /// <returns>Aucun contenu si la mise à jour est réussie</returns>
    /// <response code="204">Si la mise à jour est réussie</response>
    /// <response code="400">Si l'ID ne correspond pas au produit ou si le modèle est invalide</response>
    /// <response code="404">Si le produit n'existe pas</response>
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateProduit(int id, [FromBody] Produit produit)
    {
        if (produit == null || id != produit.Id)
            return BadRequest("L'ID du produit ne correspond pas");

        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var exists = await _produitsService.ExistsAsync(id);

        if (!exists)
            return NotFound($"Produit avec l'ID {id} non trouvé");

        await _produitsService.UpdateAsync(produit);

        return NoContent();
    }

    /// <summary>
    /// Supprime un produit spécifique
    /// </summary>
    /// <param name="id">Identifiant du produit à supprimer</param>
    /// <returns>Aucun contenu si la suppression est réussie</returns>
    /// <response code="204">Si la suppression est réussie</response>
    /// <response code="404">Si le produit n'existe pas</response>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteProduit(int id)
    {
        var exists = await _produitsService.ExistsAsync(id);

        if (!exists)
            return NotFound($"Produit avec l'ID {id} non trouvé");

        await _produitsService.DeleteAsync(id);

        return NoContent();
    }
}
```

### Documentation des modèles de données

Vous pouvez également documenter vos modèles de données pour fournir plus de détails sur les propriétés et les contraintes :

```csharp
/// <summary>
/// Représente un produit dans le système
/// </summary>
public class Produit
{
    /// <summary>
    /// Identifiant unique du produit
    /// </summary>
    /// <example>1</example>
    public int Id { get; set; }

    /// <summary>
    /// Nom du produit
    /// </summary>
    /// <example>Clavier ergonomique</example>
    [Required(ErrorMessage = "Le nom du produit est requis")]
    [StringLength(100, ErrorMessage = "Le nom ne peut pas dépasser 100 caractères")]
    public string Nom { get; set; }

    /// <summary>
    /// Description détaillée du produit
    /// </summary>
    /// <example>Un clavier conçu pour réduire la fatigue lors d'une utilisation prolongée</example>
    [StringLength(500)]
    public string Description { get; set; }

    /// <summary>
    /// Prix unitaire du produit en euros
    /// </summary>
    /// <example>49.99</example>
    [Range(0.01, 10000, ErrorMessage = "Le prix doit être entre 0.01€ et 10,000€")]
    [RegularExpression(@"^\d+(\.\d{1,2})?$", ErrorMessage = "Le prix doit avoir maximum 2 décimales")]
    [DisplayFormat(DataFormatString = "{0:C}")]
    public decimal Prix { get; set; }

    /// <summary>
    /// Catégorie à laquelle appartient le produit
    /// </summary>
    /// <example>Périphériques</example>
    [Required]
    public string Categorie { get; set; }

    /// <summary>
    /// Indique si le produit est en stock
    /// </summary>
    /// <example>true</example>
    public bool EnStock { get; set; }

    /// <summary>
    /// Quantité disponible en stock
    /// </summary>
    /// <example>100</example>
    [Range(0, int.MaxValue)]
    public int QuantiteStock { get; set; }

    /// <summary>
    /// Date d'ajout du produit au catalogue
    /// </summary>
    /// <example>2023-01-15</example>
    [DataType(DataType.Date)]
    public DateTime DateAjout { get; set; }
}
```

### Attributs de documentation

En plus des commentaires XML, vous pouvez utiliser divers attributs pour enrichir la documentation Swagger :

#### [Produces] et [Consumes]

Indiquent les formats de média que votre API peut produire ou consommer :

```csharp
[Produces("application/json", "application/xml")]
[Consumes("application/json")]
[HttpPost]
public ActionResult<Produit> CreateProduit([FromBody] Produit produit)
{
    // Implémentation
}
```

#### [ProducesResponseType]

Spécifie les types de réponse possibles et leurs codes HTTP associés :

```csharp
[HttpGet]
[ProducesResponseType(typeof(IEnumerable<Produit>), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
public ActionResult<IEnumerable<Produit>> GetAllProduits()
{
    // Implémentation
}
```

#### [ApiExplorerSettings]

Contrôle si une action ou un contrôleur doit être inclus dans la documentation Swagger :

```csharp
[ApiExplorerSettings(IgnoreApi = true)]
[HttpGet("internal")]
public IActionResult InternalEndpoint()
{
    // Cet endpoint ne sera pas visible dans Swagger
    return Ok();
}
```

### Utilisation de Swashbuckle.AspNetCore.Annotations

Pour une documentation encore plus riche, utilisez le package d'annotations :

```csharp
[HttpGet("{id}")]
[SwaggerOperation(
    Summary = "Obtenir un produit par son ID",
    Description = "Récupère un produit unique en utilisant son identifiant unique",
    OperationId = "GetProductById",
    Tags = new[] { "Produits" }
)]
[SwaggerResponse(StatusCodes.Status200OK, "Le produit a été trouvé", typeof(Produit))]
[SwaggerResponse(StatusCodes.Status404NotFound, "Le produit n'existe pas")]
[SwaggerResponse(StatusCodes.Status500InternalServerError, "Erreur interne du serveur")]
public ActionResult<Produit> GetProduitById(int id)
{
    // Implémentation
}
```

N'oubliez pas d'activer les annotations dans la configuration :

```csharp
services.AddSwaggerGen(c =>
{
    // Autres configurations...
    c.EnableAnnotations();
});
```

### Exemples de requêtes et réponses

Les exemples de requêtes et de réponses rendent votre documentation beaucoup plus utile. Voici comment les ajouter :

#### Méthode 1 : Via les attributs [SwaggerRequestExample] et [SwaggerResponseExample]

Pour cette méthode, installez le package `Swashbuckle.AspNetCore.Examples` ou `Swashbuckle.AspNetCore.Filters` :

```csharp
// Définir la classe d'exemple
public class ProduitRequestExample : IExamplesProvider<Produit>
{
    public Produit GetExamples()
    {
        return new Produit
        {
            Nom = "Clavier ergonomique",
            Description = "Un clavier conçu pour réduire la fatigue",
            Prix = 49.99m,
            Categorie = "Périphériques",
            EnStock = true,
            QuantiteStock = 100,
            DateAjout = DateTime.Now
        };
    }
}

// Utiliser dans le contrôleur
[HttpPost]
[SwaggerRequestExample(typeof(Produit), typeof(ProduitRequestExample))]
[SwaggerResponseExample(StatusCodes.Status201Created, typeof(ProduitResponseExample))]
public ActionResult<Produit> CreateProduit([FromBody] Produit produit)
{
    // Implémentation
}
```

N'oubliez pas d'activer les exemples dans la configuration :

```csharp
services.AddSwaggerGen(c =>
{
    // Autres configurations...
    c.ExampleFilters();
});

services.AddSwaggerExamplesFromAssemblyOf<ProduitRequestExample>();
```

#### Méthode 2 : Via l'attribut [SwaggerSchema] et la propriété Example

```csharp
public class Produit
{
    [SwaggerSchema(Description = "Identifiant unique du produit", Format = "int32", Example = "1")]
    public int Id { get; set; }

    [SwaggerSchema(Description = "Nom du produit", Example = "Clavier ergonomique")]
    public string Nom { get; set; }

    // Autres propriétés...
}
```

#### Méthode 3 : Via ISchemaFilter pour les exemples plus complexes

```csharp
public class ProduitSchemaFilter : ISchemaFilter
{
    public void Apply(OpenApiSchema schema, SchemaFilterContext context)
    {
        if (context.Type == typeof(Produit))
        {
            schema.Example = new OpenApiObject
            {
                ["id"] = new OpenApiInteger(1),
                ["nom"] = new OpenApiString("Clavier ergonomique"),
                ["description"] = new OpenApiString("Un clavier conçu pour réduire la fatigue"),
                ["prix"] = new OpenApiDouble(49.99),
                ["categorie"] = new OpenApiString("Périphériques"),
                ["enStock"] = new OpenApiBoolean(true),
                ["quantiteStock"] = new OpenApiInteger(100),
                ["dateAjout"] = new OpenApiString(DateTime.Now.ToString("yyyy-MM-dd"))
            };
        }
    }
}

// Ajouter à la configuration
services.AddSwaggerGen(c =>
{
    // Autres configurations...
    c.SchemaFilter<ProduitSchemaFilter>();
});
```

### Organisation avec tags et groupes

Pour une meilleure organisation, utilisez des tags pour regrouper vos endpoints :

#### Au niveau du contrôleur

```csharp
[ApiController]
[Route("api/[controller]")]
[SwaggerTag("Gestion des produits", "Endpoints pour créer, lire, mettre à jour et supprimer des produits")]
public class ProduitsController : ControllerBase
{
    // Méthodes...
}
```

#### Au niveau de l'action

```csharp
[HttpGet]
[SwaggerOperation(Tags = new[] { "Produits - Lecture" })]
public ActionResult<IEnumerable<Produit>> GetAllProduits()
{
    // Implémentation
}
```

#### Personnaliser l'ordre des tags

```csharp
services.AddSwaggerGen(c =>
{
    // Autres configurations...

    c.TagActionsBy(api => {
        if (api.GroupName != null)
            return new[] { api.GroupName };

        if (api.ActionDescriptor is ControllerActionDescriptor controllerActionDescriptor)
            return new[] { controllerActionDescriptor.ControllerName };

        return new[] { "Others" };
    });

    c.DocInclusionPredicate((name, api) => true);

    // Définir l'ordre des tags
    c.OrderActionsBy(apiDesc => $"{apiDesc.ActionDescriptor.RouteValues["controller"]}_{apiDesc.RelativePath}");
});
```

## 12.6.3. Personnalisation de l'interface Swagger UI

L'interface utilisateur de Swagger peut être personnalisée de nombreuses façons pour l'adapter à vos besoins spécifiques et à l'identité visuelle de votre entreprise.

### Options de configuration de base

```csharp
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");

    // Page d'accueil = Swagger UI
    c.RoutePrefix = string.Empty;

    // Fermer les sections par défaut
    c.DocExpansion(DocExpansion.None);

    // Cacher la section Models
    c.DefaultModelsExpandDepth(-1);

    // Afficher la durée des requêtes
    c.DisplayRequestDuration();

    // Activer les liens profonds pour les opérations
    c.EnableDeepLinking();

    // Utiliser un thème sombre
    c.EnableTryItOutByDefault();

    // Filtrer par tag
    c.DisplayOperationId();

    // Trier les endpoints par méthode HTTP
    c.SortTagsBy((a, b) => string.Compare(a, b, StringComparison.InvariantCultureIgnoreCase));

    // Titre du navigateur
    c.DocumentTitle = "Documentation API - Mon Entreprise";
});
```

### Personnalisation de l'apparence avec CSS

#### 2. Intégrez le fichier CSS dans Swagger UI

Une fois votre fichier CSS créé, vous devez l'intégrer dans Swagger UI :

```csharp
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");

    // Injecter le CSS personnalisé
    c.InjectStylesheet("/swagger-ui/custom.css");
});
```

#### 3. Assurez-vous que les fichiers statiques sont correctement servis

Pour que votre fichier CSS soit correctement servi, ajoutez le middleware pour les fichiers statiques :

```csharp
// Dans Configure() ou dans Program.cs
app.UseStaticFiles();
```

### Ajout d'un logo personnalisé

#### 1. Placez votre logo dans le dossier wwwroot

Ajoutez votre logo (par exemple logo.png) dans le dossier `wwwroot/images/`.

#### 2. Personnalisez Swagger UI pour utiliser votre logo

```csharp
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");

    // Logo personnalisé
    c.InjectStylesheet("/swagger-ui/custom.css");
});
```

Puis dans votre fichier CSS :

```css
/* Remplacer le logo Swagger par votre propre logo */
.swagger-ui .topbar-wrapper img {
    content: url(/images/logo.png);
    height: 40px;
}

/* Ou, si vous préférez cacher complètement le logo Swagger */
.swagger-ui .topbar .download-url-wrapper {
    display: none;
}
```

### Ajout de JavaScript personnalisé

Vous pouvez également ajouter du JavaScript personnalisé pour enrichir l'interface Swagger UI :

#### 1. Créez un fichier JavaScript personnalisé

Créez un fichier `custom.js` dans votre dossier `wwwroot/swagger-ui/` :

```javascript
// Attendre que Swagger UI soit complètement chargé
window.onload = function() {
    // Exemple : ajouter un bouton personnalisé
    var topbarElement = document.getElementsByClassName('topbar')[0];
    if (topbarElement) {
        var customButton = document.createElement('button');
        customButton.className = 'btn custom-btn';
        customButton.innerText = 'Documentation PDF';
        customButton.onclick = function() {
            window.open('/api/documentation/pdf', '_blank');
        };
        topbarElement.appendChild(customButton);
    }

    // Exemple : modifier le comportement des sections
    var opblocks = document.getElementsByClassName('opblock');
    for (var i = 0; i < opblocks.length; i++) {
        opblocks[i].addEventListener('click', function(e) {
            console.log('Section cliquée:', this.id);
        });
    }
};
```

#### 2. Intégrez le fichier JavaScript dans Swagger UI

```csharp
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");

    c.InjectStylesheet("/swagger-ui/custom.css");
    c.InjectJavascript("/swagger-ui/custom.js");
});
```

### Personnalisation des sélecteurs d'endpoints

Si votre API a plusieurs versions ou environnements, vous pouvez personnaliser les sélecteurs :

```csharp
app.UseSwaggerUI(c =>
{
    // Plusieurs versions d'API
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "API Version 1.0");
    c.SwaggerEndpoint("/swagger/v2/swagger.json", "API Version 2.0");
    c.SwaggerEndpoint("/swagger/beta/swagger.json", "API Beta (Expérimentale)");

    // Grouper par environnement (dans un menu déroulant)
    c.ConfigObject.Urls = new[]
    {
        new UrlDescriptor { Name = "Production - v1", Url = "/swagger/prod-v1/swagger.json" },
        new UrlDescriptor { Name = "Production - v2", Url = "/swagger/prod-v2/swagger.json" },
        new UrlDescriptor { Name = "Staging - v1", Url = "/swagger/stage-v1/swagger.json" },
        new UrlDescriptor { Name = "Development - Latest", Url = "/swagger/dev/swagger.json" }
    };

    c.DisplayRequestDuration();
});
```

### Configuration de l'authentification dans Swagger UI

Si votre API utilise l'authentification, vous pouvez configurer Swagger UI pour inclure les jetons d'authentification dans les requêtes :

#### Authentification par jeton JWT (Bearer)

```csharp
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Mon API", Version = "v1" });

    // Définir le schéma de sécurité (JWT Bearer)
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Entrez 'Bearer [espace] votre_jeton' dans le champ ci-dessous."
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});
```

#### Authentification par clé API

```csharp
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Mon API", Version = "v1" });

    c.AddSecurityDefinition("ApiKey", new OpenApiSecurityScheme
    {
        Name = "X-API-KEY",
        Type = SecuritySchemeType.ApiKey,
        Scheme = "ApiKeyScheme",
        In = ParameterLocation.Header,
        Description = "Entrez votre clé API dans le champ ci-dessous."
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "ApiKey"
                },
                In = ParameterLocation.Header
            },
            new List<string>()
        }
    });
});
```

#### Authentification OAuth 2.0

```csharp
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Mon API", Version = "v1" });

    c.AddSecurityDefinition("oauth2", new OpenApiSecurityScheme
    {
        Type = SecuritySchemeType.OAuth2,
        Flows = new OpenApiOAuthFlows
        {
            Implicit = new OpenApiOAuthFlow
            {
                AuthorizationUrl = new Uri("https://auth.monapi.com/connect/authorize"),
                TokenUrl = new Uri("https://auth.monapi.com/connect/token"),
                Scopes = new Dictionary<string, string>
                {
                    { "read:produits", "Lecture des produits" },
                    { "write:produits", "Création/modification des produits" },
                    { "admin", "Accès administrateur complet" }
                }
            }
        }
    });
});

// Dans UseSwaggerUI
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");

    // Configuration OAuth2
    c.OAuthClientId("swagger-ui");
    c.OAuthClientSecret("swagger-ui-secret"); // Attention à ne pas exposer de vrais secrets en production
    c.OAuthRealm("swagger-ui-realm");
    c.OAuthAppName("Swagger UI");
    c.OAuthScopeSeparator(" ");
    c.OAuthUsePkce();

    // Pour tester facilement
    c.ConfigObject.AdditionalItems.Add("withCredentials", true);
});
```

### Utilisation de filtres pour personnaliser la documentation générée

Les filtres vous permettent de modifier la documentation générée avant qu'elle ne soit servie. Voici quelques exemples utiles :

#### Filtre de schéma pour personnaliser les exemples

```csharp
public class ExampleSchemaFilter : ISchemaFilter
{
    public void Apply(OpenApiSchema schema, SchemaFilterContext context)
    {
        if (context.Type == typeof(Produit))
        {
            // Ajouter un exemple complet
            schema.Example = new OpenApiObject
            {
                ["id"] = new OpenApiInteger(1),
                ["nom"] = new OpenApiString("Clavier ergonomique"),
                ["prix"] = new OpenApiDouble(49.99),
                ["enStock"] = new OpenApiBoolean(true),
                ["categorie"] = new OpenApiString("Périphériques")
            };
        }
        else if (context.Type == typeof(ErrorResponse))
        {
            schema.Example = new OpenApiObject
            {
                ["code"] = new OpenApiInteger(404),
                ["message"] = new OpenApiString("Ressource non trouvée"),
                ["details"] = new OpenApiString("L'élément demandé n'existe pas dans la base de données")
            };
        }
    }
}

// Ajout du filtre dans la configuration
services.AddSwaggerGen(c =>
{
    // Autres configurations...
    c.SchemaFilter<ExampleSchemaFilter>();
});
```

#### Filtre d'opération pour ajouter des en-têtes personnalisés

```csharp
public class CustomHeaderOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        // Ajouter un en-tête à toutes les opérations
        if (operation.Parameters == null)
            operation.Parameters = new List<OpenApiParameter>();

        operation.Parameters.Add(new OpenApiParameter
        {
            Name = "X-Correlation-ID",
            In = ParameterLocation.Header,
            Required = false,
            Schema = new OpenApiSchema
            {
                Type = "string",
                Format = "uuid",
                Example = new OpenApiString(Guid.NewGuid().ToString())
            },
            Description = "ID de corrélation pour le suivi des requêtes"
        });

        // Ajouter un en-tête uniquement pour certaines opérations spécifiques
        var routeAttributes = context.MethodInfo.GetCustomAttributes(true)
                                .OfType<RouteAttribute>()
                                .Select(attr => attr.Template)
                                .ToList();

        if (routeAttributes.Any(route => route?.Contains("admin") == true))
        {
            operation.Parameters.Add(new OpenApiParameter
            {
                Name = "X-Admin-Token",
                In = ParameterLocation.Header,
                Required = true,
                Schema = new OpenApiSchema { Type = "string" },
                Description = "Jeton d'administration requis"
            });
        }
    }
}

// Ajout du filtre dans la configuration
services.AddSwaggerGen(c =>
{
    // Autres configurations...
    c.OperationFilter<CustomHeaderOperationFilter>();
});
```

#### Filtre de document pour ajouter des serveurs ou des informations globales

```csharp
public class MultiEnvironmentDocumentFilter : IDocumentFilter
{
    public void Apply(OpenApiDocument swaggerDoc, DocumentFilterContext context)
    {
        // Ajouter plusieurs environnements comme serveurs
        swaggerDoc.Servers = new List<OpenApiServer>
        {
            new OpenApiServer { Url = "https://api.production.com", Description = "Serveur de production" },
            new OpenApiServer { Url = "https://api.staging.com", Description = "Serveur de pré-production" },
            new OpenApiServer { Url = "https://localhost:5001", Description = "Serveur de développement local" }
        };

        // Ajouter des extensions personnalisées
        swaggerDoc.Extensions.Add("x-api-owner", new OpenApiString("Équipe Backend"));
        swaggerDoc.Extensions.Add("x-api-status", new OpenApiString("Beta"));

        // Si nécessaire, modifiez des informations globales
        swaggerDoc.Info.Contact = new OpenApiContact
        {
            Name = "Support Technique",
            Email = "support@monentreprise.com",
            Url = new Uri("https://support.monentreprise.com")
        };
    }
}

// Ajout du filtre dans la configuration
services.AddSwaggerGen(c =>
{
    // Autres configurations...
    c.DocumentFilter<MultiEnvironmentDocumentFilter>();
});
```

### Utilisation de ReDoc comme alternative à Swagger UI

ReDoc est une alternative populaire à Swagger UI qui offre une interface plus élégante et axée sur la documentation. Pour l'utiliser :

#### 1. Installez le package NuGet

```bash
dotnet add package Swashbuckle.AspNetCore.ReDoc
```

#### 2. Configurez ReDoc dans votre application

```csharp
// Dans Configure ou équivalent
app.UseReDoc(c =>
{
    c.SpecUrl = "/swagger/v1/swagger.json";
    c.DocumentTitle = "Mon API - Documentation";
    c.RoutePrefix = "docs"; // Accessible via /docs

    // Options supplémentaires
    c.HideHostname();
    c.HideDownloadButton();
    c.ExpandResponses("200,201");
    c.RequiredPropsFirst();
    c.NoAutoAuth();
    c.PathInMiddlePanel();
    c.HideLoading();
    c.NativeScrollbars();
    c.OnlyRequiredInSamples();
    c.SortPropsAlphabetically();
});
```

Avec cette configuration, votre documentation sera accessible via `/docs` avec une interface ReDoc, tandis que l'interface Swagger UI reste accessible via `/swagger`.

#### 3. Personnalisation de l'apparence de ReDoc

```csharp
app.UseReDoc(c =>
{
    c.SpecUrl = "/swagger/v1/swagger.json";
    c.DocumentTitle = "Mon API - Documentation";
    c.RoutePrefix = "docs";

    // Thème personnalisé
    c.Theme.Colors.Primary = "#1a3c6e";
    c.Theme.Typography.Font = "Open Sans, sans-serif";
    c.Theme.Typography.HeadingsFont = "Montserrat, sans-serif";
    c.Theme.Logo.Gutter = "20px";

    // En-tête personnalisé
    c.HeadContent = @"
        <link href='https://fonts.googleapis.com/css?family=Montserrat:300,400,700|Open+Sans:300,400,700' rel='stylesheet'>
        <style>
            body { margin: 0; padding: 0; }
            .api-info .title { color: #1a3c6e !important; }
        </style>
    ";
});
```

## 12.6.4. Génération de clients API

L'un des grands avantages de Swagger/OpenAPI est la possibilité de générer automatiquement des clients API dans différents langages de programmation. Cela permet aux consommateurs de votre API d'intégrer facilement votre service dans leurs applications.

### Utilisation de NSwag pour générer des clients

NSwag est une bibliothèque puissante pour générer des clients API à partir de spécifications OpenAPI. Elle prend en charge plusieurs langages, notamment C#, TypeScript, JavaScript, etc.

#### 1. Installation des packages NuGet

```bash
dotnet add package NSwag.AspNetCore
dotnet add package NSwag.MSBuild
```

#### 2. Configuration dans le projet

Modifiez votre fichier de projet (.csproj) pour inclure la tâche de génération du client :

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <!-- ... autres éléments ... -->

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <!-- Chemin où sera généré le client API -->
    <OpenApiGenerateClientSdkOnBuild>true</OpenApiGenerateClientSdkOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <!-- ... autres références ... -->
    <PackageReference Include="NSwag.AspNetCore" Version="13.20.0" />
    <PackageReference Include="NSwag.MSBuild" Version="13.20.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>

  <!-- Tâche de génération du client C# -->
  <Target Name="NSwag" AfterTargets="Build" Condition="'$(Configuration)' == 'Debug'">
    <Exec Command="$(NSwagExe_Net80) run nswag.json" />
  </Target>
</Project>
```

#### 3. Création du fichier de configuration NSwag

Créez un fichier `nswag.json` à la racine de votre projet :

```json
{
  "runtime": "Net80",
  "defaultVariables": null,
  "documentGenerator": {
    "fromDocument": {
      "url": "https://localhost:5001/swagger/v1/swagger.json",
      "output": null
    }
  },
  "codeGenerators": {
    "openApiToCSharpClient": {
      "clientBaseClass": null,
      "configurationClass": null,
      "generateClientClasses": true,
      "generateClientInterfaces": true,
      "clientBaseInterface": null,
      "injectHttpClient": true,
      "disposeHttpClient": true,
      "protectedMethods": [],
      "generateExceptionClasses": true,
      "exceptionClass": "ApiException",
      "wrapDtoExceptions": true,
      "useHttpClientCreationMethod": false,
      "httpClientType": "System.Net.Http.HttpClient",
      "useHttpRequestMessageCreationMethod": false,
      "useBaseUrl": true,
      "generateBaseUrlProperty": true,
      "generateSyncMethods": false,
      "generatePrepareRequestAndProcessResponseAsAsyncMethods": false,
      "exposeJsonSerializerSettings": false,
      "clientClassAccessModifier": "public",
      "typeAccessModifier": "public",
      "generateContractsOutput": false,
      "contractsNamespace": null,
      "contractsOutputFilePath": null,
      "namespace": "MonApi.Client",
      "outputFilePath": "Generated/ApiClient.cs",
      "className": "{controller}Client",
      "operationGenerationMode": "SingleClientFromOperationId",
      "additionalNamespaceUsages": [
        "System.Threading.Tasks"
      ],
      "additionalContractNamespaceUsages": [],
      "generateOptionalParameters": false,
      "generateJsonMethods": false,
      "enforceFlagEnums": false,
      "parameterArrayType": "System.Collections.Generic.IEnumerable",
      "parameterDictionaryType": "System.Collections.Generic.IDictionary",
      "responseArrayType": "System.Collections.Generic.ICollection",
      "responseDictionaryType": "System.Collections.Generic.IDictionary",
      "wrapResponses": false,
      "wrapResponseMethods": [],
      "generateResponseClasses": true,
      "responseClass": "SwaggerResponse",
      "requiredPropertiesMustBeDefined": true,
      "dateType": "System.DateTimeOffset",
      "jsonConverters": null,
      "anyType": "object",
      "dateTimeType": "System.DateTimeOffset",
      "timeType": "System.TimeSpan",
      "timeSpanType": "System.TimeSpan",
      "arrayType": "System.Collections.Generic.ICollection",
      "arrayInstanceType": "System.Collections.Generic.List",
      "dictionaryType": "System.Collections.Generic.IDictionary",
      "dictionaryInstanceType": "System.Collections.Generic.Dictionary",
      "arrayBaseType": "System.Collections.ObjectModel.Collection",
      "dictionaryBaseType": "System.Collections.Generic.Dictionary",
      "classStyle": "Poco",
      "generateDefaultValues": true,
      "generateDataAnnotations": true,
      "excludedTypeNames": [],
      "excludedParameterNames": [],
      "handleReferences": false,
      "generateImmutableArrayProperties": false,
      "generateImmutableDictionaryProperties": false,
      "jsonSerializerSettingsTransformationMethod": null,
      "inlineNamedDictionaries": false,
      "inlineNamedTuples": true,
      "generateDtoTypes": true,
      "generateOptionalPropertiesAsNullable": false,
      "templateDirectory": null,
      "typeNameGeneratorType": null,
      "propertyNameGeneratorType": null,
      "enumNameGeneratorType": null,
      "serviceHost": null,
      "serviceSchemes": null,
      "output": "Generated/ApiClient.cs"
    }
  }
}
```

Avec cette configuration, à chaque compilation en mode Debug, un client C# sera généré dans le dossier `Generated` de votre projet.

### Génération de clients TypeScript/Angular

Pour générer un client TypeScript (utilisable dans Angular, React, etc.), modifiez la section `codeGenerators` de votre fichier `nswag.json` :

```json
"codeGenerators": {
  "openApiToTypeScriptClient": {
    "className": "{controller}Client",
    "moduleName": "",
    "namespace": "",
    "typeScriptVersion": 4.3,
    "template": "Angular",
    "promiseType": "Promise",
    "httpClass": "HttpClient",
    "withCredentials": true,
    "useSingletonProvider": true,
    "injectionTokenType": "InjectionToken",
    "rxJsVersion": 7.0,
    "dateTimeType": "Date",
    "nullValue": "undefined",
    "generateClientClasses": true,
    "generateClientInterfaces": true,
    "generateOptionalParameters": false,
    "output": "../client-app/src/app/api/api-client.ts"
  }
}
```

### Utilisation de SwaggerCodegen CLI

Swagger Codegen est un autre outil populaire pour générer des clients à partir de spécifications OpenAPI. Il prend en charge plus de 40 langages.

#### Utilisation via Docker

```bash
# Télécharger l'image Docker
docker pull swaggerapi/swagger-codegen-cli

# Générer un client Java
docker run --rm -v ${PWD}:/local swaggerapi/swagger-codegen-cli generate \
  -i http://localhost:5000/swagger/v1/swagger.json \
  -l java \
  -o /local/generated-clients/java

# Générer un client Python
docker run --rm -v ${PWD}:/local swaggerapi/swagger-codegen-cli generate \
  -i http://localhost:5000/swagger/v1/swagger.json \
  -l python \
  -o /local/generated-clients/python
```

#### Utilisation via npm (OpenAPI Generator)

```bash
# Installation
npm install @openapitools/openapi-generator-cli -g

# Générer un client JavaScript
openapi-generator-cli generate -i http://localhost:5000/swagger/v1/swagger.json -g javascript -o ./generated-clients/javascript

# Générer un client PHP
openapi-generator-cli generate -i http://localhost:5000/swagger/v1/swagger.json -g php -o ./generated-clients/php
```

### Génération de clients dans Visual Studio

Si vous utilisez Visual Studio, vous pouvez également générer des clients directement depuis l'interface :

1. Cliquez avec le bouton droit sur votre projet
2. Sélectionnez "Ajouter" > "Service connecté"
3. Choisissez "OpenAPI"
4. Saisissez l'URL de votre document OpenAPI (ex: https://localhost:5001/swagger/v1/swagger.json)
5. Configurez les options et cliquez sur "Terminer"

Visual Studio générera un client C# dans votre projet avec toutes les classes nécessaires.

### Personnalisation des clients générés

Vous pouvez personnaliser les clients générés en modifiant les paramètres de génération. Par exemple, pour NSwag :

```json
"openApiToCSharpClient": {
  // ...configuration de base...

  // Personnalisations avancées
  "useHttpRequestMessage": true,
  "generateUpdateJsonSerializerSettingsMethod": true,
  "serializeTypeInformation": false,
  "generateJsonMethods": true,
  "generateClientClasses": true,
  "generateDtoTypes": true,
  "injectHttpClient": true,
  "disposeHttpClient": false,
  "protectedMethods": [],

  // Nom de classe personnalisé
  "className": "MonApiServiceClient",

  // Options de génération pour HTTP
  "useHttpClientCreationMethod": true,
  "clientBaseClass": "BaseClient",
  "configurationClass": "MonApiClientConfiguration",

  // Modèles personnalisés
  "templateDirectory": "./Templates",

  // Options pour les exceptions
  "exceptionClass": "MonApiException",
  "generateExceptionClasses": true,
  "wrapDtoExceptions": true,
  "exceptionSchema": "{controller}Exception"
}
```

## 12.6.5. Intégration avec autres outils

Swagger/OpenAPI peut être intégré avec plusieurs autres outils pour améliorer votre flux de travail de développement d'API.

### Intégration avec Postman

Postman est un outil populaire pour tester les API REST. Vous pouvez facilement importer votre spécification OpenAPI dans Postman :

#### 1. Exportation de la spécification OpenAPI

Si vous utilisez Swagger UI, vous pouvez télécharger le fichier JSON depuis l'interface ou directement depuis l'URL `/swagger/v1/swagger.json`.

#### 2. Importation dans Postman

1. Ouvrez Postman
2. Cliquez sur "Import" dans le menu supérieur
3. Sélectionnez "File" ou "Link" (si vous utilisez l'URL directe)
4. Choisissez le fichier JSON ou entrez l'URL (ex: https://votre-api.com/swagger/v1/swagger.json)
5. Cliquez sur "Import"

Postman créera automatiquement une collection avec tous vos endpoints, organisés par tags et avec les paramètres pré-configurés.

#### 3. Génération de code depuis Postman

Une fois importée, vous pouvez également générer du code client à partir de Postman :

1. Ouvrez un endpoint dans la collection
2. Cliquez sur "</> Code" dans le coin droit
3. Sélectionnez le langage souhaité (JavaScript, C#, Python, etc.)
4. Copiez le code généré

### Intégration avec Jenkins/Azure DevOps/GitHub Actions

Vous pouvez intégrer la validation de votre documentation API dans vos pipelines CI/CD :

#### Exemple avec GitHub Actions

Créez un fichier `.github/workflows/validate-openapi.yml` :

```yaml
name: Validate OpenAPI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Install Swagger CLI
      run: npm install -g @apidevtools/swagger-cli

    - name: Run API and generate Swagger JSON
      run: |
        dotnet build
        dotnet run --project src/MonApi/MonApi.csproj &
        sleep 10  # Attendre que l'API démarre
        curl -o swagger.json http://localhost:5000/swagger/v1/swagger.json

    - name: Validate Swagger JSON
      run: swagger-cli validate swagger.json

    - name: Check for breaking changes (if PR)
      if: github.event_name == 'pull_request'
      run: |
        curl -o main-swagger.json https://api.monentreprise.com/swagger/v1/swagger.json
        npm install -g swagger-diff
        swagger-diff main-swagger.json swagger.json --fail-on-incompatible
```

#### Exemple avec Azure DevOps

```yaml
# azure-pipelines.yml
trigger:
- main
- develop

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '16.x'
  displayName: 'Install Node.js'

- script: |
    npm install -g @apidevtools/swagger-cli
  displayName: 'Install Swagger CLI'

- script: |
    dotnet build
    dotnet run --project src/MonApi/MonApi.csproj &
    sleep 10
    curl -o swagger.json http://localhost:5000/swagger/v1/swagger.json
  displayName: 'Generate Swagger JSON'

- script: |
    swagger-cli validate swagger.json
  displayName: 'Validate Swagger JSON'

- task: PublishBuildArtifacts@1
  inputs:
    pathtoPublish: 'swagger.json'
    artifactName: 'api-documentation'
  displayName: 'Publish API Documentation'
```

### Génération de documentation PDF

Vous pouvez générer des fichiers PDF à partir de votre documentation Swagger pour des distributions hors ligne :

#### 1. Installation de wkhtmltopdf

Installez d'abord l'outil [wkhtmltopdf](https://wkhtmltopdf.org/downloads.html).

#### 2. Création d'un contrôleur pour générer le PDF

```csharp
[ApiExplorerSettings(IgnoreApi = true)]
[Route("api/documentation")]
[ApiController]
public class DocumentationController : ControllerBase
{
    [HttpGet("pdf")]
    public async Task<IActionResult> GeneratePdf()
    {
        var baseUrl = $"{Request.Scheme}://{Request.Host}";
        var redocUrl = $"{baseUrl}/docs";

        // Chemin de sortie pour le PDF
        var outputPath = Path.Combine(Path.GetTempPath(), "api-documentation.pdf");

        try
        {
            // Configuration du processus wkhtmltopdf
            var process = new Process
            {
                StartInfo = new ProcessStartInfo
                {
                    FileName = "wkhtmltopdf",
                    Arguments = $"--enable-javascript --javascript-delay 2000 \"{redocUrl}\" \"{outputPath}\"",
                    RedirectStandardOutput = true,
                    RedirectStandardError = true,
                    UseShellExecute = false,
                    CreateNoWindow = true
                }
            };

            // Démarrer le processus et attendre qu'il se termine
            process.Start();
            string output = await process.StandardOutput.ReadToEndAsync();
            string error = await process.StandardError.ReadToEndAsync();
            await process.WaitForExitAsync();

            // Vérifier si le processus a réussi
            if (process.ExitCode != 0)
            {
                return StatusCode(500, $"Erreur lors de la génération du PDF: {error}");
            }

            // Lire le fichier PDF
            var pdfBytes = await System.IO.File.ReadAllBytesAsync(outputPath);

            // Nettoyer les fichiers temporaires
            System.IO.File.Delete(outputPath);

            // Retourner le PDF
            return File(pdfBytes, "application/pdf", "API-Documentation.pdf");
        }
        catch (Exception ex)
        {
            return StatusCode(500, $"Une erreur s'est produite: {ex.Message}");
        }
    }
}
```

#### 3. Ajout d'un bouton dans Swagger UI pour télécharger le PDF

Dans votre fichier JavaScript personnalisé (`wwwroot/swagger-ui/custom.js`), ajoutez le code suivant :

```javascript
window.onload = function() {
    setTimeout(function() {
        // Trouver l'élément topbar de Swagger UI
        var topbarElement = document.getElementsByClassName('topbar')[0];
        if (topbarElement) {
            // Créer un bouton pour télécharger la documentation PDF
            var pdfButton = document.createElement('a');
            pdfButton.className = 'btn download-btn';
            pdfButton.innerText = 'Télécharger PDF';
            pdfButton.href = '/api/documentation/pdf';
            pdfButton.target = '_blank';
            pdfButton.style.marginRight = '10px';
            pdfButton.style.backgroundColor = '#1a3c6e';
            pdfButton.style.color = 'white';
            pdfButton.style.padding = '6px 12px';
            pdfButton.style.borderRadius = '4px';
            pdfButton.style.textDecoration = 'none';

            // Ajouter le bouton à la barre supérieure
            topbarElement.appendChild(pdfButton);
        }
    }, 1000); // Attendre 1 seconde pour s'assurer que Swagger UI est chargé
};
```

#### 4. Approche alternative avec DinkToPdf

Si vous préférez ne pas dépendre d'un exécutable externe, vous pouvez utiliser la bibliothèque `DinkToPdf` pour générer des PDF directement dans votre API :

```csharp
// Installez d'abord le package NuGet
// dotnet add package DinkToPdf

[ApiExplorerSettings(IgnoreApi = true)]
[Route("api/documentation")]
[ApiController]
public class DocumentationController : ControllerBase
{
    private readonly IConverter _converter;

    public DocumentationController(IConverter converter)
    {
        _converter = converter;
    }

    [HttpGet("pdf")]
    public IActionResult GeneratePdf()
    {
        var baseUrl = $"{Request.Scheme}://{Request.Host}";
        var redocUrl = $"{baseUrl}/docs";

        // Créer un WebClient pour obtenir le contenu HTML de la page ReDoc
        string htmlContent;
        using (var client = new WebClient())
        {
            htmlContent = client.DownloadString(redocUrl);
        }

        // Configuration pour la conversion en PDF
        var doc = new HtmlToPdfDocument()
        {
            GlobalSettings = {
                ColorMode = ColorMode.Color,
                Orientation = Orientation.Portrait,
                PaperSize = PaperKind.A4,
                Margins = new MarginSettings { Top = 10, Bottom = 10, Left = 10, Right = 10 },
                DocumentTitle = "Documentation API"
            },
            Objects = {
                new ObjectSettings {
                    PagesCount = true,
                    HtmlContent = htmlContent,
                    WebSettings = {
                        DefaultEncoding = "utf-8",
                        EnableJavascript = true,
                        LoadImages = true,
                        EnableIntelligentShrinking = true,
                        JavascriptDelay = 2000, // Attendre 2 secondes pour que JS s'exécute
                    },
                    HeaderSettings = {
                        FontName = "Arial",
                        FontSize = 9,
                        Right = "Page [page] sur [toPage]",
                        Line = true
                    },
                    FooterSettings = {
                        FontName = "Arial",
                        FontSize = 9,
                        Line = true,
                        Center = "Documentation générée le " + DateTime.Now.ToString("dd/MM/yyyy")
                    }
                }
            }
        };

        // Convertir en PDF
        var pdfBytes = _converter.Convert(doc);

        // Retourner le PDF
        return File(pdfBytes, "application/pdf", "API-Documentation.pdf");
    }
}
```

N'oubliez pas d'enregistrer le service dans votre `Startup.cs` ou `Program.cs` :

```csharp
// Pour .NET 6 et versions ultérieures
builder.Services.AddSingleton(typeof(IConverter), new SynchronizedConverter(new PdfTools()));

// Pour les versions antérieures
services.AddSingleton(typeof(IConverter), new SynchronizedConverter(new PdfTools()));
```

### Génération de maquettes API (Mock Server)

Vous pouvez utiliser votre spécification OpenAPI pour générer un serveur de maquettes qui renvoie des réponses simulées basées sur vos exemples et schémas.

#### 1. Utilisation de Prism

Prism est un outil populaire pour créer des maquettes d'API à partir de spécifications OpenAPI :

```bash
# Installation
npm install -g @stoplight/prism-cli

# Démarrer un serveur de maquettes
prism mock https://localhost:5001/swagger/v1/swagger.json

# Avec des exemples dynamiques générés
prism mock https://localhost:5001/swagger/v1/swagger.json --dynamic
```

Vous pouvez maintenant faire des requêtes à `http://localhost:4010` et recevoir des réponses simulées basées sur votre documentation.

#### 2. Intégration d'un serveur de maquettes dans votre API

Vous pouvez également intégrer un serveur de maquettes directement dans votre API pour des environnements de développement ou de test :

```csharp
public class MockResponseMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IWebHostEnvironment _env;

    public MockResponseMiddleware(RequestDelegate next, IWebHostEnvironment env)
    {
        _next = next;
        _env = env;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Vérifier si la requête est destinée au serveur de maquettes
        if (context.Request.Path.StartsWithSegments("/api/mock"))
        {
            // Extraire le chemin réel de l'API
            var actualPath = context.Request.Path.Value.Replace("/api/mock", "/api");
            var method = context.Request.Method;

            // Vérifier si nous sommes en environnement de développement
            if (_env.IsDevelopment())
            {
                // Charger les exemples depuis un fichier JSON
                var mockDataPath = Path.Combine(_env.ContentRootPath, "MockData", "responses.json");
                if (System.IO.File.Exists(mockDataPath))
                {
                    var mockData = JsonSerializer.Deserialize<Dictionary<string, Dictionary<string, object>>>(
                        await System.IO.File.ReadAllTextAsync(mockDataPath)
                    );

                    // Rechercher la réponse pour ce chemin et cette méthode
                    var key = $"{method}:{actualPath}";
                    if (mockData.ContainsKey(key))
                    {
                        context.Response.ContentType = "application/json";
                        context.Response.StatusCode = 200;
                        await context.Response.WriteAsync(JsonSerializer.Serialize(mockData[key]));
                        return;
                    }
                }

                // Réponse par défaut si aucun exemple n'est trouvé
                context.Response.ContentType = "application/json";
                context.Response.StatusCode = 200;
                await context.Response.WriteAsync("{\"message\": \"Mock response not configured for this endpoint\"}");
                return;
            }
        }

        // Passer à l'étape suivante du pipeline si ce n'est pas une requête de maquette
        await _next(context);
    }
}

// Enregistrer le middleware dans Program.cs ou Startup.cs
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api/mock"),
    appBuilder => appBuilder.UseMiddleware<MockResponseMiddleware>()
);
```

### Surveillance et gouvernance des API

La spécification OpenAPI peut également être utilisée pour surveiller les changements dans votre API et assurer une gouvernance appropriée.

#### 1. Détection des changements d'API avec Swagger Diff

Swagger Diff est un outil qui permet de comparer deux spécifications OpenAPI et de détecter les changements entre elles :

```bash
# Installation
npm install -g swagger-diff

# Comparaison entre deux versions
swagger-diff https://api.exemple.com/v1/swagger.json https://api.exemple.com/v2/swagger.json

# Vérifier les changements qui cassent la compatibilité
swagger-diff https://api.exemple.com/v1/swagger.json https://api.exemple.com/v2/swagger.json --fail-on-incompatible
```

#### 2. Intégration dans le processus de revue de code

Vous pouvez intégrer cette vérification dans votre processus de revue de code pour éviter les changements qui cassent la compatibilité :

```yaml
# .github/workflows/api-compatibility.yml
name: API Compatibility Check

on:
  pull_request:
    branches: [ main ]
    paths:
      - 'src/Api/**'

jobs:
  check-compatibility:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'

    - name: Install Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'

    - name: Install swagger-diff
      run: npm install -g swagger-diff

    - name: Build and Run API
      run: |
        dotnet build src/Api/Api.csproj
        dotnet run --project src/Api/Api.csproj --urls=http://localhost:5000 &
        sleep 10

    - name: Get current API specification
      run: curl -o new-swagger.json http://localhost:5000/swagger/v1/swagger.json

    - name: Get production API specification
      run: curl -o prod-swagger.json https://api.production.com/swagger/v1/swagger.json

    - name: Compare API specifications
      run: |
        swagger-diff prod-swagger.json new-swagger.json > diff.txt
        cat diff.txt
        if grep -q "BREAKING" diff.txt; then
          echo "Breaking changes detected!"
          exit 1
        fi
```

### Intégration avec des outils de conception d'API

Plusieurs outils de conception d'API prennent en charge la spécification OpenAPI et peuvent être intégrés dans votre flux de travail.

#### 1. Swagger Editor

Swagger Editor est un éditeur en ligne pour les spécifications OpenAPI. Vous pouvez l'intégrer dans votre application :

```csharp
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon API V1");

    // Ajouter un lien vers Swagger Editor
    c.HeadContent += @"
        <style>
            .topbar-wrapper .link {
                display: flex;
                align-items: center;
                margin-right: 10px;
            }
        </style>
        <script>
            setTimeout(function() {
                const topbarEl = document.getElementsByClassName('topbar')[0];
                if (topbarEl) {
                    const linkEl = document.createElement('a');
                    linkEl.classList.add('link');
                    linkEl.setAttribute('href', 'https://editor.swagger.io/?url=' + window.location.origin + '/swagger/v1/swagger.json');
                    linkEl.setAttribute('target', '_blank');
                    linkEl.innerText = 'Ouvrir dans Swagger Editor';

                    const downloadUrlWrapperEl = document.getElementsByClassName('download-url-wrapper')[0];
                    topbarEl.insertBefore(linkEl, downloadUrlWrapperEl);
                }
            }, 1000);
        </script>";
});
```

#### 2. Intégration avec Stoplight Studio

Stoplight Studio est un outil de conception d'API qui prend en charge OpenAPI. Vous pouvez exporter votre spécification vers Stoplight :

```csharp
[ApiExplorerSettings(IgnoreApi = true)]
[Route("api/documentation")]
[ApiController]
public class DocumentationController : ControllerBase
{
    [HttpGet("export/stoplight")]
    public IActionResult ExportToStoplight()
    {
        var baseUrl = $"{Request.Scheme}://{Request.Host}";
        var swaggerUrl = $"{baseUrl}/swagger/v1/swagger.json";

        var stoplightUrl = $"https://stoplight.io/api/v1/import?url={Uri.EscapeDataString(swaggerUrl)}&name=MonAPI";

        return Redirect(stoplightUrl);
    }
}
```

Vous pouvez ajouter un bouton dans Swagger UI pour exporter vers Stoplight :

```javascript
// Dans custom.js
setTimeout(function() {
    var topbarElement = document.getElementsByClassName('topbar')[0];
    if (topbarElement) {
        var stoplightBtn = document.createElement('a');
        stoplightBtn.className = 'btn stoplight-btn';
        stoplightBtn.innerText = 'Éditer dans Stoplight';
        stoplightBtn.href = '/api/documentation/export/stoplight';
        stoplightBtn.target = '_blank';
        stoplightBtn.style.marginRight = '10px';
        stoplightBtn.style.backgroundColor = '#6c47db';
        stoplightBtn.style.color = 'white';

        topbarElement.appendChild(stoplightBtn);
    }
}, 1000);
```

### Génération de tests automatisés

Vous pouvez utiliser votre spécification OpenAPI pour générer des tests d'API automatisés :

#### 1. Avec Dredd

Dredd est un outil qui permet de tester vos API en fonction de leur documentation :

```bash
# Installation
npm install -g dredd

# Initialiser la configuration
dredd init https://localhost:5001/swagger/v1/swagger.json http://localhost:5000

# Exécuter les tests
dredd
```

#### 2. Avec Postman/Newman

```bash
# Installation de Newman
npm install -g newman

# Convertir la spécification OpenAPI en collection Postman
npm install -g openapi-to-postman

openapi-to-postman convert -s https://localhost:5001/swagger/v1/swagger.json -o postman-collection.json

# Exécuter les tests avec Newman
newman run postman-collection.json -e environment.json
```

### API Gateway et gestion des API

Si vous utilisez un API Gateway comme Kong, AWS API Gateway ou Azure API Management, vous pouvez importer votre spécification OpenAPI :

#### 1. Azure API Management

Vous pouvez automatiser l'import avec Azure CLI :

```bash
# Importer la spécification
az apim api import --path /api --resource-group MonGroupe --service-name MonService --api-id mon-api --specification-url https://localhost:5001/swagger/v1/swagger.json --specification-format OpenApi

# Créer un produit pour l'API
az apim product api add --api-id mon-api --product-id illimité --resource-group MonGroupe --service-name MonService
```

#### 2. AWS API Gateway

Avec AWS CLI :

```bash
# Créer une API à partir de la spécification
aws apigateway import-rest-api --body file://swagger.json --region eu-west-1

# Déployer l'API
aws apigateway create-deployment --rest-api-id abcde1234 --stage-name prod
```

#### 3. Kong

Avec l'API déclarative de Kong :

```bash
curl -X POST http://kong:8001/services \
  --data name=mon-api \
  --data url=http://backend-service:8000

curl -X POST http://kong:8001/services/mon-api/routes \
  --data paths[]=/api

curl -X POST http://kong:8001/services/mon-api/plugins \
  --data name=openapi-validator \
  --data config.spec_url=https://localhost:5001/swagger/v1/swagger.json
```

## Bonnes pratiques pour la documentation API

La documentation est souvent négligée, mais elle est essentielle pour l'expérience des développeurs qui utilisent votre API. Voici quelques bonnes pratiques :

### 1. Documenter de manière exhaustive

- Documentez tous les endpoints, paramètres, en-têtes et réponses possibles
- Incluez des exemples réalistes pour chaque requête et réponse
- Décrivez clairement les erreurs possibles et leurs significations
- Utilisez une terminologie cohérente dans toute la documentation

### 2. Maintenir la documentation à jour

- Intégrez la mise à jour de la documentation dans votre processus de développement
- Utilisez les tests de CI/CD pour vérifier que la documentation correspond au code
- Versionnez votre documentation comme vous versionnez votre API

### 3. Organiser logiquement

- Groupez les endpoints par fonctionnalité ou ressource
- Utilisez des tags pour faciliter la navigation
- Suivez une structure cohérente pour chaque endpoint

### 4. Inclure des informations de haut niveau

- Ajoutez une introduction qui explique l'objectif de l'API
- Documentez le processus d'authentification et d'autorisation
- Incluez des guides de démarrage rapide et d'intégration
- Mentionnez les limites de taux, quotas ou autres restrictions

### 5. Appliquer les principes RESTful dans la documentation

- Utilisez les verbes HTTP de manière cohérente (GET, POST, PUT, DELETE)
- Respectez la hiérarchie des ressources dans vos URLs
- Documentez les relations entre les ressources

### 6. Penser aux performances

- Activez Swagger/ReDoc uniquement en développement par défaut
- Utilisez la mise en cache pour les documents OpenAPI
- Considérez l'impact sur les performances de l'application

### 7. Sécuriser votre documentation

- Ne pas exposer d'informations sensibles dans la documentation
- Protéger l'accès à la documentation en production si nécessaire
- Filtrer les endpoints internes de la documentation publique

## Conclusion

La documentation API avec Swagger/OpenAPI est un investissement qui porte ses fruits en améliorant la découvrabilité, l'utilisabilité et la maintenance de votre API. En suivant les meilleures pratiques présentées dans ce chapitre, vous pouvez créer une documentation complète, précise et utile qui améliore l'expérience des développeurs.

Une bonne documentation n'est pas simplement un complément facultatif, mais un composant essentiel d'une API bien conçue. Elle réduit le temps de support, facilite l'intégration et encourage l'adoption de votre API.

La documentation Swagger/OpenAPI offre également des avantages au-delà de la simple documentation, comme la génération automatique de clients, la création de maquettes et l'intégration avec des outils tiers. Ces fonctionnalités peuvent considérablement accélérer votre cycle de développement et améliorer la qualité de votre API.

En investissant du temps dans une documentation de qualité avec Swagger/OpenAPI, vous investissez dans le succès à long terme de votre API.

## Ressources supplémentaires

- [Documentation officielle de Swagger](https://swagger.io/docs/)
- [Documentation officielle de Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)
- [Documentation officielle de NSwag](https://github.com/RicoSuter/NSwag)
- [OpenAPI Initiative](https://www.openapis.org/)
- [Tutoriels Swagger sur Microsoft Learn](https://learn.microsoft.com/fr-fr/aspnet/core/tutorials/web-api-help-pages-using-swagger)
- [ReDoc - Alternative à Swagger UI](https://github.com/Redocly/redoc)
- [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)
- [Stoplight - Plateforme de conception API](https://stoplight.io/)

⏭️
