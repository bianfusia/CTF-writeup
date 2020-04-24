# THM - wgelctf

## IP address
10.10.37.40

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.37.40
```
2. The name results will result as shown:
```
root@kali:~# nmap -p- -sV -sC -oN nmap/initial 10.10.37.40
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-24 04:16 UTC
Nmap scan report for ip-10-10-37-40.eu-west-1.compute.internal (10.10.37.40)
Host is up (0.00078s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:DB:E0:19:ED:D8 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.27 seconds
```
- Port ```22``` and port ```80``` is open.
- going to http://10.10.37.40:80 brings you to apache default page.

3. Explore other potential directories with gobuster.
```bash
#install gobuster if you dont have it already
sudo apt install gobuster

#run gobuster with dirbuster wordlists
gobuster dir -u http://10.10.37.40 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

- You will notice that ```/sitemap``` is available.
```
/sitemap (Status: 301)
/server-status (Status: 403)
```

4. ```/sitemap``` will bring you to unapp template website. try running gobusters again on ```/sitemap```.
```bash
gobuster dir -u http://10.10.37.40/sitemap -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

- This will return a few more results:
```
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/sass (Status: 301)
```
