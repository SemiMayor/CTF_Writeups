# ptmCommandExecutor (m0lecon Teaser 2023 CTF)
Description - Do I have RCE on this?
Category - misc
Points - 72
Author - @matpro

This is the second easiest challenges in m0lecon Teaser 2023, but I wasn't able to solve it. Below is what I've learnt after the end of the competition. 

## Basic analysis
The ptmCommandExecutor program asks for command input (3 commands are accepted, including: get_flag, ls, and lp, but access is restricted for get_flag) and executes it.

## Solution
get_flag\x00 (The null byte should be typed in directly)

## My attempt
The solution above is provided by others after the end of the competition, but they didn't create any writeups or explain how they get the solution. So I try to figure it out on my own (spoiler: it wasn't completely successful)
