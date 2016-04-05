---
layout: post
title: Disabling list view throttling temporarily or even permanently
date: 2011-11-25 16:24:00.000000000 -08:00
type: post
published: true
status: publish
categories:
- Powershell
- SharePoint 2010
tags: []
---
<p>A troublesome issue when working with very large (5000 items or more) SharePoint 2010 lists is list view throttling. It was introduced in SP2010 to limit the amount of requests that can be made as that inhibited server performance and for day to day running this is fine. But when you're trying to migrate content from a huge MOSS 2007 list to an SP2010 list with view throttling it is a big issue.</p>
<p>There are a couple of things you can do to work around this issue.</p>
<p><strong><u>Temporarily</u> disable using SharePoint Central Admin:</strong></p>
<p>Manage web applications → Select Web Application → General Settings → Resource Throttling<br />
You then get to this page</p>
<div class="separator" style="clear:both;text-align:center;"><a style="margin-left:1em;margin-right:1em;" href="http://josephkirwin.files.wordpress.com/2011/11/capture.png"><img src="{{ site.baseurl }}/assets/de6d5-capture.png" alt="" width="320" height="221" border="0" /></a></div>
<p>You then have some useful options here including thresholds and Http throttling.<br />
The options that you need to change to temporarily disable List View throttling are:</p>
<ul>
<li> Set the "Daily time window for large queries" to enabled for the duration you require. This will temporarily remove the throttling,</li>
<li>You can also disable "HTTP request monitoring and throttling", this will de-restrict the web front end request limits, from a SharePoint perspective at least.</li>
</ul>
<p><strong><u>Permanently</u> disable throttling using PowerShell</strong></p>
<p>In the SharePoint Management Shell run the following command against your webapp:</p>

``` powershell
$web = Get-SPWeb
$list = $web.Lists[“”]
$list.Enablethrottling = $false
$list.Update()
$web.Dispose()
```

<p>This will permanently disable throttling and cannot be changed using the UI.</p>
