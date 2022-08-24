---
layout: post
title: SharePoint - How to display stack trace error in the browser.
type: post
published: true
status: publish
categories:
- SharePoint
tags: []
---
Here's an oldie, but a goodie!

SharePoint provides "friendly error messages", that are comparable to an inane conversation with a stranger about the weather at the bus stop. For example: "An unexpected error has occurred" (The plot thickens...)

Sometimes you just want to know why you can't browse to a SharePoint page, without having to root through ULS/IIS logs on the server.

To disable these custom error messages:

1. On the server find the web application config in question. Usually under `C:\inetpub\wwwroot\wss\VirtualDirectories\web.config`
2. Backup the web.config
3. Open the xml with your favourite text editor (e.g. notepad, notepad++ etc.)
4. Search for "CallStack" it should be under the node named `<SafeMode>` 
5. Change the callstack attribute's value from `false` to `true`
6. Search for `customErrors` and change the attribute's value from `On` to `Off`
7. Save the web.config

Now when you navigate to the contentious page you should get a full stack trace and details!
