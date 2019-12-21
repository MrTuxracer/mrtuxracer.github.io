---
title: OSCP Course and Exam Review
categories:
- Certifications
---

As you may have noticed - it went quiet on my blog in the last few weeks. I was heavily working on the challenging [Offensive-Security](http://www.offensive-security.com) Labs to obtain my [**O**ffensive-**S**ecurity **C**ertified **P**rofessional](http://www.offensive-security.com/information-security-certifications/oscp-offensive-security-certified-professional/) (OSCP) certification. AND ! Yesterday! I received the mail from Offensive-Security that I have successfully completed all requirements for the OSCP certification! I'm really happy about that because it opens a new door in my career :-) ! [![cert-logo-oscp]({{ site.baseurl }}/assets/cert-logo-oscp.png)]({{ site.baseurl }}/assets/cert-logo-oscp.png)  
**But what's so special about this certification ?**

A quote from the Offensive-Security website summarizes it best:

> The OSCP certification, in my opinion, proves that it’s holder is able to identify vulnerabilities, create and modify exploit code, exploit hosts, and successfully preform tasks on the compromised systems over various operating systems

After completing my [eCPPT exam](https://www.rcesecurity.com/2012/03/ecppt-course-and-exam-review/ "eCPPT Course and Exam review"), which is more an entry-level certification to web-application security, I decided to take the OSCP course, because there are a lot of good and interesting reviews about its strengths over at [ethicalhacker.net](http://www.ethicalhacker.net/). Ok, I have to admit, I've not been 100% sure if I really should take this course - even at the point while clicking on "Submit payment" ;-), because there are a lot of reviews out there which say that it is real pain! (More about this later).

Basically the OSCP Course (well officially it's called PWB - Pentesting with Backtrack) is completely different to the eCPPT. It's a "real" network penetration testing course where you start with information gathering and end up in local privilege escalation to take over root or SYSTEM rights. An overview about the course syllabus can be found [here](http://www.offensive-security.com/documentation/penetration-testing-with-backtrack.pdf).

Since most of the web-vector attack techniques have already been covered by my eCPPT work, I focused more on other parts of the course, like:

*   Manually m<span style="line-height: 13px;">odifying existing exploits under Linux and Windows - Yes, I do not mean Metasploit!</span>
*   Transferring files to target systems - ..there are ways I did not even think about!
*   Client - side attacks - Most common these days - Praise JavaScript :-D
*   "Port Fun" - Redirecting and Tunneling traffic through network segments - Sometimes common firewalls are just useless when there are no outgoing rules...

And the most interesting =  
![oscp-esc]({{ site.baseurl }}/assets/oscp-esc.png)
Privilege Escalation - The "GOT ROOT" messages feels like turning Godmode on ;-) !!!

**The Courseware!**

A lot! About 330 pages of pure written PDF and endless hours of video material. Great stuff - nothing more to say, but no pain until here.

**The Labs!**

There are a lot of lab machines which reside in different firewalled network - segements, like they are common in most real network-scenarios, which I daily encounter at my customer sites.

It started quite easy with some older vulnerabilities, that directly resulted in SYSTEM level access. But it was getting harder. I started with pwning the first machines - in only a few hours I got around 30% of the "public" network segment. I thought: "Well if this is the ongoing niveau, I should cancel this course, try to get my money back, and try to comment every review I've read so far.

But it was still getting harder - even in the public segment. The variety of vulnerabilities grew and most of them did only result in a limited shell. I needed to think with increased regularity about the Offensive - Security motto "Try Harder™". Until I got to a box called "PAIN". It was quite easy to get shell access to this machine, but then the problems start. I did not find anything to further escalate my privileges. Nothing. I searched for about 2 days and found....nothing - I felt like I really suck and I asked myself for one moment if this is the right way in my career :-D !

Until the point I started to think in a different way - a more linux-like way - about the more common things on a linux system (Sorry I don't want to disclose too many details, because I do not want to destroy your fun) and found the key to the /root/ directory!! The first point where I felt like working with Godmode turned on :-)

I went on and I ended in having all hosts pwned except one called: "SUFFERENCE". The next generation of pain. I was working for nearly 3 days on this box but did not find a way to the root - but wait...after my exam I got an idea about how to crack it...

The other network segments were quite funny too...In the end I was able to pwn 45 out of 49 hosts to SYSTEM / Root - Level, a great result in my opinion :-)

**The PAIN !**

Although I have pwned 45 hosts, I did not feel - somehow - ready for the final exam - challenge. But unfortunately my labtime had come to an end. The final exam challenge is a Capture-The-Flag (CTF) style real-world scenario, which you need to exploit in order to obtain your certification. You've got 24 hours to complete the CTF and another 24 hours to write and hand over the documentation.

OK, I have scheduled my CTF on a saturday afternoon, and had a lot of sleep before to be ready for what was coming. I received the mail with my instuctions and was a bit astonished. Less machines then I had expected and one special challenge. Pwning a host gives you a different number of points (all together: 100 points), and you need at least 70 points to pass. Because of the small number of hosts, I started to think: OK - that's going to be challenging - and it was. The usage of Metasploit is very, very limited, which is great in my opinion, because using semi-automated tools for penetration testing purposes do not show that you have understood what you are doing.

My new strategy: Pwn machines to get the minimum number of points :-) I have completed the special challenge, which was by the way a really **great idea** and I completed another host quite quickly. This meant 50 points. Great - I thought.

But the PAIN started to return at this point. I was working on the next host - without seeing any results...for hours!! 8 hours passed by, and I started to get nervous. But for good: I took another Clubmate and left my flat to re-organize my thoughts. While pwning my Clubmate I've been hit by another idea. Back at the CTF, I had a deeper look at a special configuration condition on the host, which attracted my attention. AND! FINALLY! I was able to pwn the host to ROOT. That resulted in the minimum amount of points to pass. I decided to go to bed at this point. That was a good decision. The next monring I was able to pwn all other systems to complete the certification with 100 of 100 points :-)

The documentation part wasn't a big problem, since it was perfectly teached in the eCPPT course and therefore easy to do for me (my documentation: 291 pages including appendix)

**Who should take this course and who not?**

Although it's one of the more expensive courses - it's a very good investment into your penetration testing career. But it is not an entry-level course! If you're new to networking or security in general - this course could hit you to the ground. The keypoints to survive this challenge:

1.  Information gathering! You need a moderate understanding of network and system concepts - especially on linux based systems.
2.  Documentation. You have to submit a fully detailed documentation about your course **AND **exam findings.
3.  Without ever having coded/scripted one line by yourself, you're completely lost.

After this awesome challenge, I decided to take the advanced OSCE...next year :-)
