---
layout: post
title:  "OpenStack - Deal with Partitions instead of Devices for Ceph"
date:   2015-10-05 18:30:51
categories: rdo rdo-manager ospd heat yaml ceph osd hiera
tags: [rdo, rdo-manager, ospd, heat, yaml, ceph, osd, hiera]
---
For a RH-OSP PoC platform, I had to deploy OSD and journal on the same hard drive.          
            
It's not recommended to have both on the same disk because you're going to have bad IO performance.         
            
But if you need to do this, follow these quick steps:           
            
# 1. Modify hiera file		
            
The first step is to modify the Ceph hieradata file.        
	    
In our case, we will using two disks with two partitions on each disk:          
        
* OSD: /dev/sdX1
* Journal: /dev/sdX2
        
{% highlight yaml %}
[stack@undercloud ~]$ vim ~/my-cloud/templates/puppet/hieradata/ceph.yaml
[...]
ceph::profile::params::osds:
  '/dev/sdb1':
      journal: '/dev/sdb2'
  '/dev/sdc1':
      journal: '/dev/sdc2'
[...]
{% endhighlight %}

# 2. Create a firstboot template	
        
The second step consist to create a script for erasing and creating partitions for the Ceph storage.	
	        
Any configuration possible via cloud-init may be performed at this step, either by applying cloud-config yaml or running arbitrary additional scripts.		
            
{% highlight yaml %}
[stack@undercloud ~]$ vim ~/my-cloud/templates/firstboot/ceph_wipe.yaml
heat_template_version: 2014-10-16

resources:
  userdata:
    type: OS::Heat::MultipartMime
    properties:
      parts:
      - config: {get_resource: wipe_ceph_disk}

  wipe_ceph_disk:
    type: OS::Heat::SoftwareConfig
    properties:
      config: |
        #!/bin/bash
        /bin/hostnamectl | grep 'hostname' | grep ceph -q ;
        if [ $? == 0 ]; then
        for i in {b,c,d,e,f,g,h,i}; do sgdisk -Z /dev/sd${i}; sgdisk -n 1:0:+900G -n 2:0:+20G /dev/sd${i};done
        fi

outputs:
  OS::stack_id:
    value: {get_resource: userdata}
{% endhighlight %}
            
            
**Don't forget to map the file in the ressource_registry scope in your heat environment file**      
        
{% highlight yaml %}
[stack@undercloud ~]$ vim ~/my-cloud/infra-environment.yaml
resource_registry:  
OS::TripleO::NodeUserData: /home/stack/my-cloud/templates/firstboot/ceph_wipe.yaml

[...]
{% endhighlight %}
            
More info: [here](http://docs.openstack.org/developer/tripleo-docs/advanced_deployment/extra_config.html#firstboot-extra-configuration) or [here](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Director_Installation_and_Usage/sect-Advanced-Scenario_3_Using_the_CLI_to_Create_an_Advanced_Overcloud_with_Ceph_Nodes.html#sect-Advanced-Configuring_Ceph_Storage)

