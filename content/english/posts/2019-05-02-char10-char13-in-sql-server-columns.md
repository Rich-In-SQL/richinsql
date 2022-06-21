---
title: 'CHAR(10) & CHAR(13) in SQL Server Columns'
date: 2019-05-02T16:00:47+01:00
author: Rich
layout: post
permalink: /char10-char13-in-sql-server-columns
categories:
  - sqlserver
tags:
  - SQL
  - sqlserver
  - t-sql
---

Recently I needed to export some data from a table that was being used to hold information from a contact form, it had lots of information inside of it but for some reason the Comments column was causing lots of problems when loaded up into Excel, there would be formatting issues and text appearing on a new line when in SQL it appeared that all of that text should be on one single line.

In this demonstration, I am going to show what caused that, at least for me in my scenario.

I first need a table to hold all of the data in

```
    CREATE TABLE #NewLine
    (
    ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    Data nvarchar(200),
    Name varchar(50)
    )
```

Now I have a table built, I can Insert some data into the newly created template, you can see CHAR(13) & CHAR(10) are added.

```
    INSERT INTO #NewLine (Data,Name)
    VALUES
    ('This is some data' + CHAR(13)+CHAR(10) + 'something else','Bob'),
    ('This is some data' + CHAR(13)+CHAR(10) + 'something else','Jeff'),
    ('This is some data' + CHAR(13)+CHAR(10) + 'something else','June'),
    ('This is some data' +CHAR(13)+CHAR(10) + 'something else','Charlotte'),
    ('This is some data' + CHAR(13)+CHAR(10) + 'something else','Barry'),
    ('This is some data' + CHAR(13)+CHAR(10) + 'something else','John')
```

Select the data from that table

```
    SELECT * FROM #NewLine
```

The results show that the data looks normal right?

![](/img/newline-data1.png)

But what if I was to export the dataset to a CSV file, would the results be the same? Let's find out. Export the data to a CSV file and open it up in [Notepad++](https://notepad-plus-plus.org/) if you don't have Notepad++, download it. Once you have Notepad++ installed go to the view menu, select show symbol and tick show all characters.

![](/img/newline-data4.png)

With the file now open in Notepad++ and all the hidden characters showing you can see it doesn't look right, some of the data is appearing on a new line. If I were to load this in excel it wouldn't be correct.

![](/img/newline-data2.png)

The reason for that is that excel reads CHAR(10) and CHAR(13) as line breaks and acts on them accordingly printing the text onto new lines.

To fix that, you need to replace CHAR(13) and CHAR(10) with a blank replacement

```
    SET NOCOUNT ON;
    SELECT REPLACE(REPLACE(Data,CHAR(13),''),CHAR(10),'') FROM #NewLine
```

Re-run the above select, choosing to export the data to CSV.

Now open the CSV file, again in Notepad++ this time you can see the line breaks are gone and the CSV is displaying as expected.

![](/img/newline-data3.png)
