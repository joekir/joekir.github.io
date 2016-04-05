---
layout: post
title: Get access to edit sealed, read only columns in SP2010
date: 2011-11-24 18:51:00.000000000 -08:00
type: post
published: true
status: publish
categories:
- Powershell
- SharePoint 2010
tags: []
---
<p>When using SharePoint content migration software this is really useful because if the column is read-only and sealed you can't write to it! (Unless you do a few tricks... )</p>
<p><u><strong>Using SharePoint Management Shell (2010):</strong></u></p>

{% highlight powershell %}
#Get the web, list and column objects<br />
$web = Get-SPWeb
$list = $web.Lists[""]
$column = $list.Fields[""]
#Change the readonly and sealed properties and update objects
$column.ReadOnlyField = $false
$column.Sealed = $false
$column.Update()
$list.Update()
#Give us the new column settings for our reference
$column
$web.Dispose()
{% endhighlight %}

<p>Click <a href="http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spfield_properties.aspx" target="_blank">here</a> for more SPField properties.</p>
