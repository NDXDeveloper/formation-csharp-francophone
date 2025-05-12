# 14.6. Packaging MSIX

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

MSIX est un format de package d'application Windows moderne développé par Microsoft pour remplacer les technologies plus anciennes comme MSI, AppX et ClickOnce. Ce format offre une installation et une désinstallation propres, une distribution simplifiée et une meilleure sécurité pour vos applications Windows.

Ce chapitre vous guidera à travers le processus de création, distribution et mise à jour de packages MSIX pour vos applications .NET, avec une attention particulière à la sécurité et à la signature des packages.

## 14.6.1. Création de packages

### Qu'est-ce que MSIX ?

MSIX est un format de package universel Windows qui combine les meilleurs éléments des technologies MSI, AppX, ClickOnce et App-V. Ses principaux avantages sont :

- **Installation propre** : Les applications sont conteneurisées, ce qui évite les conflits avec d'autres logiciels
- **Désinstallation complète** : Aucun résidu n'est laissé lors de la désinstallation
- **Réduction de l'impact disque** : Partage des fichiers communs entre applications
- **Mise à jour en direct** : Les applications peuvent être mises à jour même en cours d'exécution
- **Sécurité renforcée** : Les applications s'exécutent dans un environnement isolé

### Prérequis pour la création de packages MSIX

Pour créer des packages MSIX, vous aurez besoin de :

- **Windows 10 version 1709 (build 16299) ou supérieure**
- **Visual Studio 2019 ou supérieure** avec la charge de travail "Développement de bureau en C#" et le composant "Outils de la plateforme Windows universelle"
- **Kit SDK Windows 10 (10.0.17763.0 ou supérieur)**
- **Un certificat de signature de code** (obligatoire pour l'installation)

### Création d'un package MSIX avec Visual Studio

Voici comment créer un package MSIX pour votre application existante :

#### 1. Ajoutez un projet d'empaquetage Windows

1. **Ouvrez votre solution** dans Visual Studio
2. **Cliquez avec le bouton droit sur la solution** dans l'Explorateur de solutions
3. Sélectionnez **Ajouter > Nouveau projet**
4. Recherchez "**Windows Application Packaging Project**" (Projet d'empaquetage d'application Windows)
5. Donnez-lui un nom (généralement `[NomDeLApp].Package`)
6. Cliquez sur **Créer**

#### 2. Configurez les références d'application

1. Dans le nouveau projet d'empaquetage, **cliquez avec le bouton droit** sur "Applications"
2. Sélectionnez **Ajouter une référence**
3. Cochez la case à côté de votre projet d'application principale
4. Cliquez sur **OK**

#### 3. Configurez le manifeste du package

1. Double-cliquez sur le fichier **Package.appxmanifest** dans le projet d'empaquetage
2. L'éditeur de manifeste s'ouvre avec plusieurs onglets :

   a. **Application** :
      - Définissez le nom d'affichage, la description et les couleurs
      - Configurez l'application de démarrage (généralement votre application principale)

   b. **Présentation visuelle** :
      - Ajoutez des logos et des écrans de démarrage
      - Vous aurez besoin d'images en différentes tailles (50x50, 150x150, etc.)

   c. **Fonctionnalités** :
      - Sélectionnez les fonctionnalités requises par votre application
      - Pour les applications de bureau classiques, vous aurez généralement besoin de "runFullTrust"

   d. **Packaging** :
      - Définissez le nom du package, la version et l'éditeur
      - Générez un nouveau GUID pour "Package family name" si nécessaire

#### 4. Configurer les ressources visuelles

Les packages MSIX nécessitent des ressources visuelles comme les icônes et les logos :

1. Cliquez avec le bouton droit sur le projet d'empaquetage
2. Sélectionnez **Ajouter > Nouvel élément**
3. Choisissez **Ressources visuelles** (Visual Assets)
4. Suivez l'assistant pour générer automatiquement les ressources à partir d'une image source

Ou ajoutez manuellement chaque image dans le dossier Assets du projet.

#### 5. Créez des certificats de test pour le développement

Pour tester votre package MSIX en développement :

1. Cliquez avec le bouton droit sur le projet d'empaquetage
2. Sélectionnez **Propriétés**
3. Accédez à l'onglet **Signature de package**
4. Choisissez "**Créer un certificat de test...**"
5. Remplissez les informations du certificat
6. Cliquez sur **OK**

Notez que ce certificat est uniquement pour les tests. Pour la distribution, vous aurez besoin d'un certificat officiel (voir section 14.6.5).

#### 6. Générez le package MSIX

Pour créer le package final :

1. Définissez le projet d'empaquetage comme **projet de démarrage** (clic droit > Définir comme projet de démarrage)
2. Changez la configuration de build en **Release**
3. Cliquez avec le bouton droit sur le projet d'empaquetage
4. Sélectionnez **Publier > Créer des packages d'application**
5. Choisissez **Sideloading** ou **Microsoft Store** selon votre canal de distribution
6. Configurez la signature (utilisez le certificat créé précédemment pour les tests)
7. Sélectionnez les architectures cibles (x86, x64, ARM64)
8. Cliquez sur **Créer**

Visual Studio générera un dossier contenant :
- Le fichier `.msix` ou `.msixbundle` (si vous avez choisi plusieurs architectures)
- Un script PowerShell `Add-AppDevPackage.ps1` pour l'installation manuelle
- Un certificat `.cer` pour l'installation du certificat

### Utilisation de l'outil MSIX Packaging Tool

Microsoft propose également un outil dédié pour créer des packages MSIX à partir d'applications existantes :

1. **Téléchargez et installez le [MSIX Packaging Tool](https://www.microsoft.com/en-us/p/msix-packaging-tool/9n5lw3jbcxkf)** depuis le Microsoft Store

2. **Lancez l'outil** et choisissez l'option appropriée :
   - "Application Installer" pour convertir un installateur existant (MSI, EXE)
   - "Create package on this computer" pour empaqueter une application déjà installée
   - "Create package on remote machine" pour empaqueter sur un ordinateur distant

3. **Suivez l'assistant** qui vous guidera à travers :
   - La préparation de l'ordinateur
   - L'installation de l'application
   - La capture des modifications système
   - La configuration du manifeste
   - La génération du package MSIX

Cet outil est particulièrement utile pour convertir des applications existantes pour lesquelles vous ne disposez pas du code source.

### Création de package MSIX via ligne de commande

Vous pouvez également créer des packages MSIX à l'aide d'outils en ligne de commande, ce qui est utile pour l'automatisation et l'intégration continue :

```batch
# Création d'un package MSIX avec MakeAppx.exe (inclus dans le SDK Windows)
MakeAppx.exe pack /d "C:\Chemin\Vers\SourcesApp" /p "C:\Sortie\MonApplication.msix"

# Signature du package avec SignTool.exe
SignTool.exe sign /fd SHA256 /a /f "MyCertificate.pfx" /p "MotDePasse" "C:\Sortie\MonApplication.msix"
```

Pour l'intégration à un pipeline CI/CD, vous pouvez utiliser MSBuild :

```xml
<Project>
  <PropertyGroup>
    <AppxPackageDir>$(OutDir)AppPackages\</AppxPackageDir>
    <AppxBundle>Always</AppxBundle>
    <AppxBundlePlatforms>x86|x64|arm</AppxBundlePlatforms>
    <UapAppxPackageBuildMode>StoreUpload</UapAppxPackageBuildMode>
    <AppxPackageSigningEnabled>true</AppxPackageSigningEnabled>
    <PackageCertificateThumbprint>YOUR_CERTIFICATE_THUMBPRINT</PackageCertificateThumbprint>
  </PropertyGroup>
</Project>
```

### Personnalisation avancée du package

Vous pouvez personnaliser davantage votre package MSIX en :

#### 1. Définissant des protocoles personnalisés

Permet à votre application de répondre à des URL personnalisées comme `monapp://action` :

```xml
<!-- Dans le fichier Package.appxmanifest -->
<Extensions>
  <uap:Extension Category="windows.protocol">
    <uap:Protocol Name="monapp">
      <uap:DisplayName>Mon Application</uap:DisplayName>
    </uap:Protocol>
  </uap:Extension>
</Extensions>
```

#### 2. Configurant les extensions de fichiers

Associe des types de fichiers à votre application :

```xml
<Extensions>
  <uap:Extension Category="windows.fileTypeAssociation">
    <uap:FileTypeAssociation Name="montype">
      <uap:DisplayName>Fichier Mon Application</uap:DisplayName>
      <uap:SupportedFileTypes>
        <uap:FileType>.mapp</uap:FileType>
      </uap:SupportedFileTypes>
    </uap:FileTypeAssociation>
  </uap:Extension>
</Extensions>
```

#### 3. Ajoutant des tâches en arrière-plan

Pour les applications qui doivent exécuter des tâches même lorsqu'elles ne sont pas au premier plan :

```xml
<Extensions>
  <Extension Category="windows.backgroundTasks" EntryPoint="MonApplication.BackgroundTask">
    <BackgroundTasks>
      <Task Type="timer"/>
    </BackgroundTasks>
  </Extension>
</Extensions>
```

## 14.6.2. Distribution via Microsoft Store

Le Microsoft Store est le canal de distribution officiel pour les applications Windows, offrant visibilité, gestion des mises à jour et sécurité.

### Avantages de la distribution via le Store

- **Visibilité** : Votre application apparaît dans le catalogue du Store
- **Mises à jour automatiques** : Les nouvelles versions sont automatiquement installées
- **Gestion des paiements** : Le Store gère les achats intégrés et les abonnements
- **Analytics** : Obtenez des données sur les installations et l'utilisation
- **Validation automatique** : Le processus de certification garantit la qualité et la sécurité

### Création d'un compte développeur

Pour distribuer sur le Microsoft Store, vous devez d'abord créer un compte développeur :

1. Visitez le [Centre des partenaires Microsoft](https://partner.microsoft.com/dashboard)
2. Créez un compte (individuel ou entreprise)
3. Payez les frais d'inscription (à partir de $19 pour les comptes individuels, $99 pour les entreprises)
4. Complétez le processus de vérification

### Préparation de votre application pour le Store

Avant de soumettre votre application :

1. **Vérifiez les exigences techniques** :
   - Consultez les [Directives de certification des applications](https://docs.microsoft.com/windows/uwp/publish/certification-requirements)
   - Testez avec l'[Outil de validation d'applications Windows](https://docs.microsoft.com/windows/uwp/debug-test-perf/windows-app-certification-kit)

2. **Préparez les ressources marketing** :
   - Captures d'écran (au moins une, jusqu'à 10)
   - Icônes et logos en différentes tailles
   - Description, fonctionnalités et notes de version
   - Vidéos promotionnelles (facultatif)

3. **Configurez les options de tarification** :
   - Gratuit ou payant
   - Achats intégrés ou abonnements
   - Période d'essai

### Soumission de votre package MSIX au Store

Pour soumettre votre application :

1. **Connectez-vous au [Centre des partenaires](https://partner.microsoft.com/dashboard)**

2. **Créez une nouvelle soumission** :
   - Cliquez sur "Créer une application"
   - Réservez un nom d'application unique

3. **Configurez les propriétés de l'application** :
   - Catégorie et sous-catégorie
   - Âge et classification de contenu
   - Mots-clés pour la recherche

4. **Téléchargez les packages** :
   - Téléchargez votre fichier `.msixbundle` ou `.msix`
   - Attendez l'analyse automatique
   - Corrigez les erreurs éventuelles

5. **Configurez les informations de tarification et de disponibilité** :
   - Prix de base
   - Marchés cibles
   - Date de disponibilité

6. **Soumettez pour certification** :
   - Cliquez sur "Soumettre pour certification"
   - Le processus prend généralement 1 à 3 jours ouvrables

### Suivi de votre soumission

Après la soumission :

1. **Vérifiez le statut** dans le Centre des partenaires
2. **Répondez aux retours** si des problèmes sont identifiés
3. Une fois approuvée, votre application sera **publiée automatiquement** dans le Store

### Promotion de votre application Store

Pour augmenter la visibilité de votre application :

1. **Partagez le lien du Store** sur vos canaux de communication
2. **Utilisez des campagnes promotionnelles** via le Centre des partenaires
3. **Collectez des avis positifs** en encourageant les utilisateurs satisfaits
4. **Mettez régulièrement à jour** votre application pour maintenir sa visibilité

## 14.6.3. Distribution hors Store

Bien que le Microsoft Store offre de nombreux avantages, la distribution hors Store (ou "sideloading") peut être nécessaire dans certains cas.

### Pourquoi distribuer hors Store ?

- **Applications d'entreprise** : Applications internes non destinées au public
- **Applications spécialisées** : Applications pour un public restreint
- **Contournement des restrictions du Store** : Certaines fonctionnalités peuvent être limitées dans le Store
- **Contrôle total du processus de distribution** : Sans intermédiaire

### Prérequis pour la distribution hors Store

Pour installer des applications MSIX hors Store, les conditions suivantes doivent être remplies :

1. **Windows 10 version 1709 ou supérieure** sur les ordinateurs cibles
2. **Mode développeur activé** ou "Sideloading" configuré via les stratégies de groupe
3. **Certificat de signature installé** sur les ordinateurs cibles

### Méthodes de distribution hors Store

#### 1. Installation manuelle

La méthode la plus simple pour les petits déploiements :

1. **Distribuez le package MSIX** et le certificat (`.cer`)
2. **Les utilisateurs installent d'abord le certificat** :
   - Double-cliquez sur le fichier `.cer`
   - Sélectionnez "Installer le certificat"
   - Choisissez "Machine locale" (peut nécessiter des droits d'administrateur)
   - Sélectionnez "Placer tous les certificats dans le magasin suivant"
   - Cliquez sur "Parcourir" et sélectionnez "Autorités de certification racines de confiance"
   - Terminez l'assistant

3. **Puis installent l'application** :
   - Double-cliquez sur le fichier `.msix` ou `.msixbundle`
   - Cliquez sur "Installer"

#### 2. Script PowerShell

Pour simplifier l'installation, Visual Studio génère automatiquement un script PowerShell avec le package :

1. **Faites un clic droit sur `Add-AppDevPackage.ps1`**
2. Sélectionnez **Exécuter avec PowerShell**
3. Le script :
   - Installe automatiquement le certificat
   - Installe l'application
   - Gère les erreurs courantes

#### 3. Serveur web interne

Pour les déploiements d'entreprise plus importants :

1. **Hébergez les fichiers** sur un serveur web interne
2. **Créez une page web** avec des liens d'installation
3. Les utilisateurs accèdent à l'URL et **installent directement** :
   ```html
   <a href="ms-appinstaller:?source=https://monserveur.interne/apps/MonApplication.msixbundle">
     Installer Mon Application
   </a>
   ```

#### 4. Utilisation des fichiers App Installer

Les fichiers `.appinstaller` permettent d'automatiser les mises à jour :

1. **Créez un fichier XML `.appinstaller`** :
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <AppInstaller Uri="https://monserveur.interne/apps/MonApplication.appinstaller"
                 Version="1.0.0.0">
     <MainPackage Name="MonApplication"
                  Version="1.0.0.0"
                  Publisher="CN=MonEntreprise"
                  Uri="https://monserveur.interne/apps/MonApplication.msixbundle"/>
     <UpdateSettings>
       <OnLaunch HoursBetweenUpdateChecks="12"/>
     </UpdateSettings>
   </AppInstaller>
   ```

2. **Hébergez le fichier** avec votre package MSIX
3. **Partagez le lien** vers le fichier `.appinstaller`

#### 5. Déploiement via des outils de gestion de systèmes

Pour les entreprises avec une infrastructure IT :

1. **Microsoft Intune** :
   - Téléchargez le MSIX dans Intune
   - Assignez à des groupes d'utilisateurs ou d'appareils
   - Déployez automatiquement via les stratégies

2. **System Center Configuration Manager (SCCM)** :
   - Créez une application dans SCCM
   - Utilisez le package MSIX comme source d'installation
   - Déployez sur des collections d'ordinateurs

### Activation du mode Sideloading via la stratégie de groupe

Pour les environnements d'entreprise, configurez le sideloading de manière centralisée :

1. **Ouvrez l'éditeur de stratégie de groupe**
2. Naviguez vers **Configuration ordinateur > Modèles d'administration > Composants Windows > Déploiement du package d'application**
3. Activez **Autoriser le déploiement de tous les packages d'applications de confiance**
4. **Appliquez la stratégie** aux ordinateurs cibles

## 14.6.4. Mise à jour des packages

L'un des avantages majeurs du format MSIX est sa gestion élégante des mises à jour.

### Types de mises à jour MSIX

MSIX prend en charge trois types de mises à jour :

1. **Mise à jour complète** : Remplacement complet de l'ancienne version
2. **Mise à jour différentielle** : Mise à jour plus petite qui contient uniquement les changements
3. **Mise à jour en direct** : Installation sans fermer l'application en cours d'exécution

### Mise à jour des applications Store

Pour les applications du Microsoft Store, les mises à jour sont gérées automatiquement :

1. **Créez une nouvelle soumission** dans le Centre des partenaires
2. **Incrémentez le numéro de version** dans le manifeste
3. **Téléchargez le nouveau package**
4. **Soumettez pour certification**

Une fois approuvée, la mise à jour sera automatiquement distribuée aux utilisateurs.

### Mise à jour des applications hors Store

Pour les applications distribuées en dehors du Store :

#### 1. Mise à jour manuelle

La méthode la plus simple mais la moins pratique :

1. **Créez un nouveau package** avec un numéro de version supérieur
2. **Distribuez-le** aux utilisateurs
3. **Les utilisateurs installent** la nouvelle version par-dessus l'ancienne

#### 2. Mise à jour automatique avec des fichiers App Installer

Offre une expérience similaire au Store :

1. **Mettez à jour le fichier `.appinstaller`** avec les détails de la nouvelle version :
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <AppInstaller Uri="https://monserveur.interne/apps/MonApplication.appinstaller"
                 Version="1.1.0.0">
     <MainPackage Name="MonApplication"
                  Version="1.1.0.0"
                  Publisher="CN=MonEntreprise"
                  Uri="https://monserveur.interne/apps/MonApplication_1.1.0.0.msixbundle"/>
     <UpdateSettings>
       <OnLaunch HoursBetweenUpdateChecks="12"/>
       <AutomaticBackgroundTask/>
     </UpdateSettings>
   </AppInstaller>
   ```

2. **Placez le nouveau package** à l'emplacement spécifié
3. **Les applications installées via ce fichier** vérifieront automatiquement les mises à jour

#### 3. Configuration des options de mise à jour

Vous pouvez configurer le comportement de mise à jour dans le fichier `.appinstaller` :

```xml
<UpdateSettings>
  <!-- Vérifier les mises à jour au lancement -->
  <OnLaunch HoursBetweenUpdateChecks="24"/>

  <!-- Vérifier en arrière-plan -->
  <AutomaticBackgroundTask/>

  <!-- Mise à jour obligatoire -->
  <ForceUpdateFromAnyVersion>true</ForceUpdateFromAnyVersion>

  <!-- URL de mise à jour alternative -->
  <UpdateBlocksActivation>true</UpdateBlocksActivation>
</UpdateSettings>
```

### Création de packages de mise à jour différentiels

Pour économiser de la bande passante, créez des packages différentiels :

1. **Utilisez l'outil MakeAppx** :
   ```batch
   MakeAppx.exe diff /sv 1.0.0.0 /ov 1.1.0.0 /ip "C:\Packages\v1.0.0.0" /op "C:\Packages\v1.1.0.0" /dp "C:\Packages\Diff\v1.0.0.0_to_v1.1.0.0.msix"
   ```

2. **Signez le package différentiel** :
   ```batch
   SignTool.exe sign /fd SHA256 /a /f MyCertificate.pfx /p MotDePasse "C:\Packages\Diff\v1.0.0.0_to_v1.1.0.0.msix"
   ```

3. **Référencez-le dans le fichier `.appinstaller`** :
   ```xml
   <AppInstaller Uri="...">
     <MainPackage ... />
     <Dependencies>
       <Package Name="MonApplication"
                Publisher="CN=MonEntreprise"
                Version="1.0.0.0"
                ProcessorArchitecture="x64"
                Uri="https://monserveur.interne/apps/v1.0.0.0_to_v1.1.0.0.msix"/>
     </Dependencies>
   </AppInstaller>
   ```

### Test des mises à jour

Avant de déployer une mise à jour :

1. **Testez sur un environnement similaire à la production**
2. **Vérifiez la préservation des données utilisateur**
3. **Testez le processus de restauration** en cas de problème
4. **Vérifiez les performances** après la mise à jour

## 14.6.5. Certificats et signature

La signature numérique est un élément essentiel de la sécurité des packages MSIX.

### Importance de la signature

Tous les packages MSIX doivent être signés numériquement pour :

- **Garantir l'intégrité** : Assure que le package n'a pas été modifié
- **Vérifier l'identité de l'éditeur** : Prouve qui a créé le package
- **Établir la confiance** : Les utilisateurs voient qui a créé l'application
- **Autoriser l'installation** : Windows refuse d'installer des packages non signés

### Types de certificats

Plusieurs types de certificats peuvent être utilisés pour signer des packages MSIX :

#### 1. Certificats de test (développement uniquement)

- **Créés localement** via Visual Studio
- **Ne sont pas reconnus** par d'autres ordinateurs sans installation manuelle
- **Ne doivent jamais être utilisés** en production
- **Gratuits** mais limités

#### 2. Certificats d'autorité de certification (CA)

- **Achetés auprès d'entreprises de confiance** comme DigiCert, GlobalSign, Comodo
- **Reconnus automatiquement** par la plupart des ordinateurs
- **Validés par des tiers** qui vérifient l'identité de l'acheteur
- **Coûtent entre 100€ et 500€** par an selon le niveau de validation

#### 3. Certificats du Microsoft Store

- **Générés automatiquement** lors de la publication dans le Store
- **Utilisés uniquement** pour les applications du Store
- **Inclus dans les frais** du compte développeur

### Obtention d'un certificat de signature de code

Pour obtenir un certificat pour la distribution :

1. **Sélectionnez une autorité de certification** reconnue
2. **Achetez un certificat de signature de code**
3. **Fournissez les preuves d'identité** demandées
4. **Recevez et installez le certificat**

### Génération d'un certificat auto-signé (pour les tests)

Pour le développement et les tests internes :

#### Via PowerShell

```powershell
# Générer un certificat auto-signé
New-SelfSignedCertificate -Type Custom -Subject "CN=Mon Entreprise, O=Mon Organisation, C=FR" -KeyUsage DigitalSignature -FriendlyName "Mon Certificat MSIX" -CertStoreLocation "Cert:\CurrentUser\My" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3", "2.5.29.19={text}")

# Exporter le certificat vers un fichier PFX
$password = ConvertTo-SecureString -String "MotDePasse" -Force -AsPlainText
Export-PfxCertificate -cert "Cert:\CurrentUser\My\[THUMBPRINT]" -FilePath MyCertificate.pfx -Password $password

# Exporter le certificat public (pour l'installation)
Export-Certificate -cert "Cert:\CurrentUser\My\[THUMBPRINT]" -FilePath MyCertificate.cer
```

#### Via Visual Studio

1. **Ouvrez les propriétés du projet d'empaquetage**
2. **Accédez à l'onglet "Signature de package"**
3. **Cliquez sur "Créer un certificat de test"**
4. **Remplissez les informations demandées**

### Signature d'un package MSIX

Une fois que vous avez un certificat, vous pouvez signer votre package :

#### Via Visual Studio

1. **Ouvrez les propriétés du projet d'empaquetage**
2. **Accédez à l'onglet "Signature de package"**
3. **Sélectionnez "Sélectionner à partir du fichier"** et choisissez votre fichier PFX
4. **Entrez le mot de passe du certificat**
5. **Configurez l'horodatage** (recommandé)
6. **Créez le package** comme d'habitude

#### Via ligne de commande

```batch
# Signer un package MSIX
SignTool.exe sign /fd SHA256 /a /f "MyCertificate.pfx" /p "MotDePasse" "C:\Packages\MonApplication.msix"

# Avec horodatage (recommandé)
SignTool.exe sign /fd SHA256 /a /f "MyCertificate.pfx" /p "MotDePasse" /tr "http://timestamp.digicert.com" /td SHA256 "C:\Packages\MonApplication.msix"
```

### Horodatage des signatures

L'horodatage est une étape cruciale car il permet à la signature de rester valide même après l'expiration du certificat :

1. **Utilisez un service d'horodatage reconnu** :
   - http://timestamp.digicert.com
   - http://timestamp.globalsign.com/tsa/r6advanced1
   - http://tsa.starfieldtech.com

2. **Spécifiez l'algorithme d'horodatage** (généralement SHA256)

3. **Vérifiez le succès de l'horodatage** dans les propriétés du fichier signé

### Gestion des certificats pour les déploiements d'entreprise

Pour les déploiements à grande échelle :

#### 1. Utilisation de certificats d'entreprise

1. **Créez une autorité de certification d'entreprise** avec Active Directory
2. **Émettez des certificats de signature de code** depuis cette autorité
3. **Distribuez le certificat racine** via la stratégie de groupe
4. **Signez vos packages** avec les certificats émis

#### 2. Configuration du magasin de certificats approuvés

Distribuez les certificats via la stratégie de groupe :

1. **Ouvrez l'éditeur de stratégie de groupe**
2. Naviguez vers **Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies de clé publique > Autorités de certification racines de confiance**
3. **Importez votre certificat racine**

#### 3. Renouvellement des certificats

Les certificats ont une durée de validité limitée, généralement entre 1 et 3 ans. Il est crucial de planifier le renouvellement avant leur expiration :

1. **Suivez les dates d'expiration** :
   - Créez un calendrier de renouvellement
   - Configurez des alertes 2-3 mois avant l'expiration
   - Documentez la procédure de renouvellement

2. **Processus de renouvellement** :
   - Contactez l'autorité de certification pour renouveler
   - Générez une nouvelle requête de certificat (CSR)
   - Recevez et installez le nouveau certificat
   - Mettez à jour tous les scripts de signature

3. **Transition entre certificats** :
   - Utilisez l'horodatage pour les packages signés avec l'ancien certificat
   - Commencez à utiliser le nouveau certificat dès réception
   - Ne republiez pas forcément toutes les anciennes versions

4. **Double signature (recommandée)** :
   - Lorsque vous approchez de la date d'expiration, vous pouvez appliquer une double signature
   - Signez d'abord avec l'ancien certificat (encore valide)
   - Puis signez avec le nouveau certificat

```batch
# Double signature d'un package MSIX
SignTool.exe sign /fd SHA256 /f "AncienCertificat.pfx" /p "MotDePasse1" "MonApplication.msix"
SignTool.exe sign /fd SHA256 /f "NouveauCertificat.pfx" /p "MotDePasse2" /as "MonApplication.msix"
```

### Révocation de certificats

Si votre certificat est compromis ou si vous devez révoquer une application :

1. **Contactez votre autorité de certification** pour révoquer le certificat

2. **Pour les applications Store** :
   - Retirez l'application du Store via le Centre des partenaires
   - Ou publiez rapidement une version mise à jour avec un nouveau certificat

3. **Pour les applications hors Store** :
   - Distribuez une liste de révocation de certificats (CRL) via la stratégie de groupe
   - Publiez une nouvelle version signée avec un nouveau certificat
   - Communiquez clairement avec vos utilisateurs

4. **En cas d'incident de sécurité** :
   - Documentez la chronologie et l'impact
   - Suivez votre procédure de réponse aux incidents
   - Informez les parties concernées

### Meilleures pratiques de sécurité pour les certificats

1. **Stockage sécurisé des certificats privés** :
   - Utilisez un HSM (Hardware Security Module) si possible
   - Sinon, stockez les fichiers PFX dans un emplacement sécurisé
   - Limitez l'accès à ceux qui en ont besoin

2. **Protection par mot de passe** :
   - Utilisez des mots de passe forts pour les fichiers PFX
   - Changez régulièrement les mots de passe
   - Utilisez un gestionnaire de mots de passe pour les stocker

3. **Séparation des environnements** :
   - Utilisez des certificats différents pour le développement, le test et la production
   - N'utilisez jamais de certificats de production dans les environnements de développement

4. **Audit et journalisation** :
   - Enregistrez toutes les utilisations des certificats
   - Vérifiez régulièrement les journaux pour détecter les activités suspectes
   - Établissez un processus de vérification régulier

### Vérification des signatures

Pour vérifier qu'un package MSIX est correctement signé :

1. **Inspection visuelle** :
   - Les propriétés du fichier MSIX montrent le signataire dans l'onglet "Signatures numériques"
   - Vérifiez que le certificat est valide et approuvé

2. **Vérification via PowerShell** :
   ```powershell
   Get-AuthenticodeSignature -FilePath "C:\Packages\MonApplication.msix"
   ```

3. **Vérification avec SignTool** :
   ```batch
   SignTool.exe verify /pa "C:\Packages\MonApplication.msix"
   ```

4. **Test d'installation** :
   - Essayez d'installer le package sur un système propre
   - Vérifiez qu'aucun avertissement de sécurité n'apparaît

## Scénarios avancés et astuces

### Gestion des capacités étendues

Les applications MSIX s'exécutent dans un conteneur isolé, ce qui peut limiter certaines fonctionnalités. Voici comment contourner ces limitations :

#### Exécution en mode "runFullTrust"

Pour les applications de bureau classiques :

```xml
<!-- Dans Package.appxmanifest -->
<Extensions>
  <desktop:Extension Category="windows.fullTrustProcess" Executable="MonApplication.exe" />
  <rescap:Capability Name="runFullTrust" />
</Extensions>
```

N'oubliez pas d'ajouter les espaces de noms appropriés :

```xml
<Package
  xmlns="http://schemas.microsoft.com/appx/manifest/foundation/windows10"
  xmlns:desktop="http://schemas.microsoft.com/appx/manifest/desktop/windows10"
  xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities">
```

#### Accès aux fichiers système

Pour accéder à des emplacements en dehors du conteneur :

```xml
<Capabilities>
  <rescap:Capability Name="broadFileSystemAccess" />
</Capabilities>
```

#### Communication entre applications

Pour permettre la communication entre vos applications MSIX :

```xml
<Extensions>
  <com:Extension Category="windows.comServer">
    <com:ComServer>
      <com:ExeServer Executable="MonApplication.exe" Arguments="-ComServer">
        <com:Class Id="40DED47F-B5E2-42EF-B78C-47A8ADFFD8C3" DisplayName="Mon Serveur COM" />
      </com:ExeServer>
    </com:ComServer>
  </com:Extension>
</Extensions>
```

### Création de packages MSIX modificateurs

Les packages modificateurs permettent d'ajouter des configurations ou des personnalisations à une application de base :

1. **Créez un package principal** avec l'application de base

2. **Créez un package modificateur** avec les personnalisations :
   ```xml
   <Package>
     <Dependencies>
       <uap4:MainPackageDependency Name="MonApplicationBase" />
     </Dependencies>
     <!-- Fichiers et configurations supplémentaires -->
   </Package>
   ```

3. **Distribuez les packages séparément** ou ensemble

### Déploiement pour différentes architectures

Pour prendre en charge plusieurs architectures (x86, x64, ARM64) :

1. **Créez des packages spécifiques à chaque architecture** :
   - Configurez les plateformes cibles dans Visual Studio
   - Ou utilisez des commandes MSBuild distinctes

2. **Créez un bundle MSIX** :
   ```batch
   MakeAppx.exe bundle /d C:\Packages\Input /p C:\Packages\Output\MonApplication.msixbundle
   ```

3. **Signez le bundle** comme un package normal :
   ```batch
   SignTool.exe sign /fd SHA256 /a /f MyCertificate.pfx /p MotDePasse "C:\Packages\Output\MonApplication.msixbundle"
   ```

### Débogage d'applications MSIX

Si vous rencontrez des problèmes avec vos packages MSIX :

1. **Utilisez l'Event Viewer** :
   - Consultez les journaux sous "Applications and Services Logs > Microsoft > Windows > AppxPackagingOM"
   - Filtrez par ID d'événement pour trouver les erreurs spécifiques

2. **Activez la journalisation détaillée** :
   ```batch
   MakeAppx.exe pack /d C:\SourceApp /p C:\Output\App.msix /l
   ```

3. **Utilisez l'outil MSIX Packaging Tool** pour diagnostiquer les problèmes de conversion

4. **Pour les problèmes d'installation** :
   - Exécutez `Add-AppxPackage -Register AppxManifest.xml -Debug` pour voir les messages détaillés
   - Vérifiez les journaux d'installation dans `%LOCALAPPDATA%\Packages`

### Scripts de déploiement complets

Pour automatiser le déploiement MSIX dans un environnement d'entreprise :

```powershell
# Script de déploiement MSIX complet
param(
    [string]$PackagePath,
    [string]$CertPath,
    [string]$CertPassword,
    [switch]$ForceInstallCert,
    [switch]$RemoveExistingVersion
)

# Vérifier si le package existe
if (-not (Test-Path $PackagePath)) {
    Write-Error "Le package $PackagePath n'existe pas."
    exit 1
}

# Installer le certificat si nécessaire
if ($ForceInstallCert -or -not (Test-Path "Cert:\LocalMachine\Root\$CertThumbprint")) {
    Write-Host "Installation du certificat..."
    $securePassword = ConvertTo-SecureString -String $CertPassword -Force -AsPlainText
    Import-PfxCertificate -FilePath $CertPath -CertStoreLocation Cert:\LocalMachine\Root -Password $securePassword
    Write-Host "Certificat installé."
}

# Vérifier si une version est déjà installée
$packageName = (Get-AppxPackageManifest -Path $PackagePath).Package.Identity.Name
$existingPackage = Get-AppxPackage -Name $packageName

if ($existingPackage -and $RemoveExistingVersion) {
    Write-Host "Suppression de la version existante ($($existingPackage.Version))..."
    Remove-AppxPackage -Package $existingPackage.PackageFullName
}

# Installer le package
Write-Host "Installation du package MSIX..."
try {
    Add-AppxPackage -Path $PackagePath -ForceApplicationShutdown
    Write-Host "Package installé avec succès."
} catch {
    Write-Error "Erreur lors de l'installation du package: $_"
    exit 1
}
```

## Résumé

Le packaging MSIX est une technologie moderne qui offre de nombreux avantages pour le déploiement d'applications Windows :

- **Installation et désinstallation propres** : Pas de résidus ou de conflits
- **Mise à jour en direct** : Les applications peuvent être mises à jour sans interruption
- **Sécurité renforcée** : L'exécution conteneurisée et la signature obligatoire
- **Distribution flexible** : Via le Store ou en entreprise (sideloading)
- **Compatibilité étendue** : Fonctionne avec les applications traditionnelles et modernes

Lorsque vous travaillez avec MSIX, n'oubliez pas ces points clés :

1. **Planifiez votre stratégie de certification** dès le début
2. **Testez vos packages** dans différents environnements
3. **Automatisez le processus** de création et de signature
4. **Documentez les procédures** de déploiement et de mise à jour
5. **Utilisez des fichiers `.appinstaller`** pour les mises à jour automatiques

En suivant ces bonnes pratiques, vous pourrez profiter pleinement des avantages du format MSIX pour vos applications Windows.

## Exercices pratiques

1. Créez un projet Windows Forms ou WPF simple et empaquetez-le en MSIX.

2. Générez un certificat auto-signé et utilisez-le pour signer votre package MSIX.

3. Créez un fichier `.appinstaller` pour votre application et testez le processus de mise à jour.

4. Déployez votre application MSIX sur au moins trois ordinateurs différents et documentez les problèmes éventuels.

5. Créez un package MSIX qui prend en charge plusieurs architectures (x86 et x64).

⏭️ 14.7. [Configurations de déploiement](/14-deploiement-et-distribution/14-7-configurations-de-deploiement.md)
