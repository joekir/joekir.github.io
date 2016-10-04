---
layout: post
title: "[How To] Enabling chromium distiller reader mode"
comments: true
type: post
published: true
status: publish
---

Since firefox version 16-ish they've had a [reader mode](https://support.mozilla.org/en-US/kb/firefox-reader-view-clutter-free-web-pages) available for webpages. I really like it as it lets you read the content without all the blinky-light-clutter. As Google are advertising people, while they do have the ability to simplify the DOM, they don't make [the option](https://github.com/chromium/dom-distiller) the setting easily available for obvious reasons!

Here's how to enable it in chrome/chromium.

the `/user/bin/chromium` startup script notes:

```sh 
# Some rudimentary support for user flags is provided via a chromium-flags.conf
# config file placed in $HOME/.config/ (or $XDG_CONFIG_HOME). Arguments are
# split on whitespace and shell quoting rules apply but no further parsing is
# performed. In case of improper quoting anywhere in the file, a fatal error is
# raised. Lines starting with a hash symbol (#) are skipped.
```
Therefore to enable the "Distill page" option from your chromium dropdown menu. 

{% gist b8732fcb673613cd03d1ee25edec94c8 %}

