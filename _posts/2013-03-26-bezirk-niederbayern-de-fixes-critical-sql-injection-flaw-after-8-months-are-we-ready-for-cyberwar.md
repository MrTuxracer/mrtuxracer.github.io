---
title: Bezirk-Niederbayern.de Fixes Critical SQL-Injection Flaw After 8 Months - Are
  We Ready For CyberWar?
categories:
- Coordinations
---
That's amazing bad. Where should I start? In July 2012 I've reported a critical SQL - Injection flaw on the official website of Lower Bavaria alongside another small XSS flaw to the owner of the website. The answer did not take that long asking for further details of the flaw and how to exploit it. So far so good, I handed over all necessary information on both flaws.

![ia36-2]({{ site.baseurl }}/assets/ia36-2.png)

![ia36-1]({{ site.baseurl }}/assets/ia36-1.png)

I thought they would start investigating the issue, since the flaw allowed an attacker to gain complete access to the backend - database including all information and probably over the filesystem too. For legal reasons I did not test that deep. That's like an open invitation to every blackhat to deface it or spread malware etc.

I asked several times if any help is needed to further investigate and fix the issue, and received the answer, that no help is needed at this point. Great! After over a month I contacted the official press officer again, asking for an ETA on the patch since the flaw is still world-wide-open. The answer was astonishing:

> Our IT department states that our systems are secure.

Well. No they aren't! Still exploitable. Still hackable. Still pwnable. So I asked for a direct contact in the IT department to clarify things - otherwise it's like whispering down the lane. If you're security addicted, you should stop reading here, since the answer will make you cry:

> There is no need to contact our IT department, because our website is not hosted on our own servers, so we're not having a problem here.

Time to LOL. And time for some arrogance. I tried to contact them again, but never got any further answer. Why ? The hole was still open to exploit. Time to put some heavy armaments into game. I forwarded the whole story to the Data Protection Officer of Bavaria who started to do his job with quite a fast success - after only 2 months the flaws  have been fixed.

Everyone is talking about CyberWar in the public TV, and Germany is planning to establish an own so-called "Cyber - Army". But now, ask yourself the following: Don't we need to protect our own assets first ? If we're not able - due to arrogance or ignorance reasons - to even protect our country websites from the **BASIC **attack types, what are the legit reasons for establishing such an army then ? Offense over Defense ? To hype the word "CyberWar" ?

Another example needed? Read [this](http://www.sicherheit-online.org/261/bundesagentur-fuer-arbeit-beluegt-bfdi-zu-sicherheitsluecken-und-angriffen/) interesting article from Sicherheit-Online.org.

p.s.

The quotes are translated from German. If a reporter is interested in the whole uncut story, just leave me a message.
