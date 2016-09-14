---
layout: post
title: Lamport's Hash Chains for Server Authentication
type: post
published: true
status: publish
categories:
- authentication
- identity
---
My recent blog posts have had a unifying theme: how can a server prove to a client its legitimacy prior to client authentication?

I spoke to some authentication experts about my [previous](/2016/08/02/serverside-otp-part1/) [ideas](/2016/08/05/serverside-otp-part2/) on this topic.
This compelled me to think of some improvements.

The [2nd](/2016/08/05/serverside-otp-part2/) iteration was an improvement as it provided protection against façade-forwarding of the otp token. But still it required the client to protect the confidentiality of a secret. Ideally I would like it if the client only had to guarantee integrity, but didn't have to guarantee confidentiality [[1](https://en.wikipedia.org/wiki/Information_security#Key_concepts)]. 

In brainstorming this further, I recalled a scheme devised by the brilliant, Turing-award winning [Leslie Lamport](https://en.wikipedia.org/wiki/Leslie_Lamport) (was that introduction fanboi-ish enough?!). He devised something that was later used for [S/Key](https://en.wikipedia.org/wiki/S/KEY) known as [Hash Chains](https://en.wikipedia.org/wiki/Hash_chain). To summarize, the server maintains a forward hash of a secret, and the client presents the pre-image of that.

So, to use this for serverside authentication, how about we use Lamport's Scheme in the opposite direction, and also include the IP address?

#### Proposed Scheme

###### Initial Exchange  
*server gives the client :*          
$$hash_{999} = hash(server\_IP|hash_{998})$$

###### Future Authentications
*The client visits the site at some later time, and the server presents this:*             
$$hash_{998}$$

*so the client can then check:*         
$$hash(DNS\_resolved\_IP|hash_{998}) == hash_{999}$$     

If this comparison is false (and the solution is engineered adequately) then it's likely that the server is not legitimate.

#### What about the first hash, what's the original plaintext?

*The bootstrapping sequence server side would need to be:*              
$$hash_{1} = hash(IP|random)$$           
$$hash_{2} = hash(IP|hash_{1})$$               
$$hash_{3} = hash(IP|hash_{2})$$               
$$...$$              

The client would count down from $$hash_{999}$$ but stop at $$hash_{2}$$. At which point after the client is authenticated, the server would require a password reset and a hash-count reset.

#### What is good about this?

1. If malware on the client machine can exfiltrate $$hash_{999}$$, a pre-image attack on that to obtain 
$$IP|hash_{998}$$ is difficult (Even with poor choices like MD5 [[humbled-by-md5](/2016/05/05/humbled-by-md5/)]). Therefore if the client can guarantee anti-tamper of the hash that it is storing, but not confidentiality, it will still be able to securely authenticate the server.
2. We can synchronize the time to reset the hash counter ( i.e. when it reaches 2) with a recommended password reset based on the permutation strength of the password stored. (*I've been meaning to blog about computing the time required to crack a password based on the permutation and length. Basically, assume as soon as the password is hashed, it will be breached, then estimate how long you have.*)     
3. It may look at first like binding the IP address into the hashing scheme is overly rigid. It is. But this could be loosened to an dedicated IP range. Which in IPv6 which would be much less likely to change over time.

#### Caveats

* The initial client sign-up is not at all protected via this scheme. [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) is a great mitigation for this, but only if the site is on the browser [preload list](https://hstspreload.appspot.com/).
* In this design, it is assumed that malware, phishing and dns hijacking are easier to perform than hijacking a statically allocated IP range. 

*Again, questions + comments, contact via the links below.*
