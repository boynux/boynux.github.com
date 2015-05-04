---
layout: post
title: Raspberry Pi VPN Server
excerpt: How to quickly set up a VPN server using Raspberry Pi, It's very easy to have Raspberry Pi VPN server at home. No external keyboard or HDMI needed to do this tutorial. All you need is ...
---

I needed a way to access my home private network remotely. I decided to utilize existing **Raspberry Pi** as a VPN server. I'm going to install `Raspbian` as choice of operating system. 

[<img class="size-medium wp-image-900 alignright" title="Rapberry Pi VPN" alt="Raspberry Pi" src="http://www.boynux.com/wp-content/uploads/2014/04/Raspi_Colour_R-248x300.png" width="248" height="300" />][1] <script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>
<ins class="adsbygoogle" style="display: inline-block; width: 336px; height: 280px;" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7819924448"></ins><script type="text/javascript">// 
(adsbygoogle = window.adsbygoogle || []).push({});
// </script>

## 1\. Requirement:

*   Raspberry Pi
*   SD card (min 8Gb)
*   SD Card reader to use with your Notebook/Desktop
*   Another ready to use Linux OS (Notebook, Desktop or ...)
*   LAN Cable to connect Raspberry Pi to home network.
*   USB cable + power adapter for Raspberry Pi power connection (Mobile charger!).

## 2\. Partitioning: 
At least two partitions required, but choice of more partitions is up to use. I choose the minimum. One FAT filesystem for boot and other one `ext4` as Linux `root`. You can use any disk management too, I use `parted`:

    $ sudo parted /dev/mmcblk0

    (parted) mktable MSDOS
    (parted) mkpart primary 1 512MB
    (parted) mkpart primary 512MB  -1
    (parted) set 1 boot on

And now making `filesystems`: 

    $ sudo mkfs.vfat -F 16 /dev/mmcblk0p01
    $ sudo mkfs.ext4 /dev/mmcblk0p2

## 3\. Raspbian

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

Installing `Raspbian` which is a Debian port for Raspberry Pi. Considering you already have debootstrap installed on your Linux. Issue the following commands: 

    $ sudo mkdir /tmp/root
    $ sudo mount -t ext4 /dev/mmcblk0p2 /tmp/root
    $ sudo debootstrap --foreign --arch armhf stable /tmp/root/  http://mirror.nus.edu.sg/raspbian/raspbian/

## 4\. chroot
I use QEMU user emulator to `chroot` to ARM environment to continue installation. In order to use QEMU I installed `qemu-user-static` package in Ubuntu.

    $ sudo apt-get install qemu-user-static
    $ sudo cp /usr/bin/qemu-arm-static /tmp/root/usr/bin
    $ sudo mount -t proc none /tmp/root/proc
    $ sudo mount /dev /tmp/root/dev -o bind
    $ sudo chroot /tmp/root

## 5\. Second stage 
It's time to run `debootstrap` second state. I'm in `chroot` environment from previous step (4). 

    I have no name!@localhost:# ./debootstrap/debootstrap --second-stage

## 6\. Setting up 

I'm going to install some more packages and initial setup for new environement. Setup APT sources and install some packages: 

    I have no name!@localhost:# echo "deb http://archive.raspbian.org/raspbian stable main contrib non-free" | tee -a /etc/apt/sources.list
    I have no name!@localhost:# echo "deb-src http://archive.raspbian.org/raspbian stable main contrib non-free" | tee -a /etc/apt/sources.list
    I have no name!@localhost:# wget http://archive.raspbian.org/raspbian.public.key -O- | apt-key add raspbian.public.key
    I have no name!@localhost:# apt-get update
    I have no name!@localhost:# apt-get install bash-completion locales openssh-server
    I have no name!@localhost:# dpkg-reconfigure locales
    I have no name!@localhost:# service ssh stop

Set root password: 

    I have no name!@localhost:# passwd

Set host name: 

    I have no name!@localhost:# hostname boynux-vpn.localdomain
    I have no name!@localhost:# echo boynux-vpn | tee /etc/hostname
    I have no name!@localhost:# echo "127.0.0.1 boynux-vpn boynux-vpn.localdomain" | tee -a /etc/hosts

Enable eth0 and set to DHCP: 

    I have no name!@localhost:# echo -e "nauto eth0niface eth0 inet dhcp" | tee -a /etc/network/interfaces
    
Configure `fstab`: 

    I have no name!@localhost:# blkid -o export  /dev/mmcblk0p1 2&gt;&1 | grep UUID | sed '/.*/s/$/ /boot vfat defaults,noauto 0 1/;' | tee -a /etc/fstab
    I have no name!@localhost:# blkid -o export  /dev/mmcblk0p2 2&gt;&1 | grep UUID= | sed '/.*/s/$/ / ext4 defaults,noatime 0 1/;' | tee -a /etc/fstab
    
Done. 

    I have no name!@localhost:# exit

## 6\. Installing Kernel & Firmware:

    $ git clone --depth 1 https://github.com/raspberrypi/firmware.git
    $ sudo mount /dev/mmcblk0p1 /tmp/root/boot
    $ sudo cp firmware/boot/* /tmp/root/boot -a
    $ sudo cp firmware/modules /tmp/root/lib/ -a
    $ echo "dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait" | sudo tee /tmp/root/boot/cmdline.txt
    $ sudo umount /tmp/root/dev /tmp/root/proc /tmp/root/boot /tmp/root/</pre>

## 7\. Boot

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

Now it's time to boot into Raspberry Pi. Insert SD Card into Raspberry Pi and plug network and power cables. Wait for boot to complete. 

## 8\. SSH 
Now ssh to our new server :) 
I find IP using `nmap`. 

    $ nmap -sn 192.168.1.0/24
    
Find the correct IP and SSH into server. 

    $ ssh root@192.168.1.xx

Run depmod: 

    # depmod -a

## 9\. VPN 

Now it's time for VPN service setup. 

    # apt-get install pptpdi
    
Open `/etc/pptpd.conf` and add these lines at the end: 

    localip 10.0.0.1
    remoteip 10.0.0.230-254
    
Then edit `/etc/ppp/chap-secrets` and add VPN username/passwords there like:

    test-username     pptpd    my-password    *
    
Then restart pptpd server: 

    # service pptpd restart

## 10\. Masqurading
Allow VPN users to access Internet (optional). 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>


    # echo "net.ipv4.ip_forward=1" | tee -a /etc/sysctl.conf
    # sysctl -p
    # echo post-up iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -d 0/0 -j MASQUERADE | tee -a /etc/network/interfaces
    
That's it. Now try to connect to your box through VPN.

[1]: http://www.boynux.com/wp-content/uploads/2014/04/Raspi_Colour_R.png
