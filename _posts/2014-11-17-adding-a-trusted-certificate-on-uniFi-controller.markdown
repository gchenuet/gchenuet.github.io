layout: post
title:  "Adding a trusted certificate on UniFi Controller"
date:   2014-11-17 14:00:31
categories: unfi ssl trusted ubiquiti
tags: [ubiquiti, unifi, ssl, wifi, java]
---

At work, I had to replace the self-signed certificate on our Ubiquiti UniFi Controller with a trusted certificate.

It is pretty simple and requires only two commands:

*Note that unifi.chain.crt needs to contain the complete CA chain, including the intermediate CAs and the root CA.* 

{% highlight bash %}
openssl pkcs12 -export -in unifi.crt -inkey unifi.key -certfile unifi.chain.crt -out unifi.p12 -name unifi -password pass:aircontrolenterprise
{% endhighlight %}

{% highlight bash %}
keytool -importkeystore -srckeystore unifi.p12 -srcstoretype PKCS12 -srcstorepass aircontrolenterprise -destkeystore /usr/lib/unifi/data/keystore -storepass aircontrolenterprise
{% endhighlight %}

{% highlight bash %}
service unifi restart 
{% endhighlight %} 
