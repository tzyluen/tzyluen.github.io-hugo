+++
topics = []
tags = ["infosec", "networking", "sniffing", "macos", "wlan", "wireshark"]
draft = false
date = "2016-02-13T00:00:00+08:00"
title = "Capture Network Traffic on WLAN (macOS)"
description = ""
toc = true

+++

{{< toc >}}

## MacOS's Airport
is a built-in wireless utility comes preinstalled in MacOS.

```
$ ll /usr/local/bin/airport 
lrwxr-xr-x  1 root  wheel  89 Jun 22  2016 /usr/local/bin/airport@ -> /System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport
```

Perform a wireless broadcast scan to get the list of access points in the neighborhood:

{{< fluid_img "/img/airport-wireless-broadcast-scan-01.png" >}}

Sniff 802.11 frames on channel 1, the output is in pcap format and can be opened with tcpdump/wireshark:

```
$ airport en0 sniff 1
Capturing 802.11 frames on en0.
^CSession saved to /tmp/airportSniffwUKb60.cap.
```


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml>}}


---
## Wireshark
can also be used to capture wireless network traffic. To do that, open the wireshark's capture option and select the Promiscuous and Monitor checkbox, optionally add capture filter `'not broadcast and not multicast'` to filter out noises:

{{< fluid_img "/img/wlan-wireshark-capture-options-01.png" >}}

Since the wireless traffic is encrypted with WPA2, it won't be useful unless decrypted. To enable decryption on-the-fly, navigate to Preferences &rarr; Protocols &rarr; IEEE 802.11, and select Enable decryption and add the decryption keys.

Depends on the type of encryption used, the following lists the WEP, WPA-PWD and WPA-PSK format:

* `wep:a1:b2:c3:d4:e5`
* `wpa-pwd:myPassword:mySSID`
* `wpa-psk:0102030405060708091011...6061626364`

To generate the WPA-PSK of your wireless credentials, use the [WPA-PSK Generator](https://www.wireshark.org/tools/wpa-psk.html)[1] provided by Wireshark.

In order to capture the handshake for a machine, force the machine to (re-)join the network while the capture is in progress. Repeat this for all machines whose traffic you want to see. Figure below shows the captured/decrypted DNS traffic.

{{< fluid_img "/img/wlan-wireshark-sniff-dns-decrypted-01.png" >}}


---
**References:**
1. https://www.wireshark.org/tools/wpa-psk.html
