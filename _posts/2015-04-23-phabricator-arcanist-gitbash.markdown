---
layout:     post-no-pic
title:      "Using Arcanist CLI tool with Git-Bash"
subtitle:   "Using the Phabricator Arcanist CLI tool on Windows with ANSI colors and auto-completion"
date:       2015-04-23 19:38:00
author:     "Thomas Barthelemy"
tags:       [windows, phabricator]
---

I've been recently playing around with Phabricator to plan a deployment for the company and one of the big
piece that comes along with Phabricator is the Arcanist Command-Line interface (CLI) that allows to interact
with your Version Control System (VCS) like Git, Mercurial or SVN.

This tool comes as a "beta" for Windows and is announced to be made to work with git-bash, the shell-style console
based on mingw64 installed with Git for Windows.

# How to install Arcanist on Windows

Installing Arcanist requires a bit of work, you will need:

- PHP
- Git
- Arcanist
- libphutil

## PHP
Installing PHP on windows in pretty straightforward, the method is up to you but I like using
[Chocolatey](http://chocolatey.org/ "Chocolatey") which sums it up to:

    choco install php
    
Then you need to enable the PHP Curl extension, in the PHP Directory:

 - Rename or copy php.ini-development to php.ini
 - Set the extension_dir to your PHP ext folder (php\ext)
 - Uncomment the ;extension=php_curl.dll by removing the **;**

Finally make sure the PHP bin is in your PATH Environment variable,
then in a new console you can check that PHP is found coorectly:

    $ php -version
    PHP 5.6.3 (cli) (built: Nov 12 2014 17:19:35)
    Copyright (c) 1997-2014 The PHP Group
    Zend Engine v2.6.0, Copyright (c) 1998-2014 Zend Technologies
    
## Git
Git for Windows is a simple installer as well, again you may use Chocolatey:
    
    choco install git
     
Again, make sure it works and try your new git-bash around:

    $ git --version
    git version 1.9.5.msysgit.0
    
## Arcanist and libphutil

Arcanist and libphutil need to be cloned from a git repository (already using git!),
so create a directory for those tools (e.g. c:/utils) then clone the 2 repositories inside:

    $ cd c:/utils 
    $ git clone https://github.com/phacility/libphutil.git
    $ git clone https://github.com/phacility/arcanist.git
    
You should then have the 2 directories in your utils folder.
Then add your arcanist/bin folder to your PATH environment variable (just like PHP).

Finally you can check that everything works from you Git-Bash:

    $ php --version
    PHP 5.6.3 (cli) (built: Nov 12 2014 17:19:35)
    Copyright (c) 1997-2014 The PHP Group
    Zend Engine v2.6.0, Copyright (c) 1998-2014 Zend Technologies
    
    $ git --version
    git version 1.9.5.msysgit.0
    
    $ arc
    Usage Exception: No command provided. Try `arc help`.

# Adding Tab Completion

One useful thing to have is tab completion: pressing tab will tell you what are your possibilities with Arcanist,
but this feature is not set-up by default and requires a small change.

First find you **.bash_profile** file, which should be under:

    C:\Users\XXXXX\.bash_profile

If the file does not exists, you may create it using your console (Git-bash for example):

    $ touch c:\Users\xxxx\.bash_profile

Then assuming you cloned Arcanist in *C:\\utils\\*, add the following line to it:

    source "C:\utils\arcanist\resources\shell\bash-completion"

You can then start a new git-bash console,
go into a git repository (arcanist folder for example), then type "arc " + TAB.
You should see something like:

    $ arc
    alias           diff            lint            tasks
    amend           download        linters         time
    anoid           export          list            todo
    backout         feature         paste           unit
    branch          flag            patch           upgrade
    browse          get-config      revert          upload
    close           help            set-config      version
    close-revision  land            start           which
    cover           liberate        stop

# Fixing Ansi colors

It seems that for some people the Ansi color is not escaped properly and thus appear like this:

    $ arc
    ←[1mUsage Exception:←[m No command provided. Try `arc help`.
    
This has been reporter here: [T5724](https://secure.phabricator.com/T5724 "T5724")
There are two ways to fix this:

## Removing the Ansi colors:

The simple and quickest solution is to remove the Ansi coloring completely.
Arcanist will only send colors if your TERM is compatible so you can:

Remove the Terminal type completely:

    unset TERM

Or Set to another Terminal type:

    export TERM=xterm
    
The default TERM value for Git-bash should be **cygwin**.

## Hack to keep the Ansi colors:

I really wanted to keep the Ansi coloring so I realised the following:

The issue lies in the PHP output and not in the Terminal,
so passing the result through a terminal command would solve the color:

    $ arc
    ←[1mUsage Exception:←[m No command provided. Try `arc help`.
    $ arc|cat
    Usage Exception: No command provided. Try `arc help`.

One hack to fix it is to customize your bash profile with a little function that will override the
behavior of the **arc** command:

    function arc(){
        command arc $@|cat
    }

**EDIT**:
More recent versions of phputils use the error output as well which needs an extra addition in order
to be caught by our cat:

    function arc(){
        command arc $@ 2>&1|cat
    }

## Completion error on non-git folders:
 
Finally I noticed than trying to tab-complete on a non-git folder would output the following:

    $ arc sh.exe": --trace: command not found
    
    ^[[1mException^[[m
    ...

Which is the result of the following error:

    $ arc shell-complete
    Exception
    Argument 1 passed to idx() must be of the type array, null given, called in C:\u
    tils\arcanist\src\workflow\ArcanistWorkflow.php on line 618 and defined
    (Run with `--trace` for a full exception trace.)

To get rid of that I modified the bash-completion file (the one added as source) to check for 
an error on this line specifically

    ...
    OPTS=$(echo | arc shell-complete --current ${COMP_CWORD} -- ${COMP_WORDS[@]})
    
    if [[ $OPTS == *"ArcanistWorkflow.php on line 618"* ]]; then
        return 0
    fi
    ...

Although this is a quick hack that isn't so clean,
it will probably break itself if the file is updated which isn't a bad thing actually.

# Some useful links

- [Arcanist User Guide](https://secure.phabricator.com/book/phabricator/article/arcanist/)
- [Arcanist User Guide: Windows](https://secure.phabricator.com/book/phabricator/article/arcanist_windows/)
