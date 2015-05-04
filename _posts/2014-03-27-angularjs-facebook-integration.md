---
layout: post
title: AngularJs Facebook integration how to
excerpt: Have you ever wanted to integrate third party frameworks into AngluarJs? This is an explanation of AngularJs Facebook integration, in great details. I show you how to deal with asynchronous calls and prmises.
---

If you've ever written any code in Javascript, you've already experienced asynchronous calls complexities. But that wouldn't be much difficult to overcome when you understand the basics.

[<img class="size-medium wp-image-845 alignright" title="AngularJS Facebook" alt="AngularJS Facebook" src="http://www.boynux.com/wp-content/uploads/2014/03/angularjs-facebook-300x120.png" width="300" height="120" />][1]  

## Asynchronous Complexity

Things start getting more tricky when dealing with two or more different frameworks which work asynchronously and you need them to work nicely together. My problem was AngularJs Facebook API integration. After lots of investigation I managed to successfully make them work together. If you've a similar situation, read on to find out more about this. 

## Any problem? 
<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>
<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

The problem comes when you can not control when different frameworks done their initialization and calls. Because some calls and initialization happen asynchronously the code some times works and some times fails randomly or might not work at all. Other problem is synchronizing AngularJs scopes with Facebook API calls, so you can get latest updates easily in Angular friendly way. 

## Angular Providers?
The fact that you have to initialize Facebook API with APPID before start using it, leads us to use Angular Providers. These builders can run at application configuration time. At this time not any module created yet, so you have a chance to initialize Facebook API successfully. Let's take a look at Providers construct, according to AngularJs documentation, providers can be created with the following syntax: 

{% highlight javascript linenos %}
var module = angular.module('myModule', []);
module.provider('myServiceName', function myServiceNameProvider() {
    // Initialization goes here.

    this.$get = ["$service1", "$service2", function ($service1, $service2) {

        // Specific service logics goes here.
        // This method ($get) must return an object, which is called service and injected suing Angular $injector
        return new function () {
            // Actual service implementation.
        }
    }];
});
{% endhighlight %}

Important note: We don't have access to any of AngularJs or  3rd party modules in provider function, because they're in `config` phase too. The only module that is available is `$injector`.  But later in factory method which is `$get`, you can inject any module you need. Another thing you must consider is, provider function is called when you call config function or your application and injecting this module to your application module. But $get which actually creates service is called when you inject your service in other parts of your application module (e.g.. Controllers). So these functions won't execute at the same time. 

## Facebook APi? 
So, how about Facebook API? What we need to do to initialize? Again according to Facebook documentation, the initialization of Facebook API is pretty simple and straight forward. Take a look at this code: 

{% highlight javascript linenos %}
<div id="fb-root"></div>
<script type="text/javascript">
    window.fbAsyncInit = function() { 
        FB.init({ 
            appId : '{your-app-id}', 
            status : true, 
            xfbml : true 
        }); 
    }; 

    (function(d, s, id) { 
        var js, fjs = d.getElementsByTagName(s)[0]; 
        if (d.getElementById(id)) { return; } 

        js = d.createElement(s); 
        js.id = id; js.src = "//connect.facebook.net/en_US/all.js";
        fjs.parentNode.insertBefore(js, fjs); 
    }(document, 'script', 'facebook-jssdk'));
</script>
{% endhighlight %}

I divided that code into 3 parts to explain what it does. First part is a simple place holder for API to inject scripts and HTML into DOM. The important part is id and should be fb-root. This element must be visible meaning that should not be `display: none;` or `visibility: hidden;` set. 

    <div id="fb-root"></div>

Second part is actual initialization function which is assigned to `fbAsyncInit` property of window object. When Facebook SDK is loaded (in 3rd part) it looks for this callback in window object and runs it. This function we simply calls `FB.init` method passing some parameters including `appId` key from our Facebook application. This is also very well documented in Facebook developers documentation. Hence, I don't get any further into details. 

{% highlight javascript linenos %}
window.fbAsyncInit = function() { 
    FB.init({ 
        appId : '{your-app-id}', 
        status : true, 
        xfbml : true 
    }); 
};
{% endhighlight %}

And last part in creating script DOM element to inject Javascript element with Facebook SDK source address.

{% highlight javascript linenos %}
(function(d, s, id) { 
    var js, fjs = d.getElementsByTagName(s)[0]; 
    if (d.getElementById(id)) { return; } 

    js = d.createElement(s); 
    js.id = id; js.src = "//connect.facebook.net/en_US/all.js";
    fjs.parentNode.insertBefore(js, fjs); 
}(document, 'script', 'facebook-jssdk'));
{% endhighlight %}

## Facebook Provider: 
Now let's create Angular Provider for Facebook API step by step. Following Angular documentation I created a simple service provider. First creating new module for Facebook: 

{% highlight javascript linenos %}
var module = angular.module ('bnx.module.facebook', []); 
{% endhighlight %}

Next is defining provider function to initialize Facebook API. I've put some default values there, but still there is a chance for client to initialize API with new parameters later. I'll go through these properties later. The only one that I would like now to point is `defaultParameters`. This is default initialization values for Facebook API. If you don't override those values in your application's configuration phase default ones will be used.

{% highlight javascript linenos %}
module.provider ('facebook', function facebookProvider () {
    var initialized = false;
    var defaultParams = { appId: '1234567890', status: true, cookie: true, xfbml: true };
    var facebookEvents = {
        'auth': [
            'authResponseChange', 
            'statusChange', 
            'login', 
            'logout'
         ]
    }
{% endhighlight %}

This init function is very similar to the one which is provided by Facebook documentations. The only exception is `processPostInitiailizeQ` method call , which is supposed to execute any callbacks which is stacked for post initialize time. Note how this function is exposed to public by binding it to `this`. We are also changing `initialized` property to true. You'll see later why we need that property. 

{% highlight javascript linenos %}
this.init = function (params) {
    window.fbAsyncInit = function() {
        $.extend (defaultParams, params);
        FB.init(defaultParams);

        initialized = true;
        console.log ("Facebook initialization done.");

        processPostInitiailizeQ ();
    };
};
{% endhighlight %}

Those two private methods are registering and executing post process callbacks respectively. 
First one simply pushes given parameters into a `Q` array. First parameter is actual callback function. 
Second one is the context which the function have to bind to. Third parameter is method arguments. Second function loops through all registered items by first function and executing given callback using `apply`.

{% highlight javascript linenos %}
function executeWhenInitialized (callback, self, args)
    Q.push ([callback, self, args]);
};

var processPostInitiailizeQ = function ()
    while (item = Q.shift ()) {

        func = item[0];
        self = item[1];
        args = item[2];

        func.apply (self, args);
    }
};
{% endhighlight %}

Voilà, Here is the service definition itself. This is fairly simple and clear. I injected `$rootScope` and `$q` into service which is allowed here, but not in provider function itself. 

{% highlight javascript linenos %}
this.$get = ["$rootScope", "$q",  function ($rootScope, $q) {
{% endhighlight %}

What is this `promise` thing? In fact you don't really need to use this but here I utilize the Angular $q module. This gives us an ability to simply execute our callbacks whenever data is ready from Facebook calls instead of passing our callbacks directly to our service interface. For more information on `$apply` and `$digest` read [AngulaJs apply explained][2]. 

{% highlight javascript linenos %}
var promise = function (func) {
    var deferred = $q.defer ();

    func (function (response) {
        if (response && response.error) {
            deferred.reject (response);
        } else {
            deferred.resolve (response);
        }

        $rootScope.$digest ();
    });

    return deferred.promise;
};
{% endhighlight %}

<div style="float: right;"> <ins class="adsbygoogle" style="display: inline-block; width: 300px; height: 250px;" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7261521241"></ins><script type="text/javascript">(adsbygoogle = window.adsbygoogle || []).push({}); </script> </div> 
These lines of code are defining registerEventHandlers method. This method registers to Facebook API events a simple handler which broadcast those events through Angualr application. With this method we don't need to bind our callbacks directly into Facebook and we can use `$rootScope.$on`. This is more native and makes code easier to write and maintain. This function is here because we need `$rootScope` to be available. It uses `facebookEvent` property which is defined at the beginning of the factory method. You can add or modify different Facebook event you might be ingested in. The format of events would be `fb.<domain>.<event>` (e.g. `fb.auth.login`). 

{% highlight javascript linenos %}
var registerEventHandlers = function () {
    angular.forEach (facebookEvents, function (events, domain) {
        angular.forEach (events, function (_event) {
            FB.Event.subscribe (domain + '.' + _event, function (response) {
                $rootScope.$broadcast('fb.' + domain + '.' + _event, response);
            });
        });
    });
};
{% endhighlight %}

The next two methods are the actual functionalities the we are going to provide using this service. I created two for sample, though these two would be the most useful ones. Now you see how we utilized Angular `$q` to return promise to client instead of receiving callback as an argument and pass it over to Facebook API. This approach leads to cleaner code. Other API functionalities can be implemented in almost same way.

{% highlight javascript linenos %}
var login = function (params) {
    return promise (function (callback) {
        FB.login (function (response) {
            callback (response);
        }, params);
    }, $rootScope);
}

var api = function (path) {
    return promise (function (callback) {
        FB.api (path, function (response) {
            callback (response);
        }, params);
    }, $rootScope);
}
{% endhighlight %}

But, we still didn't register event handlers for Facebook API to broadcast events through Angular scopes. The reason is we need an initialized Facebook API first. But when this happens? Well, depends on different factors like network speed and also client's browser performance. If we just call `registerEventHandlers` somewhere in the code, it might randomly work or fail! We need a mechanism to ensure that we call this method only and only when API is completely initialized and ready. Now guess why we have a `initialized` property in provider. First check whether API has initialized yet, if not we register our method with `executeWhenInitialized`. Remember this function ensures that our request will be executed after initialization is done. And if it's already done, then good to go, just call method directly.

{% highlight javascript linenos %}
if (!initialized) {
    executeWhenInitialized (registerEventHandlers, this, []);
} else {
    registerEventHandlers ();
}
{% endhighlight %}

Our factory method (ie. `$get`) should return an object after execution, this object will be injected by Angualr to other modules when requested. So we expose the minimum required functionalities. And that's `api` and `login` methods.

{% highlight javascript linenos %}
return  {
    api: api,
    login: login
}
{% endhighlight %}

One can extend this factory to expose more functionalities as required. 

## Missing parts? 

Well, still some parts are missing! That's right, what happened to Facebook `<div id='fb-root'></div>` and loading API Javascript? Because those requirements are DOM properties we have to put them in Angular directives. What is Directive?  According to AngularJs documentation, Directives are high level markers that tell Angular HTML compiler to attach a specified behaviour to that DOM element or even transform the DOM element and its children. Directive would be very simple, here is the code:

{% highlight javascript linenos %}
module.directive ('facebook', function ($location, facebook) {
    var template = "&lt;div id='fb-root'&gt;&lt;/div&gt;";     
    return {         
        restrict:'EA',         
        template: template,     
    } 
});
{% endhighlight %}

<ins class="adsbygoogle" style="display: inline-block; width: 728px; height: 15px;" data-ad-client="ca-pub-7360583392867579" data-ad-slot="7680323644"></ins><script>(adsbygoogle = window.adsbygoogle || []).push({}); </script> 
## How to use it:

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

I guess you already know how to use this new service. But to make article complete I provide a simple Angular example. Create our application module and define its dependencies, which is our new Facebook module.     

{% highlight javascript linenos %}
var myApp = angular.module ('myApp', ['bnx.module.facebook']);
{% endhighlight %}

And now in config part of our app we inject `facebookProvider`. And then we call `init` method of provider with parameters object if required. 

{% highlight javascript linenos %}
myApp.config (['facebookProvider', function (facebookProvider) {
    facebookProvider.init ({appId: "abcdefg"});
}]);
{% endhighlight %}

Here I use two features of our new service, first one is how we register event handler via Angular for our Facebook events. And second one how I utilized promise object to retrieve Facebook user information when we are connected. 

{% highlight javascript linenos %}
myApp.run (['$rootScope', 'facebook', function ($rootScope, facebook) {
    $rootScope.$on ('fb.auth.authResponseChange', function (event, response) {
        if (response == 'connected') {
            facebook.api ('me').then (function (result) {
                $rootScope.userInfo = result;
            });
        } else {
                $rootScope.userInfo = null;
        }
    }
{% endhighlight %}

And now we are ready to call our Facebook service login. This actually logs in user through Facebook oAuth. 

        facebook.login ();
    }]);

And finally add this HTML tag to your page: 

    <div facebook></div>

That's it for now. I hope you find it useful. Please don't forget to leave comments via the box at the bottom. And please share it if you think it could be useful to others. You can find complete facebook module source code on [github][3]. 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

References: 

* [AngularJs documentation][4]
* [Facebook Javascript documentation][5]
* [AngularJs apply explained][2]

[1]: http://www.boynux.com/wp-content/uploads/2014/03/angularjs-facebook.png
[2]: http://www.boynux.com/angularjs-apply-explained/ "AngularJs apply explained"
[3]: https://github.com/boynux/AngularFacebook "Facebook module on GitHub"
[4]: http://docs.angularjs.org/guide
[5]: https://developers.facebook.com/docs/javascript
