# 8.7. Commandes et MVVM (Model-View-ViewModel)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le pattern MVVM (Model-View-ViewModel) est l'un des patterns architecturaux les plus populaires pour développer des applications WPF. Il permet de séparer clairement la logique métier (Model), l'interface utilisateur (View) et la logique de présentation (ViewModel), ce qui rend le code plus maintenable, testable et évolutif.

## 8.7.1. Pattern MVVM

### Principes fondamentaux du MVVM

Le pattern MVVM repose sur trois composants principaux :

1. **Model (Modèle)** : Représente les données et la logique métier
2. **View (Vue)** : Représente l'interface utilisateur (fenêtres, contrôles, etc.)
3. **ViewModel (Modèle de Vue)** : Fait le lien entre le Model et la View

![Schéma MVVM](https://i.imgur.com/QFRnqJ3.png)

### Avantages du MVVM

- **Séparation des préoccupations** : Chaque composant a une responsabilité bien définie
- **Testabilité** : Les ViewModels peuvent être testés indépendamment de l'interface utilisateur
- **Réutilisabilité** : Les ViewModels peuvent être réutilisés pour différentes vues
- **Facilité de maintenance** : Les modifications apportées à l'interface utilisateur n'affectent pas la logique métier

### Exemple simple de structure MVVM

#### Model (Données et logique métier)

```textmate
// Classe de modèle simple
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime BirthDate { get; set; }

    public string FullName => $"{FirstName} {LastName}";
    public int Age => DateTime.Now.Year - BirthDate.Year -
        (DateTime.Now.DayOfYear < BirthDate.DayOfYear ? 1 : 0);
}

// Service ou repository pour accéder aux données
public class PeopleService
{
    public List<Person> GetPeople()
    {
        // Dans un cas réel, cela pourrait être une requête à une base de données
        return new List<Person>
        {
            new Person { FirstName = "Jean", LastName = "Dupont", BirthDate = new DateTime(1985, 5, 15) },
            new Person { FirstName = "Marie", LastName = "Martin", BirthDate = new DateTime(1990, 8, 22) }
        };
    }

    public void SavePerson(Person person)
    {
        // Logique pour sauvegarder une personne
    }
}
```


#### ViewModel (Logique de présentation)

```textmate
public class PersonViewModel : INotifyPropertyChanged
{
    private readonly PeopleService _peopleService;
    private Person _selectedPerson;

    public PersonViewModel(PeopleService peopleService)
    {
        _peopleService = peopleService;
        LoadPeople();
    }

    public ObservableCollection<Person> People { get; private set; }

    public Person SelectedPerson
    {
        get => _selectedPerson;
        set
        {
            _selectedPerson = value;
            OnPropertyChanged();
        }
    }

    private void LoadPeople()
    {
        var people = _peopleService.GetPeople();
        People = new ObservableCollection<Person>(people);

        if (People.Any())
        {
            SelectedPerson = People.First();
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


#### View (Interface utilisateur)

```xml
<Window x:Class="MvvmSample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MVVM Demo" Height="350" Width="500">
    <Grid Margin="10">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Liste des personnes -->
        <ListBox Grid.Column="0"
                 ItemsSource="{Binding People}"
                 SelectedItem="{Binding SelectedPerson}"
                 DisplayMemberPath="FullName"
                 Margin="0,0,5,0"/>

        <!-- Détails de la personne sélectionnée -->
        <StackPanel Grid.Column="1" Margin="5,0,0,0">
            <TextBlock Text="Détails de la personne" FontWeight="Bold" Margin="0,0,0,10"/>

            <TextBlock Text="Prénom:"/>
            <TextBox Text="{Binding SelectedPerson.FirstName, UpdateSourceTrigger=PropertyChanged}"
                     Margin="0,0,0,10"/>

            <TextBlock Text="Nom:"/>
            <TextBox Text="{Binding SelectedPerson.LastName, UpdateSourceTrigger=PropertyChanged}"
                     Margin="0,0,0,10"/>

            <TextBlock Text="Date de naissance:"/>
            <DatePicker SelectedDate="{Binding SelectedPerson.BirthDate}"
                        Margin="0,0,0,10"/>

            <TextBlock Text="Âge:"/>
            <TextBlock Text="{Binding SelectedPerson.Age}" Margin="0,0,0,10"/>
        </StackPanel>
    </Grid>
</Window>
```


Code-behind minimal :

```textmate
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        // Création des services et du ViewModel
        var peopleService = new PeopleService();
        var viewModel = new PersonViewModel(peopleService);

        // Définition du DataContext
        DataContext = viewModel;
    }
}
```


## 8.7.2. ICommand

L'interface `ICommand` est un élément clé du pattern MVVM qui permet de lier des actions de l'interface utilisateur (comme des clics de bouton) à des méthodes du ViewModel sans code-behind.

### Définition de l'interface ICommand

```textmate
public interface ICommand
{
    // Détermine si la commande peut être exécutée
    bool CanExecute(object parameter);

    // Exécute la commande
    void Execute(object parameter);

    // Événement déclenché lorsque CanExecute peut changer
    event EventHandler CanExecuteChanged;
}
```


### Utilisation de ICommand en XAML

```xml
<Button Content="Enregistrer"
        Command="{Binding SaveCommand}"
        CommandParameter="{Binding SelectedPerson}"/>
```


### Implémentation d'une commande personnalisée

```textmate
public class SaveCommand : ICommand
{
    private readonly Action<Person> _saveAction;
    private readonly Func<Person, bool> _canSaveFunc;

    public SaveCommand(Action<Person> saveAction, Func<Person, bool> canSaveFunc = null)
    {
        _saveAction = saveAction ?? throw new ArgumentNullException(nameof(saveAction));
        _canSaveFunc = canSaveFunc;
    }

    public bool CanExecute(object parameter)
    {
        if (_canSaveFunc == null)
            return true;

        return parameter is Person person && _canSaveFunc(person);
    }

    public void Execute(object parameter)
    {
        if (parameter is Person person)
        {
            _saveAction(person);
        }
    }

    public event EventHandler CanExecuteChanged;

    public void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}
```


Utilisation dans le ViewModel :

```textmate
public class PersonViewModel : INotifyPropertyChanged
{
    private readonly PeopleService _peopleService;
    private Person _selectedPerson;

    // Commande pour enregistrer
    public ICommand SaveCommand { get; }

    public PersonViewModel(PeopleService peopleService)
    {
        _peopleService = peopleService;
        LoadPeople();

        // Initialisation de la commande
        SaveCommand = new SaveCommand(
            // Action d'enregistrement
            person => {
                _peopleService.SavePerson(person);
                MessageBox.Show("Personne enregistrée avec succès !");
            },
            // Condition d'activation
            person => person != null && !string.IsNullOrEmpty(person.FirstName) &&
                     !string.IsNullOrEmpty(person.LastName)
        );
    }

    // Reste du code ViewModel...
}
```


## 8.7.3. RelayCommand

L'implémentation d'une commande personnalisée étant fastidieuse, on utilise généralement une classe générique `RelayCommand` (ou `DelegateCommand` dans certains frameworks) qui simplifie ce processus.

### Implémentation de base de RelayCommand

```textmate
public class RelayCommand : ICommand
{
    private readonly Action<object> _execute;
    private readonly Predicate<object> _canExecute;

    public RelayCommand(Action<object> execute, Predicate<object> canExecute = null)
    {
        _execute = execute ?? throw new ArgumentNullException(nameof(execute));
        _canExecute = canExecute;
    }

    public bool CanExecute(object parameter)
    {
        return _canExecute == null || _canExecute(parameter);
    }

    public void Execute(object parameter)
    {
        _execute(parameter);
    }

    public event EventHandler CanExecuteChanged
    {
        add { CommandManager.RequerySuggested += value; }
        remove { CommandManager.RequerySuggested -= value; }
    }
}
```


### RelayCommand générique (pour typer le paramètre)

```textmate
public class RelayCommand<T> : ICommand
{
    private readonly Action<T> _execute;
    private readonly Predicate<T> _canExecute;

    public RelayCommand(Action<T> execute, Predicate<T> canExecute = null)
    {
        _execute = execute ?? throw new ArgumentNullException(nameof(execute));
        _canExecute = canExecute;
    }

    public bool CanExecute(object parameter)
    {
        if (parameter == null && typeof(T).IsValueType)
            return false;

        return _canExecute == null || _canExecute((T)parameter);
    }

    public void Execute(object parameter)
    {
        _execute((T)parameter);
    }

    public event EventHandler CanExecuteChanged
    {
        add { CommandManager.RequerySuggested += value; }
        remove { CommandManager.RequerySuggested -= value; }
    }
}
```


### Utilisation de RelayCommand dans un ViewModel

```textmate
public class PersonViewModel : INotifyPropertyChanged
{
    private readonly PeopleService _peopleService;
    private Person _selectedPerson;

    // Commandes
    public ICommand SaveCommand { get; }
    public ICommand DeleteCommand { get; }
    public ICommand AddCommand { get; }

    public PersonViewModel(PeopleService peopleService)
    {
        _peopleService = peopleService;
        LoadPeople();

        // Initialisation des commandes
        SaveCommand = new RelayCommand<Person>(
            // Action d'enregistrement
            person => {
                _peopleService.SavePerson(person);
                MessageBox.Show("Personne enregistrée avec succès !");
            },
            // Condition d'activation
            person => person != null && !string.IsNullOrEmpty(person.FirstName) &&
                     !string.IsNullOrEmpty(person.LastName)
        );

        DeleteCommand = new RelayCommand<Person>(
            // Action de suppression
            person => {
                People.Remove(person);
                if (People.Any())
                    SelectedPerson = People.First();
            },
            // Condition d'activation
            person => person != null
        );

        AddCommand = new RelayCommand(
            // Action d'ajout
            _ => {
                var newPerson = new Person { FirstName = "Nouveau", LastName = "Contact" };
                People.Add(newPerson);
                SelectedPerson = newPerson;
            }
        );
    }

    // Reste du code ViewModel...
}
```


Interface utilisateur utilisant ces commandes :

```xml
<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>

    <!-- Liste des personnes -->
    <ListBox Grid.Row="0" Grid.Column="0"
             ItemsSource="{Binding People}"
             SelectedItem="{Binding SelectedPerson}"
             DisplayMemberPath="FullName"
             Margin="0,0,5,0"/>

    <!-- Détails de la personne sélectionnée -->
    <StackPanel Grid.Row="0" Grid.Column="1" Margin="5,0,0,0">
        <TextBlock Text="Détails de la personne" FontWeight="Bold" Margin="0,0,0,10"/>

        <TextBlock Text="Prénom:"/>
        <TextBox Text="{Binding SelectedPerson.FirstName, UpdateSourceTrigger=PropertyChanged}"
                 Margin="0,0,0,10"/>

        <TextBlock Text="Nom:"/>
        <TextBox Text="{Binding SelectedPerson.LastName, UpdateSourceTrigger=PropertyChanged}"
                 Margin="0,0,0,10"/>

        <TextBlock Text="Date de naissance:"/>
        <DatePicker SelectedDate="{Binding SelectedPerson.BirthDate}"
                    Margin="0,0,0,10"/>

        <TextBlock Text="Âge:"/>
        <TextBlock Text="{Binding SelectedPerson.Age}" Margin="0,0,0,10"/>
    </StackPanel>

    <!-- Boutons d'action -->
    <StackPanel Grid.Row="1" Grid.Column="0" Grid.ColumnSpan="2"
                Orientation="Horizontal" HorizontalAlignment="Center" Margin="0,10,0,0">
        <Button Content="Ajouter"
                Command="{Binding AddCommand}"
                Width="100" Margin="5"/>

        <Button Content="Enregistrer"
                Command="{Binding SaveCommand}"
                CommandParameter="{Binding SelectedPerson}"
                Width="100" Margin="5"/>

        <Button Content="Supprimer"
                Command="{Binding DeleteCommand}"
                CommandParameter="{Binding SelectedPerson}"
                Width="100" Margin="5"/>
    </StackPanel>
</Grid>
```


### AsyncRelayCommand pour .NET 8

Pour les opérations asynchrones, .NET 8 prend en charge les commandes asynchrones :

```textmate
public class AsyncRelayCommand : ICommand
{
    private readonly Func<object, Task> _executeAsync;
    private readonly Predicate<object> _canExecute;
    private bool _isExecuting;

    public AsyncRelayCommand(Func<object, Task> executeAsync, Predicate<object> canExecute = null)
    {
        _executeAsync = executeAsync ?? throw new ArgumentNullException(nameof(executeAsync));
        _canExecute = canExecute;
    }

    public bool CanExecute(object parameter)
    {
        return !_isExecuting && (_canExecute == null || _canExecute(parameter));
    }

    public async void Execute(object parameter)
    {
        if (!CanExecute(parameter))
            return;

        try
        {
            _isExecuting = true;
            RaiseCanExecuteChanged();

            await _executeAsync(parameter);
        }
        finally
        {
            _isExecuting = false;
            RaiseCanExecuteChanged();
        }
    }

    public event EventHandler CanExecuteChanged;

    protected void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}
```


Utilisation dans un ViewModel :

```textmate
public class ProductViewModel : INotifyPropertyChanged
{
    private readonly IProductService _productService;
    private bool _isLoading;

    public ObservableCollection<Product> Products { get; } = new ObservableCollection<Product>();

    public bool IsLoading
    {
        get => _isLoading;
        set
        {
            _isLoading = value;
            OnPropertyChanged();
        }
    }

    public ICommand LoadProductsCommand { get; }

    public ProductViewModel(IProductService productService)
    {
        _productService = productService;

        LoadProductsCommand = new AsyncRelayCommand(
            async _ => {
                IsLoading = true;

                try
                {
                    var products = await _productService.GetProductsAsync();

                    Products.Clear();
                    foreach (var product in products)
                    {
                        Products.Add(product);
                    }
                }
                finally
                {
                    IsLoading = false;
                }
            }
        );
    }

    // Reste du code ViewModel...
}
```


## 8.7.4. Frameworks MVVM (MVVM Light, Prism, etc.)

Plusieurs frameworks ont été développés pour faciliter l'implémentation du pattern MVVM. Voici les plus populaires :

### MVVM Light / Community Toolkit MVVM

Le toolkit MVVM Light est l'un des frameworks MVVM les plus populaires pour WPF. Il a été remplacé par Community Toolkit MVVM pour .NET 8.

Pour l'installer via NuGet :
```
dotnet add package CommunityToolkit.Mvvm
```


Exemple d'utilisation :

```textmate
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

// ViewModel avec ObservableObject et commandes générées automatiquement
public partial class PersonViewModel : ObservableObject
{
    private readonly PeopleService _peopleService;

    // Génère automatiquement la propriété avec notification de changement
    [ObservableProperty]
    private ObservableCollection<Person> people;

    [ObservableProperty]
    private Person selectedPerson;

    // Génère automatiquement une commande IRelayCommand
    [RelayCommand]
    private void Save(Person person)
    {
        if (person == null) return;

        _peopleService.SavePerson(person);
        MessageBox.Show("Personne enregistrée avec succès !");
    }

    [RelayCommand]
    private void Delete(Person person)
    {
        if (person == null) return;

        People.Remove(person);
        if (People.Any())
            SelectedPerson = People.First();
    }

    [RelayCommand]
    private void Add()
    {
        var newPerson = new Person { FirstName = "Nouveau", LastName = "Contact" };
        People.Add(newPerson);
        SelectedPerson = newPerson;
    }

    public PersonViewModel(PeopleService peopleService)
    {
        _peopleService = peopleService;

        // Initialisation des données
        People = new ObservableCollection<Person>(_peopleService.GetPeople());

        if (People.Any())
        {
            SelectedPerson = People.First();
        }
    }
}
```


Utilisations en XAML :

```xml
<Button Content="Ajouter" Command="{Binding AddCommand}" />
<Button Content="Enregistrer" Command="{Binding SaveCommand}" CommandParameter="{Binding SelectedPerson}" />
<Button Content="Supprimer" Command="{Binding DeleteCommand}" CommandParameter="{Binding SelectedPerson}" />
```


### Prism

Prism est un framework complet pour développer des applications modulaires en suivant le pattern MVVM. Il offre de nombreuses fonctionnalités comme le modèle composant-événement, la navigation, et l'injection de dépendances.

Pour l'installer via NuGet :
```
dotnet add package Prism.WPF
dotnet add package Prism.DryIoc # ou Prism.Unity pour utiliser Unity comme conteneur DI
```


Exemple d'application Prism de base :

```textmate
// Application principale
public class App : PrismApplication
{
    protected override Window CreateShell()
    {
        return Container.Resolve<MainWindow>();
    }

    protected override void RegisterTypes(IContainerRegistry containerRegistry)
    {
        // Enregistrement des services et des vues
        containerRegistry.RegisterSingleton<IPeopleService, PeopleService>();
        containerRegistry.RegisterForNavigation<PersonView, PersonViewModel>();
    }
}

// ViewModel avec DelegateCommand de Prism
public class PersonViewModel : BindableBase
{
    private readonly IPeopleService _peopleService;

    private ObservableCollection<Person> _people;
    public ObservableCollection<Person> People
    {
        get => _people;
        set => SetProperty(ref _people, value);
    }

    private Person _selectedPerson;
    public Person SelectedPerson
    {
        get => _selectedPerson;
        set => SetProperty(ref _selectedPerson, value);
    }

    // Commandes Prism
    public DelegateCommand<Person> SaveCommand { get; }
    public DelegateCommand<Person> DeleteCommand { get; }
    public DelegateCommand AddCommand { get; }

    public PersonViewModel(IPeopleService peopleService)
    {
        _peopleService = peopleService;

        // Initialisation des données
        People = new ObservableCollection<Person>(_peopleService.GetPeople());

        if (People.Any())
        {
            SelectedPerson = People.First();
        }

        // Initialisation des commandes
        SaveCommand = new DelegateCommand<Person>(Save, CanSave)
            .ObservesProperty(() => SelectedPerson);

        DeleteCommand = new DelegateCommand<Person>(Delete, CanDelete)
            .ObservesProperty(() => SelectedPerson);

        AddCommand = new DelegateCommand(Add);
    }

    private void Save(Person person)
    {
        _peopleService.SavePerson(person);
        MessageBox.Show("Personne enregistrée avec succès !");
    }

    private bool CanSave(Person person)
    {
        return person != null &&
               !string.IsNullOrEmpty(person.FirstName) &&
               !string.IsNullOrEmpty(person.LastName);
    }

    private void Delete(Person person)
    {
        People.Remove(person);
        if (People.Any())
            SelectedPerson = People.First();
    }

    private bool CanDelete(Person person)
    {
        return person != null;
    }

    private void Add()
    {
        var newPerson = new Person { FirstName = "Nouveau", LastName = "Contact" };
        People.Add(newPerson);
        SelectedPerson = newPerson;
    }
}
```


### ReactiveUI

ReactiveUI est un framework MVVM qui utilise la programmation réactive (Reactive Extensions - Rx) pour gérer les flux d'événements.

Pour l'installer via NuGet :
```
dotnet add package ReactiveUI.WPF
```


Exemple d'utilisation :

```textmate
public class PersonViewModel : ReactiveObject
{
    private readonly IPeopleService _peopleService;

    // Propriétés réactives
    private ObservableCollection<Person> _people;
    public ObservableCollection<Person> People
    {
        get => _people;
        set => this.RaiseAndSetIfChanged(ref _people, value);
    }

    private Person _selectedPerson;
    public Person SelectedPerson
    {
        get => _selectedPerson;
        set => this.RaiseAndSetIfChanged(ref _selectedPerson, value);
    }

    // Commandes réactives
    public ReactiveCommand<Person, Unit> SaveCommand { get; }
    public ReactiveCommand<Person, Unit> DeleteCommand { get; }
    public ReactiveCommand<Unit, Unit> AddCommand { get; }

    public PersonViewModel(IPeopleService peopleService)
    {
        _peopleService = peopleService;

        // Initialisation des données
        People = new ObservableCollection<Person>(_peopleService.GetPeople());

        if (People.Any())
        {
            SelectedPerson = People.First();
        }

        // Conditions de validation pour les commandes
        var canSave = this.WhenAnyValue(
            x => x.SelectedPerson,
            person => person != null &&
                     !string.IsNullOrEmpty(person.FirstName) &&
                     !string.IsNullOrEmpty(person.LastName)
        );

        var canDelete = this.WhenAnyValue(
            x => x.SelectedPerson,
            person => person != null
        );

        // Initialisation des commandes
        SaveCommand = ReactiveCommand.Create<Person>(Save, canSave);
        DeleteCommand = ReactiveCommand.Create<Person>(Delete, canDelete);
        AddCommand = ReactiveCommand.Create(Add);
    }

    private void Save(Person person)
    {
        _peopleService.SavePerson(person);
        MessageBox.Show("Personne enregistrée avec succès !");
    }

    private void Delete(Person person)
    {
        People.Remove(person);
        if (People.Any())
            SelectedPerson = People.First();
    }

    private void Add()
    {
        var newPerson = new Person { FirstName = "Nouveau", LastName = "Contact" };
        People.Add(newPerson);
        SelectedPerson = newPerson;
    }
}
```


## 8.7.5. Injection de dépendances dans MVVM

L'injection de dépendances (DI) est une technique qui permet de rendre les composants plus découplés, testables et maintenables. Elle est très utile dans le pattern MVVM.

### Principe de base

Au lieu de créer des dépendances directement dans une classe, on les injecte via le constructeur ou des propriétés.

```textmate
// Sans injection de dépendances
public class PersonViewModel
{
    private readonly PeopleService _peopleService = new PeopleService();

    // Reste du code...
}

// Avec injection de dépendances
public class PersonViewModel
{
    private readonly IPeopleService _peopleService;

    public PersonViewModel(IPeopleService peopleService)
    {
        _peopleService = peopleService;
    }

    // Reste du code...
}
```


### Configuration avec Microsoft.Extensions.DependencyInjection (.NET 8)

Pour .NET 8, Microsoft fournit un conteneur d'injection de dépendances intégré :

```textmate
// Installation du package
// dotnet add package Microsoft.Extensions.DependencyInjection

using Microsoft.Extensions.DependencyInjection;

public partial class App : Application
{
    private ServiceProvider _serviceProvider;

    public App()
    {
        var services = new ServiceCollection();
        ConfigureServices(services);
        _serviceProvider = services.BuildServiceProvider();
    }

    private void ConfigureServices(ServiceCollection services)
    {
        // Enregistrement des services
        services.AddSingleton<IPeopleService, PeopleService>();
        services.AddSingleton<IDialogService, DialogService>();

        // Enregistrement des ViewModels
        services.AddTransient<PersonViewModel>();

        // Enregistrement de la fenêtre principale
        services.AddTransient<MainWindow>();
    }

    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        // Résolution de la fenêtre principale
        var mainWindow = _serviceProvider.GetRequiredService<MainWindow>();
        mainWindow.Show();
    }

    protected override void OnExit(ExitEventArgs e)
    {
        // Libération des ressources
        _serviceProvider.Dispose();

        base.OnExit(e);
    }
}
```


Utilisation dans le code-behind d'une fenêtre :

```textmate
public partial class MainWindow : Window
{
    public MainWindow(PersonViewModel viewModel)
    {
        InitializeComponent();

        // Le ViewModel est injecté automatiquement
        DataContext = viewModel;
    }
}
```

### Localisateur de services (Service Locator)

Une approche alternative à l'injection de dépendances directe est le modèle "Service Locator", bien que cette approche soit moins recommandée de nos jours :

```textmate
public static class ServiceLocator
{
    private static readonly Dictionary<Type, object> _services = new Dictionary<Type, object>();

    public static void Register<T>(T service) where T : class
    {
        _services[typeof(T)] = service;
    }

    public static T Resolve<T>() where T : class
    {
        if (_services.TryGetValue(typeof(T), out var service))
        {
            return (T)service;
        }

        throw new InvalidOperationException($"Le service {typeof(T).Name} n'est pas enregistré.");
    }
}

// Enregistrement des services
public partial class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        // Enregistrement des services
        ServiceLocator.Register<IPeopleService>(new PeopleService());
        ServiceLocator.Register<IDialogService>(new DialogService());

        // Affichage de la fenêtre principale
        var mainWindow = new MainWindow();
        mainWindow.Show();
    }
}

// Résolution des services dans un ViewModel
public class PersonViewModel : INotifyPropertyChanged
{
    private readonly IPeopleService _peopleService;
    private readonly IDialogService _dialogService;

    public PersonViewModel()
    {
        // Résolution des services via le Service Locator
        _peopleService = ServiceLocator.Resolve<IPeopleService>();
        _dialogService = ServiceLocator.Resolve<IDialogService>();
    }

    // Reste du code...
}
```


### Injection de dépendances avec Factory Pattern

Un pattern de fabrique peut également être utilisé pour créer des ViewModels avec leurs dépendances :

```textmate
// Interface pour la fabrique de ViewModels
public interface IViewModelFactory
{
    T Create<T>() where T : class;
}

// Implémentation de la fabrique
public class ViewModelFactory : IViewModelFactory
{
    private readonly IPeopleService _peopleService;
    private readonly IDialogService _dialogService;

    public ViewModelFactory(IPeopleService peopleService, IDialogService dialogService)
    {
        _peopleService = peopleService;
        _dialogService = dialogService;
    }

    public T Create<T>() where T : class
    {
        if (typeof(T) == typeof(PersonViewModel))
        {
            return new PersonViewModel(_peopleService, _dialogService) as T;
        }

        if (typeof(T) == typeof(SettingsViewModel))
        {
            return new SettingsViewModel(_dialogService) as T;
        }

        throw new ArgumentException($"Type de ViewModel non supporté : {typeof(T).Name}");
    }
}

// Utilisation dans la fenêtre
public partial class MainWindow : Window
{
    private readonly IViewModelFactory _viewModelFactory;

    public MainWindow(IViewModelFactory viewModelFactory)
    {
        InitializeComponent();

        _viewModelFactory = viewModelFactory;
        DataContext = _viewModelFactory.Create<PersonViewModel>();
    }

    private void OnSettingsClick(object sender, RoutedEventArgs e)
    {
        var settingsWindow = new SettingsWindow
        {
            DataContext = _viewModelFactory.Create<SettingsViewModel>()
        };

        settingsWindow.ShowDialog();
    }
}
```


### Utilisation de containers IoC avancés

Pour des applications plus complexes, vous pouvez utiliser des conteneurs d'inversion de contrôle (IoC) plus avancés comme Autofac, DryIoc ou Unity.

Exemple avec Autofac (.NET 8) :

```textmate
// Installation du package
// dotnet add package Autofac
// dotnet add package Autofac.Extensions.DependencyInjection

using Autofac;
using Autofac.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;

public partial class App : Application
{
    private IContainer _container;

    public App()
    {
        var services = new ServiceCollection();

        // Configuration d'Autofac
        var builder = new ContainerBuilder();

        // Enregistrement des services
        builder.RegisterType<PeopleService>().As<IPeopleService>().SingleInstance();
        builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();

        // Enregistrement des ViewModels
        builder.RegisterType<PersonViewModel>();
        builder.RegisterType<SettingsViewModel>();

        // Enregistrement des fenêtres
        builder.RegisterType<MainWindow>();
        builder.RegisterType<SettingsWindow>();

        // Intégration avec Microsoft.Extensions.DependencyInjection
        builder.Populate(services);

        _container = builder.Build();
    }

    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        // Résolution de la fenêtre principale
        var mainWindow = _container.Resolve<MainWindow>();
        mainWindow.Show();
    }

    protected override void OnExit(ExitEventArgs e)
    {
        // Libération des ressources
        _container.Dispose();

        base.OnExit(e);
    }
}
```


### Services d'application courants dans MVVM

Voici quelques services couramment utilisés dans les applications MVVM :

#### Service de dialogue

```textmate
// Interface
public interface IDialogService
{
    bool ShowConfirmation(string message, string title = "Confirmation");
    void ShowInformation(string message, string title = "Information");
    void ShowWarning(string message, string title = "Avertissement");
    void ShowError(string message, string title = "Erreur");
    string ShowInput(string prompt, string title = "Saisie", string defaultValue = "");
    T ShowDialog<T>(object viewModel = null) where T : Window, new();
}

// Implémentation
public class DialogService : IDialogService
{
    public bool ShowConfirmation(string message, string title = "Confirmation")
    {
        return MessageBox.Show(message, title, MessageBoxButton.YesNo) == MessageBoxResult.Yes;
    }

    public void ShowInformation(string message, string title = "Information")
    {
        MessageBox.Show(message, title, MessageBoxButton.OK, MessageBoxImage.Information);
    }

    public void ShowWarning(string message, string title = "Avertissement")
    {
        MessageBox.Show(message, title, MessageBoxButton.OK, MessageBoxImage.Warning);
    }

    public void ShowError(string message, string title = "Erreur")
    {
        MessageBox.Show(message, title, MessageBoxButton.OK, MessageBoxImage.Error);
    }

    public string ShowInput(string prompt, string title = "Saisie", string defaultValue = "")
    {
        var inputDialog = new InputDialog(prompt, title, defaultValue);

        if (inputDialog.ShowDialog() == true)
        {
            return inputDialog.Answer;
        }

        return null;
    }

    public T ShowDialog<T>(object viewModel = null) where T : Window, new()
    {
        var dialog = new T();

        if (viewModel != null)
        {
            dialog.DataContext = viewModel;
        }

        dialog.ShowDialog();
        return dialog;
    }
}
```


#### Service de navigation

```textmate
// Interface
public interface INavigationService
{
    void NavigateTo<T>(object parameter = null) where T : class;
    void GoBack();
    bool CanGoBack { get; }
}

// Implémentation simple pour application à page unique avec régions
public class NavigationService : INavigationService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ContentControl _contentRegion;
    private readonly Stack<(UserControl View, object Parameter)> _navigationStack = new Stack<(UserControl, object)>();

    public NavigationService(IServiceProvider serviceProvider, ContentControl contentRegion)
    {
        _serviceProvider = serviceProvider;
        _contentRegion = contentRegion;
    }

    public void NavigateTo<T>(object parameter = null) where T : class
    {
        // Créer la vue
        var view = _serviceProvider.GetService<T>() as UserControl;

        if (view == null)
        {
            throw new InvalidOperationException($"La vue {typeof(T).Name} n'a pas pu être résolue ou n'est pas un UserControl.");
        }

        // Si nous avons déjà une vue, la sauvegarder dans la pile
        if (_contentRegion.Content is UserControl currentView)
        {
            _navigationStack.Push((currentView, null));
        }

        // Naviguer vers la nouvelle vue
        _contentRegion.Content = view;

        // Injecter le paramètre si le ViewModel implémente INavigationAware
        if (view.DataContext is INavigationAware navigationAware)
        {
            navigationAware.OnNavigatedTo(parameter);
        }
    }

    public void GoBack()
    {
        if (!CanGoBack)
        {
            return;
        }

        // Récupérer la vue précédente
        var (previousView, parameter) = _navigationStack.Pop();

        // Naviguer vers la vue précédente
        _contentRegion.Content = previousView;

        // Notifier le ViewModel
        if (previousView.DataContext is INavigationAware navigationAware)
        {
            navigationAware.OnNavigatedTo(parameter);
        }
    }

    public bool CanGoBack => _navigationStack.Count > 0;
}

// Interface pour les ViewModels qui doivent être notifiés de la navigation
public interface INavigationAware
{
    void OnNavigatedTo(object parameter);
    void OnNavigatedFrom();
}
```


#### Service d'état d'application

```textmate
// Interface
public interface IApplicationStateService
{
    T GetState<T>(string key, T defaultValue = default);
    void SetState<T>(string key, T value);
    void ClearState(string key);
    void ClearAllState();
}

// Implémentation
public class ApplicationStateService : IApplicationStateService
{
    private readonly Dictionary<string, object> _state = new Dictionary<string, object>();

    public T GetState<T>(string key, T defaultValue = default)
    {
        if (_state.TryGetValue(key, out var value) && value is T typedValue)
        {
            return typedValue;
        }

        return defaultValue;
    }

    public void SetState<T>(string key, T value)
    {
        _state[key] = value;
    }

    public void ClearState(string key)
    {
        if (_state.ContainsKey(key))
        {
            _state.Remove(key);
        }
    }

    public void ClearAllState()
    {
        _state.Clear();
    }
}
```


### Avantages de l'injection de dépendances dans MVVM

1. **Testabilité accrue** : Vous pouvez facilement remplacer les dépendances réelles par des mocks pour les tests
2. **Couplage faible** : Les composants ne dépendent pas directement les uns des autres
3. **Réutilisabilité** : Les services peuvent être utilisés dans différents ViewModels
4. **Séparation des préoccupations** : Chaque service a une responsabilité unique et bien définie
5. **Facilité de maintenance** : Il est plus facile de modifier ou remplacer un service sans affecter le reste de l'application

### Exemple complet avec MVVM et injection de dépendances (.NET 8)

Voici un exemple complet qui intègre le pattern MVVM avec l'injection de dépendances dans une application WPF simple :

**Program.cs (point d'entrée pour .NET 8)**

```textmate
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System.Windows;

namespace MvvmSample;

public class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        var application = new App();
        application.InitializeComponent();
        application.Run();
    }
}

public partial class App : Application
{
    private readonly IHost _host;

    public App()
    {
        _host = Host.CreateDefaultBuilder()
            .ConfigureServices((context, services) =>
            {
                ConfigureServices(services);
            })
            .Build();
    }

    private void ConfigureServices(IServiceCollection services)
    {
        // Services
        services.AddSingleton<IPeopleService, PeopleService>();
        services.AddSingleton<IDialogService, DialogService>();
        services.AddSingleton<INavigationService, NavigationService>();

        // ViewModels
        services.AddTransient<MainViewModel>();
        services.AddTransient<PersonViewModel>();
        services.AddTransient<SettingsViewModel>();

        // Views
        services.AddTransient<MainWindow>();
        services.AddTransient<PersonView>();
        services.AddTransient<SettingsView>();
    }

    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        var mainWindow = _host.Services.GetRequiredService<MainWindow>();
        mainWindow.Show();
    }

    protected override void OnExit(ExitEventArgs e)
    {
        _host.Dispose();
        base.OnExit(e);
    }
}
```


**Services**

```textmate
// IPeopleService.cs
public interface IPeopleService
{
    IEnumerable<Person> GetPeople();
    void SavePerson(Person person);
    void DeletePerson(Person person);
}

// PeopleService.cs
public class PeopleService : IPeopleService
{
    private readonly List<Person> _people = new List<Person>
    {
        new Person { Id = 1, FirstName = "Jean", LastName = "Dupont", BirthDate = new DateTime(1985, 5, 15) },
        new Person { Id = 2, FirstName = "Marie", LastName = "Martin", BirthDate = new DateTime(1990, 8, 22) }
    };

    public IEnumerable<Person> GetPeople()
    {
        return _people.ToList();
    }

    public void SavePerson(Person person)
    {
        var existingPerson = _people.FirstOrDefault(p => p.Id == person.Id);

        if (existingPerson != null)
        {
            existingPerson.FirstName = person.FirstName;
            existingPerson.LastName = person.LastName;
            existingPerson.BirthDate = person.BirthDate;
        }
        else
        {
            person.Id = _people.Max(p => p.Id) + 1;
            _people.Add(person);
        }
    }

    public void DeletePerson(Person person)
    {
        var existingPerson = _people.FirstOrDefault(p => p.Id == person.Id);

        if (existingPerson != null)
        {
            _people.Remove(existingPerson);
        }
    }
}
```


**Modèles**

```textmate
// Person.cs
public class Person : INotifyPropertyChanged
{
    private int _id;
    private string _firstName;
    private string _lastName;
    private DateTime _birthDate;

    public int Id
    {
        get => _id;
        set
        {
            _id = value;
            OnPropertyChanged();
        }
    }

    public string FirstName
    {
        get => _firstName;
        set
        {
            _firstName = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(FullName));
        }
    }

    public string LastName
    {
        get => _lastName;
        set
        {
            _lastName = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(FullName));
        }
    }

    public DateTime BirthDate
    {
        get => _birthDate;
        set
        {
            _birthDate = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(Age));
        }
    }

    public string FullName => $"{FirstName} {LastName}";

    public int Age => DateTime.Now.Year - BirthDate.Year -
        (DateTime.Now.DayOfYear < BirthDate.DayOfYear ? 1 : 0);

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


**ViewModels**

```textmate
// ViewModelBase.cs
public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
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

// MainViewModel.cs
public class MainViewModel : ViewModelBase
{
    private readonly INavigationService _navigationService;
    private ViewModelBase _currentViewModel;

    public ViewModelBase CurrentViewModel
    {
        get => _currentViewModel;
        set => SetProperty(ref _currentViewModel, value);
    }

    public ICommand NavigateToPeopleCommand { get; }
    public ICommand NavigateToSettingsCommand { get; }

    public MainViewModel(
        INavigationService navigationService,
        PersonViewModel personViewModel,
        SettingsViewModel settingsViewModel)
    {
        _navigationService = navigationService;

        NavigateToPeopleCommand = new RelayCommand(
            _ => CurrentViewModel = personViewModel
        );

        NavigateToSettingsCommand = new RelayCommand(
            _ => CurrentViewModel = settingsViewModel
        );

        // Vue par défaut
        CurrentViewModel = personViewModel;
    }
}

// PersonViewModel.cs
public class PersonViewModel : ViewModelBase
{
    private readonly IPeopleService _peopleService;
    private readonly IDialogService _dialogService;

    private ObservableCollection<Person> _people;
    private Person _selectedPerson;

    public ObservableCollection<Person> People
    {
        get => _people;
        set => SetProperty(ref _people, value);
    }

    public Person SelectedPerson
    {
        get => _selectedPerson;
        set => SetProperty(ref _selectedPerson, value);
    }

    public ICommand SaveCommand { get; }
    public ICommand DeleteCommand { get; }
    public ICommand AddCommand { get; }

    public PersonViewModel(IPeopleService peopleService, IDialogService dialogService)
    {
        _peopleService = peopleService;
        _dialogService = dialogService;

        // Initialisation des données
        LoadPeople();

        // Initialisation des commandes
        SaveCommand = new RelayCommand<Person>(
            Save,
            person => person != null &&
                     !string.IsNullOrEmpty(person.FirstName) &&
                     !string.IsNullOrEmpty(person.LastName)
        );

        DeleteCommand = new RelayCommand<Person>(
            Delete,
            person => person != null
        );

        AddCommand = new RelayCommand(Add);
    }

    private void LoadPeople()
    {
        var people = _peopleService.GetPeople();
        People = new ObservableCollection<Person>(people);

        if (People.Any())
        {
            SelectedPerson = People.First();
        }
    }

    private void Save(Person person)
    {
        try
        {
            _peopleService.SavePerson(person);
            _dialogService.ShowInformation("Personne enregistrée avec succès !");
        }
        catch (Exception ex)
        {
            _dialogService.ShowError($"Erreur lors de l'enregistrement : {ex.Message}");
        }
    }

    private void Delete(Person person)
    {
        if (_dialogService.ShowConfirmation("Êtes-vous sûr de vouloir supprimer cette personne ?"))
        {
            try
            {
                _peopleService.DeletePerson(person);
                People.Remove(person);

                if (People.Any())
                {
                    SelectedPerson = People.First();
                }
                else
                {
                    SelectedPerson = null;
                }
            }
            catch (Exception ex)
            {
                _dialogService.ShowError($"Erreur lors de la suppression : {ex.Message}");
            }
        }
    }

    private void Add()
    {
        var newPerson = new Person
        {
            FirstName = "Nouveau",
            LastName = "Contact",
            BirthDate = DateTime.Today.AddYears(-30)
        };

        People.Add(newPerson);
        SelectedPerson = newPerson;
    }
}
```


**Vues**

```xml
<!-- MainWindow.xaml -->
<Window x:Class="MvvmSample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MVVM Demo" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Barre de navigation -->
        <StackPanel Grid.Row="0" Orientation="Horizontal" Background="#f8f9fa" Padding="10">
            <Button Content="Personnes" Command="{Binding NavigateToPeopleCommand}"
                    Margin="0,0,10,0" Padding="10,5"/>
            <Button Content="Paramètres" Command="{Binding NavigateToSettingsCommand}"
                    Padding="10,5"/>
        </StackPanel>

        <!-- Contenu principal -->
        <ContentControl Grid.Row="1" Content="{Binding CurrentViewModel}">
            <ContentControl.Resources>
                <DataTemplate DataType="{x:Type local:PersonViewModel}">
                    <local:PersonView/>
                </DataTemplate>
                <DataTemplate DataType="{x:Type local:SettingsViewModel}">
                    <local:SettingsView/>
                </DataTemplate>
            </ContentControl.Resources>
        </ContentControl>
    </Grid>
</Window>

<!-- PersonView.xaml -->
<UserControl x:Class="MvvmSample.PersonView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Grid Margin="10">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Liste des personnes -->
        <Grid Grid.Column="0" Margin="0,0,5,0">
            <Grid.RowDefinitions>
                <RowDefinition Height="*"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>

            <ListBox Grid.Row="0"
                     ItemsSource="{Binding People}"
                     SelectedItem="{Binding SelectedPerson}"
                     DisplayMemberPath="FullName"
                     Margin="0,0,0,10"/>

            <Button Grid.Row="1" Content="Ajouter" Command="{Binding AddCommand}"/>
        </Grid>

        <!-- Détails de la personne sélectionnée -->
        <Grid Grid.Column="1" Margin="5,0,0,0">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>

            <TextBlock Grid.Row="0" Text="Détails de la personne" FontWeight="Bold" Margin="0,0,0,10"/>

            <StackPanel Grid.Row="1">
                <TextBlock Text="Prénom:"/>
                <TextBox Text="{Binding SelectedPerson.FirstName, UpdateSourceTrigger=PropertyChanged}"
                         Margin="0,0,0,10"/>

                <TextBlock Text="Nom:"/>
                <TextBox Text="{Binding SelectedPerson.LastName, UpdateSourceTrigger=PropertyChanged}"
                         Margin="0,0,0,10"/>

                <TextBlock Text="Date de naissance:"/>
                <DatePicker SelectedDate="{Binding SelectedPerson.BirthDate}"
                            Margin="0,0,0,10"/>

                <TextBlock Text="Âge:"/>
                <TextBlock Text="{Binding SelectedPerson.Age}" Margin="0,0,0,10"/>
            </StackPanel>

            <StackPanel Grid.Row="2" Orientation="Horizontal" HorizontalAlignment="Right">
                <Button Content="Enregistrer"
                        Command="{Binding SaveCommand}"
                        CommandParameter="{Binding SelectedPerson}"
                        Width="100" Margin="0,0,10,0"/>

                <Button Content="Supprimer"
                        Command="{Binding DeleteCommand}"
                        CommandParameter="{Binding SelectedPerson}"
                        Width="100"/>
            </StackPanel>
        </Grid>
    </Grid>
</UserControl>
```


Le pattern MVVM et l'injection de dépendances forment une combinaison puissante pour développer des applications WPF maintenables et testables. En séparant clairement les responsabilités et en rendant les composants faiblement couplés, vous pouvez créer des applications plus robustes et évolutives.

⏭️ 8.8. [Ressources et dictionnaires de ressources](/08-windows-presentation-foundation-wpf/8-08-ressources-et-dictionnaires-de-ressources.md)
