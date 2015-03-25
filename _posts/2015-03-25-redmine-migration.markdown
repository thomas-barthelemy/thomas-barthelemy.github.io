---
layout:     post-no-pic
title:      "Let's move Redmine"
subtitle:   "When you just want to change everything!"
date:       2015-03-25 15:25:00
author:     "Thomas Barthelemy"
---

Some of the upcoming changes for me at work pushed to review all processes and infrastructure
of the company that falls into my current responsibilities and to ensure the knowledge can be transferred properly.

On this week nice adventure, I decided to have a clean, up-to-date instance of Redmine that would better fit in the current company infrastructure.

# Current situation

Our Redmine server was firstly set-up on an available Windows Computer, and kept evolving from there.

So the starting point was:

- Windows Server 2008 R2 (The version does not actually matter)
- Redmine 2.5 (Stack was installed from the [Bitnami](http://bitnami.com/ "Bitnami") native installer)

# The target

Now having a racked-server in our building data-center running VMs and docker apps, that was definitely our new host.
Our Redmine instance running quite a few plugins, upgrading to Redmine 3.0 wasn't an option for compatibility issue,
so the target was:

- Ubuntu 12.04 Running VirtualBox and Docker
- Redmine 2.6
- Updated version of our plugins
- New plugins

Also, quite a few plugins where added from the beginning, not all still in use so some plugins would have to be removed as well.

So new OS, new Version, Update plugins, Missing plugins, new plugins... hopefully we did not change the Database too.

# Let's do it

All the following assume you are using a bitnami stack deployed under:

    /opt/bitnami/apps/redmine/

With the Bitnami stack manager scrip available here:

    /opt/bitnami/ctlscript.sh
    
If it's you aren't using a Bitnami Redmine Stack then you will need to start and stop the services using:

    
    
## The new Redmine
I got the latest 2.X VM of the Redmine stack from [Bitnami](http://bitnami.com/ "Bitnami"),
created the VM from the provided hard drive.
I also set up the necessary port redirections (in my case the 80 for the web and 22 to be able to SSH) as this VM need to be accessible from the whole network.

Bitnami stacks require to activate the SSH server:

    sudo mv /etc/init/ssh.conf.back /etc/init/ssh.conf
    sudo start ssh

Also there is an info page activated by default that will show up at the bottom-right of the screen, this needs to be removed:

    sudo /opt/bitnami/apps/redmine/bnconfig --disable_banner 1
    sudo /opt/bitnami/ctlscript.sh restart apache

Yay, a working Redmine stack!
    
## The plugins

This was a rather easy step, just listing the plugins that are still necessary, then finding their last stable version (if-any) that are compatible with Redmine 2.6.
As this Redmine might not be the only one we have to deploy I decided to group the plugins we use in a [Github repository](https://github.com/we-bridge/Redmine-Template)
so it could easily be shared and updated when needed.

The repository makes use of git sub-modules (.gitmodules) whenever possible so that code already publicly available is not pushed again in this repository.

Ok so now is a good time to check if the plugins works well together, or if they require some changes
(See how to install those plugins in the [Redmine-Template Github repository](https://github.com/we-bridge/Redmine-Template)).

## Exporting the data an database

All the uploaded files are located in the "files" folder of Redmine, so we need to copy that over to the new server. Easy-peasy.

Then export the database:

    mysqldump -u root -p bitnami_redmine > backup.sql
    
For the Redmine bitnami stack the database name is bitnami_redmine, change it to whatever is your Redmine database name.

## Importing files an database

First, I would advise to stop the whole stack before doing anything:

    sudo /opt/bitnami/ctlscript.sh stop

As said above, the "files" directory needs to be copied to the new server, you know how to copy so I won't expend on that uh.

Then before restoring your database... Start you database, then restore!

    sudo /opt/bitnami/ctlscript.sh start mysql

Create a clean new database:

    mysql -u root -p 
    Password: ****
    mysql> drop database bitnami_redmine;
    mysql> create database bitnami_redmine;
    mysql> grant all privileges on bitnami_redmine.* to 'bn_redmine'@'localhost' identified by 'DATABASE_PASSWORD';

And import the backup:
    
    mysql -u root -p database_name < backup.sql

Then we need to migrate the database to its latest version:

    cd /opt/bitnami/apps/redmine/htdocs
    ruby bin/rake db:migrate RAILS_ENV=production
    
Finally let's run the plugins migration:

    rake redmine:plugins:migrate RAILS_ENV=production
    
## Restarting the stack

    sudo /opt/bitnami/ctlscript.sh restart
    
# Troubleshooting

I was lucky enough to have minimum issues
but it might not be your lucky day so if the Redmine website gives you some error after that,
check the redmine logs under

    /opt/bitnami/apps/redmine/htdocs/log/production.log
    
This will help you understand what is happening hopefully.

# Disclaimer

I obviously would encourage people to minimize the number of changes done at once,
changing that many things (OS, Redmine Versions, Plugins) at once is likely to fail if you
haven't planned properly.
Furthermore, if you use more than the top 5 most popular plugins and plan to migrate to Redmine 3:
There is a high chance that the plugins you use aren't compatible with a newer version of Redmine.