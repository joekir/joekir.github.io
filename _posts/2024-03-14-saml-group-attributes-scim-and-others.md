---
layout: post
title: SAML group attributes, SCIM and shortcomings 
---

The goals are to have an article to point to, to illustrate to people when SAML (Security Assertion Markup Language) group attributes are sufficient and when SCIM (System for Cross-domain Identity Management)  is also required to achieve security needs.

### Intro

Most service providers are now supporting SSO (Single Sign-On). This is great to see as managing onboarding and offboarding in an enterprise context is very challenging without it.

However, this only covers <u>auth</u>e<u>n</u>tication, if you want to do anything with <u>auth</u>ori<u>z</u>ation you need to do one of the following:

1. Define permissions for a given identity in the Service Provider (SP), this obviously does not scale well for centralized administration of an enterprise.
2. Pass group/permission details as attributes in the [SAML AuthN response](https://developers.onelogin.com/saml/examples/response). I will discuss this here and what scenarios it is fine for and what it is not.
3. Use something such as SCIM to synchronize groups and permissions between your centralized administration and the service-provider. I will cover what scenarios this uniquely helps with, and also some pitfalls with lack of specification definition.

### SAML AuthN Response attributes

**WARNING**, there is no such thing as "SAML Group Attributes"[^1] in the specification. Essentially you can pass attributes, but need the Identity Provider (IdP) and SP to know what that format looks like ahead of time, below is an illustration of what you might receive in the SAML AuthN Response as the SP:

```xml
<saml2:Attribute Name="groups" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
    <saml2:AttributeValue>Developer</saml2:AttributeValue>
    <saml2:AttributeValue>Billing-Admin</saml2:AttributeValue>
</saml2:Attribute>
```
<br/>
This can work in a scenario where you’re only doing user auth management in the UI, i.e they’re the agent that initiates contact with the SP and the SP does not allow for any delegate auth to represent them in a machine-to-machine context, such as an API key.

If you are fortunate enough to have a scenario supporting this there are other challenges with this “ad-hoc” approach in that you don’t know what permissions/roles/groups are available in the SP without some other API support. For security in the ideal case the SP fails-closed and if nothing maps correctly the user gets some base level authZ and not admin! But this is up to the SP and not defined as a requirement in SAML itself.

### SCIM

First thing to remember is SCIM[^2] is a push-based protocol. There are some providers (check this) that provide an API to check for the events instead, however there are no guarantees that the data is up-to-date. The key thing to remember with SCIM is it’s not easy to discern the difference between “no user attributes have changed” and “not receiving updates correctly for the user”, there is no concept of a “heartbeat”. The “freshness” property does seem like a large security hole that should have been a focal point of the protocol design.


<!-- https://mermaid-js.github.io/mermaid/#/./sequenceDiagram -->
<div class="mermaid">
sequenceDiagram
    title SCIM "freshness" issues
    participant idp as Identity-Provider
    participant sp as Service-Provider

    autonumber
    
    idp ->> sp: Push updates for user:0139
    sp -->> sp: update mappings user:0139 entity with changes
    idp -->> idp: admin changes group attributes on user:0139
    Note over idp,sp: many days pass ...
    Note right of sp: sp: we don't seem to have any updates on this user:0139, do we leave as is, deactivate?
</div>

I think having a heartbeat/freshness policy in SCIM would be ideal, whereby the IdP could say "if you don't hear from me in by this time, do this with the user", then there is no ambiguity on the sync delays.

There are a number of other variances between SCIM providers in how they request a user entity to be deleted, ranging from "please deactivate but retain the data" (soft-delete) to "remove everything" (hard-delete), to unspecified behavior in the middle. 

#### API Keys

Here is something worth calling out with these two approaches. If you want to have the concept of an API key that is mapped to a user's capabilities as defined in the IdP you absolutely must use SCIM! Otherwise if the user infrequently logs into the SP to propagate the group attributes to you the capabilities of the API key could become dangerously stale over time. You will still have the lack of heartbeat issues, but at least you won't be utterly decoupled.

### A New Hope? 

A more modern approach appears to be coming from the OpenID [Shared Signals Working Group](https://openid.net/wg/sharedsignals/) through something called "Continual Access Evaluation Profile"[^3], which appears to have a primary focus around AuthN in the SP, but it could have potentials to also address stale AuthZ as discussed in this post.
<br/>
<br/>

---

### Footnotes

[^1]: For the complete list of SAML attrname-format types see section 8.3 of [http://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf](http://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf)
[^2]: Okta has a better overview than the [official](https://scim.cloud/) SCIM site: [https://developer.okta.com/docs/concepts/scim/](https://developer.okta.com/docs/concepts/scim/)
[^3]: [https://openid.net/specs/openid-caep-specification-1_0.html](https://openid.net/specs/openid-caep-specification-1_0.html)