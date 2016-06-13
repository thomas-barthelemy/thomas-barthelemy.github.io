---
layout:     post-no-pic
title:      "Creating and keeping an SSH Tunnel open"
subtitle:   "How to create an SSH Tunnel to use as a SOCKS4 proxy and keep it open"
date:       2016-05-02 00:00:00
author:     "Thomas Barthelemy"
tags:       [linux]
---

The tl;dr
=========
Accessing `REMOTE_HOST:REMOTE_PORT` through an SSH TUNNEL to `SSH_HOST`:

    autossh -M 0 -f -q -N -L LOCAL_PORT:REMOTE_HOST:REMOTE_PORT SSH_USER@SSH_HOST

Adding a Reverse Proxy Into it (Apache2):

    <VirtualHost *:LISTEN_PORT>
            ServerName foo.LOCAL_SERVER_DOMAIN
    
            ProxyPreserveHost on
            ProxyRequests off
            ProxyPass / http://localhost:LOCAL_PORT/
            ProxyPassReverse / http://localhost:LOCAL_PORT/
    </VirtualHost>

Background
==========
Having to work heavily with resources in China we often have to bang our heads on the
randomness of the Great Firewall of China and its other more unknown side:

It's commonly agreed that accessing outside-of-china websites from China may be complicated
but it also works the other side and some of our 3rd party services, for Chinese clients,
are logically hosted in China.

Recently we became unable to access a specific provider which would impact the development
team outside of China, so what can be done?

On a long run, having a business VPN is definitely a fair solution but here the scope of
the issue was so limited that we decided to just hack a little solution by using a 
SSH tunnel, SOCKS4 proxy and an Apache reverse proxy.

Basic SSH Socks Proxy
=====================
Creating a basic SSH tunnel to use as a Socks4 proxy is quite easy:

    ssh -p SSH_PORT -D PROXY_PORT -g SSH_USER@SSH_HOST

This is great to use it as an all-purpose proxy in your browser,
but if we want to map one vhost to a remote one through a SSH tunnel it won't be enough.
For that we need that tunnel to go to a specific HOST:PORT

Creating host to host SSH Tunnel
================================
We have 2 Servers involved (I'll use names instead of actual IP):
 - `LOCAL_SERVER`: Our local server (outside of China)
 - `CN_SERVER`: Our server in China

A Remote service we can't access outside of China:
 - `CN_SERVICE`
 
And we want to be able to have a flow looking like:
    
    LOCAL_MACHINE <-> LOCAL_SERVER <-> CN_SERVER <-> CN_SERVICE

We assume that:
 - `LOCAL_MACHINE` is on the same network as `LOCAL_SERVER`
 - `LOCAL_SERVER` can SSH to `CN_SERVER`
 - `CN_SERVER` can access `CN_SERVICE`

So that makes opening the SSH tunnel as:

    ssh -p SSH_PORT -fnL LOCAL_PORT:CN_SERVICE:CN_SERVICE_PORT CN_SERVER_USER@CN_SERVER -N

So accessing `http://LOCAL_PORT:LOCAL_SERVER` will tunnel transparently to `CN_SERVICE:CN_SERVICE_PORT`.

Keeping Tunnel Open
===================
Now the tunnel is open but might get closed automatically as the session ends after a while
(depending on configuration) if it remains inactive or the connection is lost.

For that we used `autossh` that can be installed with:

    # Debian / Ubuntu
    apt-get install autossh
    
    # CentOS / Fedora / RHEL
    yum install autossh
    
    # FreeBSD
    pkg install autossh
    
    # OSX
    brew install autossh

And would run on `LOCAL_SERVER` as:

    autossh -M 0 -f -q -N -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3"  -p SSH_PORT -L LOCAL_PORT:CN_SERVICE:CN_SERVICE_PORT CN_SERVER_USER@CN_SERVER

 - `-M` Specify the port to monitor, `0` disable port monitoring and will restart only on ssh exit.
 - `-f` is sends autossh to the background before running SSH.
 - `-o` adds extra SSH parameters
   - `ServerAliveInternal` is the key here as it will send keep-alive packet every given seconds to avoid SSH session to time-out.
 - All other parameters are sent to SSH.

So now `LOCAL_SERVER` can access `CN_SERVICE` using SOCKS4 proxy.

Adding a Reverse Proxy
======================
In order to expose that a little better within the local network,
and as web-server (Apache2) was available,
the following was added as a virtual host:
 
    <VirtualHost *:LISTEN_PORT>
            ServerName foo.LOCAL_SERVER_DOMAIN
    
            ProxyPreserveHost on
            ProxyRequests off
            ProxyPass / http://localhost:LOCAL_PORT/
            ProxyPassReverse / http://localhost:LOCAL_PORT/
    </VirtualHost>

Now in most case we would want the `LIST_PORT` to be `80` to simply things but in our
case the `CN_SERVICE` required it to run on a very specify port as it would generate
URL using the base domain (`foo.LOCAL_SERVER_DOMAIN`) + that specific port.
This was the easiest way to handle that case, but enjoying some URL rewriting could
work as well.

So now accessing locally `foo.LOCAL_SERVER_DOMAIN` would get us the `CN_SERVICE` by going
through a SSH Tunnel that would be kept open. Problem solved!

