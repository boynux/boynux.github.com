---
layout: post
title: PHP Envirronment Specific Configurations
excerpt: In PHP, environment based configurations such as URLs hostnames and etc. is always a challenge. And it gets worse when you add more and more environments. Is there an easy way to overcome this challenge?
tags: PHP Configuration Environment
category: Configuration Management
---

### Drama

<img src="{{ bsae.url }}/img/PHP_Logo.png" alt="PHP Envrironment Specific Configurations" title="PHP Envrironment Specific Configurations" align="right" width="320" />

Most of applications now a days dealing with all sort of data storages to caching systems and on and on. By adding to this external web services, little by little you realize that you need a proper configuration management in place.

There are plenty of choices in front of you, YAML, JSON, XML, INI, Database, and a lot more. You just need to Google it!

But when project grows another requirement pops up. Working with complex projects with lots of developers and all sort of acceptance and performance tests demands for different services and *stubs*.

Now you have another problem! Multiple database IP addresses, cache systems, cloud service configs, external mocked service URLs and so on. Naturally most people think of having another configuration file for test server for instance. 

And usually this is a justification: "Well, there are only two or three different config files. It shouldn't be that difficult to keep them in sync."

But as projects evolves and more and more developers and operation teams and testers come on board, you'll have more and more of those configurations hanging around. Now you have different development environments, multiple test servers, staging server and potentially multiple (or distributed) production servers. (Someone said blue/green deployment?).

DevOps start to bang their head against walls to find a *little* bug which is hidden somewhere, neither could be found nor reproduced! Developers blame operations, operations, devops and ...

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

### Personal experience

I've be in such a situation, and because we're (I'm) a genius, We came up with a brilliant idea. A very sophisticated, comprehensive, bullet proof and unique solution! A *host aware* configuration class!
How does that work? Very *simple*! For any host you just add a new configuration files which contains those parameters supposed to be overridden. Awesome!

But wait, there is a *very small issue* with this idea! Well, if I have just 10 different config files with just 5 different environments (In reality it's far more), you'll end up with **50** different config files! Of course finding a bug that caused by configuration is a disaster!

### There are other geniuses too

Recently I was working on a large project in Ruby and I was looking for a way to avoid this issue (again)! After a couple of weeks looking into different stuff and trying this and that I found this:

> [The Twelve-Factor App][1]

Take your time to skip though it. It really worth reading. But I want to draw your attention to 3rd factor. TADA! Configuration.

It says: 

This (configuration) includes:

+ Resource handles to the database, Memcached, and other backing services
+ Credentials to external services such as Amazon S3 or Twitter
+ Per-deploy values such as the canonical hostname for the deploy

OK, I was explaining this whole thing from the beginning of this post. Maybe I could just write those 3 bullet items :)

Going a little further, you'll notice:

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

> The twelve-factor app stores config in environment variables

That's interesting! Before this I though we can only store "DEV", "TEST" and "PROD" values in ENV. Dumb! Well, it turns out we can store a lot more.

Simply the idea is having all environment based parameters which I mentioned before, to be read from environment variables and then forget about it! What does this mean? Let me shed a light on it with an example:

Imaging in a very common scenario I want to call an endpoint in my application, usually you'll see something like this:

    url = $config->that_interesting_service_url;
    Guzzle::Get(url, [$some_other_paramters]);

Looks pretty neat and clean! Wrong ...

This way you'll need to maintain all those configuration stuff and it's what we want to avoid. What to do?

    url = $_ENV['that_interesting_service_url'];
    Guzzle::Get(url, [$some_other_paramters]);

Where is config values? No where in your machine. Where does it come from? Your environment configuration management system. Have you ever heard of [Puppet][4] or [Chef][5] or maybe [Ansible][6]? Of course you did!

There's no need for those complex host based config files any more. Eventually you have to setup your infrastructure in a way or another, you just inject those environment variables into your system. Probably via `/etc/environemnt` file. If you're using *containers* that's even simpler. Just use let's say `-e` in Docker.

With this approach you hit 10 birds with one stone.

+ Simpler config files.
+ Independent environments.
+ No need to check-in sensitive info with code.
+ No more mistakes.
+ Easy debug.
+ No exponential complexity with growth of environments.
+ Separating env based configuration from application.
+ Happy operation teams.
+ Awesome devops.
+ You add 10th yourself!

### Bonus:

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>
<br />

In ruby there is a Gem called [`DotEnv`][2] which actually provides a convenient way for developers to store all those environment variables into a file that's usually called `.env`. This way they don't need to set all those values into their machines. 

Usually you don't need to check-in that file. But if you remember not to put sensitive data in them, those can be checked-in too.

Fortunately there is a similar project in PHP with the same title, and unfortunately this project lacks some of core requirements of this concept. I guess developer(s) initially didn't grasp the idea properly. 

One major issue is, you have to check-in `.env` otherwise default load function throws an exception. There is another method that doesn't, but this behavior shouldn't be the default.

The code structure is not that clean and good too. But I believe with a PHP community's help these issues will be vanished soon. At least those guys felt the necessity of such a library out there, created one and did a great job!

And here is a [PHP DotEnv][3]

If you currently don't use this kind of configuration, I hope this post helps you to get the idea. If you think it's going to facilitate configuration management and eventually deployments please leave a comment. I'll be very glad to hear about your experience and views.

[1]: http://12factor.net/
[2]: https://rubygems.org/gems/dotenv/versions/2.0.1
[3]: https://github.com/vlucas/phpdotenv
[4]: https://puppetlabs.com/
[5]: https://www.chef.io/chef/
[6]: http://www.ansible.com/home
