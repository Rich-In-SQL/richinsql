---
title: Can I change my SQL database while in a restoring state
date: 2022-08-01T09:00:00.000
author: Rich
layout: post
categories:
- sqlserver
draft: true
tags:
- SQLServer
- T-SQL
- Intermediate
---

So, you have an availability group and one of your databases gets removed, during the time that the database is out of the availability group it fails over, the primary is now the secondary and the secondary is now the primary, but wait your database is still in a restoring state - what is going to happen with your applications? Will they error or will they continue to work..let's find out. 

For the purposes of this post I have created a two node availability group called AG1, this availability group contains nodes named SQL01 & SQL02 - I have two databases RichInSQL & RichInSQL_Test, both of these databases are healthy and synchronizing as expected. 

![](/img/sql-db-restoring-1.png)

Let's remove RichInSQL_Test from the availability group

![](/img/sql-db-restoring-2.png)

![](/img/sql-db-restoring-3.png)

If we connect to the secondary, which at the moment is SQL02 you can see that the database is now in a Restoring state. 

![](/img/sql-db-restoring-4.png)

Lets perform a manual failover of the availability group making SQL01 the secondary and SQL02 the primary. 

![](/img/sql-db-restoring-5.png)

![](/img/sql-db-restoring-6.png)

If we refresh the availability group in management studio we can now see that RichInSQL_Test is in a restoring state on our primary. 

![](/img/sql-db-restoring-7.png)

Our applications are connected to the availability group through the listener DNS name, AG1, so RichInSQL_Test is going to be presenting to that application as restoring. 

If our Application now tries to make a change to the database, what is going to happen. 

![](/img/sql-db-restoring-8.png)

Well, nothing good that is for sure, as the database is in a restoring state SQL Server stops any changes from being made. Changes to the database in this state would of course bring the database out of sync with what the witness knows was a healthy state so SQL Server doesn't allow it. 
