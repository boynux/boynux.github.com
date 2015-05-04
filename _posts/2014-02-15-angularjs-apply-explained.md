---
layout: post
title: AngularJs apply explained
excerpt: If you're working on Angularjs apps and getting $digest in progress or similar errors, it means that you still need to know more about AjngularJs apply.
---

[<img class="size-medium wp-image-708 alignright" style="margin-left: 20px; margin-right: 20px;" alt="AngularJs Apply" src="http://www.boynux.com/wp-content/uploads/2014/02/angularjs-apply-300x168.png" width="300" height="168" />][1]

<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>
<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

If you're working on Angularjs apps and getting `$digest` in progress or similar errors time to time, it means that you still need to know more about AjngularJs apply functionality and syncing your asynchronous calls with AngularJs properly. 

While `$apply` and `$digest` concept are described very well in AngularJs documents, here I just share my experiments. As a rule of thumb you rarely need to call $apply (and almost never $digest except special cases like unit testing). The common usage of `$apply` would be the time you need to sync external DOM events like XHR or 3rd party libraries with your AngularJs app. If you see you are calling `$apply` many time in your app here and there, in most cases there is something wrong with your code and you need to fix it. 

## What `$apply` does? 
This is a very simple function that force AngularJs to run `$digest` and trigger all listeners and watchers about new changes in their scopes. 

## Proper usage: 
This is how you would normally call this function: 

{% highlight javascript linenos %}
$scope.$apply (function () {
    $scope.someVariable = 'changed';
});
{% endhighlight %}

And if you miss it your app is most probably does not get informed about changes and might cause out-of-sync model updates. 

## Why is there an error? 
This error means you've tried to call `$apply` in a scope where another `$apply` is still in progress. It's time to revise your code. for example this code would generate error: 

{% highlight javascript linenos %}
$http.get (url).success (function (result) {
    $scope.$apply (function () {
      $scope.newValue = result;
    });
});
{% endhighlight %}

Because all AngularJs modules (in most cases) keep their changes inside `$apply` to keep it in sync. Meaning that the above 'success' call back would be called within `$apply` itself. You don't need to (and you can't) do that again. 

## When to call $apply?

<div style="float: right;"> <ins class="adsbygoogle" style="display: inline-block; width: 336px; height: 280px;" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7819924448"></ins><script type="text/javascript">(adsbygoogle = window.adsbygoogle || []).push({}); </script> </div> 

Most of usual and common asynchronous DOM functionalities like window, timeout, Ajax calls and ... are already provided as AngularJs modules and they in sync with it (ie. `$window`, `$timeout`, `$http`). But still there is some cases that you need to call this function. 

Recently I was working on a project which is using AngularJs and Facebook API and because Facebook API library is asynchronous, I was having difficulties to keep them in sync. Obviously this is one of rare cases that you need to use `$apply`. 

Let's see this in a simple example, while this example could be in AngularJs way and not using `$apply`, but I just write it in the way that it needs to call `$apply`. (this example is not a good practice). 

{% highlight javascript linenos %}
angularApp.controller ('ExampleCrtl', [$scope, function ($scope) {
    window.setTimeout (function () {
        $scope.$apply (function () {
            $scope.somethingChanged = "I'm changed";
        });
    }, 1000);
}]);
{% endhighlight %}


And just for clarification here is the same example in AngularJs way: 

{% highlight javascript linenos %}
angularApp.controller ('ExampleCrtl', ['$scope', '$timeout', 
function ($scope, $timeout) {
    $timeout (function () {
        $scope.somethingChanged = "I'm changed";
    }, 1000);
}]);
{% endhighlight %}

## References:

* [AnuglarJs Scopes doc](http://docs.angularjs.org/api/ng.$rootScope.Scope)

<ins class="adsbygoogle" style="display: block;" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="auto"></ins><script type="text/javascript">(adsbygoogle = window.adsbygoogle || []).push({});</script>

[1]: http://www.boynux.com/wp-content/uploads/2014/02/angularjs-apply.png
