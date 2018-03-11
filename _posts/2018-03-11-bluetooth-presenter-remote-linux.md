---
layout: post
title: Configuring bluetooth remote pointer on Linux 
comments: true
type: post
published: true
status: publish
---

TLDR;
```
# udevadm hwdb --update
```

I recently purchase the [Satechi Wireless Presenter](https://satechi.net/products/satechi-aluminum-wireless-presenter) to make it easier for some upcoming talks.

It seemed pretty cool because it was simple (left,right buttons) with an additional laser pointer and it's rechargeable.

To my surprise, it didn't immediately work with my Ubuntu 16.04 work laptop.

### Diagnosing the issue

```
$ bluetoothctl
[NEW] Controller F4:D8:51:26:F5:98 laptop [default]
[NEW] Device 20:9B:A6:67:D5:DA SoundBuds Slim
[NEW] Device 20:79:01:1A:D8:16 ST-APA
[bluetooth]# info 20:79:01:1A:D8:16 
Device 20:79:01:1A:D8:16
 Name: ST-APA
 Alias: ST-APA
 Class: 0x000540
 Icon: input-keyboard
 Paired: no
 Trusted: no
 Blocked: no
 Connected: no
 LegacyPairing: no
 ...
 ...
```

Following the obvious steps to `pair`, `trust` and `connect` with bluetoothctl it's easy to get it to the stage where the device connection looks good.

To test that you're actually receiving signals the first debug tool I tried was `btmon`   
One of the immediate errors I encountered was 

```
* Unknown packet (code 17 len 32)                              [hci0]
```

But if I pressed the left and right buttons, I could see that the packets were at least coming through

```
> ACL Data RX: Handle 256 flags 0x02 dlen 14                   [hci0]
      Channel: xx len 10 [PSM xx mode 0] {chan 1}
        xx xx xx xx 50 xx xx xx xx xx                    ....P.....
> ACL Data RX: Handle 256 flags 0x02 dlen 14                   [hci0]
      Channel: xx len 10 [PSM xx mode 0] {chan 1}
        xx xx xx xx 4f xx xx xx xx xx                    ....O.....
```

_To highlight, I've ommited everything other than the 2 chars that changed for the button presses_

After this I wanted to see whether there was an issue with how those inputs were being mapped.
So I then tried `evtest`

```
# evtest --grab /dev/input/event18

...
...
loads of stuff
...
...

Event: time 1520782687.296605, -------------- SYN_REPORT ------------
Event: time 1520782687.319079, type 4 (EV_MSC), code 4 (MSC_SCAN), value 7004f
Event: time 1520782687.319079, type 1 (EV_KEY), code 106 (KEY_RIGHT), value 0
Event: time 1520782687.319079, -------------- SYN_REPORT ------------
Event: time 1520782688.309012, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70050
Event: time 1520782688.309012, type 1 (EV_KEY), code 105 (KEY_LEFT), value 1
```

Clearly we can now see that the key left and right are coming through fine.
My next idea was to look at what `udev` rules I had.

Turns out I had none in `/etc/udev/hwdb.d` 

I read a couple of articles, then stumbled across this:

```
# udevadm hwdb --update
```

upon doing that this binary blob was created `/etc/udev/hwdb.bin` and suddenly the presenter remote worked perfectly. Perhaps Satechi could include that 1-liner in their documentation? 😉 
