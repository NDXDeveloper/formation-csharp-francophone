# 4.6. Méthodes locales (C# 7+)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les méthodes locales sont une fonctionnalité introduite dans C# 7.0 qui vous permet de définir des méthodes à l'intérieur d'autres méthodes. Cette capacité apparemment simple apporte une grande flexibilité dans l'organisation de votre code et la gestion de sa portée. Dans cette section, nous allons explorer comment définir et utiliser des méthodes locales, ainsi que leurs avantages et cas d'utilisation.

## Introduction aux méthodes locales

Avant C# 7.0, si vous vouliez créer une méthode utilisée uniquement par une autre méthode, vous deviez la définir au niveau de la classe. Avec les méthodes locales, vous pouvez désormais définir ces méthodes d'assistance directement à l'intérieur de la méthode qui les utilise.

Pour comprendre la différence, regardons un exemple simple :

**Avant C# 7.0 (méthode privée au niveau de la classe) :**

```csharp
public class Calculateur
{
    public int CalculerSommeCarres(int[] nombres)
    {
        return SommeDesCarres(nombres);
    }

    private int SommeDesCarres(int[] nombres)
    {
        int somme = 0;
        foreach (int nombre in nombres)
        {
            somme += nombre * nombre;
        }
        return somme;
    }
}
```

**Avec C# 7.0+ (méthode locale) :**

```csharp
public class Calculateur
{
    public int CalculerSommeCarres(int[] nombres)
    {
        int SommeDesCarres(int[] values)
        {
            int somme = 0;
            foreach (int nombre in values)
            {
                somme += nombre * nombre;
            }
            return somme;
        }

        return SommeDesCarres(nombres);
    }
}
```

Comme vous pouvez le voir, dans le second exemple, la méthode `SommeDesCarres` est définie à l'intérieur de la méthode `CalculerSommeCarres`. Elle est ainsi "locale" à cette méthode.

## 4.6.1. Définition dans d'autres méthodes

### Syntaxe de base

La syntaxe pour définir une méthode locale est similaire à celle d'une méthode normale, à la différence que vous la placez à l'intérieur d'une autre méthode :

```csharp
public void MethodeExterne()
{
    // Définition d'une méthode locale
    void MethodeLocale()
    {
        // Corps de la méthode locale
        Console.WriteLine("Je suis une méthode locale !");
    }

    // Appel de la méthode locale
    MethodeLocale();

    // Autre code...
}
```

### Caractéristiques des méthodes locales

Les méthodes locales ont plusieurs caractéristiques importantes à comprendre :

#### 1. Elles peuvent accéder aux variables de la méthode englobante

```csharp
public void TraiterDonnees(int[] donnees)
{
    int seuil = 10; // Variable de la méthode englobante

    bool EstValide(int valeur)
    {
        return valeur > seuil; // Accès à la variable 'seuil'
    }

    foreach (int valeur in donnees)
    {
        if (EstValide(valeur))
        {
            Console.WriteLine($"Valeur valide trouvée : {valeur}");
        }
    }
}
```

#### 2. Elles peuvent être définies n'importe où dans la méthode englobante

Contrairement aux variables, qui doivent être déclarées avant d'être utilisées, les méthodes locales peuvent être définies n'importe où dans la méthode englobante, même après leur utilisation :

```csharp
public void ExemplePositionnement()
{
    // La méthode est appelée avant d'être définie
    AfficherMessage("Bonjour");

    // Puis définie plus tard
    void AfficherMessage(string message)
    {
        Console.WriteLine(message);
    }
}
```

Cependant, par souci de lisibilité, il est généralement préférable de définir les méthodes locales avant ou juste après leur première utilisation.

#### 3. Elles prennent en charge tous les modificateurs de paramètres

Les méthodes locales peuvent utiliser tous les modificateurs de paramètres standard comme `ref`, `out`, `in` et `params` :

```csharp
public bool TryParse(string input, out int result)
{
    result = 0;

    bool EstDigit(char c)
    {
        return char.IsDigit(c);
    }

    void AjouterChiffre(char chiffre, ref int valeurCourante)
    {
        valeurCourante = valeurCourante * 10 + (chiffre - '0');
    }

    if (string.IsNullOrEmpty(input))
        return false;

    foreach (char c in input)
    {
        if (!EstDigit(c))
            return false;

        AjouterChiffre(c, ref result);
    }

    return true;
}
```

#### 4. Elles peuvent être récursives

Les méthodes locales peuvent s'appeler elles-mêmes, permettant de créer des algorithmes récursifs :

```csharp
public int Factorielle(int n)
{
    if (n < 0)
        throw new ArgumentException("Le nombre doit être positif ou nul");

    int CalculFactorielle(int x)
    {
        if (x <= 1)
            return 1;
        else
            return x * CalculFactorielle(x - 1);
    }

    return CalculFactorielle(n);
}
```

#### 5. Elles peuvent utiliser des génériques

Les méthodes locales peuvent également être génériques :

```csharp
public T TrouverMaximum<T>(T[] valeurs) where T : IComparable<T>
{
    if (valeurs == null || valeurs.Length == 0)
        throw new ArgumentException("Le tableau ne peut pas être vide ou null");

    T Maximum<U>(U[] elements) where U : IComparable<U>
    {
        U max = elements[0];
        for (int i = 1; i < elements.Length; i++)
        {
            if (elements[i].CompareTo(max) > 0)
                max = elements[i];
        }
        return max;
    }

    return Maximum(valeurs);
}
```

#### 6. Elles peuvent être asynchrones

Vous pouvez définir des méthodes locales asynchrones avec le mot-clé `async` :

```csharp
public async Task ProcessAsync(string[] urls)
{
    async Task<string> TelechargerContenuAsync(string url)
    {
        using (HttpClient client = new HttpClient())
        {
            return await client.GetStringAsync(url);
        }
    }

    List<Task<string>> taches = new List<Task<string>>();

    foreach (string url in urls)
    {
        taches.Add(TelechargerContenuAsync(url));
    }

    string[] resultats = await Task.WhenAll(taches);

    // Traitement des résultats...
}
```

#### 7. Elles peuvent être statiques

À partir de C# 8.0, vous pouvez définir des méthodes locales statiques. Cela garantit qu'elles n'accèdent pas par inadvertance aux variables de la méthode englobante :

```csharp
public void TraiterDonnees(int[] donnees)
{
    int seuil = 10;

    // La méthode locale statique ne peut pas accéder à 'seuil'
    static bool EstPair(int valeur)
    {
        // Erreur de compilation si on essaie d'utiliser 'seuil' ici
        return valeur % 2 == 0;
    }

    foreach (int valeur in donnees)
    {
        if (EstPair(valeur) && valeur > seuil)
        {
            Console.WriteLine($"Valeur paire supérieure au seuil : {valeur}");
        }
    }
}
```

### Limitations des méthodes locales

Les méthodes locales ont quelques limitations par rapport aux méthodes normales :

1. Elles ne peuvent pas contenir de membres statiques.
2. Elles n'acceptent pas les attributs comme `[Obsolete]` (avant C# 9.0).
3. Elles ne peuvent pas être utilisées comme délégués directement (mais peuvent être enveloppées dans un délégué).

## 4.6.2. Avantages et cas d'utilisation

Les méthodes locales offrent plusieurs avantages qui les rendent utiles dans certains scénarios.

### Avantages des méthodes locales

#### 1. Portée limitée

Les méthodes locales ne sont pas accessibles en dehors de la méthode qui les contient, ce qui permet :
- De réduire la surface d'API de votre classe
- D'éviter que d'autres parties du code n'utilisent des méthodes destinées à un usage interne spécifique
- De mieux encapsuler le code

```csharp
public class GestionnaireDocuments
{
    public Document TraiterDocument(string contenu)
    {
        // Cette logique de validation n'est pertinente que pour TraiterDocument
        bool EstValide(string texte)
        {
            return !string.IsNullOrEmpty(texte) && texte.Length > 10;
        }

        if (!EstValide(contenu))
            throw new ArgumentException("Contenu de document invalide");

        // Traitement du document...
        return new Document(contenu);
    }

    // Autres méthodes qui n'ont pas besoin d'accéder à EstValide()...
}
```

#### 2. Accès aux variables locales et paramètres

Contrairement aux méthodes privées au niveau de la classe, les méthodes locales peuvent accéder directement aux variables locales et aux paramètres de la méthode englobante :

```csharp
public List<int> FiltrerEtTransformer(List<int> nombres, int seuil, int facteur)
{
    // La méthode locale accède à 'facteur' de la méthode englobante
    int Transformer(int n)
    {
        return n * facteur;
    }

    var resultat = new List<int>();
    foreach (var n in nombres)
    {
        if (n > seuil)
        {
            resultat.Add(Transformer(n));
        }
    }

    return resultat;
}
```

#### 3. Meilleure lisibilité du code

Les méthodes locales permettent de placer le code d'assistance près de son point d'utilisation, ce qui améliore la lisibilité :

```csharp
public double CalculerStatistiques(double[] valeurs, out double moyenne, out double mediane)
{
    if (valeurs == null || valeurs.Length == 0)
        throw new ArgumentException("Le tableau ne peut pas être vide ou null");

    // Méthodes locales pour les calculs auxiliaires
    double CalculerMoyenne(double[] data)
    {
        double somme = 0;
        foreach (var val in data)
            somme += val;
        return somme / data.Length;
    }

    double CalculerMediane(double[] data)
    {
        // Copie pour ne pas modifier l'original
        double[] copie = new double[data.Length];
        Array.Copy(data, copie, data.Length);
        Array.Sort(copie);

        int milieu = copie.Length / 2;
        if (copie.Length % 2 == 0)
            return (copie[milieu - 1] + copie[milieu]) / 2;
        else
            return copie[milieu];
    }

    double CalculerEcartType(double[] data, double moy)
    {
        double sommeCarresEcarts = 0;
        foreach (var val in data)
            sommeCarresEcarts += Math.Pow(val - moy, 2);
        return Math.Sqrt(sommeCarresEcarts / data.Length);
    }

    // Utilisation des méthodes locales
    moyenne = CalculerMoyenne(valeurs);
    mediane = CalculerMediane(valeurs);
    double ecartType = CalculerEcartType(valeurs, moyenne);

    return ecartType;
}
```

#### 4. Performance potentiellement améliorée

Dans certains cas, les méthodes locales peuvent être plus performantes que les méthodes privées et les expressions lambda :

- Elles ne nécessitent pas de créer une instance de délégué comme les expressions lambda
- Elles peuvent être implémentées de manière plus efficace par le compilateur
- Les méthodes locales statiques évitent la création de closures

#### 5. Clarté d'intention

Les méthodes locales expriment clairement qu'une logique est strictement liée à la méthode qui la contient, ce qui communique mieux l'intention :

```csharp
public async Task<User> AuthenticateUserAsync(string username, string password)
{
    // Méthode locale pour valider les entrées
    void ValidateInputs()
    {
        if (string.IsNullOrEmpty(username))
            throw new ArgumentException("Le nom d'utilisateur ne peut pas être vide");
        if (string.IsNullOrEmpty(password))
            throw new ArgumentException("Le mot de passe ne peut pas être vide");
    }

    // Méthode locale pour journaliser les tentatives
    async Task LogLoginAttemptAsync(bool success)
    {
        var log = new LoginAttempt
        {
            Username = username,
            Timestamp = DateTime.UtcNow,
            Success = success,
            IpAddress = GetClientIp()
        };

        await _loginRepository.SaveAsync(log);
    }

    // Exécution principale
    ValidateInputs();

    var user = await _userRepository.FindByUsernameAsync(username);
    if (user == null || !VerifyPassword(user, password))
    {
        await LogLoginAttemptAsync(false);
        return null;
    }

    await LogLoginAttemptAsync(true);
    return user;
}
```

### Cas d'utilisation courants

Voici des situations où les méthodes locales sont particulièrement utiles :

#### 1. Algorithmes récursifs

Les méthodes locales sont parfaites pour implémenter des algorithmes récursifs qui ne sont pas nécessaires ailleurs :

```csharp
public List<FileInfo> TrouverFichiers(string repertoire, string pattern)
{
    var resultat = new List<FileInfo>();

    void Explorer(string rep)
    {
        // Ajouter les fichiers du répertoire actuel
        foreach (var fichier in Directory.GetFiles(rep, pattern))
        {
            resultat.Add(new FileInfo(fichier));
        }

        // Explorer récursivement les sous-répertoires
        foreach (var sousRep in Directory.GetDirectories(rep))
        {
            Explorer(sousRep);
        }
    }

    Explorer(repertoire);
    return resultat;
}
```

#### 2. Validation d'entrée

Isoler la logique de validation dans des méthodes locales :

```csharp
public Product CreateProduct(string name, decimal price, int stock)
{
    void ValidateProduct()
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Le nom du produit ne peut pas être vide");

        if (price <= 0)
            throw new ArgumentException("Le prix doit être positif");

        if (stock < 0)
            throw new ArgumentException("Le stock ne peut pas être négatif");
    }

    ValidateProduct();

    // Création du produit après validation
    return new Product
    {
        Name = name,
        Price = price,
        StockQuantity = stock
    };
}
```

#### 3. Itérateurs et générateurs

Les méthodes locales sont utiles dans les itérateurs ou générateurs complexes :

```csharp
public IEnumerable<int> GenerateSequence(int start, int count)
{
    if (count < 0)
        throw new ArgumentException("Le nombre d'éléments doit être positif ou nul");

    IEnumerable<int> SequenceGenerator()
    {
        int current = start;
        for (int i = 0; i < count; i++)
        {
            // Logique complexe pour générer la séquence
            yield return current;
            current = current * 2 + 1;
        }
    }

    // Vérifications préliminaires, puis génération de la séquence
    return SequenceGenerator();
}
```

#### 4. Opérations asynchrones

Organiser des opérations asynchrones complexes :

```csharp
public async Task<OrderResult> ProcessOrderAsync(Order order)
{
    // Vérifications initiales
    if (order == null)
        throw new ArgumentNullException(nameof(order));

    // Méthode locale pour vérifier la disponibilité des articles
    async Task<bool> VerifierDisponibiliteAsync()
    {
        foreach (var item in order.Items)
        {
            bool disponible = await _inventoryService.IsAvailableAsync(item.ProductId, item.Quantity);
            if (!disponible)
                return false;
        }
        return true;
    }

    // Méthode locale pour traiter le paiement
    async Task<PaymentResult> TraiterPaiementAsync()
    {
        var paymentRequest = new PaymentRequest
        {
            Amount = order.TotalAmount,
            CustomerId = order.CustomerId,
            PaymentMethod = order.PaymentMethod
        };

        return await _paymentService.ProcessPaymentAsync(paymentRequest);
    }

    // Méthode locale pour mettre à jour l'inventaire
    async Task MettreAJourInventaireAsync()
    {
        foreach (var item in order.Items)
        {
            await _inventoryService.DecreaseStockAsync(item.ProductId, item.Quantity);
        }
    }

    // Exécution du traitement de la commande
    if (!await VerifierDisponibiliteAsync())
        return new OrderResult { Success = false, Message = "Articles non disponibles" };

    var paymentResult = await TraiterPaiementAsync();
    if (!paymentResult.Success)
        return new OrderResult { Success = false, Message = "Échec du paiement: " + paymentResult.Message };

    await MettreAJourInventaireAsync();
    await _orderRepository.SaveAsync(order);

    return new OrderResult { Success = true, OrderId = order.Id };
}
```

#### 5. Algorithmes complexes divisés en étapes

Diviser un algorithme complexe en étapes logiques sans polluer l'espace de noms de la classe :

```csharp
public Image ProcessImage(Image source)
{
    // Étape 1: Redimensionner l'image
    Image Redimensionner(Image img)
    {
        // Logique de redimensionnement...
        return img; // Simplifié pour l'exemple
    }

    // Étape 2: Ajuster les couleurs
    Image AjusterCouleurs(Image img)
    {
        // Logique d'ajustement des couleurs...
        return img; // Simplifié pour l'exemple
    }

    // Étape 3: Appliquer des filtres
    Image AppliquerFiltres(Image img)
    {
        // Application de divers filtres...
        return img; // Simplifié pour l'exemple
    }

    // Exécution de l'algorithme complet
    var imageRedimensionnee = Redimensionner(source);
    var imageAjustee = AjusterCouleurs(imageRedimensionnee);
    var imageFinale = AppliquerFiltres(imageAjustee);

    return imageFinale;
}
```

### Bonnes pratiques pour les méthodes locales

Pour tirer le meilleur parti des méthodes locales, voici quelques bonnes pratiques à suivre :

#### 1. Gardez les méthodes locales courtes et focalisées

Les méthodes locales devraient avoir une responsabilité unique et clairement définie :

```csharp
// Bon exemple - chaque méthode a une responsabilité unique
public double CalculerPret(double montant, double tauxAnnuel, int dureeEnMois)
{
    double CalculerTauxMensuel()
    {
        return tauxAnnuel / 12 / 100;
    }

    double CalculerPaiementMensuel(double taux)
    {
        return montant * taux * Math.Pow(1 + taux, dureeEnMois)
               / (Math.Pow(1 + taux, dureeEnMois) - 1);
    }

    var tauxMensuel = CalculerTauxMensuel();
    return CalculerPaiementMensuel(tauxMensuel);
}
```

#### 2. Placez les méthodes locales à un endroit logique

Généralement, définissez les méthodes locales au début ou à la fin de la méthode englobante, ou juste avant leur première utilisation :

```csharp
// Au début de la méthode (bon pour donner une vue d'ensemble)
public void Methode()
{
    // Définition des méthodes locales
    void Etape1() { /* ... */ }
    void Etape2() { /* ... */ }

    // Corps principal
    Etape1();
    Etape2();
}

// Ou juste avant l'utilisation (bon pour le contexte immédiat)
public void AutreMethode()
{
    // Quelques opérations initiales...

    // Définie juste avant son utilisation
    void OperationSpecifique() { /* ... */ }
    OperationSpecifique();

    // Suite du code...
}
```

#### 3. Utilisez des méthodes locales statiques quand approprié

À partir de C# 8.0, utilisez le mot-clé `static` pour les méthodes locales qui n'ont pas besoin d'accéder aux variables locales de la méthode englobante. Cela peut améliorer les performances et éviter des erreurs :

```csharp
public void TraiterDonnees(List<int> donnees)
{
    // Cette méthode ne dépend pas des variables de TraiterDonnees
    // donc elle peut être statique
    static bool EstNombrePremier(int n)
    {
        if (n <= 1) return false;
        if (n == 2) return true;
        if (n % 2 == 0) return false;

        var limite = (int)Math.Sqrt(n);
        for (int i = 3; i <= limite; i += 2)
        {
            if (n % i == 0) return false;
        }

        return true;
    }

    // Utilisation
    var nombresPremiers = donnees.Where(n => EstNombrePremier(n)).ToList();
    // Suite du traitement...
}
```

#### 4. Nommez vos méthodes locales de manière descriptive

Comme pour toutes les méthodes, utilisez des noms qui décrivent clairement leur fonction :

```csharp
// Bon exemple - noms descriptifs
public decimal CalculerTaxes(decimal montant, string region)
{
    decimal ObtenirTauxTVA(string reg)
    {
        switch (reg.ToUpper())
        {
            case "FR": return 0.20m;
            case "BE": return 0.21m;
            case "DE": return 0.19m;
            default: return 0.20m;
        }
    }

    decimal CalculerMontantTVA(decimal mont, decimal taux)
    {
        return mont * taux;
    }

    var tauxTVA = ObtenirTauxTVA(region);
    return CalculerMontantTVA(montant, tauxTVA);
}
```

#### 5. Convertissez les méthodes locales complexes en méthodes privées si nécessaire

Si une méthode locale devient trop complexe ou réutilisable ailleurs, envisagez de la transformer en méthode privée au niveau de la classe :

```csharp
// Si la logique devient complexe ou réutilisable
public class CalculateurGeometrique
{
    public double CalculerAirePolygone(Point[] points)
    {
        // Pour un algorithme complexe, mieux vaut une méthode privée
        return CalculerAirePolygoneAvecAlgorithmeShoe(points);
    }

    private double CalculerAirePolygoneAvecAlgorithmeShoe(Point[] points)
    {
        // Implémentation de l'algorithme du "lacet de chaussure"
        // (Shoelace formula ou algorithme de Gauss)
        double aire = 0;
        int j = points.Length - 1;

        for (int i = 0; i < points.Length; i++)
        {
            aire += (points[j].X + points[i].X) * (points[j].Y - points[i].Y);
            j = i;
        }

        return Math.Abs(aire / 2);
    }
}
```

## Conclusion

Les méthodes locales constituent un outil puissant dans la boîte à outils de tout développeur C#. Elles vous permettent d'encapsuler la logique là où elle est utilisée, d'améliorer la lisibilité du code et parfois même d'optimiser les performances.

En résumé, utilisez les méthodes locales lorsque :
- La logique n'est pertinente que dans le contexte d'une méthode spécifique
- Vous avez besoin d'accéder directement aux variables locales et paramètres
- Vous souhaitez clairement communiquer que certaines fonctionnalités font partie d'un algorithme plus vaste
- Vous implémentez des algorithmes récursifs ou itératifs complexes
- Vous voulez organiser votre code de manière plus modulaire sans exposer des méthodes privées inutiles

Les méthodes locales représentent une excellente façon d'améliorer la structure et la lisibilité de votre code tout en maintenant une bonne encapsulation.

⏭️
