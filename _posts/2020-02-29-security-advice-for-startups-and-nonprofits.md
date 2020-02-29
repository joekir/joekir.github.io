---
layout: post
title: Security Advice for Startups and Non-Profits 
comments: true
type: post
published: true
status: publish
---

I was thinking that this could be useful, I've worked security for large and small businesses and think I can have a stab at what the smaller category should be concerned with.

First lets categorize the audience, this is for a small company < 200 employees with [Seed funding](https://en.wikipedia.org/wiki/Seed_money) or [Series-A funding](https://en.wikipedia.org/wiki/Series_A_round), this can be extrapolated to a comparative non-profit.

Larger than Series-A, you should potentially review your threat-model and invest more money into security.

Things to do (in order):

### 1. Buy these books and read them
[Threat Modeling: Designing for Security](https://www.goodreads.com/book/show/18379732-threat-modeling), this will allow you to think and reason about whether a given risk applies to your business.

[How to Measure Anything in Cybersecurity Risk](https://www.goodreads.com/book/show/26518108-how-to-measure-anything-in-cybersecurity-risk), this will help extend the prior learnings such that you may be able to estimate the risk and therefore make more aggressive tradeoffs in buying some mitigation vs explicitly accepting the risk, both are legitimate strategies.

### 2. Rule out targeted attacks 
Sorry, you're not important enough for these to apply to you, so don't let anyone convince you to buy protections for these cases at this stage:
  - <s><a href='https://en.wikipedia.org/wiki/Targeted_threat'>targeted attacks</a></s>
  - <s><a href='https://en.wikipedia.org/wiki/Insider_threat'>insider threat</a></s>
  - <s><a href='https://en.wikipedia.org/wiki/Zero-day_(computing)'>zero days</a></s>

### 3. Focus on ubiquitous attacks

Ubiquitous attacks are ones that can impact anyone on the internet, and you essentially do not want your business to be in that pool of potential victims if you can help it.

#### [Denial of Service](https://en.wikipedia.org/wiki/Denial-of-service_attack)

There is a concept in security, with a handy initialism "CIA" (Confidentiality, Integrity, Availability) these examples fall strictly under the Availability category.

There is the classic case of some attacker causing downtime to your website, however there is additionally a newer flavour causing downtime to your whole organization called "[Ransomeware](https://en.wikipedia.org/wiki/Ransomware)" - if all your systems get encrypted and you have to pay someone to get back to normal operations that could be an existential risk to your startup.

solutions you may want to try:
- use [chromebooks](https://www.google.com/intl/en_ca/chromebook/) throughout your organization
- purchase a backup solution, preferably one that is robust against ransomeware, e.g. [carbonite](https://www.carbonite.com/)
- put [Cloudflare](https://www.cloudflare.com/) in front of your site. It's cheap, it's extremely easy to setup, do a "set-and-forget" and remove denial-of-service as a concern from your day

#### Accidental exposure of customer data

Actually spend time to put safe-guards in place to prevent your team from accidentally putting customer data in public places. There's not a silver-bullet solution for this, I'm just emphasizing this is a good place to spend your security "time-budget".

### 4. Have a strategy for how your organization does authentication

Use single-sign-on wherever you can, this may be via some identity-provider you purchase (e.g. [onelogin](https://www.onelogin.com/)) or via [OIDC with GSuite](https://gsuite.google.com/learn-more/gsuite-expands-identity-services.html). 

If you must share credentials, purchase a business account with [1Password](https://1password.com/), I'm not being a shill here, I've reviewed the security and cryptography of this product, it's excellent and by far the best in this category. I use it for home and business.

Get your users into the habit of not-even-knowing what the password for something is, just generate it in there and share if needed. Also it will prompt you for things like password breaches and missing two-factor-authentication.

### 4. Don't hire a security person

At this stage this is not a good use of your budget. Instead I'd recommend contracting a consultancy for small periods of time to review and tweak your setups. You do not have to do this if you can't spare the budget.

### 5. Use an online productivity suite

Seriously don't use client side software, it's too much time investment for you at this stage.
Use either:

- [GSuite](https://gsuite.google.ca)
- [Office365](https://www.office.com/)

These also strongly mitigate the phishing risk to your employees, however you may additionally want to purchase some inexpensive, yearly training on phishing for your employees.

_For a fantastic (though 5 years dated) overview on securing GSuite see this [guide](https://blog.trailofbits.com/2015/07/07/how-to-harden-your-google-apps/) from trail-of-bits._


Hopefully some of this was useful to you, happy to expand and evolve this over time based on feedback!
