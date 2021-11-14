# TP3 : Your own shiet

# Intro

Dans ce tp, j'ai décidé de mettre en fonctionnement **un service d'hébergement de fichiers**. Au début je suis partie sur le service **SparkleShare** mais vu qu'il n'y avait pas d'interface graphique, je me suis tourné vers le service **SeaFile**. Malheureusement je n'ai pas réussi à le faire fonctionner. Finalement j'ai décidé de me tourner vers le service **NextCloud** que l'on a vu dans les temps précédents.

# Installation du service NextCloud

Afin de commencer cette installation, nous aurons besoin d'une machine virtuel.

Je suis parti sur une machine linux(**web.linux.tp3**), je lui ai mit une carte **NAT**, et une carte **Host-Only** avec une IP static(10.10.1.5).


| Machine         | IP            | Service                 | Port ouvert | IPs autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.linux.tp3` | `10.10.1.5` | Serveur Web et Base de données            | 80/tcp , 3306/tcp , 19999/tcp           | 10.10.1.10             |


**On peut maintenant commencer :**

> mise à jour de la machine

```
$ sudo dnf update -y
```

> installation d'apache

```
$ sudo dnf install httpd -y
```

> démarage d'apache et on fait en sorte qu'apache démarre au démarage de la machine

```
$ sudo systemctl start httpd.service
```
```
$ sudo systemctl enable httpd.service
```

> On rajoute le port 80 qui correspond au service apache et on pense à reload le firewall

```
$ sudo firewall-cmd --add-port=80/tcp --permanent 
```
```
$ sudo firewall-cmd --reload
```

> install des ressources pour serv web

```
$ sudo dnf install epel-release -y

$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

$ sudo dnf module enable php:remi-7.4 -y

$ sudo dnf install httpd mariadb-server vim wget zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp -y
```

> création des dossiers pour le site

```
$ sudo mkdir /etc/httpd/sites-available

$ sudo mkdir /etc/httpd/sites-enabled

$ sudo mkdir /var/www/sub-domains
```

> on inclue le dossier du site dans la conf d'apache 

```
$ sudo cat /etc/httpd/conf/httpd.conf
[...]
Include /etc/httpd/sites-enabled

``` 
``` 
$sudo cat /etc/httpd/sites-available/web.linux.tp3
<VirtualHost *:80>
  DocumentRoot /var/www/sub-domains/web.linux.tp3/html/
  ServerName  nextcloud.tp3.linux

  <Directory /var/www/sub-domains/web.linux.tp3/html/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

> on fait un lien entre les dossiers du site

``` 
$ sudo ln -s /etc/httpd/sites-available/web.linux.tp3 /etc/httpd/sites-enabled/
``` 

> on créer le dossier qui contiendra le contenu du site

``` 
$ sudo mkdir -p /var/www/sub-domains/web.linux.tp3/html
``` 

> pour vérifier si la machine est à l'heure

``` 
$ timedatectl
``` 

> installation de nextcloud

``` 
$ wget https://download.nextcloud.com/server/releases/nextcloud-22.2.0.zip

$ unzip nextcloud-22.2.0.zip
``` 

> on déplace nextcloud vers le dossier crée précédement

``` 
$ sudo mv * /var/www/sub-domains/web.linux.tp3/html/
``` 

> on atribue les droits du site à apache

``` 
$ sudo chown -Rf apache.apache /var/www/sub-domains/web.linux.tp3/html/
``` 

> on redémarre le serveur

``` 
$ sudo systemctl restart httpd
``` 

**Pour accéder au site sur son ordinateur personnel, nous devons rajouter dans le fichier hosts, le nom et l'ip de la machine où se trouve la plateforme NextCloud.**

```
C:\Windows\System32\drivers\etc\hosts

127.0.0.1 localhost
::1 localhost
10.10.1.5 web.linux.tp3
```


**À ce stade, on peut accèder à la page d'accueil de NextCloud en accèdent à 10.10.1.5 à partir d'un navigateur.**

# Configuration de la base de donnée

Pour que le service fonctionne il va nous falloir une base de donnée.

> On installe mariadb pour gérer la bdd

``` 
$ sudo dnf install mariadb-server -y
``` 

> On lance le service mariadb, et on lui demande de démarer à chaque fois que la machine se lance. 

``` 
$ sudo systemctl start mariadb
$ sudo systemctl enable mariadb
``` 

> On doit maintenant sécuriser la base de donnée en mettant un mot de passe root et en réinitialisant la base de donnée par défaut de mariadb.

``` 
$ sudo mysql_secure_installation
``` 

> On ouvre le port 3306 pour mariadb, et on reload le firewall.

```
$ sudo firewall-cmd --add-port=3306/tcp --permanent

$ sudo firewall-cmd --reload
```

> On lance mariadb, en s'identifiant en tant que root.

```
sudo mysql -u root -p
```

> On peut maintenant lancer les requêtes suivantes, création d'un utilisateur nextcloud, création de la base de donnée nextcloud, on accorde tous les privilèges à l'utilisateur nextcloud.

```
CREATE USER 'nextcloud'@'10.10.1.5' IDENTIFIED BY 'nextcloud';

CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.10.1.5';

FLUSH PRIVILEGES;
```

# Lancement de Nextcloud

Connectons-nous à 10.10.1.5, sur un navigateur.

Nous arrivons sur la page d'accueil de nextcloud.

On peut rentrer les informations demandées :

Nom utilisateur -> admin
Mot de passe utilisateur -> admin

Type de base de données -> Mariadb

Nom de la base de données -> nextcloud
Mot de passe de la base de données -> nextcloud

Ip de la base de données -> 10.10.1.5

**On peut bien sûr changer ces informations selon la configuration de chacun.**

Une fois lancer, NextCloud va créer tous les services auxquels ont à besoin pour que la plateforme marche.

# Monitoring avec Netdata

Pour s'assurer du bon fonctionnement de notre machine, nous allons utiliser un service permettant de connaitre en temps réels les performences de la machine.

> on install netdata

```
$ sudo su -
$ bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)
$ exit
```

> on ouvre le port utilisé par netdata et on reload le firewall

```
sudo firewall-cmd --add-port=19999/tcp --permanent
sudo firewall-cmd --reload
```

> on redemarre le service, et on fait en sorte qu'il démarre au démarage de la machine

```
sudo systemctl restart netdata
sudo systemctl enable netdata
```

En allant sur 10.10.1.5:19999 sur un navigateur, on peut maintenant voir les performence en tant réel de la machine.

# Backup

Pour s'asssurer de ne jamais perdre les fichiers de notre plateforme, ou les informations stockées dans la base de donnée.
Nous allons créer une nouvelle machine, qui s'occupera de sauvegarder nos informations régulièrement.
Nous utiliserons aussi **Netdata** sur cette nouvelle machine.

Je suis parti sur une machine linux(**backup.linux.tp3**), je lui ai mit une carte **NAT**, et une carte **Host-Only** avec une IP static(10.10.1.10).

| Machine         | IP            | Service                 | Port ouvert | IPs autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `backup.linux.tp3` | `10.10.1.10` | Serveur de backup            | 19999/tcp          | 10.10.1.5             |

> On commence par mettre à jour la machine

```
$ sudo dnf update -y
```

> On fait les mêmes manipulations pour rajouter NetData comme on a vu plus haut.

## Partage NFS

> On crée un dossier backup pour la machine web.linux.tp3

```
$ sudo mkdir -p /srv/backup/web.linux.tp3/
```

> On installe les outils d'nfs

```
$ sudo dnf -y install nfs-utils
```

> On défini le domaine des deux machines

```
$ sudo cat /etc/idmapd.conf
[...]
Domain = tp3.linux
[...]
```

```
$ sudo cat /etc/exports
/srv/backup/web.linux.tp3 10.10.1.5(rw,no_root_squash)
```

> On fait en sorte que le service nfs se lance au démarage de la machine

```
$ sudo systemctl enable --now rpcbind nfs-server 
```

> On ajoute le service nfs au firewall et on reload

```
$ sudo firewall-cmd --add-service=nfs --permanent
$ sudo firewall-cmd --reload
```

## Point de montage

Sur **web.linux.tp3** :

> On crée un point de montage qui se monte automatiquement

```
$ sudo cat /etc/fstab
[...]
backup.linux.tp3:/srv/backup/web.linux.tp3 /srv/backup               nfs     defaults        0 0
```

# FIN

Voici ce qu'on a fait :

- Un serveur web en y incluant la plateforme NextCloud.
- Une interface NetData pour surveiller les performances des deux machines.
- Une machine s'occupant de sauvegarde les fichiers du serveur où se trouve NextCloud.

