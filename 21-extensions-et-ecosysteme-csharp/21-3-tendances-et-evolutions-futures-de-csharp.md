# 21.3. Tendances et évolutions futures de C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

C# est un langage en constante évolution, avec de nouvelles versions apportant régulièrement des améliorations et des fonctionnalités innovantes. Dans cette section, nous allons explorer les tendances actuelles et les évolutions futures du langage.

## 21.3.1. Caractéristiques des versions récentes

Voici un aperçu des principales nouveautés introduites dans les versions récentes de C#.

### C# 11 (2022)

#### Raw string literals
```csharp
// Plus besoin d'échapper les guillemets dans les chaînes
string json = """
    {
        "name": "Jean Dupont",
        "email": "jean@example.com",
        "address": {
            "street": "123 rue de la Paix",
            "city": "Paris"
        }
    }
    """;
```

#### UTF-8 string literals
```csharp
// Création directe de tableaux de bytes UTF-8
ReadOnlySpan<byte> bytes = "Hello World"u8;
```

#### Interface members static virtual
```csharp
public interface IMathOperations<T> where T : IMathOperations<T>
{
    static abstract T Zero { get; }
    static abstract T operator +(T left, T right);
}

public struct Point : IMathOperations<Point>
{
    public double X, Y;

    public static Point Zero => new Point(0, 0);

    public static Point operator +(Point left, Point right)
    {
        return new Point(left.X + right.X, left.Y + right.Y);
    }
}
```

#### Pattern matching amélioré
```csharp
// List patterns
if (numbers is [var first, .. var middle, var last])
{
    Console.WriteLine($"First: {first}, Last: {last}");
}

// Or patterns avec des types
object value = GetValue();
if (value is string s or char c)
{
    Console.WriteLine($"Got text: {value}");
}
```

### C# 12 (2023)

#### Primary constructors pour les classes
```csharp
public class Logger(IConsole console)
{
    // console est automatiquement disponible comme membre
    public void Log(string message)
    {
        console.WriteLine($"[{DateTime.Now}] {message}");
    }
}

// Utilisation
var logger = new Logger(new ConsoleWrapper());
logger.Log("Hello World");
```

#### Collection expressions
```csharp
// Syntaxe simplifiée pour créer des collections
int[] numbers = [1, 2, 3, 4, 5];
List<string> names = ["Alice", "Bob", "Charlie"];

// Spread operator
int[] allNumbers = [..numbers, 6, 7, 8];
```

#### Interceptors
```csharp
// Permet d'intercepter des appels de méthode pendant la compilation
[CompilerGenerated]
[InterceptsLocation(@"C:\MyProject\Program.cs", line: 10, character: 25)]
public static void InterceptedMethod(this MyClass instance)
{
    // Code alternatif exécuté à la place
}
```

### C# 13 (2024+)

#### Partial properties
```csharp
public partial class Customer
{
    // Déclaration partielle de propriété
    public partial string Name { get; set; }
}

// Dans un autre fichier
public partial class Customer
{
    // Implémentation partielle
    private string _name;
    public partial string Name
    {
        get => _name;
        set
        {
            if (string.IsNullOrEmpty(value))
                throw new ArgumentException("Name cannot be empty");
            _name = value;
        }
    }
}
```

#### Field-backed properties
```csharp
public class Person
{
    // Le compilateur génère automatiquement le champ
    public required string Name { get; set; }

    // Accès direct au champ généré
    public void SetNameSilently(string value)
    {
        _name = value; // Accès au champ généré
    }
}
```

## 21.3.2. Roadmap C# et .NET

### Vision à long terme

La roadmap de C# et .NET se concentre sur plusieurs axes principaux :

1. **Performance** : Amélioration continue des performances d'exécution
2. **Productivité** : Simplification de la syntaxe et des patterns courants
3. **Cloud-first** : Optimisation pour les déploiements cloud
4. **Cross-platform** : Support améliioré pour tous les OS
5. **Innovation** : Intégration des nouvelles technologies

### Calendrier de release

| Version | Date de sortie | Support |
|---------|---------------|---------|
| .NET 8 | Novembre 2023 | LTS (jusqu'en 2026) |
| .NET 9 | Novembre 2024 | STS (18 mois) |
| .NET 10 | Novembre 2025 | LTS (3 ans) |
| .NET 11 | Novembre 2026 | STS (18 mois) |

### Thèmes futurs

#### Native AOT (Ahead-of-Time compilation)
```csharp
// Configuration pour Native AOT dans le fichier .csproj
<PropertyGroup>
    <PublishAot>true</PublishAot>
</PropertyGroup>
```

Avantages :
- Démarrage instantané des applications
- Empreinte mémoire réduite
- Pas besoin du runtime .NET installé

#### Discrimination des unions (Union Types)
```csharp
// Syntaxe proposée pour les types union
public Result<T, TError> ProcessData<T, TError>(T data)
{
    try
    {
        // Traitement...
        return Result.Success(processedData);
    }
    catch (Exception ex)
    {
        return Result.Error(ex);
    }
}

// Pattern matching amélioré
switch (result)
{
    case Result.Success<T> success:
        Console.WriteLine($"Success: {success.Value}");
        break;
    case Result.Error<TError> error:
        Console.WriteLine($"Error: {error.Value}");
        break;
}
```

## 21.3.3. Source generators

Les Source Generators permettent de générer du code pendant la compilation, offrant des possibilités puissantes pour réduire le boilerplate et améliorer les performances.

### Qu'est-ce qu'un Source Generator ?

Un Source Generator est une classe qui :
- Hérite de `ISourceGenerator` ou implémente `IIncrementalGenerator`
- Analyse le code source pendant la compilation
- Génère du code supplémentaire

### Exemple basique d'un Source Generator

```csharp
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Text;
using System.Text;

[Generator]
public class AutoPropertiesGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        // Filtrer les classes avec l'attribut [GenerateProperties]
        var provider = context.SyntaxProvider
            .CreateSyntaxProvider(
                predicate: (s, _) => s is ClassDeclarationSyntax,
                transform: (ctx, _) => GetClassDeclaration(ctx))
            .Where(static m => m is not null);

        // Générer le code pour chaque classe trouvée
        context.RegisterSourceOutput(provider, GenerateCode);
    }

    private ClassDeclarationSyntax GetClassDeclaration(GeneratorSyntaxContext context)
    {
        var classDeclaration = (ClassDeclarationSyntax)context.Node;
        var classSymbol = context.SemanticModel.GetDeclaredSymbol(classDeclaration);

        if (classSymbol?.GetAttributes().Any(a => a.AttributeClass?.Name == "GeneratePropertiesAttribute") == true)
        {
            return classDeclaration;
        }

        return null;
    }

    private void GenerateCode(SourceProductionContext context, ClassDeclarationSyntax classDeclaration)
    {
        var className = classDeclaration.Identifier.Text;
        var namespaceName = GetNamespace(classDeclaration);

        var sourceBuilder = new StringBuilder();
        sourceBuilder.AppendLine($"namespace {namespaceName}");
        sourceBuilder.AppendLine("{");
        sourceBuilder.AppendLine($"    public partial class {className}");
        sourceBuilder.AppendLine("    {");

        // Générer des propriétés automatiques
        sourceBuilder.AppendLine("        public string AutoProperty1 { get; set; }");
        sourceBuilder.AppendLine("        public int AutoProperty2 { get; set; }");

        sourceBuilder.AppendLine("    }");
        sourceBuilder.AppendLine("}");

        var sourceText = SourceText.From(sourceBuilder.ToString(), Encoding.UTF8);
        context.AddSource($"{className}.g.cs", sourceText);
    }
}
```

### Utilisation du Source Generator

```csharp
// Attribut pour marquer les classes à traiter
[AttributeUsage(AttributeTargets.Class)]
public class GeneratePropertiesAttribute : Attribute { }

// Classe partielle utilisant le générateur
[GenerateProperties]
public partial class Customer
{
    public string CustomProperty { get; set; }

    // Les propriétés AutoProperty1 et AutoProperty2
    // seront générées automatiquement
}
```

### Applications pratiques des Source Generators

#### 1. Génération de serializers optimisés

```csharp
[Generator]
public class JsonSerializerGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        // Génère des serializers JSON optimisés pour des classes spécifiques
    }
}

// Utilisation
[GenerateJsonSerializer]
public partial class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

#### 2. Génération de mappers

```csharp
[Generator]
public class AutoMapperGenerator : IIncrementalGenerator
{
    // Génère automatiquement des méthodes de mapping entre objets
}

// Utilisation
[GenerateMapper(typeof(CustomerDto))]
public partial class Customer
{
    // Méthode ToDto() sera générée automatiquement
}
```

#### 3. Génération de code pour DI

```csharp
[AttributeUsage(AttributeTargets.Assembly)]
public class RegisterServicesAttribute : Attribute { }

[assembly: RegisterServices]

// Le générateur crée automatiquement la configuration DI
// basée sur les attributs de service
```

## 21.3.4. C# sur WebAssembly

WebAssembly (WASM) permet d'exécuter du code C# directement dans le navigateur, ouvrant de nouvelles possibilités pour le développement web.

### Blazor WebAssembly

Blazor WebAssembly permet de créer des applications web frontend entièrement en C#.

```csharp
// Page Blazor simple
@page "/counter"

<h1>Counter</h1>

<p role="status">Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

### Composants avancés Blazor

```csharp
@page "/todo"
@using TodoApp.Data

<h3>Todo List</h3>

@if (todos == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <input @bind="newTodo" placeholder="Enter new todo" />
    <button @onclick="AddTodo">Add</button>

    <ul>
        @foreach (var todo in todos)
        {
            <li>
                <input type="checkbox"
                       checked="@todo.IsCompleted"
                       @onchange="@((e) => ToggleTodo(todo, (bool)e.Value!))" />
                <span style="@(todo.IsCompleted ? "text-decoration: line-through" : "")">
                    @todo.Text
                </span>
                <button @onclick="@(() => RemoveTodo(todo))">Delete</button>
            </li>
        }
    </ul>
}

@code {
    private List<TodoItem>? todos;
    private string newTodo = "";

    protected override async Task OnInitializedAsync()
    {
        todos = await TodoService.GetTodosAsync();
    }

    private async Task AddTodo()
    {
        if (!string.IsNullOrWhiteSpace(newTodo))
        {
            var todo = new TodoItem { Text = newTodo, IsCompleted = false };
            await TodoService.AddTodoAsync(todo);
            todos.Add(todo);
            newTodo = "";
        }
    }

    private async Task ToggleTodo(TodoItem todo, bool isCompleted)
    {
        todo.IsCompleted = isCompleted;
        await TodoService.UpdateTodoAsync(todo);
    }

    private async Task RemoveTodo(TodoItem todo)
    {
        await TodoService.DeleteTodoAsync(todo.Id);
        todos.Remove(todo);
    }
}
```

### Interopérabilité JavaScript

```csharp
// Appeler du JavaScript depuis C#
@inject IJSRuntime JSRuntime

@code {
    protected override async Task OnInitializedAsync()
    {
        // Appeler une fonction JavaScript
        await JSRuntime.InvokeVoidAsync("console.log", "Hello from Blazor!");

        // Récupérer une valeur depuis JavaScript
        var userAgent = await JSRuntime.InvokeAsync<string>("navigator.userAgent");
    }
}
```

### WASI (WebAssembly System Interface)

```csharp
// Exemple de console app compilée en WASM
using System;
using System.IO;

// Application console standard qui peut être compilée en WASM
public class Program
{
    public static void Main(string[] args)
    {
        Console.WriteLine("C# running on WebAssembly!");

        // Accès aux fichiers via WASI
        if (File.Exists("data.txt"))
        {
            var content = File.ReadAllText("data.txt");
            Console.WriteLine($"File content: {content}");
        }
    }
}
```

## 21.3.5. Intelligence artificielle avec ML.NET

ML.NET est le framework d'apprentissage automatique de Microsoft pour .NET, permettant d'intégrer l'IA dans les applications C#.

### Introduction à ML.NET

```csharp
// Installation du package
// dotnet add package Microsoft.ML
// dotnet add package Microsoft.ML.Recommender

using Microsoft.ML;
using Microsoft.ML.Data;
```

### Exemple : Prédiction de sentiment

```csharp
// Modèle de données
public class SentimentData
{
    [LoadColumn(0)]
    public string SentimentText;

    [LoadColumn(1), ColumnName("Label")]
    public bool Sentiment;
}

public class SentimentPrediction
{
    [ColumnName("PredictedLabel")]
    public bool Prediction { get; set; }

    public float Probability { get; set; }

    public float Score { get; set; }
}

// Entraînement et prédiction
public class SentimentAnalysis
{
    private readonly MLContext _mlContext;
    private ITransformer _model;

    public SentimentAnalysis()
    {
        _mlContext = new MLContext();
    }

    public void TrainModel(string dataPath)
    {
        // Charger les données
        var data = _mlContext.Data.LoadFromTextFile<SentimentData>(dataPath,
            hasHeader: true, separatorChar: '\t');

        // Préparer les données
        var split = _mlContext.Data.TrainTestSplit(data, testFraction: 0.2);

        // Créer le pipeline d'entraînement
        var pipeline = _mlContext.Transforms.Text
            .FeaturizeText(outputColumnName: "Features", inputColumnName: nameof(SentimentData.SentimentText))
            .Append(_mlContext.BinaryClassification.Trainers.SdcaLogisticRegression());

        // Entraîner le modèle
        _model = pipeline.Fit(split.TrainSet);

        // Évaluer le modèle
        var predictions = _model.Transform(split.TestSet);
        var metrics = _mlContext.BinaryClassification.Evaluate(predictions);

        Console.WriteLine($"Accuracy: {metrics.Accuracy:F2}");
    }

    public SentimentPrediction Predict(string text)
    {
        var predictionEngine = _mlContext.Model.CreatePredictionEngine<SentimentData, SentimentPrediction>(_model);
        var prediction = predictionEngine.Predict(new SentimentData { SentimentText = text });

        return prediction;
    }

    public void SaveModel(string modelPath)
    {
        _mlContext.Model.Save(_model, _mlContext.Data.GetSchema(_model.Transform(_mlContext.Data.LoadFromEnumerable(new List<SentimentData>()))), modelPath);
    }

    public void LoadModel(string modelPath)
    {
        _model = _mlContext.Model.Load(modelPath, out var modelSchema);
    }
}
```

### Exemple : Système de recommandation

```csharp
// Modèle de données pour les recommandations
public class MovieRating
{
    [KeyType(count: 262111)]
    public uint userId;

    [KeyType(count: 180)]
    public uint movieId;

    public float rating;
}

public class MovieRatingPrediction
{
    public float Score { get; set; }
}

// Système de recommandation
public class MovieRecommender
{
    private readonly MLContext _mlContext;
    private ITransformer _model;

    public MovieRecommender()
    {
        _mlContext = new MLContext();
    }

    public void TrainCollaborativeFiltering(string ratingsPath)
    {
        // Charger les données de notation
        var data = _mlContext.Data.LoadFromTextFile<MovieRating>(ratingsPath,
            hasHeader: true, separatorChar: '\t');

        // Créer le pipeline
        var pipeline = _mlContext.Transforms.Conversion
            .MapValueToKey(outputColumnName: "userId", inputColumnName: nameof(MovieRating.userId))
            .Append(_mlContext.Transforms.Conversion.MapValueToKey(outputColumnName: "movieId",
                inputColumnName: nameof(MovieRating.movieId)))
            .Append(_mlContext.Recommendation().Trainers.MatrixFactorization(
                labelColumnName: nameof(MovieRating.rating),
                matrixColumnIndexColumnName: "userId",
                matrixRowIndexColumnName: "movieId"));

        // Entraîner le modèle
        _model = pipeline.Fit(data);
    }

    public async Task<List<(int MovieId, float Score)>> GetRecommendations(int userId, int numberOfRecommendations = 10)
    {
        var engine = _mlContext.Model.CreatePredictionEngine<MovieRating, MovieRatingPrediction>(_model);

        var movieIds = GetAllMovieIds(); // Méthode pour obtenir tous les IDs de films
        var recommendations = new List<(int MovieId, float Score)>();

        foreach (var movieId in movieIds)
        {
            var prediction = engine.Predict(new MovieRating { userId = (uint)userId, movieId = (uint)movieId });
            recommendations.Add((movieId, prediction.Score));
        }

        return recommendations
            .OrderByDescending(x => x.Score)
            .Take(numberOfRecommendations)
            .ToList();
    }
}
```

### Exemple : Vision par ordinateur

```csharp
// Classification d'images
public class ImageClassifier
{
    private readonly MLContext _mlContext;
    private ITransformer _model;

    public ImageClassifier()
    {
        _mlContext = new MLContext();
    }

    public void TrainImageClassification(string dataPath, string imagesFolder)
    {
        // Charger les données
        var images = _mlContext.Data.LoadFromTextFile<ImageData>(dataPath,
            hasHeader: true, separatorChar: '\t');

        // Preprocess images
        var pipeline = _mlContext.Transforms.Conversion
            .MapValueToKey("Label", nameof(ImageData.Label))
            .Append(_mlContext.Transforms.LoadImages(
                "ImageReal", imagesFolder, nameof(ImageData.ImagePath)))
            .Append(_mlContext.Transforms.ResizeImages(
                "ImageReal", 224, 224, "ImageReal"))
            .Append(_mlContext.Transforms.ExtractPixels(
                "Pixels", "ImageReal",
                outputAsFloatArray: true,
                scaleImage: 1f / 255f))
            .Append(_mlContext.MulticlassClassification.Trainers.ImageClassification(
                featureColumnName: "Pixels",
                labelColumnName: "Label",
                architecture: ImageClassificationTrainer.Architecture.InceptionV3,
                epoch: 100,
                batchSize: 10,
                learningRate: 0.01f))
            .Append(_mlContext.Transforms.Conversion.MapKeyToValue("PredictedLabel"));

        // Entraîner le modèle
        _model = pipeline.Fit(images);
    }

    public string ClassifyImage(string imagePath)
    {
        var engine = _mlContext.Model.CreatePredictionEngine<ImageData, ImagePrediction>(_model);

        var image = new ImageData
        {
            ImagePath = imagePath,
            Label = "Unknown" // Ne sera pas utilisé pour la prédiction
        };

        var prediction = engine.Predict(image);
        return prediction.PredictedLabel;
    }
}

public class ImageData
{
    [LoadColumn(0)]
    public string ImagePath;

    [LoadColumn(1)]
    public string Label;
}

public class ImagePrediction
{
    public float[] Score;
    public string PredictedLabel;
}
```

### Intégration d'ONNX

```csharp
// Utilisation de modèles ONNX pré-entraînés
public class OnnxImageClassifier
{
    private readonly InferenceSession _session;

    public OnnxImageClassifier(string modelPath)
    {
        _session = new InferenceSession(modelPath);
    }

    public async Task<List<(string Label, float Score)>> Classify(byte[] imageBytes)
    {
        // Préprocessing de l'image
        var tensor = await PreprocessImage(imageBytes);

        // Exécuter l'inférence
        var inputs = new List<NamedOnnxValue>
        {
            NamedOnnxValue.CreateFromTensor("input", tensor)
        };

        using var results = _session.Run(inputs);
        var output = results.First().AsEnumerable<float>().ToList();

        // Post-processing des résultats
        return PostprocessResults(output);
    }

    private async Task<DenseTensor<float>> PreprocessImage(byte[] imageBytes)
    {
        // Conversion et normalisation de l'image
        // Retourner un tensor approprié pour le modèle
    }

    private List<(string Label, float Score)> PostprocessResults(List<float> scores)
    {
        // Convertir les scores en labels avec probabilités
    }
}
```

## Conclusion

L'écosystème C# continue d'évoluer rapidement, offrant aux développeurs de nouvelles possibilités :

1. **Performance et modernité** : Les nouvelles versions apportent des améliorations significatives de performance et des syntaxes plus concises
2. **Source Generators** : Permettent de générer du code optimisé pendant la compilation
3. **WebAssembly** : Ouvre de nouvelles possibilités pour les applications web
4. **IA/ML** : ML.NET rend l'apprentissage automatique accessible aux développeurs .NET

Ces évolutions positionnent C# comme un langage moderne, performant et polyvalent, adapté aux défis technologiques actuels et futurs. Les développeurs peuvent ainsi créer des applications innovantes tout en bénéficiant de la robustesse de l'écosystème .NET.

⏭️ 22. [Ressources et communauté](/22-ressources-et-communaute/README.md)
