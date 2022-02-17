# Dockerfiles

### Simple (git - net-tools - iputils-ping - python3 - gcc)

```docker
FROM ubuntu:18.04

RUN apt-get update && apt-get install -y git \
	net-tools \
	iputils-ping \
	python3 \
	gcc \
	python-pip \
	wget \
	curl \
	golang \
	nano
```

### Apache server with cmd.php file

```docker
FROM php:7.0-apache

MAINTAINER levi <levi@gmail.com>

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y net-tools
```
