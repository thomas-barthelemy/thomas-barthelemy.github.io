---
layout:     post-no-pic
title:      "On Setting Up Development Environments"
subtitle:   "Some thoughts on today's way to set up a development environment quickly and efficiently"
date:       2016-08-09 00:00:00
author:     "Thomas Barthelemy"
tags:       [linux, docker, vagrant, windows]
---

Development What?
=================

In software development we always have dependencies that needs to be
set up, whether it's directly part of the stack you are working on
or an underlying dependency.

You need an Apache2 and MySQL server here,
an NGINX and PostgreSQL server there... Soon enough you computer is
running a bazillion things concurrently and if you are lucky it will
just take some resources but sooner or later apache will try to listen
on the same port as NGINX or any other collision.

Sure you could totally be sure nothing starts automatically and start it
when needs be but then you will need a PHP5 here and PHP7 there... now
it gets a more annoying to solve.

Let's see what have been commonly used solutions.

Juggling Manually
=================

As mentioned above, you can try to isolate all your stacks and ensure
they don't bind on the same ports. This means not using a globally
installed bin of your app. No `apt-get install` for you!
I see most of your ran away to the next section already, I will
also add that this is not the nicest way to share development
environments.

If you ever have a need to do this, for example setting up 2 XAMP stack
side by side without having them collide I'd strongly advise to look at
the [Bitnami Infrastructure Stacks](https://bitnami.com/stacks/infrastructure)
that provide very nice installers to achieve that.


Self Contained, the lucky few
=============================

There are a lucky few that are mostly self contained already, mostly
javascript based that aren't too picky on their version.

With a globally installed `NodeJS` / `NPM` / `grunt` a developer can
survive and handle a bunch of projects, but that does require you to
work on those stacks although sooner or later someone in the team will
get the brand new NodeJS that breaks it all.

Another noticeable technology that won't trouble you much are platform
specific development that already support targeting different version
like `.NET`, `Java with Android`, `iOS`... If you are in that case,
you might not appreciate how much those tools do for you.

Virtualization
==============

One direct logic thought is Virtualization with tools like `VirtualBox`,
`VMware` or `HyperV`. This is a very neat approach in many ways,
you have one Virtual Machine (VM) for each stack, you are free to set
up everything you need on it and it won't impact any other stack.
Although it's an old solution, it's still one of the most used solution.

Configuring the VM
------------------

A common problem is to actually get the environment up, including the
VM configuration (shared folders, forwarded ports, allocated ram).
 It's definitely OK to do it once, but you definitely want to do it
manually every time you start a new project and/or for every team member.
 
**Vagrant** and its successor **Otto** are amazing tool that will let you set up a configuration
file (`Vagrantfile`) in the root of your project with all the information
needed to set up your box like the VM config or the OS.

You set it up once and then all you will have to do is a `vagrant up`.
The first time it will download a base box of your choice that will
then be reused across project.

While Vagrant supports basic provisioning, it's harder to use it with
a more complicated flow so plenty tools can come to the rescue.

**Ansible**, **Chef**, **Salt** and **Puppet** are all nice tools
originally created to help set up and maintain large number of servers.
All can be used for smaller orchestration scenario like setting up your
VM with the right tools. I personally very much enjoyed Ansible as it's
fairly simple and is natively compatible with probably all the tools
you might need.

Note: Windows isn't supported as the control machine with Ansible.

Pros
----

 - Allows you to run Windows env in Linux and vice-versa
 - Easy to scale
 - VirtualBox, Vagrant and Ansible are all free tools
 - VM are totally independent from each other, no collision between stacks
 - Find or Make your own base box with your stack already set up
 
Cons
----

 - VM consume a lot of resources, beef up your ram and disk space
 (especially noticeable if you are dealing with micro-services)
 - The provisioning (all the `apt-get` for example) will happen for each
  team member or new project. Consider an `apt` cache within your network
  if you expect heavy traffic because of that.
 - Share folders are slow, very slow. Reading and writing in a share folder
 are not as efficient as a native file system, if you want more details
 on that I'd recommend [Mitchell Hashimoto comparison on VM filesystem performances](http://mitchellh.com/comparing-filesystem-performance-in-virtual-machines).
 - Boot time, you will need to wait for the VM to start each time, not
 a major issue though.
 
Containers
==========

You haven't been on the internet in the last 2 years if you haven't
heard anything about `Containers` or the most famous tool of the market:
[Docker](https://www.docker.com/what-docker).

While I'm not here to discuss why comparing a container to a VM is dangerous,
I can still compare those as development environment tools.

Docker offers operating-system-level virtualization, which solve the issues
mentioned in the VM part.

Pros
----

 - No extra resources, docker itself usage is minimal
 - Native filesystem, no slow down anywhere
 - Make or find your own docker image with what you need
 - No boot time
 - Very active community, you will find countless docker images in docker hub.

Cons
----

 - Docker won't get you cross-platform, if you are on Unix, you will have
to run Unix-based container
(doesn't have to be the same version or distribution).
 - Docker is mainly a Linux tool, Docker for Windows is based on VMs as well
 - Docker requires a 64-bit installation regardless of your Ubuntu version.
 Additionally, your kernel must be 3.10 at minimum.
 The latest 3.10 minor version or a newer maintained version are also acceptable.
 
Note: Vagrant supports docker as well!
 
SaaS IDE
========

This a little special category, as it's based on the previous solution
but provided as a SaaS.

There have been quite a few nice tools on the market in the past years,
the most noticeable for me are [Cloud9 IDE](https://c9.io/) and
[Eclipse Che](http://www.eclipse.org/che/).
 
Those gets you your environment in the cloud, already isolated.

Pros
----

 - Native experience
 - No setup trouble
 - No resource trouble
 - Can code as long as you have internet and a web browser.
 
Cons
----
 - You won't get far without an internet connection
 - You rely on a 3rd party or need to deploy it yourself
 - The same IDE, for everybody (although you could definitely use `vim` too)
 
 
Conclusion
==========

We've seen quite a few tools I used along the way and there is definitely
not unique answer. Docker on Unix based environment haad the best result
for me, vagrant and Ansible are my go-to for Windows/OSx Machines.
Feel free to have a look at my [Symfony backend boilerplate](https://github.com/thomas-barthelemy/symfony-backend-boilerplate)
showing both the Vagrant-Ansible combo and the Docker environment.

What are the tools that you use to save your time when setting up
development environment?

