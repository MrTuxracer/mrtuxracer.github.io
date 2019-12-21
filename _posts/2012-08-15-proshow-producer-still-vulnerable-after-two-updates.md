---
title: ProShow Producer still vulnerable after two updates!
categories:
- Advisory
---
[This](http://security.inshell.net/advisory/30) is quite a sad story and also a perfect example of the ignorance or maybe arrogance of many software vendors. I've reported the Buffer Overflow vulnerability to the vendor named "Photodex" and also received an answer, which sounds like they start to work on the issue:

> <pre>Thank you for contacting Photodex. I've sent a report of this 
> situation down the line to the appropriate departments so that 
> they can take a closer look.  We appreciate your time and 
> patience in this matter.</pre>

but then...I did not receive any further messages. Absolutely nothing. After a while I decided to publish the issue the [usual ways](http://seclists.org/fulldisclosure/2012/Jul/15) and noticed them again about the issue including a direct link to the Full-Disclosure Mailinglist.

The answer ? Quite a funny one (am I talking to a bot ?!?!):

> <pre>Thanks so much for that feedback.  Let us know if you need anything else and we will certainly assist.</pre>

In the meanwhile some cool guys over at Metasploit have created a [working exploit](http://www.metasploit.com/modules/exploit/windows/fileformat/proshow_load_bof) for their framework, which allows to exploit the issue on all actual operating systems (Windows XP SP3 / Windows 7 SP1 (default)). Good work guys - I like your work ;-) !

Photodex released two further updates bringing the ProShow Producer to version 5.0.3280 including the same vulnerability which they did not touch during the update process:

[![]({{ site.baseurl }}/assets/proshow_5_0_3280.png "proshow_5_0_3280")]({{ site.baseurl }}/assets/proshow_5_0_3280.png)

And by the way: You may also have noticed the price of the ProShow Producer software as stated on their website:

> **Full version** _from $249.95_

They sell their software for about 250 bucks per license but do not even care about their product security and therefor customer security?! Now you can simply throw the money out of the next window or buy another product from a more security-aware [company](http://www.adobe.com). May the force be with you :-) !
