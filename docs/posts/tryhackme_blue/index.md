---
date: 2024-07-04
categories:
 - walkthrough

tags:
 - samba
 - windows
 - tryhackme
---

# Blue Walkthrough

In this lab you will learn how to use the **Eternal Blue** exploit in order to get access to a windows machine.  This machine is considered as an easy introduction for beginners.

<!-- more -->

## Update Host File

> Start the target machine and add the IP address to your host file.

| os | path |
| --- | --- |
| linux / mac | /etc/hosts |
| windows | C:\Windows\System32\etc\drivers\host |

```
<ip> blue
```

## Task 1 - Recon

>Attempt to perform some recon and investigation into the target(s) prior to scanning

### ping host

> Confirm the host to see if responds to ICMP requests.

**command**
```
ping blue -c 4
```

**result**

```
64 bytes from blue (10.10.15.43): icmp_seq=1 ttl=125 time=331 ms
64 bytes from blue (10.10.15.43): icmp_seq=2 ttl=125 time=251 ms
64 bytes from blue (10.10.15.43): icmp_seq=3 ttl=125 time=274 ms
64 bytes from blue (10.10.15.43): icmp_seq=4 ttl=125 time=297 ms

--- blue ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 251.086/288.358/331.494/29.719 ms
```

![Made with VHS](https://vhs.charm.sh/vhs-4n2fXGRCtLKxxekL2Ihq22.gif)

We can successfully ping the host.

---

> Scan the target(s) to uncover potential areas of entry, afterwords attempt to enumerate in order to gather more information about the potential areas of entry (list directors, users, ,etc)

### Port Scan
> Scan the target machine using nmap in order to find open ports.  

```
nmap -T4 -p- blue
```

**results**

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
```

![Made with VHS](https://vhs.charm.sh/vhs-1dZTaSWX5IKGu1DOybBpGU.gif)

We can see the open ports on the target machine.
With this information we can answer the question below.

!!! Success "How many ports are open with a port number under 1000?"
    3


### Service Detection
> Perform a more detailed nmap scan to enumerate services running on the open ports.

**command**
```
nmap -T4 -sV -sC blue
```

**results**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-04 14:31 EST
Nmap scan report for blue (10.10.15.43)
Host is up (0.31s latency).
Not shown: 991 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2024-07-04T19:33:39+00:00
|_ssl-date: 2024-07-04T19:33:46+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2024-07-03T19:20:11
|_Not valid after:  2025-01-02T19:20:11
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49159/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h00m00s, deviation: 2h14m10s, median: 0s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-07-04T14:33:38-05:00
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:40:bf:78:6a:f9 (unknown)
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-07-04T19:33:38
|_  start_date: 2024-07-04T19:20:09

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 129.19 seconds

```
It looks like the **smb-os-discovery** script has done has a huge favor and determined the operating system type and version.

A quick google search will lead us to details about an OS level vulnerability named "Eternal Blue"

!!! warning "ms17-010 - Eternal Blue"

    
    OS level vulnerability found for Windows 7 Professional 7601 Service Pack 1

    https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/

### Verify Exploit

> Use an nmap script to test if **ms17-010** is exploitable on the target machine

**command**
```
nmap --script=smb-vuln-ms17-010.nse -p445 blue
```

**result**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-04 14:50 EST
Nmap scan report for blue (10.10.15.43)
Host is up (0.31s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

Nmap done: 1 IP address (1 host up) scanned in 3.55 seconds

```

Sure enough, nmap has confirmed that the target machine is vulnerable to this specific vulnerability. 



### Answers


!!! Success "How many ports are open with a port number under 1000?"
    3


!!! success "What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)"

    ms17-010


## Task 2 - Gain Access
