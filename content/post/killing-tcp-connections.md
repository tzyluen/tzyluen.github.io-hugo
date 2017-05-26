+++
topics = []
tags = [ "networking", "linux", "infosec", "gdb"]
draft = false
date = "2016-03-05T00:00:00+08:00"
title = "Killing TCP connections"
description = ""
toc = true

+++

# Tcpkill
enables priviledged user to kill TCP connections, it uses the tcpdump expression. It's part of the **dsniff** package. The default degree of brute force to use in killing a connection is 3, fast connections may require a higher number in order to land a RST in the moving receive window.
```bash
tcpkill [-i interface] [-1...9] expression
```

For example, to kill the established and also to prevent any connections from 192.168.1.7 (client) to 192.168.1.9 (server) on port 22, use the following parameters:
```bash
192.168.1.9:~$ sudo /usr/sbin/tcpkill -i enp0s3 src 192.168.1.7 and dst port 22
tcpkill: listening on enp0s3 [src 192.168.1.7 and dst port 22] 
192.168.1.7:57502 > 192.168.1.9:22: R 462778052:462778052(0) win 0
192.168.1.7:57502 > 192.168.1.9:22: R 462782148:462782148(0) win 0
192.168.1.7:57502 > 192.168.1.9:22: R 462790340:462790340(0) win 0
...
```

On the receiving end 192.168.1.7 (client), connection is killed and new connection is denied:
```bash
192.168.1.9:~$ wpacket_write_wait: Connection to 192.168.1.9: Broken pipe
192.168.1.7:~$ 
192.168.1.7:~$ ssh tzy@192.168.1.9
192.168.1.9's password: 
ssh_dispatch_run_fatal: Connection to 192.168.1.9: Broken pipe
192.168.1.7:~$ 
```


---
# GDB
If only the socket required to close but the process must keep alive, we could attach the process under a debugger and close the FD (file descriptor).

To illustrate, open a simple client (192.168.1.7) and server (192.168.1.9) TCP connection:
```bash
192.168.1.9:~$ ncat -v -l -p 9999
Ncat: Version 7.40 ( https://nmap.org/ncat )
Ncat: Listening on :::9999
Ncat: Listening on 0.0.0.0:9999
Ncat: Connection from 192.168.1.7.
Ncat: Connection from 192.168.1.7:65253.
hello world
```

```bash
192.168.1.7:~$ nc 192.168.1.9 9999
hello world
```

```bash
192.168.1.9:~$ netstat -ptuW
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 192.168.1.9:9999        192.168.1.7:65253       ESTABLISHED 4050/ncat           
...
```
Locate and identify the target socket's file, `$pid`, i.e., `4050` and also the `FD`, i.e., `5u`

```bash
192.168.1.9:~$ lsof -np $pid 
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
ncat    4050  tzy    5u  IPv4  31028      0t0    TCP 192.168.1.9:9999->192.168.1.7:65253 (ESTABLISHED)
```

Then attach the process to gdb, close the socket with the `FD` identifier and detach the debugger with quit:
```bash
192.168.1.9:~$ gdb -p $pid
...
(gdb) call close(5u)
$1 = 0
(gdb) quit
```
