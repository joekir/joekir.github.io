---
layout: post
title: An idea for improving push based two-factor authentication
comments: true
type: post
published: true
status: publish
categories:
- 2FA
- Veerless
---

Over the past 6 months I researched bidirectional two-factor auth for TOTP/HOTP approaches (see [Veerless](https://veerless.josephkirwin.com/)). This year I spoke at a conference about it and since then I've been asked occasionally about my opinions on push-based 2FA solutions e.g. Duo Push or Okta Push.

In the rest of this post I'll refer to TOTP/HOTP style as "inline" and I'll refer to push-based/sms as "out-of-band". As to me these are the 2 general classes of 2FA.

### Strengths of inline

- Robust against password dumps, without custom social engineering there is no way for attackers to utilize the passwords en masse. 
- No network requirement. Sure there is a network requirement for the primary factor, but the mobile device's TOTP solution can be completely offline.

### Strengths of out-of-band

- The push provider can defend against the inline phishing approach that the Veerless design mitigated. General phishing is not possible, as the user doesn't provide a 2nd factor directly.
- Less user typing, aka friction (the enemy of security!)

## Weakness in push based two-factor auth

The key problem I see is actually a user behaviour one. If this can be reduced then out-of-band may in most cases be preferable to inline. 

Imagine a user receives many notifications on their device per day. Security minded people may be quite diligent about dealing with these, however people that aren't concerned may have no qualms to ok a big green button, just to get the noise out of their way.

With push based 2FA the security burden is on the user to tie context between an event that occurred on one device (primary password login) and some other notification on another device (mobile 2nd factor confirmation). 

My suspicion is that a there is a certain group of users that would confirm a push notification regardless of whether they had visited the primary login site.

## A solution to bind cognition - Snap2FA

*In the game of [snap](https://en.wikipedia.org/wiki/Slapjack#Snap) you and your adversary turn over a face-down deck of cards until there are 2 consecutive matches, then you slam your hand on the pile saying "SNAP!"*

The proposed "Happy Path" flow:

* user visits example[.]com/login on their laptop/tablet
* they're presented with the usual primary factors (username/password)
* they're also presented with a 3x4 grid of pictures/tiles/images

<img alt="A 3x4 grid of random pictures" src="/assets/Snap2FA_grid1.jpg" width="480" height="320" />

* they select one that they like the best
* the server verifies the primary factors, notes the image the user chose and sends the array of images in a random order as a push request to the device
* On the device the user sees the 3x4 grid, but now the order is different. This prevents users from always choosing top-left for example

<img alt="The 3x4 grid of random pictures, but in a different order" src="/assets/Snap2FA_grid2.jpg" width="480" height="320" />

* for the user to confirm that they made the original request they shout **"SNAPPPP !!!eleventy111!"** as loud as they can in the library and match the image.

In the case where an attacker obtains a huge password dump and does a "spray and pray" in the hopes some users will **approve** on the push 2FA. The user will see a grid and be told to choose a picture. Their options at this point:

1. I don't remember looking at any images, **THIS IS A PHISH!!!**
2. Meh, I like the look of this Skull n crossbones one (1 in 12 chance of a user accidentally pwning themselves ~ 8%)

## Conclusion

Type 1 users above would most likely not be phished by an out-of-band 2fa when they hadn't clicked the primary, so no big benefit for them, but also not much friction increase. 

However this is a massive win for the type 2 users. I don't believe any user actually wants to compromise themselves, so putting that aside. The chance of them dismissing this and accidentally choosing the correct one for th attacker has gone from a 50/50 choice (probably more biased in the **approve/yes** camp due to social/psychological conditioning) to only 8% chance.

If someone (Duo/Okta) wants to conduct some experiments to see if these type 2 users exist, go for it and please share with the community :D 
