---
layout:     post-no-pic
title:      "Running PHPUnit Build with HarborMaster and DryDock"
subtitle:   "How to set-up basic CI steps in Phabricator using Docker, HarborMaster and DryDock"
date:       2015-09-11 16:24:00
author:     "Thomas Barthelemy"
tags:       [phabricator, git]
---

# The basics

 - **DryDock** is a Phabricator application to "Allocate Software Resources", for you it means it will contain the information to reach the different servers used during the CI by HarborMaster.
 - **HarborMaster** is the Build/CI application of Phabricator allowing you to define build-plan that can then be applied to one or more repositories.
 - **Docker** will be used to create fresh build environments, if you don't know about Docker it's advised you start [here](https://www.docker.com/whatisdocker)

# Setting up DryDock

The first step is to tell Phabricator what server will be used to run your build, for that you need to:

 - Create a credential in **Passphrase** for your server (SSH key or password) and note its id (We will say it's `K2000`).
 - Create a **Blueprint** in **DryDock** of type `Working Copy` and note its id (We will say it's `42`)
 
 Once this is done, you will see that your **Blueprint** is said to have "no resources attached".
 It is not possible to create a resource from the web interface so you will need to run the following command:
 
    ./bin/drydock create-resource --blueprint 42 --name localhost --attributes host=localhost,platform=linux,remote=true,port=22,path=/var/drydock,credential=2000

What are we really doing here?

 - `./bin/drydock create-resource`: We want to create a new resource
 - `--blueprint 42`: for the blueprint with id = 42
 - `--name localhost`: The name of the created resource will be `localhost`
 - `--attributes`: The resource configuration
   - `host=localhost`: The IP or hostname of the server
   - `platform=linux`: The type of server (`windows` or `linux`)
   - `remote=true`: Whether it's a remote server or not
   - `port=22`: The port that will be used to SSH into the server
   - `path=/var/drydock`: Where the build will happen
   - `credential=2000`: The **Passphrase** credential id used to SSH into the server
 
 At this point your **DryDock Blueprint** should proudly display the newly created resource.
 
# Setting up a Docker to run your builds

Here is a sample Docker used to run our PHPUnit builds:

[github.com/thomas-barthelemy/docker-phpunit-harbormaster](https://github.com/thomas-barthelemy/docker-phpunit-harbormaster)

There are 2 main things here to keep in mind:

 - The `Dockerfile` have to contains everything that needs to always be in the environment such as the `apt-get install`
 - The `Entrypoint.sh` that does everything that is build-dependent such as running phpunit, but also the migrations, composer install...
 
 You can easily modify this docker to fit other needs, or simply use one of the many other availables.
 
# HarborMaster

Everything is ready for you to create your first **Build Plan*:

 1. Go in **HarborMaster**
 2. Click on `Manage Build Plan`
 3. Click on `New Build Plan` and create your new build plan

To run command on your server you first need to `Lease` an host,
for that add a `Build step` of type `Lease Host`:

 - `Name`: Give it a friendly name, you will need it later
 - `Depends On`: Leave that empty if it's your first step
 - `Artifact Name`: The name of your drydock resource (The one you used for `--name=`)
 - `Platform`: `linux` or `windows`
 
 Once you have that done you are ready to create steps of type `Run Command`.
 Those command will be ran in `$path/$buildId` where 
 `$path` is the directory passed with the `path=` parameter and
 `$buildId` is the current build id.

# Considerations

Right now **DryDock** is in a very early stage and a little extra leg-work might be necessary,
for example our flow is based on **Differential** code review rather than **Audit** code review
which means it can not directly be "checked out".
For that reason the following commands are set right after the lease:

    git clone ${repository.uri} ./
    ~/.arc_install/arcanist/bin/arc patch D${buildable.revision}
    
    docker run -v $(pwd):/code/ -v /home/phab-srv/.composer/cache/:/root/composer/cache/ webridge/phpunit-xxxx

It might be easier to user existing CI system with existing plugins for Phabricator like Jenkins,
but this way offers more flexibility and an incredible view of the unlimited possibilities of the
Docker - HarborMaster - DryDock combo!
