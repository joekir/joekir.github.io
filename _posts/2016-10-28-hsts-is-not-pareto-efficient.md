---
layout: post
title: HSTS isn't Pareto Efficient
comments: true
type: post
published: true
status: publish
categories:
- TLS 
- SSL
- Web Standards
- DDOS
---

*What a fancy title!* :unamused:

Just thought I'd throw it in there instead of saying "HSTS is great for some, at the expense of others!" I only just learned about this term during the first week of [this edX course](https://www.edx.org/course/cyber-security-economics-delftx-secon101x) on Cyber Security Economics. It's worth checking out.

I still find such few sites use HSTS (HTTP Strict Transport Security), fewer still use it with optimal settings.

### Why it's an excellent header for those that use it?

##### HSTS level 1

If you have HSTS setup for your site, with a reasonable cache expiry time, say 1 year (31536000). For an attacker to pharm/phish your users they would need to :         
- Compromise the DNS of one of your users.          
- Obtain a valid SSL certificate for your domain.        

The weakness of level 1 is that if they attack one of your users before they have ever visited your site, then they only need to satisfy option 1. As they could just server your site on http for the user.

##### HSTS level 2 

Same as above except you also add your domain to the preload list.
[https://hstspreload.appspot.com/](https://hstspreload.appspot.com/)

This handles the case where a user has never visited your site before and hence not cached the HSTS setting, as now it ships with the browsers.

##### Summary

When used correctly it's a giant hurdle for attackers to deal with if they with to pharm/phish credentials.

### Why it's a terrible header for those that don't use it?

Obviously those that don't use it won't have the benefits outlined above. But also here's a DOS (Denial of Service) scenario for all those sites that don't use HTTPS. I'll speculate on why they don't :    
 - They are concerned about performance...       
 - SSL Certificates are expensive. [#LetsEncrypt](https://letsencrypt.org/)         
 - Green locks are a rival brand's logo?          

##### Scenarios

**DOS for a given user**           
An attacker could compromise the local cache DNS for a user. This could be via malware, or cache poisoning or some other means. They then serve a fake version of the site, except its in HTTPS with the HSTS header set with the max cache expiry of 2 years (63072000). The attacker can then stop serving the site and manipulating DNS, and the user will not be able to connect to the original site as it is not served over http. 

**DOS for all users**         
An attacker compromises DNS somehow or even just hijacks the domain temporarily, and gets a valid certificate from a CA, they then add this to the HSTS preload list ([https://hstspreload.appspot.com/](https://hstspreload.appspot.com/)). Once the site owner regains control of the domain or the attacker stops compromising DNS, the site can no longer be visited by **ANY** user.

An excerpt from the [preload page](https://hstspreload.appspot.com/): 

> Be aware that inclusion in the preload list cannot easily be undone. Domains can be removed, but it takes months for a change to reach users with a Chrome update and we cannot make guarantees about other browsers. Don't request inclusion unless you're sure that you can support HTTPS for your entire site and all its subdomains the long term.

### Recommendations

1. Use SSL 
2. Use HSTS with a decent cache lifetime 
3. Consider adding it to the preload list if you meet the requirements.
 
Do not do 2 and 3 without 1 or you will disappear from the internet!










