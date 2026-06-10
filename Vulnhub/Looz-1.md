# Looz 1 | vulnhub | walkthrough

| Category          | Details       |
|-------------------|---------------|
| Platform          | [looz](https://www.vulnhub.com/entry/looz-1,732/)|
| Difficulty        | easy          |
| Target machine    | Ubuntu        |
| Attacker Machine  | Kali linux    |
| Tools             | nmap, Gobuster, ffuf, hydra |

# Reconnaissance

Found IP:- 10.0.2.18<br>
let’s scan the machines for ports and services<br>
`nmap -A -T4 <ip address>`

[Nmap result](../Vulnhub/images/Looz/image.png)

nmap results show:<br>
| Port      | Service       |
|-----------|---------------|
| 22        | ssh           |
| 80        | http          |
| 8081      | http          |
| 3306      | mysql         |

Let’s check out the webservers that are running

[Webpage](../Vulnhub/images/Looz/image-1.png)

# Enumeration

Let’s enumerate for directories using gobuster

[Gobuster](../Vulnhub/images/Looz/image-2.png)

didn’t return anything useful<br>
Let’s check the source code

[Source Code](../Vulnhub/images/Looz/image-3.png)

Well well well found something good<br>
Username: john<br>
password: y0uC@n’tbr3akIT<br>
turns out these credentials are used to login to wordpress but the login page returns an `404 error`<br>
it's possible the word press is on another port or sub-domain 

[sub-domain scan](../Vulnhub/images/Looz/image-4.png)

ffuf returns a sub-domain, add both domain to `/etc/hosts`<br>
looz.com & wp.looz.com

dirb on the new domain 

[dirb scan](../Vulnhub/images/Looz/image-5.png)

Got the login page

[Wordpress login page](../Vulnhub/images/Looz/image-6.png)

Let's use the credentials we found earlier.

[Wordpress Login](../Vulnhub/images/Looz/image-7.png)

we are in!!!<br>
after some searching around i found usernames and the privilege they have

[User list](../Vulnhub/images/Looz/image-8.png)

Let’s use Hydra to crack “gandalf” password

# Exploitation 

```
hydra -l gandalf -p <path to wordlist> ip-address ssh
```
[Hydra](../Vulnhub/images/Looz/image-9.png)

Got the password: `highschoolmusical` 😅<br>
Let’s go ahead and login into ssh using these credentials

[ssh Login](../Vulnhub/images/Looz/image-10.png)

ssh Login Succesfull 

# Privilege Escalation 

Found gandalf is not allowed to run sudo, so we move towards SUID binaries
```
find / -prem -4000 -type f -exec ls -al {}\;2>/dev/null
```

[SUID files](../Vulnhub/images/Looz/image-11.png)

Found a interesting file shell_testv1.0<br>
lets execute this file

[shell_testv1.0](../Vulnhub/images/Looz/image-12.png)

There you have it ROOT access. that was quick 

[root flag](../Vulnhub/images/Looz/image-13.png)
[user flag](../Vulnhub/images/Looz/image-14.png)

Found user & root flag !!!!!!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.<br>
Happy Hacking ~!!!
