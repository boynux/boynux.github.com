---
layout: post
title: Your consumer's system is your trusted storage.
excerpt: Don't waste your compute and storage resources to store and process data that could be stored into client's system. Your consumer's system is your trusted storage.
tags: storage client encryption
category: Ideas
---

### Use databases for good!

<img src="{{ site.url }}/img/database.png" alt="Your consumer's system is your trusted storage." title="Your consumer's system is your trusted storage." align="left" />

I always see developers tend to store all sort of information in their secure databases with the hope that those information never leaks into public or being changed by unauthorized users. That's a very valid concern indeed. We have to keep consumer's data and information safe. That's extremely important. But not all information worth to be treated like that. Some information can simply be kept in client's system. 

By increasing the Internet usage globally and modern browsers and standards (eg.HTML) with amazing capabilities, we see more and more companies and service providers are moving away from desktop applications toward web and mobile ones. These tools are a great potential to store these kind of data on your application server's behalf.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

### Browser is your friend

Let's look at some places that we can stuff some data in it. 

+ Query strings
+ Cookies
+ [Web storage][1]

The last one is obvious, it's made for that purpose but not all browsers at time of writing this post are 100% supporting that feature. My focus is on two other possibilities.

### Something simple

For example, If we're working on a web site and we want to track user's last visited page, the obvious options would be upon any page request at server side we store requested address in user's session or if it's an authenticated users most probably a database record. But if this requirement is really something nice but not a must to have, why not using cookies? We can store user's last visited page in cookie on the user's browser, of course if browser removes that cookie for any reason we loose that information. But that's totally depends on business requirement. This is just another option. 

### Welcome to the show

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="rectangle"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

Let's see a more practical example. Once I was working on a complex application which supposed to collect some information from client, then processes them and based on that shows a sequence of pages to the user. So we needed to track last user state to be able to show the next page correctly. The sequence is not predefined and based on user interaction and provided information might change.

So the ovbious way to do that is to store those information and page sequence in session or any data store and then create user experience based on that. Bit it was not that simple. That particular app supposed to work exactly the same way on *feature phones* (The old phones with very basic web browsers). That is a business requirement. You can not change it.

*Just to give you some context, we couldn't even use Javascript :)*

What we came across, is to use query string to track these data. After all those information are user's data. So it doesn't matter to expose that to user herself.

Now there is another issue, *What if user tries to change those information and injects some corrupted data into the system. The solution is as simple. We added a simple signature to those data. By doing that user is not able to alter those data anymore, because she can't regenerate that hash again. She does not have enough data to do that (and algorithm as well).

Of course after doing all that we encoded those data into URL safe *base64* just to make sure it's strange enough to a normal user ;)

### Tackle the security

How often you had to click a *email verification* link! How does those links work?

It seems that, a key to a record in database has been sent to you. When you clock on the link you send that information to server, it'll fetch data, perhaps check expiration date and then validates user email!

But do we need to do all that to validate user's email? There are at least four DB transactions happening here:

1. Storing user's data into databse with a unique key
2. Retrieving those information again
3. Finally validating user's email
4. Well, we need to delete unneeded/expired data too

We can eliminate three of these transactions. The idea is, we store all those information at client side. In that token obviously. What you've to do is to 

1. Create an object with needed information
2. Create a signature and add to it
3. Marshal that object
4. `base64` and send it to user

No database interaction at all. Now when user clicks on the link, you'll get the token. Then you just decode *base64* and restore that object, check for date, if it's valid then you regenerate signature based on provided data and compare it to the signature which is in the data. If it matches then it's a genuine data! Update user table with verified email address.

<div class="ads"> 
    <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-5768423765640512" data-ad-slot="7013600384" data-ad-format="horizontal"></ins> 
</div> 
<script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>
<br />

If client tries to change anything in the marshaled data, it'll break the data signature. And that obviously invalidates those information. Expiry date is also there, you don't need to do anything about that.

The other benefit you'll get for free is that, if for any reason, you want to invalidate all those verification data, you can just change the signature encryption key. All get invalidated at once! I don't why you may need to do that, it just poped up in my mind ;)

You can do the same thing for a more common features, like auto login. You can store things like username, expiry date and some other information like page style and stuff like that + signature (which is generated based on user password and probably another secret key in that data). Set a cookie and done. When user wants to login just get that cookie information, check expiry date, 

* Expired? Delete cookie and ask to login again. 
* Not expired? Verify signature with actual signature, correct? *Welcome back!*

The good thing is, if user changes her password, it automatically invalidates the cookie, so you don't need to do anything about that as well.

I can't think of how many other possibilities are laid in that. If you've ever used something similar or you could think of any other possibility please leave a comment.

<sup>Image from essentialsql.com website</sup>

[1]: http://dev.w3.org/html5/webstorage/
