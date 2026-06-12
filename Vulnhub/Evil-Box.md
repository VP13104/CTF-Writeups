# Evil Box 1 | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Evil Box](https://www.vulnhub.com/entry/evilbox-one,736/)              |
| Difficulty        | easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Gobuster, ssh2john, John  |

# Reconnaissance

Let’s find the ip of our target machine

[nmap scan](../Vulnhub/images/evil-box/image.png)

Found IP:- 10.0.2.19<br>
let’s scan the machines for ports and services

[nmap scan](../Vulnhub/images/evil-box/image-1.png)

nmap results
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |

Let’s check out the webpage

[Webpage](../Vulnhub/images/evil-box/image-2.png)

# Enumeration

Let’s enumerate for directories using gobuster

[Gobuster](../Vulnhub/images/evil-box/image-3.png)

Found directory `/secret`

[secret](../Vulnhub/images/evil-box/image-4.png)

Just a black Directory<br>
Let’s dig deeper for PHP files
```
gobuster dir -u http://10.0.2.19/secret/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php
```

Gobuster reveals a php file `evili.php`

[Evil.php](../Vulnhub/images/evil-box/image-5.png)

Again a blank directory<br>
I think this is hinting towards an LFI vulnerability, so let's try some basic actions<br>
finally ended up at `command=/etc/passwd`
You can go manually or try parameter fuzzing which is a lot faster 😅

[/etc/passwd](../Vulnhub/images/evil-box/image-6.png)

Found users : root & mowree<br>
let’s go ahead a search for ssh key
```
http://10.0.2.19/secret/evil.php?command=/home/mowree/.ssh/id_rsa 
```

[ssh key](../Vulnhub/images/evil-box/image-7.png)

# Exploitation

Found the key for mowree, save it into a file

[curl ssh key](../Vulnhub/images/evil-box/image-8.png)

[change ssh-key file permissions](../Vulnhub/images/evil-box/image-9.png)

convert the key file into hash file <br>
[ssh2john](../Vulnhub/images/evil-box/image-10.png)

use john to crack hash file<br>
[john](../Vulnhub/images/evil-box/image-11.png)

password cracked!! `unicorn`

# Initial Access

let’s login via ssh

[ssh login](../Vulnhub/images/evil-box/image-12.png)

GOT in!!!

# Privilege Escalation 

Let's check sudo permissions the user has `sudo -l` nothing useful.<br>
After some snooping around i discover that the user has write permission on passwd file.

[passwd](../Vulnhub/images/evil-box/image-13.png)

Since we have write access we can edit the passwords hash into this file and gain root access

Use openssl
``` 
openssl passwd -1 -salt root passwd123 
# this will provide a password hash which we will use to replace the ‘x’ in

root:x:0:0:root:/root:/bin/bash
```

[openssl](../Vulnhub/images/evil-box/image-14.png)

[editing passwd](../Vulnhub/images/evil-box/image-15.png)

save it & change user to root using the password just used `passwd123`

[root access](../Vulnhub/images/evil-box/image-16.png)

And there you have it root access !!!!!!!

[user flag](../Vulnhub/images/evil-box/image-17.png)

[root flag](../Vulnhub/images/evil-box/image-18.png)

Root flag & User Flag found!<br>

Hope this Walkthrough was fun, easy to follow and helpful to you.<br>
Happy Hacking ~!!!
