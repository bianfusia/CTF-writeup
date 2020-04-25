# THM - Tom Ghost

## IP Address
10.10.102.57

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.102.57
```
- Results:
```
root@kali:~# nmap -p- -sV -sC -oN nmap/initial 10.10.102.57
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-25 12:11 UTC
Nmap scan report for ip-10-10-102-57.eu-west-1.compute.internal (10.10.102.57)
Host is up (0.00070s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/9.0.30
MAC Address: 02:C4:D3:5A:26:6A (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.83 seconds
```

2. So the description of the room is very straightforward on the exploit. We will use ```searchsploit``` to find the suitable exploit script to use.
```bash
#update searchsploit as this is quite a recent exploit
searchsploit -u

#run search
searchsploit AJP
```

- Results:
```
root@kali:~# searchsploit AJP
---------------------------------- ----------------------------------------
 Exploit Title                    |  Path
                                  | (/usr/share/exploitdb/)
---------------------------------- ----------------------------------------
AjPortal2Php - 'PagePrefix' Remot | exploits/php/webapps/3752.txt
Apache Tomcat - AJP 'Ghostcat Fil | exploits/multiple/webapps/48143.py
---------------------------------- ----------------------------------------
Shellcodes: No Result
Papers: No Result
```

3. Lets try running the exploit 
