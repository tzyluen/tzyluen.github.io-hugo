+++
title = "Traceroute, Firewalls & Geo-IP"
description = "Reading Traceroute"
topics = []
tags = ["infosec", "networking", "hping3", "nmap", "traceroute", "geo-ip", "scanning"]
draft = false
date = "2016-01-24T00:00:00+08:00"
toc = true

+++

{{< toc >}}

## Traceroute
`Traceroute` is useful for diagnosing networking problems, e.g., end-to-end connectivty, complement with `ping`. It can also be used to pinpoint the location of devices, routers and firewalls. The tracerouting tools fundamentally rely on the IP packet's field - TTL (Time-To-Live, decremented at each hop, dies at 0), they send short-life IP packets and wait for Time Exceeded ICMP packets reporting the death of these packets from a router, consequently reveal the route.

```
Mode:
-I, --icmp      Use ICMP ECHO for tracerouting
-T, --tcp       Use TCP SYN for tracerouting (default port is 80)
-U, --udp       Use UDP to particular port for tracerouting
                (instead of increasing the port per each probe), default port is 53
```

### ICMP mode
Default `traceroute` uses `ICMP ECHO` packets.

```
$ traceroute <target>
```

### UDP mode

Default dest port is 53 (DNS).
```
$ traceroute -U <target>
```

For security reasons, the default traceroute's UDP packets and ICMP Echo packets are often blocked. To evade firewalls, the following techniques can be used.

### TCP mode

`tcptraceroute` or ` -T` of the `traceroute` uses TCP SYN packets (SYN packet is the first step TCP three-way handshake), which usually not blocked by firewalls, and as long as the destination port is opened. To specify the dest port, uses -p 80 (HTTP) or 443 (HTTPS), that normally allowed to egress for probes. 

```
$ tcptraceroute <target>
$ sudo traceroute -T <target>

# -w: wait 10 secs before timeout
# -q: set the number of probe packets per hop, default is 3, hence RTT1, RTT2, RTT3
$ traceroute -w 10 -q 3 <target>
```

{{< fluid_img src="/img/plain-traceroute-my.png" >}}

With ICMP Echo request, the packet was blocked after 58.27.14.58. With TCP SYN handshake, it passed two more hops.

{{< fluid_img src="/img/tcp-traceroute-my.png" >}}


### Output format explanation {#output-format-explanation}

```
           v--- the router/ip-addr traversed by the packet 
[Hop]     [Hostname/(IP-addr)]      [RTT1]  [RTT2]  [RTT3]
 ^--- transit no. of the route       ^---- round-trip time
```

The round-trip time is the latency (delay between sending the packet and getting the response).

By default, traceroute sends 3 packets per TTL increment. Each column [RTT1]...[RTT3] corresponds to the time it took to get response (round-trip time). 3 different packets give a better sampling of the latency, it also helps for situation where multi-path exist (different link). For instance, the packet is routed to different link in hop 2:

```
2  175.137.109.62 (175.137.109.62)  38.360 ms       [RTT1]
   175.137.109.70 (175.137.109.70)  38.219 ms       [RTT2]
   175.137.109.62 (175.137.109.62)  38.123 ms       [RTT3]
```

Another common scenario is timeout/packet dropped. For instance, 2 out of 3 traceroute packets were dropped/timeout in hop 4:

```
4  10.55.32.88 (10.55.32.88)   113.878 ms * *
```


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## Hping3

The `hping3` feature equivalent of `traceroute` with ICMP protocol `-1 --icmp`:

```
-V: verbose
-1: icmp

# hping3 --traceroute -V -1 <target>
```


Note: TCP, UDP (`-2 --udp`) mode can be used to traceroute on port 80, 443, 53, both are useful to identify where the packet get blocked:

```
-S: set SYN flag
-p: port

# hping3 --traceroute -S -p 443 <target>
```

`-p 80`
{{< fluid_img "/img/hping3-traceroute-syn-port80-my.png" >}}

`-p 443`
{{< fluid_img "/img/hping3-traceroute-syn-port443-my.png" >}}

`-p 53`
{{< fluid_img "/img/hping3-traceroute-syn-port53-my.png" >}}


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## InTrace

Traceroute-like enumerates IP hops by exploiting existing TCP connections.

```
# intrace -i eth0 -h <target>
```

Establish a TCP connection to port 80:

{{< fluid_img "/img/intrace-ncat-usm-my-port80.png" >}}

And it's capable to identified the target host is behind a NAT:

{{< fluid_img "/img/intrace-usm-my.png" >}}

Traceroute to remotely initiated connections:

{{< fluid_img "/img/intrace-netstat-tanp-local.png" >}}
{{< fluid_img "/img/intrace-remote-initiated-connection.png" >}}


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## Nmap: traceroute-geolocation script

is a tool to pinpoint the nodes and traverse the network path with geo location. Nmap `traceroute-geolocation.nse` supports geolocation, it lists the geographic locations of each hop and output the results to KML format plottable on Google Maps.


```
# nmap --traceroute --script traceroute-geolocation.nse --script-args 'traceroute-geolocation.kmlfile=<target>.kml' <target>
```

{{< fluid_img "/img/nmap-traceroute-nsysu.jpg" >}}
{{< fluid_img "/img/nmap-traceroute-nsysu-gmap.png" >}}
