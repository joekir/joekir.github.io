---
layout: post
title: What if we had mutex revocation lists?
type: post
published: true
status: publish
categories:
- malware
---
#### Background

This is an idea I thought of in late 2014. Recently I reflected on it and wondered why it still doesn’t exist. Please use the contact details in the footer if you want to lambaste me about why it should not. But first let me pitch it!

When I worked in Anti-Malware I observed the creations of system mutexes ([mutually-exclusive lock](https://msdn.microsoft.com/en-us/library/hw29w7t1.aspx)). The most common reason for this was to allow the application to check if it had already infected that machine. The reason malware authors cared about that was that any useless work performed needlessly opens up risk to detection. Other use cases I observed were for multi-threaded malware applications. 

The error handling for the the acquisition of these mutexes was on occasion naive. A user could immunise their machine against some malware by creating and acquiring the same mutex. Forcing the malware crash when it didn't gracefully handle the error.

However using long running powershell scripts to continually hold mutexes did not seem like an ideal design. Instead I wanted to see if I could override the mutex acquisition code in the framework to request it check a blacklist first. To do this I read one of Erez Metula’s (Intermediate language reversing guru) wonderful papers [[1](https://appsec-labs.com/wp-content/uploads/2014/10/NET-Framework-rootkits-backdoors-inside-your-framework.pdf)]. 

What Erez discusses in that paper is terrifying. Most AV vendors would have trouble discovering framework level injection, as it's common for a customer to send in the exe or dll for analysis that they believe is exhibiting malicious behaviour, however they may send in a benign program who’s api calls are being manipulated at the framework level.

A quote from Erez’s paper that I don’t quite agree with:

> “The Strong-Naming mechanism does not check the actual signature of a loaded DLL but just blindly loads a DLL from inside a directory containing the DLL signature string.”

Though I haven’t analysed SN, from my experimentation I have observed that the binary appears to only be partially integrity-checked. In Erez’s paper they manipulated the binary within the bounds of not modifying the .NET metadata directory. This may not be a design flaw for .NET, the framework may require the lack of integrity checks for things like code generation, I don’t know.

The restrictions to modifying the framework's IL are:      
* Can only use APIs already declared in the header. (#Strings stream)      
* Can only reuse text already declared in the #US (user-strings) stream.       
* Line numbers and max-stack need to be updated.     

#### A better alternative to D.I.Y

A Mutex Revocation List (MRL).

What if the framework itself could be provisioned with a mutex blacklist endpoint to call, a local blacklist cache, or even a hybrid of both?

I use the analogy of CRL ([Certificate Revocation List](https://technet.microsoft.com/en-us/library/ee619754.aspx)) as it has the notion of a hybrid approach for validation. Otherwise just think of this idea as the framework allowing the user to specify a mutex-blacklist via CLR Security Policy or Java/Dalvik VM Policy.

If the format of such a list could be standardized amongst framework vendors then a domain administrator or end user could choose to configure it to subscribe to a list of their choosing, e.g. virustotal.

##### Arguments against this approach *(that I’m teeing up for defeat :P )*

> Doesn’t this devolve to `match-the-hash`, the classically flawed AV vendor game?

No, I don't think so. As the malware authors use these markers to check if they have already infected a specific machine, the mutex names can’t be truly random

*(unless they use a CNC to coordinate the truly-random names, but in that case the malware's dependancy on establishing that communication channel would increase its chances of detection too. So if that is the outcome, it is still an improvement for defenders)*

they need to be either static within the version of the malware family that was published, or they need to be created by a generation algorithm such that each strain of the malware is able to query the mutex's existance. If this is the case, that logic could be reverse engineered and published to a backend such that an api call could test for a mutex family.

Hence they are cornered into using either non-random naming, or choosing some other system infection marker such as the file system or network, which is much more heavily watched by most AV agents.

*Questions + comments, contact via the links below.*
