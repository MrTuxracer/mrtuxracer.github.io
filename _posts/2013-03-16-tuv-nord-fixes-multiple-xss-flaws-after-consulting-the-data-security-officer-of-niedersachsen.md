---
title: TÜV-Nord Fixes Multiple XSS Flaws after Consulting the Data Security Officer
  of Niedersachsen
categories:
- Coordinations
---
Hello readers!

Take a moment and read the following article on Wikipedia about the German [TÜV](http://en.wikipedia.org/wiki/T%C3%9CV) which is described as:

> TÜVs (German pronunciation: [ˈtʏf]; short for German: Technischer Überwachungs-Verein, English: Technical Inspection Association) are German organizations that work to validate the safety of products of all kinds to protect humans and the environment against hazards. As independent consultants, they examine plants, motor vehicles, energy installations, amusement rides, devices and products (e.g. consumer goods) which require monitoring. The many subsidiaries of the TÜVs can also act as project developers for energy and traffic concepts, as problem solvers in environmental protection, and as certification bodies. Many of the TÜV organizations also provide certification for various international standards, such as ISO9001:2008 (quality management system) and ISO/TS16949 (automotive quality management system).

Now further imagine, you report several security issues (in July 2012!) on their official website to this organazition and you receive an answer from the online marketing department saying that it is investigating the problems. So far so good, sounds like they really care about this situation. Although those security issues only affect the data security indirectly, it's still a security problem (if exploited) to all available customer information, like the famous account hijacking - I think I do not need to further say what is possible with this kind of holes on a website where you can enter personal information into account.

Then...nothing happens, I tried to contact them several more times during **the first half year** of the presence of these exploitable XSS security holes, but did not receive any kind of further reaction. Interesting point since I had already an personal contact who answered my report (and it wasn't some kind of [Photodex-like-Bot](https://www.rcesecurity.com/2012/08/proshow-producer-still-vulnerable-after-two-updates/) :-D). You may ask yourself ( like I did): wtf ?!?

After trying to contact the organization via other mail addresses without any reactions, I decided to consult the local country based data protection officer of  Niedersachsen where the organization resides at to bring more attention to this situation. After only a few days I received an answer to my request and the data protection officer started to do his job :-)

The most interesting fact: After consulting the data protection officer, all flaws have been fixed in only some weeks, which is quite fast for such a big organization! Looks like this is the new way to go... :-)

Really! Trust me! I don't want to blame anyone here, but an organization who represents "Security" in Germany is not able to manage security issues on their own website? Come on - is this the way you deal with security? If so, I should be careful when entering one of your "checked" elevators to the 11th  floor next time...

![ia37]({{ site.baseurl }}/assets/ia37.png)

Reference: [IA37](http://security.inshell.net/advisory/37)
