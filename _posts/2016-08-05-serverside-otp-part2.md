---
layout: post
title: Serverside One-Time-Pad (Part2)
type: post
published: true
status: publish
categories:
- authentication
- identity
---

After the [last post](/2016/08/02/serverside-otp-part1/) I wrote, a friend of mine ([@tobyjsullivan](https://twitter.com/tobyjsullivan)) astutely pointed out that there was still a weakness in that the facade site could read the server-side-otp from the legit site and present it to the client on their fraud site.

My strategy to protect against this was to think of some way that the code would only be contextually valid while on the legitimate site.
One idea was what if the reverse traceroute information was stored in the HOTP, then the client could invert their traceroute to the server, and verify that it was served from the place it was saying. However what I wasn't fully aware of until I read [this paper](http://www-bcf.usc.edu/~katzbass//papers/reverse_traceroute-nsdi10.pdf) and watched [this talk](https://www.usenix.org/conference/nsdi10-0/reverse-traceroute) was most often reverse traceroute is not symmetric with the forward traceroute. So while more difficult for an attacker to forge, it would not be a viable implementation option.

Instead I went for more simple, but not quite as bulletproof approach. The server can embed its IP address in the HOTP (Hmac-based One Time Pad)

Server generates:     
$$HOTP = truncate(hmac(key,ctr|serverIP))$$

Client generates:     
$$HOTP = truncate(hmac(key,ctr|phishingSiteIP))$$

The client compares the two and if they do not match, the client would have reason to believe they are not on the legitimate site and would not put in their client HOTP code. They could then safely follow up by reseting their password, while the 2nd factor protects their account from  attacker compromise.

*I'm working on some sample code to demonstrate this flow, I'll update this post with a link at a later date.*
