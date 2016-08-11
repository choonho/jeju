# SAIO (Swift All In One)

**prerequsite**

Keyword | Value 
---- | ----
USER | root
DISK | sdb
BRANCH | origin/stable/juno

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
mkfs.xfs /dev/${DISK}
~~~

append /etc/fstab

~~~bash
echo "/dev/${DISK} /mnt/${DISK} xfs noatime,nodiratime,nobarrier,logbufs=8 0 0" >> /etc/fstab
~~~

~~~bash
mkdir /mnt/${DISK}
mount /mnt/${DISK}
mkdir /mnt/${DISK}/1 /mnt/${DISK}/2 /mnt/${DISK}/3 /mnt/${DISK}/4
chown ${USER}:${USER} /mnt/${DISK}/*
mkdir /srv
for x in {1..4}; do sudo ln -s /mnt/${DISK}/$x /srv/$x; done
sudo mkdir -p /srv/1/node/${DISK}1 /srv/1/node/${DISK}5 \
              /srv/2/node/${DISK}2 /srv/2/node/${DISK}6 \
              /srv/3/node/${DISK}3 /srv/3/node/${DISK}7 \
              /srv/4/node/${DISK}4 /srv/4/node/${DISK}8 \
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
cd swift
git checkout ${BRANCH}
cd -
~~~

Build a development installation of swift

~~~bash
cd $HOME/swift; sudo pip install -r requirements.txt; sudo python setup.py develop; cd -
~~~

Install swifts test dependencies

~~~bash
cd $HOME/swift; sudo pip install -r test-requirements.txt
~~~

## Setting up rsync

Create /etc/rsyncd.conf

~~~bash
sudo cp $HOME/swift/doc/saio/rsyncd.conf /etc/
sudo sed -i "s/<your-user-name>/${USER}/" /etc/rsyncd.conf
~~~

edit /etc/default/rsync

~~~text
RSYNC_ENABLE=true
RSYNC_OPTS=''
~~~

On platforms with SELinux in Enforcing mode, ether set to Permissive

~~~bash
setenforce Permissive
~~~

Start the rsync daemon

~~~bash
service rsync restart
~~~

Verify rsync is accepting connections for all servers

~~~bash
rsync rsync://pub@localhost/
~~~


## Configuring each node

Populate the /etc/swift directory itself:

~~~bash
rm -rf /etc/swift
cd $HOME/swift/doc; sudo cp -r saio/swift /etc/swift; cd -
chown -R ${USER}:${USER} /etc/swift
~~~

Update <your-user-name> references in the Swift config files:

~~~bash
find /etc/swift/ -name \*.conf | xargs sudo sed -i "s/<your-user-name>/${USER}/"
~~~

## Setting up scripts for running Swfit

Copy the SAIO scripts for resetting the environment

~~~bash
mkdir -p $HOME/bin
cd $HOME/swift/doc; cp saio/bin/* $HOME/bin; cd -
chmod +x $HOME/bin/*
~~~

Install the sample configuration file for running tests:

~~~bash
cp $HOME/swift/test/sample.conf /etc/swift/test.conf
~~~

Add an environment variable for running tests below:

~~~bash
echo "export SWIFT_TEST_CONFIG_FILE=/etc/swift/test.conf" >> $HOME/.bashrc
~~~

Be sure that your PATH includes the bin directory:
then construct the initial rings using the provided script:

~~~bash
echo "export PATH=${PATH}:$HOME/bin" >> $HOME/.bashrc
. $HOME/.bashrc
sudo sed -i "s/sdb/${DISK}/" $HOME/bin/remakerings
remakerings
~~~



# Reference
http://docs.openstack.org/developer/swift/development_saio.html

