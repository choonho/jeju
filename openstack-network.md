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
CONTROLLER  | override_real_controller(ex. 10.0.0.11)
NOVA_PASS   | nova_pass
NEUTRON_PASS | neutron_pass
RABBIT_PASS | rabbit_pass
MY_IP   | override_real_host_IP (ex. 10.0.0.21)
TUNNEL_IP   | override_real_tunnel_IP (ex. 10.0.1.21)
EXTERNAL_INTERFACE | override_external_network_interface_name (ex. eth2)



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

# Add a networking component

## Install and configure compute node

edit /etc/sysctl.conf

~~~text
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
~~~

Implement the changes:

~~~bash
sysctl -p
~~~

* To install the Networking components

~~~bash
apt-get -y install neutron-plugin-ml2 neutron-plugin-openvswitch-agent \
  neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent
~~~

* To configure the Networking common components

edit /etc/neutron/neutron.conf

~~~text
[DEFAULT]
state_path = /var/lib/neutron
lock_path = $state_path/lock

rpc_backend = rabbit

core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True

auth_strategy = keystone

notification_driver = neutron.openstack.common.notifier.rpc_notifier
[quotas]
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[keystone_authtoken]
auth_uri = http://${CONTROLLER}:5000
auth_url = http://${CONTROLLER}:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = ${NEUTRON_PASS}

[database]
[service_providers]
service_provider=LOADBALANCER:Haproxy:neutron.services.loadbalancer.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
service_provider=VPN:openswan:neutron.services.vpn.service_drivers.ipsec.IPsecVPNDriver:default

[oslo_messaging_rabbit]
rabbit_host = ${CONTROLLER}
rabbit_userid = openstack
rabbit_password = ${RABBIT_PASS}

~~~

* To configure the Modular Layer 2 (ML2) plug-in

edit /etc/neutron/plugins/ml2/ml2_conf.ini

~~~text
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = gre
mechanism_drivers = openvswitch

[ml2_type_flat]
flat_networks = external
[ml2_type_vlan]
[ml2_type_gre]
tunnel_id_ranges = 1:1000

[ml2_type_vxlan]
[securitygroup]
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

[ovs]
local_ip = ${TUNNEL_IP}
bridge_mappings = external:br-ex

[agent]
tunnel_types = gre

~~~

edit /etc/neutron/l3_agent.ini

~~~text
[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
external_network_bridge =
router_delete_namespaces = True
~~~

edit /etc/neutron/dhcp_agent.ini

~~~text
[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
dhcp_delete_namespaces = True
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
~~~

edit /etc/neutron/dnsmasq-neutron.conf

~~~text
dhcp-option-force=26,1454
~~~

Kill any existing dnsmasq processes:

~~~bash
pkill dnsmasq
~~~

edit /etc/neutron/metadata_agent.ini

~~~text
[DEFAULT]
auth_uri = http://${CONTROLLER}:5000
auth_url = http://${CONTROLLER}:35357
auth_region = RegionOne
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = ${NEUTRON_PASS}

nova_metadata_ip = ${CONTROLLER}
metadata_proxy_shared_secret = METADATA_SECRET
~~~

* To configure the Open vSwitch (OVS) service

Restart the OVS service:

~~~bash
service openvswitch-switch restart
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex ${EXTERNAL_INTERFACE}

service neutron-plugin-openvswitch-agent restart
service neutron-l3-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart

~~~

