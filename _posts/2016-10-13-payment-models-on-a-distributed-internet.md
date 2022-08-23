---
layout: post
title: Payment models on a distributed Internet 
comments: true
type: post
published: true
status: publish
categories:
- authentication
- distributed
---

I really think [ipfs.io](https://ipfs.io) (**not** ipfs.com) is an excellent concept, the execution is slick and the research of prior efforts in that direction are diligent.

Recently I [listened to Juan Bennett](https://changelog.com/podcast/204) talk about IPFS and some examples on the changelog podcast. He gave examples of IRC on top of IPFS, which is a very compelling dynamic content story. IPFS for free static content isn't a conceptual reach either.

However, there are some things that exist in the current server-client internet that are tough to conceptualize if we move from HTTP to IPFS. Take the example of a movie that I want to stream from Netflix. Getting a large media file from a local peer would be really great speed improvement and would ease the load on the Internet backbone even if peer-to-peer was used for media streaming alone [[1](https://variety.com/2015/digital/news/netflix-bandwidth-usage-internet-traffic-1201507187/)].

But how could the payment for the content work?

1) **1 person pays, many people watch**

We descend to how it is currently with torrents, someone takes the payment hit to obtain the content and then distributes it freely amongst their peers.  

One might recall that there are cryptographic approaches to allow multiple, specific users to decrypt a blob of data.

e.g. Document is encrypted with a symmetric key $$K_{D}$$ and then it is appended with a trailer of the symmetric key encrypted with each valid user's public key $$Enc_{pub user 1}(K_{D})$$, $$Enc_{pub user 2}(K_{D})$$, $$Enc_{pub user 3}(K_{D})$$... . While this is good in a `loyal generals` scenario, it's not so great in a `traitorous generals` scenario [[2](https://en.wikipedia.org/wiki/Byzantine_fault\_tolerance)]. Because once a single user obtains $$K_{D}$$ they could distribute the unencrypted media to all peers on the network.

2) **Nobody pays, but everyone is implored to donate**

Similar to how patreon works (though hopefully more secure) where the media is freely distributed using a peer protocol. But there is a centralized or even decentralized donation portal. This still could be financially successful, an example being Radiohead's album ["In Rainbows"](https://en.wikipedia.org/wiki/In_Rainbows) that was offered as a pay-what-you-wish in 2007.

3) **Pay for your shard**

The most similar concept on the current Internet I can think of would be a Kickstarter campaign. The content is not released until the money is obtained to create it.

I'd envisage the content creators provide a unique encrypted fragment to each of their customers. The amount of fragmentation would equate to how much they need to make on the content. Once they have reached their goals the network would have the all the shards and then the whole network could obtain the full content. 

This creates an interesting anti-greed design, such that a company only earns based on what it thinks the market will tolerate. But they will not earn more than this, unless the price of shards is not constant.

If the demand for the product is not as high as they expected, then they can release the remaining shards for free at a given date.


**Conclusion**

I really can't see any businesses opting for the first 2 options. So to me it seems that either we have a tiered internet whereby paid content can only use server-client and free content is peer-to-peer.

The alternative is to embrace a new paradigm where the content only becomes available once a certain amount of people pay for it.

*If there are other payment approaches that you can think of please comment.*




