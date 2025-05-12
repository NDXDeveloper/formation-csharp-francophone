# 13.2. MAUI (Multi-platform App UI)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 13.2.1. Introduction et architecture

### Qu'est-ce que .NET MAUI ?

.NET MAUI (Multi-platform App UI) est un framework moderne de développement cross-platform créé par Microsoft, évolution directe de Xamarin.Forms. MAUI permet aux développeurs C# de créer des applications natives qui fonctionnent sur plusieurs plateformes à partir d'une base de code unique:

- Android
- iOS
- macOS
- Windows

MAUI est intégré directement dans l'écosystème .NET unifié (à partir de .NET 6) et n'est plus un produit séparé comme l'était Xamarin. Cette intégration offre une expérience de développement plus fluide et cohérente.

### Histoire et évolution

Pour comprendre MAUI, il est utile de connaître son évolution:

- **Xamarin.Forms** (2014-2021) : Première plateforme cross-platform de Microsoft pour le développement mobile
- **Project MAUI** (2020) : Annonce de l'évolution de Xamarin.Forms
- **Première preview** (2021) : Sortie des premières versions tests
- **Version officielle** (Mai 2022) : Lancement avec .NET 6
- **Améliorations continues** : Versions majeures avec chaque nouvelle version de .NET

### Architecture fondamentale

L'architecture de MAUI est conçue pour maximiser le partage de code tout en permettant un accès complet aux fonctionnalités natives de chaque plateforme:

```
┌───────────────────────────────────────────────────────┐
│               Application MAUI (C# et XAML)           │
├───────────────────────────────────────────────────────┤
│  Contrôles MAUI  │   Services Partagés  │ Ressources  │
├───────────────────────────────────────────────────────┤
│                 Handlers & Renderers                  │
├────────────┬────────────┬────────────┬────────────────┤
│  Android   │    iOS     │   macOS    │    Windows     │
└────────────┴────────────┴────────────┴────────────────┘
```

#### Les couches principales de l'architecture:

1. **Application MAUI** - Votre code C# et XAML partagé entre toutes les plateformes
2. **Contrôles et services** - Éléments d'interface, services partagés, ressources communes
3. **Handlers et Renderers** - Convertissent les contrôles MAUI en contrôles natifs de chaque plateforme
4. **Plateformes cibles** - Code spécifique aux plateformes (Android, iOS, macOS, Windows)

### Nouveautés par rapport à Xamarin.Forms

MAUI apporte plusieurs améliorations significatives par rapport à Xamarin.Forms:

- **Projet unique** - Un seul projet au lieu d'un projet par plateforme
- **Performances améliorées** - Redesign complet des contrôles pour de meilleures performances
- **Support desktop natif** - Intégration complète des applications de bureau (Windows/macOS)
- **Hot Reload** - Visualisation des changements en temps réel
- **Syntaxe simplifiée** - APIs plus modernes et consistantes
- **Intégration dans .NET** - Compatibilité avec l'écosystème .NET unifié

### Environnement de développement requis

Pour développer avec .NET MAUI, vous aurez besoin de:

- **Visual Studio 2022** (Windows) ou **Visual Studio pour Mac**
- **SDK .NET 6** ou supérieur
- **Charge de travail** "Développement d'applications mobiles avec .NET"
- **Émulateurs** ou appareils physiques pour tester

## 13.2.2. Développement multiplateforme

### Structure d'un projet MAUI

Lorsque vous créez un nouveau projet MAUI, vous remarquerez une structure différente des projets Xamarin traditionnels:

```
MonProjetMAUI/
├── Platforms/                  # Code spécifique aux plateformes
│   ├── Android/
│   │   ├── MainActivity.cs
│   │   └── MainApplication.cs
│   ├── iOS/
│   │   └── AppDelegate.cs
│   ├── MacCatalyst/            # Pour macOS
│   │   └── AppDelegate.cs
│   └── Windows/
│       └── App.xaml
├── Resources/                  # Ressources partagées
│   ├── Fonts/                  # Polices personnalisées
│   ├── Images/                 # Images partagées
│   ├── Raw/                    # Fichiers bruts
│   └── Styles/                 # Styles partagés
├── App.xaml                    # Point d'entrée XAML
├── App.xaml.cs                 # Code-behind du point d'entrée
├── AppShell.xaml               # Navigation Shell
├── MauiProgram.cs              # Configuration de l'application
└── MonProjetMAUI.csproj        # Fichier de projet unique
```

Le concept clé est que toutes les plateformes sont maintenant définies dans un **projet unique** avec une structure claire pour le code spécifique à chaque plateforme.

### Création d'un premier projet MAUI

Voici les étapes détaillées pour créer votre première application MAUI:

1. Ouvrez Visual Studio 2022
2. Sélectionnez "Créer un nouveau projet"
3. Recherchez "MAUI" et sélectionnez ".NET MAUI App"
4. Nommez votre projet (ex: "PremiereAppMAUI")
5. Configurez le projet et cliquez sur "Créer"

Visual Studio génère un projet avec une page d'accueil simple qui fonctionne sur toutes les plateformes prises en charge.

### Configuration du projet

Le fichier `.csproj` est le cœur de la configuration d'un projet MAUI. Voici un exemple typique:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFrameworks>net8.0-android;net8.0-ios;net8.0-maccatalyst</TargetFrameworks>
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net8.0-windows10.0.19041.0</TargetFrameworks>
    <OutputType>Exe</OutputType>
    <RootNamespace>PremiereAppMAUI</RootNamespace>
    <UseMaui>true</UseMaui>
    <SingleProject>true</SingleProject>
    <ImplicitUsings>enable</ImplicitUsings>

    <!-- Versions -->
    <ApplicationDisplayVersion>1.0</ApplicationDisplayVersion>
    <ApplicationVersion>1</ApplicationVersion>

    <!-- Configurations pour iOS et Mac -->
    <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">14.2</SupportedOSPlatformVersion>
    <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'maccatalyst'">14.0</SupportedOSPlatformVersion>
    <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">21.0</SupportedOSPlatformVersion>
    <SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">10.0.17763.0</SupportedOSPlatformVersion>
    <TargetPlatformMinVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">10.0.17763.0</TargetPlatformMinVersion>
  </PropertyGroup>

  <ItemGroup>
    <!-- Images et autres ressources -->
    <MauiIcon Include="Resources\AppIcon\appicon.svg" ForegroundFile="Resources\AppIcon\appiconfg.svg" Color="#512BD4" />
    <MauiSplashScreen Include="Resources\Splash\splash.svg" Color="#512BD4" BaseSize="128,128" />
    <MauiImage Include="Resources\Images\*" />
    <MauiFont Include="Resources\Fonts\*" />
    <MauiAsset Include="Resources\Raw\**" LogicalName="%(RecursiveDir)%(Filename)%(Extension)" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Maui.Controls" Version="$(MauiVersion)" />
    <PackageReference Include="Microsoft.Maui.Controls.Compatibility" Version="$(MauiVersion)" />
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="8.0.0" />
  </ItemGroup>

</Project>
```

Points importants à noter:
- `<TargetFrameworks>` définit les plateformes cibles
- `<SingleProject>true</SingleProject>` active le concept de projet unique
- `<UseMaui>true</UseMaui>` active le SDK MAUI
- Les versions minimales des OS sont définies pour chaque plateforme
- Les ressources sont regroupées par catégories (images, polices, etc.)

### Point d'entrée de l'application

Le point d'entrée principal est défini dans `MauiProgram.cs`:

```csharp
using Microsoft.Extensions.Logging;

namespace PremiereAppMAUI;

public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            });

#if DEBUG
        builder.Logging.AddDebug();
#endif

        // Enregistrement des services (injection de dépendances)
        builder.Services.AddSingleton<MainPage>();
        builder.Services.AddSingleton<MainViewModel>();

        return builder.Build();
    }
}
```

Ce fichier est crucial car il:
- Configure l'application MAUI
- Définit les ressources globales (comme les polices)
- Enregistre les services pour l'injection de dépendances
- Configure les options de journalisation

### Cycle de vie de l'application

MAUI définit plusieurs événements dans le cycle de vie d'une application que vous pouvez gérer dans le fichier `App.xaml.cs`:

```csharp
namespace PremiereAppMAUI;

public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        MainPage = new AppShell();
    }

    protected override void OnStart()
    {
        // L'application démarre
        base.OnStart();
    }

    protected override void OnSleep()
    {
        // L'application passe en arrière-plan
        base.OnSleep();
    }

    protected override void OnResume()
    {
        // L'application revient au premier plan
        base.OnResume();
    }
}
```

### Gestion code partagé vs spécifique

MAUI offre plusieurs façons de traiter le code spécifique à chaque plateforme:

#### 1. Dossiers Platforms

Le code spécifique à chaque plateforme est organisé dans les sous-dossiers du dossier `Platforms`. Par exemple, pour Android:

```csharp
// Platforms/Android/MainActivity.cs
namespace PremiereAppMAUI;

[Activity(Theme = "@style/Maui.SplashTheme", MainLauncher = true)]
public class MainActivity : MauiAppCompatActivity
{
    // Code spécifique à Android
}
```

#### 2. Compilations conditionnelles

```csharp
public void PrendreMeilleureCaptureEcran()
{
#if ANDROID
    // Code uniquement pour Android
    var screenshot = Platform.CurrentActivity.Window.DecorView;
#elif IOS
    // Code uniquement pour iOS
    var screenshot = UIScreen.MainScreen.Capture();
#elif WINDOWS
    // Code uniquement pour Windows
    var screenshot = new Windows.UI.WindowCapture();
#elif MACCATALYST
    // Code uniquement pour macOS
    var screenshot = NSScreen.MainScreen.Screenshot();
#else
    throw new PlatformNotSupportedException("Plateforme non supportée");
#endif
}
```

#### 3. Interface d'abstraction avec implémentations par plateforme

```csharp
// Interface partagée
public interface IPhotoService
{
    Task<Stream> PrendrePhotoAsync();
}

// Dans le dossier Platforms/Android
public class PhotoService : IPhotoService
{
    public async Task<Stream> PrendrePhotoAsync()
    {
        // Implémentation spécifique à Android
    }
}

// Dans le dossier Platforms/iOS
public class PhotoService : IPhotoService
{
    public async Task<Stream> PrendrePhotoAsync()
    {
        // Implémentation spécifique à iOS
    }
}
```

L'enregistrement du service se fait dans `MauiProgram.cs`:

```csharp
builder.Services.AddSingleton<IPhotoService, PhotoService>();
```

## 13.2.3. Création d'interfaces utilisateur

### XAML fondamental

MAUI utilise XAML (eXtensible Application Markup Language) pour définir les interfaces utilisateur de manière déclarative. Voici un exemple de fichier XAML pour une page simple:

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:viewmodels="clr-namespace:PremiereAppMAUI.ViewModels"
             x:Class="PremiereAppMAUI.MainPage"
             Title="Accueil">

    <ContentPage.BindingContext>
        <viewmodels:MainViewModel />
    </ContentPage.BindingContext>

    <ScrollView>
        <VerticalStackLayout Spacing="25" Padding="30">
            <Image Source="dotnet_bot.png"
                   HeightRequest="185"
                   SemanticProperties.Description="Le bot .NET" />

            <Label Text="Bonjour, MAUI !"
                   FontSize="32"
                   HorizontalOptions="Center" />

            <Label Text="{Binding Message}"
                   FontSize="18"
                   HorizontalOptions="Center" />

            <Button Text="Cliquez-moi"
                    Command="{Binding ClickCommand}"
                    HorizontalOptions="Center" />

            <HorizontalStackLayout HorizontalOptions="Center">
                <Label Text="Compteur: "
                       VerticalOptions="Center" />
                <Label Text="{Binding Count}"
                       FontAttributes="Bold"
                       VerticalOptions="Center" />
            </HorizontalStackLayout>
        </VerticalStackLayout>
    </ScrollView>
</ContentPage>
```

Le fichier de code-behind associé (MainPage.xaml.cs):

```csharp
namespace PremiereAppMAUI;

public partial class MainPage : ContentPage
{
    public MainPage(MainViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
}
```

### Types de pages

MAUI propose plusieurs types de pages pour structurer vos applications:

#### ContentPage

La page la plus basique, contenant un seul contenu principal:

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="PremiereAppMAUI.DetailPage"
             Title="Détails">
    <VerticalStackLayout>
        <Label Text="Page de détails" />
    </VerticalStackLayout>
</ContentPage>
```

#### TabbedPage

Permet de naviguer entre différentes pages à l'aide d'onglets:

```xml
<TabbedPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            xmlns:local="clr-namespace:PremiereAppMAUI"
            x:Class="PremiereAppMAUI.TabbedMainPage"
            Title="Application">
    <local:HomePage />
    <local:SettingsPage />
    <local:ProfilePage />
</TabbedPage>
```

#### FlyoutPage

Crée un menu latéral (drawer) pour la navigation:

```xml
<FlyoutPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            xmlns:local="clr-namespace:PremiereAppMAUI"
            x:Class="PremiereAppMAUI.FlyoutMainPage">
    <FlyoutPage.Flyout>
        <local:MenuPage />
    </FlyoutPage.Flyout>
    <FlyoutPage.Detail>
        <NavigationPage>
            <x:Arguments>
                <local:HomePage />
            </x:Arguments>
        </NavigationPage>
    </FlyoutPage.Detail>
</FlyoutPage>
```

#### NavigationPage

Permet une navigation par pile (pages empilées):

```xml
<NavigationPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                xmlns:local="clr-namespace:PremiereAppMAUI"
                x:Class="PremiereAppMAUI.NavigationRoot">
    <x:Arguments>
        <local:HomePage />
    </x:Arguments>
</NavigationPage>
```

#### Shell (Recommandé)

MAUI Shell est le système de navigation recommandé, offrant une expérience moderne et cohérente:

```xml
<Shell xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
       xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
       xmlns:local="clr-namespace:PremiereAppMAUI"
       x:Class="PremiereAppMAUI.AppShell"
       Title="PremiereAppMAUI"
       FlyoutBehavior="Flyout">

    <Shell.FlyoutHeader>
        <Grid BackgroundColor="#4CAF50" HeightRequest="100">
            <Label Text="Mon Application"
                   VerticalOptions="Center"
                   HorizontalOptions="Center"
                   TextColor="White"
                   FontSize="24"/>
        </Grid>
    </Shell.FlyoutHeader>

    <ShellContent Title="Accueil"
                  Icon="home.png"
                  ContentTemplate="{DataTemplate local:MainPage}"
                  Route="MainPage" />

    <ShellContent Title="Profil"
                  Icon="profile.png"
                  ContentTemplate="{DataTemplate local:ProfilePage}"
                  Route="ProfilePage" />

    <ShellContent Title="Paramètres"
                  Icon="settings.png"
                  ContentTemplate="{DataTemplate local:SettingsPage}"
                  Route="SettingsPage" />

    <TabBar>
        <Tab Title="Produits" Icon="products.png">
            <ShellContent ContentTemplate="{DataTemplate local:ProductsPage}"
                          Route="ProductsPage" />
        </Tab>
        <Tab Title="Catégories" Icon="categories.png">
            <ShellContent ContentTemplate="{DataTemplate local:CategoriesPage}"
                          Route="CategoriesPage" />
        </Tab>
    </TabBar>
</Shell>
```

L'enregistrement des routes Shell se fait dans le code-behind:

```csharp
public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();

        // Enregistrement des routes pour la navigation
        Routing.RegisterRoute(nameof(ItemDetailPage), typeof(ItemDetailPage));
        Routing.RegisterRoute(nameof(NewItemPage), typeof(NewItemPage));
    }
}
```

### Navigation entre les pages

#### Navigation Shell (recommandée)

```csharp
// Navigation vers une page enregistrée
await Shell.Current.GoToAsync(nameof(ItemDetailPage));

// Navigation avec paramètres
await Shell.Current.GoToAsync($"{nameof(ItemDetailPage)}?id={item.Id}");

// Navigation absolue (depuis la racine)
await Shell.Current.GoToAsync("//MainPage");

// Navigation relative (retour)
await Shell.Current.GoToAsync("..");
```

Pour recevoir les paramètres, utilisez l'attribut `QueryProperty`:

```csharp
[QueryProperty(nameof(ItemId), "id")]
public partial class ItemDetailPage : ContentPage
{
    private string _itemId;
    public string ItemId
    {
        get => _itemId;
        set
        {
            _itemId = value;
            LoadItem(value);
        }
    }

    private void LoadItem(string itemId)
    {
        // Chargement de l'item avec l'ID spécifié
    }
}
```

#### Navigation traditionnelle

Si vous utilisez `NavigationPage`, vous pouvez utiliser les méthodes de navigation traditionnelles:

```csharp
// Navigation vers une nouvelle page
await Navigation.PushAsync(new DetailPage(item));

// Retour à la page précédente
await Navigation.PopAsync();

// Navigation à la racine
await Navigation.PopToRootAsync();

// Navigation modale
await Navigation.PushModalAsync(new ModalPage());
await Navigation.PopModalAsync();
```

### Le pattern MVVM

MAUI supporte nativement le pattern MVVM (Model-View-ViewModel) qui permet de séparer la logique métier de l'interface utilisateur:

#### Modèle

```csharp
public class Item
{
    public string Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public string ImageUrl { get; set; }
}
```

#### ViewModel

```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows.Input;

namespace PremiereAppMAUI.ViewModels
{
    public class MainViewModel : INotifyPropertyChanged
    {
        private string _message;
        private int _count;

        public string Message
        {
            get => _message;
            set => SetProperty(ref _message, value);
        }

        public int Count
        {
            get => _count;
            set => SetProperty(ref _count, value);
        }

        public ICommand ClickCommand { get; }

        public MainViewModel()
        {
            _message = "Bienvenue dans votre application MAUI!";
            _count = 0;
            ClickCommand = new Command(OnClicked);
        }

        private void OnClicked()
        {
            Count++;
            Message = Count == 1
                ? "Vous avez cliqué 1 fois!"
                : $"Vous avez cliqué {Count} fois!";
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected bool SetProperty<T>(ref T field, T value, [CallerMemberName] string propertyName = null)
        {
            if (EqualityComparer<T>.Default.Equals(field, value))
                return false;

            field = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
            return true;
        }
    }
}
```

#### Vue

La vue a été présentée précédemment avec le XAML pour MainPage.

### Data Binding

Le binding de données est un concept fondamental dans MAUI:

#### Types de binding

```xml
<!-- Binding simple -->
<Label Text="{Binding Name}" />

<!-- Binding avec chemin -->
<Label Text="{Binding Address.City}" />

<!-- Binding avec conversion -->
<Label Text="{Binding Price, StringFormat='{0:C}'}" />

<!-- Binding bidirectionnel -->
<Entry Text="{Binding UserName, Mode=TwoWay}" />

<!-- Binding avec source explicite -->
<Label Text="{Binding Source={x:Reference nameSlider}, Path=Value}" />

<!-- Multi-binding -->
<Label>
    <Label.Text>
        <MultiBinding StringFormat="{}{0} {1}">
            <Binding Path="FirstName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </Label.Text>
</Label>
```

#### Méthodes de binding

- **OneTime** - Binding effectué une seule fois
- **OneWay** - Les changements dans le ViewModel sont reflétés dans la vue (par défaut)
- **TwoWay** - Les changements dans la vue et le ViewModel sont synchronisés
- **OneWayToSource** - Les changements dans la vue sont reflétés dans le ViewModel

### Styles et ressources

MAUI permet de définir des styles pour les contrôles afin d'assurer une cohérence visuelle:

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="PremiereAppMAUI.StyledPage">

    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Style pour les titres -->
            <Style x:Key="TitleStyle" TargetType="Label">
                <Setter Property="FontSize" Value="24" />
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="TextColor" Value="#333333" />
                <Setter Property="Margin" Value="0,0,0,10" />
            </Style>

            <!-- Style pour les sous-titres -->
            <Style x:Key="SubtitleStyle" TargetType="Label">
                <Setter Property="FontSize" Value="18" />
                <Setter Property="FontAttributes" Value="Italic" />
                <Setter Property="TextColor" Value="#666666" />
                <Setter Property="Margin" Value="0,0,0,5" />
            </Style>

            <!-- Style pour les boutons -->
            <Style x:Key="PrimaryButtonStyle" TargetType="Button">
                <Setter Property="BackgroundColor" Value="#4CAF50" />
                <Setter Property="TextColor" Value="White" />
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="CornerRadius" Value="8" />
                <Setter Property="Padding" Value="20,10" />
                <Setter Property="Margin" Value="0,10,0,0" />
            </Style>
        </ResourceDictionary>
    </ContentPage.Resources>

    <ScrollView>
        <VerticalStackLayout Padding="20">
            <Label Text="Bienvenue" Style="{StaticResource TitleStyle}" />
            <Label Text="Découvrez votre nouvelle application"
                   Style="{StaticResource SubtitleStyle}" />

            <Button Text="Commencer"
                    Style="{StaticResource PrimaryButtonStyle}" />
        </VerticalStackLayout>
    </ScrollView>
</ContentPage>
```

### Styles globaux

Pour définir des styles globaux pour toute l'application, utilisez `App.xaml`:

```xml
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="PremiereAppMAUI.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>

            <!-- Couleurs globales -->
            <Color x:Key="PrimaryColor">#4CAF50</Color>
            <Color x:Key="SecondaryColor">#FF9800</Color>
            <Color x:Key="TextColor">#333333</Color>
            <Color x:Key="BackgroundColor">#F5F5F5</Color>

            <!-- Styles globaux -->
            <Style TargetType="Label">
                <Setter Property="TextColor" Value="{StaticResource TextColor}" />
                <Setter Property="FontFamily" Value="OpenSansRegular" />
            </Style>

            <Style TargetType="Button">
                <Setter Property="BackgroundColor" Value="{StaticResource PrimaryColor}" />
                <Setter Property="TextColor" Value="White" />
                <Setter Property="FontFamily" Value="OpenSansSemibold" />
                <Setter Property="CornerRadius" Value="8" />
                <Setter Property="Padding" Value="14,10" />
            </Style>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

## 13.2.4. Accès aux API natives

### Essentials - Fonctionnalités communes

MAUI intègre .NET MAUI Essentials, qui fournit un accès unifié aux fonctionnalités de l'appareil:

#### État de connectivité

```csharp
using Microsoft.Maui.Networking;

// Vérifier l'état de la connectivité
var networkAccess = Connectivity.Current.NetworkAccess;
if (networkAccess == NetworkAccess.Internet)
{
    // L'appareil est connecté à Internet
}

// Écouter les changements de connectivité
Connectivity.Current.ConnectivityChanged += (s, e) =>
{
    var access = e.NetworkAccess;
    var profiles = e.ConnectionProfiles;

    if (access == NetworkAccess.Internet)
        Console.WriteLine("Internet disponible!");
    else
        Console.WriteLine("Internet non disponible!");

    if (profiles.Contains(ConnectionProfile.WiFi))
        Console.WriteLine("WiFi disponible!");
};
```
### Géolocalisation

```csharp
using Microsoft.Maui.Devices.Sensors;

try
{
    var location = await Geolocation.Default.GetLastKnownLocationAsync();

    if (location != null)
    {
        Console.WriteLine($"Latitude: {location.Latitude}, Longitude: {location.Longitude}");
        Console.WriteLine($"Altitude: {location.Altitude}");
        Console.WriteLine($"Précision: {location.Accuracy} mètres");
    }
    else
    {
        // Obtenir la position actuelle si aucune position récente n'est disponible
        var request = new GeolocationRequest(GeolocationAccuracy.Medium);
        location = await Geolocation.Default.GetLocationAsync(request);

        Console.WriteLine($"Latitude: {location.Latitude}, Longitude: {location.Longitude}");
    }
}
catch (FeatureNotSupportedException)
{
    // La géolocalisation n'est pas supportée sur l'appareil
    await DisplayAlert("Erreur", "La géolocalisation n'est pas supportée sur cet appareil", "OK");
}
catch (PermissionException)
{
    // L'accès à la géolocalisation n'est pas autorisé
    await DisplayAlert("Erreur", "Permission de géolocalisation refusée", "OK");
}
catch (Exception)
{
    // Erreur générale
    await DisplayAlert("Erreur", "Impossible d'obtenir la position", "OK");
}
```

### Accéléromètre

```csharp
using Microsoft.Maui.Devices.Sensors;

// Vérifier si l'accéléromètre est disponible
if (Accelerometer.Default.IsSupported)
{
    // Configurer le suivi des mouvements
    if (!Accelerometer.Default.IsMonitoring)
    {
        // Définir la vitesse de mise à jour (rapide, jeu, normale, économie d'énergie)
        Accelerometer.Default.ShakeDetected += Accelerometer_ShakeDetected;
        Accelerometer.Default.ReadingChanged += Accelerometer_ReadingChanged;

        // Démarrer la surveillance
        Accelerometer.Default.Start(SensorSpeed.Game);
    }
}

// Gestionnaire d'événements pour la détection de secousses
private void Accelerometer_ShakeDetected(object sender, EventArgs e)
{
    // L'appareil a été secoué
    MainThread.BeginInvokeOnMainThread(() =>
    {
        statusLabel.Text = "Appareil secoué!";
    });
}

// Gestionnaire d'événements pour les changements de valeurs
private void Accelerometer_ReadingChanged(object sender, AccelerometerChangedEventArgs e)
{
    // Récupérer les valeurs X, Y et Z actuelles
    var data = e.Reading;
    MainThread.BeginInvokeOnMainThread(() =>
    {
        xLabel.Text = $"X: {data.Acceleration.X:F2}";
        yLabel.Text = $"Y: {data.Acceleration.Y:F2}";
        zLabel.Text = $"Z: {data.Acceleration.Z:F2}";
    });
}

// Arrêter la surveillance lorsqu'elle n'est plus nécessaire
protected override void OnDisappearing()
{
    if (Accelerometer.Default.IsMonitoring)
    {
        Accelerometer.Default.Stop();
        Accelerometer.Default.ShakeDetected -= Accelerometer_ShakeDetected;
        Accelerometer.Default.ReadingChanged -= Accelerometer_ReadingChanged;
    }

    base.OnDisappearing();
}
```

### Batterie

```csharp
using Microsoft.Maui.Devices;

// Vérifier le niveau de batterie actuel
var batteryLevel = Battery.Default.ChargeLevel; // 0.0 à 1.0
var percentage = batteryLevel * 100;
var state = Battery.Default.State;
var source = Battery.Default.PowerSource;

Console.WriteLine($"Niveau de batterie: {percentage:F0}%");

switch (state)
{
    case BatteryState.Charging:
        Console.WriteLine("La batterie est en charge");
        break;
    case BatteryState.Discharging:
        Console.WriteLine("La batterie se décharge");
        break;
    case BatteryState.Full:
        Console.WriteLine("La batterie est pleine");
        break;
    case BatteryState.NotCharging:
        Console.WriteLine("La batterie n'est pas en charge");
        break;
    case BatteryState.NotPresent:
        Console.WriteLine("Pas de batterie détectée");
        break;
    case BatteryState.Unknown:
        Console.WriteLine("État de la batterie inconnu");
        break;
}

// Suivre les changements de batterie
Battery.Default.BatteryInfoChanged += (s, e) =>
{
    Console.WriteLine($"Niveau de batterie changé: {e.ChargeLevel * 100:F0}%");
};
```

### Partage

```csharp
using Microsoft.Maui.ApplicationModel.DataTransfer;

public async Task PartagerTexte()
{
    await Share.Default.RequestAsync(new ShareTextRequest
    {
        Text = "Découvrez cette application incroyable !",
        Title = "Partager avec vos amis"
    });
}

public async Task PartagerFichier(string cheminFichier)
{
    await Share.Default.RequestAsync(new ShareFileRequest
    {
        Title = "Partager ce document",
        File = new ShareFile(cheminFichier)
    });
}

public async Task PartagerMultiplesFichiers(List<string> cheminsFichiers)
{
    var fichiers = new List<ShareFile>();
    foreach (var chemin in cheminsFichiers)
    {
        fichiers.Add(new ShareFile(chemin));
    }

    await Share.Default.RequestAsync(new ShareMultipleFilesRequest
    {
        Title = "Partager ces documents",
        Files = fichiers
    });
}
```

### Préférences

```csharp
using Microsoft.Maui.Storage;

// Enregistrer des données
Preferences.Default.Set("thème", "sombre");
Preferences.Default.Set("notifications", true);
Preferences.Default.Set("volume", 0.75);

// Récupérer des données avec valeurs par défaut
var thème = Preferences.Default.Get("thème", "clair");
var notifications = Preferences.Default.Get("notifications", false);
var volume = Preferences.Default.Get("volume", 0.5);

// Vérifier si une clé existe
if (Preferences.Default.ContainsKey("dernière_utilisation"))
{
    var dernièreUtilisation = Preferences.Default.Get("dernière_utilisation", DateTime.Now);
}

// Supprimer une préférence
Preferences.Default.Remove("paramètre_temporaire");

// Supprimer toutes les préférences
Preferences.Default.Clear();
```

### Lampe de poche

```csharp
using Microsoft.Maui.Devices;

public async Task UtiliserLampeTorche()
{
    try
    {
        // Vérifier si la lampe de poche est supportée
        if (Flashlight.Default.IsSupported)
        {
            // Allumer la lampe de poche
            await Flashlight.Default.TurnOnAsync();

            // Attendre 2 secondes
            await Task.Delay(2000);

            // Éteindre la lampe de poche
            await Flashlight.Default.TurnOffAsync();
        }
        else
        {
            await DisplayAlert("Non supporté", "La lampe de poche n'est pas disponible sur cet appareil", "OK");
        }
    }
    catch (FeatureNotSupportedException)
    {
        await DisplayAlert("Erreur", "La lampe de poche n'est pas supportée", "OK");
    }
    catch (PermissionException)
    {
        await DisplayAlert("Erreur", "Permission d'utiliser la lampe de poche refusée", "OK");
    }
    catch (Exception ex)
    {
        await DisplayAlert("Erreur", $"Une erreur s'est produite: {ex.Message}", "OK");
    }
}
```

### Vibreur

```csharp
using Microsoft.Maui.Devices;

public void Vibrer()
{
    // Vibrer pendant 500 millisecondes
    if (Vibration.Default.IsSupported)
    {
        Vibration.Default.Vibrate(500);
    }
}

public void VibrerMotif()
{
    // Vibrer selon un motif personnalisé (en millisecondes)
    if (Vibration.Default.IsSupported)
    {
        var motif = new[] { 500, 200, 500, 200, 1000 };  // durée, pause, durée, pause, durée
        Vibration.Default.Vibrate(motif);
    }
}

public void ArrêterVibration()
{
    // Arrêter toute vibration en cours
    Vibration.Default.Cancel();
}
```

### Gestion des permissions

La gestion des permissions est essentielle pour accéder aux fonctionnalités natives:

```csharp
using Microsoft.Maui.ApplicationModel;

public async Task<bool> DemanderPermissionAppareilPhoto()
{
    var status = await Permissions.CheckStatusAsync<Permissions.Camera>();

    if (status == PermissionStatus.Granted)
        return true;

    if (status == PermissionStatus.Denied && DeviceInfo.Platform == DevicePlatform.iOS)
    {
        // Sur iOS, si l'utilisateur a refusé une fois, on doit l'informer
        await DisplayAlert("Permission requise",
            "Veuillez autoriser l'accès à l'appareil photo dans les paramètres", "OK");
        return false;
    }

    status = await Permissions.RequestAsync<Permissions.Camera>();
    return status == PermissionStatus.Granted;
}

// Exemple d'utilisation
public async Task PrendrePhoto()
{
    bool autorisé = await DemanderPermissionAppareilPhoto();

    if (autorisé)
    {
        // Code pour prendre une photo
    }
    else
    {
        await DisplayAlert("Permission refusée",
            "Impossible de prendre une photo sans accès à l'appareil photo", "OK");
    }
}
```

Vérifier plusieurs permissions:

```csharp
public async Task<bool> VérifierPermissions()
{
    var statutLocalisation = await Permissions.CheckStatusAsync<Permissions.LocationWhenInUse>();
    var statutPhotos = await Permissions.CheckStatusAsync<Permissions.Photos>();

    List<Permissions.BasePermission> permissionsÀDemander = new();

    if (statutLocalisation != PermissionStatus.Granted)
        permissionsÀDemander.Add(new Permissions.LocationWhenInUse());

    if (statutPhotos != PermissionStatus.Granted)
        permissionsÀDemander.Add(new Permissions.Photos());

    if (permissionsÀDemander.Any())
    {
        bool afficherRationale = false;

        // Vérifier si nous devons expliquer pourquoi nous avons besoin des permissions
        foreach (var permission in permissionsÀDemander)
        {
            if (await Permissions.ShouldShowRationaleAsync(permission))
            {
                afficherRationale = true;
                break;
            }
        }

        if (afficherRationale)
        {
            await DisplayAlert("Permissions requises",
                "Cette application a besoin d'accéder à votre localisation et à vos photos pour fonctionner correctement.",
                "OK");
        }

        // Demander toutes les permissions nécessaires
        foreach (var permission in permissionsÀDemander)
        {
            var result = await Permissions.RequestAsync(permission);
            if (result != PermissionStatus.Granted)
                return false;
        }
    }

    return true;
}
```

### Services personnalisés spécifiques aux plateformes

Pour les fonctionnalités plus avancées ou non couvertes par Essentials, vous pouvez créer des interfaces et des implémentations spécifiques à chaque plateforme:

```csharp
// Interface partagée
public interface IExportService
{
    Task<bool> ExportPdf(string contenu, string nomFichier);
}

// Implémentation Android (Platforms/Android/ExportService.cs)
public class ExportService : IExportService
{
    public async Task<bool> ExportPdf(string contenu, string nomFichier)
    {
        try
        {
            // Code spécifique à Android pour générer et exporter un PDF
            var context = Platform.CurrentActivity;
            // ...
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur d'exportation PDF: {ex.Message}");
            return false;
        }
    }
}

// Implémentation iOS (Platforms/iOS/ExportService.cs)
public class ExportService : IExportService
{
    public async Task<bool> ExportPdf(string contenu, string nomFichier)
    {
        try
        {
            // Code spécifique à iOS pour générer et exporter un PDF
            // ...
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erreur d'exportation PDF: {ex.Message}");
            return false;
        }
    }
}
```

L'enregistrement du service dans `MauiProgram.cs`:

```csharp
builder.Services.AddSingleton<IExportService, ExportService>();
```

## 13.2.5. Contrôles et layouts

### Contrôles de base

#### Labels et texte

```xml
<!-- Label simple -->
<Label Text="Bonjour MAUI!"
       FontSize="24"
       TextColor="Blue"
       HorizontalOptions="Center" />

<!-- Label multiligne -->
<Label Text="Ceci est un texte multiligne qui peut s'étendre sur plusieurs lignes et s'adapter à la largeur disponible."
       LineBreakMode="WordWrap" />

<!-- Label avec formatage -->
<Label FontAttributes="Bold,Italic"
       TextDecorations="Underline"
       LineHeight="1.5" />

<!-- Label avec HTML -->
<Label TextType="Html">
    <Label.Text>
        <![CDATA[
            <b>Texte en gras</b>, <i>texte en italique</i> et <u>texte souligné</u><br>
            <a href="https://dotnet.microsoft.com/maui">Lien vers MAUI</a>
        ]]>
    </Label.Text>
</Label>

<!-- Label avec style FormattedText pour une mise en forme avancée -->
<Label>
    <Label.FormattedText>
        <FormattedString>
            <Span Text="Texte normal " />
            <Span Text="en gras " FontAttributes="Bold" />
            <Span Text="et en " />
            <Span Text="couleur" TextColor="Red" />
        </FormattedString>
    </Label.FormattedText>
</Label>
```

#### Entrée de texte

```xml
<!-- Champ de texte simple -->
<Entry Placeholder="Entrez votre nom"
       Text="{Binding UserName}"
       Keyboard="Text"
       MaxLength="50"
       ReturnType="Next" />

<!-- Entrée de mot de passe -->
<Entry Placeholder="Mot de passe"
       IsPassword="True"
       Text="{Binding Password}" />

<!-- Entrée avec validation -->
<Entry Placeholder="Email"
       Text="{Binding Email}"
       Keyboard="Email"
       TextChanged="OnEmailTextChanged" />

<!-- Zone de texte multiligne -->
<Editor Placeholder="Description"
        Text="{Binding Description}"
        AutoSize="TextChanges"
        MaxLength="500" />
```

#### Boutons

```xml
<!-- Bouton standard -->
<Button Text="Cliquez-moi"
        Clicked="OnButtonClicked"
        BackgroundColor="Blue"
        TextColor="White"
        CornerRadius="10" />

<!-- Bouton avec icône -->
<Button Text="Enregistrer"
        ImageSource="save_icon.png" />

<!-- Bouton bordé -->
<Button Text="Annuler"
        BorderColor="Gray"
        BorderWidth="2"
        BackgroundColor="Transparent"
        TextColor="Black" />

<!-- Bouton d'image -->
<ImageButton Source="heart.png"
             Clicked="OnLikeClicked"
             BackgroundColor="Transparent"
             CornerRadius="30"
             HeightRequest="60"
             WidthRequest="60" />
```

#### Sélection

```xml
<!-- Case à cocher -->
<CheckBox IsChecked="{Binding AcceptTerms, Mode=TwoWay}"
          Color="Blue" />

<!-- Commutateur (On/Off) -->
<Switch IsToggled="{Binding NotificationsEnabled, Mode=TwoWay}"
        OnColor="Green"
        ThumbColor="White" />

<!-- Sélecteur de date -->
<DatePicker Date="{Binding BirthDate, Mode=TwoWay}"
            MinimumDate="01/01/1900"
            MaximumDate="{Binding MaxDate}"
            Format="dd/MM/yyyy" />

<!-- Sélecteur d'heure -->
<TimePicker Time="{Binding MeetingTime, Mode=TwoWay}"
            Format="HH:mm" />

<!-- Liste déroulante -->
<Picker Title="Choisissez une option"
        ItemsSource="{Binding Options}"
        SelectedItem="{Binding SelectedOption}"
        TitleColor="Gray" />

<!-- Curseur -->
<Slider Minimum="0"
        Maximum="100"
        Value="{Binding SliderValue}"
        MinimumTrackColor="Blue"
        MaximumTrackColor="LightGray"
        ThumbColor="DarkBlue" />

<!-- Barre de progression -->
<ProgressBar Progress="{Binding ProgressValue}"
             ProgressColor="Green" />
```

#### Images

```xml
<!-- Image simple -->
<Image Source="logo.png"
       HeightRequest="100"
       Aspect="AspectFit" />

<!-- Image depuis une URL -->
<Image Source="https://dotnet.microsoft.com/static/images/illustrations/maui-dot-net.svg"
       Aspect="AspectFill"
       WidthRequest="200" />

<!-- Image avec options d'accessibilité -->
<Image Source="icon.png"
       SemanticProperties.Description="Logo de l'application"
       SemanticProperties.Hint="Image décorative" />

<!-- Image circulaire -->
<Frame CornerRadius="75"
       HeightRequest="150"
       WidthRequest="150"
       IsClippedToBounds="True"
       Padding="0"
       BorderColor="Transparent">
    <Image Source="profile.jpg"
           Aspect="AspectFill" />
</Frame>
```

### Contrôles de collections

#### CollectionView (Recommandé)

Le `CollectionView` est le contrôle moderne recommandé pour afficher des listes de données:

```xml
<CollectionView ItemsSource="{Binding Products}"
                SelectionMode="Single"
                SelectionChanged="OnProductSelected">
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <Grid Padding="10">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto" />
                    <RowDefinition Height="Auto" />
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto" />
                    <ColumnDefinition Width="*" />
                </Grid.ColumnDefinitions>

                <Image Grid.RowSpan="2"
                       Source="{Binding ImageUrl}"
                       Aspect="AspectFill"
                       HeightRequest="60"
                       WidthRequest="60" />

                <Label Grid.Column="1"
                       Text="{Binding Name}"
                       FontAttributes="Bold" />

                <Label Grid.Row="1"
                       Grid.Column="1"
                       Text="{Binding Price, StringFormat='{0:C}'}"
                       TextColor="Green" />
            </Grid>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

#### Disposition des éléments

Le `CollectionView` peut afficher les éléments de différentes façons:

```xml
<!-- Liste verticale (par défaut) -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.ItemsLayout>
        <LinearItemsLayout Orientation="Vertical" ItemSpacing="10" />
    </CollectionView.ItemsLayout>
    <!-- ... -->
</CollectionView>

<!-- Liste horizontale -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.ItemsLayout>
        <LinearItemsLayout Orientation="Horizontal" ItemSpacing="10" />
    </CollectionView.ItemsLayout>
    <!-- ... -->
</CollectionView>

<!-- Grille -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.ItemsLayout>
        <GridItemsLayout Orientation="Vertical"
                         Span="2"
                         HorizontalItemSpacing="10"
                         VerticalItemSpacing="10" />
    </CollectionView.ItemsLayout>
    <!-- ... -->
</CollectionView>
```

#### Groupement

```xml
<CollectionView ItemsSource="{Binding GroupedProducts}">
    <CollectionView.GroupHeaderTemplate>
        <DataTemplate>
            <Label Text="{Binding Name}"
                   BackgroundColor="LightGray"
                   FontAttributes="Bold"
                   Padding="10" />
        </DataTemplate>
    </CollectionView.GroupHeaderTemplate>

    <CollectionView.ItemTemplate>
        <DataTemplate>
            <!-- Modèle pour chaque élément du groupe -->
            <Grid Padding="10">
                <!-- ... -->
            </Grid>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

Le ViewModel associé:

```csharp
public ObservableCollection<ProductGroup> GroupedProducts { get; }

public MainViewModel()
{
    // Créer des groupes
    var electronics = new ProductGroup("Électronique", new List<Product>
    {
        new Product { Name = "Smartphone", Price = 799.99m },
        new Product { Name = "Laptop", Price = 1299.99m },
        // ...
    });

    var clothing = new ProductGroup("Vêtements", new List<Product>
    {
        new Product { Name = "T-shirt", Price = 24.99m },
        new Product { Name = "Jeans", Price = 59.99m },
        // ...
    });

    GroupedProducts = new ObservableCollection<ProductGroup> { electronics, clothing };
}

public class ProductGroup : List<Product>
{
    public string Name { get; }

    public ProductGroup(string name, List<Product> products) : base(products)
    {
        Name = name;
    }
}
```

#### CarouselView

```xml
<CarouselView ItemsSource="{Binding Slides}"
              PeekAreaInsets="40"
              Loop="True">
    <CarouselView.ItemTemplate>
        <DataTemplate>
            <Frame HasShadow="True"
                   BorderColor="LightGray"
                   CornerRadius="10"
                   Margin="20"
                   Padding="0">
                <Grid>
                    <Image Source="{Binding ImageUrl}"
                           Aspect="AspectFill"
                           HeightRequest="200" />
                    <Label Text="{Binding Title}"
                           TextColor="White"
                           BackgroundColor="#80000000"
                           Padding="10"
                           VerticalOptions="End"
                           HorizontalOptions="FillAndExpand" />
                </Grid>
            </Frame>
        </DataTemplate>
    </CarouselView.ItemTemplate>
</CarouselView>
```

#### IndicatorView (pour CarouselView)

```xml
<Grid>
    <CarouselView x:Name="carouselView"
                  ItemsSource="{Binding Slides}"
                  Loop="True">
        <!-- ... -->
    </CarouselView>

    <IndicatorView HorizontalOptions="Center"
                   VerticalOptions="End"
                   Margin="0,0,0,20"
                   SelectedIndicatorColor="Blue"
                   IndicatorColor="LightGray"
                   IndicatorSize="10"
                   ItemsSource="{Binding Slides}"
                   SelectedPosition="{Binding Source={x:Reference carouselView}, Path=Position}" />
</Grid>
```

### Layouts principaux

#### StackLayout (Vertical et Horizontal)

```xml
<!-- Empilement vertical (le plus commun) -->
<StackLayout Spacing="10" Padding="20">
    <Label Text="Titre" FontSize="24" />
    <Entry Placeholder="Nom" />
    <Entry Placeholder="Email" />
    <Button Text="Envoyer" />
</StackLayout>

<!-- Empilement horizontal -->
<HorizontalStackLayout Spacing="10">
    <Label Text="Quantité:" VerticalOptions="Center" />
    <Button Text="-" WidthRequest="40" />
    <Label Text="1" VerticalOptions="Center" />
    <Button Text="+" WidthRequest="40" />
</HorizontalStackLayout>
```

> Note: Depuis .NET 6, il est recommandé d'utiliser `VerticalStackLayout` et `HorizontalStackLayout` au lieu de `StackLayout` pour des performances améliorées.

#### Grid

```xml
<Grid RowDefinitions="Auto,*,Auto"
      ColumnDefinitions="*,*"
      RowSpacing="10"
      ColumnSpacing="10"
      Padding="20">

    <!-- En-tête (s'étend sur les deux colonnes) -->
    <Label Text="Formulaire"
           Grid.ColumnSpan="2"
           FontSize="24"
           HorizontalOptions="Center" />

    <!-- Contenu principal -->
    <VerticalStackLayout Grid.Row="1" Grid.Column="0">
        <Label Text="Informations personnelles" FontAttributes="Bold" />
        <Entry Placeholder="Nom" />
        <Entry Placeholder="Prénom" />
        <DatePicker />
    </VerticalStackLayout>

    <VerticalStackLayout Grid.Row="1" Grid.Column="1">
        <Label Text="Informations de contact" FontAttributes="Bold" />
        <Entry Placeholder="Email" />
        <Entry Placeholder="Téléphone" />
        <Entry Placeholder="Adresse" />
    </VerticalStackLayout>

    <!-- Pied de page -->
    <Button Text="Annuler"
            Grid.Row="2"
            Grid.Column="0" />

    <Button Text="Enregistrer"
            Grid.Row="2"
            Grid.Column="1" />
</Grid>
```

#### FlexLayout

Inspiré de Flexbox en CSS, ce layout est très flexible pour organiser les éléments:

```xml
<FlexLayout Direction="Row"
            Wrap="Wrap"
            JustifyContent="SpaceAround"
            AlignItems="Center"
            AlignContent="SpaceAround">

    <Button Text="Bouton 1" />
    <Button Text="Bouton 2" />
    <Button Text="Bouton plus long 3" />
    <Button Text="4" />
    <Button Text="Bouton 5" />
    <Button Text="Bouton 6" />
</FlexLayout>
```

#### AbsoluteLayout

Permet un positionnement précis des éléments:

```xml
<AbsoluteLayout>
    <!-- Position par coordonnées -->
    <Label Text="Texte positionné"
           AbsoluteLayout.LayoutBounds="20, 20, 100, 25" />

    <!-- Position proportionnelle -->
    <Button Text="Bouton"
            AbsoluteLayout.LayoutBounds="0.5, 0.5, 200, 50"
            AbsoluteLayout.LayoutFlags="PositionProportional" />

    <!-- Position et taille proportionnelles -->
    <Frame BackgroundColor="LightBlue"
           AbsoluteLayout.LayoutBounds="0, 0, 1, 0.3"
           AbsoluteLayout.LayoutFlags="SizeProportional" />
</AbsoluteLayout>
```

### Contrôles de navigation

#### Tabs

```xml
<TabBar>
    <Tab Title="Accueil" Icon="home.png">
        <ShellContent ContentTemplate="{DataTemplate local:HomePage}" />
    </Tab>
    <Tab Title="Profil" Icon="profile.png">
        <ShellContent ContentTemplate="{DataTemplate local:ProfilePage}" />
    </Tab>
    <Tab Title="Paramètres" Icon="settings.png">
        <ShellContent ContentTemplate="{DataTemplate local:SettingsPage}" />
    </Tab>
</TabBar>
```

#### Flyout (Menu latéral)

```xml
<Shell ...
       FlyoutBehavior="Flyout">

    <Shell.FlyoutHeader>
        <Grid HeightRequest="200" BackgroundColor="#4CAF50">
            <Image Source="logo.png"
                   Aspect="AspectFit"
                   HeightRequest="100" />
        </Grid>
    </Shell.FlyoutHeader>

    <Shell.FlyoutFooter>
        <Label Text="© 2025 Mon Application"
               HorizontalTextAlignment="Center"
               Padding="10" />
    </Shell.FlyoutFooter>

    <FlyoutItem Title="Accueil" Icon="home.png">
        <ShellContent ContentTemplate="{DataTemplate local:HomePage}" />
    </FlyoutItem>

    <FlyoutItem Title="Profil" Icon="profile.png">
        <ShellContent ContentTemplate="{DataTemplate local:ProfilePage}" />
    </FlyoutItem>

    <FlyoutItem Title="Paramètres" Icon="settings.png">
        <ShellContent ContentTemplate="{DataTemplate local:SettingsPage}" />
    </FlyoutItem>

    <MenuItem Text="Déconnexion"
              IconImageSource="logout.png"
              Command="{Binding LogoutCommand}" />
</Shell>
```

#### Personnalisation des menus dans Shell

```csharp
// Dans le code-behind
protected override void OnAppearing()
{
    base.OnAppearing();

    // Ajouter un élément de menu dynamiquement
    Shell.Current.Items.Add(new FlyoutItem
    {
        Title = "Nouveautés",
        Icon = "new.png",
        Items =
        {
            new ShellContent
            {
                Content = new NewsPage()
            }
        }
    });

    // Ajouter un élément de menu contextuel
    Shell.Current.FlyoutHeader = new FlyoutHeaderControl();

    // Ajouter une action de menu
    Shell.Current.Items.Add(new MenuItem
    {
        Text = "À propos",
        Command = new Command(() => DisplayAlert("À propos", "Version 1.0", "OK"))
    });
}
```

### Contrôles personnalisés

La création de contrôles personnalisés est essentielle pour la réutilisation des éléments d'interface communs.

#### Création d'un contrôle simple

```csharp
using Microsoft.Maui.Controls;

namespace MonApplication.Controls
{
    public class RatingControl : HorizontalStackLayout
    {
        // Propriété bindable pour le nombre d'étoiles
        public static readonly BindableProperty RatingProperty =
            BindableProperty.Create(
                nameof(Rating),
                typeof(int),
                typeof(RatingControl),
                0,
                propertyChanged: OnRatingChanged);

        // Propriété bindable pour la taille des étoiles
        public static readonly BindableProperty StarSizeProperty =
            BindableProperty.Create(
                nameof(StarSize),
                typeof(double),
                typeof(RatingControl),
                24.0,
                propertyChanged: OnStarSizeChanged);

        // Propriété bindable pour la couleur des étoiles
        public static readonly BindableProperty StarColorProperty =
            BindableProperty.Create(
                nameof(StarColor),
                typeof(Color),
                typeof(RatingControl),
                Colors.Gold,
                propertyChanged: OnStarColorChanged);

        // Propriété pour accéder à Rating
        public int Rating
        {
            get => (int)GetValue(RatingProperty);
            set => SetValue(RatingProperty, value);
        }

        // Propriété pour accéder à StarSize
        public double StarSize
        {
            get => (double)GetValue(StarSizeProperty);
            set => SetValue(StarSizeProperty, value);
        }

        // Propriété pour accéder à StarColor
        public Color StarColor
        {
            get => (Color)GetValue(StarColorProperty);
            set => SetValue(StarColorProperty, value);
        }

        // Événement pour notifier des changements de notation
        public event EventHandler<int> RatingChanged;

        // Constructeur
        public RatingControl()
        {
            Spacing = 5;
            HorizontalOptions = LayoutOptions.Start;
            UpdateStars();

            // Ajouter un gestionnaire de tap
            TapGestureRecognizer tapGesture = new TapGestureRecognizer();
            tapGesture.Tapped += OnStarTapped;
            this.GestureRecognizers.Add(tapGesture);
        }

        // Gestionnaire d'événement de tap
        private void OnStarTapped(object sender, EventArgs e)
        {
            if (sender is Grid starGrid && starGrid.BindingContext is int position)
            {
                Rating = position + 1;
                RatingChanged?.Invoke(this, Rating);
            }
        }

        // Méthode pour mettre à jour les étoiles lors du changement de la propriété Rating
        private static void OnRatingChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var control = (RatingControl)bindable;
            control.UpdateStars();
        }

        // Méthode pour mettre à jour la taille des étoiles
        private static void OnStarSizeChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var control = (RatingControl)bindable;
            control.UpdateStars();
        }

        // Méthode pour mettre à jour la couleur des étoiles
        private static void OnStarColorChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var control = (RatingControl)bindable;
            control.UpdateStars();
        }

        // Méthode pour créer et mettre à jour les étoiles
        private void UpdateStars()
        {
            Children.Clear();

            for (int i = 0; i < 5; i++)
            {
                var starGrid = new Grid
                {
                    HeightRequest = StarSize,
                    WidthRequest = StarSize,
                    BindingContext = i
                };

                var emptyStar = new Image
                {
                    Source = "star_empty.png",
                    Aspect = Aspect.AspectFit
                };

                var filledStar = new Image
                {
                    Source = "star_filled.png",
                    Aspect = Aspect.AspectFit,
                    IsVisible = i < Rating,
                    // Appliquer la couleur pour les étoiles colorées
                    TintColor = StarColor
                };

                starGrid.Children.Add(emptyStar);
                starGrid.Children.Add(filledStar);

                Children.Add(starGrid);
            }
        }
    }
}
```

#### Utilisation du contrôle personnalisé

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:controls="clr-namespace:MonApplication.Controls"
             x:Class="MonApplication.ProductDetailPage">

    <VerticalStackLayout Padding="20">
        <Label Text="Évaluation du produit:" />

        <controls:RatingControl Rating="{Binding ProductRating, Mode=TwoWay}"
                               StarSize="32"
                               StarColor="Gold" />

        <Label Text="{Binding ProductRating, StringFormat='Note actuelle: {0}/5'}" />
    </VerticalStackLayout>
</ContentPage>
```

### Comportements et déclencheurs

Les comportements et déclencheurs permettent d'ajouter des fonctionnalités réutilisables aux contrôles.

#### Comportement personnalisé

```csharp
using Microsoft.Maui.Controls;

namespace MonApplication.Behaviors
{
    public class EmailValidationBehavior : Behavior<Entry>
    {
        protected override void OnAttachedTo(Entry entry)
        {
            entry.TextChanged += OnEntryTextChanged;
            base.OnAttachedTo(entry);
        }

        protected override void OnDetachingFrom(Entry entry)
        {
            entry.TextChanged -= OnEntryTextChanged;
            base.OnDetachingFrom(entry);
        }

        private void OnEntryTextChanged(object sender, TextChangedEventArgs e)
        {
            if (!(sender is Entry entry))
                return;

            bool isValid = IsValidEmail(e.NewTextValue);
            entry.TextColor = isValid ? Colors.Black : Colors.Red;
        }

        private bool IsValidEmail(string email)
        {
            if (string.IsNullOrWhiteSpace(email))
                return false;

            try
            {
                var addr = new System.Net.Mail.MailAddress(email);
                return addr.Address == email;
            }
            catch
            {
                return false;
            }
        }
    }
}
```

Utilisation dans XAML:

```xml
<Entry Placeholder="Email"
       Text="{Binding Email}">
    <Entry.Behaviors>
        <local:EmailValidationBehavior />
    </Entry.Behaviors>
</Entry>
```

#### Déclencheurs

Les déclencheurs permettent de réagir aux changements de propriétés ou d'événements:

```xml
<Entry Placeholder="Mot de passe">
    <Entry.Triggers>
        <!-- Déclencher un changement de couleur quand le texte est vide -->
        <Trigger TargetType="Entry"
                 Property="Text"
                 Value="">
            <Setter Property="BackgroundColor" Value="LightPink" />
        </Trigger>

        <!-- Déclencher une action quand l'entrée reçoit le focus -->
        <EventTrigger Event="Focused">
            <local:FocusedTriggerAction />
        </EventTrigger>
    </Entry.Triggers>
</Entry>
```

### Exemples pratiques

#### Exemple 1: Écran de connexion

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:behaviors="clr-namespace:MonApplication.Behaviors"
             x:Class="MonApplication.LoginPage"
             Title="Connexion">

    <Grid RowDefinitions="Auto,*,Auto"
          Padding="20">

        <!-- Logo et titre -->
        <VerticalStackLayout Grid.Row="0"
                             HorizontalOptions="Center"
                             Spacing="20"
                             Margin="0,50,0,0">
            <Image Source="app_logo.png"
                   HeightRequest="120"
                   WidthRequest="120" />
            <Label Text="Mon Application"
                   FontSize="28"
                   FontAttributes="Bold"
                   HorizontalOptions="Center" />
        </VerticalStackLayout>

        <!-- Formulaire de connexion -->
        <VerticalStackLayout Grid.Row="1"
                             VerticalOptions="Center"
                             Spacing="20">

            <Entry Placeholder="Email ou nom d'utilisateur"
                   Text="{Binding Username}"
                   ReturnType="Next">
                <Entry.Behaviors>
                    <behaviors:RequiredFieldBehavior />
                </Entry.Behaviors>
            </Entry>

            <Grid ColumnDefinitions="*,Auto">
                <Entry Grid.Column="0"
                       Placeholder="Mot de passe"
                       Text="{Binding Password}"
                       IsPassword="{Binding IsPasswordHidden}"
                       ReturnType="Done"
                       ReturnCommand="{Binding LoginCommand}">
                    <Entry.Behaviors>
                        <behaviors:RequiredFieldBehavior />
                    </Entry.Behaviors>
                </Entry>

                <Button Grid.Column="1"
                        Text="{Binding PasswordVisibilityIcon}"
                        Command="{Binding TogglePasswordVisibilityCommand}"
                        WidthRequest="50"
                        BackgroundColor="Transparent"
                        TextColor="Gray" />
            </Grid>

            <Button Text="Se connecter"
                    Command="{Binding LoginCommand}"
                    IsEnabled="{Binding CanLogin}"
                    HeightRequest="50"
                    Margin="0,20,0,0" />

            <Label Text="{Binding ErrorMessage}"
                   TextColor="Red"
                   IsVisible="{Binding HasError}"
                   HorizontalOptions="Center" />

            <Label Text="Mot de passe oublié ?"
                   TextColor="Blue"
                   TextDecorations="Underline"
                   HorizontalOptions="Center">
                <Label.GestureRecognizers>
                    <TapGestureRecognizer Command="{Binding ForgotPasswordCommand}" />
                </Label.GestureRecognizers>
            </Label>
        </VerticalStackLayout>

        <!-- Bouton d'inscription -->
        <Button Grid.Row="2"
                Text="Créer un compte"
                Command="{Binding RegisterCommand}"
                BackgroundColor="Transparent"
                TextColor="{StaticResource Primary}"
                BorderColor="{StaticResource Primary}"
                BorderWidth="1"
                Margin="0,0,0,30" />
    </Grid>
</ContentPage>
```

ViewModel associé:

```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows.Input;

namespace MonApplication.ViewModels
{
    public class LoginViewModel : INotifyPropertyChanged
    {
        private string _username;
        private string _password;
        private bool _isPasswordHidden = true;
        private string _errorMessage;
        private bool _hasError;
        private IAuthService _authService;

        public string Username
        {
            get => _username;
            set
            {
                if (_username != value)
                {
                    _username = value;
                    OnPropertyChanged();
                    OnPropertyChanged(nameof(CanLogin));
                }
            }
        }

        public string Password
        {
            get => _password;
            set
            {
                if (_password != value)
                {
                    _password = value;
                    OnPropertyChanged();
                    OnPropertyChanged(nameof(CanLogin));
                }
            }
        }

        public bool IsPasswordHidden
        {
            get => _isPasswordHidden;
            set
            {
                if (_isPasswordHidden != value)
                {
                    _isPasswordHidden = value;
                    OnPropertyChanged();
                    OnPropertyChanged(nameof(PasswordVisibilityIcon));
                }
            }
        }

        public string PasswordVisibilityIcon => IsPasswordHidden ? "👁️" : "🔒";

        public string ErrorMessage
        {
            get => _errorMessage;
            set
            {
                if (_errorMessage != value)
                {
                    _errorMessage = value;
                    OnPropertyChanged();
                    HasError = !string.IsNullOrEmpty(value);
                }
            }
        }

        public bool HasError
        {
            get => _hasError;
            set
            {
                if (_hasError != value)
                {
                    _hasError = value;
                    OnPropertyChanged();
                }
            }
        }

        public bool CanLogin => !string.IsNullOrWhiteSpace(Username) &&
                                !string.IsNullOrWhiteSpace(Password);

        public ICommand LoginCommand { get; }
        public ICommand TogglePasswordVisibilityCommand { get; }
        public ICommand ForgotPasswordCommand { get; }
        public ICommand RegisterCommand { get; }

        public LoginViewModel(IAuthService authService)
        {
            _authService = authService;

            LoginCommand = new Command(OnLoginClicked, () => CanLogin);
            TogglePasswordVisibilityCommand = new Command(OnTogglePasswordVisibility);
            ForgotPasswordCommand = new Command(OnForgotPassword);
            RegisterCommand = new Command(OnRegister);
        }

        private async void OnLoginClicked()
        {
            try
            {
                var result = await _authService.LoginAsync(Username, Password);
                if (result.Success)
                {
                    // Naviguer vers la page principale
                    await Shell.Current.GoToAsync("//MainPage");
                }
                else
                {
                    ErrorMessage = result.Message;
                }
            }
            catch (Exception ex)
            {
                ErrorMessage = "Une erreur s'est produite. Veuillez réessayer.";
            }
        }

        private void OnTogglePasswordVisibility()
        {
            IsPasswordHidden = !IsPasswordHidden;
        }

        private async void OnForgotPassword()
        {
            await Shell.Current.GoToAsync("ResetPasswordPage");
        }

        private async void OnRegister()
        {
            await Shell.Current.GoToAsync("RegisterPage");
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```

#### Exemple 2: Liste de tâches avec MVVM

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:viewmodels="clr-namespace:MonApplication.ViewModels"
             x:Class="MonApplication.TodoListPage"
             Title="Mes tâches">

    <ContentPage.BindingContext>
        <viewmodels:TodoListViewModel />
    </ContentPage.BindingContext>

    <Grid RowDefinitions="Auto,*,Auto"
          Padding="20">

        <!-- En-tête -->
        <VerticalStackLayout Grid.Row="0"
                             Spacing="10"
                             Margin="0,0,0,10">
            <Label Text="Liste de tâches"
                   FontSize="24"
                   FontAttributes="Bold" />

            <Label Text="{Binding TaskSummary}"
                   FontSize="14"
                   TextColor="Gray" />

            <BoxView HeightRequest="1"
                     Color="LightGray"
                     Margin="0,10,0,0" />
        </VerticalStackLayout>

        <!-- Liste des tâches -->
        <RefreshView Grid.Row="1"
                     IsRefreshing="{Binding IsRefreshing}"
                     Command="{Binding RefreshCommand}">
            <CollectionView ItemsSource="{Binding Tasks}"
                            SelectionMode="None"
                            EmptyView="Aucune tâche pour le moment. Ajoutez-en une !">

                <CollectionView.ItemTemplate>
                    <DataTemplate>
                        <SwipeView>
                            <SwipeView.RightItems>
                                <SwipeItems>
                                    <SwipeItem Text="Supprimer"
                                              BackgroundColor="Red"
                                              Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodels:TodoListViewModel}}, Path=DeleteTaskCommand}"
                                              CommandParameter="{Binding}" />
                                </SwipeItems>
                            </SwipeView.RightItems>

                            <Grid Padding="5"
                                  ColumnDefinitions="Auto,*,Auto">

                                <CheckBox Grid.Column="0"
                                         IsChecked="{Binding IsCompleted, Mode=TwoWay}"
                                         VerticalOptions="Center" />

                                <Label Grid.Column="1"
                                       Text="{Binding Title}"
                                       VerticalOptions="Center"
                                       TextDecorations="{Binding IsCompleted, Converter={StaticResource BoolToTextDecorationConverter}}"
                                       TextColor="{Binding IsCompleted, Converter={StaticResource BoolToColorConverter}}" />

                                <ImageButton Grid.Column="2"
                                            Source="edit.png"
                                            BackgroundColor="Transparent"
                                            Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodels:TodoListViewModel}}, Path=EditTaskCommand}"
                                            CommandParameter="{Binding}" />
                            </Grid>
                        </SwipeView>
                    </DataTemplate>
                </CollectionView.ItemTemplate>
            </CollectionView>
        </RefreshView>

        <!-- Ajout de tâche -->
        <Grid Grid.Row="2"
              ColumnDefinitions="*,Auto"
              Margin="0,10,0,0">

            <Entry Grid.Column="0"
                   Placeholder="Nouvelle tâche..."
                   Text="{Binding NewTaskTitle}"
                   ReturnCommand="{Binding AddTaskCommand}" />

            <Button Grid.Column="1"
                    Text="Ajouter"
                    Command="{Binding AddTaskCommand}" />
        </Grid>
    </Grid>
</ContentPage>
```

ViewModel associé:

```csharp
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows.Input;

namespace MonApplication.ViewModels
{
    public class TodoListViewModel : INotifyPropertyChanged
    {
        private ObservableCollection<TodoItem> _tasks;
        private string _newTaskTitle;
        private bool _isRefreshing;
        private ITodoService _todoService;

        public ObservableCollection<TodoItem> Tasks
        {
            get => _tasks;
            set
            {
                if (_tasks != value)
                {
                    _tasks = value;
                    OnPropertyChanged();
                    UpdateTaskSummary();
                }
            }
        }

        public string NewTaskTitle
        {
            get => _newTaskTitle;
            set
            {
                if (_newTaskTitle != value)
                {
                    _newTaskTitle = value;
                    OnPropertyChanged();
                    ((Command)AddTaskCommand).ChangeCanExecute();
                }
            }
        }

        public bool IsRefreshing
        {
            get => _isRefreshing;
            set
            {
                if (_isRefreshing != value)
                {
                    _isRefreshing = value;
                    OnPropertyChanged();
                }
            }
        }

        private string _taskSummary;
        public string TaskSummary
        {
            get => _taskSummary;
            set
            {
                if (_taskSummary != value)
                {
                    _taskSummary = value;
                    OnPropertyChanged();
                }
            }
        }

        public ICommand RefreshCommand { get; }
        public ICommand AddTaskCommand { get; }
        public ICommand DeleteTaskCommand { get; }
        public ICommand EditTaskCommand { get; }

        public TodoListViewModel(ITodoService todoService)
        {
            _todoService = todoService;
            _tasks = new ObservableCollection<TodoItem>();

            RefreshCommand = new Command(async () => await LoadTasksAsync());
            AddTaskCommand = new Command(async () => await AddTaskAsync(), () => !string.IsNullOrWhiteSpace(NewTaskTitle));
            DeleteTaskCommand = new Command<TodoItem>(async (task) => await DeleteTaskAsync(task));
            EditTaskCommand = new Command<TodoItem>(async (task) => await EditTaskAsync(task));

            // Charger les tâches au démarrage
            LoadTasksAsync().ConfigureAwait(false);
        }

        private async Task LoadTasksAsync()
        {
            try
            {
                IsRefreshing = true;
                var tasks = await _todoService.GetTasksAsync();

                Tasks.Clear();
                foreach (var task in tasks)
                {
                    Tasks.Add(task);
                }

                UpdateTaskSummary();
            }
            catch (Exception ex)
            {
                await Shell.Current.DisplayAlert("Erreur", $"Impossible de charger les tâches: {ex.Message}", "OK");
            }
            finally
            {
                IsRefreshing = false;
            }
        }

        private async Task AddTaskAsync()
        {
            if (string.IsNullOrWhiteSpace(NewTaskTitle))
                return;

            try
            {
                var newTask = new TodoItem
                {
                    Id = Guid.NewGuid().ToString(),
                    Title = NewTaskTitle,
                    IsCompleted = false,
                    CreatedAt = DateTime.Now
                };

                await _todoService.AddTaskAsync(newTask);
                Tasks.Add(newTask);
                NewTaskTitle = string.Empty;
                UpdateTaskSummary();
            }
            catch (Exception ex)
            {
                await Shell.Current.DisplayAlert("Erreur", $"Impossible d'ajouter la tâche: {ex.Message}", "OK");
            }
        }

        private async Task DeleteTaskAsync(TodoItem task)
        {
            try
            {
                var confirm = await Shell.Current.DisplayAlert("Confirmation",
                    $"Êtes-vous sûr de vouloir supprimer '{task.Title}' ?", "Oui", "Non");

                if (confirm)
                {
                    await _todoService.DeleteTaskAsync(task.Id);
                    Tasks.Remove(task);
                    UpdateTaskSummary();
                }
            }
            catch (Exception ex)
            {
                await Shell.Current.DisplayAlert("Erreur", $"Impossible de supprimer la tâche: {ex.Message}", "OK");
            }
        }

        private async Task EditTaskAsync(TodoItem task)
        {
            string result = await Shell.Current.DisplayPromptAsync("Modifier la tâche",
                "Nouveau titre:", initialValue: task.Title);

            if (!string.IsNullOrWhiteSpace(result) && result != task.Title)
            {
                task.Title = result;
                await _todoService.UpdateTaskAsync(task);
            }
        }

        private void UpdateTaskSummary()
        {
            int total = Tasks.Count;
            int completed = Tasks.Count(t => t.IsCompleted);

            TaskSummary = $"{completed} sur {total} tâches terminées";
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }

    public class TodoItem : INotifyPropertyChanged
    {
        private string _id;
        private string _title;
        private bool _isCompleted;
        private DateTime _createdAt;

        public string Id
        {
            get => _id;
            set
            {
                if (_id != value)
                {
                    _id = value;
                    OnPropertyChanged();
                }
            }
        }

        public string Title
        {
            get => _title;
            set
            {
                if (_title != value)
                {
                    _title = value;
                    OnPropertyChanged();
                }
            }
        }

        public bool IsCompleted
        {
            get => _isCompleted;
            set
            {
                if (_isCompleted != value)
                {
                    _isCompleted = value;
                    OnPropertyChanged();
                }
            }
        }

        public DateTime CreatedAt
        {
            get => _createdAt;
            set
            {
                if (_createdAt != value)
                {
                    _createdAt = value;
                    OnPropertyChanged();
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```

### Ressources et bonnes pratiques

#### Organisation du code MAUI

Pour maintenir un projet MAUI propre et maintenable, suivez cette structure:

```
MonApplication/
├── Models/                    # Classes de modèles de données
│   ├── TodoItem.cs
│   └── User.cs
├── ViewModels/                # ViewModels pour MVVM
│   ├── BaseViewModel.cs
│   ├── LoginViewModel.cs
│   └── TodoListViewModel.cs
├── Views/                     # Pages de l'application
│   ├── LoginPage.xaml
│   └── TodoListPage.xaml
├── Services/                  # Services et interfaces
│   ├── Interfaces/
│   │   ├── IAuthService.cs
│   │   └── ITodoService.cs
│   ├── AuthService.cs
│   └── TodoService.cs
├── Controls/                  # Contrôles personnalisés
│   ├── RatingControl.cs
│   └── CircularImageView.cs
├── Behaviors/                 # Comportements réutilisables
│   ├── EmailValidationBehavior.cs
│   └── NumericValidationBehavior.cs
├── Converters/                # Convertisseurs de valeurs
│   ├── BoolToColorConverter.cs
│   └── DateTimeToStringConverter.cs
├── Resources/                 # Ressources de l'application
│   ├── Fonts/
│   ├── Images/
│   ├── Raw/
│   └── Styles/
│       ├── Colors.xaml
│       └── Styles.xaml
├── Platforms/                 # Code spécifique aux plateformes
│   ├── Android/
│   ├── iOS/
│   ├── MacCatalyst/
│   └── Windows/
├── App.xaml                   # Point d'entrée XAML
├── App.xaml.cs                # Code-behind du point d'entrée
├── AppShell.xaml              # Navigation Shell
└── MauiProgram.cs             # Configuration de l'application
```

#### Conseils pour de meilleures performances

1. **Utilisez les nouveaux layouts** - Préférez `VerticalStackLayout` et `HorizontalStackLayout` à `StackLayout` pour de meilleures performances.

2. **Limitez les liaisons** - Chaque liaison (binding) a un coût en performances. Limitez-les au minimum nécessaire.

3. **Recyclez les vues** - Utilisez `CollectionView` plutôt que `ListView` pour un meilleur recyclage des cellules.

4. **Chargement à la demande** - Implémentez le chargement à la demande (lazy loading) pour les collections volumineuses:

```csharp
public class IncrementalLoadingCollection : ObservableCollection<Item>, IIncrementalLoading
{
    private int _currentPage = 0;
    private const int ItemsPerPage = 20;
    private bool _hasMoreItems = true;
    private readonly IDataService _dataService;

    public IncrementalLoadingCollection(IDataService dataService)
    {
        _dataService = dataService;
    }

    public bool HasMoreItems => _hasMoreItems;

    public async Task<LoadMoreItemsResult> LoadMoreItemsAsync()
    {
        if (!_hasMoreItems)
            return new LoadMoreItemsResult { Count = 0 };

        try
        {
            var items = await _dataService.GetItemsAsync(_currentPage, ItemsPerPage);

            if (items.Count < ItemsPerPage)
                _hasMoreItems = false;

            foreach (var item in items)
                this.Add(item);

            _currentPage++;
            return new LoadMoreItemsResult { Count = (uint)items.Count };
        }
        catch (Exception)
        {
            _hasMoreItems = false;
            return new LoadMoreItemsResult { Count = 0 };
        }
    }
}
```

5. **Optimisez les images** - Redimensionnez les images à la taille nécessaire avant de les inclure dans votre application:

```csharp
// Définir la taille maximale des images
Image profileImage = new Image
{
    HeightRequest = 100, // Définir la hauteur exacte
    WidthRequest = 100,  // Définir la largeur exacte
    Aspect = Aspect.AspectFill, // Maintenir le ratio
    Source = "profile_image.jpg"
};

// Utiliser des caches d'images
// Installez le package NuGet FFImageLoading.Maui
CachedImage cachedImage = new CachedImage
{
    Source = "https://example.com/large_image.jpg",
    CacheDuration = TimeSpan.FromDays(7),
    DownsampleToViewSize = true, // Réduit l'image à la taille affichée
    RetryCount = 3,
    RetryDelay = 250
};
```

6. **Utilisez l'AOT (Ahead of Time)** - Activez la compilation AOT pour améliorer les performances au démarrage:

```xml
<PropertyGroup>
    <PublishAot>true</PublishAot>
</PropertyGroup>
```

7. **Minimisez le travail dans le thread UI** - Déplacez les opérations longues vers des threads en arrière-plan:

```csharp
// Mauvais exemple - bloque l'UI
private void OnButtonClicked(object sender, EventArgs e)
{
    // Opération longue directement dans le thread UI
    var result = ProcessLargeDataSet();
    DisplayResults(result);
}

// Bon exemple - utilise un thread en arrière-plan
private async void OnButtonClicked(object sender, EventArgs e)
{
    // Afficher un indicateur de chargement
    loadingIndicator.IsVisible = true;

    // Exécuter l'opération longue sur un thread en arrière-plan
    var result = await Task.Run(() => ProcessLargeDataSet());

    // Mettre à jour l'UI sur le thread principal
    loadingIndicator.IsVisible = false;
    DisplayResults(result);
}
```

8. **Utilisez des animations matérielles** - Activez l'accélération matérielle pour les animations:

```csharp
// Activation de l'accélération matérielle
image.IsVisible = true;
ViewExtensions.CancelAnimations(image);

// Animation efficace avec accélération matérielle
await image.TranslateTo(100, 0, 500, Easing.SpringOut);
```

9. **Adoptez les contrôles optimisés** - Utilisez les contrôles natifs optimisés pour chaque plateforme quand c'est possible:

```csharp
#if ANDROID
// Code spécifique à Android pour performance maximale
#elif IOS
// Code spécifique à iOS pour performance maximale
#endif
```

10. **Réduisez les conversions de type** - Minimisez les conversions de type qui peuvent être coûteuses:

```csharp
// Moins efficace - provoque une conversion
object value = 10;
int number = (int)value;

// Plus efficace - pas de conversion
int directNumber = 10;
```

#### Meilleures pratiques de développement MAUI

1. **Suivez le pattern MVVM** - Séparez clairement la logique d'affichage et la logique métier:

```csharp
public class BaseViewModel : INotifyPropertyChanged
{
    protected bool SetProperty<T>(ref T backingField, T value,
                                [CallerMemberName] string propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(backingField, value))
            return false;

        backingField = value;
        OnPropertyChanged(propertyName);
        return true;
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

2. **Utilisez l'injection de dépendances** - Enregistrez et résolvez les services via DI:

```csharp
// Dans MauiProgram.cs
builder.Services.AddSingleton<IDataService, DataService>();
builder.Services.AddSingleton<MainViewModel>();
builder.Services.AddSingleton<MainPage>();

// Dans le constructeur de la page
public MainPage(MainViewModel viewModel)
{
    InitializeComponent();
    BindingContext = viewModel;
}
```

3. **Implémentez la navigation Shell** - Utilisez Shell pour une navigation optimisée:

```csharp
// Enregistrement des routes
public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();

        Routing.RegisterRoute(nameof(ItemDetailPage), typeof(ItemDetailPage));
        Routing.RegisterRoute(nameof(ProfilePage), typeof(ProfilePage));
    }
}

// Navigation vers ces routes
await Shell.Current.GoToAsync($"{nameof(ItemDetailPage)}?id={itemId}");
```

4. **Utilisez des ressources partagées** - Centralisez les styles et ressources:

```xml
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyMauiApp.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>

            <!-- Couleurs globales -->
            <Color x:Key="PrimaryColor">#512BD4</Color>
            <Color x:Key="SecondaryColor">#DFD8F7</Color>

            <!-- Styles de base -->
            <Style TargetType="Label" x:Key="BaseLabel">
                <Setter Property="FontFamily" Value="OpenSansRegular" />
                <Setter Property="TextColor" Value="{StaticResource TextPrimary}" />
            </Style>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

5. **Utilisez les handlers personnalisés** - Personnalisez l'apparence des contrôles natifs:

```csharp
// Dans MauiProgram.cs
builder.ConfigureMauiHandlers(handlers =>
{
    handlers.AddHandler<Entry, CustomEntryHandler>();
});

// Classe de handler personnalisé
public class CustomEntryHandler : EntryHandler
{
    protected override void ConnectHandler(MauiControls.Entry platformView)
    {
        base.ConnectHandler(platformView);

        // Personnalisation spécifique à Android
        #if ANDROID
        platformView.SetBackgroundColor(Android.Graphics.Color.Transparent);
        #endif

        // Personnalisation spécifique à iOS
        #if IOS
        platformView.BorderStyle = UIKit.UITextBorderStyle.None;
        #endif
    }
}
```

6. **Utilisez les compiled bindings** - Activez les liaisons compilées pour de meilleures performances:

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MyApp.ViewModels"
             x:Class="MyApp.Views.MainPage"
             x:DataType="local:MainViewModel">

    <!-- Les liaisons sont compilées, pas résoluées dynamiquement -->
    <Label Text="{Binding Name}" />
</ContentPage>
```

7. **Appliquez les bonnes pratiques d'accessibilité** - Rendez votre application accessible à tous:

```xml
<Image Source="logo.png"
       SemanticProperties.Description="Logo de l'application"
       SemanticProperties.Hint="Image représentant le logo de l'entreprise" />

<Button Text="Connexion"
        SemanticProperties.Hint="Se connecter à votre compte"
        AutomationId="LoginButton" />
```

8. **Utilisez des modèles de pages réutilisables** - Créez des templates de pages communs:

```xml
<!-- BasePage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.Templates.BasePage">
    <Grid RowDefinitions="Auto,*,Auto">
        <!-- En-tête commun -->
        <Grid Grid.Row="0" BackgroundColor="{StaticResource PrimaryColor}">
            <Label x:Name="TitleLabel"
                   TextColor="White"
                   FontSize="20"
                   HorizontalOptions="Center" />
        </Grid>

        <!-- Contenu de la page (à remplacer) -->
        <ContentView Grid.Row="1" x:Name="ContentContainer" />

        <!-- Pied de page commun -->
        <Grid Grid.Row="2" BackgroundColor="{StaticResource SecondaryColor}">
            <Label Text="© 2025 Mon Application"
                   HorizontalOptions="Center" />
        </Grid>
    </Grid>
</ContentPage>
```

```csharp
// BasePage.xaml.cs
public partial class BasePage : ContentPage
{
    public static readonly BindableProperty TitleTextProperty =
        BindableProperty.Create(nameof(TitleText), typeof(string), typeof(BasePage), string.Empty,
            propertyChanged: (bindable, oldValue, newValue) =>
            {
                ((BasePage)bindable).TitleLabel.Text = (string)newValue;
            });

    public static readonly BindableProperty ContentProperty =
        BindableProperty.Create(nameof(Content), typeof(View), typeof(BasePage), null,
            propertyChanged: (bindable, oldValue, newValue) =>
            {
                ((BasePage)bindable).ContentContainer.Content = (View)newValue;
            });

    public string TitleText
    {
        get => (string)GetValue(TitleTextProperty);
        set => SetValue(TitleTextProperty, value);
    }

    public new View Content
    {
        get => (View)GetValue(ContentProperty);
        set => SetValue(ContentProperty, value);
    }

    public BasePage()
    {
        InitializeComponent();
    }
}
```

9. **Implémentez le mode sombre** - Adaptez votre application aux préférences des utilisateurs:

```xml
<!-- AppTheme permet de définir des valeurs différentes selon le thème -->
<Style TargetType="Label">
    <Setter Property="TextColor"
            Value="{AppTheme Light=Black, Dark=White}" />
</Style>

<!-- Ou définir des ressources par thème -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <Color x:Key="BackgroundColor">White</Color>
                <Color x:Key="TextColor">Black</Color>
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <Color x:Key="BackgroundColor">#121212</Color>
                <Color x:Key="TextColor">White</Color>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

10. **Utilisez GenericAnimations** - Créez des animations réutilisables:

```csharp
public static class AnimationExtensions
{
    public static Task<bool> FadeInAsync(this VisualElement element, uint duration = 250, Easing easing = null)
    {
        element.Opacity = 0;
        return element.FadeTo(1, duration, easing);
    }

    public static Task<bool> FadeOutAsync(this VisualElement element, uint duration = 250, Easing easing = null)
    {
        return element.FadeTo(0, duration, easing);
    }

    public static async Task<bool> ShakeAsync(this VisualElement element, uint duration = 500)
    {
        var tasks = new List<Task>();
        var originalX = element.TranslationX;

        tasks.Add(element.TranslateTo(originalX - 15, element.TranslationY, duration / 5));
        tasks.Add(element.TranslateTo(originalX + 15, element.TranslationY, duration / 5));
        tasks.Add(element.TranslateTo(originalX - 10, element.TranslationY, duration / 5));
        tasks.Add(element.TranslateTo(originalX + 10, element.TranslationY, duration / 5));
        tasks.Add(element.TranslateTo(originalX, element.TranslationY, duration / 5));

        await Task.WhenAll(tasks);
        return true;
    }
}

// Utilisation
await myButton.FadeInAsync();
await errorLabel.ShakeAsync();
```

### Utilisation multiplateforme efficace

1. **Adaptez l'interface selon l'appareil** - Utilisez `DeviceIdiom` pour adapter l'UI:

```csharp
if (DeviceInfo.Current.Idiom == DeviceIdiom.Phone)
{
    // Configuration pour téléphone
    mainGrid.ColumnDefinitions = new ColumnDefinitionCollection { new ColumnDefinition() };
    Grid.SetRow(detailsPanel, 1);
    Grid.SetColumn(detailsPanel, 0);
}
else if (DeviceInfo.Current.Idiom == DeviceIdiom.Tablet ||
         DeviceInfo.Current.Idiom == DeviceIdiom.Desktop)
{
    // Configuration pour tablette/bureau
    mainGrid.ColumnDefinitions = new ColumnDefinitionCollection
    {
        new ColumnDefinition(new GridLength(1, GridUnitType.Star)),
        new ColumnDefinition(new GridLength(2, GridUnitType.Star))
    };
    Grid.SetRow(detailsPanel, 0);
    Grid.SetColumn(detailsPanel, 1);
}
```

2. **Créez des interfaces adaptatives avec XAML** - Utilisez `OnIdiom` et `OnPlatform`:

```xml
<StackLayout Spacing="{OnIdiom Phone=10, Tablet=20, Desktop=30}">
    <Label Text="Titre"
           FontSize="{OnPlatform Android=24, iOS=28, WinUI=26}" />

    <Grid ColumnDefinitions="{OnIdiom Phone='*',
                                     Default='*,*'}">
        <!-- Contenu adaptatif -->
    </Grid>
</StackLayout>
```

3. **Utilisez les ressources spécifiques aux plateformes** - Organisez vos ressources par plateforme:

```
Resources/
├── Images/
│   ├── logo.png               # Image partagée
│   └── logo@2x.png            # Version haute résolution
├── Images/android/
│   └── logo.png               # Version Android
├── Images/ios/
│   └── logo.png               # Version iOS
└── Images/windows/
    └── logo.png               # Version Windows
```

4. **Définissez des configurations par plateforme** - Utilisez des configurations conditionnelles dans le fichier projet:

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-android'))">
    <AndroidSigningKeyStore>myapp.keystore</AndroidSigningKeyStore>
    <AndroidSigningKeyAlias>key</AndroidSigningKeyAlias>
</PropertyGroup>

<PropertyGroup Condition="$(TargetFramework.Contains('-ios'))">
    <CodesignKey>Apple Development: John Doe</CodesignKey>
    <CodesignProvision>MyApp Development</CodesignProvision>
</PropertyGroup>
```

### Débogage et test

1. **Utilisez Hot Reload** - Accélérez votre développement avec Hot Reload:

```csharp
// 1. Démarrez votre application en mode debug
// 2. Modifiez le XAML ou le code C#
// 3. Enregistrez le fichier pour voir les changements en direct
```

2. **Déboguer les liaisons** - Utilisez les outils de diagnostic pour les liaisons:

```xml
<Label Text="{Binding Name, DebugString=true}" />
```

3. **Journalisation structurée** - Utilisez un système de journalisation robuste:

```csharp
// Dans MauiProgram.cs
builder.Logging.AddDebug();

// Dans le code
private readonly ILogger<MainPage> _logger;

public MainPage(ILogger<MainPage> logger)
{
    _logger = logger;
    InitializeComponent();

    _logger.LogInformation("Page initialisée");
}

// Différents niveaux de journalisation
_logger.LogTrace("Message de trace détaillé");
_logger.LogDebug("Message de débogage");
_logger.LogInformation("Information générale");
_logger.LogWarning("Avertissement");
_logger.LogError("Erreur");
_logger.LogCritical("Erreur critique");
```

4. **Tests unitaires** - Créez des tests unitaires pour votre logique métier:

```csharp
// Installer les packages NuGet:
// - xunit
// - Moq (pour le mocking)

[Fact]
public async Task LoginCommand_WithValidCredentials_ShouldSucceed()
{
    // Arrange
    var authServiceMock = new Mock<IAuthService>();
    authServiceMock
        .Setup(x => x.LoginAsync(It.IsAny<string>(), It.IsAny<string>()))
        .ReturnsAsync(new AuthResult { Success = true });

    var viewModel = new LoginViewModel(authServiceMock.Object);
    viewModel.Username = "test@example.com";
    viewModel.Password = "password123";

    // Act
    await viewModel.LoginCommand.ExecuteAsync(null);

    // Assert
    Assert.False(viewModel.HasError);
    authServiceMock.Verify(x => x.LoginAsync("test@example.com", "password123"), Times.Once);
}
```

5. **Tests d'UI automatisés** - Utilisez Xamarin.UITest (compatible avec MAUI):

```csharp
[Test]
public void Login_WithValidCredentials_ShouldNavigateToMainPage()
{
    app.EnterText("UsernameEntry", "test@example.com");
    app.EnterText("PasswordEntry", "password123");
    app.Tap("LoginButton");

    // Vérifier que la page principale est affichée
    app.WaitForElement("WelcomeLabel");
    Assert.IsTrue(app.Query("WelcomeLabel").Any());
}
```

### Ressources et documentation

Pour approfondir vos connaissances sur MAUI, voici quelques ressources essentielles:

1. **Documentation officielle** - Microsoft Learn MAUI Documentation
2. **Dépôt GitHub** - https://github.com/dotnet/maui
3. **Communauté** - .NET MAUI Community Toolkit
4. **Forums** - Stack Overflow avec le tag [maui]
5. **Chaînes YouTube** - .NET MAUI & Xamarin channels for tutorials
6. **Blogs** - Microsoft DevBlogs, community developer blogs

---

Ce guide complet sur .NET MAUI vous a présenté les concepts fondamentaux et avancés pour développer des applications multiplateformes modernes avec C# et XAML. En suivant ces bonnes pratiques, vous pourrez créer des applications performantes, maintenables et offrant une excellente expérience utilisateur sur toutes les plateformes prises en charge.

⏭️ 13.3. [WinUI 3](/13-applications-de-bureau-modernes-avec-dotnet/13-3-winui-3.md)
