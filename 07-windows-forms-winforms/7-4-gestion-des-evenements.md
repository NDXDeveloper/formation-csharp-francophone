# 7.4. Gestion des événements

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les événements sont au cœur de la programmation Windows Forms et permettent de répondre aux interactions de l'utilisateur ainsi qu'aux changements d'état des contrôles. Cette section explore le modèle d'événements .NET et comment l'utiliser efficacement dans vos applications.

## 7.4.1. Modèle d'événement .NET

Le modèle d'événement .NET est basé sur le modèle délégué, où un délégué est essentiellement un type qui représente une référence à une méthode. Les événements utilisent généralement le délégué `EventHandler` ou `EventHandler<TEventArgs>` pour permettre à plusieurs méthodes de s'abonner à un événement.

### Structure de base des événements

```textmate
// .NET Framework 4.7.2 et .NET 8
public class SimpleEventDemo
{
    // 1. Déclarer un délégué (ou utiliser EventHandler prédéfini)
    public delegate void MonDéléguéEvénement(object sender, EventArgs e);

    // 2. Déclarer un événement basé sur ce délégué
    public event MonDéléguéEvénement MonÉvénement;

    // Utilisation du délégué générique EventHandler
    public event EventHandler AutreÉvénement;

    // 3. Méthode pour déclencher l'événement
    protected virtual void DéclencherMonÉvénement()
    {
        // Vérifier si l'événement a des abonnés
        MonÉvénement?.Invoke(this, EventArgs.Empty);
    }

    protected virtual void DéclencherAutreÉvénement()
    {
        // Vérification et déclenchement en une seule ligne
        AutreÉvénement?.Invoke(this, EventArgs.Empty);
    }

    // Méthode qui effectue une action et déclenche l'événement
    public void EffectuerAction()
    {
        Console.WriteLine("Action effectuée");
        DéclencherMonÉvénement();
    }
}

// Utilisation des événements
public void TesterÉvénements()
{
    var demo = new SimpleEventDemo();

    // S'abonner à l'événement avec une méthode nommée
    demo.MonÉvénement += GestionnaireÉvénement;

    // S'abonner à l'événement avec une expression lambda
    demo.AutreÉvénement += (sender, e) => {
        Console.WriteLine("AutreÉvénement déclenché via lambda");
    };

    // Déclencher l'événement
    demo.EffectuerAction();

    // Se désabonner de l'événement
    demo.MonÉvénement -= GestionnaireÉvénement;
}

// Méthode gestionnaire d'événement
private void GestionnaireÉvénement(object sender, EventArgs e)
{
    Console.WriteLine("MonÉvénement a été déclenché");
}
```


### EventArgs personnalisés

Pour passer des données supplémentaires lors du déclenchement d'un événement, vous pouvez créer une classe personnalisée qui hérite de `EventArgs`.

```textmate
// .NET Framework 4.7.2 et .NET 8
// Classe d'arguments d'événement personnalisée
public class ProgressionEventArgs : EventArgs
{
    public int Pourcentage { get; }
    public string Message { get; }

    public ProgressionEventArgs(int pourcentage, string message)
    {
        Pourcentage = pourcentage;
        Message = message;
    }
}

// Classe qui utilise les arguments d'événement personnalisés
public class TraitementFichier
{
    // Événement avec arguments personnalisés
    public event EventHandler<ProgressionEventArgs> ProgressionModifiée;

    public void TraiterFichier(string chemin)
    {
        // Simuler un traitement de fichier
        for (int i = 0; i <= 100; i += 10)
        {
            // Effectuer une étape du traitement
            System.Threading.Thread.Sleep(500); // Simuler un délai

            // Notifier les abonnés de la progression
            DéclencherProgressionModifiée(i, $"Traitement du fichier : {i}% terminé");
        }
    }

    protected virtual void DéclencherProgressionModifiée(int pourcentage, string message)
    {
        // Créer les arguments d'événement personnalisés
        var args = new ProgressionEventArgs(pourcentage, message);

        // Déclencher l'événement avec les arguments
        ProgressionModifiée?.Invoke(this, args);
    }
}

// Utilisation de l'événement avec arguments personnalisés
public void SuivreProgressionFichier()
{
    var traitementFichier = new TraitementFichier();

    // S'abonner à l'événement
    traitementFichier.ProgressionModifiée += (sender, e) => {
        Console.WriteLine($"{e.Message} - {e.Pourcentage}%");

        // Mettre à jour une barre de progression dans l'interface
        progressBar1.Value = e.Pourcentage;
        labelStatut.Text = e.Message;
    };

    // Démarrer le traitement
    traitementFichier.TraiterFichier("exemple.txt");
}
```


### Modèle de délégué Action et Func

En plus du modèle d'événement traditionnel, .NET propose les délégués génériques `Action` et `Func` qui peuvent être utilisés pour des callbacks plus simples.

```textmate
// .NET Framework 4.7.2 et .NET 8
public class ModèleCallback
{
    // Action prend des paramètres et ne retourne rien
    public void ExécuterAvecCallback(Action<string> callback)
    {
        // Effectuer une opération
        string résultat = "Opération terminée";

        // Appeler le callback avec le résultat
        callback(résultat);
    }

    // Func prend des paramètres et retourne une valeur
    public void ExécuterAvecRetour(Func<string, bool> validateur)
    {
        string donnée = "Exemple";

        // Utiliser la fonction de validation
        bool estValide = validateur(donnée);

        if (estValide)
        {
            Console.WriteLine("Données valides");
        }
        else
        {
            Console.WriteLine("Données invalides");
        }
    }
}

// Utilisation
public void UtiliserCallbacks()
{
    var modèle = new ModèleCallback();

    // Utiliser Action avec une lambda
    modèle.ExécuterAvecCallback(message => {
        Console.WriteLine($"Callback reçu : {message}");
    });

    // Utiliser Func avec une lambda
    modèle.ExécuterAvecRetour(donnée => {
        // Valider la donnée
        return !string.IsNullOrEmpty(donnée);
    });
}
```


### Délégués asynchrones

Avec la programmation asynchrone moderne en .NET, les événements peuvent être utilisés avec async/await.

```textmate
// .NET Framework 4.7.2 et .NET 8
public class TraitementAsyncDemo
{
    public event EventHandler<ProgressionEventArgs> ProgressionModifiée;
    public event EventHandler<bool> TraitementTerminé;

    public async Task TraiterAsync(string donnée)
    {
        try
        {
            // Notifier du début du traitement
            ProgressionModifiée?.Invoke(this, new ProgressionEventArgs(0, "Démarrage du traitement"));

            // Simuler un traitement asynchrone
            for (int i = 0; i <= 100; i += 20)
            {
                // Simuler une tâche asynchrone
                await Task.Delay(500);

                // Notifier de la progression
                ProgressionModifiée?.Invoke(this, new ProgressionEventArgs(i, $"Progression : {i}%"));
            }

            // Notifier de la fin réussie
            TraitementTerminé?.Invoke(this, true);
            return;
        }
        catch (Exception)
        {
            // Notifier de l'échec
            TraitementTerminé?.Invoke(this, false);
            throw;
        }
    }
}

// Utilisation avec async/await
public async Task DémontrerTraitementAsync()
{
    var demo = new TraitementAsyncDemo();

    // S'abonner aux événements
    demo.ProgressionModifiée += (sender, e) => {
        labelStatut.Text = e.Message;
        progressBar1.Value = e.Pourcentage;
    };

    demo.TraitementTerminé += (sender, succès) => {
        if (succès)
        {
            labelStatut.Text = "Traitement terminé avec succès";
        }
        else
        {
            labelStatut.Text = "Échec du traitement";
        }
    };

    // Démarrer le traitement asynchrone
    await demo.TraiterAsync("données");

    // Code exécuté après la fin du traitement
    MessageBox.Show("Traitement asynchrone terminé");
}
```


## 7.4.2. Événements communs (Click, Load, etc.)

Windows Forms propose de nombreux événements prédéfinis pour ses contrôles. Voici les événements les plus couramment utilisés :

### Événements de cycle de vie d'un formulaire

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // S'abonner aux événements du cycle de vie
        this.Load += Form1_Load;
        this.Shown += Form1_Shown;
        this.FormClosing += Form1_FormClosing;
        this.FormClosed += Form1_FormClosed;
    }

    private void Form1_Load(object sender, EventArgs e)
    {
        // Déclenché avant l'affichage initial du formulaire
        Console.WriteLine("Formulaire en cours de chargement");

        // Idéal pour initialiser les contrôles, charger les données, etc.
        ChargerDonnées();
    }

    private void Form1_Shown(object sender, EventArgs e)
    {
        // Déclenché après que le formulaire soit affiché à l'utilisateur
        Console.WriteLine("Formulaire affiché");

        // Idéal pour définir le focus, afficher les messages de bienvenue, etc.
        textBoxNom.Focus();
    }

    private void Form1_FormClosing(object sender, FormClosingEventArgs e)
    {
        // Déclenché avant que le formulaire ne se ferme
        Console.WriteLine($"Formulaire en cours de fermeture, raison : {e.CloseReason}");

        // Demander confirmation à l'utilisateur
        if (DonnéesModifiées)
        {
            DialogResult result = MessageBox.Show(
                "Voulez-vous enregistrer les modifications avant de quitter ?",
                "Confirmation",
                MessageBoxButtons.YesNoCancel,
                MessageBoxIcon.Question);

            if (result == DialogResult.Yes)
            {
                SauvegarderDonnées();
            }
            else if (result == DialogResult.Cancel)
            {
                // Annuler la fermeture
                e.Cancel = true;
            }
        }
    }

    private void Form1_FormClosed(object sender, FormClosedEventArgs e)
    {
        // Déclenché après la fermeture du formulaire
        Console.WriteLine("Formulaire fermé");

        // Libérer les ressources, effectuer le nettoyage final
        Dispose(true);
    }

    // Méthodes fictives pour l'exemple
    private bool DonnéesModifiées => true;
    private void ChargerDonnées() { }
    private void SauvegarderDonnées() { }
}
```


### Événements d'interface utilisateur

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // S'abonner aux événements des contrôles
        buttonValider.Click += ButtonValider_Click;
        textBoxNom.TextChanged += TextBoxNom_TextChanged;
        checkBoxAccepter.CheckedChanged += CheckBoxAccepter_CheckedChanged;
        comboBoxOptions.SelectedIndexChanged += ComboBoxOptions_SelectedIndexChanged;
        listBoxÉléments.SelectedIndexChanged += ListBoxÉléments_SelectedIndexChanged;
        numericUpDownQuantité.ValueChanged += NumericUpDownQuantité_ValueChanged;
        dateTimePickerDate.ValueChanged += DateTimePickerDate_ValueChanged;
        trackBarVolume.Scroll += TrackBarVolume_Scroll;
    }

    private void ButtonValider_Click(object sender, EventArgs e)
    {
        // Déclenché lorsque l'utilisateur clique sur le bouton
        MessageBox.Show("Formulaire validé");
    }

    private void TextBoxNom_TextChanged(object sender, EventArgs e)
    {
        // Déclenché à chaque modification du texte
        TextBox textBox = (TextBox)sender;
        labelAperçu.Text = textBox.Text;

        // Activer le bouton si le texte n'est pas vide
        buttonValider.Enabled = !string.IsNullOrWhiteSpace(textBox.Text);
    }

    private void CheckBoxAccepter_CheckedChanged(object sender, EventArgs e)
    {
        // Déclenché lorsque l'état de la case à cocher change
        CheckBox checkBox = (CheckBox)sender;
        buttonValider.Enabled = checkBox.Checked;
    }

    private void ComboBoxOptions_SelectedIndexChanged(object sender, EventArgs e)
    {
        // Déclenché lorsque la sélection change dans la liste déroulante
        ComboBox comboBox = (ComboBox)sender;
        labelOption.Text = $"Option sélectionnée : {comboBox.SelectedItem}";
    }

    private void ListBoxÉléments_SelectedIndexChanged(object sender, EventArgs e)
    {
        // Déclenché lorsque la sélection change dans la liste
        ListBox listBox = (ListBox)sender;

        if (listBox.SelectedIndex != -1)
        {
            labelÉlément.Text = $"Élément sélectionné : {listBox.SelectedItem}";
        }
        else
        {
            labelÉlément.Text = "Aucun élément sélectionné";
        }
    }

    private void NumericUpDownQuantité_ValueChanged(object sender, EventArgs e)
    {
        // Déclenché lorsque la valeur change
        NumericUpDown numericUpDown = (NumericUpDown)sender;
        labelQuantité.Text = $"Quantité : {numericUpDown.Value}";
    }

    private void DateTimePickerDate_ValueChanged(object sender, EventArgs e)
    {
        // Déclenché lorsque la date sélectionnée change
        DateTimePicker dateTimePicker = (DateTimePicker)sender;
        labelDate.Text = $"Date : {dateTimePicker.Value.ToShortDateString()}";
    }

    private void TrackBarVolume_Scroll(object sender, EventArgs e)
    {
        // Déclenché pendant le déplacement du curseur
        TrackBar trackBar = (TrackBar)sender;
        labelVolume.Text = $"Volume : {trackBar.Value}%";
    }
}
```


### Événements de validation

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Activer la validation automatique
        this.AutoValidate = AutoValidate.EnablePreventFocusChange;

        // S'abonner aux événements de validation
        textBoxEmail.Validating += TextBoxEmail_Validating;
        textBoxEmail.Validated += TextBoxEmail_Validated;

        // S'abonner à l'événement de validation du formulaire
        this.FormClosing += Form1_FormClosing;
    }

    private void TextBoxEmail_Validating(object sender, CancelEventArgs e)
    {
        // Déclenché lorsque le contrôle est sur le point de perdre le focus
        TextBox textBox = (TextBox)sender;
        string email = textBox.Text.Trim();

        // Valider l'adresse email avec une expression régulière
        string pattern = @"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
        bool estValide = System.Text.RegularExpressions.Regex.IsMatch(email, pattern);

        if (!estValide)
        {
            // Définir le message d'erreur
            errorProvider.SetError(textBox, "Veuillez entrer une adresse email valide");

            // Annuler le changement de focus
            e.Cancel = true;
        }
        else
        {
            // Effacer le message d'erreur
            errorProvider.SetError(textBox, "");
        }
    }

    private void TextBoxEmail_Validated(object sender, EventArgs e)
    {
        // Déclenché après une validation réussie
        TextBox textBox = (TextBox)sender;
        Console.WriteLine($"Email validé : {textBox.Text}");
    }

    private void Form1_FormClosing(object sender, FormClosingEventArgs e)
    {
        // Vérifier si tous les contrôles sont valides
        if (!ValidateChildren())
        {
            // Annuler la fermeture si la validation échoue
            DialogResult result = MessageBox.Show(
                "Le formulaire contient des erreurs. Voulez-vous quand même le fermer ?",
                "Validation",
                MessageBoxButtons.YesNo,
                MessageBoxIcon.Warning);

            if (result == DialogResult.No)
            {
                e.Cancel = true;
            }
        }
    }
}
```


### Événements de focus et navigation par tab

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // S'abonner aux événements de focus
        textBoxNom.Enter += Control_Enter;
        textBoxNom.Leave += Control_Leave;

        textBoxEmail.Enter += Control_Enter;
        textBoxEmail.Leave += Control_Leave;

        // S'abonner à l'événement de navigation par tab
        this.KeyDown += Form1_KeyDown;
    }

    private void Control_Enter(object sender, EventArgs e)
    {
        // Déclenché lorsque le contrôle reçoit le focus
        Control control = (Control)sender;

        // Mettre en évidence le contrôle actif
        if (control is TextBox textBox)
        {
            textBox.BackColor = Color.LightYellow;

            // Sélectionner tout le texte pour faciliter la modification
            textBox.SelectAll();
        }

        // Afficher une aide contextuelle
        if (control.Tag != null)
        {
            labelAide.Text = control.Tag.ToString();
        }
    }

    private void Control_Leave(object sender, EventArgs e)
    {
        // Déclenché lorsque le contrôle perd le focus
        Control control = (Control)sender;

        // Restaurer l'apparence normale
        if (control is TextBox textBox)
        {
            textBox.BackColor = SystemColors.Window;
        }

        // Effacer l'aide contextuelle
        labelAide.Text = "";
    }

    private void Form1_KeyDown(object sender, KeyEventArgs e)
    {
        // Gérer la navigation personnalisée
        if (e.KeyCode == Keys.Enter)
        {
            // Simuler un appui sur la touche Tab
            SendKeys.Send("{TAB}");

            // Marquer l'événement comme traité
            e.Handled = true;
            e.SuppressKeyPress = true; // Empêcher le "bip" de Windows
        }
    }
}
```


### Événements de glisser-déposer

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // Configurer le ListBox source comme source de drag-and-drop
        listBoxSource.MouseDown += ListBoxSource_MouseDown;

        // Configurer le ListBox cible comme cible de drag-and-drop
        listBoxCible.AllowDrop = true;
        listBoxCible.DragEnter += ListBoxCible_DragEnter;
        listBoxCible.DragDrop += ListBoxCible_DragDrop;

        // Configurer le PictureBox comme cible de drag-and-drop pour les fichiers
        pictureBoxImage.AllowDrop = true;
        pictureBoxImage.DragEnter += PictureBoxImage_DragEnter;
        pictureBoxImage.DragDrop += PictureBoxImage_DragDrop;
    }

    private void ListBoxSource_MouseDown(object sender, MouseEventArgs e)
    {
        // Démarrer l'opération de drag-and-drop
        ListBox listBox = (ListBox)sender;
        int index = listBox.IndexFromPoint(e.X, e.Y);

        if (index != ListBox.NoMatches)
        {
            // Obtenir l'élément sous le curseur
            string élément = listBox.Items[index].ToString();

            // Démarrer l'opération de drag-and-drop
            listBox.DoDragDrop(élément, DragDropEffects.Copy | DragDropEffects.Move);
        }
    }

    private void ListBoxCible_DragEnter(object sender, DragEventArgs e)
    {
        // Vérifier si les données sont au format attendu
        if (e.Data.GetDataPresent(DataFormats.StringFormat))
        {
            // Indiquer que l'opération est autorisée
            e.Effect = DragDropEffects.Copy;
        }
        else
        {
            // Refuser l'opération
            e.Effect = DragDropEffects.None;
        }
    }

    private void ListBoxCible_DragDrop(object sender, DragEventArgs e)
    {
        // Récupérer les données déposées
        if (e.Data.GetDataPresent(DataFormats.StringFormat))
        {
            string élément = (string)e.Data.GetData(DataFormats.StringFormat);

            // Ajouter l'élément à la liste cible
            ListBox listBox = (ListBox)sender;
            listBox.Items.Add(élément);
        }
    }

    private void PictureBoxImage_DragEnter(object sender, DragEventArgs e)
    {
        // Vérifier si les données sont des fichiers
        if (e.Data.GetDataPresent(DataFormats.FileDrop))
        {
            string[] fichiers = (string[])e.Data.GetData(DataFormats.FileDrop);

            // Vérifier si c'est une image
            if (fichiers.Length == 1 && EstFichierImage(fichiers[0]))
            {
                e.Effect = DragDropEffects.Copy;
            }
            else
            {
                e.Effect = DragDropEffects.None;
            }
        }
        else
        {
            e.Effect = DragDropEffects.None;
        }
    }

    private void PictureBoxImage_DragDrop(object sender, DragEventArgs e)
    {
        // Récupérer les fichiers déposés
        if (e.Data.GetDataPresent(DataFormats.FileDrop))
        {
            string[] fichiers = (string[])e.Data.GetData(DataFormats.FileDrop);

            if (fichiers.Length == 1 && EstFichierImage(fichiers[0]))
            {
                try
                {
                    // Charger l'image dans le PictureBox
                    PictureBox pictureBox = (PictureBox)sender;
                    pictureBox.Image = Image.FromFile(fichiers[0]);
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"Erreur lors du chargement de l'image : {ex.Message}", "Erreur", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }
        }
    }

    private bool EstFichierImage(string chemin)
    {
        // Vérifier l'extension du fichier
        string extension = Path.GetExtension(chemin).ToLower();
        return extension == ".jpg" || extension == ".jpeg" || extension == ".png" || extension == ".gif" || extension == ".bmp";
    }
}
```


## 7.4.3. Création d'événements personnalisés

Dans certains cas, vous devrez créer vos propres événements pour notifier d'autres parties de votre application des changements ou actions importantes.

### Création d'un événement simple

```textmate
// .NET Framework 4.7.2 et .NET 8
public class CompteurPersonnalisé : Control
{
    private int _valeur;
    private int _minimum = 0;
    private int _maximum = 100;

    // Déclarer l'événement pour la modification de la valeur
    public event EventHandler ValeurModifiée;

    // Propriété pour la valeur actuelle
    public int Valeur
    {
        get => _valeur;
        set
        {
            // Valider la nouvelle valeur
            int nouvelleValeur = Math.Max(_minimum, Math.Min(_maximum, value));

            // Vérifier si la valeur a réellement changé
            if (_valeur != nouvelleValeur)
            {
                _valeur = nouvelleValeur;

                // Déclencher l'événement
                DéclencherValeurModifiée();

                // Rafraîchir l'affichage
                Invalidate();
            }
        }
    }

    // Propriétés pour les valeurs minimale et maximale
    public int Minimum
    {
        get => _minimum;
        set
        {
            if (_minimum != value)
            {
                _minimum = value;

                // Ajuster la valeur actuelle si nécessaire
                if (_valeur < _minimum)
                {
                    Valeur = _minimum;
                }
            }
        }
    }

    public int Maximum
    {
        get => _maximum;
        set
        {
            if (_maximum != value)
            {
                _maximum = value;

                // Ajuster la valeur actuelle si nécessaire
                if (_valeur > _maximum)
                {
                    Valeur = _maximum;
                }
            }
        }
    }

    public CompteurPersonnalisé()
    {
        // Configurer le contrôle
        SetStyle(ControlStyles.UserPaint | ControlStyles.ResizeRedraw | ControlStyles.DoubleBuffer, true);
        Size = new Size(100, 30);

        // Initialiser la valeur
        _valeur = 0;
    }

    // Méthode protégée pour déclencher l'événement
    protected virtual void DéclencherValeurModifiée()
    {
        // Appeler les abonnés à l'événement
        ValeurModifiée?.Invoke(this, EventArgs.Empty);
    }

    // Gestion du dessin du contrôle
    protected override void OnPaint(PaintEventArgs e)
    {
        base.OnPaint(e);

        Graphics g = e.Graphics;

        // Dessiner le fond
        using (SolidBrush brush = new SolidBrush(BackColor))
        {
            g.FillRectangle(brush, ClientRectangle);
        }

        // Dessiner la bordure
        using (Pen pen = new Pen(Color.Gray))
        {
            g.DrawRectangle(pen, 0, 0, Width - 1, Height - 1);
        }

        // Calculer la largeur de la barre de progression
        float pourcentage = (_valeur - _minimum) / (float)(_maximum - _minimum);
        int largeurProgressBar = (int)(ClientRectangle.Width * pourcentage);

        // Dessiner la barre de progression
        using (SolidBrush brush = new SolidBrush(Color.DodgerBlue))
        {
            g.FillRectangle(brush, 0, 0, largeurProgressBar, Height);
        }

        // Dessiner le texte
        string texte = $"{_valeur}";
        using (StringFormat format = new StringFormat())
        {
            format.Alignment = StringAlignment.Center;
            format.LineAlignment = StringAlignment.Center;

            using (SolidBrush brush = new SolidBrush(ForeColor))
            {
                g.DrawString(texte, Font, brush, ClientRectangle, format);
            }
        }
    }

    // Gestion des clics de souris
    protected override void OnMouseClick(MouseEventArgs e)
    {
        base.OnMouseClick(e);

        // Calculer la nouvelle valeur en fonction de la position du clic
        float pourcentage = (float)e.X / Width;
        Valeur = _minimum + (int)((_maximum - _minimum) * pourcentage);
    }
}
```


### Événement avec arguments personnalisés

```textmate
// .NET Framework 4.7.2 et .NET 8
// Classe d'arguments d'événement personnalisée
public class ValeurChangéeEventArgs : EventArgs
{
    public int AncienneValeur { get; }
    public int NouvelleValeur { get; }
    public bool AnnulerChangement { get; set; }

    public ValeurChangéeEventArgs(int ancienneValeur, int nouvelleValeur)
    {
        AncienneValeur = ancienneValeur;
        NouvelleValeur = nouvelleValeur;
        AnnulerChangement = false;
    }
}

// Contrôle avec événement personnalisé qui permet l'annulation
public class CompteurAvancé : Control
{
    private int _valeur;

    // Déclaration d'événements avec arguments personnalisés
    public event EventHandler<ValeurChangéeEventArgs> ValeurChangée;
    public event EventHandler<ValeurChangéeEventArgs> ValeurChangement;

    // Propriété pour la valeur
    public int Valeur
    {
        get => _valeur;
        set
        {
            if (_valeur != value)
            {
                // Créer les arguments d'événement
                ValeurChangéeEventArgs args = new ValeurChangéeEventArgs(_valeur, value);

                // Déclencher l'événement avant le changement (permet l'annulation)
                DéclencherValeurChangement(args);

                // Vérifier si le changement a été annulé
                if (!args.AnnulerChangement)
                {
                    // Appliquer le changement
                    int ancienneValeur = _valeur;
                    _valeur = value;

                    // Déclencher l'événement après le changement
                    DéclencherValeurChangée(new ValeurChangéeEventArgs(ancienneValeur, _valeur));

                    // Rafraîchir l'affichage
                    Invalidate();
                }
            }
        }
    }

    // Méthode pour déclencher l'événement avant le changement
    protected virtual void DéclencherValeurChangement(ValeurChangéeEventArgs args)
    {
        ValeurChangement?.Invoke(this, args);
    }

    // Méthode pour déclencher l'événement après le changement
    protected virtual void DéclencherValeurChangée(ValeurChangéeEventArgs args)
    {
        ValeurChangée?.Invoke(this, args);
    }

    // Reste de l'implémentation du contrôle (dessin, gestion de la souris, etc.)
    // ...
}

// Utilisation des événements personnalisés
public partial class Form1 : Form
{
    private CompteurAvancé compteur;

    public Form1()
    {
        InitializeComponent();

        // Créer et configurer le compteur
        compteur = new CompteurAvancé
        {
            Location = new Point(50, 50),
            Size = new Size(200, 30)
        };

        // S'abonner à l'événement avant changement (avec possibilité d'annuler)
        compteur.ValeurChangement += (sender, e) =>
        {
            // Vérifier si la nouvelle valeur est négative
            if (e.NouvelleValeur < 0)
            {
                MessageBox.Show("Les valeurs négatives ne sont pas autorisées", "Erreur", MessageBoxButtons.OK, MessageBoxIcon.Error);

                // Annuler le changement
                e.AnnulerChangement = true;
            }

            // Demander confirmation pour les grandes valeurs
            if (e.NouvelleValeur > 100)
            {
                DialogResult result = MessageBox.Show(
                    $"Êtes-vous sûr de vouloir définir une valeur supérieure à 100 ? ({e.NouvelleValeur})",
                    "Confirmation",
                    MessageBoxButtons.YesNo,
                    MessageBoxIcon.Question);

                if (result == DialogResult.No)
                {
                    // Annuler le changement
                    e.AnnulerChangement = true;
                }
            }
        };

        // S'abonner à l'événement après changement
        compteur.ValeurChangée += (sender, e) =>
        {
            labelValeur.Text = $"Valeur : {e.NouvelleValeur} (précédemment {e.AncienneValeur})";

            // Journaliser le changement
            Console.WriteLine($"Valeur modifiée : {e.AncienneValeur} -> {e.NouvelleValeur}");
        };

        // Ajouter le contrôle au formulaire
        this.Controls.Add(compteur);

        // Ajouter des boutons pour manipuler la valeur
        Button buttonAugmenter = new Button
        {
            Text = "+10",
            Location = new Point(50, 100),
            Size = new Size(50, 30)
        };

        buttonAugmenter.Click += (sender, e) =>
        {
            compteur.Valeur += 10;
        };

        Button buttonDiminuer = new Button
        {
            Text = "-10",
            Location = new Point(110, 100),
            Size = new Size(50, 30)
        };

        buttonDiminuer.Click += (sender, e) =>
        {
            compteur.Valeur -= 10;
        };

        this.Controls.Add(buttonAugmenter);
        this.Controls.Add(buttonDiminuer);
    }
}
```


### Événements avec pattern d'observateur

Le pattern d'observateur est un modèle de conception qui permet à un objet (sujet) de notifier d'autres objets (observateurs) des changements d'état.

```textmate
// .NET Framework 4.7.2 et .NET 8
// Interface pour l'observateur
public interface IObservateur<T>
{
    void Notifier(string propriété, T ancienneValeur, T nouvelleValeur);
}

// Classe observable
public class ProduitObservable
{
    private string _nom;
    private decimal _prix;
    private int _quantité;

    // Liste des observateurs
    private List<IObservateur<object>> _observateurs = new List<IObservateur<object>>();

    // Propriétés avec notification
    public string Nom
    {
        get => _nom;
        set
        {
            if (_nom != value)
            {
                string ancienneValeur = _nom;
                _nom = value;
                NotifierObservateurs("Nom", ancienneValeur, value);
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
                decimal ancienneValeur = _prix;
                _prix = value;
                NotifierObservateurs("Prix", ancienneValeur, value);
            }
        }
    }

    public int Quantité
    {
        get => _quantité;
        set
        {
            if (_quantité != value)
            {
                int ancienneValeur = _quantité;
                _quantité = value;
                NotifierObservateurs("Quantité", ancienneValeur, value);
            }
        }
    }

    // Méthodes pour gérer les observateurs
    public void AjouterObservateur(IObservateur<object> observateur)
    {
        if (!_observateurs.Contains(observateur))
        {
            _observateurs.Add(observateur);
        }
    }

    public void SupprimerObservateur(IObservateur<object> observateur)
    {
        _observateurs.Remove(observateur);
    }

    // Méthode pour notifier les observateurs
    private void NotifierObservateurs(string propriété, object ancienneValeur, object nouvelleValeur)
    {
        foreach (var observateur in _observateurs)
        {
            observateur.Notifier(propriété, ancienneValeur, nouvelleValeur);
        }
    }
}

// Implémentation d'un observateur qui met à jour l'interface utilisateur
public class ObservateurUI : IObservateur<object>
{
    private Label _labelNom;
    private Label _labelPrix;
    private Label _labelQuantité;

    public ObservateurUI(Label labelNom, Label labelPrix, Label labelQuantité)
    {
        _labelNom = labelNom;
        _labelPrix = labelPrix;
        _labelQuantité = labelQuantité;
    }

    public void Notifier(string propriété, object ancienneValeur, object nouvelleValeur)
    {
        // Mettre à jour l'interface utilisateur selon la propriété modifiée
        switch (propriété)
        {
            case "Nom":
                _labelNom.Text = $"Nom : {nouvelleValeur}";
                break;

            case "Prix":
                _labelPrix.Text = $"Prix : {nouvelleValeur:C}";
                break;

            case "Quantité":
                _labelQuantité.Text = $"Quantité : {nouvelleValeur}";
                break;
        }
    }
}

// Utilisation du pattern d'observateur
public partial class Form1 : Form
{
    private ProduitObservable _produit;

    public Form1()
    {
        InitializeComponent();

        // Créer le produit observable
        _produit = new ProduitObservable
        {
            Nom = "Produit exemple",
            Prix = 19.99m,
            Quantité = 10
        };

        // Créer l'observateur
        ObservateurUI observateur = new ObservateurUI(labelNom, labelPrix, labelQuantité);

        // Ajouter l'observateur au produit
        _produit.AjouterObservateur(observateur);

        // Configurer les contrôles pour modifier le produit
        textBoxNom.TextChanged += (sender, e) =>
        {
            _produit.Nom = textBoxNom.Text;
        };

        numericUpDownPrix.ValueChanged += (sender, e) =>
        {
            _produit.Prix = numericUpDownPrix.Value;
        };

        numericUpDownQuantité.ValueChanged += (sender, e) =>
        {
            _produit.Quantité = (int)numericUpDownQuantité.Value;
        };

        // Initialiser les contrôles
        textBoxNom.Text = _produit.Nom;
        numericUpDownPrix.Value = _produit.Prix;
        numericUpDownQuantité.Value = _produit.Quantité;
    }
}
```


## 7.4.4. Gestion des événements de clavier et souris

Les événements clavier et souris sont essentiels pour créer des interfaces utilisateur interactives. Windows Forms propose un ensemble complet d'événements pour gérer ces interactions.

### Événements clavier

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // S'abonner aux événements clavier du formulaire
        this.KeyDown += Form1_KeyDown;
        this.KeyUp += Form1_KeyUp;
        this.KeyPress += Form1_KeyPress;

        // Activer les événements clavier pour le formulaire
        this.KeyPreview = true;

        // S'abonner aux événements clavier d'un contrôle spécifique
        textBoxSaisie.KeyDown += TextBoxSaisie_KeyDown;
        textBoxSaisie.KeyUp += TextBoxSaisie_KeyUp;
        textBoxSaisie.KeyPress += TextBoxSaisie_KeyPress;
    }

    // Événements clavier au niveau du formulaire

    private void Form1_KeyDown(object sender, KeyEventArgs e)
    {
        // Déclenché lorsqu'une touche est enfoncée
        // e.KeyCode contient le code de la touche
        // e.Modifiers contient les modificateurs (Ctrl, Alt, Shift)

        // Afficher les informations de la touche
        labelTouche.Text = $"Touche enfoncée : {e.KeyCode} (Code : {(int)e.KeyCode})";

        // Vérifier les modificateurs
        string modificateurs = "";
        if (e.Control) modificateurs += "Ctrl + ";
        if (e.Alt) modificateurs += "Alt + ";
        if (e.Shift) modificateurs += "Shift + ";

        if (!string.IsNullOrEmpty(modificateurs))
        {
            labelModificateurs.Text = $"Modificateurs : {modificateurs.TrimEnd(' ', '+')}";
        }
        else
        {
            labelModificateurs.Text = "Modificateurs : Aucun";
        }

        // Gérer les raccourcis clavier
        if (e.Control && e.KeyCode == Keys.S)
        {
            // Ctrl+S : Sauvegarder
            MessageBox.Show("Sauvegarde en cours...", "Raccourci clavier", MessageBoxButtons.OK, MessageBoxIcon.Information);

            // Marquer l'événement comme traité
            e.Handled = true;
        }
        else if (e.Control && e.KeyCode == Keys.O)
        {
            // Ctrl+O : Ouvrir
            MessageBox.Show("Ouverture d'un fichier...", "Raccourci clavier", MessageBoxButtons.OK, MessageBoxIcon.Information);

            // Marquer l'événement comme traité
            e.Handled = true;
        }
        else if (e.Alt && e.KeyCode == Keys.F4)
        {
            // Alt+F4 : Fermer avec confirmation
            DialogResult result = MessageBox.Show("Voulez-vous vraiment fermer l'application ?", "Fermeture", MessageBoxButtons.YesNo, MessageBoxIcon.Question);

            if (result == DialogResult.No)
            {
                // Annuler la fermeture
                e.Handled = true;
            }
        }

        // Navigation avec les flèches
        if (e.KeyCode == Keys.Left || e.KeyCode == Keys.Right || e.KeyCode == Keys.Up || e.KeyCode == Keys.Down)
        {
            labelNavigation.Text = $"Navigation : Flèche {e.KeyCode}";
        }
    }

    private void Form1_KeyUp(object sender, KeyEventArgs e)
    {
        // Déclenché lorsqu'une touche est relâchée
        labelRelachement.Text = $"Touche relâchée : {e.KeyCode}";

        // Réinitialiser les informations après un délai
        Timer timer = new Timer();
        timer.Interval = 1000;
        timer.Tick += (s, args) =>
        {
            labelRelachement.Text = "";
            timer.Stop();
            timer.Dispose();
        };
        timer.Start();
    }

    private void Form1_KeyPress(object sender, KeyPressEventArgs e)
    {
        // Déclenché pour les touches qui produisent un caractère
        // e.KeyChar contient le caractère correspondant à la touche

        labelCaractere.Text = $"Caractère : '{e.KeyChar}' (Code ASCII : {(int)e.KeyChar})";

        // Intercepter certains caractères
        if (e.KeyChar == '@')
        {
            MessageBox.Show("Caractère @ détecté !", "KeyPress", MessageBoxButtons.OK, MessageBoxIcon.Information);

            // Marquer l'événement comme traité
            e.Handled = true;
        }
    }

    // Événements clavier au niveau d'un contrôle spécifique

    private void TextBoxSaisie_KeyDown(object sender, KeyEventArgs e)
    {
        // Gérer les touches spéciales dans le TextBox
        if (e.KeyCode == Keys.Enter)
        {
            // Valider la saisie lorsque l'utilisateur appuie sur Entrée
            TextBox textBox = (TextBox)sender;
            labelSaisie.Text = $"Saisie validée : {textBox.Text}";

            // Effacer le TextBox
            textBox.Clear();

            // Empêcher le "bip" de Windows
            e.SuppressKeyPress = true;
            e.Handled = true;
        }
        else if (e.KeyCode == Keys.Escape)
        {
            // Annuler la saisie lorsque l'utilisateur appuie sur Échap
            TextBox textBox = (TextBox)sender;
            textBox.Clear();

            // Empêcher le "bip" de Windows
            e.SuppressKeyPress = true;
            e.Handled = true;
        }
    }

    private void TextBoxSaisie_KeyUp(object sender, KeyEventArgs e)
    {
        // Mise à jour de l'interface après la saisie
        TextBox textBox = (TextBox)sender;
        labelCaractèresRestants.Text = $"Caractères restants : {100 - textBox.Text.Length}";
    }

    private void TextBoxSaisie_KeyPress(object sender, KeyPressEventArgs e)
    {
        // Filtrer les caractères saisis
        TextBox textBox = (TextBox)sender;

        // Exemple : autoriser uniquement les chiffres
        if (checkBoxChiffresUniquement.Checked)
        {
            // Autoriser les chiffres et la touche Backspace (code 8)
            if (!char.IsDigit(e.KeyChar) && e.KeyChar != 8)
            {
                // Ignorer le caractère
                e.Handled = true;
            }
        }

        // Exemple : limiter la longueur
        if (textBox.Text.Length >= 100 && e.KeyChar != 8)
        {
            // Ignorer le caractère si la limite est atteinte
            e.Handled = true;
        }

        // Exemple : convertir automatiquement en majuscules
        if (checkBoxMajuscules.Checked && char.IsLetter(e.KeyChar))
        {
            // Convertir en majuscule
            e.KeyChar = char.ToUpper(e.KeyChar);
        }
    }
}
```


### Événements souris

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    // Variables pour suivre l'état de la souris
    private bool _glisseEnCours = false;
    private Point _pointDébutGlisse;
    private Point _positionInitialeContôle;
    private Control _contrôleGlissé = null;

    // Variables pour le dessin interactif
    private List<Point> _points = new List<Point>();
    private bool _dessinEnCours = false;

    public Form1()
    {
        InitializeComponent();

        // S'abonner aux événements souris du formulaire
        this.MouseMove += Form1_MouseMove;
        this.MouseEnter += Form1_MouseEnter;
        this.MouseLeave += Form1_MouseLeave;

        // S'abonner aux événements souris du panel de dessin
        panelDessin.Paint += PanelDessin_Paint;
        panelDessin.MouseDown += PanelDessin_MouseDown;
        panelDessin.MouseMove += PanelDessin_MouseMove;
        panelDessin.MouseUp += PanelDessin_MouseUp;

        // S'abonner aux événements souris du contrôle glissable
        labelGlissable.MouseDown += ControlGlissable_MouseDown;
        labelGlissable.MouseMove += ControlGlissable_MouseMove;
        labelGlissable.MouseUp += ControlGlissable_MouseUp;

        // S'abonner aux événements souris du bouton
        buttonTest.MouseEnter += ButtonTest_MouseEnter;
        buttonTest.MouseLeave += ButtonTest_MouseLeave;
        buttonTest.MouseDown += ButtonTest_MouseDown;
        buttonTest.MouseUp += ButtonTest_MouseUp;
        buttonTest.Click += ButtonTest_Click;
        buttonTest.DoubleClick += ButtonTest_DoubleClick;
    }

    // Événements souris au niveau du formulaire

    private void Form1_MouseMove(object sender, MouseEventArgs e)
    {
        // Afficher la position de la souris
        labelPosition.Text = $"Position : X={e.X}, Y={e.Y}";

        // Afficher le type de bouton
        string boutons = "";
        if (e.Button == MouseButtons.Left) boutons += "Gauche ";
        if (e.Button == MouseButtons.Right) boutons += "Droit ";
        if (e.Button == MouseButtons.Middle) boutons += "Milieu ";

        if (string.IsNullOrEmpty(boutons))
        {
            labelBoutons.Text = "Boutons : Aucun";
        }
        else
        {
            labelBoutons.Text = $"Boutons : {boutons.Trim()}";
        }
    }

    private void Form1_MouseEnter(object sender, EventArgs e)
    {
        // La souris entre dans la zone du formulaire
        labelStatut.Text = "Statut : Souris dans le formulaire";
    }

    private void Form1_MouseLeave(object sender, EventArgs e)
    {
        // La souris quitte la zone du formulaire
        labelStatut.Text = "Statut : Souris hors du formulaire";

        // Réinitialiser les informations de position
        labelPosition.Text = "Position : -";
        labelBoutons.Text = "Boutons : -";
    }

    // Événements souris pour le panel de dessin

    private void PanelDessin_Paint(object sender, PaintEventArgs e)
    {
        // Dessiner les points enregistrés
        if (_points.Count > 1)
        {
            using (Pen pen = new Pen(Color.Blue, 2))
            {
                e.Graphics.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;
                e.Graphics.DrawLines(pen, _points.ToArray());
            }
        }
    }

    private void PanelDessin_MouseDown(object sender, MouseEventArgs e)
    {
        if (e.Button == MouseButtons.Left)
        {
            // Démarrer un nouveau dessin
            _dessinEnCours = true;
            _points.Clear();
            _points.Add(e.Location);

            labelDessin.Text = "Dessin : en cours";
        }
    }

    private void PanelDessin_MouseMove(object sender, MouseEventArgs e)
    {
        if (_dessinEnCours && e.Button == MouseButtons.Left)
        {
            // Ajouter le point au dessin
            _points.Add(e.Location);

            // Rafraîchir le panel pour redessiner
            panelDessin.Invalidate();
        }
    }

    private void PanelDessin_MouseUp(object sender, MouseEventArgs e)
    {
        if (e.Button == MouseButtons.Left)
        {
            // Terminer le dessin
            _dessinEnCours = false;

            // Ajouter le dernier point
            _points.Add(e.Location);

            // Rafraîchir le panel
            panelDessin.Invalidate();

            labelDessin.Text = $"Dessin : terminé ({_points.Count} points)";
        }
    }

    // Événements souris pour le contrôle glissable

    private void ControlGlissable_MouseDown(object sender, MouseEventArgs e)
    {
        if (e.Button == MouseButtons.Left)
        {
            // Démarrer le glissement
            _glisseEnCours = true;
            _contrôleGlissé = (Control)sender;

            // Enregistrer la position initiale
            _pointDébutGlisse = e.Location;
            _positionInitialeContôle = _contrôleGlissé.Location;

            // Changer le curseur
            _contrôleGlissé.Cursor = Cursors.SizeAll;

            labelGlissable.BackColor = Color.LightBlue;
        }
    }

    private void ControlGlissable_MouseMove(object sender, MouseEventArgs e)
    {
        if (_glisseEnCours)
        {
            // Calculer le déplacement
            int deltaX = e.X - _pointDébutGlisse.X;
            int deltaY = e.Y - _pointDébutGlisse.Y;

            // Déplacer le contrôle
            _contrôleGlissé.Location = new Point(
                _positionInitialeContôle.X + deltaX,
                _positionInitialeContôle.Y + deltaY);

            // Afficher les informations de position
            labelGlissableInfo.Text = $"Position : X={_contrôleGlissé.Location.X}, Y={_contrôleGlissé.Location.Y}";
        }
    }

    private void ControlGlissable_MouseUp(object sender, MouseEventArgs e)
    {
        if (e.Button == MouseButtons.Left && _glisseEnCours)
        {
            // Terminer le glissement
            _glisseEnCours = false;

            // Restaurer le curseur
            _contrôleGlissé.Cursor = Cursors.Default;

            labelGlissable.BackColor = Color.White;

            // Afficher le déplacement
            int deltaX = _contrôleGlissé.Location.X - _positionInitialeContôle.X;
            int deltaY = _contrôleGlissé.Location.Y - _positionInitialeContôle.Y;

            labelGlissableInfo.Text += $" (déplacé de {deltaX}, {deltaY})";
        }
    }

    // Événements souris pour le bouton

    private void ButtonTest_MouseEnter(object sender, EventArgs e)
    {
        // Mettre en évidence le bouton
        Button button = (Button)sender;
        button.BackColor = Color.LightYellow;

        labelBoutonInfo.Text = "Bouton : Survol";
    }

    private void ButtonTest_MouseLeave(object sender, EventArgs e)
    {
        // Restaurer l'apparence du bouton
        Button button = (Button)sender;
        button.BackColor = SystemColors.Control;

        labelBoutonInfo.Text = "Bouton : Normal";
    }

    private void ButtonTest_MouseDown(object sender, MouseEventArgs e)
    {
        // Effet visuel pour le bouton enfoncé
        Button button = (Button)sender;
        button.BackColor = Color.LightBlue;

        labelBoutonInfo.Text = "Bouton : Enfoncé";
    }

    private void ButtonTest_MouseUp(object sender, MouseEventArgs e)
    {
        // Restaurer l'effet de survol
        Button button = (Button)sender;
        button.BackColor = Color.LightYellow;

        labelBoutonInfo.Text = "Bouton : Relâché";
    }

    private void ButtonTest_Click(object sender, EventArgs e)
    {
        // Gérer le clic sur le bouton
        labelBoutonInfo.Text = "Bouton : Cliqué";

        // Afficher un message
        int nombreClics = int.Parse(labelCompteurClics.Text.Split(':')[1].Trim()) + 1;
        labelCompteurClics.Text = $"Nombre de clics : {nombreClics}";
    }

    private void ButtonTest_DoubleClick(object sender, EventArgs e)
    {
        // Gérer le double-clic sur le bouton
        labelBoutonInfo.Text = "Bouton : Double-cliqué";

        // Réinitialiser le compteur
        labelCompteurClics.Text = "Nombre de clics : 0";
    }
}
```


### Événements de la molette de la souris

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    private int _valeurZoom = 100;
    private float _échelle = 1.0f;

    public Form1()
    {
        InitializeComponent();

        // S'abonner à l'événement de la molette
        this.MouseWheel += Form1_MouseWheel;

        // Configurer le PictureBox pour l'exemple de zoom
        pictureBoxImage.SizeMode = PictureBoxSizeMode.Zoom;
        try
        {
            pictureBoxImage.Image = Image.FromFile("exemple.jpg");
        }
        catch
        {
            // Créer une image par défaut si le fichier n'existe pas
            Bitmap bitmap = new Bitmap(300, 200);
            using (Graphics g = Graphics.FromImage(bitmap))
            {
                g.Clear(Color.LightGray);
                g.DrawString("Image d'exemple", new Font("Arial", 12), Brushes.Black, 80, 90);
            }
            pictureBoxImage.Image = bitmap;
        }
    }

    private void Form1_MouseWheel(object sender, MouseEventArgs e)
    {
        // e.Delta > 0 : molette vers le haut
        // e.Delta < 0 : molette vers le bas

        // Afficher les informations de la molette
        labelMolette.Text = $"Molette : Delta = {e.Delta}";

        // Déterminer le contrôle sous le curseur
        Control contrôle = GetChildAtPoint(e.Location);

        if (contrôle == pictureBoxImage)
        {
            // Zoom sur l'image
            if (e.Delta > 0)
            {
                // Zoom avant
                _valeurZoom += 10;
                if (_valeurZoom > 200) _valeurZoom = 200;
            }
            else
            {
                // Zoom arrière
                _valeurZoom -= 10;
                if (_valeurZoom < 50) _valeurZoom = 50;
            }

            // Mettre à jour l'échelle
            _échelle = _valeurZoom / 100f;

            // Mettre à jour l'affichage
            labelZoom.Text = $"Zoom : {_valeurZoom}%";

            // Redimensionner l'image
            if (pictureBoxImage.SizeMode == PictureBoxSizeMode.AutoSize)
            {
                // L'exemple suivant fonctionne avec SizeMode.AutoSize
                // Créer une nouvelle image redimensionnée
                if (pictureBoxImage.Image != null)
                {
                    Image imageOriginale = pictureBoxImage.Tag as Image ?? pictureBoxImage.Image;

                    // Enregistrer l'image originale la première fois
                    if (pictureBoxImage.Tag == null)
                    {
                        pictureBoxImage.Tag = pictureBoxImage.Image;
                    }

                    // Calculer les nouvelles dimensions
                    int largeur = (int)(imageOriginale.Width * _échelle);
                    int hauteur = (int)(imageOriginale.Height * _échelle);

                    // Créer la nouvelle image redimensionnée
                    Bitmap nouvelle = new Bitmap(largeur, hauteur);
                    using (Graphics g = Graphics.FromImage(nouvelle))
                    {
                        g.InterpolationMode = System.Drawing.Drawing2D.InterpolationMode.HighQualityBicubic;
                        g.DrawImage(imageOriginale, 0, 0, largeur, hauteur);
                    }

                    // Mettre à jour l'image
                    pictureBoxImage.Image = nouvelle;
                }
            }
            else
            {
                // Pour les autres modes (Zoom, StretchImage), le zoom se fait automatiquement
                // On peut simplement ajuster la taille du PictureBox
                pictureBoxImage.Size = new Size(
                    (int)(pictureBoxImage.Width * (_échelle / (_échelle - (e.Delta > 0 ? 0.1f : -0.1f)))),
                    (int)(pictureBoxImage.Height * (_échelle / (_échelle - (e.Delta > 0 ? 0.1f : -0.1f))))
                );
            }
        }
        else if (contrôle is ListBox listBox)
        {
            // Faire défiler la ListBox
            int visibleItems = listBox.ClientSize.Height / listBox.ItemHeight;
            int scrollAmount = e.Delta > 0 ? -1 : 1;

            // Ajuster la ligne du haut
            int topIndex = listBox.TopIndex;
            topIndex += scrollAmount;

            // Vérifier les limites
            if (topIndex < 0) topIndex = 0;
            if (topIndex > listBox.Items.Count - visibleItems)
                topIndex = Math.Max(0, listBox.Items.Count - visibleItems);

            listBox.TopIndex = topIndex;
        }
        else if (contrôle is NumericUpDown numericUpDown)
        {
            // Incrémenter/décrémenter la valeur
            if (e.Delta > 0)
            {
                // Augmenter la valeur
                if (numericUpDown.Value < numericUpDown.Maximum)
                {
                    numericUpDown.Value += numericUpDown.Increment;
                }
            }
            else
            {
                // Diminuer la valeur
                if (numericUpDown.Value > numericUpDown.Minimum)
                {
                    numericUpDown.Value -= numericUpDown.Increment;
                }
            }
        }
        else if (contrôle is TrackBar trackBar)
        {
            // Ajuster la position du curseur
            if (e.Delta > 0)
            {
                // Augmenter la valeur
                if (trackBar.Value < trackBar.Maximum)
                {
                    trackBar.Value = Math.Min(trackBar.Maximum, trackBar.Value + trackBar.SmallChange);
                }
            }
            else
            {
                // Diminuer la valeur
                if (trackBar.Value > trackBar.Minimum)
                {
                    trackBar.Value = Math.Max(trackBar.Minimum, trackBar.Value - trackBar.SmallChange);
                }
            }
        }
    }
}
```


### Événements de glisser-déposer avancés

Le glisser-déposer est une fonctionnalité puissante qui permet aux utilisateurs de déplacer des données entre différentes parties de l'application ou même entre applications.

```textmate
// .NET Framework 4.7.2 et .NET 8
public partial class Form1 : Form
{
    // Format personnalisé pour le glisser-déposer
    private readonly string FormatPersonnalisé = "MonApplication.Élément";

    public Form1()
    {
        InitializeComponent();

        // Configurer le ListView source comme source de drag-and-drop
        listViewSource.View = View.Details;
        listViewSource.Columns.Add("ID", 50);
        listViewSource.Columns.Add("Nom", 150);
        listViewSource.Columns.Add("Description", 200);
        listViewSource.FullRowSelect = true;
        listViewSource.MultiSelect = true;

        // Ajouter des éléments au ListView source
        ListViewItem item1 = new ListViewItem("1");
        item1.SubItems.Add("Élément 1");
        item1.SubItems.Add("Description de l'élément 1");
        item1.Tag = new { ID = 1, Nom = "Élément 1", Description = "Description de l'élément 1" };

        ListViewItem item2 = new ListViewItem("2");
        item2.SubItems.Add("Élément 2");
        item2.SubItems.Add("Description de l'élément 2");
        item2.Tag = new { ID = 2, Nom = "Élément 2", Description = "Description de l'élément 2" };

        ListViewItem item3 = new ListViewItem("3");
        item3.SubItems.Add("Élément 3");
        item3.SubItems.Add("Description de l'élément 3");
        item3.Tag = new { ID = 3, Nom = "Élément 3", Description = "Description de l'élément 3" };

        listViewSource.Items.AddRange(new ListViewItem[] { item1, item2, item3 });

        // Configurer le ListView cible comme destination de drag-and-drop
        listViewCible.View = View.Details;
        listViewCible.Columns.Add("ID", 50);
        listViewCible.Columns.Add("Nom", 150);
        listViewCible.Columns.Add("Description", 200);
        listViewCible.FullRowSelect = true;
        listViewCible.AllowDrop = true;

        // S'abonner aux événements de drag-and-drop du ListView source
        listViewSource.ItemDrag += ListViewSource_ItemDrag;

        // S'abonner aux événements de drag-and-drop du ListView cible
        listViewCible.DragEnter += ListViewCible_DragEnter;
        listViewCible.DragOver += ListViewCible_DragOver;
        listViewCible.DragDrop += ListViewCible_DragDrop;

        // Configurer le Panel comme zone de suppression
        panelCorbeille.AllowDrop = true;
        panelCorbeille.BackColor = Color.LightPink;
        panelCorbeille.BorderStyle = BorderStyle.FixedSingle;

        // S'abonner aux événements du Panel
        panelCorbeille.DragEnter += PanelCorbeille_DragEnter;
        panelCorbeille.DragLeave += PanelCorbeille_DragLeave;
        panelCorbeille.DragDrop += PanelCorbeille_DragDrop;

        // Configurer la zone de texte pour accepter des fichiers
        textBoxFichier.AllowDrop = true;
        textBoxFichier.DragEnter += TextBoxFichier_DragEnter;
        textBoxFichier.DragDrop += TextBoxFichier_DragDrop;
    }

    // Événements du ListView source

    private void ListViewSource_ItemDrag(object sender, ItemDragEventArgs e)
    {
        // Démarrer l'opération de drag-and-drop
        ListView listView = (ListView)sender;

        // Sélectionner l'élément sous le curseur s'il n'est pas déjà sélectionné
        if (e.Item is ListViewItem item && !item.Selected)
        {
            // Désélectionner tous les éléments
            foreach (ListViewItem i in listView.Items)
            {
                i.Selected = false;
            }

            // Sélectionner l'élément sous le curseur
            item.Selected = true;
        }

        // Préparer les données pour le glisser-déposer
        List<ListViewItem> élémentsSélectionnés = new List<ListViewItem>();
        foreach (ListViewItem item in listView.SelectedItems)
        {
            élémentsSélectionnés.Add(item);
        }

        // S'il y a des éléments sélectionnés
        if (élémentsSélectionnés.Count > 0)
        {
            // Créer l'objet DataObject
            DataObject data = new DataObject();

            // Ajouter les éléments au format personnalisé
            data.SetData(FormatPersonnalisé, élémentsSélectionnés);

            // Ajouter également un format texte pour la compatibilité avec d'autres applications
            StringBuilder texte = new StringBuilder();
            foreach (ListViewItem item in élémentsSélectionnés)
            {
                texte.AppendLine($"{item.Text}\t{item.SubItems[1].Text}\t{item.SubItems[2].Text}");
            }
            data.SetData(DataFormats.Text, texte.ToString());

            // Démarrer l'opération de drag-and-drop
            listView.DoDragDrop(data, DragDropEffects.Copy | DragDropEffects.Move);
        }
    }

    // Événements du ListView cible

    private void ListViewCible_DragEnter(object sender, DragEventArgs e)
    {
        // Vérifier si les données sont dans un format accepté
        if (e.Data.GetDataPresent(FormatPersonnalisé) || e.Data.GetDataPresent(DataFormats.Text))
        {
            // Vérifier si l'utilisateur maintient la touche Ctrl (copie) ou non (déplacement)
            if ((e.KeyState & 8) == 8) // Ctrl est enfoncé
            {
                e.Effect = DragDropEffects.Copy;
            }
            else
            {
                e.Effect = DragDropEffects.Move;
            }
        }
        else
        {
            e.Effect = DragDropEffects.None;
        }
    }

    private void ListViewCible_DragOver(object sender, DragEventArgs e)
    {
        // Mettre en évidence l'élément sous le curseur
        ListView listView = (ListView)sender;

        // Convertir les coordonnées d'écran en coordonnées de client
        Point clientPoint = listView.PointToClient(new Point(e.X, e.Y));

        // Trouver l'élément sous le curseur
        ListViewItem itemSousCurseur = listView.GetItemAt(clientPoint.X, clientPoint.Y);

        // Mettre à jour l'interface
        if (itemSousCurseur != null)
        {
            // Mettre en évidence l'élément
            labelDragInfo.Text = $"Au-dessus de : {itemSousCurseur.Text}";
        }
        else
        {
            labelDragInfo.Text = "Au-dessus de : -";
        }
    }

    private void ListViewCible_DragDrop(object sender, DragEventArgs e)
    {
        ListView listView = (ListView)sender;

        // Traiter le format personnalisé
        if (e.Data.GetDataPresent(FormatPersonnalisé))
        {
            // Récupérer les éléments
            List<ListViewItem> éléments = e.Data.GetData(FormatPersonnalisé) as List<ListViewItem>;

            if (éléments != null)
            {
                // Ajouter les éléments à la liste cible
                foreach (ListViewItem item in éléments)
                {
                    // Créer un nouvel élément (pour éviter les problèmes de propriété)
                    ListViewItem nouvelItem = new ListViewItem(item.Text);
                    foreach (ListViewItem.ListViewSubItem subItem in item.SubItems)
                    {
                        if (subItem != item.SubItems[0]) // Éviter de dupliquer le premier sous-élément
                        {
                            nouvelItem.SubItems.Add(subItem.Text);
                        }
                    }
                    nouvelItem.Tag = item.Tag;

                    // Ajouter à la liste cible
                    listView.Items.Add(nouvelItem);
                }

                // Si c'est un déplacement, supprimer les éléments de la source
                if (e.Effect == DragDropEffects.Move)
                {
                    foreach (ListViewItem item in éléments)
                    {
                        item.ListView.Items.Remove(item);
                    }
                }
            }
        }
        // Traiter le format texte
        else if (e.Data.GetDataPresent(DataFormats.Text))
        {
            string texte = (string)e.Data.GetData(DataFormats.Text);
            string[] lignes = texte.Split(new[] { '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);

            foreach (string ligne in lignes)
            {
                string[] colonnes = ligne.Split('\t');

                if (colonnes.Length >= 1)
                {
                    ListViewItem nouvelItem = new ListViewItem(colonnes[0]);

                    // Ajouter les colonnes supplémentaires si elles existent
                    for (int i = 1; i < colonnes.Length && i < 3; i++)
                    {
                        nouvelItem.SubItems.Add(colonnes[i]);
                    }

                    // Compléter avec des valeurs vides si nécessaire
                    while (nouvelItem.SubItems.Count < 3)
                    {
                        nouvelItem.SubItems.Add("");
                    }

                    // Ajouter à la liste
                    listView.Items.Add(nouvelItem);
                }
            }
        }

        // Mise à jour de l'interface
        labelDragInfo.Text = "Éléments déposés";
    }

    // Événements du Panel "corbeille"

    private void PanelCorbeille_DragEnter(object sender, DragEventArgs e)
    {
        // Vérifier si les données sont dans un format accepté
        if (e.Data.GetDataPresent(FormatPersonnalisé) || e.Data.GetDataPresent(DataFormats.Text))
        {
            // Afficher l'effet de suppression
            e.Effect = DragDropEffects.Move;
            panelCorbeille.BackColor = Color.Red;
            labelCorbeille.ForeColor = Color.White;
        }
        else
        {
            e.Effect = DragDropEffects.None;
        }
    }

    private void PanelCorbeille_DragLeave(object sender, EventArgs e)
    {
        // Restaurer l'apparence normale
        panelCorbeille.BackColor = Color.LightPink;
        labelCorbeille.ForeColor = Color.Black;
    }

    private void PanelCorbeille_DragDrop(object sender, DragEventArgs e)
    {
        // Traiter le format personnalisé
        if (e.Data.GetDataPresent(FormatPersonnalisé))
        {
            // Récupérer les éléments
            List<ListViewItem> éléments = e.Data.GetData(FormatPersonnalisé) as List<ListViewItem>;

            if (éléments != null)
            {
                // Demander confirmation
                DialogResult result = MessageBox.Show(
                    $"Voulez-vous vraiment supprimer {éléments.Count} élément(s) ?",
                    "Confirmation de suppression",
                    MessageBoxButtons.YesNo,
                    MessageBoxIcon.Question);

                if (result == DialogResult.Yes)
                {
                    // Supprimer les éléments
                    foreach (ListViewItem item in éléments)
                    {
                        item.ListView.Items.Remove(item);
                    }

                    // Mettre à jour l'interface
                    labelCorbeille.Text = $"{éléments.Count} élément(s) supprimé(s)";
                }
            }
        }

        // Restaurer l'apparence normale
        panelCorbeille.BackColor = Color.LightPink;
        labelCorbeille.ForeColor = Color.Black;
    }

    // Événements du TextBox pour les fichiers

    private void TextBoxFichier_DragEnter(object sender, DragEventArgs e)
    {
        // Accepter uniquement les fichiers
        if (e.Data.GetDataPresent(DataFormats.FileDrop))
        {
            string[] fichiers = (string[])e.Data.GetData(DataFormats.FileDrop);

            // Vérifier si c'est un fichier texte
            if (fichiers.Length == 1 && fichiers[0].EndsWith(".txt", StringComparison.OrdinalIgnoreCase))
            {
                e.Effect = DragDropEffects.Copy;
                textBoxFichier.BackColor = Color.LightGreen;
            }
            else
            {
                e.Effect = DragDropEffects.None;
            }
        }
        else
        {
            e.Effect = DragDropEffects.None;
        }
    }

    private void TextBoxFichier_DragDrop(object sender, DragEventArgs e)
    {
        // Récupérer les fichiers
        if (e.Data.GetDataPresent(DataFormats.FileDrop))
        {
            string[] fichiers = (string[])e.Data.GetData(DataFormats.FileDrop);

            if (fichiers.Length == 1 && fichiers[0].EndsWith(".txt", StringComparison.OrdinalIgnoreCase))
            {
                try
                {
                    // Lire le contenu du fichier
                    string contenu = File.ReadAllText(fichiers[0]);

                    // Afficher le contenu dans le TextBox
                    textBoxFichier.Text = contenu;

                    // Afficher le nom du fichier
                    labelFichier.Text = $"Fichier : {Path.GetFileName(fichiers[0])}";
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"Erreur lors de la lecture du fichier : {ex.Message}", "Erreur", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }
        }

        // Restaurer l'apparence normale
        textBoxFichier.BackColor = SystemColors.Window;
    }
}
```


Ce chapitre a présenté les fondamentaux de la gestion des événements dans les applications Windows Forms. Les événements permettent de créer des interfaces utilisateur réactives et interactives, en répondant aux actions de l'utilisateur et aux changements d'état des contrôles. En comprenant le modèle d'événement .NET et en maîtrisant les différents types d'événements disponibles, vous pouvez développer des applications Windows Forms sophistiquées qui offrent une expérience utilisateur optimale.

⏭️ 7.5. [Mise en page et design](/07-windows-forms-winforms/7-5-mise-en-page-et-design.md)
