---
layout:     post-no-pic
title:      "What makes the best Git commit?"
subtitle:   "Open discussion on how much can be included in a git commit without impeding productivity"
date:       2016-04-20 00:00:00
author:     "Thomas Barthelemy"
tags:       [git]
---

I ran into this great article on
[19 Tips For Everyday Git Use](http://www.alexkras.com/19-git-tips-for-everyday-use/)
that was entertaining a little reflexion on what makes a good git commit message.
I believe this to be a very interesting reflexion that deserve a separate topic.

**So what makes a great commit message?**

Well this is highly limited in quality by another question:

What makes a good commit?
========================

<img alt="commit message meme"
     src="/img/2016-04-20/meme-commit-message.jpg"
     style="float: right;"
 >

That might be trivial to some, but
**if your commit contains 3 features, 2 bug fix and a refactor: the commit message will be bad**
and most likely look something like: `updated stuff`

So the first thing is: limit your commit to one single task (one fix or one feature or one...)

Why would we do that? Well it will make the commit message have more sense, which is where
I'll be going into after but this has a lot of other advantages as well. The major two
I want to highlight here:

**It's Easy to understand the git tree** by looking at it through `git log` or any git tool.
You really don't want to be looking at a never ending list of "changed stuff" and "updated multiple things"
which, let's face it, is totally the same as no commit message at all: you get 0 information from it. 
 
**It's Easy to revert a single change** using `git revert`
Try to revert that failed bug-fix if it was included along other things that modified the same file. Good luck.
 
The perfect git commit message
==============================

Now we know that our commit contains a single task, let's create a great message by including:

 - **The Issue Id**: If you have an Issue Tracker (Github, Jira, Phabricator...) you
 should include the task id, it will help going back to any conversation and details for it.
 some platforms also use it to automatically track time and perform actions. 

 - **Short description of the task**: To have a quick understanding of what this commit
 modified.
 
 - **Why this modification was done**: This is something usually not done but can help
 clarify quite a lot of things in some case. For example:
 
        f123456 Fixes T238 changed encoding from mpeg2video to h264 
        f123456 Fixes T238 changed encoding from mpeg2video to h264 as not supported on mobile
     
The second one is definitely clearer as the reason for the change is sometimes more
important than the change itself to understand the commit. Now let's be honest, this
is not always needed so put a reason when it actually adds value to the message.
 
This is really what needs to be done, now there is a few other things to consider:
 
  - Using `git commit -m "foo"` is cool, and do keep that under 50 characters
  but don't be afraid to add more lines when relevant by either omitting the last `"`
  or by adding extra `-m`:

        $ git commit -m "foo
        > bar"
    
        $ git commit -m "foo" -m "bar"

With the right messages you can also do a bit of bash-fu to generate summary between
releases (tags) for a changelog or see how 2 branches have diverged.
Here is an extract from `Arcanist` repository: 

    $ git log --pretty="%h||%an||%ar||%s" HEAD...HEAD~5 | column -t -s '||'
    a2ab38d    epriestley        4 days ago     Improve performance of `arc branch` in Git with many branches
    737f5c0    Mukunda Modell    11 days ago    Allow amending revisions without commandeering first
    8701e6c    epriestley        3 weeks ago    Strip tips out of commit messages from `arc backout`
    3d7ac86    Aviv Eyal         5 weeks ago    Make callsigns optional in arcanist
    ccbaee5    epriestley        6 weeks ago    When `arc` pushes to the staging area, tell Phabricator what we did

I see the git ninja coming up: Hey, you can use `-5` instead of `HEAD...HEAD~5`. Yes, you
totally can, but this is just to introduce the ease it would be to compare current head
(or master) with latest release tag using `HEAD...v1.0.0` instead.

I admit you might not remember or want to rewrite that, so here are some git-alias version:

    [alias]
        changelog = "!f() { git log --pretty='%h||%an||%ar||%s' $1...$2 | column -t -s '||';}; f"
        changelog = "!sh -c \"git log --pretty='%h||%an||%ar||%s' $1...$2 | column -t -s '||' \""
        
You can also default to `HEAD...` if only one parameter. for example:

    [alias]
        changelog = "!f() { if [[ $2 == '' ]]; then $2 = $1 && $1 = HEAD; fi; git log --pretty='%h||%an||%ar||%s' $1...$2 | column -t -s '||';}; f"

What Else?
==========

Much more to be seen or understood with git, like what to do when your feature branch contains
more than one commit and the joy & pain of `git rebase` and squashing using `git rebase -i`.

Also if you are looking for more git-alias magic you can have a look at my previous article
[Git Config and Bash Profile for Windows](/2015/10/31/git-aliases/) which contains
a lot that works just fine for unix system as well.

**What do you think makes a great commit message ?**
And feel free to share some of your secret git-fu!