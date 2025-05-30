---
title: "Quickstart Guide How to Install Docker a Raspberry Pi"
date:
  created: 2017-01-06

linktitle: "Installing Docker on a Raspberry Pi"
slug: "cloud-dienste-fur-private-anwender"

tags:
- Docker
- RaspberryPi
- "#HowTo"

authors:
- harry
---
# Docker on my Raspberry PI Cluster

<!-- more -->

## Installation

See here https://docs.docker.com/engine/installation/linux/debian/

Purge any older repositories (not necessary with a clean, new debian installation)
```
$ apt-get purge "lxc-docker*"
$ apt-get purge "docker.io*"
```

Update package information, ensure that APT works with the https method, and that CA certificates are installed.
```
$ apt-get update
$ apt-get install apt-transport-https ca-certificates gnupg2
```

Add the new GPG key.
```
$ sudo apt-key adv \
       --keyserver hkp://ha.pool.sks-keyservers.net:80 \
       --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

Edit `/etc/apt/sources.list.d/docker.list `
```
deb https://apt.dockerproject.org/repo debian-jessie main
```

Verify that APT is pulling from the right repository.
```
$ apt-cache policy docker-engine
docker-engine:
  Installed: (none)
  Candidate: 1.12.5-0~debian-jessie
  Version table:
     1.12.5-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main armhf Packages
     1.12.4-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main armhf Packages
     1.12.3-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main armhf Packages
     1.12.2-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main armhf Packages
     1.12.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main armhf Packages
```

Verify docker has been installed correctly
```
$ sudo docker run armhf/hello-world
Unable to find image 'armhf/hello-world:latest' locally
latest: Pulling from armhf/hello-world
a0691bf12e4e: Pull complete
Digest: sha256:9701edc932223a66e49dd6c894a11db8c2cf4eccd1414f1ec105a623bf16b426
Status: Downloaded newer image for armhf/hello-world:latest

Hello from Docker on armhf!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```
### Installation docker-machine
```
$ curl -L https://github.com/docker/machine/releases/download/v0.9.0-rc2/docker-machine-Linux-armhf >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine
```

```
$ docker-machine -v
docker-machine version 0.9.0-rc2, build 7b19591
```
### Installation docker-compose
```
$ apt-get install python-pip
```
```
$ pip install docker-compose
```
```
$ docker-compose version
docker-compose version 1.9.0, build 2585387
docker-py version: 1.10.6
CPython version: 2.7.9
OpenSSL version: OpenSSL 1.0.1t  3 May 2016
```
