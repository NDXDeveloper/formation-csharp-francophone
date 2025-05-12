# 7.6. Menus et barres d'outils

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les menus et barres d'outils constituent des éléments essentiels de toute interface graphique moderne, offrant à l'utilisateur un accès organisé aux différentes fonctionnalités de l'application. Windows Forms propose plusieurs contrôles permettant d'implémenter ces composants d'interface de manière simple et efficace.

## 7.6.1. MenuStrip

`MenuStrip` est le contrôle principal pour créer des menus dans une application Windows Forms. Il remplace le contrôle `MainMenu` utilisé dans les versions antérieures de .NET Framework.

### Création d'un menu de base

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerMenuPrincipal()
{
    // Création du contrôle MenuStrip
    MenuStrip menuPrincipal = new MenuStrip();
    menuPrincipal.Dock = DockStyle.Top;

    // Création des éléments de menu principaux
    ToolStripMenuItem menuFichier = new ToolStripMenuItem("&Fichier");
    ToolStripMenuItem menuEdition = new ToolStripMenuItem("&Edition");
    ToolStripMenuItem menuAffichage = new ToolStripMenuItem("&Affichage");
    ToolStripMenuItem menuOutils = new ToolStripMenuItem("&Outils");
    ToolStripMenuItem menuAide = new ToolStripMenuItem("&Aide");

    // Ajout des éléments principaux au menu
    menuPrincipal.Items.Add(menuFichier);
    menuPrincipal.Items.Add(menuEdition);
    menuPrincipal.Items.Add(menuAffichage);
    menuPrincipal.Items.Add(menuOutils);
    menuPrincipal.Items.Add(menuAide);

    // Création des sous-menus pour le menu Fichier
    ToolStripMenuItem menuNouveau = new ToolStripMenuItem("&Nouveau", null, MenuNouveau_Click);
    menuNouveau.ShortcutKeys = Keys.Control | Keys.N;

    ToolStripMenuItem menuOuvrir = new ToolStripMenuItem("&Ouvrir...", null, MenuOuvrir_Click);
    menuOuvrir.ShortcutKeys = Keys.Control | Keys.O;

    ToolStripMenuItem menuEnregistrer = new ToolStripMenuItem("&Enregistrer", null, MenuEnregistrer_Click);
    menuEnregistrer.ShortcutKeys = Keys.Control | Keys.S;

    ToolStripMenuItem menuEnregistrerSous = new ToolStripMenuItem("Enregistrer &sous...", null, MenuEnregistrerSous_Click);

    ToolStripSeparator separateur1 = new ToolStripSeparator();

    ToolStripMenuItem menuImprimer = new ToolStripMenuItem("&Imprimer...", null, MenuImprimer_Click);
    menuImprimer.ShortcutKeys = Keys.Control | Keys.P;

    ToolStripMenuItem menuAperçuAvantImpression = new ToolStripMenuItem("Aperçu avant impression", null, MenuAperçuAvantImpression_Click);

    ToolStripSeparator separateur2 = new ToolStripSeparator();

    ToolStripMenuItem menuQuitter = new ToolStripMenuItem("&Quitter", null, MenuQuitter_Click);
    menuQuitter.ShortcutKeys = Keys.Alt | Keys.F4;

    // Ajout des sous-menus au menu Fichier
    menuFichier.DropDownItems.Add(menuNouveau);
    menuFichier.DropDownItems.Add(menuOuvrir);
    menuFichier.DropDownItems.Add(menuEnregistrer);
    menuFichier.DropDownItems.Add(menuEnregistrerSous);
    menuFichier.DropDownItems.Add(separateur1);
    menuFichier.DropDownItems.Add(menuImprimer);
    menuFichier.DropDownItems.Add(menuAperçuAvantImpression);
    menuFichier.DropDownItems.Add(separateur2);
    menuFichier.DropDownItems.Add(menuQuitter);

    // Création des sous-menus Nouveau
    ToolStripMenuItem menuNouveauDocument = new ToolStripMenuItem("&Document", null, MenuNouveauDocument_Click);
    ToolStripMenuItem menuNouveauProjet = new ToolStripMenuItem("&Projet", null, MenuNouveauProjet_Click);

    // Ajout des sous-menus au menu Nouveau
    menuNouveau.DropDownItems.Add(menuNouveauDocument);
    menuNouveau.DropDownItems.Add(menuNouveauProjet);

    // Création des sous-menus pour le menu Edition
    ToolStripMenuItem menuAnnuler = new ToolStripMenuItem("&Annuler", null, MenuAnnuler_Click);
    menuAnnuler.ShortcutKeys = Keys.Control | Keys.Z;

    ToolStripMenuItem menuRétablir = new ToolStripMenuItem("&Rétablir", null, MenuRétablir_Click);
    menuRétablir.ShortcutKeys = Keys.Control | Keys.Y;

    ToolStripSeparator separateur3 = new ToolStripSeparator();

    ToolStripMenuItem menuCouper = new ToolStripMenuItem("Co&uper", null, MenuCouper_Click);
    menuCouper.ShortcutKeys = Keys.Control | Keys.X;

    ToolStripMenuItem menuCopier = new ToolStripMenuItem("&Copier", null, MenuCopier_Click);
    menuCopier.ShortcutKeys = Keys.Control | Keys.C;

    ToolStripMenuItem menuColler = new ToolStripMenuItem("C&oller", null, MenuColler_Click);
    menuColler.ShortcutKeys = Keys.Control | Keys.V;

    ToolStripMenuItem menuSupprimer = new ToolStripMenuItem("&Supprimer", null, MenuSupprimer_Click);
    menuSupprimer.ShortcutKeys = Keys.Delete;

    ToolStripSeparator separateur4 = new ToolStripSeparator();

    ToolStripMenuItem menuSélectionnerTout = new ToolStripMenuItem("Sélectionner &tout", null, MenuSélectionnerTout_Click);
    menuSélectionnerTout.ShortcutKeys = Keys.Control | Keys.A;

    // Ajout des sous-menus au menu Edition
    menuEdition.DropDownItems.Add(menuAnnuler);
    menuEdition.DropDownItems.Add(menuRétablir);
    menuEdition.DropDownItems.Add(separateur3);
    menuEdition.DropDownItems.Add(menuCouper);
    menuEdition.DropDownItems.Add(menuCopier);
    menuEdition.DropDownItems.Add(menuColler);
    menuEdition.DropDownItems.Add(menuSupprimer);
    menuEdition.DropDownItems.Add(separateur4);
    menuEdition.DropDownItems.Add(menuSélectionnerTout);

    // Ajout du MenuStrip au formulaire
    this.Controls.Add(menuPrincipal);
    this.MainMenuStrip = menuPrincipal;
}

// Gestionnaires d'événements pour les éléments de menu
private void MenuNouveau_Click(object sender, EventArgs e)
{
    // Un menu avec sous-menus ne déclenche généralement pas d'action directe
}

private void MenuNouveauDocument_Click(object sender, EventArgs e)
{
    MessageBox.Show("Création d'un nouveau document");
}

private void MenuNouveauProjet_Click(object sender, EventArgs e)
{
    MessageBox.Show("Création d'un nouveau projet");
}

private void MenuOuvrir_Click(object sender, EventArgs e)
{
    OpenFileDialog dlg = new OpenFileDialog();
    dlg.Filter = "Tous les fichiers (*.*)|*.*";
    if (dlg.ShowDialog() == DialogResult.OK)
    {
        MessageBox.Show("Ouverture du fichier : " + dlg.FileName);
    }
}

private void MenuEnregistrer_Click(object sender, EventArgs e)
{
    MessageBox.Show("Enregistrement du fichier");
}

private void MenuEnregistrerSous_Click(object sender, EventArgs e)
{
    SaveFileDialog dlg = new SaveFileDialog();
    dlg.Filter = "Tous les fichiers (*.*)|*.*";
    if (dlg.ShowDialog() == DialogResult.OK)
    {
        MessageBox.Show("Enregistrement sous : " + dlg.FileName);
    }
}

private void MenuImprimer_Click(object sender, EventArgs e)
{
    PrintDialog dlg = new PrintDialog();
    if (dlg.ShowDialog() == DialogResult.OK)
    {
        MessageBox.Show("Impression en cours...");
    }
}

private void MenuAperçuAvantImpression_Click(object sender, EventArgs e)
{
    MessageBox.Show("Aperçu avant impression");
}

private void MenuQuitter_Click(object sender, EventArgs e)
{
    this.Close();
}

private void MenuAnnuler_Click(object sender, EventArgs e)
{
    MessageBox.Show("Annuler");
}

private void MenuRétablir_Click(object sender, EventArgs e)
{
    MessageBox.Show("Rétablir");
}

private void MenuCouper_Click(object sender, EventArgs e)
{
    MessageBox.Show("Couper");
}

private void MenuCopier_Click(object sender, EventArgs e)
{
    MessageBox.Show("Copier");
}

private void MenuColler_Click(object sender, EventArgs e)
{
    MessageBox.Show("Coller");
}

private void MenuSupprimer_Click(object sender, EventArgs e)
{
    MessageBox.Show("Supprimer");
}

private void MenuSélectionnerTout_Click(object sender, EventArgs e)
{
    MessageBox.Show("Sélectionner tout");
}
```


### Personnalisation des menus

Les menus peuvent être personnalisés avec des images, des cases à cocher, et des boutons radio :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void AjouterMenuAvecOptions()
{
    // Création d'un menu Affichage personnalisé
    ToolStripMenuItem menuAffichage = (ToolStripMenuItem)menuPrincipal.Items["menuAffichage"];

    // Création d'un sous-menu Mode avec boutons radio
    ToolStripMenuItem menuMode = new ToolStripMenuItem("&Mode");

    ToolStripMenuItem menuModeNormal = new ToolStripMenuItem("&Normal");
    menuModeNormal.Checked = true;
    menuModeNormal.CheckOnClick = true;
    menuModeNormal.Click += (s, e) => {
        // S'assurer qu'un seul mode est sélectionné
        menuModeNormal.Checked = true;
        menuModeCompact.Checked = false;
        menuModeÉtendu.Checked = false;
    };

    ToolStripMenuItem menuModeCompact = new ToolStripMenuItem("&Compact");
    menuModeCompact.CheckOnClick = true;
    menuModeCompact.Click += (s, e) => {
        menuModeNormal.Checked = false;
        menuModeCompact.Checked = true;
        menuModeÉtendu.Checked = false;
    };

    ToolStripMenuItem menuModeÉtendu = new ToolStripMenuItem("É&tendu");
    menuModeÉtendu.CheckOnClick = true;
    menuModeÉtendu.Click += (s, e) => {
        menuModeNormal.Checked = false;
        menuModeCompact.Checked = false;
        menuModeÉtendu.Checked = true;
    };

    menuMode.DropDownItems.Add(menuModeNormal);
    menuMode.DropDownItems.Add(menuModeCompact);
    menuMode.DropDownItems.Add(menuModeÉtendu);

    // Création d'options avec cases à cocher
    ToolStripMenuItem menuBarreOutils = new ToolStripMenuItem("Barre d'&outils");
    menuBarreOutils.Checked = true;
    menuBarreOutils.CheckOnClick = true;
    menuBarreOutils.CheckStateChanged += (s, e) => {
        // Afficher ou masquer la barre d'outils
        toolStrip.Visible = menuBarreOutils.Checked;
    };

    ToolStripMenuItem menuBarreÉtat = new ToolStripMenuItem("Barre d'&état");
    menuBarreÉtat.Checked = true;
    menuBarreÉtat.CheckOnClick = true;
    menuBarreÉtat.CheckStateChanged += (s, e) => {
        // Afficher ou masquer la barre d'état
        statusStrip.Visible = menuBarreÉtat.Checked;
    };

    // Création d'un sous-menu Thème avec des images
    ToolStripMenuItem menuThème = new ToolStripMenuItem("&Thème");

    ToolStripMenuItem menuThèmeClair = new ToolStripMenuItem("&Clair");
    // Créer une petite icône de 16x16 pixels pour représenter le thème clair
    Bitmap iconeClair = new Bitmap(16, 16);
    using (Graphics g = Graphics.FromImage(iconeClair))
    {
        g.FillRectangle(Brushes.White, 0, 0, 16, 16);
        g.DrawRectangle(Pens.Gray, 0, 0, 15, 15);
    }
    menuThèmeClair.Image = iconeClair;
    menuThèmeClair.Click += (s, e) => {
        // Appliquer le thème clair
        AppliquerThème("Clair");
        // Cocher cet élément et décocher les autres
        menuThèmeClair.Checked = true;
        menuThèmeSombre.Checked = false;
        menuThèmeBleu.Checked = false;
    };

    ToolStripMenuItem menuThèmeSombre = new ToolStripMenuItem("&Sombre");
    // Créer une icône pour le thème sombre
    Bitmap iconeSombre = new Bitmap(16, 16);
    using (Graphics g = Graphics.FromImage(iconeSombre))
    {
        g.FillRectangle(new SolidBrush(Color.FromArgb(50, 50, 50)), 0, 0, 16, 16);
        g.DrawRectangle(Pens.Gray, 0, 0, 15, 15);
    }
    menuThèmeSombre.Image = iconeSombre;
    menuThèmeSombre.Click += (s, e) => {
        // Appliquer le thème sombre
        AppliquerThème("Sombre");
        // Mise à jour des cases à cocher
        menuThèmeClair.Checked = false;
        menuThèmeSombre.Checked = true;
        menuThèmeBleu.Checked = false;
    };

    ToolStripMenuItem menuThèmeBleu = new ToolStripMenuItem("&Bleu");
    // Créer une icône pour le thème bleu
    Bitmap iconeBleu = new Bitmap(16, 16);
    using (Graphics g = Graphics.FromImage(iconeBleu))
    {
        g.FillRectangle(new SolidBrush(Color.FromArgb(200, 220, 255)), 0, 0, 16, 16);
        g.DrawRectangle(Pens.Gray, 0, 0, 15, 15);
    }
    menuThèmeBleu.Image = iconeBleu;
    menuThèmeBleu.Click += (s, e) => {
        // Appliquer le thème bleu
        AppliquerThème("Bleu");
        // Mise à jour des cases à cocher
        menuThèmeClair.Checked = false;
        menuThèmeSombre.Checked = false;
        menuThèmeBleu.Checked = true;
    };

    menuThème.DropDownItems.Add(menuThèmeClair);
    menuThème.DropDownItems.Add(menuThèmeSombre);
    menuThème.DropDownItems.Add(menuThèmeBleu);

    // Ajout des éléments au menu Affichage
    menuAffichage.DropDownItems.Add(menuMode);
    menuAffichage.DropDownItems.Add(new ToolStripSeparator());
    menuAffichage.DropDownItems.Add(menuBarreOutils);
    menuAffichage.DropDownItems.Add(menuBarreÉtat);
    menuAffichage.DropDownItems.Add(new ToolStripSeparator());
    menuAffichage.DropDownItems.Add(menuThème);
}

private void AppliquerThème(string nomThème)
{
    // Fonction fictive pour appliquer un thème
    MessageBox.Show($"Application du thème {nomThème}");
}
```


### Menus dynamiques

On peut créer des menus qui se remplissent dynamiquement, comme un menu "Fichiers récents" :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerMenuFichiersRécents()
{
    // Obtenir une référence au menu Fichier
    ToolStripMenuItem menuFichier = (ToolStripMenuItem)menuPrincipal.Items["menuFichier"];

    // Ajouter un séparateur et l'élément "Fichiers récents"
    menuFichier.DropDownItems.Add(new ToolStripSeparator());

    ToolStripMenuItem menuFichiersRécents = new ToolStripMenuItem("Fichiers &récents");
    menuFichier.DropDownItems.Add(menuFichiersRécents);

    // Fonction pour mettre à jour la liste des fichiers récents
    void MettreAJourFichiersRécents()
    {
        // Vider le menu
        menuFichiersRécents.DropDownItems.Clear();

        // Liste fictive de fichiers récents (dans une application réelle, cela viendrait d'une configuration)
        List<string> fichiersRécents = new List<string>
        {
            @"C:\Documents\Document1.txt",
            @"C:\Documents\Rapport.docx",
            @"C:\Projets\Projet1\Main.cs",
            @"D:\Images\Photo.jpg"
        };

        // Ajouter chaque fichier au menu
        if (fichiersRécents.Count > 0)
        {
            for (int i = 0; i < fichiersRécents.Count; i++)
            {
                string fichier = fichiersRécents[i];
                string nomFichier = Path.GetFileName(fichier);

                // Créer un élément de menu avec numéro et nom de fichier
                ToolStripMenuItem item = new ToolStripMenuItem(
                    $"&{i + 1} {nomFichier}",
                    null,
                    (s, e) => OuvrirFichierRécent(fichier));

                // Ajouter un tooltip avec le chemin complet
                item.ToolTipText = fichier;

                menuFichiersRécents.DropDownItems.Add(item);
            }

            // Ajouter un séparateur et "Effacer la liste"
            menuFichiersRécents.DropDownItems.Add(new ToolStripSeparator());
            menuFichiersRécents.DropDownItems.Add(
                new ToolStripMenuItem("&Effacer la liste", null, (s, e) => EffacerFichiersRécents()));
        }
        else
        {
            // Afficher un élément désactivé si la liste est vide
            ToolStripMenuItem itemVide = new ToolStripMenuItem("(Aucun fichier récent)");
            itemVide.Enabled = false;
            menuFichiersRécents.DropDownItems.Add(itemVide);
        }
    }

    // Initialiser le menu
    MettreAJourFichiersRécents();
}

private void OuvrirFichierRécent(string chemin)
{
    MessageBox.Show($"Ouverture du fichier récent : {chemin}");
    // Dans une application réelle, ajouter le fichier en haut de la liste et la mettre à jour
}

private void EffacerFichiersRécents()
{
    MessageBox.Show("Liste des fichiers récents effacée");
    // Dans une application réelle, vider la liste et mettre à jour le menu
}
```


## 7.6.2. ContextMenuStrip

Le contrôle `ContextMenuStrip` permet de créer des menus contextuels (menus qui apparaissent lors d'un clic droit) pour n'importe quel contrôle.

### Menu contextuel de base

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerMenuContextuel()
{
    // Création du menu contextuel
    ContextMenuStrip menuContextuel = new ContextMenuStrip();

    // Ajout des éléments au menu contextuel
    ToolStripMenuItem menuCouperContextuel = new ToolStripMenuItem("Couper", null, MenuCouper_Click);
    menuCouperContextuel.ShortcutKeys = Keys.Control | Keys.X;
    menuCouperContextuel.ShowShortcutKeys = true;

    ToolStripMenuItem menuCopierContextuel = new ToolStripMenuItem("Copier", null, MenuCopier_Click);
    menuCopierContextuel.ShortcutKeys = Keys.Control | Keys.C;
    menuCopierContextuel.ShowShortcutKeys = true;

    ToolStripMenuItem menuCollerContextuel = new ToolStripMenuItem("Coller", null, MenuColler_Click);
    menuCollerContextuel.ShortcutKeys = Keys.Control | Keys.V;
    menuCollerContextuel.ShowShortcutKeys = true;

    ToolStripSeparator separateur = new ToolStripSeparator();

    ToolStripMenuItem menuSélectionnerToutContextuel = new ToolStripMenuItem("Sélectionner tout", null, MenuSélectionnerTout_Click);
    menuSélectionnerToutContextuel.ShortcutKeys = Keys.Control | Keys.A;
    menuSélectionnerToutContextuel.ShowShortcutKeys = true;

    // Ajout des éléments au menu
    menuContextuel.Items.Add(menuCouperContextuel);
    menuContextuel.Items.Add(menuCopierContextuel);
    menuContextuel.Items.Add(menuCollerContextuel);
    menuContextuel.Items.Add(separateur);
    menuContextuel.Items.Add(menuSélectionnerToutContextuel);

    // Création d'un TextBox pour tester le menu contextuel
    TextBox textBox = new TextBox();
    textBox.Multiline = true;
    textBox.Size = new Size(300, 200);
    textBox.Location = new Point(50, 100);
    textBox.ContextMenuStrip = menuContextuel;

    // Ajout du TextBox au formulaire
    this.Controls.Add(textBox);

    // Ajouter un gestionnaire d'événement pour le menu contextuel
    menuContextuel.Opening += MenuContextuel_Opening;
}

private void MenuContextuel_Opening(object sender, CancelEventArgs e)
{
    // Obtenir une référence au menu contextuel
    ContextMenuStrip menu = sender as ContextMenuStrip;

    // Obtenir une référence au contrôle qui a ouvert le menu
    Control contrôle = menu.SourceControl;

    // Si le contrôle est un TextBox, adapter le menu en fonction de son état
    if (contrôle is TextBox textBox)
    {
        bool texteEstSélectionné = textBox.SelectionLength > 0;
        bool peutColler = Clipboard.ContainsText();

        // Activer/désactiver les éléments de menu en fonction de l'état
        menu.Items["menuCouperContextuel"].Enabled = texteEstSélectionné;
        menu.Items["menuCopierContextuel"].Enabled = texteEstSélectionné;
        menu.Items["menuCollerContextuel"].Enabled = peutColler;
    }
}
```

### Menus contextuels spécifiques

Les menus contextuels peuvent être adaptés à différents types de contrôles :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerMenusContextuelsSpécifiques()
{
    // Menu contextuel pour un DataGridView
    ContextMenuStrip menuContextuelGrid = new ContextMenuStrip();

    menuContextuelGrid.Items.Add(new ToolStripMenuItem("Ajouter une ligne", null, (s, e) => {
        DataGridView grid = menuContextuelGrid.SourceControl as DataGridView;
        if (grid != null)
        {
            // Ajouter une ligne (dans une application réelle, utilisez votre source de données)
            (grid.DataSource as DataTable)?.Rows.Add();
        }
    }));

    menuContextuelGrid.Items.Add(new ToolStripMenuItem("Supprimer la ligne", null, (s, e) => {
        DataGridView grid = menuContextuelGrid.SourceControl as DataGridView;
        if (grid != null && grid.CurrentRow != null)
        {
            // Supprimer la ligne
            grid.Rows.Remove(grid.CurrentRow);
        }
    }));

    menuContextuelGrid.Items.Add(new ToolStripSeparator());

    menuContextuelGrid.Items.Add(new ToolStripMenuItem("Copier la cellule", null, (s, e) => {
        DataGridView grid = menuContextuelGrid.SourceControl as DataGridView;
        if (grid != null && grid.CurrentCell != null)
        {
            // Copier la valeur de la cellule dans le presse-papiers
            Clipboard.SetText(grid.CurrentCell.Value?.ToString() ?? "");
        }
    }));

    // Menu contextuel pour des images
    ContextMenuStrip menuContextuelImage = new ContextMenuStrip();

    menuContextuelImage.Items.Add(new ToolStripMenuItem("Agrandir", null, (s, e) => {
        PictureBox pictureBox = menuContextuelImage.SourceControl as PictureBox;
        if (pictureBox != null)
        {
            // Simuler un agrandissement (normalement, vous ouvririez une forme de zoom)
            pictureBox.SizeMode = PictureBoxSizeMode.Zoom;
            pictureBox.Size = new Size(pictureBox.Width * 2, pictureBox.Height * 2);
        }
    }));

    menuContextuelImage.Items.Add(new ToolStripMenuItem("Restaurer", null, (s, e) => {
        PictureBox pictureBox = menuContextuelImage.SourceControl as PictureBox;
        if (pictureBox != null)
        {
            // Restaurer la taille originale
            pictureBox.SizeMode = PictureBoxSizeMode.Normal;
            pictureBox.Size = new Size(pictureBox.Image.Width, pictureBox.Image.Height);
        }
    }));

    menuContextuelImage.Items.Add(new ToolStripSeparator());

    menuContextuelImage.Items.Add(new ToolStripMenuItem("Enregistrer sous...", null, (s, e) => {
        PictureBox pictureBox = menuContextuelImage.SourceControl as PictureBox;
        if (pictureBox != null && pictureBox.Image != null)
        {
            SaveFileDialog dlg = new SaveFileDialog();
            dlg.Filter = "Images PNG (*.png)|*.png|Images JPEG (*.jpg)|*.jpg|Tous les fichiers (*.*)|*.*";
            dlg.DefaultExt = "png";

            if (dlg.ShowDialog() == DialogResult.OK)
            {
                // Enregistrer l'image
                string extension = Path.GetExtension(dlg.FileName).ToLower();

                if (extension == ".png")
                {
                    pictureBox.Image.Save(dlg.FileName, System.Drawing.Imaging.ImageFormat.Png);
                }
                else if (extension == ".jpg" || extension == ".jpeg")
                {
                    pictureBox.Image.Save(dlg.FileName, System.Drawing.Imaging.ImageFormat.Jpeg);
                }
                else
                {
                    pictureBox.Image.Save(dlg.FileName);
                }
            }
        }
    }));

    // Création des contrôles et association des menus contextuels
    DataGridView dataGridView = new DataGridView();
    dataGridView.Size = new Size(400, 200);
    dataGridView.Location = new Point(50, 350);
    dataGridView.ContextMenuStrip = menuContextuelGrid;

    // Configuration minimale du DataGridView
    dataGridView.AutoGenerateColumns = true;
    DataTable table = new DataTable();
    table.Columns.Add("ID", typeof(int));
    table.Columns.Add("Nom", typeof(string));
    table.Columns.Add("Valeur", typeof(decimal));
    table.Rows.Add(1, "Article 1", 10.99m);
    table.Rows.Add(2, "Article 2", 25.50m);
    dataGridView.DataSource = table;

    // Création d'une image
    PictureBox pictureBox = new PictureBox();
    pictureBox.Size = new Size(200, 150);
    pictureBox.Location = new Point(500, 350);
    pictureBox.BorderStyle = BorderStyle.FixedSingle;
    pictureBox.ContextMenuStrip = menuContextuelImage;

    // Créer une image simple
    Bitmap image = new Bitmap(200, 150);
    using (Graphics g = Graphics.FromImage(image))
    {
        g.Clear(Color.LightBlue);
        g.FillEllipse(Brushes.Blue, 50, 50, 100, 50);
        g.DrawString("Exemple", new Font("Arial", 12), Brushes.White, 70, 65);
    }
    pictureBox.Image = image;

    // Ajout des contrôles au formulaire
    this.Controls.Add(dataGridView);
    this.Controls.Add(pictureBox);
}
```


### Menu contextuel dynamique

On peut créer des menus contextuels qui s'adaptent en fonction de l'élément ou de la position du clic :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerMenuContextuelDynamique()
{
    // Création du ListView
    ListView listView = new ListView();
    listView.Size = new Size(400, 300);
    listView.Location = new Point(50, 600);
    listView.View = View.Details;
    listView.FullRowSelect = true;
    listView.GridLines = true;

    // Configuration des colonnes
    listView.Columns.Add("ID", 50);
    listView.Columns.Add("Nom", 150);
    listView.Columns.Add("Type", 100);
    listView.Columns.Add("Taille", 80);

    // Ajout d'éléments
    ListViewItem item1 = new ListViewItem("1");
    item1.SubItems.AddRange(new string[] { "Document.docx", "Document", "25 KB" });
    item1.Tag = "document";

    ListViewItem item2 = new ListViewItem("2");
    item2.SubItems.AddRange(new string[] { "Image.jpg", "Image", "500 KB" });
    item2.Tag = "image";

    ListViewItem item3 = new ListViewItem("3");
    item3.SubItems.AddRange(new string[] { "Vidéo.mp4", "Vidéo", "10 MB" });
    item3.Tag = "video";

    ListViewItem item4 = new ListViewItem("4");
    item4.SubItems.AddRange(new string[] { "Projet.zip", "Archive", "2 MB" });
    item4.Tag = "archive";

    listView.Items.AddRange(new ListViewItem[] { item1, item2, item3, item4 });

    // Création du menu contextuel
    ContextMenuStrip menuContextuelListe = new ContextMenuStrip();

    // Abonnement à l'événement Opening pour créer un menu dynamique
    menuContextuelListe.Opening += (sender, e) => {
        // Effacer les éléments existants
        menuContextuelListe.Items.Clear();

        // Déterminer si un élément est sélectionné
        bool élémentSélectionné = listView.SelectedItems.Count > 0;

        // Actions de base - toujours disponibles
        ToolStripMenuItem itemActualiser = new ToolStripMenuItem("Actualiser", null, (s, args) => {
            MessageBox.Show("Actualisation de la liste");
        });
        menuContextuelListe.Items.Add(itemActualiser);

        // S'il n'y a pas d'élément sélectionné, terminer ici
        if (!élémentSélectionné)
        {
            menuContextuelListe.Items.Add(new ToolStripMenuItem("Ajouter un nouvel élément...", null, (s, args) => {
                MessageBox.Show("Ajout d'un nouvel élément");
            }));
            return;
        }

        // Si un élément est sélectionné, ajouter un séparateur
        menuContextuelListe.Items.Add(new ToolStripSeparator());

        // Obtenir l'élément sélectionné
        ListViewItem itemSélectionné = listView.SelectedItems[0];
        string nomFichier = itemSélectionné.SubItems[1].Text;

        // Actions communes pour tous les types de fichiers
        menuContextuelListe.Items.Add(new ToolStripMenuItem($"Ouvrir \"{nomFichier}\"", null, (s, args) => {
            MessageBox.Show($"Ouverture de {nomFichier}");
        }));

        menuContextuelListe.Items.Add(new ToolStripMenuItem("Renommer...", null, (s, args) => {
            MessageBox.Show($"Renommer {nomFichier}");
        }));

        menuContextuelListe.Items.Add(new ToolStripMenuItem("Supprimer", null, (s, args) => {
            if (MessageBox.Show($"Voulez-vous vraiment supprimer {nomFichier} ?", "Confirmation",
                               MessageBoxButtons.YesNo, MessageBoxIcon.Question) == DialogResult.Yes)
            {
                listView.Items.Remove(itemSélectionné);
            }
        }));

        // Actions spécifiques en fonction du type de fichier
        string typeFichier = itemSélectionné.Tag as string;

        menuContextuelListe.Items.Add(new ToolStripSeparator());

        switch (typeFichier)
        {
            case "document":
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Éditer", null, (s, args) => {
                    MessageBox.Show($"Édition de {nomFichier}");
                }));
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Imprimer", null, (s, args) => {
                    MessageBox.Show($"Impression de {nomFichier}");
                }));
                break;

            case "image":
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Afficher", null, (s, args) => {
                    MessageBox.Show($"Affichage de {nomFichier}");
                }));
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Éditer", null, (s, args) => {
                    MessageBox.Show($"Édition de {nomFichier}");
                }));
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Utiliser comme fond d'écran", null, (s, args) => {
                    MessageBox.Show($"{nomFichier} définie comme fond d'écran");
                }));
                break;

            case "video":
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Lire", null, (s, args) => {
                    MessageBox.Show($"Lecture de {nomFichier}");
                }));
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Extraire l'audio", null, (s, args) => {
                    MessageBox.Show($"Extraction de l'audio de {nomFichier}");
                }));
                break;

            case "archive":
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Extraire ici", null, (s, args) => {
                    MessageBox.Show($"Extraction de {nomFichier} ici");
                }));
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Extraire vers...", null, (s, args) => {
                    MessageBox.Show($"Extraction de {nomFichier} vers...");
                }));
                menuContextuelListe.Items.Add(new ToolStripMenuItem("Ajouter des fichiers...", null, (s, args) => {
                    MessageBox.Show($"Ajout de fichiers à {nomFichier}");
                }));
                break;
        }
    };

    // Associer le menu contextuel au ListView
    listView.ContextMenuStrip = menuContextuelListe;

    // Ajouter le ListView au formulaire
    this.Controls.Add(listView);
}
```


## 7.6.3. ToolStrip et StatusStrip

Les contrôles `ToolStrip` et `StatusStrip` permettent de créer des barres d'outils et des barres d'état respectivement.

### Création d'une barre d'outils

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerBarreOutils()
{
    // Création de la barre d'outils
    ToolStrip toolStrip = new ToolStrip();
    toolStrip.Dock = DockStyle.Top;

    // Définir des images pour les boutons
    // Dans une application réelle, utilisez des icônes appropriées
    ImageList imageList = new ImageList();
    imageList.ColorDepth = ColorDepth.Depth32Bit;
    imageList.ImageSize = new Size(16, 16);

    // Créer quelques images simples pour l'exemple
    Bitmap nouveau = new Bitmap(16, 16);
    using (Graphics g = Graphics.FromImage(nouveau))
    {
        g.FillRectangle(Brushes.White, 0, 0, 16, 16);
        g.DrawRectangle(Pens.Black, 2, 2, 11, 11);
    }

    Bitmap ouvrir = new Bitmap(16, 16);
    using (Graphics g = Graphics.FromImage(ouvrir))
    {
        g.FillRectangle(Brushes.Tan, 2, 4, 12, 10);
        g.DrawRectangle(Pens.Black, 2, 4, 12, 10);
        g.DrawLine(Pens.Black, 2, 4, 6, 2);
        g.DrawLine(Pens.Black, 6, 2, 14, 2);
        g.DrawLine(Pens.Black, 14, 2, 14, 4);
    }

    Bitmap enregistrer = new Bitmap(16, 16);
    using (Graphics g = Graphics.FromImage(enregistrer))
    {
        g.FillRectangle(Brushes.LightBlue, 2, 2, 12, 12);
        g.DrawRectangle(Pens.Blue, 2, 2, 12, 12);
        g.FillRectangle(Brushes.White, 4, 7, 8, 5);
        g.DrawRectangle(Pens.Black, 4, 7, 8, 5);
    }

    imageList.Images.AddRange(new Image[] { nouveau, ouvrir, enregistrer });

    // Bouton Nouveau
    ToolStripButton btnNouveau = new ToolStripButton();
    btnNouveau.DisplayStyle = ToolStripItemDisplayStyle.Image;
    btnNouveau.Image = imageList.Images[0];
    btnNouveau.ToolTipText = "Nouveau (Ctrl+N)";
    btnNouveau.Click += (s, e) => MenuNouveau_Click(s, e);

    // Bouton Ouvrir
    ToolStripButton btnOuvrir = new ToolStripButton();
    btnOuvrir.DisplayStyle = ToolStripItemDisplayStyle.Image;
    btnOuvrir.Image = imageList.Images[1];
    btnOuvrir.ToolTipText = "Ouvrir (Ctrl+O)";
    btnOuvrir.Click += (s, e) => MenuOuvrir_Click(s, e);

    // Bouton Enregistrer
    ToolStripButton btnEnregistrer = new ToolStripButton();
    btnEnregistrer.DisplayStyle = ToolStripItemDisplayStyle.Image;
    btnEnregistrer.Image = imageList.Images[2];
    btnEnregistrer.ToolTipText = "Enregistrer (Ctrl+S)";
    btnEnregistrer.Click += (s, e) => MenuEnregistrer_Click(s, e);

    // Séparateur
    ToolStripSeparator separateur1 = new ToolStripSeparator();

    // Bouton avec texte et image
    ToolStripButton btnImprimer = new ToolStripButton();
    btnImprimer.Text = "Imprimer";
    btnImprimer.Image = enregistrer;  // Utiliser la même image pour simplifier
    btnImprimer.DisplayStyle = ToolStripItemDisplayStyle.ImageAndText;
    btnImprimer.ToolTipText = "Imprimer le document (Ctrl+P)";
    btnImprimer.Click += (s, e) => MenuImprimer_Click(s, e);

    // Séparateur
    ToolStripSeparator separateur2 = new ToolStripSeparator();

    // Liste déroulante
    ToolStripDropDownButton btnZoom = new ToolStripDropDownButton();
    btnZoom.Text = "Zoom";
    btnZoom.ToolTipText = "Définir le niveau de zoom";

    // Éléments de la liste déroulante
    ToolStripMenuItem zoom50 = new ToolStripMenuItem("50%");
    zoom50.Click += (s, e) => { MessageBox.Show("Zoom à 50%"); };

    ToolStripMenuItem zoom100 = new ToolStripMenuItem("100%");
    zoom100.Click += (s, e) => { MessageBox.Show("Zoom à 100%"); };

    ToolStripMenuItem zoom200 = new ToolStripMenuItem("200%");
    zoom200.Click += (s, e) => { MessageBox.Show("Zoom à 200%"); };

    // Ajout des éléments à la liste déroulante
    btnZoom.DropDownItems.Add(zoom50);
    btnZoom.DropDownItems.Add(zoom100);
    btnZoom.DropDownItems.Add(zoom200);

    // ComboBox dans la barre d'outils
    ToolStripComboBox comboStyle = new ToolStripComboBox();
    comboStyle.ToolTipText = "Style du texte";
    comboStyle.Items.AddRange(new object[] { "Normal", "Gras", "Italique", "Souligné" });
    comboStyle.SelectedIndex = 0;
    comboStyle.Width = 100;
    comboStyle.SelectedIndexChanged += (s, e) => {
        MessageBox.Show($"Style sélectionné : {comboStyle.SelectedItem}");
    };

    // TextBox dans la barre d'outils
    ToolStripTextBox textRecherche = new ToolStripTextBox();
    textRecherche.PlaceholderText = "Rechercher...";
    textRecherche.Width = 150;
    textRecherche.KeyDown += (s, e) => {
        if (e.KeyCode == Keys.Enter)
        {
            MessageBox.Show($"Recherche de : {textRecherche.Text}");
        }
    };

    // Bouton de recherche
    ToolStripButton btnRechercher = new ToolStripButton();
    btnRechercher.Text = "Rechercher";
    btnRechercher.DisplayStyle = ToolStripItemDisplayStyle.Text;
    btnRechercher.Click += (s, e) => {
        MessageBox.Show($"Recherche de : {textRecherche.Text}");
    };

    // Ajout des contrôles à la barre d'outils
    toolStrip.Items.Add(btnNouveau);
    toolStrip.Items.Add(btnOuvrir);
    toolStrip.Items.Add(btnEnregistrer);
    toolStrip.Items.Add(separateur1);
    toolStrip.Items.Add(btnImprimer);
    toolStrip.Items.Add(separateur2);
    toolStrip.Items.Add(btnZoom);
    toolStrip.Items.Add(comboStyle);
    toolStrip.Items.Add(new ToolStripSeparator());
    toolStrip.Items.Add(textRecherche);
    toolStrip.Items.Add(btnRechercher);

    // Ajout de la barre d'outils au formulaire
    this.Controls.Add(toolStrip);
}
```


### Barres d'outils multiples

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerBarresOutilsMultiples()
{
    // Barre d'outils principale
    ToolStrip toolStripPrincipal = new ToolStrip();
    toolStripPrincipal.Dock = DockStyle.Top;

    // Boutons de la barre d'outils principale
    ToolStripButton btnNouveau = new ToolStripButton("Nouveau");
    ToolStripButton btnOuvrir = new ToolStripButton("Ouvrir");
    ToolStripButton btnEnregistrer = new ToolStripButton("Enregistrer");

    toolStripPrincipal.Items.AddRange(new ToolStripItem[] {
        btnNouveau, btnOuvrir, btnEnregistrer
    });

    // Barre d'outils de formatage
    ToolStrip toolStripFormatage = new ToolStrip();
    toolStripFormatage.Dock = DockStyle.Top;

    // Boutons de formatage
    ToolStripButton btnGras = new ToolStripButton("G");
    btnGras.Font = new Font(btnGras.Font, FontStyle.Bold);
    btnGras.DisplayStyle = ToolStripItemDisplayStyle.Text;
    btnGras.ToolTipText = "Gras (Ctrl+G)";

    ToolStripButton btnItalique = new ToolStripButton("I");
    btnItalique.Font = new Font(btnItalique.Font, FontStyle.Italic);
    btnItalique.DisplayStyle = ToolStripItemDisplayStyle.Text;
    btnItalique.ToolTipText = "Italique (Ctrl+I)";

    ToolStripButton btnSouligné = new ToolStripButton("S");
    btnSouligné.Font = new Font(btnSouligné.Font, FontStyle.Underline);
    btnSouligné.DisplayStyle = ToolStripItemDisplayStyle.Text;
    btnSouligné.ToolTipText = "Souligné (Ctrl+U)";

    ToolStripSeparator sep1 = new ToolStripSeparator();

    // ComboBox pour la police
    ToolStripComboBox comboPolice = new ToolStripComboBox();
    comboPolice.Items.AddRange(new string[] { "Arial", "Times New Roman", "Courier New", "Verdana" });
    comboPolice.SelectedIndex = 0;
    comboPolice.Width = 150;

    // ComboBox pour la taille
    ToolStripComboBox comboTaille = new ToolStripComboBox();
    comboTaille.Items.AddRange(new string[] { "8", "9", "10", "11", "12", "14", "16", "18", "20", "24", "28", "36" });
    comboTaille.SelectedIndex = 4;  // 12 points par défaut
    comboTaille.Width = 50;

    toolStripFormatage.Items.AddRange(new ToolStripItem[] {
        btnGras, btnItalique, btnSouligné, sep1, comboPolice, comboTaille
    });

    // ToolStripContainer pour permettre le déplacement des barres d'outils
    ToolStripContainer container = new ToolStripContainer();
    container.Dock = DockStyle.Fill;

    // Ajouter les barres d'outils au conteneur
    container.TopToolStripPanel.Controls.Add(toolStripPrincipal);
    container.TopToolStripPanel.Controls.Add(toolStripFormatage);

    // Zone de contenu pour le texte
    RichTextBox richTextBox = new RichTextBox();
    richTextBox.Dock = DockStyle.Fill;

    // Ajouter le RichTextBox au conteneur
    container.ContentPanel.Controls.Add(richTextBox);

    // Gérer les événements de formatage
    btnGras.Click += (s, e) => {
        richTextBox.SelectionFont = new Font(richTextBox.SelectionFont,
            richTextBox.SelectionFont.Style ^ FontStyle.Bold);
    };

    btnItalique.Click += (s, e) => {
        richTextBox.SelectionFont = new Font(richTextBox.SelectionFont,
            richTextBox.SelectionFont.Style ^ FontStyle.Italic);
    };

    btnSouligné.Click += (s, e) => {
        richTextBox.SelectionFont = new Font(richTextBox.SelectionFont,
            richTextBox.SelectionFont.Style ^ FontStyle.Underline);
    };

    comboPolice.SelectedIndexChanged += (s, e) => {
        if (richTextBox.SelectionFont != null)
        {
            richTextBox.SelectionFont = new Font(
                comboPolice.SelectedItem.ToString(),
                richTextBox.SelectionFont.Size,
                richTextBox.SelectionFont.Style);
        }
    };

    comboTaille.SelectedIndexChanged += (s, e) => {
        if (richTextBox.SelectionFont != null)
        {
            richTextBox.SelectionFont = new Font(
                richTextBox.SelectionFont.FontFamily,
                float.Parse(comboTaille.SelectedItem.ToString()),
                richTextBox.SelectionFont.Style);
        }
    };

    // Ajouter le conteneur au formulaire
    this.Controls.Add(container);
}
```


### Création d'une barre d'état

```textmate
// .NET Framework 4.7.2 et .NET 8
private void CréerBarreÉtat()
{
    // Création de la barre d'état
    StatusStrip statusStrip = new StatusStrip();
    statusStrip.Dock = DockStyle.Bottom;

    // Label standard
    ToolStripStatusLabel labelInfo = new ToolStripStatusLabel();
    labelInfo.Text = "Prêt";
    labelInfo.BorderSides = ToolStripStatusLabelBorderSides.Right;
    labelInfo.BorderStyle = Border3DStyle.Etched;

    // Label avec un spring (qui s'étire pour remplir l'espace)
    ToolStripStatusLabel labelEspace = new ToolStripStatusLabel();
    labelEspace.Spring = true;

    // Label pour afficher la position du curseur
    ToolStripStatusLabel labelPosition = new ToolStripStatusLabel();
    labelPosition.Text = "Ln 1, Col 1";
    labelPosition.BorderSides = ToolStripStatusLabelBorderSides.Left | ToolStripStatusLabelBorderSides.Right;
    labelPosition.BorderStyle = Border3DStyle.Etched;

    // Label pour afficher l'état du verrouillage
    ToolStripStatusLabel labelVerrouillage = new ToolStripStatusLabel();
    labelVerrouillage.Text = "NUM";
    labelVerrouillage.BorderSides = ToolStripStatusLabelBorderSides.Left;
    labelVerrouillage.BorderStyle = Border3DStyle.Etched;

    // ProgressBar dans la barre d'état
    ToolStripProgressBar progressBar = new ToolStripProgressBar();
    progressBar.Width = 100;
    progressBar.Value = 0;

    // DropDownButton pour sélectionner l'encodage
    ToolStripDropDownButton dropDownEncodage = new ToolStripDropDownButton();
    dropDownEncodage.Text = "UTF-8";

    ToolStripMenuItem menuUTF8 = new ToolStripMenuItem("UTF-8");
    menuUTF8.Click += (s, e) => {
        dropDownEncodage.Text = "UTF-8";
    };

    ToolStripMenuItem menuUTF16 = new ToolStripMenuItem("UTF-16");
    menuUTF16.Click += (s, e) => {
        dropDownEncodage.Text = "UTF-16";
    };

    ToolStripMenuItem menuANSI = new ToolStripMenuItem("ANSI");
    menuANSI.Click += (s, e) => {
        dropDownEncodage.Text = "ANSI";
    };

    dropDownEncodage.DropDownItems.AddRange(new ToolStripItem[] {
        menuUTF8, menuUTF16, menuANSI
    });

    // Ajouter les éléments à la barre d'état
    statusStrip.Items.AddRange(new ToolStripItem[] {
        labelInfo, labelEspace, progressBar, labelPosition, labelVerrouillage, dropDownEncodage
    });

    // Ajouter la barre d'état au formulaire
    this.Controls.Add(statusStrip);

    // Créer un RichTextBox pour tester la barre d'état
    RichTextBox richTextBox = new RichTextBox();
    richTextBox.Dock = DockStyle.Fill;

    // Mettre à jour la position du curseur dans la barre d'état
    richTextBox.SelectionChanged += (s, e) => {
        int ligne = richTextBox.GetLineFromCharIndex(richTextBox.SelectionStart) + 1;
        int colonne = richTextBox.SelectionStart - richTextBox.GetFirstCharIndexFromLine(ligne - 1) + 1;
        labelPosition.Text = $"Ln {ligne}, Col {colonne}";
    };

    // Détecter les modifications pour afficher l'état
    bool documentModifié = false;
    richTextBox.TextChanged += (s, e) => {
        if (!documentModifié)
        {
            documentModifié = true;
            labelInfo.Text = "Document modifié";
        }
    };

    // Mettre à jour l'état des touches de verrouillage
    this.KeyDown += (s, e) => MettreAJourVerrouillage();

    void MettreAJourVerrouillage()
    {
        labelVerrouillage.Text = "";

        if (Control.IsKeyLocked(Keys.NumLock))
            labelVerrouillage.Text += "NUM ";

        if (Control.IsKeyLocked(Keys.CapsLock))
            labelVerrouillage.Text += "MAJ ";

        if (Control.IsKeyLocked(Keys.Scroll))
            labelVerrouillage.Text += "DÉFIL";

        if (labelVerrouillage.Text == "")
            labelVerrouillage.Text = "---";
    }

    // Simuler une opération de longue durée avec une barre de progression
    Button btnSimulerTâche = new Button();
    btnSimulerTâche.Text = "Simuler une tâche";
    btnSimulerTâche.Dock = DockStyle.Bottom;
    btnSimulerTâche.Click += async (s, e) => {
        btnSimulerTâche.Enabled = false;
        labelInfo.Text = "Traitement en cours...";

        // Désactiver le RichTextBox pendant le traitement
        richTextBox.Enabled = false;

        // Boucle pour simuler une opération longue avec mise à jour de la barre de progression
        for (int i = 0; i <= 100; i++)
        {
            progressBar.Value = i;
            await Task.Delay(50); // Attendre 50 ms pour simuler le temps de traitement
        }

        // Réinitialiser l'interface
        labelInfo.Text = "Traitement terminé";
        progressBar.Value = 0;
        richTextBox.Enabled = true;
        btnSimulerTâche.Enabled = true;
    };

    // Ajouter les contrôles au formulaire
    this.Controls.Add(richTextBox);
    this.Controls.Add(btnSimulerTâche);

    // Initialiser l'état des touches de verrouillage
    MettreAJourVerrouillage();
}
```


### Barre d'état personnalisée avec contrôle de rendu

```textmate
// .NET Framework 4.7.2 et .NET 8
public class BarreÉtatPersonnalisée : StatusStrip
{
    private ToolStripStatusLabel _labelÉtat;
    private ToolStripStatusLabel _labelPosition;
    private ToolStripStatusLabel _labelEncodage;
    private ToolStripProgressBar _progressBar;

    public string TexteÉtat
    {
        get { return _labelÉtat.Text; }
        set { _labelÉtat.Text = value; }
    }

    public string TextePosition
    {
        get { return _labelPosition.Text; }
        set { _labelPosition.Text = value; }
    }

    public string TexteEncodage
    {
        get { return _labelEncodage.Text; }
        set { _labelEncodage.Text = value; }
    }

    public int ValeurProgression
    {
        get { return _progressBar.Value; }
        set { _progressBar.Value = value; }
    }

    public BarreÉtatPersonnalisée()
    {
        this.Dock = DockStyle.Bottom;
        this.RenderMode = ToolStripRenderMode.Professional;

        // Créer un rendu personnalisé
        this.Renderer = new RenduBarreÉtatPersonnalisé();

        // Créer les éléments de la barre d'état
        _labelÉtat = new ToolStripStatusLabel("Prêt");
        _labelÉtat.BorderSides = ToolStripStatusLabelBorderSides.Right;
        _labelÉtat.AutoSize = true;
        _labelÉtat.Width = 100;

        ToolStripStatusLabel spring = new ToolStripStatusLabel();
        spring.Spring = true;

        _progressBar = new ToolStripProgressBar();
        _progressBar.Width = 100;
        _progressBar.Visible = false;

        _labelPosition = new ToolStripStatusLabel("Ln 1, Col 1");
        _labelPosition.BorderSides = ToolStripStatusLabelBorderSides.Left | ToolStripStatusLabelBorderSides.Right;

        _labelEncodage = new ToolStripStatusLabel("UTF-8");
        _labelEncodage.BorderSides = ToolStripStatusLabelBorderSides.Left;

        // Ajouter les éléments à la barre d'état
        this.Items.AddRange(new ToolStripItem[] {
            _labelÉtat, spring, _progressBar, _labelPosition, _labelEncodage
        });
    }

    // Démarrer une tâche avec progression
    public async Task DémarrerTâcheAsync(string message, Func<IProgress<int>, Task> tâche)
    {
        // Afficher le message dans la barre d'état
        TexteÉtat = message;

        // Rendre la barre de progression visible
        _progressBar.Value = 0;
        _progressBar.Visible = true;

        // Créer un rapport de progression
        var progression = new Progress<int>(valeur => {
            _progressBar.Value = valeur;
        });

        try
        {
            // Exécuter la tâche
            await tâche(progression);

            // Afficher un message de succès
            TexteÉtat = "Opération terminée avec succès";
        }
        catch (Exception ex)
        {
            // Afficher un message d'erreur
            TexteÉtat = $"Erreur : {ex.Message}";
        }
        finally
        {
            // Masquer la barre de progression
            await Task.Delay(2000); // Laisser le message visible pendant 2 secondes
            _progressBar.Visible = false;
            TexteÉtat = "Prêt";
        }
    }

    // Classe de rendu personnalisé pour la barre d'état
    private class RenduBarreÉtatPersonnalisé : ToolStripProfessionalRenderer
    {
        public RenduBarreÉtatPersonnalisé() : base(new CouleurBarreÉtat()) { }

        protected override void OnRenderToolStripBorder(ToolStripRenderEventArgs e)
        {
            // Dessiner une bordure personnalisée
            base.OnRenderToolStripBorder(e);

            // Ajouter une ligne supplémentaire en haut
            using (Pen pen = new Pen(Color.FromArgb(180, 180, 180)))
            {
                e.Graphics.DrawLine(pen, 0, 0, e.ToolStrip.Width, 0);
            }
        }

        protected override void OnRenderStatusStripSizingGrip(ToolStripRenderEventArgs e)
        {
            // Personnaliser le grip de redimensionnement
            Rectangle rect = new Rectangle(
                e.ToolStrip.Width - 16, e.ToolStrip.Height - 16, 16, 16);

            using (SolidBrush brush = new SolidBrush(Color.FromArgb(100, 0, 122, 204)))
            {
                // Dessiner 3 petits carrés pour le grip
                e.Graphics.FillRectangle(brush, rect.X + 10, rect.Y + 2, 3, 3);
                e.Graphics.FillRectangle(brush, rect.X + 6, rect.Y + 6, 3, 3);
                e.Graphics.FillRectangle(brush, rect.X + 2, rect.Y + 10, 3, 3);

                e.Graphics.FillRectangle(brush, rect.X + 10, rect.Y + 6, 3, 3);
                e.Graphics.FillRectangle(brush, rect.X + 6, rect.Y + 10, 3, 3);

                e.Graphics.FillRectangle(brush, rect.X + 10, rect.Y + 10, 3, 3);
            }
        }
    }

    // Classe pour personnaliser les couleurs
    private class CouleurBarreÉtat : ProfessionalColorTable
    {
        public override Color StatusStripGradientBegin => Color.FromArgb(240, 240, 240);
        public override Color StatusStripGradientEnd => Color.FromArgb(220, 220, 220);
    }
}
```


## 7.6.4. Actions communes et raccourcis clavier

Pour rendre votre application plus intuitive et efficace, il est important d'implémenter des raccourcis clavier cohérents pour les actions communes.

### Implémentation des raccourcis clavier

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class FormPrincipal : Form
{
    // Dictionnaire associant des actions à leurs raccourcis
    private Dictionary<Keys, Action> _raccourcis = new Dictionary<Keys, Action>();

    public FormPrincipal()
    {
        InitializeComponent();

        // Configurer les raccourcis clavier
        ConfigurerRaccourcis();

        // Intercepter les touches pour gérer les raccourcis
        this.KeyPreview = true;
        this.KeyDown += FormPrincipal_KeyDown;
    }

    private void ConfigurerRaccourcis()
    {
        // Raccourcis de fichier
        _raccourcis[Keys.Control | Keys.N] = NouveauDocument;
        _raccourcis[Keys.Control | Keys.O] = OuvrirDocument;
        _raccourcis[Keys.Control | Keys.S] = EnregistrerDocument;
        _raccourcis[Keys.Control | Keys.Shift | Keys.S] = EnregistrerDocumentSous;
        _raccourcis[Keys.Control | Keys.P] = ImprimerDocument;
        _raccourcis[Keys.Alt | Keys.F4] = () => this.Close();

        // Raccourcis d'édition
        _raccourcis[Keys.Control | Keys.Z] = Annuler;
        _raccourcis[Keys.Control | Keys.Y] = Rétablir;
        _raccourcis[Keys.Control | Keys.X] = Couper;
        _raccourcis[Keys.Control | Keys.C] = Copier;
        _raccourcis[Keys.Control | Keys.V] = Coller;
        _raccourcis[Keys.Delete] = Supprimer;
        _raccourcis[Keys.Control | Keys.A] = SélectionnerTout;
        _raccourcis[Keys.Control | Keys.F] = Rechercher;
        _raccourcis[Keys.F3] = RechercherSuivant;
        _raccourcis[Keys.Control | Keys.H] = Remplacer;

        // Raccourcis d'affichage
        _raccourcis[Keys.Control | Keys.Add] = ZoomPlus;
        _raccourcis[Keys.Control | Keys.Subtract] = ZoomMoins;
        _raccourcis[Keys.Control | Keys.D0] = ZoomNormal;
        _raccourcis[Keys.F11] = BasculeModePleinÉcran;

        // Raccourcis spéciaux
        _raccourcis[Keys.F5] = Actualiser;
        _raccourcis[Keys.F1] = AfficherAide;
    }

    private void FormPrincipal_KeyDown(object sender, KeyEventArgs e)
    {
        // Vérifier si une combinaison de touches est associée à une action
        Keys touche = e.KeyData;

        if (_raccourcis.TryGetValue(touche, out Action action))
        {
            // Exécuter l'action
            action?.Invoke();

            // Marquer l'événement comme traité
            e.Handled = true;
            e.SuppressKeyPress = true;
        }
    }

    // Méthodes pour les actions
    private void NouveauDocument()
    {
        MessageBox.Show("Nouveau document", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void OuvrirDocument()
    {
        OpenFileDialog dlg = new OpenFileDialog();
        dlg.Filter = "Tous les fichiers (*.*)|*.*";
        if (dlg.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show($"Ouverture du fichier : {dlg.FileName}", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }

    private void EnregistrerDocument()
    {
        MessageBox.Show("Enregistrer document", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void EnregistrerDocumentSous()
    {
        SaveFileDialog dlg = new SaveFileDialog();
        dlg.Filter = "Tous les fichiers (*.*)|*.*";
        if (dlg.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show($"Enregistrement du fichier sous : {dlg.FileName}", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }

    private void ImprimerDocument()
    {
        PrintDialog dlg = new PrintDialog();
        if (dlg.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show("Impression en cours...", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }

    private void Annuler()
    {
        MessageBox.Show("Annuler", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Rétablir()
    {
        MessageBox.Show("Rétablir", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Couper()
    {
        MessageBox.Show("Couper", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Copier()
    {
        MessageBox.Show("Copier", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Coller()
    {
        MessageBox.Show("Coller", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Supprimer()
    {
        MessageBox.Show("Supprimer", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void SélectionnerTout()
    {
        MessageBox.Show("Sélectionner tout", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Rechercher()
    {
        MessageBox.Show("Rechercher", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void RechercherSuivant()
    {
        MessageBox.Show("Rechercher suivant", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Remplacer()
    {
        MessageBox.Show("Remplacer", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void ZoomPlus()
    {
        MessageBox.Show("Zoom +", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void ZoomMoins()
    {
        MessageBox.Show("Zoom -", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void ZoomNormal()
    {
        MessageBox.Show("Zoom normal (100%)", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void BasculeModePleinÉcran()
    {
        if (this.FormBorderStyle == FormBorderStyle.None)
        {
            // Rétablir le mode normal
            this.FormBorderStyle = FormBorderStyle.Sizable;
            this.WindowState = FormWindowState.Normal;
            MessageBox.Show("Mode normal", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            // Activer le mode plein écran
            this.FormBorderStyle = FormBorderStyle.None;
            this.WindowState = FormWindowState.Maximized;
            MessageBox.Show("Mode plein écran", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }

    private void Actualiser()
    {
        MessageBox.Show("Actualiser", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void AfficherAide()
    {
        MessageBox.Show("Aide", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }
}
```


### Système d'actions centralisées

Pour une application plus complexe, il est préférable d'utiliser un système d'actions centralisées. Voici un exemple simple d'une classe `GestionnaireActions` :

```textmate
// .NET Framework 4.7.2 et .NET 8
public class Action
{
    public string Nom { get; set; }
    public string Texte { get; set; }
    public string Description { get; set; }
    public Keys Raccourci { get; set; }
    public Image Icône { get; set; }
    public bool EstActivée { get; set; } = true;
    public event EventHandler Exécutée;

    public void Exécuter()
    {
        if (EstActivée)
        {
            Exécutée?.Invoke(this, EventArgs.Empty);
        }
    }
}

public class GestionnaireActions
{
    private Dictionary<string, Action> _actions = new Dictionary<string, Action>();
    private Dictionary<Keys, Action> _raccourcis = new Dictionary<Keys, Action>();

    // Ajouter une action
    public void AjouterAction(Action action)
    {
        if (!string.IsNullOrEmpty(action.Nom))
        {
            _actions[action.Nom] = action;

            // Enregistrer le raccourci si défini
            if (action.Raccourci != Keys.None)
            {
                _raccourcis[action.Raccourci] = action;
            }
        }
    }

    // Obtenir une action par son nom
    public Action ObtenirAction(string nom)
    {
        if (_actions.TryGetValue(nom, out Action action))
        {
            return action;
        }
        return null;
    }

    // Traiter une touche pour déclencher l'action correspondante
    public bool TraiterTouche(Keys touche)
    {
        if (_raccourcis.TryGetValue(touche, out Action action))
        {
            action.Exécuter();
            return true;
        }
        return false;
    }

    // Créer un élément de menu à partir d'une action
    public ToolStripMenuItem CréerÉlémentMenu(string nomAction)
    {
        Action action = ObtenirAction(nomAction);
        if (action == null) return null;

        ToolStripMenuItem item = new ToolStripMenuItem(action.Texte);
        item.ToolTipText = action.Description;
        item.Image = action.Icône;
        item.ShortcutKeys = action.Raccourci;
        item.Enabled = action.EstActivée;

        item.Click += (s, e) => action.Exécuter();

        // Mettre à jour l'élément de menu si l'action change
        action.Exécutée += (s, e) => item.Enabled = action.EstActivée;

        return item;
    }

    // Créer un bouton de barre d'outils à partir d'une action
    public ToolStripButton CréerBoutonToolStrip(string nomAction)
    {
        Action action = ObtenirAction(nomAction);
        if (action == null) return null;

        ToolStripButton bouton = new ToolStripButton(action.Texte);
        bouton.ToolTipText = $"{action.Description} ({action.Raccourci})";
        bouton.Image = action.Icône;
        bouton.Enabled = action.EstActivée;

        bouton.Click += (s, e) => action.Exécuter();

        // Mettre à jour le bouton si l'action change
        action.Exécutée += (s, e) => bouton.Enabled = action.EstActivée;

        return bouton;
    }
}
```


### Utilisation du gestionnaire d'actions

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class FormPrincipal : Form
{
    private GestionnaireActions _gestionnaireActions = new GestionnaireActions();

    public FormPrincipal()
    {
        InitializeComponent();

        // Initialiser les actions
        InitialiserActions();

        // Créer les menus et barres d'outils
        CréerMenus();
        CréerBarreOutils();

        // Configurer le traitement des touches
        this.KeyPreview = true;
        this.KeyDown += (s, e) => {
            if (_gestionnaireActions.TraiterTouche(e.KeyData))
            {
                e.Handled = true;
                e.SuppressKeyPress = true;
            }
        };
    }

    private void InitialiserActions()
    {
        // Actions Fichier
        Action actionNouveau = new Action
        {
            Nom = "fichier.nouveau",
            Texte = "&Nouveau",
            Description = "Créer un nouveau document",
            Raccourci = Keys.Control | Keys.N
        };
        actionNouveau.Exécutée += (s, e) => NouveauDocument();

        Action actionOuvrir = new Action
        {
            Nom = "fichier.ouvrir",
            Texte = "&Ouvrir...",
            Description = "Ouvrir un document existant",
            Raccourci = Keys.Control | Keys.O
        };
        actionOuvrir.Exécutée += (s, e) => OuvrirDocument();

        Action actionEnregistrer = new Action
        {
            Nom = "fichier.enregistrer",
            Texte = "&Enregistrer",
            Description = "Enregistrer le document actuel",
            Raccourci = Keys.Control | Keys.S
        };
        actionEnregistrer.Exécutée += (s, e) => EnregistrerDocument();

        // Ajouter les actions au gestionnaire
        _gestionnaireActions.AjouterAction(actionNouveau);
        _gestionnaireActions.AjouterAction(actionOuvrir);
        _gestionnaireActions.AjouterAction(actionEnregistrer);

        // Actions Edition
        Action actionCouper = new Action
        {
            Nom = "edition.couper",
            Texte = "Co&uper",
            Description = "Couper la sélection",
            Raccourci = Keys.Control | Keys.X
        };
        actionCouper.Exécutée += (s, e) => Couper();

        Action actionCopier = new Action
        {
            Nom = "edition.copier",
            Texte = "&Copier",
            Description = "Copier la sélection",
            Raccourci = Keys.Control | Keys.C
        };
        actionCopier.Exécutée += (s, e) => Copier();

        Action actionColler = new Action
        {
            Nom = "edition.coller",
            Texte = "C&oller",
            Description = "Coller le contenu du presse-papiers",
            Raccourci = Keys.Control | Keys.V
        };
        actionColler.Exécutée += (s, e) => Coller();

        // Ajouter les actions au gestionnaire
        _gestionnaireActions.AjouterAction(actionCouper);
        _gestionnaireActions.AjouterAction(actionCopier);
        _gestionnaireActions.AjouterAction(actionColler);
    }

    private void CréerMenus()
    {
        // Création du MenuStrip
        MenuStrip menuStrip = new MenuStrip();
        menuStrip.Dock = DockStyle.Top;

        // Menu Fichier
        ToolStripMenuItem menuFichier = new ToolStripMenuItem("&Fichier");
        menuFichier.DropDownItems.Add(_gestionnaireActions.CréerÉlémentMenu("fichier.nouveau"));
        menuFichier.DropDownItems.Add(_gestionnaireActions.CréerÉlémentMenu("fichier.ouvrir"));
        menuFichier.DropDownItems.Add(_gestionnaireActions.CréerÉlémentMenu("fichier.enregistrer"));

        // Menu Edition
        ToolStripMenuItem menuEdition = new ToolStripMenuItem("&Edition");
        menuEdition.DropDownItems.Add(_gestionnaireActions.CréerÉlémentMenu("edition.couper"));
        menuEdition.DropDownItems.Add(_gestionnaireActions.CréerÉlémentMenu("edition.copier"));
        menuEdition.DropDownItems.Add(_gestionnaireActions.CréerÉlémentMenu("edition.coller"));

        // Ajouter les menus au MenuStrip
        menuStrip.Items.Add(menuFichier);
        menuStrip.Items.Add(menuEdition);

        // Ajouter le MenuStrip au formulaire
        this.Controls.Add(menuStrip);
    }

    private void CréerBarreOutils()
    {
        // Création de la barre d'outils
        ToolStrip toolStrip = new ToolStrip();
        toolStrip.Dock = DockStyle.Top;

        // Ajouter les boutons
        toolStrip.Items.Add(_gestionnaireActions.CréerBoutonToolStrip("fichier.nouveau"));
        toolStrip.Items.Add(_gestionnaireActions.CréerBoutonToolStrip("fichier.ouvrir"));
        toolStrip.Items.Add(_gestionnaireActions.CréerBoutonToolStrip("fichier.enregistrer"));

        toolStrip.Items.Add(new ToolStripSeparator());

        toolStrip.Items.Add(_gestionnaireActions.CréerBoutonToolStrip("edition.couper"));
        toolStrip.Items.Add(_gestionnaireActions.CréerBoutonToolStrip("edition.copier"));
        toolStrip.Items.Add(_gestionnaireActions.CréerBoutonToolStrip("edition.coller"));

        // Ajouter la barre d'outils au formulaire
        this.Controls.Add(toolStrip);
    }

    // Méthodes pour les actions
    private void NouveauDocument()
    {
        MessageBox.Show("Nouveau document", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void OuvrirDocument()
    {
        OpenFileDialog dlg = new OpenFileDialog();
        dlg.Filter = "Tous les fichiers (*.*)|*.*";
        if (dlg.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show($"Ouverture du fichier : {dlg.FileName}", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }

    private void EnregistrerDocument()
    {
        MessageBox.Show("Enregistrer document", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Couper()
    {
        MessageBox.Show("Couper", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Copier()
    {
        MessageBox.Show("Copier", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    private void Coller()
    {
        MessageBox.Show("Coller", "Action", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }
}
```


### Raccourcis clavier standard pour les applications Windows

Voici une liste des raccourcis clavier standard que les utilisateurs s'attendent à retrouver dans une application Windows :

```textmate
// .NET Framework 4.7.2 et .NET 8
public static class RaccourcisStandard
{
    // Raccourcis de fichier
    public static readonly Keys Nouveau = Keys.Control | Keys.N;
    public static readonly Keys Ouvrir = Keys.Control | Keys.O;
    public static readonly Keys Enregistrer = Keys.Control | Keys.S;
    public static readonly Keys EnregistrerSous = Keys.Control | Keys.Shift | Keys.S;
    public static readonly Keys Imprimer = Keys.Control | Keys.P;
    public static readonly Keys Quitter = Keys.Alt | Keys.F4;

    // Raccourcis d'édition
    public static readonly Keys Annuler = Keys.Control | Keys.Z;
    public static readonly Keys Rétablir = Keys.Control | Keys.Y;
    public static readonly Keys Couper = Keys.Control | Keys.X;
    public static readonly Keys Copier = Keys.Control | Keys.C;
    public static readonly Keys Coller = Keys.Control | Keys.V;
    public static readonly Keys Supprimer = Keys.Delete;
    public static readonly Keys SélectionnerTout = Keys.Control | Keys.A;

    // Raccourcis de recherche
    public static readonly Keys Rechercher = Keys.Control | Keys.F;
    public static readonly Keys RechercherSuivant = Keys.F3;
    public static readonly Keys Remplacer = Keys.Control | Keys.H;
    public static readonly Keys AllerÀ = Keys.Control | Keys.G;

    // Raccourcis de formatage (pour éditeurs de texte)
    public static readonly Keys Gras = Keys.Control | Keys.B;
    public static readonly Keys Italique = Keys.Control | Keys.I;
    public static readonly Keys Souligné = Keys.Control | Keys.U;

    // Raccourcis d'affichage
    public static readonly Keys ZoomPlus = Keys.Control | Keys.Add;
    public static readonly Keys ZoomMoins = Keys.Control | Keys.Subtract;
    public static readonly Keys ZoomNormal = Keys.Control | Keys.D0;
    public static readonly Keys PleinÉcran = Keys.F11;

    // Raccourcis de navigation
    public static readonly Keys Précédent = Keys.Alt | Keys.Left;
    public static readonly Keys Suivant = Keys.Alt | Keys.Right;
    public static readonly Keys PageSuivante = Keys.Control | Keys.Tab;
    public static readonly Keys PagePrécédente = Keys.Control | Keys.Shift | Keys.Tab;

    // Raccourcis d'aide
    public static readonly Keys Aide = Keys.F1;

    // Raccourcis système
    public static readonly Keys Actualiser = Keys.F5;
    public static readonly Keys Propriétés = Keys.Alt | Keys.Enter;

    // Méthode utilitaire pour obtenir une représentation textuelle d'un raccourci
    public static string ObtenirTexteRaccourci(Keys raccourci)
    {
        string texte = string.Empty;

        if ((raccourci & Keys.Control) == Keys.Control)
            texte += "Ctrl+";

        if ((raccourci & Keys.Shift) == Keys.Shift)
            texte += "Maj+";

        if ((raccourci & Keys.Alt) == Keys.Alt)
            texte += "Alt+";

        // Extraire la touche sans les modificateurs
        Keys touchePrincipale = raccourci & Keys.KeyCode;

        // Traitement spécial pour certaines touches
        switch (touchePrincipale)
        {
            case Keys.Add:
                texte += "+";
                break;
            case Keys.Subtract:
                texte += "-";
                break;
            case Keys.D0:
            case Keys.D1:
            case Keys.D2:
            case Keys.D3:
            case Keys.D4:
            case Keys.D5:
            case Keys.D6:
            case Keys.D7:
            case Keys.D8:
            case Keys.D9:
                texte += touchePrincipale.ToString().Replace("D", "");
                break;
            default:
                texte += touchePrincipale.ToString();
                break;
        }

        return texte;
    }
}
```


### Résumé

Les menus et barres d'outils sont des éléments essentiels de l'interface utilisateur Windows qui permettent d'organiser et de présenter les fonctionnalités de l'application de manière cohérente et accessible. En combinant menus, barres d'outils et raccourcis clavier, vous offrez à vos utilisateurs plusieurs façons d'accéder aux mêmes fonctions, ce qui améliore l'ergonomie et l'efficacité de votre application.

Les contrôles `MenuStrip`, `ContextMenuStrip`, `ToolStrip` et `StatusStrip` de Windows Forms offrent une grande flexibilité pour créer des interfaces utilisateur professionnelles et personnalisées. La mise en place d'un système d'actions centralisé permet de maintenir une cohérence entre ces différents éléments et facilite la maintenance du code.

Assurez-vous de suivre les conventions standard de Windows pour les raccourcis clavier afin que vos utilisateurs puissent utiliser leur expérience acquise avec d'autres applications pour naviguer rapidement dans la vôtre.

⏭️ 7.7. [Dialogues et messages](/07-windows-forms-winforms/7-7-dialogues-et-messages.md)
