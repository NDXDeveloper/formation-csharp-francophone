# 14.3. Déploiement d'applications WPF

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Windows Presentation Foundation (WPF) est un framework d'interface utilisateur qui permet de créer des applications Windows riches et modernes. Le déploiement d'applications WPF nécessite une attention particulière en raison de leur architecture spécifique et de leur utilisation intensive de ressources graphiques. Ce chapitre vous guidera à travers les différentes méthodes et considérations pour déployer efficacement vos applications WPF.

## 14.3.1. Packaging et installation

### Méthodes de packaging

Il existe plusieurs façons d'empaqueter votre application WPF pour la distribution aux utilisateurs finaux. Chaque méthode a ses avantages et ses inconvénients.

#### 1. Projet d'installation Visual Studio (MSI)

Visual Studio permet de créer des installateurs MSI via les projets d'installation.

**Avantages :**
- Interface utilisateur d'installation familière
- Support intégré pour l'installation des prérequis
- Entrée dans "Programmes et fonctionnalités" pour la désinstallation
- Gestion des raccourcis et des associations de fichiers

**Étapes de création :**

1. **Ajoutez un projet d'installation à votre solution**
   - Clic droit sur la solution → Ajouter → Nouveau projet
   - Choisissez "Setup Project" dans la catégorie "Other Project Types" → "Setup and Deployment"

2. **Configurez le contenu de l'installation**
   - Clic droit sur le projet d'installation → Ajouter → Sortie du projet
   - Sélectionnez votre projet WPF et la configuration (généralement Release)

3. **Ajoutez des raccourcis**
   - Dans l'éditeur de fichiers, clic droit sur "Application Folder" → Créer un raccourci
   - Déplacez le raccourci vers "User's Desktop" ou "User's Programs Menu"

4. **Définissez les prérequis**
   - Clic droit sur le projet → Propriétés → Prérequis
   - Cochez ".NET Framework" avec la version appropriée

5. **Personnalisez l'interface d'installation**
   - Double-cliquez sur "User Interface" dans la vue du projet
   - Ajoutez ou supprimez des dialogues selon vos besoins

6. **Générez l'installateur**
   - Compilez le projet pour créer le fichier MSI

#### 2. WiX Toolset

WiX (Windows Installer XML) est un outil puissant pour créer des installateurs Windows professionnels.

**Avantages :**
- Plus flexible que les projets d'installation Visual Studio
- Contrôle complet via XML
- Support avancé pour toutes les fonctionnalités de Windows Installer
- Open source et gratuit

**Étapes de création :**

1. **Installez l'extension WiX Toolset pour Visual Studio**

2. **Ajoutez un projet WiX à votre solution**
   - Clic droit sur la solution → Ajouter → Nouveau projet
   - Sélectionnez "WiX Toolset" → "WiX Project"

3. **Créez un fichier Product.wxs** (exemple simplifié) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
  <Product Id="*" Name="MonApplicationWPF" Language="1033" Version="1.0.0.0"
           Manufacturer="MaSociété" UpgradeCode="PUT-GUID-HERE">
    <Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />

    <MajorUpgrade DowngradeErrorMessage="Une version plus récente est déjà installée." />
    <MediaTemplate EmbedCab="yes" />

    <Feature Id="ProductFeature" Title="MonApplicationWPF" Level="1">
      <ComponentGroupRef Id="ProductComponents" />
      <ComponentRef Id="ApplicationShortcut" />
    </Feature>

    <!-- Vérifie si .NET Framework est installé -->
    <PropertyRef Id="NETFRAMEWORK48"/>
    <Condition Message="Cette application requiert .NET Framework 4.8. Veuillez l'installer puis réessayer.">
      <![CDATA[Installed OR NETFRAMEWORK48]]>
    </Condition>
  </Product>

  <Fragment>
    <Directory Id="TARGETDIR" Name="SourceDir">
      <Directory Id="ProgramFilesFolder">
        <Directory Id="INSTALLFOLDER" Name="MonApplicationWPF" />
      </Directory>
      <Directory Id="ProgramMenuFolder">
        <Directory Id="ApplicationProgramsFolder" Name="MonApplicationWPF"/>
      </Directory>
      <Directory Id="DesktopFolder" Name="Desktop" />
    </Directory>
  </Fragment>

  <Fragment>
    <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
      <Component Id="ProductComponent" Guid="*">
        <File Id="MonApplicationWPFEXE" Source="$(var.MonApplicationWPF.TargetPath)" KeyPath="yes" />
        <!-- Ajoutez d'autres fichiers ici -->
      </Component>
    </ComponentGroup>

    <DirectoryRef Id="ApplicationProgramsFolder">
      <Component Id="ApplicationShortcut" Guid="*">
        <Shortcut Id="ApplicationStartMenuShortcut"
                 Name="MonApplicationWPF"
                 Description="Description de mon application"
                 Target="[#MonApplicationWPFEXE]"
                 WorkingDirectory="INSTALLFOLDER"/>
        <RemoveFolder Id="CleanUpShortcut" Directory="ApplicationProgramsFolder" On="uninstall"/>
        <RegistryValue Root="HKCU" Key="Software\MaSociété\MonApplicationWPF"
                      Name="installed" Type="integer" Value="1" KeyPath="yes"/>
      </Component>
    </DirectoryRef>
  </Fragment>
</Wix>
```

4. **Ajoutez les références au projet WPF**
   - Clic droit sur le projet WiX → Ajouter une référence → Projets → Sélectionnez votre projet WPF

5. **Compilez le projet pour générer l'installateur MSI**

#### 3. MSIX (Windows App Package)

MSIX est un format de package moderne pour Windows 10/11 qui remplace les anciens formats comme MSI et AppX.

**Avantages :**
- Installation/désinstallation propre (pas de résidus)
- Mise à jour simplifiée avec préservation des données
- Installation sans interruption de l'application en cours d'exécution
- Distribution possible via le Microsoft Store

**Étapes de création :**

1. **Ajoutez un projet d'empaquetage Windows à votre solution**
   - Clic droit sur la solution → Ajouter → Nouveau projet
   - Sélectionnez "Windows Application Packaging Project"

2. **Configurez les applications à empaqueter**
   - Clic droit sur "Applications" dans le projet d'empaquetage → Ajouter référence
   - Sélectionnez votre projet WPF

3. **Configurez le manifeste du package**
   - Double-cliquez sur "Package.appxmanifest"
   - Configurez les informations du package (nom, description, icônes, etc.)

4. **Créez un certificat pour la signature**
   - Clic droit sur le projet d'empaquetage → Publier → Créer App Packages
   - Suivez l'assistant pour créer un certificat de test ou utiliser un certificat existant

5. **Générez le package MSIX**
   - Exécutez la commande "Créer App Packages" depuis le menu Publier
   - Ou compilez le projet en configuration Release

#### 4. ClickOnce

ClickOnce est une technologie de déploiement Microsoft qui permet de créer des applications auto-mises à jour qui peuvent être facilement installées depuis un serveur Web ou un partage réseau.

**Avantages :**
- Installation par utilisateur (pas besoin de droits administrateur)
- Mise à jour automatique
- Installation Web à partir d'une URL
- Fonctionnement en ligne et hors ligne

**Étapes de création :**

1. **Configurez les propriétés de publication ClickOnce**
   - Clic droit sur le projet WPF → Propriétés → Onglet "Publish"
   - Définissez l'emplacement de publication (URL, FTP, système de fichiers)
   - Configurez les options d'installation (bureau, menu démarrer)
   - Définissez la stratégie de mise à jour

2. **Configurez les prérequis**
   - Cliquez sur "Prerequisites" dans l'onglet Publish
   - Sélectionnez les composants requis (.NET Framework, etc.)

3. **Publiez l'application**
   - Cliquez sur "Publish Now" dans l'onglet Publish
   - Ou utilisez MSBuild avec la cible "Publish"

### Choix du packaging selon les besoins

| Méthode | Scénario idéal | Complexité |
|---------|----------------|------------|
| MSI (Visual Studio) | Applications d'entreprise simples | Moyenne |
| WiX Toolset | Applications complexes nécessitant un contrôle précis | Élevée |
| MSIX | Applications modernes pour Windows 10/11 | Moyenne |
| ClickOnce | Applications fréquemment mises à jour | Faible |

## 14.3.2. Ressources et actifs

Les applications WPF utilisent généralement de nombreuses ressources (images, polices, fichiers XAML, etc.) qui doivent être correctement incluses dans le déploiement.

### Types de ressources dans WPF

#### 1. Ressources intégrées (Embedded)

Les ressources intégrées sont compilées directement dans l'assembly de l'application.

**Comment les configurer :**
1. Ajoutez le fichier au projet
2. Ouvrez les propriétés du fichier
3. Définissez "Build Action" sur "Resource"

**Accès dans le code XAML :**
```xml
<Image Source="/MonApplication;component/Images/logo.png" />
```

**Accès dans le code C# :**
```csharp
var uri = new Uri("/MonApplication;component/Images/logo.png", UriKind.Relative);
var bitmap = new BitmapImage(uri);
```

**Avantages :**
- Packaging simple (un seul fichier)
- Pas de problèmes de chemins relatifs
- Protection contre la modification

**Inconvénients :**
- Augmentation de la taille de l'assembly
- Pas facilement modifiables après déploiement

#### 2. Ressources de contenu (Content)

Les ressources de contenu sont des fichiers séparés copiés dans le répertoire de sortie.

**Comment les configurer :**
1. Ajoutez le fichier au projet
2. Définissez "Build Action" sur "Content"
3. Définissez "Copy to Output Directory" sur "Copy if newer" ou "Copy always"

**Accès dans le code XAML :**
```xml
<Image Source="Images/logo.png" />
```

**Accès dans le code C# :**
```csharp
var bitmap = new BitmapImage(new Uri("Images/logo.png", UriKind.Relative));
```

**Avantages :**
- Facilement remplaçables après déploiement
- Réduction de la taille de l'assembly principal
- Possibilité de chargement à la demande

**Inconvénients :**
- Nécessite une gestion correcte des chemins
- Risque de fichiers manquants

#### 3. Ressources localisées (Localization)

Pour les applications multilingues, WPF offre un excellent support pour la localisation.

**Utilisation des fichiers .resx :**
1. Ajoutez un fichier de ressources (Resources.resx) au projet
2. Ajoutez des variantes pour différentes cultures (Resources.fr.resx, Resources.es.resx)
3. Définissez les chaînes localisées dans chaque fichier

**Accès aux ressources localisées :**
```csharp
var texteLocalise = Properties.Resources.MaChaineDeTexte;
```

**Localisation dynamique avec XAML :**
```xml
<TextBlock Text="{x:Static properties:Resources.MaChaineDeTexte}" />
```

### Gestion des polices personnalisées

Les polices sont un élément important du design des applications WPF et nécessitent une attention particulière lors du déploiement.

**Inclusion des polices dans l'application :**

1. **Ajoutez la police au projet**
   - Créez un dossier "Fonts"
   - Ajoutez le fichier de police (.ttf, .otf)
   - Définissez "Build Action" sur "Resource"

2. **Déclarez la police dans App.xaml**
```xml
<Application.Resources>
    <FontFamily x:Key="MaPolicePersonnalisee">/MonApplication;component/Fonts/MaPolice.ttf#Ma Police</FontFamily>
</Application.Resources>
```

3. **Utilisez la police dans votre application**
```xml
<TextBlock FontFamily="{StaticResource MaPolicePersonnalisee}" Text="Texte avec ma police" />
```

**Considérations sur les licences de polices :**
- Vérifiez toujours les droits d'utilisation des polices dans vos applications
- Certaines polices nécessitent l'achat d'une licence pour le déploiement commercial

### Optimisation des ressources pour le déploiement

1. **Compression d'images**
   - Utilisez des formats efficaces (PNG pour les graphiques, JPEG pour les photos)
   - Optimisez la taille des images avant de les inclure
   - Envisagez d'utiliser des outils d'optimisation d'images

2. **Ressources à la demande**
   - Chargez les ressources volumineuses uniquement lorsqu'elles sont nécessaires
   - Utilisez des techniques comme le lazyloading pour les images

```csharp
// Exemple de chargement à la demande
private void ChargementImageALaDemande()
{
    var bitmap = new BitmapImage();
    bitmap.BeginInit();
    bitmap.UriSource = new Uri("Images/grandeImage.jpg", UriKind.Relative);
    bitmap.CacheOption = BitmapCacheOption.OnLoad;
    bitmap.EndInit();
    monImage.Source = bitmap;
}
```

3. **Ressources distinctes par résolution**
   - Fournissez différentes tailles d'images pour différentes résolutions d'écran
   - Utilisez un système de nommage cohérent (image.png, image@2x.png)

```csharp
private BitmapImage ObtenirImageSelonResolution()
{
    double facteurEchelle = PresentationSource.FromVisual(this).CompositionTarget.TransformToDevice.M11;
    string nomFichier = facteurEchelle > 1.5 ? "image@2x.png" : "image.png";
    return new BitmapImage(new Uri($"Images/{nomFichier}", UriKind.Relative));
}
```

## 14.3.3. Mise à jour des applications

La mise à jour des applications WPF est un aspect crucial du cycle de vie de l'application. Différentes méthodes existent selon votre stratégie de déploiement.

### Mise à jour avec ClickOnce

ClickOnce offre un mécanisme intégré pour les mises à jour automatiques.

**Configuration des mises à jour ClickOnce :**

1. **Définissez la stratégie de mise à jour**
   - Ouvrez les propriétés du projet → Onglet "Publish"
   - Cliquez sur "Updates" pour ouvrir la boîte de dialogue de configuration

2. **Options de configuration**
   - **Vérifier les mises à jour :** À chaque démarrage / À intervalles réguliers
   - **Intervalle de vérification :** Quotidien, hebdomadaire, etc.
   - **URL de mise à jour :** Emplacement où l'application vérifiera les nouvelles versions

3. **Mise à jour obligatoire**
   - Vous pouvez forcer les utilisateurs à utiliser la dernière version
   - Cochez "L'application doit être mise à jour avant utilisation"

4. **Déploiement d'une mise à jour**
   - Incrémentez le numéro de version dans les propriétés de l'assembly
   - Republiez l'application au même emplacement

**Vérification programmatique des mises à jour :**

```csharp
private void VerifierMisesAJour()
{
    try
    {
        ApplicationDeployment deploiement = ApplicationDeployment.CurrentDeployment;
        UpdateCheckInfo infos = deploiement.CheckForDetailedUpdate();

        if (infos.UpdateAvailable)
        {
            bool doUpdate = true;
            if (!infos.IsUpdateRequired)
            {
                // Demander à l'utilisateur s'il veut mettre à jour
                doUpdate = MessageBox.Show("Une mise à jour est disponible. Souhaitez-vous l'installer maintenant ?",
                                           "Mise à jour disponible", MessageBoxButton.YesNo) == MessageBoxResult.Yes;
            }

            if (doUpdate)
            {
                deploiement.Update();
                MessageBox.Show("L'application a été mise à jour et va redémarrer.");
                System.Windows.Forms.Application.Restart();
                Application.Current.Shutdown();
            }
        }
    }
    catch (DeploymentDownloadException dde)
    {
        MessageBox.Show("La mise à jour a échoué : " + dde.Message);
    }
    catch (InvalidDeploymentException ide)
    {
        MessageBox.Show("L'application n'a pas été déployée avec ClickOnce.");
    }
}
```

### Mise à jour avec MSIX

MSIX offre un mécanisme moderne pour les mises à jour d'applications.

**Avantages des mises à jour MSIX :**
- Mise à jour sans interruption de l'application en cours d'exécution
- Restauration automatique en cas d'échec
- Mise à jour partielle (seuls les fichiers modifiés sont téléchargés)

**Déploiement d'une mise à jour MSIX :**
1. Incrémentez la version dans le manifeste du package
2. Recompilez et redistribuez le package MSIX

### Mise à jour personnalisée

Pour les applications ne pouvant pas utiliser ClickOnce ou MSIX, vous pouvez implémenter votre propre système de mise à jour.

**Composants d'un système de mise à jour personnalisé :**

1. **Vérification des mises à jour**
   - Serveur Web ou service API qui fournit les informations de version
   - Client qui vérifie périodiquement les nouvelles versions

```csharp
public class VerificateurMisesAJour
{
    private readonly string _urlVerification;
    private readonly Version _versionActuelle;

    public VerificateurMisesAJour(string urlVerification)
    {
        _urlVerification = urlVerification;
        _versionActuelle = Assembly.GetExecutingAssembly().GetName().Version;
    }

    public async Task<MiseAJourInfo> VerifierMiseAJourDisponible()
    {
        using (HttpClient client = new HttpClient())
        {
            try
            {
                string json = await client.GetStringAsync(_urlVerification);
                var infoMiseAJour = JsonSerializer.Deserialize<MiseAJourInfo>(json);

                if (Version.Parse(infoMiseAJour.DerniereVersion) > _versionActuelle)
                {
                    return infoMiseAJour;
                }

                return null; // Pas de mise à jour disponible
            }
            catch (Exception ex)
            {
                // Gérer les erreurs de connexion
                return null;
            }
        }
    }
}

public class MiseAJourInfo
{
    public string DerniereVersion { get; set; }
    public string UrlTelechargement { get; set; }
    public string NotesVersion { get; set; }
    public bool MiseAJourObligatoire { get; set; }
}
```

2. **Téléchargement et installation**
   - Téléchargez le nouveau package d'installation
   - Fermez l'application actuelle
   - Exécutez le nouveau programme d'installation

```csharp
public async Task TelechargerEtInstallerMiseAJour(MiseAJourInfo info)
{
    string fichierTemp = Path.Combine(Path.GetTempPath(), "MiseAJour.exe");

    using (HttpClient client = new HttpClient())
    {
        using (var response = await client.GetAsync(info.UrlTelechargement, HttpCompletionOption.ResponseHeadersRead))
        {
            response.EnsureSuccessStatusCode();

            using (var streamSource = await response.Content.ReadAsStreamAsync())
            using (var streamDestination = new FileStream(fichierTemp, FileMode.Create, FileAccess.Write, FileShare.None))
            {
                await streamSource.CopyToAsync(streamDestination);
            }
        }
    }

    // Lancer l'installateur et fermer l'application actuelle
    Process.Start(new ProcessStartInfo
    {
        FileName = fichierTemp,
        Arguments = "/silent", // Pour une installation silencieuse
        UseShellExecute = true
    });

    Application.Current.Shutdown();
}
```

3. **Notification à l'utilisateur**
   - Informez l'utilisateur des mises à jour disponibles
   - Proposez d'installer maintenant ou plus tard

```xml
<Window x:Class="MonApplication.FenetreMiseAJour"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Mise à jour disponible" Height="300" Width="450">
    <StackPanel Margin="20">
        <TextBlock Text="Une nouvelle version est disponible !" FontSize="18" FontWeight="Bold"/>
        <TextBlock x:Name="txtVersion" Text="Version 1.0.2" Margin="0,10,0,0"/>
        <TextBlock Text="Notes de version :" Margin="0,10,0,0" FontWeight="Bold"/>
        <TextBox x:Name="txtNotes" Height="100" Margin="0,5,0,0" IsReadOnly="True"
                 TextWrapping="Wrap" VerticalScrollBarVisibility="Auto"/>
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right" Margin="0,20,0,0">
            <Button Content="Mettre à jour plus tard" Width="150" Margin="0,0,10,0" Click="BtnPlus_Tard_Click"/>
            <Button Content="Installer maintenant" Width="150" Click="BtnInstaller_Click"/>
        </StackPanel>
    </StackPanel>
</Window>
```

### Stratégies de mise à jour recommandées

1. **Cadence de mise à jour**
   - Définissez une cadence prévisible (mensuelle, trimestrielle)
   - Communiquez cette cadence aux utilisateurs

2. **Mises à jour progressives**
   - Déployez d'abord auprès d'un groupe pilote
   - Étendez progressivement à tous les utilisateurs

3. **Delta updates**
   - Si possible, utilisez des mises à jour différentielles (seuls les fichiers modifiés)
   - Réduit le temps de téléchargement et d'installation

4. **Annonces de fin de support**
   - Informez clairement des versions qui ne seront plus prises en charge
   - Encouragez les utilisateurs à mettre à jour

## 14.3.4. Sécurité et permissions

Les applications WPF, comme toutes les applications Windows, doivent prendre en compte les problématiques de sécurité et de gestion des permissions.

### Niveau de confiance et modèle de sécurité

#### Applications à confiance totale vs confiance partielle

Les applications WPF fonctionnent généralement en confiance totale, ce qui signifie qu'elles ont un accès complet aux ressources du système.

**Considérations sur la confiance :**
- Les applications ClickOnce peuvent fonctionner en confiance partielle
- La confiance partielle limite l'accès à certaines ressources (système de fichiers, registre, etc.)
- La confiance totale requiert généralement des droits d'administrateur pour l'installation

### Signature de code

La signature numérique de votre application permet de garantir son authenticité et son intégrité.

**Pourquoi signer votre application :**
- Gain de confiance des utilisateurs
- Évite les avertissements de sécurité Windows
- Nécessaire pour certains types de déploiement (MSIX, Store)

**Comment signer votre application :**

1. **Obtenir un certificat**
   - Achetez un certificat auprès d'une autorité de certification (Comodo, DigiCert, etc.)
   - Ou créez un certificat auto-signé pour les tests

2. **Signer l'application avec SignTool**
   - Utilisez l'outil SignTool.exe fourni avec le SDK Windows
   ```
   signtool sign /f MyCertificate.pfx /p password /d "Description" /t http://timestamp.digicert.com MonApplication.exe
   ```

3. **Signature dans Visual Studio**
   - Ouvrez les propriétés du projet
   - Accédez à l'onglet "Signature"
   - Choisissez le fichier de certificat (.pfx)
   - Activez l'option "Signer l'assembly"

### Élévation de privilèges et manifeste d'application

Les applications WPF peuvent nécessiter des privilèges élevés pour certaines opérations.

**Configuration du manifeste pour l'élévation de privilèges :**

1. **Ouvrez le fichier app.manifest**
   - Si le fichier n'existe pas, ajoutez-le au projet
   - Clic droit sur le projet → Ajouter → Nouvel élément → Fichier manifeste d'application

2. **Configurez le niveau de privilèges requis**
   ```xml
   <requestedExecutionLevel level="requireAdministrator" uiAccess="false" />
   ```

   Options pour le niveau :
   - **asInvoker** : Même niveau que le processus parent (par défaut)
   - **requireAdministrator** : Demande des privilèges administrateur
   - **highestAvailable** : Le plus haut niveau disponible pour l'utilisateur

3. **Activez le manifeste dans les propriétés du projet**
   - Ouvrez les propriétés du projet
   - Accédez à l'onglet "Application"
   - Dans "Manifeste", sélectionnez "Utiliser ce fichier manifeste"

### Sécurisation des données sensibles

Les applications WPF manipulent souvent des données sensibles qui doivent être protégées.

**1. Protection des chaînes de connexion**

```csharp
// Chiffrement des chaînes de connexion
private string ChiffrerChaineConnexion(string chaineConnexion)
{
    byte[] donnees = Encoding.UTF8.GetBytes(chaineConnexion);
    byte[] donneesChiffrees = ProtectedData.Protect(
        donnees,
        null,
        DataProtectionScope.CurrentUser);

    return Convert.ToBase64String(donneesChiffrees);
}

// Déchiffrement des chaînes de connexion
private string DechiffrerChaineConnexion(string chaineChiffree)
{
    byte[] donneesChiffrees = Convert.FromBase64String(chaineChiffree);
    byte[] donnees = ProtectedData.Unprotect(
        donneesChiffrees,
        null,
        DataProtectionScope.CurrentUser);

    return Encoding.UTF8.GetString(donnees);
}
```

**2. Stockage sécurisé des informations d'identification**

```csharp
// Utilisation du gestionnaire d'informations d'identification Windows
public void EnregistrerIdentifiants(string utilisateur, string motDePasse)
{
    var credential = new Credential
    {
        Target = "MonApplicationWPF",
        UserName = utilisateur,
        Password = motDePasse,
        Type = CredentialType.Generic,
        PersistanceType = PersistanceType.LocalComputer
    };

    credential.Save();
}

public bool RecupererIdentifiants(out string utilisateur, out string motDePasse)
{
    utilisateur = null;
    motDePasse = null;

    var credential = new Credential { Target = "MonApplicationWPF" };
    if (credential.Load())
    {
        utilisateur = credential.UserName;
        motDePasse = credential.Password;
        return true;
    }

    return false;
}
```

**3. Protection contre les attaques XAML injection**

Les applications WPF qui chargent dynamiquement du XAML doivent se protéger contre les injections XAML.

```csharp
// Méthode non sécurisée - ÉVITER
private void ChargementXamlNonSecurise(string xamlString)
{
    // DANGEREUX : peut exécuter du code malveillant
    var element = XamlReader.Parse(xamlString);
    monConteneur.Children.Add(element as UIElement);
}

// Méthode sécurisée - PRÉFÉRER
private void ChargementXamlSecurise(string xamlString)
{
    // Créer un ParserContext restrictif
    var context = new ParserContext();
    var restriction = new XamlReaderSettings();
    restriction.AllowEventHandlers = false; // Empêche les gestionnaires d'événements
    context.XamlReaderSettings = restriction;

    try
    {
        var element = XamlReader.Parse(xamlString, context);
        monConteneur.Children.Add(element as UIElement);
    }
    catch (XamlParseException ex)
    {
        // Gérer l'exception de façon sécurisée
        Logger.LogError($"Échec de l'analyse XAML: {ex.Message}");
    }
}
```

### Bonnes pratiques de sécurité pour les applications WPF

1. **Principe du moindre privilège**
   - Ne demandez que les permissions nécessaires au fonctionnement de l'application
   - Séparez les composants qui nécessitent des privilèges élevés

2. **Validation des entrées utilisateur**
   - Validez toutes les entrées utilisateur avant traitement
   - Utilisez des expressions régulières pour valider les formats

```csharp
private bool EstEmailValide(string email)
{
    // Expression régulière pour valider un format d'email
    string pattern = @"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
    return Regex.IsMatch(email, pattern);
}
```

3. **Logs de sécurité**
   - Journalisez les événements de sécurité importants
   - Stockez les logs de façon sécurisée

```csharp
public void JournaliserEvenementSecurite(string message, SeveriteEvenement severite)
{
    // Journalisation sécurisée avec horodatage et contexte utilisateur
    string utilisateurActuel = Thread.CurrentPrincipal?.Identity?.Name ?? "Inconnu";
    string entreeLog = $"{DateTime.Now:yyyy-MM-dd HH:mm:ss} | {severite} | {utilisateurActuel} | {message}";

    // Écriture dans un fichier log sécurisé avec accès restreint
    File.AppendAllText(
        Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.CommonApplicationData),
        "MonApplication", "SecureLogs.txt"),
        entreeLog + Environment.NewLine);
}

public enum SeveriteEvenement
{
    Information,
    Avertissement,
    Erreur,
    Critique
}
```

4. **Mises à jour de sécurité**
   - Intégrez un processus pour déployer rapidement les correctifs de sécurité
   - Implémentez un système de notification pour les vulnérabilités critiques

## 14.3.5. Installation silencieuse et automatisée

L'installation silencieuse permet de déployer des applications WPF sans intervention de l'utilisateur, ce qui est particulièrement utile pour les déploiements en entreprise.

### Installation silencieuse avec MSI

Les installateurs MSI prennent en charge l'installation silencieuse via des options de ligne de commande.

**Commandes d'installation silencieuse :**

```batch
# Installation silencieuse complète (aucune interface utilisateur)
msiexec /i MonApplicationWPF.msi /qn

# Installation silencieuse avec barre de progression
msiexec /i MonApplicationWPF.msi /qb

# Installation silencieuse avec journalisation
msiexec /i MonApplicationWPF.msi /qn /l*v C:\Logs\installation.log

# Installation silencieuse avec paramètres personnalisés
msiexec /i MonApplicationWPF.msi /qn INSTALLDIR="C:\Applications\MonAppWPF" ALLUSERS=1
```

**Paramètres MSI courants :**

| Paramètre | Description |
|-----------|-------------|
| /i | Installation du package |
| /x | Désinstallation du package |
| /qn | Aucune interface utilisateur |
| /qb | Interface utilisateur de base (barre de progression) |
| /l*v | Journalisation détaillée |
| INSTALLDIR | Répertoire d'installation personnalisé |
| ALLUSERS | 1 = installation pour tous les utilisateurs, 0 = utilisateur actuel uniquement |

### Installation silencieuse avec WiX

Avec un installateur WiX, vous pouvez définir des propriétés par défaut dans votre fichier wxs pour faciliter l'installation silencieuse :

```xml
<Property Id="WIXUI_INSTALLDIR" Value="INSTALLFOLDER" />
<Property Id="ALLUSERS" Value="1" />
<UIRef Id="WixUI_InstallDir" />

<!-- Pour support de l'installation silencieuse -->
<Property Id="WIXUI_EXITDIALOGOPTIONALTEXT" Value="Installation terminée avec succès." />
<UI>
    <Property Id="WIXUI_ENABLEWELCOMEDIALOG" Value="0" />
    <Property Id="WIXUI_EXITDIALOGOPTIONALCHECKBOX" Value="0" />
</UI>
```

### Installation silencieuse avec ClickOnce

ClickOnce prend également en charge les installations non interactives :

```batch
# Installation silencieuse ClickOnce
start /wait MonApplication.application /silent

# Avec paramètres
start /wait MonApplication.application /silent /parameterName:value
```

### Déploiement automatisé avec des scripts

Pour les déploiements à grande échelle, utilisez des scripts d'automatisation.

**Script PowerShell pour déploiement automatisé :**

```powershell
# Script de déploiement automatisé
param(
    [string]$InstallerPath,
    [string]$LogPath = "C:\Logs\Installation",
    [string]$InstallDir = "C:\Program Files\MonApplicationWPF",
    [switch]$ForceReinstall
)

# Créer le répertoire de logs
if (-not (Test-Path $LogPath)) {
    New-Item -Path $LogPath -ItemType Directory -Force
}

$logFile = Join-Path $LogPath "installation_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

# Vérifier si l'application est déjà installée
$appInstalled = Get-WmiObject -Class Win32_Product | Where-Object { $_.Name -eq "MonApplicationWPF" }

if ($appInstalled -and -not $ForceReinstall) {
    Write-Output "L'application est déjà installée. Version: $($appInstalled.Version)"
    Write-Output "Pour réinstaller, utilisez le paramètre -ForceReinstall"
    Exit 0
}

if ($appInstalled -and $ForceReinstall) {
    Write-Output "Désinstallation de la version existante..."
    $uninstallResult = Start-Process -FilePath "msiexec.exe" -ArgumentList "/x $($appInstalled.IdentifyingNumber) /qn" -Wait -PassThru
    if ($uninstallResult.ExitCode -ne 0) {
        Write-Error "Échec de la désinstallation. Code: $($uninstallResult.ExitCode)"
        Exit 1
    }
}

# Installation de l'application
Write-Output "Installation de MonApplicationWPF..."
$installArgs = "/i `"$InstallerPath`" /qn INSTALLDIR=`"$InstallDir`" /l*v `"$logFile`" ALLUSERS=1"
$installResult = Start-Process -FilePath "msiexec.exe" -ArgumentList $installArgs -Wait -PassThru

if ($installResult.ExitCode -eq 0) {
    Write-Output "Installation réussie!"
} else {
    Write-Error "Échec de l'installation. Code: $($installResult.ExitCode)"
    Write-Error "Consultez le journal pour plus de détails: $logFile"
    Exit 1
}

# Vérification post-installation
$exePath = Join-Path $InstallDir "MonApplicationWPF.exe"
if (Test-Path $exePath) {
    Write-Output "Vérification réussie. L'application est installée à: $exePath"
} else {
    Write-Error "Vérification échouée. Le fichier exécutable n'a pas été trouvé."
    Exit 1
}

Exit 0
```

### Déploiement automatisé avec des outils de gestion de système

Pour les environnements d'entreprise, utilisez des outils de gestion de système pour automatiser les déploiements.

#### 1. System Center Configuration Manager (SCCM)

SCCM permet de déployer des applications à l'échelle de l'entreprise.

**Étapes de base pour déployer avec SCCM :**
1. Créez un package SCCM contenant votre installateur MSI
2. Configurez un programme d'installation silencieuse
3. Créez un déploiement ciblant des collections d'ordinateurs
4. Planifiez le déploiement (immédiat ou programmé)

#### 2. Group Policy Software Installation (GPSI)

Utilisez les stratégies de groupe Windows pour déployer des applications MSI.

**Configuration de base pour GPSI :**
1. Ouvrez la Console de gestion des stratégies de groupe (GPMC)
2. Créez ou modifiez une GPO
3. Naviguez vers "Configuration ordinateur" > "Paramètres logiciels" > "Installation de logiciels"
4. Clic droit > "Nouveau" > "Package"
5. Sélectionnez votre fichier MSI depuis un partage réseau
6. Choisissez "Affecté" pour l'installation automatique

#### 3. Intune / Microsoft Endpoint Manager

Pour les environnements modernes, utilisez Intune pour le déploiement d'applications.

**Étapes de base pour déployer avec Intune :**
1. Téléchargez votre application (MSI ou MSIX) sur Intune
2. Configurez les paramètres d'installation
3. Assignez l'application à des groupes d'utilisateurs ou d'appareils
4. Définissez l'intention de déploiement (Disponible ou Requis)

### Considérations pour les installations automatisées

1. **Gestion des redémarrages**
   - Spécifiez les comportements de redémarrage
   - Pour MSI : `/norestart` ou `/forcerestart`

2. **Vérification du succès de l'installation**
   - Vérifiez les codes de sortie du processus d'installation
   - Implémentez des vérifications post-installation

3. **Gestion des dépendances**
   - Assurez-vous que toutes les dépendances sont installées
   - Utilisez des détections de prérequis

```powershell
# Vérification de .NET Framework
$dotNetVersion = Get-ChildItem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP" -Recurse |
                 Get-ItemProperty -Name Version -ErrorAction SilentlyContinue |
                 Where-Object { $_.PSChildName -eq "Full" } |
                 ForEach-Object { $_.Version }

if ($dotNetVersion -lt [Version]"4.8") {
    Write-Error ".NET Framework 4.8 ou supérieur est requis pour cette application."
    Exit 1
}
```

4. **Installation parallèle de plusieurs versions**
   - Utilisez différents identifiants de produit (ProductCode) pour chaque version
   - Configurez des chemins d'installation distincts

```xml
<!-- Configuration WiX pour support multi-version -->
<Product Id="*" Name="MonApplicationWPF $(var.Version)" Version="$(var.Version)"
        Manufacturer="MaSociété" UpgradeCode="PUT-GUID-HERE">
    <!-- Utiliser un nouvel ID de produit pour chaque version -->
    <Package InstallScope="perMachine" InstallPrivileges="elevated" Compressed="yes" />

    <!-- Ne pas désinstaller les versions précédentes -->
    <MajorUpgrade AllowSameVersionUpgrades="yes"
                  DowngradeErrorMessage="Une version plus récente est déjà installée."
                  Schedule="afterInstallInitialize"
                  MigrateFeatures="no" />

    <!-- Installer dans un dossier avec numéro de version -->
    <Directory Id="TARGETDIR" Name="SourceDir">
        <Directory Id="ProgramFilesFolder">
            <Directory Id="COMPANYFOLDER" Name="MaSociété">
                <Directory Id="INSTALLFOLDER" Name="MonApplicationWPF $(var.Version)">
                    <!-- Contenu de l'application -->
                </Directory>
            </Directory>
        </Directory>
    </Directory>
</Product>
```

5. **Rollback en cas d'échec**
   - Prévoyez une stratégie de restauration
   - Sauvegardez les données importantes avant la mise à jour

```powershell
# Sauvegarde avant installation
$backupDir = "$env:TEMP\AppBackup_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
New-Item -Path $backupDir -ItemType Directory -Force

# Sauvegarde des fichiers de configuration
if (Test-Path "$env:APPDATA\MonApplicationWPF\settings.xml") {
    Copy-Item "$env:APPDATA\MonApplicationWPF\settings.xml" -Destination $backupDir
}

# Après échec d'installation
if ($installResult.ExitCode -ne 0) {
    Write-Warning "Installation échouée, restauration des paramètres..."
    if (Test-Path "$backupDir\settings.xml") {
        Copy-Item "$backupDir\settings.xml" -Destination "$env:APPDATA\MonApplicationWPF\" -Force
    }
}
```

## Résumé

Le déploiement d'applications WPF offre de nombreuses options, de l'installation classique aux méthodes automatisées pour les environnements d'entreprise. Points clés à retenir :

1. **Packaging et installation** : Choisissez entre MSI, WiX, MSIX ou ClickOnce selon vos besoins spécifiques.

2. **Ressources et actifs** : Gérez correctement les ressources de votre application, en particulier pour les applications riches en contenu multimédia.

3. **Mise à jour des applications** : Implémentez une stratégie de mise à jour efficace, que ce soit via ClickOnce, MSIX ou un système personnalisé.

4. **Sécurité et permissions** : Sécurisez votre application en suivant les meilleures pratiques de signature et de gestion des permissions.

5. **Installation silencieuse et automatisée** : Utilisez les options d'installation silencieuse et les outils de déploiement d'entreprise pour les déploiements à grande échelle.

En maîtrisant ces aspects du déploiement, vous pourrez distribuer vos applications WPF de manière efficace et sécurisée, tout en offrant une expérience utilisateur optimale lors de l'installation et des mises à jour.

## Exercices pratiques

1. Créez un projet WPF simple et publiez-le en utilisant ClickOnce avec vérification automatique des mises à jour.

2. Construisez un installateur MSI pour votre application WPF qui vérifie la présence de .NET Framework et installe les composants prérequis si nécessaire.

3. Créez un script PowerShell pour déployer automatiquement votre application WPF sur plusieurs ordinateurs dans un réseau local.

4. Implémentez un système de mise à jour personnalisé pour votre application WPF qui télécharge et installe les mises à jour depuis un serveur Web.

5. Configurez votre application WPF pour qu'elle fonctionne correctement avec différents niveaux de privilèges utilisateur en utilisant le manifeste d'application.

⏭️
