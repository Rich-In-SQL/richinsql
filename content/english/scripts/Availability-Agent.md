---
title: "Availability Agent"
date: 2022-02-01T00:00:00Z
description: "Availability agent will check which node in an availability group is master and disable jobs on secondary nodes."

weight: 1
categories: ["sqlserver"]

thumbnail: "images/scripts/availability-agent.png"
tools_github: "https://github.com/Rich-In-SQL/Availability-Agent"

tools_info:
- title: "Types :"
  content: "T-SQL"
#- title: "Colors :"
#  content: "Blue, Purple, White, Orange"

tools_images:
- image: "images/scripts/availability-agent.png"
#- image: "images/tools/figma.jpg"
---

Availabiltiy Agent is a T-SQL script that is designed to be created as a stored procedure and referenced inside a SQL Agent Job, 
The agent job will then run every nMinutes to disable jobs on the secondary node that would otherwise cause a job failure.

### Downloading

The script can be downloaded from the projects GitHub, the link of which can be found in the sidebar of this post.

### Requirements


- Ability to create stored procedures
- Ability to create tables
- Ability to drop stored procedures
- A database with the name DBA_Tasks
- Schema in the above mentioned database called DBA
- SQL Server 2008+

When running this script if the an object exists in the DBA_Tasks database under the schema DBA with the name DBA.ExcludedJobCheck it will be dropped and this stored procedure created in it's place.

### Installation 

These steps should be first carried out on the primary node in your availabiltiy group.

1. Ensure your connected to the primary node in your availability group
2. Change database context to the database you would like the soloution installed within.
3. Run the ExcludedJobCheck.sql script from the zip file, this assumes you have a database called DBA_Tasks which is where the stored procedure will resdide. If this is no the case change ```@database_name=N'DBA_Tasks'``` to the database name of your choosing. 

### Configuration

Once the soloution has been installed, you should find a table inside the database called ```[dbo].[ExcludedJobs]``` this holds all of the agent jobs you want to be in scope of the soloution. 

To add a job simply run the below insert adding in the name of your jobs where required

```
INSERT INTO [dbo].[ExcludedJobs] ([Job_name])
VALUES
('YourJobName')
```

If you have some jobs already in this table from a previous installation that you would like to move out of scope, simply set the Active flag to 0 

```
UPDATE [dbo].[ExcludedJobs] SET Active = 0 WHERE ID = 1
```

Setting the active flag to 0 will ensure that the job isn't in scope of the soloution when it runs, this will leave the agent job active on all nodes in the availability group.