---
layout: post
title: How to consolidate Docker logs in Logstash
excerpt: While using docker is extremely easy, when talking about production dealing with application logs inside Docker is a real pain. There are couple of ways to solve this issue.
tags: DevOps Docker Container Logstash Systemd CoreOs Syslog
category: Devops
---

### Introduction

<span style="float:right">
<img src="{{ site.url }}/img/logstash.png" width="161" alt="How to pack Logstash forwarder in Docker containers" title="How to pack Logstash forwarder in Docker containers" />
<br />
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="How to pack Logstash forwarder in Docker containers" title="How to pack Logstash forwarder in Docker containers" />
</span>

Using Docker eases deployment pains and every sysadmin and devops I know - including myself - enjoys working with containers. But when it comes to production, we all love to consolidate logs from all over our infrastructure into a single accessible location. The first obvious choice for such a stack is [Elastc][1] + [Logstash][2] + [Kibana][3] (ELK).

But when dealing with docker it's a big pain. There are numerous ways developed to solve this issue, and recently Docker by releasing [version 1.6][4] introduced *Logging Driver*. Which solves at least some of these problems.

Here I'm going to review a few common ways to consolidate Docker logs using [Logstash Forwarder][5].

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

### Possible ways

I just list a couple of ways we can send logs to ELK stack using Logstash forwarder, these are popular ones:

* Mounted volumes
* Pipe-in apps std* to remote Syslog
* Syslog enabled app + Mounted volume
* Docker Syslog driver
* Journald facility in [CoreOs][7]
* [Logspout][6]
* Commercial solutions

There are other methods too, but I guess they may fall into one of the above categories. Now let's find out what is pros and cons with each solution.

#### Mounted volumes

Obviously it's the easiest and most common! If you use Google you'll find out this is one of most recommended ways, as it's also mentioned in a [blog post][8] by Docker themselves.

How it works, in your app, instead of writing log messages to std* you write them into a file. Then you mount a volume when running containers with `-v` to the path in which your application is writing logs.

Let's imagine, your app writes logs into `/foo/app.log`. Docker command may look like this:

    docker run -t -v /var/log/app:/foo my-app-image

Ok, now [Logstash Forwarder][5] can be configured to harvest those files in host file system, by mounting that volume (read-only) to forwarder container as well. 

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

Let's see what's benefits of this approach:

* Easy to deploy
* Not depending on host services

What are the cons?

* Complexity of managing log files within application.
* It violates 12-Factor app [rule of logging][9].
* Forwarder configuration depends on apps file names.

Among all "manging log files by application itself" is the worst. Because application doesn't need to handle that complexities, specially in multi-threaded applications. But if your app is currently storing logs inside files. Shouldn't be a bug deal.

Note: I've seen around the net some suggestion about writing application logs to stdout and then piping (or redirecting) them to a file. I don't recommend that for a very simple reason. If the application your piping to (or redirecting) dies, your application will get SIGPIP and usually dies. Or at least blocks until there's another process to read from the pipe. Try to avoid that.

#### Pipe-in apps std* to remote Syslog

This means using something like: 

    ./app-run.sh 2>&1 | nc -u syslog-server 514

Pros: 

* Easy to deploy
* Application simply writes logs to std*
* No extra volume or dependency on files.
* Forwarder configuration is standard in all hosts

Cons:

* Piping is a problem (read the note above)
* Depends on host Syslog server*

*You may consider sending Logs directly to Logstash Forwarder Syslog listener, this way  you don't even need host Syslog! But as a result if forwarder dies you'll loose all new logs.

#### Syslog enabled app + Mounted volume

In this scenario, since your app is logging directly to syslog socket, the only thing you have to do is to mount `/dev/log` to your container.

    docker run -v /dev/log:/dev/log my-app 

Then you can configure host Syslog server to write those logs into specific files like `/var/log/user.log` or simply in `/var/log/syslog` and forwarder can monitor that files. Since writing that files is handled by Syslog itself, you don't need to be worried about multi-thread apps.
Pros:

* Easy to deploy
* No need to do anything to your application.
* It's persistent and safe.

Cons:

* Depends on the host's Syslog
* It violates 12-Factor app [rule of logging][9].

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

#### Docker Syslog driver

This one is pretty cool, but you may not be able to upgrade to latest version soon, if you're depending on standard package repositories. If that's fine for you to upgrade to latest version, then should be fine.

The idea is Docker writes all logs from apps (std*) to host Syslog. Then you just harvest `/var/log/syslog` file and you're good to go. The only thing you've to do is to run Docker instance with Syslog feature enabled:

    docker run --log-driver=syslog -t my-app

Pros:

* Easy deployment
* No need to change application
* It's very safe and persistent

Cons: 

* Depends on the host's Syslog

#### Journald facility in [CoreOs][7]

This one is very specific to CoreOs. CoreOs is actually doing the hard job for you. It collects all Docker logs for you and send them to Journald. And it'll be accessible via `journalctl` command.

You don't need to do anything with app or containers. Only thing that has to be done is forwarding logs to a file:

    journalctl -f --json | tee -a /var/log/systemd

Now forwarder an harvest `/var/log/syslog` file. 

Pros:

* No need to change application
* No need special config for forwarder (no depends on app)
* Fairly safe.

Cons: 

* Depends on host (journald)

Why *fairly safe* because if the journalctl pipe dies or gets restarted you might get some duplicate logs. There are some ways to lessen that effect. But it's better to have two of a little bit logs rather than having no log at all ;)

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

#### [Logspout][6]

What's that? A small docker container that reads raw logs from Docker socket `/var/run/docker.sock` and send it to syslog server. Well I don't thing it's good option because of two problems: 

1. If Logspout container starts after app's container it can not detect container and sends logs to syslog.
2. If for any reason Docker endpoint or logs format changes in future, this solution may break.

Pros: 

* No need to change application's behavior

Cons:

* Complex to deploy (extra dependency)
* Depends on Docker internals ans API
* Not safe and consistent

#### Conclution

All methods that explained here are very popular and whether it fits a particular project varies from a project to another. I put together this post here so that when I decide to choose an approach I know about all its consequences, and I hope it;s useful for you too.

Of course all this information will change during the time and one method might become more preferable to another. I'll try to keep this post updated as much as I can.

That's pretty much all. If you know any other solution that I've missed here, I'd like to know about it. Or if you're using any of these methods and having a good experiences with that please leave your comments below.


[1]: https://www.elastic.co/
[2]: http://logstash.net/
[3]: https://www.elastic.co/products/kibana
[4]: https://blog.docker.com/2015/04/docker-release-1-6/
[5]: {{ site.url }}/logstash-forwarder-docekr/
[6]: https://github.com/gliderlabs/logspout
[7]: https://coreos.com/
[8]: https://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/
[9]: http://12factor.net/logs
