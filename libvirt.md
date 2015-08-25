# About Libvirt

# Environment

Keyword  | Value
-----    | -----
LIBVIRT_GIT | git://libvirt.org/libvirt.git
LIBVIRT_DIR | libvirt

# Installation

## Build from source

~~~bash
apt-get -y install git
git clone ${LIBVIRT_GIT}
apt-get -y build-dep libvirt-bin
apt-get -y install libtool autopoint xsltproc w3c-dtd-xhtml python-dev autoconf libnl-route-3-dev libxml-xpath-perl
cd ${LIBVIRT_DIR}
./autogen.sh
make -j 8
make install
~~~

## Update ld.conf

edit /etc/ld.so.conf.d/libvirt.conf

~~~text
/usr/lib/libvirt.so.0
~~~

# Start service

To start libvirt service, libvirtd is daemon process

~~~bash
/usr/local/sbin/libvirtd -d

echo "Testing..."
virsh version
~~~


