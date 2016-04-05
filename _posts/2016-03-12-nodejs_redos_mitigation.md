---
layout: post
title: Mitigating Catastrophic Backtracking in Node.js Regular Expressions
date: 2016-03-12 10:14:00 -08:00
type: post
published: true
status: publish
categories:
- JavaScript
excerpt: Snippet of code to isolate Node.js Regular Expression execution with a timeout.
---

Recently I've been learning a bit about Node.js    
I have a guilty pleasure of really enjoying writing Regular Expressions, I think it stems from some anti-spam work I did a while back.    

The issue of <a href="http://www.rexegg.com/regex-explosive-quantifiers.html" target="_blank">catastrophic backtracking</a> is a plague on regex, and can be a pain to test as it can only happen with certain input that may not be covered during testing.

Libraries like [node-rsa](https://github.com/rzcoder/node-rsa/issues/30) on npm have had issues where a catastrophic backtrack can cause a full denial of service (DOS) of the node process. My angle of interest is from the information security side, where a malicious adversary may craft a payload such as this with the intent to actually DOS your server-side JavaScript.

As the RegExp constructor in JavaScript doesn't have a timeout feature (which it should!) I thought of a trick using Node.js's core [vm](https://nodejs.org/api/vm.html) module. (available in v0.12.x, v4.x and v5.x branches)

{% highlight javascript %}
const util = require('util');
const vm = require('vm');
 
var sandbox = {
    result: null
};
 
var context = vm.createContext(sandbox);
console.log('Sandbox initialized: ' + vm.isContext(sandbox));
 
var script = new vm.Script('result = /^(A+)*B/
    .test(\'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAC\');');
 
try{
    // One could argue if a RegExp hasn't processed in a given time.
    // then, its likely it will take exponential time.
    script.runInContext(context, { timeout: '1000' }); // milliseconds
} catch(e){
    console.log('ReDos occurred'); // Take some remedial action here...
}
 
console.log(util.inspect(sandbox)); // Check the results
{% endhighlight %}

Ideally using JavaScript's [protatypal inheritance](http://javascript.crockford.com/prototypal.html) one would overload the RegExp() constructor to include a timeout to tidy away all this code. I didn't go to those lengths here, but perhaps I would in future.
