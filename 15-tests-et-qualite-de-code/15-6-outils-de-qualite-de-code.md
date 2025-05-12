# 15.6. Outils de qualité de code

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

La qualité du code est un aspect fondamental du développement logiciel professionnel. Un code de haute qualité est plus facile à maintenir, à étendre et présente moins de bugs. Heureusement, dans l'écosystème .NET, plusieurs outils existent pour vous aider à maintenir et améliorer la qualité de votre code C#.

Dans ce chapitre, nous explorerons les outils de qualité de code les plus populaires et puissants, et comment les intégrer dans votre flux de travail quotidien et vos pipelines d'intégration continue.

## 15.6.1. SonarQube

SonarQube est l'une des plateformes d'analyse de code les plus complètes et populaires, qui prend en charge de nombreux langages, dont C#.

### Qu'est-ce que SonarQube ?

SonarQube est un outil d'analyse de code statique qui inspecte votre code pour détecter :
- Les bugs potentiels
- Les vulnérabilités de sécurité
- Les "code smells" (pratiques de code indésirables)
- La duplication de code
- La couverture de code par les tests
- La complexité du code

### Installation et configuration

#### Installation de SonarQube

Il existe plusieurs façons d'installer SonarQube :

1. **Installation locale** :
   - Téléchargez SonarQube depuis [le site officiel](https://www.sonarqube.org/downloads/)
   - Décompressez l'archive
   - Configurez une base de données (PostgreSQL recommandé)
   - Lancez SonarQube avec le script de démarrage approprié

2. **Docker** (approche recommandée pour les débutants) :
   ```bash
   docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
   ```

3. **Cloud** : SonarCloud est la version hébergée de SonarQube
   - Inscrivez-vous sur [SonarCloud](https://sonarcloud.io/)
   - Pas besoin d'installation, tout est géré dans le cloud

#### Configuration initiale

Après l'installation :

1. Accédez à `http://localhost:9000` (ou l'URL appropriée)
2. Connectez-vous avec les identifiants par défaut (admin/admin)
3. Changez immédiatement le mot de passe par défaut
4. Créez un projet et générez un token d'accès

### Analyse d'un projet .NET

#### Installation du scanner SonarQube

Pour analyser un projet .NET, vous aurez besoin du scanner SonarQube :

```bash
dotnet tool install --global dotnet-sonarscanner
```

#### Analyser un projet

```bash
# Démarrer l'analyse
dotnet sonarscanner begin /k:"NomDuProjet" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="votre-token"

# Compiler le projet
dotnet build

# Terminer l'analyse
dotnet sonarscanner end /d:sonar.login="votre-token"
```

Pour inclure la couverture de code :

```bash
# Démarrer l'analyse avec configuration de couverture
dotnet sonarscanner begin /k:"NomDuProjet" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="votre-token" /d:sonar.cs.opencover.reportsPaths="**/coverage.opencover.xml"

# Compiler le projet
dotnet build

# Exécuter les tests avec couverture
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover

# Terminer l'analyse
dotnet sonarscanner end /d:sonar.login="votre-token"
```

### Interprétation des résultats

Après l'analyse, accédez au tableau de bord de SonarQube et explorez les résultats :

1. **Vue d'ensemble** : Affiche un résumé des métriques de qualité
2. **Issues** : Liste tous les problèmes détectés
3. **Measures** : Affiche des métriques détaillées (complexité, duplication, etc.)
4. **Code** : Permet de naviguer dans le code source et voir les problèmes in-situ

#### Catégories de problèmes

SonarQube classe les problèmes en trois catégories :

1. **Bugs** : Problèmes qui vont ou pourraient causer un comportement incorrect
2. **Vulnérabilités** : Problèmes de sécurité potentiels
3. **Code Smells** : Problèmes de maintenabilité qui rendent le code difficile à comprendre ou modifier

Chaque problème est également classé par sévérité :
- Bloqueur
- Critique
- Majeur
- Mineur
- Info

### Configuration personnalisée

Vous pouvez personnaliser les règles d'analyse en créant un profil de qualité :

1. Accédez à Administration > Quality Profiles
2. Créez un nouveau profil basé sur le profil Sonar Way
3. Activez/désactivez des règles selon vos besoins
4. Associez le profil à votre projet

Pour une configuration plus avancée, créez un fichier `sonar-project.properties` à la racine de votre projet :

```properties
sonar.projectKey=NomDuProjet
sonar.projectName=Nom Du Projet
sonar.projectVersion=1.0

sonar.sources=src
sonar.tests=tests
sonar.exclusions=**/bin/**,**/obj/**,**/*.Designer.cs
sonar.coverage.exclusions=**/*Tests.cs,**/Program.cs

sonar.cs.opencover.reportsPaths=**/coverage.opencover.xml
```

### Bonnes pratiques avec SonarQube

1. **Définissez une Quality Gate** : Une "Quality Gate" définit les critères minimaux de qualité que votre code doit satisfaire
2. **Analysez régulièrement** : Idéalement dans votre pipeline CI/CD
3. **Fixez les problèmes progressivement** : Commencez par les problèmes critiques et bloquants
4. **Ne désactivez pas les règles sans réflexion** : Si une règle est gênante, comprenez pourquoi avant de la désactiver
5. **Utilisez les exclusions avec parcimonie** : Préférez corriger les problèmes plutôt que de les exclure

## 15.6.2. NDepend

NDepend est un outil d'analyse de code puissant spécifiquement conçu pour la plateforme .NET. Il offre des analyses approfondies et des visualisations sophistiquées.

### Qu'est-ce que NDepend ?

NDepend est un outil commercial qui se distingue par :
- Des analyses détaillées de la structure du code
- Un langage de requête dédié (CQLinq) pour explorer le code
- Des visualisations avancées (matrices de dépendances, graphes)
- Une intégration étroite avec Visual Studio
- Des centaines de règles de qualité de code prédéfinies

### Installation et configuration

1. **Téléchargez NDepend** depuis le [site officiel](https://www.ndepend.com/download)
2. **Installez l'extension Visual Studio** ou utilisez la version standalone
3. **Activez votre licence** ou utilisez la version d'essai

### Analyse d'un projet avec NDepend

#### Dans Visual Studio

1. Cliquez sur le menu NDepend > Attach New NDepend Project to VS Solution
2. Configurez les options d'analyse selon vos besoins
3. Cliquez sur "Analyze"

#### En ligne de commande

```bash
NDepend.Console.exe "CheminVersVotreSolution.sln" /OutDir "CheminVersRapport"
```

### Fonctionnalités principales

#### 1. Tableau de bord et métriques

NDepend fournit un tableau de bord complet avec des métriques de qualité :
- Dettes techniques (en temps de développement)
- Métriques de qualité (complexité, cohésion, couplage)
- Tendances au fil du temps

#### 2. Code Query LINQ (CQLinq)

CQLinq est un langage de requête puissant pour explorer votre code :

```csharp
// Exemple : trouver les méthodes trop complexes
from m in Methods
where m.CyclomaticComplexity > 15
orderby m.CyclomaticComplexity descending
select new { m, m.CyclomaticComplexity }
```

#### 3. Visualisations

NDepend offre plusieurs visualisations utiles :

1. **Matrice de dépendances** : Montre les dépendances entre assemblies/namespaces/classes
2. **Graphe de dépendances** : Visualisation des dépendances sous forme de graphe
3. **Treemap métrique** : Représentation visuelle de métriques sur l'ensemble du code
4. **Evolution temporelle** : Suivi de l'évolution des métriques dans le temps

#### 4. Règles et contraintes

NDepend inclut plus de 200 règles prédéfinies regroupées en catégories :
- Architecture
- Maintenabilité
- Performances
- Convention de nommage
- etc.

Vous pouvez aussi créer vos propres règles avec CQLinq :

```csharp
// Règle personnalisée : les classes dans un namespace spécifique doivent être sealed
warnif count > 0 from t in Types
where t.NameLike("MyCompany.Security") &&
      !t.IsSealed &&
      !t.IsStatic
select t
```

### Interprétation des résultats

NDepend génère un rapport HTML détaillé avec :
1. **Résumé des problèmes** : Classés par criticité
2. **Dette technique** : Estimation du temps nécessaire pour corriger les problèmes
3. **Violations des règles** : Liste détaillée des violations avec explications
4. **Tendances** : Évolution de la qualité du code dans le temps

### Intégration avec l'environnement de développement

NDepend s'intègre bien dans votre workflow :
1. **Visual Studio** : Extension dédiée
2. **Azure DevOps** : Extension disponible
3. **Jenkins/TeamCity** : Via ligne de commande ou plugins
4. **SonarQube** : Complémentaire à NDepend (ne remplace pas SonarQube)

### Bonnes pratiques avec NDepend

1. **Analysez régulièrement** : Intégrez NDepend dans votre CI/CD
2. **Utilisez les tendances** : Suivez l'évolution de la qualité du code
3. **Adaptez les règles** : Configurez les règles en fonction de votre contexte
4. **Explorez avec CQLinq** : Utilisez CQLinq pour investiguer des problèmes spécifiques
5. **Formation de l'équipe** : Formez votre équipe à l'interprétation des résultats

## 15.6.3. ReSharper/Rider

ReSharper est une extension pour Visual Studio, tandis que Rider est un IDE complet. Les deux sont développés par JetBrains et offrent des fonctionnalités avancées d'analyse et d'amélioration de code.

### ReSharper

#### Qu'est-ce que ReSharper ?

ReSharper est une extension premium pour Visual Studio qui offre :
- Des analyses de code en temps réel
- Des suggestions d'amélioration de code
- Des refactorings automatisés
- Une navigation améliorée dans le code
- Des outils de productivité

#### Installation et configuration

1. **Téléchargez ReSharper** depuis le [site de JetBrains](https://www.jetbrains.com/resharper/download/)
2. **Installez l'extension** en suivant les instructions
3. **Activez votre licence** ou utilisez la version d'essai

#### Fonctionnalités principales

1. **Analyse de code en temps réel** :
   - Détection de problèmes potentiels pendant que vous tapez
   - Suggestions d'améliorations de code
   - Vérification des conventions de codage

2. **Refactoring** :
   - Extraction de méthode/interface/classe
   - Renommage intelligent (prend en compte toutes les références)
   - Changement de signature
   - Déplacement de code

3. **Navigation** :
   - Accès rapide aux déclarations et implémentations
   - Recherche avancée (tout, partout)
   - Structure hiérarchique des classes et appels

4. **Génération de code** :
   - Complétion de code intelligente
   - Templates de code
   - Génération automatique de constructeurs, propriétés, etc.

5. **Tests unitaires** :
   - Exécution et débogage de tests dans l'IDE
   - Couverture de code
   - Génération de tests

#### Inspection du code

ReSharper offre de nombreuses inspections de code :

```csharp
// ReSharper détectera que cette condition est toujours vraie
if (x != null || x == null)
{
    // Code
}

// ReSharper suggérera d'utiliser var
string name = "John";  // Suggestion: utiliser var

// ReSharper détectera les conversions redondantes
int x = (int)10;  // Conversion inutile
```

Pour une vérification complète du projet :
1. ReSharper > Inspect > Code Issues in Solution
2. Examinez les résultats et corrigez les problèmes

#### Nettoyage de code

Le "Code Cleanup" de ReSharper permet d'appliquer automatiquement plusieurs corrections :

1. ReSharper > Tools > Cleanup Code
2. Choisissez un profil de nettoyage ou créez-en un personnalisé
3. Sélectionnez la portée (fichier, projet, solution)
4. Exécutez le nettoyage

#### Profils de configuration

Vous pouvez créer et partager des profils de configuration ReSharper :

1. ReSharper > Options > Code Editing > C#
2. Configurez vos préférences
3. Exportez les paramètres (sous forme de fichier DotSettings)
4. Partagez avec votre équipe (idéalement via le contrôle de source)

### Rider

#### Qu'est-ce que Rider ?

Rider est un IDE complet pour .NET qui intègre toutes les fonctionnalités de ReSharper, plus :
- Un éditeur basé sur la plateforme IntelliJ
- Un débogueur intégré
- Une prise en charge native de .NET, .NET Core et .NET Framework
- Le support pour ASP.NET, Xamarin, Unity, etc.

#### Installation et configuration

1. **Téléchargez Rider** depuis le [site de JetBrains](https://www.jetbrains.com/rider/download/)
2. **Installez l'application** en suivant les instructions
3. **Activez votre licence** ou utilisez la version d'essai

#### Fonctionnalités principales

Rider inclut toutes les fonctionnalités de ReSharper, plus :

1. **Interface utilisateur moderne** :
   - Interface utilisateur personnalisable
   - Thèmes sombres et clairs
   - Disposition flexible des fenêtres

2. **Performance améliorée** :
   - Conçu pour gérer de grandes solutions
   - Analyse de code en arrière-plan
   - Démarrage rapide

3. **Outils intégrés** :
   - Terminal intégré
   - Contrôle de source (Git, SVN)
   - REST Client
   - Base de données

4. **Support multiplateforme** :
   - Windows, macOS, Linux
   - Docker intégré
   - Déploiement distant

#### Inspection et nettoyage de code

Rider propose les mêmes fonctionnalités d'inspection et de nettoyage que ReSharper :

1. Code > Inspect Code
2. Examinez les résultats dans l'outil "Inspection Results"
3. Corrigez les problèmes individuellement ou en lot

Pour le nettoyage de code :
1. Code > Code Cleanup
2. Choisissez un profil et une portée
3. Exécutez le nettoyage

### Comparaison entre ReSharper et Rider

| Fonctionnalité | ReSharper | Rider |
|----------------|-----------|-------|
| Type | Extension VS | IDE complet |
| Plateforme | Windows uniquement | Windows, macOS, Linux |
| Analyse de code | ✅ | ✅ |
| Refactoring | ✅ | ✅ |
| Performance | Peut ralentir VS | Optimisé |
| Débogueur | Utilise VS | Intégré |
| Contrôle de source | Via VS | Intégré |
| Support Unity | Via plugin | Intégré |

### Bonnes pratiques avec ReSharper/Rider

1. **Standardisez les règles d'équipe** : Partagez un fichier DotSettings commun
2. **Utilisez des raccourcis clavier** : Apprenez les raccourcis pour améliorer votre productivité
3. **Nettoyage de code régulier** : Incluez le nettoyage dans votre routine
4. **Personnalisez selon vos besoins** : Adaptez les règles à votre contexte
5. **Utilisez les solutions de ReSharper** : Acceptez les suggestions quand elles ont du sens

## 15.6.4. Intégration dans les pipelines CI/CD

L'intégration des outils de qualité de code dans vos pipelines CI/CD (Continuous Integration/Continuous Deployment) permet d'automatiser les vérifications de qualité et de maintenir des standards cohérents.

### Principes de base

Les vérifications de qualité de code devraient être exécutées :
1. À chaque commit (pour des vérifications rapides)
2. À chaque pull request (pour des vérifications plus approfondies)
3. Sur la branche principale (pour des analyses complètes)

### Intégration avec Azure DevOps

#### SonarQube dans Azure DevOps

```yaml
# azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  solution: '**/*.sln'

steps:
- task: SonarQubePrepare@4
  inputs:
    SonarQube: 'SonarQube'
    scannerMode: 'MSBuild'
    projectKey: 'MonProjet'
    projectName: 'Mon Projet'
    extraProperties: |
      sonar.cs.opencover.reportsPaths=$(Build.SourcesDirectory)/TestResults/Coverage/coverage.opencover.xml
      sonar.coverage.exclusions=**/*Tests.cs

- task: DotNetCoreCLI@2
  displayName: 'Build'
  inputs:
    command: 'build'
    projects: '$(solution)'
    arguments: '--configuration $(buildConfiguration)'

- task: DotNetCoreCLI@2
  displayName: 'Test with coverage'
  inputs:
    command: 'test'
    projects: '**/*Tests/*.csproj'
    arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover'

- task: SonarQubeAnalyze@4

- task: SonarQubePublish@4
  inputs:
    pollingTimeoutSec: '300'
```

#### NDepend dans Azure DevOps

Pour intégrer NDepend dans Azure DevOps, utilisez l'extension NDepend ou la ligne de commande :

```yaml
# azure-pipelines.yml avec NDepend
steps:
# Autres étapes (build, test, etc.)

- task: PowerShell@2
  displayName: 'Run NDepend Analysis'
  inputs:
    targetType: 'inline'
    script: |
      & "$(NDependConsolePath)" "$(Build.SourcesDirectory)/MySolution.sln" /OutDir "$(Build.ArtifactStagingDirectory)/NDepend"

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)/NDepend'
    artifactName: 'NDepend Analysis'
```

#### ReSharper Command Line Tools

JetBrains fournit des outils en ligne de commande pour intégrer les vérifications ReSharper dans vos pipelines :

```yaml
# azure-pipelines.yml avec InspectCode
steps:
# Autres étapes

- task: PowerShell@2
  displayName: 'Run ReSharper InspectCode'
  inputs:
    targetType: 'inline'
    script: |
      dotnet tool install -g JetBrains.ReSharper.GlobalTools
      jb inspectcode "$(Build.SourcesDirectory)/MySolution.sln" -o="$(Build.ArtifactStagingDirectory)/resharper-report.xml" --format=Xml

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)/resharper-report.xml'
    artifactName: 'ReSharper Analysis'
```

### Intégration avec GitHub Actions

#### SonarCloud avec GitHub Actions

```yaml
# .github/workflows/build.yml
name: Build
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    name: Build
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Important pour SonarCloud

      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 6.0.x

      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11

      - name: Cache SonarCloud packages
        uses: actions/cache@v1
        with:
          path: ~\sonar\cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar

      - name: Install SonarCloud scanner
        run: dotnet tool install --global dotnet-sonarscanner

      - name: Begin SonarCloud analysis
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: dotnet sonarscanner begin /k:"MonOrganisation_MonProjet" /o:"mon-organisation" /d:sonar.login="${{ secrets.SONAR_TOKEN }}" /d:sonar.host.url="https://sonarcloud.io"

      - name: Build
        run: dotnet build --configuration Release

      - name: Test with coverage
        run: dotnet test --configuration Release --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover

      - name: End SonarCloud analysis
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: dotnet sonarscanner end /d:sonar.login="${{ secrets.SONAR_TOKEN }}"
```

#### ReSharper Command Line avec GitHub Actions

```yaml
# .github/workflows/code-quality.yml
name: Code Quality Check

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  resharper-analysis:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 6.0.x

    - name: Install ReSharper command line tools
      run: dotnet tool install -g JetBrains.ReSharper.GlobalTools

    - name: Run ReSharper inspectcode
      run: jb inspectcode MySolution.sln -o="resharper-report.xml" --format=Xml

    - name: Upload ReSharper results
      uses: actions/upload-artifact@v2
      with:
        name: resharper-analysis
        path: resharper-report.xml
```

### Intégration avec Jenkins

#### SonarQube avec Jenkins

```groovy
// Jenkinsfile
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        dotnet sonarscanner begin /k:"MonProjet" /d:sonar.host.url="${SONAR_HOST_URL}" /d:sonar.login="${SONAR_AUTH_TOKEN}"
                        dotnet build
                        dotnet test --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover
                        dotnet sonarscanner end /d:sonar.login="${SONAR_AUTH_TOKEN}"
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
```

### Quality Gates dans vos pipelines

Les "Quality Gates" sont des critères de qualité minimaux que votre code doit satisfaire pour que le build soit considéré comme réussi.

#### Exemple de Quality Gate avec SonarQube

Dans SonarQube, créez une Quality Gate avec des conditions comme :
- Couverture de code > 80%
- Aucun bug critique ou bloquant
- Dette technique < 5 jours

Configurez ensuite votre pipeline pour échouer si la Quality Gate n'est pas satisfaite :

```yaml
# Azure DevOps
- task: SonarQubePublish@4
  inputs:
    pollingTimeoutSec: '300'

- task: PowerShell@2
  displayName: 'Check Quality Gate'
  inputs:
    targetType: 'inline'
    script: |
      $status = Invoke-RestMethod -Uri "$(SonarQubeURL)/api/qualitygates/project_status?projectKey=MonProjet" -Headers @{Authorization = "Bearer $(SonarQubeToken)"}
      if ($status.projectStatus.status -ne "OK") {
        Write-Error "Quality Gate failed: $($status.projectStatus.status)"
        exit 1
      }
```

### Bonnes pratiques pour l'intégration CI/CD

1. **Automatisez tout** : Toutes les vérifications de qualité devraient être automatisées
2. **Échouez rapidement** : Les problèmes critiques devraient faire échouer le build
3. **Équilibrez rigueur et pragmatisme** : Définissez des standards réalistes
4. **Évolution progressive** : Commencez avec des règles de base, puis augmentez progressivement l'exigence
5. **Rapports clairs** : Assurez-vous que les rapports de qualité sont facilement accessibles
6. **Ne cassez pas le build pour des problèmes mineurs** : Utilisez des niveaux de sévérité appropriés

## 15.6.5. Surveillance continue de la qualité

La surveillance continue de la qualité du code consiste à suivre l'évolution des métriques de qualité au fil du temps pour identifier les tendances et prendre des mesures correctives si nécessaire.

### Suivi des tendances

#### Dashboards SonarQube

SonarQube offre des tableaux de bord pour suivre l'évolution des métriques :
1. Accédez à votre projet dans SonarQube
2. Cliquez sur "Measures" > "Activity"
3. Sélectionnez les métriques à suivre (bugs, vulnérabilités, dette technique, etc.)
4. Observez les graphiques pour identifier les tendances

#### Rapports NDepend

NDepend génère des rapports de tendance qui montrent l'évolution de la qualité :
1. Exécutez plusieurs analyses NDepend au fil du temps
2. Consultez la section "Trend" du rapport
3. Identifiez les métriques qui s'améliorent ou se dégradent

### Notifications et alertes

#### Alertes SonarQube

Configurez des notifications pour être alerté en cas de problèmes :
1. Administration > Configuration > Emails
2. Activez les notifications
3. Configurez les événements qui déclenchent des alertes (nouvelles vulnérabilités, dégradation de Quality Gate, etc.)

#### Intégration Slack/Teams

Intégrez vos outils de qualité avec vos plateformes de communication :

```yaml
# Azure DevOps avec notification Teams
- task: PowerShell@2
  displayName: 'Notify Teams'
  condition: failed()
  inputs:
    targetType: 'inline'
    script: |
      $body = @{
        "@type" = "MessageCard"
        "@context" = "http://schema.org/extensions"
        "themeColor" = "0078D7"
        "title" = "Build $(Build.BuildNumber) failed Quality Gate"
        "text" = "The build $(Build.BuildNumber) for $(Build.Repository.Name) failed the Quality Gate. [View Results]($(SonarQubeURL)/dashboard?id=$(SonarProjectKey))"
      } | ConvertTo-Json

      Invoke-RestMethod -Uri "$(TeamsWebhookUrl)" -Method Post -Body $body -ContentType "application/json"
```

### Stratégies de remédiation

#### Plans d'action SonarQube

SonarQube permet de créer des plans d'action pour gérer les problèmes détectés :
1. Accédez à l'onglet "Issues"
2. Filtrez les problèmes par criticité
3. Assignez des problèmes à des développeurs
4. Définissez des échéances

#### Focus sur la dette technique

Priorisez la réduction de la dette technique :

1. **Règle du scout** : Laissez le code dans un meilleur état que vous ne l'avez trouvé
2. **Budget de refactoring** : Consacrez un pourcentage du temps de développement au remboursement de la dette technique
3. **Remboursement opportuniste** : Corrigez les problèmes dans le code sur lequel vous travaillez déjà
4. **Refactoring ciblé** : Identifiez les zones critiques avec le plus de dette et concentrez vos efforts
5. **Prévention** : Adoptez des pratiques qui évitent d'accumuler de la dette technique

### Revues de code efficaces

Les revues de code sont essentielles pour maintenir la qualité :

#### Automatisation des vérifications

Utilisez des outils pour automatiser les vérifications répétitives :

```yaml
# GitHub Actions pour les PR
name: Code Review

on:
  pull_request:
    branches: [ main ]

jobs:
  code-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 6.0.x

      - name: Install tools
        run: |
          dotnet tool install -g dotnet-format
          dotnet tool install -g JetBrains.ReSharper.GlobalTools

      - name: Check formatting
        run: dotnet format --verify-no-changes

      - name: Run ReSharper inspections
        run: jb inspectcode MySolution.sln -o="resharper-report.xml" --format=Xml

      - name: Comment PR with issues
        uses: actions/github-script@v5
        with:
          script: |
            const fs = require('fs');
            const xml2js = require('xml2js');

            const reportXml = fs.readFileSync('resharper-report.xml', 'utf8');
            let report;
            xml2js.parseString(reportXml, (err, result) => {
              report = result;
            });

            const issues = report.Report.IssueTypes[0].IssueType;
            if (issues.length > 0) {
              let comment = '### ReSharper Code Review\n\n';
              comment += 'I found the following issues:\n\n';

              issues.forEach(issue => {
                comment += `- **${issue.$.Severity}**: ${issue.$.Description}\n`;
              });

              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: comment
              });
            }
```

#### Checklists de revue

Créez des checklists pour les revues de code :

```markdown
## Code Review Checklist

### Général
- [ ] Le code suit les guidelines de l'équipe
- [ ] La complexité est minimale
- [ ] Pas de duplication de code
- [ ] Nommage clair et significatif

### Performance
- [ ] Pas de requêtes N+1
- [ ] Optimisation des requêtes LINQ
- [ ] Utilisation appropriée des collections

### Sécurité
- [ ] Validation des entrées
- [ ] Prévention des injections SQL
- [ ] Gestion sécurisée des authentifications

### Tests
- [ ] Tests unitaires pour la nouvelle fonctionnalité
- [ ] Couverture de code suffisante
- [ ] Cas limites testés
```

### Gamification de la qualité

Rendez l'amélioration de la qualité du code plus engageante :

1. **Tableaux de bord d'équipe** : Affichez les métriques de qualité sur des écrans visibles
2. **Objectifs collectifs** : Définissez des objectifs d'équipe pour la réduction des bugs/vulnérabilités
3. **Compétitions amicales** : Organisez des "bug squash days" ou des défis de refactoring
4. **Reconnaissance** : Célébrez les améliorations significatives

### Métriques à suivre dans le temps

Voici les métriques clés à surveiller :

1. **Dette technique** : Temps estimé pour corriger tous les problèmes
2. **Couverture de code** : Pourcentage de code couvert par des tests
3. **Complexité cyclomatique** : Complexité des méthodes et classes
4. **Duplications** : Pourcentage de code dupliqué
5. **Bugs et vulnérabilités** : Nombre de problèmes critiques
6. **Violations de règles** : Tendance des problèmes de code

### Amélioration continue

L'amélioration de la qualité du code est un processus continu :

1. **Rétroactions régulières** : Organisez des sessions pour discuter de la qualité du code
2. **Ajustement des règles** : Affinez vos règles en fonction de votre contexte
3. **Formation continue** : Organisez des sessions de formation sur les bonnes pratiques
4. **Documentation des décisions** : Expliquez pourquoi certaines règles sont adoptées/ignorées

## Mise en pratique : Exemple complet

Voyons comment mettre en place une stratégie complète de qualité de code pour un projet .NET :

### Étape 1 : Configuration initiale

1. **Standardisez le style de code** :
   - Créez un fichier `.editorconfig` :

```editorconfig
# EditorConfig is awesome: https://EditorConfig.org

root = true

[*]
charset = utf-8
end_of_line = crlf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

[*.cs]
# Naming styles
dotnet_naming_rule.interface_should_be_begins_with_i.severity = suggestion
dotnet_naming_rule.interface_should_be_begins_with_i.symbols = interface
dotnet_naming_rule.interface_should_be_begins_with_i.style = begins_with_i

dotnet_naming_symbols.interface.applicable_kinds = interface
dotnet_naming_symbols.interface.applicable_accessibilities = public, internal, private, protected, protected_internal

dotnet_naming_style.begins_with_i.required_prefix = I
dotnet_naming_style.begins_with_i.required_suffix =
dotnet_naming_style.begins_with_i.word_separator =
dotnet_naming_style.begins_with_i.capitalization = pascal_case

# Code style
csharp_style_var_for_built_in_types = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = true:suggestion

# Expression-bodied members
csharp_style_expression_bodied_methods = when_on_single_line:suggestion
csharp_style_expression_bodied_constructors = false:none
csharp_style_expression_bodied_operators = when_on_single_line:suggestion
csharp_style_expression_bodied_properties = true:suggestion
csharp_style_expression_bodied_indexers = true:suggestion
csharp_style_expression_bodied_accessors = true:suggestion
```

2. **Installez les packages d'analyse** :

```xml
<!-- Dans votre fichier .csproj -->
<PropertyGroup>
  <EnableNETAnalyzers>true</EnableNETAnalyzers>
  <AnalysisLevel>latest</AnalysisLevel>
  <AnalysisMode>All</AnalysisMode>
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
</PropertyGroup>

<ItemGroup>
  <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" Version="6.0.0" />
  <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.435" PrivateAssets="all" />
  <AdditionalFiles Include="$(SolutionDir)stylecop.json" Link="stylecop.json" />
</ItemGroup>
```

3. **Configurez SonarQube** :
   - Créez un fichier `sonar-project.properties` :

```properties
sonar.projectKey=MonEntreprise_MonProjet
sonar.projectName=Mon Projet
sonar.projectVersion=1.0

sonar.sources=src
sonar.tests=tests
sonar.exclusions=**/bin/**,**/obj/**,**/wwwroot/lib/**,**/*.Designer.cs
sonar.cs.opencover.reportsPaths=**/coverage.opencover.xml
sonar.coverage.exclusions=**/*Tests.cs,**/Program.cs
```

### Étape 2 : Mise en place du pipeline CI/CD

Créez un pipeline CI/CD qui intègre les vérifications de qualité :

```yaml
# azure-pipelines.yml
trigger:
- main
- feature/*

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  solution: '**/*.sln'

stages:
- stage: Build
  jobs:
  - job: BuildAndTest
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        projects: '$(solution)'

    - task: SonarQubePrepare@4
      inputs:
        SonarQube: 'SonarQube'
        scannerMode: 'MSBuild'
        projectKey: 'MonEntreprise_MonProjet'
        projectName: 'Mon Projet'
        extraProperties: |
          sonar.cs.opencover.reportsPaths=$(Build.SourcesDirectory)/TestResults/Coverage/coverage.opencover.xml
          sonar.coverage.exclusions=**/*Tests.cs,**/Program.cs

    - task: DotNetCoreCLI@2
      displayName: 'Build'
      inputs:
        command: 'build'
        projects: '$(solution)'
        arguments: '--configuration $(buildConfiguration) --no-restore'

    - task: DotNetCoreCLI@2
      displayName: 'Run style check'
      inputs:
        command: 'custom'
        custom: 'format'
        arguments: '--verify-no-changes --verbosity diagnostic'

    - task: DotNetCoreCLI@2
      displayName: 'Run tests with coverage'
      inputs:
        command: 'test'
        projects: '**/*Tests/*.csproj'
        arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover'

    - task: SonarQubeAnalyze@4

    - task: SonarQubePublish@4
      inputs:
        pollingTimeoutSec: '300'

    - task: PowerShell@2
      displayName: 'Check SonarQube Quality Gate'
      inputs:
        targetType: 'inline'
        script: |
          $status = Invoke-RestMethod -Uri "$(SonarQubeURL)/api/qualitygates/project_status?projectKey=MonEntreprise_MonProjet" -Headers @{Authorization = "Bearer $(SonarQubeToken)"}
          if ($status.projectStatus.status -ne "OK") {
            Write-Host "##vso[task.logissue type=warning]Quality Gate failed: $($status.projectStatus.status)"
          }

    - task: DotNetCoreCLI@2
      displayName: 'Publish'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --no-build --output $(Build.ArtifactStagingDirectory)'
        zipAfterPublish: true

    - task: PublishBuildArtifacts@1
      inputs:
        pathtoPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'
```

### Étape 3 : Configuration des revues de code

Configurez des règles de protection de branche dans GitHub/Azure DevOps :

```json
// GitHub Branch Protection Rules (via API)
{
  "required_status_checks": {
    "strict": true,
    "contexts": [
      "build",
      "tests",
      "SonarQube Code Analysis"
    ]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "dismissal_restrictions": {},
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true,
    "required_approving_review_count": 1
  },
  "restrictions": null
}
```

### Étape 4 : Formation et mise en place culturelle

1. **Documentation** :
   - Créez un guide de style de code
   - Documentez les règles adoptées et pourquoi
   - Fournissez des exemples de bonnes pratiques

2. **Formation** :
   - Organisez des sessions sur les outils de qualité
   - Expliquez les métriques et leur signification
   - Instaurez des revues de code constructives

3. **Culture** :
   - Valorisez la qualité du code
   - Célébrez les améliorations
   - Donnez du temps pour le refactoring

## Comparaison des outils

Pour vous aider à choisir les bons outils, voici une comparaison :

| Aspect | SonarQube | NDepend | ReSharper/Rider |
|--------|-----------|---------|-----------------|
| **Type** | Plateforme web | Outil .NET | Extension VS/IDE |
| **Prix** | Gratuit (Community) à payant | Commercial | Commercial |
| **Focus** | Qualité globale | Architecture et métriques | Productivité et analyse |
| **Installation** | Serveur dédié/Docker | Installation locale | Installation locale |
| **Langages** | Multi-langage | .NET uniquement | Principalement .NET |
| **Intégration CI/CD** | Excellente | Bonne | Moyenne |
| **Visualisations** | Bonnes | Excellentes | Bonnes |
| **Refactoring** | Non | Non | Excellent |
| **Analyses** | En profondeur | Très détaillées | En temps réel |
| **Performance** | Moyenne | Rapide | Peut ralentir VS |
| **Courbe d'apprentissage** | Moyenne | Élevée | Faible |

### Quand utiliser quel outil ?

- **SonarQube** : Pour une analyse complète de la qualité du code au niveau projet/organisation
- **NDepend** : Pour des analyses approfondies d'architecture et de dépendances
- **ReSharper/Rider** : Pour le développement quotidien et les refactorings

### Combinaison idéale

Pour une stratégie complète, considérez cette combinaison :

1. **ReSharper/Rider** pour les développeurs au quotidien
2. **SonarQube** dans le pipeline CI/CD pour l'analyse continue
3. **NDepend** pour des analyses périodiques approfondies de l'architecture

## Conclusion

Les outils de qualité de code sont essentiels pour maintenir et améliorer la qualité des applications .NET. En combinant différents outils et en intégrant des vérifications automatisées dans vos processus de développement, vous pouvez :

- Détecter les problèmes tôt dans le cycle de développement
- Maintenir des standards de qualité cohérents
- Réduire la dette technique
- Améliorer la maintenabilité du code
- Faciliter l'intégration de nouveaux développeurs

L'investissement dans la qualité du code est payant à long terme : moins de bugs, développement plus rapide des nouvelles fonctionnalités et coûts de maintenance réduits.

## Ressources additionnelles

### Documentation officielle

- [SonarQube Documentation](https://docs.sonarqube.org/latest/)
- [NDepend Documentation](https://www.ndepend.com/docs/getting-started-with-ndepend)
- [ReSharper Documentation](https://www.jetbrains.com/help/resharper/Introduction.html)
- [Rider Documentation](https://www.jetbrains.com/help/rider/Introduction.html)

### Livres recommandés

- "Clean Code: A Handbook of Agile Software Craftsmanship" par Robert C. Martin
- "Refactoring: Improving the Design of Existing Code" par Martin Fowler
- "The Art of Unit Testing" par Roy Osherove
- "Working Effectively with Legacy Code" par Michael Feathers

### Formations en ligne

- Pluralsight - Cours sur la qualité de code .NET
- LinkedIn Learning - Tutoriels sur SonarQube, ReSharper
- Microsoft Learn - Modules sur les bonnes pratiques .NET

### Communauté

- [.NET Community Standup](https://dotnet.microsoft.com/en-us/live)
- [C# Corner](https://www.c-sharpcorner.com/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/c%23)
- [Reddit r/csharp](https://www.reddit.com/r/csharp/)

N'oubliez pas que la qualité du code est un voyage, pas une destination. Commencez petit, progressez régulièrement et cultivez une culture d'excellence technique dans votre équipe.

⏭️ 16. [Sécurité en C#](/16-securite-en-csharp/README.md)
