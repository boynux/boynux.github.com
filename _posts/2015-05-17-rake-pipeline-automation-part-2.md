---
layout: post
title: How to use Rake in build pipeline automation (part II)
excerpt: Rake is a powerful and easy to use task management written in Ruby. Let's see how we can use Rake in build pipelines. Part II
tags: Continues_Delivery Continues_Integration Rake Ruby PHP
category: Devops
---

### Introducing

<img width='160' src="{{ site.url }}/img/ruby-logo.png" alt="How to use Rake in build pipeline automation." title="How to use Rake in build pipeline automation." align="right" />

In the [first post][1] I explained how to use Rake to automate PHP `composer` dependency management. And avoid running `composer` tasks multiple times if it's already done and there are no changes in `composer.json` and eventually `composer.lock` files.

Now let's take one step further. Normally in pipelines in commit stage (or sometimes next stage) there is *unit test* job. Let's see how we can extend our `Rakefile` to handle this job too.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

### Dependency matters

The important point with implementing `unit test` task is, it's actually depends on on build task. That's what *task management* tools suppose to help us with. So in other to do that with Rake, we just create a new task named `test` and then make it dependent to `build task.

Let's see how it looks like:

    ....

    task :test => [:build] do
      puts "Running unit tests ..."
      sh './vendor/bin/phpunit tests/'
    end

Simple and easy, we just created a new task `:test` and made it dependent to `:build` task. From now on, if you run `rake test` without calling `build` first, it'll be done automatically. And any consequent run of `test` will not execute build, but the tests only.

### Arguments to survive

But what if in development stage we'd like to execute only a subset of tests? Let's say based on test groups? The solution is task arguments. Rake provides an easy way to send some extra arguments into task when we're calling them.

For this example I assume that we want to send a particular `group name` to `test` task. And then we pass that `group` into `phpunit` command line.

This is the code:

    task :test, [:group] => [:build] do |t, args|
      puts "Running unit tests ..."

      extra_args = "--group #{args.group}" if args.group
      sh "./vendor/bin/phpunit #{extra_args} tests/"
    end

The second parameter to `task` (ie. `[:group]`) is a list of arguments. Then in block you need to define two parameters, first one (ie `t`) is always a reference to task itself and second one is arguments.

The rest is very straight forward, we add `--group` argument to a string named `extra_args` if argument is not `nil`. Meaning that there is a `group` name provided by user. And then we pass that parameter to `phpunit` commnad.

In order to execute the above task with a specific group name, issue the following command:

    rake test[item001]

Of course you can use other methods to execute only a subset of tests like `--filter`. It's up to you. But the whole thing is pretty much like what I explained you just need to tailor it to your needs.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

### Defaults are good!

If you think running tests with `rake test` is too much to do, because you run the frequently then you can define a default task for you Rake. And make it so it runs tests by default. The what you need to do to execute test is just `rake`!

Here is how to do that:

    ....

    task :default => [:test]

### Extend it

You can all other related tasks like *Test coverage* too. It up to you to extend your rakefile and task as according to your need but make sure that you don't break any dependency.

### Show time

Again here is a video shows how those changes work in action:

<iframe width="560" height="315" src="https://www.youtube.com/embed/UrqLsuDIHFg" frameborder="0" allowfullscreen></iframe>

As usual please leave your comments and suggestion below, and thanks for reading :)

