---
title: Refreshing Object Meta Data 
date: 2022-05-16T09:00:00.000+01:00
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

You have a stored procedure that uses a view to insert data into a table that you have created somewhere along the line the base table that the view references changes, the data types on one of the columns gets amended and all of a suddent your stored procedure starts failing. Why? Lets have a look. 

Lets create out a base table that we can reference in a view, this is just a simple table 

```
CREATE TABLE dbo.Customers
(
    ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    Customer_Name nvarchar(50),
    Credit_Available INT
)
```

Now we need to create view to reference that table 

```
CREATE VIEW dbo.vw_Customers
AS
SELECT
    ID
    Customer_Name
    Credit_Available
FROM dbo.Customers
```

We now need a staging table that our stored procedure is going to insert the customer data into every day.

```
CREATE TABLE dbo.Customers
(
    ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    CensusDate DATE,
    Customer_Name nvarchar(50),
    Credit_Available INT
)
```

Finally the store procedure to bring all of this together

```
CREATE PROCEDURE dbo.sp_Customers_Audit

AS

SELECT 
    ID,
    Customer_Name,
    Credit_Available
FROM dbo.vw_Customers
```

That stored procedure should now execute perfectly 

```
EXEC dbo.sp_Customers_Audit
```

Now, someone is going to come along and change dbo.Customers becuase Credit_Available should be yes or no which is a BIT not an INT so lets do that. 

```
ALTER TABLE dbo.Customers ALTER COLUMN Credit_Available BIT
```

Let's run that stored procedure again and see what happens now 

```
EXEC dbo.sp_Customers_Audit
```

It fails, but why? If we look at the meta data of the view it shows that Credit_Available is still an INT, even though we have changed the base table's schema from an INT to a BIT the view META data for the table hasn't been refreshed. So how do we refresh it. 

Running a select on the view won't do anything, the meta data will remain, we can prove that by running the test 

```
SELECT * FROM dbo.customers
```

What we need to do is open the view in alter mode and execute the transaction, so if you right click on the view and select ALTER as from the context menu, when the code opens just press execute or hit F5

Refresh the schema for that view in SSMS and now the meta data has changed, the stored procedure should now work as expected.

```
EXEC dbo.sp_Customers_Audit
```



