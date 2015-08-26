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
CONTROLLER  | override_real_controller
ADMIN_PASS  | admin_pass
DEMO_PASS   | demo_pass
NOVA_PASS   | nova_pass
CINDER_PASS | cinder_pass
GLANCE_PASS | glance_pass
NEUTRON_PASS | neutron_pass
DASH_DBPASS | dash_dbpass
RABBIT_PASS | rabbit_pass
ADMIN_TOKEN | admin_token
MY_IP   | override_real_host_IP


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

# Add the Compute Service

## Install and configure a compute node

Install the packages

~~~bash
apt-get -y install nova-compute sysfsutils
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
iscsi_helper=tgtadm
libvirt_use_virtio_for_bridges=True
connection_type=libvirt
root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf
verbose=True
ec2_private_dns_show_ip=True
api_paste_config=/etc/nova/api-paste.ini
volumes_path=/var/lib/nova/volumes
enabled_apis=ec2,osapi_compute,metadata

rpc_backend = rabbit
auth_strategy = keystone

my_ip = ${MY_IP}

vnc_enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = ${MY_IP}
novncproxy_base_url = http://${CONTROLLER}:6080/vnc_auto.html

network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[oslo_messaging_rabbit]
rabbit_host = ${CONTROLLER}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}


[keystone_authtoken]
auth_uri = http://${CONTROLLER}:5000
auth_url = http://${CONTROLLER}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = nova
password = ${NOVA_PASS}

[glance]
host = ${CONTROLLER}

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[neutron]
url = http://${CONTROLLER}:9696
auth_strategy = keystone
admin_auth_url = http://${CONTROLLER}:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = ${NEUTRON_PASS}

~~~

By default, the Ubuntu packages create SQLite database.

~~~bash
rm f /var/lib/nova/nova.sqlite
~~~

# Add a networking component

## Install and configure compute node

edit /etc/sysctl.conf


Implement the changes:

~~~bash
sysctl -p
~~~

* To install the Networking components

~~~bash
apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent
~~~

* To configure the Networking common components

edit /etc/neutron/neutron.conf

~~~text
~~~

* To configure the Modular Layer 2 (ML2) plug-in

edit /etc/neutron/plugins/ml2/ml2_conf.ini

~~~text
~~~

* To configure the Open vSwitch (OVS) service

Restart the OVS service:

~~~bash
service openvswitch-switch restart
~~~

