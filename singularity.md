# Sigularity

## Prerequisite

~~~bash
yum install -y git maven
~~~

## Environment

Keyword | Value
---- | ----
MASTER01 | 192.168.0.1
MASTER02 | 192.168.0.2
MASTER03 | 192.168.0.3
MYIP | 192.168.0.1
 
## Get source

~~~bash
cd /root/
rm -rf Singularity
git clone https://github.com/HubSpot/Singularity.git
~~~

## To build Singularity

~~~bash
yum install -y npm
npm install -g brunch bower
~~~

## Build Sigularity

~~~bash
cd /root
cd Singularity
cd SingularityService
mvn clean install
~~~

TODO:
copy build package to /usr/local/bin


# To run

create sigularity configuration file

~~~bash
mkdir /etc/sigularity/
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
  master: zk://${MASTER01}:2181,${MASTER02}:2181,${MASTER03}:2181/mesos
  defaultCpus: 1
  defaultMemory: 128
  frameworkName: Singularity
  frameworkId: Singularity
  frameworkFailoverTimeout: 1000000

zookeeper:
  quorum: ${MASTER01}:2181,${MASTER02}:2181,${MASTER03}:2181
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
  baseUrl: http://${MYIP}:7099/singularity
~~~

## Create start script

edit /usr/lib/systemd/system/singularity.service

~~~
[Unit]
Description=Singularity Master
After=network.target
Wants=network.target

[Service]
ExecStart=/usr/bin/java -jar /usr/local/bin/SingularityService-*-shaded.jar server /etc/singularity/singularity.yaml
Restart=always
RestartSec=20
LimitNOFILE=16384

[Install]
WantedBy=multi-user.target
~~~
