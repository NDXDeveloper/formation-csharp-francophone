## 7.1. Introduction à WinForms

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Windows Forms (WinForms) est une technologie Microsoft permettant de créer des applications de bureau graphiques pour Windows. Conçue initialement pour .NET Framework, elle est désormais également disponible pour .NET Core et les versions plus récentes (.NET 5+). WinForms propose une approche événementielle pour développer des interfaces utilisateur et se distingue par sa simplicité de développement, notamment grâce à son concepteur visuel intégré aux IDE comme Visual Studio.
### 7.1.1. Architecture et cycle de vie
L'architecture de Windows Forms repose sur un modèle orienté événements et sur une hiérarchie de contrôles.
#### Hiérarchie des contrôles
Chaque élément d'interface dans WinForms est un contrôle qui hérite ultimement de la classe `System.Windows.Forms.Control` :
``` csharp
// Hiérarchie de base
// Control (classe de base pour tous les contrôles)
//   ↳ ScrollableControl
//     ↳ ContainerControl
//       ↳ Form (fenêtre principale de l'application)
```
#### Modèle événementiel
WinForms utilise un système d'événements pour gérer les interactions utilisateur :
``` csharp
// .NET Framework 4.7.2
public partial class MainForm : Form
{
    public MainForm()
    {
        InitializeComponent();

        // Abonnement à un événement
        button1.Click += Button1_Click;
    }

    // Gestionnaire d'événement
    private void Button1_Click(object sender, EventArgs e)
    {
        MessageBox.Show("Le bouton a été cliqué !");
    }
}
```
#### Cycle de vie d'un formulaire
Le cycle de vie d'un formulaire WinForms comprend plusieurs phases et événements clés :
``` csharp
// .NET Framework 4.7.2
public partial class MainForm : Form
{
    public MainForm()
    {
        InitializeComponent();

        // Événements du cycle de vie
        this.Load += MainForm_Load;
        this.Shown += MainForm_Shown;
        this.FormClosing += MainForm_FormClosing;
        this.FormClosed += MainForm_FormClosed;
    }

    // 1. Constructeur (déjà exécuté à ce stade)

    // 2. Événement Load : déclenché lorsque le formulaire est chargé mais avant d'être affiché
    private void MainForm_Load(object sender, EventArgs e)
    {
        // Initialisation des données, connexions aux services, etc.
        Console.WriteLine("Formulaire en cours de chargement...");
    }

    // 3. Événement Shown : déclenché lorsque le formulaire est affiché pour la première fois
    private void MainForm_Shown(object sender, EventArgs e)
    {
        // Le formulaire est maintenant visible pour l'utilisateur
        Console.WriteLine("Formulaire affiché à l'utilisateur");
    }

    // 4. Événement FormClosing : déclenché lorsque le formulaire commence à se fermer
    private void MainForm_FormClosing(object sender, FormClosingEventArgs e)
    {
        // Possibilité d'annuler la fermeture
        DialogResult result = MessageBox.Show(
            "Voulez-vous vraiment quitter ?",
            "Confirmation",
            MessageBoxButtons.YesNo);

        if (result == DialogResult.No)
        {
            e.Cancel = true; // Annule la fermeture
            Console.WriteLine("Fermeture annulée");
        }
        else
        {
            Console.WriteLine("Formulaire en cours de fermeture...");
        }
    }

    // 5. Événement FormClosed : déclenché après la fermeture du formulaire
    private void MainForm_FormClosed(object sender, FormClosedEventArgs e)
    {
        // Nettoyage final, libération des ressources
        Console.WriteLine("Formulaire fermé");
    }

    // Méthode Dispose : pour la libération des ressources
    protected override void Dispose(bool disposing)
    {
        if (disposing && (components != null))
        {
            components.Dispose();
        }
        base.Dispose(disposing);
    }
}
```
#### Point d'entrée de l'application
Le point d'entrée d'une application WinForms est généralement défini dans le fichier `Program.cs` :
``` csharp
// .NET Framework 4.7.2
static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new MainForm());
    }
}

// .NET 8
static class Program
{
    [STAThread]
    static void Main()
    {
        ApplicationConfiguration.Initialize(); // Nouveau dans .NET 6+
        Application.Run(new MainForm());
    }
}
```
#### Boucle de messages Windows
WinForms s'appuie sur la boucle de messages Windows pour gérer les événements :
``` csharp
// Exemple simplifié de la boucle de messages (interne à WinForms)
// Ce code est conceptuel et n'est pas directement utilisable
public static void Run(Form mainForm)
{
    // Affiche le formulaire
    mainForm.Show();

    // Boucle de messages
    while (GetMessage(out var msg))
    {
        TranslateMessage(ref msg);
        DispatchMessage(ref msg);
    }
}
```
#### Threads et affichage
WinForms utilise un modèle à thread unique pour l'interface utilisateur :
``` csharp
// .NET Framework 4.7.2 et .NET 8
private async void button1_Click(object sender, EventArgs e)
{
    // Désactiver le bouton pendant l'opération
    button1.Enabled = false;

    try
    {
        // Opération longue sur un thread secondaire
        var result = await Task.Run(() => {
            // Simulation d'une opération longue
            Thread.Sleep(3000);
            return "Opération terminée !";
        });

        // Mise à jour de l'interface (thread UI)
        resultLabel.Text = result;
    }
    finally
    {
        // Réactiver le bouton
        button1.Enabled = true;
    }
}

// Accès à l'interface depuis un thread secondaire
private void PerformBackgroundOperation()
{
    Thread thread = new Thread(() => {
        // Simulation d'une opération longue
        Thread.Sleep(2000);

        // Mise à jour sécurisée de l'interface
        if (this.InvokeRequired)
        {
            this.Invoke(new Action(() => {
                resultLabel.Text = "Traitement terminé";
            }));
        }
        else
        {
            resultLabel.Text = "Traitement terminé";
        }
    });

    thread.Start();
}
```
### 7.1.2. Designer visuel vs code
WinForms offre deux approches complémentaires pour la création d'interfaces : le designer visuel et le code manuel.
#### Designer visuel
Le designer visuel permet de construire l'interface par glisser-déposer et génère automatiquement le code dans un fichier partiel (généralement nommé `FormName.Designer.cs`).
**Avantages du designer visuel :**
- Création rapide d'interfaces par glisser-déposer
- Ajustement visuel des propriétés et de la mise en page
- Génération automatique du code d'initialisation
- Prévisualisation WYSIWYG (What You See Is What You Get)

**Exemple de code généré par le designer :**
``` csharp
// Fichier généré automatiquement: MainForm.Designer.cs
// Valable pour .NET Framework 4.7.2 et .NET 8
partial class MainForm
{
    private System.ComponentModel.IContainer components = null;

    private System.Windows.Forms.Button button1;
    private System.Windows.Forms.TextBox textBox1;
    private System.Windows.Forms.Label label1;

    private void InitializeComponent()
    {
        this.button1 = new System.Windows.Forms.Button();
        this.textBox1 = new System.Windows.Forms.TextBox();
        this.label1 = new System.Windows.Forms.Label();
        this.SuspendLayout();

        // button1
        this.button1.Location = new System.Drawing.Point(12, 41);
        this.button1.Name = "button1";
        this.button1.Size = new System.Drawing.Size(75, 23);
        this.button1.TabIndex = 0;
        this.button1.Text = "Valider";
        this.button1.UseVisualStyleBackColor = true;

        // textBox1
        this.textBox1.Location = new System.Drawing.Point(12, 12);
        this.textBox1.Name = "textBox1";
        this.textBox1.Size = new System.Drawing.Size(200, 20);
        this.textBox1.TabIndex = 1;

        // label1
        this.label1.AutoSize = true;
        this.label1.Location = new System.Drawing.Point(12, 67);
        this.label1.Name = "label1";
        this.label1.Size = new System.Drawing.Size(35, 13);
        this.label1.TabIndex = 2;
        this.label1.Text = "Résultat";

        // MainForm
        this.AutoScaleDimensions = new System.Drawing.SizeF(6F, 13F);
        this.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font;
        this.ClientSize = new System.Drawing.Size(284, 261);
        this.Controls.Add(this.label1);
        this.Controls.Add(this.textBox1);
        this.Controls.Add(this.button1);
        this.Name = "MainForm";
        this.Text = "Ma première application";
        this.ResumeLayout(false);
        this.PerformLayout();
    }
}
```
#### Création par code
La création d'interfaces par code offre une alternative au designer visuel :
``` csharp
// .NET Framework 4.7.2 ou .NET 8
public partial class CodeBasedForm : Form
{
    private Button submitButton;
    private TextBox inputTextBox;
    private Label resultLabel;

    public CodeBasedForm()
    {
        // Initialisation de la fenêtre
        this.Text = "Formulaire créé par code";
        this.Size = new Size(400, 300);

        // Création des contrôles
        inputTextBox = new TextBox
        {
            Location = new Point(20, 20),
            Size = new Size(250, 20)
        };

        submitButton = new Button
        {
            Text = "Soumettre",
            Location = new Point(20, 50),
            Size = new Size(100, 30)
        };

        resultLabel = new Label
        {
            Text = "En attente...",
            Location = new Point(20, 90),
            AutoSize = true
        };

        // Gestion de l'événement
        submitButton.Click += SubmitButton_Click;

        // Ajout des contrôles au formulaire
        this.Controls.Add(inputTextBox);
        this.Controls.Add(submitButton);
        this.Controls.Add(resultLabel);
    }

    private void SubmitButton_Click(object sender, EventArgs e)
    {
        if (!string.IsNullOrWhiteSpace(inputTextBox.Text))
        {
            resultLabel.Text = $"Vous avez saisi : {inputTextBox.Text}";
        }
        else
        {
            resultLabel.Text = "Veuillez saisir du texte";
        }
    }
}
```
#### Approche hybride
Une approche hybride combine designer et code :
``` csharp
// .NET Framework 4.7.2 ou .NET 8
public partial class HybridForm : Form
{
    // Le constructeur et InitializeComponent sont générés par le Designer
    public HybridForm()
    {
        InitializeComponent();

        // Configuration supplémentaire par code
        ConfigureAdvancedLayout();
        SetupDataBindings();
    }

    private void ConfigureAdvancedLayout()
    {
        // Création dynamique de contrôles supplémentaires
        for (int i = 0; i < 5; i++)
        {
            Button dynamicButton = new Button
            {
                Text = $"Bouton {i+1}",
                Location = new Point(20, 150 + (i * 30)),
                Size = new Size(100, 25),
                Tag = i  // Stockage de données associées
            };

            dynamicButton.Click += DynamicButton_Click;
            this.Controls.Add(dynamicButton);
        }
    }

    private void DynamicButton_Click(object sender, EventArgs e)
    {
        Button clickedButton = sender as Button;
        int buttonId = (int)clickedButton.Tag;

        MessageBox.Show($"Bouton dynamique {buttonId + 1} cliqué");
    }

    private void SetupDataBindings()
    {
        // Configuration de data binding par code
        // (pour les contrôles créés avec le designer)
        textBox1.DataBindings.Add("Text", myDataSource, "PropertyName");
    }
}
```
#### Comparaison des approches

| Aspect | Designer visuel | Code manuel | Approche hybride |
| --- | --- | --- | --- |
| Rapidité de développement | ★★★★★ | ★★★ | ★★★★ |
| Contrôle précis | ★★★ | ★★★★★ | ★★★★★ |
| Facilité de maintenance | ★★★ | ★★★★ | ★★★★ |
| Génération dynamique | ★ | ★★★★★ | ★★★★★ |
| Apprentissage | ★★★★★ | ★★★ | ★★★★ |
### 7.1.3. Windows Forms avec .NET Core/.NET 5+
À partir de .NET Core 3.0, Microsoft a introduit la prise en charge de Windows Forms, permettant aux développeurs de créer des applications WinForms modernes avec la plateforme .NET unifiée.
#### Principales différences entre WinForms .NET Framework et .NET Core/.NET 8
``` csharp
// Création d'un projet .NET Framework 4.7.2 (ligne de commande)
// Pas directement possible avec la CLI .NET, généralement créé avec Visual Studio

// Création d'un projet .NET 8 (ligne de commande)
// dotnet new winforms -n MonApplication -f net8.0
```
##### Modifications du point d'entrée
``` csharp
// .NET Framework 4.7.2
static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new MainForm());
    }
}

// .NET 8
static class Program
{
    [STAThread]
    static void Main()
    {
        // Cette méthode configure l'application pour utiliser les styles visuels et
        // configure automatiquement d'autres paramètres
        ApplicationConfiguration.Initialize();
        Application.Run(new MainForm());
    }
}
```
##### Différences de références et packages
``` xml
<!-- .NET Framework 4.7.2 (app.config) -->
<configuration>
    <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
    </startup>
</configuration>

<!-- .NET 8 (projet .csproj) -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <UseWindowsForms>true</UseWindowsForms>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```
#### Nouvelles fonctionnalités dans .NET 8
.NET 8 apporte plusieurs améliorations à Windows Forms :
##### 1. Haute résolution et prise en charge des écrans haute densité (DPI)
``` csharp
// .NET 8
public partial class MainForm : Form
{
    public MainForm()
    {
        InitializeComponent();

        // Activer la prise en charge DPI améliorée
        this.SetAutoScaleMode(AutoScaleMode.Dpi);

        // Vérifier la mise à l'échelle DPI actuelle
        float dpiScale = DeviceDpi / 96f;
        Console.WriteLine($"Échelle DPI actuelle: {dpiScale}x");
    }

    // Gestion des changements de DPI
    protected override void OnDpiChanged(DpiChangedEventArgs e)
    {
        base.OnDpiChanged(e);

        // Mettre à jour les contrôles pour la nouvelle échelle DPI
        float oldDpi = e.DeviceDpiOld / 96f;
        float newDpi = e.DeviceDpiNew / 96f;

        Console.WriteLine($"DPI changé de {oldDpi}x à {newDpi}x");
    }
}
```
##### 2. Support du thème sombre de Windows 10/11
``` csharp
// .NET 8
public partial class MainForm : Form
{
    public MainForm()
    {
        InitializeComponent();

        // Détecter si le thème sombre est activé
        bool isDarkMode = IsSystemInDarkMode();

        // Appliquer le thème approprié
        ApplyTheme(isDarkMode);

        // Ajouter un interrupteur de thème
        Button themeToggle = new Button
        {
            Text = "Changer de thème",
            Location = new Point(20, 200),
            Size = new Size(150, 30)
        };

        themeToggle.Click += (s, e) => {
            isDarkMode = !isDarkMode;
            ApplyTheme(isDarkMode);
        };

        this.Controls.Add(themeToggle);
    }

    private bool IsSystemInDarkMode()
    {
        using (var key = Registry.CurrentUser.OpenSubKey(@"Software\Microsoft\Windows\CurrentVersion\Themes\Personalize"))
        {
            if (key != null)
            {
                object value = key.GetValue("AppsUseLightTheme");
                if (value != null && value is int intValue)
                {
                    return intValue == 0; // 0 = Mode sombre, 1 = Mode clair
                }
            }
        }
        return false; // Par défaut, mode clair
    }

    private void ApplyTheme(bool darkMode)
    {
        if (darkMode)
        {
            // Appliquer le thème sombre
            this.BackColor = Color.FromArgb(32, 32, 32);
            this.ForeColor = Color.White;

            // Appliquer à tous les contrôles
            ApplyThemeToControls(this.Controls, true);
        }
        else
        {
            // Appliquer le thème clair
            this.BackColor = SystemColors.Control;
            this.ForeColor = SystemColors.ControlText;

            // Appliquer à tous les contrôles
            ApplyThemeToControls(this.Controls, false);
        }
    }

    private void ApplyThemeToControls(Control.ControlCollection controls, bool darkMode)
    {
        foreach (Control control in controls)
        {
            if (darkMode)
            {
                if (control is Button btn)
                {
                    btn.BackColor = Color.FromArgb(60, 60, 60);
                    btn.ForeColor = Color.White;
                    btn.FlatStyle = FlatStyle.Flat;
                    btn.FlatAppearance.BorderColor = Color.Gray;
                }
                else if (control is TextBox txt)
                {
                    txt.BackColor = Color.FromArgb(45, 45, 45);
                    txt.ForeColor = Color.White;
                }
                else if (control is Panel || control is GroupBox)
                {
                    control.BackColor = Color.FromArgb(50, 50, 50);
                    control.ForeColor = Color.White;

                    // Récursion pour les conteneurs
                    ApplyThemeToControls(control.Controls, darkMode);
                }
                else
                {
                    control.BackColor = Color.FromArgb(32, 32, 32);
                    control.ForeColor = Color.White;
                }
            }
            else
            {
                // Réinitialiser au thème par défaut
                control.BackColor = SystemColors.Control;
                control.ForeColor = SystemColors.ControlText;

                if (control is Button btn)
                {
                    btn.UseVisualStyleBackColor = true;
                    btn.FlatStyle = FlatStyle.Standard;
                }
                else if (control is TextBox txt)
                {
                    txt.BackColor = SystemColors.Window;
                    txt.ForeColor = SystemColors.WindowText;
                }
                else if (control is Panel || control is GroupBox)
                {
                    // Récursion pour les conteneurs
                    ApplyThemeToControls(control.Controls, darkMode);
                }
            }
        }
    }
}
```
##### 3. Améliorations des performances
``` csharp
// .NET 8
public partial class OptimizedForm : Form
{
    public OptimizedForm()
    {
        InitializeComponent();

        // Réduire le scintillement
        this.DoubleBuffered = true;

        // Pour les opérations de rendu personnalisées
        this.SetStyle(ControlStyles.OptimizedDoubleBuffer |
                     ControlStyles.AllPaintingInWmPaint |
                     ControlStyles.UserPaint, true);

        // Chargement différé des contrôles complexes
        this.Load += OptimizedForm_Load;
    }

    private async void OptimizedForm_Load(object sender, EventArgs e)
    {
        // Afficher une interface minimale immédiatement
        statusLabel.Text = "Chargement des composants...";

        // Charger les composants lourds de manière asynchrone
        await Task.Run(() => {
            // Préparation des données, initialisation des ressources...
            Thread.Sleep(1000); // Simulation de chargement
        });

        // Ajouter les contrôles lourds seulement quand ils sont prêts
        InitializeComplexControls();

        statusLabel.Text = "Prêt";
    }

    private void InitializeComplexControls()
    {
        // Suspendre la mise en page pour éviter les multiples rendus
        this.SuspendLayout();

        try
        {
            // Ajout de nombreux contrôles d'un coup
            for (int i = 0; i < 100; i++)
            {
                flowLayoutPanel1.Controls.Add(new Label
                {
                    Text = $"Élément {i}",
                    AutoSize = true
                });
            }
        }
        finally
        {
            // Reprendre la mise en page
            this.ResumeLayout();
        }
    }

    // Optimisation du rendu pour les contrôles personnalisés
    protected override void OnPaint(PaintEventArgs e)
    {
        // Utilisation de la mise en cache graphique
        using (BufferedGraphicsContext context = new BufferedGraphicsContext())
        {
            using (BufferedGraphics bg = context.Allocate(e.Graphics, this.ClientRectangle))
            {
                // Dessiner dans le buffer
                bg.Graphics.Clear(this.BackColor);

                // Rendu optimisé...

                // Afficher le buffer
                bg.Render();
            }
        }
    }
}
```
##### 4. Intégration avec les nouvelles fonctionnalités de C# et .NET
``` csharp
// .NET 8
public partial class ModernForm : Form
{
    // Utilisation des propriétés init (C# 9+)
    public record UserData
    {
        public required string Name { get; init; }
        public required string Email { get; init; }
        public DateTime RegisteredDate { get; init; } = DateTime.Now;
    }

    // Utilisation des types nullables (C# 8+)
    private List<UserData>? _users;

    public ModernForm()
    {
        InitializeComponent();

        // Utilisation d'initialisation de collection (C# 12)
        _users =
        [
            new() { Name = "Alice Smith", Email = "alice@example.com" },
            new() { Name = "Bob Johnson", Email = "bob@example.com" },
            new() { Name = "Carol Brown", Email = "carol@example.com" }
        ];

        // Pattern matching (C# 10+)
        button1.Click += (sender, e) =>
        {
            if (sender is Button { Tag: int id })
            {
                ProcessButtonClick(id);
            }
        };

        // Utilisation de LINQ amélioré
        var filteredUsers = _users.Where(u => u.Email.Contains("example.com"))
                                 .OrderBy(u => u.Name)
                                 .ToList();

        // Utilisation d'interfaces .NET modernes
        LoadUsersAsync();
    }

    private async Task LoadUsersAsync()
    {
        try
        {
            await using var client = new HttpClient();
            using var response = await client.GetAsync("https://api.example.com/users");

            // Pattern matching et exceptions filtrées (C# 10)
            if (response is { IsSuccessStatusCode: true })
            {
                var json = await response.Content.ReadAsStringAsync();
                dataGridView1.DataSource = JsonSerializer.Deserialize<List<UserData>>(json);
            }
        }
        catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
        {
            MessageBox.Show("Service non disponible");
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur: {ex.Message}");
        }
    }

    private void ProcessButtonClick(int id)
    {
        // Utilisation des fonctionnalités récentes de C#
        var message = id switch
        {
            1 => "Premier bouton cliqué",
            2 => "Deuxième bouton cliqué",
            _ => $"Bouton {id} cliqué"
        };

        MessageBox.Show(message);
    }
}
```
##### 5. Déploiement simplifié
``` xml
<!-- .NET 8 - fichier .csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <UseWindowsForms>true</UseWindowsForms>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>

    <!-- Options de publication -->
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
    <PublishTrimmed>true</PublishTrimmed>
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
</Project>
```

``` csharp
// Commande pour publier l'application (.NET 8)
// dotnet publish -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:PublishTrimmed=true
```
#### Compatibilité et migration
La migration de Windows Forms de .NET Framework vers .NET 8 implique plusieurs considérations :
``` csharp
// .NET 8 - Compatibilité avec le code plus ancien
public partial class MigratedForm : Form
{
    public MigratedForm()
    {
        InitializeComponent();

        // Vérifier si nous sommes sur .NET Framework ou .NET Core/.NET
        string runtimeInfo = GetRuntimeInfo();
        infoLabel.Text = runtimeInfo;

        // Utiliser des blocs de préprocesseur pour un code spécifique à la plateforme
#if NETCOREAPP || NET5_0_OR_GREATER
        // Code spécifique à .NET Core ou .NET 5+
        UseModernFeatures();
#else
        // Code spécifique à .NET Framework
        UseLegacyApproach();
#endif
    }

    private string GetRuntimeInfo()
    {
        string frameworkDescription = System.Runtime.InteropServices.RuntimeInformation
            .FrameworkDescription;

        return $"Exécution sur: {frameworkDescription}";
    }

#if NETCOREAPP || NET5_0_OR_GREATER
    private void UseModernFeatures()
    {
        // Fonctionnalités disponibles uniquement dans .NET Core / .NET
        modernButton.Text = "Fonctionnalité moderne";
        modernButton.Visible = true;
    }
#else
    private void UseLegacyApproach()
    {
        // Approche compatible avec .NET Framework
        legacyButton.Text = "Fonctionnalité classique";
        legacyButton.Visible = true;
    }
#endif
}
```
##### Techniques de migration

```textmate
// Étapes de migration (commentaires conceptuels)

// 1. Analyser les dépendances
// Utiliser l'outil d'analyse de portabilité .NET
// dotnet tool install -g upgrade-assistant

// 2. Créer un nouveau projet .NET 8
// dotnet new winforms -n MigratedApp -f net8.0

// 3. Copier les fichiers sources
// Copier les fichiers .cs et les ressources du projet original

// 4. Mettre à jour les références de packages
// Remplacer les packages .NET Framework par leurs équivalents .NET

// 5. Résoudre les problèmes de compatibilité
public class CompatibilityHelper
{
    // Exemple: Remplacement d'une fonctionnalité non disponible
    public static void ShowColorDialog(Control parent, Action<Color> onColorSelected)
    {
#if NETCOREAPP || NET5_0_OR_GREATER
        // Approche moderne
        ColorDialog dialog = new ColorDialog();
        if (dialog.ShowDialog(parent) == DialogResult.OK)
        {
            onColorSelected(dialog.Color);
        }
#else
        // Approche compatible avec .NET Framework
        using (var dialog = new System.Windows.Forms.ColorDialog())
        {
            if (dialog.ShowDialog() == DialogResult.OK)
            {
                onColorSelected(dialog.Color);
            }
        }
#endif
    }

    // Exemple: Adaptation pour les API déconseillées
    public static void SetControlFont(Control control, string fontName, float fontSize)
    {
#if NETCOREAPP || NET5_0_OR_GREATER
        // .NET 5+ utilise des méthodes modernes pour la gestion des polices
        control.Font = new Font(fontName, fontSize);
#else
        // .NET Framework utilise l'ancienne approche
        control.Font = new Font(fontName, fontSize, FontStyle.Regular);
#endif
    }
}
```


##### Exemple complet de migration

```textmate
// .NET Framework 4.7.2 (avant migration)
using System;
using System.Data;
using System.Data.SqlClient; // Package spécifique à .NET Framework
using System.Drawing;
using System.Windows.Forms;

namespace LegacyApp
{
    public partial class DataForm : Form
    {
        private SqlConnection connection;

        public DataForm()
        {
            InitializeComponent();

            string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
            connection = new SqlConnection(connectionString);

            this.Load += DataForm_Load;
        }

        private void DataForm_Load(object sender, EventArgs e)
        {
            try
            {
                connection.Open();
                LoadData();
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Erreur de connexion: {ex.Message}");
            }
        }

        private void LoadData()
        {
            string query = "SELECT ID, Name, CreatedDate FROM Customers";
            SqlDataAdapter adapter = new SqlDataAdapter(query, connection);

            DataSet dataSet = new DataSet();
            adapter.Fill(dataSet, "Customers");

            dataGridView1.DataSource = dataSet.Tables["Customers"];
        }

        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                connection?.Close();
                connection?.Dispose();
            }

            base.Dispose(disposing);
        }
    }
}

// .NET 8 (après migration)
using System;
using System.Data;
using Microsoft.Data.SqlClient; // Package moderne pour .NET
using System.Drawing;
using System.Windows.Forms;
using System.Threading.Tasks;

namespace ModernApp
{
    public partial class DataForm : Form
    {
        private SqlConnection? connection;

        public DataForm()
        {
            InitializeComponent();

            // Utilisation des secrets ou configuration moderne
            string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;TrustServerCertificate=True;";
            connection = new SqlConnection(connectionString);

            this.Load += DataForm_Load;
        }

        private async void DataForm_Load(object sender, EventArgs e)
        {
            try
            {
                // Utilisation des méthodes asynchrones
                await connection!.OpenAsync();
                await LoadDataAsync();
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Erreur de connexion: {ex.Message}");
            }
        }

        private async Task LoadDataAsync()
        {
            loadingIndicator.Visible = true;

            try
            {
                string query = "SELECT ID, Name, CreatedDate FROM Customers";

                // Approche moderne avec SqlDataAdapter
                DataTable dataTable = new DataTable();

                await Task.Run(() => {
                    using SqlDataAdapter adapter = new SqlDataAdapter(query, connection);
                    adapter.Fill(dataTable);
                });

                // Alternative: Utilisation de System.Data.Common
                // Méthode plus moderne avec SqlCommand
                /*
                dataTable.Columns.Add("ID", typeof(int));
                dataTable.Columns.Add("Name", typeof(string));
                dataTable.Columns.Add("CreatedDate", typeof(DateTime));

                using SqlCommand command = new SqlCommand(query, connection);
                using SqlDataReader reader = await command.ExecuteReaderAsync();

                while (await reader.ReadAsync())
                {
                    DataRow row = dataTable.NewRow();
                    row["ID"] = reader.GetInt32(0);
                    row["Name"] = reader.GetString(1);
                    row["CreatedDate"] = reader.GetDateTime(2);
                    dataTable.Rows.Add(row);
                }
                */

                dataGridView1.DataSource = dataTable;
            }
            finally
            {
                loadingIndicator.Visible = false;
            }
        }

        // Utilisation des implémentations modernes d'IDisposable avec support async
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                connection?.Close();
                connection?.Dispose();
            }

            base.Dispose(disposing);
        }

        // Méthode async pour la fermeture propre des ressources (appelée au besoin)
        public async Task CloseAsync()
        {
            if (connection != null && connection.State != ConnectionState.Closed)
            {
                await connection.CloseAsync();
            }
        }
    }
}
```


#### Intégration avec des frameworks modernes (MVVM, DI)

.NET 8 permet d'intégrer plus facilement Windows Forms avec des modèles d'architecture moderne :

```textmate
// .NET 8 - Intégration avec MVVM et injection de dépendances
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using System.Windows.Forms;

// Programme principal avec injection de dépendances
class Program
{
    [STAThread]
    static void Main()
    {
        ApplicationConfiguration.Initialize();

        // Configurer l'hôte d'application avec DI
        var host = Host.CreateDefaultBuilder()
            .ConfigureServices(services =>
            {
                // Enregistrer les services
                services.AddSingleton<ICustomerRepository, CustomerRepository>();
                services.AddTransient<CustomerViewModel>();
                services.AddTransient<MainForm>();
            })
            .Build();

        // Résoudre le formulaire principal avec ses dépendances injectées
        var mainForm = host.Services.GetRequiredService<MainForm>();
        Application.Run(mainForm);
    }
}

// Interface du répertoire
public interface ICustomerRepository
{
    Task<List<Customer>> GetCustomersAsync();
    Task<bool> SaveCustomerAsync(Customer customer);
}

// Implémentation du répertoire
public class CustomerRepository : ICustomerRepository
{
    public async Task<List<Customer>> GetCustomersAsync()
    {
        // Simulation de récupération de données asynchrone
        await Task.Delay(500);
        return new List<Customer>
        {
            new Customer { Id = 1, Name = "Jean Martin", Email = "jean@exemple.fr" },
            new Customer { Id = 2, Name = "Marie Dupont", Email = "marie@exemple.fr" }
        };
    }

    public async Task<bool> SaveCustomerAsync(Customer customer)
    {
        // Simulation de sauvegarde asynchrone
        await Task.Delay(500);
        return true;
    }
}

// Modèle
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

// ViewModel de base avec implémentation de INotifyPropertyChanged
public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler? PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetProperty<T>(ref T field, T value, [CallerMemberName] string? propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value))
            return false;

        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}

// ViewModel spécifique
public class CustomerViewModel : ViewModelBase
{
    private readonly ICustomerRepository _repository;
    private List<Customer> _customers = new();
    private Customer? _selectedCustomer;
    private bool _isLoading;

    public CustomerViewModel(ICustomerRepository repository)
    {
        _repository = repository;
    }

    public List<Customer> Customers
    {
        get => _customers;
        private set => SetProperty(ref _customers, value);
    }

    public Customer? SelectedCustomer
    {
        get => _selectedCustomer;
        set => SetProperty(ref _selectedCustomer, value);
    }

    public bool IsLoading
    {
        get => _isLoading;
        private set => SetProperty(ref _isLoading, value);
    }

    public async Task LoadCustomersAsync()
    {
        IsLoading = true;
        try
        {
            Customers = await _repository.GetCustomersAsync();
        }
        finally
        {
            IsLoading = false;
        }
    }

    public async Task<bool> SaveSelectedCustomerAsync()
    {
        if (SelectedCustomer == null)
            return false;

        return await _repository.SaveCustomerAsync(SelectedCustomer);
    }
}

// Formulaire principal avec MVVM
public partial class MainForm : Form
{
    private readonly CustomerViewModel _viewModel;

    // Injection de dépendances via le constructeur
    public MainForm(CustomerViewModel viewModel)
    {
        InitializeComponent();
        _viewModel = viewModel;

        // Configurer les liaisons de données
        SetupDataBindings();

        // Charger les données au démarrage
        this.Load += async (s, e) => await LoadDataAsync();
    }

    private void SetupDataBindings()
    {
        // Liaison de la liste des clients
        customerBindingSource.DataSource = _viewModel;
        customerBindingSource.DataMember = nameof(CustomerViewModel.Customers);

        dataGridView1.DataSource = customerBindingSource;

        // Liaison du client sélectionné
        customerBindingSource.CurrentChanged += (s, e) => {
            if (customerBindingSource.Current is Customer customer)
            {
                _viewModel.SelectedCustomer = customer;
            }
        };

        // Liaison de l'indicateur de chargement
        _viewModel.PropertyChanged += (s, e) => {
            if (e.PropertyName == nameof(CustomerViewModel.IsLoading))
            {
                loadingIndicator.Visible = _viewModel.IsLoading;
            }
        };
    }

    private async Task LoadDataAsync()
    {
        try
        {
            await _viewModel.LoadCustomersAsync();
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur lors du chargement des données : {ex.Message}");
        }
    }

    private async void saveButton_Click(object sender, EventArgs e)
    {
        try
        {
            bool success = await _viewModel.SaveSelectedCustomerAsync();
            if (success)
            {
                MessageBox.Show("Client enregistré avec succès !");
            }
            else
            {
                MessageBox.Show("Veuillez sélectionner un client.");
            }
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Erreur lors de l'enregistrement : {ex.Message}");
        }
    }
}
```


#### Comparaison des performances .NET Framework vs .NET 8

```textmate
// Benchmark des performances (.NET 8)
public class PerformanceComparison
{
    public static void RunBenchmark()
    {
        Console.WriteLine("Démarrage du benchmark...");

        // Test de création de contrôles
        TestControlCreation(1000);

        // Test de rendu
        TestRendering(100);

        // Test de gestion d'événements
        TestEventHandling(10000);

        Console.WriteLine("Benchmark terminé");
    }

    private static void TestControlCreation(int count)
    {
        Console.WriteLine($"Création de {count} contrôles...");

        var stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();

        using (var form = new Form())
        {
            form.SuspendLayout();

            for (int i = 0; i < count; i++)
            {
                var button = new Button
                {
                    Text = $"Bouton {i}",
                    Location = new Point(10, 10 + (i % 30) * 30),
                    Size = new Size(80, 25)
                };

                form.Controls.Add(button);
            }

            form.ResumeLayout();
        }

        stopwatch.Stop();
        Console.WriteLine($"Temps écoulé: {stopwatch.ElapsedMilliseconds} ms");
    }

    private static void TestRendering(int iterations)
    {
        Console.WriteLine($"Test de rendu avec {iterations} itérations...");

        using (var form = new Form { Size = new Size(800, 600) })
        using (var bmp = new Bitmap(800, 600))
        using (var g = Graphics.FromImage(bmp))
        {
            // Créer quelques contrôles pour le test
            for (int i = 0; i < 50; i++)
            {
                form.Controls.Add(new Button
                {
                    Text = $"Bouton {i}",
                    Location = new Point(10 + (i % 10) * 80, 10 + (i / 10) * 30),
                    Size = new Size(70, 25)
                });
            }

            var stopwatch = new System.Diagnostics.Stopwatch();
            stopwatch.Start();

            for (int i = 0; i < iterations; i++)
            {
                // Simuler un rendu du formulaire
                using (var formGraphics = form.CreateGraphics())
                {
                    form.Refresh();
                }
            }

            stopwatch.Stop();
            Console.WriteLine($"Temps écoulé: {stopwatch.ElapsedMilliseconds} ms");
        }
    }

    private static void TestEventHandling(int count)
    {
        Console.WriteLine($"Test de gestion de {count} événements...");

        int eventCounter = 0;
        Button button = new Button();

        // Ajouter un gestionnaire d'événements
        button.Click += (s, e) => { eventCounter++; };

        var stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();

        // Simuler des clics de bouton
        for (int i = 0; i < count; i++)
        {
            button.PerformClick();
        }

        stopwatch.Stop();
        Console.WriteLine($"Temps écoulé: {stopwatch.ElapsedMilliseconds} ms pour {eventCounter} événements");
    }
}
```


Ce chapitre d'introduction à Windows Forms a couvert les fondamentaux de cette technologie de développement d'applications de bureau, en mettant en évidence les différences entre .NET Framework 4.7.2 et .NET 8. Les développeurs peuvent maintenant créer des applications Windows modernes avec WinForms tout en bénéficiant des avantages du framework .NET moderne, notamment les performances améliorées, la prise en charge de nouvelles fonctionnalités et l'intégration avec des pratiques de développement contemporaines.

⏭️ 7.2. [Création d'une première application](/07-windows-forms-winforms/7-2-creation-dune-premiere-application.md)
