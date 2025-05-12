# 15.4. Analyse de code et refactoring

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

L'analyse de code et le refactoring sont deux pratiques essentielles pour maintenir et améliorer la qualité de votre code C#. L'analyse de code permet d'identifier les problèmes potentiels, tandis que le refactoring est le processus de restructuration du code existant sans en modifier le comportement externe.

Dans ce chapitre, nous explorerons les outils et techniques qui vous aideront à écrire un code C# plus propre, plus maintenable et de meilleure qualité.

## 15.4.1. Analyseurs statiques

Les analyseurs statiques examinent votre code sans l'exécuter pour détecter des problèmes potentiels comme les bugs, les vulnérabilités de sécurité, les violations de style et les performances sous-optimales.

### Analyseurs intégrés de .NET

Depuis .NET 5, Microsoft a intégré des analyseurs de code dans le SDK .NET par défaut, basés sur le framework Roslyn (le compilateur C#).

Pour activer les analyseurs intégrés dans votre projet, ajoutez les lignes suivantes à votre fichier `.csproj` :

```xml
<PropertyGroup>
  <EnableNETAnalyzers>true</EnableNETAnalyzers>
  <AnalysisLevel>latest</AnalysisLevel>
  <AnalysisMode>All</AnalysisMode>
</PropertyGroup>
```

Les niveaux d'avertissement peuvent être configurés comme suit :

```xml
<PropertyGroup>
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  <!-- Ou pour des règles spécifiques -->
  <WarningsAsErrors>CS8600;CS8602;CS8603</WarningsAsErrors>
</PropertyGroup>
```

### Installation de packages d'analyseurs supplémentaires

Vous pouvez ajouter plus d'analyseurs via NuGet :

```bash
# Analyseurs Microsoft de qualité de code
dotnet add package Microsoft.CodeAnalysis.NetAnalyzers

# Analyseurs de sécurité
dotnet add package Microsoft.CodeAnalysis.BannedApiAnalyzers
dotnet add package Microsoft.CodeAnalysis.FxCopAnalyzers

# Analyseurs de style
dotnet add package Microsoft.CodeAnalysis.CSharp.CodeStyle
```

### Configuration avec .editorconfig

Le fichier `.editorconfig` permet de configurer les règles d'analyse et de style de manière cohérente pour tous les membres de l'équipe :

```editorconfig
# Fichier .editorconfig à la racine de votre solution
root = true

[*.cs]
# Indentation et espacement
indent_style = space
indent_size = 4
tab_width = 4

# Nouvelles lignes
end_of_line = crlf
insert_final_newline = true

# Règles de style C#
csharp_style_var_for_built_in_types = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = true:suggestion

# Règles d'analyse
dotnet_diagnostic.CS8600.severity = error  # Convertir le littéral null ou la valeur null possible en type non nullable
dotnet_diagnostic.CA1063.severity = warning  # Implémenter IDisposable correctement
```

### Utilisation de Roslyn Analyzers dans Visual Studio

Visual Studio inclut des analyseurs Roslyn par défaut. Pour les configurer :

1. Cliquez-droit sur le projet dans l'Explorateur de solutions
2. Sélectionnez "Propriétés"
3. Naviguez vers l'onglet "Analyse de code"
4. Activez "Activer l'analyse de code .NET"

![Configuration des analyseurs dans Visual Studio](https://i.imgur.com/example1.png)

### Exemple d'analyse statique en action

Voici comment les analyseurs peuvent vous aider :

```csharp
// Le code original avec des problèmes
public class DataProcessor
{
    private SqlConnection connection;

    public void ProcessData(string[] data)
    {
        connection = new SqlConnection("connectionString");
        connection.Open();

        // Traitement des données

        // Oubli de fermer la connexion
    }
}
```

Les analyseurs détecteront plusieurs problèmes :

1. Possible fuite de ressources (la connexion SQL n'est pas fermée)
2. La classe doit implémenter IDisposable
3. La chaîne de connexion est codée en dur

Code corrigé suite aux avertissements d'analyse :

```csharp
public class DataProcessor : IDisposable
{
    private SqlConnection? connection;
    private bool disposed = false;

    public void ProcessData(string[] data)
    {
        connection = new SqlConnection(ConfigurationManager.ConnectionStrings["MainDB"].ConnectionString);

        try
        {
            connection.Open();
            // Traitement des données
        }
        finally
        {
            connection.Close();
        }
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                connection?.Dispose();
            }

            disposed = true;
        }
    }
}
```

## 15.4.2. Style Cop

StyleCop est un outil d'analyse qui se concentre spécifiquement sur les règles de style et de formatage du code C#.

### Installation de StyleCop Analyzers

Pour ajouter StyleCop à votre projet :

```bash
dotnet add package StyleCop.Analyzers
```

Ou via NuGet Package Manager dans Visual Studio :
1. Clic droit sur le projet → Gérer les packages NuGet
2. Rechercher "StyleCop.Analyzers"
3. Installer le package

### Configuration de StyleCop

Créez un fichier `stylecop.json` à la racine de votre projet :

```json
{
  "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
  "settings": {
    "documentationRules": {
      "companyName": "VotreEntreprise",
      "copyrightText": "Copyright (c) {companyName}. Tous droits réservés.",
      "xmlHeader": true,
      "headerDecoration": "-----------------------------------------------------------------------"
    },
    "orderingRules": {
      "usingDirectivesPlacement": "outsideNamespace",
      "systemUsingDirectivesFirst": true
    },
    "namingRules": {
      "includeInferredTupleElementNames": true,
      "tupleElementNameCasing": "camelCase"
    },
    "readabilityRules": {
      "allowBuiltInTypeAliases": false
    }
  }
}
```

Assurez-vous que ce fichier est inclus dans votre projet (`.csproj`) :

```xml
<ItemGroup>
  <AdditionalFiles Include="stylecop.json" />
</ItemGroup>
```

### Règles de style importantes

StyleCop applique de nombreuses règles, dont :

1. **Organisation du code** :
   - Les directives `using` doivent être correctement ordonnées
   - Les membres de classe doivent suivre un ordre spécifique
   - Les espaces de noms doivent correspondre à la structure des dossiers

2. **Documentation** :
   - Les classes et méthodes publiques doivent être documentées
   - Le format XML doit être respecté

3. **Nommage** :
   - Les noms de variables doivent suivre les conventions camelCase/PascalCase
   - Les noms doivent être descriptifs

4. **Mise en forme** :
   - Espacement des accolades
   - Utilisation correcte des parenthèses
   - Longueur des lignes

### Exemple de correction StyleCop

Code avec violations StyleCop :

```csharp
using System.Collections.Generic;
using System;

namespace myApplication {
    class userManager {
        public bool add_user(string userName) {
            // Code ici
            return true;
        }
    }
}
```

Code corrigé selon les règles StyleCop :

```csharp
using System;
using System.Collections.Generic;

namespace MyApplication
{
    /// <summary>
    /// Gère les opérations relatives aux utilisateurs.
    /// </summary>
    public class UserManager
    {
        /// <summary>
        /// Ajoute un nouvel utilisateur au système.
        /// </summary>
        /// <param name="userName">Le nom de l'utilisateur à ajouter.</param>
        /// <returns>True si l'utilisateur a été ajouté avec succès, sinon False.</returns>
        public bool AddUser(string userName)
        {
            // Code ici
            return true;
        }
    }
}
```

### Ignorer des règles spécifiques

Si certaines règles ne conviennent pas à votre projet, vous pouvez les désactiver :

```csharp
// Désactiver une règle pour une ligne spécifique
#pragma warning disable SA1633 // Le fichier doit avoir un en-tête
using System;
#pragma warning restore SA1633

// Désactiver une règle pour une méthode
[SuppressMessage("StyleCop.CSharp.DocumentationRules",
                 "SA1600:ElementsMustBeDocumented",
                 Justification = "Méthode privée interne")]
private void MéthodeInterne()
{
    // Code ici
}
```

Ou globalement dans le fichier `.editorconfig` :

```editorconfig
[*.cs]
dotnet_diagnostic.SA1633.severity = none
```

## 15.4.3. Métriques de code

Les métriques de code sont des mesures qui aident à évaluer la qualité, la complexité et la maintenabilité du code.

### Métriques de code importantes

1. **Complexité cyclomatique** : Mesure le nombre de chemins d'exécution indépendants dans une méthode.
   - Valeur recommandée : < 10
   - Une valeur élevée indique un code difficile à tester et à maintenir.

2. **Profondeur d'héritage** : Nombre de niveaux d'héritage.
   - Valeur recommandée : < 5
   - Une hiérarchie trop profonde peut rendre le code difficile à comprendre.

3. **Couplage entre classes** : Nombre de dépendances entre classes.
   - Valeur recommandée : aussi bas que possible
   - Un couplage élevé rend les modifications plus difficiles.

4. **Lignes de code** : Nombre de lignes de code par méthode/classe.
   - Valeur recommandée pour les méthodes : < 30 lignes
   - Valeur recommandée pour les classes : < 500 lignes

5. **Cohésion** : Mesure à quel point les méthodes d'une classe sont liées.
   - Valeur recommandée : élevée (les méthodes travaillent sur les mêmes données)

### Outils pour mesurer les métriques de code

#### Visual Studio intégré

Visual Studio Professional et Enterprise incluent un outil d'analyse de métriques de code :

1. Menu "Analyser" → "Calculer les métriques de code pour la solution"
2. Les résultats s'affichent dans la fenêtre "Résultats des métriques de code"

#### Extension Visual Studio Metrics

Pour Visual Studio Community ou des métriques plus avancées :

1. Menu "Extensions" → "Gérer les extensions"
2. Recherchez "Metrics" ou "Code Metrics"
3. Installez une extension comme "Code Metrics" ou "NDepend Lite"

#### NDepend

NDepend est un outil commercial puissant pour l'analyse approfondie du code :

```bash
# Installation de la ligne de commande NDepend (version d'évaluation)
dotnet tool install -g NDepend.Console
```

Exemple d'utilisation :

```bash
ndepend.console MaSolution.sln
```

NDepend génère un rapport détaillé avec :
- Tableau de bord de qualité
- Tendances et régressions
- Visualisations du code (matrices de dépendance, graphes...)
- Métriques détaillées par méthode/classe/namespace

#### Outil en ligne de commande de métriques

Pour les pipelines CI/CD :

```bash
dotnet add package Microsoft.CodeAnalysis.Metrics
dotnet metrics MaLibrairie.csproj
```

### Interprétation des métriques et résolution des problèmes

| Métrique | Problème potentiel | Solution |
|----------|-------------------|----------|
| Complexité cyclomatique > 10 | Méthode trop complexe | Décomposer en méthodes plus petites |
| Lignes de code > 30 par méthode | Méthode trop longue | Extraire des méthodes plus spécifiques |
| Couplage élevé | Dépendances excessives | Utiliser l'injection de dépendances et les interfaces |
| Faible cohésion | Classe avec trop de responsabilités | Appliquer le principe de responsabilité unique |
| Profondeur d'héritage > 5 | Hiérarchie trop complexe | Préférer la composition à l'héritage |

## 15.4.4. Techniques de refactoring

Le refactoring est l'art d'améliorer la structure du code sans changer son comportement externe. Voici les techniques les plus utiles.

### Extraction de méthode

Isoler une portion de code dans une méthode dédiée.

Avant :
```csharp
public decimal CalculerPaiement(Employé employé)
{
    decimal salaire = employé.SalaireBase;

    if (employé.AnnéesAncienneté > 5)
    {
        decimal bonus = employé.SalaireBase * 0.1m;
        salaire += bonus;
    }

    if (employé.Performance > 8)
    {
        decimal prime = employé.SalaireBase * 0.15m;
        salaire += prime;
    }

    return salaire;
}
```

Après :
```csharp
public decimal CalculerPaiement(Employé employé)
{
    decimal salaire = employé.SalaireBase;

    salaire += CalculerBonusAncienneté(employé);
    salaire += CalculerPrimePerformance(employé);

    return salaire;
}

private decimal CalculerBonusAncienneté(Employé employé)
{
    if (employé.AnnéesAncienneté > 5)
    {
        return employé.SalaireBase * 0.1m;
    }
    return 0;
}

private decimal CalculerPrimePerformance(Employé employé)
{
    if (employé.Performance > 8)
    {
        return employé.SalaireBase * 0.15m;
    }
    return 0;
}
```

### Déplacement de méthode

Déplacer une méthode vers une classe où elle est plus pertinente.

Avant :
```csharp
public class Client
{
    public string Nom { get; set; }
    public string Email { get; set; }
    public List<Commande> Commandes { get; set; }

    public decimal CalculerTotalCommandes()
    {
        decimal total = 0;
        foreach (var commande in Commandes)
        {
            total += commande.CalculerTotal();
        }
        return total;
    }
}
```

Après :
```csharp
public class Client
{
    public string Nom { get; set; }
    public string Email { get; set; }
    public List<Commande> Commandes { get; set; }
}

public class GestionnaireCommandes
{
    public decimal CalculerTotalCommandes(Client client)
    {
        decimal total = 0;
        foreach (var commande in client.Commandes)
        {
            total += commande.CalculerTotal();
        }
        return total;
    }
}
```

### Remplacement de conditionnels par polymorphisme

Remplacer une structure conditionnelle par une hiérarchie de classes.

Avant :
```csharp
public class Employé
{
    public enum TypeEmployé { Permanent, Temporaire, Stagiaire }

    public TypeEmployé Type { get; set; }
    public decimal SalaireBase { get; set; }

    public decimal CalculerPaiement()
    {
        switch (Type)
        {
            case TypeEmployé.Permanent:
                return SalaireBase + (SalaireBase * 0.2m);
            case TypeEmployé.Temporaire:
                return SalaireBase;
            case TypeEmployé.Stagiaire:
                return SalaireBase * 0.5m;
            default:
                throw new ArgumentException("Type d'employé non reconnu");
        }
    }
}
```

Après :
```csharp
public abstract class Employé
{
    public decimal SalaireBase { get; set; }

    public abstract decimal CalculerPaiement();
}

public class EmployéPermanent : Employé
{
    public override decimal CalculerPaiement()
    {
        return SalaireBase + (SalaireBase * 0.2m);
    }
}

public class EmployéTemporaire : Employé
{
    public override decimal CalculerPaiement()
    {
        return SalaireBase;
    }
}

public class Stagiaire : Employé
{
    public override decimal CalculerPaiement()
    {
        return SalaireBase * 0.5m;
    }
}
```

### Encapsulation de champ

Transformer un champ public en propriété avec accesseurs et mutateurs.

Avant :
```csharp
public class Personne
{
    public string nom;
    public int age;
}
```

Après :
```csharp
public class Personne
{
    private string _nom;
    private int _age;

    public string Nom
    {
        get { return _nom; }
        set { _nom = value; }
    }

    public int Age
    {
        get { return _age; }
        set
        {
            if (value < 0)
                throw new ArgumentException("L'âge ne peut pas être négatif");
            _age = value;
        }
    }
}
```

### Introduction d'un objet paramètre

Remplacer une longue liste de paramètres par un objet.

Avant :
```csharp
public void CréerCommande(string nomClient, string adresse, string ville,
                         string codePostal, string pays, List<string> produits,
                         decimal montant, bool estPrioritaire)
{
    // Création de commande
}
```

Après :
```csharp
public class AdresseLivraison
{
    public string Adresse { get; set; }
    public string Ville { get; set; }
    public string CodePostal { get; set; }
    public string Pays { get; set; }
}

public class DétailsCommande
{
    public string NomClient { get; set; }
    public AdresseLivraison Adresse { get; set; }
    public List<string> Produits { get; set; }
    public decimal Montant { get; set; }
    public bool EstPrioritaire { get; set; }
}

public void CréerCommande(DétailsCommande détails)
{
    // Création de commande
}
```

### Extraction d'interface

Créer une interface à partir d'une classe pour diminuer le couplage.

Avant :
```csharp
public class EmailService
{
    public void EnvoyerEmail(string destinataire, string sujet, string corps)
    {
        // Envoi d'email
    }
}

public class GestionnaireNotifications
{
    private EmailService _emailService = new EmailService();

    public void NotifierUtilisateur(Utilisateur utilisateur, string message)
    {
        _emailService.EnvoyerEmail(utilisateur.Email, "Notification", message);
    }
}
```

Après :
```csharp
public interface IServiceNotification
{
    void EnvoyerEmail(string destinataire, string sujet, string corps);
}

public class EmailService : IServiceNotification
{
    public void EnvoyerEmail(string destinataire, string sujet, string corps)
    {
        // Envoi d'email
    }
}

public class GestionnaireNotifications
{
    private readonly IServiceNotification _serviceNotification;

    public GestionnaireNotifications(IServiceNotification serviceNotification)
    {
        _serviceNotification = serviceNotification;
    }

    public void NotifierUtilisateur(Utilisateur utilisateur, string message)
    {
        _serviceNotification.EnvoyerEmail(utilisateur.Email, "Notification", message);
    }
}
```

### Utiliser les fonctionnalités de Visual Studio

Visual Studio offre des outils de refactoring intégrés :

1. **Extraction de méthode** : Sélectionner du code → clic droit → Refactoriser → Extraire méthode
2. **Renommage** : Clic droit sur un symbole → Renommer (ou F2)
3. **Extraction d'interface** : Clic droit sur une classe → Refactoriser → Extraire interface
4. **Encapsulation de champ** : Clic droit sur un champ → Refactoriser → Encapsuler champ

## 15.4.5. Code smells et remèdes

Les "code smells" (odeurs de code) sont des indicateurs de problèmes potentiels dans le code. Voici comment les identifier et les résoudre.

### Méthode trop longue

**Symptômes** :
- Méthode de plus de 20-30 lignes
- Plusieurs niveaux d'indentation
- Commentaires pour séparer des blocs logiques

**Solutions** :
- Extraire des méthodes
- Appliquer le principe de responsabilité unique
- Créer des classes auxiliaires

Exemple de résolution :

```csharp
// Avant - Méthode trop longue
public void TraiterCommande(Commande commande)
{
    // Validation
    if (commande == null)
        throw new ArgumentNullException(nameof(commande));
    if (string.IsNullOrEmpty(commande.RéférenceClient))
        throw new ArgumentException("Référence client requise");
    if (commande.Produits.Count == 0)
        throw new ArgumentException("Commande sans produits");

    // Vérification stock
    bool stockSuffisant = true;
    foreach (var produit in commande.Produits)
    {
        var stockProduit = _stockService.VérifierStock(produit.Code);
        if (stockProduit < produit.Quantité)
        {
            stockSuffisant = false;
            break;
        }
    }
    if (!stockSuffisant)
    {
        _notificationService.NotifierRupture(commande);
        return;
    }

    // Calcul prix
    decimal total = 0;
    foreach (var produit in commande.Produits)
    {
        decimal prixUnitaire = _prixService.ObtenirPrix(produit.Code);
        total += prixUnitaire * produit.Quantité;
    }
    commande.Total = total;

    // Enregistrement commande
    _commandeRepository.Enregistrer(commande);

    // Notification client
    _notificationService.EnvoyerConfirmation(commande);
}
```

```csharp
// Après - Méthode découpée en responsabilités distinctes
public void TraiterCommande(Commande commande)
{
    ValiderCommande(commande);

    if (!VérifierDisponibilitéStock(commande))
    {
        _notificationService.NotifierRupture(commande);
        return;
    }

    CalculerPrixTotal(commande);
    EnregistrerCommande(commande);
    NotifierClient(commande);
}

private void ValiderCommande(Commande commande)
{
    if (commande == null)
        throw new ArgumentNullException(nameof(commande));
    if (string.IsNullOrEmpty(commande.RéférenceClient))
        throw new ArgumentException("Référence client requise");
    if (commande.Produits.Count == 0)
        throw new ArgumentException("Commande sans produits");
}

private bool VérifierDisponibilitéStock(Commande commande)
{
    foreach (var produit in commande.Produits)
    {
        var stockProduit = _stockService.VérifierStock(produit.Code);
        if (stockProduit < produit.Quantité)
        {
            return false;
        }
    }
    return true;
}

private void CalculerPrixTotal(Commande commande)
{
    decimal total = 0;
    foreach (var produit in commande.Produits)
    {
        decimal prixUnitaire = _prixService.ObtenirPrix(produit.Code);
        total += prixUnitaire * produit.Quantité;
    }
    commande.Total = total;
}

private void EnregistrerCommande(Commande commande)
{
    _commandeRepository.Enregistrer(commande);
}

private void NotifierClient(Commande commande)
{
    _notificationService.EnvoyerConfirmation(commande);
}
```

### Classe God (trop grande)

**Symptômes** :
- Classe avec trop de champs et méthodes (plus de 500 lignes)
- Responsabilités multiples et non liées
- Forte dépendance d'autres classes

**Solutions** :
- Décomposer en classes plus petites et spécialisées
- Extraire des hiérarchies de classes
- Appliquer le modèle Composite

### Duplication de code

**Symptômes** :
- Portions de code identiques ou très similaires
- Mêmes algorithmes implémentés différemment

**Solutions** :
- Extraire dans des méthodes utilitaires
- Créer des classes de base communes
- Utiliser l'héritage ou la composition

### Utilisation excessive de commentaires

**Symptômes** :
- Beaucoup de commentaires expliquant "comment" le code fonctionne
- Commentaires qui maintiennent des sections de code désactivées

**Solutions** :
- Rendre le code auto-explicatif
- Extraire des méthodes avec des noms descriptifs
- Remplacer les commentaires par des assertions ou des tests

### Dépendances rigides

**Symptômes** :
- Classes qui instancient directement leurs dépendances
- Couplage fort entre classes
- Difficultés à tester en isolation

**Solutions** :
- Injection de dépendances
- Utilisation d'interfaces
- Application du principe d'inversion de dépendance

Exemple de résolution :

```csharp
// Avant - Dépendance rigide
public class RapportService
{
    private readonly DatabaseRepository _repository = new DatabaseRepository();

    public Rapport GénérerRapportVentes(DateTime début, DateTime fin)
    {
        var données = _repository.ObtenirVentes(début, fin);

        // Logique de génération de rapport

        return new Rapport(données);
    }
}
```

```csharp
// Après - Dépendance inversée
public interface IRepository
{
    IEnumerable<Vente> ObtenirVentes(DateTime début, DateTime fin);
}

public class DatabaseRepository : IRepository
{
    public IEnumerable<Vente> ObtenirVentes(DateTime début, DateTime fin)
    {
        // Implémentation
    }
}

public class RapportService
{
    private readonly IRepository _repository;

    public RapportService(IRepository repository)
    {
        _repository = repository;
    }

    public Rapport GénérerRapportVentes(DateTime début, DateTime fin)
    {
        var données = _repository.ObtenirVentes(début, fin);

        // Logique de génération de rapport

        return new Rapport(données);
    }
}
```

### Classe anémique

**Symptômes** :
- Classe avec seulement des propriétés et peu ou pas de comportement
- Logique métier située dans d'autres classes de service

**Solutions** :
- Déplacer la logique métier dans les classes appropriées
- Implémenter le modèle de domaine riche
- Utiliser des classes de valeur pour les concepts immuables

### Méthode avec trop de paramètres

**Symptômes** :
- Méthodes avec plus de 3-4 paramètres
- Difficultés à comprendre l'ordre et le sens des paramètres

**Solutions** :
- Créer des objets paramètres
- Décomposer la méthode en méthodes plus petites
- Utiliser des méthodes d'extension pour les configurations optionnelles

### Switches répétés

**Symptômes** :
- Structures switch/case basées sur le même critère, réparties dans le code
- Besoin de modifier plusieurs endroits lors de l'ajout d'un nouveau cas

**Solutions** :
- Remplacer par du polymorphisme
- Utiliser le modèle Strategy
- Implémenter des tables de correspondance (dictionnaires)

### Utilisation d'indices magiques

```csharp
// Avant - Indices magiques
public decimal CalculerRemise(decimal montant, int typeClient)
{
    if (typeClient == 1)
        return montant * 0.1m;
    else if (typeClient == 2)
        return montant * 0.15m;
    else if (typeClient == 3)
        return montant * 0.2m;
    else
        return 0m;
}
```

```csharp
// Après - Utilisation d'énumérations et de constantes
public enum TypeClient
{
    Standard = 1,
    Premium = 2,
    VIP = 3
}

public class TauxRemise
{
    public const decimal STANDARD = 0.1m;
    public const decimal PREMIUM = 0.15m;
    public const decimal VIP = 0.2m;
}

public decimal CalculerRemise(decimal montant, TypeClient typeClient)
{
    switch (typeClient)
    {
        case TypeClient.Standard:
            return montant * TauxRemise.STANDARD;
        case TypeClient.Premium:
            return montant * TauxRemise.PREMIUM;
        case TypeClient.VIP:
            return montant * TauxRemise.VIP;
        default:
            return 0m;
    }
}
```

### Complexité cognitive élevée

**Symptômes** :
- Conditions imbriquées et complexes
- Expressions booléennes longues et difficiles à comprendre
- Multiples conditions de sortie dans une méthode

**Solutions** :
- Extraire les conditions complexes en méthodes bien nommées
- Utiliser des clauses de garde en début de méthode
- Simplifier les expressions booléennes en les décomposant

Exemple de résolution :

```csharp
// Avant - Complexité cognitive élevée
public bool EstÉligiblePourPromotion(Employé employé)
{
    if (employé != null &&
        (employé.Performance > 8 ||
         (employé.Performance > 7 && employé.AnnéesAncienneté > 5)) &&
        employé.DernièrePromotion.AddYears(2) < DateTime.Now &&
        !employé.EstEnPériodeEssai &&
        (employé.Département == "Ventes" ||
         employé.Département == "Marketing" ||
         (employé.Département == "R&D" && employé.Niveau > 3)))
    {
        return true;
    }
    return false;
}
```

```csharp
// Après - Complexité réduite avec méthodes explicites
public bool EstÉligiblePourPromotion(Employé employé)
{
    if (employé == null)
        return false;

    if (employé.EstEnPériodeEssai)
        return false;

    if (!APerformanceSuffisante(employé))
        return false;

    if (!IntervallePromotionRespecté(employé))
        return false;

    if (!DépartementÉligible(employé))
        return false;

    return true;
}

private bool APerformanceSuffisante(Employé employé)
{
    return employé.Performance > 8 ||
           (employé.Performance > 7 && employé.AnnéesAncienneté > 5);
}

private bool IntervallePromotionRespecté(Employé employé)
{
    return employé.DernièrePromotion.AddYears(2) < DateTime.Now;
}

private bool DépartementÉligible(Employé employé)
{
    return employé.Département == "Ventes" ||
           employé.Département == "Marketing" ||
           (employé.Département == "R&D" && employé.Niveau > 3);
}
```

### YAGNI (You Aren't Gonna Need It)

**Symptômes** :
- Code qui n'est pas utilisé actuellement
- Fonctionnalités implémentées "au cas où"
- Abstractions inutilement complexes

**Solutions** :
- Supprimer le code non utilisé
- Mettre en œuvre les fonctionnalités uniquement lorsqu'elles sont nécessaires
- Simplifier les abstractions

### Couplage temporel

**Symptômes** :
- Méthodes qui doivent être appelées dans un ordre spécifique
- Dépendances cachées entre différentes parties du code

**Solutions** :
- Rendre l'ordre explicite en regroupant les méthodes
- Utiliser des objets d'état
- Appliquer le modèle Builder

Exemple de résolution :

```csharp
// Avant - Couplage temporel
public class Commande
{
    public void Créer(Client client)
    {
        // Création de la commande
    }

    public void AjouterProduit(Produit produit, int quantité)
    {
        // Ajout du produit
    }

    public void CalculerTotal()
    {
        // Calcul du total
    }

    public void Soumettre()
    {
        // Soumission de la commande
    }
}

// Utilisation - Ordre implicite
var commande = new Commande();
commande.Créer(client);
commande.AjouterProduit(produit1, 2);
commande.AjouterProduit(produit2, 1);
commande.CalculerTotal(); // Doit être appelé après l'ajout des produits
commande.Soumettre(); // Doit être appelé après le calcul du total
```

```csharp
// Après - Utilisation du pattern Builder
public class CommandeBuilder
{
    private Commande _commande = new Commande();

    public CommandeBuilder PourClient(Client client)
    {
        _commande.Client = client;
        return this;
    }

    public CommandeBuilder AvecProduit(Produit produit, int quantité)
    {
        _commande.AjouterProduit(produit, quantité);
        return this;
    }

    public Commande Construire()
    {
        _commande.CalculerTotal();
        return _commande;
    }
}

// Utilisation - Ordre explicite
var commande = new CommandeBuilder()
    .PourClient(client)
    .AvecProduit(produit1, 2)
    .AvecProduit(produit2, 1)
    .Construire();

commandeService.Soumettre(commande);
```

### Law of Demeter (Principe de moindre connaissance)

**Symptômes** :
- Chaînes d'appels de méthode (a.getB().getC().getD().méthode())
- Dépendances transitives excessives

**Solutions** :
- Éviter les chaînes d'appels
- Ajouter des méthodes déléguées
- Restructurer la hiérarchie des objets

Exemple de résolution :

```csharp
// Avant - Violation de la loi de Demeter
public class Client
{
    public Adresse Adresse { get; set; }
}

public class Adresse
{
    public Ville Ville { get; set; }
}

public class Ville
{
    public string CodePostal { get; set; }
}

// Utilisation - Chaîne d'appels
public bool EstDansParis(Client client)
{
    return client.Adresse.Ville.CodePostal.StartsWith("75");
}
```

```csharp
// Après - Respect de la loi de Demeter
public class Client
{
    public Adresse Adresse { get; set; }

    public string ObtenirCodePostal()
    {
        return Adresse?.ObtenirCodePostal();
    }

    public bool EstDansVille(string prefixeCodePostal)
    {
        string codePostal = ObtenirCodePostal();
        return codePostal != null && codePostal.StartsWith(prefixeCodePostal);
    }
}

public class Adresse
{
    public Ville Ville { get; set; }

    public string ObtenirCodePostal()
    {
        return Ville?.CodePostal;
    }
}

public class Ville
{
    public string CodePostal { get; set; }
}

// Utilisation - Sans chaîne d'appels
public bool EstDansParis(Client client)
{
    return client.EstDansVille("75");
}
```

### Dette technique et stratégie de remboursement

La dette technique représente le coût futur d'avoir choisi une solution rapide mais imparfaite au lieu d'une approche plus complète.

**Causes communes de dette technique** :
- Pression des délais
- Manque de connaissances
- Absence de revues de code
- Documentation insuffisante
- Tests insuffisants

**Stratégies de remboursement** :

1. **Refactoring opportuniste** :
   - Améliorer le code au fur et à mesure des modifications
   - Règle du scout : « Laissez le code plus propre que vous ne l'avez trouvé »

2. **Refactoring planifié** :
   - Dédier du temps spécifique aux efforts de refactoring
   - Intégrer le refactoring dans les cycles de développement

3. **Réécriture ciblée** :
   - Identifier les modules problématiques
   - Réécrire ces modules tout en maintenant les interfaces

4. **Refactoring préventif** :
   - Mettre en place des analyses automatiques
   - Définir des seuils de qualité pour les métriques de code

## Outils et pratiques pour maintenir la qualité du code

### Revues de code

Les revues de code sont essentielles pour maintenir la qualité :

1. **Revues pré-commit** :
   - Vérifier le code avant qu'il ne soit intégré
   - Se concentrer sur la lisibilité, la maintenabilité et les problèmes potentiels

2. **Pair programming** :
   - Deux développeurs travaillent ensemble sur le même code
   - Revue de code continue et transfert de connaissances

3. **Pull Requests / Merge Requests** :
   - Processus formel avant de fusionner les changements
   - Utiliser les outils de CI pour automatiser certaines vérifications

### Intégration continue et qualité de code

Intégrez des outils d'analyse de code dans votre pipeline CI :

```yaml
# Exemple de configuration Azure DevOps
trigger:
- main

pool:
  vmImage: 'windows-latest'

steps:
- task: DotNetCoreCLI@2
  displayName: 'Build'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration Release'

- task: DotNetCoreCLI@2
  displayName: 'Run Tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration Release --collect:"XPlat Code Coverage"'

- task: DotNetCoreCLI@2
  displayName: 'Run Code Analysis'
  inputs:
    command: 'custom'
    custom: 'tool'
    arguments: 'run roslynator analyze'

- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
```

### Documentation vivante

Maintenez une documentation qui évolue avec le code :

- **Tests comme documentation** :
  - Les tests unitaires documentent le comportement attendu
  - Les tests d'intégration documentent les interactions entre composants

- **Documentation en code** :
  - Commentaires XML sur les classes et méthodes publiques
  - Génération automatique de documentation (DocFX, Sandcastle)

- **Exemples de code** :
  - Fournir des exemples pratiques d'utilisation
  - Mettre à jour les exemples à chaque changement d'API

## Ressources pour approfondir

### Livres recommandés

- "Refactoring: Improving the Design of Existing Code" par Martin Fowler
- "Clean Code: A Handbook of Agile Software Craftsmanship" par Robert C. Martin
- "Working Effectively with Legacy Code" par Michael Feathers

### Outils en ligne

- [Sonar Cloud](https://sonarcloud.io/) - Analyse de code en ligne
- [NDepend](https://www.ndepend.com/) - Analyse avancée pour .NET
- [CodeScene](https://codescene.io/) - Analyse de la dette technique

### Formations et certifications

- Pluralsight - Cours sur le Clean Code et le refactoring
- Certification Microsoft - "MCSD: App Builder"
- Certifications Scrum - Pour les aspects organisationnels de la qualité de code

## Conclusion

L'analyse de code et le refactoring sont des compétences fondamentales pour tout développeur C#. En pratiquant régulièrement ces techniques, vous améliorerez progressivement la qualité de votre code, réduirez la dette technique et faciliterez l'évolution de vos applications.

Rappelez-vous que la qualité du code n'est pas un objectif ponctuel mais un processus continu. Prenez l'habitude d'analyser et de refactoriser votre code régulièrement, et vous constaterez rapidement des améliorations dans la maintenabilité et l'évolutivité de vos projets.

⏭️
