---
layout: post
title: Updates to the design of Veerless 
comments: true
type: post
published: true
status: publish
categories:
- authentication
- identification
---

In some previous posts [[1](https://www.josephkirwin.com/2016/08/02/serverside-otp-part1/)],[[2](https://www.josephkirwin.com/2016/08/05/serverside-otp-part2/)],[[3](https://www.josephkirwin.com/2016/09/12/server-authentication-with-lamports-scheme/)] I discussed how bi-directional 2FA might happen and what the value of it is with regards to phishing, specifically the credential theft variety.

Since then I've worked on a reference implementation of this scheme called [Veerless](https://veerless.josephkirwin.com) (code is at [https://github.com/joekir/veerless](https://github.com/joekir/veerless))

### 1st iteration

So that I could demo what the mechanism was I thought the flow could be:     

- User inputs username     
- Server provides server otp     
- A browser plugin checks this:     
  - [Fail] If it's invalid, it gives the client a warning and will not generate the client otp.     
  - [Success] It generates the client otp code for the user to manually input.

##### pros

- User cannot provide their password or 2FA code to a fraudulent server.
- User cannot be offline "rubber-hosed" into giving the access as they don't actually know their 2FA code at any given time.
- As the flow is so explicit it makes it more clear to potential users of this approach how it works. 

##### cons

- The user experience is poor, and most likely wouldn't be well adopted.
- The protocol isn't opt-in

### 2nd iteration

Some feedback I had from a few friends was that I could use HTTP headers instead.
While I could see that this would be an improvement, for some reason I thought knowing when the exchange was to occur would have many edge cases. It turns out that it was ok.

*Thanks to [@vkm514](https://twitter.com/vkm514) for helping me refine this!*

So now the demo, as it is currently at [veerless.josephkirwin.com](https://veerless.josephkirwin.com) works as follows:

- On the site's login page it presents an empty `X-Veerless-Init` header, and a username field. (Note as with all of these, the login page IP must be static or a static IP block, as that is embedded in the token to prevent site-in-the-middle attacks)
- The chrome extension uses a beta API called [declarative web requests](https://developer.chrome.com/extensions/declarativeWebRequest) which compiles static rules presumably to the v8 level instead of just JavaScript level like the [WebRequest](https://developer.chrome.com/extensions/webRequest) API. These rules are listening for the various HTTP headers.
- Once it receives this, it then begins listening for an `X-Veerless-Response` header containing the server's otp code that corresponds the their username.
- The plugin checks this otp code against their expected one
  - [Fail] Cancels the web response, and pops up a notification detailing why.
  - [Success] Allows the user to continue and pops up their client 2FA in a notification box.

##### pros

- (As above) User cannot provide their password or 2FA code to a fraudulent server.
- (As above) User cannot be offline "rubber-hosed" into giving the access as they don't actually know their 2FA code at any given time.
- User does not experience anything different from the current login flow they've become accustomed to.
- The extension can provide a better experience via blocking the fraudulent site immediately.

##### cons
*these are evolving the more refined the solution becomes*

- In all this there's still not a clear approach for the client to manage otps for multiple sites. (more on that shortly)
- It's not as evident to the audience receiving the demo what is actually happening, I need a way to display the HTTP headers to them while they login.
- Securing the serverSecret clientside may be difficult. There are options to either move the secret handling code to an Android app and have the chrome extension relay just the info and dns resolution over [Google-Cloud-Messaging](https://developers.google.com/cloud-messaging/) or instead take an approach outlined in my blog post [https://www.josephkirwin.com/2016/09/12/server-authentication-with-lamports-scheme/](https://www.josephkirwin.com/2016/09/12/server-authentication-with-lamports-scheme/) where only integrity is required for the client, not confidentiality.


### Future Work

The problem I thought of with client side credential management

> The user needs to know which otp to use for the given server, in our example we only use the demo site so its not a problem. But if you have many otps, you don't want to have to check them all for a match each time. 

> You do not want to index with the username as thats mostly the same across sites and you don't want to index using dns as this whole design is supposed to avoid that specific dependency. So what do you index the otp's with? 

##### My proposed solution.

So recall from [Serverside One-Time-Pad (Part2)](https://www.josephkirwin.com/2016/08/05/serverside-otp-part2/) we had the HOTP stuff, well its actually using TOTP for the implementation, but that is not the key takeaway here. So the equation is


$$TOTP = truncate(hmac(serverSecret,timeStep|serverIP))$$

A couple of key HMAC properties that make this great in comparison to a hash

1. Knowing the message part of a HMAC doesn't (or shouldn't if its a well selected HMAC) allow you to create an existential forgery. 
2. Upon truncation [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) is much more resilient to collisions in the same keyspace then a hash algorithm is, which is of course an un-keyed function. This is how TOTP and HOTP are relatively secure under truncation, but a SHA family hash alone may not be.

So based on these 2 concepts, we can add another `X-Header` into the protocol.     
Called `X-Veerless-Seed` that contains the t0 value to compute the time step for the HMAC.
The likelihood of a user creating 2 usernames at the same time would be quite rare, and this edge case could be handled via client-side rejection of t0 at the registration step with the server.

With this header in place, the client can use `X-Veerless-Seed` as an index into their table of available otps. 

t0      |username|serverSecret|
--------|--------|------------|
1234567 | joe    | abcd135256 |
8901234 | cool75 | 36cdef0561 |
...     | ...    | ...        |

**Check out the demo at [veerless.josephkirwin.com](https://veerless.josephkirwin.com) and please give me some feedback in the comments!** 

