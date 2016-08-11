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

edit /var/lib/zookeeper/myid

~~~text
${MYID}
~~~

## Update zookeeper zoo.cfg

edit /etc/zookeeper/conf/zoo.cfg

~~~text
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

maxClientCnxns=50
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/var/lib/zookeeper
# the port at which the clients will connect
clientPort=2181
server.1=${MASTER01}:2888:3888
server.2=${MASTER02}:2888:3888
server.3=${MASTER03}:2888:3888
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
