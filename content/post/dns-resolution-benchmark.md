+++
title = "DNS resolution benchmark"
description = "Test the DNS resolution time"
topics = []
tags = ["networking", "linux", "dns", "benchmark"]
draft = false
date = "2016-03-12T00:00:00+08:00"
toc = true

+++


# dig & grep
A straightforward way is by using the `dig` util from the `dnsutils` package and grep the results. This works well for quick debug on-the-go:
```bash
# apt-get install dnsutils
```
```bash
$ dig @202.188.0.132 archive.org | grep "Query time:"
;; Query time: 356 msec

$ dig @8.8.8.8 archive.org | grep "Query time:"
;; Query time: 48 msec
```

`202.188.0.132` is ISP TMnet's name server, and `8.8.8.8` is from Google public name server.



---
# namebench
is a DNS benchmark utility allows user to search for the fastest DNS servers available. It combines data from web browser history, tcpdump output, and standardized datasets to make recommendation.  It comes with console and GUI mode. To tweak the benchmark settings, edit the file `/etc/namebench/namebench.cfg`. For example, to limit the name servers to test by opting the '`include global DNS providers`' only, add the desired name servers into the `[global]` section, and excludes the '`include regional DNS services`' :

```bash
[global]
...
51 58.6.115.42=OpenNIC
52 58.6.115.43=OpenNIC_2
53 72.14.189.120=OpenNIC_3
54 195.46.39.39=SafeDNS
55 195.46.39.40=SafeDNS_2
```
To run in **console mode**, pass additional parameters i.e.,:
```bash 
-r RUN_COUNT, --runs=RUN_COUNT: Number of test runs to perform on each nameserver.
-z CONFIG, --config=CONFIG: Config file to use.
-x, --no_gui: Disable GUI
-i INPUT_SOURCE, --input=INPUT_SOURCE: Import hostnames from an filename or application (alexa...
-t TEMPLATE, --template=TEMPLATE: ascii, html...

$ namebench -z /etc/namebench/namebench.cfg -x -i alexa -t ascii
```

The benchmark result report in `ascii` format:
```bash
Fastest individual response (in milliseconds):
----------------------------------------------
Google Public DN #### 18.82386
SYS-192.168.1.1  ##### 22.26114
OpenDNS-2        ###### 27.55594
OpenDNS          ###### 27.99487
UltraDNS         ########### 57.75499
UltraDNS-2       #################################### 191.39409
DynGuide         ##################################### 196.90704
SafeDNS_2        ##################################################### 285.64906

Mean response (in milliseconds):
--------------------------------
SYS-192.168.1.1  ########### 96.45
OpenDNS          ############## 121.70
Google Public DN ############## 123.50
OpenDNS-2        ############### 130.10
UltraDNS         ############################# 250.59
DynGuide         #################################### 319.88
UltraDNS-2       ######################################## 348.53
SafeDNS_2        ##################################################### 473.00
.
. <snipped>
.
Recommended configuration (fastest + nearest):
----------------------------------------------
nameserver 192.168.1.1     # SYS-192.168.1.1  
nameserver 8.8.4.4         # Google Public DNS-2  
nameserver 208.67.222.222  # OpenDNS-2  
```

For **GUI mode**, simply invoke the program and select the desired options:
```bash
$ namebench
```
The GUI mode will generate a report in HTML format and plot a set of comprehensive charts to depict the performance of each tested DNS server.

The following shows the overall result and it recommends:

<img src="/img/dns-benchmark-namebench-chart05-recommendation.png" alt="dns-benchmark-namebench-chart01-mean-response-duration" style="width: 700px;" />


Mean Response Duration:

![dns-benchmark-namebench-chart01-mean-response-duration](/img/dns-benchmark-namebench-chart01-mean-response-duration.png)


Fastest Invidual Response Duration:

![dns-benchmark-namebench-chart02-fastest-individual-response-duration](/img/dns-benchmark-namebench-chart02-fastest-individual-response-duration.png)


Response Distribution (first 200ms):

![dns-benchmark-namebench-chart03-response-distribution-chart-first200ms](/img/dns-benchmark-namebench-chart03-response-distribution-chart-first200ms.png)


Response Distribution (full):

![dns-benchmark-namebench-chart04-response-distribution-chart-full](/img/dns-benchmark-namebench-chart04-response-distribution-chart-full.png)

