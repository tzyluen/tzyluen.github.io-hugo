+++
topics = []
tags = [ "networking", "infosec", "wireshark" ]
draft = false
description = ""
date = "2016-03-26T00:00:00+08:00"
title = "Wireshark: Changing the default column display"
toc = true

+++

The following setup is intended to streamline the column display for effective analysis when looking at HTTP and HTTPS traffic. The default columns are: `No (Packet number)`, `Time`, `Source`, `Destination`, `Protocol`, `Length`, and `Info`.

![changing-wireshark-columns-settings-01](/img/changing-wireshark-columns-settings-01.png)

---
# Changing the column display

To change the default column display, navigate to `Preferences` :

![changing-wireshark-columns-settings-02](/img/changing-wireshark-columns-settings-02.png)

From the `Preferences`, expands `Appearance` and select `Columns`, then add the new desired column e.g., `Src port` of type `Src port (unresolved)`:

![changing-wireshark-columns-settings-03](/img/changing-wireshark-columns-settings-03.png)

The following shows the final column settings, `No (Packet number)` and `Length` are removed, `Source` and `Destination` addresses changed to `unresolved` to shows actual ip address. Additionally, two new columns: `Src port` and `Dst port` (both `unresolved`) are added to shows actual port number.

![changing-wireshark-columns-settings-04](/img/changing-wireshark-columns-settings-04.png)


---
# Changing the Time Display Format

The default format is '`Seconds Since Beginning of Capture`' which isn't useful if the time of the day is of interest. To change it, navigate to `View` &rarr; `Time Display Format`, and select  `Date and Time of Day`:

![changing-wireshark-columns-settings-05](/img/changing-wireshark-columns-settings-05.png)

Also, change the time precision to `Seconds` to shorten the space required:

![changing-wireshark-columns-settings-06](/img/changing-wireshark-columns-settings-06.png)


---
# Adding HTTP Server Names

Set the display filter as &apos;`http.request`&apos; to only display HTTP requests. Then, from the `Packet Details` pane, expand the `Hypertext Transfer Protocol` and right-click `Host` from the HTTP header and select `Apply as Column` :

![changing-wireshark-columns-settings-07](/img/changing-wireshark-columns-settings-07.png)


The host from the HTTP requests (both GET and POST) will now shown as a column. The final column settings are: 

* Time (Date and Time of Day): YYYY-MM-DD HH:MM:SS
* Source Address (unresolved)
* Source Port (unresolved)
* Destination Address (unresolved)
* Destination Port (unresolved)
* Protocol
* Host
* Info

![changing-wireshark-columns-settings-08](/img/changing-wireshark-columns-settings-08.png)


---
# Adding HTTPS Server Names

1. Use a display filter such as `tcp.port == 443` to narrow down the HTTPS traffic.

2. Look for `TLS` or `Client Hello` from the Protocol and Info columns.

3. Expand the `Secure Sockets Layer` &rarr; `TLSv1.2 Record Layer` &rarr; `Handshake Protocol` &rarr; `Extension: server_name` &rarr; `Server Name Indication extension`, select the `Server Name` field.

4. Then, right-click on the field and select `Apply as Column`:

![changing-wireshark-columns-settings-09](/img/changing-wireshark-columns-settings-09.png)

Use the `ssl.handshake.extensions_server_name` display filter to query server names from the HTTPS traffic:

![changing-wireshark-columns-settings-10](/img/changing-wireshark-columns-settings-10.png)
