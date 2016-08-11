# About Docker

## Installation

This document is guide to compile docker from source

## Prerequisites

To build docker, we needs to install docker.
Building docker is done by docker container.

~~~bash
docker -v
~~~

If docker does not exist, install docker first.

## Get docker source

Get the latest Docker source from github


~~~bash
git clone https://git@github.com/docker/docker
cd docker
make build
~~~

