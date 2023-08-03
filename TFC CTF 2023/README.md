# TFC CTF 2023 writeups

I've solved 6 challenges in this competition, including:  
ducky notes (web, warmup), pass (reverse, warmup), the first two crypto challenges (warmup), list (forensic, easy) and one misc challenge (which is warmup of the warmup).

Some of them are so simple that I will not make separate pages for them, so let me just talk about them here.

## Warmup Misc challenge

Just look at the rules and the example flag is the correct answer. Nothing special.

## BABY DUCKY NOTES

A simple website that allows you to register, login, create posts and view them. Clicking on "View", one arrives at `web/posts/<username>` and your own posts are shown.
I wondered if we can take a look at admin's post, so I changed the URL to `web/posts/admin`. Surprisingly, it worked, and I got the flag.

## MAYDAY!

This is the first crypto challenge in the competition. Description below:  
We are sinking! The nearest ship got our SOS call, but they replied in pure gobbledygook! Are ye savvy enough to decode the message,
or will we be sleepin’ with the fish tonight? All hands on deck!  
Whiskey Hotel Four Tango Dash Alpha Romeo Three Dash Yankee Oscar Uniform Dash Sierra One November Kilo India November Golf Dash Four Bravo Zero Uniform Seven  
Flag format: TFCCTF{RESUL7-H3R3}

My initial guess was that maybe it was the initials. Some words were numbers so I guessed they're actually numbers as there were numbers in the flag format. I was right.

Solution: TFCCTF{WH4T-AR3-YOU-S1NKING-4B0U7}  

## DIZZY

The second crypto challenge but just as easy as the first one. Description below:  
Embark on ‘Dizzy’, a carousel ride through cryptography! This warmup challenge spins you around the basics of ciphers and keys. Sharpen your mind, find the flag, and remember — in crypto, it’s fun to get a little dizzy!  
T4 l16 _36 510 _27 s26 _11 320 414 {6 }39 C2 T0 m28 317 y35 d31 F1 m22 g19 d38 z34 423 l15 329 c12 ;37 19 h13 _30 F5 t7 C3 325 z33 _21 h8 n18 132 k24

It seemed to me that the number after each character may be the position of the character.  
Solution: TFCCTF{th15_ch4ll3ng3_m4k3s_m3_d1zzy_;d}
