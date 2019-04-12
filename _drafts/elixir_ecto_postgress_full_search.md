---
layout: post
title:  "Full text search agains Postgresql using Ecto"
author: azku
tags: [postgresql, elixir, ecto]
categories: []
---


Building on the previous post about importing camping data into postgres, I would like to make some simple searches against the data.
I would like that seach to look into countries, places and campsites all at ones, similar to what google does in google maps.

I started my search at the bottom as I am a newby in pretty much every technology I am using and I found a powerfull enough feature in Postgres. So I decided to make the full text search there.

