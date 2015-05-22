---
layout: post
published: false
title: Containers should be Fat or Thin?
excerpt: There are lots of discussions around the net about containers to be fat or thin. Some say they meant to be thin and oppositions insist that you need to have some other processes too. What's the way to go?
tags:
  - Docker
  - Containers
category: Devops
---

### Introduction

I was using containers for about 3 years ago to deploy small apps inside LXC Containers in our private data-center. So naturally because of the way LXC installs base OS inside container the it goes through the whole init process. Meaning that I was almost booting a minimal Linux (ubuntu) inside containers. And nothings seemed to be wrong since my experience was mainly with virtual machines and they tend to be the same.

I was using containers only because they were lighter and easier to replicate and archive since I was using `squashfs` and `aufs` to manage base images.

But with docker and its fantastic wrapper around LXC things went much easier. Instead of building the whole `Squashfs` image and mounting `aufs` layers with lots of bash scripts, now you can use a simple API and client to do the whole thing.

Something else has changed too, Docker by introducing Docker hub and lots of different and ready to use images, encouraged people to limit container to *only one process*. They even sometime later in a blog post mentioned that as a [lean and simple container][3] and discouraged a much more complicated multiprocess containers.

So for like many others at first it sounded a little bit strange, I need some tools to access into my containers and that's normally a SSH server. Then I need some other monitoring tools as well like SAR or maybe `monit`. But after a moment of thought I realized I might not need all that stuff. It can be much more simpler without all that.

But it turns out not all people out there think the same way. And there are plenty of valid reasons why one might need to run multiple processes inside a container.

Things got more hot when [Phusion][1] guys created a modified Ubuntu image called [BaseImage][2]. That's nothing but a simple Ubuntu image to run multiple processes inside a container. Init processes is handled by `runin` and you can simply run let's say `SSHD` server along with your main process. Then they created a blog post about that to defend their idea and explain the reasoning behind using this concept.

### What really matters

But beside all these fights and debates, what we should logically do and how we should treat containers? Here I'm going to list a few valid and invalid reason that might lead you to treat a simple container as more complex units.

This is just my thoughts about this concept and I may miss some use cases, so if you see it's so please let me know. I'll be glad to add that to my list. And finally I'm not against fat containers at all but I'm more toward thin containers. So you may find this reasonings a little bit biased as well (Just to be honest).

### When not to use FAT containers.

1. Accessing into container

This one falls into several categories. The reason for accessing inside containers could be collecting logs to monitoring process. Docker provides an easy way to access containers and run simple commands to get state of a process for instance and that's `docker exec`. So for monitoring purpose you really don't need to take all the trouble to have a fat container.

For things like accessing log files and other stuff you can use (and better to use) Docker volumes. By mounting docker containers a volume with `--volume` argument you can simple expose those information to the host machine (or other containers) and use them.

And finally if you want to see stdout of container again `docker logs` command is your best friend. And of course it works with single process containers.

2. Connecting two or more processes.

Except a few rare cases by using `volumes` and `network` ports you're able to connect to application processes to each other and indeed it's much easier to handle, monitor and audit them in different containers.

3. Debugging your application.

This one is very interesting. If you want to debug your application inside docker image you really don't need to run multiple processes inside that. The way docker works (and most other containers) when it starts it simply 

[1]: https://github.com/phusion
[2]: https://github.com/phusion/baseimage-docker
[3]: https://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/
[4]: https://blog.phusion.nl/2015/01/20/baseimage-docker-fat-containers-treating-containers-vms/
