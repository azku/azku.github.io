---
layout: post
title:  "Microservices aren't silver bullet either"
author: azku
tags: [docker, rpi]
categories: []
---

I thought that microservices allow you to forget about the hardware.

I thought microservices compel you to write all the software you are going to need, in a configuration file. Then forget about what you actually have installed in your machine.

I should have thought it twice.

I wanted to try docker, and it so happens that when I tried to take my project based on bitwalker/alpine-elixir-phoenix:latest, in which I was working on my Intel based station, and move it to a RPI... well it didn't work.

That image that looked cool, based on alpine-elixir and all, is not made for arm architecture.

In hindsight, I should have known better. You do not install the same Linux on Intel based and ARM based hardware.

I just want to point out that things are always more complicated than how they look in the surface.

Now imaging wrapping 3 boosters together, launching them to space and landing them back vertically! scratch the surface there.

