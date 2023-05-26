# ptmCommandExecutor (m0lecon Teaser 2023 CTF)
Description - Do I have RCE on this?
Category - misc
Points - 72
Author - @matpro

This is the second easiest challenges in m0lecon Teaser 2023, but I wasn't able to solve it. Below is what I've learnt after the end of the competition. 

## Basic analysis
The ptmCommandExecutor program asks for command input (3 commands are accepted, including: get_flag, ls, and lp, but access is restricted for get_flag) and executes it.  

![](ptm1.png)

## Solution
get_flag\x00 (The null byte should be typed in directly)

## My attempt
The solution above is provided by others after the end of the competition, but they didn't create any writeups or explain how they get the solution. So I try to figure it out on my own (spoiler: it wasn't completely successful):

The program is a binary executable, we can use the `file` command to get some basic information about it.
```
ptmCommandExecutor: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f685ce785d19af7c66c2297eacd04c771fce437d, for GNU/Linux 3.2.0, stripped
```
`ELF` means it's an executable, `LSB` means little endian is used, `stripped` means debugging symbols are removed, which makes it difficult to debug (e.g. you can't search the main function directly).

I tried commands like ltrace, strace, strings on the binary file, but didn't find anything suspicious. So a full reverse is needed to figure out how the program works.

## Reverse Engineering
I used ghidra, an open-source reverse engineering software to reverse the file.

![](ptm_rev1.png)

The first thing to do to reverse a file is to find the main function. Since the debugging symbols are stripped, we would find nothing by searching for 'main'. But for a ELF file, the program code is always stored in the '.text' section, so we can start from there. You can see that in the 'Program Trees' panel in the upper-left corner, those in the panel are headers in the file.

In the middle panels, we have the disassembled code, these are called the assembly code. It's not easy to understand them, but we can focus on the calls and jumps in the assembly code, which outline the structure of the program. Since main is usually the first function called in a program, the first function called here (look at the highlighted line) is usually the main function. But notice that it is not directly calling the main function, but instead calls `__libc_start_main` which takes the address of the main function as a parameter.
