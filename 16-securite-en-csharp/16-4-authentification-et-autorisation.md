# 16.4. Authentification et autorisation

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'authentification et l'autorisation sont deux concepts fondamentaux de la sécurité des applications. En termes simples :
- **L'authentification** vérifie QUI vous êtes (votre identité)
- **L'autorisation** détermine ce que vous avez le DROIT de faire (vos permissions)

Dans ce chapitre, nous explorerons les différentes méthodes pour implémenter ces concepts dans vos applications C#.

## 16.4.1. Systèmes d'authentification

### Comprendre les mécanismes d'authentification

L'authentification est le processus de vérification de l'identité d'un utilisateur. Plusieurs méthodes existent pour authentifier les utilisateurs :

1. **Authentification par mot de passe** : La méthode la plus classique
2. **Authentification par certificat** : Utilisation de certificats numériques
3. **Authentification par jeton (token)** : Utilisation de tokens JWT, OAuth, etc.
4. **Authentification biométrique** : Empreintes digitales, reconnaissance faciale, etc.
5. **Authentification par lien unique** : Envoi d'un lien de connexion par email
6. **Single Sign-On (SSO)** : Connexion unique pour plusieurs applications

### Identity Framework dans ASP.NET Core

ASP.NET Core Identity est le système d'authentification intégré de Microsoft pour les applications .NET. Voici comment l'implémenter dans une application ASP.NET Core :

#### 1. Installation et configuration

```csharp
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);

// Ajouter la base de données pour Identity
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Configurer Identity
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    // Politiques de mot de passe
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireLowercase = true;

    // Verrouillage de compte
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.Lockout.MaxFailedAccessAttempts = 5;

    // Confirmer email et numéro de téléphone
    options.SignIn.RequireConfirmedEmail = true;
    options.SignIn.RequireConfirmedPhoneNumber = false;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

// Configuration des cookies
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.ExpireTimeSpan = TimeSpan.FromMinutes(30);
    options.LoginPath = "/Account/Login";
    options.AccessDeniedPath = "/Account/AccessDenied";
    options.SlidingExpiration = true;
});

var app = builder.Build();

// Middleware d'authentification
app.UseAuthentication();
app.UseAuthorization();
```

#### 2. Définition des modèles

```csharp
// ApplicationUser.cs
public class ApplicationUser : IdentityUser
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime DateOfBirth { get; set; }
    // Propriétés supplémentaires...
}

// ApplicationDbContext.cs
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // DbSets pour vos autres modèles...
}
```

#### 3. Création d'un contrôleur d'authentification

```csharp
[AllowAnonymous]
public class AccountController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly IEmailSender _emailSender;

    public AccountController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        IEmailSender emailSender)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _emailSender = emailSender;
    }

    [HttpGet]
    public IActionResult Register()
    {
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Register(RegisterViewModel model)
    {
        if (ModelState.IsValid)
        {
            var user = new ApplicationUser
            {
                UserName = model.Email,
                Email = model.Email,
                FirstName = model.FirstName,
                LastName = model.LastName,
                DateOfBirth = model.DateOfBirth
            };

            var result = await _userManager.CreateAsync(user, model.Password);

            if (result.Succeeded)
            {
                // Générer un token de confirmation d'email
                var token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
                var confirmationLink = Url.Action("ConfirmEmail", "Account",
                    new { userId = user.Id, token = token }, Request.Scheme);

                // Envoyer l'email de confirmation
                await _emailSender.SendEmailAsync(model.Email, "Confirmez votre compte",
                    $"Veuillez confirmer votre compte en <a href='{confirmationLink}'>cliquant ici</a>.");

                return RedirectToAction("RegisterConfirmation");
            }

            foreach (var error in result.Errors)
            {
                ModelState.AddModelError(string.Empty, error.Description);
            }
        }

        return View(model);
    }

    [HttpGet]
    public IActionResult Login(string returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;

        if (ModelState.IsValid)
        {
            var result = await _signInManager.PasswordSignInAsync(
                model.Email,
                model.Password,
                model.RememberMe,
                lockoutOnFailure: true);

            if (result.Succeeded)
            {
                if (!string.IsNullOrEmpty(returnUrl) && Url.IsLocalUrl(returnUrl))
                {
                    return Redirect(returnUrl);
                }
                else
                {
                    return RedirectToAction("Index", "Home");
                }
            }

            if (result.RequiresTwoFactor)
            {
                return RedirectToAction("SendCode", new { ReturnUrl = returnUrl, RememberMe = model.RememberMe });
            }

            if (result.IsLockedOut)
            {
                return RedirectToAction("Lockout");
            }
            else
            {
                ModelState.AddModelError(string.Empty, "Tentative de connexion invalide.");
                return View(model);
            }
        }

        return View(model);
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return RedirectToAction("Index", "Home");
    }

    // Autres actions : ConfirmEmail, ForgotPassword, ResetPassword, etc.
}
```

### Authentification par token (JWT)

JSON Web Tokens (JWT) est une méthode populaire pour authentifier les utilisateurs dans les API RESTful. Voici comment l'implémenter :

#### 1. Configuration JWT

```csharp
// Dans Program.cs
var builder = WebApplication.CreateBuilder(args);

// Configuration pour JWT
var jwtSettings = builder.Configuration.GetSection("JwtSettings");
var secretKey = Encoding.ASCII.GetBytes(jwtSettings["SecretKey"]);

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = true;
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(secretKey),
        ValidateIssuer = true,
        ValidIssuer = jwtSettings["Issuer"],
        ValidateAudience = true,
        ValidAudience = jwtSettings["Audience"],
        ValidateLifetime = true,
        ClockSkew = TimeSpan.Zero
    };
});

// ...

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
```

#### 2. Service de gestion des tokens JWT

```csharp
public interface IJwtService
{
    string GenerateToken(ApplicationUser user, IList<string> roles);
    ClaimsPrincipal GetPrincipalFromToken(string token);
}

public class JwtService : IJwtService
{
    private readonly IConfiguration _configuration;

    public JwtService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(ApplicationUser user, IList<string> roles)
    {
        var jwtSettings = _configuration.GetSection("JwtSettings");
        var secretKey = Encoding.ASCII.GetBytes(jwtSettings["SecretKey"]);

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id),
            new Claim(ClaimTypes.Name, user.UserName),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        // Ajouter les rôles aux claims
        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddHours(1), // Token valide pendant 1 heure
            Issuer = jwtSettings["Issuer"],
            Audience = jwtSettings["Audience"],
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(secretKey),
                SecurityAlgorithms.HmacSha256Signature)
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);

        return tokenHandler.WriteToken(token);
    }

    public ClaimsPrincipal GetPrincipalFromToken(string token)
    {
        var jwtSettings = _configuration.GetSection("JwtSettings");
        var secretKey = Encoding.ASCII.GetBytes(jwtSettings["SecretKey"]);

        var tokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(secretKey),
            ValidateIssuer = true,
            ValidIssuer = jwtSettings["Issuer"],
            ValidateAudience = true,
            ValidAudience = jwtSettings["Audience"],
            ValidateLifetime = false // Pour la validation de tokens expirés
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        var principal = tokenHandler.ValidateToken(token, tokenValidationParameters, out var securityToken);

        if (!(securityToken is JwtSecurityToken jwtSecurityToken) ||
            !jwtSecurityToken.Header.Alg.Equals(SecurityAlgorithms.HmacSha256,
            StringComparison.InvariantCultureIgnoreCase))
        {
            throw new SecurityTokenException("Token invalide");
        }

        return principal;
    }
}
```

#### 3. Contrôleur d'authentification JWT

```csharp
[Route("api/[controller]")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly IJwtService _jwtService;

    public AuthController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        IJwtService jwtService)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _jwtService = jwtService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        var user = await _userManager.FindByNameAsync(model.Username);

        if (user == null)
        {
            return Unauthorized(new { message = "Nom d'utilisateur ou mot de passe incorrect" });
        }

        var result = await _signInManager.CheckPasswordSignInAsync(user, model.Password, lockoutOnFailure: true);

        if (result.Succeeded)
        {
            var roles = await _userManager.GetRolesAsync(user);
            var token = _jwtService.GenerateToken(user, roles);

            return Ok(new
            {
                token = token,
                expiration = DateTime.UtcNow.AddHours(1),
                user = new
                {
                    id = user.Id,
                    userName = user.UserName,
                    email = user.Email,
                    roles = roles
                }
            });
        }

        if (result.IsLockedOut)
        {
            return StatusCode(StatusCodes.Status423Locked, new { message = "Compte verrouillé" });
        }

        return Unauthorized(new { message = "Nom d'utilisateur ou mot de passe incorrect" });
    }

    [HttpPost("refresh-token")]
    public async Task<IActionResult> RefreshToken([FromBody] RefreshTokenModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        try
        {
            var principal = _jwtService.GetPrincipalFromToken(model.Token);
            var userId = principal.FindFirst(ClaimTypes.NameIdentifier)?.Value;

            if (string.IsNullOrEmpty(userId))
            {
                return BadRequest(new { message = "Token invalide" });
            }

            var user = await _userManager.FindByIdAsync(userId);

            if (user == null)
            {
                return BadRequest(new { message = "Utilisateur introuvable" });
            }

            var roles = await _userManager.GetRolesAsync(user);
            var newToken = _jwtService.GenerateToken(user, roles);

            return Ok(new
            {
                token = newToken,
                expiration = DateTime.UtcNow.AddHours(1)
            });
        }
        catch (Exception)
        {
            return BadRequest(new { message = "Échec du rafraîchissement du token" });
        }
    }
}
```

### Authentification externe

ASP.NET Core Identity facilite l'intégration des fournisseurs d'authentification externes comme Google, Facebook, Microsoft, etc.

```csharp
// Dans Program.cs
builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = builder.Configuration["Authentication:Google:ClientId"];
        options.ClientSecret = builder.Configuration["Authentication:Google:ClientSecret"];
    })
    .AddFacebook(options =>
    {
        options.AppId = builder.Configuration["Authentication:Facebook:AppId"];
        options.AppSecret = builder.Configuration["Authentication:Facebook:AppSecret"];
    })
    .AddMicrosoftAccount(options =>
    {
        options.ClientId = builder.Configuration["Authentication:Microsoft:ClientId"];
        options.ClientSecret = builder.Configuration["Authentication:Microsoft:ClientSecret"];
    });
```

Ajoutez ensuite les méthodes correspondantes dans votre contrôleur d'authentification :

```csharp
[HttpPost]
[AllowAnonymous]
public IActionResult ExternalLogin(string provider, string returnUrl = null)
{
    var redirectUrl = Url.Action("ExternalLoginCallback", "Account",
        new { ReturnUrl = returnUrl });
    var properties = _signInManager.ConfigureExternalAuthenticationProperties(
        provider, redirectUrl);
    return Challenge(properties, provider);
}

[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> ExternalLoginCallback(string returnUrl = null, string remoteError = null)
{
    // Code pour gérer la réponse du fournisseur externe
    // et créer/connecter un compte local associé
}
```

## 16.4.2. RBAC et ABAC

### Role-Based Access Control (RBAC)

Le contrôle d'accès basé sur les rôles (RBAC) est une approche qui accorde les permissions en fonction des rôles attribués aux utilisateurs.

#### 1. Configuration des rôles avec Identity

```csharp
// Dans Program.cs
// Assurez-vous que RoleManager est configuré
builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

// Dans une méthode d'initialisation
public static async Task InitializeRolesAsync(IServiceProvider serviceProvider)
{
    var roleManager = serviceProvider.GetRequiredService<RoleManager<IdentityRole>>();
    var userManager = serviceProvider.GetRequiredService<UserManager<ApplicationUser>>();

    string[] roleNames = { "Admin", "Manager", "User" };

    foreach (var roleName in roleNames)
    {
        if (!await roleManager.RoleExistsAsync(roleName))
        {
            await roleManager.CreateAsync(new IdentityRole(roleName));
        }
    }

    // Créer un utilisateur administrateur par défaut
    var adminUser = new ApplicationUser
    {
        UserName = "admin@example.com",
        Email = "admin@example.com",
        EmailConfirmed = true
    };

    var userExists = await userManager.FindByEmailAsync(adminUser.Email);

    if (userExists == null)
    {
        var result = await userManager.CreateAsync(adminUser, "Admin@123456");

        if (result.Succeeded)
        {
            await userManager.AddToRoleAsync(adminUser, "Admin");
        }
    }
}
```

#### 2. Utilisation des attributs d'autorisation basée sur les rôles

```csharp
// Dans un contrôleur ou une action
[Authorize(Roles = "Admin")]
public IActionResult AdminDashboard()
{
    return View();
}

[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports()
{
    return View();
}

[AllowAnonymous]
public IActionResult PublicInfo()
{
    return View();
}
```

#### 3. Vérification des rôles dans le code

```csharp
// Dans un service ou un contrôleur
public class UserService
{
    private readonly UserManager<ApplicationUser> _userManager;

    public UserService(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }

    public async Task<bool> IsInRoleAsync(string userId, string role)
    {
        var user = await _userManager.FindByIdAsync(userId);

        if (user == null)
        {
            return false;
        }

        return await _userManager.IsInRoleAsync(user, role);
    }

    public async Task<bool> CanEditUserAsync(string currentUserId, string targetUserId)
    {
        // Les administrateurs peuvent modifier n'importe quel utilisateur
        if (await IsInRoleAsync(currentUserId, "Admin"))
        {
            return true;
        }

        // Les managers peuvent modifier d'autres utilisateurs sauf les admins
        if (await IsInRoleAsync(currentUserId, "Manager"))
        {
            return !await IsInRoleAsync(targetUserId, "Admin");
        }

        // Les utilisateurs ne peuvent modifier que leur propre profil
        return currentUserId == targetUserId;
    }
}
```

#### 4. Vérification des rôles dans les vues Razor

```html
@using Microsoft.AspNetCore.Identity
@inject UserManager<ApplicationUser> UserManager
@inject SignInManager<ApplicationUser> SignInManager

<nav>
    <ul>
        <li><a href="/">Accueil</a></li>

        @if (SignInManager.IsSignedIn(User))
        {
            <li><a href="/profile">Mon profil</a></li>

            @if (User.IsInRole("Admin"))
            {
                <li><a href="/admin">Administration</a></li>
            }

            @if (User.IsInRole("Admin") || User.IsInRole("Manager"))
            {
                <li><a href="/reports">Rapports</a></li>
            }

            <li>
                <form asp-controller="Account" asp-action="Logout" method="post">
                    <button type="submit">Déconnexion</button>
                </form>
            </li>
        }
        else
        {
            <li><a href="/account/login">Connexion</a></li>
            <li><a href="/account/register">Inscription</a></li>
        }
    </ul>
</nav>
```

### Attribute-Based Access Control (ABAC)

Le contrôle d'accès basé sur les attributs (ABAC) est plus flexible que le RBAC car il utilise une combinaison d'attributs de l'utilisateur, de la ressource et du contexte pour prendre des décisions d'autorisation.

#### 1. Création d'une stratégie d'autorisation personnalisée

```csharp
// Dans Program.cs
builder.Services.AddAuthorization(options =>
{
    // Stratégie basée sur l'âge
    options.AddPolicy("AdultsOnly", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "DateOfBirth") &&
            DateTime.TryParse(context.User.FindFirst("DateOfBirth").Value, out var dateOfBirth) &&
            dateOfBirth.AddYears(18) <= DateTime.Today));

    // Stratégie basée sur le département
    options.AddPolicy("FinanceDepartment", policy =>
        policy.RequireClaim("Department", "Finance"));

    // Stratégie combinant rôle et département
    options.AddPolicy("SalesManager", policy =>
        policy.RequireRole("Manager")
            .RequireClaim("Department", "Sales"));

    // Stratégie avec exigence personnalisée
    options.AddPolicy("DocumentCreator", policy =>
        policy.Requirements.Add(new DocumentCreatorRequirement()));
});

// Enregistrer le gestionnaire d'exigences
builder.Services.AddScoped<IAuthorizationHandler, DocumentCreatorHandler>();
```

#### 2. Créer des exigences personnalisées

```csharp
// DocumentCreatorRequirement.cs
public class DocumentCreatorRequirement : IAuthorizationRequirement
{
    // Cette classe peut contenir des paramètres pour la vérification
}

// DocumentCreatorHandler.cs
public class DocumentCreatorHandler : AuthorizationHandler<DocumentCreatorRequirement>
{
    private readonly IDocumentService _documentService;

    public DocumentCreatorHandler(IDocumentService documentService)
    {
        _documentService = documentService;
    }

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        DocumentCreatorRequirement requirement)
    {
        // Aucune ressource à vérifier
        if (context.Resource == null)
        {
            return;
        }

        // Vérifier si la ressource est un document
        if (!(context.Resource is Document document))
        {
            return;
        }

        // Vérifier si l'utilisateur est le créateur du document
        var userId = context.User.FindFirstValue(ClaimTypes.NameIdentifier);

        if (document.CreatedById == userId)
        {
            context.Succeed(requirement);
        }
    }
}
```

#### 3. Utilisation des stratégies d'autorisation

```csharp
// Sur un contrôleur ou une action
[Authorize(Policy = "AdultsOnly")]
public IActionResult AdultContent()
{
    return View();
}

[Authorize(Policy = "FinanceDepartment")]
public IActionResult FinancialReports()
{
    return View();
}

// Autorisation basée sur une ressource dans l'action
public async Task<IActionResult> EditDocument(int id)
{
    var document = await _documentService.GetDocumentByIdAsync(id);

    if (document == null)
    {
        return NotFound();
    }

    var authorizationResult = await _authorizationService.AuthorizeAsync(
        User, document, "DocumentCreator");

    if (!authorizationResult.Succeeded)
    {
        return Forbid();
    }

    return View(document);
}
```

## 16.4.3. Gestion des tokens

La gestion des tokens est essentielle pour les systèmes d'authentification modernes, en particulier pour les API RESTful et les applications SPA (Single Page Applications).

### Tokens JWT (JSON Web Tokens)

Les JWT sont des tokens autonomes qui contiennent toutes les informations nécessaires pour authentifier un utilisateur.

#### 1. Structure d'un JWT

Un JWT se compose de trois parties : l'en-tête (header), la charge utile (payload) et la signature.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

- **Header** : Contient le type de token et l'algorithme de signature
- **Payload** : Contient les claims (informations sur l'utilisateur)
- **Signature** : Garantit l'intégrité du token

#### 2. Implémentation d'un service de refresh token

```csharp
// RefreshToken.cs
public class RefreshToken
{
    public string Token { get; set; }
    public string JwtId { get; set; }
    public DateTime CreationDate { get; set; }
    public DateTime ExpiryDate { get; set; }
    public bool Used { get; set; }
    public bool Invalidated { get; set; }
    public string UserId { get; set; }
    public ApplicationUser User { get; set; }
}

// IRefreshTokenService.cs
public interface IRefreshTokenService
{
    Task<string> GenerateRefreshTokenAsync(string userId, string jwtId);
    Task<(bool isValid, string userId, string jwtId)> ValidateRefreshTokenAsync(string token);
    Task InvalidateRefreshTokenAsync(string token);
}

// RefreshTokenService.cs
public class RefreshTokenService : IRefreshTokenService
{
    private readonly ApplicationDbContext _context;

    public RefreshTokenService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<string> GenerateRefreshTokenAsync(string userId, string jwtId)
    {
        // Générer un token aléatoire
        var randomBytes = new byte[32];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(randomBytes);
        }

        var refreshToken = new RefreshToken
        {
            Token = Convert.ToBase64String(randomBytes),
            JwtId = jwtId,
            UserId = userId,
            CreationDate = DateTime.UtcNow,
            ExpiryDate = DateTime.UtcNow.AddDays(7), // Valide pendant 7 jours
            Used = false,
            Invalidated = false
        };

        // Invalider les anciens tokens
        var existingTokens = await _context.RefreshTokens
            .Where(rt => rt.UserId == userId && !rt.Used && !rt.Invalidated)
            .ToListAsync();

        foreach (var token in existingTokens)
        {
            token.Invalidated = true;
        }

        await _context.RefreshTokens.AddAsync(refreshToken);
        await _context.SaveChangesAsync();

        return refreshToken.Token;
    }

    public async Task<(bool isValid, string userId, string jwtId)> ValidateRefreshTokenAsync(string token)
    {
        var refreshToken = await _context.RefreshTokens
            .Include(rt => rt.User)
            .FirstOrDefaultAsync(rt => rt.Token == token);

        if (refreshToken == null)
        {
            return (false, null, null);
        }

        // Vérifier si le token est valide
        if (refreshToken.Used ||
            refreshToken.Invalidated ||
            DateTime.UtcNow > refreshToken.ExpiryDate)
        {
            return (false, null, null);
        }

        // Marquer le token comme utilisé
        refreshToken.Used = true;
        await _context.SaveChangesAsync();

        return (true, refreshToken.UserId, refreshToken.JwtId);
    }

    public async Task InvalidateRefreshTokenAsync(string token)
    {
        var refreshToken = await _context.RefreshTokens
            .FirstOrDefaultAsync(rt => rt.Token == token);

        if (refreshToken != null)
        {
            refreshToken.Invalidated = true;
            await _context.SaveChangesAsync();
        }
    }
}
```

#### 3. Intégration dans le contrôleur d'authentification

```csharp
// AuthController.cs
[Route("api/[controller]")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly IJwtService _jwtService;
    private readonly IRefreshTokenService _refreshTokenService;

    public AuthController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        IJwtService jwtService,
        IRefreshTokenService refreshTokenService)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _jwtService = jwtService;
        _refreshTokenService = refreshTokenService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        var user = await _userManager.FindByNameAsync(model.Username);

        if (user == null)
        {
            return Unauthorized(new { message = "Nom d'utilisateur ou mot de passe incorrect" });
        }

        var result = await _signInManager.CheckPasswordSignInAsync(user, model.Password, lockoutOnFailure: true);

        if (result.Succeeded)
        {
            var roles = await _userManager.GetRolesAsync(user);
            var (token, jwtId) = _jwtService.GenerateTokenWithId(user, roles);

            // Générer un refresh token
            var refreshToken = await _refreshTokenService.GenerateRefreshTokenAsync(user.Id, jwtId);

            return Ok(new
            {
                token = token,
                refreshToken = refreshToken,
                expiration = DateTime.UtcNow.AddHours(1),
                user = new
                {
                    id = user.Id,
                    userName = user.UserName,
                    email = user.Email,
                    roles = roles
                }
            });
        }

        if (result.IsLockedOut)
        {
            return StatusCode(StatusCodes.Status423Locked, new { message = "Compte verrouillé" });
        }

        return Unauthorized(new { message = "Nom d'utilisateur ou mot de passe incorrect" });
    }

    [HttpPost("refresh-token")]
    public async Task<IActionResult> RefreshToken([FromBody] RefreshTokenModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        try
        {
            // Valider le JWT pour extraire le jti (JWT ID)
            var principal = _jwtService.GetPrincipalFromToken(model.Token);
            var jwtId = principal.Claims.SingleOrDefault(c => c.Type == JwtRegisteredClaimNames.Jti)?.Value;
            var userId = principal.FindFirst(ClaimTypes.NameIdentifier)?.Value;

            if (string.IsNullOrEmpty(jwtId) || string.IsNullOrEmpty(userId))
            {
                return BadRequest(new { message = "Token invalide" });
            }

            // Valider le refresh token
            var (isValid, tokenUserId, tokenJwtId) = await _refreshTokenService.ValidateRefreshTokenAsync(model.RefreshToken);

            if (!isValid || tokenUserId != userId || tokenJwtId != jwtId)
            {
                return BadRequest(new { message = "Refresh token invalide" });
            }

            // Générer un nouveau token
            var user = await _userManager.FindByIdAsync(userId);
            var roles = await _userManager.GetRolesAsync(user);
            var (newToken, newJwtId) = _jwtService.GenerateTokenWithId(user, roles);

            // Générer un nouveau refresh token
            var newRefreshToken = await _refreshTokenService.GenerateRefreshTokenAsync(user.Id, newJwtId);

            return Ok(new
            {
                token = newToken,
                refreshToken = newRefreshToken,
                expiration = DateTime.UtcNow.AddHours(1)
            });
        }
        catch (Exception)
        {
            return BadRequest(new { message = "Échec du rafraîchissement du token" });
        }
    }

    [HttpPost("logout")]
    [Authorize]
    public async Task<IActionResult> Logout([FromBody] LogoutModel model)
    {
        if (!string.IsNullOrEmpty(model.RefreshToken))
        {
            // Invalider le refresh token
            await _refreshTokenService.InvalidateRefreshTokenAsync(model.RefreshToken);
        }

        return Ok(new { message = "Déconnexion réussie" });
    }
}
```

### Sécurité des tokens

Pour sécuriser correctement les tokens JWT, suivez ces bonnes pratiques :

1. **Utiliser HTTPS** pour toutes les communications
2. **Limiter la durée de validité** des tokens (1 heure pour les JWT, 7-14 jours pour les refresh tokens)
3. **Stocker les tokens correctement** :
   - JWT : stocké dans la mémoire du client (pas dans localStorage)
   - Refresh tokens : stockés dans un cookie HTTP-only avec l'attribut Secure
4. **Valider tous les aspects du token** (signature, émetteur, audience, expiration)
5. **Mettre en place une liste noire** pour les tokens révoqués
6. **Utiliser des claims supplémentaires** pour renforcer la sécurité (comme l'empreinte digitale du navigateur)

## 16.4.4. Multi-factor authentication

L'authentification multi-facteurs (MFA) renforce considérablement la sécurité en exigeant plusieurs formes de vérification.

### Principe du MFA

L'authentification multi-facteurs s'appuie sur la combinaison de plusieurs types de facteurs :
1. **Quelque chose que vous savez** : mot de passe, PIN
2. **Quelque chose que vous possédez** : téléphone, token matériel, email
3. **Quelque chose que vous êtes** : empreinte digitale, reconnaissance faciale

### Configuration du MFA avec ASP.NET Core Identity

#### 1. Configuration initiale

```csharp
// Dans Program.cs
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    // Configuration générale d'Identity
    // ...

    // Activer l'authentification à deux facteurs
    options.SignIn.RequireConfirmedAccount = true;
    options.SignIn.RequireTwoFactorAuthentication = true;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders()
.AddTokenProvider<EmailTokenProvider<ApplicationUser>>("Email")
.AddTokenProvider<PhoneNumberTokenProvider<ApplicationUser>>("Phone")
.AddTokenProvider<AuthenticatorTokenProvider<ApplicationUser>>("Authenticator");
```

#### 2. Implémentation pour application mobile (Google Authenticator, Microsoft Authenticator, etc.)

```csharp
// Dans le contrôleur de gestion du compte
public class ManageController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly UrlEncoder _urlEncoder;

    private const string AuthenticatorUriFormat = "otpauth://totp/{0}:{1}?secret={2}&issuer={0}&digits=6";

    public ManageController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        UrlEncoder urlEncoder)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _urlEncoder = urlEncoder;
    }

    [HttpGet]
    public async Task<IActionResult> EnableAuthenticator()
    {
        var user = await _userManager.GetUserAsync(User);

        if (user == null)
        {
            return NotFound();
        }

        var model = new EnableAuthenticatorViewModel();
        await LoadSharedKeyAndQrCodeUriAsync(user, model);

        return View(model);
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> EnableAuthenticator(EnableAuthenticatorViewModel model)
    {
        var user = await _userManager.GetUserAsync(User);

        if (user == null)
        {
            return NotFound();
        }

        if (!ModelState.IsValid)
        {
            await LoadSharedKeyAndQrCodeUriAsync(user, model);
            return View(model);
        }

        // Vérifier le code
        var verificationCode = model.Code.Replace(" ", string.Empty).Replace("-", string.Empty);
        var is2faTokenValid = await _userManager.VerifyTwoFactorTokenAsync(
            user, _userManager.Options.Tokens.AuthenticatorTokenProvider, verificationCode);

        if (!is2faTokenValid)
        {
            ModelState.AddModelError("Code", "Le code de vérification est invalide.");
            await LoadSharedKeyAndQrCodeUriAsync(user, model);
            return View(model);
        }

        await _userManager.SetTwoFactorEnabledAsync(user, true);

        // Générer des codes de récupération
        var recoveryCodes = await _userManager.GenerateNewTwoFactorRecoveryCodesAsync(user, 10);

        return RedirectToAction(nameof(ShowRecoveryCodes), new { RecoveryCodes = recoveryCodes });
    }

    private async Task LoadSharedKeyAndQrCodeUriAsync(ApplicationUser user, EnableAuthenticatorViewModel model)
    {
        // Charger ou générer la clé partagée
        var unformattedKey = await _userManager.GetAuthenticatorKeyAsync(user);

        if (string.IsNullOrEmpty(unformattedKey))
        {
            await _userManager.ResetAuthenticatorKeyAsync(user);
            unformattedKey = await _userManager.GetAuthenticatorKeyAsync(user);
        }

        model.SharedKey = FormatKey(unformattedKey);

        var email = await _userManager.GetEmailAsync(user);
        model.AuthenticatorUri = GenerateQrCodeUri(email, unformattedKey);
    }

    private string FormatKey(string unformattedKey)
    {
        var result = new StringBuilder();

        for (var i = 0; i < unformattedKey.Length; i++)
        {
            if (i > 0 && i % 4 == 0)
            {
                result.Append(" ");
            }

            result.Append(unformattedKey[i]);
        }

        return result.ToString().ToLowerInvariant();
    }

    private string GenerateQrCodeUri(string email, string unformattedKey)
    {
        return string.Format(
            AuthenticatorUriFormat,
            _urlEncoder.Encode("VotreApplication"),
            _urlEncoder.Encode(email),
            unformattedKey);
    }
}
```

#### 3. Gestion de la connexion avec 2FA

```csharp
// Dans AccountController.cs
[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> LoginWith2fa(bool rememberMe, string returnUrl = null)
{
    // Vérifier que l'utilisateur a déjà saisi son mot de passe
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

    if (user == null)
    {
        throw new InvalidOperationException("Impossible de charger l'utilisateur pour l'authentification à deux facteurs.");
    }

    var model = new LoginWith2faViewModel { RememberMe = rememberMe };
    ViewData["ReturnUrl"] = returnUrl;

    return View(model);
}

[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> LoginWith2fa(LoginWith2faViewModel model, bool rememberMe, string returnUrl = null)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }

    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

    if (user == null)
    {
        throw new InvalidOperationException("Impossible de charger l'utilisateur pour l'authentification à deux facteurs.");
    }

    var authenticatorCode = model.TwoFactorCode.Replace(" ", string.Empty).Replace("-", string.Empty);

    var result = await _signInManager.TwoFactorAuthenticatorSignInAsync(
        authenticatorCode, rememberMe, model.RememberMachine);

    if (result.Succeeded)
    {
        _logger.LogInformation("Utilisateur connecté avec 2FA.");
        return RedirectToLocal(returnUrl);
    }

    if (result.IsLockedOut)
    {
        _logger.LogWarning("Compte utilisateur verrouillé.");
        return RedirectToAction(nameof(Lockout));
    }

    _logger.LogWarning("Code d'authentificateur invalide saisi.");
    ModelState.AddModelError(string.Empty, "Code d'authentificateur invalide.");

    return View(model);
}

[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> LoginWithRecoveryCode(string returnUrl = null)
{
    // Vérifier que l'utilisateur a déjà saisi son mot de passe
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

    if (user == null)
    {
        throw new InvalidOperationException("Impossible de charger l'utilisateur pour l'authentification avec code de récupération.");
    }

    ViewData["ReturnUrl"] = returnUrl;

    return View();
}

[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> LoginWithRecoveryCode(LoginWithRecoveryCodeViewModel model, string returnUrl = null)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }

    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

    if (user == null)
    {
        throw new InvalidOperationException("Impossible de charger l'utilisateur pour l'authentification avec code de récupération.");
    }

    var recoveryCode = model.RecoveryCode.Replace(" ", string.Empty);

    var result = await _signInManager.TwoFactorRecoveryCodeSignInAsync(recoveryCode);

    if (result.Succeeded)
    {
        _logger.LogInformation("Utilisateur connecté avec un code de récupération.");
        return RedirectToLocal(returnUrl);
    }

    if (result.IsLockedOut)
    {
        _logger.LogWarning("Compte utilisateur verrouillé.");
        return RedirectToAction(nameof(Lockout));
    }

    _logger.LogWarning("Code de récupération invalide saisi.");
    ModelState.AddModelError(string.Empty, "Code de récupération invalide.");

    return View(model);
}
```

### Authentification par e-mail ou SMS

ASP.NET Core Identity supporte également l'envoi de codes de vérification par e-mail ou SMS.

```csharp
// Dans Program.cs, configurez les services pour l'envoi d'emails ou de SMS
builder.Services.AddTransient<IEmailSender, EmailSender>();
builder.Services.AddTransient<ISmsSender, SmsSender>();

// Dans AccountController.cs
[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> LoginWithCode(string provider, bool rememberMe, string returnUrl = null)
{
    // Vérifier que l'utilisateur a déjà saisi son mot de passe
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

    if (user == null)
    {
        throw new InvalidOperationException("Impossible de charger l'utilisateur pour l'authentification à deux facteurs.");
    }

    // Générer et envoyer le code
    var code = await _userManager.GenerateTwoFactorTokenAsync(user, provider);

    if (string.IsNullOrWhiteSpace(code))
    {
        return View("Error");
    }

    if (provider == "Email")
    {
        await _emailSender.SendEmailAsync(
            await _userManager.GetEmailAsync(user),
            "Code de sécurité",
            $"Votre code de sécurité est : {code}");
    }
    else if (provider == "Phone")
    {
        await _smsSender.SendSmsAsync(
            await _userManager.GetPhoneNumberAsync(user),
            $"Votre code de sécurité est : {code}");
    }

    return RedirectToAction(nameof(VerifyCode), new
    {
        Provider = provider,
        RememberMe = rememberMe,
        ReturnUrl = returnUrl
    });
}

[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> VerifyCode(string provider, bool rememberMe, string returnUrl = null)
{
    // Vérifier que l'utilisateur a déjà saisi son mot de passe
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

    if (user == null)
    {
        return View("Error");
    }

    return View(new VerifyCodeViewModel
    {
        Provider = provider,
        RememberMe = rememberMe,
        ReturnUrl = returnUrl
    });
}

[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> VerifyCode(VerifyCodeViewModel model)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }

    var result = await _signInManager.TwoFactorSignInAsync(
        model.Provider,
        model.Code,
        model.RememberMe,
        model.RememberBrowser);

    if (result.Succeeded)
    {
        return RedirectToLocal(model.ReturnUrl);
    }

    if (result.IsLockedOut)
    {
        return View("Lockout");
    }

    ModelState.AddModelError(string.Empty, "Code invalide.");
    return View(model);
}
```

## 16.4.5. Gestion des sessions

La gestion des sessions permet de maintenir l'état de l'utilisateur entre les requêtes et de contrôler la durée de validité de l'authentification.

### Sessions dans ASP.NET Core

#### 1. Configuration des sessions

```csharp
// Dans Program.cs
// Ajouter les services de session
builder.Services.AddDistributedMemoryCache(); // En production, utilisez Redis ou SQL Server
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30); // Durée d'inactivité avant expiration
    options.Cookie.HttpOnly = true; // Le cookie n'est pas accessible via JavaScript
    options.Cookie.IsEssential = true; // Le cookie est essentiel pour le fonctionnement de l'application
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // Cookie transmis uniquement en HTTPS
});

var app = builder.Build();

// Ajouter le middleware de session (avant UseRouting)
app.UseSession();
```

#### 2. Utilisation des sessions

```csharp
// Dans un contrôleur
public class HomeController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        // Lire une valeur de session
        var username = HttpContext.Session.GetString("Username");

        // Stocker une valeur en session
        HttpContext.Session.SetString("LastVisit", DateTime.Now.ToString());

        // Stocker un objet complexe (nécessite une sérialisation)
        var user = new UserInfo { Id = 1, Name = "John", Role = "Admin" };
        HttpContext.Session.SetString("UserInfo", JsonSerializer.Serialize(user));

        return View();
    }
}
```

### Configuration des cookies d'authentification

ASP.NET Core utilise des cookies pour maintenir l'état d'authentification entre les requêtes.

```csharp
// Dans Program.cs
builder.Services.ConfigureApplicationCookie(options =>
{
    // Paramètres du cookie d'authentification
    options.Cookie.Name = "YourAppAuth";
    options.Cookie.HttpOnly = true;
    options.ExpireTimeSpan = TimeSpan.FromMinutes(60);
    options.SlidingExpiration = true; // Réinitialise le temps d'expiration à chaque requête
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // HTTPS uniquement

    // Redirection en cas d'accès non autorisé
    options.LoginPath = "/Account/Login";
    options.AccessDeniedPath = "/Account/AccessDenied";

    // Protection avancée contre la falsification
    options.Cookie.SameSite = SameSiteMode.Strict; // Protège contre les attaques CSRF
});
```

### Bonnes pratiques de gestion des sessions

1. **Limiter les données stockées en session** :
   - Stockez uniquement les informations essentielles
   - Évitez de stocker des données sensibles
   - Préférez stocker des identifiants plutôt que des objets complets

2. **Configurer des timeouts appropriés** :
   - Sessions courtes pour les applications sensibles (15-30 minutes)
   - Option de "Remember Me" avec vérification périodique

3. **Mettre en place un mécanisme de déconnexion** :
   - Déconnexion explicite
   - Déconnexion automatique après inactivité
   - Gestion des sessions simultanées (option de déconnexion des autres sessions)

```csharp
// Exemple de déconnexion avec gestion des sessions
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Logout()
{
    // Nettoyer les données de session
    HttpContext.Session.Clear();

    // Déconnecter l'utilisateur
    await _signInManager.SignOutAsync();

    // Informer l'utilisateur
    _logger.LogInformation("Utilisateur déconnecté.");

    return RedirectToAction("Index", "Home");
}

// Exemple d'action pour terminer les autres sessions
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> LogoutOtherSessions()
{
    var user = await _userManager.GetUserAsync(User);

    if (user == null)
    {
        return NotFound();
    }

    // Changer le jeton de sécurité pour invalider les autres sessions
    await _userManager.UpdateSecurityStampAsync(user);

    // Renouveler le cookie pour la session actuelle
    await _signInManager.RefreshSignInAsync(user);

    StatusMessage = "Toutes les autres sessions ont été déconnectées.";

    return RedirectToAction(nameof(Index));
}
```

4. **Sécuriser les cookies de session** :
   - Attribut HttpOnly pour empêcher l'accès JavaScript
   - Attribut Secure pour limiter aux connexions HTTPS
   - Attribut SameSite pour prévenir les attaques CSRF
   - Protection avec anti-forgery tokens pour les opérations sensibles

```csharp
// Utilisation de l'attribut ValidateAntiForgeryToken pour protéger contre les attaques CSRF
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> UpdateProfile(ProfileViewModel model)
{
    // Code pour mettre à jour le profil
}

// Dans la vue Razor, inclure le jeton anti-falsification
@using (Html.BeginForm("UpdateProfile", "Account", FormMethod.Post))
{
    @Html.AntiForgeryToken()
    <!-- Champs du formulaire -->
    <button type="submit">Mettre à jour</button>
}
```

5. **Considérer des alternatives pour les applications à grande échelle** :
   - Redis pour le stockage de session distribué
   - Base de données pour une persistance à long terme
   - JWT pour les API sans état (stateless)

```csharp
// Configuration du stockage de session avec Redis
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "SampleInstance";
});

builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});
```

### Gestion des sessions dans les API REST avec JWT

Pour les API REST qui utilisent JWT, la "session" est gérée côté client. Voici quelques bonnes pratiques :

1. **Utiliser des expirations courtes pour les JWT** (15-60 minutes)
2. **Mettre en place un système de refresh tokens** pour renouveler l'accès
3. **Maintenir une liste noire de tokens révoqués** pour la déconnexion
4. **Utiliser une signature forte** et vérifier tous les aspects du token

```csharp
// Exemple de service pour gérer une liste noire de tokens JWT
public class TokenBlacklistService
{
    private readonly IDistributedCache _cache;

    public TokenBlacklistService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task BlacklistTokenAsync(string jwtId, DateTime expiryDate)
    {
        var expiryTime = expiryDate - DateTime.UtcNow;

        // Stocker dans la liste noire jusqu'à l'expiration du token
        await _cache.SetStringAsync(
            $"blacklist_{jwtId}",
            DateTime.UtcNow.ToString(),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = expiryTime
            });
    }

    public async Task<bool> IsTokenBlacklistedAsync(string jwtId)
    {
        var value = await _cache.GetStringAsync($"blacklist_{jwtId}");
        return value != null;
    }
}
```

En intégrant ce service dans le middleware d'authentification JWT, vous pouvez vérifier si un token a été révoqué :

```csharp
// Dans Program.cs
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    // Configuration JWT standard
    // ...

    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = async context =>
        {
            var tokenBlacklistService = context.HttpContext.RequestServices
                .GetRequiredService<TokenBlacklistService>();

            var jwtId = context.SecurityToken.Id;

            if (await tokenBlacklistService.IsTokenBlacklistedAsync(jwtId))
            {
                context.Fail("Token has been revoked");
            }
        }
    };
});
```

## Résumé

L'authentification et l'autorisation sont des aspects critiques de la sécurité des applications. En résumé :

1. **Authentification** :
   - Utilisez des systèmes établis comme ASP.NET Core Identity
   - Implémentez l'authentification multi-facteurs (MFA)
   - Gérez soigneusement les tokens et les sessions

2. **Autorisation** :
   - Utilisez le RBAC pour un contrôle d'accès basé sur les rôles
   - Implémentez l'ABAC pour des règles d'accès plus flexibles
   - Créez des politiques d'autorisation personnalisées pour les besoins complexes

3. **Sécurité** :
   - Configurez correctement les cookies et les sessions
   - Utilisez HTTPS pour toutes les communications
   - Protégez contre les attaques CSRF avec les jetons anti-falsification
   - Mettez en place des mécanismes de déconnexion efficaces

En suivant ces pratiques, vous construirez des applications C# robustes et sécurisées, offrant une expérience utilisateur fluide tout en protégeant les données sensibles.

⏭️
