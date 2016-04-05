---
layout: post
title: SharePoint 2010 Web Application Health Score
date: 2012-02-20 19:43:00.000000000 -08:00
type: post
published: true
status: publish
categories:
- Powershell
- SharePoint 2010
tags: []
---
<p>As default in SharePoint 2010, each Web Application emits an <a href="http://msdn.microsoft.com/en-us/library/ee557206.aspx">SharePoint Health Score</a> that ranges from 0 to 10 and is an aggregation of many performance stats from the SharePoint farm, including Asp.Net request queue length and available RAM on the Web front end, amongst other factors.</p>
<p>You can obtain the score from a Http response header using F12 dev tools or Fiddler relatively easily, however there doesn't appear to be a property in the SPWebApplication object in the SharePoint API to return the score, so to programmatically obtain the information you have to sniff headers!</p>
<p>I actually found this PowerShell script on a site of a SharePoint blogger named "Cuban's Bullspit", the site now appears to be taken down, which is unfortunate. The script did not work for me at first as some authentication tweaks were required. In its state below it can now be used to return the X-SharePoint health score from a Http response header to determine the SP Health Score of a given Web Application.</p>

{% highlight powershell %}
function Get-SPHealthScore([string]$url){
    $sys = new-object system.Uri($url)
    $request = [System.Net.WebRequest]::Create($sys)
    $request.UseDefaultCredentials = $true
    $request.Method = “HEAD”
    $response = $request.GetResponse()
    $score = $response.Headers.Get(“X-SharePointHealthScore”)
    if($score -ne $null) {
       Write-Host "The SharePoint Health Score for" $url ":" $score
    }
    else {
       Write-Host "The specified URL is not a SharePoint 2010 site or does not contain a health score"
    }
}
{% endhighlight %}

<p>Get-SPHealthScore http:<span class="rem">//servername:1234/sites/site</span></p>
