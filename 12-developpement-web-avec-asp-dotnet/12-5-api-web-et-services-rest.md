# 12.5. API Web et services REST

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 12.5.1. Création d'API RESTful

### Qu'est-ce qu'une API REST ?

Une API REST (Representational State Transfer) est un style d'architecture pour construire des services web. Elle utilise les méthodes HTTP standard et est devenue la méthode dominante pour créer des interfaces de programmation sur le web.

#### Principes fondamentaux de REST

1. **Architecture client-serveur** : Séparation claire entre client et serveur
2. **Sans état (Stateless)** : Chaque requête du client contient toutes les informations nécessaires
3. **Mise en cache** : Les réponses doivent indiquer si elles peuvent être mises en cache
4. **Interface uniforme** : Identification des ressources, manipulation par représentations, messages auto-descriptifs
5. **Système en couches** : Le client ne sait pas s'il communique directement avec le serveur ou un intermédiaire

#### Ressources et identifiants

Dans REST, tout est considéré comme une ressource, identifiée par une URL (Uniform Resource Locator) :

- `/api/products` : Collection de produits
- `/api/products/5` : Produit spécifique avec ID 5
- `/api/customers/10/orders` : Commandes du client avec ID 10

#### Méthodes HTTP et opérations CRUD

REST utilise les méthodes HTTP pour différentes opérations sur les ressources :

| Méthode HTTP | Opération CRUD | Description | Exemple |
|--------------|----------------|-------------|---------|
| GET | Read | Récupérer une ressource | `GET /api/products/5` |
| POST | Create | Créer une nouvelle ressource | `POST /api/products` |
| PUT | Update | Mettre à jour une ressource existante (complète) | `PUT /api/products/5` |
| PATCH | Update | Mettre à jour partiellement une ressource | `PATCH /api/products/5` |
| DELETE | Delete | Supprimer une ressource | `DELETE /api/products/5` |

#### Codes de statut HTTP

Les API REST utilisent les codes de statut HTTP pour indiquer le résultat d'une opération :

| Code | Catégorie | Description | Exemple d'utilisation |
|------|-----------|-------------|----------------------|
| 200 | Success | OK | Requête GET réussie |
| 201 | Success | Created | Ressource créée avec succès (POST) |
| 204 | Success | No Content | Requête réussie sans contenu à renvoyer (DELETE) |
| 400 | Client Error | Bad Request | Requête mal formulée |
| 401 | Client Error | Unauthorized | Authentification nécessaire |
| 403 | Client Error | Forbidden | Authentifié mais non autorisé |
| 404 | Client Error | Not Found | Ressource non trouvée |
| 500 | Server Error | Internal Server Error | Erreur côté serveur |

### Création d'une API REST avec ASP.NET Core

#### 1. Configurer le projet

Pour créer une API REST avec ASP.NET Core, commencez par créer un projet :

```bash
dotnet new webapi -n MonApi
```

Ou utilisez Visual Studio avec le modèle "ASP.NET Core Web API".

#### 2. Structure du projet

Un projet API REST typique avec ASP.NET Core a cette structure :

```
MonApi/
  ├─ Controllers/        // Contrôleurs API
  │   ├─ ProductsController.cs
  │   ├─ CustomersController.cs
  ├─ Models/            // Modèles de données
  │   ├─ Product.cs
  │   ├─ Customer.cs
  ├─ DTOs/              // Objets de transfert de données
  │   ├─ ProductDto.cs
  │   ├─ CustomerDto.cs
  ├─ Services/          // Services métier
  │   ├─ IProductService.cs
  │   ├─ ProductService.cs
  ├─ Data/              // Accès aux données (si applicable)
  │   ├─ ApplicationDbContext.cs
  ├─ Program.cs         // Point d'entrée et configuration
  ├─ appsettings.json   // Configuration
```

#### 3. Configuration dans Program.cs

Voici un exemple de configuration pour une API REST dans ASP.NET Core :

```csharp
var builder = WebApplication.CreateBuilder(args);

// Ajouter les services nécessaires au conteneur
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Configuration de la sérialisation JSON
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.WriteIndented = true;
    });

// Ajouter le support de la documentation API avec Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Ajouter des services personnalisés
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configurer le pipeline de requêtes HTTP
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();

// Configuration des routes d'API
app.MapControllers();

app.Run();
```

#### 4. Création des modèles

Commencez par définir les modèles de données qui représentent vos ressources :

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public bool IsAvailable { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}
```

#### 5. DTOs (Data Transfer Objects)

Les DTOs sont utilisés pour transférer des données entre l'API et le client, sans exposer directement les modèles internes :

```csharp
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public bool IsAvailable { get; set; }
}

public class CreateProductDto
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [StringLength(500)]
    public string Description { get; set; }

    [Range(0.01, 10000)]
    public decimal Price { get; set; }

    public bool IsAvailable { get; set; } = true;
}

public class UpdateProductDto
{
    [StringLength(100)]
    public string Name { get; set; }

    [StringLength(500)]
    public string Description { get; set; }

    [Range(0.01, 10000)]
    public decimal? Price { get; set; }

    public bool? IsAvailable { get; set; }
}
```

#### 6. Services métier

Créez des services pour encapsuler la logique métier :

```csharp
public interface IProductService
{
    Task<IEnumerable<Product>> GetAllProductsAsync();
    Task<Product> GetProductByIdAsync(int id);
    Task<Product> CreateProductAsync(Product product);
    Task<Product> UpdateProductAsync(int id, Product product);
    Task<bool> DeleteProductAsync(int id);
}

public class ProductService : IProductService
{
    private readonly ApplicationDbContext _context;

    public ProductService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Product>> GetAllProductsAsync()
    {
        return await _context.Products.ToListAsync();
    }

    public async Task<Product> GetProductByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }

    public async Task<Product> CreateProductAsync(Product product)
    {
        product.CreatedAt = DateTime.UtcNow;

        _context.Products.Add(product);
        await _context.SaveChangesAsync();

        return product;
    }

    public async Task<Product> UpdateProductAsync(int id, Product product)
    {
        var existingProduct = await _context.Products.FindAsync(id);

        if (existingProduct == null)
            return null;

        // Mise à jour des propriétés
        existingProduct.Name = product.Name;
        existingProduct.Description = product.Description;
        existingProduct.Price = product.Price;
        existingProduct.IsAvailable = product.IsAvailable;
        existingProduct.UpdatedAt = DateTime.UtcNow;

        _context.Products.Update(existingProduct);
        await _context.SaveChangesAsync();

        return existingProduct;
    }

    public async Task<bool> DeleteProductAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);

        if (product == null)
            return false;

        _context.Products.Remove(product);
        await _context.SaveChangesAsync();

        return true;
    }
}
```

## 12.5.2. Controllers API

Les contrôleurs API sont au cœur de votre API REST. Ils définissent les points de terminaison (endpoints) et implémentent la logique de traitement des requêtes HTTP.

### Création d'un contrôleur API de base

Dans ASP.NET Core, un contrôleur API hérite généralement de `ControllerBase` (et non de `Controller` qui est pour les applications MVC avec des vues) :

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(IProductService productService, ILogger<ProductsController> logger)
    {
        _productService = productService;
        _logger = logger;
    }

    // Les actions seront définies ici
}
```

L'attribut `[ApiController]` active plusieurs comportements spécifiques aux API, comme la validation automatique du modèle et l'inférence des paramètres de liaison.

L'attribut `[Route("api/[controller]")]` définit le préfixe d'URL pour toutes les actions de ce contrôleur. `[controller]` est remplacé par le nom du contrôleur sans le suffixe "Controller", donc ici "Products".

### Actions API et méthodes HTTP

Voici comment implémenter les différentes actions CRUD dans un contrôleur API :

#### GET - Récupérer toutes les ressources

```csharp
// GET: api/products
[HttpGet]
public async Task<ActionResult<IEnumerable<ProductDto>>> GetProducts()
{
    try
    {
        var products = await _productService.GetAllProductsAsync();

        // Mapper les produits vers les DTOs
        var productDtos = products.Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Description = p.Description,
            Price = p.Price,
            IsAvailable = p.IsAvailable
        });

        return Ok(productDtos);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error retrieving products");
        return StatusCode(500, "Une erreur est survenue lors de la récupération des produits.");
    }
}
```

#### GET - Récupérer une ressource spécifique

```csharp
// GET: api/products/5
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    try
    {
        var product = await _productService.GetProductByIdAsync(id);

        if (product == null)
        {
            return NotFound($"Produit avec ID {id} non trouvé.");
        }

        var productDto = new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            IsAvailable = product.IsAvailable
        };

        return Ok(productDto);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error retrieving product {ProductId}", id);
        return StatusCode(500, "Une erreur est survenue lors de la récupération du produit.");
    }
}
```

#### POST - Créer une ressource

```csharp
// POST: api/products
[HttpPost]
public async Task<ActionResult<ProductDto>> CreateProduct(CreateProductDto createProductDto)
{
    try
    {
        // ModelState est validé automatiquement grâce à [ApiController]

        var product = new Product
        {
            Name = createProductDto.Name,
            Description = createProductDto.Description,
            Price = createProductDto.Price,
            IsAvailable = createProductDto.IsAvailable
        };

        var createdProduct = await _productService.CreateProductAsync(product);

        var productDto = new ProductDto
        {
            Id = createdProduct.Id,
            Name = createdProduct.Name,
            Description = createdProduct.Description,
            Price = createdProduct.Price,
            IsAvailable = createdProduct.IsAvailable
        };

        // Retourne 201 Created avec l'URL de la nouvelle ressource
        return CreatedAtAction(nameof(GetProduct), new { id = productDto.Id }, productDto);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error creating product");
        return StatusCode(500, "Une erreur est survenue lors de la création du produit.");
    }
}
```

#### PUT - Mettre à jour une ressource

```csharp
// PUT: api/products/5
[HttpPut("{id}")]
public async Task<IActionResult> UpdateProduct(int id, UpdateProductDto updateProductDto)
{
    try
    {
        var existingProduct = await _productService.GetProductByIdAsync(id);

        if (existingProduct == null)
        {
            return NotFound($"Produit avec ID {id} non trouvé.");
        }

        // Mise à jour des propriétés si elles sont fournies
        if (updateProductDto.Name != null)
            existingProduct.Name = updateProductDto.Name;

        if (updateProductDto.Description != null)
            existingProduct.Description = updateProductDto.Description;

        if (updateProductDto.Price.HasValue)
            existingProduct.Price = updateProductDto.Price.Value;

        if (updateProductDto.IsAvailable.HasValue)
            existingProduct.IsAvailable = updateProductDto.IsAvailable.Value;

        var updatedProduct = await _productService.UpdateProductAsync(id, existingProduct);

        return NoContent(); // 204 No Content
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error updating product {ProductId}", id);
        return StatusCode(500, "Une erreur est survenue lors de la mise à jour du produit.");
    }
}
```

#### DELETE - Supprimer une ressource

```csharp
// DELETE: api/products/5
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteProduct(int id)
{
    try
    {
        var result = await _productService.DeleteProductAsync(id);

        if (!result)
        {
            return NotFound($"Produit avec ID {id} non trouvé.");
        }

        return NoContent(); // 204 No Content
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error deleting product {ProductId}", id);
        return StatusCode(500, "Une erreur est survenue lors de la suppression du produit.");
    }
}
```

### Gestion des retours (ActionResult)

ASP.NET Core offre plusieurs types de `ActionResult` pour les API :

| Type de retour | Description | Méthode d'assistance |
|---------------|-------------|---------------------|
| Ok | 200 OK avec un corps de réponse | `return Ok(data);` |
| Created | 201 Created avec un corps de réponse et URL de la ressource | `return CreatedAtAction(...);` |
| Accepted | 202 Accepted avec un corps de réponse | `return Accepted(data);` |
| NoContent | 204 No Content sans corps de réponse | `return NoContent();` |
| BadRequest | 400 Bad Request | `return BadRequest("Message");` |
| Unauthorized | 401 Unauthorized | `return Unauthorized();` |
| Forbidden | 403 Forbidden | `return Forbid();` |
| NotFound | 404 Not Found | `return NotFound("Message");` |
| StatusCode | Code de statut personnalisé | `return StatusCode(418, "I'm a teapot");` |

### Contrôleurs API avec Entity Framework Core

Voici un exemple plus complet d'un contrôleur API utilisant Entity Framework Core directement (sans couche de service) :

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<CustomersController> _logger;

    public CustomersController(ApplicationDbContext context, ILogger<CustomersController> logger)
    {
        _context = context;
        _logger = logger;
    }

    // GET: api/customers
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Customer>>> GetCustomers()
    {
        return await _context.Customers.ToListAsync();
    }

    // GET: api/customers/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Customer>> GetCustomer(int id)
    {
        var customer = await _context.Customers.FindAsync(id);

        if (customer == null)
        {
            return NotFound();
        }

        return customer;
    }

    // GET: api/customers/5/orders
    [HttpGet("{id}/orders")]
    public async Task<ActionResult<IEnumerable<Order>>> GetCustomerOrders(int id)
    {
        var customerExists = await _context.Customers.AnyAsync(c => c.Id == id);

        if (!customerExists)
        {
            return NotFound($"Client avec ID {id} non trouvé.");
        }

        var orders = await _context.Orders
            .Where(o => o.CustomerId == id)
            .ToListAsync();

        return orders;
    }

    // POST: api/customers
    [HttpPost]
    public async Task<ActionResult<Customer>> CreateCustomer(Customer customer)
    {
        _context.Customers.Add(customer);
        await _context.SaveChangesAsync();

        return CreatedAtAction(nameof(GetCustomer), new { id = customer.Id }, customer);
    }

    // PUT: api/customers/5
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateCustomer(int id, Customer customer)
    {
        if (id != customer.Id)
        {
            return BadRequest("L'ID dans l'URL ne correspond pas à l'ID dans le corps de la requête.");
        }

        _context.Entry(customer).State = EntityState.Modified;

        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!CustomerExists(id))
            {
                return NotFound();
            }
            else
            {
                throw;
            }
        }

        return NoContent();
    }

    // DELETE: api/customers/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteCustomer(int id)
    {
        var customer = await _context.Customers.FindAsync(id);

        if (customer == null)
        {
            return NotFound();
        }

        _context.Customers.Remove(customer);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    private bool CustomerExists(int id)
    {
        return _context.Customers.Any(e => e.Id == id);
    }
}
```

### Contrôleurs minimaux (Minimal API)

À partir de .NET 6, ASP.NET Core prend en charge les "API minimales" qui permettent de définir des endpoints sans utiliser de contrôleurs :

```csharp
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);

// Services...
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddDbContext<ApplicationDbContext>();

var app = builder.Build();

// Middleware...

// API Endpoints
app.MapGet("/api/products", async (ApplicationDbContext db) =>
{
    return await db.Products.ToListAsync();
})
.WithName("GetProducts")
.Produces<List<Product>>(StatusCodes.Status200OK);

app.MapGet("/api/products/{id}", async (int id, ApplicationDbContext db) =>
{
    var product = await db.Products.FindAsync(id);

    if (product == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(product);
})
.WithName("GetProduct")
.Produces<Product>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status404NotFound);

app.MapPost("/api/products", async (Product product, ApplicationDbContext db) =>
{
    db.Products.Add(product);
    await db.SaveChangesAsync();

    return Results.Created($"/api/products/{product.Id}", product);
})
.WithName("CreateProduct")
.Accepts<Product>("application/json")
.Produces<Product>(StatusCodes.Status201Created);

app.MapPut("/api/products/{id}", async (int id, Product product, ApplicationDbContext db) =>
{
    var existingProduct = await db.Products.FindAsync(id);

    if (existingProduct == null)
    {
        return Results.NotFound();
    }

    existingProduct.Name = product.Name;
    existingProduct.Description = product.Description;
    existingProduct.Price = product.Price;
    existingProduct.IsAvailable = product.IsAvailable;

    await db.SaveChangesAsync();

    return Results.NoContent();
})
.WithName("UpdateProduct")
.Accepts<Product>("application/json")
.Produces(StatusCodes.Status204NoContent)
.Produces(StatusCodes.Status404NotFound);

app.MapDelete("/api/products/{id}", async (int id, ApplicationDbContext db) =>
{
    var product = await db.Products.FindAsync(id);

    if (product == null)
    {
        return Results.NotFound();
    }

    db.Products.Remove(product);
    await db.SaveChangesAsync();

    return Results.NoContent();
})
.WithName("DeleteProduct")
.Produces(StatusCodes.Status204NoContent)
.Produces(StatusCodes.Status404NotFound);

app.Run();
```

Les API minimales sont plus concises mais peuvent être moins adaptées pour les grandes API complexes.

## 12.5.3. Sérialisation et désérialisation

La sérialisation est le processus de conversion d'objets .NET en formats comme JSON ou XML pour l'envoi sur le réseau. La désérialisation est le processus inverse.

### Sérialisation JSON avec System.Text.Json

Depuis .NET Core 3.0, `System.Text.Json` est la bibliothèque de sérialisation JSON par défaut, remplaçant Newtonsoft.Json (Json.NET).

#### Configuration globale

Vous pouvez configurer la sérialisation JSON globalement dans `Program.cs` :

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Style de nommage (camelCase est le standard pour JSON)
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;

        // Indentation (pour lisibilité, désactiver en production)
        options.JsonSerializerOptions.WriteIndented = true;

        // Ignorer les valeurs nulles
        options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;

        // Convertisseurs personnalisés
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());

        // Gestion des références circulaires
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    });
```

#### Attributs de sérialisation

Vous pouvez personnaliser la sérialisation au niveau des propriétés avec des attributs :

```csharp
public class Product
{
    public int Id { get; set; }

    [JsonPropertyName("product_name")]
    public string Name { get; set; }

    [JsonPropertyOrder(-1)]  // Apparaît en premier
    public decimal Price { get; set; }

    [JsonIgnore]  // Exclu de la sérialisation
    public decimal Cost { get; set; }

    [JsonConverter(typeof(JsonStringEnumConverter))]
    public ProductCategory Category { get; set; }

    [JsonInclude]  // Inclut un champ privé
    private DateTime _createdAt;

    [JsonExtensionData]  // Stocke les propriétés supplémentaires
    public Dictionary<string, JsonElement> ExtensionData { get; set; }
}
```

#### Sérialisation manuelle

Vous pouvez sérialiser et désérialiser manuellement si nécessaire :

```csharp
// Sérialisation
string json = JsonSerializer.Serialize(product, new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    WriteIndented = true
});

// Désérialisation
Product product = JsonSerializer.Deserialize<Product>(json, new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
});
```

#### Sérialisation conditionnelle

Pour une sérialisation conditionnelle, vous pouvez créer des convertisseurs personnalisés :

```csharp
public class ProductConverter : JsonConverter<Product>
{
    public override Product Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        // Logique de désérialisation personnalisée
    }

    public override void Write(Utf8JsonWriter writer, Product value, JsonSerializerOptions options)
    {
        writer.WriteStartObject();

        writer.WriteNumber("id", value.Id);
        writer.WriteString("name", value.Name);
        writer.WriteNumber("price", value.Price);

        // Sérialisation conditionnelle
        if (value.IsAvailable)
        {
            writer.WriteString("status", "In Stock");
        }
        else
        {
            writer.WriteString("status", "Out of Stock");
        }

        writer.WriteEndObject();
    }
}
```

### Utilisation de Newtonsoft.Json (Json.NET)

Si vous préférez ou avez besoin de Newtonsoft.Json, vous pouvez l'ajouter à votre projet :

```csharp
// Installer le package
// dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson

// Dans Program.cs
builder.Services.AddControllers()
    .AddNewtonsoftJson(options =>
    {
        options.SerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();
        options.SerializerSettings.ReferenceLoopHandling = ReferenceLoopHandling.Ignore;
        options.SerializerSettings.NullValueHandling = NullValueHandling.Ignore;
        options.SerializerSettings.Converters.Add(new StringEnumConverter());
    });
```

Attributs courants avec Newtonsoft.Json :

```csharp
public class Product
{
    public int Id { get; set; }

    [JsonProperty("product_name")]
    public string Name { get; set; }

    [JsonIgnore]
    public decimal Cost { get; set; }

    [JsonRequired]
    public decimal Price { get; set; }

    [JsonConverter(typeof(StringEnumConverter))]
    public ProductCategory Category { get; set; }
}
```

## 12.5.4. Formatters (JSON, XML)

Les formatters déterminent comment sérialiser et désérialiser le contenu des requêtes et réponses HTTP. ASP.NET Core inclut des formatters pour JSON, XML et d'autres formats.

### Configuration des formatters

Vous pouvez configurer les formatters dans `Program.cs` :

```csharp
builder.Services.AddControllers(options =>
{
    // Configurer les formatters
    options.OutputFormatters.RemoveType<XmlDataContractSerializerOutputFormatter>();
    options.InputFormatters.RemoveType<XmlDataContractSerializerInputFormatter>();

    // Définir JSON comme format par défaut
    options.Filters.Add(new ProducesAttribute("application/json"));
    options.Filters.Add(new ConsumesAttribute("application/json"));
})
.AddJsonOptions(/* Configuration JSON */)
.AddXmlSerializerFormatters(); // Ajouter le support XML
```

### Content Negotiation

La négociation de contenu permet de servir différents formats selon l'en-tête `Accept` de la requête :

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json", "application/xml")] // Formats supportés
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetProducts()
    {
        // Retourne des produits dans le format demandé
        // (JSON ou XML selon l'en-tête Accept)
        return Ok(_products);
    }
}
```

Le client peut spécifier son format préféré :
- `Accept: application/json` → Réponse en JSON
- `Accept: application/xml` → Réponse en XML
- `Accept: application/json, application/xml;q=0.9` → Préfère JSON, accepte XML comme alternative

### Formats personnalisés

Vous pouvez créer des formatters personnalisés en implémentant `IInputFormatter` et `IOutputFormatter` :

```csharp
public class CsvOutputFormatter : TextOutputFormatter
{
    public CsvOutputFormatter()
    {
        SupportedMediaTypes.Add(MediaTypeHeaderValue.Parse("text/csv"));
        SupportedEncodings.Add(Encoding.UTF8);
        SupportedEncodings.Add(Encoding.Unicode);
    }

    protected override bool CanWriteType(Type type)
    {
        if (typeof(IEnumerable<object>).IsAssignableFrom(type) ||
            typeof(ICollection<object>).IsAssignableFrom(type))
        {
            return true;
        }
        return false;
    }

    public override async Task WriteResponseBodyAsync(OutputFormatterWriteContext context, Encoding selectedEncoding)
    {
        var response = context.HttpContext.Response;
        var buffer = new StringBuilder();

        if (context.Object is IEnumerable<object> enumerable)
        {
            var properties = enumerable.FirstOrDefault()?.GetType().GetProperties();

            if (properties != null)
            {
                // Écrire l'en-tête
                buffer.AppendLine(string.Join(",", properties.Select(p => p.Name)));

                // Écrire les données
                foreach (var obj in enumerable)
                {
                    var values = properties.Select(p => FormatValue(p.GetValue(obj)));
                    buffer.AppendLine(string.Join(",", values));
                }
            }
        }

        await response.WriteAsync(buffer.ToString(), selectedEncoding);
    }

    private static string FormatValue(object value)
    {
        if (value == null)
            return "";

        if (value is string s)
            return $"\"{s.Replace("\"", "\"\"")}\"";

        return value.ToString();
    }
}
```

Enregistrement du formatter personnalisé :

```csharp
// Dans Program.cs
builder.Services.AddControllers(options =>
{
    options.OutputFormatters.Add(new CsvOutputFormatter());
});
```

Utilisation :

```csharp
[HttpGet]
[Produces("application/json", "text/csv")]
public ActionResult<IEnumerable<Product>> GetProducts()
{
    return Ok(_products);
}
```

## 12.5.5. Routes et conventions d'API

Les routes définissent comment les URLs sont mappées aux actions du contrôleur. ASP.NET Core offre plusieurs façons de configurer les routes des API.

### Routage par attributs

Le routage par attributs est la méthode la plus courante pour les API web :

```csharp
[ApiController]
[Route("api/[controller]")]  // Préfixe pour toutes les actions
public class ProductsController : ControllerBase
{
    // GET: api/products
    [HttpGet]
    public IActionResult Get() { ... }

    // GET: api/products/5
    [HttpGet("{id}")]
    public IActionResult Get(int id) { ... }

    // GET: api/products/category/electronics
    [HttpGet("category/{category}")]
    public IActionResult GetByCategory(string category) { ... }

    // GET: api/products/search?term=phone&maxPrice=500
    [HttpGet("search")]
    public IActionResult Search(string term, decimal? maxPrice) { ... }

    // POST: api/products
    [HttpPost]
    public IActionResult Post([FromBody] Product product) { ... }

    // PUT: api/products/5
    [HttpPut("{id}")]
    public IActionResult Put(int id, [FromBody] Product product) { ... }

    // DELETE: api/products/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id) { ... }
}
```

### Contraintes de route

Vous pouvez ajouter des contraintes pour valider les paramètres de route :

```csharp
// Contrainte d'entier
[HttpGet("{id:int}")]
public IActionResult GetProduct(int id) { ... }

// Contrainte de longueur minimale
[HttpGet("slug/{slug:minlength(3)}")]
public IActionResult GetBySlug(string slug) { ... }

// Contrainte d'expression régulière
[HttpGet("code/{code:regex(^[A-Z]\\d{3}$)}")]
public IActionResult GetByCode(string code) { ... }

// Contraintes combinées
[HttpGet("archive/{year:int:min(2000)}/{month:range(1,12)}")]
public IActionResult GetArchive(int year, int month) { ... }
```

Types de contraintes courants :
- `int` : Entier
- `bool` : Booléen
- `datetime` : Date et heure
- `decimal` : Nombre décimal
- `double` : Nombre à virgule flottante
- `float` : Nombre à virgule flottante
- `guid` : GUID
- `minlength(value)` : Longueur minimale
- `maxlength(value)` : Longueur maximale
- `length(min,max)` : Plage de longueur
- `min(value)` : Valeur minimale
- `max(value)` : Valeur maximale
- `range(min,max)` : Plage de valeurs
- `alpha` : Caractères alphabétiques
- `regex(pattern)` : Expression régulière
- `required` : Paramètre obligatoire

### Préfixes de route

Vous pouvez définir des préfixes de route pour organiser vos API :

```csharp
// Niveau contrôleur
[Route("api/v1/[controller]")]  // Version dans l'URL
public class ProductsController : ControllerBase { ... }

// Avec espace de noms
[Route("api/[area]/[controller]")]
[Area("Catalog")]
public class ProductsController : ControllerBase { ... }
```

### Routage conventionnel

Le routage conventionnel définit des modèles de route centralisés dans `Program.cs` :

```csharp
app.MapControllerRoute(
    name: "api",
    pattern: "api/{controller=Home}/{action=Index}/{id?}");
```

Cependant, pour les API REST, le routage par attributs est généralement préféré car il est plus explicite.

### Versionnement d'API

Le versionnement est important pour maintenir la compatibilité avec les clients existants lors de l'évolution de l'API. ASP.NET Core prend en charge plusieurs stratégies de versionnement.

#### 1. Versionnement dans l'URL

```csharp
// Contrôleurs différents pour chaque version
[ApiController]
[Route("api/v1/products")]
public class ProductsV1Controller : ControllerBase { ... }

[ApiController]
[Route("api/v2/products")]
public class ProductsV2Controller : ControllerBase { ... }

// OU version dans le même contrôleur
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1() { ... }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2() { ... }
}
```

#### 2. Versionnement par en-tête HTTP

```csharp
// Configuration
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new HeaderApiVersionReader("api-version");
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

// Contrôleur
[ApiController]
[Route("api/products")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase { ... }
```

Client : `api-version: 2.0` dans les en-têtes HTTP.

#### 3. Versionnement par type de média

```csharp
// Configuration
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new MediaTypeApiVersionReader();
});

// Contrôleur
[ApiController]
[Route("api/products")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase { ... }
```

Client : `Accept: application/json;v=2.0` dans les en-têtes HTTP.

### Conventions d'API

Les conventions d'API permettent d'appliquer des règles cohérentes à travers tous vos contrôleurs.

#### ApplicationModel Conventions

```csharp
// Créer une convention personnalisée
public class ApiExplorerGroupPerVersionConvention : IControllerModelConvention
{
    public void Apply(ControllerModel controller)
    {
        var controllerNamespace = controller.ControllerType.Namespace;
        var apiVersion = controllerNamespace.Split('.').Last();

        controller.ApiExplorer.GroupName = apiVersion;
    }
}

// Appliquer la convention
builder.Services.AddControllers(options =>
{
    options.Conventions.Add(new ApiExplorerGroupPerVersionConvention());
});
```

#### Conventions de route

```csharp
// Dans Program.cs
app.MapControllers()
   .RequireAuthorization() // Toutes les routes nécessitent une authentification
   .RequireCors("ApiPolicy"); // Toutes les routes utilisent cette politique CORS
```

#### Route Tokens

Les tokens de route permettent d'insérer dynamiquement des valeurs dans les routes :

```csharp
// Configuration du token personnalisé
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("tenant", typeof(TenantRouteConstraint));
});

// Utilisation
[Route("api/{tenant}/products")]
public class ProductsController : ControllerBase { ... }
```

### RESTful URLs (Bonnes pratiques)

Voici quelques bonnes pratiques pour concevoir des URLs RESTful :

1. **Utiliser des noms pluriels pour les collections** :
   - ✅ `/api/products` (collection)
   - ✅ `/api/products/5` (élément spécifique)
   - ❌ `/api/product`

2. **Représenter les hiérarchies avec des segments d'URL** :
   - ✅ `/api/customers/42/orders` (commandes du client 42)
   - ✅ `/api/customers/42/orders/7` (commande 7 du client 42)

3. **Utiliser des noms de ressources et non des verbes** :
   - ✅ `GET /api/products` (obtenir tous les produits)
   - ❌ `GET /api/getProducts`

4. **Utiliser les méthodes HTTP pour les actions CRUD** :
   - ✅ `GET /api/products` (lire tous)
   - ✅ `POST /api/products` (créer)
   - ✅ `PUT /api/products/5` (mettre à jour tout)
   - ✅ `PATCH /api/products/5` (mettre à jour partiellement)
   - ✅ `DELETE /api/products/5` (supprimer)
   - ❌ `GET /api/deleteProduct/5`

5. **Utiliser les query string pour le filtrage et la pagination** :
   - ✅ `GET /api/products?category=electronics`
   - ✅ `GET /api/products?minPrice=100&maxPrice=500`
   - ✅ `GET /api/products?page=2&pageSize=10`

6. **Utiliser les sous-ressources pour les relations** :
   - ✅ `GET /api/products/5/reviews` (critiques du produit 5)
   - ✅ `POST /api/products/5/reviews` (ajouter une critique)

7. **Utiliser des URL spécifiques pour les actions non CRUD** :
   - ✅ `POST /api/orders/5/cancel` (annuler la commande)
   - ✅ `POST /api/products/5/rate` (noter le produit)

### Pièges courants à éviter

1. **Exposer les détails de l'implémentation dans l'URL** :
   - ❌ `/api/sql-products`
   - ❌ `/api/products-table`

2. **Ignorer les conventions HTTP** :
   - ❌ Utiliser GET pour modifier des données
   - ❌ Renvoyer 200 OK pour une création réussie (utilisez 201 Created)

3. **Ne pas gérer les erreurs correctement** :
   - ❌ Renvoyer 200 OK avec un message d'erreur
   - ❌ Utiliser des codes d'erreur personnalisés non standard

4. **Créer des API trop bavards** :
   - ❌ `/api/get-all-products-with-details-and-reviews`

5. **Ignorer l'aspect "sans état" (stateless)** :
   - ❌ Dépendre de sessions pour les opérations API

## Exemple complet d'une API RESTful

Voici un exemple complet d'une API RESTful pour une application de gestion de bibliothèque :

### Modèles

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public string ISBN { get; set; }
    public int PublicationYear { get; set; }
    public bool IsAvailable { get; set; }
    public string Genre { get; set; }

    // Navigation properties
    public ICollection<Review> Reviews { get; set; }
    public ICollection<Loan> Loans { get; set; }
}

public class Review
{
    public int Id { get; set; }
    public int BookId { get; set; }
    public string ReviewerName { get; set; }
    public int Rating { get; set; }  // 1-5
    public string Comment { get; set; }
    public DateTime CreatedAt { get; set; }

    // Navigation property
    public Book Book { get; set; }
}

public class Loan
{
    public int Id { get; set; }
    public int BookId { get; set; }
    public int PatronId { get; set; }
    public DateTime LoanDate { get; set; }
    public DateTime DueDate { get; set; }
    public DateTime? ReturnDate { get; set; }

    // Navigation properties
    public Book Book { get; set; }
    public Patron Patron { get; set; }
}

public class Patron
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string MembershipNumber { get; set; }

    // Navigation property
    public ICollection<Loan> Loans { get; set; }
}
```

### DTOs

```csharp
public class BookDto
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public string ISBN { get; set; }
    public int PublicationYear { get; set; }
    public bool IsAvailable { get; set; }
    public string Genre { get; set; }
    public double AverageRating { get; set; }
}

public class BookDetailDto : BookDto
{
    public IEnumerable<ReviewDto> Reviews { get; set; }
}

public class CreateBookDto
{
    [Required]
    [StringLength(200)]
    public string Title { get; set; }

    [Required]
    [StringLength(100)]
    public string Author { get; set; }

    [StringLength(13)]
    [RegularExpression(@"^(?=(?:\D*\d){10}(?:(?:\D*\d){3})?$)[\d-]+$")]
    public string ISBN { get; set; }

    [Range(1000, 3000)]
    public int PublicationYear { get; set; }

    [StringLength(50)]
    public string Genre { get; set; }
}

public class ReviewDto
{
    public int Id { get; set; }
    public string ReviewerName { get; set; }
    public int Rating { get; set; }
    public string Comment { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class CreateReviewDto
{
    [Required]
    [StringLength(100)]
    public string ReviewerName { get; set; }

    [Range(1, 5)]
    public int Rating { get; set; }

    [StringLength(1000)]
    public string Comment { get; set; }
}

public class LoanDto
{
    public int Id { get; set; }
    public int BookId { get; set; }
    public string BookTitle { get; set; }
    public int PatronId { get; set; }
    public string PatronName { get; set; }
    public DateTime LoanDate { get; set; }
    public DateTime DueDate { get; set; }
    public DateTime? ReturnDate { get; set; }
    public bool IsOverdue => !ReturnDate.HasValue && DateTime.Now > DueDate;
}

public class CreateLoanDto
{
    [Required]
    public int BookId { get; set; }

    [Required]
    public int PatronId { get; set; }

    [Required]
    public DateTime DueDate { get; set; }
}
```

### Contrôleurs API

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class BooksController : ControllerBase
{
    private readonly IBookRepository _bookRepository;
    private readonly ILogger<BooksController> _logger;

    public BooksController(IBookRepository bookRepository, ILogger<BooksController> logger)
    {
        _bookRepository = bookRepository;
        _logger = logger;
    }

    // GET: api/books
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<BookDto>>> GetBooks([FromQuery] string genre = null,
                                                                  [FromQuery] string author = null,
                                                                  [FromQuery] bool? available = null,
                                                                  [FromQuery] int page = 1,
                                                                  [FromQuery] int pageSize = 10)
    {
        var books = await _bookRepository.GetBooksAsync(genre, author, available, page, pageSize);

        return Ok(books.Select(book => new BookDto
        {
            Id = book.Id,
            Title = book.Title,
            Author = book.Author,
            ISBN = book.ISBN,
            PublicationYear = book.PublicationYear,
            IsAvailable = book.IsAvailable,
            Genre = book.Genre,
            AverageRating = book.Reviews.Any() ? book.Reviews.Average(r => r.Rating) : 0
        }));
    }

    // GET: api/books/5
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<BookDetailDto>> GetBook(int id)
    {
        var book = await _bookRepository.GetBookByIdAsync(id);

        if (book == null)
        {
            return NotFound();
        }

        var bookDetailDto = new BookDetailDto
        {
            Id = book.Id,
            Title = book.Title,
            Author = book.Author,
            ISBN = book.ISBN,
            PublicationYear = book.PublicationYear,
            IsAvailable = book.IsAvailable,
            Genre = book.Genre,
            AverageRating = book.Reviews.Any() ? book.Reviews.Average(r => r.Rating) : 0,
            Reviews = book.Reviews.Select(r => new ReviewDto
            {
                Id = r.Id,
                ReviewerName = r.ReviewerName,
                Rating = r.Rating,
                Comment = r.Comment,
                CreatedAt = r.CreatedAt
            })
        };

        return Ok(bookDetailDto);
    }

    // POST: api/books
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<BookDto>> CreateBook(CreateBookDto createBookDto)
    {
        var book = new Book
        {
            Title = createBookDto.Title,
            Author = createBookDto.Author,
            ISBN = createBookDto.ISBN,
            PublicationYear = createBookDto.PublicationYear,
            IsAvailable = true,
            Genre = createBookDto.Genre,
            Reviews = new List<Review>(),
            Loans = new List<Loan>()
        };

        await _bookRepository.AddBookAsync(book);

        var bookDto = new BookDto
        {
            Id = book.Id,
            Title = book.Title,
            Author = book.Author,
            ISBN = book.ISBN,
            PublicationYear = book.PublicationYear,
            IsAvailable = book.IsAvailable,
            Genre = book.Genre,
            AverageRating = 0
        };

        return CreatedAtAction(nameof(GetBook), new { id = book.Id }, bookDto);
    }

    // PUT: api/books/5
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateBook(int id, CreateBookDto updateBookDto)
    {
        var book = await _bookRepository.GetBookByIdAsync(id);

        if (book == null)
        {
            return NotFound();
        }

        book.Title = updateBookDto.Title;
        book.Author = updateBookDto.Author;
        book.ISBN = updateBookDto.ISBN;
        book.PublicationYear = updateBookDto.PublicationYear;
        book.Genre = updateBookDto.Genre;

        await _bookRepository.UpdateBookAsync(book);

        return NoContent();
    }

    // DELETE: api/books/5
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteBook(int id)
    {
        var book = await _bookRepository.GetBookByIdAsync(id);

        if (book == null)
        {
            return NotFound();
        }

        await _bookRepository.DeleteBookAsync(book);

        return NoContent();
    }

    // GET: api/books/5/reviews
    [HttpGet("{id}/reviews")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<IEnumerable<ReviewDto>>> GetBookReviews(int id)
    {
        if (!await _bookRepository.BookExistsAsync(id))
        {
            return NotFound();
        }

        var reviews = await _bookRepository.GetBookReviewsAsync(id);

        var reviewDtos = reviews.Select(r => new ReviewDto
        {
            Id = r.Id,
            ReviewerName = r.ReviewerName,
            Rating = r.Rating,
            Comment = r.Comment,
            CreatedAt = r.CreatedAt
        });

        return Ok(reviewDtos);
    }

    // POST: api/books/5/reviews
    [HttpPost("{id}/reviews")]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ReviewDto>> AddBookReview(int id, CreateReviewDto createReviewDto)
    {
        if (!await _bookRepository.BookExistsAsync(id))
        {
            return NotFound();
        }

        var review = new Review
        {
            BookId = id,
            ReviewerName = createReviewDto.ReviewerName,
            Rating = createReviewDto.Rating,
            Comment = createReviewDto.Comment,
            CreatedAt = DateTime.UtcNow
        };

        await _bookRepository.AddReviewAsync(review);

        var reviewDto = new ReviewDto
        {
            Id = review.Id,
            ReviewerName = review.ReviewerName,
            Rating = review.Rating,
            Comment = review.Comment,
            CreatedAt = review.CreatedAt
        };

        return CreatedAtAction(nameof(GetBookReviews), new { id }, reviewDto);
    }

    // POST: api/books/5/checkout
    [HttpPost("{id}/checkout")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<LoanDto>> CheckoutBook(int id, [FromBody] CreateLoanDto createLoanDto)
    {
        var book = await _bookRepository.GetBookByIdAsync(id);

        if (book == null)
        {
            return NotFound();
        }

        if (!book.IsAvailable)
        {
            return BadRequest("Ce livre n'est pas disponible pour l'emprunt.");
        }

        var loan = new Loan
        {
            BookId = id,
            PatronId = createLoanDto.PatronId,
            LoanDate = DateTime.UtcNow,
            DueDate = createLoanDto.DueDate
        };

        book.IsAvailable = false;

        await _bookRepository.AddLoanAsync(loan);
        await _bookRepository.UpdateBookAsync(book);

        // Pour simplicité, on suppose que le service a déjà chargé Patron
        var patron = await _bookRepository.GetPatronByIdAsync(createLoanDto.PatronId);

        var loanDto = new LoanDto
        {
            Id = loan.Id,
            BookId = loan.BookId,
            BookTitle = book.Title,
            PatronId = loan.PatronId,
            PatronName = patron.Name,
            LoanDate = loan.LoanDate,
            DueDate = loan.DueDate,
            ReturnDate = loan.ReturnDate
        };

        return Ok(loanDto);
    }

    // POST: api/books/5/return
    [HttpPost("{id}/return")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<LoanDto>> ReturnBook(int id)
    {
        var book = await _bookRepository.GetBookByIdAsync(id);

        if (book == null)
        {
            return NotFound();
        }

        if (book.IsAvailable)
        {
            return BadRequest("Ce livre n'est pas actuellement emprunté.");
        }

        var loan = await _bookRepository.GetActiveBookLoanAsync(id);

        if (loan == null)
        {
            return BadRequest("Impossible de trouver l'emprunt actif pour ce livre.");
        }

        loan.ReturnDate = DateTime.UtcNow;
        book.IsAvailable = true;

        await _bookRepository.UpdateLoanAsync(loan);
        await _bookRepository.UpdateBookAsync(book);

        var patron = await _bookRepository.GetPatronByIdAsync(loan.PatronId);

        var loanDto = new LoanDto
        {
            Id = loan.Id,
            BookId = loan.BookId,
            BookTitle = book.Title,
            PatronId = loan.PatronId,
            PatronName = patron.Name,
            LoanDate = loan.LoanDate,
            DueDate = loan.DueDate,
            ReturnDate = loan.ReturnDate
        };

        return Ok(loanDto);
    }
}
```

C'est un exemple assez complet d'une API REST avec ASP.NET Core, montrant les bonnes pratiques en terme de conception d'API, utilisation des codes HTTP appropriés, documentation des réponses possibles et structuration du code.

## Conclusion

La création d'API Web et de services REST avec ASP.NET Core est un processus bien structuré qui s'appuie sur des conventions établies. En suivant les principes REST et en utilisant les fonctionnalités d'ASP.NET Core, vous pouvez créer des API robustes, évolutives et faciles à utiliser.

Points clés à retenir :

1. **Principes REST** : Utilisez les méthodes HTTP appropriées, structurez logiquement vos ressources, et utilisez correctement les codes de statut HTTP.

2. **Contrôleurs API** : Héritez de `ControllerBase` pour les API, utilisez les attributs de routage, et implémentez les opérations CRUD standard.

3. **DTOs et modèles** : Utilisez des DTOs pour la communication avec les clients, séparés de vos modèles de domaine internes.

4. **Sérialisation** : Configurez la sérialisation JSON selon vos besoins, et supportez éventuellement d'autres formats comme XML.

5. **Routing et conventions** : Utilisez des routes claires et cohérentes, avec des contraintes pour valider les entrées.

ASP.NET Core fournit tous les outils nécessaires pour construire des API modernes et performantes qui peuvent évoluer avec vos besoins. Que vous construisiez une petite API pour une application mobile ou un système complexe d'API pour une architecture microservices, les principes et techniques présentés dans ce chapitre vous aideront à créer des interfaces robustes et professionnelles.

⏭️ 12.6. [Documentation API avec Swagger/OpenAPI](/12-developpement-web-avec-asp-dotnet/12-6-documentation-api-avec-swagger-openapi.md)
