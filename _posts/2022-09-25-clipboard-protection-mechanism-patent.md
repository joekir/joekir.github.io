---
layout: post
published: true
status: publish
title: Explaining how Android Secure-Copy-Paste (US10754929B2) works
type: post
---

Back in the days of 2016 I worked for BlackBerry, they don't make phones anymore but they were doing Android phones back then!.
A security issue that I was **very** unhappy about in [AOSP](https://source.android.com/) was the security of data transferred across applications via the [Global Clipboard](https://developer.android.com/develop/ui/views/touch-and-input/copy-paste).

To solve this I provided a design to our development teams that leveraged something called a ContentProvider and an implicit feature of Android copy/paste called [`CoerceToText`](https://developer.android.com/reference/android/content/ClipData.Item#coerceToText(android.content.Context)). This design was later patented in [US10754929B2](https://patents.google.com/patent/US10754929B2) / [EP3208713B1](https://patents.google.com/patent/EP3208713B1) which is a bit sad, and I hope it doesn't limit someone from using this approach today 🤞.

I think this design has some modern relevance since in iOS 16 Apple have chosen to introduce a [`Pasteboard permission`](https://www.apple.com/ca/ios/ios-16/features/) to address the security issue in a different way, whereby if a target application wants to paste, a user is asked every time if they should `allow` or `disallow` ([UIPasteControl](https://developer.apple.com/documentation/uikit/uipastecontrol?changes=_5). Whether this stays as is currently, whereby the user of the device needs to confirm access for every-single paste or they allowlist the app for a target app forever, neither of these are as good for UX or security as the approach described here. I do wonder if Apple considered the inverted approach and why they did not want to go with that 🤔. 

## Overview

### Key Components

<!-- https://mermaid-js.github.io/mermaid/#/flowchart -->
<div class="mermaid">
flowchart LR
    classDef vulnerable fill:#f1ac56
    classDef critical fill:#ffcccb
    classDef safe fill:#daf7a6
    classDef resetColor fill:#ffffff

    subgraph Device
        subgraph sourceApp[Source App]
            contentProvider[Content Provider]
            click contentProvider href "https://developer.android.com/reference/android/content/ContentProvider"
        end
        
        subgraph clipboard[Global Clipboard]
        end
        
        subgraph targetApp[Target App]
            contentResolver[Content Resolver]
            click contentResolver href "https://developer.android.com/reference/android/content/ContentResolver"
        end
    end

    class clipboard critical
    class sourceApp,targetApp safe
</div>

### Sharing Sequence Diagrams

_N.B I chose not to overlay this on the diagram above as there are too many dimensions (e.g. time) to display it nice and simplicistic_

<!-- https://mermaid-js.github.io/mermaid/#/./sequenceDiagram -->
<div class="mermaid">
sequenceDiagram
    title Normal Android Copy/Paste
    participant source as Source App
    participant clipboard as Global Clipboard
    participant target as Target App
    participant other as Malicious App

    autonumber
    
    source ->> clipboard: PUT text "my secretz"
    par
        clipboard ->> target: GET text "my secretz"
        clipboard ->> other: GET text "my secretz"
    end
</div>

<!-- https://mermaid-js.github.io/mermaid/#/./sequenceDiagram -->
<div class="mermaid">
sequenceDiagram
    title Android with Secure Copy/Paste
    participant source as Source App
    participant sourceCP as Content Provider
    participant clipboard as Global Clipboard
    participant target as Target App
    participant targetCR as Content Resolver

    autonumber

    source -->> sourceCP: User selects which app(s) they want to copy to
    sourceCP -->> sourceCP: Stores some text in its database, and generates a unique "Content URL" for it
    sourceCP -->> sourceCP: Looks up the unique ID's of the apps within the device and adds that metadata to the content
    source ->> clipboard: PUT text "content://sourceapp.foo.biz/abc123"
    clipboard ->> target: GET text "content://sourceapp.foo.biz/abc123"
    target -->> targetCR: do we know what this URL is?
    targetCR ->> sourceCP: Can I resolve the URL?
    sourceCP ->> sourceCP: lookup ID of application received by IBinder IPC connection
    alt if in allowlist
        sourceCP ->> targetCR: return "my secretz"
    else if not in allowlist
        sourceCP ->> targetCR: return "content://sourceapp.foo.biz/abc123"
        Note right of sourceCP: this is what an unauthorized app would see
    end
</div>

## Wrap up

It would be nice to see some end to clipboard-stealing malware, one of the earlier articles I was aware of for Android [was in 2014](https://arstechnica.com/information-technology/2014/11/using-a-password-manager-on-android-it-may-be-wide-open-to-sniffing-attacks/) and for the most part it's still here today almost 10 years on.

Personally I don't like either the UX or the security of Apple's approach in iOS 16 as it is:

1. More frictionful as you need to approve the target app for every single paste (or if too many users complain about this it will be made less secure with some time-bound or forever-bound allowlist that defeats the point of the feature).
2. More susceptible to social engineering, the "victim" is already interacting with the malicious app, and hence it would be easy for said malicious app to convince a user to just breeze through some "allow this" checkbox without really comprehending the impact of it. The approach laid out above let's the user make that decision at time-of-copy when their context on the content is more fresh and they know (in 99% of cases) which app(s) it should be going to.
3. There is a [confused deputy](https://en.m.wikipedia.org/wiki/Confused_deputy_problem) opportunity, as the allow/deny UI that is presented to the user does not make it clear **what** the current clipboard content is or **where** it is coming from hence they may inadvertently grant access to a stale piece of sensitive data.

If Apple can use the above design without some patent issues that would be dope, and I'd love to help in anyway if someone from there stumbles upon this :)
