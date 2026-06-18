# Empire-Lupinone | vulnhub | walkthrough

| Category          | Details           |
|---                |---                |
|Platform           | [Empire ~ VulnHub](https://www.vulnhub.com/entry/empire-lupinone,750/)  |
|Difficulty         | Medium            |
|Target Machine     | linux             |
|Attacker Machine   | Kali linux        |
|Tools              | nmap, gobuster, ffuf, john, [GTFobins](https://gtfobins.org/), [Seclists](https://github.com/danielmiessler/SecLists), [cyberchef](https://cyberchef.io/) |

# Reconnaissance
The target machine is identified at 10.0.2.16.
Perform an Nmap scan to enumerate the open ports and running services:<br>
`nmap -A -T4 10.0.2.16`

[Nmap Scan](../Vulnhub/images/Empire/image.png)
<br>
|Port | Service |
|-----|---------|
|22   | ssh     |
|80   | http    |

Browse to the HTTP service to inspect the web application.

[Webpage](../Vulnhub/images/Empire/image-1.png)
<br>

The homepage consists of a simple static image with no immediately obvious functionality.<br>
* Inspecting the page source does not reveal any useful information.
* The `robots.txt` file references an `~myfiles` directory, but attempting to access it returns a `404 Not Found` response.

# Enumeration 
Start by performing directory enumeration with Gobuster.
```
gobuster dir -u http://10.0.2.16/ -w <path to wordlist>
```
[Gobuster](../Vulnhub/images/Empire/image-2.png)

The Gobuster scan does not uncover any immediately useful directories.<br>
Because the application references user-style directories `(~myfiles)`, use `ffuf` to fuzz for additional hidden paths.
```
ffuf -u http://10.0.2.16/~FUZZ -w <path to wordlist>
```
[ffuf scan](../Vulnhub/images/Empire/image-3.png)

The fuzzing process discovers a hidden directory named `~secret`.
```
http://10.0.2.16/~secret
```

[secret directory](../Vulnhub/images/Empire/image-4.png)

The page contains a useful hint that advances the enumeration process.<br>
Based on the hint, icex64 appears to be a valid username, and it strongly suggests that an SSH private key is hidden somewhere on the server.<br>
Continue fuzzing within the discovered directory to search for hidden files, including possible text files.
```
ffuf -u http://10.0.2.16/~secret/.FUZZ -w <path to wordlist> -ic -fc 403,404 -e .txt
```
[ffuf 2nd scan](../Vulnhub/images/Empire/image-5.png)

The scan identifies a hidden file named `.mysecret.txt`. Inspecting its contents reveals a long encoded string.<br>
```
http://10.0.2.16/~secret/.mysecret.txt/
```
The file contains what appears to be an encoded SSH private key. let’s decrypt it<br>
but first need to figure out the encryption<br>

Use a cipher identification tool to determine the encoding format.

[Chiper Identifier](../Vulnhub/images/Empire/image-6.png)

The data is Base58-encoded. Decoding it with CyberChef reveals its original contents.<br>
let’s use cyberchef to decrypt this.

[Cyber Chef](../Vulnhub/images/Empire/image-7.png)

# Exploitation

The decoded output is an encrypted SSH private key. Save it locally as `priv_key.txt`.

[Priv-key.txt](../Vulnhub/images/Empire/image-8.png)

Since the private key is passphrase-protected, recover the passphrase before attempting authentication.<br>
Use `ssh2john` to extract a crackable hash from the private key, then run it through John the Ripper.

[ssh2john](../Vulnhub/images/Empire/image-9.png)

Once the hash has been generated, crack it with John the Ripper.

[John](../Vulnhub/images/Empire/image-10.png)

The recovered credentials are:<br>
username: icex64<br>
password: P@55w0rd!<br>
hash file: priv_key.txt

# Initial Access 
```
chmod 600 priv_key.txt
ssh icex64@10.0.2.16 -i priv_key.txt
```
[ssh Login](../Vulnhub/images/Empire/image-11.png)

Using the recovered passphrase and private key successfully establishes an SSH session as `icex64`.

# Privilege Escalation 

Enumerate the user's `sudo` privileges:

[sudo -l](../Vulnhub/images/Empire/image-13.png)

The permitted files warrant closer inspection.

[note.txt & heist.py](../Vulnhub/images/Empire/image-14.png)

Reviewing `heist.py` shows that it imports Python's built-in `webbrowser` module.<br>
lets check it out `cd /usr/lib/python3.9/webbrowser.py` <br>
searched around internet about webbroswer library and i came across [this article](https://www.hackingarticles.in/linux-privilege-escalation-python-library-hijacking/)

By modifying the imported module to execute arbitrary commands, it is possible to spawn a shell as the target user. 
- open the file and add a line
`os.system("/bin/bash")`

[heist.py](../Vulnhub/images/Empire/image-15.png)

```
sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
```
[user switch](../Vulnhub/images/Empire/image-16.png)

The library hijacking attack successfully provides a shell as the arsene user, but further privilege escalation is still required.<br>
but still not root privilege<br>
Enumerating `arsene`'s sudo permissions reveals another privilege escalation opportunity `sudo -l`

[sudo arsene](../Vulnhub/images/Empire/image-17.png)

The arsene account is permitted to execute pip with elevated privileges.<br>
Consulting GTFOBins shows that `pip` can be abused to obtain a root shell.
```
TF=$(mktemp -d)

echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py

sudo pip install $TF
```
[binary exploit](../Vulnhub/images/Empire/image-18.png)

<b>Executing the GTFOBins payload successfully spawns a root shell.</b>

[User flag](../Vulnhub/images/Empire/image-12.png)<br>
[root flag](../Vulnhub/images/Empire/image-19.png)

Root flag & User Flag found!

I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!