+++
draft = false
date = "2016-09-03T00:00:00+08:00"
title = "Port-knocking"
description = "a stealth method to open ports for incoming connections"
tags = ["networking", "infosec", "linux", "port-scanning", "scanning", "firewall", "stealth", "port-knocking"]
topics = []
toc = true

+++

{{< toc >}}


---
## Introduction

Port-knocking is a stealth method to open ports that the firewall keeps closed by default. A port-knock server listens to all traffic on an ethernet (or PPP) interface, looking for a special "knock" sequences of port-hits.

A client system makes these port-hits by sending a TCP (or UDP) packet to a port on the server. When the server detects a specific sequence of port-hits, it runs a command defined in its configuration file. This can be used to open up holes in a firewall for quick access. The primary benefit is that it suppresses the regular port scan and appears as not availale.

Essentially, the port-knocking strategy is security by obscurity. 



---
## Enabling knockd

A prerequisite to enable port-knocking is `iptables`. The `knockd` is a small port-knock daemon that implements the port-knock server, dynamically manipulate the firewall rules to open and close ports.

1. Install:
    ```
    192.168.1.9:~$ sudo apt-get install knockd
    ```

2. Set the default configuration, enable the service at init and set the target network interface:
    ```
    192.168.1.9:~$ sudo vim /etc/default/knockd
    ...
    START_KNOCKD=1
    KNOCKD_OPTS="-i enp0s3"
    ```

3. Set the desired port-knocking sequence `1300`, `8888`, `9394` in `/etc/knockd.conf`:
    ```
    [options]
            #UseSyslog
            logfile     = /var/log/knockd.log

    [openSSH]
            #sequence    = 1300:tcp,8888:udp,9394:tcp
            sequence    = 1300,8888,9394
            seq_timeout = 10
            command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
            tcpflags    = syn

    [closeSSH]
            sequence    = 9394,8888,1300
            seq_timeout = 10
            command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
            tcpflags    = syn
    ```

4. Start the service:
    ```
    192.168.1.9:~$ sudo systemctl start knockd
    ```



---
## Port-knocking

The utility from the `knockd` package. 

### Open the port

```
192.168.1.11:~$ knock -v 192.168.1.9 1300 8888 9394
hitting tcp 192.168.1.9:1300
hitting tcp 192.168.1.9:8888
hitting tcp 192.168.1.9:9394
```

Note: it's possible to combine TCP and UDP port:

```
192.168.1.11:~$ knock -v 192.168.1.9 1300:tcp 8888:udp 9394:tcp
```

### Close the port

```
192.168.1.11:~$ knock -v 192.168.1.9 9394 8888 1300
```



---
## Using Hping3 / Nmap

The following comes handy for any remote system to makes port-hits without install the `knockd` package to get `knock`.

```
-S: set SYN tcp flag
-p: port
-c: count, stop after sending (and receiving) count response packets

$ hping3 -S -p $ARG -c 1 $HOST
```

A simple shell script `port-knocking.sh` to knocks target host `$HOST` of `$@` (list of sequences) iteratively.

{{< highlight bash >}}
#!/bin/sh
HOST=$1
shift
for ARG in "$@"
do
    hping3 -S -p $ARG -c 1 $HOST
    #nmap -Pn --host-timeout 100 --max-retries 0 -p $ARG $HOST
done
{{< / highlight >}}


### Open the port

1. Initial state - 192.168.1.9 port 22/tcp closed:

    {{< fluid_img "/img/port-knocking-01.png" >}}

    Connection timed out:

    {{< fluid_img "/img/port-knocking-02.png" >}}

2. Knocks and opens 192.168.1.9:22 with SYN tcpflag and port 1300, 8888, 9394 sequence:

    {{< fluid_img "/img/port-knocking-03.png" >}}

3. Connects to the host:

    {{< fluid_img "/img/port-knocking-05.png" >}}

4. The `/var/log/knockd.log` entries:

    ```
    ...
    [2016-09-03 18:20] 192.168.1.11: openSSH: Stage 1
    [2016-09-03 18:20] 192.168.1.11: openSSH: Stage 2
    [2016-09-03 18:20] 192.168.1.11: openSSH: Stage 3
    [2016-09-03 18:20] 192.168.1.11: openSSH: OPEN SESAME
    [2016-09-03 18:20] openSSH: running command: /sbin/iptables -A INPUT -s 192.168.1.11 -p tcp --dport 22 -j ACCEPT
    ```


### Close the port

1. Runs the `port-knocking.sh` script and pass the close port sequence 9394, 8888, 1300:

    {{< fluid_img "/img/port-knocking-04.png" >}}

3. The `/var/log/knockd.log` entries:

    ```
    ...
    [2016-09-03 19:37] 192.168.1.11: closeSSH: Stage 1
    [2016-09-03 19:37] 192.168.1.11: closeSSH: Stage 2
    [2016-09-03 19:37] 192.168.1.11: closeSSH: Stage 3
    [2016-09-03 19:37] 192.168.1.11: closeSSH: OPEN SESAME
    [2016-09-03 19:37] closeSSH: running command: /sbin/iptables -D INPUT -s 192.168.1.11 -p tcp --dport 22 -j ACCEPT
    ```



---
## Alternatives

There are a lot of existing implementations available to support various platforms [1].
For instance, a more advanced `fwknop` [2] which implements an authorization scheme known as Single Packet Authorization (SPA) for strong service concealment.



---
**References:**
1. http://www.portknocking.org/view/implementations
2. https://github.com/mrash/fwknop
