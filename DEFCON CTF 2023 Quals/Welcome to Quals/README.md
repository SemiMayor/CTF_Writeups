# Welcome to Quals (DEF CON CTF 2023 Qualifiers)

Category - intro  
Points - 10

This is the warmup challenge of DEF CON CTF 2023 Quals

## Basic Analysis

We were told nothing and given nothing about the challenge, so I just nc to the target to see what do we get.
Upon connecting, it told us to give a payload there.
I tried `ls`, but the response was an error message mentioning something called `yf`.
I wondered what it is, so I typed in `yf` instead, and I got some filenames.
I further confirmed that `ls` had been executed when I input `yf`, since when I typed `yf -l`, it gave an error of unknown `ls` options.

Obviously, the input was encoded by shifting the alphabets. We simply needed to shift our payload to get it executed in the target.

## Solution

I automated the work with a bash script. Basically, it shifts by 13 the alphabets of the command, and sends it through netcat.
```
#!/bin/bash

cmd="cat ../welcome_flag.txt"
for ((i=0; i<${#cmd}; i++)){
        char=${cmd:i:1}
        ascii=$(printf "%d" "'$char")
        if [ $ascii -le 122 ] && [ $ascii -ge 97 ]
        then
                ascii=`expr $ascii + 13`
        fi
        if [ $ascii -gt 122 ]
        then
                ascii=`expr $ascii - 26`
        fi
        encode_char[$i]=$(printf "\x$(printf "%x" $ascii)")
}

for ((i=0; i<${#encode_char[@]}; i++)){
        encode_str+=${encode_char[$i]}
}
echo "$encode_str"

nc welcome-to-quals-vfnva65rlchqk.shellweplayaga.me 10001 << ENDING
        ticket{RentAssociation9804n23:iQqGU13cuCqjx7KcxOmBotxLqrNJCo389Q6epFvmtFX4AXFG}
        $encode_str
ENDING
```
