+++
topics = []
tags = ["networking", "infosec", "privacy", "linux", "dns"]
draft = false
date = "2016-03-19T00:00:00+08:00"
title = "Encrypting DNS Traffic"
description = ""
toc = true

+++

{{< toc >}}

---
## DNSCrypt
can be used to increase web browsing privacy and thwart DNS traffic sniffing. It enables encryption and authentication on DNS traffic between the local computer and the remote DNS resolver. It helps to mask the domain resolution (sent in clear text) to the DNS server, before the HTTPS connection initiated to the target website.

### Install

```
# apt-get install dnscrypt-proxy
```

### Select a resolver
Note: An up-to-date dnscrypt-resolvers.csv is also available from [https://github.com/jedisct1/dnscrypt-proxy/blob/master/dnscrypt-resolvers.csv](https://github.com/jedisct1/dnscrypt-proxy/blob/master/dnscrypt-resolvers.csv)

1. Select a resolver from `/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv`.

2. Filter and print the list of resolvers from the dnscrypt-resolvers.csv
    ```
    $ awk -F'[,]' '{print $1 "|" $4 "|" $12}' < /usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv
    ```

3. Get a distinct list by country:
    ```
    $ awk -F'[,]' '!($4 in arr){print $4} {arr[$4]++}' < /usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv
    ```

4. Pick a reliable server such as:

    * fvz-anyone (by OpenNIC)
    * fvz-anyone-ipv6
    * adguard-dns-family-ns1 (Adguard DNS with safesearch and adult content blocking)
    * adguard-dns-ns1 (Remove ads and protect your computer from malware)


5. Update the `dnscrypt-proxy.conf` with the target resolver:
    ```
    $ vim /etc/dnscrypt-proxy/dnscrypt-proxy.conf
    ...
    ResolverName fvz-anyone
    ```

### Modify resolv.conf

1. Modify the resolv.conf file and replace the current resolver addresses with address that the dnscrypt-proxy daemon listening on, as defined in the following files:
    ```
    /etc/dnscrypt-proxy/dnscrypt-proxy.conf
    /etc/systemd/system/sockets.target.wants/dnscrypt-proxy.socket
    ```

    resolv.conf:
    ```
    nameserver 127.0.2.1
    ```

### Start systemd service

```
$ sudo systemctl start dnscrypt-proxy
$ sudo systemctl status dnscrypt-proxy
```

### Verify the DNS traffic is encrypted

Use wireshark and set the capture filter as:

```
host 192.168.1.9 and not broadcast
```

And set the display filter as:

```
ip.addr == 185.121.177.177
```

Note: `fvz-anyone` resolver ip address is `185.121.177.177`

The encrypted DNS traffic will appear as malformed packet (in red) and was mistakenly identified as QUIC protocol during the test:

{{< fluid_img "/img/dnscrypt-encrypted-traffic-wireshark-01.png" >}}
