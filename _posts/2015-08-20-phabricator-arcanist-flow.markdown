---
layout:     post-no-pic
title:      "Simple flow using Arcanist and Differential for code review (Part 1)"
subtitle:   "A step by step flow on how to use Arcanist with git on Phabricator, including pre-push code review. In this first part we will focus on the developer flow"
date:       2015-08-20 12:00:00
author:     "Thomas Barthelemy"
tags:       [phabricator, git]
---

# Arcanist flow for developers

## Cloning the repository
The first time you get on a project, with an existing repository you have to clone it.
All you have to do is git clone using the command provided in the repository page in **Diffusion**.

## First time on a Phabricator
The first time you use **Arcanist** on a **Phabricator** server, you will be asked for a certificate installed with:

    arc install-certificate

This command will ask you for your certificate and provide you will a link to get a new one, so don't worry about that.
The link will ask you to log-in if not already into **Phabricator** and will then display the certificate
that you can copy/past.

You can now use **Arcanist** with this **Phabricator** server.

## What do I have to do?

You can always go in **Phabricator**, in the Homepage, in the **Maniphest** application,
or in a project board to see your assigned tasks.

Another more *geeky* alternative is:

    arc tasks

This will list the tasks assigned to you in the current project (i.e. current git repository),
it displays id, title, status, and priority: all you need to get started!

There are other options you can apply such as `--unassigned` to see all unassigned tasks
or `--limit <max>` to limit the number of tasks displayed.

*Note that by default tasks are sorted by priority*

You can open any **Phabricator** object (task, diff, file...) in your web browser if you have its id:

    arc browse <object id>

## I Got a task, now I want to start!

### Setting the task in progress
First, you can tell other people you are working on the Task:

	arc start <task id>

This also will help track your time spent on the it.

Tasks name start with a `T` so it will be something like `arc start T20`.
*Note that the name is not case sensitive so `t20` will work too.*

### Creating the feature branch 
	
To start a new branch, to work on your task:

	# Let's be sure we are on the master branch
	arc feature master
    arc feature txx_do_something

This will create a branch called txx_do_something,
you can still use `git checkout master` + `git checkout -b txx_do_something` if you prefer, it's the same.

*Note that `arc feature` has an alias called `arc branch`. Same command!*

## How do I commit?

### Chose what to commit

First you need to chose what you will commit:

	git add ./path/to/file

If you have a lot of file you can use an interactive add:

	git add -i
	
*Note: If you want to add everything you may go to the next step directly.*

### Creating a diff

Once you selected what you wanted to commit, it's time for some **code-review** using **Diferential**:

    arc diff

This will ask you if you want to commit (if you haven't) then (if not provided) will ask you for some extra information
such as the reviewers (i.e who will review you code!).
If you wish to diff everything (add everything then diff) you can use `arc diff -a`.

Once this is done you will be given an URL for your diff where the code review will happen.

At this time you are no longer working on the task as you are waiting for the code review so:

    arc stop Txx

You can also use `arc stop` with no parameters to stop progress on all tasks.

### Reviewer said my code is bad :(

To update your diff in order to fix the comments from the reviewer you can just do your modification and then use
`arc diff` again. This will automatically update your existing diff.

Don't forget to `arc start Txx` and `arc stop` when you work on a task!

## Going for another task

### See what's up

**Arcanist** provides a few commands to help you see what's going on:

    # Lists all your diff with their status (needs review, accepted...)
    arc list
	
	# List all your on-going feature (i.e branch) and their latest diff status
	arc feature

### My diff is accepted!

Once your diff is accepted, you can land that feature into master:

	## let's go on the right branch
    arc feature txx_do_something
	## And merge/push
	arc land

This will merge the current branch on master, delete the feature branch and push master to the remote phabricator.

## Closing your tasks

Once your feature is landed, you can close your task:

    arc close txx
	
This will be done automatically if your commit message (during `arc diff`) contains `fixes txx`

# Please Help!

Remember you can always do:

	arc help
	arc <command> --help
	
To get plenty of information on what you can do!
