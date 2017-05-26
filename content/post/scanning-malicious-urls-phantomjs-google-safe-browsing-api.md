+++
title = "Scanning malicious urls (PhantomJS + Google Safe Browsing API)"
topics = []
tags = ["web", "js", "infosec", "malware", "google", "api", "scanning"]
draft = false
description = ""
date = "2015-11-11T00:00:00+08:00"

+++


My examples are in JS, on PhantomJS headless browser, it could be easily adapted to other languages. The script traverses a webpage and harvests all the URLs therein to check for malware/malicious sites through the Google Safe Browsing API.

```bash
192.168.1.9:~$ phantomjs chk-malinks.js http://some.malware.site
1: hshd.io
2: sourceforge.net
3: popup.taboola.com
4: www.geeksvip.com
5: my.hear.com
...
159: nba1001.net
160: www.pressroomvip.com
161: tracking.lifestylejournal.com
162: dsct2.com
163: www.historynut.com
164: www.buro247.my
gsafe response json:{
  "matches": [
    {
      "threatType": "MALWARE",
      "platformType": "ANY_PLATFORM",
      "threat": {
        "url": "nba1001.net"
      },
      "cacheDuration": "300s",
      "threatEntryType": "URL"
    }
  ]
}
```
---
### Google Safe Browsing

To use Google Safe Browsing APIs, get the API key from Google Developer Console. [1] The API calls is limit to 500 urls per request, and maximum of 10,000 requests per day.


* Line 7-8 defines the API key and the Google Safe Browsing API URL.
* Line 67-85 defines the Request settings, and line 71 is the Request header.
* Line 74-83 is the Request body. The request body includes the client information (ID and version) and the threat information (the list names and the URLs). 
* Line 88-97 make the request to Google Safe Browsing API server.

The rest of the code are auxiliary functions:

* Line 35-61 parse the webpage, extract and collect URLs.

{{< gist tzyluen 4c41d7a0d6002fe828f9d932232813ed>}}

---
**References:**
[1] https://developers.google.com/safe-browsing/v4/get-started
