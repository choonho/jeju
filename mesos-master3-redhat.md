# Mesos Setup

# Environment

Keyword | Value
----    | -----
MASTER01 | 192.168.0.1
MASTER02 | 192.168.0.2
MASTER03 | 192.168.0.3
MYID	| 1
DEV	| eth0

# Setup Mesos

~~~bash
rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
~~~

# Installation

~~~bash
yum -y install net-tools
yum -y install mesos
yum -y install mesosphere-zookeeper
~~~

# Edit Zookeeper

## Update myid

myid must be unique on cluster

edit /etc/zookeeper/conf/myid

~~~text
${MYID}
~~~

## Update zookeeper zoo.cfg

~~~bash
echo "server.1=${MASTER01}:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
echo "server.2=${MASTER02}:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
echo "server.3=${MASTER03}:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
~~~

## Restart Zookeeper

~~~bash
systemctl restart zookeeper.service
~~~

# Edit mesos

Update date zookeeper with proper IP

edit /etc/mesos/zk

~~~text
zk://${MASTER01}:2181,${MASTER02}:2181,${MASTER03}:2181/mesos
~~~

## Update Quorum

edit /etc/mesos-master/quorum

~~~text
2
~~~

## Stop mesos slave

~~~bash
service mesos-slave stop
echo "manual" > /etc/init/mesos-slave.override
update-rc.d -f mesos-slave remove
~~~

edit /etc/mesos-master/ip

~~~bash
hostname -i > /etc/mesos-master/ip
~~~

## Restart mesos master

~~~bash
service mesos-master restart
~~~

# Reference

https://mesosphere.com/downloads/
