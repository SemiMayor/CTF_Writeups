# OMGzip (DEFCON CTF 2023 Qualifiers)

Description - OMG, it's a .zip!  
Category - Quals  
Points - 56  

Quote from the authors
> This challenge is intended as an easier reverse engineering challenge for players unfamiliar with the category or assembly.
> 
> Inside the file below, you will find a Python script and a compressed flag file. Read over the Python code, write code that does the opposite, and decompress the flag.
> 
> The original version of this challenge was pretty trivially solvable by ChatGPT, so I made it more difficult before its release and added a bunch of random comments and changed a bunch of variable names to make doing that take more time. We'll see if I was successful or not.

I didn't work on this challenge because I was too focuesed on openhouse. Just a few hours before the end of the competition, I realized that there are challenges
easier than openhouse. But it was too late to work on this one.

This is a straight forward reverse challenge, all you need to do is understand the python code and write a code that does the opposite,
but it is not so easy to understand how the code works. Instead of presenting in detail how the code works and how to reverse it,
I would like to keep it brief and focus on showing methods I used to help myself understand the code.

## How the code works?

Basically the python code compresses a file in two steps.

First, it execute a function `compress()`, which looks for repeating bytes (or characters) like `AAAAAA`, and replace it with something like `\xff\x03A`, where the first byte is for indication of compressed
bytes, second byte stores the number of repeating bytes - 3 (or - 2 in some cases), and the third byte store the repeating byte.

Second, it execute a function `encode()`, which encodes the compressed string one byte at a time using `_travesty()`,
in which it uses a tree diagram similar to the Huffman Tree.
Upon encoding each byte, `_magic()` is called, which swaps two nodes in the Tree diagram.

## Difficulties in reversing

The program is fairly complicated and long, so it would take some time if only static analysis (code inspection) is used.
Some comments and variable names are confusing, like the Tree nodes with parent node and two child nodes are named as Family with "overbearing parents",
"successful_firstborn" and "conflicted_stepchild",
and the tree diagram method is not something explainable in a few sentences either.
Furthermore, some lines of code are written in a confusing way that I can hardly guess what will happen during static analysis.

## Methods I adopted to understand the code

### Dynamic analysis with pdb
Run the code with python debugger pdb, and see what each function does on its input. This may be the quickest way to learn how it works.
One can also trace the functions to see what each line does.

### Drawing diagrams
The function `_create_family()` creates tree nodes recursively, and is not so easy to trace it with debugger.
In static analysis, drawing diagram helps a lot in understanding what this recursive function creates.

### Copy the function to a new program for debug
In particular, the function `_magic()` is hard to understand. The last line of the function looks buggy and I cannot guess its function by static analysis.
Dynamic analysis is difficult either, because the tree is too large to be printed.
I decided to move the entire function to a new program so I can input small trees and do dynamic analysis on the function.
Maybe there are better ways than to make a new program, but this is what I came up with, and it works reasonably well.

## Solution

I do not show the decompression code or explain how to write it, because it is not particularly difficult.
Just need a clear understanding of how the file is compressed, and come up with a strategy to reverse it.

Here's the flag:
