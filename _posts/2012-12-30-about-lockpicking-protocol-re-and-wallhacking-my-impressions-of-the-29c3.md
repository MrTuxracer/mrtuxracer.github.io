---
title: 'About Lockpicking, Protocol RE and Wallhacking: My Impressions Of The #29C3'
categories: 
- Conferences
---
The 29th annual [Chaos Communication Congress](http://www.ccc.de/) under the slogan "[Not my Department](https://events.ccc.de/congress/2012/wiki/Main_Page)" arrived again in the most beautiful city in the world: Hamburg! The Congress moved from the Congress Center in Berlin, where people had to sit stacked (according to some fables :-D), to the [CCH](http://www.cch.de) in Hamburg which offers a lot more space for the heavily growing number of CCC members. It was my first time visiting the Chaos Congress and I have to say that **IT IS** a very impressive event with many interesting talks, workshops, (political) discussions and a lot of [Club Mate](http://www.club-mate.de). I need to dismiss this xmas - thing completely next year ;-) !

Right after leaving the metro, I had to lol the first time - the Congress Center itself has already been defaced :-D:

![29c3-1]({{ site.baseurl }}/assets/29c3-1.jpg)

And the second time about the very professional "attached" access points :-D :-D:

![29c3-2]({{ site.baseurl }}/assets/29c3-2.jpg)

One big and interesting facet of the event are the talks on many different topics! The first talk I attended was "[Analytical Summary of the BlackHole Exploit Kit - Almost Everything You Ever Wanted To Know About The BlackHole Exploit Kit](http://www.youtube.com/watch?v=1RvhYSWasb4)":

![29c3-3]({{ site.baseurl }}/assets/29c3-3.jpg)

The topic really sounds interesting, but the talk itself was just miserably presented! The talk started with a statement like "Oh that's quite a new slideset, I haven't seen yet". Well...nothing more to say. If I would speak in front of thousands of visitors, I would prepare my self - seriously! The content itself was OK for an analytical summary of the probably best-known underground exploit kit currently available for purchase, because you need a lot more time to present a deep-going analysis of how things are working.

The talk about lockpicking: "[Open Source Schlüssel und Schlösser - Offene Quellen zum Bösen und Guten: von downloadbaren Handschellenschlüsseln zu sicheren elektronischen Schlössern](http://www.youtube.com/watch?v=3JK3TO_crc8)" was - in my opinion - the best presented one.  
![29c3-4]({{ site.baseurl }}/assets/29c3-4.jpg)  
I've never had a look at the obviously heavily discussed topic "lockpicking" but this talk inspired me a lot. It looks like it's quite easy to hack the actually most common keys (and handcuffs - Question: Why does the police still use them ?) without a lot of effort - even the electronic ones, which might have implemented a great electronical protection, but obiously a weak mechanical one :-D. So where to buy a small lockpicking set ?

"[WRITING A THUMBDRIVE FROM SCRATCH - Prototyping Active Disk Antiforensics](http://www.youtube.com/watch?v=D8Im0_KUEf8)"

![29c3-5]({{ site.baseurl }}/assets/29c3-5.jpg)]({{ site.baseurl }}/assets/29c3-5.jpg)

Okay, that one deeply impressed me! Travis Goodspeed (I tend to call him "**yoda**" :-D ) presented his own facedancer board, a USB-Emulation/Connection/Simulation/Fuzzing - Tool with a user-land Python interface :-) This board could e.g. be used to low-level fuzz the USB - Protocol and therefor attached devices and drivers. Damn, I have to get such a board! Really! Yes, yes, yes - a lot of funny ideas arise at my horizon :-)

"[Russia’s Surveillance State](http://www.youtube.com/watch?v=ArXZiMviAEQ)"  
![29c3-6]({{ site.baseurl }}/assets/29c3-6.jpg)  
An interesting political talk about the surveillance situation in Russia and how the government forces **ALL** local ISPs to buy and implement nation-wide surveillance mechanisms to - obiously - observe mainly political opposition members. It's hard to believe that there are states which implement those techs in a much more aggressive way than Germany. This strengthens my resolve in never visiting this country if I risk my personal freedom when I use a smartphone for example! The more interesting question which came up in my mind during the talk is - if they have already implemented such surveillance techs, then **why the hell** do they obviously not care about all those DDoS and fraud kids in their country ?!?!

"[SECURITY EVALUATION OF RUSSIAN GOST CIPHER - Survey of All Known Attacks on Russian Government Encryption Standard](http://events.ccc.de/congress/2012/Fahrplan/events/5225.en.html)"

![29c3-7]({{ site.baseurl }}/assets/29c3-7.jpg)

Dr Nicolas T. Courtois the author of several attacks on the Russian Gost cipher and presented his findings with quite heavy details. Beside the fact that you need a diploma in mathematics, you need a lot of more time to present such a heavy topic (obviously that's the reason why he had to skip a lot of ppt pages :-D ). He tried to illustrate the mathmatical stuff using some funny rabbit analogies, but although it was hard to understand the different attack techs, which - academically - broke the cipher, if you don't have the mathematical background. Anyways another great talk!

"[FURTHER HACKS ON THE CALYPSO PLATFORM - or how to turn a phone into a BTS](http://www.youtube.com/watch?v=B1od4x9L3t4)"

![29c3-8]({{ site.baseurl }}/assets/29c3-8.jpg)

A technically interesting talk for those who are interested in mobile techs, but that's not my favourite hacking - topic.

"[The future of protocol reversing and simulation applied on ZeroAccess botnet - Mapping your enemy Botnet with Netzob](http://www.youtube.com/watch?v=snT-6qIewIQ)"

![29c3-9]({{ site.baseurl }}/assets/29c3-9.jpg)

This was the most interesting talk of my day! The guys from [Amossys](http://www.amossys.fr/) presented their protocol reverse engineering tool called "[Netzob](http://www.netzob.org)" and demo'ed their tool on the currently active P2P - botnet ZeroAccess, where they managed to receive e.g. the list of nodes using the reverse engineered protocol. Therefor it's able to handle unknown protocols and build up interesting things like simulated communication chains. Additionally you've got the possibility to export your RE work to the peach fuzzer which makes the tool very interesting for me. I definitively need to have a closer look at this great-looking tool!

The day ended with a look at the nice traffic consumption:

![29c3-10]({{ site.baseurl }}/assets/29c3-10.jpg)

And while leaving the building...the Radisson Blu hotel near the Congress Center has been hacked using some interesting kind of "wallhack" :-D:

![29c3-11]({{ site.baseurl }}/assets/29c3-11.jpg)

All-in-All it was a great experience - I'll be back next year ! :-)

All talks are available [here](http://www.youtube.com/user/28c3).
