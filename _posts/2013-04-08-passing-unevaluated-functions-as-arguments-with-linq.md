---
layout: post
title: Passing Unevaluated Functions as Arguments with Linq
date: 2013-04-08 21:10:00.000000000 -07:00
type: post
published: true
status: publish
categories:
- C#
- Functional
- Linq
tags: []
---
<p>I was pleased with this bit of code today:</p>
{% highlight csharp %}
  List myList = new List{"test","monkey"};
  string[] animals = new string[]{"giraffe","goat","monkey"};
  IEnumerable reduced = myList.Where(item => animals.Any(item.Contains)).ToList();
  
  //Returns "monkey"
{% endhighlight %}

Filtering a list using items from another list would usually require a developer to use a nested foreach. I'm a big fan of linq for its consise syntax, readability and the functional aspects. The <a href="http://msdn.microsoft.com/en-us/library/bb534972%28v=vs.90%29.aspx" target="_blank">System.Linq .Any()</a> extension method here has parameters IEnumarable and Predicate, however the first argument can be waived if you are passing an unevaluated predicate function.</p>
<p>I was doing some programming for the local zoo today, of course!</p>
