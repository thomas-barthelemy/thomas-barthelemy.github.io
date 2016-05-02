---
layout:     post-no-pic
title:      "Spotlight: Re:dash (redash.io)"
subtitle:   "An Open Source Platform To Make Your Company Data Driven"
date:       2016-05-02 00:00:00
author:     "Thomas Barthelemy"
tags:       [spotlight]
---

On today's Spotlight we have **[redash.io](http://redash.io)**,
which is the type of tool you discover and then wonder why nobody thought of that.

<img alt="redash dashboard"
     src="http://redash.io/static/img/redash_screenshot_dashboard.png"
     style="max-width: 80%;"
>

## Introduction

The idea behind re:dash is to integrate with an existing database and provide
an in-browser query editor and create great interactive visualization on the result.

It's offered either as a SaaS (starting at $29/Month) or on-premise (Free / Open Source)

## Setup

Setting up Re:dash is fairly easy and there is a few possibilities, here is how I
deployed it on-premise following the
[Quick Start Guide](http://docs.redash.io/en/latest/setup.html) using `docker-compose`:

 1. Getting the source: `git clone https://github.com/getredash/redash.git`
 2. Update `docker-compose-example.yml` for containers config if needed (default exposed port...)
 3. Rename `docker-compose-example.yml` to `docker-compose.yml`
 4. Starting only the DB: `docker-compose up -d postgres`
 5. Creating the DB Schema: `./setup/docker/create_database.sh`
 6. Running all containers: `docker-compose up -d`
 
you can then stop/start the whole stack using
`docker-compose start` and `docker-compose stop`

The first time you can login using the user/password `admin`/`admin`, consider changing
it as soon as possible.

Other deploy options include:

 - AWS AMI
 - Google Compute Engine Image
 - Self install script
 - [Bitnami](https://bitnami.com/stack/redash) installer
 
Overall it's was pretty quick and worked as expected, kudos for that!


## Awesome Features

 - Sharing Queries between developers for optimization and advices
 - Creating in a minute some great interactive data visualization (table/charts/map...)
 - Exporting Data Result as CSV / Excel
 - User Rights management for letting people only view/export data
 - One-click Embeddable visualization (iframe)
 - The amount of DB supported (18 on the list so far)

## Limitations

 - Although it seems to be a very important part, the collaboration on query and
visualization is quite limited to being able to read/write it.
 - It would be cool to see more features for query optimization, I could see myself
collaborating with a few other people on a major query to fine-tune it, having extended
report on the query runtime.
 - A little more support to label chart easily without having to actually have to modify
the query to give data the right label.
 - Some settings/parameters for embedding visualization, to show/hide some options
 
## Conclusion

Re:dash is a very well thought platform getting you the sweet visualization in < 10min,
and it's open source. It definitely has some rough edge here and there but repository is
very active and contributors are self-conscious on the current limitations, working hard
to improve it.

Bottom-line: Seriously, give [redash.io](http://redash.io/) a try! 



