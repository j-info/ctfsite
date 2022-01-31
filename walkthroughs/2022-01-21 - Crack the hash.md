# Crack the hash
**Date:** January 21st 2022

[**Link**](https://tryhackme.com/room/crackthehash) to the **Crack the hash** room on TryHackMe.

<br>

## Objectives

This room requires that you crack 9 different password hashes of varying types.

<br>

## Identifying the hash types

A few notse on identifying hashes before we begin. Sometimes you'll run a hash through a tool to identify it and multiple options will be displayed so you'll have to trial and error your way through finding the correct one to crack it. Sometimes these tools will also output incorrect information. These are just a couple of the joys of hash cracking but the good news is I'm already starting to be able to identify several of the hash types just by looking at them since they have a general structure and format to them in most cases.
Resources for identifying hash types

Also, before starting to crack passwords you'll in many cases need to take some time and identify what type of password hash it is. You'll then use that information with the tools so they know how to crack the hash.

<br>

**Resources for identifying hash types**

There are several tools and resources available to help with these tasks and I'll list a few of them here:
- [hashcat.net](https://hashcat.net/wiki/doku.php?id=example_hashes) - A good listing and as a bonus it lists the hash mode number you plug in when using hashcat to crack the hash.
- [hashes.com](https://hashes.com/en/tools/hash_identifier) - A website with a simple interface. Just paste the hash into the box and it will attempt to identify it's type.
- [tunnelsup.com](https://www.tunnelsup.com/hash-analyzer/) - Another website.
- **name-that-hash** - This is a command line python script that you can install via:  `sudo apt install name-that-hash`. What's great about this tool is it will list both the mode number for hashcat and format type for John the Ripper.

<br>

To get started I added all of these hashes to a single text file:
```
──(kali㉿kali)-[~/work]
└─$ cat hashes.txt          
48bb6e862e54f2a795ffc4e541caed4d
CBFDAC6008F9CAB4083784CBD1874F76618D2A97
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
279412f945939ba78ce0758d3fd83daa
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
1DFECA0C002AE40B8619ECF94819CC1B
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
```

And then ran the file through the command line utility listed above called **name-that-hash** ``nth`` for a list of possible types on each hash:
```
┌──(kali㉿kali)-[~/work]
└─$ nth -f hashes.txt --no-banner -a

48bb6e862e54f2a795ffc4e541caed4d

Most Likely 
MD5, HC: 0 JtR: raw-md5 Summary: Used for Linux Shadow files.
MD4, HC: 900 JtR: raw-md4
NTLM, HC: 1000 JtR: nt Summary: Often used in Windows Active Directory.
Domain Cached Credentials, HC: 1100 JtR: mscach


CBFDAC6008F9CAB4083784CBD1874F76618D2A97

Most Likely 
SHA-1, HC: 100 JtR: raw-sha1 Summary: Used for checksums.
HMAC-SHA1 (key = $salt), HC: 160 JtR: hmac-sha1
Double SHA-1, HC: 4500 
RIPEMD-160, HC: 6000 JtR: ripemd-160


1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

Most Likely 
SHA-256, HC: 1400 JtR: raw-sha256 Summary: 256-bit key and is a good partner-function 
for AES. Can be used in Shadow files.
Snefru-256, JtR: snefru-256
RIPEMD-256, JtR: dynamic_140
Haval-256 (3 rounds), JtR: dynamic_140


$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

Most Likely 
bcrypt, HC: 3200 JtR: bcrypt
Blowfish(OpenBSD), HC: 3200 JtR: bcrypt Summary: Can be used in Linux Shadow Files.
Woltlab Burning Board 4.x, 


279412f945939ba78ce0758d3fd83daa

Most Likely 
MD5, HC: 0 JtR: raw-md5 Summary: Used for Linux Shadow files.
MD4, HC: 900 JtR: raw-md4
NTLM, HC: 1000 JtR: nt Summary: Often used in Windows Active Directory.
Domain Cached Credentials, HC: 1100 JtR: mscach


F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

Most Likely 
SHA-256, HC: 1400 JtR: raw-sha256 Summary: 256-bit key and is a good partner-function 
for AES. Can be used in Shadow files.
Snefru-256, JtR: snefru-256
RIPEMD-256, JtR: dynamic_140
Haval-256 (3 rounds), JtR: dynamic_140


1DFECA0C002AE40B8619ECF94819CC1B

Most Likely 
MD5, HC: 0 JtR: raw-md5 Summary: Used for Linux Shadow files.
MD4, HC: 900 JtR: raw-md4
NTLM, HC: 1000 JtR: nt Summary: Often used in Windows Active Directory.
Domain Cached Credentials, HC: 1100 JtR: mscach


$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSr
HVXgMpdjS6xeKZAs02.

Most Likely 
SHA-512 Crypt, HC: 1800 JtR: sha512crypt


e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme

Most Likely 
SHA-1, HC: 100 JtR: raw-sha1 Summary: Used for checksums.
HMAC-SHA1 (key = $salt), HC: 160 JtR: hmac-sha1
Double SHA-1, HC: 4500 
RIPEMD-160, HC: 6000 JtR: ripemd-160
```

Now that we have that information let's move on to cracking.

<br>

## Cracking the hashes

I prefer to use hashcat for most of my password cracking due to how fast it is.

**TIP:** You can greatly increase the speed of hashcat if you run it on your host machine instead of inside a virtual machine because it can take advantage of your video card. You'll see the **-D 2** argument in all of my hashcat commands, and that tells it to use my video card.

<br>

**Resources for cracking the hashes**

We have several tools and resources available when it comes to cracking password hashes. Two of the more popular command line tools are:
- **John the Ripper**
- **hashcat**

And a popular website:
- [crackstation.net](https://crackstation.net/) - This website uses [rainbow tables](https://www.geeksforgeeks.org/understanding-rainbow-table-attack/) for quick results.

<br>

As a FYI the command line tools have issues cracking multiple password hashes in a single file if there are different types present, so at this point I created a separate file for each hash to use with the command line tools.

For my own learning I went through and cracked all of these hashes using both John the Ripper and hashcat, so you'll see the commands necessary in both tools for each hash. I hope you find that useful! I also let you know whether or not the crackstation website will crack the password or not.

<br>

**Hash #1 -** 48bb6e862e54f2a795ffc4e541caed4d
- Hash type identified: **MD5**
- hashcat command: `hashcat -m 0 -D 2 hash1.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=raw-md5 hash1.txt`
- crackstation website: Works

<br>

**Hash #2 -** CBFDAC6008F9CAB4083784CBD1874F76618D2A97
- Hash type identified: **SHA1**
- hashcat command: `hashcat -m 100 -D 2 hash2.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=raw-sha1 hash2.txt`
- crackstation website: Works

<br>

**Hash #3 -** 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
- Hash type identified: **SHA256**
- hashcat command: `hashcat -m 1400 -D 2 hash3.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=raw-sha256 hash3.txt`
- crackstation website: Works

<br>

**Hash #4 -** $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

**NOTE:** I used a different wordlist here to save you a little time. The password happens to be in both this and rockyou, and this list is much smaller to process.
- Hash type identified: **bcrypt**
- hashcat command: `hashcat -m 3200 -D 2 hash4.txt directory-list-2.3-medium.txt`
- JtR command: `john --wordlist=/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --format=bcrypt hash4.txt`
- crackstation website: Does not work. This password is salted and crackstation cannot handle salted passwords.

<br>

**Hash #5 -** 279412f945939ba78ce0758d3fd83daa

**NOTE:** This hash will not be cracked using the standard rockyou wordlist. The crackstation website does however crack it for you. I'll list the commands that would have worked for hashcat and JtR had the password been in rockyou.
- Hash type identified: **MD4**
- hashcat command: `hashcat -m 900 -D 2 hash5.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=raw-md4 hash5.txt`
- crackstation website: Works

<br>

**Hash #6 -** F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
- Hash type identified: **SHA256**
- hashcat command: `hashcat -m 1400 -D 2 hash6.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=raw-sha256 hash6.txt`
- crackstation website: Works

<br>

**Hash #7 -** 1DFECA0C002AE40B8619ECF94819CC1B

**NOTE:** name-that-hash lists both MD5 and MD4 before NTLM as most likely solutions, but this turned out to be NTLM.
- Hash type identified: **NTLM**
- hashcat command: `hashcat -m 1000 -D 2 hash7.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=nt hash7.txt`
- crackstation website: Works

<br>

**Hash #8 -** $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
- Hash type identified: **SHA-512 Crypt**
- hashcat command: `hashcat -m 1800 -D 2 hash8.txt rockyou.txt`
- JtR command: `john --wordlist=rockyou.txt --format=sha512crypt hash8.txt`
- crackstation website: Does not work. This password is salted and crackstation cannot handle salted passwords.

<br>

**Hash #9 -** e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme

**NOTE:** name-that-hash listed SHA-1 as the top choice, but given the fact there is a salt at the end of the hash we know it cannot be that. Because of that I went straight for the 2nd option listed: HMAC-SHA1.
- Hash type identified: **HMAC-SHA1**
- hashcat command: `hashcat -m 160 -D 2 hash9.txt rockyou.txt`
- JtR command: `N/A` : John will not crack this one - see open issue at: [**Link**](https://github.com/openwall/john/issues/4259)
- crackstation website: Does not work. This password is salted and crackstation cannot handle salted passwords.

<br>

Many thanks to:
- [**ben**](https://tryhackme.com/p/ben) for creating this room.
- **TryHackMe** for hosting this room.


Visit TryHackMe at: [**https://tryhackme.com**](https://tryhackme.com)
