---
title: Ok Google, Give Me All Your Internal DNS Information!
categories:
- BugBounty
---
In late January, I have found and reported a Server-Side Request Forgery (SSRF) vulnerability on [toolbox.googleapps.com](https://toolbox.googleapps.com/) to Google's VRP, which could be used to discover and query internal Google DNS servers to extract all kinds of corporate information like used internal IP addresses across the company as well as A and NS records exposing all kinds of hosts like Google's Active Directory structures and also a fancy Minecraft server! :-)  So here's a quick write-up about the vulnerability.

As you might already know, the G-Suite Toolbox can be used to perform all kinds of trouble-shootings. Among all the available tools, there is one called "Dig" which - on Linux - can be used to query a DNS server for its records of a given domain, just like A- or MX records. In this case Google implemented a nice web interface for that tool to visually lookup DNS information. While it looks like a useful tool to query DNS servers from a Google perspective...

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-0.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-0.png)

...the "Name server" field probably raises the attention of every bug hunter ;-) While playing around and trying to query 127.0.0.1 for DNS records, the web application simply responds with a "Server did not respond message":

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-1.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-1.png)

So it really looks like the tool is trying to connect to 127.0.0.1:53 to fetch the DNS information for my domain  - this really smells like a Server Side Request Forgery, doesn't it?

### Ok Google, give me a responding internal DNS server!

Thanks to BurpSuite's intruder, brute-forcing for responding IP addresses can be quickly automated by instrumenting the corresponding HTTP POST "nameserver" parameter:

{% highlight html %}
POST /apps/dig/lookup HTTP/1.1
Host: toolbox.googleapps.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:51.0) Gecko/20100101 Firefox/51.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Referer: https://toolbox.googleapps.com/apps/dig/
Content-Length: 107
Cookie: csrftoken=NE5nKGrbPNRoEwm0mahDzop9iJfsxU4H; _ga=GA1.2.2102640869.1486420030; _gat=1
Connection: close

csrfmiddlewaretoken=NE5nKGrbPNRoEwm0mahDzop9iJfsxU4H&domain=www.rcesecurity.com&nameserver=§127.0.0.1§&typ=a
{% endhighlight %}

Some minutes later I've found one promising internal IP, which responded to the request - however just with an empty A record for my domain:

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-2.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-2.png)

Since I do know about my own domain very well, it's even more interesting to find out whether it's possible to extract some internal information from Google, which are not publicly available.

### Ok Google, give me your internal domain name!

Et voila. Found something [here](https://news.ycombinator.com/from?site=corp.google.com). It seems like Google is using "corp.google.com" as their corporate domain - at least some of their tools including one called "MoMa - Inside Google" is hosted under that domain. Now you could either brute force all subdomains of "corp.google.com" using the very same POST request to discover more subdomains (that's what I actually did), or just google for them, which will [reveal](https://chromium.googlesource.com/webm/libvpx/+/cbecf57f3e0d85a7b7f97f3ab7c507f6fe640a93/.mailmap) an interesting A record called "ad.corp.google.com".

### Ok Google, just give me all your A records for "ad.corp.google.com"!

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-3.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-3.png)

This looks even more interesting in comparison to what is available on public DNS records:

{% highlight bash %}
dig A ad.corp.google.com @8.8.8.8

; <<>> DiG 9.8.3-P1 <<>> A ad.corp.google.com @8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 5981
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;ad.corp.google.com.		IN	A

;; AUTHORITY SECTION:
corp.google.com.	59	IN	SOA	ns3.google.com. dns-admin.google.com. 147615698 900 900 1800 60

;; Query time: 28 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Wed Feb 15 23:56:05 2017
;; MSG SIZE  rcvd: 86
{% endhighlight %}

This is now very internal!

### Ok Google, give me the NS records (and their internal IPs) associated with that domain!

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-4.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-4.png)

### Ok Google, let's get more specific. Give me information about the "gc._msdcs"!

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-5.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-5.png)

### Ok Google, anything else you want me to see?

[![]({{ site.baseurl }}/assets/googleapps_dig_ssrf-6.png)]({{ site.baseurl }}/assets/googleapps_dig_ssrf-6.png)

After reporting this vulnerability to Google via their VRP, they quickly fixed this vulnerability. Thanks Google for the nice bounty!
