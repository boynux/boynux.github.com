---
layout: post
title: How to use Rake in build pipeline automation.
excerpt: Rake is a powerful and easy to use task management written in Ruby. Let's see how we can use Rake in build pipelines.
tags: Continues_Delivery Continues_Integration Rake Ruby PHP
category: Devops
---

### Introduction

<img width='160' src="{{ site.url }}/img/ruby-logo.png" alt="How to use Rake in build pipeline automation." title="How to use Rake in build pipeline automation." align="right" />

Are you currently using *build pipelines* or *continues integration* servers? If yes, before continue to read the rest let's take a look at your jobs.Assuming you are using PHP if you have something like this:

    Job Name: Build
    Command: composer install --no-dev

    Job Name: Test
    Command phpunit --config=phpunit.xml test/ 

    ...

It means that you're not using any task management tool. Why should I use it, you ask? Well, most of developer who are working with Java, C, C++, .. are quite familiar with build tools and task managements. But it's less known in interpreted languages. Those languages definitely need such tools, to facilitate build process, which is usually a chain of complex tasks. These tasks mostly should be executed in particular order to produce the desired artifact.

For example, in `C` language (which most of you did some small coding with that) in order to build an executable artifact, first we need to compile all source codes into objects (this process by itself is actually two tasks but usually compilers do it in one run). Then you need to call linker to link objects to proper libraries (statically or dynamically). In more complex build process, one may need to build several other libraries too.

In reality, if you want to manually do all these jobs a few problems arises:

1. Memorizing many commands.
2. Maintaining proper order.
3. Memorizing all dependencies.
4. Cleaning up all the mess.

<div style="float: left; margin: 4px;">
  <ins class="adsbygoogle" style="display:inline-block;width:300px;height:250px" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7261521241"> </ins> 
  <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

These are a few of problems that you might have. But I would like to draw your attention to item number "3". That has a special meaning. If you change a single file in your project you may not need to compile and link all files to build your artifact again, rather in most cases you'll need to compile just a few files that have dependencies to the modified file. And then in case of dynamically linked libraries you may only need to link that particular affected library, not the whole project.

But why should I care? I'm not compiling PHP or Ruby!? You're right! But I have a few reasons that might be interesting for you to use task managements in those languages too.

First of all, it's not only about compiling and linking, these tools can handle other tasks like:

* Running unit tests.
* Environment preparations.
* Acceptance tests preparations.
* Running Acceptance tests.
* Cleaning up resources.
* Managing dependencies.
* And few more ...

The other obvious reason is the build pipelines and CI jobs. By using tools such as Rake you have more standard and clean jobs. Also developers and automation servers use exactly same method to build, test and deploy projects. Which helps us to avoid mistakes and uncaught bugs due to differences in build, test or deploy.

So if you are not using them yet, I hope this tiny how to helps to get started.

### Tools

There are quite a few task management tools out there and more or less you can choose anyone you like. They all try to solve a very similar problem which is *Manging Tasks*. To name a few:

* Make
* Ant (or nAnt)
* Gradle
* Grunt
* Maven
* [Rake][1]
* ...

Perhaps you already know `make`. That's one of the oldest out there. Almost every C source code that I worked with got one. The rest except `grunt` and `rake` are very popular in `Java`. `grunt` is very popular across `javascript` communities. And finally `rake` my favorite which I'm going to explain here.

Why `rake`? Because it's very easy to use. Provides almost every thing that you may need to get the job done. Written in `Ruby` and because it uses *internal Ruby DSL* you can directly use full ruby power in tasks if you like to.

Here is an interesting [article][2] about Rake by *Martin Fowler*. In that article he explains what is build tool and details of how to use it with lots of good examples.

But here my focus is mostly on *Automation* rather than *Rake* itself.

### Getting started

First things first, although you may not need to compile and link your project, still you need to run your dependency management tool like, `composer` in PHP or `bundler` in Ruby. Let's see how we can do it with `Rake`. I assume you are using PHP but in other languages you just need to change command to the one you want to use.

`Rakefile`

    task :build do
      sh 'composer install --env=prod'
    end

What it does, it defines a task named `build` and from `do` to `end` is definition of task. `sh` executes its following string in system, in this case it runs `composer`. You can have more complex commands there.

Now let's run this taks:

    $ rake build

That's all you have to do.

Let's make it a little bit more practical. You may need to run this command in different environments, like `dev'. What you can do is to read this information from `env` for instance.

`Rakefile`

    task :build do
      sh "composer install --#{ENV['BUILD_ENV']}"
    end

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

OK, what's that strange syntax? It's Ruby string interpolation. It executes the command in `#{...}` and replace it with its result. In this case `BUILD_ENV` env variable value.

Now the following command will run `composer` with different environment:

    $ BUILD_ENV=dev rake build

    # or

    $ BUILD_ENV=no-dev rake build


But one the benefits of using task managements is you don't need to run a task multiple times if it has a same effect. In above example if you run build task several times, it does exactly the same thing. Resolves dependencies and installs required packages (maybe from cache in next runs). 


How to do it?

`Rakefile`

    task :build => ['.build']

    file '.build' do
      puts "Installing dependencies ..."
      sh "composer install --env=#{ENV['BUILD_ENV']}"

      touch '.build'
    end


<div style="float: right; margin: 4px;">
  <ins class="adsbygoogle" style="display:inline-block;width:300px;height:250px" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7261521241"> </ins> 
  <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

Introducing `File` task. What it does is very simple. It gets executed only if the provided filename (eg. `.build`) does not exists. And what we did is just creating that file whenever we execute `build` task by making it dependency of file task. (via `=>` operator)

But there is another issue, what if we really change 'composer.lock' file and we need to run update again? Then we have to do `rm .build`! Not a good solution though. Let's look at the procedure:

If `composer.lock` changes we have to run `composer update` other than that it's just waste of time because sometimes it takes an age to run :)

Now let's update our `Rakefile`:

`Rakefile`

    task :build => ['.build']

    file '.build' => ['composer.lock'] do
      puts "Installing dependencies ..."
      sh "composer install --env=#{ENV['BUILD_ENV']}"
      touch '.build'
    end

    rule 'composer.lock'

Looks good! Now if you change `composer.lock` then `build` task will automatically know it have to run `update` command again. Otherwise it will be executed only once. `rule` task here ensures that `composer.lock` changes will be tracked.

Let's go one step further:

We now that any changes in `composer.json` file, which contains all dependencies should generate new `composer.lock` file. Which contains all resolved package version.

Dependencies chain looks like this:

    build => composer.lock => composer.json

So let's apply that:

`Rakefile`

    task :build => ['.build'] do
    end

    file '.build' => ['composer.lock'] do
      puts "Installing dependencies ..."
      sh "composer install --env=#{ENV['BUILD_ENV']}"
      touch '.build'
    end

    rule 'composer.lock' => ['composer.json'] do
      puts "Update composer.lock file ..."
      sh 'composer update && touch composer.lock'
    end

Now our pipeline first stage is done. In your commit stage the only thing you have to do is to run `rake build`! And exactly same thing in development, assuming that you have proper environment variable set (optionally you can set a default value for that.)

### Demo time

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

I prepared a short video to show how it works in action:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FHUr0HO_Gqc" frameborder="0" allowfullscreen></iframe>

In [next post][3] I'll go through next stage which is `Unit test` stage. Meanwhile here is short video of above `Rakefile` in action.

[1]: http://en.wikipedia.org/wiki/Rake_%28software%29
[2]: http://martinfowler.com/articles/rake.html
[3]: {{ site.url }}/rake-pipeline-automation-part-2
