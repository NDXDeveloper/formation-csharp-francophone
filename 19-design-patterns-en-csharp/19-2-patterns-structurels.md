# 19.2. Patterns structurels en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les **patterns structurels** sont des modèles de conception qui s'occupent de la manière dont les objets et les classes sont composés pour former des structures plus larges. Ils facilitent la création de structures complexes en simplifiant les relations entre objets.

## Qu'est-ce qu'un Pattern Structurel ?

Les patterns structurels vous aident à :
- **Assembler des objets et des classes** en structures plus grandes
- **Simplifier les interfaces complexes**
- **Gérer les relations entre objets** de manière flexible
- **Adapter des interfaces incompatibles**

## 19.2.1. Adapter - "Le traducteur"

### Qu'est-ce que l'Adapter ?

L'Adapter permet à **deux interfaces incompatibles de travailler ensemble**. C'est comme un adaptateur électrique qui vous permet d'utiliser un appareil français aux États-Unis.

### Quand l'utiliser ?

- Quand vous devez utiliser une classe existante avec une interface incompatible
- Pour intégrer des bibliothèques tierces
- Pour faire évoluer votre code sans casser l'existant

### Exemple simple : Adaptateur de médias

```csharp
// Interface cible que notre code attend
public interface IMediaPlayer
{
    void Play(string filename);
}

// Classe existante avec une interface différente
public class AdvancedMediaPlayer
{
    public void PlayVlc(string filename)
    {
        Console.WriteLine($"Lecture VLC: {filename}");
    }

    public void PlayMp4(string filename)
    {
        Console.WriteLine($"Lecture MP4: {filename}");
    }
}

// Adapter qui fait le pont entre les deux interfaces
public class MediaAdapter : IMediaPlayer
{
    private AdvancedMediaPlayer advancedPlayer;

    public MediaAdapter(string fileType)
    {
        advancedPlayer = new AdvancedMediaPlayer();
    }

    public void Play(string filename)
    {
        string extension = Path.GetExtension(filename).ToLower();

        switch (extension)
        {
            case ".vlc":
                advancedPlayer.PlayVlc(filename);
                break;
            case ".mp4":
                advancedPlayer.PlayMp4(filename);
                break;
            default:
                Console.WriteLine($"Format {extension} non supporté");
                break;
        }
    }
}

// Player principal qui utilise l'adaptateur
public class AudioPlayer : IMediaPlayer
{
    private MediaAdapter mediaAdapter;

    public void Play(string filename)
    {
        string extension = Path.GetExtension(filename).ToLower();

        if (extension == ".mp3")
        {
            Console.WriteLine($"Lecture MP3: {filename}");
        }
        else
        {
            // Utiliser l'adaptateur pour les autres formats
            mediaAdapter = new MediaAdapter(extension);
            mediaAdapter.Play(filename);
        }
    }
}

// Utilisation
var player = new AudioPlayer();
player.Play("music.mp3");        // Lecture MP3: music.mp3
player.Play("movie.mp4");        // Lecture MP4: movie.mp4
player.Play("video.vlc");        // Lecture VLC: video.vlc
```

### Exemple avec intégration de bibliothèque tierce

```csharp
// Interface de notre système
public interface ILogger
{
    void LogError(string message);
    void LogInfo(string message);
}

// Bibliothèque tierce avec une interface différente
public class ThirdPartyLogger
{
    public void WriteError(string msg, DateTime timestamp)
    {
        Console.WriteLine($"[ERROR {timestamp}] {msg}");
    }

    public void WriteInformation(string msg, DateTime timestamp)
    {
        Console.WriteLine($"[INFO {timestamp}] {msg}");
    }
}

// Adapter pour intégrer la bibliothèque tierce
public class ThirdPartyLoggerAdapter : ILogger
{
    private ThirdPartyLogger thirdPartyLogger;

    public ThirdPartyLoggerAdapter()
    {
        thirdPartyLogger = new ThirdPartyLogger();
    }

    public void LogError(string message)
    {
        thirdPartyLogger.WriteError(message, DateTime.Now);
    }

    public void LogInfo(string message)
    {
        thirdPartyLogger.WriteInformation(message, DateTime.Now);
    }
}

// Utilisation
ILogger logger = new ThirdPartyLoggerAdapter();
logger.LogInfo("Application démarrée");
logger.LogError("Une erreur est survenue");
```

## 19.2.2. Bridge - "Séparer l'abstraction de l'implémentation"

### Qu'est-ce que le Bridge ?

Le Bridge **sépare une abstraction de son implémentation**, permettant aux deux de varier indépendamment.

### Quand l'utiliser ?

- Quand vous voulez éviter une explosion combinatoire de classes
- Pour pouvoir changer l'implémentation sans affecter le client
- Quand l'abstraction et l'implémentation doivent évoluer séparément

### Exemple simple : Formes et couleurs

```csharp
// Interface implementation (ce qui est "sous" le pont)
public interface IColor
{
    string GetColor();
}

// Implémentations concrètes
public class RedColor : IColor
{
    public string GetColor()
    {
        return "Rouge";
    }
}

public class BlueColor : IColor
{
    public string GetColor()
    {
        return "Bleu";
    }
}

// Abstraction (ce qui est "au-dessus" du pont)
public abstract class Shape
{
    protected IColor color;

    protected Shape(IColor color)
    {
        this.color = color;
    }

    public abstract void Draw();
}

// Abstractions raffinées
public class Circle : Shape
{
    public Circle(IColor color) : base(color)
    {
    }

    public override void Draw()
    {
        Console.WriteLine($"Dessiner un cercle {color.GetColor()}");
    }
}

public class Square : Shape
{
    public Square(IColor color) : base(color)
    {
    }

    public override void Draw()
    {
        Console.WriteLine($"Dessiner un carré {color.GetColor()}");
    }
}

// Utilisation
var redCircle = new Circle(new RedColor());
var blueSquare = new Square(new BlueColor());

redCircle.Draw();    // Dessiner un cercle Rouge
blueSquare.Draw();   // Dessiner un carré Bleu
```

### Exemple plus avancé : Appareils et télécommandes

```csharp
// Interface implementation (Device)
public interface IDevice
{
    void TurnOn();
    void TurnOff();
    void SetVolume(int volume);
    int GetVolume();
    bool IsEnabled();
}

// Implémentations concrètes
public class TV : IDevice
{
    private bool isOn = false;
    private int volume = 50;

    public void TurnOn()
    {
        isOn = true;
        Console.WriteLine("TV allumée");
    }

    public void TurnOff()
    {
        isOn = false;
        Console.WriteLine("TV éteinte");
    }

    public void SetVolume(int volume)
    {
        this.volume = Math.Max(0, Math.Min(100, volume));
        Console.WriteLine($"Volume TV: {this.volume}");
    }

    public int GetVolume() => volume;
    public bool IsEnabled() => isOn;
}

public class Radio : IDevice
{
    private bool isOn = false;
    private int volume = 30;

    public void TurnOn()
    {
        isOn = true;
        Console.WriteLine("Radio allumée");
    }

    public void TurnOff()
    {
        isOn = false;
        Console.WriteLine("Radio éteinte");
    }

    public void SetVolume(int volume)
    {
        this.volume = Math.Max(0, Math.Min(100, volume));
        Console.WriteLine($"Volume Radio: {this.volume}");
    }

    public int GetVolume() => volume;
    public bool IsEnabled() => isOn;
}

// Abstraction (Remote)
public class RemoteControl
{
    protected IDevice device;

    public RemoteControl(IDevice device)
    {
        this.device = device;
    }

    public void TogglePower()
    {
        if (device.IsEnabled())
        {
            device.TurnOff();
        }
        else
        {
            device.TurnOn();
        }
    }

    public void VolumeUp()
    {
        device.SetVolume(device.GetVolume() + 10);
    }

    public void VolumeDown()
    {
        device.SetVolume(device.GetVolume() - 10);
    }
}

// Abstraction raffinée
public class AdvancedRemoteControl : RemoteControl
{
    public AdvancedRemoteControl(IDevice device) : base(device)
    {
    }

    public void Mute()
    {
        device.SetVolume(0);
        Console.WriteLine("Appareil mis en silencieux");
    }
}

// Utilisation
var tv = new TV();
var remote = new AdvancedRemoteControl(tv);

remote.TogglePower();  // TV allumée
remote.VolumeUp();     // Volume TV: 60
remote.Mute();         // Appareil mis en silencieux
```

## 19.2.3. Composite - "Traiter les objets et les composites de manière uniforme"

### Qu'est-ce que le Composite ?

Le Composite permet de **composer des objets en structures d'arbre** et de travailler avec ces structures comme avec des objets individuels.

### Quand l'utiliser ?

- Pour représenter des hiérarchies d'objets
- Quand vous voulez traiter les objets individuels et les compositions de manière uniforme
- Pour créer des structures récursives

### Exemple simple : Système de fichiers

```csharp
// Composant abstrait
public abstract class FileSystemComponent
{
    protected string name;

    public FileSystemComponent(string name)
    {
        this.name = name;
    }

    public abstract void Display(int depth = 0);
    public abstract long GetSize();

    protected string GetIndent(int depth)
    {
        return new string(' ', depth * 2);
    }
}

// Feuille (Leaf)
public class File : FileSystemComponent
{
    private long size;

    public File(string name, long size) : base(name)
    {
        this.size = size;
    }

    public override void Display(int depth = 0)
    {
        Console.WriteLine($"{GetIndent(depth)}- {name} ({size} bytes)");
    }

    public override long GetSize()
    {
        return size;
    }
}

// Composite
public class Folder : FileSystemComponent
{
    private List<FileSystemComponent> components = new List<FileSystemComponent>();

    public Folder(string name) : base(name)
    {
    }

    public void Add(FileSystemComponent component)
    {
        components.Add(component);
    }

    public void Remove(FileSystemComponent component)
    {
        components.Remove(component);
    }

    public override void Display(int depth = 0)
    {
        Console.WriteLine($"{GetIndent(depth)}+ {name}/");
        foreach (var component in components)
        {
            component.Display(depth + 1);
        }
    }

    public override long GetSize()
    {
        return components.Sum(c => c.GetSize());
    }
}

// Utilisation
var root = new Folder("Root");
var documents = new Folder("Documents");
var photos = new Folder("Photos");

var file1 = new File("resume.docx", 24576);
var file2 = new File("presentation.pptx", 2048000);
var photo1 = new File("beach.jpg", 1048576);
var photo2 = new File("sunset.jpg", 2097152);

documents.Add(file1);
documents.Add(file2);
photos.Add(photo1);
photos.Add(photo2);

root.Add(documents);
root.Add(photos);

root.Display();
Console.WriteLine($"\nTaille totale: {root.GetSize()} bytes");

/* Output:
+ Root/
  + Documents/
    - resume.docx (24576 bytes)
    - presentation.pptx (2048000 bytes)
  + Photos/
    - beach.jpg (1048576 bytes)
    - sunset.jpg (2097152 bytes)

Taille totale: 5218304 bytes
*/
```

### Exemple avancé : Interface graphique

```csharp
// Composant abstrait
public abstract class UIComponent
{
    protected string name;
    protected int x, y, width, height;

    public UIComponent(string name, int x, int y, int width, int height)
    {
        this.name = name;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }

    public abstract void Draw();
    public abstract void HandleClick(int clickX, int clickY);
}

// Feuilles
public class Button : UIComponent
{
    public Button(string name, int x, int y, int width, int height)
        : base(name, x, y, width, height)
    {
    }

    public override void Draw()
    {
        Console.WriteLine($"Bouton '{name}' à ({x}, {y}) - {width}x{height}");
    }

    public override void HandleClick(int clickX, int clickY)
    {
        if (clickX >= x && clickX <= x + width &&
            clickY >= y && clickY <= y + height)
        {
            Console.WriteLine($"Bouton '{name}' cliqué!");
        }
    }
}

public class TextBox : UIComponent
{
    public TextBox(string name, int x, int y, int width, int height)
        : base(name, x, y, width, height)
    {
    }

    public override void Draw()
    {
        Console.WriteLine($"TextBox '{name}' à ({x}, {y}) - {width}x{height}");
    }

    public override void HandleClick(int clickX, int clickY)
    {
        if (clickX >= x && clickX <= x + width &&
            clickY >= y && clickY <= y + height)
        {
            Console.WriteLine($"TextBox '{name}' sélectionnée!");
        }
    }
}

// Composite
public class Panel : UIComponent
{
    private List<UIComponent> components = new List<UIComponent>();

    public Panel(string name, int x, int y, int width, int height)
        : base(name, x, y, width, height)
    {
    }

    public void Add(UIComponent component)
    {
        components.Add(component);
    }

    public override void Draw()
    {
        Console.WriteLine($"Panel '{name}' à ({x}, {y}) - {width}x{height}");
        foreach (var component in components)
        {
            component.Draw();
        }
    }

    public override void HandleClick(int clickX, int clickY)
    {
        // Transformer les coordonnées relatives au panel
        int relativeX = clickX - x;
        int relativeY = clickY - y;

        foreach (var component in components)
        {
            component.HandleClick(relativeX, relativeY);
        }
    }
}

// Utilisation
var mainWindow = new Panel("Fenêtre principale", 0, 0, 800, 600);
var loginPanel = new Panel("Panel de connexion", 50, 50, 300, 200);

var usernameBox = new TextBox("Username", 10, 30, 200, 30);
var passwordBox = new TextBox("Password", 10, 80, 200, 30);
var loginButton = new Button("Se connecter", 10, 130, 100, 40);

loginPanel.Add(usernameBox);
loginPanel.Add(passwordBox);
loginPanel.Add(loginButton);

mainWindow.Add(loginPanel);

mainWindow.Draw();
mainWindow.HandleClick(120, 180); // Clic sur le bouton
```

## 19.2.4. Decorator - "Ajouter des responsabilités dynamiquement"

### Qu'est-ce que le Decorator ?

Le Decorator permet d'**ajouter dynamiquement des fonctionnalités** à un objet sans modifier sa structure.

### Quand l'utiliser ?

- Pour étendre les fonctionnalités d'un objet sans sous-classer
- Quand vous avez besoin de combinaisons flexibles de comportements
- Pour appliquer des responsabilités qu'on peut retirer

### Exemple simple : Café avec options

```csharp
// Interface composant
public interface ICoffee
{
    string GetDescription();
    double GetCost();
}

// Composant de base
public class SimpleCoffee : ICoffee
{
    public string GetDescription()
    {
        return "Café simple";
    }

    public double GetCost()
    {
        return 1.0;
    }
}

// Décorateur abstrait
public abstract class CoffeeDecorator : ICoffee
{
    protected ICoffee coffee;

    public CoffeeDecorator(ICoffee coffee)
    {
        this.coffee = coffee;
    }

    public abstract string GetDescription();
    public abstract double GetCost();
}

// Décorateurs concrets
public class Milk : CoffeeDecorator
{
    public Milk(ICoffee coffee) : base(coffee)
    {
    }

    public override string GetDescription()
    {
        return coffee.GetDescription() + ", Lait";
    }

    public override double GetCost()
    {
        return coffee.GetCost() + 0.5;
    }
}

public class Sugar : CoffeeDecorator
{
    public Sugar(ICoffee coffee) : base(coffee)
    {
    }

    public override string GetDescription()
    {
        return coffee.GetDescription() + ", Sucre";
    }

    public override double GetCost()
    {
        return coffee.GetCost() + 0.2;
    }
}

public class WhippedCream : CoffeeDecorator
{
    public WhippedCream(ICoffee coffee) : base(coffee)
    {
    }

    public override string GetDescription()
    {
        return coffee.GetDescription() + ", Crème fouettée";
    }

    public override double GetCost()
    {
        return coffee.GetCost() + 0.7;
    }
}

// Utilisation
ICoffee myCoffee = new SimpleCoffee();
Console.WriteLine($"{myCoffee.GetDescription()} = ${myCoffee.GetCost()}");

// Ajouter du lait
myCoffee = new Milk(myCoffee);
Console.WriteLine($"{myCoffee.GetDescription()} = ${myCoffee.GetCost()}");

// Ajouter du sucre
myCoffee = new Sugar(myCoffee);
Console.WriteLine($"{myCoffee.GetDescription()} = ${myCoffee.GetCost()}");

// Ajouter de la crème fouettée
myCoffee = new WhippedCream(myCoffee);
Console.WriteLine($"{myCoffee.GetDescription()} = ${myCoffee.GetCost()}");

/* Output:
Café simple = $1
Café simple, Lait = $1.5
Café simple, Lait, Sucre = $1.7
Café simple, Lait, Sucre, Crème fouettée = $2.4
*/
```

### Exemple avancé : Système de compression de fichiers

```csharp
// Interface composant
public interface IDataSource
{
    void WriteData(byte[] data);
    byte[] ReadData();
}

// Composant de base
public class FileDataSource : IDataSource
{
    private string filename;

    public FileDataSource(string filename)
    {
        this.filename = filename;
    }

    public void WriteData(byte[] data)
    {
        File.WriteAllBytes(filename, data);
        Console.WriteLine($"Données écrites dans {filename}");
    }

    public byte[] ReadData()
    {
        byte[] data = File.ReadAllBytes(filename);
        Console.WriteLine($"Données lues depuis {filename}");
        return data;
    }
}

// Décorateur abstrait
public abstract class DataSourceDecorator : IDataSource
{
    protected IDataSource dataSource;

    public DataSourceDecorator(IDataSource dataSource)
    {
        this.dataSource = dataSource;
    }

    public abstract void WriteData(byte[] data);
    public abstract byte[] ReadData();
}

// Décorateur de compression
public class CompressionDecorator : DataSourceDecorator
{
    public CompressionDecorator(IDataSource dataSource) : base(dataSource)
    {
    }

    public override void WriteData(byte[] data)
    {
        byte[] compressedData = Compress(data);
        Console.WriteLine($"Compression des données ({data.Length} -> {compressedData.Length} bytes)");
        dataSource.WriteData(compressedData);
    }

    public override byte[] ReadData()
    {
        byte[] compressedData = dataSource.ReadData();
        byte[] decompressedData = Decompress(compressedData);
        Console.WriteLine($"Décompression des données ({compressedData.Length} -> {decompressedData.Length} bytes)");
        return decompressedData;
    }

    private byte[] Compress(byte[] data)
    {
        // Simulation de compression
        using (var output = new MemoryStream())
        {
            using (var compressor = new System.IO.Compression.GZipStream(output, System.IO.Compression.CompressionLevel.Optimal))
            {
                compressor.Write(data, 0, data.Length);
            }
            return output.ToArray();
        }
    }

    private byte[] Decompress(byte[] data)
    {
        // Simulation de décompression
        using (var input = new MemoryStream(data))
        using (var decompressor = new System.IO.Compression.GZipStream(input, System.IO.Compression.CompressionMode.Decompress))
        using (var output = new MemoryStream())
        {
            decompressor.CopyTo(output);
            return output.ToArray();
        }
    }
}

// Décorateur de chiffrement
public class EncryptionDecorator : DataSourceDecorator
{
    public EncryptionDecorator(IDataSource dataSource) : base(dataSource)
    {
    }

    public override void WriteData(byte[] data)
    {
        byte[] encryptedData = Encrypt(data);
        Console.WriteLine($"Chiffrement des données");
        dataSource.WriteData(encryptedData);
    }

    public override byte[] ReadData()
    {
        byte[] encryptedData = dataSource.ReadData();
        byte[] decryptedData = Decrypt(encryptedData);
        Console.WriteLine($"Déchiffrement des données");
        return decryptedData;
    }

    private byte[] Encrypt(byte[] data)
    {
        // Simulation simple de chiffrement (XOR)
        byte[] result = new byte[data.Length];
        byte key = 123; // Clé simple pour l'exemple

        for (int i = 0; i < data.Length; i++)
        {
            result[i] = (byte)(data[i] ^ key);
        }

        return result;
    }

    private byte[] Decrypt(byte[] data)
    {
        // Le déchiffrement XOR est identique au chiffrement
        return Encrypt(data);
    }
}

// Utilisation
string filename = "data.txt";
byte[] testData = Encoding.UTF8.GetBytes("Ceci est un message secret!");

// Source de données simple
IDataSource dataSource = new FileDataSource(filename);

// Ajouter compression et chiffrement
dataSource = new CompressionDecorator(dataSource);
dataSource = new EncryptionDecorator(dataSource);

// Écrire les données (elles seront chiffrées puis compressées)
dataSource.WriteData(testData);

// Lire les données (elles seront décompressées puis déchiffrées)
byte[] readData = dataSource.ReadData();
string result = Encoding.UTF8.GetString(readData);
Console.WriteLine($"Données récupérées: {result}");
```

## 19.2.5. Façade - "Interface simplifiée"

### Qu'est-ce que la Façade ?

La Façade fournit une **interface simplifiée** pour un sous-système complexe.

### Quand l'utiliser ?

- Pour masquer la complexité d'un système
- Pour fournir un point d'entrée unique
- Pour réduire les dépendances entre systèmes

### Exemple simple : Système home cinéma

```csharp
// Composants complexes du système
public class TV
{
    public void TurnOn() => Console.WriteLine("TV allumée");
    public void TurnOff() => Console.WriteLine("TV éteinte");
    public void SetInput(string input) => Console.WriteLine($"TV entrée: {input}");
}

public class SoundSystem
{
    public void TurnOn() => Console.WriteLine("Système audio allumé");
    public void TurnOff() => Console.WriteLine("Système audio éteint");
    public void SetVolume(int volume) => Console.WriteLine($"Volume: {volume}");
    public void SetSurroundSound() => Console.WriteLine("Son surround activé");
}

public class DVDPlayer
{
    public void TurnOn() => Console.WriteLine("Lecteur DVD allumé");
    public void TurnOff() => Console.WriteLine("Lecteur DVD éteint");
    public void Play(string movie) => Console.WriteLine($"Lecture: {movie}");
    public void Stop() => Console.WriteLine("Arrêt de la lecture");
}

public class Projector
{
    public void TurnOn() => Console.WriteLine("Projecteur allumé");
    public void TurnOff() => Console.WriteLine("Projecteur éteint");
    public void WideScreenMode() => Console.WriteLine("Mode écran large");
}

public class Lights
{
    public void Dim(int level) => Console.WriteLine($"Lumières tamisées: {level}%");
    public void TurnOn() => Console.WriteLine("Lumières allumées");
}

// Façade qui simplifie l'utilisation
public class HomeTheaterFacade
{
    private TV tv;
    private SoundSystem soundSystem;
    private DVDPlayer dvdPlayer;
    private Projector projector;
    private Lights lights;

    public HomeTheaterFacade()
    {
        tv = new TV();
        soundSystem = new SoundSystem();
        dvdPlayer = new DVDPlayer();
        projector = new Projector();
        lights = new Lights();
    }

    public void WatchMovie(string movie)
    {
        Console.WriteLine("Préparation pour regarder un film...");

        lights.Dim(10);
        projector.TurnOn();
        projector.WideScreenMode();
        tv.TurnOn();
        tv.SetInput("DVD");
        soundSystem.TurnOn();
        soundSystem.SetSurroundSound();
        soundSystem.SetVolume(50);
        dvdPlayer.TurnOn();
        dvdPlayer.Play(movie);

        Console.WriteLine("Film prêt à être regardé!");
    }

    public void EndMovie()
    {
        Console.WriteLine("Fin du film...");

        dvdPlayer.Stop();
        dvdPlayer.TurnOff();
        soundSystem.TurnOff();
        tv.TurnOff();
        projector.TurnOff();
        lights.TurnOn();

        Console.WriteLine("Système éteint!");
    }
}

// Utilisation
var homeTheater = new HomeTheaterFacade();
homeTheater.WatchMovie("Inception");
// ... après avoir regardé le film
homeTheater.EndMovie();
```

### Exemple avancé : API d'envoi de notifications

```csharp
// Services complexes
public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        Console.WriteLine($"Email envoyé à {to}: {subject}");
    }
}

public class SMSService
{
    public void SendSMS(string phoneNumber, string message)
    {
        Console.WriteLine($"SMS envoyé à {phoneNumber}: {message}");
    }
}

public class PushNotificationService
{
    public void SendPushNotification(string deviceId, string title, string message)
    {
        Console.WriteLine($"Push envoyé à {deviceId}: {title} - {message}");
    }
}

public class SlackService
{
    public void SendSlackMessage(string channel, string message)
    {
        Console.WriteLine($"Message Slack envoyé à #{channel}: {message}");
    }
}

// Base de données pour récupérer les informations
public class UserDatabase
{
    private Dictionary<string, UserInfo> users = new Dictionary<string, UserInfo>
    {
        {"user1", new UserInfo { Email = "user1@email.com", Phone = "1234567890", DeviceId = "device123" } },
        {"user2", new UserInfo { Email = "user2@email.com", Phone = "0987654321", DeviceId = "device456" } }
    };

    public UserInfo GetUserInfo(string userId)
    {
        users.TryGetValue(userId, out var userInfo);
        return userInfo ?? new UserInfo();
    }
}

public class UserInfo
{
    public string Email { get; set; }
    public string Phone { get; set; }
    public string DeviceId { get; set; }
}

// Façade pour simplifier l'envoi de notifications
public class NotificationFacade
{
    private EmailService emailService;
    private SMSService smsService;
    private PushNotificationService pushService;
    private SlackService slackService;
    private UserDatabase userDatabase;

    public NotificationFacade()
    {
        emailService = new EmailService();
        smsService = new SMSService();
        pushService = new PushNotificationService();
        slackService = new SlackService();
        userDatabase = new UserDatabase();
    }

    public void SendAllNotifications(string userId, string subject, string message)
    {
        var userInfo = userDatabase.GetUserInfo(userId);

        // Envoyer par tous les canaux disponibles
        if (!string.IsNullOrEmpty(userInfo.Email))
        {
            emailService.SendEmail(userInfo.Email, subject, message);
        }

        if (!string.IsNullOrEmpty(userInfo.Phone))
        {
            // Simplifier le message pour SMS
            string shortMessage = message.Length > 160 ? message.Substring(0, 157) + "..." : message;
            smsService.SendSMS(userInfo.Phone, shortMessage);
        }

        if (!string.IsNullOrEmpty(userInfo.DeviceId))
        {
            pushService.SendPushNotification(userInfo.DeviceId, subject, message);
        }
    }

    public void SendUrgentAlert(string subject, string message)
    {
        Console.WriteLine($"ALERTE URGENTE: {subject}");

        // Envoyer à tous les canaux d'urgence
        slackService.SendSlackMessage("alerts", $"🚨 {subject}: {message}");

        // Envoyer à tous les utilisateurs
        var allUsers = new[] { "user1", "user2" }; // Simplified for example
        foreach (var userId in allUsers)
        {
            SendAllNotifications(userId, $"URGENT: {subject}", message);
        }
    }

    public void SendWelcomeMessage(string userId)
    {
        var userInfo = userDatabase.GetUserInfo(userId);
        string subject = "Bienvenue dans notre application!";
        string message = "Merci de nous avoir rejoint. Voici comment commencer...";

        // Séquence de bienvenue personnalisée
        emailService.SendEmail(userInfo.Email, subject, message);

        // Envoyer aussi une push notification après 5 secondes
        if (!string.IsNullOrEmpty(userInfo.DeviceId))
        {
            pushService.SendPushNotification(userInfo.DeviceId, "N'oubliez pas...",
                "Consultez votre email pour des conseils utiles");
        }
    }
}

// Utilisation simplifiée
var notifications = new NotificationFacade();

// Envoyer une notification simple
notifications.SendAllNotifications("user1", "Nouveau message", "Vous avez un nouveau message");

// Envoyer une alerte urgente
notifications.SendUrgentAlert("Maintenance système", "Le système sera indisponible de 2h à 4h");

// Envoyer un message de bienvenue
notifications.SendWelcomeMessage("user2");
```

## 19.2.6. Proxy - "Représentant ou substitut"

### Qu'est-ce que le Proxy ?

Le Proxy fournit un **substitut ou un représentant** d'un autre objet pour contrôler l'accès à celui-ci.

### Types de Proxy courants :

1. **Virtual Proxy** : Charge un objet à la demande
2. **Protection Proxy** : Contrôle l'accès à un objet
3. **Remote Proxy** : Représente un objet distant
4. **Cache Proxy** : Met en cache des résultats

### Exemple simple : Virtual Proxy pour images

```csharp
// Interface commune
public interface IImage
{
    void Display();
    void Resize(int width, int height);
}

// Objet réel (coûteux à créer)
public class RealImage : IImage
{
    private string filename;
    private byte[] imageData;
    private int width;
    private int height;

    public RealImage(string filename)
    {
        this.filename = filename;
        LoadImageFromDisk();
    }

    private void LoadImageFromDisk()
    {
        Console.WriteLine($"Chargement de l'image {filename} depuis le disque...");
        // Simulation du chargement coûteux
        System.Threading.Thread.Sleep(1000);

        // Simuler le chargement de données
        imageData = new byte[1024 * 1024]; // 1MB
        width = 1920;
        height = 1080;

        Console.WriteLine($"Image {filename} chargée ({width}x{height})");
    }

    public void Display()
    {
        Console.WriteLine($"Affichage de l'image {filename} ({width}x{height})");
    }

    public void Resize(int width, int height)
    {
        this.width = width;
        this.height = height;
        Console.WriteLine($"Image {filename} redimensionnée à {width}x{height}");
    }
}

// Proxy qui charge l'image à la demande
public class ImageProxy : IImage
{
    private string filename;
    private RealImage realImage;

    public ImageProxy(string filename)
    {
        this.filename = filename;
    }

    public void Display()
    {
        // Charger l'image réelle seulement si nécessaire
        if (realImage == null)
        {
            realImage = new RealImage(filename);
        }
        realImage.Display();
    }

    public void Resize(int width, int height)
    {
        // Certaines opérations peuvent être faites sans charger l'image
        Console.WriteLine($"Préparation du redimensionnement à {width}x{height}");

        // Charger seulement si vraiment nécessaire
        if (realImage == null)
        {
            realImage = new RealImage(filename);
        }
        realImage.Resize(width, height);
    }
}

// Utilisation
Console.WriteLine("Création du proxy...");
IImage image = new ImageProxy("grande_image.jpg");
Console.WriteLine("Proxy créé, image non encore chargée");

// L'image est chargée seulement maintenant
image.Display();

// Redimensionnement
image.Resize(800, 600);
```

### Exemple avancé : Protection Proxy

```csharp
// Interface pour l'accès aux données sensibles
public interface ISecureDatabase
{
    string ReadData(string query);
    void WriteData(string data);
    void DeleteData(string condition);
}

// Objet réel avec accès à la base de données
public class SecureDatabase : ISecureDatabase
{
    public string ReadData(string query)
    {
        Console.WriteLine($"Exécution requête: {query}");
        return "Données sensibles...";
    }

    public void WriteData(string data)
    {
        Console.WriteLine($"Écriture données: {data}");
    }

    public void DeleteData(string condition)
    {
        Console.WriteLine($"Suppression données: {condition}");
    }
}

// Système d'authentification
public enum UserRole
{
    Admin,
    User,
    Guest
}

public class User
{
    public string Name { get; set; }
    public UserRole Role { get; set; }
    public string Password { get; set; }
}

// Protection Proxy avec authentification et autorisation
public class DatabaseProtectionProxy : ISecureDatabase
{
    private SecureDatabase realDatabase;
    private User currentUser;
    private Dictionary<string, User> users;

    public DatabaseProtectionProxy()
    {
        realDatabase = new SecureDatabase();
        // Initialiser quelques utilisateurs pour l'exemple
        users = new Dictionary<string, User>
        {
            {"admin", new User { Name = "admin", Role = UserRole.Admin, Password = "admin123" }},
            {"user1", new User { Name = "user1", Role = UserRole.User, Password = "user123" }},
            {"guest", new User { Name = "guest", Role = UserRole.Guest, Password = "guest123" }}
        };
    }

    public bool Login(string username, string password)
    {
        if (users.TryGetValue(username, out var user) && user.Password == password)
        {
            currentUser = user;
            Console.WriteLine($"Connexion réussie en tant que {user.Name} ({user.Role})");
            return true;
        }

        Console.WriteLine("Échec de la connexion");
        return false;
    }

    public void Logout()
    {
        Console.WriteLine($"Déconnexion de {currentUser?.Name}");
        currentUser = null;
    }

    public string ReadData(string query)
    {
        if (currentUser == null)
        {
            throw new UnauthorizedAccessException("Vous devez être connecté");
        }

        if (currentUser.Role == UserRole.Guest)
        {
            throw new UnauthorizedAccessException("Les invités ne peuvent pas lire les données");
        }

        Console.WriteLine($"[{currentUser.Name}] Accès en lecture autorisé");
        return realDatabase.ReadData(query);
    }

    public void WriteData(string data)
    {
        if (currentUser == null)
        {
            throw new UnauthorizedAccessException("Vous devez être connecté");
        }

        if (currentUser.Role != UserRole.Admin)
        {
            throw new UnauthorizedAccessException("Seuls les administrateurs peuvent écrire");
        }

        Console.WriteLine($"[{currentUser.Name}] Accès en écriture autorisé");
        realDatabase.WriteData(data);
    }

    public void DeleteData(string condition)
    {
        if (currentUser == null)
        {
            throw new UnauthorizedAccessException("Vous devez être connecté");
        }

        if (currentUser.Role != UserRole.Admin)
        {
            throw new UnauthorizedAccessException("Seuls les administrateurs peuvent supprimer");
        }

        // Double vérification pour les suppressions
        Console.WriteLine($"ATTENTION: Tentative de suppression par {currentUser.Name}");
        Console.WriteLine("Confirmer la suppression? (y/N)");
        // Dans un vrai système, on attendrait la confirmation
        Console.WriteLine("Suppression confirmée");
        realDatabase.DeleteData(condition);
    }
}

// Utilisation
var database = new DatabaseProtectionProxy();

// Tentative sans connexion
try
{
    database.ReadData("SELECT * FROM users");
}
catch (UnauthorizedAccessException ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}

// Connexion en tant qu'utilisateur normal
database.Login("user1", "user123");
var data = database.ReadData("SELECT * FROM public_data");

try
{
    database.WriteData("INSERT INTO users...");
}
catch (UnauthorizedAccessException ex)
{
    Console.WriteLine($"Erreur: {ex.Message}");
}

// Connexion en tant qu'admin
database.Login("admin", "admin123");
database.WriteData("INSERT INTO logs...");
database.DeleteData("WHERE date < '2023-01-01'");
```

## 19.2.7. Flyweight - "Partager pour économiser"

### Qu'est-ce que le Flyweight ?

Le Flyweight permet d'**économiser de la mémoire** en partageant efficacement des données communes entre plusieurs objets.

### Concepts clés :

- **État intrinsèque** : données partagées entre objets
- **État extrinsèque** : données uniques à chaque objet
- **Factory** : gère la création et le partage des flyweights

### Exemple simple : Caractères dans un traitement de texte

```csharp
// Flyweight stockant l'état intrinsèque (partagé)
public class CharacterStyle
{
    public string Font { get; private set; }
    public int Size { get; private set; }
    public string Color { get; private set; }

    public CharacterStyle(string font, int size, string color)
    {
        Font = font;
        Size = size;
        Color = color;
    }

    public void Display(char character, int x, int y)
    {
        Console.WriteLine($"Caractère '{character}' à ({x},{y}) - {Font}, {Size}pt, {Color}");
    }
}

// Factory pour gérer les flyweights
public class CharacterStyleFactory
{
    private Dictionary<string, CharacterStyle> styles = new Dictionary<string, CharacterStyle>();

    public CharacterStyle GetStyle(string font, int size, string color)
    {
        string key = $"{font}-{size}-{color}";

        if (!styles.ContainsKey(key))
        {
            styles[key] = new CharacterStyle(font, size, color);
            Console.WriteLine($"Nouveau style créé: {key}");
        }
        else
        {
            Console.WriteLine($"Style réutilisé: {key}");
        }

        return styles[key];
    }

    public int GetStyleCount()
    {
        return styles.Count;
    }
}

// Contexte stockant l'état extrinsèque (unique)
public class Character
{
    private char character;
    private int x;
    private int y;
    private CharacterStyle style; // Référence partagée

    public Character(char character, int x, int y, CharacterStyle style)
    {
        this.character = character;
        this.x = x;
        this.y = y;
        this.style = style;
    }

    public void Display()
    {
        style.Display(character, x, y);
    }
}

// Document utilisant les caractères
public class Document
{
    private List<Character> characters = new List<Character>();
    private CharacterStyleFactory styleFactory = new CharacterStyleFactory();

    public void AddCharacter(char character, int x, int y, string font, int size, string color)
    {
        CharacterStyle style = styleFactory.GetStyle(font, size, color);
        characters.Add(new Character(character, x, y, style));
    }

    public void Display()
    {
        foreach (var character in characters)
        {
            character.Display();
        }

        Console.WriteLine($"\nNombre total de caractères: {characters.Count}");
        Console.WriteLine($"Nombre de styles créés: {styleFactory.GetStyleCount()}");
    }
}

// Utilisation
var document = new Document();

// Ajouter du texte avec différents styles
string text = "Hello World!";
for (int i = 0; i < text.Length; i++)
{
    document.AddCharacter(text[i], i * 10, 10, "Arial", 12, "Black");
}

// Ajouter plus de texte avec certains styles partagés
for (int i = 0; i < "Hello".Length; i++)
{
    document.AddCharacter("Hello"[i], i * 10, 30, "Arial", 12, "Red");
}

document.Display();
```

### Exemple avancé : Particules dans un jeu

```csharp
// Flyweight pour les types de particules
public class ParticleType
{
    public string Name { get; private set; }
    public string Texture { get; private set; }
    public double Mass { get; private set; }
    public string Color { get; private set; }

    public ParticleType(string name, string texture, double mass, string color)
    {
        Name = name;
        Texture = texture;
        Mass = mass;
        Color = color;
    }

    public void Render(double x, double y, double size, double rotation)
    {
        Console.WriteLine($"Particule {Name} à ({x:F1},{y:F1}) " +
                         $"- texture: {Texture}, masse: {Mass}, " +
                         $"couleur: {Color}, taille: {size:F1}, rotation: {rotation:F1}°");
    }

    public void UpdatePhysics(Particle particle, double deltaTime)
    {
        // Calculer les effets physiques basés sur la masse
        particle.VelocityY += 9.81 * Mass * deltaTime; // Gravité

        // Appliquer la vitesse
        particle.X += particle.VelocityX * deltaTime;
        particle.Y += particle.VelocityY * deltaTime;

        // Appliquer la rotation
        particle.Rotation += particle.AngularVelocity * deltaTime;
    }
}

// Contexte pour chaque particule individuelle
public class Particle
{
    public double X { get; set; }
    public double Y { get; set; }
    public double VelocityX { get; set; }
    public double VelocityY { get; set; }
    public double Size { get; set; }
    public double Rotation { get; set; }
    public double AngularVelocity { get; set; }
    public double LifeTime { get; set; }

    private ParticleType type;

    public Particle(ParticleType type, double x, double y, double velocityX, double velocityY)
    {
        this.type = type;
        X = x;
        Y = y;
        VelocityX = velocityX;
        VelocityY = velocityY;
        Size = 1.0;
        Rotation = 0;
        AngularVelocity = new Random().NextDouble() * 360 - 180;
        LifeTime = 5.0;
    }

    public void Update(double deltaTime)
    {
        type.UpdatePhysics(this, deltaTime);
        LifeTime -= deltaTime;

        // Réduire la taille avec le temps
        Size = Math.Max(0.1, LifeTime / 5.0);
    }

    public void Render()
    {
        type.Render(X, Y, Size, Rotation);
    }

    public bool IsAlive()
    {
        return LifeTime > 0;
    }
}

// Factory pour gérer les types de particules
public class ParticleTypeFactory
{
    private Dictionary<string, ParticleType> types = new Dictionary<string, ParticleType>();

    public ParticleType GetParticleType(string name)
    {
        if (!types.ContainsKey(name))
        {
            switch (name)
            {
                case "fire":
                    types[name] = new ParticleType("fire", "fire.png", 0.1, "Orange");
                    break;
                case "smoke":
                    types[name] = new ParticleType("smoke", "smoke.png", 0.05, "Gray");
                    break;
                case "explosion":
                    types[name] = new ParticleType("explosion", "explosion.png", 0.15, "Yellow");
                    break;
                default:
                    types[name] = new ParticleType(name, "default.png", 0.1, "White");
                    break;
            }
            Console.WriteLine($"Nouveau type de particule créé: {name}");
        }

        return types[name];
    }

    public int GetTypeCount()
    {
        return types.Count;
    }
}

// Système de particules
public class ParticleSystem
{
    private List<Particle> particles = new List<Particle>();
    private ParticleTypeFactory typeFactory = new ParticleTypeFactory();
    private Random random = new Random();

    public void CreateExplosion(double x, double y, int count = 50)
    {
        var explosionType = typeFactory.GetParticleType("explosion");
        var fireType = typeFactory.GetParticleType("fire");
        var smokeType = typeFactory.GetParticleType("smoke");

        for (int i = 0; i < count; i++)
        {
            // Angle aléatoire
            double angle = random.NextDouble() * Math.PI * 2;
            double speed = random.NextDouble() * 100 + 50;

            // Vitesses basées sur l'angle
            double vx = Math.Cos(angle) * speed;
            double vy = Math.Sin(angle) * speed;

            // Choisir le type de particule
            ParticleType type;
            if (i < count * 0.2) type = explosionType;
            else if (i < count * 0.6) type = fireType;
            else type = smokeType;

            particles.Add(new Particle(type, x, y, vx, vy));
        }
    }

    public void Update(double deltaTime)
    {
        for (int i = particles.Count - 1; i >= 0; i--)
        {
            particles[i].Update(deltaTime);

            if (!particles[i].IsAlive())
            {
                particles.RemoveAt(i);
            }
        }
    }

    public void Render()
    {
        Console.WriteLine($"\n--- Frame ---");
        foreach (var particle in particles)
        {
            particle.Render();
        }

        Console.WriteLine($"Particules actives: {particles.Count}");
        Console.WriteLine($"Types de particules: {typeFactory.GetTypeCount()}");
    }
}

// Utilisation
var particleSystem = new ParticleSystem();

// Créer une explosion
particleSystem.CreateExplosion(400, 300, 100);

// Simuler quelques frames
for (int frame = 0; frame < 5; frame++)
{
    particleSystem.Update(0.1);
    particleSystem.Render();
    Console.WriteLine("\n-----------------");
}
```

## Résumé des Patterns Structurels

| Pattern | But principal | Cas d'utilisation typique |
|---------|--------------|--------------------------|
| **Adapter** | Adapter des interfaces | Intégration de bibliothèques tierces |
| **Bridge** | Séparer abstraction/implémentation | Éviter l'explosion combinatoire |
| **Composite** | Structures arborescentes | Systèmes de fichiers, UI |
| **Decorator** | Ajouter des responsabilités | Ajout de fonctionnalités flexibles |
| **Façade** | Interface simplifiée | Masquer la complexité |
| **Proxy** | Contrôler l'accès | Chargement lazy, sécurité |
| **Flyweight** | Partager des données | Optimisation mémoire |

## Bonnes pratiques

1. **Choisir le bon pattern** : Analysez votre problème avant de choisir
2. **Éviter la surconception** : N'utilisez pas un pattern s'il complique inutilement le code
3. **Documenter vos choix** : Expliquez pourquoi vous avez choisi un pattern particulier
4. **Tester thoroughly** : Les patterns peuvent introduire de la complexité
5. **Combiner avec parcimonie** : Certains patterns fonctionnent bien ensemble, d'autres non

## Exercices pratiques

### Exercice 1 : Adapter pour base de données
Créez un adaptateur pour utiliser différentes bases de données (MySQL, PostgreSQL, MongoDB) avec une interface commune.

### Exercice 2 : Composite pour menus
Implémentez un système de menus avec des items simples et des sous-menus en utilisant le pattern Composite.

### Exercice 3 : Decorator pour filtres d'image
Créez des décorateurs pour appliquer différents filtres à des images (flou, noir et blanc, luminosité).

### Exercice 4 : Façade pour API complexe
Créez une façade pour simplifier l'utilisation d'une API REST complexe.

### Exercice 5 : Proxy pour cache
Implémentez un proxy de cache pour une base de données qui stocke en mémoire les résultats fréquemment demandés.

## Conclusion

Les patterns structurels vous aident à organiser et assembler des objets de manière efficace. Ils sont particulièrement utiles pour :

- Adapter des composants incompatibles
- Gérer la complexité des systèmes
- Optimiser les performances
- Créer des structures flexibles et extensibles

N'oubliez pas que ces patterns sont des outils, pas des solutions universelles. Utilisez-les quand ils apportent une réelle valeur à votre projet, et n'hésitez pas à les adapter à vos besoins spécifiques.

⏭️ 19.3. [Patterns comportementaux](/19-design-patterns-en-csharp/19-3-patterns-comportementaux.md)
