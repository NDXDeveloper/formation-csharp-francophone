# 13.4. Applications de bureau empaquetées (MSIX)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 13.4.1. Création de packages MSIX

### Qu'est-ce que MSIX ?

MSIX est un format de package moderne pour les applications Windows, créé par Microsoft pour remplacer les technologies d'installation traditionnelles comme MSI, AppX et ClickOnce. Il combine les meilleures fonctionnalités de ces anciennes technologies tout en apportant de nouvelles capacités pour les applications modernes.

Le format MSIX offre plusieurs avantages :
- Installation fiable avec un taux de réussite proche de 100%
- Désinstallation propre et complète
- Installation sans interaction utilisateur (silencieuse)
- Isolation des applications
- Mise à jour simplifiée
- Optimisation de la bande passante réseau

### Anatomie d'un package MSIX

Un package MSIX est essentiellement une archive ZIP qui contient :

1. **Fichiers d'application** - Le code et les ressources de votre application
2. **AppxManifest.xml** - Un fichier de manifeste qui décrit l'application, ses capacités et ses besoins
3. **AppxBlockMap.xml** - Une carte des blocs de données pour permettre les téléchargements et mises à jour incrémentiels
4. **Signature digitale** - Utilisée pour vérifier l'intégrité et l'authenticité du package

### Méthodes de création de packages MSIX

Il existe plusieurs façons de créer un package MSIX :

#### 1. À partir du code source avec Visual Studio

Si vous disposez du code source de votre application, la méthode la plus simple est d'utiliser Visual Studio :

1. Ouvrez votre solution dans Visual Studio 2019 ou ultérieur
2. Ajoutez un projet "Windows Application Packaging Project" (WAPP) à votre solution
3. Faites un clic droit sur "Dependencies" dans le projet WAPP, puis "Add Project Reference..." et sélectionnez votre projet d'application
4. Définissez votre application comme point d'entrée
5. Configurez les propriétés du package (nom, version, etc.)
6. Générez le projet pour créer votre package MSIX

```xml
<!-- Extrait simplifié d'un fichier AppxManifest.xml -->
<Package
  xmlns="http://schemas.microsoft.com/appx/manifest/foundation/windows10"
  xmlns:uap="http://schemas.microsoft.com/appx/manifest/uap/windows10"
  xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities">

  <Identity Name="MonApplication"
            Version="1.0.0.0"
            Publisher="CN=MonEntreprise" />

  <Properties>
    <DisplayName>Mon Application</DisplayName>
    <PublisherDisplayName>Mon Entreprise</PublisherDisplayName>
    <Logo>Assets\StoreLogo.png</Logo>
  </Properties>

  <Dependencies>
    <TargetDeviceFamily Name="Windows.Desktop"
                       MinVersion="10.0.17763.0"
                       MaxVersionTested="10.0.19041.0" />
  </Dependencies>

  <Applications>
    <Application Id="App"
                 Executable="MonApp.exe"
                 EntryPoint="Windows.FullTrustApplication">
      <uap:VisualElements DisplayName="Mon Application"
                          Square150x150Logo="Assets\Square150x150Logo.png"
                          Square44x44Logo="Assets\Square44x44Logo.png"
                          Description="Description de mon application"
                          BackgroundColor="transparent">
      </uap:VisualElements>
    </Application>
  </Applications>

  <Capabilities>
    <rescap:Capability Name="runFullTrust" />
  </Capabilities>
</Package>
```

#### 2. À partir d'installateurs existants avec MSIX Packaging Tool

Si vous ne disposez pas du code source ou souhaitez convertir une application existante :

1. Installez l'outil "MSIX Packaging Tool" depuis le Microsoft Store
2. Lancez l'outil et sélectionnez "Application package"
3. Choisissez entre une machine virtuelle ou votre ordinateur local pour la capture
4. Lancez le processus d'installation de votre application
5. Effectuez les personnalisations nécessaires (premiers lancements, configurations)
6. Terminez le processus pour générer le package MSIX

Cette méthode est idéale pour les applications d'entreprise dont vous n'avez pas le code source.

#### 3. Ligne de commande et automatisation

Pour les besoins d'automatisation, vous pouvez utiliser les outils en ligne de commande :

```powershell
# Utilisation de la ligne de commande MSIX Packaging Tool
MsixPackagingTool.exe create-package --template template.xml

# Ou utilisation de MakeAppx.exe (inclus dans le SDK Windows)
MakeAppx.exe pack /d C:\SourcApp /p MonApplication.msix

# Signature du package
SignTool.exe sign /fd SHA256 /a /f MyCertificate.pfx /p MotDePasse MonApplication.msix
```

### Configuration du manifeste MSIX

Le fichier AppxManifest.xml est crucial pour définir comment votre application fonctionne. Voici les sections principales :

1. **Identity** - Identifie de manière unique votre application (nom, version, éditeur)
2. **Properties** - Propriétés affichées dans le Microsoft Store ou lors de l'installation
3. **Dependencies** - Définit les exigences système de votre application
4. **Capabilities** - Autorisations requises par votre application
5. **Applications** - Définit le point d'entrée et les détails visuels

### Signature du package

Tous les packages MSIX doivent être signés numériquement. Vous avez plusieurs options :

1. **Certificat auto-signé** - Pour le test et développement
2. **Certificat d'entreprise** - Pour le déploiement dans votre organisation
3. **Certificat du Microsoft Store** - Pour la distribution sur le Store

```powershell
# Création d'un certificat auto-signé
New-SelfSignedCertificate -Type Custom -Subject "CN=MonEntreprise" -KeyUsage DigitalSignature -FriendlyName "MonCertificat" -CertStoreLocation "Cert:\CurrentUser\My" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3", "2.5.29.19={text}")

# Exportation du certificat en fichier PFX
$password = ConvertTo-SecureString -String "MotDePasse" -Force -AsPlainText
Export-PfxCertificate -cert "Cert:\CurrentUser\My\<empreinte du certificat>" -FilePath MonCertificat.pfx -Password $password
```

## 13.4.2. Déploiement et mise à jour

### Méthodes de déploiement

Les packages MSIX peuvent être déployés de plusieurs façons :

#### 1. Installation manuelle

L'utilisateur peut double-cliquer sur un fichier .msix pour l'installer. Windows vérifiera la signature et installera l'application.

#### 2. Fichier App Installer (.appinstaller)

Le fichier .appinstaller permet d'installer et de mettre à jour automatiquement les applications. Il contient des informations sur l'emplacement du package et sa mise à jour.

```xml
<!-- Exemple simplifié de fichier .appinstaller -->
<AppInstaller Uri="https://monentreprise.com/apps/MonApp.appinstaller"
              Version="1.0.0.0">
  <MainPackage Name="MonApplication"
               Version="1.0.0.0"
               Publisher="CN=MonEntreprise"
               Uri="https://monentreprise.com/apps/MonApp.msix" />
  <UpdateSettings>
    <OnLaunch HoursBetweenUpdateChecks="24" />
  </UpdateSettings>
</AppInstaller>
```

L'utilisateur installe initialement l'application via le fichier .appinstaller, et ensuite, l'application vérifiera périodiquement les mises à jour selon les paramètres définis.

#### 3. Microsoft Store

Si vous distribuez votre application via le Microsoft Store, Microsoft gère toute l'infrastructure de déploiement et de mise à jour pour vous.

#### 4. Déploiement d'entreprise

Pour les environnements d'entreprise, vous disposez de plusieurs options :

- **Microsoft Intune** - Déployez les packages MSIX via Intune en tant qu'applications métier
- **System Center Configuration Manager (SCCM)** - Utilisez le processus d'ajout d'application standard
- **PowerShell** - Utilisez les cmdlets Add-AppxPackage pour les scripts de déploiement
- **DISM (Deployment Image Servicing and Management)** - Pour intégrer des applications dans les images Windows

```powershell
# Installation via PowerShell
Add-AppxPackage -Path C:\Packages\MonApplication.msix

# Installation pour tous les utilisateurs (nécessite des privilèges administratifs)
Add-AppxProvisionedPackage -Online -PackagePath C:\Packages\MonApplication.msix -SkipLicense
```

### Stratégies de mise à jour

L'un des grands avantages de MSIX est la facilité de mise à jour. Voici quelques stratégies :

#### 1. Mise à jour différentielle

MSIX prend en charge les mises à jour différentielles, ce qui signifie que seuls les blocs de données modifiés sont téléchargés lors d'une mise à jour.

```xml
<!-- Dans le fichier .appinstaller -->
<UpdateSettings>
  <OnLaunch HoursBetweenUpdateChecks="24" />
  <AutomaticBackgroundTask />
  <ForceUpdateFromAnyVersion>true</ForceUpdateFromAnyVersion>
</UpdateSettings>
```

#### 2. Mise à jour côte à côte

Contrairement aux installateurs traditionnels, MSIX permet d'installer la nouvelle version pendant que l'ancienne est en cours d'exécution. L'ancienne version est remplacée au redémarrage de l'application.

#### 3. Retour à une version antérieure

En cas de problème avec une mise à jour, MSIX permet de revenir facilement à une version précédente.

```powershell
# Revenir à une version spécifique
Add-AppxPackage -Path C:\Packages\MonApplication_1.0.0.0.msix -ForceApplicationShutdown
```

### Gestion du cycle de vie de l'application

MSIX simplifie également la gestion du cycle de vie des applications :

1. **Installation** - Processus fiable avec vérification d'intégrité
2. **Mises à jour** - Automatiques ou manuelles, avec support différentiel
3. **Désinstallation** - Propre et complète, sans résidus

```powershell
# Désinstallation d'un package MSIX
Remove-AppxPackage -Package MonApplication_1.0.0.0_x64__1234567890abc
```

## 13.4.3. Sécurité et isolation

### Modèle de conteneur d'application

L'une des caractéristiques les plus importantes de MSIX est le modèle de conteneur. Chaque application MSIX s'exécute dans un conteneur léger qui :

1. **Isole les fichiers** - L'application ne peut pas accéder directement aux fichiers système critiques
2. **Virtualise le registre** - Les modifications du registre sont redirigées vers un emplacement spécifique à l'application
3. **Sépare les processus** - Les processus de l'application sont isolés des autres applications

Cette isolation présente plusieurs avantages :

- Protection du système d'exploitation contre les applications malveillantes
- Prévention des conflits entre applications
- Désinstallation propre sans laisser de résidus

### Virtualisation des fichiers et du registre

Lorsqu'une application MSIX tente d'écrire dans des zones protégées du système :

```
C:\Program Files\Windows\...
C:\Windows\...
HKEY_LOCAL_MACHINE\SOFTWARE\...
```

Ces écritures sont automatiquement redirigées vers :

```
C:\Users\Utilisateur\AppData\Local\Packages\MonApplication_1234567890abc\
```

Cela permet à l'application de fonctionner comme prévu, tout en protégeant le système.

### Gestion des permissions

MSIX utilise un système de déclaration de capacités pour gérer les autorisations :

```xml
<Capabilities>
  <!-- Accès complet (applications de bureau) -->
  <rescap:Capability Name="runFullTrust" />

  <!-- Autres capacités spécifiques -->
  <uap:Capability Name="documentsLibrary" />
  <DeviceCapability Name="microphone" />
  <DeviceCapability Name="webcam" />
</Capabilities>
```

Ces déclarations permettent :
- À l'utilisateur de connaître les autorisations demandées par l'application
- Au système d'exploitation d'appliquer les restrictions appropriées
- Aux administrateurs de contrôler les capacités autorisées

### Package Support Framework (PSF)

Certaines applications anciennes peuvent nécessiter des accès qui ne sont pas compatibles avec le modèle de conteneur MSIX. Le Package Support Framework (PSF) est une solution open-source qui permet de corriger ces problèmes de compatibilité :

1. Intégrez les DLL du PSF dans votre package MSIX
2. Configurez le fichier config.json pour définir les fixups nécessaires
3. Le PSF intercepte les appels problématiques et les redirige correctement

```json
{
  "applications": [
    {
      "id": "MonApplication",
      "executable": "MonApp.exe",
      "workingDirectory": "",
      "fixups": [
        {
          "dll": "FileRedirectionFixup.dll",
          "config": {
            "redirectedPaths": {
              "packageRelative": [
                {
                  "base": "VFS\\ProgramFilesX64\\MonApplication\\Config",
                  "patterns": ["*.ini", "*.cfg"]
                }
              ]
            }
          }
        }
      ]
    }
  ]
}
```

## 13.4.4. Intégration avec l'App Store

### Publier sur le Microsoft Store

Le Microsoft Store est une plateforme de distribution idéale pour les applications MSIX publiques :

1. **Création d'un compte développeur** - Inscrivez-vous sur le Partner Center (frais uniques)
2. **Réservation du nom** - Réservez le nom de votre application
3. **Préparation du package** - Assurez-vous que votre package répond aux exigences du Store
4. **Soumission** - Téléchargez votre package et fournissez les métadonnées (descriptions, captures d'écran)
5. **Certification** - Microsoft teste votre application pour s'assurer qu'elle répond aux normes
6. **Publication** - Une fois approuvée, votre application est disponible pour les utilisateurs

### Avantages de la distribution via le Store

Le Microsoft Store offre plusieurs avantages :

1. **Visibilité** - Des millions d'utilisateurs potentiels
2. **Mise à jour automatique** - Gestion des mises à jour par Microsoft
3. **Analyse** - Statistiques d'utilisation et rapports de crash
4. **Monétisation** - Vente d'applications, achats intégrés ou publicités
5. **Confiance** - Les applications sont vérifiées par Microsoft

### Store d'entreprise privé

Pour les entreprises, Microsoft propose le "Store privé" :

1. **Microsoft Store for Business** - Portail pour les entreprises
2. **Applications privées** - Distribuez des applications uniquement à votre organisation
3. **Licences en volume** - Achetez des licences en masse pour vos employés
4. **Gestion centralisée** - Contrôlez quelles applications sont disponibles

```powershell
# Configuration du Store d'entreprise privé via PowerShell
Set-WinStoreWebAccountID -AccountId "user@entreprise.com"
Set-WinStoreWebAccountSID -SID "S-1-5-21-..."
```

### Création de packages pour le Store

Lors de la préparation d'un package pour le Store, tenez compte des éléments suivants :

1. **Signature** - Le Store re-signera votre package avec un certificat Microsoft
2. **Logos et icônes** - Fournissez tous les actifs graphiques requis dans le manifeste
3. **Capacités** - Limitez-les au minimum nécessaire (meilleure expérience utilisateur)
4. **Tests de validation** - Utilisez l'outil Windows App Certification Kit (WACK)

```xml
<!-- Exemple de déclaration d'actifs visuels -->
<uap:VisualElements
  DisplayName="Mon Application"
  Square150x150Logo="Assets\Square150x150Logo.png"
  Square44x44Logo="Assets\Square44x44Logo.png"
  Description="Description de mon application"
  BackgroundColor="transparent">
  <uap:DefaultTile
    Wide310x150Logo="Assets\Wide310x150Logo.png"
    Square71x71Logo="Assets\SmallTile.png"
    Square310x310Logo="Assets\LargeTile.png">
    <uap:ShowNameOnTiles>
      <uap:ShowOn Tile="square150x150Logo"/>
      <uap:ShowOn Tile="wide310x150Logo"/>
      <uap:ShowOn Tile="square310x310Logo"/>
    </uap:ShowNameOnTiles>
  </uap:DefaultTile>
  <uap:SplashScreen Image="Assets\SplashScreen.png" />
</uap:VisualElements>
```

### Mise à jour des applications du Store

Les mises à jour via le Store sont simples :

1. Créez une nouvelle version du package MSIX
2. Incrémentez le numéro de version dans le manifeste
3. Soumettez le nouveau package au Store
4. Une fois approuvé, il sera automatiquement proposé aux utilisateurs

Microsoft gère automatiquement la distribution des mises à jour différentielles, ce qui économise de la bande passante et accélère le processus.

---

En résumé, MSIX représente une avancée significative dans la façon de distribuer et gérer les applications Windows. Que vous soyez un développeur indépendant ou une grande entreprise, MSIX offre des avantages considérables en termes de fiabilité, de sécurité et de facilité de gestion par rapport aux technologies d'installation précédentes.

⏭️ 14. [Déploiement et distribution](/14-deploiement-et-distribution/README.md)
