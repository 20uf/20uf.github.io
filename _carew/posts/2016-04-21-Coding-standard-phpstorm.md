---
title:  Le coding standard Symfony2 avec CodeSniffer et PHPStorm
layout: post
tags:
    - outils
    - développeur
    - Symfony2
    - Coding standard
    - CodeSniffer
---

Un point important concernant la qualité de code
------------------------------------------------

Pour s'assurer de la normalisation des projets, l'installation d'outil comme PHP [PHP_CodeSniffer](http://pear.php.net/package/PHP_CodeSniffer) facilite le respect des conventions de codage.

    pear install PHP_CodeSniffer

CodeSniffer inclut 8 standards par defaut: PSR2, Squiz, Zend, MySource, PHPCS, PSR1 and PEAR.
Nous allons ajouter le standard de Symfony2.


Le standart doit maintenant exister dans la liste:

    phpcs -i

Nous pouvons en définir un par défaut:

    phpcs --config-set default_standard Symfony2

Pour l'utiliser il suffit à de se rendre dans un de nos projets Symfony2 et de taper la commande suivante :

    phpcs src/


Configuration dans PHPStorm
---------------------------

Allez dans File > Settings > Languages & Frameworks > PHP > Code Sniffer  

Indiquez dans l'executable phpcs puis validez !

![CodeSniffer](/img/1_phpstorm_codesniffer.png)

Dernier point à finaliser, l'inspection:
File > Settings > Editor > Code Style > Inspections > PHP > PHP Code Sniffer validation
  
Activez le et mettez la sévérité forte.

![Inspection](/img/2_phpstorm_setting_inspections.png)
