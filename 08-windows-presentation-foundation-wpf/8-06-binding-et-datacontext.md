# 8.6. Binding et DataContext

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le binding de données est l'un des concepts les plus puissants de WPF, permettant de lier des propriétés de l'interface utilisateur à des sources de données. Cette approche est un pilier du pattern MVVM (Model-View-ViewModel), qui sépare clairement la logique métier de l'interface utilisateur.

## 8.6.1. Syntax de binding

### Structure de base d'une expression de binding

```xml
<TextBlock Text="{Binding Path=PropertyName, Source=sourceObject, Mode=OneWay}" />
```


Éléments principaux d'une expression de binding :
- **Path** : Chemin vers la propriété à lier
- **Source** : Source de données explicite
- **Mode** : Mode de liaison (OneWay, TwoWay, OneTime, OneWayToSource, Default)
- **UpdateSourceTrigger** : Quand mettre à jour la source (PropertyChanged, LostFocus, Explicit)
- **Converter** : Convertisseur de valeur
- **StringFormat** : Format d'affichage pour les valeurs
- **FallbackValue** : Valeur à utiliser si le binding échoue
- **TargetNullValue** : Valeur à utiliser si la source est null

### Bindings simples

```xml
<!-- Binding à une propriété -->
<TextBox Text="{Binding Name}" />

<!-- Binding avec format de chaîne -->
<TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" />

<!-- Binding avec valeur de fallback -->
<TextBlock Text="{Binding Address, FallbackValue='Adresse non disponible'}" />

<!-- Binding avec valeur pour null -->
<TextBlock Text="{Binding Comment, TargetNullValue='Aucun commentaire'}" />
```


### Bindings avec source explicite

```xml
<!-- Binding à un élément nommé -->
<TextBox x:Name="inputName" />
<TextBlock Text="{Binding Text, ElementName=inputName}" />

<!-- Binding à une ressource statique -->
<Window.Resources>
    <local:Person x:Key="currentUser" Name="John Doe" Age="30" />
</Window.Resources>

<TextBlock Text="{Binding Name, Source={StaticResource currentUser}}" />

<!-- Binding à une ressource dynamique -->
<TextBlock Text="{Binding Name, Source={DynamicResource currentUser}}" />

<!-- Binding à une propriété dépendante (RelativeSource) -->
<TextBlock Text="{Binding Background, RelativeSource={RelativeSource Self}}" />

<!-- Binding à l'ancêtre -->
<TextBlock Text="{Binding Title, RelativeSource={RelativeSource AncestorType=Window}}" />
```


### Bindings hiérarchiques

```xml
<!-- Binding à une propriété imbriquée -->
<TextBlock Text="{Binding Customer.Address.City}" />

<!-- Binding indexé (pour les collections) -->
<TextBlock Text="{Binding Clients[0].Name}" />

<!-- Binding à une collection d'objets -->
<ListBox ItemsSource="{Binding Products}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}" Margin="0,0,5,0" />
                <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" />
            </StackPanel>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```


### DataContext

Le `DataContext` est la source de données par défaut pour tous les bindings d'un élément et de ses enfants. Il est généralement défini au niveau de la fenêtre ou d'un conteneur principal.

```xml
<!-- Définition du DataContext au niveau de la fenêtre -->
<Window x:Class="MyApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Ma Fenêtre"
        DataContext="{Binding Source={StaticResource MainViewModel}}">

    <!-- Tous les bindings à l'intérieur utiliseront MainViewModel comme source par défaut -->
    <StackPanel>
        <TextBlock Text="{Binding Title}" />
        <TextBox Text="{Binding SearchQuery}" />
        <ListBox ItemsSource="{Binding Items}" />
    </StackPanel>
</Window>
```


En code-behind :

```textmate
public MainWindow()
{
    InitializeComponent();

    // Affectation du DataContext en code
    DataContext = new MainViewModel();
}
```


### Binding avec MultiBinding

MultiBinding permet de combiner plusieurs sources en une seule valeur cible en utilisant un convertisseur.

```xml
<Window.Resources>
    <!-- Convertisseur qui combine plusieurs valeurs -->
    <local:FullNameConverter x:Key="FullNameConverter" />
</Window.Resources>

<TextBlock>
    <TextBlock.Text>
        <MultiBinding Converter="{StaticResource FullNameConverter}">
            <Binding Path="FirstName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>
```


Implémentation du convertisseur :

```textmate
public class FullNameConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        if (values.Length == 2 && values[0] is string firstName && values[1] is string lastName)
        {
            return $"{firstName} {lastName}";
        }

        return string.Empty;
    }

    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        if (value is string fullName)
        {
            string[] parts = fullName.Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);

            if (parts.Length >= 2)
            {
                return new object[] { parts[0], parts[1] };
            }
        }

        return new object[] { string.Empty, string.Empty };
    }
}
```


### PriorityBinding

`PriorityBinding` permet de spécifier plusieurs bindings par ordre de priorité, utile pour afficher rapidement une valeur par défaut ou mise en cache pendant qu'une valeur plus précise mais plus lente à obtenir est calculée.

```xml
<TextBlock>
    <TextBlock.Text>
        <PriorityBinding>
            <!-- Binding haute priorité (rapide mais peut-être pas à jour) -->
            <Binding Path="CachedData" FallbackValue="Chargement..." />

            <!-- Binding basse priorité (plus précis mais peut être lent) -->
            <Binding Path="LiveData" IsAsync="True" />
        </PriorityBinding>
    </TextBlock.Text>
</TextBlock>
```


## 8.6.2. Modes de binding

WPF propose plusieurs modes de binding qui définissent comment les modifications sont propagées entre la source et la cible.

### OneWay

Les modifications de la source mettent à jour la cible, mais pas l'inverse. Idéal pour l'affichage de données en lecture seule.

```xml
<TextBlock Text="{Binding Status, Mode=OneWay}" />
```


### TwoWay

Les modifications sont propagées dans les deux sens. Utilisé pour les contrôles d'édition.

```xml
<TextBox Text="{Binding UserName, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
```


### OneTime

La cible est mise à jour une seule fois au moment où le binding est initialisé. Utile pour les données statiques ou rarement modifiées.

```xml
<TextBlock Text="{Binding AppVersion, Mode=OneTime}" />
```


### OneWayToSource

Les modifications de la cible mettent à jour la source, mais pas l'inverse. Cas d'utilisation moins fréquent.

```xml
<Slider Value="{Binding Volume, Mode=OneWayToSource}" />
```


### Default

Chaque propriété dépendante a un mode de binding par défaut :
- OneWay : Pour la plupart des propriétés en lecture seule (Text des TextBlock, etc.)
- TwoWay : Pour les contrôles d'édition (Text des TextBox, IsChecked des CheckBox, etc.)
- OneTime : Rarement utilisé par défaut

```xml
<!-- Utilise le mode par défaut (TwoWay pour TextBox.Text) -->
<TextBox Text="{Binding UserName}" />
```


### UpdateSourceTrigger

Détermine quand les changements de la cible sont propagés à la source dans un binding TwoWay ou OneWayToSource.

```xml
<!-- Mise à jour immédiate de la source à chaque frappe -->
<TextBox Text="{Binding SearchQuery, UpdateSourceTrigger=PropertyChanged}" />

<!-- Mise à jour de la source uniquement quand le focus est perdu -->
<TextBox Text="{Binding Email, UpdateSourceTrigger=LostFocus}" />

<!-- Mise à jour de la source uniquement quand explicitement demandé -->
<TextBox x:Name="customTextBox"
         Text="{Binding CustomValue, UpdateSourceTrigger=Explicit}" />
```


Dans le code-behind, pour un binding Explicit :

```textmate
// Force la mise à jour de la source
BindingExpression bindingExpression = customTextBox.GetBindingExpression(TextBox.TextProperty);
bindingExpression.UpdateSource();
```


### Bindings Delayed

Pour WPF sous .NET 8, vous pouvez utiliser des bindings retardés pour limiter les mises à jour fréquentes :

```xml
<!-- Mise à jour après 500 ms d'inactivité -->
<TextBox Text="{Binding SearchQuery, UpdateSourceTrigger=PropertyChanged, Delay=500}" />
```


## 8.6.3. Conversion de données

Les convertisseurs permettent de transformer les données entre la source et la cible du binding.

### IValueConverter

Interface de base pour la conversion de valeurs uniques.

```textmate
public class BoolToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool isVisible)
        {
            return isVisible ? Visibility.Visible : Visibility.Collapsed;
        }

        return Visibility.Collapsed;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is Visibility visibility)
        {
            return visibility == Visibility.Visible;
        }

        return false;
    }
}
```


Utilisation en XAML :

```xml
<Window.Resources>
    <local:BoolToVisibilityConverter x:Key="BoolToVisibilityConverter" />
</Window.Resources>

<StackPanel>
    <CheckBox x:Name="showDetailsCheckBox" Content="Afficher les détails" IsChecked="True" />

    <StackPanel Visibility="{Binding IsChecked, ElementName=showDetailsCheckBox,
                            Converter={StaticResource BoolToVisibilityConverter}}">
        <TextBlock Text="Détails supplémentaires" FontWeight="Bold" />
        <TextBlock Text="Voici des informations détaillées sur l'élément." />
    </StackPanel>
</StackPanel>
```


### Convertisseurs avec paramètres

```textmate
public class BoolToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool isVisible)
        {
            // Si le paramètre "Invert" est passé, inverser la logique
            if (parameter is string param && param == "Invert")
            {
                isVisible = !isVisible;
            }

            return isVisible ? Visibility.Visible : Visibility.Collapsed;
        }

        return Visibility.Collapsed;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is Visibility visibility)
        {
            bool result = visibility == Visibility.Visible;

            // Si le paramètre "Invert" est passé, inverser la logique
            if (parameter is string param && param == "Invert")
            {
                result = !result;
            }

            return result;
        }

        return false;
    }
}
```


Utilisation avec paramètre :

```xml
<TextBlock Visibility="{Binding IsHidden, Converter={StaticResource BoolToVisibilityConverter},
                        ConverterParameter=Invert}" />
```


### IMultiValueConverter

Interface pour convertir plusieurs valeurs en une seule.

```textmate
public class LogicalAndConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        // Vérifie si toutes les valeurs sont true
        return values.All(v => v is bool b && b);
    }

    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```


Utilisation :

```xml
<Window.Resources>
    <local:LogicalAndConverter x:Key="LogicalAndConverter" />
</Window.Resources>

<Button Content="Valider">
    <Button.IsEnabled>
        <MultiBinding Converter="{StaticResource LogicalAndConverter}">
            <Binding Path="IsNameValid" />
            <Binding Path="IsEmailValid" />
            <Binding Path="HasAcceptedTerms" />
        </MultiBinding>
    </Button.IsEnabled>
</Button>
```


### Convertisseurs intégrés

WPF fournit quelques convertisseurs intégrés :

```xml
<!-- BooleanToVisibilityConverter -->
<TextBlock Text="Article indisponible"
           Visibility="{Binding IsOutOfStock,
           Converter={StaticResource BooleanToVisibilityConverter}}" />
```


### Création d'une bibliothèque de convertisseurs réutilisables

```textmate
// Convertisseur booléen inversé
public class InverseBoolConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool b)
        {
            return !b;
        }

        return false;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool b)
        {
            return !b;
        }

        return false;
    }
}

// Convertisseur de chaîne en majuscules/minuscules
public class StringCaseConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is string s)
        {
            if (parameter is string param)
            {
                switch (param.ToLower())
                {
                    case "upper":
                        return s.ToUpper();
                    case "lower":
                        return s.ToLower();
                    case "title":
                        // Convertit chaque première lettre de mot en majuscule
                        TextInfo textInfo = culture.TextInfo;
                        return textInfo.ToTitleCase(s.ToLower());
                }
            }

            return s;
        }

        return string.Empty;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value;
    }
}

// Convertisseur pour formatage conditionnel
public class ComparisonConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value == null || parameter == null)
            return false;

        // Essaie de convertir la valeur et le paramètre en nombres
        if (double.TryParse(value.ToString(), out double doubleValue) &&
            double.TryParse(parameter.ToString(), out double threshold))
        {
            return doubleValue > threshold;
        }

        return value.ToString() == parameter.ToString();
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```


Définition en ressources XAML :

```xml
<Window.Resources>
    <local:InverseBoolConverter x:Key="InverseBoolConverter" />
    <local:StringCaseConverter x:Key="StringCaseConverter" />
    <local:ComparisonConverter x:Key="ComparisonConverter" />

    <Style x:Key="HighlightStyle" TargetType="TextBlock">
        <Style.Triggers>
            <DataTrigger Binding="{Binding Value, Converter={StaticResource ComparisonConverter}, ConverterParameter=50}" Value="True">
                <Setter Property="Foreground" Value="Red" />
                <Setter Property="FontWeight" Value="Bold" />
            </DataTrigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<StackPanel>
    <CheckBox Content="Désactiver l'édition" IsChecked="{Binding IsEditingDisabled}" />
    <TextBox IsEnabled="{Binding IsEditingDisabled, Converter={StaticResource InverseBoolConverter}}" />

    <TextBlock Text="{Binding UserName, Converter={StaticResource StringCaseConverter}, ConverterParameter=upper}" />

    <TextBlock Text="{Binding Score}" Style="{StaticResource HighlightStyle}" />
</StackPanel>
```


## 8.6.4. ValidationRules

Les règles de validation permettent de vérifier les entrées utilisateur dans un binding et de fournir un retour visuel.

### Validation simple

```textmate
public class NotEmptyValidationRule : ValidationRule
{
    public override ValidationResult Validate(object value, CultureInfo cultureInfo)
    {
        string input = value as string;

        if (string.IsNullOrWhiteSpace(input))
        {
            return new ValidationResult(false, "Le champ ne peut pas être vide.");
        }

        return ValidationResult.ValidResult;
    }
}
```


Utilisation en XAML :

```xml
<TextBox>
    <TextBox.Text>
        <Binding Path="UserName" UpdateSourceTrigger="PropertyChanged">
            <Binding.ValidationRules>
                <local:NotEmptyValidationRule />
            </Binding.ValidationRules>
        </Binding>
    </TextBox.Text>
</TextBox>
```


### Validation avec paramètres

```textmate
public class StringLengthValidationRule : ValidationRule
{
    public int MinLength { get; set; } = 0;
    public int MaxLength { get; set; } = int.MaxValue;
    public string ErrorMessage { get; set; } = "La longueur du texte est invalide.";

    public override ValidationResult Validate(object value, CultureInfo cultureInfo)
    {
        string input = value as string ?? string.Empty;

        if (input.Length < MinLength || input.Length > MaxLength)
        {
            return new ValidationResult(false, ErrorMessage);
        }

        return ValidationResult.ValidResult;
    }
}
```


Utilisation avec paramètres :

```xml
<TextBox>
    <TextBox.Text>
        <Binding Path="Password" UpdateSourceTrigger="PropertyChanged">
            <Binding.ValidationRules>
                <local:StringLengthValidationRule MinLength="8" MaxLength="20"
                    ErrorMessage="Le mot de passe doit contenir entre 8 et 20 caractères." />
            </Binding.ValidationRules>
        </Binding>
    </TextBox.Text>
</TextBox>
```


### ValidationRule avec accès au DataContext

```textmate
public class PasswordMatchValidationRule : ValidationRule
{
    public override ValidationResult Validate(object value, CultureInfo cultureInfo)
    {
        var bindingExpression = (value as BindingExpression);

        if (bindingExpression != null)
        {
            // Accès au DataContext
            var viewModel = bindingExpression.DataItem as RegistrationViewModel;

            if (viewModel != null)
            {
                string confirmPassword = value.ToString();

                if (confirmPassword != viewModel.Password)
                {
                    return new ValidationResult(false, "Les mots de passe ne correspondent pas.");
                }
            }
        }

        return ValidationResult.ValidResult;
    }
}
```


Une approche plus moderne avec ValidationRule et DependencyObject :

```textmate
public class PasswordMatchValidationRule : ValidationRule
{
    public DependencyProperty PasswordProperty { get; set; }

    public override ValidationResult Validate(object value, CultureInfo cultureInfo)
    {
        if (value is string confirmPassword && PasswordProperty != null)
        {
            var password = PasswordProperty.GetValue(DataContext) as string;

            if (confirmPassword != password)
            {
                return new ValidationResult(false, "Les mots de passe ne correspondent pas.");
            }
        }

        return ValidationResult.ValidResult;
    }
}
```


### Style de validation et messages d'erreur

```xml
<Style TargetType="TextBox">
    <Setter Property="Validation.ErrorTemplate">
        <Setter.Value>
            <ControlTemplate>
                <StackPanel>
                    <AdornedElementPlaceholder />
                    <TextBlock Text="{Binding [0].ErrorContent}" Foreground="Red" />
                </StackPanel>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
    <Style.Triggers>
        <Trigger Property="Validation.HasError" Value="True">
            <Setter Property="BorderBrush" Value="Red" />
            <Setter Property="ToolTip" Value="{Binding RelativeSource={RelativeSource Self},
                                              Path=(Validation.Errors)[0].ErrorContent}" />
        </Trigger>
    </Style.Triggers>
</Style>
```


### Validation côté ViewModel (IDataErrorInfo)

```textmate
public class UserViewModel : INotifyPropertyChanged, IDataErrorInfo
{
    private string _userName;
    public string UserName
    {
        get => _userName;
        set
        {
            _userName = value;
            OnPropertyChanged(nameof(UserName));
        }
    }

    // IDataErrorInfo implementation
    public string Error => null;

    public string this[string columnName]
    {
        get
        {
            string error = null;

            switch (columnName)
            {
                case nameof(UserName):
                    if (string.IsNullOrWhiteSpace(UserName))
                    {
                        error = "Le nom d'utilisateur est requis.";
                    }
                    else if (UserName.Length < 3)
                    {
                        error = "Le nom d'utilisateur doit contenir au moins 3 caractères.";
                    }
                    break;
            }

            return error;
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


Binding avec validation IDataErrorInfo :

```xml
<TextBox Text="{Binding UserName, UpdateSourceTrigger=PropertyChanged, ValidatesOnDataErrors=True}" />
```


### Validation avec INotifyDataErrorInfo (.NET 4.5+ et .NET Core)

```textmate
public class UserViewModel : INotifyPropertyChanged, INotifyDataErrorInfo
{
    private string _userName;
    private readonly Dictionary<string, List<string>> _errors = new Dictionary<string, List<string>>();

    public string UserName
    {
        get => _userName;
        set
        {
            _userName = value;
            ValidateUserName();
            OnPropertyChanged(nameof(UserName));
        }
    }

    private void ValidateUserName()
    {
        var errors = new List<string>();

        if (string.IsNullOrWhiteSpace(UserName))
        {
            errors.Add("Le nom d'utilisateur est requis.");
        }
        else if (UserName.Length < 3)
        {
            errors.Add("Le nom d'utilisateur doit contenir au moins 3 caractères.");
        }

        if (errors.Any())
        {
            _errors[nameof(UserName)] = errors;
        }
        else if (_errors.ContainsKey(nameof(UserName)))
        {
            _errors.Remove(nameof(UserName));
        }

        ErrorsChanged?.Invoke(this, new DataErrorsChangedEventArgs(nameof(UserName)));
    }

    // INotifyDataErrorInfo implementation
    public bool HasErrors => _errors.Any();

    public event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;

    public IEnumerable GetErrors(string propertyName)
    {
        if (string.IsNullOrEmpty(propertyName) || !_errors.ContainsKey(propertyName))
        {
            return null;
        }

        return _errors[propertyName];
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


Binding avec validation INotifyDataErrorInfo :

```xml
<TextBox Text="{Binding UserName, UpdateSourceTrigger=PropertyChanged, ValidatesOnNotifyDataErrors=True}" />
```


## 8.6.5. Change Notification (INotifyPropertyChanged)

Pour que les bindings détectent les changements dans les objets sources, ces objets doivent implémenter l'interface `INotifyPropertyChanged`.

### Implémentation de base

```textmate
public class PersonViewModel : INotifyPropertyChanged
{
    private string _name;
    private int _age;

    public string Name
    {
        get => _name;
        set
        {
            if (_name != value)
            {
                _name = value;
                OnPropertyChanged(nameof(Name));
            }
        }
    }

    public int Age
    {
        get => _age;
        set
        {
            if (_age != value)
            {
                _age = value;
                OnPropertyChanged(nameof(Age));
                // Mettre à jour aussi les propriétés dépendantes
                OnPropertyChanged(nameof(IsAdult));
            }
        }
    }

    public bool IsAdult => Age >= 18;

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


### Classe de base réutilisable

```textmate
public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetProperty<T>(ref T field, T value, [CallerMemberName] string propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value))
        {
            return false;
        }

        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
```


Utilisation de la classe de base :

```textmate
public class CustomerViewModel : ViewModelBase
{
    private string _firstName;
    private string _lastName;

    public string FirstName
    {
        get => _firstName;
        set
        {
            if (SetProperty(ref _firstName, value))
            {
                OnPropertyChanged(nameof(FullName));
            }
        }
    }

    public string LastName
    {
        get => _lastName;
        set
        {
            if (SetProperty(ref _lastName, value))
            {
                OnPropertyChanged(nameof(FullName));
            }
        }
    }

    public string FullName => $"{FirstName} {LastName}";
}
```


### Notification de changement pour les collections

```textmate
public class ProductsViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public ProductsViewModel()
    {
        Products = new ObservableCollection<Product>();

        // Ajouter des produits
        Products.Add(new Product { Name = "Pomme", Price = 1.99 });
        Products.Add(new Product { Name = "Banane", Price = 2.49 });
        Products.Add(new Product { Name = "Orange", Price = 1.79 });
    }

    public void AddProduct(Product product)
    {
        Products.Add(product); // La collection notifie automatiquement l'UI
    }

    public void RemoveProduct(Product product)
    {
        Products.Remove(product); // La collection notifie automatiquement l'UI
    }
}
```


Binding à une collection observable :

```xml
<ListBox ItemsSource="{Binding Products}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}" Margin="0,0,5,0" />
                <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" />
            </StackPanel>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

### Collections avec tri, filtrage et groupement en temps réel

```textmate
public class AdvancedCollectionViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;
    private string _searchQuery;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public string SearchQuery
    {
        get => _searchQuery;
        set
        {
            if (SetProperty(ref _searchQuery, value))
            {
                // Rafraîchir la vue de la collection quand la requête change
                ProductsView.Refresh();
            }
        }
    }

    // La vue qui sera liée à l'UI
    public ICollectionView ProductsView { get; private set; }

    public AdvancedCollectionViewModel()
    {
        Products = new ObservableCollection<Product>();

        // Ajouter des produits
        Products.Add(new Product { Name = "Pomme", Price = 1.99, Category = "Fruits" });
        Products.Add(new Product { Name = "Banane", Price = 2.49, Category = "Fruits" });
        Products.Add(new Product { Name = "Orange", Price = 1.79, Category = "Fruits" });
        Products.Add(new Product { Name = "Carotte", Price = 0.99, Category = "Légumes" });
        Products.Add(new Product { Name = "Brocoli", Price = 2.29, Category = "Légumes" });
        Products.Add(new Product { Name = "Tomate", Price = 1.49, Category = "Légumes" });
        Products.Add(new Product { Name = "Pain", Price = 2.99, Category = "Boulangerie" });
        Products.Add(new Product { Name = "Croissant", Price = 1.29, Category = "Boulangerie" });

        // Créer une vue de la collection
        ProductsView = CollectionViewSource.GetDefaultView(Products);

        // Configurer le filtrage
        ProductsView.Filter = FilterProducts;

        // Configurer le tri
        ProductsView.SortDescriptions.Add(new SortDescription("Category", ListSortDirection.Ascending));
        ProductsView.SortDescriptions.Add(new SortDescription("Name", ListSortDirection.Ascending));

        // Configurer le groupement
        ProductsView.GroupDescriptions.Add(new PropertyGroupDescription("Category"));
    }

    private bool FilterProducts(object item)
    {
        if (string.IsNullOrWhiteSpace(SearchQuery))
            return true;

        if (item is Product product)
        {
            return product.Name.Contains(SearchQuery, StringComparison.OrdinalIgnoreCase) ||
                   product.Category.Contains(SearchQuery, StringComparison.OrdinalIgnoreCase);
        }

        return false;
    }
}
```


Binding à la vue de collection avec groupement :

```xml
<Window.Resources>
    <!-- Template pour les en-têtes de groupe -->
    <DataTemplate x:Key="GroupHeaderTemplate">
        <TextBlock Text="{Binding Name}" FontWeight="Bold" FontSize="14" Margin="0,10,0,5"/>
    </DataTemplate>
</Window.Resources>

<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Barre de recherche -->
    <StackPanel Orientation="Horizontal" Margin="10">
        <TextBlock Text="Recherche:" VerticalAlignment="Center" Margin="0,0,5,0"/>
        <TextBox Text="{Binding SearchQuery, UpdateSourceTrigger=PropertyChanged}" Width="200"/>
    </StackPanel>

    <!-- Liste des produits groupés -->
    <ListView Grid.Row="1" ItemsSource="{Binding ProductsView}" Margin="10">
        <ListView.GroupStyle>
            <GroupStyle HeaderTemplate="{StaticResource GroupHeaderTemplate}"/>
        </ListView.GroupStyle>
        <ListView.ItemTemplate>
            <DataTemplate>
                <Grid Margin="5">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="*"/>
                        <ColumnDefinition Width="Auto"/>
                    </Grid.ColumnDefinitions>
                    <TextBlock Text="{Binding Name}" VerticalAlignment="Center"/>
                    <TextBlock Grid.Column="1" Text="{Binding Price, StringFormat=\{0:C\}}" Margin="20,0,0,0"/>
                </Grid>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</Grid>
```


## 8.6.6. CollectionView et filtrage

CollectionView est une interface puissante pour manipuler les vues des collections avec filtrage, tri et groupement.

### Utilisation de base de CollectionView

```textmate
public class ProductsViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public ICollectionView ProductsView { get; private set; }

    public ProductsViewModel()
    {
        Products = new ObservableCollection<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99, Category = "Électronique" },
            new Product { Id = 2, Name = "Smartphone", Price = 699.99, Category = "Électronique" },
            new Product { Id = 3, Name = "Chaise de bureau", Price = 199.99, Category = "Mobilier" },
            new Product { Id = 4, Name = "Écran 27\"", Price = 349.99, Category = "Électronique" },
            new Product { Id = 5, Name = "Bureau", Price = 299.99, Category = "Mobilier" }
        };

        ProductsView = CollectionViewSource.GetDefaultView(Products);
    }
}
```


Binding à une CollectionView :

```xml
<ListBox ItemsSource="{Binding ProductsView}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}" Margin="0,0,5,0" />
                <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" />
            </StackPanel>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```


### Filtrage avec CollectionView

```textmate
public class ProductsViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;
    private string _filterText;
    private decimal _maxPrice = decimal.MaxValue;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public string FilterText
    {
        get => _filterText;
        set
        {
            if (SetProperty(ref _filterText, value))
            {
                ProductsView.Refresh();
            }
        }
    }

    public decimal MaxPrice
    {
        get => _maxPrice;
        set
        {
            if (SetProperty(ref _maxPrice, value))
            {
                ProductsView.Refresh();
            }
        }
    }

    public ICollectionView ProductsView { get; private set; }

    public ProductsViewModel()
    {
        Products = new ObservableCollection<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Électronique" },
            new Product { Id = 2, Name = "Smartphone", Price = 699.99m, Category = "Électronique" },
            new Product { Id = 3, Name = "Chaise de bureau", Price = 199.99m, Category = "Mobilier" },
            new Product { Id = 4, Name = "Écran 27\"", Price = 349.99m, Category = "Électronique" },
            new Product { Id = 5, Name = "Bureau", Price = 299.99m, Category = "Mobilier" }
        };

        ProductsView = CollectionViewSource.GetDefaultView(Products);
        ProductsView.Filter = FilterProducts;
    }

    private bool FilterProducts(object item)
    {
        if (item is Product product)
        {
            // Filtre par texte
            bool matchesText = string.IsNullOrWhiteSpace(FilterText) ||
                               product.Name.Contains(FilterText, StringComparison.OrdinalIgnoreCase) ||
                               product.Category.Contains(FilterText, StringComparison.OrdinalIgnoreCase);

            // Filtre par prix
            bool matchesPrice = product.Price <= MaxPrice;

            return matchesText && matchesPrice;
        }

        return false;
    }
}
```


Interface utilisateur pour le filtrage :

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Filtrage par texte -->
    <StackPanel Orientation="Horizontal" Margin="10,10,10,5">
        <TextBlock Text="Recherche:" VerticalAlignment="Center" Margin="0,0,5,0"/>
        <TextBox Text="{Binding FilterText, UpdateSourceTrigger=PropertyChanged}" Width="200"/>
    </StackPanel>

    <!-- Filtrage par prix -->
    <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="10,5,10,10">
        <TextBlock Text="Prix max:" VerticalAlignment="Center" Margin="0,0,5,0"/>
        <Slider Value="{Binding MaxPrice}" Minimum="0" Maximum="1000" Width="150" TickFrequency="100"
                IsSnapToTickEnabled="True" TickPlacement="BottomRight" Margin="0,0,10,0"/>
        <TextBlock Text="{Binding MaxPrice, StringFormat=\{0:C\}}" VerticalAlignment="Center"/>
    </StackPanel>

    <!-- Liste des produits filtrés -->
    <ListView Grid.Row="2" ItemsSource="{Binding ProductsView}" Margin="10">
        <ListView.View>
            <GridView>
                <GridViewColumn Header="Nom" DisplayMemberBinding="{Binding Name}" Width="150"/>
                <GridViewColumn Header="Catégorie" DisplayMemberBinding="{Binding Category}" Width="100"/>
                <GridViewColumn Header="Prix" Width="100">
                    <GridViewColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" HorizontalAlignment="Right"/>
                        </DataTemplate>
                    </GridViewColumn.CellTemplate>
                </GridViewColumn>
            </GridView>
        </ListView.View>
    </ListView>
</Grid>
```


### Tri interactif avec CollectionView

```textmate
public class ProductsViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;
    private string _sortProperty;
    private bool _isSortAscending = true;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public string SortProperty
    {
        get => _sortProperty;
        set
        {
            // Si on clique sur la même colonne, on inverse l'ordre de tri
            if (_sortProperty == value)
            {
                IsSortAscending = !IsSortAscending;
            }
            else
            {
                SetProperty(ref _sortProperty, value);
                IsSortAscending = true;
            }

            ApplySorting();
        }
    }

    public bool IsSortAscending
    {
        get => _isSortAscending;
        set
        {
            if (SetProperty(ref _isSortAscending, value))
            {
                ApplySorting();
            }
        }
    }

    public ICollectionView ProductsView { get; private set; }

    public ProductsViewModel()
    {
        Products = new ObservableCollection<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Électronique" },
            new Product { Id = 2, Name = "Smartphone", Price = 699.99m, Category = "Électronique" },
            new Product { Id = 3, Name = "Chaise de bureau", Price = 199.99m, Category = "Mobilier" },
            new Product { Id = 4, Name = "Écran 27\"", Price = 349.99m, Category = "Électronique" },
            new Product { Id = 5, Name = "Bureau", Price = 299.99m, Category = "Mobilier" }
        };

        ProductsView = CollectionViewSource.GetDefaultView(Products);

        // Tri initial
        SortProperty = "Name";
    }

    private void ApplySorting()
    {
        if (string.IsNullOrEmpty(SortProperty))
            return;

        ProductsView.SortDescriptions.Clear();

        ListSortDirection direction = IsSortAscending ?
            ListSortDirection.Ascending : ListSortDirection.Descending;

        ProductsView.SortDescriptions.Add(new SortDescription(SortProperty, direction));
    }

    // Commandes pour trier
    public ICommand SortByNameCommand => new RelayCommand(() => SortProperty = "Name");
    public ICommand SortByCategoryCommand => new RelayCommand(() => SortProperty = "Category");
    public ICommand SortByPriceCommand => new RelayCommand(() => SortProperty = "Price");
}
```


Interface pour le tri interactif :

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Boutons de tri -->
    <StackPanel Orientation="Horizontal" Margin="10">
        <TextBlock Text="Trier par:" VerticalAlignment="Center" Margin="0,0,10,0"/>
        <Button Content="Nom" Command="{Binding SortByNameCommand}" Margin="5,0"/>
        <Button Content="Catégorie" Command="{Binding SortByCategoryCommand}" Margin="5,0"/>
        <Button Content="Prix" Command="{Binding SortByPriceCommand}" Margin="5,0"/>
        <CheckBox Content="Ordre croissant" IsChecked="{Binding IsSortAscending}"
                  VerticalAlignment="Center" Margin="15,0,0,0"/>
    </StackPanel>

    <!-- Liste des produits -->
    <ListView Grid.Row="1" ItemsSource="{Binding ProductsView}" Margin="10">
        <ListView.View>
            <GridView>
                <GridViewColumn Header="Nom" DisplayMemberBinding="{Binding Name}" Width="150"/>
                <GridViewColumn Header="Catégorie" DisplayMemberBinding="{Binding Category}" Width="100"/>
                <GridViewColumn Header="Prix" Width="100">
                    <GridViewColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" HorizontalAlignment="Right"/>
                        </DataTemplate>
                    </GridViewColumn.CellTemplate>
                </GridViewColumn>
            </GridView>
        </ListView.View>
    </ListView>
</Grid>
```


### Groupement avec CollectionView

```textmate
public class ProductsViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;
    private bool _isGroupingEnabled;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public bool IsGroupingEnabled
    {
        get => _isGroupingEnabled;
        set
        {
            if (SetProperty(ref _isGroupingEnabled, value))
            {
                ApplyGrouping();
            }
        }
    }

    public ICollectionView ProductsView { get; private set; }

    public ProductsViewModel()
    {
        Products = new ObservableCollection<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Électronique" },
            new Product { Id = 2, Name = "Smartphone", Price = 699.99m, Category = "Électronique" },
            new Product { Id = 3, Name = "Chaise de bureau", Price = 199.99m, Category = "Mobilier" },
            new Product { Id = 4, Name = "Écran 27\"", Price = 349.99m, Category = "Électronique" },
            new Product { Id = 5, Name = "Bureau", Price = 299.99m, Category = "Mobilier" },
            new Product { Id = 6, Name = "Clavier", Price = 89.99m, Category = "Électronique" },
            new Product { Id = 7, Name = "Souris", Price = 59.99m, Category = "Électronique" },
            new Product { Id = 8, Name = "Étagère", Price = 149.99m, Category = "Mobilier" }
        };

        ProductsView = CollectionViewSource.GetDefaultView(Products);
    }

    private void ApplyGrouping()
    {
        ProductsView.GroupDescriptions.Clear();

        if (IsGroupingEnabled)
        {
            ProductsView.GroupDescriptions.Add(new PropertyGroupDescription("Category"));
        }
    }
}
```


Interface pour le groupement :

```xml
<Window.Resources>
    <!-- Template pour les en-têtes de groupe -->
    <DataTemplate x:Key="GroupHeaderTemplate">
        <Expander IsExpanded="True">
            <Expander.Header>
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="{Binding Name}" FontWeight="Bold" FontSize="14"/>
                    <TextBlock Text="{Binding ItemCount, StringFormat= ({0} articles)}"
                               Margin="10,0,0,0" Foreground="#666"/>
                </StackPanel>
            </Expander.Header>
        </Expander>
    </DataTemplate>
</Window.Resources>

<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Option de groupement -->
    <CheckBox Content="Grouper par catégorie"
              IsChecked="{Binding IsGroupingEnabled}"
              Margin="10"/>

    <!-- Liste des produits groupés ou non -->
    <ListView Grid.Row="1" ItemsSource="{Binding ProductsView}" Margin="10">
        <ListView.GroupStyle>
            <GroupStyle HeaderTemplate="{StaticResource GroupHeaderTemplate}"/>
        </ListView.GroupStyle>
        <ListView.View>
            <GridView>
                <GridViewColumn Header="Nom" DisplayMemberBinding="{Binding Name}" Width="150"/>
                <GridViewColumn Header="Catégorie" DisplayMemberBinding="{Binding Category}" Width="100"/>
                <GridViewColumn Header="Prix" Width="100">
                    <GridViewColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" HorizontalAlignment="Right"/>
                        </DataTemplate>
                    </GridViewColumn.CellTemplate>
                </GridViewColumn>
            </GridView>
        </ListView.View>
    </ListView>
</Grid>
```


### Navigation dans une CollectionView

```textmate
public class ProductsViewModel : ViewModelBase
{
    private ObservableCollection<Product> _products;

    public ObservableCollection<Product> Products
    {
        get => _products;
        set => SetProperty(ref _products, value);
    }

    public ICollectionView ProductsView { get; private set; }

    // Propriété pour l'élément courant
    public Product CurrentProduct
    {
        get => ProductsView.CurrentItem as Product;
        set
        {
            if (value != null)
            {
                ProductsView.MoveCurrentTo(value);
                OnPropertyChanged();
            }
        }
    }

    // Commandes de navigation
    public ICommand NextCommand => new RelayCommand(
        () => ProductsView.MoveCurrentToNext(),
        () => ProductsView.CurrentPosition < ProductsView.Cast<object>().Count() - 1
    );

    public ICommand PreviousCommand => new RelayCommand(
        () => ProductsView.MoveCurrentToPrevious(),
        () => ProductsView.CurrentPosition > 0
    );

    public ICommand FirstCommand => new RelayCommand(
        () => ProductsView.MoveCurrentToFirst(),
        () => ProductsView.CurrentPosition > 0
    );

    public ICommand LastCommand => new RelayCommand(
        () => ProductsView.MoveCurrentToLast(),
        () => ProductsView.CurrentPosition < ProductsView.Cast<object>().Count() - 1
    );

    public ProductsViewModel()
    {
        Products = new ObservableCollection<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Électronique" },
            new Product { Id = 2, Name = "Smartphone", Price = 699.99m, Category = "Électronique" },
            new Product { Id = 3, Name = "Chaise de bureau", Price = 199.99m, Category = "Mobilier" },
            new Product { Id = 4, Name = "Écran 27\"", Price = 349.99m, Category = "Électronique" },
            new Product { Id = 5, Name = "Bureau", Price = 299.99m, Category = "Mobilier" }
        };

        ProductsView = CollectionViewSource.GetDefaultView(Products);

        // S'abonner aux changements de l'élément courant
        ProductsView.CurrentChanged += (s, e) =>
        {
            OnPropertyChanged(nameof(CurrentProduct));
            // Mettre à jour la disponibilité des commandes
            CommandManager.InvalidateRequerySuggested();
        };
    }
}
```


Interface pour naviguer dans une CollectionView :

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>

    <!-- Liste des produits -->
    <ListView ItemsSource="{Binding ProductsView}"
              SelectedItem="{Binding CurrentProduct}"
              Margin="10">
        <ListView.View>
            <GridView>
                <GridViewColumn Header="Nom" DisplayMemberBinding="{Binding Name}" Width="150"/>
                <GridViewColumn Header="Catégorie" DisplayMemberBinding="{Binding Category}" Width="100"/>
                <GridViewColumn Header="Prix" Width="100">
                    <GridViewColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Price, StringFormat=\{0:C\}}" HorizontalAlignment="Right"/>
                        </DataTemplate>
                    </GridViewColumn.CellTemplate>
                </GridViewColumn>
            </GridView>
        </ListView.View>
    </ListView>

    <!-- Détails du produit sélectionné -->
    <GroupBox Grid.Row="1" Header="Détails du produit" Margin="10">
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

            <TextBlock Text="ID:" Grid.Row="0" Grid.Column="0" Margin="0,0,10,5"/>
            <TextBlock Text="{Binding CurrentProduct.Id}" Grid.Row="0" Grid.Column="1" Margin="0,0,0,5"/>

            <TextBlock Text="Nom:" Grid.Row="1" Grid.Column="0" Margin="0,0,10,5"/>
            <TextBlock Text="{Binding CurrentProduct.Name}" Grid.Row="1" Grid.Column="1" Margin="0,0,0,5"/>

            <TextBlock Text="Catégorie:" Grid.Row="2" Grid.Column="0" Margin="0,0,10,5"/>
            <TextBlock Text="{Binding CurrentProduct.Category}" Grid.Row="2" Grid.Column="1" Margin="0,0,0,5"/>

            <TextBlock Text="Prix:" Grid.Row="3" Grid.Column="0" Margin="0,0,10,5"/>
            <TextBlock Text="{Binding CurrentProduct.Price, StringFormat=\{0:C\}}" Grid.Row="3" Grid.Column="1"/>
        </Grid>
    </GroupBox>

    <!-- Boutons de navigation -->
    <StackPanel Grid.Row="2" Orientation="Horizontal" HorizontalAlignment="Center" Margin="10">
        <Button Content="Premier" Command="{Binding FirstCommand}" Width="70" Margin="5"/>
        <Button Content="Précédent" Command="{Binding PreviousCommand}" Width="70" Margin="5"/>
        <Button Content="Suivant" Command="{Binding NextCommand}" Width="70" Margin="5"/>
        <Button Content="Dernier" Command="{Binding LastCommand}" Width="70" Margin="5"/>
    </StackPanel>
</Grid>
```


Les bindings et le DataContext sont des concepts fondamentaux en WPF qui permettent de créer des applications réactives suivant le pattern MVVM. Ils offrent une séparation claire entre l'interface utilisateur et la logique métier, facilitant la maintenance et les tests unitaires.

⏭️ 8.7. [Commandes et MVVM (Model-View-ViewModel)](/08-windows-presentation-foundation-wpf/8-07-commandes-et-mvvm-model-view-viewmodel.md)
