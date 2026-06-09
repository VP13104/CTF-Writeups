# Deathnote | Vulnhub walkthrough
* Platform: Vulnhub
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
nmap output<br>
from the results of the scan, we discover ports<br>
22 ssh, 80 http<br>
2. let’s open the website using our browser type http://ip_address/ <br>
(if you face any error then add the ip address in your /etc/hosts file)<br>
<img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/0d0cb57b-6c52-49a6-aed2-801da27f4923" />

 <img width="433" height="116" alt="image" src="https://github.com/user-attachments/assets/fe2ff82f-9b5d-4a71-aed6-93ce6cd922f5" />

<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/002a0337-a110-4e3a-b1d8-22936de69403" />
<br>
lets head back to the website now,<br>
the page works and loads all the contents smoothly,<br>
At first glance itself we can see HINT written on the page lets check it out<br>
<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/7799408e-566b-4fec-a66f-5d148a90031d" />
<br><br>
ha! found something interesting.
<br><br>
<img width="940" height="413" alt="image" src="https://github.com/user-attachments/assets/704e1463-8fff-4fdd-85e9-c06a265622a5" />
<br><br>
 
the first hint is about an file notes.txt somewhere in the server<br>
& the second hint is about comment section<br>
I will go ahead and search for the notes.txt file<br>
let’s enumerate the webpage for directories, I will use gobuster tool for this<br>
```gobuster dir -u http://deathnote.vuln/ -w <path to wordlist>``` <br><br>
 
Gobuster scan<br>
after gobuster does it thing i found robots.txt,<br>
still no notes.txt but robots.txt can be very useful as robots.txt is a text file that instructs automated web bots on how to crawl and/or index a website.
 
robots.txt
!!another hint, This one leads to an jpg image
let’s curl the image and view it
curl http://deathnote.vuln/important.jpg
Press enter or click to view image in full size
 
img file
Another hint!!, this time it hints towards an file user.txt
mostly an usernames file that we can use to brute force later
both files notes.txt & user.txt are still at large yet to be found.
 
Let’s use nikto to scan the webpage for vulnerabilities, we may get something useful there
nikto -h http://deathnote.vuln/
Press enter or click to view image in full size
 
nikto scan
And i did find something useful, nikto found a browsable directory /wordpress/wp-content/uploads
Press enter or click to view image in full size
 
Press enter or click to view image in full size
 
Press enter or click to view image in full size
 
Finally found notes.txt & user.txt files
let’s download and check it out
wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt
wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt
Press enter or click to view image in full size
 
Turns out it is a file filled with usernames and possible passwords,
so lets use it to brute-force the ssh login
i’ll use hydra here,
hydra -L user.txt -P notes.txt deathnote.vuln ssh
Press enter or click to view image in full size
 
Hydra
username: l
password: death4me
let’s login to ssh
ssh l@deathnote.vuln
Press enter or click to view image in full size
 
ssh login
YAY! the login was successful, let’s search for the user flag
user.txt file is right out the gate at l’s directory.
its encoded txt, after some browsing found its called brainfuck, also a found many online decoders
Press enter or click to view image in full size
 
user flag
After roaming around the shell i found /opt directory and the files in it interesting
found two files case.wav & hint
yes i know its another hint, a hex coded data is inside case.wav
as the hint says lets use cyberchef
Press enter or click to view image in full size
 
Press enter or click to view image in full size
 
turns out it was hex and base64 encoded
the decryption turns out to be a password: kiraisevil
Press enter or click to view image in full size
 
got access, its base64 encoded txt
let’s decode and see what is says
Press enter or click to view image in full size
 
protect the following ?
both opt and misa didnt turn out useful more of a distraction
when i checked user permissions of kira, Turns out the user has privileges
i tried escalating to super user with the same password
Press enter or click to view image in full size
 
And there i have root access!!!
Press enter or click to view image in full size
 
Root Flag!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
I Hope this writeup will be interesting and useful to you
Happy Hacking!

