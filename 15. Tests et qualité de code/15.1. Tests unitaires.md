# 15.1. Tests unitaires en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les tests unitaires sont une pratique fondamentale dans le développement logiciel moderne. Ils consistent à tester individuellement les plus petites parties testables d'une application (généralement des méthodes) pour vérifier qu'elles fonctionnent correctement. En C#, plusieurs frameworks de test sont disponibles pour faciliter cette tâche.

## Pourquoi les tests unitaires sont importants

Avant de plonger dans les détails techniques, comprenons pourquoi les tests unitaires sont essentiels :

- **Détection précoce des bugs** : Identifiez les problèmes avant qu'ils n'atteignent la production
- **Refactoring en toute confiance** : Modifiez votre code en sachant que vous détecterez rapidement toute régression
- **Documentation vivante** : Les tests servent de documentation sur le comportement attendu du code
- **Design amélioré** : Écrire des tests incite naturellement à créer un code plus modulaire et moins couplé

## 15.1.1. MSTest

MSTest est le framework de test natif de Microsoft, intégré directement dans Visual Studio.

### Installation

Pour les projets .NET 5+ ou .NET Core :

```bash
dotnet add package MSTest.TestAdapter
dotnet add package MSTest.TestFramework
```

Pour les projets .NET Framework, MSTest est déjà intégré à Visual Studio.

### Structure d'un projet de test MSTest

Pour créer un projet de test MSTest :

1. Dans Visual Studio : **Fichier → Nouveau → Projet → Projet de test MSTest**
2. Avec dotnet CLI : `dotnet new mstest -n MonProjetTests`

### Exemple de base

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace MaCalculatrice.Tests
{
    [TestClass]
    public class CalculatriceTests
    {
        [TestMethod]
        public void Addition_DeuxNombresPositifs_RetourneSommeCorrecte()
        {
            // Arrangement
            var calculatrice = new Calculatrice();

            // Action
            int resultat = calculatrice.Addition(2, 3);

            // Assertion
            Assert.AreEqual(5, resultat);
        }

        [TestMethod]
        public void Division_ParZero_LanceException()
        {
            // Arrangement
            var calculatrice = new Calculatrice();

            // Action & Assertion
            Assert.ThrowsException<DivideByZeroException>(() => calculatrice.Division(10, 0));
        }
    }
}
```

### Attributs importants de MSTest

- `[TestClass]` : Marque une classe contenant des méthodes de test
- `[TestMethod]` : Marque une méthode comme étant un test unitaire
- `[TestInitialize]` : Méthode exécutée avant chaque test
- `[TestCleanup]` : Méthode exécutée après chaque test
- `[ClassInitialize]` : Méthode statique exécutée une seule fois avant tous les tests de la classe
- `[ClassCleanup]` : Méthode statique exécutée une seule fois après tous les tests de la classe
- `[Ignore]` : Permet d'ignorer temporairement un test

### Exemple avec configuration initiale

```csharp
[TestClass]
public class ClientServiceTests
{
    private ClientService _service;
    private Mock<IClientRepository> _mockRepository;

    [TestInitialize]
    public void Initialize()
    {
        // Préparation commune à tous les tests
        _mockRepository = new Mock<IClientRepository>();
        _service = new ClientService(_mockRepository.Object);
    }

    [TestMethod]
    public void ObtenirClient_ClientExiste_RetourneClient()
    {
        // Arrangement spécifique à ce test
        var clientAttendu = new Client { Id = 1, Nom = "Dupont" };
        _mockRepository.Setup(r => r.ObtenirParId(1)).Returns(clientAttendu);

        // Action
        var resultat = _service.ObtenirClient(1);

        // Assertion
        Assert.AreEqual(clientAttendu.Nom, resultat.Nom);
    }

    [TestCleanup]
    public void Cleanup()
    {
        // Nettoyage après chaque test
        _service = null;
    }
}
```

## 15.1.2. NUnit

NUnit est l'un des frameworks de test les plus populaires pour .NET, inspiré par JUnit.

### Installation

```bash
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package Microsoft.NET.Test.Sdk
```

### Structure d'un projet NUnit

Pour créer un projet de test NUnit :

1. Dans Visual Studio : **Fichier → Nouveau → Projet → Projet de test NUnit**
2. Avec dotnet CLI : `dotnet new nunit -n MonProjetTests`

### Exemple de base avec NUnit

```csharp
using NUnit.Framework;

namespace MaCalculatrice.Tests
{
    [TestFixture]
    public class CalculatriceTests
    {
        [Test]
        public void Addition_DeuxNombresPositifs_RetourneSommeCorrecte()
        {
            // Arrangement
            var calculatrice = new Calculatrice();

            // Action
            int resultat = calculatrice.Addition(2, 3);

            // Assertion
            Assert.That(resultat, Is.EqualTo(5));
        }

        [Test]
        public void Division_ParZero_LanceException()
        {
            // Arrangement
            var calculatrice = new Calculatrice();

            // Action & Assertion
            Assert.Throws<DivideByZeroException>(() => calculatrice.Division(10, 0));
        }
    }
}
```

### Attributs importants de NUnit

- `[TestFixture]` : Marque une classe contenant des tests
- `[Test]` : Marque une méthode comme étant un test unitaire
- `[SetUp]` : Méthode exécutée avant chaque test
- `[TearDown]` : Méthode exécutée après chaque test
- `[OneTimeSetUp]` : Méthode exécutée une seule fois avant tous les tests
- `[OneTimeTearDown]` : Méthode exécutée une seule fois après tous les tests
- `[Ignore("Raison")]` : Pour ignorer temporairement un test
- `[Category("Catégorie")]` : Pour organiser les tests par catégories

### Particularités de NUnit

NUnit propose une syntaxe d'assertion fluide et lisible :

```csharp
// Assertions classiques
Assert.That(resultat, Is.EqualTo(5));
Assert.That(liste, Has.Count.EqualTo(3));
Assert.That(chaine, Does.Contain("test"));
Assert.That(exception, Is.Not.Null);

// Plusieurs assertions en une seule
Assert.Multiple(() => {
    Assert.That(resultat.Nom, Is.EqualTo("Dupont"));
    Assert.That(resultat.Age, Is.GreaterThan(18));
    Assert.That(resultat.Email, Is.Not.Empty);
});
```

## 15.1.3. xUnit

xUnit est un framework de test plus récent et minimaliste, créé par l'un des auteurs de NUnit. Il est utilisé par l'équipe .NET Core pour tester le framework lui-même.

### Installation

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package Microsoft.NET.Test.Sdk
```

### Structure d'un projet xUnit

Pour créer un projet de test xUnit :

1. Dans Visual Studio : **Fichier → Nouveau → Projet → Projet de test xUnit**
2. Avec dotnet CLI : `dotnet new xunit -n MonProjetTests`

### Exemple de base avec xUnit

```csharp
using Xunit;

namespace MaCalculatrice.Tests
{
    public class CalculatriceTests
    {
        [Fact]
        public void Addition_DeuxNombresPositifs_RetourneSommeCorrecte()
        {
            // Arrangement
            var calculatrice = new Calculatrice();

            // Action
            int resultat = calculatrice.Addition(2, 3);

            // Assertion
            Assert.Equal(5, resultat);
        }

        [Fact]
        public void Division_ParZero_LanceException()
        {
            // Arrangement
            var calculatrice = new Calculatrice();

            // Action & Assertion
            Assert.Throws<DivideByZeroException>(() => calculatrice.Division(10, 0));
        }
    }
}
```

### Particularités de xUnit

xUnit se distingue par sa philosophie minimaliste :

- Pas d'attribut de classe spécial (les classes publiques sont automatiquement des classes de test)
- `[Fact]` au lieu de `[Test]` ou `[TestMethod]`
- Utilise le constructeur/destructeur pour les initialisations/nettoyages par test (au lieu de `[SetUp]` et `[TearDown]`)
- Utilise `IClassFixture<T>` pour l'initialisation au niveau de la classe
- `[Theory]` et `[InlineData]` pour les tests paramétrés (voir section 15.1.6)

### Gestion de l'initialisation dans xUnit

```csharp
public class ClientServiceTests : IDisposable
{
    private readonly ClientService _service;
    private readonly Mock<IClientRepository> _mockRepository;

    // Exécuté avant chaque test (équivalent de [SetUp] ou [TestInitialize])
    public ClientServiceTests()
    {
        _mockRepository = new Mock<IClientRepository>();
        _service = new ClientService(_mockRepository.Object);
    }

    [Fact]
    public void ObtenirClient_ClientExiste_RetourneClient()
    {
        // Arrangement
        var clientAttendu = new Client { Id = 1, Nom = "Dupont" };
        _mockRepository.Setup(r => r.ObtenirParId(1)).Returns(clientAttendu);

        // Action
        var resultat = _service.ObtenirClient(1);

        // Assertion
        Assert.Equal(clientAttendu.Nom, resultat.Nom);
    }

    // Exécuté après chaque test (équivalent de [TearDown] ou [TestCleanup])
    public void Dispose()
    {
        // Nettoyage
    }
}
```

### Initialisation partagée entre tests dans xUnit

Pour partager une même initialisation entre plusieurs tests :

```csharp
// Classe de fixture partagée
public class DatabaseFixture : IDisposable
{
    public SqlConnection Connection { get; private set; }

    public DatabaseFixture()
    {
        Connection = new SqlConnection("connection_string");
        Connection.Open();
        // Initialisation de la base de données de test
    }

    public void Dispose()
    {
        Connection.Close();
    }
}

// Classe de test utilisant la fixture
public class ClientRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public ClientRepositoryTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public void ObtenirClients_RetourneTousLesClients()
    {
        // Utilise _fixture.Connection
        var repository = new ClientRepository(_fixture.Connection);
        var clients = repository.ObtenirTous();
        Assert.NotEmpty(clients);
    }
}
```

## 15.1.4. Assertions et vérifications

Les assertions sont au cœur des tests unitaires. Elles permettent de vérifier que le comportement réel correspond au comportement attendu.

### Assertions communes à tous les frameworks

Bien que la syntaxe varie, tous les frameworks offrent des assertions similaires :

| Type de vérification | MSTest | NUnit | xUnit |
|----------------------|--------|-------|-------|
| Égalité | `Assert.AreEqual(expected, actual)` | `Assert.That(actual, Is.EqualTo(expected))` | `Assert.Equal(expected, actual)` |
| Inégalité | `Assert.AreNotEqual(notExpected, actual)` | `Assert.That(actual, Is.Not.EqualTo(notExpected))` | `Assert.NotEqual(notExpected, actual)` |
| Référence nulle | `Assert.IsNull(obj)` | `Assert.That(obj, Is.Null)` | `Assert.Null(obj)` |
| Référence non nulle | `Assert.IsNotNull(obj)` | `Assert.That(obj, Is.Not.Null)` | `Assert.NotNull(obj)` |
| Valeur vraie | `Assert.IsTrue(condition)` | `Assert.That(condition, Is.True)` | `Assert.True(condition)` |
| Valeur fausse | `Assert.IsFalse(condition)` | `Assert.That(condition, Is.False)` | `Assert.False(condition)` |
| Collection contient | `CollectionAssert.Contains(collection, item)` | `Assert.That(collection, Does.Contain(item))` | `Assert.Contains(item, collection)` |
| Vérification d'exception | `Assert.ThrowsException<T>(action)` | `Assert.Throws<T>(() => action)` | `Assert.Throws<T>(() => action)` |

### Assertions spécifiques à chaque framework

Chaque framework possède ses propres assertions spécialisées :

#### MSTest

```csharp
// Collections
CollectionAssert.AreEqual(expectedCollection, actualCollection);
CollectionAssert.AllItemsAreNotNull(collection);
CollectionAssert.AllItemsAreUnique(collection);

// Chaînes de caractères
StringAssert.Contains(string1, string2);
StringAssert.StartsWith(string1, string2);
StringAssert.Matches(string1, regularExpression);

// Avec message personnalisé
Assert.AreEqual(5, result, "La somme devrait être égale à 5");
```

#### NUnit

```csharp
// Assertions fluides
Assert.That(actual, Is.EqualTo(expected).Within(0.001)); // Comparaison de nombres à virgule flottante
Assert.That(collection, Has.Member(item).And.No.Member(otherItem));
Assert.That(dateTime, Is.EqualTo(DateTime.Now).Within(TimeSpan.FromSeconds(1)));

// Contraintes sur collections
Assert.That(collection, Is.All.GreaterThan(0));
Assert.That(dictionary, Contains.Key("clé"));
Assert.That(collection, Is.Ordered);

// Assertions multiples
Assert.Multiple(() => {
    Assert.That(result.Success, Is.True);
    Assert.That(result.Message, Is.Not.Empty);
    Assert.That(result.Data, Is.Not.Null);
});
```

#### xUnit

```csharp
// Collections
Assert.Collection(list,
    item => Assert.Equal("premier", item),
    item => Assert.Equal("deuxième", item)
);
Assert.All(collection, item => Assert.True(item > 0));

// Types
Assert.IsType<string>(result);
Assert.IsAssignableFrom<IEnumerable>(result);

// Assertions avec message personnalisé
Assert.True(condition, "Message d'erreur personnalisé");
```

### Bonnes pratiques pour les assertions

1. **Une assertion principale par test** : Bien qu'il soit possible de faire plusieurs assertions, concentrez-vous sur une seule vérification principale par test.

2. **Nommage explicite** : Nommez vos tests pour qu'ils décrivent clairement le scénario testé et le résultat attendu, en suivant un modèle comme `[Méthode]_[Scénario]_[RésultatAttendu]`.

3. **Pattern AAA** : Structurez vos tests selon le pattern Arrange-Act-Assert (Préparation-Action-Vérification) pour une meilleure lisibilité.

4. **Messages d'erreur utiles** : Fournissez des messages personnalisés pour faciliter le débogage en cas d'échec.

## 15.1.5. Code coverage (Couverture de code)

La couverture de code est une métrique qui indique quel pourcentage de votre code est testé. Visual Studio Enterprise et les outils d'intégration continue proposent cette fonctionnalité.

### Types de couverture

- **Line coverage** : Pourcentage de lignes de code exécutées
- **Branch coverage** : Pourcentage de branches conditionnelles exécutées (if/else, switch)
- **Method coverage** : Pourcentage de méthodes appelées
- **Class coverage** : Pourcentage de classes utilisées

### Configuration dans Visual Studio

1. **Visual Studio Enterprise** : Ouvrez le menu **Test → Analyser la couverture de code pour tous les tests**
2. **Visual Studio Community/Professional** : Utilisez des extensions comme "Fine Code Coverage"

### Configuration avec dotnet CLI et outils externes

Utilisez des outils comme Coverlet pour générer des rapports de couverture :

```bash
dotnet add package coverlet.collector
dotnet add package coverlet.msbuild
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura
```

Pour visualiser les résultats, utilisez ReportGenerator :

```bash
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"./coverage.cobertura.xml" -targetdir:"./coveragereport" -reporttypes:Html
```

### Interprétation de la couverture

Une couverture de 100% ne garantit pas l'absence de bugs, mais une faible couverture signale souvent des parties risquées du code.

Bonnes pratiques :
- Visez au moins 70-80% de couverture pour le code métier critique
- Priorisez les sections complexes ou sujettes aux erreurs
- Évitez d'optimiser uniquement pour la couverture sans vous soucier de la qualité des tests

## 15.1.6. Tests paramétrés

Les tests paramétrés permettent d'exécuter la même logique de test avec différentes données d'entrée, réduisant ainsi la duplication de code.

### Tests paramétrés avec MSTest

```csharp
[TestClass]
public class CalculatriceTests
{
    [DataTestMethod]
    [DataRow(1, 2, 3)]
    [DataRow(0, 0, 0)]
    [DataRow(-1, -2, -3)]
    [DataRow(int.MaxValue, 1, int.MinValue)] // Provoquera un overflow
    public void Addition_AvecDiversesValeurs_RetourneSommeCorrecte(int a, int b, int resultatAttendu)
    {
        // Arrangement
        var calculatrice = new Calculatrice();

        // Action
        int resultat = calculatrice.Addition(a, b);

        // Assertion
        Assert.AreEqual(resultatAttendu, resultat);
    }
}
```

Pour des données plus complexes, utilisez `DynamicData` :

```csharp
[TestClass]
public class UtilisateurTests
{
    private static IEnumerable<object[]> UtilisateursValides()
    {
        yield return new object[] { new Utilisateur { Nom = "Dupont", Age = 25 }, true };
        yield return new object[] { new Utilisateur { Nom = "Martin", Age = 18 }, true };
        yield return new object[] { new Utilisateur { Nom = "", Age = 15 }, false };
    }

    [TestMethod]
    [DynamicData(nameof(UtilisateursValides), DynamicDataSourceType.Method)]
    public void EstValide_DifferentsUtilisateurs_RetourneResultatAttendu(Utilisateur utilisateur, bool resultatAttendu)
    {
        // Action
        bool resultat = utilisateur.EstValide();

        // Assertion
        Assert.AreEqual(resultatAttendu, resultat);
    }
}
```

### Tests paramétrés avec NUnit

```csharp
[TestFixture]
public class CalculatriceTests
{
    [Test]
    [TestCase(1, 2, 3)]
    [TestCase(0, 0, 0)]
    [TestCase(-1, -2, -3)]
    [TestCase(int.MaxValue, 1, int.MinValue)]
    public void Addition_AvecDiversesValeurs_RetourneSommeCorrecte(int a, int b, int resultatAttendu)
    {
        // Arrangement
        var calculatrice = new Calculatrice();

        // Action
        int resultat = calculatrice.Addition(a, b);

        // Assertion
        Assert.That(resultat, Is.EqualTo(resultatAttendu));
    }
}
```

Pour des données complexes, utilisez `TestCaseSource` :

```csharp
[TestFixture]
public class UtilisateurTests
{
    private static IEnumerable<TestCaseData> UtilisateursValides()
    {
        yield return new TestCaseData(new Utilisateur { Nom = "Dupont", Age = 25 }).Returns(true);
        yield return new TestCaseData(new Utilisateur { Nom = "Martin", Age = 18 }).Returns(true);
        yield return new TestCaseData(new Utilisateur { Nom = "", Age = 15 }).Returns(false);
    }

    [Test]
    [TestCaseSource(nameof(UtilisateursValides))]
    public bool EstValide_DifferentsUtilisateurs_RetourneResultatAttendu(Utilisateur utilisateur)
    {
        // Action & Return (NUnit permet de retourner directement la valeur à tester)
        return utilisateur.EstValide();
    }
}
```

### Tests paramétrés avec xUnit

xUnit utilise `[Theory]` et `[InlineData]` pour les tests paramétrés :

```csharp
public class CalculatriceTests
{
    [Theory]
    [InlineData(1, 2, 3)]
    [InlineData(0, 0, 0)]
    [InlineData(-1, -2, -3)]
    [InlineData(int.MaxValue, 1, int.MinValue)]
    public void Addition_AvecDiversesValeurs_RetourneSommeCorrecte(int a, int b, int resultatAttendu)
    {
        // Arrangement
        var calculatrice = new Calculatrice();

        // Action
        int resultat = calculatrice.Addition(a, b);

        // Assertion
        Assert.Equal(resultatAttendu, resultat);
    }
}
```

Pour des données plus complexes, utilisez `[MemberData]` ou `[ClassData]` :

```csharp
public class UtilisateurTests
{
    public static IEnumerable<object[]> UtilisateursValides()
    {
        yield return new object[] { new Utilisateur { Nom = "Dupont", Age = 25 }, true };
        yield return new object[] { new Utilisateur { Nom = "Martin", Age = 18 }, true };
        yield return new object[] { new Utilisateur { Nom = "", Age = 15 }, false };
    }

    [Theory]
    [MemberData(nameof(UtilisateursValides))]
    public void EstValide_DifferentsUtilisateurs_RetourneResultatAttendu(Utilisateur utilisateur, bool resultatAttendu)
    {
        // Action
        bool resultat = utilisateur.EstValide();

        // Assertion
        Assert.Equal(resultatAttendu, resultat);
    }
}
```

Avec `[ClassData]` pour une classe de données dédiée :

```csharp
public class UtilisateursValidesData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { new Utilisateur { Nom = "Dupont", Age = 25 }, true };
        yield return new object[] { new Utilisateur { Nom = "Martin", Age = 18 }, true };
        yield return new object[] { new Utilisateur { Nom = "", Age = 15 }, false };
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

public class UtilisateurTests
{
    [Theory]
    [ClassData(typeof(UtilisateursValidesData))]
    public void EstValide_DifferentsUtilisateurs_RetourneResultatAttendu(Utilisateur utilisateur, bool resultatAttendu)
    {
        // Action
        bool resultat = utilisateur.EstValide();

        // Assertion
        Assert.Equal(resultatAttendu, resultat);
    }
}
```

## Exécution des tests

### Dans Visual Studio

- **Explorateur de tests** : Fenêtre dédiée pour exécuter et gérer les tests
- **Raccourcis** : Ctrl+R, T (exécuter tous les tests), Ctrl+R, Ctrl+T (exécuter le test courant)

### Avec dotnet CLI

```bash
# Exécuter tous les tests
dotnet test

# Exécuter un projet de test spécifique
dotnet test ./MesTests/MesTests.csproj

# Filtrer les tests à exécuter
dotnet test --filter "FullyQualifiedName~CalculatriceTests"
dotnet test --filter "Category=IntegrationTest"

# Exécuter en parallèle
dotnet test --parallel
```

## Conclusion

Les tests unitaires sont un investissement qui améliore la qualité du code et facilite la maintenance à long terme. Chaque framework offre des avantages spécifiques :

- **MSTest** : Bien intégré à Visual Studio, documentation officielle Microsoft
- **NUnit** : Syntaxe flexible, assertions fluides, grande communauté
- **xUnit** : Moderne, minimaliste, utilisé par l'équipe .NET Core

Le choix du framework dépend souvent des préférences de l'équipe ou des standards du projet. Quelle que soit votre préférence, l'important est d'adopter une culture de test systématique dans votre développement.

## Ressources complémentaires

- [Documentation officielle MSTest](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest)
- [Documentation NUnit](https://docs.nunit.org/)
- [Documentation xUnit](https://xunit.net/docs/getting-started/netcore/cmdline)
- [Livre "The Art of Unit Testing" par Roy Osherove](https://www.amazon.fr/Art-Unit-Testing-Roy-Osherove/dp/1617290890)

⏭️
