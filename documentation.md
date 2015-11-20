# (DOC) LIN2 - Quentin Théo - Mini projet

## Configuration de la machine 

 * Debian 8
 * 20 Go
 * 512 Mo RAM

## Configuration de l'installation de Debian 
* Langue : anglais
* continent : europe
* Pays : Suisse
* Langage : en_GB_UTF8
* Nom de la machine : debian
* Nom de domaine : CeQueVousVoulez
* Nom de l'utilisateur : CeQueVousVoulez 
* Compte : CeQueVousVoulez
* Mot de passe : CeQueVousVoulez

## Options d'installation 
    Utiliser les paramètres recommandés

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
Mettre en commentaires la 1ère et 2ème ligne à l'aide du #.
Puis y insérer la ligne générée par debgen soit : 
```    
2. deb http://ftp.ch.debian.org/debian stable main
``` 
## Installation du SSH 
``` 
apt-get install openssh-server
```
