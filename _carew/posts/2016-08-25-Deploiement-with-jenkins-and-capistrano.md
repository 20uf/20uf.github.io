---
title:  Déployer avec jenkins et capifony 2
layout: post
tags:
    - Jenkins
    - Capifony
---

Déployer avec jenkins et capifony 2
-----------------------------------

Installation de capifony

    gem install capifony
    gem install capistrano-ext

J'utilse une dépendance pour effectuer des taches avant la compression [Copy with triggers](https://github.com/facile-it/capistrano-strategy-copy-with-triggers)

    gem install 'capistrano-strategy-copy-with-triggers'

Capifony n'est pas compatible avec capistrano 3.

    gem uninstall capistrano

#### Liens externes:

[Scripts exemples](https://github.com/20uf/capifony)
[Capifony](http://capifony.org/)
