# Grotesque 1 | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Grotesque  1](https://www.vulnhub.com/entry/grotesque-101,658/)              |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Gobuster, john the ripper, Keeweb  |


# Reconnaissance
Let’s find the IP address of the machine

[arp-scan](../Vulnhub/images/grotesque-1/image.png)

IP:- 10.0.2.22
let’s scan the machines for ports and services

[nmap scan](../Vulnhub/images/grotesque-1/image-1.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 66/tcp    | http          |
| 80/tcp    | http          |

Let’s check out the webpage

[webpage port 66](../Vulnhub/images/grotesque-1/image-2.png)

[webpage port 80](../Vulnhub/images/grotesque-1/image-3.png)

# Enumeration

Enumeration for directories using gobuster for both ports 66 and 80 didnt yield anything useful.

[Gobuster ](../Vulnhub/images/grotesque-1/image-4.png)

The webpage at port 66 reveals a zip file to download the whole project, lets analyze the files

[Zip file](../Vulnhub/images/grotesque-1/image-5.png)

[zip file contents](../Vulnhub/images/grotesque-1/image-6.png)

Unzip the file<br>
- analyze _vvmlist directory
- cat _vvmlist/* | sort | uniq

[_vvmlist](../Vulnhub/images/grotesque-1/image-7.png)

Found a WordPress page on port `80 /lyricsblog` <br>
let’s check it out.

[wordpress](../Vulnhub/images/grotesque-1/image-8.png)

A simple page with song lyrics.<br>
enumerate the directories using Gobuster

[gobuster](../Vulnhub/images/grotesque-1/image-9.png)

`http://ip address/lyricsblog/wp-admin` <br>
its redirecting to a login page wp-login.php

[login page](../Vulnhub/images/grotesque-1/image-10.png)

To find usernames of a wordpress webpage best use wpscan
```
wpscan --url http://10.0.2.22/lyricsblog -e at -e ap -e u
```

[wpscan](../Vulnhub/images/grotesque-1/image-11.png)

There you have it.
Username: erdalkomurcu<br>
tried brute force attack to crack the password but no payloads worked. Just a hunch since it was a lyrics page what if one of the lyrics is the password. 

[lyric](../Vulnhub/images/grotesque-1/image-12.png)

copy the lyrics and paste into a .txt file remove all the space except the ones between the paragraphs.<br>
-> convert it md5 & then convert it into uppercase

[password](../Vulnhub/images/grotesque-1/image-13.png)

username: erdalkomurcu<br>
password: BC78C6AB38E114D6135409E44F7CDDA2

# Exploitation

[wordpress login](../Vulnhub/images/grotesque-1/image-14.png)

Login successful.<br>
Now let's create a reverse shell using the wordpress site<br> 
`Appearance -> Theme File editor -> archive.php` <br>
Here upload an php-reverse shell from [pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

[reverse-shell.php](../Vulnhub/images/grotesque-1/image-15.png)

start a listener
```
nc -lvp 4444
```
Now to run the reverse shell visit:-
```
http://<target-ip>/lyricsblog/wp-content/themes/twentytwentyone/archive.php
```

[archive.php](../Vulnhub/images/grotesque-1/image-16.png)

[shell](../Vulnhub/images/grotesque-1/image-17.png)

Reverse shell achieved. stabalize the shell using 
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

# Privilege Escalation
Let’s move up our privileges. after some snooping around found a interesting file at `/var/www/html/lyricsblog/wp-config.php`

[wp-config.php](../Vulnhub/images/grotesque-1/image-18.png)

Found username and password.<br>
`raphael & _double_trouble_ ` <br>
change user using these credentials

[su raphael](../Vulnhub/images/grotesque-1/image-19.png)

## Root Access

checking contents of current directory `ls -la `

[ls -la](../Vulnhub/images/grotesque-1/image-20.png)

Found a hidden keepass database file, move it to your attack box<br>
First we need to crack the password for that we need to convert `.chadroot.kbdx` into hash file and then run it through john

[crack password using john](../Vulnhub/images/grotesque-1/image-21.png)

Found the password, lets use it and open the database using [keeweb](https://app.keeweb.info/)

[keeweb](../Vulnhub/images/grotesque-1/image-22.png)

got four passwords try them all until you gain root access

[passwords](../Vulnhub/images/grotesque-1/image-23.png)

There you have it root access with password `.:.subjective.:.`

[user flag](../Vulnhub/images/grotesque-1/image-25.png)

[root flag](../Vulnhub/images/grotesque-1/image-24.png)

Root flag & User Flag found!

Hope this Walkthrough was fun, easy to follow and helpful to you.<br>
Happy Hacking ~!!!

