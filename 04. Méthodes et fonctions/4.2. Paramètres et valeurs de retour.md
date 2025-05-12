# 4.2. Paramètres et valeurs de retour en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les paramètres et les valeurs de retour sont essentiels pour créer des méthodes flexibles et réutilisables en C#. Cette section explore les différentes façons de passer des données à vos méthodes et de récupérer des résultats.

## 4.2.1. Passage par valeur vs référence (ref, out)

En C#, il existe deux façons principales de passer des arguments à une méthode : par valeur et par référence. Comprendre la différence est crucial pour manipuler correctement les données.

### Passage par valeur (comportement par défaut)

Par défaut, les arguments sont passés **par valeur** en C#. Cela signifie qu'une copie de la valeur est créée et transmise à la méthode. Toute modification effectuée sur ce paramètre à l'intérieur de la méthode n'affecte pas la variable d'origine.

```csharp
public void ModifierNombre(int nombre)
{
    nombre = nombre * 2; // Cette modification n'affecte que la copie locale
    Console.WriteLine($"Valeur à l'intérieur de la méthode : {nombre}");
}

// Utilisation
int x = 10;
ModifierNombre(x);
Console.WriteLine($"Valeur après l'appel de la méthode : {x}");

// Résultat :
// Valeur à l'intérieur de la méthode : 20
// Valeur après l'appel de la méthode : 10 (inchangée)
```

Il est important de distinguer ce qui se passe avec les types valeur et les types référence :

- **Types valeur** (int, double, bool, struct, etc.) : une copie complète de la valeur est créée
- **Types référence** (classes, interfaces, tableaux, etc.) : une copie de la référence est créée, pointant vers le même objet en mémoire

Avec les types référence, même s'ils sont passés par valeur, vous pouvez modifier l'objet lui-même (mais pas remplacer la référence) :

```csharp
public void ModifierListe(List<int> liste)
{
    // Ceci modifie l'objet liste (fonctionne car nous modifions l'objet, pas la référence)
    liste.Add(999);

    // Mais ceci ne modifie que la copie locale de la référence
    liste = new List<int>(); // N'affecte pas la liste d'origine
}

// Utilisation
List<int> nombres = new List<int> { 1, 2, 3 };
ModifierListe(nombres);
Console.WriteLine(string.Join(", ", nombres)); // Affiche : 1, 2, 3, 999
```

### Passage par référence avec le mot-clé `ref`

Le mot-clé `ref` permet de passer une variable par référence. Cela signifie que la méthode travaille directement sur la variable d'origine, pas sur une copie.

```csharp
public void DoublerNombre(ref int nombre)
{
    nombre = nombre * 2; // Modifie la variable d'origine
}

// Utilisation
int x = 10;
DoublerNombre(ref x); // Notez le mot-clé 'ref' aussi à l'appel
Console.WriteLine(x); // Affiche : 20
```

Points importants concernant `ref` :
- La variable doit être initialisée avant d'être passée avec `ref`
- Le mot-clé `ref` est nécessaire à la fois dans la signature de la méthode et lors de l'appel
- Fonctionne avec les types valeur et les types référence

```csharp
// Avec un type référence
public void ReinitialiserListe(ref List<int> liste)
{
    // Ceci remplace complètement la référence d'origine
    liste = new List<int>();
}

// Utilisation
List<int> nombres = new List<int> { 1, 2, 3 };
ReinitialiserListe(ref nombres);
Console.WriteLine(nombres.Count); // Affiche : 0 (liste vide)
```

### Passage par référence avec le mot-clé `out`

Le mot-clé `out` est similaire à `ref`, mais avec deux différences importantes :
1. La variable n'a pas besoin d'être initialisée avant l'appel
2. La méthode **doit** assigner une valeur au paramètre avant de terminer

`out` est principalement utilisé pour retourner plusieurs valeurs d'une méthode :

```csharp
public bool TryDiviser(int dividende, int diviseur, out int resultat)
{
    if (diviseur == 0)
    {
        resultat = 0; // Doit assigner une valeur même en cas d'échec
        return false;
    }

    resultat = dividende / diviseur;
    return true;
}

// Utilisation
int x = 10;
int y = 2;
int z;

if (TryDiviser(x, y, out z))
{
    Console.WriteLine($"Résultat : {z}"); // Affiche : Résultat : 5
}
else
{
    Console.WriteLine("Division impossible");
}

// Avec C# 7+, déclaration inline lors de l'appel
if (TryDiviser(x, 0, out int resultat))
{
    Console.WriteLine($"Résultat : {resultat}");
}
else
{
    Console.WriteLine("Division impossible"); // Ce message s'affiche
}
```

### Comparaison entre `ref` et `out`

| Caractéristique | `ref` | `out` |
|----------------|-------|-------|
| Nécessite initialisation avant appel | Oui | Non |
| Nécessite assignation dans la méthode | Non | Oui |
| Usage typique | Modifier une variable existante | Retourner plusieurs valeurs |
| Mot-clé requis lors de l'appel | Oui | Oui |

### Quand utiliser chaque approche ?

- **Passage par valeur** (par défaut) : Pour la plupart des cas, surtout quand vous avez juste besoin de lire les paramètres
- **ref** : Quand vous voulez modifier une variable existante
- **out** : Quand vous voulez retourner plusieurs valeurs depuis une méthode

## 4.2.2. Paramètres de tableau params

Le mot-clé `params` permet de passer un nombre variable d'arguments du même type à une méthode. Il doit être appliqué au dernier paramètre de la méthode, qui doit être un tableau à une dimension.

### Syntaxe de base

```csharp
public int Somme(params int[] nombres)
{
    int total = 0;
    foreach (int nombre in nombres)
    {
        total += nombre;
    }
    return total;
}

// Utilisation flexible
int resultat1 = Somme(1, 2);                  // 3
int resultat2 = Somme(1, 2, 3, 4, 5);         // 15
int resultat3 = Somme();                      // 0 (tableau vide)

// On peut aussi passer un tableau directement
int[] tableau = { 10, 20, 30 };
int resultat4 = Somme(tableau);               // 60
```

### Avantages des paramètres params

1. **Flexibilité** : Les appelants peuvent fournir un nombre quelconque d'arguments
2. **Lisibilité** : Rend le code d'appel plus propre sans avoir à créer explicitement un tableau
3. **Polyvalence** : Accepte à la fois des arguments individuels et des tableaux

### Combinaison avec d'autres paramètres

`params` peut être combiné avec d'autres paramètres, mais il doit toujours être le dernier :

```csharp
public string Formater(string format, params object[] valeurs)
{
    return string.Format(format, valeurs);
}

// Utilisation
string message = Formater("Bonjour {0}, vous avez {1} ans", "Marie", 29);
Console.WriteLine(message); // Affiche : Bonjour Marie, vous avez 29 ans
```

### Exemple pratique : Méthode d'enregistrement des logs

```csharp
public void Log(string niveau, string message, params object[] donnees)
{
    string timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
    string baseLog = $"[{timestamp}] {niveau}: {message}";

    if (donnees.Length > 0)
    {
        string donneesString = string.Join(", ", donnees);
        baseLog += $" - Données: {donneesString}";
    }

    Console.WriteLine(baseLog);
}

// Utilisation
Log("INFO", "Application démarrée");
Log("WARNING", "Fichier introuvable", "config.json", "C:/app/config/");
Log("ERROR", "Exception détectée", new Exception("Détail de l'erreur"), 404);
```

## 4.2.3. Tuples comme valeurs de retour (C# 7+)

À partir de C# 7.0, les tuples offrent une façon élégante de retourner plusieurs valeurs d'une méthode sans avoir à créer une classe spécifique ou à utiliser des paramètres `out`.

### Syntaxe de base

```csharp
public (int Quotient, int Reste) Diviser(int dividende, int diviseur)
{
    int quotient = dividende / diviseur;
    int reste = dividende % diviseur;

    return (quotient, reste); // Retourne un tuple
}

// Utilisation
var resultat = Diviser(10, 3);
Console.WriteLine($"Quotient: {resultat.Quotient}, Reste: {resultat.Reste}");
// Affiche : Quotient: 3, Reste: 1
```

### Différentes façons d'accéder aux valeurs

```csharp
// 1. Par les noms définis dans la signature
var resultat1 = Diviser(10, 3);
Console.WriteLine($"{resultat1.Quotient} et {resultat1.Reste}");

// 2. Par décomposition (destructuring)
var (quotient, reste) = Diviser(10, 3);
Console.WriteLine($"{quotient} et {reste}");

// 3. Par position (Item1, Item2, etc.) si les noms sont omis
(int q, int r) = Diviser(10, 3); // Autre forme de décomposition
Console.WriteLine($"{q} et {r}");
```

### Tuples sans noms explicites

Vous pouvez aussi définir des tuples sans nommer les éléments :

```csharp
public (int, int) DiviserSansNoms(int dividende, int diviseur)
{
    return (dividende / diviseur, dividende % diviseur);
}

// Utilisation
var res = DiviserSansNoms(10, 3);
Console.WriteLine($"{res.Item1} et {res.Item2}"); // Utilisation des noms génériques
```

### Cas d'utilisation courants

1. **Retourner plusieurs valeurs calculées** :

```csharp
public (double Min, double Max, double Moyenne) CalculerStatistiques(int[] nombres)
{
    return (
        nombres.Min(),
        nombres.Max(),
        nombres.Average()
    );
}

// Utilisation
var stats = CalculerStatistiques(new[] { 1, 5, 10, 15, 20 });
Console.WriteLine($"Min: {stats.Min}, Max: {stats.Max}, Moyenne: {stats.Moyenne}");
```

2. **Méthodes de validation retournant un statut et un message** :

```csharp
public (bool EstValide, string Message) ValiderUtilisateur(string nom, string email)
{
    if (string.IsNullOrEmpty(nom))
        return (false, "Le nom est requis");

    if (string.IsNullOrEmpty(email) || !email.Contains("@"))
        return (false, "Email invalide");

    return (true, "Utilisateur valide");
}

// Utilisation
var validation = ValiderUtilisateur("Jean", "jean@example.com");
if (validation.EstValide)
{
    Console.WriteLine("Utilisateur enregistré");
}
else
{
    Console.WriteLine($"Erreur: {validation.Message}");
}
```

3. **Extraction et transformation de données** :

```csharp
public (string Nom, int Age, string[] Competences) ExtraireInfosProfil(string profilJson)
{
    // Dans un cas réel, utilisez System.Text.Json ou Newtonsoft.Json
    // Simplifié pour l'exemple
    return ("Marie Dupont", 32, new[] { "C#", "SQL", "Azure" });
}
```

## 4.2.4. Discards (C# 7+)

Les discards (représentés par le caractère `_`) sont des variables temporaires qui sont délibérément inutilisées. Ils sont utiles quand vous voulez ignorer certaines valeurs retournées par une méthode.

### Utilisation de base

```csharp
// Ignorer certaines valeurs lors de la décomposition d'un tuple
var (quotient, _) = Diviser(10, 3); // On ignore le reste
Console.WriteLine($"Quotient: {quotient}"); // On utilise seulement le quotient

// Avec plusieurs discards
(_, _, double moyenne) = CalculerStatistiques(new[] { 1, 2, 3, 4, 5 });
Console.WriteLine($"Moyenne: {moyenne}"); // On utilise seulement la moyenne
```

### Avec les méthodes out

Les discards sont particulièrement utiles avec les méthodes utilisant des paramètres `out` dont vous n'avez pas besoin :

```csharp
// Exemple avec int.TryParse
string input = "123";
if (int.TryParse(input, out int nombre))
{
    Console.WriteLine($"Conversion réussie: {nombre}");
}

// Si on veut juste savoir si la conversion est possible, sans utiliser le résultat
if (int.TryParse(input, out _))
{
    Console.WriteLine("La chaîne contient un nombre valide");
}
```

### Discards multiples avec des valeurs différentes

Contrairement aux variables normales, vous pouvez utiliser plusieurs fois le discard `_` dans la même portée, même pour des valeurs de types différents :

```csharp
// Exemple avec plusieurs appels de méthode
bool success1 = int.TryParse("123", out _);
bool success2 = double.TryParse("45.67", out _);
bool success3 = DateTime.TryParse("2023-05-01", out _);

// Tout ceci est valide dans la même portée
```

### Limites des discards

1. Vous ne pouvez pas lire la valeur d'un discard (c'est une variable en écriture seule)
2. Vous ne pouvez pas utiliser un discard comme argument nommé

```csharp
// Invalide - tentative de lire un discard
if (int.TryParse("123", out _))
{
    Console.WriteLine(_); // Erreur de compilation
}

// Invalide - tentative d'utiliser un discard comme argument nommé
Diviser(dividende: 10, diviseur: _); // Erreur de compilation
```

### Cas d'utilisation pratiques

1. **Ignorer les informations inutiles** :

```csharp
// Si vous ne vous intéressez qu'au nom du fichier sans l'extension
var (nom, _) = (Path.GetFileNameWithoutExtension(fichier), Path.GetExtension(fichier));
```

2. **Simplifier les appels de méthode avec de nombreux paramètres out** :

```csharp
string date = "2023-05-15";
if (DateTime.TryParse(date, out var dateTime))
{
    // On n'a besoin que de l'année et du mois
    var (annee, mois, _) = (dateTime.Year, dateTime.Month, dateTime.Day);
    Console.WriteLine($"Année: {annee}, Mois: {mois}");
}
```

3. **Utilisation dans les expressions switch pour C# 8+** :

```csharp
public string ClassifierNombre(int nombre) => nombre switch
{
    < 0 => "Négatif",
    0 => "Zéro",
    _ => "Positif"  // Cas par défaut avec discard
};
```

### Résumé des discards

Les discards sont un moyen élégant de:
- Indiquer qu'une valeur retournée n'est pas pertinente
- Améliorer la lisibilité du code en ignorant explicitement les valeurs inutilisées
- Éviter d'avoir à créer des variables temporaires que vous n'utiliserez pas

## Bonnes pratiques pour les paramètres et valeurs de retour

1. **Utilisez les types les plus appropriés** :
   - `ref` : pour modifier une variable existante
   - `out` : pour retourner plusieurs valeurs quand vous avez besoin d'un statut de retour
   - `params` : pour des méthodes avec un nombre variable d'arguments
   - Tuples : pour des méthodes retournant plusieurs valeurs associées
   - Discards : pour ignorer des valeurs dont vous n'avez pas besoin

2. **Nommez explicitement vos tuples** pour améliorer la lisibilité :
   ```csharp
   // Préférez ceci...
   public (int Quotient, int Reste) Diviser(int a, int b)

   // ...à ceci
   public (int, int) Diviser(int a, int b)
   ```

3. **Privilégiez les tuples aux paramètres `out` multiples** pour une meilleure lisibilité :
   ```csharp
   // Préférez ceci...
   public (bool Succes, int Resultat, string Message) Traiter(string input)

   // ...à ceci
   public bool Traiter(string input, out int resultat, out string message)
   ```

4. **Utilisez `params` avec modération** : pratique pour les API utilisateur mais peut masquer la complexité

5. **Documentez vos méthodes** avec des commentaires XML pour expliquer clairement le rôle des paramètres et des valeurs de retour :
   ```csharp
   /// <summary>
   /// Divise deux nombres et retourne le quotient et le reste.
   /// </summary>
   /// <param name="dividende">Le nombre à diviser</param>
   /// <param name="diviseur">Le diviseur</param>
   /// <returns>Un tuple contenant le quotient et le reste de la division</returns>
   /// <exception cref="DivideByZeroException">Si le diviseur est zéro</exception>
   public (int Quotient, int Reste) Diviser(int dividende, int diviseur)
   ```

La maîtrise de ces différentes techniques de gestion des paramètres et des valeurs de retour vous permettra d'écrire un code C# plus expressif, plus flexible et plus maintenable.

⏭️
