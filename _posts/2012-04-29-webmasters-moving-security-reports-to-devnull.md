---
title: Webmasters moving security reports to /dev/null ?
categories:
- Playground
---
![]({{ site.baseurl }}/assets/dumpster.jpg "dumpster")

Hello readers,

There are good and there are bad "webmasters". I suppose that everyone who has ever reported (or better: tried to report) a security issue on a website to the responsible webmaster, faces at least one time in his or her life the problem of not receiving any reaction, not even after several tries.

I'm [regularly reporting](http://security.inshell.net/category/custom) issues on websites to webmasters, as I am on the ethical side of researching. In most of my cases the webmaster (e.g. namely: [T-Systems](http://security.inshell.net/advisory/4) and the [University of Hagen](http://security.inshell.net/advisory/7)) reacts immediately after my notification and fixes the issue. A professional conversation with a "Thank you" at the end, shows that they already understood how to handle such kind of reports. And this is quite motivating at all, if you know that someone is thankful for things you are reporting :-), like T-Systems did:

> <pre>Dear Mr. Ahrens,
>
> Thank you very much for respecting our request!
> The process was very transparent and accommodating.
> Thank you again for your help and understanding.
>
> Sincerely,
> Maria S.</pre>

The World Wide Web grows. Day by day. And every day great new things are developed. And great things where millions of Euros are exchanged every day, also attract the bad boys. Fraud, databreaches and theft on nearly every corner of the www. That's reality. So why are there still webmasters who do not want to accept serious 3rd-party security reports?

Let's discuss some possible ways a webmaster thinks about when receiving a security report.

**_1. "Oh, there is a security report about an issue on my website. Hm. I don't like to see how someone shows me the mistakes I've made. --> /dev/null"_**

Well this is the worst and also most naive option. Even the products of the big players like Google, Facebook and Microsoft have bugs inside - or better called: mistakes made by "professional" developers. By increasing complexity of applications, the number of possible security issues raise too. This is the way of doing "[security through obscurity](http://en.wikipedia.org/wiki/Security_through_obscurity)", but this does not keep the bad boy off from using your website for example for phising purposes using a Cross-Site Scripting flaw or  directly stealing your customer information via SQL Injection. The bad boy is not going to report anything to you! He uses the theft data to make the best profit out of it and he will come back regularly if the issue is unfixed!

**_2. "Oh, there is a security report about an issue on my website. Hm. I do not have time or money to investigate --> /dev/null"._**

Hahaha, well this is funny :-)! Nothing more. If you don't have time to have a look at security issues, you should stay away from the internet and better open up an "offline" store downtown ;-)

**_3\. "Oh, there is a security report about an issue on my website. Hm. I do not understand what he or she wants to tell me. --> /dev/null"_**

Well this is not a big problem! Personally... I'm always offering my help for free. So why don't accepting the help ? And let me say that I've been working with webmasters who have accepted my help. Both can learn a lot out of this process! :-)

Now the interesting question: What to do with webmasters who think in one of the above mentioned ways and refuse to react to a security report ? The Full Disclosure thing is working like a charme regarding applications. If someone discovers a new flaw and the author of the product refuses to release an update / patch / fix, the customer can just remove the vulnerable product and buy a better one.

But this isn't working for websites, as they are something static and the damages araising from publishing vulnerabilities directly to the companies could be greater than you probably expect. What to do in such a case? Good question, without an answer :-(. Keep the vulnerability private and continue to notice the webmaster ? Publish it anonymously ?

**All in all: There is no reason for moving serious security reports to /dev/null! I do not want to blame or extort you, I only want to help making the web a more secure place! **
