# 8.5. Styles et templates

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les styles et templates constituent l'un des aspects les plus puissants de WPF. Ils permettent de personnaliser complètement l'apparence des contrôles tout en préservant leur comportement, facilitant ainsi la création d'interfaces utilisateur cohérentes et personnalisées.

## 8.5.1. Définition et application de styles

Un style en WPF est un ensemble de valeurs de propriétés qui peuvent être appliquées à plusieurs éléments, permettant de maintenir une apparence cohérente et de réduire la duplication de code XAML.

### Syntaxe de base d'un style

```xml
<Style TargetType="Button">
    <Setter Property="Background" Value="Blue"/>
    <Setter Property="Foreground" Value="White"/>
    <Setter Property="Padding" Value="10,5"/>
    <Setter Property="FontWeight" Value="Bold"/>
    <Setter Property="Margin" Value="5"/>
</Style>
```


### Définir des styles dans les ressources

Les styles sont généralement définis dans une collection de ressources pour être réutilisables.

```xml
<Window.Resources>
    <!-- Style pour les boutons -->
    <Style x:Key="PrimaryButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#007bff"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="Margin" Value="5"/>
    </Style>

    <!-- Style pour les TextBox -->
    <Style x:Key="StandardTextBoxStyle" TargetType="TextBox">
        <Setter Property="BorderBrush" Value="#ced4da"/>
        <Setter Property="Padding" Value="8,5"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Margin" Value="5"/>
    </Style>
</Window.Resources>
```


### Application des styles

```xml
<!-- Application explicite par référence à la clé -->
<Button Content="Enregistrer" Style="{StaticResource PrimaryButtonStyle}"/>
<TextBox Text="Exemple" Style="{StaticResource StandardTextBoxStyle}"/>

<!-- Application implicite (sans clé) - s'applique à tous les boutons -->
<Window.Resources>
    <Style TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="Padding" Value="10,5"/>
        <Setter Property="Margin" Value="5"/>
    </Style>
</Window.Resources>

<StackPanel>
    <!-- Ces boutons utiliseront automatiquement le style défini ci-dessus -->
    <Button Content="Bouton 1"/>
    <Button Content="Bouton 2"/>
    <Button Content="Bouton 3"/>
</StackPanel>
```


### Styles avec gestionnaires d'événements

```xml
<Window.Resources>
    <Style x:Key="HoverButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Padding" Value="10,5"/>
        <Setter Property="Margin" Value="5"/>
        <EventSetter Event="MouseEnter" Handler="Button_MouseEnter"/>
        <EventSetter Event="MouseLeave" Handler="Button_MouseLeave"/>
    </Style>
</Window.Resources>
```


Code-behind associé :

```textmate
private void Button_MouseEnter(object sender, MouseEventArgs e)
{
    if (sender is Button button)
    {
        button.Background = new SolidColorBrush(Colors.LightBlue);
    }
}

private void Button_MouseLeave(object sender, MouseEventArgs e)
{
    if (sender is Button button)
    {
        button.Background = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#f8f9fa"));
    }
}
```


### Création de bibliothèque de styles cohérente

```xml
<!-- ResourceDictionary dans App.xaml -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="/Styles/Colors.xaml"/>
            <ResourceDictionary Source="/Styles/Buttons.xaml"/>
            <ResourceDictionary Source="/Styles/TextElements.xaml"/>
            <ResourceDictionary Source="/Styles/PanelStyles.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```


Dans un fichier `Colors.xaml` :

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- Palette de couleurs -->
    <Color x:Key="PrimaryColor">#007bff</Color>
    <Color x:Key="SecondaryColor">#6c757d</Color>
    <Color x:Key="SuccessColor">#28a745</Color>
    <Color x:Key="DangerColor">#dc3545</Color>
    <Color x:Key="WarningColor">#ffc107</Color>
    <Color x:Key="InfoColor">#17a2b8</Color>
    <Color x:Key="LightColor">#f8f9fa</Color>
    <Color x:Key="DarkColor">#343a40</Color>

    <!-- Brosses solides -->
    <SolidColorBrush x:Key="PrimaryBrush" Color="{StaticResource PrimaryColor}"/>
    <SolidColorBrush x:Key="SecondaryBrush" Color="{StaticResource SecondaryColor}"/>
    <SolidColorBrush x:Key="SuccessBrush" Color="{StaticResource SuccessColor}"/>
    <SolidColorBrush x:Key="DangerBrush" Color="{StaticResource DangerColor}"/>
    <SolidColorBrush x:Key="WarningBrush" Color="{StaticResource WarningColor}"/>
    <SolidColorBrush x:Key="InfoBrush" Color="{StaticResource InfoColor}"/>
    <SolidColorBrush x:Key="LightBrush" Color="{StaticResource LightColor}"/>
    <SolidColorBrush x:Key="DarkBrush" Color="{StaticResource DarkColor}"/>
</ResourceDictionary>
```


Dans un fichier `Buttons.xaml` :

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- Style de base pour tous les boutons -->
    <Style x:Key="BaseButtonStyle" TargetType="Button">
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Setter Property="FontWeight" Value="SemiBold"/>
        <Setter Property="BorderThickness" Value="1"/>
    </Style>

    <!-- Bouton primaire -->
    <Style x:Key="PrimaryButton" TargetType="Button" BasedOn="{StaticResource BaseButtonStyle}">
        <Setter Property="Background" Value="{StaticResource PrimaryBrush}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="BorderBrush" Value="{StaticResource PrimaryBrush}"/>
    </Style>

    <!-- Bouton secondaire -->
    <Style x:Key="SecondaryButton" TargetType="Button" BasedOn="{StaticResource BaseButtonStyle}">
        <Setter Property="Background" Value="{StaticResource SecondaryBrush}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="BorderBrush" Value="{StaticResource SecondaryBrush}"/>
    </Style>

    <!-- Bouton de danger -->
    <Style x:Key="DangerButton" TargetType="Button" BasedOn="{StaticResource BaseButtonStyle}">
        <Setter Property="Background" Value="{StaticResource DangerBrush}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="BorderBrush" Value="{StaticResource DangerBrush}"/>
    </Style>
</ResourceDictionary>
```


## 8.5.2. Héritage de styles

L'héritage de styles permet de créer une hiérarchie de styles, où un style enfant hérite de toutes les propriétés d'un style parent et peut les remplacer ou en ajouter de nouvelles.

### Création d'un style de base

```xml
<Window.Resources>
    <!-- Style de base pour les contrôles d'entrée -->
    <Style x:Key="BaseInputStyle" TargetType="{x:Type Control}">
        <Setter Property="Margin" Value="5"/>
        <Setter Property="Padding" Value="8,5"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="BorderBrush" Value="#ced4da"/>
        <Setter Property="FontFamily" Value="Segoe UI"/>
        <Setter Property="FontSize" Value="12"/>
    </Style>

    <!-- Style pour TextBox qui hérite du style de base -->
    <Style x:Key="CustomTextBoxStyle" TargetType="TextBox" BasedOn="{StaticResource BaseInputStyle}">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#495057"/>
    </Style>

    <!-- Style pour PasswordBox qui hérite également du style de base -->
    <Style x:Key="CustomPasswordBoxStyle" TargetType="PasswordBox" BasedOn="{StaticResource BaseInputStyle}">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#495057"/>
        <Setter Property="PasswordChar" Value="●"/>
    </Style>

    <!-- Style pour ComboBox qui hérite et étend le style de base -->
    <Style x:Key="CustomComboBoxStyle" TargetType="ComboBox" BasedOn="{StaticResource BaseInputStyle}">
        <Setter Property="Background" Value="White"/>
        <Setter Property="Foreground" Value="#495057"/>
        <Setter Property="IsEditable" Value="False"/>
        <Setter Property="Height" Value="30"/>
    </Style>
</Window.Resources>
```


### Héritage de styles implicites

```xml
<Window.Resources>
    <!-- Style implicite pour les boutons -->
    <Style TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Padding" Value="10,5"/>
        <Setter Property="Margin" Value="5"/>
    </Style>

    <!-- Style implicite pour les boutons dans un GroupBox -->
    <Style TargetType="Button" BasedOn="{StaticResource {x:Type Button}}">
        <Setter Property="Background" Value="LightBlue"/>
        <Style.Resources>
            <Style TargetType="Border">
                <Setter Property="CornerRadius" Value="5"/>
            </Style>
        </Style.Resources>
    </Style>
</Window.Resources>

<StackPanel>
    <!-- Ce bouton utilise le style implicite normal -->
    <Button Content="Bouton standard"/>

    <GroupBox Header="Groupe">
        <!-- Ces boutons utilisent le style implicite pour les boutons dans un GroupBox -->
        <StackPanel>
            <Button Content="Bouton dans GroupBox 1"/>
            <Button Content="Bouton dans GroupBox 2"/>
        </StackPanel>
    </GroupBox>
</StackPanel>
```


### Styles multi-niveaux

```xml
<Window.Resources>
    <!-- Style de base -->
    <Style x:Key="BaseButtonStyle" TargetType="Button">
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Setter Property="FontWeight" Value="Normal"/>
        <Setter Property="BorderThickness" Value="1"/>
    </Style>

    <!-- Style intermédiaire qui hérite du style de base -->
    <Style x:Key="OutlinedButtonStyle" TargetType="Button" BasedOn="{StaticResource BaseButtonStyle}">
        <Setter Property="Background" Value="Transparent"/>
        <Setter Property="BorderThickness" Value="1"/>
    </Style>

    <!-- Styles spécifiques qui héritent du style intermédiaire -->
    <Style x:Key="OutlinedPrimaryButton" TargetType="Button" BasedOn="{StaticResource OutlinedButtonStyle}">
        <Setter Property="Foreground" Value="#007bff"/>
        <Setter Property="BorderBrush" Value="#007bff"/>
    </Style>

    <Style x:Key="OutlinedDangerButton" TargetType="Button" BasedOn="{StaticResource OutlinedButtonStyle}">
        <Setter Property="Foreground" Value="#dc3545"/>
        <Setter Property="BorderBrush" Value="#dc3545"/>
    </Style>
</Window.Resources>

<StackPanel>
    <Button Content="Bouton primaire" Style="{StaticResource OutlinedPrimaryButton}"/>
    <Button Content="Bouton danger" Style="{StaticResource OutlinedDangerButton}"/>
</StackPanel>
```


## 8.5.3. ControlTemplate

Le `ControlTemplate` définit la structure visuelle d'un contrôle. En redéfinissant le template, vous pouvez complètement changer l'apparence d'un contrôle tout en conservant son comportement.

### Template de base pour un bouton

```xml
<Window.Resources>
    <!-- Template personnalisé pour un bouton -->
    <ControlTemplate x:Key="RoundedButtonTemplate" TargetType="Button">
        <Border x:Name="border"
                Background="{TemplateBinding Background}"
                BorderBrush="{TemplateBinding BorderBrush}"
                BorderThickness="{TemplateBinding BorderThickness}"
                CornerRadius="20">
            <ContentPresenter HorizontalAlignment="Center"
                              VerticalAlignment="Center"
                              Margin="{TemplateBinding Padding}"/>
        </Border>
        <ControlTemplate.Triggers>
            <Trigger Property="IsMouseOver" Value="True">
                <Setter TargetName="border" Property="Background" Value="LightBlue"/>
            </Trigger>
            <Trigger Property="IsPressed" Value="True">
                <Setter TargetName="border" Property="Background" Value="DarkBlue"/>
                <Setter TargetName="border" Property="BorderBrush" Value="Navy"/>
            </Trigger>
            <Trigger Property="IsEnabled" Value="False">
                <Setter TargetName="border" Property="Background" Value="LightGray"/>
                <Setter TargetName="border" Property="BorderBrush" Value="Gray"/>
            </Trigger>
        </ControlTemplate.Triggers>
    </ControlTemplate>

    <!-- Style qui utilise le template personnalisé -->
    <Style x:Key="RoundedButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#007bff"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="BorderBrush" Value="#0062cc"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Template" Value="{StaticResource RoundedButtonTemplate}"/>
    </Style>
</Window.Resources>

<StackPanel>
    <Button Content="Bouton avec coins arrondis"
            Style="{StaticResource RoundedButtonStyle}"
            Width="200"
            Margin="10"/>
</StackPanel>
```


### Template avancé avec états visuels

```xml
<Window.Resources>
    <!-- Template avancé pour un bouton moderne -->
    <ControlTemplate x:Key="ModernButtonTemplate" TargetType="Button">
        <Grid>
            <!-- Ombre portée -->
            <Border x:Name="shadow"
                    Background="#22000000"
                    CornerRadius="5"
                    Margin="3,3,0,0"
                    Opacity="0.5"/>

            <!-- Fond du bouton -->
            <Border x:Name="background"
                    Background="{TemplateBinding Background}"
                    BorderBrush="{TemplateBinding BorderBrush}"
                    BorderThickness="{TemplateBinding BorderThickness}"
                    CornerRadius="5">

                <!-- Effet de superposition -->
                <Border x:Name="overlay"
                        Background="Transparent"
                        CornerRadius="5">
                    <ContentPresenter x:Name="contentPresenter"
                                     HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                                     VerticalAlignment="{TemplateBinding VerticalContentAlignment}"
                                     Margin="{TemplateBinding Padding}"/>
                </Border>
            </Border>
        </Grid>
        <ControlTemplate.Triggers>
            <Trigger Property="IsMouseOver" Value="True">
                <Setter TargetName="overlay" Property="Background" Value="#33FFFFFF"/>
            </Trigger>
            <Trigger Property="IsPressed" Value="True">
                <Setter TargetName="shadow" Property="Opacity" Value="0"/>
                <Setter TargetName="background" Property="Margin" Value="3,3,0,0"/>
                <Setter TargetName="overlay" Property="Background" Value="#33000000"/>
            </Trigger>
            <Trigger Property="IsEnabled" Value="False">
                <Setter TargetName="shadow" Property="Opacity" Value="0"/>
                <Setter TargetName="background" Property="Background" Value="#EEEEEE"/>
                <Setter TargetName="background" Property="BorderBrush" Value="#CCCCCC"/>
                <Setter TargetName="contentPresenter" Property="TextElement.Foreground" Value="#999999"/>
            </Trigger>
        </ControlTemplate.Triggers>
    </ControlTemplate>

    <!-- Style qui utilise le template moderne -->
    <Style x:Key="ModernButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#4CAF50"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="BorderBrush" Value="#388E3C"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="Padding" Value="20,10"/>
        <Setter Property="Margin" Value="10"/>
        <Setter Property="Template" Value="{StaticResource ModernButtonTemplate}"/>
    </Style>
</Window.Resources>

<StackPanel>
    <Button Content="Bouton Moderne"
            Style="{StaticResource ModernButtonStyle}"
            Width="200"/>

    <Button Content="Bouton Désactivé"
            Style="{StaticResource ModernButtonStyle}"
            Width="200"
            IsEnabled="False"/>
</StackPanel>
```


### Remplacement de template pour les contrôles standard

```xml
<Window.Resources>
    <!-- Template personnalisé pour un CheckBox -->
    <ControlTemplate x:Key="ToggleSwitchTemplate" TargetType="CheckBox">
        <Grid>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>

            <!-- Fond du contrôle de type interrupteur -->
            <Border x:Name="SwitchTrack"
                    Width="40"
                    Height="20"
                    CornerRadius="10"
                    Background="#ccc"
                    Margin="0,0,10,0">
                <!-- Bouton de l'interrupteur -->
                <Border x:Name="SwitchThumb"
                        Width="16"
                        Height="16"
                        CornerRadius="8"
                        Background="White"
                        HorizontalAlignment="Left"
                        Margin="2,0,0,0"/>
            </Border>

            <!-- Contenu du CheckBox -->
            <ContentPresenter Grid.Column="1"
                              VerticalAlignment="Center"
                              Content="{TemplateBinding Content}"/>
        </Grid>
        <ControlTemplate.Triggers>
            <Trigger Property="IsChecked" Value="True">
                <Setter TargetName="SwitchTrack" Property="Background" Value="#2196F3"/>
                <Setter TargetName="SwitchThumb" Property="HorizontalAlignment" Value="Right"/>
                <Setter TargetName="SwitchThumb" Property="Margin" Value="0,0,2,0"/>
            </Trigger>
            <Trigger Property="IsEnabled" Value="False">
                <Setter TargetName="SwitchTrack" Property="Opacity" Value="0.5"/>
                <Setter TargetName="SwitchThumb" Property="Opacity" Value="0.5"/>
            </Trigger>
        </ControlTemplate.Triggers>
    </ControlTemplate>

    <!-- Style qui utilise le template d'interrupteur -->
    <Style x:Key="ToggleSwitchStyle" TargetType="CheckBox">
        <Setter Property="Template" Value="{StaticResource ToggleSwitchTemplate}"/>
        <Setter Property="Padding" Value="0"/>
        <Setter Property="VerticalAlignment" Value="Center"/>
    </Style>
</Window.Resources>

<StackPanel Margin="20">
    <CheckBox Content="Activer la notification"
              Style="{StaticResource ToggleSwitchStyle}"
              Margin="0,0,0,10"/>

    <CheckBox Content="Mode sombre"
              Style="{StaticResource ToggleSwitchStyle}"
              IsChecked="True"
              Margin="0,0,0,10"/>

    <CheckBox Content="Option désactivée"
              Style="{StaticResource ToggleSwitchStyle}"
              IsEnabled="False"/>
</StackPanel>
```


### Personnalisation avancée de ProgressBar

```xml
<Window.Resources>
    <!-- Template pour une barre de progression avec dégradé -->
    <ControlTemplate x:Key="GradientProgressBarTemplate" TargetType="ProgressBar">
        <Grid x:Name="TemplateRoot" SnapsToDevicePixels="true">
            <Border BorderBrush="{TemplateBinding BorderBrush}"
                    BorderThickness="{TemplateBinding BorderThickness}"
                    Background="{TemplateBinding Background}"
                    CornerRadius="8">
                <Border x:Name="PART_Track"
                        CornerRadius="8"
                        Background="#E0E0E0"/>
            </Border>
            <Border Name="PART_Indicator"
                    HorizontalAlignment="Left"
                    CornerRadius="8"
                    Margin="0">
                <Border.Background>
                    <LinearGradientBrush StartPoint="0,0" EndPoint="1,0">
                        <GradientStop Color="#2196F3" Offset="0"/>
                        <GradientStop Color="#4CAF50" Offset="1"/>
                    </LinearGradientBrush>
                </Border.Background>
            </Border>
            <TextBlock x:Name="ProgressText"
                       HorizontalAlignment="Center"
                       VerticalAlignment="Center"
                       Foreground="#333333"
                       FontWeight="Bold"/>
        </Grid>
        <ControlTemplate.Triggers>
            <Trigger Property="Orientation" Value="Vertical">
                <Setter TargetName="TemplateRoot" Property="LayoutTransform">
                    <Setter.Value>
                        <RotateTransform Angle="-90"/>
                    </Setter.Value>
                </Setter>
                <Setter TargetName="ProgressText" Property="LayoutTransform">
                    <Setter.Value>
                        <RotateTransform Angle="90"/>
                    </Setter.Value>
                </Setter>
            </Trigger>
            <DataTrigger Binding="{Binding Value, RelativeSource={RelativeSource Self}}" Value="0">
                <Setter TargetName="ProgressText" Property="Text" Value="0%"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding Value, RelativeSource={RelativeSource Self}}" Value="100">
                <Setter TargetName="ProgressText" Property="Text" Value="Terminé!"/>
            </DataTrigger>
        </ControlTemplate.Triggers>
    </ControlTemplate>

    <!-- Style pour la barre de progression personnalisée -->
    <Style x:Key="GradientProgressBarStyle" TargetType="ProgressBar">
        <Setter Property="Background" Value="Transparent"/>
        <Setter Property="BorderBrush" Value="Transparent"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="Height" Value="20"/>
        <Setter Property="Template" Value="{StaticResource GradientProgressBarTemplate}"/>
    </Style>
</Window.Resources>

<StackPanel Margin="20">
    <ProgressBar Style="{StaticResource GradientProgressBarStyle}"
                 Value="35"
                 Width="300"
                 Margin="0,0,0,20"/>

    <ProgressBar Style="{StaticResource GradientProgressBarStyle}"
                 Value="75"
                 Width="300"
                 Margin="0,0,0,20"/>

    <ProgressBar Style="{StaticResource GradientProgressBarStyle}"
                 Value="100"
                 Width="300"/>
</StackPanel>
```


## 8.5.4. DataTemplate

Les `DataTemplate` définissent la façon dont les données sont présentées visuellement, particulièrement utiles pour personnaliser l'affichage des éléments dans les contrôles basés sur des collections.

### DataTemplate de base

```xml
<Window.Resources>
    <!-- DataTemplate simple pour un objet Person -->
    <DataTemplate x:Key="PersonTemplate">
        <StackPanel Orientation="Horizontal" Margin="5">
            <Ellipse Width="32" Height="32" Fill="#1976D2" Margin="0,0,10,0"/>
            <StackPanel>
                <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
                <TextBlock Text="{Binding Email}" Foreground="#666666" FontSize="11"/>
            </StackPanel>
        </StackPanel>
    </DataTemplate>
</Window.Resources>

<ListBox ItemsSource="{Binding People}"
         ItemTemplate="{StaticResource PersonTemplate}"
         Margin="10"
         Width="300"/>
```


### DataTemplate avec sélecteur

```xml
<Window.Resources>
    <!-- Sélecteur de DataTemplate basé sur le type -->
    <DataTemplateSelector x:Key="ItemTemplateSelector">
        <!-- Implémentation en code-behind -->
    </DataTemplateSelector>

    <!-- Template pour les personnes -->
    <DataTemplate x:Key="PersonTemplate">
        <Border BorderBrush="#E0E0E0" BorderThickness="0,0,0,1" Padding="5">
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>
                <Ellipse Width="40" Height="40" Fill="#1976D2" Margin="0,0,10,0"/>
                <StackPanel Grid.Column="1">
                    <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
                    <TextBlock Text="{Binding Role}" Foreground="#666666"/>
                    <TextBlock Text="{Binding Email}" Foreground="#666666" FontSize="11"/>
                </StackPanel>
            </Grid>
        </Border>
    </DataTemplate>

    <!-- Template pour les projets -->
    <DataTemplate x:Key="ProjectTemplate">
        <Border BorderBrush="#E0E0E0" BorderThickness="0,0,0,1" Padding="5">
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <Rectangle Width="40" Height="40" Fill="#FF5722" Margin="0,0,10,0"/>
                <StackPanel Grid.Column="1">
                    <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
                    <TextBlock Text="{Binding Description}" Foreground="#666666" TextWrapping="Wrap"/>
                </StackPanel>
                <TextBlock Grid.Column="2"
                           Text="{Binding Status}"
                           VerticalAlignment="Center"
                           Margin="10,0"/>
            </Grid>
        </Border>
    </DataTemplate>
</Window.Resources>

<ListView ItemsSource="{Binding Items}"
          ItemTemplateSelector="{StaticResource ItemTemplateSelector}"
          Margin="10"
          Width="500"
          BorderThickness="0"/>
```


Implémentation du sélecteur en code-behind :

```textmate
public class ItemTemplateSelector : DataTemplateSelector
{
    public override DataTemplate SelectTemplate(object item, DependencyObject container)
    {
        if (container is FrameworkElement element)
        {
            if (item is Person)
                return element.FindResource("PersonTemplate") as DataTemplate;
            else if (item is Project)
                return element.FindResource("ProjectTemplate") as DataTemplate;
        }

        return null;
    }
}
```


### DataTemplate pour ListView avec GridView

```xml
<Window.Resources>
    <!-- DataTemplate pour GridViewColumn -->
    <DataTemplate x:Key="StatusTemplate">
        <Border CornerRadius="5"
                Padding="5,2"
                Background="{Binding StatusColor}">
            <TextBlock Text="{Binding Status}"
                       Foreground="White"
                       FontSize="11"
                       FontWeight="Bold"
                       HorizontalAlignment="Center"/>
        </Border>
    </DataTemplate>
</Window.Resources>

<ListView ItemsSource="{Binding Tickets}" Margin="10" Height="300">
    <ListView.View>
        <GridView>
            <GridViewColumn Header="ID" Width="60"
                            DisplayMemberBinding="{Binding Id}"/>
            <GridViewColumn Header="Titre" Width="200"
                            DisplayMemberBinding="{Binding Title}"/>
            <GridViewColumn Header="Priorité" Width="80"
                            DisplayMemberBinding="{Binding Priority}"/>
            <GridViewColumn Header="Assigné à" Width="120"
                            DisplayMemberBinding="{Binding AssignedTo}"/>
            <GridViewColumn Header="Statut" Width="100"
                            CellTemplate="{StaticResource StatusTemplate}"/>
            <GridViewColumn Header="Date" Width="100"
                            DisplayMemberBinding="{Binding Date, StringFormat=\{0:dd/MM/yyyy\}}"/>
        </GridView>
    </ListView.View>
</ListView>
```


### DataTemplate hiérarchiques (HierarchicalDataTemplate)

```xml
<Window.Resources>
    <!-- Template pour les noeuds d'arbre -->
    <HierarchicalDataTemplate x:Key="FolderTemplate"
                             ItemsSource="{Binding Children}">
        <StackPanel Orientation="Horizontal">
            <!-- Icône qui change selon l'état IsExpanded -->
            <Image Width="16" Height="16">
                <Image.Style>
                    <Style TargetType="Image">
                        <Setter Property="Source" Value="/Images/folder-closed.png"/>
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding RelativeSource={RelativeSource AncestorType=TreeViewItem}, Path=IsExpanded}" Value="True">
                                <Setter Property="Source" Value="/Images/folder-open.png"/>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </Image.Style>
            </Image>
            <TextBlock Text="{Binding Name}" Margin="5,0,0,0"/>
        </StackPanel>
    </HierarchicalDataTemplate>

    <!-- Template pour les fichiers (noeuds feuilles) -->
    <DataTemplate x:Key="FileTemplate">
        <StackPanel Orientation="Horizontal">
            <Image Source="{Binding IconPath}" Width="16" Height="16"/>
            <TextBlock Text="{Binding Name}" Margin="5,0,0,0"/>
        </StackPanel>
    </DataTemplate>

    <!-- Sélecteur qui choisit le bon template selon le type -->
    <local:FileSystemTemplateSelector x:Key="FileSystemTemplateSelector"
                                     FolderTemplate="{StaticResource FolderTemplate}"
                                     FileTemplate="{StaticResource FileTemplate}"/>
</Window.Resources>

<TreeView ItemsSource="{Binding RootFolders}"
          ItemTemplateSelector="{StaticResource FileSystemTemplateSelector}"
          Width="300"
          Height="400"
          Margin="10"/>
```


Implémentation du sélecteur :

```textmate
public class FileSystemTemplateSelector : DataTemplateSelector
{
    public DataTemplate FolderTemplate { get; set; }
    public DataTemplate FileTemplate { get; set; }

    public override DataTemplate SelectTemplate(object item, DependencyObject container)
    {
        if (item is FolderItem)
            return FolderTemplate;
        else
            return FileTemplate;
    }
}
```


### DataTemplate avec ConditionalTemplate (.NET 8)

Dans .NET 8, vous pouvez utiliser une nouvelle fonctionnalité appelée `DataTemplateSelector` inline avec des expressions conditionnelles :

```xml
<ListBox ItemsSource="{Binding Items}" Margin="10">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <ContentControl Content="{Binding}">
                <ContentControl.ContentTemplate>
                    <DataTemplate DataType="{x:Type local:Person}">
                        <StackPanel Orientation="Horizontal">
                            <Ellipse Width="30" Height="30" Fill="Blue" Margin="0,0,5,0"/>
                            <TextBlock Text="{Binding Name}" VerticalAlignment="Center"/>
                        </StackPanel>
                    </DataTemplate>
                </ContentControl.ContentTemplate>
                <ContentControl.Style>
                    <Style TargetType="ContentControl">
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding RelativeSource={RelativeSource Self}, Path=Content}" Value="{x:Type local:Project}">
                                <Setter Property="ContentTemplate">
                                    <Setter.Value>
                                        <DataTemplate DataType="{x:Type local:Project}">
                                            <StackPanel Orientation="Horizontal">
                                                <Rectangle Width="30" Height="30" Fill="Green" Margin="0,0,5,0"/>
                                                <TextBlock Text="{Binding Name}" VerticalAlignment="Center"/>
                                            </StackPanel>
                                        </DataTemplate>
                                    </Setter.Value>
                                </Setter>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </ContentControl.Style>
            </ContentControl>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```


## 8.5.5. Triggers (Property, Data, Event)

Les triggers permettent de changer l'apparence ou le comportement d'un élément en fonction de conditions spécifiques.

### PropertyTrigger

```xml
<Window.Resources>
    <!-- Style avec Property Trigger -->
    <Style x:Key="HoverButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#212529"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Style.Triggers>
            <!-- Déclencheur basé sur la propriété IsMouseOver -->
            <Trigger Property="IsMouseOver" Value="True">
                <Setter Property="Background" Value="#e9ecef"/>
                <Setter Property="BorderBrush" Value="#ced4da"/>
            </Trigger>
            <!-- Déclencheur basé sur la propriété IsPressed -->
            <Trigger Property="IsPressed" Value="True">
                <Setter Property="Background" Value="#dee2e6"/>
                <Setter Property="BorderBrush" Value="#adb5bd"/>
            </Trigger>
            <!-- Déclencheur basé sur la propriété IsEnabled -->
            <Trigger Property="IsEnabled" Value="False">
                <Setter Property="Opacity" Value="0.5"/>
            </Trigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<StackPanel Margin="10">
    <Button Content="Bouton avec effets de survol"
            Style="{StaticResource HoverButtonStyle}"
            Width="200"
            Margin="0,0,0,10"/>

    <Button Content="Bouton désactivé"
            Style="{StaticResource HoverButtonStyle}"
            Width="200"
            IsEnabled="False"/>
</StackPanel>
```


### MultiTrigger

```xml
<Window.Resources>
    <!-- Style avec Multi Trigger -->
    <Style x:Key="SpecialStateButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#212529"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Style.Triggers>
            <!-- Déclencheur multiple qui exige plusieurs conditions -->
            <MultiTrigger>
                <MultiTrigger.Conditions>
                    <Condition Property="IsMouseOver" Value="True"/>
                    <Condition Property="IsEnabled" Value="True"/>
                </MultiTrigger.Conditions>
                <MultiTrigger.Setters>
                    <Setter Property="Background" Value="#007bff"/>
                    <Setter Property="Foreground" Value="White"/>
                </MultiTrigger.Setters>
            </MultiTrigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<StackPanel Margin="10">
    <Button Content="Bouton à conditions multiples"
            Style="{StaticResource SpecialStateButtonStyle}"
            Width="200"/>
</StackPanel>
```


### DataTrigger

```xml
<Window.Resources>
    <!-- Style avec Data Trigger -->
    <Style x:Key="PriorityIndicatorStyle" TargetType="TextBlock">
        <Setter Property="Padding" Value="5,2"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="Foreground" Value="White"/>
        <Style.Triggers>
            <!-- Déclencheurs basés sur la valeur liée -->
            <DataTrigger Binding="{Binding Priority}" Value="Haute">
                <Setter Property="Background" Value="#dc3545"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding Priority}" Value="Moyenne">
                <Setter Property="Background" Value="#fd7e14"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding Priority}" Value="Basse">
                <Setter Property="Background" Value="#28a745"/>
            </DataTrigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<ListBox ItemsSource="{Binding Tasks}" Margin="10">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <Grid Margin="5">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{Binding Title}" VerticalAlignment="Center"/>
                <TextBlock Grid.Column="1"
                           Text="{Binding Priority}"
                           Style="{StaticResource PriorityIndicatorStyle}"/>
            </Grid>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```


### MultiDataTrigger

```xml
<Window.Resources>
    <!-- Style avec Multi Data Trigger -->
    <Style x:Key="TaskStatusStyle" TargetType="TextBlock">
        <Setter Property="Padding" Value="5,2"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Style.Triggers>
            <!-- Déclencheur basé sur plusieurs conditions de données -->
            <MultiDataTrigger>
                <MultiDataTrigger.Conditions>
                    <Condition Binding="{Binding Status}" Value="En cours"/>
                    <Condition Binding="{Binding Priority}" Value="Haute"/>
                </MultiDataTrigger.Conditions>
                <MultiDataTrigger.Setters>
                    <Setter Property="Background" Value="#dc3545"/>
                    <Setter Property="Foreground" Value="White"/>
                    <Setter Property="Text" Value="URGENT"/>
                </MultiDataTrigger.Setters>
            </MultiDataTrigger>

            <MultiDataTrigger>
                <MultiDataTrigger.Conditions>
                    <Condition Binding="{Binding Status}" Value="Terminé"/>
                    <Condition Binding="{Binding IsVerified}" Value="True"/>
                </MultiDataTrigger.Conditions>
                <MultiDataTrigger.Setters>
                    <Setter Property="Background" Value="#28a745"/>
                    <Setter Property="Foreground" Value="White"/>
                    <Setter Property="Text" Value="VÉRIFIÉ"/>
                </MultiDataTrigger.Setters>
            </MultiDataTrigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<ItemsControl ItemsSource="{Binding Tasks}" Margin="10">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Grid Margin="5">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{Binding Title}" VerticalAlignment="Center"/>
                <TextBlock Grid.Column="1"
                           Text="{Binding Status}"
                           Style="{StaticResource TaskStatusStyle}"/>
            </Grid>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```


### EventTrigger avec animations

```xml
<Window.Resources>
    <!-- Style avec Event Trigger pour animations -->
    <Style x:Key="AnimatedButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Style.Triggers>
            <!-- Déclencheur basé sur l'événement MouseEnter -->
            <EventTrigger RoutedEvent="Button.MouseEnter">
                <BeginStoryboard>
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetProperty="(Button.Background).(SolidColorBrush.Color)"
                                        To="#007bff"
                                        Duration="0:0:0.2"/>
                        <ColorAnimation Storyboard.TargetProperty="(Button.Foreground).(SolidColorBrush.Color)"
                                        To="White"
                                        Duration="0:0:0.2"/>
                    </Storyboard>
                </BeginStoryboard>
            </EventTrigger>

            <!-- Déclencheur basé sur l'événement MouseLeave -->
            <EventTrigger RoutedEvent="Button.MouseLeave">
                <BeginStoryboard>
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetProperty="(Button.Background).(SolidColorBrush.Color)"
                                        To="#f8f9fa"
                                        Duration="0:0:0.3"/>
                        <ColorAnimation Storyboard.TargetProperty="(Button.Foreground).(SolidColorBrush.Color)"
                                        To="Black"
                                        Duration="0:0:0.3"/>
                    </Storyboard>
                </BeginStoryboard>
            </EventTrigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<StackPanel Margin="20">
    <Button Content="Bouton avec animation"
            Style="{StaticResource AnimatedButtonStyle}"
            Width="200"/>
</StackPanel>
```


### Triggers en cascade et priorité

```xml
<Window.Resources>
    <!-- Style avec plusieurs triggers -->
    <Style x:Key="ComplexButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#212529"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Style.Triggers>
            <!-- Les triggers sont évalués dans l'ordre - les derniers ont priorité -->
            <Trigger Property="IsMouseOver" Value="True">
                <Setter Property="Background" Value="#e9ecef"/>
                <Setter Property="BorderBrush" Value="#ced4da"/>
            </Trigger>

            <Trigger Property="IsPressed" Value="True">
                <Setter Property="Background" Value="#dee2e6"/>
                <Setter Property="BorderBrush" Value="#adb5bd"/>
            </Trigger>

            <Trigger Property="IsEnabled" Value="False">
                <Setter Property="Opacity" Value="0.5"/>
                <Setter Property="Background" Value="#f8f9fa"/>
            </Trigger>

            <!-- Ce trigger a priorité sur IsMouseOver car il est défini après -->
            <Trigger Property="Tag" Value="Primary">
                <Setter Property="Background" Value="#007bff"/>
                <Setter Property="Foreground" Value="White"/>
                <Setter Property="BorderBrush" Value="#0062cc"/>
            </Trigger>

            <!-- Même avec IsMouseOver="True", si le Tag est "Danger", ce style a priorité -->
            <MultiTrigger>
                <MultiTrigger.Conditions>
                    <Condition Property="Tag" Value="Danger"/>
                    <Condition Property="IsMouseOver" Value="True"/>
                </MultiTrigger.Conditions>
                <MultiTrigger.Setters>
                    <Setter Property="Background" Value="#bd2130"/>
                    <Setter Property="Foreground" Value="White"/>
                    <Setter Property="BorderBrush" Value="#bd2130"/>
                </MultiTrigger.Setters>
            </MultiTrigger>
        </Style.Triggers>
    </Style>
</Window.Resources>

<StackPanel Margin="10">
    <Button Content="Bouton normal"
            Style="{StaticResource ComplexButtonStyle}"
            Width="200"
            Margin="0,0,0,10"/>

    <Button Content="Bouton primaire"
            Style="{StaticResource ComplexButtonStyle}"
            Tag="Primary"
            Width="200"
            Margin="0,0,0,10"/>

    <Button Content="Bouton danger"
            Style="{StaticResource ComplexButtonStyle}"
            Tag="Danger"
            Width="200"
            Margin="0,0,0,10"/>

    <Button Content="Bouton désactivé"
            Style="{StaticResource ComplexButtonStyle}"
            Width="200"
            IsEnabled="False"/>
</StackPanel>
```


## 8.5.6. Visual States

Les Visual States permettent de définir différents états visuels pour un contrôle. WPF inclut un système d'états visuels intégré pour les contrôles standard, que vous pouvez personnaliser via les templates.

### Définition d'états visuels dans un ControlTemplate

```xml
<Window.Resources>
    <!-- Template avec Visual States -->
    <ControlTemplate x:Key="ModernButtonTemplate" TargetType="Button">
        <Border x:Name="border"
                Background="{TemplateBinding Background}"
                BorderBrush="{TemplateBinding BorderBrush}"
                BorderThickness="{TemplateBinding BorderThickness}"
                CornerRadius="5"
                Padding="{TemplateBinding Padding}">
            <ContentPresenter x:Name="contentPresenter"
                             HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                             VerticalAlignment="{TemplateBinding VerticalContentAlignment}"
                             Content="{TemplateBinding Content}"
                             ContentTemplate="{TemplateBinding ContentTemplate}"/>
        </Border>

        <ControlTemplate.Resources>
            <Storyboard x:Key="MouseOverAnimation">
                <ColorAnimation Storyboard.TargetName="border"
                                Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                To="#007bff"
                                Duration="0:0:0.2"/>
                <ColorAnimation Storyboard.TargetName="contentPresenter"
                                Storyboard.TargetProperty="(TextElement.Foreground).(SolidColorBrush.Color)"
                                To="White"
                                Duration="0:0:0.2"/>
            </Storyboard>

            <Storyboard x:Key="PressedAnimation">
                <ColorAnimation Storyboard.TargetName="border"
                                Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                To="#0062cc"
                                Duration="0:0:0.1"/>
                <ThicknessAnimation Storyboard.TargetName="border"
                                   Storyboard.TargetProperty="Margin"
                                   To="2,2,0,0"
                                   Duration="0:0:0.1"/>
            </Storyboard>

            <Storyboard x:Key="NormalAnimation">
                <ColorAnimation Storyboard.TargetName="border"
                                Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                To="#f8f9fa"
                                Duration="0:0:0.3"/>
                <ColorAnimation Storyboard.TargetName="contentPresenter"
                                Storyboard.TargetProperty="(TextElement.Foreground).(SolidColorBrush.Color)"
                                To="#212529"
                                Duration="0:0:0.3"/>
                <ThicknessAnimation Storyboard.TargetName="border"
                                   Storyboard.TargetProperty="Margin"
                                   To="0"
                                   Duration="0:0:0.1"/>
            </Storyboard>

            <Storyboard x:Key="DisabledAnimation">
                <DoubleAnimation Storyboard.TargetName="border"
                                 Storyboard.TargetProperty="Opacity"
                                 To="0.5"
                                 Duration="0:0:0.2"/>
            </Storyboard>
        </ControlTemplate.Resources>

        <ControlTemplate.Triggers>
            <Trigger Property="IsMouseOver" Value="True">
                <Trigger.EnterActions>
                    <BeginStoryboard Storyboard="{StaticResource MouseOverAnimation}"/>
                </Trigger.EnterActions>
                <Trigger.ExitActions>
                    <BeginStoryboard Storyboard="{StaticResource NormalAnimation}"/>
                </Trigger.ExitActions>
            </Trigger>

            <Trigger Property="IsPressed" Value="True">
                <Trigger.EnterActions>
                    <BeginStoryboard Storyboard="{StaticResource PressedAnimation}"/>
                </Trigger.EnterActions>
                <Trigger.ExitActions>
                    <BeginStoryboard Storyboard="{StaticResource NormalAnimation}"/>
                </Trigger.ExitActions>
            </Trigger>

            <Trigger Property="IsEnabled" Value="False">
                <Trigger.EnterActions>
                    <BeginStoryboard Storyboard="{StaticResource DisabledAnimation}"/>
                </Trigger.EnterActions>
            </Trigger>
        </ControlTemplate.Triggers>
    </ControlTemplate>

    <!-- Style qui utilise le template avec états visuels -->
    <Style x:Key="ModernButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#212529"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="5"/>
        <Setter Property="Template" Value="{StaticResource ModernButtonTemplate}"/>
    </Style>
</Window.Resources>

<StackPanel Margin="20">
    <Button Content="Bouton avec animations d'état"
            Style="{StaticResource ModernButtonStyle}"
            Width="250"
            Margin="0,0,0,10"/>

    <Button Content="Bouton désactivé"
            Style="{StaticResource ModernButtonStyle}"
            Width="250"
            IsEnabled="False"/>
</StackPanel>
```


### VisualStateManager

Le `VisualStateManager` est une approche plus déclarative et organisée pour définir les états visuels.

```xml
<Window.Resources>
    <!-- Template avec VisualStateManager -->
    <ControlTemplate x:Key="ModernToggleButtonTemplate" TargetType="ToggleButton">
        <Border x:Name="border"
                Background="{TemplateBinding Background}"
                BorderBrush="{TemplateBinding BorderBrush}"
                BorderThickness="{TemplateBinding BorderThickness}"
                CornerRadius="20"
                Padding="5">
            <Grid>
                <Ellipse x:Name="switchTrack"
                         Width="40"
                         Height="20"
                         Fill="#ccc"
                         HorizontalAlignment="Left"/>

                <Ellipse x:Name="switchThumb"
                         Width="16"
                         Height="16"
                         Fill="White"
                         HorizontalAlignment="Left"
                         Margin="2,0,0,0"/>

                <ContentPresenter x:Name="contentPresenter"
                                 Content="{TemplateBinding Content}"
                                 Margin="50,0,0,0"
                                 VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <ControlTemplate.Resources>
            <CubicEase x:Key="EaseOut" EasingMode="EaseOut"/>
        </ControlTemplate.Resources>

        <VisualStateManager.VisualStateGroups>
            <VisualStateGroup x:Name="CommonStates">
                <!-- État normal -->
                <VisualState x:Name="Normal"/>

                <!-- État survolé -->
                <VisualState x:Name="MouseOver">
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetName="switchTrack"
                                       Storyboard.TargetProperty="(Shape.Fill).(SolidColorBrush.Color)"
                                       To="#aaa"
                                       Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>

                <!-- État désactivé -->
                <VisualState x:Name="Disabled">
                    <Storyboard>
                        <DoubleAnimation Storyboard.TargetName="border"
                                        Storyboard.TargetProperty="Opacity"
                                        To="0.5"
                                        Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>
            </VisualStateGroup>

            <!-- Groupe pour les états Checked/Unchecked -->
            <VisualStateGroup x:Name="CheckStates">
                <!-- État décoché -->
                <VisualState x:Name="Unchecked"/>

                <!-- État coché -->
                <VisualState x:Name="Checked">
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetName="switchTrack"
                                       Storyboard.TargetProperty="(Shape.Fill).(SolidColorBrush.Color)"
                                       To="#2196F3"
                                       Duration="0:0:0.2"
                                       EasingFunction="{StaticResource EaseOut}"/>

                        <ThicknessAnimation Storyboard.TargetName="switchThumb"
                                           Storyboard.TargetProperty="Margin"
                                           To="22,0,0,0"
                                           Duration="0:0:0.2"
                                           EasingFunction="{StaticResource EaseOut}"/>
                    </Storyboard>
                </VisualState>

                <!-- État indéterminé (pour les ToggleButton à trois états) -->
                <VisualState x:Name="Indeterminate">
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetName="switchTrack"
                                       Storyboard.TargetProperty="(Shape.Fill).(SolidColorBrush.Color)"
                                       To="#9e9e9e"
                                       Duration="0:0:0.2"
                                       EasingFunction="{StaticResource EaseOut}"/>

                        <ThicknessAnimation Storyboard.TargetName="switchThumb"
                                           Storyboard.TargetProperty="Margin"
                                           To="12,0,0,0"
                                           Duration="0:0:0.2"
                                           EasingFunction="{StaticResource EaseOut}"/>
                    </Storyboard>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
    </ControlTemplate>

    <!-- Style qui utilise le template avec VisualStateManager -->
    <Style x:Key="ModernToggleStyle" TargetType="ToggleButton">
        <Setter Property="Background" Value="Transparent"/>
        <Setter Property="BorderBrush" Value="Transparent"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="Margin" Value="10"/>
        <Setter Property="Template" Value="{StaticResource ModernToggleButtonTemplate}"/>
    </Style>
</Window.Resources>

<StackPanel Margin="20">
    <ToggleButton Content="Option 1"
                  Style="{StaticResource ModernToggleStyle}"
                  Margin="0,0,0,10"/>

    <ToggleButton Content="Option 2"
                  Style="{StaticResource ModernToggleStyle}"
                  IsChecked="True"
                  Margin="0,0,0,10"/>

    <ToggleButton Content="Option 3 (indéterminé)"
                  Style="{StaticResource ModernToggleStyle}"
                  IsThreeState="True"
                  IsChecked="{x:Null}"
                  Margin="0,0,0,10"/>

    <ToggleButton Content="Option désactivée"
                  Style="{StaticResource ModernToggleStyle}"
                  IsEnabled="False"/>
</StackPanel>
```


### États visuels personnalisés

Vous pouvez également définir vos propres états visuels personnalisés.

```xml
<Window.Resources>
    <!-- Template avec états visuels personnalisés -->
    <ControlTemplate x:Key="CustomStateButtonTemplate" TargetType="Button">
        <Border x:Name="border"
                Background="{TemplateBinding Background}"
                BorderBrush="{TemplateBinding BorderBrush}"
                BorderThickness="{TemplateBinding BorderThickness}"
                CornerRadius="5"
                Padding="{TemplateBinding Padding}">
            <ContentPresenter x:Name="contentPresenter"
                             HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                             VerticalAlignment="{TemplateBinding VerticalContentAlignment}"/>
        </Border>

        <VisualStateManager.VisualStateGroups>
            <!-- Groupe d'états standard -->
            <VisualStateGroup x:Name="CommonStates">
                <VisualState x:Name="Normal"/>

                <VisualState x:Name="MouseOver">
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetName="border"
                                       Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                       To="#007bff"
                                       Duration="0:0:0.2"/>
                        <ColorAnimation Storyboard.TargetName="contentPresenter"
                                       Storyboard.TargetProperty="(TextElement.Foreground).(SolidColorBrush.Color)"
                                       To="White"
                                       Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>

                <VisualState x:Name="Pressed">
                    <Storyboard>
                        <ColorAnimation Storyboard.TargetName="border"
                                       Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                       To="#0062cc"
                                       Duration="0:0:0.1"/>
                        <ColorAnimation Storyboard.TargetName="contentPresenter"
                                       Storyboard.TargetProperty="(TextElement.Foreground).(SolidColorBrush.Color)"
                                       To="White"
                                       Duration="0:0:0.1"/>
                        <ThicknessAnimation Storyboard.TargetName="border"
                                           Storyboard.TargetProperty="Margin"
                                           To="2,2,0,0"
                                           Duration="0:0:0.1"/>
                    </Storyboard>
                </VisualState>

                <VisualState x:Name="Disabled">
                    <Storyboard>
                        <DoubleAnimation Storyboard.TargetName="border"
                                        Storyboard.TargetProperty="Opacity"
                                        To="0.5"
                                        Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>
            </VisualStateGroup>

            <!-- Groupe d'états personnalisés -->
            <VisualStateGroup x:Name="HighlightStates">
                <VisualState x:Name="NotHighlighted"/>

                <VisualState x:Name="Highlighted">
                    <Storyboard>
                        <DoubleAnimation Storyboard.TargetName="border"
                                        Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleX)"
                                        To="1.05"
                                        Duration="0:0:0.2"/>
                        <DoubleAnimation Storyboard.TargetName="border"
                                        Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleY)"
                                        To="1.05"
                                        Duration="0:0:0.2"/>
                        <ColorAnimation Storyboard.TargetName="border"
                                       Storyboard.TargetProperty="(Border.BorderBrush).(SolidColorBrush.Color)"
                                       To="Gold"
                                       Duration="0:0:0.2"/>
                        <ThicknessAnimation Storyboard.TargetName="border"
                                           Storyboard.TargetProperty="BorderThickness"
                                           To="2"
                                           Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
    </ControlTemplate>

    <!-- Style qui utilise le template avec états personnalisés -->
    <Style x:Key="HighlightableButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="#f8f9fa"/>
        <Setter Property="Foreground" Value="#212529"/>
        <Setter Property="BorderBrush" Value="#dee2e6"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="15,8"/>
        <Setter Property="Margin" Value="10"/>
        <Setter Property="Template" Value="{StaticResource CustomStateButtonTemplate}"/>
        <Setter Property="RenderTransformOrigin" Value="0.5,0.5"/>
        <Setter Property="RenderTransform">
            <Setter.Value>
                <ScaleTransform ScaleX="1" ScaleY="1"/>
            </Setter.Value>
        </Setter>
    </Style>
</Window.Resources>

<StackPanel Margin="20">
    <Button x:Name="highlightableButton"
            Content="Bouton avec état personnalisé"
            Style="{StaticResource HighlightableButtonStyle}"
            Width="250"
            Click="HighlightableButton_Click"/>
</StackPanel>
```


Le code-behind pour gérer le changement d'état personnalisé :

```textmate
private bool isHighlighted = false;

private void HighlightableButton_Click(object sender, RoutedEventArgs e)
{
    isHighlighted = !isHighlighted;

    // Changement d'état visuel personnalisé
    VisualStateManager.GoToState(highlightableButton,
                                isHighlighted ? "Highlighted" : "NotHighlighted",
                                true);
}
```


### Intégration du VisualStateManager avec le MVVM (.NET 8)

Dans .NET 8, vous pouvez utiliser le `VisualStateManager` avec le pattern MVVM en associant les états à des propriétés du ViewModel :

```xml
<Window.Resources>
    <Style x:Key="StatefulButtonStyle" TargetType="Button">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Border x:Name="border"
                            Background="{TemplateBinding Background}"
                            BorderBrush="{TemplateBinding BorderBrush}"
                            BorderThickness="{TemplateBinding BorderThickness}"
                            CornerRadius="5"
                            Padding="{TemplateBinding Padding}">
                        <ContentPresenter HorizontalAlignment="Center" VerticalAlignment="Center"/>
                    </Border>

                    <VisualStateManager.VisualStateGroups>
                        <VisualStateGroup x:Name="CommonStates">
                            <VisualState x:Name="Normal"/>

                            <VisualState x:Name="MouseOver">
                                <Storyboard>
                                    <ColorAnimation Storyboard.TargetName="border"
                                                  Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                                  To="#e9ecef"
                                                  Duration="0:0:0.2"/>
                                </Storyboard>
                            </VisualState>

                            <VisualState x:Name="Pressed">
                                <Storyboard>
                                    <ColorAnimation Storyboard.TargetName="border"
                                                  Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                                  To="#dee2e6"
                                                  Duration="0:0:0.1"/>
                                </Storyboard>
                            </VisualState>
                        </VisualStateGroup>

                        <VisualStateGroup x:Name="LoadingStates">
                            <VisualState x:Name="Idle"/>

                            <VisualState x:Name="Loading">
                                <Storyboard>
                                    <ObjectAnimationUsingKeyFrames Storyboard.TargetName="loadingIndicator"
                                                                  Storyboard.TargetProperty="Visibility">
                                        <DiscreteObjectKeyFrame KeyTime="0" Value="{x:Static Visibility.Visible}"/>
                                    </ObjectAnimationUsingKeyFrames>

                                    <ObjectAnimationUsingKeyFrames Storyboard.TargetName="contentPresenter"
                                                                  Storyboard.TargetProperty="Visibility">
                                        <DiscreteObjectKeyFrame KeyTime="0" Value="{x:Static Visibility.Collapsed}"/>
                                    </ObjectAnimationUsingKeyFrames>

                                    <DoubleAnimation Storyboard.TargetName="rotateTransform"
                                                   Storyboard.TargetProperty="Angle"
                                                   From="0"
                                                   To="360"
                                                   Duration="0:0:1"
                                                   RepeatBehavior="Forever"/>
                                </Storyboard>
                            </VisualState>

                            <VisualState x:Name="Success">
                                <Storyboard>
                                    <ColorAnimation Storyboard.TargetName="border"
                                                  Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                                  To="#28a745"
                                                  Duration="0:0:0.3"/>

                                    <ColorAnimation Storyboard.TargetName="contentPresenter"
                                                  Storyboard.TargetProperty="(TextElement.Foreground).(SolidColorBrush.Color)"
                                                  To="White"
                                                  Duration="0:0:0.3"/>
                                </Storyboard>
                            </VisualState>

                            <VisualState x:Name="Error">
                                <Storyboard>
                                    <ColorAnimation Storyboard.TargetName="border"
                                                  Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                                                  To="#dc3545"
                                                  Duration="0:0:0.3"/>

                                    <ColorAnimation Storyboard.TargetName="contentPresenter"
                                                  Storyboard.TargetProperty="(TextElement.Foreground).(SolidColorBrush.Color)"
                                                  To="White"
                                                  Duration="0:0:0.3"/>
                                </Storyboard>
                            </VisualState>
                        </VisualStateGroup>
                    </VisualStateManager.VisualStateGroups>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</Window.Resources>

<Button Content="Enregistrer"
        Style="{StaticResource StatefulButtonStyle}"
        Command="{Binding SaveCommand}"
        Width="150"
        Margin="10">
    <Button.Triggers>
        <DataTrigger Binding="{Binding IsLoading}" Value="True">
            <Setter Property="VisualStateManager.VisualStateGroups[1].CurrentState" Value="Loading"/>
        </DataTrigger>
        <DataTrigger Binding="{Binding IsSuccess}" Value="True">
            <Setter Property="VisualStateManager.VisualStateGroups[1].CurrentState" Value="Success"/>
        </DataTrigger>
        <DataTrigger Binding="{Binding IsError}" Value="True">
            <Setter Property="VisualStateManager.VisualStateGroups[1].CurrentState" Value="Error"/>
        </DataTrigger>
    </Button.Triggers>
</Button>
```


Le ViewModel associé pourrait ressembler à ceci :

```textmate
public class MainViewModel : INotifyPropertyChanged
{
    private bool _isLoading;
    private bool _isSuccess;
    private bool _isError;

    public bool IsLoading
    {
        get => _isLoading;
        set
        {
            _isLoading = value;
            OnPropertyChanged(nameof(IsLoading));
        }
    }

    public bool IsSuccess
    {
        get => _isSuccess;
        set
        {
            _isSuccess = value;
            OnPropertyChanged(nameof(IsSuccess));
        }
    }

    public bool IsError
    {
        get => _isError;
        set
        {
            _isError = value;
            OnPropertyChanged(nameof(IsError));
        }
    }

    public ICommand SaveCommand => new RelayCommand(async () =>
    {
        // Réinitialiser les états
        IsSuccess = false;
        IsError = false;

        // Définir l'état de chargement
        IsLoading = true;

        try
        {
            // Simuler une opération asynchrone
            await Task.Delay(2000);

            // Simuler une réussite ou un échec aléatoire
            var random = new Random();
            if (random.Next(2) == 0)
            {
                throw new Exception("Erreur simulée");
            }

            // Définir l'état de réussite
            IsLoading = false;
            IsSuccess = true;

            // Réinitialiser après un délai
            await Task.Delay(2000);
            IsSuccess = false;
        }
        catch
        {
            // Définir l'état d'erreur
            IsLoading = false;
            IsError = true;

            // Réinitialiser après un délai
            await Task.Delay(2000);
            IsError = false;
        }
    });

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


Les styles et templates en WPF offrent une grande flexibilité pour personnaliser l'apparence et le comportement des contrôles. En combinant les styles, les templates et les triggers, vous pouvez créer des interfaces utilisateur riches et réactives qui offrent une expérience utilisateur cohérente et attrayante.

⏭️
