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
apt-get -y install ubuntu-cloud-keyring
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
apt-get -y install mariadb-server python-mysqldb
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
apt-get -y install rabbitmq-server
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
apt-get -y install keystone python-openstackclient apache2 libapache2-mod-wsgi memcached python-memcache
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
--publicurl http://${HOSTNAME}:5000/v2.0 \
--internalurl http://${HOSTNAME}:5000/v2.0 \
--adminurl http://${HOSTNAME}:35357/v2.0 \
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

## Install and configure

* Create glance user

~~~bash
echo "Create glance user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--password-prompt glance

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--project service --user glance admin

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--name glance --description "OpenStack Image Service" image
~~~

* Create the Image service API endpoint:

~~~bash
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--publicurl http://${HOSTNAME}:9292 \
--internalurl http://${HOSTNAME}:9292 \
--adminurl http://${HOSTNAME}:9292 \
--region RegionOne \
image
~~~

* To install and configure the Image service components

~~~bash
apt-get -y install glance python-glanceclient
~~~

edit /etc/glance/glance-api.conf

~~~text
[DEFAULT]
bind_host = 0.0.0.0
bind_port = 9292

log_file = /var/log/glance/api.log
backlog = 4096

registry_host = 0.0.0.0
registry_port = 9191

registry_client_protocol = http
notification_driver = noop

rabbit_host = ${HOSTNAME}
rabbit_port = 5672
rabbit_use_ssl = false
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}
rabbit_virtual_host = /
rabbit_notification_exchange = glance
rabbit_notification_topic = notifications
rabbit_durable_queues = False

qpid_notification_exchange = glance
qpid_notification_topic = notifications
qpid_hostname = localhost
qpid_port = 5672
qpid_username =
qpid_password =
qpid_sasl_mechanisms =
qpid_reconnect_timeout = 0
qpid_reconnect_limit = 0
qpid_reconnect_interval_min = 0
qpid_reconnect_interval_max = 0
qpid_reconnect_interval = 0
qpid_heartbeat = 5
# Set to 'ssl' to enable SSL
qpid_protocol = tcp
qpid_tcp_nodelay = True

delayed_delete = False
scrub_time = 43200
scrubber_datadir = /var/lib/glance/scrubber

image_cache_dir = /var/lib/glance/image-cache/

[oslo_policy]
[database]
backend = sqlalchemy
connection = mysql://glance:${GLANCE_DBPASS}@${HOSTNAME}/glance

[oslo_concurrency]

[keystone_authtoken]
auth_uri = http://${HOSTNAME}:5000
auth_url = http://${HOSTNAME}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = ${GLANCE_PASS}

[paste_deploy]
flavor=keystone

[store_type_location_strategy]
[profiler]
[task]
[taskflow_executor]
[glance_store]
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
~~~


edit /etc/glance/glance-registry.conf

~~~text
[DEFAULT]
bind_host = 0.0.0.0
bind_port = 9191
log_file = /var/log/glance/registry.log
backlog = 4096
api_limit_max = 1000
limit_param_default = 25
notification_driver = noop
rabbit_host = ${HOSTNAME}
rabbit_port = 5672
rabbit_use_ssl = false
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}
rabbit_virtual_host = /
rabbit_notification_exchange = glance
rabbit_notification_topic = notifications
rabbit_durable_queues = False

qpid_notification_exchange = glance
qpid_notification_topic = notifications
qpid_hostname = localhost
qpid_port = 5672
qpid_username =
qpid_password =
qpid_sasl_mechanisms =
qpid_reconnect_timeout = 0
qpid_reconnect_limit = 0
qpid_reconnect_interval_min = 0
qpid_reconnect_interval_max = 0
qpid_reconnect_interval = 0
qpid_heartbeat = 5
qpid_protocol = tcp
qpid_tcp_nodelay = True

[oslo_policy]
[database]
backend = sqlalchemy
connection = mysql://glance:${GLANCE_DBPASS}@${HOSTNAME}/glance
[keystone_authtoken]
auth_uri = http://${HOSTNAME}:5000
auth_url = http://${HOSTNAME}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = ${GLANCE_PASS}

[paste_deploy]
flavor = keystone

[profiler]
~~~

* Populate the Image service database:

~~~bash
su -s /bin/sh -c "glance-manage db_sync" glance
~~~

* To finalize installation

~~~bash
service glance-registry restart
service glance-api restart
~~~

~~~bash
rm -f /var/lib/glance/glance.sqlite
~~~


# Add the Compute service

## Install and configure controller node

* Create nova user

~~~bash
echo "Create nova user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--password-prompt nova

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--project service --user nova admin

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--name nova --description "OpenStack Compute Service" nova
~~~

* Create the Compute service API endpoint:

~~~bash
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--publicurl http://${HOSTNAME}:8774/v2/%\(tenant_id\)s \
--internalurl http://${HOSTNAME}:8774/v2/%\(tenant_id\)s \
--adminurl http://${HOSTNAME}:8774/v2/%\(tenant_id\)s \
--region RegionOne \
compute
~~~

* To install and configure Compute controller components

~~~bash
apt-get -y install nova-api nova-cert nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler python-novaclient
~~~

edit /etc/nova/nova.conf

~~~text
[DEFAULT]

dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
logdir=/var/log/nova
state_path=/var/lib/nova
lock_path=/var/lock/nova
force_dhcp_release=True
libvirt_use_virtio_for_bridges=True
verbose=True
ec2_private_dns_show_ip=True
api_paste_config=/etc/nova/api-paste.ini
enabled_apis=ec2,osapi_compute,metadata
rpc_backend = rabbit
auth_strategy = keystone
my_ip = ${MY_IP}
vncserver_listen = ${MY_IP}
vncserver_proxyclient_address = ${MY_IP}

network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[database]
connection = mysql://nova:${NOVA_DBPASS}@${HOSTNAME}/nova

[oslo_messaging_rabbit]
rabbit_host = ${HOSTNAME}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}

[keystone_authtoken]
auth_uri = http://${HOSTNAME}:5000
auth_url = http://${HOSTNAME}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = nova
password = ${NOVA_PASS}

[glance]
host = ${HOSTNAME}

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[neutron]
url = http://${HOSTNAME}:9696
auth_strategy = keystone
admin_auth_url = http://${HOSTNAME}:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = ${NEUTRON_PASS}

~~~

* To finalize installation

~~~bash
service nova-api restart
service nova-cert restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
~~~

By default, the Ubuntu packages create an SQLite database.

~~~bash
rm -f /var/lib/nova/nova.sqlite
~~~

# Add a networking component

## OpenStack Networking (neutron)

### Install and configure controller node

* Create neutron user

~~~bash
echo "Create neutron user"
openstack user create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--password-prompt neutron

openstack role add \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--project service --user neutron admin

openstack service create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--name neutron --description "OpenStack Networking Service" network
~~~

* Create the Networking service API endpoint:

~~~bash
openstack endpoint create \
--os-token ${ADMIN_TOKEN} --os-url http://${HOSTNAME}:35357/v2.0 \
--publicurl http://${HOSTNAME}:9696 \
--adminurl http://${HOSTNAME}:9696 \
--internalurl http://${HOSTNAME}:9696 \
--region RegionOne \
network
~~~

* To  install the Networking components

~~~bash
apt-get -y install neutron-server neutron-plugin-ml2 python-neutronclient
~~~

* To configure the Networking server component

edit /etc/neutron/neutron.conf

~~~text
[DEFAULT]
core_plugin = ml2
service_plugins = router
auth_strategy = keystone
allow_overlapping_ips = True
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
nova_url = http://${HOSTNAME}:8774/v2
rpc_backend=rabbit

[matchmaker_redis]
[matchmaker_ring]
[quotas]
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[keystone_authtoken]
auth_uri = http://${HOSTNAME}:5000
auth_url = http://${HOSTNAME}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = ${NEUTRON_PASS}

[database]
connection = mysql://neutron:${NEUTRON_DBPASS}@${HOSTNAME}/neutron

[nova]
auth_url = http://${HOSTNAME}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
region_name = RegionOne
project_name = service
username = nova
password = ${NOVA_PASS}

[oslo_concurrency]
lock_path = $state_path/lock

[oslo_policy]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
rabbit_host = ${HOSTNAME}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}
~~~

edit /etc/neutron/plugins/ml2/ml2_conf.ini

~~~text
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = gre
mechanism_drivers = openvswitch

[ml2_type_flat]

[ml2_type_vlan]

[ml2_type_gre]
tunnel_id_ranges = 1:1000


[ml2_type_vxlan]
[securitygroup]
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
~~~

* To finalize the installation

~~~bash
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
~~~

Restart the Compute service

~~~bash
service nova-api restart
service neutron-server restart
~~~

# Add the dashboard
# Add the Block Storage service


