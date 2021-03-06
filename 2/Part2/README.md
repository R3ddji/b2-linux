# TP2 pt. 2 : Maintien en condition opérationnelle

# Sommaire

- [TP2 pt. 2 : Maintien en condition opérationnelle](#tp2-pt-2--maintien-en-condition-opérationnelle)
- [Sommaire](#sommaire)
- [I. Monitoring](#i-monitoring)
  - [2. Setup](#2-setup)
- [II. Backup](#ii-backup)
  - [2. Partage NFS](#2-partage-nfs)
  - [3. Backup de fichiers](#3-backup-de-fichiers)
  - [4. Unité de service](#4-unité-de-service)
    - [A. Unité de service](#a-unité-de-service)
    - [B. Timer](#b-timer)
    - [C. Contexte](#c-contexte)
  - [5. Backup de base de données](#5-backup-de-base-de-données)
  - [6. Petit point sur la backup](#6-petit-point-sur-la-backup)
- [III. Reverse Proxy](#iii-reverse-proxy)
  - [1. Introooooo](#1-introooooo)
  - [2. Setup simple](#2-setup-simple)
  - [3. Bonus HTTPS](#3-bonus-https)
- [IV. Firewalling](#iv-firewalling)
  - [1. Présentation de la syntaxe](#1-présentation-de-la-syntaxe)
  - [2. Mise en place](#2-mise-en-place)
    - [A. Base de données](#a-base-de-données)
    - [B. Serveur Web](#b-serveur-web)
    - [C. Serveur de backup](#c-serveur-de-backup)
    - [D. Reverse Proxy](#d-reverse-proxy)
    - [E. Tableau récap](#e-tableau-récap)

# I. Monitoring

## 2. Setup

🌞 **Setup Netdata**

```
$ sudo su -
```
```
$ bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)
```
```
$ exit
```

🌞 **Manipulation du *service* Netdata**
```
$ sudo systemctl is-enabled netdata
enabled
```
```
$ sudo systemctl status netdata
[...]
● netdata.service - Real time performance monitoring
   Active: active (running) since Tue 2021-10-12 13:33:42 CEST; 9min ago
[...]
```
```
$ sudo ss -naplt
[...]
LISTEN         0              128                             [::]:19999                           [::]:*             users:(("netdata",pid=1351,fd=6))
[...]
```
```
sudo firewall-cmd --add-port=19999/tcp --permanent
```

🌞 **Setup Alerting**

```
$ sudo /opt/netdata/etc/netdata/edit-config health_alarm_notify.conf
#------------------------------------------------------------------------------
# discord (discordapp.com) global notification options

# multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/897127125247754290/55azJJF-hdmaIa6Dk6o1Ru5ZM0epX8l6AlSbuBSNh-TvuM_u7vZBCNAYbKSAcpUSz_PI"

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="général"
```
```
$ /opt/netdata/usr/libexec/netdata/plugins.d/alarm-notify.sh test

# SENDING TEST WARNING ALARM TO ROLE: sysadmin
2021-10-12 13:54:06: alarm-notify.sh: DEBUG: Loading config file '/opt/netdata/usr/lib/netdata/conf.d/health_alarm_notify.conf'...
2021-10-12 13:54:06: alarm-notify.sh: DEBUG: Loading config file '/opt/netdata/etc/netdata/health_alarm_notify.conf'...
2021-10-12 13:54:06: alarm-notify.sh: DEBUG: Cannot find sendmail command in the system path. Disabling email notifications.
2021-10-12 13:54:06: alarm-notify.sh: DEBUG: Cannot find aws command in the system path.  Disabling Amazon SNS notifications.
```
⚠️⚠️⚠️ Une fois que vos tests d'alertes fonctionnent, vous **DEVEZ taper la commande qui suit pour que votre alerting fonctionne correctement** ⚠️⚠️⚠️

```bash
sudo sed -i 's/curl=""/curl="\/opt\/netdata\/bin\/curl -k"/' /opt/netdata/etc/netdata/health_alarm_notify.conf
```

🌞 **Config alerting**
```
$ sudo ./edit-config health.d/cpu.conf
 alarm: ram_usage
        on: system.ram
        lookup: average -1m percentage of used
        units: %
        every: 1m
        avertir : $this > 50
        info: The percentage of RAM being used by the system.
```
```
[leo@web netdata]$ sudo killall -USR2 netdata
```
```
[leo@web netdata]$ stress --vm-bytes $(awk '/MemAvailable/{printf "%d\n", $2 * 0.98;}' < /proc/meminfo)k --vm-keep -m 1
```

**Sur Discord :**
```
web.tp2.linux/db.tp2.linux needs attention, system.ram (ram), ram in use = 90.2%
ram in use = 90.2%
Percentage of used RAM. High RAM utilization. It may affect the performance of applications. If there is no swap space available, OOM Killer can start killing processes. You might want to check per-process memory usage to find the top consumers.
```

# II. Backup

🖥️ **VM `backup.tp2.linux`**

## 2. Partage NFS

🌞 **Setup environnement**
```
[leo@backup srv]$ ls
backup
```
```
[leo@backup backup]$ ls
web.tp2.linux
```
```
[leo@backup backup]$ sudo vim /etc/idmapd.conf
Domain = tp2.linux
```

🌞 **Setup partage NFS**

```
[leo@backup backup]$ sudo dnf -y install nfs-utils
```
```
[leo@backup ]$ sudo cat /etc/idmapd.conf
Domain = tp2.linux
```
```
[leo@backup backup]$ sudo cat /etc/exports
/srv/backup/web.tp2.linux 10.102.1.11(rw,no_root_squash)
```
```
[leo@backup backup]$ sudo systemctl enable --now rpcbind nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```
```
[leo@backup backup]$ sudo firewall-cmd --add-service=nfs
success
```

🌞 **Setup points de montage sur `web.tp2.linux`**

- [sur le même site, y'a ça](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=2)
- monter le dossier `/srv/backups/web.tp2.linux` du serveur NFS dans le dossier `/srv/backup/` du serveur Web
- vérifier...
  - avec une commande `mount` que la partition est bien montée
  - avec une commande `df -h` qu'il reste de la place
  - avec une commande `touch` que vous avez le droit d'écrire dans cette partition
- faites en sorte que cette partition se monte automatiquement grâce au fichier `/etc/fstab`

🌟 **BONUS** : partitionnement avec LVM

- ajoutez un disque à la VM `backup.tp2.linux`
- utilisez LVM pour créer une nouvelle partition (5Go ça ira)
- monter automatiquement cette partition au démarrage du système à l'aide du fichier `/etc/fstab`
- cette nouvelle partition devra être montée sur le dossier `/srv/backup/`

## 3. Backup de fichiers

**Un peu de scripting `bash` !** Le scripting est le meilleur ami de l'admin, vous allez pas y couper hihi.  

La syntaxe de `bash` est TRES particulière, mais ce que je vous demande de réaliser là est un script minimaliste.

Votre script **DEVRA**...

- comporter un shebang
- comporter un commentaire en en-tête qui indique le but du script, en quelques mots
- comporter un commentaire qui indique l'auteur et la date d'écriture du script

Par exemple :

```bash
#!/bin/bash
# Simple backup script
# it4 - 09/10/2021

...
```

🌞 **Rédiger le script de backup `/srv/tp2_backup.sh`**

> Dans un premier temps, le script sera générique. Vous le développez sur la machine que vous voulez, et vous ferez vos tests de sauvegarde sur les dossiers que vous voulez. Par un script "générique" j'entends qu'il doit pouvoir sauvegarder n'importe quel dossier, vers n'importe quel autre dossier de destination. Ce n'est qu'un peu plus tard (dans [la partie 4. Unité de service > C. Contexte]) que vous adapterez le script pour qu'il sauvegarde réellement les bons dossiers, afin de backup NextCloud.

- le script crée une archive compressée `.tar.gz` du dossier ciblé
  - cela se fait avec la commande `tar`
- l'archive générée doit s'appeler `tp2_backup_YYMMDD_HHMMSS.tar.gz`
  - vous remplacerez évidemment `YY` par l'année (`21`), `MM` par le mois (`10`), etc.
  - ces infos sont déterminées dynamiquement au moment où le script s'exécute à l'aide de la commande `date`
- le script utilise la commande `rsync` afin d'envoyer la sauvegarde dans le dossier de destination
- il **DOIT** pouvoir être appelé de la sorte :

```bash
$ ./tp2_backup.sh <DESTINATION> <DOSSIER_A_BACKUP>
```

📁 **Fichier `/srv/tp2_backup.sh`**

> **Il est strictement hors de question d'utiliser `sudo` dans le contenu d'un script.**  
Il est envisageable, en revanche, que le script doive être lancé avec root ou la commande `sudo` afin d'obtenir des droits élevés pendant son exécution.

🌞 **Tester le bon fonctionnement**

- exécuter le script sur le dossier de votre choix
- prouvez que la backup s'est bien exécutée
- **tester de restaurer les données**
  - récupérer l'archive générée, et vérifier son contenu

🌟 **BONUS**

- faites en sorte que votre script ne conserve que les 5 backups les plus récentes après le `rsync`
- faites en sorte qu'on puisse passer autant de dossier qu'on veut au script : `./tp2_backup.sh <DESTINATION> <DOSSIER1> <DOSSIER2> <DOSSIER3>...` et n'obtenir qu'une seule archive
- utiliser [Borg](https://borgbackup.readthedocs.io/en/stable/) plutôt que `rsync`

## 4. Unité de service

Lancer le script à la main c'est bien. **Le mettre dans une joulie *unité de service* et l'exécuter à intervalles réguliers, de manière automatisée, c'est mieux.**

Le but va être de créer un *service* systemd pour que vous puissiez interagir avec votre script de sauvegarde en faisant :

```bash
$ sudo systemctl start tp2_backup
$ sudo systemctl status tp2_backup
```

Ensuite on créera un *timer systemd* qui permettra de déclencher le lancement de ce *service* à intervalles réguliers.

**La classe nan ?**

![systemd can do that](./pics/suprised-cat.jpg)

---

### A. Unité de service

🌞 **Créer une *unité de service*** pour notre backup

- c'est juste un fichier texte hein
- doit se trouver dans le dossier `/etc/systemd/system/`
- doit s'appeler `tp2_backup.service`
- le contenu :

```bash
[Unit]
Description=Our own lil backup service (TP2)

[Service]
ExecStart=/srv/tp2_backup.sh <DESTINATION> <DOSSIER>
Type=oneshot
RemainAfterExit=no

[Install]
WantedBy=multi-user.target
```

> Pour les tests, sauvegardez le dossier de votre choix, peu importe lequel.

🌞 **Tester le bon fonctionnement**

- n'oubliez pas d'exécuter `sudo systemctl daemon-reload` à chaque ajout/modification d'un *service*
- essayez d'effectuer une sauvegarde avec `sudo systemctl start backup`
- prouvez que la backup s'est bien exécutée
  - vérifiez la présence de la nouvelle archive

---

### B. Timer

Un *timer systemd* permet l'exécution d'un *service* à intervalles réguliers.

🌞 **Créer le *timer* associé à notre `tp2_backup.service`**

- toujours juste un fichier texte
- dans le dossier `/etc/systemd/system/` aussi
- fichier `tp2_backup.timer`
- contenu du fichier :

```bash
[Unit]
Description=Periodically run our TP2 backup script
Requires=tp2_backup.service

[Timer]
Unit=tp2_backup.service
OnCalendar=*-*-* *:*:00

[Install]
WantedBy=timers.target
```

> Le nom du *timer* doit être rigoureusement identique à celui du *service*. Seule l'extension change : de `.service` à `.timer`. C'est notamment grâce au nom identique que systemd sait que ce *timer* correspond à un *service* précis.

🌞 **Activez le timer**

- démarrer le *timer* : `sudo systemctl start tp2_backup.timer`
- activer le au démarrage avec une autre commande `systemctl`
- prouver que...
  - le *timer* est actif actuellement
  - qu'il est paramétré pour être actif dès que le système boot

🌞 **Tests !**

- avec la ligne `OnCalendar=*-*-* *:*:00`, le *timer* déclenche l'exécution du *service* toutes les minutes
- vérifiez que la backup s'exécute correctement

---

### C. Contexte

🌞 **Faites en sorte que...**

- votre backup s'exécute sur la machine `web.tp2.linux`
- le dossier sauvegardé est celui qui contient le site NextCloud (quelque part dans `/var/`)
- la destination est le dossier NFS monté depuis le serveur `backup.tp2.linux`
- la sauvegarde s'exécute tous les jours à 03h15 du matin
- prouvez avec la commande `sudo systemctl list-timers` que votre *service* va bien s'exécuter la prochaine fois qu'il sera 03h15

📁 **Fichier `/etc/systemd/system/tp2_backup.timer`**  
📁 **Fichier `/etc/systemd/system/tp2_backup.service`**

## 5. Backup de base de données

Sauvegarder des dossiers c'est bien. Mais sauvegarder aussi les bases de données c'est mieux.

🌞 **Création d'un script `/srv/tp2_backup_db.sh`**

- il utilise la commande `mysqldump` pour récupérer les données de la base de données
- cela génère un fichier `.sql` qui doit ensuite être compressé en `.tar.gz`
- il s'exécute sur la machine `db.tp2.linux`
- il s'utilise de la façon suivante :

```bash
$ ./tp2_backup_db.sh <DESTINATION> <DATABASE>
```

📁 **Fichier `/srv/tp2_backup_db.sh`**  

🌞 **Restauration**

- tester la restauration de données
- c'est à dire, une fois la sauvegarde effectuée, et le `tar.gz` en votre possession, tester que vous êtes capables de restaurer la base dans l'état au moment de la sauvegarde
  - il faut réinjecter le fichier `.sql` dans la base à l'aide d'une commmande `mysql`

🌞 ***Unité de service***

- pareil que pour la sauvegarde des fichiers ! On va faire de ce script une *unité de service*.
- votre script `/srv/tp2_backup_db.sh` doit pouvoir se lancer grâce à un *service* `tp2_backup_db.service`
- le *service* est exécuté tous les jours à 03h30 grâce au *timer* `tp2_backup_db.timer`
- prouvez le bon fonctionnement du *service* ET du *timer*

📁 **Fichier `/etc/systemd/system/tp2_backup_db.timer`**  
📁 **Fichier `/etc/systemd/system/tp2_backup_db.service`**

## 6. Petit point sur la backup

A ce stade vous avez :

- un script qui tourne sur `web.tp2.linux` et qui **sauvegarde les fichiers de NextCloud**
- un script qui tourne sur `db.tp2.linux` et qui **sauvegarde la base de données de NextCloud**
- toutes **les backups sont centralisées** sur `backup.tp2.linux`
- **tout est géré de façon automatisée**
  - les scripts sont packagés dans des *services*
  - les services sont déclenchés par des *timers*
  - tout est paramétré pour s'allumer quand les machines boot (les *timers* comme le serveur NFS)

🔥🔥 **That is clean shit.** 🔥🔥

# III. Reverse Proxy

## 1. Introooooo

Un *reverse proxy* est un outil qui sert d'intermédiaire entre le client et un serveur donné (souvent un serveur Web).

**C'est l'admin qui le met en place, afin de protéger l'accès au serveur Web.**

Une fois en place, le client devra saisir l'IP (ou le nom) du *reverse proxy* pour accéder à l'application Web (ce ne sera plus directement l'IP du serveur Web).

Un *reverse proxy* peut permettre plusieurs choses :

- chiffrement
  - c'est lui qui mettra le HTTPS en place (protocole HTTP + chiffrement avec le protocole TLS)
  - on pourrait le faire directement avec le serveur Web (Apache) dans notre cas
  - pour de meilleures performances, il est préférable de dédier une machine au chiffrement HTTPS, et de laisser au serveur web un unique job : traiter les requêtes HTTP
- répartition de charge
  - plutôt qu'avoir un seul serveur Web, on peut en setup plusieurs
  - ils hébergent tous la même application
  - le *reverse proxy* enverra les clients sur l'un ou l'autre des serveurs Web, afin de répartir la charge à traiter
- d'autres trucs
  - caching de ressources statiques (CSS, JSS, images, etc.)
  - tolérance de pannes
  - ...

---

**Dans ce TP on va setup un reverse proxy NGINX très simpliste.**

![Apache at the back hihi](./pics/nginx-at-the-front-apache-at-the-back.jpg)

## 2. Setup simple

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |
| `front.tp2.linux`  | `10.102.1.14` | Reverse Proxy           | ?           | ?             |

🖥️ **VM `front.tp2.linu`x**

**Déroulez la [📝**checklist**📝](#checklist) sur cette VM.**

🌞 **Installer NGINX**

- vous devrez d'abord installer le paquet `epel-release` avant d'installer `nginx`
  - EPEL c'est des dépôts additionnels pour Rocky
  - NGINX n'est pas présent dans les dépôts par défaut que connaît Rocky
- le fichier de conf principal de NGINX est `/etc/nginx/nginx.conf`

🌞 **Tester !**

- lancer le *service* `nginx`
- le paramétrer pour qu'il démarre seul quand le système boot
- repérer le port qu'utilise NGINX par défaut, pour l'ouvrir dans le firewall
- vérifier que vous pouvez joindre NGINX avec une commande `curl` depuis votre PC

🌞 **Explorer la conf par défaut de NGINX**

- repérez l'utilisateur qu'utilise NGINX par défaut
- dans la conf NGINX, on utilise le mot-clé `server` pour ajouter un nouveau site
  - repérez le bloc `server {}` dans le fichier de conf principal
- par défaut, le fichier de conf principal inclut d'autres fichiers de conf
  - mettez en évidence ces lignes d'inclusion dans le fichier de conf principal

🌞 **Modifier la conf de NGINX**

- pour que ça fonctionne, le fichier `/etc/hosts` de la machine **DOIT** être rempli correctement, conformément à la **[📝**checklist**📝](#checklist)**
- supprimer le bloc `server {}` par défaut, pour ne plus présenter la page d'accueil NGINX
- créer un fichier `/etc/nginx/conf.d/web.tp2.linux.conf` avec le contenu suivant :
  - j'ai sur-commenté pour vous expliquer les lignes, n'hésitez pas à dégommer mes lignes de commentaires

```bash
[it4@localhost nginx]$ cat conf.d/web.tp2.linux.conf 
server {
    # on demande à NGINX d'écouter sur le port 80 pour notre NextCloud
    listen 80;

    # ici, c'est le nom de domaine utilisé pour joindre l'application
    # ce n'est pas le nom du reverse proxy, mais le nom que les clients devront saisir pour atteindre le site
    server_name web.tp2.linux; # ici, c'est le nom de domaine utilisé pour joindre l'application (pas forcéme

    # on définit un comportement quand la personne visite la racine du site (http://web.tp2.linux/)
    location / {
        # on renvoie tout le trafic vers la machine web.tp2.linux
        proxy_pass http://web.tp2.linux;
    }
}
```

## 3. Bonus HTTPS

**Etape bonus** : mettre en place du chiffrement pour que nos clients accèdent au site de façon plus sécurisée.

🌟 **Générer la clé et le certificat pour le chiffrement**

- il existe plein de façons de faire
- nous allons générer en une commande la clé et le certificat
- puis placer la clé et le cert dans les endroits standards pour la distribution Rocky Linux

```bash
# On se déplace dans un dossier où on peut écrire
$ cd ~

# Génération de la clé et du certificat
# Attention à bien saisir le nom du site pour le "Common Name"
$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
[...]
Common Name (eg, your name or your server\'s hostname) []:web.tp2.linux
[...]

# On déplace la clé et le certificat dans les dossiers standards sur Rocky
# En le renommant
$ sudo mv server.key /etc/pki/tls/private/web.tp2.linux.key
$ sudo mv server.crt /etc/pki/tls/certs/web.tp2.linux.crt

# Setup des permissions restrictives
$ sudo chown root:root /etc/pki/tls/private/web.tp2.linux.key
$ sudo chown root:root /etc/pki/tls/certs/web.tp2.linux.crt
$ sudo chmod 400 /etc/pki/tls/private/web.tp2.linux.key
$ sudo chmod 644 /etc/pki/tls/certs/web.tp2.linux.crt
```

🌟 **Modifier la conf de NGINX**

- inspirez-vous de ce que vous trouvez sur internet
- il n'y a que deux lignes à ajouter
  - une ligne pour préciser le chemin du certificat
  - une ligne pour préciser le chemin de la clé
- et une ligne à modifier
  - préciser qu'on écoute sur le port 443, avec du chiffrement
- n'oubliez pas d'ouvrir le port 443/tcp dans le firewall

🌟 **TEST**

- connectez-vous sur `https://web.tp2.linux` depuis votre PC
- petite avertissement de sécu : normal, on a signé nous-mêmes le certificat
  - vous pouvez donc "Accepter le risque" (le nom du bouton va changer suivant votre navigateur)
  - avec `curl` il faut ajouter l'option `-k` pour désactiver cette vérification

# IV. Firewalling

**On va rendre nos firewalls un peu plus agressifs.**

Actuellement je vous ai juste demandé d'autoriser le trafic sur tel ou tel port. C'est bien.

**Maintenant on va restreindre le trafic niveau IP aussi.**

Par exemple : notre base de données `db.tp2.linux` n'est accédée que par le serveur Web `web.tp2.linux`, et par aucune autre machine.  
On va donc configurer le firewall de la base de données pour qu'elle n'accepte QUE le trafic qui vient du serveur Web.

**On va *harden* ("durcir" en français) la configuration de nos firewalls.**

## 1. Présentation de la syntaxe

> **N'oubliez pas d'ajouter `--permanent` sur toutes les commandes `firewall-cmd`** si vous souhaitez que le changement reste effectif après un rechargement de FirewallD.

**Première étape** : définir comme politique par défaut de TOUT DROP. On refuse tout, et on whiteliste après.

Il existe déjà une zone appelée `drop` qui permet de jeter tous les paquets. Il suffit d'ajouter nos interfaces dans cette zone.

```bash
$ sudo firewall-cmd --list-all # on voit qu'on est par défaut dans la zone "public"
$ sudo firewall-cmd --set-default-zone=drop # on configure la zone "drop" comme zone par défaut
$ sudo firewall-cmd --zone=drop --add-interface=enp0s8 # ajout explicite de l'interface host-only à la zone "drop"
```

**Ensuite**, on peut créer une nouvelle zone, qui autorisera le trafic lié à telle ou telle IP source :

```bash
$ sudo firewall-cmd --add-zone=ssh # le nom "ssh" est complètement arbitraire. C'est clean de faire une zone par service.
```

**Puis** on définit les règles visant à autoriser un trafic donné :

```bash
$ sudo firewall-cmd --zone=ssh --add-source=10.102.1.1/32 # 10.102.1.1 sera l'IP autorisée
$ sudo firewall-cmd --zone=ssh --add-port=22/tcp # uniquement le trafic qui vient 10.102.1.1, à destination du port 22/tcp, sera autorisé
```

**Le comportement de FirewallD sera alors le suivant :**

- si l'IP source d'un paquet est `10.102.1.1`, il traitera le paquet comme étant dans la zone `ssh`
- si l'IP source est une autre IP, et que le paquet arrive par l'interface `enp0s8` alors le paquet sera géré par la zone `drop` (le paquet sera donc *dropped* et ne sera jamais traité)

> *L'utilisation de la notation `IP/32` permet de cibler une IP spécifique. Si on met le vrai masque `10.102.1.1/24` par exemple, on autorise TOUT le réseau `10.102.1.0/24`, et non pas un seul hôte. Ce `/32` c'est un truc qu'on voit souvent en réseau, pour faire référence à une IP unique.*

![Cut here to activate firewall :D](./pics/cut-here-to-activate-firewall-best-label-for-lan-cable.jpg)

## 2. Mise en place

### A. Base de données

🌞 **Restreindre l'accès à la base de données `db.tp2.linux`**

- seul le serveur Web doit pouvoir joindre la base de données sur le port 3306/tcp
- vous devez aussi autoriser votre accès SSH
- n'hésitez pas à multiplier les zones (une zone `ssh` et une zone `db` par exemple)

> Quand vous faites une connexion SSH, vous la faites sur l'interface Host-Only des VMs. Cette interface est branchée à un Switch qui porte le nom du Host-Only. Pour rappel, votre PC a aussi une interface branchée à ce Switch Host-Only.  
C'est depuis cette IP que la VM voit votre connexion. C'est cette IP que vous devez autoriser dans le firewall de votre VM pour SSH.

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

- `sudo firewall-cmd --get-active-zones`
- `sudo firewall-cmd --get-default-zone`
- `sudo firewall-cmd --list-all --zone=?`

### B. Serveur Web

🌞 **Restreindre l'accès au serveur Web `web.tp2.linux`**

- seul le reverse proxy `front.tp2.linux` doit accéder au serveur web sur le port 80
- n'oubliez pas votre accès SSH

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

### C. Serveur de backup

🌞 **Restreindre l'accès au serveur de backup `backup.tp2.linux`**

- seules les machines qui effectuent des backups doivent être autorisées à contacter le serveur de backup *via* NFS
- n'oubliez pas votre accès SSH

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

### D. Reverse Proxy

🌞 **Restreindre l'accès au reverse proxy `front.tp2.linux`**

- seules les machines du réseau `10.102.1.0/24` doivent pouvoir joindre le proxy
- n'oubliez pas votre accès SSH

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

### E. Tableau récap

🌞 **Rendez-moi le tableau suivant, correctement rempli :**

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |
| `front.tp2.linux`  | `10.102.1.14` | Reverse Proxy           | ?           | ?             |