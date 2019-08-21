---
layout: post
title:  "Full text search agains Postgresql using Ecto"
author: azku
tags: [postgresql, elixir, ecto]
categories: []
---


Building on the previous post about importing camping data into postgres, I would like to make some simple searches against the data.
I would like that seach to look into countries, places and campsites all at ones, similar to what google does in google maps.

After a little bit of thought, I  decided to use postgreses power and do the search at the database level. There are 3 related tables in which I would like to search and one column on each table.

I have created a materialized view against
