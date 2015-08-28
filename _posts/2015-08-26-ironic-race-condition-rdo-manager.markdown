---
layout: post
title:  "Ironic Race Condition in RDO Manager Deployment"
date:   2015-08-26 10:30:51
categories: rdo rdo-manager ospd ironic nova
tags: [rdo, rdo-manager, ospd, ironic, nova, race condition, instance, deployment]
---
For a deployment with RDO Manager, I encountered a problem with Ironic & Nova during a deployment of ~30 physical servers.

By default, Nova is able to launch 10 instances builds to run concurrently.
But, actually, Ironic can't deal with it and cause a race condition...

The problem is Ironic try to attach a wrong profile/instance to the node.
_Ex: a Compute profile on a Controller server._

{% highlight bash %}
2015-08-24 18:18:05.228 1200 ERROR oslo_messaging.rpc.dispatcher [-] Exception during message handling: Node 702d0321-5e9d-49f2-b914-0831bcbdfebd is associated with instance ccc2dc87-81b1-4769-8019-ffa5e3ecb979.
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher Traceback (most recent call last):
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo_messaging/rpc/dispatcher.py", line 142, in _dispatch_and_reply
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     executor_callback))
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo_messaging/rpc/dispatcher.py", line 186, in _dispatch
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     executor_callback)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo_messaging/rpc/dispatcher.py", line 130, in _do_dispatch
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     result = func(ctxt, **new_args)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo_messaging/rpc/server.py", line 142, in inner
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     return func(*args, **kwargs)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/conductor/manager.py", line 405, in update_node
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     node_obj.save()
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/objects/base.py", line 143, in wrapper
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     return fn(self, ctxt, *args, **kwargs)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/objects/node.py", line 265, in save
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     self.dbapi.update_node(self.uuid, updates)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/db/sqlalchemy/api.py", line 338, in update_node
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     return self._do_update_node(node_id, values)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/db/sqlalchemy/api.py", line 364, in _do_update_node
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher     instance=ref.instance_uuid)
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher __NodeAssociated: Node 702d0321-5e9d-49f2-b914-0831bcbdfebd is associated with instance ccc2dc87-81b1-4769-8019-ffa5e3ecb979.__
2015-08-24 18:18:05.228 1200 TRACE oslo_messaging.rpc.dispatcher
{% endhighlight %}

I've found a workaround who consist to:
* Decrease the number of max concurrent builds from *10* to *2* 
* Reduce the pool size of RPC thread from *64* to *4*

In Nova conf file :

/etc/nova/nova.conf

{% highlight bash %}
max_concurrent_builds=2
{% endhighlight %}

In Ironic conf file :

/etc/ironic/ironic.conf

{% highlight bash %}
rpc_thread_pool_size = 4
{% endhighlight %}
