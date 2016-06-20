---
title:  Optimisez votre front avec Symfony
layout: post
tags:
    - uglify-js
    - compass
    - sass
---
Optimisez votre front avec Symfony
----------------------------------

Il faut installer uglify-js

    npm install uglify-js -g

Ainsi que uglifycss:

    npm install uglifycss -g
    

Dans votre fichier de configuration config.yml

    assetic:
        debug:          "%kernel.debug%"
        use_controller: '%kernel.debug%'
        bundles:        [ AppBundle ]
        ruby: /usr/local/bin/ruby
        filters:
            cssrewrite: ~
            sass:
                bin: /usr/local/bin/sass
            compass:
                bin: /usr/local/bin/compass
                cache_location: '%kernel.cache_dir%'
            uglifyjs:
                bin: /usr/bin/uglifyjs
            uglifycss:
                bin: /usr/bin/uglifycss

+ Documentations:
    + http://symfony.com/doc/current/best_practices/web-assets.html
    + http://symfony.com/doc/current/cookbook/assetic/uglifyjs.html
    + http://symfony.com/doc/current/cookbook/assetic/uglifyjs.html