---
layout: post
title: Intermediate Language Ergonomics
date: 2015-10-02 17:31:04.000000000 -07:00
type: post
published: true
status: publish
categories:
- Application Security
- bytecode
- Java
tags:
- bytecode
- Java
- Security
excerpt: Is there a way we can revise high level language code layout such that its
  intuitive to understand how the lower level code behaves?
---
>**Could we have a high level language where the flow of code maps to the underlying intermediate language's code flow?**

Here's an example of some troublesome group of not-obvious integer overflows in Java

``` java
public static void main(String[] args) {
  /* Lets assume I just pass in any integer argument here, e.g. 1
     So we expect it to print 0x0000000080000000 
     (still a positive number)
     because the max positive integer was 0x7fffffff<br />
  */
  int a = Integer.valueOf(args[0]);
  long k = Integer.MAX_VALUE + a;
  System.out.println(Long.toHexString(k)); 
  // we unfortunately get 0xffffffff80000000
}
```

the bytecode ordering can help give us more insight into why this overflows (I've commented below what the crucial bit of the code does for the none-bytecode aficionados)

``` java
aload_0
iconst_0
aaload
invokestatic  #2 // Method java/lang/Integer.valueOf:(Ljava/lang/String;)
invokevirtual #3 // Method java/lang/Integer.intValue:()
istore_1

// That bit above just got the arg from the commandline
// and stored the result to variable number 1
// Its not the code in question


ldc #5   // int 2147483647 - push an integer constant #5 from the constant pool.
iload_1  // load int from local variable 1
iadd     // add the 2 ints on the stack and return an int
i2l      // consume integer from stack, extend and push as a long on the stack 
lstore_2 // store the long as local variable 2

// The bit below is just the printing part
// its also not the part to focus on.

getstatic     #6 // Field java/lang/System.out:Ljava/io/PrintStream;
lload_2
invokestatic  #7 // Method java/lang/Long.toHexString:(J)Ljava/lang/String;
invokevirtual #8 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
return
```

if we then compare this with the solution that makes this code to behave as we expect

``` java
int a = Integer.valueOf(args[0]);
long k = (long)Integer.MAX_VALUE + a;
System.out.println(Long.toHexString(k)); // Now we get 0x0000000080000000
```


``` java
ldc2_w  #5  // long 2147483647l - load the constant from the pool as a wide(long)
iload_1     // load int from local variable 1
i2l         // consume integer from stack, extend and push as a long on the stack
ladd        // add the 2 longs on the stack and return an long<br />
lstore_2
```


OK, so other than being a rubbish Java dev, what is my gripe here?
>**Could we have a high level language where the flow of code maps to the underlying intermediate language's code flow?**

e.g. why can't the Java look like this, to ensure that by default we don't run into this issue?

```
/*
   If we were to lend the := from ALGOL derivatives
   using =: to represent right->left assignment
   and := as the usual Java left->right assignment
*/
Integer.valueOf(args[0]) =: int a;
(long)Integer.MAX_VALUE + a =: long k;
System.out.println(Long.toHexString(k));
```

Seeing the code this way better illustrates that the addition is performed first then it is assigned to the variable.

As an analogy, if a child wants to move the contents of 2 half-size buckets of sand and make a full-sized sand castle in one trip, then they'll need to start with a full sized bucket to carry it! (_I wish that could have been as poetic as [die hard 3's 5ltr and 3ltr jug problem](https://www.youtube.com/watch?v=BVtQNK_ZUJg)_)

I realize that there are issues with associativity and many other things there. However if the high level language could more **intuitively** follow how the lower layer behaves, then we could avoid some unexpected security issues for many junior (and senior) developers.
