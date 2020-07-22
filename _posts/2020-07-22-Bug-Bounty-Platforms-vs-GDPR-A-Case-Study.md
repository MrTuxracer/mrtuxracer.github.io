---
title: 'Bug Bounty Platforms vs. GDPR: A Case Study'
categories:
- GDPR
- BugBounty
---
## What Do Bug Bounty Platforms Store About Their Hackers?
I do care a lot about data protection and privacy things. I've also been in the situation, where a bug bounty platform was able to track me down due to an incident, which was the initial trigger to ask myself: 

How did they do it? And do I know what these platforms store about me and how they protect this (my) data? 
Not really. So why not create a little case study to find out what data they process? 

One utility that comes in quite handy when trying to get this kind of information (at least for Europeans) is the General
Data Protection Regulation (GDPR). The law's main intention is to give people an extensive right to access and restrict their personal data. Although GDPR is a 
law of the European Union, it is extra-territorial in scope. So as soon as a company collects data about a European citizen/resident, 
the company is automatically required to comply with GDPR. This is the case for all bug bounty platforms that I am 
currently registered on. They probably cover 98% of the world-wide market: [HackerOne](https://www.hackerone.com), 
[Bugcrowd](https://www.bugcrowd.com), [Synack](https://www.synack.com), [Intigriti](https://www.intigriti.com), and 
[Zerocopter](https://www.zerocopter.com). 

Spoiler: All of them have to be GDPR-compliant, but not all seem to have proper processes in place to address GDPR 
requests.

## Creating an Even Playing Field
To create an even playing field, I've sent out the same GDPR request to all bug bounty platforms. Since the scenario should be as realistic and real-world as possible, no platform was explicitly informed beforehand that the request, respectively, their answer, is part of a study.

- All platforms were given the same questions, which should cover most of their GDPR response processes (see [Art. 15 GDPR](https://gdpr-info.eu/art-15-gdpr/)):<br>
  ![]({{ site.baseurl }}/assets/gdpr-1.png)
- All platforms were given the same email aliases to include in their responses.
- All platforms were asked to hand over a full copy of my personal data.
- All platforms were given a deadline of one month to respond to the request. Given the increasing COVID situation back in April, all platforms were offered an extension of the deadline (as per [Art. 12 par. 3 GDPR](https://gdpr-info.eu/art-12-gdpr/)).

## Analyzing the Results
First of all, to compare the responses that are quite different in style, completeness, accuracy, and thoroughness, I've decided to only count answers that are a part of the official answer. 
After the official response, discussions are not considered here, because accepting those might create advantages across competitors. This should give a clear understanding of how thoroughly each platform reads and answers the GDPR request.

Instead of going with a kudos (points) system, I've decided to use a "traffic light" rating:

| Indicator | Expectation | 
| --- | --- | 
|<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg>| All good, everything provided, expectations met.|
|<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="orange"/></svg> | Improvable, at least one (obvious) piece of information is missing, only implicitly answered. |
|<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg>| Left out, missing a substantial amount of data or a significant data point and/or unmet expectations.|

This light system is then applied to the different GDPR questions and derived points either from the questions themselves or from the data provided. 

## Results Overview
To give you a quick overview of how the different platforms performed, here's a summary showing the lights indicators. For a detailed explanation of the indicators, have a look at the detailed response evaluations below. 

| Question | HackerOne | Bugcrowd | Synack | Intigriti | Zerocopter |
|-----|--------|--------|--------|--------|--------|
| Did the platform meet the deadline?<br>*([Art. 12 par. 3 GDPR](https://gdpr-info.eu/art-12-gdpr/))* |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Did the platform explicitly validate my identity for all provided email addresses?<br>*([Art. 12 par. 6 GDPR](https://gdpr-info.eu/art-12-gdpr/))* | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Did the platform hand over the results for free? <br>*([Art. 12 par. 5 GDPR](https://gdpr-info.eu/art-12-gdpr/))*| <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Did the platform provide a <u>full</u> copy of my data?<br>*([Art. 15 par. 3 GDPR](https://gdpr-info.eu/art-15-gdpr/))*|<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="orange"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Is the <u>provided</u> data accurate?<br>*([Art. 5 par. 1 (d) GDPR](https://gdpr-info.eu/art-5-gdpr/))*|<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Specific question: Which personal data about me is stored and/or processed by you?<br>*([Art. 15 par. 1 (b) GDPR](https://gdpr-info.eu/art-15-gdpr/))* | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Specific question: What is the purpose of processing this data?<br>*([Art. 15 par. 1 (a) GDPR](https://gdpr-info.eu/art-15-gdpr/))* | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Specific question: Who has received or will receive my personal data (including recipients in third countries and international organizations)?<br>*([Art. 15 par. 1 (c) GDPR](https://gdpr-info.eu/art-15-gdpr/))* |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="orange"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> | <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |
| Specific question: If the personal data wasn't supplied directly by me, where does it originate from?<br>*([Art. 15 par. 1 (g) GDPR](https://gdpr-info.eu/art-15-gdpr/))*| <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg></center> |
| Specific question: If my personal data has been transferred to a third country or organization, which guarantees are given based on article 46 GDPR?<br>*([Art. 15 par. 2 GDPR](https://gdpr-info.eu/art-15-gdpr/) and [Art. 46 GDPR](https://gdpr-info.eu/art-46-gdpr/))*| <center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |<center><svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg></center> |


## Detailed Answers
### HackerOne
Request sent out: 01st April 2020<br>
Response received: 30th April 2020<br>
Response style: Email with attachment<br>
Sample of their response:<br>
![]({{ site.baseurl }}/assets/hackerone-gdpr.png)

| Question | Official Answer | Comments | Indicator |
|-----|--------|--------|--------|
| Did the platform meet the deadline? |  Yes, without an extension. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Did the platform explicitly validate my identity for all provided email addresses? | Via email. | I had to send a random, unique code from each of the mentioned email addresses. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Did the platform hand over the results for free? | No fee was charged. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Did the platform provide a <u>full</u> copy of my data? | No. | A copy of the VPN access logs/packet dumps was/were not provided.<br><br>However, since this is not a general feature, I do not consider this to be a significant data point, but still a missing one. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="orange"/></svg> |
| Is the <u>provided</u> data accurate? | Yes. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Which personal data about me is stored and/or processed by you? | First- & last name, email address, IP addresses, phone number, social identities (Twitter, Facebook, LinkedIn), address, shirt size, bio, website, payment information, VPN access & packet log | HackerOne provided a quite extensive list of IP addresses (both IPv4 and IPv6) that I have used, but based on the provided dataset it is not possible to say when they started recording/how long those are retained.<br><br>HackerOne explicitly mentioned that they are actively logging VPN packets for specific programs. However, they currently do not have any ability to search in it for personal data (it's also not used for anything according to HackerOne)  | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| What is the purpose of processing this data? | Operate our Services, fulfill our contractual obligations in our service contracts with customers, to review and enforce compliance with our terms, guidelines, and policies, To analyze the use of the Services in order to understand how we can improve our content and service offerings and products,  For administrative and other business purposes, Matching finders to customer programs | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Who has received or will receive my personal data (including recipients in third countries and international organizations)? | Zendesk, PayPal, Slack, Intercom, Coinbase, CurrencyCloud, Sterling | While analyzing the provided dataset, I noticed that the list was missing a specific third-party called "TripActions", which is used to book everything around live hacking events. This is a missing data point, but it's also only a non-general one, so the indicator is only orange.<br><br>HackerOne added the data point as a result of this study. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="orange"/></svg> |
| If the personal data wasn't supplied directly by me, where does it originate from? |  HackerOne does not enrich data. |  - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| If my personal data has been transferred to a third country or organization, which guarantees are given based on article 46 GDPR?  | This question wasn't answered as part of the official response. | I've notified HackerOne about the missing information afterwards, and they've provided the following:<br><br>*Vendors must undergo due diligence as required by GDPR, and where applicable, model clauses are in place.* | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |

#### Remarks
HackerOne provided an automated and tool-friendly report. While the primary information was summarized in an email, I've
received quite a huge JSON file, which was quite easily parsable using your preferred scripting language. However, if a 
non-technical person would receive the data this way, they'd probably have issues getting useful information out of it.

### Bugcrowd
Request sent out: 1st April 2020<br>
Response received: 17th April 2020<br>
Response style: Email with a screenshot of an Excel table<br>
Sample of their response:<br>
![]({{ site.baseurl }}/assets/bugcrowd-gdpr.png)

| Question | Official Answer | Comments | Indicator |
|-----|--------|--------|--------|
| Did the platform meet the deadline? | Yes, without an extension. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> | 
| Did the platform explicitly validate my identity for all provided email addresses? |  No identity validation was performed. | I've sent the request to their official support channel, but there was no explicit validation to verify it's really me, for neither of my provided email addresses. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| Did the platform hand over the results for free? |  No fee was charged. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Did the platform provide a <u>full</u> copy of my data? | No. | Bugcrowd provided a screenshot of what looks like an Excel file with a couple of information on it. In fact, the screenshot you can see above is not even a sample but their complete response. <br>However, the provided data is not complete since it misses a lot of data points that can be found on the researcher portal, such as a history of authenticated devices (IP addresses see [your sessions on your Bugcrowd profile](https://bugcrowd.com/settings/active_sessions)), my ISC2 membership number, everything around the identity verification. <br><br>There might be more data points such as logs collected through their (for some programs) required proxies or VPN endpoints, which is required by some programs, but no information was provided about that.<br><br>Bugcrowd did neither provide anything about all other given email addresses, nor did they deny to have anything related to them.| <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| Is the <u>provided</u> data accurate? | No. | The provided data isn't accurate. Address information, as well as email addresses and payment information are super old (it does not reflect my current Bugcrowd settings), which indicates that Bugcrowd stores more than they've provided. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| Which personal data about me is stored and/or processed by you? | First & last name, address, shirt size, country code, LinkedIn profile, GooglePlus address, previous email address, PayPal email address, website, current IP sign-in, bank information, and the Payoneer ID | This was only implicitly answered through the provided copy of my data.<br><br> As mentioned before, it seems like there is a significant amount of information missing. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| What is the purpose of processing this data? |  - | This question wasn't answered.  |  <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| Who has received or will receive my personal data (including recipients in third countries and international organizations)? |  - | This question wasn't answered.  | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| If the personal data wasn't supplied directly by me, where does it originate from?  |  - |  This question wasn't answered. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| If my personal data has been transferred to a third country or organization, which guarantees are given based on article 46 GDPR?  |  - |  This question wasn't answered. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |

#### Remarks
The "copy of my data" was essentially the screenshot of the Excel file, as shown above. I was astonished about the compactness of the answer and asked again to answer all the provided questions as per GDPR. What followed was quite a long discussion with the responsible personnel at Bugcrowd. I've mentioned more than once that the provided data is inaccurate and incomplete and that they've left out most of the questions, which I have a right by law to get an answer to. Still, they insisted that all answers were GDPR-compliant and complete.

I've also offered them an extension of the deadline in case they needed more time to evaluate all questions. However, Bugcrowd did not want to take the extension. The discussion ended with the following answer on 17th April:
> We've done more to respond to you that any other single GDPR request we've ever received since the law was passed. We've done so during a global pandemic when I think everyone would agree that the world has far more important issues that it is facing. I need to now turn back to those things. 

I've given up at that point.

### Synack
Request sent out: 25th March 2020<br>
Response received: 03th July 2020<br>
Response style: Email with a collection of PDFs, DOCXs, XLSXs <br>
Sample of their response:<br>
![]({{ site.baseurl }}/assets/synack-gdpr.png)

| Question | Answer | Comments | Indicator |
|-----|--------|--------|--------|
| Did the platform meet the deadline? | Yes, with an extension of 2 months. | Synack explicitly requested the extension. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Did the platform explicitly validate my identity for all provided email addresses? | No. | I've sent the initial request via their official support channel, but no further identity verification was done. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> | 
| Did the platform hand over the results for free? |  No fee was charged. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Did the platform provide a <u>full</u> copy of my data? | Very likely not. | So essentially, Synack uses a VPN solution called LaunchPoint, which requires every participant to go through when testing targets. What they do know - at least - is when I am connected to the VPN, which target I am connected to, and how long I am connected to it. However, neither a connection log nor a full dump was provided as part of the data copy. Since I do consider this to be a significant piece of information in the context of Synack that wasn't provided, the indicator is red.<br><br>Synack did neither provide anything about all other given email addresses nor did they deny to have anything related to them. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| Is the <u>provided</u> data accurate? | Yes. | The data that was provided is accurate, though. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Which personal data about me is stored and/or processed by you? |  First- &last name, address, email address, social handles (Twitter, Facebook, LinkedIn, GitHub), phone number, VPN username, date of birth, answers to surveys, helpdesk tickets, passport details, payment data and history, copy of W-8BEN, last visited date, last clicks on link tracking in emails, browser type and version, operating system, job title, professional certifications, gender. | This was only implicitly answered through the provided data copy.<br><br>Except for the VPN username, nothing else was mentioned in regard to their VPN. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> | 
| What is the purpose of processing this data? | - |   This question wasn't answered.  |  <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| Who has received or will receive my personal data (including recipients in third countries and international organizations)? | - | This was only implicitly answered through the provided copy of my data. <br><br> The dataset contains a number of documents from different vendors/services, which implicitly answers the question, but also only partially since services like Zendesk are missing entirely. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |  
| If the personal data wasn't supplied directly by me, where does it originate from? | - |   This question wasn't answered. |   <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> |
| If my personal data has been transferred to a third country or organization, which guarantees are given based on article 46 GDPR? | - |  This question wasn't answered. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/></svg> | 

#### Remarks
The communication process with Synack was rather slow because it seems like it takes them some time to get information from different vendors.

### Intigriti
Request sent out: 07th April 2020<br>
Response received: 04th May 2020<br>
Response style: Email with PDF and JSON attachments.<br>
Sample of their response:<br>
![]({{ site.baseurl }}/assets/intigriti-gdpr.png)

| Question | Answer | Comments | Indicator | 
|-----|--------|--------|--------|
| Did the platform meet the deadline? | Yes, without an extension. | - |  <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |
| Did the platform explicitly validate my identity for all provided email addresses? | Yes. | Via email. I had to send a random, unique code from each of the mentioned email addresses. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |
| Did the platform hand over the results for free? |  No fee was charged. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |
| Did the platform provide a <u>full</u> copy of my data? | Yes. | I couldn't find any missing data points. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Is the <u>provided</u> data accurate? | Yes. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Which personal data about me is stored and/or processed by you? |  First- & lastname, address, phone number, email address, website address, Twitter handle, LinkedIn page, shirt size, passport data, email conversation history, accepted program invites, payment information (banking and PayPal), payout history, IP address history of successful logins, history of accepted program terms and conditions, followed programs, reputation tracking, the time when a submission has been viewed.<br><br>Data categories processed: User profile information, Identification history information, Personal preference information, Communication preference information, Public preference information, Payment methods, Payout information, Platform reputation information, Program application information, Program credential information, Program invite information, Program reputation information, Program TAC acceptance information, Submission information, Support requests, Commercial requests, Program preference information, Mail group subscription information, CVR Download information, Demo request information, Testimonial information, Contact request information. | I couldn’t find any missing data points. <br><br>A long, long time ago, Intigriti had a VPN solution enabled for some of their customers, but I haven't seen it active anymore since then, so I do not consider this data point anymore. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |  
| What is the purpose of processing this data? |  Purpose: Public profile display, Customer relationship management, Identification & authorization, Payout transaction processing, Bookkeeping, Identity checking, Preference management, Researcher support & community management, Submission creation & management, Submission triaging, Submission handling by company, Program credential handling, Program inviting, Program application handling, Status reporting, Reactive notification mail sending, Pro-active notification mail sending, Platform logging & performance analysis. | - |<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> | 
| Who has received or will receive my personal data (including recipients in third countries and international organizations)? | Intercom, Mailerlite, Google Cloud Services, Amazon Web Services, Atlas, Onfido, Several payment providers (TransferWise, PayPal, Pioneer), business accounting software (Yuki), Intigriti staff, Intigriti customers, encrypted backup storage (unnamed), Amazon SES. | I've noticed a little contradiction in their report: while saying data is transferred to these parties (which includes third-country companies such as Google and Amazon), they also included a "Data Transfer" section saying "We do not transfer any personal information to a third country." <br><br>After gathering for clarification, Intigriti told me that they're only hosting in the Europe region in regard to AWS and Google. |<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |  
| If the personal data wasn't supplied directly by me, where does it originate from? | Intigriti does not enrich data.|  - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> | 
| If my personal data has been transferred to a third country or organization, which guarantees are given based on article 46 GDPR?  | - |  This information wasn’t explicitly provided, but can be found in their privacy policy: “We will ensure that any transfer of personal data to countries outside of the European Economic Area will take place pursuant to the appropriate safeguards.”<br><br>However, "appropriate safeguards" are not defined. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/> | 

#### Remarks
Intigriti provided the most well-written and structured report of all queried platforms, allowing a non-technical reader to get all the necessary information quickly. In addition to that, a bundle of JSON files were provided to read in all data programmatically.

### Zerocopter
Request sent out: 14th April 2020<br>
Response received: 12th May 2020<br>
Response style: Email with PDF<br>
Sample of their response:<br>
![]({{ site.baseurl }}/assets/zerocopter-gdpr.png)

| Question | Answer | Comments | Indicator |
|-----|--------|--------|--------|
| Did the platform meet the deadline? | Yes, without an extension. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> | 
| Did the platform explicitly validate my identity for all provided email addresses? | Yes | Zerocopter validated all email addresses that I've mentioned in my request by asking personal questions about the account in question and by letting me send emails with randomly generated strings from each address. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> | 
| Did the platform hand over the results for free? |  No fee was charged. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |
| Did the platform provide a <u>full</u> copy of my data? | Yes. | I couldn't find any missing data points. | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Is the <u>provided</u> data accurate? | Yes. | - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/></svg> |
| Which personal data about me is stored and/or processed by you? |  First-, last name, country of residence, bio, email address, passport details, company address, payment details, email conversations, VPN log data (retained for one month), metadata about website visits (such as IP addresses, browser type, date and time), personal information as part of security reports, time spent on pages, contact information with Zerocopter themselves such as provided through email, marketing information (through newsletters). | I couldn’t find any missing data points. |<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> | 
| What is the purpose of processing this data? |  Optimisation Website, Application, Services, and provision of information, Implementation of the agreement between you and Zerocopter (maintaining contact) | - |<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> | 
| Who has received or will receive my personal data (including recipients in third countries and international organizations)? | Some data might be transferred outside the European Economic Area, but only with my consent, unless it is required for agreement implementation between Zerocopter and me, if there is an obligation to transmit it to government agencies, a training event is held, or the business gets reorganized. | Zerocopter did not explicitly name any of these third-parties, except for "HubSpot". |<svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/> | 
| If the personal data wasn't supplied directly by me, where does it originate from?  |  Zerocopter does not enrich data. |  - | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="green"/> |
| If my personal data has been transferred to a third country or organization, which guarantees are given based on article 46 GDPR?  | - | This information wasn't explicitly provided, but can be found in their privacy policy: " These third parties (processors) process your personal data exclusively within our assignment and we conclude processor agreements with these third parties which are compliant with the requirements of GDPR (or its Netherlands ratification AVG)". | <svg height="30" width="30"><circle cx="10" cy="10" r="10" fill="red"/> |

#### Remarks
For the largest part, Zerocopter did only cite their privacy policy, which is a bit hard to read for non-legal people.

## Conclusion
For me, this small study holds a couple of interesting findings that might or might not surprise you:
- In general, it seems that European bug bounty platforms like Intigriti and Zerocopter generally do better or rather seem to be better prepared for incoming GDPR requests than their US competitors.
- Bugcrowd and Synack seem to lack a couple of processes to adequately address GDPR requests, which unfortunately also includes proper identity verification.
- Compared to Bugcrowd and Synack, HackerOne did quite well, considering they are also US-based. So being a US platform is no excuse for not providing a proper GDPR response.
- None of the platforms has explicitly and adequately described the safeguards required from their partners to protect personal data. HackerOne has handed over this data after their official response, Intigriti and Zerocopter have not explicitly answered that question. However, both have (vague) statements about it in their corresponding privacy policies. This point does not seem to be a priority for the platforms, or it’s probably a rather rarely asked question.
   
See you next year ;-)

