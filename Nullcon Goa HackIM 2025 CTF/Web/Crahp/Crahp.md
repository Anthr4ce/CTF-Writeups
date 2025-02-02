# **Web Challenge - CRC Hash Collision Exploit**

---

## **Challenge Description**

Dans ce challenge, nous avons accès au code source d'un script PHP ainsi qu'à deux indices :

- **Indice 1** : [Lien vers la documentation PHP sur crc32()](https://www.php.net/manual/en/function.crc32.php#28012)
- **Indice 2** : [Lien vers une implémentation CRC8 en PHP](https://stackoverflow.com/questions/507041/crc8-check-in-php/73305496#73305496)

L'objectif est d'obtenir le **flag** en contournant un mécanisme d'authentification basé sur des **hashes CRC**.

### **📌 CRC, CRC16 et CRC8**

🔹 **CRC (Cyclic Redundancy Check)**  
Le **CRC** est une méthode de détection d'erreurs utilisée principalement pour vérifier l'intégrité des données dans les transmissions et le stockage. Il **n'est pas cryptographiquement sécurisé**, ce qui signifie qu’il est vulnérable aux collisions.

🔹 **CRC16 et CRC8**  
Ce sont des **variantes du CRC** qui utilisent **16 bits (CRC16)** et **8 bits (CRC8)** pour produire une somme de contrôle.

- **CRC16** génère une valeur entre **0x0000 et 0xFFFF** (65 536 possibilités).
- **CRC8** génère une valeur entre **0x00 et 0xFF** (256 possibilités).

🔹 **Pourquoi c'est faible ?**  
Contrairement aux vrais hash cryptographiques comme **SHA-256**, le CRC est **très facile à casser** car il n'est pas conçu pour la sécurité. Deux entrées différentes peuvent souvent produire le **même CRC**, ce qui permet d'exploiter des **collisions**.

✅ **En résumé :**  
CRC est une **somme de contrôle rapide mais non sécurisée** utilisée pour **détecter** des erreurs, pas pour la **protection des données**.

---

## **Analyse du Code Source**

Voici les parties importantes du code PHP fournies dans le challenge :

### **1️⃣ Vérification de l'authentification**

Le script vérifie si l'utilisateur soumet un mot de passe via un formulaire POST :
```php
<?php
    if(isset($_POST['password']) && strlen($MYPASSWORD) == strlen($_POST['password'])) {
        $pwhash1 = crc16($MYPASSWORD);
        $pwhash2 = crc8($MYPASSWORD);

        $password = $_POST['password'];
        $pwhash3 = crc16($password);
        $pwhash4 = crc8($password);

        if($MYPASSWORD == $password) {
            die("oops. Try harder!");
        }
        if($pwhash1 != $pwhash3) {
            die("Oops. Nope. Try harder!");
        }
        if($pwhash2 != $pwhash4) {
            die("OoOps. Not quite. Try harder!");
        }
        $access = true;
    
        if($access) {
            echo "You win a flag: $FLAG";
        } else {
            echo "Denied! :-(";
        }
        } else {
        echo "Try harder!";
        }
?>
```

### **2️⃣ Points clés à retenir**

- **Le mot de passe original est `AdM1nP@assW0rd!`**.
- **Nous ne pouvons pas soumettre ce même mot de passe directement** (`if($MYPASSWORD == $password)`).
- **La validation se fait via deux fonctions CRC (`crc16` et `crc8`)** :
    - `crc16($MYPASSWORD) == crc16($password)`
    - `crc8($MYPASSWORD) == crc8($password)`
- **Si ces deux valeurs sont identiques, nous obtenons le flag !**

## **Exploitation : Collision CRC**

Les fonctions **CRC16 et CRC8** ne sont **pas cryptographiquement sécurisées**. Il est possible de trouver une **collision** (un mot de passe différent générant les mêmes valeurs CRC).

### **🎯 Objectif**

Trouver un mot de passe **différent** de `AdM1nP@assW0rd!` qui a **la même valeur CRC16 et CRC8**.

### **Étapes de l’Exploitation**

Nous allons utiliser **un script Python** pour :

1. **Calculer les valeurs CRC16 et CRC8**.

```python
from pwn import *
import itertools
import string
import multiprocessing

def crc16(data: str):
    crc = 0xFFFF
    for char in data:
        crc ^= ord(char)
        for _ in range(8):
            if crc & 1:
                crc = (crc >> 1) ^ 0xA001
            else:
                crc >>= 1
    return crc

def crc8(data: str):
    crc8Table = [
    0x00, 0x07, 0x0E, 0x09, 0x1C, 0x1B, 0x12, 0x15,
    0x38, 0x3F, 0x36, 0x31, 0x24, 0x23, 0x2A, 0x2D,
    0x70, 0x77, 0x7E, 0x79, 0x6C, 0x6B, 0x62, 0x65,
    0x48, 0x4F, 0x46, 0x41, 0x54, 0x53, 0x5A, 0x5D,
    0xE0, 0xE7, 0xEE, 0xE9, 0xFC, 0xFB, 0xF2, 0xF5,
    0xD8, 0xDF, 0xD6, 0xD1, 0xC4, 0xC3, 0xCA, 0xCD,
    0x90, 0x97, 0x9E, 0x99, 0x8C, 0x8B, 0x82, 0x85,
    0xA8, 0xAF, 0xA6, 0xA1, 0xB4, 0xB3, 0xBA, 0xBD,
    0xC7, 0xC0, 0xC9, 0xCE, 0xDB, 0xDC, 0xD5, 0xD2,
    0xFF, 0xF8, 0xF1, 0xF6, 0xE3, 0xE4, 0xED, 0xEA,
    0xB7, 0xB0, 0xB9, 0xBE, 0xAB, 0xAC, 0xA5, 0xA2,
    0x8F, 0x88, 0x81, 0x86, 0x93, 0x94, 0x9D, 0x9A,
    0x27, 0x20, 0x29, 0x2E, 0x3B, 0x3C, 0x35, 0x32,
    0x1F, 0x18, 0x11, 0x16, 0x03, 0x04, 0x0D, 0x0A,
    0x57, 0x50, 0x59, 0x5E, 0x4B, 0x4C, 0x45, 0x42,
    0x6F, 0x68, 0x61, 0x66, 0x73, 0x74, 0x7D, 0x7A,
    0x89, 0x8E, 0x87, 0x80, 0x95, 0x92, 0x9B, 0x9C,
    0xB1, 0xB6, 0xBF, 0xB8, 0xAD, 0xAA, 0xA3, 0xA4,
    0xF9, 0xFE, 0xF7, 0xF0, 0xE5, 0xE2, 0xEB, 0xEC,
    0xC1, 0xC6, 0xCF, 0xC8, 0xDD, 0xDA, 0xD3, 0xD4,
    0x69, 0x6E, 0x67, 0x60, 0x75, 0x72, 0x7B, 0x7C,
    0x51, 0x56, 0x5F, 0x58, 0x4D, 0x4A, 0x43, 0x44,
    0x19, 0x1E, 0x17, 0x10, 0x05, 0x02, 0x0B, 0x0C,
    0x21, 0x26, 0x2F, 0x28, 0x3D, 0x3A, 0x33, 0x34,
    0x4E, 0x49, 0x40, 0x47, 0x52, 0x55, 0x5C, 0x5B,
    0x76, 0x71, 0x78, 0x7F, 0x6A, 0x6D, 0x64, 0x63,
    0x3E, 0x39, 0x30, 0x37, 0x22, 0x25, 0x2C, 0x2B,
    0x06, 0x01, 0x08, 0x0F, 0x1A, 0x1D, 0x14, 0x13,
    0xAE, 0xA9, 0xA0, 0xA7, 0xB2, 0xB5, 0xBC, 0xBB,
    0x96, 0x91, 0x98, 0x9F, 0x8A, 0x8D, 0x84, 0x83,
    0xDE, 0xD9, 0xD0, 0xD7, 0xC2, 0xC5, 0xCC, 0xCB,
    0xE6, 0xE1, 0xE8, 0xEF, 0xFA, 0xFD, 0xF4, 0xF3
    ]

    crc = 0
    for char in data:
        crc = crc8Table[(crc ^ ord(char)) & 0xFF]
    return crc & 0xFF
```
2. **Génération de la collision** et **vérifier si elles produisent la même valeur CRC**.

```php
original_password = "AdM1nP@assW0rd!"
crc16_target = crc16(original_password)
crc8_target = crc8(original_password)
```

3. **Optimisation : Multi-threading**

Le bruteforce peut être très lent.
J'ai donc utilisé PwnTools et multiprocessing pour accélérer la recherche en répartissant les calculs sur plusieurs cœurs avec un offset.

```python
original_password = "AdM1nP@assW0rd!"
crc16_target = crc16(original_password)
crc8_target = crc8(original_password)

#char to test
charset = string.ascii_letters + string.digits + string.punctuation
password_length = len(original_password)

#multi threading
def brute_force(start_idx):
    #offset pour aller plus vite
    for attempt in itertools.islice(itertools.product(charset, repeat=password_length), start_idx, None, multiprocessing.cpu_count()):
        test_password = ''.join(attempt)

        if test_password != original_password and crc16(test_password) == crc16_target and crc8(test_password) == crc8_target:
            print(f"lmaoo got a lmaoollision: {test_password}")
            return

if __name__ == "__main__":
    num_cores = multiprocessing.cpu_count()  
    pool = multiprocessing.Pool(processes=num_cores)
    pool.map(brute_force, range(num_cores))
```

### **Mot de passe trouvé**
Après exécution, le script a trouvé un mot de passe **différent** de `AdM1nP@assW0rd!` mais avec les **mêmes valeurs CRC16 et CRC8**.

![Screenshot](images/mdp_found.png)

Et la flag:

![Screenshot](images/flag_found.png)


## **Conclusion**

🔹 **Le problème :** L'utilisation de **CRC16 et CRC8 pour la validation d'un mot de passe** est une très mauvaise pratique car les CRC **ne sont pas résistants aux collisions**.  
🔹 **L'attaque :** Nous avons utilisé **un script Python en multi-threading** pour trouver une **collision CRC** et contourner l'authentification.  
🔹 **Le résultat :** En soumettant un mot de passe différent mais avec la même empreinte CRC, nous avons réussi à **récupérer le flag** !

Pour éviter ce genre de vulnérabilité, il faut **toujours utiliser des fonctions de hachage cryptographiques comme SHA256 ou bcrypt**, et **éviter les CRC pour la sécurité**.
