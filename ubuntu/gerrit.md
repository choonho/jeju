# Gerrit Code Review - Quick get started guide

Keyword | Value
---- | ----
VER  | 2.11.5
REPO | /opt/gerrit

## Requirements

Do you already have Java installed?

~~~bash
apt-get update
apt-get install -y wget
java -version
~~~

## Create a user to host the Gerrit service

We will run the service as a non-privileged user on system

~~~bash
useradd -d ${REPO} -m -s /bin/bash gerrit
~~~

# Download Gerrit

It's time to download the archive that contains the Gerrit web and ssh service.

~~~bash
su gerrit
wget -O ${REPO}/gerrit-${VER}.war https://www.gerritcodereview.com/download/gerrit-${VER}.war
cd ~/
java -jar gerrit-${VER}.war init --batch -d ~/gerrit_site
~~~
