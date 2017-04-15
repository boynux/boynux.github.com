---
layout: post
published: true
title: Containers should be Fat or Thin?
excerpt: There are lots of discussions around the net about containers to be fat or thin. Some say they meant to be thin and oppositions insist that you need to have some other processes too. What's the way to go?
tags:
  - Docker
  - Containers
category: Devops
---

### Introduction

<div style="float: right">
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="Load Balancing in Docker" title="Load Balancing in Docker" />
</div>

I used to use containers for about 3 years ago to deploy small apps inside LXC Containers in our private data-center. So naturally because of the way LXC installs a base OS inside a container, it goes through the whole init process. Meaning that I was almost booting a minimal Linux (Ubuntu) inside containers. And nothings seemed to be wrong since my experience was mainly with virtual machines and they look similar.

I was using containers only because they were lighter and easier to replicate and archive with `squashfs` and `aufs` to manage base images.

<div style="float: left; width: 320px">
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

But with Docker and its fantastic interface to LXC things start to change. Instead of building the whole `Squashfs` image and mounting `aufs` layers with lots of bash scripts, now you can use a simple API and client to do the whole thing in a very similar way.

Something else has changed too, Docker by introducing Docker hub and lots of different and ready to use images, advocates an *only one process per container* philosophy. They even sometime later in a blog post mentioned it as a [lean and simple container][3] and discouraged a much more complicated multiprocess containers.

So to me like many others at first it sounds a little bit strange, I need some tools to access into my containers and that's normally a SSH server. Then I need some other monitoring tools as well like SAR or maybe `monit`. But after a moment of thought I realized that I might not need all that stuff. It can be much more simpler without all that.

But it turns out not all people out there thinking in a same way. And there are plenty of valid reasons why one might need to run multiple processes inside a container.

Things got more hot when [Phusion][1] guys created a modified Ubuntu image called [BaseImage][2]. That's nothing but a simple Ubuntu image to run multiple processes inside a container. Init processes is handled by `runit` and you can simply run let's say `SSHD` server along with your main process. Then they created a blog post about that to defend their idea and explain the reasoning behind this concept.

### What really matters

But beside all these fights and debates, what we should logically do and how we should treat containers? Here I'm going to list a few valid and invalid reason that might lead us to use a simple container as more complex units.

This is just my thoughts about this concept and I may miss some use cases, so if you see it's so please let me know. I'll be glad to add that to my list. And finally I'm not against fat containers at all but I'm more toward thin containers. So you may find this reasonings a little bit biased as well (Just to be honest).

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

### When not to use FAT containers:

First I'll go through some situations that may lead you toward multiprocess containers, but they are not actually that justifiable. So I explain why I think there are better option than multiprocess containers in those situations.

#### **Connecting two or more processes.**

Except a few rare cases by using `volumes` and `network` ports you're able to connect two or more application processes to each other and indeed it's much easier to handle [Docker provides `--link` option to for even simpler solution], monitor and audit them in different containers.  I'll cover exceptions in next section.

#### **Debugging your application.**

This one is very interesting. If you want to debug your application inside docker image you really don't need to run multiple processes inside that. The way docker works (and most other containers) when it starts, simply runs your process inside a kernel namespace. So you can directly access it within host machine. For example:

{% raw %}
    $ ID=$(docker run -d ubuntu /bin/bash -c 'while true; do echo hello world; sleep 1; done')
    $ $PID = $(docker inspect -f '{{.State.Pid}}' $ID)
    $ sudo strace -p $PID
{% endraw %}
#### **Monitoring**

This one is almost same as above. To monitor a process you just need to monitor container process. You rarely need to access inside container for that matters. Memory, CPU, Load and all that can be monitored from container itself.

{% raw %}
    $ ID=$(docker run -d ubuntu /bin/bash -c 'while true; do echo hello world; sleep 1; done')
    $ $PID = $(docker inspect -f '{{.State.Pid}}' $ID)
    $ sudo cat /proc/$PID/cmdline
{% endraw %}

#### **Directly modifying containers**

While I see this as a *Container Anti-Pattern* but if it's a requirement you can use Docker `exec` command to achieve that:

{% raw %}
    $ ID=$(docker run -d ubuntu /bin/bash -c 'while true; do echo hello world; sleep 1; done')
    $ $PID = $(docker inspect -f '{{.State.Pid}}' $ID)
    $ docker exec $PID ip add add 192.168.10.1/24 dev eth0
{% endraw %}
 
#### **Performance**

If you're thinking one container is going to use less resources, then you may need to rethink again. Containers are just your processes in isolated environment, so there's absolutely no performance overhead and considering `init` process in more complicated containers then single process containers may even have a better performance.

#### **Ease of deployment**

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

You may think deploying a single container is much easier than multiple. But considering deployment is not end of the story and you still need to monitor your processes, I would say a little bit more trouble with deployment of multiple containers definitely worth eliminating a lot of overhead of monitoring multiple processes inside one container. Docker by itself can restart your container if the process (container) dies. So if you can split your processes into separate containers then you can easily let docker take care of its execution. 

But if you run multiple processes inside a container then you can't use this feature that much. Because if a process dies inside a multiprocess container, Docker has no way to know it. You need another process inside a container like `monit` to handle the failure. And if `monit` dies? Then you might need `init` process itself or a `cron` job to take that responsibility!

#### **Auditing**

If you're providing some sort of SaaS service to your customers and want to charge them based on resources they use, then you may think packing all processes for that customer inside on container is the way to go. But if you ask me, I think the overhead of monitoring to provide a reasonable SLA to your customer might not worth that. You can still charge base on total amount of resources or IOPs that multiple containers use.

### When to use FAT containers:

And now let's see why you may consider to use fat containers.

#### **Tightly coupled processes**

In some cases your processes might communicate via SHM or other IPC methods and since Docker containers are isolated into kernel namespaces definitely it's not going to work out of the box. Although with some hacks (?) you can run multiple containers into a single namespace and you might be able to make it work this way but it's too complicated.

#### **Processes that spawn another one**

Another situation would be one process controlling and monitoring other processes. In such cases seems the most straight forward choice would be multiple processes inside on container. Although in this situation if you're willing to change you application code still you can change you supervisor to spawn new containers instead of single processes. That's pretty much what container management systems do.

#### **Migrating from real VMs**

If you're migrating from real VMs or physical servers to containers world, then it might be easier for your team to just move the whole application into a full Linux container. This is less trouble because the team does not need to learn a lot things at once and moving to containers will be much easier. But I recommend to slowly break your processes to multiple container after successful migration. So you can get the most benefit out of containers.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

### Final note

If you're really aiming to use multiple processes inside a container then  I don't recommend to use Docker at all, it's just not meant for such cases. I suggest to take a look at [LXC][5] and [OpenVZ][6] solutions and if you just want to experiment you can try [LXD][7] which is not production ready yet, but it's yet very powerful. 

These container management tools are designed to provide you a full Linux experiment inside containers and from security aspect they are much more secure than Docker.

If you're interested to use LXC and looking for a workflow similar to Docker for that you can take a look at [this post][8] which explains how to share the base Linux fs with several containers using [Squashfs][8] and [aufs][9].

[1]: https://github.com/phusion
[2]: https://github.com/phusion/baseimage-docker
[3]: https://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/
[4]: https://blog.phusion.nl/2015/01/20/baseimage-docker-fat-containers-treating-containers-vms/
[5]: https://linuxcontainers.org/lxc/introduction/
[6]: https://openvz.org/Main_Page
[7]: https://linuxcontainers.org/lxd/introduction/
[8]: http://en.wikipedia.org/wiki/SquashFS
[9]: http://en.wikipedia.org/wiki/Aufs
