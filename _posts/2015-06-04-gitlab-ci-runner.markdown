---
layout:     post-no-pic
title:      "Creating a GitLab-CI Runner with docker"
subtitle:   "The essential on how to quickly set up GitLab-CI Runner with Docker"
date:       2015-06-04 13:27:00
author:     "Thomas Barthelemy"
tags:       [linux, git, docker, other]
---

I have been looking for a quick way to create a runner for GitLab-CI,
here is how I ended up doing it:

# The base docker
First, we will start from Sameer Naik GitLab-CI Runner docker:

    docker pull sameersbn/gitlab-ci-runner:latest

# Configuring the Runner

## Mapped Folder
You will need a mapped folder to store the configuration, so let's create one:

    mkdir -p /opt/gitlab-ci-runner

*Note: The directory path does not matter, if you chose another one just change it
accordingly in the mapping (-v) below*

## Runner Setup
A GitLab-CI Runner requires your GitLab URL and a Token, those are provided in your 
GitLab-CI Project Runners page.

Hopefully setting that up is fairly easy:

    docker run --name gitlab-ci-runner -it --rm \
        -v /opt/gitlab-ci-runner:/home/gitlab_ci_runner/data \
      sameersbn/gitlab-ci-runner:latest app:setup

*Note: you might need to `sudo` that command*
A little explanation on the command parameters:

- `--name` Gives a name to your container, we will need that later
- `-it` will give us an interactive TTY
- `--rm` will automatically remove the container if it exists
- `-v` mounts a volume
- `app:setup` will run the GitLab-CI setup script

This will ask you for the URL and Token to add your Runner.
Once completed, you should see the Runner showing up on GitLab-CI

## Accessing the container
Your runner might be "ready" you still need to install all the requirements for whatever
script will be running (maybe you need PHP, Python, pip...), and you might already know 
that
[installing a SSH Server on a Docker container is just not the way to go](https://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/).

So what to do? Easy:

    docker exec -it gitlab-ci-runner bash

*Note: This will only work for docker version `1.3.0` or higher.*

### Running the Runner

Once all your CI runner requirements installed, you have to run the GitLab-CI runner 
script before you exit the container:

    root@GitLab-ci-runner:/home/gitlab_ci_runner/gitlab-ci-runner/bin# nohup ./runner &
    [1] 117

You can now exit the container, you runner is good to go.

# What's left to do?
Everything that is left to do is in your GitLab-CI.

## Defining a job
You have to define your job (i.e. What will the runner actually do) in the GitLab-CI 
Job section of your project. It will run in the root of a freshly cloned version of 
your commits/tags so no need to worry about paths.

## Activate Testing in GitLab for the project
In GitLab you need to go in your project `Settings -> Services -> GitLab CI -> Test 
Settings` to be sure everything is ok.

And that should be it!

# Start / Stop the container

    docker stop gitlab-ci-runner
    docker start gitlab-ci-runner
    docker restart gitlab-ci-runner

# Other considerations
As it is, your container is not easy to scale, backup, share...
One possibility is to (once your requirements are set up) to use `docker commit`
and then export the container in an archive.

If possible you can also do the requirements install using a `dockerfile`, that will make
your runner incredibly easy to re-create should the need arise.







