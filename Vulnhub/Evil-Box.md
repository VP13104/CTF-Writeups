# Evil Box 1 | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Evil Box](https://www.vulnhub.com/entry/evilbox-one,736/)              |
| Difficulty        | easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Gobuster, ssh2john, John  |

# Reconnaissance

Begin by identifying the target machine's IP address on the local network.
[nmap scan](../Vulnhub/images/evil-box/image.png)

The target is identified at:- 10.0.2.19<br>
Next, perform an Nmap scan to enumerate open ports and running services.

[nmap scan](../Vulnhub/images/evil-box/image-1.png)

nmap results
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |

Browse to the web service running on port `80` to inspect the application.

[Webpage](../Vulnhub/images/evil-box/image-2.png)

# Enumeration

Perform directory enumeration using Gobuster to discover hidden resources.

[Gobuster](../Vulnhub/images/evil-box/image-3.png)

Gobuster discovers a hidden directory named `/secret`.

[secret](../Vulnhub/images/evil-box/image-4.png)

The directory itself appears blank and does not expose any visible content.<br>
Continue enumeration by searching for PHP files within the /secret directory.
```
gobuster dir -u http://10.0.2.19/secret/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php
```

Gobuster reveals a php file `evil.php`

[Evil.php](../Vulnhub/images/evil-box/image-5.png)

Accessing the discovered PHP file initially produces no visible output.<br>
The behavior suggests that the application may be vulnerable to Local File Inclusion (LFI), so begin testing common file inclusion payloads.<br>
After testing several inputs, requesting `command=/etc/passwd` successfully discloses the system's password file, confirming the LFI vulnerability.<br>
This parameter can be discovered manually or through automated parameter fuzzing, which significantly speeds up the process.

[/etc/passwd](../Vulnhub/images/evil-box/image-6.png)

he exposed `/etc/passwd` file reveals two notable users: `root` and `mowree`.<br>
Since LFI is available, attempt to retrieve the SSH private key for `mowree`.
```
http://10.0.2.19/secret/evil.php?command=/home/mowree/.ssh/id_rsa 
```

[ssh key](../Vulnhub/images/evil-box/image-7.png)

# Exploitation

The LFI vulnerability exposes mowree's SSH private key. Save the retrieved key locally.

[curl ssh key](../Vulnhub/images/evil-box/image-8.png)

[change ssh-key file permissions](../Vulnhub/images/evil-box/image-9.png)

Use ssh2john to convert the encrypted private key into a format compatible with John the Ripper. <br>
[ssh2john](../Vulnhub/images/evil-box/image-10.png)

Run John the Ripper against the extracted hash to recover the passphrase protecting the SSH key.<br>
[john](../Vulnhub/images/evil-box/image-11.png)

John successfully recovers the key's passphrase: `unicorn`

# Initial Access

Use the recovered private key and passphrase to authenticate over SSH as `mowree`.

[ssh login](../Vulnhub/images/evil-box/image-12.png)

Authentication succeeds, providing an interactive shell on the target.

# Privilege Escalation 

Enumerating `sudo` privileges with `sudo -l` does not reveal any obvious escalation paths.<br>
Further filesystem enumeration reveals that the current user has write access to `/etc/passwd`, which presents a direct privilege escalation opportunity.

[passwd](../Vulnhub/images/evil-box/image-13.png)

Because `/etc/passwd` is writable, insert a known password hash for the `root` account, allowing authentication with a password of your choice.

Use openssl
``` 
openssl passwd -1 -salt root passwd123 
# This command generates a password hash that can replace the placeholder (x) in the root entry of /etc/passwd:

root:x:0:0:root:/root:/bin/bash
```

[openssl](../Vulnhub/images/evil-box/image-14.png)

[editing passwd](../Vulnhub/images/evil-box/image-15.png)

Save the modified file, then switch to the `root` user using the password corresponding to the injected hash (`passwd123` in this example).

[root access](../Vulnhub/images/evil-box/image-16.png)

The modified `/etc/passwd` entry grants successful authentication as `root`, resulting in full system compromise.

[user flag](../Vulnhub/images/evil-box/image-17.png)

[root flag](../Vulnhub/images/evil-box/image-18.png)

Root flag & User Flag found!<br>

I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!