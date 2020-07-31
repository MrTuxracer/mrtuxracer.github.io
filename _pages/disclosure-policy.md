---
title: Security Disclosure Policy
permalink: /disclosure-policy/
sitemap: false
---
### Introduction

Releasing vulnerability information is an all-time discussed topic with many different attitudes. Every policy has its own advantages and disadvantages, but I really like [CERT's 45-day policy](http://www.cert.org/vulnerability-analysis/vul-disclosure.cfm). So I have adopted their policy with some small changes and use it for my own disclosure process.

### Assignment Process

Every new disclosure will be assigned with an official CVE identifier (as per MITRE).

### Notification Process

In this phase, the vendor will be initially notified about the vulnerability. The notification might already include the 
vulnerability details. However, this ultimately depends on the contact possibilities offered by the vendor. It does 
usually also include a **preset disclosure date**, which is set **14 days** after the first notification. 

### Disclosure Process

If the vendor does not respond to the notification(s) or does not acknowledge the vulnerability until the initial deadline ends, the vulnerability will be disclosed immediately. I decided to take this strict first deadline for a straightforward reason: In 95% of my previous coordination attempts, a vendor who does not respond within the first 14 days, likely won't respond at all. 

If the vendor responds, the initial deadline will be extended to the **45 days final** **deadline**. While working on the issue until the final deadline, I expect regular status updates on the report. If there is none, the vulnerability will be made public after 45 days regardless of its state.

However, if the vendor has a good explanation or the vulnerability affects a wide range of users or systems, the deadline might be extended.

As soon as either the initial or final deadline ends the details of the vulnerability will be made public depending on the relationship between RCE Security and the vendor (e.g. previously negotiated NDA due to active contracts). The disclosure happens across various places, amongst them are: [Full-Disclosure](http://seclists.org/fulldisclosure/)<span style="font-size: 16px; line-height: 1.428571429;"> and </span>[Bugtraq.](http://www.securityfocus.com/archive/1)

Since I do believe in maximum transparency and effective ways for administrators and penetration testers to test vulnerable systems, I will also publish either a Proof-of-Concept or an exploit alongside a blog article.

### Summary

The advisory will be immediately published using the </span>[Full-Disclosure](http://seclists.org/fulldisclosure/)<span style="font-size: 16px; line-height: 1.428571429;"> and </span>[Bugtraq ](http://www.securityfocus.com/archive/1)mailing - lists, either when:

*   **14-Days initial deadline ends:** The vendor does not respond to any of the initial notifications.
*   **45-Days final deadline ends: **The vendor does not meet the final disclosure date without an extension.
*   An official update is released by the vendor.
*   A third party publishes an advisory on the same issue.
*   The vendor hasn't responded to multiple, previous coordination attempts.
