---
title: "Distributed Minio with Docker"
date:
  created: 2017-01-09

linktitle: "Distributed Minio with Docker"
slug: "distributed-minio-with-docker"

description: "Running distributed Minio Docker containers on a Raspberry Pi Cluster."

tags:
- Minio
- Docker
- RaspberryPi

authors:
- harry
---
![](https://pbs.twimg.com/media/CycsocEWQAA0y94.png#c)
Ok, I think I will switch to english for the next posts ;-)

I've created an [Minio](https://minio.io) Docker-Image for ARM. Which can be found on [GitHub](https://github.com/pixelchrome/minio-arm) or on [Dockerhub](https://hub.docker.com/r/pixelchrome/minio-arm/).

Today I want to show you how to run Minio it in [distributed mode](http://docs.minio.io/docs/distributed-minio-quickstart-guide). The distributed mode can give you high availability and data protection.

<!-- more -->

# Preparing the Nodes
## Storage Directory
You need a directory on each node to store the data and an additional one for storing the configuration.
```
$ sudo mkdir -p /export/minio
$ sudo mkdir /export/minio-config
```

## Docker pull command
On each Node you need to pull the minio-docker-arm image.

```
$ sudo docker pull pixelchrome/minio-arm
```

# Docker run command
And as a last command it is necessary to run the container on each node
```
$ sudo docker run --rm -p 9000:9000 --name minio \
  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \
  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  -v /export/minio:/export \
  -v /export/minio-config:/root/.minio \
  --net=host \
  pixelchrome/minio-arm server \
  http://pi1/export http://pi2/export \
  http://pi3/export http://pi4/export \
  http://pi5/export http://pi6/export
```

# Information
Details how to run Minio in distributed mode, can you find on the [Documents-Website](http://docs.minio.io/docs/distributed-minio-quickstart-guide)

![Distributed Minio Docker Screenshot](../images/20170109-distributed_minio_docker_small.jpg)
