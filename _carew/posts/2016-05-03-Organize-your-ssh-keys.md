---
title:  Se simplifier la vie avec vos clefs ssh
layout: post
tags:
    - ssh
    - debian
---

Organisez vos clefs ssh
-----------------------

#### Générer une clé privée et une clé publique

Une clé SSH vous permet d'établir une connexion sécurisée entre votre ordinateur et un serveur.

Pour générer une nouvelle clé SSH, utilisez la commande suivante:

    ssh-keygen -t rsa -C "votre@email.com"

Cette commande vous demandera un emplacement et un nom pour stocker la paire de clés et un mot de passe. 
Je conseille d'utiliser l'emplacement par defaut correspondant et de nommer votre clé selon son utilité, par exemple pour github 

    ~/.ssh/id_github

Une seconde question vous demande un passphrase, il est fortement recommandé d'en utiliser un

    Enter passphrase (empty for no passphrase): **************

Vous devriez avoir un retour de ce type:

    Your identification has been saved in /home/demo/.ssh/id_rsa.
    Your public key has been saved in /home/demo/.ssh/id_rsa.pub.
    The key fingerprint is:
    4a:dd:0a:c6:35:4e:3f:ed:27:38:8c:74:44:4d:93:67 demo@a
    The key's randomart image is:
    +--[ RSA 2048]----+
    |          .oo.   |
    |         .  o.E  |
    |        + .  o   |
    |     . = = .     |
    |      = S = .    |
    |     o + = +     |
    |      . o + o .  |
    |           . o   |
    |                 |
    +-----------------+

Votre clé est maintenant générée et prête à defaut utilisée.

Si vous souhaitez copier votre clé sur un serveur distant, vous pouvez procéder ainsi:

    cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"

Une fois que vous avez copié vos clés SSH vers votre serveur, vérifier que vous pouvez vous connecter avec les clés SSH:

    ssh user@host


#### Configurer ses clés

Une astuce pour se simplifier la vie avec vos clefs ssh, creez un fichier dans la dossier .ssh :

    vim ~/.ssh/config

Créez une configuration differentes pour chaque serveurs ssh:

    Host github.com
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa

    Host gitlab.domain.fr
        HostName gitlab.domain.fr
        User git
        IdentityFile ~/.ssh/id_other_rsa

Et le tour est joué !


#### Quelques options:
   
| Commande   |      Paramètre      | Description |
|:----------|:-------------:|------:|
| `hote` |  hostname/ip | adresse de la machine |
| `RSAAuthentication`  | yes/no | authentification RSA, clé publique/privé généré avec "ssh-keygen"
| `PubKeyAuthentication` | yes/no | s'authentifier avec une clé public |
| `PasswordAuthentication` | yes/no | authentification avec mot de passe (a pas confondre avec passphrase) |
| `CheckHostIP` | yes/no | Verifie l'IP host n'est pas une usurpation DNS |
| `IdentityFile` | ~/.ssh/id_dsa | défini la clé privé a utiliser pour s'authentifier |
| `User` | user | compte à utiliser pour l'identification |
| `Port` | port | numéro de port à utiliser |
| `HashKnownHosts` | yes/no | Permet d'avoir un fichier known_hosts plus lisible |


#### Sécurité sur votre serveur SSH

Le login root ne doit pas être autorisée par l'intermédiaire des clés SSH.

Pour ce faire, ouvrez le fichier SSH config sur votre serveur distant:

    vim /etc/ssh/sshd_config

Dans ce fichier, trouvez la ligne qui comprend PermitRootLogin et modifiez le pour garantir que seuls les utilisateurs peuvent se connecter avec leur clé SSH:
PermitRootLogin sans mot de passe

    PermitRootLogin without-password

Mettez les changements en vigueur:

    reload ssh
