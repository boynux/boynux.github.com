---
layout: post
title: Advanced Docker Networking
excerpt: Using Linux containers and specially Docker solves lots of deployments problems easily that would be difficult otherwise. By adding some advanced networking features make it even more powerful.
tags: Docker Networking Kernel_Namespaces
category: Sysadmins
---

<div style="float: right">
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="Load Balancing in Docker" title="Load Balancing in Docker" />
</div>

In past few years Linux containers has extremely changed the way we deploy applications. With arising of Docker and its simple way of handling containers the new concept of *Container as a service* has emerged.

Docker made things easier and harder. Encapsulating apps in containers and deploying them as immutable images, ensures that application code is decoupled from running environment as much as possible. But this concept has some consequences too: 

> Things we used to work with in VMs and Servers world are not applicable any more. Hence we have to extend our knowledge to catchup with this fast growing technologies.

I'm going to cover one of the aspects which used to be a simple tool in any sysamin tool box, but it's not any more.

### Docker Networking

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

By default Docker creates a bridge `docker0`. But sometimes is not desired to expose certain container's traffic to others and isolate them, for security reasons for instance.

Things get a little bit complicated here, since Docker does not provide any helper command to do it easily. Hence we have to do the hard job!

Let's get started!

*I assume you are using Ubuntu, if not you may need to find respective package names to access to referred tools.*

First things first, to follow this tutorial you must have `iproute2` and `bridge-utils` and `docker` packages on your machine.

    $ sudo apt-get install -y iproute2 bridge-utils
    $ curl https://get.docker.com | sh

### First scenario

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

I want to attach a simple network interface controller (NIC) to running Docker container (usually called guest-only/dummy interface). What's it good for? Well, firstly it's simple, secondly sometimes you may need a dummy interface in your container like [this post][1].

First run a container (We keep Docker container id for later references):

    $ DID=$(sudo docker run -d ubuntu:latest /bin/bash -c 'while true; do echo "running... " && sleep 1; done;')

To continue we need this Container process ID:

{% raw %}
    $ DPID=$(sudo docker inspect -f '{{.State.Pid}}' $DID)
{% endraw %}

Let's give the NIC we are going to create a name (can be any unique name):

    $ VNET="vnet${DID:0:8}"

The following commands creates a new namespace for our container process, and attached this new interface to that process namespace. So it becomes visible to our container.

    # Create a new namespace
    $ sudo mkdir -p /var/run/netns
    $ sudo ln -s /proc/$DPID/ns/net /var/run/netns/$DPID

    # Attach new dummy interface to container process namespace.
    $ sudo ip link add $VNET type dummy
    $ sudo ip link set $VNET netns $DPID

Now it's time to configure our new NIC. `iproute2` tool chain has a subset of commands that executes networking commands in specific namespaces. That's `ip netns exec`.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

    # Rename it to eth1
    $ sudo ip netns exec $DPID ip link set dev $VNET name eth1
    # Bring it up
    $ sudo ip netns exec $DPID ip link set eth1 up
    # Assign new IP address
    $ sudo ip netns exec $DPID ip addr add "172.16.0.10/24" dev eth1
    $ sudo ip netns exec $DPID ip rout add "172.16.0.10" dev eth1 scope link

`127.16.0.10/24` is the IP I set for this interface and I named it `eth1`. If all above commands where successful you can check to see if this new NIC is actually attached to your container?

    $ sudo docker exec $DID ip addr

Output of last command from my test looks like this:

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    203: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:ac:11:00:48 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.72/16 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:acff:fe11:48/64 scope link
           valid_lft forever preferred_lft forever
    205: eth1: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
        link/ether ba:06:c0:73:6d:ed brd ff:ff:ff:ff:ff:ff
        inet 172.16.0.10/24 scope global eth1
           valid_lft forever preferred_lft forever
        inet6 fe80::b806:c0ff:fe73:6ded/64 scope link
           valid_lft forever preferred_lft forever

`eth1` with assigned IP is listed as container NICs.

### Connect it to somewhere

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

If we connect this NIC to somewhere (a tunnel) it's more useful indeed. The procedure is very much the same as above. With two small differences:

1. We create an additional NIC in our host.
2. We create a tunnel between those two.

This way we can communicate between our host and container in isolated tunnel. It's usually called host-only network.

*kill previous container with `sudo docker kill $DID` if you didn't.*

Follow previous scenario steps up to to the command that I commented as `# Attach new dummy interface to container process namespace.`. But don't execute the following commands.

New we create NIC as a tunnel interface:

    $ sudo ip link add $VNET  type veth peer name vnet01
    $ sudo ip link set vnet01 up
    $ sudo ip addr add 192.168.6.1/24 dev vnet01

`vnet01` belongs to host machine and `$VNET` to container, but not added yet.

To add `vnet01` to container you should follow exactly the previous example commands:
    
    $ sudo ip link set $VNET netns $DPID

    # Rename it to eth1
    $ sudo ip netns exec $DPID ip link set dev $VNET name eth1
    # Bring it up
    $ sudo ip netns exec $DPID ip link set eth1 up
    # Assign new IP address
    $ sudo ip netns exec $DPID ip addr add "192.168.6.2/24" dev eth1
    $ sudo ip netns exec $DPID ip rout add "192.168.6.2" dev eth1 scope link

Now let's try to see it really works: 

    # Ping container new IP
    $ ping 192.168.6.2

And: `sudo docker exec $DID ip addr` shows:

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    206: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:ac:11:00:49 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.73/16 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:acff:fe11:49/64 scope link
           valid_lft forever preferred_lft forever
    209: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 7a:e2:f9:86:9e:87 brd ff:ff:ff:ff:ff:ff
        inet 192.168.6.2/24 scope global eth1
           valid_lft forever preferred_lft forever
        inet6 fe80::78e2:f9ff:fe86:9e87/64 scope link
           valid_lft forever preferred_lft forever

### Enable Internet access

That's very simple, guess! Right, just enable masquerading and IP forwarding. Should IP explain how?

### Make it a bridge

We can just create a bridge on host machine and attach host NIC ie. `vnet01` to it. Then for other container you can follow above commands and attach to the same bridge. Usually called bridge network:

<div class="ads"> 

    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

    $ brctl addbr br01
    $ brctl addif br01 vnet01

It's ready. Would you like to connect your container to any physical network? Right, add physical NIC to the same bridge. But be careful you're exposing your container to external network (security wise).

Did you find it informative? Leave your comments below (or share it to others).

[1]: {{ site.url }}/load-balancing-in-docker
