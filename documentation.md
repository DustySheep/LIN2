# (DOC) LIN2 - Quentin Théo - Mini projet
=================================
=

## Configuration de la machine 
---------

 * Debian 8
 * 20 Go
 * 512 Mo RAM

## Configuration de l'installation de Debian 
--------------
* Langue : anglais
* continent : europe
* Pays : Suisse
* Langage : en_GB_UTF8
* Nom de la machine : debian
* Nom de domaine : CeQueVousVoulez
* Nom de l'utilisateur : CeQueVousVoulez 
* Compte : CeQueVousVoulez
* Mot de passe : CeQueVousVoulez

## Modification des sources Debian
----------------------
Etant donné que l'installation s'est effectué __manuellement__ avec une image iso, 
nous devons modifier les sources pour que la machine ne pointe plus sur cette image.
 Aller sur le site de [debgen](http://debgen.simplylinux.ch/). 

Sélectionner les informations suivantes : 
* Switzerland
* Stable (wheezy)
* 64 bits
* Cocher les cases __Main__ et __Security__

Puis cliquer sur  __Generate sources-list__

Une pop-up s'ouvre alors et propose une ligne de commande à copier dans le fichier __sources.list__.
### Marche à suivre
```
1. nano /etc/apt/sources.list
``` 
Mettre en commentaires la 1ère et 2ème ligne à l'aide du #.
Puis y insérer la ligne générée par debgen soit : 
```    
2. deb http://ftp.ch.debian.org/debian stable main
``` 
Sauver et fermer le fichier puis lancer la commande suivante pour que le système applique ces modifications.
```
apt-get update
```
## Installation du SSH 
-------------
``` 
apt-get install openssh-server
```
### Configuration du SSH

```
nano /etc/ssh/sshd_config
```
Modifier dans la rubrique __Authentification__ pour éviter que l'on puisse se connecter en __root__ au serveur.
```
permitRootLogin : NO
```
## Installation de Nginx
------------------------------
```
apt-get install nginx
```

### Configuration Nginx
Premièrement, il faut éditer le fichier de configuration Nginx:
```
nano /etc/nginx/sites-available/default
```
Dans ce fichier, remplacer les informations suivantes :
```
root /usr/share/nginx/www/;
server_name VotreAdresseIp ou NomDeDomaine;
```
Décommenter
``` 
location ~\.php$ {.......}
```
à l'intérieur : 
```fastcgi_pass unix:/var/run/php5-fpm.sock;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name$;
include fastcgi_params;
```
Afin que chaque utilisateur ne puisse pas voir les dossiers des autres utilisateurs, nous avons retirer les droits de lecture sur le dossier __www__. 

Il est nécessaire lors de la création du dossier de l'utilisateur, de le nommer propriétaire de son dossier à l'aide de la commande __chown__. 

Afin d'améliorer la sécurité interne il est indispensable d'enlever les droits d'exécution aux __other__ pour chaque dossier user. Ainsi ils n'auront accès qu'a leur propre dossier.

#### Configuration par utilisateur
Pour chaque utilisateur il est nécessaire de créer un fichier de configuration.
Ce fichier permet de spécifier l'endroit où l'utilisateur peut déposer ses fichiers afin que le serveur puisse les traiter. 
Ce fichier doit se trouver dans le dossier __conf.d__ de Nginx. Le nom du fichier doit être le même que le nom de l'utilisateur. Exemple : __NomUtilisateur.conf__

Veuillez insérer les informations suivantes dans le fichier :
```
server {
    listen 80;
    root /usr/share/nginx/www/NomUtilisateur;
    index index.php index.html index.htm;
    server_name localhost;
    location / {
            try_files $uri $uri/ /index.php;
    }
    location usr/share/www/NomUtilisateur/.php$ {
            try_files $uri =404;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
    }
}
```

## Installation de PHP5-fpm et PHP5-MySql
```
apt-get install php5-fpm php5-mysql
```

### Configuration PHP5-fpm
Se déplacer dans le fichier /etc/php5/fpm/php.ini
Modifier la ligne suivante :
```
cgi.fix_pathinfo = 0
```
Celà va ralentir la requête mais la rend plus sûre.

Il est maintenant nécessaire de changer le port d'écoute de php5 qui pointe sur son socket.
Toujours dans le même fichier, modifier la ligne suivante :
```
listen = /var/run/php5-fpm-sock
```
Relancer le service 
```
service php5-fpm restart
```

## Installation de MariaDB
 ```
apt-get install mariadb-server mariadb-client
```
Configurez le compte root.
### Configuration de MariaDB
afin d'améliorer la sécurité de MariaDB, exécuter la commande : 
```
mysqsl_secure_installation
```
Celle-ci va supprimer les comptes anonymes, réinitialisé le mot de passe root, empêcher la connexion en root en dehors de localhost.
### Création d'un utilisateur
```
GRANT ALL PRIVILEGES ON UserDb.* TO Username@localhost IDENTIFIED BY 'UserPassword';
FLUSH PRIVILEGES;
quit
```































