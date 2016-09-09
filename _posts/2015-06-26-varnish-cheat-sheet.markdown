---
layout: post
title:  "Varnish Cheat Sheet"
date:   2015-06-26 10:30:51
categories: varnish commands
tags: [useful commands, logs, varnish, web, caching, cache]
---
Helpful commands about Varnish Cache:
<!--excerpt-->
## Varnish > V3

#### Look at an incoming client request of a specific URL:

{% highlight bash %}
varnishlog -c -m RxURL:"post/123.htm"
{% endhighlight %}

#### Look at a a backend request of a specific URL:

{% highlight bash %}
varnishlog -b -m TxURL:"post/123.htm"
{% endhighlight %}

#### See requests for one specific hostname: 

{% highlight bash %}
varnishlog -c -m RxHeader:"Host: yauuu.me"
{% endhighlight %}

#### See the age of the cache objects for a specific hostname:

{% highlight bash %}
varnishlog -c -m RxHeader:"Host: yauuu.me" | grep Age
{% endhighlight %}

## Varnish < V3

#### Look at an incoming client request of a specific URL:

{% highlight bash %}
varnishlog -c -o RxURL post/123.htm
{% endhighlight %}

#### Look at a a backend request of a specific URL:

{% highlight bash %}
varnishlog -b -o TxURL post/123.htm
{% endhighlight %}

#### See requests for one specific hostname:

{% highlight bash %}
varnishlog -c -o RxHeader "Host: yauuu.me"
{% endhighlight %}

#### See the age of the cache objects for a specific hostname:

{% highlight bash %}
varnishlog -c -o RxHeader "Host: yauuu.me" | grep Age
{% endhighlight %}
