---
title:  Se simplifier la vie avec vos clefs ssh
layout: post
tags:
    - ssh
    - debian
---

Organisez vos clefs ssh
-----------------------

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
   
| commande   |      paramètre      | | description |
|----------|:-------------:|------||:------|
| hote |  hostname/ip |      | adresse de la machine |
| RSAAuthentication  | yes/no |       | authentification RSA, clé publique/privé généré avec "ssh-keygen"
| PubKeyAuthentication | yes/no |       | s'authentifier avec une clé public |
| PasswordAuthentication | yes/no |       | autorise l'authentification de base avec mot de passe (a pas confondre avec passphrase) |
| CheckHostIP | yes/no |       | Verifie l'IP host n'est pas une usurpation DNS |
| IdentityFile | ~/.ssh/id_dsa |       | défini la clé privé a utiliser pour s'authentifier |
| User | user |       | compte à utiliser pour l'identification |
| Port | port |       | numéro de port à utiliser |
| HashKnownHosts | yes/no |       | Permet d'avoir un fichier known_hosts plus lisible |