---
layout: post
title:  "RDO / OSP-d: Play with LDAP"
date:   2015-11-05 18:00:51
categories: rdo rdo-manager ospd heat yaml ldap osd hiera
tags: [rdo, rdo-manager, ospd, heat, yaml, ldap, osd, hiera]
---
# WHAT ?	

LDAP is an open standard application protocol for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network.	
For example: a common usage is to provide a single sign on where one password for a user is shared between many services.	

In OpenStack, and more especially in Keystone (Identity service), this feature is fully supported.	
But if you have already played with OSP-d, you know LDAP templates aren't included for the moment.	

I propose to you a quick introduction about how to enable LDAP in an OSP-d deployment via puppet params.	

# HOW ?		

A simple way to apply LDAP configuration to OpenStack can be to modifying puppet configuration data.	
Following the OSP documentation‌, we need to add LDAP parameters into our environment file.	

## Add LDAP Parameters	

Example: /home/stack/my_overcloud/infra-environment.yaml	
{% highlight yaml %}	
resources_registry:
[...]
parameters:
  controllerExtraConfig:
    keystone::roles::admin:email : 'ospadmin@ospoc.lan'
    keystone::roles::admin:password: 'R3dH4t'
    keystone::ldap::identity_driver : 'keystone.identity.backends.ldap.Identity'
    keystone::ldap::url : 'ldap://192.168.42.10:389'
    keystone::ldap::user : 'uid=admin,cn=users,cn=accounts,dc=ospoc,dc=lan'
    keystone::ldap::password : 'R3dH4t'
    keystone::ldap::suffix : 'dc=ospoc,dc=lan'
    keystone::ldap::user_tree_dn : 'cn=users,cn=accounts,dc=ospoc,dc=lan'
    keystone::ldap::user_allow_create : 'False'
    keystone::ldap::user_allow_update : 'False'
    keystone::ldap::user_allow_delete : 'False'
    keystone::ldap::user_mail_attribute: 'mail'
    keystone::ldap::query_scope: 'sub'
    keystone::ldap::user_id_attribute: 'uid'
    keystone::ldap::user_name_attribute: 'uid'
    keystone::ldap::group_tree_dn : 'cn=groups,cn=accounts,dc=ospoc,dc=lan'
    keystone::ldap::group_objectclass : 'groupOfNames'
    keystone::ldap::group_allow_create : 'False'
    keystone::ldap::group_allow_update : 'False'
    keystone::ldap::group_allow_delete : 'False'
    keystone::ldap::user_filter : '(memberof=cn=openstack_enabled,cn=groups,cn=accounts,dc=ospoc,dc=lan)'
    keystone::ldap::assignment_driver : 'keystone.assignment.backends.sql.Assignment'
{% endhighlight %}	

You can find more informations about LDAP puppet class here.	

## Run it !	

Please check that you have the environment file (infra_environment.yaml in our case) in your stack deployment line.	

Example:	
{% highlight yaml %}	
openstack overcloud deploy --templates -e /home/stack/my_overcloud/infra_environment.yaml	
{% endhighlight %}	

Deploy your Overcloud as usual, the configuration will be apply during the deployment.		

Hope this is useful for you 	
Feel free to comment or review it.	

