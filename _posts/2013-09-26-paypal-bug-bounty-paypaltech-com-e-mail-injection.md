---
title: 'PayPal Bug Bounty: PayPaltech.com E-Mail Injection'
categories:
- BugBounty
---
Bag the bug! I've reported another interesting vulnerability to the PayPal site security team in May 2013 affecting their domain www.paypaltech.com, which is in scope of the official Bug Bounty program.

But this time, it's not one of the common web vulnerabilities! I'm talking about a quite hazardous E-Mail Injection vulnerability paired with a less interesting Fullpath-Disclosure vulnerability, which - by the way - was really shocking...guess why ;-)

The E-Mail injection vulnerability allowed an attacker to send emails using the PayPal - servers with some restrictions via crafted GET requests. The parameters of the GET request were directly and without any filters used in a PHP [mail](http://php.net/manual/de/function.mail.php) call:

[![paypaltech-email-injection]({{ site.baseurl }}/assets/paypaltech-email-injection.png)]({{ site.baseurl }}/assets/paypaltech-email-injection.png)

Unfortunately PayPal did not implement any kind of technical restriction (like a captcha for example) regarding the number of GET requests that could be sent to the script. This type of vulnerability could become very annoying, because an attacker only has to call the script using some kind of loop while parsing his own email-database - sounds quite interesting for a spammer or phisher.

Good to see that this issue has been fixed now - although the fix took several months :-(. Anyways, I'd like to thank PayPal for the nice bounty payment and the "[Wall of Fame](https://www.paypal.com/webapps/mpp/security-tools/wall-of-fame-honorable-mention)" listing.

[![paypal-hof]({{ site.baseurl }}/assets/paypal-hof.png)]({{ site.baseurl }}/assets/paypal-hof.png)
