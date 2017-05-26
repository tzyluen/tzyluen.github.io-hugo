+++
topics = []
tags = ["infosec", "networking", "traceroute", "scanning"]
draft = false
date = "2016-02-06T00:00:00+08:00"
title = "Dynamic Traceroute"
description = "mtr - a network diagnostic tool"

+++

### mtr
Often times, troublesome networks won't show up in the results of a few packets. `mtr` combines the functionality of the `traceroute` and `ping` utils and enables user to constantly poll a remote server to see the latency and performance changes over time. It's not installed by default on most linux systems, simply get it from the distribution and package manager of choice:
```bash
$ sudo apt-get install mtr
```

To debug remotely over ssh where no GUI is available, use the `--curses` or `-t` option:
```bash
$ mtr --curses
$ mtr -t
```
Use `'h'` key for help to switch/toggle the views, or cycle the view during the runtime with `y` key.

Network block mode with `mtr -t -y1`:

![mtr-traceroute-network-block-mode](/img/mtr-traceroute-network-block-mode.jpg)


Country code mode with `mtr -t -y2`:

![mtr-traceroute-country-mode](/img/mtr-traceroute-country-mode.jpg)


A very useful feature is the `--report` option, which generate a statistical report based on the number of cycles of traceroute to the target. 
```bash
# -c COUNT:  the number of report cycles
# -r, --report: generate a statistical report
# -w, --report-wide: (do not truncate hostnames)

$ mtr -c 30 --report youku.com
```
![mtr-traceroute-report-mode](/img/mtr-traceroute-report-mode.png)
