---
title: Marc O'Polo and United Cinemas International Fix XSS Security Flaws
categories: 
- Coordinations
---
Another day, some new XSS flaws. At first the big fashion label [Marc O'Polo](http://www.marc-o-polo.com) fixed a major Cross-Site Scripting issue in their online shop system. Good news, because a malicious attacker was able to use this security hole to hijack (and steal) every account including all personal customer details. I'm glad to see that this has been fixed :-)  
![ia45]({{ site.baseurl }}/assets/ia45.png)  
Additionally the [United Cinemas International](http://www.uci-kinowelt.de) (also known as UCI in Germany) fixed a bigger Cross-Site Scripting flaw, which could be abused on every single page of the website to steal user credentials and e.g. inject malicious iframes for further browser exploitation or...a little better... "anonymously" buying cinema tickets :-D.![ia48]({{ site.baseurl }}/assets/ia48.png)  
The quite interesting part here is that both companies did not respond to any of my official mails - probably the reason why both flaws were open 24/7 for around half a year - I found both in June 2012 !  Then ...by the end of 2012 I tried to contact both companies using Facebook - and both responded within only a few days. Awesome! I should report more flaws using Facebook, looks like this is the most appreciated way of communication.
