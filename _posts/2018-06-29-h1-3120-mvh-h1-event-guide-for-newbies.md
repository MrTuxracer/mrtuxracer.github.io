---
title: 'H1-3120: MVH! (H1 Event Guide for Newbies)'
categories:
- BugBounty
---
Here's another late post about my coolest bug bounty achievement so far! In May I've participated in HackerOne's H1-3120 in the beautiful city of Amsterdam with the goal to break some [Dropbox](https://www.dropbox.com) stuff. It was a really tough target, but I still managed to find some juicy bugs! According to [d0nutptr](https://twitter.com/d0nutptr) of the Dropbox team, these caused some internal troubles leading to some serious phone calls ;-). In the end I was awarded three out of the four titles of the evening: "The Exalted", "The Assassin" and finally also the "Most Valuable Hacker" (MVH)!

[![]({{ site.baseurl }}/assets/H1-3120-1.jpg)]({{ site.baseurl }}/assets/H1-3120-1.jpg)[![]({{ site.baseurl }}/assets/H1-3120-2.jpg)]({{ site.baseurl }}/assets/H1-3120-2.jpg)

However, I do not only want to talk about my achievements (there are a lot of pictures available on the [here](https://www.facebook.com/pg/Hacker0x01/photos/?tab=album&album_id=1852603941438068)), but rather give some tips for all those upcoming event first timers ;-).

### You (usually) get to know the targets right before the event!  
Typically, HackerOne provides you with a near-final scope a few days before the event. This means you will have some time to explore the target and already collect bugs before the actual event kicks off. My advice here is: hack as much as you can because of one awesome thing: **Bounties will be split during the first 30 minutes! **This means if 5 people submit the same vulnerability during the first 30 minutes, the bounty for the vulnerability will be equally split amongst everyone, i.e.: 1000 USD/5 people = 200 USD for everyone.

### Do your recon prior to the event!  
I usually do a lot of recon before I actually start hacking. While this already leads to bugs for myself, it creates another big advantage: I always find low-hanging or seemingly unexploitable things and while working with other hackers during the event, somebody might ask you if there is any endpoint vulnerable to an open redirect or whatever, because he/she actually needs it to complete an exploit chain. Why not collect some additional love here and provide them with your seemingly unexploitable stuff? They might help you out afterwards too!

### Late-double-check your findings the night before the actual event!  
Bad luck might hit you! It is possible that you find stuff, which the program fixes right before the event kicks off. Trust me, while working on bugs for H1-415, I found some really awesome bugs during the preparation phase for the event. However, while I was (more or less accidentally) checking one of my bugs the night before the event, I noticed that it has been fixed. From this point, I told myself to always late-double-check my bugs to avoid getting N/As.

### Try to prevent Frans Rosen from submitting / submit your bugs using bountyplz!  
While it is always a good idea to prevent [Frans](https://twitter.com/fransrosen) from using his machine to submit his findings, you should also use his tool "[bountyplz](https://github.com/fransr/bountyplz)" to mass-submit findings especially if you have so many vulnerabilities on your waiting list, that it could be difficult to submit them all during the 30 minutes bounty split time. Since the H1 reporting form isn't really made for quick submissions, it will help you getting your reports in before the deadline ends.

### Collaborate during the event!**  
Yes, it is still a competition, but don't underestimate it! You will learn that everybody has his/her own specialties that can help exploiting bugs! On this way I have learned about [Corb3nik's](https://twitter.com/Corb3nik) really awesome JavaScript skills!

Thank you very much HackerOne and Dropbox for organizing such an awesome event! I'm already looking forward to all the future events :-)
