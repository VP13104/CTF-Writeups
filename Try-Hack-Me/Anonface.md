# Anonface | Tryhackme | Writeup


| Category          | Details               |
|-------------------|-----------------------|
| Platform          | [Anonface](https://tryhackme.com/room/bsidesgtanonforce) |
|  Difficulty       | Easy                  |
| Target machine    | Ubuntu Linux          |
| Attacker Machine  | Kali linux            |
| Tools             | Nmap, gpg, John the ripper      |



# Reconnaissance
Scan the target machine to identify open ports and running services.

[Nmap Scan](../Try-Hack-Me/images/anonface/nmap-scan.png)

The Nmap scan reveals the following open ports:

| Port      | Service       |
| --------- |---------------|
| 21/tcp    | ftp           |
| 22/tcp    | ssh           |

# Enumeration 

Let's check whether the FTP service allows anonymous authentication.

[FTP login](../Try-Hack-Me/images/anonface/ftp-files.png)

* User Flag at `/home/melodias/user.txt`

After logging in anonymously via FTP, we gain access to the following files and directories:
- `/etc/passwd`
- `notread` Directory (world-readable, writable, and executable)
- `private.asc` 
- `backup.gpg` from `notread`

[private.asc](../Try-Hack-Me/images/anonface/private-asc.png)

[backup.gpg](../Try-Hack-Me/images/anonface/backup-gpg.png)

`backup.gpg` is an encrypted file, and `private.asc` appears to be a GPG private key. Since GPG uses public-key cryptography, importing the private key may allow us to decrypt the backup if we can recover its passphrase.

# Exploitation

Since GPG relies on public/private key pairs, our first step is to import the recovered private key.<br>
here, `private.asc` is the private key<br>
Let's use gpg tool to import the key and decrypt `backup.gpg`
```
gpg --import private.asc
```
[import private](../Try-Hack-Me/images/anonface/import-asc-1.png)

The import fails because the private key is protected by a passphrase.<br>
We can extract the hash from the private key and use John the Ripper to recover the passphrase:<br>

```
gpg2john private.asc > pgp.hash

john --wordlist=/usr/share/worlists/rockyou.txt pgp.hash
```
[john](../Try-Hack-Me/images/anonface/gpg.png)

After recovering the passphrase, import the private key again. This time the import succeeds.

[import private](../Try-Hack-Me/images/anonface/import-asc-2.png)

With the private key successfully imported, decrypt `backup.gpg`
```
gpg --output backup.out --decrypt backup.gpg
```

[decrypt backup.gpg](../Try-Hack-Me/images/anonface/decrypt-gpg.png)

The decrypted contents are written to `backup.out` file

[backup.out](../Try-Hack-Me/images/anonface/backup-out.png)

The decrypted file contains what appears to be a backup of `/etc/shadow`, including password hashes for the `melodias` and `root` accounts.<br>
Extract the password hashes into a text file and crack them using John the Ripper.<br>

[hash file](../Try-Hack-Me/images/anonface/password-hashs.png)

[john](../Try-Hack-Me/images/anonface/john-password.png)

John successfully recovers the root account password.

# Privilege Escalation 

Using the recovered credentials, log in to the machine over SSH as root.

[ssh](../Try-Hack-Me/images/anonface/root-access.png)

After obtaining root access, retrieve both the user and root flags to complete the room.

[USER FLAG](../Try-Hack-Me/images/anonface/user-flag.png)

[ROOT FLAG](../Try-Hack-Me/images/anonface/root-flag.png)

Root flag & User Flag found!<br>
Hope this Writeup was fun, easy to follow and helpful to you.<br>

Happy Hacking ~!!!