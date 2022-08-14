---
title: Dynamically creating a SQL Server table with dynamic SQL
date: 2022-08-08T09:00:00.000+01:00
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

Sometimes there is a requirement to build out a table based on another table in your SQL Server programmatically, for example you may have a list of tables you wish to build but have no idea what the columns in the schema are. 

In this post, we are going to look at how we can use dynamic SQL to build a table programmatically which you can then use in your scripts. 


We are going to make use of the Information Schema, this is specific to each database, in this example, all of the table we want to target are within the same StackOverflow database. If you would like to find out more about getting information from the Information Schema we wrote a post on that [here](/2022/04/using-sql-servers-information-schema).


```
SELECT 
	COLUMN_NAME 
FROM 
	StackOverflow.INFORMATION_SCHEMA.COLUMNS
WHERE  
	TABLE_NAME = 'Posts'
```

Here is the base query that will get the columns for the table Posts, this is no good to us in Dynamic SQL in it's current form as it would produce our dynamic SQL for every row a column is returned on and we don't want that. 

```
SELECT  
	substring(list, 1, len(list) - 1)
FROM   
	(
		SELECT 
			COLUMN_NAME + ','
		FROM 
			StackOverflow.INFORMATION_SCHEMA.COLUMNS
		WHERE  
			TABLE_NAME = 'Posts'
		ORDER BY 
			ORDINAL_POSITION
		FOR XML PATH('')
	) AS T(list)
```

What we can do is use XML PATH to list the columns out to a single list, you can see that we are concatenating a comma to the end of each column inside the inner select, this is so that each column is properly separated, in the outer query we are making use of SUBSTRING to remove the comma from the end of the string, this is because a comma is not required for the last column in the create table expression.

```
CONCAT(QUOTENAME(UPPER(COLUMN_NAME)), + '' + UPPER(DATA_TYPE), + CASE WHEN DATA_TYPE IN ('DATE','DATETIME','INT') THEN ',' ELSE '(' + CAST(CHARACTER_MAXIMUM_LENGTH as varchar)  + '),' END)
```

What we now need to do is create properly create our column expressions for the command, each column needs to have a data type where a data type is available, so what we are going to do is CONCAT the following columns 

- COLUMN_NAME - This is the name of the column
- DATA_TYPE - This is the data type of the column
- CHARACTER_MAXIMUM_LENGTH - This is the max length of the data type for that column

If -1 is provided in the CHARACTER_MAXIMUM_LENGTH this is the MAX value for that data type E.G *NVARCHAR(MAX)*

```
SELECT  
	substring(list, 1, len(list) - 1)
FROM   
	(
		SELECT CONCAT(QUOTENAME(UPPER(COLUMN_NAME)), + '' + UPPER(DATA_TYPE), + CASE WHEN DATA_TYPE IN ('DATE','DATETIME','INT') THEN ',' ELSE '(' + CASE WHEN CHARACTER_MAXIMUM_LENGTH = '-1' THEN 'MAX' ELSE CAST(CHARACTER_MAXIMUM_LENGTH as varchar)  END + '),' END)
		FROM 
			StackOverflow.INFORMATION_SCHEMA.COLUMNS
		WHERE  
			TABLE_NAME = 'Posts'
		ORDER BY 
			ORDINAL_POSITION
		FOR XML PATH('')
	) AS T(list)
```

This should now give you something that looks like the above. 

```
DECLARE 
	@destinationDatabase SYSNAME,
	@destSchemaName SYSNAME,
	@tableName SYSNAME,
	@sql NVARCHAR(MAX)

SET @destinationDatabase = 'StackOverflow_New'
SET @destSchemaName = 'dbo'
SET @tableName = 'Posts'

DECLARE
	@cStat varchar(400) = 'IF NOT EXISTS (SELECT table_name FROM ' + QUOTENAME(@destinationDatabase) + '.INFORMATION_SCHEMA.TABLES WHERE table_schema = ''' + @destSchemaName + ''' AND table_name = ''' + @tableName + ''') CREATE TABLE ' + @destSchemaName + '.' + @tableName + ' ( ',
	@cEnd varchar(400) = ' )',
	@column_listOut nvarchar(MAX)
```
Now comes the fun part, here we are declaring some variables 

- @destinationDatabase SYSNAME - This will hold the name of the database we want the new table to be created in
- @destSchemaName SYSNAME - This will hold the name of the schema where the table will be created
- @tableName SYSNAME - This is the name of the table we are creating. 
- @cStat varchar(400) - This is the start of the CREATE TABLE command
- @cEnd varchar(400) = ' )' - This is the end of the CREATE TABLE command
- @column_listOut nvarchar(MAX) - This is the column list that will be output when @column_list is executed.

```
DECLARE @column_list nvarchar(MAX) = 'SELECT @column_list = substring(list, 1, len(list) - 1)
FROM   (SELECT CONCAT(QUOTENAME(UPPER(COLUMN_NAME)), + '' '' + UPPER(DATA_TYPE), + CASE WHEN DATA_TYPE IN (''DATE'',''DATETIME'',''INT'') THEN '','' ELSE ''('' + CASE WHEN CHARACTER_MAXIMUM_LENGTH = ''-1'' THEN ''MAX'' ELSE CAST(CHARACTER_MAXIMUM_LENGTH as varchar) END + '' ),'' END)
		FROM ' + @destinationDatabase + '.INFORMATION_SCHEMA.COLUMNS
		WHERE  TABLE_NAME = ' + '''' + @tableName + '''' + '
		ORDER BY ORDINAL_POSITION
		FOR XML PATH('''')) AS T(list)'
```

We need to put the columns that we have obtained from the table's Information Schema into a variable, this is declared as NVARCHAR(MAX) 

```
EXEC sp_executesql @column_list, N'@column_list nvarchar(MAX) out', @column_listOut OUT
```

We then need to execute the @column_list SQL to return the column list which will be populated into @column_listOut

```
SET @sql = @cStat + @column_listOut + @cEnd
```

The last thing to do is bring everything together, concatenate the cStat which is the start of the CREATE TABLE command then the column_listOutput finally the cEnd which is just the closing brackets at the end of the query. 

```
IF NOT EXISTS (
    SELECT
        table_name
    FROM
        [StackOverflow_New].INFORMATION_SCHEMA.TABLES
    WHERE
        table_schema = 'dbo'
        AND table_name = 'Posts'
) CREATE TABLE dbo.Posts (
    [ID] INT,
    [ACCEPTEDANSWERID] INT,
    [ANSWERCOUNT] INT,
    [BODY] NVARCHAR(MAX),
    [CLOSEDDATE] DATETIME,
    [COMMENTCOUNT] INT,
    [COMMUNITYOWNEDDATE] DATETIME,
    [CREATIONDATE] DATETIME,
    [FAVORITECOUNT] INT,
    [LASTACTIVITYDATE] DATETIME,
    [LASTEDITDATE] DATETIME,
    [LASTEDITORDISPLAYNAME] NVARCHAR(40),
    [LASTEDITORUSERID] INT,
    [OWNERUSERID] INT,
    [PARENTID] INT,
    [POSTTYPEID] INT,
    [SCORE] INT,
    [TAGS] NVARCHAR(150),
    [TITLE] NVARCHAR(250),
    [VIEWCOUNT] INT
)
```

If we select the final query, we should have something like this.

Here is what the entire script should look like once your done 

```
DECLARE 
	@destinationDatabase SYSNAME,
	@destSchemaName SYSNAME,
	@tableName SYSNAME,
	@sql NVARCHAR(MAX)

SET @destinationDatabase = 'StackOverflow'
SET @destSchemaName = 'dbo'
SET @tableName = 'Posts'

DECLARE
	@cStat varchar(400) = 'IF NOT EXISTS (SELECT table_name FROM ' + QUOTENAME(@destinationDatabase) + '.INFORMATION_SCHEMA.TABLES WHERE table_schema = ''' + @destSchemaName + ''' AND table_name = ''' + @tableName + ''') CREATE TABLE ' + @destSchemaName + '.' + @tableName + ' ( ',
	@cEnd varchar(400) = ' )',
	@column_listOut nvarchar(MAX)

DECLARE @column_list nvarchar(MAX) = 'SELECT @column_list = substring(list, 1, len(list) - 1)
FROM   (SELECT CONCAT(QUOTENAME(UPPER(COLUMN_NAME)), + '' '' + UPPER(DATA_TYPE), + CASE WHEN DATA_TYPE IN (''DATE'',''DATETIME'',''INT'') THEN '','' ELSE ''('' + CASE WHEN CHARACTER_MAXIMUM_LENGTH = ''-1'' THEN ''MAX'' ELSE CAST(CHARACTER_MAXIMUM_LENGTH as varchar) END + '' ),'' END)
		FROM ' + @destinationDatabase + '.INFORMATION_SCHEMA.COLUMNS
		WHERE  TABLE_NAME = ' + '''' + @tableName + '''' + '
		ORDER BY ORDINAL_POSITION
		FOR XML PATH('''')) AS T(list)'

EXEC sp_executesql @column_list, N'@column_list nvarchar(MAX) out', @column_listOut OUT

SET @sql = @cStat + @column_listOut + @cEnd
```

Here is a solution we made to loop over some tables dynamically and create them.

```
DECLARE 
	@cnt INT, 
	@sql nvarchar(MAX),
	@archiveConfig sysname

/* ================================================
--
-- Prepare the loop variables
--
-- ================================================	*/
SET @cnt = 1

DECLARE 
		@mCNT nvarchar(max) = 'SELECT @mCNTOut = MAX(ID) FROM ' + QUOTENAME(@archiveConfig) + '.[dbo].Configuration'
DECLARE 
		@mCNTOut INT

EXEC sp_executesql @mCNT, N'@mCNTOut INT out', @mCNTOut OUT

/* ================================================
--
-- BEGIN THE WHILE LOOP
--
-- ================================================ */

WHILE @cnt <= @mCNTOut

BEGIN

	DECLARE 
			@tableName nvarchar(max) = 'SELECT @tableNameOut = ObjectName FROM ' + QUOTENAME(@archiveConfig) + '.' + '[dbo].Configuration WHERE ID = ' + CAST(@cnt as varchar)
	DECLARE 
			@tableNameOut SYSNAME
	EXEC sp_executesql @tableName, N'@tableNameOut SYSNAME out', @tableNameOut OUT	

	DECLARE 
			@database nvarchar(max) = 'SELECT @database = DatabaseName FROM ' + QUOTENAME(@archiveConfig) + '.' + '[dbo].Configuration WHERE ID =  ' + CAST(@cnt as varchar)
	DECLARE 
			@databaseOut SYSNAME

	EXEC sp_executesql @database, N'@database SYSNAME out', @databaseOut OUT

	DECLARE 
			@archiveDatabase nvarchar(max) = 'SELECT @archiveDatabase = ArchiveDatabase FROM ' + QUOTENAME(@archiveConfig) + '.' + '[dbo].Configuration WHERE ID = ' + CAST(@cnt as varchar)
	DECLARE 
			@archiveDatabaseOut SYSNAME

	EXEC sp_executesql @archiveDatabase, N'@archiveDatabase SYSNAME out', @archiveDatabaseOut OUT

	DECLARE 
			@schemaName nvarchar(max) = 'SELECT @schemaName = TableSchema FROM ' + QUOTENAME(@archiveConfig) + '.' + '[dbo].Configuration WHERE ID = ' + CAST(@cnt as varchar)
	DECLARE 
			@schemaNameOut SYSNAME
	EXEC sp_executesql @schemaName, N'@schemaName SYSNAME out', @schemaNameOut OUT

	DECLARE 
			@destSchemaName nvarchar(max) = 'SELECT @destSchemaName = DestinationTableSchema  FROM ' + QUOTENAME(@archiveConfig) + '.' + '[dbo].Configuration WHERE ID = ' + CAST(@cnt as varchar)
	DECLARE 
			@destSchemaNameOut SYSNAME
	EXEC sp_executesql @destSchemaName, N'@destSchemaName SYSNAME out', @destSchemaNameOut OUT

	DECLARE 
			@keycolumn nvarchar(max) = 'SELECT @keycolumn = KeyColumn FROM ' + QUOTENAME(@archiveConfig) + '.' + '[dbo].Configuration WHERE ID = ' + CAST(@cnt as varchar)
	DECLARE 
			@keycolumnOut SYSNAME
	EXEC sp_executesql @keycolumn, N'@keycolumn SYSNAME out', @keycolumnOut OUT

	/* ================================================
	--
	-- Create the schema
	-- Check if the schema exists in the archive 
	--
	-- ================================================ */

	SET @SQL = NULL

	DECLARE 
		@sCnt nvarchar(max) = 'SELECT @sCnt = COUNT(SCHEMA_NAME) FROM ' + @archiveDatabaseout + '.INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = ''' + @destSchemaNameout  + ''''

	DECLARE 
		@sCntOut INT
	SET 
		@sCntOut = 0

	EXEC sp_executesql @sCnt, N'@sCnt int out', @sCntOut OUT

	SET @sql = 'CREATE SCHEMA ' + QUOTENAME(@destSchemaNameOut)
	
	BEGIN 

		BEGIN TRY		

		IF @sCntOut = 0 

		BEGIN
			EXEC(@SQL)
		END

		END TRY
		BEGIN CATCH

			ROLLBACK TRAN

		END CATCH 

	END 

	/* ================================================
	--
	-- Build out the table
	-- Check if the table exists in the archive 
	--
	-- ================================================ */

	SET @SQL = NULL

	DECLARE 
		@cStat varchar(400) = 'IF NOT EXISTS (SELECT table_name FROM ' + QUOTENAME(@archiveDatabaseout) + '.INFORMATION_SCHEMA.TABLES WHERE table_schema = ''' + @destSchemaNameout + ''' AND table_name = ''' + @tableNameout + ''') CREATE TABLE ' + @destSchemaNameout + '.' + @tableNameout + ' ( ',
		@cEnd varchar(400) = ' )',
		@column_listOut nvarchar(MAX)

	DECLARE @column_list nvarchar(MAX) = 'SELECT @column_list = substring(list, 1, len(list) - 1)
	FROM   (SELECT CONCAT(QUOTENAME(UPPER(COLUMN_NAME)), + '' '' + UPPER(DATA_TYPE), + CASE WHEN DATA_TYPE IN (''DATE'',''DATETIME'',''INT'') THEN '','' ELSE ''('' + CASE WHEN CHARACTER_MAXIMUM_LENGTH = ''-1'' THEN ''MAX'' ELSE CAST(CHARACTER_MAXIMUM_LENGTH as varchar) END + '' ),'' END)
			FROM ' + @databaseout + '.INFORMATION_SCHEMA.COLUMNS
			WHERE  TABLE_NAME = ' + '''' + @tablenameout + '''' + '
		   ORDER BY ORDINAL_POSITION
		   FOR XML PATH('''')) AS T(list)'

	EXEC sp_executesql @column_list, N'@column_list nvarchar(MAX) out', @column_listOut OUT

	SET @sql = @cStat + @column_listOut + @cEnd	
	 
	BEGIN 

		BEGIN TRY		

			EXEC(@SQL)

		END TRY
		BEGIN CATCH

			ROLLBACK TRAN

		END CATCH 

	END	

	SET @cnt = @cnt + 1

END
```


