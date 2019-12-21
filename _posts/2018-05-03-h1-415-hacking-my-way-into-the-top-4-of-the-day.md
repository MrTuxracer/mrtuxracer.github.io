---
title: 'H1-415: Hacking My Way Into the Top 4 of the Day'
categories:
- BugBounty
excerpt: "I've always wanted to visit San Francisco! So I was really happy about an email from [HackerOne](https://www.hackerone.com) inviting me to this beautiful city in April. But they did not cover all the costs for my international flights and the hotel room just for my personal city trip - they had something really nasty in mind: hacking Oath!"
---
I've always wanted to visit San Francisco! So I was really happy about an email from [HackerOne](https://www.hackerone.com) inviting me to this beautiful city in April. But they did not cover all the costs for my international flights and the hotel room just for my personal city trip - they had something really nasty in mind: hacking Oath! If you don't know [Oath](https://www.oath.com) - they own brands like Yahoo, AOL, Tumblr, Techcrunch amongst others.

So while a free trip to San Francisco by itself is already an awesome thing, HackerOne did a great job in organizing a live hacking event which currently has no equal...and this does not only apply to the logo ;-)

[![]({{ site.baseurl }}/assets/h1-415-logo.png)]({{ site.baseurl }}/assets/h1-415-logo.png)

The event itself took place in a beautiful coworking space in downtown San Francisco on the 14th floor with a nice view over San Francisco, including a tasty breakfast. This breakfast was indeed needed for the upcoming 9 hours of hacking kung-fu! The hacking was finally kicked off at 10:00 with a pretty nice scope to hack on. However, the scope itself has already been announced a couple of days prior to the event itself, so that everyone had the chance to prepare some nice vulnerabilities and bring them to the event. The only tricky thing was to verify the vulnerabilities again before submitting them during the event to make sure they haven't been fixed by a last-minute patch ;-)

As part of this preparation I found almost 20 vulnerabilities ranging from Cross-Site Scripting up to some nice SQL Injection chains. The first 60 minutes of the event were covered by a blackout period where everybody had the chance to submit their findings without having to fear duplicates! The good thing about this approach was that duplicates have been paid out by splitting the bounty amount amongst all hackers that reported the same vulnerability. Luckily my personal dupe count was just at 3 resulting in my smallest bounty of USD 50\. After this blackout period all duplicates were handled as usual - first come, first serve.

After 9 hours of continuous hacking my personal day ended with 25 vulnerability submissions, a maximum single payout of 5.000 USD and an overall rank of 4 on the [event leaderboard](https://hackerone.com/hackathons/h1415_2018/live):

[![]({{ site.baseurl }}/assets/h1-415-2.png)]({{ site.baseurl }}/assets/h1-415-2.png)

At the end of the day [Oath paid an overall of 400.000 USD](https://www.oath.com/2018/04/20/we-invited-40-of-the-world-s-best-security-researchers-to-hack-o/) (yes it's 6 digits!) to all participating hackers, which has been the biggest event so far!

However, there was more to this event than just getting bounties. During the event I met so many talented hackers like [@yaworsk](https://twitter.com/yaworsk), [@arneswinnen](https://twitter.com/arneswinnen), [@securinti](https://twitter.com/securinti), [@smiegles](https://twitter.com/smiegles), [@SebMorin1](https://twitter.com/SebMorin1), [@thedawgyg](https://twitter.com/thedawgyg), [@seanmeals](https://twitter.com/seanmeals), [@Corb3nik](https://twitter.com/Corb3nik), [@Rhynorater](https://twitter.com/Rhynorater) , [@prebenve](https://twitter.com/prebenve), [@ngalongc](https://twitter.com/ngalongc), the famous [@filedescriptor](https://twitter.com/filedescriptor) and many, many more which is by far more valuable than any bounty! Thank you so much for being part of this community!

[![]({{ site.baseurl }}/assets/h1-415-1.jpg)]({{ site.baseurl }}/assets/h1-415-1.jpg)

On this way I would like to thank HackerOne and specifically Ted Kramer for organizing a really awesome event and [Ben Sadeghipour](https://twitter.com/NahamSec) for giving me the chance to show my skills! A special thanks is going to the whole HackerOne triaging team for triaging hundreds of vulnerability reports and paying them out right on stage - just another day at work, right ;-) ?

It was a truly amazing experience - see you on the next event!
