---
layout: post
title: How to use Logstash forwarder in Docker containers (part II)
excerpt: A tiny how-to to use Logstash-Forwarder docker image right from Docker HUB. With configuration and basic setup.
tags: Docker Logstash Lumberjack Container DockerHub
category: DevOps
---

Introduction
============

<span style="float:right">
<img src="{{ site.url }}/img/logstash.png" width="161" alt="How to pack Logstash forwarder in Docker containers" title="How to pack Logstash forwarder in Docker containers" />
<br />
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="How to pack Logstash forwarder in Docker containers" title="How to pack Logstash forwarder in Docker containers" />
</span>

Last week I wrote a [post][1] on how to create a Logstash Forwarder Docker image. But the exact explanation on how to use that was missing. So I decided to write another post to explain it in more details.

So, what I did was I created a Docker image from the same Docker file which is provided in previous post and also in my [Github][2] account. Now I'm going to use the same image to show you how you can setup and run your own forwarder. You can find the image in [Docker Hub][5]

Prerequisites
==============

I assume that you already have setup a Logstash server. And you have access to its appropriate *certificate*. If you need more information on how to generate OpenSSL certs read [this][3]

Another important thing to consider is that you need to have to proper DNS record (or in /etc/hosts) to match that certificate for your Lostash server. Otherwise forwarder can't establish a SSL connection.

And finally you need a forwarder configuration which should look like this:

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />


    {
      # The network section covers network configuration :)
      "network": {
        # Down stream logstash server. You can change this to a fixed server
        # or you can leave it as it is and provide server address with 
        # env vars to Docker run command. (this defaults to: logstash:5000)
        "servers": [ "LOGSTASH_SERVER" ],

        # The path to your trusted ssl CA file. 
        # You shouldn't change this path. Unless you change the 
        # actual filename.
        "ssl ca": "/certs/logstash-forwarder.crt",

        # Network timeout in seconds. This is most important for
        # logstash-forwarder determining whether to stop waiting for an
        # acknowledgement from the downstream server. If an timeout is reached,
        # logstash-forwarder will assume the connection or server is bad and
        # will connect to a server chosen at random from the servers list.
        "timeout": 15
      },

      # The list of files configurations
      # You should change this part to match your needs.
      "files": [
        # An array of hashes. Each hash tells what paths to watch and
        # what fields to annotate on events from those paths.
        {
          "paths": [
            # single paths are fine
            "/var/log/messages",
            # globs are fine too, they will be periodically evaluated
            # to see if any new files match the wildcard.
            "/var/log/*.log"
          ],

          # A dictionary of fields to annotate on each event.
          "fields": { "type": "syslog" }
        }, {
          # A path of "-" means stdin.
          "paths": [ "-" ],
          "fields": { "type": "stdin" }
        }, {
          "paths": [
            "/var/log/apache/httpd-*.log"
          ],
          "fields": { "type": "apache" }
        }
      ]
    }


This config is almost same copy of Logstash Forwarder sample. You can find it [here][4]

A few notes: You shouldn't change certificate file path since it should be read from within docker container. And you'll provide it when you mount a volume to your container. 

Other thing is server address, you can provide $LOGSTASH_SERVER env var to Docker run but in that case you have to keep the server address as what I added in sample config.

And finally you have to save all those files (config + cert) in a directory structure like this. Hmmm, actually it's not that important but if you have different directory structure then you have to change Docker run command to match yours.

    ./certs/logstash-forwarder.crt
    ./config/logstash-forwarder.conf

Docker run
==========

Let's see how our Docker run command looks like:

    docker run -t \
        -v $PWD/certs:/certs:ro \
        -v $PWD/config:/logstash-forwarder-conf:ro \ 
        -v /var/cache/logstash-forwarder:/home/logstash \ 
        -v /var/log:/var/log \
        -e  LOGSTASH_SERVER=whatever-server:5000 \
         boynux/logstash-forwarder


Then just follow Docker stdout and make sure that it's started properly. If anything goes wrong you can try to figure out the problem by reading logstash output log.

 In this command I also mounted `/var/log` from host to Docker instance, so it'll harvest `/var/log/messages` and `/var/log/*.log` files from host machines, you may need to change it too.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

If everything goes well, you'll see a message saying that it has connected to server and ready to send logs. Since in config file there is also `stdin` configured as an input to forwarder you could run docker instance with `-i` option and whatever you type in goes directly to Logstash server.

Did I missed anything? Please let know by leaving your comments below.

[1]: {{ site.url }}/logstash-forwarder-docker/
[2]: https://github.com/boynux/docker-logstash-forwarder
[3]: https://www.openssl.org/docs/HOWTO/certificates.txt
[4]: https://github.com/elastic/logstash-forwarder/blob/master/logstash-forwarder.conf.example
[5]: https://registry.hub.docker.com/u/boynux/logstash-forwarder/
