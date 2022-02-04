+++
topics = []
date = "2016-01-29T00:00:00+08:00"
title = "Reading Traceroute"
tags = ["infosec", "networking", "traceroute", "scanning"]
draft = false
description = "How To A Read Traceroute Report"
toc = false

+++

This post extends the discussion on traceroute in previous post [Traceroute, Firewalls & Geo-IP]({{< relref "post/traceroute-firewalls-geoip.md#output-format-explanation" >}}), and focused on intepreting the traceroute report.

#### Output format explanation:
```
           v--- the router/ip-addr traversed by the packet 
[Hop]     [Hostname/(IP-addr)]      [RTT1]  [RTT2]  [RTT3]
 ^--- transit no. of the route       ^---- round-trip time
```
The round-trip time (RTT) is the latency (delay between sending the packet and getting the response).

By default, traceroute sends 3 packets per TTL increment. Each column [RTT1]...[RTT3] corresponds to the time it took to get response (round-trip time). 3 different packets give a better sampling of the latency, it also helps for situation where multi-path exist (different link). The unit is in **ms (milliseconds)**. For example,

```
[Hop]   [IP-addr]       [RTT1]      [RTT2]      [RTT3]
  7     204.15.20.45    31.757ms    53.862ms    53.844ms  
```

Discussion with real world examples were covered in [Traceroute, Firewalls & Geo-IP]({{< relref "post/traceroute-firewalls-geoip.md#output-format-explanation" >}}).


---
#### Reading the traceroute:

* **The Hop times:**
    * Consistent times are the main thing to read and evaluate from a traceroute report.
    * Check the RTT of the three packets are consistent per hops. Look at the pattern of multiple traceroute reports.
    * Times >150ms are considered long for a round-trip within same continental; however is normal if traveled across ocean.

* **Increasing latency towards the target:**
    * Sudden increase of response time (including packet loss) in a hop and continuous increasing often indicates issue for the hop (the router), the * also suggests either packet loss or the node simply overloaded:

    ```
     ...
     2  175.137.109.62 (175.137.109.62)    14.947ms    41.973ms    41.883ms  
     3  175.137.109.69 (175.137.109.69)    14.348ms    43.621ms    43.614ms  
     4  10.55.192.57 (10.55.192.57)       309.880ms   309.820ms   309.808ms  
     5  219.158.33.25 (219.158.33.25)     481.462ms   481.399ms       *  
     6  219.158.102.97 (219.158.102.97)   491.782ms   506.038ms   505.972ms  
     7  219.158.24.133 (219.158.24.133)   991.870ms  1091.789ms       *
    ```

* **High latency in the middle that remains consistent:**
    * An jump in latency but remain consistent till the rest does not indicate an issue.

    ```
     ...
     2  175.137.109.70      15.848ms    15.722ms    36.279ms  
     3  175.137.109.61      13.277ms    36.372ms    36.332ms  
     4  10.55.208.185       35.285ms    35.240ms    58.096ms  
     5  27.111.228.94      109.352ms    99.348ms   106.301ms <- [a jump] 
     6  157.240.41.36       33.487ms    59.674ms    59.660ms  
     7  204.15.20.45        31.757ms    53.862ms    53.844ms  
     8  173.252.67.145      36.569ms    59.341ms    59.342ms  
     9  31.13.78.35         30.283ms    56.421ms    56.358ms
    ```

* **High latency in the beginning hops:**
    * If it's first few hops, it indicates local network/subnet issue.

* **Timeouts at the beginning:**
    * If the following hops responded without issue, then it's normal. The router may be configured not to respond to traceroute requests such as ICMP packets, or short-lived TTL packets.

* **Timeouts at the very end:**
    * The target may be blocking ICMP requests or packets involving short-lived TTL flags. However, the target is reachable with normal HTTP/HTTPS request.
    * The packet reached the target but unable to response back due to some issues on the destination point of the return path. Should not affect normal connection.
    * Network problem and affecting the connection.

---
**References:**
1. [How to read a traceroute](https://www.inmotionhosting.com/support/website/how-to/read-traceroute)
