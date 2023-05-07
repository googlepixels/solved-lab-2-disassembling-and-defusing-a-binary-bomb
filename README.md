Download Link: https://assignmentchef.com/product/solved-lab-2-disassembling-and-defusing-a-binary-bomb
<br>
<h2><a name="_Toc3392"></a>Overview</h2>

The nefarious Dr. Evil has planted a slew of “binary bombs” on our machines. A binary bomb is a program that consists of a sequence of phases. Each phase expects you to type a particular string on stdin. If you type the correct string, then the phase is defused and the bomb proceeds to the next phase. Otherwise, the bomb explodes by printing “BOOM!!!” and then terminating. The bomb is defused when every phase has been defused.

There are too many bombs for us to deal with, so we are giving everyone a bomb to defuse. Your mission, which you have no choice but to accept, is to defuse your bomb before the due date. Good luck, and welcome to the bomb squad!

<h2><a name="_Toc3393"></a>Instructions</h2>

The bombs were constructed specifically for 64-bit machines. You should do this assignment on the 64-bit Linux VM and be sure it works there (or at least test your solution there before submitting it!) so that you can be sure it works when we grade it. In fact, there is a rumor that Dr. Evil has ensured the bomb will always blow up if run elsewhere. There are several other tamper-proofing devices built into the bomb as well, or so they say.

<strong>Perform an update to your course-materials directory </strong>on the VM by running the update-course command from a terminal window. On the script’s success, you should find the provided code for lab2 in your course-materials directory. As a convenience, here is an archive of the course-materials directory as of this lab assignment: <a href="https://spark-public.s3.amazonaws.com/hardware/labs/lab2.tar.gz">lab2.tar.gz</a><a href="https://spark-public.s3.amazonaws.com/hardware/labs/lab2.tar.gz">.</a> The lab2 directory should then contain the following files:

<ul>

 <li><sub>bomb</sub>: The executable binary bomb</li>

 <li><sub>c</sub>: Source file with the bomb’s main routine</li>

 <li><sub>txt</sub>: File in which you write your defusing solution</li>

</ul>

Your job is to defuse the bomb. You can use many tools to help you with this; please look at the tools section for some tips and ideas. The best way is to use a debugger to step through the disassembled binary.

The bomb has 5 regular phases. The 6th phase is extra credit (worth half as much as a regular phase), and rumor has it that a secret 7th phase exists. If it does and you can find and defuse it, you will receive additional extra credit points. The phases get progressively harder to defuse, but the expertise you gain as you move from phase to phase should offset this difficulty. Nonetheless, the latter phases are not easy, so please don’t wait until the last minute to start. (If you’re stumped, check the hints section at the end of this document.)

The bomb ignores blank input lines. If you run your bomb with a command line argument, for example,

./bomb defuser.txt

then it will read the input lines from defuser.txt until it reaches EOF (end of file), and <strong><sub>then switch over </sub>to stdin </strong>(standard input from the terminal). In a moment of weakness, Dr. Evil added this feature so you don’t have to keep retyping the solutions to phases you have already defused.

To avoid accidentally detonating the bomb, you will need to learn how to single-step through the assembly code and how to set breakpoints. You will also need to learn how to inspect both the registers and the memory states. One of the nice side-effects of doing the lab is that you will get very good at using a debugger. This is a crucial skill that will pay big dividends the rest of your career.

<h2><a name="_Toc3394"></a>Resources</h2>

There are a number of online resources that will help you understand any assembly instructions you may encounter while examining the bomb. In particular, the programming manuals for x86-64 processors distributed by Intel and AMD are exceptionally valuable. They both describe the same ISA, but sometimes one may be easier to understand than the other.

<strong>Useful for this Lab</strong>

<ul>

 <li><a href="http://download.intel.com/products/processor/manual/325383.pdf">Intel Instruction Reference</a></li>

 <li><a href="https://support.amd.com/us/Processor_TechDocs/24594_APM_v3.pdf">AMD Instruction Reference</a></li>

</ul>

<strong>Not Directly Useful, but Good Brainfood Nonetheless</strong>

<ul>

 <li><a href="http://download.intel.com/products/processor/manual/253665.pdf">Intel 64 and IA-32 Architectures Software Developer’s Manual Volume 1: Basic Architecture</a></li>

 <li><a href="http://download.intel.com/products/processor/manual/325384.pdf">Intel 64 and IA-32 Architectures Software Developer’s Manual Combined Volumes 3A and 3B: System </a><a href="http://download.intel.com/products/processor/manual/325384.pdf">Programming Guide, Parts 1 and</a> <a href="http://download.intel.com/products/processor/manual/325384.pdf">2</a></li>

 <li><a href="https://support.amd.com/us/Processor_TechDocs/24592_APM_v1.pdf">AMD64 Architecture Programmer’s Manual Volume 1: Application Programming</a></li>

 <li><a href="https://support.amd.com/us/Processor_TechDocs/24593_APM_v2.pdf">AMD64 Architecture Programmer’s Manual Volume 2: System Programming</a></li>

 <li><a href="https://support.amd.com/us/Processor_TechDocs/26568_APM_v4.pdf">AMD64 Architecture Programmer’s Manual Volume 4: 128-bit and 256 bit media instructions</a></li>

 <li><a href="https://support.amd.com/us/Processor_TechDocs/26569_APM_v5.pdf">AMD64 Architecture Programmer’s Manual Volume 5: 64-Bit Media and x87 Floating-Point Instructions </a><strong>x86-64 Calling Conventions</strong></li>

</ul>

As a reminder, the x86-64 ISA passes the first six arguments to a function in registers. Registers are used in the following order: <sub>rdi</sub>, <sub>rsi</sub>, <sub>rdx</sub>, <sub>rcx</sub>, <sub>r8</sub>, <sub>r9</sub>. The return value for functions is passed in <sub>rax</sub>.

<h2><a name="_Toc3395"></a>Tools (Read This!!)</h2>

There are many ways of defusing your bomb. You can examine it in great detail without ever running the program, and figure out exactly what it does. This is a useful technique, but it not always easy to do. You can also run it under a debugger, watch what it does step by step, and use this information to defuse it. This is probably the fastest way of defusing it.

We do make one request, please do not use brute force! You could write a program that will try every possible key to find the right one, but the number of possibilities is so large that you won’t be able to try them all in time.

There are many tools which are designed to help you figure out both how programs work, and what is wrong when they don’t work. Here is a list of some of the tools you may find useful in analyzing your bomb, and hints on how to use them.

<ul>

 <li><sub>gdb</sub>: The GNU debugger is a command line debugger tool available on virtually every platform. You can trace through a program line by line, examine memory and registers, look at both the source code and assembly code (we are not giving you the source code for most of your bomb), set breakpoints, set memory watch points, and write scripts. Here are some tips for using <sub>gdb</sub>.

  <ul>

   <li>To keep the bomb from blowing up every time you type in a wrong input, you’ll want to learn how to set breakpoints.</li>

   <li>The CS:APP Student Site has a very handy <a href="http://csapp.cs.cmu.edu/public/docs/gdbnotes-x86-64.pdf">gdb summary</a> (there is also a <a href="http://heather.cs.ucdavis.edu/~matloff/UnixAndC/CLanguage/Debug.html">more extensive tutorial</a><a href="http://heather.cs.ucdavis.edu/~matloff/UnixAndC/CLanguage/Debug.html">)</a>.</li>

   <li>For other documentation, type <sub>help </sub>at the <sub>gdb </sub>command prompt, or type “man gdb”, or “info gdb” at a Unix prompt. Some people also like to run <sub>gdb </sub>under gdb-mode in emacs</li>

  </ul></li>

 <li><sub>objdump -t bomb</sub>: This will print out the bomb’s symbol table. The symbol table includes the names of all functions and global variables in the bomb, the names of all the functions the bomb calls, and their addresses. You may learn something by looking at the function names!</li>

 <li><sub>objdump -d bomb</sub>: Use this to disassemble all of the code in the bomb. You can also just look at individual functions. Reading the assembler code can tell you how the bomb works. Although <sub>objdump </sub>–<sub>d </sub>gives you a lot of information, it doesn’t tell you the whole story. Calls to system-level functions may look cryptic. For example, a call to sscanf might appear as: 8048c36: e8 99 fc ff ff call 80488d4 &lt;<sub>_init+0x1a0&gt; </sub>To determine that the call was to <sub>sscanf</sub>, you would need to disassemble within <sub>gdb</sub>.</li>

 <li><sub>strings -t x bomb</sub>: This utility will display the printable strings in your bomb and their offset within</li>

</ul>

the bomb.

Looking for a particular tool? How about documentation? Don’t forget, the commands <sub>apropos </sub>and <sub>man </sub>are your friends. In particular, <sub>man ascii </sub>is more useful than you’d think. If you get stumped, use the course’s discussion board.

<h2><a name="_Toc3396"></a>Hints</h2>

If you’re still having trouble figuring out what your bomb is doing, here are some hints for what to think about at each stage: (1) comparison, (2) loops, (3) switch statements, (4) recursion, (5) pointers and arrays, (6) sorting linked lists.


