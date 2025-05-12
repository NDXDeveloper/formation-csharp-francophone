# 7.5. Mise en page et design

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Une interface utilisateur bien conçue est essentielle pour offrir une expérience utilisateur optimale. Windows Forms propose plusieurs mécanismes pour organiser et agencer vos contrôles de manière efficace et esthétique.

## 7.5.1. Contrôles de conteneur (Panel, GroupBox)

Les contrôles de conteneur permettent de regrouper logiquement d'autres contrôles et facilitent la gestion de leur positionnement.

### Panel

Le Panel est un conteneur simple qui peut être utilisé pour regrouper des contrôles connexes. Il peut également servir de surface de défilement lorsque son contenu est plus grand que sa taille visible.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerPanels()
{
    // Création d'un panel standard
    Panel panelFormulaire = new Panel
    {
        Location = new Point(20, 20),
        Size = new Size(350, 200),
        BorderStyle = BorderStyle.FixedSingle,
        AutoScroll = true // Active le défilement automatique
    };

    // Ajout de contrôles au panel
    Label labelNom = new Label
    {
        Location = new Point(10, 15),
        Size = new Size(100, 23),
        Text = "Nom :"
    };

    TextBox textBoxNom = new TextBox
    {
        Location = new Point(120, 12),
        Size = new Size(200, 23)
    };

    Label labelAdresse = new Label
    {
        Location = new Point(10, 45),
        Size = new Size(100, 23),
        Text = "Adresse :"
    };

    TextBox textBoxAdresse = new TextBox
    {
        Location = new Point(120, 42),
        Size = new Size(200, 23)
    };

    Label labelEmail = new Label
    {
        Location = new Point(10, 75),
        Size = new Size(100, 23),
        Text = "Email :"
    };

    TextBox textBoxEmail = new TextBox
    {
        Location = new Point(120, 72),
        Size = new Size(200, 23)
    };

    Button buttonEnregistrer = new Button
    {
        Location = new Point(220, 120),
        Size = new Size(100, 30),
        Text = "Enregistrer"
    };

    // Ajout des contrôles au panel
    panelFormulaire.Controls.Add(labelNom);
    panelFormulaire.Controls.Add(textBoxNom);
    panelFormulaire.Controls.Add(labelAdresse);
    panelFormulaire.Controls.Add(textBoxAdresse);
    panelFormulaire.Controls.Add(labelEmail);
    panelFormulaire.Controls.Add(textBoxEmail);
    panelFormulaire.Controls.Add(buttonEnregistrer);

    // Création d'un panel avec défilement
    Panel panelListe = new Panel
    {
        Location = new Point(20, 240),
        Size = new Size(350, 150),
        BorderStyle = BorderStyle.FixedSingle,
        AutoScroll = true
    };

    // Ajout de nombreux contrôles pour tester le défilement
    for (int i = 0; i < 20; i++)
    {
        CheckBox checkBox = new CheckBox
        {
            Location = new Point(10, i * 25 + 10),
            Size = new Size(300, 20),
            Text = $"Option {i + 1}"
        };

        panelListe.Controls.Add(checkBox);
    }

    // Ajout des panels au formulaire
    this.Controls.Add(panelFormulaire);
    this.Controls.Add(panelListe);
}
```


### GroupBox

Le GroupBox est similaire au Panel mais ajoute une bordure avec un titre, ce qui le rend parfait pour regrouper des contrôles liés fonctionnellement.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerGroupBoxes()
{
    // Création d'un GroupBox pour les informations personnelles
    GroupBox groupBoxInfosPerso = new GroupBox
    {
        Location = new Point(20, 20),
        Size = new Size(350, 180),
        Text = "Informations personnelles"
    };

    // Ajout de contrôles au GroupBox
    Label labelNom = new Label
    {
        Location = new Point(15, 25),
        Size = new Size(100, 23),
        Text = "Nom :"
    };

    TextBox textBoxNom = new TextBox
    {
        Location = new Point(125, 22),
        Size = new Size(200, 23)
    };

    Label labelPrenom = new Label
    {
        Location = new Point(15, 55),
        Size = new Size(100, 23),
        Text = "Prénom :"
    };

    TextBox textBoxPrenom = new TextBox
    {
        Location = new Point(125, 52),
        Size = new Size(200, 23)
    };

    Label labelDateNaissance = new Label
    {
        Location = new Point(15, 85),
        Size = new Size(100, 23),
        Text = "Date de naissance :"
    };

    DateTimePicker dateTimePickerNaissance = new DateTimePicker
    {
        Location = new Point(125, 82),
        Size = new Size(200, 23),
        Format = DateTimePickerFormat.Short
    };

    Label labelTelephone = new Label
    {
        Location = new Point(15, 115),
        Size = new Size(100, 23),
        Text = "Téléphone :"
    };

    MaskedTextBox maskedTextBoxTelephone = new MaskedTextBox
    {
        Location = new Point(125, 112),
        Size = new Size(200, 23),
        Mask = "00 00 00 00 00"
    };

    groupBoxInfosPerso.Controls.Add(labelNom);
    groupBoxInfosPerso.Controls.Add(textBoxNom);
    groupBoxInfosPerso.Controls.Add(labelPrenom);
    groupBoxInfosPerso.Controls.Add(textBoxPrenom);
    groupBoxInfosPerso.Controls.Add(labelDateNaissance);
    groupBoxInfosPerso.Controls.Add(dateTimePickerNaissance);
    groupBoxInfosPerso.Controls.Add(labelTelephone);
    groupBoxInfosPerso.Controls.Add(maskedTextBoxTelephone);

    // Création d'un GroupBox pour les préférences
    GroupBox groupBoxPreferences = new GroupBox
    {
        Location = new Point(20, 210),
        Size = new Size(350, 150),
        Text = "Préférences de contact"
    };

    // Création des boutons radio
    RadioButton radioButtonEmail = new RadioButton
    {
        Location = new Point(15, 25),
        Size = new Size(150, 20),
        Text = "Email",
        Checked = true // Option sélectionnée par défaut
    };

    RadioButton radioButtonTelephone = new RadioButton
    {
        Location = new Point(15, 50),
        Size = new Size(150, 20),
        Text = "Téléphone"
    };

    RadioButton radioButtonCourrier = new RadioButton
    {
        Location = new Point(15, 75),
        Size = new Size(150, 20),
        Text = "Courrier postal"
    };

    // Création des cases à cocher
    CheckBox checkBoxNewsletter = new CheckBox
    {
        Location = new Point(180, 25),
        Size = new Size(150, 20),
        Text = "Newsletter"
    };

    CheckBox checkBoxOffres = new CheckBox
    {
        Location = new Point(180, 50),
        Size = new Size(150, 20),
        Text = "Offres spéciales"
    };

    CheckBox checkBoxSondages = new CheckBox
    {
        Location = new Point(180, 75),
        Size = new Size(150, 20),
        Text = "Sondages"
    };

    groupBoxPreferences.Controls.Add(radioButtonEmail);
    groupBoxPreferences.Controls.Add(radioButtonTelephone);
    groupBoxPreferences.Controls.Add(radioButtonCourrier);
    groupBoxPreferences.Controls.Add(checkBoxNewsletter);
    groupBoxPreferences.Controls.Add(checkBoxOffres);
    groupBoxPreferences.Controls.Add(checkBoxSondages);

    // Ajout des GroupBox au formulaire
    this.Controls.Add(groupBoxInfosPerso);
    this.Controls.Add(groupBoxPreferences);
}
```


### Avantages et cas d'utilisation

#### Panel
- **Avantages** : Léger, flexible, peut avoir un défilement, peut avoir une bordure simple
- **Cas d'utilisation** : Regrouper des contrôles connexes, créer des zones défilables, diviser une interface en sections

#### GroupBox
- **Avantages** : Fournit un titre descriptif, visuel professionnel, organisation claire
- **Cas d'utilisation** : Regrouper des contrôles liés fonctionnellement (ex: options de configuration, données d'un formulaire)

## 7.5.2. Layout managers (TableLayoutPanel, FlowLayoutPanel)

Les gestionnaires de mise en page simplifient considérablement l'organisation des contrôles et permettent de créer des interfaces qui s'adaptent dynamiquement aux changements de taille.

### TableLayoutPanel

Le TableLayoutPanel organise les contrôles dans une grille de lignes et de colonnes, ce qui facilite la création de formulaires complexes avec un alignement précis.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerTableLayoutPanel()
{
    // Création d'un TableLayoutPanel
    TableLayoutPanel tableLayoutPanel = new TableLayoutPanel
    {
        Location = new Point(20, 20),
        Size = new Size(500, 300),
        ColumnCount = 3,
        RowCount = 4,
        Dock = DockStyle.Fill,
        Padding = new Padding(5),
        CellBorderStyle = TableLayoutPanelCellBorderStyle.Single
    };

    // Configuration des colonnes
    tableLayoutPanel.ColumnStyles.Clear();
    tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 30F));  // Colonne 1 : 30%
    tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 40F));  // Colonne 2 : 40%
    tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 30F));  // Colonne 3 : 30%

    // Configuration des lignes
    tableLayoutPanel.RowStyles.Clear();
    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Absolute, 40F));       // Ligne 1 : hauteur fixe
    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 33.33F));     // Ligne 2 : proportionnelle
    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 33.33F));     // Ligne 3 : proportionnelle
    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 33.34F));     // Ligne 4 : proportionnelle

    // Ajout des contrôles d'en-tête (première ligne)
    Label labelEntete1 = new Label
    {
        Text = "Produit",
        TextAlign = ContentAlignment.MiddleCenter,
        Dock = DockStyle.Fill,
        Font = new Font("Segoe UI", 9F, FontStyle.Bold)
    };

    Label labelEntete2 = new Label
    {
        Text = "Description",
        TextAlign = ContentAlignment.MiddleCenter,
        Dock = DockStyle.Fill,
        Font = new Font("Segoe UI", 9F, FontStyle.Bold)
    };

    Label labelEntete3 = new Label
    {
        Text = "Prix",
        TextAlign = ContentAlignment.MiddleCenter,
        Dock = DockStyle.Fill,
        Font = new Font("Segoe UI", 9F, FontStyle.Bold)
    };

    tableLayoutPanel.Controls.Add(labelEntete1, 0, 0);
    tableLayoutPanel.Controls.Add(labelEntete2, 1, 0);
    tableLayoutPanel.Controls.Add(labelEntete3, 2, 0);

    // Ajout de produits (lignes suivantes)
    string[] produits = { "Ordinateur portable", "Smartphone", "Tablette" };
    string[] descriptions = {
        "Ordinateur portable haute performance avec processeur i7",
        "Smartphone avec écran 6.7 pouces, 128GB",
        "Tablette 10 pouces, idéale pour la lecture et la navigation web"
    };
    string[] prix = { "999,99 €", "699,99 €", "349,99 €" };

    for (int i = 0; i < produits.Length; i++)
    {
        // Colonne 1 : Produit
        Label labelProduit = new Label
        {
            Text = produits[i],
            TextAlign = ContentAlignment.MiddleLeft,
            Dock = DockStyle.Fill,
            Margin = new Padding(3)
        };

        // Colonne 2 : Description
        TextBox textBoxDescription = new TextBox
        {
            Text = descriptions[i],
            Multiline = true,
            ReadOnly = true,
            Dock = DockStyle.Fill,
            Margin = new Padding(3)
        };

        // Colonne 3 : Prix avec bouton
        Panel panelPrix = new Panel
        {
            Dock = DockStyle.Fill,
            Margin = new Padding(0)
        };

        Label labelPrix = new Label
        {
            Text = prix[i],
            TextAlign = ContentAlignment.MiddleCenter,
            Dock = DockStyle.Top,
            Height = 30
        };

        Button buttonAjouter = new Button
        {
            Text = "Ajouter au panier",
            Dock = DockStyle.Bottom,
            Height = 30
        };

        panelPrix.Controls.Add(labelPrix);
        panelPrix.Controls.Add(buttonAjouter);

        // Ajout au TableLayoutPanel
        tableLayoutPanel.Controls.Add(labelProduit, 0, i + 1);
        tableLayoutPanel.Controls.Add(textBoxDescription, 1, i + 1);
        tableLayoutPanel.Controls.Add(panelPrix, 2, i + 1);
    }

    // Ajout du TableLayoutPanel au formulaire
    this.Controls.Add(tableLayoutPanel);
}
```


### FlowLayoutPanel

Le FlowLayoutPanel dispose les contrôles de manière séquentielle, en passant à la ligne lorsque nécessaire. C'est idéal pour les interfaces où l'ordre des contrôles est important mais leur position exacte n'est pas critique.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerFlowLayoutPanel()
{
    // Création d'un FlowLayoutPanel horizontal
    FlowLayoutPanel flowLayoutHorizontal = new FlowLayoutPanel
    {
        Location = new Point(20, 20),
        Size = new Size(500, 100),
        FlowDirection = FlowDirection.LeftToRight,
        WrapContents = true,
        BorderStyle = BorderStyle.FixedSingle,
        Padding = new Padding(5),
        AutoScroll = true
    };

    // Ajout de boutons au FlowLayoutPanel horizontal
    for (int i = 1; i <= 15; i++)
    {
        Button button = new Button
        {
            Text = $"Bouton {i}",
            Size = new Size(90, 35),
            Margin = new Padding(5)
        };

        flowLayoutHorizontal.Controls.Add(button);
    }

    // Création d'un FlowLayoutPanel vertical
    FlowLayoutPanel flowLayoutVertical = new FlowLayoutPanel
    {
        Location = new Point(20, 140),
        Size = new Size(200, 300),
        FlowDirection = FlowDirection.TopDown,
        WrapContents = false,
        BorderStyle = BorderStyle.FixedSingle,
        Padding = new Padding(5),
        AutoScroll = true
    };

    // Ajout de contrôles au FlowLayoutPanel vertical
    string[] options = {
        "Accueil", "Produits", "Services", "Clients",
        "Factures", "Rapports", "Paramètres", "Aide"
    };

    foreach (string option in options)
    {
        // Création d'un contrôle composé
        Panel panelOption = new Panel
        {
            Size = new Size(170, 40),
            Margin = new Padding(5)
        };

        Label labelOption = new Label
        {
            Text = option,
            Font = new Font("Segoe UI", 10F),
            Location = new Point(40, 10),
            AutoSize = true
        };

        PictureBox pictureBox = new PictureBox
        {
            Size = new Size(24, 24),
            Location = new Point(5, 8),
            BackColor = Color.LightGray
        };

        panelOption.Controls.Add(pictureBox);
        panelOption.Controls.Add(labelOption);

        // Ajout de l'effet de survol
        panelOption.MouseEnter += (sender, e) => {
            panelOption.BackColor = Color.LightBlue;
        };

        panelOption.MouseLeave += (sender, e) => {
            panelOption.BackColor = SystemColors.Control;
        };

        flowLayoutVertical.Controls.Add(panelOption);
    }

    // Création d'un FlowLayoutPanel pour un formulaire dynamique
    FlowLayoutPanel flowLayoutFormulaire = new FlowLayoutPanel
    {
        Location = new Point(240, 140),
        Size = new Size(280, 300),
        FlowDirection = FlowDirection.TopDown,
        WrapContents = false,
        BorderStyle = BorderStyle.FixedSingle,
        Padding = new Padding(10),
        AutoScroll = true
    };

    // Création d'une structure de formulaire dynamique
    string[] champs = { "Nom", "Prénom", "Email", "Téléphone", "Adresse", "Ville", "Code postal", "Pays" };

    foreach (string champ in champs)
    {
        // Création d'un label
        Label label = new Label
        {
            Text = champ + " :",
            Size = new Size(260, 20),
            Margin = new Padding(0, 5, 0, 0)
        };

        // Création d'un TextBox
        TextBox textBox = new TextBox
        {
            Size = new Size(260, 25),
            Margin = new Padding(0, 0, 0, 10)
        };

        flowLayoutFormulaire.Controls.Add(label);
        flowLayoutFormulaire.Controls.Add(textBox);
    }

    // Ajout d'un bouton de validation
    Button buttonValider = new Button
    {
        Text = "Valider",
        Size = new Size(100, 30),
        Margin = new Padding(80, 10, 0, 0)
    };

    flowLayoutFormulaire.Controls.Add(buttonValider);

    // Ajout des FlowLayoutPanels au formulaire
    this.Controls.Add(flowLayoutHorizontal);
    this.Controls.Add(flowLayoutVertical);
    this.Controls.Add(flowLayoutFormulaire);
}
```


### Avantages et cas d'utilisation

#### TableLayoutPanel
- **Avantages** : Alignement précis, redimensionnement proportionnel, gestion complète des cellules
- **Cas d'utilisation** : Formulaires complexes, tableaux de données, interfaces en grille

#### FlowLayoutPanel
- **Avantages** : Disposition automatique, gestion du débordement, réorganisation dynamique
- **Cas d'utilisation** : Barres d'outils, listes d'options, galeries d'images, interfaces flexibles

## 7.5.3. Ancrages et docking

Les ancrages et le docking permettent de définir comment les contrôles se comportent lorsque leur conteneur est redimensionné.

### Ancrages (Anchor)

L'ancrage définit les bords du contrôle qui restent à une distance fixe des bords correspondants de son conteneur.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void DémontrerAncrages()
{
    // Panel pour contenir les démos d'ancrage
    Panel panelDémo = new Panel
    {
        Location = new Point(20, 20),
        Size = new Size(600, 400),
        BorderStyle = BorderStyle.FixedSingle,
        BackColor = Color.LightGray
    };

    // Bouton ancré en haut à gauche (par défaut)
    Button buttonTopLeft = new Button
    {
        Text = "Haut Gauche\n(défaut)",
        Location = new Point(10, 10),
        Size = new Size(100, 50),
        BackColor = Color.LightBlue,
        // Anchor = AnchorStyles.Top | AnchorStyles.Left (valeur par défaut)
    };

    // Bouton ancré en haut à droite
    Button buttonTopRight = new Button
    {
        Text = "Haut Droite",
        Location = new Point(panelDémo.ClientSize.Width - 110, 10),
        Size = new Size(100, 50),
        BackColor = Color.LightPink,
        Anchor = AnchorStyles.Top | AnchorStyles.Right
    };

    // Bouton ancré en bas à gauche
    Button buttonBottomLeft = new Button
    {
        Text = "Bas Gauche",
        Location = new Point(10, panelDémo.ClientSize.Height - 60),
        Size = new Size(100, 50),
        BackColor = Color.LightGreen,
        Anchor = AnchorStyles.Bottom | AnchorStyles.Left
    };

    // Bouton ancré en bas à droite
    Button buttonBottomRight = new Button
    {
        Text = "Bas Droite",
        Location = new Point(panelDémo.ClientSize.Width - 110, panelDémo.ClientSize.Height - 60),
        Size = new Size(100, 50),
        BackColor = Color.LightYellow,
        Anchor = AnchorStyles.Bottom | AnchorStyles.Right
    };

    // TextBox ancré à gauche, droite et haut (s'étire horizontalement)
    TextBox textBoxTop = new TextBox
    {
        Text = "Ancré à gauche, droite et haut - s'étire horizontalement",
        Location = new Point(120, 25),
        Size = new Size(panelDémo.ClientSize.Width - 240, 23),
        Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
    };

    // ListBox ancré aux quatre côtés (s'étire dans toutes les directions)
    ListBox listBoxCenter = new ListBox
    {
        Location = new Point(120, 70),
        Size = new Size(panelDémo.ClientSize.Width - 240, panelDémo.ClientSize.Height - 140),
        Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right
    };

    // Remplir la ListBox avec des données de démonstration
    for (int i = 1; i <= 50; i++)
    {
        listBoxCenter.Items.Add($"Élément {i}");
    }

    // Button ancré à gauche, droite et bas
    Button buttonBottom = new Button
    {
        Text = "Ancré en bas (gauche et droite)",
        Location = new Point(120, panelDémo.ClientSize.Height - 60),
        Size = new Size(panelDémo.ClientSize.Width - 240, 50),
        Anchor = AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right
    };

    // Ajout des contrôles au panel
    panelDémo.Controls.Add(buttonTopLeft);
    panelDémo.Controls.Add(buttonTopRight);
    panelDémo.Controls.Add(buttonBottomLeft);
    panelDémo.Controls.Add(buttonBottomRight);
    panelDémo.Controls.Add(textBoxTop);
    panelDémo.Controls.Add(listBoxCenter);
    panelDémo.Controls.Add(buttonBottom);

    // Label explicatif
    Label labelExplication = new Label
    {
        Text = "Redimensionnez la fenêtre pour voir comment les contrôles s'adaptent selon leur ancrage",
        Location = new Point(20, 430),
        Size = new Size(600, 20),
        Font = new Font("Segoe UI", 9F, FontStyle.Italic)
    };

    // Ajout du panel et du label au formulaire
    this.Controls.Add(panelDémo);
    this.Controls.Add(labelExplication);
}
```


### Docking (Dock)

Le docking permet d'attacher un contrôle à un ou plusieurs bords de son conteneur, et éventuellement de le faire remplir tout l'espace disponible.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void DémontrerDocking()
{
    // Panel pour contenir les démos de docking
    Panel panelDémo = new Panel
    {
        Location = new Point(20, 20),
        Size = new Size(600, 400),
        BorderStyle = BorderStyle.FixedSingle,
        BackColor = Color.LightGray
    };

    // Barre d'outils (dock en haut)
    ToolStrip toolStrip = new ToolStrip
    {
        Dock = DockStyle.Top,
        BackColor = Color.LightSteelBlue,
        RenderMode = ToolStripRenderMode.System
    };

    // Ajout de boutons à la barre d'outils
    toolStrip.Items.Add(new ToolStripButton("Nouveau", null, (s, e) => MessageBox.Show("Nouveau")));
    toolStrip.Items.Add(new ToolStripButton("Ouvrir", null, (s, e) => MessageBox.Show("Ouvrir")));
    toolStrip.Items.Add(new ToolStripButton("Enregistrer", null, (s, e) => MessageBox.Show("Enregistrer")));
    toolStrip.Items.Add(new ToolStripSeparator());
    toolStrip.Items.Add(new ToolStripButton("Couper", null, (s, e) => MessageBox.Show("Couper")));
    toolStrip.Items.Add(new ToolStripButton("Copier", null, (s, e) => MessageBox.Show("Copier")));
    toolStrip.Items.Add(new ToolStripButton("Coller", null, (s, e) => MessageBox.Show("Coller")));

    // Barre d'état (dock en bas)
    StatusStrip statusStrip = new StatusStrip
    {
        Dock = DockStyle.Bottom,
        BackColor = Color.LightSteelBlue
    };

    // Ajout d'éléments à la barre d'état
    statusStrip.Items.Add("Prêt");
    statusStrip.Items.Add(new ToolStripStatusLabel() { Spring = true, Alignment = ToolStripItemAlignment.Right }); // Spacer
    statusStrip.Items.Add(DateTime.Now.ToShortDateString());

    // Panel de navigation (dock à gauche)
    Panel panelNavigation = new Panel
    {
        Dock = DockStyle.Left,
        Width = 150,
        BackColor = Color.Lavender
    };

    // Ajout de contrôles au panel de navigation
    ListBox listBoxNavigation = new ListBox
    {
        Dock = DockStyle.Fill,
        BorderStyle = BorderStyle.None
    };

    listBoxNavigation.Items.AddRange(new string[] {
        "Accueil", "Produits", "Services", "Clients",
        "Factures", "Rapports", "Paramètres", "Aide"
    });

    panelNavigation.Controls.Add(listBoxNavigation);

    // Panel de propriétés (dock à droite)
    Panel panelPropriétés = new Panel
    {
        Dock = DockStyle.Right,
        Width = 200,
        BackColor = Color.Linen
    };

    // Titre du panel des propriétés
    Label labelPropriétés = new Label
    {
        Text = "Propriétés",
        Dock = DockStyle.Top,
        Height = 30,
        TextAlign = ContentAlignment.MiddleCenter,
        Font = new Font("Segoe UI", 9F, FontStyle.Bold),
        BackColor = Color.AntiqueWhite
    };

    // Contenu du panel des propriétés
    PropertyGrid propertyGrid = new PropertyGrid
    {
        Dock = DockStyle.Fill,
        HelpVisible = false,
        ToolbarVisible = false
    };

    panelPropriétés.Controls.Add(propertyGrid);
    panelPropriétés.Controls.Add(labelPropriétés);

    // Zone de contenu principal (dock au centre - remplit l'espace restant)
    RichTextBox richTextBox = new RichTextBox
    {
        Dock = DockStyle.Fill,
        BorderStyle = BorderStyle.None,
        Text = "Cette zone utilise DockStyle.Fill et occupe tout l'espace restant après que les autres contrôles soient positionnés.\n\n" +
               "Le docking permet de créer facilement des interfaces complexes qui s'adaptent au redimensionnement.\n\n" +
               "Essayez de redimensionner la fenêtre pour voir comment les contrôles s'ajustent automatiquement."
    };

    // Ajout des contrôles au panel de démonstration dans le bon ordre
    // Note : L'ordre d'ajout est important pour le docking
    panelDémo.Controls.Add(richTextBox);      // Remplit l'espace central (doit être ajouté en premier)
    panelDémo.Controls.Add(panelNavigation);  // Dock à gauche
    panelDémo.Controls.Add(panelPropriétés);  // Dock à droite
    panelDémo.Controls.Add(toolStrip);        // Dock en haut
    panelDémo.Controls.Add(statusStrip);      // Dock en bas

    // Label explicatif
    Label labelExplication = new Label
    {
        Text = "Exemple d'interface avec docking - redimensionnez la fenêtre pour voir l'effet",
        Location = new Point(20, 430),
        Size = new Size(600, 20),
        Font = new Font("Segoe UI", 9F, FontStyle.Italic)
    };

    // Ajout du panel et du label au formulaire
    this.Controls.Add(panelDémo);
    this.Controls.Add(labelExplication);
}
```


### Combinaison de Docking et d'Ancrage

Il est parfois utile de combiner docking et ancrage pour créer des mises en page complexes.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void DémontrerCombinaisonDockingAncrage()
{
    // Panel principal avec docking
    Panel panelPrincipal = new Panel
    {
        Dock = DockStyle.Fill,
        Padding = new Padding(10)
    };

    // Panel d'en-tête
    Panel panelEntête = new Panel
    {
        Dock = DockStyle.Top,
        Height = 60,
        BackColor = Color.LightSteelBlue
    };

    Label labelTitre = new Label
    {
        Text = "Application de démonstration",
        Font = new Font("Segoe UI", 14F, FontStyle.Bold),
        Location = new Point(20, 15),
        AutoSize = true
    };

    Button buttonLogin = new Button
    {
        Text = "Connexion",
        Size = new Size(100, 30),
        Anchor = AnchorStyles.Top | AnchorStyles.Right,
        Location = new Point(panelEntête.Width - 120, 15)
    };

    panelEntête.Controls.Add(labelTitre);
    panelEntête.Controls.Add(buttonLogin);

    // Panel de contenu
    Panel panelContenu = new Panel
    {
        Dock = DockStyle.Fill,
        Padding = new Padding(10)
    };

    // Panel latéral gauche (utilise un pourcentage de la largeur)
    Panel panelGauche = new Panel
    {
        Dock = DockStyle.Left,
        Width = panelContenu.ClientSize.Width / 4, // 25% de la largeur
        BackColor = Color.WhiteSmoke,
        Padding = new Padding(5)
    };

    TreeView treeView = new TreeView
    {
        Dock = DockStyle.Fill
    };

    // Ajout de nœuds à l'arborescence
    TreeNode racine1 = new TreeNode("Dossiers");
    racine1.Nodes.Add(new TreeNode("Documents"));
    racine1.Nodes.Add(new TreeNode("Images"));
    racine1.Nodes.Add(new TreeNode("Musique"));

    TreeNode racine2 = new TreeNode("Projets");
    racine2.Nodes.Add(new TreeNode("Projet A"));
    racine2.Nodes.Add(new TreeNode("Projet B"));
    racine2.Nodes.Add(new TreeNode("Projet C"));

    treeView.Nodes.Add(racine1);
    treeView.Nodes.Add(racine2);
    treeView.ExpandAll();

    panelGauche.Controls.Add(treeView);

    // Zone centrale avec des contrôles ancrés
    Panel panelCentre = new Panel
    {
        Dock = DockStyle.Fill,
        Padding = new Padding(10, 5, 10, 10)
    };

    // Barre d'outils du panel central
    ToolStrip toolStrip = new ToolStrip
    {
        Dock = DockStyle.Top
    };

    toolStrip.Items.Add(new ToolStripButton("Vue liste"));
    toolStrip.Items.Add(new ToolStripButton("Vue icônes"));
    toolStrip.Items.Add(new ToolStripButton("Vue détails"));

    // Zone de contenu principal
    ListView listView = new ListView
    {
        Dock = DockStyle.Fill,
        View = View.Details
    };

    listView.Columns.Add("Nom", 150);
    listView.Columns.Add("Date", 120);
    listView.Columns.Add("Taille", 80);

    // Ajout d'éléments à la liste
    for (int i = 1; i <= 20; i++)
    {
        ListViewItem item = new ListViewItem($"Fichier {i}");
        item.SubItems.Add(DateTime.Now.AddDays(-i).ToShortDateString());
        item.SubItems.Add($"{i * 10} KB");

        listView.Items.Add(item);
    }

    // Zone de détails avec ancrage
    Panel panelDétails = new Panel
    {
        Dock = DockStyle.Bottom,
        Height = 100,
        BackColor = Color.AliceBlue
    };

    Label labelDétails = new Label
    {
        Text = "Détails du fichier",
        Font = new Font("Segoe UI", 9F, FontStyle.Bold),
        Location = new Point(10, 10),
        AutoSize = true
    };

    TextBox textBoxDétails = new TextBox
    {
        Multiline = true,
        ReadOnly = true,
        Location = new Point(10, 30),
        Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right,
        Size = new Size(panelDétails.Width - 20, panelDétails.Height - 40)
    };

    panelDétails.Controls.Add(labelDétails);
    panelDétails.Controls.Add(textBoxDétails);

    // Association d'événement pour afficher les détails
    listView.SelectedIndexChanged += (sender, e) => {
        if (listView.SelectedItems.Count > 0)
        {
            ListViewItem item = listView.SelectedItems[0];
            textBoxDétails.Text = $"Nom: {item.Text}\r\n" +
                                 $"Date: {item.SubItems[1].Text}\r\n" +
                                 $"Taille: {item.SubItems[2].Text}";
        }
        else
        {
            textBoxDétails.Text = "";
        }
    };

    // Assemblage des panneaux
    panelCentre.Controls.Add(listView);
    panelCentre.Controls.Add(toolStrip);
    panelCentre.Controls.Add(panelDétails);

    panelContenu.Controls.Add(panelCentre);
    panelContenu.Controls.Add(panelGauche);

    panelPrincipal.Controls.Add(panelContenu);
    panelPrincipal.Controls.Add(panelEntête);

    // Ajout du panel principal au formulaire
    this.Controls.Add(panelPrincipal);

    // Définir une taille minimum pour éviter les problèmes de redimensionnement
    this.MinimumSize = new Size(800, 600);
}
```


### Avantages et cas d'utilisation

#### Ancrage (Anchor)
- **Avantages** : Maintient les distances relatives, permet le redimensionnement proportionnel
- **Cas d'utilisation** : Contrôles qui doivent conserver leur position relative (ex: boutons en bas à droite, champs de texte qui s'étirent en largeur)

#### Docking (Dock)
- **Avantages** : Attache les contrôles aux bords, adapte automatiquement la taille, simplifie les interfaces complexes
- **Cas d'utilisation** : Barres d'outils, panneaux latéraux, zones de contenu principal, interfaces à plusieurs volets

## 7.5.4. Design adaptatif

Le design adaptatif consiste à créer des interfaces qui s'adaptent aux différentes tailles d'écran et résolutions.

### Mise à l'échelle automatique de formulaire

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class FormAdaptatif : Form
{
    private Size _tailleInitiale;
    private Dictionary<Control, Rectangle> _positionsInitiales = new Dictionary<Control, Rectangle>();
    private float _facteurEchelleX = 1.0f;
    private float _facteurEchelleY = 1.0f;

    public FormAdaptatif()
    {
        InitializeComponent();

        // Configurer le formulaire pour le redimensionnement
        this.MinimumSize = new Size(800, 600);
        this.AutoScaleMode = AutoScaleMode.Font;

        // Ajouter un gestionnaire pour le chargement du formulaire
        this.Load += FormAdaptatif_Load;

        // Ajouter un gestionnaire pour le redimensionnement
        this.SizeChanged += FormAdaptatif_SizeChanged;
    }

    private void FormAdaptatif_Load(object sender, EventArgs e)
    {
        // Enregistrer la taille initiale du formulaire
        _tailleInitiale = this.ClientSize;

        // Enregistrer la position initiale et la taille de tous les contrôles
        EnregistrerPositionsInitiales(this.Controls);
    }

    private void EnregistrerPositionsInitiales(Control.ControlCollection controls)
    {
        foreach (Control control in controls)
        {
            // Enregistrer la position et la taille de ce contrôle
            _positionsInitiales[control] = new Rectangle(control.Left, control.Top, control.Width, control.Height);

            // Traiter récursivement les contrôles enfants (sauf pour certains types de conteneurs spéciaux)
            if (control.Controls.Count > 0 &&
                !(control is TableLayoutPanel) &&
                !(control is FlowLayoutPanel) &&
                !(control is SplitContainer))
            {
                EnregistrerPositionsInitiales(control.Controls);
            }
        }
    }

    private void FormAdaptatif_SizeChanged(object sender, EventArgs e)
    {
        if (_tailleInitiale.Width == 0 || _tailleInitiale.Height == 0)
            return;

        // Calculer les facteurs d'échelle
        _facteurEchelleX = (float)this.ClientSize.Width / _tailleInitiale.Width;
        _facteurEchelleY = (float)this.ClientSize.Height / _tailleInitiale.Height;

        // Redimensionner tous les contrôles
        RedimensionnerControles(this.Controls);
    }

    private void RedimensionnerControles(Control.ControlCollection controls)
    {
        // Désactiver temporairement le redessinage
        this.SuspendLayout();

        foreach (Control control in controls)
        {
            // Vérifier si le contrôle a une position initiale enregistrée
            if (_positionsInitiales.TryGetValue(control, out Rectangle positionInitiale))
            {
                // Ne pas redimensionner certains types de conteneurs qui gèrent eux-mêmes la mise en page
                if (!(control is TableLayoutPanel) &&
                    !(control is FlowLayoutPanel) &&
                    !(control is SplitContainer))
                {
                    // Calculer la nouvelle position et taille
                    int x = (int)(positionInitiale.X * _facteurEchelleX);
                    int y = (int)(positionInitiale.Y * _facteurEchelleY);
                    int largeur = (int)(positionInitiale.Width * _facteurEchelleX);
                    int hauteur = (int)(positionInitiale.Height * _facteurEchelleY);

                    // Appliquer la nouvelle position et taille
                    control.Location = new Point(x, y);
                    control.Size = new Size(largeur, hauteur);

                    // Ajuster la taille de police si nécessaire
                    if (control is Label || control is Button || control is CheckBox || control is RadioButton)
                    {
                        // Facteur d'échelle moyen
                        float facteurMoyen = (float)Math.Sqrt(_facteurEchelleX * _facteurEchelleY);

                        // Ne pas redimensionner la police à chaque petit changement
                        if (facteurMoyen > 1.2f || facteurMoyen < 0.8f)
                        {
                            float taillePoliceOriginale = control.Font.Size;
                            float nouvelleTaillePolice = taillePoliceOriginale * facteurMoyen;

                            // Limiter la taille minimum et maximum
                            nouvelleTaillePolice = Math.Max(7, Math.Min(nouvelleTaillePolice, 24));

                            // Créer et appliquer la nouvelle police
                            control.Font = new Font(control.Font.FontFamily, nouvelleTaillePolice,
                                                    control.Font.Style, control.Font.Unit);
                        }
                    }

                    // Traiter récursivement les contrôles enfants
                    if (control.Controls.Count > 0)
                    {
                        RedimensionnerControles(control.Controls);
                    }
                }
            }
        }

        // Réactiver le redessinage
        this.ResumeLayout();
    }
}
```


### Adaptation à différentes résolutions d'écran

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class FormMultiRésolution : Form
{
    // Enum pour les modes d'affichage
    private enum ModeAffichage
    {
        Compact,    // Pour les petites résolutions
        Normal,     // Pour les résolutions moyennes
        Étendu      // Pour les grandes résolutions
    }

    private ModeAffichage _modeActuel = ModeAffichage.Normal;

    public FormMultiRésolution()
    {
        InitializeComponent();

        // Ajouter un gestionnaire pour le chargement du formulaire
        this.Load += FormMultiRésolution_Load;

        // Ajouter un gestionnaire pour le redimensionnement
        this.SizeChanged += FormMultiRésolution_SizeChanged;
    }

    private void FormMultiRésolution_Load(object sender, EventArgs e)
    {
        // Détecter la résolution d'écran et ajuster l'interface
        AdapterAuModeAffichage(DéterminerModeAffichage());
    }

    private void FormMultiRésolution_SizeChanged(object sender, EventArgs e)
    {
        // Adapter l'interface en fonction de la taille de la fenêtre
        ModeAffichage nouveauMode = DéterminerModeAffichage();

        // Ne changer que si le mode a changé
        if (nouveauMode != _modeActuel)
        {
            AdapterAuModeAffichage(nouveauMode);
        }
    }

    private ModeAffichage DéterminerModeAffichage()
    {
        // Déterminer le mode d'affichage en fonction de la largeur
        int largeur = this.ClientSize.Width;

        if (largeur < 600)
        {
            return ModeAffichage.Compact;
        }
        else if (largeur < 1000)
        {
            return ModeAffichage.Normal;
        }
        else
        {
            return ModeAffichage.Étendu;
        }
    }

    private void AdapterAuModeAffichage(ModeAffichage mode)
    {
        // Stocker le nouveau mode
        _modeActuel = mode;

        // Adapter l'interface en fonction du mode
        switch (mode)
        {
            case ModeAffichage.Compact:
                ConfigurerModeCompact();
                break;
            case ModeAffichage.Normal:
                ConfigurerModeNormal();
                break;
            case ModeAffichage.Étendu:
                ConfigurerModeÉtendu();
                break;
        }
    }

    private void ConfigurerModeCompact()
    {
        // Configuration pour les petites résolutions
        // Exemple avec un SplitContainer
        if (splitContainer1 != null)
        {
            // Masquer le panel de gauche
            splitContainer1.Panel1Collapsed = true;
        }

        // Réduire ou cacher certains éléments optionnels
        if (panelOptions != null)
        {
            panelOptions.Visible = false;
        }

        // Simplifier la barre d'outils
        if (toolStrip1 != null)
        {
            foreach (ToolStripItem item in toolStrip1.Items)
            {
                // Ne garder que les boutons essentiels
                item.Visible = item.Tag != null && (string)item.Tag == "Essential";
            }
        }

        // Afficher un menu contextuel pour accéder aux options masquées
        if (buttonOptions != null)
        {
            buttonOptions.Visible = true;
        }

        // Mettre à jour le libellé du mode
        labelMode.Text = "Mode Compact";
    }

    private void ConfigurerModeNormal()
    {
        // Configuration pour les résolutions moyennes
        if (splitContainer1 != null)
        {
            // Afficher le panel de gauche avec une taille modérée
            splitContainer1.Panel1Collapsed = false;
            splitContainer1.SplitterDistance = this.ClientSize.Width / 4; // 25% de la largeur
        }

        // Afficher les options de base
        if (panelOptions != null)
        {
            panelOptions.Visible = true;
        }

        // Configurer la barre d'outils
        if (toolStrip1 != null)
        {
            foreach (ToolStripItem item in toolStrip1.Items)
            {
                // Afficher tous les boutons sauf les options avancées
                item.Visible = item.Tag == null ||
                               (string)item.Tag == "Essential" ||
                               (string)item.Tag == "Standard";
            }
        }

        // Cacher le bouton d'options supplémentaires
        if (buttonOptions != null)
        {
            buttonOptions.Visible = false;
        }

        // Mettre à jour le libellé du mode
        labelMode.Text = "Mode Normal";
    }

    private void ConfigurerModeÉtendu()
    {
        // Configuration pour les grandes résolutions
        if (splitContainer1 != null)
        {
            // Afficher le panel de gauche avec une taille généreuse
            splitContainer1.Panel1Collapsed = false;
            splitContainer1.SplitterDistance = this.ClientSize.Width / 3; // 33% de la largeur
        }

        // Afficher toutes les options
        if (panelOptions != null)
        {
            panelOptions.Visible = true;

            // Ajouter des contrôles supplémentaires si nécessaire
            if (!panelOptions.Controls.Contains(panelOptionsAvancées))
            {
                panelOptions.Controls.Add(panelOptionsAvancées);
            }
        }

        // Afficher tous les éléments de la barre d'outils
        if (toolStrip1 != null)
        {
            foreach (ToolStripItem item in toolStrip1.Items)
            {
                item.Visible = true;
            }
        }

        // Cacher le bouton d'options supplémentaires
        if (buttonOptions != null)
        {
            buttonOptions.Visible = false;
        }

        // Mettre à jour le libellé du mode
        labelMode.Text = "Mode Étendu";
    }
}
```

### Contrôles responsifs

```textmate
// .NET Framework 4.7.2 et .NET 8
public class PanneauResponsif : Panel
{
    // Seuils de largeur pour les différents modes
    private const int SEUIL_COMPACT = 300;
    private const int SEUIL_NORMAL = 600;

    // Mode d'affichage actuel
    private enum ModeAffichage { Compact, Normal, Large }
    private ModeAffichage _modeActuel = ModeAffichage.Normal;

    // Contrôles
    private Label _labelTitre;
    private TextBox _textBoxRecherche;
    private Button _buttonRecherche;
    private ComboBox _comboBoxFiltre;
    private Label _labelFiltre;
    private Button _buttonAfficherTout;
    private Button _buttonOptions;

    public PanneauResponsif()
    {
        // Initialiser le panneau
        this.Dock = DockStyle.Top;
        this.Height = 60;
        this.BackColor = Color.WhiteSmoke;
        this.Padding = new Padding(10);

        // Créer les contrôles
        _labelTitre = new Label
        {
            Text = "Recherche",
            Font = new Font("Segoe UI", 12F, FontStyle.Bold),
            AutoSize = true,
            Location = new Point(10, 15)
        };

        _textBoxRecherche = new TextBox
        {
            Width = 200,
            Location = new Point(120, 15)
        };

        _buttonRecherche = new Button
        {
            Text = "Rechercher",
            Location = new Point(330, 14),
            Size = new Size(100, 26)
        };

        _labelFiltre = new Label
        {
            Text = "Filtre :",
            AutoSize = true,
            Location = new Point(450, 18)
        };

        _comboBoxFiltre = new ComboBox
        {
            Width = 150,
            Location = new Point(500, 15),
            DropDownStyle = ComboBoxStyle.DropDownList
        };

        // Ajouter des éléments au ComboBox
        _comboBoxFiltre.Items.AddRange(new object[] { "Tous", "Documents", "Images", "Vidéos" });
        _comboBoxFiltre.SelectedIndex = 0;

        _buttonAfficherTout = new Button
        {
            Text = "Afficher tout",
            Location = new Point(660, 14),
            Size = new Size(100, 26)
        };

        _buttonOptions = new Button
        {
            Text = "⋮",
            Width = 30,
            Location = new Point(770, 14),
            Size = new Size(30, 26)
        };

        // Ajouter les contrôles au panneau
        this.Controls.Add(_labelTitre);
        this.Controls.Add(_textBoxRecherche);
        this.Controls.Add(_buttonRecherche);
        this.Controls.Add(_labelFiltre);
        this.Controls.Add(_comboBoxFiltre);
        this.Controls.Add(_buttonAfficherTout);
        this.Controls.Add(_buttonOptions);

        // Abonner à l'événement de redimensionnement
        this.SizeChanged += PanneauResponsif_SizeChanged;

        // Configurer l'affichage initial
        AdapterAuModeAffichage(DéterminerModeAffichage());
    }

    private void PanneauResponsif_SizeChanged(object sender, EventArgs e)
    {
        // Adapter l'interface en fonction de la largeur du panneau
        ModeAffichage nouveauMode = DéterminerModeAffichage();

        // Ne changer que si le mode a changé
        if (nouveauMode != _modeActuel)
        {
            AdapterAuModeAffichage(nouveauMode);
        }
    }

    private ModeAffichage DéterminerModeAffichage()
    {
        // Déterminer le mode d'affichage en fonction de la largeur
        int largeur = this.ClientSize.Width;

        if (largeur < SEUIL_COMPACT)
        {
            return ModeAffichage.Compact;
        }
        else if (largeur < SEUIL_NORMAL)
        {
            return ModeAffichage.Normal;
        }
        else
        {
            return ModeAffichage.Large;
        }
    }

    private void AdapterAuModeAffichage(ModeAffichage mode)
    {
        // Stocker le nouveau mode
        _modeActuel = mode;

        // Adapter l'interface en fonction du mode
        switch (mode)
        {
            case ModeAffichage.Compact:
                ConfigurerModeCompact();
                break;
            case ModeAffichage.Normal:
                ConfigurerModeNormal();
                break;
            case ModeAffichage.Large:
                ConfigurerModeLarge();
                break;
        }
    }

    private void ConfigurerModeCompact()
    {
        // Mode très étroit - afficher uniquement le titre, la recherche et le menu
        this.Height = 60;

        _labelTitre.Visible = true;
        _labelTitre.Location = new Point(10, 18);

        _textBoxRecherche.Visible = true;
        _textBoxRecherche.Location = new Point(90, 15);
        _textBoxRecherche.Width = this.ClientSize.Width - 150;

        _buttonRecherche.Visible = false;

        _labelFiltre.Visible = false;
        _comboBoxFiltre.Visible = false;
        _buttonAfficherTout.Visible = false;

        _buttonOptions.Visible = true;
        _buttonOptions.Location = new Point(this.ClientSize.Width - 50, 14);

        // Configurer le menu d'options
        _buttonOptions.Click += (sender, e) => {
            // Créer un menu contextuel
            ContextMenuStrip menu = new ContextMenuStrip();
            menu.Items.Add("Rechercher", null, (s, args) => _buttonRecherche.PerformClick());
            menu.Items.Add("-"); // Séparateur
            menu.Items.Add("Filtrer par tous", null, (s, args) => _comboBoxFiltre.SelectedIndex = 0);
            menu.Items.Add("Filtrer par documents", null, (s, args) => _comboBoxFiltre.SelectedIndex = 1);
            menu.Items.Add("Filtrer par images", null, (s, args) => _comboBoxFiltre.SelectedIndex = 2);
            menu.Items.Add("Filtrer par vidéos", null, (s, args) => _comboBoxFiltre.SelectedIndex = 3);
            menu.Items.Add("-"); // Séparateur
            menu.Items.Add("Afficher tout", null, (s, args) => _buttonAfficherTout.PerformClick());

            // Afficher le menu
            menu.Show(_buttonOptions, new Point(0, _buttonOptions.Height));
        };
    }

    private void ConfigurerModeNormal()
    {
        // Mode moyen - afficher la recherche et les contrôles principaux
        this.Height = 60;

        _labelTitre.Visible = true;
        _labelTitre.Location = new Point(10, 18);

        _textBoxRecherche.Visible = true;
        _textBoxRecherche.Location = new Point(90, 15);
        _textBoxRecherche.Width = 150;

        _buttonRecherche.Visible = true;
        _buttonRecherche.Location = new Point(250, 14);
        _buttonRecherche.Text = "Rechercher";

        _labelFiltre.Visible = true;
        _labelFiltre.Location = new Point(360, 18);

        _comboBoxFiltre.Visible = true;
        _comboBoxFiltre.Location = new Point(410, 15);
        _comboBoxFiltre.Width = 120;

        _buttonAfficherTout.Visible = false;

        _buttonOptions.Visible = true;
        _buttonOptions.Location = new Point(this.ClientSize.Width - 50, 14);

        // Configurer le menu d'options réduit
        _buttonOptions.Click += (sender, e) => {
            ContextMenuStrip menu = new ContextMenuStrip();
            menu.Items.Add("Afficher tout", null, (s, args) => _buttonAfficherTout.PerformClick());
            menu.Show(_buttonOptions, new Point(0, _buttonOptions.Height));
        };
    }

    private void ConfigurerModeLarge()
    {
        // Mode large - afficher tous les contrôles
        this.Height = 60;

        _labelTitre.Visible = true;
        _labelTitre.Location = new Point(10, 18);

        _textBoxRecherche.Visible = true;
        _textBoxRecherche.Location = new Point(120, 15);
        _textBoxRecherche.Width = 200;

        _buttonRecherche.Visible = true;
        _buttonRecherche.Location = new Point(330, 14);
        _buttonRecherche.Size = new Size(100, 26);

        _labelFiltre.Visible = true;
        _labelFiltre.Location = new Point(450, 18);

        _comboBoxFiltre.Visible = true;
        _comboBoxFiltre.Location = new Point(500, 15);
        _comboBoxFiltre.Width = 150;

        _buttonAfficherTout.Visible = true;
        _buttonAfficherTout.Location = new Point(660, 14);

        _buttonOptions.Visible = false;
    }
}
```


### Bonnes pratiques pour le design adaptatif

- **Définir des seuils** : Identifiez les points de rupture où l'interface doit changer
- **Tester sur différentes résolutions** : Vérifiez que votre interface est utilisable à différentes tailles d'écran
- **Privilégier les tailles relatives** : Utilisez des pourcentages plutôt que des valeurs fixes pour les tailles
- **Utiliser judicieusement le docking et l'ancrage** : Combinez ces techniques pour obtenir un comportement adaptatif
- **Cacher plutôt que compresser** : Sur les petites résolutions, il est préférable de cacher certains éléments non essentiels plutôt que de les compresser
- **Fournir des alternatives** : Proposez des menus contextuels ou des boutons d'options pour accéder aux fonctionnalités masquées
- **Penser à l'expérience utilisateur** : L'interface doit rester cohérente et intuitive quelle que soit la résolution

## 7.5.5. Thèmes et apparence

Windows Forms permet de personnaliser l'apparence des applications pour offrir une expérience visuelle cohérente et agréable.

### Personnalisation de base

```textmate
// .NET Framework 4.7.2 et .NET 8
private void AppliquerThèmeDeBase()
{
    // Couleurs personnalisées
    Color couleurPrimaire = Color.FromArgb(65, 105, 225);    // Bleu royal
    Color couleurSecondaire = Color.FromArgb(240, 248, 255); // AliceBlue
    Color couleurAccent = Color.FromArgb(255, 69, 0);        // Orange-rouge
    Color couleurTexte = Color.FromArgb(50, 50, 50);         // Gris foncé

    // Appliquer les couleurs au formulaire
    this.BackColor = couleurSecondaire;
    this.ForeColor = couleurTexte;

    // Personnaliser les boutons
    foreach (Control control in this.Controls)
    {
        if (control is Button button)
        {
            button.BackColor = couleurPrimaire;
            button.ForeColor = Color.White;
            button.FlatStyle = FlatStyle.Flat;
            button.FlatAppearance.BorderColor = Color.FromArgb(couleurPrimaire.R - 20, couleurPrimaire.G - 20, couleurPrimaire.B - 20);
            button.FlatAppearance.BorderSize = 1;

            // Ajouter des effets de survol et de clic
            button.MouseEnter += (sender, e) => {
                button.BackColor = Color.FromArgb(couleurPrimaire.R + 30, couleurPrimaire.G + 30, couleurPrimaire.B + 30);
            };

            button.MouseLeave += (sender, e) => {
                button.BackColor = couleurPrimaire;
            };

            button.MouseDown += (sender, e) => {
                button.BackColor = Color.FromArgb(couleurPrimaire.R - 30, couleurPrimaire.G - 30, couleurPrimaire.B - 30);
            };

            button.MouseUp += (sender, e) => {
                button.BackColor = couleurPrimaire;
            };
        }
        else if (control is TextBox textBox)
        {
            textBox.BorderStyle = BorderStyle.FixedSingle;
            textBox.BackColor = Color.White;
            textBox.ForeColor = couleurTexte;
        }
        else if (control is Label label)
        {
            label.ForeColor = couleurTexte;

            // Style spécial pour les titres
            if (label.Tag != null && label.Tag.ToString() == "Titre")
            {
                label.ForeColor = couleurPrimaire;
                label.Font = new Font(label.Font.FontFamily, label.Font.Size + 2, FontStyle.Bold);
            }
        }
        else if (control is CheckBox checkBox)
        {
            checkBox.ForeColor = couleurTexte;
        }
        else if (control is RadioButton radioButton)
        {
            radioButton.ForeColor = couleurTexte;
        }
        else if (control is GroupBox groupBox)
        {
            groupBox.ForeColor = couleurPrimaire;
            groupBox.Font = new Font(groupBox.Font, FontStyle.Bold);
        }
        else if (control is Panel panel)
        {
            // Panels avec un style spécial
            if (panel.Tag != null && panel.Tag.ToString() == "Accentué")
            {
                panel.BackColor = couleurAccent;
                panel.ForeColor = Color.White;
            }
            else
            {
                panel.BackColor = couleurSecondaire;
            }
        }
    }
}
```


### Système de thèmes

```textmate
// .NET Framework 4.7.2 et .NET 8
public class ThèmeApplication
{
    // Propriétés du thème
    public Color CouleurPrimaire { get; set; }
    public Color CouleurSecondaire { get; set; }
    public Color CouleurAccent { get; set; }
    public Color CouleurTexte { get; set; }
    public Color CouleurFond { get; set; }
    public Font PoliceStandard { get; set; }
    public Font PoliceTitre { get; set; }
    public int RayonCoins { get; set; }
    public BorderStyle StyleBordure { get; set; }
    public FlatStyle StyleBouton { get; set; }

    // Thèmes prédéfinis
    public static ThèmeApplication ThèmeClair()
    {
        return new ThèmeApplication
        {
            CouleurPrimaire = Color.FromArgb(65, 105, 225),    // Bleu royal
            CouleurSecondaire = Color.FromArgb(240, 248, 255), // AliceBlue
            CouleurAccent = Color.FromArgb(255, 69, 0),        // Orange-rouge
            CouleurTexte = Color.FromArgb(50, 50, 50),         // Gris foncé
            CouleurFond = Color.White,
            PoliceStandard = new Font("Segoe UI", 9F),
            PoliceTitre = new Font("Segoe UI", 12F, FontStyle.Bold),
            RayonCoins = 0,
            StyleBordure = BorderStyle.FixedSingle,
            StyleBouton = FlatStyle.Flat
        };
    }

    public static ThèmeApplication ThèmeSombre()
    {
        return new ThèmeApplication
        {
            CouleurPrimaire = Color.FromArgb(25, 25, 112),     // Bleu nuit
            CouleurSecondaire = Color.FromArgb(47, 79, 79),    // Slate foncé
            CouleurAccent = Color.FromArgb(255, 140, 0),       // Orange foncé
            CouleurTexte = Color.FromArgb(220, 220, 220),      // Gris très clair
            CouleurFond = Color.FromArgb(30, 30, 30),          // Presque noir
            PoliceStandard = new Font("Segoe UI", 9F),
            PoliceTitre = new Font("Segoe UI", 12F, FontStyle.Bold),
            RayonCoins = 0,
            StyleBordure = BorderStyle.FixedSingle,
            StyleBouton = FlatStyle.Flat
        };
    }

    public static ThèmeApplication ThèmeBleuModerne()
    {
        return new ThèmeApplication
        {
            CouleurPrimaire = Color.FromArgb(0, 120, 215),     // Bleu Windows 10
            CouleurSecondaire = Color.FromArgb(230, 240, 250), // Bleu très pâle
            CouleurAccent = Color.FromArgb(0, 99, 177),        // Bleu accent
            CouleurTexte = Color.FromArgb(33, 33, 33),         // Presque noir
            CouleurFond = Color.White,
            PoliceStandard = new Font("Segoe UI", 9F),
            PoliceTitre = new Font("Segoe UI", 12F, FontStyle.Bold),
            RayonCoins = 2,
            StyleBordure = BorderStyle.None,
            StyleBouton = FlatStyle.Flat
        };
    }

    // Appliquer le thème à un formulaire
    public void AppliquerThème(Form formulaire)
    {
        formulaire.BackColor = CouleurFond;
        formulaire.ForeColor = CouleurTexte;
        formulaire.Font = PoliceStandard;

        // Parcourir récursivement tous les contrôles
        AppliquerThèmeAuxControles(formulaire.Controls);
    }

    private void AppliquerThèmeAuxControles(Control.ControlCollection controls)
    {
        foreach (Control control in controls)
        {
            // Configurer le contrôle en fonction de son type
            if (control is Button button)
            {
                StylerBouton(button);
            }
            else if (control is Label label)
            {
                StylerLabel(label);
            }
            else if (control is TextBox textBox)
            {
                StylerTextBox(textBox);
            }
            else if (control is Panel panel)
            {
                StylerPanel(panel);
            }
            else if (control is GroupBox groupBox)
            {
                StylerGroupBox(groupBox);
            }
            else if (control is CheckBox checkBox)
            {
                StylerCheckBox(checkBox);
            }
            else if (control is RadioButton radioButton)
            {
                StylerRadioButton(radioButton);
            }
            else if (control is ComboBox comboBox)
            {
                StylerComboBox(comboBox);
            }
            else if (control is ListBox listBox)
            {
                StylerListBox(listBox);
            }
            else if (control is DataGridView gridView)
            {
                StylerDataGridView(gridView);
            }

            // Appliquer récursivement aux contrôles enfants
            if (control.Controls.Count > 0)
            {
                AppliquerThèmeAuxControles(control.Controls);
            }
        }
    }

    // Méthodes de style pour chaque type de contrôle
    private void StylerBouton(Button button)
    {
        button.BackColor = CouleurPrimaire;
        button.ForeColor = button.Enabled ? Color.White : Color.LightGray;
        button.FlatStyle = StyleBouton;
        button.Font = PoliceStandard;

        if (StyleBouton == FlatStyle.Flat)
        {
            button.FlatAppearance.BorderColor = CouleurAccent;
            button.FlatAppearance.BorderSize = 1;
            button.FlatAppearance.MouseOverBackColor = Color.FromArgb(
                Math.Min(255, CouleurPrimaire.R + 30),
                Math.Min(255, CouleurPrimaire.G + 30),
                Math.Min(255, CouleurPrimaire.B + 30));
            button.FlatAppearance.MouseDownBackColor = Color.FromArgb(
                Math.Max(0, CouleurPrimaire.R - 30),
                Math.Max(0, CouleurPrimaire.G - 30),
                Math.Max(0, CouleurPrimaire.B - 30));
        }

        // Si le bouton a un tag spécial
        if (button.Tag != null)
        {
            if (button.Tag.ToString() == "Accent")
            {
                button.BackColor = CouleurAccent;
            }
            else if (button.Tag.ToString() == "Secondaire")
            {
                button.BackColor = CouleurSecondaire;
                button.ForeColor = CouleurTexte;
            }
        }
    }

    private void StylerLabel(Label label)
    {
        label.ForeColor = CouleurTexte;
        label.BackColor = Color.Transparent;

        // Si le label a un tag spécial
        if (label.Tag != null)
        {
            if (label.Tag.ToString() == "Titre")
            {
                label.Font = PoliceTitre;
                label.ForeColor = CouleurPrimaire;
            }
            else if (label.Tag.ToString() == "Sous-titre")
            {
                label.Font = new Font(PoliceStandard.FontFamily, PoliceStandard.Size + 1, FontStyle.Bold);
                label.ForeColor = CouleurPrimaire;
            }
        }
    }

    private void StylerTextBox(TextBox textBox)
    {
        textBox.BorderStyle = StyleBordure;
        textBox.BackColor = Color.White;
        textBox.ForeColor = CouleurTexte;

        // Style personnalisé avec bordure peinte
        if (StyleBordure == BorderStyle.None)
        {
            // Créer une bordure personnalisée
            textBox.BorderStyle = BorderStyle.None;

            // Ajouter un gestionnaire de dessin pour la bordure personnalisée
            Panel panelContainer = new Panel
            {
                Padding = new Padding(1),
                BackColor = CouleurPrimaire
            };

            // Insérer le TextBox dans le Panel
            if (textBox.Parent != null)
            {
                Point location = textBox.Location;
                Size size = textBox.Size;

                textBox.Parent.Controls.Remove(textBox);

                panelContainer.Size = new Size(size.Width + 2, size.Height + 2);
                panelContainer.Location = location;

                textBox.Dock = DockStyle.Fill;
                panelContainer.Controls.Add(textBox);

                textBox.Parent.Controls.Add(panelContainer);
            }
        }
    }

    private void StylerPanel(Panel panel)
    {
        panel.BackColor = CouleurFond;
        panel.ForeColor = CouleurTexte;

        // Si le panel a un tag spécial
        if (panel.Tag != null)
        {
            if (panel.Tag.ToString() == "Primaire")
            {
                panel.BackColor = CouleurPrimaire;
                panel.ForeColor = Color.White;
            }
            else if (panel.Tag.ToString() == "Secondaire")
            {
                panel.BackColor = CouleurSecondaire;
            }
            else if (panel.Tag.ToString() == "Accent")
            {
                panel.BackColor = CouleurAccent;
                panel.ForeColor = Color.White;
            }
        }
    }

    private void StylerGroupBox(GroupBox groupBox)
    {
        groupBox.ForeColor = CouleurPrimaire;
        groupBox.BackColor = CouleurFond;
        groupBox.Font = new Font(PoliceStandard, FontStyle.Bold);
    }

    private void StylerCheckBox(CheckBox checkBox)
    {
        checkBox.ForeColor = CouleurTexte;
        checkBox.BackColor = Color.Transparent;
    }

    private void StylerRadioButton(RadioButton radioButton)
    {
        radioButton.ForeColor = CouleurTexte;
        radioButton.BackColor = Color.Transparent;
    }

    private void StylerComboBox(ComboBox comboBox)
    {
        comboBox.BackColor = Color.White;
        comboBox.ForeColor = CouleurTexte;

        if (comboBox.FlatStyle != FlatStyle.System)
        {
            comboBox.FlatStyle = FlatStyle.Flat;
        }
    }

    private void StylerListBox(ListBox listBox)
    {
        listBox.BackColor = Color.White;
        listBox.ForeColor = CouleurTexte;
        listBox.BorderStyle = StyleBordure;
    }

    private void StylerDataGridView(DataGridView gridView)
    {
        gridView.BackgroundColor = CouleurFond;
        gridView.ForeColor = CouleurTexte;
        gridView.GridColor = CouleurSecondaire;

        gridView.ColumnHeadersDefaultCellStyle.BackColor = CouleurPrimaire;
        gridView.ColumnHeadersDefaultCellStyle.ForeColor = Color.White;
        gridView.ColumnHeadersDefaultCellStyle.Font = new Font(PoliceStandard, FontStyle.Bold);

        gridView.DefaultCellStyle.BackColor = CouleurFond;
        gridView.DefaultCellStyle.ForeColor = CouleurTexte;

        gridView.RowHeadersDefaultCellStyle.BackColor = CouleurSecondaire;

        gridView.AlternatingRowsDefaultCellStyle.BackColor = Color.FromArgb(
            Math.Max(0, CouleurFond.R - 10),
            Math.Max(0, CouleurFond.G - 10),
            Math.Max(0, CouleurFond.B - 10));
    }
}
```


### Utilisation du système de thèmes

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class FormPrincipal : Form
{
    private ThèmeApplication _thèmeActuel;

    public FormPrincipal()
    {
        InitializeComponent();

        // Configurer le menu de thèmes
        menuItemThèmeClair.Click += (sender, e) => ChangerThème(ThèmeApplication.ThèmeClair());
        menuItemThèmeSombre.Click += (sender, e) => ChangerThème(ThèmeApplication.ThèmeSombre());
        menuItemThèmeBleu.Click += (sender, e) => ChangerThème(ThèmeApplication.ThèmeBleuModerne());

        // Appliquer le thème par défaut
        _thèmeActuel = ThèmeApplication.ThèmeClair();
        _thèmeActuel.AppliquerThème(this);
    }

    private void ChangerThème(ThèmeApplication nouveauThème)
    {
        _thèmeActuel = nouveauThème;
        _thèmeActuel.AppliquerThème(this);

        // Enregistrer la préférence de thème dans les paramètres utilisateur
        Properties.Settings.Default.ThèmeNom = nouveauThème == ThèmeApplication.ThèmeClair() ? "Clair" :
                                            nouveauThème == ThèmeApplication.ThèmeSombre() ? "Sombre" : "Bleu";
        Properties.Settings.Default.Save();
    }

    // Charger le thème enregistré au démarrage
    private void FormPrincipal_Load(object sender, EventArgs e)
    {
        string thèmeEnregistré = Properties.Settings.Default.ThèmeNom;

        switch (thèmeEnregistré)
        {
            case "Sombre":
                ChangerThème(ThèmeApplication.ThèmeSombre());
                break;
            case "Bleu":
                ChangerThème(ThèmeApplication.ThèmeBleuModerne());
                break;
            default:
                ChangerThème(ThèmeApplication.ThèmeClair());
                break;
        }
    }
}
```


### Personnalisation des contrôles avec OwnerDrawn

```textmate
// .NET Framework 4.7.2 et .NET 8
public class BoutonPersonnalisé : Button
{
    // Propriétés de personnalisation
    public Color CouleurNormale { get; set; } = Color.FromArgb(65, 105, 225);
    public Color CouleurSurvol { get; set; } = Color.FromArgb(95, 135, 255);
    public Color CouleurClic { get; set; } = Color.FromArgb(35, 75, 195);
    public Color CouleurBordure { get; set; } = Color.FromArgb(25, 65, 185);
    public Color CouleurTexte { get; set; } = Color.White;
    public int RayonCoins { get; set; } = 8;

    // État actuel du bouton
    private bool _survol = false;
    private bool _enfoncé = false;

    public BoutonPersonnalisé()
    {
        // Configurer le bouton
        this.FlatStyle = FlatStyle.Flat;
        this.FlatAppearance.BorderSize = 0;
        this.FlatAppearance.MouseOverBackColor = Color.Transparent;
        this.FlatAppearance.MouseDownBackColor = Color.Transparent;

        // Abonner aux événements de la souris
        this.MouseEnter += (s, e) => { _survol = true; this.Invalidate(); };
        this.MouseLeave += (s, e) => { _survol = false; this.Invalidate(); };
        this.MouseDown += (s, e) => { if (e.Button == MouseButtons.Left) { _enfoncé = true; this.Invalidate(); } };
        this.MouseUp += (s, e) => { _enfoncé = false; this.Invalidate(); };

        // S'assurer que le bouton est redessiné lorsqu'il est activé/désactivé
        this.EnabledChanged += (s, e) => this.Invalidate();
    }

    protected override void OnPaint(PaintEventArgs e)
    {
        base.OnPaint(e);

        Graphics g = e.Graphics;
        g.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;

        // Déterminer la couleur de fond en fonction de l'état
        Color couleurFond;

        if (!this.Enabled)
        {
            couleurFond = Color.Gray;
        }
        else if (_enfoncé)
        {
            couleurFond = CouleurClic;
        }
        else if (_survol)
        {
            couleurFond = CouleurSurvol;
        }
        else
        {
            couleurFond = CouleurNormale;
        }

        // Dessiner le fond arrondi
        Rectangle rect = new Rectangle(0, 0, this.Width - 1, this.Height - 1);
        GraphicsPath path = ArrondiRectanglePath(rect, RayonCoins);

        using (SolidBrush brush = new SolidBrush(couleurFond))
        {
            g.FillPath(brush, path);
        }

        // Dessiner la bordure
        using (Pen pen = new Pen(CouleurBordure, 1))
        {
            g.DrawPath(pen, path);
        }

        // Dessiner le texte centré
        TextRenderer.DrawText(g, this.Text, this.Font, rect, CouleurTexte,
                             TextFormatFlags.HorizontalCenter | TextFormatFlags.VerticalCenter);

        // Dessiner l'image si présente
        if (this.Image != null)
        {
            int imageX = (this.Width - this.Image.Width) / 2;
            int imageY = (this.Height - this.Image.Height) / 2 - 10; // Plus haut que le centre

            if (imageY < 5) imageY = 5; // Marge minimum

            g.DrawImage(this.Image, imageX, imageY, this.Image.Width, this.Image.Height);

            // Ajuster la position du texte sous l'image
            Rectangle textRect = new Rectangle(0, imageY + this.Image.Height + 5,
                                              this.Width, this.Height - (imageY + this.Image.Height + 5));

            TextRenderer.DrawText(g, this.Text, this.Font, textRect, CouleurTexte,
                                 TextFormatFlags.HorizontalCenter | TextFormatFlags.VerticalCenter);
        }
    }

    // Méthode pour créer un chemin de rectangle arrondi
    private GraphicsPath ArrondiRectanglePath(Rectangle rect, int rayon)
    {
        GraphicsPath path = new GraphicsPath();

        // Limiter le rayon à la moitié de la plus petite dimension
        rayon = Math.Min(rayon, Math.Min(rect.Width, rect.Height) / 2);

        // Dessiner les segments du chemin
        path.AddArc(rect.X, rect.Y, rayon * 2, rayon * 2, 180, 90); // Coin supérieur gauche
        path.AddLine(rect.X + rayon, rect.Y, rect.Right - rayon, rect.Y); // Bord supérieur

        path.AddArc(rect.Right - rayon * 2, rect.Y, rayon * 2, rayon * 2, 270, 90); // Coin supérieur droit
        path.AddLine(rect.Right, rect.Y + rayon, rect.Right, rect.Bottom - rayon); // Bord droit

        path.AddArc(rect.Right - rayon * 2, rect.Bottom - rayon * 2, rayon * 2, rayon * 2, 0, 90); // Coin inférieur droit
        path.AddLine(rect.Right - rayon, rect.Bottom, rect.X + rayon, rect.Bottom); // Bord inférieur

        path.AddArc(rect.X, rect.Bottom - rayon * 2, rayon * 2, rayon * 2, 90, 90); // Coin inférieur gauche
        path.AddLine(rect.X, rect.Bottom - rayon, rect.X, rect.Y + rayon); // Bord gauche

        path.CloseFigure();
        return path;
    }
}

// Exemple d'utilisation avec une ListBox personnalisée
public class ListBoxPersonnalisée : ListBox
{
    // Propriétés de personnalisation
    public Color CouleurSélection { get; set; } = Color.FromArgb(65, 105, 225);
    public Color CouleurFondAlterné { get; set; } = Color.FromArgb(240, 240, 240);
    public int HauteurÉlément { get; set; } = 30;
    public int MargeGauche { get; set; } = 5;

    public ListBoxPersonnalisée()
    {
        // Configurer la ListBox
        this.DrawMode = DrawMode.OwnerDrawFixed;
        this.ItemHeight = HauteurÉlément;
        this.IntegralHeight = false;

        // Gérer l'événement DrawItem
        this.DrawItem += ListBoxPersonnalisée_DrawItem;
    }

    private void ListBoxPersonnalisée_DrawItem(object sender, DrawItemEventArgs e)
    {
        // Vérifier si l'index est valide
        if (e.Index < 0 || e.Index >= this.Items.Count)
            return;

        // Obtenir l'objet à dessiner
        string texte = this.Items[e.Index].ToString();

        // Dessiner le fond
        Color couleurFond;
        if ((e.State & DrawItemState.Selected) == DrawItemState.Selected)
        {
            couleurFond = CouleurSélection;
        }
        else
        {
            couleurFond = e.Index % 2 == 0 ? this.BackColor : CouleurFondAlterné;
        }

        using (SolidBrush brush = new SolidBrush(couleurFond))
        {
            e.Graphics.FillRectangle(brush, e.Bounds);
        }

        // Dessiner le texte
        Color couleurTexte = (e.State & DrawItemState.Selected) == DrawItemState.Selected ?
                            Color.White : this.ForeColor;

        Rectangle rectTexte = new Rectangle(
            e.Bounds.X + MargeGauche,
            e.Bounds.Y,
            e.Bounds.Width - MargeGauche * 2,
            e.Bounds.Height
        );

        TextRenderer.DrawText(e.Graphics, texte, this.Font, rectTexte, couleurTexte,
                             TextFormatFlags.VerticalCenter | TextFormatFlags.Left);

        // Dessiner le focus si nécessaire
        if ((e.State & DrawItemState.Focus) == DrawItemState.Focus)
        {
            ControlPaint.DrawFocusRectangle(e.Graphics, e.Bounds);
        }
    }
}

// Exemple d'un contrôle ComboBox personnalisé
public class ComboBoxPersonnalisé : ComboBox
{
    // Propriétés de personnalisation
    public Color CouleurBordure { get; set; } = Color.FromArgb(65, 105, 225);
    public Color CouleurFlèche { get; set; } = Color.FromArgb(65, 105, 225);
    public int ÉpaisseurBordure { get; set; } = 1;

    public ComboBoxPersonnalisé()
    {
        // Configurer le ComboBox
        this.DrawMode = DrawMode.OwnerDrawFixed;
        this.DropDownStyle = ComboBoxStyle.DropDownList;

        // Gérer les événements de dessin
        this.DrawItem += ComboBoxPersonnalisé_DrawItem;
    }

    protected override void OnPaint(PaintEventArgs e)
    {
        base.OnPaint(e);

        // Dessiner la bordure
        Rectangle rect = new Rectangle(0, 0, this.Width - 1, this.Height - 1);

        using (Pen pen = new Pen(CouleurBordure, ÉpaisseurBordure))
        {
            e.Graphics.DrawRectangle(pen, rect);
        }

        // Dessiner la flèche déroulante
        int largeurFlèche = 14;
        int hauteurFlèche = 6;

        Rectangle rectFlèche = new Rectangle(
            this.Width - largeurFlèche - 8,
            (this.Height - hauteurFlèche) / 2,
            largeurFlèche,
            hauteurFlèche);

        Point[] pointsFlèche = {
            new Point(rectFlèche.X, rectFlèche.Y),
            new Point(rectFlèche.X + rectFlèche.Width, rectFlèche.Y),
            new Point(rectFlèche.X + rectFlèche.Width / 2, rectFlèche.Y + rectFlèche.Height)
        };

        using (SolidBrush brush = new SolidBrush(CouleurFlèche))
        {
            e.Graphics.FillPolygon(brush, pointsFlèche);
        }
    }

    private void ComboBoxPersonnalisé_DrawItem(object sender, DrawItemEventArgs e)
    {
        // Vérifier si l'index est valide
        if (e.Index < 0 || e.Index >= this.Items.Count)
            return;

        // Obtenir l'objet à dessiner
        string texte = this.Items[e.Index].ToString();

        // Dessiner le fond
        e.DrawBackground();

        // Dessiner le texte
        using (SolidBrush brush = new SolidBrush(e.ForeColor))
        {
            Rectangle rectTexte = new Rectangle(
                e.Bounds.X + 5,
                e.Bounds.Y,
                e.Bounds.Width - 10,
                e.Bounds.Height
            );

            TextRenderer.DrawText(e.Graphics, texte, this.Font, rectTexte, e.ForeColor,
                                 TextFormatFlags.VerticalCenter | TextFormatFlags.Left);
        }

        // Dessiner le focus si nécessaire
        e.DrawFocusRectangle();
    }
}
```


### Bonnes pratiques en matière de mise en page et design

1. **Planifier avant de coder**
   - Esquissez votre interface à l'avance
   - Identifiez les zones et groupes logiques de contrôles
   - Pensez à la façon dont l'interface évoluera lors du redimensionnement

2. **Utiliser les bons conteneurs**
   - Choisissez le conteneur approprié à chaque situation (Panel, GroupBox, etc.)
   - Utilisez TableLayoutPanel pour les mises en page en grille
   - Utilisez FlowLayoutPanel pour les collections de contrôles similaires

3. **Appliquer correctement ancrage et docking**
   - Utilisez le docking pour les éléments qui doivent remplir ou s'accrocher à un bord
   - Utilisez l'ancrage pour les éléments qui doivent maintenir une distance fixe
   - Combinez les deux techniques pour des mises en page complexes

4. **Concevoir pour différentes résolutions**
   - Testez votre application sur différentes tailles d'écran
   - Adaptez votre interface en fonction de l'espace disponible
   - Pensez à la mise à l'échelle des polices et des contrôles

5. **Maintenir la cohérence visuelle**
   - Utilisez un système de thèmes cohérent
   - Standardisez les couleurs, polices et espacements
   - Créez des classes réutilisables pour les éléments d'interface personnalisés

6. **Favoriser l'accessibilité**
   - Utilisez des contrastes suffisants entre texte et fond
   - Assurez-vous que les textes sont lisibles
   - Prévoyez des tailles de contrôles suffisantes pour une utilisation tactile

7. **Optimiser les performances**
   - Utilisez SuspendLayout() et ResumeLayout() lors de modifications multiples
   - Évitez de redessiner inutilement les contrôles
   - Groupez les contrôles dans des conteneurs pour faciliter la gestion

En suivant ces bonnes pratiques, vous créerez des interfaces utilisateur professionnelles, cohérentes et agréables à utiliser, qui s'adaptent à différentes tailles d'écran et résolutions tout en maintenant une apparence visuelle attrayante.

⏭️
