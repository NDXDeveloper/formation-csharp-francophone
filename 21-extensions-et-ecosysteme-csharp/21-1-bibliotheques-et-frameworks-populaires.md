# 21. Extensions et écosystème C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 21.1. Bibliothèques et frameworks populaires

Dans cette section, nous allons explorer les bibliothèques et frameworks les plus utilisés dans l'écosystème C#. Ces outils vous permettront d'être plus productif et d'ajouter des fonctionnalités avancées à vos applications.

### 21.1.1. Newtonsoft.Json et System.Text.Json

La sérialisation JSON est essentielle dans le développement moderne. C# dispose de deux principales bibliothèques pour gérer le JSON.

#### Newtonsoft.Json (Json.NET)

Json.NET est la bibliothèque JSON la plus populaire pour .NET. Elle offre une flexibilité et des fonctionnalités avancées.

**Installation :**
```bash
dotnet add package Newtonsoft.Json
```

**Exemple de base :**
```csharp
using Newtonsoft.Json;

// Classe exemple
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
    public DateTime BirthDate { get; set; }
}

// Sérialisation
Person person = new Person
{
    FirstName = "Jean",
    LastName = "Dupont",
    Age = 30,
    BirthDate = new DateTime(1993, 5, 15)
};

string json = JsonConvert.SerializeObject(person);
Console.WriteLine(json);
// {"FirstName":"Jean","LastName":"Dupont","Age":30,"BirthDate":"1993-05-15T00:00:00"}

// Désérialisation
Person deserializedPerson = JsonConvert.DeserializeObject<Person>(json);
```

**Personnalisation avec des attributs :**
```csharp
public class Product
{
    [JsonProperty("productName")]
    public string Name { get; set; }

    [JsonProperty("price")]
    public decimal Price { get; set; }

    [JsonIgnore]
    public string InternalCode { get; set; }
}
```

**Gestion des dates et formatage :**
```csharp
// Formatage personnalisé
var settings = new JsonSerializerSettings
{
    DateFormatString = "yyyy-MM-dd",
    Formatting = Formatting.Indented
};

string jsonFormatted = JsonConvert.SerializeObject(person, settings);
```

#### System.Text.Json

Introduit dans .NET Core 3.0, System.Text.Json est une alternative haute performance incluse dans .NET.

**Avantages :**
- Inclus dans .NET (pas besoin de package externe)
- Plus rapide et consomme moins de mémoire
- Support UTF-8 natif

**Exemple de base :**
```csharp
using System.Text.Json;

// Sérialisation
Person person = new Person
{
    FirstName = "Jean",
    LastName = "Dupont",
    Age = 30
};

string json = JsonSerializer.Serialize(person);

// Désérialisation
Person deserializedPerson = JsonSerializer.Deserialize<Person>(json);
```

**Options de sérialisation :**
```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    WriteIndented = true,
    IgnoreNullValues = true
};

string json = JsonSerializer.Serialize(person, options);
```

**Attributs System.Text.Json :**
```csharp
using System.Text.Json.Serialization;

public class Product
{
    [JsonPropertyName("productName")]
    public string Name { get; set; }

    [JsonIgnore]
    public string InternalCode { get; set; }

    [JsonConverter(typeof(JsonStringEnumConverter))]
    public ProductStatus Status { get; set; }
}
```

### 21.1.2. AutoMapper

AutoMapper est une bibliothèque qui simplifie le mapping d'objets, transformant un type d'objet vers un autre automatiquement.

**Installation :**
```bash
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

**Configuration de base :**
```csharp
using AutoMapper;

// Classes source et destination
public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public Address Address { get; set; }
}

public class CustomerDto
{
    public int Id { get; set; }
    public string FullName { get; set; }
    public string Email { get; set; }
    public string City { get; set; }
}

// Profil de mapping
public class CustomerProfile : Profile
{
    public CustomerProfile()
    {
        CreateMap<Customer, CustomerDto>()
            .ForMember(dest => dest.FullName,
                       opt => opt.MapFrom(src => $"{src.FirstName} {src.LastName}"))
            .ForMember(dest => dest.City,
                       opt => opt.MapFrom(src => src.Address.City));
    }
}
```

**Utilisation d'AutoMapper :**
```csharp
// Configuration
var config = new MapperConfiguration(cfg => {
    cfg.AddProfile<CustomerProfile>();
});

IMapper mapper = config.CreateMapper();

// Mapping
Customer customer = new Customer
{
    Id = 1,
    FirstName = "Jean",
    LastName = "Dupont",
    Email = "jean.dupont@email.com",
    Address = new Address { City = "Paris" }
};

CustomerDto dto = mapper.Map<CustomerDto>(customer);
```

**Intégration avec Dependency Injection :**
```csharp
// Dans Startup.cs ou Program.cs
services.AddAutoMapper(typeof(Program));

// Utilisation dans un contrôleur
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly IMapper _mapper;

    public CustomersController(IMapper mapper)
    {
        _mapper = mapper;
    }

    [HttpGet("{id}")]
    public async Task<CustomerDto> GetCustomer(int id)
    {
        var customer = await _customerRepository.GetByIdAsync(id);
        return _mapper.Map<CustomerDto>(customer);
    }
}
```

**Mapping avancé avec collections :**
```csharp
// Mapping de collections
List<Customer> customers = await _customerRepository.GetAllAsync();
List<CustomerDto> customerDtos = _mapper.Map<List<CustomerDto>>(customers);

// Mapping conditionnel
CreateMap<Customer, CustomerDto>()
    .ForMember(dest => dest.Email,
               opt => opt.Condition(src => !string.IsNullOrEmpty(src.Email)))
    .ForMember(dest => dest.IsVip,
               opt => opt.MapFrom(src => src.TotalPurchases > 1000));
```

### 21.1.3. FluentValidation

FluentValidation est une bibliothèque pour créer des règles de validation de façon fluide et lisible.

**Installation :**
```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore
```

**Exemple de base :**
```csharp
using FluentValidation;

public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
    public string PhoneNumber { get; set; }
}

public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(x => x.FirstName)
            .NotEmpty().WithMessage("Le prénom est requis")
            .Length(2, 50).WithMessage("Le prénom doit contenir entre 2 et 50 caractères");

        RuleFor(x => x.LastName)
            .NotEmpty().WithMessage("Le nom est requis")
            .Length(2, 50).WithMessage("Le nom doit contenir entre 2 et 50 caractères");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("L'email est requis")
            .EmailAddress().WithMessage("Format d'email invalide");

        RuleFor(x => x.Age)
            .NotEmpty().WithMessage("L'âge est requis")
            .GreaterThanOrEqualTo(18).WithMessage("L'âge doit être supérieur ou égal à 18 ans")
            .LessThanOrEqualTo(120).WithMessage("L'âge doit être inférieur à 120 ans");
    }
}
```

**Utilisation du validateur :**
```csharp
Customer customer = new Customer
{
    FirstName = "Jean",
    LastName = "D",
    Email = "invalid-email",
    Age = 16
};

CustomerValidator validator = new CustomerValidator();
var result = validator.Validate(customer);

if (!result.IsValid)
{
    foreach (var error in result.Errors)
    {
        Console.WriteLine($"Propriété: {error.PropertyName}, Message: {error.ErrorMessage}");
    }
}
```

**Règles avancées :**
```csharp
public class OrderValidator : AbstractValidator<Order>
{
    public OrderValidator()
    {
        // Validation personnalisée
        RuleFor(x => x.OrderDate)
            .Must(BeInTheFuture).WithMessage("La date de commande doit être dans le futur");

        // Validation conditionnelle
        RuleFor(x => x.ShippingAddress)
            .NotEmpty().When(x => x.RequiresShipping);

        // Validation dépendante
        RuleFor(x => x.TotalAmount)
            .GreaterThan(0)
            .LessThanOrEqualTo(x => x.Customer.CreditLimit);

        // Validation avec code async
        RuleFor(x => x.CustomerEmail)
            .MustAsync(async (email, cancellation) => {
                return await _customerService.EmailExistsAsync(email);
            }).WithMessage("L'email client n'existe pas");
    }

    private bool BeInTheFuture(DateTime date)
    {
        return date > DateTime.UtcNow;
    }
}
```

**Intégration avec ASP.NET Core :**
```csharp
// Configuration dans Startup.cs ou Program.cs
services.AddFluentValidation(fv =>
{
    fv.RegisterValidatorsFromAssemblyContaining<CustomerValidator>();
    fv.RunDefaultMvcValidationAfterFluentValidationExecutes = false;
});

// Utilisation dans un contrôleur
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> CreateCustomer([FromBody] Customer customer)
    {
        // FluentValidation s'exécute automatiquement
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        // Logique de création
        await _customerService.CreateAsync(customer);
        return Ok();
    }
}
```

### 21.1.4. MediatR

MediatR est une bibliothèque qui implémente le pattern Mediator pour découpler l'envoi de requêtes de leur gestion.

**Installation :**
```bash
dotnet add package MediatR
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection
```

**Exemple avec IRequest/IRequestHandler :**
```csharp
using MediatR;

// Requête pour créer un client
public class CreateCustomerCommand : IRequest<int>
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

// Handler pour la requête
public class CreateCustomerHandler : IRequestHandler<CreateCustomerCommand, int>
{
    private readonly ICustomerRepository _repository;

    public CreateCustomerHandler(ICustomerRepository repository)
    {
        _repository = repository;
    }

    public async Task<int> Handle(CreateCustomerCommand request, CancellationToken cancellationToken)
    {
        var customer = new Customer
        {
            FirstName = request.FirstName,
            LastName = request.LastName,
            Email = request.Email
        };

        await _repository.AddAsync(customer);
        return customer.Id;
    }
}
```

**Utilisation dans un contrôleur :**
```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly IMediator _mediator;

    public CustomersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> CreateCustomer([FromBody] CreateCustomerCommand command)
    {
        var customerId = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetCustomer), new { id = customerId }, command);
    }
}
```

**Configuration dans Program.cs :**
```csharp
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));
```

**Requêtes et Notifications :**
```csharp
// Requête avec réponse
public class GetCustomerQuery : IRequest<CustomerDto>
{
    public int Id { get; set; }
}

// Notification (sans réponse)
public class CustomerCreatedNotification : INotification
{
    public Customer Customer { get; set; }
}

// Handler pour la notification
public class SendWelcomeEmailHandler : INotificationHandler<CustomerCreatedNotification>
{
    private readonly IEmailService _emailService;

    public SendWelcomeEmailHandler(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public async Task Handle(CustomerCreatedNotification notification, CancellationToken cancellationToken)
    {
        await _emailService.SendWelcomeEmail(notification.Customer.Email);
    }
}
```

**Pipelines et comportements :**
```csharp
// Comportement pour la validation
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(TRequest request,
                                      RequestHandlerDelegate<TResponse> next,
                                      CancellationToken cancellationToken)
    {
        // Validation
        var context = new ValidationContext<TRequest>(request);
        var validationResults = await Task.WhenAll(_validators.Select(v => v.ValidateAsync(context, cancellationToken)));
        var failures = validationResults.SelectMany(r => r.Errors).Where(f => f != null).ToList();

        if (failures.Count != 0)
        {
            throw new ValidationException(failures);
        }

        // Continuer le pipeline
        return await next();
    }
}

// Enregistrement du comportement
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```

### 21.1.5. Serilog et autres frameworks de logging

Le logging est crucial pour le debugging et le monitoring des applications. Serilog est l'un des frameworks de logging les plus populaires pour .NET.

**Installation de Serilog :**
```bash
dotnet add package Serilog
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

**Configuration dans Program.cs :**
```csharp
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
    .WriteTo.Console()
    .WriteTo.File("logs/myapp.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

try
{
    Log.Information("Starting web application");

    var builder = WebApplication.CreateBuilder(args);

    // Ajout de Serilog
    builder.Host.UseSerilog();

    // Configuration des services
    var app = builder.Build();

    // Configuration du pipeline
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

**Utilisation de Serilog dans une classe :**
```csharp
public class CustomerService
{
    private readonly ILogger<CustomerService> _logger;

    public CustomerService(ILogger<CustomerService> logger)
    {
        _logger = logger;
    }

    public async Task<Customer> CreateCustomerAsync(CreateCustomerRequest request)
    {
        _logger.LogInformation("Creating customer with email {Email}", request.Email);

        try
        {
            var customer = new Customer
            {
                FirstName = request.FirstName,
                LastName = request.LastName,
                Email = request.Email
            };

            await _repository.AddAsync(customer);

            _logger.LogInformation("Customer {CustomerId} created successfully", customer.Id);
            return customer;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to create customer with email {Email}", request.Email);
            throw;
        }
    }
}
```

**Logging structuré avec propriétés :**
```csharp
public class OrderProcessor
{
    private readonly ILogger<OrderProcessor> _logger;

    public async Task ProcessOrder(Order order)
    {
        using (_logger.BeginScope("Processing order {OrderId}", order.Id))
        {
            _logger.LogInformation("Order processing started for customer {CustomerId}", order.CustomerId);

            // Logique de traitement
            var result = await ProcessPayment(order);

            if (result.IsSuccess)
            {
                _logger.LogInformation("Order {OrderId} processed successfully. Amount: {Amount:C}, Payment Method: {PaymentMethod}",
                    order.Id, order.TotalAmount, order.PaymentMethod);
            }
            else
            {
                _logger.LogWarning("Order {OrderId} failed. Reason: {Reason}",
                    order.Id, result.ErrorMessage);
            }
        }
    }
}
```

**Configuration avancée de Serilog :**
```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("System", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithProperty("Application", "MyApp")
    .WriteTo.Console(outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {Message:lj}{NewLine}{Exception}")
    .WriteTo.File(
        path: "logs/myapp-.txt",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7,
        fileSizeLimitBytes: 10_000_000,
        rollOnFileSizeLimit: true,
        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {SourceContext} {Message:lj}{NewLine}{Exception}")
    .CreateLogger();
```

#### Autres frameworks de logging

**NLog :**
```csharp
// Configuration dans nlog.config
/*
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <targets>
    <target name="file" xsi:type="File" fileName="logs/${shortdate}.log" />
    <target name="console" xsi:type="Console" />
  </targets>
  <rules>
    <logger name="*" minlevel="Info" writeTo="file,console" />
  </rules>
</nlog>
*/

// Utilisation dans le code
private static readonly Logger Logger = LogManager.GetCurrentClassLogger();

public void ProcessOrder(Order order)
{
    Logger.Info("Processing order {OrderId}", order.Id);
    // ...
}
```

**log4net :**
```csharp
// Configuration dans App.config ou Web.config
log4net.Config.XmlConfigurator.Configure();

private static readonly ILog log = LogManager.GetLogger(typeof(MyClass));

public void DoWork()
{
    log.Info("Starting work");
    // ...
}
```

**Comparaison des frameworks de logging :**

| Caractéristique | Serilog | NLog | log4net | ILogger (.NET) |
|----------------|---------|------|---------|----------------|
| Performance | Très bonne | Excellente | Bonne | Excellente |
| Logging structuré | Excellent | Bon | Limité | Bon |
| Configuration | Code/JSON | XML/JSON | XML | JSON |
| Sinks/Targets | Nombreux | Nombreux | Nombreux | Via providers |
| Popularité | Très haute | Haute | Moyenne | Haute (intégré) |

## Conclusion

Ces bibliothèques et frameworks représentent une partie essentielle de l'écosystème C#. Elles vous permettent de :

- **Newtonsoft.Json/System.Text.Json** : Gérer facilement la sérialisation JSON
- **AutoMapper** : Simplifier le mapping entre objets
- **FluentValidation** : Créer des validations robustes et maintenables
- **MediatR** : Implémenter le pattern CQRS et découpler votre application
- **Serilog** : Avoir un logging structuré et configurable

L'apprentissage et la maîtrise de ces outils vous rendront beaucoup plus productif dans vos projets C#.

⏭️
