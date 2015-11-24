# Mesos Setup

# Environment

Keyword | Value
----    | -----
MASTER01 | 192.168.0.1
MASTER02 | 192.168.0.2
MASTER03 | 192.168.0.3
DEV	| eth0


# Setup Mesos

~~~bash
rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
~~~

# Installation

~~~bash
yum -y install net-tools
yum -y install docker
yum -y install mesos
~~~



# Edit mesos

Update date zookeeper with proper IP

edit /etc/mesos/zk

~~~text
zk://${MASTER01}:2181,${MASTER02}:2181,${MASTER03}:2181/mesos
~~~


## Stop mesos master

~~~bash
systemctl stop mesos-master.service
systemctl disable mesos-master.service
~~~

## Restart mesos slave

update /etc/mesos-slave/ip

~~~bash
hostname -i > /etc/mesos-slave/ip
~~~

## Update containerizers

edit /etc/mesos-slave/containerizers

~~~text
docker,mesos
~~~

We support docker

## Update registration timeout

timeout update

create /etc/mesos-slave/executor_registration_timeout

~~~bash
echo "5mins" > /etc/mesos-slave/executor_registration_timeout
~~~

# Start docker, mesos-slave

~~~bash
systemctl enable docker.service
systemctl start docker.service

systemctl enable mesos-slave.service
systemctl start mesos-slave.service
~~~

# Reference

https://mesosphere.com/downloads/
