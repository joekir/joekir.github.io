---
layout: post
title: Micro-Optimization for C# Loops
date: 2011-11-28 23:30:00.000000000 -08:00
type: post
published: true
status: publish
categories:
- C#
- CPU
- Project Euler
tags: []
---
<p>I was recently trying some of the basic problems from <a href="http://projecteuler.net/" target="_blank">Project Euler</a>, which is highly recommendable for interesting problems to solve that also teach you lessons on how to improve the way you code.</p>
<p>I had to optimise the loop that I was doing a for through a large amount of items with each increment performing a computation. To shave 10ths of a second from my computation I tried micro-optimising by using a decrementing loop.</p>
<p>Here's a generalised version of what I did</p>
<p>Let max be the number that you want to loop until.</p>
<p><strong>Incremental Loop</strong></p>
{% highlight csharp %}
for (int i = 0; i < max; i++) {
}
{% endhighlight %}
<p><strong>Decremental Loop</strong></p>


{% highlight csharp %}
for (int i = max - 1;  i >= 0;  i--) {
}
{% endhighlight %}

<p><u>Things to note</u></p>
<ul>
<li>The decremented version is only faster when the calculation of max-1 is less complex than whatever is in the curly braces.</li>
<li>From a high level view, the reason that the decremental loop is faster is that most CPUs already have a decrement to zero functionality in-built so on an MSIL level it is just duplicating a section of this code. Whereas incrementing to an arbitrary maximum is not something that is fundamental.</li>
<li>This only makes the performance notably faster if there are many items to iterate through, there are nested loops and also it the code within the curly braces is relatively time consuming.</li>
</ul>
<p><u>Good discussions and articles on this</u></p>
<ul>
<li><a href="http://www.dotnetperls.com/decrement-optimization" target="_blank">Dot Net Pearls</a><u></u></li>
<li><a href="http://stackoverflow.com/questions/1656506/which-of-these-pieces-of-code-is-faster-in-java" target="_blank">Stack Overflow discussion on the topic with great responses</a></li>
</ul>
<p>Anyone with better explanations of why decrementing is better, feel free to comment...</p>
