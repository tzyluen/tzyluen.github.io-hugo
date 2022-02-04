+++
title = "Automate port scanning with Nmap & Ndiff"
description = "To observe the changes of open ports over time"
topics = []
tags = ["infosec", "scanning", "port-scanning", "nmap", "automation"]
draft = false
date = "2016-05-18T00:00:00+08:00"
toc = true

+++

{{< toc >}}


---
## Nmap Ndiff

`Ndiff` is a tool to aid in the comparison of `Nmap` scans. Ndiff, like the standard diff utility, compares two scans at a time. It takes two Nmap XML output files and prints the differences between them. The differences observed are:

1. Host states (e.g. up to down)
2. Port states (e.g. open to closed)
3. Service versions (from -sV)
4. OS matches (from -O)
5. Script output



### Scan and interpret the results/diffs

1. Do a fast scan `-F`, and output result in XML format:

    ```
    # nmap -F -sS -sV -oX 192-168-1-10-$(date +%F-%R) 192.168.1.10 
    ```

2. Do a full TCP port scan, service/version detection and output result in XML format:

    ```
    # nmap -p 1-65535 -sS -sV -oX 192-168-1-10-$(date +%F-%R) 192.168.1.10 
    ```

3. Compare the two results with `ndiff`:

    ```
    -v, --verbose
       Include all hosts and ports in the output, not only those that have changed.
    
    $ ndiff -v file1 file2
    ```

4. The full TCP port scan (`-p 1-65535`) exposed 3 additional ports:

    {{< fluid_img "/img/scanning-a-network-on-a-schedule-with-nmap-ndiff-01.png" >}}



---
## Automation

Put the regular `nmap` scan into a script and `ndiff` its XML format results, and `mailto` the target recipient(s):

sh script: `001-LAN-192-168-1-daily.sh`

```bash
#!/bin/sh
BASENAME="001-LAN-192-168-1-daily-scan"
TARGETS="192.168.1.1/24"
OPTIONS="-v --top-ports 1000 -T4 -sV"
date=`date +%F`
cd /home/tzy/scans
nmap $OPTIONS $TARGETS -oA $BASENAME-$date > /dev/null
if [ -e $BASENAME-prev.xml ]; then
       ndiff $BASENAME-prev.xml $BASENAME-$date.xml > $BASENAME-diff-$date
       echo "*** NDIFF RESULTS ***"
       cat $BASENAME-diff-$date
       echo
fi
echo "*** NMAP RESULTS ***"
cat $BASENAME-$date.nmap
ln -sf $BASENAME-$date.xml $BASENAME-prev.xml
```

Note: weekly scan can be extended to full TCP scan such as: `-v -p 1-65535 -T4 -sV`

{{< glow_text_app_menu "Crontab:" >}}

```
MAILTO=tzy
0 0 * * 1-6 /root/cron/001-LAN-192-168-1-daily.sh    # daily at 00:00AM, Mon-Sat 
0 0 * * sun /root/cron/001-LAN-192-168-1-weekly.sh   # weekly at 00:00AM, Sun
```

And it can be easily extended to include weekly, monthly, policy-based scans with carefully crafted scanning strategy.
