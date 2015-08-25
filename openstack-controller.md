# Architecture
## Overview
The OpenStack project is an open source cloud computing platform that supports all types of cloud environments. The project aims for simple implementation, massive scalability, and a rich set of features.

## Conceptual architecture
![Conceptual architecture](http://docs.openstack.org/kilo/install-guide/install/apt/content/figures/1/a/common/figures/openstack_kilo_conceptual_arch.png)

## Example architectures

![Example architecture](http://docs.openstack.org/kilo/install-guide/install/apt/content/figures/1/a/common/figures/installguidearch-neutron-hw.png)

Network layout is

![Network layout](http://docs.openstack.org/kilo/install-guide/install/apt/content/figures/1/a/common/figures/installguidearch-neutron-networks.png)

# Basic environment
This example installs three-node architecture with OpenStack Networking(neutron).

Keyword     | Value
-----       | -----
ADMIN_PASS  | admin_pass
CINDER_DBPASS | cinder_dbpass
CINDER_PASS | cinder_pass
DASH_DBPASS | dash_dbpass
RABBIT_PASS | rabbit_pass


## Before you begin
## Security
## Networking
## Network Time Protocol(NTP)
You must install NTP to properly synchronize services among nodes. We recommend that you configure the controller node to reference more accurate servers and other nodes to reference the controller node.

To install the NTP service
~~~bash
apt-get -y install ntp
~~~

## OpenStack packages
* Install the Ubuntu cloud archive keyring and repository
~~~bash
apt-get install ubuntu-cloud-keyring
echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/kilo main" > /etc/apt/sources.list.d/cloudarchive-kilo.list
~~~

* Update the package lists
~~~bash
apt-get update
~~~

## SQL database
* To install and configure the database server
~~~bash
apt-get install mariadb-server python-mysqldb
~~~

* To finalize installation
~~~bash
service mysql restart
~~~


## Message queue
* To install the message queue service
~~~bash
apt-get install rabbitmq-server
~~~

* To configure the message queue service
* Add the openstack user
~~~bash
rabbitmqctl add_uer openstack ${RABBIT_PASS}
~~~
* Permit configuration, write, and read access for the openstack user:
~~~bash
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
~~~

# Add the Identity service
# Add the Image service
# Add the Compute service
# Add a networking component
# Add the dashboard
# Add the Block Storage service


