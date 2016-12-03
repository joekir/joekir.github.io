---
layout: post
title: Introducing Elfin
comments: true
type: post
published: true
status: publish
categories:
- DNS 
- Attacks
- Defence
- OpSec
---
***--The homoglyph domain variant finder--***

While we wait for our anti-phishing technology to improve (see [Veerless](https://veerless.josephkirwin.com){:target="_blank"}) it seems like it would be proactive for companies to try acquire some of the cheap domains that look similar to theirs.

There are a couple of ways to do 'lookalikes':

- charset changes, e.g cyrlic ['O'](http://www.fileformat.info/info/unicode/char/043E/index.htm){:target="_blank"} vs ascii 'O' - it's tricky to distinguish, but fortunately this was mitigated somewhat by browsers rendering it as its [punycode](https://en.wikipedia.org/wiki/Punycode){:target="_blank"} form e.g. "xn- -" urls.
- [Leet](https://en.wikipedia.org/wiki/Leet){:target="_blank"} substitutions - these may be effective for extremely gullable victims, but if I were to prioritize a company's domain spend it wouldn't be for these.
- Exploiting that Basic Auth url schemes **!!!!STILL EXIST!!!!** I know, it's insane, but all the major browsers still support them. An example from the [bug I filed with chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=661005){:target="_blank"}:     
  `https://www.google.com:443+q=elon@tesla.com`      
  Sadly that does actually pass the username "www.google.com" and password "443+q=elon" to tesla.com.      
  Firefox, Safari and IE warn the user that this is about to happen and they can chooe to continue or not, however chrome/chromium just proceeds. No wonder its so easy to phish a user!
- [Homographs](https://en.wikipedia.org/wiki/IDN_homograph_attack){:target="_blank"}/[Homoglpyhs](https://en.wikipedia.org/wiki/Homoglyph){:target="_blank"}   - here we reach my favourite type, as there is not an easy solution available to the browser vendors, and they are almost indistinguishable from the real thing. This is what [elfin](https://elfin.josephkirwin.com){:target="_blank"} helps with. Examples of these would be "rn" looks like "m" or capital "i" looks like lowercase "l". 

#### Case Study - Cloudflare.com

The reverse whois lookup for registrar : "CLOUDFLARE, INC." returns all of [these](http://viewdns.info/reversewhois/?q=CLOUDFLARE%2C+INC.){:target="_blank"}. 

So they went to the length of purchasing "clloudfllare.com" however that isn't nearly as great of an attack as "cIoudflare.com" (notice that is actually an uppercase "i" in the second letter of the domain, the horrible thing about the more common webfonts like Arial is that they don't have kerning and they are often sans-serif, so it's very tricky to spot these frauds.  

Here's an example of the one's that Elfin found that cloudflare should look into - [https://elfin.josephkirwin.com/search?q=cloudflare.com](https://elfin.josephkirwin.com/search?q=cloudflare.com){:target="_blank"}

### Conclusion

Services like MarkMonitor can be very expensive and actually not comprohensive in acquiring all the domains that attackers may use to phish your customers. Instead I implore you to **go try elfin** (**it's free** ††) and see what homoglyph domain variants it can find for you!

**[https://elfin.josephkirwin.com](https://elfin.josephkirwin.com){:target="_blank"}**

†† - *note, you still need to purchase the domains yourself, but they're usually pretty cheap compared to the cost of phishing :D*
