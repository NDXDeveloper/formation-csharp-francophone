# 2.6 Enregistrement des commentaires et documentation en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Écrire du code ne consiste pas seulement à créer des instructions que l'ordinateur peut comprendre, mais aussi à rendre ce code compréhensible pour les autres développeurs (et pour vous-même quelques mois plus tard !). Les commentaires et la documentation jouent un rôle crucial dans ce processus. Ils permettent d'expliquer le fonctionnement du code, les décisions de conception, et facilitent la maintenance et la collaboration.

## 2.6.1 Commentaires de ligne et de bloc

Les commentaires sont des portions de texte dans votre code qui sont ignorées par le compilateur. Ils sont destinés uniquement aux lecteurs humains du code. C# prend en charge deux types de commentaires standard : les commentaires de ligne et les commentaires de bloc.

### Commentaires de ligne

Les commentaires de ligne commencent par deux barres obliques (`//`) et continuent jusqu'à la fin de la ligne. Ils sont parfaits pour les explications brèves.

```csharp
// Ceci est un commentaire de ligne
int age = 25; // Un commentaire peut aussi suivre une instruction

// Les commentaires de ligne peuvent s'étendre sur plusieurs lignes
// comme ceci, mais chaque ligne doit commencer
// par les deux barres obliques
```

### Commentaires de bloc

Les commentaires de bloc commencent par `/*` et se terminent par `*/`. Tout ce qui se trouve entre ces deux délimiteurs est ignoré par le compilateur, même si cela s'étend sur plusieurs lignes.

```csharp
/* Ceci est un commentaire de bloc
   qui s'étend sur plusieurs lignes
   sans besoin de préfixe sur chaque ligne */

int somme = 5 + /* commentaire au milieu d'une instruction */ 10;

/* Les commentaires de bloc peuvent être utiles pour
   désactiver temporairement des portions de code
   int x = 10;
   int y = 20;
   Console.WriteLine(x + y);
*/
```

### Bonnes pratiques pour les commentaires

1. **Commentez le pourquoi, pas le quoi**

   Le code lui-même devrait être suffisamment clair pour montrer **ce** qu'il fait. Les commentaires devraient expliquer **pourquoi** il le fait ainsi.

   ```csharp
   // Mauvais commentaire - décrit simplement le code
   // Incrémente compteur de 1
   compteur++;

   // Bon commentaire - explique la raison
   // Augmente le compteur pour chaque utilisateur actif
   compteur++;
   ```

2. **Gardez les commentaires à jour**

   Des commentaires obsolètes sont pires que pas de commentaires du tout, car ils peuvent induire en erreur.

3. **Utilisez des commentaires pour marquer les tâches à faire**

   Les marqueurs comme `TODO:`, `HACK:`, `FIXME:` sont reconnus par de nombreux éditeurs et IDEs.

   ```csharp
   // TODO: Ajouter la validation d'entrée
   // FIXME: Corriger le bug quand la date est vide
   // HACK: Solution temporaire jusqu'à la refactorisation
   ```

4. **Évitez les commentaires excessifs**

   Trop de commentaires peuvent rendre le code difficile à lire. Si vous sentez le besoin d'expliquer chaque ligne, c'est peut-être un signe que votre code pourrait être simplifié.

5. **Utilisez les commentaires pour désactiver du code pendant le développement**

   ```csharp
   /*
   // Version expérimentale
   CalculerVersion2();
   */

   // Version stable
   CalculerVersion1();
   ```

### Commentaires en-tête de fichier

Il est courant d'inclure un commentaire en tête de fichier avec des informations sur le fichier, comme l'auteur, la date de création, une description, etc.

```csharp
//--------------------------------------------------------------
// Nom du fichier : CalculateurTVA.cs
// Auteur : Marie Dupont
// Date de création : 01/01/2023
// Description : Classe pour calculer la TVA sur différents produits
// Historique des modifications :
// 15/01/2023 - Ajout du support des taux de TVA réduits
//--------------------------------------------------------------
```

## 2.6.2 Commentaires XML pour la documentation

C# offre un système spécial de commentaires XML qui permet non seulement de documenter votre code, mais aussi de générer automatiquement une documentation externe dans divers formats (HTML, fichiers d'aide, etc.).

### Syntaxe de base

Les commentaires XML commencent par trois barres obliques (`///`) ou par `/** */` pour les blocs. Ils doivent être placés immédiatement avant l'élément à documenter (classe, méthode, propriété, etc.).

```csharp
/// <summary>
/// Calcule la somme de deux nombres entiers.
/// </summary>
/// <param name="a">Le premier nombre à additionner.</param>
/// <param name="b">Le second nombre à additionner.</param>
/// <returns>La somme des deux nombres.</returns>
public int Additionner(int a, int b)
{
    return a + b;
}
```

### Balises XML courantes

| Balise | Description | Usage |
|--------|-------------|-------|
| `<summary>` | Description générale de l'élément | Obligatoire pour toute documentation |
| `<param>` | Description d'un paramètre | Une pour chaque paramètre de méthode |
| `<returns>` | Description de la valeur de retour | Pour les méthodes qui retournent une valeur |
| `<remarks>` | Informations supplémentaires | Pour des détails qui n'appartiennent pas au résumé |
| `<example>` | Exemple d'utilisation | Pour montrer comment utiliser l'élément |
| `<exception>` | Exception qui peut être levée | Une pour chaque type d'exception possible |
| `<seealso>` | Référence croisée | Pour pointer vers des éléments liés |
| `<value>` | Description de la valeur d'une propriété | Pour les propriétés |
| `<typeparam>` | Description d'un paramètre de type | Pour les méthodes et classes génériques |

### Exemples de commentaires XML

#### Documentation d'une classe

```csharp
/// <summary>
/// Représente un client avec ses informations personnelles et ses commandes.
/// </summary>
/// <remarks>
/// Cette classe fait partie du module de gestion clientèle et implémente
/// l'interface IClient pour garantir la compatibilité avec le système CRM.
/// </remarks>
public class Client
{
    // Propriétés et méthodes...
}
```

#### Documentation d'une propriété

```csharp
/// <summary>
/// Obtient ou définit le nom complet du client.
/// </summary>
/// <value>Une chaîne de caractères représentant le nom et le prénom.</value>
public string NomComplet { get; set; }
```

#### Documentation d'une méthode avec exceptions

```csharp
/// <summary>
/// Effectue un retrait sur le compte bancaire.
/// </summary>
/// <param name="montant">Le montant à retirer. Doit être positif.</param>
/// <returns>Le nouveau solde après le retrait.</returns>
/// <exception cref="ArgumentException">
/// Levée si le montant est négatif ou nul.
/// </exception>
/// <exception cref="SoldeInsuffisantException">
/// Levée si le solde actuel est inférieur au montant du retrait.
/// </exception>
/// <example>
/// <code>
/// CompteBancaire compte = new CompteBancaire(1000);
/// decimal nouveauSolde = compte.Retirer(500); // nouveauSolde sera 500
/// </code>
/// </example>
public decimal Retirer(decimal montant)
{
    if (montant <= 0)
    {
        throw new ArgumentException("Le montant doit être positif", nameof(montant));
    }

    if (Solde < montant)
    {
        throw new SoldeInsuffisantException("Solde insuffisant pour ce retrait");
    }

    Solde -= montant;
    return Solde;
}
```

#### Documentation d'une classe générique

```csharp
/// <summary>
/// Représente une pile générique d'éléments.
/// </summary>
/// <typeparam name="T">Type des éléments stockés dans la pile.</typeparam>
public class Pile<T>
{
    // Implémentation...
}
```

### Références à d'autres types et membres

Vous pouvez créer des liens vers d'autres types, membres ou paramètres en utilisant les balises spéciales :

```csharp
/// <summary>
/// Convertit un <see cref="Client"/> en <see cref="ClientDTO"/>.
/// </summary>
/// <param name="client">Le client à convertir.</param>
/// <returns>Un objet <see cref="ClientDTO"/>.</returns>
/// <seealso cref="ConvertirEnClient"/>
public ClientDTO ConvertirEnDTO(Client client)
{
    // Implémentation...
}
```

### Formatage du texte dans les commentaires XML

Vous pouvez utiliser certaines balises HTML de base pour formater le texte dans vos commentaires XML :

```csharp
/// <summary>
/// Analyse une chaîne de caractères et extrait les informations suivantes :
/// <list type="bullet">
/// <item><description>Nom</description></item>
/// <item><description>Âge</description></item>
/// <item><description>Email</description></item>
/// </list>
///
/// <para>Le format attendu est : "Nom,Âge,Email"</para>
///
/// <para>Exemples :</para>
/// <code>
/// "Jean Dupont,42,jean.dupont@exemple.com"
/// "Marie Martin,35,marie.martin@exemple.com"
/// </code>
/// </summary>
public void AnalyserChaine(string chaine)
{
    // Implémentation...
}
```

## 2.6.3 Génération de documentation

Les commentaires XML ne sont pas seulement utiles lorsque vous lisez le code. Ils peuvent également être utilisés pour générer une documentation externe complète. Voici comment procéder.

### Configuration du projet pour la génération de documentation

Avant de pouvoir générer une documentation, vous devez configurer votre projet pour qu'il crée un fichier XML de documentation lors de la compilation.

#### Dans Visual Studio

1. **Faites un clic droit sur votre projet** dans l'Explorateur de solutions.
2. Sélectionnez **Propriétés**.
3. Allez dans l'onglet **Build**.
4. Cochez la case **Fichier de documentation XML**.
5. Spécifiez un chemin pour le fichier XML (par défaut, il sera placé dans le dossier de sortie de compilation).

#### Manuellement dans le fichier de projet (.csproj)

Vous pouvez également ajouter cette configuration directement dans votre fichier `.csproj` :

```xml
<PropertyGroup>
  <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\$(AssemblyName).xml</DocumentationFile>
</PropertyGroup>
```

### Outils de génération de documentation

Une fois que vous avez configuré votre projet pour générer le fichier XML de documentation, vous pouvez utiliser divers outils pour créer une documentation exploitable.

#### 1. DocFX

[DocFX](https://dotnet.github.io/docfx/) est un outil open-source développé par Microsoft pour générer une documentation à partir de commentaires XML.

Installation via la ligne de commande :
```
dotnet tool install -g docfx
```

Création d'un projet de documentation :
```
docfx init -q
```

Génération de la documentation :
```
docfx docfx.json
```

#### 2. Sandcastle

[Sandcastle](https://github.com/EWSoftware/SHFB) est un autre outil populaire pour générer une documentation à partir de commentaires XML en C#.

#### 3. Doxygen

[Doxygen](https://www.doxygen.nl/) est un générateur de documentation multilangage qui prend en charge les commentaires XML de C#.

### VSdocman

[VSdocman](https://www.helixoft.com/vsdocman/overview.html) est une extension Visual Studio qui facilite la création et la maintenance des commentaires XML, ainsi que la génération de documentation.

### Visualisation dans Visual Studio

Même sans générer de documentation externe, les commentaires XML sont immédiatement utiles dans Visual Studio :

1. **IntelliSense** : Lorsque vous utilisez une méthode, Visual Studio affiche le contenu de la balise `<summary>` et les informations sur les paramètres.

2. **Fenêtre d'aperçu** : En survolant un élément, une fenêtre d'aperçu apparaît avec toute la documentation XML.

### Exemple complet de documentation générée

Voici un exemple de workflow complet pour générer une documentation avec DocFX :

1. **Configurez votre projet** pour générer le fichier XML de documentation.

2. **Installez DocFX** :
   ```
   dotnet tool install -g docfx
   ```

3. **Initialisez un projet DocFX** dans un dossier séparé :
   ```
   mkdir Documentation
   cd Documentation
   docfx init -q
   ```

4. **Modifiez le fichier de configuration** `docfx.json` pour pointer vers votre projet C# :
   ```json
   {
     "metadata": [
       {
         "src": [
           {
             "files": [
               "src/**.csproj"
             ],
             "src": "../"  // Chemin relatif vers votre solution
           }
         ],
         "dest": "api",
         "disableGitFeatures": false,
         "disableDefaultFilter": false
       }
     ],
     "build": {
       "content": [
         {
           "files": [
             "api/**.yml",
             "api/index.md"
           ]
         },
         {
           "files": [
             "articles/**.md",
             "articles/**/toc.yml",
             "toc.yml",
             "*.md"
           ]
         }
       ],
       "resource": [
         {
           "files": [
             "images/**"
           ]
         }
       ],
       "overwrite": [
         {
           "files": [
             "apidoc/**.md"
           ],
           "exclude": [
             "obj/**",
             "_site/**"
           ]
         }
       ],
       "dest": "_site",
       "globalMetadataFiles": [],
       "fileMetadataFiles": [],
       "template": [
         "default"
       ],
       "postProcessors": [],
       "markdownEngineName": "markdig",
       "noLangKeyword": false,
       "keepFileLink": false,
       "cleanupCacheHistory": false,
       "disableGitFeatures": false
     }
   }
   ```

5. **Générez les métadonnées** :
   ```
   docfx metadata
   ```

6. **Générez la documentation** :
   ```
   docfx build
   ```

7. **Prévisualisez la documentation** :
   ```
   docfx serve _site
   ```

8. Ouvrez votre navigateur à l'adresse `http://localhost:8080` pour voir votre documentation.

### Bonnes pratiques pour la documentation

1. **Soyez cohérent** : Utilisez un style uniforme dans toute votre documentation.

2. **Documentez les interfaces publiques** : Concentrez-vous sur les éléments qui seront utilisés par d'autres développeurs.

3. **Mettez à jour la documentation avec le code** : Une documentation obsolète est pire que pas de documentation.

4. **Incluez des exemples** : Les exemples concrets aident énormément à comprendre comment utiliser votre code.

5. **Pensez aux nouveaux venus** : Écrivez votre documentation comme si elle s'adressait à quelqu'un qui découvre votre projet.

6. **Expliquez les pourquoi, pas seulement les comment** : Expliquez les raisons derrière les décisions de conception importantes.

7. **Utilisez les balises appropriées** : Chaque balise XML a un but spécifique, utilisez-la en conséquence.

### Automatisation de la génération de documentation

Dans un environnement d'intégration continue (CI), vous pouvez automatiser la génération de documentation à chaque build ou release.

Exemple avec GitHub Actions :

```yaml
name: Generate Documentation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 6.0.x

    - name: Install DocFX
      run: dotnet tool install -g docfx

    - name: Generate Documentation
      run: |
        cd Documentation
        docfx metadata
        docfx build

    - name: Deploy Documentation
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./Documentation/_site
```

## Résumé

- Les **commentaires de ligne et de bloc** (`//` et `/* */`) sont utilisés pour expliquer le code aux développeurs et sont ignorés par le compilateur.
- Les **commentaires XML** (`///`) sont utilisés pour documenter les éléments du code (classes, méthodes, propriétés) de manière structurée.
- La **génération de documentation** permet de créer une documentation externe à partir des commentaires XML, facilitant la compréhension et l'utilisation de votre code par d'autres développeurs.

Prendre le temps de bien documenter votre code est un investissement qui paiera à long terme, en facilitant la maintenance, en réduisant les erreurs, et en améliorant la collaboration avec d'autres développeurs. N'oubliez pas que vous écrivez du code non seulement pour l'ordinateur, mais aussi pour les humains qui le liront plus tard !

⏭️ 3. [Tableaux et collections](/03-tableaux-et-collections/README.md)
