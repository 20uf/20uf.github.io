---
title:  Installer Docker et Docker-Compose
layout: post
tags:
    - Docker
    - Docker-compose
    - Installation
---

Installation de docker et docker-compose
-------------------------------------------

Petit cheatsheet pour l'installation de Docker sur Mint ou Ubuntu xenial.

On ajoute la clef et le dépôt:
 
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo su
    * apt-key fingerprint 0EBFCD88
    $ add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"

On met à jour.

    $ apt update
 
On recherche le paquets `docker-ce`.

    $ apt-cache madison docker-ce

```bash
    docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.09.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.06.2~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.06.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.06.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.03.2~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
    docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
```

On installe la dernière version.

    $ apt-get update docker-ce=17.12.0~ce-0~ubuntu

On vérifie que l'installation est correcte.

    $ docker version

```bash
Client:
 Version:	17.12.0-ce
 API version:	1.35
 Go version:	go1.9.2
 Git commit:	c97c6d6
 Built:	Wed Dec 27 20:11:19 2017
 OS/Arch:	linux/amd64

Server:
 Engine:
  Version:	17.12.0-ce
  API version:	1.35 (minimum version 1.12)
  Go version:	go1.9.2
  Git commit:	c97c6d6
  Built:	Wed Dec 27 20:09:53 2017
  OS/Arch:	linux/amd64
  Experimental:	false
```

On installe Docker-compose

    $ apt install docker-compose

```bash
    docker-compose version 1.8.0, build unknown
    docker-py version: 1.9.0
    CPython version: 2.7.12
    OpenSSL version: OpenSSL 1.0.2k  26 Jan 2017
```
