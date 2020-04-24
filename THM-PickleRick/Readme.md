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
3. Going into the dev console of the webpage on port ```80``` you will notice a comment.
```
<!--Note to self, remember username! Username: R1ckRul3s-->
```
- So remember Username is ```R1ckRul3s``` it will come in useful for ```ssh``` access.

4. We will try to see if there are more directories on the website:
```bash
gobuster dir -u http://10.10.79.232 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

- Results:
```
/assets (Status: 301)
/server-status (Status: 403)
```

- It has not much information. Lets switch our gobuster search:
```bash
gobuster dir -u http://10.10.79.232 -w /usr/share/wordlists/dirb/common.txt
```

- Results:
```
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/assets (Status: 301)
/index.html (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
```

- Interesting and the ```/robots.txt``` contain this ```Wubbalubbadubdub```

- Running ssh on ```R1ckRul3s``` instantly got me denied.
- I tried running ```exiftool <img path>``` on other images as well but didnt return anything useful.

5. After much iteration and researching, I found out that ```nikto``` found that ```http://10.10.79.232``` has a ```/login.php```!!!
```
root@kali:~# nikto -h 10.10.79.232
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.79.232
+ Target Hostname:    10.10.79.232
+ Target Port:        80
+ Start Time:         2020-04-24 13:26:37 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server may leak inodes via ETags, header found with file /, inode: 426, size: 5818ccf125686, mtime: gzip
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST 
+ Cookie PHPSESSID created without the httponly flag
+ OSVDB-3233: /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
+ 7889 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2020-04-24 13:27:24 (GMT0) (47 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

6. We can now login with the previously found username and password. It will bring you to this weird command panel where you can execute command(?).

- Diving into the dev console > Network you will realise ```Post``` didnt do shit to say the least. It just Post nothing.

- There is also another weird comment in the html code:
```
<!-- Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== -->
```

- I tried using SQL injection and XSS but nothing return.

- So just for fun, i tried ```ls``` and...:
```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

7. I tried ```cat Sup3rS3cretPickl3Ingred.txt``` and got trolled so bad, command is disabled.


