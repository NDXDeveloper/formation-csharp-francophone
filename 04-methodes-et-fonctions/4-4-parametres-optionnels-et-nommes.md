# 4.4. Paramètres optionnels et nommés en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les paramètres optionnels et nommés sont des fonctionnalités de C# qui rendent votre code plus flexible et lisible. Ils vous permettent de simplifier l'appel des méthodes en rendant certains paramètres facultatifs et en permettant de spécifier les paramètres par leur nom. Dans cette section, nous allons explorer en détail ces deux concepts.

## Introduction aux paramètres optionnels et nommés

Imaginez que vous ayez une méthode avec de nombreux paramètres, mais que la plupart du temps, vous n'utilisiez que quelques-uns d'entre eux. Sans paramètres optionnels, vous devriez créer plusieurs surcharges de méthodes ou passer des valeurs "nullables" pour chaque paramètre. Les paramètres optionnels résolvent ce problème en permettant d'assigner des valeurs par défaut qui seront utilisées si l'appelant ne fournit pas ces paramètres.

Les paramètres nommés complètent cette fonctionnalité en permettant de spécifier explicitement quel paramètre vous définissez, rendant le code plus lisible et permettant de fournir les paramètres dans n'importe quel ordre.

## 4.4.1. Valeurs par défaut

Les paramètres optionnels sont des paramètres de méthode qui ont une valeur par défaut. Lors de l'appel de la méthode, si vous ne fournissez pas de valeur pour ces paramètres, la valeur par défaut sera utilisée.

### Syntaxe des paramètres optionnels

Pour définir un paramètre optionnel, vous spécifiez une valeur par défaut dans la déclaration de la méthode en utilisant l'opérateur `=` :

```csharp
public void AfficherMessage(string message, bool enMajuscules = false, ConsoleColor couleur = ConsoleColor.White)
{
    if (enMajuscules)
    {
        message = message.ToUpper();
    }

    Console.ForegroundColor = couleur;
    Console.WriteLine(message);
    Console.ResetColor();
}
```

Dans cet exemple, `enMajuscules` et `couleur` sont des paramètres optionnels avec des valeurs par défaut respectivement `false` et `ConsoleColor.White`. Le paramètre `message` reste obligatoire.

### Appel de méthodes avec paramètres optionnels

Lorsque vous appelez une méthode avec des paramètres optionnels, vous pouvez choisir d'omettre ces paramètres :

```csharp
// Appel avec seulement le paramètre obligatoire
AfficherMessage("Bonjour le monde");

// Appel en spécifiant le deuxième paramètre (enMajuscules)
AfficherMessage("Bonjour le monde", true);

// Appel en spécifiant tous les paramètres
AfficherMessage("Bonjour le monde", true, ConsoleColor.Green);
```

### Règles importantes pour les paramètres optionnels

1. **Position des paramètres** : Les paramètres optionnels doivent apparaître après tous les paramètres obligatoires dans la signature de la méthode.

```csharp
// Correct
public void Methode(int obligatoire, string optionnel = "défaut") { }

// Incorrect - ne compilera pas
public void Methode(int optionnel = 0, string obligatoire) { }
```

2. **Types de valeurs par défaut** : Les valeurs par défaut doivent être :
   - Des constantes (comme des littéraux numériques, des chaînes, des valeurs `null`, etc.)
   - Des expressions de création de valeur par défaut (`default` ou `new SomeType()`)
   - Des valeurs d'énumération

```csharp
// Exemples de valeurs par défaut valides
public void ExemplesValides(
    int nombre = 42,                      // Littéral numérique
    string texte = "défaut",              // Littéral de chaîne
    double taux = 3.14,                   // Littéral décimal
    bool actif = true,                    // Littéral booléen
    char caractere = 'A',                 // Littéral caractère
    FileMode mode = FileMode.Open,        // Valeur d'énumération
    DayOfWeek jour = default,             // Valeur par défaut du type
    List<int> liste = null                // Littéral null
) { }
```

3. **Limitations** : Vous ne pouvez pas utiliser des expressions variables ou des appels de méthode comme valeurs par défaut :

```csharp
// Incorrect - ne compilera pas
private int _valeur = 10;
public void MethodeInvalide(
    int a = _valeur,                  // Variable - non valide
    string s = ObtenirValeurDefaut(), // Appel de méthode - non valide
    DateTime date = DateTime.Now      // Propriété non constante - non valide
) { }
```

### Cas d'utilisation typiques des paramètres optionnels

1. **Simplification d'API** quand la plupart des utilisateurs n'ont besoin que d'un sous-ensemble de fonctionnalités :

```csharp
public class ClientHttp
{
    public string EnvoyerRequete(
        string url,
        string methode = "GET",
        Dictionary<string, string> entetes = null,
        string corps = null,
        int timeout = 30,
        bool suivreRedirections = true
    )
    {
        // Implémentation...
        entetes = entetes ?? new Dictionary<string, string>();
        // ...
        return "Réponse";
    }
}

// Utilisation simplifiée
var client = new ClientHttp();
var reponse = client.EnvoyerRequete("https://api.exemple.com/donnees");
```

2. **Configuration avec des valeurs par défaut raisonnables** :

```csharp
public class GestionnaireConfiguration
{
    public void Configurer(
        string cheminFichier = "config.json",
        bool rechargementAutomatique = true,
        int intervalleVerification = 60
    )
    {
        // Implémentation...
    }
}

// Utilisation avec valeurs par défaut
var gestionnaire = new GestionnaireConfiguration();
gestionnaire.Configurer();

// Ou avec certaines personnalisations
gestionnaire.Configurer(cheminFichier: "settings.json");
```

3. **Méthodes Factory** avec options de personnalisation :

```csharp
public class FabriqueRapport
{
    public Rapport CreerRapport(
        DateTime dateDebut,
        DateTime dateFin,
        string titre = "Rapport",
        string auteur = "Système",
        bool inclureGraphiques = true,
        FormatRapport format = FormatRapport.PDF
    )
    {
        // Création du rapport...
        return new Rapport();
    }
}
```

### Paramètres optionnels vs. surcharge de méthodes

Avant l'introduction des paramètres optionnels, la surcharge de méthodes était la façon standard de créer des variantes d'une méthode. Voyons la différence :

**Avec surcharge de méthodes** :

```csharp
public void Dessiner(int x, int y)
{
    Dessiner(x, y, ConsoleColor.White, 1);
}

public void Dessiner(int x, int y, ConsoleColor couleur)
{
    Dessiner(x, y, couleur, 1);
}

public void Dessiner(int x, int y, ConsoleColor couleur, int epaisseur)
{
    // Implémentation complète
}
```

**Avec paramètres optionnels** :

```csharp
public void Dessiner(int x, int y, ConsoleColor couleur = ConsoleColor.White, int epaisseur = 1)
{
    // Implémentation complète
}
```

Avantages des paramètres optionnels :
- **Code plus concis** : une seule méthode au lieu de plusieurs surcharges
- **Maintenance plus facile** : une seule implémentation à modifier
- **Plus flexible** : les appelants peuvent choisir quels paramètres ils veulent spécifier

Avantages de la surcharge de méthodes :
- **Compatibilité avec les versions antérieures** : les surcharges peuvent évoluer sans casser le code existant
- **Implémentations spécialisées** : chaque surcharge peut avoir une implémentation optimisée

## 4.4.2. Ordre d'appel avec paramètres nommés

Les paramètres nommés vous permettent de spécifier explicitement quel paramètre vous passez lors de l'appel d'une méthode, en utilisant le nom du paramètre suivi du caractère `:` et de la valeur.

### Syntaxe des paramètres nommés

```csharp
// Définition de la méthode
public void ConfigurerUtilisateur(string nom, string email, int age, bool actif)
{
    // Implémentation...
}

// Appel avec paramètres nommés
ConfigurerUtilisateur(
    nom: "Alice",
    email: "alice@exemple.com",
    age: 30,
    actif: true
);
```

### Avantages des paramètres nommés

1. **Lisibilité améliorée** : Le code est auto-documenté, surtout pour les paramètres dont le but n'est pas évident par leur valeur.

```csharp
// Sans paramètres nommés - moins clair
CreerDocument("Rapport", 210, 297, true, false, 150);

// Avec paramètres nommés - beaucoup plus clair
CreerDocument(
    titre: "Rapport",
    largeur: 210,
    hauteur: 297,
    couleur: true,
    recto_verso: false,
    dpi: 150
);
```

2. **Flexibilité de l'ordre des paramètres** : Vous pouvez spécifier les paramètres dans n'importe quel ordre lorsque vous utilisez des paramètres nommés.

```csharp
// Les paramètres peuvent être dans un ordre différent de la définition
ConfigurerUtilisateur(
    age: 25,
    nom: "Thomas",
    actif: true,
    email: "thomas@exemple.com"
);
```

3. **Combiner avec les paramètres optionnels** pour n'assigner que les paramètres souhaités.

```csharp
public void EnvoyerEmail(
    string destinataire,
    string sujet,
    string corps,
    bool html = false,
    int priorite = 1,
    List<string> piecesJointes = null
)
{
    // Implémentation...
}

// Spécifier seulement certains paramètres optionnels
EnvoyerEmail(
    destinataire: "contact@exemple.com",
    sujet: "Bienvenue !",
    corps: "Contenu du message...",
    html: true
    // priorite et piecesJointes utilisent leurs valeurs par défaut
);
```

### Règles importantes pour l'utilisation des paramètres nommés

1. **Paramètres positionnels et nommés** : Vous pouvez mélanger les paramètres positionnels (sans nom) et nommés, mais tous les paramètres positionnels doivent apparaître avant les paramètres nommés.

```csharp
// Correct
AfficherMessage("Bonjour", couleur: ConsoleColor.Blue);

// Correct - tous les paramètres sont nommés
AfficherMessage(message: "Bonjour", couleur: ConsoleColor.Blue);

// Incorrect - paramètre positionnel après un paramètre nommé
// AfficherMessage(message: "Bonjour", ConsoleColor.Blue);
```

2. **Omission de paramètres optionnels** : Avec les paramètres nommés, vous pouvez omettre n'importe quel paramètre optionnel, pas seulement ceux à la fin.

```csharp
public void Configure(string nom, int timeout = 30, bool ssl = true, string version = "1.0")
{
    // Implémentation...
}

// Spécifier le premier et le dernier paramètre, en omettant ceux du milieu
Configure(nom: "Service", version: "2.0");
// Équivalent à : Configure("Service", 30, true, "2.0");
```

3. **Tous les paramètres obligatoires doivent être fournis** : Même avec les paramètres nommés, vous devez toujours fournir une valeur pour chaque paramètre obligatoire.

```csharp
// Si 'nom' est obligatoire, cet appel causerait une erreur de compilation
// Configure(version: "2.0");
```

### Cas d'utilisation pratiques pour les paramètres nommés

1. **Méthodes avec de nombreux paramètres booléens** dont le but n'est pas évident par leur valeur :

```csharp
// Sans paramètres nommés - difficile à comprendre
GenererRapport("Ventes", "2023-01", "2023-06", true, false, true, false);

// Avec paramètres nommés - beaucoup plus clair
GenererRapport(
    type: "Ventes",
    dateDebut: "2023-01",
    dateFin: "2023-06",
    inclureGraphiques: true,
    genererPDF: false,
    envoiEmail: true,
    detailsComplets: false
);
```

2. **Clarifier des méthodes avec plusieurs paramètres du même type** :

```csharp
// Sans paramètres nommés - risque de confusion
DefinirMarges(10, 20, 15, 25);

// Avec paramètres nommés - clair et explicite
DefinirMarges(
    gauche: 10,
    droite: 20,
    haut: 15,
    bas: 25
);
```

3. **Appel de constructeurs avec de nombreux paramètres** :

```csharp
var utilisateur = new Utilisateur(
    id: 12345,
    nom: "Dupont",
    prenom: "Jean",
    email: "jean.dupont@exemple.com",
    estAdmin: false,
    dateCreation: DateTime.Now
);
```

4. **API Fluent avec paramètres nommés** :

```csharp
// Créer une configuration avec une API fluent et des paramètres nommés
var config = new ConfigurationBuilder()
    .AjouterSource(type: "fichier", chemin: "config.json", obligatoire: true)
    .AjouterSource(type: "environnement", prefixe: "APP_")
    .DefinirOptions(rechargementAuto: true, intervalleMs: 5000)
    .Construire();
```

### Exemple complet : Création d'une interface flexible avec paramètres optionnels et nommés

Supposons que nous voulions créer une classe pour générer des rapports avec de nombreuses options de configuration. Voici comment les paramètres optionnels et nommés peuvent rendre cette API plus facile à utiliser :

```csharp
public class GenerateurRapport
{
    public enum FormatRapport { PDF, Excel, HTML, Word }
    public enum StyleRapport { Standard, Minimal, Detaille, Personnalise }

    public string GenererRapport(
        string titre,
        DateTime dateDebut,
        DateTime dateFin,
        string[] donnees,
        FormatRapport format = FormatRapport.PDF,
        StyleRapport style = StyleRapport.Standard,
        bool inclureEnTete = true,
        bool inclurePiedPage = true,
        bool inclureGraphiques = true,
        bool inclureTableauxResume = true,
        string auteur = "Système",
        string logoPath = null,
        string dossierDestination = "./rapports",
        bool envoyerParEmail = false,
        string[] destinatairesEmail = null,
        int resolution = 300,
        bool compresser = false,
        string motDePasse = null
    )
    {
        // Valider les entrées
        if (string.IsNullOrEmpty(titre))
            throw new ArgumentException("Le titre ne peut pas être vide", nameof(titre));

        if (dateDebut > dateFin)
            throw new ArgumentException("La date de début doit être antérieure à la date de fin");

        if (donnees == null || donnees.Length == 0)
            throw new ArgumentException("Les données ne peuvent pas être vides", nameof(donnees));

        // Configurer le rapport selon les paramètres
        Console.WriteLine($"Génération d'un rapport avec les paramètres suivants:");
        Console.WriteLine($"- Titre: {titre}");
        Console.WriteLine($"- Période: du {dateDebut:d} au {dateFin:d}");
        Console.WriteLine($"- Format: {format}");
        Console.WriteLine($"- Style: {style}");
        Console.WriteLine($"- En-tête: {inclureEnTete}");
        Console.WriteLine($"- Pied de page: {inclurePiedPage}");
        Console.WriteLine($"- Graphiques: {inclureGraphiques}");
        Console.WriteLine($"- Tableaux résumé: {inclureTableauxResume}");
        Console.WriteLine($"- Auteur: {auteur}");
        Console.WriteLine($"- Logo: {(logoPath != null ? logoPath : "Aucun")}");
        Console.WriteLine($"- Dossier de destination: {dossierDestination}");
        Console.WriteLine($"- Envoi par email: {envoyerParEmail}");

        if (envoyerParEmail && destinatairesEmail != null)
        {
            Console.WriteLine($"- Destinataires: {string.Join(", ", destinatairesEmail)}");
        }

        Console.WriteLine($"- Résolution: {resolution} DPI");
        Console.WriteLine($"- Compression: {compresser}");
        Console.WriteLine($"- Protection par mot de passe: {(motDePasse != null ? "Oui" : "Non")}");

        // Logique de génération du rapport...
        // (Simplifiée pour l'exemple)

        string nomFichier = $"{dossierDestination}/{titre.Replace(" ", "_")}_{dateDebut:yyyyMMdd}_{dateFin:yyyyMMdd}.{format.ToString().ToLower()}";
        Console.WriteLine($"\nRapport généré: {nomFichier}");

        return nomFichier;
    }
}
```

Avec cette classe, nous pouvons créer des rapports de différentes façons, selon les besoins :

```csharp
var generateur = new GenerateurRapport();

// Génération simple avec valeurs par défaut
string rapport1 = generateur.GenererRapport(
    "Ventes Mensuelles",
    new DateTime(2023, 1, 1),
    new DateTime(2023, 1, 31),
    new[] { "Données de vente..." }
);

// Génération avec format personnalisé
string rapport2 = generateur.GenererRapport(
    "Analyse Clients",
    new DateTime(2023, 1, 1),
    new DateTime(2023, 3, 31),
    new[] { "Données clients..." },
    format: GenerateurRapport.FormatRapport.Excel,
    style: GenerateurRapport.StyleRapport.Detaille
);

// Génération complète avec de nombreuses options personnalisées
string rapport3 = generateur.GenererRapport(
    titre: "Rapport Financier Annuel",
    dateDebut: new DateTime(2022, 1, 1),
    dateFin: new DateTime(2022, 12, 31),
    donnees: new[] { "Données financières..." },
    format: GenerateurRapport.FormatRapport.PDF,
    style: GenerateurRapport.StyleRapport.Personnalise,
    inclureGraphiques: true,
    inclureTableauxResume: true,
    auteur: "Département Financier",
    logoPath: "./logo.png",
    dossierDestination: "./rapports/financiers",
    envoyerParEmail: true,
    destinatairesEmail: new[] { "direction@exemple.com", "comptabilite@exemple.com" },
    resolution: 600,
    compresser: true,
    motDePasse: "confidentiel123"
);
```

Comme vous pouvez le voir, les paramètres optionnels et nommés rendent cette API très flexible :
- Les utilisateurs peuvent spécifier uniquement les options qui les intéressent
- Les appels sont auto-documentés et faciles à comprendre
- Pas besoin de créer de nombreuses surcharges pour différentes combinaisons de paramètres

### Limites et considérations

1. **Évolution des API** : Faire attention lors de la modification des valeurs par défaut ou de l'ajout de nouveaux paramètres optionnels, car cela peut affecter le comportement existant.

2. **Versionnement** : Si vous changez la valeur par défaut d'un paramètre optionnel, le comportement du code existant peut changer de manière inattendue.

3. **Interopérabilité** : Les paramètres nommés fonctionnent bien en C#, mais peuvent poser des problèmes lors de l'interopérabilité avec d'autres langages.

4. **Débordement de paramètres** : Évitez de créer des méthodes avec trop de paramètres optionnels. Envisagez plutôt d'utiliser un objet de configuration si vous avez plus de 4-5 paramètres.

```csharp
// Alternative avec objet de configuration pour de nombreux paramètres
public class OptionsRapport
{
    public string Titre { get; set; }
    public DateTime DateDebut { get; set; }
    public DateTime DateFin { get; set; }
    public string[] Donnees { get; set; }
    public FormatRapport Format { get; set; } = FormatRapport.PDF;
    public StyleRapport Style { get; set; } = StyleRapport.Standard;
    public bool InclureEnTete { get; set; } = true;
    // ... autres propriétés ...
}

public string GenererRapport(OptionsRapport options)
{
    // Implémentation...
    return "chemin_rapport.pdf";
}

// Utilisation
var options = new OptionsRapport
{
    Titre = "Rapport Annuel",
    DateDebut = new DateTime(2022, 1, 1),
    DateFin = new DateTime(2022, 12, 31),
    Donnees = new[] { "Données..." },
    Format = FormatRapport.Excel
    // Les autres propriétés utilisent leurs valeurs par défaut
};

string rapport = GenererRapport(options);
```

## Bonnes pratiques

Voici quelques bonnes pratiques pour utiliser efficacement les paramètres optionnels et nommés :

1. **Utiliser des noms de paramètres descriptifs** qui indiquent clairement leur fonction.

2. **Placer les paramètres obligatoires en premier**, suivis des paramètres optionnels.

3. **Choisir des valeurs par défaut judicieuses** qui correspondent à l'utilisation la plus courante.

4. **Documenter les paramètres** avec des commentaires XML pour expliquer leur rôle et leurs valeurs par défaut.

```csharp
/// <summary>
/// Configure un service avec les options spécifiées.
/// </summary>
/// <param name="nom">Le nom du service (obligatoire).</param>
/// <param name="timeout">Le délai d'attente en secondes avant expiration (par défaut: 30).</param>
/// <param name="retryCount">Le nombre de tentatives en cas d'échec (par défaut: 3).</param>
/// <param name="logLevel">Le niveau de journalisation (par défaut: Warning).</param>
public void ConfigurerService(
    string nom,
    int timeout = 30,
    int retryCount = 3,
    LogLevel logLevel = LogLevel.Warning)
{
    // Implémentation...
}
```

5. **Utiliser les paramètres nommés** pour les appels de méthode avec de nombreux paramètres, surtout quand plusieurs paramètres ont le même type.

6. **Éviter de modifier les valeurs par défaut** dans les versions futures de l'API pour ne pas casser le code existant.

7. **Préférer des objets de configuration** pour les méthodes avec plus de 4-5 paramètres optionnels.

## Conclusion

Les paramètres optionnels et nommés sont des fonctionnalités puissantes de C# qui rendent vos API plus flexibles et plus faciles à utiliser. Ils réduisent la nécessité de créer de nombreuses surcharges de méthodes et améliorent la lisibilité du code en permettant des appels auto-documentés.

En combinant ces deux fonctionnalités, vous pouvez créer des méthodes très configurables tout en gardant l'interface simple et intuitive pour les cas d'utilisation courants. Cela conduit à un code plus maintenable et plus facile à comprendre, tant pour vous que pour les autres développeurs qui utiliseront votre code.

⏭️ 4.5. [Méthodes d'extension](/04-methodes-et-fonctions/4-5-methodes-dextension.md)
