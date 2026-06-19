# Hackable II | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Hackable: II](https://www.vulnhub.com/entry/hackable-ii,711/)              |
| Difficulty        | easy                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             |  Nmap, Gobuster, GTFobins  |


# Reconnaissance
Begin by identifying the target machine's IP address on the local network.

[arp-scan](../Vulnhub/images/hackable-II/image.png)

Ip address: 10.0.2.25
perform an Nmap scan to enumerate open ports and running services.

[nmap scan](../Vulnhub/images/hackable-II/image-1.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |
| 80/tcp    | http          |

The Nmap scan indicates that the FTP service allows anonymous authentication, making it a good starting point for enumeration.

# Enumeration

[ftp login](../Vulnhub/images/hackable-II/image-2.png)

Anonymous FTP access exposes a file named `CALL.html`. Download it to the attacker machine for further analysis.

[CALL.html](../Vulnhub/images/hackable-II/image-3.png)

The file appears to contain a hint, but it does not immediately reveal its purpose. Keep it in mind and continue enumerating the target.<br>
let’s see the webpage on port 80

[webpage](../Vulnhub/images/hackable-II/image-4.png)

The web server displays the default Apache landing page with no obvious functionality.<br>
Perform directory enumeration with Gobuster to discover hidden resources.

[Gobuster](../Vulnhub/images/hackable-II/image-5.png)

Gobuster discovers a `/files` directory.

[/files](../Vulnhub/images/hackable-II/image-6.png)

The `/files` directory contains the same `CALL.html` file retrieved via FTP, suggesting that the FTP directory is being served by the web server.<br>
Since uploaded files appear to be web-accessible, upload a PHP reverse shell through FTP and execute it via the web server.

[reverse-shell.php](../Vulnhub/images/hackable-II/image-7.png)

Upload the payload using the FTP put command, ensuring that your current local directory contains the reverse shell file.<br>
make sure to login into ftp inside the directory where the shell file is saved<br>

[FTP reverse-shell.php](../Vulnhub/images/hackable-II/image-8.png)

# Exploitation
Start a Netcat listener on the attacker machine, then trigger the uploaded PHP payload through the browser.

[trigger reverse shell](../Vulnhub/images/hackable-II/image-9.png)

# Initial Access

[reverse shell achieved](../Vulnhub/images/hackable-II/image-10.png)

The payload successfully establishes a reverse shell on the target.
 
During post-exploitation enumeration, an `important.txt` file is discovered alongside the `shrek` user's home directory.<br>
Reading important.txt suggests executing a hidden script named `.runme.sh.`

[./runme.sh](../Vulnhub/images/hackable-II/image-11.png)

Running the script produces an MD5 hash, which can be cracked using an online database such as CrackStation.

[crackstation](../Vulnhub/images/hackable-II/image-12.png)

The recovered credentials are:
- username:- shrek
- password:- onion<br>
Use them to switch to the shrek account.

[shrek](../Vulnhub/images/hackable-II/image-13.png)

# Privilege Escalation
Enumerate the user's `sudo` privileges:

[sudo -l](../Vulnhub/images/hackable-II/image-14.png)

Consult GTFOBins to determine whether the permitted binaries can be abused for privilege escalation.

[GTFobins](../Vulnhub/images/hackable-II/image-15.png)

Ensure that the GTFOBins payload uses the correct interpreter version (python3.5 in this case) to match the target environment.

[root access](../Vulnhub/images/hackable-II/image-16.png)

Executing the modified GTFOBins payload successfully spawns a root shell.

[User Flag](../Vulnhub/images/hackable-II/image-18.png)

[Root Flag](../Vulnhub/images/hackable-II/image-17.png)

Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!!