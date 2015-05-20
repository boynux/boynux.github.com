---
layout: post
title: Ship in Docker, build in Docker
excerpt: The most important parameter to a successful build and test before deployment is to do it all in a Production-Like environment. Containers are not an exception! Yet with Docker is pretty easy to do so.
tags: 
  - Docker
  - Continues Delivery
  - Build
category: DevOps
---

### Idea

<div style="float: right">
<img src="{{ site.url }}/img/docker-logo.png" width="161" alt="Load Balancing in Docker" title="Load Balancing in Docker" />
</div>

In past few years Linux containers has extremely changed the way we deploy applications. With arising of Docker and its simple way of handling containers the new concept of *Container as a service* has emerged.
If you ship your apps in containers, or its in your road map, make sure your build and test environment is as identical as production environment. This is a key to a successful delivery.

### Docker volumes

I tried different ways to achieve this goal, yet the best way I found is to use *volumes* to mount source code inside docker container, then build the project and test it. At the end simply clean up the mess and delete containers.

I don't recommend Docker files and building image to do that except in very rare cases. The reasoning for this is fairly simple:

1. When you build, you create garbage.
2. When you test you need a to get a test result out.
3. Images are big and carrying over to next stage in pipeline is difficult.

These are the few reasons that I figured out to not to use Docker images but Docker containers to do the build and test stages.

### Be careful

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

Just be aware of one issue. Docker containers normally run with root privileges. Even if you change user inside the container it still has a root privilege from host point of view.

As a consequence any command (eg. build) that produces an artifact will have a root permission, then your build artifact will have a root permission too.

Fortunately there is simple way to solve this issue. Run Docker container as a non-root user, unless you have a very valid reason not to do so.

By passing `--user` to Docker run command you can change user to any existing user in your host. There is no need to have same user ID at container level.

### How to do it

In you pipeline steps are quite like this:

1. Pull source code.
2. Pull Docker base image with build tools included. *
3. Run Docker container with source path mounted as a volume.
4. Build the code.
5. Clean up.

Build example: 

    $ git clone my-fantastic-repo.git
    $ cd  my-fantastic-repo
    $ docker run --rm=true -t --user $(id -u) -v $(pwd):/src -w /src golang go build -v

Unit test:

    $ docker run --rm=true -t --user $(id -u) -v $(pwd):/src -w /src golang go test

And for acceptance test you can similarly run you app inside a temporary container and by linking that container to an acceptance test container run the test against it.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

That's all, simple and effective. If everything goes well then you can simply create an image from that source code and just add whatever you'll need to add. Lean!

The same approach should be taken for other language like PHP or Ruby, with an exception that you run package manager instead of actual build inside container. See [How to use Rake in build pipeline automation][1]

### Bonus

By doing so, you'll hit two birds with one stone:

1. Production-like environment for build and test
2. Your build agent/dev box do not need to have build tools except Docker!

[1]: {{ site.url }}/rake-pipeline-automation/
