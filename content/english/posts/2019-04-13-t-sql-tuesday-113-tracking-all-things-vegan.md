---
title: 'T-SQL Tuesday 113 - Tracking All Things Vegan'
date: 2019-04-13T08:30:42+01:00
author: Rich
layout: post
permalink: /t-sql-tuesday-113-tracking-all-things-vegan
categories:
  - sqlserver
tags:
  - database
  - sqlserver
  - story
  - T-SQL Tuesday
---

![](/img/T-SQL-Tuesday-Logo.jpg)

This month's T-SQL Tuesday comes from Todd Kleinhans ([Blog](https://toddkleinhans.wordpress.com/) & [Twitter](http://www.twitter.com/toddkleinhans)) he asked, &#8220;what do YOU use databases for in your personal life?&#8221;

> I’m curious- outside of work and learning, what do you personally use databases for? Tracking books you have, recipes, collections, etc? While it can be said using databases for personal use could be either overkill or a hammer in search of nails on the other hand, it is exactly what they are for- storing data.

### It must be Vegan

It is no secret that I am Vegan, but what does that have to do with databases? Well, when I first became Vegan keeping track of what was and what wasn't vegan-friendly was extremely difficult, people would often ask me if xyz was Vegan and I would have to think, have I used that before? If I didn't know, or couldn't remember I would have to look it up on a website that may itself not be 100% accurate.

To get around this problem I was making notes in an old notebook, was the make-up my sister asked for Christmas Vegan? Was the beer I had at the pub Vegan? After a while it became unmanageable, I wanted a way to be able to sort the data, track it and better manipulate it.

That is where the SQL Database came from.

### The Database Was Born

When I started as a SQL Server developer I wanted something that would give me a project, something outside of my day to day work life that I could use to collect my own data in, manipulate it and not break any of the work rules, so I designed **_Is That Vegan_** taking what I had already collected and importing that into a SQL Database, then going back and completing the gaps.

![](/img/isitvegan.png)

### It may not be perfect

The database may not be 100% perfect, I am almost certain there is room for improvement but that was and still is the point of this project, I wanted something that I could continuously learn from but also gain value from professionally, something that would allow me to capture information about something I was interested in but also allow me to grow as a developer.

### It Grows

The joys with this project are there are so many different avenues for data collection within the topic of Veganism

- Food
- Cosmetics
- Clothing
- Restaurants

To name but a few, so the database will certainly grow as there is more data added, at the moment I am adding all of the data manually through a stored procedure, it would be useful to create an input form to handle all of that but that is a topic for another post.
