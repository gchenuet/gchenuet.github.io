---
layout: post
title:  "OpenStack - Split Public API & Floating IP networks"
date:   2015-10-02 10:30:51
categories: OpenStack
tags: openstack, rdo-manager, ospd, heat, yaml, network, public-api, floating-ip
---
By default in RDO Manager/ RH-OSP, Public API and Floating IP are set in the same subnet.   
In some case, it will be useful to split these networks into different subnets and interfaces.   
<!--excerpt-->
One option is to dedicate a nic for the External network (Public API) and mount a bridge on an other nic for Floating IP:    

## 1. Apply change in YAML controller file:

{% highlight yaml %}
resources:
  OsNetConfigImpl:
    type: OS::Heat::StructuredConfig
    properties:
      group: os-apply-config
      config:
        os_net_config:
          network_config:
          [...]
          - type: ovs_bridge
            name: {get_input: bridge_name}
            mtu: 9000
            members:
             - type: interface
               name: nic5
               mtu: 9000
          - type: interface
            name: nic7
            mtu: 9000
            addresses:
             - ip_netmask: {get_param: ExternalIpSubnet}
            routes:
             - ip_netmask: 0.0.0.0/0
               next_hop: {get_param: ExternalInterfaceDefaultRoute}
            [...]
{% endhighlight %}

## 2. Check the result on controller nodes:

After deployment, you have two specific interfaces on controller nodes:    

{% highlight bash %}
[root@ctlr000 ~]# ip a
[...]
eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc pfifo-fast state UP qlen 1000
    link/ether 52:54:00:92:0f:bc brd ff:ff:ff:ff:ff:ff
    inet 10.225.192.77/25 brd 10.225.192.127 scope global eth6
       valid_lft forever preferred-lft forever
    inet 10.225.192.76/32 brd 10.225.192.127 scope global eth6
       valid_lft forever preferred-lft forever
    inet6 fe80::5054:ff:fe92:fbc/64 scope link
       valid_lft forever preferred-lft forever
10: br-ex: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc noqueue state UNKNOWN
    link/ether a2:23:80:61:b3:4d brd ff:ff:ff:ff:ff:ff
    inet6 fe80::a023:80ff:fe61:b34d/64 scope link
       valid_lft forever preferred_lft forever
[...]
{% endhighlight %}
_For this example: eth6 is dedicated for Public API & VIP and br-ex for Floating IPs_    

## 3. Create network with neutron

Once interfaces are up and running, the last step is to create network with neutron CLI and bind it on our bridge:    

{% highlight bash %}
neutron net-create fip-1 --router:external --provider:network_type flat 
--provider:physical_network datacentre
neutron subnet-create --name fip-1 --disable-dhcp --allocation-pool start=192.168.1.10,end=192.168.1.253 \
 --gateway 192.168.1.254 nova 192.168.1.0/24
{% endhighlight %}

