# (DOC) LIN2 - Quentin Th�o - Mini projet

### Configuration de la machine 

 * Debian 8
 * 20 Go
 * 512 Mo RAM

### Configuration de l'installation de Debian 
* Langue : anglais
* continent : europe
* Pays : Suisse
* Langage : en_GB_UTF8
* Nom de la machine : debian
* Nom de domaine : CeQueVousVoulez
* Nom de l'utilisateur : CeQueVousVoulez 
* Compte : CeQueVousVoulez
* Mot de passe : CeQueVousVoulez

### Options d'installation 
    Utiliser les param�tres recommand�s

### Modifications des sources Debian
Etant donn� que l'installation s'est effectu� __manuellement__ avec une image iso, 
nous devons modifier les sources pour que la machine ne pointe plus sur cette image.

    
### Installation du SSH 
``` 
apt-get install openssh-server
```
