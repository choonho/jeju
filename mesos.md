# Mesos Setup

# Environment

Keyword | Value
----    | -----
ZOOKEEPER | 127.0.0.1

# Setup Mesos

~~~bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF
DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')
CODENAME=$(lsb_release -cs)

echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" | \
  sudo tee /etc/apt/sources.list.d/mesosphere.list
sudo apt-get -y update
~~~

# Installation

~~~bash
apt-get -y install mesos
~~~

# Edit slave

Update date zookeeper with proper IP

edit /etc/mesos/zk

~~~text
zk://${ZOOKEEPER}:2181/mesos
~~~
# Reference

https://mesosphere.com/downloads/
