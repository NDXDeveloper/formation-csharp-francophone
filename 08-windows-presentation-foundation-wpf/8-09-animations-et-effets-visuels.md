# 8.9. Animations et effets visuels

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les animations et effets visuels permettent de créer des interfaces utilisateur dynamiques et attrayantes. WPF offre un système d'animation puissant et flexible qui permet de modifier les propriétés des éléments au fil du temps, créant ainsi des transitions fluides et des interactions visuellement riches.

## 8.9.1. Animations basées sur le temps

Les animations basées sur le temps (ou animations à base de temps) modifient une propriété de façon progressive sur une période définie. WPF fournit différentes classes d'animation pour différents types de propriétés.

### Types d'animations courantes

- `DoubleAnimation` : Pour animer les propriétés de type `double` (Opacity, Width, Height, etc.)
- `ColorAnimation` : Pour animer les propriétés de type `Color`
- `PointAnimation` : Pour animer les propriétés de type `Point`
- `ThicknessAnimation` : Pour animer les propriétés de type `Thickness` (Margin, Padding)

### Animation simple avec DoubleAnimation

Voici comment faire une animation simple qui fait apparaître progressivement un élément :

```xml
<Button Content="Cliquez-moi" Width="100" Height="30">
    <Button.Triggers>
        <EventTrigger RoutedEvent="Button.Loaded">
            <BeginStoryboard>
                <Storyboard>
                    <!-- Animation de l'opacité de 0 à 1 sur 2 secondes -->
                    <DoubleAnimation
                        Storyboard.TargetProperty="Opacity"
                        From="0" To="1" Duration="0:0:2" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Button.Triggers>
</Button>
```


### Animation avec déclencheur d'événement

```xml
<Rectangle Width="100" Height="100" Fill="Blue" Name="MonRectangle">
    <Rectangle.Triggers>
        <EventTrigger RoutedEvent="Rectangle.MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <!-- Animation qui élargit le rectangle lorsque la souris passe dessus -->
                    <DoubleAnimation
                        Storyboard.TargetProperty="Width"
                        To="200" Duration="0:0:0.3" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
        <EventTrigger RoutedEvent="Rectangle.MouseLeave">
            <BeginStoryboard>
                <Storyboard>
                    <!-- Animation qui réduit le rectangle lorsque la souris le quitte -->
                    <DoubleAnimation
                        Storyboard.TargetProperty="Width"
                        To="100" Duration="0:0:0.3" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Rectangle.Triggers>
</Rectangle>
```


### Fonctions d'accélération (Easing)

Les fonctions d'accélération permettent de contrôler la vitesse de l'animation au fil du temps, ce qui donne un aspect plus naturel et moins mécanique :

```xml
<DoubleAnimation
    Storyboard.TargetProperty="Width"
    From="100" To="300" Duration="0:0:1">
    <DoubleAnimation.EasingFunction>
        <ElasticEase Oscillations="2" Springiness="3" EasingMode="EaseOut"/>
    </DoubleAnimation.EasingFunction>
</DoubleAnimation>
```


Types de fonctions d'accélération courantes :
- `QuadraticEase` : Accélération ou décélération quadratique
- `CubicEase` : Accélération ou décélération cubique
- `ElasticEase` : Effet rebondissant comme un ressort
- `BounceEase` : Effet de rebond comme une balle
- `BackEase` : Léger retour en arrière avant de progresser
- `CircleEase` : Mouvement en arc circulaire
- `SineEase` : Progression sinusoïdale

Chaque fonction d'accélération peut être configurée avec un `EasingMode` :
- `EaseIn` : L'animation commence lentement et accélère
- `EaseOut` : L'animation commence rapidement et ralentit
- `EaseInOut` : L'animation commence lentement, accélère, puis ralentit à la fin

### Animation de couleurs

```xml
<Ellipse Width="100" Height="100" Name="MonCercle">
    <Ellipse.Fill>
        <SolidColorBrush Color="Red" />
    </Ellipse.Fill>
    <Ellipse.Triggers>
        <EventTrigger RoutedEvent="Ellipse.MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <!-- Animation de la couleur de rouge à bleu -->
                    <ColorAnimation
                        Storyboard.TargetProperty="(Ellipse.Fill).(SolidColorBrush.Color)"
                        To="Blue" Duration="0:0:0.5" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
        <EventTrigger RoutedEvent="Ellipse.MouseLeave">
            <BeginStoryboard>
                <Storyboard>
                    <!-- Animation de retour à la couleur rouge -->
                    <ColorAnimation
                        Storyboard.TargetProperty="(Ellipse.Fill).(SolidColorBrush.Color)"
                        To="Red" Duration="0:0:0.5" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Ellipse.Triggers>
</Ellipse>
```


## 8.9.2. Animations basées sur les keyframes

Les animations à base de keyframes (images clés) vous permettent de définir plusieurs valeurs intermédiaires pour une propriété, offrant un contrôle plus précis sur l'animation.

### Animation avec DoubleAnimationUsingKeyFrames

```xml
<Button Content="Bouton animé" Width="100" Height="40">
    <Button.Triggers>
        <EventTrigger RoutedEvent="Button.Click">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimationUsingKeyFrames
                        Storyboard.TargetProperty="Width"
                        Duration="0:0:2">
                        <!-- Première image clé à 0.0 seconde -->
                        <LinearDoubleKeyFrame Value="100" KeyTime="0:0:0" />
                        <!-- Deuxième image clé à 0.5 seconde -->
                        <LinearDoubleKeyFrame Value="200" KeyTime="0:0:0.5" />
                        <!-- Troisième image clé à 1.0 seconde -->
                        <EasingDoubleKeyFrame Value="150" KeyTime="0:0:1">
                            <EasingDoubleKeyFrame.EasingFunction>
                                <BounceEase Bounces="2" EasingMode="EaseOut" />
                            </EasingDoubleKeyFrame.EasingFunction>
                        </EasingDoubleKeyFrame>
                        <!-- Dernière image clé à 2.0 secondes -->
                        <LinearDoubleKeyFrame Value="100" KeyTime="0:0:2" />
                    </DoubleAnimationUsingKeyFrames>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Button.Triggers>
</Button>
```


### Types d'images clés

1. **LinearKeyFrame** : Interpolation linéaire (constante) entre les valeurs
2. **DiscreteKeyFrame** : Pas d'interpolation, passage direct d'une valeur à l'autre
3. **SplineKeyFrame** : Interpolation suivant une courbe de Bézier définie par des points de contrôle
4. **EasingKeyFrame** : Interpolation utilisant une fonction d'accélération

### Animation complexe avec ColorAnimationUsingKeyFrames

```xml
<Rectangle Width="200" Height="100" Name="rectColor">
    <Rectangle.Fill>
        <SolidColorBrush Color="Yellow" />
    </Rectangle.Fill>
    <Rectangle.Triggers>
        <EventTrigger RoutedEvent="Rectangle.MouseDown">
            <BeginStoryboard>
                <Storyboard>
                    <ColorAnimationUsingKeyFrames
                        Storyboard.TargetProperty="(Rectangle.Fill).(SolidColorBrush.Color)"
                        Duration="0:0:3" RepeatBehavior="Forever">
                        <!-- Cycle de couleurs -->
                        <LinearColorKeyFrame Value="Red" KeyTime="0:0:0.5" />
                        <LinearColorKeyFrame Value="Green" KeyTime="0:0:1" />
                        <LinearColorKeyFrame Value="Blue" KeyTime="0:0:1.5" />
                        <LinearColorKeyFrame Value="Purple" KeyTime="0:0:2" />
                        <LinearColorKeyFrame Value="Orange" KeyTime="0:0:2.5" />
                        <LinearColorKeyFrame Value="Yellow" KeyTime="0:0:3" />
                    </ColorAnimationUsingKeyFrames>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Rectangle.Triggers>
</Rectangle>
```


## 8.9.3. Storyboards

Un Storyboard est un conteneur pour les animations qui vous permet de les regrouper, les contrôler et les synchroniser.

### Création et gestion d'un Storyboard en XAML

```xml
<Window x:Class="AnimationDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Animation Demo" Height="350" Width="525">
    <Window.Resources>
        <!-- Définition du Storyboard comme ressource -->
        <Storyboard x:Key="MonStoryboard">
            <DoubleAnimation
                Storyboard.TargetName="MonRectangle"
                Storyboard.TargetProperty="Width"
                From="100" To="300" Duration="0:0:1"
                AutoReverse="True" RepeatBehavior="2x" />
            <DoubleAnimation
                Storyboard.TargetName="MonRectangle"
                Storyboard.TargetProperty="Height"
                From="50" To="150" Duration="0:0:1.5"
                AutoReverse="True" RepeatBehavior="2x" />
        </Storyboard>
    </Window.Resources>

    <Grid>
        <Rectangle Name="MonRectangle" Fill="Blue" Width="100" Height="50" />

        <StackPanel VerticalAlignment="Bottom" Margin="10">
            <Button Content="Démarrer l'animation" Click="StartAnimation_Click" Margin="5" />
            <Button Content="Arrêter l'animation" Click="StopAnimation_Click" Margin="5" />
            <Button Content="Mettre en pause" Click="PauseAnimation_Click" Margin="5" />
            <Button Content="Reprendre" Click="ResumeAnimation_Click" Margin="5" />
        </StackPanel>
    </Grid>
</Window>
```


### Contrôle du Storyboard en code-behind

```textmate
public partial class MainWindow : Window
{
    private Storyboard _storyboard;

    public MainWindow()
    {
        InitializeComponent();

        // Récupérer le Storyboard défini dans les ressources
        _storyboard = (Storyboard)FindResource("MonStoryboard");
    }

    private void StartAnimation_Click(object sender, RoutedEventArgs e)
    {
        _storyboard.Begin(this);
    }

    private void StopAnimation_Click(object sender, RoutedEventArgs e)
    {
        _storyboard.Stop(this);
    }

    private void PauseAnimation_Click(object sender, RoutedEventArgs e)
    {
        _storyboard.Pause(this);
    }

    private void ResumeAnimation_Click(object sender, RoutedEventArgs e)
    {
        _storyboard.Resume(this);
    }
}
```


### Création de Storyboard en code

```textmate
private void CreateAndStartStoryboard()
{
    // Créer un nouveau Storyboard
    Storyboard storyboard = new Storyboard();

    // Créer une animation
    DoubleAnimation animation = new DoubleAnimation
    {
        From = 0,
        To = 360,
        Duration = new Duration(TimeSpan.FromSeconds(2)),
        RepeatBehavior = RepeatBehavior.Forever
    };

    // Définir la cible de l'animation
    Storyboard.SetTarget(animation, MonRectangle);
    Storyboard.SetTargetProperty(animation, new PropertyPath("(UIElement.RenderTransform).(RotateTransform.Angle)"));

    // S'assurer que le rectangle a une transformation de rotation
    if (MonRectangle.RenderTransform is not RotateTransform)
    {
        MonRectangle.RenderTransform = new RotateTransform();
    }

    // Ajouter l'animation au Storyboard
    storyboard.Children.Add(animation);

    // Démarrer le Storyboard
    storyboard.Begin(this);
}
```


### Événements de Storyboard

```textmate
private void ConfigureStoryboardEvents()
{
    Storyboard storyboard = (Storyboard)FindResource("MonStoryboard");

    // S'abonner aux événements
    storyboard.Completed += Storyboard_Completed;
    storyboard.CurrentTimeInvalidated += Storyboard_TimeChanged;

    storyboard.Begin(this, true); // Le second paramètre indique si le Storyboard doit être arrêté à la fin
}

private void Storyboard_Completed(object sender, EventArgs e)
{
    MessageBox.Show("Animation terminée !");
}

private void Storyboard_TimeChanged(object sender, EventArgs e)
{
    // Cet événement est déclenché à chaque mise à jour de l'horloge interne du Storyboard
    Storyboard storyboard = (Storyboard)sender;
    TimeSpan currentTime = storyboard.GetCurrentTime(this);

    // Vous pouvez utiliser le temps actuel pour synchroniser d'autres éléments
    statusText.Text = $"Temps écoulé : {currentTime.TotalSeconds:F2} secondes";
}
```


## 8.9.4. Effets et transformations

Les effets et transformations permettent de modifier l'apparence et la position des éléments.

### Transformations courantes

1. **TranslateTransform** : Déplace un élément horizontalement et/ou verticalement
2. **ScaleTransform** : Redimensionne un élément
3. **RotateTransform** : Fait pivoter un élément
4. **SkewTransform** : Incline un élément
5. **TransformGroup** : Combine plusieurs transformations
6. **MatrixTransform** : Transformation personnalisée à l'aide d'une matrice

### Exemple de transformations

```xml
<Grid>
    <Grid.Resources>
        <Storyboard x:Key="RotateAnimation">
            <DoubleAnimation
                Storyboard.TargetName="myRotateTransform"
                Storyboard.TargetProperty="Angle"
                From="0" To="360" Duration="0:0:2"
                RepeatBehavior="Forever" />
        </Storyboard>
    </Grid.Resources>

    <Button Content="Transformation" Width="100" Height="40" Margin="20">
        <Button.RenderTransform>
            <TransformGroup>
                <ScaleTransform x:Name="myScaleTransform" ScaleX="1" ScaleY="1" />
                <RotateTransform x:Name="myRotateTransform" Angle="0" />
                <TranslateTransform x:Name="myTranslateTransform" X="0" Y="0" />
            </TransformGroup>
        </Button.RenderTransform>
        <Button.Triggers>
            <EventTrigger RoutedEvent="Button.MouseEnter">
                <BeginStoryboard>
                    <Storyboard>
                        <DoubleAnimation
                            Storyboard.TargetName="myScaleTransform"
                            Storyboard.TargetProperty="ScaleX"
                            To="1.5" Duration="0:0:0.2" />
                        <DoubleAnimation
                            Storyboard.TargetName="myScaleTransform"
                            Storyboard.TargetProperty="ScaleY"
                            To="1.5" Duration="0:0:0.2" />
                    </Storyboard>
                </BeginStoryboard>
            </EventTrigger>
            <EventTrigger RoutedEvent="Button.MouseLeave">
                <BeginStoryboard>
                    <Storyboard>
                        <DoubleAnimation
                            Storyboard.TargetName="myScaleTransform"
                            Storyboard.TargetProperty="ScaleX"
                            To="1" Duration="0:0:0.2" />
                        <DoubleAnimation
                            Storyboard.TargetName="myScaleTransform"
                            Storyboard.TargetProperty="ScaleY"
                            To="1" Duration="0:0:0.2" />
                    </Storyboard>
                </BeginStoryboard>
            </EventTrigger>
            <EventTrigger RoutedEvent="Button.Loaded">
                <BeginStoryboard Storyboard="{StaticResource RotateAnimation}" />
            </EventTrigger>
        </Button.Triggers>
    </Button>
</Grid>
```


### Point de centre de transformation

Par défaut, les transformations s'appliquent par rapport au point d'origine (0,0) de l'élément. Vous pouvez modifier ce comportement en définissant les propriétés `CenterX` et `CenterY` :

```xml
<Rectangle Width="100" Height="50" Fill="Blue">
    <Rectangle.RenderTransform>
        <RotateTransform Angle="45" CenterX="50" CenterY="25" />
    </Rectangle.RenderTransform>
</Rectangle>
```


### Effets visuels

WPF propose plusieurs effets visuels qui peuvent être appliqués aux éléments :

1. **BlurEffect** : Effet de flou
2. **DropShadowEffect** : Ombre portée
3. **BevelEffect** : Effet de biseautage (uniquement dans .NET Framework)
4. **EmBossEffect** : Effet de relief (uniquement dans .NET Framework)

```xml
<Button Content="Bouton avec effets" Width="150" Height="50">
    <Button.Effect>
        <DropShadowEffect Color="Black" Direction="320"
                          ShadowDepth="5" Opacity="0.5" BlurRadius="5" />
    </Button.Effect>
    <Button.Triggers>
        <EventTrigger RoutedEvent="Button.MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation
                        Storyboard.TargetProperty="(Button.Effect).(DropShadowEffect.BlurRadius)"
                        To="15" Duration="0:0:0.2" />
                    <DoubleAnimation
                        Storyboard.TargetProperty="(Button.Effect).(DropShadowEffect.ShadowDepth)"
                        To="10" Duration="0:0:0.2" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
        <EventTrigger RoutedEvent="Button.MouseLeave">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation
                        Storyboard.TargetProperty="(Button.Effect).(DropShadowEffect.BlurRadius)"
                        To="5" Duration="0:0:0.2" />
                    <DoubleAnimation
                        Storyboard.TargetProperty="(Button.Effect).(DropShadowEffect.ShadowDepth)"
                        To="5" Duration="0:0:0.2" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Button.Triggers>
</Button>
```


### Animation d'effet de flou

```xml
<Border Width="200" Height="150" BorderBrush="Gray" BorderThickness="1" Margin="20">
    <Border.Effect>
        <BlurEffect Radius="0" x:Name="myBlurEffect" />
    </Border.Effect>
    <Border.Triggers>
        <EventTrigger RoutedEvent="Border.MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation
                        Storyboard.TargetName="myBlurEffect"
                        Storyboard.TargetProperty="Radius"
                        To="10" Duration="0:0:0.3" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
        <EventTrigger RoutedEvent="Border.MouseLeave">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation
                        Storyboard.TargetName="myBlurEffect"
                        Storyboard.TargetProperty="Radius"
                        To="0" Duration="0:0:0.3" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Border.Triggers>

    <Image Source="/Images/sample.jpg" Stretch="UniformToFill" />
</Border>
```


## 8.9.5. Transitions et comportements

Les transitions sont des animations qui se produisent lors de changements d'état ou de contenu.

### Animations de visibilité avec VisibilityAnimation

Créons d'abord une classe d'extension pour animer les changements de visibilité :

```textmate
// VisibilityAnimation.cs
public static class VisibilityAnimation
{
    // Propriété de pièce jointe pour indiquer si l'animation est activée
    public static readonly DependencyProperty AnimateVisibilityProperty =
        DependencyProperty.RegisterAttached(
            "AnimateVisibility",
            typeof(bool),
            typeof(VisibilityAnimation),
            new PropertyMetadata(false, OnAnimateVisibilityChanged));

    // Méthodes de pièce jointe pour lire/écrire la propriété
    public static bool GetAnimateVisibility(UIElement element)
    {
        return (bool)element.GetValue(AnimateVisibilityProperty);
    }

    public static void SetAnimateVisibility(UIElement element, bool value)
    {
        element.SetValue(AnimateVisibilityProperty, value);
    }

    // Gestionnaire d'événements pour les changements de la propriété AnimateVisibility
    private static void OnAnimateVisibilityChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is UIElement element)
        {
            if ((bool)e.NewValue)
            {
                // S'abonner aux changements de visibilité si l'animation est activée
                element.IsVisibleChanged += Element_IsVisibleChanged;
            }
            else
            {
                // Se désabonner des changements si l'animation est désactivée
                element.IsVisibleChanged -= Element_IsVisibleChanged;
            }
        }
    }

    // Gestionnaire pour les changements de visibilité
    private static void Element_IsVisibleChanged(object sender, DependencyPropertyChangedEventArgs e)
    {
        UIElement element = (UIElement)sender;

        if ((bool)e.NewValue) // L'élément devient visible
        {
            // Réinitialiser l'opacité à 0 pour l'animation
            element.Opacity = 0;

            // Créer et démarrer l'animation de fondu
            DoubleAnimation fadeIn = new DoubleAnimation
            {
                From = 0,
                To = 1,
                Duration = TimeSpan.FromMilliseconds(400)
            };

            element.BeginAnimation(UIElement.OpacityProperty, fadeIn);
        }
        else // L'élément devient invisible
        {
            // Ne rien faire ici, car l'élément est déjà caché
        }
    }
}
```


Utilisation de cette animation de visibilité :

```xml
<StackPanel>
    <CheckBox Content="Afficher le panneau" x:Name="showPanelCheckbox" Margin="10" />

    <Border Background="LightBlue" Padding="20" Margin="10"
            Visibility="{Binding IsChecked, ElementName=showPanelCheckbox, Converter={StaticResource BooleanToVisibilityConverter}}"
            local:VisibilityAnimation.AnimateVisibility="True">
        <TextBlock Text="Ce panneau apparaît et disparaît avec une animation" />
    </Border>
</StackPanel>
```


### Animations de transition de page

Voici comment créer une animation de transition entre deux contenus :

```xml
<Grid>
    <Grid.Resources>
        <Storyboard x:Key="PageTransition">
            <DoubleAnimation
                Storyboard.TargetName="content"
                Storyboard.TargetProperty="Opacity"
                From="0" To="1" Duration="0:0:0.3" />
            <ThicknessAnimation
                Storyboard.TargetName="content"
                Storyboard.TargetProperty="Margin"
                From="50,0,0,0" To="0,0,0,0" Duration="0:0:0.3">
                <ThicknessAnimation.EasingFunction>
                    <QuadraticEase EasingMode="EaseOut" />
                </ThicknessAnimation.EasingFunction>
            </ThicknessAnimation>
        </Storyboard>
    </Grid.Resources>

    <ContentControl x:Name="content" Content="{Binding CurrentPage}" Opacity="1" Margin="0">
        <ContentControl.ContentTemplate>
            <DataTemplate>
                <Grid>
                    <TextBlock Text="{Binding Title}" FontSize="24" Margin="0,0,0,20" />
                    <ContentPresenter Content="{Binding Content}" Margin="0,40,0,0" />
                </Grid>
            </DataTemplate>
        </ContentControl.ContentTemplate>
    </ContentControl>

    <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom" HorizontalAlignment="Center" Margin="0,0,0,20">
        <Button Content="Page 1" Click="Page1_Click" Margin="5" />
        <Button Content="Page 2" Click="Page2_Click" Margin="5" />
        <Button Content="Page 3" Click="Page3_Click" Margin="5" />
    </StackPanel>
</Grid>
```


Code-behind pour la transition :

```textmate
private void Page1_Click(object sender, RoutedEventArgs e)
{
    ChangePage(new PageViewModel { Title = "Page 1", Content = "Contenu de la page 1" });
}

private void Page2_Click(object sender, RoutedEventArgs e)
{
    ChangePage(new PageViewModel { Title = "Page 2", Content = "Contenu de la page 2" });
}

private void Page3_Click(object sender, RoutedEventArgs e)
{
    ChangePage(new PageViewModel { Title = "Page 3", Content = "Contenu de la page 3" });
}

private void ChangePage(PageViewModel newPage)
{
    // Mettre à jour la page actuelle
    CurrentPage = newPage;

    // Jouer l'animation de transition
    Storyboard transition = (Storyboard)FindResource("PageTransition");
    transition.Begin(this);
}
```

### Animations d'items dans un ItemsControl

Pour animer l'ajout ou la suppression d'éléments dans une liste :

```xml
<ItemsControl ItemsSource="{Binding Items}">
    <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
            <WrapPanel />
        </ItemsPanelTemplate>
    </ItemsControl.ItemsPanel>
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Border Background="LightBlue" Margin="5" Padding="10" Width="100" Height="100">
                <TextBlock Text="{Binding}" VerticalAlignment="Center" HorizontalAlignment="Center" />

                <Border.Style>
                    <Style TargetType="Border">
                        <Setter Property="Opacity" Value="1" />
                        <Setter Property="RenderTransform">
                            <Setter.Value>
                                <ScaleTransform ScaleX="1" ScaleY="1" />
                            </Setter.Value>
                        </Setter>
                        <Style.Triggers>
                            <EventTrigger RoutedEvent="Border.Loaded">
                                <BeginStoryboard>
                                    <Storyboard>
                                        <DoubleAnimation
                                            Storyboard.TargetProperty="Opacity"
                                            From="0" To="1" Duration="0:0:0.5" />
                                        <DoubleAnimation
                                            Storyboard.TargetProperty="(Border.RenderTransform).(ScaleTransform.ScaleX)"
                                            From="0.5" To="1" Duration="0:0:0.5">
                                            <DoubleAnimation.EasingFunction>
                                                <BackEase EasingMode="EaseOut" Amplitude="0.3" />
                                            </DoubleAnimation.EasingFunction>
                                        </DoubleAnimation>
                                        <DoubleAnimation
                                            Storyboard.TargetProperty="(Border.RenderTransform).(ScaleTransform.ScaleY)"
                                            From="0.5" To="1" Duration="0:0:0.5">
                                            <DoubleAnimation.EasingFunction>
                                                <BackEase EasingMode="EaseOut" Amplitude="0.3" />
                                            </DoubleAnimation.EasingFunction>
                                        </DoubleAnimation>
                                    </Storyboard>
                                </BeginStoryboard>
                            </EventTrigger>
                        </Style.Triggers>
                    </Style>
                </Border.Style>
            </Border>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```


Code-behind ou ViewModel pour ajouter des éléments avec un délai pour voir l'animation :

```textmate
public class MainViewModel : INotifyPropertyChanged
{
    private ObservableCollection<string> _items = new ObservableCollection<string>();

    public ObservableCollection<string> Items
    {
        get => _items;
        set
        {
            _items = value;
            OnPropertyChanged();
        }
    }

    public ICommand AddItemCommand { get; }
    public ICommand ClearItemsCommand { get; }

    public MainViewModel()
    {
        AddItemCommand = new RelayCommand(AddItem);
        ClearItemsCommand = new RelayCommand(ClearItems);

        // Ajouter quelques éléments initiaux
        AddInitialItems();
    }

    private async void AddInitialItems()
    {
        for (int i = 1; i <= 5; i++)
        {
            Items.Add($"Item {i}");
            // Attendre un peu entre chaque ajout pour voir l'animation
            await Task.Delay(300);
        }
    }

    private void AddItem()
    {
        Items.Add($"Item {Items.Count + 1}");
    }

    private void ClearItems()
    {
        Items.Clear();
    }

    // Implémentation de INotifyPropertyChanged
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


### Comportements d'interaction (.NET 8)

Les comportements d'interaction sont une fonctionnalité avancée qui permet d'encapsuler des fonctionnalités réutilisables. Ils sont disponibles dans le package `Microsoft.Xaml.Behaviors.Wpf`.

Installation pour .NET 8 :
```
dotnet add package Microsoft.Xaml.Behaviors.Wpf
```


Exemple d'utilisation de comportements pour animer un élément lors du clic :

```xml
<Window x:Class="BehaviorsSample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:b="http://schemas.microsoft.com/xaml/behaviors"
        Title="Behaviors Demo" Height="450" Width="800">

    <StackPanel Margin="20">
        <Button Content="Cliquez-moi" Width="120" Height="40">
            <b:Interaction.Triggers>
                <b:EventTrigger EventName="Click">
                    <b:PlaySoundAction Source="/Sounds/click.wav" />
                </b:EventTrigger>
            </b:Interaction.Triggers>
        </Button>

        <Rectangle Width="100" Height="100" Fill="Red" Margin="0,20">
            <b:Interaction.Triggers>
                <b:EventTrigger EventName="MouseEnter">
                    <b:ControlStoryboardAction Storyboard="{StaticResource PulseAnimation}" ControlStoryboardOption="Play" />
                </b:EventTrigger>
                <b:EventTrigger EventName="MouseLeave">
                    <b:ControlStoryboardAction Storyboard="{StaticResource PulseAnimation}" ControlStoryboardOption="Stop" />
                </b:EventTrigger>
            </b:Interaction.Triggers>
        </Rectangle>
    </StackPanel>

    <Window.Resources>
        <Storyboard x:Key="PulseAnimation" AutoReverse="True" RepeatBehavior="Forever">
            <DoubleAnimation
                Storyboard.TargetProperty="(Rectangle.RenderTransform).(ScaleTransform.ScaleX)"
                From="1" To="1.2" Duration="0:0:0.5" />
            <DoubleAnimation
                Storyboard.TargetProperty="(Rectangle.RenderTransform).(ScaleTransform.ScaleY)"
                From="1" To="1.2" Duration="0:0:0.5" />
        </Storyboard>
    </Window.Resources>
</Window>
```


### Création d'un comportement personnalisé

```textmate
// ColorPulseBehavior.cs
using Microsoft.Xaml.Behaviors;
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Animation;
using System.Windows.Shapes;

public class ColorPulseBehavior : Behavior<Shape>
{
    // Propriété de dépendance pour la couleur cible
    public static readonly DependencyProperty TargetColorProperty =
        DependencyProperty.Register(
            "TargetColor",
            typeof(Color),
            typeof(ColorPulseBehavior),
            new PropertyMetadata(Colors.Red));

    // Propriété de dépendance pour la durée
    public static readonly DependencyProperty DurationProperty =
        DependencyProperty.Register(
            "Duration",
            typeof(Duration),
            typeof(ColorPulseBehavior),
            new PropertyMetadata(new Duration(TimeSpan.FromSeconds(1))));

    // Propriétés pour accéder aux propriétés de dépendance
    public Color TargetColor
    {
        get => (Color)GetValue(TargetColorProperty);
        set => SetValue(TargetColorProperty, value);
    }

    public Duration Duration
    {
        get => (Duration)GetValue(DurationProperty);
        set => SetValue(DurationProperty, value);
    }

    private Storyboard _storyboard;
    private Color _originalColor;

    // Appelé quand le comportement est attaché à l'élément
    protected override void OnAttached()
    {
        base.OnAttached();

        // S'abonner à l'événement MouseEnter
        AssociatedObject.MouseEnter += AssociatedObject_MouseEnter;
        AssociatedObject.MouseLeave += AssociatedObject_MouseLeave;

        // Capturer la couleur d'origine
        _originalColor = ((SolidColorBrush)AssociatedObject.Fill).Color;

        // Créer le storyboard
        _storyboard = new Storyboard();

        // Créer l'animation de couleur
        var colorAnimation = new ColorAnimation
        {
            Duration = Duration,
            AutoReverse = true,
            RepeatBehavior = RepeatBehavior.Forever
        };

        Storyboard.SetTarget(colorAnimation, AssociatedObject);
        Storyboard.SetTargetProperty(colorAnimation, new PropertyPath("(Shape.Fill).(SolidColorBrush.Color)"));

        _storyboard.Children.Add(colorAnimation);
    }

    // Appelé quand le comportement est détaché de l'élément
    protected override void OnDetaching()
    {
        // Se désabonner des événements
        AssociatedObject.MouseEnter -= AssociatedObject_MouseEnter;
        AssociatedObject.MouseLeave -= AssociatedObject_MouseLeave;

        // Arrêter l'animation
        _storyboard.Stop(AssociatedObject);

        base.OnDetaching();
    }

    private void AssociatedObject_MouseEnter(object sender, System.Windows.Input.MouseEventArgs e)
    {
        // Mettre à jour les valeurs de l'animation
        var colorAnimation = (ColorAnimation)_storyboard.Children[0];
        colorAnimation.From = _originalColor;
        colorAnimation.To = TargetColor;

        // Démarrer l'animation
        _storyboard.Begin(AssociatedObject, true);
    }

    private void AssociatedObject_MouseLeave(object sender, System.Windows.Input.MouseEventArgs e)
    {
        // Arrêter l'animation
        _storyboard.Stop(AssociatedObject);

        // Remettre la couleur d'origine
        ((SolidColorBrush)AssociatedObject.Fill).Color = _originalColor;
    }
}
```


Utilisation du comportement personnalisé :

```xml
<Window x:Class="BehaviorsSample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:b="http://schemas.microsoft.com/xaml/behaviors"
        xmlns:local="clr-namespace:BehaviorsSample"
        Title="Behaviors Demo" Height="450" Width="800">

    <StackPanel Margin="20">
        <Rectangle Width="150" Height="100" Fill="Blue" Margin="10">
            <b:Interaction.Behaviors>
                <local:ColorPulseBehavior TargetColor="Yellow" Duration="0:0:0.7" />
            </b:Interaction.Behaviors>
        </Rectangle>

        <Ellipse Width="100" Height="100" Fill="Green" Margin="10">
            <b:Interaction.Behaviors>
                <local:ColorPulseBehavior TargetColor="Purple" Duration="0:0:1.2" />
            </b:Interaction.Behaviors>
        </Ellipse>

        <Rectangle Width="200" Height="50" Fill="Orange" Margin="10">
            <b:Interaction.Behaviors>
                <local:ColorPulseBehavior TargetColor="Red" Duration="0:0:0.5" />
            </b:Interaction.Behaviors>
        </Rectangle>
    </StackPanel>
</Window>
```


### Animation de transitions de page avec .NET 8

.NET 8 introduit des améliorations de performances pour les animations. Voici comment créer une transition de page fluide :

```textmate
// PageTransitionService.cs
public static class PageTransitionService
{
    // Propriété de pièce jointe pour activer les transitions
    public static readonly DependencyProperty EnableTransitionsProperty =
        DependencyProperty.RegisterAttached(
            "EnableTransitions",
            typeof(bool),
            typeof(PageTransitionService),
            new PropertyMetadata(false, OnEnableTransitionsChanged));

    public static bool GetEnableTransitions(ContentControl element)
    {
        return (bool)element.GetValue(EnableTransitionsProperty);
    }

    public static void SetEnableTransitions(ContentControl element, bool value)
    {
        element.SetValue(EnableTransitionsProperty, value);
    }

    // Propriété pour la durée de transition
    public static readonly DependencyProperty TransitionDurationProperty =
        DependencyProperty.RegisterAttached(
            "TransitionDuration",
            typeof(Duration),
            typeof(PageTransitionService),
            new PropertyMetadata(new Duration(TimeSpan.FromMilliseconds(300))));

    public static Duration GetTransitionDuration(ContentControl element)
    {
        return (Duration)element.GetValue(TransitionDurationProperty);
    }

    public static void SetTransitionDuration(ContentControl element, Duration value)
    {
        element.SetValue(TransitionDurationProperty, value);
    }

    // Gestionnaire de changement pour EnableTransitions
    private static void OnEnableTransitionsChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is ContentControl contentControl)
        {
            if ((bool)e.NewValue)
            {
                // Activer les transitions en s'abonnant au changement de contenu
                contentControl.ContentChanged += ContentControl_ContentChanged;
            }
            else
            {
                // Désactiver les transitions
                contentControl.ContentChanged -= ContentControl_ContentChanged;
            }
        }
    }

    // Gestionnaire pour le changement de contenu
    private static void ContentControl_ContentChanged(object sender, EventArgs e)
    {
        ContentControl contentControl = (ContentControl)sender;
        Duration duration = GetTransitionDuration(contentControl);

        // Créer et configurer l'animation
        Storyboard sb = new Storyboard();

        // Animation d'opacité
        DoubleAnimation opacityAnimation = new DoubleAnimation
        {
            From = 0,
            To = 1,
            Duration = duration,
            EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseOut }
        };
        Storyboard.SetTarget(opacityAnimation, contentControl);
        Storyboard.SetTargetProperty(opacityAnimation, new PropertyPath(UIElement.OpacityProperty));
        sb.Children.Add(opacityAnimation);

        // Animation de translation (slide-in)
        ThicknessAnimation marginAnimation = new ThicknessAnimation
        {
            From = new Thickness(20, 0, -20, 0),
            To = new Thickness(0),
            Duration = duration,
            EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseOut }
        };
        Storyboard.SetTarget(marginAnimation, contentControl);
        Storyboard.SetTargetProperty(marginAnimation, new PropertyPath(Control.MarginProperty));
        sb.Children.Add(marginAnimation);

        // Démarrer l'animation
        sb.Begin();
    }
}
```


Utilisation de ce service de transition :

```xml
<Grid>
    <ContentControl
        x:Name="pageHost"
        Content="{Binding CurrentPage}"
        local:PageTransitionService.EnableTransitions="True"
        local:PageTransitionService.TransitionDuration="0:0:0.4">
        <ContentControl.ContentTemplate>
            <DataTemplate>
                <Grid>
                    <TextBlock Text="{Binding Header}" FontSize="24" FontWeight="Bold"
                               Margin="0,0,0,20" HorizontalAlignment="Center" />
                    <ContentPresenter Content="{Binding Content}" Margin="0,40,0,0" />
                </Grid>
            </DataTemplate>
        </ContentControl.ContentTemplate>
    </ContentControl>

    <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom"
                HorizontalAlignment="Center" Margin="0,0,0,20">
        <Button Content="Page 1" Command="{Binding NavigateToPage1Command}" Margin="5" />
        <Button Content="Page 2" Command="{Binding NavigateToPage2Command}" Margin="5" />
        <Button Content="Page 3" Command="{Binding NavigateToPage3Command}" Margin="5" />
    </StackPanel>
</Grid>
```


### Optimisations des animations en .NET 8

.NET 8 apporte plusieurs améliorations de performances pour les animations WPF :

1. **Composition d'animations** : Utilise davantage le GPU pour des animations plus fluides

```textmate
// Activer les fonctionnalités avancées de composition dans App.xaml.cs
protected override void OnStartup(StartupEventArgs e)
{
    base.OnStartup(e);

    // Activer les optimisations de composition (si disponibles)
    RenderOptions.ProcessRenderMode = RenderMode.Default;
}
```


2. **Animations parallélisées** : .NET 8 parallélise mieux le traitement des animations pour de meilleures performances

3. **Améliorations du calcul des frames** : Plus précis et efficace

Pour profiter au maximum de ces optimisations, utilisez ces bonnes pratiques :

```xml
<!-- Optimisations pour .NET 8 -->
<Rectangle Width="200" Height="100" Fill="DodgerBlue">
    <!-- CacheMode améliore les performances des transformations -->
    <Rectangle.CacheMode>
        <BitmapCache />
    </Rectangle.CacheMode>

    <!-- RenderTransform est plus efficace que LayoutTransform -->
    <Rectangle.RenderTransform>
        <TransformGroup>
            <ScaleTransform x:Name="scaleTransform" />
            <RotateTransform x:Name="rotateTransform" />
        </TransformGroup>
    </Rectangle.RenderTransform>

    <Rectangle.Triggers>
        <EventTrigger RoutedEvent="Rectangle.MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <!-- Optimisé pour .NET 8 -->
                    <DoubleAnimation
                        Storyboard.TargetName="scaleTransform"
                        Storyboard.TargetProperty="ScaleX"
                        To="1.2" Duration="0:0:0.2"
                        EnableDependentAnimation="True" />
                    <DoubleAnimation
                        Storyboard.TargetName="scaleTransform"
                        Storyboard.TargetProperty="ScaleY"
                        To="1.2" Duration="0:0:0.2"
                        EnableDependentAnimation="True" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
        <EventTrigger RoutedEvent="Rectangle.MouseLeave">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation
                        Storyboard.TargetName="scaleTransform"
                        Storyboard.TargetProperty="ScaleX"
                        To="1" Duration="0:0:0.2" />
                    <DoubleAnimation
                        Storyboard.TargetName="scaleTransform"
                        Storyboard.TargetProperty="ScaleY"
                        To="1" Duration="0:0:0.2" />
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Rectangle.Triggers>
</Rectangle>
```


### Conseils pour des animations performantes

1. **Utilisez RenderTransform plutôt que LayoutTransform** lorsque possible
2. **Activez le cache des éléments** pour les animations complexes avec `CacheMode="BitmapCache"`
3. **Limitez le nombre d'éléments animés** simultanément
4. **Utilisez des durées raisonnables** (entre 200ms et 500ms pour la plupart des animations)
5. **Testez sur des appareils bas de gamme** pour vous assurer de la fluidité
6. **Évitez d'animer les propriétés qui déclenchent un re-layout** (comme Width/Height directement)
7. **Privilégiez l'animation des propriétés Transform** (Scale, Rotate, Translate)
8. **Utilisez CompositionTarget.Rendering** pour les animations personnalisées très complexes

### Exemple d'animation avancée : Carrousel 3D

```xml
<UserControl x:Class="MyApplication.Carousel3D"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Grid>
        <Viewport3D Name="viewport" ClipToBounds="True">
            <Viewport3D.Camera>
                <PerspectiveCamera
                    Position="0,0,5"
                    LookDirection="0,0,-1"
                    UpDirection="0,1,0"
                    FieldOfView="60" />
            </Viewport3D.Camera>

            <ModelVisual3D>
                <ModelVisual3D.Content>
                    <Model3DGroup x:Name="carouselGroup">
                        <!-- Les éléments du carrousel seront ajoutés ici en code -->
                    </Model3DGroup>
                </ModelVisual3D.Content>
            </ModelVisual3D>

            <ModelVisual3D>
                <ModelVisual3D.Content>
                    <DirectionalLight
                        Color="White"
                        Direction="0,0,-1" />
                </ModelVisual3D.Content>
            </ModelVisual3D>
        </Viewport3D>

        <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Bottom" Margin="10">
            <Button Content="Précédent" Click="PreviousButton_Click" Width="80" Margin="5" />
            <Button Content="Suivant" Click="NextButton_Click" Width="80" Margin="5" />
        </StackPanel>
    </Grid>
</UserControl>
```


Code-behind pour le carrousel 3D :

```textmate
public partial class Carousel3D : UserControl
{
    private readonly RotateTransform3D _rotateTransform;
    private readonly AxisAngleRotation3D _rotation;
    private readonly List<string> _imageUrls = new List<string>();
    private int _currentIndex = 0;
    private const int ItemCount = 5;
    private const double AngleIncrement = 360.0 / ItemCount;

    public Carousel3D()
    {
        InitializeComponent();

        // Initialiser la transformation de rotation
        _rotation = new AxisAngleRotation3D(new Vector3D(0, 1, 0), 0);
        _rotateTransform = new RotateTransform3D(_rotation);
        carouselGroup.Transform = _rotateTransform;

        // Charger les images d'exemple
        _imageUrls.Add("/Images/image1.jpg");
        _imageUrls.Add("/Images/image2.jpg");
        _imageUrls.Add("/Images/image3.jpg");
        _imageUrls.Add("/Images/image4.jpg");
        _imageUrls.Add("/Images/image5.jpg");

        // Créer les éléments du carrousel
        CreateCarouselItems();
    }

    private void CreateCarouselItems()
    {
        for (int i = 0; i < ItemCount; i++)
        {
            // Créer le matériau avec l'image
            Material material = new DiffuseMaterial(
                new ImageBrush(new BitmapImage(new Uri(_imageUrls[i], UriKind.Relative)))
            );

            // Créer un plan 3D (carte)
            MeshGeometry3D mesh = new MeshGeometry3D();

            // Définir les points du plan
            Point3DCollection points = new Point3DCollection
            {
                new Point3D(-0.5, -0.5, 0),  // En bas à gauche
                new Point3D(0.5, -0.5, 0),   // En bas à droite
                new Point3D(0.5, 0.5, 0),    // En haut à droite
                new Point3D(-0.5, 0.5, 0)    // En haut à gauche
            };
            mesh.Positions = points;

            // Définir les indices pour les triangles
            Int32Collection indices = new Int32Collection
            {
                0, 1, 2,  // Premier triangle
                0, 2, 3   // Deuxième triangle
            };
            mesh.TriangleIndices = indices;

            // Définir les coordonnées de texture
            PointCollection textureCoordinates = new PointCollection
            {
                new Point(0, 1),  // En bas à gauche
                new Point(1, 1),  // En bas à droite
                new Point(1, 0),  // En haut à droite
                new Point(0, 0)   // En haut à gauche
            };
            mesh.TextureCoordinates = textureCoordinates;

            // Créer le modèle géométrique
            GeometryModel3D model = new GeometryModel3D(mesh, material);

            // Positionner le modèle dans le carrousel
            double angle = i * AngleIncrement * Math.PI / 180.0;
            double x = 2.0 * Math.Sin(angle);
            double z = 2.0 * Math.Cos(angle);

            // Appliquer la transformation de position
            Transform3DGroup transformGroup = new Transform3DGroup();
            transformGroup.Children.Add(new RotateTransform3D(new AxisAngleRotation3D(new Vector3D(0, 1, 0), -i * AngleIncrement)));
            transformGroup.Children.Add(new TranslateTransform3D(x, 0, z));
            model.Transform = transformGroup;

            // Ajouter le modèle au groupe
            carouselGroup.Children.Add(model);
        }
    }

    private void RotateCarousel(double targetAngle)
    {
        // Créer l'animation
        DoubleAnimation animation = new DoubleAnimation
        {
            To = targetAngle,
            Duration = TimeSpan.FromSeconds(0.5),
            EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseInOut }
        };

        // Démarrer l'animation
        _rotation.BeginAnimation(AxisAngleRotation3D.AngleProperty, animation);
    }

    private void NextButton_Click(object sender, RoutedEventArgs e)
    {
        _currentIndex = (_currentIndex + 1) % ItemCount;
        double targetAngle = _currentIndex * AngleIncrement;
        RotateCarousel(targetAngle);
    }

    private void PreviousButton_Click(object sender, RoutedEventArgs e)
    {
        _currentIndex = (_currentIndex - 1 + ItemCount) % ItemCount;
        double targetAngle = _currentIndex * AngleIncrement;
        RotateCarousel(targetAngle);
    }
}
```


Les animations et effets visuels sont des outils puissants pour améliorer l'expérience utilisateur de vos applications WPF. En comprenant les différents types d'animations et en les appliquant judicieusement, vous pouvez créer des interfaces utilisateur plus intuitives et plus attrayantes. N'oubliez pas de toujours équilibrer les effets visuels avec les performances pour assurer une expérience fluide sur tous les appareils.

⏭️ 8.10. [Personnalisation des contrôles](/08-windows-presentation-foundation-wpf/8-10-personnalisation-des-controles.md)
