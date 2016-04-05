---
layout: post
title: The Sieve of Eratosthenes
date: 2011-12-22 18:37:00.000000000 -08:00
type: post
published: true
status: publish
categories:
- C#
- Project Euler
tags: []
---
<p><span style="font-family:inherit;">I had some more fun with </span><a href="http://projecteuler.net/" target="_blank"><span style="font-family:inherit;">Project Euler</span></a><span style="font-family:inherit;"> problems, the important thing I found about those problems is they have a classical mathematical theory/method behind them, but they always have a twist that you must optimise the code or push your boundaries of coding comfort to get a result out.</span></p>
<p><span style="font-family:inherit;">The question that I was working on at my lunch break yesterday was:</span></p>
<div>
<blockquote class="tr_bq"><p><span style="font-family:inherit;">"The prime factors of 13195 are 5, 7, 13 and 29.</span></p></blockquote>
</div>
<div>
<blockquote class="tr_bq"><p><span style="font-family:inherit;"> What is the largest prime factor of the number 600851475143 ? "</span></p></blockquote>
</div>
<p><span style="font-family:inherit;">So I broke it down into three steps</span></p>
<ul>
<li><span style="font-family:inherit;">Choose an arbitrary number</span></li>
<li><span style="font-family:inherit;">Find all the primes less than it</span></li>
<li><span style="font-family:inherit;">Check to see which of those divide into it without a remainder</span></li>
</ul>
<p>To find all the primes numbers in a set of numbers iteratively there is an algorithm from classical mathematics called <a href="http://en.wikipedia.org/wiki/Sieve_of_Eratosthenes" target="_blank">the Sieve of Eratosthenes</a> (Sounds like something from Jason and the Argonauts, right?). This iteratively moves through the set from the first prime "2" and removes each primes multiples from the set. Then once you reach the arbitrarily large number only the primes remain. However I had trouble doing this in c# as in a foreach loop you can't remove items from the list while its iterating through it, if you do it gets a big fat error.</p>
<p>There probably is a way to do it with a different type of loop I just couldn't figure it out.</p>
<p>So instead I went for something, that probably doesn't perform so great:</p>
<ul>
<li>Created a static boolean test (IsPrimeNumber) to see if a number is a prime</li>
<li>Then for every number from 2 ⇒ the large number, I used the IsPrimeNumber test and if it passed that, then it went into another if loop to check if it was a factor of the large number.</li>
</ul>
<p><strong>Here's the code for the console app</strong></p>

{% highlight csharp %}
 class Program {
        //Give a true or false if a number is a prime
        static bool IsPrimeNumber(Int64 num) {
            bool boolprime = true;
            Int64 factor = num / 2;
            Int64 i = 0;
            for (i = 2; i &lt;= factor; i++) {
                if ((num % i) == 0) {
                    boolprime = false;
                }
            }
            return boolprime;
        }
        
        static void Main(string[] args) {
            //Set the number that you want to find the prime factors
            const Int64 max = 600851475143;
            for (Int64 p = 2; p &lt; max; p++) {
                if (IsPrimeNumber(p) == true) {
                    if ((max % p) == 0) {
                        Console.WriteLine(p);
                    }
                }
            }
        }
    }
{% endhighlight %}
