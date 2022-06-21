---
title: 'Building a Formula One Race Tracker - Part Two'
date: 2019-04-16T16:00:57+01:00
author: Rich
layout: post
permalink: /building-a-formula-one-race-tracker-part-two
categories:
  - sqlserver
  - csharp
tags:
  - application
  - SQL
  - sqlserver
  - t-sql
---
**Contents**

  1. [<span style="font-weight: 400;">Part One &#8211; Project Overview</span>](/posts/2019-04-09-building-a-formula-one-race-tracker-part-one/)
  2. [<span style="font-weight: 400;">Part Two &#8211; Database Schema</span>](/posts/2019-04-16-building-a-formula-one-race-tracker-part-two/)

### The Database Schema

In Part One we looked at the project scope, why I was building this project, how the database was going to look, the columns and their respective datatypes and a mock-up of the application design, in this part I am going to show you how I went about populating the database with the objects required and then filling them objects with referential data.

```
    Use Master

    CREATE DATABASE F1Tracker;

    CREATE LOGIN [RaceTracker] WITH PASSWORD=N'StrongPassword1234!', DEFAULT_DATABASE=[F1Tracker], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF;

    GO

    USE F1Tracker;

    CREATE USER [RaceTracker] FOR LOGIN [RaceTracker];
    ALTER ROLE [db_owner] ADD MEMBER [RaceTracker];

    GO
```

First up I need to create a database to hold all of my objects, along with that I also need to create a login which will be used later on in the project to connect to the database, remembering to change the password to something other than what I have specified here.

```
    CREATE TABLE Drivers
    (
        Driver_ID INT NOT NULL,
        Forename varchar(20),
        Surname varchar(20),
        Nationality varchar(20), 
        Retired BIT   
    );
```

The driver's table will hold all the information about the drivers;

  * First name
  * Surname
  * Nationality
  * Retired state

The table will also have a primary key which I have set as the driver_id I haven't given the primary key an identity because I want to specify my integer ids for drivers.

```
    CREATE TABLE Teams
    (
        Team_ID TINYINT NOT NULL,
        Team_Name varchar(20),    
        Active BIT DEFAULT 1
    );
```

Much like the driver's table, this one hold's all the teams;

  * Team ID _**FK**_
  * Team name
  * Active Flag

Just like the driver's table, there is no identity column specified for this table as I want to specify my own integer ID's for Team_ID which is also the primary key of this table.

```
    CREATE TABLE Driver_Team
    (
        Team_ID INT,
        Driver_ID INT,
        Start_date DATETIME,
        End_date DATETIME
    );
```

The driver team table is a link table, this table is going to link all of our drivers to our teams,

  * Team ID _**FK**_ 
      * I have specified a Team\_ID which is a foreign key, this references the Team\_ID from the team's table
  * Diver ID _**FK**_ 
      * the Driver\_ID also a foreign key which references the Drivers\_ID from the driver's table
  * Start Date
  * End Date

I have also specified a start and end date for each driver with each team in this link table, this is because Formula One drivers can start and end with a particular team after each season.

_**Note:**_ If the End_Date is not populated the driver is still driving for the specified team.

```
    CREATE TABLE Team_Engine
    (
        Team_ID TINYINT,
        Engine varchar(15),
        Start_Date DATETIME,
        End_date DATETIME
    );
```

As with drivers, teams in Formula One can often change who's engine they are using in their car from season to season, with this in mind I need to be able to record this, the link table Team_Engine will allow us to do that.

  * Team ID _**FK**_
  * Engine
  * Start Date
  * End Date

The Team\_ID a foreign key which references the team's table. The start date and end date work in the same way as they do in Driver\_Team.

_**Note:**_ If the End_Date is not populated the engine listed is still being used by the team specified.

```
    CREATE TABLE Circuit
    (
        Circuit_ID TINYINT NOT NULL,
        Circuit_Name nvarchar(60),
        Circuit_Type varchar(15),
        Direction varchar(50),
        Circuit_Location nvarchar(50),
        Last_length_used DECIMAL(5,3),
        Grands_Prix_Name nvarchar(70),
        Start_date DATETIME,
        End_date DATETIME 
    );
```

The Circuit table is going to hold all of the information about the circuits

  * Circuit ID _**PK**_
  * Circuit Name
  * Circuit Type 
      * Track or City
  * Direction 
      * The direction around the track which the driver's race
  * Circuit Location 
      * Location of the Circuit
  * Last Length Used 
      * The last length that was used to race
  * Grand Prix Name
  * Start Date
  * End Date

There are a small number of the specified columns used in the final application, I have captured more information than required allowing for further reporting should that be something I want to do a bit later.

_**Note:**_ If the End_Date is not populated the circuit is still included in the season roster.

```
    CREATE TABLE Race_Types
    (    
        Race_Type_ID TINYINT NOT NULL,
        Race_Type NVARCHAR(10)
    );
```

Race Types is a referential table which is going to hold the race types, this will include practice & qualification sessions, along with the actual race event, the reason for this is so I can record each stage of the drivers' weekend in the race table and differentiate between the race and the pre-race events.

Race\_Type\_ID is the primary key in this table again there is no identity specified as I want each referential row to have an integer id of my choosing.

```
    CREATE TABLE Race
    (
        Race_ID INT IDENTITY(1,1) NOT NULL,
        Race_Date DATETIME,
        Driver_ID INT,
        Circuit_ID TINYINT,    
        Final_Position TINYINT,
        Points TINYINT,
        Season INT,
        Race_Type TINYINT
    );
```

The race table is where all of the weekend race data will get recorded, in this table we have Race_ID as the primary key however it has an identity set, starting at 1 and incrementing by 1 each time a row is inserted.

  * Race ID _**PK**_
  * Race Date
  * Driver ID _**FK**_
  * Team ID _**FK**_
  * Circuit ID _**FK**_
  * Final Position
  * Points 
      * The number of points if any accumulated for this race
  * Race Type _**FK**_

### Reference Data

The referential data that we are going to use in this project is all publically available, the majority of which can be obtained from Wikipedia. Below you will find all the data sets required for this project.

I did have to do some work to get the data into a workable state though, which I did using Google Sheets. I inserted all of the data into a sheet, one sheet for each of the referential tables I needed to build and then built a SQL insert statement directly inside Google Sheets for each row using a formula similar to this;

```
    =join(" ", arrayformula("(" & A2 & "," & "'" & B2 & "'" & "," & "'" & C2 & "'" & "," & "'" & D2 & "'" & "," & "'" & E2 & "'" & "," & "'" & F2 & "'" & "," & "'" & G2 & "'" & ")" & ","))
```

Which looked like this when complete;

![](/img/Formula-1-Drivers-Google_Sheets.png)

I have included the source to all of the referential data under each insert statement.

```
    INSERT INTO Drivers (Driver_ID,Forename,Surname,Nationality)
    VALUES
    (1,'Alexander','Albon','Thailand'),
    (2,'Valtteri','Bottas','Finland'),
    (3,'Pierre','Gasly','France'),
    (4,'Antonio','Giovinazzi','Italy'),
    (5,'Lewis','Hamilton','United Kingdom'),
    (6,'Nico','Hülkenberg','Germany'),
    (7,'Robert','Kubica','Poland'),
    (8,'Daniil','Kvyat','Russia'),
    (9,'Charles','Leclerc','Monaco'),
    (10,'Kevin','Magnussen','Denmark'),
    (11,'Lando','Norris','United Kingdom'),
    (12,'Sergio','Pérez','Mexico'),
    (13,'Kimi','Räikkönen','Finland'),
    (14,'Daniel','Ricciardo','Australia'),
    (15,'George','Russell','United Kingdom'),
    (16,'Carlos','Sainz','Spain'),
    (17,'Lance','Stroll','Canada'),
    (18,'Max','Verstappen','Netherlands'),
    (19,'Sebastian','Vettel','Germany');
```

<a href="https://en.wikipedia.org/wiki/List_of_Formula_One_drivers" target="_blank" rel="noopener noreferrer">Source</a>

```
    INSERT INTO Circuit (Circuit_ID,Circuit_Name,Circuit_Type,Direction,Circuit_Location,Last_length_used,Grands_Prix_Name)
    VALUES
    (1,'Autodromo Nazionale Monza','Race circuit','Clockwise','Monza, Italy',3.600,'Italian Grand Prix'),
    (2,'Autódromo Hermanos Rodríguez','Race circuit','Clockwise','Mexico City, Mexico',2.674,'Mexican Grand Prix'),
    (3,'Autódromo José Carlos Pace','Race circuit','Anti-clockwise','São Paulo, Brazil',2.677,'Brazilian Grand Prix'),
    (4,'Bahrain International Circuit','Race circuit','Clockwise','Sakhir, Bahrain',3.363,'Bahrain Grand Prix'),
    (5,'Baku City Circuit','Street circuit','Anti-clockwise','Baku, Azerbaijan',3.730,'European Grand Prix, Azerbaijan Grand Prix'),
    (6,'Circuit de Barcelona-Catalunya','Race circuit','Clockwise','Montmeló, Spain',2.892,'Spanish Grand Prix'),
    (7,'Circuit de Monaco','Street circuit','Clockwise','Monte Carlo, Monaco',2.074,'Monaco Grand Prix'),
    (8,'Circuit de Spa-Francorchamps','Race circuit','Clockwise','Stavelot, Belgium',4.352,'Belgian Grand Prix'),
    (9,'Circuit Gilles Villeneuve','Street circuit','Clockwise','Montreal, Canada',2.710,'Canadian Grand Prix'),
    (10,'Circuit of the Americas','Race circuit','Anti-clockwise','Austin, Texas, United States',3.426,'United States Grand Prix'),
    (11,'Circuit Paul Ricard','Race circuit','Clockwise','Le Castellet, France',3.630,'French Grand Prix'),
    (12,'Hockenheimring','Race circuit','Clockwise','Hockenheim, Germany',2.842,'German Grand Prix'),
    (13,'Hungaroring','Race circuit','Clockwise','Mogyoród, Hungary',2.722,'Hungarian Grand Prix'),
    (14,'Marina Bay Street Circuit','Street circuit','Anti-clockwise','Singapore',3.146,'Singapore Grand Prix'),
    (15,'Melbourne Grand Prix Circuit','Street circuit','Clockwise','Melbourne, Australia',3.295,'Australian Grand Prix'),
    (16,'Red Bull Ring (formerly A1-Ring and Österreichring)','Race circuit','Clockwise','Spielberg bei Knittelfeld, Austria',2.683,'Austrian Grand Prix'),
    (17,'Shanghai International Circuit','Race circuit','Clockwise','Shanghai, China',3.387,'Chinese Grand Prix'),
    (18,'Silverstone Circuit','Race circuit','Clockwise','Silverstone, United Kingdom',3.660,'British Grand Prix'),
    (19,'Sochi Autodrom','Road circuit','Clockwise','Sochi, Russia',3.634,'Russian Grand Prix'),
    (20,'Suzuka Circuit','Race circuit','Clockwise and Anti-clockwise (figure eight)','Suzuka, Japan',3.608,'Japanese Grand Prix'),
    (21,'Yas Marina Circuit','Race circuit','Anti-clockwise','Abu Dhabi, United Arab Emirates',3.451,'Abu Dhabi Grand Prix');
```

<a href="https://en.wikipedia.org/wiki/List_of_Formula_One_circuits" target="_blank" rel="noopener noreferrer">Source</a>

```
    INSERT INTO Race_Types (Race_Type_ID, Race_Type)
    VALUES
    (1,'Practice 1'),
    (2,'Practice 2'),
    (3,'Practice 3'),
    (4,'Qualifying'),
    (5,'Race');
```

The race types I populated manually as there is no source available for this data

```
    INSERT INTO Teams (Team_ID,Team_Name)
    VALUES
    (1,'Alfa Romeo'),
    (2,'Ferrari'),
    (3,'Haas'),
    (4,'McLaren'),
    (5,'Mercedes'),
    (6,'Racing Point'),
    (7,'Red Bull Racing'),
    (8,'Renault'),
    (9,'Scuderia Toro Rosso'),
    (10,'Williams');
```

<a href="https://en.wikipedia.org/wiki/List_of_Formula_One_constructors" target="_blank" rel="noopener noreferrer">Source</a>

```
    INSERT INTO Team_Engine (Team_ID, Engine,Start_date)
    VALUES
    (1,'Ferrari',GetDate()),
    (2,'Ferrari',GetDate()),
    (3,'Ferrari',GetDate()),
    (4,'Renault',GetDate()),
    (5,'Mercedes',GetDate()),
    (6,'BWT Mercedes',GetDate()),
    (7,'Honda',GetDate()),
    (8,'Renault',GetDate()),
    (9,'Honda',GetDate()),
    (10,'Mercedes',GetDate());
```

The Team_Engine table is made up of one row for each team, followed by the Engine they are currently using with a start date of the insert date, this can be changed to the date that the team actually started to use that engine if required, end date is left unpopulated as each team is currently using the engine's specified.

```
  INSERT INTO Driver_Team (Driver_ID,Team_ID,Start_Date)
    VALUES
    (1,9,GETDATE()),
    (2,5,GETDATE()),
    (3,7,GETDATE()),
    (4,1,GETDATE()),
    (5,5,GETDATE()),
    (6,8,GETDATE()),
    (7,10,GETDATE()),
    (8,9,GETDATE()),
    (9,2,GETDATE()),
    (10,2,GETDATE()),
    (11,4,GETDATE()),
    (12,6,GETDATE()),
    (13,1,GETDATE()),
    (14,8,GETDATE()),
    (15,10,GETDATE()),
    (16,4,GETDATE()),
    (17,6,GETDATE()),
    (18,7,GETDATE()),
    (19,2,GETDATE());
```

The reference data for the Driver\_Team link table is built by taking the Drivers Team\_ID from the Teams table and the ID of the Driver from the driver's table and inserting them into a single row per driver, the result of which can be seen below.

[<img class="alignnone size-full wp-image-352 img-fluid " src="/img/Formula-1-Drivers-Driver-Team-Query1.png" alt="Driver Team Table" srcset="/img/Formula-1-Drivers-Driver-Team-Query1.png 361w, /img/Formula-1-Drivers-Driver-Team-Query1-252x300.png 252w" sizes="(max-width: 361px) 100vw, 361px" />](/img/Formula-1-Drivers-Driver-Team-Query1.png)

### The Keys

Now that I have all of the tables created and the data populated I need to make sure that all the keys are applied, as per the design I need to apply a number of primary keys to the defined columns in each table, this has to be done before the foreign keys can be defined.

```
    ALTER TABLE Drivers ADD CONSTRAINT PK_Drivers_ID PRIMARY KEY (Driver_ID);
    ALTER TABLE Circuit ADD CONSTRAINT PK_Circuit_ID PRIMARY KEY (Circuit_ID);
    ALTER TABLE Race_Types ADD CONSTRAINT PK_Race_Type_Ref PRIMARY KEY (Race_Type_ID);
    ALTER TABLE Teams ADD CONSTRAINT PK_Team_ID PRIMARY KEY (Team_ID);
    ALTER TABLE Race ADD CONSTRAINT PK_Race_ID PRIMARY KEY (Race_ID);
```

Now that the primary keys are set I can specify the foreign keys for the tables that require them

```
    ALTER TABLE Team_Engine ADD CONSTRAINT FK_Team_ID FOREIGN KEY (Team_ID) REFERENCES Teams (Team_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_Type_TypeRef FOREIGN KEY (Race_Type) REFERENCES Race_Types(Race_Type_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_DriveID FOREIGN KEY (Driver_ID) REFERENCES Drivers(Driver_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_Team FOREIGN KEY (Team_ID) REFERENCES Teams(Team_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_CircuitID FOREIGN KEY (Circuit_ID) REFERENCES Circuit(Circuit_ID);
```

The reason that I like to add the constraints for primary and foreign keys outside of the table creation is that it allows me to specify a custom name for the constraint because if I was to simply execute

```
    CREATE TABLE Table1
    ( 
    Test_ID INT IDENTITY(1,1) PRIMARY KEY
    )
```

SQL Server would specify a key name for me which when troubleshooting error's relating to key constraints in my queries makes finding the offending key difficult, especially in larger tables with many keys spanning many tables.

### Let's Test

Now that the table schema is created and we have populated the referential data that we require we can run a small test against that data to make sure that the data we expect is returned, below is a query that will return all drivers and their current team from the Driver_Team link table making use of the foreign keys.

```
    SELECT 
      D.Forename,
      D.Surname,
      D.Nationality,
      T.Team_Name,
      DT.Start_date,
      DT.End_date 
    FROM 
      Driver_Team DT 

    INNER JOIN Drivers D ON 
      D.Driver_ID = DT.Driver_ID 
      
    INNER JOIN Teams T ON 
      T.Team_ID = DT.Team_ID
```

You should get some like this in the output

![](/img/Formula-1-Drivers-Driver-Team-Query.png)

In future seasons if a driver moves between teams each driver will have multiple rows returned but as explained above the current team will be the one where the End_Date is NULL.

Next up is how I built the stored procedures needed to get the data in and out of the database.