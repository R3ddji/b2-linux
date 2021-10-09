# TP2 pt. 1 : Gestion de service

# Sommaire

- [TP2 pt. 1 : Gestion de service](#tp2-pt-1--gestion-de-service)
- [I. Un premier serveur web](#i-un-premier-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)
- [II. Une stack web plus avancée](#ii-une-stack-web-plus-avancée)
    - [A. Serveur Web et NextCloud](#a-serveur-web-et-nextcloud)
    - [B. Base de données](#b-base-de-données)
    - [C. Finaliser l'installation de NextCloud](#c-finaliser-linstallation-de-nextcloud)

# I. Un premier serveur web

## 1. Installation

🖥️ **VM web.tp2.linux**

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
```
[leo@web ~]$ systemctl is-enabled httpd.service
enabled
```

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

### A. Serveur Web et NextCloud

🌞 Install du serveur Web et de NextCloud sur `web.tp2.linux`

```
[leo@web ~]$ dnf install epel-release
```
```
[leo@web ~]$ sudo dnf update
```
```
[leo@web ~]$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```
```
[leo@web ~]$ sudo dnf module list php
[...]
Rocky Linux 8 - AppStream
Name               Stream                 Profiles                                Summary
php                7.2 [d]                common [d], devel, minimal              PHP scripting language
php                7.3                    common [d], devel, minimal              PHP scripting language
php                7.4                    common [d], devel, minimal              PHP scripting language

Remi's Modular repository for Enterprise Linux 8 - x86_64
Name               Stream                 Profiles                                Summary
php                remi-7.2               common [d], devel, minimal              PHP scripting language
php                remi-7.3               common [d], devel, minimal              PHP scripting language
php                remi-7.4               common [d], devel, minimal              PHP scripting language
php                remi-8.0               common [d], devel, minimal              PHP scripting language
php                remi-8.1               common [d], devel, minimal              PHP scripting language
```
```
[leo@web ~]$ sudo dnf module enable php:remi-7.4
```
```
[leo@web ~]$ dnf module list php
[...]
php               remi-7.4 [e]              common [d], devel, minimal             PHP scripting language
[...]
```
```
[leo@web ~]$ sudo dnf install httpd mariadb-server vim wget zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp
```
```
[leo@web ~]$ sudo vim /etc/httpd/sites-available/web.tp2.linux
  DocumentRoot /var/www/sub-domains/web.tp2.linux/html/
  ServerName  web.tp2.linux

  <Directory /var/www/sub-domains/web.tp2.linux/html/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```
```
[leo@web httpd]$ sudo ln -s /etc/httpd/sites-available/web.tp2.linux /etc/httpd/sites-enabled/
```
```
[leo@web httpd]$ sudo mkdir -p /var/www/sub-domains/web.tp2.linux/html
```
```
[leo@web httpd]$ timedatectl
               Local time: Sat 2021-10-09 16:05:52 CEST
           Universal time: Sat 2021-10-09 14:05:52 UTC
                 RTC time: Sat 2021-10-09 14:05:52
                Time zone: Europe/Paris (CEST, +0200)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```
```
[leo@web ~]$ wget https://download.nextcloud.com/server/releases/nextcloud-21.0.1.zip
```
```
[leo@web ~]$ unzip nextcloud-21.0.1.zip
```
```
[leo@web ~]$ cd nextcloud/
```
```
[leo@web nextcloud]$ sudo cp -Rf * /var/www/sub-domains/web.tp2.linux/html/
```
```
[leo@web ~]$ chown -Rf apache.apache /var/www/sub-domains/web.tp2.linux/html
```
```
[leo@web ~]$ sudo mkdir /var/www/sub-domains/web.tp2.linux/html/data
```
```
[leo@web ]$ sudo mv /var/www/sub-domains/web.tp2.linux/html/data /var/www/sub-domains/web.tp2.linux/
```
```
[leo@web html]$ sudo systemctl restart httpd
```
📁 **Fichier `/etc/httpd/conf/httpd.conf`**  
📁 **Fichier `/etc/httpd/conf/sites-available/web.tp2.linux`**

### B. Base de données

🌞 **Install de MariaDB sur `db.tp2.linux`**

```
[leo@db ~]$ sudo dnf install mariadb-server
```
```
[leo@db ~]$ mysql_secure_installation
```
```
[leo@db ~]$ sudo ss -naplt
[sudo] password for leo:
State     Recv-Q    Send-Q       Local Address:Port       Peer Address:Port   Process
LISTEN    0         128                0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=832,fd=5))
LISTEN    0         80                       *:3306                  *:*       users:(("mysqld",pid=41296,fd=21))
LISTEN    0         128                   [::]:22                 [::]:*       users:(("sshd",pid=832,fd=7))
```

🌞 **Préparation de la base pour NextCloud**

```
MariaDB [(none)]> CREATE USER 'web'@'10.102.1.11' IDENTIFIED BY 'azerty';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS web CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON web.* TO 'web'@'10.102.1.11';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)
```
🌞 **Exploration de la base de données**
```
[leo@web ]$ sudo mysql -u nextcloud -h 10.102.1.12 -pmeow
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.3.28-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```
```
SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |         |
| performance_schema |
| web                |
+--------------------+
5 rows in set (0.001 sec)
```
```
use web
Database changed
```
```
SHOW TABLES;
Empty set (0.001 sec)
```
```
SELECT user FROM mysql.user;
+-----------+
| user      |
+-----------+
| root      |
| root      |
| root      |
+-----------+
```

### C. Finaliser l'installation de NextCloud

🌞 sur votre PC
```
#
127.0.0.1 localhost
::1 localhost
10.102.1.11	web.tp2.linux`
```
![](./img/Nextcloud.png)

🌞 **Exploration de la base de données**
```
MariaDB [nextcloud]> SELECT COUNT(*) from information_schema.tables where TABLE_SCHEMA = 'nextcloud';
+----------+
| COUNT(*) |
+----------+
|       99 |
+----------+
1 row in set (0.000 sec)
```

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | 80           | ?             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | 3306           |  10.102.1.11            |