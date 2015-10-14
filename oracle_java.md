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
apt-get install -y software-properties-common
sudo apt-add-repository ppa:webupd${VERSION}team/java
sudo apt-get update
sudo apt-get install -y oracle-java${VERSION}-installer
~~~


