# Deathnote | Vulnhub walkthrough
* Platform: [Vulnhub](https://www.vulnhub.com/entry/deathnote-1,739/)
* Attack Machine: Kali linux
* Tools: nmap, gobuster, nikto, hydra, brainfuck decoder, cyberchef
---

# Discovery
Let's Begin by identifying the target machine's IP address with a simple network scan:<br>
```nmap <ip range>``` <br><br>
<img width="740" height="490" alt="image" src="https://github.com/user-attachments/assets/f856fc21-acfe-41c7-aaae-6e827b5183d0" />

# Reconnaissance
The scan identifies the target machine at 10.0.2.13.
1. Next, perform an aggressive Nmap scan to enumerate open ports and services:
-A (aggresive scan) -T4 (timing template)
```nmap -A -T4 10.0.2.13``` <br>
<img width="740" height="447" alt="image" src="https://github.com/user-attachments/assets/a4d096c6-ab0c-4f73-9341-37919b138a0a" />
<br>

#### Nmap results
22 ssh, 80 http<br>
2. Browse to the web service at `http://<target-ip>/` to inspect the application.<br>
(if you face any error then add the ip address in your /etc/hosts file)<br>
<img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/0d0cb57b-6c52-49a6-aed2-801da27f4923" />

<img width="433" height="116" alt="image" src="https://github.com/user-attachments/assets/fe2ff82f-9b5d-4a71-aed6-93ce6cd922f5" />

<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/002a0337-a110-4e3a-b1d8-22936de69403" />
<br><br>
lets head back to the website now,<br>
the page works and loads all the contents smoothly,<br>
The homepage prominently displays a HINT section, making it a good starting point for further enumeration.<br><br>
<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/7799408e-566b-4fec-a66f-5d148a90031d" />
<br><br>
The hint reveals useful information that points toward additional files on the server.
<br><br>
<img width="940" height="413" alt="image" src="https://github.com/user-attachments/assets/704e1463-8fff-4fdd-85e9-c06a265622a5" />
<br>

# Enemuration
 
The first hint references a file named `notes.txt` somewhere on the server, while the second suggests inspecting the page comments.<br>
To locate these files, perform directory enumeration with Gobuster.<br>
let’s enumerate the webpage for directories, I will use gobuster tool for this<br>
```gobuster dir -u http://deathnote.vuln/ -w <path to wordlist>``` <br><br>
 
### Gobuster scan<br>
Gobuster discovers a `robots.txt` file, which often contains interesting paths or hidden resources.<br>
still no notes.txt but robots.txt can be very useful as robots.txt is a text file that instructs automated web bots on how to crawl and/or index a website.<br>
<img width="940" height="297" alt="image" src="https://github.com/user-attachments/assets/123dc7d6-089e-476b-95e0-30078258f4a8" />

`robots.tx`t reveals another clue that points to an image file named `important.jpg`.<br>
let’s curl the image and view it<br>
```curl http://deathnote.vuln/important.jpg``` <br><br>
<img width="940" height="302" alt="image" src="https://github.com/user-attachments/assets/13e954d1-d626-4d88-ad14-425944bfbc31" />

Inspecting the image provides another clue, this time referencing a file named `user.txt`.<br>
Based on the context, `user.txt` is likely to contain usernames that could be useful for credential attacks.<br>
At this stage, neither `notes.txt` nor `user.txt` has been located.<br>
 
Run a Nikto scan to identify misconfigurations or exposed resources on the web server.<br>
```nikto -h http://deathnote.vuln/``` <br><br>
<img width="940" height="333" alt="image" src="https://github.com/user-attachments/assets/0264d5c7-9156-4ebb-996b-df4cd5fa70a9" />

 
### nikto scan
Nikto identifies a browsable directory at `/wordpress/wp-content/uploads`, providing access to publicly accessible files.<br>
<img width="940" height="487" alt="image" src="https://github.com/user-attachments/assets/f5525419-0cf4-4ab0-b696-aebc467a7e0e" />

<img width="940" height="404" alt="image" src="https://github.com/user-attachments/assets/2cd1e1a3-431f-45e0-b693-ec175bbc4b90" />

<img width="940" height="795" alt="image" src="https://github.com/user-attachments/assets/c804581b-e132-4b37-a3f8-3cf18bcf6d2d" />
<br>

Within the uploads directory, both notes.txt and user.txt are discovered.
let’s download and check it out<br>
`wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt` <br>
`wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt`<br><br>

<img width="940" height="269" alt="image" src="https://github.com/user-attachments/assets/25b7f5e4-c588-4fa2-a8e7-feabd6a2f5cd" />
<br>
The downloaded files contain a list of usernames and candidate passwords, making them ideal for an SSH brute-force attack using Hydra.

# Exploitation

`hydra -L user.txt -P notes.txt deathnote.vuln ssh` 
<br>
 
Hydra successfully recovers the following valid SSH credentials:
username: l <br>
password: death4me

# Initial Access 

`ssh l@deathnote.vuln` <br>
<img width="940" height="231" alt="image" src="https://github.com/user-attachments/assets/c966ba0a-b508-404d-a497-42b1da549492" />
 
### ssh login
The recovered credentials provide successful SSH access to the target.<br>
After logging in, enumerate the user's home directory to locate the user flag.<br>
user.txt file is right out the gate at l’s directory.<br>
The `user.txt` file is encoded using the Brainfuck esoteric programming language. Decoding it with an online Brainfuck interpreter reveals the user flag. [decoder](https://brainfuck.net/).<br>
<b> USER FLAG </b><br>
<img width="940" height="284" alt="image" src="https://github.com/user-attachments/assets/0020007a-82ad-4607-9a66-0719b2f4d787" />
<br>

# Privilege Escalation

Further filesystem enumeration reveals several interesting files under the /opt directory.<br>
Two files stand out: `case.wav` and `hint`, both of which appear relevant to the challenge.<br>
yes i know its another hint, Following the guidance in the hint file, inspect `case.wav`. It contains encoded data that can be decoded using CyberChef.<br>
<img width="940" height="238" alt="image" src="https://github.com/user-attachments/assets/e356f0a6-cd4b-4b61-99e3-268b06c829c5" />
<br>
<img width="940" height="466" alt="image" src="https://github.com/user-attachments/assets/ac324b7b-ce37-4a92-9c2a-ea22d36f0b81" />
<br>

Decoding the contents reveals multiple encoding layers (Hex followed by Base64), ultimately producing the `password:kiraisevil`<br>
<img width="940" height="141" alt="image" src="https://github.com/user-attachments/assets/b5d8d03d-d709-4e80-9884-439b2f731025" />
<br> 
Using the recovered password grants access to another encoded file. Decoding the Base64 content reveals an additional clue.<br>
<img width="940" height="124" alt="image" src="https://github.com/user-attachments/assets/4943d38e-0200-40da-bf68-55c4b6441791" />
<br>

protect the following ?<br>
The references to misa and the /opt files ultimately serve as distractions and do not directly contribute to privilege escalation.<br>
Enumerating sudo privileges for the kira account reveals elevated permissions.<br>
Using the recovered password with sudo successfully grants root privileges.<br>
<img width="940" height="215" alt="image" src="https://github.com/user-attachments/assets/74ab0133-670e-435c-b1eb-10c28464ab79" />
<br>

At this point, full root access to the system is obtained.

<img width="940" height="231" alt="image" src="https://github.com/user-attachments/assets/936224ad-a4c0-476f-bed5-fdfcbee93f4a" />

The root flag can now be retrieved, completing the machine.<br>

I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!