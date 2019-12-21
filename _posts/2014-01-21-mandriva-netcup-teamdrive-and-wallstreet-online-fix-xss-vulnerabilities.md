---
title: Mandriva, Netcup, Teamdrive and Wallstreet-Online Fix XSS Vulnerabilities
categories:
- Coordinations
---

It's 2014 and I have to tidy up my discovery archive a bit ;-) . Before joining the [Internetwache.org](http://www.internetwache.org) project I have coordinated all found vulnerabilities by myself and these are the last ones which have been fixed in late 2013\. All further website-based vulnerabilities will be released over at our project page - because I'm going to focus on application security here - except for bug bounty discoveries :-)

I think I do not need to say anything about the risks of Cross-Site Scripting anymore - all following issues may have led to account hijacking and data theft on each website. Let's just quickly summarize and rank the different findings based on the reaction time and professionalism.

### 1. [Netcup](http://www.netcup.de)

A Germany-based hoster who offers quite cheap servers and web-space with a great support. The reaction of Netcup was the fastest one of all: only 1 day for a friendly and thankful reaction and only 3 further days to a quite nicely hardened system with a serious blacklisting function that locks attacked accounts. Great job - that's the way I like it :-) !

[![netcup-xss]({{ site.baseurl }}/assets/netcup-xss-1024x444.png)]({{ site.baseurl }}/assets/netcup-xss.png)

### 2. [Teamdrive](http://www.teamdrive.com/)

Another Germany-based cloud service interested in security. After two failed coordination attempts, the issue has been fixed within 3 weeks.

[![teamdrive-xss]({{ site.baseurl }}/assets/teamdrive-xss-1024x482.png)]({{ site.baseurl }}/assets/teamdrive-xss.png)

### 3. [Wallstreet-Online](http://www.wallstreet-online.de/)

Ok, this is an interesting story since you would assume if "money" is in the game the administrators will act quickly. Negative. Although the reaction was quite fast (2 days), I've never heard again from them - even after asking for a status update several times. After I tested (more-or-less accidentally) the page again, I noticed that the issue has been fixed - resulting in an exploitable XSS open for at least half a year.

[![wallstreet-xss]({{ site.baseurl }}/assets/wallstreet-xss-1024x359.png)]({{ site.baseurl }}/assets/wallstreet-xss.png)

### 4. [Mandriva](http://www.mandriva.com)

And ranked last: Mandriva's online store suffered from **_A LOT_** of XSS issues! First contacted in August 2012 via mail -> no response, tried again several times --> no response. Then I tried to contact them using Facebook --> response received after waiting for 3 weeks. I received a ticket ID...and...again --> no response. As the issues were exploitable after half a year, I asked again for a status --> no response. Then in late 2013 they've relaunched their online store (now called "ServicePlace") and fixed all issues with that...and guess: without a notification. So all in all the Mandriva shop was exploitable for nearly 1,5 years. Really sad story.

[![mandriva-xss]({{ site.baseurl }}/assets/mandriva-xss-1024x511.png)]({{ site.baseurl }}/assets/mandriva-xss.png)
