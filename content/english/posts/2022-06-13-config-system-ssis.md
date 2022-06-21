---
title: Creating a configuration system for SSIS
date: 2022-06-13T08:00:00.000+01:00
author: Rich
layout: post
categories:
- sqlserver
- ssis
draft: false
tags:
- SQLServer
- T-SQL
- SSIS
- Advanced
---

Do you have a complicated SSIS package that has steps which include configuration where things could change? File locations or rundates that need to be updated regularly and often face a whole world of pain when the infrastructure team adjust something because then you have to update every config reference in your package? 

Let's have a look at how we can use SQL Server to hold the configuration for us and SSIS bring it out at run time. 

**Note:** An assumption has been made that you know how to create a **Connection Manager** to point to your new database.

First we are going to need a database, if you already have a database where configuration is kept use that otherwise, let's create one;

```
CREATE DATABASE SSISConfig;
```

Next let's create some tables in this database to hold our config

```
CREATE TABLE dbo.Package
(
	PackageID INT IDENTITY(1,1) NOT NULL,
	PackageName varchar(100),
	PackageGUID UNIQUEIDENTIFIER,
	Description varchar(100),
	Active BIT DEFAULT 1
); 
```

We are going to need a primary key on this table so let's add that next 

```
ALTER TABLE dbo.Package ADD CONSTRAINT PK_Package_PackageID PRIMARY KEY (PackageID);
```

| Column Name  | Column Description  |
|---|---|
| PackageID  |  UniqueIdentifier of the row in this table, also the primary key |
|  PackageName | The name of the package as it is in SSIS  |
|  PackageGUID | The Package GUID from SSIS  |
|  Description |  A short description of the package |
|  Active |  If this package configuration is enabled |

Now we need a table to store the configuration for this package, because each package can have multiple configurations we are going to create a new table to hold that information 

```
CREATE TABLE dbo.PackageConfig
(
	[PackageID] INT NOT NULL,
	[ConfigName] varchar(50) NOT NULL
	[ConfigValue] varchar(50),
	[Active] BIT DEFAULT 1
);
```

A foreign key is also needed as we only want configurations to be added for packages that exist 

```
ALTER TABLE dbo.PackageConfig ADD CONSTRAINT FK_PackageConfig_PackageID FOREIGN KEY (PackageID) REFERENCES [dbo].[Package] (PackageID);
```

| Column Name  | Column Description  |
|---|---|
| PackageID  |  Reference to the package in dbo.Package |
|  ConfigName | The name of the configuration item  |
|  ConfigValue | The value of the configuration item  |
|  Active |  Is the configuration item enabled or not |

Now that we have the configuration tables set up in SQL, we need to add the logic to get the configuration from SSIS, at the end we should have something that looks like this 

![](/img/ssis-config-1.png)

Let's break this down.

### Variables

First, we need a couple of variables to store the configuration we receive from SQL

| Variable Name  | Variable Data Type  | Variable Description
|---|---|---|
| ConfigName  |  String | The name of the configuration item in SQL prefixed with Confg_ |
|  ConfigValue | String  | The value of the configuration item |
|  Configurations | System.Object  | An object to store the results from the query |

### Get Config 

Bring an **Execute SQL Task** event onto the canvas and double click it 

![](/img/ssis-execute-sql-task.png)

Once open set the result set to **Full Result Set** because we want to store the results from the query into the Configurations variable object we created earlier.

![](/img/ssis-config-2.png)

Click in the **SQL Statement** box and add the following code

```
SELECT 

pc.ConfigName,
pc.ConfigValue

FROM PackageConfig PC 

INNER JOIN Package P 
	ON PC.PackageID = P.PackageID
WHERE 
	p.PackageGUID = ? AND pc.Active = 1
```

The question mark in the SQL query above will take the value from the Parameter mapping which we will setup in a moment. 

![](/img/ssis-config-3.png)

Now click Result set from the left hand side of the window 

![](/img/ssis-config-4.png)

In the ResultName type a name for the results "Configurations" works, then select the Configurations variable we created before.

![](/img/ssis-config-5.png)

Click Parameter Mappings from the sidebar 

![](/img/ssis-config-6.png)

Click Add, and add in the System::PackageID as shown below

![](/img/ssis-config-7.png)

Once that is all done, click OK.

### Clear Config Values

Bring a script task onto the canvas and double click it 

![](/img/ssis-script-task.png)

![](/img/ssis-config-8.png)

Click the **Edit Script** button

Inside the Main method we need to put the following code, this is going to clear any of the configuration items so they are ready for use in the event they had already been populated.

```
Variables pkgVars = Dts.Variables;

foreach(Variable pkgVar in pkgVars)
{
	string varval = pkgVar.QualifiedName.ToString();
	if(varval.StartsWith("User::confg_"))
	{
		pkgVar.Value = string.Empty;
	}
}
```

![](/img/ssis-config-8.png)

In the read write variables box add;

* User::ConfigName
* User::ConfigValue 
* User::Configurations

Click **OK** once that is done.

### Set Config Values

Drag a **Foreach loop container** onto the canvas and double click it 

![](/img/ssis-foreach-loop.png)

![](/img/ssis-config-9.png)

Set the Enumerator to **Foreach ADO Enumerator** in the ADO object source variable drop down select User::Configurations and make sure **Rows in first table** is selected

![](/img/ssis-config-10.png)

Click the variable mappings from the side bar 

![](/img/ssis-config-11.png)

Add the following variables

![](/img/ssis-config-12.png)

Once done, click OK.

Now that the Foreach loop is setup, drag a script task inside it and name it **Set Config**

![](/img/ssis-config-16.png)

Once it is there, double click it and click Edit Script

![](/img/ssis-config-8.png)

In the main method, add the following code, this code is going to set our configuration items 

```
string configName = Dts.Variables["User::ConfigName"].Value.ToString();
string configValue = Dts.Variables["User::ConfigValue"].Value.ToString();

if(Dts.Variables.Contains("confg_" + configName))
{
	Dts.Variables["User::confg_" + configName].Value = configValue;
}
```

### Adding a package to SQL

Right click on the SSIS canvas in an empty space and select properties, this should open a new pane which looks like this;

![](/img/ssis-config-17.png)

Copy the ID without the `{ }` we need that in a moment. 

```
INSERT INTO dbo.Package ([PackageName],[PackageGUID],[Description])
VALUES
('Your Package Name','The Package GUID we selected above','A Description)
```

### Adding Configuration Items

Now we want to add the configuration items that are going to be used by this SSIS package, we can do this first in SQL by inserting values into the PackageConfig table like this;

```
INSERT INTO dbo.PackageConfig (PackageID, ConfigName, ConfigValue)
VALUES
(1,'RunDate','20220612')
```
PackageID will be the ID of the package you are setting the configuration for, this is obtained from dbo.Package.

Now that we have created a value in PackageConfig called rundate we need to add that into SSIS.

![](/img/ssis-config-13.png)

All configuration items that are obtained from SQL are prefixed with **Confg_** but you don't need to add the prefix to the ConfigName in SQL itself.

Now we just need to add the new configurations to the Clear Configuration block & Set Configuration blocks.

The Clear configuration block should now look like this; 

![](/img/ssis-config-14.png)

This Set Configuration Block should now look like this; 

![](/img/ssis-config-15.png)