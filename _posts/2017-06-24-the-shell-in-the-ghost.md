---
layout: post
title: The Shell in the Ghost 
comments: true
type: post
published: true
status: publish
categories:
- Cloud
- Security
---
!["grayfox deviant art"](https://pre01.deviantart.net/3f19/th/pre/f/2014/360/a/7/gray_fox_metal_gear_solid_stealth_camo_by_ourouterheaven-d8bdu9g.jpg)
I recently came across an incredible concept and implementation by [Michael Rash](http://cipherdyne.org/author.html){:target="_blank"}. It's called [fwknop](http://www.cipherdyne.org/fwknop/){:target="_blank"} which stands for `Fire Wall KNock OPerator`, **wait though, don't stop reading yet!!**

Before you close the browser in utter disgust at me suggesting port-knocking in 2017, **this isn't just port knocking**, in fact it's a shame it's even called that as it leads to confusion with [certain people mistaking](https://hn.algolia.com/?utm_source=opensearch&utm_medium=search&utm_campaign=opensearch&query=fwknop&sort=byPopularity&prefix&page=0&dateRange=all&type=story){:target="_blank"} it for `security-thru-obscurity`

*Allow me to paraphrase/describe what the concept is here*

!["fwknop flow"](/assets/fwknopd_flow.svg)


* All ports on the server are set to default DENY 
* The fwknop daemon (fwknopd) will sit behind a udp port (defaults to udp/62201) and sniff the dropped packets
* It is looking for a SPA (Single-Packet-Auth) from the client in the form of either a HMAC'd or GPG signed value of `source_IP + port_to_open` (source is included to prevent replay attacks in the event of a packet capture)
* If the fwknopd finds a valid signed packet it will then open the requested port e.g. tcp/22 for ssh temporarily for only that source IP address.
	* A neat piece to this is the addition of the IPTABLES rule  
	```
		iptables -I INPUT 1 -i eth0 -p tcp -m conntrack \ 
			--ctstate ESTABLISHED,RELATED -j ACCEPT
	```
	so that established sessions will remain even after the temporary rule issued by fwknop has expired
* The client can then connect via ssh (*no excuse for anything less than key based!!*) as they would normally

The crux to this is leveraging the fact that UDP packets don't require a handshake, and also that fwknopd is sniffing instead of filtering packets hence there is little opportunity for side channel attacks

For any attacker scanning this box it essentially doesn't have any ports open. Coupled with the cryptography aspect to SPA, it wouldn't be a good use of an attacker's time to try brute force or even perform any level of reconaisance, as they just don't get any feedback from the box. I think this could be a great addition to hardening a cloud bastion, see my [previous post](/2017/06/13/secure-bosh-deployments-on-gcp/){:target="_blank"} for details on that setup in Google Cloud Platform

Makes me wonder how many of these "stealth-bastions" live out there 🤔     
**Go find some and tell me about how you did it!!**

### Reading

- [My guide to Setting up fwknop on GCP](https://gist.github.com/joekir/c721e42ac0a164ed3ed5fbf5fa709d24#file-setting-up-fwknop-md){:target="_blank"}
- [Michael Rash's fwknop tutorial](http://www.cipherdyne.org/fwknop/docs/fwknop-tutorial.html){:target="_blank"}
- [Digitalocean's guide to deploying fwknop](https://www.digitalocean.com/community/tutorials/how-to-use-fwknop-to-enable-single-packet-authentication-on-ubuntu-12-04){:target="_blank"}
