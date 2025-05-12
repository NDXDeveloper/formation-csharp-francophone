# 8.1. Introduction à WPF

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Windows Presentation Foundation (WPF) représente une évolution majeure dans le développement d'interfaces utilisateur pour les applications Windows. Lancé avec .NET Framework 3.0 en 2006, WPF offre une approche moderne pour créer des interfaces riches et interactives.

## 8.1.1. Comparaison avec WinForms

WPF et Windows Forms sont deux technologies de Microsoft pour créer des applications de bureau, mais elles diffèrent fondamentalement dans leur conception et leurs capacités.

### Différences architecturales

```textmate
// WinForms (.NET Framework 4.7.2)
public partial class SimpleWinFormsApp : Form
{
    public SimpleWinFormsApp()
    {
        InitializeComponent();

        // Code du concepteur généré manuellement ou par le designer
        this.Text = "Application WinForms";
        this.Size = new Size(400, 300);

        Button button = new Button();
        button.Text = "Cliquez-moi";
        button.Location = new Point(150, 120);
        button.Click += Button_Click;

        this.Controls.Add(button);
    }

    private void Button_Click(object sender, EventArgs e)
    {
        MessageBox.Show("Bonjour depuis WinForms!");
    }
}

// WPF (.NET Framework 4.7.2 ou .NET 8)
// MainWindow.xaml
<Window x:Class="SimpleWpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Application WPF" Height="300" Width="400">
    <Grid>
        <Button Content="Cliquez-moi" HorizontalAlignment="Center"
                VerticalAlignment="Center" Click="Button_Click"/>
    </Grid>
</Window>

// MainWindow.xaml.cs
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
        MessageBox.Show("Bonjour depuis WPF!");
    }
}
```


### Tableau comparatif WinForms vs WPF

| Aspect | Windows Forms | WPF |
|--------|---------------|-----|
| **Modèle de programmation** | Principalement code (imperatif) | Séparation entre XAML (déclaratif) et code |
| **Rendu graphique** | GDI/GDI+ (basé sur le CPU) | DirectX (utilisation du GPU) |
| **Layout** | Positionnement absolu | Layouts dynamiques et adaptatifs |
| **Personnalisation** | Limitée, basée sur les contrôles | Styles, templates, et binding de données avancés |
| **Animation** | Basique, généralement via Timer | Système complet d'animation intégré |
| **Vecteurs graphiques** | Support limité | Support natif SVG et graphiques vectoriels |
| **Évolutivité** | Modeste | Haute (via le binding, MVVM, etc.) |
| **Courbe d'apprentissage** | Relativement douce | Plus abrupte, concepts plus nombreux |
| **3D** | Non supporté nativement | Support natif pour la 3D |
| **Support multi-écrans** | Basique | Avancé |

### Avantages de WPF par rapport à WinForms

1. **Capacités graphiques supérieures** grâce au rendu DirectX
2. **Séparation claire** entre la présentation (XAML) et la logique (code-behind)
3. **Personnalisation avancée** via les styles et les templates
4. **Data binding** puissant et flexible
5. **Support de l'architecture MVVM** (Model-View-ViewModel)
6. **Animations fluides** intégrées au framework
7. **Indépendance de la résolution** grâce aux unités de mesure indépendantes des pixels
8. **Contrôles composites** faciles à créer
9. **Support multimédia** amélioré
10. **Internationalisation** et accessibilité avancées

### Avantages de WinForms par rapport à WPF

1. **Simplicité et familiarité** pour les développeurs venant du monde Windows
2. **Designer visuel plus intuitif** pour les débutants
3. **Performance potentiellement meilleure** pour les applications simples
4. **Moins de ressources consommées** pour des applications basiques
5. **Démarrage plus rapide** des applications
6. **Courbe d'apprentissage plus douce**
7. **Base de code existante** et communauté mature
8. **Contrôles de tierce partie** nombreux et bien établis

## 8.1.2. Architecture de WPF

WPF est construit sur une architecture en couches qui sépare distinctement les responsabilités et qui offre une grande flexibilité.

### Les couches fondamentales de WPF

```
┌─────────────────────────────────────────────┐
│            APPLICATIONS WPF                 │
├─────────────────────────────────────────────┤
│    CONTRÔLES ET ÉLÉMENTS                    │
│  (Buttons, TextBox, Canvas, Grid, etc.)     │
├─────────────────────────────────────────────┤
│             FRAMEWORK                       │
│  (Layout, Styling, Binding, Animation)      │
├─────────────────────────────────────────────┤
│               MEDIA                         │
│        (2D, 3D, Text, Imaging)              │
├─────────────────────────────────────────────┤
│            COMPOSITION                      │
│          (milcore.dll)                      │
├─────────────────────────────────────────────┤
│            DIRECTX / USER32                 │
└─────────────────────────────────────────────┘
```


### La distinction entre les espaces de noms clés

```textmate
// .NET Framework 4.7.2 et .NET 8
// Importation des espaces de noms essentiels WPF
using System.Windows;                // Éléments de base de WPF (Window, Application, etc.)
using System.Windows.Controls;       // Contrôles UI (Button, TextBox, etc.)
using System.Windows.Media;          // Graphiques et rendu (Brushes, Colors, etc.)
using System.Windows.Media.Animation; // Animations
using System.Windows.Shapes;         // Formes géométriques (Rectangle, Ellipse, etc.)
using System.Windows.Documents;      // Documents (TextBlock avancé, FlowDocument, etc.)
using System.Windows.Input;          // Gestion des entrées (souris, clavier, etc.)
using System.Windows.Navigation;     // Navigation entre pages
using System.Windows.Data;           // Liaison de données (Binding, etc.)
```


### Le modèle de programmation XAML

XAML (eXtensible Application Markup Language) est un langage déclaratif basé sur XML qui est au cœur de WPF. Il permet de séparer l'interface utilisateur de la logique applicative.

```xml
<!-- Exemple XAML montrant différents aspects -->
<Window x:Class="WpfArchitectureDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:WpfArchitectureDemo"
        Title="Architecture WPF" Height="450" Width="800">

    <!-- Ressources au niveau de la fenêtre -->
    <Window.Resources>
        <!-- Style pour les boutons -->
        <Style TargetType="Button">
            <Setter Property="Margin" Value="5"/>
            <Setter Property="Padding" Value="10,5"/>
            <Setter Property="Background" Value="#3498db"/>
            <Setter Property="Foreground" Value="White"/>
        </Style>

        <!-- Animation pour l'élément rectangle -->
        <Storyboard x:Key="colorAnimation">
            <ColorAnimation
                Storyboard.TargetName="animatedRectangle"
                Storyboard.TargetProperty="(Rectangle.Fill).(SolidColorBrush.Color)"
                From="Blue" To="Red" Duration="0:0:1" AutoReverse="True"
                RepeatBehavior="Forever"/>
        </Storyboard>
    </Window.Resources>

    <!-- Layout principal avec Grid -->
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- En-tête -->
        <TextBlock Grid.Row="0" Text="Architecture WPF"
                   FontSize="24" Margin="10"
                   HorizontalAlignment="Center"/>

        <!-- Contenu principal -->
        <Grid Grid.Row="1">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="1*"/>
                <ColumnDefinition Width="1*"/>
            </Grid.ColumnDefinitions>

            <!-- Panel gauche -->
            <StackPanel Grid.Column="0" Margin="10">
                <TextBlock Text="Éléments UI" FontWeight="Bold" Margin="0,0,0,10"/>

                <!-- Contrôles WPF communs -->
                <Button Content="Bouton standard" Margin="0,5"/>
                <TextBox Text="Zone de texte" Margin="0,5"/>
                <CheckBox Content="Case à cocher" IsChecked="True" Margin="0,5"/>
                <ComboBox Margin="0,5">
                    <ComboBoxItem Content="Option 1"/>
                    <ComboBoxItem Content="Option 2"/>
                    <ComboBoxItem Content="Option 3"/>
                </ComboBox>
                <Slider Minimum="0" Maximum="100" Value="50" Margin="0,5"/>

                <!-- Data binding simple -->
                <TextBlock Text="Exemple de Binding:" Margin="0,10,0,5"/>
                <Slider x:Name="valueSlider" Minimum="0" Maximum="100" Value="25"/>
                <TextBlock Text="{Binding Value, ElementName=valueSlider, StringFormat=Valeur: {0:F1}}"
                           Margin="0,5"/>
            </StackPanel>

            <!-- Panel droit - Démonstrations graphiques -->
            <Canvas Grid.Column="1" Margin="10">
                <TextBlock Canvas.Left="10" Canvas.Top="10"
                           Text="Capabilities graphiques" FontWeight="Bold"/>

                <!-- Formes vectorielles -->
                <Rectangle Canvas.Left="20" Canvas.Top="40"
                           Width="100" Height="60"
                           Fill="Green" Stroke="DarkGreen" StrokeThickness="2"/>

                <Ellipse Canvas.Left="140" Canvas.Top="40"
                         Width="100" Height="60"
                         Fill="Orange" Stroke="DarkOrange" StrokeThickness="2"/>

                <!-- Effets graphiques -->
                <Border Canvas.Left="20" Canvas.Top="120"
                        Width="100" Height="60"
                        Background="Purple" CornerRadius="10">
                    <Border.Effect>
                        <DropShadowEffect ShadowDepth="5" Opacity="0.7"/>
                    </Border.Effect>
                </Border>

                <!-- Gradients -->
                <Rectangle Canvas.Left="140" Canvas.Top="120"
                           Width="100" Height="60">
                    <Rectangle.Fill>
                        <LinearGradientBrush StartPoint="0,0" EndPoint="1,1">
                            <GradientStop Color="Yellow" Offset="0"/>
                            <GradientStop Color="Red" Offset="1"/>
                        </LinearGradientBrush>
                    </Rectangle.Fill>
                </Rectangle>

                <!-- Rectangle animé -->
                <Rectangle x:Name="animatedRectangle"
                           Canvas.Left="80" Canvas.Top="200"
                           Width="100" Height="60"
                           Fill="Blue" Stroke="Navy" StrokeThickness="2">
                    <Rectangle.Triggers>
                        <EventTrigger RoutedEvent="Rectangle.Loaded">
                            <BeginStoryboard Storyboard="{StaticResource colorAnimation}"/>
                        </EventTrigger>
                    </Rectangle.Triggers>
                </Rectangle>
            </Canvas>
        </Grid>

        <!-- Pied de page -->
        <StackPanel Grid.Row="2" Orientation="Horizontal"
                    HorizontalAlignment="Center" Margin="10">
            <Button Content="Action 1" Click="Button_Click"/>
            <Button Content="Action 2" Click="Button_Click"/>
        </StackPanel>
    </Grid>
</Window>
```


```textmate
// Code-behind (.NET Framework 4.7.2 et .NET 8)
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
        Button button = sender as Button;
        MessageBox.Show($"Vous avez cliqué sur: {button?.Content}");
    }
}
```


### Le système d'événements routés

WPF utilise un système avancé d'événements qui peuvent se propager à travers l'arbre visuel selon trois modes :

1. **Événements directs** : Similaires aux événements classiques .NET
2. **Événements tunneling** : Se propagent du haut vers le bas (de la racine vers l'élément cible)
3. **Événements bubbling** : Se propagent du bas vers le haut (de l'élément cible vers la racine)

```textmate
// .NET Framework 4.7.2 et .NET 8
private void DemonstrationEvenementsRoutes()
{
    // Création d'une fenêtre de démonstration
    Window demoWindow = new Window
    {
        Title = "Démonstration des événements routés",
        Width = 500,
        Height = 400,
        WindowStartupLocation = WindowStartupLocation.CenterScreen
    };

    // Création d'une grille principale
    Grid mainGrid = new Grid
    {
        Margin = new Thickness(20)
    };

    // Création d'un StackPanel
    StackPanel panel = new StackPanel
    {
        Background = Brushes.LightBlue,
        Margin = new Thickness(10)
    };

    // Création d'un bouton
    Button button = new Button
    {
        Content = "Cliquez-moi",
        Padding = new Thickness(10, 5),
        Margin = new Thickness(20),
        HorizontalAlignment = HorizontalAlignment.Center
    };

    // Création d'une zone de texte pour les logs
    TextBox logTextBox = new TextBox
    {
        Margin = new Thickness(10),
        Height = 200,
        IsReadOnly = true,
        VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
        TextWrapping = TextWrapping.Wrap
    };

    // Ajout des gestionnaires d'événements pour la fenêtre
    demoWindow.PreviewMouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"TUNNELING: PreviewMouseDown sur Window\n");
    };

    demoWindow.MouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"BUBBLING: MouseDown sur Window\n");
    };

    // Ajout des gestionnaires d'événements pour la grille
    mainGrid.PreviewMouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"TUNNELING: PreviewMouseDown sur Grid\n");
    };

    mainGrid.MouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"BUBBLING: MouseDown sur Grid\n");
    };

    // Ajout des gestionnaires d'événements pour le panel
    panel.PreviewMouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"TUNNELING: PreviewMouseDown sur StackPanel\n");
    };

    panel.MouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"BUBBLING: MouseDown sur StackPanel\n");
    };

    // Ajout des gestionnaires d'événements pour le bouton
    button.PreviewMouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"TUNNELING: PreviewMouseDown sur Button\n");
    };

    button.MouseDown += (sender, e) =>
    {
        logTextBox.AppendText($"BUBBLING: MouseDown sur Button\n");

        // Pour démontrer comment arrêter la propagation:
        // e.Handled = true;
    };

    // Bouton pour effacer les logs
    Button clearButton = new Button
    {
        Content = "Effacer les logs",
        Margin = new Thickness(10),
        Padding = new Thickness(5),
        HorizontalAlignment = HorizontalAlignment.Right
    };

    clearButton.Click += (sender, e) =>
    {
        logTextBox.Clear();
    };

    // Construction de la hiérarchie des contrôles
    panel.Children.Add(button);
    mainGrid.Children.Add(panel);
    mainGrid.Children.Add(logTextBox);
    Grid.SetRow(logTextBox, 1);
    mainGrid.Children.Add(clearButton);
    Grid.SetRow(clearButton, 2);

    // Configuration des lignes de la grille
    mainGrid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto });
    mainGrid.RowDefinitions.Add(new RowDefinition { Height = new GridLength(1, GridUnitType.Star) });
    mainGrid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto });

    // Définition du contenu de la fenêtre
    demoWindow.Content = mainGrid;

    // Affichage de la fenêtre
    demoWindow.ShowDialog();
}
```


### Dependency Properties

Les Dependency Properties sont un système avancé de propriétés qui étend les propriétés CLR classiques avec des fonctionnalités supplémentaires comme l'héritage de valeurs, les animations, et le data binding.

```textmate
// .NET Framework 4.7.2 et .NET 8
// Exemple de définition d'une Dependency Property
public class CustomControl : Control
{
    // 1. Déclaration de la Dependency Property avec un identifiant statique
    public static readonly DependencyProperty CustomTextProperty =
        DependencyProperty.Register(
            // Nom de la propriété
            "CustomText",
            // Type de la propriété
            typeof(string),
            // Type propriétaire
            typeof(CustomControl),
            // Métadonnées (valeur par défaut, callbacks, etc.)
            new PropertyMetadata(
                "Valeur par défaut",
                // Callback appelé quand la valeur change
                (d, e) => ((CustomControl)d).OnCustomTextChanged(e.OldValue, e.NewValue)
            )
        );

    // 2. Wrapper CLR pour la Dependency Property
    public string CustomText
    {
        get { return (string)GetValue(CustomTextProperty); }
        set { SetValue(CustomTextProperty, value); }
    }

    // 3. Méthode appelée lorsque la valeur change
    protected virtual void OnCustomTextChanged(object oldValue, object newValue)
    {
        // Logique à exécuter quand la propriété change
        // Par exemple: mise à jour de l'apparence visuelle
    }

    // Méthode pour valider la valeur avant de l'affecter
    private static bool ValidateCustomText(object value)
    {
        string text = value as string;
        if (string.IsNullOrEmpty(text))
            return false;
        return true;
    }

    // Méthode pour calculer une valeur si non spécifiée
    private static object CoerceCustomText(DependencyObject d, object baseValue)
    {
        string value = (string)baseValue;
        if (value.Length > 50)
            return value.Substring(0, 50);
        return baseValue;
    }
}
```


### Arborescence visuelle et logique

WPF utilise deux types d'arborescences pour organiser les éléments de l'interface:

1. **Arborescence visuelle** : Représente la structure de rendu de l'interface
2. **Arborescence logique** : Représente la structure des contrôles et leur relation parent-enfant

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExplorerArborescences()
{
    // Créer une interface simple
    Button button = new Button
    {
        Content = "Explorer les arborescences"
    };

    button.Click += (sender, e) =>
    {
        StringBuilder sb = new StringBuilder();

        // Récupérer le parent logique
        DependencyObject parent = LogicalTreeHelper.GetParent(button);
        sb.AppendLine($"Parent logique: {parent?.GetType().Name ?? "Aucun"}");

        // Récupérer le parent visuel
        DependencyObject visualParent = VisualTreeHelper.GetParent(button);
        sb.AppendLine($"Parent visuel: {visualParent?.GetType().Name ?? "Aucun"}");

        // Explorer l'arborescence visuelle d'un bouton
        sb.AppendLine("\nArborescence visuelle d'un Button:");
        ExplorerArborescenceVisuelle(button, sb, 0);

        // Afficher le résultat
        MessageBox.Show(sb.ToString(), "Exploration des arborescences");
    };

    // Créer une fenêtre pour afficher le bouton
    Window window = new Window
    {
        Title = "Démonstration des arborescences",
        Width = 300,
        Height = 200,
        Content = button
    };

    window.ShowDialog();
}

private void ExplorerArborescenceVisuelle(DependencyObject element, StringBuilder sb, int niveau)
{
    if (element == null) return;

    // Indentation selon le niveau
    string indent = new string(' ', niveau * 2);

    // Afficher le type de l'élément
    sb.AppendLine($"{indent}+ {element.GetType().Name}");

    // Explorer les enfants
    int count = VisualTreeHelper.GetChildrenCount(element);
    for (int i = 0; i < count; i++)
    {
        DependencyObject child = VisualTreeHelper.GetChild(element, i);
        ExplorerArborescenceVisuelle(child, sb, niveau + 1);
    }
}
```


## 8.1.3. WPF avec .NET Core/.NET 5+

Depuis .NET Core 3.0 (et par extension .NET 5 et versions ultérieures), WPF est disponible en dehors du .NET Framework traditionnel. Cette évolution apporte plusieurs avantages et quelques différences notables.

### Migration de WPF vers .NET Core/.NET

```xml
<!-- Ancien fichier projet WPF (.NET Framework 4.7.2) -->
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{...}</ProjectGuid>
    <OutputType>WinExe</OutputType>
    <RootNamespace>WpfFrameworkApp</RootNamespace>
    <AssemblyName>WpfFrameworkApp</AssemblyName>
    <TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>
    <FileAlignment>512</FileAlignment>
    <ProjectTypeGuids>{60dc8134-eba5-43b8-bcc9-bb4bc16c2548};{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}</ProjectTypeGuids>
    <WarningLevel>4</WarningLevel>
    <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
    <Deterministic>true</Deterministic>
  </PropertyGroup>
  <!-- ... autres propriétés et références ... -->
</Project>

<!-- Nouveau fichier projet WPF (.NET 8) -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```


### Avantages de WPF avec .NET 5+ par rapport à .NET Framework

1. **Performance améliorée** : Optimisations du compilateur et du runtime
2. **Déploiement plus flexible** : Options de déploiement autonome et trimming
3. **Accès aux nouveautés de C#** : C# 9, 10, 11, 12 avec leurs fonctionnalités modernes
4. **Développement cross-plateforme** : Développement possible sur macOS et Linux avec Visual Studio Code
5. **Cycle de vie moderne** : Mises à jour plus fréquentes et support actif
6. **Intégration des packages NuGet** : Gestion des dépendances simplifiée
7. **Support des fonctionnalités modernes** : Comme les records, les types nullables, et les nouveaux types génériques

### Limites et différences

1. **Contrôles COM manquants** : Certains contrôles comme WebBrowser basé sur IE ne sont pas disponibles
2. **Différences de comportement subtiles** : Certaines API se comportent légèrement différemment
3. **Certaines API liées à Windows ne sont pas disponibles** ou ont été remplacées
4. **Code natif** : L'interopérabilité avec le code natif nécessite une approche différente
5. **Absence de certains types de projets** : Comme les add-ins WPF ou les custom controls libraries

### Exemple d'application WPF moderne avec .NET 8

```textmate
// Program.cs (.NET 8)
using System.Windows;

namespace ModernWpfApp;

public class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        var app = new Application();
        app.StartupUri = new Uri("MainWindow.xaml", UriKind.Relative);
        app.Run();
    }
}
```


```xml
<!-- MainWindow.xaml (.NET 8) -->
<Window x:Class="ModernWpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ModernWpfApp"
        mc:Ignorable="d"
        Title="Application WPF moderne" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <StackPanel Grid.Row="0" Margin="10">
            <TextBlock Text="WPF avec .NET 8"
                       FontSize="24"
                       HorizontalAlignment="Center"/>
            <TextBlock Text="Démonstration des fonctionnalités modernes"
                       FontSize="14"
                       HorizontalAlignment="Center"
                       Margin="0,5,0,15"/>
        </StackPanel>

        <TabControl Grid.Row="1" Margin="10">
            <TabItem Header="Nouvelles fonctionnalités C#">
                <ScrollViewer>
                    <StackPanel Margin="10">
                        <TextBlock Text="Exemples de C# moderne dans WPF"
                                   FontWeight="Bold"
                                   Margin="0,0,0,10"/>

                        <Button Content="Tester les fonctionnalités C# modernes"
                                Click="BtnModernCSharp_Click"
                                Padding="10,5"
                                Margin="0,5"/>

                        <TextBox x:Name="txtCSharpOutput"
                                 Height="200"
                                 IsReadOnly="True"
                                 TextWrapping="Wrap"
                                 VerticalScrollBarVisibility="Auto"
                                 Margin="0,10"/>
                    </StackPanel>
                </ScrollViewer>
            </TabItem>

            <TabItem Header="Nouvelles API">
                <Grid Margin="10">
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="*"/>
                    </Grid.RowDefinitions>

                    <StackPanel Grid.Row="0">
                        <TextBlock Text="Exemples d'API modernes"
                                   FontWeight="Bold"
                                   Margin="0,0,0,10"/>

                        <WrapPanel>
                            <Button Content="HTTP Client"
                                    Click="BtnHttpClient_Click"
                                    Margin="5" Padding="10,5"/>

                            <Button Content="JSON"
                                    Click="BtnJson_Click"
                                    Margin="5" Padding="10,5"/>

                            <Button Content="File System"
                                    Click="BtnFileSystem_Click"
                                    Margin="5" Padding="10,5"/>
                        </WrapPanel>
                    </StackPanel>

                    <TextBox Grid.Row="1"
                             x:Name="txtApiOutput"
                             IsReadOnly="True"
                             TextWrapping="Wrap"
                             VerticalScrollBarVisibility="Auto"
                             Margin="0,10"/>
                </Grid>
            </TabItem>

            <TabItem Header="Performance">
                <StackPanel Margin="10">
                    <TextBlock Text="Démonstration de performance"
                               FontWeight="Bold"
                               Margin="0,0,0,10"/>

                    <Button Content="Générer 10 000 éléments"
                            Click="BtnPerformance_Click"
                            Padding="10,5"
                            Margin="0,5"/>

                    <TextBlock x:Name="txtPerformanceResult"
                               Margin="0,10"/>

                    <Border BorderBrush="LightGray"
                            BorderThickness="1"
                            Margin="0,10">
                        <ScrollViewer MaxHeight="200">
                            <ItemsControl x:Name="performanceItemsControl">
                                <ItemsControl.ItemTemplate>
                                    <DataTemplate>
                                        <Border Margin="2"
                                                Padding="5"
                                                BorderBrush="LightGray"
                                                BorderThickness="1">
                                            <TextBlock Text="{Binding}"/>
                                        </Border>
                                    </DataTemplate>
                                </ItemsControl.ItemTemplate>
                                <ItemsControl.ItemsPanel>
                                    <ItemsPanelTemplate>
                                        <WrapPanel/>
                                    </ItemsPanelTemplate>
                                </ItemsControl.ItemsPanel>
                            </ItemsControl>
                        </ScrollViewer>
                    </Border>
                </StackPanel>
            </TabItem>
        </TabControl>

        <StatusBar Grid.Row="2">
            <StatusBarItem>
                <TextBlock x:Name="statusText" Text="Prêt"/>
            </StatusBarItem>
        </StatusBar>
    </Grid>
</Window>
```


```textmate
// MainWindow.xaml.cs (.NET 8)
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;

namespace ModernWpfApp
{
    public partial class MainWindow : Window
    {
        private readonly HttpClient _httpClient = new();

        public MainWindow()
        {
            InitializeComponent();
        }

        private void BtnModernCSharp_Click(object sender, RoutedEventArgs e)
        {
            StringBuilder output = new();

            // Exemple 1: Records (C# 9+)
            output.AppendLine("=== RECORDS (C# 9+) ===");
            Person person = new("John", "Doe", 30);
            output.AppendLine($"Person: {person}");

            // Déconstruction
            var (firstName, lastName, _) = person;
            output.AppendLine($"Déconstruit: {firstName} {lastName}");

            // With-expression
            Person modifiedPerson = person with { Age = 31 };
            output.AppendLine($"Modifié: {modifiedPerson}");
            output.AppendLine();

            // Exemple 2: Pattern matching (C# 9+)
            output.AppendLine("=== PATTERN MATCHING (C# 9+) ===");
            object[] items = { "hello", 123, true, null, new Person("Alice", "Smith", 25) };

            foreach (var item in items)
            {
                string description = item switch
                {
                    string s => $"Chaîne de longueur {s.Length}",
                    int n when n > 100 => "Grand nombre",
                    int n => $"Petit nombre: {n}",
                    Person { FirstName: "Alice" } p => $"Alice ({p.LastName})",
                    Person p => $"Personne: {p.FirstName}",
                    null => "Valeur null",
                    _ => "Autre type"
                };
                output.AppendLine($"{item?.GetType()?.Name ?? "null"}: {description}");
            }
            output.AppendLine();

            // Exemple 3: Fonctions d'extension sur les collections (C# moderne)
            output.AppendLine("=== COLLECTIONS MODERNES ===");

            List<int> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]; // Collection expressions (C# 12)

            var evenNumbers = numbers.Where(n => n % 2 == 0).ToList();
            output.AppendLine($"Nombres pairs: {string.Join(", ", evenNumbers)}");

            var squares = numbers.Select(n => n * n);
            output.AppendLine($"Carrés: {string.Join(", ", squares)}");

            var sum = numbers.Sum();
            output.AppendLine($"Somme: {sum}");

            txtCSharpOutput.Text = output.ToString();
        }

        private async void BtnHttpClient_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                statusText.Text = "Chargement des données...";

                // Utilisation moderne de HttpClient avec async/await
                var response = await _httpClient.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");

                // Utilisation de System.Text.Json (moderne)
                using var document = JsonDocument.Parse(response);
                var root = document.RootElement;

                StringBuilder output = new();
                output.AppendLine("Données récupérées avec HttpClient moderne:");
                output.AppendLine($"Title: {root.GetProperty("title").GetString()}");
                output.AppendLine($"Body: {root.GetProperty("body").GetString()}");
                output.AppendLine();
                output.AppendLine("JSON brut:");
                output.AppendLine(JsonSerializer.Serialize(document, new JsonSerializerOptions { WriteIndented = true }));

                txtApiOutput.Text = output.ToString();
                statusText.Text = "Données chargées avec succès";
            }
            catch (Exception ex)
            {
                txtApiOutput.Text = $"Erreur: {ex.Message}";
                statusText.Text = "Erreur lors du chargement des données";
            }
        }

        private void BtnJson_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                // Création d'objets pour la sérialisation
                var user = new User
                {
                    Id = 1,
                    Name = "Jean Dupont",
                    Email = "jean.dupont@example.com",
                    IsActive = true,
                    RegisterDate = DateTime.Now,
                    Roles = ["user", "editor"],
                    Address = new Address
                    {
                        Street = "123 Rue Principale",
                        City = "Paris",
                        ZipCode = "75001",
                        Country = "France"
                    }
                };

                // Sérialisation avec System.Text.Json
                var options = new JsonSerializerOptions
                {
                    WriteIndented = true,
                    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
                };

                string json = JsonSerializer.Serialize(user, options);

                // Désérialisation
                var deserializedUser = JsonSerializer.Deserialize<User>(json, options);

                StringBuilder output = new();
                output.AppendLine("=== SÉRIALISATION JSON MODERNE ===");
                output.AppendLine("Objet sérialisé en JSON:");
                output.AppendLine(json);
                output.AppendLine();
                output.AppendLine("Objet désérialisé:");
                output.AppendLine($"Nom: {deserializedUser?.Name}");
                output.AppendLine($"Email: {deserializedUser?.Email}");
                output.AppendLine($"Rôles: {string.Join(", ", deserializedUser?.Roles ?? [])}");
                output.AppendLine($"Adresse: {deserializedUser?.Address?.City}, {deserializedUser?.Address?.Country}");

                txtApiOutput.Text = output.ToString();
                statusText.Text = "Démonstration JSON terminée";
            }
            catch (Exception ex)
            {
                txtApiOutput.Text = $"Erreur: {ex.Message}";
                statusText.Text = "Erreur lors de la démonstration JSON";
            }
        }

        private async void BtnFileSystem_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                // Utilisation des API modernes de système de fichiers
                string tempPath = Path.GetTempPath();
                string demoFolder = Path.Combine(tempPath, "WpfDotNet8Demo");
                string demoFile = Path.Combine(demoFolder, "demo.txt");

                StringBuilder output = new();
                output.AppendLine("=== API MODERNES DE SYSTÈME DE FICHIERS ===");
                output.AppendLine($"Dossier temporaire: {tempPath}");

                // Créer un dossier s'il n'existe pas
                if (!Directory.Exists(demoFolder))
                {
                    Directory.CreateDirectory(demoFolder);
                    output.AppendLine($"Dossier créé: {demoFolder}");
                }
                else
                {
                    output.AppendLine($"Dossier existant: {demoFolder}");
                }

                // Écriture asynchrone dans un fichier
                string content = "Ceci est un test de l'API de fichier moderne.\r\n" +
                                 $"Généré le {DateTime.Now:g}\r\n" +
                                 "Avec .NET 8 et WPF!";

                await File.WriteAllTextAsync(demoFile, content);
                output.AppendLine($"Fichier écrit: {demoFile}");

                // Lecture asynchrone du fichier
                string readContent = await File.ReadAllTextAsync(demoFile);
                output.AppendLine("\nContenu lu du fichier:");
                output.AppendLine(readContent);

                // Informations sur le fichier
                var fileInfo = new FileInfo(demoFile);
                output.AppendLine("\nInformations sur le fichier:");
                output.AppendLine($"Taille: {fileInfo.Length} octets");
                output.AppendLine($"Créé le: {fileInfo.CreationTime:g}");
                output.AppendLine($"Dernière modification: {fileInfo.LastWriteTime:g}");

                txtApiOutput.Text = output.ToString();
                statusText.Text = "Démonstration du système de fichiers terminée";
            }
            catch (Exception ex)
            {
                txtApiOutput.Text = $"Erreur: {ex.Message}";
                statusText.Text = "Erreur lors de la démonstration du système de fichiers";
            }
        }

        private void BtnPerformance_Click(object sender, RoutedEventArgs e)
        {
            const int itemCount = 10_000;

            Stopwatch stopwatch = Stopwatch.StartNew();

            // Génération de nombreux éléments (test de performance)
            List<string> items = new(itemCount);
            for (int i = 0; i < itemCount; i++)
            {
                items.Add($"Item {i}");
            }

            stopwatch.Stop();

            performanceItemsControl.ItemsSource = items.Take(100).ToList(); // Afficher seulement les 100 premiers pour des raisons de performance

            txtPerformanceResult.Text = $"Généré {itemCount} éléments en {stopwatch.ElapsedMilliseconds} ms\n" +
                                       $"(Affichage limité aux 100 premiers éléments)";

            statusText.Text = $"Démonstration de performance terminée";
        }
    }

    // Classes modernes utilisées dans les exemples

    // Record (C# 9+)
    public record Person(string FirstName, string LastName, int Age);

    // Classes pour l'exemple JSON
    public class User
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        public string? Email { get; set; }
        public bool IsActive { get; set; }
        public DateTime RegisterDate { get; set; }
        public List<string> Roles { get; set; } = [];
        public Address? Address { get; set; }
    }

    public class Address
    {
        public string? Street { get; set; }
        public string? City { get; set; }
        public string? ZipCode { get; set; }
        public string? Country { get; set; }
    }
}
```


### Configuration des applications WPF modernes

.NET 5+ introduit de nouvelles façons de configurer les applications WPF, comme l'utilisation de l'hôte générique:

```textmate
// Program.cs (.NET 8)
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System.IO;
using System.Windows;

namespace ModernWpfApp;

public class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        // Création de l'hôte avec DI, configuration, logging, etc.
        var host = Host.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((context, config) =>
            {
                // Ajouter des sources de configuration
                config.SetBasePath(Directory.GetCurrentDirectory())
                      .AddJsonFile("appsettings.json", optional: true)
                      .AddEnvironmentVariables()
                      .AddCommandLine(args);
            })
            .ConfigureServices((context, services) =>
            {
                // Configuration des services DI
                ConfigureServices(context.Configuration, services);
            })
            .Build();

        // Créer et exécuter l'application WPF
        var app = new Application();
        var mainWindow = host.Services.GetRequiredService<MainWindow>();
        app.Run(mainWindow);
    }

    private static void ConfigureServices(IConfiguration configuration, IServiceCollection services)
    {
        // Enregistrer les services
        services.AddSingleton<IConfiguration>(configuration);

        // Enregistrer les services HTTP
        services.AddHttpClient();

        // Enregistrer les services métier
        services.AddSingleton<IDataService, DataService>();

        // Enregistrer la fenêtre principale
        services.AddSingleton<MainWindow>();
    }
}

// Interfaces et implémentations de services
public interface IDataService
{
    Task<string> GetDataAsync();
}

public class DataService : IDataService
{
    private readonly HttpClient _httpClient;
    private readonly IConfiguration _configuration;

    public DataService(HttpClient httpClient, IConfiguration configuration)
    {
        _httpClient = httpClient;
        _configuration = configuration;
    }

    public async Task<string> GetDataAsync()
    {
        string apiUrl = _configuration["ApiSettings:Url"] ?? "https://api.example.com";
        return await _httpClient.GetStringAsync(apiUrl);
    }
}
```


### Utilisation de contrôles modernes dans WPF avec .NET 5+

Avec .NET 5 et versions ultérieures, WPF peut tirer parti de diverses bibliothèques de contrôles modernes via NuGet:

```xml
<!-- Extrait de fichier projet (.NET 8) -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <!-- Bibliothèques de contrôles modernes -->
    <PackageReference Include="MahApps.Metro" Version="2.4.10" />
    <PackageReference Include="MaterialDesignThemes" Version="4.9.0" />

    <!-- Implémentation MVVM moderne -->
    <PackageReference Include="CommunityToolkit.Mvvm" Version="8.2.2" />

    <!-- Autres bibliothèques utiles -->
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Http" Version="8.0.0" />
  </ItemGroup>

  <ItemGroup>
    <None Update="appsettings.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```


### Déploiement des applications WPF .NET 5+

Le déploiement des applications WPF a été considérablement modernisé dans .NET 5 et versions ultérieures :

```xml
<!-- Configurations de déploiement dans .csproj pour .NET 8 -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>

    <!-- Options de publication -->
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
    <PublishReadyToRun>true</PublishReadyToRun>
    <PublishTrimmed>true</PublishTrimmed>
    <TrimMode>partial</TrimMode>
    <IncludeNativeLibrariesForSelfExtract>true</IncludeNativeLibrariesForSelfExtract>

    <!-- Version et métadonnées -->
    <Version>1.0.0</Version>
    <Authors>Votre Nom</Authors>
    <Company>Votre Entreprise</Company>
    <ApplicationIcon>app.ico</ApplicationIcon>
  </PropertyGroup>

  <!-- ... autres éléments ... -->
</Project>
```


Commande de publication pour générer une application autonome:
```shell script
dotnet publish -c Release -r win-x64 --self-contained true
```


## Résumé

WPF représente une avancée majeure par rapport à Windows Forms en termes de capacités graphiques, de flexibilité et d'architecture. Sa séparation claire entre l'interface utilisateur (XAML) et la logique de l'application facilite le développement et la maintenance d'applications complexes.

Avec l'arrivée de .NET Core 3.0 puis .NET 5 et versions ultérieures, WPF a évolué pour tirer parti des améliorations modernes du framework .NET tout en conservant sa puissance et sa flexibilité. Les applications WPF développées sur .NET 5+ bénéficient de:

- Une meilleure performance et efficacité des ressources
- Un déploiement simplifié, y compris la possibilité de créer des applications autonomes
- L'accès aux fonctionnalités modernes de C# et .NET
- Une intégration simplifiée avec l'injection de dépendances, la configuration et la journalisation
- La possibilité de développer et de tester sur des plateformes autres que Windows

Bien que WPF ne soit pas parfait pour tous les scénarios (notamment les applications cross-plateformes), il reste un choix solide pour développer des applications Windows riches et performantes avec une interface utilisateur sophistiquée.

⏭️ 8.2. [XAML et fondamentaux](/08-windows-presentation-foundation-wpf/8-02-xaml-et-fondamentaux.md)
