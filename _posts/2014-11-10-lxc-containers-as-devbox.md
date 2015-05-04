---
layout: post
title: Using LXC Containers as DevBox
excerpt: LXC Containers are light and fast, doesn't consume lots of reasources and are very suitlable as a DevBox. But using them is tricky. Here is some practices to make them easier to use.
---

I've been using LXC containers as a DevBox (ie. development environment) and also deployment containers in past few years. I've seen many developers using other tools like Vagrant + Virtual box and other full virtualizations and it's very common. Advantages of using container for developent boxes are, they are light weight and fast. During the time I observed some developers failing to use containers mainly because they don't know how to use them efficiently.

Introduction:
=========
<a href="http://www.boynux.com/wp-content/uploads/2014/11/Thalassa_Hellas_container_ship_on_its_maiden_voyage_on_the_river_Elbe.png"><img src="http://www.boynux.com/wp-content/uploads/2014/11/Thalassa_Hellas_container_ship_on_its_maiden_voyage_on_the_river_Elbe.png" alt="Thalassa_Hellas_container_ship_on_its_maiden_voyage_on_the_river_Elbe" width="400" height="300" class="alignnone size-full wp-image-1123" alt="Containers as DevBox" /></a>

During this time I developed a few good patterns that helped me to use containers much easier.

> LXC comes with a very low level tool chain and unlike more high level hypervisors like ***VirtualBox*** utilizing it, is much more difficult and tedious.

<div class="ads">
<!-- Responsive Display -->
<ins class="adsbygoogle adslot_1"
     style="display:block"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="4587256441"
     data-ad-format="horizontal"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

Problems:
=======
* One of the main reasons that developers stay away from containers is that, there is not any out-of-box tool to package containers is a single file and transfer or share them.
* The other reason is creating or cloning new containers is sometimes difficult and bogus.
* Setting up new container is not always a straight forward and easy task.

Patterns:
=========
> There are a few patterns that I'm going to share here to make your life easier when dealing with containers and maybe you'll become a fan of containers like me.

Here is a short list of what you'll need to know:

* Squashfs
* AUFS
* Bash scripting
* LXC basic commands

First things to do:
------------------
for host environment you will need:

* LXC tool chain
* apt-cacher-ng (in case of Debian based containers)
* Squashfs tools
* AUFS tools

If you are using Mac or Windows as a host you should also install any Linux distro (you don't need desktop just command line) in a VirtualBox or Hyper-V or any other hypervisor.

> Whenever I refer to host, I mean a Linux host which is hosting your containers

You should create a base or template container. What I mean by template container is the minimum OS setup that you use mostly in your DevBox. For me it's just as little as:

* OS version (I mainly use Ubuntu - 12.04/14.04)
* SCM tools (Git and Mercurial)
* Editor (VIM + Several plugins and tools)
* Programming environment (Python + Pip + tools or PHP + Apache + tools)
* Shell (Zsh or bash + tools)

In order to create a template container for the first time you'll need to use LXC tool chain as it is. But later on you just need to use your custom script unless you want to create another template from scratch.

Setting up initial template:
--------------------------
By using `lxc-create` tool you can create your initial template container. Here I just call my template `ubuntu` with Ubuntu as an OS of my choice and 14.04 LTS is the release.

    $ sudo lxc-create -n ubuntu -t ubuntu -- -r 14.04

It'll take a while depending your internet speed to complete container installation. When it's done you need to login to your container and setup the rest. You can `ssh` to your container using the following command:

    $ sudo lxc-start -n ubutnu -d
    $ ssh ubuntu@ubuntu

First thing to do here is to setup `apt-caceher-ng`, feel free to skip it if you don't need that. But I highly recommend using that specially if you're using reasonable amount of containers. Remember you'll need to setup `apt-cacher-ng` in your host before this step. By default LXC uses `10.0.3.1` as a default gateway which is your host IP, you'll need to change it if you're using different IP.

    $ echo 'Acquire::http::Proxy "http://10.0.3.1:3142"' | sudo tee /etc/apt/apt.conf.d/00proxy
    $ sudo apt-get update
    $ sudo apt-get upgrade

Now continue setting up your basic requirements, for instance let's setup a simple PHP DevBox with Git and Zsh:

    $ sudo apt-get install zsh php5 apache2 mysql-server vim-nox git

    # Enable userdir apache modules
    $ mkdir public_html
    $ a2enmod userdir

    # Setup zsh
    $ sudo usermod -s /bin/zsh ubuntu

    # Enable no password for ubuntu user (default user)
    $ echo "ubuntu ALL=NOPASSWD:ALL" | sudo tee -a /etc/sudoers

    # You can setup your VIM dotfiles also
    # Optionally you can copy your host public key in .ssh/authorized_keys
    # and ...

<div class="ads">
<!-- Responsive Display -->
<ins class="adsbygoogle adslot_1"
     style="display:block"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="4587256441"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

Create template:
---------------
Now it's time to convert your container to the actual reusable container template. In order to do this the architecture of containers will be like this:

      Container
    --------------
      Aufs (rw)
    --------------
    Squashfs (ro)

The idea is to have a base Squashfs file system that represents our template (from the previous step). This layer will be shared with all containers and since it's read only non can modify it. To enable second layer of read/write filesystem, I'm going to use Aufs. So every container will have it's own Aufs layer to store new and modified files.

So, let's create our squashfs image from template.

    $ sudo lxc-stop -n ubuntu # Stop container if it's already running
    $ sudo mkdir /var/lib/lxc/images # creates image directory to store templates
    $ sudo mksquashfs -b 65535 /var/lib/lxc/ubuntu/rootfs /var/lib/lxc/images/ubuntu.sqfs

***Note that default `lxc` container path in Ubuntu is `/var/lib/lxc` other distros might be different please check before proceeding with this step.***

Setting up Aufs layer:
----------------------

At this stage we need to setup Aufs (rw) layer on top of squashfs in order to be able to write content over squashfs. This is the directory structure that I use, I recommend to use that as well but feel free to change it according to your taste.

    +/var
        +lib/
            +lxc/
                +images/
                    ubuntu.sqfs
                +ubuntu/         # container
                    +rootfs      # this shoud be empty
                    +rootfs.rw
                    +rootfs.ro   # this should be empty
                    +config      # this is LXC config file

So you'll need to create `rootfs.ro` and `rootfs.rw` directories and also you must removes all files and directories inside `rootfs`, we don't need them anymore.

    $ sudo mkdir /var/lib/lxc/ubuntu/rootfs.{rw,ro}
    $ sudo rm -rf /var/lib/lxc/ubuntu/rootfs/*

Now we are ready to setup our container filesystem:

    # mount squashfs image
    $ sudo mount -t squashfs /var/lib/lxc/images/ubuntu.sqfs /var/lib/lxc/ubuntu/rootfs.ro
    
    # mount aufs on top of squashfs in rootfs
    $ sudo mount -t aufs -o br:/var/lib/lxc/ubuntu/rootfs.rw=rw:/var/lib/lxc/ubuntu/rootfs.ro=ro,xino=/dev/shm/ubuntu.xino none /var/lib/lxc/ubuntu/rootfs

That's all now we are ready to start our container again. You can do it by just using `lxc-start` command:

    $ sudo lxc-start -n ubuntu -d

ssh to your container again and make  sure everything works as it should.

Create new container based on template:
---------------------------------------

Now here is the fun part, when you need to create new container based to your template (`ubuntu` in our case) you just need to create a container folder structure based on what I've provided above and change at least MAC address and container's name in new container's config. 

Let's do it:

    $ sudo mkdir -p /var/lib/lxc/container-1/rootfs{,rw,ro}
    $ sudo cp /var/lib/lxc/{ubuntu,container-1}/config
    $ sudo $EDITOR /var/lib/lxc/container-1/config # change what you need here

And done! In a few seconds you have your new container ready and setup! Now just run the above `mount` commands again and start your new container.

<div class="ads">
<!-- Responsive Display -->
<ins class="adsbygoogle adslot_1"
     style="display:block"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="4587256441"
     data-ad-format="rectangle"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>


    # mount squashfs image
    $ sudo mount -t squashfs /var/lib/lxc/images/ubuntu.sqfs /var/lib/lxc/container-1/rootfs.ro
    
    # mount aufs on top of squashfs in rootfs
    $ sudo mount -t aufs -o br:/var/lib/lxc/container-1/rootfs.rw=rw:/var/lib/lxc/container-1/rootfs.ro=ro,xino=/dev/shm/container-1.xino none /var/lib/lxc/container-1/rootfs

    # Start contaienr and connect to it
    $ sudo lxc-start -n container-1 -d
    $ sudo lxc-attach -n container-1

Make things even simpler:
-----------------------
During the time I developed my own set of bash scripts that actually does the whole job for me much easier. Except creating sqashfs image template which needs to be done manually. Since I rarely setup new image and I have a few images for day to day use I didn't bother to create a script for this part, if you are not as lazy as me please share yours with me ;)

> You can find scripts in my [GitHub](https://github.com/boynux/LXCTools) account.

You can use them similar to LXC toolchain:

    # to create a new container from template
    $ LXCTools/lxc-clone <tempalate name> <new name>

    # to start a container and SSH to it (one command)
    $ LXCTools/lxc-ssh <container name>

    # To satrt container and setup file sharing, port forwarding and SSH key transfer
    $ LXCTools/lxc-setup <container name>

Final notes:
------------

I created all these patterns and scripts just for my personal use, and I'm aware that they certainly have a lot of shortages. Please provide you suggestions and fixes  or features through GitHub and leave your comments below. 

References:
-----------

* [LXC](https://linuxcontainers.org/)
* [Squashfs](http://squashfs.sourceforge.net/)
* [AUFS](http://aufs.sourceforge.net/aufs.html)
