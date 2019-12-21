---
title: Modern Lords of War
categories:
- News
---
[The Wassenaar Arrangement](http://www.wassenaar.org/controllists/2014/WA-LIST%20%2814%29%202/WA-LIST%20%2814%29%202.pdf). Maybe you have already heard about that. With the implementation of this multilateral export control regime on conventional arms, dual-use goods and technologies, security researchers like me could be called lords of war and weapons dealers now - sounds cool, but unfortunately it's not.

While Google has officially [commented on the problems](http://googleonlinesecurity.blogspot.fr/2015/07/google-wassenaar-arrangement-and.html), I would like to add an interesting and clarifying comparison of two recent cases, because they perfectly describe the consequences of this arrangement.

I would like to encourage other security researchers to post on this topic to get more public attention and probably to make law makers even more aware of their sloppy definitions, which lead to a really dangerous situation.

### Export Controls for the Bad and Evil

So maybe you should read about [the story](http://tekwizz123.blogspot.de/2015/07/final-year-dissertation-paper-release.html) of my blogging colleague tekwizz123 who isn't able to publish his dissertation in full details because the Wassenaar is not 100% clear in its definitions and this is where it hit Grant. According to Wassenaar all kinds of "intrusion software" and "technology" need to be licensed before they could be privately disclosed (exported) to an outside vendor. Its sloppy definition of "intrusion software" covers:

> "Software" specially designed or modified to avoid detection by 'monitoring tools', or to defeat 'protective countermeasures', of a computer or network-capable device, and performing any of the following:
> 
> a. The extraction of data or information, from a computer or network-capable device, or the modification of system or user data; or  
> b. The modification of the standard execution path of a program or process in order to allow the execution of externally provided instructions.

According to this definition, Grant's dissertation would indeed be an export, because he somehow demonstrated how the standard execution path of a process was modified. But let's have a look at the second case, which has just one thing in common with Grant's case: an export as per Wassenaar definition, but both cases are as different as a unicorn and Batman.

Important to mention is that - in comparison to Grant, who's living in the UK, I am living in Germany and the implementation process of the Wassenaar might be slightly different as it's up to the different participating states to implement the changes into law. While there is a [Request for Comments period](http://www.regulations.gov/#!docketDetail;D=BIS-2015-0011) in the USA, Germany has already added the Wassenaar to its [export control laws](http://www.ausfuhrkontrolle.info/ausfuhrkontrolle/de/gueterlisten/anhaenge_egdualusevo/index.html).

### Unicorns vs. Batman

The second example: Italy-based HackingTeam. Among the recently leaked information are different proofs that they have sold their perfect example of an "intrusion software" called "RCS" to oppressive governments like the [Sudan](https://wikileaks.org/hackingteam/emails?q=sudan&mfrom=&mto=&title=&notitle=&date=&nofrom=&noto=&count=50&sort=0#searchresult). Additionally HackingTeam is [listed by reporters without borders](http://surveillance.rsf.org/en/hacking-team/) as one of the enemies of the internet. RCS had several 0days included to start targeted attacks against anyone and was even able to persist itself in the Bios. Plus it was sold to oppressive regimes to probably do exactly these scary things on government opponents. While this is not only a clear violation of the Wassenaar, but also of international embargoes, it clearly shows that we do need regulations here for companies providing those commercial malware.

Now, have a look at the security research community, which Grant is an important part of. Public security research is performed by smart people in form of papers, proof-of-concept codes and exploits or responsible disclosures like used in Bug Bounties or even [Microsoft's BlueHat](https://technet.microsoft.com/en-us/security/dn425049.aspx) program. This way of vulnerability coordination has helped the industry over years to provide adequate fixes for security vulnerabilities before they became public.

In return, this is not only useful for the vendors, who get (most of the times) free vulnerability reports to increase their overall product security, but also for the end customer who receives security updates before a blackhat abuses the flaws. But what if everybody has to apply for an export license first, while just in mind to help securing systems ? The process is not only taking a lot more time, but it's not guaranteed that you will receive a license without handing over all your vulnerability details to your government. This may lead to the situation where the researcher doesn't even want to start disclosing the vulnerability anymore, which results in an even higher risk to anyone.

...yes, but the heads behind Wassenaar put Grant's research on a level with HackingTeam's RCS.![what_meme]({{ site.baseurl }}/assets/what_meme.jpg)

### Any Solutions Available?

Probably. Let's consider the published [BIS FAQ](http://bis.doc.gov/index.php/policy-guidance/faqs#subcat200), which states that the following things won't be controlled:

> 1.  Information on how to search for, discover or identify a vulnerability in a system, including vulnerability scanning;
> 2.  Information about the vulnerability, including causes of the vulnerability; and
> 3.  Information on testing the vulnerability, including ‘fuzzing’ or otherwise trying different inputs to determine what happens; and
> 4.  Information on analyzing the execution or functionality of programs and processes running on a computer, including decompiling or disassembling code and dumping mem

So this could mean, that the basic security research is indeed not considered a controllable good. But it explicitly mentions that

> Information "required for" developing, testing, refining, and evaluating "intrusion software", in order, for example, technical data to create a controllable exploit that can reliably and predictably defeat protective countermeasures and extract information.

needs to be controlled. This means every work without an exploit is legal, and the moment you include an exploit you're moving into the dark corners of law. Luckily the Wassenaar explicitly exempts one option, which seems to bypass everything:

> Controls do not apply to "technology" "in the public domain", to "basic scientific research" or to the minimum necessary information for patent applications.

Whereas "public domain" is clearly defined:

> "In the public domain" This means "technology" or "software" which has been made available without restrictions upon its further dissemination

This is also supported by the BIS FAQ:

> Third, export controls do not apply to any technology or software that is "published" or otherwise made publicly available.

This means something like Full Disclosure could be a possible way of going "public domain" as defined by Wassenaar and the ultimate solution for every security researcher. But this also means systems are exposed to a higher risk due to the disclosure-delivered-patched time-frames. There are people who are in favor of this, but I'm a friend of responsible disclosure.

Unfortunately there's another problem in regards to bug bounties: You only get paid and accredited if you disclose the vulnerability to the vendor before the public disclosure. This means that it could be the end of (at least) international bug bounty programs, if you have to first apply for an export license to be ready to submit your bug to a bounty program. Bug Bounties are not only based on fast submissions, but they also rely on the power of many different creative heads from all over the world!

So, do you - governments - really want to destroy the security research community?
