---
permalink: /2015/10/23/all_is_not_lost/
layout: post
title: 'All is not lost : right bitwise shifts'
date: 2015-10-23 08:56:02.000000000 -07:00
type: post
published: true
status: publish
tags:
- binary arithmetic
excerpt: Data can be retrieved from right bitwise shift when exclusive OR'd with itself.
---
A friend forwarded me this blog post, it was inspiring:    
[https://krypt05.blogspot.in/2015/10/reversing-shift-xor-operation.html](https://krypt05.blogspot.in/2015/10/reversing-shift-xor-operation.html)
    
It got me thinking about some strange properties that exist in binary arithmetic

<!-- https://gist.github.com/mikelove/cbf6eb431406852ba725 -->

This equation

$$x>>y = z$$

where the shift amount **y** is known and the result **z** is also known. doesn't have a unique solution, as the right-hand shift loses data.

However this equation:

$$(x >> y) \oplus x = z$$

where the shift amount **y** is known and the result **z** is also known.   
Has a unique solution.

**Pseudo-reasoning**

let's change our x to be an array of bits, as that's what it really is.
    
$$(x\_{0..n} >> y) \oplus x\_{0..n} = z$$

what we can reason about this here is that \\(x\_{0..n} &gt;&gt; y\\) when xor'd with \\(x\_{0..n}\\) we're actually solving **n** simple simultaneous equations of the form

$$x_{i-y} \oplus x_{i} = {1|0}$$

but if the x subscript is less than zero, x is just zero. So we actually have **y** degrees of freedom in this simultaneous system, hence it is solvable via a chain substitution.

**A 4 bit unsigned example**

$$(x >> 2) \oplus x = 8$$

ok, so now to write this out in pseudo-binary-goofy-stupid-me-notation:

$$({x_0 x_1 x_2 x_3} >> 2) \oplus {x_0 x_1 x_2 x_3} = 1000$$

Simplifying to

$${0 0 x_0 x_1} \oplus {x_0 x_1 x_2 x_3} = 1000$$

now we can see that the following equations need to be satisfied

$$0 \oplus x_0 = 1 \therefore x_0=1 $$
$$0 \oplus x_1 = 0 \therefore x_1=0 $$
$$x_0 \oplus x_2 = 0 \text{ substituting } x_0 \implies x_2=1$$ 
$$x_1 \oplus x_3 = 0 \text{ substituting } x_1 \implies x_3=0$$

Checking the solution for b1010 aka 10 decimal
$$(10 >> 2) \oplus 10 = 8$$

**Why is that important?**    
From a security perspective, we must be careful to avoid mistakes like xoring a shift with itself, as it converts the desired [surjection](https://en.wikipedia.org/wiki/Surjection) into a [bijection](https://en.wikipedia.org/wiki/Bijection), aka we lose our many-to-one. 
