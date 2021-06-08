---
layout: post
title: "SQL Server Settings"
subtitle: "Configuration Script for Deployments"
background: '/img/sql-server-settings/hdd.jpg'
---

# Template Script
Configuring SQL Server repeatedly can take up a lot of time.  Here's a script I use to configure SQL Server instances after installing.
<script src="https://gist.github.com/pnvnd/690146b33c353daafc51042aac3520bb.js"></script>

## Physical Memory
For my purposes, keeping RAM maxed out from the default installation slows down the server when things get busy, so here are my guidelines:
- For servers with 20GB RAM or more: 8GB for Operating System, and the rest for SQL Server
- For servers between 16GB to 20GB: 4GB for Operating System, the rest of SQL Server
- Otherwise, for all other configurations, split memory between the operating system and SQL Server evenly.

## Cost Threshold for Parallelism
Set this value to 50 if unsure.  For .NET C# applications I've worked with 43 worked best.  Test by starting at 50 and go up/down by 3 to see what values gives the best performance.

## Max Degree Parallelism
Set this number to be same as the number of logical processors on the system, but cap this at 8.