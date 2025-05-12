# 1.4. Installation de l'environnement de développement

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Pour développer en C#, vous aurez besoin d'un environnement de développement adapté. Il existe plusieurs options excellentes, chacune avec ses avantages spécifiques. Dans cette section, nous allons explorer les principales solutions et vous guider dans la configuration de votre environnement de développement C#.

## 1.4.1. Visual Studio (Community, Professional, Enterprise)

Visual Studio est l'IDE (Environnement de Développement Intégré) phare de Microsoft pour le développement C# et .NET. Il offre une expérience complète et optimisée pour le développement d'applications sur la plateforme .NET.

### Éditions disponibles

**Visual Studio Community**
- **Prix** : Gratuit pour les développeurs individuels, l'éducation, et les petites équipes
- **Caractéristiques** : Comprend la plupart des fonctionnalités essentielles
- **Idéal pour** : Étudiants, contributeurs open source, freelances, petites entreprises

**Visual Studio Professional**
- **Prix** : Abonnement payant (environ 45€/mois ou via les programmes Microsoft)
- **Caractéristiques** : Toutes les fonctionnalités de Community + outils professionnels supplémentaires
- **Idéal pour** : Développeurs professionnels et équipes de taille moyenne

**Visual Studio Enterprise**
- **Prix** : Abonnement premium (environ 250€/mois ou via les programmes Microsoft)
- **Caractéristiques** : Toutes les fonctionnalités de Professional + outils avancés de test, architecture et DevOps
- **Idéal pour** : Grandes organisations et équipes de développement complexes

### Principales fonctionnalités

- **IntelliSense** : Autocomplétion de code intelligente et contextuelle
- **Débogueur avancé** : Points d'arrêt, surveillance des variables, exécution pas à pas
- **Concepteurs visuels** : Pour WPF, Windows Forms, bases de données
- **Refactorisation de code** : Outils pour améliorer et restructurer le code
- **Analyse de code** : Détection automatique de problèmes potentiels
- **Intégration Git** : Gestion de version directement dans l'IDE
- **Extensions** : Vaste écosystème d'extensions pour étendre les fonctionnalités

### Installation de Visual Studio

1. **Téléchargement** : Visitez [le site officiel de Visual Studio](https://visualstudio.microsoft.com/fr/) et téléchargez l'édition souhaitée

2. **Lancement de l'installateur** : Exécutez le programme d'installation téléchargé

3. **Sélection des charges de travail** : Choisissez les composants à installer :
   - **Développement .NET Desktop** : Pour les applications Windows (WPF, Windows Forms)
   - **Développement web ASP.NET** : Pour les applications web et API
   - **Développement multiplateforme .NET** : Pour les applications mobiles et multiplateformes

   ![Charges de travail Visual Studio](https://via.placeholder.com/500x300?text=S%C3%A9lection+des+charges+de+travail)

4. **Installation** : Cliquez sur "Installer" et attendez que l'installation se termine (peut prendre entre 10 et 60 minutes selon votre connexion et les composants sélectionnés)

5. **Premier lancement** : Connectez-vous avec un compte Microsoft (optionnel) et choisissez vos préférences d'environnement

## 1.4.2. Visual Studio Code avec extensions C#

Visual Studio Code (VS Code) est un éditeur de code léger mais puissant, multiplateforme et gratuit. Bien que moins complet que Visual Studio pour le développement .NET, il offre une excellente alternative, particulièrement pour les développeurs travaillant sur différents systèmes d'exploitation.

### Avantages de VS Code

- **Léger et rapide** : Démarrage instantané et faible empreinte mémoire
- **Multiplateforme** : Fonctionne sur Windows, macOS et Linux
- **Gratuit et open source** : Accessible à tous sans restrictions
- **Hautement personnalisable** : Extensions nombreuses et configuration flexible
- **Intégration Git** : Support natif pour la gestion de version
- **Terminal intégré** : Exécution de commandes sans quitter l'éditeur

### Installation de VS Code et configuration pour C#

1. **Téléchargement et installation** : Rendez-vous sur [le site de VS Code](https://code.visualstudio.com/) et téléchargez la version correspondant à votre système d'exploitation

2. **Installation du SDK .NET** : Téléchargez et installez le [SDK .NET](https://dotnet.microsoft.com/download) approprié (généralement la dernière version LTS)

3. **Installation des extensions C#** :
   - Ouvrez VS Code
   - Allez dans la vue Extensions (Ctrl+Shift+X)
   - Recherchez et installez les extensions suivantes :
     - **C#** (extension officielle de Microsoft)
     - **C# Dev Kit** (suite d'outils C# complète)
     - **IntelliCode** (autocomplétion IA)
     - **.NET Core Test Explorer** (pour exécuter les tests)

   ![Extensions VS Code](https://via.placeholder.com/500x300?text=Extensions+VS+Code+pour+C%23)

4. **Configuration de OmniSharp** : L'extension C# utilise OmniSharp pour fournir des fonctionnalités comme l'autocomplétion et la navigation. La première fois que vous ouvrez un projet C#, VS Code téléchargera et configurera OmniSharp automatiquement.

### Utilisation et limites

VS Code est excellent pour :
- Le développement d'applications console
- Les API web avec ASP.NET Core
- Les bibliothèques .NET
- Les projets open source

Limitations par rapport à Visual Studio :
- Pas de concepteurs visuels intégrés (UI)
- Moins d'outils de débogage spécialisés
- Capacités de refactoring réduites
- Absence de certains outils spécifiques à .NET

## 1.4.3. Rider de JetBrains

JetBrains Rider est un IDE puissant pour le développement C# et .NET, offrant une alternative robuste à Visual Studio, particulièrement appréciée des développeurs habitués aux autres produits JetBrains comme IntelliJ IDEA ou PyCharm.

### Caractéristiques principales

- **Performance** : Souvent plus rapide que Visual Studio, particulièrement sur les grands projets
- **Multiplateforme** : Fonctionne sur Windows, macOS et Linux
- **Intelligence de code** : Inspection de code avancée et suggestions d'amélioration
- **Refactorisation puissante** : Plus de 60 opérations de refactorisation automatiques
- **Support complet de .NET** : Applications .NET Framework, .NET Core et .NET 5+
- **Intégration Unity** : Support dédié pour le développement de jeux Unity
- **Tests intégrés** : Exécution et débogage des tests unitaires
- **Interface utilisateur** : Design moderne avec thèmes personnalisables

### Installation de Rider

1. **Téléchargement** : Visitez [le site de JetBrains Rider](https://www.jetbrains.com/rider/) et téléchargez la version d'essai (30 jours) ou achetez une licence

2. **Installation** : Exécutez l'installateur et suivez les instructions à l'écran

3. **Installation du SDK .NET** : Si ce n'est pas déjà fait, téléchargez et installez le [SDK .NET](https://dotnet.microsoft.com/download)

4. **Premier lancement** : Configurez vos préférences et importez éventuellement des paramètres d'un autre produit JetBrains

### Licences et coûts

- **Essai gratuit** : 30 jours complets
- **Abonnement annuel** : À partir de 159€/an pour les particuliers
- **Abonnement pour organisations** : À partir de 269€/an par utilisateur
- **Licence gratuite** : Disponible pour les étudiants, enseignants, projets open source et startups éligibles

### Avantages par rapport à Visual Studio

- **Performance sur les gros projets**
- **Meilleure expérience multiplateforme** (particulièrement sur macOS et Linux)
- **Navigation dans le code plus intuitive**
- **Refactorisation de code plus avancée**
- **Moins de ressources système requises**

## 1.4.4. Configuration initiale et premiers pas

Quelle que soit l'option que vous choisissez, voici un guide pour configurer votre environnement et créer votre premier projet C#.

### Configuration de l'environnement

#### Configurer Visual Studio
1. **Personnalisation de l'interface** : Choisissez entre le thème clair ou sombre (Outils > Options > Environnement > Paramètres généraux)

2. **Extensions recommandées** :
   - ReSharper (payant, mais puissant pour la refactorisation)
   - CodeMaid (nettoyage et organisation du code)
   - Productivity Power Tools (ensemble d'outils de productivité)

3. **Paramètres de codage** :
   - Configurez les conventions de nommage (Outils > Options > Éditeur de texte > C# > Style de code)
   - Activez "Format on Save" pour un formatage automatique

#### Configurer VS Code
1. **settings.json** : Adaptez les paramètres spécifiques à C# :
```json
{
     "editor.formatOnSave": true,
     "omnisharp.enableRoslynAnalyzers": true,
     "omnisharp.enableEditorConfigSupport": true,
     "csharp.semanticHighlighting.enabled": true
   }
```


2. **Raccourcis clavier** : Personnalisez les raccourcis pour maximiser votre productivité

#### Configurer Rider
1. **Plugins** : Installez des plugins supplémentaires depuis le marketplace intégré

2. **Paramètres d'inspection** : Ajustez les règles de qualité de code selon vos préférences ou normes d'équipe

### Création d'un premier projet

#### Avec Visual Studio

1. **Créer un nouveau projet** :
   - Lancez Visual Studio
   - Cliquez sur "Créer un nouveau projet"
   - Filtrez pour les projets C# et sélectionnez "Application console"
   - Donnez un nom à votre projet et choisissez l'emplacement

   ![Nouveau projet Visual Studio](https://via.placeholder.com/500x300?text=Cr%C3%A9ation+de+projet+Visual+Studio)

2. **Écrire et exécuter du code** :
   - Le fichier Program.cs s'ouvre automatiquement
   - Il contient déjà une structure de base
   - Modifiez ou ajoutez du code
   - Appuyez sur F5 pour exécuter ou Ctrl+F5 pour exécuter sans débogage

```textmate
// Exemple de code pour votre premier projet
Console.WriteLine("Bonjour, C#!");
Console.Write("Entrez votre nom : ");
string nom = Console.ReadLine() ?? "Anonyme";
Console.WriteLine($"Bienvenue dans le monde de C#, {nom}!");
Console.ReadKey(); // Attend une touche avant de fermer
```


#### Avec VS Code

1. **Créer un nouveau projet** :
   - Ouvrez VS Code
   - Ouvrez un terminal (Ctrl+`)
   - Naviguez vers le dossier souhaité
   - Utilisez les commandes .NET CLI:
```shell script
dotnet new console -n MonPremierProjet
     cd MonPremierProjet
     code . # Ouvre le projet dans VS Code
```


2. **Exécuter le projet** :
   - Dans le terminal, tapez `dotnet run`
   - Ou utilisez le bouton de débogage (F5) après avoir configuré launch.json

#### Avec Rider

1. **Créer un nouveau projet** :
   - Lancez Rider
   - Cliquez sur "New Solution"
   - Sélectionnez "Console Application"
   - Configurez le nom, l'emplacement et la version .NET
   - Cliquez sur "Create"

2. **Exécuter le projet** :
   - Appuyez sur Shift+F10 ou cliquez sur le bouton "Run"
   - La fenêtre de console s'affiche avec le résultat

### Structure d'un projet C# de base

Un projet console C# minimal contient les éléments suivants :

- **Program.cs** : Point d'entrée de l'application
- **[NomProjet].csproj** : Fichier de projet XML définissant la configuration
- **obj/** et **bin/** : Dossiers pour les fichiers intermédiaires et compilés
- **global.json** (optionnel) : Spécifie la version du SDK .NET

### Conseils pour démarrer

1. **Explorez l'IDE** : Familiarisez-vous avec l'interface et les raccourcis clavier

2. **Utilisez l'aide contextuelle** : Survolez le code pour voir des informations, ou appuyez sur F1 pour la documentation

3. **Consultez les exemples** : Explorez les modèles de projets fournis avec votre IDE

4. **Débogage précoce** : Apprenez à utiliser le débogueur dès le début - posez des points d'arrêt (F9) et exécutez pas à pas (F10/F11)

5. **Solution Explorer / Explorateur de solutions** : Organisez vos fichiers et projets de façon logique

6. **Créez une solution** : Pour les projets plus complexes, créez une solution contenant plusieurs projets (bibliothèque de classes, tests, etc.)

### Résolution des problèmes courants

- **SDK manquant** : Assurez-vous d'avoir installé le SDK .NET approprié
- **Erreurs de restauration NuGet** : Vérifiez votre connexion Internet ou les sources NuGet
- **OmniSharp ne démarre pas** (VS Code) : Redémarrez VS Code ou vérifiez les journaux
- **IntelliSense ne fonctionne pas** : Reconstruisez la solution ou rechargez l'espace de travail

---

Avec votre environnement de développement configuré, vous êtes maintenant prêt à plonger dans l'apprentissage du langage C# et à commencer à créer vos premières applications. Dans les chapitres suivants, nous explorerons la syntaxe fondamentale du langage et les concepts de programmation orientée objet qui sont au cœur de C#.

⏭️
