---
layout: post
title: Docker on Jessie
excerpt: Installing Docker on Debian Jessie with Configurable Docker Options
tags:
  - SystemD
  - Docker
  - Linux
category: DevOps
---

If you have ever tried recently to install Docker on classic a Linux system you might realized that **Docker Options** which you used to configure in `/etc/default/docker` is not working anymore. That's because the latest Docker install removes that possibility although it installs `/etc/default/docker` but surprisingly it has no effects.

But fortunately the solution is very simple. By overriding `systemd` unit file for Docker. This short note explains how I did this on my development machines:

1. To install Docker binaries:

        $ curl https://get.docker.com | sh

2. Add current user to Docker Group  (optional)

    *By adding your user to Docker group you no longer need to use Docker  CLI with `sudo` command.*

        $ sudo usermod -aG docker $USER

3. Make sure `/etc/default/docker` exists:

        $ touch /etc/default/docker

4. Override `systemd` unit:

        $ sudo mkdir /etc/systemd/system/docker.service.d

    Now create `override.conf` file in that path and copy the following lines:

        [Service]
        EnvironmentFile=/etc/default/docker
        ExecStart=
        ExecStart=/usr/bin/docker daemon -H fd:// ${DOCEKR_OPTS}

5. Reload systemd daemon:

        $ sudo systemctl daemon-reload

6. And finally restart Docker service:

        $ sudo systemctl restart docker.service


Now you can add your new options to `DOCKER_OPTS` in `/etc/default/docker` file and restart Docker daemon.

