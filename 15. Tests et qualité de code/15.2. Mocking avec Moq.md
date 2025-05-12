# 15.2. Mocking avec Moq

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## Introduction au mocking

Lorsque vous écrivez des tests unitaires, vous voulez tester une unité de code de manière isolée. Cependant, la plupart des classes dépendent d'autres classes, ce qui peut compliquer les tests. C'est là qu'interviennent les "mocks" (simulacres).

**Le mocking** est une technique qui permet de remplacer des dépendances réelles par des objets simulés lors des tests. Ces objets simulés (mocks) imitent le comportement des dépendances réelles, mais de manière contrôlée.

**Moq** (prononcé "mock") est l'une des bibliothèques de mocking les plus populaires pour C# et .NET.

### Pourquoi utiliser des mocks ?

- **Isoler le code testé** : Tester uniquement la classe ciblée, sans dépendre du fonctionnement correct de ses dépendances
- **Accélérer les tests** : Éviter les opérations lentes (accès réseau, base de données, système de fichiers)
- **Simuler des conditions difficiles** : Tester des scénarios d'erreur ou des cas limites difficiles à reproduire
- **Vérifier les interactions** : S'assurer que votre classe interagit correctement avec ses dépendances

## 15.2.1. Création de mocks

### Installation de Moq

Commençons par installer le package Moq dans votre projet de test :

```bash
dotnet add package Moq
```

Ou via NuGet Package Manager dans Visual Studio :
1. Clic droit sur le projet de test -> Gérer les packages NuGet
2. Rechercher "Moq"
3. Installer la dernière version stable

### Exemple de base : le contexte

Imaginons que nous avons une application de gestion de clients avec les interfaces et classes suivantes :

```csharp
// L'interface que nous allons simuler (mock)
public interface IClientRepository
{
    Client GetById(int id);
    bool Save(Client client);
    IEnumerable<Client> GetAll();
}

// Notre classe métier qui dépend du repository
public class ClientService
{
    private readonly IClientRepository _repository;

    public ClientService(IClientRepository repository)
    {
        _repository = repository;
    }

    public Client GetClient(int id)
    {
        return _repository.GetById(id);
    }

    public bool UpdateClientName(int id, string newName)
    {
        var client = _repository.GetById(id);
        if (client == null)
            return false;

        client.Name = newName;
        return _repository.Save(client);
    }
}

// La classe Client
public class Client
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

### Création d'un mock simple

Voici comment créer un mock de base avec Moq :

```csharp
using Moq;
using Xunit; // Ou autre framework de test

public class ClientServiceTests
{
    [Fact]
    public void GetClient_ExistingId_ReturnsClient()
    {
        // Arrangement
        var mockRepository = new Mock<IClientRepository>();
        var expectedClient = new Client { Id = 1, Name = "Jean Dupont", Email = "jean@example.com" };

        // Configuration du mock : quand GetById(1) est appelé, retourne expectedClient
        mockRepository.Setup(repo => repo.GetById(1)).Returns(expectedClient);

        var clientService = new ClientService(mockRepository.Object);

        // Action
        var result = clientService.GetClient(1);

        // Assertion
        Assert.Equal(expectedClient, result);
    }
}
```

Dans cet exemple :
1. Nous créons un mock de l'interface `IClientRepository`
2. Nous configurons ce mock pour qu'il retourne un client spécifique quand `GetById(1)` est appelé
3. Nous utilisons `.Object` pour obtenir l'instance mockée à passer au `ClientService`
4. Nous vérifions que le service retourne bien le client attendu

### Configuration des comportements

Moq offre plusieurs façons de configurer le comportement des mocks :

#### Retourner des valeurs spécifiques

```csharp
// Retourner une valeur simple
mockRepository.Setup(repo => repo.GetById(1)).Returns(expectedClient);

// Retourner différentes valeurs selon les paramètres
mockRepository.Setup(repo => repo.GetById(It.IsAny<int>())).Returns((int id) => {
    if (id == 1) return new Client { Id = 1, Name = "Jean" };
    if (id == 2) return new Client { Id = 2, Name = "Marie" };
    return null;
});
```

#### Lancer des exceptions

```csharp
// Simuler une exception
mockRepository.Setup(repo => repo.GetById(-1)).Throws<ArgumentException>();
```

#### Simuler des méthodes void

```csharp
// Pour les méthodes void, utilisez Setup() avec le type générique 'void'
mockRepository.Setup(repo => repo.DeleteClient(It.IsAny<int>())).Verifiable();
```

#### Propriétés mockées

```csharp
// Mocker une propriété
mockRepository.Setup(repo => repo.ConnectionString).Returns("Server=myServerAddress;Database=myDatabase;");

// Propriété avec getter et setter
mockRepository.SetupProperty(repo => repo.IsConnected);
mockRepository.Object.IsConnected = true; // Vous pouvez maintenant modifier cette propriété
```

### Utilisation de It.IsAny, It.Is, It.IsInRange

Moq fournit des helpers pour créer des matchers flexibles :

```csharp
// Correspond à n'importe quel entier
mockRepository.Setup(repo => repo.GetById(It.IsAny<int>())).Returns(new Client());

// Correspond aux entiers positifs
mockRepository.Setup(repo => repo.GetById(It.Is<int>(id => id > 0))).Returns(new Client());

// Correspond à une plage de valeurs
mockRepository.Setup(repo => repo.GetById(It.IsInRange(1, 10, Range.Inclusive))).Returns(new Client());
```

## 15.2.2. Vérification des interactions

Un aspect puissant du mocking est la possibilité de vérifier les interactions entre votre code et ses dépendances.

### Vérifier qu'une méthode a été appelée

```csharp
[Fact]
public void UpdateClientName_ExistingClient_SavesChanges()
{
    // Arrangement
    var mockRepository = new Mock<IClientRepository>();
    var existingClient = new Client { Id = 1, Name = "Ancien Nom" };

    mockRepository.Setup(repo => repo.GetById(1)).Returns(existingClient);
    mockRepository.Setup(repo => repo.Save(It.IsAny<Client>())).Returns(true);

    var clientService = new ClientService(mockRepository.Object);

    // Action
    var result = clientService.UpdateClientName(1, "Nouveau Nom");

    // Assertion
    Assert.True(result);

    // Vérification : s'assurer que Save() a été appelé une fois avec n'importe quel client
    mockRepository.Verify(repo => repo.Save(It.IsAny<Client>()), Times.Once);
}
```

### Vérifier les détails de l'appel

```csharp
[Fact]
public void UpdateClientName_ExistingClient_SavesWithCorrectName()
{
    // Arrangement
    var mockRepository = new Mock<IClientRepository>();
    var existingClient = new Client { Id = 1, Name = "Ancien Nom" };

    mockRepository.Setup(repo => repo.GetById(1)).Returns(existingClient);
    mockRepository.Setup(repo => repo.Save(It.IsAny<Client>())).Returns(true);

    var clientService = new ClientService(mockRepository.Object);

    // Action
    var result = clientService.UpdateClientName(1, "Nouveau Nom");

    // Vérification : s'assurer que Save() a été appelé avec un client
    // dont le nom est "Nouveau Nom"
    mockRepository.Verify(repo => repo.Save(It.Is<Client>(c =>
        c.Id == 1 && c.Name == "Nouveau Nom")), Times.Once);
}
```

### Options de vérification avec Times

La classe `Times` permet de spécifier le nombre d'appels attendus :

```csharp
// Vérifie que la méthode a été appelée exactement une fois
mockRepository.Verify(repo => repo.Save(It.IsAny<Client>()), Times.Once);

// Vérifie que la méthode a été appelée exactement n fois
mockRepository.Verify(repo => repo.Save(It.IsAny<Client>()), Times.Exactly(2));

// Vérifie que la méthode n'a jamais été appelée
mockRepository.Verify(repo => repo.GetById(2), Times.Never);

// Vérifie que la méthode a été appelée au moins une fois
mockRepository.Verify(repo => repo.Save(It.IsAny<Client>()), Times.AtLeastOnce);

// Vérifie que la méthode a été appelée au plus une fois
mockRepository.Verify(repo => repo.Save(It.IsAny<Client>()), Times.AtMostOnce);
```

### Vérifier toutes les interactions configurées

```csharp
// À la fin de votre test, pour vérifier que toutes les méthodes marquées
// comme .Verifiable() ont bien été appelées
mockRepository.Verify();
```

### Vérifier qu'aucune autre méthode n'a été appelée

```csharp
// Vérifier qu'aucune autre méthode que celles configurées n'a été appelée
mockRepository.VerifyNoOtherCalls();
```

## 15.2.3. Mocking avancé

### Callbacks

Les callbacks permettent d'exécuter du code personnalisé quand une méthode mockée est appelée :

```csharp
[Fact]
public void SaveClient_ValidClient_SetsCreationDate()
{
    // Arrangement
    var mockRepository = new Mock<IClientRepository>();
    Client savedClient = null;

    mockRepository.Setup(repo => repo.Save(It.IsAny<Client>()))
        .Callback<Client>(client => savedClient = client)
        .Returns(true);

    var clientService = new ClientService(mockRepository.Object);
    var client = new Client { Id = 0, Name = "Nouveau Client" };

    // Action
    clientService.CreateClient(client);

    // Assertion
    Assert.NotNull(savedClient);
    Assert.Equal(DateTime.Today, savedClient.CreationDate.Date);
}
```

### Séquences de retours

Vous pouvez configurer une méthode pour retourner différentes valeurs à chaque appel :

```csharp
// La première fois retourne true, puis false pour les appels suivants
mockRepository.SetupSequence(repo => repo.Save(It.IsAny<Client>()))
    .Returns(true)
    .Returns(false)
    .Throws<Exception>();
```

### Mocks auto-initialisés (AutoMocking)

Pour les classes avec de nombreuses dépendances, un conteneur auto-mock peut être utile :

```csharp
// Ajoutez le package Moq.AutoMock
// dotnet add package Moq.AutoMock

[Fact]
public void UsingAutoMocker()
{
    // Crée un AutoMockContainer qui génère automatiquement les mocks nécessaires
    var mocker = new AutoMocker();

    // Configurer un mock spécifique
    mocker.GetMock<IClientRepository>()
        .Setup(repo => repo.GetById(1))
        .Returns(new Client { Id = 1, Name = "Client" });

    // Créer l'instance avec toutes les dépendances mockées automatiquement
    var service = mocker.CreateInstance<ClientService>();

    // Utiliser le service
    var client = service.GetClient(1);

    // Vérifier les interactions
    mocker.GetMock<IClientRepository>()
        .Verify(repo => repo.GetById(1), Times.Once);
}
```

### Mocking des propriétés et des événements

```csharp
// Mocker une propriété avec changement de valeur
var mockRepository = new Mock<IClientRepository>();
mockRepository.SetupProperty(repo => repo.Status);
mockRepository.Object.Status = "Connected";
Assert.Equal("Connected", mockRepository.Object.Status);

// Événements
mockRepository.SetupAdd(repo => repo.ClientAdded += It.IsAny<EventHandler>());
mockRepository.SetupRemove(repo => repo.ClientAdded -= It.IsAny<EventHandler>());

// Déclencher l'événement
mockRepository.Raise(repo => repo.ClientAdded += null, new EventArgs());
```

### Mocking de méthodes protégées ou privées

Pour tester des méthodes non publiques, vous devez généralement modifier la conception pour les rendre testables. Une approche consiste à utiliser des classes de base virtuelles et des mocks partiels :

```csharp
public abstract class ClientServiceBase
{
    protected virtual bool ValidateClient(Client client)
    {
        // Logique de validation complexe
        return client != null && !string.IsNullOrEmpty(client.Name);
    }
}

public class ClientService : ClientServiceBase
{
    private readonly IClientRepository _repository;

    public ClientService(IClientRepository repository)
    {
        _repository = repository;
    }

    public bool SaveClient(Client client)
    {
        if (!ValidateClient(client))
            return false;

        return _repository.Save(client);
    }
}

[Fact]
public void SaveClient_InvalidClient_ReturnsFalseWithoutSaving()
{
    // Créer un mock partiel de la classe de base
    var mockService = new Mock<ClientServiceBase> { CallBase = true };

    // Configurer la méthode protégée
    mockService.Protected()
        .Setup<bool>("ValidateClient", ItExpr.IsAny<Client>())
        .Returns(false);

    // Créer un mock du repository
    var mockRepository = new Mock<IClientRepository>();

    // Instancier le service concret avec mocks
    var service = new ClientService(mockRepository.Object);

    // Action
    var result = service.SaveClient(new Client());

    // Assertion
    Assert.False(result);
    mockRepository.Verify(repo => repo.Save(It.IsAny<Client>()), Times.Never);
}
```

Note : Mocker des méthodes privées est généralement considéré comme une mauvaise pratique. Si vous avez besoin de tester une méthode privée directement, c'est souvent le signe d'un problème de conception.

## 15.2.4. Alternatives à Moq (NSubstitute, FakeItEasy)

Bien que Moq soit très populaire, d'autres bibliothèques de mocking existent avec leurs propres avantages.

### NSubstitute

NSubstitute est connu pour sa simplicité et sa syntaxe intuitive.

Installation :
```bash
dotnet add package NSubstitute
```

Exemple de base :
```csharp
using NSubstitute;
using Xunit;

public class ClientServiceTests
{
    [Fact]
    public void GetClient_ExistingId_ReturnsClient()
    {
        // Arrangement
        var repository = Substitute.For<IClientRepository>();
        var expectedClient = new Client { Id = 1, Name = "Jean Dupont" };

        // Configuration
        repository.GetById(1).Returns(expectedClient);

        var service = new ClientService(repository);

        // Action
        var result = service.GetClient(1);

        // Assertion
        Assert.Equal(expectedClient, result);

        // Vérification
        repository.Received(1).GetById(1);
    }
}
```

Caractéristiques de NSubstitute :
- Syntaxe plus concise et naturelle
- Pas besoin d'appeler `.Object` comme avec Moq
- `Received()` pour vérifier les appels
- `Returns()` directement sur l'appel de méthode mockée

Configuration des comportements :
```csharp
// Retourne différentes valeurs selon les paramètres
repository.GetById(Arg.Any<int>()).Returns(args => {
    var id = (int)args[0];
    if (id == 1) return new Client { Id = 1, Name = "Jean" };
    return null;
});

// Lance une exception
repository.GetById(-1).Throws<ArgumentException>();

// Pour des méthodes void
repository.When(x => x.DeleteClient(5))
    .Do(info => { throw new InvalidOperationException(); });
```

Vérification des appels :
```csharp
// Vérifie qu'une méthode a été appelée exactement n fois
repository.Received(1).GetById(1);

// Vérifie qu'une méthode a été appelée au moins n fois
repository.Received(Quantity.AtLeast(1)).Save(Arg.Any<Client>());

// Vérifie qu'une méthode n'a pas été appelée
repository.DidNotReceive().DeleteClient(Arg.Any<int>());
```

### FakeItEasy

FakeItEasy met l'accent sur la simplicité et la cohérence.

Installation :
```bash
dotnet add package FakeItEasy
```

Exemple de base :
```csharp
using FakeItEasy;
using Xunit;

public class ClientServiceTests
{
    [Fact]
    public void GetClient_ExistingId_ReturnsClient()
    {
        // Arrangement
        var repository = A.Fake<IClientRepository>();
        var expectedClient = new Client { Id = 1, Name = "Jean Dupont" };

        // Configuration
        A.CallTo(() => repository.GetById(1)).Returns(expectedClient);

        var service = new ClientService(repository);

        // Action
        var result = service.GetClient(1);

        // Assertion
        Assert.Equal(expectedClient, result);

        // Vérification
        A.CallTo(() => repository.GetById(1)).MustHaveHappenedOnceExactly();
    }
}
```

Caractéristiques de FakeItEasy :
- Utilisation de l'API fluide avec `A.Fake<T>()` pour créer les mocks
- `A.CallTo()` pour configurer les comportements
- `MustHaveHappened...` pour vérifier les appels
- Syntaxe cohérente entre les configurations et les vérifications

Configuration des comportements :
```csharp
// Retourne différentes valeurs selon les paramètres
A.CallTo(() => repository.GetById(A<int>.That.IsGreaterThan(0)))
    .Returns(new Client { Name = "Client valide" });

// Lance une exception
A.CallTo(() => repository.GetById(-1)).Throws<ArgumentException>();

// Pour les méthodes void
A.CallTo(() => repository.DeleteClient(A<int>._))
    .Invokes((int id) => { /* code personnalisé */ });
```

Vérification des appels :
```csharp
// Vérifie qu'une méthode a été appelée exactement une fois
A.CallTo(() => repository.GetById(1)).MustHaveHappenedOnceExactly();

// Vérifie qu'une méthode a été appelée au moins une fois
A.CallTo(() => repository.Save(A<Client>._)).MustHaveHappened();

// Vérifie qu'une méthode a été appelée un nombre spécifique de fois
A.CallTo(() => repository.Save(A<Client>._)).MustHaveHappened(2, Times.Exactly);

// Vérifie qu'une méthode n'a jamais été appelée
A.CallTo(() => repository.DeleteClient(A<int>._)).MustNotHaveHappened();
```

### Comparaison des bibliothèques

| Fonctionnalité | Moq | NSubstitute | FakeItEasy |
|----------------|-----|-------------|------------|
| Création d'un mock | `new Mock<T>()` | `Substitute.For<T>()` | `A.Fake<T>()` |
| Configuration | `mock.Setup(...).Returns(...)` | `mock.Method(...).Returns(...)` | `A.CallTo(() => mock.Method(...)).Returns(...)` |
| Accès à l'instance mockée | `mock.Object` | Directement | Directement |
| Vérification des appels | `mock.Verify(...)` | `mock.Received()...` | `A.CallTo(() => ...).MustHaveHappened()` |
| Syntaxe | Verbeux | Concis | Verbeux mais cohérent |
| Courbe d'apprentissage | Moyenne | Faible | Faible |
| Matchers de paramètres | `It.IsAny<T>()` | `Arg.Any<T>()` | `A<T>._` |

### Quelle bibliothèque choisir ?

- **Moq** : Excellent choix si vous voulez une bibliothèque mature avec beaucoup de fonctionnalités et une documentation abondante.
- **NSubstitute** : Idéal si vous préférez une syntaxe simple et intuitive, particulièrement adapté aux débutants.
- **FakeItEasy** : Bon choix si vous appréciez une API cohérente et un modèle mental simplifié.

Au final, toutes ces bibliothèques sont capables de répondre à la plupart des besoins de mocking. Le choix dépend souvent des préférences de l'équipe ou des standards du projet.

## Bonnes pratiques de mocking

Pour conclure, voici quelques conseils pour bien utiliser les mocks dans vos tests :

1. **Mockez les interfaces, pas les classes concrètes** : Concevez votre code avec des interfaces pour faciliter les tests.

2. **Ne mockez que les dépendances directes** : Évitez de mocker des classes que vous testez.

3. **Limitez les vérifications d'interactions** : Concentrez-vous sur les résultats plutôt que sur les détails d'implémentation.

4. **Ne mockez pas les types que vous ne possédez pas** : Créez des wrappers autour des API externes.

5. **Utilisez des noms clairs pour vos tests** : Suivez un modèle comme `[Méthode]_[Scénario]_[RésultatAttendu]`.

6. **Organisez vos tests** : Structurez-les selon le pattern Arrange-Act-Assert pour une meilleure lisibilité.

## Ressources complémentaires

- [Documentation officielle de Moq](https://github.com/moq/moq4)
- [Documentation de NSubstitute](https://nsubstitute.github.io/)
- [Documentation de FakeItEasy](https://fakeiteasy.github.io/)
- [Livre "The Art of Unit Testing" par Roy Osherove](https://www.amazon.fr/Art-Unit-Testing-Roy-Osherove/dp/1617290890)

⏭️
