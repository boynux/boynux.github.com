---
layout: post
title: How to pack Logstash forwarder in Docker containers
excerpt: Creating Logstash forwarder (aka. Lumberjack) in Docker may look pretty easy and straight forward. But what if container dies and you want to start a new one! Something strange is going to happen...
tags: Docker Logstash Lumberjack Container
category: DevOps
---

### Problem

<span style="float:right">
<img src="{{ site.url }}/img/logstash.png" width="161" alt="How to pack Logstash forwarder in Docker containers" title="How to pack Logstash forwarder in Docker containers" />
<br />
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="How to pack Logstash forwarder in Docker containers" title="How to pack Logstash forwarder in Docker containers" />
</span>

There are plenty of articles out there about how to create Logstash forwarder Docker image. For example [here][1], [here][2] and [here][3] or use Google to find even more.

So why another one? The answer is very simple. I tried them all and there is a big problem with all of them! If Docker container gets recreated for any reason (system restart, image update or ...), all log evens in monitored files will resend again to Logstash server.

### Idea

By investigating a lot and reading [Logstash forwarder source code][4], finally I found the missing piece of this puzzle. A quick look into the source code shows us [here][5] there is a special file with hardcoded name as `.logtsash-forwarder` which is being created in the program's working directory, and actually stores all the information about last harvested points in monitored log files.

What happens is, when Docker container gets destroyed and recreated again these information is lost and forwarder has no clue where to start harvesting again. So what is does? Starts from the beginning.

### Solution

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

Simply store that file in host file system by mounting is as a volume with Docker `-v` option. But there is another problem! If you look at [here][6] in source code again, you'll notice that it calls `os.Rename`. What's wrong with that? It turns out if you only mount this file into Docker image, this call fails. Because in Docker it's not possible to move files from Docker filesystem into mounted filesystem due to system call restrictions.

### Solution

Obviously the solution is to mount a directory instead of file itself. And here is a simple Docker file that does this:

    FROM marvambass/oracle-java8

    RUN apt-key adv --keyserver pool.sks-keyservers.net --recv-keys 46095ACC8548582C1A2699A9D27D666CD88E42B4 \
        && echo 'deb http://packages.elasticsearch.org/logstashforwarder/debian stable main' | tee /etc/apt/sources.list.d/logstashforwarder.list \
        && apt-get update; apt-get install -y logstash-forwarder

    ENV PATH /opt/logstash-forwarder/bin:$PATH

    COPY entrypoint.sh /

    VOLUME ["/logstash-forwarder-conf", "/certs", /home/logstash]

    WORKDIR /home/logstash

    ENTRYPOINT ["/entrypoint.sh"]
    CMD ["logstash-forwarder", "-config=/etc/logstash-forwarder.conf"]

There are two important points here, in `VOLUME` you can see `/logstash` which is exposed and `WORKDIR` which is pointing to `/logstash`. Here is a complete code in [github][7]. This is actually fork of another [forwarder project][3] with these modifications.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

### How to use it

If you want to persist positions of harvested logs in files simply mount `/logstash` to host filesystem.

    docker run -v /var/cache/logstash:/home/logstash docker-logstash-forwarder ...

And next time you update image or start a new container for forwarder it'll start harvesting logs from the point it's left them.

**Update: DOCKERFILE has been updated. See [GITHUB][7] commit for more info.

### [Continue on part II][8]


[1]: https://denibertovic.com/post/docker-and-logstash-smarter-log-management-for-your-containers/
[2]: https://github.com/million12/docker-logstash-forwarder
[3]: https://gowalker.org/github.com/digital-wonderland/docker-logstash-forwarder
[4]: https://github.com/elastic/logstash-forwarder
[5]: https://github.com/elastic/logstash-forwarder/blob/master/registrar.go#L31
[6]: https://github.com/elastic/logstash-forwarder/blob/4b6c987646bdc199eabf9b8f2f5ad57ff860b28e/registrar_other.go#L10
[7]: https://github.com/boynux/docker-logstash-forwarder
[8]: {{ site.url }}/logstash-forwader-docker-part-2/

