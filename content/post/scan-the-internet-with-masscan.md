+++
tags = ["infosec", "networking", "linux", "port-scanning", "vulnerability-scanner", "scanning", "masscan"]
draft = false
description = ""
topics = []
date = "2017-03-17T00:00:00+08:00"
title = "Scan the Internet with Masscan"
toc = true

+++

{{< toc >}}


---
The base system used to perform the scans:
```
root@192.168.1.11:~# uname -a
Linux kali 4.9.0-kali3-amd64 #1 SMP Debian 4.9.13-1kali2 (2017-03-07) x86_64 GNU/Linux
```

## Scan large IP block

Scan the entire `175.0.0.0/8` for `port 22, 80, 445`:

{{< fluid_img "/img/scan-the-internet-with-masscan-00.png" >}}

Note: It's risky to perform such as scan as it may trigger some IDS/IPS and get your IP blocked/blacklisted from accessing them permanently.

Scan a Yahoo IP block for port tcp/80:

1. Trace the Yahoo's IP block:

    ```
    root@192.168.1.11:~# mtr -y1 yahoo.com
    ```

    {{< fluid_img "/img/scan-the-internet-with-masscan-01.png" >}}

2. Scan the entire IP block (32768 of hosts):

    {{< fluid_img "/img/scan-the-internet-with-masscan-02.png" >}}



---
## Exclude IP blocks of sensitive part of the Internet

1. Ensure the `excludefile` is defined in `/etc/masscan/masscan.conf`:

    ```
    excludefile=/etc/masscan/exclude.conf
    ```

2. Append the following into the exclude list to prevent scanning them by accident:

    {{< fluid_img "/img/scan-the-internet-with-masscan-03.png" >}}

3. Or, for a non-persistent exclude list, use `--excludefile` option: 

    ```
    masscan 175.0.0.0/8 -p80 --rate 10000 --excludefile exclude.txt
    ```



---
## Include IP blocks for targeted IP blocks

1. Similar to `excludefile`, the file format for the include list:

    ```
    # Include list:
    # For targeting organizations or verticals with multiple blocks
    75.0.0.0/8               # AT& Internet Services
    175.145.0.0/16           # TMNet
    
    # Private IPv4 addresses, to scan entire organization network
    192.168.0.0/16           # class C
    172.16.0.0/12            # class B
    10.0.0.0/8               # class A
    ```

2. Scan with the include file option:

    ```
    masscan -p80 --includefile include.txt
    ```



---
## Transmission Rates

Depends on the point A-to-B network infrastructure and NIC, it can be scaled up to 25 million packets/second.

1. The default transmit rate is 100 packets/second. Takes approx. 6 minutes to scan 32768 hosts on port tcp/80:

    ```
    masscan 98.139.128.0/17 -p80 --rate 100
    ```

    {{< fluid_img "/img/scan-the-internet-with-masscan-04.png" >}}

2. With 10,000 packets/second takes approx. 8 seconds to scan 32768 hosts on port tcp/80:

    ```
    masscan 98.139.128.0/17 -p80 --rate 10000
    ```

    {{< fluid_img "/img/scan-the-internet-with-masscan-05.png" >}}

Note: the known max. transmission rates are:

1. Windows - 250,000 packets/second
2. Linux - 2,500,000 packets/second
3. PF\_RING driver - 25,000,000 packets/second



---
## Specify ports and ranges

```
masscan 175.145.0.0/16 -p22,80,445 --rate 1000
```

{{< fluid_img "/img/scan-the-internet-with-masscan-06.png" >}}



---
## Pull the services and banners

1. Grab the banners, i.e., HTTP server version, title, and etc.

    ```
    masscan 175.145.0.0/16 -p22,80,445 --rate 10000 --banners
    ```

2. Some interesting information:

    {{< fluid_img "/img/scan-the-internet-with-masscan-08.png" >}}



---
## Output formats

1. Sets the output format to binary `-oB` and saves the output in the given filename, which can be read with `--readscan`, and optionally output into a new format later:

    ```
    -oB: binary
    -oL: list
    -oG: grepable
    -oX: xml

    masscan 175.145.0.0/16 -p22,80,445 --banners --rate 10000 -oB 175-145-0-0-masscan.bin
    ```

    {{< fluid_img "/img/scan-the-internet-with-masscan-07.png" >}}

2. Read the saved binary file into new `xml` format file that can be used for parsing and reporting:

    ```
    masscan 175.145.0.0/16 --readscan 175-145-0-0-masscan.bin -oX 175-145-0-0-masscan.xml
    ```



---
## Manage config for different scanning strategies

Save the configuration into a file, multiple conf files for different objectives and strategies. For example, a configuration file for a particular ip block, i.e.,

1. Save the current settings into a conf file (`--echo` and redirect to `175-145-0-0.conf`):

    ```
    masscan 175.145.0.0/16 -p22,80,445 --banners --rate 10000 --echo > 175-145-0-0.conf
    ```

2. Use the saved profile for the same objective:

    ```
    masscan -c 175-145-0-0.conf
    ```
