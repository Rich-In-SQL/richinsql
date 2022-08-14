---
title: "Frequent Agent"
date: 2021-01-23T00:00:00Z
description: "Frequent agent will move agent job logs into a archive table of your choosing."

weight: 1
categories: ["sqlserver"]

thumbnail: "images/scripts/frequent-agent.png"
tools_github: "https://github.com/Rich-In-SQL/Frequent-Agent"

tools_info:
- title: "Types :"
  content: "T-SQL"
#- title: "Colors :"
#  content: "Blue, Purple, White, Orange"

tools_images:
- image: "images/scripts/frequent-agent.png"
#- image: "images/tools/figma.jpg"

---

This soloution will take logs for active SQL Agent Jobs and move them into a reporting table in your database of choice, when you have lots and lots of SQL Agent logs right clicking and selecting view history can take an age to load, having your logs somewhere else that you can index and manage will allow you to view the history of jobs directly from SSMS but also see what happened to a SQL Agent Job from a point in time in the past. 

### Downloading

The script can be downloaded from the projects GitHub, the link of which can be found in the sidebar of this post.

### Requirements


- Ability to create stored procedures
- Ability to create tables
- Ability to drop stored procedures
- A database with the name DBA_Tasks
- Schema in the above mentioned database called DBA
- SQL Server 2008+

When running this script if the an object exists in the DBA_Tasks database under the schema DBA with the name p_Cleanup_Frequent_Job_History it will be dropped and this stored procedure created in it's place.

### Installation 

1. Run the Frequent_agent_check.sql script on your SQL enviroment, if you are running an availability group make sure the script is installed on all nodes.

### Configuration

There is no configuration for this soloution, but you will need to configure the SQL Agent Job that runs it, the Frequent_agent_check.sql script will create a Agent Job when you install it, this comes without a schedule so you will need to set one that fits in with your business. 

### Further Reading

This stored procedure makes use of purge_jobhistory we only specify the job name, no date range is specified for the high frquency jobs as in our instance we didn't want to keep any of this data in MSDB at all.