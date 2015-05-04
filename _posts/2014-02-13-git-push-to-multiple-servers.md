---
layout: post
title: GIT push to multiple servers
excerpt: How to push GIT repository into two remote repository master branches with different commits. Here is a step-by-step guide to git push to multiple repositories.
---

## Intro

![GIT Push to multiple servers][1] 

I was recently working on a project which was hosted on AppEngine and I also wanted to keep on copy of my repository on github, but in my config file I had some secret keys that I didn't want to push to github public repo. One of my friends Milad suggested to keep that config file in a separate branch let's say 'release' and in your master which is supposed to go into github, remove that file from your repo. Yeeees! Sound grate! 

Now I have a branch called release that's going to be hosted in AppEngine and my master goes into github! Essentially git push to multiple servers. If you have a similar problem and you want an easy solution for that, here is a step by step guide to do that. 

<script type="text/javascript" src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" async=""></script>
<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

## 1. Assumptions:

*   Assume that my git origin is already pointing to AppEngine repository and it means that I push my repository directly into AppEngine master branch which is my production server.
*   I have created an empty github repo and it's URL is https://github.com/boynux/myrepo.git

## 2. Adding new remote (ie. github):

    $ git remote add github https://github.com/boynux/myrepo.git

The above command will add new remote called github into your git config. 

## 3. Creating a new branch `release`:

    $ git checkout -b release -t origin/master
    $ git rm --cached config.py
    $ git commit -m "secret keys removed"
    $ git push origin release:master

This will create a new branch called `release` which would be merged with master of github repo. Now checkout into master and remove config or secret files. You may also add it to .gitignore to avoid adding that mistakenly into master again. 

**Note:**  `git rm --cahched config.py` does not remove file history in GIT. Obviously the config.py file is still accessible throughout GIT history. If you want to completely wipe-out this file from your GIT history  please read "[How to remove credentials from GIT history][2]". 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

## 4. Push to github: 
Now it's time to push master into its new home at github: 

    $ git push github master

## 4\. Make it automatic: 

We need to instruct GIT to push right branch into remote master branch based on current working branch. you can do this by providing remote branch in push so for origin (ie AppEngine) & github respectively with this commands: 

    $ git push origin release:master
    $ git push github master:master

But it's little bit inconvenient to every time provide remote, local branch and remote branch let's make it more automatic. I'm going to change default GIT push behaviour to match local and remote branches: 

    $ git branch master --set-upstream github/master
    $ git config push.default upstream

Well, we are done. Now you can simply checkout to release branch do changes and when you are done,  simply type: 

    $ git checkout release
    # do other stuff here
    $ git add <changes>
    $ git commit
    $ git push

Then when you are done checkout to master and merge master with latest release changes and push that into github with the same command: 

    $ git checkout master
    $ git merge release
    $ git commit
    $ git push

That's it.  : -)

[1]: http://www.boynux.com/wp-content/uploads/2014/02/git.png
[2]: http://www.boynux.com/angularjs-apply-explained/ "How to remove credentials from GIT history"
