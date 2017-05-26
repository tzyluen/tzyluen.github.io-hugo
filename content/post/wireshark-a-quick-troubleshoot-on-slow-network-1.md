+++
description = ""
topics = []
date = "2016-04-23T00:00:00+08:00"
title = "Wireshark: A quick troubleshoot on slow network (1)"
tags = ["networking", "wireshark", "troubleshoot", "packet-analysis"]
draft = false
toc = true

+++

**Runs wireshark and starts capturing the network packets until the statistic builds.**

# Quick drill into errors and connection issues

1. Navigates to `Analyze` &rarr; `Expert Info`, a high number of errors and warnings indicates problems.

    ![wireshark-a-quick-troubleshoot-on-slow-network-03](/img/wireshark-a-quick-troubleshoot-on-slow-network-03.png)


# Identify the protocols with high traffic


1. Navigates to `Statistics` &rarr; `Protocol Hierarchy`:

    ![wireshark-a-quick-troubleshoot-on-slow-network-01](/img/wireshark-a-quick-troubleshoot-on-slow-network-01.png)

2. Observes the resulting Protocol Hierarchy Statistic and pins down the suspicious/troubling protocol (e.g., high percentage of P2P or broadcast), right-click and select `Apply As Filter` &rarr; `Selected` to apply the filter for further analysis.


# Get the connection speed to a site

1. Use a display filter such as `ip.addr == 192.168.1.7 && ip.addr == 202.170.56.12 && tcp.port == 22` to filter the SSH traffic of the target site.

    ![wireshark-a-quick-troubleshoot-on-slow-network-05](/img/wireshark-a-quick-troubleshoot-on-slow-network-05.png)

2. Navigates to Wireshark menu `Statistics` &rarr; `Flow Graph`.
    2. Select show only `Displayed packets`.
    2. Right-click on the TCP flow diagram and zoom to increase the details of the packet.
    2. Observe the connection establishment timestamp and response time. High number of retransmissions, and re-established connections indicate problems.

    ![wireshark-a-quick-troubleshoot-on-slow-network-06](/img/wireshark-a-quick-troubleshoot-on-slow-network-06.png)

# Get the time spent in waiting for a response

1. To show the time difference from the previous packet, add the `Delta time` into the display columns. From the column displays, right-click and select `Column Preferences` &rarr; add a new column and set the type as `Delta time`.

    ![wireshark-a-quick-troubleshoot-on-slow-network-04](/img/wireshark-a-quick-troubleshoot-on-slow-network-04.png)


# Identify the bad packets (TCP errors) ratio

1. Navigates to `Statistics` &rarr; `I/O Graph`. Customizes the IO graphs with:

    * TCP errors (`tcp.analysis.flags`) to filters the bad TCP packets.
    
    * Retransmission (`tcp.analysis.retransmission || tcp.analysis.fast_retransmission`) filters the retransmitted packets.
    
    A consistently high TCP errors and retransmission ratio often indicate problems. For instance, approx. half of the total packets were indicated in figure below:

    ![wireshark-a-quick-troubleshoot-on-slow-network-02](/img/wireshark-a-quick-troubleshoot-on-slow-network-02.png)
