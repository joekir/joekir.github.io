---
layout: post
title: Z3 Constraint Solver Tips 
comments: true
type: post
published: true
status: publish
---
In my opinion an invaluable tool to have in the toolbox for a security professional is a constraint solver. It's one of those things that isn't used often but when it is it's very powerful.

My solver of choice is [Z3](https://github.com/Z3Prover/z3), I've liked it for a long time, when I first encountered it I didn't even know it was Z3, I used to program in C# and there was a tool I used called [Pex](https://www.microsoft.com/en-us/research/project/pex-and-moles-isolation-and-white-box-unit-testing-for-net/) that was a wrapper around Z3 that would find inputs that violated assertions/constraints you'd made. It was a great tool from Microsoft Research and eventually got rolled into a product in Visual Studio called [Code Digger](https://marketplace.visualstudio.com/items?itemName=RiSEResearchinSoftwareEngineering.MicrosoftCodeDigger).

A few years back I thought it would be great if you could take a language like C, parse that to an AST then generate Z3-python from that, but in this more general use I found it too hard to deal with all the edge cases. So I gave up on that.

More recently though I've found Z3 to be an useful tool when performing security assessments, especially so in a dense piece of code, for example:

* inverting obfuscation
* identifying inputs that would trigger numerical overflows
* finding collisions
* buffer over-read/writes (providing you model the allocator to allocate free-variable z3 Arrays)

It works particularly well in self contained functions that don't allocate heap memory or call APIs, as modelling those can be challenging. 

## Tips

Here's some tips I've learned that I think are useful, most of these stem from compiler theory and I learned them from this [great talk](https://youtu.be/ruNFcH-KibY) from a Haskell conference (I don't write Haskell, the talk isn't heavy on that and the content helped me put a common name the things I'd already been doing) 

1. Inline all functions within the scope you're trying to model 
 
2. Static Single Assignment (SSA)     
  In normal code we're used to seeing things like `a = a + 1` but constraint solver logic is more like maths where there is no assignment order, you can just check equalities. Therefore this would become :
  ```
  s = z3.Solver()
  a0, a1 = z3.BitVec("a0 a1", 32)
  s.add(a1 == a0 + 1)
  ```
  This is known as Static Single Assignment, I came across this by accident when I was modelling functions in Z3-python, and then later learned this is a common compiler term.
3. Loops     
  You can unroll them if possible, but I figured out an interesting approach in Python that allows you to do some meta-programming, so one of the issues in a bounded loop with z3 solver item allocations is you don't know how many you're going to have to allocate, Say we have this silly Java example: 
  ```
  int foo(int a) {
      for(int i = 0; i < 10; i++){
        a += 6;
      } 
  
      a *= 5;
      return a;
  }
  ```
  say we are given the return value is 320 and we want to know the input (obviously this is trivial and you could do it in your head, but this is to illustrate more of the bookkeeping than the solver power)   
  
    One of the issues here for SSA is how can we know how many a<sub>n</sub>'s to allocate for the assignments, and how can we then track what what a<sub>n</sub> we're on when we try to multiply by 5?
   
      Here's something I've found helpful in z3 python:
      ```
      import z3
      
      # just a helper to make it easier
      # to read in the loop below
      def name(i):
        return "a"+str(i)
      
      s = z3.Solver()
      
      x = 0
      aVars = {}
      aVars[name(x)] = z3.BitVec(name(x),32)
      x += 1
      
      for i in range(0, 10):
        aVars[name(x)] = z3.BitVec(name(x), 32)
        s.add(aVars[name(x)] == (aVars[name(x-1)] + 6))
        x += 1
      
      aVars[name(x)] = z3.BitVec(name(x), 32)
      s.add(aVars[name(x)] == (aVars[name(x-1)] * 5))
      s.add(aVars[name(x)] == 320) # the return value we know
      
      check = s.check()
      if check.r == 1:
        print s.model()  
      ```
      This prints : 
      ```
      [a11 = 320,
       a10 = 64,
       a9 = 58,
       a8 = 52,
       a7 = 46,
       a6 = 40,
       a5 = 34,
       a4 = 28,
       a3 = 22,
       a2 = 16,
       a1 = 10,
       a0 = 4]
      ```
  So we can see the original input for a was 4. 
  
I've found this trick of dynamically naming variables and handling the bookkeeping in a Python map to be very effective. You could also use something like `aVars[aVars.keys()[-1]]` if you need the last one but I find it easier to keep a separate allocator count as we rarely need just the last element when we're doing the solver assertions.  

### Covering old ground

I wrote [this post](https://www.josephkirwin.com/2015/10/23/all_is_not_lost/) in 2015 on how to do simultaneous equations at the bit-level to invert shift-xor operations. The example from that post is easy to solve with z3

```
import z3

# solve:
# ( x >> 2 ) ^ x == 8

s = z3.Solver()

x = z3.BitVec('x',32)
s.add(((x >> 2) ^ x) == 8)

check = s.check()
if check.r == 1:
  print s.model() # prints [x = 10]
```
