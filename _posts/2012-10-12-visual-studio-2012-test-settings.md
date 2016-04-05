---
layout: post
title: Visual Studio 2012 Test Settings
date: 2012-10-12 18:20:00.000000000 -07:00
type: post
published: true
status: publish
categories:
- Configuration
- Unit Testing
- Visual Studio 2012
tags: []
---
Is it just me or does Visual Studio 2012 seem incomplete? For many reasons I've been seeing lately with the testing aspects this seems to be the case.    

**<u>Test Settings configurations</u>**

- Visual Studio 2008 – it was called a <a href="http://msdn.microsoft.com/en-us/library/ms182480(v=vs.90).aspx" target="_blank" rel="nofollow">.testrunconfig file</a>. You could configure with a GUI and it had xsd templating so you could use IntelliSense when hand coding and create from the new item menu.
- Visual Studio 2010 – it was specified in a <a href="http://msdn.microsoft.com/en-us/library/ee256991(v=vs.100).aspx" target="_blank" rel="nofollow">.testsettings file</a> and it had GUI interaction and xml templating for customization. This can still be used in VS2012 via a ForcedLegacyMode element in the xml, however this is noted as a performance inhibitor for the test runner.
- Visual Studio 2012 – It's now specified in a <a href="http://msdn.microsoft.com/en-us/library/jj635153.aspx" target="_blank" rel="nofollow">.runsettings file</a>, with no GUI config editting, IntelliSense support or item creation templating.

I do hope that Microsoft make a move to resolve this issue in service pack 1 as currently in Visual Studio 2012 you can only create the legacy VS2010 testsettings file using the add new item context menu. This was a real oversight on their part as to create the recommended VS2012 configuration file you have to add an xml file to the project, rename the extension type and add in the sample xml from the msdn page.

**<u>Test Runner Binding Redirects</u>**      
I find this strange about the VS2012 Mstest.exe.config. Look at the runtime binding redirect:

{% highlight xml %}
<probing privatePath="PrivateAssemblies;PublicAssemblies; 
  PrivateAssemblies\DataCollectors;PrivateAssemblies\DataCollectors\x86">
<assemblyIdentity 
  name="Microsoft.VisualStudio.QualityTools.UnitTestFramework" 
  publicKeyToken="b03f5f7f11d50a3a" culture="neutral">
{% endhighlight %}

<p>Totally weird, the redirect is basically saying if you call mstest from VS2012 (11+) then you run the VS2010 mstest? It seems ok to do this for VS2010 Service packs, but for a whole new IDE I think a new version of mstest would be good. Again, perhaps they're going for a phased delivery approach.</p>
