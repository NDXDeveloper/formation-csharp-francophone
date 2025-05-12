# 8.4. Contrôles WPF

🔝 Retour au [Sommaire](/SOMMAIRE.md)

WPF (Windows Presentation Foundation) offre une riche bibliothèque de contrôles pour créer des interfaces utilisateur interactives et attrayantes. Ces contrôles sont optimisés pour fonctionner avec le système de liaison de données et de styles de WPF, ce qui permet de créer des applications hautement personnalisables.

## 8.4.1. Contrôles de base

Les contrôles de base constituent les éléments fondamentaux de toute interface WPF. Ils permettent de capturer des entrées utilisateur et d'afficher des informations simples.

### TextBlock

Le `TextBlock` est un contrôle léger utilisé pour afficher du texte. Contrairement au `Label`, il n'est pas conçu pour être interactif.

```xml
<TextBlock Text="Ceci est un TextBlock"
           FontSize="16"
           FontWeight="Bold"
           Foreground="Blue"
           TextWrapping="Wrap"
           Margin="10"/>
```


### TextBlock avec formatage inline

```xml
<TextBlock TextWrapping="Wrap" Margin="10">
    <Run Text="Texte normal suivi de " />
    <Run Text="texte en gras" FontWeight="Bold" />
    <Run Text=" puis " />
    <Run Text="texte en italique" FontStyle="Italic" />
    <LineBreak />
    <Run Text="Nouvelle ligne avec " />
    <Run Text="couleur différente" Foreground="Red" />
</TextBlock>
```


### Label

Le `Label` est similaire au `TextBlock` mais optimisé pour fonctionner avec les contrôles d'entrée en définissant la propriété `Target`.

```xml
<StackPanel Margin="10">
    <Label Content="Nom:" Target="{Binding ElementName=txtName}"/>
    <TextBox x:Name="txtName" Width="200" HorizontalAlignment="Left"/>

    <!-- Utilisation du raccourci d'accès avec underscore -->
    <Label Content="_Email:" Target="{Binding ElementName=txtEmail}"/>
    <TextBox x:Name="txtEmail" Width="200" HorizontalAlignment="Left"/>
</StackPanel>
```


### TextBox

Le `TextBox` permet la saisie et l'édition de texte.

```xml
<!-- TextBox simple -->
<TextBox Width="200"
         Margin="10"
         Text="Texte par défaut"
         ToolTip="Entrez du texte ici"/>

<!-- TextBox multilignes -->
<TextBox Width="200"
         Height="100"
         Margin="10"
         TextWrapping="Wrap"
         AcceptsReturn="True"
         VerticalScrollBarVisibility="Auto"
         Text="Texte multiligne.&#x0A;Nouvelle ligne."/>

<!-- TextBox en lecture seule -->
<TextBox Width="200"
         Margin="10"
         Text="Texte en lecture seule"
         IsReadOnly="True"/>

<!-- TextBox pour mot de passe -->
<PasswordBox Width="200"
             Margin="10"
             PasswordChar="*"/>
```


### Boutons

WPF offre plusieurs types de boutons pour différentes situations.

#### Button

```xml
<!-- Bouton standard -->
<Button Content="Cliquez-moi"
        Width="100"
        Margin="10"
        Click="Button_Click"/>

<!-- Bouton avec image et texte -->
<Button Margin="10" Padding="5" Width="120">
    <StackPanel Orientation="Horizontal">
        <Image Source="/Images/save.png" Width="16" Height="16" Margin="0,0,5,0"/>
        <TextBlock Text="Enregistrer"/>
    </StackPanel>
</Button>

<!-- Bouton avec style différent -->
<Button Content="Bouton spécial"
        Width="120"
        Height="40"
        Margin="10"
        Background="#3498db"
        Foreground="White"
        BorderThickness="0"
        FontWeight="Bold">
    <Button.Resources>
        <!-- Style pour l'effet de survol -->
        <Style TargetType="Button">
            <Style.Triggers>
                <Trigger Property="IsMouseOver" Value="True">
                    <Setter Property="Background" Value="#2980b9"/>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Button.Resources>
</Button>
```


#### CheckBox

```xml
<!-- CheckBox simple -->
<CheckBox Content="Option 1" Margin="10"/>

<!-- CheckBox à trois états -->
<CheckBox Content="Option à trois états"
          IsThreeState="True"
          Margin="10"/>

<!-- CheckBox groupés -->
<StackPanel Margin="10">
    <TextBlock Text="Sélectionnez vos langages préférés:" Margin="0,0,0,5"/>
    <CheckBox Content="C#" Margin="20,2,0,2"/>
    <CheckBox Content="Java" Margin="20,2,0,2"/>
    <CheckBox Content="Python" Margin="20,2,0,2"/>
    <CheckBox Content="JavaScript" Margin="20,2,0,2"/>
</StackPanel>
```


#### RadioButton

```xml
<!-- RadioButton simples dans un groupe -->
<StackPanel Margin="10">
    <TextBlock Text="Choisissez votre sexe:" Margin="0,0,0,5"/>
    <RadioButton Content="Homme" GroupName="Gender" Margin="20,2,0,2"/>
    <RadioButton Content="Femme" GroupName="Gender" Margin="20,2,0,2"/>
    <RadioButton Content="Autre" GroupName="Gender" Margin="20,2,0,2"/>

    <TextBlock Text="Choisissez votre tranche d'âge:" Margin="0,15,0,5"/>
    <RadioButton Content="Moins de 18 ans" GroupName="Age" Margin="20,2,0,2"/>
    <RadioButton Content="18-30 ans" GroupName="Age" Margin="20,2,0,2"/>
    <RadioButton Content="31-50 ans" GroupName="Age" Margin="20,2,0,2"/>
    <RadioButton Content="Plus de 50 ans" GroupName="Age" Margin="20,2,0,2"/>
</StackPanel>
```


#### ToggleButton

```xml
<StackPanel Margin="10">
    <TextBlock Text="Options de format:" Margin="0,0,0,5"/>

    <StackPanel Orientation="Horizontal">
        <ToggleButton Content="G"
                      FontWeight="Bold"
                      Width="30"
                      Margin="2"
                      ToolTip="Gras"/>

        <ToggleButton Content="I"
                      FontStyle="Italic"
                      Width="30"
                      Margin="2"
                      ToolTip="Italique"/>

        <ToggleButton Content="S"
                      TextDecorations="Underline"
                      Width="30"
                      Margin="2"
                      ToolTip="Souligné"/>
    </StackPanel>
</StackPanel>
```


### Contrôles numériques

```xml
<!-- Slider -->
<StackPanel Margin="10">
    <TextBlock Text="Volume:" Margin="0,0,0,5"/>
    <Slider Minimum="0"
            Maximum="100"
            Value="50"
            Width="200"
            TickFrequency="10"
            TickPlacement="BottomRight"
            IsSnapToTickEnabled="True"/>
</StackPanel>

<!-- ProgressBar -->
<StackPanel Margin="10">
    <TextBlock Text="Progression:" Margin="0,0,0,5"/>
    <ProgressBar Minimum="0"
                 Maximum="100"
                 Value="75"
                 Height="20"
                 Width="200"/>
</StackPanel>
```


### ComboBox

```xml
<!-- ComboBox simple -->
<ComboBox Width="200" Margin="10">
    <ComboBoxItem Content="Option 1"/>
    <ComboBoxItem Content="Option 2"/>
    <ComboBoxItem Content="Option 3"/>
    <ComboBoxItem Content="Option 4"/>
</ComboBox>

<!-- ComboBox avec élément sélectionné -->
<ComboBox Width="200" Margin="10" SelectedIndex="1">
    <ComboBoxItem Content="Rouge"/>
    <ComboBoxItem Content="Vert"/>
    <ComboBoxItem Content="Bleu"/>
    <ComboBoxItem Content="Jaune"/>
</ComboBox>

<!-- ComboBox avec items personnalisés -->
<ComboBox Width="250" Margin="10">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal">
                <Rectangle Width="16" Height="16" Fill="{Binding Color}" Margin="0,0,5,0"/>
                <TextBlock Text="{Binding Name}"/>
            </StackPanel>
        </DataTemplate>
    </ComboBox.ItemTemplate>
    <!-- Les items seraient définis dans le code-behind ou via binding -->
</ComboBox>
```


## 8.4.2. Contrôles de contenu

Les contrôles de contenu permettent d'organiser et de présenter des informations de manière structurée.

### GroupBox

```xml
<GroupBox Header="Informations personnelles" Margin="10" Padding="10">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0" Grid.Column="0" Text="Nom:" Margin="0,0,10,5"/>
        <TextBox Grid.Row="0" Grid.Column="1" Margin="0,0,0,5"/>

        <TextBlock Grid.Row="1" Grid.Column="0" Text="Email:" Margin="0,0,10,5"/>
        <TextBox Grid.Row="1" Grid.Column="1" Margin="0,0,0,5"/>

        <TextBlock Grid.Row="2" Grid.Column="0" Text="Téléphone:" Margin="0,0,10,5"/>
        <TextBox Grid.Row="2" Grid.Column="1"/>
    </Grid>
</GroupBox>
```


### Expander

```xml
<Expander Header="Détails supplémentaires" Margin="10" Padding="10">
    <StackPanel>
        <TextBlock Text="Ces informations sont masquées par défaut mais peuvent être affichées en cliquant sur la flèche."
                   TextWrapping="Wrap" Margin="0,0,0,10"/>
        <TextBlock Text="Vous pouvez ajouter autant de contenu que nécessaire à l'intérieur d'un Expander."
                   TextWrapping="Wrap"/>
    </StackPanel>
</Expander>

<!-- Expander avec style personnalisé -->
<Expander Header="Options avancées"
          Margin="10"
          Padding="10"
          Background="#f8f9fa"
          BorderBrush="#dee2e6"
          BorderThickness="1">
    <StackPanel>
        <CheckBox Content="Activer le mode débogage" Margin="0,0,0,5"/>
        <CheckBox Content="Enregistrer les journaux détaillés" Margin="0,0,0,5"/>
        <CheckBox Content="Utiliser l'authentification à deux facteurs" Margin="0,0,0,5"/>
        <Button Content="Réinitialiser les paramètres"
                HorizontalAlignment="Right"
                Margin="0,10,0,0"
                Padding="10,5"/>
    </StackPanel>
</Expander>
```


### ScrollViewer

```xml
<!-- ScrollViewer avec contenu défilable -->
<Border BorderBrush="Gray" BorderThickness="1" Margin="10">
    <ScrollViewer Height="200"
                  VerticalScrollBarVisibility="Auto"
                  HorizontalScrollBarVisibility="Auto">
        <StackPanel Width="400">
            <TextBlock Text="Contenu avec défilement"
                       FontSize="18"
                       FontWeight="Bold"
                       Margin="10"/>

            <TextBlock Margin="10" TextWrapping="Wrap">
                Lorem ipsum dolor sit amet, consectetur adipiscing elit.
                Sed non risus. Suspendisse lectus tortor, dignissim sit amet,
                adipiscing nec, ultricies sed, dolor. Cras elementum ultrices diam.
                Maecenas ligula massa, varius a, semper congue, euismod non, mi.
                Proin porttitor, orci nec nonummy molestie, enim est eleifend mi,
                non fermentum diam nisl sit amet erat. Duis semper. Duis arcu massa,
                scelerisque vitae, consequat in, pretium a, enim. Pellentesque congue.
                Ut in risus volutpat libero pharetra tempor. Cras vestibulum bibendum augue.
                Praesent egestas leo in pede.
            </TextBlock>

            <!-- Image plus large que la zone visible -->
            <Rectangle Height="300" Fill="LightBlue" Margin="10"/>

            <TextBlock Margin="10" TextWrapping="Wrap">
                Plus de texte pour illustrer le défilement vertical...
                Nullam eu ante vel est convallis dignissim. Fusce suscipit,
                wisi nec facilisis facilisis, est dui fermentum leo, quis tempor
                ligula erat quis odio. Nunc porta vulputate tellus. Nunc rutrum
                turpis sed pede. Sed bibendum. Aliquam posuere. Nunc aliquet,
                augue nec adipiscing interdum, lacus tellus malesuada massa,
                quis varius mi purus non odio.
            </TextBlock>
        </StackPanel>
    </ScrollViewer>
</Border>
```


### Border

```xml
<!-- Border simple -->
<Border BorderBrush="Gray"
        BorderThickness="1"
        Padding="10"
        Margin="10">
    <TextBlock Text="Contenu avec bordure" TextWrapping="Wrap"/>
</Border>

<!-- Border avec coin arrondis et dégradé -->
<Border BorderBrush="#2c3e50"
        BorderThickness="2"
        CornerRadius="8"
        Padding="15"
        Margin="10">
    <Border.Background>
        <LinearGradientBrush StartPoint="0,0" EndPoint="1,1">
            <GradientStop Color="#ecf0f1" Offset="0"/>
            <GradientStop Color="#bdc3c7" Offset="1"/>
        </LinearGradientBrush>
    </Border.Background>
    <TextBlock Text="Bordure stylisée avec coins arrondis et dégradé"
               TextWrapping="Wrap"
               FontWeight="Bold"/>
</Border>

<!-- Border comme carte d'information -->
<Border BorderBrush="#e0e0e0"
        BorderThickness="1"
        CornerRadius="5"
        Padding="15"
        Margin="10"
        Background="White">
    <Border.Effect>
        <DropShadowEffect ShadowDepth="2"
                          Opacity="0.2"
                          BlurRadius="5"
                          Color="Black"/>
    </Border.Effect>
    <StackPanel>
        <TextBlock Text="Titre de la carte"
                   FontWeight="Bold"
                   FontSize="16"
                   Margin="0,0,0,10"/>
        <TextBlock Text="Contenu de la carte avec description détaillée. Ce contrôle Border peut être utilisé pour créer des interfaces modernes de type carte."
                   TextWrapping="Wrap"
                   Margin="0,0,0,10"/>
        <Button Content="Action"
                HorizontalAlignment="Right"
                Padding="15,5"/>
    </StackPanel>
</Border>
```


### ViewBox

```xml
<!-- ViewBox pour mettre à l'échelle le contenu -->
<Border BorderBrush="Gray"
        BorderThickness="1"
        Margin="10"
        Height="200"
        Width="200">
    <Viewbox Stretch="Uniform" StretchDirection="Both">
        <StackPanel Width="400" Height="400">
            <TextBlock Text="Contenu mis à l'échelle"
                       FontSize="36"
                       FontWeight="Bold"
                       HorizontalAlignment="Center"/>
            <Ellipse Width="200"
                     Height="200"
                     Fill="Green"
                     Margin="0,20"/>
            <TextBlock Text="Ce contenu s'adapte au conteneur"
                       FontSize="24"
                       HorizontalAlignment="Center"/>
        </StackPanel>
    </Viewbox>
</Border>
```


## 8.4.3. Contrôles de données

Les contrôles de données permettent d'afficher et d'interagir avec des collections d'éléments.

### ListBox

```xml
<!-- ListBox simple -->
<ListBox Width="200" Margin="10">
    <ListBoxItem Content="Élément 1"/>
    <ListBoxItem Content="Élément 2"/>
    <ListBoxItem Content="Élément 3"/>
    <ListBoxItem Content="Élément 4"/>
    <ListBoxItem Content="Élément 5"/>
</ListBox>

<!-- ListBox avec sélection multiple -->
<ListBox Width="200"
         Margin="10"
         SelectionMode="Multiple">
    <ListBoxItem Content="Option A"/>
    <ListBoxItem Content="Option B"/>
    <ListBoxItem Content="Option C"/>
    <ListBoxItem Content="Option D"/>
</ListBox>

<!-- ListBox avec items personnalisés -->
<ListBox Width="300" Margin="10">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <Border BorderBrush="LightGray"
                    BorderThickness="0,0,0,1"
                    Padding="5">
                <Grid>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition Width="*"/>
                    </Grid.ColumnDefinitions>

                    <Ellipse Grid.Column="0"
                             Width="32"
                             Height="32"
                             Fill="{Binding Color}"
                             Margin="0,0,10,0"/>

                    <StackPanel Grid.Column="1">
                        <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
                        <TextBlock Text="{Binding Description}"
                                   TextWrapping="Wrap"
                                   Opacity="0.7"/>
                    </StackPanel>
                </Grid>
            </Border>
        </DataTemplate>
    </ListBox.ItemTemplate>
    <!-- Les items seraient définis dans le code-behind ou via binding -->
</ListBox>
```


### ListView

```xml
<!-- ListView simple -->
<ListView Width="400" Height="200" Margin="10">
    <ListViewItem Content="Élément de liste A"/>
    <ListViewItem Content="Élément de liste B"/>
    <ListViewItem Content="Élément de liste C"/>
</ListView>

<!-- ListView avec colonnes (GridView) -->
<ListView Width="500" Height="200" Margin="10">
    <ListView.View>
        <GridView>
            <GridViewColumn Header="ID" Width="50"/>
            <GridViewColumn Header="Nom" Width="150"/>
            <GridViewColumn Header="Email" Width="200"/>
            <GridViewColumn Header="Statut" Width="100"/>
        </GridView>
    </ListView.View>

    <ListViewItem>
        <ListViewItem.Content>
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="50"/>
                    <ColumnDefinition Width="150"/>
                    <ColumnDefinition Width="200"/>
                    <ColumnDefinition Width="100"/>
                </Grid.ColumnDefinitions>

                <TextBlock Grid.Column="0" Text="1"/>
                <TextBlock Grid.Column="1" Text="Jean Dupont"/>
                <TextBlock Grid.Column="2" Text="jean.dupont@example.com"/>
                <TextBlock Grid.Column="3" Text="Actif" Foreground="Green"/>
            </Grid>
        </ListViewItem.Content>
    </ListViewItem>

    <ListViewItem>
        <ListViewItem.Content>
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="50"/>
                    <ColumnDefinition Width="150"/>
                    <ColumnDefinition Width="200"/>
                    <ColumnDefinition Width="100"/>
                </Grid.ColumnDefinitions>

                <TextBlock Grid.Column="0" Text="2"/>
                <TextBlock Grid.Column="1" Text="Marie Martin"/>
                <TextBlock Grid.Column="2" Text="marie.martin@example.com"/>
                <TextBlock Grid.Column="3" Text="Inactif" Foreground="Red"/>
            </Grid>
        </ListViewItem.Content>
    </ListViewItem>

    <ListViewItem>
        <ListViewItem.Content>
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="50"/>
                    <ColumnDefinition Width="150"/>
                    <ColumnDefinition Width="200"/>
                    <ColumnDefinition Width="100"/>
                </Grid.ColumnDefinitions>

                <TextBlock Grid.Column="0" Text="3"/>
                <TextBlock Grid.Column="1" Text="Pierre Durand"/>
                <TextBlock Grid.Column="2" Text="pierre.durand@example.com"/>
                <TextBlock Grid.Column="3" Text="En attente" Foreground="Orange"/>
            </Grid>
        </ListViewItem.Content>
    </ListViewItem>
</ListView>
```


### DataGrid (.NET Framework 4.7.2)

Pour utiliser `DataGrid` avec .NET Framework 4.7.2, vous devez d'abord ajouter une référence au package `System.Windows.Controls.DataGrid`.

```xml
<!-- DataGrid simple -->
<DataGrid AutoGenerateColumns="True"
          Width="500"
          Height="200"
          Margin="10">
    <!-- Les données seraient définies dans le code-behind ou via binding -->
</DataGrid>

<!-- DataGrid avec colonnes personnalisées -->
<DataGrid AutoGenerateColumns="False"
          Width="500"
          Height="200"
          Margin="10"
          CanUserAddRows="False"
          CanUserDeleteRows="False"
          CanUserResizeRows="False"
          RowHeaderWidth="0">
    <DataGrid.Columns>
        <DataGridTextColumn Header="ID" Binding="{Binding Id}" Width="50"/>
        <DataGridTextColumn Header="Nom" Binding="{Binding Name}" Width="150"/>
        <DataGridTextColumn Header="Email" Binding="{Binding Email}" Width="200"/>
        <DataGridTemplateColumn Header="Statut" Width="100">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <TextBlock Text="{Binding Status}"
                               Foreground="{Binding StatusColor}"/>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>
        <DataGridTemplateColumn Header="Actions" Width="100">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <Button Content="Edit"
                                Padding="5,2"
                                Margin="0,0,5,0"
                                Command="{Binding EditCommand}"/>
                        <Button Content="X"
                                Padding="5,2"
                                Background="Red"
                                Foreground="White"
                                Command="{Binding DeleteCommand}"/>
                    </StackPanel>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>
    </DataGrid.Columns>
    <!-- Les données seraient définies dans le code-behind ou via binding -->
</DataGrid>
```


### DataGrid (.NET 8)

Avec .NET 8, vous devez utiliser le package NuGet `Microsoft.WindowsDesktop.App.WPF`.

```xml
<!-- DataGrid dans .NET 8 -->
<DataGrid AutoGenerateColumns="False"
          Width="500"
          Height="200"
          Margin="10"
          GridLinesVisibility="Horizontal"
          BorderThickness="0"
          RowHeaderWidth="0">
    <DataGrid.Columns>
        <DataGridTextColumn Header="ID" Binding="{Binding Id}" Width="50"/>
        <DataGridTextColumn Header="Nom" Binding="{Binding Name}" Width="150"/>
        <DataGridTextColumn Header="Email" Binding="{Binding Email}" Width="200"/>
        <DataGridTemplateColumn Header="Statut" Width="100">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <Border Background="{Binding StatusBackgroundColor}"
                            CornerRadius="4"
                            Padding="5,2">
                        <TextBlock Text="{Binding Status}"
                                   Foreground="{Binding StatusForegroundColor}"
                                   HorizontalAlignment="Center"/>
                    </Border>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>
    </DataGrid.Columns>
    <!-- Les données seraient définies dans le code-behind ou via binding -->
</DataGrid>
```


### TreeView

```xml
<!-- TreeView simple -->
<TreeView Width="250" Height="200" Margin="10">
    <TreeViewItem Header="Dossier racine" IsExpanded="True">
        <TreeViewItem Header="Documents">
            <TreeViewItem Header="Rapport.docx"/>
            <TreeViewItem Header="Budget.xlsx"/>
            <TreeViewItem Header="Présentation.pptx"/>
        </TreeViewItem>
        <TreeViewItem Header="Images">
            <TreeViewItem Header="Photo1.jpg"/>
            <TreeViewItem Header="Photo2.jpg"/>
            <TreeViewItem Header="Logo.png"/>
        </TreeViewItem>
        <TreeViewItem Header="Musique">
            <TreeViewItem Header="Chanson1.mp3"/>
            <TreeViewItem Header="Chanson2.mp3"/>
        </TreeViewItem>
    </TreeViewItem>
</TreeView>

<!-- TreeView avec items personnalisés -->
<TreeView Width="300" Height="250" Margin="10">
    <TreeView.Resources>
        <HierarchicalDataTemplate DataType="{x:Type local:FolderItem}"
                                 ItemsSource="{Binding Items}">
            <StackPanel Orientation="Horizontal">
                <Image Source="/Images/folder.png" Width="16" Height="16" Margin="0,0,5,0"/>
                <TextBlock Text="{Binding Name}"/>
            </StackPanel>
        </HierarchicalDataTemplate>

        <DataTemplate DataType="{x:Type local:FileItem}">
            <StackPanel Orientation="Horizontal">
                <Image Source="{Binding IconPath}" Width="16" Height="16" Margin="0,0,5,0"/>
                <TextBlock Text="{Binding Name}"/>
            </StackPanel>
        </DataTemplate>
    </TreeView.Resources>
    <!-- Les items seraient définis dans le code-behind ou via binding -->
</TreeView>
```


## 8.4.4. Contrôles de navigation

Les contrôles de navigation permettent aux utilisateurs de naviguer entre différentes sections ou pages de l'application.

### TabControl

```xml
<!-- TabControl simple -->
<TabControl Width="400" Height="250" Margin="10">
    <TabItem Header="Informations générales">
        <StackPanel Margin="10">
            <TextBlock Text="Informations générales"
                       FontWeight="Bold"
                       Margin="0,0,0,10"/>
            <TextBlock Text="Cette section contient des informations générales sur le projet."
                       TextWrapping="Wrap"/>
        </StackPanel>
    </TabItem>

    <TabItem Header="Paramètres">
        <StackPanel Margin="10">
            <TextBlock Text="Paramètres"
                       FontWeight="Bold"
                       Margin="0,0,0,10"/>
            <CheckBox Content="Activer la fonctionnalité A" Margin="0,0,0,5"/>
            <CheckBox Content="Activer la fonctionnalité B" Margin="0,0,0,5"/>
            <CheckBox Content="Activer la fonctionnalité C" Margin="0,0,0,5"/>
        </StackPanel>
    </TabItem>

    <TabItem Header="Aide">
        <ScrollViewer Margin="10">
            <TextBlock TextWrapping="Wrap">
                <Run FontWeight="Bold" FontSize="14">Aide et documentation</Run>
                <LineBreak/>
                <LineBreak/>
                Cette section contient l'aide et la documentation pour l'application.
                Vous pouvez trouver des informations sur l'utilisation des différentes fonctionnalités.
                <LineBreak/>
                <LineBreak/>
                Pour plus d'informations, consultez le manuel d'utilisation complet ou contactez le support technique.
            </TextBlock>
        </ScrollViewer>
    </TabItem>
</TabControl>

<!-- TabControl avec style personnalisé -->
<TabControl Width="400" Height="250" Margin="10">
    <TabControl.Resources>
        <Style TargetType="TabItem">
            <Setter Property="Background" Value="#f8f9fa"/>
            <Setter Property="BorderBrush" Value="#dee2e6"/>
            <Setter Property="BorderThickness" Value="1,1,1,0"/>
            <Setter Property="Padding" Value="15,8"/>
            <Setter Property="Margin" Value="0,0,5,0"/>
            <Style.Triggers>
                <Trigger Property="IsSelected" Value="True">
                    <Setter Property="Background" Value="White"/>
                    <Setter Property="BorderThickness" Value="1,1,1,0"/>
                    <Setter Property="FontWeight" Value="Bold"/>
                </Trigger>
            </Style.Triggers>
        </Style>
    </TabControl.Resources>

    <TabItem Header="Accueil">
        <Border Background="White" BorderBrush="#dee2e6" BorderThickness="1" Padding="15">
            <TextBlock Text="Page d'accueil" TextWrapping="Wrap"/>
        </Border>
    </TabItem>

    <TabItem Header="Profil">
        <Border Background="White" BorderBrush="#dee2e6" BorderThickness="1" Padding="15">
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="*"/>
                </Grid.RowDefinitions>

                <TextBlock Grid.Row="0" Text="Profil utilisateur"
                           FontWeight="Bold" FontSize="14" Margin="0,0,0,10"/>

                <StackPanel Grid.Row="1">
                    <TextBlock Text="Nom d'utilisateur: JohnDoe" Margin="0,0,0,5"/>
                    <TextBlock Text="Email: johndoe@example.com" Margin="0,0,0,5"/>
                    <TextBlock Text="Rôle: Administrateur" Margin="0,0,0,15"/>
                    <Button Content="Modifier le profil"
                            HorizontalAlignment="Left"
                            Padding="10,5"/>
                </StackPanel>
            </Grid>
        </Border>
    </TabItem>

    <TabItem Header="Paramètres">
        <Border Background="White" BorderBrush="#dee2e6" BorderThickness="1" Padding="15">
            <TextBlock Text="Page des paramètres" TextWrapping="Wrap"/>
        </Border>
    </TabItem>
</TabControl>
```


### Menu et ContextMenu

```xml
<!-- Menu principal -->
<Menu DockPanel.Dock="Top" Background="#f8f9fa">
    <MenuItem Header="_Fichier">
        <MenuItem Header="_Nouveau" InputGestureText="Ctrl+N"/>
        <MenuItem Header="_Ouvrir" InputGestureText="Ctrl+O"/>
        <MenuItem Header="_Enregistrer" InputGestureText="Ctrl+S"/>
        <Separator/>
        <MenuItem Header="_Imprimer" InputGestureText="Ctrl+P"/>
        <Separator/>
        <MenuItem Header="_Quitter" InputGestureText="Alt+F4"/>
    </MenuItem>

    <MenuItem Header="_Édition">
        <MenuItem Header="_Annuler" InputGestureText="Ctrl+Z"/>
        <MenuItem Header="_Rétablir" InputGestureText="Ctrl+Y"/>
        <Separator/>
        <MenuItem Header="_Couper" InputGestureText="Ctrl+X"/>
        <MenuItem Header="C_opier" InputGestureText="Ctrl+C"/>
        <MenuItem Header="Co_ller" InputGestureText="Ctrl+V"/>
        <Separator/>
        <MenuItem Header="Sélectionner _tout" InputGestureText="Ctrl+A"/>
    </MenuItem>

    <MenuItem Header="_Affichage">
        <MenuItem Header="Zoom avant" InputGestureText="Ctrl++"/>
        <MenuItem Header="Zoom arrière" InputGestureText="Ctrl+-"/>
        <MenuItem Header="Zoom normal" InputGestureText="Ctrl+0"/>
        <Separator/>
        <MenuItem Header="Plein écran" InputGestureText="F11"/>
    </MenuItem>

    <MenuItem Header="_Aide">
        <MenuItem Header="Documentation"/>
        <MenuItem Header="Raccourcis clavier"/>
        <Separator/>
        <MenuItem Header="À propos"/>
    </MenuItem>
</Menu>

<!-- ContextMenu sur un élément -->
<TextBox Width="200" Height="100" Margin="10"
         Text="Cliquez avec le bouton droit pour ouvrir le menu contextuel">
    <TextBox.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Couper" Command="Cut"/>
            <MenuItem Header="Copier" Command="Copy"/>
            <MenuItem Header="Coller" Command="Paste"/>
            <Separator/>
            <MenuItem Header="Sélectionner tout" Command="SelectAll"/>
        </ContextMenu>
    </TextBox.ContextMenu>
</TextBox>

<!-- ContextMenu avec icônes -->
<Border Width="200" Height="100" Margin="10"
        Background="LightGray"
        BorderBrush="Gray"
        BorderThickness="1">
    <Border.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Ouvrir">
                <MenuItem.Icon>
                    <Image Source="/Images/open.png" Width="16" Height="16"/>
                </MenuItem.Icon>
            </MenuItem>

            <MenuItem Header="Enregistrer">
                <MenuItem.Icon>
                    <Image Source="/Images/save.png" Width="16" Height="16"/>
                </MenuItem.Icon>
            </MenuItem>

            <Separator/>

            <MenuItem Header="Supprimer">
                <MenuItem.Icon>
                    <Image Source="/Images/delete.png" Width="16" Height="16"/>
                </MenuItem.Icon>
            </MenuItem>

            <MenuItem Header="Propriétés">
                <MenuItem.Icon>
                    <Image Source="/Images/properties.png" Width="16" Height="16"/>
                </MenuItem.Icon>
            </MenuItem>
        </ContextMenu>
    </Border.ContextMenu>
    <TextBlock Text="Zone avec menu contextuel. Cliquez avec le bouton droit pour voir le menu."
               TextWrapping="Wrap"
               VerticalAlignment="Center"
               HorizontalAlignment="Center"/>
</Border>
```


### Frame et Page

WPF permet de naviguer entre différentes pages via le contrôle `Frame`. Les `Page` sont des vues spécifiques conçues pour la navigation.

```xml
<!-- Frame simple -->
<Frame Name="MainFrame"
       Source="Pages/HomePage.xaml"
       NavigationUIVisibility="Visible"
       Margin="10"/>

<!-- Frame avec navigation personnalisée -->
<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Barre de navigation personnalisée -->
    <StackPanel Grid.Row="0"
                Orientation="Horizontal"
                Background="#007bff"
                Padding="10">
        <Button Content="Accueil"
                Margin="0,0,10,0"
                Padding="10,5"
                Background="Transparent"
                Foreground="White"
                BorderThickness="0"
                Click="NavigateToHome"/>

        <Button Content="Produits"
                Margin="0,0,10,0"
                Padding="10,5"
                Background="Transparent"
                Foreground="White"
                BorderThickness="0"
                Click="NavigateToProducts"/>

        <Button Content="À propos"
                Margin="0,0,10,0"
                Padding="10,5"
                Background="Transparent"
                Foreground="White"
                BorderThickness="0"
                Click="NavigateToAbout"/>
    </StackPanel>

    <!-- Frame pour le contenu -->
    <Frame Grid.Row="1"
           Name="ContentFrame"
           NavigationUIVisibility="Hidden"/>
</Grid>
```


Le code-behind pourrait ressembler à ceci :

```textmate
private void NavigateToHome(object sender, RoutedEventArgs e)
{
    ContentFrame.Navigate(new Uri("Pages/HomePage.xaml", UriKind.Relative));
}

private void NavigateToProducts(object sender, RoutedEventArgs e)
{
    ContentFrame.Navigate(new Uri("Pages/ProductsPage.xaml", UriKind.Relative));
}

private void NavigateToAbout(object sender, RoutedEventArgs e)
{
    ContentFrame.Navigate(new Uri("Pages/AboutPage.xaml", UriKind.Relative));
}
```


Exemple de contenu pour une `Page` (HomePage.xaml) :

```xml
<Page x:Class="MyApp.Pages.HomePage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      Title="Accueil">

    <Grid Background="White" Margin="20">
        <StackPanel>
            <TextBlock Text="Bienvenue sur notre application"
                       FontSize="24"
                       FontWeight="Bold"
                       Margin="0,0,0,20"/>

            <TextBlock TextWrapping="Wrap" Margin="0,0,0,15">
                Ceci est la page d'accueil de notre application.
                Vous pouvez naviguer vers d'autres pages en utilisant
                les boutons de navigation en haut de l'écran.
            </TextBlock>

            <Button Content="Commencer"
                    HorizontalAlignment="Left"
                    Padding="15,5"/>
        </StackPanel>
    </Grid>
</Page>
```


## 8.4.5. DatePicker, ColorPicker et autres contrôles spécialisés

WPF inclut plusieurs contrôles spécialisés pour des tâches spécifiques.

### DatePicker

```xml
<!-- DatePicker simple -->
<DatePicker Width="200" Margin="10"/>

<!-- DatePicker avec valeur par défaut -->
<DatePicker Width="200"
            Margin="10"
            SelectedDate="{Binding TodayDate}"/>

<!-- DatePicker avec formatage personnalisé -->
<DatePicker Width="200"
            Margin="10"
            SelectedDateFormat="Long"/>

<!-- DatePicker avec plage de dates restreinte -->
<DatePicker Width="200"
            Margin="10"
            DisplayDateStart="{Binding StartDate}"
            DisplayDateEnd="{Binding EndDate}"/>

<!-- DatePicker avec style personnalisé -->
<DatePicker Width="200"
            Margin="10">
    <DatePicker.Resources>
        <Style TargetType="DatePickerTextBox">
            <Setter Property="Background" Value="#f8f9fa"/>
            <Setter Property="BorderBrush" Value="#ced4da"/>
            <Setter Property="Padding" Value="5"/>
        </Style>
    </DatePicker.Resources>
</DatePicker>
```


### Calendar

```xml
<!-- Calendar simple -->
<Calendar Margin="10"/>

<!-- Calendar avec sélection multiple -->
<Calendar Margin="10"
          SelectionMode="MultipleRange"/>

<!-- Calendar avec style personnalisé -->
<Calendar Margin="10"
          Background="White"
          BorderBrush="#ced4da"
          BorderThickness="1">
    <Calendar.Resources>
        <Style TargetType="CalendarDayButton">
            <Setter Property="Background" Value="#f8f9fa"/>
            <Setter Property="BorderBrush" Value="Transparent"/>
            <Style.Triggers>
                <Trigger Property="IsSelected" Value="True">
                    <Setter Property="Background" Value="#007bff"/>
                    <Setter Property="Foreground" Value="White"/>
                </Trigger>
                <Trigger Property="IsToday" Value="True">
                    <Setter Property="FontWeight" Value="Bold"/>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Calendar.Resources>
</Calendar>
```


### ColorPicker (WPF Extended Toolkit)

Le contrôle `ColorPicker` n'est pas inclus dans WPF de base. Vous devez utiliser le package NuGet Extended WPF Toolkit.

```xml
<!-- Pour utiliser le ColorPicker, ajoutez d'abord la référence au package -->
<!-- xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit" -->

<!-- ColorPicker simple -->
<xctk:ColorPicker Width="200" Margin="10"/>

<!-- ColorPicker avec valeur par défaut -->
<xctk:ColorPicker Width="200"
                  Margin="10"
                  SelectedColor="Red"/>

<!-- ColorPicker avancé -->
<xctk:ColorPicker Width="200"
                  Margin="10"
                  DisplayColorAndName="True"
                  ColorMode="ColorCanvas"
                  ShowAdvancedButton="True"
                  AdvancedButtonHeader="Options avancées"
                  StandardButtonHeader="Couleurs standard"
                  StandardColors="{Binding StandardColors}"
                  AvailableColors="{Binding AvailableColors}"
                  UsingAlphaChannel="True"/>
```


### Slider

```xml
<!-- Slider horizontal -->
<Slider Width="200"
        Minimum="0"
        Maximum="100"
        Value="50"
        Margin="10"/>

<!-- Slider vertical -->
<Slider Orientation="Vertical"
        Height="200"
        Minimum="0"
        Maximum="100"
        Value="50"
        Margin="10"/>

<!-- Slider avec graduations -->
<Slider Width="200"
        Minimum="0"
        Maximum="100"
        Value="50"
        TickFrequency="10"
        TickPlacement="BottomRight"
        IsSnapToTickEnabled="True"
        Margin="10"/>

<!-- Slider avec style personnalisé -->
<Slider Width="200"
        Minimum="0"
        Maximum="100"
        Value="50"
        Margin="10">
    <Slider.Resources>
        <Style TargetType="Thumb">
            <Setter Property="Background" Value="#007bff"/>
            <Setter Property="Height" Value="20"/>
            <Setter Property="Width" Value="20"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="Thumb">
                        <Ellipse Fill="{TemplateBinding Background}"
                                Width="{TemplateBinding Width}"
                                Height="{TemplateBinding Height}"/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </Slider.Resources>
</Slider>
```


### MediaElement

```xml
<!-- MediaElement pour lire des fichiers audio ou vidéo -->
<MediaElement Source="Videos/sample.mp4"
              Width="400"
              Height="225"
              LoadedBehavior="Manual"
              Margin="10"
              Name="mediaPlayer"/>

<!-- Contrôles pour le MediaElement -->
<StackPanel Orientation="Horizontal" Margin="10">
    <Button Content="Lire"
            Click="PlayButton_Click"
            Padding="10,5"
            Margin="0,0,5,0"/>

    <Button Content="Pause"
            Click="PauseButton_Click"
            Padding="10,5"
            Margin="0,0,5,0"/>

    <Button Content="Arrêter"
            Click="StopButton_Click"
            Padding="10,5"
            Margin="0,0,5,0"/>

    <Slider Width="200"
            Margin="10,0"
            Minimum="0"
            Maximum="{Binding ElementName=mediaPlayer, Path=NaturalDuration.TimeSpan.TotalSeconds}"
            Value="{Binding ElementName=mediaPlayer, Path=Position.TotalSeconds, Mode=TwoWay}"/>
</StackPanel>
```


Code-behind pour les boutons de contrôle du MediaElement :

```textmate
private void PlayButton_Click(object sender, RoutedEventArgs e)
{
    mediaPlayer.Play();
}

private void PauseButton_Click(object sender, RoutedEventArgs e)
{
    mediaPlayer.Pause();
}

private void StopButton_Click(object sender, RoutedEventArgs e)
{
    mediaPlayer.Stop();
}
```


### PasswordBox

```xml
<!-- PasswordBox simple -->
<PasswordBox Width="200"
             Margin="10"
             PasswordChar="*"/>

<!-- PasswordBox avec indice -->
<PasswordBox Width="200"
             Margin="10"
             PasswordChar="*"
             MaxLength="20">
    <PasswordBox.ToolTip>
        <ToolTip>
            <TextBlock>
                Le mot de passe doit contenir au moins 8 caractères,<LineBreak/>
                une majuscule, une minuscule et un chiffre.
            </TextBlock>
        </ToolTip>
    </PasswordBox.ToolTip>
</PasswordBox>

<!-- PasswordBox avec style personnalisé -->
<PasswordBox Width="200"
             Margin="10"
             PasswordChar="●"
             Padding="5"
             Background="#f8f9fa"
             BorderBrush="#ced4da"
             BorderThickness="1"/>
```


### ProgressBar

```xml
<!-- ProgressBar standard -->
<ProgressBar Width="200"
             Height="20"
             Minimum="0"
             Maximum="100"
             Value="75"
             Margin="10"/>

<!-- ProgressBar indéterminé -->
<ProgressBar Width="200"
             Height="20"
             IsIndeterminate="True"
             Margin="10"/>

<!-- ProgressBar avec style personnalisé -->
<ProgressBar Width="200"
             Height="10"
             Minimum="0"
             Maximum="100"
             Value="75"
             Margin="10"
             Foreground="#28a745"
             Background="#e9ecef"
             BorderBrush="Transparent"/>
```


### Contrôles spécialisés personnalisés

Vous pouvez créer des contrôles personnalisés pour des besoins spécifiques. Voici un exemple de contrôle personnalisé simple :

```xml
<!-- NumericUpDown personnalisé -->
<Grid Width="200" Margin="10">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="Auto"/>
    </Grid.ColumnDefinitions>

    <TextBox Grid.Column="0"
             Text="{Binding NumericValue, UpdateSourceTrigger=PropertyChanged}"
             TextAlignment="Right"
             Padding="5"/>

    <StackPanel Grid.Column="1" Width="30">
        <Button Content="▲"
                Height="15"
                Command="{Binding IncrementCommand}"/>

        <Button Content="▼"
                Height="15"
                Command="{Binding DecrementCommand}"/>
    </StackPanel>
</Grid>

<!-- Contrôle de notation par étoiles personnalisé -->
<StackPanel Orientation="Horizontal" Margin="10">
    <TextBlock Text="Note: " VerticalAlignment="Center" Margin="0,0,5,0"/>
    <ItemsControl ItemsSource="{Binding StarItems}">
        <ItemsControl.ItemsPanel>
            <ItemsPanelTemplate>
                <StackPanel Orientation="Horizontal"/>
            </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
            <DataTemplate>
                <Button Background="Transparent"
                        BorderThickness="0"
                        Padding="2"
                        Command="{Binding SetRatingCommand}"
                        CommandParameter="{Binding Rating}">
                    <Path Data="M 0,0 L 5,0 L 7,5 L 9,0 L 14,0 L 10,7 L 12,14 L 7,10 L 2,14 L 4,7 Z"
                          Fill="{Binding StarBrush}"
                          Stretch="Uniform"
                          Width="20"
                          Height="20"/>
                </Button>
            </DataTemplate>
        </ItemsControl.ItemTemplate>
    </ItemsControl>
</StackPanel>
```


### RichTextBox

```xml
<!-- RichTextBox simple -->
<RichTextBox Width="400"
             Height="200"
             Margin="10">
    <FlowDocument>
        <Paragraph>
            <Run Text="Ceci est un texte dans un RichTextBox. " FontWeight="Bold"/>
            <Run Text="Vous pouvez utiliser différents"/>
            <Run Text=" styles " FontStyle="Italic"/>
            <Run Text="dans votre texte."/>
        </Paragraph>
        <Paragraph>
            <Run Text="Voici un deuxième paragraphe avec du texte "/>
            <Run Text="coloré " Foreground="Red"/>
            <Run Text="pour illustration."/>
        </Paragraph>
    </FlowDocument>
</RichTextBox>

<!-- Barre d'outils pour RichTextBox -->
<StackPanel Margin="10">
    <ToolBar>
        <Button Command="EditingCommands.ToggleBold" ToolTip="Gras">
            <TextBlock Text="G" FontWeight="Bold"/>
        </Button>
        <Button Command="EditingCommands.ToggleItalic" ToolTip="Italique">
            <TextBlock Text="I" FontStyle="Italic"/>
        </Button>
        <Button Command="EditingCommands.ToggleUnderline" ToolTip="Souligné">
            <TextBlock Text="S" TextDecorations="Underline"/>
        </Button>
        <Separator/>
        <ComboBox Width="100" SelectedIndex="0">
            <ComboBoxItem>Normal</ComboBoxItem>
            <ComboBoxItem>Titre 1</ComboBoxItem>
            <ComboBoxItem>Titre 2</ComboBoxItem>
            <ComboBoxItem>Citation</ComboBoxItem>
        </ComboBox>
        <ComboBox Width="120" SelectedIndex="0">
            <ComboBoxItem>Times New Roman</ComboBoxItem>
            <ComboBoxItem>Arial</ComboBoxItem>
            <ComboBoxItem>Calibri</ComboBoxItem>
            <ComboBoxItem>Comic Sans MS</ComboBoxItem>
        </ComboBox>
        <ComboBox Width="50" SelectedIndex="2">
            <ComboBoxItem>10</ComboBoxItem>
            <ComboBoxItem>11</ComboBoxItem>
            <ComboBoxItem>12</ComboBoxItem>
            <ComboBoxItem>14</ComboBoxItem>
            <ComboBoxItem>16</ComboBoxItem>
            <ComboBoxItem>18</ComboBoxItem>
        </ComboBox>
    </ToolBar>

    <RichTextBox Height="200"
                 VerticalScrollBarVisibility="Auto"
                 HorizontalScrollBarVisibility="Auto"
                 BorderBrush="#ced4da"
                 BorderThickness="1"/>
</StackPanel>
```


Les contrôles WPF offrent une grande flexibilité et peuvent être personnalisés de manière approfondie grâce au système de styles et de templates. Combinés avec la liaison de données (data binding), ils permettent de créer des interfaces utilisateur riches et interactives.

Pour des applications complexes, il est recommandé d'utiliser des patterns comme MVVM (Model-View-ViewModel) pour séparer clairement la logique métier de l'interface utilisateur et faciliter la maintenance et les tests.

⏭️
