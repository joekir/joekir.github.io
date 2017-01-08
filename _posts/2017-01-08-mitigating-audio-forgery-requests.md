---
layout: post
title: A suggestion on how to mitigate Cross Audio Request Forgery (CARF)
comments: true
type: post
published: true
status: publish
categories:
- Defence
- Alexa
- Siri

---

When the big tech players decided to create voice activated assistants (Siri/Alexa/Ok Google/Cortana/\<insert contender here\>) I think they were probably cognisant that attackers would use this opportunity to conduct the audio equivalent of a Cross-Site-Request-Forgery. If they were, why don't we have many protection mechanisms available for this?

### Current Protections

Since Android M, "Ok Google" has defaulted to use voice training to establish a profile of your voice and authenticate audio request, so someone else's voice would not trigger the request. This mechanism is not bulletproof as was shown by these [security researchers](http://www.theregister.co.uk/2015/10/15/headphones_make_smartphones_hackable/).

However based on [this mind-blowing talk from Adobe](http://www.bbc.com/news/technology-37899902), this type of protection will not be holding water for long, as the technology is now available to synthesize someone saying anything, providing you have a 20 minute recording of their voice, and with the correlation strong correlation of "fame" to "audio recordings available online" from a targeted attack perspective this will work well.

Also, something I haven't confirmed is whether Google's vocal profiling only covers the trigger phrase "Ok, Google", hence exposing it to vocal replay attacks. Or whether it authenticates all audio it hears. Still the Adobe approach above could bypass this protection. 

### My Suggestions

- BASIC : Allow user's to name their device. So instead of "Ok, Google. Show me how to get to the Guggenheim" they could say "Ok, Boris. What is 2+1?". This would reduce ubiquitous attacks, though the voice profiling already handles that. But also in the targeted Adobe synthesizer example the attacker would need to know the name of the victim's device. (default credentials are evil...)

- ADVANCED : 2 approaches, depending on the amount of co-users. But both based on the same concept that the name of the device changes per request or per unit time.
   - Change per request. This is only truly viable if it's a single user household. You have a dictionary (size n) or names, with the user's name or other common names removed from it. You then have a random seed value (t) less than n that correlates to the current name of the device on this request. Then after each request you multiply t by some random constant, modulo n and this becomes the new t. So every request the trigger name would change.

  - Change per unit time. This is the option to go with in a multi-user household, and is essentially a variant of Time-Based-One-Time-Pad (you know like on the Google Authenticator thingy!), but with a modulo n to find the given index t into the dictionary of potential device names. <br><br>example request each side computes:<br><br>
   $$TOTP = HMAC(secret  ||  \lfloor{(\frac{currentTime - initialTime}{timeStep})})$$
   <br>
   $$ t = TOTP\bmod{n} $$
   <br><br>
   This secret from the HMAC above would need to be synced amongst all the client devices, e.g. android wear app where the name changes every 30 seconds. 

Of the two "advanced" approaches I prefer the time-based, as there is no state for each device to maintain. In the event based approach imagine you say "Susan, find the nearest..." but the Alexa device did not hear that correctly. If your watch has now incremented it's index, you will be out of sync on the names. (note a tolerance threshold could be used to solve this) 

Thanks to the independence of state management in the time based approach this also accommodates multiple users right out of the gate, and you could even use different seeds for each user. Though there is no notion of access granularity for voice yet, so this would not be great in a [byzantine usage](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance#The_Byzantine_Generals.27_Problem), for example an office environment.

Something I overlooked? Other protections already available? Comment below :D

##### Related Reading
- [Alexa is not even remotely secure and really I dont care](https://gizmodo.com/alexa-is-not-even-remotely-secure-and-really-i-dont-car-1764761117)
- [Tweet by Jeremiah Grossman](https://twitter.com/jeremiahg/status/818191812066033664)


