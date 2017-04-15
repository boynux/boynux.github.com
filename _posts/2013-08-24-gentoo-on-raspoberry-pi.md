---
layout: post
permalink: /45
title: Gentoo on Raspberry Pi
excerpt: How to install Gentoo on Raspberry Pi without Keyboard, mouse and display attached to your Raspberry-Pi. Step-By-Step guide to do that!
---

[<img align='right' class="alignnone" alt="" src="{{ site.baseurl }}/images/474px-Raspberry_Pi_Logo.svg.png" width="128" height="162" />][1]
[<img align='right' class="alignnone" alt="" src="{{ site.baseurl }}/images/573px-Gentoo_Linux_logo_matte.svg.png" width="138" height="144" />][2]

How to install Gentoo on Raspberry Pi without Keyboard, mouse and display attached to your Raspberry-Pi. 

I decided to install a new OS on my new Raspberry Pi, then I realized that I don't have a USB Keyboard and HDMI / Composite cable! So I decided to try installing OS directly into SD card using my laptop and then booting directly into my Raspberry Pi.  And it just worked amazingly at the first shot. Why Gentoo? Because I've lots of experience with installing Gentoo servers over remote SSH, and I just feel very comfortable with that. 

## Here is, step by step, how to install Gentoo on Raspberry Pi:

### 1- Requirements:

*   Ubuntu (I've 13.04 installed on my laptop, but other Linux distros should work)
*   Raspberry-Pi  (I've got Model B with 512MB RAM + LAN)
*   Micro-USB cable (or just your cellphone charger)
*   LAN Cable (for network connection after installation)
*   Min. 4 GB SD Card (better to use class-10, but mine is class-4)
*   Card-Reader to attach SD card to your laptop (my laptop has it built-in)
*   Minimum 1 hour free time :)

<img class="aligncenter" alt="" src="{{ site.baseurl }}/images/800px-RaspberryPi.jpg" width="360" height="240" />


### 2- Preparing SD card


<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

Well, this is fairly easy, you need to create partitions on your SD card. First attach your SD card to your laptop/PC and use any partition editor software you prefer. I prefer parted so I explain it here: You need at least these 3 partitions: 

	Name             Size              FS Type           Type
	boot             Min. 32 MB        FAT-16            Primary
	swap             1024 MB           SWAP              Primary
	root             All remaining     Ext-2/3/4         Primary

After creating all these partitions the final partition table should look like this: 

	$ sudo parted /dev/sdb
	....
	(parted) mklabel msdos
	(parted) mkpart primary ext2 1 32 
	(parted) mkpart primary ext2 33 1057
	(parted) mkpart primary ext2 1058 3951
	(parted) set 1 boot on
	(parted) print
	....
	Number  Start   End     Size    Type     File system     Flags
	 1      1049kB  32.5MB  31.5MB  primary  fat16           boot, lba
	 2      32.5MB  1057MB  1024MB  primary  linux-swap(v1)
	 3      1058MB  3951MB  2893MB  primary  ext

Now you need to make proper file systems on SD card: 

	$ sudo mkfs.vfat -F 16 /dev/sdb1 
	$ sudo mkfs.ext4 -N 803200 /dev/sdb3
	$ sudo mkswap /dev/sdb2

*Note: for ext4, if you have 8GB SD card you don't need to use `-N 803200`, this is only for smaller size partitions to have enough inodes available (Gentoo portage will fill-up inodes tables quickly)*

On my box, SD card is probed as `/dev/sdb` you may need to use different drive name, you can use `dmesg | tail` to find yours.

Now we are good to go! 

### 3- Installing base OS: 

You need to download Gentoo stage-3 and portage files from mirrors, just go to Gentoo web site and look for mirrors. select your closest mirror and follow these paths:

* `/releases/arm/autobuilds/current-stage3-armv6j_hardfp/stage3-armv6j_hardfp-*.tar.bz2` 
* `/releases/snapshots/current/portage-latest.tar.bz2` 

Go to `/tmp` path and download those two files, I prefer wget to download but you can use whatever you want: 

	$ cd /tmp
	$ wget http://gentoo.aditsu.net:8000/releases/arm/autobuilds/current-stage3-armv6j_hardfp/stage3-armv6j_hardfp-20130816.tar.bz2
	$ wget http://gentoo.aditsu.net:8000/snapshots/portage-latest.tar.bz2

you need to extract those files into your SD card root and usr directories respectively, so first we need to mount SD card `root` & `boot` partitions: 

	$ sudo mkdir /tmp/raspberry
	$ sudo mount /dev/sdb3 /tmp/raspberry
	$ sudo mkdir /tmp/raspberry/boot
	$ sudo mount /dev/sdb1 /tmp/raspberry/boot</pre> And extract files in SD card: 

	$ cd /tmp
	$ sudo tar xjf stage3-armv6j_hardfp-20130816.tar.bz2 -C /tmp/raspberry
	$ sudo tar xjf portage-latest.tar.bz2 -C /tmp/raspberry/usr 

Download Raspberry-Pi firmware and Linux kernel, we can cross-compile kernel but it's out of scope of this post, maybe in another post I'll go thought that as well. Well, all these files are available on GitHub, here: 

<https://github.com/raspberrypi/firmware/> 

	$ cd /tmp
	$ git clone --depth 1 git://github.com/raspberrypi/firmware/
	$ sudo cp firmware/boot/* /tmp/raspberry/boot/
	$ sudo cp -r firmware/modules /tmp/raspberry/lib/

Now you have installed Gentoo on Raspberry Pi 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

### 3- Basic setup: 

Now we need to do some basic setup in order to be able to boot Raspberry-Pi, This is mainly `/etc/fstab` settings and kernel parameters. If your partition setup is same as what I proposed your can create your fstab as follows. First we need to get `UUID` of partitions: 

	$ sudo blkid

	/dev/sdb1: SEC_TYPE="msdos" UUID="54D6-304C" TYPE="vfat"
	/dev/sdb2: UUID="af11b045-7992-44e2-81bf-04a09ba3959f" TYPE="swap"
	/dev/sdb3: UUID="926ae355-30cf-43fb-a443-7834d6b2c566" TYPE="ext4"

we will use `UUID`s in our `fstab`, now edit `/tmp/raspberry/etc/fstab`, remove or comment all lines (using #) and enter the following lines instead (please replace `UUID` from output of previous command accordingly): 

	$ sudo nano /tmp/raspberry/etc/fstab</pre>

	UUID=54D6-304C                            /boot   vfat  noauto  1 2
	UUID=af11b045-7992-44e2-81bf-04a09ba3959f none    swap  sw      0 0
	UUID=926ae355-30cf-43fb-a443-7834d6b2c566 /       ext4  noatime 0 1

And create a file called `cmdline.txt` in boot folder and paste the following line in that, these are kernel options: 

	$ sudo nano /tmp/raspberry/boot/cmdline.txt

	dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p3 rootfstype=ext4 elevator=deadline rootwait 

The most important thing in `cmdline.txt` is **root path**, if your partition schema is like what I proposed that should be OK, otherwise change it accordingly. 

### 4- chroot to new environment: 

Well, If you have a HDMI cable and Keyboard you can simply insert SD into you RPi and turn it on, you'll get login prompt and good to go, but do you remember? I don't have keyboard and HDMI cable. That's why we have step 4. 

Actually the trick is here, because RPi is ARM based and my laptop is AMD64 I can't simply chroot to new environment, because binaries are not compatible with my laptop architecture, to address this issue I'm going to use qemu user-land emulation. 

First, you need to install qemu user-mode emulator binary with static libraries build, in Ubuntu it simply could be installed from standard repositories: 

	$ sudo apt-get install qemu-user-static 

Now in order to chroot to new environment with emulation we need to copy qemu binary file into SD card: 

	$ sudo cp `which qemu-arm-static` /tmp/raspberry/usr/bin/

Mount required paths: 

	$ sudo mount -o bind /dev /tmp/raspberry/dev
	$ sudo mount -t sysfs none /tmp/raspberry/sys
	$ sudo mount -t proc none /tmp/raspberry/proc

	$ sudo chroot /tmp/raspberry /usr/bin/qemu-arm-static /bin/bash

You should see your nice Gentoo prompt now: 

	localhost / #

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="recangle"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

### 5- Setup new environment: 

We are almost there :) 

If you issue `uname -ra` you'll get something like this (note architecture type): 

	Linux my-thinkpad 2.6.32 #42-Ubuntu SMP Tue Aug 13 19:40:39 UTC 2013 armv7l GNU/Linux 

Now we need to do some initial setup, issue the following commands in your Gentoo: 

	# source /etc/profile
	# update-env

Set your correct time zone: 

	# rm /etc/localtime
	# ln -s /usr/share/timezone/Asia/kuala_Lumpur /etc/localtime

Set password: 

	# passwd

Setup `eth0` auto start: 

	# ln -s /etc/init.d/net.lo /etc/init.d/net.eh0
	# rc-update add net.eth0 boot

Enable software clock, because RPi does not have hardware clock: 

	# rc-update del hwclock boot
	# rc-update add swclock boot 

Enable SSH server: 

	# rc-update add sshd default 

Select proper profile: This command lists available profiles choose `armv6j`, in my case it's no 21: 

	# eselect profile list
	# eselect profile set 21

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

Now you can use emerge to install any extra packages, I'll leave this to you because requirements would be different. You may continue to step 6 without installing any extra packages. 

### 6- Boot Paspberry-Pi: 

Yes, we are done! We just need to unmount SD card and boot our RPi with Gentoo: 

	# exit
	$ sudo umount /tmp/raspberry/dev
	$ sudo umount /tmp/raspberry/proc
	$ sudo umount /tmp/raspberry/sys
	$ sudo umount /tmp/raspberry/boot
	$ sudo umount /tmp/raspberry/

Remove your SD card from laptop/PC and insert it into RPi. If you use Ubuntu, like me, you can setup LAN Internet sharing in order to access RPi: *NetworkManager -> Edit connections... Choose Auto eth0 -> Edit*. In IPv4 settings choose `Shared to other computers` option. 

Alternatively you can install and setup `DHCP` server and proper `iptables` for masquerading or simply use your internet gateway switch if you have. 

Now connect RPi Lan to your laptop/PC LAN, then connect RPi mini-USB to power. Wait until RPi boots up, this may take a while. 

**Finding IP:** 

I use nmap to find ip: 

	$ ifconfig eth0 | grep "inet addr"
		inet addr:<strong>10.42.0.1</strong>  Bcast:10.42.0.255  Mask:255.255.255.0

	$ namp -sn 10.42.0.0/24

	Starting Nmap 6.00 ( http://nmap.org ) at 2013-08-24 10:42 MYT
	Nmap scan report for 10.42.0.1
	Host is up (0.00063s latency).
	Nmap scan report for 10.42.0.87
	Host is up (0.00066s latency).
	Nmap done: 256 IP addresses (2 hosts up) scanned in 3.25 seconds 

As you can see my RPi address is 10.42.0.87, now SSH to RPi: 

	$ ssh root@10.42.0.87

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

<img class="size-medium wp-image-245 aligncenter" alt="Screenshot from 2013-08-24 22:59:35" src="{{ site.baseurl }}/images/Screenshot-from-2013-08-24-225935-300x191.png" width="300" height="191" />

Happy Rasperry-Pi-ing :)

References & useful links: 

*   <a href="http://www.amazon.com/s/?_encoding=UTF8&camp=1789&creative=390957&linkCode=ur2&pageMinusResults=1&suo=1392096656752&tag=boynux-20&url=search-alias%3Daps#/ref=nb_sb_noss_1?url=search-alias%3Daps&field-keywords=raspberry%20pi&sprefix=raspberr%2Caps&rh=i%3Aaps%2Ck%3Araspberry%20pi&sepatfbtf=true&tc=1392096659229" target="_blank">Raspberry Pi on Amazon</a>
*   [http://wiki.gentoo.org/wiki/Raspberry_Pi][2]
*   <http://wiki.gentoo.org/wiki/Raspberry_Pi_Quick_Install_Guide>
*   <https://github.com/raspberrypi/firmware>
*   <http://elinux.org/RPiconfig>
*   <http://elinux.org/R-Pi_Troubleshooting>
*   <http://www.raspberrypi.org>

<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>

[1]: http://wiki.gentoo.org/wiki/Raspberry_Pi
[2]: http://www.raspberrypi.org
