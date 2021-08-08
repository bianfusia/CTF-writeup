# Common CTF References

## THM box setup
tryhackme virtual box do not have pip installed.
```
sudo apt install python3-pip
```

install gobuster as well
```
sudo apt-get install gobuster
```

## IP setup

note that in this script below, I made an assumption that you have assigned ```IP``` to the target's IP address:
```bash
export IP=<ip address>
```

## Nmap
Normal full port scan:
```bash
nmap -sC -sV -p- -oN fullscan $IP
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

## Telnet
Accessing Telnet:
```bash
telnet $IP [port]
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

## Recommended Burpsuite Additional Module Copy As Python-Requests
1. Go to ```Extender``` Tab
2. Under ```BApp Store``` Tab
3. Search for ```Copy As Python-Requests```
4. This allows you to change the get or post requests into a python format when you right click on the ```intercept``` response and choose ```Copy As Python-Requests```
5. Good for python scripting.
