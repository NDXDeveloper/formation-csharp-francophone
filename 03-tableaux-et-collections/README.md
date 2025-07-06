# 3. Tableaux et collections

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les données sont le cœur de toute application. Savoir les organiser, les stocker et les manipuler efficacement fait la différence entre une application basique et un logiciel professionnel performant. C# excelle dans ce domaine grâce à son écosystème riche et varié de structures de données.

## 🎯 Pourquoi maîtriser les collections ?

**Impact critique sur les performances** : Le choix d'une structure de données inappropriée peut ralentir votre application de façon dramatique.

**Lisibilité du code** : Les bonnes collections rendent votre code plus expressif et maintenable.

**Évolutivité** : Des structures bien choisies facilitent l'ajout de nouvelles fonctionnalités.

**Consommation mémoire** : Une gestion optimisée des collections peut réduire significativement l'empreinte mémoire.

## 🗺️ Votre arsenal de structures de données

### 📊 **Tableaux : la fondation solide**
Les structures les plus fondamentales et performantes :

**🎯 Tableaux unidimensionnels**
- Accès ultra-rapide par indice
- Taille fixe déterminée à la création
- Idéal pour les données de taille connue

**🎯 Tableaux multidimensionnels**
- Matrices et structures 2D/3D
- Parfait pour les calculs scientifiques
- Représentation naturelle des données spatiales

**🎯 Tableaux en dents de scie**
- Tableaux de tableaux de tailles différentes
- Flexibilité maximale pour les structures irrégulières
- Optimisation mémoire pour les données éparses

### 📚 **Collections génériques : flexibilité et puissance**
Les outils modernes pour la plupart de vos besoins :

**🔧 `List<T>` - Le couteau suisse**
- Redimensionnement automatique
- Insertion/suppression efficace
- Idéal pour 90% des cas d'usage

**🗂️ `Dictionary<TKey, TValue>` - Recherche instantanée**
- Accès O(1) par clé
- Parfait pour les mappages et caches
- Incontournable pour les données associatives

**🎯 `HashSet<T>` - Collections uniques**
- Élimination automatique des doublons
- Tests d'appartenance ultra-rapides
- Opérations ensemblistes (union, intersection)

**📋 `Queue<T>` et `Stack<T>` - Structures spécialisées**
- File d'attente (FIFO) et pile (LIFO)
- Algorithmes et gestion d'états
- Traitement séquentiel optimisé

### 🚀 **Collections spécialisées : cas d'usage avancés**

**⚡ Collections concurrentes**
- Thread-safe par conception
- `ConcurrentDictionary<T>`, `ConcurrentQueue<T>`
- Applications multi-threading sécurisées

**🔒 Collections immuables**
- Immuabilité garantie
- Programmation fonctionnelle
- Sécurité et prévisibilité accrues

**👁️ Collections observables**
- Notifications de changements
- Interfaces utilisateur réactives
- Binding automatique avec WPF/MAUI

## 🎨 **Nouveau dans .NET 8 vs .NET Framework 4.7.2**

**🆕 .NET 8 - Les nouveautés**
- **Span<T>** et **Memory<T>** : manipulation haute performance
- **Collection expressions** : syntaxe simplifiée
- **Pattern matching** sur les collections
- **Performances** considérablement améliorées

**🏛️ .NET Framework 4.7.2 - Les classiques**
- Collections génériques éprouvées
- Compatibilité maximale
- Fonctionnalités stables et documentées

## 💡 **Guide de sélection : la bonne collection pour le bon usage**

### 🎯 **Scénarios courants**

**📊 Données de taille connue et fixe**
→ **Tableaux** pour les performances maximales

**📈 Collection dynamique simple**
→ **List<T>** pour la polyvalence

**🔍 Recherche fréquente par clé**
→ **Dictionary<TKey, TValue>** pour la rapidité

**🎯 Élimination des doublons**
→ **HashSet<T>** pour l'unicité

**🔄 Traitement séquentiel**
→ **Queue<T>** ou **Stack<T>** selon le besoin

**⚡ Applications multi-threading**
→ **Collections concurrentes** pour la sécurité

## 🎯 **Ce que vous allez maîtriser**

À la fin de ce chapitre, vous serez expert en :
- ✅ **Choix optimal** de la structure de données
- ✅ **Performances** : éviter les pièges courants
- ✅ **Manipulation** : CRUD efficace sur toutes les collections
- ✅ **Itération** : `foreach`, LINQ et méthodes modernes
- ✅ **Initialisation** : syntaxes concises et expressives
- ✅ **Conversion** : passage fluide entre collections
- ✅ **Bonnes pratiques** : code maintenable et performant

## 🚀 **Méthode d'apprentissage**

**Exemples concrets** : Chaque collection illustrée avec des cas d'usage réels.

**Comparaisons pratiques** : Tableaux de performance et critères de choix.

**Pièges à éviter** : Erreurs courantes et solutions.

**Évolution du code** : Migration des pratiques anciennes vers les modernes.

## 🏆 **Objectif : développeur efficace**

Que vous construisiez une API REST, une application desktop, un jeu ou une solution IoT, ces structures de données sont vos outils quotidiens. Les maîtriser vous permettra d'écrire du code :
- **Performant** : choix optimaux pour chaque situation
- **Lisible** : intention claire et code expressif
- **Maintenable** : évolution facile et débogage simplifié
- **Professionnel** : respect des standards et bonnes pratiques

---

⏭️ **Commençons par la base** : [3.1. Tableaux](/03-tableaux-et-collections/3-1-tableaux.md)
