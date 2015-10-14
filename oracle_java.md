# About This document

This document is show how to install Oracle Java

# Environment

Keyword  | Value
-----    | -----
VERSION  | 8

# Installation

Current default version is Java8
If you want to install another version, change environment

~~~bash
apt-get install -y software-properties-common debconf-utils
sudo apt-add-repository ppa:webupd${VERSION}team/java
sudo apt-get update
echo "oracle-java${VERSION}-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections
sudo apt-get install -y oracle-java${VERSION}-installer
~~~


