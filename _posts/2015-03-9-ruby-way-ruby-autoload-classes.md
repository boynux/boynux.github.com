---
layout: post
title: Ruby autoload class
excerpt: It's very easy in Ruby to implement classes to autoload. If you don't know about Ruby autoload class feature here is a short description on how to utilize this feature to have cleaner codes.
---

In Ruby a common way to load classes is using `require`. recently I realized that in large projects with lots of classes this approach has some significant problems:

*   Loading all class at project bootstrap causes unnecessary overhead.
*   Unclear definition of modules and classes.
*   Difficaulty in adding/removing classes and modules.

<div class="ads">
<!-- Responsive Display -->
<ins class="adsbygoogle adslot_1"
     style="display:block"
     data-ad-client="ca-pub-5768423765640512"
     data-ad-slot="4587256441"
     data-ad-format="horizontal"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

Let's see an example, I've a module with some other modules and classes defined like this:

`File: lib/boynux.rb`

    module Boynux
    end
    
    require 'lib/boynux/module1/class1'
    require 'lib/boynux/module1/class2'
    require 'lib/boynux/module1/module12/another_class1'
    require 'lib/boynux/module2/another_class1'
    require 'lib/boynux/module3/class3'
    ...

As you see the code is not very clear which modules and classes are nested. You'll need to closely examine class path to understand that. When those `require` statements grow bigger and bigger. Things get more complex and tedius. Finally it's more probable to make mistakes in defining classes and modules.

### The Solution

To solve above problems, we can use Ruby `autoload` method, defined in `Module` class. First of all `autoload` loads classes whenever you try to access them, meaning you don't need to load all classes at bootstrap stage of your app. This will add a significant performance to your application bootstrap stage. Secondly, your modules and classes will be more clear and easier to maintain. and finally refactoring your project adding/removing and moving classes and modules are a lot easier.

<div class="ads">
    <!-- Responsive Display -->
    <ins class="adsbygoogle adslot_1"
         style="display:block"
         data-ad-client="ca-pub-5768423765640512"
         data-ad-slot="4587256441"
         data-ad-format="rectangle"></ins>
    <script>
    (adsbygoogle = window.adsbygoogle || []).push({});
    </script>
</div>

Let's see how we can refactor above code with `autoload` and you can see the result.

`File: lib/boynux.rb`

    module Boynux
        module Module1
            autoload :Class1, 'lib/boynux/module1/class1'
            autoload :Class2, 'lib/boynux/module1/class2'

            module Module12
                autoload :AnotherClass1, 'lib/boynux/module1/another_class1'
            end
        end

        module Module2
            autoload :AnotherClass1, 'lib/boynux/module2/another_class1'
        end

        module Module3
            autolaod :Class3, 'lib/boynux/module3/class3'
        end
    end

Personnay I prefer `autoload` feature and latest approach rather than `require`ing all clases like first example.

`autoload` syntax is very simple and clear, first argument is a class name `symbol` and second argument a path to the actual class' source.

You can find more info about `autoload` at ruby docs [Module#autoload](http://ruby-doc.org/core-2.1.0/Module.html#method-i-autoload)

Of course this apporach also have its own drawbacks, namely if you make a typo in class path won't know that unless you try to access that class somewhere in the code. Which might cause unexpected runtime exceptions (assuming you have decent unit test in place this shouldn't be a big problem anyway).

I'd love to hear your thoughts about this feature and wheather you're currently using it or not.  Please leave your comments below.
