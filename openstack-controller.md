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
CINDER_PASS | cinder_pass
DASH_DBPASS | dash_dbpass
RABBIT_PASS | rabbit_pass
NOVA_DBPASS | nova_dbpass
GLANCE_DBPASS | glance_dbpass
CINDER_DBPASS | cinder_dbpass
KEYSTONE_DBPASS | keystone_dbpass
ADMIN_TOKEN | admin_token


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

edit /etc/mysql/conf.d/mysql_openstack.cnf

~~~text
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
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
rabbitmqctl add_user openstack ${RABBIT_PASS}
~~~
* Permit configuration, write, and read access for the openstack user:
~~~bash
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
~~~

# Add the Identity service

## Create keystone database and update privileges.

~~~bash
mysql -u root -p <<EOF
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
IDENTIFIED BY '${NOVA_DBPASS}';
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
IDENTIFIED BY '${GLANCE_DBPASS}';
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '${KEYSTONE_DBPASS}';
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
IDENTIFIED BY '${NEUTRON_DBPASS}';
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
IDENTIFIED BY '${CINDER_DBPASS}';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
IDENTIFIED BY '${NOVA_DBPASS}';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
IDENTIFIED BY '${GLANCE_DBPASS}';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY '${KEYSTONE_DBPASS}';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY '${NEUTRON_DBPASS}';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
IDENTIFIED BY '${CINDER_DBPASS}';
FLUSH PRIVILEGES;
EOF
~~~

Install keystone packages
~~~bash
apt-get install keystone python-openstackclient apache2 libapache2-mod-wsgi memcached python-memcache
~~~

edit /etc/keystone/keystone.conf
~~~text
[DEFAULT]

admin_token = ${ADMIN_TOKEN}
log_dir = /var/log/keystone

[assignment]
[auth]
[cache]
[catalog]
[credential]

[database]
connection = mysql://keystone:${KEYSTONE_DBPASS}@${HOSTNAME}/keystone

[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[eventlet_server_ssl]
[federation]
[fernet_tokens]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[matchmaker_ring]

[memcache]
servers = localhost:11211

[oauth1]
[os_inherit]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[resource]

[revoke]
driver = keystone.contrib.revoke.backends.sql.Revoke

[role]
[saml]
[signing]
[ssl]

[token]
provider = keystone.token.providers.uuid.Provider
driver = keystone.token.persistence.backends.memcache.Token

[trust]

[extra_headers]
Distribution = Ubuntu
~~~

* Populate the Identity service database:
~~~bash
su -s /bin/sh -c "keystone-manage db_sync" keystone
~~~

* To configure the Apache HTTP Server

~~~bash
echo "ServerName ${HOSTNAME}" >> /etc/apache2/apache2.conf
~~~

edit /etc/apache2/sites-available/wsgi-keystone.conf

~~~text
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /var/www/cgi-bin/keystone/main
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    LogLevel info
    ErrorLog /var/log/apache2/keystone-error.log
    CustomLog /var/log/apache2/keystone-access.log combined
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /var/www/cgi-bin/keystone/admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    LogLevel info
    ErrorLog /var/log/apache2/keystone-error.log
    CustomLog /var/log/apache2/keystone-access.log combined
</VirtualHost>
~~~

* Enable the Identity service virtual hosts:
~~~bash
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
~~~

* Create the directory structure for the WSGI components:
~~~bash
mkdir -p /var/www/cgi-bin/keystone
~~~

* Copy the WSGI components from the upstream repository into this directory:
~~~bash
apt-get install -y curl
curl http://git.openstack.org/cgit/openstack/keystone/plain/httpd/keystone.py?h=stable/kilo \
  | tee /var/www/cgi-bin/keystone/main /var/www/cgi-bin/keystone/admin
~~~

* Adjust ownership and permissions on this directory and the files in it:
~~~bash
chown -R keystone:keystone /var/www/cgi-bin/keystone
chmod 755 /var/www/cgi-bin/keystone/*
~~~

* To finalize installtion
~~~bash
service apache2 restart
rm -f /var/lib/keystone/keystone.db
service keystone restart
~~~

## Create the service entity and API endpoint

* To configure prerequisites
* To create the service entity and API endpoint

~~~bash
echo "Create the service entity for the Identity service"
openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--name keystone --description "OpenStack Identity" identity

echo "Create the Identity service API endpoint"
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--publicurl http://controller:5000/v2.0 \
--internalurl http://controller:5000/v2.0 \
--adminurl http://controller:35357/v2.0 \
--region RegionOne \
identity
~~~

## Create projects, users, and roles

The identity service provides authentication services for each OpenStack service.
The authentication service uses a combination of domains, projects (tenants), users, and roles.

* To create tenants, users, and roles

~~~bash
echo "Create the admin project"
openstack project create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--description "Admin Project" admin

echo "Create the admin user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--password-prompt admin

echo "Create the admin role"
openstack role create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
admin

echo "Add the admin role to the admin project and user"
openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--project admin --user admin admin
~~~

* This guide uses a service project that contains a unique user for each service that you add to your environment

~~~bash
openstack project create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--description "Service Project" service
~~~

* Regular (non-admin)tasks should use an unprivileged project and user.

~~~bash
openstack project create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--description "Demo Project" demo

openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--password-prompt demo

openstack role create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
user

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--project demo --user demo user
~~~


# Add the Image service
# Add the Compute service
# Add a networking component
# Add the dashboard
# Add the Block Storage service


