# 8.3. Mise en page avec les Panels

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les contrôles de mise en page (panels) constituent la pierre angulaire de la création d'interfaces utilisateur en WPF. Ils déterminent comment les éléments enfants sont organisés visuellement. WPF fournit plusieurs panels, chacun avec ses propres caractéristiques et cas d'utilisation.

## 8.3.1. Grid

Le `Grid` est le panel le plus puissant et flexible de WPF. Il organise les éléments en rangées et colonnes, similaire à une table HTML ou une feuille de calcul.

### Définition de base d'un Grid

```xml
<!-- Structure de base d'un Grid -->
<Grid>
    <!-- Définition des lignes -->
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/> <!-- Hauteur basée sur le contenu -->
        <RowDefinition Height="*"/>    <!-- Remplit l'espace disponible (proportion) -->
        <RowDefinition Height="2*"/>   <!-- Prend deux fois plus d'espace que le précédent -->
        <RowDefinition Height="100"/>  <!-- Hauteur fixe en pixels -->
    </Grid.RowDefinitions>

    <!-- Définition des colonnes -->
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="150"/> <!-- Largeur fixe en pixels -->
        <ColumnDefinition Width="*"/>   <!-- Remplit l'espace disponible -->
        <ColumnDefinition Width="Auto"/> <!-- Largeur basée sur le contenu -->
    </Grid.ColumnDefinitions>

    <!-- Placement des éléments dans les cellules -->
    <TextBlock Grid.Row="0" Grid.Column="0" Text="Ligne 0, Colonne 0"/>
    <Button Grid.Row="0" Grid.Column="1" Content="Ligne 0, Colonne 1"/>
    <TextBox Grid.Row="1" Grid.Column="0" Text="Ligne 1, Colonne 0"/>
</Grid>
```


### Types de dimensions dans un Grid

Le Grid offre trois types de dimensions pour les lignes et colonnes :

1. **Valeur fixe** (ex: `Height="100"`) - Définit une taille en pixels
2. **Auto** (`Height="Auto"`) - Dimensionne en fonction du contenu
3. **Proportionnelle** (`Height="*"` ou `Height="2*"`) - Distribue l'espace disponible selon les proportions

### Positionnement avancé des éléments

```xml
<!-- Positionnement avancé dans un Grid -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>

    <!-- Élément couvrant plusieurs cellules -->
    <Button Grid.Row="0" Grid.Column="0"
            Grid.RowSpan="2" Grid.ColumnSpan="2"
            Content="Grand bouton couvrant 4 cellules"/>

    <!-- Éléments individuels -->
    <TextBlock Grid.Row="0" Grid.Column="2" Text="Colonne 3"/>
    <TextBlock Grid.Row="1" Grid.Column="2" Text="Ligne 2, Colonne 3"/>
    <TextBlock Grid.Row="2" Grid.Column="0" Text="Ligne 3, Colonne 1"/>
    <TextBlock Grid.Row="2" Grid.Column="1" Grid.ColumnSpan="2"
               Text="Ligne 3, Colonnes 2-3"/>
</Grid>
```


### Marges et alignement

```xml
<!-- Marges et alignement dans un Grid -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>

    <!-- Différentes marges et alignements -->
    <Button Grid.Row="0" Grid.Column="0"
            Content="Centré"
            Margin="10"
            HorizontalAlignment="Center"
            VerticalAlignment="Center"/>

    <Button Grid.Row="0" Grid.Column="1"
            Content="Remplir"
            Margin="10"
            HorizontalAlignment="Stretch"
            VerticalAlignment="Stretch"/>

    <Button Grid.Row="1" Grid.Column="0"
            Content="En haut à droite"
            Margin="10"
            HorizontalAlignment="Right"
            VerticalAlignment="Top"/>

    <Button Grid.Row="1" Grid.Column="1"
            Content="En bas à gauche"
            Margin="10,20,30,40" <!-- Différentes marges: gauche,haut,droite,bas -->
            HorizontalAlignment="Left"
            VerticalAlignment="Bottom"/>
</Grid>
```


### Grille avec espacement

```xml
<!-- Grid avec espacement entre les cellules -->
<Grid ShowGridLines="True"> <!-- Affiche les lignes pour la démo -->
    <!-- Ajoutez un espacement entre les cellules -->
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="10"/> <!-- Ligne d'espacement -->
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="10"/> <!-- Colonne d'espacement -->
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>

    <!-- Contenu uniquement dans les cellules utilisables -->
    <Button Grid.Row="0" Grid.Column="0" Content="Cellule 1"/>
    <Button Grid.Row="0" Grid.Column="2" Content="Cellule 2"/>
    <Button Grid.Row="2" Grid.Column="0" Content="Cellule 3"/>
    <Button Grid.Row="2" Grid.Column="2" Content="Cellule 4"/>
</Grid>
```


### Grilles imbriquées

```xml
<!-- Grilles imbriquées -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- En-tête -->
    <TextBlock Grid.Row="0" Text="Démonstration de grilles imbriquées"
               FontSize="20" FontWeight="Bold" Margin="10"/>

    <!-- Contenu principal avec grille imbriquée -->
    <Grid Grid.Row="1">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="200"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Menu latéral -->
        <StackPanel Grid.Column="0" Background="LightGray">
            <Button Content="Option 1" Margin="5"/>
            <Button Content="Option 2" Margin="5"/>
            <Button Content="Option 3" Margin="5"/>
        </StackPanel>

        <!-- Zone de contenu avec une autre grille imbriquée -->
        <Grid Grid.Column="1">
            <Grid.RowDefinitions>
                <RowDefinition Height="*"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>

            <!-- Contenu principal -->
            <TextBlock Grid.Row="0" Text="Zone de contenu principal" Margin="10"/>

            <!-- Barre d'état -->
            <StatusBar Grid.Row="1">
                <StatusBarItem Content="Prêt"/>
            </StatusBar>
        </Grid>
    </Grid>
</Grid>
```


### Exemple complet d'un layout d'application

```xml
<!-- Layout d'application complet avec Grid -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/> <!-- En-tête -->
        <RowDefinition Height="*"/>    <!-- Contenu principal -->
        <RowDefinition Height="Auto"/> <!-- Barre d'état -->
    </Grid.RowDefinitions>

    <!-- En-tête de l'application -->
    <Border Grid.Row="0" Background="#2c3e50" Padding="10">
        <Grid>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>

            <TextBlock Grid.Column="0" Text="MonApplication"
                       Foreground="White" FontSize="18" FontWeight="Bold"/>

            <StackPanel Grid.Column="2" Orientation="Horizontal">
                <Button Content="Profil" Background="Transparent" Foreground="White"
                        BorderThickness="0" Margin="5,0"/>
                <Button Content="Déconnexion" Background="Transparent" Foreground="White"
                        BorderThickness="0" Margin="5,0"/>
            </StackPanel>
        </Grid>
    </Border>

    <!-- Contenu principal -->
    <Grid Grid.Row="1">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="220"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Menu latéral -->
        <Border Grid.Column="0" Background="#34495e">
            <StackPanel Margin="5">
                <TextBlock Text="MENU" Foreground="#95a5a6" Margin="10,20,0,10"/>

                <Button Content="Tableau de bord" Foreground="White"
                        Background="Transparent" BorderThickness="0"
                        HorizontalAlignment="Stretch" HorizontalContentAlignment="Left"
                        Padding="10,8" Margin="0,2"/>

                <Button Content="Projets" Foreground="White"
                        Background="Transparent" BorderThickness="0"
                        HorizontalAlignment="Stretch" HorizontalContentAlignment="Left"
                        Padding="10,8" Margin="0,2"/>

                <Button Content="Tâches" Foreground="White"
                        Background="Transparent" BorderThickness="0"
                        HorizontalAlignment="Stretch" HorizontalContentAlignment="Left"
                        Padding="10,8" Margin="0,2"/>

                <Button Content="Rapports" Foreground="White"
                        Background="Transparent" BorderThickness="0"
                        HorizontalAlignment="Stretch" HorizontalContentAlignment="Left"
                        Padding="10,8" Margin="0,2"/>

                <Button Content="Paramètres" Foreground="White"
                        Background="Transparent" BorderThickness="0"
                        HorizontalAlignment="Stretch" HorizontalContentAlignment="Left"
                        Padding="10,8" Margin="0,2"/>
            </StackPanel>
        </Border>

        <!-- Zone de contenu -->
        <Grid Grid.Column="1" Margin="20">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
            </Grid.RowDefinitions>

            <TextBlock Grid.Row="0" Text="Tableau de bord"
                       FontSize="24" FontWeight="Bold" Margin="0,0,0,20"/>

            <Grid Grid.Row="1">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="*"/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>

                <!-- Cartes d'information -->
                <Border Grid.Row="0" Grid.Column="0" Background="#3498db"
                        CornerRadius="5" Margin="0,0,10,10" Padding="15">
                    <StackPanel>
                        <TextBlock Text="Projets actifs" Foreground="White"/>
                        <TextBlock Text="12" Foreground="White"
                                   FontSize="36" FontWeight="Bold"/>
                    </StackPanel>
                </Border>

                <Border Grid.Row="0" Grid.Column="1" Background="#2ecc71"
                        CornerRadius="5" Margin="10,0,0,10" Padding="15">
                    <StackPanel>
                        <TextBlock Text="Tâches complétées" Foreground="White"/>
                        <TextBlock Text="64" Foreground="White"
                                   FontSize="36" FontWeight="Bold"/>
                    </StackPanel>
                </Border>

                <!-- Liste des tâches -->
                <Border Grid.Row="1" Grid.Column="0" Grid.ColumnSpan="2"
                        BorderBrush="#bdc3c7" BorderThickness="1"
                        CornerRadius="5" Padding="0">
                    <DockPanel>
                        <Border DockPanel.Dock="Top" Background="#ecf0f1"
                                BorderBrush="#bdc3c7" BorderThickness="0,0,0,1"
                                Padding="10">
                            <TextBlock Text="Tâches récentes" FontWeight="Bold"/>
                        </Border>

                        <ListView BorderThickness="0">
                            <ListViewItem>Mise à jour du site web</ListViewItem>
                            <ListViewItem>Correction de bugs</ListViewItem>
                            <ListViewItem>Réunion avec les clients</ListViewItem>
                            <ListViewItem>Rapport hebdomadaire</ListViewItem>
                            <ListViewItem>Planification du sprint</ListViewItem>
                        </ListView>
                    </DockPanel>
                </Border>
            </Grid>
        </Grid>
    </Grid>

    <!-- Barre d'état -->
    <StatusBar Grid.Row="2" Background="#ecf0f1">
        <StatusBarItem Content="Prêt"/>
        <Separator/>
        <StatusBarItem Content="Utilisateur: Admin"/>
        <Separator/>
        <StatusBarItem Content="Dernière mise à jour: Aujourd'hui 14:30"/>
    </StatusBar>
</Grid>
```


## 8.3.2. StackPanel

Le `StackPanel` organise les éléments dans une seule rangée ou colonne. C'est l'un des layouts les plus simples et les plus faciles à utiliser.

### Orientation verticale (par défaut)

```xml
<!-- StackPanel avec orientation verticale (par défaut) -->
<StackPanel Margin="10">
    <TextBlock Text="Titre du formulaire" FontSize="18" FontWeight="Bold" Margin="0,0,0,10"/>
    <TextBlock Text="Nom:"/>
    <TextBox Margin="0,0,0,10"/>
    <TextBlock Text="Email:"/>
    <TextBox Margin="0,0,0,10"/>
    <TextBlock Text="Message:"/>
    <TextBox Height="100" TextWrapping="Wrap" AcceptsReturn="True" Margin="0,0,0,10"/>
    <Button Content="Envoyer" HorizontalAlignment="Right" Padding="20,5"/>
</StackPanel>
```


### Orientation horizontale

```xml
<!-- StackPanel avec orientation horizontale -->
<StackPanel Orientation="Horizontal" Margin="10">
    <Button Content="Précédent" Padding="10,5" Margin="0,0,5,0"/>
    <Button Content="Annuler" Padding="10,5" Margin="0,0,5,0"/>
    <Button Content="Suivant" Padding="10,5"/>
</StackPanel>
```


### Alignement des éléments

```xml
<!-- Alignement des éléments dans un StackPanel -->
<StackPanel Margin="10">
    <TextBlock Text="Centré" HorizontalAlignment="Center"/>
    <TextBlock Text="À droite" HorizontalAlignment="Right"/>
    <TextBlock Text="À gauche" HorizontalAlignment="Left"/>
    <TextBlock Text="Étiré" HorizontalAlignment="Stretch"
               Background="LightGray"/>

    <!-- Dans un StackPanel horizontal, VerticalAlignment contrôle l'alignement -->
    <StackPanel Orientation="Horizontal" Height="100" Background="LightGray" Margin="0,10">
        <TextBlock Text="Haut" VerticalAlignment="Top" Margin="5"/>
        <TextBlock Text="Centre" VerticalAlignment="Center" Margin="5"/>
        <TextBlock Text="Bas" VerticalAlignment="Bottom" Margin="5"/>
        <TextBlock Text="Étiré" VerticalAlignment="Stretch"
                   Background="LightBlue" Margin="5"/>
    </StackPanel>
</StackPanel>
```


### StackPanels imbriqués

```xml
<!-- StackPanels imbriqués -->
<StackPanel Margin="10">
    <TextBlock Text="Formulaire d'inscription"
               FontSize="18" FontWeight="Bold"
               Margin="0,0,0,15"/>

    <!-- Groupe Informations personnelles -->
    <Border BorderBrush="Gray" BorderThickness="1" Padding="10" Margin="0,0,0,10">
        <StackPanel>
            <TextBlock Text="Informations personnelles"
                       FontWeight="Bold" Margin="0,0,0,10"/>

            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>

                <!-- Colonne de gauche -->
                <StackPanel Grid.Column="0" Margin="0,0,5,0">
                    <TextBlock Text="Prénom:"/>
                    <TextBox Margin="0,0,0,10"/>
                    <TextBlock Text="Date de naissance:"/>
                    <DatePicker Margin="0,0,0,10"/>
                </StackPanel>

                <!-- Colonne de droite -->
                <StackPanel Grid.Column="1" Margin="5,0,0,0">
                    <TextBlock Text="Nom:"/>
                    <TextBox Margin="0,0,0,10"/>
                    <TextBlock Text="Téléphone:"/>
                    <TextBox Margin="0,0,0,10"/>
                </StackPanel>
            </Grid>
        </StackPanel>
    </Border>

    <!-- Groupe Adresse -->
    <Border BorderBrush="Gray" BorderThickness="1" Padding="10" Margin="0,0,0,10">
        <StackPanel>
            <TextBlock Text="Adresse" FontWeight="Bold" Margin="0,0,0,10"/>
            <TextBlock Text="Rue:"/>
            <TextBox Margin="0,0,0,10"/>

            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>

                <StackPanel Grid.Column="0" Margin="0,0,5,0">
                    <TextBlock Text="Code postal:"/>
                    <TextBox Margin="0,0,0,10"/>
                </StackPanel>

                <StackPanel Grid.Column="1" Margin="5,0,5,0">
                    <TextBlock Text="Ville:"/>
                    <TextBox Margin="0,0,0,10"/>
                </StackPanel>

                <StackPanel Grid.Column="2" Margin="5,0,0,0">
                    <TextBlock Text="Pays:"/>
                    <ComboBox Margin="0,0,0,10">
                        <ComboBoxItem Content="France"/>
                        <ComboBoxItem Content="Belgique"/>
                        <ComboBoxItem Content="Suisse"/>
                        <ComboBoxItem Content="Canada"/>
                    </ComboBox>
                </StackPanel>
            </Grid>
        </StackPanel>
    </Border>

    <!-- Boutons -->
    <StackPanel Orientation="Horizontal" HorizontalAlignment="Right" Margin="0,10,0,0">
        <Button Content="Annuler" Padding="15,5" Margin="0,0,10,0"/>
        <Button Content="S'inscrire" Padding="15,5" Background="#2ecc71" Foreground="White"/>
    </StackPanel>
</StackPanel>
```


### Limitation et débordement

```xml
<!-- Gestion du débordement dans un StackPanel -->
<Border BorderBrush="Gray" BorderThickness="1" Height="200" Margin="10">
    <!-- Le StackPanel ne s'arrête pas à la limite de la Border et continue en dehors -->
    <StackPanel>
        <TextBlock Text="Le StackPanel n'a pas de limite de défilement intégrée"
                   Margin="10" FontWeight="Bold"/>
        <Rectangle Height="50" Fill="Red" Margin="10"/>
        <Rectangle Height="50" Fill="Green" Margin="10"/>
        <Rectangle Height="50" Fill="Blue" Margin="10"/>
        <Rectangle Height="50" Fill="Yellow" Margin="10"/>
        <Rectangle Height="50" Fill="Purple" Margin="10"/>
        <!-- Les éléments continuent et certains sont invisibles -->
    </StackPanel>
</Border>

<!-- Solution: Envelopper le StackPanel dans un ScrollViewer -->
<Border BorderBrush="Gray" BorderThickness="1" Height="200" Margin="10">
    <ScrollViewer>
        <StackPanel>
            <TextBlock Text="Avec ScrollViewer, tous les éléments sont accessibles"
                       Margin="10" FontWeight="Bold"/>
            <Rectangle Height="50" Fill="Red" Margin="10"/>
            <Rectangle Height="50" Fill="Green" Margin="10"/>
            <Rectangle Height="50" Fill="Blue" Margin="10"/>
            <Rectangle Height="50" Fill="Yellow" Margin="10"/>
            <Rectangle Height="50" Fill="Purple" Margin="10"/>
        </StackPanel>
    </ScrollViewer>
</Border>
```


## 8.3.3. WrapPanel

Le `WrapPanel` organise les éléments en séquence horizontale ou verticale et passe à la ligne lorsqu'il n'y a plus d'espace disponible.

### Orientation horizontale (par défaut)

```xml
<!-- WrapPanel avec orientation horizontale (par défaut) -->
<Border BorderBrush="Gray" BorderThickness="1" Margin="10">
    <WrapPanel>
        <Button Content="Bouton 1" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 2" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 3" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 4" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 5" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 6" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 7" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 8" Margin="5" Padding="10,5"/>
        <Button Content="Bouton avec un texte plus long" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 10" Margin="5" Padding="10,5"/>
    </WrapPanel>
</Border>
```


### Orientation verticale

```xml
<!-- WrapPanel avec orientation verticale -->
<Border BorderBrush="Gray" BorderThickness="1" Margin="10" Height="200">
    <WrapPanel Orientation="Vertical">
        <Button Content="Bouton 1" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 2" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 3" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 4" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 5" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 6" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 7" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 8" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 9" Margin="5" Padding="10,5"/>
        <Button Content="Bouton 10" Margin="5" Padding="10,5"/>
    </WrapPanel>
</Border>
```


### Taille des éléments dans un WrapPanel

```xml
<!-- WrapPanel harmonise la taille des éléments -->
<WrapPanel Margin="10" ItemWidth="150" ItemHeight="80">
    <Button Content="Bouton 1" Margin="5"/>
    <Button Content="Bouton avec un texte beaucoup plus long" Margin="5"/>
    <Button Content="Bouton 3" Margin="5"/>
    <Button Content="4" Margin="5"/>
    <Rectangle Fill="BlueViolet" Margin="5"/>
    <Ellipse Fill="OrangeRed" Margin="5"/>
</WrapPanel>
```


### Cas d'utilisation : Galerie d'images

```xml
<!-- Galerie d'images avec WrapPanel -->
<ScrollViewer Margin="10" HorizontalScrollBarVisibility="Disabled">
    <WrapPanel>
        <!-- Simulons des images avec des rectangles colorés -->
        <Border Margin="5" BorderBrush="Gray" BorderThickness="1">
            <Grid Width="150" Height="100">
                <Rectangle Fill="LightBlue"/>
                <TextBlock Text="Image 1" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <Border Margin="5" BorderBrush="Gray" BorderThickness="1">
            <Grid Width="150" Height="100">
                <Rectangle Fill="LightGreen"/>
                <TextBlock Text="Image 2" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <Border Margin="5" BorderBrush="Gray" BorderThickness="1">
            <Grid Width="150" Height="100">
                <Rectangle Fill="LightCoral"/>
                <TextBlock Text="Image 3" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <Border Margin="5" BorderBrush="Gray" BorderThickness="1">
            <Grid Width="150" Height="100">
                <Rectangle Fill="LightGoldenrodYellow"/>
                <TextBlock Text="Image 4" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <Border Margin="5" BorderBrush="Gray" BorderThickness="1">
            <Grid Width="150" Height="100">
                <Rectangle Fill="LightSalmon"/>
                <TextBlock Text="Image 5" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <Border Margin="5" BorderBrush="Gray" BorderThickness="1">
            <Grid Width="150" Height="100">
                <Rectangle Fill="LightSeaGreen"/>
                <TextBlock Text="Image 6" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Grid>
        </Border>

        <!-- Ajoutez plus d'images selon vos besoins -->
    </WrapPanel>
</ScrollViewer>
```

### Cas d'utilisation : Interface de tags

```xml
<!-- Interface de tags avec WrapPanel -->
<StackPanel Margin="10">
    <TextBlock Text="Sélectionnez des tags:" Margin="0,0,0,5"/>

    <WrapPanel>
        <Border Background="#3498db" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="HTML" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#2ecc71" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="CSS" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#e74c3c" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="JavaScript" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#9b59b6" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="React" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#f1c40f" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="Angular" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#1abc9c" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="Vue.js" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#34495e" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="TypeScript" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <Border Background="#e67e22" CornerRadius="15" Margin="3" Padding="8,4">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="Node.js" Foreground="White"/>
                <Button Content="x" Background="Transparent" BorderThickness="0"
                        Foreground="White" FontSize="10" Padding="4,0" Margin="4,0,0,0"/>
            </StackPanel>
        </Border>

        <!-- Bouton pour ajouter un nouveau tag -->
        <Border BorderBrush="Gray" BorderThickness="1" CornerRadius="15"
                Margin="3" Padding="8,4">
            <TextBlock Text="+ Ajouter un tag" Foreground="Gray"/>
        </Border>
    </WrapPanel>
</StackPanel>
```


## 8.3.4. DockPanel

Le `DockPanel` permet d'ancrer des éléments sur les bords (haut, bas, gauche, droite) ou au centre de son espace.

### Utilisation de base

```xml
<!-- DockPanel de base -->
<DockPanel Margin="10">
    <Border DockPanel.Dock="Top" Background="LightBlue" Height="50">
        <TextBlock Text="En-tête (Top)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border DockPanel.Dock="Bottom" Background="LightGreen" Height="50">
        <TextBlock Text="Pied de page (Bottom)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border DockPanel.Dock="Left" Background="LightCoral" Width="100">
        <TextBlock Text="Gauche (Left)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border DockPanel.Dock="Right" Background="LightYellow" Width="100">
        <TextBlock Text="Droite (Right)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border Background="LightGray">
        <TextBlock Text="Contenu central (remplit l'espace restant)"
                   VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>
</DockPanel>
```


### Propriété LastChildFill

```xml
<!-- DockPanel avec LastChildFill="False" -->
<DockPanel LastChildFill="False" Margin="10" Height="300">
    <Border DockPanel.Dock="Top" Background="LightBlue" Height="50">
        <TextBlock Text="En-tête (Top)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border DockPanel.Dock="Bottom" Background="LightGreen" Height="50">
        <TextBlock Text="Pied de page (Bottom)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border DockPanel.Dock="Left" Background="LightCoral" Width="100">
        <TextBlock Text="Gauche (Left)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <Border DockPanel.Dock="Right" Background="LightYellow" Width="100">
        <TextBlock Text="Droite (Right)" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </Border>

    <!-- Avec LastChildFill="False", une zone vide apparaît au centre -->
</DockPanel>
```


### Ordre des éléments dans un DockPanel

```xml
<!-- L'ordre des éléments est important dans un DockPanel -->
<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Premier exemple: Gauche puis Haut -->
    <DockPanel Grid.Row="0">
        <TextBlock Text="Cet exemple montre Left puis Top" DockPanel.Dock="Top"
                   Background="LightGray" Padding="5"/>

        <Border DockPanel.Dock="Left" Background="LightBlue" Width="100">
            <TextBlock Text="Left" VerticalAlignment="Center" HorizontalAlignment="Center"/>
        </Border>

        <Border DockPanel.Dock="Top" Background="LightGreen" Height="50">
            <TextBlock Text="Top" VerticalAlignment="Center" HorizontalAlignment="Center"/>
        </Border>

        <Border Background="LightGray">
            <TextBlock Text="Contenu" VerticalAlignment="Center" HorizontalAlignment="Center"/>
        </Border>
    </DockPanel>

    <!-- Second exemple: Haut puis Gauche -->
    <DockPanel Grid.Row="1">
        <TextBlock Text="Cet exemple montre Top puis Left" DockPanel.Dock="Top"
                   Background="LightGray" Padding="5"/>

        <Border DockPanel.Dock="Top" Background="LightGreen" Height="50">
            <TextBlock Text="Top" VerticalAlignment="Center" HorizontalAlignment="Center"/>
        </Border>

        <Border DockPanel.Dock="Left" Background="LightBlue" Width="100">
            <TextBlock Text="Left" VerticalAlignment="Center" HorizontalAlignment="Center"/>
        </Border>

        <Border Background="LightGray">
            <TextBlock Text="Contenu" VerticalAlignment="Center" HorizontalAlignment="Center"/>
        </Border>
    </DockPanel>
</Grid>
```


### Cas d'utilisation : Layout d'application standard

```xml
<!-- Layout d'application standard avec DockPanel -->
<DockPanel Margin="0">
    <!-- Barre de menus -->
    <Menu DockPanel.Dock="Top" Background="#f5f5f5">
        <MenuItem Header="Fichier">
            <MenuItem Header="Nouveau"/>
            <MenuItem Header="Ouvrir"/>
            <MenuItem Header="Enregistrer"/>
            <Separator/>
            <MenuItem Header="Quitter"/>
        </MenuItem>
        <MenuItem Header="Édition">
            <MenuItem Header="Couper"/>
            <MenuItem Header="Copier"/>
            <MenuItem Header="Coller"/>
        </MenuItem>
        <MenuItem Header="Aide">
            <MenuItem Header="À propos"/>
        </MenuItem>
    </Menu>

    <!-- Barre d'outils -->
    <ToolBarTray DockPanel.Dock="Top" Background="#f0f0f0">
        <ToolBar>
            <Button Content="Nouveau" Padding="5,2"/>
            <Button Content="Ouvrir" Padding="5,2"/>
            <Button Content="Enregistrer" Padding="5,2"/>
            <Separator/>
            <Button Content="Couper" Padding="5,2"/>
            <Button Content="Copier" Padding="5,2"/>
            <Button Content="Coller" Padding="5,2"/>
        </ToolBar>
    </ToolBarTray>

    <!-- Barre d'état -->
    <StatusBar DockPanel.Dock="Bottom" Background="#f5f5f5">
        <StatusBarItem Content="Prêt"/>
        <Separator/>
        <StatusBarItem Content="Ligne: 1, Colonne: 1"/>
        <StatusBarItem HorizontalAlignment="Right" Content="100%"/>
    </StatusBar>

    <!-- Panneau de navigation -->
    <Border DockPanel.Dock="Left" Width="200" BorderBrush="#e0e0e0" BorderThickness="0,0,1,0">
        <TreeView>
            <TreeViewItem Header="Projet A" IsExpanded="True">
                <TreeViewItem Header="Fichier 1"/>
                <TreeViewItem Header="Fichier 2"/>
                <TreeViewItem Header="Fichier 3"/>
            </TreeViewItem>
            <TreeViewItem Header="Projet B">
                <TreeViewItem Header="Fichier 1"/>
                <TreeViewItem Header="Fichier 2"/>
            </TreeViewItem>
            <TreeViewItem Header="Projet C"/>
        </TreeView>
    </Border>

    <!-- Panneau des propriétés -->
    <Border DockPanel.Dock="Right" Width="250" BorderBrush="#e0e0e0" BorderThickness="1,0,0,0">
        <DockPanel>
            <TextBlock DockPanel.Dock="Top" Background="#f5f5f5" Padding="5"
                       Text="Propriétés" FontWeight="Bold"/>

            <ScrollViewer VerticalScrollBarVisibility="Auto">
                <StackPanel Margin="5">
                    <GroupBox Header="Général">
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

                            <TextBlock Grid.Row="0" Grid.Column="0" Text="Nom:" Margin="0,5,10,5"/>
                            <TextBox Grid.Row="0" Grid.Column="1" Margin="0,5,0,5"/>

                            <TextBlock Grid.Row="1" Grid.Column="0" Text="Type:" Margin="0,5,10,5"/>
                            <ComboBox Grid.Row="1" Grid.Column="1" Margin="0,5,0,5"/>

                            <TextBlock Grid.Row="2" Grid.Column="0" Text="Taille:" Margin="0,5,10,5"/>
                            <TextBox Grid.Row="2" Grid.Column="1" Margin="0,5,0,5" IsReadOnly="True" Text="1.2 KB"/>
                        </Grid>
                    </GroupBox>

                    <GroupBox Header="Avancé" Margin="0,10,0,0">
                        <StackPanel>
                            <CheckBox Content="Lecture seule" Margin="0,5"/>
                            <CheckBox Content="Caché" Margin="0,5"/>
                            <CheckBox Content="Archivé" Margin="0,5"/>
                        </StackPanel>
                    </GroupBox>
                </StackPanel>
            </ScrollViewer>
        </DockPanel>
    </Border>

    <!-- Zone principale de contenu -->
    <Border Background="White" BorderBrush="#e0e0e0" BorderThickness="1">
        <Grid>
            <TabControl>
                <TabItem Header="Document1.txt">
                    <TextBox AcceptsReturn="True" TextWrapping="Wrap"
                             VerticalScrollBarVisibility="Auto"
                             Text="Ceci est le contenu du document 1.&#x0a;&#x0a;Vous pouvez éditer ce texte."/>
                </TabItem>
                <TabItem Header="Document2.txt">
                    <TextBox AcceptsReturn="True" TextWrapping="Wrap"
                             VerticalScrollBarVisibility="Auto"
                             Text="Ceci est le contenu du document 2."/>
                </TabItem>
                <TabItem Header="+">
                    <!-- Onglet pour créer un nouveau document -->
                </TabItem>
            </TabControl>
        </Grid>
    </Border>
</DockPanel>
```


## 8.3.5. Canvas

Le `Canvas` est un conteneur qui permet de positionner les éléments de manière absolue en utilisant des coordonnées (x,y).

### Positionnement de base

```xml
<!-- Canvas avec positionnement absolu -->
<Canvas Width="400" Height="300" Background="LightGray" Margin="10">
    <!-- Rectangle rouge en haut à gauche -->
    <Rectangle Canvas.Left="10" Canvas.Top="10" Width="100" Height="50" Fill="Red"/>

    <!-- Cercle bleu au milieu -->
    <Ellipse Canvas.Left="150" Canvas.Top="100" Width="80" Height="80" Fill="Blue"/>

    <!-- Texte en bas à droite -->
    <TextBlock Canvas.Left="250" Canvas.Top="200" Text="Position absolue" FontSize="16"/>

    <!-- Triangle (à l'aide d'un Polygon) -->
    <Polygon Canvas.Left="50" Canvas.Top="150"
             Points="0,0 50,50 100,0"
             Fill="Green"/>

    <!-- Ligne -->
    <Line Canvas.Left="200" Canvas.Top="50"
          X1="0" Y1="0" X2="100" Y2="100"
          Stroke="Purple" StrokeThickness="2"/>
</Canvas>
```


### Superposition d'éléments

```xml
<!-- Canvas avec éléments superposés -->
<Canvas Width="300" Height="200" Background="LightGray" Margin="10">
    <!-- Les éléments sont dessinés dans l'ordre où ils apparaissent -->
    <Rectangle Canvas.Left="50" Canvas.Top="50" Width="150" Height="100" Fill="LightBlue"/>
    <Ellipse Canvas.Left="100" Canvas.Top="80" Width="100" Height="100" Fill="LightGreen" Opacity="0.7"/>
    <TextBlock Canvas.Left="120" Canvas.Top="120" Text="Superposition" FontWeight="Bold"/>
</Canvas>
```


### Contrôle du z-index

```xml
<!-- Contrôle de l'ordre z avec ZIndex -->
<Canvas Width="300" Height="200" Background="LightGray" Margin="10">
    <!-- Malgré l'ordre dans le XAML, ZIndex contrôle l'ordre d'empilement -->
    <Rectangle Canvas.Left="50" Canvas.Top="50" Width="150" Height="100"
               Fill="LightBlue" Canvas.ZIndex="3"/>

    <Ellipse Canvas.Left="100" Canvas.Top="80" Width="100" Height="100"
             Fill="LightGreen" Opacity="0.7" Canvas.ZIndex="1"/>

    <TextBlock Canvas.Left="120" Canvas.Top="120" Text="Z-Index"
               FontWeight="Bold" Canvas.ZIndex="2"/>
</Canvas>
```


### Dessin vectoriel

```xml
<!-- Utilisation du Canvas pour le dessin vectoriel -->
<Canvas Width="400" Height="300" Background="White" Margin="10">
    <!-- Fond avec dégradé -->
    <Rectangle Width="400" Height="300">
        <Rectangle.Fill>
            <LinearGradientBrush StartPoint="0,0" EndPoint="1,1">
                <GradientStop Color="LightBlue" Offset="0"/>
                <GradientStop Color="White" Offset="1"/>
            </LinearGradientBrush>
        </Rectangle.Fill>
    </Rectangle>

    <!-- Soleil -->
    <Ellipse Canvas.Left="50" Canvas.Top="50" Width="60" Height="60">
        <Ellipse.Fill>
            <RadialGradientBrush>
                <GradientStop Color="Yellow" Offset="0"/>
                <GradientStop Color="Orange" Offset="1"/>
            </RadialGradientBrush>
        </Ellipse.Fill>
    </Ellipse>

    <!-- Maison -->
    <Rectangle Canvas.Left="150" Canvas.Top="150" Width="100" Height="80" Fill="Brown"/>
    <Polygon Points="150,150 250,150 200,100" Canvas.Left="0" Canvas.Top="0" Fill="Red"/>

    <!-- Porte -->
    <Rectangle Canvas.Left="185" Canvas.Top="180" Width="30" Height="50" Fill="DarkBrown"/>

    <!-- Fenêtre -->
    <Rectangle Canvas.Left="165" Canvas.Top="170" Width="15" Height="15" Fill="LightYellow"/>
    <Rectangle Canvas.Left="220" Canvas.Top="170" Width="15" Height="15" Fill="LightYellow"/>

    <!-- Arbre -->
    <Rectangle Canvas.Left="300" Canvas.Top="160" Width="15" Height="70" Fill="Brown"/>
    <Ellipse Canvas.Left="275" Canvas.Top="100" Width="65" Height="90" Fill="Green"/>

    <!-- Chemin -->
    <Path Canvas.Left="200" Canvas.Top="230" Stroke="Gray" StrokeThickness="10">
        <Path.Data>
            <PathGeometry>
                <PathFigure StartPoint="0,0">
                    <LineSegment Point="100,0"/>
                </PathFigure>
            </PathGeometry>
        </Path.Data>
    </Path>
</Canvas>
```


### Mini-éditeur de dessin

```xml
<!-- Interface simplifiée d'un éditeur de dessin -->
<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- Barre d'outils -->
    <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="0,0,0,10">
        <Button Content="Sélection" Padding="10,5" Margin="0,0,5,0"/>
        <Button Content="Rectangle" Padding="10,5" Margin="0,0,5,0"/>
        <Button Content="Ellipse" Padding="10,5" Margin="0,0,5,0"/>
        <Button Content="Ligne" Padding="10,5" Margin="0,0,5,0"/>
        <Rectangle Width="1" Height="24" Fill="LightGray" Margin="5,0"/>
        <StackPanel Orientation="Horizontal" VerticalAlignment="Center" Margin="5,0">
            <TextBlock Text="Couleur:" VerticalAlignment="Center" Margin="0,0,5,0"/>
            <Rectangle Width="20" Height="20" Fill="Black" Stroke="Gray" Margin="0,0,5,0"/>
            <ComboBox Width="60" SelectedIndex="0">
                <ComboBoxItem Content="Noir"/>
                <ComboBoxItem Content="Rouge"/>
                <ComboBoxItem Content="Bleu"/>
                <ComboBoxItem Content="Vert"/>
            </ComboBox>
        </StackPanel>
        <StackPanel Orientation="Horizontal" VerticalAlignment="Center" Margin="5,0">
            <TextBlock Text="Épaisseur:" VerticalAlignment="Center" Margin="0,0,5,0"/>
            <ComboBox Width="50" SelectedIndex="0">
                <ComboBoxItem Content="1px"/>
                <ComboBoxItem Content="2px"/>
                <ComboBoxItem Content="3px"/>
                <ComboBoxItem Content="5px"/>
            </ComboBox>
        </StackPanel>
    </StackPanel>

    <!-- Zone de dessin (Canvas) -->
    <Border Grid.Row="1" BorderBrush="Gray" BorderThickness="1">
        <Canvas Background="White">
            <!-- Exemple d'éléments dessinés -->
            <Rectangle Canvas.Left="50" Canvas.Top="50" Width="100" Height="80"
                       Stroke="Black" StrokeThickness="2" Fill="Transparent"/>

            <Ellipse Canvas.Left="200" Canvas.Top="100" Width="120" Height="80"
                     Stroke="Blue" StrokeThickness="2" Fill="Transparent"/>

            <Line Canvas.Left="0" Canvas.Top="0" X1="400" Y1="50" X2="500" Y2="150"
                  Stroke="Red" StrokeThickness="3"/>

            <!-- Poignées de sélection (simulées) -->
            <Rectangle Canvas.Left="46" Canvas.Top="46" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="96" Canvas.Top="46" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="146" Canvas.Top="46" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="46" Canvas.Top="90" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="146" Canvas.Top="90" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="46" Canvas.Top="126" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="96" Canvas.Top="126" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
            <Rectangle Canvas.Left="146" Canvas.Top="126" Width="8" Height="8"
                       Fill="White" Stroke="Black" StrokeThickness="1"/>
        </Canvas>
    </Border>
</Grid>
```

## 8.3.6. Layouts combinés et complexes

Dans les applications réelles, il est rare d'utiliser un seul type de panel. La puissance de WPF vient de la possibilité de combiner différents panels pour créer des interfaces complexes et flexibles.

### Combinaison de différents panels

```xml
<!-- Interface complexe combinant différents panels -->
<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/> <!-- En-tête -->
        <RowDefinition Height="*"/>    <!-- Contenu principal -->
        <RowDefinition Height="Auto"/> <!-- Barre d'état -->
    </Grid.RowDefinitions>

    <!-- En-tête avec DockPanel -->
    <DockPanel Grid.Row="0" Background="#2c3e50" LastChildFill="True">
        <Button DockPanel.Dock="Right" Content="Profil"
                Background="Transparent" Foreground="White"
                BorderThickness="0" Margin="10,5"/>

        <Button DockPanel.Dock="Right" Content="Paramètres"
                Background="Transparent" Foreground="White"
                BorderThickness="0" Margin="10,5"/>

        <TextBlock Text="Application de démonstration"
                   Foreground="White" FontSize="18"
                   VerticalAlignment="Center" Margin="15,10"/>
    </DockPanel>

    <!-- Contenu principal avec Grid et panels imbriqués -->
    <Grid Grid.Row="1">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="220"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Navigation latérale avec StackPanel -->
        <Border Grid.Column="0" Background="#34495e">
            <StackPanel Margin="10">
                <TextBlock Text="NAVIGATION" Foreground="#95a5a6" Margin="5,10"/>

                <Button Content="Tableau de bord" Margin="0,5"
                        Foreground="White" Background="Transparent"
                        BorderThickness="0" HorizontalContentAlignment="Left"/>

                <Button Content="Utilisateurs" Margin="0,5"
                        Foreground="White" Background="Transparent"
                        BorderThickness="0" HorizontalContentAlignment="Left"/>

                <Button Content="Statistiques" Margin="0,5"
                        Foreground="White" Background="Transparent"
                        BorderThickness="0" HorizontalContentAlignment="Left"/>

                <TextBlock Text="ACTIONS" Foreground="#95a5a6" Margin="5,20,5,10"/>

                <Button Content="Créer un rapport" Margin="0,5"
                        Foreground="White" Background="Transparent"
                        BorderThickness="0" HorizontalContentAlignment="Left"/>

                <Button Content="Importer des données" Margin="0,5"
                        Foreground="White" Background="Transparent"
                        BorderThickness="0" HorizontalContentAlignment="Left"/>

                <Button Content="Exporter des données" Margin="0,5"
                        Foreground="White" Background="Transparent"
                        BorderThickness="0" HorizontalContentAlignment="Left"/>
            </StackPanel>
        </Border>

        <!-- Contenu principal avec DockPanel -->
        <DockPanel Grid.Column="1">
            <!-- En-tête de section -->
            <Border DockPanel.Dock="Top" Background="#ecf0f1" Padding="15">
                <Grid>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="*"/>
                        <ColumnDefinition Width="Auto"/>
                    </Grid.ColumnDefinitions>

                    <StackPanel Grid.Column="0">
                        <TextBlock Text="Tableau de bord" FontSize="20" FontWeight="Bold"/>
                        <TextBlock Text="Vue d'ensemble de l'application" Foreground="#7f8c8d"/>
                    </StackPanel>

                    <StackPanel Grid.Column="1" Orientation="Horizontal">
                        <Button Content="Exporter" Padding="10,5" Margin="0,0,10,0"/>
                        <Button Content="Actualiser" Padding="10,5"/>
                    </StackPanel>
                </Grid>
            </Border>

            <!-- Statistiques avec WrapPanel -->
            <WrapPanel DockPanel.Dock="Top" Margin="15,15,0,0">
                <Border Width="220" Height="120" Margin="0,0,15,15" Background="#3498db" CornerRadius="5">
                    <Grid Margin="15">
                        <Grid.RowDefinitions>
                            <RowDefinition Height="Auto"/>
                            <RowDefinition Height="*"/>
                            <RowDefinition Height="Auto"/>
                        </Grid.RowDefinitions>

                        <TextBlock Grid.Row="0" Text="UTILISATEURS" Foreground="#fff" Opacity="0.7"/>
                        <TextBlock Grid.Row="1" Text="1,257" Foreground="White"
                                   FontSize="36" FontWeight="Bold" VerticalAlignment="Center"/>
                        <TextBlock Grid.Row="2" Text="+12% ce mois" Foreground="#fff" Opacity="0.7"/>
                    </Grid>
                </Border>

                <Border Width="220" Height="120" Margin="0,0,15,15" Background="#2ecc71" CornerRadius="5">
                    <Grid Margin="15">
                        <Grid.RowDefinitions>
                            <RowDefinition Height="Auto"/>
                            <RowDefinition Height="*"/>
                            <RowDefinition Height="Auto"/>
                        </Grid.RowDefinitions>

                        <TextBlock Grid.Row="0" Text="REVENUS" Foreground="#fff" Opacity="0.7"/>
                        <TextBlock Grid.Row="1" Text="8,521 €" Foreground="White"
                                   FontSize="36" FontWeight="Bold" VerticalAlignment="Center"/>
                        <TextBlock Grid.Row="2" Text="+8% ce mois" Foreground="#fff" Opacity="0.7"/>
                    </Grid>
                </Border>

                <Border Width="220" Height="120" Margin="0,0,15,15" Background="#e74c3c" CornerRadius="5">
                    <Grid Margin="15">
                        <Grid.RowDefinitions>
                            <RowDefinition Height="Auto"/>
                            <RowDefinition Height="*"/>
                            <RowDefinition Height="Auto"/>
                        </Grid.RowDefinitions>

                        <TextBlock Grid.Row="0" Text="COMMANDES" Foreground="#fff" Opacity="0.7"/>
                        <TextBlock Grid.Row="1" Text="427" Foreground="White"
                                   FontSize="36" FontWeight="Bold" VerticalAlignment="Center"/>
                        <TextBlock Grid.Row="2" Text="+5% ce mois" Foreground="#fff" Opacity="0.7"/>
                    </Grid>
                </Border>
            </WrapPanel>

            <!-- Contenu détaillé avec ScrollViewer et Grid imbriqué -->
            <ScrollViewer VerticalScrollBarVisibility="Auto" Margin="15,0,15,15">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                    </Grid.RowDefinitions>

                    <!-- Graphique (simulé) -->
                    <Border Grid.Row="0" BorderBrush="#bdc3c7" BorderThickness="1"
                            CornerRadius="5" Padding="15" Margin="0,0,0,15">
                        <StackPanel>
                            <TextBlock Text="Évolution des ventes" FontWeight="Bold" Margin="0,0,0,10"/>

                            <!-- Simuler un graphique avec Canvas -->
                            <Canvas Height="200">
                                <!-- Lignes horizontales de référence -->
                                <Line X1="0" Y1="0" X2="500" Y2="0"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="0" Y1="50" X2="500" Y2="50"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="0" Y1="100" X2="500" Y2="100"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="0" Y1="150" X2="500" Y2="150"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="0" Y1="200" X2="500" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>

                                <!-- Lignes verticales de référence -->
                                <Line X1="0" Y1="0" X2="0" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="100" Y1="0" X2="100" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="200" Y1="0" X2="200" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="300" Y1="0" X2="300" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="400" Y1="0" X2="400" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>
                                <Line X1="500" Y1="0" X2="500" Y2="200"
                                      Stroke="#ecf0f1" StrokeThickness="1"/>

                                <!-- Courbe de données simulée (Polyline) -->
                                <Polyline Points="0,150 100,100 200,120 300,80 400,50 500,70"
                                          Stroke="#3498db" StrokeThickness="3"
                                          Fill="#3498db" Opacity="0.2"/>

                                <!-- Labels -->
                                <TextBlock Canvas.Left="0" Canvas.Top="205" Text="Jan"/>
                                <TextBlock Canvas.Left="100" Canvas.Top="205" Text="Fév"/>
                                <TextBlock Canvas.Left="200" Canvas.Top="205" Text="Mar"/>
                                <TextBlock Canvas.Left="300" Canvas.Top="205" Text="Avr"/>
                                <TextBlock Canvas.Left="400" Canvas.Top="205" Text="Mai"/>
                                <TextBlock Canvas.Left="500" Canvas.Top="205" Text="Juin"/>
                            </Canvas>
                        </StackPanel>
                    </Border>

                    <!-- Tableau de données -->
                    <Border Grid.Row="1" BorderBrush="#bdc3c7" BorderThickness="1"
                            CornerRadius="5" Padding="0">
                        <DockPanel>
                            <Border DockPanel.Dock="Top" Background="#ecf0f1"
                                    BorderBrush="#bdc3c7" BorderThickness="0,0,0,1"
                                    Padding="15,10">
                                <TextBlock Text="Dernières transactions" FontWeight="Bold"/>
                            </Border>

                            <ListView BorderThickness="0">
                                <ListView.View>
                                    <GridView>
                                        <GridViewColumn Header="ID" Width="60"/>
                                        <GridViewColumn Header="Date" Width="120"/>
                                        <GridViewColumn Header="Client" Width="150"/>
                                        <GridViewColumn Header="Montant" Width="100"/>
                                        <GridViewColumn Header="Statut" Width="100"/>
                                    </GridView>
                                </ListView.View>

                                <ListViewItem>
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Text="1001" Width="60"/>
                                        <TextBlock Text="01/06/2023" Width="120"/>
                                        <TextBlock Text="Dupont S.A." Width="150"/>
                                        <TextBlock Text="1 250,00 €" Width="100"/>
                                        <TextBlock Text="Payé" Foreground="Green" Width="100"/>
                                    </StackPanel>
                                </ListViewItem>

                                <ListViewItem>
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Text="1002" Width="60"/>
                                        <TextBlock Text="03/06/2023" Width="120"/>
                                        <TextBlock Text="Martin & Co" Width="150"/>
                                        <TextBlock Text="865,50 €" Width="100"/>
                                        <TextBlock Text="En attente" Foreground="Orange" Width="100"/>
                                    </StackPanel>
                                </ListViewItem>

                                <ListViewItem>
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Text="1003" Width="60"/>
                                        <TextBlock Text="05/06/2023" Width="120"/>
                                        <TextBlock Text="Léger SARL" Width="150"/>
                                        <TextBlock Text="2 540,75 €" Width="100"/>
                                        <TextBlock Text="Payé" Foreground="Green" Width="100"/>
                                    </StackPanel>
                                </ListViewItem>

                                <ListViewItem>
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Text="1004" Width="60"/>
                                        <TextBlock Text="08/06/2023" Width="120"/>
                                        <TextBlock Text="Mercier Inc." Width="150"/>
                                        <TextBlock Text="780,00 €" Width="100"/>
                                        <TextBlock Text="Annulé" Foreground="Red" Width="100"/>
                                    </StackPanel>
                                </ListViewItem>

                                <ListViewItem>
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Text="1005" Width="60"/>
                                        <TextBlock Text="10/06/2023" Width="120"/>
                                        <TextBlock Text="Simon & Fils" Width="150"/>
                                        <TextBlock Text="1 820,25 €" Width="100"/>
                                        <TextBlock Text="Payé" Foreground="Green" Width="100"/>
                                    </StackPanel>
                                </ListViewItem>
                            </ListView>
                        </DockPanel>
                    </Border>
                </Grid>
            </ScrollViewer>
        </DockPanel>
    </Grid>

    <!-- Barre d'état -->
    <StatusBar Grid.Row="2" Background="#ecf0f1">
        <StatusBarItem Content="Dernière mise à jour: 15/06/2023 14:30"/>
        <Separator/>
        <StatusBarItem Content="Utilisateur: admin@example.com"/>
        <StatusBarItem Content="Version 1.0" HorizontalAlignment="Right"/>
    </StatusBar>
</Grid>
```


### Techniques pour les interfaces complexes

#### Utilisation de définitions de grille proportionnelles et imbriquées

```xml
<!-- Grilles imbriquées et proportionnelles -->
<Grid Margin="10">
    <!-- Définition de colonnes avec grid splitter -->
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="2*" MinWidth="200"/>
        <ColumnDefinition Width="Auto"/>
        <ColumnDefinition Width="3*" MinWidth="300"/>
    </Grid.ColumnDefinitions>

    <!-- Panneau de navigation -->
    <Grid Grid.Column="0" Background="#f5f5f5">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0" Text="Navigation" FontWeight="Bold" Margin="10"/>

        <TreeView Grid.Row="1" BorderThickness="0" Background="Transparent">
            <TreeViewItem Header="Documents" IsExpanded="True">
                <TreeViewItem Header="Rapports"/>
                <TreeViewItem Header="Factures"/>
                <TreeViewItem Header="Contrats"/>
            </TreeViewItem>
            <TreeViewItem Header="Images">
                <TreeViewItem Header="Photographies"/>
                <TreeViewItem Header="Illustrations"/>
            </TreeViewItem>
            <TreeViewItem Header="Musique"/>
            <TreeViewItem Header="Vidéos"/>
        </TreeView>
    </Grid>

    <!-- GridSplitter pour redimensionner manuellement -->
    <GridSplitter Grid.Column="1" Width="5" HorizontalAlignment="Center"
                  VerticalAlignment="Stretch" Background="#bdc3c7"/>

    <!-- Panneau de contenu -->
    <Grid Grid.Column="2">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- Barre d'outils -->
        <ToolBarTray Grid.Row="0">
            <ToolBar>
                <Button Content="Nouveau" Padding="5,2"/>
                <Button Content="Ouvrir" Padding="5,2"/>
                <Button Content="Enregistrer" Padding="5,2"/>
                <Separator/>
                <Button Content="Couper" Padding="5,2"/>
                <Button Content="Copier" Padding="5,2"/>
                <Button Content="Coller" Padding="5,2"/>
            </ToolBar>
        </ToolBarTray>

        <!-- En-tête du document -->
        <Grid Grid.Row="1" Margin="10">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>

            <TextBlock Grid.Row="0" Grid.Column="0" Text="Titre:" Margin="0,0,10,5"/>
            <TextBox Grid.Row="0" Grid.Column="1" Text="Rapport mensuel" Margin="0,0,0,5"/>

            <TextBlock Grid.Row="1" Grid.Column="0" Text="Auteur:" Margin="0,0,10,0"/>
            <TextBox Grid.Row="1" Grid.Column="1" Text="Jean Dupont"/>
        </Grid>

        <!-- Contenu du document -->
        <TabControl Grid.Row="2" Margin="10,0,10,10">
            <TabItem Header="Édition">
                <DockPanel>
                    <StackPanel DockPanel.Dock="Top" Orientation="Horizontal" Background="#f5f5f5">
                        <ComboBox Width="150" Margin="5" SelectedIndex="0">
                            <ComboBoxItem Content="Arial"/>
                            <ComboBoxItem Content="Times New Roman"/>
                            <ComboBoxItem Content="Calibri"/>
                        </ComboBox>

                        <ComboBox Width="50" Margin="0,5,5,5" SelectedIndex="2">
                            <ComboBoxItem Content="10"/>
                            <ComboBoxItem Content="11"/>
                            <ComboBoxItem Content="12"/>
                            <ComboBoxItem Content="14"/>
                            <ComboBoxItem Content="16"/>
                        </ComboBox>

                        <Button Content="G" FontWeight="Bold" Width="30" Margin="0,5,2,5"/>
                        <Button Content="I" FontStyle="Italic" Width="30" Margin="0,5,2,5"/>
                        <Button Content="S" TextDecorations="Underline" Width="30" Margin="0,5,5,5"/>
                    </StackPanel>

                    <TextBox AcceptsReturn="True" TextWrapping="Wrap"
                             VerticalScrollBarVisibility="Auto"
                             Text="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed non risus. Suspendisse lectus tortor, dignissim sit amet, adipiscing nec, ultricies sed, dolor.&#x0A;&#x0A;Cras elementum ultrices diam. Maecenas ligula massa, varius a, semper congue, euismod non, mi. Proin porttitor, orci nec nonummy molestie, enim est eleifend mi, non fermentum diam nisl sit amet erat.&#x0A;&#x0A;Duis semper. Duis arcu massa, scelerisque vitae, consequat in, pretium a, enim. Pellentesque congue. Ut in risus volutpat libero pharetra tempor. Cras vestibulum bibendum augue."
                             Padding="5"/>
                </DockPanel>
            </TabItem>

            <TabItem Header="Aperçu">
                <ScrollViewer VerticalScrollBarVisibility="Auto">
                    <StackPanel Background="White" Margin="20" MaxWidth="800">
                        <TextBlock Text="RAPPORT MENSUEL" FontSize="24"
                                   FontWeight="Bold" HorizontalAlignment="Center"
                                   Margin="0,20,0,10"/>

                        <TextBlock Text="Jean Dupont" FontSize="16"
                                   HorizontalAlignment="Center" Margin="0,0,0,30"/>

                        <TextBlock TextWrapping="Wrap" Margin="0,0,0,10">
                            Lorem ipsum dolor sit amet, consectetur adipiscing elit.
                            Sed non risus. Suspendisse lectus tortor, dignissim sit amet,
                            adipiscing nec, ultricies sed, dolor.
                        </TextBlock>

                        <TextBlock TextWrapping="Wrap" Margin="0,0,0,10">
                            Cras elementum ultrices diam. Maecenas ligula massa, varius a,
                            semper congue, euismod non, mi. Proin porttitor, orci nec nonummy
                            molestie, enim est eleifend mi, non fermentum diam nisl sit amet erat.
                        </TextBlock>

                        <TextBlock TextWrapping="Wrap" Margin="0,0,0,10">
                            Duis semper. Duis arcu massa, scelerisque vitae, consequat in,
                            pretium a, enim. Pellentesque congue. Ut in risus volutpat libero
                            pharetra tempor. Cras vestibulum bibendum augue.
                        </TextBlock>
                    </StackPanel>
                </ScrollViewer>
            </TabItem>
        </TabControl>

        <!-- Barre d'actions -->
        <DockPanel Grid.Row="3" LastChildFill="False" Background="#f5f5f5">
            <Button DockPanel.Dock="Right" Content="Publier"
                    Background="#2ecc71" Foreground="White"
                    Padding="15,5" Margin="10"/>

            <Button DockPanel.Dock="Right" Content="Enregistrer"
                    Padding="15,5" Margin="0,10,10,10"/>

            <Button DockPanel.Dock="Right" Content="Annuler"
                    Padding="15,5" Margin="0,10,10,10"/>
        </DockPanel>
    </Grid>
</Grid>
```

#### Conception adaptative et réactive

```xml
<!-- Interface réactive avec panels combinés -->
<Grid Margin="10">
    <!-- Ajout d'un ViewBox pour adapter le contenu à la taille de la fenêtre -->
    <Viewbox Stretch="Uniform" StretchDirection="DownOnly">
        <Grid Width="1024" Height="768">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>

            <!-- En-tête -->
            <Border Grid.Row="0" Background="#3498db" Padding="20,10">
                <DockPanel LastChildFill="True">
                    <StackPanel DockPanel.Dock="Right" Orientation="Horizontal">
                        <Button Content="Connexion" Background="Transparent"
                                Foreground="White" BorderThickness="0" Margin="5,0"/>
                        <Button Content="Inscription" Background="White"
                                Foreground="#3498db" BorderThickness="0"
                                Padding="10,5" Margin="5,0"/>
                    </StackPanel>

                    <TextBlock Text="MonApplication" FontSize="22"
                               Foreground="White" VerticalAlignment="Center"/>
                </DockPanel>
            </Border>

            <!-- Contenu principal -->
            <Grid Grid.Row="1">
                <!-- Contenu dynamique qui s'adapte en fonction de la largeur -->
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="400"/>
                </Grid.ColumnDefinitions>

                <!-- Zone principale -->
                <ScrollViewer Grid.Column="0" VerticalScrollBarVisibility="Auto">
                    <WrapPanel Orientation="Horizontal" Margin="20">
                        <!-- Cartes de contenu -->
                        <Border Width="300" Height="200" Margin="0,0,20,20"
                                BorderBrush="#e0e0e0" BorderThickness="1"
                                CornerRadius="5" Background="White">
                            <DockPanel Margin="15">
                                <TextBlock DockPanel.Dock="Top" Text="Carte 1"
                                           FontSize="18" FontWeight="Bold" Margin="0,0,0,10"/>
                                <Button DockPanel.Dock="Bottom" Content="Voir plus"
                                        HorizontalAlignment="Right" Padding="10,5"
                                        Margin="0,10,0,0"/>
                                <TextBlock TextWrapping="Wrap">
                                    Lorem ipsum dolor sit amet, consectetur adipiscing elit.
                                    Ut efficitur fringilla odio in condimentum.
                                </TextBlock>
                            </DockPanel>
                        </Border>

                        <Border Width="300" Height="200" Margin="0,0,20,20"
                                BorderBrush="#e0e0e0" BorderThickness="1"
                                CornerRadius="5" Background="White">
                            <DockPanel Margin="15">
                                <TextBlock DockPanel.Dock="Top" Text="Carte 2"
                                           FontSize="18" FontWeight="Bold" Margin="0,0,0,10"/>
                                <Button DockPanel.Dock="Bottom" Content="Voir plus"
                                        HorizontalAlignment="Right" Padding="10,5"
                                        Margin="0,10,0,0"/>
                                <TextBlock TextWrapping="Wrap">
                                    Suspendisse potenti. Etiam ac sapien vel libero
                                    accumsan commodo. Vivamus condimentum.
                                </TextBlock>
                            </DockPanel>
                        </Border>

                        <Border Width="300" Height="200" Margin="0,0,20,20"
                                BorderBrush="#e0e0e0" BorderThickness="1"
                                CornerRadius="5" Background="White">
                            <DockPanel Margin="15">
                                <TextBlock DockPanel.Dock="Top" Text="Carte 3"
                                           FontSize="18" FontWeight="Bold" Margin="0,0,0,10"/>
                                <Button DockPanel.Dock="Bottom" Content="Voir plus"
                                        HorizontalAlignment="Right" Padding="10,5"
                                        Margin="0,10,0,0"/>
                                <TextBlock TextWrapping="Wrap">
                                    Sed eu sapien ut nunc iaculis luctus. Donec accumsan libero
                                    eget magna aliquet, id convallis.
                                </TextBlock>
                            </DockPanel>
                        </Border>

                        <Border Width="300" Height="200" Margin="0,0,20,20"
                                BorderBrush="#e0e0e0" BorderThickness="1"
                                CornerRadius="5" Background="White">
                            <DockPanel Margin="15">
                                <TextBlock DockPanel.Dock="Top" Text="Carte 4"
                                           FontSize="18" FontWeight="Bold" Margin="0,0,0,10"/>
                                <Button DockPanel.Dock="Bottom" Content="Voir plus"
                                        HorizontalAlignment="Right" Padding="10,5"
                                        Margin="0,10,0,0"/>
                                <TextBlock TextWrapping="Wrap">
                                    Morbi ornare porttitor ex, eget pellentesque nunc tempor vel.
                                    Ut nec tortor maximus, condimentum.
                                </TextBlock>
                            </DockPanel>
                        </Border>
                    </WrapPanel>
                </ScrollViewer>

                <!-- Barre latérale -->
                <Border Grid.Column="1" BorderBrush="#e0e0e0" BorderThickness="1,0,0,0">
                    <DockPanel>
                        <Border DockPanel.Dock="Top" Background="#f8f9fa"
                                BorderBrush="#e0e0e0" BorderThickness="0,0,0,1"
                                Padding="15,10">
                            <TextBlock Text="Informations" FontWeight="Bold"/>
                        </Border>

                        <StackPanel Margin="15">
                            <TextBlock Text="Dernières mises à jour"
                                       FontWeight="Bold" Margin="0,0,0,10"/>

                            <Border BorderBrush="#e0e0e0" BorderThickness="0,0,0,1" Padding="0,0,0,10" Margin="0,0,0,10">
                                <StackPanel>
                                    <TextBlock Text="Titre de la mise à jour 1" FontWeight="Bold"/>
                                    <TextBlock Text="10/06/2023" Foreground="#6c757d" Margin="0,2,0,5"/>
                                    <TextBlock TextWrapping="Wrap">
                                        Brève description de la mise à jour
                                        et des nouvelles fonctionnalités ajoutées.
                                    </TextBlock>
                                </StackPanel>
                            </Border>

                            <Border BorderBrush="#e0e0e0" BorderThickness="0,0,0,1" Padding="0,0,0,10" Margin="0,0,0,10">
                                <StackPanel>
                                    <TextBlock Text="Titre de la mise à jour 2" FontWeight="Bold"/>
                                    <TextBlock Text="05/06/2023" Foreground="#6c757d" Margin="0,2,0,5"/>
                                    <TextBlock TextWrapping="Wrap">
                                        Autre mise à jour avec description des changements
                                        et améliorations du système.
                                    </TextBlock>
                                </StackPanel>
                            </Border>

                            <Border Padding="0,0,0,10">
                                <StackPanel>
                                    <TextBlock Text="Titre de la mise à jour 3" FontWeight="Bold"/>
                                    <TextBlock Text="01/06/2023" Foreground="#6c757d" Margin="0,2,0,5"/>
                                    <TextBlock TextWrapping="Wrap">
                                        Troisième exemple de mise à jour avec
                                        des corrections de bugs importants.
                                    </TextBlock>
                                </StackPanel>
                            </Border>

                            <Button Content="Toutes les mises à jour"
                                    HorizontalAlignment="Right"
                                    Padding="10,5" Margin="0,10,0,0"/>

                            <TextBlock Text="Statistiques"
                                       FontWeight="Bold" Margin="0,20,0,10"/>

                            <Grid>
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="*"/>
                                    <ColumnDefinition Width="*"/>
                                </Grid.ColumnDefinitions>
                                <Grid.RowDefinitions>
                                    <RowDefinition Height="Auto"/>
                                    <RowDefinition Height="Auto"/>
                                </Grid.RowDefinitions>

                                <Border Grid.Column="0" Grid.Row="0"
                                        Background="#e8f4f8" Padding="10"
                                        Margin="0,0,5,5" CornerRadius="5">
                                    <StackPanel>
                                        <TextBlock Text="Utilisateurs" FontWeight="Bold"/>
                                        <TextBlock Text="12,450" FontSize="18"/>
                                    </StackPanel>
                                </Border>

                                <Border Grid.Column="1" Grid.Row="0"
                                        Background="#e8f8e8" Padding="10"
                                        Margin="5,0,0,5" CornerRadius="5">
                                    <StackPanel>
                                        <TextBlock Text="Produits" FontWeight="Bold"/>
                                        <TextBlock Text="534" FontSize="18"/>
                                    </StackPanel>
                                </Border>

                                <Border Grid.Column="0" Grid.Row="1"
                                        Background="#f8e8e8" Padding="10"
                                        Margin="0,5,5,0" CornerRadius="5">
                                    <StackPanel>
                                        <TextBlock Text="Commandes" FontWeight="Bold"/>
                                        <TextBlock Text="2,840" FontSize="18"/>
                                    </StackPanel>
                                </Border>

                                <Border Grid.Column="1" Grid.Row="1"
                                        Background="#f8f8e8" Padding="10"
                                        Margin="5,5,0,0" CornerRadius="5">
                                    <StackPanel>
                                        <TextBlock Text="Revenu" FontWeight="Bold"/>
                                        <TextBlock Text="98,540 €" FontSize="18"/>
                                    </StackPanel>
                                </Border>
                            </Grid>
                        </StackPanel>
                    </DockPanel>
                </Border>
            </Grid>

            <!-- Pied de page -->
            <Border Grid.Row="2" Background="#f8f9fa"
                    BorderBrush="#e0e0e0" BorderThickness="0,1,0,0"
                    Padding="20,15">
                <DockPanel>
                    <TextBlock DockPanel.Dock="Right" Text="© 2023 MonApplication"/>

                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="Mentions légales" Margin="0,0,20,0"/>
                        <TextBlock Text="Confidentialité" Margin="0,0,20,0"/>
                        <TextBlock Text="Contact" Margin="0,0,20,0"/>
                    </StackPanel>
                </DockPanel>
            </Border>
        </Grid>
    </Viewbox>
</Grid>
```


### Bonnes pratiques pour les layouts complexes

1. **Structure de base claire**
   - Utilisez un `Grid` comme conteneur racine pour les interfaces complexes
   - Divisez l'interface en zones fonctionnelles distinctes (en-tête, contenu, pied de page, etc.)
   - Utilisez des commentaires pour documenter la structure XAML

2. **Optimisation des performances**
   - Évitez les layouts trop profondément imbriqués (plus de 10 niveaux)
   - Utilisez `SharedSizeGroup` pour aligner des éléments dans différents panels
   - Préférez `Canvas` pour les éléments très nombreux ou qui changent fréquemment de position
   - Utilisez `VirtualizingStackPanel` pour les listes longues

3. **Flexibilité et maintenabilité**
   - Définissez des styles et templates réutilisables
   - Utilisez des UserControls pour encapsuler des composants d'interface complexes
   - Concevez pour différentes tailles d'écran en utilisant des dimensions proportionnelles
   - Testez vos interfaces avec différentes tailles de fenêtre

4. **Techniques avancées**
   - Utilisez les `GridSplitter` pour permettre aux utilisateurs de redimensionner les sections
   - Implémentez des techniques de Layout Virtualization pour les grandes collections
   - Combinez `ScrollViewer` avec les panels appropriés pour gérer le débordement
   - Utilisez `Viewbox` pour adapter proportionnellement le contenu

```xml
<!-- Exemple de bonnes pratiques pour un layout complexe -->
<Grid>
    <!-- Définition des lignes principales -->
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/> <!-- En-tête -->
        <RowDefinition Height="*"/>    <!-- Contenu principal -->
        <RowDefinition Height="Auto"/> <!-- Pied de page -->
    </Grid.RowDefinitions>

    <!-- En-tête de l'application -->
    <Border Grid.Row="0" x:Name="HeaderSection" Background="{StaticResource HeaderBackground}">
        <!-- Contenu de l'en-tête... -->
    </Border>

    <!-- Contenu principal avec Grid.IsSharedSizeScope pour aligner des éléments -->
    <Grid Grid.Row="1" Grid.IsSharedSizeScope="True">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" SharedSizeGroup="NavigationWidth"/>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Navigation latérale -->
        <UserControl Grid.Column="0" x:Name="NavigationPanel">
            <!-- Contenu encapsulé dans un UserControl -->
        </UserControl>

        <!-- Splitter pour redimensionner -->
        <GridSplitter Grid.Column="1" Width="5" ResizeBehavior="PreviousAndNext"
                      HorizontalAlignment="Stretch" VerticalAlignment="Stretch"/>

        <!-- Zone de contenu principal -->
        <Border Grid.Column="2" x:Name="ContentSection">
            <!-- Utilisez ScrollViewer pour un défilement fluide -->
            <ScrollViewer VerticalScrollBarVisibility="Auto">
                <!-- Contenu dynamique -->
            </ScrollViewer>
        </Border>
    </Grid>

    <!-- Pied de page -->
    <Border Grid.Row="2" x:Name="FooterSection">
        <!-- Contenu du pied de page... -->
    </Border>
</Grid>
```


## Résumé

Cette section a couvert les différents types de panels disponibles dans WPF et leur utilisation pour créer des interfaces utilisateur bien structurées :

- **Grid** : Le panel le plus puissant, permettant d'organiser les éléments en rangées et colonnes avec un contrôle précis sur l'emplacement.
- **StackPanel** : Arrange les éléments dans une seule ligne ou colonne, idéal pour les groupes d'éléments séquentiels.
- **WrapPanel** : Place les éléments séquentiellement et passe à la ligne quand il n'y a plus d'espace, parfait pour les collections d'éléments de taille similaire.
- **DockPanel** : Permet d'ancrer les éléments sur les bords ou au centre, idéal pour les layouts d'application standard.
- **Canvas** : Offre un positionnement absolu avec des coordonnées x,y, utile pour les applications de dessin ou les interfaces très personnalisées.

Nous avons également vu comment combiner ces panels pour créer des interfaces complexes et adaptatives qui peuvent s'ajuster à différentes tailles d'écran et conditions d'affichage.

Les bonnes pratiques pour la conception de layouts comprennent l'utilisation de dimensions proportionnelles, la virtualisation pour les grandes collections, l'encapsulation des composants complexes dans des UserControls, et l'utilisation judicieuse de l'imbrication de panels pour maximiser la flexibilité tout en maintenant de bonnes performances.

La maîtrise de ces différents panels et de leur combinaison est essentielle pour créer des interfaces WPF robustes, flexibles et attrayantes qui offrent une excellente expérience utilisateur.

⏭️
