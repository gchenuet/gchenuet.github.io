---
layout: post
title:  "Ironic Race Condition in RDO Manager Deployment"
date:   2015-08-26 10:30:51
categories: rdo rdo-manager ospd ironic nova
tags: [rdo, rdo-manager, ospd, ironic, nova, race condition, instance, deployment]
---
With RDO Manager, I encountered a problem with Ironic & Nova during a deployment of ~30 physical servers.

By default, Nova is able to launch 10 instances builds to run concurrently.  
But actually, Ironic can't deal with it and cause a race condition...

The problem is Ironic try to attach a wrong profile/instance to the node.   
_Ex: Compute profile on a Controller server._    

{% highlight bash %}
2015-08-24 18:18:05.228 1200 ERROR oslo-messaging.rpc.dispatcher [-] Exception during message handling: Node 702d0321-5e9d-49f2-b914-0831bcbdfebd is associated with instance ccc2dc87-81b1-4769-8019-ffa5e3ecb979.
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher Traceback (most recent call last):
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo-messaging/rpc/dispatcher.py", line 142, in dispatch-and-reply
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     executor-callback))
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo-messaging/rpc/dispatcher.py", line 186, in dispatch
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     executor-callback)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo-messaging/rpc/dispatcher.py", line 130, in do-dispatch
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     result = func(ctxt, new-args)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo-messaging/rpc/server.py", line 142, in inner
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     return func(args, kwargs)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/conductor/manager.py", line 405, in update-node
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     node-obj.save()
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/objects/base.py", line 143, in wrapper
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     return fn(self, ctxt, args, kwargs)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/objects/node.py", line 265, in save
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     self.dbapi.update-node(self.uuid, updates)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/db/sqlalchemy/api.py", line 338, in update-node
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     return self.-do-update-node(node-id, values)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/ironic/db/sqlalchemy/api.py", line 364, in do-update-node
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher     instance=ref.instance-uuid)
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher **NodeAssociated: Node 702d0321-5e9d-49f2-b914-0831bcbdfebd is associated with instance ccc2dc87-81b1-4769-8019-ffa5e3ecb979.**
2015-08-24 18:18:05.228 1200 TRACE oslo-messaging.rpc.dispatcher
{% endhighlight %}

The workaround consists to:   
*    Decrease the number of max concurrent builds from **10** to **2**   
*    Reduce the pool size of RPC thread from **64** to **4**

**/etc/nova/nova.conf**  
{% highlight bash %}
max_concurrent_builds=2
{% endhighlight %}

**/etc/ironic/ironic.conf**  
{% highlight bash %}
rpc_thread-pool_size = 4
{% endhighlight %}    

Don't forget to restart Ironic & Nova services.    
This is the _default_ value, you can increase them to find a better adjustment.
