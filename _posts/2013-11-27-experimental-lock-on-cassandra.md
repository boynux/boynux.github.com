---
layout: post
title: Experimental Cassandra Lock
excerpt: How to create an experimental Cassandra distributed lock system to be able to do synchronised insertion/updates on Cassandra.
---

## Intro 

[<img align='right' class="wp-image-438 alignright" style="margin-left: 20px; margin-right: 20px;" alt="Cassandra Lock" src="http://www.boynux.com/wp-content/uploads/2013/11/chef-says-okay-clip-art-300x296.png" width="300" height="296" />][1]
[Cassandra][3] is my favourite column oriented data store, I've decided to create a simple Cassandra lock system to be able to do synchronised updates on Cassandra, I hear you say there are several DLMs like Zookeeper or Memcached can be implemented as a simple locking system, but I like a solution purely based on Cassandra itself. After lots of surfing on the net I came up with nothing! absolutely nothing! well, I created one (better to say experimented). 

## Disclaimer 
Disclaimer normally comes at the end of article but, I put it here because, this is PURELY EXPERIMENTAL IMPLEMENTATION AND THIS SOLUTION WAS NOT USED IN PRODUCTION, THIS IS JUST A CRAZY IDEA INTO ACTION. Actually I'm not sure it'll work at all :-) - sigh - If you are looking for something that works please stop reading here and go back to [Google](http://www.google.com?q=distributed lock manager) otherwise you are welcome to test it and improve it. 

<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async="true"></script>
<div class="ads"> <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> </div> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

## Idea 
The original idea is not mine and you can find it on Apache Cassandra [web site][2] and it uses Lamport's Bakery locking algorithm. You can find more information about that in [Wikipedia][3]. 

> Lamport envisioned a bakery with a numbering machine at its entrance so each customer is given a unique number. Numbers increase by one as customers enter the store. A global counter displays the number of the customer that is currently being served. All other customers must wait in a queue until the baker finishes serving the current customer and the next number is displayed. When the customer is done shopping and has disposed of his or her number, the clerk increments the number, allowing the next customer to be served. That customer must draw another number from the numbering machine in order to shop again. -- Wikipedia

##  Definition 
I use two `Column Families`, `Numbers` and `Choosing`. Whenever a process wants to enter a critical section 

1. first it must get a new number, in order to do that it must look into `Choosing` `CF` for `true` value regardless of column names to make sure that nobody is withdrawing new number at the moment. 
2. Then informs other processes by puting its `PID` as column name (in actual code PID is optional and any other `unique ID` can be used) into `Choosing` `CF` with value of `true`, meaning that it's going to get new number from `Numbers` `CF`. 
3. Then the process withdraws new number from `Numbers` `CF` by fetching the last number which is registered in `Numbers` `CF`, and incrementing it by one. 
4. Then puts that new number into `Numbers` `CF` along with its `PID` as column name. 
5. Releases the `Choosing` lock by changing the `true` to `false` then in its own column (identified by `PID`). 
6. At this point the process waits for all other processes to get their number  again by looking into `Choosing` `CF`.
7. When all process got their numbers it waits for process with lower numbers to leave critical section and remove their PID from Numbers CF. 
8. As we can not guarantee any two processes don't get the same number, we use both chosen number and PID (unique id) to find the higher priority process to enter critical section. 
9. When the process finds that there is no one with higher priority in the list then it will enter the critical section. 
10. Lastly when a process wants to leave critical section simply removes its PID from both `Numbers` and `Choosing` `CF`s. 

> Talk is cheap, show me the code -- Torvalds

## Implementation 
I used `PHP` to implement this concept and [PHPCassa][4] library is used to communicate with Cassandra. The column families in Cassandra (ie. `Numbers` and `Choosing`) must be created with consistency level of QUORUM. Both column families have row key and column name as integers and column values as boolean defined. The `PHP` code is available on [GitHub][2]. 

<div class="ads"> <ins class="adsbygoogle" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> </div> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script>

## Final note 
Well, that's all for now, thank you for reading this up to here :-) and please leave comments if you have any interesting idea about this. All patch, fixes, improvements are welcomed only on GitHub.

## Resources

+ [http://cassandra.apache.org/][3] 
+ [http://wiki.apache.org/cassandra/Locking][4] 
+ [http://en.wikipedia.org/wiki/Lamport%27s_bakery_algorithm][5] 
+ [http://thobbs.github.io/phpcassa/][6]

 [1]: http://www.boynux.com/wp-content/uploads/2013/11/chef-says-okay-clip-art.png
 [2]: https://github.com/boynux/Cassandra-Bakery "Cassandra Bakery lock implementation"
 [3]: http://cassandra.apache.org/ "Cassandra"
 [4]: http://wiki.apache.org/cassandra/Locking "Locking"
 [5]: http://en.wikipedia.org/wiki/Lamport%27s_bakery_algorithm "Wikipedia"
 [6]: http://thobbs.github.io/phpcassa/ "phpcassa"
