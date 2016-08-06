---
layout: post
title: Serverside One-Time-Pad (Part1)
type: post
published: true
status: publish
categories:
- authentication
- identity
---

***N.B. I've since written a [Part2](/2016/08/05/serverside-otp-part2/) to this, to address some original implementation concerns.***


It has become almost normal to see 2-factor-authentication on many of the most popular sites on the internet. The strongest of these schemes are the offline schemes: Time-Based One-Time Password ([TOTP](https://tools.ietf.org/html/rfc6238)) and Event-Based One-Time Password (EOTP/[HOTP](https://www.ietf.org/rfc/rfc4226.txt)).

These are strong at protecting unauthorized access, however I think we're missing out on some impersonation protection that they could provide.

A scenario to illustrate the problem:

>*Joe is presented with a phishing site that impersonates some site that he normally uses with 2FA. He's not savvy enough to notice the phish, it looks and behaves identically, has a valid TLS certificate etc.*

>*He inputs his username and password, and the site directs to the next stage in the login flow; the TOTP page. Still the site looks legitimate, so he looks at Google Authenticator and inputs the code that it shows.*

>*The malicious adversary hosting the site, then uses Joe's username, password and TOTP on the real site to fraudulently login as Joe.*

>*The implementation changes from site to site, but the RFC recommends leaving the code valid for greater than the 30 second limit that you see on apps like Google Authenticator, this is to account for network latency or resource competition serverside. Meaning the attacker would have time to use this phish in realtime.*

>*The legitimate site has no way of knowing this phish as these OTP algorithms are offline by design.*

But we could do much better here!

Both RFC's ([TOTP](https://tools.ietf.org/html/rfc6238) and [HOTP](https://www.ietf.org/rfc/rfc4226.txt)) require an initial seed to be shared between server and client, such that they can generate subsequent codes.

**At this time, why could we not initialize both a server-side seed and a client-side seed?**

The login flow would then be unchanged except for:    

- Google Authenticator/Authy would show:     

| server code | client code |
:----------:|:-----------:|
a4bd3f|2e0cbf

- The login page would show its server code (which is somewhat guarded by the username/password combo, and also is not as sensitive as the client code, therefore could change at a different cadence to the client side). With a box to input the client code.

<form>
<label for="stotp">Server's code (TOTP): </label>
<input type="text" id="stotp" readonly value="a4bd3f"><br>
<label for="ctotp">Please Input Client TOTP:</label>
<input type="text" placeholder="2e0cbf" name="client totp" id="ctotp"/>
<button type="button">Submit</button>
</form>


The user could then check that the server code was correct, prior to inputting the client code. 

From the attacker's perspective, this means that with phishing they could still successfully capture the user's username/password. But they may not be able to obtain the TOTP code, as the user would have a decent metric to detect a server-side forgery. The user could then change their password after this event of distrust occured. 

*Sidenote, I've also been looking into Zero-knowledge-password-proof (ZKPP) schemes to acheive server and client mutual agreement on passwords. It's a bit more involved than this solution though, I guess I'll detail that later.*
