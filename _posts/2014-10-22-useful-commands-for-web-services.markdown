---
layout: post
title:  "Useful commands for web services"
date:   2014-10-22 18:56:51
categories: web server commands
---
Some helpful commands for web services:

#### List of currently established connections:

{% highlight bash %}
netstat -na | grep 'ESTA' | wc -l 

netstat -an | grep :80 | grep -i EST | wc -l
{% endhighlight %}

#### Number of IP connections and IPs connected to port 80:

{% highlight bash %}
netstat -plan|grep :80|awk {'print $5'}|cut -d: -f 1|sort|uniq -c|sort -nk 1
{% endhighlight %}

