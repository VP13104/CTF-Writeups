# Empire-Lupinone | vulnhub | walkthrough

| Category          | Details           |
|---                |---                |
|Platform           | [Empire ~ VulnHub](https://www.vulnhub.com/entry/empire-lupinone,750/)  |
|Difficulty         | Medium            |
|Target Machine     | linux             |
|Attacker Machine   | Kali linux        |
|Tools              | nmap, gobuster, ffuf, john, [GTFobins](https://gtfobins.org/), [Seclists](https://github.com/danielmiessler/SecLists), [cyberchef](https://cyberchef.io/) |

# Reconnaissance
Found IP as `10.0.2.16`
let’s scan the machine for running services and ports<br>
`nmap -A -T4 10.0.2.16`

[Nmap Scan](../Vulnhub/images/Empire/image.png)
<br>
|Port | Service |
|-----|---------|
|22   | ssh     |
|80   | http    |

Let's check out the webpage

[Webpage](../Vulnhub/images/Empire/image-1.png)
<br>

A static page with just a image displayed<br>
* checked out the source code didnt yield to anything useful.
* `robots.txt` lead to an `~myfiles` directory that yeiled an 404 error

# Enumeration 
Let's go ahead with a directory scan 
```
gobuster dir -u http://10.0.2.16/ -w <path to wordlist>
```
[Gobuster](../Vulnhub/images/Empire/image-2.png)

directory search also didnt show up with anything interesting.<br>
since there is a mention of `~myfiles` i am hoping to find something more by fuzzing
```
ffuf -u http://10.0.2.16/~FUZZ -w <path to wordlist>
```
[ffuf scan](../Vulnhub/images/Empire/image-3.png)

A directory called `secret` lets check it out
```
http://10.0.2.16/~secret
```

[secret directory](../Vulnhub/images/Empire/image-4.png)

A hint, this is good<br>
from the hint we can determine that `icex64` is a username and there is a ssh key file somewhere waiting to be found.
let’s fuzz some more for the ssh file, which is probably an `.txt` file
```
ffuf -u http://10.0.2.16/~secret/.FUZZ -w <path to wordlist> -ic -fc 403,404 -e .txt
```
[ffuf 2nd scan](../Vulnhub/images/Empire/image-5.png)

found `.mysecret.txt file`, lets check it out<br>
```
http://10.0.2.16/~secret/.mysecret.txt/
```
and inside it is a long encrypted text probably the ssh_key we are looking for. let’s decrypt it
but first need to figure out the encryption<br>

let’s use an cipher identifier to figure this out

[Chiper Identifier](../Vulnhub/images/Empire/image-6.png)

it’s base58!<br>
let’s use cyberchef to decrypt this.

[Cyber Chef](../Vulnhub/images/Empire/image-7.png)

# Exploitation

The decrypted data is a ssh private key, save it into a txt file

[Priv-key.txt](../Vulnhub/images/Empire/image-8.png)

Now to crack the password to ssh<br>
we need to convert `priv-key.txt` into an hash file then run it through john

let’s use ssh2john to convert

[ssh2john](../Vulnhub/images/Empire/image-9.png)

now we have the hash file ssh_key lets run it through john

[John](../Vulnhub/images/Empire/image-10.png)

lets try and login with ssh using

username: icex64<br>
password: P@55w0rd!<br>
hash file: priv_key.txt

# Initial Access 
```
chmod 600 priv_key.txt
ssh icex64@10.0.2.16 -i priv_key.txt
```
[ssh Login](../Vulnhub/images/Empire/image-11.png)

Login successfull!!!!!!

# Privilege Escalation 

run sudo -l to check for user permissions

[sudo](../Vulnhub/images/Empire/image-13.png)

lets check out these files

[note.txt & heist.py](../Vulnhub/images/Empire/image-14.png)

looks like heist.py is calling an python library webbrowser<br>
lets check it out `cd /usr/lib/python3.9/webbrowser.py` <br>
searched around internet about webbroswer library and i came across [this article](https://www.hackingarticles.in/linux-privilege-escalation-python-library-hijacking/)

let’s levrage this and switch users 
- open the file and add a line
`os.system("/bin/bash")`

[heist.py](../Vulnhub/images/Empire/image-15.png)

```
sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
```
[user switch](../Vulnhub/images/Empire/image-16.png)

Yess!! user switched<br>
but still not root privilege<br>
So, lets continiue to dig `sudo -l`

[sudo arsene](../Vulnhub/images/Empire/image-17.png)

user has root permission on pip binary.<br>
let’s check out GTFobins for ways to escalate this to root access
```
TF=$(mktemp -d)

echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py

sudo pip install $TF
```
[binary exploit](../Vulnhub/images/Empire/image-18.png)

<b>yay!! finally we have root access</b>

[User flag](../Vulnhub/images/Empire/image-12.png)<br>
[root flag](../Vulnhub/images/Empire/image-19.png)

Root flag & User Flag found!

Hope this Walkthrough was fun, easy to follow and helpful to you.<br>

Happy Hacking ~!!!