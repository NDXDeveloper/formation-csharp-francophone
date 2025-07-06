# 16. Sécurité en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

![Sécurité en C#](https://via.placeholder.com/800x200?text=S%C3%A9curit%C3%A9+en+C%23)

## La sécurité : votre responsabilité numéro 1

Dans un monde où **une faille de sécurité peut détruire une entreprise en quelques heures**, maîtriser la sécurité applicative n'est plus une option - c'est une **compétence de survie**. Ce chapitre vous révèle comment créer des applications .NET blindées contre les menaces modernes.

> **⚡ Réalité du terrain :**
> 43% des cyberattaques ciblent les applications web. 95% des failles exploitées sont connues depuis plus d'un an. La différence ? Les développeurs qui appliquent les bonnes pratiques de sécurité.

---

## 🚨 L'urgence sécuritaire en chiffres

### **💰 Impact économique des failles**
```
Coût moyen d'une faille de données :
├── Global : 4.45 millions $
├── Temps de détection : 277 jours
├── Temps de confinement : 70 jours
└── Impact réputationnel : -7.5% de valorisation
```

### **🎯 Vulnérabilités les plus exploitées**
1. **Injection SQL** - 32% des attaques web
2. **Authentification cassée** - 28% des failles
3. **Exposition de données** - 25% des incidents
4. **Configuration défaillante** - 19% des vulnérabilités

---

## 🛡️ L'arsenal défensif .NET

### **🔐 Protection des données sensibles**

**Secrets Management** - Plus jamais en dur
- **Azure Key Vault** : Coffre-fort cloud enterprise
- **User Secrets** : Développement local sécurisé
- **Environment Variables** : Configuration flexible
- **DPAPI** : Chiffrement Windows natif

**Cryptographie moderne** - Standards industriels
- **BCrypt/Argon2** : Hachage passwords bulletproof
- **AES-256** : Chiffrement symétrique de référence
- **RSA/ECC** : Cryptographie asymétrique moderne
- **HMAC** : Intégrité et authenticité garanties

### **🚪 Contrôle d'accès multicouche**

**Authentification robuste**
- **JWT** : Tokens stateless et performants
- **OAuth 2.0/OpenID Connect** : Standards industriels
- **Multi-Factor Auth** : Sécurité renforcée
- **Certificate Authentication** : Maximum sécurisé

**Autorisation granulaire**
- **Role-Based Access Control** : Permissions par rôles
- **Attribute-Based Access Control** : Contrôle contextuel
- **Claims-Based Authorization** : Flexibilité maximale
- **Resource-Based Authorization** : Sécurité par ressource

---

## ⚔️ Combat contre les vulnérabilités classiques

### **🗡️ Injection SQL : L'ennemi juré**

**❌ Code vulnérable (JAMAIS faire ça)**
```csharp
// DANGER : Injection SQL garantie
string sql = $"SELECT * FROM Users WHERE Username = '{username}'";
```

**✅ Code sécurisé (TOUJOURS faire ça)**
```csharp
// Paramètres : Bouclier anti-injection
using var command = new SqlCommand(
    "SELECT * FROM Users WHERE Username = @username", connection);
command.Parameters.AddWithValue("@username", username);

// Entity Framework : Protection native
var user = context.Users
    .Where(u => u.Username == username)
    .FirstOrDefault();
```

### **🔒 Authentification cassée : Le talon d'Achille**

**Configuration ASP.NET Core sécurisée**
```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ClockSkew = TimeSpan.Zero // Pas de tolérance temporelle
        };
    });

// Politique de mots de passe stricte
services.Configure<IdentityOptions>(options =>
{
    options.Password.RequiredLength = 12;
    options.Password.RequireUppercase = true;
    options.Password.RequireNonAlphanumeric = true;
    options.Lockout.MaxFailedAccessAttempts = 3;
    options.Lockout.LockoutTimeSpan = TimeSpan.FromMinutes(15);
});
```

---

## 🌐 Sécurité Web : Blindage multicouche

### **🔐 HTTPS partout et toujours**
```csharp
// Configuration HTTPS stricte
app.UseHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubdomains = true;
    options.Preload = true;
});

app.UseHttpsRedirection();

// Headers de sécurité modernes
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");

    await next();
});
```

### **⚡ Protection CSRF avancée**
```csharp
// Anti-CSRF automatique
services.AddAntiforgery(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.Cookie.HttpOnly = true;
});

// Dans vos formulaires
@Html.AntiForgeryToken()

// Validation automatique
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult UpdateProfile(UserModel model) { }
```

---

## 🔑 Gestion des secrets : Niveau enterprise

### **Azure Key Vault : Coffre-fort ultime**
```csharp
// Configuration seamless
public void ConfigureServices(IServiceCollection services)
{
    var keyVaultUrl = configuration["KeyVault:Url"];
    var credential = new DefaultAzureCredential();

    configuration.AddAzureKeyVault(keyVaultUrl, credential);

    // Usage transparent
    var connectionString = configuration["Database:ConnectionString"];
    var apiKey = configuration["ExternalApi:Key"];
}
```

### **User Secrets : Développement sécurisé**
```bash
# Configuration locale sécurisée
dotnet user-secrets init
dotnet user-secrets set "Database:ConnectionString" "Server=localhost;..."
dotnet user-secrets set "Api:Key" "your-secret-api-key"
```

```csharp
// Accès automatique en développement
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Les secrets sont automatiquement chargés en dev
        var dbConnection = Configuration["Database:ConnectionString"];
    }
}
```

---

## 🧪 Tests de sécurité automatisés

### **Validation d'entrée systématique**
```csharp
[Test]
public void DevraitRejeterLesInjectionsSql()
{
    var inputs = new[]
    {
        "'; DROP TABLE Users; --",
        "admin'/*",
        "' OR '1'='1",
        "<script>alert('xss')</script>"
    };

    foreach (var maliciousInput in inputs)
    {
        var result = userService.ValidateUsername(maliciousInput);
        Assert.False(result.IsValid);
        Assert.Contains("caractères non autorisés", result.ErrorMessage);
    }
}
```

### **Tests d'autorisation**
```csharp
[Test]
public async Task DevraitInterdireAccesNonAutorise()
{
    // Arrange
    var client = factory.CreateClient();

    // Act - Tentative d'accès sans authentification
    var response = await client.GetAsync("/api/admin/users");

    // Assert
    Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
}
```

---

## 🎯 Roadmap sécuritaire par contexte

### **🏢 Applications d'entreprise**
```yaml
Priorités:
  - Identity Server intégré
  - Authentification AD/LDAP
  - Audit et logs centralisés
  - Politiques de groupe
  - Certification SOC2/ISO27001
```

### **🌐 APIs publiques**
```yaml
Priorités:
  - Rate limiting strict
  - API Gateway avec WAF
  - JWT avec rotation automatique
  - Monitoring des anomalies
  - Tests de pénétration réguliers
```

### **📱 Applications client-serveur**
```yaml
Priorités:
  - Certificate pinning
  - Chiffrement local des données
  - Communication mTLS
  - Protection anti-reverse engineering
  - Updates de sécurité automatiques
```

---

## 🚨 Top 10 OWASP : Protection intégrée

| Vulnérabilité | Protection .NET | Implémentation |
|---------------|-----------------|----------------|
| **A01: Broken Access Control** | Authorization Policies | `[Authorize(Policy = "AdminOnly")]` |
| **A02: Cryptographic Failures** | .NET Cryptography APIs | `Rfc2898DeriveBytes`, `AesCng` |
| **A03: Injection** | Parameterized Queries | Entity Framework, Dapper |
| **A04: Insecure Design** | Secure by Design | Threat Modeling intégré |
| **A05: Security Misconfiguration** | Configuration Validation | `IOptionsSnapshot<T>` |
| **A06: Vulnerable Components** | NuGet Audit | `dotnet list package --vulnerable` |
| **A07: Authentication Failures** | ASP.NET Identity | Multi-factor, Account lockout |
| **A08: Data Integrity Failures** | Digital Signatures | `RSACryptoServiceProvider` |
| **A09: Logging Failures** | Structured Logging | Serilog, Application Insights |
| **A10: SSRF** | HTTP Client Validation | Whitelist des domaines autorisés |

---

## 🔍 Monitoring et détection d'intrusion

### **Logging sécurisé intelligent**
```csharp
public class SecurityEventLogger
{
    private readonly ILogger<SecurityEventLogger> _logger;

    public void LogFailedAuthentication(string username, string ipAddress)
    {
        _logger.LogWarning("Failed authentication attempt for {Username} from {IpAddress}",
            username, ipAddress);

        // Déclencher alertes si tentatives répétées
        if (IsRepeatFailure(username, ipAddress))
        {
            _logger.LogCritical("Multiple failed attempts detected - potential brute force attack");
        }
    }
}
```

### **Métriques de sécurité**
```csharp
// Surveillance en temps réel
services.AddApplicationInsightsTelemetry();

// Métriques custom de sécurité
public class SecurityMetrics
{
    private readonly IMetrics _metrics;

    public void RecordFailedLogin() =>
        _metrics.CreateCounter<int>("security.failed_logins").Add(1);

    public void RecordSuspiciousActivity() =>
        _metrics.CreateCounter<int>("security.suspicious_activity").Add(1);
}
```

---

## 🚀 Quick Start sécurisé

### **✅ Checklist de démarrage**
- [ ] **HTTPS activé** partout (dev inclus)
- [ ] **Secrets externalisés** (jamais dans le code)
- [ ] **Input validation** systématique
- [ ] **Authorization** configurée par défaut
- [ ] **Logging sécurisé** en place
- [ ] **Headers sécurisés** configurés
- [ ] **Dépendances auditées** régulièrement

### **🛡️ Configuration minimale sécurisée**
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Sécurité by default
    services.AddHsts(options => options.MaxAge = TimeSpan.FromDays(365));
    services.AddAntiforgery();
    services.AddDataProtection();

    // Authentication obligatoire par défaut
    services.AddAuthentication().AddJwtBearer();
    services.AddAuthorization(options =>
    {
        options.FallbackPolicy = new AuthorizationPolicyBuilder()
            .RequireAuthenticatedUser()
            .Build();
    });
}
```

---

## 🎖️ La mentalité sécuritaire

### **Principes fondamentaux**
- **Defense in Depth** : Plusieurs couches de protection
- **Principle of Least Privilege** : Accès minimal nécessaire
- **Fail Secure** : Échec sécurisé par défaut
- **Security by Design** : Sécurité dès la conception

### **Culture d'équipe**
- **Threat Modeling** : Modéliser les menaces régulièrement
- **Security Reviews** : Révision sécuritaire du code
- **Red Team Exercises** : Tests d'intrusion internes
- **Continuous Learning** : Veille sécuritaire permanente

---

## 🎯 L'objectif : Sécurité transparente

**Votre application atteint l'excellence sécuritaire quand :**
- **Les développeurs** appliquent naturellement les bonnes pratiques
- **Les utilisateurs** ne subissent aucune friction sécuritaire
- **Les attaques** sont détectées et bloquées automatiquement
- **L'audit** révèle zéro vulnérabilité critique
- **La confiance** est totale de toutes les parties prenantes

---

## 🚀 Votre mission sécuritaire

La sécurité n'est pas un feature à ajouter en fin de projet - c'est **l'ADN de toute application moderne**. Les techniques que vous allez maîtriser ne vous rendront pas seulement meilleur développeur, elles feront de vous un **gardien de la confiance numérique**.

**Prêt à devenir un développeur que les hackers redoutent ?**

⏭️ Commençons par les fondations avec **16.1. [Gestion des chaînes de connexion](/16-securite-en-csharp/16-1-gestion-des-chaines-de-connexion.md)** - l'art de protéger vos secrets les plus précieux.
