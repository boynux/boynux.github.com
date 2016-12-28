---
layout: post
title: Modern Microservices with Chromosome Configuration
tags: DevOps Agile Microservices
category: Ideas
published: false
---

As software is growing into Microservices and Nanoservices more and more the complexities of software architecture, configuration management and maintenances grow exponentially. Once a piece of software which was consisted of a single app with a single configturation and now splatted into tens or hundreds of tiny standalone self-contained pieces of applications. And having the mindset and experience of monolith apps naturally tells us to split the configurations along with monolith.

I saw in many cases that this practice worked well but at the same time cost and overhead of maintaining and managing them also grew. Sometimes managing configurations across large cluster microservices goes head to head with management of the services.

For some reason I tend to think of microservices (or nanoservices) as a living creature, built out of cells (ie. Services). And this thinking leads me to a different point of view. How a large orchestrated cluster of nanoservices (ie. Human body) can possibly keep and propagate all the respective configurations to different organs to do their job properly? The answer is *DNA and Chromosomes*.

So let's see how we can apply this pattern into software and microservices. The idea is this, each service carries a full immutable configuration of the whole system, like:

  * How to use service discovery to find a service
  * What are Certificates and Credentials to communicate wit other services
  * What protocols the other services use
  * and so on ...

The idea is simple, the configuration is encrypted and backed into each and every service along with its signature. Each service knows how to verify and decrypt this configuration. But the decrypted configuration itself has multiple sections of encrypted configurations and each service has the key that only decrpt the parts that it needed by that service and not the other parts.

At the build time each service's build process fetches the configuration and embeds that into the service. Starting up service verifies the integrity of configuration with its signature, then it decrypts the configuration with a shared key and finally using its key it tries to decrypt the parts that is intended to use.

