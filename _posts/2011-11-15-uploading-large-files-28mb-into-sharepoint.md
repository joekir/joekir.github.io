---
layout: post
title: Uploading large files (>28MB) into SharePoint
type: post
published: true
status: publish
categories:
- IIS
- IIS7
- SharePoint
tags: []
---
SharePoint lists have a default file size upload limit and it's set at 50MB, although IIS7 applies a further capping of 28MB, due to the maxAllowedContentValue in the web.config. If the IIS7 threshold is lower than the SharePoint limit then every time you try to upload a document that is too large you will see a Http 404 error and not a SharePoint error.
The issue here is not just the web application settings in central admin. Execution timeouts and connection timeouts must be changed.

To change the maximum file size that you can upload to SharePoint:

1. SharePoint max upload size - Central admin → Application Management → General Web Application Settings and set the max upload to whatever size you require.
2. IIS connection timeout - Change configuration in IIS from the default 2 mins.
3. 12/14 Hive webapp execution timeout settings - In the layouts folder in the 12 or 14 hive there is a web.config, add an execution timeout attribute to the httpRuntime node.
4. Inetpub execution timeout settings - Open the web.config file for the site from your inetpub virtual directory and add an execution timeout attribute to the httpRuntime node. (Note this input is in KiloBytes, the largest value you can put for the maxrequestlength value here is 2GB (2097151KB), see here for reference)
5. IIS7 onwards exclusively - Add a requestLimits node to the configuration section of the wss virtual directory web.config, this should contain an attribute named "maxAllowedContentLength" with a value of whatever the SP max upload size that you require, e.g. 52428800 (50MB) 
6. IISRESET

This is extensively detailed in [this Microsoft Knowledge Base article](https://docs.microsoft.com/en-US/sharepoint/troubleshoot/lists-and-libraries/request-timed-out-when-upload-large-file-to-library).

An alternative method is to follow good business practice and not create MS Office documents that are humongous as they are pretty unstable!!
