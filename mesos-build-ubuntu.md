# Mesos Build

# Environment

Keyword | Value
----    | -----
GIT | https://github.com/apache/mesos.git

# Build Mesos from source

## Download source file

~~~bash
cd /root/
rm -rf mesos
git clone ${GIT}
~~~

## System Requirements

~~~bash
apt-get update
apt-get install -y openjdk-7-jdk
apt-get install -y autoconf libtool
apt-get -y install build-essential python-dev python-boto libcurl4-nss-dev libsasl2-dev maven libapr1-dev libsvn-dev
~~~

# Compile Mesos


~~~bash
cd /root/mesos
./bootstrap
mkdir build
cd build
../configure
make
~~~

# Reference

http://mesos.apache.org/gettingstarted/

