---
layout: post
title: Using secure chat for vulnerability disclosure 
comments: true
type: post
published: true
status: publish
categories:
- Email
- Messaging
- Security

---

#### Preamble
When a security researcher wants to share a security issue with a company, they usually use PGP over email. This kind of sucks, mostly because it's common for the company to respond in plaintext totally ruining the secure line, which was for their benefit and their user's benefit anyway. 

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Always assume most people will respond to a PGP-encrypted email with a plaintext one. Quoting your original text. Happens all the time.</p>&mdash; Joanna Rutkowska (@rootkovska) <a href="https://twitter.com/rootkovska/status/830750306799607808">February 12, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script> *See, I'm not the only one to note this issue...*

So this got me thinking, why couldn't an incident response team have a service that "bootstraps" a security researcher into a secure chat? e.g. Signal/WhatsApp/Telegram/ChooseAnyChatServiceMoreSecureThanEmail

#### Main thought

What if this "bootstrap service" could also handle the privacy aspect? Allowing us to broker communication channels without disclosing the static handle or phone number?

I'm thinking a service could exist where you can obtain a GUID/UUID that temporarily points to a number/handle-for-a-service by which a company could contact you on. Signal/WhatsApp/Telegram/FB Messenger/Wickr/etc could have the ability to route communications to the service e.g https://comm-broker.service/&lt;GUID&gt; where this ephemeral route would then hand off to the real one, ideally without ever revealing the number but if it did that's not too bad, more that you don't have to have your number publicized for spammers.

Maybe next time I'm thinking about disclosing some bug to a company I'll offer them to contact me on [Signal](https://whispersystems.org/){:target="_blank"} instead of PGP encrypting my message to them. If someone else reading this has tried, let me know if it was tolerated. 
