# (DOC) LIN2 - Quentin Théo 


Ce document décrit l'installation d'un serveur web avec les services suivant : 
 * SSH
 * Nginx
 * PHP-FPM
 * MariaDB
 
## Configuration de la machine 
 * Debian 8
 * 20 Go
 * 512 Mo RAM

## Configuration de l'installation de Debian 
* Langue : anglais
* Continent : europe
* Pays : Suisse
* Langage : en_GB_UTF8
* Nom de la machine : debian
* Nom de domaine : CeQueVousVoulez
* Nom de l'utilisateur : CeQueVousVoulez 
* Compte : CeQueVousVoulez
* Mot de passe : CeQueVousVoulez

## Création d'un utilisateur
```
adduser NomUtilisateur
```
Spécifier un mot de passe puis valider toutes les informations suivantes jusqu'à la création du compte.

## Modification des sources Debian
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
Mettre en commentaires les sources pointant sur le cd-rom #.
Puis y insérer la ligne générée par debgen soit : 
```    
2. deb http://ftp.ch.debian.org/debian stable main
``` 
Sauver et fermer le fichier puis lancer la commande suivante pour que le système applique ces modifications.
```
apt-get update
```

## Installation du SSH 
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

Redémarrer le service ssh

## Installation de Nginx
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

insérer à l'intérieur des accolades : 

```
fastcgi_pass unix:/var/run/php5-fpm.sock;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name$;
include fastcgi_params;
```

#### Configuration par utilisateur
Pour chaque utilisateur il est nécessaire de créer un fichier de configuration.
Ce fichier permet de spécifier l'endroit où l'utilisateur peut déposer ses fichiers afin que le serveur puisse les traiter. 
Ce fichier doit se trouver dans le dossier __/etc/nginx/conf.d__ de Nginx. Le nom du fichier doit être le même que le nom de l'utilisateur. Exemple : __NomUtilisateur.conf__

Veuillez insérer les informations suivantes dans le fichier :

```
server {
    listen 80;
    root /usr/share/nginx/www/NomUtilisateur;
    index index.php index.html index.htm;
    server_name NomUtilisateur;
    location / {
            try_files $uri $uri/ /index.php;
    }
    location ~ \.php$ {
            try_files $uri =404;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
            fastcgi_pass unix:/var/run/NomUtilisateur.sock;
    }
}
```

Redemarrer le service

```
/etc/init.d/nginx restart
```
## Droits utilisateur
Comme nous avons décidé précédement que le dossier __www__ sera le dossier où les utilisateurs vont déposer leur site web, il est nécessaire de le créer dans __/usr/share/nginx/__
Afin que chaque utilisateur ne puisse pas voir les dossiers des autres, nous avons retirer les droits de lecture sur le dossier __www__ au groupe d'utilisateurs __other__. 
```
chmod o-r /usr/share/nginx/www
```

Il est nécessaire lors de la création du dossier de l'utilisateur, de le nommer propriétaire de son dossier à l'aide de la commande __chown__.
```
chown NomUtilisateur DossierUtilisateur
```

Afin d'améliorer la sécurité interne il est indispensable d'enlever les droits d'exécution au groupe __other__ pour chaque dossier des utilisateurs. Ainsi ils n'auront accès qu'a leur propre répertoire.
```
chmod o-x /usr/share/nginx/www/*
```


## Installation de PHP5-fpm et PHP5-MySql
```
apt-get install php5-fpm php5-mysql
```

### Configuration PHP5-fpm
Modifier le fichier __/etc/php5/fpm/php.ini__

A ligne suivante :

```
773: cgi.fix_pathinfo = 1
```

Cela va ralentir la requête mais la rend plus sûre.

Il est maintenant nécessaire de changer le port d'écoute de php5 qui pointe sur son socket.
Dans le fichier __/etc/php5/fpm/pool.d/www.conf__, décommenter, si nécessaire, la ligne suivante :
```
listen = /var/run/php5-fpm-sock
```
### Configuration par utilisateur
Comme pour nginx, il est nécessaire de mettre en place un fichier de configuration propre à chaque utlisateur.
Faire une copie du fichier __www.conf__ en le nomant __NomUtilisateur.conf__. Modifier les informations suivantes:

```
[www] -> [NomUtilisateur]
user = NomUtilisateur
group = NomUtilisateur
[...]
listen = /var/run/NomUtilisateur.sock
[..]
listen.owner = NomUtilisateur
listen.group = NomUtilisateur
```

Relancer le service

```
/etc/init.d/php5-fpm restart
```

## Installation de MariaDB
 ```
apt-get install mariadb-server mariadb-client
```

Suivre les instructions pour configurer le compte root

### Configuration de MariaDB
afin d'améliorer la sécurité de MariaDB, exécuter la commande :

```
mysqsl_secure_installation
```

Celle-ci va supprimer les comptes anonymes, réinitialisé le mot de passe root, empêcher la connexion en root en dehors de localhost.

### Création d'un utilisateur
Connexion en tant que root
```
mysql -u root -p
```

Création d'une base l'utilisateur lui accordant tout les droits sur celle-ci
```
CREATE DATABASE UtilisateurDb;
GRANT ALL PRIVILEGES ON UtilisateurDb.* TO NomUtilisateur@localhost IDENTIFIED BY 'UserPassword';
FLUSH PRIVILEGES;
quit
```

Ces privilèges permettent à l'utilisateur d'avoir accès seulement à sa base de données.































