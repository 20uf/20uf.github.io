---
title:  Optimiser son CLI PHP en désactivant XDEBUG
layout: post
tags:
    - PHP CLI
    - Xdebug
    - Performance
---

Optimiser son CLI PHP en désactivant XDEBUG
-------------------------------------------

[Xdebug](https://xdebug.org/) est un outil incontournable, mais il peut être pénalisant lorsqu'on lance un composer, ou quelconques taches en CLI.

Jusqu'à aujourd'hui je le gardais activé n'ayant pas de solution lors de mes utilisations du coverage de PHPunit, celui-ci étant requis.

Bon c'est vrai je n'avais pas spécialement creusé le sujet.. souvent pris par d'autres priorités. 

Mais maintenant c'est fini ! Je me suis décidé d'arrêter de négliger ce point, xdebug à des effets de ralentissement sur mes pipelines CI.

Allez on attaque le sujet :)

On commence par désactiver xdebug en mode CLI:

Pour PHP5:

    $ php5dismod -s cli xdebug

Pour PHP7

    $ phpdismod -s cli xdebug

Et hop déjà on souffle, maintenant il reste à lancer une commande en activant xdebug:

    $ /usr/bin/php -dzend_extension=xdebug.so -dxdebug.remote_enable=1 <votre commande>

On peut se simplifier la vie avec un alias:

    $ alias php-xdebug='/usr/bin/php -dzend_extension=xdebug.so -dxdebug.remote_enable=1'

Je vous conseille d'ajouter directement la ligne ci-dessus dans le `.bashrc`

    $ vim ~/.bashrc

C'est parfois si simple.. la preuve !

    $ php-xdebug vendor/bin/phpunit --coverage-html build/
    PHPUnit 5.7.21 by Sebastian Bergmann and contributors.
    
    ...............................................................  63 / 110 ( 57%)
    ...............................................                 110 / 110 (100%)
    
    Time: 7.79 seconds, Memory: 35.75MB
    
    OK (110 tests, 162 assertions)

