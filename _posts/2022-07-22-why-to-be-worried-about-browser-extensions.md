---
layout: post
published: true
status: publish
title: 'Why to be concerned about Browser Extensions?'
type: post
---

_I don't care about the tech stuff [skip](#what-do-i-do-to-protect-myself) to the safety recommendations._

### Imagine you have an online kitchen store

<div class="mermaid">
flowchart LR
    classDef vulnerable fill:#f1ac56
    classDef critical fill:#ffcccb
    classDef safe fill:#daf7a6
    classDef resetColor fill:#ffffff

    subgraph Browser
        direction LR
        subgraph ActiveTab[Active Tab]
            websiteJS[kitchen_store.js]
            contentJS1[content.js]
            subgraph iframe[fa:fa-lock iframe]
                paymentJS[payment.js]
                contentJS[content.js]
            end 
        end

        subgraph ExtensionBackend[Extension Backend]
            backendJS[malicious_backend.js]
        end
    end

    subgraph paymentSite[Payment Site]
    end

    class Browser,ActiveTab,iframe,ExtensionBackend resetColor
    class backendJS,contentJS,contentJS1 critical
    class paymentSite,paymentJS,websiteJS safe

    contentJS --> backendJS
    contentJS1 --> backendJS
    paymentJS -- https --> paymentSite
    linkStyle 0 stroke-dasharray: 5 5,stroke:red
    linkStyle 1 stroke-dasharray: 5 5,stroke:red
</div>

In this store, you have some JavaScript for all your kitchen-y stuff, like spatulas and stuff.
For anything that's risky, like handling credit-cards for customer payments you use a 3rd party like Adyen, PayPal or Stripe.
To do this, they recommend putting it in an inline frame ([iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)) and this also allows you to wrap that content in a content security policy ([CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)). 

You would add the CSP mostly to ensure that even if there's some scary code that can get into that iframe, it can only send data to the payment site's hostname, so any _credit card numbers_ that a customer puts in there are safe from being sent to the wrong place.

Ok, so what's bad?

1. While a website (your kitchen store) can control where the iframe can send data, **by design** Browser Extensions are EXEMPT from this policy!! 
    !["I do what I want"](/assets/i-will-do-what-i-want.gif)
2. the extension's [content scripts](https://developer.chrome.com/docs/extensions/mv3/content_scripts/) can be loaded into any and all frames in the browser with simple bit of seemingly-innocuous config:
```json
{
    ...
    "all_frames": true,
    ...
}
```
3. Through a well-defined communication mechanism the `content.js` script is able to exfiltrate any data it can access out to `malicious_backend.js`, and that script is then free to exfiltrate data from the browser to a domain of the attacker's choosing :(

### What do I do to protect myself?

**Browser Users**

Don't use browser extensions. The only one you actually need is an adblocker, to achieve this without a Browser Extension here's the recommendations (either would suffice):
  - Use [Brave Browser](https://brave.com) with its built-in adblocker (it's Chromium under the hood anyway).
  - Use a custom DNS resolver such as [NextDNS](https://nextdns.io/?from=jsbhvf3z) or host it yourself on a [pi-hole](https://pi-hole.net/) to drop the DNS for advertising domains.

**Website Owners**

Be aware that you can't do anything to protect your customers from this threat.
If they start reporting anomalous thefts and blaming you, at least add this discussion to your support troubleshooting list.

Btw you can't enumerate the list of browser-extensions present for privacy reasons, I can attest after having exhaustively tried.

**Browser Developers**

Figure out a solution? I made some suggestions here that weren't well received:
* [bugzilla](https://bugzilla.mozilla.org/show_bug.cgi?id=1629624)
* [bugs.chromium.org](https://bugs.chromium.org/p/chromium/issues/detail?id=1070315)

Time to use those brains and come up with a genius solution to this mess for consumers.

### Some recent improvements

No, the problem is <u>still</u> here, a malicious browser extension can still steal any of your data, regardless of what the website does to help protect you. But there's been a few changes that at least make the maliciousness more gauche and (theoretically) detectable by the AppStores. That being said, the [2018 announcement to ban obfuscation](https://blog.chromium.org/2018/10/trustworthy-chrome-extensions-by-default.html) doesn't seem to have made any difference, I flagged an app the other day using [jscrambler](https://jscrambler.com/) the other day.

In [Manifest V3](https://developer.chrome.com/docs/extensions/mv3/intro/mv3-overview/) some safety changes have landed, such as not allowing remote script to execute in an extension!! I didn't actually know that was allowed in V2... how was that ever thought to be a good idea from Chrome/Firefox store review perspective?!

Background Scripts are now replaced with Service Workers; however, no changes to where content scripts can be injected or adherence to CSP.
