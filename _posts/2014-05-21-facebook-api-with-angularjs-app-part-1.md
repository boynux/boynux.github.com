---
layout: post
title: Using Facebook API with AngularJs app - Part 1
excerpt: In this part I'm going to cover how to use Facebook API with AngularJs App to implement Like, Recommend and Share buttons. Let's add some cool social features in our AngularJs apps easily.
---

<img class="size-medium wp-image-845 aligncenter" title="AngularJS Facebook" alt="Facebook API with AngularJs app" src="{{ site.url }}/img/angularjs-facebook-300x120.png" width="300" height="120" />

Integrating social network authentication and features into web sites are very common nowadays. There are growing number of web applications here and there providing their services via social network authentication protocols. 

In another post I explained details of creating AngularJs module to integrate Facebook Api into your web applications.

Here I create a sample Facebook App and explain how one can actually use that library in a live example. This means you can see the actual code in work here and login and and post/share into your Facebook account using this app. This is not going to be very sophisticated and fancy application but quite explanatory to show some common use cases of Facebook Api using fast growing and power full Javascript framework AngularJs.

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Responsive Display --><ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="auto"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

In this part I'm going to cover Like/Recommend buttons. 

## 1\. Before you start

Before starting to write any application using Facebook oAuth and Graph API you need to register a Facebook app <a title="Facebook apps" href="https://developers.facebook.com/apps" target="_blank">here</a>. you will need a Facebook app id. I did create one for sake of this article.
  
So, whenever you see `<app-id>` it means you must replace that with your actual Facebook App ID, when developing your application. Facebook App ID is not something secret so it worthless to say you can sue in in Javascript applications.

Another thing I would like to mention is that, this sample application is only uses Facebook Javascript API, other languages like PHP, Python are out of scope of this article.

## 2. Let's start

Before you continue to read this article make sure you've already been through my other post [AngularJs Facebook integration how to](http://www.boynux.com/angularjs-facebook-integration/) if you are not. That article explains how to create an AngularJs modules that encapsulates Facebook Api and make it kind of Angular Friendly.You can get the module Javascript file from [here](https://github.com/boynux/AngularFacebook). I'm going to use the exact file from same repository.I include those two lines in order to use Facebook and AngularJs API.

## 3. Basic Angular App setup

The code snippet which will be provided shortly, is a very basic AngularJs application setup and initializing aforementioned Facebook module.

    var app = angular.module ('myFacebookApp', ['bnx.module.facebook']);

And we need this HTML snippet somewhere in our document body. This directive is actually loads Facebook Javascripts SDK. 

    <div ng-app="myFacebookApp">
        <facebook app-id="<app-id>"></facebook>
    </div>

We are done! Very simple :) Now we have initialized Facebook API and it's ready to be used.
  
## 4. Like button

Let's create a Like button, right here. The only thing that we need to do is adding this HTML code somewhere between your Angular app tags.

<facebook-like></facebook-like>

<script src="//ajax.googleapis.com/ajax/libs/angularjs/1.2.15/angular.min.js"></script>
<script src="https://raw.githubusercontent.com/boynux/AngularFacebook/master/facebook.js" async></script>
<script language="javascript">
    var app = angular.module ('myFacebookApp', ['bnx.module.facebook']);
</script> 
  
<div ng-app="myFacebookApp">
  <facebook app-id="1491187207767298"></facebook> 
  <facebook-like></facebook-like>

    <p>Voila, do you see Like button just above this line? Click to like this page!
    Now let&#39;s try another type of like button and also recommend button:</p>
    <div class="highlight"><pre><code class="language-javascript" data-lang="javascript">    <span class="o">&lt;</span><span class="nx">facebook</span><span class="o">-</span><span class="nx">like</span> <span class="nx">layout</span><span class="o">=</span><span class="s2">&quot;button_count&quot;</span><span class="o">&gt;&lt;</span><span class="err">/facebook-like&gt;</span>
    <span class="o">&lt;</span><span class="nx">facebook</span><span class="o">-</span><span class="nx">like</span> <span class="nx">layout</span><span class="o">=</span><span class="s2">&quot;recommend&quot;</span><span class="o">&gt;&lt;</span><span class="err">/facebook-like&gt;</span>
    </code></pre></div>
    <p><p> 
    <facebook-like layout="box_count"></facebook-like> 
    <br /> 
    <br /> 
    <facebook-like action="recommend"></facebook-like> 
    <p></p>

    <p>The other button type is share: </p>
    <div class="highlight"><pre><code class="language-javascript" data-lang="javascript"><span class="o">&lt;</span><span class="nx">facebook</span><span class="o">-</span><span class="nx">like</span> <span class="nx">share</span><span class="o">=</span><span class="kc">true</span><span class="o">&gt;&lt;</span><span class="err">/facebook-like&gt;</span>
    </code></pre></div>

<div>
   <facebook-like  action="like" share='true'>&nbsp;</facebook-like>
</div>
    <br /> 

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <!-- Display Rect Medium --><ins class="adsbygoogle" style="display:inline-block;width:300px;height:250px" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7261521241"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

<p>And show faces: </p>
<div class="highlight"><pre><code class="language-javascript" data-lang="javascript"><span class="o">&lt;</span><span class="nx">facebook</span><span class="o">-</span><span class="nx">like</span> <span class="nx">show</span><span class="o">-</span><span class="nx">faces</span><span class="o">=</span><span class="kc">true</span><span class="o">&gt;&lt;</span><span class="err">/facebook-like&gt;</span>
</code></pre></div>
<p><facebook-like show-faces='true'></facebook-like></p>

<p><a href="http://www.boynux.com/facebook-api-with-angularjs-app-part-2/">Continue to second part ...</a></p>

<p>In part two we will see how we can integrate Facebook login feature in our Angular app. Keep in touch. In the mean time you can leave your comments or suggestion.</p>

</div>
