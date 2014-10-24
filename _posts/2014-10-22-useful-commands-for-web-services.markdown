---
layout: post
title:  "Useful commands for web services"
date:   2014-10-22 18:56:51
categories: web server commands
tags: [useful commands, logs, nginx, apache, web]
---
Some helpful commands for web services:

#### List of currently established connections:

{% highlight bash %}
netstat -na | grep 'ESTA' | wc -l 

netstat -an | grep :80 | grep -i EST | wc -l
{% endhighlight %}

#### Number of IP connections and IPs connected to port 80:

{% highlight bash %}
netstat -plan | grep :80 | awk {'print $5'} | cut -d: -f 1 | sort | uniq -c | sort -nk 1
{% endhighlight %}

#### Most viewed pages 

{% highlight bash %}
awk '{print $7}' /path/to/log | sort | uniq -c | sort -rn | head -10
{% endhighlight %}

#### Top ten referrers

{% highlight bash %}
awk '{print $11}' /path/to/log | sort | uniq -c | sort -rn | head -10
{% endhighlight %}

#### Sort process by used memory

{% highlight bash %}
ps aux --sort -rss 
{% endhighlight %}
