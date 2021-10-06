# TP2 pt. 1 : Gestion de service

# Sommaire

- [TP2 pt. 1 : Gestion de service](#tp2-pt-1--gestion-de-service)
- [I. Un premier serveur web](#i-un-premier-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)
- [II. Une stack web plus avancée](#ii-une-stack-web-plus-avancée)
  - [1. Intro](#1-intro)
  - [2. Setup](#2-setup)
    - [A. Serveur Web et NextCloud](#a-serveur-web-et-nextcloud)
    - [B. Base de données](#b-base-de-données)
    - [C. Finaliser l'installation de NextCloud](#c-finaliser-linstallation-de-nextcloud)

# I. Un premier serveur web

## 1. Installation

🖥️ **VM web.tp2.linux**

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | ?           | ?             |

🌞 **Installer le serveur Apache**

```
[leo@web ~]$ sudo dnf install httpd
```

```
[leo@web ~]$ sudo cat /etc/httpd/conf/httpd.conf

ServerRoot "/etc/httpd"

Listen 80

Include conf.modules.d/*.conf

User apache
Group apache


ServerAdmin root@localhost


<Directory />
    AllowOverride none
    Require all denied
</Directory>


DocumentRoot "/var/www/html"

<Directory "/var/www">
    AllowOverride None
    Require all granted
</Directory>

<Directory "/var/www/html">
    Options Indexes FollowSymLinks

    AllowOverride None

    Require all granted
</Directory>

<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

<Files ".ht*">
    Require all denied
</Files>

ErrorLog "logs/error_log"

LogLevel warn

<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common

    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>


    CustomLog "logs/access_log" combined
</IfModule>

<IfModule alias_module>


    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"

</IfModule>

<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>

<IfModule mime_module>
    TypesConfig /etc/mime.types

    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz



    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>

AddDefaultCharset UTF-8

<IfModule mime_magic_module>
    MIMEMagicFile conf/magic
</IfModule>


EnableSendfile on

IncludeOptional conf.d/*.conf
```

🌞 **Démarrer le service Apache**
  ```
  [leo@web ~]$ sudo systemctl start httpd.service
  ```
  ```
  [leo@web ~]$ sudo systemctl enable httpd.service
  Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
  ```
  ```
  [leo@web ~]$ sudo ss -alnpt
State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port         Process
LISTEN          0               128                            0.0.0.0:22                          0.0.0.0:*             users:(("sshd",pid=830,fd=5))
LISTEN          0               128                                  *:80                                *:*             users:(("httpd",pid=24128,fd=4),("httpd",pid=24127,fd=4),("httpd",pid=24126,fd=4),("httpd",pid=24124,fd=4))
LISTEN          0               128                               [::]:22                             [::]:*             users:(("sshd",pid=830,fd=7))
```
```
  [leo@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent 
  success
```
```
  [leo@web ~]$ sudo firewall-cmd --reload
 success
```

🌞 **TEST**

- vérifier que le service est démarré
```
[leo@web ~]$ sudo systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-09-29 16:33:38 CEST; 12min ago
     Docs: man:httpd.service(8)
 Main PID: 24124 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 11385)
   Memory: 25.6M
   CGroup: /system.slice/httpd.service
           ├─24124 /usr/sbin/httpd -DFOREGROUND
           ├─24125 /usr/sbin/httpd -DFOREGROUND
           ├─24126 /usr/sbin/httpd -DFOREGROUND
           ├─24127 /usr/sbin/httpd -DFOREGROUND
           └─24128 /usr/sbin/httpd -DFOREGROUND

Sep 29 16:33:38 web.tp2.linux systemd[1]: Starting The Apache HTTP Server...
Sep 29 16:33:38 web.tp2.linux systemd[1]: Started The Apache HTTP Server.
Sep 29 16:33:38 web.tp2.linux httpd[24124]: Server configured, listening on: port 80
```
- vérifier qu'il est configuré pour démarrer automatiquement
```
[leo@web ~]$ systemctl is-enabled httpd.service
enabled
```
- vérifier avec une commande curl localhost que vous joignez votre serveur web localement
```
[leo@web ~]$ curl localhost
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }


  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }

  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }

  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }

  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }

  img {

    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }

  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }

  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }

  /* Desktop  View Options */

  @media (min-width: 768px)  {

    body {
      padding: 10em 20% !important;
    }

    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }

    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;


    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }

  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }

    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }


  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>

      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software it working
            correctly.</p>
      </div>

      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>


        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>

          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>

          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>

        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src= "icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>
      </div>
      </div>

      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```

```
curl 10.102.1.11:80
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
Just visiting?
This website you are visiting is either experiencing problems or could be going through maintenance.
If you would like the let the administrators of this website know that you've seen this page instead of the page
you've expected, you should send them an email. In general, mail sent to the name "webmaster" and directed to the
website's domain should reach the appropriate person.
The most common email address to send to is: "webmaster@example.com"
Note:
The Rocky Linux distribution is a stable and reproduceable platform based on the sources of Red Hat Enterprise Linux
(RHEL). With this in mind, please understand that:
Neither the Rocky Linux Project nor the Rocky Enterprise Software Foundation have anything to do with this website or
its content.
The Rocky Linux Project nor the RESF have "hacked" this webserver: This test page is included with the distribution.
For more information about Rocky Linux, please visit the Rocky Linux website.
I am the admin, what do I do?
You may now add content to the webroot directory for your software.
For systems using the Apache Webserver: You can add content to the directory /var/www/html/. Until you do so, people
visiting your website will see this page. If you would like this page to not be shown, follow the instructions in:
/etc/httpd/conf.d/welcome.conf.
For systems using Nginx: You can add your content in a location of your choice and edit the root configuration
directive in /etc/nginx/nginx.conf.

Apache™ is a registered trademark of the Apache Software Foundation in the United States and/or other countries.
NGINX™ is a registered trademark of F5 Networks, Inc..
Au caractère Ligne:1 : 1
+ curl 10.102.1.11:80
+ ~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation : (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest], WebEx
   ception
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```

## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

```
 [leo@web ~]$ chkconfig httpd on
Note: Forwarding request to 'systemctl enable httpd.service'.
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ====
Authentication is required to manage system service or unit files.
Authenticating as: leo
Password:
==== AUTHENTICATION COMPLETE ====
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
Authentication is required to reload the systemd state.
Authenticating as: leo
Password:
==== AUTHENTICATION COMPLETE ====
```
```
[leo@web ~]$ sudo systemctl list-unit-files
[...]
httpd.service                              enabled
[...]
```
```
[leo@web ~]$ sudo cat /lib/systemd/system/httpd.service
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#       [Service]
#       Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
🌞 **Déterminer sous quel utilisateur tourne le processus Apache**
```
[leo@web ~]$ sudo cat /etc/httpd/conf/httpd.conf
User apache
```
```
[leo@web ~]$ ps -ef | grep apache
apache       861     837  0 14:08 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       863     837  0 14:08 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       864     837  0 14:08 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       865     837  0 14:08 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```
```
[leo@web testpage]$ ls -al
total 12
drwxr-xr-x.  2 root root   24 Sep 29 16:23 .
drwxr-xr-x. 89 root root 4096 Sep 29 16:26 ..
-rw-r--r--.  1 root root 7621 Jun 11 17:23 index.html
```

🌞 **Changer l'utilisateur utilisé par Apache**

```
[leo@web testpage]$ sudo adduser leoapache
```
```
[leo@web testpage]$ sudo cat /etc/httpd/conf/httpd.conf
[...]
User leoapache
Group leoapache
[...]
```
```
[leo@web ~]$ sudo systemctl reboot httpd.service
```
```
[leo@web ~]$ ps -ef
[...]
leoapac+     866     845  0 15:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
leoapac+     867     845  0 15:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
leoapac+     868     845  0 15:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
leoapac+     869     845  0 15:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
[...]
```
🌞 **Faites en sorte que Apache tourne sur un autre port**
```
[leo@web ~]$ sudo vim /etc/httpd/conf/httpd.conf
[...]
Listen 90
[...]
```
```
[leo@web ~]$ sudo firewall-cmd --add-port=90/tcp --permanent
```
```
[leo@web ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
```
```
sudo systemctl reload httpd
```
```
[leo@web ~]$ sudo ss -naptl
State                    Recv-Q                   Send-Q                                       Local Address:Port                                       Peer Address:Port                   Process
LISTEN                   0                        128                                                0.0.0.0:22                                              0.0.0.0:*                       users:(("sshd",pid=833,fd=5))
LISTEN                   0                        128                                                   [::]:22                                                 [::]:*                       users:(("sshd",pid=833,fd=7))
LISTEN                   0                        128                                                      *:90                                                    *:*                       users:(("httpd",pid=2984,fd=4),("httpd",pid=2983,fd=4),("httpd",pid=2982,fd=4),("httpd",pid=2980,fd=4))
```
```
[leo@web ~]$ curl localhost:90
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
```
```
PS C:\Users\lbour> curl http://10.102.1.11:90/
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
Just visiting?
This website you are visiting is either experiencing problems or could be going through maintenance.
If you would like the let the administrators of this website know that you've seen this page instead of the page
you've expected, you should send them an email. In general, mail sent to the name "webmaster" and directed to the
website's domain should reach the appropriate person.
The most common email address to send to is: "webmaster@example.com"
Note:
The Rocky Linux distribution is a stable and reproduceable platform based on the sources of Red Hat Enterprise Linux
(RHEL). With this in mind, please understand that:
Neither the Rocky Linux Project nor the Rocky Enterprise Software Foundation have anything to do with this website or
its content.
The Rocky Linux Project nor the RESF have "hacked" this webserver: This test page is included with the distribution.
For more information about Rocky Linux, please visit the Rocky Linux website.
I am the admin, what do I do?
You may now add content to the webroot directory for your software.
For systems using the Apache Webserver: You can add content to the directory /var/www/html/. Until you do so, people
visiting your website will see this page. If you would like this page to not be shown, follow the instructions in:
/etc/httpd/conf.d/welcome.conf.
For systems using Nginx: You can add your content in a location of your choice and edit the root configuration
directive in /etc/nginx/nginx.conf.

Apache™ is a registered trademark of the Apache Software Foundation in the United States and/or other countries.
NGINX™ is a registered trademark of F5 Networks, Inc..
Au caractère Ligne:1 : 1
+ curl http://10.102.1.11:90/
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation : (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest], WebEx
   ception
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```

📁 **Fichier `/etc/httpd/conf/httpd.conf`** (que vous pouvez renommer si besoin, vu que c'est le même nom que le dernier fichier demandé)

# II. Une stack web plus avancée

## 1. Intro

> Pas mal de blabla nécessaire avant de vous lancer. Lisez bien la partie en entier.

Le serveur web `web.tp2.linux` sera le serveur qui accueillera les clients. C'est sur son IP que les clients devront aller pour visiter le site web.  

La serveur de base de données `db.tp2.linux` sera un serveur uniquement accessible depuis `web.tp2.linux`. Les clients ne pourront pas y accéder. Le serveur de base de données stocke les infos nécessaires au serveur web, pour le bon fonctionnement du site web.

---

Bon j'ai un peu réfléchi et le but pour nous là c'est juste d'avoir un serv web + une db, peu importe ce que c'est le site. J'ai pas envie d'aller deep dans la conf de l'un ou de l'autre avec vous pour le moment. Donc on va installer un truc un peu clé en main : Nextcloud.

En plus c'est utile comme truc : c'est un p'tit serveur pour héberger ses fichiers via une WebUI, style Google Drive. Mais on l'héberge nous-mêmes :)

Il y a [**une doc officielle Rocky Linux** plutôt bien fichue pour l'install de Nextcloud](https://docs.rockylinux.org/guides/cms/cloud_server_using_nextcloud/#next-steps), je vous laisse la suivre.

⚠️⚠️⚠️**ATTENTION** lisez bien toute la suite avant de vous lancer dans l'install, **lisez la partie en entier.** Ca vous évitera bien des soucis. ⚠️⚠️⚠️

Dans la doc officielle, on vous fait installer le serveur web (Apache) et la base de données (MariaDB) sur la même machine. **Ce n'est PAS ce que nous voulons : nous voulons avoir chaque service sur une machine dédiée.** Donc vous allez devoir ajuster la doc un petit peu. Rien de bien violent :

- quand on vous parle du serveur web, vous faites ça sur `web.tp2.linux`
- quand on vous parle de la base de données, vous faites ça sur `db.tp2.linux`
- à la fin, quand c'est en place, on vous demande d'aller l'interface Web de nextcloud et d'y saisir l'IP de la base de données, pour que NextCloud puisse s'y connecter. Vous saisirez ici l'IP de `db.tp2.linux` à la place de `localhost`

---
---

**Aussi** dans la doc vous est fourni un lien pour installer MariaDB de façon secure. C'est parfait, suivez-le.  
**Par contre** on ne vous donnez aucune infos sur quelle base de données créer dans le serveur de base de données.  
**Donc** avant de démarrer NextCloud, vous devrez :

- démarrer le service de base de données sur `db.tp2.linux`
- vous connecter au serveur de base de données
- exécutez des commandes SQL pour configurer la base, qu'elle soit utilisable par NextCloud

---
---

**Enfin, quelques tips en vrac pour que vous dérouliez l'install dans de bonnes conditions.** Certains tips vont vous paraître random. Beaucoup moins une fois que vous aurez déroulé la doc :

➜ ***Lisez*** et ***suivez bien toutes les instructions***. Toutes les étapes sont strictement nécessaires.

➜ Vous pouvez récupérer votre **timezone** plus facilement en une simple commande : `timedatectl`.

➜ **Si on vous parle d'un dossier et qu'il n'existe pas, créez-le.** Si vous ne savez quelles permissions lui donner, donnez-lui les mêmes permissions que les autres fichiers/dossiers qui se trouvent dans le même dossier.

➜ **Une fois que vous aurez fini d'installer NextCloud**, vous devez visiter l'interface Web. Vous ouvrirez donc votre navigateur, et vous rendrez à l'URL `http://web.tp2.linux`. La page d'accueil de NextCloud s'affichera. **NE VOUS CONNECTEZ PAS** et passez à l'installation de la base de données. C'est sur cet écran que vous indiquerez à NextCloud comment se connecter à votre base, une fois que vous l'aurez installé.

➜ Dans la doc, il est dit : "As noted earlier, we are using the ***"Apache Sites Enabled"*** procedure found here to configure Apache"

Cette ***"Apache Sites Enabled" procedure*** fait référence à une façon d'organiser le dossier `/etc/httpd` pour pas que ce soit le bordel :

- on a créé cette façon de faire car Apache est souvent utilisé pour héberger plusieurs sites web en même temps
- l'idée c'est de faire
  - un dossier `sites-available/` qui contient un fichier de configuration par site web
  - un dossier `sites-enabled/` qui contient des liens vers les fichiers de `sites-available`
- de cette façon il est plus simple de s'y retrouver :
  - un fichier par site, c'est clean
  - un dossier dédié à la conf des sites, et pas à la conf d'Apache plus générale, c'est clean
  - si on veut mettre un site hors-ligne sans faire de la crasse (commenter des lignes, supprimer la conf liée au site etc.) c'est EZ on a juste a supprimer le lien dans `sites-enabled/` ce qui garde intact le vrai fichier de conf dans `sites-available/`
- bref c'est clean quoi :)

**SAUF QUE** si vous suivez juste la doc, ça va pas fonctionner. En effet, les fichiers dans `sites-enabled/` ne seront jamais lus par défaut. Vous devez donc ajouter la ligne suivante dans le fichier `/etc/httpd/httpd.conf` (tout en bas) :

```bash
IncludeOptional sites-enabled/*
```

## 2. Setup

🖥️ **VM db.tp2.linux**

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | ?           | ?             |

> Ce tableau devra figurer à la fin du rendu, avec les ? remplacés par la bonne valeur (un seul tableau à la fin). Je vous le remets à chaque fois, à des fins de clarté, pour lister les machines qu'on a à chaque instant du TP.

### A. Serveur Web et NextCloud

**Créez les 2 machines et déroulez la [📝**checklist**📝](#checklist).**

🌞 Install du serveur Web et de NextCloud sur `web.tp2.linux`

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/cms/cloud_server_using_nextcloud/#next-steps)
  - **uniquement pour le serveur Web + NextCloud**, vous ferez la base de données MariaDB après
  - quand ils parlent de la base de données, juste vous sautez l'étape, on le fait après :)
- je veux dans le rendu **toutes** les commandes réalisées
  - n'oubliez pas la commande `history` qui permet de voir toutes les commandes tapées précédemment

Une fois que vous avez la page d'accueil de NextCloud sous les yeux avec votre navigateur Web, **NE VOUS CONNECTEZ PAS** et continuez le TP

📁 **Fichier `/etc/httpd/conf/httpd.conf`**  
📁 **Fichier `/etc/httpd/conf/sites-available/web.tp2.linux`**

### B. Base de données

🌞 **Install de MariaDB sur `db.tp2.linux`**

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/database/database_mariadb-server/)
- manipulation 
- je veux dans le rendu **toutes** les commandes réalisées
- vous repérerez le port utilisé par MariaDB avec une commande `ss` exécutée sur `db.tp2.linux`

🌞 **Préparation de la base pour NextCloud**

- une fois en place, il va falloir préparer une base de données pour NextCloud :
  - connectez-vous à la base de données à l'aide de la commande `sudo mysql -u root`
  - exécutez les commandes SQL suivantes :

```sql
# Création d'un utilisateur dans la base, avec un mot de passe
# L'adresse IP correspond à l'adresse IP depuis laquelle viendra les connexions. Cela permet de restreindre les IPs autorisées à se connecter.
# Dans notre cas, c'est l'IP de web.tp2.linux
# "meow" c'est le mot de passe :D
CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'meow';

# Création de la base de donnée qui sera utilisée par NextCloud
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

# On donne tous les droits à l'utilisateur nextcloud sur toutes les tables de la base qu'on vient de créer
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';

# Actualisation des privilèges
FLUSH PRIVILEGES;
```

> Par défaut, vous avez le droit de vous connectez localement à la base si vous êtes `root`. C'est pour ça que `sudo mysql -u root` fonctionne, sans nous demander de mot de passe. Evidemment, n'importe quelles autres conditions ne permettent pas une connexion aussi facile à la base.

🌞 **Exploration de la base de données**

- afin de tester le bon fonctionnement de la base de données, vous allez essayer de vous connecter, comme NextCloud le fera :
  - depuis la machine `web.tp2.linux` vers l'IP de `db.tp2.linux`
  - vous pouvez utiliser la commande `mysql` pour vous connecter à une base de données depuis la ligne de commande
    - par exemple `mysql -u <USER> -h <IP_DATABASE> -p`
- utilisez les commandes SQL fournies ci-dessous pour explorer la base

```sql
SHOW DATABASES;
USE <DATABASE_NAME>;
SHOW TABLES;
```

- trouver une commande qui permet de lister tous les utilisateurs de la base de données

> Les utilisateurs de la base de données sont différents des utilisateurs du système Linux sous-jacent. Les utilisateurs de la base définissent des identifiants utilisés pour se connecter à la base afin d'y voir ou d'y modifier des données.

### C. Finaliser l'installation de NextCloud

🌞 sur votre PC

- modifiez votre fichier `hosts` (oui, celui de votre PC, de votre hôte)
  - pour pouvoir joindre l'IP de la VM en utilisant le nom `web.tp2.linux`
- avec un navigateur, visitez NextCloud à l'URL `http://web.tp2.linux`
  - c'est possible grâce à la modification de votre fichier `hosts`
- on va vous demander un utilisateur et un mot de passe pour créer un compte admin
  - ne saisissez rien pour le moment
- cliquez sur "Storage & Database" juste en dessous
  - choisissez "MySQL/MariaDB"
  - saisissez les informations pour que NextCloud puisse se connecter avec votre base
- saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation

🌞 **Exploration de la base de données**

- connectez vous en ligne de commande à la base de données après l'installation terminée
- déterminer combien de tables ont été crées par NextCloud lors de la finalisation de l'installation
  - ***bonus points*** si la réponse à cette question est automatiquement donnée par une requête SQL