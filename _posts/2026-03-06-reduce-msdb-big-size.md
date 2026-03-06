---
layout: default
title: "How to reduce MSDB size from 42Gb to 200Mb"
description: "How to troubleshoot an oversized SQL Server msdb database and reduce it from 42GB to 200MB by cleaning Service Broker queues, Database Mail data, and SQL Agent history."
date: 2026-03-06
---

Recently I’ve got a spare minute to see why an old test server was running too slow… I had nothing to do with this, but I was very anxious to find out what was wrong with the server.

First thing, I opened Resource Monitor and looked at the overall load. The sqlserv.exe process took up 100% of CPU and generated a large disk queue exceeding 300… whereas the number greater than 1 is considered problematic.

When analyzing disk activity, I observed continuous IO operations in msdb:
