---
title: 
date: 2022-02-28T09:00:00.000+01:00
image: "/images/post/apple-health-powerBi-dashboard.jpg"
author: Rich
layout: post
categories:
- sqlserver
draft: true
tags:
- SQLServer
- T-SQL
- Advanced
---

There are many ways you can version control your database, you can create a Visual Studio SQL Database project and add that to your source control of choice, you can buy a professional product to do the work for you or you can write something yourself, we decided to go with the latter option and write something ourselves. 

The idea was that we wanted a script that we could run against any database and the following objects would be exported 

- Tables including primary keys
- Views
- Stored Procedures
- Constraints excluding primary keys

Each exported object would have it's own file, and each object type would be stored in a folder of that objects type, for example */Stored Procedures/StoredProcedure1.sql*

We explored a number of ways that we could go about doing this before settling on using [DBATools](https://dbatools.io/) to do a lot of the heavy lifting for us, [DBATools](https://dbatools.io/) already has [modules](https://dbatools.io/commands/) for most things in SQL Server so we just wrote a script around that it to export what we wanted from the server. 
As the script needed to run without intervention it needed to have logging and error handling built in, at each stage it would write to a text file what it was doing, which object it was exporting and any errors that it came across in the process. 
The script also needed to accept parameters, if it didn't running the script would mean opening it up and editing it every time it needed running, which would be a pain.

