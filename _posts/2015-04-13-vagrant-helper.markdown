---
layout:     post-no-pic
title:      "Vagrant Helper"
subtitle:   "Building a OS-Independent CLI tool to avoid having to SSH into your vagrant"
date:       2015-04-13 18:00:00
author:     "Thomas Barthelemy"
---

Our development environment often includes Vagrant machines,
and that means a bit of "Vagrant ssh" to run various development related commands.
Now that's something that bothers me, having to switch from one to the other, and remembering
all the main commands of all the different stacks.

So I tried to make little script that would work for everyone, and we have people on all main OS (OSX, Linux, Windows).
The core restriction I put to myself are the following:

- Works on all OS with Vagrant and Git
- Should not require support for other languages (let's not add requirements)
- Should act as shortcut for main actions of the stack (Here we will take the example of PHP/Symfony2)

What I quickly noted is that Gi Bash would not support all scripting well especially variables, so native shell-style
becomes quickly complicated.

# Finding if we are in or outside Vagrant

The main trick here is that when a developer would run the script, it would actually re-run the script from vagrant.
The vagrant username for our development environment are called "vagrant",
so first step is to check if we are in or out Vagrant:

    #!/bin/bash
    if [[ $USER != 'vagrant' ]]; then
        # Call the script inside vagrant
    else
        # Do what you need to do
    fi

Assuming nobody have a username "vagrant", that will do just find: **we know where we are!**

# Sending parameters 

Most of the commands we need to run inside vagrant take parameters,
let's see what we can do with vagrant SSH:

    $ vagrant ssh --help
    Usage: vagrant ssh [options] [name] [-- extra ssh args]
    
    Options:
    
        -c, --command COMMAND            Execute an SSH command directly
        -p, --plain                      Plain mode, leaves authentication up to user
        -h, --help                       Print this help

The eye goes to the -c to execute a command, let's try that for PHPUnit.
To run the test with the appropriate configuration it would look like:

    if [[ $USER != 'vagrant' ]]; then
        vagrant ssh -c "/vagrant/utils/run_tests.sh"
    else
        /vagrant/bin/phpunit -c /vagrant/app
    fi
    
Well this won't work on Windows has it will wrongly interpret the parameter given to Vagrant,
and will require to specifically run it through bash instead:

        vagrant ssh -c "bash /vagrant/utils/run_tests.sh"
        
We could easily differentiate the running OS to palliate this special case using uname:
 
    if [[ `uname` = *"NT"* ]]; then
        # In windows
     else
        # In Unix
    fi

But that will quickly become exhausting, but the real problem comes with achieving this:

    # On the Dev env
    $ ./utils/run_tests.sh example

    # This will send the parameter to vagrant
    vagrant ssh -c ""
    
    # Which would then run in Vagrant:
    bin/phpunit -c app --group example
    
Sadly vagrant ssh will mess up the arguments and tries to interpret other arguments.
If we look back at the help we can see something interesting:

    $ vagrant ssh --help
    Usage: vagrant ssh [options] [name] [-- extra ssh args]
    
So this allows us to run this from bash directly without using "-c" which will work on all platforms:

    if [[ $USER != 'vagrant' ]]; then
        vagrant ssh -- "bash /vagrant/utils/run_tests.sh $@"
        
With that we get everything we need to do our little scripts.

# Finalizing

A little addition is the -t parameter to SSH which will open a pseudo-tty with the current session,
you will get better formatting and color this way (at least with PHPUnit).

    vagrant ssh -- -t "bash /vagrant/utils/run_tests.sh $@"
    
After that it's up to you to add the logic you want in shell script:

    #!/bin/bash
    if [[ $USER != 'vagrant' ]]; then
        vagrant ssh -- -t "bash /vagrant/utils/run_tests.sh $@"
    else
        cd /vagrant
    
        param="-c app";
    
        while [ "$1" != "" ]; do
            case $1 in
                -c | --coverage )       param="$param --coverage-html web/cov/"
                                        ;;
                -t | --testdox )        param="$param --testdox"
                                        ;;
                -g | --group )          shift
                                        param="$param --group $1"
                                        ;;
                -f | --file )           shift
                                        for file in $( find . -iname "$1*" | grep "Test\.php$" ); do
                                            param="$param --filter $(basename $file .php) $file"
                                            echo "Found matching test file: $file"
                                            break
                                        done
                                        ;;
                * )                     echo 'Invalid paramters, run -h or --help for more information'
                                        exit 1
            esac
            shift
        done
        echo ''
        bin/phpunit $param
    fi

