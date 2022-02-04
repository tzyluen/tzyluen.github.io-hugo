+++
title = "Hping3: Use Cases"
topics = []
tags = ["infosec", "networking", "hping3", "port-scanning", "scanning", "stealth"]
draft = false 
description = "Stealth scan and firewall evasion host discovery"
date = "2015-11-21T00:00:00+08:00"
toc = true

+++

{{< toc >}}

## hping3 and the firewall

```
Mode
  default mode     TCP
  -0  --rawip      RAW IP mode
  -1  --icmp       ICMP mode
  -2  --udp        UDP mode
  -8  --scan       SCAN mode.
                   Example: hping --scan 1-30,70-90 -S www.target.host
  -9  --listen     listen mode
```

### ICMP mode

The typical `ping` utility and the `hping3` equivalent, sending ICMP-echo and receiving ICMP-reply:

```
$ ping google.com -c 3
PING google.com (216.58.196.14) 56(84) bytes of data.
64 bytes from google.com (216.58.196.14): icmp_seq=1 ttl=57 time=23.0 ms
64 bytes from google.com (216.58.196.14): icmp_seq=2 ttl=57 time=21.9 ms
64 bytes from google.com (216.58.196.14): icmp_seq=3 ttl=57 time=25.3 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 21.960/23.479/25.386/1.436 ms


$ ping utm.my -c 5
PING utm.my (161.139.21.51) 56(84) bytes of data.

--- utm.my ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4103ms
```

```
$ sudo hping3 -1 utm.my -c 5
HPING utm.my (eth0 161.139.21.51): icmp mode set, 28 headers + 0 data bytes

--- utm.my hping statistic ---
5 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
```

Note: if the default ping 'ICMP echo' request was blocked, use other modes and ports, e.g., TCP, UDP.


### TCP mode

TCP ping to port 80

```
-S, set SYN flag
-p, set port number
-c, count

$ sudo hping3 -S utm.my -p 80 -c 3
HPING utm.my (enp0s3 161.139.21.51): S set, 40 headers + 0 data bytes
len=46 ip=161.139.21.51 ttl=57 DF id=0 sport=80 flags=SA seq=0 win=14600 rtt=27.6 ms
len=46 ip=161.139.21.51 ttl=57 DF id=0 sport=80 flags=SA seq=1 win=14600 rtt=31.4 ms
len=46 ip=161.139.21.51 ttl=57 DF id=0 sport=80 flags=SA seq=2 win=14600 rtt=135.1 ms

--- utm.my hping statistic ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 27.6/64.7/135.1 ms
```

the ping responses with `flags=SA`, the abbreviated flags are `SYN/ACK` from the TCP flags:

{{< compact_table
"Flag          | Description"
"CWR           | Congestion Window Reduced (CWR) flag is set by the sending host to indicate that it received a TCP segment with the ECE flag set."
"ECE (ECN-Echo)| indicate that the TCP peer is ECN capable during 3-way handshake."
"URG           | indicates that the URGent pointer field is significant"
"ACK           | indicates that the ACKnowledgment field is significant (Sometimes abbreviated by tcpdump as \".\")"
"PSH           | Push function"
"RST           | Reset the connection (Seen on rejected connections)"
"SYN           | Synchronize sequence numbers (Seen on new connections)"
"FIN           | No more data from sender (Seen after a connection is closed)"
>}}

source: https://doc.pfsense.org/index.php/What_are_TCP_Flags


### UDP mode

UDP ping to port 53 (DNS), if port 53 (DNS) is not filtered, but closed/not listening on the server:

```
host: 192.168.1.9
53                         ALLOW IN    Anywhere                  
53 (v6)                    ALLOW IN    Anywhere (v6)             
```

```
192.168.1.10:~# nmap -sT -sU 192.168.1.9
...
Not shown: 999 open|filtered ports, 995 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
53/tcp   closed domain
80/tcp   open   http
3000/tcp closed ppp
8080/tcp open   http-proxy
53/udp   closed domain
...
```

And the TCP responses with `flags=RA`, i.e., `RST/ACK`, similarly the UDP ping response and both with 0 packet loss.

```
192.168.1.10:~# hping3 -S 192.168.1.9 -p 53 -c 1
HPING 192.168.1.9 (eth0 192.168.1.9): S set, 40 headers + 0 data bytes
len=46 ip=192.168.1.9 ttl=64 DF id=21027 sport=53 flags=RA seq=0 win=0 rtt=5.8 ms

--- 192.168.1.9 hping statistic ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 5.8/5.8/5.8 ms


192.168.1.10:~# hping3 -2 192.168.1.9 -p 53 -c 1
HPING 192.168.1.9 (eth0 192.168.1.9): udp mode set, 28 headers + 0 data bytes
ICMP Port Unreachable from ip=192.168.1.9 name=UNKNOWN   
status=0 port=2391 seq=0

--- 192.168.1.9 hping statistic ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 30.5/30.5/30.5 ms
```


Likewise, if port 53 (DNS) is filtered (rejected), the response flags would also be `RA`.

```
host: 192.168.1.9
53/tcp                     REJECT IN   Anywhere                  
53/udp                     REJECT IN   Anywhere                  
53/tcp (v6)                REJECT IN   Anywhere (v6)             
53/udp (v6)                REJECT IN   Anywhere (v6)
```

```
192.168.1.10:~# hping3 -S 192.168.1.9 -p 53 -c 1
HPING 192.168.1.9 (eth0 192.168.1.9): S set, 40 headers + 0 data bytes
len=46 ip=192.168.1.9 ttl=64 DF id=0 sport=53 flags=RA seq=0 win=0 rtt=6.2 ms

--- 192.168.1.9 hping statistic ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 6.2/6.2/6.2 ms


192.168.1.10:~# hping3 -2 192.168.1.9 -p 53 -c 1
HPING 192.168.1.9 (eth0 192.168.1.9): udp mode set, 28 headers + 0 data bytes
ICMP Port Unreachable from ip=192.168.1.9 name=UNKNOWN   
status=0 port=1480 seq=0

--- 192.168.1.9 hping statistic ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 63.7/63.7/63.7 ms
```

### SCAN mode

`-8  --scan` mode can be used to automated low-level port scanner (port range), possibly to uncover subtle aspects of hosts behind the firewall.

```
192.168.1.10:~# hping3 -8 21-80 -S 192.168.1.9
Scanning 192.168.1.9 (192.168.1.9), port 21-80
60 ports to scan, use -V to see all the replies
+----+-----------+---------+---+-----+-----+-----+
|port| serv name |  flags  |ttl| id  | win | len |
+----+-----------+---------+---+-----+-----+-----+
   22 ssh        : .S..A...  64     0 29200    46
   80 http       : .S..A...  64     0 29200    46
All replies received. Done.
```
