---
layout:     post-no-pic
title:      "Spotlight: Tyk.io"
subtitle:   "An open source, fast and scalable API management platform featuring an API gateway, analytics, developer portal and dashboard"
date:       2016-01-04 18:21:00
author:     "Thomas Barthelemy"
tags:       [spotlight]
---

**2016** is here, and as part of this year challenges I gave myself is "**Spotlight**".
Spotlight will be a series of (short) articles on tool/platform/project I use,
most of the time open source, that deserve to be more popular.
The article itself will be about what I found to be awesome in those tool/platform/project
but also the limitations and issues I had with it.

In today's Spotlight: [Tyk](https://tyk.io/)

## Introduction

Tyk is an open source API gateway in Go:
it comes between your Server and your clients in order to modify and monitor the requests.

## Availability

  - **Source Code**: Tyk is open source and the code is available on github [here](https://github.com/TykTechnologies/tyk)
  - **On Premise**: Installing Tyk on premise is not a one-liner but fairly well documented in the [setup documentation](https://tyk.io/v1.9/setup/installation-options/)
  - **SaaS**: Multiple pricing, but the free-tier includes all the features and a few limits that you won't reach that easly (e.g. 50k reqests daily).

## Integration

I was expecting having to change a bunch of configs but actually
you create an API pointing to your original API, and you request it instead.
Done.

## Awesome Features

This is not the complete list of features, just the one I really enjoyed.

  - **Analytics**: how many times / what was requested / return code. This is great to see errors and usage.
  - **Security**: Set a endpoint public or with login, set usage quotas and blacklist/whitelist endpoints.
  - **Endpoint Customization**: You can modify headers, set caching or even mock an answer for each endpoints.
  - **Import**: You can import your API definition from tools like Swagger, amazing to get started quickly.

Tyk also allows you to setup a Developer portal (with Doc / API keys and all the usual needed to manage it)
but I haven't tested that feature yet.

## Limitations

I haven't had any major issue with the platform,
but Tyk is still in its early age and a few minor bugs may appear
although the Github issue tracking seems quite reactive.

My main issue with the platform was the **few amount of details provided in the analytics**,
there are definitely plenty of info but
I can only know that there were X failed requests with error code xxx.
I'd love to see the requests that have failed in details (Request/Response) to help the debugging.

## Conclusion

I Think **Tyk is a great platform for people who wants to get more out of their API**.
It might have a lot less value for those who use a BaaS which already provides analytics and API management.

That's it for this Spotlight, check out [tyk.io](http://tyk.io) to know more about it.
You have other similar platform or tools for a next Spotlight? Leave it in the comments!
