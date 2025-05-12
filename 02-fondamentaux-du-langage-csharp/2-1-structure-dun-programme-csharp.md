# 2.1 Structure d'un programme C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La structure d'un programme C# est le fondement sur lequel repose toute application. Comprendre cette structure est essentiel pour tout développeur, qu'il soit débutant ou expérimenté. Dans cette section, nous allons explorer les composants fondamentaux qui constituent un programme C#.

## 2.1.1 Espaces de noms (namespaces)

Les espaces de noms (ou "namespaces" en anglais) sont des conteneurs qui organisent les classes et autres types en groupes logiques. Ils jouent un rôle crucial dans la structuration et la gestion du code.

### Pourquoi utiliser des espaces de noms ?

- **Éviter les conflits de noms** : Imaginez que vous utilisez deux bibliothèques différentes, chacune contenant une classe nommée `Logger`. Sans espaces de noms, comment l'ordinateur saurait-il quelle classe `Logger` vous voulez utiliser ?
- **Organiser le code** : Les espaces de noms permettent de regrouper des fonctionnalités similaires.
- **Faciliter la réutilisation** : Ils rendent votre code plus facile à partager et à intégrer dans d'autres projets.

### Déclaration d'un espace de noms

```csharp
namespace MonApplication
{
    // Classes, interfaces, structures, etc.
}
```

### Utilisation d'un espace de noms

Pour utiliser des types définis dans un espace de noms, vous avez deux options :

1. **Directive `using`** (recommandée) :
```csharp
using System;
using System.Collections.Generic;

// Maintenant vous pouvez utiliser Console et List directement
Console.WriteLine("Bonjour");
List<string> maListe = new List<string>();
```

2. **Nom complet** :
```csharp
// Sans directive using, vous devez spécifier le chemin complet
System.Console.WriteLine("Bonjour");
System.Collections.Generic.List<string> maListe = new System.Collections.Generic.List<string>();
```

### Espaces de noms courants

- `System` : Types fondamentaux, exceptions, etc.
- `System.Collections.Generic` : Collections typées (List, Dictionary, etc.)
- `System.Linq` : Requêtes et manipulations de données
- `System.IO` : Entrées/sorties de fichiers
- `System.Net` : Fonctionnalités réseau
- `System.Windows.Forms` : Interface utilisateur Windows (WinForms)
- `System.Xml` : Traitement XML

### Espaces de noms imbriqués

Les espaces de noms peuvent être imbriqués pour créer une hiérarchie :

```csharp
namespace MonEntreprise
{
    namespace MonProduit
    {
        public class MaClasse
        {
            // ...
        }
    }
}
```

Cela peut aussi s'écrire :

```csharp
namespace MonEntreprise.MonProduit
{
    public class MaClasse
    {
        // ...
    }
}
```

Pour accéder à `MaClasse`, on écrirait :

```csharp
using MonEntreprise.MonProduit;

// Puis
MaClasse instance = new MaClasse();

// Ou directement sans using
MonEntreprise.MonProduit.MaClasse instance = new MonEntreprise.MonProduit.MaClasse();
```

## 2.1.2 Classes et fichiers

En C#, les classes sont les composants fondamentaux qui encapsulent les données (attributs) et les comportements (méthodes).

### Relation entre classes et fichiers

- En général, on place **une classe par fichier** (bonne pratique)
- Le nom du fichier correspond habituellement au nom de la classe (par exemple, `MaClasse.cs`)
- Un fichier peut contenir plusieurs classes, mais c'est généralement déconseillé pour la lisibilité

### Structure d'une classe

```csharp
using System;  // Les directives using sont en haut du fichier

namespace MonApplication
{
    public class MaClasse
    {
        // Attributs (champs) - stockent les données
        private int _monAttribut;
        public string NomPublic;

        // Propriétés - accès contrôlé aux données
        public int MaPropriete { get; set; }

        // Constructeur - initialise les objets
        public MaClasse()
        {
            _monAttribut = 0;
            NomPublic = "Valeur par défaut";
        }

        // Méthodes - comportements
        public void MaMethode()
        {
            Console.WriteLine("Exécution de ma méthode");
        }

        // Méthode avec paramètres et valeur de retour
        public int Calculer(int a, int b)
        {
            return a + b + _monAttribut;
        }
    }

    // Une autre classe dans le même fichier (éviter si possible)
    internal class ClasseSecondaire
    {
        // ...
    }
}
```

### Types de classes

- **Classes publiques** (`public class`) : Accessibles depuis n'importe où
- **Classes internes** (`internal class`) : Accessibles uniquement dans le même assembly
- **Classes statiques** (`static class`) : Contiennent uniquement des membres statiques, ne peuvent pas être instanciées
- **Classes abstraites** (`abstract class`) : Servent de base à d'autres classes, ne peuvent pas être instanciées directement
- **Classes partielles** (`partial class`) : Permettent de définir une classe dans plusieurs fichiers

### Organisation typique d'un fichier C#

1. Directives `using`
2. Déclaration de l'espace de noms
3. Classes et autres types

## 2.1.3 Point d'entrée (méthode Main)

Tout programme C# exécutable doit avoir un **point d'entrée**, c'est-à-dire l'endroit où l'exécution commence. Ce point d'entrée est traditionnellement une méthode appelée `Main`.

### Caractéristiques de la méthode Main

- Elle doit être `static`
- Elle est généralement `public` ou `private` (les deux fonctionnent)
- Elle peut retourner `void` ou `int` (code de retour pour le système d'exploitation)
- Elle peut prendre un paramètre `string[] args` pour les arguments de ligne de commande

### Variations de la méthode Main

```csharp
// Version la plus courante
static void Main(string[] args)
{
    Console.WriteLine("Démarrage du programme");
    // Code du programme...
}

// Avec code de retour
static int Main(string[] args)
{
    // Code du programme...
    return 0; // 0 signifie généralement "succès"
}

// Sans paramètres de ligne de commande
static void Main()
{
    // Code du programme...
}
```

### Exemple complet d'un programme simple

```csharp
using System;

namespace MonPremierProgramme
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Bonjour le monde !");

            // Utilisation des arguments de ligne de commande
            if (args.Length > 0)
            {
                Console.WriteLine("Arguments reçus :");
                foreach (string arg in args)
                {
                    Console.WriteLine($"  - {arg}");
                }
            }

            Console.WriteLine("Appuyez sur une touche pour quitter...");
            Console.ReadKey();
        }
    }
}
```

Pour exécuter ce programme avec des arguments, on pourrait utiliser :
```
MonPremierProgramme.exe arg1 "argument 2" arg3
```

### Plusieurs méthodes Main dans un même projet

Si vous avez plusieurs classes avec une méthode `Main` dans un projet, vous devez spécifier quelle classe contient le point d'entrée à utiliser :

1. Dans Visual Studio : Projet > Propriétés > Onglet Application > "Classe de démarrage"
2. En ligne de commande : `csc /main:MonNamespace.MaClasse Fichier1.cs Fichier2.cs`

## 2.1.4 Programmes top-level (C# 9+)

À partir de C# 9.0 (disponible avec .NET 5 en novembre 2020), une nouvelle syntaxe simplifiée a été introduite : les **programmes top-level**. Cette approche permet d'écrire des programmes plus concis en évitant d'avoir à créer explicitement une classe et une méthode `Main`.

### Syntaxe des programmes top-level

Au lieu d'écrire :

```csharp
using System;

namespace MonProgramme
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Bonjour le monde !");
        }
    }
}
```

Vous pouvez simplement écrire :

```csharp
using System;

Console.WriteLine("Bonjour le monde !");
```

Le compilateur génère automatiquement une classe et une méthode `Main` pour vous.

### Fonctionnalités des programmes top-level

- Vous pouvez toujours accéder aux arguments de ligne de commande via la variable `args`
- Vous pouvez définir des fonctions locales
- Vous pouvez retourner un code de sortie avec `return`
- Vous pouvez toujours définir des espaces de noms et des classes dans le même fichier

### Exemple complet d'un programme top-level

```csharp
using System;

// Code directement au niveau supérieur
Console.WriteLine("Bonjour depuis un programme top-level !");

// Accès aux arguments de ligne de commande
foreach (var arg in args)
{
    Console.WriteLine($"Argument: {arg}");
}

// Fonction locale
void AfficherMessage(string message)
{
    Console.WriteLine($"Message: {message}");
}

// Appel de la fonction locale
AfficherMessage("C'est si simple maintenant !");

// Vous pouvez toujours définir des classes
class Utilisateur
{
    public string Nom { get; set; }
    public void Saluer()
    {
        Console.WriteLine($"Bonjour, je m'appelle {Nom}");
    }
}

// Et les utiliser
var utilisateur = new Utilisateur { Nom = "Marie" };
utilisateur.Saluer();

// Retourner un code de sortie (optionnel)
return 0;
```

### Limitations des programmes top-level

- Un seul fichier dans le projet peut contenir du code top-level (sinon, il y aurait ambiguïté sur le point d'entrée)
- Les programmes top-level sont particulièrement adaptés aux petits programmes et aux scripts, mais pour les applications complexes, la structure traditionnelle peut être plus appropriée pour la clarté et l'organisation

### Choix entre style traditionnel et top-level

| Style traditionnel | Style top-level |
|--|--|
| Plus explicite | Plus concis |
| Meilleur pour les grandes applications | Idéal pour les petits programmes |
| Familier pour les développeurs venant d'autres langages | Simplifie l'apprentissage pour les débutants |
| Nécessaire pour les versions de C# antérieures à 9.0 | Nécessite C# 9.0 ou ultérieur |

## Résumé

- Les **espaces de noms** organisent votre code en groupes logiques et évitent les conflits de noms.
- Les **classes** sont les blocs de construction fondamentaux qui encapsulent données et comportements.
- Le **point d'entrée** traditionnel est une méthode `Main` statique dans une classe.
- Les **programmes top-level** (C# 9+) permettent d'écrire du code plus concis sans créer explicitement une classe et une méthode `Main`.

En comprenant ces concepts fondamentaux, vous avez maintenant une base solide pour développer des applications en C#, quelle que soit leur complexité.

⏭️ 2.2. [Types de données](/02-fondamentaux-du-langage-csharp/2-2-types-de-donnees.md)
