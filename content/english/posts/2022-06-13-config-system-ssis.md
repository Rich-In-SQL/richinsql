---
title: Creating a configuration system for SSIS
date: 2022-06-13T09:00:00.000+01:00
image: "/images/post/apple-health-powerBi-dashboard.jpg"
author: Rich
layout: post
categories:
- sqlserver
- ssis
draft: true
tags:
- SQLServer
- T-SQL
- SSIS
- Advanced
---


First we are going to need a database, if you already have a database where configuration is kept use that otherwise, let's create one

```
CREATE DATABASE dbo.Package
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
CREATE DATABASE dbo.PackageConfig
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

| Variable Name  | Variable Data Type  | Variable Desciption
|---|---|
| ConfigName  |  String | Reference to the package in dbo.Package |
|  ConfigValue | String  | Reference to the package in dbo.Package |
|  Configurations | System.Object  | Reference to the package in dbo.Package |

### Get Config 

Bring an *Execute SQL Task* event onto the canvas and double click it 

![](/img/ssis-config-2.png)

Set the result set to **Full Result Set** becuase we want to store the results from the query into the object we created earlier.

Click in the SQL Statement box and add the following code

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

![](/img/ssis-config-6.png)

![](/img/ssis-config-7.png)

### Clear Config Values

Bring a script task onto the canvas and double click it 

![](/img/ssis-config-8.png)

Click the Edit Script button

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

Click ok once that is done.

### Set Config Values

Drag a **Foreach loop container** onto the canvas and double click it 

![](/img/ssis-config-9.png)

![](/img/ssis-config-10.png)

![](/img/ssis-config-11.png)

![](/img/ssis-config-12.png)

Now that the Foreach loop is setup, drag a script task inside it once it is there, double click it and click Edit Script

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

### Adding Configuration Items 

In SQL we can do this by inserting values into the PackageConfig table 

```
INSERT INTO dbo.PackageConfig (PackageID, ConfigName, ConfigValue)
VALUES
(1,'RunDate','20220612')
```

PackageID will be the ID of the package you are setting the configuration for, this is obtained from dbo.Package.

Now that we have created a value in PackageConfig called run date we need to add that into SSIS

![](/img/ssis-config-13.png)

All configuration items that are obtained from SQL are prefixed with **Confg_** but you don't need to add the prefix to the ConfigName in SQL itself.

Now we just need to add the new configurations to the Clear Configuration block & Set Configuration blocks

The Clear configuration block should now look like this 

![](/img/ssis-config-14.png)

This Set Configuration Block should now look like this 

![](/img/ssis-config-15.png)