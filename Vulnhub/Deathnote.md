# Deathnote | Vulnhub walkthrough
* Platform: [Vulnhub](https://www.vulnhub.com/entry/deathnote-1,739/)
* Attack Machine: Kali linux
* Tools: nmap, gobuster, nikto, hydra, brainfuck decoder, cyberchef
---
The room is based on Death Note its an Japanese psychological thriller anime.
lets dive into the machine
# Discovery
Let's Find the machines IP using simple nmap scan<br>
```nmap <ip range>``` <br><br>
<img width="740" height="490" alt="image" src="https://github.com/user-attachments/assets/f856fc21-acfe-41c7-aaae-6e827b5183d0" />

# Reconnaissance
Found the machine IP 10.0.2.13
1. Let’s initiate a nmap scan to get more info on the machine
-A (aggresive scan) -T4 (timing template)
```nmap -A -T4 10.0.2.13``` <br>
<img width="740" height="447" alt="image" src="https://github.com/user-attachments/assets/a4d096c6-ab0c-4f73-9341-37919b138a0a" />
<br>

#### Nmap results
22 ssh, 80 http<br>
2. let’s open the wesite using our browser type http://ip_address/ <br>
(if you face any error then add the ip address in your /etc/hosts file)<br>
<img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/0d0cb57b-6c52-49a6-aed2-801da27f4923" />

<img width="433" height="116" alt="image" src="https://github.com/user-attachments/assets/fe2ff82f-9b5d-4a71-aed6-93ce6cd922f5" />

<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/002a0337-a110-4e3a-b1d8-22936de69403" />
<br><br>
lets head back to the website now,<br>
the page works and loads all the contents smoothly,<br>
At first glance itself we can see HINT written on the page lets check it out<br><br>
<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/7799408e-566b-4fec-a66f-5d148a90031d" />
<br><br>
ha! found something interesting.
<br><br>
<img width="940" height="413" alt="image" src="https://github.com/user-attachments/assets/704e1463-8fff-4fdd-85e9-c06a265622a5" />
<br>

# Enemuration
 
the first hint is about an file notes.txt somewhere in the server<br>
& the second hint is about comment section<br>
I will go ahead and search for the notes.txt file<br>
let’s enumerate the webpage for directories, I will use gobuster tool for this<br>
```gobuster dir -u http://deathnote.vuln/ -w <path to wordlist>``` <br><br>
 
### Gobuster scan<br>
after gobuster does it thing i found robots.txt,<br>
still no notes.txt but robots.txt can be very useful as robots.txt is a text file that instructs automated web bots on how to crawl and/or index a website.<br>
<img width="940" height="297" alt="image" src="https://github.com/user-attachments/assets/123dc7d6-089e-476b-95e0-30078258f4a8" />

!!another hint, This one leads to an jpg image<br>
let’s curl the image and view it<br>
```curl http://deathnote.vuln/important.jpg``` <br><br>
<img width="940" height="302" alt="image" src="https://github.com/user-attachments/assets/13e954d1-d626-4d88-ad14-425944bfbc31" />

Another hint!!, this time it hints towards an file user.txt<br>
mostly an usernames file that we can use to brute force later<br>
both files notes.txt & user.txt are still at large yet to be found.<br>
 
Let’s use nikto to scan the webpage for vulnerabilities, we may get something useful there<br>
```nikto -h http://deathnote.vuln/``` <br><br>
<img width="940" height="333" alt="image" src="https://github.com/user-attachments/assets/0264d5c7-9156-4ebb-996b-df4cd5fa70a9" />

 
### nikto scan
And i did find something useful, nikto found a browsable directory /wordpress/wp-content/uploads <br>
<img width="940" height="487" alt="image" src="https://github.com/user-attachments/assets/f5525419-0cf4-4ab0-b696-aebc467a7e0e" />

<img width="940" height="404" alt="image" src="https://github.com/user-attachments/assets/2cd1e1a3-431f-45e0-b693-ec175bbc4b90" />

<img width="940" height="795" alt="image" src="https://github.com/user-attachments/assets/c804581b-e132-4b37-a3f8-3cf18bcf6d2d" />
<br>

Finally found notes.txt & user.txt files
let’s download and check it out<br>
`wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt` <br>
`wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt`<br><br>

<img width="940" height="269" alt="image" src="https://github.com/user-attachments/assets/25b7f5e4-c588-4fa2-a8e7-feabd6a2f5cd" />
<br>
Turns out it is a file filled with usernames and possible passwords,
so lets use it to brute-force the ssh login

# Exploitation

`hydra -L user.txt -P notes.txt deathnote.vuln ssh` 
<br>
 
### Hydra
username: l <br>
password: death4me

# Initial Access 

`ssh l@deathnote.vuln` <br>
<img width="940" height="231" alt="image" src="https://github.com/user-attachments/assets/c966ba0a-b508-404d-a497-42b1da549492" />
 
### ssh login
YAY! the login was successful, let’s search for the user flag<br>
user.txt file is right out the gate at l’s directory.<br>
its encoded txt, after some browsing found its called brainfuck, also a found many online [decoder](https://brainfuck.net/).<br>
<b> USER FLAG </b><br>
<img width="940" height="284" alt="image" src="https://github.com/user-attachments/assets/0020007a-82ad-4607-9a66-0719b2f4d787" />
<br>

# Privilege Escalation

After roaming around the shell i found /opt directory and the files in it interesting<br>
found two files case.wav & hint<br>
yes i know its another hint, a hex coded data is inside case.wav<br>
as the hint says lets use cyberchef<br>
<img width="940" height="238" alt="image" src="https://github.com/user-attachments/assets/e356f0a6-cd4b-4b61-99e3-268b06c829c5" />
<br>
<img width="940" height="466" alt="image" src="https://github.com/user-attachments/assets/ac324b7b-ce37-4a92-9c2a-ea22d36f0b81" />
<br>

turns out it was hex and base64 encoded<br>
the decryption turns out to be a password: kiraisevil<br>
<img width="940" height="141" alt="image" src="https://github.com/user-attachments/assets/b5d8d03d-d709-4e80-9884-439b2f731025" />
<br> 
got access, its base64 encoded txt<br>
let’s decode and see what is says<br>
<img width="940" height="124" alt="image" src="https://github.com/user-attachments/assets/4943d38e-0200-40da-bf68-55c4b6441791" />
<br>

protect the following ?<br>
both opt and misa didnt turn out useful more of a distraction<br>
when i checked user permissions of kira, Turns out the user has privileges<br>
i tried escalating to super user with the same password<br>
<img width="940" height="215" alt="image" src="https://github.com/user-attachments/assets/74ab0133-670e-435c-b1eb-10c28464ab79" />
<br>

And there i have root access!!!
<img width="940" height="231" alt="image" src="https://github.com/user-attachments/assets/936224ad-a4c0-476f-bed5-fdfcbee93f4a" />

🏁 Root Flag!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!<br>
I Hope this writeup will be interesting and useful to you<br>
Happy Hacking!

