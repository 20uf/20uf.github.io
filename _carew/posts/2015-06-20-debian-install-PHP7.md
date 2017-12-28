---
title:  installer PHP 7 pour apache avec les dépôts dotdeb
layout: post
tags:
    - Jessie
    - Dotdeb
    - PHP7
---

installer PHP 7 pour apache avec les dépôts dotdeb 
---------------------

Ajouter les dépots dotdeb:

     echo "deb http://packages.dotdeb.org jessie all" > /etc/apt/sources.list.d/dotdeb.list
     wget https://www.dotdeb.org/dotdeb.gpg && apt-key add dotdeb.gpg

Ou 

    apt-get install python-software-properties && LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php

Mettre à jour les dépots

     apt-get update && apt-get upgrade

Vérification de la présence de php7: 

    apt-cache policy php7.0

Si vous migrez à partir de php5, nettoyez les paquets:

    apt-get --purge remove php5* && apt-get autoremove

On installe PHP7 & FPM

    apt install php7.0 php7.0-fpm

On contrôle la version:

    php -v

    > PHP 7.0.7-4+deb.sury.org~trusty+1 (cli) ( NTS )
    > Copyright (c) 1997-2016 The PHP Group
    > Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
    > with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies

Configuration
#####

On modifie le php.ini:

    vim /etc/php/7.0/fpm/php.ini

Ligne 768:

    cgi.fix_pathinfo=0

Ligne 798:

    upload_max_filesize = 32M

Ligne 656:

    post_max_size = 32M

Ligne 912:

    date.timezone = Europe/Paris

Ligne 1306:

    session.save_path = "/tmp"

On modifie www.conf

    vim /etc/php/7.0/fpm/pool.d/www.conf

Ligne 37

    listen = /run/php/php7.0-fpm.sock

par:

    listen = 127.0.0.1:9000

On installe les modules supplémentaire:

    apt-get install php7.0-cli php7.0-mysql php7.0-common php7.0-curl php7.0-opcache php7.0-json php-xml php-xdebug php7.0-mbstring libapache2-mod-php7.0 php7.0-dev php7.0-mcrypt php7.0-memcache php7.0-readline php7.0-tidy

On relance les services:

    service apache2 restart && service php7.0-fpm restart
