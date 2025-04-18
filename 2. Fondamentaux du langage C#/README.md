# 2. Fondamentaux du langage C#

![Fondamentaux du langage C#](https://via.placeholder.com/600x150?text=Fondamentaux+du+langage+C%23)

Après avoir créé votre premier programme "Hello World" et compris les bases de la structure d'un projet C#, il est temps d'explorer les fondamentaux du langage C#. Cette section vous fournira les connaissances essentielles qui constituent la base de tout programme C#, quelle que soit sa complexité.

Le langage C# est riche et expressif, conçu pour être à la fois puissant et accessible. Créé par Microsoft sous la direction d'Anders Hejlsberg (également créateur de Turbo Pascal et architecte clé de Delphi), C# a été développé comme un langage orienté objet moderne et sûr du point de vue des types. Depuis sa première version en 2002, C# a continuellement évolué pour intégrer de nouvelles fonctionnalités tout en maintenant sa philosophie initiale.

## Philosophie et caractéristiques de C#

C# possède plusieurs caractéristiques fondamentales qui définissent sa nature :

### Un langage orienté objet

C# est conçu autour du paradigme de la programmation orientée objet (POO). Tout le code C# s'exécute dans le contexte d'un objet, et presque tout en C# est un objet. Cela facilite l'organisation du code en unités logiques et réutilisables.

### Sûreté de type (Type-safe)

C# est un langage fortement typé. Cela signifie que le compilateur vérifie rigoureusement les types de données avant l'exécution, réduisant ainsi les erreurs potentielles. Cette sûreté de type permet de détecter de nombreux problèmes dès la compilation.

### Gestion automatique de la mémoire

Contrairement à des langages comme C ou C++, C# utilise un garbage collector qui libère automatiquement la mémoire occupée par les objets qui ne sont plus utilisés. Cela simplifie considérablement la programmation et élimine une source majeure de bugs comme les fuites de mémoire.

### Expressivité et élégance

Le langage C# est conçu pour être expressif et lisible. Il offre une syntaxe claire et concise qui permet aux développeurs de communiquer efficacement leurs intentions à travers le code.

### Évolution constante

Depuis C# 1.0, le langage a considérablement évolué à travers plusieurs versions majeures, chacune apportant de nouvelles fonctionnalités et améliorations :

- **C# 2.0** (2005) : Génériques, méthodes anonymes, types nullables
- **C# 3.0** (2007) : LINQ, expressions lambda, types anonymes, initialisation d'objets
- **C# 4.0** (2010) : Types dynamiques, paramètres optionnels
- **C# 5.0** (2012) : Programmation asynchrone (async/await)
- **C# 6.0** (2015) : Membres d'expression, filtrages d'exceptions, interpolation de chaînes
- **C# 7.0-7.3** (2017-2018) : Tuples, pattern matching, ref returns, local functions
- **C# 8.0** (2019) : Types référence nullables, switch expressions, interfaces par défaut
- **C# 9.0** (2020) : Records, init-only setters, top-level statements
- **C# 10.0** (2021) : Global usings, file-scoped namespaces, record structs
- **C# 11.0** (2022) : Raw string literals, required members, auto-default structs
- **C# 12.0** (2023) : Primary constructors, collection expressions, inline arrays

Dans ce tutoriel, nous allons nous concentrer sur les fonctionnalités communes à .NET Framework 4.7.2 et .NET 8, tout en mettant en évidence les différences importantes et les nouvelles fonctionnalités disponibles uniquement dans .NET 8.

## Les principes clés à retenir

Avant de plonger dans les détails techniques, gardez à l'esprit ces principes fondamentaux du langage C# :

1. **Tout est un objet** : En C#, presque tout (même les types primitifs) peut être traité comme un objet.

2. **Le code est organisé hiérarchiquement** : C# utilise des espaces de noms, des classes, des méthodes et des blocs pour organiser le code de manière hiérarchique.

3. **Les accolades délimitent les blocs** : Les blocs de code sont délimités par des accolades `{ }`.

4. **Les instructions se terminent par des points-virgules** : Chaque instruction individuelle doit se terminer par un point-virgule `;`.

5. **La casse est importante** : C# fait la distinction entre majuscules et minuscules. Par exemple, `myVariable` et `MyVariable` sont considérés comme deux identifiants différents.

6. **Le nommage suit des conventions** : C# utilise différentes conventions de nommage selon le type d'élément (PascalCase pour les classes, camelCase pour les variables locales, etc.).

7. **Les commentaires sont importants** : C# offre des commentaires sur une ligne (`//`), des commentaires sur plusieurs lignes (`/* */`) et des commentaires de documentation XML (`///`).

## Ce que vous allez apprendre

Dans cette section sur les fondamentaux du langage C#, nous allons couvrir :

- La structure d'un programme C#, y compris les espaces de noms, les classes et le point d'entrée
- Les variables et les types de données fondamentaux
- Les opérateurs et expressions
- Les structures de contrôle comme les conditions et les boucles
- Les tableaux et collections
- Les méthodes et paramètres
- La gestion des exceptions

Ces connaissances constituent le socle sur lequel repose tout programme C#, que vous développiez une simple application console ou une application web complexe.

Prêt à découvrir ces fondamentaux ? Commençons par examiner la structure d'un programme C#.
