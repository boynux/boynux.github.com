---
layout: post
title: Load Balancing in Docker
excerpt: How to implement Layer-4 in Docker containers. Layer-4 Load balance is a very powerful method and adding that to Docker eco-system is very interesting.
tags: Docker Load_Balancing Container LVS Route Networking
category: Sysadmins
---

<div style="float: right">
<img src="{{ site.url }}/img/lvs4p-200.gif" width="161" alt="Load Balancing in Docker" title="Load Balancing in Docker" />
<br />
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="Load Balancing in Docker" title="Load Balancing in Docker" />
</div>

In [previous post][1] I explained a little bit about different methods of load balancing. I promised to cover *Direct route* method in another post. Here it is.

### Demo time

For the demo I'm going to use a very powerful tool in Linux kernel, [LVS][3]. This module provides different methods for load balance traffics. Including NAT, Tunneling and Direct Route. Here I'm going to implement Direct Route method within [Docker][2] containers.

#### Prerequisites

I'm using Ubuntu, other distros should work too. But commands and installations might but a little bit different.

*I assume that you have decent knowledge of Linux, Docker and Linux Networking. Otherwise it might be a little bit difficult for you to understand the concept.*

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

1. This setup require Docker on your system. If you are using Ubuntu 14.04 or later version, you can install latest Docker daemon with the following command: 

{% highlight bash linenos %}
$ curl http://get.docker.com | sh
$ sudo usermod -aG docker $USER
{% endhighlight %}

Now re-login to your ssytem.

2. We need IPV kernel module on your **Docker Host** machines. In order to install that in Ubuntu issue the following command:

{% highlight bash linenos %}
$ sudo apt-get install ipvsadm
{% endhighlight %}

3. To make sure that every thing is fine try the following commands:

{% highlight bash linenos %}
$ docker run hello-world
$ sudo ipvsadm -l
{% endhighlight %}

4. Get the following source code from GIT.

{% highlight bash linenos %}
$ git clone https://github.com/boynux/docker-load-balancer
{% endhighlight %}

There shouldn't be any error from execution of above commands.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

#### Building images

For this demo, I'm going to create 3 Docker images with Nginx installed. They just show a simple HTML page with their hostname in it. This way we can see how load balancing happens and every time you refresh pages it'll show different hostname.

Go to the source code directory you just pulled from github.

1. Create Nginx Docker image:

{% highlight bash linenos %}
$ cd docker-load-balancer
$ cd nginx
$ docker build -t boynux:nginx .
{% endhighlight %}

2. Create IPVS image:

{% highlight bash linenos %}
$ cd docker-load-balancer
$ cd ipvs
$ docker build -t boynux:ipvs .
{% endhighlight %}

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

#### Running the demo

Now to run the whole setup issue the following command:
Please note that the following IP could be any IP. This is entry point to load balancer. Just make sure it's not within your computer or Docker subnet.

    $ cd docker-load-balancer
    $ ./run 172.20.10.1

Now if you try to open that IP (above IP) in your browser or with curl. If you refresh your system the hostname will change.

To stop the demo just press ctrl-c as instructed. It'll clean-up everything.
For more details you can see code. It's very simple and easy to understand.

Any questions? Please leave comments.

[1]: {{ site.url }}/load-balancing-mthods
[2]: http://docker.com
[3]: http://www.linuxvirtualserver.org/index.html
