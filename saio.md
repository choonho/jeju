# SAIO (Swift All In One)

**prerequsite**

Keyword | Value 
---- | ----
USER | root

## Installing dependencies
~~~bash
apt-get update
apt-get -y install curl gcc memcached rsync sqlite3 xfsprogs \
                     git-core libffi-dev python-setuptools
~~~

~~~bash
apt-get -y install python-coverage python-dev python-nose \
                     python-simplejson python-xattr python-eventlet \
                     python-greenlet python-pastedeploy \
                     python-netifaces python-pip python-dnspython \
                     python-mock
~~~

## Using a partition for storage

Set up a single partition:

~~~bash
mkfs.xfs /dev/sdb
~~~

append /etc/fstab
~~~text
/dev/sdb /mnt/sdb1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0
~~~

~~~bash
mkdir /mnt/sdb
mount /mnt/sdb
mkdir /mnt/sdb/1 /mnt/sdb/2 /mnt/sdb/3 /mnt/sdb/4
chown ${USER}:${USER} /mnt/sdb/*
mkdir /srv
for x in {1..4}; do sudo ln -s /mnt/sdb/$x /srv/$x; done
sudo mkdir -p /srv/1/node/sdb1 /srv/1/node/sdb5 \
              /srv/2/node/sdb2 /srv/2/node/sdb6 \
              /srv/3/node/sdb3 /srv/3/node/sdb7 \
              /srv/4/node/sdb4 /srv/4/node/sdb8 \
              /var/run/swift
chown -R ${USER}:${USER} /var/run/swift
# **Make sure to include the trailing slash after /srv/$x/**
for x in {1..4}; do sudo chown -R ${USER}:${USER} /srv/$x/; done
~~~

## Getting the code

Check out the python-swiftclient repo

~~~bash
cd $HOME; git clone https://github.com/openstack/python-swiftclient.git
~~~

Build a development installation of python-swiftclient

~~~bash
cd $HOME/python-swiftclient; sudo python setup.py develop; cd -
~~~

Check out the swift repo

~~~bash
cd $HOME
git clone https://github.com/openstack/swift.git
~~~

Build a development installation of swift

~~~bash
cd $HOME/swift; sudo pip install -r requirements.txt; sudo python setup.py develop; cd -
~~~

Install swift's test dependencies

~~~bash
cd $HOME/swift; sudo pip install -r test-requirements.txt
~~~

## Setting up rsync

Create /etc/rsyncd.conf

~~~bash
sudo cp $HOME/swift/doc/saio/rsyncd.conf /etc/
sudo sed -i "s/<your-user-name>/${USER}/" /etc/rsyncd.conf
~~~





# Reference
http://docs.openstack.org/developer/swift/development_saio.html

