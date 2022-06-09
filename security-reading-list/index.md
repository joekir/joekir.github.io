---
layout: page
title: Security Reading List
---

### Security Books You <u>Should</u> Read

_I've had quite a few people that are new to the field of information security ask about some "key reads", so instead of replying with the same info each time I created a link to that for ease-of-reference (and I will keep this evergreen!)._

| Title          | Topic					| Why |
| :-------------|:------------- |:-----|
|[Secure By Design](https://www.manning.com/books/secure-by-design){:target="\_blank"}|Application Security|If you are a developer, and you have to read only one of these books read this one. It will allow you to "design away" a huge swathe of vulnerabilities from the software you write, if you can achieve this you will be loved by your security team(s).|
|[Serious Cryptography](https://nostarch.com/seriouscrypto){:target="\_blank"}|Cryptography|The most currently relevant cryptography book I've read, even provides an introduction to quantum computing and what post-quantum attacks look like on classical cryptography schemes. This author is well known and has contributed to major advances like [siphash](https://www.aumasson.jp/siphash/){:target="\_blank"}, [BLAKE](https://www.blake2.net/){:target="\_blank"} and [Argon2](https://github.com/P-H-C/phc-winner-argon2){:target="\_blank"}, this should illustrate to you they aren't just a theorist but also a practitioner!|
|[Threat Modeling - Designing for Security](https://threatmodelingbook.com){:target="\_blank"}|Application Security|The de-facto Threat Modeling guide, goes from fundamentals to more advanced techniques that I haven't used in practice. Even if you read 25% of this book you'd advance your mental model for application security.|
|[Silence on the Wire](https://nostarch.com/silence.htm){:target="\_blank"}|Network Security|For anyone that has tried to do network forensics or reverse-engineering, you'll be nodding your head throughout this book. Surprisingly, this book still feels relevant in the "Web 2.0&trade;" world we're currently in (approx. 15 years ago at time of writing this). This book will help you realize that anonymity over a network is difficult, which is challenge for privacy and a boon for network defenders!|
|[The Art of Software Security Assessment](https://www.pearson.com/us/higher-education/program/Dowd-Art-of-Software-Security-Assessment-The-Identifying-and-Preventing-Software-Vulnerabilities/PGM306255.html){:target="\_blank"}|Vuln Hunting|If you want to someday to be able to read and comprehend a [Google Project Zero](https://googleprojectzero.blogspot.com/){:target="\_blank"} write-up then this is how you get ready for that. This is very comprehensive and used to be the required reading though perhaps in cloud security age it may be aging fast.|
|[How to Measure Anything](https://www.howtomeasureanything.com/3rd-edition/){:target="\_blank"}|Quantification of Risk|Firstly, don't let the _"cheesyness"_ of the websites and quotes around this book discourage you. Sadly they do themselves no favours by making it look like some MLM scheme. This really is legit, and backed by basic mathematical foundations. Additionally the author later attempted an application of this to cybersecurity ([How to Measure Anything in Cybersecurity Risk](https://www.howtomeasureanything.com/cybersecurity/){:target="\_blank"}), I'd only recommmend that after reading the first book as it skips some of the fundamental points that are more useful than the domain-context exercise. <br><br>You'll (hopefully) reach an epiphany that currently in software development and security we model far too many things as a Boolean, when in fact everything has a confidence-interval. Risk estimation is not just about estimating the likelihood and impact of an event, but also your own (and your systems) confidence in what those estimates. Once you have those you can tolerate far more uncertainty while still prioritizing what needs to be worked on next.|

### Security Books you <u>Should Not</u> Read

_There are seriously too many to list, lots of bad ones, I think I've read about 20 bad security books. However, the obvious advice is never read **anything** by Kevin Mitnick._
