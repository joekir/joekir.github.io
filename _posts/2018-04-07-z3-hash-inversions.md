---
layout: post
title: Using Z3 to invert non-cryptographic hashes 
comments: true
type: post
published: true
status: publish
---

# Intro

A few months back I was reading some Java code using [MurmurHash3](https://en.wikipedia.org/wiki/MurmurHash#MurmurHash3). This algorithm is quite commonly used for quick hashing, examples where it's used:
- [Optimizely](https://github.com/optimizely/java-sdk/blob/master/core-api/src/main/java/com/optimizely/ab/bucketing/internal/MurmurHash3.java)
- [Guava](https://google.github.io/guava/releases/19.0/api/docs/com/google/common/hash/Hashing.html)
- [Apache Hive](https://github.com/apache/hive/blob/master/storage-api/src/java/org/apache/hive/common/util/Murmur3.java)

Some other examples in the same category are [cityhash](https://github.com/google/cityhash), [FNV hash](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function) etc

# Related Tangent

At 28c3 in 2007 Alexander Klink and Julian Wälde did a talk that I really liked called ["Efficient Denial of Service Attacks on Web Application Platforms"](https://events.ccc.de/congress/2011/Fahrplan/attachments/2007_28C3_Effective_DoS_on_web_application_platforms.pdf) ([video](https://www.youtube.com/watch?v=R2Cq3CLI6H8)) 

This talk was deeper than the content I'm about to discuss as this is about the issues with non-cryptographic hash functions not being fully mixed (and in many cases invertible) such that you can "flood" a hash bucket and the lookup which would normally be an _O(1)_ operation decends to that of a linked list - _O(n)_. 

After this talk there was a fever in open source land to fix these hashing issues in Ruby, PHP, Java etc. Allegedly the only language that knew of this already was Perl, and they'd kept the details to themselves since 2002 😂

The one that still puzzles me is Microsoft's .NET framework (the lanuage I'm most familiar with the internals of), they use something that was patented by [Niels Fergurson](https://patents.google.com/patent/US2013026242) (who I greatly respect), it's distribution allegedly doesn't suffer the issues in the above talk, yet the hash appears to be invertible, so how can this be so? 
It's name is `Marvin` ([github link](https://github.com/dotnet/coreclr/blob/32f0f9721afb584b4a14d69135bea7ddc129f755/src/vm/marvin32.cpp#L219))

# Back on Track

**Would you want to use MurmurHash3 for cryptographic hashing?!**

Obviously this is a rhetorical question, why would anyone use an invertible hash function for cryptographic purposes 🤔

Most of the Java implementations I listed in the Intro reference from [Yonik's implementation](https://github.com/yonik/java_util/blob/master/src/util/hash/MurmurHash3.java) **‡**. So this is the one I have analyzed with the [Z3 theorem prover](https://github.com/Z3Prover/z3).

So this is the example I analyzed with Z3.
At this point it could be useful to read my previous post [Z3 Constraint Solver Tips](https://www.josephkirwin.com/2017/11/16/constraint-solver-tips/) as it's useful for things like dynamic [SSA](https://en.wikipedia.org/wiki/Static_single_assignment_form) in the following. 

{% gist 17e861daf29d2eec4cbbc69f0ba673e1 %}

  **‡** _which he notes is in "public domain" to allow people to use or license as they see fit", he's clearly not to blame if someone uses it for cryptographic purposes, he never stated that anywhere_


# Final Thoughts and Future Work

As the size of the input array increases the collisions also do, so you can add hints based on known structure of the plaintext, or you can iteratively exclude inputs that collide to the same hash until you get a meaningful input. I'm not sure on how coupled the values are that are chosen, or whether the relationship of hints to length is linear or otherwise. But I'd like to look into that.

I'd also like to tidy up some of the dynamic name assignment, potentially as a helper python library so that it can be used to tackle other solver problems.

# Related Reading

- [SipHash: a fast short-input PRF](https://www.131002.net/siphash/) - this expands on the 28c3 work and also provides a solution to this problem which is to use a cryptographic hash function that is faster than any of the incumbent, non-cryptographic ones. 

- [Breaking Murmur: Hash-flooding DoS Reloaded](http://emboss.github.io/blog/2012/12/14/breaking-murmur-hash-flooding-dos-reloaded/) - Martin Boßlet's extensive coverage on the topic of hash-flooding. 

