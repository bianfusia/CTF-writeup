# Common Linux Scripts references

note that in this script below, I made an assumption that you have assigned ```IP``` to the target's IP address:
```bash
export IP = <ip address>
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

## Sample Hashcat Brute-Forcing
```bash
hashcat -D 1 -m 1800 hash.txt /usr/share/wordlists/rockyou.txt --force
```
## Sample John The Ripper Brute-Forcing
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
