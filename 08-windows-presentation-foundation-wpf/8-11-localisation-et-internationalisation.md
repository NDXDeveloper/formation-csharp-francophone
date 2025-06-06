# 8.11. Localisation et internationalisation

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La localisation et l'internationalisation sont des aspects essentiels pour développer des applications destinées à un public international. WPF offre plusieurs outils et techniques pour créer des applications multilingues et adaptées à différentes cultures.

## 8.11.1. Resources .resx

Les fichiers de ressources (.resx) sont le moyen principal de gérer le contenu localisable dans les applications .NET.

### Création de fichiers de ressources

1. **Ajout d'un fichier de ressources**:
   - Dans l'Explorateur de solutions, faites un clic droit sur votre projet
   - Sélectionnez **Ajouter** > **Nouvel élément**
   - Choisissez **Fichier de ressources** et nommez-le (par exemple `Strings.resx`)

2. **Organisation des ressources**:
   - Créez un fichier de ressources par défaut (ex: `Strings.resx`)
   - Ajoutez des fichiers pour chaque culture supportée (ex: `Strings.fr.resx`, `Strings.es.resx`, etc.)

   ![Structure de fichiers de ressources](https://i.imgur.com/example.png)

### Édition des fichiers de ressources

L'éditeur de ressources offre une interface tabulaire pour ajouter et modifier des paires clé/valeur:

1. **Fichier de ressources par défaut** (`Strings.resx`):
   - Clé: `Greeting`
   - Valeur: `Hello!`

   - Clé: `WelcomeMessage`
   - Valeur: `Welcome to our application!`

2. **Fichier de ressources français** (`Strings.fr.resx`):
   - Clé: `Greeting`
   - Valeur: `Bonjour !`

   - Clé: `WelcomeMessage`
   - Valeur: `Bienvenue dans notre application !`

3. **Fichier de ressources espagnol** (`Strings.es.resx`):
   - Clé: `Greeting`
   - Valeur: `¡Hola!`

   - Clé: `WelcomeMessage`
   - Valeur: `¡Bienvenido a nuestra aplicación!`

### Configuration de l'accès aux ressources

Par défaut, Visual Studio génère automatiquement une classe pour accéder aux ressources.

#### .NET Framework 4.7.2

Pour les projets .NET Framework, vérifiez les propriétés du fichier de ressources:
1. Cliquez-droit sur le fichier `.resx`
2. Sélectionnez **Propriétés**
3. Assurez-vous que **Générateur de code personnalisé** est défini sur `PublicResXFileCodeGenerator`
4. La propriété **Modificateur d'accès** doit être définie sur `Public`

#### .NET 8

Pour les projets .NET Core/.NET 8, ajoutez la configuration suivante au fichier `.csproj`:

```xml
<ItemGroup>
  <EmbeddedResource Update="Resources\Strings.resx">
    <Generator>PublicResXFileCodeGenerator</Generator>
    <LastGenOutput>Strings.Designer.cs</LastGenOutput>
  </EmbeddedResource>
  <EmbeddedResource Update="Resources\Strings.fr.resx">
    <DependentUpon>Strings.resx</DependentUpon>
  </EmbeddedResource>
  <EmbeddedResource Update="Resources\Strings.es.resx">
    <DependentUpon>Strings.resx</DependentUpon>
  </EmbeddedResource>
</ItemGroup>
```


### Utilisation des ressources en code

Vous pouvez accéder aux chaînes localisées dans votre code C# :

```textmate
// Assurez-vous d'avoir l'espace de noms correct
using VotreApplication.Resources;

// Dans une méthode ou un événement
public void AfficherMessage()
{
    // Accès à la chaîne localisée via la classe générée automatiquement
    MessageBox.Show(Strings.Greeting);

    // La chaîne affichée dépendra de la culture actuelle du thread
    titleLabel.Content = Strings.WelcomeMessage;
}
```


## 8.11.2. Binding aux ressources localisées

WPF facilite l'utilisation des ressources localisées directement dans les fichiers XAML grâce aux extensions de marquage.

### Configuration de l'accès aux ressources en XAML

Tout d'abord, ajoutez une référence à vos ressources localisées dans votre fichier XAML :

```xml
<Window x:Class="VotreApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:resources="clr-namespace:VotreApplication.Resources"
        Title="MainWindow" Height="450" Width="800">
    <!-- Contenu de la fenêtre -->
</Window>
```


### Utilisation directe des ressources

Vous pouvez utiliser les ressources directement dans les attributs de contrôles :

```xml
<TextBlock Text="{x:Static resources:Strings.Greeting}"
           FontSize="24"
           HorizontalAlignment="Center"
           Margin="0,20,0,0" />

<Label Content="{x:Static resources:Strings.WelcomeMessage}"
       HorizontalAlignment="Center"
       Margin="0,10,0,0" />
```


### Utilisation avec des objets de localisation

Une approche plus flexible consiste à créer une classe dédiée pour la localisation :

```textmate
// LocalizationManager.cs
using System.ComponentModel;
using System.Globalization;
using System.Resources;

namespace VotreApplication.Localization
{
    public class LocalizationManager : INotifyPropertyChanged
    {
        private static LocalizationManager _instance;
        private ResourceManager _resourceManager;
        private CultureInfo _currentCulture;

        public static LocalizationManager Instance
        {
            get
            {
                if (_instance == null)
                {
                    _instance = new LocalizationManager();
                }
                return _instance;
            }
        }

        private LocalizationManager()
        {
            // Initialisation avec le ResourceManager de vos chaînes
            _resourceManager = Resources.Strings.ResourceManager;
            _currentCulture = CultureInfo.CurrentCulture;
        }

        public CultureInfo CurrentCulture
        {
            get { return _currentCulture; }
            set
            {
                if (_currentCulture != value)
                {
                    _currentCulture = value;
                    // Mettre à jour la culture du thread UI
                    CultureInfo.CurrentUICulture = value;
                    // Notifier les changements pour tous les messages
                    OnPropertyChanged(string.Empty);
                }
            }
        }

        public string this[string key]
        {
            get
            {
                return _resourceManager.GetString(key, _currentCulture);
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```


Cette classe peut être utilisée comme source de liaison dans XAML :

```xml
<Window x:Class="VotreApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:localization="clr-namespace:VotreApplication.Localization"
        Title="MainWindow" Height="450" Width="800">

    <Window.Resources>
        <!-- Référence à notre gestionnaire de localisation -->
        <ObjectDataProvider x:Key="LocManager"
                           ObjectType="{x:Type localization:LocalizationManager}"
                           MethodName="get_Instance" />
    </Window.Resources>

    <StackPanel>
        <!-- Liaison aux ressources localisées via le gestionnaire -->
        <TextBlock Text="{Binding Source={StaticResource LocManager}, Path=[Greeting]}"
                   FontSize="24"
                   HorizontalAlignment="Center"
                   Margin="0,20,0,0" />

        <Label Content="{Binding Source={StaticResource LocManager}, Path=[WelcomeMessage]}"
               HorizontalAlignment="Center"
               Margin="0,10,0,0" />
    </StackPanel>
</Window>
```


### LocBaml Tool pour les ressources XAML (.NET Framework 4.7.2)

Pour les projets .NET Framework, vous pouvez utiliser l'outil LocBaml pour extraire et localiser le contenu des fichiers XAML compilés.

1. **Générer un fichier .resources.dll** :
   - Compilez votre application avec l'option `/updateuid`
   - Utilisez LocBaml pour extraire les chaînes
   - Traduisez les chaînes
   - Regénérez les assemblies localisées

## 8.11.3. Changement de culture à l'exécution

L'une des fonctionnalités les plus importantes d'une application localisée est la possibilité de changer de langue pendant l'exécution.

### Configuration initiale de la culture

Au démarrage de l'application, vous pouvez définir la culture par défaut :

```textmate
// App.xaml.cs
protected override void OnStartup(StartupEventArgs e)
{
    base.OnStartup(e);

    // Définir la culture par défaut (peut venir des paramètres utilisateur)
    CultureInfo.CurrentCulture = new CultureInfo("fr-FR");
    CultureInfo.CurrentUICulture = new CultureInfo("fr-FR");

    // Si vous utilisez le LocalizationManager
    Localization.LocalizationManager.Instance.CurrentCulture = new CultureInfo("fr-FR");
}
```


### Interface pour changer de langue

```xml
<StackPanel>
    <Label Content="Choisissez votre langue :" Margin="10" />

    <ComboBox Name="languageComboBox" Width="200" Margin="10" SelectionChanged="LanguageComboBox_SelectionChanged">
        <ComboBoxItem Content="English" Tag="en-US" />
        <ComboBoxItem Content="Français" Tag="fr-FR" />
        <ComboBoxItem Content="Español" Tag="es-ES" />
    </ComboBox>

    <!-- Contenu localisé -->
    <TextBlock Text="{Binding Source={StaticResource LocManager}, Path=[Greeting]}"
               FontSize="24"
               HorizontalAlignment="Center"
               Margin="20" />
</StackPanel>
```


```textmate
// MainWindow.xaml.cs
private void LanguageComboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (languageComboBox.SelectedItem is ComboBoxItem selectedItem)
    {
        string cultureName = selectedItem.Tag.ToString();
        CultureInfo culture = new CultureInfo(cultureName);

        // Mettre à jour la culture
        Localization.LocalizationManager.Instance.CurrentCulture = culture;

        // Si vous n'utilisez pas le LocalizationManager, vous devrez peut-être
        // recharger la fenêtre ou les contrôles pour qu'ils prennent en compte la nouvelle culture
    }
}
```


### Persistance des préférences linguistiques

Pour sauvegarder les préférences linguistiques de l'utilisateur :

```textmate
// Enregistrer les préférences
private void SaveLanguagePreference(string cultureName)
{
    Properties.Settings.Default.PreferredCulture = cultureName;
    Properties.Settings.Default.Save();
}

// Charger les préférences
private void LoadLanguagePreference()
{
    string savedCulture = Properties.Settings.Default.PreferredCulture;

    if (!string.IsNullOrEmpty(savedCulture))
    {
        try
        {
            CultureInfo culture = new CultureInfo(savedCulture);
            Localization.LocalizationManager.Instance.CurrentCulture = culture;

            // Sélectionner l'élément correspondant dans le ComboBox
            foreach (ComboBoxItem item in languageComboBox.Items)
            {
                if (item.Tag.ToString() == savedCulture)
                {
                    languageComboBox.SelectedItem = item;
                    break;
                }
            }
        }
        catch (CultureNotFoundException)
        {
            // Utiliser la culture par défaut si la culture sauvegardée n'est pas valide
        }
    }
}
```


## 8.11.4. Formatage selon la culture

La localisation ne concerne pas seulement la traduction des textes, mais aussi l'adaptation du formatage des nombres, dates, monnaies, etc. selon les conventions culturelles.

### Formatage des nombres

```textmate
// Déclaration de nombres à formater
double number = 1234567.89;

// Formatage selon différentes cultures
string usFormatted = number.ToString("N", new CultureInfo("en-US")); // 1,234,567.89
string frFormatted = number.ToString("N", new CultureInfo("fr-FR")); // 1 234 567,89
string deFormatted = number.ToString("N", new CultureInfo("de-DE")); // 1.234.567,89
```


### Formatage des devises

```textmate
// Déclaration d'un montant monétaire
decimal amount = 1234.56m;

// Formatage selon différentes cultures
string usCurrency = amount.ToString("C", new CultureInfo("en-US")); // $1,234.56
string frCurrency = amount.ToString("C", new CultureInfo("fr-FR")); // 1 234,56 €
string jpCurrency = amount.ToString("C", new CultureInfo("ja-JP")); // ¥1,235
```


### Formatage des dates

```textmate
// Date à formater
DateTime date = new DateTime(2023, 5, 15);

// Formatage selon différentes cultures
string usDate = date.ToString("D", new CultureInfo("en-US")); // Monday, May 15, 2023
string frDate = date.ToString("D", new CultureInfo("fr-FR")); // lundi 15 mai 2023
string deDate = date.ToString("D", new CultureInfo("de-DE")); // Montag, 15. Mai 2023
```


### Formatage dans XAML avec des liaisons

Pour appliquer un formatage culturel dans XAML :

```xml
<StackPanel>
    <!-- Formatage des nombres -->
    <TextBlock>
        <TextBlock.Text>
            <MultiBinding StringFormat="{}{0} en format américain : {1}">
                <Binding Source="Nombre" />
                <Binding Source="{StaticResource MyNumber}" StringFormat="{}{0:N}"
                         ConverterCulture="en-US" />
            </MultiBinding>
        </TextBlock.Text>
    </TextBlock>

    <TextBlock>
        <TextBlock.Text>
            <MultiBinding StringFormat="{}{0} en format français : {1}">
                <Binding Source="Nombre" />
                <Binding Source="{StaticResource MyNumber}" StringFormat="{}{0:N}"
                         ConverterCulture="fr-FR" />
            </MultiBinding>
        </TextBlock.Text>
    </TextBlock>

    <!-- Formatage des dates -->
    <TextBlock>
        <TextBlock.Text>
            <MultiBinding StringFormat="{}{0} en format américain : {1}">
                <Binding Source="Date" />
                <Binding Source="{StaticResource MyDate}" StringFormat="{}{0:D}"
                         ConverterCulture="en-US" />
            </MultiBinding>
        </TextBlock.Text>
    </TextBlock>

    <TextBlock>
        <TextBlock.Text>
            <MultiBinding StringFormat="{}{0} en format français : {1}">
                <Binding Source="Date" />
                <Binding Source="{StaticResource MyDate}" StringFormat="{}{0:D}"
                         ConverterCulture="fr-FR" />
            </MultiBinding>
        </TextBlock.Text>
    </TextBlock>
</StackPanel>
```


### Exemple complet : Un écran de facturation localisé

Combinons ces concepts dans un exemple plus complet :

```xml
<Window x:Class="VotreApplication.InvoiceWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:VotreApplication.Localization"
        Title="{Binding Source={StaticResource LocManager}, Path=[InvoiceTitle]}"
        Height="500" Width="700">

    <Window.Resources>
        <ObjectDataProvider x:Key="LocManager"
                           ObjectType="{x:Type local:LocalizationManager}"
                           MethodName="get_Instance" />
    </Window.Resources>

    <Grid Margin="20">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <!-- En-tête avec sélection de langue -->
        <StackPanel Grid.Row="0" Orientation="Horizontal" HorizontalAlignment="Right">
            <TextBlock Text="{Binding Source={StaticResource LocManager}, Path=[Language]}"
                       VerticalAlignment="Center" Margin="0,0,10,0" />
            <ComboBox Name="languageComboBox" Width="120" SelectionChanged="LanguageComboBox_SelectionChanged">
                <ComboBoxItem Content="English" Tag="en-US" />
                <ComboBoxItem Content="Français" Tag="fr-FR" />
                <ComboBoxItem Content="Español" Tag="es-ES" />
            </ComboBox>
        </StackPanel>

        <!-- Informations de facture -->
        <StackPanel Grid.Row="1" Margin="0,20,0,10">
            <TextBlock Text="{Binding Source={StaticResource LocManager}, Path=[InvoiceNumber]}"
                       FontWeight="Bold" />
            <TextBlock Text="INV-2023-001" Margin="0,5,0,0" />

            <TextBlock Text="{Binding Source={StaticResource LocManager}, Path=[InvoiceDate]}"
                       FontWeight="Bold" Margin="0,15,0,0" />
            <TextBlock Margin="0,5,0,0">
                <Run Text="{Binding InvoiceDate, StringFormat={}{0:d}, ConverterCulture={Binding Source={StaticResource LocManager}, Path=CurrentCulture}}" />
            </TextBlock>
        </StackPanel>

        <!-- Articles de la facture -->
        <DataGrid Grid.Row="2" ItemsSource="{Binding InvoiceItems}"
                  AutoGenerateColumns="False" Margin="0,10,0,0">
            <DataGrid.Columns>
                <DataGridTextColumn Header="{Binding Source={StaticResource LocManager}, Path=[ProductCode]}"
                                    Binding="{Binding Code}" Width="100" />
                <DataGridTextColumn Header="{Binding Source={StaticResource LocManager}, Path=[Description]}"
                                    Binding="{Binding Description}" Width="*" />
                <DataGridTextColumn Header="{Binding Source={StaticResource LocManager}, Path=[Quantity]}"
                                    Binding="{Binding Quantity}" Width="80" />
                <DataGridTextColumn Header="{Binding Source={StaticResource LocManager}, Path=[UnitPrice]}"
                                    Binding="{Binding UnitPrice, StringFormat={}{0:C}, ConverterCulture={Binding Source={StaticResource LocManager}, Path=CurrentCulture}}"
                                    Width="120" />
                <DataGridTextColumn Header="{Binding Source={StaticResource LocManager}, Path=[Total]}"
                                    Binding="{Binding Total, StringFormat={}{0:C}, ConverterCulture={Binding Source={StaticResource LocManager}, Path=CurrentCulture}}"
                                    Width="120" />
            </DataGrid.Columns>
        </DataGrid>

        <!-- Totaux -->
        <Grid Grid.Row="3" Margin="0,15,0,0">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="120" />
            </Grid.ColumnDefinitions>
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto" />
                <RowDefinition Height="Auto" />
                <RowDefinition Height="Auto" />
            </Grid.RowDefinitions>

            <TextBlock Grid.Column="1" Grid.Row="0"
                       Text="{Binding Source={StaticResource LocManager}, Path=[Subtotal]}"
                       HorizontalAlignment="Right" Margin="0,5" />
            <TextBlock Grid.Column="2" Grid.Row="0"
                       Text="{Binding Subtotal, StringFormat={}{0:C}, ConverterCulture={Binding Source={StaticResource LocManager}, Path=CurrentCulture}}"
                       HorizontalAlignment="Right" Margin="0,5" />

            <TextBlock Grid.Column="1" Grid.Row="1"
                       Text="{Binding Source={StaticResource LocManager}, Path=[Tax]}"
                       HorizontalAlignment="Right" Margin="0,5" />
            <TextBlock Grid.Column="2" Grid.Row="1"
                       Text="{Binding Tax, StringFormat={}{0:C}, ConverterCulture={Binding Source={StaticResource LocManager}, Path=CurrentCulture}}"
                       HorizontalAlignment="Right" Margin="0,5" />

            <Rectangle Grid.Column="1" Grid.Row="2" Grid.ColumnSpan="2" Height="1" Fill="Black" Margin="0,5" />

            <TextBlock Grid.Column="1" Grid.Row="2"
                       Text="{Binding Source={StaticResource LocManager}, Path=[GrandTotal]}"
                       FontWeight="Bold" HorizontalAlignment="Right" Margin="0,10,0,0" />
            <TextBlock Grid.Column="2" Grid.Row="2"
                       Text="{Binding GrandTotal, StringFormat={}{0:C}, ConverterCulture={Binding Source={StaticResource LocManager}, Path=CurrentCulture}}"
                       FontWeight="Bold" HorizontalAlignment="Right" Margin="0,10,0,0" />
        </Grid>
    </Grid>
</Window>
```


```textmate
// InvoiceWindow.xaml.cs
public partial class InvoiceWindow : Window, INotifyPropertyChanged
{
    private ObservableCollection<InvoiceItem> _invoiceItems;
    private DateTime _invoiceDate = DateTime.Now;
    private decimal _subtotal;
    private decimal _tax;
    private decimal _grandTotal;

    public event PropertyChangedEventHandler PropertyChanged;

    public InvoiceWindow()
    {
        InitializeComponent();
        DataContext = this;

        // Initialiser les articles de la facture
        _invoiceItems = new ObservableCollection<InvoiceItem>
        {
            new InvoiceItem { Code = "PROD-001", Description = "Laptop Computer", Quantity = 1, UnitPrice = 1299.99m },
            new InvoiceItem { Code = "PROD-002", Description = "External Monitor", Quantity = 2, UnitPrice = 249.95m },
            new InvoiceItem { Code = "PROD-003", Description = "Wireless Mouse", Quantity = 1, UnitPrice = 29.99m }
        };

        // Calculer les totaux
        CalculateTotals();

        // Charger la culture préférée
        LoadLanguagePreference();
    }

    public ObservableCollection<InvoiceItem> InvoiceItems
    {
        get { return _invoiceItems; }
        set
        {
            _invoiceItems = value;
            OnPropertyChanged(nameof(InvoiceItems));
        }
    }

    public DateTime InvoiceDate
    {
        get { return _invoiceDate; }
        set
        {
            _invoiceDate = value;
            OnPropertyChanged(nameof(InvoiceDate));
        }
    }

    public decimal Subtotal
    {
        get { return _subtotal; }
        set
        {
            _subtotal = value;
            OnPropertyChanged(nameof(Subtotal));
        }
    }

    public decimal Tax
    {
        get { return _tax; }
        set
        {
            _tax = value;
            OnPropertyChanged(nameof(Tax));
        }
    }

    public decimal GrandTotal
    {
        get { return _grandTotal; }
        set
        {
            _grandTotal = value;
            OnPropertyChanged(nameof(GrandTotal));
        }
    }

    private void CalculateTotals()
    {
        // Calculer le sous-total
        Subtotal = InvoiceItems.Sum(item => item.Total);

        // Calculer la taxe (par exemple 20%)
        Tax = Math.Round(Subtotal * 0.2m, 2);

        // Calculer le total général
        GrandTotal = Subtotal + Tax;
    }

    private void LanguageComboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        if (languageComboBox.SelectedItem is ComboBoxItem selectedItem)
        {
            string cultureName = selectedItem.Tag.ToString();
            CultureInfo culture = new CultureInfo(cultureName);

            // Mettre à jour la culture
            Localization.LocalizationManager.Instance.CurrentCulture = culture;

            // Enregistrer la préférence
            SaveLanguagePreference(cultureName);
        }
    }

    private void SaveLanguagePreference(string cultureName)
    {
        Properties.Settings.Default.PreferredCulture = cultureName;
        Properties.Settings.Default.Save();
    }

    private void LoadLanguagePreference()
    {
        string savedCulture = Properties.Settings.Default.PreferredCulture;

        if (!string.IsNullOrEmpty(savedCulture))
        {
            try
            {
                CultureInfo culture = new CultureInfo(savedCulture);
                Localization.LocalizationManager.Instance.CurrentCulture = culture;

                // Sélectionner l'élément correspondant dans le ComboBox
                foreach (ComboBoxItem item in languageComboBox.Items)
                {
                    if (item.Tag.ToString() == savedCulture)
                    {
                        languageComboBox.SelectedItem = item;
                        break;
                    }
                }
            }
            catch (CultureNotFoundException)
            {
                // Utiliser la culture par défaut si la culture sauvegardée n'est pas valide
            }
        }
    }

    protected void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}

// Classe pour les articles de facture
public class InvoiceItem : INotifyPropertyChanged
{
    private string _code;
    private string _description;
    private int _quantity;
    private decimal _unitPrice;

    public event PropertyChangedEventHandler PropertyChanged;

    public string Code
    {
        get { return _code; }
        set
        {
            _code = value;
            OnPropertyChanged(nameof(Code));
        }
    }

    public string Description
    {
        get { return _description; }
        set
        {
            _description = value;
            OnPropertyChanged(nameof(Description));
        }
    }

    public int Quantity
    {
        get { return _quantity; }
        set
        {
            _quantity = value;
            OnPropertyChanged(nameof(Quantity));
            OnPropertyChanged(nameof(Total));
        }
    }

    public decimal UnitPrice
    {
        get { return _unitPrice; }
        set
        {
            _unitPrice = value;
            OnPropertyChanged(nameof(UnitPrice));
            OnPropertyChanged(nameof(Total));
        }
    }

    public decimal Total
    {
        get { return Quantity * UnitPrice; }
    }

    protected void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


### Différences entre .NET Framework 4.7.2 et .NET 8

#### .NET Framework 4.7.2

Dans .NET Framework, vous devez :
- Compiler les ressources avec des outils comme ResGen
- Utiliser des assemblies satellites pour les ressources localisées
- Utiliser LocBaml pour localiser les fichiers XAML compilés

#### .NET 8

.NET 8 apporte plusieurs améliorations :
- Meilleure intégration des ressources avec le système de build
- Support amélioré pour les cultures neutres
- Support amélioré du format ICU (International Components for Unicode)
- Chargement à la demande des ressources localisées

```textmate
// Exemple de fonctionnalité .NET 8 - Ressources dynamiques avec CollectionResourceManager
using System.Resources;
using System.Collections.Generic;

// Pour les environnements où les ressources peuvent changer dynamiquement
var dynamicResources = new Dictionary<string, string>
{
    { "DynamicGreeting", "Hello, dynamic world!" }
};

var resourceManager = new CollectionResourceManager(dynamicResources);
string greeting = resourceManager.GetString("DynamicGreeting", CultureInfo.CurrentUICulture);
```


## Récapitulatif

La localisation et l'internationalisation sont des aspects importants du développement d'applications WPF destinées à un public international. En utilisant les fichiers de ressources .resx, le binding aux ressources localisées, le changement de culture à l'exécution et le formatage selon la culture, vous pouvez créer des applications qui s'adaptent parfaitement aux préférences linguistiques et culturelles de vos utilisateurs.

Les bonnes pratiques à retenir :

1. **Organisez vos ressources** en fichiers .resx par culture
2. **Évitez les chaînes codées en dur** dans votre code et XAML
3. **Utilisez les mécanismes de binding** pour accéder aux ressources localisées
4. **Permettez à l'utilisateur de changer de langue** à l'exécution
5. **Tenez compte des conventions culturelles** pour le formatage des nombres, dates et monnaies

⏭️ 9. [Accès aux données avec ADO.NET](/09-acces-aux-donnees-avec-ado-dotnet/README.md)
