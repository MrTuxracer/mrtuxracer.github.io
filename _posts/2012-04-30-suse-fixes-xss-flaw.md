---
title: SUSE fixes XSS flaw
categories:
- Coordinations
---
![]({{ site.baseurl }}/assets/suse-poc.png "suse-poc")

Some days ago...I have found a Cross-Site Scripting Vulnerability on [www.suse.com](http://www.suse.com) - the home of the famous Linux distribution.

Using this bug, an attacker could temporarily inject arbitrary code with required user interaction into the context of the website and therefor a successful exploitation of the vulnerability allows for example cookie theft, session hijacking or (visible) client side context manipulation. This could directly lead to the execution of malicious scripts in the client browser and allows -under some circumstances - further exploitation of browser or plugin vulnerabilities, resulting in the worst case scenario: a compromised visitor system.

After the discovery, I've reported this bug directly to the support team who have immediatley fixed the issue. At this point I would like to thank **Mr. Meissner** from the SUSE - Support team for the very friendly and professional way of working and dealing with my report! I'm glad to see, that you care about the security of your visitors :-) !!
