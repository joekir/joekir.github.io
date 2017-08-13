---
layout: post
title: PoC ~ hiding code in plain html and JavaScript
comments: true
type: post
published: true
status: publish
---
I played around with the language *"WhiteSpace"* a year or so ago and thought of an idea then, took me while to revisit it.

For those that have never heard of it, [WhiteSpace](https://en.wikipedia.org/wiki/Whitespace_(programming_language)){:target="blank"} is a toy language that uses only `[spaces]` `[Tabs]` and `[LineFeeds]` in different combinations to provide rudimentary computing functionality. It's stack based and in my experience has about a 50:1 scaling ratio e.g. 1 println is 50 lines of whitespace! Unfortunately the University of Durham website for it is no longer working, but it [was up](https://web.archive.org/web/20170318183338/http://compsoc.dur.ac.uk/whitespace/tutorial.html){:target="blank"} earlier in the year, thanks for saving that archive.org!

### The idea

- JavaScript doesn't care about whitespace (not the WhiteSpace language, I mean actually it doesn't need any whitespace to correctly function, as can be seen by minifiers)
- WhiteSpace doesn't care about anything but whitespace (obviously)

So what if they were overlayed together? Could the JavaScript pull down a JavaScript based WhiteSpace interpretter and interpret itself?

Let's see! - [https://josephkirwin.com/polyglot1.html](/polyglot1.html) 

### TLDR;

Be careful when analysing potentially-malicious scripts, there could be more there than you think 😉
