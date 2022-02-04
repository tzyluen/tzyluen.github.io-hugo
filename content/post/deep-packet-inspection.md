+++
draft = false
description = ""
topics = []
tags = ["infosec", "networking", "packet-analysis", "deep-packet-inspection", "ndpi", "iptables"]
date = "2017-03-24T00:00:00+08:00"
title = "Deep packet inspection"
toc = true

+++

{{< toc >}}



---
## Introduction

Port-independent, P2P, and encrypted protocols and packets have made the conventional network traffics analysis that based on packet header (transport protocol and application ports) obsolete. Deep Packet Inspection (DPI) technology can be used to identify and classify these encrypted, port-independent, P2P protocols. 

This post explore the popular open source implementations:

* nDPI library [1]
* ndpi-netfilter [2]

Other similar open source DPI tools are:

* L7-filter [3], open source, GPLv2.
* Libprotoident [4], open source, LGPL.
* PACE (Protocol and Application Classification Engine) [5], commercial.
* NBAR (Network Based Application Recognition) by Cisco [6], commercial.

Note: ndpi-netfilter tested only on Ubuntu 14.04.1 LTS (kernel 3.13.0-37-generic). During the initial experiment of this setup on machine with kernel v4.x, it did not work. The following Debian/wheezy machine with kernel v3.2 were used instead:

OS/kernel version:

```
tzy@192.168.1.12:~$ uname -a
Linux blackstar 3.2.0-4-amd64 #1 SMP Debian 3.2.63-2+deb7u2 x86_64 GNU/Linux
```



---
## nDPI
### Build and install ndpi-netfilter

1. Download `ndpi-netfilter` from https://github.com/betolj/ndpi-netfilter and extract to `/usr/src`.
2. Install dependencies:

    ```
    root@192.168.1.12:~# apt-get install linux-source
    root@192.168.1.12:~# apt-get install libtool
    root@192.168.1.12:~# apt-get install autoconf
    root@192.168.1.12:~# apt-get install pkg-config
    root@192.168.1.12:~# apt-get install subversion
    root@192.168.1.12:~# apt-get install libpcap-dev 
    root@192.168.1.12:~# apt-get install iptables-dev 
    ```

3. Build and install nDPI:

    ```
    root@192.168.1.12:~# cd /usr/src/ndpi-netfilter-master
    root@192.168.1.12:/usr/src/ndpi-netfilter-master# tar xvzf nDPI.tar.gz
    root@192.168.1.12:/usr/src/ndpi-netfilter-master# cd nDPI
    root@192.168.1.12:/usr/src/ndpi-netfilter-master/nDPI# ./autogen.sh
    root@192.168.1.12:/usr/src/ndpi-netfilter-master/nDPI# make
    root@192.168.1.12:/usr/src/ndpi-netfilter-master/nDPI# make install
    ```

4. Build and install ndpi-netfilter:

    ```
    root@192.168.1.12:~# cd ..
    root@192.168.1.12:~# NDPI_PATH=/usr/src/ndpi-netfilter-master/nDPI make
    root@192.168.1.12:~# make modules_install
    root@192.168.1.12:~# cp /usr/src/ndpi-netfilter-master/ipt/libxt_ndpi.so /lib/xtables/
    ```



---
## Realtime capture

Use the `ndpiReader` utility to capture and classify network packets in realtime.

1. Runs SSH server `192.168.1.12` on port `26/tcp` instead of the default `22/tcp`:

    {{< fluid_img "/img/ndpi-deep-packet-inspection-03.png" >}}

2. From a client host `192.168.1.9`, port scans `192.168.1.12`, it shows as `26/tcp rsftp` service:

    {{< fluid_img "/img/ndpi-deep-packet-inspection-04.png" >}}

3. SSH on non-standard port 26, and non-port based packets e.g., Facebook, Twitter, GMail, Youtube, Google, Apple, Cloudflare, etc., were detected.

    {{< fluid_img "/img/ndpi-deep-packet-inspection-02.png" >}}



---
## Iptables/netfilter

To filter port-independent packets, get the list of supported protocols as shown below. Below are some popular protocols supported by ndpi:

```
root@192.168.1.12:~# iptables -m ndpi -h
...
ndpi match options:
--ssh Match for SSH protocol packets.
--gnutella Match for GNUTELLA protocol packets.
--edonkey Match for EDONKEY protocol packets.
--facebook Match for FACEBOOK protocol packets.
--twitter Match for TWITTER protocol packets.
--webm Match for WEBM protocol packets.
--quic Match for QUIC protocol packets.
--whatsapp Match for WHATSAPP protocol packets.
--whatsapp_voice Match for WHATSAPP_VOICE protocol packets.
--snapchat Match for SNAPCHAT protocol packets.
--youtube Match for YOUTUBE protocol packets.
...
```


### SSH

To drop all incoming ssh packets, do:

```
root@192.168.1.12:~# iptables -I INPUT -m ndpi --ssh -j DROP

iptables -A INPUT -m limit --limit 2/min -j LOG --log-prefix "[IPTABLES-INPUT-DROPPED] " --log-level 4
iptables -A OUTPUT -m limit --limit 2/min -j LOG --log-prefix "[IPTABLES-OUTPUT-DROPPED] " --log-level 4
```

Note: the last two lines are for logging purposes.


### Youtube

To drop all incoming youtube based packets, do:

```
iptables -A INPUT -m ndpi --youtube -j DROP
```

### Social Networks

To drop all incoming and outgoing social networks, do:

```
iptables -A INPUT -m ndpi --twitter -j DROP
iptables -A OUTPUT -m ndpi --twitter -j DROP
iptables -A INPUT -m ndpi --facebook -j DROP
iptables -A OUTPUT -m ndpi --facebook -j DROP
```


---
**References:**
1. nDPI - http://www.ntop.org/products/deep-packet-inspection/ndpi/
2. ndpi-netfilter - https://github.com/betolj/ndpi-netfilter/README
3. L7-filter - http://l7-filter.clearos.com
4. Libprotoident - https://research.wand.net.nz/software/libprotoident.php
5. PACE (Protocol and Application Classification Engine) - https://www.ipoque.com/products/dpi-engine-rsrpace-2
6. NBAR (Network Based Application Recognition) - http://www.cisco.com/c/en/us/products/ios-nx-os-software/network-based-application-recognition-nbar/index.html
