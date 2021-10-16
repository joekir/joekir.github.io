---
layout: post
title: Creating a Rocky Linux TemplateVM on Qubes 
comments: true
type: post
published: true
status: publish
---

> _Why was this so hard to do...?_

If you've stumbled upon this via Googlin' you'll probably find this helpful, if you're just reading stuff on my blog it probably won't be so interesting to you so skip to something else ;)

Now that CentOS is no longer the workhorse for server linux that it once was ([announcement](https://blog.centos.org/2020/12/future-is-centos-stream/)), the community are probably going to want
to move to RockyLinux as this provides that downstream distro stability that CentOS previous did.

## Steps

1. From the `dom0` terminal emulator use `qvm-create` as I'm not sure exactly how you create a [`TemplateVM`](https://www.qubes-os.org/doc/templates/) using the Qubes Manager GUI:
   `qvm-create --class TemplateVM rocky --property virt_mode=HVM -l gray`
   
   _\* it doesn't have to be gray, just feels like the right colour for a template_
2. Once that is created you can make any changes you wish via the Qubes Manager GUI, specifically one I needed for this install was to bump the initial memory up from 400MB
3. TemplateVMs don't usually have a [`NetworkVM`](https://www.qubes-os.org/doc/networking/) attached so double check you're connected to `sys-firewall` for the install piece
4. In one of your existing QubesVMs go download the bootable ISO that you need from https://rockylinux.org/download
   and move it over to the `/tmp` dir for simplicity
5. back in `dom0` terminal run
   `qvm-start rocky --cdrom=<your other vm name>:/tmp/Rocky-8.4-x86_64-boot.iso`
   assuming you have enough init memory allocated this will eventually get you to the installation GUI for Rocky
6. For reasons tbd the Rocky Linux installer cannot see the bootable media as an installation source
   to figure out what your ethernet configuration needs to be run the following in `dom0` terminal:
   `qvm-prefs rocky`, noting down the `visible_gateway`, `visible_ip`, `visible_netmask`
7. Back in the Rocky installer GUI, go to the network configuration and locate the IPv4 settings for eth0, you'll need to switch
   from DHCP to manual, and plug in those values from step-6, additionally you'll need a DNS provider for the next step to work, so just use something like
   1.1.1.1 (cloudflare), 9.9.9.9 (IBM) or even 8.8.8.8 (Google) for sake of the install only.
8. Now go to the failing setup for source installation and use this URL https://download.rockylinux.org/pub/rocky/8/BaseOS/x86_64/os/ or something similar to that depending on how quickly this guide ages ;)
9. The rest should of the steps in the GUI should now be unblocked and your TemplateVM can be created 🎉

I hope someone stumbles across this and finds it helpful!
