---
layout: post
title: Sharepoint Migration Tool (SMPT) Woes
categories: Windows
comments: true
tags:
  - Sharepoint 2013
  - Sharepoint Migration Tool
  - SPMT
  - Office 365
  - Sharepoint
---

The time has come to migrate our Sharepoint 2013 Server to Microsoft 365. It's been a few days of fighting with the tool provided by Microsoft. It isn't a bad tool, but it isn't forthcoming with error messages when you run into issues.

## Error Messages

Here are some of the error messages I've recieved while working with this tool:

Please re-enter your user ID or password

![Login Dialog](\assets\2021-8-31\SPMT-UserID.PNG)


2021-08-31 09:04:43.8415|ERROR|GLB|8|Csom call with TraceCorrelationId No TraceCorrelationId from OnPrem Microsoft.SharePoint.MigrationTool.MigrationLib.SharePoint.MigSPConfigException: Web Issue when doing SP Query ---> Microsoft.SharePoint.Migration.Common.Exceptions.WebInternalErrorException ---> System.Net.WebException: The remote server returned an error: (500) Internal Server Error.

## Quick fix information

I ran a wireshark capture and noticed what URL the tool was attempting to contact

![Wireshark Capture](\assets\2021-8-31\SPMT-Wireshark-Headers.PNG)

As you can see it was attempting to contact /acct/_vti_bin/client.svc/ProcessQuery

Note: You can ignore the /acct/ in that path, it was specific to my site.

You can see the response in blue, System.ServiceModel.ServiceActivationException

This led me to a StackOverflow post that saved me finally:

<a href="https://stackoverflow.com/questions/56846573/sharepoint-2013-why-getting-500-error-while-creating-clientcontext">SharePoint 2013: Why getting 500 error while creating clientContext?</a>

It turns out we have a couple of services that need to be turned on in order for the CSOM to connect to the backend. Each of these were turned off on the production machine for some reason.

![IIS Admin Service](\assets\2021-8-31\SPMT-IIS-Admin-Service.PNG)

![Services On Server](\assets\2021-8-31\SPMT-Services-On-Server.PNG)

Once I turned these on I was ready to go! The migration went through successfully and my data got sent up to Sharepoint Online.


I know this is a very edge case, but hopefully if you're reading this, it'll help you out!

Got any questions? Drop me a line on twitter!
