---
layout: post
title: Operator Algebra
type: post
published: true
status: publish
categories:
- Math
- Solvers
---
I read a book that stated a classic brain teaser recently:

<details> 
  <summary>  
Given the numbers 1,3,4 and 6 total 24 using any operators (including parentheses), you must use each number only once.<i>(hint: expand for answer)</i>
  </summary>
   Solution: $$\frac{6}{(1-3/4)}=24$$
</details><br/>

It's not too easy to express this as a formula, but I like 
$$1{\_}3{\_}4{\_}6 = 24$$ where the underscores represent a constrained variable which in the basic case can be any of $$[{/},{*},{+},{-}]$$, and the numbers can be switched based on whether it would change the outcome.

I wrote a solver ([https://github.com/joekir/operator-algebra/blob/master/solver.py](https://github.com/joekir/operator-algebra/blob/master/solver.py)) for these types of equations that's quite rough around the edges. But my goal wasn't to efficiently solve these, someone better could/can do that.

I wrote the solver to help me find answers to: 

1. How complex is the permutation space?
2. Does the complexity linearly increase as you add variables?
3. Are there any cases where there are many solutions?
4. Is this just something puzzling, or does it have a use. Or even better does it have a use for information security?

I'm still working on the answers to these questions, except for 3 that we can immediately answer with yes, if we switch 7 with 6 in the question above, and 24 for 28. then we have 

>   Solution 1: $$(1 + 7) * 3 + 4=28$$

>   Solution 2: $$\frac{7}{( 1 - 3 / 4 )}=28$$

Some interesting properties to point out about this stuff:

- It's undecidedly [commutative](https://en.wikipedia.org/wiki/Commutative_property)

> e.g. $$7{\_}1 = 8$$

When the underscore is one of $$[{*},{+}]$$ then we can swap these numbers around without issue. Otherwise we can't.

- It's undecidedly [associative](https://en.wikipedia.org/wiki/Associative_property)

> e.g. $$3 {\_} 4 {\_} 2 = 6$$

The Parentheses can only be arbitrarily switched when the [bodmas](https://en.wikipedia.org/wiki/Order_of_operations) Left-to-Right ordering is consistent. (*Hence why I haven't thrown exponents in the operator mix yet!*)

As you can tell this post is just a starting point. If you read this and know of some other work on this stuff that I could read or want to write an `uber-solver` in some functional language, let me know via one of the contact method below!
