# 12.7. Authentification et autorisation

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'authentification et l'autorisation sont deux concepts fondamentaux de la sécurité des applications web. Avant de plonger dans les détails, clarifions ces deux termes souvent confondus :

- **Authentification** : Processus qui vérifie l'identité d'un utilisateur ("Qui êtes-vous ?")
- **Autorisation** : Processus qui détermine ce qu'un utilisateur authentifié est autorisé à faire ("Que pouvez-vous faire ?")

## 12.7.1. Identity Framework

ASP.NET Core Identity est un framework complet qui facilite la gestion des utilisateurs, des mots de passe, des profils et des rôles dans vos applications ASP.NET Core.

### Installation et configuration de base

Pour intégrer Identity dans votre projet ASP.NET Core :

1. **Installer les packages NuGet nécessaires** :
   ```
   dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
   dotnet add package Microsoft.AspNetCore.Identity.UI
   ```

2. **Configurer Identity dans votre fichier `Program.cs`** :

   ```csharp
   // Programme.cs
   var builder = WebApplication.CreateBuilder(args);

   // Ajout des services Entity Framework
   builder.Services.AddDbContext<ApplicationDbContext>(options =>
       options.UseSqlServer(
           builder.Configuration.GetConnectionString("DefaultConnection")));

   // Configuration d'Identity
   builder.Services.AddDefaultIdentity<IdentityUser>(options =>
   {
       // Configuration des options de mot de passe
       options.Password.RequireDigit = true;
       options.Password.RequireLowercase = true;
       options.Password.RequireUppercase = true;
       options.Password.RequireNonAlphanumeric = true;
       options.Password.RequiredLength = 8;

       // Configuration de la connexion
       options.SignIn.RequireConfirmedAccount = true;
   })
   .AddEntityFrameworkStores<ApplicationDbContext>();

   // Configuration des cookies
   builder.Services.ConfigureApplicationCookie(options =>
   {
       options.Cookie.HttpOnly = true;
       options.ExpireTimeSpan = TimeSpan.FromMinutes(60);
       options.LoginPath = "/Identity/Account/Login";
       options.AccessDeniedPath = "/Identity/Account/AccessDenied";
   });
   ```

3. **Créer une classe `ApplicationDbContext`** qui hérite de `IdentityDbContext` :

   ```csharp
   public class ApplicationDbContext : IdentityDbContext
   {
       public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
           : base(options)
       {
       }
   }
   ```

4. **Ajouter le middleware d'authentification** dans votre pipeline HTTP :

   ```csharp
   var app = builder.Build();

   // Autres middlewares...

   app.UseAuthentication(); // Doit être placé avant UseAuthorization()
   app.UseAuthorization();

   app.MapRazorPages();
   app.MapControllers();

   app.Run();
   ```

### Modèles d'utilisateurs personnalisés

Par défaut, Identity utilise la classe `IdentityUser` qui fournit des propriétés de base. Vous pouvez étendre cette classe pour ajouter vos propres propriétés :

```csharp
public class ApplicationUser : IdentityUser
{
    // Propriétés personnalisées
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime BirthDate { get; set; }
    public string ProfilePictureUrl { get; set; }
}
```

N'oubliez pas de modifier votre configuration pour utiliser cette classe personnalisée :

```csharp
builder.Services.AddDefaultIdentity<ApplicationUser>(options => {
    // Options...
})
.AddEntityFrameworkStores<ApplicationDbContext>();
```

### Gestion des utilisateurs

Identity propose l'interface `UserManager<TUser>` pour gérer les utilisateurs programmatiquement :

```csharp
public class UserService
{
    private readonly UserManager<ApplicationUser> _userManager;

    public UserService(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }

    // Créer un nouvel utilisateur
    public async Task<IdentityResult> CreateUserAsync(string email, string password)
    {
        var user = new ApplicationUser { UserName = email, Email = email };
        return await _userManager.CreateAsync(user, password);
    }

    // Trouver un utilisateur par email
    public async Task<ApplicationUser> FindUserByEmailAsync(string email)
    {
        return await _userManager.FindByEmailAsync(email);
    }

    // Vérifier si un utilisateur est dans un rôle
    public async Task<bool> IsInRoleAsync(ApplicationUser user, string role)
    {
        return await _userManager.IsInRoleAsync(user, role);
    }

    // Ajouter un utilisateur à un rôle
    public async Task<IdentityResult> AddToRoleAsync(ApplicationUser user, string role)
    {
        return await _userManager.AddToRoleAsync(user, role);
    }
}
```

### Utilisation des vues Identity prédéfinies

ASP.NET Core Identity inclut des pages prêtes à l'emploi pour l'inscription, la connexion et la gestion des comptes. Pour les utiliser, ajoutez ces lignes dans `Program.cs` :

```csharp
// Ajout du support pour les pages Razor
builder.Services.AddRazorPages();

// Dans la configuration du pipeline
app.MapRazorPages();
```

Vous pouvez personnaliser ces pages en générant leur code source dans votre projet :

```
dotnet aspnet-codegenerator identity -dc YourApp.Data.ApplicationDbContext
```

## 12.7.2. JWT et tokens

Les JSON Web Tokens (JWT) sont une méthode moderne pour authentifier les utilisateurs, particulièrement utile pour les API RESTful et les applications SPA (Single Page Applications).

### Qu'est-ce qu'un JWT ?

Un JWT est un token compact et autonome qui permet de transmettre de manière sécurisée des informations entre deux parties sous forme d'objet JSON. Ces tokens peuvent être vérifiés et approuvés car ils sont signés numériquement.

Un JWT se compose de trois parties séparées par des points (`.`) :
1. **Header** : Contient le type du token et l'algorithme de signature
2. **Payload** : Contient les revendications (claims) comme l'identité de l'utilisateur
3. **Signature** : Assure l'intégrité du token

### Configuration de JWT dans ASP.NET Core

Pour utiliser les JWT dans votre application ASP.NET Core :

1. **Installer le package NuGet** :
   ```
   dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
   ```

2. **Configurer l'authentification JWT** dans `Program.cs` :

   ```csharp
   // Ajouter la configuration JWT
   builder.Services.AddAuthentication(options =>
   {
       options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
       options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
   })
   .AddJwtBearer(options =>
   {
       options.TokenValidationParameters = new TokenValidationParameters
       {
           ValidateIssuer = true,
           ValidateAudience = true,
           ValidateLifetime = true,
           ValidateIssuerSigningKey = true,
           ValidIssuer = builder.Configuration["Jwt:Issuer"],
           ValidAudience = builder.Configuration["Jwt:Audience"],
           IssuerSigningKey = new SymmetricSecurityKey(
               Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
       };
   });
   ```

3. **Ajouter les paramètres JWT** dans votre fichier `appsettings.json` :

   ```json
   {
     "Jwt": {
       "Key": "votre_clé_secrète_très_longue_et_sécurisée",
       "Issuer": "https://votreapi.com",
       "Audience": "https://votreapp.com",
       "DurationInMinutes": 60
     }
   }
   ```

### Génération de tokens JWT

Voici un exemple de service pour générer des tokens JWT :

```csharp
public class TokenService
{
    private readonly IConfiguration _configuration;

    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateJwtToken(ApplicationUser user, IList<string> roles)
    {
        // Récupérer la clé secrète depuis la configuration
        var key = Encoding.ASCII.GetBytes(_configuration["Jwt:Key"]);

        // Préparer les claims (revendications)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id),
            new Claim(ClaimTypes.Name, user.UserName),
            new Claim(ClaimTypes.Email, user.Email)
        };

        // Ajouter les rôles comme claims
        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        // Créer le token
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(Convert.ToDouble(_configuration["Jwt:DurationInMinutes"])),
            Issuer = _configuration["Jwt:Issuer"],
            Audience = _configuration["Jwt:Audience"],
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature)
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);

        return tokenHandler.WriteToken(token);
    }
}
```

### Contrôleur d'authentification

Voici un exemple de contrôleur qui gère l'authentification des utilisateurs et la génération de tokens JWT :

```csharp
[Route("api/[controller]")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly TokenService _tokenService;

    public AuthController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        TokenService tokenService)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _tokenService = tokenService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginModel model)
    {
        // Vérifier si le modèle est valide
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        // Trouver l'utilisateur par email
        var user = await _userManager.FindByEmailAsync(model.Email);
        if (user == null)
            return Unauthorized(new { message = "Email ou mot de passe incorrect" });

        // Vérifier le mot de passe
        var result = await _signInManager.CheckPasswordSignInAsync(user, model.Password, false);
        if (!result.Succeeded)
            return Unauthorized(new { message = "Email ou mot de passe incorrect" });

        // Récupérer les rôles de l'utilisateur
        var roles = await _userManager.GetRolesAsync(user);

        // Générer le token JWT
        var token = _tokenService.GenerateJwtToken(user, roles);

        // Retourner le token
        return Ok(new
        {
            token = token,
            expiration = DateTime.UtcNow.AddMinutes(Convert.ToDouble(_configuration["Jwt:DurationInMinutes"]))
        });
    }
}

// Modèle pour la connexion
public class LoginModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    public string Password { get; set; }
}
```

### Utilisation des tokens JWT côté client

Dans une application frontend, les tokens JWT sont généralement stockés dans le localStorage ou les cookies, puis envoyés dans l'en-tête Authorization des requêtes HTTP :

```javascript
// Exemple en JavaScript
fetch('https://api.exemple.com/ressource-protegee', {
  headers: {
    'Authorization': 'Bearer ' + localStorage.getItem('token')
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

## 12.7.3. Politiques d'autorisation

Les politiques d'autorisation permettent de définir des règles qui déterminent si un utilisateur a accès à une ressource. ASP.NET Core offre un système d'autorisation basé sur les politiques très flexible.

### Configuration des politiques

Configurez vos politiques d'autorisation dans la méthode `ConfigureServices` de `Program.cs` :

```csharp
builder.Services.AddAuthorization(options =>
{
    // Politique simple basée sur un rôle
    options.AddPolicy("AdminsOnly", policy => policy.RequireRole("Admin"));

    // Politique basée sur une revendication (claim)
    options.AddPolicy("PremiumUsers", policy =>
        policy.RequireClaim("SubscriptionLevel", "Premium"));

    // Politique avec plusieurs exigences
    options.AddPolicy("SeniorManagers", policy =>
        policy.RequireRole("Manager")
              .RequireClaim("EmploymentYears", "5", "6", "7", "8", "9", "10"));

    // Politique avec une exigence personnalisée
    options.AddPolicy("MinimumAge", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
});

// Enregistrer le gestionnaire d'exigences personnalisées
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

### Exigences et gestionnaires personnalisés

Pour les règles d'autorisation complexes, vous pouvez créer des exigences et des gestionnaires personnalisés :

1. **Définir une exigence** :

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }

    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}
```

2. **Créer un gestionnaire d'exigence** :

```csharp
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        // Vérifier si l'utilisateur a une revendication 'DateOfBirth'
        if (!context.User.HasClaim(c => c.Type == "DateOfBirth"))
            return Task.CompletedTask;

        // Récupérer la date de naissance
        var dateOfBirthClaim = context.User.FindFirst(c => c.Type == "DateOfBirth");
        var dateOfBirth = DateTime.Parse(dateOfBirthClaim.Value);

        // Calculer l'âge
        var age = DateTime.Today.Year - dateOfBirth.Year;
        if (dateOfBirth.Date > DateTime.Today.AddYears(-age))
            age--;

        // Vérifier si l'âge répond à l'exigence
        if (age >= requirement.MinimumAge)
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}
```

### Application des politiques

Vous pouvez appliquer des politiques de plusieurs façons :

1. **Sur des contrôleurs ou des actions** :

```csharp
[Authorize(Policy = "AdminsOnly")]
public class AdminController : Controller
{
    // Méthodes du contrôleur...
}

[Authorize(Policy = "PremiumUsers")]
public IActionResult PremiumContent()
{
    return View();
}
```

2. **Sur des endpoints dans le routage** :

```csharp
app.MapGet("/admin/dashboard", () => "Dashboard administrateur")
   .RequireAuthorization("AdminsOnly");

app.MapGet("/premium-content", () => "Contenu premium")
   .RequireAuthorization("PremiumUsers");
```

3. **Vérification manuelle dans le code** :

```csharp
public class ContentController : Controller
{
    private readonly IAuthorizationService _authorizationService;

    public ContentController(IAuthorizationService authorizationService)
    {
        _authorizationService = authorizationService;
    }

    public async Task<IActionResult> ViewDocument(int documentId)
    {
        var document = await _documentService.GetDocumentAsync(documentId);

        // Vérifier l'autorisation
        var authResult = await _authorizationService.AuthorizeAsync(
            User, document, "DocumentOwnerPolicy");

        if (!authResult.Succeeded)
            return Forbid();

        return View(document);
    }
}
```

## 12.7.4. OAuth et OpenID Connect

OAuth 2.0 et OpenID Connect (OIDC) sont des protocoles standard pour l'authentification et l'autorisation, permettant aux utilisateurs de s'authentifier via des fournisseurs d'identité externes comme Google, Facebook, Microsoft, etc.

### Différence entre OAuth 2.0 et OpenID Connect

- **OAuth 2.0** : Protocole d'autorisation qui permet à une application d'accéder à des ressources au nom d'un utilisateur
- **OpenID Connect** : Extension d'OAuth 2.0 qui ajoute une couche d'authentification, permettant de vérifier l'identité de l'utilisateur

### Configuration d'un fournisseur externe

Voici comment configurer l'authentification avec Google :

1. **Installer le package NuGet** :
   ```
   dotnet add package Microsoft.AspNetCore.Authentication.Google
   ```

2. **Configurer le service** dans `Program.cs` :

   ```csharp
   builder.Services.AddAuthentication()
       .AddGoogle(options =>
       {
           options.ClientId = configuration["Authentication:Google:ClientId"];
           options.ClientSecret = configuration["Authentication:Google:ClientSecret"];
           options.CallbackPath = "/signin-google"; // URL de redirection OAuth
       });
   ```

3. **Ajouter les paramètres** dans `appsettings.json` :

   ```json
   {
     "Authentication": {
       "Google": {
         "ClientId": "votre-client-id",
         "ClientSecret": "votre-client-secret"
       }
     }
   }
   ```

De même, vous pouvez configurer d'autres fournisseurs comme Facebook, Microsoft, Twitter, etc.

### Intégration d'Identity Server

Pour les applications plus complexes, Identity Server 4 est une solution open-source qui implémente OpenID Connect et OAuth 2.0 pour ASP.NET Core.

1. **Installer le package** :
   ```
   dotnet add package IdentityServer4
   dotnet add package IdentityServer4.AspNetIdentity
   ```

2. **Configuration de base dans `Program.cs`** :

   ```csharp
   builder.Services.AddIdentityServer()
       .AddApiAuthorization<ApplicationUser, ApplicationDbContext>()
       .AddDeveloperSigningCredential(); // Pour le développement uniquement

   // Ajouter la validation de token Identity Server
   builder.Services.AddAuthentication()
       .AddIdentityServerJwt();
   ```

3. **Configuration du pipeline** :

   ```csharp
   // Ajouter le middleware Identity Server
   app.UseIdentityServer();

   // Autres middlewares...
   app.UseAuthentication();
   app.UseAuthorization();
   ```

### Exemple d'authentification dans un client

Voici un exemple simple d'utilisation d'OpenID Connect dans une application cliente ASP.NET Core MVC :

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = "Cookies";
    options.DefaultChallengeScheme = "oidc";
})
.AddCookie("Cookies")
.AddOpenIdConnect("oidc", options =>
{
    options.Authority = "https://votre-serveur-identite.com";
    options.ClientId = "client_id";
    options.ClientSecret = "client_secret";
    options.ResponseType = "code";
    options.SaveTokens = true;
    options.Scope.Add("openid");
    options.Scope.Add("profile");
    options.Scope.Add("email");
    options.Scope.Add("api1");
    options.GetClaimsFromUserInfoEndpoint = true;
});
```

## 12.7.5. Authentification à deux facteurs

L'authentification à deux facteurs (2FA) ajoute une couche de sécurité supplémentaire en exigeant deux formes d'identification différentes avant d'autoriser l'accès.

### Activation de la 2FA dans ASP.NET Core Identity

1. **Configurer les options** dans `Program.cs` :

   ```csharp
   builder.Services.AddDefaultIdentity<ApplicationUser>(options =>
   {
       // Autres options...
       options.SignIn.RequireConfirmedAccount = true;

       // Options 2FA
       options.Tokens.AuthenticatorTokenProvider = TokenOptions.DefaultAuthenticatorProvider;
       options.Tokens.AuthenticatorIssuer = "VotreApplication";
   });
   ```

2. **Générer une clé secrète** pour l'authentification par application :

   ```csharp
   public class TwoFactorController : Controller
   {
       private readonly UserManager<ApplicationUser> _userManager;

       public TwoFactorController(UserManager<ApplicationUser> userManager)
       {
           _userManager = userManager;
       }

       [HttpGet]
       [Authorize]
       public async Task<IActionResult> EnableAuthenticator()
       {
           var user = await _userManager.GetUserAsync(User);
           if (user == null)
               return NotFound();

           // Générer une clé unique pour l'utilisateur
           var unformattedKey = await _userManager.GetAuthenticatorKeyAsync(user);
           if (string.IsNullOrEmpty(unformattedKey))
           {
               await _userManager.ResetAuthenticatorKeyAsync(user);
               unformattedKey = await _userManager.GetAuthenticatorKeyAsync(user);
           }

           var model = new EnableAuthenticatorViewModel
           {
               SharedKey = FormatKey(unformattedKey),
               AuthenticatorUri = GenerateQrCodeUri(user.Email, unformattedKey)
           };

           return View(model);
       }

       private string FormatKey(string unformattedKey)
       {
           var result = new StringBuilder();
           int currentPosition = 0;

           while (currentPosition + 4 < unformattedKey.Length)
           {
               result.Append(unformattedKey.Substring(currentPosition, 4)).Append(" ");
               currentPosition += 4;
           }

           if (currentPosition < unformattedKey.Length)
               result.Append(unformattedKey.Substring(currentPosition));

           return result.ToString().ToLowerInvariant();
       }

       private string GenerateQrCodeUri(string email, string unformattedKey)
       {
           return string.Format(
               "otpauth://totp/{0}:{1}?secret={2}&issuer={0}&digits=6",
               Uri.EscapeDataString("VotreApplication"),
               Uri.EscapeDataString(email),
               unformattedKey);
       }
   }
   ```

3. **Vérifier le code d'authentification** :

   ```csharp
   [HttpPost]
   [Authorize]
   public async Task<IActionResult> ValidateAuthenticatorCode(string code)
   {
       var user = await _userManager.GetUserAsync(User);
       if (user == null)
           return NotFound();

       // Vérifier le code
       var isValid = await _userManager.VerifyTwoFactorTokenAsync(
           user,
           _userManager.Options.Tokens.AuthenticatorTokenProvider,
           code);

       if (!isValid)
       {
           ModelState.AddModelError("Code", "Le code d'authentification n'est pas valide.");
           return View();
       }

       // Activer la 2FA
       await _userManager.SetTwoFactorEnabledAsync(user, true);

       // Générer des codes de récupération
       var recoveryCodes = await _userManager.GenerateNewTwoFactorRecoveryCodesAsync(user, 10);

       return RedirectToAction(nameof(ShowRecoveryCodes), new { codes = recoveryCodes });
   }
   ```

4. **Afficher les codes de récupération** :

   ```csharp
   [HttpGet]
   [Authorize]
   public IActionResult ShowRecoveryCodes(IEnumerable<string> codes)
   {
       if (codes == null)
           return NotFound();

       var model = new ShowRecoveryCodesViewModel
       {
           RecoveryCodes = codes
       };

       return View(model);
   }
   ```

### Gestion du processus de connexion avec 2FA

Lorsque la 2FA est activée, le processus de connexion comporte une étape supplémentaire :

```csharp
[HttpPost]
public async Task<IActionResult> Login(LoginViewModel model)
{
    if (ModelState.IsValid)
    {
        var result = await _signInManager.PasswordSignInAsync(
            model.Email,
            model.Password,
            model.RememberMe,
            lockoutOnFailure: true);

        if (result.Succeeded)
        {
            return RedirectToLocal(model.ReturnUrl);
        }
        else if (result.RequiresTwoFactor)
        {
            return RedirectToAction(nameof(LoginWith2fa), new { returnUrl = model.ReturnUrl, rememberMe = model.RememberMe });
        }
        else if (result.IsLockedOut)
        {
            return View("Lockout");
        }
        else
        {
            ModelState.AddModelError(string.Empty, "Tentative de connexion non valide.");
            return View(model);
        }
    }

    return View(model);
}

[HttpGet]
public async Task<IActionResult> LoginWith2fa(bool rememberMe, string returnUrl = null)
{
    // S'assurer que l'utilisateur a déjà passé la première étape
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();
    if (user == null)
        return RedirectToAction(nameof(Login));

    var model = new LoginWith2faViewModel
    {
        RememberMe = rememberMe,
        ReturnUrl = returnUrl
    };

    return View(model);
}

[HttpPost]
public async Task<IActionResult> LoginWith2fa(LoginWith2faViewModel model)
{
    if (!ModelState.IsValid)
        return View(model);

    // Récupérer l'utilisateur
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();
    if (user == null)
        throw new InvalidOperationException("Impossible de charger l'utilisateur à 2FA.");

    // Vérifier le code d'authentification
    var authenticatorCode = model.TwoFactorCode.Replace(" ", string.Empty).Replace("-", string.Empty);
    var result = await _signInManager.TwoFactorAuthenticatorSignInAsync(
        authenticatorCode, model.RememberMe, model.RememberMachine);

    if (result.Succeeded)
    {
        return RedirectToLocal(model.ReturnUrl);
    }
    else if (result.IsLockedOut)
    {
        return View("Lockout");
    }
    else
    {
        ModelState.AddModelError(string.Empty, "Code d'authentification non valide.");
        return View(model);
    }
}
```

### Utilisation des SMS comme deuxième facteur

ASP.NET Core Identity permet également d'utiliser les SMS comme méthode de 2FA :

1. **Configurer un fournisseur SMS** (par exemple, Twilio) :

   ```csharp
   public class SmsService : ISmsSender
   {
       private readonly TwilioClient _client;
       private readonly string _fromNumber;

       public SmsService(IConfiguration configuration)
       {
           var accountSid = configuration["Twilio:AccountSid"];
           var authToken = configuration["Twilio:AuthToken"];
           _fromNumber = configuration["Twilio:PhoneNumber"];

           TwilioClient.Init(accountSid, authToken);
       }

       public Task SendSmsAsync(string number, string message)
       {
           return MessageResource.CreateAsync(
               to: new Twilio.Types.PhoneNumber(number),
               from: new Twilio.Types.PhoneNumber(_fromNumber),
               body: message);
       }
   }
   ```

2. **Enregistrer le service** :

   ```csharp
   builder.Services.AddTransient<ISmsSender, SmsService>();
   ```

3. **Configurer le fournisseur de token SMS** :

   ```csharp
   builder.Services.AddDefaultIdentity<ApplicationUser>(options =>
   {
       // Autres options...
       options.Tokens.ChangePhoneNumberTokenProvider = "Phone";
   });
   ```

## Résumé et meilleures pratiques

Pour terminer ce chapitre, voici quelques meilleures pratiques concernant l'authentification et l'autorisation dans ASP.NET Core :

1. **Sécurité des mots de passe** :
   - Utilisez les paramètres de complexité d'Identity
   - Stockez uniquement des hachages de mots de passe (Identity le fait automatiquement)
   - Encouragez l'utilisation de gestionnaires de mots de passe

2. **Configuration des cookies** :
   - Utilisez toujours HTTPS en production
   - Définissez des délais d'expiration raisonnables (ExpireTimeSpan)
   - Activez HttpOnly pour empêcher l'accès JavaScript aux cookies
   - Utilisez SameSite=Strict quand c'est possible

3. **JWT** :
   - Utilisez des clés secrètes fortes et longues
   - Définissez une durée de validité courte pour les tokens
   - Stockez les tokens en toute sécurité côté client
   - Implémentez un mécanisme de révocation si nécessaire

4. **Autorisations** :
   - Créez des politiques granulaires plutôt que de s'appuyer uniquement sur les rôles
   - Vérifiez l'autorisation au plus près de l'accès aux ressources
   - Évitez de coder en dur les vérifications d'autorisation, utilisez des politiques nommées

5. **2FA** :
   - Offrez plusieurs options de 2FA (application, SMS)
   - Fournissez des codes de récupération
   - Éduquez les utilisateurs sur l'importance de la 2FA

6. **Général** :
   - Ne jamais stocker d'informations sensibles en clair
   - Journaliser les événements d'authentification/autorisation importants
   - Utiliser des CAPTCHA pour prévenir les attaques automatisées
   - Mettre en place des délais progressifs en cas d'échecs de connexion répétés
   - Tester régulièrement la sécurité de votre application

En suivant ces recommandations et en implémentant correctement les mécanismes décrits dans ce chapitre, vous pouvez construire un système d'authentification et d'autorisation robuste pour vos applications ASP.NET Core.

## Code complet d'un système d'authentification basique

Voici un exemple complet d'un contrôleur d'authentification basique avec ASP.NET Core Identity et JWT :

```csharp
using System;
using System.Collections.Generic;
using System.IdentityModel.Tokens.Jwt;
using System.Linq;
using System.Security.Claims;
using System.Text;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Tokens;

namespace VotreApplication.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthController : ControllerBase
    {
        private readonly UserManager<ApplicationUser> _userManager;
        private readonly SignInManager<ApplicationUser> _signInManager;
        private readonly IConfiguration _configuration;

        public AuthController(
            UserManager<ApplicationUser> userManager,
            SignInManager<ApplicationUser> signInManager,
            IConfiguration configuration)
        {
            _userManager = userManager;
            _signInManager = signInManager;
            _configuration = configuration;
        }

        [HttpPost("register")]
        public async Task<IActionResult> Register([FromBody] RegisterModel model)
        {
            if (!ModelState.IsValid)
                return BadRequest(ModelState);

            var user = new ApplicationUser
            {
                UserName = model.Email,
                Email = model.Email,
                FirstName = model.FirstName,
                LastName = model.LastName
            };

            var result = await _userManager.CreateAsync(user, model.Password);

            if (!result.Succeeded)
                return BadRequest(result.Errors);

            // Ajouter au rôle par défaut
            await _userManager.AddToRoleAsync(user, "User");

            // Connecter l'utilisateur
            await _signInManager.SignInAsync(user, isPersistent: false);

            // Générer un token de confirmation d'email
            var code = await _userManager.GenerateEmailConfirmationTokenAsync(user);

            // TODO: Envoyer l'email de confirmation

            return Ok(new { message = "Inscription réussie. Veuillez confirmer votre email." });
        }

        [HttpPost("login")]
        public async Task<IActionResult> Login([FromBody] LoginModel model)
        {
            if (!ModelState.IsValid)
                return BadRequest(ModelState);

            // Vérifier si l'utilisateur existe
            var user = await _userManager.FindByEmailAsync(model.Email);
            if (user == null)
                return Unauthorized(new { message = "Email ou mot de passe incorrect" });

            // Vérifier le mot de passe
            var result = await _signInManager.CheckPasswordSignInAsync(
                user, model.Password, lockoutOnFailure: true);

            if (result.Succeeded)
            {
                // Générer le token JWT
                var token = await GenerateJwtToken(user);

                return Ok(new
                {
                    token = token,
                    expiration = DateTime.UtcNow.AddMinutes(
                        Convert.ToDouble(_configuration["Jwt:DurationInMinutes"]))
                });
            }

            if (result.RequiresTwoFactor)
                return StatusCode(403, new { message = "Authentification à deux facteurs requise" });

            if (result.IsLockedOut)
                return StatusCode(423, new { message = "Compte verrouillé. Veuillez réessayer plus tard." });

            return Unauthorized(new { message = "Email ou mot de passe incorrect" });
        }

        [HttpPost("logout")]
        public async Task<IActionResult> Logout()
        {
            await _signInManager.SignOutAsync();
            return Ok(new { message = "Déconnexion réussie" });
        }

        [HttpPost("2fa")]
        public async Task<IActionResult> LoginWith2fa([FromBody] TwoFactorModel model)
        {
            if (!ModelState.IsValid)
                return BadRequest(ModelState);

            // S'assurer que l'utilisateur a passé la première étape
            var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();
            if (user == null)
                return BadRequest(new { message = "Erreur lors de la validation 2FA" });

            var authenticatorCode = model.TwoFactorCode.Replace(" ", string.Empty).Replace("-", string.Empty);

            var result = await _signInManager.TwoFactorAuthenticatorSignInAsync(
                authenticatorCode, model.RememberMe, model.RememberMachine);

            if (result.Succeeded)
            {
                var token = await GenerateJwtToken(user);

                return Ok(new
                {
                    token = token,
                    expiration = DateTime.UtcNow.AddMinutes(
                        Convert.ToDouble(_configuration["Jwt:DurationInMinutes"]))
                });
            }

            if (result.IsLockedOut)
                return StatusCode(423, new { message = "Compte verrouillé. Veuillez réessayer plus tard." });

            return Unauthorized(new { message = "Code d'authentification non valide" });
        }

        [HttpPost("reset-password")]
        public async Task<IActionResult> ForgotPassword([FromBody] ForgotPasswordModel model)
        {
            if (!ModelState.IsValid)
                return BadRequest(ModelState);

            var user = await _userManager.FindByEmailAsync(model.Email);
            if (user == null || !(await _userManager.IsEmailConfirmedAsync(user)))
                return Ok(new { message = "Si l'email existe dans notre système, un lien de réinitialisation a été envoyé." });

            var code = await _userManager.GeneratePasswordResetTokenAsync(user);

            // TODO: Envoyer l'email de réinitialisation

            return Ok(new { message = "Si l'email existe dans notre système, un lien de réinitialisation a été envoyé." });
        }

        private async Task<string> GenerateJwtToken(ApplicationUser user)
        {
            var key = Encoding.ASCII.GetBytes(_configuration["Jwt:Key"]);

            // Obtenir les rôles de l'utilisateur
            var roles = await _userManager.GetRolesAsync(user);

            // Créer les claims
            var claims = new List<Claim>
            {
                new Claim(ClaimTypes.NameIdentifier, user.Id),
                new Claim(ClaimTypes.Name, user.UserName),
                new Claim(ClaimTypes.Email, user.Email)
            };

            // Ajouter les rôles comme claims
            foreach (var role in roles)
            {
                claims.Add(new Claim(ClaimTypes.Role, role));
            }

            // Créer le token
            var tokenDescriptor = new SecurityTokenDescriptor
            {
                Subject = new ClaimsIdentity(claims),
                Expires = DateTime.UtcNow.AddMinutes(
                    Convert.ToDouble(_configuration["Jwt:DurationInMinutes"])),
                SigningCredentials = new SigningCredentials(
                    new SymmetricSecurityKey(key),
                    SecurityAlgorithms.HmacSha256Signature),
                Issuer = _configuration["Jwt:Issuer"],
                Audience = _configuration["Jwt:Audience"]
            };

            var tokenHandler = new JwtSecurityTokenHandler();
            var token = tokenHandler.CreateToken(tokenDescriptor);

            return tokenHandler.WriteToken(token);
        }
    }

    // Modèle d'inscription
    public class RegisterModel
    {
        [Required]
        [EmailAddress]
        public string Email { get; set; }

        [Required]
        [StringLength(100, MinimumLength = 6)]
        public string Password { get; set; }

        [Compare("Password")]
        public string ConfirmPassword { get; set; }

        [Required]
        public string FirstName { get; set; }

        [Required]
        public string LastName { get; set; }
    }

    // Modèle de connexion
    public class LoginModel
    {
        [Required]
        [EmailAddress]
        public string Email { get; set; }

        [Required]
        public string Password { get; set; }

        public bool RememberMe { get; set; }
    }

    // Modèle 2FA
    public class TwoFactorModel
    {
        [Required]
        [StringLength(7, MinimumLength = 6)]
        public string TwoFactorCode { get; set; }

        public bool RememberMe { get; set; }

        public bool RememberMachine { get; set; }
    }

    // Modèle de récupération de mot de passe
    public class ForgotPasswordModel
    {
        [Required]
        [EmailAddress]
        public string Email { get; set; }
    }
}
```

Ce code fournit un contrôleur complet pour gérer :
- L'inscription de nouveaux utilisateurs
- La connexion des utilisateurs (avec support JWT)
- L'authentification à deux facteurs
- La déconnexion
- La réinitialisation de mot de passe

Pour l'utiliser, vous devrez également configurer ASP.NET Core Identity et JWT comme expliqué précédemment dans ce chapitre.

## Pour aller plus loin

Une fois que vous maîtrisez les bases de l'authentification et de l'autorisation, vous pouvez explorer ces sujets avancés :

1. **Implémentation de reCAPTCHA** pour protéger contre les bots
2. **Journalisation et audit** des tentatives d'authentification
3. **Intégration avec un fournisseur d'identité externe** comme Azure AD ou Auth0
4. **Mise en place de JWT avec refresh tokens** pour prolonger la session sans nouvelle authentification
5. **Intégration biométrique** avec WebAuthn pour une authentification sans mot de passe
6. **Single Sign-On (SSO)** entre plusieurs applications

En conclusion, ASP.NET Core offre un écosystème riche et flexible pour implémenter des systèmes d'authentification et d'autorisation sécurisés et adaptés à vos besoins spécifiques. La clé est de comprendre les différentes options disponibles et de choisir celles qui correspondent le mieux aux exigences de votre application.

⏭️
