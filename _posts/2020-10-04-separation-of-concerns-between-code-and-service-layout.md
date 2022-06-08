---
layout: post
title: Separation of concerns between code and service layout 
type: post
published: true
status: publish
toc: true
---

No doubt, when I write this (assuming someone reads it) someone might say:

> "Oh, you should have used Garibaldi.... or Ozymandias or Plato.js"

...or some other framework. Please let me know in the comments, I couldn't find anything on this and must have been searching for the wrong keywords.
When that happens I'll add an 'edit' here to route anyone looking for that that way.

# Problem Statement

Initially you may start out building a piece of software, and that may grow to be a huge monolithic codebase. There's benefits to everything being together initially, but then it's frustrating later on if things are glued together and you're unable to split it apart easily. 

An additional challenge is that SREs/ops/devops people don't have enough fine-grained control to split up the functionality efficiently.

So are these 2 things achievable:

1. Some design that allows you to easily break apart (or pull together) components of a system
2. This design allows a separation-of-concern from the developers and lets operations people chose the layout

# Inspiration

I learned in this previous project [github.com/joekir/indelible](https://github.com/joekir/indelible/blob/master/cmd/indelible/main.go#L89) that [GRPC](https://grpc.io/) could easily switch it's "transport medium" without needing to change the functionality of the code.

This made me think, what if you could abstract the transport mechanism for how a function is called in code. 
Specifically splitting it into:
1. Normal ptr to address in your current process space and exec
![inprocess](/assets/inprocess.png)
2. Interprocess Communication (Unix socket for example)
![ipc](/assets/ipc.png)
3. Network Communication (TCP/UDP)
![network_call](/assets/network_call.png)


# Missing Pieces

aka _"What currently doesn't exist and would need to for this thing to exist"_

Firstly, you'd need some way to [decorate](https://en.wikipedia.org/wiki/Decorator_pattern) a function/module/class/etc in the coding language to indicate that implicitly a protobuf should be generated for it.

e.g. (in pseudo code)

~~~ ruby
  @ComponentBoundary
  func PrintSomething(String title, Integer count=5){
    ...
~~~

which would produce

~~~ proto
...
service <TODO> {
  rpc PrintSomething (PrintSomethingArgs) returns (google.protobuf.Empty) {}
}

message PrintSomethingArgs {
  required string title = 1;
  optional uint32 count = 2;
}
...
~~~


Once you have that, you'd want every function call that goes between one of those decorated functions to have some interstitial code that makes a GRPC call which can either do
- Passthru - where it simply bypasses all the GRPC and allows the normal function call in-process    
          _~OR~_
- Uses some configuration to determine whether it should make the GRPC call over IPC (Interprocess Communication) to another process/container on the same host, or over TCP/UDP to a process over the network.
    - Additionally handling authentication and all other nicities that GRPC would usually handle

Then you'd need some runtime configuration that the operations team could leverage to indicate how that function should be "currently" accessed. 
There'd also need to be some linter/compiler additions to make the language aware of when you're trying to access something that is over a component boundary and expecting to have some shared state. 

# Benefits

1. Performance/Reliability/Cost engineering can focus on the production layout of a system and have more granular controls to make it efficient
2. Developers no longer need to think about GRPC and whether a call will be local or over the network, if it has a decorator it's assumed "it could be" 

# Taking it further

I'm a security engineer, so I like security stuff sometimes.
There's another "constraint" that could be interesting here with the decorator. 

Generally Network > Container > Shared memory, in terms of defending against exploitation. So it could be that you have some security critical code that you only want to be available at IPC or network level.
This decorator could then include that you do not want to allow passthru behaviour, e.g.

~~~ ruby
  @ComponentBoundary(![passthru])
  func SecCriticalCode(Object input, Integer count){
    ...
~~~
<br>
That's it, I haven't coded this thing yet because 1 it seems hard, 2 someone else might have already done it, 3 it could be useless and I just haven't learned why yet. Let me know in the comments! :) 


