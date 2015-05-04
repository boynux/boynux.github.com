---
layout: post
title: How to remove credentials from GIT history!
excerpt: Removing files from your GIT branch does not make them inaccessible throughout your GIT history. If you want to remove credentials from GIT read on.
---

If you happen to read my other article about **[GIT push to multiple servers][2]** you might noticed that if you have your secret files pushed in your GIT at first place, just removing those files from your branch does not make them inaccessible throughout your GIT history. 

Reviewing the GIT history can easily reveal those information. If you could change your secret information and create new ones, then you are fine. But what if you can't really change those information? Here I explain how to alter your GIT history to get rid of those footprints. <h3 style="display: inline-block;">
  So, let's purge credentials from GIT history.

[<img class="size-medium wp-image-720 alignright" style="margin-left: 20px; margin-right: 20px;" title="remove credentials from GIT" alt="git-alter" src="http://www.boynux.com/wp-content/uploads/2014/02/git-later-300x108.png" width="300" height="108" />][1]</h3>

<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>
<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

_WARNING: ALTERING ANY RCS (INCLUDING GIT) IS HIGHLY DISCOURAGED BECAUSE IT KILLS THE PURPOSE OF RCS, I HIGHLY RECOMMEND TO CHANGE YOUR CREDENTIALS IF POSSIBLE AND LEAVE YOUR RCS HISTORY INTACT._

## How is it possible?
GIT has a command called 'filter-branch'. This powerful tool allows you to rewrite your GIT history. I'm going to use this tool to alter GIT history and remove all references to my secret files. 

## `filter-branch` in action:

`--index-filter` argument to `filter-branch` causes rewrite history for all indexes in GIT. 

The actual command to run over every index stored in GIT is a parameter to this argument. We are going to use '`git rm --cached --ignore-unmatch secrets.py`'. No need to say, you need to replace '`secrets.py`' with your secret files name(s) that your are going to remove. 

To ignore empty commits that filter-branch may create we can use '`--prune-empty`'. 

To maintain tags we can use '`--tag-name-filter`' argument with 'cat' command. This simply updates those tags pointing to a rewritten object. 

And finally for revision list we use '`--all`' to alter history for all revisions.

## Put it all together:

    git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch secrets.py' --prune-empty --tag-name-filter cat -- --all

## Final notes: 

By using '`filter-branch`' you can alter your GIT history. Common use case of this command is to remove credentials from your GIT history. But please keep in mind that if there are such files renamed in your GIT history it's your responsibility to find them. One can use 

    git log --name-only --follow -- secrets.py

to find such a renames/moves. 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

Reference: 

* GIT man page. (`man git-filter-branch`)

[1]: http://www.boynux.com/wp-content/uploads/2014/02/git-later.png
[2]: http://www.boynux.com/git-push-to-multiple-servers/ "GIT push to multiple servers"
