# 7.7. Dialogues et messages

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les dialogues et messages sont des éléments essentiels de l'interface utilisateur qui permettent d'interagir avec l'utilisateur pour afficher des informations, demander des confirmations, ou recueillir des données. Windows Forms propose plusieurs dialogues standards ainsi que la possibilité de créer des dialogues personnalisés.
## 7.7.1. MessageBox et dialogues standards
### MessageBox pour afficher des messages
La classe `MessageBox` permet d'afficher rapidement des messages à l'utilisateur. C'est l'une des façons les plus simples d'interagir avec l'utilisateur.
``` csharp
// .NET Framework 4.7.2 et .NET 8
private void AfficherMessageBox()
{
    // Message simple
    MessageBox.Show("Ceci est un message simple.");

    // Message avec titre
    MessageBox.Show("Message avec titre.", "Titre du message");

    // Message avec boutons et icône
    DialogResult résultat = MessageBox.Show(
        "Voulez-vous enregistrer vos modifications ?",
        "Confirmation",
        MessageBoxButtons.YesNoCancel,
        MessageBoxIcon.Question);

    // Traitement du résultat
    switch (résultat)
    {
        case DialogResult.Yes:
            // Code pour enregistrer
            MessageBox.Show("Modifications enregistrées.", "Information",
                MessageBoxButtons.OK, MessageBoxIcon.Information);
            break;
        case DialogResult.No:
            // Code pour ignorer les modifications
            MessageBox.Show("Modifications ignorées.", "Information",
                MessageBoxButtons.OK, MessageBoxIcon.Information);
            break;
        case DialogResult.Cancel:
            // Code pour annuler l'opération
            MessageBox.Show("Opération annulée.", "Information",
                MessageBoxButtons.OK, MessageBoxIcon.Information);
            break;
    }
}
```
### Types de MessageBox
Le `MessageBox` peut être personnalisé avec différents boutons et icônes pour s'adapter au contexte :
``` csharp
// .NET Framework 4.7.2 et .NET 8
private void AfficherDifférentsTypesMessageBox()
{
    // Message d'information
    MessageBox.Show(
        "L'opération a été effectuée avec succès.",
        "Information",
        MessageBoxButtons.OK,
        MessageBoxIcon.Information);

    // Message d'avertissement
    MessageBox.Show(
        "Attention, cette action peut prendre plusieurs minutes.",
        "Avertissement",
        MessageBoxButtons.OKCancel,
        MessageBoxIcon.Warning);

    // Message d'erreur
    MessageBox.Show(
        "Une erreur s'est produite lors de l'opération.",
        "Erreur",
        MessageBoxButtons.RetryCancel,
        MessageBoxIcon.Error);

    // Message de question
    MessageBox.Show(
        "Voulez-vous continuer avec cette opération ?",
        "Question",
        MessageBoxButtons.YesNo,
        MessageBoxIcon.Question);

    // Message avec bouton par défaut spécifié
    MessageBox.Show(
        "Cette action va supprimer des fichiers. Continuer ?",
        "Confirmation",
        MessageBoxButtons.YesNo,
        MessageBoxIcon.Warning,
        MessageBoxDefaultButton.Button2);  // "Non" est le bouton par défaut
}
```
### Dialogues standards intégrés
Windows Forms propose plusieurs dialogues standards pour les opérations courantes :
``` csharp
// .NET Framework 4.7.2 et .NET 8
private void UtiliserDialoguesStandards()
{
    // Boîte de dialogue de sélection de couleur
    using (ColorDialog colorDialog = new ColorDialog())
    {
        colorDialog.Color = Color.Blue;  // Couleur initiale
        colorDialog.FullOpen = true;     // Afficher les couleurs personnalisées
        colorDialog.AllowFullOpen = true; // Permettre d'ouvrir la section couleurs personnalisées
        colorDialog.ShowHelp = true;     // Afficher le bouton d'aide

        if (colorDialog.ShowDialog() == DialogResult.OK)
        {
            // Utiliser la couleur sélectionnée
            this.BackColor = colorDialog.Color;
        }
    }

    // Boîte de dialogue de sélection de police
    using (FontDialog fontDialog = new FontDialog())
    {
        fontDialog.Font = this.Font;         // Police initiale
        fontDialog.ShowColor = true;         // Permettre de choisir la couleur
        fontDialog.ShowEffects = true;       // Afficher les effets (barré, souligné)
        fontDialog.MinSize = 8;              // Taille minimale
        fontDialog.MaxSize = 72;             // Taille maximale

        if (fontDialog.ShowDialog() == DialogResult.OK)
        {
            // Utiliser la police sélectionnée
            Label label = new Label();
            label.Text = "Texte avec la police sélectionnée";
            label.Font = fontDialog.Font;
            label.ForeColor = fontDialog.Color;
            label.AutoSize = true;
            label.Location = new Point(50, 50);
            this.Controls.Add(label);
        }
    }

    // Boîte de dialogue d'impression
    using (PrintDialog printDialog = new PrintDialog())
    {
        printDialog.AllowSomePages = true;   // Permettre la sélection de pages
        printDialog.PrinterSettings.MinimumPage = 1;
        printDialog.PrinterSettings.MaximumPage = 100;
        printDialog.PrinterSettings.FromPage = 1;
        printDialog.PrinterSettings.ToPage = 10;

        if (printDialog.ShowDialog() == DialogResult.OK)
        {
            // Utiliser les paramètres d'impression
            MessageBox.Show($"Imprimer de la page {printDialog.PrinterSettings.FromPage} " +
                           $"à la page {printDialog.PrinterSettings.ToPage}");
        }
    }

    // Boîte de dialogue d'aperçu avant impression
    PrintDocument document = new PrintDocument();
    document.PrintPage += (sender, e) => {
        // Code pour dessiner la page
        e.Graphics.DrawString("Page d'exemple", new Font("Arial", 12), Brushes.Black, 50, 50);
    };

    using (PrintPreviewDialog previewDialog = new PrintPreviewDialog())
    {
        previewDialog.Document = document;
        previewDialog.ShowDialog();
    }

    // Boîte de dialogue des paramètres de page
    using (PageSetupDialog pageSetupDialog = new PageSetupDialog())
    {
        pageSetupDialog.Document = document;
        pageSetupDialog.AllowMargins = true;     // Permettre de modifier les marges
        pageSetupDialog.AllowPaper = true;       // Permettre de choisir le format du papier
        pageSetupDialog.AllowOrientation = true; // Permettre de choisir l'orientation

        if (pageSetupDialog.ShowDialog() == DialogResult.OK)
        {
            // Utiliser les paramètres de page
            MessageBox.Show($"Format de page sélectionné : {pageSetupDialog.PageSettings.PaperSize.PaperName}");
        }
    }
}
```
### InputBox pour demander une entrée utilisateur
Contrairement à d'autres plateformes, Windows Forms ne fournit pas directement une boîte de dialogue `InputBox` pour recueillir une entrée utilisateur. Voici comment en créer une simple :
``` csharp
// .NET Framework 4.7.2 et .NET 8
/// <summary>
/// Affiche une boîte de dialogue avec un champ de saisie.
/// </summary>
/// <param name="titre">Titre de la boîte de dialogue</param>
/// <param name="message">Message à afficher</param>
/// <param name="valeurParDéfaut">Valeur par défaut du champ de saisie</param>
/// <returns>La valeur saisie, ou null si l'utilisateur a annulé</returns>
public static string InputBox(string titre, string message, string valeurParDéfaut = "")
{
    Form form = new Form();
    Label label = new Label();
    TextBox textBox = new TextBox();
    Button buttonOk = new Button();
    Button buttonAnnuler = new Button();

    form.Text = titre;
    form.ClientSize = new Size(300, 150);
    form.FormBorderStyle = FormBorderStyle.FixedDialog;
    form.StartPosition = FormStartPosition.CenterScreen;
    form.MinimizeBox = false;
    form.MaximizeBox = false;

    label.Text = message;
    label.Location = new Point(10, 20);
    label.AutoSize = true;

    textBox.Text = valeurParDéfaut;
    textBox.Location = new Point(10, 50);
    textBox.Width = 280;

    buttonOk.Text = "OK";
    buttonOk.DialogResult = DialogResult.OK;
    buttonOk.Location = new Point(120, 100);
    buttonOk.Anchor = AnchorStyles.Bottom | AnchorStyles.Right;

    buttonAnnuler.Text = "Annuler";
    buttonAnnuler.DialogResult = DialogResult.Cancel;
    buttonAnnuler.Location = new Point(205, 100);
    buttonAnnuler.Anchor = AnchorStyles.Bottom | AnchorStyles.Right;

    form.AcceptButton = buttonOk;
    form.CancelButton = buttonAnnuler;

    form.Controls.Add(label);
    form.Controls.Add(textBox);
    form.Controls.Add(buttonOk);
    form.Controls.Add(buttonAnnuler);

    return form.ShowDialog() == DialogResult.OK ? textBox.Text : null;
}

// Exemple d'utilisation
private void UtiliserInputBox()
{
    string nom = InputBox("Saisie", "Entrez votre nom :", "");
    if (nom != null)
    {
        MessageBox.Show($"Bonjour, {nom} !");
    }
    else
    {
        MessageBox.Show("Opération annulée.");
    }
}
```
## 7.7.2. Création de dialogues personnalisés
Lorsque les dialogues standards ne suffisent pas, vous pouvez créer des dialogues personnalisés en héritant de la classe `Form`.
### Création d'un dialogue simple
``` csharp
// .NET Framework 4.7.2 et .NET 8
public class DialoguePersonnalisé : Form
{
    private TextBox txtNom;
    private TextBox txtEmail;
    private CheckBox chkNewsletters;
    private Button btnOK;
    private Button btnAnnuler;

    // Propriétés pour accéder aux données saisies
    public string Nom => txtNom.Text;
    public string Email => txtEmail.Text;
    public bool AbonnéNewsletters => chkNewsletters.Checked;

    public DialoguePersonnalisé()
    {
        // Configuration du formulaire
        this.Text = "Informations utilisateur";
        this.ClientSize = new Size(350, 200);
        this.FormBorderStyle = FormBorderStyle.FixedDialog;
        this.MaximizeBox = false;
        this.MinimizeBox = false;
        this.StartPosition = FormStartPosition.CenterParent;

        // Création des contrôles
        Label lblNom = new Label();
        lblNom.Text = "Nom :";
        lblNom.Location = new Point(20, 20);
        lblNom.AutoSize = true;

        txtNom = new TextBox();
        txtNom.Location = new Point(150, 20);
        txtNom.Width = 180;

        Label lblEmail = new Label();
        lblEmail.Text = "Email :";
        lblEmail.Location = new Point(20, 60);
        lblEmail.AutoSize = true;

        txtEmail = new TextBox();
        txtEmail.Location = new Point(150, 60);
        txtEmail.Width = 180;

        chkNewsletters = new CheckBox();
        chkNewsletters.Text = "S'abonner aux newsletters";
        chkNewsletters.Location = new Point(20, 100);
        chkNewsletters.Width = 310;

        btnOK = new Button();
        btnOK.Text = "OK";
        btnOK.DialogResult = DialogResult.OK;
        btnOK.Location = new Point(174, 150);

        btnAnnuler = new Button();
        btnAnnuler.Text = "Annuler";
        btnAnnuler.DialogResult = DialogResult.Cancel;
        btnAnnuler.Location = new Point(255, 150);

        // Ajout des contrôles au formulaire
        this.Controls.Add(lblNom);
        this.Controls.Add(txtNom);
        this.Controls.Add(lblEmail);
        this.Controls.Add(txtEmail);
        this.Controls.Add(chkNewsletters);
        this.Controls.Add(btnOK);
        this.Controls.Add(btnAnnuler);

        // Configuration des boutons par défaut
        this.AcceptButton = btnOK;
        this.CancelButton = btnAnnuler;

        // Validation des champs avant de fermer le dialogue
        this.FormClosing += (s, e) => {
            if (this.DialogResult == DialogResult.OK)
            {
                if (string.IsNullOrWhiteSpace(txtNom.Text))
                {
                    MessageBox.Show("Veuillez entrer un nom.", "Erreur",
                        MessageBoxButtons.OK, MessageBoxIcon.Error);
                    txtNom.Focus();
                    e.Cancel = true;
                    return;
                }

                if (string.IsNullOrWhiteSpace(txtEmail.Text) || !txtEmail.Text.Contains("@"))
                {
                    MessageBox.Show("Veuillez entrer une adresse email valide.", "Erreur",
                        MessageBoxButtons.OK, MessageBoxIcon.Error);
                    txtEmail.Focus();
                    e.Cancel = true;
                    return;
                }
            }
        };
    }
}

// Exemple d'utilisation du dialogue personnalisé
private void UtiliserDialoguePersonnalisé()
{
    using (DialoguePersonnalisé dialogue = new DialoguePersonnalisé())
    {
        if (dialogue.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show($"Nom: {dialogue.Nom}\nEmail: {dialogue.Email}\n" +
                           $"Abonné aux newsletters: {dialogue.AbonnéNewsletters}",
                           "Informations saisies", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show("Opération annulée.", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}
```
### Dialogue avec un design plus élaboré
``` csharp
// .NET Framework 4.7.2 et .NET 8
public class DialogueOptions : Form
{
    private TabControl tabControl;
    private NumericUpDown nudNombreÉléments;
    private CheckBox chkSauvegardeAuto;
    private ComboBox cboThème;
    private RadioButton optTailleSmall;
    private RadioButton optTailleMedium;
    private RadioButton optTailleLarge;
    private Button btnOK;
    private Button btnAnnuler;
    private Button btnAppliquer;

    // Propriétés pour accéder aux options
    public int NombreÉléments => (int)nudNombreÉléments.Value;
    public bool SauvegardeAutomatique => chkSauvegardeAuto.Checked;
    public string Thème => cboThème.SelectedItem.ToString();
    public string TailleInterface
    {
        get
        {
            if (optTailleSmall.Checked) return "Petite";
            if (optTailleMedium.Checked) return "Moyenne";
            if (optTailleLarge.Checked) return "Grande";
            return "Moyenne";
        }
    }

    public DialogueOptions()
    {
        // Configuration du formulaire
        this.Text = "Options";
        this.ClientSize = new Size(400, 300);
        this.FormBorderStyle = FormBorderStyle.FixedDialog;
        this.MaximizeBox = false;
        this.MinimizeBox = false;
        this.StartPosition = FormStartPosition.CenterParent;

        // Création du TabControl
        tabControl = new TabControl();
        tabControl.Location = new Point(10, 10);
        tabControl.Size = new Size(380, 240);

        // Onglet Général
        TabPage tabGénéral = new TabPage("Général");

        Label lblNombreÉléments = new Label();
        lblNombreÉléments.Text = "Nombre d'éléments récents :";
        lblNombreÉléments.Location = new Point(10, 20);
        lblNombreÉléments.AutoSize = true;

        nudNombreÉléments = new NumericUpDown();
        nudNombreÉléments.Location = new Point(200, 20);
        nudNombreÉléments.Width = 60;
        nudNombreÉléments.Minimum = 1;
        nudNombreÉléments.Maximum = 20;
        nudNombreÉléments.Value = 10;

        chkSauvegardeAuto = new CheckBox();
        chkSauvegardeAuto.Text = "Sauvegarde automatique";
        chkSauvegardeAuto.Location = new Point(10, 60);
        chkSauvegardeAuto.Checked = true;
        chkSauvegardeAuto.AutoSize = true;

        tabGénéral.Controls.Add(lblNombreÉléments);
        tabGénéral.Controls.Add(nudNombreÉléments);
        tabGénéral.Controls.Add(chkSauvegardeAuto);

        // Onglet Apparence
        TabPage tabApparence = new TabPage("Apparence");

        Label lblThème = new Label();
        lblThème.Text = "Thème :";
        lblThème.Location = new Point(10, 20);
        lblThème.AutoSize = true;

        cboThème = new ComboBox();
        cboThème.Location = new Point(100, 20);
        cboThème.Width = 150;
        cboThème.DropDownStyle = ComboBoxStyle.DropDownList;
        cboThème.Items.AddRange(new string[] { "Clair", "Sombre", "Bleu", "Personnalisé" });
        cboThème.SelectedIndex = 0;

        GroupBox grpTaille = new GroupBox();
        grpTaille.Text = "Taille de l'interface";
        grpTaille.Location = new Point(10, 60);
        grpTaille.Size = new Size(340, 100);

        optTailleSmall = new RadioButton();
        optTailleSmall.Text = "Petite";
        optTailleSmall.Location = new Point(20, 20);
        optTailleSmall.AutoSize = true;

        optTailleMedium = new RadioButton();
        optTailleMedium.Text = "Moyenne";
        optTailleMedium.Location = new Point(20, 50);
        optTailleMedium.AutoSize = true;
        optTailleMedium.Checked = true;

        optTailleLarge = new RadioButton();
        optTailleLarge.Text = "Grande";
        optTailleLarge.Location = new Point(20, 80);
        optTailleLarge.AutoSize = true;

        grpTaille.Controls.Add(optTailleSmall);
        grpTaille.Controls.Add(optTailleMedium);
        grpTaille.Controls.Add(optTailleLarge);

        tabApparence.Controls.Add(lblThème);
        tabApparence.Controls.Add(cboThème);
        tabApparence.Controls.Add(grpTaille);

        tabControl.Controls.Add(tabGénéral);
        tabControl.Controls.Add(tabApparence);

        // Boutons
        btnOK = new Button();
        btnOK.Text = "OK";
        btnOK.DialogResult = DialogResult.OK;
        btnOK.Location = new Point(220, 260);
        btnOK.Width = 80;

        btnAnnuler = new Button();
        btnAnnuler.Text = "Annuler";
        btnAnnuler.DialogResult = DialogResult.Cancel;
        btnAnnuler.Location = new Point(310, 260);
        btnAnnuler.Width = 80;

        btnAppliquer = new Button();
        btnAppliquer.Text = "Appliquer";
        btnAppliquer.Location = new Point(130, 260);
        btnAppliquer.Width = 80;
        btnAppliquer.Click += (s, e) => AppliquerOptions();

        // Ajout des contrôles au formulaire
        this.Controls.Add(tabControl);
        this.Controls.Add(btnOK);
        this.Controls.Add(btnAnnuler);
        this.Controls.Add(btnAppliquer);

        // Configuration des boutons par défaut
        this.AcceptButton = btnOK;
        this.CancelButton = btnAnnuler;

        // Intercepter le clic sur OK pour appliquer les options
        this.FormClosing += (s, e) => {
            if (this.DialogResult == DialogResult.OK)
            {
                AppliquerOptions();
            }
        };
    }

    private void AppliquerOptions()
    {
        // Dans une application réelle, cette méthode appliquerait les options
        MessageBox.Show($"Options appliquées :\n" +
                       $"- Nombre d'éléments : {NombreÉléments}\n" +
                       $"- Sauvegarde auto : {SauvegardeAutomatique}\n" +
                       $"- Thème : {Thème}\n" +
                       $"- Taille : {TailleInterface}",
                       "Options appliquées", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }
}

// Exemple d'utilisation du dialogue d'options
private void UtiliserDialogueOptions()
{
    using (DialogueOptions dialogue = new DialogueOptions())
    {
        if (dialogue.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show("Options enregistrées.", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show("Modifications annulées.", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}
```
### Dialogue avec progression
``` csharp
// .NET Framework 4.7.2 et .NET 8
public class DialogueProgression : Form
{
    private ProgressBar progressBar;
    private Label lblStatut;
    private Button btnAnnuler;
    private CancellationTokenSource cts;

    public DialogueProgression(string titre)
    {
        // Configuration du formulaire
        this.Text = titre;
        this.ClientSize = new Size(400, 120);
        this.FormBorderStyle = FormBorderStyle.FixedDialog;
        this.MaximizeBox = false;
        this.MinimizeBox = false;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ControlBox = false;  // Désactiver le bouton de fermeture

        // Création des contrôles
        progressBar = new ProgressBar();
        progressBar.Location = new Point(20, 20);
        progressBar.Size = new Size(360, 30);
        progressBar.Style = ProgressBarStyle.Continuous;
        progressBar.Minimum = 0;
        progressBar.Maximum = 100;

        lblStatut = new Label();
        lblStatut.Location = new Point(20, 60);
        lblStatut.Size = new Size(360, 20);
        lblStatut.Text = "Initialisation...";

        btnAnnuler = new Button();
        btnAnnuler.Text = "Annuler";
        btnAnnuler.Location = new Point(300, 90);
        btnAnnuler.Size = new Size(80, 23);
        btnAnnuler.Click += (s, e) => {
            cts?.Cancel();
            lblStatut.Text = "Annulation en cours...";
            btnAnnuler.Enabled = false;
        };

        // Ajout des contrôles au formulaire
        this.Controls.Add(progressBar);
        this.Controls.Add(lblStatut);
        this.Controls.Add(btnAnnuler);

        // Créer un jeton d'annulation
        cts = new CancellationTokenSource();
    }

    // Propriété pour obtenir le jeton d'annulation
    public CancellationToken Token => cts.Token;

    // Méthode pour mettre à jour la progression
    public void MettreÀJourProgression(int pourcentage, string message)
    {
        // S'assurer que la mise à jour se fait sur le thread UI
        if (this.InvokeRequired)
        {
            this.Invoke(new Action(() => MettreÀJourProgression(pourcentage, message)));
            return;
        }

        progressBar.Value = pourcentage;
        lblStatut.Text = message;

        // Rafraîchir l'interface
        Application.DoEvents();
    }

    // Méthode pour exécuter une tâche asynchrone
    public async Task<bool> ExécuterTâcheAsync(Func<IProgress<(int, string)>, CancellationToken, Task> tâche)
    {
        try
        {
            // Créer un rapporteur de progression
            var progression = new Progress<(int pourcentage, string message)>(info => {
                MettreÀJourProgression(info.pourcentage, info.message);
            });

            // Exécuter la tâche
            await tâche(progression, cts.Token);

            // Si on arrive ici, la tâche s'est terminée avec succès
            this.DialogResult = DialogResult.OK;
            return true;
        }
        catch (OperationCanceledException)
        {
            // La tâche a été annulée
            this.DialogResult = DialogResult.Cancel;
            return false;
        }
        catch (Exception ex)
        {
            // Une erreur s'est produite
            MessageBox.Show($"Une erreur s'est produite: {ex.Message}", "Erreur",
                           MessageBoxButtons.OK, MessageBoxIcon.Error);
            this.DialogResult = DialogResult.Abort;
            return false;
        }
        finally
        {
            // Libérer les ressources
            cts.Dispose();
        }
    }
}

// Exemple d'utilisation du dialogue de progression
private async void UtiliserDialogueProgression()
{
    using (DialogueProgression dialogue = new DialogueProgression("Opération en cours"))
    {
        // Lancer la tâche en arrière-plan
        Task<bool> tâche = dialogue.ExécuterTâcheAsync(async (progress, token) => {
            // Simuler une tâche longue
            for (int i = 0; i <= 100; i += 5)
            {
                // Vérifier si l'annulation a été demandée
                token.ThrowIfCancellationRequested();

                // Simuler un travail
                await Task.Delay(200, token);

                // Rapporter la progression
                progress.Report((i, $"Traitement en cours... {i}%"));
            }
        });

        // Afficher le dialogue pendant que la tâche s'exécute
        dialogue.ShowDialog();

        // Vérifier le résultat de la tâche
        bool réussite = await tâche;

        if (réussite)
        {
            MessageBox.Show("Opération terminée avec succès.", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show("L'opération a été annulée.", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}
```

## 7.7.3. DialogResult

La classe `DialogResult` est utilisée pour déterminer comment un dialogue a été fermé. Elle permet de coordonner les actions entre un formulaire parent et un formulaire de dialogue.

### Utilisation de DialogResult

```textmate
// .NET Framework 4.7.2 et .NET 8
private void UtiliserDialogResult()
{
    // Exemple 1 : MessageBox
    DialogResult résultat = MessageBox.Show(
        "Voulez-vous enregistrer avant de quitter ?",
        "Confirmation",
        MessageBoxButtons.YesNoCancel,
        MessageBoxIcon.Question);

    switch (résultat)
    {
        case DialogResult.Yes:
            MessageBox.Show("Enregistrement en cours...");
            break;
        case DialogResult.No:
            MessageBox.Show("Fermeture sans enregistrement...");
            break;
        case DialogResult.Cancel:
            MessageBox.Show("Opération annulée");
            break;
    }

    // Exemple 2 : Définir le DialogResult d'un bouton
    Button btnOK = new Button();
    btnOK.Text = "OK";
    btnOK.DialogResult = DialogResult.OK;

    Button btnAnnuler = new Button();
    btnAnnuler.Text = "Annuler";
    btnAnnuler.DialogResult = DialogResult.Cancel;

    // Exemple 3 : Définir le DialogResult manuellement
    Form form = new Form();
    form.Text = "Exemple DialogResult";

    Button btnFermerOK = new Button();
    btnFermerOK.Text = "Fermer (OK)";
    btnFermerOK.Location = new Point(50, 50);
    btnFermerOK.Click += (s, e) => {
        // Exécuter du code avant de fermer le formulaire
        MessageBox.Show("Traitement avant fermeture...");
        // Définir le DialogResult et fermer le formulaire
        form.DialogResult = DialogResult.OK;
    };

    Button btnFermerAnnuler = new Button();
    btnFermerAnnuler.Text = "Fermer (Annuler)";
    btnFermerAnnuler.Location = new Point(50, 100);
    btnFermerAnnuler.Click += (s, e) => {
        form.DialogResult = DialogResult.Cancel;
    };

    form.Controls.Add(btnFermerOK);
    form.Controls.Add(btnFermerAnnuler);

    // Exemple 4 : Utiliser le résultat d'un dialogue
    if (form.ShowDialog() == DialogResult.OK)
    {
        MessageBox.Show("Vous avez cliqué sur OK");
    }
    else
    {
        MessageBox.Show("Vous avez cliqué sur Annuler");
    }
}
```


### Valeurs de DialogResult

L'énumération `DialogResult` définit les valeurs suivantes :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExaminerValeursDialogResult()
{
    // None : Aucun résultat, le dialogue reste généralement ouvert
    Button btnSansFermeture = new Button();
    btnSansFermeture.DialogResult = DialogResult.None;

    // OK : Action confirmée, dialogue fermé avec succès
    Button btnOK = new Button();
    btnOK.DialogResult = DialogResult.OK;

    // Cancel : Action annulée
    Button btnAnnuler = new Button();
    btnAnnuler.DialogResult = DialogResult.Cancel;

    // Abort : Action abandonnée (généralement en cas d'erreur)
    Button btnAbandonner = new Button();
    btnAbandonner.DialogResult = DialogResult.Abort;

    // Retry : Recommencer l'action
    Button btnRéessayer = new Button();
    btnRéessayer.DialogResult = DialogResult.Retry;

    // Ignore : Ignorer l'erreur et continuer
    Button btnIgnorer = new Button();
    btnIgnorer.DialogResult = DialogResult.Ignore;

    // Yes : Réponse affirmative à une question
    Button btnOui = new Button();
    btnOui.DialogResult = DialogResult.Yes;

    // No : Réponse négative à une question
    Button btnNon = new Button();
    btnNon.DialogResult = DialogResult.No;
}
```


### Validation avant fermeture

Il est souvent nécessaire de valider les entrées utilisateur avant de fermer un dialogue avec un résultat positif :

```textmate
// .NET Framework 4.7.2 et .NET 8
public class DialogueValidation : Form
{
    private TextBox txtNom;
    private TextBox txtÂge;
    private Button btnOK;
    private Button btnAnnuler;

    public string Nom => txtNom.Text;
    public int Âge { get; private set; }

    public DialogueValidation()
    {
        this.Text = "Validation";
        this.ClientSize = new Size(300, 150);
        this.FormBorderStyle = FormBorderStyle.FixedDialog;
        this.StartPosition = FormStartPosition.CenterParent;
        this.MaximizeBox = false;
        this.MinimizeBox = false;

        Label lblNom = new Label();
        lblNom.Text = "Nom :";
        lblNom.Location = new Point(20, 20);
        lblNom.AutoSize = true;

        txtNom = new TextBox();
        txtNom.Location = new Point(100, 20);
        txtNom.Width = 180;

        Label lblÂge = new Label();
        lblÂge.Text = "Âge :";
        lblÂge.Location = new Point(20, 50);
        lblÂge.AutoSize = true;

        txtÂge = new TextBox();
        txtÂge.Location = new Point(100, 50);
        txtÂge.Width = 180;

        btnOK = new Button();
        btnOK.Text = "OK";
        btnOK.DialogResult = DialogResult.OK;
        btnOK.Location = new Point(120, 100);

        btnAnnuler = new Button();
        btnAnnuler.Text = "Annuler";
        btnAnnuler.DialogResult = DialogResult.Cancel;
        btnAnnuler.Location = new Point(200, 100);

        this.Controls.Add(lblNom);
        this.Controls.Add(txtNom);
        this.Controls.Add(lblÂge);
        this.Controls.Add(txtÂge);
        this.Controls.Add(btnOK);
        this.Controls.Add(btnAnnuler);

        this.AcceptButton = btnOK;
        this.CancelButton = btnAnnuler;

        // Intercepter le clic sur OK pour valider les données
        this.FormClosing += (s, e) => {
            if (this.DialogResult == DialogResult.OK)
            {
                // Valider le nom
                if (string.IsNullOrWhiteSpace(txtNom.Text))
                {
                    MessageBox.Show("Veuillez entrer un nom.", "Erreur",
                        MessageBoxButtons.OK, MessageBoxIcon.Error);
                    txtNom.Focus();
                    e.Cancel = true;  // Empêcher la fermeture du formulaire
                    return;
                }

                // Valider l'âge
                if (!int.TryParse(txtÂge.Text, out int âge) || âge < 0 || âge > 120)
                {
                    MessageBox.Show("Veuillez entrer un âge valide (entre 0 et 120).", "Erreur",
                        MessageBoxButtons.OK, MessageBoxIcon.Error);
                    txtÂge.Focus();
                    e.Cancel = true;  // Empêcher la fermeture du formulaire
                    return;
                }

                // Stocker l'âge validé
                Âge = âge;
            }
        };
    }
}

// Exemple d'utilisation
private void UtiliserDialogueValidation()
{
    using (DialogueValidation dialogue = new DialogueValidation())
    {
        if (dialogue.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show($"Nom: {dialogue.Nom}, Âge: {dialogue.Âge}",
                           "Données validées", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        else
        {
            MessageBox.Show("Opération annulée.", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}
```


## 7.7.4. Dialogues de fichiers et dossiers

Windows Forms propose plusieurs dialogues standards pour la gestion des fichiers et dossiers.

### OpenFileDialog

Le dialogue `OpenFileDialog` permet à l'utilisateur de sélectionner un ou plusieurs fichiers à ouvrir :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void UtiliserOpenFileDialog()
{
    using (OpenFileDialog dlg = new OpenFileDialog())
    {
        // Configuration
        dlg.Title = "Sélectionner un fichier";
        dlg.Filter = "Documents texte (*.txt)|*.txt|Documents Word (*.docx)|*.docx|Tous les fichiers (*.*)|*.*";
        dlg.FilterIndex = 1;  // L'index commence à 1 (premier filtre)
        dlg.DefaultExt = "txt";
        dlg.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
        dlg.Multiselect = false;  // Sélection d'un seul fichier
        dlg.CheckFileExists = true;  // Vérifier que le fichier existe
        dlg.RestoreDirectory = true;  // Revenir au dossier initial après fermeture

        if (dlg.ShowDialog() == DialogResult.OK)
        {
            // Utiliser le fichier sélectionné
            string fichier = dlg.FileName;
            MessageBox.Show($"Fichier sélectionné : {fichier}", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }

    // Sélection multiple de fichiers
    using (OpenFileDialog dlg = new OpenFileDialog())
    {
        dlg.Title = "Sélectionner plusieurs fichiers";
        dlg.Filter = "Images (*.jpg;*.png;*.gif)|*.jpg;*.png;*.gif|Tous les fichiers (*.*)|*.*";
        dlg.Multiselect = true;  // Sélection multiple

        if (dlg.ShowDialog() == DialogResult.OK)
        {
            // Utiliser les fichiers sélectionnés
            string[] fichiers = dlg.FileNames;
            string message = $"Fichiers sélectionnés ({fichiers.Length}) :\n";
            foreach (string fichier in fichiers)
            {
                message += $"- {Path.GetFileName(fichier)}\n";
            }
            MessageBox.Show(message, "Information", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}
```


### SaveFileDialog

Le dialogue `SaveFileDialog` permet à l'utilisateur de spécifier un fichier pour enregistrer des données :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void UtiliserSaveFileDialog()
{
    using (SaveFileDialog dlg = new SaveFileDialog())
    {
        // Configuration
        dlg.Title = "Enregistrer le document";
        dlg.Filter = "Document texte (*.txt)|*.txt|Document Word (*.docx)|*.docx|Tous les fichiers (*.*)|*.*";
        dlg.FilterIndex = 1;
        dlg.DefaultExt = "txt";
        dlg.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
        dlg.FileName = "Document sans titre";  // Nom de fichier par défaut
        dlg.OverwritePrompt = true;  // Demander confirmation si le fichier existe déjà
        dlg.CreatePrompt = false;  // Ne pas demander confirmation pour créer le fichier
        dlg.RestoreDirectory = true;

        if (dlg.ShowDialog() == DialogResult.OK)
        {
            // Utiliser le fichier spécifié
            string fichier = dlg.FileName;
            MessageBox.Show($"Document enregistré sous : {fichier}", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);

            // Dans une application réelle, vous écririez les données dans le fichier
            try
            {
                // Exemple d'écriture de texte dans le fichier
                File.WriteAllText(fichier, "Contenu du document");
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Erreur lors de l'enregistrement : {ex.Message}", "Erreur",
                               MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}
```


### FolderBrowserDialog

Le dialogue `FolderBrowserDialog` permet à l'utilisateur de sélectionner un dossier :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void UtiliserFolderBrowserDialog()
{
    using (FolderBrowserDialog dlg = new FolderBrowserDialog())
    {
        // Configuration
        dlg.Description = "Sélectionner un dossier";

        // .NET Framework 4.7.2
        dlg.RootFolder = Environment.SpecialFolder.Desktop;  // Dossier de départ
        dlg.SelectedPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);  // Sélection initiale
        dlg.ShowNewFolderButton = true;  // Autoriser la création de nouveaux dossiers

        // .NET Core / .NET 8 a des options supplémentaires
        #if NET5_0_OR_GREATER
        dlg.UseDescriptionForTitle = true;  // Utiliser la description comme titre
        dlg.AutoUpgradeEnabled = true;  // Utiliser le dialogue moderne
        #endif

        if (dlg.ShowDialog() == DialogResult.OK)
        {
            // Utiliser le dossier sélectionné
            string dossier = dlg.SelectedPath;
            MessageBox.Show($"Dossier sélectionné : {dossier}", "Information",
                           MessageBoxButtons.OK, MessageBoxIcon.Information);

            // Exemple d'utilisation : lister les fichiers du dossier
            try
            {
                string[] fichiers = Directory.GetFiles(dossier);
                string message = $"Fichiers dans le dossier ({fichiers.Length}) :\n";
                foreach (string fichier in fichiers.Take(10))  // Limiter à 10 fichiers pour l'affichage
                {
                    message += $"- {Path.GetFileName(fichier)}\n";
                }
                if (fichiers.Length > 10)
                {
                    message += "...";
                }
                MessageBox.Show(message, "Information", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Erreur lors de la lecture du dossier : {ex.Message}", "Erreur",
                               MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}
```


### Filtres de fichiers

Les filtres de fichiers dans les dialogues `OpenFileDialog` et `SaveFileDialog` permettent de limiter les types de fichiers affichés et sélectionnables :

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExemplesFiltresFichiers()
{
    // Filtre simple : un seul type de fichier
    string filtreTxt = "Fichiers texte (*.txt)|*.txt";

    // Filtre avec plusieurs extensions pour un même type
    string filtreImages = "Images (*.jpg;*.jpeg;*.png;*.gif;*.bmp)|*.jpg;*.jpeg;*.png;*.gif;*.bmp";

    // Filtre avec plusieurs types de fichiers séparés par des |
    string filtreDocuments = "Documents Word (*.doc;*.docx)|*.doc;*.docx|Documents PDF (*.pdf)|*.pdf|Documents texte (*.txt)|*.txt";

    // Toujours inclure l'option "Tous les fichiers" à la fin
    string filtreComplet = "Images (*.jpg;*.png)|*.jpg;*.png|Documents (*.doc;*.pdf)|*.doc;*.pdf|Tous les fichiers (*.*)|*.*";

    // Exemple d'utilisation
    using (OpenFileDialog dlg = new OpenFileDialog())
    {
        dlg.Filter = filtreComplet;
        dlg.FilterIndex = 1;  // Premier filtre sélectionné par défaut

        if (dlg.ShowDialog() == DialogResult.OK)
        {
            MessageBox.Show($"Fichier sélectionné : {dlg.FileName}");
        }
    }

    // Création dynamique de filtres
    Dictionary<string, string[]> typesFichiers = new Dictionary<string, string[]>
    {
        { "Images", new[] { "jpg", "jpeg", "png", "gif", "bmp" } },
        { "Documents", new[] { "doc", "docx", "pdf", "txt" } },
        { "Vidéos", new[] { "mp4", "avi", "mov", "wmv" } }
    };

    string filtreDynamique = "";
    foreach (var type in typesFichiers)
    {
        string extensions = string.Join(";*.", type.Value);
        filtreDynamique += $"{type.Key} (*.{extensions})|*.{extensions}|";
    }
    filtreDynamique += "Tous les fichiers (*.*)|*.*";

    MessageBox.Show($"Filtre dynamique créé : {filtreDynamique}");
}
```


### Utilisation avancée

```textmate
// .NET Framework 4.7.2 et .NET 8
private void ExemplesAvancésDialoguesFichiers()
{
    // Exemple 1 : Dialogue personnalisé combinant sélection de fichier et options
    OpenFileDialog dlgFichier = new OpenFileDialog();
    dlgFichier.Filter = "Images (*.jpg;*.png)|*.jpg;*.png|Tous les fichiers (*.*)|*.*";
    dlgFichier.Title = "Sélectionner une image";

    // Créer un formulaire personnalisé pour afficher le dialogue avec des options
    Form formOptions = new Form();
    formOptions.Text = "Options d'ouverture";
    formOptions.Size = new Size(500, 400);
    formOptions.StartPosition = FormStartPosition.CenterScreen;
    formOptions.FormBorderStyle = FormBorderStyle.FixedDialog;
    formOptions.MaximizeBox = false;
    formOptions.MinimizeBox = false;

    // Ajouter un FileDialog dans un formulaire personnalisé
    FileDialogControlHost host = new FileDialogControlHost(dlgFichier);
    host.Dock = DockStyle.Fill;

    // Ajouter des options supplémentaires
    Panel panelOptions = new Panel();
    panelOptions.Dock = DockStyle.Bottom;
    panelOptions.Height = 100;

    CheckBox chkOuvrirEnLecture = new CheckBox();
    chkOuvrirEnLecture.Text = "Ouvrir en lecture seule";
    chkOuvrirEnLecture.Location = new Point(20, 20);

    CheckBox chkModifierTaille = new CheckBox();
    chkModifierTaille.Text = "Redimensionner à l'ouverture";
    chkModifierTaille.Location = new Point(20, 50);

    panelOptions.Controls.Add(chkOuvrirEnLecture);
    panelOptions.Controls.Add(chkModifierTaille);

    formOptions.Controls.Add(host);
    formOptions.Controls.Add(panelOptions);

    if (formOptions.ShowDialog() == DialogResult.OK)
    {
        string fichier = dlgFichier.FileName;
        bool lecturSeule = chkOuvrirEnLecture.Checked;
        bool redimensionner = chkModifierTaille.Checked;

        MessageBox.Show($"Fichier : {fichier}\nLecture seule : {lecturSeule}\nRedimensionner : {redimensionner}");
    }

    // Exemple 2 : Enregistrer le dernier dossier utilisé
    string dernierDossier = Properties.Settings.Default.DernierDossier;
    if (string.IsNullOrEmpty(dernierDossier) || !Directory.Exists(dernierDossier))
    {
        dernierDossier = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
    }

    using (OpenFileDialog dlg = new OpenFileDialog())
    {
        dlg.InitialDirectory = dernierDossier;
        dlg.Filter = "Documents texte (*.txt)|*.txt|Tous les fichiers (*.*)|*.*";

        if (dlg.ShowDialog() == DialogResult.OK)
        {
            // Enregistrer le dossier du fichier sélectionné
            Properties.Settings.Default.DernierDossier = Path.GetDirectoryName(dlg.FileName);
            Properties.Settings.Default.Save();

            MessageBox.Show($"Fichier sélectionné : {dlg.FileName}\nDossier enregistré : {Properties.Settings.Default.DernierDossier}");
        }
    }
}

// Classe utilitaire pour héberger un FileDialog dans un formulaire personnalisé
public class FileDialogControlHost : Control
{
    private FileDialog _dialog;
    private NativeWindow _window;

    public FileDialogControlHost(FileDialog dialog)
    {
        _dialog = dialog;
        _window = new NativeWindow();
    }

    protected override void OnHandleCreated(EventArgs e)
    {
        base.OnHandleCreated(e);

        if (this.ParentForm != null)
        {
            _window.AssignHandle(this.Handle);
            this.ParentForm.BeginInvoke(new Action(() => {
                _dialog.ShowDialog(this.ParentForm);
                this.ParentForm.DialogResult = _dialog.DialogResult;
                this.ParentForm.Close();
            }));
        }
    }

    protected override void OnHandleDestroyed(EventArgs e)
    {
        _window.ReleaseHandle();
        base.OnHandleDestroyed(e);
    }
}
```


### Résumé

Les dialogues sont un élément essentiel de l'interface utilisateur qui permettent d'interagir avec l'utilisateur de manière standardisée. Windows Forms propose une variété de dialogues prédéfinis (MessageBox, OpenFileDialog, SaveFileDialog, etc.) et permet également de créer des dialogues personnalisés pour répondre à des besoins spécifiques.

L'utilisation de `DialogResult` permet de coordonner les actions entre les formulaires et de déterminer comment l'utilisateur a répondu à un dialogue. Les dialogues de fichiers et dossiers facilitent la sélection et l'enregistrement de fichiers avec des options flexibles pour filtrer les types de fichiers affichés.

En combinant ces outils, vous pouvez créer des interfaces utilisateur intuitives et conviviales qui guident efficacement l'utilisateur dans ses interactions avec votre application.

⏭️ 7.8. [Liaison de données (Data Binding)](/07-windows-forms-winforms/7-8-liaison-de-donnees-data-binding.md)
