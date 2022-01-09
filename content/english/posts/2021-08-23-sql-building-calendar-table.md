---
title: Building a calendar table
date: 2021-08-23T09:00:00+01:00
author: Rich
layout: post
permalink: /sql-building-calendar-table
categories:
  - sqlserver
tags:
  - storedproc
  - calendar
  - SQL
---

I have been working on a project at work that needed to make use of a calendar, the data warehouse already has a calendar table but it wasn't fit for my needs, some columns that I wanted for my soloution were missing etc. so I set about creating my own. 

<!--more-->

I used the [following code](https://stackoverflow.com/questions/5635594/how-to-create-a-calendar-table-for-100-years-in-sql) as a starting point and built upon that to get me a calendar table that would be fit for my application.

It is worth noting that the below is a stored procedure, the script will create a procedure called p_BuildCalendarTable which accepts varchar(4) parameters for both *@StartDate* and *@EndDate*, I did this so that I could have the calendar build process inside my application and just pass the years as and when I wanted to update to refresh the calendar.

The start date would be the date you want the calendar to start, for example 1999 and the end date would be when you want the calendar to end, in my example I used 2040, this is what that would look like when you come to executing it. 

```
EXEC p_BuildCalendarTable @StartDate = '1999', @EndDate = '2040'
```

This is also built specifically for the United Kingdom so holidays would be different depending on your region and working days might be different for your use case depending on if you class a Saturday/Sunday as a working day.

Here is the code, I also have it on my [Github](https://github.com/RichInSQL/SQL-Calendar), which is where you can find the latest version, should I change anything. 

**DISCLAIMER** WHILE A LARGE PORTION OF THIS CODE IS MINE THE MAJORITY OF IT WAS TAKEN FROM [HERE](https://stackoverflow.com/questions/5635594/how-to-create-a-calendar-table-for-100-years-in-sql) I HAVE AMENDED IT TO SUIT MY NEEDS AND WRAPPED IT IN A STORED PROCEDURE.

```
CREATE PROCEDURE [Auxilary].[p_BuildCalendarTable]

@StartDate varchar(4),
@EndDate varchar(4)

AS

BEGIN
	SET NOCOUNT ON;
    DECLARE @SQL nvarchar(MAX)

    IF (NOT EXISTS (SELECT * 
                 FROM INFORMATION_SCHEMA.SCHEMATA 
                 WHERE SCHEMA_NAME = 'Auxilary'))

    BEGIN
		
		SET @SQL = NULL
        SET @SQL = 'CREATE SCHEMA Auxilary'
		EXEC sp_executesql @SQL

    END

    IF (NOT EXISTS (SELECT * 
                    FROM INFORMATION_SCHEMA.TABLES 
                    WHERE TABLE_SCHEMA = 'Auxilary' 
                    AND  TABLE_NAME = 'Calendar'))

    BEGIN

        CREATE TABLE [Auxilary].[Calendar] 
        (
        [Date] datetime NOT NULL,
        [Year] int NOT NULL,
        [Quarter] int NOT NULL,
        [Month] int NOT NULL,
		[MonthName] varchar(10),
		[MonthStartDate] DATETIME,
		[MonthEndDate] DATETIME,
        [Week] int NOT NULL,
        [WeekOfMonth] int NULL,
		[WeekStartDate] DATETIME,
		[WeekEndDate] DATETIME,
        [Day] int NOT NULL,
		[DayOfWeekName] varchar(10),
        [DayOfYear] int NOT NULL,
        [Weekday] int NOT NULL,
        [IsWorkingDay] [BIT],
        [IsHoliday] BIT,
        [Fiscal_Year] int NOT NULL,
        [Fiscal_Quarter] int NOT NULL,
        [Fiscal_Month] int NOT NULL
        PRIMARY KEY CLUSTERED ([Date])
        );

        ALTER TABLE [Auxilary].[Calendar]

        ADD CONSTRAINT [Calendar_ck] CHECK (  ([Year] > 1900)
        AND ([Quarter] BETWEEN 1 AND 4)
        AND ([Month] BETWEEN 1 AND 12)
        AND ([Week]  BETWEEN 1 AND 53)
        AND ([Day] BETWEEN 1 AND 31)
        AND ([DayOfYear] BETWEEN 1 AND 366)
        AND ([Weekday] BETWEEN 1 AND 7)
        AND ([Fiscal_Year] > 1900)
        AND ([Fiscal_Quarter] BETWEEN 1 AND 4)
        AND ([Fiscal_Month] BETWEEN 1 AND 12))

    END

    ELSE

    BEGIN

        TRUNCATE TABLE [Auxilary].[Calendar]

    END 

        /* CREATE THE FUNCTIONS WE NEED */

        IF (NOT EXISTS(SELECT * FROM sys.objects WHERE Type IN ('IF','TF','FN') and Name = 'Computus'))

        BEGIN

			SET @SQL = NULL
			SET @SQL = 
            'CREATE FUNCTION Auxilary.Computus
            
            (
                @Y INT -- The year we are calculating easter sunday for
            )
            RETURNS DATETIME
            AS
            BEGIN
                DECLARE
                    @a INT,
                    @b INT,
                    @c INT,
                    @d INT,
                    @e INT,
                    @f INT,
                    @g INT,
                    @h INT,
                    @i INT,
                    @k INT,
                    @L INT,
                    @m INT
                SET @a = @Y % 19
                SET @b = @Y / 100
                SET @c = @Y % 100
                SET @d = @b / 4
                SET @e = @b % 4
                SET @f = (@b + 8) / 25
                SET @g = (@b - @f + 1) / 3
                SET @h = (19 * @a + @b - @d - @g + 15) % 30
                SET @i = @c / 4
                SET @k = @c % 4
                SET @L = (32 + 2 * @e + 2 * @i - @h - @k) % 7
                SET @m = (@a + 11 * @h + 22 * @L) / 451
                RETURN(DATEADD(month, ((@h + @L - 7 * @m + 114) / 31)-1, cast(cast(@Y AS VARCHAR) AS Datetime)) + ((@h + @L - 7 * @m + 114) % 31))
            END'

			EXEC sp_executesql @SQL

        END 
        
        IF (NOT EXISTS(SELECT * FROM sys.objects WHERE Type IN ('IF','TF','FN') and Name = 'Numbers'))

        BEGIN


			SET @SQL = NULL
			SET @SQL =
            'CREATE FUNCTION Auxilary.Numbers
            (
                @AFrom INT,
                @ATo INT,
                @AIncrement INT
            )
                RETURNS @RetNumbers TABLE
            (
                [Number] int PRIMARY KEY NOT NULL
            )
            AS
            BEGIN
                WITH Numbers(n)
                AS
                (
                    SELECT 
                        @AFrom AS n
                    UNION ALL
                    SELECT 
                        (n + @AIncrement) AS n
                    FROM 
                        Numbers
                    WHERE
                        n < @ATo
                )
                INSERT @RetNumbers
                SELECT 
                    n 
                FROM 
                    Numbers
                OPTION(MAXRECURSION 0)
                RETURN;
            END'

			EXEC sp_executesql @SQL
        
        END

        IF (NOT EXISTS(SELECT * FROM sys.objects WHERE Type IN ('IF','TF','FN') and Name = 'iNumbers'))

        BEGIN


			SET @SQL = NULL
			SET @SQL = 
            'CREATE FUNCTION Auxilary.iNumbers
            (
                @AFrom INT,
                @ATo INT,
                @AIncrement INT
            )
            RETURNS TABLE
            AS
            RETURN
            (
            WITH Numbers(n)
            AS
            (
                SELECT 
                    @AFrom AS n
                UNION ALL
                SELECT 
                    (n + @AIncrement) AS n
                FROM 
                    Numbers
                WHERE
                    n < @ATo
            )
                SELECT 
                    n AS Number 
                FROM 
                    Numbers
            )'

			EXEC sp_executesql @SQL
        
        END

        /* POPULATE THE CAL TABLE */

        SET DATEFIRST 1;

        WITH Dates(Date)
        -- A recursive CTE that produce all dates between the dates provided
        AS
        (
            SELECT 
                cast(@StartDate AS DateTime) Date 
            UNION ALL                           
            SELECT 
                (Date + 1) AS Date
            FROM 
                Dates
            WHERE
            Date < cast(@EndDate AS DateTime) -1
        ),

        DatesAndThursdayInWeek(Date, Thursday)
        -- The weeks can be found by counting the thursdays in a year so we find
        -- the thursday in the week for a particular date
        AS
        (
            SELECT
                Date,
                CASE DATEPART(weekday,Date)
                    WHEN 1 THEN Date + 3
                    WHEN 2 THEN Date + 2
                    WHEN 3 THEN Date + 1
                    WHEN 4 THEN Date
                    WHEN 5 THEN Date - 1
                    WHEN 6 THEN Date - 2
                    WHEN 7 THEN Date - 3
                END AS Thursday
            FROM 
                Dates
        ),

        Weeks(Week, Thursday)
        -- Now we produce the weeknumers for the thursdays
        -- ROW_NUMBER is new to SQL Server 2005
        AS
        (
        SELECT 
            ROW_NUMBER() OVER(partition by year(Date) order by Date) Week, Thursday
        FROM 
            DatesAndThursdayInWeek
        WHERE 
            DATEPART(weekday,Date) = 4
        )
        INSERT INTO Auxilary.Calendar (Date,Year,Quarter,Month,Week,Day,DayOfYear,Weekday,Fiscal_Year,Fiscal_Quarter,Fiscal_Month,IsHoliday,IsWorkingDay,WeekOfMonth)
        SELECT
            d.Date,
            YEAR(d.Date) AS Year,
            DATEPART(Quarter, d.Date) AS Quarter,
            MONTH(d.Date) AS Month,
            w.Week,
            DAY(d.Date) AS Day,
            DATEPART(DayOfYear, d.Date) AS DayOfYear,
            DATEPART(Weekday, d.Date) AS Weekday,
            YEAR(d.Date) AS Fiscal_Year,
            DATEPART(Quarter, d.Date) AS Fiscal_Quarter,
            MONTH(d.Date) AS Fiscal_Month,
            CASE
            -- http://en.wikipedia.org/wiki/List_of_holidays_by_country
                WHEN (DATEPART(DayOfYear, d.Date) = 1) -- New Year's Day
                OR (d.Date = Auxilary.Computus(YEAR(Date))-2)  -- Good Friday
                OR (d.Date = Auxilary.Computus(YEAR(Date)))    -- Easter Sunday
                OR (MONTH(d.Date) = 12 AND DAY(d.Date) = 25) -- Cristmas day
                OR (MONTH(d.Date) = 12 AND DAY(d.Date) = 26) -- Boxing day
                THEN 1
                ELSE 0
            END AS IsHoliday,
            CASE 
                WHEN DATEPART(Weekday, d.Date) IN (6,7) THEN 0
                ELSE 1
                END AS IsWorkingDay,
            CASE	
                    WHEN DATEPART(dd,d.Date) < 8 THEN 1
                    WHEN DATEPART(dd,d.Date) < 15 THEN 2
                    WHEN DATEPART(dd,d.Date) < 22 THEN 3
                    WHEN DATEPART(dd,d.Date) < 29 THEN 4
                    ELSE 5
                END AS WeekOfMonth

        FROM 
            DatesAndThursdayInWeek d

            -- This join is for getting the week into the result set
            INNER JOIN Weeks w
            on d.Thursday = w.Thursday

        OPTION(MAXRECURSION 0)

        --Im the UK when Christmas, Boxing Day or New Year fall on a weekend the bank holiday is carried 

        ;WITH Christmas AS
        (
            SELECT 

                DATEADD(dd,+2,Date) as Date,
                Day,
                Weekday

            FROM 
                [Auxilary].[Calendar]

            WHERE 
                Month = 12
                AND Day = 25
                AND Weekday IN (6,7)
        )

        UPDATE d

        SET ISHoliday = 1

        FROM [Auxilary].[Calendar] d

        INNER JOIN Christmas C ON d.Date = C.Date


        ;WITH BoxingDay AS
        (
        SELECT 

            DATEADD(dd,+2,Date) as Date,
            Day,
            Weekday

        FROM 
            [Auxilary].[Calendar]

        WHERE 
            Month = 12
            AND Day = 26
            AND Weekday IN (6,7)

        )

        UPDATE d

        SET ISHoliday = 1

        FROM [Auxilary].[Calendar] d

        INNER JOIN BoxingDay B ON d.Date = B.Date

        ;WITH NewYear AS
        (
        SELECT 

            DATEADD(dd,+2,Date) as Date,
            Day,
            Weekday

        FROM 
            [Auxilary].[Calendar]

        WHERE 
            Month = 1
            AND Day = 1
            AND Weekday IN (6,7)

        )

        UPDATE d

        SET ISHoliday = 1

        FROM [Auxilary].[Calendar] d

        INNER JOIN NewYear n ON d.Date = n.Date
		
		UPDATE Auxilary.Calendar 
		SET 
		[DayOfWeekName] = DATENAME(dw,[Date]),
		[MonthName] = DATENAME(MM,[Date]),
		[MonthStartDate] = DATEADD(month, DATEDIFF(month, 0, [Date]), 0),
		[MonthEndDate] = DATEADD(month, ((YEAR([Date]) - 1900) * 12) + MONTH([Date]), -1),
		[WeekStartDate] = DATEADD(dd, -(DATEPART(dw, [Date])-1), [Date]),
		[WeekEndDate] = DATEADD(dd, 7-(DATEPART(dw, [Date])), [Date])


END
```

