# Admin Guide
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
EXTERNAL_NETWORK_CIDR | override_external_network_cidr (ex. 203.0.113.0/24)
FLOATING_IP_START | override_floating_ip_start (ex. 203.0.113.101)
FLOATING_IP_END | override_floating_ip_start (ex. 203.0.113.200)
EXTERNAL_NETWORK_GATEWAY | override_gateway_ip (ex. 203.0.113.1)
TENANT_NETWORK_CIDR | override_tenant_network_cir (ex. 192.168.1.0/24)
TENANT_NETWORK_GATEWAY | override_tenant_network_gateway (ex. 192.168.1.1)


# Add a networking component

## Create external network

Before running, make sure you already update environments like 'source admin-openrc.sh'

### To create the external network

Create the network:

~~~bash
neutron net-create ext-net --router:external \
  --provider:physical_network external --provider:network_type flat
~~~


### To create a subnet on the external network

Create the subnet

~~~bash
neutron subnet-create ext-net ${EXTERNAL_NETWORK_CIDR} --name ext-subnet \
  --allocation-pool start=${FLOATING_IP_START},end=${FLOATING_IP_END} \
  --disable-dhcp --gateway ${EXTERNAL_NETWORK_GATEWAY}
~~~

## Create Tenant network

### Create the network

~~~bash
neutron net-create demo-net
~~~

### Create subnet on the tenant network

~~~bash
neutron subnet-create demo-net ${TENANT_NETWORK_CIDR} \
  --name demo-subnet --gateway ${TENANT_NETWORK_GATEWAY}
~~~

### Create router

To create a router on the tenant network and attach the external and tenant networks to it

Create the router:

~~~bash
neutron router-create demo-router
~~~

Attach the router to the demo tenant subnet:

~~~bash
neutron router-interface-add demo-router demo-subnet
~~~

Attach the router to the external network by settting it as the gateway:

~~~bash
neutron router-gateway-set demo-router ext-net
~~~
