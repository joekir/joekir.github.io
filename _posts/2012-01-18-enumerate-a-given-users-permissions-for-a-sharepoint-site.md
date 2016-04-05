---
layout: post
title: Enumerate a given user's permissions for a SharePoint site
date: 2012-01-18 20:01:00.000000000 -08:00
type: post
published: true
status: publish
categories:
- MOSS
- Powershell
- SharePoint
- SharePoint 2010
tags: []
---
<p>The <a href="http://technet.microsoft.com/en-us/library/cc721640.aspx">Limited Access</a> permission level in SharePoint is a topic of confusion and annoyance, it can pretty much range from being null/empty to having full control of a specific library and list, with no way to be certain of this from the site level permissions.</p>
<p>This week I had to illustrate to a client what permissions a given user/AD group had for a given site, in SP2010 there is a way to check this in the UI, however in MOSS there isn't an easy way to do it.</p>
<p>So I used PowerShell and its assembly reflection capabilities to enumerate through the lists, libraries and subsites beneath the site and list the permissions levels that the user had.</p>
<p>Its worth noting that I first wrote this in C# and then manually wrote the PowerShell script based on the C# class. If I had known about <a href="http://josephkirwin.com/2012/01/15/converting-c-into-powershell/" target="_blank">this feature of Reflector v7</a> at that point I could've improved my productivity using reflection. Here's the code I used:</p>

{% highlight powershell %}
# Reflects SharePoint dll so we can run against MOSS or SP2010, 
# note must change the version to 14 for SP2010 or 12 for MOSS!
[System.Reflection.Assembly]::Load(“Microsoft.SharePoint, 
  Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c”)
# Site Collection URL
$sitecol = New-Object Microsoft.SharePoint
        .SPSite("http://yourwebapp/sites/yoursitecollection")
# Subsite url name
$sitename = "yoursitename"
$web = $sitecol.Allwebs[$sitename]
# Username you are looking for
$username = "DOMAIN\user"
# Go through the site and if you find the user has permissions on a list 
# read them out to me
function GetAccessForGivenUser ($u,$w) {
    write-host $w.title
    foreach ($list in $w.lists) {
        write-host $list.Title
        foreach ($SPRS in $list.roleassignments) {
            if($SPRS.Member.loginname -eq $u) {
                Write-Host $SPRS.Member.loginname
                foreach ($RoleDef in $SPRS.RoleDefinitionBindings) {
                    Write-Host $RoleDef.Name
                }
            }
        }
    }
}
GetAccessForGivenUser $username $web
# Then for subsites
foreach ($subweb in $web.webs) {
    try {
        GetAccessForGivenUser $username $subweb
    } 
    finally {
        $subweb.dispose()
    }
}
{% endhighlight %}

<p>The C# equivalent of this script was adapted from code written by <a href="http://stackoverflow.com/questions/2248211/getting-all-sites-lists-and-user-permissions-in-sharepoint">Zincorp on StackOverflow</a></p>
