---
layout: post
title: Humbled by MD5 
type: post
published: true
status: publish
categories:
- Android
---

When working on a recent android project, I came across the fact that even though Google recommends that all Android APKs should use SHA1andRSA or better when signing packages. I found that they themselves use MD5withRSA for their apps. 

`keytool -list -printcert -jarfile YouTube.apk` (there's a lot of different ways to check this!)

```
Signer #1:

Signature:

Owner: CN=Unknown, OU="Google, Inc", O="Google, Inc", L=Mountain View, ST=CA, C=US
Issuer: CN=Unknown, OU="Google, Inc", O="Google, Inc", L=Mountain View, ST=CA, C=US
Serial number: 4934987e
Valid from: Mon Dec 01 18:07:58 PST 2008 until: Fri Apr 18 19:07:58 PDT 2036
Certificate fingerprints:
	 MD5:  D0:46:FC:5D:1F:C3:CD:0E:57:C5:44:40:97:CD:54:49
	 SHA1: 24:BB:24:C0:5E:47:E0:AE:FA:68:A5:8A:76:61:79:D9:B6:13:A6:00
	 SHA256: 3D:7A:12:23:01:9A:A3:9D:9E:A0:E3:43:6A:B7:C0:89:6B:FB:4F:B6:79:F4:DE:5F:E7:C2:3F:32:6C:8F:99:4A
	 Signature algorithm name: MD5withRSA
	 Version: 1
```

Note these are self signed, which is unfortunate. I had tried to find some info to see why they chose not to have an actual CA. The Play Store acts as a shim for this "root of trust" but the story for apk sideloading would benefit from stronger identity verification... 

### My original naivety

When I first began working in information security, MD5 was already touted as something that should not be used because it is **HORRIBLY BROKEN**!

After observing this Google apk quirk, I had a quick read about [HashClash](https://marc-stevens.nl/p/hashclash/) and the many ways that you could collide MD5s of images and php scripts etc. 

But I didn't delve into the detail at that point, I just thought about the question 

> 
"Wouldn't it be interesting if someone could create a forgery of YouTube with the same certificate?"

I clearly wasn't the only one dismissing MD5withRSA as insecure, [F-Droid](https://gitlab.com/fdroid/fdroidserver/issues/26) had a similar panic, earlier this year.

### A Brief, rudimentary overview of how android apk signing works

The apk is a jar really, it uses a variant of jarsigner ([SignApk](https://android.googlesource.com/platform/build/+/7e447ed/tools/signapk/SignApk.java)) to create its certificates.

When dealing with apk signing, the directory of interest inside the apk is:     
/META-INF/     
  - MANIFEST.MF     
  - CERT.SF     
  - CERT.RSA     

The CERT.SF contains digests of all the objects that need to be signed, these are SHA1 or better usually. This somewhat duplicates the MANIFEST.MF contents and even contains a digest of the manifest itself. CERT.RSA is then the signature over CERT.SF. (Note the hashing is used, as digital signatures can only sign a limited amount of data efficiently and hashing helps thunk the size down) 

_If (unlike linux) for you more is better than less you can checkout [nelenkov's blog](https://nelenkov.blogspot.ca/2013/04/android-code-signing.html?view=classic) for very detailed info on this topic._

### Why I think Google's decision to not upgrade MD5 is a fine and secure one

There are 3 categories of attack on a hash algorithm.     
1. pre-image attack     
   `Given hash(message1) it is difficult to find message1`     
2. second pre-image attack     
   `Given message1 it should be difficult to find a message2 such that`    
   `hash(message1) == hash(message2) and message1 != message2`      
3. collision attack     
  `Choose a message1 != message2 where hash(message1) == hash(message2)`    

MD5 is not known to be weak to 1 or 2. But it is weak to 3, and there is a special case for hash functions using [merkle-damgard constructions](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction), of a ["chosen prefix collision attack"](http://www.mathstat.dal.ca/~selinger/md5collision/).

I'm now alluding to the fact that as pre-image and second pre-image attacks are not possible, a forgery of the apk is not possible. As option 3 (and its more optimal variant) require that the attacker has control of both messages, i.e would need to modify YouTube.apk, hence changing the hash, and thwarting the forgery.

So actually as Google Play restricts other applications from using MD5withRSA for normal dev apps, they've mitigated a potential risk of a malicious developer POC-ing a chosen-prefix-collision attack on 2 apps that they control (I'm still not sure what result could be achieved by this attack)

>
Other than that, as MD5 is secure for #1 and #2 above, this is actually a fine approach.

#### Some interesting "_actual_" research on issues with Android packages
-[ 2015 - Blackhat : "What you can do to an apk without its private key"](https://www.blackhat.com/docs/ldn-15/materials/london-15-Xiao-What-Can-You-Do-To-An-APK-Without-Its-Private-Key-wp.pdf)    
-[ 2013 - University of Amsterdam : "Weak key cracking of Android applications"](https://os3.nl/_media/2013-2014/courses/ot/cedric_sharon.pdf)    
-[ 2014 - Virus Bulletin : "Leaving our zip undone: How to abuse zip to deliver malware apps" ](https://www.virusbulletin.com/uploads/pdf/conference/vb2014/VB2014-Panakkal.pdf)    
