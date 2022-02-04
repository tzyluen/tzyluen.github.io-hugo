+++
date = "2016-02-27T00:00:00+08:00"
title = "Tracing Malicious Network Traffic"
description = "with Nettop / Netstat / Ss to Display a List of Sockets & Routes"
topics = []
tags = ["infosec", "networking", "malware", "nettop", "netstat", "flowtop", "dns", "google", "scanning"]
draft = false
toc = true

+++

{{< toc >}}

## Nettop
Assuming an unknown/suspicious output (i.e., no chat client is being used, but random chat domain appeared: vmp.boldchat.com) is spotted from a DNS monitoring such as in previous post titled - [Sniff DNS queries]({{< relref "post/sniff-dns-queries.md" >}}):

![dnstop-monitoring-01](/img/dnstop-monitoring-large-count-01.png)

Under macOS, the `nettop` util provides list of sockets and routes in details that help to trace down the process that established the connection to the unknown domain:

```
use keys to toggle:
d, delta output
c, collapse all
e, expand all
j, bring up the column selection menu
h, help

$ nettop
```

{{< fluid_img "/img/macos-nettop-monitoring-01.jpg" >}}

---
### Google Transparency Report: Safe Browsing Tool
Since the process is from Firefox on port https (443), the Google Transparency Report [Safe Browsing tool](https://www.google.com/transparencyreport/safebrowsing/diagnostic/) can be used to check if the site is malicious and unsafe.


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## Netstat

On linux, `ss` or `netstat` can be used:

```
-l, --listening       display listening sockets
-n, --numeric         don't resolve service names
-p, --program         show the pid/name of the program
-r, --resolve         resolve numeric address/ports
-t, --tcp             display only TCP sockets
-u, --udp             display only UDP sockets

$ ss -rptu
```

{{< fluid_img "/img/linux-ss-monitoring-01.png" >}}

Similarly, `netstat` would work too:

```
$ sudo netstat -ptuW       <- (to see non-owned process info)
```

{{< fluid_img "/img/linux-netstat-monitoring-01.png" >}}


{{< rawhtml >}}
<br/><br/>
{{< /rawhtml >}}


---
## Flowtop

The `flowtop` from the `netsniff-ng` package also works:

{{< fluid_img "/img/linux-flowtop-monitoring-01.png" >}}
