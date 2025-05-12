# 5.8. Modificateurs d'accès

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les modificateurs d'accès en C# sont des mots-clés qui déterminent la visibilité et l'accessibilité des types (classes, structs, interfaces) et de leurs membres (méthodes, propriétés, champs) dans votre application. Un choix judicieux des modificateurs d'accès est essentiel pour créer un code sécurisé, maintenable et qui respecte le principe d'encapsulation de la programmation orientée objet.

## 5.8.1. public, private, protected, internal

C# propose cinq modificateurs d'accès de base et une combinaison, chacun avec un niveau de visibilité différent.

### Public

Le modificateur `public` offre le niveau d'accès le plus large. Les membres `public` sont accessibles sans restriction, depuis n'importe quelle partie du code.

```csharp
public class Utilisateur
{
    // Accessible de partout
    public string Nom { get; set; }

    // Méthode publique
    public void AfficherInfos()
    {
        Console.WriteLine($"Utilisateur: {Nom}, ID: {id}");
    }

    // Membre privé
    private int id;
}

public class Program
{
    public static void Main()
    {
        Utilisateur user = new Utilisateur();
        user.Nom = "Jean Dupont";   // OK - la propriété est publique
        user.AfficherInfos();       // OK - la méthode est publique
        // user.id = 123;           // Erreur - le champ est privé
    }
}
```


### Private

Le modificateur `private` offre le niveau d'accès le plus restrictif. Les membres `private` sont accessibles uniquement à l'intérieur de la classe ou du struct qui les contient.

```csharp
public class CompteBancaire
{
    // Champs privés - accessibles uniquement à l'intérieur de la classe
    private decimal solde;
    private string numeroCompte;
    private readonly decimal fraisTransaction = 1.5m;

    // Constructeur public
    public CompteBancaire(string numero, decimal soldeInitial)
    {
        numeroCompte = numero;
        solde = soldeInitial;
    }

    // Propriété en lecture seule exposant un champ privé
    public decimal Solde
    {
        get { return solde; }
    }

    // Méthodes publiques qui manipulent les champs privés
    public void Deposer(decimal montant)
    {
        if (montant <= 0)
        {
            throw new ArgumentException("Le montant doit être positif");
        }

        solde += montant;
        EnregistrerTransaction("Dépôt", montant);
    }

    public bool Retirer(decimal montant)
    {
        if (montant <= 0)
        {
            throw new ArgumentException("Le montant doit être positif");
        }

        if (solde >= montant + fraisTransaction)
        {
            solde -= (montant + fraisTransaction);
            EnregistrerTransaction("Retrait", montant);
            return true;
        }

        return false;
    }

    // Méthode privée - uniquement utilisée en interne
    private void EnregistrerTransaction(string type, decimal montant)
    {
        Console.WriteLine($"Transaction: {type} de {montant}€ sur le compte {numeroCompte}");
        Console.WriteLine($"Frais: {fraisTransaction}€");
        Console.WriteLine($"Nouveau solde: {solde}€");
    }
}

public class Program
{
    public static void Main()
    {
        CompteBancaire compte = new CompteBancaire("FR7630001007941234567890185", 1000);

        Console.WriteLine($"Solde initial: {compte.Solde}€");

        compte.Deposer(500);
        compte.Retirer(200);

        // Tentatives d'accès incorrectes
        // compte.solde = 2000;                // Erreur - 'solde' est privé
        // compte.EnregistrerTransaction(...); // Erreur - la méthode est privée
        // compte.fraisTransaction = 0;        // Erreur - le champ est privé et readonly
    }
}
```


### Protected

Le modificateur `protected` permet l'accès aux membres à l'intérieur de la classe elle-même et à toutes les classes qui en héritent, même si elles sont dans un autre assembly.

```csharp
public class Vehicule
{
    // Champs protected - accessibles dans cette classe et les classes dérivées
    protected string marque;
    protected string modele;
    protected int annee;

    // Constructeur public
    public Vehicule(string marque, string modele, int annee)
    {
        this.marque = marque;
        this.modele = modele;
        this.annee = annee;
    }

    // Méthode protected - accessible dans cette classe et les classes dérivées
    protected virtual string ObtenirSpecifications()
    {
        return $"{marque} {modele} ({annee})";
    }

    // Méthode publique qui utilise une méthode protected
    public void AfficherInfos()
    {
        Console.WriteLine($"Véhicule: {ObtenirSpecifications()}");
    }
}

// Classe dérivée
public class Voiture : Vehicule
{
    private int nombrePortes;

    public Voiture(string marque, string modele, int annee, int nombrePortes)
        : base(marque, modele, annee)
    {
        this.nombrePortes = nombrePortes;
    }

    // Surcharge d'une méthode protected
    protected override string ObtenirSpecifications()
    {
        // Accès aux champs protected de la classe de base
        return $"{marque} {modele} ({annee}) - {nombrePortes} portes";
    }

    // Méthode publique qui utilise des membres protected
    public void AfficherDetailsVoiture()
    {
        Console.WriteLine($"Voiture: {ObtenirSpecifications()}");
        Console.WriteLine($"Fabricant: {marque}"); // Accès au champ protected de la classe de base
    }
}

public class Program
{
    public static void Main()
    {
        Vehicule vehicule = new Vehicule("Generique", "Base", 2023);
        vehicule.AfficherInfos();

        Voiture voiture = new Voiture("Renault", "Clio", 2023, 5);
        voiture.AfficherInfos();
        voiture.AfficherDetailsVoiture();

        // Tentatives d'accès incorrectes
        // vehicule.marque = "Autre";                // Erreur - 'marque' est protected
        // vehicule.ObtenirSpecifications();         // Erreur - la méthode est protected
    }
}
```


### Internal

Le modificateur `internal` limite l'accès aux membres au sein du même assembly. Les membres `internal` sont accessibles à toutes les classes du même projet ou assembly, mais pas depuis d'autres assemblies (DLL ou EXE).

```csharp
// Dans un fichier du projet/assembly principal
namespace MonApplication.Core
{
    // Classe internal - accessible uniquement dans cet assembly
    internal class ConfigurationInterne
    {
        public string CheminBaseDonnees { get; set; }
        public int PortServeur { get; set; }

        public void ChargerConfiguration()
        {
            Console.WriteLine("Chargement de la configuration interne...");
        }
    }

    // Classe publique avec membres internal
    public class GestionnaireApplication
    {
        // Propriété internal - accessible uniquement dans cet assembly
        internal bool EstInitialise { get; private set; }

        // Méthode internal - accessible uniquement dans cet assembly
        internal void Initialiser()
        {
            Console.WriteLine("Initialisation du gestionnaire...");
            EstInitialise = true;
        }

        // Méthode publique
        public void Demarrer()
        {
            if (!EstInitialise)
            {
                Initialiser(); // Appel à une méthode internal
            }

            Console.WriteLine("Application démarrée");
        }
    }

    // Classe publique qui utilise des membres internal
    public class ComposantApplicatif
    {
        public void Executer()
        {
            // Accès à une classe internal dans le même assembly
            ConfigurationInterne config = new ConfigurationInterne();
            config.CheminBaseDonnees = "Data Source=localhost;";
            config.ChargerConfiguration();

            // Accès à des membres internal dans le même assembly
            GestionnaireApplication gestionnaire = new GestionnaireApplication();
            gestionnaire.Initialiser(); // OK - même assembly

            if (gestionnaire.EstInitialise) // OK - même assembly
            {
                Console.WriteLine("Gestionnaire initialisé, exécution du composant...");
            }
        }
    }
}

// Dans un autre assembly (projet référençant le premier)
namespace MonApplication.Client
{
    using MonApplication.Core;

    public class Program
    {
        public static void Main()
        {
            // Accessible car public
            GestionnaireApplication gestionnaire = new GestionnaireApplication();
            gestionnaire.Demarrer(); // OK - méthode publique

            // Tentatives d'accès incorrectes
            // ConfigurationInterne config = new ConfigurationInterne(); // Erreur - la classe est internal
            // gestionnaire.Initialiser();                               // Erreur - la méthode est internal
            // bool init = gestionnaire.EstInitialise;                   // Erreur - la propriété est internal

            // Accessible car public
            ComposantApplicatif composant = new ComposantApplicatif();
            composant.Executer(); // OK - méthode publique
        }
    }
}
```


### Private Protected (depuis C# 7.2)

Le modificateur `private protected` limite l'accès aux classes dérivées, mais uniquement dans le même assembly. C'est une combinaison de `private` et `protected`.

```csharp
// Dans l'assembly principal
namespace MonApplication.Core
{
    public class ClasseBase
    {
        // Accessible uniquement par cette classe et les classes dérivées dans cet assembly
        private protected string donneesSensibles;

        public ClasseBase(string donnees)
        {
            donneesSensibles = donnees;
        }

        // Méthode private protected
        private protected virtual void TraiterDonneesSensibles()
        {
            Console.WriteLine($"Traitement de: {donneesSensibles}");
        }
    }

    // Classe dérivée dans le même assembly
    public class ClasseDerivee : ClasseBase
    {
        public ClasseDerivee(string donnees) : base(donnees)
        {
        }

        public void TraiterDonnees()
        {
            // Accès au champ private protected de la classe de base
            Console.WriteLine($"Accès aux données: {donneesSensibles}");

            // Appel de la méthode private protected
            TraiterDonneesSensibles();
        }

        // Surcharge d'une méthode private protected
        private protected override void TraiterDonneesSensibles()
        {
            Console.WriteLine($"Traitement spécialisé de: {donneesSensibles}");
        }
    }
}

// Dans un autre assembly
namespace MonApplication.Client
{
    using MonApplication.Core;

    // Classe dérivée dans un autre assembly
    public class ClasseDeriveExterne : ClasseBase
    {
        public ClasseDeriveExterne(string donnees) : base(donnees)
        {
        }

        public void TraiterDonnees()
        {
            // Tentatives d'accès incorrectes
            // Console.WriteLine(donneesSensibles);     // Erreur - private protected est limité à l'assembly original
            // TraiterDonneesSensibles();               // Erreur - private protected est limité à l'assembly original
        }
    }
}
```


### Protected Internal

Le modificateur `protected internal` permet l'accès aux membres à toutes les classes dans le même assembly et à toutes les classes dérivées, y compris celles qui sont dans d'autres assemblies.

```csharp
// Dans l'assembly principal
namespace MonApplication.Framework
{
    public class ComposantBase
    {
        // Accessible dans l'assembly courant et par toutes les classes dérivées
        protected internal string Configuration { get; set; }

        // Méthode protected internal
        protected internal virtual void Configurer()
        {
            Console.WriteLine($"Configuration du composant: {Configuration}");
        }
    }

    // Autre classe dans le même assembly (pas de dérivation)
    public class Utilitaire
    {
        public void ConfigurerComposant(ComposantBase composant)
        {
            // Accès aux membres protected internal (car même assembly)
            composant.Configuration = "Configuration par défaut";
            composant.Configurer();
        }
    }
}

// Dans un autre assembly
namespace MonApplication.Extension
{
    using MonApplication.Framework;

    // Classe dérivée dans un autre assembly
    public class ComposantEtendu : ComposantBase
    {
        public void Initialiser()
        {
            // Accès aux membres protected internal depuis une classe dérivée
            Configuration = "Configuration étendue";
            Configurer();
        }

        // Surcharge d'une méthode protected internal
        protected internal override void Configurer()
        {
            Console.WriteLine($"Configuration avancée: {Configuration}");
        }
    }

    // Classe non dérivée dans un autre assembly
    public class GestionnaireComposants
    {
        public void Utiliser(ComposantBase composant)
        {
            // Tentatives d'accès incorrectes
            // composant.Configuration = "Autre config";     // Erreur - accessible uniquement dans l'assembly original ou les classes dérivées
            // composant.Configurer();                       // Erreur - accessible uniquement dans l'assembly original ou les classes dérivées

            // Mais on peut utiliser un composant dérivé
            ComposantEtendu composantEtendu = new ComposantEtendu();
            composantEtendu.Initialiser(); // OK - cette méthode a accès aux membres protected internal
        }
    }
}
```


## 5.8.2. Combinaisons et meilleures pratiques

### Tableau comparatif des modificateurs d'accès

| Modificateur       | Même classe | Classes dérivées (même assembly) | Classes non dérivées (même assembly) | Classes dérivées (autre assembly) | Classes non dérivées (autre assembly) |
|--------------------|-------------|----------------------------------|--------------------------------------|-----------------------------------|---------------------------------------|
| private            | ✓           | ✗                                | ✗                                    | ✗                                 | ✗                                     |
| protected          | ✓           | ✓                                | ✗                                    | ✓                                 | ✗                                     |
| internal           | ✓           | ✓                                | ✓                                    | ✗                                 | ✗                                     |
| protected internal | ✓           | ✓                                | ✓                                    | ✓                                 | ✗                                     |
| private protected  | ✓           | ✓                                | ✗                                    | ✗                                 | ✗                                     |
| public             | ✓           | ✓                                | ✓                                    | ✓                                 | ✓                                     |

### Meilleures pratiques

Voici quelques bonnes pratiques pour l'utilisation des modificateurs d'accès :

```csharp
public class BonnesPratiques
{
    // 1. Déclarez les champs comme privés et utilisez des propriétés pour les exposer
    private string _nom;

    public string Nom
    {
        get { return _nom; }
        set { _nom = value; }
    }

    // 2. Utilisez readonly pour les champs qui ne changent pas après l'initialisation
    private readonly string _identifiant;

    // 3. Utilisez protected pour les membres destinés à être utilisés par les classes dérivées
    protected virtual void InitialiserDonnees()
    {
        // Implémentation
    }

    // 4. Utilisez internal pour limiter l'accès au sein d'un seul assembly
    internal void MethodeInterne()
    {
        // Implémentation réservée à l'assembly
    }

    // 5. Rendez les constructeurs publics uniquement si nécessaire
    private BonnesPratiques()
    {
        _identifiant = Guid.NewGuid().ToString();
    }

    // Constructeur public qui appelle le constructeur privé
    public BonnesPratiques(string nom) : this()
    {
        _nom = nom;
    }

    // 6. Utilisez des méthodes publiques pour les API stables
    public string ObtenirInfosComplet()
    {
        return $"ID: {_identifiant}, Nom: {_nom}";
    }
}

// 7. Utilisez les classes sealed pour empêcher l'héritage quand ce n'est pas nécessaire
public sealed class ClasseNonDerivable
{
    // Implémentation
}

// 8. Utilisez internal pour les classes d'implémentation non publiques
internal class ImplementationInterne
{
    // Détails d'implémentation cachés du monde extérieur
}
```


### Exemples de conception avec différents modificateurs

```csharp
namespace ExempleApplication.Core
{
    // Classe de base publique avec une API soigneusement conçue
    public abstract class ServiceBase
    {
        // Champs privés
        private readonly string _serviceId;
        private bool _estInitialise;

        // État protégé partagé avec les classes dérivées
        protected Dictionary<string, object> Configuration { get; private set; }

        // Constructeur protégé - ne peut être appelé que par les classes dérivées
        protected ServiceBase(string serviceId)
        {
            _serviceId = serviceId;
            Configuration = new Dictionary<string, object>();
        }

        // Méthode publique - partie de l'API stable
        public void Initialiser()
        {
            if (_estInitialise)
                return;

            // Appel au Template Method pattern
            InitialiserService();
            _estInitialise = true;

            // Journalisation d'audit interne
            JournaliserEvenement($"Service {_serviceId} initialisé");
        }

        // Méthode abstraite protégée - à implémenter par les classes dérivées
        protected abstract void InitialiserService();

        // Méthode protégée virtuelle - peut être surchargée, mais a une implémentation par défaut
        protected virtual void JournaliserEvenement(string message)
        {
            Console.WriteLine($"[{DateTime.Now}] {message}");
        }

        // Méthode interne - accessible dans l'assembly mais pas à l'extérieur
        internal void ReinitialiserParAdmin()
        {
            _estInitialise = false;
            Configuration.Clear();
            Console.WriteLine($"Service {_serviceId} réinitialisé par un administrateur");
        }

        // Méthode private protected - accessible uniquement par les classes dérivées dans cet assembly
        private protected void DefinirEtatCritique(string raison)
        {
            Console.WriteLine($"CRITIQUE: Service {_serviceId} en état critique: {raison}");
            // Logique spécifique à l'implémentation interne
        }
    }

    // Implémentation concrète
    public class ServiceMessagerie : ServiceBase
    {
        private string _adresseServeur;

        public ServiceMessagerie(string adresseServeur)
            : base("MSG-" + Guid.NewGuid().ToString().Substring(0, 8))
        {
            _adresseServeur = adresseServeur;
        }

        // Implémentation d'une méthode abstraite protégée
        protected override void InitialiserService()
        {
            Configuration["serveur"] = _adresseServeur;
            Configuration["timeout"] = 30000;
            Configuration["retries"] = 3;

            Console.WriteLine($"Service de messagerie initialisé sur {_adresseServeur}");
        }

        // Méthode publique spécifique à cette implémentation
        public void EnvoyerMessage(string destinataire, string contenu)
        {
            if (!Configuration.ContainsKey("serveur"))
            {
                // Utilisation de la méthode private protected
                DefinirEtatCritique("Configuration du serveur manquante");
                return;
            }

            Console.WriteLine($"Envoi à {destinataire}: {contenu}");
        }

        // Surcharge d'une méthode protégée
        protected override void JournaliserEvenement(string message)
        {
            // Ajoute plus de détails que l'implémentation de base
            base.JournaliserEvenement($"[MESSAGERIE] {message}");
        }
    }

    // Utilitaire interne dans le même assembly
    internal class AdminServices
    {
        public void ReinitialiserTousServices(List<ServiceBase> services)
        {
            foreach (var service in services)
            {
                // Peut appeler des méthodes internal
                service.ReinitialiserParAdmin();
            }
        }
    }
}
```


## 5.8.3. Contrôle de la visibilité

Le contrôle de la visibilité est un aspect important de la conception des API et du respect des principes d'encapsulation. Voici quelques techniques avancées pour gérer la visibilité de votre code.

### Visibilité asymétrique dans les propriétés

```csharp
public class Utilisateur
{
    // Propriété avec accesseur public en lecture et privé en écriture
    public string Nom { get; private set; }

    // Propriété avec accesseur public en lecture et protected en écriture
    public DateTime DateInscription { get; protected set; }

    // Propriété avec accesseur public en lecture et internal en écriture
    public int NombreConnexions { get; internal set; }

    public Utilisateur(string nom)
    {
        Nom = nom;
        DateInscription = DateTime.Now;
        NombreConnexions = 0;
    }

    // Méthode pour changer le nom (car set est private)
    public void ChangerNom(string nouveauNom, string motDePasse)
    {
        if (ValiderMotDePasse(motDePasse))
        {
            Nom = nouveauNom;
        }
    }

    private bool ValiderMotDePasse(string motDePasse)
    {
        // Logique de validation
        return motDePasse == "motdepasse"; // Simplifié pour l'exemple
    }
}

// Classe dérivée
public class UtilisateurPremium : Utilisateur
{
    public UtilisateurPremium(string nom) : base(nom)
    {
        // Peut modifier DateInscription car son setter est protected
        DateInscription = DateTime.Now.AddMonths(-1); // Comme s'il était membre depuis plus longtemps
    }
}

// Classe dans le même assembly
public class GestionnaireUtilisateurs
{
    public void EnregistrerConnexion(Utilisateur utilisateur)
    {
        // Peut modifier NombreConnexions car son setter est internal
        utilisateur.NombreConnexions++;
    }
}
```


### Visibilité hiérarchique avec les classes imbriquées

```csharp
public class BaseDonnees
{
    // Champ privé
    private string connectionString;

    // Constructeur public
    public BaseDonnees(string connectionString)
    {
        this.connectionString = connectionString;
    }

    // Classe imbriquée avec accès aux membres privés de la classe englobante
    public class Transaction
    {
        private BaseDonnees db;
        private bool estActive;

        internal Transaction(BaseDonnees database)
        {
            db = database;
            estActive = true;
            Console.WriteLine($"Transaction démarrée sur {db.connectionString}");
        }

        public void Valider()
        {
            if (estActive)
            {
                Console.WriteLine("Transaction validée");
                estActive = false;
            }
        }

        public void Annuler()
        {
            if (estActive)
            {
                Console.WriteLine("Transaction annulée");
                estActive = false;
            }
        }
    }

    // Classe imbriquée privée - uniquement accessible dans BaseDonnees
    private class Journal
    {
        public static void Enregistrer(string message)
        {
            Console.WriteLine($"Journal: {message}");
        }
    }

    // Méthode publique qui crée une instance de la classe imbriquée
    public Transaction CommencerTransaction()
    {
        Journal.Enregistrer("Début transaction");
        return new Transaction(this);
    }
}

public class Program
{
    public static void Main()
    {
        BaseDonnees db = new BaseDonnees("Server=localhost;Database=test");

        // Utilisation de la classe imbriquée
        BaseDonnees.Transaction transaction = db.CommencerTransaction();
        transaction.Valider();

        // Impossible d'accéder à la classe imbriquée privée
        // BaseDonnees.Journal.Enregistrer("Test");  // Erreur - Journal est privé

        // Impossible de créer directement une Transaction
        // BaseDonnees.Transaction t = new BaseDonnees.Transaction(db);  // Erreur - constructeur est internal
    }
}
```


### Assemblies à accès interne et attributs InternalsVisibleTo

Vous pouvez utiliser l'attribut `InternalsVisibleTo` pour permettre à un assembly spécifique d'accéder aux membres `internal` d'un autre assembly, ce qui est utile pour les tests unitaires.

```csharp
// Dans l'assembly principal (Exemple.Core)
using System.Runtime.CompilerServices;

// Permet à l'assembly de tests d'accéder aux membres internes
[assembly: InternalsVisibleTo("Exemple.Tests")]

namespace Exemple.Core
{
    // Classe publique avec membres internes
    public class CalculateurAvance
    {
        // Méthode interne
        internal static double CalculerTauxIntermediaire(double base1, double base2)
        {
            return (base1 * 0.4) + (base2 * 0.6);
        }

        // Classe interne
        internal class ParametresCalcul
        {
            public double Multiplicateur { get; set; }
            public int FacteurAjustement { get; set; }
        }

        // Méthode publique qui utilise des membres internes
        public double CalculerResultat(double valeur1, double valeur2)
        {
            double taux = CalculerTauxIntermediaire(valeur1, valeur2);

            ParametresCalcul parametres = new ParametresCalcul
            {
                Multiplicateur = 1.5,
                FacteurAjustement = 3
            };

            return taux * parametres.Multiplicateur + parametres.FacteurAjustement;
        }
    }
}

// Dans l'assembly de test (Exemple.Tests)
using Exemple.Core;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Exemple.Tests
{
    [TestClass]
    public class CalculateurTests
    {
        [TestMethod]
        public void TestCalculTauxIntermediaire()
        {
            // Accès à la méthode interne grâce à InternalsVisibleTo
            double resultat = CalculateurAvance.CalculerTauxIntermediaire(10, 20);
            Assert.AreEqual(16, resultat);
        }

        [TestMethod]
        public void TestParametresCalcul()
        {
            // Accès à la classe interne grâce à InternalsVisibleTo
            CalculateurAvance.ParametresCalcul parametres = new CalculateurAvance.ParametresCalcul
            {
                Multiplicateur = 2.0,
                FacteurAjustement = 5
            };

            Assert.AreEqual(2.0, parametres.Multiplicateur);
            Assert.AreEqual(5, parametres.FacteurAjustement);
        }
    }
}
```


### Conseils pour une bonne gestion de la visibilité

1. **Cachez l'implémentation, exposez l'API** : Rendez privés tous les détails d'implémentation et n'exposez que les membres nécessaires.

2. **Préférez une visibilité restreinte par défaut** : Commencez avec le modificateur d'accès le plus restrictif possible, puis élargissez uniquement si nécessaire.

3. **Utilisez protected avec parcimonie** : Ne rendez protected que les membres qui sont vraiment destinés à être utilisés ou remplacés par les classes dérivées.

4. **Utilisez internal pour les composants à l'échelle de l'assembly** : Cela permet de garder une API propre tout en permettant la collaboration entre classes d'un même assembly.

5. **Utilisez private protected pour un contrôle précis de l'héritage** : Idéal pour les cadres d'application où vous voulez contrôler l'extension dans le même assembly.

6. **Documentez le contrat public** : Assurez-vous que tous les membres publics sont bien documentés et représentent une API stable.

7. **Pensez à la maintenance à long terme** : Une fois qu'une API est publique, il est difficile de la changer sans casser la compatibilité.

Les modificateurs d'accès sont des outils puissants pour créer des APIs robustes et maintenir l'encapsulation. Une bonne utilisation de ces modificateurs rend votre code plus sûr, plus maintenable et plus facile à comprendre.

⏭️
