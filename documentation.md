# (DOC) LIN2 - Quentin Th�o 


Ce document d�crit l'installation d'un serveur web avec les services suivant : 
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

## Cr�ation d'un utilisateur
```
adduser NomUtilisateur
```
Sp�cifier un mot de passe puis valider toutes les informations suivantes jusqu'� la cr�ation du compte.

## Modification des sources Debian
Etant donn� que l'installation s'est effectu� __manuellement__ avec une image iso, 
nous devons modifier les sources pour que la machine ne pointe plus sur cette image.
Aller sur le site de [debgen](http://debgen.simplylinux.ch/). 

S�lectionner les informations suivantes : 
* Switzerland
* Stable (wheezy)
* 64 bits
* Cocher les cases __Main__ et __Security__

Puis cliquer sur  __Generate sources-list__

Une pop-up s'ouvre alors et propose une ligne de commande � copier dans le fichier __sources.list__.

### Marche � suivre
```
1. nano /etc/apt/sources.list
``` 
Mettre en commentaires les sources pointant sur le cd-rom #.
Puis y ins�rer la ligne g�n�r�e par debgen soit : 
```    
2. deb http://ftp.ch.debian.org/debian stable main
``` 
Sauver et fermer le fichier puis lancer la commande suivante pour que le syst�me applique ces modifications.
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
Modifier dans la rubrique __Authentification__ pour �viter que l'on puisse se connecter en __root__ au serveur.
```
permitRootLogin : NO
```

Red�marrer le service ssh

## Installation de Nginx
```
apt-get install nginx
```

### Configuration Nginx
Premi�rement, il faut �diter le fichier de configuration Nginx:
```
nano /etc/nginx/sites-available/default
```

Dans ce fichier, remplacer les informations suivantes :

```
root /usr/share/nginx/www/;
server_name VotreAdresseIp ou NomDeDomaine;
```

D�commenter

``` 
location ~\.php$ {.......}
```

ins�rer � l'int�rieur des accolades : 

```
fastcgi_pass unix:/var/run/php5-fpm.sock;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name$;
include fastcgi_params;
```

#### Configuration par utilisateur
Pour chaque utilisateur il est n�cessaire de cr�er un fichier de configuration.
Ce fichier permet de sp�cifier l'endroit o� l'utilisateur peut d�poser ses fichiers afin que le serveur puisse les traiter. 
Ce fichier doit se trouver dans le dossier __/etc/nginx/conf.d__ de Nginx. Le nom du fichier doit �tre le m�me que le nom de l'utilisateur. Exemple : __NomUtilisateur.conf__

Veuillez ins�rer les informations suivantes dans le fichier :

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
Comme nous avons d�cid� pr�c�dement que le dossier __www__ sera le dossier o� les utilisateurs vont d�poser leur site web, il est n�cessaire de le cr�er dans __/usr/share/nginx/__
Afin que chaque utilisateur ne puisse pas voir les dossiers des autres, nous avons retirer les droits de lecture sur le dossier __www__ au groupe d'utilisateurs __other__. 
```
chmod o-r /usr/share/nginx/www
```

Il est n�cessaire lors de la cr�ation du dossier de l'utilisateur, de le nommer propri�taire de son dossier � l'aide de la commande __chown__.
```
chown NomUtilisateur DossierUtilisateur
```

Afin d'am�liorer la s�curit� interne il est indispensable d'enlever les droits d'ex�cution au groupe __other__ pour chaque dossier des utilisateurs. Ainsi ils n'auront acc�s qu'a leur propre r�pertoire.
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

Cela va ralentir la requ�te mais la rend plus s�re.

Il est maintenant n�cessaire de changer le port d'�coute de php5 qui pointe sur son socket.
Dans le fichier __/etc/php5/fpm/pool.d/www.conf__, d�commenter, si n�cessaire, la ligne suivante :
```
listen = /var/run/php5-fpm-sock
```
### Configuration par utilisateur
Comme pour nginx, il est n�cessaire de mettre en place un fichier de configuration propre � chaque utlisateur.
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
afin d'am�liorer la s�curit� de MariaDB, ex�cuter la commande :

```
mysqsl_secure_installation
```

Celle-ci va supprimer les comptes anonymes, r�initialis� le mot de passe root, emp�cher la connexion en root en dehors de localhost.

### Cr�ation d'un utilisateur
Connexion en tant que root
```
mysql -u root -p
```

Cr�ation d'une base l'utilisateur lui accordant tout les droits sur celle-ci
```
CREATE DATABASE UtilisateurDb;
GRANT ALL PRIVILEGES ON UtilisateurDb.* TO NomUtilisateur@localhost IDENTIFIED BY 'UserPassword';
FLUSH PRIVILEGES;
quit
```

Ces privil�ges permettent � l'utilisateur d'avoir acc�s seulement � sa base de donn�es.































