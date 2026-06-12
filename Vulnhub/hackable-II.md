# Hackable II | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Hackable: II](https://www.vulnhub.com/entry/hackable-ii,711/)              |
| Difficulty        | easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             |  Nmap, Gobuster, GTFobins  |


# Reconnaissance
First up find the machine ip address

[arp-scan](../Vulnhub/images/hackable-II/image.png)

Ip address: 10.0.2.25
scan the machine for services and open ports

[nmap scan](../Vulnhub/images/hackable-II/image-1.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |
| 80/tcp    | http          |

FTP allows anonymous login, Lets check it out

# Enumeration

[ftp login](../Vulnhub/images/hackable-II/image-2.png)

found a file CALL.html let’s get the file into kali box and investigate it

[CALL.html](../Vulnhub/images/hackable-II/image-3.png)

maybe a hint but i don’t get it, let’s comaback to this later if somethings links back here.<br>
let’s see the webpage on port 80

[webpage](../Vulnhub/images/hackable-II/image-4.png)

apache default page<br>
Let’s run gobuster and search for directories or files

[Gobuster](../Vulnhub/images/hackable-II/image-5.png)

found direcory /files

[/files](../Vulnhub/images/hackable-II/image-6.png)

same file as earlier seems like the files directory is running on ftp server<br>
we could use this and upload a php-reverse-shell and then run it through the webpage

[reverse-shell.php](../Vulnhub/images/hackable-II/image-7.png)

upload the file into the ftp server using “put” command
make sure to login into ftp inside the directory where the shell file is saved<br>

[FTP reverse-shell.php](../Vulnhub/images/hackable-II/image-8.png)

# Exploitation
start a listener on kali box and execute the exploit

[trigger reverse shell](../Vulnhub/images/hackable-II/image-9.png)

# Initial Access

[reverse shell achieved](../Vulnhub/images/hackable-II/image-10.png)

Got reverse shell
 
Found a txt file “important.txt” along with user directory “shrek”<br>
when opened the txt file it hinted to run a .sh file “.runme.sh”

[./runme.sh](../Vulnhub/images/hackable-II/image-11.png)

Got a md5 hash in return, Let’s crack the hash

[crackstation](../Vulnhub/images/hackable-II/image-12.png)

we now have
username:- shrek<br>
password:- onion<br>
let’s use it and change user

[shrek](../Vulnhub/images/hackable-II/image-13.png)

# Privilege Escalation
checking for sudo permission

[sudo -l](../Vulnhub/images/hackable-II/image-14.png)

Let’s search gtfobins for exploit

[GTFobins](../Vulnhub/images/hackable-II/image-15.png)

Remeber to change the python version here to python3.5

[root access](../Vulnhub/images/hackable-II/image-16.png)

there we have it ROOT access

[User Flag](../Vulnhub/images/hackable-II/image-18.png)

[Root Flag](../Vulnhub/images/hackable-II/image-17.png)

Root flag & User Flag found!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.<br>
Happy Hacking ~!!!
