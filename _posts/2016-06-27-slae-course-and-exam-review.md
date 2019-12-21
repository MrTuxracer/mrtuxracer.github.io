---
title: SLAE Course and Exam Review
categories:
- Certifications
---
As you may have noticed, I have posted a [couple of articles](https://www.rcesecurity.com/category/slae/) about my [SecurityTube Linux Assembly Expert](http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/) exam during the last months. Now that I have successfully completed the course, I just want to share my thoughts about it for those of you who think about taking the course but are unsure if it's the right one.

## What is it about?

The SLAE course is basically an online, self-learning course brought to you by SecurityTube and aimed at shellcoding on Linux. Quoted from their website:

> The SecurityTube Linux Assembly Expert (SLAE) is an online course and certification which focuses on teaching the basics of 32-bit assembly language for the Intel Architecture (IA-32) family of processors on the Linux platform and applying it to Infosec. Once we are through with the basics, we will look at writing shellcode, encoders, decoders, crypters and other advanced low level applications.

## Who should take it?

It's an entry-level course, which teaches you everything around shellcoding on 32-Bit Linux from the very basics. So in case you have no foreknowledge in Assembly or shellcoding, it's absolutely no problem and you can enroll. But be prepared that you should bring some Linux as well as C skills in order to follow the techniques and tricks, otherwise you would need some time to get into it.

## What about the course materials?

The course materials consist of ~260 pages of slides, which come along with approx. 9 hours of video material. These 9 hours which are presented by Vivek Ramachandran himself are split into 35 modules covering different topics from very basic things like:

*   32Bit Architecture Basics
*   CPU Modes and Memory
*   Hello World with GDB excercise!
*   Assembly Language Basics:
    *   Data Types
    *   Moving Data
    *   The Stack
    *   Arithmetics
    *   Multiplication and Division
    *   Logical Instructions
    *   Control Instructions
    *   Procedures
    *   State Saving
    *   Strings
*   Libc and NASM

Up to very specific and really interesting topics:

*   Shellcode Basics
*   Exit Shellcode
*   JMP-CALL-POP Shellcodes
*   Execve Shellcode
*   XOR Encoder/Decoder
*   Metasploit Encoders
*   NOT Encoder
*   Insertion Encoder
*   Polymorphism
*   Analyzing Shellcode
*   Crypters

The quality of both the slides and the videos is quite good, although there are sometimes some annoying sounds coming from a nearby street, which you can hear in the videos. However this doesn't matter that much since Vivek is constantly attracting your attention to the in-depth contents of the discussed topic.

## The Course and Exam Concept

Unlike many other courses (see my previously passed [OSCP](https://www.rcesecurity.com/2013/05/oscp-course-and-exam-review/)), where you have to complete a CTF-style exam and write a documentation, the SLAE goes a slightly different approach. You can start the exam at any time you want, regardless of your progress on the course materials. So you can do it after working through the slides and videos or even while you're working through them. The reason for this is that the exam isn't delivered as a dedicated piece, but it's part of the initial set of course materials. This does also mean that there is no time-limit on the exam, because it cannot be simply estimated how quick the student is going through the slides and how much foreknowledge he has.

The exam itself consists of 7 questions or rather tasks. It's not that kind of multiple-choice stuff you might know from expensive courses, instead they are just broadly described. One example, which you can also find in the [publicly available exam video](http://www.securitytube.net/video/6984):

*   Create a custom crypter.
    *   You can (but do not need to) use any existing encryption schema
    *   You can use any programming language.

This broadly defined scope forces the student to put some creativity into his exam work, because there are unlimited ways of solving this assignment. The only rule (beside the scope of the assignments) is that the student has to create a public blog post describing his creativity for every assignment in order to pass the exam. These two conditions lead to a very interesting and open-minded behavior because you can simply google all exam posts, just like:

*   [SLAE: Custom Crypter (Linux/x86)](https://www.rcesecurity.com/2016/04/slae-custom-crypter-linux-x86/)
*   [http://cloud101.eu/linux/security/programming/slae32/2013/05/07/slae-assignment-7-custom-crypter.html](http://cloud101.eu/linux/security/programming/slae32/2013/05/07/slae-assignment-7-custom-crypter.html)
*   [http://networkfilter.blogspot.de/2015/08/slae-assignment-7-custom-crypter.html](http://networkfilter.blogspot.de/2015/08/slae-assignment-7-custom-crypter.html)

and countless others. Although a student could simply copy the results from another student to pass the exam, I guess the SecurityTube team will likely check that ;-) So this is a real plus for this course!

In addition to the blog posts, you could also get extra points by interacting with the the community. So posting new shellcodes to Exploit-DB or shell-storm, twittering about topics or answering other's question will get you some extra points.

## Conclusion

For just **USD150**, you will get a really amazing, competitive amount of in-depth technical information on 32-Bit Linux shellcoding! Although I already had some quite decent Assembly skills due to my previous work on Windows-based exploits, it was somehow still a new world to discover, and it was presented by Vivek very well! Before registering for this course, I haven't had a real look at shellcoding itself, not to mention polymorphism and shellcode encryption at all. However after this course I have written polymorphic versions of 3rd party shellcodes and even created my own crypter. Yay :-)

**Pros:**

*   With USD150 it's a really cheap course with in-depth technical information on the topic
*   The explanatory videos describe the topics very detailed and easy to follow
*   The exam approach is open-minded and gives a great value back to the whole security community

**Cons:**

*   Sometimes there are some background noises in the videos which are a bit annoying but they do not lower the course experience that much

All in all this course is a very good investment into your security career. It's even a good addition to your OSCP preparation!
