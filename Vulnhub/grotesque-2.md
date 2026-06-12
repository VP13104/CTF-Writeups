# Grotesque 2| Vulnhub | Walkthrough


| Category          | Details               |
|----------         |---------              |
| Platform          | [Grotesque  2](https://www.vulnhub.com/entry/grotesque-2,673/)              |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, ffuf, crackstation, hydra, pspy64  |

# Reconnaissance
Let’s find the IP address of the machine

[arp-sudo](../Vulnhub/images/grotesque-2/image.png)

IP:- 10.0.2.23<br>
let’s scan the machines for ports and services

[Nmap Scan](../Vulnhub/images/grotesque-2/image-1.png)

nmap results show:
22/tcp ssh & way too many other ports ranging from 32–600<br>
let’s make a list of these ports and fuzz it with the ip address to figure out the actual port where the webpage is hosted.
```
seq 32 600 > port.txt
```
[port list](../Vulnhub/images/grotesque-2/image-2.png)

use ffuf to fuzz the machine -fw 39 will filter out all the response with Words=39 this way we can remove all noisy false positives.
```
ffuf -u http://<ip address>:FUZZ/ -w <path to port list> -fw 39
```
[ffuf](../Vulnhub/images/grotesque-2/image-3.png)

# Enumeration
Looks like we hit jackpot at port 258.
Let’s check it out

[webpage 258](../Vulnhub/images/grotesque-2/image-4.png)

At first glance we have got a user list. Let’s save it into a txt file.

[username.txt](../Vulnhub/images/grotesque-2/image-5.png)

Next up we see the message<br>
“clap clap 👌 - 💯” checked the source code and found out the emojis aren’t just emojis they are images.

[source code](../Vulnhub/images/grotesque-2/image-6.png)

Downloaded the images and ran it through binwalk, steghide, stegseek but did not get anythig useful. used AI for some help 😅 and i got a hint to zoom into the images. And there you have it

[hand emoji](../Vulnhub/images/grotesque-2/image-7.png)

Found a md5 hash
used crackstation to crack it

[crackstation](../Vulnhub/images/grotesque-2/image-8.png)

Got partial result. its a 50/50 chance that this is correct but need confirmatiom.<br>
If we go back to the message it says “👌 — 100”
so in that logic<br>
b6e705ea1249e2bb7b0fd7dac9fcd1b3 - 100 =<br> b6e705ea1249e2bb7b0fd7dac9fcd0b3<br>
Now back to crackstation

[crackstation_2](../Vulnhub/images/grotesque-2/image-9.png)

Ha!! so “solomon1” was the right answer after all

# Exploitation

Now we have usernames and password, use hydra to find out the correct username
```
hydra -L username.txt -p solomon1 ssh://ip_address
```
[hydra](../Vulnhub/images/grotesque-2/image-10.png)

username: angel<br>
password: solomon1

# Initial Access

[ssh login](../Vulnhub/images/grotesque-2/image-11.png)

Logged in!!!

# Privilege Escalation

user is not allowed to run sudo commands<br>
I did some browsing around the user files and folders found a directory named `quiet` that has a ton of files named with random numbers 

[quiet directory](../Vulnhub/images/grotesque-2/image-12.png)

Let’s use pspy64 to find all cronjobs running

[pspy64](../Vulnhub/images/grotesque-2/image-13.png)

two scripts 
- /root/check.sh 
- /root/write.sh 

are executed every two minutes, Also every file in the “quiet” directory is owned by the root.<br>
Let’s disrupt this process and keep an eye out for changes in “quiet” and “/” directories.
1. delete all files in quiet
2. list out files in `/`
```
cd /quiet
rm -rf *
ls -lh 
ls -lh /
```

[the above process](../Vulnhub/images/grotesque-2/image-14.png)

After deleting and monitoring it multiple times.<br>
inside “/” directory a file named “rootcreds.txt” is created

the file has root user credentials, use it to gain root control

[rootcreds.txt](../Vulnhub/images/grotesque-2/image-15.png)

[user.txt](../Vulnhub/images/grotesque-2/image-17.png)

[root.txt](../Vulnhub/images/grotesque-2/image-16.png)

Root flag & User Flag found!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.

Happy Hacking ~!!!

---

### PS

Even i wanted know what and how the rootcreds.txt appeared from.
Well i checked `chech.sh` and `write.sh` file

[corn job files](../Vulnhub/images/grotesque-2/image-18.png)

its a cron job on loop where check.sh checks quiet directory if it is empty then it will append lines “root & sweetchild” into `rootcreads.txt` and changing the file permission so everyone can read, write and execute the file<br>
Overall purpose of check.sh<br>
Logic:
1.	Go to `/home/angel/quiet`
2.	Check if directory is empty
3.	If empty:
    - create/append a file containing root credentials
    - make it world-accessible

Overall, Purpose of write.sh<br>
Make sure the directory is never empty<br>
So, when we delete the directory and wait for check.sh to run, once it does file rootcreds.txt gets created.
