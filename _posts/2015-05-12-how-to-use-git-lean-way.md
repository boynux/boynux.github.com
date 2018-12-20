---
layout: post
title: How to use Git in a Lean way
excerpt: We all know how to use Git. We all know Lean philosophy too. Now the question is "Do we use Git toward Lean philosophy?"
tags: Lean Git Continues_Delivery Continues_Integration
category: Methodologies
---

<img src="{{ site.url }}/img/git-logo.png" width="161" alt="How to use Git in a Lean way" title="How to use Git in a Lean way" />

## Lean

Let's review what Lean philosophy tells us in computer science. Lean is based on these three key principles which actually came from Toyota Production System (TPS): 

* Just-In-Time
    + Eliminate waste
* Stop The Line Culture
    + Mistake-Proof the System
* Relentless Improvement
    + Learn Through Experiment

<sup>Source: The Toyota Production System (1978)</sup>

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="rectangle"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

## Git

Git is a fantastic and pretty popular *Distributed Version Control System* which can be used in lots of different workflows. Of course there is nothing good or bad about them, but let's see how we can use it to get closer to Lean principles.

To continue, I need to jump to a few more concepts that I'm going to refer to them later. So I just skip through them quickly here:

* Continues integration

> Continuous integration (CI) is a practice - in software engineering - of merging all developer working copies with a shared mainline several times a day -- <sup>wikipedia</sup>

* Continues Delivery

> Continuous Delivery (CD) is a software engineering approach in which teams keep producing valuable software in short cycles and ensure that the software can be reliably released at any time. -- <sup>wikipedia</sup>

## What can we do?

Now that's a question, "What can we do to improve our Git workflow toward this principle?". I list my thoughts and reasoning for each of them here:

**1. Small commits:**

This one is the easiest to explain. With smaller commits you are actually eliminating the waste. Why? Because in order to commit your code, you need to develop logical checkpoints for you code. This makes you think about what you're doing. You'll realize that in order to commit you have to re-think about the goal you're going to achieve at the end. Now, that's the question: 

*Are these changes, that I'm going to commit really toward my goal?*

if not, don't commit your code! Just delete it!

**2. Meaningful commit messages:**

This one also falls in previous category. By trying to briefly explain what you've done in this commit, you're actually refreshing your brain and thinking about "What values this commit is going to have to your code!".

*If you cannot explain how these changes are valuable to your code toward that goal, delete it!*

**3. Push a working code.**

This one is very important. With each commit you may break some functionalities of your code. To some extent that's fine. But when you want to push your commits, whether it's just one or multiple commits on your local repository to remote, make sure it doesn't break any functionality. Run all unit tests. Run acceptance tests as much as possible.

Now one thing, if you are using CI and upon every push CI server will check your code against acceptance tests and unit tests, you may prefer to let CI server do the hard job of running acceptance test and unit tests for you. But my recommendation is at least make sure that all your unit tests are passing.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

**4. Push as soon as you can**

Well, this clear that if you push your code as soon as you can, you'll get feedback from CD, CI and other developers and teams as soon as possible. That's Lean.

The other fact is, if you push regularly you're actually experimenting your changes. This is  what "Relentless Improvement" is about. So don't let your code stays over night (which I did today :) ). If you cannot push it at the end of the day, delete it (seriously)!

**5. Always push to *master* branch.**

*I don't want to start a debate here again. So don't leave comments about how branching and feature branching is good. If you think it's good use it.*

But why no branching? With branching you're actually producing lots of waste! Why?

Look, when you don't push your code into *main* branch you're violating *Stop The Line Culture*. The reason is, if every developer works on his own branch at the time you push your changes, there is no chance to merge your code with other branches and run tests against them. You may merge master to your branch before push, or you may setup CI server to merge every branch with master and run test against that too. But it's rather difficult to do the same thing with all branches.

Even if you have a CI server with ability to merge different branches with each other, things get even more complicated. When developers change their code in separate branch at most they try to merge that with master (if they don't forget) but it's impossible to test merging with all possible branches from other developers. Hence, the moment they push their code and CI server tries to merge all branches, definitely there will be a merge conflict.

What really happens is, after finishing the work item, developers merge their code into master and push. For example this branch was around for a week, and there is another branch for couple of days. Once the other developers want to merge their code into master, since another branch is already merged it's very probable to get merge conflicts, or even worse, failing acceptance and unit tests. The problem is one week is to late to detect the issue. Developers already forgot what they did even two days ago.

The other issue with branching, is that it's affecting collaboration, one of important concepts of Lean. With pushing every commit several times a day into *main* branch, if any changes from any development team breaks the code, it encourages developers to communicate effectively with each other to eliminate the problem.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

**6. Let *CD* server do the hard job**

This falls into *Relentless Improvement*, by pushing small part of code which is working and potentially can go into production, you're gradually improving your product toward the main goal, which is the work item you're working on. By letting CI server run all sort of tests that you have, you're experimenting your changes in - close to - production environment too, which is fantastic. If something goes wrong, you catch it as soon as possible.

In Lean, there is no place for manual deployment. Pipelines are there to break by developers. To do experiments, and to deliver as fast as possible. You just write enough code to do the job. No less, no more!

## Conclusion

Be Lean and use Git the way it work for you and your team!
