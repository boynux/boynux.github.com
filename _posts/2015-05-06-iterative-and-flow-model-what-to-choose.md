---
layout: post
title: Iterative and Flow model what to choose
excerpt: In agile world most common development method is the one which is known as iterative. Breaking tasks into small 2-4 weeks iterations. But there is another option, Flow model!
tags: Agile Scrum Kanban Flow Iteration
category: methodologies
---

### What is Iterative model:

<img src='{{ site.url }}/img/iterative_and_incremental_development.png' title="Iterative and Flow model" alt="Iterative and Flow model" width='318' align='right' />

Most of common development methods in agile are known as [*Iterative models*][1]. Those methods break features into small stories and work happens within *iterations* with minimal planning. These minimal plans are based on product backlog which is considered part of a larger plan (which may not even exists) and usually happens at the beginning of each iteration.

This model is so well known between [*agile*][2] practitioners which became a De facto part of agile.

Iterations are *timeboxed* and typically 2-4 weeks long. These iterations should involve the whole release cycle from planning to testing (except release itself). Common phases in each iteration are:

+ Planning
+ Requirement Analysis
+ Design
+ Development
+ Automated testing (unit testing & acceptance testing)
+ Exploratory testing

At the end, new features will be demonstrated to stakeholders for feedback. At this point project must be ready for release with these new feature or without. Those not ready to release features might require couple of more iterations to become available for release.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

<br />
Teams are usually cross functional and as autonomous as possible to decrease *noise factor* and have more predictable *velocity*. This way stakeholder are able to roughly estimate how many iterations a particular feature may require to be ready for market release, so that they prioritize their backlog items based on that information and of course other business factors.

[*Scrum*][3] is one of well known methodologies which is neatly based on this idea. Roles are pretty well defined and iterations are strictly planned. In Scrum there is no excuse for breaking time frames or changing requirements within sprints.

### What is Flow model:

Flow model is less know development method among agile practitioners. The idea with flow is to breakdown those iterations into even finer steps so that it's possible to eliminate iterations at all. The work boundaries are defined by *work items* rather than time frames. But still smaller iterations can happen at different stages like development stage itself for example.

Flow model originally emerged from [*lean*][4] methodology and later found its path into software industry and specially agile community with lean movement.

The flow works likes this: 

> Each work item individually should make a working software with minimum bugs and in extreme end a releasable software.

Each work item is picked from a larger backlog and goes through more or less the following steps:

+ Requirement analysis
+ Design
+ Development
+ Testing
+ Release

Flow is the fundamental part of [*Continues Delivery*][5] and in that each code check-in should end up in stage environment and ideally production.

This method encourages good quality product with minimum waste. It also encourage team work and self discipline within a team. The goal is to deliver valuable software as soon as possible with optimum quality.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> 
    <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
</div>

<br />
In lean philosophy everything not adding value to the customer is a waste. Which can be translated in software as:

+ Unnecessary code.
+ Gold plating.
+ Delays/Defects due to lack of communication.
+ Manual testing (which can be automated).
+ Uncertainty about requirement.
+ Unnecessary documentations.
+ Cross team delays.
+ Incomplete features
+ and more ...

The exceptional method which works best with flow and can visualize it amazingly is [*Kanban*][6]. You can read more about *Kanban* [here][7].

### What to choose?

Iterative and Flow model, there are pros and cons in both of them. But in general if you require rapid feature releases and short release cycles (hours?) then Flow model is a good choice. Bit iterative model fits the products that need to release certain features at once and time to market is not as important (weeks?). If this's your requirement then iterative model is a better choice.

What do you think about these methods? If you have any experience with any of them please leave your comments below or tweet me.

<small>Image from WikiPedia</small>

[1]: {{ site.url }}/img/iterative_and_incremental_development.png
[2]: http://en.wikipedia.org/wiki/Agile_software_development
[3]: http://en.wikipedia.org/wiki/Scrum_%28software_development%29
[4]: http://en.wikipedia.org/wiki/Lean_software_development#See_the_whole
[5]: http://en.wikipedia.org/wiki/Continuous_delivery
[6]: http://en.wikipedia.org/wiki/Kanban_%28development%29
[7]: {{ site.url }}/why-you-cant-do-kanban
