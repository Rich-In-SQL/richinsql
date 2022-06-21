---
title: 'Building a Formula One Race Tracker - Part One'
date: 2019-04-09T16:00:06+01:00
author: Rich
layout: post
permalink: /building-a-formula-one-race-tracker-part-one
categories:
  - 'csharp'
  - sqlserver
tags:
  - application
  - SQL
  - sqlserver
  - t-sql
---
**Contents**

  1. [Project Overview](/posts/2019-04-09-building-a-formula-one-race-tracker-part-one/)
  2. [Part Two &#8211; Database Schema](/posts/2019-04-16-building-a-formula-one-race-tracker-part-two/)

<span style="font-weight: 400;">A little while ago I answered a </span>[<span style="font-weight: 400;">question</span>](https://stackoverflow.com/questions/54277268/f1-season-database-individual-races/54296233#54296233) <span style="font-weight: 400;">over on StackOverflow from someone who was asking how they could design a database that would allow them to track individual races in a Formula One season, this question formed the foundation for this series of posts which will hopefully demonstrate my building process of a small simplistic web application, using a lot of what I have learned over the last five years.  </span>

**What do we need?**

<span style="font-weight: 400;">As we are going to be building a C# web application, there are a few things we will need before we get started;</span>

_**Note:**_ This project is available in full over on [Git Hub](https://github.com/RichInSQL/Formula1RaceTracker)

**Microsoft SQL Server**

<span style="font-weight: 400;">I am going to need a version of SQL Server, for this project I am going to be using SQL Server 2017 development edition you can grab the development edition of SQL Server from the </span>[<span style="font-weight: 400;">Visual Studio downloads</span>](https://my.visualstudio.com/downloads) <span style="font-weight: 400;">page for learning and personal development. </span>

**Visual Studio**

<span style="font-weight: 400;">This project will be written in C# in Visual Studio 2017 I am going to be using the community version which again is available from the <a href="https://my.visualstudio.com/downloads">Visual Studio downloads</a> page </span>

**The Requirements**

<span style="font-weight: 400;">Before I start to write any code I need to understand what the goals are for the project, not only so that I have a clear understanding of where the project will go and it's function but also so that I don’t stray off track and start writing additional features that could be implemented in future releases and avoid </span>[<span style="font-weight: 400;">scope creep</span>](https://en.wikipedia.org/wiki/Scope_creep) <span style="font-weight: 400;">in the current release.</span>

**The Database**

<li style="font-weight: 400;">
  <span style="font-weight: 400;">A Simple web application that will allow for</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Inputting of race results and recording results against individual drivers</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Viewing an overall table of race results for a given season</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">The ability to record each race split across an entire weekend. </span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Practice Sessions</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Qualification</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Race Day</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">The ability for the application to calculate the points for a driver based on their finishing position.</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">1 (25)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">2 (18)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">3 (15)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">4 (12)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">5 (10)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">6 (8)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">7 (6)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">8 (4)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">9 (2)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">10 (1)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">FL (1)</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">The ability to track drivers and the team in which they are driving for between seasons.</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Drivers will have a start and end date for each team they race for.</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">A driver with an end date populated for a team will indicate that the driver no longer drives for that team. </span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">The ability to track which engines teams are using between seasons.</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Engines much like drivers will have a start and end date for each team.</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">A populated end date will mean the engine is no longer in use for this team. </span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">The ability to track race circuits and include them or not include them for an individual season. </span>
</li>

**The Web Application**

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Lightweight and fast loading</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Easy to use on both desktop & mobile</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Responsive</span>
</li>

**The Design**

<span style="font-weight: 400;">To start I always like to draw out what the database will look like on paper, for me it gives a good starting point of what tables the database will be made up of, which tables will need to talk to one another, which columns will become primary keys, if we will have any default values and the data types of the columns.   </span>

**The Database Design**

The first sketch that I did was a mapping of the tables, It didn't include any of the data types, what I decided to do was to map out the columns on paper, transfer that into Google Sheets and map the data types to the columns there.

![](/img/F1-Database-Design.jpg)

Once the initial drawing of the table structure was done I completed the mapping of the table data types below are all of the final data types and mappings that I came up with in Google Sheets, it outlines everything that I will need for Part Two, which is where the actual schema will be written.

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Driver Table&quot;}">
      Driver Table
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Driver_ID&quot;}">
      Driver_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Forename&quot;}">
      Forename
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:20}">
      20
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Surname&quot;}">
      Surname
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:20}">
      20
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Nationality&quot;}">
      Nationality
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:20}">
      20
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Retired&quot;}">
      Retired
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;BIT&quot;}">
      BIT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:0}">
    </td>
  </tr>
</table>

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Driver_Team&quot;}">
      Driver_Team
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Team_ID&quot;}">
      Team_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Driver_ID&quot;}">
      Driver_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Start_Date&quot;}">
      Start_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;End_Date&quot;}">
      End_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
</table>

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Teams&quot;}">
      Teams
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Team_ID&quot;}">
      Team_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Team_Name&quot;}">
      Team_Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:20}">
      20
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Active&quot;}">
      Active
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;BIT&quot;}">
      BIT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:1}">
      1
    </td>
  </tr>
</table>

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Team_Engine&quot;}">
      Team_Engine
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Team_ID&quot;}">
      Team_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Engine&quot;}">
      Engine
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:15}">
      15
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Start_Date&quot;}">
      Start_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;End_Date&quot;}">
      End_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
</table>

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Circuit&quot;}">
      Circuit
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Circuit_ID&quot;}">
      Circuit_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Circuit_Name&quot;}">
      Circuit_Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;NVARCHAR&quot;}">
      NVARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:60}">
      60
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Circuit_Type&quot;}">
      Circuit_Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:15}">
      15
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Direction&quot;}">
      Direction
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;VARCHAR&quot;}">
      VARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:50}">
      50
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Circuit_Location&quot;}">
      Circuit_Location
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;NVARCHAR&quot;}">
      NVARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:50}">
      50
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Last_Length_Used&quot;}">
      Last_Length_Used
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DECIMAL&quot;}">
      DECIMAL
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;5,3&quot;}">
      5,3
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Grand_Prix_Name&quot;}">
      Grand_Prix_Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;NVARCHAR&quot;}">
      NVARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:70}">
      70
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Start_Date&quot;}">
      Start_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;End_Date&quot;}">
      End_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
</table>

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race Types&quot;}">
      Race Types
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race_Type_ID&quot;}">
      Race_Type_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race_Type&quot;}">
      Race_Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;NVARCHAR&quot;}">
      NVARCHAR
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:3,&quot;3&quot;:10}">
      10
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
</table>

<table dir="ltr" border="1" cellspacing="0" cellpadding="0">
  <colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup> <tr>
    <td colspan="7" rowspan="1" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race&quot;}">
      Race
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Column Name&quot;}">
      Column Name
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Type&quot;}">
      Data Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Data Length&quot;}">
      Data Length
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;PK&quot;}">
      PK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;FK&quot;}">
      FK
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Identity&quot;}">
      Identity
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Default&quot;}">
      Default
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race_ID&quot;}">
      Race_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race_Date&quot;}">
      Race_Date
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;DATETIME&quot;}">
      DATETIME
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Driver_ID&quot;}">
      Driver_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Circuit_ID&quot;}">
      Circuit_ID
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Yes&quot;}">
      Yes
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Final_Position&quot;}">
      Final_Position
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Points&quot;}">
      Points
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Race_Type&quot;}">
      Race_Type
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;TINYINT&quot;}">
      TINYINT
    </td>
    
    <td>
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;No&quot;}">
      No
    </td>
    
    <td>
    </td>
  </tr>
</table>

&nbsp;

**The Application Design**

I sketched out a very rough drawing of what I wanted the web application to look like, as you can see it is going to be very simple, made up of just two pages, a submissions page and a results page which can be accessed from either of the pages, each page will have two buttons, one at the top of the page and one at the bottom, the submission page will be made up of a web form

![](/img/F1-Application-Design.jpg)

<span style="font-weight: 400;">As outlined in our requirements the application needs to be simple, responsive and fast loading, with that in mind we are going to make use of <a href="https://getbootstrap.com">Bootstrap</a>, specifically version 4. </span>

<span style="font-weight: 400;">Using the above rough sketch I set about writing the HTML for the pages, why? Once the design of the application is done I like to develop out the idea in pure HTML/CSS so I can see how the application will look, when it comes to adding in the functionality the existing HTML that was developed at this stage can be used simply adding the page functionality.</span>

<span style="font-weight: 400;">This is what the pages will look like;</span>

![](/img/Formula-One-Race-Tracker-Submission-Initial.png)

![](/img/Formula-One-Race-Tracker-Results-Initial.png)