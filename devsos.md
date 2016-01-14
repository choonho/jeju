# Mesos Setup

# Environment

Keyword | Value
----    | -----
MASTER01 | 192.168.1.1
MYID    | 1
DEV     | eth0
SINGULARITY_VER | 0.4.7

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
~~~

## Restart Zookeeper

~~~bash
systemctl restart zookeeper.service
~~~

# Edit mesos

Update date zookeeper with proper IP

edit /etc/mesos/zk

~~~text
zk://${MASTER01}:2181/mesos
~~~

## Update Quorum

edit /etc/mesos-master/quorum

~~~text
1
~~~

edit /etc/mesos-master/ip

~~~bash
hostname -i > /etc/mesos-master/ip
~~~

## Restart mesos master

~~~bash
systemctl start mesos-master.service
systemctl start mesos-slave.service
~~~

# Reference

https://mesosphere.com/downloads/


# Sigularity

## Prerequisite

~~~bash
yum install -y git maven
~~~

## Environment

Keyword | Value
---- | ----

## Get source

~~~bash
wget -O /usr/local/bin/SingularityService-${SINGULARITY_VER}-shaded.jar https://repo1.maven.org/maven2/com/hubspot/SingularityService/${SINGULARITY_VER}/SingularityService-${SINGULARITY_VER}-shaded.jar
~~~

# To run

create sigularity configuration file

~~~bash
mkdir /etc/singularity/
~~~

edit /etc/singularity/singularity.yaml

~~~text
# Run SingularityService on port 7099 and log to /var/log/singularity-access.log
server:
  type: simple
  applicationContextPath: /singularity
  connector:
    type: http
    port: 7099
  requestLog:
    appenders:
      - type: file
        currentLogFilename: /var/log/singularity-access.log
        archivedLogFilenamePattern: /var/log/singularity-access-%d.log.gz

mesos:
  master: zk://${MASTER01}:2181/mesos
  defaultCpus: 1
  defaultMemory: 128
  frameworkName: Singularity
  frameworkId: Singularity
  frameworkFailoverTimeout: 1000000

zookeeper:
  quorum: ${MASTER01}:2181
  zkNamespace: singularity
  sessionTimeoutMillis: 60000
  connectTimeoutMillis: 5000
  retryBaseSleepTimeMilliseconds: 1000
  retryMaxTries: 3

logging:
  loggers:
    "com.hubspot.singularity" : TRACE

enableCorsFilter: true
sandboxDefaultsToTaskId: false  # enable if using SingularityExecutor

ui:
  title: Singularity (lsky)
  baseUrl: http://${MASTER01}:7099/singularity
~~~

## Create start script

edit /usr/lib/systemd/system/singularity.service

~~~
[Unit]
Description=Singularity Master
After=network.target
Wants=network.target

[Service]
ExecStart=/usr/bin/java -jar /usr/local/bin/SingularityService-${SINGULARITY_VER}-shaded.jar server /etc/singularity/singularity.yaml
Restart=always
RestartSec=20
LimitNOFILE=16384

[Install]
WantedBy=multi-user.target
~~~

## Restart Singularity Service

~~~bash
systemctl start singularity.service
~~~
