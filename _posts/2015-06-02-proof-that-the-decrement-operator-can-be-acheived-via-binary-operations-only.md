---
layout: post
title: Proof that the decrement operator can be achieved [exclusively with binary
  operations]
date: 2015-06-02 15:56:47.000000000 -07:00
type: post
published: true
status: publish
categories:
- assembler
tags:
- ALU
- assembler
- binary
excerpt: proof of binary formula for decrementing a number using only neg and not.
---
A friend of mine was recently telling me how he'd done an electronics assignment where for extra credit they had to create an in-place decrement operator (by that I mean not taking the approach of taking the [2s complement](http://en.wikipedia.org/wiki/Two%27s_complement) of -1 and adding it to a number). This sounded cool and seemed like the way the ALU would do it to save allocation overhead and space.

Here's the identity he told me about:

$$x - 1 == \texttt{~}(-x)$$

where:     
"~" represents the binary complement    
"-" represents the binary negative, aka 2s complement of the number.   
 
Looks really bizarre but feels like it should make sense. I rooted about on Google search to find the proof, but either couldn't find it, or more likely didn't have my search terms fine tuned! (See [http://www.woodmann.com/searchlores/](http://www.woodmann.com/searchlores/))

Here's the forward proof I came to. (using '=' to mean comparison not assignment here)

$$x = x$$

$$x = (x+1) -1$$

due to 2s complement we know that \\(\texttt{~}x +1 = -x\\), therefore \\(x+1 = -(\texttt{~}x)\\), substituting this gives

$$x = -(\texttt{~} x) - 1$$

as we know the negation is 2s complement of the brackets, therefore

$$x = \texttt{~}(\texttt{~}x +1) +1$$

now sub back our other formula from above

$$x = \texttt{~}(-x) +1$$

and here we are

$$x -1 = \texttt{~}(-x)$$

_If you notice any errors above, or if you just need to "Throw some knowledgeballs down the webhills!", please flame me on the appropriate public channels._
