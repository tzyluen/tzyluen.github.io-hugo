+++
date = "2016-04-09T00:00:00+08:00"
title = "Extract files from captured network traffic pcap (2)"
description = ""
topics = []
tags = ["infosec", "networking", "packet-analysis", "data-carving", "tcpxtract", "tcpextract"]
draft = false
toc = true

+++


# Tcpxtract
* Supports 26 file formats, extensible (`/etc/tcpxtract.conf`), however it requires the clear start and end markers.
* Supports only TCP packets, no UDP.

Live capture from an interface and extract:
```bash
$ mkdir -p /tmp/enp0s3-tcpxtract-output
$ sudo tcpxtract -d enp0s3 -o /tmp/enp0s3-tcpxtract-output
```

Extract from pcap file:

![extracting-files-tcpxtract-01](/img/extracting-files-tcpxtract-01.png)

---
# Tcpextract
* Similar to both `tcpflow` and `tcpxtract`, it extracts all files it recognized from a pcap file or interface.
* Extracts files with their original names, instead of the index names i.e., 0000001.htm.

```bash
$ sudo apt-get install python-nids
$ git clone https://github.com/faust/tcpextract.git
$ cd tcpextract/
$ sudo python setup.py install
```

However, it is not well supported, or unstable.  It failed to extract http traffic during the experiment:

![extracting-files-tcpextract-01](/img/extracting-files-tcpextract-01.png)

So, converts `pcapng` to `pcap` file format:
```bash
$ editcap http_espn.pcapng http_espn.pcap
```

After a few attempt, it still failed silently with zero file extracted.
