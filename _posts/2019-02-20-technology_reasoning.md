---
layout: post
title:  "Basic Phoenix setup with docker"
author: azku
tags: [elixir, phoenix, docker, guardian]
categories: [website]
---

This is actually the Nth time I start to write this post. I had an idea of a Website I wanted to create and I thought to use every shiny technology I could think of. 

So there I started:

+ *I should have it inside containers so I can easily deploy it wherever I decide*. Docker comes in.
+ *I should use Phoenix in the back-end*. I have experience with Erlang, and getting to know some Elixir will be cool.
+ *Some authentication framework for Elixir and Phoenix won't hurt*. Aka Guardian
+ *I should bring some cool JavaScript framework* (now that they are hot), so I picked Vuejs.

So I started to go throught this last rabbit whole and I got really distracted. Started to bring Vuex for client side data management, vue-routes and all the jungle.

At this point I pushed down the reset button and I thought it might be better not to use a JavaScript framework for now. As Joe Armtrong sais (although in different context): *I asked for the banana and got the banana, the gorilla and the whole jungle*. Also, playing with Vue, I found it is not difficult to bring the tool later on, perhaps when a particular feature requires it.

For now, lets go with Docker and Phoenix. I will also include Guardian as a dependency but not its setup. For details on how to set up an example Phoenix authentication with Guardian [Follow this link](https://hexdocs.pm/guardian/tutorial-start.html#setup-guardian-config)



{% highlight shell %}
mix phx.new hello_world_site
{% endhighlight %}

Click "Y" when asked: "Fetch and install dependencies?"

Ones the installation is successful add Guardian and Comeonin dependencies to "hello_world_site/mix.exs"

{% highlight elixir %}
{:guardian, "~> 1.0"},
{:comeonin, "~> 4.0"},
{:bcrypt_elixir, "~> 0.12"},
{% endhighlight %}


Compile the newly added dependencies, create a simple user manager and prepare the yaml file

{% highlight shell %}
cd hello_world_site
mix deps.get
mix deps.compile
mix phx.gen.context UserManager User users username:string password:string
touch docker-compose.yml
{% endhighlight %}
Copy the following content to the touched Yaml file:

{% highlight yaml %}
version: '3.1'

services:
  web:
    image: bitwalker/alpine-elixir-phoenix:latest
    command: mix phx.server
    environment:
      - MIX_ENV=dev
      - PORT=4243
    volumes:
      - .:/opt/app
    ports:
      - "4243:4243"
    links:
      - postgres

  postgres:
    image: postgres:alpine
    ports:
      - "5432:5432"
    volumes:
      - "./volumes/postgres:/var/lib/postgresql/data"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
{% endhighlight %}

Modify line 10 in "./config/dev.exs"

{% highlight elixir %}
http: [port: System.get_env("PORT") || 4000],
{% endhighlight %}

Modify line 74 in "./config/dev.exs"

{% highlight elixir %}
hostname: "postgres",
{% endhighlight %}

We now have a dilemma. We need the container up in order to execute mix commands to generate the database. However, as soon as we launch up the containers, Phoenix will get started and it will start complaining that the database schema hasn't been generated.

We will start the containers in daemon mode, connect to the web container and execute the necessary setup.
{% highlight shell %}
docker-compose up -d
docker-compose exec web /bin/bash
mix ecto.create
mix phx.gen.context UserManager User users username:string password:string
mix ecto.migrate
docker-compose down
{% endhighlight %}

We should now be able to run the composed container and it should be stable at [http://localhost:4243](http://localhost:4243)
