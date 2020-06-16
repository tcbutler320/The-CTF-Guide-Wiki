# Service Enumeration

## Scanning for Services 

### Nmap 

Simple nmap scan to enumerate services from target

```bash
nmap [target ip]
```

Nmap scan to gather service version and banners 

```bash
nmap -sV -sT [target ip]
```

Nmap scan to gather service version, banners, and target OS on all ports

```bash
nmap -v -p- -sV -O -sS -T4 [target ip]
```

Nmap scanning all UDP ports

```bash
nmap -p- -sU [target ip] 
```

### Masscan 

{% embed url="https://github.com/robertdavidgraham/masscan" %}

Masscan is a valid alternative for nmap. It's designed specifically for scanning large network ranges, but can also be used for small one's.

#### Scan a target range and grab banners

```bash
masscan [target ip or ip range] -p[port] --banners --source-port 40000
```

