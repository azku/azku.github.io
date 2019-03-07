---
layout: post
title:  "Import and parse XML in postgresql"
author: azku
tags: [postgresql, xml, camping]
categories: []
---


I found some [campsite](https://www.eurocampings.co.uk/sitemap/campsite.es.xml) data in XML format and I wanted to formalise it into a relational database.

So I downloaded the data and started analysing it.

We have the name of the camping with some sort of id and the city/town it belonged. We also have the country. From here on, we have uneven data. Some countries have counties or departments or provinces. Others don't. So I will make relational connections between countries, places and campings and I'll dump the *zone* information in some null-able columns.

First, lets create the tables to import the data and massage it later.

{% highlight sql %}
create schema if not exists import;
drop table if exists import.camping;
create table import.camping(
       url text
);
{% endhighlight %}

I have created a schema for importing the raw data and the table that will hold the raw data.

{% highlight sql %}
insert into import.camping
(select * from unnest( xpath('/mydefns:urlset/mydefns:url/mydefns:loc/text()',  pg_read_file('/docker-entrypoint-initdb.d/campsite.es.xml')::xml, ARRAY[ARRAY['mydefns', 'http://www.sitemaps.org/schemas/sitemap/0.9']])));
{% endhighlight %}

We have the data imported now. But all the information appears as a url. Lets break it up. First we will create another temporary table to keep the data in order to facilitate further processing.

{% highlight sql %}
drop table if exists camping_division;
create table camping_division(
       id2 int,
       camping varchar(200) ,
       zone1 varchar(100) ,
       zone2 varchar(100) ,
       zone3 varchar(100) ,
       zone4 varchar(100) 
);
insert into camping_division(
select  substring(url,'([0-9]*)/$')::int as id2, substring(url,'/([^/]*)-[0-9]*/$') as camping,
        substring(url,'/([^/]*)/[^/]*-[0-9]*/$') as zone1, substring(url,'/([^/]*)/[^/]*/[^/]*-[0-9]*/$') as zone2, substring(url,'/([^/]*)/[^/]*/[^/]*/[^/]*-[0-9]*/$') as zone3, substring(url,'/([^/]*)/[^/]*/[^/]*/[^/]*/[^/]*-[0-9]*/$') as zone4
from import.camping);
{% endhighlight %}

Cool, now the data it's divided and we were able to identify the camping name. Lets try to identify the countries. Note that depending on the country it may appear as *zone4*, *zone3* or *zone2*.

{% highlight sql %}
drop table if exists campsite;
drop table if exists place;
drop table if exists country;
create table country(
       id serial primary key,
       name varchar(100) not null
);
create table place(
       id serial primary key,
       country_id int references country(id),
       city varchar(100) not null,
       zone1 varchar(100) ,
       zone2 varchar(100) ,
       zone3 varchar(100) 
);
create table campsite(
       id serial primary key,
       id2 int not null, 
       place_id  int not null references place(id),
       name varchar(200) not null
);
insert into country(name)
(select distinct zone4  from camping_division where zone4 not like '%www%');

insert into country(name)
(select distinct zone3  from camping_division where zone4  like '%www%' and zone3 not like '%www%');

insert into country(name)
(select distinct zone2  from camping_division where zone3  like '%www%' and zone2 not like '%www%');
{% endhighlight %}


Lets tackle places now.

{% highlight sql %}
insert into place(city,country_id,zone1,zone2,zone3)
(select  zone1, coalesce(c2.id,c3.id,c4.id),substring( zone2, '^(?!www).+'),substring( zone3, '^(?!www).+'),substring( zone4, '^(?!www).+')  from camping_division cd left join country c2 on c2.name=cd.zone2 left join country c3 on c3.name=cd.zone3 left join country c4 on c4.name=cd.zone4 group by cd.zone1,c2.id,c3.id,c4.id ,zone2,zone3,zone4);
{% endhighlight %}

Finally, lets import and relate the campsites.

{% highlight sql %}
insert into campsite(id2, place_id, name)
(select id2,p.id ,camping from camping_division cd left join place p on cd.zone1=p.city);
{% endhighlight %}

Done! if you have internet connection to the web and Postgresql, you can do it too.
