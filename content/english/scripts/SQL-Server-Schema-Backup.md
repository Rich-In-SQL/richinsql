---
title: "SQL Server Schema Backup"
date: 2022-09-24T00:00:00Z
draft: false
description: "Export your database schema to .sql files using PowerShell."

weight: 1
categories: ["sqlserver"]

thumbnail: "images/scripts/SQL-Server-Setup.png"
tools_github: "https://github.com/Rich-In-SQL/SQL-Schema-Backup"
tools_documentation: "https://youtu.be/yRzeTQdLaIA"

tools_info:
- title: "Types :"
  content: "T-SQL"
#- title: "Colors :"
#  content: "Blue, Purple, White, Orange"

tools_images:
- image: "images/scripts/SQL-Server-Setup.png"
#- image: "images/tools/figma.jpg"
---

# Introduction

This script uses [dbatools](https://dbatools.io/) to backup database objects from a SQL Server Database to a directory of your choosing. This script is used internally to backup our databases for GitHub.

The server & database are both user defined parameters.

## Accepted Parameters 

| Parameter | Data Type  | Mandatory |
|---|---|---|
| database | string | Yes |
| server | string | Yes |
| SourceControlDirectory | string | Yes |
| logDirectory | string | Yes |

## Getting Started

1. Clone the code to your local workstation.
2. Navigate to the directory where the script has been cloned
3. Execute the script passing in the required parameters.

### Executing the script

`.\SQL-Schema-Backup.ps1 -database 'YourDatabase' -server 'YourServer' -SourceControlDirectory 'C:\Temp\' -logDirectory 'C:\Temp\Logs'`