# 11.6. Sécurité et bonnes pratiques

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La sécurité est un aspect fondamental de toute application manipulant des données. Lorsque vous travaillez avec MySQL/MariaDB dans vos applications .NET, il est essentiel de mettre en place des mesures de protection adaptées. Cette section couvre les principales pratiques de sécurité à adopter pour protéger vos données et votre application.

## 11.6.1. Protection contre les injections SQL

Les injections SQL représentent l'une des vulnérabilités les plus courantes et dangereuses dans les applications web. Elles permettent à un attaquant d'insérer du code SQL malveillant dans vos requêtes.

### Comprendre le risque d'injection SQL

Une injection SQL se produit lorsque des données non fiables (généralement provenant de l'utilisateur) sont directement incorporées dans une requête SQL :

```csharp
// Code vulnérable aux injections SQL
string nomUtilisateur = Request.Form["nom"]; // Potentiellement malveillant
string requete = "SELECT * FROM Utilisateurs WHERE Nom = '" + nomUtilisateur + "'";
var utilisateurs = await context.Utilisateurs.FromSqlRaw(requete).ToListAsync();
```

Si un attaquant entre `' OR '1'='1`, la requête devient :
```sql
SELECT * FROM Utilisateurs WHERE Nom = '' OR '1'='1'
```

Ce qui retourne tous les utilisateurs, contournant ainsi le filtrage souhaité.

### Protection avec les requêtes paramétrées

Entity Framework Core utilise automatiquement des requêtes paramétrées pour toutes les opérations LINQ standard, ce qui constitue une excellente protection contre les injections SQL :

```csharp
// Sécurisé par défaut avec LINQ
string nomUtilisateur = Request.Form["nom"];
var utilisateurs = await context.Utilisateurs
    .Where(u => u.Nom == nomUtilisateur)
    .ToListAsync();
```

### Utilisation sécurisée de FromSqlRaw et FromSqlInterpolated

Lorsque vous devez exécuter du SQL brut, utilisez les paramètres ou l'interpolation de chaînes (qui génère automatiquement des paramètres) :

```csharp
// Méthode 1 : FromSqlRaw avec paramètres
var nomUtilisateur = Request.Form["nom"];
var utilisateurs = await context.Utilisateurs
    .FromSqlRaw("SELECT * FROM Utilisateurs WHERE Nom = {0}", nomUtilisateur)
    .ToListAsync();

// Méthode 2 : FromSqlInterpolated (plus lisible)
var utilisateurs = await context.Utilisateurs
    .FromSqlInterpolated($"SELECT * FROM Utilisateurs WHERE Nom = {nomUtilisateur}")
    .ToListAsync();
```

Les deux méthodes génèrent des requêtes paramétrées sécurisées.

### Validation et nettoyage des entrées

En plus des requêtes paramétrées, validez et nettoyez toujours les entrées utilisateur :

```csharp
// Validation côté serveur avec les attributs DataAnnotations
public class RegisterViewModel
{
    [Required]
    [StringLength(50, MinimumLength = 3)]
    [RegularExpression(@"^[a-zA-Z0-9_-]+$")]
    public string Nom { get; set; }

    // Autres propriétés...
}

// Validation dans le contrôleur
[HttpPost]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    if (ModelState.IsValid)
    {
        // Traiter les données validées
    }
    // Sinon, retourner à la vue avec des erreurs
}
```

### Principes de défense en profondeur

Ne vous fiez pas à une seule mesure de sécurité :

1. **Utilisez des requêtes paramétrées** (EF Core le fait automatiquement)
2. **Validez les entrées** côté client et serveur
3. **Appliquez le principe du moindre privilège** pour les utilisateurs de la base de données
4. **Échappez correctement les caractères spéciaux** si nécessaire
5. **Utilisez des procédures stockées** pour les opérations complexes

## 11.6.2. Gestion sécurisée des connexions

La sécurisation des connexions à la base de données est essentielle pour protéger vos données en transit et contrôler l'accès.

### Sécurisation des chaînes de connexion

Les chaînes de connexion contiennent des informations sensibles qui doivent être protégées :

1. **Ne stockez jamais les chaînes de connexion en dur dans le code** :

```csharp
// À ÉVITER
var connectionString = "Server=myserver;User=admin;Password=MonMotDePasse123!;Database=prod_db";
```

2. **Utilisez des fichiers de configuration sécurisés** :

Pour ASP.NET Core, stockez la chaîne dans `appsettings.json` :

```json
{
  "ConnectionStrings": {
    "Default": "Server=myserver;User=app_user;Password=***;Database=ma_db;SslMode=Required;"
  }
}
```

3. **Utilisez les secrets utilisateur en développement** :

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "Server=myserver;User=app_user;Password=***;Database=ma_db;"
```

4. **Utilisez des variables d'environnement ou des coffres-forts de secrets en production** :

```csharp
// Dans Program.cs
builder.Configuration.AddEnvironmentVariables();
// Ou Azure Key Vault
builder.Configuration.AddAzureKeyVault(/* configuration */);
```

### Utilisation de SSL/TLS pour les connexions

Activez toujours SSL/TLS pour chiffrer les connexions à la base de données :

```csharp
var connectionString = "Server=myserver;User=app_user;Password=***;Database=ma_db;" +
                       "SslMode=Required;"; // Force l'utilisation de SSL
```

Options de SslMode pour MySQL :
- `Required` : Exige SSL mais ne vérifie pas l'identité du serveur
- `VerifyCA` : Vérifie le certificat de l'autorité de certification
- `VerifyFull` : Vérifie l'identité du serveur et le certificat

### Création et gestion des utilisateurs MySQL

Suivez le principe du moindre privilège lors de la création d'utilisateurs de base de données :

```sql
-- Création d'un utilisateur avec accès limité
CREATE USER 'app_user'@'%' IDENTIFIED BY 'mot_de_passe_complexe';

-- Accorder uniquement les privilèges nécessaires
GRANT SELECT, INSERT, UPDATE, DELETE ON ma_boutique.* TO 'app_user'@'%';

-- Refuser explicitement les privilèges dangereux
REVOKE DROP, ALTER, CREATE, REFERENCES ON ma_boutique.* FROM 'app_user'@'%';

-- Appliquer les modifications
FLUSH PRIVILEGES;
```

Conseils pour la gestion des utilisateurs :
1. Créez des utilisateurs différents pour chaque application
2. Limitez l'accès aux tables nécessaires uniquement
3. N'utilisez jamais l'utilisateur 'root' dans les applications
4. Modifiez régulièrement les mots de passe

### Rotation des informations d'identification

Mettez en place une politique de rotation régulière des mots de passe :

1. **Création d'un nouveau compte utilisateur** avec les mêmes privilèges
2. **Mise à jour de l'application** pour utiliser le nouveau compte
3. **Révocation des privilèges** de l'ancien compte après une période de transition
4. **Suppression de l'ancien compte** une fois que tout fonctionne correctement

## 11.6.3. Encryption des données

L'encryption des données sensibles est cruciale pour protéger les informations confidentielles, même en cas de violation de la base de données.

### Encryption au niveau de la base de données

MySQL/MariaDB propose plusieurs fonctionnalités d'encryption :

1. **Encryption de connexion** (SSL/TLS) :

```sql
-- Vérifier l'état du chiffrement
SHOW STATUS LIKE 'Ssl%';

-- Configurer le serveur pour utiliser SSL (dans my.cnf)
-- ssl-ca=ca.pem
-- ssl-cert=server-cert.pem
-- ssl-key=server-key.pem
```

2. **Encryption au repos** (tablespaces et fichiers de données) :

```sql
-- Activer l'encryption pour une table
CREATE TABLE DonneesConfidentielles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    numero_carte VARCHAR(255),
    date_expiration VARCHAR(10)
) ENCRYPTION='Y';
```

### Encryption au niveau de l'application

Pour une protection supplémentaire, chiffrez les données sensibles au niveau de l'application avant de les stocker :

```csharp
// Service de chiffrement
public class EncryptionService
{
    private readonly byte[] _key;
    private readonly byte[] _iv;

    public EncryptionService(IConfiguration configuration)
    {
        // Dans la pratique, stockez ces valeurs de manière sécurisée (ex: Azure Key Vault)
        _key = Convert.FromBase64String(configuration["Encryption:Key"]);
        _iv = Convert.FromBase64String(configuration["Encryption:IV"]);
    }

    public string Encrypt(string plainText)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = _key;
            aes.IV = _iv;

            var encryptor = aes.CreateEncryptor(aes.Key, aes.IV);

            using (var ms = new MemoryStream())
            {
                using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
                using (var sw = new StreamWriter(cs))
                {
                    sw.Write(plainText);
                }

                return Convert.ToBase64String(ms.ToArray());
            }
        }
    }

    public string Decrypt(string cipherText)
    {
        // Code de déchiffrement similaire
    }
}
```

Utilisation avec EF Core :

```csharp
// Dans votre modèle
public class InformationsPaiement
{
    public int Id { get; set; }

    // Stocké chiffré dans la base
    public string NumeroCarte { get; set; }

    // Valeur décryptée, non mappée à la base
    [NotMapped]
    public string NumeroCarteEnClair
    {
        get => _encryptionService.Decrypt(NumeroCarte);
        set => NumeroCarte = _encryptionService.Encrypt(value);
    }

    // Injection du service d'encryption
    private readonly EncryptionService _encryptionService;

    public InformationsPaiement(EncryptionService encryptionService)
    {
        _encryptionService = encryptionService;
    }
}
```

### Hachage des mots de passe

Ne stockez jamais les mots de passe en clair ou simplement chiffrés. Utilisez toujours un algorithme de hachage sécurisé avec un sel :

```csharp
// Utilisation de l'API Identity pour le hachage (recommandé)
public class GestionUtilisateurs
{
    private readonly UserManager<ApplicationUser> _userManager;

    public GestionUtilisateurs(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }

    public async Task<bool> VerifierMotDePasse(ApplicationUser user, string motDePasse)
    {
        return await _userManager.CheckPasswordAsync(user, motDePasse);
    }
}

// Si vous n'utilisez pas Identity, utilisez BCrypt.Net-Next
// Install-Package BCrypt.Net-Next
public class ServiceMotDePasse
{
    public string HasherMotDePasse(string motDePasse)
    {
        return BCrypt.Net.BCrypt.HashPassword(motDePasse);
    }

    public bool VerifierMotDePasse(string motDePasse, string hash)
    {
        return BCrypt.Net.BCrypt.Verify(motDePasse, hash);
    }
}
```

### Protection des données personnelles (RGPD/GDPR)

Pour la conformité aux réglementations comme le RGPD :

1. **Identifiez clairement les données personnelles** dans votre modèle
2. **Implémentez des mécanismes de suppression et d'exportation** des données
3. **Limitez l'accès aux données personnelles** selon le principe du besoin d'en connaître
4. **Documentez votre traitement des données** et vos mesures de sécurité

## 11.6.4. Audit et journalisation

La journalisation et l'audit sont essentiels pour surveiller l'activité de la base de données, détecter les tentatives d'intrusion et respecter les exigences réglementaires.

### Journalisation dans MySQL/MariaDB

MySQL/MariaDB offre plusieurs options de journalisation :

1. **General Query Log** : Journalise toutes les requêtes (utile en développement, trop volumineux pour la production)

```sql
-- Activation du journal général
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/mysql-general.log';
```

2. **Slow Query Log** : Journalise les requêtes lentes (utile pour l'optimisation)

```sql
-- Activation du journal des requêtes lentes
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';
SET GLOBAL long_query_time = 1; -- En secondes
```

3. **Error Log** : Journalise les erreurs et événements de démarrage/arrêt

```sql
-- Consulter la configuration du journal d'erreurs
SHOW VARIABLES LIKE 'log_error';
```

### Mise en place d'une table d'audit

Créez une table d'audit pour suivre les modifications importantes :

```sql
CREATE TABLE JournalAudit (
    Id INT AUTO_INCREMENT PRIMARY KEY,
    DateAction DATETIME NOT NULL,
    Utilisateur VARCHAR(100) NOT NULL,
    TypeAction ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    TableCible VARCHAR(100) NOT NULL,
    IdEnregistrement INT NOT NULL,
    AnciennesValeurs JSON,
    NouvellesValeurs JSON
);
```

Implémentation avec EF Core :

```csharp
// Service d'audit
public class AuditService
{
    private readonly string _utilisateurCourant;

    public AuditService(IHttpContextAccessor httpContextAccessor)
    {
        _utilisateurCourant = httpContextAccessor.HttpContext?.User?.Identity?.Name ?? "Système";
    }

    public void AjouterEntreeAudit(DbContext context, EntityEntry entree)
    {
        var typeAction = entree.State.ToString();
        var tableCible = context.Model.FindEntityType(entree.Entity.GetType()).GetTableName();
        var idEnregistrement = entree.Property("Id").CurrentValue;

        var anciennesValeurs = new Dictionary<string, object>();
        var nouvellesValeurs = new Dictionary<string, object>();

        foreach (var propriete in entree.Properties)
        {
            if (entree.State == EntityState.Modified || entree.State == EntityState.Deleted)
            {
                anciennesValeurs[propriete.Metadata.Name] = propriete.OriginalValue;
            }

            if (entree.State == EntityState.Modified || entree.State == EntityState.Added)
            {
                nouvellesValeurs[propriete.Metadata.Name] = propriete.CurrentValue;
            }
        }

        context.Set<JournalAudit>().Add(new JournalAudit
        {
            DateAction = DateTime.UtcNow,
            Utilisateur = _utilisateurCourant,
            TypeAction = typeAction,
            TableCible = tableCible,
            IdEnregistrement = (int)idEnregistrement,
            AnciennesValeurs = JsonSerializer.Serialize(anciennesValeurs),
            NouvellesValeurs = JsonSerializer.Serialize(nouvellesValeurs)
        });
    }
}

// Intégration à EF Core via SaveChanges
public class AuditableDbContext : DbContext
{
    private readonly AuditService _auditService;

    public AuditableDbContext(DbContextOptions options, AuditService auditService)
        : base(options)
    {
        _auditService = auditService;
    }

    public override int SaveChanges()
    {
        var entitesModifiees = ChangeTracker.Entries()
            .Where(e => e.State == EntityState.Added ||
                        e.State == EntityState.Modified ||
                        e.State == EntityState.Deleted);

        foreach (var entree in entitesModifiees)
        {
            _auditService.AjouterEntreeAudit(this, entree);
        }

        return base.SaveChanges();
    }

    // Version asynchrone similaire pour SaveChangesAsync
}
```

### Journalisation au niveau de l'application

Utilisez un framework de journalisation comme Serilog pour enregistrer les activités importantes :

```csharp
// Installation des packages
// Install-Package Serilog.AspNetCore
// Install-Package Serilog.Sinks.File
// Install-Package Serilog.Sinks.Console

// Configuration dans Program.cs
builder.Host.UseSerilog((context, configuration) =>
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .Enrich.FromLogContext()
        .WriteTo.Console()
        .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day));

// Utilisation dans les services
public class UtilisateurService
{
    private readonly ILogger<UtilisateurService> _logger;

    public UtilisateurService(ILogger<UtilisateurService> logger)
    {
        _logger = logger;
    }

    public async Task<bool> Authentifier(string email, string motDePasse)
    {
        _logger.LogInformation("Tentative d'authentification pour {Email}", email);

        // Logique d'authentification

        if (resultat)
        {
            _logger.LogInformation("Authentification réussie pour {Email}", email);
        }
        else
        {
            _logger.LogWarning("Échec d'authentification pour {Email}", email);
        }

        return resultat;
    }
}
```

### Détection des activités suspectes

Mettez en place des mécanismes pour détecter les activités anormales :

1. **Alertes sur les échecs d'authentification répétés**
2. **Surveillance des requêtes inhabituelles ou excessives**
3. **Vérification des connexions à des heures inhabituelles**
4. **Détection des accès à des données sensibles**

## 11.6.5. Sauvegarde et restauration

La sauvegarde régulière de vos données est la dernière ligne de défense contre les pertes de données, qu'elles soient accidentelles ou malveillantes.

### Types de sauvegardes MySQL/MariaDB

MySQL/MariaDB propose plusieurs méthodes de sauvegarde :

1. **Logical backups** avec mysqldump (fichiers SQL)
2. **Physical backups** (copie des fichiers de données)
3. **Incremental backups** (sauvegardes des modifications uniquement)

### Sauvegarde avec mysqldump

L'outil mysqldump génère un script SQL qui peut recréer votre base de données :

```bash
# Sauvegarde d'une base de données complète
mysqldump -u root -p --databases ma_boutique > ma_boutique_backup.sql

# Sauvegarde avec compression
mysqldump -u root -p ma_boutique | gzip > ma_boutique_backup.sql.gz

# Sauvegarde d'une table spécifique
mysqldump -u root -p ma_boutique table1 table2 > tables_backup.sql

# Sauvegarde de la structure uniquement (sans les données)
mysqldump -u root -p --no-data ma_boutique > schema_backup.sql
```

### Automatisation des sauvegardes

Créez un script de sauvegarde automatique :

```bash
#!/bin/bash
# backup_mysql.sh

# Configuration
DB_USER="root"
DB_PASSWORD="votre_mot_de_passe"
DB_NAME="ma_boutique"
BACKUP_DIR="/chemin/vers/backups"
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/$DB_NAME-$DATE.sql.gz"

# Création du répertoire de sauvegarde s'il n'existe pas
mkdir -p $BACKUP_DIR

# Sauvegarde avec compression
mysqldump -u $DB_USER -p$DB_PASSWORD --databases $DB_NAME | gzip > $BACKUP_FILE

# Conservation des 7 dernières sauvegardes uniquement
ls -tp $BACKUP_DIR/*.sql.gz | grep -v '/$' | tail -n +8 | xargs -I {} rm -- {}

echo "Sauvegarde terminée: $BACKUP_FILE"
```

Ajoutez ce script à un job cron pour l'exécution automatique :

```bash
# Exécution quotidienne à 2h du matin
0 2 * * * /chemin/vers/backup_mysql.sh
```

### Restauration d'une sauvegarde

Pour restaurer une base de données à partir d'une sauvegarde :

```bash
# Restauration à partir d'un fichier SQL
mysql -u root -p ma_boutique < ma_boutique_backup.sql

# Restauration à partir d'un fichier compressé
gunzip < ma_boutique_backup.sql.gz | mysql -u root -p ma_boutique
```

### Sauvegarde via .NET

Vous pouvez également déclencher des sauvegardes depuis votre application .NET :

```csharp
public class ServiceSauvegarde
{
    private readonly string _cheminBackups;
    private readonly string _chaineConnexion;

    public ServiceSauvegarde(IConfiguration configuration)
    {
        _cheminBackups = configuration["Backup:Path"];
        _chaineConnexion = configuration.GetConnectionString("Default");
    }

    public async Task<string> CreerSauvegarde()
    {
        var connectionStringBuilder = new MySqlConnectionStringBuilder(_chaineConnexion);
        var database = connectionStringBuilder.Database;
        var nomFichier = $"{database}_{DateTime.Now:yyyyMMdd_HHmmss}.sql";
        var cheminComplet = Path.Combine(_cheminBackups, nomFichier);

        using (var process = new Process())
        {
            process.StartInfo.FileName = "mysqldump";
            process.StartInfo.Arguments = $"-h {connectionStringBuilder.Server} " +
                                         $"-u {connectionStringBuilder.UserID} " +
                                         $"-p{connectionStringBuilder.Password} " +
                                         $"--databases {database} " +
                                         $"-r \"{cheminComplet}\"";
            process.StartInfo.RedirectStandardOutput = true;
            process.StartInfo.UseShellExecute = false;
            process.StartInfo.CreateNoWindow = true;

            process.Start();
            await process.WaitForExitAsync();

            if (process.ExitCode != 0)
            {
                throw new Exception($"Erreur lors de la sauvegarde: code {process.ExitCode}");
            }
        }

        return cheminComplet;
    }
}
```

### Plans de reprise d'activité (PRA/DRP)

Élaborez un plan de reprise d'activité qui définit :

1. **La fréquence des sauvegardes** selon la criticité des données
2. **L'emplacement de stockage** (local, cloud, géographiquement distribué)
3. **La procédure de restauration** et les personnes responsables
4. **Les tests réguliers** de restauration pour valider le processus
5. **Le temps de reprise** maximum acceptable (RTO - Recovery Time Objective)
6. **La perte de données** maximum acceptable (RPO - Recovery Point Objective)

Pour les applications critiques, envisagez :
- La réplication en temps réel
- Les sauvegardes incrémentielles fréquentes
- Le stockage hors site des sauvegardes
- Les procédures de restauration automatisées

## Bonnes pratiques générales de sécurité

1. **Appliquez les mises à jour de sécurité** régulièrement (MySQL/MariaDB et .NET)
2. **Suivez le principe du moindre privilège** pour les utilisateurs et les applications
3. **Chiffrez les données sensibles** au repos et en transit
4. **Mettez en place une journalisation complète** et surveillez les activités suspectes
5. **Effectuez des sauvegardes régulières** et testez les procédures de restauration
6. **Réalisez des audits de sécurité** périodiques de votre application et de votre base de données
7. **Formez votre équipe** aux bonnes pratiques de sécurité
8. **Documentez vos politiques de sécurité** et assurez-vous qu'elles sont suivies

En suivant ces recommandations, vous pourrez créer des applications MySQL/MariaDB sécurisées et robustes qui protègent efficacement vos données et respectent les exigences réglementaires.

⏭️
