---
layout: post
title: Creating a hardware torbox using a RaspberryPi Zero
comments: true
type: post
published: true
status: publish
---

I liked the concept the grugq had with [PORTALofPi](https://github.com/grugq/PORTALofPi) and I guess you could use that for this, however the most useful steps in this guide are about how to go about connecting and routing traffic through it.

_all the instructions below assume you're on a Linux distro, if you're on Mac, that sucks for you, I'm sorry_ 

## Steps

_you'll need a [Raspberry-Pi Zero](https://www.raspberrypi.org/products/raspberry-pi-zero/) a really nice thing about this is you can power it and connect to it's ethernet just by plugging it into your laptop, no extra cables or power suppply needed!_

### Setting up the pi

1. Burn the [raspbian-lite image](https://www.raspberrypi.org/downloads/raspbian/) onto a microSD card
2. Edit the file `boot/config.txt` adding this line to the end of the file:    
   ```
   dtoverlay=dwc2
   ```
3. In the file `boot/cmdline.txt` add the following after the **rootwait** command:    
   ```
   modules-load=dwc2,g_ether
   ```
4. Raspbian recently changed ssh to not be enabled by default so `touch boot/ssh` to enable this
5. Add the SD card to the pi, plug in the cable to the **usb** port, **not** the power port, and connect to your machine

### Connecting from the host

6. On your host, create a wired network connection, disable ipv6, and set ipv4 to be `link-local` only
7. Once the pi has booted your should be able to run an `arp -a` to see it's ip address, if you don't see it for whatever reason, try the next step just in case
8. Test the connection with `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no pi@raspberrypi.local` password is `raspberry` (all the options are just to stop your keychain throwing a load of keys at it)
8. Once you're on the box, setup the wifi using `sudo raspi-config`, enjoying the ncurses-looking menu ([whiptail](https://en.wikibooks.org/wiki/Bash_Shell_Scripting/Whiptail))
9. Install tor `sudo apt update && sudo apt install tor`, note on next boot this service will autorun a SOCKS5 on localhost:9050
10. Exit that connection, and now setup a background port mapping, via:    
    ```
    ssh -fqCN -L 5000:localhost:9050 -o PreferredAuthentications=password -o PubkeyAuthentication=no pi@raspberrypi.local
    ```
11. Configure your system's network proxy (or browser local proxy) SOCKS5 to go through `localhost:5000`
12. Be sure to turn off the wifi connection on your host machine for extra assurance that all traffic is going through the tor-pi

Go to a site like [ip.team-cymru.org](https://ip.team-cymru.org) (sad that the TLS cert expired there..), you'll notice firstly that it says `Type: Darknet` but also that you're somewhere random in the world, each time you refresh you'll travel somewhere else, see if you can hit all 7 continents :)

### References

- https://learn.adafruit.com/turning-your-raspberry-pi-zero-into-a-usb-gadget/ethernet-gadget
- https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0
- https://opensourceforu.com/2016/11/make-tor-proxy-router-raspberry-pi/
