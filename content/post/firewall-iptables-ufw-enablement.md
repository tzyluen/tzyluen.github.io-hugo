+++
title = "Firewall: iptables/ufw enablement"
tags = ["networking", "infosec", "firewall"]
draft = false
description = ""
date = "2015-11-15T00:00:00+08:00"
toc = true

+++

# Iptables
List the configured rules:
```bash
# iptables -L
# iptables -L -t nat
```
```iptables``` contains 5 tables, (```-t```, ```--tables```): ```raw```, ```filter```, ```nat```, ```mangle``` and ```security```. In common use cases, ```filter``` and ```nat``` is used, where filter is associated with the firewall and nat is used for network address translation such as port forwarding.


___
# Enabling ufw (uncomplicated firewall)
```bash
# ufw enable
# ufw disable
```

Check the service is running:
```bash
# systemctl list-unit-files ufw.service
# systemctl status ufw
```

Check the default rules:
```bash
# ufw status verbose
# ufw show raw
```

Set the default rules:
```bash
# ufw default deny incoming
# ufw default allow outgoing
```
firewall rules are read from top-to-bottom, left-to-right; therefore, once a rule is matched the others will not be evaluated, so put specific rules first.


## Manage ```ufw``` by predefined service names
```bash
# ufw app list          // list application profiles
# ufw app info SSH      // show information on profile SSH
```

```bash
# ufw allow <service_name>
# ufw deny <service_name>       // will drop the connection immediately, client will get the timeout response
# ufw reject <service_name>     // will drop the connection immediately
```

## Extended syntax
```bash
# ufw allow from <ip_addr>/[subnet] to any app <service_name>
```

For example, SSH service:
```bash
# ufw allow ssh                 // open port 22(SSH) for incoming,
# ufw delete allow ssh          // and to delete the opened ssh rules
```

WWW service:
```bash
# ufw allow from 192.168.1.1/24 to any app WWW
# ufw delete allow from 192.168.1.1/24 to any app WWW
```

Use numeric port, open port 3000 (ntop-ng) to 192.168.1.7 only:
```bash
# ufw allow from 192.168.1.7 to any port 3000
```

## Reorders rirewall rules
Rules are read from top-to-bottom, left-to-right, so rule (8) will not work:
```bash
# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] WWW                        ALLOW IN    192.168.1.0/24            
[ 2] SSH                        ALLOW IN    192.168.1.0/24            
[ 3] 3000                       ALLOW IN    192.168.1.7               
[ 4] 8001                       ALLOW IN    192.168.1.7               
[ 5] 8005                       ALLOW IN    192.168.1.7               
[ 6] 8080                       ALLOW IN    192.168.1.7               
[ 7] 1313                       ALLOW IN    192.168.1.7               
[ 8] Anywhere                   REJECT IN   192.168.1.4
```

Delete rule (8) and recrete/insert to rule (1):
```bash
# ufw delete 8
# ufw insert 1 reject from 192.168.1.4 to any
Rule inserted
# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] Anywhere                   REJECT IN   192.168.1.4               
[ 2] WWW                        ALLOW IN    192.168.1.0/24            
[ 3] SSH                        ALLOW IN    192.168.1.0/24            
[ 4] 3000                       ALLOW IN    192.168.1.7               
[ 5] 8001                       ALLOW IN    192.168.1.7               
[ 6] 8005                       ALLOW IN    192.168.1.7               
[ 7] 8080                       ALLOW IN    192.168.1.7               
[ 8] 1313                       ALLOW IN    192.168.1.7               
```


___
## Difference between: DENY and REJECT
| Action  | Description |
|---------|-------------|
|**DENY** | <ul><li>uses the DROP from the iptables, it silently discards the incoming packets.</li><li>will keep the sender waiting until connection attempt times out.</li></ul> |
|**REJECT** | <ul><li>uses the REJECT from the iptables, it sends back an error (RST) packet to the sender of the rejected packet.</li><li>effect immediately with connection refused message.</li></ul>|
