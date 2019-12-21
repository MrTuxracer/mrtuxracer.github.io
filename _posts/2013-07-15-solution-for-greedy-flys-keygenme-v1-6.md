---
title: Solution for Greedy Fly's KeyGenMe v1.6
categories:
- Playground
---
I like puzzles, they keep your mind up2date! So I've just registered over at [crackmes.de](http://www.crackmes.de) because it really looks like a lot of fun - and I like fun especially when it comes to reversing things. But isn't it Off-Topic? No, because analyzing applications and breaking protections in AppSec is something that is closely connected to exploiting flaws: You need to find a way through the application flow, probably evading protection mechanisms and understand what the dev did wrong ;-), which leads - in the end - to an application level compromise of the protection and grants access for all those badboy pirates. These information can help the dev to implement a more secure registration/authentication process. Hardening is something that can be fun, even if you keep in mind that nothing is 100% secure!

Let's reverse the keygeneration process of an entry-level KeyGenMe. To be solved: [Greedy Fly's KeyGenMe v1.6](http://crackmes.de/users/greedy_fly/keygenme_v1.6/)

[![greedyfly16-1]({{ site.baseurl }}/assets/greedyfly16-1.png)]({{ site.baseurl }}/assets/greedyfly16-1.png)

### Where does the KeyGen - process start ?

Since there is no badboy - message (if you enter a wrong serial, the KeyGenMe exits silently) and the goodboy - message is unknown yet, you need to look for something else. A good starting point is the "Hi!!!" string: Having a look at the IDA strings window reveals:

{% highlight assembly %}
.rdata:0040227C 0000000B C user32.dll                 
.rdata:004022BA 0000000D C kernel32.dll               
.rdata:004022DE 0000000D C comctl32.dll               
.rdata:00402348 0000000A C ole32.dll                  
.rdata:004023C8 0000000A C gdi32.dll                  
.rdata:004023FA 0000000D C oleaut32.dll               
.data:00403000  0000001C C KeygenMe v1.6 by Greedy Fly
.data:0040301C  00000006 C Hi!!!                      
.data:00403050  00000006 C IMAGE
{% endhighlight %}

The string can be found @ 0x0040301C which is located in the function "DialogFunc":

{% highlight assembly %}
; INT_PTR __stdcall DialogFunc(HWND, UINT, WPARAM, LPARAM)
DialogFunc proc near
{% endhighlight %}

### Find the correct Control handles!

Scrolling down a bit below the function head, you discover the first interesting part:

{% highlight assembly %}
.text:0040124D       push    68h             ; nIDDlgItem
.text:0040124F       push    [ebp+hDlg]      ; hDlg
.text:00401252       call    GetDlgItem
{% endhighlight %}

The [GetDlgItem](http://msdn.microsoft.com/en-us/library/windows/desktop/ms645481(v=vs.85).aspx) function retrieves the handle to an control within the GUI. The famous tool [ResourceHacker](http://www.angusj.com/resourcehacker/) is able to reveal all assigned control handles:

[![greedyfly16-2]({{ site.baseurl }}/assets/greedyfly16-2.png)]({{ site.baseurl }}/assets/greedyfly16-2.png)

The Edit-Controls (called textboxes in the further article) have the handles 104 up to 107 but in a muddled order (from left to right):

1.  <span style="line-height: 14px;">Textbox = 107</span>
2.  Textbox = 106
3.  Textbox = 104
4.  Textbox = 105

This will become important later ;-) ...and btw: the tool also reveals the goodboy message: "Yeaaaaah!!! Congratulations!!! You're the best cracker!"

According to the first code snippet the third textbox is queried first by the GetDlgItem function which returns the handle for the control and stores it into EAX. The handle is copied to another location in memory, because EAX is used in the following [SendMessageA](http://msdn.microsoft.com/en-us/library/windows/desktop/ms644950(v=vs.85).aspx) CALL:

{% highlight assembly %}
.text:00401257       mov     dword_403070, eax
.text:0040125C       lea     eax, lParam
.text:00401262       push    eax             
.text:00401263       push    8               ; wParam
.text:00401265       push    0Dh             ; Msg
.text:00401267       push    dword_403070    ; hWnd
.text:0040126D       call    SendMessageA
.text:00401272       mov     ebx, eax
{% endhighlight %}

Using this SendMessage CALL the application is able to issue commands such as

{% highlight assembly %}
.text:00401265       push    0Dh             ; Msg
{% endhighlight %}

...which is a [WM_GETTEXT](http://msdn.microsoft.com/en-us/library/windows/desktop/ms632627(v=vs.85).aspx) CALL directly to the handle "dword_403070" which contains the handle for the third textbox. After the SendMessage CALL, EAX contains the length of the string loaded from the third textbox which is afterwards MOVed to EBX. The next CALL is much more interesting:

{% highlight assembly %}
.text:00401274       call    sub_4014A9
{% endhighlight %}

### The first challenge!

It's the first "real" part of the Keygeneration process:

{% highlight assembly %}
.text:004014A9 sub_4014A9      proc near               ; CODE XREF: DialogFunc+138
.text:004014A9                 inc     ebx             
.text:004014AA                 shl     ebx, 4          
.text:004014AD                 mov     esi, 64
.text:004014B2                 lea     edi, [ebx+esi]  
.text:004014B5                 imul    ebx, edi, 386   
.text:004014BB                 xor     esi, esi
.text:004014BD                 xchg    esi, ebx        
.text:004014BF                 sub     ebx, ebx
.text:004014C1                 retn
.text:004014C1 sub_4014A9      endp
{% endhighlight %}

Deciphered: EBX holds the strlen of the 3rd part of the serial. At this point it's incremented by 1:

{% highlight assembly %}
.text:004014A9       inc     ebx
{% endhighlight %}

Afterwards shifted left by 4, which is basically the same like [multipliying ](http://dcla.rkhb.de/mul.html)with 16:

{% highlight assembly %}
.text:004014AA       shl     ebx, 4
{% endhighlight %}

64 is added on top:

{% highlight assembly %}
.text:004014AD       mov     esi, 64
.text:004014B2       lea     edi, [ebx+esi]
{% endhighlight %}

multiplied by 386:

{% highlight assembly %}
.text:004014B5       imul    ebx, edi, 386
{% endhighlight %}

and finally XCHGed with /  saved in ESI:

{% highlight assembly %}
.text:004014BB       xor     esi, esi
.text:004014BD       xchg    esi, ebx        
.text:004014BF       sub     ebx, ebx
.text:004014C1       retn
{% endhighlight %}

Back in the DialogFunc application flow, ESI is compared to the value "67936":

{% highlight assembly %}
.text:00401279       cmp     esi, 67936      
.text:0040127F       jz      short loc_401286
{% endhighlight %}

But what's that value ? To find out you have to reverse the last few commands which represent the following calculation:

{% highlight assembly %}
((((strlen+1)*16)+64)*368)
{% endhighlight %}

Mathematically reversed:

{% highlight assembly %}
((((67936/368)-64)/16)-1) = ~6
{% endhighlight %}

This means: if strlen equals 6 the following jump will be taken, which is our goal:

[![greedyfly16-3]({{ site.baseurl }}/assets/greedyfly16-3.png)]({{ site.baseurl }}/assets/greedyfly16-3.png)

The next two parts are very similar. The strlen of the first textbox is compared to 6 (EDX, that is also pushed onto the Stack):

{% highlight assembly %}
.text:00401286       push    6Bh             ; nIDDlgItem ("107"=1st textbox)
.text:00401288       push    [ebp+hDlg]      ; hDlg
.text:0040128B       call    GetDlgItem
.text:00401290       mov     dword_403074, eax
.text:00401295       lea     eax, unk_403090
.text:0040129B       push    eax             ; lParam 
.text:0040129C       push    8               ; wParam
.text:0040129E       push    0Dh             ; Msg (WM_GETTEXT-0x000D)
.text:004012A0       push    dword_403074    ; hWnd
.text:004012A6       call    SendMessageA
.text:004012AB       mov     edx, eax        
.text:004012AD       push    edx             
.text:004012AE       cmp     edx, 6          
.text:004012B1       jnz     loc_40144C
{% endhighlight %}

and strlen of the fourth textbox is compared to EDX, that contains the POPed value 6:

{% highlight assembly %}
.text:004012B7       push    69h             ; nIDDlgItem ("105"=4th textbox)
.text:004012B9       push    [ebp+hDlg]      ; hDlg
.text:004012BC       call    GetDlgItem
.text:004012C1       mov     dword_403078, eax
.text:004012C6       lea     eax, unk_4030A0
.text:004012CC       push    eax             ; lParam (pointer to the buffer that is to receive the text)
.text:004012CD       push    8               ; wParam
.text:004012CF       push    0Dh             ; Msg (WM_GETTEXT-0x000D)
.text:004012D1       push    dword_403078    ; hWnd
.text:004012D7       call    SendMessageA
.text:004012DC       pop     edx             ; get EDX back from stack
.text:004012DD       cmp     eax, edx        
.text:004012DF       jz      short loc_4012E6
.text:004012E1       jmp     loc_40144C
{% endhighlight %}

This means the strings from textboxes 3,1 and 4 (in this order) are compared in its lengths to 6\. If that matches, the application takes "the right" jump. Up next: The string from the 4th textbox is PUSHed to the stack and the function "sub_4015D0" is CALLed. Thanks to IDA's great possibility to rename functions, I'll use the "addup_int_from_char" name for this function, because it's used several times later again:

{% highlight assembly %}
.text:004012E6       push    offset unk_4030A0 ; push string from 4th textbox to stack
.text:004012EB       call    addup_int_from_char ; sub_4015D0
{% endhighlight %}

This functions addups all single chars in a loop and returns the value to EAX. To better visualize what this function does, I've commented my IDA dump:

[![greedyfly16-4]({{ site.baseurl }}/assets/greedyfly16-4.png)]({{ site.baseurl }}/assets/greedyfly16-4.png)

The value from EAX is then compared against decimal 500200, which means that the value from the 4th textbox has to be less or equal to this value:

{% highlight assembly %}
.text:004012F0       cmp     eax, 7A1E8h     ; Sum (ECX) must be less than 500200
.text:004012F5       jbe     short loc_4012FC
.text:004012F7       jmp     loc_40144C
{% endhighlight %}

The following four lines

{% highlight assembly %}
.text:004012FC       cmp     byte_4030A3, 32h ; 4th char of 4th textbox must be 32h (2)
.text:00401303       jz      short loc_40130A
{% endhighlight %}

and

{% highlight assembly %}
.text:0040130A       cmp     byte_4030A5, 30h ; 6th char of 4th textbox must be 30h (0)
.text:00401311       jz      short loc_401318
{% endhighlight %}

compare the 4th and 6th char of the 4th textbox to two specific values 2 and 0\. So what do we have so far for the 4th textbox:

1.  It must have a strlen of 6
2.  It must be less than 500200
3.  Its 4th char must be 2
4.  Its 6th char must be 0

The next part is quite simple. The strlen from the 2nd textbox is first decremented, then substituted by 5 and EAX is tested, which leads to a true condition if the strlen is again 6:

{% highlight assembly %}
.text:00401318       push    6Ah             ; nIDDlgItem ("106"=2nd textbox)
.text:0040131A       push    [ebp+hDlg]      ; hDlg
.text:0040131D       call    GetDlgItem
.text:00401322       mov     dword_40307C, eax
.text:00401327       lea     eax, byte_403098
.text:0040132D       push    eax             ; lParam (pointer to the buffer that is to receive the text)
.text:0040132E       push    8               ; wParam
.text:00401330       push    0Dh             ; Msg (WM_GETTEXT-0x000D)
.text:00401332       push    dword_40307C    ; hWnd
.text:00401338       call    SendMessageA
.text:0040133D       dec     eax             ; EAX is decremented by 1
.text:0040133E       sub     eax, 5
.text:00401341       test    eax, eax
.text:00401343       jnz     loc_40144C
{% endhighlight %}

After that the values from the textboxes are pushed onto the stack:

{% highlight assembly %}
.text:00401349       push    offset unk_4030A0 ; 1st char of 4th textbox
.text:0040134E       push    offset lParam   ; contains str from 2nd textbox
.text:00401353       push    offset byte_403098 ; 1st char of 2nd textbox
.text:00401358       push    offset unk_403090 ; 1st char of 1st textbox
.text:0040135D       push    offset unk_4030A8 ; NULL byte
.text:00401362       push    4
.text:00401364       call    sub_401610      ; builds string from all chars
.text:00401369       add     esp, 18h
.text:0040136C       lea     esi, unk_4030A8 ; loads the key-string
.text:00401372       mov     ecx, 18h
{% endhighlight %}

The following loop tests if each char is between 0 and 9, which means only numeric chars are allowed in the serial:

[![greedyfly16-5]({{ site.baseurl }}/assets/greedyfly16-5.png)]({{ site.baseurl }}/assets/greedyfly16-5.png)

Then some specific single chars are compared against single values:

{% highlight assembly %}
.text:0040139F                 cmp     byte_403091, 39h ; 2nd char of 1st textbox must be 9
.text:004013A6                 jz      short loc_4013AD
.text:004013A8                 jmp     loc_40144C
.text:004013AD ; ---------------------------------------------------------------------------
.text:004013AD
.text:004013AD loc_4013AD:                             ; CODE XREF: DialogFunc+26A
.text:004013AD                 cmp     byte_403093, 37h ; 4th char of 1st textbox must be 7
.text:004013B4                 jz      short loc_4013BB
.text:004013B6                 jmp     loc_40144C
.text:004013BB ; ---------------------------------------------------------------------------
.text:004013BB
.text:004013BB loc_4013BB:                             ; CODE XREF: DialogFunc+278
.text:004013BB                 cmp     byte_403098, 38h ; 1st char of 2nd textbox must be 8
.text:004013C2                 jnz     loc_40144C
{% endhighlight %}

and

{% highlight assembly %}
.text:004013C8       push    offset unk_403090 ; points to 1st char of 1st textbox
.text:004013CD       call    addup_int_from_char
.text:004013D2       cmp     eax, 7A057h       ; decimal: 499799
.text:004013D7       jbe     short loc_4013DB
{% endhighlight %}

### The final challenge!

Great work until here. But wait something is missing! Correct: 3rd textbox. Remember: The value from the first textbox is already present in EAX. So this is first saved (MOVed) to ESI for later usage:

{% highlight assembly %}
.text:004013DB       mov     esi, eax
{% endhighlight %}

The value from the 4th textbox is pushed onto stack for further usage in the addup_int_from_char function:

{% highlight assembly %}
.text:004013DD       push    offset unk_4030A0 ; points to 1st char of 4 th textbox
.text:004013E2       call    addup_int_from_char
{% endhighlight %}

The function saves its return value to EAX, that is then MOVed to EDI:

{% highlight assembly %}
.text:004013E7       mov     edi, eax
{% endhighlight %}

Afterwards the values from ESI (contains value from first textbox) and EDI (contains value from 4th textbox) are ADDed and saved to ESI:

{% highlight assembly %}
.text:004013E9       add     esi, edi        ; add value from 4th textbox to value from 1st textbox
{% endhighlight %}

Last but not least: the value from the third textbox is compared against the addition from the 1st and 4th textboxes. If those values are the same the jump is taken to the successful - message:

{% highlight assembly %}
.text:004013EB       push    offset lParam   ; points to 1st char of 3rd textbox
.text:004013F0       call    addup_int_from_char
.text:004013F5       cmp     esi, eax        ; compare value from 3rd textbox to 4th+1st
.text:004013F7       jnz     short loc_40144C
{% endhighlight %}

### The KeyGen!

So that's the puzzle. Let's write a KeyGen for it. What do we have so far?

1st textbox:

*   Its 2nd char must be 9
*   its 4th char must be 7
*   It must be less or equal decimal 499799

2nd textbox:

*   Its 1st char must be 8

3rd textbox:

*   Its value is the addition of the first and fourth textboxes

4th textbox:

*   It must have a strlen of 6
*   It must be less than 500200
*   Its 4th char must be 2
*   Its 6th char must be 0

The normal keygenner would write a small and nice-music-driven application,but since this is just a small KeyGenMe, I'll waive ;-). Here's a Python script as a PoC:

{% highlight python %}
#!/usr/bin/python

import random

# first part with schema: x9x7xx
part1=str(random.randint(100000,499799))
part1=part1[0]+ "9" + part1[2] + "7" + part1[4:]

# second part with schema: 8xxxxx
part2=str(random.randint(800000,899999))

# fourth part with schema: xxx2x0
part4=str(random.randint(100000,500200))
part4=part4[:3] + "2" + part2[5] + "0"

# third part with schema: part1+part4 
part3=str(int(part1) + int(part4))

print "Serial: " + part1 + "-" + part2 + "-" + part3 + "-" + part4
{% endhighlight %}

...and finally pwned:

[![greedyfly16-6]({{ site.baseurl }}/assets/greedyfly16-6.png)]({{ site.baseurl }}/assets/greedyfly16-6.png)
