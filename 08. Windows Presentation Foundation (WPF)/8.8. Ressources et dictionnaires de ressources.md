# 8.8. Ressources et dictionnaires de ressources

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les ressources en WPF vous permettent de définir et réutiliser des objets tels que styles, templates, brushes et autres valeurs à travers votre application. Comprendre comment utiliser les ressources est essentiel pour créer des applications WPF bien structurées et faciles à maintenir.

## 8.8.1. Resources locales

Les ressources locales sont définies et disponibles dans un seul élément et ses enfants. Elles sont utiles lorsque vous souhaitez réutiliser des valeurs uniquement dans une partie spécifique de votre interface.

### Définition de ressources locales

```xml
<StackPanel>
    <!-- Définition des ressources locales -->
    <StackPanel.Resources>
        <SolidColorBrush x:Key="MonBrushBleu" Color="DodgerBlue"/>
        <SolidColorBrush x:Key="MonBrushVert" Color="LimeGreen"/>
        <system:Double x:Key="TaillePoliceStandard">14</system:Double>
    </StackPanel.Resources>

    <!-- Utilisation des ressources locales -->
    <TextBlock Text="Texte en bleu"
               Foreground="{StaticResource MonBrushBleu}"
               FontSize="{StaticResource TaillePoliceStandard}"/>
    <TextBlock Text="Texte en vert"
               Foreground="{StaticResource MonBrushVert}"
               FontSize="{StaticResource TaillePoliceStandard}"/>
</StackPanel>
```


Pour utiliser des types .NET standards comme `Double`, vous devez ajouter le namespace `System` :

```xml
<Window xmlns:system="clr-namespace:System;assembly=mscorlib">
    <!-- votre code ici -->
</Window>
```


### Types de ressources courantes

Voici quelques exemples de types de ressources que vous pouvez définir :

```xml
<StackPanel.Resources>
    <!-- Couleurs et brushes -->
    <Color x:Key="CouleurPrimaire">#1E88E5</Color>
    <SolidColorBrush x:Key="BrushPrimaire" Color="{StaticResource CouleurPrimaire}"/>
    <LinearGradientBrush x:Key="BrushGradient" StartPoint="0,0" EndPoint="1,1">
        <GradientStop Color="Yellow" Offset="0.0"/>
        <GradientStop Color="Orange" Offset="0.5"/>
        <GradientStop Color="Red" Offset="1.0"/>
    </LinearGradientBrush>

    <!-- Valeurs numériques -->
    <system:Double x:Key="EspacementStandard">10</system:Double>
    <system:Int32 x:Key="NombreItems">5</system:Int32>

    <!-- Chaînes de caractères -->
    <system:String x:Key="TitreApplication">Mon Application</system:String>

    <!-- Booléens -->
    <system:Boolean x:Key="EstModifiable">true</system:Boolean>

    <!-- Géométrie pour formes, icônes, etc. -->
    <Geometry x:Key="IconeEtoile">M 0,0 L 10,10 L 0,10 Z</Geometry>
</StackPanel.Resources>
```


### Accès aux ressources

Il y a deux façons principales d'accéder aux ressources en XAML :

1. **StaticResource** : Résout la ressource une seule fois au moment du chargement
2. **DynamicResource** : Résout la ressource à chaque fois qu'elle est utilisée, permettant des modifications en cours d'exécution

```xml
<!-- Utilisation de StaticResource (plus performant) -->
<TextBlock Text="Hello World" Foreground="{StaticResource MonBrushBleu}"/>

<!-- Utilisation de DynamicResource (pour les ressources qui peuvent changer) -->
<TextBlock Text="Hello World" Foreground="{DynamicResource MonBrushBleu}"/>
```


### Accès aux ressources en code

Vous pouvez également accéder aux ressources en code :

```textmate
// Récupérer une ressource à partir d'un élément
SolidColorBrush brush = (SolidColorBrush)myStackPanel.Resources["MonBrushBleu"];

// Modifier une ressource
myStackPanel.Resources["MonBrushBleu"] = new SolidColorBrush(Colors.Navy);

// Ajouter une nouvelle ressource en code
myStackPanel.Resources.Add("NouvelleChaine", "Valeur ajoutée en code");
```


## 8.8.2. Ressources d'application

Les ressources d'application sont définies au niveau de l'application et sont accessibles depuis n'importe quelle partie de votre application. Elles sont idéales pour définir des ressources globales comme les couleurs de votre marque, les styles communs, etc.

### Définition dans App.xaml

```xml
<Application x:Class="MonApplication.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:system="clr-namespace:System;assembly=mscorlib"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <!-- Couleurs de la marque -->
        <Color x:Key="CouleurPrimaire">#2196F3</Color>
        <Color x:Key="CouleurSecondaire">#FFC107</Color>
        <Color x:Key="CouleurAccent">#FF4081</Color>

        <!-- Brushes basés sur les couleurs -->
        <SolidColorBrush x:Key="BrushPrimaire" Color="{StaticResource CouleurPrimaire}"/>
        <SolidColorBrush x:Key="BrushSecondaire" Color="{StaticResource CouleurSecondaire}"/>
        <SolidColorBrush x:Key="BrushAccent" Color="{StaticResource CouleurAccent}"/>

        <!-- Tailles de police -->
        <system:Double x:Key="TaillePoliceSmall">12</system:Double>
        <system:Double x:Key="TaillePoliceNormal">14</system:Double>
        <system:Double x:Key="TaillePoliceLarge">18</system:Double>
        <system:Double x:Key="TaillePoliceHeader">24</system:Double>

        <!-- Espacements -->
        <Thickness x:Key="MargeStandard">10</Thickness>
        <Thickness x:Key="PaddingStandard">5,5,5,5</Thickness>

        <!-- Style de base pour les TextBlock -->
        <Style x:Key="TextBlockBase" TargetType="TextBlock">
            <Setter Property="FontFamily" Value="Segoe UI"/>
            <Setter Property="FontSize" Value="{StaticResource TaillePoliceNormal}"/>
            <Setter Property="Margin" Value="0,0,0,5"/>
        </Style>

        <!-- Style pour les boutons -->
        <Style x:Key="BoutonStandard" TargetType="Button">
            <Setter Property="Background" Value="{StaticResource BrushPrimaire}"/>
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="Padding" Value="15,5"/>
            <Setter Property="FontSize" Value="{StaticResource TaillePoliceNormal}"/>
        </Style>
    </Application.Resources>
</Application>
```


### Utilisation des ressources d'application

Vous pouvez utiliser ces ressources dans n'importe quelle fenêtre ou contrôle de votre application :

```xml
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="450" Width="800">
    <Grid Margin="{StaticResource MargeStandard}">
        <StackPanel>
            <TextBlock Text="Titre de l'application"
                       FontSize="{StaticResource TaillePoliceHeader}"
                       Foreground="{StaticResource BrushPrimaire}"
                       Margin="0,0,0,20"/>

            <TextBlock Text="Utilisation de styles d'application"
                       Style="{StaticResource TextBlockBase}"/>

            <Button Content="Bouton standard"
                    Style="{StaticResource BoutonStandard}"
                    Margin="0,20,0,0"/>
        </StackPanel>
    </Grid>
</Window>
```


### Accès aux ressources d'application en code

```textmate
// Dans n'importe quelle partie de votre application
SolidColorBrush brush = (SolidColorBrush)Application.Current.Resources["BrushPrimaire"];

// Modifier une ressource d'application
Application.Current.Resources["BrushPrimaire"] = new SolidColorBrush(Colors.DarkBlue);
```


## 8.8.3. Ressources externes

Les ressources externes vous permettent de séparer vos définitions de ressources dans des fichiers séparés, ce qui améliore l'organisation et la réutilisabilité.

### Création d'un dictionnaire de ressources

Créez un nouveau fichier XAML dans votre projet (clic droit sur le projet → Ajouter → Nouveau élément → ResourceDictionary) :

```xml
<!-- Couleurs.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- Définition des couleurs -->
    <Color x:Key="CouleurPrimaire">#2196F3</Color>
    <Color x:Key="CouleurSecondaire">#FFC107</Color>
    <Color x:Key="CouleurAccent">#FF4081</Color>

    <SolidColorBrush x:Key="BrushPrimaire" Color="{StaticResource CouleurPrimaire}"/>
    <SolidColorBrush x:Key="BrushSecondaire" Color="{StaticResource CouleurSecondaire}"/>
    <SolidColorBrush x:Key="BrushAccent" Color="{StaticResource CouleurAccent}"/>
</ResourceDictionary>
```


```xml
<!-- Styles.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:system="clr-namespace:System;assembly=mscorlib">
    <!-- Tailles de police -->
    <system:Double x:Key="TaillePoliceSmall">12</system:Double>
    <system:Double x:Key="TaillePoliceNormal">14</system:Double>
    <system:Double x:Key="TaillePoliceLarge">18</system:Double>
    <system:Double x:Key="TaillePoliceHeader">24</system:Double>

    <!-- Style de base pour les TextBlock -->
    <Style x:Key="TextBlockBase" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI"/>
        <Setter Property="FontSize" Value="{StaticResource TaillePoliceNormal}"/>
        <Setter Property="Margin" Value="0,0,0,5"/>
    </Style>

    <!-- Style pour les boutons -->
    <Style x:Key="BoutonStandard" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource BrushPrimaire}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="Padding" Value="15,5"/>
        <Setter Property="FontSize" Value="{StaticResource TaillePoliceNormal}"/>
    </Style>
</ResourceDictionary>
```


### Importation de ressources externes

Vous pouvez importer ces dictionnaires de ressources dans votre application ou dans d'autres dictionnaires :

```xml
<!-- Dans App.xaml -->
<Application x:Class="MonApplication.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="/MonApplication;component/Resources/Couleurs.xaml"/>
                <ResourceDictionary Source="/MonApplication;component/Resources/Styles.xaml"/>
            </ResourceDictionary.MergedDictionaries>

            <!-- Ressources additionnelles spécifiques à l'application -->
            <Thickness x:Key="MargeStandard">10</Thickness>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```


La syntaxe `/MonApplication;component/Resources/Couleurs.xaml` est le chemin de pack URI:
- `/MonApplication` : Nom de l'assembly (votre projet)
- `component` : Mot-clé indiquant une ressource interne
- `/Resources/Couleurs.xaml` : Chemin du fichier dans le projet

### Ressources externes pour les images et autres fichiers

Vous pouvez également inclure des images et d'autres fichiers comme ressources dans votre application:

1. Ajoutez les fichiers à votre projet (placez-les dans un dossier comme `/Images` ou `/Assets`)
2. Définissez la propriété "Build Action" sur "Resource" dans les propriétés du fichier

Utilisation en XAML:

```xml
<Image Source="/MonApplication;component/Images/logo.png" Width="100" Height="100" />
```


Ou en définissant des ressources:

```xml
<Window.Resources>
    <BitmapImage x:Key="LogoImage" UriSource="/MonApplication;component/Images/logo.png" />
</Window.Resources>

<Image Source="{StaticResource LogoImage}" Width="100" Height="100" />
```


## 8.8.4. ResourceDictionary et fusion

Les dictionnaires de ressources peuvent être fusionnés, ce qui vous permet de combiner plusieurs dictionnaires et d'organiser vos ressources logiquement.

### Fusion de dictionnaires

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="/MonApplication;component/Resources/Couleurs.xaml"/>
        <ResourceDictionary Source="/MonApplication;component/Resources/Styles.xaml"/>
        <ResourceDictionary Source="/MonApplication;component/Resources/Templates.xaml"/>
    </ResourceDictionary.MergedDictionaries>

    <!-- Ressources locales qui peuvent utiliser les ressources des dictionnaires fusionnés -->
    <Style x:Key="TitreSpecial" TargetType="TextBlock"
           BasedOn="{StaticResource TextBlockBase}">
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="Foreground" Value="{StaticResource BrushAccent}"/>
    </Style>
</ResourceDictionary>
```


### Ordre de fusion et priorité

L'ordre dans lequel vous fusionnez les dictionnaires est important. Si plusieurs dictionnaires définissent une ressource avec la même clé, c'est la dernière définition qui est utilisée.

```xml
<ResourceDictionary.MergedDictionaries>
    <!-- Dictionnaire de base (sera remplacé si des clés sont redéfinies) -->
    <ResourceDictionary Source="/MonApplication;component/Resources/Base.xaml"/>

    <!-- Dictionnaires spécifiques (ont priorité sur Base.xaml) -->
    <ResourceDictionary Source="/MonApplication;component/Resources/Specifique.xaml"/>
</ResourceDictionary.MergedDictionaries>
```


### Fusion conditionnelle (pour les thèmes)

Vous pouvez utiliser le code-behind pour fusionner des dictionnaires conditionnellement :

```textmate
// Dans App.xaml.cs
protected override void OnStartup(StartupEventArgs e)
{
    base.OnStartup(e);

    // Déterminer quel thème charger (par exemple, basé sur une préférence utilisateur)
    bool utilisateurPrefereModeNuit = Properties.Settings.Default.ModeSombre;

    ResourceDictionary theme = new ResourceDictionary();

    if (utilisateurPrefereModeNuit)
    {
        theme.Source = new Uri("/MonApplication;component/Resources/ThemeNuit.xaml", UriKind.Relative);
    }
    else
    {
        theme.Source = new Uri("/MonApplication;component/Resources/ThemeJour.xaml", UriKind.Relative);
    }

    // Remplacer ou ajouter le dictionnaire de thème
    ResourceDictionary existingTheme = Application.Current.Resources.MergedDictionaries
        .FirstOrDefault(d => d.Source != null && d.Source.ToString().Contains("Theme"));

    if (existingTheme != null)
    {
        int index = Application.Current.Resources.MergedDictionaries.IndexOf(existingTheme);
        Application.Current.Resources.MergedDictionaries[index] = theme;
    }
    else
    {
        Application.Current.Resources.MergedDictionaries.Add(theme);
    }
}
```


## 8.8.5. Thèmes dynamiques

Les thèmes dynamiques vous permettent de changer l'apparence de votre application pendant son exécution, souvent en réponse aux actions de l'utilisateur (comme basculer entre un thème clair et sombre).

### Structure pour les thèmes

Organisez vos thèmes en plusieurs dictionnaires de ressources :

1. **ThemeBase.xaml** - Contient les styles et ressources communs à tous les thèmes
2. **ThemeJour.xaml** et **ThemeNuit.xaml** - Contiennent les couleurs spécifiques à chaque thème

```xml
<!-- ThemeBase.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- Styles communs qui utilisent des brushes dynamiques -->
    <Style x:Key="TextBlockBase" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI"/>
        <Setter Property="Foreground" Value="{DynamicResource TextColorBrush}"/>
    </Style>

    <Style x:Key="BoutonStandard" TargetType="Button">
        <Setter Property="Background" Value="{DynamicResource PrimaryColorBrush}"/>
        <Setter Property="Foreground" Value="{DynamicResource PrimaryTextColorBrush}"/>
        <Setter Property="Padding" Value="15,5"/>
    </Style>

    <Style TargetType="Window">
        <Setter Property="Background" Value="{DynamicResource BackgroundColorBrush}"/>
    </Style>
</ResourceDictionary>
```


```xml
<!-- ThemeJour.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- Couleurs du thème clair -->
    <SolidColorBrush x:Key="BackgroundColorBrush" Color="#FFFFFF"/>
    <SolidColorBrush x:Key="TextColorBrush" Color="#333333"/>
    <SolidColorBrush x:Key="PrimaryColorBrush" Color="#2196F3"/>
    <SolidColorBrush x:Key="PrimaryTextColorBrush" Color="White"/>
    <SolidColorBrush x:Key="AccentColorBrush" Color="#FF4081"/>
</ResourceDictionary>
```


```xml
<!-- ThemeNuit.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- Couleurs du thème sombre -->
    <SolidColorBrush x:Key="BackgroundColorBrush" Color="#252525"/>
    <SolidColorBrush x:Key="TextColorBrush" Color="#E0E0E0"/>
    <SolidColorBrush x:Key="PrimaryColorBrush" Color="#1976D2"/>
    <SolidColorBrush x:Key="PrimaryTextColorBrush" Color="White"/>
    <SolidColorBrush x:Key="AccentColorBrush" Color="#FF5252"/>
</ResourceDictionary>
```


### Chargement initial du thème

Dans App.xaml, chargez le thème de base et le thème par défaut :

```xml
<Application x:Class="MonApplication.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <!-- Dictionnaire de styles communs -->
                <ResourceDictionary Source="/MonApplication;component/Resources/ThemeBase.xaml"/>

                <!-- Thème par défaut (sera remplacé dynamiquement) -->
                <ResourceDictionary Source="/MonApplication;component/Resources/ThemeJour.xaml"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```


### Implémentation du changement de thème

Créez une classe utilitaire pour gérer le changement de thème :

```textmate
// ThemeManager.cs
public class ThemeManager
{
    public enum ThemeType
    {
        Jour,
        Nuit
    }

    private static ThemeType _currentTheme = ThemeType.Jour;

    public static ThemeType CurrentTheme
    {
        get => _currentTheme;
        private set => _currentTheme = value;
    }

    public static void ChangeTheme(ThemeType theme)
    {
        // Définir le nouveau thème
        CurrentTheme = theme;

        // Créer l'URI pour le dictionnaire de ressources du thème
        string themePath = $"/MonApplication;component/Resources/Theme{theme}.xaml";
        Uri themeUri = new Uri(themePath, UriKind.Relative);

        // Rechercher le dictionnaire de thème actuel
        ResourceDictionary currentTheme = Application.Current.Resources.MergedDictionaries
            .FirstOrDefault(d => d.Source != null &&
                                 (d.Source.ToString().Contains("ThemeJour") ||
                                  d.Source.ToString().Contains("ThemeNuit")));

        // Créer le nouveau dictionnaire
        ResourceDictionary newTheme = new ResourceDictionary();
        newTheme.Source = themeUri;

        // Remplacer l'ancien thème par le nouveau
        if (currentTheme != null)
        {
            int index = Application.Current.Resources.MergedDictionaries.IndexOf(currentTheme);
            Application.Current.Resources.MergedDictionaries[index] = newTheme;
        }
        else
        {
            Application.Current.Resources.MergedDictionaries.Add(newTheme);
        }

        // Sauvegarder la préférence de l'utilisateur
        Properties.Settings.Default.ModeSombre = (theme == ThemeType.Nuit);
        Properties.Settings.Default.Save();
    }

    // Méthode pour basculer entre les thèmes
    public static void ToggleTheme()
    {
        if (CurrentTheme == ThemeType.Jour)
        {
            ChangeTheme(ThemeType.Nuit);
        }
        else
        {
            ChangeTheme(ThemeType.Jour);
        }
    }
}
```


### Utilisation du gestionnaire de thèmes

Dans votre fenêtre ou un contrôle, ajoutez un bouton pour changer de thème :

```xml
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Thèmes dynamiques" Height="450" Width="800">
    <Grid Margin="20">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <ToggleButton x:Name="themeToggle" Content="Basculer en mode nuit"
                     Click="ThemeToggle_Click"
                     HorizontalAlignment="Right"
                     Padding="10,5"/>

        <StackPanel Grid.Row="1" VerticalAlignment="Center">
            <TextBlock Text="Titre de l'application"
                       FontSize="24"
                       HorizontalAlignment="Center"
                       Margin="0,0,0,20"/>

            <TextBlock Text="Ce texte change de couleur avec le thème"
                       Style="{StaticResource TextBlockBase}"
                       HorizontalAlignment="Center"
                       Margin="0,0,0,20"/>

            <Button Content="Bouton standard"
                    Style="{StaticResource BoutonStandard}"
                    HorizontalAlignment="Center"
                    Width="150"/>
        </StackPanel>
    </Grid>
</Window>
```


Code-behind :

```textmate
// MainWindow.xaml.cs
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        // Initialiser l'état du bouton bascule en fonction du thème actuel
        themeToggle.IsChecked = ThemeManager.CurrentTheme == ThemeManager.ThemeType.Nuit;
    }

    private void ThemeToggle_Click(object sender, RoutedEventArgs e)
    {
        ThemeManager.ToggleTheme();

        // Mettre à jour le texte du bouton
        if (ThemeManager.CurrentTheme == ThemeManager.ThemeType.Nuit)
        {
            themeToggle.Content = "Basculer en mode jour";
        }
        else
        {
            themeToggle.Content = "Basculer en mode nuit";
        }
    }
}
```


### Thèmes dynamiques avec .NET 8

Pour .NET 8, vous pouvez utiliser les nouvelles fonctionnalités comme les couleurs d'accent Windows et les thèmes du système :

```textmate
// Windows 10/11 accent color
private void ApplyWindowsAccentColor()
{
    try
    {
        // Obtenir la couleur d'accent de Windows
        var uiSettings = new Windows.UI.ViewManagement.UISettings();
        var accentColor = uiSettings.GetColorValue(Windows.UI.ViewManagement.UIColorType.Accent);

        // Convertir en couleur WPF
        Color wpfAccentColor = Color.FromArgb(
            accentColor.A,
            accentColor.R,
            accentColor.G,
            accentColor.B);

        // Appliquer comme ressource
        Application.Current.Resources["AccentColorBrush"] = new SolidColorBrush(wpfAccentColor);
    }
    catch
    {
        // Fallback pour les systèmes non Windows ou en cas d'erreur
        Application.Current.Resources["AccentColorBrush"] = new SolidColorBrush(Colors.DodgerBlue);
    }
}

// Détection du thème système
private void ApplySystemTheme()
{
    try
    {
        // Déterminer si le thème du système est sombre
        var uiSettings = new Windows.UI.ViewManagement.UISettings();
        var foreground = uiSettings.GetColorValue(Windows.UI.ViewManagement.UIColorType.Foreground);

        // Si la couleur du texte est claire, c'est probablement un thème sombre
        bool isSystemDark = foreground.R > 200 && foreground.G > 200 && foreground.B > 200;

        if (isSystemDark)
        {
            ThemeManager.ChangeTheme(ThemeManager.ThemeType.Nuit);
        }
        else
        {
            ThemeManager.ChangeTheme(ThemeManager.ThemeType.Jour);
        }
    }
    catch
    {
        // Fallback
        ThemeManager.ChangeTheme(ThemeManager.ThemeType.Jour);
    }
}
```


### Bonnes pratiques pour les thèmes dynamiques

1. **Utilisez `DynamicResource` au lieu de `StaticResource`** pour les ressources qui peuvent changer avec le thème

```xml
<!-- Correct pour les thèmes dynamiques -->
   <TextBlock Foreground="{DynamicResource TextColorBrush}" />

   <!-- Ne fonctionnera pas si le thème change -->
   <TextBlock Foreground="{StaticResource TextColorBrush}" />
```


2. **Organisez vos ressources en couches** :
   - Base : Styles et templates communs
   - Thèmes : Couleurs et apparence spécifiques au thème
   - Branding : Logos et ressources spécifiques à la marque

3. **Utilisez des noms de clés sémantiques** plutôt que liés à des couleurs spécifiques :
   - Bon : `PrimaryColorBrush`, `BackgroundColorBrush`
   - Mauvais : `BlueBrush`, `WhiteBrush`

4. **Considérez l'accessibilité** : Assurez-vous que vos thèmes ont un contraste suffisant pour tous les utilisateurs

5. **Testez vos thèmes** avec différentes résolutions d'écran et paramètres d'affichage

En maîtrisant les ressources et les dictionnaires de ressources, vous pouvez créer des applications WPF cohérentes, maintenables et adaptables aux préférences des utilisateurs. Les thèmes dynamiques sont particulièrement utiles pour offrir des options de personnalisation et améliorer l'expérience utilisateur dans différentes conditions d'éclairage.

⏭️
