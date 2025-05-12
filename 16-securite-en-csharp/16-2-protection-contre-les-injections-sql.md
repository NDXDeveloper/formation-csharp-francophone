# 16.2. Protection contre les injections SQL

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les injections SQL représentent l'une des vulnérabilités les plus courantes et les plus dangereuses dans les applications qui interagissent avec des bases de données. Dans cette section, nous allons explorer comment protéger efficacement vos applications C# contre ce type d'attaque.

## 16.2.1. Paramétrage des requêtes

### Qu'est-ce qu'une injection SQL?

Une injection SQL se produit lorsqu'un attaquant insère du code SQL malveillant dans les entrées de votre application, qui est ensuite exécuté par votre base de données. Voici un exemple simple :

```csharp
// Code vulnérable
string username = userInput; // Imaginons que userInput contient: admin' --
string query = "SELECT * FROM Users WHERE Username = '" + username + "' AND Password = '" + password + "'";
// La requête devient: SELECT * FROM Users WHERE Username = 'admin' --' AND Password = '...'
// Le double tiret (--) commente le reste de la requête, ignorant la vérification du mot de passe!
```

### Requêtes paramétrées avec ADO.NET

La première ligne de défense contre les injections SQL est l'utilisation de requêtes paramétrées :

```csharp
using System.Data.SqlClient; // Pour SQL Server

public User AuthenticateUser(string username, string password)
{
    User user = null;
    string connectionString = "votre_chaine_de_connexion";

    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        // Créer une commande avec des paramètres
        string query = "SELECT * FROM Users WHERE Username = @Username AND Password = @Password";
        SqlCommand command = new SqlCommand(query, connection);

        // Ajouter les paramètres - SqlParameter s'occupe de l'échappement
        command.Parameters.AddWithValue("@Username", username);
        command.Parameters.AddWithValue("@Password", password); // En pratique, utilisez un hachage!

        connection.Open();

        using (SqlDataReader reader = command.ExecuteReader())
        {
            if (reader.Read())
            {
                user = new User
                {
                    Id = reader.GetInt32(reader.GetOrdinal("Id")),
                    Username = reader.GetString(reader.GetOrdinal("Username")),
                    // Autres propriétés...
                };
            }
        }
    }

    return user;
}
```

### Bonnes pratiques pour le paramétrage

1. **Ne jamais concaténer des entrées utilisateur directement dans des requêtes SQL**

   ```csharp
   // À ÉVITER ABSOLUMENT
   string query = "SELECT * FROM Products WHERE CategoryId = " + categoryId;

   // PRÉFÉRER
   string query = "SELECT * FROM Products WHERE CategoryId = @CategoryId";
   command.Parameters.AddWithValue("@CategoryId", categoryId);
   ```

2. **Utiliser les types de données appropriés pour les paramètres**

   ```csharp
   // Mieux que AddWithValue pour certains scénarios
   SqlParameter param = new SqlParameter("@CategoryId", SqlDbType.Int);
   param.Value = categoryId;
   command.Parameters.Add(param);
   ```

3. **Être prudent avec les requêtes dynamiques (quand elles sont vraiment nécessaires)**

   Si vous devez construire des requêtes dynamiques (par exemple, pour le tri ou le filtrage), utilisez des listes blanches pour les noms de colonnes :

   ```csharp
   public IEnumerable<Product> GetProductsSorted(string sortColumn, bool ascending)
   {
       // Liste blanche des colonnes autorisées
       var allowedColumns = new HashSet<string> { "Name", "Price", "CreatedDate" };

       // Validation du nom de colonne
       if (!allowedColumns.Contains(sortColumn))
       {
           sortColumn = "Name"; // Valeur par défaut sécurisée
       }

       string direction = ascending ? "ASC" : "DESC";

       // La requête est toujours paramétrée pour les valeurs, même si la structure est dynamique
       string query = $"SELECT * FROM Products ORDER BY {sortColumn} {direction}";

       // Implémentation de la requête...
   }
   ```

## 16.2.2. ORM et sécurité

Les ORM (Object-Relational Mappers) comme Entity Framework offrent une couche d'abstraction qui aide à prévenir les injections SQL.

### Entity Framework et injections SQL

Par défaut, Entity Framework Core utilise des requêtes paramétrées, ce qui offre une protection contre les injections SQL pour la plupart des opérations :

```csharp
// Utilisation sécurisée avec LINQ
public IEnumerable<Product> GetProductsByCategory(int categoryId)
{
    using (var context = new ShopContext())
    {
        // EF génère automatiquement une requête paramétrée sécurisée
        return context.Products
            .Where(p => p.CategoryId == categoryId)
            .ToList();
    }
}
```

### Pièges courants avec les ORM

Même avec un ORM, des vulnérabilités peuvent exister :

1. **Requêtes SQL brutes**

   ```csharp
   // DANGEREUX si userInput n'est pas validé
   var result = context.Products
       .FromSqlRaw("SELECT * FROM Products WHERE Name LIKE '%" + userInput + "%'")
       .ToList();

   // SÉCURISÉ avec paramètres
   var result = context.Products
       .FromSqlRaw("SELECT * FROM Products WHERE Name LIKE '%' + @searchTerm + '%'",
           new SqlParameter("@searchTerm", userInput))
       .ToList();
   ```

2. **EF.Functions.Like (approche recommandée pour les recherches)**

   ```csharp
   // Approche moderne et sécurisée pour Entity Framework Core
   var result = context.Products
       .Where(p => EF.Functions.Like(p.Name, $"%{userInput}%"))
       .ToList();
   ```

3. **Expressions dynamiques avec PredicateBuilder**

   Pour des requêtes complexes et dynamiques, utilisez des outils comme PredicateBuilder (de la bibliothèque LinqKit) plutôt que des chaînes SQL :

   ```csharp
   // Nécessite l'inclusion de LinqKit
   var predicate = PredicateBuilder.False<Product>();

   if (!string.IsNullOrEmpty(nameFilter))
   {
       predicate = predicate.Or(p => p.Name.Contains(nameFilter));
   }

   if (minPrice.HasValue)
   {
       predicate = predicate.Or(p => p.Price >= minPrice.Value);
   }

   var result = context.Products.Where(predicate).ToList();
   ```

## 16.2.3. Validation des entrées

La validation des entrées constitue une couche supplémentaire de protection contre les injections SQL.

### Principes de validation

1. **Valider le type et le format**

   ```csharp
   // Validation d'un identifiant numérique
   if (!int.TryParse(idInput, out int id) || id <= 0)
   {
       return BadRequest("Identifiant invalide");
   }

   // Validation d'une chaîne avec expression régulière
   if (string.IsNullOrEmpty(username) || !Regex.IsMatch(username, @"^[a-zA-Z0-9_]{3,20}$"))
   {
       return BadRequest("Nom d'utilisateur invalide");
   }
   ```

2. **Définir des limites de longueur**

   ```csharp
   // Limitation de la longueur d'un commentaire
   if (commentText.Length > 500)
   {
       commentText = commentText.Substring(0, 500);
   }
   ```

3. **Sanitisation des entrées**

   ```csharp
   // Sanitisation simple pour du HTML (considérez des bibliothèques comme HtmlSanitizer pour des cas complexes)
   string sanitizedInput = HttpUtility.HtmlEncode(userInput);
   ```

### Validation avec les attributs de modèle

Dans ASP.NET Core, utilisez les attributs de validation de données :

```csharp
public class ProductViewModel
{
    [Required(ErrorMessage = "Le nom est obligatoire")]
    [StringLength(100, MinimumLength = 3, ErrorMessage = "Le nom doit contenir entre 3 et 100 caractères")]
    public string Name { get; set; }

    [Range(0.01, 10000, ErrorMessage = "Le prix doit être compris entre 0.01 et 10000")]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }

    [RegularExpression(@"^[A-Z]{2}-\d{4}$", ErrorMessage = "Le code produit doit suivre le format XX-0000")]
    public string ProductCode { get; set; }
}

// Dans votre contrôleur
[HttpPost]
public IActionResult Create(ProductViewModel model)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }

    // Poursuivre avec l'ajout du produit...
}
```

### FluentValidation pour des règles complexes

Pour des validations plus complexes, considérez la bibliothèque FluentValidation :

```csharp
// Installer via NuGet: Install-Package FluentValidation.AspNetCore
public class ProductValidator : AbstractValidator<ProductViewModel>
{
    public ProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Le nom est obligatoire")
            .MinimumLength(3).WithMessage("Le nom doit contenir au moins 3 caractères")
            .MaximumLength(100).WithMessage("Le nom ne peut pas dépasser 100 caractères");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Le prix doit être supérieur à 0")
            .LessThanOrEqualTo(10000).WithMessage("Le prix ne peut pas dépasser 10000");

        RuleFor(x => x.ProductCode)
            .Matches(@"^[A-Z]{2}-\d{4}$").WithMessage("Le code produit doit suivre le format XX-0000")
            .MustAsync(async (code, cancellation) => await IsUniqueProductCodeAsync(code))
            .WithMessage("Ce code produit existe déjà");
    }

    private async Task<bool> IsUniqueProductCodeAsync(string code)
    {
        // Logique pour vérifier l'unicité du code...
        return true; // Exemple simplifié
    }
}

// Configuration dans Program.cs
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddFluentValidationClientsideAdapters();
builder.Services.AddValidatorsFromAssemblyContaining<ProductValidator>();
```

## 16.2.4. Principes de moindre privilège

Le principe de moindre privilège consiste à donner à chaque composant uniquement les accès dont il a besoin pour fonctionner.

### Utilisation de comptes à privilèges limités

1. **Créer des comptes de base de données spécifiques**

   ```sql
   -- Exemple SQL Server - À exécuter par un administrateur
   CREATE LOGIN AppUser WITH PASSWORD = 'ComplexPassword123!';

   USE YourDatabase;
   CREATE USER AppUser FOR LOGIN AppUser;

   -- Accorder uniquement les permissions nécessaires
   GRANT SELECT, INSERT, UPDATE ON dbo.Products TO AppUser;
   GRANT SELECT ON dbo.Categories TO AppUser;

   -- Refuser explicitement les opérations dangereuses
   DENY ALTER ON SCHEMA::dbo TO AppUser;
   DENY DELETE ON dbo.Products TO AppUser;
   ```

2. **Configurer des chaînes de connexion avec des comptes limités**

   ```json
   {
     "ConnectionStrings": {
       "ReadOnlyConnection": "Server=myServer;Database=myDB;User Id=ReadOnlyUser;Password=ReadPw;",
       "StandardConnection": "Server=myServer;Database=myDB;User Id=StandardUser;Password=StandardPw;",
       "AdminConnection": "Server=myServer;Database=myDB;User Id=AdminUser;Password=AdminPw;"
     }
   }
   ```

   ```csharp
   // Utiliser différentes connexions selon le contexte
   public class ProductRepository
   {
       private readonly string _readOnlyConnection;
       private readonly string _standardConnection;
       private readonly string _adminConnection;

       public ProductRepository(IConfiguration configuration)
       {
           _readOnlyConnection = configuration.GetConnectionString("ReadOnlyConnection");
           _standardConnection = configuration.GetConnectionString("StandardConnection");
           _adminConnection = configuration.GetConnectionString("AdminConnection");
       }

       public IEnumerable<Product> GetProducts()
       {
           // Utiliser la connexion en lecture seule
           using (var connection = new SqlConnection(_readOnlyConnection))
           {
               // Implémentation...
           }
       }

       public void UpdateProduct(Product product)
       {
           // Utiliser la connexion standard avec des droits d'écriture limités
           using (var connection = new SqlConnection(_standardConnection))
           {
               // Implémentation...
           }
       }

       public void PurgeOldProducts(DateTime cutoffDate)
       {
           // Opération sensible - utiliser la connexion admin
           using (var connection = new SqlConnection(_adminConnection))
           {
               // Implémentation...
           }
       }
   }
   ```

### Utilisation de procédures stockées

Les procédures stockées peuvent renforcer la sécurité en limitant ce que l'application peut faire :

```sql
-- Procédure stockée sécurisée
CREATE PROCEDURE GetProductsByCategory
    @CategoryId INT
AS
BEGIN
    SELECT * FROM Products WHERE CategoryId = @CategoryId;
END
```

```csharp
// Appel sécurisé de la procédure stockée
public IEnumerable<Product> GetProductsByCategory(int categoryId)
{
    var products = new List<Product>();

    using (var connection = new SqlConnection(_connectionString))
    {
        using (var command = new SqlCommand("GetProductsByCategory", connection))
        {
            command.CommandType = CommandType.StoredProcedure;
            command.Parameters.AddWithValue("@CategoryId", categoryId);

            connection.Open();
            using (var reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    products.Add(new Product
                    {
                        Id = (int)reader["Id"],
                        Name = (string)reader["Name"],
                        // Autres propriétés...
                    });
                }
            }
        }
    }

    return products;
}
```

## 16.2.5. Audit de sécurité des requêtes

L'audit des requêtes vous permet de détecter et de prévenir les tentatives d'injection SQL.

### Journalisation des requêtes

1. **Configurer la journalisation des requêtes avec Entity Framework Core**

   ```csharp
   // Dans Program.cs ou lors de la configuration du DbContext
   services.AddDbContext<ApplicationDbContext>(options =>
       options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"))
              .EnableSensitiveDataLogging() // Uniquement en développement!
              .LogTo(Console.WriteLine, LogLevel.Information));
   ```

2. **Journaliser les requêtes paramétrées avec Dapper**

   ```csharp
   public class DapperLogger : IDapperLogger
   {
       private readonly ILogger<DapperLogger> _logger;

       public DapperLogger(ILogger<DapperLogger> logger)
       {
           _logger = logger;
       }

       public void LogQuery(string sql, object parameters)
       {
           _logger.LogInformation("Exécution de la requête: {Sql} avec paramètres: {@Parameters}",
               sql, parameters);
       }
   }

   // Utilisation
   public IEnumerable<Product> GetProductsByCategory(int categoryId)
   {
       string sql = "SELECT * FROM Products WHERE CategoryId = @CategoryId";
       var parameters = new { CategoryId = categoryId };

       _logger.LogQuery(sql, parameters);

       using (var connection = new SqlConnection(_connectionString))
       {
           return connection.Query<Product>(sql, parameters);
       }
   }
   ```

### Détection des anomalies

1. **Configurer des règles d'alerte**

   ```csharp
   public class SqlInjectionDetector
   {
       private static readonly Regex _suspiciousPatterns = new Regex(
           @"(\b(ALTER|CREATE|DELETE|DROP|EXEC(UTE)?|INSERT( +INTO)?|MERGE|SELECT|TRUNCATE|UPDATE)\b)|(\b(--|\bOR\b|'|\b;|\bUNION\b))",
           RegexOptions.IgnoreCase | RegexOptions.Compiled);

       public bool IsSuspicious(string userInput)
       {
           return _suspiciousPatterns.IsMatch(userInput);
       }

       public void LogSuspiciousActivity(string userInput, string userId, string ipAddress)
       {
           if (IsSuspicious(userInput))
           {
               // Journaliser et/ou alerter
               _logger.LogWarning("Activité suspecte détectée: {Input} de l'utilisateur {UserId} à l'adresse {IpAddress}",
                   userInput, userId, ipAddress);
           }
       }
   }
   ```

2. **Intégration d'un Middleware de détection**

   ```csharp
   public class SqlInjectionDetectionMiddleware
   {
       private readonly RequestDelegate _next;
       private readonly ILogger<SqlInjectionDetectionMiddleware> _logger;
       private readonly SqlInjectionDetector _detector;

       public SqlInjectionDetectionMiddleware(RequestDelegate next,
                                             ILogger<SqlInjectionDetectionMiddleware> logger,
                                             SqlInjectionDetector detector)
       {
           _next = next;
           _logger = logger;
           _detector = detector;
       }

       public async Task InvokeAsync(HttpContext context)
       {
           // Vérifier les paramètres de requête
           foreach (var param in context.Request.Query)
           {
               if (_detector.IsSuspicious(param.Value))
               {
                   _logger.LogWarning("Tentative potentielle d'injection SQL détectée dans les paramètres de requête");
                   // Vous pouvez choisir de bloquer la requête ou simplement la journaliser
                   // context.Response.StatusCode = StatusCodes.Status400BadRequest;
                   // return;
               }
           }

           // Continuer le pipeline
           await _next(context);
       }
   }

   // Enregistrement dans Program.cs
   app.UseMiddleware<SqlInjectionDetectionMiddleware>();
   ```

### Outils et frameworks d'audit

1. **Utilisation de frameworks de sécurité**

   ```csharp
   // Exemple avec OWASP ESAPI pour .NET (nécessite l'installation via NuGet)
   public string SanitizeInput(string input)
   {
       // Nettoyer l'entrée pour prévenir les injections SQL
       return ESAPI.Encoder.SQL.Encode(input);
   }
   ```

2. **Tests de sécurité automatisés**

   ```csharp
   // Test unitaire pour vérifier la sécurité des requêtes
   [Fact]
   public void TestSqlInjectionPrevention()
   {
       // Arrange
       var repository = new ProductRepository(_connectionString);
       string maliciousInput = "1; DROP TABLE Products;--";

       // Act - Si l'injection est évitée, cela ne devrait pas provoquer d'erreur
       var result = repository.GetProductById(maliciousInput);

       // Assert
       Assert.Null(result); // Aucun produit ne devrait être trouvé avec cet ID

       // Vérifiez également que la table existe toujours
       var allProducts = repository.GetAllProducts();
       Assert.NotEmpty(allProducts);
   }
   ```

### Bonnes pratiques pour l'audit de sécurité

1. **Révision régulière du code**
   - Effectuez des révisions de code centrées sur la sécurité
   - Recherchez les cas de concaténation de chaînes dans les requêtes SQL

2. **Tests d'intrusion**
   - Engagez des experts en sécurité pour effectuer des tests d'intrusion
   - Utilisez des outils comme OWASP ZAP pour tester vos applications

3. **Formation continue**
   - Formez vos développeurs aux bonnes pratiques de sécurité
   - Restez à jour sur les nouvelles menaces et vulnérabilités

## Résumé et liste de contrôle de sécurité

Voici une liste de contrôle rapide pour vous assurer que votre application C# est protégée contre les injections SQL :

✅ Utilisez systématiquement des requêtes paramétrées avec ADO.NET
✅ Préférez les ORM comme Entity Framework Core pour la plupart des opérations
✅ Validez toutes les entrées utilisateur avant de les utiliser dans des requêtes
✅ Appliquez le principe de moindre privilège pour les accès à la base de données
✅ Utilisez des procédures stockées pour les opérations complexes
✅ Mettez en place un système de journalisation et d'audit des requêtes
✅ Effectuez des tests de sécurité réguliers
✅ Gardez vos bibliothèques et frameworks à jour

⏭️
