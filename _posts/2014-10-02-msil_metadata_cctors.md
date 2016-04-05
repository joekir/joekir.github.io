---
layout: post
title: Navigating to static constructors (.cctor) in MSIL
date: 2014-10-02 22:06:34.000000000 -07:00
type: post
published: true
status: publish
categories:
- .Net
tags:
- CLR
- IDA Pro
- MSIL
---
<p>When reverse engineering .Net code, either with an interactive disassembler or by programmatically parsing the MSIL headers. There is some low hanging fruit and some fruit that is out of reach, unless you are a giraffe.</p>
<p>In IDA Pro one of the items that you never see explicitly referenced are the <a href="http://msdn.microsoft.com/en-us/library/k9x6w0hc.aspx" target="_blank">static constructors</a> (.cctor). In malware these are popular as they can be added to an existing class and they are executed once and only once prior to the instance constructor (.ctor) being called. Which means that someone can take the existing class and add a static constructor to it that performs stealth, malicious behaviour without changing the code's flow.</p>
<p>So How do we get to these?</p>
<p>Dropping to the byte level. The .Net metadata header is effectively a relational database. It has tables for each object e.g. types, namespaces, methods and rows in each table. Each row contains various flags or indices into other tables. Hence why I like to think of it as a relational database.</p>
<p>Each user defined class will be present in what's known as the TypeDef table, this is referenced at the MSIL level as</p>
<p>060000 | 02 where the 02 references a specific table, namely TypeDef and in this example we're looking at row 6.</p>
<p>Parsing these tables is another story that is clearly defined in the <a href="http://jilc.sourceforge.net/ecma_p2_cil.shtml" target="_blank">ECMA CIL Partition 2 : metadata and semantics.</a></p>
<p>Once you navigate to a TypeDef row you have the following datastructure</p>
<ul>
<li><em>Flags (bitmask of type <em>TypeAttributes</em>) - DWORD</em></li>
<li><em>Name (index into String heap) - WORD<br />
</em></li>
<li><em>Namespac</em>e (index into #Strings heap) - WORD</li>
<li><em>Extends</em> <em>- WORD</em></li>
<li><em>FieldList (index into field table)</em> - WORD</li>
<li><em>MethodList</em> (index into method table) - WORD</li>
</ul>
<p>So from the name index we can find out what the name of the user defined type is, this is useful if you want to know about a specific class by name. Also we can find out what namespace it belongs to, you'd think that there would be a similar hierarchical reference from a namespace table to the types within it, but I don't know of such a reference.</p>
<p>The best parts about this datastructure are the lists, they point to the head of a contiguous block of methods or fields in there respective tables that belong to the type (class).</p>
<p>You probably realize where this is going, if you need to find the .cctor you could look at each row in the method table belonging to the type to see if it is the .cctor method (note that name is immune to fuzzing/obfuscation as it is reserved by the .Net runtime). Each row in the MethodDef takes following form:</p>
<ul>
<li><em>RVA to method's MSIL - DWORD<br />
</em></li>
<li><em>ImplFlags </em>(bitmask of type <em>MethodImplAttributes</em>) - WORD<a href="http://jilc.sourceforge.net/ecma_p2_cil.shtml#_Flags_for_Methods"><br />
</a></li>
<li><em>Flags </em>(bitmask of type <em>MethodAttribute</em>) - WORD<a href="http://jilc.sourceforge.net/ecma_p2_cil.shtml#_Flags_for_Methods"><br />
</a></li>
<li><em>Name</em> (index into String heap) - WORD</li>
<li><em>Signature </em>(index into Blob heap) - WORD</li>
<li><em>ParamList </em>(index into <em>Param</em> table) - WORD</li>
</ul>
<p>So you can use the name index to check the name the method within the #Strings stream (.Net ususally having 5 streams #~/#-, #Strings,#US,#GUID,#Blob). Then once you've confirmed it's the .cctor, you'll have access to the method's MSIL via the RVA.</p>
<p>One snag with the design of these List head indices is that the stop condition is defined by either the end of the table or the head index of the next contiguous run of method, fields or params, which will be pointed to by the next row in the parent table.</p>
<p>Assuming the first method in the list was indeed the static constructor (which sadly it usually is) the goofy, pseudo-SQL would be:
{% highlight SQL %}
SELECT t.name, m.RVA
FROM Types t INNER JOIN Methods m
   ON m.id = t.mlisthead JOIN Strings s
   ON s.id = m.id
WHERE s.name = '.cctor'
{% endhighlight %}
_This goofy illustrative SQL was not well thought out, or even tested_
