# 13.3. WinUI 3

🔝 Retour au [Sommaire](/SOMMAIRE.md)

## 13.3.1. Comparaison avec WPF

### Qu'est-ce que WinUI 3 ?

WinUI 3 (Windows UI Library 3) est la dernière génération de framework d'interface utilisateur native de Microsoft pour les applications Windows. Il fait partie du Windows App SDK (anciennement Project Reunion) et représente l'évolution des technologies d'interfaces Windows.

Contrairement à ses prédécesseurs, WinUI 3 est entièrement découplé du système d'exploitation Windows, ce qui offre de nombreux avantages que nous verrons plus loin.

### Principales différences avec WPF

**Tableau comparatif WinUI 3 vs WPF :**

| **Caractéristique** | **WinUI 3** | **WPF** |
|--|--|--|
| **Année de création** | 2021 (v1.0) | 2006 |
| **Langage de conception** | Fluent Design System | Aero Design (à l'origine) |
| **Type de framework** | Natif, basé sur C++ | Managé, basé sur .NET |
| **Système de rendu** | DirectX (via XAML Islands) | DirectX (via milcore.dll) |
| **Compatibilité Windows** | Windows 10 v1809+ et Windows 11 | Windows 7+ |
| **Conception XAML** | Déclaratif, édition manuelle du code | Déclaratif avec designer WYSIWYG |
| **Cycle de mises à jour** | Indépendant de Windows | Lié aux mises à jour de .NET |
| **Performances** | Optimisées (composants natifs C++) | Bonnes, mais overhead .NET |
| **Maturité de l'écosystème** | En développement | Mature, large écosystème |

### Avantages de WinUI 3 par rapport à WPF

1. **Interface moderne** : WinUI 3 intègre nativement le Fluent Design System, offrant une apparence moderne et cohérente avec les dernières directives de design Microsoft.

2. **Indépendance des versions de Windows** : Les applications WinUI 3 peuvent profiter des dernières fonctionnalités sans attendre les mises à jour de Windows, car la bibliothèque est découplée du système d'exploitation.

3. **Meilleures performances** : Étant basé sur des composants natifs C++, WinUI 3 offre généralement de meilleures performances, particulièrement pour le rendu graphique et les animations.

4. **Support étendu des fonctionnalités modernes** : Meilleure prise en charge des fonctionnalités modernes comme le mode sombre, les contrôles adaptés aux écrans tactiles, et une meilleure accessibilité.

5. **Compatibilité double** : Possibilité de créer à la fois des applications packagées (MSIX) pour le Microsoft Store et des applications desktop classiques (Win32).

### Inconvénients de WinUI 3 par rapport à WPF

1. **Maturité** : WPF existe depuis 2006 et dispose d'un large écosystème de contrôles, bibliothèques et ressources. WinUI 3 est plus récent et son écosystème est en développement.

2. **Outils de design** : WPF offre un designer WYSIWYG (What You See Is What You Get) dans Visual Studio, alors que WinUI 3 nécessite principalement l'édition manuelle du XAML.

3. **Compatibilité** : WPF fonctionne sur Windows 7 et versions ultérieures, tandis que WinUI 3 nécessite au minimum Windows 10 version 1809.

4. **Courbe d'apprentissage** : Pour les développeurs habitués à WPF, la transition peut nécessiter d'apprendre de nouvelles API et paradigmes.

### Architecture comparée

**WPF** utilise une architecture à couches :
- Couche Présentation (XAML, contrôles)
- Couche Core (layout, propriétés de dépendance)
- Media Integration Layer (rendu via DirectX)
- .NET Framework/Core (exécution managée)

**WinUI 3** utilise une architecture différente :
- Couche XAML (similaire à WPF/UWP)
- WinUI Library (contrôles et styles)
- Windows App SDK (APIs unifiées)
- Système natif Windows (exécution native)

## 13.3.2. Fluent Design System

### Principes fondamentaux

Le Fluent Design System est le langage de design moderne de Microsoft, qui met l'accent sur cinq principes clés :

1. **Lumière** : Utilisation d'effets lumineux pour guider l'attention de l'utilisateur
2. **Profondeur** : Création d'une hiérarchie spatiale pour organiser les éléments
3. **Mouvement** : Animations fluides et naturelles pour faciliter la compréhension
4. **Matériel** : Textures et effets visuels qui rappellent des matériaux réels
5. **Échelle** : Interface réactive qui s'adapte à différents appareils et contextes

WinUI 3 est conçu dès le départ pour implémenter ces principes de manière native et cohérente.

### Composants visuels clés

#### Acrylic (effet de flou)

L'effet Acrylic est une texture translucide avec un flou gaussien qui laisse transparaître le contenu en arrière-plan, créant un effet de profondeur.

```xml
<!-- Exemple d'application de l'effet Acrylic -->
<Grid>
    <Grid.Background>
        <AcrylicBrush TintColor="SteelBlue"
                      TintOpacity="0.7"
                      BackgroundSource="HostBackdrop" />
    </Grid.Background>
    <TextBlock Text="Texte avec fond Acrylic"
               Foreground="White"
               Margin="20" />
</Grid>
```

#### Reveal Highlight (effet de surbrillance)

Reveal est un effet de surbrillance qui réagit au passage de la souris ou au toucher, révélant les contours et la structure des éléments interactifs.

```xml
<!-- Exemple de bouton avec effet Reveal -->
<Button Content="Bouton avec Reveal"
        Background="{ThemeResource SystemControlBackgroundBaseLowBrush}"
        BorderBrush="{ThemeResource SystemControlForegroundBaseHighBrush}"
        BorderThickness="1" />
```

#### Shadow (ombres)

Les ombres aident à créer une hiérarchie visuelle en montrant la relation entre les éléments et leur élévation par rapport à la surface.

```xml
<!-- Exemple d'élément avec ombre -->
<Grid>
    <Grid.Shadow>
        <ThemeShadow />
    </Grid.Shadow>
    <TextBlock Text="Élément avec ombre" Padding="20" />
</Grid>
```

### Icônes et typographie

WinUI 3 utilise la police Segoe UI Variable, qui s'adapte automatiquement à différentes tailles d'écran et résolutions. Elle intègre également les Fluent System Icons, une collection d'icônes modernes cohérentes avec le Fluent Design.

```xml
<!-- Exemple d'utilisation d'icônes Fluent -->
<StackPanel Orientation="Horizontal" Spacing="10">
    <FontIcon FontFamily="Segoe Fluent Icons" Glyph="&#xE72D;" />
    <TextBlock Text="Paramètres"
               FontFamily="Segoe UI Variable Display"
               FontWeight="SemiBold" />
</StackPanel>
```

### Thèmes clairs et sombres

WinUI 3 supporte nativement les thèmes clair et sombre, avec une transition fluide entre les deux modes. Les applications peuvent réagir automatiquement aux préférences système ou proposer une option manuelle.

```xml
<!-- Exemple de styles adaptés au thème -->
<StackPanel>
    <TextBlock Text="Texte adapté au thème"
               Foreground="{ThemeResource SystemControlForegroundBaseHighBrush}" />
    <Rectangle Height="2" Margin="0,10"
               Fill="{ThemeResource SystemControlForegroundAccentBrush}" />
</StackPanel>
```

## 13.3.3. Migration depuis UWP/WPF

### Stratégies de migration

La migration d'applications existantes vers WinUI 3 peut suivre différentes approches selon le type d'application d'origine.

#### Migration depuis UWP

La migration depuis UWP est relativement simple car UWP et WinUI 3 partagent le même modèle XAML et de nombreuses API. Voici les étapes principales :

1. **Créer un nouveau projet WinUI 3**
2. **Copier le code XAML et C#** de l'application UWP
3. **Mettre à jour les espaces de noms** (remplacer `Windows.UI.Xaml` par `Microsoft.UI.Xaml`)
4. **Adapter les fonctionnalités spécifiques à UWP** (certaines API peuvent différer)
5. **Tester et optimiser l'application**

#### Migration depuis WPF

La migration depuis WPF est plus complexe car les frameworks ont des différences significatives :

1. **Approche incrémentale** : Commencer par moderniser les composants individuels
2. **XAML Islands** : Intégrer des contrôles WinUI dans l'application WPF existante
3. **Refactorisation par modules** : Migrer progressivement les fonctionnalités
4. **Réécriture complète** : Dans certains cas, il peut être préférable de réécrire l'application

### Différences de code et d'API

#### Espaces de noms

**WPF :**
```csharp
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
```

**WinUI 3 :**
```csharp
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;
using Microsoft.UI.Xaml.Media;
```

#### Gestion des fenêtres

**WPF :**
```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        this.Title = "Application WPF";
        this.Width = 800;
        this.Height = 600;
    }
}
```

**WinUI 3 :**
```csharp
public sealed partial class MainWindow : Window
{
    public MainWindow()
    {
        this.InitializeComponent();
        this.Title = "Application WinUI 3";

        // Les propriétés de la fenêtre sont gérées différemment
        // AppWindow est le nouveau système de gestion de fenêtres
        this.AppWindow.Resize(new Windows.Graphics.SizeInt32(800, 600));
    }
}
```

#### Styles et ressources

**WPF :**
```xml
<Application.Resources>
    <ResourceDictionary>
        <SolidColorBrush x:Key="PrimaryBrush" Color="#0078D7"/>
        <Style TargetType="Button">
            <Setter Property="Background" Value="{StaticResource PrimaryBrush}"/>
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="Padding" Value="10,5"/>
        </Style>
    </ResourceDictionary>
</Application.Resources>
```

**WinUI 3 :**
```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="PrimaryBrush" Color="#0078D7"/>
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <SolidColorBrush x:Key="PrimaryBrush" Color="#4CC2FF"/>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
        <Style TargetType="Button">
            <Setter Property="Background" Value="{ThemeResource PrimaryBrush}"/>
            <Setter Property="Foreground" Value="{ThemeResource SystemControlForegroundBaseHighBrush}"/>
            <Setter Property="Padding" Value="12,6"/>
        </Style>
    </ResourceDictionary>
</Application.Resources>
```

### Outils de migration

Quelques outils peuvent faciliter la migration vers WinUI 3 :

1. **XAML Modernizer** : Outil qui analyse et convertit le XAML de WPF vers WinUI
2. **.NET Upgrade Assistant** : Outil Microsoft qui aide à migrer vers les nouvelles versions de .NET
3. **Windows Community Toolkit** : Fournit des composants compatibles avec WinUI 3

### Considérations et défis

1. **Cycle de vie de l'application** : Le modèle de cycle de vie diffère entre WPF et WinUI 3
2. **Gestion des threads** : WinUI 3 utilise un modèle de thread différent
3. **Packaging et déploiement** : Options de déploiement différentes (MSIX vs. Installateurs classiques)
4. **Tests et débogage** : Nécessité d'adapter les stratégies de test
5. **Dépendances tierces** : Vérifier la compatibilité des bibliothèques tierces

## 13.3.4. Composition et Effets

### API Windows.UI.Composition

WinUI 3 permet d'accéder à l'API de composition Windows, qui offre un contrôle précis sur les effets visuels et les animations. Cette API permet de créer des expériences visuelles riches avec des performances optimales.

```csharp
// Exemple d'utilisation de l'API de composition
private void CreateCompositionEffect()
{
    // Obtenir le Compositor
    var compositor = this.Content.GetVisual().Compositor;

    // Créer un effet de flou gaussien
    var blurEffect = new Windows.UI.Composition.Effects.GaussianBlurEffect
    {
        BlurAmount = 10.0f,
        BorderMode = EffectBorderMode.Hard,
        Source = new CompositionEffectSourceParameter("source")
    };

    // Créer une fabrique d'effets à partir de l'effet de flou
    var effectFactory = compositor.CreateEffectFactory(blurEffect);

    // Créer un pinceau basé sur l'effet
    var effectBrush = effectFactory.CreateBrush();

    // Configurer le pinceau pour utiliser BackdropBrush comme source
    effectBrush.SetSourceParameter("source", compositor.CreateBackdropBrush());

    // Appliquer à un SpriteVisual
    var visual = compositor.CreateSpriteVisual();
    visual.Size = new System.Numerics.Vector2(300, 300);
    visual.Brush = effectBrush;

    // Ajouter le visual à l'arbre visuel
    ElementCompositionPreview.SetElementChildVisual(targetElement, visual);
}
```

### Animations basées sur la composition

Les animations basées sur la composition s'exécutent directement sur le GPU, offrant des performances supérieures aux animations traditionnelles basées sur le processeur.

```csharp
private void CreateCompositionAnimation()
{
    var visual = ElementCompositionPreview.GetElementVisual(myElement);
    var compositor = visual.Compositor;

    // Créer une animation de position
    var offsetAnimation = compositor.CreateVector3KeyFrameAnimation();
    offsetAnimation.InsertKeyFrame(0.0f, new Vector3(0, 0, 0));
    offsetAnimation.InsertKeyFrame(1.0f, new Vector3(100, 0, 0));
    offsetAnimation.Duration = TimeSpan.FromSeconds(1);
    offsetAnimation.IterationBehavior = AnimationIterationBehavior.Forever;
    offsetAnimation.Direction = AnimationDirection.Alternate;

    // Démarrer l'animation
    visual.StartAnimation("Offset", offsetAnimation);
}
```

### Effets visuels avancés

#### Effets d'ombre

```csharp
private void ApplyShadowEffect()
{
    var visual = ElementCompositionPreview.GetElementVisual(targetElement);
    var compositor = visual.Compositor;

    // Créer un effet d'ombre
    var dropShadow = compositor.CreateDropShadow();
    dropShadow.Color = Colors.Black;
    dropShadow.Opacity = 0.7f;
    dropShadow.BlurRadius = 15.0f;
    dropShadow.Offset = new Vector3(5, 5, 0);

    // Créer un SpriteVisual avec l'ombre
    var shadowVisual = compositor.CreateSpriteVisual();
    shadowVisual.Size = new Vector2((float)targetElement.ActualWidth, (float)targetElement.ActualHeight);
    shadowVisual.Shadow = dropShadow;

    // Appliquer l'ombre
    ElementCompositionPreview.SetElementChildVisual(shadowHost, shadowVisual);
}
```

#### Effets de flou et d'opacité

```csharp
private void ApplyBlurEffect()
{
    // Créer un effet de flou gaussien
    var blurEffect = new GaussianBlurEffect
    {
        Name = "Blur",
        BlurAmount = 10.0f,
        BorderMode = EffectBorderMode.Hard,
        Source = new CompositionEffectSourceParameter("Source")
    };

    // Créer une chaîne d'effets
    var effectChain = new CompositionEffectFactory(blurEffect);
    var brush = effectChain.CreateBrush();

    // Configurer la source
    brush.SetSourceParameter("Source", compositor.CreateBackdropBrush());

    // Appliquer à un SpriteVisual
    var visual = compositor.CreateSpriteVisual();
    visual.Size = new Vector2((float)blurPanel.ActualWidth, (float)blurPanel.ActualHeight);
    visual.Brush = brush;

    // Ajouter le visuel
    ElementCompositionPreview.SetElementChildVisual(blurPanel, visual);
}
```

### Interopérabilité avec Direct2D et DirectWrite

WinUI 3 permet également d'interagir avec les API graphiques de bas niveau comme Direct2D et DirectWrite pour des cas d'utilisation spécifiques nécessitant des performances optimales ou des effets personnalisés.

```csharp
// Exemple simplifié d'interopérabilité avec Direct2D
private void InitializeDirect2D()
{
    // Créer une fabrique Direct2D
    var d2dFactory = new SharpDX.Direct2D1.Factory();

    // Créer une surface de dessin
    var renderTarget = new SharpDX.Direct2D1.WindowRenderTarget(
        d2dFactory,
        new SharpDX.Direct2D1.RenderTargetProperties(),
        new SharpDX.Direct2D1.HwndRenderTargetProperties
        {
            Hwnd = WindowHandle,
            PixelSize = new SharpDX.Size2(Width, Height),
            PresentOptions = SharpDX.Direct2D1.PresentOptions.None
        });

    // Créer un pinceau
    var brush = new SharpDX.Direct2D1.SolidColorBrush(
        renderTarget,
        new SharpDX.Color4(1.0f, 0.0f, 0.0f, 1.0f));

    // Dessiner sur la surface
    renderTarget.BeginDraw();
    renderTarget.Clear(new SharpDX.Color4(0.0f, 0.0f, 0.0f, 0.0f));
    renderTarget.DrawEllipse(
        new SharpDX.Direct2D1.Ellipse(
            new SharpDX.Vector2(Width / 2, Height / 2),
            Width / 4,
            Height / 4),
        brush,
        2.0f);
    renderTarget.EndDraw();
}
```

### Applications pratiques

#### Parallaxe et effets de défilement

```csharp
private void SetupParallaxEffect()
{
    // Obtenir les éléments visuels
    var backgroundVisual = ElementCompositionPreview.GetElementVisual(backgroundElement);
    var foregroundVisual = ElementCompositionPreview.GetElementVisual(foregroundElement);
    var compositor = backgroundVisual.Compositor;

    // Configurer les propriétés initiales
    backgroundVisual.CenterPoint = new Vector3(
        (float)backgroundElement.ActualWidth / 2,
        (float)backgroundElement.ActualHeight / 2,
        0);

    // Créer une expression d'animation
    var scrollPosExpr = compositor.CreateExpressionAnimation("scroll.Position.Y * 0.2");
    scrollPosExpr.SetReferenceParameter("scroll", scrollViewer);

    // Associer l'animation à la propriété d'offset
    backgroundVisual.StartAnimation("Offset.Y", scrollPosExpr);
}
```

#### Transitions entre les pages

```csharp
private void SetupPageTransition()
{
    // Configurer une animation de transition de page
    var compositor = Window.Current.Compositor;

    // Animation pour la page entrante
    var enterAnimation = compositor.CreateScalarKeyFrameAnimation();
    enterAnimation.InsertKeyFrame(0, 0);
    enterAnimation.InsertKeyFrame(1, 1);
    enterAnimation.Duration = TimeSpan.FromMilliseconds(500);

    // Animation pour la page sortante
    var exitAnimation = compositor.CreateScalarKeyFrameAnimation();
    exitAnimation.InsertKeyFrame(0, 1);
    exitAnimation.InsertKeyFrame(1, 0);
    exitAnimation.Duration = TimeSpan.FromMilliseconds(500);

    // Créer la transition
    var transition = new EntranceThemeTransition();

    // Appliquer la transition
    PageTransitions.Add(transition);
}
```

### Optimisation des performances

Pour garantir les meilleures performances avec les API de composition :

1. **Minimiser les changements de contexte** entre le CPU et le GPU
2. **Réutiliser les objets de composition** plutôt que de les recréer
3. **Éviter les animations sur le thread UI principal**
4. **Optimiser les calculs de layout** pour éviter les recalculs fréquents
5. **Utiliser des techniques de mise en cache** pour les visuels complexes

---

WinUI 3 représente une évolution significative dans le développement d'applications Windows, offrant des performances natives et une expérience visuelle moderne. Bien que la migration depuis des frameworks plus anciens comme WPF puisse présenter des défis, les avantages en termes de performances, de design et d'évolutivité en font une option attrayante pour les nouvelles applications et la modernisation d'applications existantes.

⏭️
