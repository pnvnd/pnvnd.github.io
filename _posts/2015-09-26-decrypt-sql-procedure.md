---
layout: post
title: "Decrypt SQL Server Stored Procedures"
subtitle: "Tested on SQL Server 2008, 2012, and 2014"
background: '/img/sql-server-settings/hdd.jpg'
---

# Script
Many stored procedures found in SQL Server databases are obfuscated.  Use the following script to get back some code to re-create stored procedures.  Especially with SQL Server on cloud service providers, obfuscated objects (including functions) may fail during the migration to the cloud.
<script src="https://gist.github.com/pnvnd/ee6f4364bfcf92c93d9b6d04ad8417c7.js"></script>