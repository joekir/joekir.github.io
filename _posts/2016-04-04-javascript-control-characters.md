---
layout: post
title: How JavaScript control characters impact the underlying Buffer 
type: post
published: true
status: publish
categories:
- JavaScript
---
Something odd that I found today, that might disuade developers using Node.js from concatenating user string input.

``` javascript
var userInput = "\b\b\b\bwhat"; // \b is the backspace control char
var link = "http" + userInput;

console.log(link);         // "what"
console.log(link.length);  // 12
console.log(Buffer(link)); // <Buffer 68 74 74 70 08 08 08 08 77 68 61 74>    
```

The [Node.js Buffer API](https://nodejs.org/api/buffer.html) appears to still be lacking in functions to interrogate the underlying buffer. What I would like to see, is a function that can identify the _"window"_ that console.log function (for example) sees within the Buffer. Or ideally a trim to allow you to decrease the underlying buffer size to the same as the visible ascii string, as currently I can't see a workaround for that when you have a variable length string concatenation.

_From an attackers perspective to utilize the backspace type logic attack they would also need to ensure that the length the victim string ('http' in this example) and input string ('what' in this example) are the same, as the "Buffer window" appears to be dependant on the victim string's original length._ 

I guess the [Node.js C++ source code](https://github.com/nodejs/node/blob/master/src/node_buffer.cc) for the Buffer API would be the next point of call to find out why?
