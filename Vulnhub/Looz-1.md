# Looz 1 | vulnhub | walkthrough

| Category          | Details       |
|-------------------|---------------|
| Platform          | [looz](https://www.vulnhub.com/entry/looz-1,732/)|
| Difficulty        | easy          |
| Target machine    | Ubuntu        |
| Attacker Machine  | Kali linux    |
| Tools             | nmap, Gobuster, ffuf, hydra |

# Reconnaissance

IP:- 10.0.2.18<br>
Perform an Nmap scan to enumerate the open ports and running services:<br>
`nmap -A -T4 <ip address>`

[Nmap result](../Vulnhub/images/Looz/image.png)

nmap results show:<br>
| Port      | Service       |
|-----------|---------------|
| 22        | ssh           |
| 80        | http          |
| 8081      | http          |
| 3306      | mysql         |

Browse to the exposed web services to inspect the applications running on the target.

[Webpage](../Vulnhub/images/Looz/image-1.png)

# Enumeration
Start by performing directory enumeration with Gobuster.

[Gobuster](../Vulnhub/images/Looz/image-2.png)

The Gobuster scan does not reveal any interesting directories or files.<br>
Since directory enumeration yields little information, inspect the page source for hidden clues.

[Source Code](../Vulnhub/images/Looz/image-3.png)

The source code contains what appears to be a set of credentials:<br>
- Username: john
- Password: y0uC@n’tbr3akIT<br>
The credentials appear to be intended for a WordPress instance, but the expected login page returns a `404 Not Found error`.<br>
This suggests that the WordPress installation may be hosted on a different virtual host or subdomain.

[sub-domain scan](../Vulnhub/images/Looz/image-4.png)

Fuzzing identifies an additional virtual host. Add both domains to your `/etc/hosts` file:
- looz.com
- wp.looz.com

Perform directory enumeration against the newly discovered virtual host.

[dirb scan](../Vulnhub/images/Looz/image-5.png)

The scan reveals the WordPress login page.

[Wordpress login page](../Vulnhub/images/Looz/image-6.png)

Authenticate using the credentials recovered from the page source.

[Wordpress Login](../Vulnhub/images/Looz/image-7.png)

The login succeeds, providing access to the WordPress administrative interface.<br>
Enumerating the WordPress users reveals multiple accounts along with their associated roles and privileges.

[User list](../Vulnhub/images/Looz/image-8.png)

With a valid username identified, use Hydra to perform an SSH password attack against the `gandalf` account.

# Exploitation 

```
hydra -l gandalf -p <path to wordlist> ip-address ssh
```
[Hydra](../Vulnhub/images/Looz/image-9.png)

Hydra successfully recovers the following credentials:
- Username: gandalf
- Password: highschoolmusical
Use the recovered credentials to establish an SSH session as `gandalf`.

[ssh Login](../Vulnhub/images/Looz/image-10.png)

SSH authentication succeeds, providing an interactive shell on the target.

# Privilege Escalation 

he gandalf account has no useful sudo privileges, so enumerate SUID binaries for alternative privilege escalation paths.
```
find / -prem -4000 -type f -exec ls -al {}\;2>/dev/null
```

[SUID files](../Vulnhub/images/Looz/image-11.png)

Among the discovered SUID binaries, `shell_testv1.0` stands out as unusual and warrants further investigation.<br>
Executing the binary immediately results in elevated privileges.

[shell_testv1.0](../Vulnhub/images/Looz/image-12.png)

The SUID binary spawns a root shell, providing full administrative access to the system. 

[root flag](../Vulnhub/images/Looz/image-13.png)
[user flag](../Vulnhub/images/Looz/image-14.png)

Found user & root flag !!!!!!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!!