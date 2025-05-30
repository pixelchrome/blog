---
title: "Goodbye Ghost"
date:
  created: 2019-10-05

linktitle: "Goodbye Ghost"
slug: "goodbye-ghost"

description: "Its time to say goodbye to the software I've used for long time to blog here. GHOST"

tags:
- Ghost
- General
- Blog

authors:
- harry
---
![Ghost Logo](https://upload.wikimedia.org/wikipedia/commons/thumb/f/fa/Ghost-Logo.svg/2560px-Ghost-Logo.svg.png)

## Goodbye Ghost

Its time to say goodbye to the software I've used for long time to blog here. [Ghost](https://ghost.org) was a perfect piece of software when I started ([again](https://pixelchrome.org/blog/es-ist-wieder-zeit/)).

I've learned a lot during that time, e.g. [Markdown](https://daringfireball.net/projects/markdown/), configuring [NGINX](https://nginx.org) to serve data from Ghost. And how to get [nodejs](https://nodejs.org/) running on my server (which is not that easy as it sounds. more later...)


<!-- more -->

## Why?

The problems I've had:

* It relies on nodejs - especially at the beginning, there had been some restrictions with the versions. Therefore it was necessary to find a version that is supported with the OS I use and which works with Ghost.
* That made  updates of FreeBSD not that easy, as nodejs need some more specific attention (e.g. add some changes to the node4 `Makefile` to be able to run `sudo portmaster -m DISABLE_VULNERABILITIES=yes www/node4`)
* I've created my own startup script for ghost. Meanwhile there are many and better versions available. Just search for `ghost_start()` and you will find plenty of them.
* I had massive problems to get Ghost updated to newer version. In more detail, I wasn't able to **export** the *blog settings and data*. It was always a hassle to do that by hand.
* Therefore I was not able to update it regularly.
* All of that lead to a bad gut feeling (security)

I still believe that Ghost is a wonderful platform for blogging. Especially the hosted version.

## Move to /blog_old

You will find ghost still running for some time at `pixelchrome / blog_old`.
