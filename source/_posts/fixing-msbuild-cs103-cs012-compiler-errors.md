---
title: Fixing MSBuild CS103, CS012 Compiler Errors
date: 2018-06-01 22:49:00
tags: msbuild
---

Recently I needed to use MSBuild to compile a solution for a build server.

The MSBuild was location at:

`C:\Program Files (x86)\MSBuild\14.0\Bin`

The following errors were occurring despite this working in the past:

`error CS1525: Invalid expression term ‘decimal’
Syntax error, ‘,’ expected
error CS1003: Syntax error, ‘,’ expected [Web.csproj]`

This was due to C#6/C#7 features which aren’t supported by this MSBuild version.

#### The Fix

Using this MSBuild resolved the issue:

`C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\MSBuild\15.0\Bin`