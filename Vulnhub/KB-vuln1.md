# KB-VULN 1 | Vuln-Hub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [KB-Vuln 1](https://www.vulnhub.com/entry/kb-vuln-1,540/)    |
| Difficulty        | Easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, hydra |

# Reconnaissance

Identify the target machine on the local network.
```
sudo arp-scan <ip range>
```

[arp-scan](../Vulnhub/images/KB-vuln1/arp-scan.png)

Once the target IP was identified, perform an nmap scan to enumerate the available services<br>
`nmap -A -T4 <ip address>`

[nmap scan](../Vulnhub/images/KB-vuln1/nmap-scan.png)

| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |
| 80/tcp    | http          |

# Enumeration 

### FTP

Anonymous Access allowed to FTP server<br>
only one file was found `.bash_history`, While this file is no use with the initial compromise, it contained useful commands that the user had previously executed. which may come in handy down the road.

[ftp](../Vulnhub/images/KB-vuln1/ftp.png)

[bash-history](../Vulnhub/images/KB-vuln1/bash-history.png)

### HTTP

Browsing to `http://target-IP:3000/` presents a static page titled "one school"<br>
Viewing the HTML source of the website revealed a hidden developer comment: a valid username 
`<!-- Username : sysadmin -->`

[source-code](../Vulnhub/images/KB-vuln1/source-code.png)

# Exploitation 

With the valid username, perform brute-force attack against the target usgin hydra 
```
hydra -l sysadmin -P <path-to-wordlist> ssh://<ip address>
```
Hydra successfully recovered the user's password.

[hydra](../Vulnhub/images/KB-vuln1/hydra.png)

# Initial Access

Using the discovered credentials login to targer via ssh

[ssh login](../Vulnhub/images/KB-vuln1/initial-access.png)

user flag located at `/home/sysadmin`

# Privilege Escalation 

During the initial FTP enumeration, the .bash_history file referenced the following directory:
```
/etc/update-motd.d/
```

Revisiting this directory after obtaining a shell revealed the script:
```
/etc/update-motd.d/00-header
```

Checking its permissions showed that the file was writable by the sysadmin user.
```
ls -l /etc/update-motd.d/00-header
```

[00-header](../Vulnhub/images/KB-vuln1/00-header.png)

Since the script belongs to Ubuntu's Dynamic MOTD (Message of the Day) framework, it is executed as root whenever a user logs in through SSH.

This meant that any commands appended to the script would eventually be executed with root privileges.

### Exploitation

Append a paylod to this script 
```
echo 'cp /bin/bash /tmp/rootbash && chmod u+s /tmp/rootbash' >> /etc/update-motd.d/00-header
```
[payload](../Vulnhub/images/KB-vuln1/payload.png)

This payload performs two actions:
- Copies `/bin/bash` to `/tmp/rootbash`.
- Sets the SUID permission on the copied binary.

The modified script was executed automatically during the next SSH login, creating a root-owned SUID Bash binary.<br>
Exit the ssh and log back in again <br>
```
ls -l /tmp/rootbash

output
-rwsr-xr-x 1 root root ... /tmp/rootbash
```

The `s` in the owner's permission field indicates that the Set User ID (SUID) bit is enabled.

run `/tmp/rootbash -p`
Root access gained 

[root access](../Vulnhub/images/KB-vuln1/root-access.png)

Linux binaries normally execute with the permissions of the user running them.

However, when a binary has the `SUID` permission set and is owned by root, it executes with the `effective user ID (EUID)` of the file owner instead of the current user.

Executing the newly created binary with the `-p` option

The `-p` option is important because modern versions of Bash drop elevated privileges by default when executed as a SUID binary. Using -p prevents Bash from discarding the root effective UID.

Once executed, the shell inherited:

Real UID: `sysadmin`
Effective UID: `root`

This resulted in a fully privileged root shell.

[user flag](../Vulnhub/images/KB-vuln1/user-flag.png)

[root flag](../Vulnhub/images/KB-vuln1/root-flag.png)

Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!