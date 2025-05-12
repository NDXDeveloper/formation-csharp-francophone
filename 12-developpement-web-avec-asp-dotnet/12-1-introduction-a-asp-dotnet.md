# 12.1. Introduction à ASP.NET

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 12.1.1. Historique et évolution

ASP.NET est une technologie de développement web créée par Microsoft qui a considérablement évolué depuis sa première apparition.

### Les origines

ASP (Active Server Pages) a été la première technologie de pages web dynamiques de Microsoft, lancée en 1996. C'était essentiellement un environnement de script côté serveur qui permettait d'insérer du code VBScript ou JScript directement dans les pages HTML.

### La naissance d'ASP.NET

En 2002, Microsoft a lancé ASP.NET comme successeur d'ASP, dans le cadre de l'initiative .NET. Contrairement à son prédécesseur, ASP.NET était une plateforme complète de développement web qui s'appuyait sur le Common Language Runtime (CLR), permettant aux développeurs d'écrire du code en utilisant n'importe quel langage compatible .NET comme C#, VB.NET, ou même J#.

### Évolution chronologique

- **ASP.NET 1.0 (2002)** : Première version, introduisant le modèle WebForms.
- **ASP.NET 2.0 (2005)** : Améliorations majeures avec les Master Pages, les thèmes, et les contrôles de données.
- **ASP.NET 3.5 (2007)** : Intégration de LINQ et de l'AJAX.
- **ASP.NET 4.0 (2010)** : Introduction du routage URL et des modèles de page plus légers.
- **ASP.NET MVC (2009-2012)** : Émergence du pattern Model-View-Controller comme alternative à WebForms.
- **ASP.NET Web API (2012)** : Framework dédié à la création d'API RESTful.
- **ASP.NET SignalR (2013)** : Bibliothèque pour les communications en temps réel.
- **ASP.NET Core 1.0 (2016)** : Réécriture complète, multiplateforme et open-source.
- **ASP.NET Core 2.0-3.1** : Améliorations de performances et nouvelles fonctionnalités.
- **ASP.NET Core 5.0+ (2020)** : Unification avec .NET 5 et versions ultérieures.

## 12.1.2. Frameworks ASP.NET

ASP.NET propose plusieurs frameworks qui correspondent à différents styles de développement web. Comprendre leurs différences est crucial pour choisir celui qui convient le mieux à votre projet.

### ASP.NET WebForms

Le modèle WebForms a été conçu pour faciliter la transition des développeurs d'applications Windows vers le web.

**Caractéristiques principales :**
- Modèle basé sur les contrôles et les événements
- Cycle de vie de page avec postbacks
- État de vue pour maintenir l'état entre les requêtes
- Développement rapide avec glisser-déposer dans Visual Studio

**Idéal pour :**
- Les développeurs habitués à WinForms
- Les applications d'entreprise internes simples
- Projets nécessitant un développement rapide avec moins de code

### ASP.NET MVC

MVC sépare l'application en trois composants principaux : Modèle, Vue et Contrôleur.

**Caractéristiques principales :**
- Séparation claire des préoccupations
- Contrôle total sur le HTML généré
- Routage URL flexible
- Facilite les tests unitaires
- Support natif des techniques modernes de développement web

**Idéal pour :**
- Applications web complexes
- Équipes avec des spécialistes frontend et backend
- Projets nécessitant une grande testabilité

### ASP.NET Web API

Framework spécialisé dans la création d'API HTTP.

**Caractéristiques principales :**
- Conçu pour créer des services RESTful
- Support de différents formats (JSON, XML, etc.)
- Routage basé sur les conventions ou attributs
- Self-hosting possible

**Idéal pour :**
- Services backend pour applications mobiles ou SPA
- Intégrations système
- Microservices

### ASP.NET SignalR

Bibliothèque pour la communication en temps réel entre clients et serveurs.

**Caractéristiques principales :**
- Communication bidirectionnelle en temps réel
- Abstraction au-dessus de diverses techniques (WebSockets, Long Polling, etc.)
- Scaling pour plusieurs serveurs

**Idéal pour :**
- Chat en temps réel
- Tableaux de bord mis à jour en direct
- Jeux multijoueurs en ligne
- Notifications push

### Razor Pages

Introduit avec ASP.NET Core, ce modèle simplifie le développement en associant directement une page à son code-behind.

**Caractéristiques principales :**
- Modèle de page unique
- Plus simple que MVC pour les scénarios CRUD basiques
- Maintient la séparation des préoccupations

**Idéal pour :**
- Petites applications
- Pages avec peu de logique
- Développeurs débutants

### Blazor

La dernière évolution majeure permet d'écrire des applications web frontend en C# au lieu de JavaScript.

**Caractéristiques principales :**
- Utilise WebAssembly (Blazor WebAssembly) ou une connexion SignalR (Blazor Server)
- Réutilisation du code C# côté client et serveur
- Composants UI réutilisables

**Idéal pour :**
- Équipes avec expertise en C# voulant éviter JavaScript
- Applications web complexes côté client
- Partage de logique métier entre client et serveur

## 12.1.3. ASP.NET vs ASP.NET Core

ASP.NET Core représente une refonte majeure de la plateforme originale. Comprendre les différences est crucial pour décider quelle version utiliser.

### Principales différences

#### Architecture

**ASP.NET Framework traditionnel :**
- Étroitement lié à Windows et IIS
- Dépend du .NET Framework complet
- Architecture monolithique

**ASP.NET Core :**
- Multiplateforme (Windows, macOS, Linux)
- Modulaire, composé de packages NuGet
- Plus léger et optimisé pour le cloud
- Serveur web intégré (Kestrel) indépendant d'IIS

#### Performance

ASP.NET Core offre des performances nettement supérieures :
- Jusqu'à 10 fois plus rapide dans certains scénarios
- Conçu pour le cloud et les microservices
- Optimisé pour la scalabilité horizontale

#### Fonctionnalités modernes

ASP.NET Core apporte de nombreuses améliorations modernes :
- Support intégré pour la conteneurisation (Docker)
- Système de configuration unifié
- Journalisation intégrée
- Injection de dépendances native
- Environnements de développement, test et production
- Middleware configurable
- Déploiement côte à côte de différentes versions

#### Open Source

ASP.NET Core est entièrement open source :
- Développement transparent sur GitHub
- Accepte les contributions de la communauté
- Cycle de développement plus rapide

### Quand choisir quelle version ?

**Choisir ASP.NET Framework traditionnel si :**
- Vous maintenez une application existante
- Vous dépendez de bibliothèques .NET Framework spécifiques
- Vous avez besoin de fonctionnalités Windows spécifiques (COM, WCF, etc.)
- Vous êtes limité à l'infrastructure Windows

**Choisir ASP.NET Core si :**
- Vous démarrez un nouveau projet
- Vous visez une application multiplateforme
- Vous développez des microservices
- Vous ciblez des environnements cloud
- Vous recherchez les meilleures performances
- Vous voulez bénéficier des dernières fonctionnalités

### Compatibilité et migration

Microsoft fournit des outils pour aider à la migration des applications ASP.NET traditionnelles vers ASP.NET Core :

- Compatibilité via des packages spécifiques
- Possibilité de migration progressive
- Documentation détaillée pour la migration

Cependant, dans de nombreux cas, une réécriture partielle peut être nécessaire, en particulier pour les applications WebForms, car ce modèle n'existe pas dans ASP.NET Core.

### Le futur

Microsoft a clairement indiqué que ASP.NET Core est l'avenir de la plateforme. Toutes les nouvelles fonctionnalités importantes sont développées uniquement pour ASP.NET Core, tandis que l'ASP.NET Framework traditionnel ne reçoit que des mises à jour de maintenance et de sécurité.

À partir de .NET 5, Microsoft a unifié les plateformes sous un seul nom ".NET", sans le suffixe "Core", mais le nouveau framework conserve l'architecture et les avantages d'ASP.NET Core.

⏭️ 12.2. [ASP.NET MVC](/12-developpement-web-avec-asp-dotnet/12-2-asp-dotnet-mvc.md)
