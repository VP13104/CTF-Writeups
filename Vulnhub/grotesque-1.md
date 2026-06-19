# Grotesque 1 | Vulnhub | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [Grotesque  1](https://www.vulnhub.com/entry/grotesque-101,658/)              |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, Gobuster, john the ripper, Keeweb  |


# Reconnaissance
Begin by identifying the target machine's IP address on the local network.
[arp-scan](../Vulnhub/images/grotesque-1/image.png)

IP:- 10.0.2.22
perform an Nmap scan to enumerate its open ports and services.

[nmap scan](../Vulnhub/images/grotesque-1/image-1.png)

nmap results show:
| Port      | Service       |
| --------- |---------------|
| 66/tcp    | http          |
| 80/tcp    | http          |

Browse to the HTTP services running on ports 66 and 80 to inspect the web applications.

[webpage port 66](../Vulnhub/images/grotesque-1/image-2.png)

[webpage port 80](../Vulnhub/images/grotesque-1/image-3.png)

# Enumeration

Directory enumeration with Gobuster on both ports `66` and `80` does not reveal any significant findings.

[Gobuster ](../Vulnhub/images/grotesque-1/image-4.png)

The application on port 66 provides a ZIP archive containing the project source code. Download and inspect its contents for useful information.

[Zip file](../Vulnhub/images/grotesque-1/image-5.png)

[zip file contents](../Vulnhub/images/grotesque-1/image-6.png)

Unzip the file<br>
- analyze _vvmlist directory
- cat _vvmlist/* | sort | uniq

[_vvmlist](../Vulnhub/images/grotesque-1/image-7.png)

Analysis of the extracted files reveals a WordPress installation hosted at `/lyricsblog` on port `80`. <br>
let’s check it out.

[wordpress](../Vulnhub/images/grotesque-1/image-8.png)

The site appears to be a basic lyrics blog. Continue enumerating the WordPress installation and its directories.

[gobuster](../Vulnhub/images/grotesque-1/image-9.png)

`http://ip address/lyricsblog/wp-admin` <br>
The `/wp-admin` endpoint redirects to the standard WordPress login page (`wp-login.php`).
[login page](../Vulnhub/images/grotesque-1/image-10.png)

WPScan is an effective tool for enumerating WordPress users and attack surfaces.
```
wpscan --url http://10.0.2.22/lyricsblog -e at -e ap -e u
```

[wpscan](../Vulnhub/images/grotesque-1/image-11.png)

WPScan successfully identifies the following valid username: erdalkomurcu<br>
Password guessing and brute-force attempts prove unsuccessful.<br>
Given the theme of the website, consider whether one of the published lyrics may be used as the administrator's password.

[lyric](../Vulnhub/images/grotesque-1/image-12.png)

Copy the lyrics into a text file, preserving paragraph separation, generate the `MD5 hash` of the content, and convert the resulting hash to uppercase. The resulting value matches the administrator password.

[password](../Vulnhub/images/grotesque-1/image-13.png)

username: erdalkomurcu<br>
password: BC78C6AB38E114D6135409E44F7CDDA2

# Exploitation

[wordpress login](../Vulnhub/images/grotesque-1/image-14.png)

The derived credentials successfully authenticate to the WordPress admin panel.<br>
With administrative access, modify a theme template to deploy a PHP reverse shell.<br> 
`Appearance -> Theme File editor -> archive.php` <br>
Replace the contents of archive.php with a PHP reverse shell, such as the one provided by [pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

[reverse-shell.php](../Vulnhub/images/grotesque-1/image-15.png)

Start a Netcat listener on the attacker machine before triggering the payload.
```
nc -lvp 4444
```
Access the modified archive.php page to execute the payload and establish the reverse shell:-
```
http://<target-ip>/lyricsblog/wp-content/themes/twentytwentyone/archive.php
```

[archive.php](../Vulnhub/images/grotesque-1/image-16.png)

[shell](../Vulnhub/images/grotesque-1/image-17.png)

Once the reverse shell is established, upgrade it to an interactive TTY:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

# Privilege Escalation
Let’s move up our privileges. after some snooping around found a interesting file at `/var/www/html/lyricsblog/wp-config.php`

[wp-config.php](../Vulnhub/images/grotesque-1/image-18.png)

The configuration file contains credentials for another local user:<br>
- Username: raphael 
- Password: _double_trouble_  <br>
Use the recovered credentials to switch to the raphael account.

[su raphael](../Vulnhub/images/grotesque-1/image-19.png)

## Root Access

checking contents of current directory `ls -la `<br>
Enumerating files in the current directory reveals an interesting hidden KeePass database.

[ls -la](../Vulnhub/images/grotesque-1/image-20.png)

Transfer the hidden KeePass database to the attacker machine for offline analysis.<br>
Extract a crackable hash from the `.kdbx` file and use John the Ripper to recover its master password.

[crack password using john](../Vulnhub/images/grotesque-1/image-21.png)

After recovering the KeePass master password, open the database with [keeweb](https://app.keeweb.info/) to inspect its stored entries.

[keeweb](../Vulnhub/images/grotesque-1/image-22.png)

The database contains several stored credentials. Testing each one eventually yields the root password.

[passwords](../Vulnhub/images/grotesque-1/image-23.png)

One of the recovered credentials (`.:.subjective.:.`) successfully authenticates as `root`, providing full system access.

[user flag](../Vulnhub/images/grotesque-1/image-25.png)

[root flag](../Vulnhub/images/grotesque-1/image-24.png)

Root flag & User Flag found!

I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!