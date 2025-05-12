# 14.1. Compilation et build

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La compilation et le processus de build sont des étapes fondamentales dans le développement d'applications .NET. Comprendre ces mécanismes vous aidera à optimiser vos applications et à mettre en place des processus de développement efficaces.

## 14.1.1. Configuration de build

### Qu'est-ce qu'une configuration de build ?

Une configuration de build est un ensemble de paramètres qui déterminent comment votre code source sera transformé en une application exécutable. Dans les projets .NET, les configurations de build sont principalement gérées via deux mécanismes :

1. **Fichiers de projet (.csproj, .vbproj, etc.)** : Définissent la structure et les paramètres de build du projet.
2. **Fichiers de configuration de solution (.sln)** : Coordonnent la compilation de plusieurs projets liés.

### Fichiers de projet modernes (.NET Core/.NET 5+)

Les projets .NET modernes utilisent un format XML simplifié appelé "SDK-style". Voici un exemple de fichier `.csproj` basique :

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

</Project>
```

#### Éléments clés d'un fichier projet

- **Sdk** : Définit le type de SDK utilisé pour le projet (.NET, Web, Azure Functions, etc.)
- **PropertyGroup** : Contient les propriétés générales du projet
  - **OutputType** : Type de sortie (Exe, Library, etc.)
  - **TargetFramework** : Version .NET ciblée (net8.0, netstandard2.1, etc.)
  - **Nullable** : Active ou désactive les références nullables
  - **ImplicitUsings** : Importe automatiquement les namespaces courants
- **ItemGroup** : Contient les références aux packages, fichiers, et projets

### Configurations prédéfinies

Par défaut, les projets .NET incluent deux configurations principales :

1. **Debug** : Optimisée pour le développement et le débogage
2. **Release** : Optimisée pour les performances en production

Vous pouvez spécifier des paramètres différents selon la configuration :

```xml
<PropertyGroup Condition="'$(Configuration)'=='Debug'">
  <DefineConstants>DEBUG;TRACE</DefineConstants>
  <Optimize>false</Optimize>
</PropertyGroup>

<PropertyGroup Condition="'$(Configuration)'=='Release'">
  <DefineConstants>TRACE</DefineConstants>
  <Optimize>true</Optimize>
</PropertyGroup>
```

### Configurations personnalisées

Vous pouvez également créer vos propres configurations pour des scénarios spécifiques :

1. Dans Visual Studio :
   - Ouvrez le Gestionnaire de Configuration (clic droit sur la solution → Propriétés → Configuration Manager)
   - Sélectionnez "New" dans la liste déroulante "Active solution configuration"

2. Directement dans le fichier `.csproj` :
   ```xml
   <PropertyGroup Condition="'$(Configuration)'=='Staging'">
     <DefineConstants>STAGING;TRACE</DefineConstants>
     <Optimize>true</Optimize>
   </PropertyGroup>
   ```

### Variables MSBuild et propriétés personnalisées

MSBuild (le moteur de build sous-jacent de .NET) prend en charge de nombreuses variables et vous permet de définir vos propres propriétés :

```xml
<PropertyGroup>
  <AppVersion>1.2.3</AppVersion>
  <Company>Ma Société</Company>
  <Authors>John Doe</Authors>
</PropertyGroup>
```

Vous pouvez ensuite utiliser ces propriétés dans votre code via des attributs d'assembly ou des directives de préprocesseur :

```csharp
// Utilisation via attributs d'assembly
[assembly: AssemblyVersion("$(AppVersion)")]
[assembly: AssemblyCompany("$(Company)")]

// Utilisation via directives de préprocesseur
#if DEBUG
    Console.WriteLine("Mode Debug activé");
#elif STAGING
    Console.WriteLine("Mode Staging activé");
#else
    Console.WriteLine("Mode Production activé");
#endif
```

### Construction multi-plateformes

Les projets .NET modernes facilitent le ciblage de plusieurs plateformes :

```xml
<PropertyGroup>
  <TargetFrameworks>net8.0;net6.0;netstandard2.1</TargetFrameworks>
</PropertyGroup>

<!-- Code spécifique à .NET 8 -->
<ItemGroup Condition="'$(TargetFramework)' == 'net8.0'">
  <PackageReference Include="NouveauPackage" Version="1.0.0" />
</ItemGroup>
```

## 14.1.2. Optimisation

L'optimisation du processus de build peut considérablement améliorer les performances de vos applications et réduire les temps de développement.

### Optimisations du compilateur

Le compilateur .NET (Roslyn) propose plusieurs niveaux d'optimisation :

1. **Optimisation de code**
   ```xml
   <PropertyGroup>
     <Optimize>true</Optimize>
   </PropertyGroup>
   ```

   Lorsque l'optimisation est activée, le compilateur effectue :
   - L'inlining des petites méthodes
   - L'élimination du code mort
   - La simplification des expressions
   - L'optimisation des boucles

2. **Suppression des informations de débogage**
   ```xml
   <PropertyGroup>
     <DebugType>none</DebugType> <!-- Options: full, pdbonly, portable, embedded, none -->
   </PropertyGroup>
   ```

3. **Ajustement des avertissements du compilateur**
   ```xml
   <PropertyGroup>
     <WarningLevel>4</WarningLevel> <!-- 0-4, 4 étant le plus strict -->
     <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
   </PropertyGroup>
   ```

### Réduction de la taille des assemblys

Pour minimiser la taille de vos applications :

1. **Trimming** (Élagage) - Disponible depuis .NET 5
   ```xml
   <PropertyGroup>
     <PublishTrimmed>true</PublishTrimmed>
     <TrimMode>link</TrimMode> <!-- Options: copyused, link -->
   </PropertyGroup>
   ```

   Le trimming supprime les parties du framework et des bibliothèques qui ne sont pas utilisées par votre application.

2. **IL Linking**
   ```xml
   <PropertyGroup>
     <PublishTrimmed>true</PublishTrimmed>
     <TrimmerRemoveSymbols>true</TrimmerRemoveSymbols>
   </PropertyGroup>
   ```

3. **Compression des assemblys**
   ```xml
   <PropertyGroup>
     <EnableCompressionInSingleFile>true</EnableCompressionInSingleFile>
   </PropertyGroup>
   ```

### Compilation Ahead-of-Time (AOT)

La compilation AOT transforme votre code en code machine natif lors du build, plutôt qu'à l'exécution :

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

Avantages de la compilation AOT :
- Démarrage plus rapide
- Empreinte mémoire réduite
- Meilleure sécurité (moins de métadonnées exposées)

Inconvénients :
- Taille de l'application potentiellement plus grande
- Certaines fonctionnalités dynamiques de .NET peuvent être limitées

### Optimisation des ressources

1. **Minimisation et bundling des fichiers statiques (pour les applications web)**

   Pour ASP.NET Core, utilisez le middleware de gestion des ressources statiques :
   ```csharp
   app.UseStaticFiles(new StaticFileOptions
   {
       OnPrepareResponse = ctx =>
       {
           // Cache pour 1 an (en secondes)
           ctx.Context.Response.Headers.Append("Cache-Control", "public,max-age=31536000");
       }
   });
   ```

2. **Compression des fichiers binaires et des images**

   Utilisez des outils comme `WebP` pour les images et compressez vos ressources statiques.

### Builds incrémentielles

Les builds incrémentielles ne recompilent que les fichiers modifiés, ce qui accélère considérablement le processus de développement.

```xml
<PropertyGroup>
  <IncrementalBuild>true</IncrementalBuild>
</PropertyGroup>
```

## 14.1.3. Différences Debug/Release

Comprendre les différences entre les configurations Debug et Release est essentiel pour développer et déployer efficacement vos applications .NET.

### Configuration Debug

La configuration Debug est optimisée pour le développement et le débogage :

```xml
<PropertyGroup Condition="'$(Configuration)'=='Debug'">
  <DefineConstants>DEBUG;TRACE</DefineConstants>
  <Optimize>false</Optimize>
  <DebugType>full</DebugType>
  <DebugSymbols>true</DebugSymbols>
</PropertyGroup>
```

#### Caractéristiques principales :

1. **Symboles de débogage complets**
   - Permet la correspondance précise entre le code source et le code exécuté
   - Prend en charge les points d'arrêt, le pas à pas, etc.

2. **Optimisations désactivées**
   - Le code est compilé sans optimisations
   - Facilite le débogage en maintenant la structure originale du code
   - Les variables ne sont pas éliminées, même si elles ne sont pas utilisées

3. **Constante de préprocesseur DEBUG**
   - Permet d'inclure du code spécifique au débogage :
   ```csharp
   #if DEBUG
       Console.WriteLine("Cette ligne n'apparaît qu'en mode Debug");
       VerifierConsistanceDonnees(); // Méthode de diagnostic coûteuse
   #endif
   ```

4. **Vérifications supplémentaires**
   - Vérifications d'assertions activées
   - Vérifications des limites de tableaux
   - Vérifications des pointeurs null
   - Validation des arguments plus stricte

### Configuration Release

La configuration Release est optimisée pour les performances et le déploiement :

```xml
<PropertyGroup Condition="'$(Configuration)'=='Release'">
  <DefineConstants>TRACE</DefineConstants>
  <Optimize>true</Optimize>
  <DebugType>pdbonly</DebugType> <!-- ou 'none' si aucun symbole n'est nécessaire -->
  <DebugSymbols>false</DebugSymbols>
</PropertyGroup>
```

#### Caractéristiques principales :

1. **Optimisations activées**
   - Inlining des méthodes courtes
   - Élimination des variables non utilisées
   - Déroulage des boucles
   - Amélioration des performances d'exécution

2. **Symboles de débogage limités ou absents**
   - Fichiers PDB générés séparément ou omis
   - Taille d'application réduite
   - Possibilité de déboguer en production si les PDB sont conservés

3. **Code DEBUG exclu**
   - Les blocs `#if DEBUG` sont ignorés lors de la compilation
   - Les assertions et autres vérifications coûteuses sont omises

4. **Meilleures performances**
   - Démarrage plus rapide
   - Utilisation de mémoire optimisée
   - Temps d'exécution réduit

### Comparaison des performances

Voici un tableau comparatif des performances typiques entre Debug et Release :

| Aspect | Debug | Release | Différence typique |
|--------|-------|---------|-------------------|
| Taille de l'exécutable | Plus grande | Plus petite | 30-50% plus petit en Release |
| Vitesse d'exécution | Plus lente | Plus rapide | 10-30% plus rapide en Release |
| Utilisation mémoire | Plus élevée | Plus basse | 10-20% moins de mémoire en Release |
| Démarrage application | Plus lent | Plus rapide | 5-15% plus rapide en Release |

### Comment tester en configuration Release

Il est important de tester régulièrement votre application en configuration Release pour identifier les problèmes qui pourraient n'apparaître qu'en production :

1. Dans Visual Studio :
   - Sélectionnez "Release" dans la liste déroulante de configuration
   - Exécutez l'application avec Ctrl+F5 (sans débogueur)

2. Avec la CLI .NET :
   ```bash
   dotnet build --configuration Release
   dotnet run --configuration Release
   ```

3. Pour déboguer une build Release :
   - Activez la génération de PDB : `<DebugType>pdbonly</DebugType>`
   - Exécutez avec le débogueur (F5 dans Visual Studio)
   - Notez que l'expérience de débogage sera limitée (pas de modification de variables, etc.)

## 14.1.4. CI/CD pour applications .NET

L'intégration continue (CI) et le déploiement continu (CD) sont des pratiques essentielles qui automatisent la construction, les tests et le déploiement de vos applications.

### Principes fondamentaux

1. **Intégration Continue (CI)** : Automatise la compilation et les tests à chaque push de code
2. **Livraison Continue** : Prépare automatiquement les versions pour le déploiement
3. **Déploiement Continu (CD)** : Déploie automatiquement les versions en production

### Outils populaires pour CI/CD avec .NET

1. **GitHub Actions**
2. **Azure DevOps Pipelines**
3. **Jenkins**
4. **GitLab CI/CD**
5. **TeamCity**
6. **CircleCI**

### Configuration CI/CD avec GitHub Actions

Voici un exemple de workflow GitHub Actions pour une application .NET :

1. Créez un fichier `.github/workflows/dotnet.yml` dans votre dépôt :

```yaml
name: .NET CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore --configuration Release

    - name: Test
      run: dotnet test --no-build --verbosity normal --configuration Release

    - name: Publish
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      run: dotnet publish --configuration Release --output ./publish

    - name: Upload artifacts
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      uses: actions/upload-artifact@v3
      with:
        name: app
        path: ./publish
```

### Configuration CI/CD avec Azure DevOps

Voici un exemple de pipeline Azure DevOps pour une application .NET :

1. Créez un fichier `azure-pipelines.yml` à la racine de votre dépôt :

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '8.0.x'

- task: DotNetCoreCLI@2
  displayName: Restore
  inputs:
    command: 'restore'

- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: 'build'
    arguments: '--configuration $(buildConfiguration) --no-restore'

- task: DotNetCoreCLI@2
  displayName: Test
  inputs:
    command: 'test'
    arguments: '--configuration $(buildConfiguration) --no-build'

- task: DotNetCoreCLI@2
  displayName: Publish
  inputs:
    command: 'publish'
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: true

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifacts'
  inputs:
    pathtoPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'
```

### Stratégies de branches et workflows

Voici une stratégie courante pour organiser vos branches et workflows CI/CD :

1. **Branches de fonctionnalité** :
   - Créées à partir de `develop`
   - Compile et exécute les tests à chaque push
   - Merge dans `develop` via Pull Request

2. **Branche `develop`** :
   - Intègre toutes les fonctionnalités terminées
   - Déploie automatiquement vers l'environnement de développement
   - Tests d'intégration automatisés

3. **Branche `release`** :
   - Créée à partir de `develop` pour préparer une release
   - Déploie automatiquement vers l'environnement de test/QA
   - Seules les corrections de bugs y sont autorisées

4. **Branche `main`** :
   - Représente le code en production
   - Déploie automatiquement vers la production
   - Protégée par des règles strictes

### Automatisation des tests

Une partie essentielle de la CI/CD est l'exécution de différents types de tests :

```yaml
- task: DotNetCoreCLI@2
  displayName: 'Run Unit Tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--filter Category=Unit'

- task: DotNetCoreCLI@2
  displayName: 'Run Integration Tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--filter Category=Integration'
```

### Gestion des versions et tags

Automatisez la gestion des versions avec des outils comme GitVersion :

```yaml
- task: UseGitVersion@5
  inputs:
    versionSpec: '5.x'

- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    arguments: '--configuration $(buildConfiguration) /p:Version=$(GitVersion.NuGetVersion)'
```

### Déploiement avec approche "Infrastructure as Code"

Utilisez des outils comme ARM Templates, Bicep ou Terraform pour définir votre infrastructure :

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'Azure-Connection'
    resourceGroupName: 'MyResourceGroup'
    location: 'West Europe'
    templateLocation: 'Linked artifact'
    csmFile: '$(Build.SourcesDirectory)/infra/template.json'
    overrideParameters: '-webAppName MyWebApp-$(Environment)'
```

### Stratégies de déploiement avancées

Pour minimiser les temps d'arrêt et risques, utilisez des stratégies de déploiement avancées :

1. **Blue-Green Deployment**
   - Deux environnements identiques (blue et green)
   - Les nouvelles versions sont déployées sur l'environnement inactif
   - Bascule du trafic entre les deux environnements

2. **Canary Releases**
   - Déploiement progressif vers un sous-ensemble d'utilisateurs
   - Surveillance des métriques avant le déploiement complet
   - Rollback facile en cas de problème

3. **Feature Flags**
   - Activer/désactiver les fonctionnalités sans redéploiement
   - Test A/B de nouvelles fonctionnalités
   - Déploiement découplé de l'activation

## Résumé

La compilation et le processus de build sont des aspects fondamentaux du développement .NET qui peuvent considérablement influencer la qualité, les performances et la maintenabilité de vos applications. Les points clés à retenir :

1. **Configurations de build** : Comprendre et personnaliser vos fichiers de projet pour différents scénarios
2. **Optimisation** : Utiliser les outils d'optimisation pour améliorer performances et taille
3. **Debug vs Release** : Connaître les différences pour développer et déployer efficacement
4. **CI/CD** : Automatiser compilation, tests et déploiement pour un workflow de développement fluide

En maîtrisant ces concepts, vous pourrez créer des applications .NET performantes et mettre en place des processus de développement efficaces.

## Exercices pratiques

1. Créez un projet .NET avec deux configurations personnalisées : "Staging" et "Production"
2. Ajoutez du code conditionnel qui s'exécute uniquement dans certaines configurations
3. Configurez un pipeline CI/CD simple avec GitHub Actions pour votre projet
4. Mesurez et comparez les performances de votre application en mode Debug vs Release
5. Explorez les options de trimming pour réduire la taille de votre application déployée

⏭️
