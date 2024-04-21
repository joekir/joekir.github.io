---
layout: post
title: Modern Authorization (aka ReBAC) primer
---

I somewhat freestyled this writeup originally to help developers at work with some concepts used in authorization frameworks.    
But, upon reflection I thought a genericized version of this could be generally useful to anyone diving into technologies that use ReBAC or PARC (what are those? find out below!).

## Types of Access Control

**Discretionary Access Control (DAC)**: It's likely the majority of data protection systems you have encountered are DAC. This simply means that if you have access to something, you can grant access (to that object) to someone else without approval from anyone else.

**Mandatory Access Control (MAC)**: A more strict form of access control than DAC, in this scenario just because you can do something, does not mean you can allow some other principal to do something.

In practice these are rarely preferred as while more secure, they're resource intensive on the policy author and approver. [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux) is probably the most well known example of a MAC framework (for the Linux kernel), if you're interested to learn about it, check out my companion site SELinux Userland ([selinuxuser.land](https://selinuxuser.land/)).

**Role Based Access Control (RBAC)**: The things you can do in a system are based on the current role you have in an organization. Poorly (from a security/safety perspective) implemented forms of this allow for you to be in multiple roles at a given time (non-exclusive) however more sound implementations (i.e. AWS role toggling) will: 1) only allow you to be the state of one given role at any one time 2) help you in designing what the division of privileges should be such that the mutual-exclusion is optimizing the safety factor of your system. To learn more on original goals of RBAC [this early NIST paper](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=916538) is recommended, as it covers mutual-exclusion condition and its value.

**Attribute Based Access Control (ABAC)**: This is where the attributes/metadata associated with a Principal, Action or Resource impact whether it is allowed/disallowed within a system. Note, RBAC could be composed on top of an ABAC system, but not the other way around.     
Recommended reading:
* [NIST SP800-162](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-162.pdf)
* [AWS IAM - What is ABAC for AWS?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html)

**Relationship Based Access Control (ReBAC)**: The PARC interface is covered below, essentially that is ReBAC where the allowance/denial is expressed as a relationship in a (directed-acylic) graph connecting principals to resources via actions. For a more comprehensive comparison of Policy Based Access Control (examples [OPA](https://www.openpolicyagent.org/), [AWS-AVP](https://docs.aws.amazon.com/verifiedpermissions/latest/userguide/what-is-avp.html)/[Cedar](https://www.cedarpolicy.com/), Hashicorp [Sentinel](https://www.hashicorp.com/sentinel)) and ReBAC read [this article](https://authzed.com/blog/policy-based-access-control) from AuthZed.

## ABAC (and ReBAC) specific terminology

![NIST ABAC Diagram](/assets/xacml_abac.png)

**Policy Decision Point (PDP)**: These terms all originate from [OASIS XACML](https://en.wikipedia.org/wiki/XACML), PDP is the boolean decision engine on whether something should/should-not be allowed. See [policy decision point (PDP) - Glossary \| NIST CSRC](https://csrc.nist.gov/glossary/term/policy_decision_point).

**Policy Enforcement Point (PEP)**: Also from OASIS XACML and is usually a proxy-layer that either allows an action to occur or not. See [policy enforcement point (PEP) - Glossary \| NIST CSRC](https://csrc.nist.gov/glossary/term/policy_enforcement_point).

**Policy Administration Point (PAP)**: In relation to the above, this is straightforward, it's just how the policies are managed. See [Policy Administration Point - Glossary \| NIST CSRC](https://csrc.nist.gov/glossary/term/policy_administration_point).

**Policy Information Point (PIP)**: Simply the information backing store for the PDP, however this is just one piece of the information, the context that comes from the PEP i.e. request context, is also something the PDP will consider. See [Policy Information Point - Glossary \| NIST CSRC](https://csrc.nist.gov/glossary/term/policy_information_point).

**PARC: Principal, Action, Resource, Condition**: This is one of the most popular and successful ways to express what outcome you would like a permissions engine to evaluate. It is what is used in AWS [IAM](https://docs.aws.amazon.com/serverless/latest/devguide/starter-iam.html#iam_fundamentals) and Google Zanzibar ([this annotated version](https://zanzibar.tech/) of the paper from AuthZed is excellent IMO) amongst many others.