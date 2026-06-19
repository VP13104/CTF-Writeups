# Grotesque 2| Vulnhub | Walkthrough


| Category          | Details               |
|----------         |---------              |
| Platform          | [Grotesque  2](https://www.vulnhub.com/entry/grotesque-2,673/)              |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, ffuf, crackstation, hydra, pspy64  |

# Reconnaissance
Begin by identifying the target machine's IP address on the local network.

[arp-sudo](../Vulnhub/images/grotesque-2/image.png)

IP:- 10.0.2.23<br>
Next, perform an Nmap scan to enumerate its open ports and services.

[Nmap Scan](../Vulnhub/images/grotesque-2/image-1.png)

nmap results show:
The scan reveals `SSH` on port `22` along with numerous additional `TCP` ports between `32` and `600`, making it difficult to immediately identify the web service.<br>
To locate the HTTP service efficiently, generate a list of candidate ports and fuzz them using ffuf.
```
seq 32 600 > port.txt
```
[port list](../Vulnhub/images/grotesque-2/image-2.png)

The -fw 39 option filters responses containing exactly 39 words, helping eliminate repetitive false positives and making the valid result easier to identify.
```
ffuf -u http://<ip address>:FUZZ/ -w <path to port list> -fw 39
```
[ffuf](../Vulnhub/images/grotesque-2/image-3.png)

# Enumeration
The fuzzing process identifies port 258 as the location of the web application.
Let’s check it out

[webpage 258](../Vulnhub/images/grotesque-2/image-4.png)

The page displays what appears to be a list of usernames. Save these entries for later use during authentication attempts.

[username.txt](../Vulnhub/images/grotesque-2/image-5.png)

Next up we see the message<br>
“clap clap 👌 - 💯” Inspecting the page source reveals that the emojis are actually image files rather than Unicode characters.

[source code](../Vulnhub/images/grotesque-2/image-6.png)

Standard steganography tools such as `binwalk, steghide, and stegseek` do not reveal any hidden content.<br>
Closer visual inspection of the images reveals hidden text embedded within the graphics.

[hand emoji](../Vulnhub/images/grotesque-2/image-7.png)

Zooming into the image reveals an MD5 hash, which can be submitted to CrackStation for lookup.

[crackstation](../Vulnhub/images/grotesque-2/image-8.png)

CrackStation returns a likely match, but the recovered value requires further validation.<br>
The accompanying clue (👌 - 100) suggests subtracting 100 from the recovered numeric suffix.<br>
so in that logic<br>
b6e705ea1249e2bb7b0fd7dac9fcd1b3 - 100 =<br> b6e705ea1249e2bb7b0fd7dac9fcd0b3<br>
Now back to crackstation

[crackstation_2](../Vulnhub/images/grotesque-2/image-9.png)

Repeating the lookup with the adjusted hash confirms the intended password: `solomon1`.

# Exploitation

With a list of usernames and a candidate password, use Hydra to identify the valid SSH credentials.
```
hydra -L username.txt -p solomon1 ssh://ip_address
```
[hydra](../Vulnhub/images/grotesque-2/image-10.png)

Hydra successfully identifies the following credentials:<br>
username: angel<br>
password: solomon1

# Initial Access

[ssh login](../Vulnhub/images/grotesque-2/image-11.png)

The recovered credentials provide successful SSH access as the `angel` user.

# Privilege Escalation

The `angel` account has no useful `sudo` privileges.<br>
During filesystem enumeration, a directory named quiet is discovered containing numerous randomly named files.

[quiet directory](../Vulnhub/images/grotesque-2/image-12.png)

Monitor background processes with pspy64 to identify scheduled tasks running as higher-privileged users.

[pspy64](../Vulnhub/images/grotesque-2/image-13.png)

two scripts 
- /root/check.sh 
- /root/write.sh 

are executed every two minutes, Additionally, every file within the quiet directory is owned by root, indicating that these scripts likely interact with its contents.<br>
To better understand the automation, remove the files in quiet and monitor the filesystem for changes in “quiet” and “/” directories.
1. delete all files in quiet
2. list out files in `/`
```
cd /quiet
rm -rf *
ls -lh 
ls -lh /
```

[the above process](../Vulnhub/images/grotesque-2/image-14.png)

After clearing the directory and waiting for the scheduled tasks to execute, a new file named `rootcreds.txt` appears in the `/` directory.

The newly created file contains credentials for the root account, which can be used to obtain full administrative access.

[rootcreds.txt](../Vulnhub/images/grotesque-2/image-15.png)

[user.txt](../Vulnhub/images/grotesque-2/image-17.png)

[root.txt](../Vulnhub/images/grotesque-2/image-16.png)

Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!
---

### PS

Curious about how `rootcreds.txt` was generated, I examined the scheduled scripts responsible for maintaining the `quiet` directory.
Well i checked `chech.sh` and `write.sh` file

[corn job files](../Vulnhub/images/grotesque-2/image-18.png)

The `check.sh` script periodically checks whether `/home/angel/quiet` is empty. If no files are present, it creates `rootcreds.txt`, writes the root credentials to it, and assigns overly permissive permissions, making it accessible to all users. and changing the file permission so everyone can read, write and execute the file<br>
The companion script, write.sh, continuously repopulates the quiet directory to ensure it normally remains non-empty.<br>
Logic:
1.	Go to `/home/angel/quiet`
2.	Check if directory is empty
3.	If empty:
    - create/append a file containing root credentials
    - make it world-accessible

Overall, Purpose of write.sh<br>
Make sure the directory is never empty<br>
By deleting every file in `quiet` and waiting for the next scheduled execution, `check.sh` detects the empty directory and generates `rootcreds.txt`, unintentionally exposing the root credentials.