# 16.3. Cryptographie

🔝 Retour au [Sommaire](/SOMMAIRE.md)

La cryptographie est un élément fondamental de la sécurité informatique. Elle permet de protéger les données sensibles contre les accès non autorisés. Dans ce chapitre, nous explorerons les concepts de base et l'implémentation de la cryptographie en C#.

## 16.3.1. Hachage et salage

### Comprendre le hachage

Le hachage est un processus à sens unique qui transforme des données de taille variable en une chaîne de caractères de taille fixe. Contrairement au chiffrement, le hachage ne peut pas être inversé : vous ne pouvez pas retrouver les données d'origine à partir du hachage.

**Caractéristiques principales du hachage :**
- Même entrée → même sortie (déterministe)
- Petite modification de l'entrée → résultat complètement différent
- Impossible (en théorie) de retrouver l'entrée à partir du hachage
- Faible probabilité de collision (deux entrées différentes générant le même hachage)

### Algorithmes de hachage courants

Voici quelques algorithmes de hachage disponibles en C# :

1. **MD5** (Non recommandé pour la sécurité, considéré comme obsolète)
2. **SHA-1** (Non recommandé pour la sécurité)
3. **SHA-256** (Recommandé pour usages généraux)
4. **SHA-384 et SHA-512** (Recommandés pour une sécurité renforcée)
5. **PBKDF2, Argon2id, BCrypt** (Spécialement conçus pour les mots de passe)

### Exemple de hachage simple avec SHA-256

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

public static string ComputeSha256Hash(string rawData)
{
    // Créer un objet SHA256
    using (SHA256 sha256Hash = SHA256.Create())
    {
        // Calculer le hachage à partir des données brutes
        byte[] bytes = sha256Hash.ComputeHash(Encoding.UTF8.GetBytes(rawData));

        // Convertir le tableau de bytes en chaîne hexadécimale
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < bytes.Length; i++)
        {
            builder.Append(bytes[i].ToString("x2")); // "x2" donne une représentation hexadécimale
        }

        return builder.ToString();
    }
}

// Utilisation
string monTexte = "Bonjour, monde!";
string hashResultat = ComputeSha256Hash(monTexte);
Console.WriteLine($"Texte original: {monTexte}");
Console.WriteLine($"Hachage SHA-256: {hashResultat}");
```

### Pourquoi le salage est-il nécessaire ?

Le hachage simple présente des vulnérabilités, notamment face aux attaques par dictionnaire et par tables arc-en-ciel. Ces attaques consistent à pré-calculer les hachages de mots courants pour les comparer aux hachages stockés.

Le **salage** ajoute une valeur aléatoire (le sel) aux données avant de les hacher. Ainsi, même si deux utilisateurs ont le même mot de passe, leurs hachages stockés seront différents.

### Hachage et salage de mots de passe

```csharp
using System;
using System.Security.Cryptography;

public class PasswordHasher
{
    // Taille recommandée pour le sel (16 octets = 128 bits)
    private const int SaltSize = 16;

    // Nombre d'itérations pour PBKDF2 (min. recommandé: 10000)
    private const int Iterations = 10000;

    // Taille de la clé dérivée (32 octets = 256 bits)
    private const int KeySize = 32;

    public static string HashPassword(string password)
    {
        // Générer un sel aléatoire
        byte[] salt = new byte[SaltSize];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(salt);
        }

        // Dériver une clé à partir du mot de passe et du sel avec PBKDF2
        byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
            password,
            salt,
            Iterations,
            HashAlgorithmName.SHA256,
            KeySize);

        // Combiner le sel et le hachage dans un format stockable
        byte[] hashBytes = new byte[SaltSize + KeySize];
        Array.Copy(salt, 0, hashBytes, 0, SaltSize);
        Array.Copy(hash, 0, hashBytes, SaltSize, KeySize);

        // Convertir en Base64 pour un stockage facile
        return Convert.ToBase64String(hashBytes);
    }

    public static bool VerifyPassword(string hashedPassword, string password)
    {
        // Décoder le hachage stocké
        byte[] hashBytes = Convert.FromBase64String(hashedPassword);

        // Extraire le sel (les premiers SaltSize octets)
        byte[] salt = new byte[SaltSize];
        Array.Copy(hashBytes, 0, salt, 0, SaltSize);

        // Recalculer le hachage avec le mot de passe fourni et le sel extrait
        byte[] expectedHash = Rfc2898DeriveBytes.Pbkdf2(
            password,
            salt,
            Iterations,
            HashAlgorithmName.SHA256,
            KeySize);

        // Comparer le hachage calculé avec celui stocké
        for (int i = 0; i < KeySize; i++)
        {
            if (hashBytes[i + SaltSize] != expectedHash[i])
                return false;
        }

        return true;
    }
}

// Utilisation
string password = "MotDePasse123!";
string hashedPassword = PasswordHasher.HashPassword(password);
Console.WriteLine($"Mot de passe haché: {hashedPassword}");

bool isValid = PasswordHasher.VerifyPassword(hashedPassword, password);
Console.WriteLine($"Vérification: {isValid}"); // Doit afficher True

bool isInvalid = PasswordHasher.VerifyPassword(hashedPassword, "MauvaisMotDePasse");
Console.WriteLine($"Vérification incorrecte: {isInvalid}"); // Doit afficher False
```

### Utilisation de l'API moderne Identity

ASP.NET Core Identity fournit une implémentation robuste pour le hachage des mots de passe :

```csharp
using Microsoft.AspNetCore.Identity;

// À injecter via DI dans ASP.NET Core
private readonly PasswordHasher<ApplicationUser> _passwordHasher;

// Dans le constructeur
public UserService(PasswordHasher<ApplicationUser> passwordHasher)
{
    _passwordHasher = passwordHasher;
}

// Hachage d'un mot de passe
public string HashPassword(ApplicationUser user, string password)
{
    return _passwordHasher.HashPassword(user, password);
}

// Vérification d'un mot de passe
public bool VerifyPassword(ApplicationUser user, string hashedPassword, string providedPassword)
{
    var result = _passwordHasher.VerifyHashedPassword(user, hashedPassword, providedPassword);
    return result == PasswordVerificationResult.Success;
}
```

## 16.3.2. Chiffrement symétrique

Le chiffrement symétrique utilise la même clé pour chiffrer et déchiffrer les données. C'est généralement rapide et efficace pour le chiffrement de grandes quantités de données.

### Principes du chiffrement symétrique

1. **Une seule clé secrète** est utilisée pour le chiffrement et le déchiffrement
2. Les deux parties doivent connaître et protéger cette clé
3. Généralement plus rapide que le chiffrement asymétrique
4. Le principal défi est l'échange sécurisé de la clé

### Algorithmes de chiffrement symétriques courants

- **AES (Advanced Encryption Standard)** : Standard moderne, très sécurisé, tailles de clé de 128, 192 ou 256 bits
- **Triple DES** : Plus ancien et plus lent qu'AES, mais encore utilisé
- **ChaCha20** : Alternative moderne à AES, particulièrement efficace en logiciel

### Exemple de chiffrement AES en C#

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class AesEncryption
{
    // Taille du vecteur d'initialisation pour AES (16 octets)
    private const int IvSize = 16;

    // Chiffrer une chaîne avec AES
    public static string Encrypt(string plainText, string key)
    {
        // Valider les entrées
        if (string.IsNullOrEmpty(plainText))
            throw new ArgumentNullException(nameof(plainText));
        if (string.IsNullOrEmpty(key))
            throw new ArgumentNullException(nameof(key));

        byte[] iv;
        byte[] encrypted;

        // Créer un objet AES avec la clé dérivée de l'entrée utilisateur
        using (Aes aes = Aes.Create())
        {
            // Dériver une clé de 256 bits à partir de la clé fournie
            // Note: Dans une application réelle, utilisez un sel
            byte[] keyBytes = new byte[32]; // 256 bits
            byte[] passwordBytes = Encoding.UTF8.GetBytes(key);
            Array.Copy(passwordBytes, keyBytes, Math.Min(passwordBytes.Length, keyBytes.Length));

            aes.Key = keyBytes;

            // Générer un IV aléatoire
            aes.GenerateIV();
            iv = aes.IV;

            // Créer un chiffreur
            ICryptoTransform encryptor = aes.CreateEncryptor(aes.Key, aes.IV);

            // Chiffrer les données
            using (MemoryStream ms = new MemoryStream())
            {
                using (CryptoStream cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
                {
                    using (StreamWriter sw = new StreamWriter(cs))
                    {
                        sw.Write(plainText);
                    }
                    encrypted = ms.ToArray();
                }
            }
        }

        // Combiner l'IV et les données chiffrées
        byte[] result = new byte[IvSize + encrypted.Length];
        Array.Copy(iv, 0, result, 0, IvSize);
        Array.Copy(encrypted, 0, result, IvSize, encrypted.Length);

        // Convertir en Base64 pour un stockage facile
        return Convert.ToBase64String(result);
    }

    // Déchiffrer une chaîne avec AES
    public static string Decrypt(string cipherText, string key)
    {
        // Valider les entrées
        if (string.IsNullOrEmpty(cipherText))
            throw new ArgumentNullException(nameof(cipherText));
        if (string.IsNullOrEmpty(key))
            throw new ArgumentNullException(nameof(key));

        byte[] cipherBytes = Convert.FromBase64String(cipherText);

        // Extraire l'IV (les premiers IvSize octets)
        byte[] iv = new byte[IvSize];
        Array.Copy(cipherBytes, 0, iv, 0, IvSize);

        // Extraire les données chiffrées
        byte[] encrypted = new byte[cipherBytes.Length - IvSize];
        Array.Copy(cipherBytes, IvSize, encrypted, 0, encrypted.Length);

        // Déchiffrer les données
        string decrypted;

        // Créer un objet AES avec la même clé
        using (Aes aes = Aes.Create())
        {
            // Dériver la même clé de 256 bits
            byte[] keyBytes = new byte[32]; // 256 bits
            byte[] passwordBytes = Encoding.UTF8.GetBytes(key);
            Array.Copy(passwordBytes, keyBytes, Math.Min(passwordBytes.Length, keyBytes.Length));

            aes.Key = keyBytes;
            aes.IV = iv;

            // Créer un déchiffreur
            ICryptoTransform decryptor = aes.CreateDecryptor(aes.Key, aes.IV);

            // Déchiffrer les données
            using (MemoryStream ms = new MemoryStream(encrypted))
            {
                using (CryptoStream cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read))
                {
                    using (StreamReader sr = new StreamReader(cs))
                    {
                        decrypted = sr.ReadToEnd();
                    }
                }
            }
        }

        return decrypted;
    }
}

// Utilisation
string originalText = "Information confidentielle à protéger";
string key = "Cl3S3cr3t3";

string encrypted = AesEncryption.Encrypt(originalText, key);
Console.WriteLine($"Texte chiffré: {encrypted}");

string decrypted = AesEncryption.Decrypt(encrypted, key);
Console.WriteLine($"Texte déchiffré: {decrypted}");
```

### Chiffrement de fichiers avec AES

```csharp
public class FileEncryption
{
    public static void EncryptFile(string inputFile, string outputFile, string password)
    {
        byte[] salt = new byte[8];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(salt);
        }

        // Dériver la clé et l'IV à partir du mot de passe et du sel
        Rfc2898DeriveBytes key = new Rfc2898DeriveBytes(password, salt, 10000);

        using (FileStream fsCrypt = new FileStream(outputFile, FileMode.Create))
        {
            // Écrire le sel au début du fichier
            fsCrypt.Write(salt, 0, salt.Length);

            using (Aes aes = Aes.Create())
            {
                aes.Key = key.GetBytes(32); // 256 bits
                aes.IV = key.GetBytes(16);  // 128 bits

                using (CryptoStream cs = new CryptoStream(fsCrypt, aes.CreateEncryptor(), CryptoStreamMode.Write))
                {
                    using (FileStream fsIn = new FileStream(inputFile, FileMode.Open))
                    {
                        byte[] buffer = new byte[4096];
                        int read;

                        while ((read = fsIn.Read(buffer, 0, buffer.Length)) > 0)
                        {
                            cs.Write(buffer, 0, read);
                        }
                    }
                }
            }
        }
    }

    public static void DecryptFile(string inputFile, string outputFile, string password)
    {
        byte[] salt = new byte[8];

        using (FileStream fsCrypt = new FileStream(inputFile, FileMode.Open))
        {
            // Lire le sel du début du fichier
            fsCrypt.Read(salt, 0, salt.Length);

            // Dériver la même clé et IV
            Rfc2898DeriveBytes key = new Rfc2898DeriveBytes(password, salt, 10000);

            using (Aes aes = Aes.Create())
            {
                aes.Key = key.GetBytes(32);
                aes.IV = key.GetBytes(16);

                using (CryptoStream cs = new CryptoStream(fsCrypt, aes.CreateDecryptor(), CryptoStreamMode.Read))
                {
                    using (FileStream fsOut = new FileStream(outputFile, FileMode.Create))
                    {
                        byte[] buffer = new byte[4096];
                        int read;

                        while ((read = cs.Read(buffer, 0, buffer.Length)) > 0)
                        {
                            fsOut.Write(buffer, 0, read);
                        }
                    }
                }
            }
        }
    }
}

// Utilisation
FileEncryption.EncryptFile("document.docx", "document.encrypted", "MotDePasseSecurise");
FileEncryption.DecryptFile("document.encrypted", "document_recovered.docx", "MotDePasseSecurise");
```

## 16.3.3. Chiffrement asymétrique

Le chiffrement asymétrique utilise une paire de clés mathématiquement liées : une clé publique pour chiffrer et une clé privée pour déchiffrer. Il est particulièrement utile pour l'échange sécurisé de clés et les signatures numériques.

### Principes du chiffrement asymétrique

1. **Deux clés différentes** : une publique et une privée
2. Les données chiffrées avec la clé publique ne peuvent être déchiffrées qu'avec la clé privée
3. La clé publique peut être partagée librement sans compromettre la sécurité
4. Plus lent que le chiffrement symétrique, donc généralement utilisé pour les petites quantités de données
5. Souvent utilisé pour échanger des clés symétriques de manière sécurisée

### Algorithmes asymétriques courants

- **RSA** : Standard largement utilisé, basé sur la factorisation de grands nombres premiers
- **ECC (Elliptic Curve Cryptography)** : Utilise des clés plus petites que RSA pour une sécurité équivalente
- **DSA (Digital Signature Algorithm)** : Spécifiquement conçu pour les signatures numériques

### Exemple de chiffrement RSA en C#

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

public class RsaEncryption
{
    // Générer une nouvelle paire de clés RSA
    public static RSA GenerateKeys()
    {
        RSA rsa = RSA.Create(2048); // Taille de clé recommandée : au moins 2048 bits
        return rsa;
    }

    // Obtenir la clé publique au format XML
    public static string GetPublicKey(RSA rsa)
    {
        return rsa.ToXmlString(false); // false = seulement la clé publique
    }

    // Obtenir la clé privée au format XML (à conserver en lieu sûr!)
    public static string GetPrivateKey(RSA rsa)
    {
        return rsa.ToXmlString(true); // true = inclut la clé privée
    }

    // Chiffrer des données avec la clé publique
    public static byte[] Encrypt(string publicKey, string data)
    {
        using (RSA rsa = RSA.Create())
        {
            rsa.FromXmlString(publicKey);

            byte[] dataBytes = Encoding.UTF8.GetBytes(data);

            // Chiffrement RSA avec padding OAEP (recommandé)
            return rsa.Encrypt(dataBytes, RSAEncryptionPadding.OaepSHA256);
        }
    }

    // Déchiffrer des données avec la clé privée
    public static string Decrypt(string privateKey, byte[] encryptedData)
    {
        using (RSA rsa = RSA.Create())
        {
            rsa.FromXmlString(privateKey);

            byte[] decryptedData = rsa.Decrypt(encryptedData, RSAEncryptionPadding.OaepSHA256);

            return Encoding.UTF8.GetString(decryptedData);
        }
    }
}

// Utilisation
// Générer des clés
RSA rsaKeys = RsaEncryption.GenerateKeys();
string publicKey = RsaEncryption.GetPublicKey(rsaKeys);
string privateKey = RsaEncryption.GetPrivateKey(rsaKeys);

Console.WriteLine("Clé publique : " + publicKey);
Console.WriteLine("Clé privée : " + privateKey);

// Chiffrer un message
string originalMessage = "Message secret à chiffrer";
byte[] encryptedData = RsaEncryption.Encrypt(publicKey, originalMessage);
Console.WriteLine("Message chiffré (Base64) : " + Convert.ToBase64String(encryptedData));

// Déchiffrer le message
string decryptedMessage = RsaEncryption.Decrypt(privateKey, encryptedData);
Console.WriteLine("Message déchiffré : " + decryptedMessage);
```

### Signature numérique avec RSA

Les signatures numériques permettent de vérifier l'authenticité et l'intégrité des données. Le processus implique :

1. Créer un condensé (hachage) des données
2. Chiffrer ce condensé avec la clé privée pour créer la signature
3. Distribuer la signature avec les données
4. Les destinataires vérifient la signature en la déchiffrant avec la clé publique et en comparant avec le condensé recalculé

```csharp
public class DigitalSignature
{
    // Signer des données avec une clé privée RSA
    public static byte[] SignData(string privateKey, byte[] dataToSign)
    {
        using (RSA rsa = RSA.Create())
        {
            rsa.FromXmlString(privateKey);

            // Créer une signature avec SHA-256
            return rsa.SignData(dataToSign, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
        }
    }

    // Vérifier une signature avec la clé publique
    public static bool VerifySignature(string publicKey, byte[] data, byte[] signature)
    {
        using (RSA rsa = RSA.Create())
        {
            rsa.FromXmlString(publicKey);

            // Vérifier la signature
            return rsa.VerifyData(data, signature, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
        }
    }
}

// Utilisation
// Supposons que nous avons déjà généré la paire de clés
RSA rsaKeys = RsaEncryption.GenerateKeys();
string publicKey = RsaEncryption.GetPublicKey(rsaKeys);
string privateKey = RsaEncryption.GetPrivateKey(rsaKeys);

// Document à signer
string document = "Ceci est un document important qui ne doit pas être modifié.";
byte[] documentBytes = Encoding.UTF8.GetBytes(document);

// Signer le document
byte[] signature = DigitalSignature.SignData(privateKey, documentBytes);
Console.WriteLine("Signature générée (Base64) : " + Convert.ToBase64String(signature));

// Vérifier la signature (utilisateur avec la clé publique)
bool isValid = DigitalSignature.VerifySignature(publicKey, documentBytes, signature);
Console.WriteLine("Signature valide : " + isValid); // Devrait être True

// Essayons de valider un document modifié
string modifiedDocument = "Ceci est un document important qui a été modifié.";
byte[] modifiedBytes = Encoding.UTF8.GetBytes(modifiedDocument);
bool isValidModified = DigitalSignature.VerifySignature(publicKey, modifiedBytes, signature);
Console.WriteLine("Signature valide pour document modifié : " + isValidModified); // Devrait être False
```

## 16.3.4. API de cryptographie .NET

.NET fournit un ensemble complet d'API cryptographiques dans l'espace de noms `System.Security.Cryptography`. Voici un aperçu des principales classes et fonctionnalités :

### Catégories principales

1. **Algorithmes de hachage**
   - `HashAlgorithm` : Classe de base abstraite
   - Implémentations concrètes : `MD5`, `SHA1`, `SHA256`, `SHA384`, `SHA512`

2. **Algorithmes de chiffrement symétrique**
   - `SymmetricAlgorithm` : Classe de base abstraite
   - Implémentations concrètes : `Aes`, `DES`, `TripleDES`, `RC2`

3. **Algorithmes de chiffrement asymétrique**
   - `AsymmetricAlgorithm` : Classe de base abstraite
   - Implémentations concrètes : `RSA`, `DSA`, `ECDsa`, `ECDiffieHellman`

4. **Fonctions de dérivation de clé**
   - `Rfc2898DeriveBytes` : Implémentation de PBKDF2

5. **Générateurs de nombres aléatoires cryptographiques**
   - `RandomNumberGenerator` : Génération de nombres aléatoires sécurisés

### API moderne pour le chiffrement symétrique

À partir de .NET Core, Microsoft a introduit des API plus simples et plus sécurisées par défaut :

```csharp
// Utilisation des API modernes d'AES
public static byte[] EncryptDataModern(byte[] data, byte[] key)
{
    // Générer un nonce aléatoire (AES-GCM recommande 96 bits/12 octets)
    byte[] nonce = new byte[12];
    using (RandomNumberGenerator rng = RandomNumberGenerator.Create())
    {
        rng.GetBytes(nonce);
    }

    // Encrypting the data
    byte[] ciphertext = new byte[data.Length];
    byte[] tag = new byte[16]; // Authentication tag (16 octets pour AES-GCM)

    using (AesGcm aesGcm = new AesGcm(key))
    {
        aesGcm.Encrypt(nonce, data, ciphertext, tag);
    }

    // Combiner nonce + tag + ciphertext
    byte[] result = new byte[nonce.Length + tag.Length + ciphertext.Length];
    Buffer.BlockCopy(nonce, 0, result, 0, nonce.Length);
    Buffer.BlockCopy(tag, 0, result, nonce.Length, tag.Length);
    Buffer.BlockCopy(ciphertext, 0, result, nonce.Length + tag.Length, ciphertext.Length);

    return result;
}

public static byte[] DecryptDataModern(byte[] encryptedData, byte[] key)
{
    // Extraire nonce, tag et ciphertext
    byte[] nonce = new byte[12];
    byte[] tag = new byte[16];
    byte[] ciphertext = new byte[encryptedData.Length - nonce.Length - tag.Length];

    Buffer.BlockCopy(encryptedData, 0, nonce, 0, nonce.Length);
    Buffer.BlockCopy(encryptedData, nonce.Length, tag, 0, tag.Length);
    Buffer.BlockCopy(encryptedData, nonce.Length + tag.Length, ciphertext, 0, ciphertext.Length);

    byte[] plaintext = new byte[ciphertext.Length];

    using (AesGcm aesGcm = new AesGcm(key))
    {
        aesGcm.Decrypt(nonce, ciphertext, tag, plaintext);
    }

    return plaintext;
}
```

### Classe Key Management

Pour les applications plus complexes, .NET offre des API de gestion de clés :

```csharp
using System.Security.Cryptography;
using System.Security.Cryptography.X509Certificates;

// Stockage de clés protégées par certificat
public class KeyManagement
{
    // Protéger une clé symétrique avec un certificat
    public static byte[] ProtectKey(byte[] key, X509Certificate2 certificate)
    {
        using (RSA rsa = certificate.GetRSAPublicKey())
        {
            return rsa.Encrypt(key, RSAEncryptionPadding.OaepSHA256);
        }
    }

    // Récupérer une clé symétrique protégée
    public static byte[] UnprotectKey(byte[] protectedKey, X509Certificate2 certificate)
    {
        using (RSA rsa = certificate.GetRSAPrivateKey())
        {
            return rsa.Decrypt(protectedKey, RSAEncryptionPadding.OaepSHA256);
        }
    }

    // Générer une nouvelle clé AES sécurisée
    public static byte[] GenerateAesKey(int keySizeInBits = 256)
    {
        using (Aes aes = Aes.Create())
        {
            aes.KeySize = keySizeInBits;
            aes.GenerateKey();
            return aes.Key;
        }
    }
}
```

### Data Protection API

ASP.NET Core fournit une API de protection des données simplifiée pour les scénarios courants :

```csharp
// Dans Startup.cs ou Program.cs
services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\keys"))
    .SetDefaultKeyLifetime(TimeSpan.FromDays(14));

// Injection dans les contrôleurs ou services
public class SecureService
{
    private readonly IDataProtector _protector;

    public SecureService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("MyApp.SecureData");
    }

    public string ProtectData(string data)
    {
        return _protector.Protect(data);
    }

    public string UnprotectData(string protectedData)
    {
        return _protector.Unprotect(protectedData);
    }
}
```

## 16.3.5. Bonnes pratiques en cryptographie

La cryptographie est complexe, et de petites erreurs peuvent compromettre gravement la sécurité. Voici quelques pratiques recommandées :

### 1. Choix des algorithmes

- ✅ **Utilisez des algorithmes standardisés et éprouvés** :
  - Hachage : SHA-256, SHA-512, PBKDF2, Argon2id
  - Chiffrement symétrique : AES (mode GCM de préférence)
  - Chiffrement asymétrique : RSA (>= 2048 bits), ECC (>= 256 bits)

- ❌ **Évitez les algorithmes obsolètes** :
  - MD5, SHA-1 (pour les applications de sécurité)
  - DES, 3DES, RC4
  - RSA avec des clés inférieures à 2048 bits

### 2. Gestion des clés

- ✅ **Bonne gestion des clés** :
  - Générez toujours les clés et les vecteurs d'initialisation avec un générateur de nombres aléatoires cryptographiques
  - Stockez les clés de manière sécurisée (coffres-forts de clés, TPM, HSM)
  - Implémentez la rotation des clés
  - Utilisez des clés différentes pour des contextes différents

```csharp
// Bonne pratique - Génération sécurisée d'une clé AES
byte[] GenerateRandomKey(int keySizeInBytes)
{
    byte[] key = new byte[keySizeInBytes];
    using (var rng = RandomNumberGenerator.Create())
    {
        rng.GetBytes(key);
    }
    return key;
}

// Bonne pratique - Génération sécurisée d'un IV
byte[] GenerateRandomIv(int blockSizeInBytes)
{
    byte[] iv = new byte[blockSizeInBytes];
    using (var rng = RandomNumberGenerator.Create())
    {
        rng.GetBytes(iv);
    }
    return iv;
}
```

- ❌ **À éviter** :
  - Utiliser des clés codées en dur dans le code source
  - Utiliser des clés faibles ou dérivées de mots de passe faibles
  - Réutiliser les mêmes clés pour différentes opérations
  - Utiliser des valeurs constantes comme vecteurs d'initialisation

```csharp
// À ÉVITER - Clé et IV codés en dur
// Ce code est vulnérable!
private static readonly byte[] HardcodedKey = new byte[]
    { 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x10 };
private static readonly byte[] HardcodedIv = new byte[]
    { 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x10 };
```

### 3. Mise en œuvre sécurisée

- ✅ **Bonnes pratiques d'implémentation** :
  - Utilisez des bibliothèques cryptographiques établies plutôt que d'implémenter vos propres algorithmes
  - Utilisez des modes de chiffrement qui offrent une authentification (comme GCM ou EAX)
  - Validez les entrées avant le chiffrement
  - Gérez correctement les erreurs sans divulguer d'informations sensibles

```csharp
// Bonne pratique - Utilisation de AES-GCM pour l'authentification intégrée
public static (byte[] ciphertext, byte[] tag, byte[] nonce) EncryptWithAuthentication(byte[] plaintext, byte[] key)
{
    byte[] nonce = new byte[12]; // 96 bits pour AES-GCM
    byte[] ciphertext = new byte[plaintext.Length];
    byte[] tag = new byte[16]; // 128 bits pour le tag d'authentification

    using (RandomNumberGenerator rng = RandomNumberGenerator.Create())
    {
        rng.GetBytes(nonce);
    }

    using (AesGcm aesGcm = new AesGcm(key))
    {
        aesGcm.Encrypt(nonce, plaintext, ciphertext, tag);
    }

    return (ciphertext, tag, nonce);
}

// Vérification et déchiffrement avec authentification
public static byte[] DecryptWithAuthentication(byte[] ciphertext, byte[] tag, byte[] nonce, byte[] key)
{
    byte[] plaintext = new byte[ciphertext.Length];

    try
    {
        using (AesGcm aesGcm = new AesGcm(key))
        {
            aesGcm.Decrypt(nonce, ciphertext, tag, plaintext);
        }
        return plaintext;
    }
    catch (CryptographicException)
    {
        // Gestion sécurisée des erreurs - ne pas révéler de détails spécifiques
        throw new SecurityException("Échec de la vérification d'authentification ou du déchiffrement");
    }
}
```

- ❌ **À éviter** :
  - Utiliser des modes de chiffrement sans authentification (comme ECB ou CBC sans HMAC)
  - Révéler des détails d'erreur qui pourraient aider un attaquant
  - Réinventer des algorithmes cryptographiques ou les modifier

```csharp
// À ÉVITER - Mode ECB (ne protège pas les modèles)
// Ce code est vulnérable!
public static byte[] EncryptEcb(byte[] data, byte[] key)
{
    using (Aes aes = Aes.Create())
    {
        aes.Mode = CipherMode.ECB; // Mode ECB vulnérable!
        aes.Key = key;

        using (ICryptoTransform encryptor = aes.CreateEncryptor())
        {
            return encryptor.TransformFinalBlock(data, 0, data.Length);
        }
    }
}
```

### 4. Sécurité des mots de passe

- ✅ **Bonnes pratiques pour les mots de passe** :
  - Utilisez des fonctions dédiées au hachage de mots de passe (PBKDF2, Argon2id, bcrypt)
  - Utilisez un sel unique et aléatoire pour chaque mot de passe
  - Utilisez un nombre élevé d'itérations (au moins 10 000 pour PBKDF2)

```csharp
// Bonne pratique - Hachage de mot de passe moderne avec Argon2id
// Nécessite le package NuGet "Konscious.Security.Cryptography.Argon2"
public static string HashPasswordWithArgon2id(string password)
{
    byte[] salt = new byte[16];
    using (var rng = RandomNumberGenerator.Create())
    {
        rng.GetBytes(salt);
    }

    // Paramètres recommandés pour Argon2id
    int iterations = 4; // Nombre de passes
    int memorySize = 65536; // 64 MB
    int parallelism = 2; // Degré de parallélisme

    using (var argon2 = new Argon2id(Encoding.UTF8.GetBytes(password)))
    {
        argon2.Salt = salt;
        argon2.Iterations = iterations;
        argon2.MemorySize = memorySize;
        argon2.DegreeOfParallelism = parallelism;

        byte[] hash = argon2.GetBytes(32); // 256 bits

        // Format: $argon2id$v=19$m=65536,t=4,p=2$BASE64SALT$BASE64HASH
        return $"$argon2id$v=19$m={memorySize},t={iterations},p={parallelism}${Convert.ToBase64String(salt)}${Convert.ToBase64String(hash)}";
    }
}

// Vérification du mot de passe avec Argon2id
public static bool VerifyPasswordWithArgon2id(string storedHash, string password)
{
    try
    {
        // Parser le hachage stocké
        var parts = storedHash.Split('$');
        if (parts.Length != 6)
            return false;

        // Extraire le sel et les paramètres
        var algoParams = parts[3].Split(',');
        int memorySize = int.Parse(algoParams[0].Substring(2));
        int iterations = int.Parse(algoParams[1].Substring(2));
        int parallelism = int.Parse(algoParams[2].Substring(2));

        byte[] salt = Convert.FromBase64String(parts[4]);
        byte[] expectedHash = Convert.FromBase64String(parts[5]);

        // Recalculer le hachage avec les mêmes paramètres
        using (var argon2 = new Argon2id(Encoding.UTF8.GetBytes(password)))
        {
            argon2.Salt = salt;
            argon2.Iterations = iterations;
            argon2.MemorySize = memorySize;
            argon2.DegreeOfParallelism = parallelism;

            byte[] computedHash = argon2.GetBytes(expectedHash.Length);

            // Comparaison en temps constant pour éviter les attaques temporelles
            return CryptographicOperations.FixedTimeEquals(computedHash, expectedHash);
        }
    }
    catch
    {
        return false;
    }
}
```

- ❌ **À éviter** :
  - Stocker des mots de passe en clair ou avec un simple hachage sans sel
  - Utiliser des algorithmes de hachage rapides (MD5, SHA-1, SHA-256) pour les mots de passe
  - Utiliser le même sel pour tous les mots de passe

### 5. Sécurité des communications

- ✅ **Bonnes pratiques pour les communications** :
  - Utilisez TLS pour toutes les communications sensibles
  - Validez correctement les certificats
  - Utilisez des versions récentes de TLS (1.2 minimum, idéalement 1.3)
  - Mettez en place une épinglage de certificat (certificate pinning) pour les applications critiques

```csharp
// Bonne pratique - Configuration d'un client HTTP sécurisé
public HttpClient CreateSecureHttpClient()
{
    var handler = new HttpClientHandler
    {
        // Exiger des connexions HTTPS valides
        CheckCertificateRevocationList = true,
        SslProtocols = SslProtocols.Tls12 | SslProtocols.Tls13,
        ServerCertificateCustomValidationCallback = (sender, cert, chain, errors) =>
        {
            // Pour épinglage de certificat, vérifiez l'empreinte du certificat
            if (errors == SslPolicyErrors.None)
            {
                // Exemple : Vérifier que l'empreinte du certificat correspond à une valeur connue
                string thumbprint = cert.GetCertHashString();
                return thumbprint == "EXPECTED_THUMBPRINT_HERE";
            }
            return false;
        }
    };

    return new HttpClient(handler);
}
```

- ❌ **À éviter** :
  - Désactiver la validation des certificats
  - Utiliser des protocoles obsolètes (SSL 3.0, TLS 1.0, TLS 1.1)
  - Transmettre des données sensibles en clair

```csharp
// À ÉVITER - Désactivation de la validation des certificats
// Ce code est vulnérable!
var insecureHandler = new HttpClientHandler
{
    ServerCertificateCustomValidationCallback = (sender, cert, chain, errors) => true // DANGER!
};
```

### 6. Attaques par canal auxiliaire

- ✅ **Protections contre les attaques par canal auxiliaire** :
  - Utilisez des comparaisons en temps constant pour les données sensibles
  - Évitez les fuites d'informations via les messages d'erreur
  - Protégez contre les attaques par analyse de temps

```csharp
// Bonne pratique - Comparaison en temps constant
public static bool CompareHashes(byte[] hash1, byte[] hash2)
{
    // Utiliser la méthode intégrée à .NET pour une comparaison en temps constant
    return CryptographicOperations.FixedTimeEquals(hash1, hash2);
}

// Alternative si vous n'avez pas accès à CryptographicOperations
public static bool CompareHashesManual(byte[] hash1, byte[] hash2)
{
    if (hash1 == null || hash2 == null || hash1.Length != hash2.Length)
        return false;

    int result = 0;
    for (int i = 0; i < hash1.Length; i++)
    {
        // Le OR bit à bit accumule les différences sans court-circuit
        result |= hash1[i] ^ hash2[i];
    }

    return result == 0;
}
```

- ❌ **À éviter** :
  - Utiliser des comparaisons standard qui peuvent s'arrêter dès la première différence
  - Révéler des informations sur le temps de traitement ou les motifs d'accès à la mémoire
  - Retourner des messages d'erreur détaillés qui révèlent des informations sur les opérations cryptographiques

```csharp
// À ÉVITER - Comparaison non sécurisée (vulnérable aux attaques temporelles)
// Ce code est vulnérable!
public static bool CompareHashesInsecure(byte[] hash1, byte[] hash2)
{
    if (hash1.Length != hash2.Length)
        return false;

    // Comparaison vulnérable qui peut s'arrêter dès la première différence
    for (int i = 0; i < hash1.Length; i++)
    {
        if (hash1[i] != hash2[i])
            return false;
    }

    return true;
}
```

### 7. Tests et audit

- ✅ **Bonnes pratiques pour les tests et audits** :
  - Faites réviser votre code cryptographique par des experts
  - Effectuez des tests de pénétration réguliers
  - Surveillez les nouvelles vulnérabilités et mettez à jour vos systèmes
  - Documentez vos choix cryptographiques et votre implémentation

- ❌ **À éviter** :
  - Faire confiance à une implémentation sans tests rigoureux
  - Considérer la sécurité comme acquise une fois pour toutes
  - Négliger la formation continue sur les meilleures pratiques de sécurité

### 8. Exemple de mise en œuvre complète et sécurisée

Voici un exemple de classe qui met en œuvre les meilleures pratiques pour le chiffrement de données sensibles :

```csharp
using System;
using System.Security.Cryptography;
using System.Text;
using System.IO;

public class SecureCryptographyService
{
    // Constantes pour AES-GCM
    private const int KeySize = 32; // 256 bits
    private const int NonceSize = 12; // 96 bits
    private const int TagSize = 16; // 128 bits

    // Dérive une clé à partir d'un mot de passe (pour les scénarios basés sur mot de passe)
    public byte[] DeriveKey(string password, byte[] salt)
    {
        byte[] key = new byte[KeySize];
        using (var pbkdf2 = new Rfc2898DeriveBytes(
            password,
            salt,
            100000, // Nombre d'itérations élevé
            HashAlgorithmName.SHA256))
        {
            key = pbkdf2.GetBytes(KeySize);
        }
        return key;
    }

    // Génère une clé cryptographiquement forte
    public byte[] GenerateRandomKey()
    {
        byte[] key = new byte[KeySize];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(key);
        }
        return key;
    }

    // Chiffre des données avec AES-GCM (authentifié)
    public byte[] Encrypt(byte[] plaintext, byte[] key, byte[] associatedData = null)
    {
        // Valider les entrées
        if (plaintext == null || plaintext.Length == 0)
            throw new ArgumentException("Les données à chiffrer ne peuvent pas être vides", nameof(plaintext));

        if (key == null || key.Length != KeySize)
            throw new ArgumentException($"La clé doit être de {KeySize} octets", nameof(key));

        // Générer un nonce aléatoire
        byte[] nonce = new byte[NonceSize];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(nonce);
        }

        // Préparer le résultat : nonce + tag + ciphertext
        byte[] ciphertext = new byte[plaintext.Length];
        byte[] tag = new byte[TagSize];
        byte[] result = new byte[NonceSize + TagSize + plaintext.Length];

        try
        {
            // Chiffrer avec AES-GCM
            using (AesGcm aesGcm = new AesGcm(key))
            {
                aesGcm.Encrypt(
                    nonce,
                    plaintext,
                    ciphertext,
                    tag,
                    associatedData);
            }

            // Construire le résultat: nonce + tag + ciphertext
            Buffer.BlockCopy(nonce, 0, result, 0, NonceSize);
            Buffer.BlockCopy(tag, 0, result, NonceSize, TagSize);
            Buffer.BlockCopy(ciphertext, 0, result, NonceSize + TagSize, ciphertext.Length);

            return result;
        }
        catch (Exception ex)
        {
            // Gérer les erreurs sans révéler de détails spécifiques
            throw new CryptographicException("Échec du chiffrement", ex);
        }
    }

    // Déchiffre des données avec AES-GCM et vérifie l'authenticité
    public byte[] Decrypt(byte[] encryptedData, byte[] key, byte[] associatedData = null)
    {
        // Valider les entrées
        if (encryptedData == null || encryptedData.Length <= NonceSize + TagSize)
            throw new ArgumentException("Les données chiffrées sont invalides", nameof(encryptedData));

        if (key == null || key.Length != KeySize)
            throw new ArgumentException($"La clé doit être de {KeySize} octets", nameof(key));

        try
        {
            // Extraire nonce, tag et ciphertext
            byte[] nonce = new byte[NonceSize];
            byte[] tag = new byte[TagSize];
            int ciphertextLength = encryptedData.Length - NonceSize - TagSize;
            byte[] ciphertext = new byte[ciphertextLength];
            byte[] plaintext = new byte[ciphertextLength];

            Buffer.BlockCopy(encryptedData, 0, nonce, 0, NonceSize);
            Buffer.BlockCopy(encryptedData, NonceSize, tag, 0, TagSize);
            Buffer.BlockCopy(encryptedData, NonceSize + TagSize, ciphertext, 0, ciphertextLength);

            // Déchiffrer et vérifier l'authenticité
            using (AesGcm aesGcm = new AesGcm(key))
            {
                aesGcm.Decrypt(
                    nonce,
                    ciphertext,
                    tag,
                    plaintext,
                    associatedData);
            }

            return plaintext;
        }
        catch (CryptographicException)
        {
            // Gérer les erreurs d'authentification sans préciser la cause exacte
            throw new SecurityException("Échec de la vérification d'authenticité ou du déchiffrement");
        }
        catch (Exception ex)
        {
            // Gérer les autres erreurs sans révéler de détails spécifiques
            throw new CryptographicException("Échec du déchiffrement", ex);
        }
    }

    // Chiffrer une chaîne de texte
    public string EncryptString(string plaintext, byte[] key, byte[] associatedData = null)
    {
        byte[] plaintextBytes = Encoding.UTF8.GetBytes(plaintext);
        byte[] encryptedBytes = Encrypt(plaintextBytes, key, associatedData);
        return Convert.ToBase64String(encryptedBytes);
    }

    // Déchiffrer une chaîne de texte
    public string DecryptString(string encryptedText, byte[] key, byte[] associatedData = null)
    {
        byte[] encryptedBytes = Convert.FromBase64String(encryptedText);
        byte[] plaintextBytes = Decrypt(encryptedBytes, key, associatedData);
        return Encoding.UTF8.GetString(plaintextBytes);
    }

    // Chiffrer un fichier
    public void EncryptFile(string inputFile, string outputFile, byte[] key)
    {
        byte[] associatedData = Encoding.UTF8.GetBytes(Path.GetFileName(inputFile));
        byte[] fileData = File.ReadAllBytes(inputFile);
        byte[] encryptedData = Encrypt(fileData, key, associatedData);
        File.WriteAllBytes(outputFile, encryptedData);
    }

    // Déchiffrer un fichier
    public void DecryptFile(string inputFile, string outputFile, byte[] key)
    {
        byte[] associatedData = Encoding.UTF8.GetBytes(Path.GetFileName(outputFile));
        byte[] encryptedData = File.ReadAllBytes(inputFile);
        byte[] decryptedData = Decrypt(encryptedData, key, associatedData);
        File.WriteAllBytes(outputFile, decryptedData);
    }
}

// Exemple d'utilisation
class Program
{
    static void Main()
    {
        var cryptoService = new SecureCryptographyService();

        // Générer une clé
        byte[] key = cryptoService.GenerateRandomKey();

        // Chiffrer des données
        string secretMessage = "Information sensible à protéger";
        string encrypted = cryptoService.EncryptString(secretMessage, key);
        Console.WriteLine($"Message chiffré: {encrypted}");

        // Déchiffrer des données
        string decrypted = cryptoService.DecryptString(encrypted, key);
        Console.WriteLine($"Message déchiffré: {decrypted}");
    }
}
```

### 9. Résumé des meilleures pratiques

Voici un récapitulatif des points essentiels à retenir :

1. **Utilisez des algorithmes standards** validés par des experts en cryptographie
2. **Ne réinventez pas la roue** - utilisez des bibliothèques éprouvées
3. **Gérez soigneusement les clés** - génération aléatoire, stockage sécurisé, rotation
4. **Utilisez le sel et les facteurs de travail** pour le hachage des mots de passe
5. **Préférez les algorithmes authentifiés** comme AES-GCM pour le chiffrement
6. **Utilisez des longueurs de clé adéquates** selon les standards actuels
7. **Implémentez des protections contre les attaques par canal auxiliaire**
8. **Validez vos implémentations** par des tests et des audits
9. **Restez informé** sur les dernières vulnérabilités et recommandations

En suivant ces bonnes pratiques, vous pouvez considérablement améliorer la sécurité cryptographique de vos applications C#, protégeant ainsi les données sensibles contre les accès non autorisés.

⏭️
