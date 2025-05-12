# 19.3. Patterns comportementaux en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les **patterns comportementaux** sont des modèles de conception qui concernent la communication entre objets et l'assignation des responsabilités entre objets. Ils permettent de définir des algorithmes et des relations entre objets de manière flexible.

## Qu'est-ce qu'un Pattern Comportemental ?

Les patterns comportementaux vous aident à :
- **Gérer les interactions entre objets**
- **Définir comment les objets communiquent**
- **Organiser les responsabilités** de manière flexible
- **Encapsuler des comportements complexes**

## 19.3.1. Observer - "Être notifié des changements"

### Qu'est-ce que l'Observer ?

L'Observer définit un **mécanisme de notification** qui permet à des objets (observateurs) d'être automatiquement notifiés quand l'état d'un autre objet (sujet) change.

### Quand l'utiliser ?

- Pour implémenter un système d'événements
- Quand des changements d'état doivent être propagés
- Pour découpler la source de notification de ses destinataires

### Exemple simple : Station météo

```csharp
// Interface Observer
public interface IWeatherObserver
{
    void Update(float temperature, float humidity, float pressure);
    string GetName();
}

// Interface Sujet (Observable)
public interface IWeatherData
{
    void RegisterObserver(IWeatherObserver observer);
    void RemoveObserver(IWeatherObserver observer);
    void NotifyObservers();
}

// Sujet concret
public class WeatherStation : IWeatherData
{
    private List<IWeatherObserver> observers = new List<IWeatherObserver>();
    private float temperature;
    private float humidity;
    private float pressure;

    public void RegisterObserver(IWeatherObserver observer)
    {
        observers.Add(observer);
        Console.WriteLine($"{observer.GetName()} s'est abonné aux mises à jour météo");
    }

    public void RemoveObserver(IWeatherObserver observer)
    {
        observers.Remove(observer);
        Console.WriteLine($"{observer.GetName()} s'est désabonné des mises à jour météo");
    }

    public void NotifyObservers()
    {
        foreach (var observer in observers)
        {
            observer.Update(temperature, humidity, pressure);
        }
    }

    public void MeasurementsChanged()
    {
        NotifyObservers();
    }

    public void SetMeasurements(float temperature, float humidity, float pressure)
    {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        MeasurementsChanged();
    }
}

// Observers concrets
public class CurrentConditionsDisplay : IWeatherObserver
{
    public string GetName() => "Affichage Conditions Actuelles";

    public void Update(float temperature, float humidity, float pressure)
    {
        Console.WriteLine($"[{GetName()}] Conditions actuelles:");
        Console.WriteLine($"  Température: {temperature}°C");
        Console.WriteLine($"  Humidité: {humidity}%");
        Console.WriteLine($"  Pression: {pressure} hPa");
    }
}

public class ForecastDisplay : IWeatherObserver
{
    private float lastPressure = 0;

    public string GetName() => "Prévisions";

    public void Update(float temperature, float humidity, float pressure)
    {
        string forecast;

        if (pressure > lastPressure)
        {
            forecast = "Amélioration du temps";
        }
        else if (pressure == lastPressure)
        {
            forecast = "Même temps";
        }
        else
        {
            forecast = "Attention au mauvais temps";
        }

        Console.WriteLine($"[{GetName()}] {forecast}");
        lastPressure = pressure;
    }
}

// Utilisation
var weatherStation = new WeatherStation();

var currentDisplay = new CurrentConditionsDisplay();
var forecastDisplay = new ForecastDisplay();

weatherStation.RegisterObserver(currentDisplay);
weatherStation.RegisterObserver(forecastDisplay);

// Simuler des mesures météo
weatherStation.SetMeasurements(25.5f, 65.0f, 1013.25f);
Console.WriteLine("-------------------");
weatherStation.SetMeasurements(27.0f, 70.0f, 1012.0f);
```

### Version moderne avec événements C#

```csharp
// Utiliser les événements C# intégrés
public class WeatherStationModern
{
    public event EventHandler<WeatherEventArgs> WeatherChanged;

    private float temperature;
    private float humidity;
    private float pressure;

    public void SetMeasurements(float temperature, float humidity, float pressure)
    {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;

        // Déclencher l'événement
        OnWeatherChanged(new WeatherEventArgs
        {
            Temperature = temperature,
            Humidity = humidity,
            Pressure = pressure
        });
    }

    protected virtual void OnWeatherChanged(WeatherEventArgs e)
    {
        WeatherChanged?.Invoke(this, e);
    }
}

public class WeatherEventArgs : EventArgs
{
    public float Temperature { get; set; }
    public float Humidity { get; set; }
    public float Pressure { get; set; }
}

// Utilisation avec événements
var weatherStation = new WeatherStationModern();

// S'abonner avec une lambda
weatherStation.WeatherChanged += (sender, e) =>
{
    Console.WriteLine($"Nouvelle mesure: {e.Temperature}°C, {e.Humidity}%, {e.Pressure} hPa");
};

// S'abonner avec une méthode
weatherStation.WeatherChanged += HandleWeatherChange;

void HandleWeatherChange(object sender, WeatherEventArgs e)
{
    Console.WriteLine($"Alerte: Température {(e.Temperature > 30 ? "très haute" : "normale")}");
}

weatherStation.SetMeasurements(32.0f, 80.0f, 1008.0f);
```

## 19.3.2. Strategy - "Plusieurs façons de faire la même chose"

### Qu'est-ce que la Strategy ?

La Strategy définit une **famille d'algorithmes**, les encapsule et les rend interchangeables. Le pattern permet à l'algorithme de varier indépendamment des clients qui l'utilisent.

### Quand l'utiliser ?

- Quand vous avez plusieurs façons de faire la même chose
- Pour éliminer des conditionnelles complexes (if/switch)
- Pour offrir différentes variantes d'un algorithme

### Exemple simple : Système de paiement

```csharp
// Interface Strategy
public interface IPaymentStrategy
{
    bool ProcessPayment(decimal amount);
    string GetPaymentMethodName();
}

// Stratégies concrètes
public class CreditCardPayment : IPaymentStrategy
{
    private string cardNumber;
    private string cvv;
    private string expiryDate;

    public CreditCardPayment(string cardNumber, string cvv, string expiryDate)
    {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }

    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Paiement par carte de crédit de {amount:C}");
        Console.WriteLine($"Carte: ****-****-****-{cardNumber.Substring(cardNumber.Length - 4)}");

        // Simuler la validation de la carte
        bool isValid = cvv.Length == 3 && !string.IsNullOrEmpty(expiryDate);

        if (isValid)
        {
            Console.WriteLine("Paiement réussi!");
            return true;
        }
        else
        {
            Console.WriteLine("Paiement échoué - Carte invalide");
            return false;
        }
    }

    public string GetPaymentMethodName() => "Carte de crédit";
}

public class PayPalPayment : IPaymentStrategy
{
    private string email;
    private string password;

    public PayPalPayment(string email, string password)
    {
        this.email = email;
        this.password = password;
    }

    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Paiement PayPal de {amount:C}");
        Console.WriteLine($"Email: {email}");

        // Simuler la connexion PayPal
        bool isAuthenticated = !string.IsNullOrEmpty(password);

        if (isAuthenticated)
        {
            Console.WriteLine("Paiement PayPal réussi!");
            return true;
        }
        else
        {
            Console.WriteLine("Paiement échoué - Authentification PayPal échouée");
            return false;
        }
    }

    public string GetPaymentMethodName() => "PayPal";
}

public class CashPayment : IPaymentStrategy
{
    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Paiement en espèces de {amount:C}");
        Console.WriteLine("Veuillez remettre l'argent au caissier");
        return true;
    }

    public string GetPaymentMethodName() => "Espèces";
}

// Contexte qui utilise la stratégie
public class ShoppingCart
{
    private List<(string item, decimal price)> items = new List<(string, decimal)>();
    private IPaymentStrategy paymentStrategy;

    public void AddItem(string item, decimal price)
    {
        items.Add((item, price));
        Console.WriteLine($"Ajouté: {item} - {price:C}");
    }

    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        paymentStrategy = strategy;
        Console.WriteLine($"Méthode de paiement sélectionnée: {strategy.GetPaymentMethodName()}");
    }

    public void Checkout()
    {
        if (paymentStrategy == null)
        {
            Console.WriteLine("Veuillez sélectionner une méthode de paiement");
            return;
        }

        decimal total = items.Sum(item => item.price);
        Console.WriteLine($"\nTotal à payer: {total:C}");
        Console.WriteLine("Articles:");
        foreach (var item in items)
        {
            Console.WriteLine($"  - {item.item}: {item.price:C}");
        }

        bool success = paymentStrategy.ProcessPayment(total);

        if (success)
        {
            items.Clear();
            Console.WriteLine("Commande terminée avec succès!");
        }
        else
        {
            Console.WriteLine("Commande annulée");
        }
    }
}

// Utilisation
var cart = new ShoppingCart();
cart.AddItem("Livre", 25.90m);
cart.AddItem("Café", 3.50m);

// Essayer différentes méthodes de paiement
cart.SetPaymentStrategy(new CreditCardPayment("1234567812345678", "123", "12/25"));
cart.Checkout();

// Ajouter plus d'articles et payer avec PayPal
cart.AddItem("Magazine", 5.90m);
cart.SetPaymentStrategy(new PayPalPayment("user@email.com", "password123"));
cart.Checkout();
```

## 19.3.3. Command - "Encapsuler une requête"

### Qu'est-ce que la Command ?

Le Command transforme une **requête en un objet** contenant toutes les informations nécessaires à son exécution. Cela permet de paramétrer, mettre en queue et annuler des requêtes.

### Quand l'utiliser ?

- Pour créer des actions annulables (undo/redo)
- Pour implémenter des systèmes de macro
- Pour découpler l'invocateur de l'exécuteur

### Exemple simple : Télécommande universelle

```csharp
// Interface Command
public interface ICommand
{
    void Execute();
    void Undo();
    string GetDescription();
}

// Récepteurs - les objets qui font le vrai travail
public class Light
{
    private string location;
    private bool isOn;

    public Light(string location)
    {
        this.location = location;
    }

    public void On()
    {
        isOn = true;
        Console.WriteLine($"{location} allumé");
    }

    public void Off()
    {
        isOn = false;
        Console.WriteLine($"{location} éteint");
    }

    public bool IsOn => isOn;
}

public class Stereo
{
    private bool isOn;
    private int volume;

    public void On()
    {
        isOn = true;
        Console.WriteLine("Chaîne stéréo allumée");
    }

    public void Off()
    {
        isOn = false;
        Console.WriteLine("Chaîne stéréo éteinte");
    }

    public void SetVolume(int volume)
    {
        this.volume = volume;
        Console.WriteLine($"Volume réglé à {volume}");
    }

    public int GetVolume() => volume;
}

// Commands concrètes
public class LightOnCommand : ICommand
{
    private Light light;

    public LightOnCommand(Light light)
    {
        this.light = light;
    }

    public void Execute()
    {
        light.On();
    }

    public void Undo()
    {
        light.Off();
    }

    public string GetDescription() => "Lumière ON";
}

public class LightOffCommand : ICommand
{
    private Light light;

    public LightOffCommand(Light light)
    {
        this.light = light;
    }

    public void Execute()
    {
        light.Off();
    }

    public void Undo()
    {
        light.On();
    }

    public string GetDescription() => "Lumière OFF";
}

public class StereoOnWithVolumeCommand : ICommand
{
    private Stereo stereo;
    private int previousVolume;

    public StereoOnWithVolumeCommand(Stereo stereo)
    {
        this.stereo = stereo;
    }

    public void Execute()
    {
        previousVolume = stereo.GetVolume();
        stereo.On();
        stereo.SetVolume(11);
    }

    public void Undo()
    {
        stereo.Off();
        if (previousVolume > 0)
        {
            stereo.SetVolume(previousVolume);
        }
    }

    public string GetDescription() => "Stéréo ON avec volume";
}

// Commande vide (Null Object Pattern)
public class NoCommand : ICommand
{
    public void Execute() { }
    public void Undo() { }
    public string GetDescription() => "Aucune commande";
}

// Invocateur (la télécommande)
public class RemoteControl
{
    private ICommand[] onCommands;
    private ICommand[] offCommands;
    private ICommand undoCommand;

    public RemoteControl()
    {
        onCommands = new ICommand[7];
        offCommands = new ICommand[7];

        ICommand noCommand = new NoCommand();
        for (int i = 0; i < 7; i++)
        {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand;
    }

    public void SetCommand(int slot, ICommand onCommand, ICommand offCommand)
    {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void OnButtonPressed(int slot)
    {
        onCommands[slot].Execute();
        undoCommand = onCommands[slot];
    }

    public void OffButtonPressed(int slot)
    {
        offCommands[slot].Execute();
        undoCommand = offCommands[slot];
    }

    public void UndoButtonPressed()
    {
        undoCommand.Undo();
    }

    public void ShowCommands()
    {
        Console.WriteLine("\n------ Télécommande ------");
        for (int i = 0; i < onCommands.Length; i++)
        {
            Console.WriteLine($"[slot {i}] {onCommands[i].GetDescription(),-15} | {offCommands[i].GetDescription()}");
        }
        Console.WriteLine($"[undo] {undoCommand.GetDescription()}");
    }
}

// Utilisation
var remote = new RemoteControl();

// Créer les appareils
var livingRoomLight = new Light("Salon");
var kitchenLight = new Light("Cuisine");
var stereo = new Stereo();

// Créer les commandes
var livingRoomLightOn = new LightOnCommand(livingRoomLight);
var livingRoomLightOff = new LightOffCommand(livingRoomLight);
var kitchenLightOn = new LightOnCommand(kitchenLight);
var kitchenLightOff = new LightOffCommand(kitchenLight);
var stereoOnWithVolume = new StereoOnWithVolumeCommand(stereo);

// Programmer la télécommande
remote.SetCommand(0, livingRoomLightOn, livingRoomLightOff);
remote.SetCommand(1, kitchenLightOn, kitchenLightOff);
remote.SetCommand(2, stereoOnWithVolume, new NoCommand());

remote.ShowCommands();

// Utiliser la télécommande
remote.OnButtonPressed(0);  // Allumer salon
remote.OnButtonPressed(1);  // Allumer cuisine
remote.OnButtonPressed(2);  // Allumer stéréo
remote.UndoButtonPressed(); // Annuler dernière commande
```

### Exemple avancé : Macro Command

```csharp
// Commande composite (Macro)
public class MacroCommand : ICommand
{
    private ICommand[] commands;

    public MacroCommand(ICommand[] commands)
    {
        this.commands = commands;
    }

    public void Execute()
    {
        Console.WriteLine("=== Exécution de macro ===");
        foreach (var command in commands)
        {
            command.Execute();
        }
        Console.WriteLine("=== Fin de macro ===");
    }

    public void Undo()
    {
        Console.WriteLine("=== Annulation de macro ===");
        // Annuler dans l'ordre inverse
        for (int i = commands.Length - 1; i >= 0; i--)
        {
            commands[i].Undo();
        }
        Console.WriteLine("=== Fin annulation macro ===");
    }

    public string GetDescription() => "Macro";
}

// Utilisation de macro
var partyMacro = new MacroCommand(new ICommand[]
{
    new LightOnCommand(livingRoomLight),
    new LightOnCommand(kitchenLight),
    new StereoOnWithVolumeCommand(stereo)
});

remote.SetCommand(6, partyMacro, new NoCommand());

Console.WriteLine("\nExécution de la macro 'Soirée':");
remote.OnButtonPressed(6);
Console.WriteLine("\nAnnulation de la macro:");
remote.UndoButtonPressed();
```

## 19.3.4. Template Method - "Squelette d'algorithme"

### Qu'est-ce que la Template Method ?

La Template Method définit le **squelette d'un algorithme** dans une superclasse, permettant aux sous-classes de redéfinir certaines étapes sans changer la structure globale.

### Quand l'utiliser ?

- Quand plusieurs classes implémentent des algorithmes similaires
- Pour éviter la duplication de code
- Pour garantir qu'un algorithme suit une structure spécifique

### Exemple simple : Préparation de boissons

```csharp
// Classe abstraite avec la template method
public abstract class CaffeineBeverage
{
    // Template method - définit l'algorithme
    public void PrepareRecipe()
    {
        BoilWater();
        Brew();
        PourInCup();
        if (CustomerWantsCondiments())
        {
            AddCondiments();
        }
        ServeCustomer();
    }

    // Étapes communes
    private void BoilWater()
    {
        Console.WriteLine("Faire bouillir l'eau");
    }

    private void PourInCup()
    {
        Console.WriteLine("Verser dans la tasse");
    }

    private void ServeCustomer()
    {
        Console.WriteLine("Servir au client");
        Console.WriteLine("-------------------");
    }

    // Méthodes abstraites - à implémenter par les sous-classes
    protected abstract void Brew();
    protected abstract void AddCondiments();

    // Hook - méthode avec implémentation par défaut
    protected virtual bool CustomerWantsCondiments()
    {
        return true;
    }
}

// Implémentations concrètes
public class Coffee : CaffeineBeverage
{
    protected override void Brew()
    {
        Console.WriteLine("Faire passer le café dans le filtre");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("Ajouter du sucre et du lait");
    }

    protected override bool CustomerWantsCondiments()
    {
        string answer = GetUserInput();
        return answer.ToLower().StartsWith("y");
    }

    private string GetUserInput()
    {
        Console.Write("Voulez-vous du sucre et du lait (y/n)? ");
        string answer = Console.ReadLine();
        return answer ?? "n";
    }
}

public class Tea : CaffeineBeverage
{
    protected override void Brew()
    {
        Console.WriteLine("Faire infuser le thé");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("Ajouter du citron");
    }
}

// Nouvelle boisson qui n'ajoute jamais de condiments
public class BlackTea : Tea
{
    protected override bool CustomerWantsCondiments()
    {
        return false; // Toujours servir sans condiments
    }
}

// Utilisation
Console.WriteLine("Préparation d'un café:");
var coffee = new Coffee();
coffee.PrepareRecipe();

Console.WriteLine("\nPréparation d'un thé:");
var tea = new Tea();
tea.PrepareRecipe();

Console.WriteLine("\nPréparation d'un thé noir:");
var blackTea = new BlackTea();
blackTea.PrepareRecipe();
```

### Exemple avancé : Traitement de documents

```csharp
// Classe abstraite pour le traitement de documents
public abstract class DocumentProcessor
{
    // Template method
    public void ProcessDocument(string filePath)
    {
        try
        {
            LogStart(filePath);

            if (ValidateFile(filePath))
            {
                var content = LoadDocument(filePath);
                var processedContent = ParseContent(content);
                var result = TransformContent(processedContent);

                if (ShouldSaveResult())
                {
                    SaveResult(result, filePath);
                }

                LogSuccess(filePath);
            }
            else
            {
                LogError("Fichier invalide", filePath);
            }
        }
        catch (Exception ex)
        {
            LogError(ex.Message, filePath);
            HandleError(ex);
        }
    }

    // Méthodes communes
    protected void LogStart(string filePath)
    {
        Console.WriteLine($"Début du traitement: {Path.GetFileName(filePath)}");
    }

    protected void LogSuccess(string filePath)
    {
        Console.WriteLine($"Traitement réussi: {Path.GetFileName(filePath)}");
    }

    protected void LogError(string error, string filePath)
    {
        Console.WriteLine($"Erreur lors du traitement de {Path.GetFileName(filePath)}: {error}");
    }

    // Méthodes abstraites
    protected abstract string LoadDocument(string filePath);
    protected abstract object ParseContent(string content);
    protected abstract object TransformContent(object parsedContent);
    protected abstract void SaveResult(object result, string filePath);

    // Hooks avec implémentations par défaut
    protected virtual bool ValidateFile(string filePath)
    {
        return File.Exists(filePath);
    }

    protected virtual bool ShouldSaveResult()
    {
        return true;
    }

    protected virtual void HandleError(Exception ex)
    {
        // Implémentation par défaut - ne fait rien
    }
}

// Processeur pour fichiers CSV
public class CSVProcessor : DocumentProcessor
{
    protected override string LoadDocument(string filePath)
    {
        Console.WriteLine("Chargement du fichier CSV...");
        return File.ReadAllText(filePath);
    }

    protected override object ParseContent(string content)
    {
        Console.WriteLine("Analyse du contenu CSV...");
        var lines = content.Split('\n');
        var data = new List<Dictionary<string, string>>();

        if (lines.Length > 0)
        {
            var headers = lines[0].Split(',');
            for (int i = 1; i < lines.Length; i++)
            {
                var values = lines[i].Split(',');
                var row = new Dictionary<string, string>();
                for (int j = 0; j < headers.Length && j < values.Length; j++)
                {
                    row[headers[j]] = values[j];
                }
                data.Add(row);
            }
        }

        return data;
    }

    protected override object TransformContent(object parsedContent)
    {
        Console.WriteLine("Transformation des données...");
        var data = (List<Dictionary<string, string>>)parsedContent;

        // Exemple de transformation : convertir en JSON
        var json = new StringBuilder();
        json.AppendLine("[");
        for (int i = 0; i < data.Count; i++)
        {
            json.Append("  {");
            var pairs = data[i].Select(kvp => $"\"{kvp.Key}\": \"{kvp.Value}\"");
            json.Append(string.Join(", ", pairs));
            json.Append("}");
            if (i < data.Count - 1) json.Append(",");
            json.AppendLine();
        }
        json.AppendLine("]");

        return json.ToString();
    }

    protected override void SaveResult(object result, string filePath)
    {
        string outputPath = Path.ChangeExtension(filePath, ".json");
        Console.WriteLine($"Sauvegarde du résultat dans {outputPath}...");
        File.WriteAllText(outputPath, result.ToString());
    }

    protected override bool ValidateFile(string filePath)
    {
        return base.ValidateFile(filePath) && filePath.EndsWith(".csv", StringComparison.OrdinalIgnoreCase);
    }
}

// Processeur XML avec gestion d'erreur personnalisée
public class XMLProcessor : DocumentProcessor
{
    private bool silentMode;

    public XMLProcessor(bool silentMode = false)
    {
        this.silentMode = silentMode;
    }

    protected override string LoadDocument(string filePath)
    {
        Console.WriteLine("Chargement du fichier XML...");
        return File.ReadAllText(filePath);
    }

    protected override object ParseContent(string content)
    {
        Console.WriteLine("Analyse du contenu XML...");
        // Simuler l'analyse XML
        return content;
    }

    protected override object TransformContent(object parsedContent)
    {
        Console.WriteLine("Transformation vers HTML...");
        // Simuler la transformation
        return $"<html><body>{parsedContent}</body></html>";
    }

    protected override void SaveResult(object result, string filePath)
    {
        string outputPath = Path.ChangeExtension(filePath, ".html");
        Console.WriteLine($"Sauvegarde du résultat dans {outputPath}...");
        File.WriteAllText(outputPath, result.ToString());
    }

    protected override bool ShouldSaveResult()
    {
        // Ne pas sauver en mode silencieux
        return !silentMode;
    }

    protected override void HandleError(Exception ex)
    {
        if (!silentMode)
        {
            Console.WriteLine("Envoi d'un email d'alerte...");
            // Simuler l'envoi d'email
        }
    }
}

// Utilisation
var csvProcessor = new CSVProcessor();
// csvProcessor.ProcessDocument("data.csv");

var xmlProcessor = new XMLProcessor(silentMode: false);
// xmlProcessor.ProcessDocument("document.xml");
```

## 19.3.5. Iterator - "Parcourir une collection"

### Qu'est-ce que l'Iterator ?

L'Iterator fournit un moyen d'**accéder séquentiellement aux éléments** d'une collection sans exposer sa représentation interne.

### Quand l'utiliser ?

- Pour permettre le parcours d'une collection de différentes manières
- Pour masquer la complexité interne d'une structure de données
- Pour fournir une interface uniforme de parcours

### Exemple simple : Collection personnalisée

```csharp
// Interface Iterator
public interface IIterator<T>
{
    bool HasNext();
    T Next();
    void Reset();
}

// Collection personnalisée
public class BookCollection
{
    private List<Book> books = new List<Book>();

    public void AddBook(Book book)
    {
        books.Add(book);
    }

    public IIterator<Book> CreateIterator()
    {
        return new BookIterator(books);
    }

    public IIterator<Book> CreateGenreIterator(string genre)
    {
        return new BookGenreIterator(books, genre);
    }

    public IIterator<Book> CreateAuthorIterator(string author)
    {
        return new BookAuthorIterator(books, author);
    }
}

public class Book
{
    public string Title { get; set; }
    public string Author { get; set; }
    public string Genre { get; set; }
    public int Year { get; set; }

    public override string ToString()
    {
        return $"{Title} by {Author} ({Genre}, {Year})";
    }
}

// Iterator de base
public class BookIterator : IIterator<Book>
{
    private readonly List<Book> books;
    private int position;

    public BookIterator(List<Book> books)
    {
        this.books = books;
        this.position = 0;
    }

    public bool HasNext()
    {
        return position < books.Count;
    }

    public Book Next()
    {
        if (!HasNext())
        {
            throw new InvalidOperationException("No more elements");
        }

        Book book = books[position];
        position++;
        return book;
    }

    public void Reset()
    {
        position = 0;
    }
}

// Iterator filtrant par genre
public class BookGenreIterator : IIterator<Book>
{
    private readonly List<Book> books;
    private readonly string targetGenre;
    private int position;

    public BookGenreIterator(List<Book> books, string genre)
    {
        this.books = books;
        this.targetGenre = genre;
        this.position = 0;
    }

    public bool HasNext()
    {
        while (position < books.Count)
        {
            if (books[position].Genre.Equals(targetGenre, StringComparison.OrdinalIgnoreCase))
            {
                return true;
            }
            position++;
        }
        return false;
    }

    public Book Next()
    {
        if (!HasNext())
        {
            throw new InvalidOperationException("No more elements");
        }

        Book book = books[position];
        position++;
        return book;
    }

    public void Reset()
    {
        position = 0;
    }
}

// Iterator filtrant par auteur
public class BookAuthorIterator : IIterator<Book>
{
    private readonly List<Book> books;
    private readonly string targetAuthor;
    private int position;

    public BookAuthorIterator(List<Book> books, string author)
    {
        this.books = books;
        this.targetAuthor = author;
        this.position = 0;
    }

    public bool HasNext()
    {
        while (position < books.Count)
        {
            if (books[position].Author.Equals(targetAuthor, StringComparison.OrdinalIgnoreCase))
            {
                return true;
            }
            position++;
        }
        return false;
    }

    public Book Next()
    {
        if (!HasNext())
        {
            throw new InvalidOperationException("No more elements");
        }

        Book book = books[position];
        position++;
        return book;
    }

    public void Reset()
    {
        position = 0;
    }
}

// Utilisation des iterators
var library = new BookCollection();
library.AddBook(new Book { Title = "Le Seigneur des Anneaux", Author = "J.R.R. Tolkien", Genre = "Fantasy", Year = 1954 });
library.AddBook(new Book { Title = "1984", Author = "George Orwell", Genre = "Science Fiction", Year = 1949 });
library.AddBook(new Book { Title = "Le Hobbit", Author = "J.R.R. Tolkien", Genre = "Fantasy", Year = 1937 });
library.AddBook(new Book { Title = "Brave New World", Author = "Aldous Huxley", Genre = "Science Fiction", Year = 1932 });
library.AddBook(new Book { Title = "Animal Farm", Author = "George Orwell", Genre = "Satire", Year = 1945 });

// Parcourir tous les livres
Console.WriteLine("Tous les livres:");
var allBooksIterator = library.CreateIterator();
while (allBooksIterator.HasNext())
{
    Console.WriteLine(allBooksIterator.Next());
}

// Parcourir seulement les livres Fantasy
Console.WriteLine("\nLivres Fantasy:");
var fantasyIterator = library.CreateGenreIterator("Fantasy");
while (fantasyIterator.HasNext())
{
    Console.WriteLine(fantasyIterator.Next());
}

// Parcourir les livres de George Orwell
Console.WriteLine("\nLivres de George Orwell:");
var orwellIterator = library.CreateAuthorIterator("George Orwell");
while (orwellIterator.HasNext())
{
    Console.WriteLine(orwellIterator.Next());
}
```

### Version moderne avec C# IEnumerable

```csharp
// Utiliser IEnumerable<T> pour une meilleure intégration avec C#
public class ModernBookCollection : IEnumerable<Book>
{
    private List<Book> books = new List<Book>();

    public void AddBook(Book book)
    {
        books.Add(book);
    }

    // Implémentation de IEnumerable
    public IEnumerator<Book> GetEnumerator()
    {
        return books.GetEnumerator();
    }

    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    // Méthodes d'extension pour différents types de parcours
    public IEnumerable<Book> GetBooksByGenre(string genre)
    {
        foreach (var book in books)
        {
            if (book.Genre.Equals(genre, StringComparison.OrdinalIgnoreCase))
            {
                yield return book;
            }
        }
    }

    public IEnumerable<Book> GetBooksByAuthor(string author)
    {
        foreach (var book in books)
        {
            if (book.Author.Equals(author, StringComparison.OrdinalIgnoreCase))
            {
                yield return book;
            }
        }
    }

    public IEnumerable<Book> GetBooksAfterYear(int year)
    {
        foreach (var book in books)
        {
            if (book.Year > year)
            {
                yield return book;
            }
        }
    }
}

// Utilisation avec LINQ
var modernLibrary = new ModernBookCollection();
// ... ajouter des livres ...

// Utiliser foreach directement
foreach (var book in modernLibrary)
{
    Console.WriteLine(book);
}

// Utiliser LINQ pour des requêtes complexes
var scienceFictionSince1950 = modernLibrary
    .GetBooksAfterYear(1950)
    .Where(b => b.Genre == "Science Fiction")
    .OrderBy(b => b.Year);

foreach (var book in scienceFictionSince1950)
{
    Console.WriteLine(book);
}
```

## 19.3.6. State - "Changer de comportement selon l'état"

### Qu'est-ce que le State ?

Le State permet à un objet de **modifier son comportement** lorsque son état interne change. L'objet donnera l'impression que sa classe a changé.

### Quand l'utiliser ?

- Quand un objet a des comportements qui dépendent de son état
- Pour remplacer des conditionnelles complexes par des objets
- Quand des transitions d'état doivent être explicites

### Exemple simple : Distributeur automatique

```csharp
// Interface pour les états
public interface IGumballMachineState
{
    void InsertQuarter();
    void EjectQuarter();
    void TurnCrank();
    void Dispense();
}

// Contexte - la machine à chewing-gums
public class GumballMachine
{
    private IGumballMachineState soldOutState;
    private IGumballMachineState noQuarterState;
    private IGumballMachineState hasQuarterState;
    private IGumballMachineState soldState;

    private IGumballMachineState currentState;
    private int count;

    public GumballMachine(int numberOfGumballs)
    {
        soldOutState = new SoldOutState(this);
        noQuarterState = new NoQuarterState(this);
        hasQuarterState = new HasQuarterState(this);
        soldState = new SoldState(this);

        count = numberOfGumballs;
        currentState = count > 0 ? noQuarterState : soldOutState;
    }

    public void InsertQuarter()
    {
        currentState.InsertQuarter();
    }

    public void EjectQuarter()
    {
        currentState.EjectQuarter();
    }

    public void TurnCrank()
    {
        currentState.TurnCrank();
        currentState.Dispense();
    }

    public void SetState(IGumballMachineState state)
    {
        currentState = state;
    }

    public void ReleaseBall()
    {
        Console.WriteLine("Une boule de chewing-gum sort...");
        if (count > 0)
        {
            count--;
        }
    }

    public int GetCount() => count;

    public IGumballMachineState GetSoldOutState() => soldOutState;
    public IGumballMachineState GetNoQuarterState() => noQuarterState;
    public IGumballMachineState GetHasQuarterState() => hasQuarterState;
    public IGumballMachineState GetSoldState() => soldState;

    public override string ToString()
    {
        return $"\nInventaire: {count} chewing-gums\n" +
               $"État: {currentState.GetType().Name}\n";
    }
}

// États concrets
public class NoQuarterState : IGumballMachineState
{
    private GumballMachine gumballMachine;

    public NoQuarterState(GumballMachine machine)
    {
        this.gumballMachine = machine;
    }

    public void InsertQuarter()
    {
        Console.WriteLine("Pièce insérée");
        gumballMachine.SetState(gumballMachine.GetHasQuarterState());
    }

    public void EjectQuarter()
    {
        Console.WriteLine("Vous n'avez pas inséré de pièce");
    }

    public void TurnCrank()
    {
        Console.WriteLine("Vous avez tourné, mais aucune pièce n'est insérée");
    }

    public void Dispense()
    {
        Console.WriteLine("Vous devez d'abord payer");
    }
}

public class HasQuarterState : IGumballMachineState
{
    private GumballMachine gumballMachine;

    public HasQuarterState(GumballMachine machine)
    {
        this.gumballMachine = machine;
    }

    public void InsertQuarter()
    {
        Console.WriteLine("Vous ne pouvez pas insérer une autre pièce");
    }

    public void EjectQuarter()
    {
        Console.WriteLine("Pièce retournée");
        gumballMachine.SetState(gumballMachine.GetNoQuarterState());
    }

    public void TurnCrank()
    {
        Console.WriteLine("Vous avez tourné...");
        gumballMachine.SetState(gumballMachine.GetSoldState());
    }

    public void Dispense()
    {
        Console.WriteLine("Aucun chewing-gum distribué");
    }
}

public class SoldState : IGumballMachineState
{
    private GumballMachine gumballMachine;

    public SoldState(GumballMachine machine)
    {
        this.gumballMachine = machine;
    }

    public void InsertQuarter()
    {
        Console.WriteLine("Veuillez patienter, nous vous distribuons un chewing-gum");
    }

    public void EjectQuarter()
    {
        Console.WriteLine("Désolé, vous avez déjà tourné la manivelle");
    }

    public void TurnCrank()
    {
        Console.WriteLine("Tourner deux fois ne vous donne pas un autre chewing-gum!");
    }

    public void Dispense()
    {
        gumballMachine.ReleaseBall();
        if (gumballMachine.GetCount() > 0)
        {
            gumballMachine.SetState(gumballMachine.GetNoQuarterState());
        }
        else
        {
            Console.WriteLine("Oops, rupture de stock!");
            gumballMachine.SetState(gumballMachine.GetSoldOutState());
        }
    }
}

public class SoldOutState : IGumballMachineState
{
    private GumballMachine gumballMachine;

    public SoldOutState(GumballMachine machine)
    {
        this.gumballMachine = machine;
    }

    public void InsertQuarter()
    {
        Console.WriteLine("Vous ne pouvez pas insérer de pièce, la machine est en rupture de stock");
    }

    public void EjectQuarter()
    {
        Console.WriteLine("Vous ne pouvez pas éjecter, vous n'avez pas inséré de pièce");
    }

    public void TurnCrank()
    {
        Console.WriteLine("Vous avez tourné, mais il n'y a pas de chewing-gums");
    }

    public void Dispense()
    {
        Console.WriteLine("Aucun chewing-gum à distribuer");
    }
}

// Utilisation
var machine = new GumballMachine(3);
Console.WriteLine(machine);

machine.InsertQuarter();
machine.TurnCrank();
Console.WriteLine(machine);

machine.InsertQuarter();
machine.EjectQuarter();
machine.TurnCrank();
Console.WriteLine(machine);

machine.InsertQuarter();
machine.TurnCrank();
Console.WriteLine(machine);

machine.InsertQuarter();
machine.TurnCrank();
Console.WriteLine(machine);
```

### Exemple avancé : Commande de restaurant

```csharp
// État abstrait
public abstract class OrderState
{
    protected Order order;

    public OrderState(Order order)
    {
        this.order = order;
    }

    public abstract void ConfirmOrder();
    public abstract void PrepareOrder();
    public abstract void ShipOrder();
    public abstract void DeliverOrder();
    public abstract void CancelOrder();
    public abstract void PayOrder();

    protected void LogAction(string action)
    {
        Console.WriteLine($"[{GetType().Name}] {action}");
    }
}

// Contexte - la commande
public class Order
{
    private OrderState newOrderState;
    private OrderState confirmedState;
    private OrderState preparingState;
    private OrderState shippedState;
    private OrderState deliveredState;
    private OrderState cancelledState;
    private OrderState paidState;

    private OrderState currentState;
    private string orderId;
    private List<string> items;
    private decimal total;

    public Order(string orderId, List<string> items, decimal total)
    {
        this.orderId = orderId;
        this.items = items;
        this.total = total;

        newOrderState = new NewOrderState(this);
        confirmedState = new ConfirmedState(this);
        preparingState = new PreparingState(this);
        shippedState = new ShippedState(this);
        deliveredState = new DeliveredState(this);
        cancelledState = new CancelledState(this);
        paidState = new PaidState(this);

        currentState = newOrderState;
    }

    public void SetState(OrderState state)
    {
        currentState = state;
        Console.WriteLine($"État changé -> {state.GetType().Name}");
    }

    public void ConfirmOrder() => currentState.ConfirmOrder();
    public void PrepareOrder() => currentState.PrepareOrder();
    public void ShipOrder() => currentState.ShipOrder();
    public void DeliverOrder() => currentState.DeliverOrder();
    public void CancelOrder() => currentState.CancelOrder();
    public void PayOrder() => currentState.PayOrder();

    public OrderState GetNewOrderState() => newOrderState;
    public OrderState GetConfirmedState() => confirmedState;
    public OrderState GetPreparingState() => preparingState;
    public OrderState GetShippedState() => shippedState;
    public OrderState GetDeliveredState() => deliveredState;
    public OrderState GetCancelledState() => cancelledState;
    public OrderState GetPaidState() => paidState;

    public override string ToString()
    {
        return $"Commande {orderId} - Total: {total:C} - État: {currentState.GetType().Name}";
    }
}

// États concrets
public class NewOrderState : OrderState
{
    public NewOrderState(Order order) : base(order) { }

    public override void ConfirmOrder()
    {
        LogAction("Confirmation de la commande");
        order.SetState(order.GetConfirmedState());
    }

    public override void CancelOrder()
    {
        LogAction("Annulation de la commande");
        order.SetState(order.GetCancelledState());
    }

    public override void PrepareOrder() => LogAction("La commande doit d'abord être confirmée");
    public override void ShipOrder() => LogAction("La commande doit d'abord être confirmée");
    public override void DeliverOrder() => LogAction("La commande doit d'abord être confirmée");
    public override void PayOrder() => LogAction("La commande doit d'abord être confirmée");
}

public class ConfirmedState : OrderState
{
    public ConfirmedState(Order order) : base(order) { }

    public override void PrepareOrder()
    {
        LogAction("Début de la préparation");
        order.SetState(order.GetPreparingState());
    }

    public override void CancelOrder()
    {
        LogAction("Annulation de la commande confirmée");
        order.SetState(order.GetCancelledState());
    }

    public override void PayOrder()
    {
        LogAction("Paiement effectué");
        order.SetState(order.GetPaidState());
    }

    public override void ConfirmOrder() => LogAction("La commande est déjà confirmée");
    public override void ShipOrder() => LogAction("La commande doit d'abord être préparée");
    public override void DeliverOrder() => LogAction("La commande doit d'abord être préparée");
}

public class PreparingState : OrderState
{
    public PreparingState(Order order) : base(order) { }

    public override void ShipOrder()
    {
        LogAction("Commande prête pour l'expédition");
        order.SetState(order.GetShippedState());
    }

    public override void CancelOrder()
    {
        LogAction("Annulation pendant la préparation");
        order.SetState(order.GetCancelledState());
    }

    public override void ConfirmOrder() => LogAction("La commande est déjà confirmée");
    public override void PrepareOrder() => LogAction("La commande est déjà en préparation");
    public override void DeliverOrder() => LogAction("La commande doit d'abord être expédiée");
    public override void PayOrder() => LogAction("Paiement effectué pendant la préparation");
}

public class ShippedState : OrderState
{
    public ShippedState(Order order) : base(order) { }

    public override void DeliverOrder()
    {
        LogAction("Commande livrée!");
        order.SetState(order.GetDeliveredState());
    }

    public override void ConfirmOrder() => LogAction("La commande est déjà confirmée");
    public override void PrepareOrder() => LogAction("La commande est déjà préparée");
    public override void ShipOrder() => LogAction("La commande est déjà expédiée");
    public override void CancelOrder() => LogAction("Impossible d'annuler, la commande est expédiée");
    public override void PayOrder() => LogAction("Paiement effectué pendant l'expédition");
}

public class DeliveredState : OrderState
{
    public DeliveredState(Order order) : base(order) { }

    public override void PayOrder()
    {
        LogAction("Paiement à la livraison effectué");
        order.SetState(order.GetPaidState());
    }

    public override void ConfirmOrder() => LogAction("La commande est déjà confirmée");
    public override void PrepareOrder() => LogAction("La commande est déjà préparée");
    public override void ShipOrder() => LogAction("La commande est déjà expédiée");
    public override void DeliverOrder() => LogAction("La commande est déjà livrée");
    public override void CancelOrder() => LogAction("Impossible d'annuler, la commande est livrée");
}

public class CancelledState : OrderState
{
    public CancelledState(Order order) : base(order) { }

    public override void ConfirmOrder() => LogAction("Impossible de confirmer une commande annulée");
    public override void PrepareOrder() => LogAction("Impossible de préparer une commande annulée");
    public override void ShipOrder() => LogAction("Impossible d'expédier une commande annulée");
    public override void DeliverOrder() => LogAction("Impossible de livrer une commande annulée");
    public override void CancelOrder() => LogAction("La commande est déjà annulée");
    public override void PayOrder() => LogAction("Impossible de payer une commande annulée");
}

public class PaidState : OrderState
{
    public PaidState(Order order) : base(order) { }

    public override void ConfirmOrder() => LogAction("La commande est déjà confirmée et payée");
    public override void PrepareOrder() => LogAction("La commande peut maintenant être préparée");
    public override void ShipOrder() => LogAction("La commande peut être expédiée");
    public override void DeliverOrder() => LogAction("La commande peut être livrée");
    public override void CancelOrder() => LogAction("Demande de remboursement requise");
    public override void PayOrder() => LogAction("La commande est déjà payée");
}

// Utilisation
var order = new Order("ORD-001", new List<string> { "Pizza", "Boisson" }, 15.99m);
Console.WriteLine(order);

order.ConfirmOrder();
order.PayOrder();
order.PrepareOrder();
order.ShipOrder();
order.DeliverOrder();

Console.WriteLine("\nNouvelle commande:");
var order2 = new Order("ORD-002", new List<string> { "Burger" }, 8.99m);
order2.ConfirmOrder();
order2.CancelOrder();
```

## 19.3.7. Chain of Responsibility - "Passer la balle"

### Qu'est-ce que Chain of Responsibility ?

Chain of Responsibility permet de **passer des requêtes le long d'une chaîne** de handlers jusqu'à ce que l'un d'eux puisse traiter la requête.

### Quand l'utiliser ?

- Pour découpler l'émetteur d'une requête de son traitement
- Quand plusieurs objets peuvent traiter une requête
- Pour configurer dynamiquement l'ordre des traitements

### Exemple simple : Service client

```csharp
// Request
public class SupportRequest
{
    public string CustomerName { get; set; }
    public RequestType Type { get; set; }
    public string Description { get; set; }
    public Priority Priority { get; set; }

    public SupportRequest(string customerName, RequestType type, string description, Priority priority)
    {
        CustomerName = customerName;
        Type = type;
        Description = description;
        Priority = priority;
    }
}

public enum RequestType
{
    Technical,
    Billing,
    General,
    Complaint
}

public enum Priority
{
    Low,
    Medium,
    High,
    Critical
}

// Handler abstrait
public abstract class SupportHandler
{
    protected SupportHandler nextHandler;

    public void SetNext(SupportHandler handler)
    {
        nextHandler = handler;
    }

    public abstract void HandleRequest(SupportRequest request);

    protected void PassToNext(SupportRequest request)
    {
        if (nextHandler != null)
        {
            Console.WriteLine($"Transfert vers le niveau suivant...");
            nextHandler.HandleRequest(request);
        }
        else
        {
            Console.WriteLine("Aucun handler disponible pour cette requête.");
        }
    }
}

// Handlers concrets
public class TechnicalSupportHandler : SupportHandler
{
    public override void HandleRequest(SupportRequest request)
    {
        Console.WriteLine($"\n[Support Technique] Traitement de la requête de {request.CustomerName}");

        if (request.Type == RequestType.Technical && request.Priority <= Priority.Medium)
        {
            Console.WriteLine($"Résolution du problème technique: {request.Description}");
            Console.WriteLine("Requête résolue par le support technique!");
        }
        else
        {
            Console.WriteLine("Le support technique ne peut pas traiter cette requête.");
            PassToNext(request);
        }
    }
}

public class BillingSupportHandler : SupportHandler
{
    public override void HandleRequest(SupportRequest request)
    {
        Console.WriteLine($"\n[Support Facturation] Traitement de la requête de {request.CustomerName}");

        if (request.Type == RequestType.Billing)
        {
            Console.WriteLine($"Traitement de la question de facturation: {request.Description}");
            Console.WriteLine("Requête résolue par le service facturation!");
        }
        else
        {
            Console.WriteLine("Le service facturation ne peut pas traiter cette requête.");
            PassToNext(request);
        }
    }
}

public class ManagerHandler : SupportHandler
{
    public override void HandleRequest(SupportRequest request)
    {
        Console.WriteLine($"\n[Manager] Traitement de la requête de {request.CustomerName}");

        if (request.Priority >= Priority.High || request.Type == RequestType.Complaint)
        {
            Console.WriteLine($"Prise en charge par le management: {request.Description}");
            Console.WriteLine("Requête traitée par un manager!");
        }
        else
        {
            Console.WriteLine("Le manager ne peut pas traiter cette requête.");
            PassToNext(request);
        }
    }
}

public class DirectorHandler : SupportHandler
{
    public override void HandleRequest(SupportRequest request)
    {
        Console.WriteLine($"\n[Directeur] Traitement de la requête de {request.CustomerName}");

        if (request.Priority == Priority.Critical)
        {
            Console.WriteLine($"Intervention directe du directeur: {request.Description}");
            Console.WriteLine("Requête traitée au plus haut niveau!");
        }
        else
        {
            Console.WriteLine("Le directeur ne peut pas traiter cette requête.");
            PassToNext(request);
        }
    }
}

// Configuration et utilisation
var technicalSupport = new TechnicalSupportHandler();
var billingSupport = new BillingSupportHandler();
var manager = new ManagerHandler();
var director = new DirectorHandler();

// Chaîner les handlers
technicalSupport.SetNext(billingSupport);
billingSupport.SetNext(manager);
manager.SetNext(director);

// Tester différentes requêtes
var request1 = new SupportRequest("Jean Dupont", RequestType.Technical, "Mon imprimante ne fonctionne pas", Priority.Low);
technicalSupport.HandleRequest(request1);

var request2 = new SupportRequest("Marie Martin", RequestType.Billing, "Double facturation sur ma facture", Priority.Medium);
technicalSupport.HandleRequest(request2);

var request3 = new SupportRequest("Paul Durand", RequestType.Complaint, "Service très décevant", Priority.High);
technicalSupport.HandleRequest(request3);

var request4 = new SupportRequest("Sophie Leroy", RequestType.General, "Système en panne total", Priority.Critical);
technicalSupport.HandleRequest(request4);
```

### Exemple avancé : Pipeline de validation

```csharp
// Base handler avec support pour le pipeline
public abstract class ValidationHandler<T>
{
    protected ValidationHandler<T> nextHandler;

    public ValidationHandler<T> SetNext(ValidationHandler<T> handler)
    {
        nextHandler = handler;
        return handler; // Permet le chaînage fluide
    }

    public ValidationResult Validate(T item)
    {
        var result = DoValidate(item);

        if (result.IsValid && nextHandler != null)
        {
            var nextResult = nextHandler.Validate(item);
            if (!nextResult.IsValid)
            {
                result.IsValid = false;
                result.Errors.AddRange(nextResult.Errors);
            }
        }

        return result;
    }

    protected abstract ValidationResult DoValidate(T item);
}

public class ValidationResult
{
    public bool IsValid { get; set; } = true;
    public List<string> Errors { get; set; } = new List<string>();

    public void AddError(string error)
    {
        IsValid = false;
        Errors.Add(error);
    }
}

// Modèle à valider
public class UserRegistration
{
    public string Username { get; set; }
    public string Email { get; set; }
    public string Password { get; set; }
    public int Age { get; set; }
    public string Country { get; set; }
}

// Validators concrets
public class UsernameValidator : ValidationHandler<UserRegistration>
{
    protected override ValidationResult DoValidate(UserRegistration user)
    {
        var result = new ValidationResult();

        if (string.IsNullOrEmpty(user.Username))
        {
            result.AddError("Le nom d'utilisateur est requis");
        }
        else if (user.Username.Length < 3)
        {
            result.AddError("Le nom d'utilisateur doit contenir au moins 3 caractères");
        }
        else if (!System.Text.RegularExpressions.Regex.IsMatch(user.Username, @"^[a-zA-Z0-9_]+$"))
        {
            result.AddError("Le nom d'utilisateur ne peut contenir que des lettres, des chiffres et des underscores");
        }

        Console.WriteLine($"[UsernameValidator] {(result.IsValid ? "✓" : "✗")} {user.Username}");
        return result;
    }
}

public class EmailValidator : ValidationHandler<UserRegistration>
{
    protected override ValidationResult DoValidate(UserRegistration user)
    {
        var result = new ValidationResult();

        if (string.IsNullOrEmpty(user.Email))
        {
            result.AddError("L'email est requis");
        }
        else if (!System.Text.RegularExpressions.Regex.IsMatch(user.Email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$"))
        {
            result.AddError("Format d'email invalide");
        }

        Console.WriteLine($"[EmailValidator] {(result.IsValid ? "✓" : "✗")} {user.Email}");
        return result;
    }
}

public class PasswordValidator : ValidationHandler<UserRegistration>
{
    protected override ValidationResult DoValidate(UserRegistration user)
    {
        var result = new ValidationResult();

        if (string.IsNullOrEmpty(user.Password))
        {
            result.AddError("Le mot de passe est requis");
        }
        else if (user.Password.Length < 8)
        {
            result.AddError("Le mot de passe doit contenir au moins 8 caractères");
        }
        else if (!System.Text.RegularExpressions.Regex.IsMatch(user.Password, @"[A-Z]"))
        {
            result.AddError("Le mot de passe doit contenir au moins une majuscule");
        }
        else if (!System.Text.RegularExpressions.Regex.IsMatch(user.Password, @"[a-z]"))
        {
            result.AddError("Le mot de passe doit contenir au moins une minuscule");
        }
        else if (!System.Text.RegularExpressions.Regex.IsMatch(user.Password, @"[0-9]"))
        {
            result.AddError("Le mot de passe doit contenir au moins un chiffre");
        }

        Console.WriteLine($"[PasswordValidator] {(result.IsValid ? "✓" : "✗")} Password strength");
        return result;
    }
}

public class AgeValidator : ValidationHandler<UserRegistration>
{
    protected override ValidationResult DoValidate(UserRegistration user)
    {
        var result = new ValidationResult();

        if (user.Age < 13)
        {
            result.AddError("Vous devez avoir au moins 13 ans pour vous inscrire");
        }
        else if (user.Age > 120)
        {
            result.AddError("Âge invalide");
        }

        Console.WriteLine($"[AgeValidator] {(result.IsValid ? "✓" : "✗")} Age: {user.Age}");
        return result;
    }
}

public class CountryValidator : ValidationHandler<UserRegistration>
{
    private List<string> allowedCountries = new List<string> { "France", "Belgium", "Switzerland", "Canada" };

    protected override ValidationResult DoValidate(UserRegistration user)
    {
        var result = new ValidationResult();

        if (string.IsNullOrEmpty(user.Country))
        {
            result.AddError("Le pays est requis");
        }
        else if (!allowedCountries.Contains(user.Country))
        {
            result.AddError($"Le pays '{user.Country}' n'est pas pris en charge");
        }

        Console.WriteLine($"[CountryValidator] {(result.IsValid ? "✓" : "✗")} {user.Country}");
        return result;
    }
}

// Validation pipeline builder
public class ValidationPipelineBuilder<T>
{
    private ValidationHandler<T> firstHandler;
    private ValidationHandler<T> lastHandler;

    public ValidationPipelineBuilder<T> AddValidator(ValidationHandler<T> validator)
    {
        if (firstHandler == null)
        {
            firstHandler = validator;
            lastHandler = validator;
        }
        else
        {
            lastHandler.SetNext(validator);
            lastHandler = validator;
        }
        return this;
    }

    public ValidationHandler<T> Build()
    {
        return firstHandler;
    }
}

// Utilisation du pipeline de validation
var validationPipeline = new ValidationPipelineBuilder<UserRegistration>()
    .AddValidator(new UsernameValidator())
    .AddValidator(new EmailValidator())
    .AddValidator(new PasswordValidator())
    .AddValidator(new AgeValidator())
    .AddValidator(new CountryValidator())
    .Build();

// Test avec un utilisateur valide
var validUser = new UserRegistration
{
    Username = "jean_dupont",
    Email = "jean.dupont@email.com",
    Password = "Password123",
    Age = 25,
    Country = "France"
};

Console.WriteLine("=== Validation d'un utilisateur valide ===");
var validResult = validationPipeline.Validate(validUser);
Console.WriteLine($"Résultat: {(validResult.IsValid ? "SUCCÈS" : "ÉCHEC")}");

// Test avec un utilisateur invalide
var invalidUser = new UserRegistration
{
    Username = "jd",
    Email = "invalid-email",
    Password = "weak",
    Age = 10,
    Country = "Mars"
};

Console.WriteLine("\n=== Validation d'un utilisateur invalide ===");
var invalidResult = validationPipeline.Validate(invalidUser);
Console.WriteLine($"Résultat: {(invalidResult.IsValid ? "SUCCÈS" : "ÉCHEC")}");
Console.WriteLine("Erreurs:");
foreach (var error in invalidResult.Errors)
{
    Console.WriteLine($"  - {error}");
}
```

## 19.3.8. Mediator - "L'intermédiaire"

### Qu'est-ce que le Mediator ?

Le Mediator définit un objet qui **encapsule les interactions** entre un ensemble d'objets. Il évite aux objets de se référencer explicitement les uns les autres.

### Quand l'utiliser ?

- Pour réduire les dépendances entre objets
- Quand la logique d'interaction devient complexe
- Pour centraliser la communication

### Exemple simple : Chat room

```csharp
// Interface Mediator
public interface IChatMediator
{
    void SendMessage(string message, User sender);
    void AddUser(User user);
    void RemoveUser(User user);
}

// Mediator concret
public class ChatRoom : IChatMediator
{
    private List<User> users = new List<User>();
    private Dictionary<User, DateTime> lastActivity = new Dictionary<User, DateTime>();

    public void AddUser(User user)
    {
        users.Add(user);
        lastActivity[user] = DateTime.Now;
        Console.WriteLine($"[ChatRoom] {user.Name} a rejoint le chat");
        BroadcastUserList();
    }

    public void RemoveUser(User user)
    {
        users.Remove(user);
        lastActivity.Remove(user);
        Console.WriteLine($"[ChatRoom] {user.Name} a quitté le chat");
        BroadcastUserList();
    }

    public void SendMessage(string message, User sender)
    {
        lastActivity[sender] = DateTime.Now;
        Console.WriteLine($"[{sender.Name}] {message}");

        // Envoyer à tous les autres utilisateurs
        foreach (var user in users)
        {
            if (user != sender)
            {
                user.ReceiveMessage(message, sender);
            }
        }
    }

    private void BroadcastUserList()
    {
        string userList = string.Join(", ", users.Select(u => u.Name));
        foreach (var user in users)
        {
            user.UpdateUserList(userList);
        }
    }

    public void ShowActiveUsers()
    {
        Console.WriteLine("\n=== Utilisateurs actifs ===");
        foreach (var user in users)
        {
            var timeSinceLastActivity = DateTime.Now - lastActivity[user];
            Console.WriteLine($"{user.Name} (actif il y a {timeSinceLastActivity.TotalMinutes:F1} minutes)");
        }
    }
}

// Colleague - utilisateur
public class User
{
    private IChatMediator mediator;
    public string Name { get; private set; }
    private List<string> messageHistory = new List<string>();

    public User(string name, IChatMediator mediator)
    {
        Name = name;
        this.mediator = mediator;
    }

    public void SendMessage(string message)
    {
        mediator.SendMessage(message, this);
    }

    public void ReceiveMessage(string message, User sender)
    {
        string fullMessage = $"[{sender.Name}] {message}";
        messageHistory.Add(fullMessage);
        Console.WriteLine($"  -> {Name} reçoit: {fullMessage}");
    }

    public void UpdateUserList(string userList)
    {
        Console.WriteLine($"  -> {Name} voit les utilisateurs: {userList}");
    }

    public void ShowMessageHistory()
    {
        Console.WriteLine($"\n=== Historique des messages de {Name} ===");
        foreach (var message in messageHistory)
        {
            Console.WriteLine($"  {message}");
        }
    }
}

// Utilisation
var chatRoom = new ChatRoom();

var alice = new User("Alice", chatRoom);
var bob = new User("Bob", chatRoom);
var charlie = new User("Charlie", chatRoom);

chatRoom.AddUser(alice);
chatRoom.AddUser(bob);

alice.SendMessage("Salut tout le monde!");
bob.SendMessage("Salut Alice!");

chatRoom.AddUser(charlie);
charlie.SendMessage("Salut les amis!");

alice.SendMessage("Salut Charlie!");

chatRoom.ShowActiveUsers();
alice.ShowMessageHistory();
```

### Exemple simple : Système de chat

```csharp
// Interface Mediator
public interface IChatRoom
{
    void RegisterUser(User user);
    void SendMessage(string from, string to, string message);
    void SendBroadcast(string from, string message);
}

// Mediator concret
public class ChatRoom : IChatRoom
{
    private Dictionary<string, User> users = new Dictionary<string, User>();

    public void RegisterUser(User user)
    {
        users[user.Name] = user;
        SendBroadcast("Système", $"{user.Name} a rejoint le chat");
    }

    public void SendMessage(string from, string to, string message)
    {
        if (users.ContainsKey(to))
        {
            users[to].ReceiveMessage(from, message);
        }
        else
        {
            if (users.ContainsKey(from))
            {
                users[from].ReceiveMessage("Système", $"Utilisateur {to} introuvable");
            }
        }
    }

    public void SendBroadcast(string from, string message)
    {
        foreach (var user in users.Values)
        {
            if (user.Name != from)
            {
                user.ReceiveMessage(from, message);
            }
        }
    }
}

// Colleague
public class User
{
    public string Name { get; private set; }
    private IChatRoom chatRoom;

    public User(string name, IChatRoom chatRoom)
    {
        Name = name;
        this.chatRoom = chatRoom;
    }

    public void SendMessage(string to, string message)
    {
        Console.WriteLine($"[{Name}] envoie à {to}: {message}");
        chatRoom.SendMessage(Name, to, message);
    }

    public void SendBroadcast(string message)
    {
        Console.WriteLine($"[{Name}] broadcast: {message}");
        chatRoom.SendBroadcast(Name, message);
    }

    public void ReceiveMessage(string from, string message)
    {
        Console.WriteLine($"[{Name}] reçoit de {from}: {message}");
    }
}

// Utilisation
var chatRoom = new ChatRoom();

var alice = new User("Alice", chatRoom);
var bob = new User("Bob", chatRoom);
var charlie = new User("Charlie", chatRoom);

chatRoom.RegisterUser(alice);
chatRoom.RegisterUser(bob);
chatRoom.RegisterUser(charlie);

alice.SendMessage("Bob", "Salut Bob!");
bob.SendMessage("Alice", "Salut Alice! Ça va?");
charlie.SendBroadcast("Bonjour tout le monde!");
alice.SendMessage("David", "Salut!"); // David n'existe pas
```

### Exemple avancé : Système de contrôle aérien

```csharp
// Mediator pour contrôle aérien
public interface IAirTrafficControl
{
    void RegisterAircraft(Aircraft aircraft);
    void RequestTakeoff(Aircraft aircraft);
    void RequestLanding(Aircraft aircraft);
    void ReportPosition(Aircraft aircraft, string position);
    void ReportEmergency(Aircraft aircraft, string situation);
}

// Aircraft (Colleague)
public abstract class Aircraft
{
    public string CallSign { get; protected set; }
    protected IAirTrafficControl atc;
    protected string currentStatus;

    public Aircraft(string callSign, IAirTrafficControl atc)
    {
        CallSign = callSign;
        this.atc = atc;
        currentStatus = "On Ground";
    }

    public virtual void RequestTakeoff()
    {
        Console.WriteLine($"{CallSign}: Demande de décollage");
        atc.RequestTakeoff(this);
    }

    public virtual void RequestLanding()
    {
        Console.WriteLine($"{CallSign}: Demande d'atterrissage");
        atc.RequestLanding(this);
    }

    public virtual void ReportPosition(string position)
    {
        Console.WriteLine($"{CallSign}: Position actuelle - {position}");
        atc.ReportPosition(this, position);
    }

    public virtual void DeclareEmergency(string situation)
    {
        Console.WriteLine($"{CallSign}: URGENCE - {situation}");
        atc.ReportEmergency(this, situation);
    }

    public virtual void ReceiveInstruction(string instruction)
    {
        Console.WriteLine($"{CallSign}: Reçu - {instruction}");
    }

    public string GetStatus() => currentStatus;

    public void SetStatus(string status)
    {
        currentStatus = status;
        Console.WriteLine($"{CallSign}: Statut changé à {status}");
    }
}

public class PassengerAircraft : Aircraft
{
    public PassengerAircraft(string callSign, IAirTrafficControl atc) : base(callSign, atc) { }
}

public class CargoAircraft : Aircraft
{
    public CargoAircraft(string callSign, IAirTrafficControl atc) : base(callSign, atc) { }
}

public class EmergencyAircraft : Aircraft
{
    public EmergencyAircraft(string callSign, IAirTrafficControl atc) : base(callSign, atc) { }

    public override void RequestLanding()
    {
        Console.WriteLine($"{CallSign}: Demande d'atterrissage prioritaire (Ambulance)");
        atc.RequestLanding(this);
    }
}

// Mediator concret
public class AirTrafficControlCenter : IAirTrafficControl
{
    private List<Aircraft> activeAircraft = new List<Aircraft>();
    private List<string> availableRunways = new List<string> { "Runway 01", "Runway 02", "Runway 03" };
    private Dictionary<string, Aircraft> runwayAssignments = new Dictionary<string, Aircraft>();
    private int takeoffQueue = 0;
    private int landingQueue = 0;

    public void RegisterAircraft(Aircraft aircraft)
    {
        activeAircraft.Add(aircraft);
        Console.WriteLine($"Tour de contrôle: {aircraft.CallSign} enregistré dans l'espace aérien");

        // Donner les instructions initiales
        aircraft.ReceiveInstruction("Bienvenue, maintenez votre position actuelle");
    }

    public void RequestTakeoff(Aircraft aircraft)
    {
        takeoffQueue++;

        if (aircraft is EmergencyAircraft)
        {
            // Priorité aux véhicules d'urgence
            AssignRunwayForTakeoff(aircraft, true);
        }
        else if (landingQueue > 0)
        {
            aircraft.ReceiveInstruction($"Attente de décollage, {landingQueue} avions en attente d'atterrissage");
        }
        else
        {
            AssignRunwayForTakeoff(aircraft, false);
        }
    }

    public void RequestLanding(Aircraft aircraft)
    {
        landingQueue++;

        if (aircraft is EmergencyAircraft)
        {
            // Priorité absolue aux urgences
            foreach (var runway in runwayAssignments.ToList())
            {
                runway.Value.ReceiveInstruction("Dégagez immédiatement la piste, urgence médicale");
                runwayAssignments.Remove(runway.Key);
            }
            AssignRunwayForLanding(aircraft, true);
        }
        else
        {
            aircraft.ReceiveInstruction($"Entrez dans le circuit d'attente, {landingQueue} avions avant vous");
            AssignRunwayForLanding(aircraft, false);
        }
    }

    public void ReportPosition(Aircraft aircraft, string position)
    {
        // Vérifier les conflits potentiels
        foreach (var other in activeAircraft)
        {
            if (other != aircraft && other.CallSign != aircraft.CallSign)
            {
                // Simulation simple de détection de proximité
                if (position.Contains("Approche") && other.GetStatus().Contains("Approche"))
                {
                    aircraft.ReceiveInstruction("Attention, trafic détecté à proximité");
                    other.ReceiveInstruction($"Trafic détecté: {aircraft.CallSign}");
                }
            }
        }
    }

    public void ReportEmergency(Aircraft aircraft, string situation)
    {
        // Alerter tous les avions
        foreach (var other in activeAircraft)
        {
            if (other != aircraft)
            {
                other.ReceiveInstruction($"Urgence déclarée par {aircraft.CallSign}: {situation}");
            }
        }

        // Libérer une piste immédiatement
        if (runwayAssignments.Count > 0)
        {
            var firstRunway = runwayAssignments.First();
            firstRunway.Value.ReceiveInstruction("Libérez immédiatement la piste, urgence en cours");
            runwayAssignments.Remove(firstRunway.Key);
        }

        // Assigner une piste à l'avion en urgence
        if (availableRunways.Count > 0)
        {
            string runway = availableRunways[0];
            runwayAssignments[runway] = aircraft;
            aircraft.ReceiveInstruction($"Piste {runway} autorisée pour atterrissage d'urgence");
            aircraft.SetStatus("Emergency Landing");
        }
    }

    private bool AssignRunwayForTakeoff(Aircraft aircraft, bool priority)
    {
        foreach (var runway in availableRunways)
        {
            if (!runwayAssignments.ContainsKey(runway))
            {
                runwayAssignments[runway] = aircraft;
                aircraft.ReceiveInstruction($"Piste {runway} autorisée pour décollage");
                aircraft.SetStatus("Takeoff Cleared");
                takeoffQueue--;

                // Simuler la libération de la piste après le décollage
                Task.Delay(5000).ContinueWith(_ => ReleaseRunway(runway));
                return true;
            }
        }

        if (!priority)
        {
            aircraft.ReceiveInstruction("Toutes les pistes occupées, maintenez votre position");
        }

        return false;
    }

    private bool AssignRunwayForLanding(Aircraft aircraft, bool priority)
    {
        foreach (var runway in availableRunways)
        {
            if (!runwayAssignments.ContainsKey(runway))
            {
                runwayAssignments[runway] = aircraft;
                aircraft.ReceiveInstruction($"Piste {runway} autorisée pour atterrissage");
                aircraft.SetStatus("Landing Cleared");
                landingQueue--;

                // Simuler la libération de la piste après l'atterrissage
                Task.Delay(8000).ContinueWith(_ => ReleaseRunway(runway));
                return true;
            }
        }

        if (!priority)
        {
            aircraft.ReceiveInstruction("Circuit d'attente");
        }

        return false;
    }

    private void ReleaseRunway(string runway)
    {
        if (runwayAssignments.ContainsKey(runway))
        {
            var aircraft = runwayAssignments[runway];
            runwayAssignments.Remove(runway);
            Console.WriteLine($"Tour de contrôle: {runway} maintenant disponible");

            // Vérifier s'il y a des avions en attente
            if (takeoffQueue > 0 || landingQueue > 0)
            {
                Console.WriteLine("Tour de contrôle: Traitement de la file d'attente...");
            }
        }
    }
}

// Utilisation
var atc = new AirTrafficControlCenter();

var flight1 = new PassengerAircraft("AA123", atc);
var flight2 = new PassengerAircraft("BA456", atc);
var cargo1 = new CargoAircraft("FX789", atc);
var ambulance = new EmergencyAircraft("MED001", atc);

atc.RegisterAircraft(flight1);
atc.RegisterAircraft(flight2);
atc.RegisterAircraft(cargo1);
atc.RegisterAircraft(ambulance);

// Simulation de trafic
flight1.RequestTakeoff();
flight2.RequestLanding();
cargo1.RequestTakeoff();

// Simuler une urgence
Task.Delay(2000).ContinueWith(_ =>
{
    ambulance.DeclareEmergency("Problème médical à bord");
    ambulance.RequestLanding();
});

// Simuler des positions
Task.Delay(1000).ContinueWith(_ =>
{
    flight1.ReportPosition("En montée, 5000 pieds");
    flight2.ReportPosition("Approche finale");
});
```

## 19.3.9. Visitor - "Nouvelle opération sans modifier"

### Qu'est-ce que le Visitor ?

Le Visitor permet de **définir de nouvelles opérations** sans changer les classes des éléments sur lesquels il opère.

### Quand l'utiliser ?

- Pour ajouter des opérations à une structure d'objets sans la modifier
- Quand vous avez une structure stable mais des opérations qui changent
- Pour regrouper des opérations liées dans une seule classe

### Exemple simple : Calculatrice d'expressions

```csharp
// Interface Element
public interface IExpressionElement
{
    void Accept(IExpressionVisitor visitor);
}

// Visitor interface
public interface IExpressionVisitor
{
    void Visit(NumberExpression number);
    void Visit(AddExpression add);
    void Visit(SubtractExpression subtract);
    void Visit(MultiplyExpression multiply);
}

// Elements concrets
public class NumberExpression : IExpressionElement
{
    public double Value { get; private set; }

    public NumberExpression(double value)
    {
        Value = value;
    }

    public void Accept(IExpressionVisitor visitor)
    {
        visitor.Visit(this);
    }

    public override string ToString() => Value.ToString();
}

public abstract class BinaryExpression : IExpressionElement
{
    public IExpressionElement Left { get; protected set; }
    public IExpressionElement Right { get; protected set; }
    protected string Operator { get; set; }

    public BinaryExpression(IExpressionElement left, IExpressionElement right)
    {
        Left = left;
        Right = right;
    }

    public abstract void Accept(IExpressionVisitor visitor);

    public override string ToString() => $"({Left} {Operator} {Right})";
}

public class AddExpression : BinaryExpression
{
    public AddExpression(IExpressionElement left, IExpressionElement right)
        : base(left, right)
    {
        Operator = "+";
    }

    public override void Accept(IExpressionVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class SubtractExpression : BinaryExpression
{
    public SubtractExpression(IExpressionElement left, IExpressionElement right)
        : base(left, right)
    {
        Operator = "-";
    }

    public override void Accept(IExpressionVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class MultiplyExpression : BinaryExpression
{
    public MultiplyExpression(IExpressionElement left, IExpressionElement right)
        : base(left, right)
    {
        Operator = "*";
    }

    public override void Accept(IExpressionVisitor visitor)
    {
        visitor.Visit(this);
    }
}

// Visitors concrets
public class EvaluatorVisitor : IExpressionVisitor
{
    public double Result { get; private set; }

    public void Visit(NumberExpression number)
    {
        Result = number.Value;
    }

    public void Visit(AddExpression add)
    {
        add.Left.Accept(this);
        double leftValue = Result;

        add.Right.Accept(this);
        double rightValue = Result;

        Result = leftValue + rightValue;
    }

    public void Visit(SubtractExpression subtract)
    {
        subtract.Left.Accept(this);
        double leftValue = Result;

        subtract.Right.Accept(this);
        double rightValue = Result;

        Result = leftValue - rightValue;
    }

    public void Visit(MultiplyExpression multiply)
    {
        multiply.Left.Accept(this);
        double leftValue = Result;

        multiply.Right.Accept(this);
        double rightValue = Result;

        Result = leftValue * rightValue;
    }
}

public class PrinterVisitor : IExpressionVisitor
{
    private StringBuilder sb = new StringBuilder();
    private int indentLevel = 0;

    public string GetResult() => sb.ToString();

    public void Visit(NumberExpression number)
    {
        AppendLine($"Number: {number.Value}");
    }

    public void Visit(AddExpression add)
    {
        AppendLine("Addition:");
        indentLevel++;
        add.Left.Accept(this);
        add.Right.Accept(this);
        indentLevel--;
    }

    public void Visit(SubtractExpression subtract)
    {
        AppendLine("Subtraction:");
        indentLevel++;
        subtract.Left.Accept(this);
        subtract.Right.Accept(this);
        indentLevel--;
    }

    public void Visit(MultiplyExpression multiply)
    {
        AppendLine("Multiplication:");
        indentLevel++;
        multiply.Left.Accept(this);
        multiply.Right.Accept(this);
        indentLevel--;
    }

    private void AppendLine(string text)
    {
        sb.AppendLine(new string(' ', indentLevel * 2) + text);
    }
}

// Utilisation
// Expression: (5 + 3) * (10 - 4)
var expression = new MultiplyExpression(
    new AddExpression(
        new NumberExpression(5),
        new NumberExpression(3)
    ),
    new SubtractExpression(
        new NumberExpression(10),
        new NumberExpression(4)
    )
);

// Évaluer l'expression
var evaluator = new EvaluatorVisitor();
expression.Accept(evaluator);
Console.WriteLine($"Résultat: {evaluator.Result}");

// Afficher la structure
var printer = new PrinterVisitor();
expression.Accept(printer);
Console.WriteLine("\nStructure de l'expression:");
Console.WriteLine(printer.GetResult());

// Expression textuelle
Console.WriteLine($"Expression: {expression}");
```

### Exemple avancé : Compilateur simple

```csharp
// AST Node interface
public interface IASTNode
{
    void Accept(IASTVisitor visitor);
}

// AST Visitors
public interface IASTVisitor
{
    void Visit(Program program);
    void Visit(VariableDeclaration variable);
    void Visit(FunctionDeclaration function);
    void Visit(AssignmentStatement assignment);
    void Visit(PrintStatement print);
    void Visit(Identifier identifier);
    void Visit(Literal literal);
    void Visit(BinaryOperation operation);
}

// AST Nodes
public class Program : IASTNode
{
    public List<IASTNode> Statements { get; set; } = new List<IASTNode>();

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class VariableDeclaration : IASTNode
{
    public string Type { get; set; }
    public string Name { get; set; }
    public IASTNode Initializer { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class FunctionDeclaration : IASTNode
{
    public string ReturnType { get; set; }
    public string Name { get; set; }
    public List<(string Type, string Name)> Parameters { get; set; } = new List<(string, string)>();
    public Program Body { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class AssignmentStatement : IASTNode
{
    public string Variable { get; set; }
    public IASTNode Expression { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class PrintStatement : IASTNode
{
    public IASTNode Expression { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class Identifier : IASTNode
{
    public string Name { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class Literal : IASTNode
{
    public object Value { get; set; }
    public string Type { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class BinaryOperation : IASTNode
{
    public IASTNode Left { get; set; }
    public string Operator { get; set; }
    public IASTNode Right { get; set; }

    public void Accept(IASTVisitor visitor)
    {
        visitor.Visit(this);
    }
}

// Visitors concrets
public class CodeGeneratorVisitor : IASTVisitor
{
    private StringBuilder code = new StringBuilder();
    private int indentLevel = 0;
    private Dictionary<string, string> variables = new Dictionary<string, string>();

    public string GetGeneratedCode() => code.ToString();

    public void Visit(Program program)
    {
        AppendLine("// Generated C# Code");
        AppendLine("using System;");
        AppendLine("");
        AppendLine("class GeneratedProgram");
        AppendLine("{");
        indentLevel++;

        AppendLine("static void Main(string[] args)");
        AppendLine("{");
        indentLevel++;

        foreach (var statement in program.Statements)
        {
            statement.Accept(this);
        }

        indentLevel--;
        AppendLine("}");
        indentLevel--;
        AppendLine("}");
    }

    public void Visit(VariableDeclaration variable)
    {
        variables[variable.Name] = variable.Type;
        Append($"{variable.Type} {variable.Name}");

        if (variable.Initializer != null)
        {
            Append(" = ");
            variable.Initializer.Accept(this);
        }

        AppendLine(";");
    }

    public void Visit(FunctionDeclaration function)
    {
        Append($"{function.ReturnType} {function.Name}(");

        for (int i = 0; i < function.Parameters.Count; i++)
        {
            if (i > 0) Append(", ");
            Append($"{function.Parameters[i].Type} {function.Parameters[i].Name}");
        }

        AppendLine(")");
        AppendLine("{");
        indentLevel++;

        function.Body.Accept(this);

        indentLevel--;
        AppendLine("}");
    }

    public void Visit(AssignmentStatement assignment)
    {
        Append($"{assignment.Variable} = ");
        assignment.Expression.Accept(this);
        AppendLine(";");
    }

    public void Visit(PrintStatement print)
    {
        Append("Console.WriteLine(");
        print.Expression.Accept(this);
        AppendLine(");");
    }

    public void Visit(Identifier identifier)
    {
        Append(identifier.Name);
    }

    public void Visit(Literal literal)
    {
        if (literal.Type == "string")
        {
            Append($"\"{literal.Value}\"");
        }
        else
        {
            Append(literal.Value.ToString());
        }
    }

    public void Visit(BinaryOperation operation)
    {
        Append("(");
        operation.Left.Accept(this);
        Append($" {operation.Operator} ");
        operation.Right.Accept(this);
        Append(")");
    }

    private void AppendLine(string text = "")
    {
        if (!string.IsNullOrEmpty(text))
            code.AppendLine(new string(' ', indentLevel * 4) + text);
        else
            code.AppendLine();
    }

    private void Append(string text)
    {
        if (code.Length == 0 || code[code.Length - 1] == '\n')
            code.Append(new string(' ', indentLevel * 4));
        code.Append(text);
    }
}

public class ASTPrinterVisitor : IASTVisitor
{
    private StringBuilder output = new StringBuilder();
    private int indentLevel = 0;

    public string GetOutput() => output.ToString();

    public void Visit(Program program)
    {
        AppendLine("Program");
        indentLevel++;
        foreach (var statement in program.Statements)
        {
            statement.Accept(this);
        }
        indentLevel--;
    }

    public void Visit(VariableDeclaration variable)
    {
        AppendLine($"VariableDeclaration: {variable.Type} {variable.Name}");
        if (variable.Initializer != null)
        {
            indentLevel++;
            AppendLine("Initializer:");
            indentLevel++;
            variable.Initializer.Accept(this);
            indentLevel -= 2;
        }
    }

    public void Visit(FunctionDeclaration function)
    {
        AppendLine($"FunctionDeclaration: {function.ReturnType} {function.Name}");
        indentLevel++;

        AppendLine("Parameters:");
        indentLevel++;
        foreach (var param in function.Parameters)
        {
            AppendLine($"{param.Type} {param.Name}");
        }
        indentLevel--;

        AppendLine("Body:");
        indentLevel++;
        function.Body.Accept(this);
        indentLevel--;

        indentLevel--;
    }

    public void Visit(AssignmentStatement assignment)
    {
        AppendLine($"Assignment: {assignment.Variable} =");
        indentLevel++;
        assignment.Expression.Accept(this);
        indentLevel--;
    }

    public void Visit(PrintStatement print)
    {
        AppendLine("PrintStatement:");
        indentLevel++;
        print.Expression.Accept(this);
        indentLevel--;
    }

    public void Visit(Identifier identifier)
    {
        AppendLine($"Identifier: {identifier.Name}");
    }

    public void Visit(Literal literal)
    {
        AppendLine($"Literal: {literal.Value} ({literal.Type})");
    }

    public void Visit(BinaryOperation operation)
    {
        AppendLine($"BinaryOperation: {operation.Operator}");
        indentLevel++;
        AppendLine("Left:");
        indentLevel++;
        operation.Left.Accept(this);
        indentLevel--;
        AppendLine("Right:");
        indentLevel++;
        operation.Right.Accept(this);
        indentLevel--;
        indentLevel--;
    }

    private void AppendLine(string text)
    {
        output.AppendLine(new string(' ', indentLevel * 2) + text);
    }
}

// Utilisation du compilateur simple
// Créer un AST simple : calculer x = 10 + 5 et afficher le résultat
var program = new Program();

// Déclarer une variable
var varDecl = new VariableDeclaration
{
    Type = "int",
    Name = "x",
    Initializer = new BinaryOperation
    {
        Left = new Literal { Value = 10, Type = "int" },
        Operator = "+",
        Right = new Literal { Value = 5, Type = "int" }
    }
};

// Instruction d'affichage
var printStmt = new PrintStatement
{
    Expression = new Identifier { Name = "x" }
};

program.Statements.Add(varDecl);
program.Statements.Add(printStmt);

// Générer du code C#
var codeGen = new CodeGeneratorVisitor();
program.Accept(codeGen);
Console.WriteLine("Code généré:");
Console.WriteLine(codeGen.GetGeneratedCode());

// Afficher la structure de l'AST
var astPrinter = new ASTPrinterVisitor();
program.Accept(astPrinter);
Console.WriteLine("\nStructure de l'AST:");
Console.WriteLine(astPrinter.GetOutput());
```

## Résumé des Patterns Comportementaux

| Pattern | But principal | Cas d'utilisation typique |
|---------|--------------|--------------------------|
| **Observer** | Notification de changements | Systèmes d'événements, MVC |
| **Strategy** | Algorithmes interchangeables | Différentes méthodes de paiement |
| **Command** | Encapsuler une requête | Undo/Redo, Macro, Queue |
| **Template Method** | Squelette d'algorithme | Frameworks, Processus similaires |
| **Iterator** | Parcours de collections | Collections personnalisées |
| **State** | Comportement selon l'état | Machines à états, Workflow |
| **Chain of Responsibility** | Chaîne de traitement | Validation, Support client |
| **Mediator** | Centraliser la communication | Chat, Systèmes complexes |
| **Visitor** | Nouvelles opérations | Compilateurs, Structures complexes |

## Bonnes pratiques pour les Patterns Comportementaux

### 1. Observer
- Évitez les fuites mémoire (détacher les observateurs)
- Considérez les performances avec beaucoup d'observateurs
- Utilisez `WeakEventManager` pour WPF

### 2. Strategy
- Gardez les stratégies sans état quand c'est possible
- Utilisez une factory pour créer les stratégies
- Documentez quand utiliser chaque stratégie

### 3. Command
- Séparez clairement la commande de son exécution
- Stockez les paramètres nécessaires pour undo/redo
- Groupez les commandes connexes

### 4. Template Method
- Faites attention à l'ordre des étapes
- Documentez clairement les hooks
- Évitez trop de méthodes abstraites

### 5. Iterator
- Respectez les contrats IEnumerable/.NET
- Considérez la sécurité des threads
- Gérez correctement les modifications pendant l'itération

### 6. State
- Limitez le nombre d'états pour la lisibilité
- Documentez les transitions possibles
- Considérez une machine à états pour la complexité

### 7. Chain of Responsibility
- Évitez les chaînes trop longues
- Gérez le cas où personne ne peut traiter
- Documentez l'ordre de la chaîne

### 8. Mediator
- Évitez que le mediator devienne un "god object"
- Séparez les responsabilités logiquement
- Considérez plusieurs mediators pour différents domaines

### 9. Visitor
- Évitez de modifier souvent la structure visitée
- Utilisez le double dispatch intelligemment
- Considérez l'impact sur la performance

## Exercices pratiques

### Exercice 1 : Observer
Créez un système de bourse où différentes vues (graphique, tableau, alertes) s'actualisent quand le prix d'une action change.

### Exercice 2 : Strategy
Implémentez un système de tri avec différentes stratégies (bubble sort, quick sort, merge sort) que l'utilisateur peut choisir selon la taille des données.

### Exercice 3 : Command
Créez un éditeur de texte simple avec undo/redo pour les opérations d'insertion, suppression et remplacement.

### Exercice 4 : State
Modélisez une machine à café avec différents états (disponible, en préparation, maintenance, etc.) et les transitions entre eux.

### Exercice 5 : Visitor
Créez un système d'analyse de code source qui peut calculer différentes métriques (complexité cyclomatique, nombre de lignes, etc.) sans modifier l'AST.

## Combinaison de Patterns

Les patterns comportementaux fonctionnent souvent bien ensemble :

```csharp
// Exemple : Combinaison Strategy + Observer
public class ProcessingEngine
{
    private IProcessingStrategy strategy;
    private event EventHandler<ProcessingEventArgs> ProcessingStatusChanged;

    public void SetStrategy(IProcessingStrategy newStrategy)
    {
        strategy = newStrategy;
        NotifyStrategyChanged();
    }

    public async Task Process(Data data)
    {
        NotifyProcessingStarted();

        var result = await strategy.Process(data);

        NotifyProcessingCompleted(result);
    }

    private void NotifyProcessingStarted()
    {
        ProcessingStatusChanged?.Invoke(this,
            new ProcessingEventArgs { Status = "Started" });
    }

    private void NotifyProcessingCompleted(ProcessingResult result)
    {
        ProcessingStatusChanged?.Invoke(this,
            new ProcessingEventArgs { Status = "Completed", Result = result });
    }
}

// Combinaison Command + Memento pour undo/redo
public class EditorCommand : ICommand
{
    private IOriginator originator;
    private Memento memento;

    public EditorCommand(IOriginator originator)
    {
        this.originator = originator;
        memento = originator.CreateMemento();
    }

    public void Execute()
    {
        // Exécuter l'action
    }

    public void Undo()
    {
        originator.RestoreMemento(memento);
    }
}
```

## Conclusion

Les patterns comportementaux sont essentiels pour :

1. **Gérer la complexité des interactions** entre objets
2. **Rendre le code plus flexible** et maintenable
3. **Encapsuler les algorithms** de manière réutilisable
4. **Découpler les composants** pour faciliter les modifications

### Points clés à retenir :

- **Choisissez le bon pattern** selon le problème spécifique
- **Ne surchargez pas** votre code - utilisez les patterns quand ils apportent une valeur
- **Documentez vos choix** pour les autres développeurs
- **Testez thoroughly** - les patterns peuvent introduire de la complexité
- **Combinez intelligemment** les patterns quand c'est approprié

### Pour aller plus loin :

1. Pratiquez avec des projets réels
2. Lisez le "Gang of Four" pour approfondir
3. Explorez les patterns spécifiques à votre domaine
4. Participez à des code reviews pour voir comment d'autres utilisent les patterns
5. Créez vos propres variations adaptées à vos besoins

Les patterns sont des outils puissants, mais rappelez-vous que **la simplicité est souvent la meilleure solution**. Utilisez les patterns quand ils simplifient réellement votre code, pas quand ils le compliquent.

⏭️ 19.4. [Patterns architecturaux](/19-design-patterns-en-csharp/19-4-patterns-architecturaux.md)
