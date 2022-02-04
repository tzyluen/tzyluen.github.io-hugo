+++
draft = false
description = ""
topics = []
tags = [ "infosec", "networking", "linux", "ngrep", "packet-analysis", "sniffing", "http", "dns"]
date = "2016-04-16T00:00:00+08:00"
title = "Ngrep: quick peek at http traffic"
toc = false

+++

{{< toc >}}

---
## Quick peek

Monitor activities on device `eth0 port 80`:

`-W byline`: linefeeds (LF) are printed as linefeeds, more readable.

`-qt`: quiet mode and print human-readable timestamp.

```
# ngrep -d eth0 -W byline -qt port 80
```


---
## Sorts out unique User-Agent (devices)

In corporate environment, desktop/laptop OS build is often standardized. However, with the BYOD initiative, the network has becomes even more vulnerable. To quickly identify the type of devices on the network, we could do:

```
$ sudo ngrep -qt -W single -d eth0 -P~ 'User-Agent:' 'port 80' > http-user-agent.txt
$ sed 's/.*User-Agent/User-Agent/' http-user-agent.txt | sed 's/~.*//' | sed '/^$/d' > user-agents.txt
$ cat user-agents.txt | sort | uniq -c | sort -rn
  28 User-Agent: Chrome 20.0.1092.0 (Win 7)" useragent="Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.6 (KHTML, like Gecko) Chrome/20.0.1092.0 Safari/536.6
   7 User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.76 Safari/537.36
   5 User-Agent: Chrome 15.0.874.120 (Vista)" useragent="Mozilla/5.0 (Windows NT 6.0) AppleWebKit/535.2 (KHTML, like Gecko) Chrome/15.0.874.120 Safari/535.2
   4 User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:45.0) Gecko/20100101 Firefox/45.0
```

Or, use `tshark` :

```
$ sudo tshark -i eth0 -f "port 80" -Y "http contains \"User-Agent:\"" -Tfields -e http.user_agent > user-agents.txt
$ cat user-agents.txt | sort | uniq -c | sort -rn
```


---
## Monitor the occurrence of the keywords

Capture network traffic matches tcp port 80 (HTTP) on GET/POST methods:

```
# ngrep -d eth0 -q -i "^GET |^POST " tcp and port 80
```

Monitor the occurrence of the words `user` or `pass`, case insensitive:

```
# ngrep -d eth0 -wi 'user | pass' port 80
```

Read from a pcap file, search for `GET` and `POST` requests:

```
# ngrep -qt -W byline -I capture.pcap | grep 'GET\|POST'
```


---
## Monitor HTTP GET | POST traffic by IP addresses

Matches all headers containing pattern string 'HTTP' sent to/from ip address starting with 172.16:

```
# ngrep -qt -I capture.pcap 'HTTP' 'host 172.16' | grep 'GET\|POST'
```

{{< fluid_img "/img/ngrep-quick-peek-at-http-traffic-01.png" >}}


---
## DNS

Capture incoming/outgoing to/from eth0 matches DNS queries/responses:

```
# ngrep -qt -W byline -d eth0 udp and port 53
```
