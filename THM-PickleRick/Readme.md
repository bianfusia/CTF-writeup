# THM - Pickle Rick

## IP Address
10.10.79.232

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.79.232
```
2. The name results will result as shown:
```
root@kali:~# nmap -p- -sV -sC -oN nmap/initial 10.10.79.232
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-24 12:52 UTC
Nmap scan report for ip-10-10-79-232.eu-west-1.compute.internal (10.10.79.232)
Host is up (0.00066s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20:cf:86:4b:c2:5c:b9:d2:36:25:ef:89:38:57:16:c2 (RSA)
|   256 69:bf:9d:81:2b:b9:d1:3b:6d:cc:ae:a1:cf:5e:e5:2c (ECDSA)
|_  256 a0:6a:b1:1f:2f:2c:09:45:af:81:41:cb:5e:54:11:3c (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
MAC Address: 02:24:9D:FE:0C:FA (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.89 seconds
```

