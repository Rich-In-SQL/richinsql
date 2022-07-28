---
title: SQL Server Pivot
date: 2022-07-25T09:00:00.000+01:00
author: Rich
layout: post
categories:
- sqlserver
draft: false
tags:
- SQLServer
- T-SQL
- Intermediate
---

In this post we are going to have a look at a very basic [PIVOT](https://docs.microsoft.com/en-us/sql/t-sql/queries/from-using-pivot-and-unpivot?view=sql-server-ver16) table and how to go about creating one for a simple query. 

For this query, we want to know how many patients are currently in a particular status, let's have a look at this with an example. Create a table first to hold our records.

```
CREATE TABLE #PivotExample 
(
	[patient_id] INT,
	[current_status] INT,
	[contact_outcome] INT
);
```

Then we will need another table to hold our statuses.

```
CREATE TABLE #PivotExampleStatus 
(
	[Status_ID] INT,
	[status_desc] varchar(50),
	[data_field] varchar(50)
);
```
Insert the statuses into the table we just created

```
INSERT INTO #PivotExampleStatus
(
  [Status_ID], 
  [status_desc],
  [data_field]
)
VALUES
    (1, 'current_status 1','current_status'),
    (2, 'current_status 2','current_status'),
    (3, 'current_status 3','current_status'),
    (4, 'current_status 4','current_status'),
    (5, 'current_status 5','current_status'),
    (6, 'current_status 6','current_status'),
    (1, 'contact_outcome 1','contact_outcome'),
    (2, 'contact_outcome 2','contact_outcome'),
    (3, 'contact_outcome 3','contact_outcome'),
    (4, 'contact_outcome 4','contact_outcome'),
    (5, 'contact_outcome 5','contact_outcome'),
    (6, 'contact_outcome 6','contact_outcome');
```

Then our patients, we are going to put some statuses from above against some of these records. 

```    
INSERT INTO #PivotExample
(
  [patient_id], 
  [current_status],
  [contact_outcome]
)
VALUES
    (1, 1,1),
    (2, 1,1),
    (3, 2,1),
    (4, 3,1),
    (5, 4,4),
    (6, 5,3),
    (7, 6,1),
    (8, 7,3),
    (9, 3,1),
    (10, 3,1),
    (11, 3,1),
    (12, 3,2),
    (13, 1,2),
    (14, 1,3),
    (15, 1,5),
    (16, 1,1),
    (17, 2,1),
    (18, 4,1),
    (19, 1,1),
    (20, 5,6),
    (21, 3,1),
    (22, 6,2),
    (23, 3,1),
    (24, 3,1);
```

Now, we can pivot this data. Let's have a look how we are going to do that. 

First, we need to create a basic query, this will be used as the inner query in the final query. 

This query is going to select the count of all the statuses for the rows, grouped by the status description.

```
SELECT 
	COUNT(current_status) as Total,
	pes.status_desc 
FROM 
	#PivotExample PE

INNER JOIN #PivotExampleStatus PES
	ON PE.current_status = PES.Status_ID
	AND PES.data_field ='current_status'

GROUP BY 
	pes.status_desc
```

Now that we have the inner query we need to wrap it in an outer query selecting everything from it, which would give us something like this. 

```
SELECT * FROM 
(
SELECT 
	COUNT(current_status) as Total,
	pes.status_desc 
FROM 
	#PivotExample PE

INNER JOIN #PivotExampleStatus PES
	ON PE.current_status = PES.Status_ID
	AND PES.data_field ='current_status'

GROUP BY 
	pes.status_desc
) AS src
```

When wrapping queries in an outer query, it is important that they are given an alias so that the outer query knows what columns we are talking about, you can see this at the end of the above example. 

```
GROUP BY 
	pes.status_desc
) AS src
```

```
SELECT * 
	FROM 
		(
		SELECT COUNT(current_status) as Total,pes.status_desc 
		FROM 
			#PivotExample PE

		INNER JOIN #PivotExampleStatus PES
			ON PE.current_status = PES.Status_ID
			AND PES.data_field ='current_status'

			GROUP BY pes.status_desc

		) src
		PIVOT 
		( 
			SUM(Total) 
		FOR status_desc IN ([current_status 1],[current_status 2],[current_status 3],[current_status 4],[current_status 5],[current_status 6])
		) piv;
```

Now, we can add the PIVOT onto our query, at the very end of what we have already wrote we need to add

```
PIVOT 
		( 
			SUM(Total) 
		FOR status_desc IN ([current_status 1],[current_status 2],[current_status 3],[current_status 4],[current_status 5],[current_status 6])
		) piv;
```

What this is going to do it sum the Total column, which in our inner query was a count of all the statuses, grouped by status, it is then going to look for all status_desc in the provided list

```
FOR status_desc IN ([current_status 1],[current_status 2],[current_status 3],[current_status 4],[current_status 5],[current_status 6])
```

In our case, these are all the status_desc from #PivotExampleStatus

Once you have done that, you should have a query which looks something like this.

```
SELECT * 
	FROM 
		(
		SELECT COUNT(contact_outcome) as Total,pes.status_desc 
		FROM 
			#PivotExample PE

		INNER JOIN #PivotExampleStatus PES
			ON PE.contact_outcome = PES.Status_ID
			AND PES.data_field ='contact_outcome'

			GROUP BY pes.status_desc

		) src
		PIVOT 
		( 
			SUM(Total) 
		FOR status_desc IN ([contact_outcome 1],[contact_outcome 2],[contact_outcome 3],[contact_outcome 4],[contact_outcome 5],[contact_outcome 6])
		) piv;	
```

The output in SQL Management Studio should look like this, it may vary, depending on what you inserted into your tables of course.

|contact_outcome_1|contact_outcome_2|contact_outcome_3|contact_outcome_4|contact_outcome_5|contact_outcome_1|
|---|---|---|---|---|---|
|7 |2   |  8 |  2 |  2 | 2 |
