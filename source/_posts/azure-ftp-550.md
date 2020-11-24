---
title: Azure FTP 550 - Access is Denied
date: 2020/11/10 20:24:00
tags: azure, web apps
---

So, I encountered this for the first time today, the context being a colleague wanted to quickly FTP some files up to an Azure Web app. However no matter what directory they tried, they always got the same error:

![](/post/azure-ftp-550/azure-550-ftp-error.PNG)

Initially I thought they must be a permission missing from the user account, however they had got the credentials from a .PublishSettings for the web app in question

### Solution

Within the XML of the publish settings file, there's a profile entry for a **[insert app name] - Readonly - FTP**. This contains credentials for **read only access**, an easy mistake since depending on your editor, the XML file might be 1 long line. The correct FTP publish section will just have the word **FTP**.

Hope that saves someone some time!



