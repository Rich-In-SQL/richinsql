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


CREATE TABLE #PrivotExample 
(
	[patient_id] INT,
	[current_status] INT,
	[contact_outcome] INT
);

CREATE TABLE #PrivotExampleStatus 
(
	[Status_ID] INT,
	[status_desc] varchar(50),
	[data_field] varchar(50)
);

INSERT INTO #PrivotExampleStatus
(
  [Status_ID], 
  [status_desc],
  [data_field]
)
VALUES
    (1, 'Status 1','current_status'),
    (2, 'Status 2','current_status'),
    (3, 'Status 3','current_status'),
    (4, 'Status 4','current_status'),
    (5, 'Status 5','current_status'),
    (6, 'Status 6','current_status'),
    (1, 'Status 1','contact_outcome'),
    (2, 'Status 2','contact_outcome'),
    (3, 'Status 3','contact_outcome'),
    (4, 'Status 4','contact_outcome'),
    (5, 'Status 5','contact_outcome'),
    (6, 'Status 6','contact_outcome');
    
INSERT INTO #PrivotExample
(
  [patient_id], 
  [current_status],
  [contact_outcome]
)
VALUES
    (111111, 1,1),
    (111112, 1,1),
    (111113, 2,1),
    (111114, 3,1),
    (111115, 4,1),
    (111116, 5,1),
    (111117, 6,1),
    (111118, 7,1),
    (111119, 3,1),
    (1111110, 3,1),
    (1111111, 3,1),
    (1111112, 3,1),
	(1111113, 1,1),
    (1111124, 1,1),
    (1111135, 1,1),
    (1111146, 1,1),
    (1111157, 2,1),
    (1111168, 2,1),
    (1111179, 2,1),
    (11111810, 2,1),
    (11111911, 3,1),
    (111111012, 3,1),
    (111111113, 3,1),
    (111111214, 3,1);

SELECT * 
	FROM 
		(
		SELECT current_status 
		FROM 
			#PrivotExample
		) src
		PIVOT 
		( 
			COUNT(current_status) 
		FOR current_status IN ([1],[2],[3],[4],[5],[6])
		) piv;

SELECT * 
	FROM 
		(
		SELECT contact_outcome 
		FROM 
			#PrivotExample
		) src
		PIVOT 
		( 
			COUNT(contact_outcome) 
		FOR contact_outcome IN ([1],[2],[3],[4],[5],[6])
		) piv;		


DROP TABLE #PrivotExample,#PrivotExampleStatus
