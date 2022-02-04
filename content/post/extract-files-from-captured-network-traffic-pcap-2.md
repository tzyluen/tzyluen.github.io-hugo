+++
date = "2016-04-09T00:00:00+08:00"
title = "Extract files from captured network traffic pcap (2)"
description = ""
topics = []
tags = ["infosec", "networking", "packet-analysis", "data-carving", "tcpxtract", "tcpextract"]
draft = false
toc = true

+++

{{< toc >}}

---
## Tcpxtract

* Supports 26 file formats, extensible (`/etc/tcpxtract.conf`), however it requires the clear start and end markers.
* Supports only TCP packets, no UDP.

1. Live capture from an interface and extract:

    ```
    $ mkdir -p /tmp/enp0s3-tcpxtract-output
    $ sudo tcpxtract -d enp0s3 -o /tmp/enp0s3-tcpxtract-output
    ```

2. Extract from the pcap file:

    {{< fluid_img "/img/extracting-files-tcpxtract-01.png" >}}


---
## Tcpextract

* Similar to both `tcpflow` and `tcpxtract`, `tcpextract` extracts all files it recognized from a pcap file or interface.
* It also extracts files with their original names, instead of the index names i.e., 0000001.htm.

    ```
    $ sudo apt-get install python-nids
    $ git clone https://github.com/faust/tcpextract.git
    $ cd tcpextract/
    $ sudo python setup.py install
    ```

1. During the experiment, it failed to extract http traffic, the package is not well supported, and still unstable with error.

2. After a few attempts, e.g., attempted to convert between `pcapng` and `pcap` format, it still failed silently with zero file extracted.

    {{< fluid_img "/img/extracting-files-tcpextract-01.png" >}}

    ```
    $ editcap http_espn.pcapng http_espn.pcap
    ```

