---
title: "Formula One Database"
date: 2021-12-27T05:00:00Z
draft: false
description: "A Microsoft SQL Server database of Formula One information for training and development."

weight: 2
categories: ["SQL Server"]

thumbnail: "images/tools/f1-db.jpg"
tools_website_link: "https://github.com/rich-howell/FormulaOneDatabase"

tools_info:
- title: "Types :"
  content: "Database"
#- title: "Colors :"
#  content: "Blue, Purple, White, Orange"
---

Do you have an interest in Formula One? Do you need a database for T-SQL demonstrations or testing of your T-SQL code that is open source and not the adventure works database? I might just have the answer, I have put together using freely available data a database for just that.

There are not massive amounts of rows in this dataset so it won't be good for performance testing but learning to write T-SQL and following along with some of my posts - it will be perfect.

Let me introduce, the un-official Formula One Database.

### What Is Included 

**A table holding a list of all the drivers and their details**
* Drivers 
  - DriverID **PK**
  - Nationality
  - Races Entered
  - Races Started
  - Pole Positions
  - Race Wins
  - Podiums
  - Fastest Laps
  - Total Points

**A table that holds a list of Drivers and the seasons they raced**
* Driver Seasons
  - DriverID **FK (Drivers)**
  - SeasonRefID **FK**

**A table that holds which driver was the champion for which season**
* Driver Championships
  - DriverID **FK (Drivers)**
  - SeasonRefID **FK (Seasons)**

**The standing of each driver for each season**
* Driver Standings
  - DriverID **FK (Drivers)**
  - SeasonRefID **FK (Seasons)**
  - Points

**List of each constructor and the details of them constructors**
* Constructors
  - ConstructorID **PK**
  - Country Licensed
  - Races Entered
  - Races Started
  - Drivers
  - Total Entries
  - Wins
  - Points
  - Poles
  - Fastest Laps
  - Podiums
  - Constructor Championships Won
  - Driver Championships Won

**A list of all the constructors and the seasons they were active**
* Constructor Seasons
  - ConstructorID **FK (Constructors)**
  - SeasonRefID **FK (Seasons)**

**Each constructor has a registered nationality, this table holds each constructor and the nationality they are registered in**
* Constructor Nationality
  - ConstructorID **FK (Constructors)**
  - CountryID **FK (Countrys)**

**This table holds the points for each constructor for each season (currently just 2021) with positions in the championship**
* Constructor Standings 
  - ConstructorID **FK (Constructors)**
  - SeasonRefID **FK (Seasons)**
  - Points

**Each circuit and the details of that circuit**
* Circuits
  - CircuitID **PK**
   - Circuit
   - Grand Prix
   - TypeRefID **FK (CircuitTypes)**
   - DirectionRefID **FK (Directions)**
   - LastLengthUsed
   - GrandPrixHeld

**A list of all circuit locations, the CountryID is provided along with the Geographical Longitude & Latitude**
* Circuits Location
  - CircuitID
  - CountryID **FK (Countrys)**
  - Longitude
  - Latitude

**Each circuit has a map of the layout, this table holds all of them images, hosted off database**
* Circuit Images
  - CircuitID **FK (Circuits)**
  - ImageURL

**Each circuit and the season they participated**
* Circuit Seasons
  - CircuitID **FK (Circuits)**
  - SeasonRefID **FK (Seasons)**

**The type of each circuit**
* Circuit Type
  - CircuitID **FK (Circuits)**
  - TypeRefID **FK (CircuitTypes)**  

**Each circuit can be clockwise or anti-clockwise direction for race day this table holds them directions for reference in the Circuits Table**  
* Circuit Directions
  - CircuitID **FK (Circuits)**
  - DirectionRefID **FK (Directions)**
  - Type
    - Street circuit
    - Road circuit
    - Street circuit
    - Race circuit

  **Reference table for seasons used in parent tables**
* Seasons
  - SeasonID
  - SeasonRefID **PK**
  - Season
    - 1950 - 2023 

  **Reference table for countries used in parent tables**
* Country
  - CountryID
  - CountryRefID **PK**
  - Country

### Database Diagram

### Found An Issue?

If you have found an issue with the data, something isn't quite right or I have made a mistake during the import, feel free to [open an Issue](https://github.com/rich-howell/FormulaOneDatabase/issues) on the [GitHub repo](https://github.com/rich-howell/FormulaOneDatabase). 

If you would like to fix the issue yourself, I would also encourage you to create a pull request, the repo is public and a pull request would really help take this project forward. 

### Pull Requests 

If you would like to make a suggested change to the database, be that the schema or the data itself you can submit a pull request which I can then review and merge into the Main branch once tested. 

### This Sounds Cool, Where Can I Get It?

If you would like to create the database on your own instance of SQL Server you can download the installation script from the [GitHub repo](https://github.com/rich-howell/FormulaOneDatabase) and run it on your SQL Server Instance.

### Do you have some example scripts?

I could make some example scripts on how to use the database, but what would be the fun in that, the documentation above should give enough information to be able to write T-SQL queries. If you do need some help feel free to post a comment below or drop me an email and I will see if I can help you out.