---
title:  Utilisez docker pour simplifier votre vie!
layout: post
tags:
    - Docker
    - Docker-compose
    - Sandbox
---

Utilisez docker pour simplifier votre vie!
-------------------------------------------

#### Pourquoi s'orienter sur du docker.

Dans le cadre de mes projets j'ai été confronté à ce que chacun de mes développeurs se retrouve avec des configurations différentes sur leur poste de travail. 

Selon les projets cette pratique peut être problématique, dans mon cas j'ai une application centrale qui permet l'authentification entre plusieurs produits et qui exploitent du [SSO]. 
La contrainte principale est le partage de session, les vhosts doivent être **identiques**.

Je peux aussi citer d'autres arguments:
* L'installation d'un ou plusieurs projets prennent plus d'une journée pour chaque nouveau arrivants, seulement quelques minutes avec Docker.
* Les configurations entre chaque développeurs ne sont pas ISO, avec Docker les ressources sont partagées.

La stack Docker peut être pénalisante sur des systemes autres que Linux, comme par exemple l'utilisation d'une VM les performances sont réduites.
C'est le cas avec Mac OS. Windows Pro propose maintenant [Hyper V], n'utilisant aucun des deux il m'est difficile de vous donner des conseils.

#### Quelle approche aborder ?

La philosophie de [Docker] est de lancer un processus par conteneur. 
En tant que développeur je préfère avoir une approche différente, par services.

Je connais mon silo, à savoir:
- Un clusteur Apache / Nginx et PHP-x installé dessus
- Un clusteur Mysql / MariaDb / MongoDb..
- Un clusteur Memcache / Redis..

Mon objectif est de s'approcher au maximum de cet environnement.

![Silo](/img/1_silo_docker.png)

Pour cela je vais utiliser [Docker-compose] et [Make] pour simplifier la gestion:
- Installer le projet
- Lancer les containers
- Arrêter les containers
- Manipuler un container (lancer des tâches par exemple)

Ma pratique est d'avoir un package `Docker` à la racine de mes projets, on peut y placer des `Dockerfile` qui peuvent être construites à la demande. 
Cependant je préfère construire mes images en amont afin d'économiser le temps de construction à chaque installation. 

Nous avons (mon équipe) créé un dépôt [PHPDocker] qui contient une série d'images, cela nous permet de pouvoir switcher facilement et simplement sur l'une d'elle.
Ces images représentes la partie serveur web avec son langage (PHP) et des configurations adaptées à nos besoins.

#### Mise en place du Silo

Nos produits sont développés sous Symfony, nos exemples sont donc basés dessus, vous pouvez bien évidemment reproduire pour tout autre type de Framework ou pas.

Notre fichier docker `docker-compose.yml` se trouvant à la racine du projet.
Je vous invite à consulter la [documentation officielle][docker-compose-refs] concernant l'utilisation et les références.

```yaml
version: "2"

services:
    app:
        image: dockerphp/nginx:7.1-stretch
        depends_on:
            - db
            - memcached_1
            - memcached_2
        volumes:
            - .:/app
        ports:
            - "8080:443"
        extra_hosts:
         - "sso.domain-dev.fr:172.17.0.1" # L'ip du docker engine

    db:
        image: mariadb:10.3
        environment:
            v: root
        ports:
            - "3301:3306"

    memcached_1:
        image: memcached:latest
        ports:
          - "11210:11211"
        mem_limit: 1g

    memcached_2:
        image: memcached:latest
        ports:
          - "11211:11211"
        mem_limit: 1g

    pma:
        image: phpmyadmin/phpmyadmin
        depends_on:
            - db
        environment:
            PMA_HOST: db
            PMA_USER: root
            PMA_PASSWORD: root
        ports:
            - "127.0.0.1:8081:80"
```

Détaillons un peu cette configuration:

`app`: 
- Nous chargeons l'image avec Nginx et PHP 7.1.
- Nous avons trois liens, un sur la base de données et deux sur memcached.
- le port 80 du container sera accessible sur le port 8080. (https://localhost:8080)
- Nous ajoutons dans le `/etc/hosts` le domaine `sso.domain-dev.fr` sur l'ip du docker engine.

`db`: 
- Nous chargeons l'image officielle de mariadb sur la version 10.3.
_ On déclare avec la variable d'environnement `MYSQL_ROOT_PASSWORD` le mot de passe `root`.
- le port 3306 du container sera accessible sur le port 3301.

`memcached_1` et `memcached_2`: 
- Nous chargeons l'image officielle de memcached sur sa dernière version.
- On assigne 1g de mémoire à memcached .
- le port 11211 du container sera accessible sur le port 11210 et 11211.

`pma`: 
- Nous chargeons l'image officielle de phpmyadmin sur sa dernière version.
- Nous avons un lien avec la base de données.
- On déclare avec les variables d'environnements pour que phpmyadmin se connecte à la base de données.
- le port 80 du container sera accessible sur le port 8081. (http://localhost:8081)

Nous allons maintenant créer un fichier `Makefile` à la racine du projet, celui-ci va nous simplifier la vie pour manipuler les containers.

```bash
# Do not change
FIG=docker-compose
RUN=$(FIG) run --rm app
EXEC=$(FIG) exec app
CONSOLE=bin/console

.DEFAULT_GOAL := help
.PHONY: help start stop restart clear cc bash host deps

help:
	@echo ''
	@fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v fgrep | sed -e 's/\\$$//' | sed -e 's/##//'
	@echo ''

## Setup
##---------------------------------------------------------------------------
install:        ## Install and start the project
install: up vendor app/config/parameters.yml yarn info

host:           ## Add the application host to your configuration
	@echo 'Adding the local domain to /etc/hosts'
	sudo sh -c  'echo "127.0.0.1       my-app.domain-dev.fr" >> /etc/hosts'

##
## Provisioning
##---------------------------------------------------------------------------
start:          ## start the project
start: up info

stop:           ## Stop docker containers
	$(FIG) kill

restart:        ## Restart the whole project
restart: stop start warmup

bash:           ## Switch to the bash container of the application
	@$(RUN) bash

##
## Symfony
##---------------------------------------------------------------------------
clear:          ## Remove all the cache, the logs, the sessions and the built assets
clear: warmup

cc:             ## Clear the cache in dev env
cc:
	$(RUN) $(CONSOLE) cache:clear --no-warmup

clean:          ## Clean package nodejs
clean: clean_project

console:        ## Launch Console
	$(CONSOLE) $(filter-out $@,$(MAKECMDGOALS))

composer:       ## Launch Composer
	$(APP) composer $(filter-out $@,$(MAKECMDGOALS))

##
## Dependencies
##---------------------------------------------------------------------------

deps:           ## Install the project PHP and JS dependencies
deps: vendor

# Internal rules

up:
	$(FIG) up -d --remove-orphans

warmup:
	-$(EXEC) rm -rf var/cache/*
	-$(EXEC) rm -rf var/sessions/*
	-$(EXEC) rm -rf var/logs/*

vendor: composer.lock
	@$(RUN) composer install

composer.lock: composer.json
	@echo compose.lock is not up to date.

app/config/parameters.yml: app/config/parameters.yml.dist
	@$(RUN) composer run-script post-install-cmd --no-interaction

node_modules: yarn.lock
	@$(RUN) yarn run assets

yarn:
	-$(EXEC) npm run yarn:assets
	
yarn.lock: package.json
	@echo yarn.lock is not up to date.

%:
@:
```

Vous remarquez qu'il est assez simple d'ajouter de nouvelles tâches.
Vous pouvez maintenant consulter les tâches disponibles avec la commande:

    $ make

Vous aurez pour résultat:

```bash
 Setup
---------------------------------------------------------------------------
install:         Install and start the project
host:            Add the application host to your configuration

 Provisioning
---------------------------------------------------------------------------
start:           start the project
stop:            Stop docker containers
restart:         Restart the whole project
bash:            Switch to the bash App container of the application

 Cache
---------------------------------------------------------------------------
clear:           Remove all the cache, the logs, the sessions and the built assets
cc:              Clear the cache in dev env
clean:           Clean package nodejs

 Dependencies
---------------------------------------------------------------------------
deps:            Install the project PHP and JS dependencies
```

Vous l'aurez compris, maintenant l'installation d'un projet semble simple et rapide, voici en quelques lignes ce que représentent l'installation d'un projet:

    $ git clone git@gitlab.entreprise.com:namespace/project_name.git
    Cloning into 'project_name'...
    remote: Counting objects: 22512, done.
    ....
    $ make install
    cd social_live/ && make install
    docker-compose up -d --remove-orphans
    Creating network "project_name_default" with the default driver
    Pulling memcached (memcached:latest)...
    latest: Pulling from library/memcached
    94ed0c431eb5: Pull complete
    ...
    7.1-stretch: Pulling from dockerphp/nginx
    ...

Une fois terminée l'application est disponible sur https::/my-app.domain-dev.fr:8080
PhpMyAdmin sur http://my-app.domain-dev.fr:8081
Mysql tourne sur le port 3301 si vous utilisez MysqlWorkbench.

Vous en pensez quoi ? Pas mal hein ? :)

[SSO]: https://fr.wikipedia.org/wiki/Authentification_unique
[Docker]: https://www.docker.com/
[Docker-compose]: https://docs.docker.com/compose/
[Make]: https://www.gnu.org/software/make/
[PHPDocker]: https://github.com/php-docker/nginx
[Hyper V]: https://fr.wikipedia.org/wiki/Hyper-V
[docker-compose-refs]: https://docs.docker.com/compose/compose-file/
