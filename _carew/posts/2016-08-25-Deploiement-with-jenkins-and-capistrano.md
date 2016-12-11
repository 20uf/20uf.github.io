---
title:  Déployer Symfony avec jenkins et capifony 2
layout: post
tags:
    - Jenkins
    - Capifony
    - Symfony
---

Déployer Symfony avec jenkins et capifony 2
-----------------------------------

> Ce guide n'explique pas comment installer Jenkins, vous devez disposer d'un serveur déjà installé et configuré.

#### Installer Capifony

Connectez-vous en SSH sur votre serveur Jenkins et installez les dépendances suivantes:
> Installons aussi sur notre machine afin de déboguer et tester nos scripts.

Ruby 2:

    sudo apt-add-repository ppa:brightbox/ruby-ng
    sudo apt-get update
    sudo apt-get install ruby2.2 ruby2.2-dev

Capifony

    gem install capifony
    gem install capistrano-ext

Il peut être nécessaire d'effectuer des tâches sur le serveur Jenkins, comme par exemple les dépendances bowers.

Pour cela j'utilise une dépendance capistrano pour effectuer des tâches avant la compression [Copy with triggers](https://github.com/facile-it/capistrano-strategy-copy-with-triggers)

    gem install 'capistrano-strategy-copy-with-triggers'

#### Script de déploiement

> La bonne pratique est de créer un dépôt spécifique qui contient vos scripts.

Nous allons ici nous concentrer sur l'environnement de production, vous pourrez reproduire la même procédure pour les autres stagings.
Créons notre fichier Capfile où nous allons déclarer:
- Le niveau de trace
- Le chemin binaire PHP
- nos différents environnements "staging"

[Fichier Capfile](https://github.com/20uf/deployment-with-capifony/blob/master/Capfile)
```python
    load 'deploy' if respond_to?(:namespace)
    
    require 'capifony_symfony2'
    require 'capistrano-strategy-copy-with-triggers'
    
    logger.level = Logger::MAX_LEVEL
    
    # PHP binary path
    set :php_bin, "/usr/local/php5.6/bin/php"
    
    # Staging
    set :stages, %w(production staging testing)
    set :default_stage, "testing"
    set :stage_dir, "capifony/stages"
    require 'capistrano/ext/multistage'
```

Créons maintenant notre premier stage pour la production dans le dossier stages/
Je ne vais pas ici vous détailler les configurations, je vous invite à consulter [le site officiel](http://capifony.org/).

[Fichier production.rb](https://github.com/20uf/deployment-with-capifony/blob/master/stages/production.rb)
```python
    set :deploy_to, "/path/project/"
    
    set :branch, "master"
    set :symfony_env_prod, "prod"
    
    set :application, "demo-20uf"
    set :domain, "ssh.cluster.production.tld"
    set :user, "user-auth"
    
    set :use_sudo, false
    set :keep_releases, 2
    
    role :web, domain
    role :app, domain, :primary => true
    
    # Repository
    set :repository, "ssh://git@your.gitlab.tdl/project/demo.git"
    set :scm, :git
    set :deploy_via, :copy_with_triggers
    
    ## Symfony3 structure
    set :symfony_console, "bin/console"
    set :controllers_to_clear, ["app_*.php"]
    set :shared_files, ["app/config/parameters.yml"]
    set :use_composer, true
    set :composer_install_flags, '--no-dev --no-interaction --quiet --optimize-autoloader'
    set :composer_dump_autoload_flags, '--optimize'
    set :update_vendors, false
    set :copy_vendors, false
    set :writable_dirs, ["var/cache", "var/logs"]
    set :webserver_user, "www-data"
    set :use_set_permissions, true
    set :dump_assetic_assets, false
    set :model_manager, "doctrine" # ORM
```

Il faut initialiser la structure/ l'arborescence côté serveur, pour cela à partir de la racine où se trouve le capfile:

    cap deploy:setup

Ajoutons maintenant le fichier de paramètre symfony sur le serveur:

    ssh your_deploy_server
    mkdir -p /path/project/shared/app/config
    vim /path/project/shared/app/config/parameters.yml

Parfait, vous pouvez maintenant tester en exécutant le déploiement avec la commande:

    cap production deploy

Ajoutez ces scripts dans un dépôt git, celui-ci sera utilisé dans la prochaine étape.

#### Tâche Jenkins

> Les plugins Git plugin ou GitLab Plugin, Branch API Plugin, Git Parameter Plug-In, Active Choices Plug-in est requis sur votre serveur Jenkins.

Nous avons mis en place les scripts, testé en local, tout ça c'est bien, mais allons plus loin en permettant à d'autre de déployer.

Créons un nouveau Job jenkins free-style.

Cochez "Ce build a des parametres"
Ajouter pour nom "stage" et comme description "Staging"
Cochez "Types de paramètres de base" puis configurez en simple select avec un seul élément visible.
Mettez comme valeur dans "Choisissez la source pour la valeur" vos environnement séparé d'une virgule, par exemple: testing,staging,production
Mettez comme valeur dans "Choisissez Source pour la valeur par défaut" l'environnement par défaut, par exemple production.

![paramètres](https://raw.githubusercontent.com/20uf/deployment-with-capifony/master/assets/build_parameters.png)

Dans la gestion de code source, sélectionnez Git et ajouter le dépôt contenant les scripts précédemment créer.
Précisez la branche par défaut.

![paramètres](https://raw.githubusercontent.com/20uf/deployment-with-capifony/master/assets/build_gitlab.png)

Il reste à ajouter le script shell qui va exécuter capifony, selon le paramètre qui sera sélectionné par l'utilisateur.

[Build jenkins](https://github.com/20uf/deployment-with-capifony/blob/master/jenkins_build.sh)
```
    #!/bin/bash
    # Shell de deploiement
    
    # Close STDERR FD
    exec 2<&-
    # Redirect STDERR to STDOUT
    exec 2>&1
    
    MAGENTA="\033[0;35m"
    COLOREND="\e[0m"
    BLUE="\e[34m"
    GREEN="\e[32m"
    
    echo "-------------------------------------------------------------------------------------------------------------------------------------------"
    echo -e "$BLUE" ">>> Task execution ${JOB_NAME} stage: ${stage}" "$COLOREND"
    echo "-------------------------------------------------------------------------------------------------------------------------------------------"
    
    cap ${stage} deploy
    
    echo "-------------------------------------------------------------------------------------------------------------------------------------------"
    echo -e "$BLUE" "End of deployment ${JOB_NAME} `date`" "$COLOREND"
    echo "-------------------------------------------------------------------------------------------------------------------------------------------"
```

Sauvegardez et le tour est joué !

#### Tâches utiles



#### Liens externes:

[Capifony](http://capifony.org/)
[Scripts exemples](https://github.com/20uf/deployment-with-capifony)
[Jenkins](https://jenkins.io/)
