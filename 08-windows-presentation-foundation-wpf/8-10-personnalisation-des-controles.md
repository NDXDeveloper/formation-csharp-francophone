# 8.10. Personnalisation des contrôles

🔝 Retour au [Sommaire](/SOMMAIRE.md)

WPF offre une grande flexibilité pour personnaliser l'apparence et le comportement des contrôles existants, ainsi que pour créer vos propres contrôles. Cette section explore les différentes approches pour créer des contrôles personnalisés et comprendre les mécanismes fondamentaux de WPF comme les propriétés de dépendance et les événements routés.

## 8.10.1. Création de contrôles personnalisés

Il existe trois approches principales pour créer des contrôles personnalisés en WPF :

1. **Contrôles utilisateur (UserControl)** - Composition de contrôles existants
2. **Contrôles personnalisés** - Dérivés de `Control` ou d'un contrôle existant
3. **Contrôles de formulaire** - Dérivés de `FrameworkElement` pour un contrôle entièrement personnalisé

### Création d'un UserControl

Un UserControl est la méthode la plus simple pour créer un contrôle personnalisé. Il s'agit essentiellement d'un conteneur qui regroupe d'autres contrôles.

Pour créer un UserControl :

1. Cliquez-droit sur votre projet → Ajouter → Nouveau élément → UserControl WPF
2. Nommez-le, par exemple "RatingControl.xaml"

```xml
<!-- RatingControl.xaml -->
<UserControl x:Class="MonApplication.Controls.RatingControl"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             mc:Ignorable="d"
             d:DesignHeight="40" d:DesignWidth="200">
    <Grid>
        <StackPanel x:Name="starContainer" Orientation="Horizontal">
            <!-- Les étoiles seront ajoutées dynamiquement -->
        </StackPanel>
    </Grid>
</UserControl>
```


```textmate
// RatingControl.xaml.cs
using System.Windows;
using System.Windows.Controls;
using System.Windows.Input;
using System.Windows.Media;

namespace MonApplication.Controls
{
    public partial class RatingControl : UserControl
    {
        // Propriété de dépendance pour le nombre maximum d'étoiles
        public static readonly DependencyProperty MaxRatingProperty =
            DependencyProperty.Register("MaxRating", typeof(int), typeof(RatingControl),
                new PropertyMetadata(5, OnMaxRatingChanged));

        // Propriété de dépendance pour la valeur de notation actuelle
        public static readonly DependencyProperty RatingProperty =
            DependencyProperty.Register("Rating", typeof(int), typeof(RatingControl),
                new PropertyMetadata(0, OnRatingChanged));

        // Propriété de dépendance pour la couleur de l'étoile sélectionnée
        public static readonly DependencyProperty SelectedColorProperty =
            DependencyProperty.Register("SelectedColor", typeof(Brush), typeof(RatingControl),
                new PropertyMetadata(Brushes.Gold));

        // Propriété de dépendance pour la couleur de l'étoile non sélectionnée
        public static readonly DependencyProperty UnselectedColorProperty =
            DependencyProperty.Register("UnselectedColor", typeof(Brush), typeof(RatingControl),
                new PropertyMetadata(Brushes.Gray));

        // Propriétés CLR exposées publiquement
        public int MaxRating
        {
            get { return (int)GetValue(MaxRatingProperty); }
            set { SetValue(MaxRatingProperty, value); }
        }

        public int Rating
        {
            get { return (int)GetValue(RatingProperty); }
            set { SetValue(RatingProperty, value); }
        }

        public Brush SelectedColor
        {
            get { return (Brush)GetValue(SelectedColorProperty); }
            set { SetValue(SelectedColorProperty, value); }
        }

        public Brush UnselectedColor
        {
            get { return (Brush)GetValue(UnselectedColorProperty); }
            set { SetValue(UnselectedColorProperty, value); }
        }

        // Événement pour notifier que la notation a changé
        public event RoutedEventHandler RatingChanged
        {
            add { AddHandler(RatingChangedEvent, value); }
            remove { RemoveHandler(RatingChangedEvent, value); }
        }

        // Événement routé personnalisé
        public static readonly RoutedEvent RatingChangedEvent = EventManager.RegisterRoutedEvent(
            "RatingChanged", RoutingStrategy.Bubble, typeof(RoutedEventHandler), typeof(RatingControl));

        public RatingControl()
        {
            InitializeComponent();

            // Valeurs par défaut
            MaxRating = 5;
            Rating = 0;
            SelectedColor = Brushes.Gold;
            UnselectedColor = Brushes.Gray;

            // Initialiser les étoiles
            InitializeStars();
        }

        private void InitializeStars()
        {
            starContainer.Children.Clear();

            for (int i = 1; i <= MaxRating; i++)
            {
                TextBlock star = new TextBlock
                {
                    Text = "★",
                    FontSize = 24,
                    Margin = new Thickness(2, 0, 2, 0),
                    Tag = i,
                    Foreground = (i <= Rating) ? SelectedColor : UnselectedColor
                };

                star.MouseLeftButtonDown += Star_MouseLeftButtonDown;
                star.MouseEnter += Star_MouseEnter;
                star.MouseLeave += Star_MouseLeave;

                starContainer.Children.Add(star);
            }
        }

        private void Star_MouseLeftButtonDown(object sender, MouseButtonEventArgs e)
        {
            if (sender is TextBlock star)
            {
                Rating = (int)star.Tag;

                // Déclencher l'événement RatingChanged
                RoutedEventArgs args = new RoutedEventArgs(RatingChangedEvent);
                RaiseEvent(args);
            }
        }

        private void Star_MouseEnter(object sender, MouseEventArgs e)
        {
            if (sender is TextBlock hoveredStar)
            {
                int hoveredValue = (int)hoveredStar.Tag;

                // Mettre en évidence les étoiles jusqu'à celle survolée
                foreach (TextBlock star in starContainer.Children)
                {
                    int starValue = (int)star.Tag;
                    star.Foreground = (starValue <= hoveredValue) ? SelectedColor : UnselectedColor;
                }
            }
        }

        private void Star_MouseLeave(object sender, MouseEventArgs e)
        {
            // Rétablir les étoiles basées sur la notation actuelle
            foreach (TextBlock star in starContainer.Children)
            {
                int starValue = (int)star.Tag;
                star.Foreground = (starValue <= Rating) ? SelectedColor : UnselectedColor;
            }
        }

        private static void OnMaxRatingChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            if (d is RatingControl ratingControl)
            {
                ratingControl.InitializeStars();
            }
        }

        private static void OnRatingChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            if (d is RatingControl ratingControl)
            {
                // Mettre à jour l'affichage des étoiles
                foreach (TextBlock star in ratingControl.starContainer.Children)
                {
                    int starValue = (int)star.Tag;
                    star.Foreground = (starValue <= ratingControl.Rating)
                        ? ratingControl.SelectedColor
                        : ratingControl.UnselectedColor;
                }
            }
        }
    }
}
```


### Utilisation du contrôle personnalisé

```xml
<!-- Ajouter une référence au namespace -->
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="clr-namespace:MonApplication.Controls"
        Title="Test de contrôles personnalisés" Height="450" Width="800">
    <StackPanel Margin="20">
        <TextBlock Text="Évaluez notre application :" Margin="0,0,0,10" />

        <controls:RatingControl x:Name="ratingControl"
                               MaxRating="5"
                               Rating="3"
                               SelectedColor="Gold"
                               UnselectedColor="LightGray"
                               RatingChanged="RatingControl_RatingChanged" />

        <TextBlock x:Name="ratingText" Margin="0,10,0,0" />
    </StackPanel>
</Window>
```


```textmate
// MainWindow.xaml.cs
private void RatingControl_RatingChanged(object sender, RoutedEventArgs e)
{
    if (sender is Controls.RatingControl ratingControl)
    {
        ratingText.Text = $"Votre évaluation : {ratingControl.Rating} étoile(s)";
    }
}
```


## 8.10.2. Héritage de contrôles existants

L'héritage vous permet d'étendre des contrôles existants en ajoutant des fonctionnalités ou en modifiant leur comportement.

### Créer un TextBox avec effacement automatique

```textmate
// ClearableTextBox.cs
using System.Windows;
using System.Windows.Controls;
using System.Windows.Input;
using System.Windows.Media;

namespace MonApplication.Controls
{
    public class ClearableTextBox : TextBox
    {
        // Propriété de dépendance pour le texte d'indication
        public static readonly DependencyProperty PlaceholderTextProperty =
            DependencyProperty.Register("PlaceholderText", typeof(string), typeof(ClearableTextBox),
                new PropertyMetadata(string.Empty));

        // Propriété de dépendance pour la couleur du texte d'indication
        public static readonly DependencyProperty PlaceholderForegroundProperty =
            DependencyProperty.Register("PlaceholderForeground", typeof(Brush), typeof(ClearableTextBox),
                new PropertyMetadata(Brushes.Gray));

        // Propriété CLR pour le texte d'indication
        public string PlaceholderText
        {
            get { return (string)GetValue(PlaceholderTextProperty); }
            set { SetValue(PlaceholderTextProperty, value); }
        }

        // Propriété CLR pour la couleur du texte d'indication
        public Brush PlaceholderForeground
        {
            get { return (Brush)GetValue(PlaceholderForegroundProperty); }
            set { SetValue(PlaceholderForegroundProperty, value); }
        }

        // Constructeur
        static ClearableTextBox()
        {
            // Override du style par défaut
            DefaultStyleKeyProperty.OverrideMetadata(
                typeof(ClearableTextBox),
                new FrameworkPropertyMetadata(typeof(ClearableTextBox)));
        }

        public ClearableTextBox()
        {
            // Événements pour gérer le placeholder
            this.GotFocus += ClearableTextBox_GotFocus;
            this.LostFocus += ClearableTextBox_LostFocus;
            this.TextChanged += ClearableTextBox_TextChanged;

            // Appliquer le placeholder initial si le champ est vide
            UpdatePlaceholder();
        }

        private bool IsPlaceholderVisible { get; set; }

        private void ClearableTextBox_GotFocus(object sender, RoutedEventArgs e)
        {
            // Si le texte actuel est le placeholder, l'effacer
            if (IsPlaceholderVisible)
            {
                Text = string.Empty;
                Foreground = (Brush)GetValue(ForegroundProperty); // Restaurer la couleur normale
                IsPlaceholderVisible = false;
            }
        }

        private void ClearableTextBox_LostFocus(object sender, RoutedEventArgs e)
        {
            UpdatePlaceholder();
        }

        private void ClearableTextBox_TextChanged(object sender, TextChangedEventArgs e)
        {
            if (IsPlaceholderVisible && IsFocused)
            {
                IsPlaceholderVisible = false;
                Foreground = (Brush)GetValue(ForegroundProperty);
            }
        }

        private void UpdatePlaceholder()
        {
            // Si le texte est vide et le contrôle n'a pas le focus, afficher le placeholder
            if (string.IsNullOrEmpty(Text) && !IsFocused)
            {
                Text = PlaceholderText;
                Foreground = PlaceholderForeground;
                IsPlaceholderVisible = true;
            }
        }

        // Override de OnMouseDown pour ajouter le comportement de nettoyage
        protected override void OnMouseDown(MouseButtonEventArgs e)
        {
            base.OnMouseDown(e);

            // Si c'est un double-clic, effacer le texte
            if (e.ClickCount == 2)
            {
                this.Text = string.Empty;
                IsPlaceholderVisible = false;
                e.Handled = true;
            }
        }
    }
}
```


### Définir le style du contrôle personnalisé

Créez un fichier Generic.xaml dans un dossier "Themes" pour définir le style par défaut :

```xml
<!-- /Themes/Generic.xaml -->
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="clr-namespace:MonApplication.Controls">

    <!-- Style par défaut pour ClearableTextBox -->
    <Style TargetType="{x:Type local:ClearableTextBox}">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="{x:Type local:ClearableTextBox}">
                    <Grid>
                        <Border Background="{TemplateBinding Background}"
                                BorderBrush="{TemplateBinding BorderBrush}"
                                BorderThickness="{TemplateBinding BorderThickness}"
                                CornerRadius="3">
                            <Grid>
                                <ScrollViewer x:Name="PART_ContentHost" Margin="5,2" />
                                <Button x:Name="ClearButton"
                                        Visibility="Collapsed"
                                        Width="16" Height="16"
                                        HorizontalAlignment="Right"
                                        Margin="0,0,5,0"
                                        Background="Transparent"
                                        BorderBrush="Transparent"
                                        Click="ClearButton_Click">
                                    <Path Data="M0,0 L6,6 M0,6 L6,0"
                                          Stroke="Gray"
                                          StrokeThickness="1.5"
                                          Stretch="Uniform" />
                                </Button>
                            </Grid>
                        </Border>
                    </Grid>
                    <ControlTemplate.Triggers>
                        <DataTrigger Binding="{Binding Text.Length, RelativeSource={RelativeSource Self}}" Value="0">
                            <Setter TargetName="ClearButton" Property="Visibility" Value="Collapsed" />
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Text.Length, RelativeSource={RelativeSource Self}}" Value="0" Comparison="GreaterThan">
                            <Setter TargetName="ClearButton" Property="Visibility" Value="Visible" />
                        </DataTrigger>
                        <Trigger Property="IsPlaceholderVisible" Value="True">
                            <Setter TargetName="ClearButton" Property="Visibility" Value="Collapsed" />
                        </Trigger>
                    </ControlTemplate.Triggers>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>

</ResourceDictionary>
```


Ajoutez cette méthode au code de la classe ClearableTextBox :

```textmate
private void ClearButton_Click(object sender, RoutedEventArgs e)
{
    Text = string.Empty;
    IsPlaceholderVisible = false;
    this.Focus();
}
```


### Utilisation du contrôle basé sur l'héritage

```xml
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="clr-namespace:MonApplication.Controls"
        Title="Test de contrôles personnalisés" Height="450" Width="800">
    <StackPanel Margin="20">
        <TextBlock Text="TextBox avec placeholder et bouton d'effacement :" Margin="0,0,0,10" />

        <controls:ClearableTextBox PlaceholderText="Entrez votre recherche..."
                                   Width="300"
                                   Margin="0,0,0,20" />

        <controls:ClearableTextBox PlaceholderText="Entrez votre email..."
                                   PlaceholderForeground="LightBlue"
                                   Width="300" />
    </StackPanel>
</Window>
```


## 8.10.3. Propriétés de dépendance

Les propriétés de dépendance sont un concept fondamental en WPF qui permet de créer des propriétés avec des fonctionnalités avancées comme la liaison de données, l'animation, l'héritage des valeurs, etc.

### Structure d'une propriété de dépendance

Une propriété de dépendance comporte les éléments suivants :

1. **Un champ statique** de type `DependencyProperty` qui stocke l'identificateur de la propriété
2. **Une méthode d'enregistrement** qui enregistre la propriété dans le système WPF
3. **Une propriété CLR wrapper** qui fournit un accès typé à la propriété

```textmate
// Exemple simplifié
public class MonControle : Control
{
    // 1. Champ statique qui identifie la propriété
    public static readonly DependencyProperty ValeurProperty =
        DependencyProperty.Register(
            // Nom de la propriété
            "Valeur",
            // Type de la propriété
            typeof(int),
            // Type du propriétaire
            typeof(MonControle),
            // Métadonnées (valeur par défaut, callbacks, etc.)
            new PropertyMetadata(0, OnValeurChanged)
        );

    // 2. Propriété CLR qui encapsule l'accès à la propriété
    public int Valeur
    {
        get { return (int)GetValue(ValeurProperty); }
        set { SetValue(ValeurProperty, value); }
    }

    // 3. Méthode callback qui est appelée quand la valeur change
    private static void OnValeurChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        MonControle controle = (MonControle)d;
        int nouvelleValeur = (int)e.NewValue;

        // Faire quelque chose quand la valeur change
        controle.RéagirAuChangement(nouvelleValeur);
    }

    private void RéagirAuChangement(int nouvelleValeur)
    {
        // Code qui réagit au changement de valeur
    }
}
```


### Types de métadonnées pour les propriétés de dépendance

Vous pouvez configurer plusieurs aspects d'une propriété de dépendance via les métadonnées :

```textmate
new PropertyMetadata(
    // Valeur par défaut
    defaultValue: 0,

    // Gestionnaire de changement de propriété
    propertyChangedCallback: OnValeurChanged,

    // Gestionnaire de vérification (coercion) de valeur
    coerceValueCallback: CoerceValeur
)

// ou avec FrameworkPropertyMetadata pour plus d'options
new FrameworkPropertyMetadata(
    // Valeur par défaut
    defaultValue: 0,

    // Flags pour configurer le comportement
    flags: FrameworkPropertyMetadataOptions.AffectsRender |
           FrameworkPropertyMetadataOptions.BindsTwoWayByDefault,

    // Gestionnaire de changement de propriété
    propertyChangedCallback: OnValeurChanged,

    // Gestionnaire de vérification de valeur
    coerceValueCallback: CoerceValeur,

    // Est-ce que la propriété peut être animée?
    isAnimationProhibited: false
)
```


### Exemple de propriété de dépendance avec validation et vérification

```textmate
public class AgeControl : Control
{
    public static readonly DependencyProperty AgeProperty =
        DependencyProperty.Register(
            "Age",
            typeof(int),
            typeof(AgeControl),
            new FrameworkPropertyMetadata(
                defaultValue: 0,
                flags: FrameworkPropertyMetadataOptions.BindsTwoWayByDefault,
                propertyChangedCallback: OnAgeChanged,
                coerceValueCallback: CoerceAge
            ),
            ValidateAge
        );

    public int Age
    {
        get { return (int)GetValue(AgeProperty); }
        set { SetValue(AgeProperty, value); }
    }

    // Validation : vérifie si l'âge est valide avant d'accepter la valeur
    private static bool ValidateAge(object value)
    {
        int age = (int)value;
        return age >= 0 && age <= 150; // Age doit être entre 0 et 150
    }

    // Coercion : ajuste la valeur si nécessaire pour qu'elle soit dans les limites
    private static object CoerceAge(DependencyObject d, object baseValue)
    {
        int age = (int)baseValue;

        if (age < 0)
            return 0;

        if (age > 150)
            return 150;

        return age;
    }

    // Gestionnaire de changement : réagit quand la valeur change
    private static void OnAgeChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        AgeControl control = (AgeControl)d;
        int nouvelAge = (int)e.NewValue;

        // Mettre à jour le contrôle en fonction du nouvel âge
        if (nouvelAge >= 18)
        {
            control.Background = Brushes.Green;
        }
        else
        {
            control.Background = Brushes.Red;
        }
    }
}
```


## 8.10.4. Événements routés

Les événements routés sont un mécanisme d'événement spécial dans WPF qui permet aux événements de "remonter" ou "descendre" dans l'arbre visuel des contrôles.

### Types d'événements routés

1. **Événements de bulles (Bubbling)** : L'événement commence à l'élément qui l'a déclenché et remonte dans l'arbre visuel (enfant → parent)
2. **Événements tunnelants (Tunneling)** : L'événement commence à la racine et descend dans l'arbre visuel (parent → enfant)
3. **Événements directs (Direct)** : L'événement ne se propage pas et reste sur l'élément qui l'a déclenché

### Déclaration d'un événement routé personnalisé

```textmate
public class MonBoutonSpecial : Button
{
    // 1. Définir un champ statique pour l'événement
    public static readonly RoutedEvent DoubleClickEvent = EventManager.RegisterRoutedEvent(
        // Nom de l'événement
        "DoubleClick",
        // Stratégie de routage (Bubble, Tunnel, Direct)
        RoutingStrategy.Bubble,
        // Type de gestionnaire d'événement
        typeof(RoutedEventHandler),
        // Type du propriétaire
        typeof(MonBoutonSpecial)
    );

    // 2. Ajouter des accesseurs CLR pour l'événement
    public event RoutedEventHandler DoubleClick
    {
        add { AddHandler(DoubleClickEvent, value); }
        remove { RemoveHandler(DoubleClickEvent, value); }
    }

    // 3. Override de la méthode OnMouseDown pour déclencher l'événement
    protected override void OnMouseDown(MouseButtonEventArgs e)
    {
        base.OnMouseDown(e);

        if (e.ClickCount == 2)
        {
            // Créer les arguments d'événement
            RoutedEventArgs args = new RoutedEventArgs(DoubleClickEvent, this);

            // Déclencher l'événement
            RaiseEvent(args);
        }
    }
}
```


### Utilisation d'un événement routé personnalisé

```xml
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="clr-namespace:MonApplication.Controls"
        Title="Test d'événements routés" Height="450" Width="800">
    <Grid>
        <controls:MonBoutonSpecial Content="Double-cliquez moi"
                                   Width="150" Height="40"
                                   DoubleClick="MonBouton_DoubleClick" />
    </Grid>
</Window>
```


```textmate
// MainWindow.xaml.cs
private void MonBouton_DoubleClick(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Vous avez double-cliqué sur le bouton !");
}
```


### Écouter des événements routés dans l'arbre visuel

```xml
<Grid>
    <!-- Cet événement se déclenche quand on clique sur n'importe quel bouton dans la Grid -->
    <Grid.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Option 1" />
            <MenuItem Header="Option 2" />
        </ContextMenu>
    </Grid.ContextMenu>

    <StackPanel Button.Click="ToutBouton_Click">
        <Button Content="Bouton 1" />
        <Button Content="Bouton 2" />
        <Button Content="Bouton 3" />
        <Grid>
            <Button Content="Bouton dans une Grid" />
        </Grid>
    </StackPanel>
</Grid>
```


```textmate
// MainWindow.xaml.cs
private void ToutBouton_Click(object sender, RoutedEventArgs e)
{
    // sender = l'objet qui a reçu l'événement (la StackPanel)
    // e.Source = l'objet qui a déclenché l'événement (le bouton spécifique)

    Button button = (Button)e.Source;
    MessageBox.Show($"Vous avez cliqué sur : {button.Content}");
}
```


### Événements tunnelants

Les événements tunnelants sont nommés avec un préfixe "Preview", comme `PreviewMouseDown`. Ils se produisent avant les événements de bulles correspondants.

```textmate
public class MonZoneTexte : TextBox
{
    protected override void OnPreviewKeyDown(KeyEventArgs e)
    {
        base.OnPreviewKeyDown(e);

        // Bloquer certaines touches
        if (e.Key == Key.Space)
        {
            e.Handled = true; // Empêche l'événement de continuer et bloque l'espace
        }
    }
}
```


## 8.10.5. Attached properties

Les propriétés attachées (attached properties) sont des propriétés qui peuvent être "attachées" à des objets qui ne les définissent pas eux-mêmes. Elles sont largement utilisées dans WPF pour la mise en page et d'autres fonctionnalités.

### Définition d'une propriété attachée

```textmate
public class InfoBulle
{
    // 1. Définir un champ statique pour la propriété
    public static readonly DependencyProperty TexteProperty =
        DependencyProperty.RegisterAttached(
            // Nom de la propriété sans le "Get" ou "Set"
            "Texte",
            // Type de la propriété
            typeof(string),
            // Type du propriétaire
            typeof(InfoBulle),
            // Métadonnées
            new PropertyMetadata(string.Empty, OnTexteChanged)
        );

    // 2. Méthodes d'accesseur Get/Set (obligatoires pour les propriétés attachées)
    public static string GetTexte(DependencyObject obj)
    {
        return (string)obj.GetValue(TexteProperty);
    }

    public static void SetTexte(DependencyObject obj, string value)
    {
        obj.SetValue(TexteProperty, value);
    }

    // 3. Gestionnaire de changement
    private static void OnTexteChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        // Si l'objet est un FrameworkElement
        if (d is FrameworkElement element)
        {
            string nouveauTexte = (string)e.NewValue;

            if (!string.IsNullOrEmpty(nouveauTexte))
            {
                // Créer une infobulle si elle n'existe pas encore
                if (element.ToolTip == null || !(element.ToolTip is ToolTip))
                {
                    element.ToolTip = new ToolTip();
                }

                // Définir le contenu de l'infobulle
                ((ToolTip)element.ToolTip).Content = nouveauTexte;
            }
        }
    }
}
```


### Utilisation d'une propriété attachée

```xml
<Window x:Class="MonApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="clr-namespace:MonApplication.Controls"
        Title="Test de propriétés attachées" Height="450" Width="800">
    <StackPanel Margin="20">
        <!-- Utilisation de la propriété attachée -->
        <Button Content="Bouton avec infobulle"
                controls:InfoBulle.Texte="Ceci est une infobulle personnalisée"
                Width="200" Height="40" Margin="0,0,0,20" />

        <TextBlock Text="Élément avec infobulle"
                   controls:InfoBulle.Texte="Une autre infobulle"
                   Margin="0,0,0,20" />

        <CheckBox Content="Option avec infobulle"
                  controls:InfoBulle.Texte="Cochez cette case pour activer l'option X"
                  Margin="0,0,0,20" />
    </StackPanel>
</Window>
```


### Exemple de propriété attachée pour le positionnement

Les propriétés attachées sont souvent utilisées pour influencer la mise en page. Voici un exemple qui permet de positionner des éléments dans une grille sans avoir à utiliser `Grid.Row` et `Grid.Column` :

```textmate
public class GridHelpers
{
    #region Position (attachment for X,Y coordinates)

    // Propriété attachée pour la coordonnée X (colonne)
    public static readonly DependencyProperty XProperty =
        DependencyProperty.RegisterAttached(
            "X",
            typeof(int),
            typeof(GridHelpers),
            new PropertyMetadata(0, PositionChanged));

    // Propriété attachée pour la coordonnée Y (ligne)
    public static readonly DependencyProperty YProperty =
        DependencyProperty.RegisterAttached(
            "Y",
            typeof(int),
            typeof(GridHelpers),
            new PropertyMetadata(0, PositionChanged));

    // Accesseurs pour X
    public static int GetX(DependencyObject obj)
    {
        return (int)obj.GetValue(XProperty);
    }

    public static void SetX(DependencyObject obj, int value)
    {
        obj.SetValue(XProperty, value);
    }

    // Accesseurs pour Y
    public static int GetY(DependencyObject obj)
    {
        return (int)obj.GetValue(YProperty);
    }

    public static void SetY(DependencyObject obj, int value)
    {
        obj.SetValue(YProperty, value);
    }

    // Gestionnaire de changement
    private static void PositionChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is UIElement element)
        {
            // Mettre à jour Grid.Row et Grid.Column
            if (e.Property == XProperty)
            {
                Grid.SetColumn(element, (int)e.NewValue);
            }
            else if (e.Property == YProperty)
            {
                Grid.SetRow(element, (int)e.NewValue);
            }
        }
    }

    #endregion

    #region Span (attachment for ColumnSpan and RowSpan)

    // Propriété attachée pour la portée horizontale (ColumnSpan)
    public static readonly DependencyProperty SpanXProperty =
        DependencyProperty.RegisterAttached(
            "SpanX",
            typeof(int),
            typeof(GridHelpers),
            new PropertyMetadata(1, SpanChanged));

    // Propriété attachée pour la portée verticale (RowSpan)
    public static readonly DependencyProperty SpanYProperty =
        DependencyProperty.RegisterAttached(
            "SpanY",
            typeof(int),
            typeof(GridHelpers),
            new PropertyMetadata(1, SpanChanged));

    // Accesseurs pour SpanX
    public static int GetSpanX(DependencyObject obj)
    {
        return (int)obj.GetValue(SpanXProperty);
    }

    public static void SetSpanX(DependencyObject obj, int value)
    {
        obj.SetValue(SpanXProperty, value);
    }

    // Accesseurs pour SpanY
    public static int GetSpanY(DependencyObject obj)
    {
        return (int)obj.GetValue(SpanYProperty);
    }

    public static void SetSpanY(DependencyObject obj, int value)
    {
        obj.SetValue(SpanYProperty, value);
    }

    // Gestionnaire de changement
    private static void SpanChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is UIElement element)
        {
            // Mettre à jour ColumnSpan et RowSpan
            if (e.Property == SpanXProperty)
            {
                Grid.SetColumnSpan(element, (int)e.NewValue);
            }
            else if (e.Property == SpanYProperty)
            {
                Grid.SetRowSpan(element, (int)e.NewValue);
            }
        }
    }

    #endregion
}
```


Utilisation de ces propriétés attachées :

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition Height="*" />
        <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="2*" />
        <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>

    <!-- Utilisation des propriétés attachées au lieu de Grid.Row et Grid.Column -->
    <TextBlock Text="En-tête"
               controls:GridHelpers.X="0"
               controls:GridHelpers.Y="0"
               controls:GridHelpers.SpanX="3"
               FontSize="18"
               HorizontalAlignment="Center" />

    <Button Content="Menu"
            controls:GridHelpers.X="0"
            controls:GridHelpers.Y="1" />

    <TextBlock Text="Contenu principal"
               controls:GridHelpers.X="1"
               controls:GridHelpers.Y="1" />

    <StackPanel controls:GridHelpers.X="2"
                controls:GridHelpers.Y="1">
        <Button Content="Action 1" Margin="5" />
        <Button Content="Action 2" Margin="5" />
    </StackPanel>

    <TextBlock Text="Pied de page"
               controls:GridHelpers.X="0"
               controls:GridHelpers.Y="2"
               controls:GridHelpers.SpanX="3"
               HorizontalAlignment="Center" />
</Grid>
```


### Propriété attachée avec comportement

Les propriétés attachées peuvent également être utilisées pour ajouter des comportements à des éléments existants. Voici un exemple qui ajoute un comportement de focus automatique :

```textmate
public class FocusExtensions
{
    // Propriété pour activer le focus automatique
    public static readonly DependencyProperty IsFocusedProperty =
        DependencyProperty.RegisterAttached(
            "IsFocused",
            typeof(bool),
            typeof(FocusExtensions),
            new PropertyMetadata(false, OnIsFocusedChanged));

    public static bool GetIsFocused(DependencyObject obj)
    {
        return (bool)obj.GetValue(IsFocusedProperty);
    }

    public static void SetIsFocused(DependencyObject obj, bool value)
    {
        obj.SetValue(IsFocusedProperty, value);
    }

    // Gestionnaire qui applique le focus quand la propriété change
    private static void OnIsFocusedChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is UIElement element)
        {
            if ((bool)e.NewValue)
            {
                // Donner le focus à l'élément, mais pas immédiatement
                // car il pourrait ne pas être encore chargé dans l'arbre visuel
                element.Dispatcher.BeginInvoke(DispatcherPriority.Loaded, new Action(() =>
                {
                    if (element is Control control && control.Focusable)
                    {
                        control.Focus();

                        // Si c'est un TextBox, sélectionner tout le texte
                        if (control is TextBox textBox)
                        {
                            textBox.SelectAll();
                        }
                    }
                    else if (element is FrameworkElement frameworkElement)
                    {
                        frameworkElement.Focus();
                    }
                }));
            }
        }
    }

    // Propriété pour activer la sélection automatique dans un TextBox
    public static readonly DependencyProperty AutoSelectProperty =
        DependencyProperty.RegisterAttached(
            "AutoSelect",
            typeof(bool),
            typeof(FocusExtensions),
            new PropertyMetadata(false, OnAutoSelectChanged));

    public static bool GetAutoSelect(DependencyObject obj)
    {
        return (bool)obj.GetValue(AutoSelectProperty);
    }

    public static void SetAutoSelect(DependencyObject obj, bool value)
    {
        obj.SetValue(AutoSelectProperty, value);
    }

    // Gestionnaire qui configure la sélection automatique
    private static void OnAutoSelectChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is TextBox textBox && (bool)e.NewValue)
        {
            // S'abonner à l'événement GotFocus
            textBox.GotFocus += TextBox_GotFocus;

            // Se désabonner si la propriété est désactivée
            if (!(bool)e.NewValue)
            {
                textBox.GotFocus -= TextBox_GotFocus;
            }
        }
    }

    private static void TextBox_GotFocus(object sender, RoutedEventArgs e)
    {
        if (sender is TextBox textBox)
        {
            textBox.SelectAll();
        }
    }
}
```


Utilisation de ces propriétés attachées :

```xml
<StackPanel>
    <!-- Ce TextBox reçoit le focus automatiquement au chargement -->
    <TextBox controls:FocusExtensions.IsFocused="True"
             Width="200" Margin="5" />

    <!-- Ce TextBox sélectionne tout son contenu quand il reçoit le focus -->
    <TextBox Text="Texte présélectionné"
             controls:FocusExtensions.AutoSelect="True"
             Width="200" Margin="5" />
</StackPanel>
```


### Propriété attachée pour validation (.NET 8)

Dans .NET 8, vous pouvez utiliser des propriétés attachées pour implémenter des mécanismes de validation avancés :

```textmate
public class ValidationHelpers
{
    // Propriété pour les règles de validation d'email
    public static readonly DependencyProperty IsEmailProperty =
        DependencyProperty.RegisterAttached(
            "IsEmail",
            typeof(bool),
            typeof(ValidationHelpers),
            new PropertyMetadata(false, OnIsEmailChanged));

    public static bool GetIsEmail(DependencyObject obj)
    {
        return (bool)obj.GetValue(IsEmailProperty);
    }

    public static void SetIsEmail(DependencyObject obj, bool value)
    {
        obj.SetValue(IsEmailProperty, value);
    }

    // Gestionnaire qui configure la validation d'email
    private static void OnIsEmailChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is TextBox textBox && (bool)e.NewValue)
        {
            // Ajouter un gestionnaire de validation
            textBox.TextChanged += EmailTextBox_TextChanged;

            // Définir un texte d'indication (placeholder)
            if (textBox is System.Windows.Controls.TextBox wpfTextBox)
            {
                wpfTextBox.SetValue(ToolTipService.ToolTipProperty, "Entrez une adresse email valide");
            }

            // Mettre à jour l'état initial
            ValidateEmail(textBox);
        }
    }

    private static void EmailTextBox_TextChanged(object sender, TextChangedEventArgs e)
    {
        if (sender is TextBox textBox)
        {
            ValidateEmail(textBox);
        }
    }

    private static void ValidateEmail(TextBox textBox)
    {
        string email = textBox.Text;
        bool isValid = !string.IsNullOrEmpty(email) &&
                      email.Contains('@') &&
                      email.Contains('.') &&
                      email.IndexOf('@') < email.LastIndexOf('.');

        // Mettre à jour l'apparence en fonction de la validation
        if (isValid)
        {
            textBox.BorderBrush = Brushes.Green;
            textBox.Background = new SolidColorBrush(Color.FromArgb(20, 0, 255, 0));
            textBox.SetValue(ToolTipService.ToolTipProperty, "Email valide");
        }
        else
        {
            textBox.BorderBrush = Brushes.Red;
            textBox.Background = new SolidColorBrush(Color.FromArgb(20, 255, 0, 0));
            textBox.SetValue(ToolTipService.ToolTipProperty, "Entrez une adresse email valide");
        }
    }

    // Propriété pour la validation d'un champ obligatoire
    public static readonly DependencyProperty IsRequiredProperty =
        DependencyProperty.RegisterAttached(
            "IsRequired",
            typeof(bool),
            typeof(ValidationHelpers),
            new PropertyMetadata(false, OnIsRequiredChanged));

    public static bool GetIsRequired(DependencyObject obj)
    {
        return (bool)obj.GetValue(IsRequiredProperty);
    }

    public static void SetIsRequired(DependencyObject obj, bool value)
    {
        obj.SetValue(IsRequiredProperty, value);
    }

    // Gestionnaire qui configure la validation de champ obligatoire
    private static void OnIsRequiredChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is TextBox textBox && (bool)e.NewValue)
        {
            // Ajouter un gestionnaire de validation
            textBox.TextChanged += RequiredTextBox_TextChanged;

            // Ajouter un indicateur visuel (astérisque rouge)
            if (textBox.Parent is Grid parentGrid)
            {
                int column = Grid.GetColumn(textBox);
                int row = Grid.GetRow(textBox);

                TextBlock requiredIndicator = new TextBlock
                {
                    Text = "*",
                    Foreground = Brushes.Red,
                    FontWeight = FontWeights.Bold,
                    VerticalAlignment = VerticalAlignment.Top,
                    Margin = new Thickness(-5, 0, 0, 0)
                };

                Grid.SetColumn(requiredIndicator, column);
                Grid.SetRow(requiredIndicator, row);

                parentGrid.Children.Add(requiredIndicator);
            }

            // Mettre à jour l'état initial
            ValidateRequired(textBox);
        }
    }

    private static void RequiredTextBox_TextChanged(object sender, TextChangedEventArgs e)
    {
        if (sender is TextBox textBox)
        {
            ValidateRequired(textBox);
        }
    }

    private static void ValidateRequired(TextBox textBox)
    {
        bool isValid = !string.IsNullOrWhiteSpace(textBox.Text);

        // Mettre à jour l'apparence en fonction de la validation
        if (isValid)
        {
            textBox.BorderBrush = new SolidColorBrush(Color.FromRgb(171, 173, 179)); // Couleur par défaut
            textBox.Background = Brushes.White;
        }
        else
        {
            textBox.BorderBrush = Brushes.Red;
            textBox.Background = new SolidColorBrush(Color.FromArgb(20, 255, 0, 0));
            textBox.SetValue(ToolTipService.ToolTipProperty, "Ce champ est obligatoire");
        }
    }
}
```


Utilisation des propriétés attachées de validation :

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition Height="Auto" />
        <RowDefinition Height="Auto" />
        <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>

    <!-- Étiquettes -->
    <TextBlock Text="Nom :" Grid.Row="0" Grid.Column="0" Margin="5" VerticalAlignment="Center" />
    <TextBlock Text="Email :" Grid.Row="1" Grid.Column="0" Margin="5" VerticalAlignment="Center" />
    <TextBlock Text="Téléphone :" Grid.Row="2" Grid.Column="0" Margin="5" VerticalAlignment="Center" />

    <!-- Champs avec validation -->
    <TextBox Grid.Row="0" Grid.Column="1" Margin="5"
             controls:ValidationHelpers.IsRequired="True" />

    <TextBox Grid.Row="1" Grid.Column="1" Margin="5"
             controls:ValidationHelpers.IsEmail="True"
             controls:ValidationHelpers.IsRequired="True" />

    <TextBox Grid.Row="2" Grid.Column="1" Margin="5" />

    <Button Content="Envoyer" Grid.Row="3" Grid.Column="1" Margin="5"
            HorizontalAlignment="Right" Width="100" />
</Grid>
```


--

La personnalisation des contrôles dans WPF offre une grande flexibilité pour créer des interfaces utilisateur riches et interactives. En comprenant les concepts clés comme les propriétés de dépendance, les événements routés et les propriétés attachées, vous pouvez créer des contrôles qui répondent exactement à vos besoins.

Pour résumer les approches de personnalisation des contrôles :

1. **Composition (UserControl)** - Idéal pour regrouper des contrôles existants en unités réutilisables.
2. **Héritage** - Étendre un contrôle existant pour ajouter ou modifier des fonctionnalités.
3. **Propriétés de dépendance** - Créer des propriétés qui prennent en charge la liaison de données, l'animation et d'autres fonctionnalités WPF.
4. **Événements routés** - Créer des événements qui peuvent se propager à travers l'arbre visuel.
5. **Propriétés attachées** - Ajouter des fonctionnalités à des contrôles existants sans utiliser l'héritage.

En combinant ces techniques, vous pouvez créer des contrôles personnalisés riches qui offrent une excellente expérience utilisateur tout en maintenant une base de code propre et réutilisable.

⏭️ 8.11. [Localisation et internationalisation](/08-windows-presentation-foundation-wpf/8-11-localisation-et-internationalisation.md)
