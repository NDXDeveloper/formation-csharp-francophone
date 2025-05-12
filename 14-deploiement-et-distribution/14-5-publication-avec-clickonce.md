# 14.5. Publication avec ClickOnce

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction

ClickOnce est une technologie de déploiement développée par Microsoft qui permet de publier des applications Windows de manière simple et sécurisée. Elle est particulièrement adaptée aux applications qui nécessitent des mises à jour fréquentes et qui doivent être accessibles facilement par les utilisateurs finaux, sans nécessiter de droits d'administrateur pour l'installation.

Dans ce chapitre, nous allons explorer comment configurer, personnaliser et déployer des applications .NET à l'aide de ClickOnce.

## 14.5.1. Configuration

### Qu'est-ce que ClickOnce ?

ClickOnce est une technologie de déploiement qui permet :
- D'installer des applications depuis un serveur web, un partage réseau ou un support amovible
- De mettre à jour automatiquement les applications lorsqu'une nouvelle version est disponible
- D'installer des applications sans avoir besoin de droits d'administrateur
- De fonctionner en ligne et hors ligne

### Prérequis

Pour utiliser ClickOnce, vous aurez besoin :
- Visual Studio (Community, Professional ou Enterprise)
- Un projet d'application Windows (.NET Framework ou .NET Core / .NET 5+)
- Un emplacement pour héberger votre application (serveur web, partage réseau, etc.)

### Configuration de base avec Visual Studio

Voici comment configurer une publication ClickOnce pour votre application :

1. **Ouvrez les propriétés du projet**
   - Dans l'Explorateur de solutions, faites un clic droit sur votre projet
   - Sélectionnez "Propriétés"

2. **Accédez à l'onglet "Publier"**
   - Dans les versions récentes de Visual Studio, cliquez sur "Publier" dans le panneau de gauche
   - Dans les versions plus anciennes, cliquez directement sur l'onglet "Publier"

3. **Créez un nouveau profil de publication**
   - Cliquez sur "Nouveau" ou "+"
   - Sélectionnez "ClickOnce" comme méthode de déploiement
   - Cliquez sur "Suivant"

4. **Choisissez l'emplacement de publication**
   - Serveur web (FTP, FTPS)
   - Dossier (local ou partage réseau)
   - Azure Storage
   - Site web Azure

   Exemple pour un dossier local :
   ```
   C:\Deployment\MonApplication
   ```

   Exemple pour un serveur web :
   ```
   https://monserveur.com/applications/MonApplication/
   ```

5. **Configurez la méthode d'installation**
   - **À partir du Web** : Installation depuis un navigateur
   - **À partir d'un dossier ou d'un partage** : Installation depuis l'explorateur de fichiers
   - **À partir d'un CD/DVD** : Installation depuis un support amovible

6. **Définissez les paramètres de publication**
   - **Emplacement d'installation** : Où l'application sera installée sur l'ordinateur client
   - **Pour qui l'application est-elle disponible** : Juste pour l'utilisateur actuel ou pour tous les utilisateurs (requiert des droits administrateur)
   - **L'application est-elle disponible hors ligne** : Si l'application peut être exécutée sans connexion Internet

7. **Spécifiez la version de l'application**
   - Version initiale (généralement 1.0.0)
   - Méthode de numérotation automatique des versions

8. **Configurez les prérequis**
   - Cliquez sur "Prérequis" pour définir les composants nécessaires
   - Incluez le runtime .NET requis
   - Autres dépendances (SQL Server Express, etc.)

9. **Publiez l'application**
   - Cliquez sur "Terminer"
   - Puis sur "Publier"

### Configuration via ligne de commande

Vous pouvez également configurer et publier une application ClickOnce via la ligne de commande ou un script :

```batch
msbuild MonApplication.csproj /target:publish /property:Configuration=Release /property:PublishDir=C:\Deployment\MonApplication\ /property:ApplicationVersion=1.0.0.0 /property:InstallUrl=https://monserveur.com/applications/MonApplication/ /property:UpdateEnabled=true /property:UpdateMode=Foreground
```

### Structure des fichiers publiés

Après la publication, vous trouverez les fichiers suivants à l'emplacement de publication :

- **Application Files** : Dossier contenant les versions de l'application dans des sous-dossiers
- **setup.exe** : Programme d'installation facultatif qui vérifie et installe les prérequis
- **NomApplication.application** : Fichier de manifeste principal qui décrit l'application
- **publish.htm** : Page web utilisée pour l'installation depuis un navigateur

Chaque dossier de version contient :
- Les fichiers de l'application (.exe, .dll, etc.)
- Un fichier manifeste (.manifest) qui décrit la version
- Une signature numérique si l'application est signée

## 14.5.2. Mise à jour automatique

L'un des avantages majeurs de ClickOnce est sa capacité à mettre à jour automatiquement les applications.

### Configuration des mises à jour

Pour configurer les mises à jour automatiques :

1. **Ouvrez les propriétés du projet**
2. **Accédez à l'onglet "Publier"**
3. **Cliquez sur "Mises à jour"**

Vous pouvez configurer les paramètres suivants :

- **L'application doit-elle rechercher des mises à jour ?**
   - Activer/désactiver les mises à jour automatiques

- **Fréquence de vérification**
   - À chaque démarrage de l'application
   - À intervalles spécifiés (quotidien, hebdomadaire, etc.)

- **URL de mise à jour**
   - Emplacement où l'application cherchera les mises à jour
   - Par défaut, identique à l'URL d'installation, mais peut être différent

- **Mise à jour minimale requise**
   - Forcer les utilisateurs à installer la dernière version
   - Bloquer l'utilisation si la mise à jour n'est pas effectuée

- **Comportement de mise à jour**
   - **Avant le démarrage de l'application** : Vérifie et installe les mises à jour avant de lancer l'application
   - **Après le démarrage de l'application** : Vérifie en arrière-plan et propose les mises à jour

### Exemple de configuration de mises à jour automatiques

```xml
<!-- Dans le fichier de projet (.csproj) -->
<Project>
  <PropertyGroup>
    <UpdateEnabled>true</UpdateEnabled>
    <UpdateMode>Foreground</UpdateMode>
    <UpdateInterval>7</UpdateInterval>
    <UpdateIntervalUnits>Days</UpdateIntervalUnits>
    <UpdatePeriodically>true</UpdatePeriodically>
    <UpdateRequired>false</UpdateRequired>
    <MinimumRequiredVersion>1.0.0.0</MinimumRequiredVersion>
  </PropertyGroup>
</Project>
```

### Publication d'une mise à jour

Pour publier une mise à jour :

1. **Modifiez votre code source** selon les besoins
2. **Incrémentez la version dans les propriétés de l'assembly**
   - Ouvrez les propriétés du projet
   - Accédez à l'onglet "Application" > "Informations d'assembly"
   - Mettez à jour les numéros de version

3. **Publiez à nouveau l'application**
   - Utilisez le même profil de publication
   - Publiez au même emplacement

Lorsque les utilisateurs démarreront leur application, ils seront informés de la disponibilité d'une mise à jour.

### Vérification programmatique des mises à jour

Vous pouvez également vérifier et installer les mises à jour depuis votre code :

```csharp
// Vérifier si une mise à jour est disponible
private async void VerifierMisesAJour()
{
    try
    {
        // Obtenir l'instance de déploiement actuelle
        var deploymentInfo = ApplicationDeployment.CurrentDeployment;

        // Vérifier les mises à jour de manière asynchrone
        var updateInfo = await Task.Run(() => deploymentInfo.CheckForDetailedUpdate());

        if (updateInfo.UpdateAvailable)
        {
            // Une mise à jour est disponible
            var resultat = MessageBox.Show(
                $"Une mise à jour est disponible (version {updateInfo.AvailableVersion}).\n" +
                "Voulez-vous l'installer maintenant ?",
                "Mise à jour disponible",
                MessageBoxButtons.YesNo);

            if (resultat == DialogResult.Yes)
            {
                await Task.Run(() => deploymentInfo.Update());
                MessageBox.Show("Mise à jour installée. L'application va redémarrer.");
                Application.Restart();
            }
        }
    }
    catch (DeploymentDownloadException ex)
    {
        MessageBox.Show("Erreur lors du téléchargement de la mise à jour : " + ex.Message);
    }
    catch (InvalidDeploymentException)
    {
        // L'application n'a pas été déployée avec ClickOnce
        // Ou est exécutée en mode développement
    }
}
```

### Gestion des versions minimales requises

Si vous devez forcer les utilisateurs à utiliser une version minimale :

1. **Définissez la version minimale**
   - Dans les paramètres de mise à jour, activez "Exiger que les utilisateurs utilisent la dernière version"
   - Définissez la version minimale requise

2. **Publiez l'application**

Lorsqu'un utilisateur avec une version antérieure lance l'application, il est forcé de mettre à jour avant de pouvoir l'utiliser.

## 14.5.3. Personnalisation

ClickOnce permet de personnaliser l'expérience d'installation et de mise à jour.

### Page d'installation personnalisée

Vous pouvez personnaliser la page web d'installation (publish.htm) :

1. **Créez un modèle personnalisé**
   - Copiez le fichier publish.htm généré automatiquement
   - Modifiez-le selon vos besoins (logo, couleurs, instructions, etc.)

2. **Configurez l'utilisation du modèle personnalisé**
   - Dans les propriétés de publication, cochez "Utiliser un modèle de déploiement personnalisé"
   - Sélectionnez votre fichier HTML personnalisé

### Personnalisation de l'interface utilisateur d'installation

Vous pouvez personnaliser l'apparence du programme d'installation :

1. **Ajoutez une icône d'application**
   - Dans les propriétés du projet, accédez à l'onglet "Application"
   - Définissez l'icône de l'application

2. **Personnalisez les informations d'application**
   - Nom de l'éditeur
   - Nom du produit
   - Description
   - URL du support

```xml
<!-- Dans le fichier .csproj -->
<PropertyGroup>
  <PublisherName>Ma Société</PublisherName>
  <ProductName>Mon Application Incroyable</ProductName>
  <ApplicationIcon>app_icon.ico</ApplicationIcon>
  <SuiteName>Ma Suite d'Applications</SuiteName>
  <SupportUrl>https://support.masociete.com</SupportUrl>
</PropertyGroup>
```

### Paramètres de lancement

Vous pouvez configurer comment l'application se lance après l'installation :

1. **Options du bureau**
   - Créer un raccourci sur le bureau
   - Définir un nom personnalisé pour le raccourci

2. **Options du menu Démarrer**
   - Créer un dossier dans le menu Démarrer
   - Définir un nom personnalisé pour le dossier

3. **Paramètres d'activation**
   - Lancer l'application après l'installation
   - Lancer l'application à chaque connexion de l'utilisateur

### Paramètres supplémentaires au lancement

Vous pouvez passer des paramètres à votre application lors du lancement depuis ClickOnce :

```csharp
private void FormMain_Load(object sender, EventArgs e)
{
    if (ApplicationDeployment.IsNetworkDeployed)
    {
        // Récupérer les paramètres de déploiement
        string[] activationData = AppDomain.CurrentDomain.SetupInformation.ActivationArguments.ActivationData;

        if (activationData != null && activationData.Length > 0)
        {
            // Le premier élément est généralement l'URL d'activation
            string url = activationData[0];

            // Analyser les paramètres de l'URL
            Uri uri = new Uri(url);
            string query = uri.Query;

            // Traiter les paramètres
            // ...
        }
    }
}
```

### Fichiers de données

Vous pouvez également inclure des fichiers de données dans votre déploiement ClickOnce :

1. **Ajoutez des fichiers au projet**
   - Faites un clic droit sur le projet > "Ajouter" > "Élément existant"
   - Sélectionnez les fichiers à inclure

2. **Configurez les propriétés de publication**
   - Sélectionnez le fichier dans l'Explorateur de solutions
   - Dans la fenêtre Propriétés, définissez "Copier dans le répertoire de sortie" sur "Copier si plus récent"
   - Définissez "Action de génération" sur "Contenu"

3. **Accès aux fichiers de données**
   - Utilisez `Application.StartupPath` pour localiser les fichiers

```csharp
string cheminFichier = Path.Combine(Application.StartupPath, "donnees.xml");
```

## 14.5.4. Sécurité et signature

La signature numérique est un aspect important de la sécurité des applications ClickOnce. Elle garantit l'intégrité de l'application et confirme l'identité de l'éditeur.

### Pourquoi signer une application ClickOnce ?

- **Confiance de l'utilisateur** : Lorsque les utilisateurs installent une application signée, ils peuvent vérifier son authenticité
- **Éviter les avertissements de sécurité** : Les applications non signées déclenchent des avertissements de sécurité
- **Vérification d'intégrité** : Garantit que l'application n'a pas été modifiée après sa publication
- **Mises à jour automatiques** : Permet les mises à jour automatiques entre différentes versions signées

### Types de certificats

Plusieurs types de certificats peuvent être utilisés pour signer une application ClickOnce :

1. **Certificats auto-signés**
   - Créés par vous-même
   - Utiles pour les tests et les déploiements internes
   - Ne sont pas automatiquement approuvés par les ordinateurs clients

2. **Certificats d'autorité de certification (CA)**
   - Achetés auprès d'une autorité de certification reconnue (DigiCert, GlobalSign, etc.)
   - Automatiquement approuvés par la plupart des ordinateurs
   - Recommandés pour les applications commerciales ou publiques

### Création d'un certificat auto-signé

Pour les tests ou les déploiements internes, vous pouvez créer un certificat auto-signé :

1. **Utilisation de Visual Studio**
   - Dans les propriétés du projet, accédez à l'onglet "Signature"
   - Cliquez sur "Créer un certificat de test"
   - Remplissez les informations demandées
   - Cliquez sur "OK"

2. **Utilisation de PowerShell**
   ```powershell
   # Créer un certificat auto-signé
   New-SelfSignedCertificate -Type Custom -Subject "CN=Ma Société, O=Mon Organisation, C=FR" -KeyUsage DigitalSignature -FriendlyName "Mon Certificat de Signature" -CertStoreLocation "Cert:\CurrentUser\My" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3", "2.5.29.19={text}")
   ```

3. **Utilisation de l'outil de ligne de commande makecert**
   ```batch
   makecert -r -pe -n "CN=Ma Société" -ss My -sr CurrentUser -eku 1.3.6.1.5.5.7.3.3 MyCertificate.cer
   ```

### Installation d'un certificat de confiance

Pour que les utilisateurs puissent installer votre application sans avertissements, ils doivent faire confiance à votre certificat :

1. **Pour les certificats auto-signés**
   - Exportez le certificat (fichier .cer)
   - Distribuez-le aux utilisateurs
   - Les utilisateurs doivent l'installer dans le magasin "Autorités de certification racines de confiance"

2. **Installation manuelle du certificat**
   ```batch
   certutil -addstore Root MyCertificate.cer
   ```

3. **Installation via un script**
   ```powershell
   # Importer un certificat dans le magasin de confiance
   Import-Certificate -FilePath "MyCertificate.cer" -CertStoreLocation "Cert:\LocalMachine\Root"
   ```

### Configuration de la signature dans Visual Studio

Pour signer votre application ClickOnce :

1. **Ouvrez les propriétés du projet**
2. **Accédez à l'onglet "Signature"**
3. **Cochez "Signer le manifeste ClickOnce"**
4. **Sélectionnez un certificat**
   - Choisissez un certificat existant
   - Ou cliquez sur "Sélectionner dans le magasin" pour utiliser un certificat installé
   - Ou cliquez sur "Créer un certificat de test" pour en générer un nouveau
5. **Optionnel : Horodatage**
   - Cochez "Horodater le manifeste"
   - Entrez une URL de serveur d'horodatage (par exemple : http://timestamp.digicert.com)

### Signature via la ligne de commande

Vous pouvez également signer une application ClickOnce via la ligne de commande :

```batch
mage -Sign MonApplication.application -CertFile MyCertificate.pfx -Password MonMotDePasse
```

Ou avec MSBuild :

```batch
msbuild MonApplication.csproj /target:publish /property:Configuration=Release /property:SignManifests=true /property:ManifestCertificateThumbprint=ABCDEF1234567890ABCDEF1234567890ABCDEF12
```

### Vérification de la signature

Pour vérifier qu'une application ClickOnce est correctement signée :

1. **Utilisez l'outil Mage.exe**
   ```batch
   mage -Verify MonApplication.application
   ```

2. **Vérifiez visuellement**
   - Ouvrez le fichier .application ou .manifest dans un éditeur de texte
   - Cherchez la section `<Signature>` qui contient la signature numérique

## 14.5.5. Déploiement hors ligne

ClickOnce prend en charge le déploiement d'applications qui peuvent fonctionner hors ligne.

### Modes de disponibilité

ClickOnce offre trois modes de disponibilité :

1. **Disponible en ligne uniquement**
   - L'application nécessite une connexion Internet pour s'exécuter
   - Aucun raccourci n'est créé dans le menu Démarrer ou sur le bureau
   - L'application est exécutée directement depuis l'URL d'installation

2. **Disponible hors ligne également**
   - L'application est installée sur l'ordinateur de l'utilisateur
   - Des raccourcis sont créés dans le menu Démarrer et/ou sur le bureau
   - L'application peut s'exécuter sans connexion Internet
   - Les mises à jour sont vérifiées lorsqu'une connexion est disponible

3. **Disponible hors ligne uniquement**
   - L'application est installée localement comme une application traditionnelle
   - Aucune vérification de mise à jour n'est effectuée

### Configuration du mode hors ligne

Pour configurer le mode de disponibilité :

1. **Ouvrez les propriétés du projet**
2. **Accédez à l'onglet "Publier"**
3. **Cliquez sur "Options"**
4. **Sélectionnez l'option de disponibilité appropriée**

### Déploiement sur un CD/DVD ou une clé USB

Pour déployer une application ClickOnce sur un support amovible :

1. **Configurez la publication**
   - Sélectionnez "À partir d'un CD/DVD" comme méthode d'installation
   - Ou configurez un dossier local comme cible de publication

2. **Publiez l'application**
   - Les fichiers générés peuvent être copiés sur un CD/DVD ou une clé USB

3. **Création d'un autorun (facultatif)**
   - Créez un fichier autorun.inf à la racine du support
   ```
   [autorun]
   open=setup.exe
   icon=setup.exe,0
   label=Installation de Mon Application
   ```

### Gestion des fichiers de données en mode hors ligne

Les applications ClickOnce hors ligne peuvent avoir besoin de stocker des données :

1. **Dossier d'application local**
   - Dossier où l'application est installée
   - Lecture seule après l'installation
   - Accessible via `Application.StartupPath`

2. **Dossier de données d'application**
   - Dossier dédié aux données de l'application
   - Accessible en lecture/écriture
   - Accessible via `ApplicationDeployment.CurrentDeployment.DataDirectory`

```csharp
// Accès au dossier de données
string cheminDonnees = ApplicationDeployment.IsNetworkDeployed
    ? ApplicationDeployment.CurrentDeployment.DataDirectory
    : Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "MonApplication");

// Création du dossier si nécessaire
if (!Directory.Exists(cheminDonnees))
{
    Directory.CreateDirectory(cheminDonnees);
}

// Utilisation d'un fichier dans ce dossier
string cheminFichier = Path.Combine(cheminDonnees, "parametres.xml");
```

### Synchronisation des données

Pour les applications qui fonctionnent à la fois en ligne et hors ligne, vous pourriez avoir besoin de synchroniser des données :

1. **Détection de la connectivité**
   ```csharp
   bool estConnecte = System.Net.NetworkInformation.NetworkInterface.GetIsNetworkAvailable();
   ```

2. **Synchronisation des données**
   ```csharp
   private async Task SynchroniserDonnees()
   {
       if (System.Net.NetworkInformation.NetworkInterface.GetIsNetworkAvailable())
       {
           try
           {
               // Charger les données locales
               var donneesLocales = ChargerDonneesLocales();

               // Synchroniser avec le serveur
               using (var client = new HttpClient())
               {
                   var response = await client.PostAsJsonAsync("https://monapi.com/sync", donneesLocales);

                   if (response.IsSuccessStatusCode)
                   {
                       var donneesServeur = await response.Content.ReadFromJsonAsync<MesDonnees>();

                       // Mettre à jour les données locales
                       SauvegarderDonneesLocales(donneesServeur);
                   }
               }
           }
           catch (Exception ex)
           {
               // Gérer les erreurs de synchronisation
               LoggerErreur("Erreur de synchronisation : " + ex.Message);
           }
       }
   }
   ```

### Gestion des connexions intermittentes

Pour les applications qui peuvent passer du mode en ligne au mode hors ligne :

```csharp
public class GestionnaireConnectivite
{
    private bool _dernierEtatConnexion = false;

    public event EventHandler<bool> ChangementEtatConnexion;

    public void DemarrerSurveillance()
    {
        // État initial
        _dernierEtatConnexion = EstConnecte();

        // Configurer un timer pour vérifier périodiquement
        var timer = new System.Threading.Timer(VerifierConnexion, null, 0, 5000);
    }

    private void VerifierConnexion(object state)
    {
        bool etatActuel = EstConnecte();

        if (etatActuel != _dernierEtatConnexion)
        {
            _dernierEtatConnexion = etatActuel;
            ChangementEtatConnexion?.Invoke(this, etatActuel);
        }
    }

    private bool EstConnecte()
    {
        try
        {
            return System.Net.NetworkInformation.NetworkInterface.GetIsNetworkAvailable();
        }
        catch
        {
            return false;
        }
    }
}
```

Utilisation :

```csharp
private GestionnaireConnectivite _gestionnaire = new GestionnaireConnectivite();

private void FormMain_Load(object sender, EventArgs e)
{
    _gestionnaire.ChangementEtatConnexion += Gestionnaire_ChangementEtatConnexion;
    _gestionnaire.DemarrerSurveillance();
}

private void Gestionnaire_ChangementEtatConnexion(object sender, bool estConnecte)
{
    if (estConnecte)
    {
        // Passer en mode en ligne
        statusConnexion.Text = "Connecté";
        btnSynchroniser.Enabled = true;

        // Vérifier les mises à jour
        VerifierMisesAJour();
    }
    else
    {
        // Passer en mode hors ligne
        statusConnexion.Text = "Hors ligne";
        btnSynchroniser.Enabled = false;
    }
}
```

## Résumé

ClickOnce est une technologie de déploiement puissante et flexible qui permet de publier facilement des applications Windows avec des fonctionnalités de mise à jour automatique. Ses principaux avantages sont :

- Installation sans droits administrateur
- Mises à jour automatiques
- Fonctionnement en ligne et hors ligne
- Configuration et personnalisation simples
- Sécurité via la signature numérique

ClickOnce est particulièrement adapté aux applications d'entreprise, aux applications qui nécessitent des mises à jour fréquentes, et aux applications destinées à des utilisateurs non techniques.

## Exercices pratiques

1. Créez une application Windows Forms simple et publiez-la avec ClickOnce sur un dossier local.

2. Configurez les mises à jour automatiques pour votre application ClickOnce.

3. Personnalisez la page d'installation ClickOnce avec le logo de votre entreprise et des instructions personnalisées.

4. Créez un certificat auto-signé et utilisez-le pour signer votre application ClickOnce.

5. Configurez votre application ClickOnce pour fonctionner hors ligne et gérer la synchronisation des données lorsque la connexion est rétablie.

⏭️
