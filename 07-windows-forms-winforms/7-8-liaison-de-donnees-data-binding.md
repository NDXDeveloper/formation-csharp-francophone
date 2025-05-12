# 7.8. Liaison de données (Data Binding)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La liaison de données (Data Binding) est une fonctionnalité puissante de Windows Forms qui permet de connecter les contrôles de l'interface utilisateur aux sources de données sous-jacentes. Elle simplifie considérablement la synchronisation entre l'affichage et les données, réduisant ainsi la quantité de code à écrire et à maintenir.

## 7.8.1. Binding simple

Le binding simple permet de lier une propriété d'un contrôle à une propriété d'un objet ou à une valeur spécifique.

### Liaison d'un contrôle à un objet simple

```textmate
// .NET Framework 4.7.2 et .NET 8
public class Personne
{
    public string Nom { get; set; }
    public string Prénom { get; set; }
    public DateTime DateNaissance { get; set; }
    public string Email { get; set; }
    public bool Actif { get; set; }

    public int Age
    {
        get
        {
            int age = DateTime.Today.Year - DateNaissance.Year;
            if (DateNaissance.Date > DateTime.Today.AddYears(-age))
                age--;
            return age;
        }
    }
}

private void ExempleBindingSimple()
{
    // Créer un objet à lier
    Personne personne = new Personne
    {
        Nom = "Dupont",
        Prénom = "Jean",
        DateNaissance = new DateTime(1980, 5, 15),
        Email = "jean.dupont@example.com",
        Actif = true
    };

    // Créer un contrôle pour chaque propriété
    TextBox txtNom = new TextBox
    {
        Location = new Point(120, 20),
        Width = 200
    };

    TextBox txtPrénom = new TextBox
    {
        Location = new Point(120, 50),
        Width = 200
    };

    DateTimePicker dtpDateNaissance = new DateTimePicker
    {
        Location = new Point(120, 80),
        Width = 200,
        Format = DateTimePickerFormat.Short
    };

    TextBox txtEmail = new TextBox
    {
        Location = new Point(120, 110),
        Width = 200
    };

    CheckBox chkActif = new CheckBox
    {
        Location = new Point(120, 140),
        Text = "Compte actif"
    };

    Label lblAge = new Label
    {
        Location = new Point(120, 170),
        AutoSize = true
    };

    // Ajouter des labels descriptifs
    this.Controls.Add(new Label { Text = "Nom :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "Prénom :", Location = new Point(20, 53), AutoSize = true });
    this.Controls.Add(new Label { Text = "Date de naissance :", Location = new Point(20, 83), AutoSize = true });
    this.Controls.Add(new Label { Text = "Email :", Location = new Point(20, 113), AutoSize = true });
    this.Controls.Add(new Label { Text = "Statut :", Location = new Point(20, 140), AutoSize = true });
    this.Controls.Add(new Label { Text = "Âge :", Location = new Point(20, 170), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(txtNom);
    this.Controls.Add(txtPrénom);
    this.Controls.Add(dtpDateNaissance);
    this.Controls.Add(txtEmail);
    this.Controls.Add(chkActif);
    this.Controls.Add(lblAge);

    // Créer des liaisons simples entre les propriétés et les contrôles
    txtNom.DataBindings.Add("Text", personne, "Nom");
    txtPrénom.DataBindings.Add("Text", personne, "Prénom");
    dtpDateNaissance.DataBindings.Add("Value", personne, "DateNaissance");
    txtEmail.DataBindings.Add("Text", personne, "Email");
    chkActif.DataBindings.Add("Checked", personne, "Actif");
    lblAge.DataBindings.Add("Text", personne, "Age");

    // Ajouter un bouton pour afficher les données actuelles
    Button btnAfficher = new Button
    {
        Text = "Afficher les données",
        Location = new Point(120, 210),
        Width = 150
    };

    btnAfficher.Click += (s, e) =>
    {
        MessageBox.Show(
            $"Nom: {personne.Nom}\n" +
            $"Prénom: {personne.Prénom}\n" +
            $"Date de naissance: {personne.DateNaissance.ToShortDateString()}\n" +
            $"Âge: {personne.Age} ans\n" +
            $"Email: {personne.Email}\n" +
            $"Actif: {personne.Actif}",
            "Données actuelles",
            MessageBoxButtons.OK,
            MessageBoxIcon.Information);
    };

    this.Controls.Add(btnAfficher);
}
```


### Options de liaison avancées

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExempleOptionsLiaisonAvancées()
{
    // Créer un objet à lier
    Personne personne = new Personne
    {
        Nom = "Martin",
        Prénom = "Sophie",
        DateNaissance = new DateTime(1992, 8, 23),
        Email = "sophie.martin@example.com",
        Actif = true
    };

    // Créer des contrôles pour les données
    TextBox txtNomComplet = new TextBox
    {
        Location = new Point(120, 20),
        Width = 250
    };

    Label lblDateNaissance = new Label
    {
        Location = new Point(120, 50),
        AutoSize = true
    };

    TextBox txtEmail = new TextBox
    {
        Location = new Point(120, 80),
        Width = 250
    };

    // Ajouter des labels descriptifs
    this.Controls.Add(new Label { Text = "Nom complet :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "Née le :", Location = new Point(20, 50), AutoSize = true });
    this.Controls.Add(new Label { Text = "Email :", Location = new Point(20, 83), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(txtNomComplet);
    this.Controls.Add(lblDateNaissance);
    this.Controls.Add(txtEmail);

    // 1. Liaison avec formatage
    Binding bindingDate = new Binding("Text", personne, "DateNaissance");
    bindingDate.FormatString = "dd MMMM yyyy (il y a {0} ans)";
    bindingDate.FormattingEnabled = true;
    bindingDate.Format += (s, e) => {
        if (e.Value is DateTime date)
        {
            int age = DateTime.Today.Year - date.Year;
            if (date.Date > DateTime.Today.AddYears(-age))
                age--;

            // Format personnalisé avec l'âge
            e.Value = string.Format(e.Format, age);
        }
    };
    lblDateNaissance.DataBindings.Add(bindingDate);

    // 2. Liaison avec conversion
    Binding bindingNomComplet = new Binding("Text", personne, "Nom");
    bindingNomComplet.Format += (s, e) => {
        // Conversion à l'affichage (de l'objet vers le contrôle)
        if (e.Value is string nom)
        {
            e.Value = $"{personne.Prénom} {nom.ToUpper()}";
        }
    };
    bindingNomComplet.Parse += (s, e) => {
        // Conversion lors de la saisie (du contrôle vers l'objet)
        if (e.Value is string nomComplet)
        {
            string[] parts = nomComplet.Split(' ');
            if (parts.Length >= 2)
            {
                personne.Prénom = parts[0];
                personne.Nom = parts[1];
            }
            else
            {
                personne.Nom = nomComplet;
            }

            // La valeur à assigner à la propriété
            e.Value = personne.Nom;
        }
    };
    txtNomComplet.DataBindings.Add(bindingNomComplet);

    // 3. Liaison avec validation
    Binding bindingEmail = new Binding("Text", personne, "Email", true, DataSourceUpdateMode.OnValidation);
    bindingEmail.Parse += (s, e) => {
        if (e.Value is string email)
        {
            // Validation simple de l'email
            if (!email.Contains("@") || !email.Contains("."))
            {
                throw new FormatException("L'adresse email doit contenir '@' et '.'");
            }
        }
    };
    txtEmail.DataBindings.Add(bindingEmail);

    // Gestion des erreurs de parsing/validation
    bindingEmail.BindingComplete += (s, e) => {
        if (e.Exception != null && !e.Handled)
        {
            MessageBox.Show(e.Exception.Message, "Erreur de validation",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
            e.Handled = true;  // Marquer l'erreur comme traitée
        }
    };

    // Ajouter un bouton pour afficher les données actuelles
    Button btnAfficher = new Button
    {
        Text = "Afficher les données",
        Location = new Point(120, 120),
        Width = 150
    };

    btnAfficher.Click += (s, e) =>
    {
        MessageBox.Show(
            $"Nom: {personne.Nom}\n" +
            $"Prénom: {personne.Prénom}\n" +
            $"Date de naissance: {personne.DateNaissance.ToShortDateString()}\n" +
            $"Email: {personne.Email}",
            "Données actuelles",
            MessageBoxButtons.OK,
            MessageBoxIcon.Information);
    };

    this.Controls.Add(btnAfficher);
}
```


### Modes de mise à jour de la source de données

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExemplesModesMiseÀJour()
{
    // Créer un objet à lier
    Personne personne = new Personne
    {
        Nom = "Petit",
        Prénom = "Marie",
        Email = "marie.petit@example.com"
    };

    // Créer des contrôles pour illustrer différents modes de mise à jour
    TextBox txtOnPropertyChanged = new TextBox
    {
        Location = new Point(220, 20),
        Width = 200
    };

    TextBox txtOnValidation = new TextBox
    {
        Location = new Point(220, 50),
        Width = 200
    };

    TextBox txtNever = new TextBox
    {
        Location = new Point(220, 80),
        Width = 200
    };

    TextBox txtExplicite = new TextBox
    {
        Location = new Point(220, 110),
        Width = 200
    };

    Button btnMettreÀJour = new Button
    {
        Text = "Mettre à jour",
        Location = new Point(430, 110),
        Width = 100
    };

    // Ajouter des labels descriptifs
    this.Controls.Add(new Label { Text = "OnPropertyChanged (immédiat) :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "OnValidation (perte focus) :", Location = new Point(20, 53), AutoSize = true });
    this.Controls.Add(new Label { Text = "Never (jamais automatique) :", Location = new Point(20, 83), AutoSize = true });
    this.Controls.Add(new Label { Text = "Explicite (bouton) :", Location = new Point(20, 113), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(txtOnPropertyChanged);
    this.Controls.Add(txtOnValidation);
    this.Controls.Add(txtNever);
    this.Controls.Add(txtExplicite);
    this.Controls.Add(btnMettreÀJour);

    // Créer des liaisons avec différents modes de mise à jour

    // 1. OnPropertyChanged - Met à jour la source immédiatement à chaque changement
    Binding bindingImmediate = new Binding("Text", personne, "Nom", false,
                                          DataSourceUpdateMode.OnPropertyChanged);
    txtOnPropertyChanged.DataBindings.Add(bindingImmediate);

    // 2. OnValidation - Met à jour quand le contrôle perd le focus
    Binding bindingValidation = new Binding("Text", personne, "Prénom", false,
                                           DataSourceUpdateMode.OnValidation);
    txtOnValidation.DataBindings.Add(bindingValidation);

    // 3. Never - Ne met jamais à jour automatiquement
    Binding bindingNever = new Binding("Text", personne, "Email", false,
                                      DataSourceUpdateMode.Never);
    txtNever.DataBindings.Add(bindingNever);

    // 4. Binding pour mise à jour explicite
    Binding bindingExplicite = new Binding("Text", personne, "Email", false,
                                          DataSourceUpdateMode.Never);
    txtExplicite.DataBindings.Add(bindingExplicite);

    // Bouton pour mettre à jour explicitement
    btnMettreÀJour.Click += (s, e) => {
        // Mettre à jour manuellement le binding
        bindingExplicite.WriteValue();
        MessageBox.Show("Valeur mise à jour explicitement.", "Information",
                       MessageBoxButtons.OK, MessageBoxIcon.Information);
    };

    // Ajouter un bouton pour afficher l'état actuel de l'objet
    Button btnAfficherÉtat = new Button
    {
        Text = "Afficher l'état actuel",
        Location = new Point(220, 150),
        Width = 150
    };

    btnAfficherÉtat.Click += (s, e) =>
    {
        MessageBox.Show(
            $"État actuel de l'objet :\n" +
            $"Nom: {personne.Nom}\n" +
            $"Prénom: {personne.Prénom}\n" +
            $"Email: {personne.Email}",
            "État de l'objet",
            MessageBoxButtons.OK,
            MessageBoxIcon.Information);
    };

    this.Controls.Add(btnAfficherÉtat);

    // Ajouter un bouton pour rafraîchir les contrôles depuis l'objet
    Button btnRafraîchir = new Button
    {
        Text = "Rafraîchir depuis l'objet",
        Location = new Point(380, 150),
        Width = 150
    };

    btnRafraîchir.Click += (s, e) =>
    {
        // Modifier directement l'objet
        personne.Nom = "Nouveau_Nom";
        personne.Prénom = "Nouveau_Prénom";
        personne.Email = "nouveau@exemple.com";

        // Rafraîchir manuellement les contrôles
        bindingImmediate.ReadValue();
        bindingValidation.ReadValue();
        bindingNever.ReadValue();
        bindingExplicite.ReadValue();

        MessageBox.Show("Contrôles rafraîchis depuis l'objet modifié.", "Information",
                       MessageBoxButtons.OK, MessageBoxIcon.Information);
    };

    this.Controls.Add(btnRafraîchir);
}
```


## 7.8.2. Binding complexe

Le binding complexe permet de lier des contrôles à des collections d'objets, comme les contrôles de type liste (ListBox, ComboBox, DataGridView, etc.).

### Liaison à une liste d'objets

```textmate
// .NET Framework 4.7.2 et .NET 8
public class Produit
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public string Catégorie { get; set; }
    public decimal Prix { get; set; }
    public int Stock { get; set; }
    public bool Disponible { get; set; }

    public override string ToString()
    {
        return $"{Nom} ({Prix:C})";
    }
}

private void ExempleBindingComplexe()
{
    // Créer une liste de produits
    List<Produit> produits = new List<Produit>
    {
        new Produit { Id = 1, Nom = "Ordinateur portable", Catégorie = "Informatique", Prix = 899.99m, Stock = 15, Disponible = true },
        new Produit { Id = 2, Nom = "Souris sans fil", Catégorie = "Accessoires", Prix = 24.99m, Stock = 50, Disponible = true },
        new Produit { Id = 3, Nom = "Écran 24\"", Catégorie = "Informatique", Prix = 199.50m, Stock = 8, Disponible = true },
        new Produit { Id = 4, Nom = "Clavier mécanique", Catégorie = "Accessoires", Prix = 79.90m, Stock = 25, Disponible = true },
        new Produit { Id = 5, Nom = "Imprimante laser", Catégorie = "Informatique", Prix = 249.99m, Stock = 0, Disponible = false }
    };

    // Créer un ComboBox pour sélectionner un produit
    ComboBox cboProduitsSimple = new ComboBox
    {
        Location = new Point(150, 20),
        Width = 250,
        DropDownStyle = ComboBoxStyle.DropDownList
    };

    // Lier le ComboBox à la liste de produits
    cboProduitsSimple.DataSource = produits;
    // Optionnel : définir quelle propriété afficher et quelle propriété utiliser comme valeur
    cboProduitsSimple.DisplayMember = "Nom";       // Affiche la propriété Nom
    cboProduitsSimple.ValueMember = "Id";          // Utilise la propriété Id comme valeur

    // Créer un ListBox pour afficher les produits
    ListBox lstProduits = new ListBox
    {
        Location = new Point(150, 60),
        Width = 250,
        Height = 100
    };

    // Lier le ListBox à la liste de produits
    lstProduits.DataSource = produits;
    lstProduits.DisplayMember = "Nom";
    lstProduits.ValueMember = "Id";

    // Créer un DataGridView pour afficher tous les détails
    DataGridView dgvProduits = new DataGridView
    {
        Location = new Point(150, 180),
        Width = 500,
        Height = 200,
        AllowUserToAddRows = false,
        AllowUserToDeleteRows = false,
        ReadOnly = true,
        SelectionMode = DataGridViewSelectionMode.FullRowSelect,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
    };

    // Lier le DataGridView à la liste de produits
    dgvProduits.DataSource = produits;

    // Personnaliser l'affichage des colonnes
    dgvProduits.Columns["Prix"].DefaultCellStyle.Format = "C";  // Format monétaire

    // Ajouter des contrôles pour afficher les détails du produit sélectionné
    TextBox txtNom = new TextBox { Location = new Point(150, 400), Width = 200 };
    TextBox txtPrix = new TextBox { Location = new Point(150, 430), Width = 100 };
    CheckBox chkDisponible = new CheckBox { Location = new Point(150, 460), Text = "Disponible" };

    // Lier ces contrôles au produit sélectionné dans le DataGridView
    BindingSource bs = new BindingSource();
    bs.DataSource = produits;
    dgvProduits.DataSource = bs;

    txtNom.DataBindings.Add("Text", bs, "Nom");
    txtPrix.DataBindings.Add("Text", bs, "Prix", true, DataSourceUpdateMode.OnValidation, 0, "C");
    chkDisponible.DataBindings.Add("Checked", bs, "Disponible");

    // Synchroniser la sélection entre ListBox et DataGridView
    lstProduits.SelectedIndexChanged += (s, e) => {
        bs.Position = lstProduits.SelectedIndex;
    };

    bs.PositionChanged += (s, e) => {
        lstProduits.SelectedIndex = bs.Position;
    };

    // Ajouter des labels descriptifs
    this.Controls.Add(new Label { Text = "ComboBox :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "ListBox :", Location = new Point(20, 60), AutoSize = true });
    this.Controls.Add(new Label { Text = "DataGridView :", Location = new Point(20, 180), AutoSize = true });
    this.Controls.Add(new Label { Text = "Détails :", Location = new Point(20, 400), AutoSize = true });
    this.Controls.Add(new Label { Text = "Nom :", Location = new Point(50, 403), AutoSize = true });
    this.Controls.Add(new Label { Text = "Prix :", Location = new Point(50, 433), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(cboProduitsSimple);
    this.Controls.Add(lstProduits);
    this.Controls.Add(dgvProduits);
    this.Controls.Add(txtNom);
    this.Controls.Add(txtPrix);
    this.Controls.Add(chkDisponible);

    // Ajouter un bouton pour appliquer les modifications
    Button btnAppliquer = new Button
    {
        Text = "Appliquer les modifications",
        Location = new Point(260, 460),
        Width = 180
    };

    btnAppliquer.Click += (s, e) => {
        try
        {
            // Valider et appliquer les changements
            bs.EndEdit();
            MessageBox.Show("Modifications appliquées.", "Succès",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);

            // Rafraîchir l'affichage
            dgvProduits.Refresh();
            lstProduits.Refresh();
            cboProduitsSimple.Refresh();
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur lors de l'application des modifications: {ex.Message}",
                           "Erreur", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    };

    this.Controls.Add(btnAppliquer);
}
```


### Binding hiérarchique

```textmate
// .NET Framework 4.7.2 et .NET 8
public class Catégorie
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public List<Produit> Produits { get; set; } = new List<Produit>();

    public override string ToString()
    {
        return Nom;
    }
}

private void ExempleBindingHiérarchique()
{
    // Créer des catégories et produits
    List<Catégorie> catégories = new List<Catégorie>
    {
        new Catégorie
        {
            Id = 1,
            Nom = "Informatique",
            Produits = new List<Produit>
            {
                new Produit { Id = 1, Nom = "Ordinateur portable", Prix = 899.99m, Stock = 15, Disponible = true },
                new Produit { Id = 3, Nom = "Écran 24\"", Prix = 199.50m, Stock = 8, Disponible = true },
                new Produit { Id = 5, Nom = "Imprimante laser", Prix = 249.99m, Stock = 0, Disponible = false }
            }
        },
        new Catégorie
        {
            Id = 2,
            Nom = "Accessoires",
            Produits = new List<Produit>
            {
                new Produit { Id = 2, Nom = "Souris sans fil", Prix = 24.99m, Stock = 50, Disponible = true },
                new Produit { Id = 4, Nom = "Clavier mécanique", Prix = 79.90m, Stock = 25, Disponible = true }
            }
        },
        new Catégorie
        {
            Id = 3,
            Nom = "Logiciels",
            Produits = new List<Produit>
            {
                new Produit { Id = 6, Nom = "Suite bureautique", Prix = 129.99m, Stock = 100, Disponible = true },
                new Produit { Id = 7, Nom = "Antivirus", Prix = 49.90m, Stock = 200, Disponible = true }
            }
        }
    };

    // Créer une source de binding pour les catégories
    BindingSource bsCatégories = new BindingSource();
    bsCatégories.DataSource = catégories;

    // Créer une source de binding pour les produits (liée aux catégories)
    BindingSource bsProduits = new BindingSource();
    bsProduits.DataSource = bsCatégories;
    bsProduits.DataMember = "Produits";

    // Créer un ComboBox pour sélectionner une catégorie
    ComboBox cboCatégories = new ComboBox
    {
        Location = new Point(150, 20),
        Width = 200,
        DropDownStyle = ComboBoxStyle.DropDownList
    };

    // Lier le ComboBox aux catégories
    cboCatégories.DataSource = bsCatégories;
    cboCatégories.DisplayMember = "Nom";
    cboCatégories.ValueMember = "Id";

    // Créer un DataGridView pour afficher les produits de la catégorie sélectionnée
    DataGridView dgvProduits = new DataGridView
    {
        Location = new Point(150, 60),
        Width = 500,
        Height = 200,
        AllowUserToAddRows = false,
        AllowUserToDeleteRows = false,
        SelectionMode = DataGridViewSelectionMode.FullRowSelect,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
    };

    // Lier le DataGridView aux produits de la catégorie sélectionnée
    dgvProduits.DataSource = bsProduits;

    // Personnaliser l'affichage
    dgvProduits.Columns["Prix"].DefaultCellStyle.Format = "C";
    dgvProduits.Columns["Catégorie"].Visible = false;  // Masquer la colonne Catégorie (redondante)

    // Ajouter des contrôles de détail pour le produit sélectionné
    Label lblDétailProduit = new Label
    {
        Text = "Détails du produit sélectionné :",
        Location = new Point(150, 280),
        AutoSize = true,
        Font = new Font(this.Font, FontStyle.Bold)
    };

    Label lblNom = new Label { Text = "Nom :", Location = new Point(180, 310), AutoSize = true };
    TextBox txtNom = new TextBox { Location = new Point(250, 310), Width = 200 };

    Label lblPrix = new Label { Text = "Prix :", Location = new Point(180, 340), AutoSize = true };
    TextBox txtPrix = new TextBox { Location = new Point(250, 340), Width = 100 };

    Label lblStock = new Label { Text = "Stock :", Location = new Point(180, 370), AutoSize = true };
    NumericUpDown nudStock = new NumericUpDown { Location = new Point(250, 370), Width = 100, Minimum = 0, Maximum = 1000 };

    CheckBox chkDisponible = new CheckBox { Text = "Disponible", Location = new Point(250, 400) };

    // Lier les contrôles de détail au produit sélectionné
    txtNom.DataBindings.Add("Text", bsProduits, "Nom");
    txtPrix.DataBindings.Add("Text", bsProduits, "Prix", true, DataSourceUpdateMode.OnValidation, 0, "C");
    nudStock.DataBindings.Add("Value", bsProduits, "Stock");
    chkDisponible.DataBindings.Add("Checked", bsProduits, "Disponible");

    // Bouton pour appliquer les modifications
    Button btnAppliquer = new Button
    {
        Text = "Appliquer les modifications",
        Location = new Point(250, 430),
        Width = 180
    };

    btnAppliquer.Click += (s, e) => {
        try
        {
            bsProduits.EndEdit();
            MessageBox.Show("Modifications appliquées.", "Succès",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur: {ex.Message}", "Erreur",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    };

    // Bouton pour annuler les modifications
    Button btnAnnuler = new Button
    {
        Text = "Annuler les modifications",
        Location = new Point(440, 430),
        Width = 180
    };

    btnAnnuler.Click += (s, e) => {
        bsProduits.CancelEdit();
        MessageBox.Show("Modifications annulées.", "Information",
                       MessageBoxButtons.OK, MessageBoxIcon.Information);
    };

    // Ajouter un bouton pour ajouter un nouveau produit
    Button btnNouveau = new Button
    {
        Text = "Nouveau produit",
        Location = new Point(250, 470),
        Width = 180
    };

    btnNouveau.Click += (s, e) => {
        // Récupérer la catégorie sélectionnée
        Catégorie catégorieSélectionnée = (Catégorie)bsCatégories.Current;
        if (catégorieSélectionnée != null)
        {
            // Créer un nouveau produit
            Produit nouveauProduit = new Produit
            {
                Id = catégorieSélectionnée.Produits.Count > 0
                    ? catégorieSélectionnée.Produits.Max(p => p.Id) + 1
                    : 1,
                Nom = "Nouveau produit",
                Prix = 0,
                Stock = 0,
                Disponible = false,
                Catégorie = catégorieSélectionnée.Nom
            };

            // Ajouter le produit à la catégorie
            catégorieSélectionnée.Produits.Add(nouveauProduit);

            // Rafraîchir la source de binding
            bsProduits.ResetBindings(false);

            // Sélectionner le nouveau produit
            bsProduits.Position = bsProduits.Count - 1;

            MessageBox.Show("Nouveau produit ajouté.", "Succès",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    };

    // Bouton pour supprimer un produit
    Button btnSupprimer = new Button
    {
        Text = "Supprimer produit",
        Location = new Point(440, 470),
        Width = 180
    };

    btnSupprimer.Click += (s, e) => {
        if (bsProduits.Current != null)
        {
            DialogResult result = MessageBox.Show(
                "Êtes-vous sûr de vouloir supprimer ce produit ?",
                "Confirmation de suppression",
                MessageBoxButtons.YesNo,
                MessageBoxIcon.Question);

            if (result == DialogResult.Yes)
            {
                // Récupérer la catégorie et le produit sélectionnés
                Catégorie catégorieSélectionnée = (Catégorie)bsCatégories.Current;
                Produit produitSélectionné = (Produit)bsProduits.Current;

                // Supprimer le produit
                catégorieSélectionnée.Produits.Remove(produitSélectionné);

                // Rafraîchir la source de binding
                bsProduits.ResetBindings(false);

                MessageBox.Show("Produit supprimé.", "Succès",
                               MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }
        else
        {
            MessageBox.Show("Aucun produit sélectionné.", "Erreur",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    };

    // Ajouter des labels descriptifs
    this.Controls.Add(new Label { Text = "Catégorie :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "Produits :", Location = new Point(20, 60), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(cboCatégories);
    this.Controls.Add(dgvProduits);
    this.Controls.Add(lblDétailProduit);
    this.Controls.Add(lblNom);
    this.Controls.Add(txtNom);
    this.Controls.Add(lblPrix);
    this.Controls.Add(txtPrix);
    this.Controls.Add(lblStock);
    this.Controls.Add(nudStock);
    this.Controls.Add(chkDisponible);
    this.Controls.Add(btnAppliquer);
    this.Controls.Add(btnAnnuler);
    this.Controls.Add(btnNouveau);
    this.Controls.Add(btnSupprimer);
}
```


### Binding avec filtrage et tri

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExempleBindingAvecFiltrageEtTri()
{
    // Créer une liste de produits
    List<Produit> produits = new List<Produit>
    {
        new Produit { Id = 1, Nom = "Ordinateur portable", Catégorie = "Informatique", Prix = 899.99m, Stock = 15, Disponible = true },
        new Produit { Id = 2, Nom = "Souris sans fil", Catégorie = "Accessoires", Prix = 24.99m, Stock = 50, Disponible = true },
        new Produit { Id = 3, Nom = "Écran 24\"", Catégorie = "Informatique", Prix = 199.50m, Stock = 8, Disponible = true },
        new Produit { Id = 4, Nom = "Clavier mécanique", Catégorie = "Accessoires", Prix = 79.90m, Stock = 25, Disponible = true },
        new Produit { Id = 5, Nom = "Imprimante laser", Catégorie = "Informatique", Prix = 249.99m, Stock = 0, Disponible = false },
        new Produit { Id = 6, Nom = "Suite bureautique", Catégorie = "Logiciels", Prix = 129.99m, Stock = 100, Disponible = true },
        new Produit { Id = 7, Nom = "Antivirus", Catégorie = "Logiciels", Prix = 49.90m, Stock = 200, Disponible = true },
        new Produit { Id = 8, Nom = "Disque dur externe", Catégorie = "Stockage", Prix = 89.99m, Stock = 30, Disponible = true },
        new Produit { Id = 9, Nom = "Clé USB 64 Go", Catégorie = "Stockage", Prix = 19.99m, Stock = 100, Disponible = true },
        new Produit { Id = 10, Nom = "Carte graphique", Catégorie = "Informatique", Prix = 349.99m, Stock = 5, Disponible = true }
    };

    // Créer une source de binding
    BindingSource bs = new BindingSource();
    bs.DataSource = produits;

    // Créer un DataGridView pour afficher les produits
    DataGridView dgvProduits = new DataGridView
    {
        Location = new Point(20, 100),
        Width = 700,
        Height = 300,
        AllowUserToAddRows = false,
        AllowUserToDeleteRows = false,
        ReadOnly = true,
        SelectionMode = DataGridViewSelectionMode.FullRowSelect,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
    };

    // Lier le DataGridView à la source de binding
    dgvProduits.DataSource = bs;

    // Personnaliser l'affichage
    dgvProduits.Columns["Prix"].DefaultCellStyle.Format = "C";

    // Contrôles de filtrage
    Label lblFiltreCategorie = new Label
    {
        Text = "Filtrer par catégorie :",
        Location = new Point(20, 20),
        AutoSize = true
    };

    ComboBox cboFiltreCategorie = new ComboBox
    {
        Location = new Point(160, 20),
        Width = 150,
        DropDownStyle = ComboBoxStyle.DropDownList
    };

    // Remplir le ComboBox avec les catégories uniques
    cboFiltreCategorie.Items.Add("(Toutes)");
    foreach (var catégorie in produits.Select(p => p.Catégorie).Distinct().OrderBy(c => c))
    {
        cboFiltreCategorie.Items.Add(catégorie);
    }
    cboFiltreCategorie.SelectedIndex = 0;

    Label lblFiltrePrix = new Label
    {
        Text = "Prix maximum :",
        Location = new Point(320, 20),
        AutoSize = true
    };

    NumericUpDown nudFiltrePrix = new NumericUpDown
    {
        Location = new Point(420, 20),
        Width = 100,
        Minimum = 0,
        Maximum = 1000,
        Value = 1000,
        Increment = 10
    };

    Label lblFiltreNom = new Label
    {
        Text = "Rechercher :",
        Location = new Point(20, 60),
        AutoSize = true
    };

    TextBox txtFiltreNom = new TextBox
    {
        Location = new Point(120, 60),
        Width = 200
    };

    // Contrôles de tri
    Label lblTri = new Label
    {
        Text = "Trier par :",
        Location = new Point(340, 60),
        AutoSize = true
    };

    ComboBox cboTri = new ComboBox
    {
        Location = new Point(420, 60),
        Width = 150,
        DropDownStyle = ComboBoxStyle.DropDownList
    };

    // Remplir le ComboBox de tri
    cboTri.Items.AddRange(new object[] { "Nom (A-Z)", "Nom (Z-A)", "Prix (croissant)", "Prix (décroissant)", "Stock (croissant)", "Stock (décroissant)" });
    cboTri.SelectedIndex = 0;

    // Fonction pour appliquer le filtre
    Action appliquerFiltre = () => {
        bs.Filter = "";

        List<string> conditions = new List<string>();

        // Filtre par catégorie
        if (cboFiltreCategorie.SelectedIndex > 0)
        {
            string catégorie = cboFiltreCategorie.SelectedItem.ToString();
            conditions.Add($"Catégorie = '{catégorie}'");
        }

        // Filtre par prix
        decimal prixMax = nudFiltrePrix.Value;
        conditions.Add($"Prix <= {prixMax.ToString(CultureInfo.InvariantCulture)}");

        // Filtre par nom (recherche)
        if (!string.IsNullOrWhiteSpace(txtFiltreNom.Text))
        {
            string recherche = txtFiltreNom.Text.Replace("'", "''");  // Échapper les apostrophes
            conditions.Add($"Nom LIKE '%{recherche}%'");
        }

        // Appliquer tous les filtres
        if (conditions.Count > 0)
        {
            bs.Filter = string.Join(" AND ", conditions);
        }

        // Aucun résultat
        if (bs.Count == 0)
        {
            MessageBox.Show("Aucun produit ne correspond aux critères de filtrage.",
                           "Résultats de recherche", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    };

    // Fonction pour appliquer le tri
    Action appliquerTri = () => {
        switch (cboTri.SelectedIndex)
        {
            case 0:  // Nom (A-Z)
                bs.Sort = "Nom ASC";
                break;
            case 1:  // Nom (Z-A)
                bs.Sort = "Nom DESC";
                break;
            case 2:  // Prix (croissant)
                bs.Sort = "Prix ASC";
                break;
            case 3:  // Prix (décroissant)
                bs.Sort = "Prix DESC";
                break;
            case 4:  // Stock (croissant)
                bs.Sort = "Stock ASC";
                break;
            case 5:  // Stock (décroissant)
                bs.Sort = "Stock DESC";
                break;
        }
    };

    // Événements pour appliquer les filtres
    cboFiltreCategorie.SelectedIndexChanged += (s, e) => appliquerFiltre();
    nudFiltrePrix.ValueChanged += (s, e) => appliquerFiltre();

    // Appliquer la recherche quand on appuie sur Entrée dans la zone de texte
    txtFiltreNom.KeyDown += (s, e) => {
        if (e.KeyCode == Keys.Enter)
        {
            appliquerFiltre();
            e.Handled = true;
            e.SuppressKeyPress = true;  // Éviter le son de bip
        }
    };

    // Événement pour appliquer le tri
    cboTri.SelectedIndexChanged += (s, e) => appliquerTri();

    // Bouton pour appliquer les filtres
    Button btnAppliquer = new Button
    {
        Text = "Rechercher",
        Location = new Point(580, 60),
        Width = 100
    };

    btnAppliquer.Click += (s, e) => {
        appliquerFiltre();
        appliquerTri();
    };

    // Bouton pour réinitialiser les filtres
    Button btnRéinitialiser = new Button
    {
        Text = "Réinitialiser",
        Location = new Point(580, 20),
        Width = 100
    };

    btnRéinitialiser.Click += (s, e) => {
        cboFiltreCategorie.SelectedIndex = 0;
        nudFiltrePrix.Value = 1000;
        txtFiltreNom.Text = "";
        cboTri.SelectedIndex = 0;
        bs.RemoveFilter();
        bs.Sort = "Nom ASC";
    };

    // Ajouter les contrôles au formulaire
    this.Controls.Add(lblFiltreCategorie);
    this.Controls.Add(cboFiltreCategorie);
    this.Controls.Add(lblFiltrePrix);
    this.Controls.Add(nudFiltrePrix);
    this.Controls.Add(lblFiltreNom);
    this.Controls.Add(txtFiltreNom);
    this.Controls.Add(lblTri);
    this.Controls.Add(cboTri);
    this.Controls.Add(btnAppliquer);
    this.Controls.Add(btnRéinitialiser);
    this.Controls.Add(dgvProduits);

    // Appliquer le tri initial
    appliquerTri();
}
```

## 7.8.3. BindingSource et BindingList

`BindingSource` et `BindingList<T>` sont des classes essentielles pour la liaison de données avancée, offrant des fonctionnalités comme la notification de changements, le tri et le filtrage.

### BindingSource

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExempleBindingSource()
{
    // Créer une liste de produits
    List<Produit> produits = new List<Produit>
    {
        new Produit { Id = 1, Nom = "Ordinateur portable", Catégorie = "Informatique", Prix = 899.99m, Stock = 15, Disponible = true },
        new Produit { Id = 2, Nom = "Souris sans fil", Catégorie = "Accessoires", Prix = 24.99m, Stock = 50, Disponible = true },
        new Produit { Id = 3, Nom = "Écran 24\"", Catégorie = "Informatique", Prix = 199.50m, Stock = 8, Disponible = true }
    };

    // Créer une source de binding
    BindingSource bs = new BindingSource();
    bs.DataSource = produits;

    // Créer un DataGridView pour afficher les produits
    DataGridView dgvProduits = new DataGridView
    {
        Location = new Point(20, 50),
        Width = 600,
        Height = 200,
        AllowUserToAddRows = false,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
    };

    // Lier le DataGridView à la source de binding
    dgvProduits.DataSource = bs;

    // Personnaliser l'affichage
    dgvProduits.Columns["Prix"].DefaultCellStyle.Format = "C";

    // Contrôles de navigation
    Button btnPremier = new Button
    {
        Text = "Premier",
        Location = new Point(20, 260),
        Width = 100
    };

    Button btnPrécédent = new Button
    {
        Text = "Précédent",
        Location = new Point(130, 260),
        Width = 100
    };

    Label lblPosition = new Label
    {
        Text = "0 / 0",
        Location = new Point(240, 265),
        AutoSize = true,
        TextAlign = ContentAlignment.MiddleCenter
    };

    Button btnSuivant = new Button
    {
        Text = "Suivant",
        Location = new Point(300, 260),
        Width = 100
    };

    Button btnDernier = new Button
    {
        Text = "Dernier",
        Location = new Point(410, 260),
        Width = 100
    };

    // Événements de navigation
    btnPremier.Click += (s, e) => bs.MoveFirst();
    btnPrécédent.Click += (s, e) => bs.MovePrevious();
    btnSuivant.Click += (s, e) => bs.MoveNext();
    btnDernier.Click += (s, e) => bs.MoveLast();

    // Mise à jour de l'affichage de la position
    Action mettreÀJourPosition = () => {
        int position = bs.Position + 1;  // +1 car Position est base 0
        int count = bs.Count;
        lblPosition.Text = $"{position} / {count}";
    };

    bs.PositionChanged += (s, e) => mettreÀJourPosition();
    bs.ListChanged += (s, e) => mettreÀJourPosition();

    // Appeler initialement pour définir le texte
    mettreÀJourPosition();

    // Contrôles pour ajouter/supprimer/modifier
    Label lblNom = new Label { Text = "Nom :", Location = new Point(20, 300), AutoSize = true };
    TextBox txtNom = new TextBox { Location = new Point(120, 300), Width = 200 };

    Label lblPrix = new Label { Text = "Prix :", Location = new Point(20, 330), AutoSize = true };
    NumericUpDown nudPrix = new NumericUpDown
    {
        Location = new Point(120, 330),
        Width = 100,
        Minimum = 0,
        Maximum = 10000,
        DecimalPlaces = 2,
        Increment = 0.1m
    };

    Label lblCatégorie = new Label { Text = "Catégorie :", Location = new Point(20, 360), AutoSize = true };
    ComboBox cboCatégorie = new ComboBox
    {
        Location = new Point(120, 360),
        Width = 150,
        DropDownStyle = ComboBoxStyle.DropDownList
    };

    cboCatégorie.Items.AddRange(new object[] { "Informatique", "Accessoires", "Logiciels", "Stockage" });
    cboCatégorie.SelectedIndex = 0;

    Button btnAjouter = new Button
    {
        Text = "Ajouter",
        Location = new Point(350, 300),
        Width = 100
    };

    Button btnSupprimer = new Button
    {
        Text = "Supprimer",
        Location = new Point(350, 330),
        Width = 100
    };

    Button btnModifier = new Button
    {
        Text = "Modifier",
        Location = new Point(350, 360),
        Width = 100
    };

    // Événement d'ajout
    btnAjouter.Click += (s, e) => {
        // Vérifier les entrées
        if (string.IsNullOrWhiteSpace(txtNom.Text))
        {
            MessageBox.Show("Veuillez entrer un nom de produit.", "Erreur",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
            return;
        }

        // Créer un nouveau produit
        Produit nouveauProduit = new Produit
        {
            Id = produits.Count > 0 ? produits.Max(p => p.Id) + 1 : 1,
            Nom = txtNom.Text,
            Prix = nudPrix.Value,
            Catégorie = cboCatégorie.SelectedItem.ToString(),
            Stock = 0,
            Disponible = true
        };

        // Ajouter à la liste
        produits.Add(nouveauProduit);

        // Rafraîchir la source de binding
        bs.ResetBindings(false);

        // Sélectionner le nouveau produit
        bs.Position = bs.Count - 1;

        // Effacer les champs
        txtNom.Text = "";
        nudPrix.Value = 0;

        MessageBox.Show("Produit ajouté avec succès.", "Succès",
                       MessageBoxButtons.OK, MessageBoxIcon.Information);
    };

    // Événement de suppression
    btnSupprimer.Click += (s, e) => {
        if (bs.Current != null)
        {
            DialogResult result = MessageBox.Show(
                "Êtes-vous sûr de vouloir supprimer ce produit ?",
                "Confirmation de suppression",
                MessageBoxButtons.YesNo,
                MessageBoxIcon.Question);

            if (result == DialogResult.Yes)
            {
                // Supprimer le produit
                produits.Remove((Produit)bs.Current);

                // Rafraîchir la source de binding
                bs.ResetBindings(false);

                MessageBox.Show("Produit supprimé.", "Succès",
                               MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }
        else
        {
            MessageBox.Show("Aucun produit sélectionné.", "Erreur",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    };

    // Événement de modification
    btnModifier.Click += (s, e) => {
        if (bs.Current != null)
        {
            // Vérifier les entrées
            if (string.IsNullOrWhiteSpace(txtNom.Text))
            {
                MessageBox.Show("Veuillez entrer un nom de produit.", "Erreur",
                               MessageBoxButtons.OK, MessageBoxIcon.Error);
                return;
            }

            // Modifier le produit
            Produit produit = (Produit)bs.Current;
            produit.Nom = txtNom.Text;
            produit.Prix = nudPrix.Value;
            produit.Catégorie = cboCatégorie.SelectedItem.ToString();

            // Rafraîchir la source de binding
            bs.ResetBindings(false);

            MessageBox.Show("Produit modifié avec succès.", "Succès",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show("Aucun produit sélectionné.", "Erreur",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    };

    // Sélection d'un produit dans le DataGridView
    dgvProduits.SelectionChanged += (s, e) => {
        if (bs.Current != null)
        {
            Produit produit = (Produit)bs.Current;
            txtNom.Text = produit.Nom;
            nudPrix.Value = produit.Prix;
            cboCatégorie.SelectedItem = produit.Catégorie;
        }
    };

    // Exemple d'utilisation des méthodes Find et Filter de BindingSource
    Label lblRecherche = new Label { Text = "Rechercher :", Location = new Point(20, 20), AutoSize = true };
    TextBox txtRecherche = new TextBox { Location = new Point(120, 20), Width = 200 };
    Button btnRechercher = new Button { Text = "Rechercher", Location = new Point(330, 20), Width = 100 };

    btnRechercher.Click += (s, e) => {
        string recherche = txtRecherche.Text.Trim();

        if (string.IsNullOrEmpty(recherche))
        {
            // Réinitialiser le filtre
            bs.RemoveFilter();
            return;
        }

        // Appliquer un filtre sur le nom du produit
        bs.Filter = $"Nom LIKE '%{recherche.Replace("'", "''")}%'";

        // Si aucun résultat, informer l'utilisateur
        if (bs.Count == 0)
        {
            MessageBox.Show("Aucun produit ne correspond à votre recherche.",
                           "Résultats de recherche", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    };

    // Ajouter les contrôles au formulaire
    this.Controls.Add(lblRecherche);
    this.Controls.Add(txtRecherche);
    this.Controls.Add(btnRechercher);
    this.Controls.Add(dgvProduits);
    this.Controls.Add(btnPremier);
    this.Controls.Add(btnPrécédent);
    this.Controls.Add(lblPosition);
    this.Controls.Add(btnSuivant);
    this.Controls.Add(btnDernier);
    this.Controls.Add(lblNom);
    this.Controls.Add(txtNom);
    this.Controls.Add(lblPrix);
    this.Controls.Add(nudPrix);
    this.Controls.Add(lblCatégorie);
    this.Controls.Add(cboCatégorie);
    this.Controls.Add(btnAjouter);
    this.Controls.Add(btnSupprimer);
    this.Controls.Add(btnModifier);
}
```


### BindingList

```textmate
// .NET Framework 4.7.2 et .NET 8
// Classe Produit avec INotifyPropertyChanged pour notification de changements
public class ProduitAvecNotification : INotifyPropertyChanged
{
    private int _id;
    private string _nom;
    private string _catégorie;
    private decimal _prix;
    private int _stock;
    private bool _disponible;

    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    public int Id
    {
        get => _id;
        set
        {
            if (_id != value)
            {
                _id = value;
                OnPropertyChanged(nameof(Id));
            }
        }
    }

    public string Nom
    {
        get => _nom;
        set
        {
            if (_nom != value)
            {
                _nom = value;
                OnPropertyChanged(nameof(Nom));
            }
        }
    }

    public string Catégorie
    {
        get => _catégorie;
        set
        {
            if (_catégorie != value)
            {
                _catégorie = value;
                OnPropertyChanged(nameof(Catégorie));
            }
        }
    }

    public decimal Prix
    {
        get => _prix;
        set
        {
            if (_prix != value)
            {
                _prix = value;
                OnPropertyChanged(nameof(Prix));
            }
        }
    }

    public int Stock
    {
        get => _stock;
        set
        {
            if (_stock != value)
            {
                _stock = value;
                OnPropertyChanged(nameof(Stock));
                OnPropertyChanged(nameof(Disponible));  // Stock affecte la disponibilité
            }
        }
    }

    public bool Disponible
    {
        get => _disponible;
        set
        {
            if (_disponible != value)
            {
                _disponible = value;
                OnPropertyChanged(nameof(Disponible));
            }
        }
    }
}

private void ExempleBindingList()
{
    // Créer une BindingList de produits
    BindingList<ProduitAvecNotification> produits = new BindingList<ProduitAvecNotification>
    {
        new ProduitAvecNotification { Id = 1, Nom = "Ordinateur portable", Catégorie = "Informatique", Prix = 899.99m, Stock = 15, Disponible = true },
        new ProduitAvecNotification { Id = 2, Nom = "Souris sans fil", Catégorie = "Accessoires", Prix = 24.99m, Stock = 50, Disponible = true },
        new ProduitAvecNotification { Id = 3, Nom = "Écran 24\"", Catégorie = "Informatique", Prix = 199.50m, Stock = 8, Disponible = true }
    };

    // Créer une source de binding
    BindingSource bs = new BindingSource();
    bs.DataSource = produits;

    // Créer un DataGridView pour afficher les produits
    DataGridView dgvProduits = new DataGridView
    {
        Location = new Point(20, 50),
        Width = 600,
        Height = 200,
        AllowUserToAddRows = false,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
    };

    // Lier le DataGridView à la source de binding
    dgvProduits.DataSource = bs;

    // Personnaliser l'affichage
    dgvProduits.Columns["Prix"].DefaultCellStyle.Format = "C";

    // Contrôles pour la modification directe
    GroupBox grpModification = new GroupBox
    {
        Text = "Modification directe",
        Location = new Point(20, 260),
        Size = new Size(600, 150)
    };

    Label lblIdProduit = new Label { Text = "ID du produit :", Location = new Point(20, 30), AutoSize = true };
    ComboBox cboIdProduit = new ComboBox
    {
        Location = new Point(120, 30),
        Width = 100,
        DropDownStyle = ComboBoxStyle.DropDownList
    };

    foreach (var produit in produits)
    {
        cboIdProduit.Items.Add(produit.Id);
    }

    if (cboIdProduit.Items.Count > 0)
    {
        cboIdProduit.SelectedIndex = 0;
    }

    Label lblStock = new Label { Text = "Stock :", Location = new Point(20, 70), AutoSize = true };
    NumericUpDown nudStock = new NumericUpDown
    {
        Location = new Point(120, 70),
        Width = 100,
        Minimum = 0,
        Maximum = 1000
    };

    CheckBox chkDisponible = new CheckBox
    {
        Text = "Disponible",
        Location = new Point(250, 70),
        AutoSize = true
    };

    // Test d'observation des changements
    Label lblPrix = new Label { Text = "Prix :", Location = new Point(20, 100), AutoSize = true };
    NumericUpDown nudPrix = new NumericUpDown
    {
        Location = new Point(120, 100),
        Width = 100,
        Minimum = 0,
        Maximum = 10000,
        DecimalPlaces = 2,
        Increment = 0.1m
    };

    Button btnAppliquer = new Button
    {
        Text = "Appliquer",
        Location = new Point(250, 100),
        Width = 100
    };

    // Sélection d'un produit
    cboIdProduit.SelectedIndexChanged += (s, e) => {
        if (cboIdProduit.SelectedItem != null)
        {
            int idProduit = (int)cboIdProduit.SelectedItem;
            ProduitAvecNotification produit = produits.FirstOrDefault(p => p.Id == idProduit);

            if (produit != null)
            {
                nudStock.Value = produit.Stock;
                chkDisponible.Checked = produit.Disponible;
                nudPrix.Value = produit.Prix;
            }
        }
    };

    // Application des modifications
    btnAppliquer.Click += (s, e) => {
        if (cboIdProduit.SelectedItem != null)
        {
            int idProduit = (int)cboIdProduit.SelectedItem;
            ProduitAvecNotification produit = produits.FirstOrDefault(p => p.Id == idProduit);

            if (produit != null)
            {
                // Modifier les propriétés - cela déclenchera des notifications
                produit.Stock = (int)nudStock.Value;
                produit.Disponible = chkDisponible.Checked;
                produit.Prix = nudPrix.Value;

                // Le DataGridView est automatiquement mis à jour grâce aux notifications
                MessageBox.Show("Produit modifié avec succès.", "Succès",
                               MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }
    };

    // Moniteur pour observer les changements
    ListBox lstChangements = new ListBox
    {
        Location = new Point(20, 420),
        Width = 600,
        Height = 150
    };

    // Observer les changements de propriétés
    foreach (var produit in produits)
    {
        produit.PropertyChanged += (s, e) => {
            string message = $"[{DateTime.Now.ToLongTimeString()}] Produit ID {((ProduitAvecNotification)s).Id}: Propriété '{e.PropertyName}' modifiée";
            lstChangements.Items.Add(message);
            lstChangements.SelectedIndex = lstChangements.Items.Count - 1;  // Faire défiler vers le bas
        };
    }

    // Observer les changements de liste
    produits.ListChanged += (s, e) => {
        string message = $"[{DateTime.Now.ToLongTimeString()}] Liste modifiée: ";

        switch (e.ListChangedType)
        {
            case ListChangedType.ItemAdded:
                message += $"Élément ajouté à l'index {e.NewIndex}";
                break;
            case ListChangedType.ItemDeleted:
                message += $"Élément supprimé à l'index {e.NewIndex}";
                break;
            case ListChangedType.ItemChanged:
                message += $"Élément à l'index {e.NewIndex} modifié, propriété : {e.PropertyDescriptor?.Name}";
                break;
            case ListChangedType.Reset:
                message += "Liste réinitialisée";
                break;
            default:
                message += $"Type de changement: {e.ListChangedType}";
                break;
        }

        lstChangements.Items.Add(message);
        lstChangements.SelectedIndex = lstChangements.Items.Count - 1;
    };

    // Bouton pour ajouter un nouveau produit
    Button btnNouveauProduit = new Button
    {
        Text = "Ajouter un produit",
        Location = new Point(20, 380),
        Width = 150
    };

    btnNouveauProduit.Click += (s, e) => {
        // Créer un nouveau produit
        ProduitAvecNotification nouveauProduit = new ProduitAvecNotification
        {
            Id = produits.Count > 0 ? produits.Max(p => p.Id) + 1 : 1,
            Nom = $"Nouveau Produit {DateTime.Now.ToString("HHmmss")}",
            Catégorie = "Nouvelle catégorie",
            Prix = 99.99m,
            Stock = 10,
            Disponible = true
        };

        // Observer ses changements de propriétés
        nouveauProduit.PropertyChanged += (sender, evt) => {
            string message = $"[{DateTime.Now.ToLongTimeString()}] Produit ID {((ProduitAvecNotification)sender).Id}: Propriété '{evt.PropertyName}' modifiée";
            lstChangements.Items.Add(message);
            lstChangements.SelectedIndex = lstChangements.Items.Count - 1;
        };

        // Ajouter à la liste
        produits.Add(nouveauProduit);

        // Mettre à jour le ComboBox
        cboIdProduit.Items.Add(nouveauProduit.Id);

        MessageBox.Show($"Nouveau produit ajouté avec l'ID {nouveauProduit.Id}.", "Succès",
                       MessageBoxButtons.OK, MessageBoxIcon.Information);
    };

    // Bouton pour supprimer un produit
    Button btnSupprimerProduit = new Button
    {
        Text = "Supprimer le produit",
        Location = new Point(180, 380),
        Width = 150
    };

    btnSupprimerProduit.Click += (s, e) => {
        if (cboIdProduit.SelectedItem != null)
        {
            int idProduit = (int)cboIdProduit.SelectedItem;
            ProduitAvecNotification produit = produits.FirstOrDefault(p => p.Id == idProduit);

            if (produit != null)
            {
                DialogResult result = MessageBox.Show(
                    $"Êtes-vous sûr de vouloir supprimer le produit '{produit.Nom}' ?",
                    "Confirmation de suppression",
                    MessageBoxButtons.YesNo,
                    MessageBoxIcon.Question);

                if (result == DialogResult.Yes)
                {
                    // Supprimer de la liste
                    produits.Remove(produit);

                    // Mettre à jour le ComboBox
                    cboIdProduit.Items.Remove(idProduit);

                    if (cboIdProduit.Items.Count > 0)
                    {
                        cboIdProduit.SelectedIndex = 0;
                    }

                    MessageBox.Show("Produit supprimé avec succès.", "Succès",
                                   MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
            }
        }
    };

    // Ajouter les contrôles au GroupBox
    grpModification.Controls.Add(lblIdProduit);
    grpModification.Controls.Add(cboIdProduit);
    grpModification.Controls.Add(lblStock);
    grpModification.Controls.Add(nudStock);
    grpModification.Controls.Add(chkDisponible);
    grpModification.Controls.Add(lblPrix);
    grpModification.Controls.Add(nudPrix);
    grpModification.Controls.Add(btnAppliquer);

    // Ajouter les contrôles au formulaire
    this.Controls.Add(new Label { Text = "Liste des produits :", Location = new Point(20, 20), AutoSize = true });
    this.Controls.Add(dgvProduits);
    this.Controls.Add(grpModification);
    this.Controls.Add(btnNouveauProduit);
    this.Controls.Add(btnSupprimerProduit);
    this.Controls.Add(new Label { Text = "Journal des changements :", Location = new Point(20, 400), AutoSize = true });
    this.Controls.Add(lstChangements);
}
```

# 7.8.4. Validation de données

La validation des données est une étape essentielle dans le développement d'applications Windows Forms. Elle permet de garantir que les données entrées par l'utilisateur respectent certaines règles avant d'être acceptées par l'application. Windows Forms offre plusieurs mécanismes pour implémenter cette validation.

## Introduction à la validation de données

La validation des données dans Windows Forms peut être effectuée à différents niveaux :

- **Validation simple** : Vérifier la valeur d'un contrôle spécifique
- **Validation liée aux données** : Vérifier les données pendant le processus de binding
- **Validation au niveau de l'objet** : Implémenter des interfaces comme `IDataErrorInfo` ou `INotifyDataErrorInfo`

## Validation simple avec l'événement Validating

La méthode la plus directe pour valider les données est d'utiliser l'événement `Validating` des contrôles Windows Forms.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerValidationSimple()
{
    // Créer un ErrorProvider pour afficher les erreurs
    ErrorProvider errorProvider = new ErrorProvider();

    // Créer un TextBox pour le nom
    TextBox txtNom = new TextBox
    {
        Location = new Point(120, 20),
        Width = 200
    };

    // Ajouter un label
    Label lblNom = new Label
    {
        Text = "Nom :",
        Location = new Point(20, 23),
        AutoSize = true
    };

    // Configurer la validation
    txtNom.Validating += (sender, e) =>
    {
        if (string.IsNullOrWhiteSpace(txtNom.Text))
        {
            // Annuler la validation
            e.Cancel = true;

            // Afficher l'erreur
            errorProvider.SetError(txtNom, "Le nom ne peut pas être vide.");
        }
        else if (txtNom.Text.Length < 3)
        {
            e.Cancel = true;
            errorProvider.SetError(txtNom, "Le nom doit contenir au moins 3 caractères.");
        }
        else
        {
            // Effacer l'erreur si tout est correct
            errorProvider.SetError(txtNom, "");
        }
    };

    // Gérer l'événement Validated (déclenché si la validation réussit)
    txtNom.Validated += (sender, e) =>
    {
        // Effacer l'erreur
        errorProvider.SetError(txtNom, "");
    };

    // Ajouter les contrôles au formulaire
    this.Controls.Add(lblNom);
    this.Controls.Add(txtNom);

    // Configurer le comportement de validation du formulaire
    this.AutoValidate = AutoValidate.EnablePreventFocusChange;
}
```


## Validation avec Binding.Parse

Lorsque vous utilisez le data binding, vous pouvez valider les données pendant le processus de binding en utilisant l'événement `Parse` de l'objet `Binding`.

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ConfigurerValidationAvecBinding()
{
    // Créer un objet à lier
    Personne personne = new Personne
    {
        Nom = "Dupont",
        Email = "jean.dupont@example.com",
        Age = 30
    };

    // Créer des contrôles
    TextBox txtNom = new TextBox { Location = new Point(120, 20), Width = 200 };
    TextBox txtEmail = new TextBox { Location = new Point(120, 50), Width = 200 };
    NumericUpDown nudAge = new NumericUpDown { Location = new Point(120, 80), Width = 100, Minimum = 0, Maximum = 120 };

    // Ajouter des labels
    this.Controls.Add(new Label { Text = "Nom :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "Email :", Location = new Point(20, 53), AutoSize = true });
    this.Controls.Add(new Label { Text = "Âge :", Location = new Point(20, 83), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(txtNom);
    this.Controls.Add(txtEmail);
    this.Controls.Add(nudAge);

    // Créer un ErrorProvider
    ErrorProvider errorProvider = new ErrorProvider();

    // Liaison avec validation pour le nom
    Binding bindingNom = new Binding("Text", personne, "Nom", true);
    bindingNom.Parse += (sender, e) =>
    {
        if (e.Value is string nom)
        {
            if (string.IsNullOrWhiteSpace(nom))
            {
                throw new ArgumentException("Le nom ne peut pas être vide.");
            }

            if (nom.Length < 2)
            {
                throw new ArgumentException("Le nom doit contenir au moins 2 caractères.");
            }
        }
    };

    // Gérer les erreurs de validation
    bindingNom.BindingComplete += (sender, e) =>
    {
        if (e.Exception != null && !e.Handled)
        {
            errorProvider.SetError(txtNom, e.Exception.Message);
            e.Handled = true;
        }
        else
        {
            errorProvider.SetError(txtNom, "");
        }
    };

    txtNom.DataBindings.Add(bindingNom);

    // Liaison avec validation pour l'email
    Binding bindingEmail = new Binding("Text", personne, "Email", true);
    bindingEmail.Parse += (sender, e) =>
    {
        if (e.Value is string email)
        {
            if (string.IsNullOrWhiteSpace(email))
            {
                throw new ArgumentException("L'email ne peut pas être vide.");
            }

            if (!email.Contains("@") || !email.Contains("."))
            {
                throw new ArgumentException("Format d'email invalide.");
            }
        }
    };

    bindingEmail.BindingComplete += (sender, e) =>
    {
        if (e.Exception != null && !e.Handled)
        {
            errorProvider.SetError(txtEmail, e.Exception.Message);
            e.Handled = true;
        }
        else
        {
            errorProvider.SetError(txtEmail, "");
        }
    };

    txtEmail.DataBindings.Add(bindingEmail);

    // Liaison simple pour l'âge
    nudAge.DataBindings.Add("Value", personne, "Age");

    // Bouton pour afficher les données
    Button btnAfficher = new Button
    {
        Text = "Afficher les données",
        Location = new Point(120, 120),
        Width = 150
    };

    btnAfficher.Click += (sender, e) =>
    {
        try
        {
            // Forcer la validation
            this.ValidateChildren();

            MessageBox.Show(
                $"Nom: {personne.Nom}\n" +
                $"Email: {personne.Email}\n" +
                $"Âge: {personne.Age}",
                "Données actuelles",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);
        }
        catch (Exception ex)
        {
            MessageBox.Show(
                $"Erreur de validation : {ex.Message}",
                "Erreur",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
    };

    this.Controls.Add(btnAfficher);
}

// Classe Personne pour l'exemple
public class Personne
{
    public string Nom { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
}
```


## Validation avec IDataErrorInfo

L'interface `IDataErrorInfo` permet d'intégrer la validation directement dans vos objets métier, ce qui centralise la logique de validation.

```textmate
// .NET Framework 4.7.2 et .NET 8
// Classe qui implémente IDataErrorInfo
public class UtilisateurValidable : IDataErrorInfo
{
    private Dictionary<string, string> _erreurs = new Dictionary<string, string>();

    public string Nom { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
    public string Password { get; set; }

    // Propriété indexeur requise par IDataErrorInfo
    public string this[string columnName]
    {
        get
        {
            string erreur = null;

            switch (columnName)
            {
                case nameof(Nom):
                    if (string.IsNullOrWhiteSpace(Nom))
                    {
                        erreur = "Le nom est obligatoire.";
                    }
                    else if (Nom.Length < 2)
                    {
                        erreur = "Le nom doit contenir au moins 2 caractères.";
                    }
                    break;

                case nameof(Email):
                    if (string.IsNullOrWhiteSpace(Email))
                    {
                        erreur = "L'email est obligatoire.";
                    }
                    else if (!Email.Contains("@") || !Email.Contains("."))
                    {
                        erreur = "Format d'email invalide.";
                    }
                    break;

                case nameof(Age):
                    if (Age < 18)
                    {
                        erreur = "Vous devez avoir au moins 18 ans.";
                    }
                    else if (Age > 120)
                    {
                        erreur = "L'âge semble invalide.";
                    }
                    break;

                case nameof(Password):
                    if (string.IsNullOrWhiteSpace(Password))
                    {
                        erreur = "Le mot de passe est obligatoire.";
                    }
                    else if (Password.Length < 6)
                    {
                        erreur = "Le mot de passe doit contenir au moins 6 caractères.";
                    }
                    break;
            }

            // Mettre à jour le dictionnaire d'erreurs
            if (string.IsNullOrEmpty(erreur))
            {
                _erreurs.Remove(columnName);
            }
            else
            {
                _erreurs[columnName] = erreur;
            }

            return erreur;
        }
    }

    // Propriété Error requise par IDataErrorInfo
    public string Error
    {
        get { return string.Join(Environment.NewLine, _erreurs.Values); }
    }

    // Méthode pour vérifier si l'objet est valide
    public bool EstValide()
    {
        // Vérifier chaque propriété
        this[nameof(Nom)];
        this[nameof(Email)];
        this[nameof(Age)];
        this[nameof(Password)];

        // L'objet est valide s'il n'y a pas d'erreurs
        return _erreurs.Count == 0;
    }
}

private void DemoIDataErrorInfo()
{
    // Créer un utilisateur
    UtilisateurValidable utilisateur = new UtilisateurValidable
    {
        Nom = "",
        Email = "",
        Age = 0,
        Password = ""
    };

    // Créer un ErrorProvider
    ErrorProvider errorProvider = new ErrorProvider();

    // Créer les contrôles
    TextBox txtNom = new TextBox { Location = new Point(150, 20), Width = 200 };
    TextBox txtEmail = new TextBox { Location = new Point(150, 50), Width = 200 };
    NumericUpDown nudAge = new NumericUpDown { Location = new Point(150, 80), Width = 100, Minimum = 0, Maximum = 120 };
    TextBox txtPassword = new TextBox { Location = new Point(150, 110), Width = 200, PasswordChar = '*' };

    // Ajouter des labels
    this.Controls.Add(new Label { Text = "Nom :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "Email :", Location = new Point(20, 53), AutoSize = true });
    this.Controls.Add(new Label { Text = "Âge :", Location = new Point(20, 83), AutoSize = true });
    this.Controls.Add(new Label { Text = "Mot de passe :", Location = new Point(20, 113), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(txtNom);
    this.Controls.Add(txtEmail);
    this.Controls.Add(nudAge);
    this.Controls.Add(txtPassword);

    // Binding avec l'objet utilisateur
    txtNom.DataBindings.Add("Text", utilisateur, "Nom", true, DataSourceUpdateMode.OnPropertyChanged);
    txtEmail.DataBindings.Add("Text", utilisateur, "Email", true, DataSourceUpdateMode.OnPropertyChanged);
    nudAge.DataBindings.Add("Value", utilisateur, "Age", true, DataSourceUpdateMode.OnPropertyChanged);
    txtPassword.DataBindings.Add("Text", utilisateur, "Password", true, DataSourceUpdateMode.OnPropertyChanged);

    // Fonction pour valider un contrôle
    Action<Control, string> validerControle = (control, propertyName) =>
    {
        string erreur = utilisateur[propertyName];
        errorProvider.SetError(control, erreur);
    };

    // Valider les contrôles lors des événements Validated
    txtNom.Validated += (sender, e) => validerControle(txtNom, nameof(UtilisateurValidable.Nom));
    txtEmail.Validated += (sender, e) => validerControle(txtEmail, nameof(UtilisateurValidable.Email));
    nudAge.Validated += (sender, e) => validerControle(nudAge, nameof(UtilisateurValidable.Age));
    txtPassword.Validated += (sender, e) => validerControle(txtPassword, nameof(UtilisateurValidable.Password));

    // Valider initialement tous les contrôles
    validerControle(txtNom, nameof(UtilisateurValidable.Nom));
    validerControle(txtEmail, nameof(UtilisateurValidable.Email));
    validerControle(nudAge, nameof(UtilisateurValidable.Age));
    validerControle(txtPassword, nameof(UtilisateurValidable.Password));

    // Bouton de validation
    Button btnValider = new Button
    {
        Text = "Valider",
        Location = new Point(150, 150),
        Width = 100
    };

    btnValider.Click += (sender, e) =>
    {
        if (utilisateur.EstValide())
        {
            MessageBox.Show(
                "Formulaire validé avec succès !\n\n" +
                $"Nom: {utilisateur.Nom}\n" +
                $"Email: {utilisateur.Email}\n" +
                $"Âge: {utilisateur.Age}",
                "Validation réussie",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show(
                "Le formulaire contient des erreurs :\n\n" + utilisateur.Error,
                "Erreurs de validation",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
    };

    this.Controls.Add(btnValider);
}
```


## Validation avec INotifyDataErrorInfo (.NET Framework 4.5+ et .NET Core/.NET)

L'interface `INotifyDataErrorInfo` est une évolution d'`IDataErrorInfo` et offre une validation asynchrone et la gestion de plusieurs erreurs par propriété.

```textmate
// .NET Framework 4.7.2 et .NET 8
public class UtilisateurModerne : INotifyPropertyChanged, INotifyDataErrorInfo
{
    private Dictionary<string, List<string>> _erreurs = new Dictionary<string, List<string>>();
    private string _nom;
    private string _email;
    private int _age;

    public event PropertyChangedEventHandler PropertyChanged;
    public event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;

    public string Nom
    {
        get => _nom;
        set
        {
            if (_nom != value)
            {
                _nom = value;
                ValiderNom();
                OnPropertyChanged(nameof(Nom));
            }
        }
    }

    public string Email
    {
        get => _email;
        set
        {
            if (_email != value)
            {
                _email = value;
                ValiderEmail();
                OnPropertyChanged(nameof(Email));
            }
        }
    }

    public int Age
    {
        get => _age;
        set
        {
            if (_age != value)
            {
                _age = value;
                ValiderAge();
                OnPropertyChanged(nameof(Age));
            }
        }
    }

    public bool HasErrors => _erreurs.Count > 0;

    public IEnumerable GetErrors(string propertyName)
    {
        if (string.IsNullOrEmpty(propertyName) || !_erreurs.ContainsKey(propertyName))
            return null;

        return _erreurs[propertyName];
    }

    private void ValiderNom()
    {
        List<string> erreurs = new List<string>();

        if (string.IsNullOrWhiteSpace(Nom))
            erreurs.Add("Le nom est obligatoire.");
        else if (Nom.Length < 2)
            erreurs.Add("Le nom doit contenir au moins 2 caractères.");

        SetErrors(nameof(Nom), erreurs);
    }

    private void ValiderEmail()
    {
        List<string> erreurs = new List<string>();

        if (string.IsNullOrWhiteSpace(Email))
            erreurs.Add("L'email est obligatoire.");
        else if (!Email.Contains("@") || !Email.Contains("."))
            erreurs.Add("Format d'email invalide.");

        SetErrors(nameof(Email), erreurs);
    }

    private void ValiderAge()
    {
        List<string> erreurs = new List<string>();

        if (Age < 18)
            erreurs.Add("Vous devez avoir au moins 18 ans.");
        else if (Age > 120)
            erreurs.Add("L'âge semble invalide.");

        SetErrors(nameof(Age), erreurs);
    }

    private void SetErrors(string propertyName, List<string> erreurs)
    {
        bool changed = false;

        if (erreurs.Count == 0)
        {
            changed = _erreurs.Remove(propertyName);
        }
        else
        {
            if (_erreurs.ContainsKey(propertyName))
            {
                if (!_erreurs[propertyName].SequenceEqual(erreurs))
                {
                    _erreurs[propertyName] = erreurs;
                    changed = true;
                }
            }
            else
            {
                _erreurs[propertyName] = erreurs;
                changed = true;
            }
        }

        if (changed)
            ErrorsChanged?.Invoke(this, new DataErrorsChangedEventArgs(propertyName));
    }

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    public bool EstValide()
    {
        ValiderNom();
        ValiderEmail();
        ValiderAge();

        return !HasErrors;
    }
}

private void DemoINotifyDataErrorInfo()
{
    // Créer un utilisateur
    UtilisateurModerne utilisateur = new UtilisateurModerne();

    // Créer un ErrorProvider
    ErrorProvider errorProvider = new ErrorProvider();

    // Créer les contrôles
    TextBox txtNom = new TextBox { Location = new Point(150, 20), Width = 200 };
    TextBox txtEmail = new TextBox { Location = new Point(150, 50), Width = 200 };
    NumericUpDown nudAge = new NumericUpDown { Location = new Point(150, 80), Width = 100, Minimum = 0, Maximum = 120 };

    // Ajouter des labels
    this.Controls.Add(new Label { Text = "Nom :", Location = new Point(20, 23), AutoSize = true });
    this.Controls.Add(new Label { Text = "Email :", Location = new Point(20, 53), AutoSize = true });
    this.Controls.Add(new Label { Text = "Âge :", Location = new Point(20, 83), AutoSize = true });

    // Ajouter les contrôles au formulaire
    this.Controls.Add(txtNom);
    this.Controls.Add(txtEmail);
    this.Controls.Add(nudAge);

    // Binding avec l'objet utilisateur
    txtNom.DataBindings.Add("Text", utilisateur, "Nom", true, DataSourceUpdateMode.OnPropertyChanged);
    txtEmail.DataBindings.Add("Text", utilisateur, "Email", true, DataSourceUpdateMode.OnPropertyChanged);
    nudAge.DataBindings.Add("Value", utilisateur, "Age", true, DataSourceUpdateMode.OnPropertyChanged);

    // Gérer les changements d'erreurs
    utilisateur.ErrorsChanged += (sender, e) =>
    {
        string propertyName = e.PropertyName;
        Control control = null;

        switch (propertyName)
        {
            case nameof(UtilisateurModerne.Nom):
                control = txtNom;
                break;
            case nameof(UtilisateurModerne.Email):
                control = txtEmail;
                break;
            case nameof(UtilisateurModerne.Age):
                control = nudAge;
                break;
        }

        if (control != null)
        {
            var erreurs = utilisateur.GetErrors(propertyName) as List<string>;
            errorProvider.SetError(control, erreurs != null && erreurs.Count > 0 ? erreurs[0] : "");
        }
    };

    // Bouton de validation
    Button btnValider = new Button
    {
        Text = "Valider",
        Location = new Point(150, 120),
        Width = 100
    };

    btnValider.Click += (sender, e) =>
    {
        if (utilisateur.EstValide())
        {
            MessageBox.Show(
                "Formulaire validé avec succès !\n\n" +
                $"Nom: {utilisateur.Nom}\n" +
                $"Email: {utilisateur.Email}\n" +
                $"Âge: {utilisateur.Age}",
                "Validation réussie",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);
        }
        else
        {
            StringBuilder erreurs = new StringBuilder();
            erreurs.AppendLine("Le formulaire contient des erreurs :");

            var properties = new[] { nameof(UtilisateurModerne.Nom), nameof(UtilisateurModerne.Email), nameof(UtilisateurModerne.Age) };

            foreach (var property in properties)
            {
                var propertyErrors = utilisateur.GetErrors(property) as List<string>;
                if (propertyErrors != null && propertyErrors.Count > 0)
                {
                    erreurs.AppendLine();
                    erreurs.AppendLine($"{property} :");
                    foreach (var error in propertyErrors)
                    {
                        erreurs.AppendLine($"- {error}");
                    }
                }
            }

            MessageBox.Show(
                erreurs.ToString(),
                "Erreurs de validation",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
        }
    };

    this.Controls.Add(btnValider);
}
```


## Les stratégies de validation

Il existe différentes stratégies pour déterminer quand la validation doit être déclenchée :

1. **Validation à la saisie (temps réel)** : Validation pendant que l'utilisateur saisit des données.
2. **Validation à la perte de focus** : Validation lorsque l'utilisateur quitte un contrôle.
3. **Validation à la soumission** : Validation uniquement lorsque l'utilisateur tente de soumettre le formulaire.

Vous pouvez combiner ces stratégies selon les besoins de votre application.

## Bonnes pratiques pour la validation des données

1. **Centraliser la logique de validation** : Implémentez la validation dans vos modèles de données plutôt que dans le code de l'interface utilisateur.
2. **Fournir un feedback immédiat** : Informez l'utilisateur dès qu'une erreur est détectée.
3. **Utiliser des messages d'erreur clairs** : Les messages doivent indiquer précisément ce qui est incorrect et comment le corriger.
4. **Valider côté client ET côté serveur** : La validation côté client améliore l'expérience utilisateur, mais la validation côté serveur est essentielle pour la sécurité.
5. **Considérer l'accessibilité** : Assurez-vous que les erreurs sont visibles et compréhensibles pour tous les utilisateurs, y compris ceux qui utilisent des technologies d'assistance.

La validation des données est un aspect fondamental du développement d'applications Windows Forms. Elle contribue à maintenir l'intégrité des données et à améliorer l'expérience utilisateur. Windows Forms propose plusieurs mécanismes pour implémenter cette validation, permettant de choisir l'approche la plus adaptée à vos besoins.

En utilisant les techniques appropriées, vous pouvez créer des formulaires robustes qui guident les utilisateurs et garantissent que seules des données valides sont traitées par votre application..

--

La liaison de données (Data Binding) est l'un des aspects les plus puissants et les plus utiles dans le développement d'applications Windows Forms. À travers les différentes sections que nous avons explorées, nous avons vu comment cette fonctionnalité permet de connecter facilement l'interface utilisateur avec les données sous-jacentes de l'application.
## Ce que nous avons appris
Dans la section 7.8.1, nous avons découvert le **binding simple**, qui constitue la base de la liaison de données dans Windows Forms. Cette approche directe permet de lier des propriétés individuelles de contrôles à des sources de données, créant ainsi une synchronisation automatique entre l'affichage et les données.
La section 7.8.2 nous a introduits au **binding complexe**, qui étend les capacités de liaison pour gérer des scénarios plus élaborés, tels que les collections d'objets et les relations hiérarchiques. Nous avons vu comment naviguer dans des structures de données complexes et comment mettre en place des liaisons qui reflètent ces relations.
Dans la section 7.8.3, nous avons exploré le rôle crucial de **BindingSource et BindingList**, qui constituent une couche intermédiaire facilitant la gestion des liaisons entre les contrôles et les données. Ces composants offrent des fonctionnalités étendues comme le tri, le filtrage, et la navigation simplifiée dans les collections de données.
Enfin, la section 7.8.4 a abordé la **validation de données**, un aspect essentiel pour garantir l'intégrité des informations saisies par l'utilisateur. Nous avons exploré diverses techniques de validation, depuis la simple vérification de champs jusqu'aux mécanismes avancés utilisant les interfaces `IDataErrorInfo` et `INotifyDataErrorInfo`.
## Avantages du Data Binding
La liaison de données offre plusieurs avantages significatifs dans le développement d'applications Windows Forms :
1. **Séparation des préoccupations** : Elle permet de séparer clairement la logique de présentation de la logique métier, rendant le code plus maintenable.
2. **Réduction du code répétitif** : Plus besoin d'écrire du code pour transférer manuellement les données entre les contrôles et les objets métier.
3. **Mise à jour automatique** : Les modifications apportées aux données sont automatiquement reflétées dans l'interface utilisateur et vice versa.
4. **Support des scénarios complexes** : Des fonctionnalités comme le binding complexe, le BindingSource et la validation permettent de gérer efficacement des interactions de données sophistiquées.
5. **Cohérence des données** : La validation intégrée aide à maintenir l'intégrité des données en empêchant les entrées invalides.

## Bonnes pratiques pour le Data Binding
Pour tirer le meilleur parti de la liaison de données dans vos applications Windows Forms, considérez ces bonnes pratiques :
- **Utilisez des modèles de données appropriés** : Concevez vos classes de données pour qu'elles fonctionnent bien avec le data binding, en implémentant des interfaces comme `INotifyPropertyChanged`.
- **Exploitez les BindingSources** : Utilisez-les comme intermédiaires entre vos contrôles et vos données pour bénéficier de fonctionnalités supplémentaires et simplifier la gestion des liaisons.
- **Implémentez une validation robuste** : Utilisez les mécanismes de validation pour garantir l'intégrité des données à tous les niveaux.
- **Préparez-vous au debugging** : La liaison de données peut parfois être difficile à déboguer. Surveillez les événements de binding et utilisez des outils de diagnostic appropriés.
- **Considérez la performance** : Pour les grands ensembles de données, soyez attentif à l'impact potentiel sur les performances et envisagez des approches optimisées comme la virtualisation.

## Vers l'avenir
Bien que Windows Forms soit une technologie mature, la liaison de données reste un concept fondamental dans le développement d'interfaces utilisateur modernes. Les principes que vous avez appris ici sont transférables à d'autres frameworks comme WPF, MAUI, ou même les applications web avec des frameworks JavaScript.
En maîtrisant les techniques de liaison de données présentées dans ce chapitre, vous disposez d'un outil puissant pour créer des applications Windows Forms robustes, réactives et faciles à maintenir. Ces compétences vous permettront de développer plus efficacement des interfaces utilisateur qui communiquent harmonieusement avec vos modèles de données, offrant ainsi une expérience utilisateur cohérente et intuitive.
La liaison de données représente l'un des exemples les plus concrets de la façon dont un bon design architectural peut simplifier le développement tout en améliorant la qualité du produit final. En continuant à explorer et à approfondir ces concepts, vous serez bien équipé pour relever les défis de développement d'applications plus complexes dans l'écosystème .NET.

⏭️
