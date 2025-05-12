# 1.6. Votre premier programme "Hello World"

🔝 Retour au [Sommaire](/SOMMAIRE.md)

![Hello World en C#](https://via.placeholder.com/600x150?text=Hello+World+en+C%23)

La tradition dans l'apprentissage de tout langage de programmation commence par le fameux programme "Hello World". Cette première étape simple mais symbolique vous permettra de comprendre les éléments fondamentaux d'un programme C#, sa structure et son exécution. Dans cette section, nous allons créer ce programme emblématique dans les deux versions principales de .NET que nous utiliserons tout au long de ce tutoriel : .NET Framework 4.7.2 et .NET 8.

## 1.6.1. Structure d'un projet C#

Avant d'écrire notre premier code, examinons la structure typique d'un projet C#. Bien que la structure de base soit similaire entre .NET Framework et .NET moderne, il existe quelques différences notables.

### Structure d'un projet .NET Framework 4.7.2

Un projet console typique en .NET Framework 4.7.2 a la structure suivante :

```
MonProjetHelloWorld/
│
├── Properties/
│   └── AssemblyInfo.cs         // Métadonnées de l'assembly
│
├── App.config                  // Configuration de l'application
├── Program.cs                  // Point d'entrée du programme
├── MonProjetHelloWorld.csproj  // Fichier de projet XML
│
├── packages/                   // Packages NuGet (si utilisés)
│
├── obj/                        // Fichiers intermédiaires de compilation
│   └── ...
│
└── bin/                        // Fichiers binaires compilés
    └── Debug/
        ├── MonProjetHelloWorld.exe
        ├── MonProjetHelloWorld.pdb
        └── ...
```


### Structure d'un projet .NET 8

Un projet console en .NET 8 est plus épuré :

```
MonProjetHelloWorld/
│
├── Program.cs                  // Point d'entrée avec top-level statements
├── MonProjetHelloWorld.csproj  // Fichier de projet simplifié
│
├── obj/                        // Fichiers intermédiaires
│   └── ...
│
└── bin/                        // Fichiers binaires
    └── Debug/
        └── net8.0/
            ├── MonProjetHelloWorld.dll
            ├── MonProjetHelloWorld.exe    // Sur Windows uniquement
            ├── MonProjetHelloWorld.deps.json
            ├── MonProjetHelloWorld.runtimeconfig.json
            └── ...
```


### Comparaison des fichiers projet (.csproj)

#### Fichier .csproj en .NET Framework 4.7.2

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" Condition="Exists('$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props')" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{12345678-1234-1234-1234-1234567890AB}</ProjectGuid>
    <OutputType>Exe</OutputType>
    <RootNamespace>MonProjetHelloWorld</RootNamespace>
    <AssemblyName>MonProjetHelloWorld</AssemblyName>
    <TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>
    <FileAlignment>512</FileAlignment>
    <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
    <Deterministic>true</Deterministic>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <PlatformTarget>AnyCPU</PlatformTarget>
    <DebugSymbols>true</DebugSymbols>
    <DebugType>full</DebugType>
    <Optimize>false</Optimize>
    <OutputPath>bin\Debug\</OutputPath>
    <DefineConstants>DEBUG;TRACE</DefineConstants>
    <ErrorReport>prompt</ErrorReport>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>
  <!-- Références aux assemblies .NET Framework -->
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="System.Data.DataSetExtensions" />
    <Reference Include="Microsoft.CSharp" />
    <Reference Include="System.Data" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Xml" />
  </ItemGroup>
  <ItemGroup>
    <Compile Include="Program.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
  </ItemGroup>
  <ItemGroup>
    <None Include="App.config" />
  </ItemGroup>
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
</Project>
```


#### Fichier .csproj en .NET 8

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```


Observez la différence de complexité entre les deux formats. Le fichier .NET 8 est considérablement plus simple grâce au SDK style project format introduit avec .NET Core.

## 1.6.2. Comprendre la syntaxe de base

Passons maintenant à l'écriture de notre premier programme. Nous allons examiner la syntaxe dans les deux versions.

### Hello World en .NET Framework 4.7.2

Voici le code complet d'un programme Hello World en .NET Framework 4.7.2 :

```textmate
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MonProjetHelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");

            // Maintient la console ouverte jusqu'à ce que l'utilisateur appuie sur une touche
            Console.WriteLine("Appuyez sur une touche pour quitter...");
            Console.ReadKey();
        }
    }
}
```


### Hello World en .NET 8

Avec .NET 8 et les top-level statements (introduits dans C# 9), le même programme devient beaucoup plus concis :

```textmate
// Top-level statements introduits dans C# 9
Console.WriteLine("Hello, World!");

// En environnement de développement, pas besoin de ReadKey
// pour les applications console modernes
```


Si vous préférez la structure traditionnelle même en .NET 8, vous pouvez utiliser cette forme :

```textmate
using System;

namespace MonProjetHelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```


### Analyse de la syntaxe

Analysons en détail chaque élément de la syntaxe traditionnelle (.NET Framework 4.7.2) :

#### Directives using

```textmate
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
```


- Les directives `using` importent des espaces de noms (namespaces)
- Elles permettent d'utiliser des classes sans spécifier leur chemin complet
- Par exemple, `using System;` permet d'écrire `Console.WriteLine()` au lieu de `System.Console.WriteLine()`
- Visual Studio génère automatiquement plusieurs imports standards, même s'ils ne sont pas tous nécessaires pour un simple Hello World

#### Espace de noms (Namespace)

```textmate
namespace MonProjetHelloWorld
{
    // Contenu du namespace
}
```


- L'espace de noms encapsule les classes et évite les conflits de noms
- Par convention, il porte le même nom que le projet
- Il forme un conteneur logique pour organiser le code

#### Classe Program

```textmate
class Program
{
    // Contenu de la classe
}
```


- En C#, tout le code exécutable doit être contenu dans une classe
- `Program` est la classe principale conventionnelle d'une application console
- Le mot-clé `class` définit une nouvelle classe

#### Méthode Main

```textmate
static void Main(string[] args)
{
    // Instructions du programme
}
```


- `Main` est le point d'entrée de l'application
- `static` signifie que la méthode appartient à la classe elle-même (pas à une instance)
- `void` indique que la méthode ne retourne aucune valeur
- `string[] args` est un tableau de chaînes contenant les arguments de ligne de commande
- Les accolades `{ }` délimitent le bloc de code de la méthode

#### Instruction d'affichage

```textmate
Console.WriteLine("Hello, World!");
```


- `Console` est une classe du namespace `System`
- `WriteLine` est une méthode qui affiche du texte suivi d'un retour à la ligne
- Le texte entre guillemets est une chaîne de caractères (string)
- Le point-virgule `;` termine l'instruction

#### Maintenir la console ouverte

```textmate
Console.WriteLine("Appuyez sur une touche pour quitter...");
Console.ReadKey();
```


- Ces lignes empêchent la console de se fermer immédiatement après l'exécution
- `ReadKey()` attend que l'utilisateur appuie sur une touche
- C'est une pratique courante en .NET Framework pour voir la sortie du programme

### Version interactive de Hello World

Voici une version plus interactive que vous pouvez essayer :

```c#
using System;

namespace MonProjetHelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Bienvenue dans votre premier programme C#!");

            Console.Write("Veuillez entrer votre nom : ");
            string nom = Console.ReadLine();

            // Vérification si le nom est vide
            if (string.IsNullOrWhiteSpace(nom))
            {
                nom = "anonyme";
            }

            Console.WriteLine($"Bonjour, {nom}! C'est un plaisir de vous rencontrer.");

            // Affichage de la date et l'heure actuelles
            DateTime maintenant = DateTime.Now;
            Console.WriteLine($"Il est actuellement {maintenant:HH:mm} le {maintenant:dd/MM/yyyy}.");

            Console.WriteLine("\nAppuyez sur une touche pour quitter...");
            Console.ReadKey();
        }
    }
}
```


Cette version introduit plusieurs concepts supplémentaires :
- `Console.Write()` - Affiche du texte sans saut de ligne
- `Console.ReadLine()` - Lit une ligne de texte entrée par l'utilisateur
- Interpolation de chaînes avec `$"..."` (C# 6+)
- Manipulation de variables et conditions
- Formatage de date et heure

## 1.6.3. Compilation et exécution

Maintenant que nous avons écrit notre programme, voyons comment le compiler et l'exécuter dans les deux environnements.

### Compilation et exécution en .NET Framework 4.7.2

#### Avec Visual Studio

1. **Créer un nouveau projet**
   - Lancez Visual Studio
   - Sélectionnez File > New > Project...
   - Choisissez "Console App (.NET Framework)"
   - Sélectionnez ".NET Framework 4.7.2" dans la liste déroulante
   - Donnez un nom au projet (ex: "HelloWorld")
   - Cliquez sur Create/Créer

2. **Écrire le code**
   - Visual Studio génère un modèle de base, que vous pouvez modifier

3. **Compiler le projet**
   - Build > Build Solution (ou appuyez sur Ctrl+Shift+B)
   - Vérifiez les erreurs dans la fenêtre "Error List"

4. **Exécuter le programme**
   - Debug > Start Without Debugging (ou appuyez sur Ctrl+F5)
   - Ou Debug > Start Debugging (ou appuyez sur F5) pour exécuter en mode débogage

#### Avec la ligne de commande

Vous pouvez également compiler et exécuter votre programme avec la ligne de commande :

1. **Enregistrez votre code** dans un fichier nommé `Program.cs`

2. **Ouvrez une invite de commande** (Developer Command Prompt for VS)

3. **Compilez le programme**
```
csc Program.cs
```


4. **Exécutez le programme**
```
Program.exe
```


### Compilation et exécution en .NET 8

#### Avec Visual Studio

1. **Créer un nouveau projet**
   - Lancez Visual Studio
   - Sélectionnez File > New > Project...
   - Choisissez "Console App" (.NET Core)
   - Vérifiez que .NET 8.0 est sélectionné
   - Donnez un nom au projet et cliquez sur Create/Créer

2. **Écrire le code**
   - Visual Studio génère un modèle avec top-level statements par défaut

3. **Compiler et exécuter**
   - Identique à .NET Framework : utilisez Ctrl+F5 ou F5

#### Avec la CLI .NET

La CLI (Command Line Interface) .NET est un outil moderne pour travailler avec .NET :

1. **Créer un nouveau projet**
```shell script
dotnet new console -n HelloWorld
   cd HelloWorld
```


2. **Modifier le fichier Program.cs** avec votre code

3. **Exécuter le programme**
```shell script
dotnet run
```


4. **Compiler sans exécuter**
```shell script
dotnet build
```


5. **Publier l'application** (pour la distribution)
```shell script
# Pour un déploiement autonome (contient tout le runtime)
   dotnet publish -c Release -r win-x64 --self-contained true

   # Pour un déploiement dépendant du framework (nécessite .NET runtime)
   dotnet publish -c Release
```


### Processus de compilation

Il est utile de comprendre ce qui se passe lors de la compilation d'un programme C# :

#### En .NET Framework 4.7.2

1. **Le code source C#** (.cs) est analysé par le compilateur C# (`csc.exe`)
2. **Compilation en langage intermédiaire (IL)** et création d'un assembly (.exe ou .dll)
3. **Exécution par la CLR** (Common Language Runtime) qui compile le IL en code machine via JIT (Just-In-Time)

#### En .NET 8

1. **Le code source C#** est compilé par le compilateur Roslyn (`csc.dll`)
2. **Génération de code IL** et création d'un assembly (.dll)
3. **Exécution par CoreCLR** qui compile le IL en code machine

### Débogage de base

Le débogage est essentiel pour comprendre et corriger les programmes. Voici les techniques de base :

#### Points d'arrêt

- **Définir un point d'arrêt** : Cliquez dans la marge gauche ou appuyez sur F9
- L'exécution s'arrêtera à cette ligne, vous permettant d'inspecter l'état du programme

#### Exécution pas à pas

Quand le programme est arrêté à un point d'arrêt :
- **F10** : Exécuter la ligne actuelle et passer à la suivante (pas à pas principal)
- **F11** : Entrer dans une méthode (pas à pas détaillé)
- **Shift+F11** : Sortir de la méthode actuelle
- **F5** : Continuer l'exécution jusqu'au prochain point d'arrêt

#### Inspection des variables

- **Survolez une variable** avec la souris pour voir sa valeur
- Utilisez la fenêtre **Locals** pour voir toutes les variables locales
- Utilisez la fenêtre **Watch** pour surveiller des expressions spécifiques

### Messages d'erreur courants

Voici quelques erreurs fréquemment rencontrées et comment les résoudre :

1. **CS0103: Le nom 'X' n'existe pas dans le contexte actuel**
   - Vérifiez l'orthographe de la variable ou ajoutez `using` pour l'espace de noms manquant

2. **CS1002: ; attendu**
   - Vous avez oublié un point-virgule à la fin d'une instruction

3. **CS0029: Impossible de convertir implicitement le type 'X' en 'Y'**
   - Types incompatibles, utilisez une conversion explicite si approprié

4. **CS0117: 'X' ne contient pas de définition pour 'Y'**
   - La méthode ou propriété n'existe pas sur ce type, vérifiez la documentation

5. **CS0234: Le type ou le nom d'espace de noms 'X' n'existe pas dans l'espace de noms 'Y'**
   - Import manquant ou référence d'assembly manquante

### Différences de performance entre .NET Framework et .NET 8

.NET 8 offre plusieurs améliorations de performance par rapport à .NET Framework 4.7.2 :

- **Démarrage plus rapide** : Temps de chargement réduit
- **Empreinte mémoire plus faible** : Utilisation plus efficace de la mémoire
- **Compilation JIT améliorée** : Optimisations plus agressives
- **Garbage collector modernisé** : Moins de pauses et meilleure efficacité
- **API système optimisées** : Implémentations plus performantes

---

Félicitations ! Vous venez de créer, comprendre et exécuter votre premier programme C#. Ce premier pas est essentiel et constitue la base sur laquelle vous construirez vos compétences en programmation C#. Dans les chapitres suivants, nous explorerons les concepts plus avancés du langage, en commençant par les types de données et les structures de contrôle.

N'hésitez pas à expérimenter avec ce programme simple en modifiant le message affiché, en ajoutant des interactions avec l'utilisateur ou en essayant différentes méthodes de la classe `Console`. La pratique est la clé de l'apprentissage de la programmation.

⏭️ 2. [Fondamentaux du langage C#](/02-fondamentaux-du-langage-csharp/README.md)
