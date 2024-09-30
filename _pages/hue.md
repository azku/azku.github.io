---
layout: page
title: Hue Docker erabilita
permalink: /hue/
---

{% highlight yaml %}
services:
  cloudera:
      image: gethue/hue:latest
      hostname: hue
      container_name: hue
      dns: 8.8.8.8
      ports:
      - "8888:8888"
      volumes:
        - ./hue.ini:/usr/share/hue/desktop/conf/z-hue.ini
      depends_on:
        database:
          condition: service_healthy
  database:
      image: mysql
      hostname: mydatabase
      ports:
          - "3306:3306"
      command: --init-file /data/application/init.sql
      volumes:
          - data:/var/lib/mysql
          - ./init.sql:/data/application/init.sql
      environment:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: hue
      healthcheck:
        test: mysqladmin ping -h 127.0.0.1 -u $$MYSQL_USER --password=$$MYSQL_PASSWORD
        start_period: 5s
        interval: 5s
        timeout: 5s
        retries: 55
volumes:
  data:
{% endhighlight %}


init.sql edukia

{% highlight sql %}
CREATE DATABASE IF NOT EXISTS hue;
{% endhighlight %}

{% highlight conf %}
  [[database]]
    engine=mysql
    host=mydatabase
    port=3306
    user=root
    password=secret
    name=hue
  [beeswax]
    hive_server_host=10.2.32.143


{% endhighlight %}
