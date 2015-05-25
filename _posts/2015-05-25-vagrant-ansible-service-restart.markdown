---
layout:     post-no-pic
title:      "Ansible - FATAL: all hosts have already failed"
subtitle:   "When restarting a service with Ansible and Vagrant just won't work because 'All hosts have aready failed'"
date:       2015-05-25 16:03:00
author:     "Thomas Barthelemy"
tags:       [vagrant, linux, ansible]
---

I finally had a little time to "fix" a bug that recently shown up:

When trying to restart a service during a vagrant provisioning with ansible on a Windows host, I was given the following error:

    ==> default: NOTIFIED: [php | restart PHP-FPM] *********************************
    ************
    ==> default: failed: [default] => {"failed": true}
    ==> default:
    ==> default: FATAL: all hosts have already failed -- aborting

Some service would restart correctly while some other just won't work,
and restarting the service manually would work just fine.

After a quick investigation it seems to be more of a Vagrant issue than anything else so a quick
fix would be to replace:

    - name: restart PHP-FPM
    service: name=php5-fpm state=restarted

By:

    - name: restart PHP-FPM
    command: service php5-fpm restart
    

