# 7.3. Contrôles de base et leurs propriétés

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les contrôles sont les éléments fondamentaux de toute interface utilisateur Windows Forms. Cette section présente les contrôles les plus couramment utilisés, leurs propriétés principales, et comment créer vos propres contrôles personnalisés.

## 7.3.1. Button, Label, TextBox, CheckBox, etc.

Windows Forms fournit une large gamme de contrôles prêts à l'emploi pour construire vos interfaces utilisateur. Voici les contrôles de base les plus couramment utilisés.

### Button

Le contrôle Button est l'un des éléments les plus fondamentaux d'une interface utilisateur. Il permet à l'utilisateur de déclencher une action.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeButton()
{
    Button button = new Button
    {
        Text = "Cliquez-moi",
        Location = new Point(50, 50),
        Size = new Size(100, 30),
        BackColor = Color.LightBlue,
        ForeColor = Color.Black,
        Font = new Font("Arial", 9F, FontStyle.Bold),
        Name = "button1",
        TabIndex = 0,
        UseVisualStyleBackColor = true
    };

    // Attacher un gestionnaire d'événements
    button.Click += Button_Click;

    // Ajouter le bouton au formulaire
    this.Controls.Add(button);
}

private void Button_Click(object sender, EventArgs e)
{
    MessageBox.Show("Le bouton a été cliqué !");
}
```


**Propriétés importantes du Button :**
- `Text` : Le texte affiché sur le bouton
- `Image` : Une image à afficher sur le bouton
- `ImageAlign` : L'alignement de l'image par rapport au texte
- `TextAlign` : L'alignement du texte dans le bouton
- `FlatStyle` : Le style visuel du bouton (Standard, Flat, Popup, System)
- `DialogResult` : La valeur retournée si le bouton est utilisé dans une boîte de dialogue

**Événements clés :**
- `Click` : Déclenché lorsque l'utilisateur clique sur le bouton
- `MouseEnter`/`MouseLeave` : Déclenchés lorsque le curseur entre ou quitte la zone du bouton

### Label

Le contrôle Label affiche du texte non modifiable par l'utilisateur. Il est utilisé pour les titres, les descriptions et les instructions.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeLabel()
{
    Label label = new Label
    {
        Text = "Entrez votre nom :",
        Location = new Point(50, 20),
        Size = new Size(150, 20),
        AutoSize = true,
        Font = new Font("Segoe UI", 9F),
        ForeColor = Color.Navy,
        Name = "labelNom",
        TextAlign = ContentAlignment.MiddleLeft
    };

    this.Controls.Add(label);
}
```


**Propriétés importantes du Label :**
- `Text` : Le texte affiché
- `AutoSize` : Détermine si le contrôle redimensionne automatiquement sa taille pour s'adapter au contenu
- `TextAlign` : L'alignement du texte à l'intérieur du Label
- `BorderStyle` : Le style de bordure
- `UseMnemonic` : Détermine si les caractères précédés d'un & sont interprétés comme des raccourcis clavier

### TextBox

Le contrôle TextBox permet à l'utilisateur de saisir du texte.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeTextBox()
{
    TextBox textBox = new TextBox
    {
        Location = new Point(50, 50),
        Size = new Size(200, 23),
        Text = "",
        PlaceholderText = "Entrez votre nom ici", // Disponible uniquement dans .NET Core/.NET 5+
        MaxLength = 50,
        Name = "textBoxNom",
        TabIndex = 1,
        PasswordChar = '\0', // Caractère affiché pour les mots de passe
        Multiline = false
    };

    // Attacher des gestionnaires d'événements
    textBox.TextChanged += TextBox_TextChanged;

    this.Controls.Add(textBox);
}

private void TextBox_TextChanged(object sender, EventArgs e)
{
    TextBox textBox = (TextBox)sender;
    Console.WriteLine($"Texte modifié : {textBox.Text}");
}
```


**Propriétés importantes du TextBox :**
- `Text` : Le texte contenu dans le TextBox
- `PlaceholderText` (.NET 5+) : Texte indicatif affiché lorsque le TextBox est vide
- `MaxLength` : Nombre maximum de caractères autorisés
- `Multiline` : Détermine si le TextBox peut contenir plusieurs lignes
- `ReadOnly` : Détermine si le texte peut être modifié par l'utilisateur
- `PasswordChar` : Caractère affiché à la place du texte réel pour masquer une saisie sensible
- `WordWrap` : Active/désactive le retour à la ligne automatique dans un TextBox multiline
- `ScrollBars` : Configure les barres de défilement pour un TextBox multiline

**Événements clés :**
- `TextChanged` : Déclenché lorsque le texte change
- `KeyPress` : Déclenché lorsqu'un caractère est saisi
- `KeyDown`/`KeyUp` : Déclenchés lorsqu'une touche est enfoncée/relâchée

### CheckBox

Le contrôle CheckBox permet à l'utilisateur de sélectionner une option (oui/non).

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeCheckBox()
{
    CheckBox checkBox = new CheckBox
    {
        Text = "Accepter les conditions d'utilisation",
        Location = new Point(50, 80),
        Size = new Size(250, 20),
        Checked = false,
        ThreeState = false,  // Si true, permet l'état indéterminé
        Name = "checkBoxAccept",
        TabIndex = 2
    };

    // Attacher un gestionnaire d'événements
    checkBox.CheckedChanged += CheckBox_CheckedChanged;

    this.Controls.Add(checkBox);
}

private void CheckBox_CheckedChanged(object sender, EventArgs e)
{
    CheckBox checkBox = (CheckBox)sender;
    Console.WriteLine($"État : {checkBox.Checked}");
}
```


**Propriétés importantes du CheckBox :**
- `Checked` : État actuel (coché ou non)
- `CheckState` : État actuel avec possibilité d'une valeur indéterminée
- `ThreeState` : Détermine si le CheckBox peut avoir trois états (coché, non coché, indéterminé)
- `AutoCheck` : Détermine si l'état change automatiquement lors d'un clic

**Événements clés :**
- `CheckedChanged` : Déclenché lorsque l'état change
- `CheckStateChanged` : Déclenché lorsque la propriété CheckState change

### RadioButton

Le contrôle RadioButton permet à l'utilisateur de sélectionner une option parmi plusieurs choix mutuellement exclusifs.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeRadioButtons()
{
    // Premier RadioButton
    RadioButton radioButton1 = new RadioButton
    {
        Text = "Option 1",
        Location = new Point(50, 110),
        Size = new Size(100, 20),
        Checked = true,
        Name = "radioOption1",
        TabIndex = 3
    };

    // Deuxième RadioButton
    RadioButton radioButton2 = new RadioButton
    {
        Text = "Option 2",
        Location = new Point(50, 130),
        Size = new Size(100, 20),
        Checked = false,
        Name = "radioOption2",
        TabIndex = 4
    };

    // Attacher des gestionnaires d'événements
    radioButton1.CheckedChanged += RadioButton_CheckedChanged;
    radioButton2.CheckedChanged += RadioButton_CheckedChanged;

    this.Controls.Add(radioButton1);
    this.Controls.Add(radioButton2);
}

private void RadioButton_CheckedChanged(object sender, EventArgs e)
{
    RadioButton radioButton = (RadioButton)sender;
    if (radioButton.Checked) // N'exécuter que si le bouton est sélectionné
    {
        Console.WriteLine($"Option sélectionnée : {radioButton.Text}");
    }
}
```


**Propriétés importantes du RadioButton :**
- `Checked` : Indique si le bouton radio est sélectionné
- `Appearance` : Modifie l'apparence (Normal ou Button)
- `AutoCheck` : Détermine si l'état change automatiquement lors d'un clic

**Événements clés :**
- `CheckedChanged` : Déclenché lorsque l'état de sélection change

### ComboBox

Le contrôle ComboBox combine une zone de texte avec une liste déroulante, permettant à l'utilisateur de sélectionner un élément ou de saisir une valeur.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeComboBox()
{
    ComboBox comboBox = new ComboBox
    {
        Location = new Point(50, 160),
        Size = new Size(200, 23),
        DropDownStyle = ComboBoxStyle.DropDownList, // DropDown, Simple, DropDownList
        Name = "comboBoxPays",
        TabIndex = 5
    };

    // Ajouter des éléments
    comboBox.Items.Add("France");
    comboBox.Items.Add("Belgique");
    comboBox.Items.Add("Suisse");
    comboBox.Items.Add("Canada");

    // Sélectionner un élément par défaut
    comboBox.SelectedIndex = 0;

    // Attacher un gestionnaire d'événements
    comboBox.SelectedIndexChanged += ComboBox_SelectedIndexChanged;

    this.Controls.Add(comboBox);
}

private void ComboBox_SelectedIndexChanged(object sender, EventArgs e)
{
    ComboBox comboBox = (ComboBox)sender;
    Console.WriteLine($"Sélection : {comboBox.SelectedItem}");
}
```


**Propriétés importantes du ComboBox :**
- `Items` : Collection des éléments de la liste
- `SelectedIndex` : Index de l'élément sélectionné
- `SelectedItem` : Élément actuellement sélectionné
- `DropDownStyle` : Style d'affichage (DropDown, Simple, DropDownList)
- `MaxDropDownItems` : Nombre maximum d'éléments visibles lorsque la liste est déroulée
- `AutoCompleteMode` : Mode de complétion automatique
- `Sorted` : Détermine si les éléments sont triés par ordre alphabétique

**Événements clés :**
- `SelectedIndexChanged` : Déclenché lorsque la sélection change
- `TextChanged` : Déclenché lorsque le texte change (pour DropDown et Simple)
- `DropDown`/`DropDownClosed` : Déclenchés lorsque la liste déroulante s'ouvre/se ferme

### ListBox

Le contrôle ListBox affiche une liste d'éléments parmi lesquels l'utilisateur peut effectuer une ou plusieurs sélections.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeListBox()
{
    ListBox listBox = new ListBox
    {
        Location = new Point(50, 200),
        Size = new Size(200, 100),
        SelectionMode = SelectionMode.MultiSimple, // One, MultiSimple, MultiExtended
        Name = "listBoxLanguages",
        TabIndex = 6,
        Sorted = true
    };

    // Ajouter des éléments
    listBox.Items.Add("C#");
    listBox.Items.Add("JavaScript");
    listBox.Items.Add("Python");
    listBox.Items.Add("Java");
    listBox.Items.Add("TypeScript");

    // Attacher un gestionnaire d'événements
    listBox.SelectedIndexChanged += ListBox_SelectedIndexChanged;

    this.Controls.Add(listBox);
}

private void ListBox_SelectedIndexChanged(object sender, EventArgs e)
{
    ListBox listBox = (ListBox)sender;

    // Afficher tous les éléments sélectionnés
    string selections = string.Join(", ", listBox.SelectedItems.Cast<string>());
    Console.WriteLine($"Sélections : {selections}");
}
```


**Propriétés importantes du ListBox :**
- `Items` : Collection des éléments de la liste
- `SelectedIndex` : Index de l'élément sélectionné (simple sélection)
- `SelectedIndices` : Collection des indices des éléments sélectionnés (multi-sélection)
- `SelectedItem` : Élément actuellement sélectionné (simple sélection)
- `SelectedItems` : Collection des éléments sélectionnés (multi-sélection)
- `SelectionMode` : Mode de sélection (One, MultiSimple, MultiExtended)
- `MultiColumn` : Détermine si la liste s'affiche sur plusieurs colonnes
- `Sorted` : Détermine si les éléments sont triés par ordre alphabétique

**Événements clés :**
- `SelectedIndexChanged` : Déclenché lorsque la sélection change
- `DoubleClick` : Déclenché lorsqu'un élément est double-cliqué

### PictureBox

Le contrôle PictureBox affiche des images.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializePictureBox()
{
    PictureBox pictureBox = new PictureBox
    {
        Location = new Point(300, 50),
        Size = new Size(150, 150),
        SizeMode = PictureBoxSizeMode.Zoom, // Normal, StretchImage, AutoSize, CenterImage, Zoom
        Image = Image.FromFile("chemin/vers/image.jpg"), // Ou Resource.Image pour les ressources intégrées
        BorderStyle = BorderStyle.FixedSingle,
        Name = "pictureBoxLogo",
        TabIndex = 7
    };

    // Attacher un gestionnaire d'événements
    pictureBox.Click += PictureBox_Click;

    this.Controls.Add(pictureBox);
}

private void PictureBox_Click(object sender, EventArgs e)
{
    // Ouvrir un sélecteur de fichier pour changer l'image
    using (OpenFileDialog openFileDialog = new OpenFileDialog())
    {
        openFileDialog.Filter = "Images|*.jpg;*.jpeg;*.png;*.gif;*.bmp|Tous les fichiers|*.*";

        if (openFileDialog.ShowDialog() == DialogResult.OK)
        {
            try
            {
                PictureBox pictureBox = (PictureBox)sender;
                pictureBox.Image = Image.FromFile(openFileDialog.FileName);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Erreur lors du chargement de l'image : {ex.Message}");
            }
        }
    }
}
```


**Propriétés importantes du PictureBox :**
- `Image` : L'image affichée
- `SizeMode` : Comment l'image est redimensionnée/positionnée
- `ImageLocation` : URL ou chemin de fichier de l'image
- `ErrorImage` : Image affichée en cas d'erreur de chargement
- `InitialImage` : Image affichée pendant le chargement de l'image principale

**Événements clés :**
- `Click` : Déclenché lorsque l'utilisateur clique sur l'image
- `LoadCompleted` : Déclenché lorsque le chargement de l'image est terminé

### NumericUpDown

Le contrôle NumericUpDown permet à l'utilisateur de sélectionner une valeur numérique en utilisant des boutons haut/bas ou par saisie directe.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeNumericUpDown()
{
    NumericUpDown numericUpDown = new NumericUpDown
    {
        Location = new Point(300, 220),
        Size = new Size(80, 23),
        Minimum = 0,
        Maximum = 100,
        Value = 10,
        Increment = 5,
        DecimalPlaces = 0,
        ThousandsSeparator = true,
        Name = "numericQuantité",
        TabIndex = 8
    };

    // Attacher un gestionnaire d'événements
    numericUpDown.ValueChanged += NumericUpDown_ValueChanged;

    this.Controls.Add(numericUpDown);
}

private void NumericUpDown_ValueChanged(object sender, EventArgs e)
{
    NumericUpDown numericUpDown = (NumericUpDown)sender;
    Console.WriteLine($"Valeur : {numericUpDown.Value}");
}
```


**Propriétés importantes du NumericUpDown :**
- `Value` : Valeur actuelle
- `Minimum`/`Maximum` : Valeurs minimale et maximale autorisées
- `Increment` : Pas d'incrémentation/décrémentation
- `DecimalPlaces` : Nombre de décimales affichées
- `ThousandsSeparator` : Active/désactive l'affichage du séparateur de milliers
- `UpDownAlign` : Alignement des boutons haut/bas

**Événements clés :**
- `ValueChanged` : Déclenché lorsque la valeur change

### DateTimePicker

Le contrôle DateTimePicker permet à l'utilisateur de sélectionner une date et éventuellement une heure.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeDateTimePicker()
{
    DateTimePicker dateTimePicker = new DateTimePicker
    {
        Location = new Point(300, 260),
        Size = new Size(150, 23),
        Format = DateTimePickerFormat.Short, // Short, Long, Time, Custom
        CustomFormat = "dd/MM/yyyy HH:mm", // Utilisé si Format = Custom
        MinDate = new DateTime(2000, 1, 1),
        MaxDate = new DateTime(2030, 12, 31),
        Value = DateTime.Today,
        ShowUpDown = false,
        Name = "dateTimePickerDate",
        TabIndex = 9
    };

    // Attacher un gestionnaire d'événements
    dateTimePicker.ValueChanged += DateTimePicker_ValueChanged;

    this.Controls.Add(dateTimePicker);
}

private void DateTimePicker_ValueChanged(object sender, EventArgs e)
{
    DateTimePicker dateTimePicker = (DateTimePicker)sender;
    Console.WriteLine($"Date sélectionnée : {dateTimePicker.Value}");
}
```


**Propriétés importantes du DateTimePicker :**
- `Value` : Date et heure sélectionnées
- `MinDate`/`MaxDate` : Dates minimale et maximale autorisées
- `Format` : Format d'affichage (Short, Long, Time, Custom)
- `CustomFormat` : Format personnalisé
- `ShowUpDown` : Affiche des boutons haut/bas au lieu d'un calendrier déroulant
- `ShowCheckBox` : Affiche une case à cocher pour permettre une valeur nulle

**Événements clés :**
- `ValueChanged` : Déclenché lorsque la date/heure sélectionnée change

### ProgressBar

Le contrôle ProgressBar affiche visuellement la progression d'une opération.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeProgressBar()
{
    ProgressBar progressBar = new ProgressBar
    {
        Location = new Point(50, 320),
        Size = new Size(400, 23),
        Minimum = 0,
        Maximum = 100,
        Value = 0,
        Style = ProgressBarStyle.Blocks, // Blocks ou Marquee (.NET 5+)
        Step = 10,
        Name = "progressBarChargement",
        TabIndex = 10
    };

    // Exemple d'utilisation avec un Timer
    Timer timer = new Timer
    {
        Interval = 500,
        Enabled = true
    };

    timer.Tick += (sender, e) => {
        if (progressBar.Value < progressBar.Maximum)
        {
            // Incrémenter de la valeur du Step
            progressBar.PerformStep();
        }
        else
        {
            // Arrêter le timer lorsque la progression est terminée
            timer.Stop();
            MessageBox.Show("Opération terminée !");
        }
    };

    this.Controls.Add(progressBar);
    this.components.Add(timer); // Ajouter au conteneur de composants
}
```


**Propriétés importantes du ProgressBar :**
- `Value` : Valeur actuelle
- `Minimum`/`Maximum` : Valeurs minimale et maximale
- `Style` : Style d'affichage (Blocks, Continuous, Marquee)
- `Step` : Incrément utilisé par la méthode PerformStep()
- `MarqueeAnimationSpeed` : Vitesse d'animation en mode Marquee

### TrackBar (Slider)

Le contrôle TrackBar permet à l'utilisateur de sélectionner une valeur dans une plage en déplaçant un curseur.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeTrackBar()
{
    TrackBar trackBar = new TrackBar
    {
        Location = new Point(50, 360),
        Size = new Size(300, 45),
        Minimum = 0,
        Maximum = 100,
        Value = 50,
        SmallChange = 1,
        LargeChange = 10,
        TickFrequency = 10,
        TickStyle = TickStyle.Both,
        Orientation = Orientation.Horizontal,
        Name = "trackBarVolume",
        TabIndex = 11
    };

    // Attacher un gestionnaire d'événements
    trackBar.ValueChanged += TrackBar_ValueChanged;

    this.Controls.Add(trackBar);
}

private void TrackBar_ValueChanged(object sender, EventArgs e)
{
    TrackBar trackBar = (TrackBar)sender;
    Console.WriteLine($"Valeur : {trackBar.Value}");
}
```


**Propriétés importantes du TrackBar :**
- `Value` : Valeur actuelle
- `Minimum`/`Maximum` : Valeurs minimale et maximale
- `SmallChange` : Incrément pour les touches fléchées
- `LargeChange` : Incrément pour la touche Page Up/Down
- `TickFrequency` : Fréquence des marques sur la piste
- `TickStyle` : Style des marques (None, TopLeft, BottomRight, Both)
- `Orientation` : Orientation (Horizontal, Vertical)

**Événements clés :**
- `ValueChanged` : Déclenché lorsque la valeur change
- `Scroll` : Déclenché pendant le déplacement du curseur

## 7.3.2. Propriétés communes

La plupart des contrôles Windows Forms partagent un ensemble de propriétés communes qui affectent leur apparence et leur comportement.

### Propriétés liées à l'apparence

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerApparence(Control control)
{
    // Apparence générale
    control.BackColor = Color.White;
    control.ForeColor = Color.Black;
    control.Font = new Font("Segoe UI", 9F);
    control.Cursor = Cursors.Hand;

    // Bordure (pour les contrôles qui la supportent)
    if (control is TextBox || control is Label)
    {
        // Utilisation de reflection pour définir la propriété BorderStyle
        var borderStyleProperty = control.GetType().GetProperty("BorderStyle");
        if (borderStyleProperty != null)
        {
            borderStyleProperty.SetValue(control, BorderStyle.FixedSingle);
        }
    }

    // Opacité et transparence
    control.Opacity = 0.9; // .NET 5+ uniquement
    control.BackColor = Color.FromArgb(150, 255, 255, 255); // Couleur semi-transparente

    // Style de la police
    control.Font = new Font("Arial", 10F, FontStyle.Bold | FontStyle.Italic);

    // Alignement (pour les contrôles qui supportent le texte)
    if (control is Button button)
    {
        button.TextAlign = ContentAlignment.MiddleCenter;
    }
    else if (control is Label label)
    {
        label.TextAlign = ContentAlignment.MiddleLeft;
    }
}
```


**Propriétés d'apparence communes :**
- `BackColor` : Couleur d'arrière-plan
- `ForeColor` : Couleur du texte
- `Font` : Police utilisée pour le texte
- `Cursor` : Pointeur de souris affiché au survol
- `Opacity` (.NET 5+) : Niveau d'opacité du contrôle
- `BorderStyle` : Style de bordure (applicable à certains contrôles)
- `BackgroundImage` : Image d'arrière-plan
- `BackgroundImageLayout` : Comment l'image d'arrière-plan est affichée
- `RightToLeft` : Direction du texte (pour les langues écrites de droite à gauche)
- `Padding` : Espace intérieur entre le contenu et les bords du contrôle
- `Margin` : Espace extérieur entre le contrôle et les autres contrôles

### Propriétés liées à la disposition

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerDisposition(Control control)
{
    // Position et taille
    control.Location = new Point(100, 100);
    control.Size = new Size(150, 30);
    control.Anchor = AnchorStyles.Top | AnchorStyles.Left;
    control.Dock = DockStyle.None;

    // Positionnement
    control.AutoSize = true;
    control.AutoSizeMode = AutoSizeMode.GrowAndShrink;
    control.MinimumSize = new Size(50, 20);
    control.MaximumSize = new Size(300, 100);

    // Z-order
    control.BringToFront(); // Amène le contrôle au premier plan
    // ou
    control.SendToBack(); // Envoie le contrôle à l'arrière-plan
}
```


**Propriétés de disposition communes :**
- `Location` : Position du contrôle dans son conteneur parent
- `Size` : Taille du contrôle (largeur et hauteur)
- `Anchor` : Comment le contrôle est ancré aux bords de son conteneur parent
- `Dock` : Comment le contrôle est amarré dans son conteneur parent
- `AutoSize` : Si le contrôle s'adapte automatiquement à son contenu
- `AutoSizeMode` : Comment le contrôle s'adapte à son contenu
- `MinimumSize`/`MaximumSize` : Tailles minimale et maximale du contrôle
- `Margin` : Espacement autour du contrôle
- `Padding` : Espacement intérieur du contrôle

### Propriétés liées au comportement

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerComportement(Control control)
{
    // Interaction utilisateur
    control.Enabled = true;
    control.Visible = true;
    control.TabIndex = 0;
    control.TabStop = true;
    control.AllowDrop = true;

    // Focus
    control.Focus(); // Donner le focus à ce contrôle

    // Accessibilité
    control.AccessibleName = "Mon contrôle";
    control.AccessibleDescription = "Description du contrôle pour l'accessibilité";
    control.AccessibleRole = AccessibleRole.PushButton;

    // Infobulles
    ToolTip toolTip = new ToolTip();
    toolTip.SetToolTip(control, "Ceci est une infobulle");

    // Raccourcis clavier
    if (control is Button button)
    {
        button.Text = "&Valider"; // Le 'V' devient un raccourci avec Alt+V
    }
}
```


**Propriétés de comportement communes :**
- `Enabled` : Détermine si le contrôle peut répondre aux interactions
- `Visible` : Détermine si le contrôle est visible
- `TabIndex` : Ordre du contrôle dans la séquence de tabulation
- `TabStop` : Détermine si le contrôle peut recevoir le focus via la touche Tab
- `AllowDrop` : Détermine si le contrôle peut être une cible de glisser-déposer
- `CausesValidation` : Détermine si le contrôle déclenche la validation lors de la perte du focus
- `ContextMenuStrip` : Menu contextuel associé au contrôle
- `Focused` : Indique si le contrôle a le focus

### Propriétés spécifiques aux formulaires

Les formulaires (Form) ont des propriétés supplémentaires pour contrôler leur apparence et leur comportement.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerFormulaire(Form form)
{
    // Apparence générale
    form.Text = "Mon Application";
    form.Icon = new Icon("chemin/vers/icone.ico");
    form.BackColor = Color.White;
    form.ForeColor = Color.Black;
    form.Font = new Font("Segoe UI", 9F);
    form.FormBorderStyle = FormBorderStyle.Sizable;
    form.TransparencyKey = Color.Magenta; // Couleur traitée comme transparente

    // Taille et position
    form.Size = new Size(800, 600);
    form.MinimumSize = new Size(400, 300);
    form.MaximumSize = new Size(1200, 900);
    form.StartPosition = FormStartPosition.CenterScreen;
    form.WindowState = FormWindowState.Normal;

    // Comportement
    form.ShowInTaskbar = true;
    form.TopMost = false;
    form.ShowIcon = true;
    form.MinimizeBox = true;
    form.MaximizeBox = true;
    form.ControlBox = true;
    form.AcceptButton = buttonOK; // Bouton activé par la touche Entrée
    form.CancelButton = buttonAnnuler; // Bouton activé par la touche Échap

    // Pour les boîtes de dialogue modales
    form.FormBorderStyle = FormBorderStyle.FixedDialog;
    form.MaximizeBox = false;
    form.MinimizeBox = false;
    form.StartPosition = FormStartPosition.CenterParent;

    // Événements spécifiques aux formulaires
    form.Load += Form_Load;
    form.FormClosing += Form_FormClosing;
    form.FormClosed += Form_FormClosed;
    form.Shown += Form_Shown;
}

private void Form_Load(object sender, EventArgs e)
{
    // Exécuté avant que le formulaire ne soit affiché pour la première fois
    Console.WriteLine("Formulaire en cours de chargement");
}

private void Form_Shown(object sender, EventArgs e)
{
    // Exécuté après que le formulaire soit devenu visible
    Console.WriteLine("Formulaire affiché");
}

private void Form_FormClosing(object sender, FormClosingEventArgs e)
{
    // Exécuté avant la fermeture du formulaire
    // Permet d'annuler la fermeture
    DialogResult result = MessageBox.Show(
        "Voulez-vous vraiment quitter ?",
        "Confirmation",
        MessageBoxButtons.YesNo,
        MessageBoxIcon.Question);

    if (result == DialogResult.No)
    {
        e.Cancel = true; // Annule la fermeture
    }
}

private void Form_FormClosed(object sender, FormClosedEventArgs e)
{
    // Exécuté après la fermeture du formulaire
    Console.WriteLine($"Formulaire fermé, raison : {e.CloseReason}");
}
```


**Propriétés spécifiques aux formulaires :**
- `Text` : Texte affiché dans la barre de titre
- `Icon` : Icône affichée dans la barre de titre et la barre des tâches
- `FormBorderStyle` : Style de bordure (None, FixedSingle, Fixed3D, FixedDialog, Sizable, FixedToolWindow, SizableToolWindow)
- `StartPosition` : Position initiale (Manual, CenterScreen, WindowsDefaultLocation, WindowsDefaultBounds, CenterParent)
- `WindowState` : État initial (Normal, Minimized, Maximized)
- `ShowInTaskbar` : Indique si le formulaire apparaît dans la barre des tâches
- `TopMost` : Indique si le formulaire reste au-dessus des autres fenêtres
- `Opacity` : Niveau de transparence (0.0 à 1.0)
- `ShowIcon` : Indique si l'icône est affichée dans la barre de titre
- `MinimizeBox`/`MaximizeBox` : Indique si les boutons Réduire/Agrandir sont affichés
- `ControlBox` : Indique si la boîte de contrôle (avec les boutons de fermeture, etc.) est affichée
- `AcceptButton` : Bouton activé par la touche Entrée
- `CancelButton` : Bouton activé par la touche Échap
- `DialogResult` : Valeur retournée lorsque le formulaire est utilisé comme boîte de dialogue

## 7.3.3. Création de contrôles personnalisés

La création de contrôles personnalisés vous permet d'étendre les fonctionnalités des contrôles existants ou de créer des contrôles entièrement nouveaux.

### Extension d'un contrôle existant

L'approche la plus simple pour créer un contrôle personnalisé est d'étendre un contrôle existant.

```textmate
// .NET Framework 4.7.2 et .NET 8
public class TextBoxAvecValidation : TextBox
{
    // Couleurs pour les états valide/invalide
    private Color _couleurValide = Color.White;
    private Color _couleurInvalide = Color.MistyRose;

    // Propriété pour le texte de validation
    private string _texteValidation = string.Empty;
    public string TexteValidation
    {
        get => _texteValidation;
        set
        {
            _texteValidation = value;
            Valider(); // Valider à chaque changement du texte de validation
        }
    }

    // Événement personnalisé
    public event EventHandler<bool> ValidationChanged;

    // Propriété en lecture seule pour l'état de validation
    private bool _estValide = true;
    public bool EstValide => _estValide;

    // Couleurs personnalisables
    public Color CouleurValide
    {
        get => _couleurValide;
        set
        {
            _couleurValide = value;
            if (_estValide) BackColor = value;
        }
    }

    public Color CouleurInvalide
    {
        get => _couleurInvalide;
        set
        {
            _couleurInvalide = value;
            if (!_estValide) BackColor = value;
        }
    }

    // Expression régulière de validation
    private string _pattern = string.Empty;
    public string PatternValidation
    {
        get => _pattern;
        set
        {
            _pattern = value;
            Valider(); // Valider à chaque changement du pattern
        }
    }

    public TextBoxAvecValidation()
    {
        // Style par défaut
        BorderStyle = BorderStyle.FixedSingle;

        // Attacher l'événement TextChanged
        this.TextChanged += (s, e) => Valider();
    }

    // Méthode de validation
    private void Valider()
    {
        bool ancienEtat = _estValide;

        if (string.IsNullOrEmpty(Text))
        {
            // Considérer vide comme invalide si un texte de validation est défini
            _estValide = string.IsNullOrEmpty(_texteValidation);
        }
        else if (!string.IsNullOrEmpty(_pattern))
        {
            // Valider avec une expression régulière
            _estValide = System.Text.RegularExpressions.Regex.IsMatch(Text, _pattern);
        }
        else
        {
            // Par défaut, considérer comme valide
            _estValide = true;
        }

        // Mettre à jour l'apparence
        BackColor = _estValide ? _couleurValide : _couleurInvalide;

        // Déclencher l'événement si l'état a changé
        if (ancienEtat != _estValide)
        {
            ValidationChanged?.Invoke(this, _estValide);
        }
    }

    // Surcharger la méthode de création du contrôle pour initialiser l'état
    protected override void OnCreateControl()
    {
        base.OnCreateControl();
        Valider();
    }
}
```


Utilisation du contrôle personnalisé :

```textmate
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Créer et configurer le TextBox personnalisé
        TextBoxAvecValidation textBoxEmail = new TextBoxAvecValidation
        {
            Location = new Point(120, 50),
            Size = new Size(200, 23),
            PatternValidation = @"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$",
            TexteValidation = "Veuillez entrer une adresse email valide",
            CouleurValide = Color.LightGreen,
            CouleurInvalide = Color.LightPink
        };

        // Ajouter un Label pour indiquer le champ
        Label labelEmail = new Label
        {
            Text = "Email :",
            Location = new Point(50, 53),
            AutoSize = true
        };

        // Ajouter un Label pour afficher le message de validation
        Label labelValidation = new Label
        {
            Location = new Point(120, 76),
            Size = new Size(200, 20),
            ForeColor = Color.Red,
            Text = textBoxEmail.TexteValidation,
            Visible = !textBoxEmail.EstValide
        };

        // Gérer l'événement de validation
        textBoxEmail.ValidationChanged += (sender, estValide) =>
        {
            labelValidation.Visible = !estValide;
        };

        // Ajouter les contrôles au formulaire
        this.Controls.Add(labelEmail);
        this.Controls.Add(textBoxEmail);
        this.Controls.Add(labelValidation);
    }
}
```


### Création d'un contrôle à partir de zéro

Pour créer un contrôle entièrement personnalisé, vous pouvez hériter directement de la classe `Control`.

```textmate
// .NET Framework 4.7.2 et .NET 8
public class BoutonGradient : Control
{
    // Propriétés pour les couleurs du dégradé
    private Color _couleurDebut = Color.RoyalBlue;
    public Color CouleurDebut
    {
        get => _couleurDebut;
        set
        {
            _couleurDebut = value;
            Invalidate(); // Forcer le redessin
        }
    }

    private Color _couleurFin = Color.SkyBlue;
    public Color CouleurFin
    {
        get => _couleurFin;
        set
        {
            _couleurFin = value;
            Invalidate();
        }
    }

    // Arrondi des coins
    private int _rayonArrondi = 10;
    public int RayonArrondi
    {
        get => _rayonArrondi;
        set
        {
            _rayonArrondi = value;
            Invalidate();
        }
    }

    // Direction du dégradé
    public enum DirectionDegrade
    {
        Horizontal,
        Vertical,
        DiagonalHautGauche,
        DiagonalHautDroite
    }

    private DirectionDegrade _direction = DirectionDegrade.Horizontal;
    public DirectionDegrade Direction
    {
        get => _direction;
        set
        {
            _direction = value;
            Invalidate();
        }
    }

    // États du bouton
    private bool _survol = false;
    private bool _enfonce = false;

    public BoutonGradient()
    {
        // Définir la taille par défaut
        Size = new Size(120, 40);

        // Activer le double buffer pour éviter le scintillement
        SetStyle(ControlStyles.UserPaint, true);
        SetStyle(ControlStyles.AllPaintingInWmPaint, true);
        SetStyle(ControlStyles.OptimizedDoubleBuffer, true);
        SetStyle(ControlStyles.ResizeRedraw, true);
        SetStyle(ControlStyles.SupportsTransparentBackColor, true);

        // Définir la couleur de fond comme transparente
        BackColor = Color.Transparent;

        // Définir la police par défaut
        Font = new Font("Segoe UI", 9F, FontStyle.Bold);
    }

    // Gérer le dessin du contrôle
    protected override void OnPaint(PaintEventArgs e)
    {
        base.OnPaint(e);

        Graphics g = e.Graphics;
        g.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;

        // Créer un rectangle arrondi
        System.Drawing.Drawing2D.GraphicsPath path = new System.Drawing.Drawing2D.GraphicsPath();
        Rectangle rect = new Rectangle(0, 0, Width - 1, Height - 1);

        // Dessiner le rectangle arrondi
        if (_rayonArrondi > 0)
        {
            int diameter = _rayonArrondi * 2;
            Rectangle arc = new Rectangle(rect.Location, new Size(diameter, diameter));

            // Coin supérieur gauche
            path.AddArc(arc, 180, 90);

            // Coin supérieur droit
            arc.X = rect.Right - diameter;
            path.AddArc(arc, 270, 90);

            // Coin inférieur droit
            arc.Y = rect.Bottom - diameter;
            path.AddArc(arc, 0, 90);

            // Coin inférieur gauche
            arc.X = rect.Left;
            path.AddArc(arc, 90, 90);

            path.CloseFigure();
        }
        else
        {
            path.AddRectangle(rect);
        }

        // Créer le dégradé
        System.Drawing.Drawing2D.LinearGradientBrush brush;

        // Ajuster les couleurs selon l'état
        Color couleurDebut = _couleurDebut;
        Color couleurFin = _couleurFin;

        if (_enfonce)
        {
            // Inverser les couleurs si le bouton est enfoncé
            couleurDebut = _couleurFin;
            couleurFin = _couleurDebut;
        }
        else if (_survol)
        {
            // Éclaircir les couleurs au survol
            couleurDebut = EclaircirCouleur(_couleurDebut, 30);
            couleurFin = EclaircirCouleur(_couleurFin, 30);
        }

        // Créer le pinceau selon la direction
        switch (_direction)
        {
            case DirectionDegrade.Horizontal:
                brush = new System.Drawing.Drawing2D.LinearGradientBrush(
                    rect, couleurDebut, couleurFin, System.Drawing.Drawing2D.LinearGradientMode.Horizontal);
                break;

            case DirectionDegrade.Vertical:
                brush = new System.Drawing.Drawing2D.LinearGradientBrush(
                    rect, couleurDebut, couleurFin, System.Drawing.Drawing2D.LinearGradientMode.Vertical);
                break;

            case DirectionDegrade.DiagonalHautGauche:
                brush = new System.Drawing.Drawing2D.LinearGradientBrush(
                    rect, couleurDebut, couleurFin, System.Drawing.Drawing2D.LinearGradientMode.ForwardDiagonal);
                break;

            case DirectionDegrade.DiagonalHautDroite:
                brush = new System.Drawing.Drawing2D.LinearGradientBrush(
                    rect, couleurDebut, couleurFin, System.Drawing.Drawing2D.LinearGradientMode.BackwardDiagonal);
                break;

            default:
                brush = new System.Drawing.Drawing2D.LinearGradientBrush(
                    rect, couleurDebut, couleurFin, System.Drawing.Drawing2D.LinearGradientMode.Horizontal);
                break;
        }

        // Remplir le fond
        g.FillPath(brush, path);

        // Dessiner la bordure
        using (Pen pen = new Pen(Color.FromArgb(100, Color.Gray), 1))
        {
            g.DrawPath(pen, path);
        }

        // Dessiner le texte
        StringFormat sf = new StringFormat
        {
            Alignment = StringAlignment.Center,
            LineAlignment = StringAlignment.Center
        };

        // Calculer l'ombre du texte
        if (!_enfonce)
        {
            using (SolidBrush shadowBrush = new SolidBrush(Color.FromArgb(70, Color.Black)))
            {
                Rectangle shadowRect = new Rectangle(2, 2, Width, Height);
                g.DrawString(Text, Font, shadowBrush, shadowRect, sf);
            }
        }

        // Dessiner le texte principal
        using (SolidBrush textBrush = new SolidBrush(ForeColor))
        {
            Rectangle textRect = rect;

            // Décaler légèrement le texte si le bouton est enfoncé
            if (_enfonce)
            {
                textRect.Offset(1, 1);
            }

            g.DrawString(Text, Font, textBrush, textRect, sf);
        }

        // Libérer les ressources
        brush.Dispose();
        path.Dispose();
    }

    // Éclaircir une couleur d'un pourcentage donné
    private Color EclaircirCouleur(Color couleur, int pourcentage)
    {
        float facteur = pourcentage / 100f;
        int r = (int)((255 - couleur.R) * facteur) + couleur.R;
        int g = (int)((255 - couleur.G) * facteur) + couleur.G;
        int b = (int)((255 - couleur.B) * facteur) + couleur.B;

        return Color.FromArgb(couleur.A, r, g, b);
    }

    // Gérer les événements de la souris
    protected override void OnMouseEnter(EventArgs e)
    {
        _survol = true;
        Invalidate();
        base.OnMouseEnter(e);
    }

    protected override void OnMouseLeave(EventArgs e)
    {
        _survol = false;
        Invalidate();
        base.OnMouseLeave(e);
    }

    protected override void OnMouseDown(MouseEventArgs e)
    {
        if (e.Button == MouseButtons.Left)
        {
            _enfonce = true;
            Invalidate();
        }
        base.OnMouseDown(e);
    }

    protected override void OnMouseUp(MouseEventArgs e)
    {
        _enfonce = false;
        Invalidate();
        base.OnMouseUp(e);
    }
}
```


Utilisation du contrôle personnalisé :

```textmate
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Créer et configurer le bouton personnalisé
        BoutonGradient bouton = new BoutonGradient
        {
            Location = new Point(50, 50),
            Size = new Size(150, 50),
            Text = "Bouton Dégradé",
            CouleurDebut = Color.RoyalBlue,
            CouleurFin = Color.LightSkyBlue,
            Direction = BoutonGradient.DirectionDegrade.DiagonalHautGauche,
            RayonArrondi = 15,
            ForeColor = Color.White,
            Font = new Font("Segoe UI", 10F, FontStyle.Bold)
        };

        // Attacher un gestionnaire d'événements
        bouton.Click += (sender, e) => MessageBox.Show("Bouton cliqué !");

        // Ajouter au formulaire
        this.Controls.Add(bouton);
    }
}
```


### Création d'un UserControl

Les UserControls sont des contrôles composites qui regroupent plusieurs contrôles standard dans une seule unité réutilisable.

```textmate
// .NET Framework 4.7.2 et .NET 8
// Ajouter un nouveau fichier "ChampRecherche.cs" de type "User Control"
public partial class ChampRecherche : UserControl
{
    // Événement personnalisé pour la recherche
    public event EventHandler<string> Rechercher;

    // Propriétés
    public string TexteRecherche
    {
        get => textBoxRecherche.Text;
        set => textBoxRecherche.Text = value;
    }

    public string Placeholder
    {
        get => textBoxRecherche.PlaceholderText;
        set => textBoxRecherche.PlaceholderText = value;
    }

    public bool AfficherAnnuler
    {
        get => buttonAnnuler.Visible;
        set => buttonAnnuler.Visible = value;
    }

    public ChampRecherche()
    {
        InitializeComponent();

        // Configuration des événements
        buttonRechercher.Click += ButtonRechercher_Click;
        buttonAnnuler.Click += ButtonAnnuler_Click;
        textBoxRecherche.KeyPress += TextBoxRecherche_KeyPress;
    }

    // Les méthodes d'initialisation du concepteur seront générées automatiquement,
    // mais voici comment elles pourraient être implémentées manuellement :
    private void InitializeComponent()
    {
        // Créer les contrôles
        this.textBoxRecherche = new TextBox();
        this.buttonRechercher = new Button();
        this.buttonAnnuler = new Button();

        // Configurer le TextBox
        this.textBoxRecherche.Anchor = AnchorStyles.Left | AnchorStyles.Right | AnchorStyles.Top;
        this.textBoxRecherche.Location = new Point(0, 0);
        this.textBoxRecherche.Name = "textBoxRecherche";
        this.textBoxRecherche.Size = new Size(200, 23);
        this.textBoxRecherche.TabIndex = 0;

        // En .NET 5+ uniquement
        if (Environment.Version.Major >= 5)
        {
            this.textBoxRecherche.PlaceholderText = "Rechercher...";
        }

        // Configurer le bouton de recherche
        this.buttonRechercher.Anchor = AnchorStyles.Top | AnchorStyles.Right;
        this.buttonRechercher.Location = new Point(206, 0);
        this.buttonRechercher.Name = "buttonRechercher";
        this.buttonRechercher.Size = new Size(30, 23);
        this.buttonRechercher.TabIndex = 1;
        this.buttonRechercher.Text = "🔍";
        this.buttonRechercher.UseVisualStyleBackColor = true;

        // Configurer le bouton d'annulation
        this.buttonAnnuler.Anchor = AnchorStyles.Top | AnchorStyles.Right;
        this.buttonAnnuler.Location = new Point(242, 0);
        this.buttonAnnuler.Name = "buttonAnnuler";
        this.buttonAnnuler.Size = new Size(30, 23);
        this.buttonAnnuler.TabIndex = 2;
        this.buttonAnnuler.Text = "✖";
        this.buttonAnnuler.UseVisualStyleBackColor = true;
        this.buttonAnnuler.Visible = false;

        // Configurer le UserControl
        this.Controls.Add(this.buttonAnnuler);
        this.Controls.Add(this.buttonRechercher);
        this.Controls.Add(this.textBoxRecherche);
        this.Name = "ChampRecherche";
        this.Size = new Size(272, 23);
    }

    private TextBox textBoxRecherche;
    private Button buttonRechercher;
    private Button buttonAnnuler;

    private void ButtonRechercher_Click(object sender, EventArgs e)
    {
        LancerRecherche();
    }

    private void ButtonAnnuler_Click(object sender, EventArgs e)
    {
        textBoxRecherche.Clear();
        buttonAnnuler.Visible = false;
        textBoxRecherche.Focus();
    }

    private void TextBoxRecherche_KeyPress(object sender, KeyPressEventArgs e)
    {
        if (e.KeyChar == (char)Keys.Enter)
        {
            e.Handled = true;
            LancerRecherche();
        }
    }

    private void LancerRecherche()
    {
        string texte = textBoxRecherche.Text.Trim();

        if (!string.IsNullOrEmpty(texte))
        {
            // Afficher le bouton d'annulation
            buttonAnnuler.Visible = true;

            // Déclencher l'événement de recherche
            Rechercher?.Invoke(this, texte);
        }
        else
        {
            buttonAnnuler.Visible = false;
        }
    }

    // Méthode publique pour effacer le champ
    public void Effacer()
    {
        textBoxRecherche.Clear();
        buttonAnnuler.Visible = false;
    }
}
```


Utilisation du UserControl :

```textmate
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Créer et configurer le contrôle de recherche
        ChampRecherche champRecherche = new ChampRecherche
        {
            Location = new Point(20, 20),
            Size = new Size(300, 23),
            Placeholder = "Rechercher un produit...",
            Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
        };

        // Attacher un gestionnaire d'événements
        champRecherche.Rechercher += (sender, texte) =>
        {
            MessageBox.Show($"Recherche de : {texte}");
            // Effectuer la recherche et afficher les résultats
        };

        // Ajouter au formulaire
        this.Controls.Add(champRecherche);
    }
}
```


### Gestion de l'accessibilité des contrôles personnalisés

Pour rendre vos contrôles personnalisés accessibles aux technologies d'assistance, il est important d'implémenter correctement les fonctionnalités d'accessibilité.

```textmate
// .NET Framework 4.7.2 et .NET 8
public class BoutonAccessible : Button
{
    public BoutonAccessible()
    {
        // Définir les propriétés d'accessibilité par défaut
        AccessibleRole = AccessibleRole.PushButton;
        AccessibleName = "Bouton accessible";
        AccessibleDescription = "Un bouton avec des fonctionnalités d'accessibilité améliorées";
    }

    // Surcharger pour fournir des informations d'accessibilité
    protected override AccessibleObject CreateAccessibilityInstance()
    {
        return new BoutonAccessibleObject(this);
    }

    // Classe interne pour la gestion de l'accessibilité
    protected class BoutonAccessibleObject : ButtonBaseAccessibleObject
    {
        private BoutonAccessible _owner;

        public BoutonAccessibleObject(BoutonAccessible owner)
            : base(owner)
        {
            _owner = owner;
        }

        // Surcharger pour fournir un nom d'accessibilité personnalisé
        public override string Name
        {
            get
            {
                // Si l'utilisateur a défini un nom d'accessibilité, l'utiliser
                if (!string.IsNullOrEmpty(Owner.AccessibleName))
                {
                    return Owner.AccessibleName;
                }

                // Sinon, utiliser le texte du bouton
                return _owner.Text;
            }
        }

        // Surcharger pour fournir une description d'accessibilité personnalisée
        public override string Description
        {
            get
            {
                // Si l'utilisateur a défini une description d'accessibilité, l'utiliser
                if (!string.IsNullOrEmpty(Owner.AccessibleDescription))
                {
                    return Owner.AccessibleDescription;
                }

                // Sinon, utiliser le texte du bouton avec des informations supplémentaires
                return $"Bouton : {_owner.Text}";
            }
        }

        // Surcharger pour fournir l'état d'accessibilité
        public override AccessibleStates State
        {
            get
            {
                AccessibleStates state = base.State;

                if (!_owner.Enabled)
                {
                    state |= AccessibleStates.Unavailable;
                }

                if (_owner.Focused)
                {
                    state |= AccessibleStates.Focused;
                }

                return state;
            }
        }

        // Surcharger pour fournir les actions d'accessibilité
        public override void DoDefaultAction()
        {
            // Simuler un clic sur le bouton
            _owner.PerformClick();
        }
    }
}
```

## 7.3.4. Contrôles composites

Les contrôles composites sont des contrôles qui contiennent plusieurs contrôles enfants. Ils peuvent être créés soit en utilisant des UserControls, soit en regroupant des contrôles existants dans des conteneurs.

### Utilisation des Panels et GroupBoxes

Les Panels et GroupBoxes sont des contrôles conteneurs qui vous permettent d'organiser d'autres contrôles.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeCompositeControls()
{
    // Créer un GroupBox
    GroupBox groupBoxContact = new GroupBox
    {
        Text = "Informations de contact",
        Location = new Point(20, 20),
        Size = new Size(400, 200),
        Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
    };

    // Ajouter des Labels et TextBoxes au GroupBox
    Label labelNom = new Label
    {
        Text = "Nom :",
        Location = new Point(20, 30),
        AutoSize = true
    };

    TextBox textBoxNom = new TextBox
    {
        Location = new Point(120, 27),
        Size = new Size(250, 23),
        Anchor = AnchorStyles.Left | AnchorStyles.Right
    };

    Label labelPrenom = new Label
    {
        Text = "Prénom :",
        Location = new Point(20, 60),
        AutoSize = true
    };

    TextBox textBoxPrenom = new TextBox
    {
        Location = new Point(120, 57),
        Size = new Size(250, 23),
        Anchor = AnchorStyles.Left | AnchorStyles.Right
    };

    Label labelEmail = new Label
    {
        Text = "Email :",
        Location = new Point(20, 90),
        AutoSize = true
    };

    TextBox textBoxEmail = new TextBox
    {
        Location = new Point(120, 87),
        Size = new Size(250, 23),
        Anchor = AnchorStyles.Left | AnchorStyles.Right
    };

    Label labelTelephone = new Label
    {
        Text = "Téléphone :",
        Location = new Point(20, 120),
        AutoSize = true
    };

    TextBox textBoxTelephone = new TextBox
    {
        Location = new Point(120, 117),
        Size = new Size(250, 23),
        Anchor = AnchorStyles.Left | AnchorStyles.Right
    };

    Button buttonEnregistrer = new Button
    {
        Text = "Enregistrer",
        Location = new Point(295, 160),
        Size = new Size(75, 23),
        Anchor = AnchorStyles.Right
    };

    // Ajouter les contrôles au GroupBox
    groupBoxContact.Controls.Add(labelNom);
    groupBoxContact.Controls.Add(textBoxNom);
    groupBoxContact.Controls.Add(labelPrenom);
    groupBoxContact.Controls.Add(textBoxPrenom);
    groupBoxContact.Controls.Add(labelEmail);
    groupBoxContact.Controls.Add(textBoxEmail);
    groupBoxContact.Controls.Add(labelTelephone);
    groupBoxContact.Controls.Add(textBoxTelephone);
    groupBoxContact.Controls.Add(buttonEnregistrer);

    // Créer un Panel pour regrouper d'autres contrôles
    Panel panelOptions = new Panel
    {
        Location = new Point(20, 240),
        Size = new Size(400, 150),
        BorderStyle = BorderStyle.FixedSingle,
        Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
    };

    // Ajouter des CheckBoxes au Panel
    CheckBox checkBoxNewsletter = new CheckBox
    {
        Text = "S'inscrire à la newsletter",
        Location = new Point(20, 20),
        AutoSize = true
    };

    CheckBox checkBoxPromo = new CheckBox
    {
        Text = "Recevoir des offres promotionnelles",
        Location = new Point(20, 50),
        AutoSize = true
    };

    // Ajouter des RadioButtons au Panel
    RadioButton radioButtonEmail = new RadioButton
    {
        Text = "Préférence de contact par email",
        Location = new Point(20, 80),
        AutoSize = true,
        Checked = true
    };

    RadioButton radioButtonTelephone = new RadioButton
    {
        Text = "Préférence de contact par téléphone",
        Location = new Point(20, 110),
        AutoSize = true
    };

    // Ajouter les contrôles au Panel
    panelOptions.Controls.Add(checkBoxNewsletter);
    panelOptions.Controls.Add(checkBoxPromo);
    panelOptions.Controls.Add(radioButtonEmail);
    panelOptions.Controls.Add(radioButtonTelephone);

    // Ajouter les conteneurs au formulaire
    this.Controls.Add(groupBoxContact);
    this.Controls.Add(panelOptions);
}
```


### Création d'un contrôle composite avancé

Créons un contrôle composite plus avancé sous forme de UserControl pour la sélection d'une adresse.

```textmate
// .NET Framework 4.7.2 et .NET 8
// Ajouter un nouveau fichier "AdresseControl.cs" de type "User Control"
public partial class AdresseControl : UserControl
{
    // Propriétés pour accéder aux valeurs
    public string Adresse
    {
        get => textBoxAdresse.Text;
        set => textBoxAdresse.Text = value;
    }

    public string CodePostal
    {
        get => textBoxCodePostal.Text;
        set => textBoxCodePostal.Text = value;
    }

    public string Ville
    {
        get => textBoxVille.Text;
        set => textBoxVille.Text = value;
    }

    public string Pays
    {
        get => comboBoxPays.SelectedItem?.ToString() ?? "";
        set
        {
            int index = comboBoxPays.Items.IndexOf(value);
            if (index >= 0)
            {
                comboBoxPays.SelectedIndex = index;
            }
            else if (comboBoxPays.Items.Count > 0)
            {
                comboBoxPays.SelectedIndex = 0;
            }
        }
    }

    // Événements personnalisés
    public event EventHandler AdresseChanged;

    private bool _validationEnabled = true;
    public bool ValidationEnabled
    {
        get => _validationEnabled;
        set
        {
            _validationEnabled = value;
            ValiderChamps();
        }
    }

    public AdresseControl()
    {
        InitializeComponent();

        // Initialiser les pays
        InitialiserPays();

        // Attacher les gestionnaires d'événements pour la validation
        textBoxAdresse.TextChanged += TextBox_TextChanged;
        textBoxCodePostal.TextChanged += TextBox_TextChanged;
        textBoxVille.TextChanged += TextBox_TextChanged;
        comboBoxPays.SelectedIndexChanged += ComboBox_SelectedIndexChanged;

        // Effectuer une validation initiale
        ValiderChamps();
    }

    // Les méthodes d'initialisation du concepteur seront générées automatiquement,
    // mais voici comment elles pourraient être implémentées manuellement :
    private void InitializeComponent()
    {
        // Créer les contrôles
        this.tableLayoutPanel = new TableLayoutPanel();
        this.labelAdresse = new Label();
        this.textBoxAdresse = new TextBox();
        this.labelCodePostal = new Label();
        this.textBoxCodePostal = new TextBox();
        this.labelVille = new Label();
        this.textBoxVille = new TextBox();
        this.labelPays = new Label();
        this.comboBoxPays = new ComboBox();
        this.buttonValider = new Button();
        this.errorProvider = new ErrorProvider();

        // TableLayoutPanel
        this.tableLayoutPanel.ColumnCount = 2;
        this.tableLayoutPanel.RowCount = 5;
        this.tableLayoutPanel.Dock = DockStyle.Fill;
        this.tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 30F));
        this.tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 70F));
        for (int i = 0; i < 5; i++)
        {
            this.tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Absolute, 30F));
        }

        // Adresse
        this.labelAdresse.Text = "Adresse :";
        this.labelAdresse.Anchor = AnchorStyles.Left | AnchorStyles.Right;
        this.textBoxAdresse.Dock = DockStyle.Fill;

        // Code postal
        this.labelCodePostal.Text = "Code postal :";
        this.labelCodePostal.Anchor = AnchorStyles.Left | AnchorStyles.Right;
        this.textBoxCodePostal.Dock = DockStyle.Fill;

        // Ville
        this.labelVille.Text = "Ville :";
        this.labelVille.Anchor = AnchorStyles.Left | AnchorStyles.Right;
        this.textBoxVille.Dock = DockStyle.Fill;

        // Pays
        this.labelPays.Text = "Pays :";
        this.labelPays.Anchor = AnchorStyles.Left | AnchorStyles.Right;
        this.comboBoxPays.Dock = DockStyle.Fill;
        this.comboBoxPays.DropDownStyle = ComboBoxStyle.DropDownList;

        // Bouton Valider
        this.buttonValider.Text = "Valider";
        this.buttonValider.Click += ButtonValider_Click;

        // Ajouter les contrôles au TableLayoutPanel
        this.tableLayoutPanel.Controls.Add(this.labelAdresse, 0, 0);
        this.tableLayoutPanel.Controls.Add(this.textBoxAdresse, 1, 0);
        this.tableLayoutPanel.Controls.Add(this.labelCodePostal, 0, 1);
        this.tableLayoutPanel.Controls.Add(this.textBoxCodePostal, 1, 1);
        this.tableLayoutPanel.Controls.Add(this.labelVille, 0, 2);
        this.tableLayoutPanel.Controls.Add(this.textBoxVille, 1, 2);
        this.tableLayoutPanel.Controls.Add(this.labelPays, 0, 3);
        this.tableLayoutPanel.Controls.Add(this.comboBoxPays, 1, 3);
        this.tableLayoutPanel.Controls.Add(this.buttonValider, 1, 4);

        // Configurer le UserControl
        this.Controls.Add(this.tableLayoutPanel);
        this.Name = "AdresseControl";
        this.Size = new Size(400, 150);
    }

    private TableLayoutPanel tableLayoutPanel;
    private Label labelAdresse;
    private TextBox textBoxAdresse;
    private Label labelCodePostal;
    private TextBox textBoxCodePostal;
    private Label labelVille;
    private TextBox textBoxVille;
    private Label labelPays;
    private ComboBox comboBoxPays;
    private Button buttonValider;
    private ErrorProvider errorProvider;

    private void InitialiserPays()
    {
        // Liste des pays
        string[] pays = new string[]
        {
            "France",
            "Belgique",
            "Suisse",
            "Canada",
            "Luxembourg",
            "Allemagne",
            "Italie",
            "Espagne",
            "Royaume-Uni",
            "États-Unis"
        };

        // Ajouter les pays à la ComboBox
        comboBoxPays.Items.AddRange(pays);

        // Sélectionner la France par défaut
        if (comboBoxPays.Items.Count > 0)
        {
            comboBoxPays.SelectedIndex = 0;
        }
    }

    private void TextBox_TextChanged(object sender, EventArgs e)
    {
        ValiderChamps();
        NotifierChangementAdresse();
    }

    private void ComboBox_SelectedIndexChanged(object sender, EventArgs e)
    {
        ValiderChamps();
        NotifierChangementAdresse();
    }

    private void ButtonValider_Click(object sender, EventArgs e)
    {
        if (ValiderChamps())
        {
            MessageBox.Show("Adresse valide !", "Validation", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show("Veuillez corriger les erreurs avant de continuer.", "Validation", MessageBoxButtons.OK, MessageBoxIcon.Warning);
        }
    }

    private bool ValiderChamps()
    {
        if (!_validationEnabled)
        {
            errorProvider.Clear();
            return true;
        }

        bool estValide = true;

        // Valider l'adresse
        if (string.IsNullOrWhiteSpace(textBoxAdresse.Text))
        {
            errorProvider.SetError(textBoxAdresse, "L'adresse est requise");
            estValide = false;
        }
        else
        {
            errorProvider.SetError(textBoxAdresse, "");
        }

        // Valider le code postal
        string codePostal = textBoxCodePostal.Text.Trim();
        string paysSelectionne = comboBoxPays.SelectedItem?.ToString() ?? "";
        bool codePostalValide = true;

        if (string.IsNullOrWhiteSpace(codePostal))
        {
            errorProvider.SetError(textBoxCodePostal, "Le code postal est requis");
            estValide = false;
            codePostalValide = false;
        }
        else
        {
            // Valider selon le pays sélectionné
            switch (paysSelectionne)
            {
                case "France":
                    // Code postal français : 5 chiffres
                    if (!System.Text.RegularExpressions.Regex.IsMatch(codePostal, @"^\d{5}$"))
                    {
                        errorProvider.SetError(textBoxCodePostal, "Le code postal français doit contenir 5 chiffres");
                        estValide = false;
                        codePostalValide = false;
                    }
                    break;

                case "Belgique":
                    // Code postal belge : 4 chiffres
                    if (!System.Text.RegularExpressions.Regex.IsMatch(codePostal, @"^\d{4}$"))
                    {
                        errorProvider.SetError(textBoxCodePostal, "Le code postal belge doit contenir 4 chiffres");
                        estValide = false;
                        codePostalValide = false;
                    }
                    break;

                case "Suisse":
                    // Code postal suisse : 4 chiffres
                    if (!System.Text.RegularExpressions.Regex.IsMatch(codePostal, @"^\d{4}$"))
                    {
                        errorProvider.SetError(textBoxCodePostal, "Le code postal suisse doit contenir 4 chiffres");
                        estValide = false;
                        codePostalValide = false;
                    }
                    break;

                case "Canada":
                    // Code postal canadien : format A1A 1A1
                    if (!System.Text.RegularExpressions.Regex.IsMatch(codePostal, @"^[A-Za-z]\d[A-Za-z]\s?\d[A-Za-z]\d$"))
                    {
                        errorProvider.SetError(textBoxCodePostal, "Le code postal canadien doit être au format A1A 1A1");
                        estValide = false;
                        codePostalValide = false;
                    }
                    break;

                default:
                    // Pour les autres pays, vérifier simplement que le code postal n'est pas vide
                    if (string.IsNullOrWhiteSpace(codePostal))
                    {
                        errorProvider.SetError(textBoxCodePostal, "Le code postal est requis");
                        estValide = false;
                        codePostalValide = false;
                    }
                    break;
            }
        }

        if (codePostalValide)
        {
            errorProvider.SetError(textBoxCodePostal, "");
        }

        // Valider la ville
        if (string.IsNullOrWhiteSpace(textBoxVille.Text))
        {
            errorProvider.SetError(textBoxVille, "La ville est requise");
            estValide = false;
        }
        else
        {
            errorProvider.SetError(textBoxVille, "");
        }

        // Valider le pays
        if (comboBoxPays.SelectedIndex < 0)
        {
            errorProvider.SetError(comboBoxPays, "Le pays est requis");
            estValide = false;
        }
        else
        {
            errorProvider.SetError(comboBoxPays, "");
        }

        return estValide;
    }

    private void NotifierChangementAdresse()
    {
        AdresseChanged?.Invoke(this, EventArgs.Empty);
    }

    // Méthode publique pour récupérer l'adresse complète
    public string GetAdresseComplete()
    {
        return $"{Adresse}\n{CodePostal} {Ville}\n{Pays}";
    }

    // Méthode publique pour définir l'adresse complète
    public void SetAdresseComplete(string adresse, string codePostal, string ville, string pays)
    {
        Adresse = adresse;
        CodePostal = codePostal;
        Ville = ville;
        Pays = pays;
    }
}
```


Utilisation du contrôle composite :

```textmate
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Créer et configurer le contrôle d'adresse
        AdresseControl adresseControl = new AdresseControl
        {
            Location = new Point(20, 20),
            Size = new Size(400, 150),
            Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
        };

        // Définir une adresse par défaut
        adresseControl.SetAdresseComplete(
            "123 rue de la République",
            "75001",
            "Paris",
            "France"
        );

        // Attacher un gestionnaire d'événements
        adresseControl.AdresseChanged += (sender, e) =>
        {
            // Mettre à jour d'autres parties de l'application
            this.Text = $"Adresse : {adresseControl.Ville}, {adresseControl.Pays}";
        };

        // Ajouter au formulaire
        this.Controls.Add(adresseControl);

        // Ajouter un bouton pour afficher l'adresse complète
        Button buttonAfficher = new Button
        {
            Text = "Afficher l'adresse",
            Location = new Point(20, 180),
            Size = new Size(150, 30)
        };

        buttonAfficher.Click += (sender, e) =>
        {
            MessageBox.Show(
                adresseControl.GetAdresseComplete(),
                "Adresse complète",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information
            );
        };

        this.Controls.Add(buttonAfficher);
    }
}
```


### Contrôles composites avancés avec des conteneurs spécialisés

Les contrôles comme FlowLayoutPanel et TableLayoutPanel sont particulièrement utiles pour créer des interfaces utilisateur dynamiques.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void InitializeAdvancedLayoutControls()
{
    // Créer un FlowLayoutPanel pour une disposition dynamique
    FlowLayoutPanel flowLayoutPanel = new FlowLayoutPanel
    {
        Location = new Point(20, 20),
        Size = new Size(300, 150),
        FlowDirection = FlowDirection.LeftToRight,
        WrapContents = true,
        BorderStyle = BorderStyle.FixedSingle,
        AutoScroll = true,
        Padding = new Padding(5),
        Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
    };

    // Ajouter plusieurs boutons au FlowLayoutPanel
    for (int i = 1; i <= 10; i++)
    {
        Button button = new Button
        {
            Text = $"Bouton {i}",
            Size = new Size(80, 30),
            Margin = new Padding(5)
        };

        button.Click += (sender, e) =>
        {
            Button clickedButton = (Button)sender;
            MessageBox.Show($"Vous avez cliqué sur {clickedButton.Text}");
        };

        flowLayoutPanel.Controls.Add(button);
    }

    // Créer un TableLayoutPanel pour une disposition en grille
    TableLayoutPanel tableLayoutPanel = new TableLayoutPanel
    {
        Location = new Point(20, 180),
        Size = new Size(300, 200),
        ColumnCount = 3,
        RowCount = 3,
        BorderStyle = BorderStyle.FixedSingle,
        CellBorderStyle = TableLayoutPanelCellBorderStyle.Single,
        Padding = new Padding(3),
        Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right
    };

    // Définir les styles de colonnes et de lignes
    tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 33.33F));
    tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 33.33F));
    tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 33.34F));

    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 33.33F));
    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 33.33F));
    tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 33.34F));

    // Ajouter des contrôles au TableLayoutPanel
    for (int row = 0; row < 3; row++)
    {
        for (int col = 0; col < 3; col++)
        {
            if (row == 0 && col == 0)
            {
                // Ajouter un Label dans la première cellule
                Label label = new Label
                {
                    Text = "Produit",
                    TextAlign = ContentAlignment.MiddleCenter,
                    Dock = DockStyle.Fill,
                    Font = new Font("Segoe UI", 9F, FontStyle.Bold)
                };
                tableLayoutPanel.Controls.Add(label, col, row);
            }
            else if (row == 0 && col == 1)
            {
                // Ajouter un Label dans la deuxième cellule
                Label label = new Label
                {
                    Text = "Quantité",
                    TextAlign = ContentAlignment.MiddleCenter,
                    Dock = DockStyle.Fill,
                    Font = new Font("Segoe UI", 9F, FontStyle.Bold)
                };
                tableLayoutPanel.Controls.Add(label, col, row);
            }
            else if (row == 0 && col == 2)
            {
                // Ajouter un Label dans la troisième cellule
                Label label = new Label
                {
                    Text = "Prix",
                    TextAlign = ContentAlignment.MiddleCenter,
                    Dock = DockStyle.Fill,
                    Font = new Font("Segoe UI", 9F, FontStyle.Bold)
                };
                tableLayoutPanel.Controls.Add(label, col, row);
            }
            else if (col == 0)
            {
                // Ajouter des noms de produits dans la première colonne
                ComboBox comboBox = new ComboBox
                {
                    Dock = DockStyle.Fill,
                    DropDownStyle = ComboBoxStyle.DropDownList
                };

                comboBox.Items.AddRange(new string[] { "Produit A", "Produit B", "Produit C" });
                if (comboBox.Items.Count > 0)
                {
                    comboBox.SelectedIndex = 0;
                }

                tableLayoutPanel.Controls.Add(comboBox, col, row);
            }
            else if (col == 1)
            {
                // Ajouter des NumericUpDown pour les quantités dans la deuxième colonne
                NumericUpDown numericUpDown = new NumericUpDown
                {
                    Dock = DockStyle.Fill,
                    Minimum = 1,
                    Maximum = 100,
                    Value = 1
                };

                tableLayoutPanel.Controls.Add(numericUpDown, col, row);
            }
            else if (col == 2)
            {
                // Ajouter des TextBox pour les prix dans la troisième colonne
                TextBox textBox = new TextBox
                {
                    Dock = DockStyle.Fill,
                    Text = "0,00 €",
                    TextAlign = HorizontalAlignment.Right
                };

                tableLayoutPanel.Controls.Add(textBox, col, row);
            }
        }
    }

    // Ajouter un SplitContainer pour diviser l'écran en deux parties
    SplitContainer splitContainer = new SplitContainer
    {
        Location = new Point(20, 390),
        Size = new Size(300, 200),
        Orientation = Orientation.Horizontal,
        SplitterDistance = 100,
        Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right | AnchorStyles.Bottom
    };

    // Configurer le panneau supérieur
    TextBox textBoxSource = new TextBox
    {
        Multiline = true,
        Dock = DockStyle.Fill,
        ScrollBars = ScrollBars.Vertical,
        Text = "Entrez du texte ici..."
    };

    splitContainer.Panel1.Controls.Add(textBoxSource);

    // Configurer le panneau inférieur
    TextBox textBoxPreview = new TextBox
    {
        Multiline = true,
        Dock = DockStyle.Fill,
        ScrollBars = ScrollBars.Vertical,
        ReadOnly = true
    };

    splitContainer.Panel2.Controls.Add(textBoxPreview);

    // Synchroniser les deux TextBox
    textBoxSource.TextChanged += (sender, e) =>
    {
        textBoxPreview.Text = textBoxSource.Text.ToUpper();
    };

    // Déclencher l'événement pour initialiser le texte
    textBoxSource.Text = "Entrez du texte ici...";

    // Ajouter les contrôles au formulaire
    this.Controls.Add(flowLayoutPanel);
    this.Controls.Add(tableLayoutPanel);
    this.Controls.Add(splitContainer);
}
```


Ces exemples illustrent les contrôles de base de Windows Forms, leurs propriétés principales, et comment créer des contrôles personnalisés pour répondre à des besoins spécifiques. En comprenant ces concepts, vous serez en mesure de développer des interfaces utilisateur riches et interactives dans vos applications Windows Forms.

⏭️ 7.4. [Gestion des événements](/07-windows-forms-winforms/7-4-gestion-des-evenements.md)
