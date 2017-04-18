---
title:  Débugger avec Xdebug
layout: post
tags:
    - phpstorm
    - xdebug
    - php
---

Débugger avec Xdebug
-----------------------------------

> J'utilise pour ce guide l'IDE PhpStorm 2016.3. 

#### Installer xdebug

Sur Ubuntu ou ses dérivés:

    sudo apt-get install php-xdebug

Pour les autres distributions voir la (documentation officielle)[https://xdebug.org/docs/all]

On controle que l'extension est installée:

    php -m | grep xdebug

On active le debug distant e, éditant le fichier de configuration php.ini

    xdebug.remote_enable=On         # Activer le debug distant
    xdebug.remote_connect_back=On   # xDebug va automatiquement se connecter sur l'IP présente dans $_SERVER['REMOTE_ADDR']
    xdebug.remote_host=localhost    # Host à contacter si remote_connect_back est désactivé ou dans un contexte CLI
    xdebug.remote_port=9000         # Port par défaut
    xdebug.idekey=idekey            # Identifiant de session utilisé par l'IDE


Configuration de PHPStorm
-----------------------------------

On va dans Run > Edit configurations, puis cliquer sur le symbole « + »
Dans la liste on clique sur « PHP Remote Debug ».

