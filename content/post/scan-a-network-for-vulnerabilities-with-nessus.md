+++
date = "2017-03-31T00:00:00+08:00"
title = "Scan a network for vulnerabilities with Nessus"
draft = false
description = ""
topics = []
tags = ["infosec", "networking", "scanning", "nessus", "port-scanning", "vulnerability-scanner"]
toc = true

+++

{{< toc >}}


---
## Scan a network

{{< glow_text_app_menu "Target: 192.168.1.0/24" >}}

Nessus provides a set of ready-to-use templates. For general scans, the {{< glow_text_app_menu "(1) Advanced Scan" >}} and {{< glow_text_app_menu "(2) Basic Network Scan" >}} would work. The differences are the {{< glow_text_app_menu "Advanced Scan" >}} supports the {{< glow_text_app_menu "Compliance" >}} and {{< glow_text_app_menu "Plugins" >}} which can be used to fine-tune the compliance checks (credentials are required) and plugins.

{{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-01.png" >}}


### Advanced Scans

1. Navigates to {{< glow_text_app_menu "Scans" >}} &rarr; {{< glow_text_app_menu "New Scan" >}} &rarr; {{< glow_text_app_menu "Advanced Scan" >}}, insert the Name, Description, and Targets. The Schedule and Notifications options enable the scan to be performed at certain time and email the results to a list of recipients automatically.

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-02.png" >}}

2. Tune the rest of the settings:

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-03.png" >}}

3. Once all is set, save. The scan job will be listed in My Scans folder.


### Basic Network Scans

1. Navigates to {{< glow_text_app_menu "Scans" >}} &rarr; {{< glow_text_app_menu "New Scan" >}} &rarr; {{< glow_text_app_menu "Basic Network Scan" >}}, insert the Name, Description and Targets, then save.

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-04.png" >}}


### Launch A Scan

1. From the My Scans folder, select a task from the list to launch the scan.

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-05.png" >}}

2. Once the scan complete, the status bar will changed to checked.

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-06.png" >}}



---
## Results

1. The scan results are grouped by host, and vulnerabilities (color-coded by severity).

2. To prepare both summary and technical reports to circulate among teams, use the {{< glow_text_app_menu "Export" >}} &rarr; {{< glow_text_app_menu "PDF" >}} (or HTML, CSV, Nessus, Nessus DB)

3. Clicking the vulnerabilities bar will drill-down to the next-level of 192.168.1.1 vulnerabilities:

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-07.png" >}}

4. The next level of drill-down will display the list of vulnerabilities exposed by Nessus on target 192.168.1.1, and clicking the specific vulnerability will drill-down to the attack vectors info:

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-10.png" >}}

5. The description of the vulnerability, the recommended solution, and the payload used during the scan are documented:

    i. Description of the vulnerability and the attack vector.

    ii. The recommended solution.

    iii. The attack code/payload during the scan.

    iv. Network port used for this attack.

    v. A summarize of the risk factor information.

    vi. Vulnerability information on known exploit availability and publication date.

    vii. Reference information from the CVE (Common Vulnerabilities and Exposures) network.
 
    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-11.png" >}}



---
## Exports
### Executive Report

1. To generate an executive report, choose {{< glow_text_app_menu "Executive Summary" >}}:

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-08.png" >}}


### Technical Report

1. To generate a technical report, choose {{< glow_text_app_menu "Custom" >}}, select {{< glow_text_app_menu "Vulnerabilities" >}} to include the data, and group by {{< glow_text_app_menu "Host" >}}.

    {{< fluid_img "/img/scan-a-network-for-vulnerability-with-nessus-09.png" >}}



---
**References:**
1. https://www.tenable.com/products/nessus-vulnerability-scanner
