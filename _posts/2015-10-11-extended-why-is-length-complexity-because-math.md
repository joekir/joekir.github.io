---
layout: post
title: (extended) Why is length > complexity? Because math...
date: 2015-10-11 15:45:19.000000000 -07:00
type: post
published: true
status: publish
categories:
- passwords
tags:
- passwords
- Security
excerpt: a footnote to the blogpost http://malwarejake.blogspot.ca/2015/10/why-is-length-complexity-because-math.html
---
In response to [http://malwarejake.blogspot.ca/2015/10/why-is-length-complexity-because-math.html](http://malwarejake.blogspot.ca/2015/10/why-is-length-complexity-because-math.html)

Jake - Thanks, I think the blog post was needed, I've been preaching that line for a while!

I realize that your target audience was not necessarily technical, but the stuff in this post is only maths, not code.

So, the bit I have to add is, there's actually a formulaic relationship for how the power should change to match a change in base. I don't know where this formula is documented, so I just wrote it here.

A concrete example:

$$26^4 = 456976$$

$$52^4 = 7311616$$

So what is the relationship between the amount we would need to increase the power for base26 to match that of base52?

$$a^n = (2a)^o$$

where a is the starting base (e.g. base26), n is the new power that we want to figure out, and o is the old power that we set for this new base (2 x base26)    


$$\log_n a^n = \log_n (2a)^o$$

$$n\log_n a = o\log_n 2a$$

$$n = o \frac{\log_n (2a)}{ \log_n a}$$

and if we want that more generally we can just sub the 2 for whatever function we are doing to our original base.

So plugging the values back into our example

$$n = 4 \frac{\log_n 52}{\log_n 26}$$

$$n \approx 4.85$$

What is interesting about this result is that to match the doubling of the base you only have to increase the power by 0.85.

What's more interesting though is in this relationship

$$\frac{\log_n f(a)}{\log_n a}$$

providing that the function f is some linear function (which to be honest it will have to be, as we only have the full unicode space available when it comes to passwords and 0x10ffff is the max), if that is the case, the limit of the ratio is 1.

>**Bringing it back to more practical terms:
We can match the same _potential_ password complexity of a multiple of the available characters by increasing the password length by about 1.**
