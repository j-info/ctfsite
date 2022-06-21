[<img src="images-other/thm-progress.png">](TryHackMeStats.md)   [<img src="images-other/htb-progress.png">](HTBStats.md)
[<img src="images-other/additionaltraining.png">](AdditionalTraining.md)

---

<br>

## *SunsetNoontide* on Proving Grounds
#### June 20th 2022

Topics: ![](images/linux.png) ![](images/unrealircd.png) ![](images/metasploit.png) ![](images/defaultcreds.png)

Very easy machine. We find an UnrealIRCd server running on the system that's vulnerable to an RCE exploit and use that to establish a foothold on the system. Then we find default credentials for the root user and escalate our privileges that way.

[**SunsetNoontide Walkthrough**](walkthroughs/2022-06-20-SunsetNoontide.md)

<br>

---

<br>

## SANS Ransomware Summit 2022
#### June 16th and 17th 2022

Topics: ![](images/learning.png)

This summit was 14 hours split over 2 days and had some very interesting and useful information covering a wide range of ransomware topics. Some of the presentations I found the most interesting were:

+ How to engage with an attacker after you've fallen victim to an attack
+ Cryptocurrency and it's role in organized cybercrime
+ The cybercriminal supply chain and how it's differenct pieces are getting more and more specialized, and how they're starting to function more like legit businesses
+ And an incident response retelling from a targetted ransomware attack in the industrial sector

This is the 3rd summit I've attended from SANS and they're always such great events.

[**View Certificate**](certs/sans-ransomwaresummit.pdf)

<br>

---

<br>

## *FunboxRookie* on Proving Grounds
#### June 12th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/ftp.png) ![](images/jtr.png) ![](images/sudo.png)

A quick finish on this one. We find password protected .zip files on an FTP server that we can login to anonymously. We download the .zip files, crack the passwords, and unzip them to find they contain id_rsa files. We use one of those to ssh into the system and find clear text credentials inside of a .mysql_history file, and use credentials to check sudo -l which shows us we can run anything as root.

[**FunboxRookie Walkthrough**](walkthroughs/2022-06-12-FunboxRookie.md)

<br>

---

<br>

## *FunboxEasy* on Proving Grounds
#### June 9th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sqli.png) ![](images/cms.png) ![](images/sudo.png)

This box has several websites to investigate and we eventually find one we're able to login as admin to and use a php reverse shell to gain our initial foothold. After that we find user credentials in a file and use those to laterally move. The user we move to has sudo privileges for both pkexec and time, and we can use either to escalate to root.

[**FunboxEasy Walkthrough**](walkthroughs/2022-06-09-FunboxEasy.md)

<br>

---

<br>

## Security+ Certification
#### June 6th 2022

Topics: ![](images/learning.png)

The Evolve bootcamp I've been in since January will be ending in a little over 2 weeks so I'm preparing for the final project and test there, as well as spending a lot of time studying for the Security+ certification.

The bootcamp includes an exam voucher for the Security+ and I decided to take a work backwards approach and schedule a date to sit for the exam. I've always liked this approach since it let's you avoid the trap of drawing the process out longer than it needs to be and forces you to really focus on the objective.

So with that said I'll be taking the exam on June 29th, and because of that you'll see less CTF walkthroughs here than normal since I'm focusing on finishing the bootcamp and studying for the Security+.

<br>

---

<br>

## *PyExp* on Proving Grounds
#### June 4th 2022

Topics: ![](images/linux.png) ![](images/mysql.png) ![](images/hydra.png) ![](images/crypto.png) ![](images/python.png) ![](images/scripting.png)

An interesting box that finds us brute forcing our way into an exposed MySQL database to find some Fernet encrypted credentials which we decrypt to establish an initial foothold. Then we find a python script that can be executed as root to escalate our privileges.

[**PyExp Walkthrough**](walkthroughs/2022-06-04-PyExp.md)

<br>

---

<br>

## *Katana* on Proving Grounds
#### June 3rd 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/cms.png) ![](images/capabilities.png)

A lot of website enumeration was required on this CTF since there were 3 different websites and rabbit holes to dig into. We eventually find an upload form that allows us to send a php reverse shell and establish our initial foothold on the system. After that we find cap_setuid+ep set on python2.7 and use it to escalate to root.

[**Katana Walkthrough**](walkthroughs/2022-06-03-Katana.md)

<br>

---

<br>

## *CyberSploit1* on Proving Grounds
#### June 1st 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/cve.png)

This system we find a username in the web pages source code, a password in robots.txt, and then ssh over for our initial foothold. Then we see the system was running a very old version of Ubuntu that's vulnerable to the **CVE-2015-1328** aka **overlayfs** exploit which we use to escalate our privileges to root.

[**CyberSploit1 Walkthrough**](walkthroughs/2022-06-01-CyberSploit1.md)

<br>

---

<br>

## SANS Emergency Webcast - MSDT "Follina" (CVE-2022-30190)
#### May 31st 2022

Topics: ![](images/cve.png)

I attended this emergency webcast today where Jake Williams from the SANS Institute covered Follina (CVE-2022-30190) and ways to detect, mitigate, and hunt for it.

This exploit abuses the ms-mdt protocol handler and there are already several POC published. In addition it's pretty trivial to exploit.

### A couple ways to mitigate:
- Disable the MSDT protocol:
> [**Microsoft MSRC Blog**](https://msrc-blog.microsoft.com/2022/05/30/guidance-for-cve-2022-30190-microsoft-support-diagnostic-tool-vulnerability/)
- Disable troubleshooting tools via GPO:
> reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\ScriptedDiagnostics" /t REG_DWORD /v EnableDiagnostics /d 0

<br>

### A few ways to detect:
- Check for msdt.exe running, this is the one thing that will always be a constant in this exploit
- sdiagnhost.exe will be the parent process
- Check to see if winword.exe connects to websites other than *.microsoft.com or *.office.com
 
<br>

### Forensics and hunting:
- Check the Microsoft Office internet cache:
> reg query "hkcu\software\microsoft\office\16.0\common\internet\server cache"
- Yara rule by Joe Security:
> [**joesecurity.org Yara Rule**](https://joesecurity.org/resources/follina.yara)
- Sigma rules by Chris Peacock or Kostas:
> [**Chris Peacock GitHub**](https://github.com/securepeacock/sigma/blob/963289fbbc961454979d3b0219ac103a4142e1b4/rules/windows/process_creation/proc_creation_win_msdt_follina.yml) or
>[**Kostas GitHub**](https://github.com/tsale/Sigma_rules/blob/main/windows_exploitation/ms-msdt_exploitation.yml)

<br>

---

<br>

## *Thompson* on TryHackMe
#### May 30th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/tomcat.png) ![](images/msfvenom.png) ![](images/cron.png) ![](images/scripting.png)

This was a quick and easy box with default credentials that let us into the Apache Tomcat manager panel. From there we were able to deplaoy a WAR reverse shell and ended up finding a cron job that ran as root calling a script we could modify for privilege escalation.

[**Thompson Walkthrough**](walkthroughs/2022-05-30-Thompson.md)

<br>

---

<br>

## *AllSignsPoint2Pwnage* on TryHackMe
#### May 27th 2022

Topics: ![](images/windows.png) ![](images/web.png) ![](images/ftp.png) ![](images/smb.png) ![](images/crackmapexec.png)

This was a fun Windows based box that had us using FTP, SMB, PHP reverse shells, VNC, and plenty of manual enumeration.

[**AllSignsPoint2Pwnage Walkthrough**](walkthroughs/2022-05-27-AllSignsPoint2Pwnage.md)

<br>

---

<br>

## Antisyphon: Active Defense & Cyber Deception
#### May 26th 2022

Topics: ![](images/learning.png)

This was a 4 day class taught by former SANS instructor John Strand. A few of the topics covered were:
+ Various open source tools that can be easily configured and setup in your environment to detect, delay, and gather information on attackers
+ The legality of active defense: what to avoid and what is ok based on actual legal precedence
+ The MITRE Engage framework
+ Canarytokens, port spoofing, honeypots, honeyports, and other honey measures
+ 9 different hands on labs
+ And much more

This course really drove home how you can use active defense and cyber deception not just for malicious "hackback" type activies, but also for attribution, detection capabilities, and slowing attackers down giving you more time to respond. I had a great time taking this training and look forward to additional Antisyphon classes soon!

[**View Certificate**](certs/as-activedefensecyberdeception.pdf)

<br>

---

<br>


## *Biblioteca* on TryHackMe
#### May 23rd 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sqlmap.png) ![](images/sqli.png) ![](images/mysql.png) ![](images/hydra.png) ![](images/python.png) ![](images/libraryhijacking.png)

New box that came out a couple days ago. We find ourselves with a website vulnerable to SQLi which we use to obtain an initial foothold on the system. From there we laterally move after taking advantage of a weak password. And finally we use Python library hijacking to escalate to root.

[**Biblioteca Walkthrough**](walkthroughs/2022-05-23-Biblioteca.md)

<br>

---

<br>

## *Gaara* on Proving Grounds
#### May 20th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/decoding.png) ![](images/hydra.png) ![](images/suid.png)

This box has us manually enumerate a website to find encoded text, and when decoding we find a system username that we're able to brute force with hydra. After that we find that the GNU Debugger has a SUID bit set on it which allows us to escalate to root.

[**Gaara Walkthrough**](walkthroughs/2022-05-20-Gaara.md)

<br>

---

<br>

## *Anonforce* on TryHackMe
#### May 18th 2022

Topics: ![](images/linux.png) ![](images/ftp.png) ![](images/jtr.png) ![](images/cracking.png) ![](images/gpg.png) ![](images/hashcat.png)

This was also a pretty quick CTF. We only had 2 ports open: FTP and SSH, and when connecting to the FTP server and logging in as anonymous we find the entire file system available to us. We find a private PGP key and encrypted backup file and then crack the password on the PGP key and use it to decrypt the backup which gives us a copy of the /etc/shadow file. We then crack the root hash and login to the system.

[**Anonforce Walkthrough**](walkthroughs/2022-05-18-Anonforce.md)

<br>

---

<br>

## *Simple CTF* on TryHackMe
#### May 17th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/ftp.png) ![](images/hydra.png) ![](images/cracking.png)

A quick and simple CTF. We brute forced our way in via SSH using Hydra and used GTFOBins to escalate to root.

[**Simple CTF Walkthrough**](walkthroughs/2022-05-17-SimpleCTF.md)

<br>

---

<br>

## *RazorBlack* on TryHackMe
#### May 16th 2022

Topics: ![](images/windows.png) ![](images/nfs.png) ![](images/kerberos.png) ![](images/impacket.png) ![](images/cracking.png) ![](images/hashcat.png) ![](images/smb.png) ![](images/crackmapexec.png) ![](images/jtr.png) ![](images/secretsdump.png) ![](images/evilwinrm.png) ![](images/powershell.png) ![](images/kerberoast.png) ![](images/decoding.png) ![](images/robocopy.png)

This was a long box but I learned a lot and it was well done and fun to go through. We initially found files on a public NFS share that we deduced usernames from, and then used crackmapexec to find the hash of a user which we cracked and began to enumerate SMB with. 3 lateral movements later we finally land on a user who has the backup operators group and are able to exploit robocopy to escalate our privileges to system.

[**RazorBlack Walkthrough**](walkthroughs/2022-05-16-RazorBlack.md)

<br>

---

<br>

## *VulnNet: Active* on TryHackMe
#### May 14th 2022

Topics: ![](images/windows.png) ![](images/redis.png) ![](images/responder.png) ![](images/cracking.png) ![](images/hashcat.png) ![](images/crackmapexec.png) ![](images/smb.png) ![](images/impacket.png) ![](images/msfvenom.png) ![](images/metasploit.png) ![](images/powershell.png) ![](images/cve.png)

This box had us exploiting an exposed Redis instance to view files and run commands on the system, using responder to capture hashes and then cracking them with hashcat, enumerating SMB shares, creating payloads with msfvenom, and exploiting PrintNightmare for privilege escalation.

[**VulnNet: Active Walkthrough**](walkthroughs/2022-05-14-VulnNet-Active.md)

<br>

---

<br>

## *Chill Hack* on TryHackMe
#### May 10th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/ftp.png) ![](images/mysql.png) ![](images/cracking.png) ![](images/hashcat.png) ![](images/steg.png) ![](images/jtr.png) ![](images/decoding.png) ![](images/container.png) ![](images/docker.png)

There was a lot going on with this box from cracking, to decoding, and even steganography. We used a webshell embedded into the website for initial access and then found credentials for a SQL database, which led us to additional credentials we could use to ssh in. After that we were able to laterally move to a user who was a member of the docker group which allowed us to mount an image as root to find our final flag.

[**Chill Hack Walkthrough**](walkthroughs/2022-05-10-Chill-Hack.md)

<br>

---

<br>

## *Kubernetes for Everyone* on TryHackMe
#### May 7th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/grafana.png) ![](images/cve.png) ![](images/dirtraversal.png) ![](images/decoding.png) ![](images/kubernetes.png) ![](images/git.png) ![](images/cracking.png) ![](images/hashcat.png)

Brand new machine that was just released yesterday. We exploited a CVE in the Grafana web application to enumerate the system with directory traversal. After we established a foothold on the system we used the k0s distro of Kubernetes for several things, as well as finding hidden information in a local git repository. Decoding and cracking were also required on this one.

[**Kubernetes for Everyone Walkthrough**](walkthroughs/2022-05-07-Kubernetes-for-Everyone.md)

<br>

---

<br>

## Web Fundamentals Learning Path on TryHackMe
#### May 6th 2022

Topics: ![](images/learning.png)

I've been working my way through this learning path and completed it today. It consists of 32 hours of training and covers the following:
- Using Burp suite intruder, repeater, encoder/decoder, comparer, sequencer, BApp store, and more
- Went through the OWASP top 10 and also had you go complete the OWASP Juice Shop challenges
- Covered many standard vulnerability types such as IDOR, LFI, RFI, SSRF, XSS, RCE, SQLi, and more

That's 5 of the 7 learning paths complete now, and the last two I'm over 50% of the way through already. Almost there!

[**Certificate link**](https://tryhackme-certificates.s3-eu-west-1.amazonaws.com/THM-IBJZFTD0XY.png)

<br>

---

<br>

## *Net Sec Challenge* on TryHackMe
#### May 5th 2022

Topics: ![](images/web.png) ![](images/hydra.png)

This was a quick challenge that asked us to perform enumeration with nmap, brute force a couple users on an FTP server with Hydra, and then use a stealthy nmap scan to avoid IDS detection.

[**Net Sec Challenge Walkthrough**](walkthroughs/2022-05-05-Net-Sec-Challenge.md)

<br>

---

<br>

## Level 13 on TryHackMe
#### May 4th 2022

Topics: ![](images/learning.png)

I finally did it! Level 13 is the highest level you can achieve on TryHackMe and I got there today. Out of 1.1 million users on the site I'm currently ranked number 4683. Given the relatively short amount of time I've been using the site I hope that reflects the amount of time and effort I've been putting into furthering my learning.

I found and started using TryHackMe a little over 5 months ago right in the middle of their annual Advent of Cyber 3 Christmas event and have been loving it ever since. I've learned so much since then and will continue to do so on a daily basis until I get to where I want to be in cybersecurity.

<br>

![](images-other/tryhackme-level13.png)

<br>

---

<br>

## *Gotta Catch'em All!* on TryHackMe
#### May 3rd 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/decoding.png)

This was clearly an easy and for fun box that doesn't have a lot that would apply to anything real world, but it was a fun machine to walkthrough regardless!

[**Gotta Catch'em All! Walkthrough**](walkthroughs/2022-05-03-Gotta-Catchem-All.md)

<br>

---

<br>

## *Dogcat* on TryHackMe
#### April 30th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/lfi.png) ![](images/burp.png) ![](images/logpoisoning.png) ![](images/docker.png) ![](images/container.png) ![](images/escape.png)

We use LFI and Apache2 log poisoning to establish initial access on the system and find we're inside of a docker container as the www-data user. Then we figure out how to escalate to container root and eventually escape the container to the host system.

[**Dogcat Walkthrough**](walkthroughs/2022-04-30-Dogcat.md)

<br>

---

<br>

## *Wpwn* on Proving Grounds
#### April 28th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/wordpress.png) ![](images/cve.png)

On this box we find a Wordpress site that's vulnerable to CVE-2019-9978 and use that for an initial foothold on the system. We're able to escalate privileges using a password we found in the wp-config.php file. And finally gain root via sudo.

[**Wpwn Walkthrough**](walkthroughs/2022-04-28-Wpwn.md)

<br>

---

<br>

## *Fowsniff CTF* on TryHackMe
#### April 26th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/twitter.png) ![](images/pastebin.png) ![](images/cracking.png) ![](images/hashcat.png) ![](images/pop3.png) ![](images/hydra.png) ![](images/scripting.png)

We find ourselves visiting a companies website that is currently down due to a breach and end up finding that hackers posted a message on Twitter with a link to Pastebin containing password hashes for the companies employees. We crack these, use them to login to a POP3 mail server to find additional credentials, and finally get system access. Finally we take advantage of being able to write to a bash script that root runs to escalate privileges.

[**Fowsniff CTF Walkthrough**](walkthroughs/2022-04-26-Fowsniff-CTF.md)

<br>

---

<br>

## *UltraTech* on TryHackMe
#### April 22nd 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/api.png) ![](images/cracking.png) ![](images/docker.png)

With this box we find an API vulnerable to command injection and use it to gather credentials and compromise the system. After that we find we're a member of the docker group, and use that to obtain the root private ssh key.

[**UltraTech Walkthrough**](walkthroughs/2022-04-22-UltraTech.md)

<br>

---

<br>

## *Tech_Supp0rt: 1* on TryHackMe
#### April 20th 2022

Topics: ![](images/linux.png) ![](images/smb.png) ![](images/web.png) ![](images/subrion.png) ![](images/cve.png)

This is a brand new box that just came out and it let's us interact with the **Subrion** CMS system to find it's weaknesses.

[**Tech_Supp0rt: 1 Walkthrough**](walkthroughs/2022-04-20-Tech_Supp0rt1.md)

<br>

---

<br>

## *Hacker Rank* on Hack the Box
#### April 18th 2022

Topics: ![](images/learning.png)

And as a bonus to my first medium rated HTB machine it also bumped me up in level to **Hacker** on completion!

![](images-other/htb-hacker.png)

<br>

---

<br>

## *Meta* on Hack the Box
#### April 18th 2022

Topics: ![](images/linux.png) ![](images/wfuzz.png) ![](images/exiftool.png) ![](images/imagemagick.png) ![](images/sudo.png) ![](images/neofetch.png) ![](images/path.png)

Since I ran out of still active easy boxes on HTB I decided to give my first medium rated HTB machine a try. After a lot of enumeration, use of multiple CVE's, and some modification of config files and environmental variables I was successful! This was a fun one to go through.

No walkthrough yet since this is still an active box.

![](images-other/htb-meta.png)

<br>

---

<br>

## *Sumo* on Proving Grounds
#### April 14th 2022

Topics: ![](images/linux.png) ![](images/cve.png)

This was my first Proving Grounds box and I had a lot of fun going through it. To fully complete this it required the use of 2 different CVE's.

[**Sumo Walkthrough**](walkthroughs/2022-04-14-Sumo.md)

<br>

---

<br>

## *Timelapse* on Hack the Box
#### April 13th 2022

Topics: ![](images/windows.png) ![](images/smb.png) ![](images/cracking.png) ![](images/jtr.png) ![](images/openssl.png) ![](images/powershell.png) ![](images/laps.png)

This was a fun Windows based box that was just released a couple weeks ago so I can't post the walkthrough quite yet. When it's retired I'll be sure to come and add it.

![](images-other/htb-timelapse.png)

<br>

---

<br>

## *Ninja Skills* on TryHackMe
#### April 12th 2022

Topics: ![](images/linux.png)

And now I know why I haven't done a lot of Windows boxes on TryHackMe...I'm already running out of ones to choose from. If there are more they're likely hidden in the hard and above rating which I'll have to check out.

This ended up being a very quick and easy refresher on basic linux commands.

[**Ninja Skills Walkthrough**](walkthroughs/2022-04-12-Ninja-Skills.md)

<br>

---

<br>

## *Anthem* on TryHackMe
#### April 10th 2022

Topics: ![](images/windows.png) ![](images/web.png) ![](images/umbraco.png)

This Windows based CTF has us enumerate a website running the Umbraco CMS system to find our initial system access, and then manually enumerate the system after connecting via RDP to find admin credentials.

[**Anthem Walkthrough**](walkthroughs/2022-04-10-Anthem.md)

<br>

---

<br>

## SANS OSINT Summit 2022
#### April 7th 2022

Topics: ![](images/learning.png)

I attended the all day OSINT summit hosted by SANS today and it was such a fun and great day of learning. You can see from the list of topics and speakers that there were a wide variety of OSINT areas covered, as well as some truly exceptional speakers.

At the end of her talk [**Alethe Denis**](https://twitter.com/AletheDenis) had a mini OSINT challenge that she gave to everyone attending where you had to search out her previous jobs and private message her the 2nd job she ever had, as well as where it was located.

**I was the first person to complete the challenge!**

Unexpected given I'm still a beginner with OSINT and the conference had veteran industry pros attending. So either they weren't competing, or I got lucky, but either way it was a lot of fun!

Thanks to **SANS** for hosting this wonderful event and I look forward to next years!

![](images-other/SANS-OSINT-Summit-2022.png)

<br>

---

<br>

## *Blueprint* on TryHackMe
#### April 6th 2022

Topics: ![](images/windows.png) ![](images/web.png) ![](images/oscommerce.png) ![](images/sshtunnel.png) ![](images/mimikatz.png)

In this CTF we get to poke around osCommerce to obtain a shell, and then use mimikatz to pull credentials out of memory.

[**Blueprint Walkthrough**](walkthroughs/2022-04-06-Blueprint.md)

<br>

---

<br>

## *VulnNet: Roasted* on TryHackMe
#### March 31st 2022

Topics: ![](images/windows.png) ![](images/smb.png) ![](images/crackmapexec.png) ![](images/asreproast.png) ![](images/kerberoast.png) ![](images/hashcat.png) ![](images/cracking.png) ![](images/evilwinrm.png) ![](images/secretsdump.png)

I've done quite a few Linux based boxes and am starting to feel more and more comfortable in that realm, but haven't done nearly as many Windows boxes. Because of that I'm going to start getting out of my comfort zone and working on Windows boxes to increase my skills there.

[**VulnNet: Roasted Walkthrough**](walkthroughs/2022-03-31-VulnNet-Roasted.md)

<br>

---

<br>

## Antisyphon - Intro to Social Engineering
#### March 29th 2022

Topics: ![](images/learning.png)

I completed my first Antisyphon class today and it was very well done. I can see why their classes have been so highly praised and recommended!

Ed Miro ([**@c1ph0r on Twitter**](https://twitter.com/c1ph0r)) taught us the basics about social engineering in a fun and engaging way and covered the following topics:

- Common attack vectors
- Good foundational books to get started with
- Influence
- Nonverbal communication
- Effective pretexting
- Sock puppets
- Red flags to look out for if you think somebody is trying to target you

He also did live demos for:

- SEToolkit
- Metasploit
- GoPhish

<br>

---

<br>

## *Aratus* on TryHackMe
#### March 26th 2022

Topics: ![](images/linux.png) ![](images/smb.png) ![](images/python.png) ![](images/scripting.png) ![](images/cracking.png) ![](images/tcpdump.png) ![](images/ansible.png)

Brand new box that just came out yesterday and I had a lot of fun going through this one.

[**Aratus Walkthrough**](walkthroughs/2022-03-26-Aratus.md)

<br>

---

<br>

## *Pandora* on Hack the Box
#### March 23rd 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/snmp.png) ![](images/snmp-check.png) ![](images/sshtunnel.png) ![](images/pandorafms.png) ![](images/cve.png) ![](images/restrictedshell.png) ![](images/suid.png)

This one took awhile to finish with 2 lateral movements, both an internal and external webpage, and other things to trip you up along the way such as restricted shells. I had a great time going through it though!

This was also the last easy rated active machine on HTB that I needed to finish, so they're all complete for now until the next one is released.

No walkthrough since this box isn't officially retired yet.

<br>

![](images-other/htb-pandora.png)

<br>

---

<br>

## *Napping* on TryHackMe (TOP 10 FINISH at #9!!!)
#### March 19th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/html.png) ![](images/tabnabbing.png) ![](images/mysql.png) ![](images/python.png) ![](images/scripting.png) ![](images/breakout.png)

[**Napping Walkthrough**](walkthroughs/2022-03-19-Napping.md)

<br>

---

<br>

## *Backdoor* on Hack the Box
#### March 17th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/wordpress.png) ![](images/lfi.png) ![](images/gdbserver.png) ![](images/msfvenom.png) ![](images/gdb.png)

This was an interesting box with several vulnerabilities that required a bit more enumeration than other boxes I've done up to this point. Especially the part where you had to figure out what was running on port 1337. I definitely enjoyed this one!

No walkthrough since this box isn't officially retired yet.

<br>

![](images-other/htb-backdoor.png)

<br>

---

<br>

## *Validation* on Hack the Box
#### March 14th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/burp.png) ![](images/sqli.png) ![](images/docker.png)

[**Validation Walkthrough**](walkthroughs/2022-03-14-Validation.md)

<br>

---

<br>

## *Source* on TryHackMe
#### March 13th 2022

Topics: ![](images/linux.png) ![](images/webmin.png) ![](images/cve.png)

[**Source Walkthrough**](walkthroughs/2022-03-13-Source.md)

<br>

---

<br>

## Level 12 on TryHackMe
#### March 11th 2022

Topics: ![](images/learning.png)

I advanced to level 12 on TryHackMe today! Out of almost a million users I'm currently #5518 in the overall rankings.

<br>

![](images-other/tryhackme-level12.png)

<br>

---

<br>

## *Cyborg* on TryHackMe
#### March 10th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/cracking.png) ![](images/borgarchive.png) ![](images/sourcecode.png)

[**Cyborg Walkthrough**](walkthroughs/2022-03-10-Cyborg.md)

<br>

---

<br>

## *Oh My WebServer* on TryHackMe
#### March 7th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/cve.png) ![](images/docker.png)

[**Oh My WebServer Walkthrough**](walkthroughs/2022-03-07-Oh-My-WebServer.md)

<br>

---

<br>

## 90 Day Badge on TryHackMe!
#### March 6th 2022

Back in December I set a goal to take learning much more seriously, and today I hit a milestone. 90 days in a row of learning something new without missing a single day on TryHackMe! Make learning a daily habit.

<br>

![](images-other/thm-90-day-badge.png)

<br>

---

<br>

## *RouterSpace* on Hack the Box
#### March 3rd 2022

Topics: ![](images/linux.png) ![](images/androidapk.png) ![](images/emulation.png) ![](images/burp.png) ![](images/cve.png)

This was a pretty frustrating box because of the Android emulation. I ran into problem after problem that I had to look up fixes for in order to be able to install and launch the .apk file. But in the end I got it, and boy was I glad when it was finally done!

No walkthrough since this is a brand new box that isn't officially retired yet.

<br>

![](images-other/htb-routerspace.png)

<br>

---

<br>

## *Return* on Hack the Box
#### February 25th 2022

Topics: ![](images/windows.png) ![](images/web.png) ![](images/cve.png)

[**Return Walkthrough**](walkthroughs/2022-02-25-Return.md)

<br>

---

<br>

## *Archangel* on TryHackMe
#### February 22nd 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/lfi.png) ![](images/burp.png) ![](images/sourcecode.png) ![](images/logpoisoning.png) ![](images/scripting.png)

[**Archangel Walkthrough**](walkthroughs/2022-02-22-Archangel.md)

<br>

---

<br>

## *Plotted-TMS* on TryHackMe
#### February 19th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/decoding.png) ![](images/sqli.png) ![](images/mysql.png) ![](images/cracking.png) ![](images/scripting.png)

[**Plotted-TMS Walkthrough**](walkthroughs/2022-02-19-Plotted-TMS.md)

<br>

---

<br>

## *Chocolate Factory* on TryHackMe
#### February 16th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/cracking.png) ![](images/burp.png) ![](images/steg.png)

[**Chocolate Factory Walkthrough**](walkthroughs/2022-02-16-Chocolate-Factory.md)

<br>

---

<br>

## *Mustacchio* on TryHackMe
#### February 15th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/cracking.png) ![](images/burp.png) ![](images/xxe.png)

[**Mustacchio Walkthrough**](walkthroughs/2022-02-15-Mustacchio.md)

<br>

---

<br>

## *Boiler* on TryHackMe
#### February 13th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/decoding.png) ![](images/cracking.png) ![](images/sar2html.png) 

[**Boiler Walkthrough**](walkthroughs/2022-02-13-Boiler.md)

<br>

---

<br>

## *Gallery* on TryHackMe
#### February 12th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sqli.png) ![](images/mariadb.png)

[**Gallery Walkthrough**](walkthroughs/2022-02-12-Gallery.md)

<br>

---

<br>

## *Secret* on Hack the Box
#### February 10th 2022

Topics: ![](images/linux.png) ![](images/api.png) ![](images/json.png) ![](images/jwt.png) ![](images/git.png) ![](images/sourcecode.png) ![](images/burp.png) ![](images/coredump.png)

This was pretty challenging for an easy rated box and it took awhile and a lot of google searching to figure out how to do some things that were new to me like using curl to interact with an API, forge JWT tokens, create and interact with core dumps to pull information out of memory, and more.

Again, no walkthrough since this box is not officially retired.

<br>

![](images-other/htb-secret.png)

<br>

---

<br>


## *Internal* on TryHackMe
#### February 9th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sqlmap.png) ![](images/wordpress.png) ![](images/wpscan.png) ![](images/phpmyadmin.png) ![](images/mysql.png) ![](images/sshtunnel.png) ![](images/hydra.png) ![](images/jenkins.png)

[**Internal Walkthrough**](walkthroughs/2022-02-09-Internal.md)

<br>

---

<br>

## *Paper* on Hack the Box
#### February 7th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/wordpress.png) ![](images/lfi.png)

This is my first actual HTB machine other than the starting point so it was a lot of fun comparing how things are on HTB vs how they are on THM. HTB definitely holds your hand less so the difficulty I'd say is a bit higher than comparably rated machines on THM.

I also went through this with somebody from my **Evolve** bootcamp and when you see the term **I** you can take it as **we** since we both spent several hours in Discord banging our heads against a wall figuring out each piece of the puzzle. Despite that we loved every second of it!

I won't post the actual walkthrough yet since this box is only 2 days old and it's against the HTB terms of service, but here's a completion screenshot:

<br>

![](images-other/paper16.png)

<br>

---

<br>

## Level 11 on TryHackMe
#### February 6th 2022

Topics: ![](images/learning.png)

Made it to level 11 finally. I've learned so much from this site already, and there's so much more to learn!

![](images-other/tryhackme-level11.jpg)

<br>

---

<br>

## *Relevant* CTF on TryHackMe
#### February 4th 2022

Topics: ![](images/windows.png) ![](images/smb.png)

[**Relevant Walkthrough**](walkthroughs/2022-02-06-Relevant.md)

<br>

---

<br>

## Week 1 of the Evolve Bootcamp Complete
#### February 4th 2022

Topics: ![](images/learning.png)

I couldn't be happier with how well the first week went. The lead instructor, Michael Creaney, is not only extremely knowledgeable but is good at teaching as well. Sometimes you get one or the other, but not this time thankfully.

The assistant instructors are also very good and seem like they truly care. They're all Evolve alumni so they've been through the bootcamp and can help out with any general questions we have in addition to providing insight on what we're learning.

So far so good!

<br>

---

<br>

## *Wonderland* CTF on TryHackMe
#### February 4th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/steg.png) ![](images/scripting.png) 

[**Wonderland Walkthrough**](walkthroughs/2022-02-04-Wonderland.md)

<br>

---

<br>

## Today is the day my Evolve Security Bootcamp Starts!
#### January 31st 2022

Topics: ![](images/learning.png)

About 3 months ago I finished some local college courses in their Cybersecurity learning path, and it turned out that the next step in the path wasn't available in the spring semester and I'd have to wait many months until the summer semester to be able to continue on. That was too long to wait so I decided to look around for other learning opportunities.

There were several choices but I eventually ended up seriously looking at the bootcamp that **Evolve Security** offered. It had great reviews, they seemed to be well respected in the industry, and many other features such as help with job placement, a voucher for the Security+ certification at the end of the bootcamp, and other bonuses.

It was a definitely a bit pricey, but in the end I believed it would be well worth the cost given everything I found during my research. So I took the plunge and officially signed up.

Today, it finally starts! It's going to be a lot of work with classes meeting 4 days a week and every other Saturday, which equates to roughly 20-30 hours of work per week split between in class work, out of class studying, and hands on labs.

I'm excited and can't wait to get started!

<br>

---

<br>

## *GamingServer* CTF on TryHackMe
#### January 30th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sshkey.png) ![](images/cracking.png) ![](images/lxc.png) ![](images/container.png)

[**GamingServer Walkthrough**](walkthroughs/2022-01-30-GamingServer.md)

<br>

---

<br>

## *Corp* CTF on TryHackMe
#### January 28th 2022

Topics: ![](images/windows.png) ![](images/applocker.png) ![](images/powershell.png) ![](images/kerberos.png) ![](images/cracking.png) ![](images/decoding.png)

[**Corp Walkthrough**](walkthroughs/2022-01-28-Corp.md)

<br>

---

<br>

## *Retro* CTF on TryHackMe
#### January 27th 2022

Topics: ![](images/windows.png) ![](images/web.png) ![](images/wordpress.png) ![](images/mysql.png)

[**Retro Walkthrough**](walkthroughs/2022-01-27-Retro.md)

<br>

---

<br>

## *Game Zone* CTF on TryHackMe
#### January 25th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sqli.png) ![](images/sqlmap.png) ![](images/burp.png) ![](images/mysql.png) ![](images/cracking.png) ![](images/sshtunnel.png) ![](images/metasploit.png) 

[**Game Zone Walkthrough**](walkthroughs/2022-01-25-Game-Zone.md)

<br>

---

<br>

## *Anonymous* CTF on TryHackMe
#### January 24th 2022

Topics: ![](images/linux.png) ![](images/smb.png) ![](images/ftp.png) ![](images/scripting.png) ![](images/suid.png)

[**Anonymous Walkthrough**](walkthroughs/2022-01-24-Anonymous.md)

<br>

---

<br>

## *Crack the hash* CTF on TryHackMe
#### January 21st 2022

Topics: ![](images/linux.png) ![](images/hash.png) ![](images/cracking.png)

[**Crack the hash Walkthrough**](walkthroughs/2022-01-21-Crack-the-hash.md)

<br>

---

<br>

## *Pickle Rick* CTF on TryHackMe
#### January 20th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/sudo.png)

[**Pickle Rick Walkthrough**](walkthroughs/2022-01-20-Pickle-Rick.md)

<br>

---

<br>

## *Inclusion* CTF on TryHackMe
#### January 20th 2022

Topics: ![](images/linux.png) ![](images/web.png) ![](images/lfi.png) ![](images/sudo.png)

[**Check out my first walkthrough**](walkthroughs/2022-01-20-Inclusion.md) of the **Inclusion** CTF on TryHackMe.

<br>

---

<br>

## *Hello World*
#### January 31st 2022

Hello everyone who may have stumbled across this site and thank you for visiting. I'm creating this primarily as a place to post walkthroughs for CTF challenges I complete as a way to:

- Reinforce my learning
- Get in the habit of documenting everything
- And give back the community / help those who are learning as well

I'll try and publish new walkthroughs at least once a week but will often add more than that.

I'm currently learning with the eventual goal of career switching into Cybersecurity so I'll also sometimes add accomplishments or other Cybersecurity related items to this blog as well.

If you have questions, comments, or would like to set something like this up yourself and need a little help there please feel free to get in touch with me on Twitter - [**@jdotinfo**]

Again, thanks for visiting and I hope you find this resource useful!
