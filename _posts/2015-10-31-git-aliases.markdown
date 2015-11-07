---
layout:     post-no-pic
title:      "Git Config and Bash Profile for Windows"
subtitle:   "My git configuration (.gitconfig) and aliases and Mingw profile (.bash_profile)"
date:       2015-10-31 02:25:00
author:     "Thomas Barthelemy"
tags:       [git]
---

I've been wanting for a while to update and share my Git config and bash profile for Windows, so here it is!

# Bash Profile

I use Git bash which is now Mingw64 with Mintty, It's have been behaving the best for me so far with the best performances.
For those wondering, I rarely recommand Cygwin but for those needing that level of features I'd advise looking at [Babun](http://babun.github.io/)
which is a customized version of Cygwin that rocks.

So, back to my bash profile now!

    winpath() {
        if [ ${#} -eq 0 ]; then
            : skip
        elif [ -f "$1" ]; then
            local dirname=$(dirname "$1")
            local basename=$(basename "$1")
            echo "$(cd "$dirname" && pwd -W)/$basename" \
            | sed \
              -e 's|/|\\|g';
        elif [ -d "$1" ]; then
            echo "$(cd "$1" && pwd -W)" \
            | sed \
              -e 's|/|\\|g';
        else
            echo "$1" \
            | sed \
              -e 's|^/\(.\)/|\1:\\|g' \
              -e 's|/|\\|g'
        fi
    }

This is a little helper method that is not necessary for people with Cygwin,
that will produce windows style path from the git bash style when `pwd -W` is just not enough.
I rarely use it directly, so here is why:

    function explorer()
    {

        if [[ $@ == '' ]]; then
            explorer.exe "$(winpath ./)"
            return
        fi

        explorer.exe "$(winpath $@)"
    }

This allows me to `explorer` to get an explorer in the current directory,
or `explorer /path/to/something` to get an explorer on a specific directory. This works
with all types of path, even things like `~/../xxx`.

The same idea for Notepad++:

    function npp()
    {

        if [[ $@ == '' ]]; then
            /c/Program\ Files\ \(x86\)/Notepad++/notepad++.exe &
            return
        fi

        /c/Program\ Files\ \(x86\)/Notepad++/notepad++.exe \
            "$(winpath $@)"
    }

And finally a few details I added for Arcanist (Phabricator's CLI):

    export TERM=cygwin
    export EDITOR=vim
    source "C:\tools\arcanist\resources\shell\bash-completion"

Note that I installed gvim and I'm mostly not using the one provided by git-bash.

# Git Aliases

## The easy: No explaination needed

    s = status
    b = branch
    cm = commit
	co = checkout
    tags = tag -l
    branches = branch -a

## The Medium: eeh, I see

    bd = "!git branch -d $1 && echo '' && git branch -l && echo '  (Removed) $1'"
    bdf = "!git branch -D $1 && echo '' && git branch -l && echo '  (RemoveD) $1'"
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --
    squash = "!git rebase -i HEAD~$1 && echo''"
    pruno = fetch origin --prune

 - **bd** and **bdf** will respectively delete and force delete a specified branch, list the remaining branches and mentioning the one deleted.
 - **lg** shows a git log with colors and visual branches tree representation, kinda cool
 - **squash x** Squashes interatively the X latest commits from HEAD
 - **pruno** is a simple `fetch --prune` on master to remove remote deleted branches from our local cache
 
 # The Hard: Bro, do you even git?
 
    cleanup = "!git branch --merged | grep  -v '\\*\\|master\\|develop' | xargs -n 1 git branch -d"
    cleanup-origin = "!git branch -r --merged | grep -v master | sed 's/origin\\///' | xargs -n 1 git push --delete origin"
    go = "!git checkout master && git pull origin master && git pruno && git cleanup"
    go-stash = "!git stash && git go"
    go-safe = "!git go-stash; git stash apply"
 
 - **cleanup** deletes all local branches that have already been merged in your master, this includes new branches with no changes.
 - **cleanup-origin** (DANGER!) deletes all remote branches that have been already merged in your master, this also includes new branches with no changes.
 - **go** Will go back to master and will `pruno` and `cleanup`, this is great to go fresh before starting a new branch.
 - **go-stash** will do a `git stash` before the `go` but won't re-apply it after.
 - **go-safe** will do a `go-stash` and re-apply the stash after (`git stash apply`).