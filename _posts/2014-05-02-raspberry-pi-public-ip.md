---
layout: post
title: What is my Raspberry Pi public IP?
excerpt: Raspberry Pi sends public IP adress behind NAT via Email. If you have problem accessing Raspberry Pi public IP address because it changes dymanically here is what you need.
---

This is a continue to my project [Raspberry Pi VPN][2] server. In that article I explained how to create a VPN server with your Raspberry Pi. I did that and enabled my home router port forwarding for ports 22 (`SSH`) and 1723 (`PPTP`). It works perfectly but there is one small problem! Not very small though!

[<img title="Raspberry Pi Public IP" src="http://www.boynux.com/wp-content/uploads/2014/04/Raspi_Colour_R-248x300.png" alt="Raspberry Pi" class="alignright wp-image-900 size-medium" height="300" width="248" />][1]

## Introduction

Everything is working great when I know my home route public IP address, but this address is dynamically allocated by my ISP. What happens if IP changes? Of course I loose my access to VPN server. There are a few solutions to this:

*   Requesting ISP for fixed (static) public address
*   Setup a dynamic DNS with one of existing service providers.
*   Write a simple `bash` script to send IP address when it changed.

<div class="ad float-right" style="float: right;">
  <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Display Large Rectangle -->
  
  <ins class="adsbygoogle"
     style="display:inline-block;width:336px;height:280px"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="7819924448"></ins> <script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

Well, among those three options that came to mind quickly, I chose the third one. Because requesting for static public IP address might not be that easy and your ISP might charge you extra for that service. And I don't want to waste my time subscribing to one on those free dynamic DNS providers. And finally I like to try something new.

If you are interested to do the same, follow this instruction. It won't be that difficult. I consider you've already installed `Rasbian` on your Raspberry Pi, though it works with other distros as well. If not follow my other article about [Raspberry Pi VPN][2] server.

## 1\. Install required packages

I'm going to use one of free `SMTP` email provider, ie. Google Mail. You may choose any other, just need to change email server and authentication information accordingly that comes later in the configuration file. In order to use external email provider I use `sendEmail` command line tool, which is written in Perl and available in Rasbian repositories. Because Gmail uses `TLS` authentication I need to install Perl TSL support using CPan. And finally we need `curl` to fetch public IP address.

<div class="ad">
  <script async="" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Responsive Display -->
  
  <ins class="adsbygoogle" style="display:block"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="4587256441"
     data-ad-format="auto"></ins> <script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

    sudo apt-get install build-essential libssl-dev sendemail curl
    sudo cpan install IO::Socket::SSL
    sudo cpan install Net::SSLeay

## 2\. Email account

Create a new Gmail (or any other provider) email for your Raspberry Pi. I'm not going to explain how to create email account :)

## 3\. Bash script

<div class="ad" style="float: right;">
  <script async="" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Display Rect Medium -->
  
  <ins class="adsbygoogle" style="display:inline-block;width:300px;height:250px"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="7261521241"></ins> <script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

And here is a simple `bash` script that find public IP address and compare that with the latest IP address, if it's different, an email with new IP will be send out. This script uses an external web site (`ifconfig.me`) to find public IP address because my Raspberry Pi is behind NAT.

This is configuration file. Copy and paste this text in any editor change email and password to proper values and then paste into terminal. this will create a file in `/etc/sendip.conf` . You can edit this file later.

    cat >>'CONFIG' | sudo tee /etc/sendip.conf
    # sendip config file

    # SMTP Server address with port, default port is 25
    SMTP_HOST="smtp.gmail.com:587"

    # new IP Email subject 
    EMAIL_SUBJECT="My new IP address"

    # Raspberry Pi Email address, email will be send out using
    # this email address
    RASPBERRY_EMAIL="raspberry_email@gmail.com"

    # Password for above email address
    RASPBERRY_PASSWORD="raspberry_email_password"

    EMAIL_MESSAGE='
    Hello,

    My new IP address is $NEWIP.

    Regards,
    Raspberry Pi

    '
    CONFIG

And here is the script. Just copy and paste the whole thing in bash. This will create `/usr/bin/sendip`.

<div class="ad">
  <script async="" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Responsive Display -->
  
  <ins class="adsbygoogle" style="display:block"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="4587256441"
     data-ad-format="auto"></ins> <script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

    cat >>'CODE' | sudo tee /usr/bin/sendip
    #!/bin/bash

    source /etc/sendip.conf

    if [[ -z $1 ]]
    then
        echo "
    Usage: $0 &lt;email address&gt; 
    "
        exit 1
    fi

    EMAIL_ADDRESS=$1

    expand_message ()
    {
        echo "$1" | while read LINE
        do
            echo "$(eval echo $LINE)"
        done
    }

    if [[ -f "/tmp/.ipaddress" ]]
    then
        OLDIP=$(cat /tmp/.ipaddress)
    fi

    NEWIP=$(curl -s http://ifconfig.me/ip)

    if [[ "$OLDIP" != "$NEWIP" ]]
    then
        echo $NEWIP | tee /tmp/.ipaddress

        MESSAGE=$(expand_message "$EMAIL_MESSAGE")

        sendEmail -f $RASPBERRY_EMAIL -t $EMAIL_ADDRESS -s $SMTP_HOST -u $EMAIL_SUBJECT -xu $RASPBERRY_EMAIL -xp $RASPBERRY_PASSWORD -m "$MESSAGE" -o tls=yes
    fi

    CODE

Make script executable:

    sudo chmod u+x /usr/bin/sendip

## 4\. Running script 

To run script enter `sendip` command followed by your email addresss. The IP address will be fetched and an email will be send out to the provided email address. 

    sendip my_email@domain.com

If I run script for second time and my IP is the same as before, email won't send again. So script does not spam you.

## 5\. Automation 

To use this script effectively we need to make the process automatic. Then whenever IP changes this script will send IP to my email. In order to do this I use a simple hourly `crontab` script. Copy and paste the following code into bash to create one. 

    cat >>CRON | sudo tee /etc/cron.hourly/sendip 
    #!/bin/bash
    /usr/bin/sendip my_email@domain.com 2&gt;&1 &gt;&gt; /var/log/sendip.log

    CRON

And make it executable: 

    sudo chmod u+x /etc/cron.hourly/sendip

<div class="ad">
  <script async="" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Responsive Display -->
  
  <ins class="adsbygoogle" style="display:block"
     data-ad-client="ca-pub-7360583392867579"
     data-ad-slot="4587256441"
     data-ad-format="auto"></ins> <script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

  
That's all. Now you everything settled and should work properly. Please leave comments and share if you found this article useful.

[1]: http://www.boynux.com/wp-content/uploads/2014/04/Raspi_Colour_R.png
[2]: http://www.boynux.com/raspberry-pi-vpn-server/ "Raspberry Pi VPN Server"
