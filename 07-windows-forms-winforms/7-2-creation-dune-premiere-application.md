# 7.2. Création d'une première application

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Cette section explique comment créer et comprendre une application Windows Forms, en abordant la structure du projet, les fichiers générés et le développement d'une application simple.

## 7.2.1. Structure d'un projet WinForms

Un projet Windows Forms est organisé selon une structure standard qui varie légèrement entre .NET Framework et .NET moderne.

### Structure de base (.NET Framework 4.7.2)

Un projet Windows Forms typique sous .NET Framework 4.7.2 présente la structure suivante :

```
MonProjetWinForms/
│
├── Properties/
│   ├── AssemblyInfo.cs            // Informations sur l'assembly
│   ├── Resources.resx             // Ressources de l'application
│   └── Settings.settings          // Paramètres de l'application
│
├── References/                    // Références aux assemblies .NET
│
├── App.config                     // Fichier de configuration
├── Form1.cs                       // Code du formulaire principal
├── Form1.Designer.cs              // Code généré pour le formulaire
├── Form1.resx                     // Ressources du formulaire
├── Program.cs                     // Point d'entrée de l'application
└── MonProjetWinForms.csproj       // Fichier de projet
```


**Exemple de fichier de projet (.csproj) sous .NET Framework 4.7.2 :**

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" Condition="Exists('$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props')" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{A1B2C3D4-E5F6-G7H8-I9J0-K1L2M3N4O5P6}</ProjectGuid>
    <OutputType>WinExe</OutputType>
    <RootNamespace>MonProjetWinForms</RootNamespace>
    <AssemblyName>MonProjetWinForms</AssemblyName>
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
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
    <PlatformTarget>AnyCPU</PlatformTarget>
    <DebugType>pdbonly</DebugType>
    <Optimize>true</Optimize>
    <OutputPath>bin\Release\</OutputPath>
    <DefineConstants>TRACE</DefineConstants>
    <ErrorReport>prompt</ErrorReport>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="System.Data.DataSetExtensions" />
    <Reference Include="Microsoft.CSharp" />
    <Reference Include="System.Data" />
    <Reference Include="System.Deployment" />
    <Reference Include="System.Drawing" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Windows.Forms" />
    <Reference Include="System.Xml" />
  </ItemGroup>
  <ItemGroup>
    <Compile Include="Form1.cs">
      <SubType>Form</SubType>
    </Compile>
    <Compile Include="Form1.Designer.cs">
      <DependentUpon>Form1.cs</DependentUpon>
    </Compile>
    <Compile Include="Program.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
    <EmbeddedResource Include="Form1.resx">
      <DependentUpon>Form1.cs</DependentUpon>
    </EmbeddedResource>
    <EmbeddedResource Include="Properties\Resources.resx">
      <Generator>ResXFileCodeGenerator</Generator>
      <LastGenOutput>Resources.Designer.cs</LastGenOutput>
      <SubType>Designer</SubType>
    </EmbeddedResource>
    <Compile Include="Properties\Resources.Designer.cs">
      <AutoGen>True</AutoGen>
      <DependentUpon>Resources.resx</DependentUpon>
    </Compile>
    <None Include="Properties\Settings.settings">
      <Generator>SettingsSingleFileGenerator</Generator>
      <LastGenOutput>Settings.Designer.cs</LastGenOutput>
    </None>
    <Compile Include="Properties\Settings.Designer.cs">
      <AutoGen>True</AutoGen>
      <DependentUpon>Settings.settings</DependentUpon>
      <DesignTimeSharedInput>True</DesignTimeSharedInput>
    </Compile>
  </ItemGroup>
  <ItemGroup>
    <None Include="App.config" />
  </ItemGroup>
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
</Project>
```


### Structure de base (.NET 8)

Un projet Windows Forms sous .NET 8 présente une structure plus simple et plus moderne :

```
MonProjetWinForms/
│
├── Properties/
│   └── launchSettings.json         // Paramètres de lancement
│
├── Resources/                      // Dossier de ressources (optionnel)
│
├── Form1.cs                        // Code du formulaire principal
├── Form1.Designer.cs               // Code généré pour le formulaire
├── Form1.resx                      // Ressources du formulaire
├── Program.cs                      // Point d'entrée de l'application
├── MonProjetWinForms.csproj        // Fichier de projet (format SDK)
└── usings.cs                       // Importations globales (optionnel)
```


**Exemple de fichier de projet (.csproj) sous .NET 8 :**

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWindowsForms>true</UseWindowsForms>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```


### Organisation des dossiers recommandée

Pour une application plus complexe, une organisation structurée est recommandée :

```
MonApplication/
│
├── Controls/                       // Contrôles personnalisés
│   ├── CustomControl1.cs
│   └── CustomControl2.cs
│
├── Forms/                          // Formulaires de l'application
│   ├── MainForm.cs
│   ├── SettingsForm.cs
│   └── AboutForm.cs
│
├── Models/                         // Classes de modèle de données
│   ├── Customer.cs
│   └── Product.cs
│
├── Services/                       // Services et logique métier
│   ├── DataService.cs
│   └── PrintService.cs
│
├── Utils/                          // Classes utilitaires
│   ├── Logger.cs
│   └── ConfigManager.cs
│
├── Resources/                      // Ressources (images, icônes, etc.)
│   ├── Images/
│   └── Localization/
│
├── Properties/                     // Propriétés du projet
│
├── Program.cs                      // Point d'entrée
└── MonApplication.csproj           // Fichier de projet
```


### Références et packages NuGet

Les références et packages NuGet sont gérés différemment entre .NET Framework et .NET 8 :

**.NET Framework 4.7.2 (packages.config) :**
```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Newtonsoft.Json" version="13.0.3" targetFramework="net472" />
  <package id="System.Data.SQLite" version="1.0.118.0" targetFramework="net472" />
</packages>
```


**.NET 8 (PackageReference dans .csproj) :**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <!-- Propriétés du projet -->

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
    <PackageReference Include="System.Data.SQLite" Version="1.0.118" />
  </ItemGroup>
</Project>
```


## 7.2.2. Comprendre les fichiers générés

Lorsque vous créez un projet Windows Forms, plusieurs fichiers sont générés automatiquement. Voici une explication de chacun d'eux.

### Program.cs

Le fichier `Program.cs` contient le point d'entrée de l'application.

**.NET Framework 4.7.2 :**
```textmate
using System;
using System.Windows.Forms;

namespace MonProjetWinForms
{
    static class Program
    {
        /// <summary>
        /// Point d'entrée principal de l'application.
        /// </summary>
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new Form1());
        }
    }
}
```


**.NET 8 :**
```textmate
namespace MonProjetWinForms
{
    internal static class Program
    {
        /// <summary>
        ///  Point d'entrée principal de l'application.
        /// </summary>
        [STAThread]
        static void Main()
        {
            // Pour personnaliser la configuration de l'application, comme
            // définir des thèmes haute résolution ou des styles visuels
            ApplicationConfiguration.Initialize();
            Application.Run(new Form1());
        }
    }
}
```


### Form1.cs

Le fichier `Form1.cs` contient la définition de la classe du formulaire principal.

```textmate
using System;
using System.Windows.Forms;

namespace MonProjetWinForms
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        // Gestionnaires d'événements et autres méthodes personnalisées
    }
}
```


### Form1.Designer.cs

Le fichier `Form1.Designer.cs` est généré automatiquement par le concepteur visuel de Windows Forms et contient le code d'initialisation des contrôles.

```textmate
namespace MonProjetWinForms
{
    partial class Form1
    {
        /// <summary>
        /// Variable nécessaire au concepteur.
        /// </summary>
        private System.ComponentModel.IContainer components = null;

        /// <summary>
        /// Nettoyage des ressources utilisées.
        /// </summary>
        /// <param name="disposing">true si les ressources managées doivent être supprimées ; sinon, false.</param>
        protected override void Dispose(bool disposing)
        {
            if (disposing && (components != null))
            {
                components.Dispose();
            }
            base.Dispose(disposing);
        }

        #region Code généré par le Concepteur Windows Form

        /// <summary>
        /// Méthode requise pour la prise en charge du concepteur - ne modifiez pas
        /// le contenu de cette méthode avec l'éditeur de code.
        /// </summary>
        private void InitializeComponent()
        {
            this.button1 = new System.Windows.Forms.Button();
            this.textBox1 = new System.Windows.Forms.TextBox();
            this.SuspendLayout();
            //
            // button1
            //
            this.button1.Location = new System.Drawing.Point(91, 66);
            this.button1.Name = "button1";
            this.button1.Size = new System.Drawing.Size(75, 23);
            this.button1.TabIndex = 0;
            this.button1.Text = "Cliquez-moi";
            this.button1.UseVisualStyleBackColor = true;
            //
            // textBox1
            //
            this.textBox1.Location = new System.Drawing.Point(66, 28);
            this.textBox1.Name = "textBox1";
            this.textBox1.Size = new System.Drawing.Size(100, 20);
            this.textBox1.TabIndex = 1;
            //
            // Form1
            //
            this.AutoScaleDimensions = new System.Drawing.SizeF(6F, 13F);
            this.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font;
            this.ClientSize = new System.Drawing.Size(284, 111);
            this.Controls.Add(this.textBox1);
            this.Controls.Add(this.button1);
            this.Name = "Form1";
            this.Text = "Ma première application";
            this.ResumeLayout(false);
            this.PerformLayout();
        }

        #endregion

        private System.Windows.Forms.Button button1;
        private System.Windows.Forms.TextBox textBox1;
    }
}
```


### Form1.resx

Le fichier `Form1.resx` est un fichier de ressources XML qui stocke les informations de ressources pour le formulaire, comme les chaînes, les images, et les propriétés de mise en page.

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <!--
    Informations de version interne et autres métadonnées
  -->
  <assembly alias="System.Drawing" name="System.Drawing, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
  <data name="$this.Icon" type="System.Drawing.Icon, System.Drawing" mimetype="application/x-microsoft.net.object.bytearray.base64">
    <!-- Données d'icône en Base64 -->
  </data>
</root>
```


### AssemblyInfo.cs (.NET Framework 4.7.2)

Sous .NET Framework, le fichier `AssemblyInfo.cs` contient les attributs de l'assembly.

```textmate
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

// Les informations générales relatives à un assembly dépendent de
// l'ensemble d'attributs suivant. Changez les valeurs de ces attributs pour modifier les informations
// associées à un assembly.
[assembly: AssemblyTitle("MonProjetWinForms")]
[assembly: AssemblyDescription("")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("MonProjetWinForms")]
[assembly: AssemblyCopyright("Copyright © 2023")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]

// L'affectation de la valeur false à ComVisible rend les types invisibles dans cet assembly
// aux composants COM. Si vous devez accéder à un type dans cet assembly à partir de
// COM, affectez la valeur true à l'attribut ComVisible sur ce type.
[assembly: ComVisible(false)]

// Le GUID suivant est pour l'ID de la typelib si ce projet est exposé à COM
[assembly: Guid("a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6")]

// Les informations de version pour un assembly se composent des quatre valeurs suivantes :
//
//      Version principale
//      Version secondaire
//      Numéro de build
//      Révision
//
// Vous pouvez spécifier toutes les valeurs ou indiquer les numéros de build et de révision par défaut
// en utilisant '*', comme indiqué ci-dessous :
// [assembly: AssemblyVersion("1.0.*")]
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
```


### App.config (.NET Framework 4.7.2)

Sous .NET Framework, le fichier `App.config` contient les configurations de l'application.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
    </startup>
    <appSettings>
        <add key="ConnectionString" value="Data Source=localhost;Initial Catalog=MyDb;Integrated Security=True" />
    </appSettings>
</configuration>
```


### appsettings.json (.NET 8)

Sous .NET 8, `appsettings.json` est utilisé pour la configuration.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=localhost;Initial Catalog=MyDb;Integrated Security=True"
  },
  "AppSettings": {
    "Theme": "Light",
    "Language": "fr-FR"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```


### launchSettings.json (.NET 8)

Sous .NET 8, `Properties/launchSettings.json` contient les paramètres de lancement.

```json
{
  "profiles": {
    "MonProjetWinForms": {
      "commandName": "Project",
      "environmentVariables": {
        "DOTNET_ENVIRONMENT": "Development"
      }
    }
  }
}
```


## 7.2.3. Application simple étape par étape

Créons une application Windows Forms simple qui affiche un message personnalisé lorsqu'un utilisateur clique sur un bouton.

### Étape 1 : Création du projet

**.NET Framework 4.7.2 (Visual Studio) :**

1. Ouvrez Visual Studio
2. Cliquez sur `Fichier` > `Nouveau` > `Projet`
3. Sélectionnez `Application Windows Forms (.NET Framework)`
4. Donnez un nom à votre projet, par exemple `PremièreAppWinForms`
5. Sélectionnez `.NET Framework 4.7.2` dans la liste déroulante
6. Cliquez sur `Créer`

**.NET 8 (Visual Studio) :**

1. Ouvrez Visual Studio
2. Cliquez sur `Fichier` > `Nouveau` > `Projet`
3. Sélectionnez `Application Windows Forms`
4. Donnez un nom à votre projet, par exemple `PremièreAppWinForms`
5. Sélectionnez `.NET 8.0` dans la liste déroulante
6. Cliquez sur `Créer`

**.NET 8 (ligne de commande) :**

```shell script
dotnet new winforms -n PremièreAppWinForms -f net8.0
cd PremièreAppWinForms
```


### Étape 2 : Conception de l'interface utilisateur

1. Double-cliquez sur `Form1.cs` dans l'Explorateur de solutions pour ouvrir le concepteur de formulaires
2. Modifiez les propriétés du formulaire :
   - Définissez `Text` à "Ma première application WinForms"
   - Définissez `Size` à 400, 300 (largeur, hauteur)

3. Ajoutez des contrôles en les faisant glisser depuis la Boîte à outils :

   - **Label :**
     - Positionnez-le en haut du formulaire
     - Définissez `Name` à "labelNom"
     - Définissez `Text` à "Votre nom :"
     - Définissez `Location` à 20, 20
     - Définissez `AutoSize` à true

   - **TextBox :**
     - Positionnez-le à droite du label
     - Définissez `Name` à "textBoxNom"
     - Effacez `Text` (laissez-le vide)
     - Définissez `Location` à 100, 17
     - Définissez `Size` à 200, 23

   - **Button :**
     - Positionnez-le sous le TextBox
     - Définissez `Name` à "buttonSaluer"
     - Définissez `Text` à "Saluer"
     - Définissez `Location` à 100, 50
     - Définissez `Size` à 100, 30

   - **Label :** (pour le résultat)
     - Positionnez-le sous le bouton
     - Définissez `Name` à "labelResultat"
     - Effacez `Text` (laissez-le vide)
     - Définissez `Location` à 100, 100
     - Définissez `AutoSize` à true

### Étape 3 : Ajout du code

Double-cliquez sur le bouton "Saluer" pour créer un gestionnaire d'événements et ajoutez le code suivant :

```textmate
private void buttonSaluer_Click(object sender, EventArgs e)
{
    string nom = textBoxNom.Text.Trim();

    if (string.IsNullOrEmpty(nom))
    {
        // Afficher un message d'erreur
        labelResultat.Text = "Veuillez entrer votre nom !";
        labelResultat.ForeColor = Color.Red;
    }
    else
    {
        // Afficher le message de salutation
        labelResultat.Text = $"Bonjour {nom}, bienvenue dans votre première application WinForms !";
        labelResultat.ForeColor = Color.Green;
    }
}
```


### Étape 4 : Amélioration de l'expérience utilisateur

Ajoutons quelques améliorations à notre application :

1. Permettons à l'utilisateur d'appuyer sur Entrée dans le TextBox pour déclencher l'action du bouton :

```textmate
// Dans le constructeur Form1()
public Form1()
{
    InitializeComponent();

    // Configurez la touche Entrée pour qu'elle active le bouton
    this.AcceptButton = buttonSaluer;

    // Ajoutez un gestionnaire pour l'événement Load du formulaire
    this.Load += Form1_Load;
}

private void Form1_Load(object sender, EventArgs e)
{
    // Placez le focus sur le TextBox au démarrage
    textBoxNom.Focus();
}
```


2. Ajoutons un menu à notre application :

Dans le concepteur de formulaires, faites glisser un `MenuStrip` depuis la boîte à outils vers le formulaire.

- Créez un menu "Fichier" avec les éléments suivants :
  - "Nouveau" (`menuNouveau`)
  - "Quitter" (`menuQuitter`)
- Créez un menu "Aide" avec l'élément :
  - "À propos" (`menuAPropos`)

Ajoutez le code pour ces éléments de menu :

```textmate
private void menuNouveau_Click(object sender, EventArgs e)
{
    // Effacer les champs
    textBoxNom.Clear();
    labelResultat.Text = "";
    textBoxNom.Focus();
}

private void menuQuitter_Click(object sender, EventArgs e)
{
    // Fermer l'application
    this.Close();
}

private void menuAPropos_Click(object sender, EventArgs e)
{
    // Afficher un message d'information
    MessageBox.Show(
        "Ma première application Windows Forms\nDéveloppée avec " +
        #if NETCOREAPP || NET5_0_OR_GREATER
        ".NET 8",
        #else
        ".NET Framework 4.7.2",
        #endif
        "À propos",
        MessageBoxButtons.OK,
        MessageBoxIcon.Information);
}
```


### Étape 5 : Gestion des erreurs et validations

Ajoutons une validation plus robuste :

```textmate
private void textBoxNom_TextChanged(object sender, EventArgs e)
{
    // Vérifier en temps réel et donner un retour visuel
    if (string.IsNullOrWhiteSpace(textBoxNom.Text))
    {
        textBoxNom.BackColor = Color.MistyRose;
    }
    else
    {
        textBoxNom.BackColor = SystemColors.Window;
    }
}
```


### Étape 6 : Persistance des données

Ajoutons une fonctionnalité pour mémoriser le dernier nom saisi :

**.NET Framework 4.7.2 :**

```textmate
// Dans le constructeur
public Form1()
{
    InitializeComponent();
    this.AcceptButton = buttonSaluer;
    this.Load += Form1_Load;
    this.FormClosing += Form1_FormClosing;
}

private void Form1_Load(object sender, EventArgs e)
{
    // Charger le dernier nom depuis les paramètres
    textBoxNom.Text = Properties.Settings.Default.DernierNom;
    textBoxNom.Focus();
}

private void Form1_FormClosing(object sender, FormClosingEventArgs e)
{
    // Enregistrer le dernier nom dans les paramètres
    Properties.Settings.Default.DernierNom = textBoxNom.Text;
    Properties.Settings.Default.Save();
}
```


Il faut aussi créer un paramètre nommé "DernierNom" dans le fichier `Settings.settings` :
1. Double-cliquez sur `Properties/Settings.settings`
2. Ajoutez un paramètre :
   - Nom : DernierNom
   - Type : String
   - Portée : User
   - Valeur : (laissez vide)

**.NET 8 :**

```textmate
// Définir une classe pour les paramètres
public class AppSettings
{
    public string DernierNom { get; set; } = string.Empty;
}

// Dans la classe Form1
private AppSettings _settings = new AppSettings();
private readonly string _settingsPath = Path.Combine(
    Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
    "PremièreAppWinForms",
    "settings.json");

// Dans le constructeur
public Form1()
{
    InitializeComponent();
    this.AcceptButton = buttonSaluer;
    this.Load += Form1_Load;
    this.FormClosing += Form1_FormClosing;
}

private void Form1_Load(object sender, EventArgs e)
{
    // Charger les paramètres
    LoadSettings();
    textBoxNom.Text = _settings.DernierNom;
    textBoxNom.Focus();
}

private void Form1_FormClosing(object sender, FormClosingEventArgs e)
{
    // Enregistrer les paramètres
    _settings.DernierNom = textBoxNom.Text;
    SaveSettings();
}

private void LoadSettings()
{
    try
    {
        if (File.Exists(_settingsPath))
        {
            string json = File.ReadAllText(_settingsPath);
            _settings = JsonSerializer.Deserialize<AppSettings>(json) ?? new AppSettings();
        }
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Erreur lors du chargement des paramètres : {ex.Message}", "Erreur",
            MessageBoxButtons.OK, MessageBoxIcon.Error);
        _settings = new AppSettings();
    }
}

private void SaveSettings()
{
    try
    {
        // Créer le répertoire s'il n'existe pas
        Directory.CreateDirectory(Path.GetDirectoryName(_settingsPath)!);

        // Sérialiser et enregistrer
        string json = JsonSerializer.Serialize(_settings, new JsonSerializerOptions
        {
            WriteIndented = true
        });
        File.WriteAllText(_settingsPath, json);
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Erreur lors de l'enregistrement des paramètres : {ex.Message}", "Erreur",
            MessageBoxButtons.OK, MessageBoxIcon.Error);
    }
}
```


N'oubliez pas d'ajouter l'instruction using :
```textmate
using System.Text.Json;
```


### Étape 7 : Ajouter un contenu visuel avancé

Ajoutons un peu plus d'interactivité à notre application :

1. Ajoutez un `PictureBox` depuis la boîte à outils
   - Définissez `Name` à "pictureBoxAvatar"
   - Définissez `SizeMode` à "Zoom"
   - Définissez `Size` à 100, 100
   - Définissez `Location` à 20, 80
   - Définissez `BorderStyle` à "FixedSingle"

2. Ajoutez un `Button` pour choisir une image
   - Définissez `Name` à "buttonChoisirImage"
   - Définissez `Text` à "Choisir une image"
   - Définissez `Location` à 20, 190
   - Définissez `Size` à 100, 30

3. Ajoutez le code suivant pour le bouton :

```textmate
private void buttonChoisirImage_Click(object sender, EventArgs e)
{
    using (OpenFileDialog openFileDialog = new OpenFileDialog())
    {
        openFileDialog.Title = "Sélectionner une image";
        openFileDialog.Filter = "Images|*.jpg;*.jpeg;*.png;*.gif;*.bmp|Tous les fichiers|*.*";

        if (openFileDialog.ShowDialog() == DialogResult.OK)
        {
            try
            {
                pictureBoxAvatar.Image = Image.FromFile(openFileDialog.FileName);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Impossible de charger l'image : {ex.Message}", "Erreur",
                    MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}
```


### Étape 8 : Amélioration multi-formulaires

Créons un second formulaire pour les paramètres de l'application :

1. Ajoutez un nouveau formulaire au projet :
   - Clic droit sur le projet dans l'Explorateur de solutions
   - Sélectionnez `Ajouter` > `Nouveau élément`
   - Sélectionnez `Formulaire Windows Forms`
   - Nommez-le `ParametresForm.cs`
   - Cliquez sur `Ajouter`

2. Concevez le formulaire des paramètres :
   - Définissez `Text` à "Paramètres"
   - Définissez `StartPosition` à "CenterParent"
   - Définissez `FormBorderStyle` à "FixedDialog"
   - Définissez `MaximizeBox` à false
   - Définissez `MinimizeBox` à false
   - Définissez `Size` à 400, 300

3. Ajoutez les contrôles suivants :
   - **GroupBox :**
     - Définissez `Name` à "groupBoxApparence"
     - Définissez `Text` à "Apparence"
     - Définissez `Location` à 20, 20
     - Définissez `Size` à 350, 150

   - **RadioButton :** (à l'intérieur du GroupBox)
     - Définissez `Name` à "radioButtonThemeClair"
     - Définissez `Text` à "Thème clair"
     - Définissez `Location` à 20, 30
     - Définissez `Checked` à true

   - **RadioButton :** (à l'intérieur du GroupBox)
     - Définissez `Name` à "radioButtonThemeSombre"
     - Définissez `Text` à "Thème sombre"
     - Définissez `Location` à 20, 60

   - **CheckBox :** (à l'intérieur du GroupBox)
     - Définissez `Name` à "checkBoxMémoriserNom"
     - Définissez `Text` à "Mémoriser le dernier nom utilisé"
     - Définissez `Location` à 20, 100

   - **Button :**
     - Définissez `Name` à "buttonOK"
     - Définissez `Text` à "OK"
     - Définissez `Location` à 265, 220
     - Définissez `Size` à 80, 30
     - Définissez `DialogResult` à "OK"

   - **Button :**
     - Définissez `Name` à "buttonAnnuler"
     - Définissez `Text` à "Annuler"
     - Définissez `Location` à 175, 220
     - Définissez `Size` à 80, 30
     - Définissez `DialogResult` à "Cancel"

4. Ajoutez des propriétés et des méthodes au formulaire de paramètres :

```textmate
public partial class ParametresForm : Form
{
    // Propriétés pour stocker les paramètres
    public bool ThemeSombre { get; set; }
    public bool MémoriserNom { get; set; }

    public ParametresForm()
    {
        InitializeComponent();

        // Gestionnaire d'événement pour charger les paramètres actuels
        this.Load += ParametresForm_Load;
    }

    private void ParametresForm_Load(object sender, EventArgs e)
    {
        // Initialiser les contrôles avec les valeurs actuelles
        radioButtonThemeSombre.Checked = ThemeSombre;
        radioButtonThemeClair.Checked = !ThemeSombre;
        checkBoxMémoriserNom.Checked = MémoriserNom;
    }

    private void buttonOK_Click(object sender, EventArgs e)
    {
        // Mettre à jour les propriétés avec les valeurs des contrôles
        ThemeSombre = radioButtonThemeSombre.Checked;
        MémoriserNom = checkBoxMémoriserNom.Checked;

        // Le DialogResult est déjà défini à OK dans les propriétés du bouton
    }
}
```


5. Modifiez la classe AppSettings (.NET 8) pour inclure les nouveaux paramètres :

```textmate
public class AppSettings
{
    public string DernierNom { get; set; } = string.Empty;
    public bool ThemeSombre { get; set; } = false;
    public bool MémoriserNom { get; set; } = true;
}
```


6. Ajoutez un élément de menu "Paramètres" dans le menu "Fichier" de Form1 et implémentez le gestionnaire d'événements :

```textmate
private void menuParametres_Click(object sender, EventArgs e)
{
    // Créer et configurer le formulaire de paramètres
    using (var parametresForm = new ParametresForm())
    {
        // Initialiser avec les paramètres actuels
        parametresForm.ThemeSombre = _settings.ThemeSombre;
        parametresForm.MémoriserNom = _settings.MémoriserNom;

        // Afficher le formulaire comme boîte de dialogue
        if (parametresForm.ShowDialog() == DialogResult.OK)
        {
            // Récupérer les paramètres mis à jour
            _settings.ThemeSombre = parametresForm.ThemeSombre;
            _settings.MémoriserNom = parametresForm.MémoriserNom;

            // Appliquer le thème
            AppliquerTheme(_settings.ThemeSombre);

            // Gérer l'option "Mémoriser le nom"
            if (!_settings.MémoriserNom)
            {
                _settings.DernierNom = string.Empty;
            }

            // Enregistrer les paramètres
            SaveSettings();
        }
    }
}

private void AppliquerTheme(bool themeSombre)
{
    if (themeSombre)
    {
        // Appliquer le thème sombre
        this.BackColor = Color.FromArgb(50, 50, 50);
        this.ForeColor = Color.White;

        // Mettre à jour les contrôles
        foreach (Control control in this.Controls)
        {
            if (control is TextBox || control is ComboBox)
            {
                control.BackColor = Color.FromArgb(70, 70, 70);
                control.ForeColor = Color.White;
            }
            else if (control is Button)
            {
                control.BackColor = Color.FromArgb(60, 60, 60);
                control.ForeColor = Color.White;
            }
        }
    }
    else
    {
        // Appliquer le thème clair (par défaut)
        this.BackColor = SystemColors.Control;
        this.ForeColor = SystemColors.ControlText;

        // Réinitialiser les contrôles
        foreach (Control control in this.Controls)
        {
            if (control is TextBox || control is ComboBox)
            {
                control.BackColor = SystemColors.Window;
                control.ForeColor = SystemColors.WindowText;
            }
            else if (control is Button)
            {
                control.BackColor = SystemColors.Control;
                control.ForeColor = SystemColors.ControlText;
            }
        }
    }
}
```


7. Mettez à jour la méthode `Form1_Load` pour appliquer le thème au chargement :

```textmate
private void Form1_Load(object sender, EventArgs e)
{
    // Charger les paramètres
    LoadSettings();

    // Appliquer le thème
    AppliquerTheme(_settings.ThemeSombre);

    // Charger le dernier nom si l'option est activée
    if (_settings.MémoriserNom)
    {
        textBoxNom.Text = _settings.DernierNom;
    }

    // Mettre le focus sur le TextBox
    textBoxNom.Focus();
}
```


### Étape 9 : Ajout d'une barre d'état

Ajoutons une barre d'état pour afficher des informations supplémentaires :

1. Depuis la boîte à outils, faites glisser un contrôle `StatusStrip` vers le bas du formulaire
   - Définissez `Name` à "statusStrip1"

2. Ajoutez des éléments à la barre d'état :
   - Cliquez sur la flèche déroulante de la StatusStrip et ajoutez :
     - Un `ToolStripStatusLabel` (pour afficher des messages)
       - Définissez `Name` à "statusLabelInfo"
       - Définissez `Text` à "Prêt"
     - Un `ToolStripStatusLabel` (pour afficher l'heure)
       - Définissez `Name` à "statusLabelHeure"
       - Définissez `Alignment` à "ToolStripItemAlignment.Right"

3. Ajoutez un Timer pour mettre à jour l'heure :
   - Faites glisser un `Timer` depuis la boîte à outils (il apparaîtra dans la zone des composants)
   - Définissez `Name` à "timerHeure"
   - Définissez `Interval` à 1000 (1 seconde)
   - Définissez `Enabled` à true

4. Ajoutez le code suivant :

```textmate
private void timerHeure_Tick(object sender, EventArgs e)
{
    // Mettre à jour l'heure dans la barre d'état
    statusLabelHeure.Text = DateTime.Now.ToString("HH:mm:ss");
}

// Mettre à jour la méthode buttonSaluer_Click
private void buttonSaluer_Click(object sender, EventArgs e)
{
    string nom = textBoxNom.Text.Trim();

    if (string.IsNullOrEmpty(nom))
    {
        // Afficher un message d'erreur
        labelResultat.Text = "Veuillez entrer votre nom !";
        labelResultat.ForeColor = Color.Red;
        statusLabelInfo.Text = "Erreur : nom manquant";
    }
    else
    {
        // Afficher le message de salutation
        labelResultat.Text = $"Bonjour {nom}, bienvenue dans votre première application WinForms !";
        labelResultat.ForeColor = Color.Green;
        statusLabelInfo.Text = $"Salutation envoyée à {nom}";
    }
}
```


### Étape 10 : Gestion des erreurs

Ajoutons un gestionnaire d'exceptions global pour notre application :

1. Modifiez le fichier `Program.cs` :

**.NET Framework 4.7.2 :**
```textmate
static class Program
{
    [STAThread]
    static void Main()
    {
        // Ajouter un gestionnaire d'exceptions non gérées
        Application.ThreadException += Application_ThreadException;
        AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;

        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new Form1());
    }

    private static void Application_ThreadException(object sender, ThreadExceptionEventArgs e)
    {
        // Gérer les exceptions de l'interface utilisateur
        AfficherErreur("Une erreur s'est produite", e.Exception);
    }

    private static void CurrentDomain_UnhandledException(object sender, UnhandledExceptionEventArgs e)
    {
        // Gérer les exceptions non gérées
        AfficherErreur("Une erreur critique s'est produite", e.ExceptionObject as Exception);
    }

    private static void AfficherErreur(string message, Exception ex)
    {
        try
        {
            // Enregistrer l'erreur
            string logPath = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
                "PremièreAppWinForms",
                "error.log");

            Directory.CreateDirectory(Path.GetDirectoryName(logPath));

            using (StreamWriter writer = new StreamWriter(logPath, true))
            {
                writer.WriteLine($"[{DateTime.Now}] {message}: {ex.Message}");
                writer.WriteLine(ex.StackTrace);
                writer.WriteLine(new string('-', 50));
            }

            // Afficher l'erreur à l'utilisateur
            MessageBox.Show(
                $"{message}:\n{ex.Message}\n\nL'erreur a été enregistrée dans le journal.",
                "Erreur",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
        catch
        {
            // En cas d'échec de la journalisation, afficher simplement l'erreur
            MessageBox.Show(
                $"{message}:\n{ex.Message}",
                "Erreur critique",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
    }
}
```


**.NET 8 :**
```textmate
static class Program
{
    [STAThread]
    static void Main()
    {
        // Ajouter un gestionnaire d'exceptions non gérées
        Application.ThreadException += Application_ThreadException;
        AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;

        ApplicationConfiguration.Initialize();
        Application.Run(new Form1());
    }

    private static void Application_ThreadException(object sender, ThreadExceptionEventArgs e)
    {
        // Gérer les exceptions de l'interface utilisateur
        AfficherErreur("Une erreur s'est produite", e.Exception);
    }

    private static void CurrentDomain_UnhandledException(object sender, UnhandledExceptionEventArgs e)
    {
        // Gérer les exceptions non gérées
        AfficherErreur("Une erreur critique s'est produite", e.ExceptionObject as Exception);
    }

    private static void AfficherErreur(string message, Exception ex)
    {
        try
        {
            // Enregistrer l'erreur
            string logPath = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
                "PremièreAppWinForms",
                "error.log");

            Directory.CreateDirectory(Path.GetDirectoryName(logPath)!);

            File.AppendAllText(logPath,
                $"[{DateTime.Now}] {message}: {ex?.Message}\n{ex?.StackTrace}\n{new string('-', 50)}\n");

            // Afficher l'erreur à l'utilisateur
            MessageBox.Show(
                $"{message}:\n{ex?.Message}\n\nL'erreur a été enregistrée dans le journal.",
                "Erreur",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
        catch
        {
            // En cas d'échec de la journalisation, afficher simplement l'erreur
            MessageBox.Show(
                $"{message}:\n{ex?.Message}",
                "Erreur critique",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
    }
}
```


### Étape 11 : Implémentation du glisser-déposer

Ajoutons la prise en charge du glisser-déposer pour l'image :

```textmate
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Configuration des événements
        this.AcceptButton = buttonSaluer;
        this.Load += Form1_Load;
        this.FormClosing += Form1_FormClosing;

        // Configuration du glisser-déposer
        pictureBoxAvatar.AllowDrop = true;
        pictureBoxAvatar.DragEnter += PictureBoxAvatar_DragEnter;
        pictureBoxAvatar.DragDrop += PictureBoxAvatar_DragDrop;
    }

    private void PictureBoxAvatar_DragEnter(object sender, DragEventArgs e)
    {
        // Vérifier si les données sont un fichier
        if (e.Data.GetDataPresent(DataFormats.FileDrop))
        {
            string[] files = (string[])e.Data.GetData(DataFormats.FileDrop);

            // Vérifier s'il s'agit d'un fichier image
            if (files.Length == 1 && EstFichierImage(files[0]))
            {
                e.Effect = DragDropEffects.Copy;
                return;
            }
        }

        e.Effect = DragDropEffects.None;
    }

    private void PictureBoxAvatar_DragDrop(object sender, DragEventArgs e)
    {
        string[] files = (string[])e.Data.GetData(DataFormats.FileDrop);

        if (files.Length > 0 && EstFichierImage(files[0]))
        {
            try
            {
                // Charger l'image
                pictureBoxAvatar.Image = Image.FromFile(files[0]);

                // Mettre à jour la barre d'état
                statusLabelInfo.Text = $"Image chargée : {Path.GetFileName(files[0])}";
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Erreur lors du chargement de l'image : {ex.Message}",
                    "Erreur", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }

    private bool EstFichierImage(string filePath)
    {
        // Vérifier l'extension du fichier
        string extension = Path.GetExtension(filePath).ToLower();
        return extension == ".jpg" || extension == ".jpeg" ||
               extension == ".png" || extension == ".gif" ||
               extension == ".bmp";
    }
}
```


### Étape 12 : Finalisation et test

1. Exécutez l'application en appuyant sur F5 ou en cliquant sur `Déboguer` > `Démarrer le débogage`
2. Testez toutes les fonctionnalités que nous avons implémentées
3. Vérifiez que les paramètres sont correctement enregistrés et chargés
4. Testez le formulaire des paramètres et le changement de thème
5. Essayez le glisser-déposer d'images dans la zone prévue
6. Vérifiez que les messages d'erreur s'affichent correctement

### Résultat final

Nous avons maintenant créé une application Windows Forms simple mais complète qui démontre plusieurs concepts importants :

1. Création d'une interface utilisateur avec des contrôles standard
2. Gestion des événements et interactions avec l'utilisateur
3. Validation des entrées utilisateur
4. Persistance des données et paramètres
5. Interface multi-formulaires
6. Gestion des thèmes et de l'apparence
7. Prise en charge du glisser-déposer
8. Gestion des erreurs
9. Informations d'état et retour visuel

Cette application constitue une base solide que vous pouvez étendre avec des fonctionnalités plus avancées à mesure que vous développez vos compétences en développement Windows Forms.

Le code source complet de l'application est maintenant prêt à être utilisé et modifié selon vos besoins.

⏭️ 7.3. [Contrôles de base et leurs propriétés](/07-windows-forms-winforms/7-3-controles-de-base-et-leurs-proprietes.md)
