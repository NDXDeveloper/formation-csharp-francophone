# 5.7. Classes abstraites

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les classes abstraites sont un concept fondamental de la programmation orientée objet en C#. Elles permettent de définir un modèle pour d'autres classes tout en fournissant certaines implémentations communes, tout en imposant que certaines méthodes soient implémentées par les classes dérivées.

## 5.7.1. Différence avec les interfaces

Tandis que les interfaces définissent uniquement un contrat (quoi faire), les classes abstraites peuvent fournir à la fois un contrat et une implémentation partielle (quoi faire et comment le faire).

```csharp
// Interface - définit uniquement un contrat
public interface IVehicule
{
    void Demarrer();
    void Arreter();
    double CalculerConsommation(double distance);
}

// Classe abstraite - peut combiner contrat et implémentation
public abstract class VehiculeBase
{
    // Propriétés implémentées
    public string Marque { get; set; }
    public string Modele { get; set; }
    public int AnneeProduction { get; set; }

    // Constructeur - les classes abstraites peuvent avoir des constructeurs
    protected VehiculeBase(string marque, string modele, int anneeProduction)
    {
        Marque = marque;
        Modele = modele;
        AnneeProduction = anneeProduction;
    }

    // Méthode abstraite - doit être implémentée par les classes dérivées
    public abstract void Demarrer();

    // Méthode abstraite - doit être implémentée par les classes dérivées
    public abstract void Arreter();

    // Méthode concrète - fournit une implémentation par défaut
    public virtual double CalculerConsommation(double distance)
    {
        return distance * 0.1; // Implémentation par défaut
    }

    // Méthode concrète - non virtual, ne peut pas être remplacée
    public string ObtenirDescription()
    {
        return $"{Marque} {Modele} ({AnneeProduction})";
    }
}

// Implémentation concrète d'une classe abstraite
public class Voiture : VehiculeBase
{
    public double TailleMoteur { get; set; }

    // Les classes concrètes doivent appeler un constructeur de la classe de base
    public Voiture(string marque, string modele, int anneeProduction, double tailleMoteur)
        : base(marque, modele, anneeProduction)
    {
        TailleMoteur = tailleMoteur;
    }

    // Implémentation obligatoire des méthodes abstraites
    public override void Demarrer()
    {
        Console.WriteLine($"La voiture {Marque} {Modele} démarre.");
    }

    public override void Arreter()
    {
        Console.WriteLine($"La voiture {Marque} {Modele} s'arrête.");
    }

    // Surcharge de la méthode virtuelle pour personnaliser le comportement
    public override double CalculerConsommation(double distance)
    {
        // Consommation basée sur la taille du moteur
        return distance * (TailleMoteur / 20);
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        // VehiculeBase vehicule = new VehiculeBase(); // Erreur de compilation - impossible d'instancier une classe abstraite

        Voiture maVoiture = new Voiture("Toyota", "Corolla", 2022, 1.8);
        maVoiture.Demarrer();
        Console.WriteLine($"Description: {maVoiture.ObtenirDescription()}");
        double consommation = maVoiture.CalculerConsommation(100);
        Console.WriteLine($"Consommation pour 100 km: {consommation} litres");
        maVoiture.Arreter();
    }
}
```


### Principales différences entre interfaces et classes abstraites :

| Caractéristique | Classe abstraite | Interface |
|-----------------|------------------|-----------|
| Instanciation | Ne peut pas être instanciée directement | Ne peut pas être instanciée |
| Héritage | Une classe ne peut hériter que d'une seule classe abstraite | Une classe peut implémenter plusieurs interfaces |
| Constructeur | Peut avoir des constructeurs | Ne peut pas avoir de constructeurs |
| Accès aux membres | Peut contenir des membres private, protected, etc. | Tous les membres sont implicitement public (avant C# 8) |
| État | Peut contenir des champs pour stocker un état | Ne peut pas contenir de champs (uniquement des propriétés) |
| Implémentation | Peut fournir une implémentation complète, partielle ou aucune | Avant C# 8, ne peut pas fournir d'implémentation |
| Méthodes abstraites | Peut contenir des méthodes abstraites | Toutes les méthodes sont implicitement abstraites (avant C# 8) |

## 5.7.2. Méthodes abstraites

Les méthodes abstraites sont des méthodes déclarées sans implémentation qui doivent être implémentées par les classes dérivées. Elles définissent la signature et le contrat sans spécifier le comportement.

```csharp
// Classe abstraite avec méthodes abstraites
public abstract class Forme
{
    // Propriétés concrètes
    public string Couleur { get; set; }

    // Constructeur
    protected Forme(string couleur)
    {
        Couleur = couleur;
    }

    // Méthode abstraite - aucune implémentation fournie
    public abstract double CalculerSurface();

    // Méthode abstraite - aucune implémentation fournie
    public abstract double CalculerPerimetre();

    // Méthode concrète qui utilise des méthodes abstraites
    public string ObtenirInformations()
    {
        return $"Forme de couleur {Couleur}:\n" +
               $"- Surface: {CalculerSurface()} unités²\n" +
               $"- Périmètre: {CalculerPerimetre()} unités";
    }
}

// Classe concrète implémentant les méthodes abstraites
public class Cercle : Forme
{
    public double Rayon { get; set; }

    public Cercle(string couleur, double rayon) : base(couleur)
    {
        Rayon = rayon;
    }

    // Implémentation de la méthode abstraite
    public override double CalculerSurface()
    {
        return Math.PI * Rayon * Rayon;
    }

    // Implémentation de la méthode abstraite
    public override double CalculerPerimetre()
    {
        return 2 * Math.PI * Rayon;
    }
}

// Autre classe concrète implémentant les méthodes abstraites
public class Rectangle : Forme
{
    public double Largeur { get; set; }
    public double Hauteur { get; set; }

    public Rectangle(string couleur, double largeur, double hauteur) : base(couleur)
    {
        Largeur = largeur;
        Hauteur = hauteur;
    }

    // Implémentation de la méthode abstraite
    public override double CalculerSurface()
    {
        return Largeur * Hauteur;
    }

    // Implémentation de la méthode abstraite
    public override double CalculerPerimetre()
    {
        return 2 * (Largeur + Hauteur);
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        Cercle cercle = new Cercle("Rouge", 5);
        Console.WriteLine(cercle.ObtenirInformations());

        Rectangle rectangle = new Rectangle("Bleu", 4, 6);
        Console.WriteLine(rectangle.ObtenirInformations());

        // Utilisation polymorphique
        Forme[] formes = { cercle, rectangle };

        foreach (var forme in formes)
        {
            Console.WriteLine($"Forme: {forme.GetType().Name}");
            Console.WriteLine($"Surface: {forme.CalculerSurface()}");
            Console.WriteLine($"Périmètre: {forme.CalculerPerimetre()}");
            Console.WriteLine();
        }
    }
}
```


### Points clés sur les méthodes abstraites :

1. Elles sont déclarées avec le mot-clé `abstract` et n'ont pas de corps (pas d'accolades, juste un point-virgule).
2. Elles sont implicitement virtuelles (pas besoin du mot-clé `virtual`).
3. Elles doivent être implémentées par les classes dérivées non abstraites.
4. Elles ne peuvent exister que dans des classes abstraites.
5. Lors de l'implémentation, on utilise le mot-clé `override`.

## 5.7.3. Implémentation partielle

Une classe abstraite peut combiner des implémentations complètes, des implémentations partielles et des méthodes purement abstraites pour créer un modèle flexible pour les classes dérivées.

```csharp
// Classe abstraite avec implémentation partielle
public abstract class GestionnaireDocuments
{
    // État commun
    protected List<string> documents;
    protected string repertoire;

    // Constructeur
    protected GestionnaireDocuments(string repertoire)
    {
        this.repertoire = repertoire;
        this.documents = new List<string>();
        ChargerDocuments();
    }

    // Méthode concrète commune à toutes les classes dérivées
    public void AjouterDocument(string nom)
    {
        if (!documents.Contains(nom))
        {
            documents.Add(nom);
            Console.WriteLine($"Document '{nom}' ajouté.");
            SauvegarderChangements();
        }
    }

    // Méthode concrète commune à toutes les classes dérivées
    public void SupprimerDocument(string nom)
    {
        if (documents.Contains(nom))
        {
            documents.Remove(nom);
            Console.WriteLine($"Document '{nom}' supprimé.");
            SauvegarderChangements();
        }
    }

    // Méthode concrète avec implémentation par défaut, mais modifiable (virtuelle)
    public virtual void AfficherDocuments()
    {
        Console.WriteLine($"Documents dans {repertoire}:");

        if (documents.Count == 0)
        {
            Console.WriteLine("  Aucun document.");
            return;
        }

        foreach (var doc in documents)
        {
            Console.WriteLine($"  - {doc}");
        }
    }

    // Méthode concrète appelant une méthode abstraite (Template Method pattern)
    public void RafraichirDocuments()
    {
        Console.WriteLine("Rafraîchissement des documents...");
        ChargerDocuments();
        Console.WriteLine("Rafraîchissement terminé.");
    }

    // Méthodes abstraites que les classes dérivées doivent implémenter
    protected abstract void ChargerDocuments();
    protected abstract void SauvegarderChangements();

    // Méthode abstraite avec paramètre
    public abstract bool RechercherDocument(string motCle);
}

// Implémentation pour les fichiers texte
public class GestionnaireFichiersTexte : GestionnaireDocuments
{
    public GestionnaireFichiersTexte(string repertoire) : base(repertoire)
    {
    }

    // Implémentation des méthodes abstraites
    protected override void ChargerDocuments()
    {
        Console.WriteLine($"Chargement des fichiers texte depuis {repertoire}...");

        // Dans une vraie application : charger les fichiers .txt du répertoire
        documents.Clear();
        documents.Add("document1.txt");
        documents.Add("rapport.txt");
        documents.Add("notes.txt");
    }

    protected override void SauvegarderChangements()
    {
        Console.WriteLine("Sauvegarde des changements dans le système de fichiers...");
        // Simulation de la sauvegarde
    }

    public override bool RechercherDocument(string motCle)
    {
        Console.WriteLine($"Recherche de '{motCle}' dans les fichiers texte...");

        // Simulation de recherche
        foreach (var doc in documents)
        {
            if (doc.Contains(motCle))
            {
                Console.WriteLine($"Trouvé dans : {doc}");
                return true;
            }
        }

        Console.WriteLine("Aucune correspondance trouvée.");
        return false;
    }

    // Ajout d'une fonctionnalité spécifique à cette classe
    public void AnalyserContenuTexte()
    {
        Console.WriteLine("Analyse du contenu des fichiers texte...");
        // Logique spécifique
    }
}

// Implémentation pour une base de données
public class GestionnaireDocumentsDatabase : GestionnaireDocuments
{
    private readonly string connectionString;

    public GestionnaireDocumentsDatabase(string repertoire, string connectionString) : base(repertoire)
    {
        this.connectionString = connectionString;
    }

    // Surcharge d'une méthode virtuelle pour personnaliser le comportement
    public override void AfficherDocuments()
    {
        Console.WriteLine($"Documents dans la base de données (collection: {repertoire}):");

        if (documents.Count == 0)
        {
            Console.WriteLine("  Aucun document.");
            return;
        }

        foreach (var doc in documents)
        {
            // Format différent pour l'affichage des documents de base de données
            Console.WriteLine($"  > {doc} [DB]");
        }
    }

    // Implémentation des méthodes abstraites
    protected override void ChargerDocuments()
    {
        Console.WriteLine($"Chargement des documents depuis la base de données {connectionString}...");

        // Dans une vraie application : requête SQL pour charger les documents
        documents.Clear();
        documents.Add("fiche_client.pdf");
        documents.Add("facture_2023.xlsx");
        documents.Add("contrat.docx");
    }

    protected override void SauvegarderChangements()
    {
        Console.WriteLine("Sauvegarde des changements dans la base de données...");
        // Simulation de la sauvegarde dans une base de données
    }

    public override bool RechercherDocument(string motCle)
    {
        Console.WriteLine($"Exécution d'une requête SQL pour rechercher '{motCle}'...");

        // Simulation de recherche en base de données
        foreach (var doc in documents)
        {
            if (doc.Contains(motCle))
            {
                Console.WriteLine($"Trouvé dans : {doc}");
                return true;
            }
        }

        Console.WriteLine("Aucune correspondance trouvée.");
        return false;
    }

    // Ajout d'une fonctionnalité spécifique à cette classe
    public void OptimiserBaseDeDonnees()
    {
        Console.WriteLine("Optimisation de la base de données de documents...");
        // Logique spécifique
    }
}

// Utilisation
public class Program
{
    public static void Main()
    {
        // Gestionnaire de fichiers texte
        GestionnaireFichiersTexte gestionnaireTexte = new GestionnaireFichiersTexte("/documents/texte");
        gestionnaireTexte.AfficherDocuments();
        gestionnaireTexte.RechercherDocument("rapport");
        gestionnaireTexte.AjouterDocument("nouveau.txt");
        gestionnaireTexte.AnalyserContenuTexte();

        Console.WriteLine();

        // Gestionnaire de documents en base de données
        GestionnaireDocumentsDatabase gestionnaireDB = new GestionnaireDocumentsDatabase(
            "documents", "Server=localhost;Database=DocDB");
        gestionnaireDB.AfficherDocuments();
        gestionnaireDB.RechercherDocument("facture");
        gestionnaireDB.SupprimerDocument("fiche_client.pdf");
        gestionnaireDB.OptimiserBaseDeDonnees();

        Console.WriteLine();

        // Utilisation polymorphique
        GestionnaireDocuments[] gestionnaires = { gestionnaireTexte, gestionnaireDB };

        foreach (var gestionnaire in gestionnaires)
        {
            Console.WriteLine($"Type: {gestionnaire.GetType().Name}");
            gestionnaire.AfficherDocuments();
            gestionnaire.RafraichirDocuments();
            Console.WriteLine();
        }
    }
}
```


### Points clés sur l'implémentation partielle :

1. **Template Method Pattern** : Les classes abstraites implémentent souvent ce pattern où une méthode concrète définit la structure d'un algorithme et appelle des méthodes abstraites qui sont implémentées par les sous-classes.

2. **Partage du code commun** : Les classes abstraites permettent de centraliser l'implémentation commune et de ne laisser aux classes dérivées que ce qui est spécifique à leur contexte.

3. **Combinaison de méthodes** :
   - **Méthodes concrètes** : Implémentation commune pour toutes les classes dérivées.
   - **Méthodes virtuelles** : Implémentation par défaut que les classes dérivées peuvent remplacer.
   - **Méthodes abstraites** : Déclaration sans implémentation que les classes dérivées doivent fournir.

4. **Relation "est un"** : Les classes abstraites établissent une relation "est un" forte avec leurs sous-classes, contrairement aux interfaces qui établissent une relation "peut faire".

Les classes abstraites sont idéales lorsque vous avez une hiérarchie de classes qui partagent un comportement commun, mais où chaque sous-classe doit fournir certaines implémentations spécifiques. Elles offrent une combinaison de rigidité (contrat à respecter) et de flexibilité (extension et personnalisation) qui en fait un outil puissant dans la conception orientée objet.

⏭️
