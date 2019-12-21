---
title: About a Sucuri RCE...and How Not to Handle Bug Bounty Reports
categories:
- BugBounty
- RCE
---
### TL;DR
Sucuri is a self-proclaimed "most recommended website security service among web professionals" offering protection, 
monitoring and malware removal services. They **ran** a Bug Bounty program on HackerOne and also blogged about [how important security 
reports are](https://blog.sucuri.net/2015/02/vulnerability-disclosures-a-note-to-developers.html). While their 
program was still active, I've been hacking on them quite a lot which eventually **ranked me #1** on their program. 

By the end of 2017, I have found and reported an explicitly disabled SSL certificate validation in their 
server-side scanner, which could be used by an attacker with MiTM capabilities to execute arbitrary code on Sucuri's 
customer systems. 

The result: Sucuri provided me with an initial bounty of 250 USD for this issue (they added 500 USD later due to a 
misunderstanding on their side)  - out of an announced 5000 USD max bounty, fixed the issue, closed my report as 
informative and went completely silent to apparently prevent the disclosure of this issue.

**Every Sucuri customer who is using the server-side scanner and who installed it on their server <span style="color:red">
before June 2018</span> should immediately upgrade the server-side scanner to the most recent version which fixes this 
vulnerability!**

### SSL Certificate Validation is Overrated
As part of their services, Sucuri offers a custom [server-side scanner](https://kb.sucuri.net/monitoring/Server+Side+Scanner/server-side-scan), which customers can place on their servers
and which runs periodic scans to detect integrity failures / compromises. Basically the server-side scanner is just a 
custom PHP script with a random looking filename of i.e. ```sucuri-[md5].php``` which a customer can place on 
their webserver. 

**NOTE:**
Due to a copyright notice in the script itself, I cannot share the full server-side scanner script here, but will
use pseudo-code instead to show its logic. If you want to play with it by yourself, register an account with them
and grab the script by yourself ;-)

{% highlight php %}
<?php
$endpoint = "monitor2";
$pwd = "random-md5";

if(!isset($_GET['run']))
{
    exit(0);
}

if(!isset($_POST['secret']))
{
    exit(0);
}

$c = curl_init();
curl_setopt($c, CURLOPT_URL, "https://$endpoint.sucuri.net/imonitor");
curl_setopt($c, CURLOPT_POSTFIELDS, "p=$pwd&amp;q=".$_POST['secret']); 
curl_setopt($c, CURLOPT_SSL_VERIFYPEER, false);
$result = curl_exec($c);

$b64 =  base64_decode($result);

eval($b64);
?>
{% endhighlight %}

As soon as you put the script in the web root of your server and configure your Sucuri account to perform server-side
scans, the script instantly gets hit by the ``Sucuri Integrity Monitor`` with an HTTP POST request targeting 
the ``run`` method like the following:

![]({{ site.baseurl }}/assets/sucuri-mitm-rce-1.png)

This HTTP POST request does also include the ``secret`` parameter  as shown in the pseudocode above and basically triggers
a bunch of IP validations to make sure that only Sucuri is able to trigger the script. Unfortunately this part is flawed 
as hell due to stuff like:

{% highlight php %}
$_SERVER['REMOTE_ADDR'] = $_SERVER['HTTP_X_FORWARDED_FOR']
{% endhighlight %}

(But that's an entirely different topic and not covered by this post.) 

By the end of the script, a curl request is constructed which eventually triggers a callback to the Sucuri monitoring
system. However, there is one strange line in the above code:
 
{% highlight php %}
curl_setopt($c, CURLOPT_SSL_VERIFYPEER, false);
{% endhighlight %}
 
So Sucuri explicitly set ``CURLOPT_SSL_VERIFYPEER`` to ``false``. The consequences of this are [best described by the curl project itself](https://curl.haxx.se/libcurl/c/CURLOPT_SSL_VERIFYPEER.html):
> WARNING: disabling verification of the certificate allows bad guys to man-in-the-middle the communication without you 
> knowing it. Disabling verification makes the communication insecure. Just having encryption on a transfer is not 
> enough as you cannot be sure that you are communicating with the correct end-point. 

So this is not cool. 

The issued callback doesn't contain anything else than the previously mentioned secret and looks like the following:

![]({{ site.baseurl }}/assets/sucuri-mitm-rce-2.png)

The more interesting part is actually the response to the callback which contains a huge base64 string prefixed by the 
string ``WORKED:``:

![]({{ site.baseurl }}/assets/sucuri-mitm-rce-3.png)

After decoding I noticed that it's simply some PHP code which was generated on the Sucuri infrastructure to do the actual
server-side scanning. So essentially a Man-in-the-Middle attacker could simply replace the base64 string with his own 
PHP code just like ``c3lzdGVtKCJ0b3VjaCAvdG1wL3JjZSIpOw==`` which is equal to ``system("touch /tmp/rce");``:

![]({{ site.baseurl }}/assets/sucuri-mitm-rce-4.png)

Which finally leads to the execution of the arbitrary code on the customer's server:

![]({{ site.baseurl }}/assets/sucuri-mitm-rce-5.png)

### How Not to Handle Security Reports
This is actually the most interesting part, because communicating with Sucuri was a pain. Since there have been
a lot of communication back and forth between me, Sucuri and HackerOne on different ways including the platform and email,
The following is a summary of the key events of the communication and should give a good impression about Sucuri's 
way to handle security reports.

#### 2017-11-05
I've sent the initial report to Sucuri via HackerOne (report #287580)

#### 2017-11-16
Sucuri says that they are aware of the issue but CURLOPT_SSL_VERIFYPEER cannot be enabled due to many
hosters not offering the proper libraries and the attack scenario would include an attacker having MiTM on the hoster.

MiTM is required - true. But there are many ways to achieve this, and the NSA and Chinese authorities have proven to be 
capable of such scenarios in the past. And I'm not even talking about sometimes critical compliance requirements such as
PCI DSS.

#### 2017-11-17
Sucuri does not think that a MiTM is doable:
> Think about it, If MITM the way you are describing was doable, you would be able to hijack emails from 
> almost any provider (as SMTP goes unencrypted), redirect traffic by hijacking Google's 8.8.8.8 DNS and 
> create much bigger issues across the whole internet. 

Isn't that exactly the reason why we should use TLS across the world and companies such as Google try to enforce it 
wherever possible?

#### 2017-11-17
I came up with a bunch of other solutions to tackle the "proper libraries issue":
> 1. You could deliver the certificate chain containing only your CA, Intermediates and server certificate via a separate file (or as part of your PHP file) to the customer and do the verification of the server certificate within PHP, i.e. using PHP's openssl_x509_parse().
> 2. You could add a custom method on the customer-side script to verify a signature delivered with the payload sent from monitor2. As soon as the signature does not match, you could easily discard the payload before calling eval(). The signature to be used must be - of course - cryptographically secure by design.
> 3. You could also encrypt the payload to be sent to the customer site using public-private key crypto on your side and decrypt it using the public key on the client side (rather than encoding it using base64). Should also be doable in pure PHP.

#### 2017-11-29 to 2018-05-16
Sucuri went silent for half a year, where I've tried to contact them through HackerOne and directly via email. During that
period I've also requested mediation through HackerOne.

#### 2018-06-07
Suddenly out of the blue Sucuri told me that they have a fix ready for testing.

#### 2018-06-21
Sucuri rewards the minimum bounty of 250 USD because of:
> 1. A successful exploitation only works if a malicious actor uses network-level attacks (resulting in MITM) against the hosting server (or any of the intermediary hops to it) to impersonate our API. While in theory possible, this would require a lot of efforts for very little results (in term of the amount of sites affected at once versus the capacity required to conduct the attack). The fact we use anycast also doesn't guarantee a BGP hijacking attack would be successful.
> 2. The server-side scanner file contains a unique hash for every single site, which is an information the attacker would also need in order to perform any kind of attack against our customers.

#### 2018-07-18
Sucuri adds an additional 500 USD to the bounty amount because they apparently misunderstood the signature 
validation point.

#### 2018-09-15
I've requested to publicly disclose this issue because it was of so low severity for Sucuri, they shouldn't have a 
problem with publicly disclosing this issue.

#### 2018-10-12
A couple of days right before the scheduled disclosure date: Sucuri reopens the report and changes the report status to 
Informative without any further clarification. No further reply on any channel from Sucuri. That's where they went silent 
for the second time.

#### 2018-11-23
I've followed up with HackerOne about the whole story and they literally tried everything to resolve this issue by 
contacting Sucuri directly. HackerOne told me that Sucuri will close their program and the reason for the status change 
was to address some information which they feel is sensitive. 

HackerOne closes the program at their request on 2018-12-15. HackerOne even made them aware of different tools to 
censor the report, but Sucuri did not react anymore (again).

#### 2019-01-02
Agreed with HackerOne about taking the last resort disclosure option, and giving Sucuri another 180 days of additional
time to respond. They never responded.

#### 2019-06-13 to 2019-06-19
I've sent a couple of more emails directly to Sucuri (where they used to respond to) to make them aware of this blog post, 
but again: no answer at all.

#### 2019-06-20
Public disclosure in the interest of the general public according to HackerOne's last resort option.

### About HackerOne's Last Resort Option
I have tried to disclose this issue several times through HackerOne, but unfortunately Sucuri wasn't willing to
provide any disclosure timeline (have you read the mentioned blog article?) - in fact they did not even respond anymore 
in the end (not even via email) - which is why I took the last resort option after consulting with HackerOne and as per 
[their guidelines](https://www.hackerone.com/disclosure-guidelines):

> If 180 days have elapsed with the Security Team being unable or unwilling to provide a vulnerability disclosure 
> timeline, the contents of the Report may be publicly disclosed by the Finder. We believe transparency is in the 
> public's best interest in these extreme cases.
 
Since this is about an RCE affecting potentially all of Sucuri's customers who are using the server-side security scanner, 
and since there was no public or customer statement by Sucuri (at least that I am aware of) I think the general public 
deserves to know about this flaw.
