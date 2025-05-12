# Tutoriel Complet sur C# et l'Écosystème .NET
## Table des matières

## 1. [Introduction au C# et à l'écosystème .NET](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/README.md)
- 1.1. [Qu'est-ce que C# ?](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-1-quest-ce-que-csharp.md)
- 1.2. [Histoire et évolution de C#](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-2-histoire-et-evolution-de-csharp.md)
    - 1.2.1. De C# 1.0 à la dernière version
    - 1.2.2. Évolution parallèle de .NET

- 1.3. [Pourquoi choisir C# ?](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-3-pourquoi-choisir-csharp.md)
    - 1.3.1. Avantages et cas d'usage
    - 1.3.2. Comparaison avec d'autres langages (Java, JavaScript, Python)

- 1.4. [Installation de l'environnement de développement](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-4-installation-de-lenvironnement-de-developpement.md)
    - 1.4.1. Visual Studio (Community, Professional, Enterprise)
    - 1.4.2. Visual Studio Code avec extensions C#
    - 1.4.3. Rider de JetBrains
    - 1.4.4. Configuration initiale et premiers pas

- 1.5. [Vue d'ensemble de l'écosystème .NET](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-5-vue-densemble-de-lecosysteme-dotnet.md)
    - 1.5.1. .NET Framework (héritage)
    - 1.5.2. .NET Core et .NET 5+
    - 1.5.3. Différences clés et compatibilité
    - 1.5.4. NuGet et gestion des packages

- 1.6. [Votre premier programme "Hello World"](/01-introduction-au-csharp-et-a-l-ecosysteme-dotnet/1-6-votre-premier-programme-hello-world.md)
    - 1.6.1. Structure d'un projet C#
    - 1.6.2. Comprendre la syntaxe de base
    - 1.6.3. Compilation et exécution

## 2. [Fondamentaux du langage C#](/02-fondamentaux-du-langage-csharp/README.md)
- 2.1. [Structure d'un programme C#](/02-fondamentaux-du-langage-csharp/2-1-structure-dun-programme-csharp.md)
    - 2.1.1. Espaces de noms (namespaces)
    - 2.1.2. Classes et fichiers
    - 2.1.3. Point d'entrée (méthode Main)
    - 2.1.4. Programmes top-level (C# 9+)

- 2.2. [Types de données](/02-fondamentaux-du-langage-csharp/2-2-types-de-donnees.md)
    - 2.2.1. Types primitifs (int, float, bool, etc.)
    - 2.2.2. Types référence vs types valeur
    - 2.2.3. Chaînes de caractères et leurs méthodes
    - 2.2.4. Boxing et unboxing
    - 2.2.5. Types nullables et opérateurs associés

- 2.3. [Variables et constantes](/02-fondamentaux-du-langage-csharp/2-3-variables-et-constantes.md)
    - 2.3.1. Déclaration et initialisation
    - 2.3.2. Portée (scope) des variables
    - 2.3.3. Conversion de types (cast implicite/explicite)
    - 2.3.4. var et inférence de type
    - 2.3.5. readonly et const

- 2.4. [Opérateurs](/02-fondamentaux-du-langage-csharp/2-4-operateurs.md)
    - 2.4.1. Arithmétiques et d'affectation
    - 2.4.2. Logiques et bit à bit
    - 2.4.3. Comparaison et égalité
    - 2.4.4. Opérateur null-conditionnel (?.) et null-coalescent (??)
    - 2.4.5. Expression conditionnelle ternaire

- 2.5. [Structures de contrôle](/02-fondamentaux-du-langage-csharp/2-5-structures-de-controle.md)
    - 2.5.1. Conditionnelles (if, else, switch)
    - 2.5.2. Boucles (for, while, do-while, foreach)
    - 2.5.3. Instructions de saut (break, continue, return)
    - 2.5.4. Pattern matching (à partir de C# 7.0)
    - 2.5.5. Switch expressions (C# 8+)

- 2.6. [Enregistrement des commentaires et documentation](/02-fondamentaux-du-langage-csharp/2-6-enregistrement-des-commentaires-et-documentation.md)
    - 2.6.1. Commentaires de ligne et de bloc
    - 2.6.2. Commentaires XML pour la documentation
    - 2.6.3. Génération de documentation

## 3. [Tableaux et collections](/03-tableaux-et-collections/README.md)
- 3.1. [Tableaux](/03-tableaux-et-collections/3-1-tableaux.md)
    - 3.1.1. Tableaux unidimensionnels
    - 3.1.2. Tableaux multidimensionnels
    - 3.1.3. Tableaux en dents de scie (jagged arrays)
    - 3.1.4. Méthodes utiles pour manipuler des tableaux

- 3.2. [Collections standard](/03-tableaux-et-collections/3-2-collections-standard.md)
    - 3.2.1. List
    - 3.2.2. Dictionary<TKey, TValue>
    - 3.2.3. HashSet
    - 3.2.4. Queue et Stack
    - 3.2.5. LinkedList
    - 3.2.6. SortedList<TKey, TValue> et SortedDictionary<TKey, TValue>

- 3.3. [Collections spécialisées](/03-tableaux-et-collections/3-3-collections-specialisees.md)
    - 3.3.1. Collections concurrentes (ConcurrentDictionary, etc.)
    - 3.3.2. Collections immuables (ImmutableList, etc.)
    - 3.3.3. Collections observables (ObservableCollection)
    - 3.3.4. ReadOnlyCollection et autres wrappers en lecture seule

## 4. [Méthodes et fonctions](/04-methodes-et-fonctions/README.md)
- 4.1. [Définition et appel de méthodes](/04-methodes-et-fonctions/4-1-definition-et-appel-de-methodes.md)
    - 4.1.1. Signature de méthode
    - 4.1.2. Valeurs de retour et void
    - 4.1.3. Méthodes statiques vs d'instance

- 4.2. [Paramètres et valeurs de retour](/04-methodes-et-fonctions/4-2-parametres-et-valeurs-de-retour.md)
    - 4.2.1. Passage par valeur vs référence (ref, out)
    - 4.2.2. Paramètres de tableau params
    - 4.2.3. Tuples comme valeurs de retour (C# 7+)
    - 4.2.4. Discards (C# 7+)

- 4.3. [Surcharge de méthodes](/04-methodes-et-fonctions/4-3-surcharge-de-methodes.md)
    - 4.3.1. Règles de résolution de surcharge
    - 4.3.2. Bonnes pratiques

- 4.4. [Paramètres optionnels et nommés](/04-methodes-et-fonctions/4-4-parametres-optionnels-et-nommes.md)
    - 4.4.1. Valeurs par défaut
    - 4.4.2. Ordre d'appel avec paramètres nommés

- 4.5. [Méthodes d'extension](/04-methodes-et-fonctions/4-5-methodes-dextension.md)
    - 4.5.1. Création et utilisation
    - 4.5.2. LINQ comme exemple de méthodes d'extension
    - 4.5.3. Bonnes pratiques et pièges courants

- 4.6. [Méthodes locales (C# 7+)](/04-methodes-et-fonctions/4-6-methodes-locales-csharp-7-plus.md)
    - 4.6.1. Définition dans d'autres méthodes
    - 4.6.2. Avantages et cas d'utilisation

- 4.7. [Expressions de fonction (lambdas)](/04-methodes-et-fonctions/4-7-expressions-de-fonction-lambdas.md)
    - 4.7.1. Syntaxe des expressions lambda
    - 4.7.2. Lambdas comme délégués et expressions
    - 4.7.3. Capture de variables (closures)

## 5. [Programmation orientée objet en C#](/05-programmation-orientee-objet-en-csharp/README.md)
- 5.1. [Classes et objets](/05-programmation-orientee-objet-en-csharp/5-1-classes-et-objets.md)
    - 5.1.1. Création et instanciation
    - 5.1.2. Membres de classe
    - 5.1.3. Membres statiques
    - 5.1.4. Initialisation d'objets

- 5.2. [Propriétés et encapsulation](/05-programmation-orientee-objet-en-csharp/5-2-proprietes-et-encapsulation.md)
    - 5.2.1. Getters et setters
    - 5.2.2. Propriétés auto-implémentées
    - 5.2.3. Propriétés en lecture seule
    - 5.2.4. Expression-bodied properties (C# 6+)
    - 5.2.5. Init-only setters (C# 9+)

- 5.3. [Constructeurs et finaliseurs](/05-programmation-orientee-objet-en-csharp/5-3-constructeurs-et-finaliseurs.md)
    - 5.3.1. Constructeurs par défaut, paramétrés et statiques
    - 5.3.2. Initialisation d'objets
    - 5.3.3. Chaînage de constructeurs
    - 5.3.4. Finaliseurs et pattern IDisposable

- 5.4. [Héritage](/05-programmation-orientee-objet-en-csharp/5-4-heritage.md)
    - 5.4.1. Classes de base et dérivées
    - 5.4.2. Méthodes virtuelles et substitution
    - 5.4.3. Classe Object et ses méthodes
    - 5.4.4. Héritage vs composition

- 5.5. [Polymorphisme](/05-programmation-orientee-objet-en-csharp/5-5-polymorphisme.md)
    - 5.5.1. Polymorphisme d'inclusion
    - 5.5.2. Polymorphisme ad hoc (surcharge)
    - 5.5.3. Polymorphisme paramétrique (génériques)
    - 5.5.4. Cast et is/as

- 5.6. [Interfaces](/05-programmation-orientee-objet-en-csharp/5-6-interfaces.md)
    - 5.6.1. Définition et implémentation
    - 5.6.2. Interfaces multiples
    - 5.6.3. Implémentation explicite
    - 5.6.4. Interfaces par défaut (C# 8+)
    - 5.6.5. Interfaces génériques

- 5.7. [Classes abstraites](/05-programmation-orientee-objet-en-csharp/5-7-classes-abstraites.md)
    - 5.7.1. Différence avec les interfaces
    - 5.7.2. Méthodes abstraites
    - 5.7.3. Implémentation partielle

- 5.8. [Modificateurs d'accès](/05-programmation-orientee-objet-en-csharp/5-8-modificateurs-dacces.md)
    - 5.8.1. public, private, protected, internal
    - 5.8.2. Combinaisons et meilleures pratiques
    - 5.8.3. Contrôle de la visibilité

- 5.9. [Records (C# 9+)](/05-programmation-orientee-objet-en-csharp/5-9-records-csharp-9-plus.md)
    - 5.9.1. Immutabilité et égalité basée sur les valeurs
    - 5.9.2. Avec-expressions pour les copies modifiées
    - 5.9.3. Déconstruction et pattern matching

- 5.10. [Types par valeur (structs)](/05-programmation-orientee-objet-en-csharp/5-10-types-par-valeur-structs.md)
    - 5.10.1. Différence avec les classes
    - 5.10.2. Struct mutable vs immutable
    - 5.10.3. Struct records (C# 10+)

## 6. [Fonctionnalités avancées de C#](/06-fonctionnalites-avancees-de-csharp/README.md)
- 6.1. [Délégués et événements](/06-fonctionnalites-avancees-de-csharp/6-1-delegues-et-evenements.md)
    - 6.1.1. Types de délégués (Action, Func, Predicate)
    - 6.1.2. Délégués multicasts
    - 6.1.3. Événements et pattern observateur
    - 6.1.4. EventHandler et EventArgs

- 6.2. [Expressions Lambda](/06-fonctionnalites-avancees-de-csharp/6-2-expressions-lambda.md)
    - 6.2.1. Syntaxe et utilisation
    - 6.2.2. Closures
    - 6.2.3. Délégués vs lambdas
    - 6.2.4. Lambdas avec état (C# 9+)

- 6.3. [LINQ (Language Integrated Query)](/06-fonctionnalites-avancees-de-csharp/6-3-linq-language-integrated-query.md)
    - 6.3.1. Syntaxe de requête
    - 6.3.2. Syntaxe de méthode
    - 6.3.3. Opérateurs LINQ essentiels
    - 6.3.4. LINQ to Objects
    - 6.3.5. Requêtes différées et requêtes immédiates
    - 6.3.6. Projection et transformation de données
    - 6.3.7. Agrégation et groupement

- 6.4. [Génériques](/06-fonctionnalites-avancees-de-csharp/6-4-generiques.md)
    - 6.4.1. Classes et méthodes génériques
    - 6.4.2. Contraintes sur les types génériques
    - 6.4.3. Covariance et contravariance
    - 6.4.4. Interfaces génériques
    - 6.4.5. Generic type inference

- 6.5. [Gestion des exceptions](/06-fonctionnalites-avancees-de-csharp/6-5-gestion-des-exceptions.md)
    - 6.5.1. try-catch-finally
    - 6.5.2. Exceptions personnalisées
    - 6.5.3. Meilleures pratiques
    - 6.5.4. Filtres d'exception
    - 6.5.5. Performance et considérations

- 6.6. [Types anonymes et dynamiques](/06-fonctionnalites-avancees-de-csharp/6-6-types-anonymes-et-dynamiques.md)
    - 6.6.1. Objets anonymes
    - 6.6.2. Type dynamic
    - 6.6.3. Interopérabilité DLR

- 6.7. [Méthodes anonymes](/06-fonctionnalites-avancees-de-csharp/6-7-methodes-anonymes.md)
    - 6.7.1. Comparaison avec lambdas
    - 6.7.2. Cas d'usage

- 6.8. [Async/Await et programmation asynchrone](/06-fonctionnalites-avancees-de-csharp/6-8-async-await-et-programmation-asynchrone.md)
    - 6.8.1. Concepts de base
    - 6.8.2. Patterns asynchrones
    - 6.8.3. Gestion des exceptions
    - 6.8.4. Annulation et timeout
    - 6.8.5. ConfigureAwait et contexte de synchronisation
    - 6.8.6. Performance et bonnes pratiques

## 7. [Windows Forms (WinForms)](/07-windows-forms-winforms/README.md)
- 7.1. [Introduction à WinForms](/07-windows-forms-winforms/7-1-introduction-a-winforms.md)
    - 7.1.1. Architecture et cycle de vie
    - 7.1.2. Designer visuel vs code
    - 7.1.3. Windows Forms avec .NET Core/.NET 5+

- 7.2. [Création d'une première application](/07-windows-forms-winforms/7-2-creation-dune-premiere-application.md)
    - 7.2.1. Structure d'un projet WinForms
    - 7.2.2. Comprendre les fichiers générés
    - 7.2.3. Application simple étape par étape

- 7.3. [Contrôles de base et leurs propriétés](/07-windows-forms-winforms/7-3-controles-de-base-et-leurs-proprietes.md)
    - 7.3.1. Button, Label, TextBox, CheckBox, etc.
    - 7.3.2. Propriétés communes
    - 7.3.3. Création de contrôles personnalisés
    - 7.3.4. Contrôles composites

- 7.4. [Gestion des événements](/07-windows-forms-winforms/7-4-gestion-des-evenements.md)
    - 7.4.1. Modèle d'événement .NET
    - 7.4.2. Événements communs (Click, Load, etc.)
    - 7.4.3. Création d'événements personnalisés
    - 7.4.4. Gestion des événements de clavier et souris

- 7.5. [Mise en page et design](/07-windows-forms-winforms/7-5-mise-en-page-et-design.md)
    - 7.5.1. Contrôles de conteneur (Panel, GroupBox)
    - 7.5.2. Layout managers (TableLayoutPanel, FlowLayoutPanel)
    - 7.5.3. Ancrages et docking
    - 7.5.4. Design adaptatif
    - 7.5.5. Thèmes et apparence

- 7.6. [Menus et barres d'outils](/07-windows-forms-winforms/7-6-menus-et-barres-doutils.md)
    - 7.6.1. MenuStrip
    - 7.6.2. ContextMenuStrip
    - 7.6.3. ToolStrip et StatusStrip
    - 7.6.4. Actions communes et raccourcis clavier

- 7.7. [Dialogues et messages](/07-windows-forms-winforms/7-7-dialogues-et-messages.md)
    - 7.7.1. MessageBox et dialogues standards
    - 7.7.2. Création de dialogues personnalisés
    - 7.7.3. DialogResult
    - 7.7.4. Dialogues de fichiers et dossiers

- 7.8. [Liaison de données (Data Binding)](/07-windows-forms-winforms/7-8-liaison-de-donnees-data-binding.md)
    - 7.8.1. Binding simple
    - 7.8.2. Binding complexe
    - 7.8.3. BindingSource et BindingList
    - 7.8.4. Validation de données

- 7.9. [Multithreading dans WinForms](/07-windows-forms-winforms/7-9-multithreading-dans-winforms.md)
    - 7.9.1. Problématique du thread UI
    - 7.9.2. Invoke et BeginInvoke
    - 7.9.3. BackgroundWorker
    - 7.9.4. Async/await dans WinForms
    - 7.9.5. Indicateurs de progression

## 8. [Windows Presentation Foundation (WPF)](/08-windows-presentation-foundation-wpf/README.md)
- 8.1. [Introduction à WPF](/08-windows-presentation-foundation-wpf/8-1-introduction-a-wpf.md)
    - 8.1.1. Comparaison avec WinForms
    - 8.1.2. Architecture de WPF
    - 8.1.3. WPF avec .NET Core/.NET 5+

- 8.2. [XAML et fondamentaux](/08-windows-presentation-foundation-wpf/8-2-xaml-et-fondamentaux.md)
    - 8.2.1. Syntaxe XAML
    - 8.2.2. Espace de noms et préfixes
    - 8.2.3. Relation XAML/code-behind
    - 8.2.4. Markup Extensions
    - 8.2.5. Éditeur XAML et designer

- 8.3. [Mise en page avec les Panels](/08-windows-presentation-foundation-wpf/8-3-mise-en-page-avec-les-panels.md)
    - 8.3.1. Grid
    - 8.3.2. StackPanel
    - 8.3.3. WrapPanel
    - 8.3.4. DockPanel
    - 8.3.5. Canvas
    - 8.3.6. Layouts combinés et complexes

- 8.4. [Contrôles WPF](/08-windows-presentation-foundation-wpf/8-4-controles-wpf.md)
    - 8.4.1. Contrôles de base
    - 8.4.2. Contrôles de contenu
    - 8.4.3. Contrôles de données
    - 8.4.4. Contrôles de navigation
    - 8.4.5. DatePicker, ColorPicker et autres contrôles spécialisés

- 8.5. [Styles et templates](/08-windows-presentation-foundation-wpf/8-5-styles-et-templates.md)
    - 8.5.1. Définition et application de styles
    - 8.5.2. Héritage de styles
    - 8.5.3. ControlTemplate
    - 8.5.4. DataTemplate
    - 8.5.5. Triggers (Property, Data, Event)
    - 8.5.6. Visual States

- 8.6. [Binding et DataContext](/08-windows-presentation-foundation-wpf/8-6-binding-et-datacontext.md)
    - 8.6.1. Syntax de binding
    - 8.6.2. Modes de binding
    - 8.6.3. Conversion de données
    - 8.6.4. ValidationRules
    - 8.6.5. Change Notification (INotifyPropertyChanged)
    - 8.6.6. CollectionView et filtrage

- 8.7. [Commandes et MVVM (Model-View-ViewModel)](/08-windows-presentation-foundation-wpf/8-7-commandes-et-mvvm-model-view-viewmodel.md)
    - 8.7.1. Pattern MVVM
    - 8.7.2. ICommand
    - 8.7.3. RelayCommand
    - 8.7.4. Frameworks MVVM (MVVM Light, Prism, etc.)
    - 8.7.5. Injection de dépendances dans MVVM

- 8.8. [Ressources et dictionnaires de ressources](/08-windows-presentation-foundation-wpf/8-8-ressources-et-dictionnaires-de-ressources.md)
    - 8.8.1. Resources locales
    - 8.8.2. Ressources d'application
    - 8.8.3. Ressources externes
    - 8.8.4. ResourceDictionary et fusion
    - 8.8.5. Thèmes dynamiques

- 8.9. [Animations et effets visuels](/08-windows-presentation-foundation-wpf/8-9-animations-et-effets-visuels.md)
    - 8.9.1. Animations basées sur le temps
    - 8.9.2. Animations basées sur les keyframes
    - 8.9.3. Storyboards
    - 8.9.4. Effets et transformations
    - 8.9.5. Transitions et comportements

- 8.10. [Personnalisation des contrôles](/08-windows-presentation-foundation-wpf/8-10-personnalisation-des-controles.md)
    - 8.10.1. Création de contrôles personnalisés
    - 8.10.2. Héritage de contrôles existants
    - 8.10.3. Propriétés de dépendance
    - 8.10.4. Événements routés
    - 8.10.5. Attached properties

- 8.11. [Localisation et internationalisation](/08-windows-presentation-foundation-wpf/8-11-localisation-et-internationalisation.md)
    - 8.11.1. Resources .resx
    - 8.11.2. Binding aux ressources localisées
    - 8.11.3. Changement de culture à l'exécution
    - 8.11.4. Formatage selon la culture

## 9. [Accès aux données avec ADO.NET](/09-acces-aux-donnees-avec-ado-dotnet/README.md)
- 9.1. [Introduction à ADO.NET](/09-acces-aux-donnees-avec-ado-dotnet/9-1-introduction-a-ado-dotnet.md)
    - 9.1.1. Architecture d'ADO.NET
    - 9.1.2. Providers de données
    - 9.1.3. Modèle connecté vs déconnecté

- 9.2. [Connexion aux bases de données](/09-acces-aux-donnees-avec-ado-dotnet/9-2-connexion-aux-bases-de-donnees.md)
    - 9.2.1. Chaînes de connexion
    - 9.2.2. Connection pooling
    - 9.2.3. Gestion des connexions
    - 9.2.4. Sécurité des connexions

- 9.3. [Exécution de commandes SQL](/09-acces-aux-donnees-avec-ado-dotnet/9-3-execution-de-commandes-sql.md)
    - 9.3.1. SELECT, INSERT, UPDATE, DELETE
    - 9.3.2. Paramètres de commande
    - 9.3.3. ExecuteReader, ExecuteNonQuery, ExecuteScalar
    - 9.3.4. Gestion des types de données

- 9.4. [DataSet et DataAdapter](/09-acces-aux-donnees-avec-ado-dotnet/9-4-dataset-et-dataadapter.md)
    - 9.4.1. Remplissage de DataSets
    - 9.4.2. Manipulation des données
    - 9.4.3. Mise à jour de la base de données
    - 9.4.4. DataTable, DataRow et DataColumn

- 9.5. [Transactions](/09-acces-aux-donnees-avec-ado-dotnet/9-5-transactions.md)
    - 9.5.1. Création et gestion
    - 9.5.2. Isolation levels
    - 9.5.3. Transactions distribuées
    - 9.5.4. Pattern Unit of Work

- 9.6. [Procédures stockées](/09-acces-aux-donnees-avec-ado-dotnet/9-6-procedures-stockees.md)
    - 9.6.1. Appel de procédures stockées
    - 9.6.2. Paramètres IN/OUT
    - 9.6.3. Récupération des résultats
    - 9.6.4. Bonnes pratiques

## 10. [Entity Framework Core](/10-entity-framework-core/README.md)
- 10.1. [Introduction à Entity Framework Core](/10-entity-framework-core/10-1-introduction-a-entity-framework-core.md)
    - 10.1.1. ORM et ses avantages
    - 10.1.2. Installation et configuration
    - 10.1.3. Différences avec EF6
    - 10.1.4. Versions et compatibilité

- 10.2. [Approches de modélisation](/10-entity-framework-core/10-2-approches-de-modelisation.md)
    - 10.2.1. Code First
    - 10.2.2. Database First
    - 10.2.3. Model First
    - 10.2.4. Reverse Engineering

- 10.3. [DbContext et DbSet](/10-entity-framework-core/10-3-dbcontext-et-dbset.md)
    - 10.3.1. Configuration du contexte
    - 10.3.2. Gestion du cycle de vie
    - 10.3.3. Fluent API vs annotations
    - 10.3.4. Conventions de nommage

- 10.4. [Migrations](/10-entity-framework-core/10-4-migrations.md)
    - 10.4.1. Création de migrations
    - 10.4.2. Application de migrations
    - 10.4.3. Personnalisation de migrations
    - 10.4.4. Stratégies de migration en production

- 10.5. [Requêtes LINQ to Entities](/10-entity-framework-core/10-5-requetes-linq-to-entities.md)
    - 10.5.1. Requêtes de base
    - 10.5.2. Requêtes avancées
    - 10.5.3. Chargement de données associées
    - 10.5.4. Requêtes brutes SQL
    - 10.5.5. Procédures stockées et fonctions

- 10.6. [Relations entre entités](/10-entity-framework-core/10-6-relations-entre-entites.md)
    - 10.6.1. One-to-One
    - 10.6.2. One-to-Many
    - 10.6.3. Many-to-Many
    - 10.6.4. Self-referencing
    - 10.6.5. Configuration des relations avec Fluent API

- 10.7. [Chargement de données](/10-entity-framework-core/10-7-chargement-de-donnees.md)
    - 10.7.1. Eager Loading
    - 10.7.2. Lazy Loading
    - 10.7.3. Explicit Loading
    - 10.7.4. Stratégies de chargement et performance

- 10.8. [Transactions et concurrence](/10-entity-framework-core/10-8-transactions-et-concurrence.md)
    - 10.8.1. Gestion des transactions
    - 10.8.2. Résolution de conflits
    - 10.8.3. Tokens de concurrence
    - 10.8.4. Stratégies pessimistes et optimistes

## 11. [Intégration avec MySQL/MariaDB](/11-integration-avec-mysql-mariadb/README.md)
- 11.1. [Installation des connecteurs](/11-integration-avec-mysql-mariadb/11-1-installation-des-connecteurs.md)
    - 11.1.1. MySQL Connector/NET
    - 11.1.2. Pomelo.EntityFrameworkCore.MySql
    - 11.1.3. MySqlConnector

- 11.2. [Configuration de la connexion](/11-integration-avec-mysql-mariadb/11-2-configuration-de-la-connexion.md)
    - 11.2.1. Chaînes de connexion
    - 11.2.2. Options spécifiques à MySQL/MariaDB
    - 11.2.3. Configuration pour différents environnements

- 11.3. [CRUD avec MySQL/MariaDB](/11-integration-avec-mysql-mariadb/11-3-crud-avec-mysql-mariadb.md)
    - 11.3.1. Considérations spécifiques
    - 11.3.2. Requêtes paramétrées
    - 11.3.3. Types de données et mappings
    - 11.3.4. Batch operations

- 11.4. [Entity Framework Core avec MySQL/MariaDB](/11-integration-avec-mysql-mariadb/11-4-entity-framework-core-avec-mysql-mariadb.md)
    - 11.4.1. Configuration spécifique
    - 11.4.2. Migrations
    - 11.4.3. Scénarios avancés
    - 11.4.4. Fonctionnalités spécifiques à MySQL/MariaDB

- 11.5. [Optimisation des performances](/11-integration-avec-mysql-mariadb/11-5-optimisation-des-performances.md)
    - 11.5.1. Indexation
    - 11.5.2. Connection pooling
    - 11.5.3. Réglage des requêtes
    - 11.5.4. Profiling et diagnostics
    - 11.5.5. Mise en cache

- 11.6. [Sécurité et bonnes pratiques](/11-integration-avec-mysql-mariadb/11-6-securite-et-bonnes-pratiques.md)
    - 11.6.1. Protection contre les injections SQL
    - 11.6.2. Gestion sécurisée des connexions
    - 11.6.3. Encryption des données
    - 11.6.4. Audit et journalisation
    - 11.6.5. Sauvegarde et restauration

## 12. [Développement Web avec ASP.NET](/12-developpement-web-avec-asp-dotnet/README.md)
- 12.1. [Introduction à ASP.NET](/12-developpement-web-avec-asp-dotnet/12-1-introduction-a-asp-dotnet.md)
    - 12.1.1. Historique et évolution
    - 12.1.2. Frameworks ASP.NET
    - 12.1.3. ASP.NET vs ASP.NET Core

- 12.2. [ASP.NET MVC](/12-developpement-web-avec-asp-dotnet/12-2-asp-dotnet-mvc.md)
    - 12.2.1. Architecture MVC
    - 12.2.2. Contrôleurs et actions
    - 12.2.3. Vues et moteurs de template
    - 12.2.4. Modèles et validation
    - 12.2.5. Routing
    - 12.2.6. Filtres et attributs

- 12.3. [ASP.NET Core](/12-developpement-web-avec-asp-dotnet/12-3-asp-dotnet-core.md)
    - 12.3.1. Architecture et principes
    - 12.3.2. Configuration et environnements
    - 12.3.3. Dependency Injection intégrée
    - 12.3.4. Hébergement et déploiement
    - 12.3.5. Blazor (Server et WebAssembly)

- 12.4. [Razor Pages](/12-developpement-web-avec-asp-dotnet/12-4-razor-pages.md)
    - 12.4.1. Modèle de programmation
    - 12.4.2. Handlers et binding
    - 12.4.3. Comparison avec MVC
    - 12.4.4. Razor components et réutilisation

- 12.5. [API Web et services REST](/12-developpement-web-avec-asp-dotnet/12-5-api-web-et-services-rest.md)
    - 12.5.1. Création d'API RESTful
    - 12.5.2. Controllers API
    - 12.5.3. Sérialisation et désérialisation
    - 12.5.4. Formatters (JSON, XML)
    - 12.5.5. Routes et conventions d'API

- 12.6. [Documentation API avec Swagger/OpenAPI](/12-developpement-web-avec-asp-dotnet/12-6-documentation-api-avec-swagger-openapi.md)
    - 12.6.1. Installation et configuration de Swagger
    - 12.6.2. Documentation des endpoints
    - 12.6.3. Personnalisation de l'interface Swagger UI
    - 12.6.4. Génération de clients API
    - 12.6.5. Intégration avec autres outils

- 12.7. [Authentification et autorisation](/12-developpement-web-avec-asp-dotnet/12-7-authentification-et-autorisation.md)
    - 12.7.1. Identity Framework
    - 12.7.2. JWT et tokens
    - 12.7.3. Politiques d'autorisation
    - 12.7.4. OAuth et OpenID Connect
    - 12.7.5. Authentification à deux facteurs

- 12.8. [Middleware et pipeline de requêtes](/12-developpement-web-avec-asp-dotnet/12-8-middleware-et-pipeline-de-requetes.md)
    - 12.8.1. Concept de middleware
    - 12.8.2. Middleware intégré
    - 12.8.3. Création de middleware personnalisé
    - 12.8.4. Configuration du pipeline
    - 12.8.5. Diagnostics et logging

## 13. [Applications de bureau modernes avec .NET](/13-applications-de-bureau-modernes-avec-dotnet/README.md)
- 13.1. [Windows App SDK (anciennement Project Reunion)](/13-applications-de-bureau-modernes-avec-dotnet/13-1-windows-app-sdk-anciennement-project-reunion.md)
    - 13.1.1. Vue d'ensemble et objectifs
    - 13.1.2. Utilisation avec C#
    - 13.1.3. Intégration avec WinUI
    - 13.1.4. Accès aux APIs Windows modernes

- 13.2. [MAUI (Multi-platform App UI)](/13-applications-de-bureau-modernes-avec-dotnet/13-2-maui-multi-platform-app-ui.md)
    - 13.2.1. Introduction et architecture
    - 13.2.2. Développement multiplateforme
    - 13.2.3. Création d'interfaces utilisateur
    - 13.2.4. Accès aux API natives
    - 13.2.5. Contrôles et layouts

- 13.3. [WinUI 3](/13-applications-de-bureau-modernes-avec-dotnet/13-3-winui-3.md)
    - 13.3.1. Comparaison avec WPF
    - 13.3.2. Fluent Design System
    - 13.3.3. Migration depuis UWP/WPF
    - 13.3.4. Composition et Effets

- 13.4. [Applications de bureau empaquetées (MSIX)](/13-applications-de-bureau-modernes-avec-dotnet/13-4-applications-de-bureau-empaquetees-msix.md)
    - 13.4.1. Création de packages MSIX
    - 13.4.2. Déploiement et mise à jour
    - 13.4.3. Sécurité et isolation
    - 13.4.4. Intégration avec l'App Store

## 14. [Déploiement et distribution](/14-deploiement-et-distribution/README.md)
- 14.1. [Compilation et build](/14-deploiement-et-distribution/14-1-compilation-et-build.md)
    - 14.1.1. Configuration de build
    - 14.1.2. Optimisation
    - 14.1.3. Différences Debug/Release
    - 14.1.4. CI/CD pour applications .NET

- 14.2. [Déploiement d'applications Windows Forms](/14-deploiement-et-distribution/14-2-deploiement-dapplications-windows-forms.md)
    - 14.2.1. Installation classique
    - 14.2.2. Déploiement xcopy
    - 14.2.3. Considérations sur les dépendances
    - 14.2.4. Self-contained vs Framework-dependent

- 14.3. [Déploiement d'applications WPF](/14-deploiement-et-distribution/14-3-deploiement-dapplications-wpf.md)
    - 14.3.1. Packaging et installation
    - 14.3.2. Ressources et actifs
  - 14.3.3. Mise à jour des applications
  - 14.3.4. Sécurité et permissions
  - 14.3.5. Installation silencieuse et automatisée

- 14.4. [Déploiement d'applications Web ASP.NET](/14-deploiement-et-distribution/14-4-deploiement-dapplications-web-asp-dotnet.md)
  - 14.4.1. IIS
  - 14.4.2. Kestrel
  - 14.4.3. Docker et conteneurs
  - 14.4.4. Azure App Service
  - 14.4.5. Configurations pour différents environnements
  - 14.4.6. Supervision et diagnostics

- 14.5. [Publication avec ClickOnce](/14-deploiement-et-distribution/14-5-publication-avec-clickonce.md)
  - 14.5.1. Configuration
  - 14.5.2. Mise à jour automatique
  - 14.5.3. Personnalisation
  - 14.5.4. Sécurité et signature
  - 14.5.5. Déploiement hors ligne

- 14.6. [Packaging MSIX](/14-deploiement-et-distribution/14-6-packaging-msix.md)
  - 14.6.1. Création de packages
  - 14.6.2. Distribution via Microsoft Store
  - 14.6.3. Distribution hors Store
  - 14.6.4. Mise à jour des packages
  - 14.6.5. Certificats et signature

- 14.7. [Configurations de déploiement](/14-deploiement-et-distribution/14-7-configurations-de-deploiement.md)
  - 14.7.1. Transformations de configuration
  - 14.7.2. Gestion des secrets
  - 14.7.3. Stratégies de migration de données
  - 14.7.4. Rollback et récupération
  - 14.7.5. Blue-Green deployment

## 15. [Tests et qualité de code](/15-tests-et-qualite-de-code/README.md)
- 15.1. [Tests unitaires](/15-tests-et-qualite-de-code/15-1-tests-unitaires.md)
  - 15.1.1. MSTest
  - 15.1.2. NUnit
  - 15.1.3. xUnit
  - 15.1.4. Assertions et vérifications
  - 15.1.5. Code coverage
  - 15.1.6. Tests paramétrés

- 15.2. [Mocking avec Moq](/15-tests-et-qualite-de-code/15-2-mocking-avec-moq.md)
  - 15.2.1. Création de mocks
  - 15.2.2. Vérification des interactions
  - 15.2.3. Mocking avancé
  - 15.2.4. Alternatives à Moq (NSubstitute, FakeItEasy)

- 15.3. [Tests d'intégration](/15-tests-et-qualite-de-code/15-3-tests-dintegration.md)
  - 15.3.1. Configuration
  - 15.3.2. Tests de base de données
  - 15.3.3. Tests d'API
  - 15.3.4. Tests d'interface utilisateur
  - 15.3.5. Tests de performance

- 15.4. [Analyse de code et refactoring](/15-tests-et-qualite-de-code/15-4-analyse-de-code-et-refactoring.md)
  - 15.4.1. Analyseurs statiques
  - 15.4.2. Style Cop
  - 15.4.3. Métriques de code
  - 15.4.4. Techniques de refactoring
  - 15.4.5. Code smells et remèdes

- 15.5. [Débogage avancé](/15-tests-et-qualite-de-code/15-5-debogage-avance.md)
  - 15.5.1. Points d'arrêt conditionnels
  - 15.5.2. Visualisation des données
  - 15.5.3. Débogage distant
  - 15.5.4. Débogage de production
  - 15.5.5. Memory dumps et analyse post-mortem

- 15.6. [Outils de qualité de code](/15-tests-et-qualite-de-code/15-6-outils-de-qualite-de-code.md)
  - 15.6.1. SonarQube
  - 15.6.2. NDepend
  - 15.6.3. ReSharper/Rider
  - 15.6.4. Intégration dans les pipelines CI/CD
  - 15.6.5. Surveillance continue de la qualité

## 16. [Sécurité en C#](/16-securite-en-csharp/README.md)
- 16.1. [Gestion des chaînes de connexion](/16-securite-en-csharp/16-1-gestion-des-chaines-de-connexion.md)
  - 16.1.1. Protection des informations sensibles
  - 16.1.2. Utilisation de secrets
  - 16.1.3. Rotation des informations d'identification
  - 16.1.4. Environment variables et User Secrets

- 16.2. [Protection contre les injections SQL](/16-securite-en-csharp/16-2-protection-contre-les-injections-sql.md)
  - 16.2.1. Paramétrage des requêtes
  - 16.2.2. ORM et sécurité
  - 16.2.3. Validation des entrées
  - 16.2.4. Principes de moindre privilège
  - 16.2.5. Audit de sécurité des requêtes

- 16.3. [Cryptographie](/16-securite-en-csharp/16-3-cryptographie.md)
  - 16.3.1. Hachage et salage
  - 16.3.2. Chiffrement symétrique
  - 16.3.3. Chiffrement asymétrique
  - 16.3.4. API de cryptographie .NET
  - 16.3.5. Bonnes pratiques en cryptographie

- 16.4. [Authentification et autorisation](/16-securite-en-csharp/16-4-authentification-et-autorisation.md)
  - 16.4.1. Systèmes d'authentification
  - 16.4.2. RBAC et ABAC
  - 16.4.3. Gestion des tokens
  - 16.4.4. Multi-factor authentication
  - 16.4.5. Gestion des sessions

- 16.5. [Gestion sécurisée des secrets](/16-securite-en-csharp/16-5-gestion-securisee-des-secrets.md)
  - 16.5.1. Azure Key Vault
  - 16.5.2. Secret Manager
  - 16.5.3. DPAPI
  - 16.5.4. Stockage sécurisé en environnement conteneurisé
  - 16.5.5. Rotation automatique des secrets

- 16.6. [Sécurité des applications Web](/16-securite-en-csharp/16-6-securite-des-applications-web.md)
  - 16.6.1. HTTPS et certificats SSL/TLS
  - 16.6.2. Protection CSRF/XSRF
  - 16.6.3. Headers de sécurité
  - 16.6.4. CSP (Content Security Policy)
  - 16.6.5. OWASP Top 10 et prévention

## 17. [Performances et optimisation](/17-performances-et-optimisation/README.md)
- 17.1. [Gestion de la mémoire](/17-performances-et-optimisation/17-1-gestion-de-la-memoire.md)
  - 17.1.1. Types référence vs types valeur
  - 17.1.2. Stratégies d'allocation
  - 17.1.3. Span<T> et Memory<T>
  - 17.1.4. Évitement des allocations inutiles
  - 17.1.5. Pooling d'objets

- 17.2. [Garbage Collector](/17-performances-et-optimisation/17-2-garbage-collector.md)
  - 17.2.1. Fonctionnement du GC
  - 17.2.2. Générations
  - 17.2.3. Configuration et tuning
  - 17.2.4. Gestion des ressources non managées
  - 17.2.5. Weak references et finalizers

- 17.3. [Multithreading et parallélisme](/17-performances-et-optimisation/17-3-multithreading-et-parallelisme.md)
  - 17.3.1. Thread vs Task
  - 17.3.2. Parallel LINQ (PLINQ)
  - 17.3.3. Synchronisation
  - 17.3.4. Problèmes courants (deadlocks, race conditions)
  - 17.3.5. Atomic operations et interlocked
  - 17.3.6. TPL Dataflow

- 17.4. [Optimisation des requêtes LINQ](/17-performances-et-optimisation/17-4-optimisation-des-requetes-linq.md)
  - 17.4.1. Évaluation différée vs immédiate
  - 17.4.2. Analyse de performances
  - 17.4.3. Stratégies d'optimisation
  - 17.4.4. Minimisation des requêtes
  - 17.4.5. Projections et sélection

- 17.5. [Profilage des applications](/17-performances-et-optimisation/17-5-profilage-des-applications.md)
  - 17.5.1. Outils de profilage
  - 17.5.2. Mesure de performances
  - 17.5.3. Diagnostic Memory Leak
  - 17.5.4. Benchmark.NET
  - 17.5.5. Performance counters

- 17.6. [Techniques d'optimisation avancées](/17-performances-et-optimisation/17-6-techniques-doptimisation-avancees.md)
  - 17.6.1. Compilation JIT vs AOT
  - 17.6.2. Unsafe code et pointeurs
  - 17.6.3. Interop avec du code natif
  - 17.6.4. Structs en pile vs tas
  - 17.6.5. SIMD avec Vector<T>
  - 17.6.6. ReadOnlySpan et stackalloc

## 18. [Interopérabilité](/18-interoperabilite/README.md)
- 18.1. [P/Invoke et appels de code natif](/18-interoperabilite/18-1-p-invoke-et-appels-de-code-natif.md)
  - 18.1.1. Marshaling
  - 18.1.2. Callbacks
  - 18.1.3. Gestion des erreurs
  - 18.1.4. Optimisation des performances
  - 18.1.5. LibraryImport (C# 11+)

- 18.2. [COM Interop](/18-interoperabilite/18-2-com-interop.md)
  - 18.2.1. Utilisation de composants COM
  - 18.2.2. Création de wrappers
  - 18.2.3. Automation
  - 18.2.4. RCW et CCW
  - 18.2.5. Early vs Late binding

- 18.3. [Intégration avec d'autres langages](/18-interoperabilite/18-3-integration-avec-dautres-langages.md)
  - 18.3.1. Interopérabilité avec F#
  - 18.3.2. Interopérabilité avec VB.NET
  - 18.3.3. Interopérabilité avec C++/CLI
  - 18.3.4. Intégration avec Python (IronPython)
  - 18.3.5. Intégration avec R

- 18.4. [Intégration avec JavaScript/TypeScript](/18-interoperabilite/18-4-integration-avec-javascript-typescript.md)
  - 18.4.1. Blazor WebAssembly
  - 18.4.2. SignalR
  - 18.4.3. APIs REST et JSON
  - 18.4.4. gRPC-Web
  - 18.4.5. JSInterop

- 18.5. [Intégration cloud et services](/18-interoperabilite/18-5-integration-cloud-et-services.md)
  - 18.5.1. Azure Functions
  - 18.5.2. gRPC
  - 18.5.3. GraphQL
  - 18.5.4. Microservices en C#
  - 18.5.5. Event-driven architecture

## 19. [Design Patterns en C#](/19-design-patterns-en-csharp/README.md)
- 19.1. [Patterns créationnels](/19-design-patterns-en-csharp/19-1-patterns-creationnels.md)
  - 19.1.1. Singleton
  - 19.1.2. Factory Method
  - 19.1.3. Abstract Factory
  - 19.1.4. Builder
  - 19.1.5. Prototype
  - 19.1.6. Object Pool

- 19.2. [Patterns structurels](/19-design-patterns-en-csharp/19-2-patterns-structurels.md)
  - 19.2.1. Adapter
  - 19.2.2. Bridge
  - 19.2.3. Composite
  - 19.2.4. Decorator
  - 19.2.5. Façade
  - 19.2.6. Proxy
  - 19.2.7. Flyweight

- 19.3. [Patterns comportementaux](/19-design-patterns-en-csharp/19-3-patterns-comportementaux.md)
  - 19.3.1. Observer
  - 19.3.2. Strategy
  - 19.3.3. Command
  - 19.3.4. Template Method
  - 19.3.5. Iterator
  - 19.3.6. State
  - 19.3.7. Chain of Responsibility
  - 19.3.8. Mediator
  - 19.3.9. Visitor

- 19.4. [Patterns architecturaux](/19-design-patterns-en-csharp/19-4-patterns-architecturaux.md)
  - 19.4.1. MVC
  - 19.4.2. MVVM
  - 19.4.3. Repository
  - 19.4.4. Unit of Work
  - 19.4.5. Dependency Injection
  - 19.4.6. CQRS
  - 19.4.7. Event Sourcing

- 19.5. [Patterns spécifiques à .NET](/19-design-patterns-en-csharp/19-5-patterns-specifiques-a-dotnet.md)
  - 19.5.1. Disposable Pattern
  - 19.5.2. Async/await patterns
  - 19.5.3. Options Pattern
  - 19.5.4. Factory Pattern avec DI

## 20. [Projets pratiques](/20-projets-pratiques/README.md)
- 20.1. [Application de gestion avec WinForms et MySQL](/20-projets-pratiques/20-1-application-de-gestion-avec-winforms-et-mysql.md)
  - 20.1.1. Conception de la base de données
  - 20.1.2. Interface utilisateur
  - 20.1.3. Logique métier
  - 20.1.4. Rapports et exportation
  - 20.1.5. Déploiement et distribution

- 20.2. [Application CRUD avec WPF, MVVM et MariaDB](/20-projets-pratiques/20-2-application-crud-avec-wpf-mvvm-et-mariadb.md)
  - 20.2.1. Architecture MVVM
  - 20.2.2. Design de l'interface
  - 20.2.3. Services de données
  - 20.2.4. Tests automatisés
  - 20.2.5. Validation et gestion des erreurs

- 20.3. [Application Web complète avec ASP.NET Core](/20-projets-pratiques/20-3-application-web-complete-avec-asp-dotnet-core.md)
  - 20.3.1. Structure du projet
  - 20.3.2. API et frontend
  - 20.3.3. Authentification et autorisation
  - 20.3.4. Déploiement
  - 20.3.5. Documentation API avec Swagger

- 20.4. [API RESTful avec Entity Framework et Swagger](/20-projets-pratiques/20-4-api-restful-avec-entity-framework-et-swagger.md)
  - 20.4.1. Conception de l'API
  - 20.4.2. Modèles et migrations
  - 20.4.3. Controllers et endpoints
  - 20.4.4. Documentation avec Swagger/OpenAPI
  - 20.4.5. Tests et sécurité
  - 20.4.6. Déploiement en production

- 20.5. [Service Cloud avec Azure et .NET](/20-projets-pratiques/20-5-service-cloud-avec-azure-et-dotnet.md)
  - 20.5.1. Azure Functions
  - 20.5.2. Stockage et bases de données cloud
  - 20.5.3. Authentification et sécurité
  - 20.5.4. Monitoring et logging
  - 20.5.5. CI/CD pour le cloud

## 21. [Extensions et écosystème C#](/21-extensions-et-ecosysteme-csharp/README.md)
- 21.1. [Bibliothèques et frameworks populaires](/21-extensions-et-ecosysteme-csharp/21-1-bibliotheques-et-frameworks-populaires.md)
  - 21.1.1. Newtonsoft.Json et System.Text.Json
  - 21.1.2. AutoMapper
  - 21.1.3. FluentValidation
  - 21.1.4. MediatR
  - 21.1.5. Serilog et autres frameworks de logging

- 21.2. [DevOps pour projets C#](/21-extensions-et-ecosysteme-csharp/21-2-devops-pour-projets-csharp.md)
  - 21.2.1. CI/CD avec GitHub Actions
  - 21.2.2. Azure DevOps
  - 21.2.3. Tests automatisés en pipeline
  - 21.2.4. Déploiement continu
  - 21.2.5. Infrastructure as Code

- 21.3. [Tendances et évolutions futures de C#](/21-extensions-et-ecosysteme-csharp/21-3-tendances-et-evolutions-futures-de-csharp.md)
  - 21.3.1. Caractéristiques des versions récentes
  - 21.3.2. Roadmap C# et .NET
  - 21.3.3. Source generators
  - 21.3.4. C# sur WebAssembly
  - 21.3.5. Intelligence artificielle avec ML.NET

## 22. [Ressources et communauté](/22-ressources-et-communaute/README.md)
- 22.1. [Documentation officielle](/22-ressources-et-communaute/22-1-documentation-officielle.md)
- 22.2. [Communautés en ligne](/22-ressources-et-communaute/22-2-communautes-en-ligne.md)
- 22.3. [Conférences et événements](/22-ressources-et-communaute/22-3-conferences-et-evenements.md)
- 22.4. [Formation continue et certifications](/22-ressources-et-communaute/22-4-formation-continue-et-certifications.md)
- 22.5. [Blogs et podcasts sur C# et .NET](/22-ressources-et-communaute/22-5-blogs-et-podcasts-sur-csharp-et-dotnet.md)
- 22.6. [Contribution open source](/22-ressources-et-communaute/22-6-contribution-open-source.md)

