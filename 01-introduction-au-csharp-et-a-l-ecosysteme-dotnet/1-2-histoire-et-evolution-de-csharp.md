# 1.2. Histoire et évolution de C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

L'histoire de C# est intimement liée à celle de la plateforme .NET. Depuis sa création au début des années 2000, C# a connu une évolution remarquable, passant d'un langage relativement simple à un langage sophistiqué qui intègre de nombreux paradigmes de programmation modernes.

## 1.2.1. De C# 1.0 à la dernière version

### C# 1.0 (2002)
Première version officielle, lancée avec .NET Framework 1.0. Elle incluait les fonctionnalités de base de la programmation orientée objet :
- Classes et interfaces
- Propriétés et événements
- Délégués et gestion des exceptions
- Bien que complète, cette version était relativement simple comparée aux langages concurrents comme Java et C++

```textmate
// Exemple de code C# 1.0
public class Personne
{
    private string nom;

    public string Nom
    {
        get { return nom; }
        set { nom = value; }
    }

    public void AfficherNom()
    {
        Console.WriteLine("Nom: " + nom);
    }
}
```


### C# 2.0 (2005)
Lancée avec .NET Framework 2.0, cette version a apporté des améliorations majeures :
- Génériques (types paramétrés)
- Méthodes anonymes
- Types nullables
- Itérateurs
- Propriétés statiques
- Covariance et contravariance

```textmate
// Exemple de génériques en C# 2.0
public class ListeGenerique<T>
{
    public void Ajouter(T item) { /* ... */ }
    public T Obtenir(int index) { /* ... */ }
}

// Exemple de type nullable
int? nombreNullable = null;
```


### C# 3.0 (2007)
Publiée avec .NET Framework 3.5, cette version a introduit le concept révolutionnaire de LINQ (Language Integrated Query) :
- Expressions lambda
- Types anonymes
- Initialiseurs d'objets et de collections
- Extension methods
- Query expressions

```textmate
// Exemple LINQ en C# 3.0
var nombres = new[] { 1, 2, 3, 4, 5 };
var nombresPairs = nombres.Where(n => n % 2 == 0).ToList();

// Initialisation d'objet
var personne = new Personne { Nom = "Dupont", Age = 30 };
```


### C# 4.0 (2010)
Sortie avec .NET Framework 4.0, cette version a mis l'accent sur l'interopérabilité :
- Dynamic binding (type `dynamic`)
- Paramètres optionnels et nommés
- Covariance et contravariance génériques
- COM Interop amélioré

```textmate
// Exemple de type dynamic
dynamic obj = GetUnknownObject();
obj.MethodeInconnue("paramètre"); // Résolu à l'exécution

// Paramètres optionnels
public void Methode(string param1, int param2 = 0) { /* ... */ }
```


### C# 5.0 (2012)
Lancée avec .NET Framework 4.5, cette version s'est concentrée sur la programmation asynchrone :
- Mots-clés `async` et `await`
- Améliorations du Caller Info

```textmate
// Exemple de code asynchrone
public async Task<string> TelechargerDonneesAsync(string url)
{
    HttpClient client = new HttpClient();
    string resultat = await client.GetStringAsync(url);
    return resultat;
}
```


### C# 6.0 (2015)
Publiée avec .NET Framework 4.6, cette version a apporté de nombreuses améliorations syntaxiques :
- Initialiseurs de propriétés automatiques
- Membres de fonction expression-bodied
- Opérateur `nameof`
- Filtres d'exception
- Interpolation de chaînes
- Importations statiques

```textmate
// Interpolation de chaînes
string nom = "Jean";
int age = 30;
string message = $"{nom} a {age} ans";

// Propriétés auto-initialisées
public string Nom { get; set; } = "Sans nom";
```


### C# 7.0-7.3 (2017-2018)
Lancée avec .NET Framework 4.7 et les mises à jour ultérieures, cette série a introduit :
- Tuples et déconstruction
- Pattern matching
- Variables locales par référence
- Expression-bodied members étendus
- Variables locales et out variables inline
- Fonctions locales
- Async Main

```textmate
// Tuples
(string Nom, int Age) personne = ("Jean", 30);
Console.WriteLine($"{personne.Nom} a {personne.Age} ans");

// Pattern matching
if (obj is Personne p && p.Age > 18)
{
    // Utiliser p directement
}
```


### C# 8.0 (2019)
Premier langage C# principalement ciblé pour .NET Core 3.0 plutôt que .NET Framework :
- Types de référence Nullable
- Interfaces avec implémentations par défaut
- Pattern matching amélioré
- Indices et plages
- Séquences asynchrones (IAsyncEnumerable)
- Fonctions locales statiques
- Using declarations

```textmate
// Types de référence Nullable
string? nomNullable = null;

// Interfaces avec implémentations par défaut
interface IPersonne
{
    string Nom { get; }
    void Saluer() => Console.WriteLine($"Bonjour, je suis {Nom}");
}

// Indices et plages
var dernierElement = tableau[^1];
var sousTableau = tableau[1..4];
```


### C# 9.0 (2020)
Lancée avec .NET 5 :
- Records (types de données immuables)
- Init-only properties
- Top-level statements (sans Main explicite)
- Pattern matching amélioré
- Expressions fonctionnelles améliorées

```textmate
// Records
public record Personne(string Nom, int Age);

// Top-level statements
Console.WriteLine("Hello World!"); // Sans méthode Main

// Init-only properties
public class Produit
{
    public string Nom { get; init; }
}
```


### C# 10.0 (2021)
Publiée avec .NET 6 :
- Global using directives
- File-scoped namespaces
- Record structs
- Améliorations des lambdas et des déclarations de type
- Interpolated string handlers

```textmate
// Global using
global using System;
global using System.Collections.Generic;

// File-scoped namespace
namespace MonApplication;

// Du code directement ici
```


### C# 11.0 (2022)
Lancée avec .NET 7 :
- Raw string literals
- Required members
- Generic attributes
- List patterns
- UTF-8 string literals

```textmate
// Raw string literals
var json = """
{
    "nom": "Jean",
    "age": 30
}
""";

// Required members
public class Personne
{
    public required string Nom { get; set; }
}
```


### C# 12.0 (2023)
Publiée avec .NET 8 :
- Primary constructors pour les classes
- Collection expressions
- Inline arrays
- Alias de type vers tout type
- Paramètres par défaut pour les expressions lambda

```textmate
// Primary constructors
public class Personne(string nom, int age)
{
    public string Info => $"{nom} a {age} ans";
}

// Collection expressions
int[] nombres = [1, 2, 3, 4, 5];
```


## 1.2.2. Évolution parallèle de .NET

L'évolution de C# est étroitement liée à celle de la plateforme .NET. Voici comment elles ont évolué ensemble :

### .NET Framework (2002-2019)
La plateforme originale de Microsoft pour Windows :

- **.NET Framework 1.0** (2002) : Première version avec C# 1.0
- **.NET Framework 2.0** (2005) : Introducation des génériques avec C# 2.0
- **.NET Framework 3.0** (2006) : Ajout de WPF, WCF, et WF
- **.NET Framework 3.5** (2007) : Support de LINQ avec C# 3.0
- **.NET Framework 4.0** (2010) : Parallel Extensions, MEF avec C# 4.0
- **.NET Framework 4.5** (2012) : Async/await avec C# 5.0
- **.NET Framework 4.6** (2015) : Support de C# 6.0
- **.NET Framework 4.7** (2017) : Support de C# 7.0
- **.NET Framework 4.7.2** (2018) : Support de C# 7.3
- **.NET Framework 4.8** (2019) : Dernière version majeure

.NET Framework était principalement conçu pour Windows et est devenu au fil du temps un élément fondamental du système d'exploitation Windows. Il reste aujourd'hui utilisé par de nombreuses applications d'entreprise existantes, mais n'est plus en développement actif pour de nouvelles fonctionnalités majeures.

### .NET Core / .NET (2016-présent)
La nouvelle génération multiplateforme de .NET :

- **.NET Core 1.0** (2016) : Première version multiplateforme, focus sur Web et Cloud
- **.NET Core 2.0** (2017) : Améliorations majeures de performances
- **.NET Core 2.1** (2018) : Premier release LTS (Long-Term Support)
- **.NET Core 3.0** (2019) : Support pour applications Desktop Windows (WPF, WinForms)
- **.NET Core 3.1** (2019) : Second release LTS
- **.NET 5** (2020) : Unification sous le nom .NET (sans "Core"), C# 9.0
- **.NET 6** (2021) : LTS, C# 10.0, améliorations majeures de performance
- **.NET 7** (2022) : C# 11.0, améliorations des bibliothèques HTTP/JSON
- **.NET 8** (2023) : LTS, C# 12.0, Native AOT compilation

.NET Core, devenu simplement ".NET" à partir de la version 5, représente la vision moderne de Microsoft pour .NET : open source, multiplateforme (Windows, Linux, macOS), modulaire et optimisé pour le cloud.

### Relation C# et .NET

La relation entre C# et .NET est symbiotique :

- **C# évolue pour exploiter les capacités de .NET** : De nombreuses fonctionnalités de C# (comme async/await) sont conçues pour tirer parti des capacités sous-jacentes de la plateforme.

- **.NET évolue pour répondre aux besoins du langage** : La plateforme s'adapte également pour supporter les nouvelles fonctionnalités de C#, comme les types génériques ou les interfaces par défaut.

- **Transition vers l'open source** : C# et .NET ont tous deux effectué une transition vers l'open source, C# via le projet Roslyn (compilateur open source) et .NET via .NET Core.

- **Engagement de la communauté** : Le développement de C# et .NET intègre désormais activement les retours de la communauté via GitHub.

Aujourd'hui, bien que .NET Framework 4.7.2 reste largement utilisé dans les applications d'entreprise existantes, .NET 8 représente la plateforme recommandée pour tous les nouveaux développements, offrant de meilleures performances, des fonctionnalités plus modernes et la portabilité multiplateforme.

⏭️ 1.3. [Pourquoi choisir C# ?](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-3-pourquoi-choisir-csharp.md)
