# 8.2. XAML et fondamentaux

🔝 Retour au [Sommaire](/SOMMAIRE.md)

XAML (eXtensible Application Markup Language) est au cœur de WPF. Ce langage déclaratif basé sur XML permet de définir l'interface utilisateur de manière claire et séparée de la logique applicative.

## 8.2.1. Syntaxe XAML

XAML suit les règles syntaxiques de XML tout en ajoutant des conventions spécifiques pour créer des interfaces utilisateur.

### Structure de base d'un document XAML

```xml
<!-- Structure de base d'un document XAML -->
<Window x:Class="DemoApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Ma première fenêtre XAML"
        Width="500"
        Height="300">

    <!-- Contenu de la fenêtre -->
    <Grid>
        <!-- Éléments enfants -->
        <TextBlock Text="Bonjour, XAML!"
                  HorizontalAlignment="Center"
                  VerticalAlignment="Center"
                  FontSize="24"/>
    </Grid>
</Window>
```


### Balises d'éléments et propriétés

En XAML, les balises représentent généralement des classes et les attributs XML représentent des propriétés de ces classes.

```xml
<!-- Différentes façons de définir une propriété -->

<!-- 1. Syntaxe attribut -->
<Button Content="Cliquez-moi" Width="100" Height="30"/>

<!-- 2. Syntaxe élément-propriété -->
<Button Width="100" Height="30">
    <Button.Content>
        Cliquez-moi
    </Button.Content>
</Button>

<!-- 3. Contenu implicite (pour les propriétés Content) -->
<Button Width="100" Height="30">
    Cliquez-moi
</Button>
```


### Types de propriétés complexes

Pour les propriétés qui ne sont pas de type simple (comme string, int, etc.), XAML offre plusieurs approches.

```xml
<!-- Propriétés qui ne sont pas de simples types primitifs -->

<!-- Brushes (pinceaux) -->
<Rectangle Width="200" Height="100">
    <!-- Utilisation de la syntaxe élément-propriété pour une propriété complexe -->
    <Rectangle.Fill>
        <LinearGradientBrush StartPoint="0,0" EndPoint="1,1">
            <GradientStop Color="Blue" Offset="0"/>
            <GradientStop Color="LightBlue" Offset="0.5"/>
            <GradientStop Color="White" Offset="1"/>
        </LinearGradientBrush>
    </Rectangle.Fill>
</Rectangle>

<!-- Transformations -->
<TextBlock Text="Texte avec rotation" FontSize="18">
    <TextBlock.RenderTransform>
        <RotateTransform Angle="45"/>
    </TextBlock.RenderTransform>
</TextBlock>

<!-- Syntaxe plus compacte avec TypeConverter pour certains types -->
<Rectangle Width="200" Height="100" Fill="Blue"/>
<Rectangle Width="200" Height="100" Fill="#0000FF"/>
<Rectangle Width="200" Height="100" Fill="rgb(0,0,255)"/>
```


### Collections en XAML

XAML facilite également la définition de collections d'objets.

```xml
<!-- Collections d'objets -->

<!-- Collection explicite -->
<StackPanel>
    <StackPanel.Children>
        <Button Content="Bouton 1"/>
        <Button Content="Bouton 2"/>
        <Button Content="Bouton 3"/>
    </StackPanel.Children>
</StackPanel>

<!-- Collection implicite (plus courante) -->
<StackPanel>
    <Button Content="Bouton 1"/>
    <Button Content="Bouton 2"/>
    <Button Content="Bouton 3"/>
</StackPanel>

<!-- Autre exemple avec une collection explicite -->
<ComboBox Width="150">
    <ComboBox.Items>
        <ComboBoxItem Content="Option 1"/>
        <ComboBoxItem Content="Option 2"/>
        <ComboBoxItem Content="Option 3"/>
    </ComboBox.Items>
</ComboBox>

<!-- Version simplifiée -->
<ComboBox Width="150">
    <ComboBoxItem>Option 1</ComboBoxItem>
    <ComboBoxItem>Option 2</ComboBoxItem>
    <ComboBoxItem>Option 3</ComboBoxItem>
</ComboBox>
```


### Propriétés attachées

Les propriétés attachées sont un concept clé en XAML qui permet à un élément enfant d'être configuré par son parent.

```xml
<!-- Propriétés attachées -->

<!-- Grid avec définition de lignes et colonnes -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="2*"/>
    </Grid.ColumnDefinitions>

    <!-- Utilisation des propriétés attachées Grid.Row et Grid.Column -->
    <TextBlock Text="En-tête" Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"/>
    <ListBox Grid.Row="1" Grid.Column="0"/>
    <TextBox Grid.Row="1" Grid.Column="1"/>
    <Button Content="OK" Grid.Row="2" Grid.Column="1" HorizontalAlignment="Right"/>
</Grid>

<!-- Autre exemple : Canvas avec positionnement absolu -->
<Canvas>
    <Rectangle Canvas.Left="10" Canvas.Top="10" Width="100" Height="50" Fill="Red"/>
    <Ellipse Canvas.Left="120" Canvas.Top="30" Width="80" Height="80" Fill="Blue"/>
    <TextBlock Canvas.Left="50" Canvas.Top="100" Text="Positionnement absolu"/>
</Canvas>
```


## 8.2.2. Espace de noms et préfixes

Les espaces de noms XAML permettent d'organiser et de référencer des types dans différentes bibliothèques ou parties de votre application.

### Espaces de noms XAML standard

```xml
<!-- Espaces de noms standard dans un document XAML -->
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d"
        Title="Espaces de noms XAML" Height="350" Width="500">

    <!-- Corps de la fenêtre -->
</Window>
```


| Préfixe | URI d'espace de noms | Description |
| ------- | -------------------- | ----------- |
| (défaut) | http://schemas.microsoft.com/winfx/2006/xaml/presentation | Contrôles et éléments WPF standards |
| x | http://schemas.microsoft.com/winfx/2006/xaml | Fonctionnalités XAML de base (x:Class, x:Name, etc.) |
| d | http://schemas.microsoft.com/expression/blend/2008 | Support pour le design-time |
| mc | http://schemas.openxmlformats.org/markup-compatibility/2006 | Compatibilité du balisage (ignorable, etc.) |

### Définir des espaces de noms personnalisés

```xml
<!-- Déclaration d'espaces de noms personnalisés -->
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:MonApplication"
        xmlns:controls="clr-namespace:MonApplication.Controls"
        xmlns:vm="clr-namespace:MonApplication.ViewModels"
        xmlns:sys="clr-namespace:System;assembly=mscorlib"
        Title="Espaces de noms personnalisés" Height="350" Width="500">

    <Window.DataContext>
        <!-- Utilisation d'un type de l'espace de noms vm -->
        <vm:MainViewModel/>
    </Window.DataContext>

    <StackPanel>
        <!-- Utilisation d'un contrôle personnalisé depuis l'espace de noms controls -->
        <controls:RatingControl Value="3" Maximum="5"/>

        <!-- Utilisation d'un type .NET standard depuis l'espace de noms sys -->
        <TextBlock>
            <sys:DateTime>2023-12-31T23:59:59</sys:DateTime>
        </TextBlock>
    </StackPanel>
</Window>
```


### Sélection de l'assemblage externe

Pour utiliser des types provenant d'assemblages externes, il faut spécifier l'assemblage dans la déclaration de l'espace de noms.

```xml
<!-- Déclaration d'espaces de noms d'assemblages externes -->
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:materialDesign="http://materialdesigninxaml.net/winfx/xaml/themes"
        xmlns:syncfusion="http://schemas.syncfusion.com/wpf"
        xmlns:thirdParty="clr-namespace:ThirdPartyLibrary;assembly=ThirdPartyLibrary"
        Title="Assemblages externes" Height="350" Width="500">

    <Grid>
        <!-- Utilisation d'un contrôle Material Design -->
        <materialDesign:Card Padding="32" Margin="16">
            <TextBlock Text="Carte Material Design"/>
        </materialDesign:Card>

        <!-- Utilisation d'un contrôle personnalisé d'un assemblage tiers -->
        <thirdParty:CustomCalendar Grid.Row="1"/>
    </Grid>
</Window>
```


## 8.2.3. Relation XAML/code-behind

XAML et code-behind travaillent ensemble pour séparer l'interface utilisateur de la logique.

### Liaison entre XAML et code-behind

```xml
<!-- MainWindow.xaml -->
<Window x:Class="DemoApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Démo XAML et Code-behind"
        Height="300"
        Width="400"
        Loaded="Window_Loaded">

    <Grid>
        <StackPanel Margin="20">
            <TextBlock Text="Entrez votre nom:"/>
            <TextBox x:Name="txtNom" Margin="0,5,0,15"/>

            <Button x:Name="btnSaluer"
                    Content="Dire bonjour"
                    Click="btnSaluer_Click"
                    Width="120"
                    HorizontalAlignment="Left"/>

            <TextBlock x:Name="txtResultat"
                       Margin="0,20,0,0"
                       FontSize="16"
                       Foreground="Green"/>
        </StackPanel>
    </Grid>
</Window>
```


```textmate
// MainWindow.xaml.cs (.NET Framework 4.7.2 ou .NET 8)
using System.Windows;

namespace DemoApplication
{
    /// <summary>
    /// Logique d'interaction pour MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            // Initialise les composants définis en XAML
            InitializeComponent();
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            // Code exécuté au chargement de la fenêtre
            txtNom.Focus();
        }

        private void btnSaluer_Click(object sender, RoutedEventArgs e)
        {
            // Récupération de la valeur du TextBox nommé en XAML
            string nom = txtNom.Text.Trim();

            if (string.IsNullOrEmpty(nom))
            {
                txtResultat.Text = "Veuillez entrer votre nom.";
                txtResultat.Foreground = System.Windows.Media.Brushes.Red;
            }
            else
            {
                txtResultat.Text = $"Bonjour, {nom} !";
                txtResultat.Foreground = System.Windows.Media.Brushes.Green;
            }
        }
    }
}
```


### Génération du fichier code-behind par le compilateur XAML

Lorsque vous compilez une application WPF, le compilateur XAML génère une classe partielle qui sera combinée avec votre code-behind.

```textmate
// Extrait du code généré par le compilateur (ne pas modifier directement)
namespace DemoApplication
{
    public partial class MainWindow : System.Windows.Window, System.Windows.Markup.IComponentConnector
    {

        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "X.X.X.X")]
        public void InitializeComponent()
        {
            // Chargement et initialisation du XAML
            // ...
        }

        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "X.X.X.X")]
        void System.Windows.Markup.IComponentConnector.Connect(int connectionId, object target)
        {
            switch (connectionId)
            {
                case 1:
                    this.txtNom = ((System.Windows.Controls.TextBox)(target));
                    break;
                case 2:
                    this.btnSaluer = ((System.Windows.Controls.Button)(target));
                    this.btnSaluer.Click += new System.Windows.RoutedEventHandler(this.btnSaluer_Click);
                    break;
                case 3:
                    this.txtResultat = ((System.Windows.Controls.TextBlock)(target));
                    break;
            }
        }

        internal System.Windows.Controls.TextBox txtNom;
        internal System.Windows.Controls.Button btnSaluer;
        internal System.Windows.Controls.TextBlock txtResultat;
    }
}
```


### Accès aux éléments XAML depuis le code-behind

Il existe plusieurs façons d'accéder aux éléments XAML depuis le code-behind.

```xml
<!-- Accès aux éléments XAML depuis le code-behind -->
<Window x:Class="DemoApplication.ElementsAccess"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Accès aux éléments" Height="350" Width="500">

    <Grid x:Name="mainGrid">
        <StackPanel>
            <!-- Élément avec un nom x:Name -->
            <Button x:Name="btnNamed" Content="Bouton nommé" Margin="10"/>

            <!-- Élément sans nom -->
            <Button Content="Bouton sans nom" Margin="10"/>

            <!-- Container nommé avec des enfants -->
            <StackPanel x:Name="containerPanel" Margin="10">
                <TextBox x:Name="txtInput" Width="200" Margin="0,0,0,5"/>
                <Button Content="OK" Width="80" HorizontalAlignment="Right"/>
            </StackPanel>
        </StackPanel>
    </Grid>
</Window>
```


```textmate
// ElementsAccess.xaml.cs (.NET Framework 4.7.2 ou .NET 8)
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;

namespace DemoApplication
{
    public partial class ElementsAccess : Window
    {
        public ElementsAccess()
        {
            InitializeComponent();

            // 1. Accès via x:Name (façon la plus directe)
            btnNamed.Background = Brushes.LightBlue;
            txtInput.Text = "Texte initial";

            // 2. Accès via l'arbre visuel
            Button secondButton = VisualTreeHelper.GetChild(
                VisualTreeHelper.GetChild(mainGrid, 0), 1) as Button;

            if (secondButton != null)
            {
                secondButton.Background = Brushes.LightGreen;
            }

            // 3. Accès via l'arbre logique
            foreach (var child in LogicalTreeHelper.GetChildren(containerPanel))
            {
                if (child is Button button)
                {
                    button.Background = Brushes.LightSalmon;
                }
            }

            // 4. Accès via les méthodes FindName
            Button namedButton = this.FindName("btnNamed") as Button;
            if (namedButton != null)
            {
                namedButton.ToolTip = "Accédé via FindName";
            }

            // 5. Accès via méthodes de recherche génériques
            Button okButton = FindVisualChild<Button>(containerPanel,
                button => button.Content.ToString() == "OK");

            if (okButton != null)
            {
                okButton.FontWeight = FontWeights.Bold;
            }
        }

        // Méthode utilitaire pour trouver un enfant visuel avec prédicat
        private static T FindVisualChild<T>(DependencyObject parent, Predicate<T> predicate)
            where T : DependencyObject
        {
            if (parent == null) return null;

            int childCount = VisualTreeHelper.GetChildrenCount(parent);
            for (int i = 0; i < childCount; i++)
            {
                DependencyObject child = VisualTreeHelper.GetChild(parent, i);

                if (child is T typedChild && predicate(typedChild))
                {
                    return typedChild;
                }

                T result = FindVisualChild<T>(child, predicate);
                if (result != null)
                {
                    return result;
                }
            }

            return null;
        }
    }
}
```


## 8.2.4. Markup Extensions

Les extensions de balisage (Markup Extensions) permettent d'étendre les capacités du XAML en fournissant des valeurs dynamiques là où habituellement des valeurs littérales seraient attendues.

### Syntaxe générale des Markup Extensions

```xml
<!-- Syntaxe générale: {NomExtension Paramètre1=Valeur1, Paramètre2=Valeur2} -->

<!-- Exemples de Markup Extensions -->
<StackPanel>
    <!-- StaticResource pour récupérer une ressource définie ailleurs -->
    <TextBlock Text="Texte stylé" Foreground="{StaticResource AccentColorBrush}"/>

    <!-- Binding pour lier à une propriété d'une source de données -->
    <TextBlock Text="{Binding UserName}"/>

    <!-- RelativeSource pour des liaisons relatives -->
    <TextBox Text="{Binding Path=Content, RelativeSource={RelativeSource FindAncestor, AncestorType={x:Type Button}}}"/>
</StackPanel>
```


### Markup Extensions courantes en WPF

```xml
<!-- Extensions de balisage courantes dans WPF -->
<Window x:Class="DemoApplication.MarkupExtensionsDemo"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:sys="clr-namespace:System;assembly=mscorlib"
        Title="Démonstration des Extensions de Balisage"
        Height="500" Width="700">

    <!-- Définition de ressources -->
    <Window.Resources>
        <!-- Ressources simples -->
        <SolidColorBrush x:Key="AccentBrush" Color="#0078D7"/>
        <sys:Double x:Key="LargeSize">18</sys:Double>

        <!-- Styles -->
        <Style x:Key="ButtonStyle" TargetType="Button">
            <Setter Property="Background" Value="{StaticResource AccentBrush}"/>
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="Padding" Value="12,5"/>
            <Setter Property="Margin" Value="5"/>
        </Style>
    </Window.Resources>

    <Grid Margin="20">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- 1. StaticResource et DynamicResource -->
        <GroupBox Grid.Row="0" Header="StaticResource et DynamicResource" Padding="10">
            <StackPanel>
                <TextBlock Text="StaticResource: récupère la ressource à la compilation"
                           Foreground="{StaticResource AccentBrush}"
                           FontSize="{StaticResource LargeSize}"
                           Margin="0,0,0,10"/>

                <TextBlock Text="DynamicResource: récupère la ressource dynamiquement"
                           Foreground="{DynamicResource AccentBrush}"
                           FontSize="{DynamicResource LargeSize}"/>

                <Button Content="Changer les ressources"
                        Click="ChangeResources_Click"
                        HorizontalAlignment="Left"
                        Margin="0,10,0,0"/>
            </StackPanel>
        </GroupBox>

        <!-- 2. Binding -->
        <GroupBox Grid.Row="1" Header="Binding" Padding="10">
            <StackPanel>
                <TextBlock Text="Binding: lie les propriétés des objets" Margin="0,0,0,10"/>

                <Grid>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition Width="*"/>
                    </Grid.ColumnDefinitions>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                    </Grid.RowDefinitions>

                    <TextBlock Grid.Row="0" Grid.Column="0" Text="Votre nom:" Margin="0,0,10,0"/>
                    <TextBox Grid.Row="0" Grid.Column="1" x:Name="txtUserName"
                             Text="{Binding UserName, UpdateSourceTrigger=PropertyChanged}"/>

                    <TextBlock Grid.Row="1" Grid.Column="0" Text="Prévisualisation:" Margin="0,10,10,0"/>
                    <TextBlock Grid.Row="1" Grid.Column="1" Margin="0,10,0,0"
                               Text="{Binding ElementName=txtUserName, Path=Text, StringFormat='Bonjour, {0}!'}"/>
                </Grid>
            </StackPanel>
        </GroupBox>

        <!-- 3. TemplateBinding -->
        <GroupBox Grid.Row="2" Header="TemplateBinding" Padding="10">
            <StackPanel>
                <TextBlock Text="TemplateBinding: utilisé principalement dans les templates de contrôles"
                           Margin="0,0,0,10"/>

                <Button Content="Bouton avec template personnalisé" Padding="10,5">
                    <Button.Template>
                        <ControlTemplate TargetType="Button">
                            <Border Background="{TemplateBinding Background}"
                                    BorderBrush="{TemplateBinding BorderBrush}"
                                    BorderThickness="{TemplateBinding BorderThickness}"
                                    Padding="{TemplateBinding Padding}"
                                    CornerRadius="5">
                                <ContentPresenter HorizontalAlignment="Center"
                                                  VerticalAlignment="Center"/>
                            </Border>
                        </ControlTemplate>
                    </Button.Template>
                </Button>
            </StackPanel>
        </GroupBox>

        <!-- 4. x:Static -->
        <GroupBox Grid.Row="3" Header="x:Static" Padding="10">
            <StackPanel>
                <TextBlock Text="x:Static: récupère la valeur d'un champ ou propriété statique"
                           Margin="0,0,0,10"/>

                <TextBlock Text="{x:Static sys:DateTime.Now}"/>
                <TextBlock Text="{x:Static sys:Environment.MachineName}"
                           Margin="0,5,0,0"/>
                <TextBlock Text="{x:Static sys:Environment.OSVersion}"
                           Margin="0,5,0,0"/>
            </StackPanel>
        </GroupBox>

        <!-- 5. RelativeSource et MultiBinding -->
        <GroupBox Grid.Row="4" Header="RelativeSource et MultiBinding" Padding="10">
            <StackPanel>
                <TextBlock Text="RelativeSource: références relatives dans l'arbre"
                           Margin="0,0,0,10"/>

                <Border BorderBrush="Gray" BorderThickness="1" Padding="8">
                    <TextBlock Text="{Binding RelativeSource={RelativeSource FindAncestor,
                               AncestorType={x:Type Window}}, Path=Title}"/>
                </Border>

                <TextBlock Text="MultiBinding: combinaison de plusieurs valeurs"
                           Margin="0,10,0,10"/>

                <StackPanel Orientation="Horizontal">
                    <TextBox x:Name="txtFirst" Width="100" Margin="0,0,5,0" Text="Prénom"/>
                    <TextBox x:Name="txtLast" Width="100" Text="Nom"/>
                    <TextBlock Margin="10,0,0,0">
                        <TextBlock.Text>
                            <MultiBinding StringFormat="{}{0} {1}">
                                <Binding ElementName="txtFirst" Path="Text"/>
                                <Binding ElementName="txtLast" Path="Text"/>
                            </MultiBinding>
                        </TextBlock.Text>
                    </TextBlock>
                </StackPanel>
            </StackPanel>
        </GroupBox>

        <!-- 6. Autres exemples -->
        <GroupBox Grid.Row="5" Header="Autres extensions de balisage" Padding="10">
            <StackPanel>
                <Button Style="{StaticResource ButtonStyle}"
                        Content="Utilisation de StaticResource pour un style"/>

                <TextBlock Margin="0,10,0,0">
                    <TextBlock.Text>
                        <Binding Source="{x:Static sys:DateTime.Now}" StringFormat="Il est {0:HH:mm:ss}"/>
                    </TextBlock.Text>
                </TextBlock>

                <!-- x:Null pour définir explicitement une valeur null -->
                <TextBox Margin="0,10,0,0" Width="200"
                         Text="{x:Null}"/>

                <!-- x:Type pour obtenir un type -->
                <TextBlock Margin="0,10,0,0"
                           Text="{Binding Source={x:Type Button}, Path=Name}"/>
            </StackPanel>
        </GroupBox>
    </Grid>
</Window>
```


```textmate
// MarkupExtensionsDemo.xaml.cs (.NET Framework 4.7.2 ou .NET 8)
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows;
using System.Windows.Media;

namespace DemoApplication
{
    public partial class MarkupExtensionsDemo : Window, INotifyPropertyChanged
    {
        private string _userName = "Utilisateur";

        public string UserName
        {
            get => _userName;
            set
            {
                if (_userName != value)
                {
                    _userName = value;
                    OnPropertyChanged();
                }
            }
        }

        public MarkupExtensionsDemo()
        {
            InitializeComponent();
            this.DataContext = this;
        }

        private void ChangeResources_Click(object sender, RoutedEventArgs e)
        {
            // Modifier les ressources dynamiquement
            this.Resources["AccentBrush"] = new SolidColorBrush(Colors.OrangeRed);
            this.Resources["LargeSize"] = 24.0;
        }

        // Implémentation de INotifyPropertyChanged
        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```


### Création d'une Markup Extension personnalisée

Vous pouvez également créer vos propres extensions de balisage pour étendre les fonctionnalités de XAML.

```textmate
// .NET Framework 4.7.2 ou .NET 8
using System;
using System.Windows;
using System.Windows.Markup;

namespace DemoApplication
{
    // Extension de balisage personnalisée qui calcule une valeur en fonction du temps
    [MarkupExtensionReturnType(typeof(double))]
    public class TimeBasedValueExtension : MarkupExtension
    {
        // Propriétés de l'extension
        public double MinValue { get; set; } = 0;
        public double MaxValue { get; set; } = 100;
        public TimeSpan Interval { get; set; } = TimeSpan.FromSeconds(10);

        public TimeBasedValueExtension()
        {
        }

        public TimeBasedValueExtension(double minValue, double maxValue)
        {
            MinValue = minValue;
            MaxValue = maxValue;
        }

        public override object ProvideValue(IServiceProvider serviceProvider)
        {
            // Calcule une valeur entre MinValue et MaxValue basée sur le temps actuel
            DateTime now = DateTime.Now;
            double totalSeconds = (now.TimeOfDay.TotalSeconds % Interval.TotalSeconds);
            double ratio = totalSeconds / Interval.TotalSeconds;

            return MinValue + (ratio * (MaxValue - MinValue));
        }
    }

    // Extension qui récupère une valeur de configuration
    [MarkupExtensionReturnType(typeof(string))]
    public class ConfigValueExtension : MarkupExtension
    {
        public string Key { get; set; }
        public string DefaultValue { get; set; } = string.Empty;

        public ConfigValueExtension()
        {
        }

        public ConfigValueExtension(string key)
        {
            Key = key;
        }

        public override object ProvideValue(IServiceProvider serviceProvider)
        {
            if (string.IsNullOrEmpty(Key))
                return DefaultValue;

            // Récupérer la valeur depuis la configuration
            try
            {
                string value = System.Configuration.ConfigurationManager.AppSettings[Key];
                return string.IsNullOrEmpty(value) ? DefaultValue : value;
            }
            catch
            {
                return DefaultValue;
            }
        }
    }

    // Extension qui fournit un texte aléatoire pour les prototypes
    [MarkupExtensionReturnType(typeof(string))]
    public class LoremIpsumExtension : MarkupExtension
    {
        private static readonly string[] _loremTexts = new string[]
        {
            "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
            "Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
            "Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris.",
            "Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore.",
            "Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt."
        };

        public int WordCount { get; set; } = 10;
        public bool AllowParagraphs { get; set; } = false;

        public override object ProvideValue(IServiceProvider serviceProvider)
        {
            if (AllowParagraphs)
            {
                // Renvoyer des paragraphes complets
                int paragraphCount = Math.Max(1, WordCount / 20);
                return string.Join("\n\n", _loremTexts.Take(paragraphCount));
            }
            else
            {
                // Extraire le nombre de mots demandés
                string allWords = string.Join(" ", _loremTexts);
                string[] words = allWords.Split(new[] { ' ', ',', '.', ';', ':', '?' },
                    StringSplitOptions.RemoveEmptyEntries);

                return string.Join(" ", words.Take(WordCount)) + ".";
            }
        }
    }
}
```


Utilisation des extensions personnalisées en XAML :

```xml
<!-- Utilisation des extensions personnalisées -->
<Window x:Class="DemoApplication.CustomExtensionsDemo"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:DemoApplication"
        Title="Démonstration des extensions personnalisées" Height="400" Width="600">

    <StackPanel Margin="20">
        <TextBlock Text="Extensions de balisage personnalisées"
                   FontSize="18"
                   FontWeight="Bold"
                   Margin="0,0,0,20"/>

        <!-- Utilisation de TimeBasedValueExtension -->
        <ProgressBar Height="20"
                     Minimum="0"
                     Maximum="100"
                     Value="{local:TimeBasedValue MinValue=0, MaxValue=100,
                            Interval=0:0:5}"/>

        <TextBlock Text="Progression basée sur le temps (change toutes les 5 secondes)"
                   Margin="0,5,0,20"/>

        <!-- Utilisation de ConfigValueExtension -->
        <TextBlock>
            <Run Text="Valeur depuis la configuration: "/>
            <Run Text="{local:ConfigValue 'ApplicationName', DefaultValue='Application par défaut'}"/>
        </TextBlock>

        <!-- Utilisation de LoremIpsumExtension -->
        <GroupBox Header="Texte Lorem Ipsum pour prototype" Margin="0,20,0,0">
            <StackPanel Margin="10">
                <TextBlock Text="{local:LoremIpsum WordCount=30}"
                           TextWrapping="Wrap"/>

                <TextBlock Text="Paragraphes:"
                           FontWeight="Bold"
                           Margin="0,20,0,5"/>

                <TextBlock Text="{local:LoremIpsum WordCount=100, AllowParagraphs=True}"
                           TextWrapping="Wrap"/>
            </StackPanel>
        </GroupBox>
    </StackPanel>
</Window>
```


## 8.2.5. Éditeur XAML et designer

Les IDE modernes comme Visual Studio offrent des outils puissants pour travailler avec XAML, combinant un éditeur de code efficace et un designer visuel.

### Éditeur XAML dans Visual Studio

L'éditeur XAML de Visual Studio offre plusieurs fonctionnalités qui facilitent l'écriture de code XAML:

- **Coloration syntaxique** pour améliorer la lisibilité
- **IntelliSense** pour compléter les noms d'éléments, attributs et valeurs
- **Validation en temps réel** pour détecter les erreurs de syntaxe
- **Navigation entre le XAML et le code-behind**
- **Formatage automatique** du code XAML
- **Raccourcis de saisie** pour insérer rapidement des structures courantes

### Designer visuel

Le designer visuel permet de construire visuellement les interfaces sans écrire directement le XAML.

#### Modes d'affichage

Visual Studio propose plusieurs modes d'affichage pour travailler avec le XAML :

1. **Design** - Vue uniquement du designer visuel
2. **XAML** - Vue uniquement du code XAML
3. **Split** - Vue partagée entre le designer et le code

#### Outils de conception

Plusieurs outils facilitent la conception d'interfaces dans Visual Studio:

- **Boîte à outils** - Contient tous les contrôles disponibles qu'on peut glisser-déposer
- **Propriétés** - Fenêtre permettant de modifier les propriétés des éléments sélectionnés
- **Arborescence des documents** - Affiche la structure hiérarchique des éléments XAML
- **Ressources** - Gestion des ressources partagées (styles, templates, etc.)

#### Blend pour Visual Studio

Blend est un outil de conception avancé intégré à Visual Studio, offrant des fonctionnalités supplémentaires:

- **Création d'animations** visuellement
- **Édition avancée de templates**
- **Manipulation directe** des objets visuels
- **Outils artistiques** plus sophistiqués
- **Édition de comportements** sans code

### Conseils pour utiliser efficacement l'éditeur et le designer

```xml
<!-- Exemple de document XAML bien organisé pour l'édition -->
<Window x:Class="DemoApplication.WellOrganizedWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:DemoApplication"
        Title="Fenêtre bien organisée"
        Height="450"
        Width="800">

    <!-- Définir les ressources au niveau de la fenêtre -->
    <Window.Resources>
        <!-- Couleurs -->
        <SolidColorBrush x:Key="PrimaryColor" Color="#3498db"/>
        <SolidColorBrush x:Key="SecondaryColor" Color="#2ecc71"/>
        <SolidColorBrush x:Key="TextColor" Color="#2c3e50"/>

        <!-- Styles -->
        <Style x:Key="HeaderStyle" TargetType="TextBlock">
            <Setter Property="FontSize" Value="20"/>
            <Setter Property="FontWeight" Value="Bold"/>
            <Setter Property="Margin" Value="0,0,0,10"/>
            <Setter Property="Foreground" Value="{StaticResource TextColor}"/>
        </Style>

        <Style x:Key="ButtonStyle" TargetType="Button">
            <Setter Property="Background" Value="{StaticResource PrimaryColor}"/>
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="Padding" Value="15,8"/>
            <Setter Property="Margin" Value="0,5"/>
        </Style>
    </Window.Resources>

    <!-- Mise en page principale avec Grid -->
    <Grid Margin="20">
        <!-- Définition des rangées et colonnes avec commentaires -->
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/> <!-- En-tête -->
            <RowDefinition Height="*"/>     <!-- Contenu principal -->
            <RowDefinition Height="Auto"/> <!-- Pied de page -->
        </Grid.RowDefinitions>

        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="250"/> <!-- Menu latéral -->
            <ColumnDefinition Width="*"/>   <!-- Zone de contenu -->
        </Grid.ColumnDefinitions>

        <!-- En-tête spanning les deux colonnes -->
        <TextBlock Grid.Row="0"
                   Grid.Column="0"
                   Grid.ColumnSpan="2"
                   Text="Application bien organisée"
                   Style="{StaticResource HeaderStyle}"
                   FontSize="24"/>

        <!-- Menu latéral -->
        <Border Grid.Row="1"
                Grid.Column="0"
                Background="#f5f5f5"
                Margin="0,10,10,0"
                Padding="10"
                BorderBrush="#e0e0e0"
                BorderThickness="1"
                CornerRadius="5">

            <StackPanel>
                <TextBlock Text="Menu"
                           Style="{StaticResource HeaderStyle}"/>

                <Button Content="Accueil"
                        Style="{StaticResource ButtonStyle}"/>

                <Button Content="Produits"
                        Style="{StaticResource ButtonStyle}"/>

                <Button Content="Services"
                        Style="{StaticResource ButtonStyle}"/>

                <Button Content="À propos"
                        Style="{StaticResource ButtonStyle}"/>

                <Button Content="Contact"
                        Style="{StaticResource ButtonStyle}"/>
            </StackPanel>
        </Border>

        <!-- Zone de contenu principal -->
        <Grid Grid.Row="1"
              Grid.Column="1"
              Margin="0,10,0,0">

            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
            </Grid.RowDefinitions>

            <!-- En-tête de la zone de contenu -->
            <TextBlock Grid.Row="0"
                       Text="Bienvenue"
                       Style="{StaticResource HeaderStyle}"/>

            <!-- Contenu principal -->
            <ScrollViewer Grid.Row="1"
                          VerticalScrollBarVisibility="Auto">
                <StackPanel>
                    <TextBlock Text="Contenu de la page"
                               Margin="0,0,0,10"/>

                    <Border BorderBrush="#e0e0e0"
                            BorderThickness="1"
                            Padding="15"
                            Margin="0,10">
                        <TextBlock TextWrapping="Wrap">
                            Lorem ipsum dolor sit amet, consectetur adipiscing elit.
                            Nullam in dui mauris. Vivamus hendrerit arcu sed erat
                            molestie vehicula. Sed auctor neque eu tellus rhoncus
                            ut eleifend nibh porttitor.
                        </TextBlock>
                    </Border>

                    <!-- Section d'exemple avec formulaire -->
                    <GroupBox Header="Formulaire d'exemple" Margin="0,20,0,0">
                        <Grid Margin="10">
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="Auto"/>
                                <ColumnDefinition Width="*"/>
                            </Grid.ColumnDefinitions>

                            <Grid.RowDefinitions>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                            </Grid.RowDefinitions>

                            <!-- Champs du formulaire -->
                            <Label Grid.Row="0" Grid.Column="0" Content="Nom:"/>
                            <TextBox Grid.Row="0" Grid.Column="1" Margin="0,5,0,5"/>

                            <Label Grid.Row="1" Grid.Column="0" Content="Email:"/>
                            <TextBox Grid.Row="1" Grid.Column="1" Margin="0,5,0,5"/>

                            <Label Grid.Row="2" Grid.Column="0" Content="Message:"/>
                            <TextBox Grid.Row="2" Grid.Column="1"
                                     Margin="0,5,0,5"
                                     Height="80"
                                     TextWrapping="Wrap"
                                     AcceptsReturn="True"/>

                            <!-- Bouton d'envoi -->
                            <Button Grid.Row="3"
                                    Grid.Column="1"
                                    Content="Envoyer"
                                    Style="{StaticResource ButtonStyle}"
                                    HorizontalAlignment="Right"
                                    Margin="0,10,0,0"
                                    Background="{StaticResource SecondaryColor}"/>
                        </Grid>
                    </GroupBox>
                </StackPanel>
            </ScrollViewer>
        </Grid>

        <!-- Pied de page -->
        <Border Grid.Row="2"
                Grid.Column="0"
                Grid.ColumnSpan="2"
                Background="#f5f5f5"
                Margin="0,10,0,0"
                Padding="10"
                BorderBrush="#e0e0e0"
                BorderThickness="1">

            <TextBlock Text="© 2023 Démonstration XAML - Tous droits réservés"
                       HorizontalAlignment="Center"/>
        </Border>
    </Grid>
</Window>
```


### Conseils pratiques pour l'éditeur XAML et le designer

1. **Organisation du code XAML**
   - Structurez votre XAML de manière hiérarchique et cohérente
   - Utilisez des commentaires pour documenter les sections complexes
   - Maintenez une indentation cohérente pour améliorer la lisibilité

2. **Utilisation efficace du designer**
   - Utilisez le designer pour créer la structure globale, mais peaufinez les détails en XAML
   - Apprenez les raccourcis clavier pour basculer entre le code et le designer
   - Utilisez la vue partagée pour voir les changements en temps réel

3. **Gestion des ressources**
   - Définissez les ressources partagées (styles, brushes, etc.) au niveau approprié
   - Utilisez des dictionnaires de ressources séparés pour les éléments réutilisables
   - Nommez clairement les ressources pour faciliter leur identification

4. **Débogage XAML**
   - Utilisez les outils de débogage pour inspecter l'arbre visuel ou logique
   - Activez les exceptions XAML pour détecter rapidement les problèmes
   - Utilisez des outils comme Snoop ou Visual Studio Live Property Explorer pour inspecter les propriétés à l'exécution

5. **Travail en équipe**
   - Établissez des conventions de nommage et de style pour le XAML
   - Utilisez des extraits de code partagés pour les structures courantes
   - Divisez les interfaces complexes en contrôles utilisateur réutilisables

## Résumé

XAML est un langage déclaratif puissant qui offre une séparation claire entre l'interface utilisateur et la logique de l'application. Cette section a couvert les fondamentaux du XAML, notamment:

- La syntaxe XAML de base pour définir les éléments et propriétés
- La gestion des espaces de noms et préfixes pour organiser et référencer les types
- La relation entre le XAML et le code-behind, et comment ils communiquent
- Les extensions de balisage qui étendent les capacités du XAML
- Les outils d'édition et de conception pour travailler efficacement avec XAML

Maîtriser ces fondamentaux est essentiel pour développer des applications WPF sophistiquées et maintenables. La combinaison de XAML pour la présentation et de C# pour la logique permet de créer des interfaces utilisateur riches et interactives tout en maintenant un code propre et organisé.

⏭️
