+++
draft = false
date = "2016-02-20T00:00:00+08:00"
title = "Sniff DNS Queries"
description = "dnstop / tcpdump / wireshark"
topics = []
tags = ["infosec", "networking", "dns", "tcpdump", "wireshark", "tshark", "packet-analysis", "sniffing"]
toc = true

+++

{{< toc >}}

---
## Dnstop
is a console libpcap application that displays various tables of DNS traffic on a network including:

* Source IP addresses
* Destination IP addresses
* Query types
* Top level domains
* Second level domains

```
# dnstop enp0s3 -l 3
```

{{< fluid_img "/img/dnstop-monitoring-large-count-01.png" >}}

Use `ctrl-r` to reset the counter/refresh the history to get the latest queries.


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## Tcpdump

Capture packets from port 53 (DNS):
```
-i: interface
-v: verbose
-s: snaplen, snap n bytes length from each packet (default 262144).
    setting lowest n to prevent packet lost.
-n: do not convert addresses/ports to names.
-w: write the raw packets to file.

# tcpdump -i eth0 -v -s0 port 53 -w /tmp/dns-queries.pcap
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
Got 223
```

Read from captured raw pcap file:
```
-q: quick/quiet/shorter output. Less protocol information.
-A: print the payload in ASCII.
-X: print the payload in hex and ASCII.
-tttt: print a human readable timestamp, preceded by the date, on each dump line.
-r: read packets from file.

# tcpdump -ns 0 -A -tttt -r /tmp/dns-queries.pcap

2016-04-03 19:15:28.673885 IP 192.168.1.7.50906 > 192.168.1.1.53: 25364+ A? www.reddit.com. (32)
E..<......l............5.(.|c............www.reddit.com.....

2016-04-03 19:15:28.737323 IP 192.168.1.1.53 > 192.168.1.7.50906: 25364 5/0/0 CNAME reddit.map.fastly.net., A 151.101.1.140, A 151.101.65.140, A 151.101.129.140, A 151.101.193.140 (131)
E.....@.@............5.....%c............www.reddit.com.............. ...reddit.map.fastly.net..,...........e...,...........eA..,...........e...,...........e..
```


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## Wireshark / Tshark
The console version of Wireshark comes handy to quickly monitor the traffic on port 53. It works the same as the capture and display filters from Wireshark.
```
-f: capture filter
-Y: display filter

$ tshark -i enp0s3 -f "udp port 53" -Y "dns.qry.type == 1 and dns.flags.response == 0"
Capturing on 'enp0s3'
    1 0.000000000  192.168.1.7 → 192.168.1.1  DNS 72 Standard query 0xcb2a A slashdot.org
...
   18 3.274217622  192.168.1.7 → 192.168.1.1  DNS 70 Standard query 0x1c9f A reddit.com
   21 3.367215915  192.168.1.7 → 192.168.1.1  DNS 74 Standard query 0x931d A www.reddit.com
...
   51 8.045870811  192.168.1.7 → 192.168.1.1  DNS 71 Standard query 0xf2b0 A youtube.com
   52 8.057876881  192.168.1.7 → 192.168.1.1  DNS 83 Standard query 0x5358 A youtube-ui.l.google.com
...
   65 10.438546212  192.168.1.7 → 192.168.1.1  DNS 72 Standard query 0xfab8 A facebook.com
   67 10.479990482  192.168.1.7 → 192.168.1.1  DNS 74 Standard query 0x7b3d A www.google.com
```

The '`dns.qry.type == A`' will throws '`tshark: "A" cannot be found among the possible values for dns.qry.type.`'. The decimal value for a type 'A' DNS query type is 1 [1].

---
**References:**
1. DNS Type ID in decimal - [https://en.wikipedia.org/wiki/List_of_DNS_record_types](https://en.wikipedia.org/wiki/List_of_DNS_record_types)
