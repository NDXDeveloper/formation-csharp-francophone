# 14.2. Déploiement d'applications Windows Forms

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le déploiement est l'étape cruciale qui consiste à préparer votre application Windows Forms pour la distribuer aux utilisateurs finaux. Cette partie du tutoriel vous guidera à travers les différentes méthodes de déploiement, leurs avantages et inconvénients, ainsi que les meilleures pratiques pour garantir que votre application fonctionne correctement sur les ordinateurs de vos utilisateurs.

## 14.2.1. Installation classique

L'installation classique consiste à créer un programme d'installation (setup) qui guidera l'utilisateur à travers le processus d'installation de votre application. C'est la méthode la plus courante et la plus conviviale pour distribuer vos applications Windows Forms.

### Avantages de l'installation classique

- Interface utilisateur guidée pour l'installation
- Vérification préalable des prérequis système
- Installation du framework .NET si nécessaire
- Création automatique de raccourcis dans le menu Démarrer et sur le bureau
- Possibilité d'inclure des composants supplémentaires (SQL Server Express, etc.)
- Ajout/suppression de programmes Windows pour la désinstallation
- Support pour les mises à jour automatiques

### Création d'un installateur avec Visual Studio

Visual Studio propose plusieurs options pour créer des installateurs :

#### Utilisation du projet "Setup and Deployment"

1. **Ajoutez un nouveau projet à votre solution**
   - Clic droit sur la solution → Ajouter → Nouveau projet
   - Recherchez "Setup" dans la barre de recherche
   - Sélectionnez "Setup Project" (ou "Projet d'installation")

2. **Configurez les fichiers à déployer**
   - Clic droit sur le projet d'installation → Ajouter → Sortie du projet
   - Sélectionnez votre projet Windows Forms
   - Choisissez la configuration (généralement "Release")

3. **Configurez les détails de l'application**
   - Clic droit sur le projet d'installation → Vue → Éditeur de fichiers
   - Modifiez les propriétés comme "Author", "Description", "ProductName", etc.

4. **Ajoutez des raccourcis**
   - Clic droit sur le dossier "Application Folder" → Ajouter → Raccourci
   - Sélectionnez l'exécutable principal
   - Renommez le raccourci
   - Faites glisser ce raccourci vers "User's Desktop" ou "User's Programs Menu"

5. **Vérifiez les prérequis**
   - Clic droit sur le projet d'installation → Propriétés
   - Accédez à l'onglet "Prerequisites"
   - Cochez ".NET Framework" avec la version appropriée

6. **Générez l'installateur**
   - Clic droit sur le projet d'installation → Build
   - Le fichier .msi sera créé dans le dossier de sortie

#### Utilisation de WiX Toolset

WiX (Windows Installer XML) est un outil plus puissant pour créer des installateurs Windows :

1. **Installez l'extension WiX pour Visual Studio**
   - Allez dans Extensions → Gérer les extensions
   - Recherchez et installez "WiX Toolset Visual Studio Extension"

2. **Ajoutez un projet WiX à votre solution**
   - Clic droit sur la solution → Ajouter → Nouveau projet
   - Sous "WiX Toolset" → "Setup Project for WiX v3"

3. **Créez un fichier Product.wxs**
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
     <Product Id="*" Name="MonApplication" Language="1033" Version="1.0.0.0"
              Manufacturer="MaSociété" UpgradeCode="PUT-GUID-HERE">
       <Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />

       <MajorUpgrade DowngradeErrorMessage="Une version plus récente est déjà installée." />
       <MediaTemplate EmbedCab="yes" />

       <Feature Id="ProductFeature" Title="MonApplication" Level="1">
         <ComponentGroupRef Id="ProductComponents" />
         <ComponentRef Id="ApplicationShortcut" />
       </Feature>

       <!-- Vérifie si .NET Framework est installé -->
       <PropertyRef Id="NETFRAMEWORK45"/>
       <Condition Message="Cette application requiert .NET Framework 4.5. Veuillez l'installer puis réessayer.">
         <![CDATA[Installed OR NETFRAMEWORK45]]>
       </Condition>
     </Product>

     <Fragment>
       <Directory Id="TARGETDIR" Name="SourceDir">
         <Directory Id="ProgramFilesFolder">
           <Directory Id="INSTALLFOLDER" Name="MonApplication" />
         </Directory>
         <Directory Id="ProgramMenuFolder">
           <Directory Id="ApplicationProgramsFolder" Name="MonApplication"/>
         </Directory>
         <Directory Id="DesktopFolder" Name="Desktop" />
       </Directory>
     </Fragment>

     <Fragment>
       <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
         <Component Id="ProductComponent" Guid="*">
           <File Id="MonApplicationEXE" Source="$(var.MonApplication.TargetPath)" KeyPath="yes" />
           <!-- Ajoutez d'autres fichiers ici -->
         </Component>
       </ComponentGroup>

       <DirectoryRef Id="ApplicationProgramsFolder">
         <Component Id="ApplicationShortcut" Guid="*">
           <Shortcut Id="ApplicationStartMenuShortcut"
                     Name="MonApplication"
                     Description="Description de mon application"
                     Target="[#MonApplicationEXE]"
                     WorkingDirectory="INSTALLFOLDER"/>
           <RemoveFolder Id="CleanUpShortcut" Directory="ApplicationProgramsFolder" On="uninstall"/>
           <RegistryValue Root="HKCU" Key="Software\MaSociété\MonApplication"
                          Name="installed" Type="integer" Value="1" KeyPath="yes"/>
         </Component>
       </DirectoryRef>
     </Fragment>
   </Wix>
   ```

4. **Configurez les références**
   - Clic droit sur le projet WiX → Ajouter une référence
   - Ajoutez une référence à votre projet Windows Forms

5. **Générez l'installateur**
   - Clic droit sur le projet WiX → Build
   - Le fichier .msi sera créé dans le dossier de sortie

### Installation silencieuse

Pour permettre l'installation sans intervention de l'utilisateur (utile pour les déploiements d'entreprise) :

```batch
# Installation silencieuse avec MSI
msiexec /i MonApplication.msi /qn

# Installation avec journalisation
msiexec /i MonApplication.msi /qn /l*v install.log

# Désinstallation silencieuse
msiexec /x MonApplication.msi /qn
```

## 14.2.2. Déploiement xcopy

Le déploiement xcopy est une méthode simple qui consiste à copier directement les fichiers de l'application sur l'ordinateur cible sans utiliser d'installateur. Le nom vient de la commande DOS `xcopy` qui permet de copier des fichiers et des répertoires.

### Avantages du déploiement xcopy

- Simplicité extrême - pas d'installateur à créer
- Portabilité - l'application peut être exécutée depuis une clé USB
- Pas de droits administrateur requis
- Idéal pour les petites applications ou les applications internes
- Facilité de mise à jour (remplacer les fichiers existants)

### Inconvénients du déploiement xcopy

- Pas d'entrée dans "Ajout/Suppression de programmes"
- Pas de vérification automatique des prérequis
- Pas d'installation des dépendances (.NET Framework)
- Pas de création automatique de raccourcis
- Pas de registre Windows pour stocker les paramètres

### Procédure de déploiement xcopy

1. **Créez un dossier de publication**
   - Dans Visual Studio, clic droit sur le projet → Publier
   - Choisissez "Dossier" comme cible de publication
   - Sélectionnez un dossier de destination

2. **Configurez le profil de publication**
   - Mode de déploiement : Self-contained
   - Cible de déploiement : win-x64 (ou autre selon la cible)
   - Cochez "Produire un fichier unique" si vous voulez un seul exécutable

3. **Publiez l'application**
   - Cliquez sur "Publier"
   - Visual Studio générera les fichiers dans le dossier spécifié

4. **Distribuez l'application**
   - Copiez le dossier de publication sur l'ordinateur cible
   - L'utilisateur peut exécuter directement l'application .exe

### Avec la ligne de commande

Pour automatiser le déploiement xcopy :

```batch
# Publier en mode self-contained pour un seul exécutable
dotnet publish -c Release -r win-x64 --self-contained true /p:PublishSingleFile=true /p:IncludeNativeLibrariesForSelfExtract=true

# Copier vers un partage réseau
xcopy bin\Release\net8.0\win-x64\publish\*.* \\serveur\applications\MonApplication\ /E /Y

# Créer un fichier ZIP pour la distribution
powershell Compress-Archive -Path bin\Release\net8.0\win-x64\publish\*.* -DestinationPath MonApplication.zip
```

### Créer un raccourci manuellement

Bien que le déploiement xcopy ne crée pas automatiquement de raccourcis, vous pouvez fournir un script batch pour aider l'utilisateur :

```batch
@echo off
echo Création de raccourcis pour MonApplication...
powershell "$s=(New-Object -COM WScript.Shell).CreateShortcut('%userprofile%\Desktop\MonApplication.lnk');$s.TargetPath='%~dp0MonApplication.exe';$s.Save()"
powershell "$s=(New-Object -COM WScript.Shell).CreateShortcut('%appdata%\Microsoft\Windows\Start Menu\Programs\MonApplication.lnk');$s.TargetPath='%~dp0MonApplication.exe';$s.Save()"
echo Raccourcis créés avec succès !
pause
```

## 14.2.3. Considérations sur les dépendances

La gestion des dépendances est un aspect crucial du déploiement d'applications Windows Forms. Votre application peut dépendre de différentes bibliothèques, composants ou frameworks qui doivent être présents sur l'ordinateur cible.

### Types de dépendances

1. **Dépendances du framework .NET**
   - Le runtime .NET Framework ou .NET Core/.NET 5+
   - Bibliothèques de classes de base (BCL)

2. **Assemblies tierces (packages NuGet)**
   - Bibliothèques externes utilisées par votre application
   - Packages NuGet référencés dans votre projet

3. **Dépendances natives**
   - Fichiers DLL natifs (C/C++)
   - Composants COM ou ActiveX

4. **Autres dépendances**
   - Bases de données (SQL Server Express, SQLite)
   - Composants du système (Direct3D, WPF, etc.)
   - Pilotes matériels

### Identifier les dépendances

Pour identifier les dépendances de votre application :

1. **Utilisez l'Analyseur de dépendances de Visual Studio**
   - Clic droit sur un projet → Analyser → Analyser les dépendances d'assemblys

2. **Examinez les références de projet**
   - Ouvrez le nœud "Dépendances" dans l'Explorateur de solutions
   - Vérifiez les packages NuGet et les références d'assemblage

3. **Utilisez des outils tiers**
   - Dependency Walker (pour les DLL natives)
   - ILSpy (pour voir les références d'assemblies .NET)

### Gestion des dépendances .NET

#### Versions du Framework .NET

Si votre application utilise .NET Framework :

```xml
<!-- Dans le fichier .csproj -->
<PropertyGroup>
  <TargetFramework>net472</TargetFramework>
</PropertyGroup>
```

Lors du déploiement, vous devez vous assurer que la version .NET Framework requise est installée. Options possibles :

1. **Vérification préalable dans l'installateur**
   - L'installateur peut vérifier et télécharger .NET Framework si nécessaire

2. **Fournir un lien de téléchargement**
   - Incluez un lien vers le package Web Installer du framework

3. **Intégrer la version redistribuable**
   - Incluez le package redistribuable dans votre installation

#### Dépendances de packages NuGet

Les packages NuGet sont généralement inclus lors de la compilation :

1. **Assemblies privées**
   - Par défaut, les assemblies NuGet sont copiées dans le dossier de sortie

2. **Réduire le nombre de dépendances**
   - Utilisez l'option "PublishTrimmed" pour éliminer le code non utilisé :
   ```xml
   <PropertyGroup>
     <PublishTrimmed>true</PublishTrimmed>
   </PropertyGroup>
   ```

### Gestion des dépendances natives

Si votre application utilise des DLL natives :

1. **Placez-les dans le dossier de l'application**
   - Ajoutez les fichiers à votre projet avec "Copy to Output Directory" = "Copy always"

2. **Utilisez des dossiers spécifiques à l'architecture**
   - Créez des sous-dossiers "x86" et "x64" pour les versions 32 et 64 bits

3. **Utilisation de P/Invoke**
   - Si vous utilisez P/Invoke pour appeler du code natif, assurez-vous que les DLL sont correctement déployées

```csharp
// Exemple de P/Invoke avec un chemin personnalisé
[DllImport("native\\speciallib.dll", CallingConvention = CallingConvention.Cdecl)]
private static extern int MaFonctionNative(int a, int b);
```

### Considérations spéciales

#### Bases de données locales

Pour les applications utilisant une base de données locale :

1. **SQL Server LocalDB**
   - Incluez le runtime SQL Server LocalDB dans votre installation
   - Créez la base de données au premier démarrage

2. **SQLite**
   - Incluez la DLL SQLite dans votre déploiement
   - Le fichier de base de données peut être créé à la première utilisation

```csharp
// Exemple de création de base de données SQLite
private void InitializeDatabase()
{
    string dbPath = Path.Combine(
        Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
        "MonApplication", "database.db");

    if (!File.Exists(dbPath))
    {
        Directory.CreateDirectory(Path.GetDirectoryName(dbPath));
        SQLiteConnection.CreateFile(dbPath);

        using (var connection = new SQLiteConnection($"Data Source={dbPath};Version=3;"))
        {
            connection.Open();
            var command = connection.CreateCommand();
            command.CommandText = "CREATE TABLE Users (ID INTEGER PRIMARY KEY, Name TEXT, Email TEXT)";
            command.ExecuteNonQuery();
        }
    }
}
```

#### Composants COM et ActiveX

Pour les applications utilisant des composants COM ou ActiveX :

1. **Enregistrement des composants**
   - Utilisez regsvr32.exe pour enregistrer les composants lors de l'installation
   - Les droits d'administrateur sont nécessaires

2. **Isolation d'applications**
   - Vous pouvez utiliser l'isolation d'application pour éviter l'enregistrement COM

```batch
# Dans un script d'installation pour enregistrer un composant COM
regsvr32 /s MonComposant.dll
```

## 14.2.4. Self-contained vs Framework-dependent

Le choix entre un déploiement "self-contained" (autonome) et "framework-dependent" (dépendant du framework) est une décision importante qui affecte la taille, la portabilité et la facilité de maintenance de votre application.

### Déploiement dépendant du framework (Framework-dependent)

Dans un déploiement dépendant du framework, votre application nécessite que le runtime .NET soit préinstallé sur l'ordinateur cible.

#### Avantages du déploiement dépendant du framework

- **Taille réduite** : L'application est plus petite car elle ne contient pas le runtime
- **Mise à jour du framework** : Les applications bénéficient automatiquement des mises à jour de sécurité du framework
- **Économie d'espace disque** : Plusieurs applications peuvent partager le même runtime
- **Compatibilité garantie** : La version du framework est contrôlée

#### Inconvénients du déploiement dépendant du framework

- **Dépendance externe** : Le framework doit être préinstallé
- **Versions multiples** : Risque de conflits entre différentes versions du framework
- **Droits d'administration** : L'installation du framework nécessite des droits d'administrateur

#### Configuration pour un déploiement dépendant du framework

Avec .NET 5+ :

```xml
<!-- Dans le fichier .csproj -->
<PropertyGroup>
  <TargetFramework>net8.0-windows</TargetFramework>
  <UseWindowsForms>true</UseWindowsForms>
  <SelfContained>false</SelfContained>
</PropertyGroup>
```

Avec la ligne de commande :

```batch
dotnet publish -c Release -r win-x64 --self-contained false
```

### Déploiement autonome (Self-contained)

Dans un déploiement autonome, votre application inclut tout le runtime .NET nécessaire à son exécution.

#### Avantages du déploiement autonome

- **Indépendance du framework** : Aucune version de .NET n'est requise sur l'ordinateur cible
- **Contrôle complet des versions** : Vous choisissez exactement la version du runtime
- **Facilité de déploiement** : Aucun prérequis à installer
- **Possibilité d'exécuter plusieurs versions** : Différentes applications peuvent utiliser différentes versions du runtime

#### Inconvénients du déploiement autonome

- **Taille importante** : L'application inclut tout le runtime .NET (généralement 50-150 Mo)
- **Pas de mise à jour automatique du framework** : Les correctifs de sécurité ne sont pas appliqués automatiquement
- **Duplication** : Chaque application a sa propre copie du runtime

#### Configuration pour un déploiement autonome

Avec .NET 5+ :

```xml
<!-- Dans le fichier .csproj -->
<PropertyGroup>
  <TargetFramework>net8.0-windows</TargetFramework>
  <UseWindowsForms>true</UseWindowsForms>
  <SelfContained>true</SelfContained>
  <RuntimeIdentifier>win-x64</RuntimeIdentifier>
  <PublishSingleFile>true</PublishSingleFile>
  <IncludeNativeLibrariesForSelfExtract>true</IncludeNativeLibrariesForSelfExtract>
</PropertyGroup>
```

Avec la ligne de commande :

```batch
dotnet publish -c Release -r win-x64 --self-contained true /p:PublishSingleFile=true /p:IncludeNativeLibrariesForSelfExtract=true
```

### Comparaison des tailles de déploiement

Voici une comparaison approximative de la taille de déploiement pour une application Windows Forms simple :

| Type de déploiement | Taille approximative |
|----------------------|----------------------|
| Framework-dependent  | 1-5 Mo               |
| Self-contained       | 50-150 Mo            |
| Self-contained trimmed | 30-80 Mo           |
| Single file self-contained | 30-150 Mo (un seul fichier) |

### Optimisation de la taille pour les déploiements autonomes

Pour réduire la taille d'un déploiement autonome :

1. **Utilisez le trimming**
   ```xml
   <PropertyGroup>
     <PublishTrimmed>true</PublishTrimmed>
     <TrimMode>link</TrimMode>
   </PropertyGroup>
   ```

2. **Activez la compression**
   ```xml
   <PropertyGroup>
     <EnableCompressionInSingleFile>true</EnableCompressionInSingleFile>
   </PropertyGroup>
   ```

3. **Publiez pour une architecture spécifique**
   - Utilisez une RID spécifique comme `win-x64` au lieu de `win`

4. **Utilisez l'AOT (Ahead-of-Time) compilation**
   ```xml
   <PropertyGroup>
     <PublishAot>true</PublishAot>
   </PropertyGroup>
   ```

### Ciblage de plusieurs plateformes

Pour cibler plusieurs plateformes Windows :

```xml
<PropertyGroup>
  <RuntimeIdentifiers>win-x86;win-x64;win-arm64</RuntimeIdentifiers>
</PropertyGroup>
```

Pour compiler pour chaque plateforme :

```batch
dotnet publish -c Release -r win-x86 --self-contained true
dotnet publish -c Release -r win-x64 --self-contained true
dotnet publish -c Release -r win-arm64 --self-contained true
```

### Recommandations

Pour choisir entre un déploiement dépendant du framework ou autonome :

1. **Utilisez un déploiement dépendant du framework si :**
   - Vous déployez plusieurs applications .NET
   - La taille du déploiement est critique
   - Vous pouvez garantir l'installation du framework

2. **Utilisez un déploiement autonome si :**
   - Vous souhaitez une installation "zéro prérequis"
   - Vous avez besoin d'une version spécifique du runtime
   - Vous déployez dans des environnements non contrôlés

3. **Utilisez un déploiement autonome en fichier unique si :**
   - Vous voulez une expérience d'installation simplifiée
   - La portabilité est importante (ex: applications sur clé USB)
   - Vous préférez un déploiement sans fichiers externes

## Résumé

Le déploiement d'applications Windows Forms offre plusieurs options adaptées à différents scénarios. L'installation classique est idéale pour les applications destinées au grand public, tandis que le déploiement xcopy convient aux applications simples ou portables. La gestion correcte des dépendances et le choix entre un déploiement dépendant du framework ou autonome sont essentiels pour garantir que votre application fonctionne correctement sur tous les systèmes cibles.

## Exercices pratiques

1. Créez un projet WinForms simple et publiez-le en mode "self-contained" pour Windows 64 bits
2. Créez un installateur MSI pour cette application
3. Identifiez toutes les dépendances de votre application
4. Comparez la taille de l'application en mode "framework-dependent" et "self-contained"
5. Écrivez un script qui automatise le déploiement de votre application sur un dossier partagé

⏭️
