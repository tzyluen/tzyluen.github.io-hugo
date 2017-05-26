+++
title = "Tools for scanning malicious urls and websites"
description = ""
topics = []
tags = ["infosec", "web", "malware", "virustotal", "scanning", "remnux"]
draft = false
date = "2015-12-06T00:00:00+08:00"
toc = true

+++

# Online and Web-based

* https://www.virustotal.com: by VirusTotal, a subsidiary of Google
* https://exchange.xforce.ibmcloud.com: by IBM
* http://safeweb.norton.com: by Norton, Symantec
* http://www.avgthreatlabs.com/ww-en/website-safety-reports: by AVG ThreatLabs
* https://cymon.io: by https://eSentire.com
* http://www.reputationauthority.org: by WatchGuard Technologies
* http://isitphishing.org: by https://vadesecure.com

---
# Text-mode based Utility


## VirusTotalApi

```bash
$ git clone https://github.com/doomedraven/VirusTotalApi.git
```

It is an utility to search on VirusTotal databases [1] for malicious URLs and hashes of known malware. It comes pre-installed in REMnux distro [2], a free Linux Toolkit for reverse engineering and analyzing malware.

Get VirusTotal public API key from https://www.virustotal.com/en/documentation/public-api and define the API key in the local config file `$HOME/.vtapi`:

```
[vt]
apikey=<APIKEY>
type=public
```

```
remnux@remnux:~$ vt --url-report malicious-url.com
```

Positive result:

![tools-for-scanning-malicious-urls-and-websites-01](/img/tools-for-scanning-malicious-urls-and-websites-01.png)


Negative result:

![tools-for-scanning-malicious-urls-and-websites-02](/img/tools-for-scanning-malicious-urls-and-websites-02.png)


If the url is not in the database, use the `--url-scan` to submit and scan.
```
remnux@remnux:~$ vt --url-scan malicious-url.com
```

---
**References:**

[1] https://www.virustotal.com <br>
[2] https://remnux.org
