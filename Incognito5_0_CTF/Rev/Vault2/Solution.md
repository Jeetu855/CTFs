Lets decompile the code and also run it wih ltrace

The principle is the same as the previous version of the challenge.
The program compares the given input with a series of operations to determine it is the flag

checkFlag()

```c
uint64_t checkFlag(char* arg1)
{
    int64_t var_28;
    __builtin_strcpy(&var_28, "leyh{V2z4x#3q^x\"wl][0V\x7f");
    void var_a8;
    strncpy(&var_a8, arg1, 0x80);
    encrypt(&var_a8);
    int32_t rax;
    rax = strncmp(&var_a8, &var_28, 0x18) == 0;
    return rax;
}
```

encrypt()

```c
__int64 __fastcall encrypt(__int64 a1)
{
  __int64 result; // rax
  int i; // [rsp+14h] [rbp-Ch]

  for ( i = 0; ; ++i )
  {
    result = *(unsigned __int8 *)(i + a1);
    if ( !(_BYTE)result )
      break;
    *(_BYTE *)(i + a1) = rotateChar(*(_BYTE *)(i + a1) ^ (unsigned __int8)(i % 4), 3);
  }
  return result;
}
```

rotateChar()

```c
__int64 __fastcall rotateChar(char a1, int a2)
{
  if ( a1 > 96 && a1 <= 122 )
    return (unsigned int)((a1 - 97 + a2) % 26 + 97);
  if ( a1 <= 64 || a1 > 90 )
    return (unsigned __int8)a1;
  return (unsigned int)((a1 - 65 + a2) % 26 + 65);
}
```

The operations are : take the cipher text , rotate the current character by 3 and XOR with the index%4.

```python
from pwn import *

ct = b'leyh{V2z4x#3q^x\"wl][0V\x7f'

flag = ""

index = 0

for i in ct:
	if i > 96 and i <=122:
		rotatedI = (i - 97 - 3) % 26 + 97
	else:
		if i <= 64 or i > 90:
			rotatedI = i
		else:
			rotatedI = (i - 65 - 3) % 26 + 65

	letter = rotatedI ^ (index%4)
	index = index + 1

	flag = flag + chr(letter)

print(flag)
```

**_flag : ictf{R0t4t!0n_w!th_X0R}_**
