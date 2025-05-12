# 4.3. Surcharge de méthodes en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La surcharge de méthodes est une fonctionnalité puissante qui vous permet de définir plusieurs versions d'une méthode avec le même nom mais des paramètres différents. Dans cette section, nous allons explorer ce concept et apprendre comment l'utiliser efficacement.

## Qu'est-ce que la surcharge de méthodes ?

La surcharge de méthodes (ou _method overloading_ en anglais) permet de créer plusieurs méthodes portant le même nom dans une classe, à condition qu'elles se distinguent par leurs paramètres. C'est une forme de polymorphisme qui rend votre code plus intuitif et plus facile à utiliser.

Exemple simple :

```csharp
public class Calculatrice
{
    // Première version pour additionner deux entiers
    public int Additionner(int a, int b)
    {
        return a + b;
    }

    // Deuxième version pour additionner trois entiers
    public int Additionner(int a, int b, int c)
    {
        return a + b + c;
    }

    // Troisième version pour additionner deux doubles
    public double Additionner(double a, double b)
    {
        return a + b;
    }
}
```

Avec cette classe, vous pouvez appeler la méthode `Additionner` de différentes façons :

```csharp
Calculatrice calc = new Calculatrice();

int somme1 = calc.Additionner(5, 7);              // Utilise la première version
int somme2 = calc.Additionner(5, 7, 9);           // Utilise la deuxième version
double somme3 = calc.Additionner(5.5, 7.3);       // Utilise la troisième version
```

Le compilateur détermine automatiquement quelle version de la méthode appeler en fonction des arguments fournis.

## Avantages de la surcharge de méthodes

1. **Meilleure lisibilité** : Utiliser le même nom pour des opérations similaires rend le code plus intuitif
2. **Simplicité d'utilisation** : Les utilisateurs de votre classe n'ont pas besoin de mémoriser plusieurs noms de méthodes
3. **Flexibilité** : Permet de traiter différents types ou nombres de paramètres tout en conservant une interface cohérente
4. **Évolution facile** : Possibilité d'ajouter de nouvelles versions sans casser le code existant

## 4.3.1. Règles de résolution de surcharge

Pour que la surcharge fonctionne correctement, le compilateur doit pouvoir déterminer sans ambiguïté quelle version de la méthode appeler. Voici les règles qui régissent ce processus de résolution.

### Comment le compilateur choisit la bonne méthode

Le compilateur C# applique un ensemble de règles pour déterminer quelle méthode surchargée utiliser :

1. **Correspondance exacte** : Le compilateur recherche d'abord une méthode dont les paramètres correspondent exactement aux types des arguments fournis
2. **Conversions implicites** : Si aucune correspondance exacte n'est trouvée, le compilateur cherche des méthodes qui peuvent être appelées après des conversions implicites (sans perte de données)
3. **Conversions explicites** : En dernier recours, le compilateur considère les méthodes qui nécessitent des conversions explicites

### Différences qui permettent la surcharge

Une méthode surchargée doit se différencier des autres par au moins un de ces éléments :

1. **Nombre de paramètres**
2. **Types des paramètres**
3. **Ordre des paramètres**
4. **Modificateurs de paramètres** (`ref`, `out`, ou `in`)

```csharp
public class Exemple
{
    // Différent nombre de paramètres
    public void Afficher(string message) { /* ... */ }
    public void Afficher(string message, bool enGras) { /* ... */ }

    // Différents types de paramètres
    public void Traiter(int nombre) { /* ... */ }
    public void Traiter(string texte) { /* ... */ }

    // Différent ordre de paramètres
    public void Configurer(string nom, int valeur) { /* ... */ }
    public void Configurer(int valeur, string nom) { /* ... */ }

    // Différents modificateurs
    public void Modifier(int valeur) { /* ... */ }
    public void Modifier(ref int valeur) { /* ... */ }
}
```

### Éléments qui ne permettent PAS la surcharge

Les éléments suivants ne sont pas considérés par le compilateur pour distinguer les méthodes surchargées :

1. **Type de retour** : Deux méthodes qui diffèrent uniquement par leur type de retour causeront une erreur de compilation
2. **Noms des paramètres** : Deux méthodes avec les mêmes types de paramètres dans le même ordre mais des noms différents causeront une erreur
3. **Modificateurs d'accès** (`public`, `private`, etc.)
4. **Mots-clés optionnels** comme `params`

```csharp
public class ExempleIncorrect
{
    // ERREUR : Diffère uniquement par le type de retour
    public int Calculer(int a, int b) { return a + b; }
    public double Calculer(int a, int b) { return a + b; } // Ne compilera pas

    // ERREUR : Diffère uniquement par les noms des paramètres
    public void Configurer(int largeur, int hauteur) { /* ... */ }
    public void Configurer(int longueur, int epaisseur) { /* ... */ } // Ne compilera pas
}
```

### Résolution d'ambiguïté

Parfois, plusieurs surcharges peuvent sembler appropriées pour un appel donné. Dans ce cas, le compilateur choisit la "meilleure correspondance" selon des règles de priorité complexes.

Exemple d'ambiguïté :

```csharp
public class ExempleAmbiguite
{
    public void Afficher(long nombre) { Console.WriteLine($"Version long: {nombre}"); }
    public void Afficher(double nombre) { Console.WriteLine($"Version double: {nombre}"); }
}

// Utilisation
ExempleAmbiguite ex = new ExempleAmbiguite();
ex.Afficher(5); // Quel méthode sera appelée ?
```

Dans cet exemple, un `int` (5) peut être converti implicitement en `long` ou en `double`. Le compilateur choisira la conversion vers `long` car c'est la conversion "la plus proche" (de int vers long) selon les règles de conversion numérique.

### Cas spécial : paramètres optionnels vs surcharge

Les paramètres optionnels peuvent parfois créer des situations d'ambiguïté avec les surcharges :

```csharp
public class ParamsVsSurcharge
{
    public void Exemple(int a) { Console.WriteLine($"Version 1: {a}"); }
    public void Exemple(int a, int b = 10) { Console.WriteLine($"Version 2: {a}, {b}"); }
}

// Utilisation
ParamsVsSurcharge obj = new ParamsVsSurcharge();
obj.Exemple(5); // Quelle méthode sera appelée ?
```

Dans ce cas, la première méthode sera appelée car elle correspond exactement à l'appel sans avoir besoin de paramètres optionnels.

### Résolution et méthodes génériques

Les règles deviennent encore plus complexes avec les méthodes génériques :

```csharp
public class GeneriqueVsNonGenerique
{
    public void Traiter<T>(T item) { Console.WriteLine("Version générique"); }
    public void Traiter(int item) { Console.WriteLine("Version int spécifique"); }
    public void Traiter(string item) { Console.WriteLine("Version string spécifique"); }
}

// Utilisation
GeneriqueVsNonGenerique obj = new GeneriqueVsNonGenerique();
obj.Traiter(5);          // Appelle la version int spécifique
obj.Traiter("test");     // Appelle la version string spécifique
obj.Traiter(new object()); // Appelle la version générique (seule option possible)
```

Le compilateur préfère les méthodes non génériques aux méthodes génériques lorsque les deux peuvent s'appliquer.

## 4.3.2. Bonnes pratiques

Pour tirer le meilleur parti de la surcharge de méthodes tout en évitant les pièges courants, voici quelques bonnes pratiques à suivre.

### Quand utiliser la surcharge

1. **Pour des opérations conceptuellement identiques** qui diffèrent par les types ou le nombre de paramètres

```csharp
// Bon exemple : même opération, différents types d'entrée
public string Formater(DateTime date) { return date.ToString("yyyy-MM-dd"); }
public string Formater(TimeSpan duree) { return duree.ToString(@"hh\:mm\:ss"); }
```

2. **Pour proposer des versions simplifiées** d'une méthode avec des valeurs par défaut (alternative aux paramètres optionnels)

```csharp
// Version complète
public void ConfigurerConnexion(string serveur, int port, bool ssl, int timeout, bool retry)
{
    // Implémentation complète
}

// Versions simplifiées
public void ConfigurerConnexion(string serveur, int port)
{
    // Utilise des valeurs par défaut pour les autres paramètres
    ConfigurerConnexion(serveur, port, true, 30, true);
}

public void ConfigurerConnexion(string serveur)
{
    // Port par défaut
    ConfigurerConnexion(serveur, 8080);
}
```

3. **Pour des conversions de types** ou des adaptations d'API

```csharp
public void EnregistrerDonnee(int valeur) { /* ... */ }
public void EnregistrerDonnee(string valeur) { EnregistrerDonnee(int.Parse(valeur)); }
```

### Pièges à éviter

1. **Évitez de créer des surcharges avec des comportements différents**

```csharp
// MAUVAIS EXEMPLE : comportements différents
public void Traiter(int id) {
    // Récupère des données d'une base de données
}

public void Traiter(string nom) {
    // Envoie un email
    // Ce comportement est complètement différent et prête à confusion
}
```

2. **Attention aux surcharges qui diffèrent uniquement par des types proches**

```csharp
// RISQUÉ : facilement confondable
public void Calculer(int valeur) { /* ... */ }
public void Calculer(long valeur) { /* ... */ }
public void Calculer(float valeur) { /* ... */ }
```

3. **Évitez les surcharges qui diffèrent uniquement par des types référence/valeur de même nom**

```csharp
// PROBLÉMATIQUE : Confusion avec Nullable<T>
public void Traiter(int valeur) { /* ... */ }
public void Traiter(int? valeur) { /* ... */ }
```

4. **Attention aux surcharges qui diffèrent seulement par les modificateurs `ref`/`out`**

```csharp
// CONFUS : facilement source d'erreurs
public bool TryParse(string input, out int resultat) { /* ... */ }
public int TryParse(string input, ref int resultat) { /* ... */ }
```

### Principes pour des surcharges efficaces

1. **Cohérence sémantique** : Toutes les surcharges doivent réaliser conceptuellement la même opération

```csharp
// BIEN : même concept, différentes implémentations
public class DessinateurForme
{
    public void Dessiner(Cercle cercle) { /* Dessine un cercle */ }
    public void Dessiner(Rectangle rectangle) { /* Dessine un rectangle */ }
    public void Dessiner(Triangle triangle) { /* Dessine un triangle */ }
}
```

2. **Délégation à une implémentation principale** : Réduisez la duplication en faisant appel à une version "maître"

```csharp
public class GestionnaireEmail
{
    // Implémentation principale
    public bool EnvoyerEmail(string destinataire, string sujet, string corps,
                           bool html, List<string> pieces_jointes, int priorite)
    {
        // Implémentation complète
        return true;
    }

    // Surcharges qui délèguent à la version principale
    public bool EnvoyerEmail(string destinataire, string sujet, string corps)
    {
        return EnvoyerEmail(destinataire, sujet, corps, false, new List<string>(), 1);
    }

    public bool EnvoyerEmail(string destinataire, string sujet, string corps, bool html)
    {
        return EnvoyerEmail(destinataire, sujet, corps, html, new List<string>(), 1);
    }
}
```

3. **Documentation claire** des différences entre les surcharges

```csharp
/// <summary>
/// Convertit une chaîne en entier.
/// </summary>
/// <param name="texte">La chaîne à convertir</param>
/// <returns>La valeur entière</returns>
/// <exception cref="FormatException">Si la chaîne n'est pas un nombre valide</exception>
public int Convertir(string texte)
{
    return int.Parse(texte);
}

/// <summary>
/// Tente de convertir une chaîne en entier sans générer d'exception.
/// </summary>
/// <param name="texte">La chaîne à convertir</param>
/// <param name="defaut">La valeur à retourner en cas d'échec</param>
/// <returns>La valeur entière ou la valeur par défaut si la conversion échoue</returns>
public int Convertir(string texte, int defaut)
{
    if (int.TryParse(texte, out int resultat))
        return resultat;
    return defaut;
}
```

4. **Utiliser les paramètres optionnels** comme alternative aux nombreuses surcharges

```csharp
// Au lieu de nombreuses surcharges...
public void Configurer(string nom) { /* ... */ }
public void Configurer(string nom, int timeout) { /* ... */ }
public void Configurer(string nom, int timeout, bool ssl) { /* ... */ }

// Considérez plutôt :
public void Configurer(string nom, int timeout = 30, bool ssl = true)
{
    // Une seule implémentation
}
```

### Quand privilégier d'autres approches

Dans certains cas, d'autres approches peuvent être plus appropriées que la surcharge de méthodes :

1. **Méthodes avec des noms différents** pour des opérations nettement différentes

```csharp
// Plutôt que :
public void Traiter(Client client) { /* ... */ }
public void Traiter(Facture facture) { /* ... */ }

// Préférez :
public void TraiterClient(Client client) { /* ... */ }
public void TraiterFacture(Facture facture) { /* ... */ }
```

2. **Classes génériques** au lieu de nombreuses surcharges pour des types différents

```csharp
// Plutôt que :
public void Stocker(int valeur) { /* ... */ }
public void Stocker(string valeur) { /* ... */ }
public void Stocker(DateTime valeur) { /* ... */ }
// Et ainsi de suite pour chaque type...

// Préférez :
public void Stocker<T>(T valeur) { /* ... */ }
```

3. **Paramètres optionnels** ou **paramètres nommés** pour simplifier les API avec de nombreux paramètres

```csharp
// Plutôt que de nombreuses surcharges avec des combinaisons de paramètres
public void Configure(
    string nom,
    string description = "",
    bool actif = true,
    int timeout = 30,
    LogLevel niveau = LogLevel.Info)
{
    // Une seule implémentation
}

// Utilisation avec paramètres nommés
service.Configure(
    nom: "MonService",
    niveau: LogLevel.Debug,
    timeout: 60
);
```

## Exemple complet : Utilisation efficace de la surcharge

Voici un exemple complet montrant comment utiliser efficacement la surcharge de méthodes dans une classe utilitaire pour le traitement de fichiers :

```csharp
public class GestionnaireFichiers
{
    // Version principale qui fait le travail réel
    public string LireFichier(string chemin, Encoding encodage, bool ignoreCommentaires,
                            char commentaireChar, bool trim)
    {
        // Vérifications
        if (string.IsNullOrEmpty(chemin))
            throw new ArgumentException("Le chemin ne peut pas être vide", nameof(chemin));

        if (!File.Exists(chemin))
            throw new FileNotFoundException("Fichier introuvable", chemin);

        // Lecture du fichier
        string contenu = File.ReadAllText(chemin, encodage);

        // Traitement selon les options
        if (ignoreCommentaires)
        {
            var lignes = contenu.Split('\n')
                .Where(ligne => !ligne.TrimStart().StartsWith(commentaireChar.ToString()))
                .ToArray();
            contenu = string.Join("\n", lignes);
        }

        if (trim)
        {
            contenu = contenu.Trim();
        }

        return contenu;
    }

    // Surcharge simplifiée - encodage par défaut
    public string LireFichier(string chemin)
    {
        return LireFichier(chemin, Encoding.UTF8, false, '#', true);
    }

    // Surcharge avec encodage personnalisé
    public string LireFichier(string chemin, Encoding encodage)
    {
        return LireFichier(chemin, encodage, false, '#', true);
    }

    // Surcharge pour ignorer les commentaires
    public string LireFichier(string chemin, bool ignoreCommentaires, char commentaireChar = '#')
    {
        return LireFichier(chemin, Encoding.UTF8, ignoreCommentaires, commentaireChar, true);
    }

    // Méthode pour écrire dans un fichier - complètement différente sémantiquement
    // donc utilise un nom différent plutôt qu'une surcharge
    public void EcrireFichier(string chemin, string contenu, Encoding encodage = null)
    {
        encodage = encodage ?? Encoding.UTF8;
        File.WriteAllText(chemin, contenu, encodage);
    }
}
```

Utilisation :

```csharp
var gestionnaire = new GestionnaireFichiers();

// Utilisation des différentes surcharges
string contenu1 = gestionnaire.LireFichier("config.txt");
string contenu2 = gestionnaire.LireFichier("data.csv", Encoding.GetEncoding("ISO-8859-1"));
string contenu3 = gestionnaire.LireFichier("script.py", true, '#');

// Écriture - méthode différente, pas une surcharge
gestionnaire.EcrireFichier("output.txt", "Contenu à écrire");
```

## Conclusion

La surcharge de méthodes est un outil puissant en C# qui vous permet de créer des API intuitives et flexibles. En suivant les règles de résolution et les bonnes pratiques présentées dans cette section, vous pourrez utiliser efficacement cette fonctionnalité tout en évitant les pièges courants.

Points clés à retenir :
- Les méthodes surchargées doivent avoir le même nom mais des paramètres différents
- Le compilateur choisit la meilleure correspondance selon des règles précises
- Utilisez la surcharge pour des opérations conceptuellement identiques
- Déléguez à une implémentation principale pour réduire la duplication
- Dans certains cas, envisagez d'autres approches comme les paramètres optionnels ou les génériques

En maîtrisant la surcharge de méthodes, vous améliorerez la qualité et l'utilisabilité de votre code C#.

⏭️
