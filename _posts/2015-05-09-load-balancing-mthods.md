---
layout: post
title: Load Balancing Methods
excerpt: Load balancing is always an intriguing topic to me. There are plenty of ways to create a load balancer, but here I'm going to cover compelling ones in different network layers and compare them.
category: SysAdmins
tags: Load_Balance Proxy Routing NAT Anycast
---

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

When it comes to load balancing a few well known technologies pops up in mind. But before diving into them let's see how many load balancing types we have. Typically application load balancer fall into these categories (included few common examples for each):

<sub><i>These are just a handful of possible methods not all</i></sub>

+ Layer 2 Load Balancing (Data Link layer)
  + Port bundling

+ Layer 4 Load Balancing (Transport layer)
  + Network address translation (NAT)
  + Anycast
  + Direct Routing

+ Layer 7 Load Balancing (Application layer)
  + HTTP Load balancers (aka. Proxy)

What's the difference? Each solution has its own pros and cons, that's why they all exist, right? Let's just touch each one to see what's it about.

### Layer 2

Also known as `port bundling` or `port bonding` (different brands have different terms, but essentially have similar concepts). This kind of load balancers bond two or more physical links into a wider logical link. Since this works at layer 2 there is not much control over it. At the end it looks like a normal physical link.

### Layer 4

This one is particularly interesting for me. There are couple of different methods in this category. 

#### NAT

[<img src="{{ site.url }}/img/load-balancer-nat.png" alt="Load Balancing Methods NAT" title="Load Balancing Methods NAT" width="320" />][1]

The most well known method is `NAT`. `NAT` works with IPv4 protocol. Typically there is one single public IP exposed as a server/application address known as `virtual IP address`. Actual servers using a private IP subnet. Router is assigned one private IP address in the same subnet too and that `virtual IP address`. Each request to `virtual address` is distributed among real servers by changing `destination IP` and `port` according to load balancer configuration. Connection information is stored and upon receiving response from server, original IP and port are rewritten. Usually any consequent request from the same client (usually in a certain time frame) is redirected to the same server.

The main problem with NAT is network overhead, since every request should be tracked and all information about any particular route should be stored and retrieved for every single request and related response.

This method does not utilize band with efficiently too. For example if router link to public network is Ethernet (100 Mbps), load balancer can handle maximum 100Mpbs combined traffic. Even though there are multiple servers with maybe higher band width. The reason is clear, router has to handle all request and response traffics.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

#### Anycast

Anycast is another method and implemented in IPv6 protocol. It's possible to implement anycast for IPv4 but it's not that easy and requires dynamic routing. Anycast can't be used for stateful requests and can't remember routes although some routers can use cached routes but it's very limited.

This method is specially useful to distribute traffic over different geolocations.

### Routing

[<img src="{{ site.url }}/img/load-balancer-routing.png" alt="Load Balancing Methods Routing" title="Load Balancing Methods Routing" width="320" />][2]

This is very similar to NAT method but there is no address translation. What happens is, when router receives a request from client, it finds proper server to handle the request, store connection information, and delivers the packet to server. Server has exactly the same `virtual IP` as router. When it receives the packet, destination IP address is its own IP.

After processing packet it delivers response directly to client, not via router.

What's good about it? The two limitations I highlighted in NAT is not here anymore. First of all router only processes requests not responses and doesn't need to alter packet headers (ie. NAT). Secondly, since router handles traffic only in `one direction` we can utilize maximum bandwidth of servers. For example if router has an Ethernet (100 Mpbs) interface but there are 3 server and each one has an Ethernet too, we can utilize up to 300 Mbps.

### Layer 7

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

#### HTTP Proxies

This is the widely used method. Why? Because it's simple to use. HTTP proxies as their name says, can be used only for HTTP requests. `Apache`, `NginX`, `Squid`, `Privoxy` to name a few. There are commercial versions come in all sort of hardware and software too.

While they are easy to setup and pretty popular and can handle some specific HTTP features like altering cookies and handling HTTP sessions, there are some critical problems with them too. 

First of all they have all problems that NAT type has. With one extra problem. *They are not scalable, because they are working in layer 7* (Application layer) they have to process every single request like a real web server. Reading all those information requires lots of resources. Hence they can not really handle too much requests and typically they are `vertically` scalable not horizontally.

Although they can be mixed with one of previous methods to overcome these limitations. But on their own not very suitable for large scale applications with tons of requests.

### Conclusion

It's up to you which method to use. Depending on your application and requirements all these methods can be used as-is or combinations of two or more. There are plenty of customizations around many of them to make them more efficient and easy to use.

If you are currently using any of these methods I'm very excited to hear your experiences. Please leave comment or tweet me about that.

I'm personally fan of `Direct Routing` method and in next post I'll explain how to implement that with `Docker containers. Keep in touch!

[1]: {{ site.url }}/img/load-balancer-nat.png
[2]: {{ site.url }}/img/load-balancer-routing.png
