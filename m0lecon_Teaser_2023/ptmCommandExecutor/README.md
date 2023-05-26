# ptmCommandExecutor (m0lecon Teaser 2023 CTF)
Description - Do I have RCE on this?
Category - misc
Points - 72
Author - @matpro

This is the second easiest challenges in m0lecon Teaser 2023, but I wasn't able to solve it. Below is what I've learnt after the end of the competition. 

## Basic analysis
The ptmCommandExecutor program asks for command input (3 commands are accepted, including: get_flag, ls, and lp, but access is restricted for get_flag) and executes it. 
<b>(Insert photo here)</b>

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
